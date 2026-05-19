# 세션 제어와 대화 흐름

이 문서는 active model, 응답 스타일, 계획 모드, goal, transcript 관리처럼 현재 Codex 세션의 흐름을 바꾸는 slash command를 정리합니다.

## `/model`: active model 설정

현재 세션에서 사용할 모델과, 가능한 경우 reasoning effort를 선택합니다.

사용 방법:

1. composer에서 `/model`을 입력하고 `Enter`를 누릅니다.
2. popup에서 사용할 모델을 선택합니다.
3. 필요하면 reasoning effort도 선택합니다.

예:

```text
/model
```

기대 결과:

- Codex가 transcript에 새 모델을 확인해 줍니다.
- `/status`로 active model이 바뀌었는지 검증할 수 있습니다.

```text
/status
```

작업 기준:

- 가벼운 질문이나 빠른 수정: 빠른 모델
- 복잡한 리팩터링, 설계, 디버깅: reasoning이 강한 모델
- 비용/속도보다 품질이 중요한 작업: 더 강한 모델

## `/fast`: Fast mode 토글

현재 모델이 Fast service tier를 제공할 때만 사용할 수 있습니다.

사용 방법:

```text
/fast on
/fast off
/fast status
```

설정을 지속시키고 싶다면 Codex가 저장 여부를 물을 때 확인합니다.

기대 결과:

- 현재 thread에서 Fast tier가 켜졌는지 꺼졌는지 알려줍니다.
- `/statusline`으로 TUI footer에 Fast mode 상태 항목을 표시할 수 있습니다.

Fast tier는 model catalog 기반입니다. 현재 모델이 Fast tier를 advertise하지 않으면 `/fast` 명령이 표시되지 않을 수 있습니다.

## `/personality`: 커뮤니케이션 스타일 설정

프롬프트를 다시 쓰지 않고 Codex의 응답 방식을 바꿉니다.

사용 방법:

```text
/personality
```

popup에서 style을 선택합니다.

지원 style:

- `friendly`
- `pragmatic`
- `none`

`none`은 personality instruction을 비활성화합니다.

기대 결과:

- Codex가 transcript에 새 style을 확인합니다.
- 이후 응답에서 선택한 style이 적용됩니다.

active model이 personality-specific instruction을 지원하지 않으면 이 명령은 숨겨집니다.

## `/plan`: plan mode로 전환

구현을 바로 시작하기 전에 실행 계획을 먼저 받고 싶을 때 사용합니다.

사용 방법:

```text
/plan
```

inline prompt를 함께 줄 수도 있습니다.

```text
/plan 이 서비스를 새 인증 방식으로 마이그레이션하는 계획을 제안해줘
```

이미지나 긴 내용을 붙여넣은 상태에서도 inline `/plan` 인자를 사용할 수 있습니다.

기대 결과:

- Codex가 active conversation을 plan mode로 전환합니다.
- inline prompt가 있으면 첫 planning request로 사용합니다.

주의:

- task가 이미 실행 중이면 `/plan`은 일시적으로 사용할 수 없습니다.

## `/goal`: experimental task goal 설정

`/goal`은 experimental 명령입니다. `features.goals`가 켜져 있을 때만 사용할 수 있습니다.

활성화 방법:

```text
/experimental
```

또는 `config.toml`의 `[features]` 아래에 설정합니다.

```toml
[features]
goals = true
```

사용 방법:

```text
/goal Finish the migration and keep tests green
```

현재 goal 확인:

```text
/goal
```

일시 중지, 재개, 삭제:

```text
/goal pause
/goal resume
/goal clear
```

기대 결과:

- Codex가 active thread에 goal을 붙이고, 작업이 이어지는 동안 이를 추적합니다.

제약:

- goal objective는 비어 있으면 안 됩니다.
- 최대 4,000자입니다.
- 더 긴 지시는 파일에 쓰고 goal에서 그 파일을 참조하는 방식이 좋습니다.

## `/clear`: 터미널을 지우고 새 chat 시작

터미널 화면과 대화 맥락을 함께 새로 시작합니다.

```text
/clear
```

기대 결과:

- terminal view를 지웁니다.
- visible transcript를 reset합니다.
- 같은 CLI session 안에서 fresh chat을 시작합니다.

`Ctrl+L`과 차이:

