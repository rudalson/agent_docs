> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 빠른 시작 (Quickstart)

> Claude Code에 오신 것을 환영합니다!

이 빠른 시작 가이드를 통해 몇 분 안에 AI 기반 코딩 지원을 시작할 수 있습니다. 가이드를 마치면 일반적인 개발 작업에 Claude Code를 활용하는 방법을 이해하게 될 것입니다.

## 시작하기 전에

다음이 준비되어 있는지 확인하세요:

* 터미널 또는 명령 프롬프트 창
  * 터미널을 사용해 본 적이 없다면 [터미널 가이드](/docs/en/terminal-guide)를 확인하세요.
* 작업할 코드 프로젝트
* [Claude 구독](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=quickstart_prereq) (Pro, Max, Team, 또는 Enterprise), [Claude Console](https://console.anthropic.com/) 계정, 또는 [지원되는 클라우드 프로바이더](/docs/en/third-party-integrations)를 통한 접근 권한

<Note>
  이 가이드는 터미널 CLI를 다룹니다. Claude Code는 [웹](https://claude.ai/code), [데스크톱 앱](/docs/en/desktop), [VS Code](/docs/en/vs-code) 및 [JetBrains IDE](/docs/en/jetbrains), [Slack](/docs/en/slack), 그리고 [GitHub Actions](/docs/en/github-actions) 및 [GitLab](/docs/en/gitlab-ci-cd)을 통한 CI/CD 환경에서도 사용할 수 있습니다. [모든 인터페이스 보기](/docs/en/overview#use-claude-code-everywhere)를 참조하세요.
</Note>

## 1단계: Claude Code 설치

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

    `The token '&&' is not a valid statement separator` 오류가 보이면 CMD가 아닌 PowerShell 환경입니다. `'irm' is not recognized as an internal or external command` 오류가 보이면 PowerShell이 아닌 CMD 환경입니다. 프롬프트가 `PS C:\`로 표시되면 PowerShell이고 `PS` 없이 `C:\`로 표시되면 CMD입니다.

    설치 명령이 `syntax error near unexpected token '<'`, `403`, 또는 기타 curl 오류로 실패하는 경우 [설치 문제 해결](/docs/en/troubleshoot-install#find-your-error)을 참조하여 오류에 맞는 해결책과 대체 설치 방법을 확인하세요.

    네이티브 Windows 환경에서는 Claude Code가 Bash 도구를 사용할 수 있도록 [Git for Windows](https://git-scm.com/downloads/win) 설치를 권장합니다. Git for Windows가 설치되어 있지 않으면 Claude Code가 셸 도구로 PowerShell을 대신 사용합니다. WSL 설정에서는 Git for Windows가 필요하지 않습니다.

    <Info>
      네이티브 설치는 최신 버전을 유지할 수 있도록 백그라운드에서 자동으로 업데이트됩니다.
    </Info>
  </Tab>

  <Tab title="Homebrew">
    ```bash theme={null}
    brew install --cask claude-code
    ```

    Homebrew는 두 가지 카스크(cask)를 제공합니다. `claude-code`는 안정적인 릴리스 채널을 추적하며, 보통 일주일 정도 늦고 주요 리그레션이 있는 릴리스는 건너뜁니다. `claude-code@latest`는 최신 채널을 추적하며 새 버전이 배포되는 즉시 반영합니다.

    <Info>
      Homebrew 설치 방식은 자동 업데이트되지 않습니다. 최신 기능과 보안 패치를 받으려면 설치한 카스크에 따라 `brew upgrade claude-code` 또는 `brew upgrade claude-code@latest`를 실행하세요.
    </Info>
  </Tab>

  <Tab title="WinGet">
    ```powershell theme={null}
    winget install Anthropic.ClaudeCode
    ```

    <Info>
      WinGet 설치 방식은 자동 업데이트되지 않습니다. 최신 기능과 보안 패치를 받으려면 주기적으로 `winget upgrade Anthropic.ClaudeCode`를 실행하세요.
    </Info>
  </Tab>
</Tabs>

Debian, Fedora, RHEL, Alpine 환경에서는 [apt, dnf, 또는 apk](/docs/en/setup#install-with-linux-package-managers)를 통해 설치할 수도 있습니다.

설치가 정상적으로 완료되었는지 확인하려면 다음을 실행하세요:

```bash theme={null}
claude --version
```

명령이 버전 번호 뒤에 `(Claude Code)`를 출력합니다.

## 2단계: 계정 로그인

Claude Code를 사용하려면 계정이 필요합니다. `claude` 명령으로 대화형 세션을 시작하면 첫 사용 시 로그인 프롬프트가 표시됩니다:

```bash theme={null}
claude
```

Claude 구독 또는 Console 계정의 경우 프롬프트 안내에 따라 브라우저에서 인증을 완료하세요. 나중에 계정을 전환하거나 다시 인증하려면 실행 중인 세션 내부에서 `/login`을 입력하세요:

```text theme={null}
/login
```

다음 계정 유형 중 하나를 사용하여 로그인할 수 있습니다:

* [Claude Pro, Max, Team, 또는 Enterprise](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=quickstart_login) (권장)
* [Claude Console](https://console.anthropic.com/) (선불 크레딧 기반 API 접근). 첫 로그인 시 비용을 중앙에서 추적할 수 있도록 Console에 "Claude Code" 워크스페이스가 자동으로 생성됩니다.
* [Amazon Bedrock, Google Cloud's Agent Platform, 또는 Microsoft Foundry](/docs/en/third-party-integrations) (엔터프라이즈 클라우드 프로바이더)
* 조직에서 운영 중인 자체 호스팅 [Claude apps gateway](/docs/en/claude-apps-gateway): 관리자가 게이트웨이 URL을 사전 구성해 두면 `/login` 실행 시 **Cloud gateway** 화면으로 바로 연결되어 사내 SSO로 로그인할 수 있습니다.

로그인하고 나면 자격 증명이 저장되어 다시 로그인할 필요가 없습니다.

## 3단계: 첫 세션 시작하기

임의의 프로젝트 디렉토리에서 터미널을 열고 Claude Code를 시작하세요:

```bash theme={null}
cd /path/to/your/project
claude
```

`/path/to/your/project`를 작업하려는 프로젝트의 경로로 대체하세요.

상단에 버전, 현재 모델, 작업 디렉토리가 표시되는 Claude Code 프롬프트를 보게 됩니다. 사용 가능한 명령을 확인하려면 `/help`를, 이전 대화를 이어가려면 `/resume`를 입력하세요.

<Tip>
  로그인(2단계) 후 자격 증명은 시스템에 저장됩니다. 자세한 내용은 [자격 증명 관리](/docs/en/authentication#credential-management)를 참조하세요.
</Tip>

## 4단계: 첫 번째 질문하기

코드베이스를 이해하는 것부터 시작해 봅시다. 다음 명령 중 하나를 시도해 보세요:

```text theme={null}
what does this project do?
```

Claude가 파일을 분석하고 요약을 제공합니다. 더 구체적인 질문을 할 수도 있습니다:

```text theme={null}
what technologies does this project use?
```

```text theme={null}
where is the main entry point?
```

```text theme={null}
explain the folder structure
```

Claude의 자체 기능에 대해 질문할 수도 있습니다:

```text theme={null}
what can Claude Code do?
```

```text theme={null}
how do I create custom skills in Claude Code?
```

```text theme={null}
can Claude Code work with Docker?
```

<Note>
  Claude Code는 필요한 경우 프로젝트 파일을 직접 읽어옵니다. 수동으로 컨텍스트를 추가할 필요가 없습니다.
</Note>

## 5단계: 첫 번째 코드 변경하기

이제 Claude Code가 실제 코딩 작업을 하도록 해봅시다. 간단한 태스크를 시도해 보세요:

```text theme={null}
add a hello world function to the main file
```

Claude Code는 다음 과정을 거칩니다:

1. 적절한 파일 찾기
2. 제안된 변경 사항 보여주기
3. 권한 모드에 따라 파일을 변경하기 전에 승인 요청하기
4. 편집 적용하기

<Note>
  Claude Code가 파일을 변경하기 전에 물어보는지 여부는 사용자의 [권한 모드](/docs/en/permission-modes)에 따라 달라집니다. 기본 모드에서는 Claude가 각 변경 전 승인을 요청합니다. `Shift+Tab`을 눌러 모드를 순환 전환할 수 있습니다: `acceptEdits`는 파일 편집을 자동 승인하고, `plan`은 Claude가 편집 없이 변경 사항을 제안하도록 합니다. 일부 계정에는 백그라운드 안전 검사를 구동하고 위험한 작업을 차단하며 반복된 차단 후에만 프롬프트로 돌아오는 `auto` 모드도 존재합니다.
</Note>

## 6단계: Claude Code에서 Git 사용하기

Claude Code를 사용하면 대화하듯이 Git 작업을 수행할 수 있습니다:

```text theme={null}
what files have I changed?
```

```text theme={null}
commit my changes with a descriptive message
```

더 복잡한 Git 작업을 프롬프트로 요청할 수도 있습니다:

```text theme={null}
create a new branch called feature/quickstart
```

```text theme={null}
show me the last 5 commits
```

```text theme={null}
help me resolve merge conflicts
```

## 7단계: 버그 수정 및 기능 추가

Claude는 버그 수정 및 기능 구현에 능숙합니다.

자연어로 원하는 바를 설명하세요:

```text theme={null}
add input validation to the user registration form
```

또는 기존 문제를 수정하세요:

```text theme={null}
there's a bug where users can submit empty forms - fix it
```

Claude Code는 다음을 수행합니다:

* 관련 코드 위치 파악
* 컨텍스트 이해
* 솔루션 구현
* 테스트가 있는 경우 테스트 실행

## 8단계: 기타 일반적인 워크플로 테스트

Claude와 작업하는 여러 가지 방식이 있습니다:

**코드 리팩토링**

```text theme={null}
refactor the authentication module to use async/await instead of callbacks
```

**테스트 작성**

```text theme={null}
write unit tests for the calculator functions
```

**문서 업데이트**

```text theme={null}
update the README with installation instructions
```

**코드 리뷰**

```text theme={null}
review my changes and suggest improvements
```

<Tip>
  도움을 주는 동료에게 말하듯 Claude와 대화하세요. 달성하려는 목표를 설명하면 Claude가 도달할 수 있도록 도와줄 것입니다.
</Tip>

## 필수 명령 목록

일상적인 사용을 위한 가장 중요한 명령 모음입니다. 셸 명령은 터미널에서 구동하여 Claude Code를 시작하거나 재개할 때 사용합니다. 세션 명령은 Claude Code가 시작된 후 세션 내부에서 실행합니다.

**셸 명령**

| 명령                | 하는 일                                                | 예시                                |
| ------------------- | ------------------------------------------------------ | ----------------------------------- |
| `claude`            | 대화형 모드 시작                                       | `claude`                            |
| `claude "task"`     | 일회성 태스크 실행                                     | `claude "fix the build error"`      |
| `claude -p "query"` | 일회성 조회 구동 후 종료                               | `claude -p "explain this function"` |
| `claude -c`         | 현재 디렉토리에서 가장 최근 대화 이어가기              | `claude -c`                         |
| `claude -r`         | 이전 대화 재개하기                                     | `claude -r`                         |

**세션 명령**

| 명령                    | 하는 일                    | 예시     |
| ----------------------- | -------------------------- | -------- |
| `/clear`                | 대화 기록 지우기           | `/clear` |
| `/help`                 | 사용 가능한 명령 표시      | `/help`  |
| `/exit` 또는 Ctrl+D 두 번| Claude Code 종료          | `/exit`  |

전체 셸 명령 목록은 [CLI 참조 문서](/docs/en/cli-reference)를, 전체 세션 명령 목록은 [명령 참조 문서](/docs/en/commands)를 참조하세요.

## 초보자를 위한 핵심 팁

자세한 내용은 [권장 사항](/docs/en/best-practices) 및 [일반적인 워크플로](/docs/en/common-workflows)를 참조하세요.

<AccordionGroup>
  <Accordion title="요청을 구체적으로 작성하세요">
    대신: "fix the bug"

    추천: "fix the login bug where users see a blank screen after entering wrong credentials"
  </Accordion>

  <Accordion title="단계별 지침을 활용하세요">
    복잡한 태스크를 단계를 나누어 전달하세요:

    ```text theme={null}
    1. create a new database table for user profiles
    2. create an API endpoint to get and update user profiles
    3. build a webpage that allows users to see and edit their information
    ```
  </Accordion>

  <Accordion title="Claude가 먼저 탐색하도록 하세요">
    변경을 적용하기 전에 Claude가 코드를 이해하도록 합니다:

    ```text theme={null}
    analyze the database schema
    ```

    ```text theme={null}
    build a dashboard showing products that are most frequently returned by our UK customers
    ```
  </Accordion>

  <Accordion title="단축키로 시간을 절약하세요">
    * `/`를 입력하여 모든 명령과 스킬 확인
    * 명령 자동 완성을 위해 Tab 사용
    * 명령 히스토리를 보려면 ↑ 누르기
    * 권한 모드를 순환 전환하려면 `Shift+Tab` 누르기
  </Accordion>
</AccordionGroup>

## 다음 단계

기본 사항을 익혔으므로 이제 더 고급 기능을 살펴보세요:

<CardGroup cols={2}>
  <Card title="Claude Code 작동 방식" icon="microchip" href="/docs/en/how-claude-code-works">
    에이전틱 루프, 내장 도구, Claude Code가 프로젝트와 상호작용하는 방식을 이해하세요.
  </Card>

  <Card title="권장 사항 (Best practices)" icon="star" href="/docs/en/best-practices">
    효과적인 프롬프트 작성 및 프로젝트 설정으로 더 나은 결과를 얻으세요.
  </Card>

  <Card title="일반적인 워크플로" icon="graduation-cap" href="/docs/en/common-workflows">
    자주 쓰이는 태스크를 위한 단계별 가이드입니다.
  </Card>

  <Card title="Claude Code 확장하기" icon="puzzle-piece" href="/docs/en/features-overview">
    CLAUDE.md, 스킬, 훅, MCP 등을 통해 커스텀하세요.
  </Card>
</CardGroup>

## 도움받기

* **Claude Code 세션 내부**: `/help`를 입력하거나 "how do I..." 질문하기
* **문서**: 현재 계신 곳입니다! 다른 가이드를 둘러보세요
* **커뮤니티**: 팁과 지원을 위해 [Discord](https://www.anthropic.com/discord)에 참여하세요
