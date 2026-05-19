# 원격 TUI와 Codex Cloud

Codex CLI는 로컬 터미널에서만 쓰는 도구가 아닙니다. 원격 앱 서버에 TUI를 연결하거나, Codex Cloud 작업을 터미널에서 조회하고 실행할 수 있습니다.

## 원격 TUI 개요

원격 TUI 모드는 한 머신에서 Codex app server를 실행하고, 다른 머신의 Codex 터미널 UI가 그 서버에 연결하는 방식입니다.

이 구조는 다음 상황에 유용합니다.

- 개발 환경은 원격 머신에 있고 로컬 터미널에서 조작하고 싶을 때
- 서버가 접근 가능한 저장소와 도구를 가지고 있을 때
- SSH 포트 포워딩이나 내부 네트워크로 안전하게 연결할 때

## 로컬 WebSocket 리스너로 app server 실행

먼저 앱 서버를 WebSocket 리스너와 함께 시작합니다.

```bash
codex app-server --listen ws://127.0.0.1:4500
```

이후 TUI에서 해당 endpoint로 연결합니다.

```bash
codex --remote ws://127.0.0.1:4500
```

`ws://127.0.0.1`처럼 loopback 주소를 쓰는 평문 WebSocket은 로컬 또는 SSH 포트 포워딩 워크플로에 적합합니다.

## 다른 머신에서 접근 가능하게 열기

다른 머신에서 app server에 접근해야 한다면 서버를 reachable interface에 bind하고 WebSocket 인증을 설정해야 합니다.

```bash
TOKEN_FILE="$HOME/.codex/app-server-token"
openssl rand -base64 32 > "$TOKEN_FILE"
chmod 600 "$TOKEN_FILE"
codex app-server --listen ws://0.0.0.0:4500 --ws-auth capability-token --ws-token-file "$TOKEN_FILE"
```

이 설정은 모든 인터페이스에서 4500 포트를 열기 때문에 네트워크 노출 범위를 신중히 관리해야 합니다. 비로컬 클라이언트라면 WebSocket 인증과 TLS를 함께 사용하는 것이 권장됩니다.

## 원격 주소 형식

`--remote`는 다음 형식의 주소를 받을 수 있습니다.

```bash
codex --remote ws://host:port
codex --remote wss://host:port
```

권장 기준은 다음과 같습니다.

| 연결 형태 | 권장 사용처 |
| --- | --- |
| `ws://127.0.0.1:PORT` | 로컬 또는 SSH 포트 포워딩 |
| `ws://0.0.0.0:PORT` 서버 bind | 내부 네트워크에서 접근 가능한 서버 |
| `wss://host:PORT` | 비로컬 클라이언트, TLS가 필요한 연결 |

## WebSocket 인증 방식

Codex는 다음 WebSocket 인증 모드를 지원합니다.

### Capability token

서버 실행 시:

```bash
codex app-server \
  --listen ws://0.0.0.0:4500 \
  --ws-auth capability-token \
  --ws-token-file /absolute/path/to/token
```

또는 token 파일 대신 SHA-256 해시를 사용할 수 있습니다.

```bash
codex app-server \
  --listen ws://0.0.0.0:4500 \
  --ws-auth capability-token \
  --ws-token-sha256 HEX
```

### Signed bearer token

공유 secret 파일 기반으로 signed bearer token 인증을 사용할 수 있습니다.

```bash
codex app-server \
  --listen ws://0.0.0.0:4500 \
  --ws-auth signed-bearer-token \
  --ws-shared-secret-file /absolute/path/to/secret
```

필요하면 다음 옵션도 함께 설정합니다.

- `--ws-issuer`
- `--ws-audience`
- `--ws-max-clock-skew-seconds`

## TUI에서 원격 인증 토큰 전달

TUI는 WebSocket handshake 중 다음 형태의 헤더를 보냅니다.

```http
Authorization: Bearer <token>
```

예:

```bash
export CODEX_REMOTE_TOKEN="$(cat "$TOKEN_FILE")"
codex --remote wss://remote-host:4500 --remote-auth-token-env CODEX_REMOTE_TOKEN
```

Codex는 원격 인증 토큰을 `wss://` URL 또는 loopback `ws://` URL에서만 허용합니다. 평문 네트워크 구간에 토큰을 그대로 흘리지 않기 위한 제약입니다.

## Remote connections와 remote-control

Codex 앱에서 SSH 원격 프로젝트를 다룰 때는 원문 문서의 Remote connections 가이드를 참고합니다.

관리형 원격 제어 클라이언트가 필요하면 다음 명령으로 remote-control 지원이 켜진 app server 프로세스를 시작할 수 있습니다.

```bash
codex remote-control
```

## Codex Cloud 작업

`codex cloud` 명령은 터미널에서 Codex Cloud 작업을 조회하고 실행하는 기능입니다.

인자 없이 실행하면 interactive picker가 열립니다.

```bash
codex cloud
```

이 picker에서 active 또는 finished task를 살펴보고, 필요한 변경 사항을 로컬 프로젝트에 적용할 수 있습니다.

## 터미널에서 Cloud 작업 시작하기

환경 ID를 지정해 Cloud 작업을 바로 시작할 수 있습니다.

```bash
codex cloud exec --env ENV_ID "Summarize open bugs"
```

여러 시도 중 가장 좋은 결과를 고르고 싶다면 `--attempts`를 사용합니다. 값은 1에서 4까지 지정할 수 있습니다.

```bash
codex cloud exec --env ENV_ID --attempts 3 "Summarize open bugs"
```

## Environment ID 확인

Environment ID는 Codex Cloud 설정에서 가져옵니다. 확인 방법은 두 가지입니다.

- `codex cloud`를 실행한 뒤 `Ctrl+O`로 환경 선택
- 웹 dashboard에서 정확한 값 확인

인증은 기존 CLI 로그인을 따릅니다. 작업 제출에 실패하면 명령은 non-zero exit code로 종료되므로 스크립트나 CI에도 연결할 수 있습니다.

