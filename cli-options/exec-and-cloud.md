# 비대화형 실행과 Codex Cloud

`codex exec`와 `codex cloud`는 Codex를 스크립트, CI, 자동화 환경에서 쓰기 위한 핵심 명령입니다. interactive TUI를 열지 않고 결과를 stdout, JSONL, 파일로 받을 수 있습니다.

## `codex exec`

`codex exec`는 Codex를 비대화형으로 실행합니다. 짧은 별칭은 `codex e`입니다.

```bash
codex exec "fix the CI failure"
codex e "README의 설치 설명을 현재 코드에 맞게 업데이트해줘"
```

기본적으로 사람이 읽기 좋은 formatted output을 출력합니다. `--json`을 추가하면 상태 변화마다 newline-delimited JSON event를 출력합니다.

```bash
codex exec --json "테스트 실패를 분석해줘"
```

## `codex exec` 옵션

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `PROMPT` | `string \| -` | 없음 | 작업의 초기 지시입니다. `-`를 쓰면 stdin에서 프롬프트를 읽습니다. |
| `--image`, `-i` | `path[,path...]` | 없음 | 첫 메시지에 이미지를 첨부합니다. 반복 가능하며 쉼표 구분 목록도 지원합니다. |
| `--model`, `-m` | `string` | 설정값 | 이번 실행의 모델을 덮어씁니다. |
| `--oss` | `boolean` | `false` | 로컬 open source provider를 사용합니다. Ollama 실행이 필요합니다. |
| `--sandbox`, `-s` | `read-only \| workspace-write \| danger-full-access` | 설정값 | 모델이 생성한 command의 sandbox 정책입니다. |
| `--profile`, `-p` | `string` | 없음 | `config.toml`에 정의된 profile을 선택합니다. |
| `--full-auto` | `boolean` | `false` | deprecated 호환성 플래그입니다. `--sandbox workspace-write`를 사용하세요. |
| `--dangerously-bypass-approvals-and-sandbox`, `--yolo` | `boolean` | `false` | 승인 prompt와 sandbox를 우회합니다. 격리된 runner에서만 사용합니다. |
| `--cd`, `-C` | `path` | 현재 디렉터리 | 실행 전 workspace root를 설정합니다. |
| `--skip-git-repo-check` | `boolean` | `false` | Git 저장소 밖에서도 실행을 허용합니다. |
| `--ephemeral` | `boolean` | `false` | session rollout 파일을 디스크에 저장하지 않고 실행합니다. |
| `--ignore-user-config` | `boolean` | `false` | `$CODEX_HOME/config.toml`을 불러오지 않습니다. 인증은 여전히 `CODEX_HOME`을 사용합니다. |
| `--ignore-rules` | `boolean` | `false` | user/project execpolicy `.rules` 파일을 불러오지 않습니다. |
| `--output-schema` | `path` | 없음 | 최종 응답 형태를 설명하는 JSON Schema 파일입니다. Codex가 tool output을 이에 맞춰 검증합니다. |
| `--color` | `always \| never \| auto` | `auto` | stdout의 ANSI color 출력 방식을 정합니다. |
| `--json`, `--experimental-json` | `boolean` | `false` | formatted text 대신 newline-delimited JSON event를 출력합니다. |
| `--output-last-message`, `-o` | `path` | 없음 | assistant의 최종 메시지를 파일에 씁니다. downstream script에 유용합니다. |
| `-c`, `--config` | `key=value` | 없음 | 비대화형 실행의 inline 설정 override입니다. 반복 가능합니다. |

## stdin에서 프롬프트 읽기

긴 프롬프트를 파일이나 다른 명령에서 넘길 때 `PROMPT` 자리에 `-`를 사용합니다.

```bash
cat prompt.md | codex exec -
```

예: 변경된 파일 목록을 포함해 리뷰 요청하기

```bash
{
  echo "다음 변경사항을 코드 리뷰해줘."
  git diff --stat
} | codex exec -
```

## CI에서 결과 캡처

JSONL 이벤트와 최종 자연어 요약을 함께 남기는 패턴입니다.

```bash
codex exec \
  --json \
  --output-last-message codex-summary.md \
  --sandbox workspace-write \
  --ask-for-approval never \
  "lint와 test 실패를 수정해줘"
```

`--json`은 진행 상태를 기계가 읽기 쉽게 만들고, `--output-last-message`는 사람이 읽을 최종 요약을 파일로 남깁니다.

## 구조화된 최종 응답 받기

