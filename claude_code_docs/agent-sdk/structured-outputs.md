> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 에이전트로부터 구조화된 출력 가져오기

> JSON Schema, Zod 또는 Pydantic을 사용하여 에이전트 워크플로우에서 검증된 JSON을 반환합니다. 다중 대화 도구 사용 후 타입 안정성이 보장된 구조화된 데이터를 가져옵니다.

구조화된 출력(Structured outputs)을 사용하면 에이전트로부터 돌려받고자 하는 데이터의 정확한 형상을 정의할 수 있습니다. 에이전트는 작업을 완료하는 데 필요한 모든 도구를 사용할 수 있으며, 사용자는 마지막에 스키마와 일치하는 검증된 JSON을 받게 됩니다. 필요한 구조에 대해 [JSON Schema](https://json-schema.org/understanding-json-schema/about)를 정의하면 SDK가 출력을 이에 맞춰 검증하고 불일치 시 재프롬프트합니다. 재시도 제한 내에 검증이 성공하지 못하면 구조화된 데이터 대신 오류 결과가 반환됩니다. [오류 처리](#error-handling)를 참조하세요.

완전한 타입 안정성을 위해 [Zod](#type-safe-schemas-with-zod-and-pydantic) (TypeScript) 또는 [Pydantic](#type-safe-schemas-with-zod-and-pydantic) (Python)을 사용하여 스키마를 정의하고 강력한 타입의 객체를 반환받으세요.

## 구조화된 출력이 필요한 이유

에이전트는 기본적으로 자유 형식 텍스트를 반환하므로 채팅에는 유용하지만 출력을 프로그래밍 방식으로 사용해야 할 때는 적합하지 않습니다. 구조화된 출력은 애플리케이션 로직, 데이터베이스 또는 UI 구성 요소에 직접 전달할 수 있는 타입이 지정된 데이터를 제공합니다.

에이전트가 웹을 검색하여 레시피를 가져오는 레시피 앱을 생각해 보세요. 구조화된 출력이 없다면 직접 파싱해야 하는 자유 형식 텍스트를 받게 됩니다. 구조화된 출력을 사용하면 원하는 형상을 정의하고 앱에서 직접 사용할 수 있는 타입이 지정된 데이터를 얻을 수 있습니다.

<AccordionGroup>
  <Accordion title="구조화된 출력이 없는 경우">
    ```text theme={null}
    Here's a classic chocolate chip cookie recipe!

    **Chocolate Chip Cookies**
    Prep time: 15 minutes | Cook time: 10 minutes

    Ingredients:
    - 2 1/4 cups all-purpose flour
    - 1 cup butter, softened
    ...
    ```

    이를 앱에서 사용하려면 제목을 파싱하고, "15 minutes"를 숫자로 변환하고, 재료와 지침을 분리하고, 응답 간의 불일치한 서식을 처리해야 합니다.
  </Accordion>

  <Accordion title="구조화된 출력이 있는 경우">
    ```json theme={null}
    {
      "name": "Chocolate Chip Cookies",
      "prep_time_minutes": 15,
      "cook_time_minutes": 10,
      "ingredients": [
        { "item": "all-purpose flour", "amount": 2.25, "unit": "cups" },
        { "item": "butter, softened", "amount": 1, "unit": "cup" }
        // ...
      ],
      "steps": ["Preheat oven to 375°F", "Cream butter and sugar" /* ... */]
    }
  ```

    UI에서 직접 사용할 수 있는 타입이 지정된 데이터입니다.
  </Accordion>
</AccordionGroup>

## 빠른 시작

구조화된 출력을 사용하려면 원하는 데이터의 형상을 설명하는 [JSON Schema](https://json-schema.org/understanding-json-schema/about)를 정의한 다음 `outputFormat` 옵션(TypeScript) 또는 `output_format` 옵션(Python)을 통해 `query()`에 전달하세요. 에이전트가 완료되면 결과 메시지에 스키마와 일치하는 검증된 데이터가 포함된 `structured_output` 필드가 포함됩니다.

아래 예시는 에이전트에게 Anthropic을 조사하고 회사 이름, 설립 연도 및 본사 위치를 구조화된 출력으로 반환하도록 요청합니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 반환받고자 하는 데이터의 형상 정의
  const schema = {
    type: "object",
    properties: {
      company_name: { type: "string" },
      founded_year: { type: "number" },
      headquarters: { type: "string" }
    },
    required: ["company_name"]
  };

  try {
    for await (const message of query({
      prompt: "Research Anthropic and provide key company information",
      options: {
        outputFormat: {
          type: "json_schema",
          schema: schema
        }
      }
    })) {
      // 결과 메시지에는 검증된 데이터가 포함된 structured_output이 들어 있습니다
      if (message.type === "result" && message.subtype === "success" && message.structured_output) {
        console.log(message.structured_output);
        // { company_name: "Anthropic", founded_year: 2021, headquarters: "San Francisco, CA" }
      }
    }
  } catch (error) {
    // 단일 실행 query()는 error_max_structured_output_retries와 같은 오류 결과 이후 에러를 발생시킵니다;
    // 오류 처리 섹션을 참조하세요.
    console.error(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

  # 반환받고자 하는 데이터의 형상 정의
  schema = {
      "type": "object",
      "properties": {
          "company_name": {"type": "string"},
          "founded_year": {"type": "number"},
          "headquarters": {"type": "string"},
      },
      "required": ["company_name"],
  }


  async def main():
      try:
          async for message in query(
              prompt="Research Anthropic and provide key company information",
              options=ClaudeAgentOptions(
                  output_format={"type": "json_schema", "schema": schema}
              ),
          ):
              # 결과 메시지에는 검증된 데이터가 포함된 structured_output이 들어 있습니다
              if isinstance(message, ResultMessage) and message.structured_output:
                  print(message.structured_output)
                  # {'company_name': 'Anthropic', 'founded_year': 2021, 'headquarters': 'San Francisco, CA'}
      except Exception as error:
          # 단일 실행 query()는 error_max_structured_output_retries와 같은 오류 결과 이후 에러를 발생시킵니다;
          # 오류 처리 섹션을 참조하세요.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

## Zod 및 Pydantic을 활용한 타입 안전 스키마

JSON Schema를 직접 작성하는 대신 [Zod](https://zod.dev/) (TypeScript) 또는 [Pydantic](https://docs.pydantic.dev/latest/) (Python)을 사용하여 스키마를 정의할 수 있습니다. 이러한 라이브러리는 JSON Schema를 대신 생성하고 응답을 파싱하여 자동 완성 및 타입 검사를 지원하는 완전한 타입의 객체로 변환해 줍니다.

아래 예시는 요약, 단계 목록(각각 복잡도 수준 포함) 및 잠재적 위험이 포함된 기능 구현 계획 스키마를 정의합니다. 에이전트는 기능을 계획하고 타입이 지정된 `FeaturePlan` 객체를 반환합니다. 그런 다음 완전한 타입 안정성을 바탕으로 `plan.summary`에 접근하거나 `plan.steps`를 순회할 수 있습니다.

SDK는 JSON Schema draft-07로 스키마를 검증하므로 더 최신 버전을 선언하는 스키마는 거부됩니다. Zod는 기본적으로 draft 2020-12를 대상으로 하므로 스키마를 변환할 때 `target: "draft-7"`을 전달하세요.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { z } from "zod";
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // Zod로 스키마 정의
  const FeaturePlan = z.object({
    feature_name: z.string(),
    summary: z.string(),
    steps: z.array(
      z.object({
        step_number: z.number(),
        description: z.string(),
        estimated_complexity: z.enum(["low", "medium", "high"])
      })
    ),
    risks: z.array(z.string())
  });

  type FeaturePlan = z.infer<typeof FeaturePlan>;

  // SDK가 기대하는 draft-07 대상을 사용하여 JSON Schema로 변환
  const schema = z.toJSONSchema(FeaturePlan, { target: "draft-7" });

  // 쿼리에서 사용
  try {
    for await (const message of query({
      prompt:
        "Plan how to add dark mode support to a React app. Break it into implementation steps.",
      options: {
        outputFormat: {
          type: "json_schema",
          schema: schema
        }
      }
    })) {
      if (message.type === "result" && message.subtype === "success" && message.structured_output) {
        // 검증 및 완전한 타입의 결과 획득
        const parsed = FeaturePlan.safeParse(message.structured_output);
        if (parsed.success) {
          const plan: FeaturePlan = parsed.data;
          console.log(`Feature: ${plan.feature_name}`);
          console.log(`Summary: ${plan.summary}`);
          plan.steps.forEach((step) => {
            console.log(`${step.step_number}. [${step.estimated_complexity}] ${step.description}`);
          });
        }
      }
    }
  } catch (error) {
    // 단일 실행 query()는 error_max_structured_output_retries와 같은 오류 결과 이후 에러를 발생시킵니다;
    // 오류 처리 섹션을 참조하세요.
    console.error(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from pydantic import BaseModel
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  class Step(BaseModel):
      step_number: int
      description: str
      estimated_complexity: str  # 'low', 'medium', 'high'


  class FeaturePlan(BaseModel):
      feature_name: str
      summary: str
      steps: list[Step]
      risks: list[str]


  async def main():
      try:
          async for message in query(
              prompt="Plan how to add dark mode support to a React app. Break it into implementation steps.",
              options=ClaudeAgentOptions(
                  output_format={
                      "type": "json_schema",
                      "schema": FeaturePlan.model_json_schema(),
                  }
              ),
          ):
              if isinstance(message, ResultMessage) and message.structured_output:
                  # 검증 및 완전한 타입의 결과 획득
                  plan = FeaturePlan.model_validate(message.structured_output)
                  print(f"Feature: {plan.feature_name}")
                  print(f"Summary: {plan.summary}")
                  for step in plan.steps:
                      print(
                          f"{step.step_number}. [{step.estimated_complexity}] {step.description}"
                      )
      except Exception as error:
          # 단일 실행 query()는 error_max_structured_output_retries와 같은 오류 결과 이후 에러를 발생시킵니다;
          # 오류 처리 섹션을 참조하세요.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

**이점:**

* 완전한 타입 추론(TypeScript) 및 타입 힌트(Python)
* `safeParse()` 또는 `model_validate()`를 통한 런타임 검증
* 더 나은 에러 메시지
* 조합 가능하고 재사용 가능한 스키마

## 출력 형식 구성 (Output Format Configuration)

`outputFormat` (TypeScript) 또는 `output_format` (Python) 옵션은 다음을 포함하는 객체를 받습니다:

* `type`: 구조화된 출력을 위해 `"json_schema"`로 설정
* `schema`: 출력 구조를 정의하는 [JSON Schema](https://json-schema.org/understanding-json-schema/about) 객체. TypeScript에서는 `z.toJSONSchema(schema, { target: "draft-7" })`로, Python에서는 `.model_json_schema()`로 생성할 수 있습니다.

SDK는 모든 기본 유형(object, array, string, number, boolean, null), `enum`, `const`, `required`, 중첩된 객체 및 `$ref` 정의를 포함한 표준 JSON Schema 기능을 지원합니다. 지원되는 기능 및 제한 사항의 전체 목록은 [JSON Schema 제한 사항](https://platform.claude.com/docs/en/build-with-claude/structured-outputs#json-schema-limitations)을 참조하세요.

유효한 JSON Schema가 아닌 스키마는 시작 시 문제를 설명하는 오류와 함께 실행에 실패합니다. v2.1.205 이전에는 유효하지 않은 스키마가 자동으로 무시되고 에러 없이 비구조화된 텍스트를 반환했습니다.

`"format": "email"`과 같은 `format` 키워드는 어노테이션으로 허용되며 SDK 검증기에 의해 강제되지 않습니다. v2.1.205 이전에는 `format`이 포함된 모든 스키마가 유효하지 않은 것으로 처리되었습니다.

## 예시: TODO 추적 에이전트

이 예시는 구조화된 출력이 다단계 도구 사용과 함께 작동하는 방식을 보여줍니다. 에이전트는 코드베이스에서 TODO 주석을 찾은 다음 각각에 대한 git blame 정보를 조회해야 합니다. 에이전트는 사용할 도구(검색용 Grep, git 명령어 실행용 Bash)를 자율적으로 결정하고 결과를 하나의 구조화된 응답으로 결합합니다.

git blame 정보를 모든 파일에서 사용할 수 있는 것은 아닐 수 있으므로 스키마에는 선택적 필드(`author` 및 `date`)가 포함됩니다. 에이전트는 찾을 수 있는 항목을 채우고 나머지는 생략합니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // TODO 추출을 위한 구조 정의
  const todoSchema = {
    type: "object",
    properties: {
      todos: {
        type: "array",
        items: {
          type: "object",
          properties: {
            text: { type: "string" },
            file: { type: "string" },
            line: { type: "number" },
            author: { type: "string" },
            date: { type: "string" }
          },
          required: ["text", "file", "line"]
        }
      },
      total_count: { type: "number" }
    },
    required: ["todos", "total_count"]
  };

  // 에이전트는 Grep으로 TODO를 찾고, Bash로 git blame 정보를 얻습니다
  try {
    for await (const message of query({
      prompt: "Find all TODO comments in this codebase and identify who added them",
      options: {
        outputFormat: {
          type: "json_schema",
          schema: todoSchema
        }
      }
    })) {
      if (message.type === "result" && message.subtype === "success" && message.structured_output) {
        const data = message.structured_output as { total_count: number; todos: Array<{ file: string; line: number; text: string; author?: string; date?: string }> };
        console.log(`Found ${data.total_count} TODOs`);
        data.todos.forEach((todo) => {
          console.log(`${todo.file}:${todo.line} - ${todo.text}`);
          if (todo.author) {
            console.log(`  Added by ${todo.author} on ${todo.date}`);
          }
        });
      }
    }
  } catch (error) {
    // 단일 실행 query()는 error_max_structured_output_retries와 같은 오류 결과 이후 에러를 발생시킵니다;
    // 오류 처리 섹션을 참조하세요.
    console.error(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

  # TODO 추출을 위한 구조 정의
  todo_schema = {
      "type": "object",
      "properties": {
          "todos": {
              "type": "array",
              "items": {
                  "type": "object",
                  "properties": {
                      "text": {"type": "string"},
                      "file": {"type": "string"},
                      "line": {"type": "number"},
                      "author": {"type": "string"},
                      "date": {"type": "string"},
                  },
                  "required": ["text", "file", "line"],
              },
          },
          "total_count": {"type": "number"},
      },
      "required": ["todos", "total_count"],
  }


  async def main():
      # 에이전트는 Grep으로 TODO를 찾고, Bash로 git blame 정보를 얻습니다
      try:
          async for message in query(
              prompt="Find all TODO comments in this codebase and identify who added them",
              options=ClaudeAgentOptions(
                  output_format={"type": "json_schema", "schema": todo_schema}
              ),
          ):
              if isinstance(message, ResultMessage) and message.structured_output:
                  data = message.structured_output
                  print(f"Found {data['total_count']} TODOs")
                  for todo in data["todos"]:
                      print(f"{todo['file']}:{todo['line']} - {todo['text']}")
                      if "author" in todo:
                          print(f"  Added by {todo['author']} on {todo['date']}")
      except Exception as error:
          # 단일 실행 query()는 error_max_structured_output_retries와 같은 오류 결과 이후 에러를 발생시킵니다;
          # 오류 처리 섹션을 참조하세요.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

## 오류 처리 (Error handling)

구조화된 출력 생성은 에이전트가 스키마와 일치하는 유효한 JSON을 생성할 수 없을 때 실패할 수 있습니다. 이는 일반적으로 스키마가 작업에 비해 너무 복잡하거나, 작업 자체가 모호하거나, 검증 오류를 수정하려는 재시도 제한에 도달했을 때 발생합니다. 또한 검증 실패 없이도 발생할 수 있습니다: [모델 폴백](/docs/en/model-config#automatic-model-fallback)이 스트림 중간에 이미 완료된 출력을 철회하고 성공적인 재시도가 교체되지 않으면 동일한 에러와 함께 실행이 끝납니다. 스키마를 디버깅하기 전에 결과 메시지의 `errors` 목록을 확인하여 두 원인을 구별하세요.

오류가 발생하면 결과 메시지에는 무엇이 잘못되었는지 나타내는 `subtype`이 포함됩니다:

| 서브타입 (Subtype)                     | 의미                                                                                                                             |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `success`                             | 출력이 성공적으로 생성되고 검증됨                                                                                                |
| `error_max_structured_output_retries` | 여러 번의 시도(검증 실패 또는 성공적인 재시도가 없는 모델 폴백 철회) 후에도 유효한 출력이 남아 있지 않음                             |

결과는 서브타입이 `success`이지만 `structured_output` 값이 없는 상태로 끝날 수도 있습니다(예: 에이전트가 구조화된 출력을 생성하지 않고 실행이 완료된 경우). 이 경우도 실패로 처리하세요. 아래 예시는 `subtype`이 `success`이고 `structured_output`이 존재하는 경우에만 성공으로 처리하고 다른 모든 결과는 실패로 처리합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const contactSchema = {
    type: "object",
    properties: {
      name: { type: "string" },
      email: { type: "string" }
    },
    required: ["name"]
  };

  try {
    for await (const msg of query({
      prompt: "Extract contact info from the document",
      options: {
        outputFormat: {
          type: "json_schema",
          schema: contactSchema
        }
      }
    })) {
      if (msg.type === "result") {
        if (msg.subtype === "success" && msg.structured_output) {
          // 검증된 출력 사용
          console.log(msg.structured_output);
        } else if (msg.subtype === "error_max_structured_output_retries") {
          console.error("Could not produce valid output");
        } else {
          console.error("Run ended without a structured output");
        }
      }
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과 이후 에러를 발생시킵니다.
    // 실패가 오류 결과인 경우 위의 subtype 분기가 이미 실행되었습니다.
    // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
    // 여기서 실패를 처리하세요 - 더 간단한 프롬프트로 재시도, 비구조화로 폴백 등
    console.log(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

  contact_schema = {
      "type": "object",
      "properties": {
          "name": {"type": "string"},
          "email": {"type": "string"},
      },
      "required": ["name"],
  }


  async def main():
      try:
          async for message in query(
              prompt="Extract contact info from the document",
              options=ClaudeAgentOptions(
                  output_format={"type": "json_schema", "schema": contact_schema}
              ),
          ):
              if isinstance(message, ResultMessage):
                  if message.subtype == "success" and message.structured_output:
                      # 검증된 출력 사용
                      print(message.structured_output)
                  elif message.subtype == "error_max_structured_output_retries":
                      print("Could not produce valid output")
                  else:
                      print("Run ended without a structured output")
      except Exception as error:
          # 단일 실행 query()는 오류 결과 이후 에러를 발생시킵니다.
          # 실패가 오류 결과인 경우 위의 subtype 분기가 이미 실행되었습니다.
          # 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
          # 여기서 실패를 처리하세요 - 더 간단한 프롬프트로 재시도, 비구조화로 폴백 등
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

**오류 방지를 위한 팁:**

* **스키마를 명확하게 유지하세요.** 필수 필드가 많은 깊게 중첩된 스키마는 충족하기 어렵습니다. 단순하게 시작하여 필요에 따라 복잡성을 추가하세요.
* **스키마를 작업에 맞추세요.** 작업에 스키마가 요구하는 모든 정보가 포함되어 있지 않을 수 있는 경우 해당 필드를 선택 사항으로 만드세요.
* **명확한 프롬프트를 사용하세요.** 모호한 프롬프트는 에이전트가 어떤 출력을 생성해야 하는지 알기 어렵게 만듭니다.

## 관련 리소스

* [JSON Schema 문서](https://json-schema.org/): 중첩된 객체, 배열, 이넘 및 검증 제약 조건이 포함된 복잡한 스키마 정의를 위한 JSON Schema 구문 배우기
* [API 구조화된 출력](https://platform.claude.com/docs/en/build-with-claude/structured-outputs): 도구 사용 없이 단일 요청을 위해 Claude API를 직접 사용한 구조화된 출력 활용
* [커스텀 도구](/docs/en/agent-sdk/custom-tools): 구조화된 출력을 반환하기 전에 실행하는 동안 에이전트가 호출할 커스텀 도구 제공
