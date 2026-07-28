> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# SDK의 서브에이전트 (Subagents)

> 컨텍스트를 격리하고, 작업을 병렬로 실행하며, Claude Agent SDK 애플리케이션에 특화된 지침을 적용하기 위해 서브에이전트를 정의하고 호출하세요.

서브에이전트는 집중된 하위 작업을 처리하기 위해 기본 에이전트가 생설할 수 있는 별도의 에이전트 인스턴스입니다.
기본 에이전트의 프롬프트에 부담을 주지 않으면서 컨텍스트를 격리하고, 여러 분석을 병렬로 실행하며, 특화된 지침을 적용하는 데 활용하세요.

이 가이드에서는 `agents` 매개변수를 사용하여 SDK에서 서브에이전트를 정의하고 사용하는 방법을 설명합니다.

## 개요 (Overview)

세 가지 방법으로 서브에이전트를 생성할 수 있습니다:

* **프로그래밍 방식**: `query()` 옵션의 `agents` 매개변수 사용. [TypeScript](/docs/en/agent-sdk/typescript#agentdefinition) 및 [Python](/docs/en/agent-sdk/python#agentdefinition) 레퍼런스 참조
* **파일 시스템 기반**: `.claude/agents/` 디렉터리에 markdown 파일로 에이전트 정의. [파일로 서브에이전트 정의](/docs/en/sub-agents) 참조
* **내장 다목적(Built-in general-purpose)**: 별도로 정의하지 않아도 Claude가 Agent 도구를 통해 언제든지 내장 `general-purpose` 서브에이전트를 호출할 수 있음

이 가이드는 SDK 애플리케이션에 권장되는 프로그래밍 방식 접근 방식에 초점을 맞춥니다.

서브에이전트를 정의하면 Claude는 각 서브에이전트의 `description` 필드를 기반으로 호출 여부를 결정합니다. 서브에이전트를 사용할 시기를 설명하는 명확한 설명을 작성하면 Claude가 적절한 작업을 자동으로 위임합니다. 또한 프롬프트에서 이름으로 서브에이전트를 명시적으로 요청할 수도 있습니다(예: "Use the code-reviewer agent to...").

## 서브에이전트 사용의 이점

### 컨텍스트 격리 (Context Isolation)

각 서브에이전트는 자체의 새로운 대화에서 실행됩니다. 중간 도구 호출 및 결과는 서브에이전트 내부에 남아 있으며 최종 메시지만 부모에게 반환됩니다. 서브에이전트 컨텍스트에 포함되는 정확한 내용은 [서브에이전트가 상속하는 항목](#what-subagents-inherit)을 참조하세요.

**예시:** `research-assistant` 서브에이전트는 메인 대화에 해당 내용이 누적되지 않으면서 수십 개의 파일을 탐색할 수 있습니다. 부모는 서브에이전트가 읽은 모든 파일이 아닌 간결한 요약만 수신합니다.

### 병렬 처리 (Parallelization)

여러 서브에이전트가 동시에 실행될 수 있으므로 독립적인 하위 작업이 모든 작업의 시간 합계가 아니라 가장 느린 작업의 시간 내에 완료됩니다.

**예시:** 코드 리뷰 중에 순차적으로 실행하는 대신 `style-checker`, `security-scanner`, `test-coverage` 서브에이전트를 동시에 실행할 수 있습니다.

### 특화된 지침 및 지식

각 서브에이전트는 특정 전문 지식, 모범 사례 및 제약 조건이 포함된 맞춤형 시스템 프롬프트를 가질 수 있습니다.

**예시:** `database-migration` 서브에이전트는 메인 에이전트의 지침에는 불필요한 소음이 될 수 있는 SQL 모범 사례, 롤백 전략 및 데이터 무결성 검사에 대한 자세한 지식을 가질 수 있습니다.

### 도구 제한 사항 (Tool Restrictions)

서브에이전트는 특정 도구로 제한될 수 있으므로 의도하지 않은 작업의 위험을 줄일 수 있습니다.

**예시:** `doc-reviewer` 서브에이전트는 Read 및 Grep 도구에만 접근할 수 있어 분석은 수행하지만 문서를 실수로 수정하지는 않도록 보장할 수 있습니다.

## 서브에이전트 생성하기

### 프로그래밍 방식 정의 (권장)

`agents` 매개변수를 사용하여 코드에서 직접 서브에이전트를 정의합니다. Claude는 `Agent` 도구를 통해 서브에이전트를 호출하므로, 권한 프롬프트 없이 서브에이전트 호출을 자동 승인하려면 `allowedTools`에 `Agent`를 포함하세요.

이 페이지의 대부분의 예시는 최종 결과만 출력합니다. Claude가 직접 답변하지 않고 서브에이전트에 위임했는지 확인하려면 [서브에이전트 호출 감지](#detect-subagent-invocation)를 참조하세요.

이 예시는 읽기 전용 접근 권한을 가진 코드 리뷰어와 명령을 실행할 수 있는 테스트 러너라는 두 가지 서브에이전트를 생성합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


  async def main():
      async for message in query(
          prompt="Review the authentication module for security issues",
          options=ClaudeAgentOptions(
              # 서브에이전트 호출을 위한 Agent를 포함하여 이 도구들을 자동 승인
              allowed_tools=["Read", "Grep", "Glob", "Agent"],
              agents={
                  "code-reviewer": AgentDefinition(
                      # description은 Claude에게 이 서브에이전트를 사용할 시기를 알립니다
                      description="Expert code review specialist. Use for quality, security, and maintainability reviews.",
                      # prompt는 서브에이전트의 동작과 전문 지식을 정의합니다
                      prompt="""You are a code review specialist with expertise in security, performance, and best practices.

  When reviewing code:
  - Identify security vulnerabilities
  - Check for performance issues
  - Verify adherence to coding standards
  - Suggest specific improvements

  Be thorough but concise in your feedback.""",
                      # tools는 서브에이전트가 할 수 있는 작업을 제한합니다 (여기서는 읽기 전용)
                      tools=["Read", "Grep", "Glob"],
                      # model은 이 서브에이전트의 기본 모델을 재정의합니다
                      model="sonnet",
                  ),
                  "test-runner": AgentDefinition(
                      description="Runs and analyzes test suites. Use for test execution and coverage analysis.",
                      prompt="""You are a test execution specialist. Run tests and provide clear analysis of results.

  Focus on:
  - Running test commands
  - Analyzing test output
  - Identifying failing tests
  - Suggesting fixes for failures""",
                      # Bash 접근 권한을 통해 이 서브에이전트가 테스트 명령을 실행할 수 있습니다
                      tools=["Bash", "Read", "Grep"],
                  ),
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
    prompt: "Review the authentication module for security issues",
    options: {
      // 서브에이전트 호출을 위한 Agent를 포함하여 이 도구들을 자동 승인
      allowedTools: ["Read", "Grep", "Glob", "Agent"],
      agents: {
        "code-reviewer": {
          // description은 Claude에게 이 서브에이전트를 사용할 시기를 알립니다
          description:
            "Expert code review specialist. Use for quality, security, and maintainability reviews.",
          // prompt는 서브에이전트의 동작과 전문 지식을 정의합니다
          prompt: `You are a code review specialist with expertise in security, performance, and best practices.

  When reviewing code:
  - Identify security vulnerabilities
  - Check for performance issues
  - Verify adherence to coding standards
  - Suggest specific improvements

  Be thorough but concise in your feedback.`,
          // tools는 서브에이전트가 할 수 있는 작업을 제한합니다 (여기서는 읽기 전용)
          tools: ["Read", "Grep", "Glob"],
          // model은 이 서브에이전트의 기본 모델을 재정의합니다
          model: "sonnet"
        },
        "test-runner": {
          description:
            "Runs and analyzes test suites. Use for test execution and coverage analysis.",
          prompt: `You are a test execution specialist. Run tests and provide clear analysis of results.

  Focus on:
  - Running test commands
  - Analyzing test output
  - Identifying failing tests
  - Suggesting fixes for failures`,
          // Bash 접근 권한을 통해 이 서브에이전트가 테스트 명령을 실행할 수 있습니다
          tools: ["Bash", "Read", "Grep"]
        }
      }
    }
  })) {
    if ("result" in message) console.log(message.result);
  }
  ```
</CodeGroup>

### AgentDefinition 구성

| 필드                | 타입                                                         | 필수 여부 | 설명                                                                                                                                                                                                                            |
| :------------------ | :----------------------------------------------------------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `description`       | `string`                                                     | 예       | 이 에이전트를 사용할 시기에 대한 자연어 설명                                                                                                                                                                                     |
| `prompt`            | `string`                                                     | 예       | 역할과 동작을 정의하는 에이전트의 시스템 프롬프트                                                                                                                                                                                |
| `tools`             | `string[]`                                                   | 아니오   | 허용된 도구 이름 배열. 생략 시 모든 도구 상속                                                                                                                                                                                    |
| `disallowedTools`   | `string[]`                                                   | 아니오   | 에이전트의 도구 세트에서 제거할 도구 이름 배열. MCP 서버 수준 패턴도 허용됨: `mcp__server` 또는 `mcp__server__*`는 해당 서버의 모든 도구를 제거하고 `mcp__*`는 모든 서버의 모든 MCP 도구를 제거함                             |
| `model`             | `string`                                                     | 아니오   | 이 에이전트에 대한 모델 재정의. `'fable'`, `'opus'`, `'sonnet'`, `'haiku'`, `'inherit'`와 같은 에일리어스 또는 전체 모델 ID 허용. 생략 시 기본 메인 모델 적용                                                                   |
| `skills`            | `string[]`                                                   | 아니오   | 시작 시 에이전트 컨텍스트에 사전 로드할 스킬 이름 목록. 목록에 없는 스킬도 Skill 도구를 통해 계속 호출 가능                                                                                                                    |
| `memory`            | `'user' \| 'project' \| 'local'`                             | 아니오   | 이 에이전트의 메모리 소스                                                                                                                                                                                                       |
| `mcpServers`        | `(string \| object)[]`                                       | 아니오   | 이름 또는 인라인 구성을 통해 이 에이전트에 사용할 수 있는 MCP 서버                                                                                                                                                              |
| `initialPrompt`     | `string`                                                     | 아니오   | 이 에이전트가 메인 스레드 에이전트로 실행될 때 첫 번째 사용자 차례로 자동 제출됨. 서브에이전트로 호출될 때는 무시됨                                                                                                            |
| `maxTurns`          | `number`                                                     | 아니오   | 에이전트가 중지되기 전 최대 에이전트 차례 수                                                                                                                                                                                     |
| `background`        | `boolean`                                                    | 아니오   | 호출될 때 이 에이전트를 비동기(non-blocking) 백그라운드 작업으로 실행                                                                                                                                                           |
| `effort`            | `'low' \| 'medium' \| 'high' \| 'xhigh' \| 'max' \| number`  | 아니오   | 이 에이전트의 추론 노력이 수준(Reasoning effort level)                                                                                                                                                                           |
| `permissionMode`    | `PermissionMode`                                             | 아니오   | 이 에이전트 내에서 도구 실행을 위한 권한 모드                                                                                                                                                                                  |

Python SDK에서 `disallowedTools` 및 `mcpServers`와 같은 여러 단어로 된 필드 이름은 Python의 snake\_case 규칙을 따르지 않고 유선 형식(wire format)과 일치하도록 camelCase 철자를 유지합니다. 자세한 내용은 [`AgentDefinition` 레퍼런스](/docs/en/agent-sdk/python#agentdefinition)를 참조하세요.

Claude Code v2.1.198에서 두 가지 서브에이전트 동작이 변경되었습니다:

* 서브에이전트는 기본적으로 백그라운드에서 실행됩니다. [`run_in_background`](/docs/en/agent-sdk/typescript) 입력을 생략한 Agent 도구 호출은 백그라운드 서브에이전트를 시작하며, Claude는 계속 진행하기 전에 결과가 필요할 때 `run_in_background: false`를 설정합니다. v2.1.198 이전에는 `run_in_background`를 생략하면 서브에이전트가 동기적으로 실행되었습니다. Claude의 요청과 관계없이 특정 에이전트에 대한 백그라운드 실행을 강제하려면 `background` 필드를 `true`로 설정하세요.
* 서브에이전트는 메인 세션의 확장 추론(extended thinking) 구성을 상속합니다. 이전 버전에서는 메인 세션의 설정과 관계없이 서브에이전트 내부에서 확장 추론이 비활성화되었습니다.

<Note>
  Claude Code v2.1.172부터 서브에이전트는 자체 서브에이전트를 생성할 수 있습니다. 메인 에이전트보다 5단계 아래의 서브에이전트는 포그라운드에서 실행되는지 백그라운드에서 실행되는지와 관계없이 추가 서브에이전트를 생성할 수 없습니다. 서브에이전트가 다른 서브에이전트를 생성하지 못하도록 하려면 해당 `tools` 배열에서 `Agent`를 생략하거나 `disallowedTools`에 추가하세요. 전체 깊이 규칙은 [중첩된 서브에이전트](/docs/en/sub-agents#spawn-nested-subagents)를 참조하세요.
</Note>

### 파일 시스템 기반 정의 (대안)

`.claude/agents/` 디렉터리에 markdown 파일로 서브에이전트를 정의할 수도 있습니다. 이 접근 방식에 대한 자세한 내용은 [Claude Code 서브에이전트 문서](/docs/en/sub-agents)를 참조하세요. 프로그래밍 방식으로 정의된 에이전트는 동일한 이름을 가진 파일 시스템 기반 에이전트보다 우선합니다.

<Note>
  커스텀 서브에이전트를 정의하지 않더라도 Claude는 내장 `general-purpose` 서브에이전트를 생성할 수 있습니다. 이는 특화된 에이전트를 생성하지 않고도 조사나 탐색 작업을 위임하는 데 유용합니다. 권한 프롬프트 없이 이러한 호출이 자동 승인되도록 `allowedTools`에 `Agent`를 포함하세요.
</Note>

## 서브에이전트가 상속하는 항목

서브에이전트의 컨텍스트 창은 부모 대화 없이 새롭게 시작되지만 비어 있지는 않습니다. 부모에서 서브에이전트로 전달하는 유일한 콘텐츠는 Agent 도구의 프롬프트 문자열이므로 서브에이전트에 필요한 파일 경로, 에러 메시지 또는 결정 사항을 해당 프롬프트에 직접 포함하세요.

[`SendMessage`](/docs/en/tools-reference) 도구가 있는 서브에이전트는 세션에서 실행 중인 다른 이름 있는 에이전트 목록으로 시작하므로 메시지를 보낼 수 있는 이름을 알고 있습니다. Claude Code는 서브에이전트의 첫 번째 차례에 해당 목록을 자동으로 추가합니다. [포크(fork)](/docs/en/sub-agents#fork-the-current-conversation)는 부모 대화를 대신 상속하므로 목록을 받지 못합니다. 이 목록은 Claude Code v2.1.206 이상이 필요합니다.

| 서브에이전트가 수신하는 항목                                                                                                           | 서브에이전트가 수신하지 않는 항목                                 |
| :------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------------------------------------------- |
| 자체 시스템 프롬프트(`AgentDefinition.prompt`) 및 Agent 도구의 프롬프트                                                                | 부모의 대화 기록 또는 도구 결과                                    |
| 프로젝트 CLAUDE.md ([`settingSources`](/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources)를 통해 로드됨) | `AgentDefinition.skills`에 나열되지 않은 사전 로드된 스킬 콘텐츠   |
| 도구 정의 (부모로부터 상속되거나 `tools`에 있는 하위 세트)                                                                             | 부모의 시스템 프롬프트                                             |

<Note>
  부모는 서브에이전트의 최종 메시지를 Agent 도구 결과로 받지만 자체 응답에서 이를 요약할 수 있습니다. 사용자에게 보여주는 응답에 서브에이전트 출력을 그대로 보존하려면 메인 `query()` 호출에 전달하는 프롬프트 또는 `systemPrompt` 옵션에 그렇게 하도록 지침을 포함하세요.

  v2.1.210 이상에서 Claude Code는 부모가 읽기 전에 [최종 메시지에서 지침 모양의 패턴을 스캔](/docs/en/sub-agents#subagent-output-scanning)합니다. 스캔은 세 가지 유형의 패턴을 다르게 처리합니다:

  * **제어 태그 모방**: Claude Code는 하네스만 출력하는 태그(예: `<system-reminder>` 블록)를 해당 위치에서 중립화합니다. 여는 앵글 브래킷 뒤에 백슬래시를 삽입하고 아무것도 삭제하지 않습니다.
  * **권한 구성 언급**: Claude Code는 `.claude/settings.json`, `bypassPermissions` 또는 `--dangerously-skip-permissions`와 같은 권한 구성에 대한 참조를 작성된 그대로 유지합니다.
  * **차례 마커(Turn markers)**: `Human:` 또는 `Assistant:`로 시작하는 줄은 콜론 앞에 백슬래시를 추가하여 메시지가 대화 차례 경계를 모방할 수 없게 합니다.

  제어 태그 또는 권한 구성 일치의 경우 Claude Code는 일치하는 패턴의 이름을 지정하는 `[harness: ...]` 마커 줄을 앞에 추가하며, 차례 마커 일치는 마커 줄을 추가하지 않습니다. 이것이 스캔이 수행하는 유일한 수정 사항이며, 서브에이전트의 텍스트를 제거하거나 의역하지 않습니다.
</Note>

속도 제한과 같이 서브에이전트를 조기에 종료시키는 API 에러는 결과로 전달되지 않습니다. 속도 제한, 과부하 또는 서버 에러로 인해 텍스트 출력을 이미 생성한 포그라운드 서브에이전트가 중단되면 Agent 도구는 서브에이전트가 완료되지 않았다는 메모와 함께 해당 부분 출력을 반환합니다. 아무것도 출력하지 않았거나 도구 호출만 하고 텍스트가 없는 서브에이전트는 `Agent terminated early due to an API error`라는 에러 메시지와 에러 세부 정보와 함께 실패합니다. 포그라운드 및 백그라운드 동작은 [서브에이전트의 API 에러](/docs/en/sub-agents#api-errors-in-subagents)를 참조하세요.

이 부분 출력 처리는 Claude Code v2.1.199 이상이 필요합니다. v2.1.199에서는 속도 제한, 과부하 또는 서버 에러로 인해 도구 호출만 전송된 형태가 차단 메모만 포함된 빈 부분 결과로 남았습니다.

## 서브에이전트 호출하기

### 자동 호출

Claude는 작업 및 각 서브에이전트의 `description`을 기반으로 서브에이전트를 호출할 시기를 자동으로 결정합니다. 예를 들어 "Performance optimization specialist for query tuning" 설명이 포함된 `performance-optimizer` 서브에이전트를 정의하면 프롬프트에서 쿼리 최적화를 언급할 때 Claude가 이를 호출합니다.

Claude가 적절한 서브에이전트에 작업을 매칭할 수 있도록 명확하고 구체적인 설명을 작성하세요.

### 명시적 호출

Claude가 특정 서브에이전트를 사용하도록 보장하려면 프롬프트에서 이름으로 언급하세요:

```text theme={null}
"Use the code-reviewer agent to check the authentication module"
```

이렇게 하면 자동 매칭을 우회하고 지정된 서브에이전트를 직접 호출합니다.

### 동적 에이전트 구성

런타임 조건에 따라 에이전트 정의를 동적으로 생성할 수 있습니다. 이 예시는 엄격한 리뷰에 더 강력한 모델을 사용하여 다양한 엄격도 수준을 가진 보안 리뷰어를 생성합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


  # AgentDefinition을 반환하는 팩토리 함수
  # 이 패턴을 사용하면 런타임 조건에 따라 에이전트를 커스터마이징할 수 있습니다
  def create_security_agent(security_level: str) -> AgentDefinition:
      is_strict = security_level == "strict"
      return AgentDefinition(
          description="Security code reviewer",
          # 엄격도 수준에 따라 프롬프트 커스터마이징
          prompt=f"You are a {'strict' if is_strict else 'balanced'} security reviewer...",
          tools=["Read", "Grep", "Glob"],
          # 핵심 인사이트: 중요도가 높은 리뷰에는 더 뛰어난 모델 사용
          model="opus" if is_strict else "sonnet",
      )


  async def main():
      # 에이전트는 쿼리 시점에 생성되므로 각 요청에서 다른 설정을 사용할 수 있습니다
      async for message in query(
          prompt="Review this PR for security issues",
          options=ClaudeAgentOptions(
              allowed_tools=["Read", "Grep", "Glob", "Agent"],
              agents={
                  # 원하는 구성으로 팩토리 호출
                  "security-reviewer": create_security_agent("strict")
              },
          ),
      ):
          if hasattr(message, "result"):
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query, type AgentDefinition } from "@anthropic-ai/claude-agent-sdk";

  // AgentDefinition을 반환하는 팩토리 함수
  // 이 패턴을 사용하면 런타임 조건에 따라 에이전트를 커스터마이징할 수 있습니다
  function createSecurityAgent(securityLevel: "basic" | "strict"): AgentDefinition {
    const isStrict = securityLevel === "strict";
    return {
      description: "Security code reviewer",
      // 엄격도 수준에 따라 프롬프트 커스터마이징
      prompt: `You are a ${isStrict ? "strict" : "balanced"} security reviewer...`,
      tools: ["Read", "Grep", "Glob"],
      // 핵심 인사이트: 중요도가 높은 리뷰에는 더 뛰어난 모델 사용
      model: isStrict ? "opus" : "sonnet"
    };
  }

  // 에이전트는 쿼리 시점에 생성되므로 각 요청에서 다른 설정을 사용할 수 있습니다
  for await (const message of query({
    prompt: "Review this PR for security issues",
    options: {
      allowedTools: ["Read", "Grep", "Glob", "Agent"],
      agents: {
        // 원하는 구성으로 팩토리 호출
        "security-reviewer": createSecurityAgent("strict")
      }
    }
  })) {
    if ("result" in message) console.log(message.result);
  }
  ```
</CodeGroup>

## 서브에이전트 호출 감지

Claude는 Agent 도구를 통해 서브에이전트를 호출합니다. 서브에이전트가 호출되는 시점을 감지하려면 `name`이 `"Agent"`인 `tool_use` 블록을 확인하세요. 서브에이전트의 컨텍스트 내부 메시지에는 `parent_tool_use_id` 필드가 포함되어 있습니다.

<Note>
  도구 이름은 Claude Code v2.1.63에서 `"Task"`에서 `"Agent"`로 변경되었습니다. 현재 SDK 릴리스는 `tool_use` 블록에서 `"Agent"`를 출력하지만 `system:init` 도구 목록 및 `result.permission_denials[].tool_name`에서는 여전히 `"Task"`를 사용합니다. `block.name`에서 두 값을 모두 확인하면 SDK 버전 전반에 걸쳐 호환성이 확보됩니다.
</Note>

메시지 구조는 SDK마다 다릅니다. Python에서는 `message.content`를 통해 콘텐츠 블록에 직접 접근합니다. TypeScript에서는 `SDKAssistantMessage`가 Claude API 메시지를 감싸므로 `message.message.content`를 통해 콘텐츠에 접근합니다.

이 예시는 스트리밍된 메시지를 순회하며 서브에이전트가 호출될 때와 후속 메시지가 해당 서브에이전트의 실행 컨텍스트 내부에서 발생할 때 로깅합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ToolUseBlock


  async def main():
      async for message in query(
          prompt="Use the code-reviewer agent to review this codebase",
          options=ClaudeAgentOptions(
              allowed_tools=["Read", "Glob", "Grep", "Agent"],
              agents={
                  "code-reviewer": AgentDefinition(
                      description="Expert code reviewer.",
                      prompt="Analyze code quality and suggest improvements.",
                      tools=["Read", "Glob", "Grep"],
                  )
              },
          ),
      ):
          # 서브에이전트 호출 확인. 두 이름 모두 매칭: 이전 SDK 버전은 "Task"를 출력했고,
          # 현재 버전은 "Agent"를 출력합니다.
          if hasattr(message, "content") and message.content:
              for block in message.content:
                  if isinstance(block, ToolUseBlock) and block.name in (
                      "Task",
                      "Agent",
                  ):
                      print(f"Subagent invoked: {block.input.get('subagent_type')}")

          # 이 메시지가 서브에이전트의 컨텍스트 내부에서 왔는지 확인
          if hasattr(message, "parent_tool_use_id") and message.parent_tool_use_id:
              print("  (running inside subagent)")

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
          description: "Expert code reviewer.",
          prompt: "Analyze code quality and suggest improvements.",
          tools: ["Read", "Glob", "Grep"]
        }
      }
    }
  })) {
    const msg = message as any;

    // 서브에이전트 호출 확인. 두 이름 모두 매칭: 이전 SDK 버전은 "Task"를 출력했고,
    // 현재 버전은 "Agent"를 출력합니다.
    for (const block of msg.message?.content ?? []) {
      if (block.type === "tool_use" && (block.name === "Task" || block.name === "Agent")) {
        console.log(`Subagent invoked: ${block.input.subagent_type}`);
      }
    }

    // 이 메시지가 서브에이전트의 컨텍스트 내부에서 왔는지 확인
    if (msg.parent_tool_use_id) {
      console.log("  (running inside subagent)");
    }

    if ("result" in message) {
      console.log(message.result);
    }
  }
  ```
</CodeGroup>

## 서브에이전트 재개 (Resume subagents)

새로 시작하지 않고 중단된 부분부터 계속하려면 서브에이전트를 재개할 수 있습니다. 재개된 서브에이전트는 이전의 모든 도구 호출, 결과 및 추론을 포함한 전체 대화 기록을 유지합니다.

서브에이전트가 완료되면 Agent 도구 결과에 `agentId: <id>`가 포함된 텍스트 블록이 들어 있습니다. 내장된 [`Explore` 및 `Plan` 에이전트](/docs/en/sub-agents#built-in-subagents)는 단일 실행 방식이므로 `agentId`를 반환하지 않으므로 재개해야 할 경우 커스텀 에이전트나 `general-purpose`를 사용하세요. 서브에이전트를 프로그래밍 방식으로 재개하려면:

1. **세션 ID 캡처**: 첫 번째 쿼리 동안 메시지에서 `session_id` 추출
2. **에이전트 ID 추출**: Agent 도구 결과 텍스트에서 `agentId` 파싱
3. **세션 재개**: 두 번째 쿼리의 옵션에 `resume: sessionId`를 전달하고 프롬프트에 에이전트 ID 포함

<Note>
  서브에이전트의 트랜스크립트에 접근하려면 동일한 세션을 재개해야 합니다. 각 `query()` 호출은 기본적으로 새 세션을 시작하므로 동일한 세션에서 계속하려면 `resume: sessionId`를 전달하세요.

  커스텀 에이전트를 사용하는 경우 두 쿼리 모두에 대해 `agents` 매개변수에 동일한 에이전트 정의를 전달하세요.
</Note>

아래 예시는 커스텀 `endpoint-finder` 에이전트를 정의합니다. 첫 번째 쿼리는 이를 실행하고 Agent 도구 결과에서 세션 ID와 에이전트 ID를 캡처한 다음, 두 번째 쿼리는 첫 번째 분석의 컨텍스트가 필요한 후속 질문을 하기 위해 세션을 재개합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  import re
  from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ToolResultBlock

  AGENTS = {
      "endpoint-finder": AgentDefinition(
          description="Locates and catalogs API endpoints in a codebase.",
          prompt="You find and document API endpoints. Report each endpoint's path, method, and handler.",
          tools=["Read", "Grep", "Glob"],
      )
  }


  def extract_agent_id(block: ToolResultBlock) -> str | None:
      """Agent 도구 결과의 텍스트 콘텐츠에서 agentId를 추출합니다."""
      parts = block.content if isinstance(block.content, list) else [{"text": block.content}]
      for part in parts:
          if match := re.search(r"agentId:\s*([\w-]+)", part.get("text") or ""):
              return match.group(1)
      return None


  async def main():
      agent_id = None
      session_id = None

      # 첫 번째 호출 - endpoint-finder 서브에이전트 실행
      try:
          async for message in query(
              prompt="Use the endpoint-finder agent to find all API endpoints in this codebase",
              options=ClaudeAgentOptions(allowed_tools=["Read", "Grep", "Glob", "Agent"], agents=AGENTS),
          ):
              # ResultMessage에서 session_id 캡처 (이 세션을 재개하는 데 필요함)
              if hasattr(message, "session_id"):
                  session_id = message.session_id
              # 도구 결과에서 agentId 트레일러 검색
              for block in getattr(message, "content", None) or []:
                  if isinstance(block, ToolResultBlock):
                      agent_id = extract_agent_id(block) or agent_id
              # 최종 결과 출력
              if hasattr(message, "result"):
                  print(message.result)
      except Exception as error:
          # 단일 실행 query()는 오류 결과 이후 에러를 발생시키므로,
          # session_id와 agent_id는 위의 루프에서 이미 캡처되었습니다.
          print(f"Session ended with an error: {error}")

      # 두 번째 호출 - 재개 및 후속 질문
      if agent_id and session_id:
          async for message in query(
              prompt=f"Resume agent {agent_id} and list the top 3 most complex endpoints",
              options=ClaudeAgentOptions(
                  allowed_tools=["Read", "Grep", "Glob", "Agent"], agents=AGENTS, resume=session_id
              ),
          ):
              if hasattr(message, "result"):
                  print(message.result)
      else:
          print("No agentId found in the first query, so there is no subagent to resume.")


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query, type SDKMessage } from "@anthropic-ai/claude-agent-sdk";

  const agents = {
    "endpoint-finder": {
      description: "Locates and catalogs API endpoints in a codebase.",
      prompt: "You find and document API endpoints. Report each endpoint's path, method, and handler.",
      tools: ["Read", "Grep", "Glob"]
    }
  };

  // 중첩된 블록 타입을 순회하지 않고 agentId를 찾기 위해 콘텐츠를 문자열로 변환
  function extractAgentId(message: SDKMessage): string | undefined {
    if (message.type !== "assistant" && message.type !== "user") return undefined;
    const content = JSON.stringify(message.message.content);
    const match = content.match(/agentId:\s*([\w-]+)/);
    return match?.[1];
  }

  let agentId: string | undefined;
  let sessionId: string | undefined;

  // 첫 번째 호출 - endpoint-finder 서브에이전트 실행
  try {
    for await (const message of query({
      prompt: "Use the endpoint-finder agent to find all API endpoints in this codebase",
      options: { allowedTools: ["Read", "Grep", "Glob", "Agent"], agents }
    })) {
      // ResultMessage에서 session_id 캡처 (이 세션을 재개하는 데 필요함)
      if ("session_id" in message) sessionId = message.session_id;
      // 메시지 콘텐츠에서 agentId 검색 (Agent 도구 결과에 나타남)
      const extractedId = extractAgentId(message);
      if (extractedId) agentId = extractedId;
      // 최종 결과 출력
      if ("result" in message) console.log(message.result);
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과 이후 에러를 발생시키므로,
    // sessionId와 agentId는 위의 루프에서 이미 캡처되었습니다.
    console.error(`Session ended with an error: ${error}`);
  }

  // 두 번째 호출 - 재개 및 후속 질문
  if (agentId && sessionId) {
    for await (const message of query({
      prompt: `Resume agent ${agentId} and list the top 3 most complex endpoints`,
      options: { allowedTools: ["Read", "Grep", "Glob", "Agent"], agents, resume: sessionId }
    })) {
      if ("result" in message) console.log(message.result);
    }
  } else {
    console.log("No agentId found in the first query, so there is no subagent to resume.");
  }
  ```
