# 서버, MCP, 플러그인

이 문서는 Codex CLI의 서버/프로토콜 관련 명령을 정리합니다. 대부분 로컬 개발, 디버깅, 도구 연동, 플러그인 관리에 쓰이며 일부는 Experimental입니다.

## `codex app-server`

로컬 Codex app server를 실행합니다. 주로 개발과 디버깅용이며, 동작이 예고 없이 바뀔 수 있습니다.

```bash
codex app-server
```

### 옵션

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--listen` | `stdio:// \| ws://IP:PORT \| unix:// \| unix://PATH \| off` | `stdio://` | transport listener URL입니다. |
| `--ws-auth` | `capability-token \| signed-bearer-token` | 없음 | WebSocket client 인증 모드입니다. 생략하면 WebSocket auth가 비활성화됩니다. |
| `--ws-token-file` | `absolute path` | 없음 | shared capability token이 들어 있는 파일입니다. `--ws-auth capability-token`에 필요합니다. |
| `--ws-shared-secret-file` | `absolute path` | 없음 | signed JWT bearer token 검증에 사용할 HMAC shared secret 파일입니다. |
| `--ws-issuer` | `string` | 없음 | signed bearer token의 기대 `iss` claim입니다. |
| `--ws-audience` | `string` | 없음 | signed bearer token의 기대 `aud` claim입니다. |
| `--ws-max-clock-skew-seconds` | `number` | `30` | signed bearer token의 `exp`, `nbf` claim 검증 시 허용할 clock skew입니다. |
| `--analytics-default-enabled` | `boolean` | `false` | first-party app-server client의 analytics 기본값을 활성화합니다. 사용자가 config에서 opt out할 수 있습니다. |

### Transport 예

기본 JSONL-over-stdio:

```bash
codex app-server --listen stdio://
```

TCP WebSocket endpoint:

```bash
codex app-server --listen ws://127.0.0.1:4500
```

기본 Unix socket:

```bash
codex app-server --listen unix://
```

커스텀 Unix socket:

```bash
codex app-server --listen unix:///absolute/path/codex.sock
```

local transport 비활성화:

```bash
codex app-server --listen off
```

서버는 `ws://` listen URL을 받습니다. 클라이언트가 `wss://`로 접속해야 하는 환경에서는 TLS termination 또는 secure proxy를 앞에 둡니다.

schema를 생성하는 client binding 작업에서 gated field와 method를 포함해야 한다면 `--experimental`을 추가합니다.

## WebSocket 인증

Capability token 방식:

```bash
TOKEN_FILE="$HOME/.codex/app-server-token"
openssl rand -base64 32 > "$TOKEN_FILE"
chmod 600 "$TOKEN_FILE"

codex app-server \
  --listen ws://0.0.0.0:4500 \
  --ws-auth capability-token \
  --ws-token-file "$TOKEN_FILE"
```

Signed bearer token 방식:

```bash
codex app-server \
  --listen ws://0.0.0.0:4500 \
  --ws-auth signed-bearer-token \
  --ws-shared-secret-file /absolute/path/to/secret \
  --ws-issuer codex-client \
  --ws-audience codex-app-server
```

비로컬 listener를 열 때 인증을 생략하면 startup 중 경고가 나옵니다. 원격 접속에는 인증과 TLS를 함께 고려해야 합니다.

## `codex remote-control`

remote-control 지원이 켜진 app-server daemon이 실행 중인지 보장합니다.

```bash
codex remote-control
```

managed remote-control client와 SSH remote workflow가 사용하는 명령입니다. 로컬 protocol client를 직접 만들고 있다면 `codex app-server --listen ...`의 대체재가 아닙니다.

## Debug 명령

### `codex debug app-server send-message-v2`

내장 app-server test client를 통해 V2 thread/turn flow로 메시지 하나를 보냅니다.

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `USER_MESSAGE` | `string` | app-server에 보낼 메시지 텍스트입니다. |

예:

```bash
codex debug app-server send-message-v2 "hello from test client"
```

이 debug flow는 `experimentalApi: true`로 초기화하고, thread를 시작하고, turn을 보낸 뒤 server notification을 stream합니다. app-server protocol 동작을 로컬에서 재현하고 관찰할 때 씁니다.

### `codex debug models`

