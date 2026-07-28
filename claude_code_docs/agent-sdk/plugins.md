> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# SDK의 플러그인 (Plugins)

> Agent SDK를 통해 스킬, 에이전트, 후크 및 MCP 서버로 Claude Code를 확장할 수 있는 커스텀 플러그인 로드하기

플러그인을 사용하면 프로젝트 간에 공유할 수 있는 커스텀 기능으로 Claude Code를 확장할 수 있습니다. Agent SDK를 통해 로컬 디렉토리에서 커스텀 플러그인을 프로그래밍 방식으로 로드하여 에이전트 세션에 스킬, 에이전트, 후크 및 MCP 서버를 추가할 수 있습니다.

## 플러그인이란 무엇인가요?

플러그인은 다음과 같은 확장 기능을 포함할 수 있는 Claude Code 패키지입니다:

* **스킬 (Skills)**: Claude가 자율적으로 사용하는 모델 호출 가능 기능 (`/skill-name`으로도 호출 가능)
* **에이전트 (Agents)**: 특정 작업을 위한 전문 서브에이전트
* **후크 (Hooks)**: 도구 사용 및 기타 이벤트에 응답하는 이벤트 핸들러
* **MCP 서버 (MCP servers)**: Model Context Protocol을 통한 외부 도구 연동

<Note>
  `commands/` 디렉토리는 레거시 형식입니다. 신규 플러그인에는 `skills/`를 사용하세요. Claude Code는 하위 호환성을 위해 두 형식을 모두 계속 지원합니다.
</Note>

플러그인 구조 및 플러그인 생성 방법에 대한 완전한 정보는 [플러그인](/docs/en/plugins)을 참고하세요.

## 플러그인 로드하기

