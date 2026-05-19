# Codex CLI Slash Commands

Slash command는 Codex CLI 안에서 키보드 중심으로 세션을 제어하는 빠른 명령입니다. composer에서 `/`를 입력하면 slash popup이 열리고, 명령을 선택해 모델 변경, 권한 조정, diff 확인, 긴 대화 요약, 리뷰 실행 같은 작업을 터미널을 떠나지 않고 수행할 수 있습니다.

## 문서 구성

- [명령 목록](command-catalog.md): built-in slash command 전체 목록과 언제 쓰는지
- [세션 제어와 대화 흐름](session-control.md): `/model`, `/fast`, `/personality`, `/plan`, `/goal`, `/clear`, `/compact`, `/copy`, `/raw`, `/status`
- [개발 워크플로 명령](development-workflows.md): `/diff`, `/review`, `/mention`, `/init`, `/mcp`, `/apps`, `/plugins`, `/hooks`, `/agent`, `/side`
- [환경 설정과 종료](configuration-and-exit.md): `/permissions`, `/ide`, `/vim`, `/keymap`, `/statusline`, `/title`, `/theme`, `/experimental`, `/approve`, `/memories`, `/feedback`, `/logout`, `/quit`, `/exit`

## 기본 사용법

1. Codex TUI composer에서 `/`를 입력합니다.
2. 명령 이름을 일부 입력해 목록을 필터링합니다.
3. 명령을 선택하고 `Enter`를 누릅니다.

작업이 이미 실행 중이면 slash command를 입력한 뒤 `Tab`을 눌러 다음 턴에 실행되도록 큐에 넣을 수 있습니다. 큐에 들어간 slash command는 현재 턴이 끝난 뒤 파싱되며, 메뉴나 오류 메시지도 그때 표시됩니다. 큐에 넣기 전에도 slash completion은 동작합니다.

예:

```text
/review
```

```text
/model
```

```text
/plan 마이그레이션 계획을 먼저 제안해줘
```

## 가장 자주 쓰는 명령

| 목적 | 명령 |
| --- | --- |
| 모델 변경 | `/model` |
| 권한과 승인 정책 변경 | `/permissions` |
| 현재 설정 확인 | `/status` |
| 작업 전 계획 모드로 전환 | `/plan` |
| 현재 변경사항 확인 | `/diff` |
| 작업 트리 리뷰 | `/review` |
| 긴 대화 요약으로 context 절약 | `/compact` |
| 최근 답변 복사 | `/copy` 또는 `Ctrl+O` |
| 새 대화 시작 | `/new` |
| 이전 대화 재개 | `/resume` |
| 현재 대화 fork | `/fork` |
| CLI 종료 | `/quit` 또는 `/exit` |

## 실행 중인 작업에 명령 큐잉하기

Codex가 아직 응답 중이거나 명령을 실행 중일 때도 다음 작업을 미리 예약할 수 있습니다.

```text
/diff
```

입력 후 `Tab`을 누르면 현재 턴이 끝난 뒤 `/diff`가 실행됩니다. 예를 들어 Codex가 구현을 마친 직후 자동으로 diff를 보고 싶을 때 유용합니다.

큐에는 slash command뿐 아니라 일반 프롬프트나 `!` shell command도 넣을 수 있습니다.

## `/quit`와 `/exit`

`/quit`와 `/exit`는 둘 다 CLI를 종료합니다. 중요한 변경사항은 종료 전에 저장하거나 커밋해야 합니다.

```text
/quit
```

```text
/exit
```

