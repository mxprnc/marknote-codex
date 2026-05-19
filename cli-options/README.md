# Codex CLI 명령줄 옵션

이 문서는 Codex 개발자 문서의 Command line options 내용을 한국어로 번역하고, 개발자가 실제로 찾기 쉽도록 재구성한 참조 문서입니다.

Codex CLI 옵션은 크게 세 층으로 이해하면 편합니다.

1. 공통 플래그: `codex`, `codex exec`, `codex resume` 등 여러 명령에서 함께 쓰는 실행 환경 옵션
2. 명령별 옵션: `exec`, `cloud`, `mcp`, `app-server`처럼 특정 하위 명령에만 적용되는 옵션
3. 안전 관련 옵션: sandbox, approval, remote auth, execpolicy처럼 실행 권한과 보안 경계를 정하는 옵션

## 문서 구성

- [공통 플래그와 기본 실행](global-flags.md): `--model`, `--sandbox`, `--cd`, `--search`, `--config` 등 대부분의 실행에서 쓰는 옵션
- [명령 개요와 주요 워크플로](commands.md): `codex`, `resume`, `fork`, `login`, `completion`, `features`, `app`, `update` 등
- [비대화형 실행과 Cloud](exec-and-cloud.md): `codex exec`, `exec resume`, `codex cloud`, `codex apply`
- [서버, MCP, 플러그인](servers-mcp-and-plugins.md): `app-server`, `remote-control`, `mcp`, `mcp-server`, plugin marketplace, debug 명령
- [샌드박스, execpolicy, 안전 팁](sandbox-and-safety.md): macOS/Linux/Windows sandbox helper, execpolicy, 위험한 옵션 조합

## 빠른 예제

대화형 TUI 실행:

```bash
codex
```

모델과 작업 디렉터리를 지정해 실행:

```bash
codex --model gpt-5.5 --cd apps/frontend "이 앱의 라우팅 구조를 설명해줘"
```

로컬 작업을 낮은 마찰로 진행:

```bash
codex --sandbox workspace-write --ask-for-approval on-request
```

비대화형 자동화 실행:

```bash
codex exec --sandbox workspace-write --ask-for-approval never "테스트 실패를 수정해줘"
```

JSONL 이벤트와 최종 메시지 파일을 함께 남기기:

```bash
codex exec --json --output-last-message result.md "릴리즈 노트를 작성해줘"
```

Codex Cloud 작업 실행:

```bash
codex cloud exec --env ENV_ID --attempts 3 "열린 버그를 요약해줘"
```

MCP 서버 추가:

```bash
codex mcp add docs -- npx -y @example/docs-mcp
```

## 옵션 우선순위

Codex CLI는 대부분의 기본값을 `~/.codex/config.toml`에서 읽습니다. 명령줄에서 넘긴 `-c key=value` 값은 해당 실행에 한해 설정 파일보다 우선합니다.

예:

```bash
codex -c model=\"gpt-5.5\" -c web_search=\"live\" "최신 릴리즈 정보를 확인해줘"
```

값은 가능한 경우 JSON으로 파싱되고, 그렇지 않으면 문자열 리터럴로 사용됩니다. 즉 boolean이나 number는 그대로 넘길 수 있고, 문자열은 필요하면 따옴표를 이스케이프해야 합니다.

```bash
codex -c features.unified_exec=true
codex -c model_provider=\"oss\"
```

## 전역 플래그 위치

하위 명령을 실행할 때는 전역 플래그를 하위 명령 뒤에 두는 것이 안전합니다.

```bash
codex exec --model gpt-5.5 "CI 실패를 고쳐줘"
```

원문 기준으로 전역 플래그는 여러 하위 명령에 전파되지만, 명령별로 받는 옵션이 다르므로 각 명령 문서의 예제 형태를 따르는 편이 좋습니다.

