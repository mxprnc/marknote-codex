# 이미지, 리뷰, Slash 명령, 팁

이 문서는 Codex CLI에서 이미지 입력과 이미지 생성, 로컬 코드 리뷰, slash command, 실전 단축키를 사용하는 방법을 정리합니다.

## 이미지 입력

Codex는 프롬프트와 함께 스크린샷이나 디자인 시안을 읽을 수 있습니다. 인터랙티브 composer에 이미지를 붙여넣거나, 명령줄에서 파일을 지정하면 됩니다.

단일 이미지:

```bash
codex -i screenshot.png "Explain this error"
```

여러 이미지:

```bash
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

PNG, JPEG 같은 일반적인 이미지 형식을 사용할 수 있습니다. 여러 파일은 쉼표로 구분합니다.

## 이미지 입력 활용 예

오류 화면 분석:

```bash
codex -i error-screen.png "이 오류가 어느 코드 경로에서 발생할 가능성이 큰지 추정해줘"
```

디자인 시안 기반 구현:

```bash
codex --image dashboard.png "이 시안을 기준으로 현재 React 화면을 구현해줘"
```

다이어그램 요약:

```bash
codex --image arch1.png,arch2.png "두 아키텍처 다이어그램의 차이를 표로 정리해줘"
```

이미지를 사용할 때는 “무엇을 봐야 하는지”를 텍스트로 함께 적는 것이 좋습니다. 예를 들어 “접근성 문제 중심으로 봐줘”, “레이아웃 간격만 비교해줘”처럼 관점을 지정하면 결과가 더 안정적입니다.

## 이미지 생성

Codex CLI에서 직접 이미지를 생성하거나 편집할 수 있습니다. 아이콘, 배너, 일러스트, 스프라이트 시트, 플레이스홀더 아트 같은 asset을 만들 때 유용합니다.

자연어로 요청할 수 있습니다.

```text
로그인 화면에 쓸 간단한 빈 상태 일러스트를 만들어줘
```

또는 이미지 생성 skill을 명시적으로 호출할 수 있습니다.

```text
$imagegen 제품 카드에 넣을 미니멀한 노트 앱 아이콘을 생성해줘
```

기존 asset을 변형하거나 확장하려면 참조 이미지를 함께 첨부합니다.

```bash
codex -i current-icon.png "이 아이콘 스타일을 유지하되 다크 모드용 버전을 만들어줘"
```

내장 이미지 생성은 `gpt-image-2`를 사용하며 일반 Codex 사용량 한도에 포함됩니다. 대량 생성이 필요하면 `OPENAI_API_KEY`를 환경 변수로 설정한 뒤 API 기반 생성으로 처리하도록 요청할 수 있습니다. 이 경우 API 과금 체계가 적용됩니다.

## 로컬 코드 리뷰

인터랙티브 CLI에서 `/review`를 입력하면 Codex의 리뷰 preset을 열 수 있습니다.

```text
/review
```

리뷰어는 선택한 diff를 읽고, 작업 트리를 수정하지 않은 채 우선순위가 높은 actionable finding을 보고합니다. 기본적으로 현재 세션의 모델을 사용하며, `config.toml`의 `review_model`로 별도 모델을 지정할 수 있습니다.

## 리뷰 모드

| 모드 | 설명 |
| --- | --- |
| Review against a base branch | 로컬 브랜치를 선택하면 upstream과 merge base를 찾고, 현재 작업 diff의 위험을 점검 |
| Review uncommitted changes | staged, unstaged, untracked 변경을 모두 검사 |
| Review a commit | 최근 commit 목록에서 SHA를 골라 해당 변경 집합만 검사 |
| Custom review instructions | “접근성 회귀에 집중해줘” 같은 직접 지시를 넣어 리뷰 |

각 리뷰 실행은 transcript의 독립된 턴으로 남습니다. 코드가 바뀔 때마다 다시 실행해 피드백 변화를 비교할 수 있습니다.

## Slash 명령

Slash command는 `/review`, `/fork`, `/side`처럼 특수 workflow를 빠르게 실행하는 명령입니다.

Codex는 built-in slash command를 제공하고, 사용자가 팀 또는 개인용 custom command를 만들 수도 있습니다.

예:

```text
/review
/theme
/permissions
/model
```

팀에서 반복하는 작업이 있다면 custom slash command로 만들어 둘 수 있습니다.

예를 들어 다음 같은 요청을 재사용 명령으로 만들 수 있습니다.

```text
이 PR의 사용자 영향, 마이그레이션 위험, 테스트 누락을 중심으로 리뷰해줘
```

## 유용한 TUI 팁

입력창에서 `@`를 입력하면 워크스페이스 파일 검색이 열립니다.

```text
@README.md 이 문서의 설치 설명이 현재 코드와 맞는지 확인해줘
```

Codex가 실행 중일 때 `Enter`를 누르면 현재 턴에 새 지시를 주입할 수 있습니다.

```text
방금 말한 수정에서 테스트 파일도 같이 봐줘
```

Codex가 실행 중일 때 `Tab`을 누르면 다음 턴에 실행할 입력을 큐에 넣습니다. 큐에는 일반 프롬프트, slash command, `!` shell command를 넣을 수 있습니다.

```text
/review
```

또는:

```text
!npm test
```

입력 줄 앞에 `!`를 붙이면 로컬 shell command를 실행합니다.

```text
!git status --short
```

## 작업 디렉터리와 루트 설정

현재 디렉터리를 바꾸지 않고 특정 경로를 Codex 작업 루트로 지정할 수 있습니다.

```bash
codex --cd <path>
```

모노레포에서 여러 프로젝트를 함께 수정해야 한다면 추가 writable root를 지정합니다.

```bash
codex --cd apps/frontend --add-dir ../backend --add-dir ../shared
```

TUI header에는 현재 활성 경로가 표시됩니다.

## 실무 권장 패턴

작업을 시작하기 전에 필요한 환경을 먼저 준비합니다.

```bash
source .venv/bin/activate
export DATABASE_URL=postgres://localhost:5432/app
codex
```

이렇게 하면 Codex가 환경을 탐색하는 데 시간을 덜 쓰고, 실제 구현이나 분석에 더 집중할 수 있습니다.

큰 변경 전에는 `/review`를 한 번 실행하고, 변경 후 다시 실행해 회귀 위험이 줄었는지 확인하는 흐름이 좋습니다.

```text
/review
```

디자인 작업은 이미지 입력과 함께 구체적인 기준을 적습니다.

```bash
codex --image target.png "현재 화면을 이 시안에 맞춰 수정하되, 색상 토큰과 기존 컴포넌트는 유지해줘"
```