- `/clear`: 새 conversation을 시작합니다.
- `Ctrl+L`: terminal view만 지우고 현재 chat은 유지합니다.

Codex는 task가 진행 중일 때 두 동작 모두 비활성화합니다.

## `/new`: 같은 CLI session에서 새 conversation 시작

현재 terminal session은 유지하고 chat context만 새로 시작합니다.

```text
/new
```

기대 결과:

- 같은 CLI session 안에서 fresh conversation을 시작합니다.

`/clear`와 달리 현재 terminal view를 먼저 지우지는 않습니다.

## `/resume`: 저장된 conversation 재개

저장된 session 목록에서 이전 대화를 골라 이어갑니다.

```text
/resume
```

기대 결과:

- saved-session picker가 열립니다.
- 선택한 conversation transcript가 reload됩니다.
- 원래 history를 유지한 채 이어서 작업할 수 있습니다.

터미널 명령 `codex resume`과 비슷하지만, 현재 TUI 안에서 수행한다는 차이가 있습니다.

## `/fork`: 현재 conversation fork

현재 conversation을 새 thread로 복제합니다.

```text
/fork
```

기대 결과:

- fresh ID를 가진 새 thread가 만들어집니다.
- 원본 transcript는 그대로 남습니다.
- 다른 접근을 병렬로 탐색할 수 있습니다.

저장된 session을 fork하려면 TUI slash command 대신 터미널에서 `codex fork`를 실행해 session picker를 열 수 있습니다.

## `/compact`: transcript 요약

긴 대화 이후 context를 아끼기 위해 이전 turn을 concise summary로 대체합니다.

사용 방법:

```text
/compact
```

기대 결과:

- Codex가 지금까지의 대화를 요약할지 확인합니다.
- 확인하면 이전 turn을 핵심 요약으로 압축합니다.
- 중요한 결정과 맥락을 유지하면서 context window를 확보합니다.

긴 구현 세션, 조사 세션, 여러 파일을 오간 디버깅 세션 이후에 특히 유용합니다.

## `/copy`: 최근 완료 응답 복사

최근 완료된 Codex output을 clipboard로 복사합니다.

```text
/copy
```

또는 main TUI에서:

```text
Ctrl+O
```

기대 결과:

- 최신 완료 응답이나 계획 text가 clipboard에 복사됩니다.

주의:

- 현재 turn이 아직 실행 중이면 in-progress response가 아니라 최신 완료 output을 복사합니다.
- 첫 완료 output 전에는 사용할 수 없습니다.
- rollback 직후에는 일시적으로 사용할 수 없습니다.

## `/raw`: raw scrollback mode 토글

terminal selection과 copy를 더 직접적으로 하기 위해 raw scrollback mode를 켭니다.

```text
/raw
/raw on
/raw off
```

기대 결과:

- raw scrollback mode가 켜지거나 꺼집니다.
- 긴 output을 selection/copy할 때 formatting 간섭을 줄일 수 있습니다.

단축키:

```text
Alt+R
```

기본값으로 유지하려면 `config.toml`에 설정합니다.

```toml
tui.raw_output_mode = true
```

## `/status`: 세션 상태 확인

현재 Codex session의 설정과 token 사용량을 확인합니다.

```text
/status
```

확인할 수 있는 정보:

- active model
- approval policy
- writable roots
- 현재 token usage
- 남은 context capacity
- session 관련 정보

기대 결과:

- shell의 `codex status`와 유사한 요약을 TUI 안에서 확인할 수 있습니다.

큰 작업 전후에 `/status`를 실행하면 Codex가 의도한 모델과 권한으로 동작 중인지 확인하기 좋습니다.

## `/ps`: background terminal 확인

experimental background terminal 목록과 최근 출력을 보여줍니다.

```text
/ps
```

기대 결과:

- 각 background terminal의 command를 보여줍니다.
- 최대 세 줄의 최근 non-empty output을 보여줍니다.

`unified_exec`를 사용할 때 background terminal이 표시됩니다. 그렇지 않으면 목록이 비어 있을 수 있습니다.

## `/stop`: background terminal 중지

현재 session이 시작한 모든 background terminal을 중지합니다.

```text
/stop
```

Codex가 확인을 요청하면 중지할 terminal 목록을 확인하고 승인합니다.

기대 결과:

- 현재 session의 background terminal 작업이 모두 중지됩니다.

`/clean`은 여전히 `/stop`의 alias입니다.

