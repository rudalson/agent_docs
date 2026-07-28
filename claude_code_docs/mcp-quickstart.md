> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# MCP 서버에 연결하기

> Claude Code에 MCP 서버를 추가하고, 연결을 검증하며, 디스크상에서 구성을 찾습니다.

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction)을 사용하면 이슈 트래커 검색, 데이터베이스 쿼리, 웹 브라우저 제어 등 Claude Code가 기본 제공 세트 이상의 도구를 활용할 수 있습니다. 이러한 도구들은 사용자의 머신이나 호스팅된 서비스 형태로 구동되는 MCP 서버로부터 제공됩니다.

이 가이드는 Claude Code CLI를 통해 하나의 MCP 서버에 처음부터 끝까지 연결하는 과정을 안내합니다. 가이드를 마칠 즈음에는 서버가 연결되어 응답을 보내고, 디스크상의 구성 파일 위치를 파악할 수 있으며, 가장 흔한 연결 오류를 해결할 수 있게 됩니다.

<Note>
  데스크톱 앱, VS Code, 웹 등 다른 서페이스에서도 MCP 서버를 추가할 수 있습니다. [다른 서페이스에서 연결하기](#connect-from-other-surfaces)를 참조하세요.
</Note>

Claude Code에서 MCP 서버를 연결하고 구성하는 모든 방법에 대해서는 [MCP 참조 문서](/docs/en/mcp)를 확인하세요.

## 시작하기 전에

다음 항목들이 구비되어 있는지 확인하세요:

* [Claude Code가 설치되어 있고](/docs/en/quickstart) 인증이 완료된 상태
* 프로젝트 디렉토리에 열려 있는 터미널 (빈 디렉토리를 포함하여 모든 디렉토리가 가능함)

## 서버 추가 및 검증

아래 예제는 Claude Code 문서에 대한 전체 텍스트 검색 기능을 가진 호스팅 서버인 [Claude Code 문서 MCP 서버](https://code.claude.com/docs/mcp)에 연결합니다. 별도의 인증이나 특수한 구성이 필요하지 않아 설정 흐름을 테스트하는 첫 번째 서버로 적합합니다.

모든 서버에 대한 단계는 동일합니다: 추가하고, 연결 상태를 확인한 후, 세션에서 사용하고, 끝에 선택적으로 정리하는 단계를 거칩니다. 일부 서버는 [추가 MCP 서버 예제](#additional-mcp-server-examples)에 표시된 것처럼 브라우저 로그인과 같은 단계가 추가되기도 합니다. 연결할 더 많은 서버는 [Anthropic Directory](/docs/en/mcp#find-and-build-mcp-servers)에서 찾아볼 수 있습니다.

<Steps>
  <Step title="MCP 서버 추가">
    Claude Code에 서버를 등록합니다. 대화를 시작하기 전에 서버를 구성하는 것이므로 `claude` 세션 내부가 아닌 터미널에서 이 명령을 실행하세요.

    ```bash theme={null}
    claude mcp add --transport http claude-code-docs https://code.claude.com/docs/mcp
    ```

    명령어 각 요소 설명:

    * `claude mcp add`: Claude Code에 서버를 등록합니다.
    * `--transport http`: 서버가 로컬 프로세스로 실행되지 않고 URL에서 호스팅됨을 의미합니다.
    * `claude-code-docs`: 임의로 지은 이름입니다. 해당 서버를 `docs`라고 불러도 동일하게 작동합니다. Claude Code는 사용자가 선택한 이름을 사용하여 Claude의 출력 결과에서 서버의 도구에 레이블을 붙이고 `claude mcp remove`와 같은 명령에서 서버를 참조합니다.
    * `https://code.claude.com/docs/mcp`: 서버가 호스팅되어 있는 URL입니다.

    명령어는 `Added HTTP MCP server claude-code-docs with URL: https://code.claude.com/docs/mcp to local config`와 같은 확인 문구와 함께 작성한 구성 파일을 보여주는 `File modified:` 라인을 출력합니다. `local config` 부분은 해당 서버가 이 프로젝트에 한해 사용자에게 등록되었음을 의미합니다: 다른 프로젝트에서 Claude Code를 시작하면 해당 서버는 거기서 활성화되지 않습니다. 모든 프로젝트에 대해 서버를 한 번만 등록하려면 [서버 스코프 변경하기](#change-server-scope)에서 다루는 user 스코프로 추가하세요.
  </Step>

  <Step title="연결 상태 확인">
    서버가 서버 목록에 표시되는지 확인하고 상태를 점검합니다:

    ```bash theme={null}
    claude mcp list
    ```

    서버는 다음 상태 표시기와 함께 나타납니다:

    | 상태                                             | 의미                                                                                                                                                                          |
    | :----------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `✔ Connected`                                    | 사용할 준비가 완료됨. `claude-code-docs`에 대해 이 표시가 나타나야 함                                                                                                         |
    | `! Connected · tools fetch failed`               | 서버는 연결되었으나 도구 목록을 가져오지 못함. 상세 오류는 `claude mcp get <name>`을 실행하여 확인                                                                            |
    | `! Needs authentication`                         | 서버에 접근 가능하나 브라우저 로그인 또는 `--header`로 전달되는 토큰이 필요함. [로그인이 필요한 서버 연결하기](#connect-a-server-that-requires-sign-in) 참조                  |
    | `✘ Failed to connect`                            | 서버가 응답하지 않음. [문제 해결](#troubleshooting) 참조                                                                                                                      |
    | `✘ Connection error`                             | 연결 시도 중 오류가 발생함. [문제 해결](#troubleshooting) 참조                                                                                                                |
    | ``⏸ Pending approval (run `claude` to approve)`` | 아직 승인하지 않은 project 스코프 서버임. [.mcp.json 직접 편집하기](#edit-mcp-json-directly) 참조                                                                             |

    Windows 10의 기본 콘솔과 같은 일부 이전 버전의 Windows 콘솔은 이 유니코드 기호를 지원하지 않으므로 `✔` 및 `✘` 대신 `√` 및 `×`를 표시합니다.
  </Step>

  <Step title="서버 사용">
    세션을 시작하고 Claude에게 새 서버를 이름을 지정하여 사용하도록 요청합니다:

    ```bash theme={null}
    claude
    ```

    ```text theme={null}
    Use the claude-code-docs server to look up what MCP_TIMEOUT does
    ```

    <Info>
      Claude는 관련 도구를 스스로 선택하므로 일반적으로 프롬프트에서 서버 이름을 명시할 필요는 없습니다. 여기서 이름을 지정하는 것은 웹 검색과 같이 동일한 질문에 답할 수 있는 다른 도구가 아닌 새 서버를 통해 시연이 진행되도록 보장하기 위함입니다.
    </Info>

    Claude가 서버를 처음 호출할 때 새 도구 사용에 대한 권한을 요청합니다. 계속하려면 승인하세요. Claude 출력의 도구 호출에는 서버 이름이 레이블로 붙어 나타나므로, 답변이 Claude의 내장 지식이 아닌 MCP 서버에서 왔음을 확인할 수 있습니다.
  </Step>

  <Step title="서버 제거">
    이 단계는 선택 사항입니다. 실험을 마쳤다면 서버를 제거할 수 있습니다:

    ```bash theme={null}
    claude mcp remove claude-code-docs
    ```

    명령어는 `Removed MCP server "claude-code-docs" from local config`라는 확인 문구와 함께 업데이트된 파일을 나타내는 `File modified:` 라인을 표시합니다.

    <Note>
      도구 이름과 서버 지침이 매 세션마다 로드되기 때문에 연결된 각 서버는 [Claude의 컨텍스트 창](/docs/en/how-claude-code-works#the-context-window)에서 일정 공간을 차지합니다. 더 이상 사용하지 않는 서버를 제거하면 해당 공간을 확보할 수 있습니다.
    </Note>
  </Step>
</Steps>

## 서버가 저장되는 위치

`claude mcp add` 명령은 구성 파일에 서버 상세 정보를 작성합니다. 기본적으로 서버는 `local` 스코프로 등록됩니다: 본인에게만 비공개이며 현재 프로젝트에서만 활성화됩니다. 모든 프로젝트에 대해 한 번만 등록하려면 `--scope user`를 전달하고, 팀원과 공유하려면 `--scope project`를 전달하세요. [서버 스코프 변경하기](#change-server-scope)에서 두 가지를 모두 안내합니다.

<Note>
  `claude mcp add`는 PowerShell 및 명령 프롬프트를 포함한 모든 셸에서 동일하게 작동합니다. `claude` 세션 내부에서는 이미 추가한 서버를 확인하고 관리하기 위해 `/mcp` 명령을 사용하세요.
</Note>

이 페이지 뒷부분에서 각각 다룰 다른 서버 추가 방식들도 존재합니다:

* [로컬 서버 추가](#add-a-local-server): URL에 연결하는 대신 머신의 프로그램을 실행합니다.
* [.mcp.json 직접 편집](#edit-mcp-json-directly): 명령어를 사용하는 대신 직접 JSON 항목을 작성합니다.
* [로그인이 필요한 서버 연결](#connect-a-server-that-requires-sign-in): 도구가 동작하기 전에 브라우저 로그인이 필요한 호스팅 서버를 추가합니다.

### 디스크에서 구성 찾기

`claude mcp add` 명령은 `--scope` 플래그에 따라 두 개 파일에 걸쳐 저장되는 세 가지 스코프 중 하나로 서버를 기록합니다. 이 파일들을 직접 편집할 필요는 없지만 위치를 파악해 두면 디버깅 및 버전 관리에 도움이 됩니다.

| 스코프    | 파일                                                   | 사용 가능 대상                           |
| :-------- | :----------------------------------------------------- | :--------------------------------------- |
| `local`   | `~/.claude.json` (해당 프로젝트 항목 아래)             | 본인만, 이 프로젝트만 가능 (기본값)       |
| `project` | 프로젝트 루트의 `.mcp.json`                             | 프로젝트를 복제하는 모든 사람             |
| `user`    | `~/.claude.json` (최상위 `mcpServers` 키 아래)          | 본인만, 모든 프로젝트 가능               |

Windows에서 `~/.claude.json`은 `%USERPROFILE%\.claude.json`(일반적으로 `C:\Users\YourName\.claude.json`)으로 해석됩니다. [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars)을 설정한 경우 Claude Code는 대신 해당 디렉토리 내부에서 `.claude.json`을 읽습니다.

`claude mcp get claude-code-docs`를 실행하여 어떤 스코프에 서버 정의가 보관되어 있는지 확인하세요. 동일한 서버가 둘 이상의 스코프에 정의되어 있을 때 상호작용하는 방식은 [MCP 설치 스코프](/docs/en/mcp#mcp-installation-scopes)를 참조하세요.

## 서버 스코프 변경하기

서버 스코프는 추가될 때 고정되므로 스코프를 변경한다는 것은 기존 항목을 제거하고 새 스코프로 다시 추가함을 의미합니다. 아래의 두 예시는 모두 서버에 단 하나의 정의만 남도록 첫 번째 가이드의 local 항목을 제거하는 것으로 시작합니다. 이미 첫 가이드 끝에서 제거했다면 이 명령은 건너뛰세요:

```bash theme={null}
claude mcp remove claude-code-docs --scope local
```

### 모든 프로젝트에서 서버 사용하기

여전히 본인만 비공개로 사용하면서 여는 모든 프로젝트에서 활성화되도록 `user` 스코프로 서버를 다시 추가합니다:

```bash theme={null}
claude mcp add --scope user --transport http claude-code-docs https://code.claude.com/docs/mcp
```

### 팀과 서버 공유하기

프로젝트 루트의 `.mcp.json`에 작성되도록 `project` 스코프로 서버를 다시 추가합니다:

```bash theme={null}
claude mcp add --scope project --transport http claude-code-docs https://code.claude.com/docs/mcp
```

`.mcp.json`을 버전 관리에 커밋하세요. 저장소를 복제하고 Claude Code를 시작하는 팀원에게 서버 승인 프롬프트가 표시되며, 승인하면 그들에게도 서버가 연결됩니다.

## 추가 MCP 서버 예제

첫 번째 안내에서는 로그인 없이 연결되는 호스팅 서버를 사용했습니다. 아래 예제들은 동일한 추가-확인-사용 흐름을 통해 다른 두 가지 흔한 형태를 다룹니다.

### 로컬 서버 추가

로컬 stdio 서버는 URL로 연결되는 서비스가 아니라 Claude Code가 사용자의 머신에서 서브프로세스로 시작하는 프로그램입니다. 브라우저, 파일 시스템, 데이터베이스 소켓 등 로컬 리소스에 대한 접근이 필요한 도구에 사용하세요.

[Playwright MCP 서버](https://github.com/microsoft/playwright-mcp)는 시도해 보기 좋은 서버입니다: 계정이 필요 없고 Claude에게 탐색, 클릭, 읽기를 수행할 수 있는 브라우저를 제공합니다. `npx`를 통해 실행되므로 [Node.js](https://nodejs.org/en/download) 18 이상이 필요합니다.

<Steps>
  <Step title="Playwright 서버 추가">
    Claude Code가 서버를 시작할 때 실행해야 하는 명령으로 서버를 등록합니다:

    ```bash theme={null}
    claude mcp add playwright -- npx -y @playwright/mcp@latest
    ```

    이 명령은 호스팅 서버 예제와 세 가지 면에서 다릅니다:

    * 로컬 서버는 기본 `stdio` 트랜스포트를 사용하므로 `--transport` 플래그가 없습니다.
    * `--` 구분자 뒤의 모든 내용은 Claude Code가 서버를 시작하기 위해 실행하는 명령입니다.
    * `-y`는 `npx`가 확인 절차 없이 패키지를 설치하도록 지시합니다.

    명령어는 `Added stdio MCP server playwright with command: npx -y @playwright/mcp@latest to local config`와 같은 확인 문구와 함께 작성한 구성 파일을 나타내는 `File modified:` 라인을 출력합니다.

    Playwright는 머신에 이미 설치되어 있는 Chrome을 구동시킵니다. 다른 브라우저를 사용하려면 `@playwright/mcp@latest` 뒤에 `--browser firefox`와 같이 `--browser` 뒤에 브라우저 이름을 붙여 추가하세요.
  </Step>

  <Step title="연결 확인">
    `Added` 확인 문구는 항목이 저장되었음을 의미하며 명령이 정상 실행됨을 의미하지는 않습니다. 연결을 확인하세요:

    ```bash theme={null}
    claude mcp list
    ```

    `npx`가 패키지를 다운로드하는 동안 첫 번째 확인 시 `✘ Failed to connect`가 표시될 수 있으므로 잠시 기다린 후 다시 실행해 보세요. 다운로드가 완료되면 상태가 `✔ Connected`로 변경됩니다. 몇 번 재시도한 후에도 여전히 `✘ Failed to connect`가 표시되면 [문제 해결](#troubleshooting)을 참조하세요.
  </Step>

  <Step title="브라우저 사용">
    Claude에게 브라우저가 필요한 작업을 부여합니다:

    ```text theme={null}
    Use playwright to open https://example.com and tell me the page title
    ```

    작동하는 모습을 지켜볼 수 있도록 브라우저 창이 열리고, Claude 출력의 도구 호출에는 `playwright` 서버 이름과 `browser_navigate`와 같은 동작 레이블이 붙습니다.

    변경 후 페이지가 제대로 렌더링되는지 확인하기 위해 로컬 개발 서버를 가리켜 보거나 버그 리포트를 단계별로 실행하도록 해보세요.
  </Step>
</Steps>

### 로그인이 필요한 서버 연결

Sentry, Linear, Notion과 같은 호스팅 서비스는 OAuth 뒤에서 MCP 서버를 구동합니다: 서버 URL을 추가한 다음 브라우저를 통해 로그인합니다.

아래 단계에서는 Sentry를 예시로 사용합니다. 다른 서비스를 연결하려면 [Anthropic Directory](/docs/en/mcp#find-and-build-mcp-servers) 또는 해당 서비스 문서에서 찾을 수 있는 URL로 대체하세요.

<Steps>
  <Step title="서버 추가">
    `add` 명령은 Sentry의 URL을 사용한다는 점을 제외하면 문서 서버와 동일합니다:

    ```bash theme={null}
    claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
    ```

    추가 후 `claude mcp list`를 실행하면 서버가 `! Needs authentication` 상태로 표시됩니다. 이는 예상된 결과이며, 다음 단계에서 로그인을 완료합니다.
  </Step>

  <Step title="브라우저에서 인증">
    Claude Code 세션을 시작하고 MCP 패널을 엽니다:

    ```text theme={null}
    /mcp
    ```

    목록에서 `sentry`를 선택하고 Enter 키를 누른 뒤 `Authenticate`를 선택합니다. 브라우저가 열리며 Sentry 로그인 페이지가 나타납니다. 거기서 연결을 승인하세요.

    Claude Code로 돌아오면 서버 상태가 connected로 변경됩니다. 로그인이 실패하거나 브라우저가 열리지 않으면 [문제 해결](#troubleshooting)을 참조하세요.
  </Step>

  <Step title="서버 사용">
    `What Sentry projects do I have access to?`와 같이 해당 서비스가 필요한 질문을 Claude에게 던지고, 출력에서 `sentry` 서버 이름이 붙은 도구 호출을 확인하세요.
  </Step>
</Steps>

OAuth 대신 정적 토큰으로 인증하는 서버는 추가 시점에 `--header "Authorization: Bearer <token>"`으로 토큰을 받습니다. 완성된 예제는 [GitHub 예시](/docs/en/mcp#example-connect-to-github-for-code-reviews)를 참조하세요.

## .mcp.json 직접 편집

[스코프 표](#find-your-configuration-on-disk)에 포함된 모든 파일은 서버 항목에 동일한 JSON 형식을 사용합니다. 이 섹션에서는 프로젝트 스코프 파일인 `.mcp.json`을 편집합니다. 이 파일은 저장소에 커밋되어 팀의 코드형 구성(configuration-as-code) 역할을 하므로 직접 작성할 가치가 가장 높습니다.

프로젝트 루트에 `.mcp.json`을 생성하세요. 아래 예시는 이 가이드의 두 서버(HTTP로 도달하는 호스팅 문서 서버와 로컬 `stdio` 프로세스인 Playwright 서버)를 모두 정의합니다:

```json theme={null}
{
  "mcpServers": {
    "claude-code-docs": {
      "type": "http",
      "url": "https://code.claude.com/docs/mcp"
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

필드는 서버 유형에 따라 다릅니다:

* HTTP 서버의 경우 `url`은 Claude Code가 연결하는 엔드포인트입니다.
* stdio 서버의 경우 `command`와 `args`는 실행할 프로그램입니다.

파일을 저장한 후 프로젝트에서 새 Claude Code 세션을 시작하세요. Claude Code는 시작 시 `.mcp.json`을 읽습니다.

Claude Code가 프로젝트 스코프 서버를 처음 발견하면 승인을 요청합니다. 이 프롬프트는 복제한 저장소가 사용자 동의 없이 머신에서 프로세스를 실행하지 못하도록 하기 위해 존재합니다. 프롬프트를 승인하거나 놓친 경우 나중에 `/mcp`를 실행하여 승인하세요.

승인한 후 `/mcp`를 실행하여 서버가 connected로 표시되는지 확인하세요. 하나에 오류가 표시되면 [문제 해결](#troubleshooting)을 참조하세요.

## 다른 서페이스에서 연결하기

이 가이드는 `claude mcp` CLI 명령을 사용하지만 모든 Claude Code 서페이스에서 MCP 서버에 연결할 수 있습니다:

* **Claude Code 데스크톱 앱**: [Connectors UI](/docs/en/desktop#connect-external-tools)를 통해 서버를 추가합니다.
* **Claude Desktop 채팅 앱**: Claude Code와 별개의 앱입니다. 해당 앱의 `claude_desktop_config.json`에서 CLI로 서버를 복사해 오려면 macOS 또는 WSL에서 `claude mcp add-from-claude-desktop`을 실행하세요.
* **VS Code**: [MCP로 외부 도구에 연결하기](/docs/en/vs-code#connect-to-external-tools-with-mcp)를 참조하세요.
* **Claude Code on the web**: 저장소의 `.mcp.json`을 읽습니다. [.mcp.json 직접 편집하기](#edit-mcp-json-directly)를 참조하세요.
* **Claude.ai**: [claude.ai/customize/connectors](https://claude.ai/customize/connectors)에서 추가한 커넥터는 해당 계정으로 로그인할 때 CLI에 자동으로 로드됩니다. [Claude.ai의 MCP 서버 사용하기](/docs/en/mcp#use-mcp-servers-from-claude-ai)를 참조하세요.

## 문제 해결

서버가 연결되지 않는 경우 세션 내부에서 `/mcp`를 실행하거나 셸에서 `claude mcp list`를 실행하여 상태를 점검한 후 아래의 해당 증상과 일치시키세요. `/mcp` 패널에서는 세션을 벗어나지 않고도 다시 연결하거나 인증을 수행할 수 있습니다.

<AccordionGroup>
  <Accordion title="/mcp에 No MCP servers configured라고 표시됨">
    Claude Code가 현재 디렉토리에서 아무런 서버도 찾지 못했습니다. 가장 흔한 원인:

    * 다른 프로젝트에서 `claude mcp add`를 실행했습니다. Local 스코프 서버는 추가한 프로젝트(저장소 루트, 또는 git 저장소가 아니라면 해당 디렉토리)에 귀속됩니다. 지금 있는 프로젝트에서 서버를 다시 추가하거나, 프로젝트에 얽매이지 않도록 `--scope user`를 사용하여 추가하세요.
    * 잘못된 경로의 구성 파일을 편집했습니다. 올바른 파일은 `~/.claude.json` 및 `<project>/.mcp.json`입니다. Claude Code는 `~/.claude/.mcp.json`, `~/.claude/config/mcp.json`, `~/.claude/mcp.json`, `%APPDATA%\Claude\mcp.json`과 같은 경로는 읽지 않습니다. User 스코프 서버의 경우 `~/.claude.json` 내의 `mcpServers` 키에 작성되는 `claude mcp add --scope user`를 실행하세요; project 스코프 서버의 경우 프로젝트 루트의 `.mcp.json`을 편집하세요.
  </Accordion>

  <Accordion title="상태에 Failed to connect 또는 Connection error가 표시됨">
    두 상태 모두 서버가 시작되지 않았거나 URL이 응답하지 않음을 의미합니다. [로그인이 필요한 서버 연결하기](#connect-a-server-that-requires-sign-in)에서 다룬 브라우저 로그인 대신 토큰을 요구하는 HTTP 서버의 경우에도 이 상태가 나타날 수 있습니다.

    v2.1.191부터 HTTP 서버가 `404 Not Found`를 반환할 때 `/mcp`에서 해당 서버를 선택하면 Claude Code가 시도한 URL과 함께 `MCP endpoint not found at <url>. Check the URL in your MCP config.`가 표시됩니다. 이전 버전은 URL 없이 일반적인 `Error POSTing to endpoint` 메시지를 표시했었습니다. 해당 URL을 서버 문서에 기재된 MCP 엔드포인트 경로와 비교한 뒤 `claude mcp remove <name>`을 실행하고 올바른 URL로 다시 추가하세요.

    HTTP 서버의 경우 머신에서 URL에 접근 가능한지 확인하세요:

    ```bash theme={null}
    curl -I https://mcp.sentry.dev/mcp
    ```

    PowerShell에서는 `Invoke-WebRequest` 별칭 대신 실제 curl 바이너리로 요청이 전달되도록 `curl` 대신 `curl.exe`를 사용하세요.

    응답을 통해 어떤 유형의 문제인지 파악할 수 있습니다:

    * `404` 또는 `405`: 서버가 가동 중입니다. 많은 MCP 엔드포인트가 POST 요청에만 응답하므로 이 상태여도 머신에서 URL에 도달할 수 있음이 증명됩니다.
    * `401` 또는 `403`: 서버가 가동 중이며 인증이 필요합니다. [로그인이 필요한 서버 연결하기](#connect-a-server-that-requires-sign-in)의 브라우저 로그인을 사용하거나, GitHub처럼 토큰을 받는 서버의 경우 `claude mcp add` 명령에 `--header "Authorization: Bearer <token>"`을 전달하세요.
    * 아무 응답도 없음: URL 및 네트워크 상태를 확인하세요.

    stdio 서버의 경우 구성된 명령을 터미널에서 직접 실행하여 내부 오류를 확인하세요. 이 가이드의 Playwright 서버의 경우 다음을 실행하세요:

    ```bash theme={null}
    npx -y @playwright/mcp@latest
    ```

    그 다음 발생하는 현상에 따라 문제 위치를 알 수 있습니다:

    * 명령이 시작되고 입력을 기다림: 서버 자체는 작동합니다. `claude mcp get <name>`을 실행하여 거기서 표시되는 명령이 방금 실행한 내용과 일치하는지 확인하세요. 표시된 명령이 입력한 내용과 다르면 서버 명령 앞의 `--` 구분자를 누락했을 가능성이 높습니다. 서버를 제거하고 `--`를 넣어서 다시 추가하세요. `.mcp.json`을 직접 작성한 경우 문법과 위치를 확인하세요.
    * 명령 오류 발생: 메시지에 Node.js나 브라우저 등 누락된 요소가 명시됩니다.
  </Accordion>

  <Accordion title="시작 시 연결 타임아웃 발생">
    서버가 기본 30초 시작 타임아웃보다 더 오래 걸렸습니다. stdio 서버는 `npx`가 패키지를 다운로드하는 동안 첫 실행이 느릴 수 있습니다. [`MCP_TIMEOUT`](/docs/en/env-vars) 환경 변수를 밀리초 단위로 설정하여 제한을 늘리세요:

    ```bash theme={null}
    MCP_TIMEOUT=60000 claude
    ```

    PowerShell에서는 동일한 줄의 명령 앞에 변수를 설정하세요:

    ```powershell theme={null}
    $env:MCP_TIMEOUT = "60000"; claude
    ```
  </Accordion>

  <Accordion title="Server already exists 오류">
    동일한 스코프에 동일한 이름의 서버를 이미 추가했습니다. 기존 항목을 먼저 제거하거나 다른 이름을 선택하세요:

    ```bash theme={null}
    claude mcp remove claude-code-docs
    ```

    둘 이상의 스코프에 이름이 존재하는 경우 `remove`가 `exists in multiple scopes`를 보고합니다. `--scope`를 전달하여 삭제할 사본을 선택하세요 (예: `claude mcp remove claude-code-docs --scope local`).
  </Accordion>

  <Accordion title="서버는 연결되었으나 도구가 나타나지 않음">
    세션 내부에서 `/mcp`를 실행하고 해당 서버를 선택하여 도구 목록을 확인하세요. 목록이 비어 있다면 서버가 시작은 되었으나 아무런 도구도 등록하지 않은 것이며, 이는 일반적으로 API 키와 같은 필수 환경 변수가 누락되었음을 의미합니다.

    `claude mcp add` 시 `--env KEY=value`를 전달하거나 서버의 `.mcp.json` 항목 내 `env` 필드에 추가하세요. 서버 문서에 필요한 변수들이 안내되어 있습니다.
  </Accordion>

  <Accordion title=".mcp.json 변경사항이 적용되지 않음">
    Claude Code는 세션 시작 시 `.mcp.json`을 읽습니다. 파일을 편집한 후 세션을 종료하고 다시 시작하세요.

    여전히 서버가 나타나지 않으면 `/mcp`를 실행하여 구문 경고를 확인하세요. Claude Code는 형식이 잘못된 항목을 건너뛰고 문제가 되는 필드를 거기에 표시합니다.

    이전에 프롬프트가 떴을 때 서버를 거부했던 경우 프로젝트 승인을 재설정하세요:

    ```bash theme={null}
    claude mcp reset-project-choices
    ```
  </Accordion>

  <Accordion title="OAuth 로그인이 실패하거나 브라우저가 열리지 않음">
    `/mcp`를 실행하고 서버를 선택한 뒤 `Authenticate`를 다시 선택하세요. 브라우저가 자동으로 열리지 않으면 터미널에 표시된 URL을 복사하여 수동으로 열어보세요. 고정 콜백 포트 및 사전 구성된 자격 증명에 대해서는 [원격 MCP 서버 인증](/docs/en/mcp#authenticate-with-remote-mcp-servers)을 참조하세요.
  </Accordion>
</AccordionGroup>

## 다음 단계

하나의 서버가 연결되었으니 MCP가 지원하는 다른 기능들을 둘러보세요:

* Anthropic Directory에서 [더 많은 MCP 서버 찾기](/docs/en/mcp#find-and-build-mcp-servers)
* 설치 스코프를 활용하여 [팀과 서버 공유하기](/docs/en/mcp#mcp-installation-scopes)
* 관리형 설정 및 정책 제어로 [조직의 MCP 액세스 관리하기](/docs/en/managed-mcp)
* 프롬프트에서 @ 멘션으로 [MCP 리소스 참조하기](/docs/en/mcp#use-mcp-resources)
* `/` 메뉴에서 [MCP 프롬프트를 명령으로 실행하기](/docs/en/mcp#use-mcp-prompts-as-commands)
* MCP SDK로 [자체 서버 구축하기](https://modelcontextprotocol.io/quickstart/server)