옵션 구성에서 로컬 파일 시스템 경로를 지정하여 플러그인을 로드합니다. `type` 필드는 SDK가 허용하는 유일한 값인 `"local"`이어야 합니다. [마켓플레이스](/docs/en/plugin-marketplaces) 또는 원격 저장소를 통해 배포된 플러그인을 사용하려면 먼저 다운로드한 다음 로컬 디렉토리 경로를 지정하세요. SDK는 서로 다른 위치에서 여러 플러그인을 로드하는 것을 지원합니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Hello",
    options: {
      plugins: [
        { type: "local", path: "./my-plugin" },
        { type: "local", path: "/absolute/path/to/another-plugin" }
      ]
    }
  })) {
    // Plugin commands, agents, and other features are now available
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions


  async def main():
      async for message in query(
          prompt="Hello",
          options=ClaudeAgentOptions(
              plugins=[
                  {"type": "local", "path": "./my-plugin"},
                  {"type": "local", "path": "/absolute/path/to/another-plugin"},
              ]
          ),
      ):
          # Plugin commands, agents, and other features are now available
          pass


  asyncio.run(main())
  ```
</CodeGroup>

### 경로 지정 방식

플러그인 경로는 다음 중 하나일 수 있습니다:

* **상대 경로**: 현재 작업 디렉토리를 기준으로 해석되는 경로 (예: `"./plugins/my-plugin"`)
* **절대 경로**: 전체 파일 시스템 경로 (예: `"/home/user/plugins/my-plugin"`)

<Note>
  경로는 하위 디렉토리가 아닌 `skills/`, `agents/`, `hooks/`, `commands/` (레거시), 또는 `.claude-plugin/`이 위치한 플러그인의 루트 디렉토리를 가리켜야 합니다.
</Note>

## 플러그인 설치 확인하기

플러그인이 성공적으로 로드되면 시스템 초기화 메시지에 표시됩니다. 플러그인을 사용할 수 있는지 확인할 수 있습니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Hello",
    options: {
      plugins: [{ type: "local", path: "./my-plugin" }]
    }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      // Check loaded plugins
      console.log("Plugins:", message.plugins);
      // Example: [{ name: "my-plugin", path: "./my-plugin" }]

      // Plugin skills appear with the plugin name as a prefix
      console.log("Skills:", message.skills);
      // Example: ["my-plugin:greet"]

      // Plugin commands use the same prefix, and skills appear here too
      console.log("Commands:", message.slash_commands);
      // Example: ["compact", "context", "my-plugin:custom-command", "my-plugin:greet"]
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage


  async def main():
      async for message in query(
          prompt="Hello",
          options=ClaudeAgentOptions(
              plugins=[{"type": "local", "path": "./my-plugin"}]
          ),
      ):
          if isinstance(message, SystemMessage) and message.subtype == "init":
              # Check loaded plugins
              print("Plugins:", message.data.get("plugins"))
              # Example: [{"name": "my-plugin", "path": "./my-plugin"}]

              # Plugin skills appear with the plugin name as a prefix
              print("Skills:", message.data.get("skills"))
              # Example: ["my-plugin:greet"]

              # Plugin commands use the same prefix, and skills appear here too
              print("Commands:", message.data.get("slash_commands"))
              # Example: ["compact", "context", "my-plugin:custom-command", "my-plugin:greet"]


  asyncio.run(main())
  ```
</CodeGroup>

## 플러그인 스킬 사용하기

충돌을 방지하기 위해 플러그인의 스킬에는 플러그인 이름이 자동으로 네임스페이스로 추가됩니다. 직접 호출하려면 프롬프트로 `/plugin-name:skill-name`을 전송하세요.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // Load a plugin with a custom /greet skill
  for await (const message of query({
    prompt: "/my-plugin:greet", // Use plugin skill with namespace
    options: {
      plugins: [{ type: "local", path: "./my-plugin" }]
    }
  })) {
    // Claude executes the custom greeting skill from the plugin
    if (message.type === "assistant") {
      console.log(message.message.content);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, TextBlock


  async def main():
      # Load a plugin with a custom /greet skill
      async for message in query(
          prompt="/demo-plugin:greet",  # Use plugin skill with namespace
          options=ClaudeAgentOptions(
              plugins=[{"type": "local", "path": "./plugins/demo-plugin"}]
          ),
      ):
          # Claude executes the custom greeting skill from the plugin
          if isinstance(message, AssistantMessage):
              for block in message.content:
                  if isinstance(block, TextBlock):
                      print(f"Claude: {block.text}")


  asyncio.run(main())
  ```
</CodeGroup>

<Note>
  CLI를 통해 플러그인을 설치한 경우(예: `/plugin install my-plugin@marketplace`), 설치 경로를 제공하여 SDK에서도 여전히 사용할 수 있습니다. CLI로 설치된 플러그인은 `~/.claude/plugins/`에서 확인하세요.
</Note>

## 전체 예제

다음은 플러그인 로드 및 사용을 보여주는 완전한 예제입니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";
  import * as path from "path";

  async function runWithPlugin() {
    const pluginPath = path.join(__dirname, "plugins", "my-plugin");

    console.log("Loading plugin from:", pluginPath);

    for await (const message of query({
      prompt: "What custom commands do you have available?",
      options: {
        plugins: [{ type: "local", path: pluginPath }],
        maxTurns: 3
      }
    })) {
      if (message.type === "system" && message.subtype === "init") {
        console.log("Loaded plugins:", message.plugins);
        console.log("Available skills:", message.skills);
        console.log("Available commands:", message.slash_commands);
      }

      if (message.type === "assistant") {
        console.log("Assistant:", message.message.content);
      }
    }
  }

  runWithPlugin().catch(console.error);
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  """Example demonstrating how to use plugins with the Agent SDK."""

  from pathlib import Path
  import anyio
  from claude_agent_sdk import (
      AssistantMessage,
      ClaudeAgentOptions,
      SystemMessage,
      TextBlock,
      query,
  )


  async def run_with_plugin():
      """Example using a custom plugin."""
      plugin_path = Path(__file__).parent / "plugins" / "demo-plugin"

      print(f"Loading plugin from: {plugin_path}")

      options = ClaudeAgentOptions(
          plugins=[{"type": "local", "path": str(plugin_path)}],
          max_turns=3,
      )

      async for message in query(
          prompt="What custom commands do you have available?", options=options
      ):
          if isinstance(message, SystemMessage) and message.subtype == "init":
              print(f"Loaded plugins: {message.data.get('plugins')}")
              print(f"Available skills: {message.data.get('skills')}")
              print(f"Available commands: {message.data.get('slash_commands')}")

          if isinstance(message, AssistantMessage):
              for block in message.content:
                  if isinstance(block, TextBlock):
                      print(f"Assistant: {block.text}")


  if __name__ == "__main__":
      anyio.run(run_with_plugin)
  ```
</CodeGroup>

## 플러그인 구조 참조

플러그인 디렉토리에는 일반적으로 `.claude-plugin/plugin.json` 매니페스트 파일이 포함됩니다. 매니페스트는 선택 사항입니다. 생략된 경우 Claude Code가 디렉토리 레이아웃에서 구성 요소를 자동 감지합니다. 디렉토리 구성은 다음과 같습니다:

```text theme={null}
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (optional, components auto-discovered without it)
├── skills/                   # Agent Skills (invoked autonomously or via /skill-name)
│   └── my-skill/
│       └── SKILL.md
├── commands/                 # Legacy: use skills/ instead
│   └── custom-cmd.md
├── agents/                   # Custom agents
│   └── specialist.md
├── hooks/                    # Event handlers
│   └── hooks.json
└── .mcp.json                # MCP server definitions
```

플러그인 생성에 관한 자세한 정보는 다음 문서를 참고하세요:

* [플러그인](/docs/en/plugins) - 완전한 플러그인 개발 가이드
* [플러그인 참조 문서](/docs/en/plugins-reference) - 기술 사양 및 스키마

## 일반적인 사용 사례

### 개발 및 테스트

전역 설치 없이 개발 중 플러그인 로드하기:

```typescript theme={null}
plugins: [{ type: "local", path: "./dev-plugins/my-plugin" }];
```

### 프로젝트 전용 확장

팀 일관성을 위해 프로젝트 저장소에 플러그인 포함하기:

```typescript theme={null}
plugins: [{ type: "local", path: "./project-plugins/team-workflows" }];
```

### 여러 플러그인 소스 사용

서로 다른 위치의 플러그인 결합하기:

```typescript theme={null}
plugins: [
  { type: "local", path: "./local-plugin" },
  { type: "local", path: "~/.claude/custom-plugins/shared-plugin" }
];
```

## 문제 해결

### 플러그인이 로드되지 않음

초기화 메시지에 플러그인이 나타나지 않는 경우:

1. **경로 확인**: 경로가 하위 디렉토리가 아닌 `skills/`, `agents/`, `hooks/`, `commands/` (레거시), 또는 `.claude-plugin/`이 위치한 플러그인 루트 디렉토리를 가리키는지 확인하세요
2. **plugin.json 검증**: 플러그인에 매니페스트가 포함되어 있다면 올바른 JSON 구문인지 확인하세요
3. **파일 권한 확인**: 플러그인 디렉토리를 읽을 수 있는지 확인하세요

### 스킬이 나타나지 않음

플러그인 스킬이 작동하지 않는 경우:

1. **네임스페이스 사용**: 플러그인 스킬을 `/plugin-name:skill-name` 형식으로 호출하세요
2. **초기화 메시지 확인**: 스킬이 올바른 네임스페이스와 함께 `skills` 목록에 표시되는지 확인하세요
3. **스킬 파일 검증**: 각 스킬이 `skills/` 아래 자체 하위 디렉토리에 `SKILL.md` 파일을 가지고 있는지 확인하세요 (예: `skills/my-skill/SKILL.md`)

### 경로 해석 문제

상대 경로가 작동하지 않는 경우:

1. **작업 디렉토리 확인**: 상대 경로는 현재 작업 디렉토리를 기준으로 해석됩니다
2. **절대 경로 사용**: 신뢰성을 위해 절대 경로 사용을 고려해 보세요
3. **경로 정규화**: 경로 유틸리티를 사용하여 경로를 올바르게 구성하세요

## 관련 항목

* [플러그인](/docs/en/plugins) - 완전한 플러그인 개발 가이드
* [플러그인 참조 문서](/docs/en/plugins-reference) - 기술 사양
* [명령어](/docs/en/agent-sdk/slash-commands) - SDK에서 명령어 사용하기
* [서브에이전트](/docs/en/agent-sdk/subagents) - 전문 서브에이전트 활용하기
* [스킬](/docs/en/agent-sdk/skills) - 에이전트 스킬 활용하기
