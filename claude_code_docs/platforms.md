> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 플랫폼 및 통합 (Platforms and integrations)

> 어디서 Claude Code를 실행하고 무엇에 연결할지 선택하세요. CLI, 데스크톱, VS Code, JetBrains, 웹, 모바일과 Chrome, Slack, CI/CD와 같은 통합 기능을 비교해 봅니다.

Claude Code는 모든 곳에서 동일한 기본 엔진을 실행하지만, 각 사용 환경(surface)은 서로 다른 작업 방식에 맞게 조정되어 있습니다. 이 페이지는 사용자의 워크플로에 맞는 적절한 플랫폼을 선택하고 이미 사용 중인 도구를 연결하는 데 도움을 줍니다.

## 어디서 Claude Code를 실행할 것인가

선호하는 작업 방식과 프로젝트의 위치에 따라 플랫폼을 선택하세요.

| 플랫폼 | 최적의 용도 | 제공하는 기능 |
| :--- | :--- | :--- |
| [CLI](/docs/en/quickstart) | 터미널 워크플로, 스크립팅, 원격 서버 | 전체 기능 세트, [Agent SDK](/docs/en/headless), macOS(Pro 및 Max)에서의 [computer use](/docs/en/computer-use), 서드파티 공급자 |
| [Desktop](/docs/en/desktop) | 시각적 검토, 병렬 세션, 관리형 설정 | Diff 뷰어, 앱 미리보기, Pro 및 Max의 [computer use](/docs/en/desktop#let-claude-use-your-computer) 및 [Dispatch](/docs/en/desktop#sessions-from-dispatch) |
| [VS Code](/docs/en/vs-code) | 터미널로 전환하지 않고 VS Code 내부에서 작업 | 인라인 diff, 통합 터미널, 파일 컨텍스트 |
| [JetBrains](/docs/en/jetbrains) | IntelliJ, PyCharm, WebStorm 또는 기타 JetBrains IDE 내부에서 작업 | Diff 뷰어, 선택 영역 공유, 터미널 세션 |
| [Web](/docs/en/claude-code-on-the-web) | 조율이 많이 필요하지 않은 장기 작업, 또는 오프라인 상태에서도 계속되어야 하는 작업 | Anthropic 관리형 클라우드, 연결 해제 후에도 지속됨 |
| [Mobile](/docs/en/mobile) | 컴퓨터를 비운 동안 작업 시작 및 모니터링 | iOS 및 Android용 Claude 앱에서의 클라우드 세션, 로컬 세션용 [Remote Control](/docs/en/remote-control), Pro 및 Max의 Desktop용 [Dispatch](/docs/en/desktop#sessions-from-dispatch) |

CLI는 터미널 기본 작업에 가장 포괄적인 환경입니다: 스크립팅 및 Agent SDK는 CLI 전용입니다. 서드파티 공급자는 [VS Code](/docs/en/vs-code#use-third-party-providers)에서도 작동합니다. 엔터프라이즈 [Desktop](/docs/en/desktop) 배포는 Google Cloud Agent Platform을 지원하며 Desktop은 [게이트웨이 공급자](/docs/en/llm-gateway-connect#desktop-app)를 지원합니다. Amazon Bedrock 또는 Microsoft Foundry의 경우 CLI나 VS Code 또는 해당 공급자에서 Code 탭을 실행하는 [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview)를 사용하세요. Desktop 및 IDE 확장 프로그램은 시각적 검토 및 에디터와의 더 밀접한 통합을 위해 일부 CLI 전용 기능을 교환합니다. 웹은 Anthropic의 클라우드에서 실행되므로 연결을 해제한 후에도 작업이 계속됩니다. 모바일은 이러한 동일한 클라우드 세션이나 Remote Control을 통한 로컬 세션으로 접속하는 씬 클라이언트(thin client)이며, Dispatch를 사용하여 데스크톱에 작업을 전송할 수 있습니다.

동일한 프로젝트에서 여러 환경을 함께 사용할 수 있습니다. 구성, 프로젝트 메모리, MCP 서버는 로컬 환경 전반에 걸쳐 공유됩니다.

## 도구 연결

통합 기능을 사용하면 Claude가 코드베이스 외부의 서비스와 작동할 수 있습니다.

| 통합 | 기능 | 용도 |
| :--- | :--- | :--- |
| [Chrome](/docs/en/chrome) | 로그인한 세션으로 브라우저 제어 | 웹 앱 테스트, 양식 작성, API 없는 사이트 자동화 |
| [GitHub Actions](/docs/en/github-actions) | CI 파이프라인에서 Claude 실행 | 자동화된 PR 리뷰, 이슈 분류, 정기 유지 보수 |
| [GitLab CI/CD](/docs/en/gitlab-ci-cd) | GitLab용 GitHub Actions와 동일 | GitLab 기반 CI 구동 자동화 |
| [Code Review](/docs/en/code-review) | 모든 PR을 자동으로 리뷰 | 사람의 검토 전에 버그 포착 |
| [Slack](/docs/en/slack) | 채널의 `@Claude` 멘션에 응답 | 팀 채팅에서 버그 리포트를 풀 리퀘스트로 전환 |

여기 나열되지 않은 통합의 경우 [MCP 서버](/docs/en/mcp) 및 [커넥터](/docs/en/desktop#connect-external-tools)를 통해 Linear, Notion, Google Drive 또는 자체 내부 API 등 거반 모든 것을 연결할 수 있습니다.

## 터미널 자리를 비웠을 때 작업하기

Claude Code는 터미널에 없을 때 작업할 수 있는 여러 방법을 제공합니다. 작업 트리거 요소, Claude가 실행되는 위치, 설정해야 하는 작업량에 차이가 있습니다.

| | 트리거 | Claude 실행 위치 | 설정 | 최적의 용도 |
| :--- | :--- | :--- | :--- | :--- |
| [Dispatch](/docs/en/desktop#sessions-from-dispatch) | Claude 모바일 앱에서 작업 메시지 전송 | 사용자의 머신 (Desktop) | [모바일 앱을 Desktop과 페어링](https://support.claude.com/en/articles/13947068) | 자리를 비운 동안 최소한의 설정으로 작업 위임 |
| [Remote Control](/docs/en/remote-control) | [claude.ai/code](https://claude.ai/code) 또는 Claude 모바일 앱에서 실행 중인 세션 제어 | 사용자의 머신 (CLI 또는 VS Code) | `claude remote-control` 실행 | 다른 장치에서 진행 중인 작업 제어 |
| [Channels](/docs/en/channels) | Telegram, Discord 등의 채팅 앱이나 자체 서버에서 이벤트 푸시 | 사용자의 머신 (CLI) | [채널 플러그인 설치](/docs/en/channels#quickstart) 또는 [자체 구축](/docs/en/channels-reference) | CI 실패나 채팅 메시지와 같은 외부 이벤트에 대응 |
| [Slack](/docs/en/slack) | 팀 채널에서 `@Claude` 멘션 | Anthropic 클라우드 | [웹에서의 Claude Code](/docs/en/claude-code-on-the-web)가 활성화된 상태에서 [Slack 앱 설치](/docs/en/slack#setting-up-claude-code-in-slack) | 팀 채팅 기반 PR 및 리뷰 |
| [예약 작업](/docs/en/scheduled-tasks) | 일정 설정 | [CLI](/docs/en/scheduled-tasks), [Desktop](/docs/en/desktop-scheduled-tasks), 또는 [클라우드](/docs/en/routines) | 주기 선택 | 일일 리뷰와 같은 반복적인 자동화 |

어디서부터 시작해야 할지 모르겠다면 [CLI를 설치](/docs/en/quickstart)하고 프로젝트 디렉터리에서 실행하세요. 터미널 사용을 선호하지 않는 경우 [Desktop](/docs/en/desktop-quickstart)은 그래픽 인터페이스와 함께 동일한 엔진을 제공합니다.

## 관련 정보

### 플랫폼

* [CLI 빠른 시작](/docs/en/quickstart): 터미널에 첫 번째 명령 설치 및 실행
* [Desktop](/docs/en/desktop): 시각적 diff 검토, 병렬 세션, computer use 및 Dispatch
* [VS Code](/docs/en/vs-code): 에디터 내부의 Claude Code 확장 프로그램
* [JetBrains](/docs/en/jetbrains): IntelliJ, PyCharm 및 기타 JetBrains IDE용 확장 프로그램
* [웹에서의 Claude Code](/docs/en/claude-code-on-the-web): 연결 해제 후에도 계속 실행되는 클라우드 세션
* [Mobile](/docs/en/mobile): 컴퓨터를 비운 동안 작업을 시작하고 모니터링하기 위한 [iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 및 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 Claude 앱

### 통합

* [Chrome](/docs/en/chrome): 로그인한 세션으로 브라우저 작업 자동화
* [Computer use](/docs/en/computer-use): Claude가 macOS에서 앱을 열고 화면을 제어하도록 허용
* [GitHub Actions](/docs/en/github-actions): CI 파이프라인에서 Claude 실행
* [GitLab CI/CD](/docs/en/gitlab-ci-cd): GitLab용 동등한 기능
* [Code Review](/docs/en/code-review): 모든 풀 리퀘스트에 대한 자동 리뷰
* [Slack](/docs/en/slack): 팀 채팅에서 작업을 보내고 PR 다시 받기

### 원격 액세스

* [Dispatch](/docs/en/desktop#sessions-from-dispatch): 휴대폰에서 메시지로 작업을 전달하여 데스크톱 세션 생성
* [Remote Control](/docs/en/remote-control): 휴대폰이나 브라우저에서 실행 중인 세션 제어
* [Channels](/docs/en/channels): 채팅 앱이나 자체 서버의 이벤트를 세션으로 푸시
* [예약 작업](/docs/en/scheduled-tasks): 반복적인 일정에 따라 프롬프트 실행