</CodeGroup>

서브에이전트 트랜스크립트는 메인 대화와 독립적으로 지속됩니다:

* **메인 대화 압축**: 메인 대화가 압축되어도 서브에이전트 트랜스크립트는 영향을 받지 않습니다. 별도의 파일에 저장됩니다.
* **세션 지속성**: 서브에이전트 트랜스크립트는 해당 세션 내에서 지속됩니다. 동일한 세션을 재개하여 Claude Code를 다시 시작한 후에도 서브에이전트를 재개할 수 있습니다.
* **자동 정리**: 트랜스크립트는 기본적으로 30일인 `cleanupPeriodDays` 설정을 기준으로 정리됩니다.

## 도구 제한 사항 (Tool restrictions)

서브에이전트는 `tools` 필드를 통해 제한된 도구 접근 권한을 가질 수 있습니다:

* **필드 생략**: 에이전트가 사용 가능한 모든 도구 상속 (기본값)
* **도구 지정**: 에이전트는 나열된 도구만 사용할 수 있음

이 예시는 코드를 검토할 수 있지만 파일을 수정하거나 명령어를 실행할 수 없는 읽기 전용 분석 에이전트를 생성합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


  async def main():
      async for message in query(
          prompt="Analyze the architecture of this codebase",
          options=ClaudeAgentOptions(
              allowed_tools=["Read", "Grep", "Glob", "Agent"],
              agents={
                  "code-analyzer": AgentDefinition(
                      description="Static code analysis and architecture review",
                      prompt="""You are a code architecture analyst. Analyze code structure,
  identify patterns, and suggest improvements without making changes.""",
                      # 읽기 전용 도구: Edit, Write 또는 Bash 접근 권한 없음
                      tools=["Read", "Grep", "Glob"],
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
    prompt: "Analyze the architecture of this codebase",
    options: {
      allowedTools: ["Read", "Grep", "Glob", "Agent"],
      agents: {
        "code-analyzer": {
          description: "Static code analysis and architecture review",
          prompt: `You are a code architecture analyst. Analyze code structure,
  identify patterns, and suggest improvements without making changes.`,
          // 읽기 전용 도구: Edit, Write 또는 Bash 접근 권한 없음
          tools: ["Read", "Grep", "Glob"]
        }
      }
    }
  })) {
    if ("result" in message) console.log(message.result);
  }
  ```
</CodeGroup>

### 일반적인 도구 조합

| 유스케이스          | 도구                                    | 설명                                                |
| :----------------- | :-------------------------------------- | :-------------------------------------------------- |
| 읽기 전용 분석      | `Read`, `Grep`, `Glob`                  | 코드를 검토할 수 있으나 수정 또는 실행 불가           |
| 테스트 실행        | `Bash`, `Read`, `Grep`                  | 명령을 실행하고 출력을 분석할 수 있음                |
| 코드 수정          | `Read`, `Edit`, `Write`, `Grep`, `Glob` | 명령 실행 없는 완전한 읽기/쓰기 접근 권한            |
| 전체 접근 권한      | 모든 도구                               | 부모로부터 모든 도구를 상속함 (`tools` 필드 생략)    |

## 동적 워크플로우로 확장 (Scale up with dynamic workflows)

서브에이전트는 차례당 몇 개의 위임된 작업에 잘 작동합니다. 수십에서 수백 개의 에이전트를 조율하는 실행의 경우, 대화 컨텍스트 외부에서 런타임이 실행하는 스크립트로 조율을 이동하는 `Workflow` 도구를 사용하세요. 워크플로우가 차례별 서브에이전트 위임과 어떻게 다른지는 [동적 워크플로우](/docs/en/workflows)를 참조하세요.

`Workflow` 도구는 TypeScript Agent SDK v0.3.149 이상에서 사용할 수 있습니다. 워크플로우 실행을 자동 승인하려면 `allowedTools`에 `Workflow`를 포함하세요. 도구 입력 및 출력 스키마는 [TypeScript 레퍼런스](/docs/en/agent-sdk/typescript#workflow)에 나열되어 있습니다.

## 문제 해결 (Troubleshooting)

### Claude가 서브에이전트에 위임하지 않음

Claude가 서브에이전트에 위임하지 않고 직접 작업을 완료하는 경우:

* **Agent 호출 승인 여부 확인**: 서브에이전트 호출을 자동 승인하려면 `allowedTools`에 `Agent`를 포함하세요. 포함하지 않으면 Agent 호출이 `canUseTool` 콜백으로 넘어가거나 `dontAsk` 모드에서는 거부됩니다.
* **명시적 프롬프팅 사용**: 프롬프트에서 서브에이전트를 이름으로 언급하세요(예: "Use the code-reviewer agent to...").
* **명확한 설명 작성**: Claude가 작업을 적절하게 매칭할 수 있도록 서브에이전트를 언제 사용할지 정확히 설명하세요.

### 파일 시스템 기반 에이전트가 로드되지 않음

Claude Code는 `~/.claude/agents/` 및 `.claude/agents/`를 감시하고 몇 초 내에 새 에이전트 파일이나 편집된 에이전트 파일을 감지하며 다시 시작할 필요가 없습니다. 정의가 나타나지 않으면 다음 원인을 확인하세요:

* **새로운 `agents` 디렉터리**: 감시자는 세션이 시작될 때 존재했던 디렉터리만 포함하므로 새 디렉터리의 첫 번째 파일은 세션을 다시 시작해야 합니다. 이것이 가장 흔한 원인입니다.
* **유효하지 않은 프론트매터 또는 중복된 `name`**: 파일의 YAML 및 기존 에이전트가 이미 `name`을 사용하고 있는지 확인하세요.
* **`--disable-slash-commands`**: 이 플래그로 시작된 세션은 이 디렉터리를 감시하지 않으며 항상 새 파일을 로드하려면 다시 시작해야 합니다.
* **동일한 이름의 프로그래밍 방식 에이전트**: `query()`에 전달된 `agents`는 동일한 이름을 가진 파일 시스템 에이전트를 오버라이드합니다.

파일 형식은 [서브에이전트 파일 작성 방법](/docs/en/sub-agents#write-subagent-files)을 참조하세요.

### Windows에서 긴 프롬프트 오류

Windows에서는 매우 긴 프롬프트를 가진 서브에이전트가 8191자 명령줄 길이 제한으로 인해 실패할 수 있습니다. 프롬프트를 간결하게 유지하거나 복잡한 지침에는 파일 시스템 기반 에이전트를 사용하세요.

## 관련 문서

* [Claude Code 서브에이전트](/docs/en/sub-agents): 파일 시스템 기반 정의를 포함한 포괄적인 서브에이전트 문서
* [동적 워크플로우](/docs/en/workflows): 한 대화에 담기에는 너무 큰 작업을 위해 스크립트에서 여러 서브에이전트를 조율
* [SDK 개요](/docs/en/agent-sdk/overview): Claude Agent SDK 시작하기
