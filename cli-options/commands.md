# 명령 개요와 주요 워크플로

이 문서는 Codex CLI의 주요 명령을 한눈에 볼 수 있도록 정리합니다. 세부 옵션이 많은 `exec`, `cloud`, `mcp`, `sandbox` 계열은 별도 문서에서 더 자세히 다룹니다.

## 명령 성숙도

원문 문서는 각 명령에 Stable, Experimental 같은 maturity label을 붙입니다.

- Stable: 일반 사용을 기대할 수 있는 명령
- Experimental: 동작이나 옵션이 바뀔 수 있는 명령

자동화나 팀 문서에 넣을 때는 Stable 명령을 우선 사용하고, Experimental 명령은 버전 변화에 대비하는 것이 좋습니다.

## 명령 목록

| 명령 | 성숙도 | 설명 |
| --- | --- | --- |
| `codex` | Stable | 터미널 UI를 실행합니다. 공통 플래그, 프롬프트, 이미지 첨부를 받을 수 있습니다. |
| `codex app-server` | Experimental | 로컬 개발/디버깅용 Codex app server를 stdio, WebSocket, Unix socket으로 실행합니다. |
| `codex remote-control` | Experimental | remote-control 지원이 켜진 로컬 app-server daemon을 보장합니다. |
| `codex app` | Stable | macOS 또는 Windows에서 Codex Desktop 앱을 실행합니다. |
| `codex debug app-server send-message-v2` | Experimental | 내장 테스트 클라이언트로 app-server에 V2 메시지 하나를 보냅니다. |
| `codex debug models` | Experimental | Codex가 보는 raw model catalog를 출력합니다. |
| `codex apply` | Stable | Codex Cloud task의 최신 diff를 로컬 working tree에 적용합니다. 별칭: `codex a` |
| `codex cloud` | Experimental | 터미널에서 Codex Cloud task를 조회하거나 실행합니다. 별칭: `codex cloud-tasks` |
| `codex completion` | Stable | Bash, Zsh, Fish, PowerShell 등의 shell completion script를 생성합니다. |
| `codex features` | Stable | feature flag를 나열하고 `config.toml`에 enable/disable 상태를 저장합니다. |
| `codex exec` | Stable | Codex를 비대화형으로 실행합니다. 별칭: `codex e` |
| `codex execpolicy` | Experimental | execpolicy rule 파일을 평가해 명령이 허용/확인/차단될지 확인합니다. |
| `codex login` | Stable | ChatGPT OAuth, device auth, API key, access token으로 인증합니다. |
| `codex logout` | Stable | 저장된 인증 정보를 제거합니다. |
| `codex mcp` | Experimental | MCP 서버를 list/add/remove/login/logout으로 관리합니다. |
| `codex plugin marketplace` | Experimental | Git 또는 로컬 source에서 plugin marketplace를 추가/업그레이드/제거합니다. |
| `codex mcp-server` | Experimental | Codex 자체를 stdio 기반 MCP 서버로 실행합니다. |
| `codex resume` | Stable | 이전 interactive session을 이어서 엽니다. |
| `codex fork` | Stable | 이전 interactive session을 새 thread로 fork합니다. |
| `codex sandbox` | Experimental | Codex가 내부적으로 쓰는 sandbox 정책으로 임의 명령을 실행합니다. |
| `codex update` | Stable | 설치된 release가 self-update를 지원하면 Codex CLI 업데이트를 확인하고 적용합니다. |

## `codex` interactive

하위 명령 없이 `codex`를 실행하면 interactive TUI가 열립니다.

```bash
codex
```

프롬프트와 함께 시작:

```bash
codex "이 저장소의 테스트 전략을 설명해줘"
```

이미지 첨부:

```bash
codex -i screenshot.png "이 화면에서 접근성 문제를 찾아줘"
```

웹 검색은 기본적으로 cached mode입니다. 최신 웹 정보가 필요하면 `--search`를 사용합니다.

```bash
codex --search "현재 React 최신 stable 버전을 확인해줘"
```

로컬 개발 작업에서 마찰을 줄이는 권장 조합:

```bash
codex --sandbox workspace-write --ask-for-approval on-request
```

원격 app server에 연결:

```bash
codex app-server --listen ws://127.0.0.1:4500
codex --remote ws://127.0.0.1:4500
```

WebSocket 인증이 필요한 서버라면:

```bash
export CODEX_REMOTE_TOKEN="$(cat "$HOME/.codex/app-server-token")"
codex --remote wss://remote-host:4500 --remote-auth-token-env CODEX_REMOTE_TOKEN
```

## `codex resume`

