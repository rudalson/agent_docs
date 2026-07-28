> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Agent SDK 개요

> 라이브러리 형태의 Claude Code를 활용해 프로덕션급 AI 에이전트 구축하기

자율적으로 파일을 읽고, 명령을 실행하며, 웹을 검색하고, 코드를 수정하는 등의 작업을 수행하는 AI 에이전트를 구축하세요. Agent SDK는 Python 및 TypeScript로 프로그래밍할 수 있으며, Claude Code를 구동하는 것과 동일한 도구, 에이전트 루프, 컨텍스트 관리 기능을 제공합니다. 다른 언어의 경우 `-p` 플래그 및 `--output-format json` 옵션을 사용하여 [프로그래밍 방식으로 CLI를 실행](/docs/en/headless)할 수 있습니다. 에이전트 하네스(harness) 설계의 뒷이야기가 궁금하다면 블로그의 [A harness for every task: dynamic workflows in Claude Code](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code) 글을 참고하세요. 아래 예제를 실행하려면 먼저 [시작하기](#get-started)의 단계에 따라 SDK를 설치하세요.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions


  async def main():
      async for message in query(
          prompt="Find and fix the bug in auth.py",
          options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
      ):
          print(message)  # Claude reads the file, finds the bug, edits it


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Find and fix the bug in auth.ts",
    options: { allowedTools: ["Read", "Edit", "Bash"] }
  })) {
    console.log(message); // Claude reads the file, finds the bug, edits it
  }
  ```
</CodeGroup>

Agent SDK에는 파일 읽기, 명령 실행, 코드 편집을 위한 내장 도구가 포함되어 있으므로 도구 실행 로직을 별도로 구현하지 않고도 에이전트 작업을 즉시 시작할 수 있습니다. 퀵스타트로 이동하거나 SDK로 구축된 실제 에이전트를 탐색해 보세요:

<CardGroup cols={2}>
  <Card title="퀵스타트" icon="play" href="/docs/en/agent-sdk/quickstart">
    몇 분 만에 버그 수정 에이전트 구축하기
  </Card>

  <Card title="에이전트 예제" icon="star" href="https://github.com/anthropics/claude-agent-sdk-demos">
    이메일 어시스턴트, 리서치 에이전트 등
  </Card>
</CardGroup>

## 시작하기

<Steps>
  <Step title="SDK 설치">
    <Tabs>
      <Tab title="TypeScript">
        ```bash theme={null}
        npm init -y
        npm pkg set type=module
        npm install @anthropic-ai/claude-agent-sdk
        npm install --save-dev tsx
        ```

        `package.json`에 `"type": "module"`을 설정하면 에이전트 스크립트에서 최상위 `await`를 사용할 수 있으며, [tsx](https://tsx.is)를 사용해 TypeScript 파일을 직접 실행할 수 있습니다. 기존 CommonJS 프로젝트에서는 처음 두 명령을 건너뛰고 스크립트 이름을 `agent.ts` 대신 `agent.mts`로 지정하세요.
      </Tab>

      <Tab title="Python (uv)">
        [uv](https://docs.astral.sh/uv/)는 가상 환경을 자동으로 관리해 주는 빠른 Python 패키지 관리자입니다:

        ```bash theme={null}
        uv init
        uv add claude-agent-sdk
        ```
      </Tab>

      <Tab title="Python (pip)">
        가상 환경을 생성 및 활성화한 다음 패키지를 설치합니다. 가상 환경에 설치하면 최근 Debian, Ubuntu, Homebrew 환경의 시스템 Python에서 venv 외부 `pip install` 시 발생하는 `error: externally-managed-environment` 오류를 방지할 수 있습니다.

        macOS 또는 Linux:

        ```bash theme={null}
        python3 -m venv .venv
        source .venv/bin/activate
        pip install claude-agent-sdk
        ```

        Windows:

        ```powershell theme={null}
        py -m venv .venv
        .venv\Scripts\Activate.ps1
        pip install claude-agent-sdk
        ```

        PowerShell에서 실행 정책 오류로 `Activate.ps1`이 차단되는 경우 먼저 `Set-ExecutionPolicy -Scope Process RemoteSigned`를 실행하세요.

        Python 패키지에는 Python 3.10 이상이 필요합니다. pip에서 `No matching distribution found for claude-agent-sdk`라고 보고되면 인터프리터 버전이 3.10보다 낮은 것입니다. macOS나 Linux에서는 `python3 --version`, Windows에서는 `py --version`을 실행하여 버전을 확인하세요.
      </Tab>
    </Tabs>

    <Note>
      TypeScript 및 Python SDK 모두 해당 플랫폼용 네이티브 Claude Code 바이너리를 번들로 제공하므로 Claude Code를 별도로 설치할 필요가 없습니다.
    </Note>
  </Step>

  <Step title="API 키 설정">
    [Console](https://platform.claude.com/)에서 API 키를 발급받은 후 환경 변수로 설정합니다.

    macOS 또는 Linux:

    ```bash theme={null}
    export ANTHROPIC_API_KEY=sk-ant-xxxxx
    ```

    Windows PowerShell:

    ```powershell theme={null}
    $env:ANTHROPIC_API_KEY = "sk-ant-xxxxx"
    ```

    SDK는 써드파티 API 제공자를 통한 인증도 지원합니다:

    * **Amazon Bedrock**: `CLAUDE_CODE_USE_BEDROCK=1` 환경 변수를 설정하고 AWS 자격 증명 구성
    * **Claude Platform on AWS**: `CLAUDE_CODE_USE_ANTHROPIC_AWS=1` 및 `ANTHROPIC_AWS_WORKSPACE_ID`를 설정한 후 AWS 자격 증명 구성
    * **Google Cloud's Agent Platform**: `CLAUDE_CODE_USE_VERTEX=1` 환경 변수를 설정하고 Google Cloud 자격 증명 구성
    * **Microsoft Foundry**: `CLAUDE_CODE_USE_FOUNDRY=1` 환경 변수를 설정하고 Azure 자격 증명 구성

    자세한 내용은 [Amazon Bedrock](/docs/en/amazon-bedrock), [Claude Platform on AWS](/docs/en/claude-platform-on-aws), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry) 설정 가이드를 참고하세요.

    <Note>
      사전 승인을 받은 경우가 아니면, Anthropic은 서드파티 개발자가 Claude Agent SDK로 구축된 에이전트를 포함한 자체 제품에 claude.ai 로그인이나 레이트 리밋(rate limit)을 제공하는 것을 허용하지 않습니다. 본 문서에 설명된 API 키 인증 방법을 대신 사용해 주세요.
    </Note>
  </Step>

  <Step title="첫 번째 에이전트 실행">
    이 예제는 내장 도구를 사용하여 현재 디렉토리의 파일 목록을 출력하는 에이전트를 생성합니다.

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions


      async def main():
          async for message in query(
              prompt="What files are in this directory?",
              options=ClaudeAgentOptions(allowed_tools=["Bash", "Glob"]),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      for await (const message of query({
        prompt: "What files are in this directory?",
        options: { allowedTools: ["Bash", "Glob"] }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>

    예제 코드를 `agent.py` 또는 `agent.ts`로 저장하고 실행합니다. 에이전트가 디렉토리 내 파일에 대한 간단한 요약을 출력합니다.

    <Tabs>
      <Tab title="TypeScript">
        ```bash theme={null}
        npx tsx agent.ts
        ```

        CommonJS 프로젝트를 위해 스크립트 이름을 `agent.mts`로 지정한 경우 대신 `npx tsx agent.mts`를 실행하세요.
      </Tab>

      <Tab title="Python (uv)">
        ```bash theme={null}
        uv run agent.py
        ```
      </Tab>

      <Tab title="Python (pip)">
        가상 환경이 활성화된 상태에서 macOS 또는 Linux의 경우:

        ```bash theme={null}
        python3 agent.py
        ```

        Windows에서는 `python agent.py`를 실행하세요.
      </Tab>
    </Tabs>
  </Step>
</Steps>

**구축할 준비가 되셨나요?** [퀵스타트](/docs/en/agent-sdk/quickstart)를 따라 몇 분 만에 버그를 찾아 수정하는 에이전트를 만들어 보세요.

## 기능

Claude Code를 강력하게 만드는 모든 기능을 SDK에서도 사용할 수 있습니다:

<Tabs>
  <Tab title="내장 도구">
    에이전트는 설치 즉시 바로 파일을 읽고, 명령을 실행하며, 코드베이스를 검색할 수 있습니다. 주요 도구는 다음과 같습니다:

    | 도구 | 기능 |
    | --------------------------------------------------------------------------- | ------------------------------------------------------------------- |
    | **Read** | 작업 디렉토리의 모든 파일 읽기 |
    | **Write** | 새 파일 생성 |
    | **Edit** | 기존 파일 정밀 편집 |
    | **Bash** | 터미널 명령, 스크립트, git 작업 실행 |
    | **Monitor** | 백그라운드 스크립트를 감시하고 각 출력 줄을 이벤트로 감지 |
    | **Glob** | 패턴(`**/*.ts`, `src/**/*.py`)으로 파일 찾기 |
    | **Grep** | 정규표현식으로 파일 내용 검색 |
    | **WebSearch** | 최신 정보를 얻기 위해 웹 검색 |
    | **WebFetch** | 웹 페이지 콘텐츠를 가져와 파싱 |
    | **[AskUserQuestion](/docs/en/agent-sdk/user-input#handle-clarifying-questions)** | 다중 선택 옵션으로 사용자에게 명확히 하기 위한 질문하기 |

    스케줄링 및 작업 트리(worktree) 도구를 포함한 전체 목록은 [도구 참조 문서](/docs/en/tools-reference)를 참고하세요.

    다음 예제는 코드베이스에서 TODO 주석을 검색하는 에이전트를 만듭니다:

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions


      async def main():
          async for message in query(
              prompt="Find all TODO comments and create a summary",
              options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"]),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      for await (const message of query({
        prompt: "Find all TODO comments and create a summary",
        options: { allowedTools: ["Read", "Glob", "Grep"] }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>
  </Tab>

  <Tab title="후크 (Hooks)">
    에이전트 수명주기의 핵심 시점에서 커스텀 코드를 실행합니다. SDK 후크는 콜백 함수를 사용하여 에이전트 동작을 검증, 로깅, 차단 또는 변환합니다.

    **사용 가능한 후크:** `PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`, `SessionEnd`, `UserPromptSubmit` 등.

    다음 예제는 모든 파일 변경 사항을 감사 파일에 기록합니다:

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from datetime import datetime
      from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher


      async def log_file_change(input_data, tool_use_id, context):
          file_path = input_data.get("tool_input", {}).get("file_path", "unknown")
          with open("./audit.log", "a") as f:
              f.write(f"{datetime.now()}: modified {file_path}\n")
          return {}


      async def main():
          async for message in query(
              prompt="Create a file named hello.py that prints a greeting",
              options=ClaudeAgentOptions(
                  allowed_tools=["Read", "Edit"],
                  permission_mode="acceptEdits",
                  hooks={
                      "PostToolUse": [
                          HookMatcher(matcher="Edit|Write", hooks=[log_file_change])
                      ]
                  },
              ),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query, HookCallback } from "@anthropic-ai/claude-agent-sdk";
      import { appendFile } from "fs/promises";

      const logFileChange: HookCallback = async (input) => {
        const filePath = (input as any).tool_input?.file_path ?? "unknown";
        await appendFile("./audit.log", `${new Date().toISOString()}: modified ${filePath}\n`);
        return {};
      };

      for await (const message of query({
        prompt: "Create a file named hello.ts that prints a greeting",
        options: {
          allowedTools: ["Read", "Edit"],
          permissionMode: "acceptEdits",
          hooks: {
            PostToolUse: [{ matcher: "Edit|Write", hooks: [logFileChange] }]
          }
        }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>

    에이전트가 종료된 후 `cat audit.log`를 실행하여 기록된 파일 변경 사항을 확인하세요.

    [후크에 대해 더 알아보기 →](/docs/en/agent-sdk/hooks)
  </Tab>

  <Tab title="서브에이전트 (Subagents)">
    특화된 하위 작업을 처리할 전문 서브에이전트를 생성합니다. 메인 에이전트가 작업을 위임하면 서브에이전트가 결과를 다시 보고합니다.

    특화된 지침이 담긴 커스텀 에이전트를 정의하세요. 서브에이전트는 Agent 도구를 통해 호출되므로 `allowedTools`에 `Agent`를 포함하여 해당 호출을 자동 승인하도록 하세요:

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


      async def main():
          async for message in query(
              prompt="Use the code-reviewer agent to review this codebase",
              options=ClaudeAgentOptions(
                  allowed_tools=["Read", "Glob", "Grep", "Agent"],
                  agents={
                      "code-reviewer": AgentDefinition(
                          description="Expert code reviewer for quality and security reviews.",
                          prompt="Analyze code quality and suggest improvements.",
                          tools=["Read", "Glob", "Grep"],
                      )
                  },
              ),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      for await (const message of query({
        prompt: "Use the code-reviewer agent to review this codebase",
        options: {
          allowedTools: ["Read", "Glob", "Grep", "Agent"],
          agents: {
            "code-reviewer": {
              description: "Expert code reviewer for quality and security reviews.",
              prompt: "Analyze code quality and suggest improvements.",
              tools: ["Read", "Glob", "Grep"]
            }
          }
        }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>

    서브에이전트 컨텍스트 내부에서 전송된 메시지에는 `parent_tool_use_id` 필드가 포함되어 어떤 메시지가 어떤 서브에이전트 실행에 속하는지 추적할 수 있습니다.

    [서브에이전트에 대해 더 알아보기 →](/docs/en/agent-sdk/subagents)
  </Tab>

  <Tab title="MCP">
    Model Context Protocol을 통해 데이터베이스, 브라우저, API 및 [수백 가지 이상의 다른 외부 시스템](https://github.com/modelcontextprotocol/servers)에 연결합니다.

    다음 예제는 [Playwright MCP 서버](https://github.com/microsoft/playwright-mcp)를 연결하여 에이전트에 브라우저 자동화 기능을 부여합니다:

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions


      async def main():
          async for message in query(
              prompt="Open example.com and describe what you see",
              options=ClaudeAgentOptions(
                  mcp_servers={
                      "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
                  },
                  allowed_tools=["mcp__playwright__*"],
              ),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      for await (const message of query({
        prompt: "Open example.com and describe what you see",
        options: {
          mcpServers: {
            playwright: { command: "npx", args: ["@playwright/mcp@latest"] }
          },
          allowedTools: ["mcp__playwright__*"]
        }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>

    [MCP에 대해 더 알아보기 →](/docs/en/agent-sdk/mcp)
  </Tab>

  <Tab title="권한">
    에이전트가 사용할 수 있는 도구를 정밀하게 제어합니다. 안전한 작업은 허용하고, 위험한 작업은 차단하며, 민감한 작업에는 승인을 요구할 수 있습니다.

    <Note>
      대화형 승인 프롬프트 및 `AskUserQuestion` 도구에 대한 내용은 [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input)를 참고하세요.
    </Note>

    다음 예제는 코드를 분석할 수만 있고 수정할 수는 없는 읽기 전용 에이전트를 만듭니다. `allowed_tools`는 `Read`, `Glob`, `Grep`을 미리 승인하여 프롬프트 없이 실행되도록 합니다. 목록에 없는 도구는 여전히 사용 가능하지만 권한 모드로 넘어가 처리됩니다. 도구를 완전히 차단하려면 `disallowed_tools`를 사용하세요.

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions


      async def main():
          async for message in query(
              prompt="Review this code for best practices",
              options=ClaudeAgentOptions(
                  allowed_tools=["Read", "Glob", "Grep"],
              ),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      for await (const message of query({
        prompt: "Review this code for best practices",
        options: {
          allowedTools: ["Read", "Glob", "Grep"]
        }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>

    [권한에 대해 더 알아보기 →](/docs/en/agent-sdk/permissions)
  </Tab>

  <Tab title="세션">
    여러 대화 주고받기 과정에서 컨텍스트를 유지합니다. Claude는 읽은 파일, 수행된 분석, 대화 기록을 기억합니다. 나중에 세션을 재개하거나, 다른 접근 방식을 탐색하기 위해 세션을 분기(fork)할 수 있습니다.

    다음 예제는 첫 번째 쿼리에서 세션 ID를 캡처한 다음, 전체 컨텍스트를 유지한 채 재개하는 방법을 보여줍니다:

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage, ResultMessage


      async def main():
          session_id = None

          # First query: capture the session ID
          try:
              async for message in query(
                  prompt="Read the authentication module",
                  options=ClaudeAgentOptions(allowed_tools=["Read", "Glob"]),
              ):
                  if isinstance(message, SystemMessage) and message.subtype == "init":
                      session_id = message.data["session_id"]
          except Exception as error:
              # A single-shot query() raises after yielding an error result. If
              # the failure was an error result, session_id was already captured
              # by the loop above; connection or process failures yield no
              # result message.
              print(f"Session ended with an error: {error}")

          # Resume with full context from the first query
          async for message in query(
              prompt="Now find all places that call it",  # "it" = auth module
              options=ClaudeAgentOptions(resume=session_id),
          ):
              if isinstance(message, ResultMessage):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      let sessionId: string | undefined;

      // First query: capture the session ID
      try {
        for await (const message of query({
          prompt: "Read the authentication module",
          options: { allowedTools: ["Read", "Glob"] }
        })) {
          if (message.type === "system" && message.subtype === "init") {
            sessionId = message.session_id;
          }
        }
      } catch (error) {
        // A single-shot query() throws after yielding an error result. If the
        // failure was an error result, sessionId was already captured by the
        // loop above; connection or process failures yield no result message.
        console.error(`Session ended with an error: ${error}`);
      }

      // Resume with full context from the first query
      for await (const message of query({
        prompt: "Now find all places that call it", // "it" = auth module
        options: { resume: sessionId }
      })) {
        if ("result" in message) console.log(message.result);
      }
      ```
    </CodeGroup>

    [세션에 대해 더 알아보기 →](/docs/en/agent-sdk/sessions)
  </Tab>
