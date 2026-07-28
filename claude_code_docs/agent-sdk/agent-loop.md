> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 에이전트 루프의 작동 방식

> SDK 에이전트를 구동하는 메시지 수명 주기, 도구 실행, 컨텍스트 윈도우, 아키텍처를 이해합니다.

Agent SDK를 사용하면 Claude Code의 자율 에이전트 루프를 자체 애플리케이션에 임베딩할 수 있습니다. SDK는 도구, 권한, 비용 제한, 출력을 프로그래밍 방식으로 제어할 수 있게 해주는 독립형 패키지입니다. 이를 사용하기 위해 Claude Code CLI를 설치할 필요는 없습니다.

에이전트를 시작하면 SDK는 [Claude Code를 구동하는 동일한 실행 루프](/docs/en/how-claude-code-works#the-agentic-loop)를 실행합니다. 즉, Claude가 프롬프트를 평가하고, 작업을 수행하기 위해 도구를 호출하고, 결과를 받아 작업이 완료될 때까지 반복합니다. 이 페이지에서는 에이전트를 효과적으로 구축, 디버깅, 최적화할 수 있도록 해당 루프 내부에서 일어나는 일들을 설명합니다.

## 루프 개요

모든 에이전트 세션은 동일한 주기를 따릅니다:

<img src="https://mintcdn.com/claude-code/ikqp3_70mqIahteV/images/agent-loop-diagram.svg?fit=max&auto=format&n=ikqp3_70mqIahteV&q=85&s=1c6e8f28d80dba14a7287419656f1237" alt="Diagram of the agent loop: your prompt enters the agentic loop, where Claude evaluates and either requests tool calls, whose results feed back into another evaluation, or returns the final answer" width="720" height="212" data-path="images/agent-loop-diagram.svg" />

1. **프롬프트 수신.** Claude는 시스템 프롬프트, 도구 정의, 대화 기록과 함께 사용자의 프롬프트를 수신합니다. SDK는 세션 메타데이터가 포함된 `"init"` 하위 유형의 [`SystemMessage`](#메시지-유형)를 생성(yield)합니다.
2. **평가 및 응답.** Claude는 현재 상태를 평가하고 진행 방식을 결정합니다. 텍스트로 응답하거나, 하나 이상의 도구 호출을 요청하거나, 둘 다 수행할 수 있습니다. SDK는 텍스트 및 요청된 도구 호출이 포함된 [`AssistantMessage`](#메시지-유형)를 생성합니다.
3. **도구 실행.** SDK는 요청된 각 도구를 실행하고 결과를 수집합니다. 각 도구 결과 세트는 다음 결정을 위해 Claude에게 다시 전달됩니다. [후크(hooks)](/docs/en/agent-sdk/hooks)를 사용하여 도구 호출이 실행되기 전에 가로채거나 수정하거나 차단할 수 있습니다.
4. **반복.** 2단계와 3단계가 하나의 주기로 반복됩니다. 하나의 완전한 주기가 1턴(turn)입니다. Claude는 더 이상 도구 호출이 없는 응답을 생성할 때까지 도구 호출과 결과 처리를 계속합니다.
5. **결과 반환.** SDK는 텍스트 응답(도구 호출 없음)이 포함된 최종 [`AssistantMessage`](#메시지-유형)를 생성한 다음, 최종 텍스트, 토큰 사용량, 비용 및 세션 ID가 포함된 [`ResultMessage`](#메시지-유형)를 생성합니다.

간단한 질문("여기 어떤 파일들이 있나요?")은 `Glob`을 호출하고 결과를 응답하는 한두 턴만 소요될 수 있습니다. 복잡한 작업("인증 모듈을 리팩토링하고 테스트를 업데이트해줘")은 파일을 읽고, 코드를 편집하고, 테스트를 실행하면서 각 결과에 따라 Claude가 접근 방식을 조정하므로 여러 턴에 걸쳐 수십 개의 도구 호출이 연결될 수 있습니다.

## 턴(Turn)과 메시지

턴은 루프 내부의 1회 왕복입니다. Claude가 도구 호출을 포함하는 출력을 생성하면 SDK가 해당 도구를 실행하고 결과가 Claude에게 자동으로 다시 전달됩니다. 이 작업은 코드 제어권을 반환하지 않고 수행됩니다. 턴은 Claude가 도구 호출이 없는 출력을 생성할 때까지 계속되며, 이 시점에서 루프가 끝나고 최종 결과가 전달됩니다.

"auth.ts의 실패하는 테스트를 수정해줘"라는 프롬프트에 대한 전체 세션이 어떻게 보일지 생각해 보세요.

먼저 SDK는 프롬프트를 Claude에게 보내고 세션 메타데이터가 포함된 [`SystemMessage`](#메시지-유형)를 생성합니다. 그런 다음 루프가 시작됩니다:

1. **턴 1:** Claude가 `npm test`를 실행하기 위해 `Bash`를 호출합니다. SDK는 도구 호출이 포함된 [`AssistantMessage`](#메시지-유형)를 생성하고, 명령을 실행한 다음, 출력(3개 실패)이 포함된 [`UserMessage`](#메시지-유형)를 생성합니다.
2. **턴 2:** Claude가 `auth.ts` 및 `auth.test.ts`에 대해 `Read`를 호출합니다. SDK는 파일 내용을 반환하고 `AssistantMessage`를 생성합니다.
3. **턴 3:** Claude가 `auth.ts`를 수정하기 위해 `Edit`를 호출한 다음, `npm test`를 다시 실행하기 위해 `Bash`를 호출합니다. 3개 테스트가 모두 통과합니다. SDK는 `AssistantMessage`를 생성합니다.
4. **최종 턴:** Claude가 도구 호출 없이 텍스트 전용 응답을 생성합니다: "인증 버그를 수정했으며, 3개 테스트가 모두 통과했습니다." SDK는 이 텍스트가 포함된 최종 `AssistantMessage`를 생성한 후, 비용 및 사용량이 포함된 동일한 텍스트의 [`ResultMessage`](#메시지-유형)를 생성합니다.

이 과정은 도구 호출이 있는 3개 턴과 1개의 최종 텍스트 전용 응답 턴으로 총 4턴이었습니다.

도구 사용 턴만 카운트하는 `max_turns` / `maxTurns`를 사용하여 루프를 제한할 수 있습니다. 예를 들어 위의 루프에서 `max_turns=2`를 설정하면 편집 단계 이전에 중단되었을 것입니다. 지출 임계값을 기준으로 턴을 제한하기 위해 `max_budget_usd` / `maxBudgetUsd`를 사용할 수도 있습니다.

제한이 없으면 루프는 Claude가 스스로 완료할 때까지 실행되며, 이는 명확하게 범위가 지정된 작업에는 문제가 없지만 모호한 프롬프트("이 코드베이스를 개선해줘")에서는 길게 실행될 수 있습니다. 예산 설정은 프로덕션 에이전트를 위한 좋은 기본값입니다. 옵션 참조는 아래의 [턴과 예산](#턴과-예산)을 참조하세요.

## 메시지 유형

루프가 실행됨에 따라 SDK는 메시지 스트림을 생성합니다. 각 메시지에는 루프의 어느 단계에서 왔는지 알려주는 유형이 전달됩니다. 5가지 핵심 유형은 다음과 같습니다:

* **`SystemMessage`:** 세션 수명 주기 이벤트. `subtype` 필드로 구분됩니다:

  * `"init"`: 실행에 대한 세션 메타데이터. 세션 시작 중에 `SessionStart` 또는 `Setup` 후크가 실행되면 해당 [후크 수명 주기 메시지](/docs/en/agent-sdk/typescript#sdkhookstartedmessage)가 `init` 메시지보다 먼저 도착합니다.
  * `"compact_boundary"`: [압축(compaction)](#자동-압축) 후에 발생합니다.
  * `"informational"`: 루프의 일반 텍스트 상태 배너.
  * `"worker_shutting_down"`: 호스트 종료 또는 Remote Control 연결 해제로 인해 현재 턴 이후 루프가 종료됨을 나타냅니다.

  TypeScript에서는 `"init"` 이외의 각 하위 유형이 `SDKSystemMessage`의 하위 유형이 아니라 [`SDKMessage` 유니온](/docs/en/agent-sdk/typescript#sdkmessage)에서 고유한 유형입니다.
* **`AssistantMessage`:** 최종 텍스트 전용 응답을 포함하여 각 Claude 응답 후에 방출됩니다. 해당 턴의 텍스트 콘텐츠 블록 및 도구 호출 블록이 포함되어 있습니다.
* **`UserMessage`:** Claude에게 다시 전송된 도구 결과 콘텐츠와 함께 각 도구 실행 후에 방출됩니다. 또한 루프 중간에 스트리밍하는 모든 사용자 입력에 대해 방출됩니다.
* **`StreamEvent`:** 부분 메시지가 활성화된 경우에만 방출됩니다. 원시 API 스트리밍 이벤트(텍스트 델타, 도구 입력 청크)가 포함되어 있습니다. [응답 스트리밍](/docs/en/agent-sdk/streaming-output)을 참조하세요.
* **`ResultMessage`:** 에이전트 루프의 끝을 표시합니다. 최종 텍스트 결과, 토큰 사용량, 비용 및 세션 ID가 포함되어 있습니다. 작업이 성공했는지 아니면 한계에 도달했는지 확인하려면 `subtype` 필드를 검사하세요. `prompt_suggestion`과 같은 소수의 후속 시스템 이벤트가 그 뒤에 도착할 수 있으므로, 결과에서 중단(break)하지 말고 스트림을 완료할 때까지 반복하세요. [결과 처리하기](#결과-처리하기)를 참조하세요.

이 5가지 유형은 전체 에이전트 루프 수명 주기를 다룹니다. 두 SDK 모두 루프를 구동하는 데 필수적이지 않은 속도 제한 상태 및 작업 알림과 같은 관찰 가능성 이벤트도 방출합니다. 전체 목록은 [Python 메시지 유형 참조](/docs/en/agent-sdk/python#message-types) 및 [TypeScript 메시지 유형 참조](/docs/en/agent-sdk/typescript#message-types)를 참조하세요.

### 메시지 처리하기

구축하려는 항목에 따라 어떤 메시지를 처리할지 결정됩니다:

* **최종 결과만:** `ResultMessage`를 처리하여 출력, 비용, 작업의 성공 여부 또는 제한 도달 여부를 가져옵니다.
* **진행 상태 업데이트:** `AssistantMessage`를 처리하여 각 턴에서 Claude가 호출한 도구를 포함하여 무엇을 하고 있는지 확인합니다.
* **실시간 스트리밍:** 부분 메시지를 활성화(Python의 `include_partial_messages`, TypeScript의 `includePartialMessages`)하여 실시간으로 `StreamEvent` 메시지를 받습니다. [실시간 응답 스트리밍](/docs/en/agent-sdk/streaming-output)을 참조하세요.

메시지 유형을 확인하는 방식은 SDK에 따라 다릅니다:

* **Python:** `claude_agent_sdk`에서 가져온 클래스에 대해 `isinstance()`로 메시지 유형을 확인합니다(예: `isinstance(message, ResultMessage)`).
* **TypeScript:** `type` 문자열 필드를 확인합니다(예: `message.type === "assistant"`). `AssistantMessage` 및 `UserMessage`는 원시 API 메시지를 `.message` 필드로 감싸므로 콘텐츠 블록은 `message.content`가 아니라 `message.message.content`에 있습니다.

<Accordion title="예시: 메시지 유형 확인 및 결과 처리">
  <CodeGroup>
    ```python Python theme={null}
    import asyncio
    from claude_agent_sdk import query, AssistantMessage, ResultMessage


    async def main():
        try:
            async for message in query(prompt="Summarize this project"):
                if isinstance(message, AssistantMessage):
                    print(f"Turn completed: {len(message.content)} content blocks")
                if isinstance(message, ResultMessage):
                    if message.subtype == "success":
                        print(message.result)
                    else:
                        print(f"Stopped: {message.subtype}")
        except Exception as error:
            # 단발성 query()는 오류 결과를 생성한 후 예외를 발생시킵니다.
            # 오류 결과로 인한 실패였다면 위의 오류 분기가 이미 실행된 상태입니다.
            # 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
            print(f"Session ended with an error: {error}")


    asyncio.run(main())
    ```

    ```typescript TypeScript theme={null}
    import { query } from "@anthropic-ai/claude-agent-sdk";

    try {
      for await (const message of query({ prompt: "Summarize this project" })) {
        if (message.type === "assistant") {
          console.log(`Turn completed: ${message.message.content.length} content blocks`);
        }
        if (message.type === "result") {
          if (message.subtype === "success") {
            console.log(message.result);
          } else {
            console.log(`Stopped: ${message.subtype}`);
          }
        }
      }
    } catch (error) {
      // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
      // 오류 결과로 인한 실패였다면 위의 오류 분기가 이미 실행된 상태입니다.
      // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
      console.log(`Session ended with an error: ${error}`);
    }
    ```
  </CodeGroup>
</Accordion>

## 도구 실행

도구는 에이전트에게 조치를 취할 수 있는 능력을 부여합니다. 도구가 없으면 Claude는 텍스트로만 응답할 수 있습니다. 도구가 있으면 Claude는 파일을 읽고, 명령을 실행하고, 코드를 검색하고, 외부 서비스와 상호작용할 수 있습니다.

### 내장 도구

SDK에는 Claude Code를 구동하는 것과 동일한 도구가 포함되어 있습니다:

| 범주 | 도구 | 역할 |
| :------------------ | :-------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| **파일 작업** | `Read`, `Edit`, `Write` | 파일 읽기, 수정, 생성 |
| **검색** | `Glob`, `Grep` | 패턴으로 파일 찾기, 정규식으로 콘텐츠 검색 |
| **실행** | `Bash` | 쉘 명령, 스크립트, git 작업 실행 |
| **웹** | `WebSearch`, `WebFetch` | 웹 검색, 페이지 가져오기 및 파싱 |
| **탐색** | `ToolSearch` | 모든 도구를 미리 로드하는 대신 요청 시 동적으로 도구를 찾아 로드 |
| **오케스트레이션** | `Agent`, `Skill`, `AskUserQuestion`, `TaskCreate`, `TaskUpdate` | 서브에이전트 생성, 스킬 호출, 사용자에게 질문, 작업 추적 |

내장 도구 외에도 다음을 수행할 수 있습니다:

* [MCP 서버](/docs/en/agent-sdk/mcp)로 **외부 서비스 연결**(데이터베이스, 브라우저, API)
* [커스텀 도구 핸들러](/docs/en/agent-sdk/custom-tools)로 **커스텀 도구 정의**
* 재사용 가능한 워크플로우를 위해 [설정 소스](/docs/en/agent-sdk/claude-code-features)를 통해 **프로젝트 스킬 로드**

### 도구 권한

Claude는 작업에 따라 호출할 도구를 결정하지만, 사용자는 해당 호출의 실행 허용 여부를 제어할 수 있습니다. 특정 도구를 자동 승인하거나, 다른 도구를 완전히 차단하거나, 모든 도구에 승인을 요구할 수 있습니다. 실행 대상을 결정하기 위해 세 가지 옵션이 함께 작동합니다:

* **`allowed_tools` / `allowedTools`**는 나열된 도구를 자동 승인합니다. 허용된 도구 목록에 `["Read", "Glob", "Grep"]`이 지정된 읽기 전용 에이전트는 프롬프트 확인 없이 해당 도구를 실행합니다. 나열되지 않은 도구도 사용할 수 있지만 권한이 필요합니다.
* **`disallowed_tools` / `disallowedTools`**는 다른 설정과 상관없이 나열된 도구를 차단합니다. 도구가 실행되기 전에 규칙을 확인하는 순서는 [권한](/docs/en/agent-sdk/permissions)을 참조하세요.
* **`permission_mode` / `permissionMode`**는 허용 또는 거부 규칙에 해당하지 않는 도구에 일어나는 일을 제어합니다. 사용 가능한 모드는 [권한 모드](#권한-모드)를 참조하세요.

`"Bash(npm *)"`와 같은 규칙으로 개별 도구의 범위를 지정하여 특정 명령만 허용할 수도 있습니다. 전체 규칙 구문은 [권한](/docs/en/agent-sdk/permissions)을 참조하세요.

도구가 거부되면 Claude는 도구 결과로 거부 메시지를 받고 일반적으로 다른 접근 방식을 시도하거나 진행할 수 없음을 보고합니다.

### 병렬 도구 실행

Claude가 단일 턴에서 여러 도구 호출을 요청하는 경우, 두 SDK 모두 도구에 따라 이를 동시 또는 순차적으로 실행할 수 있습니다. 읽기 전용 도구(`Read`, `Glob`, `Grep` 및 읽기 전용으로 표시된 MCP 도구)는 동시에 실행할 수 있습니다. 상태를 수정하는 도구(`Edit`, `Write`, `Bash`)는 충돌을 방지하기 위해 순차적으로 실행됩니다.

커스텀 도구는 기본적으로 순차 실행됩니다. 커스텀 도구에 대해 병렬 실행을 활성화하려면 주석에 `readOnlyHint`를 설정하세요. [TypeScript](/docs/en/agent-sdk/typescript#tool) 및 [Python](/docs/en/agent-sdk/python#tool) SDK 모두 MCP SDK의 이 필드 이름을 사용합니다.

## 루프 실행 방식 제어

루프가 수행하는 턴 수, 소비 비용, Claude의 추론 깊이, 실행 전 도구 승인 필요 여부를 제한할 수 있습니다. 이 모든 옵션은 [`ClaudeAgentOptions`](/docs/en/agent-sdk/python#claudeagentoptions) (Python) / [`Options`](/docs/en/agent-sdk/typescript#options) (TypeScript)의 필드로 제공됩니다.

### 턴과 예산

| 옵션 | 제어 항목 | 기본값 |
| :--------------------------------------------- | :--------------------------- | :------- |
| 최대 턴 (`max_turns` / `maxTurns`) | 최대 도구 사용 왕복 횟수 | 제한 없음 |
| 최대 예산 (`max_budget_usd` / `maxBudgetUsd`) | 중단 전 최대 비용 | 제한 없음 |

두 제한 중 하나에 도달하면 SDK는 해당 오류 하위 유형(`error_max_turns` 또는 `error_max_budget_usd`)이 포함된 `ResultMessage`를 반환합니다. 이러한 하위 유형을 확인하는 방법은 [결과 처리하기](#결과-처리하기)를 참조하고, 구문은 [`ClaudeAgentOptions`](/docs/en/agent-sdk/python#claudeagentoptions) / [`Options`](/docs/en/agent-sdk/typescript#options)를 참조하세요.

[스트리밍 입력](/docs/en/agent-sdk/streaming-vs-single-mode)을 사용할 때 턴이 아직 실행되는 동안 보낸 메시지는 해당 턴이 max-turns 제한으로 종료될 때 대기열에 대기하며 자체 max-turns 제한으로 새 턴을 시작합니다. v2.1.205 이전에는 턴의 마지막 반복에 도착한 메시지가 종료되는 턴에 소비되어 모델에 도달하지 않고 손실될 수 있었습니다.

### 노력(Effort) 수준

`effort` 옵션은 Claude가 적용하는 추론의 양을 제어합니다. 낮은 effort 수준은 턴당 더 적은 토큰을 사용하여 비용을 줄입니다. 모든 모델이 effort 파라미터를 지원하는 것은 아닙니다. 지원하는 모델은 [Effort](https://platform.claude.com/docs/en/build-with-claude/effort)를 참조하세요.

| 수준 | 동작 | 적합한 용도 |
| :--------- | :-------------------------------- | :------------------------------------------------------------------------ |
| `"low"` | 최소한의 추론, 빠른 응답 | 파일 조회, 디렉토리 나열 |
| `"medium"` | 균형 잡힌 추론 | 일상적인 편집, 표준 작업 |
| `"high"` | 철저한 분석 | 리팩토링, 디버깅 |
| `"xhigh"` | 확장된 추론 깊이 | 코딩 및 에이전트 작업; Fable 5, Opus 4.7+, Sonnet 5에 권장됨 |
| `"max"` | 최대 추론 깊이 | 깊은 분석이 필요한 다단계 문제 |

`effort`를 설정하지 않으면 두 SDK 모두 파라미터를 설정하지 않은 상태로 두고 모델의 기본 동작에 위임합니다.

<Note>
  `effort`는 각 응답 내에서 추론 깊이를 위해 지연 시간과 토큰 비용을 교환합니다. [확장 사고(Extended thinking)](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)는 출력에 생각의 과정(chain-of-thought) 블록을 노출하는 별개의 기능입니다. 이 둘은 독립적입니다: 확장 사고가 활성화된 상태에서 `effort: "low"`를 설정하거나, 확장 사고 없이 `effort: "max"`를 설정할 수 있습니다.
</Note>

비용과 지연 시간을 줄이려면 단순하고 범위가 명확한 작업(파일 나열이나 단일 grep 실행 등)을 수행하는 에이전트에 더 낮은 effort를 사용하세요. 전체 세션에 대한 최상위 `query()` 옵션에서 `effort`를 설정하거나, [`AgentDefinition`](/docs/en/agent-sdk/subagents#agentdefinition-configuration)의 `effort` 필드로 서브에이전트별로 설정하여 세션 수준을 재정의하세요.

### 권한 모드

권한 모드 옵션(Python의 `permission_mode`, TypeScript의 `permissionMode`)은 에이전트가 도구를 사용하기 전에 승인을 요청할지 여부를 제어합니다:

| 모드 | 동작 |
| :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"default"` | 허용 규칙에 해당하지 않는 도구는 승인 콜백을 트리거합니다. 콜백이 없으면 거부됩니다. |
| `"acceptEdits"` | 파일 편집 및 일반적인 파일 시스템 명령(`mkdir`, `touch`, `mv`, `cp` 등)을 자동 승인합니다. 기타 Bash 명령은 기본 규칙을 따릅니다. |
| `"plan"` | Claude가 소스 파일을 편집하지 않고 탐색 및 계획을 수행합니다. 파일 편집은 자동 승인되지 않으며 `canUseTool` 콜백을 통해 확인합니다. |
| `"dontAsk"` | 프롬프트를 표시하지 않습니다. [권한 설정](/docs/en/settings#permission-settings)에 의해 사전 허용된 도구는 실행되고, 그 외는 모두 거부됩니다. `AskUserQuestion`, [조직에서 `ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), [`requiresUserInteraction`으로 표시된 MCP 도구](/docs/en/mcp#require-approval-for-a-specific-tool)는 명시적으로 허용했더라도 거부됩니다. |
| `"auto"` | 모델 분류기를 사용하여 권한 프롬프트를 승인하거나 거부합니다. 사용 가능 여부 및 동작은 [Auto 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)를 참조하세요. |
| `"bypassPermissions"` | 명시적인 [`ask` 규칙](/docs/en/settings#permission-settings)에 일치하는 도구, [조직에서 `ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), 사용자 상호작용이 필요한 도구를 제외한 모든 허용된 도구를 확인 없이 실행합니다. 우선순위 순서는 [권한 평가 방식](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)을 참조하세요. Unix에서 root로 실행 중일 때는 사용할 수 없습니다. 에이전트의 작업이 중요한 시스템에 영향을 주지 않는 격리된 환경에서만 사용하세요. |

대화형 애플리케이션의 경우, 승인 프롬프트를 표출하려면 도구 승인 콜백과 함께 `"default"`를 사용하세요. 개발자 머신의 자율 에이전트의 경우, `"acceptEdits"`는 다른 `Bash` 명령을 허용 규칙 뒤에 제어하면서도 파일 편집 및 일반 파일 시스템 명령(`mkdir`, `touch`, `mv`, `cp` 등)을 자동 승인합니다. `"bypassPermissions"`는 CI, 컨테이너 또는 기타 격리된 환경용으로 보관하세요. 자세한 내용은 [권한](/docs/en/agent-sdk/permissions)을 참조하세요.

### 모델

`model`을 설정하지 않으면 SDK는 인증 방식 및 구독에 따라 달라지는 Claude Code의 기본값을 사용합니다. 명시적으로 지정하여(예: `model="claude-sonnet-5"`) 특정 모델을 고정하거나 더 작고 저렴한 에이전트를 위해 작은 모델을 사용할 수 있습니다. 사용 가능한 ID는 [모델](https://platform.claude.com/docs/en/about-claude/models)을 참조하세요.

## 컨텍스트 윈도우

컨텍스트 윈도우는 세션 동안 Claude가 이용할 수 있는 정보의 총량입니다. 세션 내의 턴 사이에는 리셋되지 않습니다. 시스템 프롬프트, 도구 정의, 대화 기록, 도구 입력, 도구 출력 등 모든 것이 누적됩니다. 턴 전반에 걸쳐 동일하게 유지되는 콘텐츠(시스템 프롬프트, 도구 정의, CLAUDE.md)는 자동으로 [프롬프트 캐싱](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)되어 반복되는 접두사에 대한 비용과 지연 시간을 줄여줍니다. 커스텀 시스템 프롬프트나 `append` 텍스트가 세션 간 캐시 재사용에 미치는 영향은 [시스템 프롬프트 수정](/docs/en/agent-sdk/modifying-system-prompts#improve-prompt-caching-across-users-and-machines)을 참조하세요.

### 컨텍스트 소모 요소

SDK에서 각 구성 요소가 컨텍스트에 미치는 영향은 다음과 같습니다:

| 소스 | 로드 시점 | 영향 |
| :----------------------- | :------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **시스템 프롬프트** | 모든 요청 | 소형 고정 비용, 항상 존재 |
| **CLAUDE.md 파일** | 세션 시작 시, [`settingSources`](/docs/en/agent-sdk/claude-code-features)를 통해 | 모든 요청에 전체 콘텐츠 포함(단, 프롬프트 캐싱되므로 첫 번째 요청만 전체 비용 지불) |
| **도구 정의** | 모든 요청; MCP 스키마는 기본적으로 지연됨 | 내장 도구 스키마는 모든 요청에 로드됨. [도구 검색](/docs/en/agent-sdk/mcp#mcp-tool-search)은 MCP 도구 스키마를 기본적으로 지연하며, Google Cloud's Agent Platform 또는 타사 `ANTHROPIC_BASE_URL`에서는 사전 로드로 폴백함. 매트릭스 전체는 [도구 검색 구성](/docs/en/agent-sdk/tool-search#configure-tool-search) 참조 |
| **대화 기록** | 턴에 따라 누적됨 | 각 턴마다 성장함: 프롬프트, 응답, 도구 입력, 도구 출력 |
| **스킬 설명** | 세션 시작 시, 설정 소스를 통해 | 짧은 요약; 호출될 때만 전체 콘텐츠 로드됨 |

대형 도구 출력은 상당한 컨텍스트를 소모합니다. 큰 파일을 읽거나 장황한 출력으로 명령을 실행하면 단일 턴에서 수천 개의 토큰이 사용될 수 있습니다. 컨텍스트는 턴 전체에 걸쳐 누적되므로 많은 도구 호출이 포함된 긴 세션은 짧은 세션보다 훨씬 더 많은 컨텍스트가 쌓입니다.

### 자동 압축

컨텍스트 윈도우가 한계에 도달하면 SDK는 자동으로 대화를 압축합니다. 즉, 최근 교환 내용과 주요 결정 사항은 그대로 유지하면서 이전 기록을 요약하여 공간을 확보합니다. 이 현상이 발생하면 SDK는 스트림에 `type: "system"` 및 `subtype: "compact_boundary"` 메시지를 방출합니다(Python에서는 `SystemMessage`, TypeScript에서는 별도의 `SDKCompactBoundaryMessage` 유형).

압축은 이전 메시지를 요약본으로 대체하므로 대화 초기의 특정 지침이 유지되지 않을 수 있습니다. 영구적인 규칙은 초기 프롬프트보다는 CLAUDE.md([`settingSources`](/docs/en/agent-sdk/claude-code-features)를 통해 로드됨)에 두어야 합니다. CLAUDE.md 콘텐츠는 모든 요청에 다시 주입되기 때문입니다.

여러 방법으로 압축 동작을 맞춤화할 수 있습니다:

* **CLAUDE.md의 요약 지침:** 압축기는 다른 컨텍스트와 마찬가지로 CLAUDE.md를 읽으므로 요약할 때 보존할 내용을 알려주는 섹션을 포함할 수 있습니다. 섹션 헤더는 자유 형식(매직 문자열 아님)이며 압축기는 의도에 따라 일치시킵니다.
* **`PreCompact` 후크:** 예를 들어 전체 트랜스크립트를 보관하기 위해 압축이 일어나기 전에 커스텀 로직을 실행합니다. 후크는 `trigger` 필드(`manual` 또는 `auto`)를 수신합니다. [후크](/docs/en/agent-sdk/hooks)를 참조하세요.
* **수동 압축:** 요청 시 압축을 트리거하려면 프롬프트 문자열로 `/compact`를 보내세요. 이런 방식으로 전송된 명령은 CLI 전용 단축키가 아니라 SDK 입력입니다. [SDK에서의 명령](/docs/en/agent-sdk/slash-commands)을 참조하세요.

<Accordion title="예시: CLAUDE.md의 요약 지침">
  프로젝트의 CLAUDE.md에 압축기에게 보존할 내용을 알려주는 섹션을 추가하세요. 헤더 이름은 특별하지 않으므로 명확한 레이블을 사용하면 됩니다.

  ```markdown CLAUDE.md theme={null}
  # Summary instructions

  When summarizing this conversation, always preserve:
  - The current task objective and acceptance criteria
  - File paths that have been read or modified
  - Test results and error messages
  - Decisions made and the reasoning behind them
  ```
</Accordion>

### 컨텍스트 효율성 유지하기

장시간 실행되는 에이전트를 위한 몇 가지 전략:

* **하위 작업에 서브에이전트 사용.** 각 서브에이전트는 새로운 대화(이전 메시지 기록 없음. 단 자체 시스템 프롬프트 및 CLAUDE.md와 같은 프로젝트 수준 컨텍스트는 로드함)로 시작합니다. 상위 에이전트의 턴을 보지 못하며, 최종 응답만 도구 결과로 상위 에이전트에 반환됩니다. 상위 에이전트의 컨텍스트는 전체 하위 작업 트랜스크립트가 아닌 해당 요약본만큼만 늘어납니다. 자세한 내용은 [서브에이전트가 상속하는 항목](/docs/en/agent-sdk/subagents#what-subagents-inherit)을 참조하세요.
* **도구를 신중하게 선택.** 모든 도구 정의는 컨텍스트 공간을 차지합니다. [`AgentDefinition`](/docs/en/agent-sdk/subagents#agentdefinition-configuration)의 `tools` 필드를 사용하여 서브에이전트의 범위를 필요한 최소한의 세트로 지정하세요.
* **MCP 서버 비용 관찰.** [MCP 도구 검색](/docs/en/agent-sdk/mcp#mcp-tool-search)은 기본적으로 MCP 도구 스키마를 지연시키고 요청 시 로드합니다. 도구 검색이 꺼져 있거나, Google Cloud's Agent Platform을 사용하거나, 타사 `ANTHROPIC_BASE_URL`을 사용하는 경우, 각 MCP 서버는 모든 도구 스키마를 모든 요청에 추가하므로 많은 도구를 가진 몇몇 서버가 에이전트가 작업을 수행하기도 전에 상당한 컨텍스트를 소모할 수 있습니다.
* **일상적인 작업에 낮은 effort 사용.** 파일 읽기나 디렉토리 나열만 하면 되는 에이전트는 [effort](#노력effort-수준)를 `"low"`로 설정하세요. 토큰 사용량과 비용이 감소합니다.

기능별 컨텍스트 비용의 상세 분석은 [컨텍스트 비용 이해하기](/docs/en/features-overview#understand-context-costs)를 참조하세요.

## 세션과 연속성

SDK와의 각 상호작용은 세션을 생성하거나 계속합니다. 나중에 다시 시작하려면 `ResultMessage.session_id`(두 SDK 모두에서 이용 가능)에서 세션 ID를 캡처하세요. TypeScript SDK는 이를 init `SystemMessage`에 직접 필드로 노출하며, Python에서는 `SystemMessage.data`에 중첩되어 있습니다.

다시 시작하면 이전 턴의 전체 컨텍스트(읽은 파일, 수행된 분석, 취해진 조치)가 복원됩니다. 원본을 수정하지 않고 다른 접근 방식으로 가지를 치기(branch) 위해 세션을 포크(fork)할 수도 있습니다.

다시 시작(resume), 계속(continue), 포크(fork) 패턴에 대한 전체 가이드는 [세션 관리](/docs/en/agent-sdk/sessions)를 참조하세요. 무상태 컨테이너나 서버리스 호스트에서 세션을 다시 시작하려면 [`session_store` / `sessionStore` 어댑터](/docs/en/agent-sdk/session-storage)를 전달하여 트랜스크립트가 자체 백엔드에 미러링되어 어떤 호스트든 다시 시작할 수 있도록 하세요. Claude Code 서브프로세스는 여전히 로컬 디스크에 먼저 씁니다. 로컬 복사본이 단명(ephemeral)해야 하는 경우 `options.env`에서 `CLAUDE_CONFIG_DIR`을 임시 디렉토리로 지정하세요.

<Note>
  Python에서 `ClaudeSDKClient`는 여러 호출에 걸쳐 세션 ID를 자동으로 처리합니다. 자세한 내용은 [Python SDK 참조](/docs/en/agent-sdk/python#choosing-between-query-and-claudesdkclient)를 참조하세요.
</Note>

## 결과 처리하기

루프가 끝나면 `ResultMessage`가 발생한 일을 알려주고 출력을 제공합니다. `subtype` 필드(두 SDK 모두에서 제공)는 종료 상태를 확인하는 주요 방법입니다.

| 결과 하위 유형 | 발생 내용 | `result` 필드 이용 가능 여부 |
| :------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-----------------------: |
| `success` | Claude가 작업을 정상적으로 완료함 | 예 |
| `error_max_turns` | 완료 전에 `maxTurns` 제한에 도달함 | 아니오 |
| `error_max_budget_usd` | 완료 전에 `maxBudgetUsd` 제한에 도달함 | 아니오 |
| `error_during_execution` | 오류로 인해 루프가 중단됨(예: API 실패 또는 취소된 요청) | 아니오 |
| `error_max_structured_output_retries` | 구성된 재시도 제한 내에 유효한 구조화된 출력이 생성되지 않음: 모든 시도가 검증에 실패했거나 모델 폴백이 재시도 성공 없이 완료된 출력을 철회함 | 아니오 |

`result` 필드(최종 텍스트 출력)는 `success` 변형에만 존재하므로 읽기 전에 항상 하위 유형을 확인하세요. 모든 결과 하위 유형에는 `total_cost_usd`, `usage`, `num_turns`, `session_id`가 포함되어 있어 비용을 추적하고 오류 후에도 다시 시작할 수 있습니다. Python에서 `total_cost_usd` 및 `usage`는 선택 사항으로 입력되어 일부 오류 경로에서 `None`일 수 있으므로 형식 지정 전에 확인하세요. `usage` 필드 해석에 대한 자세한 내용은 [비용 및 사용량 추적](/docs/en/agent-sdk/cost-tracking)을 참조하세요.

<Note>
  쿼리가 오류 결과로 끝날 때:

  * 단발성 `query()` 호출은 최종 결과 메시지를 생성한 다음 `Reached maximum number of turns`와 같은 실패 텍스트가 포함된 오류를 발생시킵니다. 이 예외 발생은 의도된 것입니다. 코드가 이 이후에도 계속 실행되어야 하는 경우 루프를 try 블록으로 감싸세요. 기본 Claude Code 프로세스도 0이 아닌 코드로 종료됩니다.
  * 스트리밍 입력 세션은 활성 상태를 유지하며 메시지를 계속 보낼 수 있습니다.
</Note>

결과에는 모델이 최종 턴에서 생성을 중단한 이유를 나타내는 `stop_reason` 필드(TypeScript에서는 `string | null`, Python에서는 `str | None`)도 포함됩니다. 일반적인 값은 `end_turn`(모델이 정상 완료됨), `max_tokens`(출력 토큰 제한 도달), `refusal`(모델이 요청 거부함)입니다. 오류 결과 하위 유형에서 `stop_reason`은 루프가 끝나기 전 마지막 어시스턴트 응답의 값을 전달합니다. 거부를 감지하려면 `stop_reason === "refusal"`(TypeScript) 또는 `stop_reason == "refusal"`(Python)을 확인하세요. 전체 유형은 [`SDKResultMessage`](/docs/en/agent-sdk/typescript#sdkresultmessage) (TypeScript) 또는 [`ResultMessage`](/docs/en/agent-sdk/python#resultmessage) (Python)를 참조하세요.

## 후크(Hooks)

[후크](/docs/en/agent-sdk/hooks)는 루프의 특정 지점(도구 실행 전, 도구 반환 후, 에이전트 종료 시 등)에서 실행되는 콜백입니다. 흔히 사용되는 후크는 다음과 같습니다:

| 후크 | 실행 시점 | 일반적인 용도 |
| :------------------------------- | :---------------------------------- | :----------------------------------------- |
| `PreToolUse` | 도구 실행 전 | 입력 검증, 위험한 명령 차단 |
| `PostToolUse` | 도구 반환 후 | 출력 감사, 사이드 이펙트 트리거 |
| `UserPromptSubmit` | 프롬프트 전송 시 | 프롬프트에 추가 컨텍스트 주입 |
| `Stop` | 에이전트 종료 시 | 결과 검증, 세션 상태 저장 |
| `SubagentStart` / `SubagentStop` | 서브에이전트 생성 또는 완료 시 | 병렬 작업 결과 추적 및 집계 |
| `PreCompact` | 컨텍스트 압축 전 | 요약 전 전체 트랜스크립트 보관 |

후크는 에이전트의 컨텍스트 윈도우 내부가 아니라 애플리케이션 프로세스에서 실행되므로 컨텍스트를 소모하지 않습니다. 후크는 루프를 단축(short-circuit)할 수도 있습니다. 도구 호출을 거부하는 `PreToolUse` 후크는 실행을 방지하고 Claude는 그 대신 거부 메시지를 받습니다.

두 SDK 모두 위의 모든 이벤트를 지원합니다. TypeScript SDK에는 Python이 아직 지원하지 않는 추가 이벤트가 포함되어 있습니다. 전체 이벤트 목록, SDK별 사용 가능 여부, 전체 콜백 API는 [후크로 실행 제어하기](/docs/en/agent-sdk/hooks)를 참조하세요.

## 종합 예제

이 예제는 이 페이지의 핵심 개념을 결합하여 실패한 테스트를 수정하는 단일 에이전트를 구성합니다. 에이전트에 허용된 도구(자동 승인되어 에이전트가 자율적으로 실행됨), 프로젝트 설정, 턴 및 추론 노력에 대한 안전 제한을 구성합니다. 루프가 실행됨에 따라 재시작 가능성을 위해 세션 ID를 캡처하고 최종 결과를 처리하며 총 비용을 출력합니다.

단발성 `query()` 호출은 오류 결과를 생성한 후 예외를 발생시키므로 제한 도달 시 스크립트가 깔끔하게 종료되도록 루프를 try 블록으로 감쌉니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def run_agent():
      session_id = None

      try:
          async for message in query(
              prompt="Find and fix the bug causing test failures in the auth module",
              options=ClaudeAgentOptions(
                  allowed_tools=[
                      "Read",
                      "Edit",
                      "Bash",
                      "Glob",
                      "Grep",
                  ],  # 여기에 나열된 도구는 자동 승인됨 (프롬프트 없음)
                  setting_sources=[
                      "project"
                  ],  # 현재 디렉토리에서 CLAUDE.md, 스킬, 후크 로드
                  max_turns=30,  # 제어 불능 세션 방지
                  effort="high",  # 복잡한 디버깅을 위한 철저한 추론
              ),
          ):
              # 최종 결과 처리
              if isinstance(message, ResultMessage):
                  session_id = message.session_id  # 재시작 가능성을 위해 저장

                  if message.subtype == "success":
                      print(f"Done: {message.result}")
                  elif message.subtype == "error_max_turns":
                      # 에이전트 턴 소진. 더 높은 제한으로 다시 시작.
                      print(f"Hit turn limit. Resume session {session_id} to continue.")
                  elif message.subtype == "error_max_budget_usd":
                      print("Hit budget limit.")
                  else:
                      print(f"Stopped: {message.subtype}")
                  if message.total_cost_usd is not None:
                      print(f"Cost: ${message.total_cost_usd:.4f}")
      except Exception as error:
          # 단발성 query()는 오류 결과를 생성한 후 예외를 발생시킵니다.
          # 오류 결과로 인한 실패였다면 위의 오류 분기가 이미 실행된 상태입니다.
          # 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
          print(f"Session ended with an error: {error}")


  asyncio.run(run_agent())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  let sessionId: string | undefined;

  try {
    for await (const message of query({
      prompt: "Find and fix the bug causing test failures in the auth module",
      options: {
        allowedTools: ["Read", "Edit", "Bash", "Glob", "Grep"], // 여기에 나열된 도구는 자동 승인됨 (프롬프트 없음)
        settingSources: ["project"], // 현재 디렉토리에서 CLAUDE.md, 스킬, 후크 로드
        maxTurns: 30, // 제어 불능 세션 방지
        effort: "high" // 복잡한 디버깅을 위한 철저한 추론
      }
    })) {
      // 나중에 다시 시작할 수 있도록 세션 ID 저장
      if (message.type === "system" && message.subtype === "init") {
        sessionId = message.session_id;
      }

      // 최종 결과 처리
      if (message.type === "result") {
        if (message.subtype === "success") {
          console.log(`Done: ${message.result}`);
        } else if (message.subtype === "error_max_turns") {
          // 에이전트 턴 소진. 더 높은 제한으로 다시 시작.
          console.log(`Hit turn limit. Resume session ${sessionId} to continue.`);
        } else if (message.subtype === "error_max_budget_usd") {
          console.log("Hit budget limit.");
        } else {
          console.log(`Stopped: ${message.subtype}`);
        }
        console.log(`Cost: $${message.total_cost_usd.toFixed(4)}`);
      }
    }
  } catch (error) {
    // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
    // 오류 결과로 인한 실패였다면 위의 오류 분기가 이미 실행된 상태입니다.
    // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
    console.log(`Session ended with an error: ${error}`);
  }
  ```
</CodeGroup>

## 다음 단계

이제 루프를 이해했으므로 구축하려는 항목에 따라 이동할 위치는 다음과 같습니다:

* **아직 에이전트를 실행해보지 않았나요?** [빠른 시작](/docs/en/agent-sdk/quickstart)으로 시작하여 SDK를 설치하고 전체 작동 예제를 확인하세요.
* **프로젝트에 연동할 준비가 되었나요?** [CLAUDE.md, 스킬 및 파일 시스템 후크 로드](/docs/en/agent-sdk/claude-code-features)를 통해 에이전트가 프로젝트 규칙을 자동으로 따르도록 하세요.
* **대화형 UI를 구축 중인가요?** [스트리밍](/docs/en/agent-sdk/streaming-output)을 활성화하여 루프가 실행될 때 실시간 텍스트 및 도구 호출을 표시하세요.
* **에이전트가 할 수 있는 일에 대해 더 엄격한 제어가 필요한가요?** [권한](/docs/en/agent-sdk/permissions)으로 도구 접근을 잠그고, [후크](/docs/en/agent-sdk/hooks)를 사용하여 도구가 실행되기 전에 감사, 차단 또는 변환하세요.
* **길거나 비용이 많이 드는 작업을 실행하나요?** 주 컨텍스트를 슬림하게 유지하기 위해 격리된 작업을 [서브에이전트](/docs/en/agent-sdk/subagents)에 위임하세요.
* **서비스로 배포하나요?** 컨테이너 및 서버리스 가이드는 [Agent SDK 호스팅](/docs/en/agent-sdk/hosting)을, 세션을 자체 백엔드에 유지하려면 [세션 저장소](/docs/en/agent-sdk/session-storage)를 참조하세요.

SDK에 국한되지 않은 에이전트 루프의 광범위한 개념적 그림은 [Claude Code 작동 방식](/docs/en/how-claude-code-works)을 참조하세요. 턴 기반부터 목표 기반 및 주도적 루프까지 Claude Code에서 루프를 설계하는 실용적인 가이드는 블로그의 [루프 엔지니어링: 루프 시작하기](https://claude.com/blog/getting-started-with-loops)를 참조하세요.
