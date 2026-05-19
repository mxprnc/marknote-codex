# 샌드박스, execpolicy, 안전 팁

Codex CLI는 agent가 생성한 명령을 sandbox와 approval 정책 아래에서 실행할 수 있습니다. 이 문서는 `codex sandbox`, `codex execpolicy`, 위험한 옵션 조합, 실무 권장 패턴을 정리합니다.

## 핵심 원칙

- 로컬 무인 작업에는 `--sandbox workspace-write`를 우선 사용합니다.
- 추가 디렉터리 쓰기 권한이 필요하면 `--add-dir`를 먼저 고려합니다.
- `--dangerously-bypass-approvals-and-sandbox` 또는 `--yolo`는 dedicated sandbox VM, container, CI runner처럼 외부 격리가 있는 환경에서만 사용합니다.
- CI에서는 `--json`과 `--output-last-message`를 함께 사용하면 진행 이벤트와 최종 요약을 모두 남길 수 있습니다.

## 승인과 sandbox 조합

인터랙티브 로컬 작업:

```bash
codex --sandbox workspace-write --ask-for-approval on-request
```

읽기 전용 분석:

```bash
codex --sandbox read-only --ask-for-approval on-request "이 코드의 위험한 부분을 설명해줘"
```

비대화형 CI 작업:

```bash
codex exec \
  --sandbox workspace-write \
  --ask-for-approval never \
  "테스트 실패를 수정해줘"
```

위험한 전체 우회:

```bash
codex --yolo
```

`--yolo`는 승인을 묻지 않고 sandbox도 적용하지 않습니다. 신뢰할 수 없는 저장소, 로컬 개인 환경, 민감한 credential이 있는 머신에서는 피해야 합니다.

## `codex execpolicy`

`execpolicy` rule 파일을 저장하기 전에 평가할 수 있습니다. preview 기능입니다.

기본 형태:

```bash
codex execpolicy check --rules ~/.codex/rules/base.rules -- npm test
```

옵션:

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--rules`, `-r` | `path` 반복 가능 | 없음 | 평가할 execpolicy rule 파일 경로입니다. 여러 파일을 조합할 수 있습니다. |
| `--pretty` | `boolean` | `false` | JSON 결과를 보기 좋게 출력합니다. |
| `COMMAND...` | var-args | 없음 | 지정한 policy에 대해 검사할 명령입니다. |

여러 rule 파일 조합:

```bash
codex execpolicy check \
  --rules ~/.codex/rules/base.rules \
  --rules .codex/project.rules \
  --pretty \
  -- npm run deploy
```

출력은 가장 엄격한 결정과 matching rule 정보를 JSON으로 보여줍니다. 명령이 allow, prompt, block 중 어디에 해당하는지 확인하는 용도로 사용합니다.

## `codex sandbox`

`codex sandbox`는 Codex가 내부적으로 사용하는 sandbox 정책으로 임의 명령을 실행하는 helper입니다.

운영체제별 backend:

- macOS: Seatbelt
- Linux: Landlock + seccomp
- Windows: native Windows sandbox

공통적으로 명령은 `--` 뒤에 전달하는 형태를 사용합니다.

```bash
codex sandbox macos -- npm test
codex sandbox linux -- npm test
codex sandbox windows -- npm test
```

실제 하위 명령 이름은 설치 버전과 플랫폼에 따라 다를 수 있으므로 `codex sandbox --help`로 확인하는 것이 좋습니다.

## macOS Seatbelt 옵션

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--permissions-profile` | `NAME` | 없음 | 활성 configuration stack에서 named permissions profile을 적용합니다. |
| `--cd`, `-C` | `DIR` | 없음 | profile resolution과 command execution에 사용할 작업 디렉터리입니다. `--permissions-profile`이 필요합니다. |
| `--include-managed-config` | `boolean` | `false` | 명시적 permissions profile을 resolve할 때 managed requirements를 포함합니다. |
| `--allow-unix-socket` | `path` | 없음 | sandboxed command가 이 path 아래 Unix socket을 bind/connect할 수 있게 허용합니다. 반복 가능합니다. |
| `--log-denials` | `boolean` | `false` | 명령 실행 중 macOS sandbox denial을 `log stream`으로 캡처하고 종료 후 출력합니다. |
| `--config`, `-c` | `key=value` | 없음 | sandboxed run에 설정 override를 전달합니다. 반복 가능합니다. |
| `COMMAND...` | var-args | 없음 | macOS Seatbelt 아래에서 실행할 shell command입니다. `--` 뒤의 모든 값이 전달됩니다. |