</Tabs>

### Claude Code 기능

SDK는 Claude Code의 파일 시스템 기반 구성도 지원합니다. 기본 옵션을 사용할 때 SDK는 작업 디렉토리의 `.claude/` 및 `~/.claude/`에서 이러한 설정을 로드합니다. 로드되는 소스를 제한하려면 옵션에서 `setting_sources` (Python) 또는 `settingSources` (TypeScript)를 설정하세요.

| 기능 | 설명 | 위치 |
| ------------------------------------------------ | ----------------------------------------------------------------------------- | ---------------------------------- |
| [스킬 (Skills)](/docs/en/agent-sdk/skills) | Claude가 자동으로 사용하거나 `/name`으로 직접 호출하는 전문 기능 | `.claude/skills/*/SKILL.md` |
| [명령어 (Commands)](/docs/en/agent-sdk/slash-commands) | 레거시 형식의 커스텀 명령어. 신규 커스텀 명령어에는 스킬을 사용하세요 | `.claude/commands/*.md` |
| [메모리 (Memory)](/docs/en/agent-sdk/modifying-system-prompts) | 프로젝트 컨텍스트 및 지침 | `CLAUDE.md` 또는 `.claude/CLAUDE.md` |
| [플러그인 (Plugins)](/docs/en/agent-sdk/plugins) | 스킬, 에이전트, 후크, MCP 서버 확장 | `plugins` 옵션을 통한 프로그래밍 방식 |

