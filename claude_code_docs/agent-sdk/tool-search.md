> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 도구 검색을 통한 다수의 도구 확장 (Scale to many tools with tool search)

> 필요한 도구만 온디맨드로 탐색하고 로드하여 에이전트를 수천 개의 도구로 확장합니다.

도구 검색(Tool search)은 필요한 도구를 동적으로 탐색하고 온디맨드로 로드하여 에이전트가 수백 또는 수천 개의 도구와 작업할 수 있도록 합니다. 모든 도구 정의를 사전에 컨텍스트 창에 로드하는 대신, 에이전트는 도구 카탈로그를 검색하여 필요한 도구만 로드합니다.

이 접근 방식은 도구 라이브러리가 확장됨에 따라 발생하는 두 가지 문제를 해결합니다:

* **컨텍스트 효율성:** 도구 정의가 컨텍스트 창의 큰 부분(50개 도구가 10~20K 토큰 소비 가능)을 차지하여 실제 작업을 위한 공간이 줄어듭니다.
* **도구 선택 정확도:** 한 번에 30~50개 이상의 도구가 로드되면 도구 선택 정확도가 저하됩니다.

도구 검색은 기본적으로 활성화되어 있습니다.

## 도구 검색 작동 방식

도구 검색이 활성화되면 도구 정의가 컨텍스트 창에서 보류됩니다. 에이전트는 사용 가능한 도구의 요약을 받고, 작업에 아직 로드되지 않은 기능이 필요할 때 관련 도구를 검색합니다. 기본적으로 가장 관련성이 높은 도구가 최대 5개까지 컨텍스트에 로드되어 이후 차례 동안 사용할 수 있는 상태로 유지됩니다. SDK가 이전 메시지를 압축하여 공간을 확보할 만큼 대화가 길어지면 이전에 탐색된 도구가 제거될 수 있으며 에이전트는 필요에 따라 다시 검색합니다.

도구 검색은 Claude가 도구를 처음 탐색할 때(검색 단계) 왕복 1회를 추가하지만, 대규모 도구 세트의 경우 매 차례마다 컨텍스트가 작아져서 이것이 상쇄됩니다. 약 10개 미만의 도구를 사용하는 경우 사전에 모든 것을 로드하는 것이 일반적으로 더 빠릅니다.

기본 API 메커니즘에 대한 자세한 내용은 [API의 도구 검색](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool)을 참조하세요.