예:

```bash
codex sandbox macos \
  --permissions-profile workspace-write \
  --cd . \
  --log-denials \
  -- npm test
```

Unix socket 허용:

```bash
codex sandbox macos \
  --permissions-profile workspace-write \
  --cd . \
  --allow-unix-socket /tmp/my-app \
  -- npm run integration
```

## Linux Landlock 옵션

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--permissions-profile` | `NAME` | 없음 | 활성 configuration stack에서 named permissions profile을 적용합니다. |
| `--cd`, `-C` | `DIR` | 없음 | profile resolution과 command execution에 사용할 작업 디렉터리입니다. `--permissions-profile`이 필요합니다. |
| `--include-managed-config` | `boolean` | `false` | 명시적 permissions profile을 resolve할 때 managed requirements를 포함합니다. |
| `--config`, `-c` | `key=value` | 없음 | sandbox launch 전에 적용할 설정 override입니다. 반복 가능합니다. |
| `COMMAND...` | var-args | 없음 | Landlock + seccomp 아래에서 실행할 command입니다. executable은 `--` 뒤에 제공합니다. |

예:

```bash
codex sandbox linux \
  --permissions-profile workspace-write \
  --cd . \
  -- npm test
```

## Windows sandbox 옵션

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--permissions-profile` | `NAME` | 없음 | 활성 configuration stack에서 named permissions profile을 적용합니다. |
| `--cd`, `-C` | `DIR` | 없음 | profile resolution과 command execution에 사용할 작업 디렉터리입니다. `--permissions-profile`이 필요합니다. |
| `--include-managed-config` | `boolean` | `false` | 명시적 permissions profile을 resolve할 때 managed requirements를 포함합니다. |
| `--config`, `-c` | `key=value` | 없음 | sandbox launch 전에 적용할 설정 override입니다. 반복 가능합니다. |
| `COMMAND...` | var-args | 없음 | native Windows sandbox 아래에서 실행할 command입니다. executable은 `--` 뒤에 제공합니다. |

예:

```powershell
codex sandbox windows --permissions-profile workspace-write --cd . -- npm test
```

## 위험한 조합 피하기

### `--yolo`와 민감한 로컬 환경

나쁜 예:

```bash
codex --yolo "이 저장소를 빌드하고 배포해줘"
```

더 나은 예:

```bash
codex --sandbox workspace-write --ask-for-approval on-request "빌드 실패를 수정해줘"
```

배포, credential 접근, 외부 네트워크 요청이 포함될 수 있는 작업은 approval을 유지하는 편이 안전합니다.

### 쓰기 범위 확장을 위해 full access 사용

나쁜 예:

```bash
codex --sandbox danger-full-access "frontend와 backend를 함께 수정해줘"
```

더 나은 예:

```bash
codex \
  --cd apps/frontend \
  --add-dir ../backend \
  --sandbox workspace-write \
  "frontend와 backend를 함께 수정해줘"
```

필요한 경로만 열어 주면 실수로 무관한 디렉터리를 건드릴 가능성을 줄일 수 있습니다.

### CI에서 최종 결과만 캡처

CI에서 formatted text만 남기면 후속 처리가 어렵습니다. 다음처럼 JSONL 이벤트와 최종 메시지를 분리하면 안정적입니다.

```bash
codex exec \
  --json \
  --output-last-message codex-summary.md \
  --sandbox workspace-write \
  --ask-for-approval never \
  "문서 링크를 검증하고 깨진 링크를 수정해줘"
```

## 실무 체크리스트

- 작업 디렉터리는 `--cd`로 명시합니다.
- 모노레포에서는 필요한 sibling directory만 `--add-dir`로 추가합니다.
- local unattended 작업은 `workspace-write`를 기본값으로 생각합니다.
- 사람이 중간 판단해야 하는 작업은 `on-request`를 유지합니다.
- 스크립트에서는 `--json`을 사용하고, 사람이 읽을 요약은 `--output-last-message`로 분리합니다.
- rule 변경 전에는 `codex execpolicy check`로 의도한 allow/prompt/block 결과가 나오는지 확인합니다.

