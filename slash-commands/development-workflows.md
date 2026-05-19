# 개발 워크플로 명령

이 문서는 코드 수정, 리뷰, 파일 참조, MCP 도구, app/plugin/hook, subagent thread처럼 개발 작업 흐름에 직접 연결되는 slash command를 정리합니다.

## `/diff`: 변경사항 확인

현재 Git diff를 CLI 안에서 확인합니다.

```text
/diff
```

기대 결과:

- staged change
- unstaged change
- Git이 아직 추적하지 않는 새 파일

까지 함께 보여줍니다.

사용 예:

```text
/diff
```

Codex가 구현을 마친 직후 diff를 검토하고, 유지할 변경과 버릴 변경을 판단할 때 사용합니다.

## `/review`: working tree 리뷰

현재 working tree를 Codex에게 리뷰하게 합니다.

```text
/review
```

기대 결과:

- Codex가 동작 변화, 회귀 위험, 누락된 테스트를 중심으로 issue를 요약합니다.
- 기본적으로 현재 session model을 사용합니다.
- `config.toml`에 `review_model`을 설정하면 리뷰 전용 모델을 사용할 수 있습니다.

추천 흐름:

```text
/diff
/review
```

리뷰 후 exact file change를 다시 확인하고 싶으면 `/diff`를 이어서 사용합니다.

## `/mention`: 파일 강조/첨부

특정 파일이나 폴더를 대화에 첨부해 다음 턴에서 직접 참조하게 합니다.

```text
/mention src/lib/api.ts
```

사용 방법:

1. `/mention` 뒤에 경로 일부를 입력합니다.
2. popup에서 matching result를 선택합니다.

기대 결과:

- 선택한 파일이 conversation에 추가됩니다.
- follow-up turn에서 Codex가 해당 파일을 직접 참조합니다.

예:

```text
/mention packages/api/src/auth.ts
이 파일의 인증 실패 처리를 리뷰해줘
```

## `/init`: `AGENTS.md` scaffold 생성

현재 디렉터리에 persistent instruction 파일인 `AGENTS.md`의 초안을 생성합니다.

```text
/init
```

사용 방법:

1. Codex가 persistent instruction을 찾길 원하는 디렉터리에서 `/init`을 실행합니다.
2. 생성된 `AGENTS.md`를 검토합니다.
3. 저장소 convention에 맞게 수정하고 커밋합니다.

기대 결과:

- 향후 Codex session에서 사용할 수 있는 `AGENTS.md` scaffold가 생성됩니다.

예를 들어 monorepo에서는 root와 특정 package 아래에 서로 다른 `AGENTS.md`를 둘 수 있습니다.

## `/mcp`: MCP tools 확인

현재 session에서 Codex가 호출할 수 있는 Model Context Protocol 도구를 나열합니다.

```text
/mcp
```

자세한 server diagnostics 포함:

```text
/mcp verbose
```

기대 결과:

- 설정된 MCP server와 tool 목록을 확인할 수 있습니다.
- 외부 도구가 실제로 session에 로드되었는지 확인할 때 유용합니다.

`verbose`가 아닌 다른 인자를 넘기면 Codex는 command usage를 보여줍니다.

## `/apps`: app(connector) 탐색

사용 가능한 app(connector)을 목록에서 고르고 prompt에 삽입합니다.

```text
/apps
```

기대 결과:

- app mention이 composer에 `$app-slug` 형태로 들어갑니다.
- 이어서 Codex에게 해당 app을 사용하도록 요청할 수 있습니다.

예:

```text
$google-drive 이번 주 회의록을 찾아 요약해줘
```

실제 app 사용 가능 여부는 설치/연결 상태와 현재 Codex 환경에 따라 달라집니다.

## `/plugins`: plugin 탐색과 관리

설치된 plugin과 discoverable plugin을 탐색합니다.

```text
/plugins
```

사용 방법:

1. marketplace tab을 선택합니다.
2. plugin을 선택해 capability나 available action을 확인합니다.
3. 설치된 plugin에서 `Space`를 눌러 enabled state를 토글할 수 있습니다.

기대 결과:

- installed plugin state를 확인합니다.
- 설정상 discover 가능한 plugin을 탐색합니다.
- 필요한 plugin의 기능을 이해하고 활성/비활성 상태를 관리합니다.

## `/hooks`: lifecycle hook 확인과 관리

현재 session에 로드된 lifecycle hook 구성을 확인합니다.

```text
/hooks
```

사용 방법:

1. hook event를 선택합니다.
2. matching handler를 확인합니다.
3. 필요하면 non-managed hook을 trust, disable, re-enable합니다.

기대 결과:

- 현재 session에서 실행될 수 있는 lifecycle hook을 검토할 수 있습니다.
- managed hook은 managed로 표시되며 user hook browser에서 비활성화할 수 없습니다.

hook이 명령 실행 전후에 어떤 영향을 줄 수 있는지 확인하고 싶을 때 유용합니다.

## `/agent`: agent thread 전환

spawn된 subagent thread를 확인하거나 이어서 작업하기 위해 active thread를 전환합니다.

```text
/agent
```

사용 방법:

1. `/agent`를 입력하고 `Enter`를 누릅니다.
2. picker에서 thread를 선택합니다.

기대 결과:

- Codex가 active thread를 선택한 agent thread로 전환합니다.
- 해당 agent의 작업을 확인하거나 이어서 진행할 수 있습니다.

## `/side`: 임시 side conversation

main task의 transcript를 방해하지 않고 짧은 detour를 수행합니다.

```text
/side
```

inline 질문:

```text
/side 이 계획에서 가장 큰 회귀 위험만 확인해줘
```

기대 결과:

- 현재 conversation에서 ephemeral fork가 열립니다.
- side conversation의 transcript는 parent thread와 분리됩니다.
- side mode 중에도 TUI는 parent-thread status를 보여주므로 main task가 진행 중인지 확인할 수 있습니다.

제약:

- 다른 side conversation 안에서는 사용할 수 없습니다.
- review mode 중에는 사용할 수 없습니다.

짧은 검토, 대안 비교, 특정 계획의 리스크만 묻는 작업에 적합합니다.