이전 interactive session을 이어서 엽니다.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `SESSION_ID` | `uuid` | 없음 | 지정한 세션을 재개합니다. 생략하고 `--last`를 쓰면 가장 최근 세션을 엽니다. |
| `--last` | `boolean` | `false` | picker를 건너뛰고 현재 작업 디렉터리의 가장 최근 대화를 재개합니다. |
| `--all` | `boolean` | `false` | 현재 작업 디렉터리 밖의 세션도 포함합니다. |

예:

```bash
codex resume
codex resume --last
codex resume --last --all
codex resume 7f9f9a2e-1b3c-4c7a-9b0e-....
```

`codex resume`은 `codex`와 같은 공통 플래그를 받을 수 있으므로, 모델이나 sandbox를 바꿔 이어갈 수 있습니다.

```bash
codex resume --last --model gpt-5.5 --sandbox workspace-write
```

## `codex fork`

이전 interactive session을 새 thread로 fork합니다. 원본 transcript는 보존하면서 다른 방향으로 이어가고 싶을 때 사용합니다.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `SESSION_ID` | `uuid` | 없음 | 지정한 세션을 fork합니다. 생략하고 `--last`를 쓸 수 있습니다. |
| `--last` | `boolean` | `false` | picker를 건너뛰고 가장 최근 세션을 자동으로 fork합니다. |
| `--all` | `boolean` | `false` | 현재 작업 디렉터리 밖의 세션도 picker에 표시합니다. |

예:

```bash
codex fork
codex fork --last
codex fork --all
codex fork 7f9f9a2e-1b3c-4c7a-9b0e-....
```

## `codex login`

Codex CLI 인증을 설정합니다. 플래그 없이 실행하면 ChatGPT OAuth flow를 위해 브라우저를 엽니다.

```bash
codex login
```

| 옵션 | 설명 |
| --- | --- |
| `--with-api-key` | stdin에서 API key를 읽습니다. |
| `--with-access-token` | stdin에서 access token을 읽습니다. |
| `--device-auth` | 브라우저 창을 여는 대신 OAuth device code flow를 사용합니다. |
| `codex login status` | 현재 인증 모드를 출력합니다. 로그인되어 있으면 exit code 0으로 종료합니다. |

예:

```bash
printenv OPENAI_API_KEY | codex login --with-api-key
printenv CODEX_ACCESS_TOKEN | codex login --with-access-token
codex login --device-auth
codex login status
```

자동화 스크립트에서는 `codex login status`의 exit code를 확인해 인증 여부를 빠르게 검사할 수 있습니다.

## `codex logout`

저장된 API key와 ChatGPT 인증 정보를 제거합니다. 별도 플래그는 없습니다.

```bash
codex logout
```

## `codex completion`

shell completion script를 생성합니다. 출력은 stdout으로 나옵니다.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `SHELL` | `bash \| zsh \| fish \| power-shell \| elvish` | `bash` | completion을 생성할 shell입니다. |

예:

```bash
codex completion zsh
```

파일로 저장하는 예:

```bash
codex completion zsh > "${fpath[1]}/_codex"
```

`eval`로 바로 로드하는 방식도 가능합니다.

```bash
eval "$(codex completion zsh)"
```

## `codex features`

feature flag를 조회하고, `~/.codex/config.toml`에 enable/disable 상태를 저장합니다.

| 하위 명령 | 설명 |
| --- | --- |
| `codex features list` | 알려진 feature flag, maturity stage, effective state를 보여줍니다. |
| `codex features enable <feature>` | feature flag를 `config.toml`에 영구적으로 활성화합니다. |
| `codex features disable <feature>` | feature flag를 `config.toml`에 영구적으로 비활성화합니다. |

예:

```bash
codex features list
codex features enable unified_exec
codex features disable shell_snapshot
```

`--profile`을 함께 사용하면 root 설정이 아니라 활성 profile에 저장됩니다.

```bash
codex --profile work features enable unified_exec
```

## `codex app`

터미널에서 Codex Desktop 앱을 실행합니다. macOS에서는 workspace path를 열 수 있고, Windows에서는 열어야 할 path를 출력합니다.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `PATH` | `path` | `.` | Codex Desktop workspace path입니다. |
| `--download-url` | `url` | 없음 | 설치 중 사용할 Codex Desktop installer URL을 고급 override합니다. |

예:

```bash
codex app
codex app /path/to/workspace
```

앱이 설치되어 있으면 앱을 열고, 없으면 installer를 시작합니다.

## `codex update`

설치된 Codex CLI release가 self-update를 지원하면 업데이트를 확인하고 적용합니다.

```bash
codex update
```

debug build에서는 release build를 설치하라는 메시지만 출력될 수 있습니다.

