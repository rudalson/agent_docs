> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# Claude에게 커스텀 도구 제공하기

> Claude가 내 함수를 호출하고, API를 타격하며, 도메인 특화 작업을 수행할 수 있도록 Claude Agent SDK의 인프로세스 MCP 서버로 커스텀 도구를 정의합니다.

커스텀 도구는 대화 중에 Claude가 호출할 수 있는 사용자 정의 함수를 정의할 수 있게 해줌으로써 Agent SDK를 확장합니다. SDK의 인프로세스(in-process) MCP 서버를 사용하면 Claude에게 데이터베이스, 외부 API, 도메인 특화 로직 또는 애플리케이션에 필요한 기타 기능에 대한 접근 권한을 부여할 수 있습니다.

이 가이드에서는 입력 스키마 및 핸들러로 도구를 정의하고, 이를 MCP 서버로 번들링하여 `query`에 전달하고, Claude가 접근할 수 있는 도구를 제어하는 방법을 설명합니다. 또한 오류 처리, 도구 주석(annotations) 및 이미지와 같은 비텍스트 콘텐츠 반환 방법도 다룹니다.

## 빠른 참조

| 목표 | 수행 방법 |
| :------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 도구 정의 | 이름, 설명, 스키마 및 핸들러와 함께 [`@tool`](/docs/en/agent-sdk/python#tool) (Python) 또는 [`tool()`](/docs/en/agent-sdk/typescript#tool) (TypeScript)을 사용합니다. [커스텀 도구 생성하기](#커스텀-도구-생성하기)를 참조하세요. |
| Claude에 도구 등록 | `create_sdk_mcp_server` / `createSdkMcpServer`로 감싼 후 `query()`의 `mcpServers`에 전달합니다. [커스텀 도구 호출하기](#커스텀-도구-호출하기)를 참조하세요. |
| 도구 사전 승인 | 허용된 도구 목록에 추가합니다. [허용된 도구 구성하기](#허용된-도구-구성하기)를 참조하세요. |
| Claude 컨텍스트에서 내장 도구 제거 | 원하는 내장 도구만 나열한 `tools` 배열을 전달합니다. [허용된 도구 구성하기](#허용된-도구-구성하기)를 참조하세요. |
| Claude의 병렬 도구 호출 허용 | 사이드 이펙트(부작용)가 없는 도구에 `readOnlyHint: true`를 설정합니다. [도구 주석 추가하기](#도구-주석-추가하기)를 참조하세요. |
| Claude가 읽는 오류 메시지 제어 | 원시 예외를 노출하는 대신 메시지를 구성하기 위해 `isError: true`를 반환합니다. [오류 처리하기](#오류-처리하기)를 참조하세요. |
| 이미지 또는 파일 반환 | content 배열에서 `image` 또는 `resource` 블록을 사용합니다. [이미지 및 리소스 반환하기](#이미지-및-리소스-반환하기)를 참조하세요. |
| 기계 읽기 가능한 JSON 결과 반환 | 결과에 `structuredContent`를 설정합니다. [구조화된 데이터 반환하기](#구조화된-데이터-반환하기)를 참조하세요. |
| 다수의 도구로 확장 | 필요 시 도구를 로드하려면 [도구 검색](/docs/en/agent-sdk/tool-search)을 사용합니다. |

## 커스텀 도구 생성하기

도구는 TypeScript의 [`tool()`](/docs/en/agent-sdk/typescript#tool) 헬퍼 또는 Python의 [`@tool`](/docs/en/agent-sdk/python#tool) 데코레이터에 인자로 전달되는 4개 부분으로 정의됩니다:

* **이름(Name):** Claude가 도구를 호출할 때 사용하는 고유 식별자.
* **설명(Description):** 도구가 수행하는 작업. Claude는 이를 읽고 언제 호출할지 결정합니다.
* **입력 스키마(Input schema):** Claude가 제공해야 하는 인자. TypeScript에서 이는 항상 [Zod 스키마](https://zod.dev/)이며, 핸들러의 `args`는 스키마로부터 자동으로 타입이 지정됩니다. Python에서 이는 `{"latitude": float}`와 같이 이름을 타입에 매핑하는 딕셔너리이며 SDK가 JSON Schema로 변환해 줍니다. Python 데코레이터는 열거형(enums), 범위, 선택적 필드 또는 중첩 객체가 필요할 때 전체 [JSON Schema](https://json-schema.org/understanding-json-schema/about) 딕셔너리를 직접 수용할 수도 있습니다.
* **핸들러(Handler):** Claude가 도구를 호출할 때 실행되는 비동기 함수. 검증된 인자를 수신하며 다음을 포함하는 객체를 반환해야 합니다:
  * `content` (필수): 결과 블록의 배열. 각 블록은 `"text"`, `"image"`, `"audio"`, `"resource"`, 또는 `"resource_link"`의 `type`을 가집니다. 비텍스트 블록은 [이미지 및 리소스 반환하기](#이미지-및-리소스-반환하기)를 참조하세요.
  * `structuredContent` (선택): `content`와 함께 반환되는 기계 읽기 가능한 데이터로서의 결과를 담은 JSON 객체. [구조화된 데이터 반환하기](#구조화된-데이터-반환하기)를 참조하세요.
  * `isError` (선택): Claude가 이에 반응할 수 있도록 도구 실패를 알리려면 `true`로 설정. [오류 처리하기](#오류-처리하기)를 참조하세요.

도구를 정의한 후 [`createSdkMcpServer`](/docs/en/agent-sdk/typescript#createsdkmcpserver) (TypeScript) 또는 [`create_sdk_mcp_server`](/docs/en/agent-sdk/python#create_sdk_mcp_server) (Python)로 서버에 감싸세요. 서버는 별도의 프로세스가 아닌 애플리케이션 내부에서 인프로세스로 실행됩니다.

### 날씨 도구 예제

이 예제는 `get_temperature` 도구를 정의하고 이를 MCP 서버로 감쌉니다. 도구 구성만 수행하며, `query`에 전달하고 실행하는 방법은 아래의 [커스텀 도구 호출하기](#커스텀-도구-호출하기)를 참조하세요.

<CodeGroup>
  ```python Python theme={null}
  from typing import Any
  import httpx
  from claude_agent_sdk import tool, create_sdk_mcp_server


  # 도구 정의: 이름, 설명, 입력 스키마, 핸들러
  @tool(
      "get_temperature",
      "Get the current temperature at a location",
      {"latitude": float, "longitude": float},
  )
  async def get_temperature(args: dict[str, Any]) -> dict[str, Any]:
      async with httpx.AsyncClient() as client:
          response = await client.get(
              "https://api.open-meteo.com/v1/forecast",
              params={
                  "latitude": args["latitude"],
                  "longitude": args["longitude"],
                  "current": "temperature_2m",
                  "temperature_unit": "fahrenheit",
              },
          )
          data = response.json()

      # content 배열 반환 - Claude는 이를 도구 결과로 봄
      return {
          "content": [
              {
                  "type": "text",
                  "text": f"Temperature: {data['current']['temperature_2m']}°F",
              }
          ]
      }


  # 도구를 인프로세스 MCP 서버로 감싸기
  weather_server = create_sdk_mcp_server(
      name="weather",
      version="1.0.0",
      tools=[get_temperature],
  )
  ```

  ```typescript TypeScript theme={null}
  import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
  import { z } from "zod";

  // 도구 정의: 이름, 설명, 입력 스키마, 핸들러
  const getTemperature = tool(
    "get_temperature",
    "Get the current temperature at a location",
    {
      latitude: z.number().describe("Latitude coordinate"), // .describe()는 Claude가 보는 필드 설명을 추가함
      longitude: z.number().describe("Longitude coordinate")
    },
    async (args) => {
      // args는 스키마로부터 타입이 지정됨: { latitude: number; longitude: number }
      const response = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${args.latitude}&longitude=${args.longitude}&current=temperature_2m&temperature_unit=fahrenheit`
      );
      const data: any = await response.json();

      // content 배열 반환 - Claude는 이를 도구 결과로 봄
      return {
        content: [{ type: "text", text: `Temperature: ${data.current.temperature_2m}°F` }]
      };
    }
  );

  // 도구를 인프로세스 MCP 서버로 감싸기
  const weatherServer = createSdkMcpServer({
    name: "weather",
    version: "1.0.0",
    tools: [getTemperature]
  });
  ```
</CodeGroup>

JSON Schema 입력 형식 및 반환 값 구조를 포함한 전체 파라미터 세부 정보는 [`tool()`](/docs/en/agent-sdk/typescript#tool) TypeScript 참조 또는 [`@tool`](/docs/en/agent-sdk/python#tool) Python 참조를 참조하세요.

<Tip>
  파라미터를 선택 사항(optional)으로 만드는 방법: TypeScript에서는 Zod 필드에 `.default()`를 추가합니다. Python에서는 딕셔너리 스키마가 모든 키를 필수 항목으로 처리하므로 파라미터를 스키마에서 제외하고 설명 문자열에 명시한 뒤 핸들러에서 `args.get()`으로 읽습니다. 아래의 [`get_precipitation_chance` 도구](#도구-추가하기)에서 두 패턴을 모두 보여줍니다.
</Tip>

### 커스텀 도구 호출하기

`mcpServers` 옵션을 통해 생성한 MCP 서버를 `query`에 전달하세요. `mcpServers`에 지정한 키는 각 도구의 정격 이름(fully qualified name)에서 `{server_name}` 세그먼트(`mcp__{server_name}__{tool_name}`)가 됩니다. 권한 프롬프트 없이 도구가 실행되도록 `allowedTools`에 해당 이름을 나열하세요.

다음 코드 조각은 [위 예제](#날씨-도구-예제)의 `weatherServer`를 재사용하여 특정 위치의 날씨를 Claude에게 묻습니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={"weather": weather_server},
          allowed_tools=["mcp__weather__get_temperature"],
      )

      async for message in query(
          prompt="What's the temperature in San Francisco?",
          options=options,
      ):
          # ResultMessage는 모든 도구 호출이 완료된 후의 최종 메시지임
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "What's the temperature in San Francisco?",
    options: {
      mcpServers: { weather: weatherServer },
      allowedTools: ["mcp__weather__get_temperature"]
    }
  })) {
    // "result"는 모든 도구 호출이 완료된 후의 최종 메시지임
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```
</CodeGroup>

이 코드 조각을 [날씨 도구 예제](#날씨-도구-예제)의 도구 및 서버 정의와 하나의 파일로 결합한 다음, Python의 경우 `python weather.py`, TypeScript의 경우 `npx tsx weather.ts`로 실행하세요. Claude가 `get_temperature`를 호출하고 스크립트는 샌프란시스코의 현재 온도가 포함된 한 줄 답변을 출력합니다.

### 도구 추가하기

서버는 `tools` 배열에 나열한 만큼의 도구를 포함합니다. 서버에 여러 도구가 있는 경우 `allowedTools`에 각 도구를 개별적으로 나열하거나 와일드카드 `mcp__weather__*`를 사용하여 서버가 노출하는 모든 도구를 처리할 수 있습니다.

아래 예제는 두 번째 도구인 `get_precipitation_chance`를 정의하고, [날씨 도구 예제](#날씨-도구-예제)의 `weatherServer` 정의를 두 도구를 모두 나열하는 정의로 교체합니다.

<CodeGroup>
  ```python Python theme={null}
  # 동일한 서버를 위한 두 번째 도구 정의
  @tool(
      "get_precipitation_chance",
      "Get the hourly precipitation probability for a location. "
      "Optionally pass 'hours' (1-24) to control how many hours to return.",
      {"latitude": float, "longitude": float},
  )
  async def get_precipitation_chance(args: dict[str, Any]) -> dict[str, Any]:
      # 'hours'는 스키마에 없음 - 선택 사항으로 만들기 위해 .get()으로 읽음
      hours = args.get("hours", 12)
      async with httpx.AsyncClient() as client:
          response = await client.get(
              "https://api.open-meteo.com/v1/forecast",
              params={
                  "latitude": args["latitude"],
                  "longitude": args["longitude"],
                  "hourly": "precipitation_probability",
                  "forecast_days": 1,
              },
          )
          data = response.json()
      chances = data["hourly"]["precipitation_probability"][:hours]

      return {
          "content": [
              {
                  "type": "text",
                  "text": f"Next {hours} hours: {'%, '.join(map(str, chances))}%",
              }
          ]
      }


  # 배열에 두 도구를 모두 넣어 서버 재구성
  weather_server = create_sdk_mcp_server(
      name="weather",
      version="1.0.0",
      tools=[get_temperature, get_precipitation_chance],
  )
  ```

  ```typescript TypeScript theme={null}
  // 동일한 서버를 위한 두 번째 도구 정의
  const getPrecipitationChance = tool(
    "get_precipitation_chance",
    "Get the hourly precipitation probability for a location",
    {
      latitude: z.number(),
      longitude: z.number(),
      hours: z
        .number()
        .int()
        .min(1)
        .max(24)
        .default(12) // .default()로 파라미터를 선택 사항으로 만듦
        .describe("How many hours of forecast to return")
    },
    async (args) => {
      const response = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${args.latitude}&longitude=${args.longitude}&hourly=precipitation_probability&forecast_days=1`
      );
      const data: any = await response.json();
      const chances = data.hourly.precipitation_probability.slice(0, args.hours);

      return {
        content: [{ type: "text", text: `Next ${args.hours} hours: ${chances.join("%, ")}%` }]
      };
    }
  );

  // 배열에 두 도구를 모두 넣어 서버 재구성
  const weatherServer = createSdkMcpServer({
    name: "weather",
    version: "1.0.0",
    tools: [getTemperature, getPrecipitationChance]
  });
  ```
</CodeGroup>

[도구 검색](/docs/en/agent-sdk/tool-search)은 기본적으로 켜져 있으며 SDK MCP 도구를 지연시킵니다: Claude는 간결한 목록에서 각 도구의 이름을 확인하고 필요 시 전체 스키마를 로드합니다. 도구 검색이 비활성화되면 이 배열의 모든 도구가 매 턴마다 컨텍스트 윈도우 공간을 소모합니다. TypeScript에서는 [`tool()`](/docs/en/agent-sdk/typescript#tool)의 `extras` 인자나 [`createSdkMcpServer()`](/docs/en/agent-sdk/typescript#createsdkmcpserver)의 옵션에 `alwaysLoad: true`를 전달하여 초기 프롬프트에 도구의 전체 스키마를 유지할 수 있습니다.

### 도구 주석 추가하기

[도구 주석(Tool annotations)](https://modelcontextprotocol.io/docs/concepts/tools#tool-annotations)은 도구가 작동하는 방식을 설명하는 선택적 메타데이터입니다. TypeScript에서는 `tool()` 헬퍼의 5번째 인자로 전달하고, Python에서는 `@tool` 데코레이터의 `annotations` 키워드 인자를 통해 전달합니다. 모든 힌트 필드는 불리언(Boolean)입니다.

| 필드 | 기본값 | 의미 |
| :---------------- | :------ | :-------------------------------------------------------------------------------------------------------------------- |
| `readOnlyHint` | `false` | 도구가 환경을 수정하지 않음. 타 읽기 전용 도구와 동시 호출 가능 여부를 제어함. |
| `destructiveHint` | `true` | 도구가 파괴적인 업데이트를 수행할 수 있음. 정보 제공 전용. |
| `idempotentHint` | `false` | 동일한 인자로 반복 호출해도 추가적인 효과가 없음. 정보 제공 전용. |
| `openWorldHint` | `true` | 도구가 프로세스 외부 시스템에 접근함. 정보 제공 전용. |

주석은 메타데이터일 뿐 강제 적용 요소는 아닙니다. `readOnlyHint: true`로 표시된 도구도 핸들러 로직이 그렇다면 디스크에 쓸 수 있습니다. 핸들러 동작과 일치하도록 주석을 정확하게 유지하세요.

이 예제는 [날씨 도구 예제](#날씨-도구-예제)의 `get_temperature` 도구에 `readOnlyHint`를 추가합니다.

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import tool, ToolAnnotations


  @tool(
      "get_temperature",
      "Get the current temperature at a location",
      {"latitude": float, "longitude": float},
      annotations=ToolAnnotations(
          readOnlyHint=True
      ),  # Claude가 이를 다른 읽기 전용 호출과 일괄 처리할 수 있게 함
  )
  async def get_temperature(args):
      return {"content": [{"type": "text", "text": "..."}]}
  ```

  ```typescript TypeScript theme={null}
  import { tool } from "@anthropic-ai/claude-agent-sdk";
  import { z } from "zod";

  tool(
    "get_temperature",
    "Get the current temperature at a location",
    { latitude: z.number(), longitude: z.number() },
    async (args) => ({ content: [{ type: "text", text: `...` }] }),
    { annotations: { readOnlyHint: true } } // Claude가 이를 다른 읽기 전용 호출과 일괄 처리할 수 있게 함
  );
  ```
</CodeGroup>

[TypeScript](/docs/en/agent-sdk/typescript#toolannotations) 또는 [Python](/docs/en/agent-sdk/python#toolannotations) 참조의 `ToolAnnotations`를 참조하세요.

## 도구 접근 제어하기

[날씨 도구 예제](#날씨-도구-예제)에서는 서버를 등록하고 `allowedTools`에 도구를 나열했습니다. 이 섹션에서는 도구 이름 구성 방식과 여러 도구가 있거나 내장 도구를 제한하고자 할 때 접근 범위를 제어하는 방식을 다룹니다.

### 도구 이름 형식

MCP 도구가 Claude에 노출될 때 이름은 특정 형식을 따릅니다:

* 패턴: `mcp__{server_name}__{tool_name}`
* 예시: `weather` 서버의 `get_temperature` 도구는 `mcp__weather__get_temperature`가 됨

### 허용된 도구 구성하기

`tools` 옵션과 허용/거부 목록은 두 가지 계층에 영향을 미칩니다: 가용성(availability)은 도구가 Claude의 컨텍스트에 나타나는지 여부를 제어하고, 권한(permission)은 Claude가 시도한 호출이 승인되는지 여부를 제어합니다. `tools` 및 베어 네임(bare-name) `disallowedTools` 항목은 가용성을 변경합니다. `allowedTools` 및 범위 지정(scoped) `disallowedTools` 규칙은 권한만 변경합니다.

| 옵션 | 계층 | 효과 |
| :------------------------ | :----------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tools: ["Read", "Grep"]` | 가용성 | 나열된 내장 도구만 Claude의 컨텍스트에 포함됩니다. 나열되지 않은 내장 도구는 제거됩니다. MCP 도구는 영향받지 않습니다. |
| `tools: []` | 가용성 | 모든 내장 도구가 제거됩니다. Claude는 사용자 MCP 도구만 사용할 수 있습니다. |
| 허용된 도구 | 권한 | 나열된 도구는 권한 프롬프트 없이 실행됩니다. 나열되지 않은 도구도 사용 가능 상태로 유지되며 호출 시 [권한 흐름](/docs/en/agent-sdk/permissions)을 거칩니다. |
| 거부된 도구 | 둘 다 | `"Bash"`와 같은 베어 도구 이름은 `tools`에서 생략한 것과 동일하게 Claude의 컨텍스트에서 도구를 제거합니다. `"Bash(rm *)"`와 같은 범위 지정 규칙은 도구를 컨텍스트에 남겨두고 일치하는 호출만 거부합니다. |

내장 도구를 완전히 제거하려면 `tools`에서 생략하거나 `disallowedTools`(Python: `disallowed_tools`)에 베어 이름을 나열하세요. 둘 다 컨텍스트에서 도구를 배제하므로 Claude가 시도조차 하지 않습니다. 범위 지정 `disallowedTools` 규칙은 일치하는 호출을 차단하지만 도구를 시각적으로 남겨두므로 Claude가 시도하다 턴을 낭비할 수 있습니다. 전체 평가 순서는 [권한 구성](/docs/en/agent-sdk/permissions)을 참조하세요.

## 오류 처리하기

핸들러 오류가 발생해도 에이전트 루프는 중단되지 않습니다. SDK의 인프로세스 MCP 서버는 포착되지 않은 예외를 포착하여 오류 결과로 반환하므로, 오류 보고 방식은 쿼리 실패 여부가 아니라 Claude가 읽는 내용을 결정합니다:

| 상황 | 결과 |
| :----------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| 핸들러가 포착되지 않은 예외를 던짐 | MCP 서버가 원시 예외 메시지를 담은 오류 결과로 변환합니다. Claude는 해당 메시지를 확인하고 에이전트 루프는 계속됩니다. |
| 핸들러가 오류를 포착하여 `isError: true` (TS) / `"is_error": True` (Python) 반환 | Claude는 작성한 메시지를 확인합니다. 어떤 요청이 실패했는지, 대신 시도할 작업이 무엇인지 등 원시 예외에 없는 컨텍스트를 추가할 수 있습니다. |

두 경우 모두 Claude는 재시도하거나 다른 도구를 시도하거나 실패를 설명할 수 있습니다. 원시 예외 메시지만으로 Claude가 조치를 취하기에 부족한 경우 오류를 직접 포착하세요.

아래 예제는 핸들러 내부에서 두 가지 유형의 실패를 포착하고 Claude가 읽는 오류 메시지를 작성합니다. 200이 아닌 HTTP 상태는 응답에서 포착되어 오류 결과로 반환됩니다. 네트워크 오류나 잘못된 JSON은 감싸고 있는 `try/except` (Python) 또는 `try/catch` (TypeScript)에 의해 포착되어 역시 오류 결과로 반환됩니다. 두 경우 모두 Claude는 단순 예외 문자열 대신 실패를 설명하는 메시지를 수신합니다.

<CodeGroup>
  ```python Python theme={null}
  import json
  import httpx
  from typing import Any
  from claude_agent_sdk import tool


  @tool(
      "fetch_data",
      "Fetch data from an API",
      {"endpoint": str},  # 단순 스키마
  )
  async def fetch_data(args: dict[str, Any]) -> dict[str, Any]:
      try:
          async with httpx.AsyncClient() as client:
              response = await client.get(args["endpoint"])
              if response.status_code != 200:
                  # Claude가 반응할 수 있도록 실패를 도구 결과로 반환합니다.
                  # is_error는 이를 이상한 데이터가 아닌 실패한 호출로 표시합니다.
                  return {
                      "content": [
                          {
                              "type": "text",
                              "text": f"API error: {response.status_code} {response.reason_phrase}",
                          }
                      ],
                      "is_error": True,
                  }

              data = response.json()
              return {"content": [{"type": "text", "text": json.dumps(data, indent=2)}]}
      except Exception as e:
          # Claude가 읽는 메시지를 작성합니다. 포착되지 않은 예외는
          # 컨텍스트 없이 원시 str(e)로 Claude에게 전달됩니다.
          return {
              "content": [{"type": "text", "text": f"Failed to fetch data: {str(e)}"}],
              "is_error": True,
          }
  ```

  ```typescript TypeScript theme={null}
  import { tool } from "@anthropic-ai/claude-agent-sdk";
  import { z } from "zod";

  tool(
    "fetch_data",
    "Fetch data from an API",
    {
      endpoint: z.string().url().describe("API endpoint URL")
    },
    async (args) => {
      try {
        const response = await fetch(args.endpoint);

        if (!response.ok) {
          // Claude가 반응할 수 있도록 실패를 도구 결과로 반환합니다.
          // isError는 이를 이상한 데이터가 아닌 실패한 호출로 표시합니다.
          return {
            content: [
              {
                type: "text",
                text: `API error: ${response.status} ${response.statusText}`
              }
            ],
            isError: true
          };
        }

        const data = await response.json();
        return {
          content: [
            {
              type: "text",
              text: JSON.stringify(data, null, 2)
            }
          ]
        };
      } catch (error) {
        // Claude가 읽는 메시지를 작성합니다. 포착되지 않은 예외는
        // 컨텍스트 없이 원시 오류 메시지로 Claude에게 전달됩니다.
        return {
          content: [
            {
              type: "text",
              text: `Failed to fetch data: ${error instanceof Error ? error.message : String(error)}`
            }
          ],
          isError: true
        };
      }
    }
  );
  ```
</CodeGroup>

## 이미지 및 리소스 반환하기

도구 결과의 `content` 배열은 `text`, `image`, `audio`, `resource`, `resource_link` 블록을 수용합니다. 동일한 응답에서 이들을 혼합할 수 있습니다. TypeScript에서 오디오 블록은 디스크에 저장되고 Claude는 저장된 파일 경로가 포함된 텍스트 블록을 수신합니다. Python에서는 SDK가 도구 결과에서 오디오 블록을 제거하고 경고를 기록합니다. 리소스 링크 블록은 링크의 이름, URI, 설명이 포함된 텍스트 블록으로 변환됩니다.

### 이미지

이미지 블록은 base64로 인코딩된 이미지 바이트를 인라인으로 전달합니다. URL 필드는 없습니다. URL에 있는 이미지를 반환하려면 핸들러에서 이를 가져오고 응답 바이트를 읽은 다음 반환하기 전에 base64로 인코딩하세요. 결과는 시각적 입력으로 처리됩니다.

| 필드 | 타입 | 참고 사항 |
| :--------- | :-------- | :------------------------------------------------------------------------- |
| `type` | `"image"` | |
| `data` | `string` | Base64 인코딩된 바이트. 순수 base64만 허용되며 `data:image/...;base64,` 접두사 없음 |
| `mimeType` | `string` | 필수. 예: `image/png`, `image/jpeg`, `image/webp`, `image/gif` |

<CodeGroup>
  ```python Python theme={null}
  import base64
  import httpx
  from claude_agent_sdk import tool


  # URL에서 이미지를 가져와 Claude에게 반환하는 도구 정의
  @tool("fetch_image", "Fetch an image from a URL and return it to Claude", {"url": str})
  async def fetch_image(args):
      async with httpx.AsyncClient() as client:  # 이미지 바이트 가져오기
          response = await client.get(args["url"])

      return {
          "content": [
              {
                  "type": "image",
                  "data": base64.b64encode(response.content).decode(
                      "ascii"
                  ),  # 순수 바이트를 base64 인코딩
                  "mimeType": response.headers.get(
                      "content-type", "image/png"
                  ),  # 응답에서 MIME 타입 읽기
              }
          ]
      }
  ```

  ```typescript TypeScript theme={null}
  import { tool } from "@anthropic-ai/claude-agent-sdk";
  import { z } from "zod";

  tool(
    "fetch_image",
    "Fetch an image from a URL and return it to Claude",
    {
      url: z.string().url()
    },
    async (args) => {
      const response = await fetch(args.url); // 이미지 바이트 가져오기
      const buffer = Buffer.from(await response.arrayBuffer()); // base64 인코딩을 위해 Buffer로 읽기
      const mimeType = response.headers.get("content-type") ?? "image/png";

      return {
        content: [
          {
            type: "image",
            data: buffer.toString("base64"), // 순수 바이트를 base64 인코딩
            mimeType
          }
        ]
      };
    }
  );
  ```
</CodeGroup>

### 리소스

리소스 블록은 URI로 식별되는 콘텐츠 조각을 임베딩합니다. URI는 Claude가 참조할 레이블이며 실제 콘텐츠는 블록의 `text` 또는 `blob` 필드에 들어갑니다. 도구가 생성된 파일이나 외부 시스템의 레코드와 같이 나중에 이름으로 참조하는 것이 적절한 산출물을 생성할 때 사용하세요.

| 필드 | 타입 | 참고 사항 |
| :------------------ | :----------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `type` | `"resource"` | |
| `resource.uri` | `string` | 콘텐츠 식별자. 임의의 URI 스키마 가능 |
| `resource.text` | `string` | 텍스트인 경우의 콘텐츠. 이것이나 `blob` 중 하나만 제공함 |
| `resource.blob` | `string` | 바이너리인 경우 base64 인코딩된 콘텐츠. TypeScript 전용: Python SDK는 도구 결과에서 바이너리 리소스를 제거하고 경고를 기록함 |
| `resource.mimeType` | `string` | 선택 사항 |

이 예제는 도구 핸들러 내부에서 반환된 리소스 블록을 보여줍니다. URI `file:///tmp/report.md`는 Claude가 나중에 참조할 수 있는 레이블이며 SDK가 해당 경로에서 읽어오지 않습니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  return {
    content: [
      {
        type: "resource",
        resource: {
          uri: "file:///tmp/report.md", // Claude가 참조할 레이블이며, SDK가 읽는 경로가 아님
          mimeType: "text/markdown",
          text: "# Report\n..." // 인라인 형태의 실제 콘텐츠
        }
      }
    ]
  };
  ```

  ```python Python theme={null}
  return {
      "content": [
          {
              "type": "resource",
              "resource": {
                  "uri": "file:///tmp/report.md",  # Claude가 참조할 레이블이며, SDK가 읽는 경로가 아님
                  "mimeType": "text/markdown",
                  "text": "# Report\n...",  # 인라인 형태의 실제 콘텐츠
              },
          }
      ]
  }
  ```
</CodeGroup>

이러한 블록 형상은 MCP `CallToolResult` 타입에서 비롯됩니다. 전체 정의는 [MCP 사양](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#tool-result)을 참조하세요.

## 구조화된 데이터 반환하기

`structuredContent`는 `content` 배열과 별도로 결과에 추가되는 선택적 JSON 객체입니다. 텍스트 문자열이나 이미지에서 파싱하는 대신 Claude가 정확한 필드로 읽을 수 있는 원시 값을 반환할 때 사용하세요.

`structuredContent`가 설정되면 Claude는 JSON과 함께 `content`의 모든 이미지 또는 리소스 블록을 수신합니다. `content`에 있는 텍스트 블록은 구조화된 데이터를 중복하는 것으로 간주되어 전달되지 않습니다. 아래 예제는 차트를 이미지 블록으로 렌더링하고 동일한 핸들러의 `structuredContent`에서 그 뒤의 데이터 포인트를 반환합니다. 코드 조각에서 `chartPngBuffer`는 렌더링된 PNG 바이트를 담고 있는 `Buffer`입니다.

```typescript TypeScript theme={null}
return {
  content: [
    {
      type: "image",
      data: chartPngBuffer.toString("base64"),
      mimeType: "image/png"
    }
  ],
  structuredContent: {
    series: "temperature_2m",
    unit: "fahrenheit",
    points: [62.1, 63.4, 65.0, 64.2]
  }
};
```

<Note>
  Python `@tool` 데코레이터는 핸들러 반환 딕셔너리에서 `content` 및 `is_error`만 전달합니다. Python에서 `structuredContent`를 반환하려면 인프로세스 SDK 서버 대신 [독립형 MCP 서버](/docs/en/agent-sdk/mcp)를 실행하세요.
</Note>

## 예제: 단위 변환기

이 도구는 길이, 온도, 무게 단위 간의 값을 변환합니다. 사용자가 "100킬로미터를 마일로 변환해줘" 또는 "72°F는 섭씨로 얼마인가요?"라고 질문하면 Claude는 요청에서 올바른 단위 유형과 단위를 선택합니다.

두 가지 패턴을 시연합니다:

* **Enum 스키마:** `unit_type`이 고정된 값 세트로 제한됩니다. TypeScript에서는 `z.enum()`을 사용하세요. Python에서 딕셔너리 스키마는 enum을 지원하지 않으므로 전체 JSON Schema 딕셔너리가 필요합니다.
* **지원되지 않는 입력 처리:** 변환 쌍을 찾을 수 없는 경우 핸들러가 `isError: true`를 반환하여 Claude가 실패를 일반 결과로 처리하지 않고 사용자에게 무엇이 잘못되었는지 알려줄 수 있도록 합니다.

<CodeGroup>
  ```python Python theme={null}
  from typing import Any
  from claude_agent_sdk import tool, create_sdk_mcp_server


  # TypeScript의 z.enum()은 JSON Schema에서 "enum" 제약 조건이 됨.
  # 딕셔너리 스키마에는 동등한 표현이 없으므로 전체 JSON Schema가 필요함.
  @tool(
      "convert_units",
      "Convert a value from one unit to another",
      {
          "type": "object",
          "properties": {
              "unit_type": {
                  "type": "string",
                  "enum": ["length", "temperature", "weight"],
                  "description": "Category of unit",
              },
              "from_unit": {
                  "type": "string",
                  "description": "Unit to convert from, e.g. kilometers, fahrenheit, pounds",
              },
              "to_unit": {"type": "string", "description": "Unit to convert to"},
              "value": {"type": "number", "description": "Value to convert"},
          },
          "required": ["unit_type", "from_unit", "to_unit", "value"],
      },
  )
  async def convert_units(args: dict[str, Any]) -> dict[str, Any]:
      conversions = {
          "length": {
              "kilometers_to_miles": lambda v: v * 0.621371,
              "miles_to_kilometers": lambda v: v * 1.60934,
              "meters_to_feet": lambda v: v * 3.28084,
              "feet_to_meters": lambda v: v * 0.3048,
          },
          "temperature": {
              "celsius_to_fahrenheit": lambda v: (v * 9) / 5 + 32,
              "fahrenheit_to_celsius": lambda v: (v - 32) * 5 / 9,
              "celsius_to_kelvin": lambda v: v + 273.15,
              "kelvin_to_celsius": lambda v: v - 273.15,
          },
          "weight": {
              "kilograms_to_pounds": lambda v: v * 2.20462,
              "pounds_to_kilograms": lambda v: v * 0.453592,
              "grams_to_ounces": lambda v: v * 0.035274,
              "ounces_to_grams": lambda v: v * 28.3495,
          },
      }

      key = f"{args['from_unit']}_to_{args['to_unit']}"
      fn = conversions.get(args["unit_type"], {}).get(key)

      if not fn:
          return {
              "content": [
                  {
                      "type": "text",
                      "text": f"Unsupported conversion: {args['from_unit']} to {args['to_unit']}",
                  }
              ],
              "is_error": True,
          }

      result = fn(args["value"])
      return {
          "content": [
              {
                  "type": "text",
                  "text": f"{args['value']} {args['from_unit']} = {result:.4f} {args['to_unit']}",
              }
          ]
      }


  converter_server = create_sdk_mcp_server(
      name="converter",
      version="1.0.0",
      tools=[convert_units],
  )
  ```

  ```typescript TypeScript theme={null}
  import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
  import { z } from "zod";

  const convert = tool(
    "convert_units",
    "Convert a value from one unit to another",
    {
      unit_type: z.enum(["length", "temperature", "weight"]).describe("Category of unit"),
      from_unit: z
        .string()
        .describe("Unit to convert from, e.g. kilometers, fahrenheit, pounds"),
      to_unit: z.string().describe("Unit to convert to"),
      value: z.number().describe("Value to convert")
    },
    async (args) => {
      type Conversions = Record<string, Record<string, (v: number) => number>>;

      const conversions: Conversions = {
        length: {
          kilometers_to_miles: (v) => v * 0.621371,
          miles_to_kilometers: (v) => v * 1.60934,
          meters_to_feet: (v) => v * 3.28084,
          feet_to_meters: (v) => v * 0.3048
        },
        temperature: {
          celsius_to_fahrenheit: (v) => (v * 9) / 5 + 32,
          fahrenheit_to_celsius: (v) => ((v - 32) * 5) / 9,
          celsius_to_kelvin: (v) => v + 273.15,
          kelvin_to_celsius: (v) => v - 273.15
        },
        weight: {
          kilograms_to_pounds: (v) => v * 2.20462,
          pounds_to_kilograms: (v) => v * 0.453592,
          grams_to_ounces: (v) => v * 0.035274,
          ounces_to_grams: (v) => v * 28.3495
        }
      };

      const key = `${args.from_unit}_to_${args.to_unit}`;
      const fn = conversions[args.unit_type]?.[key];

      if (!fn) {
        return {
          content: [
            {
              type: "text",
              text: `Unsupported conversion: ${args.from_unit} to ${args.to_unit}`
            }
          ],
          isError: true
        };
      }

      const result = fn(args.value);
      return {
        content: [
          {
            type: "text",
            text: `${args.value} ${args.from_unit} = ${result.toFixed(4)} ${args.to_unit}`
          }
        ]
      };
    }
  );

  const converterServer = createSdkMcpServer({
    name: "converter",
    version: "1.0.0",
    tools: [convert]
  });
  ```
</CodeGroup>

서버가 정의되면 날씨 예제와 동일한 방식으로 `query`에 전달하세요. 이 예제는 루프에서 3개의 서로 다른 프롬프트를 보내 동일한 도구가 서로 다른 단위 유형을 처리하는 모습을 보여줍니다. 각 응답에 대해 해당 턴 동안 Claude가 수행한 도구 호출이 포함된 `AssistantMessage` 객체를 검사하고 각 `ToolUseBlock`을 출력한 후 최종 `ResultMessage` 텍스트를 출력합니다. 이를 통해 Claude가 자체 지식으로 답변하는 대신 언제 도구를 사용하는지 확인할 수 있습니다.

[도구 검색](/docs/en/agent-sdk/tool-search)이 기본적으로 켜져 있으므로 Claude가 지연된 도구 스키마를 로드함에 따라 출력에 `ToolSearch` 호출이 포함될 수도 있습니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import (
      query,
      ClaudeAgentOptions,
      ResultMessage,
      AssistantMessage,
      ToolUseBlock,
  )


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={"converter": converter_server},
          allowed_tools=["mcp__converter__convert_units"],
      )

      prompts = [
          "Convert 100 kilometers to miles.",
          "What is 72°F in Celsius?",
          "How many pounds is 5 kilograms?",
      ]

      for prompt in prompts:
          try:
              async for message in query(prompt=prompt, options=options):
                  if isinstance(message, AssistantMessage):
                      for block in message.content:
                          if isinstance(block, ToolUseBlock):
                              print(f"[tool call] {block.name}({block.input})")
                  elif isinstance(message, ResultMessage) and message.subtype == "success":
                      print(f"Q: {prompt}\nA: {message.result}\n")
          except Exception as error:
              # 단발성 query()는 오류 결과를 생성한 후 예외를 발생시킵니다.
              # 성공 결과만 위에 출력되므로 실패를 여기서 처리하고 다음 프롬프트로 진행합니다.
              print(f"Call failed: {error}")


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const prompts = [
    "Convert 100 kilometers to miles.",
    "What is 72°F in Celsius?",
    "How many pounds is 5 kilograms?"
  ];

  for (const prompt of prompts) {
    try {
      for await (const message of query({
        prompt,
        options: {
          mcpServers: { converter: converterServer },
          allowedTools: ["mcp__converter__convert_units"]
        }
      })) {
        if (message.type === "assistant") {
          for (const block of message.message.content) {
            if (block.type === "tool_use") {
              console.log(`[tool call] ${block.name}`, block.input);
            }
          }
        } else if (message.type === "result" && message.subtype === "success") {
          console.log(`Q: ${prompt}\nA: ${message.result}\n`);
        }
      }
    } catch (error) {
      // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
      // 성공 결과만 위에 로깅되므로 실패를 여기서 처리하고 다음 프롬프트로 진행합니다.
      console.error(`Call failed: ${error}`);
    }
  }
  ```
</CodeGroup>

## 다음 단계

커스텀 도구는 비동기 함수를 표준 인터페이스로 감쌉니다. 이 페이지의 패턴을 동일한 서버 내에서 혼합할 수 있습니다. 단일 서버에 데이터베이스 도구, API 게이트웨이 도구, 이미지 렌더러를 나란히 배치할 수 있습니다.

여기서부터:

* 서버에 수십 개의 도구가 생성되는 경우, Claude가 필요로 할 때까지 로딩을 지연시키는 [도구 검색](/docs/en/agent-sdk/tool-search)을 참조하세요.
* 직접 구축하는 대신 외부 MCP 서버(파일 시스템, GitHub, Slack)에 연결하려면 [MCP 서버 연결하기](/docs/en/agent-sdk/mcp)를 참조하세요.
* 자동으로 실행되는 도구와 승인이 필요한 도구를 제어하려면 [권한 구성하기](/docs/en/agent-sdk/permissions)를 참조하세요.

## 관련 문서

* [TypeScript SDK 참조](/docs/en/agent-sdk/typescript)
* [Python SDK 참조](/docs/en/agent-sdk/python)
* [MCP 문서](https://modelcontextprotocol.io)
* [SDK 개요](/docs/en/agent-sdk/overview)
