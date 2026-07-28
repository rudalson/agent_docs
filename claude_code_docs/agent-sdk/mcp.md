> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# MCP로 외부 도구에 연결하기

> MCP 서버를 구성하여 외부 도구로 에이전트를 확장합니다. 전송 방식 타입, 대규모 도구 세트를 위한 도구 검색, 인증, 오류 처리를 다룹니다.

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/docs/getting-started/intro)은 AI 에이전트를 외부 도구 및 데이터 소스에 연결하기 위한 오픈 표준입니다. MCP를 사용하면 에이전트가 커스텀 도구 구현을 작성하지 않고도 데이터베이스를 쿼리하고, Slack 및 GitHub과 같은 API와 통합하며, 기타 서비스에 연결할 수 있습니다.

MCP 서버는 로컬 프로세스로 실행되거나, HTTP를 통해 연결되거나, SDK 애플리케이션 내부에서 직접 실행될 수 있습니다.

<Note>
  이 페이지에서는 Agent SDK용 MCP 구성을 다룹니다. 모든 프로젝트에서 로드되도록 Claude Code CLI에 MCP 서버를 추가하려면 [MCP 설치 범위](/docs/en/mcp#mcp-installation-scopes)를 참조하세요.
</Note>

## 빠른 시작

이 예제는 [HTTP 전송 방식](#httpsse-서버)을 사용하는 [Claude Code 문서](https://code.claude.com/docs) MCP 서버에 연결하고 와일드카드가 적용된 [`allowedTools`](#mcp-도구-허용하기)를 사용하여 해당 서버의 모든 도구를 허용합니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Use the docs MCP server to explain what hooks are in Claude Code",
    options: {
      mcpServers: {
        "claude-code-docs": {
          type: "http",
          url: "https://code.claude.com/docs/mcp"
        }
      },
      allowedTools: ["mcp__claude-code-docs__*"]
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={
              "claude-code-docs": {
                  "type": "http",
                  "url": "https://code.claude.com/docs/mcp",
              }
          },
          allowed_tools=["mcp__claude-code-docs__*"],
      )

      async for message in query(
          prompt="Use the docs MCP server to explain what hooks are in Claude Code",
          options=options,
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```
</CodeGroup>

에이전트가 문서 서버에 연결하고, 후크에 관한 정보를 검색한 후 결과를 반환합니다.

## MCP 서버 추가하기

`query()` 호출 시 코드 내부에서, 또는 [`settingSources`](#설정-파일에서)를 통해 로드되는 `.mcp.json` 파일에서 MCP 서버를 구성할 수 있습니다.

### 코드 내부에서

`mcpServers` 옵션에 MCP 서버를 직접 전달합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "List files in my project",
    options: {
      mcpServers: {
        filesystem: {
          command: "npx",
          args: ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
        }
      },
      allowedTools: ["mcp__filesystem__*"]
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={
              "filesystem": {
                  "command": "npx",
                  "args": [
                      "-y",
                      "@modelcontextprotocol/server-filesystem",
                      "/Users/me/projects",
                  ],
              }
          },
          allowed_tools=["mcp__filesystem__*"],
      )

      async for message in query(prompt="List files in my project", options=options):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```
</CodeGroup>

### 설정 파일에서

프로젝트 루트에 `.mcp.json` 파일을 생성합니다. 기본 `query()` 옵션에서 활성화되어 있는 `project` 설정 소스가 켜져 있을 때 이 파일을 읽어옵니다. `settingSources`를 명시적으로 설정하는 경우 이 파일이 로드되도록 `"project"`를 포함해야 합니다:

```json theme={null}
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
    }
  }
}
```

## 연결 타이밍

`options.mcpServers`로 전달한 서버는 쿼리가 시작되는 즉시 연결을 시작합니다. 기본적으로 연결은 비차단식(non-blocking)입니다: 첫 번째 턴은 대기 없이 시작되고, 각 서버의 도구는 연결이 완료되면 사용 가능해집니다. {/* min-version: 2.1.142 */}Claude Code v2.1.142 이전에는 시작 시 연결 배치에 대해 최대 5초간 대기(block)했습니다.

모든 서버에 대해 제한된 시작 대기 시간을 복원하려면 [`MCP_CONNECTION_NONBLOCKING`](/docs/en/env-vars) 환경 변수를 `0`으로 설정하세요. 대기 시간은 [`MCP_CONNECT_TIMEOUT_MS`](/docs/en/env-vars)에 의해 최대 5초로 제한되며, 해당 제한 시간까지 보류 중인 서버는 백그라운드에서 계속 연결을 시도합니다.

특정 서버의 도구를 첫 턴 전에 사용 가능하게 만들려면 해당 구성에 `alwaysLoad: true`를 설정하세요. 그러면 다른 서버가 백그라운드에서 계속 연결되는 동안 해당 서버의 연결을 위해 시작 대기를 진행(동일한 5초 시작 제한 시간 적용)합니다. `alwaysLoad` 필드에는 Claude Code v2.1.121 이상이 필요합니다. 도구 검색에 대한 `alwaysLoad` 필드의 효과는 [도구 검색에서 서버 제외하기](/docs/en/mcp#exempt-a-server-from-deferral)를 참조하세요.

하위 유형이 `init`인 `system` 메시지는 방출되는 시점에 각 서버의 상태를 보고합니다. 아직 연결 중인 서버의 상태는 `pending`입니다. 사용 불가능한 서버를 감지하려는 경우 `connected` 이외의 모든 상태를 실패로 취급하기보다는 `failed` 또는 `needs-auth` 상태를 확인하세요. 전체 상태 확인은 [오류 처리](#오류-처리)를 참조하세요.

## MCP 도구 허용하기

MCP 도구는 Claude가 사용하기 전에 명시적인 권한이 필요합니다. 권한이 없으면 Claude에게 도구가 사용 가능한 것으로 보이지만 호출할 수는 없습니다.

### 도구 명명 규칙

MCP 도구는 `mcp__<server-name>__<tool-name>` 명명 패턴을 따릅니다. 예를 들어 `list_issues` 도구를 가진 `"github"`라는 이름의 GitHub 서버는 `mcp__github__list_issues`가 됩니다.

### allowedTools로 자동 승인하기

Claude가 권한 프롬프트 없이 특정 MCP 도구를 사용할 수 있도록 `allowedTools`를 사용하여 자동 승인합니다:

<CodeGroup>
  ```typescript TypeScript hidelines={1,-1} theme={null}
  const _ = {
    options: {
      mcpServers: {
        // 내 서버 목록
      },
      allowedTools: [
        "mcp__github__*", // github 서버의 모든 도구
        "mcp__db__query", // db 서버의 query 도구만
        "mcp__slack__send_message" // slack 서버의 send_message 도구만
      ]
    }
  };
  ```

  ```python Python theme={null}
  options = ClaudeAgentOptions(
      mcp_servers={
          # 내 서버 목록
      },
      allowed_tools=[
          "mcp__github__*",  # github 서버의 모든 도구
          "mcp__db__query",  # db 서버의 query 도구만
          "mcp__slack__send_message",  # slack 서버의 send_message 도구만
      ],
  )
  ```
</CodeGroup>

와일드카드(`*`)를 사용하면 개별 도구를 일일이 나열하지 않고도 서버의 모든 도구를 허용할 수 있습니다.

<Note>
  **MCP 접근에는 권한 모드보다 `allowedTools`를 선호하세요.** `permissionMode: "acceptEdits"`는 MCP 도구를 자동 승인하지 않습니다(파일 편집 및 파일 시스템 Bash 명령만 승인). `permissionMode: "bypassPermissions"`는 MCP 도구를 자동 승인하지만 대부분의 다른 안전 프롬프트도 비활성화하므로 필요 이상으로 광범위합니다. 남아있는 프롬프트는 [권한 평가 방식](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)을 참조하세요. `allowedTools`에 와일드카드를 지정하면 원하는 정확한 MCP 서버만 부여되고 그 이상은 부여되지 않습니다. 전체 비교는 [권한 모드](/docs/en/agent-sdk/permissions#permission-modes)를 참조하세요.
</Note>

### 사용 가능한 도구 탐색하기

MCP 서버가 제공하는 도구를 확인하려면 해당 서버의 문서를 확인하거나 `system` init 메시지의 `tools` 배열을 검사하세요. MCP 도구 이름은 `mcp__`로 시작합니다.

MCP 서버는 기본적으로 백그라운드에서 연결되므로 세션 시작 시 init 메시지가 완료되기 전에도 도착할 수 있습니다: 이 시점에 `tools` 배열에는 내장 도구만 나열되고 `mcp_servers`에는 각 서버의 `pending` 상태가 표시됩니다. init 메시지가 전송되기 전에 서버 연결을 위해 최대 5초간 대기하려면 [`MCP_CONNECTION_NONBLOCKING`](/docs/en/env-vars) 환경 변수를 `0`으로 설정하세요. 제한 시간 내에 연결된 서버는 여기에 `mcp__` 도구를 나열하고, 더 느린 서버는 백그라운드에서 계속 연결을 시도합니다:

```bash theme={null}
export MCP_CONNECTION_NONBLOCKING=0
```

해당 변수가 설정되면 이 필터는 MCP 도구 이름을 출력합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const options = {
    mcpServers: {
      // 내 서버 목록
    },
  };

  for await (const message of query({ prompt: "...", options })) {
    if (message.type === "system" && message.subtype === "init") {
      const mcpTools = message.tools.filter((name) => name.startsWith("mcp__"));
      console.log("Available MCP tools:", mcpTools);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={
              # 내 서버 목록
          },
      )
      async for message in query(prompt="...", options=options):
          if isinstance(message, SystemMessage) and message.subtype == "init":
              mcp_tools = [t for t in message.data.get("tools", []) if t.startswith("mcp__")]
              print("Available MCP tools:", mcp_tools)


  asyncio.run(main())
  ```
</CodeGroup>

Claude에게 특정 서버에서 사용 가능한 도구 목록을 알려달라고 요청할 수도 있습니다.

## 전송 방식 타입 (Transport types)

MCP 서버는 서로 다른 전송 프로토콜을 사용하여 에이전트와 통신합니다. 서버 문서를 확인하여 지원하는 전송 방식을 확인하세요:

* 문서에서 **실행할 명령**(예: `npx @modelcontextprotocol/server-filesystem`)을 제공하는 경우 stdio 사용
* 문서에서 **URL**을 제공하는 경우 HTTP 또는 SSE 사용
* 코드 내부에서 직접 커스텀 도구를 작성하는 경우 SDK MCP 서버 사용

### stdio 서버

stdin/stdout을 통해 통신하는 로컬 프로세스입니다. 동일한 머신에서 실행하는 MCP 서버에 사용하세요:

<Tabs>
  <Tab title="코드 내부에서">
    <CodeGroup>
      ```typescript TypeScript hidelines={1,-1} theme={null}
      const _ = {
        options: {
          mcpServers: {
            filesystem: {
              command: "npx",
              args: ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
            }
          },
          allowedTools: ["mcp__filesystem__read_file", "mcp__filesystem__list_directory"]
        }
      };
      ```

      ```python Python theme={null}
      options = ClaudeAgentOptions(
          mcp_servers={
              "filesystem": {
                  "command": "npx",
                  "args": [
                      "-y",
                      "@modelcontextprotocol/server-filesystem",
                      "/Users/me/projects",
                  ],
              }
          },
          allowed_tools=["mcp__filesystem__read_file", "mcp__filesystem__list_directory"],
      )
      ```
    </CodeGroup>
  </Tab>

  <Tab title=".mcp.json">
    ```json theme={null}
    {
      "mcpServers": {
        "filesystem": {
          "command": "npx",
          "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
        }
      }
    }
    ```
  </Tab>
</Tabs>

### HTTP/SSE 서버

클라우드 호스팅 MCP 서버 및 원격 API에는 HTTP 또는 SSE를 사용하세요:

<Tabs>
  <Tab title="코드 내부에서">
    <CodeGroup>
      ```typescript TypeScript hidelines={1,-1} theme={null}
      const _ = {
        options: {
          mcpServers: {
            "remote-api": {
              type: "sse",
              url: "https://api.example.com/mcp/sse",
              headers: {
                Authorization: `Bearer ${process.env.API_TOKEN}`
              }
            }
          },
          allowedTools: ["mcp__remote-api__*"]
        }
      };
      ```

      ```python Python theme={null}
      options = ClaudeAgentOptions(
          mcp_servers={
              "remote-api": {
                  "type": "sse",
                  "url": "https://api.example.com/mcp/sse",
                  "headers": {"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
              }
          },
          allowed_tools=["mcp__remote-api__*"],
      )
      ```
    </CodeGroup>
  </Tab>

  <Tab title=".mcp.json">
    ```json theme={null}
    {
      "mcpServers": {
        "remote-api": {
          "type": "sse",
          "url": "https://api.example.com/mcp/sse",
          "headers": {
            "Authorization": "Bearer ${API_TOKEN}"
          }
        }
      }
    }
    ```
  </Tab>
</Tabs>

스트리밍 가능한 HTTP 전송 방식의 경우 대신 `"type": "http"`를 사용하세요. `.mcp.json` 및 기타 JSON 구성 파일에서는 `"streamable-http"`가 `"http"`의 별칭으로 허용됩니다. 프로그래밍 방식 `mcpServers` 옵션은 `"http"`만 수용합니다.

### SDK MCP 서버

별도의 서버 프로세스를 실행하는 대신 애플리케이션 코드 내에서 커스텀 도구를 직접 정의하세요. 구현 세부 정보는 [커스텀 도구 가이드](/docs/en/agent-sdk/custom-tools)를 참조하세요.

{/* min-version: 2.1.210 */}[`initialize` 제어 요청](/docs/en/agent-sdk/typescript#sdkcontrolinitializeresponse)에 의해 등록된 SDK MCP 서버는 Claude Code가 요청을 처리하는 즉시 연결을 시작합니다.

## MCP 도구 검색

많은 MCP 도구가 구성되어 있으면 도구 정의가 컨텍스트 윈도우의 상당 부분을 소모할 수 있습니다. 도구 검색은 컨텍스트에서 도구 정의를 보류하고 각 턴마다 Claude에게 필요한 도구만 로드함으로써 이 문제를 해결합니다.

도구 검색은 기본적으로 활성화되어 있습니다. 구성 옵션, 모범 사례 및 커스텀 SDK 도구와 함께 도구 검색을 사용하는 방법은 [도구 검색](/docs/en/agent-sdk/tool-search)을 참조하세요.

## 인증

대부분의 MCP 서버는 외부 서비스에 접근하기 위해 인증이 필요합니다. 서버 구성의 환경 변수를 통해 자격 증명을 전달하세요.

### 환경 변수를 통해 자격 증명 전달하기

`env` 필드를 사용하여 API 키, 토큰 및 기타 자격 증명을 MCP 서버에 전달하세요:

<Tabs>
  <Tab title="코드 내부에서">
    <CodeGroup>
      ```typescript TypeScript hidelines={1,-1} theme={null}
      const _ = {
        options: {
          mcpServers: {
            "api-server": {
              command: "npx",
              args: ["-y", "@your-org/api-mcp-server"],
              env: {
                API_KEY: process.env.API_KEY
              }
            }
          },
          allowedTools: ["mcp__api-server__*"]
        }
      };
      ```

      ```python Python theme={null}
      options = ClaudeAgentOptions(
          mcp_servers={
              "api-server": {
                  "command": "npx",
                  "args": ["-y", "@your-org/api-mcp-server"],
                  "env": {"API_KEY": os.environ["API_KEY"]},
              }
          },
          allowed_tools=["mcp__api-server__*"],
      )
      ```
    </CodeGroup>
  </Tab>

  <Tab title=".mcp.json">
    ```json theme={null}
    {
      "mcpServers": {
        "api-server": {
          "command": "npx",
          "args": ["-y", "@your-org/api-mcp-server"],
          "env": {
            "API_KEY": "${API_KEY}"
          }
        }
      }
    }
    ```

    `${API_KEY}` 구문은 런타임에 환경 변수로 확장됩니다.
  </Tab>
</Tabs>

### 원격 서버용 HTTP 헤더

HTTP 및 SSE 서버의 경우 서버 구성에 인증 헤더를 직접 전달하세요:

<Tabs>
  <Tab title="코드 내부에서">
    <CodeGroup>
      ```typescript TypeScript hidelines={1,-1} theme={null}
      const _ = {
        options: {
          mcpServers: {
            "secure-api": {
              type: "http",
              url: "https://api.example.com/mcp",
              headers: {
                Authorization: `Bearer ${process.env.API_TOKEN}`
              }
            }
          },
          allowedTools: ["mcp__secure-api__*"]
        }
      };
      ```

      ```python Python theme={null}
      options = ClaudeAgentOptions(
          mcp_servers={
              "secure-api": {
                  "type": "http",
                  "url": "https://api.example.com/mcp",
                  "headers": {"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
              }
          },
          allowed_tools=["mcp__secure-api__*"],
      )
      ```
    </CodeGroup>
  </Tab>

  <Tab title=".mcp.json">
    ```json theme={null}
    {
      "mcpServers": {
        "secure-api": {
          "type": "http",
          "url": "https://api.example.com/mcp",
          "headers": {
            "Authorization": "Bearer ${API_TOKEN}"
          }
        }
      }
    }
    ```

    `${API_TOKEN}` 구문은 런타임에 환경 변수로 확장됩니다.
  </Tab>
</Tabs>

헤더로 인증된 원격 서버의 전체 작동 예제는 [리포지토리에서 이슈 목록 조회하기](#리포지토리에서-이슈-목록-조회하기)를 참조하세요.

### OAuth2 인증

[MCP 사양은 인가를 위해 OAuth 2.1을 지원합니다](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization). SDK는 브라우저를 열거나 대화형 OAuth 흐름을 실행하지 않습니다. 구성된 서버가 인가 챌린지를 반환하고 저장된 토큰을 이용할 수 없는 경우 해당 서버의 도구 없이 에이전트 실행이 진행되며 서버는 `needs-auth` 상태를 보고합니다. 기본적으로 서버는 백그라운드에서 연결되므로 [시스템 init 메시지](/docs/en/agent-sdk/typescript#sdksystemmessage)의 `mcp_servers` 배열에 해당 서버의 `pending` 상태가 계속 표시될 수 있습니다. 서버에 자격 증명이 필요한지 확인하려면 TypeScript SDK에서 `mcpServerStatus()`, Python에서 [`get_mcp_status()`](/docs/en/agent-sdk/python#methods)를 폴링하거나 `MCP_CONNECTION_NONBLOCKING=0`을 설정하여 init 메시지 전에 연결을 대기하세요.

자격 증명을 공급하려면 자체 애플리케이션에서 OAuth 흐름을 완료하고 결과 액세스 토큰을 서버의 `headers`에 전달하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  // 앱에서 OAuth 흐름을 완료한 후.
  // OAuth 제공자에 맞는 getAccessTokenFromOAuthFlow 구현
  const accessToken = await getAccessTokenFromOAuthFlow();

  const options = {
    mcpServers: {
      "oauth-api": {
        type: "http",
        url: "https://api.example.com/mcp",
        headers: {
          Authorization: `Bearer ${accessToken}`
        }
      }
    },
    allowedTools: ["mcp__oauth-api__*"]
  };
  ```

  ```python Python theme={null}
  # 앱에서 OAuth 흐름을 완료한 후.
  # OAuth 제공자에 맞는 get_access_token_from_oauth_flow 구현
  access_token = await get_access_token_from_oauth_flow()

  options = ClaudeAgentOptions(
      mcp_servers={
          "oauth-api": {
              "type": "http",
              "url": "https://api.example.com/mcp",
              "headers": {"Authorization": f"Bearer {access_token}"},
          }
      },
      allowed_tools=["mcp__oauth-api__*"],
  )
  ```
</CodeGroup>

## 예제

### 리포지토리에서 이슈 목록 조회하기

이 예제는 원격 [GitHub MCP 서버](https://github.com/github/github-mcp-server)에 연결하여 최근 이슈 목록을 조회합니다. 이 예제에는 MCP 연결 및 도구 호출을 검증하기 위한 디버그 로깅이 포함되어 있습니다.

실행하기 전에 조회하려는 리포지토리에 대한 읽기 접근 권한이 있는 [GitHub 개인용 액세스 토큰](https://github.com/settings/personal-access-tokens)을 생성하고 이를 환경 변수로 설정하세요:

```bash theme={null}
export GITHUB_TOKEN=YOUR_GITHUB_PAT
```

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "List the 3 most recent issues in anthropics/claude-code",
    options: {
      mcpServers: {
        github: {
          type: "http",
          url: "https://api.githubcopilot.com/mcp/",
          headers: {
            Authorization: `Bearer ${process.env.GITHUB_TOKEN}`
          }
        }
      },
      allowedTools: ["mcp__github__list_issues"]
    }
  })) {
    // MCP 서버가 성공적으로 연결되었는지 검증
    if (message.type === "system" && message.subtype === "init") {
      console.log("MCP servers:", message.mcp_servers);
    }

    // Claude가 MCP 도구를 호출할 때 로깅
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if (block.type === "tool_use" && block.name.startsWith("mcp__")) {
          console.log("MCP tool called:", block.name);
        }
      }
    }

    // 최종 결과 출력
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  import os
  from claude_agent_sdk import (
      query,
      ClaudeAgentOptions,
      ResultMessage,
      SystemMessage,
      AssistantMessage,
  )


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={
              "github": {
                  "type": "http",
                  "url": "https://api.githubcopilot.com/mcp/",
                  "headers": {"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"},
              }
          },
          allowed_tools=["mcp__github__list_issues"],
      )

      async for message in query(
          prompt="List the 3 most recent issues in anthropics/claude-code",
          options=options,
      ):
          # MCP 서버가 성공적으로 연결되었는지 검증
          if isinstance(message, SystemMessage) and message.subtype == "init":
              print("MCP servers:", message.data.get("mcp_servers"))

          # Claude가 MCP 도구를 호출할 때 로깅
          if isinstance(message, AssistantMessage):
              for block in message.content:
                  if hasattr(block, "name") and block.name.startswith("mcp__"):
                      print("MCP tool called:", block.name)

          # 최종 결과 출력
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```
</CodeGroup>

### 데이터베이스 쿼리하기

이 예제는 Postgres 데이터베이스를 쿼리하기 위해 [DBHub](https://github.com/bytebase/dbhub)를 사용합니다. 에이전트는 데이터베이스 스키마를 자동으로 탐색하고 SQL 쿼리를 작성하여 결과를 반환합니다.

DBHub의 `execute_sql` 도구는 제한하지 않는 한 쓰기를 포함하여 에이전트가 방출하는 모든 SQL을 실행합니다. [DBHub 구성 파일](https://dbhub.ai/config/toml)에 `readonly = true`를 설정하면 DBHub가 `INSERT`, `UPDATE`, `DELETE` 및 DDL 문을 거부하므로 에이전트가 쓰기 작업을 수행하려 해도 데이터가 수정되지 않습니다. DBHub는 구성을 로드할 때 프로세스 환경에서 `${DATABASE_URL}`을 해석하므로 연결 문자열이 파일에 노출되지 않습니다. 스크립트 옆에 다음 `dbhub.toml`을 만드세요:

```toml dbhub.toml theme={null}
[[sources]]
id = "production"
dsn = "${DATABASE_URL}"

[[tools]]
name = "execute_sql"
source = "production"
readonly = true
```

그러면 스크립트는 연결 문자열을 직접 전달하는 대신 DBHub에 구성 파일을 지정합니다. 실행하기 전에 `DATABASE_URL` 환경 변수를 데이터베이스 연결 문자열로 설정하세요. 자리표시자 값을 자체 데이터베이스 정보로 교체하세요:

```bash theme={null}
export DATABASE_URL=postgresql://user:password@localhost:5432/mydb
```

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    // 자연어 쿼리 - Claude가 SQL을 작성함
    prompt: "How many users signed up last week? Break it down by day.",
    options: {
      mcpServers: {
        postgres: {
          command: "npx",
          // dbhub.toml에 readonly = true가 설정되어 execute_sql이 쓰기를 거부함
          args: ["-y", "@bytebase/dbhub", "--config", "dbhub.toml"]
        }
      },
      allowedTools: ["mcp__postgres__execute_sql"]
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={
              "postgres": {
                  "command": "npx",
                  # dbhub.toml에 readonly = true가 설정되어 execute_sql이 쓰기를 거부함
                  "args": [
                      "-y",
                      "@bytebase/dbhub",
                      "--config",
                      "dbhub.toml",
                  ],
              }
          },
          allowed_tools=["mcp__postgres__execute_sql"],
      )

      # 자연어 쿼리 - Claude가 SQL을 작성함
      async for message in query(
          prompt="How many users signed up last week? Break it down by day.",
          options=options,
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```
</CodeGroup>

## 오류 처리

MCP 서버는 여러 이유로 연결에 실패할 수 있습니다: 서버 프로세스가 설치되지 않았거나, 자격 증명이 유효하지 않거나, 원격 서버에 접근할 수 없는 경우 등입니다.

SDK는 각 쿼리 시작 시 하위 유형이 `init`인 `system` 메시지를 방출합니다. 이 메시지에는 각 MCP 서버의 연결 상태가 포함됩니다. `status` 필드는 `"pending"`, `"connected"`, `"failed"`, `"needs-auth"`, `"disabled"`일 수 있습니다. 기본적으로 연결은 [비차단 방식](#연결-타이밍)이므로 init 메시지가 방출될 때 정상적인 서버도 `"pending"`을 보고하는 경우가 많습니다. 사용할 수 없는 서버를 감지하려면 `"pending"`을 실패로 처리하지 말고 `"failed"` 또는 `"needs-auth"`를 확인하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  try {
    for await (const message of query({
      prompt: "Process data",
      options: {
        mcpServers: {
          // dataServer를 자체 서버 구성으로 교체
          "data-processor": dataServer
        }
      }
    })) {
      if (message.type === "system" && message.subtype === "init") {
        const unavailableServers = message.mcp_servers.filter(
          (s) => s.status === "failed" || s.status === "needs-auth"
        );

        if (unavailableServers.length > 0) {
          console.warn("Unavailable MCP servers:", unavailableServers);
        }
      }

      if (message.type === "result" && message.subtype === "error_during_execution") {
        console.error("Execution failed");
      }
    }
  } catch (error) {
    // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
    // 오류 결과였다면 위의 오류 하위 유형 분기가 이미 실행된 상태입니다.
    // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
    console.log(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage, ResultMessage


  async def main():
      # data_server를 자체 서버 구성으로 교체
      options = ClaudeAgentOptions(mcp_servers={"data-processor": data_server})

      try:
          async for message in query(prompt="Process data", options=options):
              if isinstance(message, SystemMessage) and message.subtype == "init":
                  unavailable_servers = [
                      s
                      for s in message.data.get("mcp_servers", [])
                      if s.get("status") in ("failed", "needs-auth")
                  ]

                  if unavailable_servers:
                      print(f"Unavailable MCP servers: {unavailable_servers}")

              if (
                  isinstance(message, ResultMessage)
                  and message.subtype == "error_during_execution"
              ):
                  print("Execution failed")
      except Exception as error:
          # 단발성 query()는 오류 결과를 생성한 후 예외를 발생시킵니다.
          # 오류 결과였다면 위의 오류 하위 유형 분기가 이미 실행된 상태입니다.
          # 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

## 문제 해결

### 서버 상태가 "failed"로 표시됨

`init` 메시지를 확인하여 연결에 실패한 서버를 파악하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  if (message.type === "system" && message.subtype === "init") {
    for (const server of message.mcp_servers) {
      if (server.status === "failed") {
        console.error(`Server ${server.name} failed to connect`);
      }
    }
  }
  ```

  ```python Python theme={null}
  if isinstance(message, SystemMessage) and message.subtype == "init":
      for server in message.data.get("mcp_servers", []):
          if server.get("status") == "failed":
              print(f"Server {server['name']} failed to connect")
  ```
</CodeGroup>

`"pending"` 상태는 서버가 연결 실패한 것이 아니라 아직 연결 중임을 의미합니다. 세션 후반에 업데이트된 상태를 구하려면 TypeScript SDK에서 쿼리의 `mcpServerStatus()` 메서드를 호출하거나, Python에서 [`ClaudeSDKClient.get_mcp_status()`](/docs/en/agent-sdk/python#methods)를 호출하세요.

일반적인 원인:

* **누락된 환경 변수**: 필요한 토큰 및 자격 증명이 설정되었는지 확인하세요. stdio 서버의 경우 `env` 필드가 서버에서 기대하는 값과 일치하는지 확인하세요.
* **서버가 설치되지 않음**: `npx` 명령의 경우 패키지가 존재하고 Node.js가 PATH에 포함되어 있는지 확인하세요.
* **잘못된 연결 문자열**: 데이터베이스 서버의 경우 연결 문자열 형식 및 데이터베이스 접근 가능 여부를 확인하세요.
* **네트워크 문제**: 원격 HTTP/SSE 서버의 경우 URL에 접근 가능한지, 방화벽이 연결을 허용하는지 확인하세요.

### 도구가 호출되지 않음

Claude에 도구가 보이지만 사용하지 않는 경우 `allowedTools`를 통해 권한을 부여했는지 확인하세요:

<CodeGroup>
  ```typescript TypeScript hidelines={1,-1} theme={null}
  const _ = {
    options: {
      mcpServers: {
        // 내 서버 목록
      },
      allowedTools: ["mcp__servername__*"] // 이 서버의 호출 자동 승인
    }
  };
  ```

  ```python Python theme={null}
  options = ClaudeAgentOptions(
      mcp_servers={
          # 내 서버 목록
      },
      allowed_tools=["mcp__servername__*"],  # 이 서버의 호출 자동 승인
  )
  ```
</CodeGroup>

### 연결 타임아웃

MCP 서버 연결은 기본적으로 30초 후 타임아웃됩니다. 서버 시작에 이보다 오랜 시간이 걸리면 연결이 실패합니다. [`MCP_TIMEOUT`](/docs/en/env-vars) 환경 변수를 밀리초 단위로 설정하여 제한을 늘리세요. 시작 시간이 더 필요한 서버의 경우 다음 사항도 고려하세요:

* 사용 가능한 경우 더 가벼운 서버 사용
* 에이전트를 시작하기 전에 서버 웜업 진행
* 느린 초기화 원인에 대해 서버 로그 확인

### 도구 출력이 최대 허용 토큰을 초과함

SDK는 Claude Code와 동일한 MCP 출력 제한을 적용합니다. 도구 결과가 25,000토큰보다 크면 전체 출력이 파일로 저장되고 도구 결과는 파일 경로가 명시된 오류 메시지로 교체되어 에이전트가 출력을 부분별로 다시 읽을 수 있게 합니다. [`MAX_MCP_OUTPUT_TOKENS`](/docs/en/env-vars) 환경 변수로 제한을 늘리세요. 디스크 지속성 폴백 및 `anthropic/maxResultSizeChars` 도구별 주석을 포함한 전체 동작은 [MCP 출력 제한 및 경고](/docs/en/mcp#mcp-output-limits-and-warnings)를 참조하세요.

## 관련 리소스

* **[커스텀 도구 가이드](/docs/en/agent-sdk/custom-tools)**: SDK 애플리케이션 내부에서 인프로세스로 실행되는 자체 MCP 서버 구축
* **[권한](/docs/en/agent-sdk/permissions)**: `allowedTools` 및 `disallowedTools`로 에이전트가 사용할 수 있는 MCP 도구 제어
* **[MCP 출력 제한 및 경고](/docs/en/mcp#mcp-output-limits-and-warnings)**: `MAX_MCP_OUTPUT_TOKENS`를 초과하는 도구 결과를 SDK가 처리하는 방식
* **[TypeScript SDK 참조](/docs/en/agent-sdk/typescript)**: MCP 구성 옵션을 포함한 전체 API 참조
* **[Python SDK 참조](/docs/en/agent-sdk/python)**: MCP 구성 옵션을 포함한 전체 API 참조
* **[MCP 서버 디렉토리](https://github.com/modelcontextprotocol/servers)**: 데이터베이스, API 등을 위한 사용 가능한 MCP 서버 둘러보기
