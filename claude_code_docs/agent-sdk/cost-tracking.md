> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 비용 및 사용량 추적하기

> Claude Agent SDK를 사용하여 토큰 사용량을 추적하고, 비용을 추정하며, 프롬프트 캐싱을 구성하는 방법을 알아봅니다.

Claude Agent SDK는 Claude와의 각 상호작용에 대해 상세한 토큰 사용량 정보를 제공합니다. 이 가이드에서는 특히 병렬 도구 사용 및 다단계 대화를 처리할 때 사용량을 올바르게 추적하고 비용 보고를 이해하는 방법을 설명합니다.

전체 API 문서는 [TypeScript SDK 참조](/docs/en/agent-sdk/typescript) 및 [Python SDK 참조](/docs/en/agent-sdk/python)를 참조하세요.

<Warning>
  `total_cost_usd` 및 `costUSD` 필드는 확정적인 청구 데이터가 아니라 클라이언트 측 추정치입니다. SDK는 빌드 시 번들로 제공된 가격표를 기반으로 로컬에서 이를 계산하므로 다음과 같은 경우 실제 청구 금액과 차이가 날 수 있습니다:

  * 가격이 변경되는 경우
  * 설치된 SDK 버전이 모델을 인식하지 못하는 경우
  * 클라이언트가 모델링할 수 없는 청구 규칙이 적용되는 경우

  이 필드는 개발 분석 및 대략적인 예산 산정에 사용하세요. 확정적인 청구 내역은 [Usage and Cost API](https://platform.claude.com/docs/en/build-with-claude/usage-cost-api) 또는 [Claude Console](https://platform.claude.com/usage)의 Usage 페이지를 참조하세요. 이 필드를 기준으로 최종 사용자에게 요금을 청구하거나 재무적 결정을 내리지 마세요.
</Warning>

## 토큰 사용량 이해하기

TypeScript 및 Python SDK는 서로 다른 필드 이름으로 동일한 사용량 데이터를 노출합니다:

* **TypeScript**는 각 어시스턴트 메시지(`message.message.id`, `message.message.usage`)에서 단계별(per-step) 토큰 세부 내역을, 결과 메시지의 `modelUsage`를 통해 모델별 비용을, 결과 메시지에서 누적 합계를 제공합니다.
* **Python**은 각 어시스턴트 메시지(`message.usage`, `message.message_id`)에서 단계별 토큰 세부 내역을, 결과 메시지의 `model_usage`를 통해 모델별 비용을, 결과 메시지에서 누적 합계(`total_cost_usd` 및 `usage` 딕셔너리)를 제공합니다.

두 SDK 모두 동일한 기본 비용 모델을 사용하고 동일한 세분성을 노출합니다. 차이점은 필드 명명 규칙과 단계별 사용량이 중첩되는 위치입니다.

비용 추적은 SDK가 사용량 데이터의 범위를 지정하는 방식을 이해하는 데 달려 있습니다:

* **`query()` 호출:** SDK의 `query()` 함수를 1회 호출하는 것입니다. 단일 호출에는 여러 단계(Claude가 응답하고, 도구를 사용하고, 결과를 얻고, 다시 응답함)가 포함될 수 있습니다. 각 호출은 끝에 1개의 [`result`](/docs/en/agent-sdk/typescript#sdkresultmessage) 메시지를 생성합니다.
* **단계(Step):** `query()` 호출 내의 단일 요청/응답 주기입니다. 각 단계는 토큰 사용량이 포함된 어시스턴트 메시지를 생성합니다.
* **세션:** 세션 ID로 연결된 일련의 `query()` 호출(`resume` 옵션 사용)입니다. 세션 내의 각 `query()` 호출은 독립적으로 자체 비용을 보고합니다.

다음 다이어그램은 단일 `query()` 호출의 메시지 스트림을 보여주며, 각 단계에서 보고되는 토큰 사용량과 끝에서의 누적 추정치를 나타냅니다:

<img src="https://mintcdn.com/claude-code/ikqp3_70mqIahteV/images/agent-sdk/message-usage-flow.svg?fit=max&auto=format&n=ikqp3_70mqIahteV&q=85&s=68497aee338e01cc745323af7aea378e" alt="Diagram showing a query producing two steps of messages. Step 1 has four assistant messages sharing the same ID and usage (count once), Step 2 has one assistant message with a new ID, and the final result message shows the estimated total_cost_usd." width="760" height="520" data-path="images/agent-sdk/message-usage-flow.svg" />

<Steps>
  <Step title="각 단계는 어시스턴트 메시지를 생성합니다">
    Claude가 응답할 때 하나 이상의 어시스턴트 메시지를 보냅니다. TypeScript에서 각 어시스턴트 메시지는 `id` 및 토큰 수(`input_tokens`, `output_tokens`)가 포함된 [`usage`](https://platform.claude.com/docs/en/api/messages) 객체를 가지는 중첩된 `BetaMessage`(`message.message`를 통해 접근)를 포함합니다. Python에서 `AssistantMessage` 데이터클래스는 `message.usage` 및 `message.message_id`를 통해 동일한 데이터를 직접 노출합니다. Claude가 한 턴에서 여러 도구를 사용할 때 해당 턴의 모든 메시지는 동일한 ID를 공유하므로 이중 계산을 방지하기 위해 ID별로 중복을 제거하세요.
  </Step>

  <Step title="결과 메시지는 누적 추정치를 제공합니다">
    `query()` 호출이 완료되면 SDK는 `total_cost_usd` 및 누적 `usage`가 포함된 결과 메시지를 방출합니다. 이는 TypeScript([`SDKResultMessage`](/docs/en/agent-sdk/typescript#sdkresultmessage))와 Python([`ResultMessage`](/docs/en/agent-sdk/python#resultmessage)) 모두에서 이용할 수 있습니다. 여러 `query()` 호출을 수행하는 경우(예: 다중 턴 세션) 각 결과는 해당 개별 호출의 비용만 반영합니다. 추정 총액만 필요한 경우 단계별 사용량을 무시하고 이 단일 값을 읽으면 됩니다.
  </Step>
</Steps>

## 쿼리의 총 비용 구하기

결과 메시지([TypeScript](/docs/en/agent-sdk/typescript#sdkresultmessage), [Python](/docs/en/agent-sdk/python#resultmessage))는 `query()` 호출에 대한 에이전트 루프의 끝을 표시합니다. 여기에는 해당 호출의 모든 단계에 대한 누적 추정 비용인 `total_cost_usd`가 포함되어 있습니다. 이는 성공 결과와 오류 결과 모두에서 작동합니다. 세션을 사용하여 여러 `query()` 호출을 수행하는 경우 각 결과는 해당 개별 호출의 비용만 반영합니다.

3가지 결과 수준 필드는 에이전트가 [서브에이전트](/docs/en/agent-sdk/subagents)를 생성할 때 카운트하는 내용이 다릅니다. 전체 트리의 토큰 정산을 위해 Python의 `modelUsage` 또는 `model_usage`를 사용하세요. `usage` 필드는 중첩이 발생하는 즉시 실제보다 적게 집계됩니다.

| 필드 | 서브에이전트 활동 |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| `usage` | 제외됨. 최상위 에이전트 루프만 카운트하므로 서브에이전트 내부에서 소모된 토큰은 추가되지 않음 |
| `total_cost_usd` | 포함됨. 최상위 루프와 함께 서브에이전트 요청을 함께 카운트함 |
| `modelUsage` / `model_usage` | 포함됨. 최상위 루프와 함께 서브에이전트 요청을 모델별로 구분하여 함께 카운트함 |

다음 예제는 `query()` 호출의 메시지 스트림을 반복하고 `result` 메시지가 도착할 때 총 비용을 출력합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  try {
    for await (const message of query({ prompt: "Summarize this project" })) {
      if (message.type === "result") {
        console.log(`Total cost: $${message.total_cost_usd}`);
      }
    }
  } catch (error) {
    // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
    // 오류 결과였더라도 total_cost_usd를 전달받았으며 위의 분기가 이미 실행된 상태입니다.
    // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
    console.error(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ResultMessage
  import asyncio


  async def main():
      try:
          async for message in query(prompt="Summarize this project"):
              if isinstance(message, ResultMessage):
                  print(f"Total cost: ${message.total_cost_usd or 0}")
      except Exception as error:
          # 단발성 query()는 오류 결과를 생성한 후 예외를 발생시킵니다.
          # 오류 결과였더라도 total_cost_usd를 전달받았으며 위의 분기가 이미 실행된 상태입니다.
          # 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

## 단계별 및 모델별 사용량 추적하기

이 섹션의 예제는 TypeScript 필드 이름을 사용합니다. Python의 동등한 필드는 단계별 사용량의 경우 [`AssistantMessage.usage`](/docs/en/agent-sdk/python#assistantmessage) 및 `AssistantMessage.message_id`이며, 모델별 세부 내역의 경우 [`ResultMessage.model_usage`](/docs/en/agent-sdk/python#resultmessage)입니다.

### 단계별 사용량 추적하기

각 어시스턴트 메시지에는 토큰 수치가 포함된 `id` 및 `usage` 객체를 가지는 중첩된 `BetaMessage`(`message.message`를 통해 접근)가 포함되어 있습니다. Claude가 도구를 병렬로 사용할 때 여러 메시지가 동일한 `id`와 동일한 사용량 데이터를 공유합니다. 이중 계상을 방지하기 위해 이미 카운트한 ID를 추적하고 중복을 건너뛰세요.

<Warning>
  병렬 도구 호출은 중첩된 `BetaMessage`가 동일한 `id` 및 동일한 사용량을 공유하는 여러 어시스턴트 메시지를 생성합니다. 정확한 단계별 토큰 수치를 얻으려면 항상 ID별로 중복을 제거하세요.
</Warning>

다음 예제는 모든 단계에서 입력 및 출력 토큰을 누적하며 각 고유 메시지 ID를 한 번만 카운트합니다:

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

const seenIds = new Set<string>();
let totalInputTokens = 0;
let totalOutputTokens = 0;

try {
  for await (const message of query({ prompt: "Summarize this project" })) {
    if (message.type === "assistant") {
      const msgId = message.message.id;

      // 병렬 도구 호출은 동일한 ID를 공유하므로 한 번만 카운트함
      if (!seenIds.has(msgId)) {
        seenIds.add(msgId);
        totalInputTokens += message.message.usage.input_tokens;
        totalOutputTokens += message.message.usage.output_tokens;
      }
    }
  }
} catch (error) {
  // 단발성 query()는 오류 결과를 생성한 후 예외를 던지므로
  // 아래 총합은 실패 전까지 실행된 단계를 반영합니다.
  console.error(`Session ended with an error: ${error}`);
}

console.log(`Steps: ${seenIds.size}`);
console.log(`Input tokens: ${totalInputTokens}`);
console.log(`Output tokens: ${totalOutputTokens}`);
```

### 모델별 사용량 세부 분해

결과 메시지에는 모델 이름별 토큰 수 및 비용 매핑인 [`modelUsage`](/docs/en/agent-sdk/typescript#modelusage)가 포함됩니다. 이는 여러 모델을 실행할 때(예: 서브에이전트용 Haiku 및 주 에이전트용 Opus) 토큰이 소모되는 위치를 확인하는 데 유용합니다.

다음 예제는 쿼리를 실행하고 사용된 각 모델의 비용 및 토큰 세부 내역을 출력합니다:

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({ prompt: "Summarize this project" })) {
    if (message.type !== "result") continue;

    for (const [modelName, usage] of Object.entries(message.modelUsage)) {
      console.log(`${modelName}: $${usage.costUSD.toFixed(4)}`);
      console.log(`  Input tokens: ${usage.inputTokens}`);
      console.log(`  Output tokens: ${usage.outputTokens}`);
      console.log(`  Cache read: ${usage.cacheReadInputTokens}`);
      console.log(`  Cache creation: ${usage.cacheCreationInputTokens}`);
    }
  }
} catch (error) {
  // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
  // 오류 결과였다면 위의 모델별 세부 내역이 이미 출력된 상태입니다.
  // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
  console.error(`Session ended with an error: ${error}`);
}
```

## 여러 호출 간 비용 누적하기

각 `query()` 호출은 자체 `total_cost_usd`를 반환합니다. SDK는 세션 수준의 총합을 제공하지 않으므로 애플리케이션이 여러 `query()` 호출을 수행하는 경우(예: 다중 턴 세션 또는 여러 사용자 간) 직접 합계를 누적하세요.

다음 예제는 두 번의 `query()` 호출을 순차적으로 실행하고 각 호출의 `total_cost_usd`를 누적 합계에 더하여 호출별 비용과 합산 비용을 모두 출력합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 여러 query() 호출 간 누적 비용 추적
  let totalSpend = 0;

  const prompts = [
    "Read the files in src/ and summarize the architecture",
    "List all exported functions in src/auth.ts"
  ];

  for (const prompt of prompts) {
    try {
      for await (const message of query({ prompt })) {
        if (message.type === "result") {
          totalSpend += message.total_cost_usd;
          console.log(`This call: $${message.total_cost_usd}`);
        }
      }
    } catch (error) {
      // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
      // 오류 결과였다면 이 호출의 비용은 이미 카운트된 상태입니다.
      // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
      // 다음 프롬프트로 진행합니다.
      console.error(`Call failed: ${error}`);
    }
  }

  console.log(`Total spend: $${totalSpend.toFixed(4)}`);
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ResultMessage
  import asyncio


  async def main():
      # 여러 query() 호출 간 누적 비용 추적
      total_spend = 0.0

      prompts = [
          "Read the files in src/ and summarize the architecture",
          "List all exported functions in src/auth.ts",
      ]

      for prompt in prompts:
          try:
              async for message in query(prompt=prompt):
                  if isinstance(message, ResultMessage):
                      cost = message.total_cost_usd or 0
                      total_spend += cost
                      print(f"This call: ${cost}")
          except Exception as error:
              # 단발성 query()는 오류 결과를 생성한 후 예외를 발생시킵니다.
              # 오류 결과였다면 이 호출의 비용은 이미 카운트된 상태입니다.
              # 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
              # 다음 프롬프트로 진행합니다.
              print(f"Call failed: {error}")

      print(f"Total spend: ${total_spend:.4f}")


  asyncio.run(main())
  ```
</CodeGroup>

## 오류, 캐싱 및 토큰 불일치 처리하기

정확한 비용 추적을 위해 실패한 대화, 캐시 토큰 요율, 간혹 발생하는 보고 불일치를 고려하세요.

### 출력 토큰 불일치 해결

드문 경우지만 동일한 ID를 가진 메시지에 대해 서로 다른 `output_tokens` 값이 관찰될 수 있습니다. 이 경우:

1. **가장 높은 값 사용:** 그룹의 마지막 메시지에 일반적으로 정확한 합계가 포함됩니다.
2. **결과 메시지 선호:** 결과 메시지의 `total_cost_usd`는 모든 단계에 걸친 SDK의 누적 추정치를 반영하므로 직접 단계별 값을 합산하는 것보다 신뢰할 수 있습니다. 다만 이 역시 추정치이므로 실제 청구서와 다를 수 있습니다.
3. **불일치 보고:** [Claude Code GitHub 리포지토리](https://github.com/anthropics/claude-code/issues)에 이슈를 제출하세요.

### 실패한 대화의 비용 추적

성공 및 오류 결과 메시지 모두 `usage` 및 `total_cost_usd`를 포함합니다. 대화가 중간에 실패하더라도 실패 시점까지 토큰이 소모되었습니다. `subtype`과 상관없이 항상 결과 메시지에서 비용 데이터를 읽으세요.

### 캐시 토큰 추적

Agent SDK는 반복되는 콘텐츠에 대한 비용을 줄이기 위해 [프롬프트 캐싱](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)을 자동으로 사용합니다. 캐싱을 직접 구성할 필요는 없습니다. usage 객체에는 캐시 추적을 위한 두 가지 추가 필드가 포함됩니다:

* `cache_creation_input_tokens`: 새 캐시 항목을 생성하는 데 사용된 토큰(표준 입력 토큰보다 높은 요율 적용).
* `cache_read_input_tokens`: 기존 캐시 항목에서 읽은 토큰(할인된 요율 적용).

캐싱 절감 효과를 이해하려면 이들을 `input_tokens`와 별도로 추적하세요. TypeScript에서 이 필드들은 [`Usage`](/docs/en/agent-sdk/typescript#usage) 객체에 타입이 지정되어 있습니다. Python에서는 [`ResultMessage.usage`](/docs/en/agent-sdk/python#resultmessage) 딕셔너리의 키로 나타납니다(예: `message.usage.get("cache_read_input_tokens", 0)`).

### 프롬프트 캐시 TTL을 1시간으로 연장

API 키로 인증하거나 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry에서 실행할 때 SDK가 작성한 캐시 항목은 기본적으로 5분의 TTL을 사용합니다. 동일한 시스템 프롬프트 및 컨텍스트에 대해 간격이 5분 이상 벌어지는 짧은 세션을 많이 실행하는 경우 세션 간에 캐시가 만료되어 각 새 세션이 전체 입력 가격을 지불합니다.

캐시 쓰기에 대해 1시간의 TTL을 요청하려면 [`ENABLE_PROMPT_CACHING_1H`](/docs/en/env-vars) 환경 변수를 설정하세요. 쉘 또는 컨테이너 환경에서 내보내거나 `options.env`를 통해 전달할 수 있습니다.

다음 예제는 Amazon Bedrock에서 실행 중인 에이전트에 대해 1시간 TTL을 활성화합니다. `CLAUDE_CODE_USE_BEDROCK`을 설정하므로 [Amazon Bedrock](/docs/en/amazon-bedrock)을 위한 올바른 AWS 자격 증명이 필요하며 자격 증명이 없으면 쿼리가 실패합니다.

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import ClaudeAgentOptions, query
  import asyncio


  async def main():
      options = ClaudeAgentOptions(
          env={
              "CLAUDE_CODE_USE_BEDROCK": "1",
              "ENABLE_PROMPT_CACHING_1H": "1",
          },
      )

      async for message in query(prompt="Summarize this project", options=options):
          print(message)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const options = {
    env: {
      ...process.env,
      CLAUDE_CODE_USE_BEDROCK: "1",
      ENABLE_PROMPT_CACHING_1H: "1",
    },
  };

  for await (const message of query({ prompt: "Summarize this project", options })) {
    console.log(message);
  }
  ```
</CodeGroup>

1시간 TTL을 가진 캐시 쓰기는 5분 쓰기보다 더 높은 요율로 청구되므로, 이를 활성화하면 쓰기 비용을 높이는 대신 더 많은 캐시 읽기 혜택을 얻게 됩니다. 자세한 내용은 [프롬프트 캐싱 요금](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)을 참조하세요. Claude 구독 사용자는 이미 1시간 TTL을 자동으로 받으므로 이 변수를 설정할 필요가 없습니다.

## 관련 문서

* [TypeScript SDK 참조](/docs/en/agent-sdk/typescript) - 전체 API 문서
* [SDK 개요](/docs/en/agent-sdk/overview) - SDK 시작하기
* [SDK 권한](/docs/en/agent-sdk/permissions) - 도구 권한 관리