`--output-schema`는 Codex의 최종 응답 형태를 JSON Schema로 제한할 때 사용합니다.

```bash
codex exec \
  --output-schema schema/review-result.schema.json \
  "이 변경사항을 리뷰하고 JSON 형식으로 결과를 반환해줘"
```

자동화 파이프라인에서 후속 처리가 필요한 경우 유용합니다.

## `codex exec resume`

비대화형 세션도 이전 실행을 이어갈 수 있습니다.

기본 형태:

```bash
codex exec resume --last "방금 찾은 race condition을 수정해줘"
```

특정 세션 ID:

```bash
codex exec resume 7f9f9a2e-1b3c-4c7a-9b0e-.... "계획을 구현해줘"
```

옵션:

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `SESSION_ID` | `uuid` | 없음 | 지정한 세션을 재개합니다. 생략하고 `--last`를 쓸 수 있습니다. |
| `--last` | `boolean` | `false` | 현재 작업 디렉터리의 가장 최근 대화를 재개합니다. |
| `--all` | `boolean` | `false` | 현재 작업 디렉터리 밖의 세션도 포함합니다. |
| `--image`, `-i` | `path[,path...]` | 없음 | follow-up prompt에 이미지를 첨부합니다. 쉼표 구분 또는 반복 플래그를 지원합니다. |
| `PROMPT` | `string \| -` | 없음 | 재개 직후 보낼 선택적 follow-up 지시입니다. `-`로 stdin을 읽을 수 있습니다. |

예:

```bash
codex exec resume --last --image failure.png "이 실패 화면을 기준으로 원인을 수정해줘"
codex exec resume --last --all "다음 단계로 진행해줘"
```

## `codex cloud`

`codex cloud`는 터미널에서 Codex Cloud task를 조회하거나 실행합니다. 기본 명령은 interactive picker를 엽니다.

```bash
codex cloud
```

인증은 메인 CLI와 같은 credential을 사용합니다. 작업 제출이 실패하면 non-zero exit code로 종료되므로 CI나 스크립트에 연결할 수 있습니다.

## `codex cloud exec`

Cloud task를 직접 제출합니다.

```bash
codex cloud exec --env ENV_ID "Summarize open bugs"
```

옵션:

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `QUERY` | `string` | 없음 | task prompt입니다. 생략하면 Codex가 interactive하게 세부 내용을 물어봅니다. |
| `--env` | `ENV_ID` | 필수 | 대상 Codex Cloud environment identifier입니다. `codex cloud`로 목록을 확인합니다. |
| `--attempts` | `1-4` | `1` | Cloud가 실행할 assistant attempt 수입니다. best-of-N 용도입니다. |

best-of-3 실행:

```bash
codex cloud exec --env ENV_ID --attempts 3 "열린 버그를 요약하고 우선순위를 제안해줘"
```

## `codex cloud list`

최근 cloud task를 나열합니다. 필터링과 pagination을 지원합니다.

```bash
codex cloud list --env ENV_ID
```

옵션:

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--env` | `ENV_ID` | 없음 | environment identifier로 task를 필터링합니다. |
| `--limit` | `1-20` | `20` | 반환할 task 최대 개수입니다. |
| `--cursor` | `string` | 없음 | 이전 요청에서 받은 pagination cursor입니다. |
| `--json` | `boolean` | `false` | plain text 대신 machine-readable JSON을 출력합니다. |

자동화 예:

```bash
codex cloud list --env ENV_ID --limit 10 --json
```

JSON payload에는 `tasks` 배열과 선택적 `cursor` 값이 들어갑니다. 각 task에는 보통 다음 정보가 포함됩니다.

- `id`
- `url`
- `title`
- `status`
- `updated_at`
- `environment_id`
- `environment_label`
- `summary`
- `is_review`
- `attempt_total`

plain text 출력은 task URL과 상태 정보를 보여줍니다.

## `codex apply`

Codex Cloud task의 가장 최근 diff를 로컬 저장소에 적용합니다. 인증되어 있어야 하며, 해당 task에 접근 권한이 필요합니다. 별칭은 `codex a`입니다.

```bash
codex apply TASK_ID
codex a TASK_ID
```

옵션:

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `TASK_ID` | `string` | 로컬 working tree에 적용할 diff를 가진 Codex Cloud task identifier입니다. |

Codex는 patch가 적용된 파일을 출력합니다. `git apply`가 충돌 등으로 실패하면 non-zero exit code로 종료합니다.

