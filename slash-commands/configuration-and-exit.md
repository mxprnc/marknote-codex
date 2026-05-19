# 환경 설정과 종료

이 문서는 permissions, IDE context, keymap, theme, status line, experimental feature, memory, feedback, logout, exit처럼 Codex CLI 환경을 조정하거나 세션을 종료하는 slash command를 정리합니다.

## `/permissions`: 권한과 승인 정책 변경

Codex가 먼저 묻지 않고 수행할 수 있는 작업 범위를 바꿉니다.

```text
/permissions
```

사용 방법:

1. `/permissions`를 입력하고 `Enter`를 누릅니다.
2. 현재 comfort level에 맞는 approval preset을 선택합니다.

예:

- `Auto`: 손이 덜 가는 로컬 작업
- `Read Only`: 수정 전 분석과 검토 중심

기대 결과:

- Codex가 updated policy를 transcript에 알립니다.
- 이후 action은 다시 변경하기 전까지 새 approval mode를 따릅니다.

작업 중 권한을 느슨하게 하거나 조이는 대표 명령입니다.

## `/approve`: auto review denial 1회 승인

automatic reviewer가 최근 action을 거부했지만, 사용자가 한 번 재시도하고 싶을 때 사용합니다.

```text
/approve
```

사용 방법:

1. `/approve`를 입력합니다.
2. Codex가 보여주는 denied action을 확인합니다.
3. retry를 승인합니다.

기대 결과:

- 현재 session policy 아래에서 해당 denied action을 한 번 재시도합니다.

`/approve`는 일반 권한 변경 명령이 아니라, 최근 auto review denial에 대한 one-time retry 승인입니다.

## `/experimental`: experimental feature 토글

optional 또는 experimental feature를 CLI 안에서 켜고 끕니다.

```text
/experimental
```

사용 방법:

1. `/experimental`을 입력합니다.
2. 원하는 feature를 토글합니다. 예: Apps, Smart Approvals
3. Codex가 restart를 요청하면 재시작합니다.

기대 결과:

- feature 선택이 config에 저장됩니다.
- 필요한 경우 restart 후 적용됩니다.

## `/memories`: memory 설정

memory 사용과 생성을 TUI에서 조정합니다.

```text
/memories
```

선택 가능한 방향:

- 기존 memory 사용
- 새 memory 생성
- memory behavior 비활성화

기대 결과:

- 이후 session에 적용될 memory 관련 설정이 업데이트됩니다.

## `/skills`: skill 선택과 적용

작업에 맞는 local skill을 선택해 다음 요청이 해당 skill의 지침을 따르게 합니다.

```text
/skills
```

사용 방법:

1. `/skills`를 입력합니다.
2. 적용할 skill을 선택합니다.

기대 결과:

- 선택한 skill context가 삽입됩니다.
- 다음 request가 해당 skill instruction을 따릅니다.

예를 들어 문서, 프레젠테이션, 이미지 생성, 스프레드시트 작업처럼 전문 workflow가 필요한 경우 유용합니다.

## `/ide`: IDE context 포함

열린 파일, 현재 selection 등 사용 가능한 IDE context를 다음 프롬프트에 포함합니다.

```text
/ide
```

inline 설명을 덧붙일 수도 있습니다.

```text
/ide 현재 선택한 함수의 오류 처리를 개선해줘
```

기대 결과:

- 다음 prompt에 IDE context가 포함됩니다.
- 열린 파일이나 selection을 다시 설명하지 않아도 됩니다.

## `/vim`: composer Vim mode 토글

composer의 Vim mode를 현재 session에서 켜거나 끕니다.

```text
/vim
```

기대 결과:

- 기본 composer editing mode와 Vim normal/insert behavior 사이를 전환합니다.

새 session에서도 Vim mode를 기본값으로 쓰려면 `config.toml`에 설정합니다.

```toml
tui.vim_mode_default = true
```

## `/keymap`: TUI shortcut remap

TUI 키보드 단축키 binding을 확인, 수정, 저장합니다.

```text
/keymap
```

사용 방법:

