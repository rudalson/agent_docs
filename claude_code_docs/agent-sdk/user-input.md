> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 승인 및 사용자 입력 처리 (Handle approvals and user input)

> Claude의 승인 요청 및 명확한 질문을 사용자에게 표면화하고 사용자의 결정을 SDK로 반환합니다.

작업을 수행하는 동안 Claude는 때때로 사용자에게 확인을 받아야 합니다. 파일을 삭제하기 전에 권한이 필요하거나, 새 프로젝트에 어떤 데이터베이스를 사용할지 물어봐야 할 수 있습니다. 애플리케이션은 사용자에게 이러한 요청을 표면화하여 Claude가 사용자 입력과 함께 작업을 계속할 수 있도록 해야 합니다.

Claude는 두 가지 상황에서 사용자 입력을 요청합니다: **도구 사용 권한이 필요한 경우**(파일 삭제나 명령어 실행 등)와 **명확히 할 질문이 있는 경우**(`AskUserQuestion` 도구 사용). 두 상황 모두 `canUseTool` 콜백을 트리거하며, 응답을 반환할 때까지 실행을 일시 중지합니다. 이는 Claude가 작업을 완료하고 다음 메시지를 기다리는 일반적인 대화 차례와 다릅니다.

명확히 할 질문의 경우 Claude가 질문과 옵션을 생성합니다. 귀하의 역할은 이를 사용자에게 제시하고 사용자의 선택을 반환하는 것입니다. 이 흐름에 자체 질문을 추가할 수는 없습니다. 사용자에게 직접 질문해야 하는 경우 애플리케이션 로직에서 별도로 처리하세요.