<Note>
  도구 검색은 Claude Sonnet 4.5, Claude Haiku 4.5, Claude Opus 4.5 및 이후 모델에서 지원됩니다. 현재 목록은 [API 문서의 모델 호환성](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool#model-compatibility)을 참조하세요. Google Cloud의 Agent Platform에서는 최소 지원 모델이 Claude Sonnet 4.5 및 Claude Opus 4.5입니다.
</Note>

## 도구 검색 구성

도구 검색은 기본적으로 활성화되어 있습니다. Google Cloud의 Agent Platform에서는 기본적으로 비활성화되어 있으며, Claude Sonnet 4.5 이상 및 Claude Opus 4.5 이상에서 지원됩니다. 또한 대부분의 프록시가 `tool_reference` 블록을 전달하지 않기 때문에 `ANTHROPIC_BASE_URL`이 퍼스트 파티가 아닌 호스트를 가리킬 때도 비활성화됩니다. `ENABLE_TOOL_SEARCH` 환경 변수를 사용하여 어느 기본값이든 재정의할 수 있습니다:

| 값       | 동작                                                                                                                                                                                                                                                                     |
| :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| (미설정) | 도구 검색이 켜집니다. 도구 정의가 지연되고 온디맨드로 탐색됩니다. Google Cloud의 Agent Platform 또는 퍼스트 파티가 아닌 `ANTHROPIC_BASE_URL`에서는 사전에 로드하는 방식으로 폴백합니다.                                                                               |
| `true`   | 도구 검색이 항상 켜집니다. SDK는 Google Cloud의 Agent Platform 및 프록시를 통할 때도 베타 헤더를 전송합니다. Sonnet 4.5 또는 Opus 4.5 이전의 Google Cloud Agent Platform 모델이나 `tool_reference` 블록을 지원하지 않는 프록시에서는 요청이 실패합니다.                   |
| `auto`   | 모든 도구 정의의 결합된 토큰 수를 모델의 컨텍스트 창과 비교합니다. 10%를 초과하면 도구 검색이 활성화됩니다. 10% 미만이면 모든 도구가 컨텍스트에 정상적으로 로드됩니다.                                                                                                   |
| `auto:N` | 사용자 지정 백분율을 사용하는 `auto`와 동일합니다. `auto:5`는 도구 정의가 컨텍스트 창의 5%를 초과할 때 활성화됩니다. 값이 낮을수록 더 일찍 활성화됩니다.                                                                                                                 |
| `false`  | 도구 검색이 꺼집니다. 모든 도구 정의가 매 차례마다 컨텍스트에 로드됩니다.                                                                                                                                                                                                |

[`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`](/docs/en/env-vars)를 설정하면 도구 검색이 계속 꺼진 상태로 유지되며 `ENABLE_TOOL_SEARCH`로 이를 재정의할 수 없습니다. 이 변수는 `defer_loading` 도구 정의 및 `tool_reference` 콘텐츠 블록에 필요한 베타 헤더를 제거합니다.

도구 검색은 원격 MCP 서버에서 오든 [커스텀 SDK MCP 서버](/docs/en/agent-sdk/custom-tools)에서 오든 등록된 모든 도구에 적용됩니다. `auto`를 사용할 때 임계값은 모든 서버에 걸친 모든 도구 정의의 결합된 크기를 기준으로 합니다.

`query()`의 `env` 옵션에서 값을 설정하세요. TypeScript에서 `env`는 하위 프로세스 환경을 교체하므로 `...process.env`를 전개하여 상속된 변수를 유지하세요. Python에서 `env`는 상속된 환경 위에 병합됩니다. 이 예시는 많은 도구를 노출하는 원격 MCP 서버에 연결하고, 와일드카드로 모든 도구를 사전 승인하며, 정의가 컨텍스트 창의 5%를 초과할 때 도구 검색이 활성화되도록 `auto:5`를 사용합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  try {
    for await (const message of query({
      prompt: "Find and run the appropriate database query",
      options: {
        mcpServers: {
          "enterprise-tools": {
            // 원격 MCP 서버에 연결
            type: "http",
            url: "https://tools.example.com/mcp"
          }
        },
        allowedTools: ["mcp__enterprise-tools__*"], // 와일드카드로 이 서버의 모든 도구 사전 승인
        env: {
          ...process.env, // env는 하위 프로세스 환경을 교체하므로 상속된 변수 유지
          ENABLE_TOOL_SEARCH: "auto:5" // 도구가 컨텍스트의 5%를 초과할 때 도구 검색 활성화
        }
      }
    })) {
      if (message.type === "result" && message.subtype === "success") {
        console.log(message.result);
      }
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과 반환 후 에러를 발생시킵니다
    console.log(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      options = ClaudeAgentOptions(
          mcp_servers={
              "enterprise-tools": {
                  "type": "http",
                  "url": "https://tools.example.com/mcp",
              }
          },
          allowed_tools=[
              "mcp__enterprise-tools__*"
          ],  # 와일드카드로 이 서버의 모든 도구 사전 승인
          env={
              "ENABLE_TOOL_SEARCH": "auto:5"  # 도구가 컨텍스트의 5%를 초과할 때 도구 검색 활성화
          },
      )

      try:
          async for message in query(
              prompt="Find and run the appropriate database query",
              options=options,
          ):
              if isinstance(message, ResultMessage) and message.subtype == "success":
                  print(message.result)
      except Exception as error:
          # 단일 실행 query()는 오류 결과 반환 후 에러를 발생시킵니다
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

이 예시를 실행하려면 `https://tools.example.com/mcp`를 자체 MCP 서버의 URL로 교체하세요. 성공하면 결과 텍스트가 콘솔에 출력됩니다.

단일 실행 `query()` 호출이므로 SDK는 오류 결과를 반환한 후 에러를 발생시키므로 예시는 루프를 try 블록으로 감쌉니다. 실행이 실패한 이유를 확인하려면 루프 내부에서 결과 메시지의 `subtype`(예: `error_during_execution`)을 검사하세요. 결과 메시지에 대한 자세한 내용은 [결과 처리](/docs/en/agent-sdk/agent-loop#handle-the-result)를 참조하세요.

`ENABLE_TOOL_SEARCH`를 `"false"`로 설정하면 도구 검색이 비활성화되고 매 차례마다 모든 도구 정의가 컨텍스트에 로드됩니다. 이렇게 하면 검색 왕복이 제거되므로 도구 세트가 작고(~10개 미만) 정의가 컨텍스트 창에 여유롭게 들어갈 때 더 빠를 수 있습니다.

## 도구 탐색 최적화

검색 메커니즘은 쿼리를 도구 이름 및 설명과 매칭합니다. `search_slack_messages`와 같은 이름은 `query_slack`보다 더 넓은 범위의 요청에 대해 노출됩니다. 구체적인 키워드가 포함된 설명("Search Slack messages by keyword, channel, or date range")은 일반적인 설명("Query Slack")보다 더 많은 쿼리와 매칭됩니다.

사용 가능한 도구 범주를 나열하는 시스템 프롬프트 섹션을 추가할 수도 있습니다. 이렇게 하면 에이전트에 검색할 도구 종류에 대한 컨텍스트가 제공됩니다. TypeScript의 `systemPrompt` 옵션 또는 Python의 `system_prompt`를 통해 텍스트를 전달하고, 프레셋의 프롬프트에 텍스트를 대체하는 대신 추가하는 `append`와 함께 `claude_code` 프레셋을 사용하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: "You can search for tools to interact with Slack, GitHub, and Jira."
    }
  }
  ```

  ```python Python theme={null}
  options = ClaudeAgentOptions(
      system_prompt={
          "type": "preset",
          "preset": "claude_code",
          "append": "You can search for tools to interact with Slack, GitHub, and Jira.",
      }
  )
  ```
</CodeGroup>

시스템 프롬프트 옵션의 전체 세트는 [시스템 프롬프트 수정](/docs/en/agent-sdk/modifying-system-prompts)을 참조하세요.

## 제한 사항

* **최대 도구 수:** 카탈로그에 10,000개 도구
* **검색 결과:** 기본적으로 검색당 가장 관련성이 높은 도구를 최대 5개 반환
* **모델 지원:** Claude Sonnet 4.5, Claude Haiku 4.5, Claude Opus 4.5 및 이후 모델; 현재 목록은 [API 문서의 모델 호환성](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool#model-compatibility)을 참조하세요. Google Cloud의 Agent Platform에서는 Claude Sonnet 4.5 이상 및 Claude Opus 4.5 이상.

## 관련 문서

* [API의 도구 검색](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool): 커스텀 구현을 포함한 도구 검색에 대한 전체 API 문서
* [MCP 서버 연결](/docs/en/agent-sdk/mcp): MCP 서버를 통해 외부 도구에 연결
* [커스텀 도구](/docs/en/agent-sdk/custom-tools): SDK MCP 서버로 자체 도구 구축
* [TypeScript SDK 레퍼런스](/docs/en/agent-sdk/typescript): 전체 API 레퍼런스
* [Python SDK 레퍼런스](/docs/en/agent-sdk/python): 전체 API 레퍼런스
