> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# SDK에서 Claude Code 기능 사용하기

> 프로젝트 지침, 스킬, 후크 및 기타 Claude Code 기능을 SDK 에이전트에 로드합니다.

Agent SDK는 Claude Code와 동일한 기반 위에 구축되었으므로 SDK 에이전트는 파일 시스템 기반의 동일한 기능인 프로젝트 지침(`CLAUDE.md` 및 규칙), 스킬, 후크 등에 접근할 수 있습니다.

`settingSources`를 생략하면 `query()`는 Claude Code CLI와 동일한 파일 시스템 설정(사용자, 프로젝트 및 로컬 설정, CLAUDE.md 파일, `.claude/` 스킬, 에이전트 및 명령)을 읽습니다. 이러한 설정 없이 실행하려면 `settingSources: []`를 전달하여 에이전트를 프로그래밍 방식으로 구성한 항목으로만 제한하세요. 관리형 정책 설정과 전역 `~/.claude.json` 구성은 이 옵션과 상관없이 읽힙니다. [settingSources가 제어하지 않는 항목](#settingsources가-제어하지-않는-항목)을 참조하세요.

각 기능이 수행하는 역할과 사용 시기에 대한 개념적 개요는 [Claude Code 확장하기](/docs/en/features-overview)를 참조하세요.

## settingSources로 파일 시스템 설정 제어하기

설정 소스 옵션(Python의 [`setting_sources`](/docs/en/agent-sdk/python#claudeagentoptions), TypeScript의 [`settingSources`](/docs/en/agent-sdk/typescript#settingsource))은 SDK가 로드하는 파일 시스템 기반 설정을 제어합니다. 명시적 목록을 전달하여 특정 소스를 옵트인하거나 빈 배열을 전달하여 사용자, 프로젝트 및 로컬 설정을 비활성화할 수 있습니다.

이 예제는 `settingSources`를 `["user", "project"]`로 설정하여 사용자 수준 및 프로젝트 수준 설정을 모두 로드합니다:

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ResultMessage
  import asyncio


  async def main():
      async for message in query(
          prompt="Help me refactor the auth module",
          options=ClaudeAgentOptions(
              # "user"는 ~/.claude/에서 로드하고, "project"는 현재 디렉토리의 ./.claude/에서 로드합니다.
              # 둘을 함께 사용하면 에이전트가 두 위치의 CLAUDE.md, 스킬, 후크, 권한에 접근할 수 있습니다.
              setting_sources=["user", "project"],
              allowed_tools=["Read", "Edit", "Bash"],
          ),
      ):
          if isinstance(message, AssistantMessage):
              for block in message.content:
                  if hasattr(block, "text"):
                      print(block.text)
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(f"\nResult: {message.result}")


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Help me refactor the auth module",
    options: {
      // "user"는 ~/.claude/에서 로드하고, "project"는 현재 디렉토리의 ./.claude/에서 로드합니다.
      // 둘을 함께 사용하면 에이전트가 두 위치의 CLAUDE.md, 스킬, 후크, 권한에 접근할 수 있습니다.
      settingSources: ["user", "project"],
      allowedTools: ["Read", "Edit", "Bash"]
    }
  })) {
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if (block.type === "text") console.log(block.text);
      }
    }
    if (message.type === "result" && message.subtype === "success") {
      console.log(`\nResult: ${message.result}`);
    }
  }
  ```
</CodeGroup>

이 코드 실행 시 어시스턴트의 응답이 stdout으로 출력되고, 실행이 완료되면 최종 결과 줄이 출력됩니다.

각 소스는 특정 위치에서 설정을 로드합니다. 여기서 `<cwd>`는 `cwd` 옵션을 통해 전달한 작업 디렉토리이며 지정되지 않은 경우 프로세스의 현재 디렉토리입니다. 전체 유형 정의는 [`SettingSource`](/docs/en/agent-sdk/typescript#settingsource) (TypeScript) 또는 [`SettingSource`](/docs/en/agent-sdk/python#settingsource) (Python)를 참조하세요.

| 소스 | 로드 항목 | 위치 |
| :---------- | :------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"project"` | 프로젝트 CLAUDE.md, `.claude/rules/*.md`, 프로젝트 스킬, 프로젝트 후크, 프로젝트 `settings.json` | `settings.json` 및 후크는 `<cwd>/.claude/`; CLAUDE.md 및 규칙은 `<cwd>` 및 모든 상위 디렉토리; 스킬은 `<cwd>` 및 저장소 루트까지의 모든 상위 디렉토리 |
| `"user"` | 사용자 CLAUDE.md, `~/.claude/rules/*.md`, 사용자 스킬, 사용자 설정 | `~/.claude/` |
| `"local"` | CLAUDE.local.md, `.claude/settings.local.json` | `settings.local.json`은 `<cwd>/.claude/`; CLAUDE.local.md는 `<cwd>` 및 모든 상위 디렉토리 |