Codex가 보는 raw model catalog를 JSON으로 출력합니다.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `--bundled` | `boolean` | `false` | remote models endpoint를 refresh하지 않고 현재 Codex binary에 포함된 catalog만 출력합니다. |

예:

```bash
codex debug models
codex debug models --bundled
```

## `codex mcp`

Model Context Protocol 서버 설정을 `~/.codex/config.toml`에 저장하고 관리합니다.

```bash
codex mcp list
```

### 하위 명령

| 명령 | 옵션/형태 | 설명 |
| --- | --- | --- |
| `list` | `--json` | 설정된 MCP 서버 목록을 보여줍니다. `--json`으로 기계가 읽기 쉬운 출력을 받을 수 있습니다. |
| `get <name>` | `--json` | 특정 서버 설정을 보여줍니다. `--json`은 raw config entry를 출력합니다. |
| `add <name>` | `-- <command...> \| --url <value>` | stdio launcher command 또는 streamable HTTP URL로 서버를 등록합니다. |
| `remove <name>` | 없음 | 저장된 MCP 서버 정의를 삭제합니다. |
| `login <name>` | `--scopes scope1,scope2` | streamable HTTP 서버의 OAuth login을 시작합니다. |
| `logout <name>` | 없음 | streamable HTTP 서버의 저장된 OAuth credential을 제거합니다. |

### `codex mcp add` 옵션

| 옵션 | 타입 | 설명 |
| --- | --- | --- |
| `COMMAND...` | stdio transport | MCP 서버를 실행할 executable과 arguments입니다. `--` 뒤에 제공합니다. |
| `--env KEY=VALUE` | repeatable | stdio 서버 실행 시 적용할 환경 변수입니다. |
| `--url` | `https://...` | stdio 대신 streamable HTTP 서버를 등록합니다. `COMMAND...`와 함께 쓸 수 없습니다. |
| `--bearer-token-env-var` | `ENV_VAR` | streamable HTTP 서버 연결 시 bearer token으로 보낼 환경 변수 이름입니다. |

stdio 서버 추가:

```bash
codex mcp add docs -- npx -y @example/docs-mcp
```

환경 변수와 함께 stdio 서버 추가:

```bash
codex mcp add db --env DATABASE_URL=postgres://localhost/app -- node ./mcp-db-server.js
```

streamable HTTP 서버 추가:

```bash
codex mcp add search --url https://mcp.example.com
```

bearer token 환경 변수 사용:

```bash
export MCP_TOKEN=...
codex mcp add search --url https://mcp.example.com --bearer-token-env-var MCP_TOKEN
```

OAuth `login`과 `logout`은 streamable HTTP 서버에서만 동작하며, 해당 서버가 OAuth를 지원해야 합니다.

## `codex mcp-server`

Codex 자체를 stdio 기반 MCP 서버로 실행합니다. 다른 agent나 도구가 Codex를 MCP server로 소비해야 할 때 사용합니다.

```bash
codex mcp-server
```

이 명령은 전역 configuration override를 상속하며, downstream client가 연결을 닫으면 종료됩니다.

## `codex plugin marketplace`

Codex가 탐색하고 설치할 수 있는 plugin marketplace source를 관리합니다.

### 하위 명령

| 명령 | 옵션/형태 | 설명 |
| --- | --- | --- |
| `add <source>` | `[--ref REF] [--sparse PATH]` | GitHub shorthand, Git URL, SSH URL, local marketplace root에서 marketplace를 설치합니다. |
| `upgrade [marketplace-name]` | 없음 | 지정한 Git marketplace 하나를 refresh합니다. 이름을 생략하면 설정된 모든 Git marketplace를 refresh합니다. |
| `remove <marketplace-name>` | 없음 | 설정된 plugin marketplace를 제거합니다. |

### Source 형식

`codex plugin marketplace add`는 다음 source를 받을 수 있습니다.

- GitHub shorthand: `owner/repo`
- ref가 포함된 GitHub shorthand: `owner/repo@ref`
- HTTP/HTTPS Git URL
- SSH Git URL
- local marketplace root directory

Git ref 고정:

```bash
codex plugin marketplace add owner/repo --ref main
```

sparse checkout 사용:

```bash
codex plugin marketplace add owner/repo --sparse plugins --sparse skills
```

`--sparse`는 Git source에서만 지원되며 반복할 수 있습니다.