콜백은 무기한 대기 상태로 유지될 수 있습니다. 콜백이 반환될 때까지 실행이 일시 중지된 상태로 유지되며, SDK는 쿼리 자체가 취소될 때만 대기를 취소합니다. 프로세스를 정상적으로 실행 상태로 유지할 수 있는 시간보다 사용자가 응답하는 데 더 오래 걸릴 수 있는 경우, 프로세스를 종료하고 지속된 세션에서 나중에 재개할 수 있게 해주는 [`defer` 훅 결정](/docs/en/hooks#defer-a-tool-call-for-later)을 반환하세요.

이 가이드에서는 각 유형의 요청을 감지하고 적절하게 응답하는 방법을 보여줍니다.

## Claude에게 입력이 필요한 시점 감지

쿼리 옵션에 `canUseTool` 콜백을 전달합니다. 콜백은 Claude에게 사용자 입력이 필요할 때마다 트리거되어 도구 이름과 입력을 인자로 받습니다:

<CodeGroup>
  ```python Python theme={null}
  async def handle_tool_request(tool_name, input_data, context):
      # 사용자에게 프롬프트를 표시하고 허용 또는 거부 반환
      ...


  options = ClaudeAgentOptions(can_use_tool=handle_tool_request)
  ```

  ```typescript TypeScript theme={null}
  async function handleToolRequest(toolName, input, options) {
    // options에는 { signal: AbortSignal, suggestions?: PermissionUpdate[] }가 포함됨
    // 사용자에게 프롬프트를 표시하고 허용 또는 거부 반환
  }

  const options = { canUseTool: handleToolRequest };
  ```
</CodeGroup>

콜백은 다음 두 가지 경우에 트리거됩니다:

1. **도구 승인 필요**: Claude가 [권한 규칙](/docs/en/agent-sdk/permissions)이나 권한 모드에 의해 자동 승인되지 않은 도구를 사용하고자 할 때. 도구의 `tool_name`을 확인하세요(예: `"Bash"`, `"Write"`).
2. **Claude의 질문**: Claude가 `AskUserQuestion` 도구를 호출할 때. `tool_name == "AskUserQuestion"`인지 확인하여 다르게 처리하세요. `tools` 배열을 지정하는 경우 작동하려면 `AskUserQuestion`을 포함해야 합니다. 세부 정보는 [명확히 할 질문 처리](#handle-clarifying-questions)를 참조하세요.

<Warning>
  **자동 승인된 도구에 대해서는 콜백이 호출되지 않습니다.** [권한 평가 흐름](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)의 이전 단계에서 이뤄진 승인(허용 규칙 또는 `acceptEdits`, `bypassPermissions`와 같은 모드)은 `canUseTool`을 참조하기 전에 호출을 해결합니다. `allowed_tools`에 도구를 그대로 나열하면 ask 규칙이나 `plan` 모드가 호출을 프롬프트로 다시 라우팅하지 않는 한 해당 도구에 대한 `canUseTool` 검사는 절대 실행되지 않습니다. 모든 도구 호출에 적용되어야 하는 로직의 경우 흐름의 나머지 부분 전에 실행되고 요청을 허용, 거부 또는 수정할 수 있는 [`PreToolUse` 훅](/docs/en/agent-sdk/hooks)을 사용하세요.

  `AskUserQuestion`, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구 및 [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구는 허용 규칙이 일치하더라도 콜백에 도달합니다. `dontAsk` 모드에서는 콜백을 호출하지 않고 이러한 호출이 거부됩니다.
</Warning>

Claude가 승인을 기다리고 있을 때 외부 알림(Slack, 이메일, 푸시)을 보내려면 [`PermissionRequest` 훅](/docs/en/agent-sdk/hooks#available-hooks)을 사용할 수도 있습니다.

## 도구 승인 요청 처리

쿼리 옵션에 `canUseTool` 콜백을 전달하면 권한 흐름의 이전 단계에서 승인되지 않은 도구를 Claude가 사용하고자 할 때 콜백이 실행됩니다. 콜백은 세 가지 인자를 받습니다:

| 인자                                | 설명                                                                                                                                                                                                                                                                                                                           |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `toolName`                          | Claude가 사용하고자 하는 도구 이름 (예: `"Bash"`, `"Write"`, `"Edit"`)                                                                                                                                                                                                                                                                |
| `input`                             | Claude가 도구에 전달하는 매개변수. 내용은 도구에 따라 다름.                                                                                                                                                                                                                                                                           |
| `options` (TS) / `context` (Python) | 선택적 `suggestions`(재프롬프트를 방지하기 위한 제안된 `PermissionUpdate` 항목) 및 취소 신호를 포함하는 추가 컨텍스트. TypeScript에서 `signal`은 `AbortSignal`이며; Python에서 신호 필드는 향후 사용을 위해 예약됨. Python의 경우 [`ToolPermissionContext`](/docs/en/agent-sdk/python#toolpermissioncontext) 참조. |

`input` 객체에는 도구별 매개변수가 포함됩니다. 공통 예시:

| 도구    | 입력 필드                              |
| ------- | -------------------------------------- |
| `Bash`  | `command`, `description`, `timeout`    |
| `Write` | `file_path`, `content`                 |
| `Edit`  | `file_path`, `old_string`, `new_string` |
| `Read`  | `file_path`, `offset`, `limit`         |

전체 입력 스키마는 SDK 레퍼런스를 참조하세요: [Python](/docs/en/agent-sdk/python#tool-input%2Foutput-types) | [TypeScript](/docs/en/agent-sdk/typescript#tool-input-types).

이 정보를 사용자에게 표시하여 작업 허용 또는 거부 여부를 결정하게 한 후 적절한 응답을 반환할 수 있습니다.

다음 예시는 Claude에게 테스트 파일을 생성하고 삭제하도록 요청합니다. Claude가 각 작업을 시도할 때 콜백은 터미널에 도구 요청을 출력하고 y/n 승인을 프롬프트로 표시합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio

  from claude_agent_sdk import ClaudeAgentOptions, ResultMessage, query
  from claude_agent_sdk.types import (
      HookMatcher,
      PermissionResultAllow,
      PermissionResultDeny,
      ToolPermissionContext,
  )


  async def can_use_tool(
      tool_name: str, input_data: dict, context: ToolPermissionContext
  ) -> PermissionResultAllow | PermissionResultDeny:
      # 도구 요청 표시
      print(f"\nTool: {tool_name}")
      if tool_name == "Bash":
          print(f"Command: {input_data.get('command')}")
          if input_data.get("description"):
              print(f"Description: {input_data.get('description')}")
      else:
          print(f"Input: {input_data}")

      # 사용자 승인 받기
      response = input("Allow this action? (y/n): ")

      # 사용자 응답을 기반으로 허용 또는 거부 반환
      if response.lower() == "y":
          # 허용: 도구가 원본(또는 수정된) 입력으로 실행됨
          return PermissionResultAllow(updated_input=input_data)
      else:
          # 거부: 도구가 실행되지 않고 Claude가 메시지를 확인함
          return PermissionResultDeny(message="User denied this action")


  # 필수 해결책: 더미 훅이 can_use_tool을 위해 스트림을 열린 상태로 유지함
  async def dummy_hook(input_data, tool_use_id, context):
      return {"continue_": True}


  async def prompt_stream():
      yield {
          "type": "user",
          "message": {
              "role": "user",
              "content": "Create a test file in /tmp and then delete it",
          },
      }


  async def main():
      async for message in query(
          prompt=prompt_stream(),
          options=ClaudeAgentOptions(
              can_use_tool=can_use_tool,
              hooks={"PreToolUse": [HookMatcher(matcher=None, hooks=[dummy_hook])]},
          ),
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";
  import * as readline from "readline";

  // 터미널에서 사용자 입력을 프롬프트로 받는 헬퍼
  function prompt(question: string): Promise<string> {
    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });
    return new Promise((resolve) =>
      rl.question(question, (answer) => {
        rl.close();
        resolve(answer);
      })
    );
  }

  for await (const message of query({
    prompt: "Create a test file in /tmp and then delete it",
    options: {
      canUseTool: async (toolName, input) => {
        // 도구 요청 표시
        console.log(`\nTool: ${toolName}`);
        if (toolName === "Bash") {
          console.log(`Command: ${input.command}`);
          if (input.description) console.log(`Description: ${input.description}`);
        } else {
          console.log(`Input: ${JSON.stringify(input, null, 2)}`);
        }

        // 사용자 승인 받기
        const response = await prompt("Allow this action? (y/n): ");

        // 사용자 응답을 기반으로 허용 또는 거부 반환
        if (response.toLowerCase() === "y") {
          // 허용: 도구가 원본(또는 수정된) 입력으로 실행됨
          return { behavior: "allow", updatedInput: input };
        } else {
          // 거부: 도구가 실행되지 않고 Claude가 메시지를 확인함
          return { behavior: "deny", message: "User denied this action" };
        }
      }
    }
  })) {
    if ("result" in message) console.log(message.result);
  }
  ```
</CodeGroup>

<Note>
  Python에서 `can_use_tool`은 [스트리밍 모드](/docs/en/agent-sdk/streaming-vs-single-mode)가 필요합니다. `query(prompt=generator)` 또는 `ClaudeSDKClient.connect(prompt=async_iterable)`를 통해 유한한 메시지 스트림을 전달할 때, 등록된 훅이나 프로세스 내 MCP 서버가 스트림을 계속 열어두지 않으면 마지막 메시지 후 권한 콜백이 호출되기 전에 SDK가 입력 스트림을 닫습니다. 위 예시는 `{"continue_": True}`를 반환하는 `PreToolUse` 훅으로 스트림을 열어 둡니다. 프롬프트 없이 연결하고 `ClaudeSDKClient.query()`를 통해 메시지를 전송하면 스트림이 자체적으로 열린 상태로 유지되므로 훅이 필요하지 않습니다.
</Note>

이 예시는 `y` 이외의 다른 입력이 거부로 처리되는 `y/n` 흐름을 사용합니다. 실제로는 사용자가 요청을 수정하거나 피드백을 제공하거나 Claude를 완전히 다른 방향으로 전환할 수 있는 더 풍부한 UI를 구축할 수 있습니다. 응답할 수 있는 모든 방법은 [도구 요청에 응답하기](#respond-to-tool-requests)를 참조하세요.

### 도구 요청에 응답하기

콜백은 두 가지 응답 타입 중 하나를 반환합니다:

| 응답      | Python                                     | TypeScript                            |
| --------- | ------------------------------------------ | ------------------------------------- |
| **허용**  | `PermissionResultAllow(updated_input=...)` | `{ behavior: "allow", updatedInput }` |
| **거부**  | `PermissionResultDeny(message=...)`        | `{ behavior: "deny", message }`       |

허용할 때 수정된 입력(TypeScript의 `updatedInput`, Python의 `updated_input`)을 반환하지 않는 한 도구는 Claude가 요청한 입력으로 실행됩니다.

거부할 때 이유를 설명하는 메시지를 제공하세요. Claude는 이 메시지를 확인하고 접근 방식을 조절할 수 있습니다.

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk.types import PermissionResultAllow, PermissionResultDeny

  # 도구 실행 허용
  return PermissionResultAllow(updated_input=input_data)

  # 도구 차단
  return PermissionResultDeny(message="User rejected this action")
  ```

  ```typescript TypeScript theme={null}
  // 도구 실행 허용
  return { behavior: "allow", updatedInput: input };

  // 도구 차단
  return { behavior: "deny", message: "User rejected this action" };
  ```
</CodeGroup>

허용 또는 거부 외에도 도구의 입력을 수정하거나 Claude가 접근 방식을 조절하는 데 도움이 되는 컨텍스트를 제공할 수 있습니다:

* **승인**: Claude가 요청한 대로 도구 실행 허용
* **변경 후 승인**: 실행 전 입력 수정 (예: 경로 정화, 제약 조건 추가)
* **승인 및 기억**: 제안된 권한 규칙을 에코하여 다음번에 일치하는 호출이 프롬프트를 건너뛰도록 함
* **거절**: 도구를 차단하고 Claude에게 이유 설명
* **대안 제안**: 차단하되 사용자가 원하는 방향으로 Claude 안내
* **전체 방향 전환**: [스트리밍 입력](/docs/en/agent-sdk/streaming-vs-single-mode)을 사용하여 Claude에게 완전히 새로운 지침 전달

<Tabs>
  <Tab title="승인">
    사용자가 있는 그대로 작업을 승인합니다. 콜백의 `input`을 변경 없이 그대로 전달하면 Claude가 요청한 대로 정확히 도구가 실행됩니다.

    <CodeGroup>
      ```python Python theme={null}
      async def can_use_tool(tool_name, input_data, context):
          print(f"Claude wants to use {tool_name}")
          approved = await ask_user("Allow this action?")

          if approved:
              return PermissionResultAllow(updated_input=input_data)
          return PermissionResultDeny(message="User declined")
      ```

      ```typescript TypeScript theme={null}
      canUseTool: async (toolName, input) => {
        console.log(`Claude wants to use ${toolName}`);
        const approved = await askUser("Allow this action?");

        if (approved) {
          return { behavior: "allow", updatedInput: input };
        }
        return { behavior: "deny", message: "User declined" };
      };
      ```
    </CodeGroup>
  </Tab>

  <Tab title="변경 후 승인">
    사용자가 승인하지만 먼저 요청을 수정하고자 합니다. 도구가 실행되기 전에 입력을 변경할 수 있습니다. Claude는 결과를 보게 되지만 수정한 사실은 알지 못합니다. 매개변수 정화, 제약 조건 추가, 접근 범위 제어 시 유용합니다.

    <CodeGroup>
      ```python Python theme={null}
      async def can_use_tool(tool_name, input_data, context):
          if tool_name == "Bash":
              # 사용자가 승인했지만 모든 명령어를 샌드박스로 제한
              sandboxed_input = {**input_data}
              sandboxed_input["command"] = input_data["command"].replace(
                  "/tmp", "/tmp/sandbox"
              )
              return PermissionResultAllow(updated_input=sandboxed_input)
          return PermissionResultAllow(updated_input=input_data)
      ```

      ```typescript TypeScript theme={null}
      canUseTool: async (toolName, input) => {
        if (toolName === "Bash") {
          // 사용자가 승인했지만 모든 명령어를 샌드박스로 제한
          const sandboxedInput = {
            ...input,
            command: input.command.replace("/tmp", "/tmp/sandbox")
          };
          return { behavior: "allow", updatedInput: sandboxedInput };
        }
        return { behavior: "allow", updatedInput: input };
      };
      ```
    </CodeGroup>
  </Tab>

  <Tab title="승인 및 기억">
    사용자가 승인하고 이러한 호출에 대해 다시 질문받지 않기를 원합니다. 세 번째 콜백 인자는 준비된 [`PermissionUpdate`](/docs/en/agent-sdk/typescript#permissionupdate) 항목 배열인 `suggestions`를 전달합니다. 적용하려면 이를 `updatedPermissions`에 에코하세요. `localSettings` 대상을 사용하는 제안은 규칙을 `.claude/settings.local.json`에 기록하여 향후 세션에서 일치하는 호출에 대한 프롬프트를 건너끕니다.

    Python 예시는 `claude-agent-sdk` 0.1.80 이상이 필요합니다.

    <CodeGroup>
      ```python Python theme={null}
      async def can_use_tool(tool_name, input_data, context):
          choice = await ask_user(f"Allow {tool_name}?", ["once", "always", "no"])

          if choice == "always":
              persist = [
                  s for s in context.suggestions if s.destination == "localSettings"
              ]
              return PermissionResultAllow(
                  updated_input=input_data, updated_permissions=persist
              )
          if choice == "once":
              return PermissionResultAllow(updated_input=input_data)
          return PermissionResultDeny(message="User declined")
      ```

      ```typescript TypeScript theme={null}
      canUseTool: async (toolName, input, { suggestions = [] }) => {
        const choice = await askUser(`Allow ${toolName}?`, ["once", "always", "no"]);

        if (choice === "always") {
          const persist = suggestions.filter(
            (s) => s.destination === "localSettings"
          );
          return {
            behavior: "allow",
            updatedInput: input,
            updatedPermissions: persist
          };
        }
        if (choice === "once") {
          return { behavior: "allow", updatedInput: input };
        }
        return { behavior: "deny", message: "User declined" };
      };
      ```
    </CodeGroup>
  </Tab>

  <Tab title="거절">
    사용자가 이 작업이 실행되는 것을 원하지 않습니다. 도구를 차단하고 이유를 설명하는 메시지를 제공하세요. Claude는 이 메시지를 보고 다른 접근 방식을 시도할 수 있습니다.

    <CodeGroup>
      ```python Python theme={null}
      async def can_use_tool(tool_name, input_data, context):
          approved = await ask_user(f"Allow {tool_name}?")

          if not approved:
              return PermissionResultDeny(message="User rejected this action")
          return PermissionResultAllow(updated_input=input_data)
      ```

      ```typescript TypeScript theme={null}
      canUseTool: async (toolName, input) => {
        const approved = await askUser(`Allow ${toolName}?`);

        if (!approved) {
          return {
            behavior: "deny",
            message: "User rejected this action"
          };
        }
        return { behavior: "allow", updatedInput: input };
      };
      ```
    </CodeGroup>
  </Tab>

  <Tab title="대안 제안">
    사용자가 이 특정 작업을 원하지는 않지만 다른 아이디어가 있습니다. 도구를 차단하고 메시지에 지침을 포함하세요. Claude는 이 메시지를 읽고 귀하의 피드백을 바탕으로 어떻게 진행할지 결정합니다.

    <CodeGroup>
      ```python Python theme={null}
      async def can_use_tool(tool_name, input_data, context):
          if tool_name == "Bash" and "rm" in input_data.get("command", ""):
              # 사용자가 삭제를 원하지 않으므로 대신 아카이브 제안
              return PermissionResultDeny(
                  message="User doesn't want to delete files. They asked if you could compress them into an archive instead."
              )
          return PermissionResultAllow(updated_input=input_data)
      ```

      ```typescript TypeScript theme={null}
      canUseTool: async (toolName, input) => {
        if (toolName === "Bash" && input.command.includes("rm")) {
          // 사용자가 삭제를 원하지 않으므로 대신 아카이브 제안
          return {
            behavior: "deny",
            message:
              "User doesn't want to delete files. They asked if you could compress them into an archive instead."
          };
        }
        return { behavior: "allow", updatedInput: input };
      };
      ```
    </CodeGroup>
  </Tab>

  <Tab title="전체 방향 전환">
    단순한 피드백이 아니라 완전히 방향을 바꾸려면 [스트리밍 입력](/docs/en/agent-sdk/streaming-vs-single-mode)을 사용하여 Claude에게 새 지침을 직접 전송하세요. 이렇게 하면 현재 도구 요청이 우회되고 Claude에게 따라야 할 완전히 새로운 지침이 전달됩니다.
  </Tab>
</Tabs>

## 명확히 할 질문 처리

유효한 접근 방식이 여러 개 있는 작업에서 Claude에게 더 많은 방향 제시가 필요한 경우 `AskUserQuestion` 도구를 호출합니다. 이렇게 하면 `toolName`이 `AskUserQuestion`으로 설정된 상태에서 `canUseTool` 콜백이 트리거됩니다. 입력에는 객관식 옵션 형태의 Claude 질문이 포함되며, 이를 사용자에게 표시하고 사용자의 선택을 반환합니다.

<Tip>
  명확히 할 질문은 Claude가 계획을 제안하기 전에 코드베이스를 탐색하고 질문을 하는 [`plan` 모드](/docs/en/agent-sdk/permissions#plan-mode-plan)에서 특히 흔합니다. 따라서 계획 모드는 변경하기 전에 요구 사항을 수집하도록 Claude에 요청하는 대화형 워크플로우에 이상적입니다.
</Tip>

다음 단계는 명확히 할 질문을 처리하는 방법을 보여줍니다:

<Steps>
  <Step title="canUseTool 콜백 전달">
    쿼리 옵션에 `canUseTool` 콜백을 전달합니다. 기본적으로 `AskUserQuestion`을 사용할 수 있습니다. Claude의 기능을 제한하기 위해 `tools` 배열을 지정하는 경우(예: `Read`, `Glob`, `Grep`만 있는 읽기 전용 에이전트), 해당 배열에 `AskUserQuestion`을 포함하세요. 그렇지 않으면 Claude가 명확히 할 질문을 할 수 없습니다:

    <CodeGroup>
      ```python Python theme={null}
      async for message in query(
          prompt="Analyze this codebase",
          options=ClaudeAgentOptions(
              # tools 목록에 AskUserQuestion 포함
              tools=["Read", "Glob", "Grep", "AskUserQuestion"],
              can_use_tool=can_use_tool,
          ),
      ):
          print(message)
      ```

      ```typescript TypeScript theme={null}
      for await (const message of query({
        prompt: "Analyze this codebase",
        options: {
          // tools 목록에 AskUserQuestion 포함
          tools: ["Read", "Glob", "Grep", "AskUserQuestion"],
          canUseTool: async (toolName, input) => {
            // 여기서 명확히 할 질문 처리
          }
        }
      })) {
        console.log(message);
      }
      ```
    </CodeGroup>
  </Step>

  <Step title="AskUserQuestion 감지">
    콜백에서 `toolName`이 `AskUserQuestion`과 같은지 확인하여 다른 도구와 다르게 처리합니다:

    <CodeGroup>
      ```python Python theme={null}
      async def can_use_tool(tool_name: str, input_data: dict, context):
          if tool_name == "AskUserQuestion":
              # 사용자로부터 답변을 수집하는 구현
              return await handle_clarifying_questions(input_data)
          # 다른 도구는 정상 처리
          return await prompt_for_approval(tool_name, input_data)
      ```

      ```typescript TypeScript theme={null}
      canUseTool: async (toolName, input) => {
        if (toolName === "AskUserQuestion") {
          // 사용자로부터 답변을 수집하는 구현
          return handleClarifyingQuestions(input);
        }
        // 다른 도구는 정상 처리
        return promptForApproval(toolName, input);
      };
      ```
    </CodeGroup>
  </Step>

  <Step title="질문 입력 파싱">
    입력에는 `questions` 배열 형태로 Claude의 질문이 포함됩니다. 각 질문에는 `question`(표시할 텍스트), `options`(선택지) 및 `multiSelect`(다중 선택 허용 여부)가 포함됩니다:

    ```json theme={null}
    {
      "questions": [
        {
          "question": "How should I format the output?",
          "header": "Format",
          "options": [
            { "label": "Summary", "description": "Brief overview" },
            { "label": "Detailed", "description": "Full explanation" }
          ],
          "multiSelect": false
        },
        {
          "question": "Which sections should I include?",
          "header": "Sections",
          "options": [
            { "label": "Introduction", "description": "Opening context" },
            { "label": "Conclusion", "description": "Final summary" }
          ],
          "multiSelect": true
        }
      ]
    }
    ```

    전체 필드 설명은 [질문 형식](#question-format)을 참조하세요.
  </Step>

  <Step title="사용자로부터 답변 수집">
    사용자에게 질문을 표시하고 사용자의 선택을 수집합니다. 이를 수행하는 방법은 애플리케이션에 따라 다릅니다(터미널 프롬프트, 웹 폼, 모바일 대화 상자 등).
  </Step>

  <Step title="Claude에 답변 반환">
    각 키가 `question` 텍스트이고 각 값이 선택한 옵션의 `label`인 레코드로 `answers` 객체를 구성합니다:

    | 질문 객체 출처                                               | 용도 |
    | ------------------------------------------------------------ | ---- |
    | `question` 필드 (예: `"How should I format the output?"`)    | 키   |
    | 선택한 옵션의 `label` 필드 (예: `"Summary"`)                 | 값   |

    다중 선택 질문의 경우 레이블 배열을 전달하거나 `", "`로 연결하세요. [자유 텍스트 입력을 지원](#support-free-text-input)하는 경우 사용자의 커스텀 텍스트를 값으로 사용하세요.

    <CodeGroup>
      ```python Python theme={null}
      return PermissionResultAllow(
          updated_input={
              "questions": input_data.get("questions", []),
              "answers": {
                  "How should I format the output?": "Summary",
                  "Which sections should I include?": ["Introduction", "Conclusion"],
              },
          }
      )
      ```

      ```typescript TypeScript theme={null}
      return {
        behavior: "allow",
        updatedInput: {
          questions: input.questions,
          answers: {
            "How should I format the output?": "Summary",
            "Which sections should I include?": "Introduction, Conclusion"
          }
        }
      };
      ```
    </CodeGroup>
  </Step>
</Steps>

### 질문 형식 (Question format)

입력에는 `questions` 배열 형태로 Claude가 생성한 질문이 포함됩니다. 각 질문에는 다음 필드가 있습니다:

| 필드          | 설명                                                                                                                                                 |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `question`    | 표시할 전체 질문 텍스트                                                                                                                              |
| `header`      | 질문의 짧은 레이블 (최대 12자)                                                                                                                       |
| `options`     | 각각 `label`과 `description`이 있는 2~4개 선택지 배열. TypeScript: 선택적으로 `preview` (아래 [참조](#option-previews-typescript))                   |
| `multiSelect` | `true`인 경우 사용자가 여러 옵션을 선택할 수 있음                                                                                                     |

콜백이 받는 구조:

```json theme={null}
{
  "questions": [
    {
      "question": "How should I format the output?",
      "header": "Format",
      "options": [
        { "label": "Summary", "description": "Brief overview of key points" },
        { "label": "Detailed", "description": "Full explanation with examples" }
      ],
      "multiSelect": false
    }
  ]
}
```

#### 옵션 미리보기 (TypeScript)

`toolConfig.askUserQuestion.previewFormat`은 각 옵션에 `preview` 필드를 추가하여 앱이 레이블과 함께 시각적 목업을 표시할 수 있게 합니다. 이 설정이 없으면 Claude는 미리보기를 생성하지 않으며 필드가 없습니다.

| `previewFormat` | `preview`에 포함되는 내용                                                                                    |
| :-------------- | :----------------------------------------------------------------------------------------------------------- |
| 미설정 (기본값) | 필드 없음. Claude가 미리보기를 생성하지 않음.                                                                |
| `"markdown"`    | ASCII 아티팩트 및 펜스 코드 블록                                                                             |
| `"html"`        | 스타일이 적용된 `<div>` 프래그먼트 (SDK는 콜백이 실행되기 전에 `<script>`, `<style>`, `<!DOCTYPE>`를 거부함) |

형식은 세션의 모든 질문에 적용됩니다. Claude는 시각적 비교가 도움이 되는 옵션(레이아웃 선택, 색상 테마)에는 `preview`를 포함하고 그렇지 않은 옵션(예/아니오 확인, 텍스트 전용 선택)에는 생략합니다. 렌더링하기 전에 `undefined` 여부를 확인하세요.

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Help me choose a card layout",
  options: {
    toolConfig: {
      askUserQuestion: { previewFormat: "html" }
    },
    canUseTool: async (toolName, input) => {
      // input.questions[].options[].preview는 HTML 문자열 또는 undefined
      return { behavior: "allow", updatedInput: input };
    }
  }
})) {
  // ...
}
```

HTML 미리보기가 포함된 옵션 예시:

```json theme={null}
{
  "label": "Compact",
  "description": "Title and metric value only",
  "preview": "<div style=\"padding:12px;border:1px solid #ddd;border-radius:8px\"><div style=\"font-size:12px;color:#666\">Active users</div><div style=\"font-size:28px;font-weight:600\">1,284</div></div>"
}
```

### 응답 형식

각 질문의 `question` 필드를 선택한 옵션의 `label`에 매핑하는 `answers` 객체를 반환합니다:

| 필드        | 설명                                                                                                  |
| ----------- | ----------------------------------------------------------------------------------------------------- |
| `questions` | 원본 질문 배열을 그대로 전달함 (도구 처리에 필요)                                                     |
| `answers`   | 키는 질문 텍스트이고 값은 선택한 레이블인 객체                                                        |
| `response`  | 사용자가 구조화된 질문에 답하는 대신 직접 입력한 선택적 자유 형식 응답                                 |

다중 선택 질문의 경우 레이블 배열을 전달하거나 `", "`로 연결하세요. "Other" 옵션과 같은 질문별 자유 텍스트의 경우 [자유 텍스트 입력 지원](#support-free-text-input)에 나와 있는 것처럼 사용자의 커스텀 텍스트를 `answers[question]`에 넣으세요. UI에서 사용자가 질문 카드를 닫고 특정 질문에 대한 답변이 아닌 일반적인 응답을 입력할 수 있는 경우에만 `response`를 설정하세요. `response`가 설정되면 Claude는 질문별 답변 목록 대신 "The user responded: …"를 수신합니다.

```json theme={null}
{
  "questions": [
    // ...
  ],
  "answers": {
    "How should I format the output?": "Summary",
    "Which sections should I include?": ["Introduction", "Conclusion"]
  }
}
```

#### 자유 텍스트 입력 지원

Claude의 사전 정의된 옵션이 항상 사용자가 원하는 바를 포함하지는 않습니다. 사용자가 자체 답변을 입력할 수 있도록 하려면:

* Claude 옵션 뒤에 텍스트 입력을 받는 추가 "Other" 선택지를 표시하세요
* 사용자의 커스텀 텍스트를 답변 값으로 사용하세요 ("Other"라는 단어가 아닌)

전체 구현은 아래 [완전한 예시](#complete-example)를 참조하세요.

### 완전한 예시

Claude는 계속 진행하기 위해 사용자 입력이 필요할 때 명확히 할 질문을 합니다. 예를 들어 모바일 앱의 테크 스택을 결정하는 데 도움을 달라는 요청을 받으면 Claude는 크로스 플랫폼 대 네이티브, 백엔드 선호도 또는 대상 플랫폼에 대해 물어볼 수 있습니다. 이러한 질문은 Claude가 추측하지 않고 사용자의 선호도에 일치하는 결정을 내리도록 도와줍니다.

이 예시는 터미널 애플리케이션에서 이러한 질문을 처리합니다. 각 단계별 동작은 다음과 같습니다:

1. **요청 라우팅**: `canUseTool` 콜백이 도구 이름이 `"AskUserQuestion"`인지 확인하고 전용 핸들러로 라우팅
2. **질문 표시**: 핸들러가 `questions` 배열을 순회하며 번호가 지정된 옵션과 함께 각 질문을 출력
3. **입력 수집**: 사용자는 번호를 입력하여 옵션을 선택하거나 자유 텍스트를 직접 입력할 수 있음(예: "jquery", "i don't know")
4. **답변 매핑**: 코드가 입력이 숫자인지(옵션의 레이블 사용) 또는 자유 텍스트인지(텍스트 직접 사용) 확인
5. **Claude에 반환**: 응답에 원본 `questions` 배열과 `answers` 매핑을 모두 포함

TypeScript 버전은 `ask.ts`로 저장하고 `npx tsx ask.ts`로 실행하거나, Python 버전은 `ask.py`로 저장하고 `python ask.py`로 실행하세요.

<CodeGroup>
  ```python Python theme={null}
  import asyncio

  from claude_agent_sdk import ClaudeAgentOptions, ResultMessage, query
  from claude_agent_sdk.types import HookMatcher, PermissionResultAllow


  def parse_response(response: str, options: list) -> str:
      """사용자 입력을 옵션 번호(들) 또는 자유 텍스트로 파싱합니다."""
      try:
          indices = [int(s.strip()) - 1 for s in response.split(",")]
          labels = [options[i]["label"] for i in indices if 0 <= i < len(options)]
          return ", ".join(labels) if labels else response
      except ValueError:
          return response


  async def handle_ask_user_question(input_data: dict) -> PermissionResultAllow:
      """Claude의 질문을 표시하고 사용자의 답변을 수집합니다."""
      answers = {}

      for q in input_data.get("questions", []):
          print(f"\n{q['header']}: {q['question']}")

          options = q["options"]
          for i, opt in enumerate(options):
              print(f"  {i + 1}. {opt['label']} - {opt['description']}")
          if q.get("multiSelect"):
              print("  (Enter numbers separated by commas, or type your own answer)")
          else:
              print("  (Enter a number, or type your own answer)")

          response = input("Your choice: ").strip()
          answers[q["question"]] = parse_response(response, options)

      return PermissionResultAllow(
          updated_input={
              "questions": input_data.get("questions", []),
              "answers": answers,
          }
      )


  async def can_use_tool(
      tool_name: str, input_data: dict, context
  ) -> PermissionResultAllow:
      # AskUserQuestion을 질문 핸들러로 라우팅
      if tool_name == "AskUserQuestion":
          return await handle_ask_user_question(input_data)
      # 이 예시에서는 다른 도구들을 자동 승인
      return PermissionResultAllow(updated_input=input_data)


  async def prompt_stream():
      yield {
          "type": "user",
          "message": {
              "role": "user",
              "content": "Help me decide on the tech stack for a new mobile app",
          },
      }


  # 필수 해결책: 더미 훅이 can_use_tool을 위해 스트림을 열린 상태로 유지함
  async def dummy_hook(input_data, tool_use_id, context):
      return {"continue_": True}


  async def main():
      async for message in query(
          prompt=prompt_stream(),
          options=ClaudeAgentOptions(
              can_use_tool=can_use_tool,
              hooks={"PreToolUse": [HookMatcher(matcher=None, hooks=[dummy_hook])]},
          ),
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";
  import * as readline from "readline/promises";

  // 터미널에서 사용자 입력을 프롬프트로 받는 헬퍼
  async function prompt(question: string): Promise<string> {
    const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
    const answer = await rl.question(question);
    rl.close();
    return answer;
  }

  // 사용자 입력을 옵션 번호(들) 또는 자유 텍스트로 파싱
  function parseResponse(response: string, options: any[]): string {
    const indices = response.split(",").map((s) => parseInt(s.trim()) - 1);
    const labels = indices
      .filter((i) => !isNaN(i) && i >= 0 && i < options.length)
      .map((i) => options[i].label);
    return labels.length > 0 ? labels.join(", ") : response;
  }

  // Claude의 질문을 표시하고 사용자 답변 수집
  async function handleAskUserQuestion(input: any) {
    const answers: Record<string, string> = {};

    for (const q of input.questions) {
      console.log(`\n${q.header}: ${q.question}`);

      const options = q.options;
      options.forEach((opt: any, i: number) => {
        console.log(`  ${i + 1}. ${opt.label} - ${opt.description}`);
      });
      if (q.multiSelect) {
        console.log("  (Enter numbers separated by commas, or type your own answer)");
      } else {
        console.log("  (Enter a number, or type your own answer)");
      }

      const response = (await prompt("Your choice: ")).trim();
      answers[q.question] = parseResponse(response, options);
    }

    // Claude에게 답변 반환 (원본 questions를 포함해야 함)
    return {
      behavior: "allow",
      updatedInput: { questions: input.questions, answers }
    };
  }

  async function main() {
    for await (const message of query({
      prompt: "Help me decide on the tech stack for a new mobile app",
      options: {
        canUseTool: async (toolName, input) => {
          // AskUserQuestion을 질문 핸들러로 라우팅
          if (toolName === "AskUserQuestion") {
            return handleAskUserQuestion(input);
          }
          // 이 예시에서는 다른 도구들을 자동 승인
          return { behavior: "allow", updatedInput: input };
        }
      }
    })) {
      if ("result" in message) console.log(message.result);
    }
  }

  main();
  ```
</CodeGroup>

## 제한 사항 (Limitations)

* **서브에이전트**: `AskUserQuestion`은 현재 Agent 도구를 통해 생성된 서브에이전트에서는 사용할 수 없습니다
* **질문 제한**: 각 `AskUserQuestion` 호출은 각각 2~4개 옵션이 있는 1~4개 질문을 지원합니다

## 사용자 입력을 받는 기타 방법

`canUseTool` 콜백과 `AskUserQuestion` 도구가 대부분의 승인 및 설명 시나리오를 다루지만, SDK는 사용자 입력을 받는 다른 방법도 제공합니다:

### 스트리밍 입력 (Streaming input)

다음이 필요할 때 [스트리밍 입력](/docs/en/agent-sdk/streaming-vs-single-mode)을 사용하세요:

* **작업 중간에 에이전트 중단**: Claude가 작업하는 동안 취소 신호를 보내거나 방향 전환
* **추가 컨텍스트 제공**: Claude가 물어볼 때까지 기다리지 않고 필요한 정보 추가
* **채팅 인터페이스 구축**: 사용자가 오래 실행되는 작업 동안 후속 메시지를 전송하도록 허용

스트리밍 입력은 승인 체크포인트뿐만 아니라 실행 전체에 걸쳐 사용자가 에이전트와 상호작용하는 대화형 UI에 이상적입니다.

### 커스텀 도구 (Custom tools)

다음이 필요할 때 [커스텀 도구](/docs/en/agent-sdk/custom-tools)를 사용하세요:

* **구조화된 입력 수집**: `AskUserQuestion`의 객관식 형식을 넘어서는 폼, 위저드 또는 다단계 워크플로우 구축
* **외부 승인 시스템 통합**: 기존 티켓팅, 워크플로우 또는 승인 플랫폼 연결
* **도메인 특화 상호작용 구현**: 코드 리뷰 인터페이스나 배포 체크리스트처럼 애플리케이션 요구 사항에 맞춘 도구 생성

커스텀 도구는 상호작용에 대한 완전한 제어권을 제공하지만 내장된 `canUseTool` 콜백을 사용하는 것보다 더 많은 구현 작업이 필요합니다.

## 관련 리소스

* [권한 구성](/docs/en/agent-sdk/permissions): 권한 모드 및 규칙 설정
* [훅으로 실행 제어](/docs/en/agent-sdk/hooks): 에이전트 생애주기의 주요 시점에서 커스텀 코드 실행
* [TypeScript SDK 레퍼런스](/docs/en/agent-sdk/typescript#canusetool): 전체 canUseTool API 문서
