> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 개요 (Overview)

> Claude Code는 코드베이스를 읽고, 파일을 편집하고, 명령을 실행하며, 개발 도구와 통합되는 에이전틱(agentic) 코딩 도구입니다. 터미널, IDE, 데스크톱 앱, 브라우저에서 사용할 수 있습니다.

Claude Code는 기능 구현, 버그 수정, 개발 작업 자동화를 지원하는 AI 기반 코딩 어시스턴트입니다. 전체 코드베이스를 이해하고 여러 파일과 도구에 걸쳐 작업을 수행할 수 있습니다.

## 시작하기

Claude Code는 터미널, IDE 확장 프로그램, 데스크톱 앱, 웹 등 여러 환경에서 실행됩니다. 아래 탭에서 하나를 선택하여 시작하세요. 대부분의 환경에서는 [Claude 구독](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=overview_pricing) 또는 [Anthropic Console](https://console.anthropic.com/) 계정이 필요합니다. 터미널 CLI 및 VS Code는 [서드파티 공급자](/docs/en/third-party-integrations)도 지원합니다.

<Tabs>
  <Tab title="터미널">
    터미널에서 직접 Claude Code와 작업할 수 있는 풀기능 CLI입니다. 명령줄에서 파일을 편집하고, 명령을 실행하며, 전체 프로젝트를 관리할 수 있습니다.

    Claude Code를 설치하려면 다음 방법 중 하나를 사용하세요:

    <Tabs>
      <Tab title="네이티브 설치 (권장)">
        **macOS, Linux, WSL:**

        ```bash theme={null}
        curl -fsSL https://claude.ai/install.sh | bash
        ```

        **Windows PowerShell:**

        ```powershell theme={null}
        irm https://claude.ai/install.ps1 | iex
        ```

        **Windows CMD:**

        ```batch theme={null}
        curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
        ```

        `The token '&&' is not a valid statement separator`라는 메시지가 표시되면 CMD가 아닌 PowerShell에 있는 것입니다. `'irm' is not recognized as an internal or external command`라는 메시지가 표시되면 PowerShell이 아닌 CMD에 있는 것입니다. PowerShell에 있을 때는 프롬프트에 `PS C:\`가 표시되고, CMD에 있을 때는 `PS` 없이 `C:\`가 표시됩니다.

        설치 명령이 `syntax error near unexpected token '<'`, `403` 또는 기타 curl 오류로 실패하는 경우, [설치 문제 해결](/docs/en/troubleshoot-install#find-your-error)을 참조하여 오류에 맞는 해결 방법을 찾거나 대안 설치 방법을 확인하세요.

        네이티브 Windows에서는 Claude Code가 Bash 도구를 사용할 수 있도록 [Git for Windows](https://git-scm.com/downloads/win) 설치를 권장합니다. Git for Windows가 설치되어 있지 않은 경우 Claude Code는 셸 도구로 PowerShell을 대신 사용합니다. WSL 설정에는 Git for Windows가 필요하지 않습니다.

        <Info>
          네이티브 설치는 백그라운드에서 자동으로 업데이트되어 항상 최신 버전을 유지합니다.
        </Info>
      </Tab>

      <Tab title="Homebrew">
        ```bash theme={null}
        brew install --cask claude-code
        ```

        Homebrew는 두 가지 cask를 제공합니다. `claude-code`는 일반적으로 약 1주일 뒤처지고 주요 리그레션이 발생한 릴리스를 건너뛰는 안정(stable) 릴리스 채널을 추적합니다. `claude-code@latest`는 최신(latest) 채널을 추적하며 새 버전이 출시되는 즉시 업데이트를 받습니다.

        <Info>
          Homebrew 설치는 자동 업데이트되지 않습니다. 최신 기능과 보안 패치를 받으려면 설치한 cask에 따라 `brew upgrade claude-code` 또는 `brew upgrade claude-code@latest`를 실행하세요.
        </Info>
      </Tab>

      <Tab title="WinGet">
        ```powershell theme={null}
        winget install Anthropic.ClaudeCode
        ```

        <Info>
          WinGet 설치는 자동 업데이트되지 않습니다. 최신 기능과 보안 패치를 받으려면 주기적으로 `winget upgrade Anthropic.ClaudeCode`를 실행하세요.
        </Info>
      </Tab>
    </Tabs>

    Debian, Fedora, RHEL, Alpine에서는 [apt, dnf 또는 apk](/docs/en/setup#install-with-linux-package-managers)로 설치할 수도 있습니다.

    그런 다음 프로젝트 디렉터리에서 Claude Code를 시작하세요. `your-project`를 시스템의 프로젝트 디렉터리 경로로 대체합니다:

    ```bash theme={null}
    cd your-project
    claude
    ```

    처음 사용할 때 로그인하라는 프롬프트가 표시됩니다. 그것으로 끝입니다! [빠른 시작 계속하기 →](/docs/en/quickstart)

    <Tip>
      설치 옵션, 수동 업데이트 또는 삭제 지침은 [고급 설정](/docs/en/setup)을 참조하세요. 문제가 발생하면 [설치 문제 해결](/docs/en/troubleshoot-install)을 방문하세요.
    </Tip>
  </Tab>

  <Tab title="VS Code">
    VS Code 확장 프로그램은 인라인 diff, @-mentions, 계획 검토, 대화 기록을 에디터에서 직접 제공합니다.

    * [VS Code용 설치](vscode:extension/anthropic.claude-code)
    * [Cursor용 설치](cursor:extension/anthropic.claude-code)

    또는 확장 보기(Mac: `Cmd+Shift+X`, Windows/Linux: `Ctrl+Shift+X`)에서 "Claude Code"를 검색하세요. 설치 후 명령 팔레트(`Cmd+Shift+P` / `Ctrl+Shift+P`)를 열고 "Claude Code"를 입력한 다음 **Open in New Tab**을 선택합니다.

    [VS Code로 시작하기 →](/docs/en/vs-code#get-started)
  </Tab>

  <Tab title="데스크톱 앱">
    IDE나 터미널 외부에서 Claude Code를 실행하기 위한 독립형 앱입니다. diff를 시각적으로 검토하고, 여러 세션을 나란히 실행하며, 반복 작업을 예약하고, 클라우드 세션을 시작할 수 있습니다.

    다운로드 및 설치:

    * [macOS](https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect?utm_source=claude_code\&utm_medium=docs) (Intel 및 Apple Silicon)
    * [Windows](https://claude.ai/api/desktop/win32/x64/setup/latest/redirect?utm_source=claude_code\&utm_medium=docs) (x64)
    * [Windows ARM64](https://claude.ai/api/desktop/win32/arm64/setup/latest/redirect?utm_source=claude_code\&utm_medium=docs)

    설치 후 Claude를 실행하고 로그인한 다음 **Code** 탭을 클릭하여 코딩을 시작하세요. [유료 구독](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=overview_desktop_pricing)이 필요합니다.

    [데스크톱 앱 알아보기 →](/docs/en/desktop-quickstart)
  </Tab>

  <Tab title="웹">
    로컬 설정 없이 브라우저에서 Claude Code를 실행하세요. 시간이 오래 걸리는 작업을 시작하고 완료 후 돌아오거나, 로컬에 없는 리포지토리에서 작업하거나, 여러 작업을 병렬로 실행할 수 있습니다. 데스크톱 브라우저 및 [iOS/Android용 Claude 앱](/docs/en/mobile)에서 사용할 수 있습니다.

    [claude.ai/code](https://claude.ai/code)에서 코딩을 시작하세요.

    [웹에서 시작하기 →](/docs/en/web-quickstart)
  </Tab>

  <Tab title="JetBrains">
    대화형 diff 보기 및 선택 컨텍스트 공유 기능이 포함된 IntelliJ IDEA, PyCharm, WebStorm 및 기타 JetBrains IDE용 플러그인입니다.

    JetBrains Marketplace에서 [Claude Code 플러그인](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-)을 설치하고 IDE를 다시 시작하세요. 이 플러그인에는 별도로 설치된 Claude Code CLI가 필요합니다. [JetBrains 설정 단계](/docs/en/jetbrains#installation)를 참조하세요.

    [JetBrains로 시작하기 →](/docs/en/jetbrains)
  </Tab>
</Tabs>

## 할 수 있는 작업

Claude Code를 사용할 수 있는 몇 가지 방법은 다음과 같습니다:

<AccordionGroup>
  <Accordion title="미뤄두었던 작업 자동화" icon="wand-magic-sparkles">
    Claude Code는 지루한 작업을 대신 처리합니다: 테스트되지 않은 코드에 대한 테스트 작성, 프로젝트 전반의 린트 오류 수정, 머지 충돌 해결, 종속성 업데이트, 릴리스 노트 작성 등.

    ```bash theme={null}
    claude "write tests for the auth module, run them, and fix any failures"
    ```
  </Accordion>

  <Accordion title="기능 구축 및 버그 수정" icon="hammer">
    원하는 바를 자연어로 설명하세요. Claude Code가 접근 방식을 계획하고, 여러 파일에 걸쳐 코드를 작성하며, 정상 작동을 확인합니다.

    버그의 경우 오류 메시지를 붙여넣거나 증상을 설명하세요. Claude Code가 코드베이스에서 문제를 추적하고 근본 원인을 파악하여 수정을 구현합니다. 자세한 예시는 [공통 워크플로](/docs/en/common-workflows)를 참조하세요.
  </Accordion>

  <Accordion title="커밋 및 풀 리퀘스트 생성" icon="code-branch">
    Claude Code는 git과 직접 작동합니다. 변경 사항을 스테이징하고, 커밋 메시지를 작성하며, 브랜치를 생성하고, 풀 리퀘스트를 엽니다.

    ```bash theme={null}
    claude "commit my changes with a descriptive message"
    ```

    CI에서는 [GitHub Actions](/docs/en/github-actions) 또는 [GitLab CI/CD](/docs/en/gitlab-ci-cd)를 사용하여 코드 리뷰 및 이슈 분류를 자동화할 수 있습니다.
  </Accordion>

  <Accordion title="MCP로 도구 연결" icon="plug">
    [Model Context Protocol (MCP)](/docs/en/mcp)은 AI 도구를 외부 데이터 소스에 연결하기 위한 개방형 표준입니다. MCP를 사용하면 Claude Code가 Google Drive의 디자인 문서를 읽고, Jira의 티켓을 업데이트하고, Slack에서 데이터를 가져오거나, 커스텀 도구를 사용할 수 있습니다. [MCP 빠른 시작](/docs/en/mcp-quickstart)에서 첫 번째 서버를 종단 간 연결해보세요.
  </Accordion>

  <Accordion title="지침, 스킬, 훅으로 맞춤 설정" icon="sliders">
    [`CLAUDE.md`](/docs/en/memory)는 프로젝트 루트에 추가하는 Markdown 파일로, Claude Code가 모든 세션 시작 시 읽습니다. 코딩 표준, 아키텍처 결정 사항, 선호하는 라이브러리, 리뷰 체크리스트를 설정하는 데 사용하세요. 또한 Claude는 작업하면서 [자동 메모리(auto memory)](/docs/en/memory#auto-memory)를 작성하여, 별도로 작성하지 않아도 세션 전반에 걸쳐 빌드 명령 및 디버깅 인사이트와 같은 학습 내용을 저장합니다.

    `/review-pr`이나 `/deploy-staging`처럼 팀이 공유할 수 있는 반복 가능한 워크플로를 패키징하려면 [스킬](/docs/en/skills)을 만드세요.

    [훅](/docs/en/hooks)을 사용하면 모든 파일 편집 후 자동 포맷팅이나 커밋 전 린트 실행과 같이 Claude Code 작업 전후에 셸 명령을 실행할 수 있습니다.
  </Accordion>

  <Accordion title="에이전트 팀 실행 및 커스텀 에이전트 구축" icon="users">
    작업의 서로 다른 부분에서 동시에 작업하는 [여러 Claude Code 에이전트](/docs/en/sub-agents)를 생성하세요. 리드 에이전트가 작업을 조율하고, 하위 작업을 할당하며, 결과를 병합합니다.

    여러 전체 세션을 병렬로 실행하고 하나의 화면에서 관찰하려면 [백그라운드 에이전트](/docs/en/agent-view)를 사용하세요. 완전한 커스텀 워크플로의 경우 [Agent SDK](/docs/en/agent-sdk/overview)를 사용하여 오케스트레이션, 도구 액세스, 권한을 완벽히 제어하면서 Claude Code의 도구와 기능으로 구동되는 자체 에이전트를 구축할 수 있습니다.
  </Accordion>

  <Accordion title="CLI로 파이프, 스크립트 작성 및 자동화" icon="terminal">
    Claude Code는 조합 가능하며 유닉스 철학을 따릅니다. 로그를 파이프 처리하거나, CI에서 실행하거나, 다른 도구와 체이닝하세요:

    ```bash theme={null}
    # 최근 로그 출력 분석
    tail -200 app.log | claude -p "Slack me if you see any anomalies"

    # CI에서 번역 자동화
    claude -p "translate new strings into French and raise a PR for review"

    # 여러 파일에 걸친 일괄 작업
    git diff main --name-only | claude -p "review these changed files for security issues"
    ```

    전체 명령 및 플래그 세트는 [CLI 참조](/docs/en/cli-reference)를 참조하세요.
  </Accordion>

  <Accordion title="반복 작업 예약" icon="clock">
    아침 PR 리뷰, 야간 CI 실패 분석, 주간 종속성 감사, PR 머지 후 문서 동기화 등 반복되는 작업을 자동화하도록 Claude를 일정에 따라 실행하세요.

    * [루틴(Routines)](/docs/en/routines)은 Anthropic이 관리하는 인프라에서 실행되므로 컴퓨터가 꺼져 있어도 계속 실행됩니다. API 호출이나 GitHub 이벤트로 트리거할 수도 있습니다. 웹, 데스크톱 앱 또는 CLI에서 `/schedule`을 실행하여 생성하세요.
    * [데스크톱 예약 작업](/docs/en/desktop-scheduled-tasks)은 로컬 파일 및 도구에 직접 접근하여 로컬 머신에서 실행됩니다.
    * [`/loop`](/docs/en/scheduled-tasks)는 빠른 폴링을 위해 CLI 세션 내에서 프롬프트를 반복합니다.
  </Accordion>

  <Accordion title="어디서나 작업" icon="globe">
    세션은 단일 환경에 고정되지 않습니다. 컨텍스트가 변경됨에 따라 환경 간에 작업을 이동할 수 있습니다:

    * [Remote Control](/docs/en/remote-control)을 사용하여 자리를 비우고 휴대폰이나 모든 브라우저에서 작업을 계속하세요.
    * 휴대폰에서 [Dispatch](/docs/en/desktop#sessions-from-dispatch)로 작업을 메시지로 보내고 생성된 데스크톱 세션을 여세요.
    * [웹](/docs/en/claude-code-on-the-web) 또는 [Claude 모바일 앱](/docs/en/mobile)에서 실행 시간이 긴 작업을 시작한 다음 `claude --teleport`로 터미널에 불러오세요. 텔레포트에는 claude.ai 구독이 필요합니다.
    * `/desktop`을 실행하여 시각적으로 diff를 검토할 수 있는 [데스크톱 앱](/docs/en/desktop)에서 현재 터미널 세션을 계속하세요. macOS 및 x64 Windows에서 사용할 수 있습니다.
    * 팀 채팅에서 작업 라우팅: [Slack](/docs/en/slack)에서 버그 리포트와 함께 `@Claude`를 멘션하고 풀 리퀘스트를 다시 받으세요.
  </Accordion>
</AccordionGroup>

## 어디서나 Claude Code 사용

각 [사용 환경(surface)](/docs/en/glossary#surface)은 동일한 Claude Code 엔진에 연결되므로 CLAUDE.md 파일, 설정 및 MCP 서버가 모든 환경에서 동등하게 작동합니다.

위의 [터미널](/docs/en/quickstart), [VS Code](/docs/en/vs-code), [JetBrains](/docs/en/jetbrains), [데스크톱](/docs/en/desktop), [웹](/docs/en/claude-code-on-the-web) 환경 외에도 Claude Code는 CI/CD, 채팅 및 브라우저 워크플로와 통합됩니다:

| 하고 싶은 작업... | 최선의 옵션 |
| --- | --- |
| 휴대폰이나 다른 장치에서 로컬 세션 계속하기 | [Remote Control](/docs/en/remote-control) |
| Telegram, Discord, iMessage 또는 자체 웹훅의 이벤트를 세션으로 푸시하기 | [채널(Channels)](/docs/en/channels) |
| 로컬에서 작업 시작 후 모바일에서 계속하기 | [`claude --cloud`](/docs/en/claude-code-on-the-web#from-terminal-to-web) 후 [Claude 모바일 앱](/docs/en/mobile) |
| 반복적인 일정으로 Claude 실행하기 | [루틴(Routines)](/docs/en/routines) 또는 [데스크톱 예약 작업](/docs/en/desktop-scheduled-tasks) |
| PR 리뷰 및 이슈 분류 자동화 | [GitHub Actions](/docs/en/github-actions) 또는 [GitLab CI/CD](/docs/en/gitlab-ci-cd) |
| 모든 PR에서 자동 코드 리뷰 받기 | [GitHub Code Review](/docs/en/code-review) |
| Slack에서 풀 리퀘스트로 버그 리포트 라우팅 | [Slack](/docs/en/slack) |
| 라이브 웹 애플리케이션 디버깅 | [Chrome](/docs/en/chrome) |
| 자체 워크플로를 위한 커스텀 에이전트 구축 | [Agent SDK](/docs/en/agent-sdk/overview) |

## 다음 단계

Claude Code를 설치한 후 이 가이드들을 통해 더 깊이 알아보세요.

* [빠른 시작](/docs/en/quickstart): 코드베이스 탐색부터 수정 사항 커밋까지 첫 번째 실제 작업 진행해보기
* [지침 및 메모리 저장](/docs/en/memory): CLAUDE.md 파일 및 자동 메모리를 통해 Claude에게 영구적인 지침 제공하기
* [공통 워크플로](/docs/en/common-workflows) 및 [모범 사례](/docs/en/best-practices): Claude Code를 최대한 활용하기 위한 패턴
* [모든 작업을 위한 하네스](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code): Claude Code 팀이 대규모 서브에이전트 오케스트레이션을 위해 [동적 워크플로](/docs/en/workflows)를 사용하는 방법
* [설정](/docs/en/settings): 사용자의 워크플로에 맞게 Claude Code 구성하기
* [문제 해결](/docs/en/troubleshooting): 일반적인 문제 해결 방법
* [code.claude.com](https://code.claude.com/): 데모, 요금제, 제품 세부 정보
