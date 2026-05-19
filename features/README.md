# Codex CLI 기능 개요

이 문서는 Codex CLI가 단순한 채팅 도구를 넘어 어떤 개발 워크플로를 지원하는지 정리합니다. 원문 개발자 문서의 내용을 바탕으로, 실제 개발자가 CLI를 사용할 때 필요한 기능과 명령을 중심으로 번역 및 재구성했습니다.

## 문서 구성

- [인터랙티브 TUI 사용](interactive-tui.md): `codex`로 여는 전체 화면 터미널 UI, 입력 방식, 단축키, 테마, 프롬프트 편집
- [대화 재개와 자동화 실행](resume-and-exec.md): `resume`, `exec`, 단일 프롬프트 실행, 쉘 자동완성
- [원격 TUI와 Codex Cloud](remote-and-cloud.md): 원격 앱 서버 연결, WebSocket 인증, Codex Cloud 작업 실행
- [모델, 설정, 권한, 도구 연동](models-config-and-security.md): 모델 선택, 기능 플래그, 승인 모드, 웹 검색, MCP, 서브에이전트
- [이미지, 리뷰, Slash 명령, 팁](media-review-and-shortcuts.md): 이미지 입력/생성, 로컬 코드 리뷰, slash command, 실전 단축키

## 빠른 시작

대화형으로 Codex를 시작하려면:

```bash
codex
```

처음부터 요청을 함께 넘기려면:

```bash
codex "이 코드베이스 구조를 설명해줘"
```

비대화형 자동화 작업으로 실행하려면:

```bash
codex exec "CI 실패 원인을 찾아 수정해줘"
```

이전 세션을 이어서 작업하려면:

```bash
codex resume --last
```

## 기능을 선택하는 기준

| 상황 | 추천 기능 |
| --- | --- |
| Codex와 대화하면서 수정 과정을 실시간으로 보고 싶다 | 인터랙티브 TUI |
| 이전 맥락을 이어서 작업하고 싶다 | `codex resume` |
| 스크립트나 CI에서 Codex를 호출하고 싶다 | `codex exec` |
| 큰 작업을 병렬로 나누고 싶다 | 서브에이전트 |
| PR 전 위험 요소를 점검하고 싶다 | `/review` |
| 스크린샷이나 디자인 시안을 함께 설명하고 싶다 | 이미지 입력 |
| 아이콘, 배너, 플레이스홀더 이미지를 만들고 싶다 | 이미지 생성 |
| 다른 머신의 Codex 앱 서버에 붙고 싶다 | 원격 TUI |
| Codex Cloud 작업을 터미널에서 시작하고 싶다 | `codex cloud` |

