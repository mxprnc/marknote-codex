# Slash Command 목록

Codex CLI에는 다양한 built-in slash command가 포함되어 있습니다. composer에서 `/`를 입력한 뒤 명령 이름을 일부 입력하면 목록을 필터링할 수 있습니다.

## 전체 명령 요약

| 명령 | 목적 | 언제 쓰나 |
| --- | --- | --- |
| `/permissions` | Codex가 먼저 묻지 않고 할 수 있는 일을 설정 | Auto와 Read Only 사이를 전환하거나 승인 요구사항을 조정할 때 |
| `/ide` | 열린 파일, 현재 선택 영역 등 IDE context 포함 | IDE에서 열어둔 맥락을 다음 프롬프트에 반영하고 싶을 때 |
| `/keymap` | TUI 키보드 단축키 remap | shortcut binding을 보고 `config.toml`에 저장하고 싶을 때 |
| `/vim` | composer Vim mode 토글 | Vim normal/insert 방식과 기본 composer 편집 방식을 전환할 때 |
| `/sandbox-add-read-dir` | sandbox read access 디렉터리 추가 | Windows에서 현재 readable root 밖의 절대 경로를 읽어야 할 때 |
| `/agent` | active agent thread 전환 | 생성된 subagent thread를 확인하거나 이어서 작업할 때 |
| `/apps` | app(connector)을 탐색하고 프롬프트에 삽입 | `$app-slug` 형태로 app을 첨부해 사용 요청을 할 때 |
| `/plugins` | 설치/탐색 가능한 plugin 보기 | plugin 도구를 확인하거나 설치/활성 상태를 관리할 때 |
| `/hooks` | lifecycle hook 확인 | hook 설정을 보고 신뢰/비활성/재활성 처리할 때 |
| `/clear` | 터미널을 지우고 새 chat 시작 | UI와 대화 맥락을 함께 초기화하고 싶을 때 |
| `/compact` | visible conversation 요약 | 긴 세션 후 context window를 아끼고 핵심 맥락만 남길 때 |
| `/copy` | 최근 완료된 Codex output 복사 | 직접 드래그하지 않고 최신 응답이나 계획을 복사할 때 |
| `/diff` | Git diff와 untracked file 표시 | 커밋 전 Codex 수정사항을 검토할 때 |
| `/exit` | CLI 종료 | `/quit`의 다른 이름 |
| `/experimental` | experimental feature 토글 | Apps, Smart Approvals 같은 optional feature를 켤 때 |
| `/approve` | auto review denial 1회 재시도 승인 | 자동 리뷰가 거부한 최근 action을 한 번 재시도할 때 |
| `/memories` | memory 사용과 생성 설정 | memory injection/generation을 TUI에서 켜거나 끌 때 |
| `/skills` | skill 탐색과 사용 | 작업에 맞는 local skill 지침을 적용하고 싶을 때 |
| `/feedback` | Codex maintainers에게 로그 전송 | 문제 보고나 diagnostics 공유가 필요할 때 |
| `/init` | 현재 디렉터리에 `AGENTS.md` scaffold 생성 | 저장소 또는 하위 디렉터리의 지속 지침을 만들 때 |
| `/logout` | Codex 로그아웃 | 공유 머신에서 local credential을 지울 때 |
| `/mcp` | 설정된 MCP tools 목록 표시 | 현재 세션에서 Codex가 호출 가능한 외부 도구를 확인할 때 |
| `/mention` | 파일을 대화에 첨부 | 다음 턴에서 특정 파일/폴더를 직접 참조하게 할 때 |
| `/model` | active model과 reasoning 선택 | 작업 전 모델을 바꾸거나 깊은 reasoning 모델로 전환할 때 |
| `/fast` | Fast service tier 토글 | 현재 모델의 Fast tier를 켜고 끄거나 상태를 확인할 때 |
| `/plan` | plan mode로 전환 | 구현 전에 실행 계획을 먼저 받으려 할 때 |
| `/goal` | experimental task goal 설정/보기/중지/재개/삭제 | 큰 작업 중 지속 목표를 thread에 붙이고 싶을 때 |
| `/personality` | 응답 커뮤니케이션 스타일 선택 | 더 간결하게, 설명적으로, 협업적으로 응답 방식을 조정할 때 |
| `/ps` | background terminal과 최근 출력 확인 | long-running command 상태를 main transcript 밖에서 확인할 때 |
| `/stop` | 모든 background terminal 중지 | 현재 세션이 시작한 background terminal 작업을 취소할 때 |
| `/fork` | 현재 대화를 새 thread로 fork | 현재 transcript를 보존하며 다른 접근을 탐색할 때 |
| `/side` | 임시 side conversation 시작 | main thread를 방해하지 않고 짧은 follow-up을 물어볼 때 |
| `/raw` | raw scrollback mode 토글 | 긴 출력에서 selection/copy를 더 직접적으로 하고 싶을 때 |
| `/resume` | 저장된 대화 재개 | 이전 CLI 세션을 이어서 작업할 때 |
| `/new` | 같은 CLI session 안에서 새 conversation 시작 | 터미널은 유지하고 chat context만 새로 시작할 때 |
| `/quit` | CLI 종료 | 세션을 즉시 떠날 때 |
| `/review` | working tree 리뷰 요청 | Codex 작업 후 또는 커밋 전 로컬 변경을 검토할 때 |
| `/status` | 세션 설정과 token 사용량 표시 | active model, approval, writable root, context capacity를 확인할 때 |
| `/debug-config` | config layer와 requirements 진단 | 설정 precedence와 policy requirements 문제를 디버깅할 때 |
| `/statusline` | TUI footer item 설정 | model/context/limits/git/tokens/session 표시를 고르고 저장할 때 |
| `/title` | terminal window/tab title item 설정 | project, status, branch, model, task progress 등을 title에 표시할 때 |
| `/theme` | syntax highlighting theme 선택 | terminal syntax theme를 preview하고 저장할 때 |

## 중복과 별칭

원문 목록에는 `/hooks`가 두 번 등장합니다. 실제 의미는 lifecycle hook 설정을 보고 관리하는 하나의 명령으로 이해하면 됩니다.

`/quit`와 `/exit`는 같은 종료 동작입니다.

`/stop`은 background terminal을 멈추는 명령이고, `/clean`은 여전히 `/stop`의 alias로 사용할 수 있습니다.

## 사용 가능 조건이 있는 명령

일부 명령은 환경이나 feature flag에 따라 보이지 않을 수 있습니다.

| 명령 | 조건 |
| --- | --- |
| `/fast` | 현재 모델 catalog가 Fast tier를 제공해야 합니다. |
| `/personality` | active model이 personality-specific instruction을 지원해야 합니다. |
| `/goal` | `features.goals`가 켜져 있어야 합니다. |
| `/sandbox-add-read-dir` | Windows native CLI에서만 사용 가능합니다. |
| `/ps` | `unified_exec` 사용 시 background terminal이 표시됩니다. |
| `/side` | 다른 side conversation 안이나 review mode 중에는 사용할 수 없습니다. |
| `/plan` | task가 이미 실행 중일 때는 일시적으로 사용할 수 없습니다. |

## 개발자가 기억하면 좋은 조합

구현 직후 검토:

```text
/diff
/review
```

긴 세션 정리:

```text
/compact
/status
```

새로운 방향 탐색:

```text
/fork
```

짧은 부가 질문:

```text
/side 이 계획에서 가장 큰 리스크만 확인해줘
```

설정 문제 디버깅:

```text
/status
/debug-config
```

