# 공통 플래그와 기본 실행

공통 플래그는 기본 `codex` 명령과 여러 하위 명령에서 실행 환경을 정하는 옵션입니다. 모델, 작업 디렉터리, sandbox, 승인 방식, 이미지 입력, 원격 연결, 설정 override 등이 여기에 포함됩니다.

## 기본 입력

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `PROMPT` | `string` | 세션을 시작할 때 사용할 선택적 텍스트 지시입니다. 생략하면 미리 채워진 메시지 없이 TUI가 열립니다. |
| `--image`, `-i` | `path[,path...]` | 초기 프롬프트에 이미지 파일을 하나 이상 첨부합니다. 여러 경로는 쉼표로 구분하거나 플래그를 반복합니다. |

예:

```bash
codex "이 코드베이스의 핵심 모듈을 설명해줘"
codex -i screenshot.png "이 오류 화면을 분석해줘"
codex --image before.png,after.png "두 화면의 UI 차이를 정리해줘"
```

## 모델과 provider

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--model`, `-m` | `string` | 설정값 | 설정에 지정된 모델을 이번 실행에서만 덮어씁니다. 예: `gpt-5.4`, `gpt-5.5` |
| `--oss` | `boolean` | `false` | 로컬 오픈소스 모델 provider를 사용합니다. `-c model_provider="oss"`와 같습니다. Ollama가 실행 중인지 검증합니다. |
| `--profile`, `-p` | `string` | 없음 | `~/.codex/config.toml`에 정의된 configuration profile을 불러옵니다. |

예:

```bash
codex --model gpt-5.5 "리팩터링 계획을 세워줘"
codex --oss "이 작은 스크립트를 설명해줘"
codex --profile work "회사 프로젝트 설정으로 실행해줘"
```

`--oss`는 로컬 Ollama 기반 워크플로를 쓸 때 편하지만, 모델 품질과 도구 처리 능력은 선택한 로컬 모델에 좌우됩니다.

## Sandbox와 승인

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `--sandbox`, `-s` | `read-only \| workspace-write \| danger-full-access` | 모델이 생성한 shell command에 적용할 sandbox 정책을 고릅니다. |
| `--ask-for-approval`, `-a` | `untrusted \| on-request \| never` | 명령 실행 전 Codex가 언제 사람 승인을 요청할지 정합니다. |
| `--dangerously-bypass-approvals-and-sandbox`, `--yolo` | `boolean` | 승인과 sandbox를 모두 우회하고 모든 명령을 실행합니다. 외부에서 강하게 격리된 환경에서만 사용해야 합니다. |

권장 조합:

```bash
codex --sandbox workspace-write --ask-for-approval on-request
```

비대화형 실행에서 멈추면 안 되는 경우:

```bash
codex exec --sandbox workspace-write --ask-for-approval never "lint 실패를 수정해줘"
```

위험한 전체 권한 실행:

```bash
codex --dangerously-bypass-approvals-and-sandbox
```

`--yolo`는 편의 옵션이 아니라 위험 옵션입니다. VM, 컨테이너, CI runner처럼 외부에서 격리된 환경이 아니라면 피하는 것이 좋습니다.

참고로 `on-failure` approval 모드는 deprecated입니다. 인터랙티브 실행은 `on-request`, 비대화형 실행은 `never`를 우선 고려합니다.

## 작업 디렉터리와 추가 쓰기 권한

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `--cd`, `-C` | `path` | 요청 처리 전에 agent의 작업 디렉터리를 설정합니다. |
| `--add-dir` | `path` | 메인 workspace와 함께 추가 디렉터리에 쓰기 권한을 부여합니다. 여러 경로는 플래그를 반복합니다. |

예:

```bash
codex --cd apps/frontend "현재 앱 구조를 설명해줘"
```

모노레포에서 여러 패키지를 함께 수정해야 할 때:

```bash
codex \
  --cd apps/frontend \
  --add-dir ../backend \
  --add-dir ../shared \
  "프론트엔드와 API 타입 변경을 함께 반영해줘"
```

추가 쓰기 권한이 필요할 때는 `--sandbox danger-full-access`보다 `--add-dir`를 먼저 고려하는 것이 안전합니다.

## 웹 검색

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--search` | `boolean` | `false` | live web search를 활성화합니다. 기본 `web_search = "cached"` 대신 `web_search = "live"`를 적용합니다. |

예:

```bash
codex --search "현재 Codex CLI 최신 옵션을 확인하고 요약해줘"
```

캐시 검색은 미리 색인된 결과를 사용하고, live 검색은 최신성이 필요한 경우에 적합합니다. 보안 관점에서는 웹 결과를 항상 신뢰할 수 없는 입력으로 봐야 합니다.

## TUI 표시와 원격 연결

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--no-alt-screen` | `boolean` | `false` | TUI alternate screen mode를 비활성화합니다. 이번 실행에서 `tui.alternate_screen` 설정을 덮어씁니다. |
| `--remote` | `ws://host:port \| wss://host:port` | 없음 | interactive TUI를 원격 app-server WebSocket endpoint에 연결합니다. |
| `--remote-auth-token-env` | `ENV_VAR` | 없음 | `--remote` 연결 시 bearer token을 읽을 환경 변수 이름입니다. |

원격 연결 예:

```bash
codex app-server --listen ws://127.0.0.1:4500
codex --remote ws://127.0.0.1:4500
```

인증 토큰을 사용하는 경우:

```bash
export CODEX_REMOTE_TOKEN="$(cat ~/.codex/app-server-token)"
codex --remote wss://remote-host:4500 --remote-auth-token-env CODEX_REMOTE_TOKEN
```

`--remote-auth-token-env`는 반드시 `--remote`와 함께 사용합니다. 토큰은 `wss://` URL 또는 host가 `localhost`, `127.0.0.1`, `::1`인 `ws://` URL에서만 전송됩니다.

원격 모드는 `codex`, `codex resume`, `codex fork`에서 지원됩니다. 다른 하위 명령은 remote mode를 거부합니다.

## 기능 플래그와 설정 override

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `--enable` | `feature` | feature flag를 강제로 켭니다. 내부적으로 `-c features.<name>=true`로 변환됩니다. 반복 가능합니다. |
| `--disable` | `feature` | feature flag를 강제로 끕니다. 내부적으로 `-c features.<name>=false`로 변환됩니다. 반복 가능합니다. |
| `--config`, `-c` | `key=value` | 설정 값을 이번 실행에서만 덮어씁니다. 값은 가능하면 JSON으로 파싱되고, 아니면 문자열로 사용됩니다. |

예:

```bash
codex --enable unified_exec "테스트를 실행하고 실패를 수정해줘"
codex --disable shell_snapshot "현재 변경사항을 설명해줘"
codex -c model=\"gpt-5.5\" -c web_search=\"live\" "최신 문서를 확인해줘"
```

`--enable`과 `--disable`은 일회성 override입니다. 설정 파일에 영구 저장하려면 `codex features enable <feature>` 또는 `codex features disable <feature>`를 사용합니다.

