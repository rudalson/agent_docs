> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Claude Code 작동 방식

> 에이전트 루프, 내장 도구, 그리고 Claude Code가 프로젝트와 상호작용하는 방식을 이해해 보세요.

Claude Code는 터미널에서 실행되는 에이전트 어시스턴트입니다. 코딩에 매우 탁월하지만, 문서 작성, 빌드 실행, 파일 검색, 주제 조사 등 명령줄에서 할 수 있는 모든 작업을 지원합니다.

이 가이드는 핵심 아키텍처, 내장 기능, 그리고 [Claude Code와 효과적으로 작업하기 위한 팁](#work-effectively-with-claude-code)을 다룹니다. 단계별 설명은 [일반적인 워크플로](/docs/en/common-workflows)를 참조하세요. 스킬, MCP, 훅과 같은 확장 기능은 [Claude Code 확장하기](/docs/en/features-overview)를 참조하세요.

## 에이전트 루프 (The agentic loop)

Claude에 작업을 부여하면 **컨텍스트 수집**, **작업 수행**, **결과 검증**의 세 단계를 거쳐 진행됩니다. 이 단계들은 자연스럽게 이어집니다. Claude는 코드를 이해하기 위해 파일을 검색하든, 변경 사항을 편집하든, 작업을 확인하기 위해 테스트를 실행하든 관계없이 전 과정에서 도구를 사용합니다.

<img src="https://mintcdn.com/claude-code/ikqp3_70mqIahteV/images/agentic-loop.svg?fit=max&auto=format&n=ikqp3_70mqIahteV&q=85&s=4a30fb7ce2815012a9f27c955e2c6bb0" alt="Diagram of the agentic loop: Your prompt leads to Claude gathering context, taking action, verifying results, and repeating until task complete. You can interrupt at any point." width="720" height="280" data-path="images/agentic-loop.svg" />

이 루프는 질문에 따라 맞춤 적용됩니다. 코드베이스에 대한 질문은 컨텍스트 수집만 필요할 수 있습니다. 버그 수정은 세 단계를 반복하여 거칩니다. 리팩토링에는 광범위한 검증이 포함될 수 있습니다. Claude는 이전 단계에서 배운 내용을 바탕으로 각 단계에 필요한 사항을 결정하여 수십 가지 작업을 연결하고 그 과정에서 수정을 거칩니다.

사용자도 이 루프의 일부입니다. 언제든지 중단하여 Claude를 다른 방향으로 조율하거나, 추가 컨텍스트를 제공하거나, 다른 접근 방식을 시도하도록 요청할 수 있습니다. Claude는 자율적으로 작업하지만 사용자의 입력에 즉각 응답합니다.

에이전트 루프는 두 가지 구성 요소로 구동됩니다: 추론하는 [모델](#models)과 작업을 수행하는 [도구](#tools). Claude Code는 Claude를 둘러싼 **에이전트 하네스(agentic harness)** 역할을 합니다: 언어 모델을 유능한 코딩 에이전트로 전환하는 도구, 컨텍스트 관리, 실행 환경을 제공합니다.

### 모델

Claude Code는 Claude 모델을 사용하여 코드를 이해하고 작업에 대해 추론합니다. Claude는 모든 언어로 된 코드를 읽고, 구성 요소가 어떻게 연결되는지 이해하며, 목표를 달성하기 위해 무엇을 변경해야 하는지 파악할 수 있습니다. 복잡한 작업의 경우 작업을 단계별로 나누고, 실행하며, 배운 내용을 바탕으로 조정합니다.

다양한 트레이드오프를 가진 [여러 모델](/docs/en/model-config)을 사용할 수 있습니다. Sonnet은 대부분의 코딩 작업을 잘 처리합니다. Opus는 복잡한 아키텍처 결정에 대해 더 강력한 추론을 제공합니다. 세션 중에 `/model`로 전환하거나 `claude --model <name>`으로 시작하세요.

이 가이드에서 "Claude가 선택함" 또는 "Claude가 결정함"이라고 말할 때, 추론을 수행하는 주체는 모델입니다.

### 도구

도구는 Claude Code를 에이전트로 만들어 주는 핵심입니다. 도구가 없으면 Claude는 텍스트로만 응답할 수 있습니다. 도구가 있으면 Claude는 코드를 읽고, 파일을 편집하고, 명령을 실행하고, 웹을 검색하고, 외부 서비스와 상호작용할 수 있습니다. 각 도구 사용은 루프로 피드백되는 정보를 반환하여 Claude의 다음 결정을 보조합니다.

내장 도구는 일반적으로 5가지 범주로 나뉘며, 각 범주는 서로 다른 종류의 에이전트 능력을 나타냅니다.

| 범주              | Claude가 할 수 있는 작업                                                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **파일 작업**   | 파일 읽기, 코드 편집, 새 파일 생성, 이름 변경 및 재구성                                                                                                |
| **검색**            | 패턴으로 파일 찾기, 정규식으로 내용 검색, 코드베이스 탐색                                                                                           |
| **실행**         | 셸 명령 실행, 서버 시작, 테스트 실행, git 사용                                                                                                         |
| **웹**               | 웹 검색, 문서 가져오기, 오류 메시지 조회                                                                                                   |
| **코드 인텔리전스** | 편집 후 타입 오류 및 경고 확인, 정의로 이동, 참조 찾기 ([코드 인텔리전스 플러그인](/docs/en/discover-plugins#code-intelligence) 필요) |

이것들이 기본 기능입니다. Claude에는 서브에이전트를 생성하고, 질문을 던지고, 기타 오케스트레이션 작업을 수행하는 도구도 있습니다. 전체 목록은 [Claude에서 사용할 수 있는 도구](/docs/en/tools-reference)를 참조하세요.

Claude는 프롬프트와 과정에서 학습한 내용을 바탕으로 사용할 도구를 선택합니다. "실패한 테스트 수정"을 요청하면 Claude는 다음과 같이 수행할 수 있습니다:

1. 실패 원인을 확인하기 위해 테스트 스위트 실행
2. 오류 출력 읽기
3. 관련 소스 파일 검색
4. 코드를 이해하기 위해 해당 파일 읽기
5. 문제를 수정하기 위해 파일 편집
6. 검증을 위해 테스트 다시 실행

각 도구 사용은 다음 단계의 기반이 되는 새로운 정보를 Claude에게 제공합니다. 이것이 바로 실행 중인 에이전트 루프입니다.

**기본 기능 확장:** 내장 도구는 기반을 제공합니다. [스킬](/docs/en/skills)로 Claude가 아는 내용을 확장하고, [MCP](/docs/en/mcp)로 외부 서비스에 연결하며, [훅](/docs/en/hooks)으로 워크플로를 자동화하고, [서브에이전트](/docs/en/sub-agents)에 작업을 위임할 수 있습니다. 이러한 확장 기능은 핵심 에이전트 루프 위에 층을 형성합니다. 필요에 맞는 연동을 선택하려면 [Claude Code 확장하기](/docs/en/features-overview)를 참조하세요.

## Claude가 접근할 수 있는 항목

이 가이드는 터미널에 중점을 둡니다. Claude Code는 [VS Code](/docs/en/vs-code), [JetBrains IDE](/docs/en/jetbrains) 및 기타 환경에서도 실행됩니다.

디렉토리에서 `claude`를 실행하면 Claude Code는 다음에 대한 접근 권한을 얻습니다:

* **프로젝트.** 해당 디렉토리 및 하위 디렉토리의 파일과 사용자 허가를 받은 다른 위치의 파일.
* **터미널.** 실행할 수 있는 모든 명령: 빌드 도구, git, 패키지 관리자, 시스템 유틸리티, 스크립트. 명령줄에서 할 수 있는 일이라면 Claude도 할 수 있습니다.
* **Git 상태.** 현재 브랜치, 커밋되지 않은 변경 사항 및 최근 커밋 내역.
* **[CLAUDE.md](/docs/en/memory).** 매 세션마다 Claude가 알아야 하는 프로젝트별 지침, 규칙 및 컨텍스트를 저장하는 마크다운 파일.
* **[자동 메모리](/docs/en/memory#auto-memory).** 프로젝트 패턴 및 사용자 선호도와 같이 작업하면서 Claude가 자동으로 저장하는 학습 내용. MEMORY.md의 처음 200줄 또는 25KB 중 먼저 도달하는 분량이 각 세션 시작 시 로드됩니다.
* **사용자가 구성한 확장 기능.** 외부 서비스를 위한 [MCP 서버](/docs/en/mcp), 워크플로를 위한 [스킬](/docs/en/skills), 위임된 작업을 위한 [서브에이전트](/docs/en/sub-agents), 브라우저 상호작용을 위한 [Claude in Chrome](/docs/en/chrome).

Claude는 전체 프로젝트를 볼 수 있으므로 프로젝트 전체에 걸쳐 작업할 수 있습니다. "인증 버그 수정"을 요청하면 관련 파일을 검색하고, 컨텍스트를 이해하기 위해 여러 파일을 읽고, 파일 전체에 걸쳐 조율된 편집을 수행하고, 테스트를 실행하여 수정을 검증하고, 요청 시 변경 사항을 커밋합니다. 이는 현재 파일만 보는 인라인 코드 어시스턴트와는 다릅니다.

## 환경 및 인터페이스

위에서 설명한 에이전트 루프, 도구 및 기능은 Claude Code를 사용하는 모든 곳에서 동일합니다. 달라지는 것은 코드가 실행되는 위치와 인터페이스 상호작용 방식입니다.

### 실행 환경

Claude Code는 세 가지 환경에서 실행되며, 각 환경은 코드가 실행되는 위치에 따라 서로 다른 트레이드오프를 가집니다.

| 환경        | 코드가 실행되는 위치                         | 사용 사례                                                   |
| ------------------ | --------------------------------------- | ---------------------------------------------------------- |
| **로컬 (Local)**          | 사용자의 컴퓨터                            | 기본값. 파일, 도구 및 환경에 대한 전체 접근 권한 |
| **클라우드 (Cloud)**          | Anthropic이 관리하는 VM                   | 작업 오프로드, 로컬에 없는 저장소 작업        |
| **Remote Control** | 사용자의 컴퓨터, 브라우저에서 제어 | 실행 및 파일은 로컬에 유지하면서 웹 UI 사용   |

### 인터페이스

터미널, [데스크톱 앱](/docs/en/desktop), [IDE 확장 기능](/docs/en/vs-code), [claude.ai/code](https://claude.ai/code), [Remote Control](/docs/en/remote-control), [Slack](/docs/en/slack), [CI/CD 파이프라인](/docs/en/github-actions)을 통해 Claude Code에 접근할 수 있습니다. 인터페이스는 Claude를 보고 상호작용하는 방식을 결정하지만, 근본적인 에이전트 루프는 동일합니다. 전체 목록은 [어디서나 Claude Code 사용하기](/docs/en/overview#use-claude-code-everywhere)를 참조하세요.

## 세션 활용하기

Claude Code는 작업에 따라 대화 내용을 로컬에 저장합니다. 각 메시지, 도구 사용 및 결과는 `~/.claude/projects/` 아래의 일반 텍스트 JSONL 파일에 기록되어 세션 [되돌리기(rewinding)](#undo-changes-with-checkpoints), [재개 및 포크(forking)](#resume-or-fork-sessions)를 가능하게 합니다. Claude가 코드를 변경하기 전에 필요한 경우 되돌릴 수 있도록 영향을 받는 파일을 스냅샷으로 저장합니다. 경로, 보존 및 이 데이터를 지우는 방법은 [`~/.claude` 내 애플리케이션 데이터](/docs/en/claude-directory#application-data)를 참조하세요.

**세션은 독립적입니다.** 각 새 세션은 이전 세션의 대화 내역 없이 새로운 컨텍스트 윈도우로 시작합니다. Claude는 [자동 메모리](/docs/en/memory#auto-memory)를 사용하여 세션 간에 학습 내용을 유지할 수 있으며, [CLAUDE.md](/docs/en/memory)에 자체 지속적인 지침을 추가할 수 있습니다.

### 브랜치 간 작업

각 Claude Code 대화는 현재 디렉토리에 연결된 세션입니다. `/resume` 선택기는 기본적으로 현재 워크트리의 세션을 보여주며, 키보드 단축키를 통해 다른 워크트리나 프로젝트로 목록을 확장할 수 있습니다. 전체 선택기 단축키 및 이름 해제 방식은 [세션 관리](/docs/en/sessions#use-the-session-picker)를 참조하세요.

Claude는 현재 브랜치의 파일을 봅니다. 브랜치를 전환하면 Claude는 새 브랜치의 파일을 보지만 대화 내역은 그대로 유지됩니다. Claude는 전환 후에도 논의했던 내용을 기억합니다.

세션은 디렉토리에 연결되어 있으므로 개별 브랜치에 대해 별도의 디렉토리를 생성하는 [git worktrees](/docs/en/worktrees)를 사용하여 병렬 Claude 세션을 실행할 수 있습니다.

### 세션 재개 또는 포크

`claude --continue` 또는 `claude --resume`으로 세션을 재개하면 동일한 세션 ID 아래에서 다시 열리고 기존 대화에 새 메시지가 추가됩니다. `--fork-session` 또는 `/branch`로 포크하면 내역이 새 세션 ID로 복사되고 원본은 변경되지 않은 상태로 유지됩니다.

<img src="https://mintcdn.com/claude-code/ikqp3_70mqIahteV/images/session-continuity.svg?fit=max&auto=format&n=ikqp3_70mqIahteV&q=85&s=04ed0984a58e4127e05b3640265241a3" alt="Diagram of session continuity: resume continues the same session, fork creates a new branch with a new ID." width="560" height="280" data-path="images/session-continuity.svg" />

재개 플래그, `/resume` 선택기, 이름 지정 및 동일한 세션이 두 터미널에서 열려 있을 때 발생하는 상황은 [세션 관리](/docs/en/sessions)를 참조하세요.

### 컨텍스트 윈도우

Claude의 컨텍스트 윈도우에는 대화 내역, 파일 내용, 명령 출력, [CLAUDE.md](/docs/en/memory), [자동 메모리](/docs/en/memory#auto-memory), 로드된 스킬 및 시스템 지침이 담깁니다. 작업에 따라 컨텍스트가 채워집니다. Claude는 자동으로 압축을 수행하지만 대화 초기의 지침이 손실될 수 있습니다. 지속적인 규칙은 CLAUDE.md에 두고 `/context`를 실행하여 공간을 차지하는 요소를 확인하세요.

무엇이 언제 로드되는지에 대한 대화형 안내는 [컨텍스트 윈도우 살펴보기](/docs/en/context-window)를 참조하세요.

#### 컨텍스트가 찼을 때

Claude Code는 한계에 도달함에 따라 컨텍스트를 자동으로 관리합니다. 이전 도구 출력을 먼저 지운 다음 필요에 따라 대화를 요약합니다. 요청 사항과 주요 코드 조각은 보존되지만 대화 초기의 상세 지침은 손실될 수 있습니다. 대화 내역에 의존하기보다 지속적인 규칙은 CLAUDE.md에 작성하세요.

압축 시 보존할 내용을 제어하려면 CLAUDE.md에 "Compact Instructions" 섹션을 추가하거나 중점 사항과 함께 `/compact`를 실행하세요 (예: `/compact focus on the API changes`).

단일 파일이나 도구 출력이 너무 커서 각 요약 직후 컨텍스트가 다시 채워지는 경우, Claude Code는 몇 번의 시도 후 자동 압축을 중단하고 루프를 도는 대신 오류를 표시합니다. 복구 단계는 [Auto-compaction stops with a thrashing error](/docs/en/troubleshooting#auto-compaction-stops-with-a-thrashing-error)를 참조하세요.

`/context`를 실행하여 공간을 사용하는 항목을 확인하세요. MCP 도구 정의는 기본적으로 지연되며 [도구 검색](/docs/en/mcp#scale-with-mcp-tool-search)을 통해 요청 시 로드되므로, Claude가 특정 도구를 사용할 때까지는 도구 이름만 컨텍스트를 소비합니다. 서버별 비용을 확인하려면 `/mcp`를 실행하세요.

#### 스킬 및 서브에이전트로 컨텍스트 관리

압축 외에도 다른 기능을 사용하여 컨텍스트로 로드되는 항목을 제어할 수 있습니다.

[스킬](/docs/en/skills)은 요청 시 로드됩니다. Claude는 세션 시작 시 스킬 설명을 보지만 스킬을 사용할 때만 전체 내용이 로드됩니다. 수동으로 호출하는 스킬의 경우 `disable-model-invocation: true`를 설정하여 필요한 시점까지 설명을 컨텍스트에서 제외할 수 있습니다. 직접 작성하지 않은 스킬의 경우 설정에서 [`skillOverrides`](/docs/en/skills#override-skill-visibility-from-settings)를 사용하여 동일하게 처리할 수 있습니다.

[서브에이전트](/docs/en/sub-agents)는 메인 대화와 완전히 분리된 자체의 새로운 컨텍스트를 얻습니다. 해당 작업은 메인 컨텍스트를 비대하게 만들지 않습니다. 완료되면 요약을 반환합니다. 이러한 격리 덕분에 서브에이전트가 긴 세션에 도움이 됩니다.

각 기능의 비용은 [컨텍스트 비용](/docs/en/features-overview#understand-context-costs)을, 컨텍스트 관리에 대한 팁은 [토큰 사용량 줄이기](/docs/en/costs#reduce-token-usage)를 참조하세요.

## 체크포인트 및 권한으로 안전하게 작업하기

Claude에는 두 가지 안전 메커니즘이 있습니다: 체크포인트를 사용하면 파일 변경 사항을 취소할 수 있고, 권한을 사용하면 확인 없이 Claude가 수행할 수 있는 작업을 제어할 수 있습니다.

### 체크포인트로 변경 사항 취소

**파일 편집은 되돌릴 수 있습니다.** Claude가 파일을 편집하기 전에 현재 내용을 스냅샷으로 저장합니다. 문제가 발생하면 `Esc` 키를 두 번 눌러 이전 상태로 되돌리거나 Claude에게 취소를 요청하세요.

체크포인트는 git과 별개이며 대화를 재개할 때도 사용 가능 상태로 유지됩니다. 파일 변경 사항만 적용되며, 복원 시 [심볼릭 링크 및 하드 링크 파일은 건너끕니다](/docs/en/checkpointing#symlinked-and-hard-linked-paths-not-restored). 원격 시스템(데이터베이스, API, 배포)에 영향을 주는 작업은 체크포인트를 생성할 수 없으므로, Claude가 외부 부작용이 있는 명령을 실행하기 전에 사용자에게 확인을 요청하는 이유가 여기에 있습니다.

### Claude가 수행할 수 있는 작업 제어

`Shift+Tab`을 눌러 권한 모드를 순환 전환하세요:

* **Manual**: Claude가 파일 편집 및 셸 명령 전에 확인 요청
* **Accept edits**: Claude가 파일 편집 및 `mkdir`, `mv`와 같은 일반 파일 시스템 명령을 확인 없이 실행, 기타 명령은 여전히 확인 요청
* **Plan**: 소스 파일을 편집하지 않고 Claude가 코드베이스를 탐색하고 플랜을 제안
* **Auto**: 백그라운드 안전 검사와 함께 Claude가 모든 작업을 평가

`.claude/settings.json`에서 특정 명령을 허용하여 Claude가 매번 묻지 않도록 설정할 수도 있습니다.이는 `npm test` 또는 `git status`와 같이 신뢰할 수 있는 명령에 유용합니다. 설정 범위는 조직 전체 정책부터 개인 선호도까지 지정할 수 있습니다. 자세한 내용은 [권한](/docs/en/permissions)을 참조하세요.

***

## Claude Code와 효과적으로 작업하기

다음 팁은 Claude Code로부터 더 나은 결과를 얻는 데 도움이 됩니다.

### Claude Code에 도움 요청하기

Claude Code는 사용법을 설명해 줄 수 있습니다. "훅은 어떻게 설정하나요?" 또는 "CLAUDE.md를 구성하는 가장 좋은 방법은 무엇인가요?"와 같은 질문을 하면 Claude가 설명해 줍니다.

내장 명령도 설정을 안내합니다:

* `/init`은 프로젝트용 CLAUDE.md 생성을 안내합니다.
* `/doctor`는 설치 및 구성 문제를 진단하고 수정할 수 있는 설정 점검을 실행합니다.

### 대화 형식으로 진행하기

Claude Code는 대화형입니다. 완벽한 프롬프트가 필요하지 않습니다. 원하는 바로 시작한 다음 구체화하세요:

```text theme={null}
Fix the login bug
```

\[Claude가 조사하고 시도함]

```text theme={null}
That's not quite right. The issue is in the session handling.
```

\[Claude가 접근 방식 조정]

첫 번째 시도가 올바르지 않더라도 처음부터 다시 시작할 필요가 없습니다. 반복해서 개선해 나가세요.

#### 중단 및 조율

턴이 끝나기를 기다리거나 처음부터 다시 시작하지 않고도 언제든지 Claude의 방향을 재설정할 수 있습니다:

* **`Esc` 누르기**: Claude를 즉시 중단합니다. 실행 중인 도구 호출이 취소되고 Claude는 다음 지침을 기다립니다.
* **수정 사항을 입력하고 `Enter` 누르기**: 실행 중인 도구를 중단하지 않고 보냅니다. Claude는 현재 작업이 완료되는 즉시 이를 읽고 다음 단계를 결정하기 전에 조정합니다.

### 처음부터 구체적으로 작성하기

초기 프롬프트가 정밀할수록 수정 횟수가 줄어듭니다. 특정 파일을 참조하고, 제약 조건을 언급하며, 예시 패턴을 제시하세요.

```text theme={null}
The checkout flow is broken for users with expired cards.
Check src/payments/ for the issue, especially token refresh.
Write a failing test first, then fix it.
```

막연한 프롬프트도 동작하지만 방향을 잡는 데 더 많은 시간을 소비하게 됩니다. 위와 같이 구체적인 프롬프트는 첫 번째 시도에서 성공하는 경우가 많습니다.

### Claude에게 검증할 수 있는 기준 제공하기

Claude는 스스로 작업을 검증할 수 있을 때 더 나은 성능을 냅니다. 테스트 케이스를 포함하거나, 예상 UI 스크린샷을 붙여넣거나, 원하는 출력을 정의하세요.

```text theme={null}
Implement validateEmail. Test cases: 'user@example.com' → true,
'invalid' → false, 'user@.com' → false. Run the tests after.
```

시각적 작업의 경우 디자인 스크린샷을 붙여넣고 구현 결과를 이에 비교하도록 Claude에게 요청하세요.

### 구현 전에 탐색하기

복잡한 문제의 경우 코딩과 조사를 분리하세요. 플랜 모드(`Shift+Tab` 두 번)를 사용하여 먼저 코드베이스를 분석하세요:

```text theme={null}
Read src/auth/ and understand how we handle sessions.
Then create a plan for adding OAuth support.
```

플랜을 검토하고 대화를 통해 다듬은 후 Claude가 구현하도록 하세요. 이 2단계 접근 방식은 코드로 바로 뛰어드는 것보다 더 나은 결과를 만들어 냅니다.

### 지시하지 말고 위임하기

유능한 동료에게 위임한다고 생각하세요. 컨텍스트와 방향성을 제공한 다음 Claude가 세부 사항을 파악하도록 신뢰하세요:

```text theme={null}
The checkout flow is broken for users with expired cards.
The relevant code is in src/payments/. Can you investigate and fix it?
```

읽어야 할 파일이나 실행할 명령을 지정할 필요가 없습니다. Claude가 이를 알아서 파악합니다.

## 다음 단계

<CardGroup cols={2}>
  <Card title="기능으로 확장" icon="puzzle-piece" href="/docs/en/features-overview">
    스킬, MCP 연결 및 커스텀 명령 추가
  </Card>

  <Card title="일반적인 워크플로" icon="graduation-cap" href="/docs/en/common-workflows">
    일반적인 작업을 위한 단계별 가이드
  </Card>
</CardGroup>