## 다른 Claude 도구와의 Agent SDK 비교

Claude Platform은 Claude를 활용해 구축할 수 있는 다양한 방법을 제공합니다. Agent SDK가 어디에 적합한지 비교해 보세요:

<Tabs>
  <Tab title="Agent SDK vs Client SDK">
    [Anthropic Client SDK](https://platform.claude.com/docs/en/api/client-sdks)는 직접적인 API 액세스를 제공하므로 프롬프트를 전송하고 도구 실행을 직접 구현해야 합니다. **Agent SDK**는 도구 실행 기능이 내장된 Claude를 제공합니다.

    Client SDK를 사용하는 경우 도구 실행 루프를 직접 작성해야 합니다. Agent SDK를 사용하면 Claude가 자율적으로 처리합니다. 다음 의도적 의사 코드는 그 차이를 보여줍니다:

    <CodeGroup>
      ```python Python theme={null}
      # Client SDK: You implement the tool loop
      response = client.messages.create(...)
      while response.stop_reason == "tool_use":
          result = your_tool_executor(response.tool_use)
          response = client.messages.create(tool_result=result, **params)

      # Agent SDK: Claude handles tools autonomously
      async for message in query(prompt="Fix the bug in auth.py"):
          print(message)
      ```

      ```typescript TypeScript theme={null}
      // Client SDK: You implement the tool loop
      let response = await client.messages.create({ ...params });
      while (response.stop_reason === "tool_use") {
        const result = yourToolExecutor(response.tool_use);
        response = await client.messages.create({ tool_result: result, ...params });
      }

      // Agent SDK: Claude handles tools autonomously
      for await (const message of query({ prompt: "Fix the bug in auth.ts" })) {
        console.log(message);
      }
      ```
    </CodeGroup>
  </Tab>

  <Tab title="Agent SDK vs Claude Code CLI">
    동일한 기능, 다른 인터페이스:

    | 사용 사례 | 최선의 선택 |
    | ----------------------- | ----------- |
    | 대화형 개발 | CLI |
    | CI/CD 파이프라인 | SDK |
    | 커스텀 애플리케이션 | SDK |
    | 일회성 작업 | CLI |
    | 프로덕션 자동화 | SDK |

    많은 팀이 두 가지를 모두 사용합니다: 일상 개발에는 CLI를, 프로덕션에는 SDK를 사용합니다. 워크플로는 서로 직접 호환됩니다.
  </Tab>

  <Tab title="Agent SDK vs Managed Agents">
    [Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview)는 호스팅된 REST API입니다: Anthropic이 에이전트와 샌드박스를 실행하고 애플리케이션은 이벤트를 전송하고 결과를 스트리밍받습니다. **Agent SDK**는 자체 프로세스 내에서 에이전트 루프를 실행하는 라이브러리입니다.

    | | Agent SDK | Managed Agents |
    | ------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
    | **실행 위치** | 귀하의 프로세스, 귀하의 인프라 | Anthropic 관리 인프라 |
    | **인터페이스** | Python 또는 TypeScript 라이브러리 | REST API |
    | **작업 대상** | 귀하 인프라 상의 파일 | 세션당 관리형 샌드박스 |
    | **세션 상태** | 귀하의 파일 시스템에 저장되는 JSONL | Anthropic 호스팅 이벤트 로그 |
    | **커스텀 도구** | 프로세스 내 Python 또는 TypeScript 함수 | Claude가 도구를 트리거하면 사용자가 실행 후 결과 반환 |
    | **적합한 분야** | 로컬 프로토타이핑, 사용자 파일 시스템 및 서비스에서 직접 작업하는 에이전트 | 샌드박스나 세션 인프라 운영 없이 구축하는 프로덕션 에이전트, 장시간 실행되거나 비동기적인 세션 |

    일반적인 경로는 로컬에서 Agent SDK로 프로토타입을 제작한 다음 프로덕션용으로 Managed Agents로 이동하는 것입니다.
  </Tab>
</Tabs>

## 변경 이력 (Changelog)

SDK 업데이트, 버그 수정 및 신규 기능에 대한 전체 변경 이력을 확인하세요:

* **TypeScript SDK**: [CHANGELOG.md 보기](https://github.com/anthropics/claude-agent-sdk-typescript/blob/main/CHANGELOG.md)
* **Python SDK**: [CHANGELOG.md 보기](https://github.com/anthropics/claude-agent-sdk-python/blob/main/CHANGELOG.md)

## 버그 보고

Agent SDK 사용 중 버그나 문제가 발생한 경우:

* **TypeScript SDK**: [GitHub 이슈 제보](https://github.com/anthropics/claude-agent-sdk-typescript/issues)
* **Python SDK**: [GitHub 이슈 제보](https://github.com/anthropics/claude-agent-sdk-python/issues)

## 브랜딩 가이드라인

Claude Agent SDK를 연동하는 파트너의 경우 Claude 브랜딩 사용은 선택 사항입니다. 제품 내에서 Claude를 참조할 때:

**허용됨:**

* "Claude Agent" (드롭다운 메뉴에 권장)
* "Claude" (이미 "Agents"로 라벨 지정된 메뉴 내부일 때)
* "{YourAgentName} Powered by Claude" (기존 에이전트 이름이 있는 경우)

**허용되지 않음:**

* "Claude Code" 또는 "Claude Code Agent"
* Claude Code 브랜딩이 적용된 ASCII 아트나 Claude Code를 흉내 낸 시각적 요소

귀하의 제품은 독자적인 브랜딩을 유지해야 하며 Claude Code나 Anthropic 제품처럼 보여서는 안 됩니다. 브랜딩 준수에 관한 질문은 Anthropic [영업팀](https://www.anthropic.com/contact-sales)에 문의하세요.

## 라이선스 및 약관

Claude Agent SDK 사용은 해당 구성 요소의 LICENSE 파일에 달리 명시된 라이선스가 적용되는 특정 구성 요소나 종속성을 제외하고, 제품 및 서비스를 구축하여 귀하의 고객 및 최종 사용자에게 제공하는 경우를 포함하여 [Anthropic 상용 서비스 약관](https://www.anthropic.com/legal/commercial-terms)의 적용을 받습니다.

## 다음 단계

<CardGroup cols={2}>
  <Card title="퀵스타트" icon="play" href="/docs/en/agent-sdk/quickstart">
    몇 분 만에 버그를 찾아 수정하는 에이전트 구축하기
  </Card>

  <Card title="에이전트 예제" icon="star" href="https://github.com/anthropics/claude-agent-sdk-demos">
    이메일 어시스턴트, 리서치 에이전트 등
  </Card>

  <Card title="TypeScript SDK" icon="code" href="/docs/en/agent-sdk/typescript">
    전체 TypeScript API 참조 및 예제
  </Card>

  <Card title="Python SDK" icon="code" href="/docs/en/agent-sdk/python">
    전체 Python API 참조 및 예제
  </Card>
</CardGroup>