`settingSources`를 생략하는 것은 `["user", "project", "local"]`과 동일합니다.

`cwd` 옵션은 SDK가 프로젝트 수준 입력을 탐색할 위치를 결정합니다. CLAUDE.md 및 규칙은 `<cwd>` 및 모든 상위 디렉토리에서 로드됩니다. 스킬은 `<cwd>` 및 저장소 루트까지의 모든 상위 디렉토리에서 로드됩니다. 프로젝트 `settings.json` 및 후크는 상위 디렉토리 폴백 없이 `<cwd>/.claude/`에서만 로드됩니다.

### settingSources가 제어하지 않는 항목

`settingSources`는 사용자, 프로젝트 및 로컬 설정을 다룹니다. 해당 값에 상관없이 읽히는 몇 가지 입력이 있습니다:

| 입력 | 동작 | 비활성화 방법 |
| :----------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 관리형 정책 설정 | MDM plist, 레지스트리 정책 또는 관리형 설정 파일과 같은 엔드포인트 관리형 정책이 호스트에서 로드됩니다. [서버 관리형 설정](/docs/en/server-managed-settings)은 세션이 조직 OAuth 로그인 또는 직접 구성된 API 키로 인증될 때 [자격이 되는 구성](/docs/en/server-managed-settings#platform-availability)에서 가져옵니다 | 엔드포인트 정책: 호스트에서 관리형 설정 파일, plist 또는 레지스트리 정책 제거. 서버 관리형 설정: 조직 관리자가 제어하며 SDK에서 비활성화할 수 없음 |
| `~/.claude.json` 전역 구성 | 항상 읽힘 | `env`에서 `CLAUDE_CONFIG_DIR`로 위치 이전 |
| `~/.claude/projects/<project>/memory/`의 자동 메모리 | 세션 시작 시 시스템 프롬프트에 로드됩니다. 에이전트는 전용 메모리 도구 대신 표준 `Write` 및 `Edit` 도구로 새 메모리를 기록하므로, 에이전트가 메모리를 저장하려면 해당 도구가 활성화되어 있어야 합니다 | 설정에서 `autoMemoryEnabled: false`를 지정하거나 `env`에서 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`을 지정 |
| [claude.ai MCP 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai) | 세션이 claude.ai 로그인으로 인증될 때 로드됩니다. 모델 요청만 수행할 수 있는 [`claude setup-token`](/docs/en/authentication#generate-a-long-lived-token)의 토큰을 `CLAUDE_CODE_OAUTH_TOKEN`이 보유할 때는 로드되지 않습니다. `mcpServers: {}`를 전달해도 커넥터가 억제되지 않습니다 | 설정에서 `strictMcpConfig: true`, [`disableClaudeAiConnectors: true`](/docs/en/mcp#disable-claude-ai-connectors)를 지정하거나 `env`에서 `ENABLE_CLAUDEAI_MCP_SERVERS=false`를 지정 |

<Warning>
  멀티 테넌트 격리를 위해 기본 `query()` 옵션에 의존하지 마세요. 위 입력들은 `settingSources`와 무관하게 읽히기 때문에 SDK 프로세스가 호스트 수준 구성 및 디렉토리별 메모리를 집어들 수 있습니다. 멀티 테넌트 배포의 경우 각 테넌트를 자체 파일 시스템에서 실행하고 `settingSources: []`와 함께 `env`에 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`을 설정하세요. [서버 관리형 설정](/docs/en/server-managed-settings)은 프로세스가 조직 자격 증명으로 인증할 때 가져오므로 파일 시스템 격리로 제거되지 않습니다. [안전한 배포](/docs/en/agent-sdk/secure-deployment)를 참조하세요.
</Warning>

## 프로젝트 지침 (CLAUDE.md 및 규칙)

`CLAUDE.md` 파일 및 `.claude/rules/*.md` 파일은 에이전트에 코딩 규칙, 빌드 명령, 아키텍처 결정 및 지침 등 프로젝트에 대한 영구 컨텍스트를 제공합니다. `settingSources`에 `"project"`가 포함되면(위 예시처럼) SDK는 세션 시작 시 이러한 파일을 컨텍스트로 로드합니다. 그러면 에이전트는 모든 프롬프트에서 반복하지 않아도 프로젝트 규칙을 따릅니다.

### CLAUDE.md 로드 위치

| 수준 | 위치 | 로드 시점 |
| :-------------------- | :---------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| 프로젝트 (루트) | `<cwd>/CLAUDE.md` 또는 `<cwd>/.claude/CLAUDE.md` | `settingSources`에 `"project"` 포함 시 |
| 프로젝트 규칙 | `<cwd>/.claude/rules/*.md` 및 모든 상위 디렉토리의 `.claude/rules/*.md` | `settingSources`에 `"project"` 포함 시 |
| 프로젝트 (상위 디렉토리) | `cwd` 상위 디렉토리의 `CLAUDE.md` 파일 | `settingSources`에 `"project"` 포함 시, 세션 시작 시 로드됨 |
| 프로젝트 (하위 디렉토리) | `cwd` 하위 디렉토리의 `CLAUDE.md` 파일 | `settingSources`에 `"project"` 포함 시, 에이전트가 해당 하위 트리에서 파일을 읽을 때 요청 시 로드됨 |
| 로컬 | `<cwd>/CLAUDE.local.md` 및 모든 상위 디렉토리의 `CLAUDE.local.md` | `settingSources`에 `"local"` 포함 시 |
| 사용자 | `~/.claude/CLAUDE.md` | `settingSources`에 `"user"` 포함 시 |
| 사용자 규칙 | `~/.claude/rules/*.md` | `settingSources`에 `"user"` 포함 시 |

모든 수준은 가산적(additive)입니다. 프로젝트 및 사용자 CLAUDE.md 파일이 모두 존재하면 에이전트에 둘 다 표시됩니다. 수준 간에 엄격한 우선순위 규칙은 없습니다. 지침이 충돌하면 Claude가 이를 해석하는 방식에 따라 결과가 결정됩니다. 충돌하지 않는 규칙을 작성하거나 더 구체적인 파일에 우선순위를 명시하세요 ("이 프로젝트 지침은 충돌하는 사용자 수준 기본값보다 우선합니다").

<Tip>
  CLAUDE.md 파일을 사용하지 않고 `systemPrompt`를 통해 직접 컨텍스트를 주입할 수도 있습니다. [시스템 프롬프트 수정](/docs/en/agent-sdk/modifying-system-prompts)을 참조하세요. 대화형 Claude Code 세션과 SDK 에이전트 간에 동일한 컨텍스트를 공유하고 싶을 때 CLAUDE.md를 사용하세요.
</Tip>

CLAUDE.md 콘텐츠 구성 및 구조화 방식은 [Claude 메모리 관리](/docs/en/memory)를 참조하세요.

## 스킬(Skills)

스킬은 에이전트에 전문 지식과 호출 가능한 워크플로우를 제공하는 마크다운 파일입니다. 세션마다 로드되는 `CLAUDE.md`와 달리 스킬은 요청 시 로드됩니다. 에이전트는 시작 시 스킬 설명을 받고 관련성이 있을 때 전체 콘텐츠를 로드합니다.

스킬은 `settingSources`를 통해 파일 시스템에서 탐색됩니다. `query()`의 `skills` 옵션을 생략하면 탐색된 사용자 및 프로젝트 스킬이 활성화되고 CLI 동작과 일치하도록 Skill 도구를 이용할 수 있습니다. 활성화되는 스킬을 제어하려면 `skills`를 `"all"`, 스킬 이름 목록, 또는 모두 비활성화하는 `[]`로 전달하세요. `skills`가 설정되면 SDK는 `allowedTools`에 Skill 도구를 자동으로 추가합니다. 명시적인 `tools` 목록을 함께 전달하는 경우 Claude가 스킬을 호출할 수 있도록 해당 목록에 `"Skill"`을 포함하세요.

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage
  import asyncio


  # settingSources에 "project"가 포함되면
  # .claude/skills/의 스킬이 자동으로 탐색됩니다.
  async def main():
      async for message in query(
          prompt="Review this PR using our code review checklist",
          options=ClaudeAgentOptions(
              setting_sources=["user", "project"],
              skills="all",
              allowed_tools=["Read", "Grep", "Glob"],
          ),
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // settingSources에 "project"가 포함되면
  // .claude/skills/의 스킬이 자동으로 탐색됩니다.
  for await (const message of query({
    prompt: "Review this PR using our code review checklist",
    options: {
      settingSources: ["user", "project"],
      skills: "all",
      allowedTools: ["Read", "Grep", "Glob"]
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```
</CodeGroup>

<Note>
  스킬은 파일 시스템 아티팩트(`.claude/skills/<name>/SKILL.md`)로 생성되어야 합니다. SDK에는 스킬 등록을 위한 프로그래밍 방식의 API가 없습니다. 전체 내용은 [SDK에서의 에이전트 스킬](/docs/en/agent-sdk/skills)을 참조하세요.
</Note>

스킬 생성 및 사용에 대한 자세한 내용은 [SDK에서의 에이전트 스킬](/docs/en/agent-sdk/skills)을 참조하세요.

## 후크(Hooks)

SDK는 두 가지 후크 정의 방식을 지원하며, 나란히 함께 실행됩니다:

* **파일 시스템 후크:** `settings.json`에 정의된 쉘 명령으로, `settingSources`에 관련 소스가 포함될 때 로드됩니다. 이는 [대화형 Claude Code 세션](/docs/en/hooks-guide)에서 구성하는 것과 동일한 후크입니다.
* **프로그래밍 방식 후크:** `query()`에 직접 전달되는 콜백 함수입니다. 애플리케이션 프로세스에서 실행되며 구조화된 결정을 반환할 수 있습니다. [후크로 실행 제어하기](/docs/en/agent-sdk/hooks)를 참조하세요.

두 유형 모두 동일한 후크 수명 주기 동안 실행됩니다. 프로젝트의 `.claude/settings.json`에 이미 후크가 있고 `settingSources: ["project"]`를 설정하면 추가 구성 없이 해당 후크가 SDK에서 자동으로 실행됩니다.

후크 콜백은 도구 입력을 수신하고 결정 딕셔너리를 반환합니다. `{}`를 반환하면 도구 진행이 허용됩니다. 실행을 차단하려면 `permissionDecision: "deny"` 및 `permissionDecisionReason`을 포함하는 `hookSpecificOutput` 객체를 반환하세요. 이 사유는 도구 결과로 Claude에게 전송됩니다. 최상위 `decision` 및 `reason` 필드는 `PreToolUse`에서 더 이상 사용되지 않습니다(deprecated). 전체 콜백 시그니처 및 반환 유형은 [후크 가이드](/docs/en/agent-sdk/hooks)를 참조하세요.

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage
  import asyncio


  # PreToolUse 후크 콜백. 위치 인자:
  #   input_data: tool_name, tool_input, hook_event_name을 가진 HookInput 딕셔너리
  #   tool_use_id: str | None, 가로챈 도구 호출의 ID
  #   context: HookContext, 향후 abort-signal 지원을 위한 예약 필드
  async def audit_bash(input_data, tool_use_id, context):
      command = input_data.get("tool_input", {}).get("command", "")
      if "rm -rf" in command:
          return {
              "hookSpecificOutput": {
                  "hookEventName": "PreToolUse",
                  "permissionDecision": "deny",
                  "permissionDecisionReason": "Destructive command blocked",
              }
          }
      return {}  # 빈 딕셔너리: 도구 진행 허용


  # .claude/settings.json의 파일 시스템 후크는
  # settingSources가 이를 로드할 때 자동으로 실행됩니다. 프로그래밍 방식 후크를 추가할 수도 있습니다:
  async def main():
      async for message in query(
          prompt="Refactor the auth module",
          options=ClaudeAgentOptions(
              setting_sources=["project"],  # .claude/settings.json에서 후크 로드
              hooks={
                  "PreToolUse": [
                      HookMatcher(matcher="Bash", hooks=[audit_bash]),
                  ]
              },
          ),
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query, type HookInput, type HookJSONOutput } from "@anthropic-ai/claude-agent-sdk";

  // PreToolUse 후크 콜백. HookInput은 hook_event_name에 따른 구별된 유니온이므로
  // 이를 좁히면 TypeScript가 이 이벤트에 대해 올바른 tool_input 형상을 파악합니다.
  const auditBash = async (input: HookInput): Promise<HookJSONOutput> => {
    if (input.hook_event_name !== "PreToolUse") return {};
    const toolInput = input.tool_input as { command?: string };
    if (toolInput.command?.includes("rm -rf")) {
      return {
        hookSpecificOutput: {
          hookEventName: "PreToolUse",
          permissionDecision: "deny",
          permissionDecisionReason: "Destructive command blocked",
        },
      };
    }
    return {}; // 빈 객체: 도구 진행 허용
  };

  // .claude/settings.json의 파일 시스템 후크는
  // settingSources가 이를 로드할 때 자동으로 실행됩니다. 프로그래밍 방식 후크를 추가할 수도 있습니다:
  for await (const message of query({
    prompt: "Refactor the auth module",
    options: {
      settingSources: ["project"], // .claude/settings.json에서 후크 로드
      hooks: {
        PreToolUse: [{ matcher: "Bash", hooks: [auditBash] }]
      }
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```
</CodeGroup>

### 어떤 후크 유형을 사용할지 선택하기

| 후크 유형 | 적합한 용도 |
| :---------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **파일 시스템** (`settings.json`) | CLI 및 SDK 세션 간 후크 공유. `"command"`(쉘 스크립트), `"http"`(엔드포인트로 POST), `"mcp_tool"`(연결된 MCP 서버 도구 호출), `"prompt"`(LLM이 프롬프트 평가), `"agent"`(검증 에이전트 생성) 지원. 주 에이전트 및 생성되는 모든 서브에이전트에서 실행됨 |
| **프로그래밍 방식** (`query()`의 콜백) | 애플리케이션 전용 로직, 구조화된 결정, 인프로세스 연동. 이들 역시 서브에이전트 내부에서 실행됨. 콜백의 첫 번째 인자인 후크 입력에는 어떤 에이전트가 후크를 실행했는지 식별하는 `agent_id` 및 `agent_type` 필드가 전달됨 |

<Note>
  TypeScript SDK는 `SessionStart`, `SessionEnd`, `TeammateIdle`, `TaskCompleted`를 포함하여 Python보다 더 많은 후크 이벤트를 지원합니다. 전체 이벤트 호환성 표는 [후크 가이드](/docs/en/agent-sdk/hooks)를 참조하세요.
</Note>

프로그래밍 방식 후크에 대한 전체 설명은 [후크로 실행 제어하기](/docs/en/agent-sdk/hooks)를 참조하세요. 파일 시스템 후크 구문은 [후크](/docs/en/hooks)를 참조하세요.

## 올바른 기능 선택하기

Agent SDK는 에이전트의 동작을 확장할 수 있는 여러 방법을 제공합니다. 어느 것을 사용할지 불분명하다면 이 표에서 공통 목표에 적합한 접근 방식을 확인하세요.

| 목표 | 사용 항목 | SDK 표면 |
| :---------------------------------------------------------------- | :-------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 에이전트가 항상 따르는 프로젝트 규칙 설정 | [CLAUDE.md](/docs/en/memory) | `settingSources: ["project"]`가 이를 자동으로 로드 |
| 에이전트에 관련 시 로드하는 참고 자료 제공 | [스킬](/docs/en/agent-sdk/skills) | `settingSources` + `skills` 옵션 |
| 재사용 가능한 워크플로우 실행(배포, 리뷰, 릴리스) | [사용자 호출 가능 스킬](/docs/en/agent-sdk/skills) | `settingSources` + `skills` 옵션 |
| 새로운 컨텍스트(조사, 리뷰)로 격리된 하위 작업 위임 | [서브에이전트](/docs/en/agent-sdk/subagents) | `agents` 파라미터 + `allowedTools: ["Agent"]` |
| 공유 작업 목록 및 직접 인터에이전트 메시징으로 여러 Claude Code 인스턴스 조율 | [에이전트 팀](/docs/en/agent-teams) | SDK 옵션을 통해 직접 구성되지 않음. 에이전트 팀은 한 세션이 팀 리더 역할을 하여 독립된 동료 간의 작업을 조율하는 CLI 기능임 |
| 도구 호출 시 결정론적 로직 실행(감사, 차단, 변환) | [후크](/docs/en/agent-sdk/hooks) | 콜백이 있는 `hooks` 파라미터, 또는 `settingSources`를 통해 로드된 쉘 스크립트 |
| 외부 서비스에 대한 구조화된 도구 접근을 Claude에게 제공 | [MCP](/docs/en/agent-sdk/mcp) | `mcpServers` 파라미터 |

<Tip>
  **서브에이전트 vs 에이전트 팀:** 서브에이전트는 단명적이고 격리되어 있습니다: 새로운 대화, 단일 작업, 상위에 요약 반환. 에이전트 팀은 작업 목록을 공유하고 서로 직접 메시지를 주고받는 여러 독립적인 Claude Code 인스턴스를 조율합니다. 에이전트 팀은 CLI 기능입니다. 자세한 내용은 [서브에이전트가 상속하는 항목](/docs/en/agent-sdk/subagents#what-subagents-inherit) 및 [에이전트 팀 비교](/docs/en/agent-teams#compare-with-subagents)를 참조하세요.
</Tip>

활성화하는 모든 기능은 에이전트의 컨텍스트 윈도우를 늘립니다. 기능별 비용 및 레이어 결합 방식은 [Claude Code 확장하기](/docs/en/features-overview#understand-context-costs)를 참조하세요.

## 관련 리소스

* [Claude Code 확장하기](/docs/en/features-overview): 비교 표 및 컨텍스트 비용 분석을 포함한 모든 확장 기능의 개념적 개요
* [SDK에서의 스킬](/docs/en/agent-sdk/skills): 프로그래밍 방식으로 스킬을 사용하는 전체 가이드
* [서브에이전트](/docs/en/agent-sdk/subagents): 격리된 하위 작업을 위한 서브에이전트 정의 및 호출
* [후크](/docs/en/agent-sdk/hooks): 주요 실행 지점에서 에이전트 동작 가로채기 및 제어
* [권한](/docs/en/agent-sdk/permissions): 모드, 규칙, 콜백을 통한 도구 접근 제어
* [시스템 프롬프트](/docs/en/agent-sdk/modifying-system-prompts): CLAUDE.md 파일 없이 컨텍스트 주입
