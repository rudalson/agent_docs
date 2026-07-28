> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 권한 구성하기

> 권한 모드, 후크 및 선언적 허용/거부 규칙을 사용하여 에이전트의 도구 사용 방식을 제어합니다.

Claude Agent SDK는 Claude가 도구를 사용하는 방식을 관리할 수 있는 권한 제어 기능을 제공합니다. 권한 모드와 규칙을 사용하여 자동으로 허용되는 항목을 정의하고, 런타임 시에 다른 모든 항목을 처리하려면 [`canUseTool` 콜백](/docs/en/agent-sdk/user-input)을 사용하세요.

<Note>
  이 페이지에서는 권한 모드와 규칙에 대해 다룹니다. 사용자가 런타임 시 도구 요청을 승인하거나 거부하는 대화형 승인 흐름을 구현하려면 [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input)를 참고하세요.
</Note>

## 권한 평가 방식

Claude가 도구 사용을 요청하면 SDK는 다음 순서로 권한을 검사합니다:

<Steps>
  <Step title="후크 (Hooks)">
    [후크](/docs/en/agent-sdk/hooks)를 가장 먼저 실행합니다. 후크는 호출을 즉시 거부하거나 다음 단계로 전달할 수 있습니다. `allow`를 반환하는 후크는 아래의 거부(deny) 및 질문(ask) 규칙 검사를 건너뛰지 않으며, 후크 결과와 관계없이 해당 규칙들이 평가됩니다.
  </Step>

  <Step title="거부 규칙 (Deny rules)">
    `disallowed_tools` 및 [settings.json](/docs/en/settings#permission-settings)의 `deny` 규칙을 검사합니다. 거부 규칙에 일치하면 `bypassPermissions` 모드라 하더라도 도구가 차단됩니다. `Bash`와 같은 단순 이름 거부 규칙은 이 평가가 시작되기 전에 Claude의 컨텍스트에서 도구를 제거하므로 이 단계에서는 `Bash(rm *)`와 같이 범위가 지정된 규칙만 검사됩니다.
  </Step>

  <Step title="질문 규칙 (Ask rules)">
    [settings.json](/docs/en/settings#permission-settings)의 `ask` 규칙을 검사합니다. 질문 규칙에 일치하면 `bypassPermissions` 모드라 하더라도 확인을 위해 [`canUseTool` 콜백](/docs/en/agent-sdk/user-input)으로 넘어갑니다.

    사용자 상호작용이 필요한 도구도 동일하게 작동합니다: `AskUserQuestion` 및 서버가 [`_meta["anthropic/requiresUserInteraction"]`](/docs/en/mcp#require-approval-for-a-specific-tool)을 설정한 MCP 도구는 허용 규칙에 일치하더라도 항상 콜백으로 넘어갑니다. `dontAsk` 모드에서는 이 두 가지 경우가 프롬프트를 표시하지 않으므로 대신 모두 거부됩니다. {/* min-version: 2.1.199 */}MCP 어노테이션에는 Claude Code v2.1.199 이상이 필요합니다.

    조직에서 `ask`로 설정한 [claude.ai 커넥터](/docs/en/mcp#organization-controls-on-connector-tools) 도구도 이 단계에서 일반 흐름을 벗어납니다. `bypassPermissions` 모드이거나 허용 규칙에 일치하더라도 모든 호출이 콜백으로 넘어갑니다. 콜백은 `Your organization requires approval for this tool`이라는 사유를 전달받습니다. `dontAsk` 모드에서는 해당 모드가 프롬프트를 표시하지 않으므로 대신 호출이 거부됩니다.
  </Step>

  <Step title="권한 모드 (Permission mode)">
    활성화된 [권한 모드](#permission-modes)를 적용합니다. `bypassPermissions`는 이 단계에 도달한 모든 요청을 승인합니다. `acceptEdits`는 파일 작업을 승인합니다. `plan`은 허용 규칙에 관계없이 파일 편집 및 셸 쓰기 도구를 `canUseTool` 콜백으로 라우팅하므로 계획 작성 단계에서는 쓰기 작업이 자동 승인될 수 없습니다. 기타 모드는 다음 단계로 넘어갑니다.
  </Step>

  <Step title="허용 규칙 (Allow rules)">
    `allowed_tools` 및 settings.json의 `allow` 규칙을 검사합니다. 규칙에 일치하면 도구가 승인됩니다.
  </Step>

  <Step title="canUseTool 콜백">
    위의 어떤 단계에서도 결정되지 않은 경우, 결정을 위해 [`canUseTool` 콜백](/docs/en/agent-sdk/user-input)을 호출합니다. `dontAsk` 모드에서는 이 단계를 건너뛰고 도구가 거부됩니다.
  </Step>
</Steps>

<img src="https://mintcdn.com/claude-code/jYgs7qigNjO1Badj/images/agent-sdk/permissions-flow.svg?fit=max&auto=format&n=jYgs7qigNjO1Badj&q=85&s=c771ad9085b1277d3708027a49c744bc" alt="Diagram of the six-step permission evaluation flow matching the steps above: a tool request passes through hooks, deny rules, ask rules, permission mode, allow rules, and canUseTool. Hooks, deny rules, and canUseTool can route down to Blocked; permission mode bypass, allow rules, and canUseTool can route up to Execute; ask rules route to canUseTool." width="1180" height="260" data-path="images/agent-sdk/permissions-flow.svg" />

v2.1.198부터 이 평가 순서에서 절대 도달할 수 없는 `canUseTool` 콜백을 전달하면 TypeScript SDK는 쿼리가 생성될 때 Node.js 프로세스 경고를 한 번 출력합니다. 경고 코드는 `CLAUDE_SDK_CAN_USE_TOOL_SHADOWED`입니다. 두 가지 구성이 이를 트리거합니다:

* 권한 모드 단계에 도달하는 모든 호출을 자동 승인하는 `permissionMode: 'bypassPermissions'`
* 콜백을 참조하기 전에 해당 도구 전체를 자동 승인하는 `"Read"`와 같은 단순 `allowedTools` 항목

`Bash(ls *)`와 같은 지정자가 포함된 항목이나 `acceptEdits` 모드는 경고를 트리거하지 않으며, 설정 파일에서 들어오는 허용 규칙은 이 검사에 보이지 않습니다.

`process.on('warning', ...)`으로 수신하여 해당 코드를 기록하거나 억제할 수 있습니다. 모드 및 규칙에 상관없이 모든 도구 호출을 제어하려면 대신 [`PreToolUse` 후크](/docs/en/agent-sdk/hooks)를 사용하세요.

이 페이지는 **허용 및 거부 규칙**과 **권한 모드**에 중점을 둡니다. 다른 단계에 대한 내용은 다음을 참고하세요:

* **후크:** 커스텀 코드를 실행하여 도구 요청을 허용, 거부 또는 수정합니다. [후크로 실행 제어하기](/docs/en/agent-sdk/hooks)를 참고하세요.
* **canUseTool 콜백:** 이전 단계에서 호출이 해결되지 않을 때 런타임 시 사용자에게 승인을 요청합니다. [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input)를 참고하세요.

## 허용 및 거부 규칙

`allowed_tools` 및 `disallowed_tools` (TypeScript: `allowedTools` / `disallowedTools`)는 위 평가 흐름의 허용 및 거부 규칙 목록에 항목을 추가합니다. 허용 규칙은 승인에만 영향을 미칩니다: `allowed_tools`에 나열되지 않은 도구라도 Claude가 여전히 사용할 수 있으며 권한 모드로 넘어갑니다. 거부 규칙은 단순 도구 이름을 지정하는지, 도구 내의 패턴 범위를 지정하는지에 따라 다르게 동작합니다.

| 옵션 | 효과 |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `allowed_tools=["Read", "Grep"]` | `Read` 및 `Grep`이 자동 승인됩니다. 여기에 나열되지 않은 도구도 여전히 존재하며 권한 모드 및 `canUseTool`로 넘어갑니다. |
| `disallowed_tools=["Bash"]` | `Bash` 도구 정의가 요청에서 제거됩니다. Claude는 이 도구를 볼 수 없으며 실행을 시도할 수도 없습니다. |
| `disallowed_tools=["Bash(rm *)"]` | `Bash`가 계속 사용 가능한 상태로 유지됩니다. `rm *`에 일치하는 호출은 `bypassPermissions`를 포함한 모든 권한 모드에서 거부됩니다. 다른 `Bash` 호출은 권한 모드로 넘어갑니다. |
| `disallowed_tools=["*"]` | 모든 도구 정의가 요청에서 제거됩니다. 거부 규칙에서는 도구 이름 글로브(glob)가 지원됩니다: `"*"`는 모든 도구에 일치하고 `"mcp__*"`는 모든 서버의 모든 MCP 도구에 일치합니다. |

허용 규칙은 리터럴 `mcp__<server>__` 접두사 뒤에만 도구 이름 글로브를 허용합니다. 서버 세그먼트에는 글로브가 없어야 규칙이 구성한 특정 서버를 지정할 수 있습니다: `mcp__puppeteer__*`는 `puppeteer` 서버의 모든 도구에 일치하고, `mcp__github__get_*`는 해당 서버의 `get_` 도구들에 일치합니다. `allowed_tools=["*"]` 또는 `allowed_tools=["mcp__*"]`와 같이 고정되지 않은 항목은 시작 경고와 함께 무시되며 아무것도 자동 승인하지 않습니다.

`Read` 및 `Edit`에 대한 범위 지정 규칙은 경로 패턴을 사용합니다. `Edit(path)` 규칙은 `Write` 및 `NotebookEdit`을 포함해 파일을 쓰는 모든 내장 도구를 제어합니다; `Write(path)` 규칙은 파일 권한 검사 시 일치되지 않습니다.

절대 파일 시스템 경로에는 `//path`를 사용하세요: `Edit(//secrets/**)` 거부 규칙은 디스크의 `/secrets` 하위 어디서든 쓰기를 차단합니다. 선두 슬래시가 하나만 있는 `Edit(/secrets/**)`는 규칙의 출처를 기준으로 위치가 지정됩니다. `allowed_tools` 또는 `disallowed_tools`를 통해 전달되는 규칙의 경우 세션의 작업 디렉토리를 의미하므로 디스크의 `/secrets`를 차단하지 않습니다. 4가지 고정 방식과 설정 파일의 규칙 해석 방법에 대해서는 [Read 및 Edit 규칙](/docs/en/permissions#read-and-edit)을 참고하세요.

<Warning>
  **자동 승인된 도구는 절대로 `canUseTool`에 도달하지 않습니다.** `acceptEdits`, `bypassPermissions` 또는 허용 규칙에 의해 이전 단계에서 승인된 도구 호출은 `canUseTool` 콜백을 건너뛰므로, 해당 콜백에 설정한 권한 검사는 그 도구에 대해 소리 없이 우회됩니다. `AskUserQuestion`, [`_meta["anthropic/requiresUserInteraction"]`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구, 그리고 [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구는 허용 규칙에 일치하더라도 여전히 콜백에 도달합니다.

  보호 범위는 항목의 형식에 따라 다릅니다: `Read`나 `mcp__github__get_issue`와 같은 단순 이름은 해당 도구에 대한 모든 호출을 자동 승인하는 반면, `Bash(ls *)`와 같이 범위가 지정된 규칙은 일치하는 호출만 자동 승인하고 다른 `Bash` 호출은 여전히 콜백으로 넘어갑니다. 모든 도구 호출에 대해 반드시 실행되어야 하는 검사의 경우 [`PreToolUse` 후크](/docs/en/agent-sdk/hooks)를 사용하세요: 후크는 다른 모든 단계보다 먼저 실행되며, 후크 거부는 `bypassPermissions` 모드에서도 적용됩니다.
</Warning>

엄격하게 제한된 에이전트의 경우 `allowedTools`와 `permissionMode: "dontAsk"`를 함께 사용하세요. 위 Warning에 나온 항상 프롬프트되는 도구를 제외하고 나열된 도구는 승인되며, 나열되지 않은 나머지 도구는 프롬프트를 띄우지 않고 완전히 거부됩니다:

```typescript theme={null}
const options = {
  allowedTools: ["Read", "Glob", "Grep"],
  permissionMode: "dontAsk"
};
```

<Warning>
  **`allowed_tools`는 `bypassPermissions`를 제약하지 않습니다.** `allowed_tools`는 나열된 도구만 미리 승인할 뿐입니다. 나열되지 않은 도구는 어떠한 허용 규칙에도 일치하지 않고 권한 모드로 넘어가며, 여기서 `bypassPermissions`가 이를 승인해 버립니다. `allowed_tools=["Read"]`와 `permission_mode="bypassPermissions"`를 함께 설정하더라도 `Bash`, `Write`, `Edit`을 포함한 모든 도구가 승인됩니다. `bypassPermissions`가 필요하지만 특정 도구를 차단하고 싶다면 `disallowed_tools`를 사용하세요.
</Warning>

`.claude/settings.json`에서 선언적으로 허용, 거부, 질문 규칙을 구성할 수도 있습니다. 이 규칙들은 `project` 설정 소스가 활성화되어 있을 때 읽히며, 기본 `query()` 옵션에서는 활성화되어 있습니다. `setting_sources` (TypeScript: `settingSources`)를 명시적으로 설정하는 경우 적용을 위해 `"project"`를 포함해야 합니다. 규칙 구문은 [권한 설정](/docs/en/settings#permission-settings)을 참고하세요.

## 권한 모드 (Permission modes)

권한 모드는 Claude의 도구 사용 방식에 대한 전역 제어를 제공합니다. `query()`를 호출할 때 권한 모드를 설정하거나 스트리밍 세션 중에 동적으로 변경할 수 있습니다.

### 사용 가능한 모드

SDK는 다음 권한 모드들을 지원합니다:

| 모드 | 설명 | 도구 동작 |
| :------------------ | :--------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default` | 표준 권한 동작 | 자동 승인 없음; 일치하지 않는 도구는 `canUseTool` 콜백을 트리거함 |
| `dontAsk` | 프롬프트 띄우는 대신 거부 | `allowed_tools`나 규칙에 의해 미리 승인되지 않은 항목은 모두 거부됨; 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools) 및 사용자 상호작용이 필요한 도구는 미리 승인했더라도 거부됨. `canUseTool`은 전혀 호출되지 않음 |
| `acceptEdits` | 파일 편집 자동 승인 | 파일 편집 및 [파일 시스템 작업](#accept-edits-mode-acceptedits)(`mkdir`, `rm`, `mv` 등)이 자동으로 승인됨 |
| `bypassPermissions` | 권한 검사 우회 | 명시적인 [`ask` 규칙](#how-permissions-are-evaluated)에 일치하는 도구, 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), 사용자 상호작용이 필요한 도구를 제외하고 권한 프롬프트 없이 실행됨 (주의해서 사용) |
| `plan` | 계획 작성 모드 | 소스 파일을 편집하지 않고 탐색 및 계획 작성만 수행함; 파일 편집은 자동 승인되지 않으며 `canUseTool` 콜백을 통해 프롬프트를 띄움 |
| `auto` | 모델 분류기 승인 | 모델 분류기가 권한 프롬프트를 승인하거나 거부함. 이용 가능 여부는 [Auto 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)를 참고하세요 |

<Warning>
  **서브에이전트 상속:** 서브에이전트는 부모 세션의 권한 모드를 상속받습니다. 부모가 `bypassPermissions`, `acceptEdits`, 또는 `auto`를 사용하는 경우를 제외하고 [`AgentDefinition`의 `permissionMode`](/docs/en/agent-sdk/typescript#agentdefinition)로 이를 재정의할 수 있습니다. 해당 세 모드는 모든 서브에이전트에 적용되며 서브에이전트별로 재정의할 수 없습니다.

  서브에이전트는 메인 에이전트와 다른 시스템 프롬프트 및 덜 제약된 동작을 가질 수 있으므로, `bypassPermissions`를 상속받으면 완전한 자율 시스템 액세스 권한이 부여됩니다. 명시적인 [`ask` 규칙](#how-permissions-are-evaluated), 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), 사용자 상호작용이 필요한 도구는 여전히 프롬프트를 강제합니다.
</Warning>

### 권한 모드 설정

쿼리를 시작할 때 권한 모드를 한 번 설정하거나, 세션이 활성화되어 있는 동안 동적으로 변경할 수 있습니다.

<Tabs>
  <Tab title="쿼리 시작 시">
    쿼리를 생성할 때 `permission_mode` (Python) 또는 `permissionMode` (TypeScript)를 전달합니다. 이 모드는 동적으로 변경되지 않는 한 전체 세션에 적용됩니다.

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import query, ClaudeAgentOptions


      async def main():
          async for message in query(
              prompt="Help me refactor this code",
              options=ClaudeAgentOptions(
                  permission_mode="default",  # Set the mode here
              ),
          ):
              if hasattr(message, "result"):
                  print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      async function main() {
        for await (const message of query({
          prompt: "Help me refactor this code",
          options: {
            permissionMode: "default" // Set the mode here
          }
        })) {
          if ("result" in message) {
            console.log(message.result);
          }
        }
      }

      main();
      ```
    </CodeGroup>
  </Tab>

  <Tab title="스트리밍 도중">
    세션 중간에 모드를 변경하려면 `set_permission_mode()` (Python) 또는 `setPermissionMode()` (TypeScript)를 호출하세요. 새로운 모드는 이후의 모든 도구 요청에 즉시 적용됩니다. 이를 통해 제한적인 상태로 시작하여 신뢰가 쌓임에 따라 권한을 완화할 수 있습니다. 예를 들어 Claude의 초기 접근 방식을 검토한 후 `acceptEdits`로 전환할 수 있습니다.

    <CodeGroup>
      ```python Python theme={null}
      import asyncio
      from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions


      async def main():
          async with ClaudeSDKClient(
              options=ClaudeAgentOptions(
                  permission_mode="default",  # Start in default mode
              )
          ) as client:
              await client.query("Help me refactor this code")

              # Change mode dynamically mid-session
              await client.set_permission_mode("acceptEdits")

              # Process messages with the new permission mode
              async for message in client.receive_response():
                  if hasattr(message, "result"):
                      print(message.result)


      asyncio.run(main())
      ```

      ```typescript TypeScript theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";

      async function main() {
        const q = query({
          prompt: "Help me refactor this code",
          options: {
            permissionMode: "default" // Start in default mode
          }
        });

        // Change mode dynamically mid-session
        await q.setPermissionMode("acceptEdits");

        // Process messages with the new permission mode
        for await (const message of q) {
          if ("result" in message) {
            console.log(message.result);
          }
        }
      }

      main();
      ```
    </CodeGroup>
  </Tab>
</Tabs>

### 모드 상세 내용

#### 파일 편집 수용 모드 (`acceptEdits`)

Claude가 프롬프트 없이 코드를 편집할 수 있도록 파일 작업을 자동 승인합니다. 다른 도구(파일 시스템 작업이 아닌 Bash 명령어 등)는 여전히 일반 권한을 요구합니다.

**자동 승인되는 작업:**

* 파일 편집 (Edit, Write 도구)
* 파일 시스템 명령어: `mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed`

두 가지 모두 작업 디렉토리 또는 `additionalDirectories` 내부의 경로에만 적용됩니다. 해당 범위를 벗어난 경로 및 보호된 경로에 대한 쓰기는 여전히 프롬프트를 띄웁니다.

**사용 시기:** 프로토타이핑 중이거나 격리된 디렉토리에서 작업할 때와 같이 Claude의 편집을 신뢰하고 빠른 반복 작업을 원할 때.

#### 프롬프트 비활성화 모드 (`dontAsk`)

모든 권한 프롬프트를 거부로 변환합니다. `allowed_tools`, `settings.json` 허용 규칙 또는 후크에 의해 미리 승인된 도구는 정상 실행됩니다. 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools) 및 사용자 상호작용이 필요한 도구는 허용 규칙에 일치하더라도 거부됩니다. 그 외 모든 항목은 `canUseTool`을 호출하지 않고 거부됩니다.

**사용 시기:** 헤드리스(headless) 에이전트를 위해 고정되고 명시적인 도구 영역을 원하고 `canUseTool` 부재에 암묵적으로 의존하기보다 명확한 거부를 선호할 때.

#### 권한 우회 모드 (`bypassPermissions`)

프롬프트 없이 모든 도구 사용을 자동 승인합니다. 후크는 계속 실행되며 필요한 경우 작업을 차단할 수 있습니다.

<Warning>
  극도로 주의해서 사용하세요. 이 모드에서 Claude는 시스템에 대한 완전한 액세스 권한을 가집니다. 가능한 모든 작업을 신뢰할 수 있는 통제된 환경에서만 사용하세요.

  `allowed_tools`는 이 모드를 제한하지 않습니다. 나열된 도구뿐만 아니라 모든 도구가 승인됩니다. 거부 규칙(`disallowed_tools`), 명시적인 `ask` 규칙, 후크는 모드 검사 전에 평가되므로 여전히 도구를 차단할 수 있습니다. 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools) 및 사용자 상호작용이 필요한 도구는 여전히 `canUseTool` 콜백으로 넘어갑니다.
</Warning>

#### 계획 작성 모드 (`plan`)

Claude는 소스 파일을 편집하지 않고 코드베이스를 탐색하고 계획을 생성합니다. 읽기 전용 도구는 default 모드와 동일하게 실행됩니다.

파일 편집은 허용 규칙에 일치하더라도 plan 모드에서는 절대로 자동 승인되지 않습니다. 대신 `canUseTool` 콜백을 통해 프롬프트를 띄웁니다. {/* min-version: 2.1.212 */}Claude Code v2.1.212 이상에서는 `touch` 및 `rm`과 같이 파일을 수정하는 셸 명령어도 동일하게 `canUseTool` 콜백에 도달합니다.

Claude는 계획을 확정하기 전에 요구 사항을 명확히 하기 위해 `AskUserQuestion`을 사용할 수 있습니다. 이러한 프롬프트 처리에 대해서는 [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input#handle-clarifying-questions)를 참고하세요.

**사용 시기:** 코드 검수 중이거나 변경 사항이 적용되기 전에 승인해야 하는 경우와 같이 Claude가 변경 사항을 실행하지 않고 제안만 하도록 하고 싶을 때.

## 관련 리소스

권한 평가 흐름의 다른 단계에 대한 내용:

* [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input): 대화형 승인 프롬프트 및 명확성 요구 질문
* [후크 가이드](/docs/en/agent-sdk/hooks): 에이전트 수명주기의 핵심 시점에서 커스텀 코드 실행
* [권한 규칙](/docs/en/settings#permission-settings): `settings.json` 내 선언적 허용/거부 규칙
