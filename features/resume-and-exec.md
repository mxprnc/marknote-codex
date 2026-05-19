# 대화 재개와 자동화 실행

Codex는 로컬에 대화 transcript를 저장합니다. 덕분에 이전 작업 맥락을 다시 설명하지 않고 이어서 진행할 수 있고, `exec` 서브커맨드로 비대화형 자동화도 구성할 수 있습니다.

## 이전 대화 재개하기

가장 일반적인 재개 명령:

```bash
codex resume
```

이 명령은 최근 인터랙티브 세션 목록을 picker로 보여줍니다. 항목을 선택하면 요약을 확인할 수 있고, `Enter`를 눌러 다시 열 수 있습니다.

## `resume` 주요 옵션

| 명령 | 용도 |
| --- | --- |
| `codex resume` | 현재 작업 디렉터리 기준 최근 세션 선택 |
| `codex resume --all` | 현재 디렉터리와 관계없이 모든 로컬 세션 표시 |
| `codex resume --last` | 현재 작업 디렉터리의 가장 최근 세션으로 바로 이동 |
| `codex resume --last --all` | 디렉터리 필터 없이 가장 최근 세션으로 이동 |
| `codex resume <SESSION_ID>` | 특정 세션 ID로 직접 재개 |

세션 ID는 picker, `/status`, 또는 `~/.codex/sessions/` 아래의 저장 파일에서 확인할 수 있습니다.

## 재개된 세션이 유지하는 정보

재개된 세션은 다음 정보를 이어받습니다.

- 원래 transcript
- 계획(plan) 기록
- 승인(approval) 흐름
- 기존 지시사항과 작업 맥락

즉, Codex는 과거 대화에서 이미 합의한 설계나 발견한 문제를 활용하면서 새 지시를 수행할 수 있습니다.

작업 디렉터리를 바꾸고 싶다면 `--cd`를 사용합니다.

```bash
codex --cd /path/to/project resume --last
```

추가로 접근 가능한 루트를 열고 싶다면 `--add-dir`를 사용할 수 있습니다.

```bash
codex --cd apps/frontend --add-dir ../backend --add-dir ../shared
```

## 비대화형 실행: `codex exec`

빠른 자동화나 스크립트 연동에는 `exec`를 사용합니다. `exec`는 인터랙티브 UI를 열지 않고 Codex를 실행한 뒤 최종 계획과 결과를 `stdout`으로 출력합니다.

```bash
codex exec "fix the CI failure"
```

예를 들어 다음과 같은 작업에 적합합니다.

- CI 실패 원인 분석과 수정
- changelog 자동 업데이트
- 이슈 목록 정리
- PR 전 문서 스타일 점검
- 반복적인 코드 정리 작업

## 자동화 실행에서도 대화 재개하기

비대화형 자동화에서도 이전 세션을 이어갈 수 있습니다.

```bash
codex exec resume --last "Fix the race conditions you found"
```

특정 세션 ID를 지정할 수도 있습니다.

```bash
codex exec resume 7f9f9a2e-1b3c-4c7a-9b0e-.... "Implement the plan"
```

이 방식은 인터랙티브 세션에서 문제를 조사한 뒤, 나중에 스크립트나 별도 실행으로 구현을 이어가고 싶을 때 유용합니다.

## 단일 프롬프트로 빠르게 실행하기

간단한 질문이나 일회성 분석은 `codex`에 프롬프트를 바로 전달하면 됩니다.

```bash
codex "explain this codebase"
```

Codex는 현재 작업 디렉터리를 읽고, 필요한 계획을 세운 뒤 응답을 터미널에 스트리밍하고 종료합니다.

특정 디렉터리를 대상으로 실행하려면 `--path` 또는 `--cd` 같은 옵션을 함께 사용할 수 있습니다.

```bash
codex --cd packages/api "이 패키지의 public API를 요약해줘"
```

모델을 명시할 수도 있습니다.

```bash
codex --model gpt-5.5 "리팩터링 위험이 큰 부분을 찾아줘"
```

## 쉘 자동완성

자주 쓰는 옵션과 서브커맨드를 빠르게 입력하려면 shell completion을 설치합니다.

```bash
codex completion bash
codex completion zsh
codex completion fish
```

`zsh`를 쓴다면 `~/.zshrc` 끝에 다음을 추가할 수 있습니다.

```bash
# ~/.zshrc
eval "$(codex completion zsh)"
```

새 터미널을 열고 `codex`를 입력한 뒤 `Tab`을 누르면 자동완성이 동작합니다.

만약 `command not found: compdef` 오류가 나오면, `eval "$(codex completion zsh)"`보다 앞에 다음 줄을 추가합니다.

```bash
autoload -Uz compinit && compinit
```

## 실행 전 환경 준비 팁

Codex를 실행하기 전에 개발 환경을 미리 준비해두면 불필요한 탐색과 토큰 사용을 줄일 수 있습니다.

- Python 가상환경, Node 버전 관리자, language-specific 환경을 미리 활성화하기
- 필요한 daemon이나 로컬 DB를 먼저 실행하기
- 테스트나 빌드에 필요한 환경 변수를 export하기
- 모노레포라면 `--cd`와 `--add-dir`로 작업 범위를 명확히 지정하기