1. shortcut context를 선택합니다.
2. 변경할 action을 선택합니다.
3. 새 binding을 입력하거나 기존 binding을 제거합니다.

기대 결과:

- active keymap이 업데이트됩니다.
- custom binding이 `config.toml`의 `tui.keymap`에 저장됩니다.

binding 이름 예:

- `ctrl-a`
- `shift-enter`
- `page-down`

context-specific binding은 `tui.keymap.global`보다 우선합니다. 빈 binding list는 해당 action을 unbind합니다.

## `/statusline`: footer item 설정

TUI 하단 status line에 표시할 항목을 고르고 순서를 바꿉니다.

```text
/statusline
```

사용 방법:

1. picker에서 항목을 toggle합니다.
2. 순서를 조정합니다.
3. 확인합니다.

기대 결과:

- footer status line이 즉시 업데이트됩니다.
- 설정이 `config.toml`의 `tui.status_line`에 저장됩니다.

사용 가능한 항목:

- model
- model + reasoning
- context stats
- rate limits
- git branch
- token counters
- session id
- current directory / project root
- Codex version

## `/title`: terminal title item 설정

terminal window 또는 tab title에 표시할 항목을 설정합니다.

```text
/title
```

기대 결과:

- terminal title이 즉시 업데이트됩니다.
- 설정이 `config.toml`의 `tui.terminal_title`에 저장됩니다.

사용 가능한 항목:

- app name
- project
- spinner
- status
- thread
- git branch
- model
- task progress

## `/theme`: syntax theme 선택

TUI의 syntax highlighting theme를 preview하고 저장합니다.

```text
/theme
```

사용 방법:

1. theme picker에서 theme를 미리 봅니다.
2. 마음에 드는 theme를 확인합니다.

기대 결과:

- syntax highlighting이 업데이트됩니다.
- 선택값이 `config.toml`의 `tui.theme`에 저장됩니다.

## `/sandbox-add-read-dir`: sandbox read directory 추가

Windows native CLI에서만 사용할 수 있습니다. sandbox command가 현재 readable root 밖의 절대 경로를 읽어야 할 때 사용합니다.

```text
/sandbox-add-read-dir C:\absolute\directory\path
```

사용 방법:

1. existing absolute directory path를 인자로 제공합니다.
2. 경로가 올바른지 확인합니다.

기대 결과:

- Codex가 Windows sandbox policy를 refresh합니다.
- 이후 sandbox에서 실행되는 command가 해당 directory를 읽을 수 있습니다.

## `/debug-config`: config layer 진단

effective setting이 `config.toml`과 다르게 보일 때 설정 layer와 policy source를 확인합니다.

```text
/debug-config
```

확인할 수 있는 정보:

- config layer order
- on/off state
- policy source
- `allowed_approval_policies`
- `allowed_sandbox_modes`
- `mcp_servers`
- `rules`
- `enforce_residency`
- `experimental_network`

기대 결과:

- precedence가 낮은 layer부터 높은 layer까지 진단 정보를 출력합니다.
- 어떤 설정이 어디에서 왔는지 추적할 수 있습니다.

## `/feedback`: feedback 전송

문제 보고나 diagnostics 공유를 위해 로그를 Codex maintainers에게 보냅니다.

```text
/feedback
```

사용 방법:

1. `/feedback`을 입력합니다.
2. prompt를 따라 logs 또는 diagnostics 포함 여부를 선택합니다.

기대 결과:

- Codex가 요청한 diagnostics를 수집하고 maintainers에게 제출합니다.

민감한 정보가 포함될 수 있으므로 제출 전 내용을 주의 깊게 확인하는 것이 좋습니다.

## `/logout`: 로그아웃

현재 사용자 session의 local credential을 지웁니다.

```text
/logout
```

공유 머신, 임시 환경, pair programming 장비에서 작업을 마친 뒤 사용하면 좋습니다.

## `/quit`와 `/exit`: CLI 종료

CLI를 즉시 종료합니다.

```text
/quit
```

또는:

```text
/exit
```

기대 결과:

- Codex CLI가 종료됩니다.

중요한 변경사항은 종료 전에 저장하거나 커밋해야 합니다.

