> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 실시간 응답 스트리밍 (Stream responses in real-time)

> 텍스트 및 도구 호출이 스트리밍될 때 Agent SDK로부터 실시간 응답을 받습니다.

기본적으로 Agent SDK는 Claude가 각 응답 생성을 마친 후 완전한 `AssistantMessage` 객체를 생성(yield)합니다. 텍스트 및 도구 호출이 생성될 때 점진적 업데이트를 받으려면 옵션에서 `include_partial_messages` (Python) 또는 `includePartialMessages` (TypeScript)를 `true`로 설정하여 부분 메시지 스트리밍을 활성화하세요.

<Tip>
  이 페이지에서는 출력 스트리밍(실시간으로 토큰 수신)을 다룹니다. 입력 모드(메시지 전송 방식)에 대해서는 [에이전트에 메시지 전송](/docs/en/agent-sdk/streaming-vs-single-mode)을 참조하세요. 또한 [CLI를 통해 Agent SDK를 사용하여 응답을 스트리밍](/docs/en/headless)할 수도 있습니다.
</Tip>

## 스트리밍 출력 활성화

스트리밍을 활성화하려면 옵션에서 `include_partial_messages` (Python) 또는 `includePartialMessages` (TypeScript)를 `true`로 설정하세요. 이렇게 하면 SDK가 일반적인 `AssistantMessage` 및 `ResultMessage` 외에도 원시 API 이벤트가 포함된 `StreamEvent` 메시지를 도착하는 대로 출력합니다.

그러면 코드에서 다음을 수행해야 합니다:

1. 각 메시지의 유형을 확인하여 `StreamEvent`를 다른 메시지 유형과 구별합니다.
2. `StreamEvent`의 경우 `event` 필드를 추출하고 해당 `type`을 확인합니다.
3. 실제 텍스트 청크가 포함된 `delta.type`이 `text_delta`인 `content_block_delta` 이벤트를 찾습니다.

아래 예시는 스트리밍을 활성화하고 텍스트 청크가 도착하는 대로 출력합니다. 중첩된 유형 확인을 확인하세요. 먼저 `StreamEvent`, 그 다음 `content_block_delta`, 그 다음 `text_delta`입니다:

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions
  from claude_agent_sdk.types import StreamEvent
  import asyncio


  async def stream_response():
      options = ClaudeAgentOptions(
          include_partial_messages=True,
          allowed_tools=["Bash", "Read"],
      )

      async for message in query(prompt="List the files in my project", options=options):
          if isinstance(message, StreamEvent):
              event = message.event
              if event.get("type") == "content_block_delta":
                  delta = event.get("delta", {})
                  if delta.get("type") == "text_delta":
                      print(delta.get("text", ""), end="", flush=True)


  asyncio.run(stream_response())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "List the files in my project",
    options: {
      includePartialMessages: true,
      allowedTools: ["Bash", "Read"]
    }
  })) {
    if (message.type === "stream_event") {
      const event = message.event;
      if (event.type === "content_block_delta") {
        if (event.delta.type === "text_delta") {
          process.stdout.write(event.delta.text);
        }
      }
    }
  }
  ```
</CodeGroup>

## StreamEvent 레퍼런스

부분 메시지가 활성화되면 객체로 래핑된 원시 Claude API 스트리밍 이벤트를 받습니다. 해당 유형은 각 SDK에서 이름이 다릅니다:

* **Python**: `StreamEvent` (`claude_agent_sdk.types`에서 가져옴)
* **TypeScript**: `type: 'stream_event'`를 갖는 `SDKPartialAssistantMessage`

둘 다 누적된 텍스트가 아닌 원시 Claude API 이벤트를 포함합니다. 텍스트 델타를 직접 추출하고 누적해야 합니다. 각 유형의 구조는 다음과 같습니다:

<CodeGroup>
  ```python Python theme={null}
  @dataclass
  class StreamEvent:
      uuid: str  # 이 이벤트의 고유 식별자
      session_id: str  # 세션 식별자
      event: dict[str, Any]  # 원시 Claude API 스트림 이벤트
      parent_tool_use_id: str | None  # 항상 None
  ```

  ```typescript TypeScript theme={null}
  type SDKPartialAssistantMessage = {
    type: "stream_event";
    event: BetaRawMessageStreamEvent; // Anthropic SDK 출처
    parent_tool_use_id: string | null;
    uuid: UUID;
    session_id: string;
    ttft_ms?: number; // 첫 번째 토큰까지 걸린 시간(ms), message_start 이벤트에만 존재
  };
  ```
</CodeGroup>

`parent_tool_use_id` 필드는 Python에서 항상 `None`, TypeScript에서 `null`입니다. 스트림 이벤트는 메인 세션에 대해서만 출력되며 서브에이전트의 토큰 수준 델타는 전달되지 않습니다. 출력을 서브에이전트에 귀속시키려면 `parent_tool_use_id`를 전달하는 전체 메시지를 사용하세요. [서브에이전트 호출 감지](/docs/en/agent-sdk/subagents#detect-subagent-invocation)를 참조하세요.

`event` 필드에는 [Claude API](https://platform.claude.com/docs/en/build-with-claude/streaming#event-types)의 원시 스트리밍 이벤트가 포함됩니다. 일반적인 이벤트 유형은 다음과 같습니다:

| 이벤트 유형 (Event Type) | 설명                                         |
| :-------------------- | :------------------------------------------- |
| `message_start`       | 새 메시지 시작                               |
| `content_block_start` | 새 콘텐츠 블록 시작(텍스트 또는 도구 사용)     |
| `content_block_delta` | 콘텐츠의 점진적 업데이트                       |
| `content_block_stop`  | 콘텐츠 블록 종료                             |
| `message_delta`       | 메시지 수준 업데이트 (종료 이유, 사용량)      |
| `message_stop`        | 메시지 종료                                 |

## 메시지 흐름 (Message Flow)

부분 메시지가 활성화되면 메시지가 다음 순서로 수신됩니다:

```text theme={null}
StreamEvent (message_start)
StreamEvent (content_block_start) - 텍스트 블록
StreamEvent (content_block_delta) - 텍스트 청크들...
StreamEvent (content_block_stop)
StreamEvent (content_block_start) - tool_use 블록
StreamEvent (content_block_delta) - 도구 입력 청크들...
StreamEvent (content_block_stop)
StreamEvent (message_delta)
StreamEvent (message_stop)
AssistantMessage - 모든 콘텐츠가 포함된 완전한 메시지
... 도구 실행 ...
... 다음 차례를 위한 추가 스트리밍 이벤트들 ...
ResultMessage - 최종 결과
```

부분 메시지가 활성화되지 않은 경우(Python의 `include_partial_messages`, TypeScript의 `includePartialMessages`), `StreamEvent`를 제외한 모든 메시지 유형을 받습니다. 일반적인 유형에는 `SystemMessage` (세션 초기화), `AssistantMessage` (완전한 응답), `ResultMessage` (최종 결과) 및 대화 기록이 압축된 시점을 나타내는 콤팩트 바운더리 메시지(TypeScript의 `SDKCompactBoundaryMessage`, Python의 서브타입이 `"compact_boundary"`인 `SystemMessage`)가 포함됩니다.

## 텍스트 응답 스트리밍

생성되는 텍스트를 즉시 표시하려면 `delta.type`이 `text_delta`인 `content_block_delta` 이벤트를 찾으세요. 이 이벤트에는 점진적 텍스트 청크가 포함되어 있습니다. 아래 예시는 각 청크가 도착하는 대로 출력합니다:

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions
  from claude_agent_sdk.types import StreamEvent
  import asyncio


  async def stream_text():
      options = ClaudeAgentOptions(include_partial_messages=True)

      async for message in query(prompt="Explain how databases work", options=options):
          if isinstance(message, StreamEvent):
              event = message.event
              if event.get("type") == "content_block_delta":
                  delta = event.get("delta", {})
                  if delta.get("type") == "text_delta":
                      # 각 텍스트 청크가 도착하는 대로 출력
                      print(delta.get("text", ""), end="", flush=True)

      print()  # 최종 줄바꿈


  asyncio.run(stream_text())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Explain how databases work",
    options: { includePartialMessages: true }
  })) {
    if (message.type === "stream_event") {
      const event = message.event;
      if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
        process.stdout.write(event.delta.text);
      }
    }
  }

  console.log(); // 최종 줄바꿈
  ```
</CodeGroup>

## 도구 호출 스트리밍

도구 호출도 점진적으로 스트리밍됩니다. 도구가 시작되는 시점을 추적하고, 입력이 생성되는 대로 수신하며, 완료 시점을 확인할 수 있습니다. 아래 예시는 호출되는 현재 도구를 추적하고 스트리밍되는 JSON 입력을 누적합니다. 세 가지 이벤트 유형을 사용합니다:

* `content_block_start`: 도구 시작
* `input_json_delta`가 포함된 `content_block_delta`: 입력 청크 도착
* `content_block_stop`: 도구 호출 완료

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions
  from claude_agent_sdk.types import StreamEvent
  import asyncio


  async def stream_tool_calls():
      options = ClaudeAgentOptions(
          include_partial_messages=True,
          allowed_tools=["Read", "Bash"],
      )

      # 현재 도구를 추적하고 입력 JSON 누적
      current_tool = None
      tool_input = ""

      async for message in query(prompt="Read the README.md file", options=options):
          if isinstance(message, StreamEvent):
              event = message.event
              event_type = event.get("type")

              if event_type == "content_block_start":
                  # 새 도구 호출 시작
                  content_block = event.get("content_block", {})
                  if content_block.get("type") == "tool_use":
                      current_tool = content_block.get("name")
                      tool_input = ""
                      print(f"Starting tool: {current_tool}")

              elif event_type == "content_block_delta":
                  delta = event.get("delta", {})
                  if delta.get("type") == "input_json_delta":
                      # 입력되는 JSON을 스트리밍에 따라 누적
                      chunk = delta.get("partial_json", "")
                      tool_input += chunk
                      print(f"  Input chunk: {chunk}")

              elif event_type == "content_block_stop":
                  # 도구 호출 완료 - 최종 입력 표시
                  if current_tool:
                      print(f"Tool {current_tool} called with: {tool_input}")
                      current_tool = None


  asyncio.run(stream_tool_calls())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 현재 도구를 추적하고 입력 JSON 누적
  let currentTool: string | null = null;
  let toolInput = "";

  for await (const message of query({
    prompt: "Read the README.md file",
    options: {
      includePartialMessages: true,
      allowedTools: ["Read", "Bash"]
    }
  })) {
    if (message.type === "stream_event") {
      const event = message.event;

      if (event.type === "content_block_start") {
        // 새 도구 호출 시작
        if (event.content_block.type === "tool_use") {
          currentTool = event.content_block.name;
          toolInput = "";
          console.log(`Starting tool: ${currentTool}`);
        }
      } else if (event.type === "content_block_delta") {
        if (event.delta.type === "input_json_delta") {
          // 입력되는 JSON을 스트리밍에 따라 누적
          const chunk = event.delta.partial_json;
          toolInput += chunk;
          console.log(`  Input chunk: ${chunk}`);
        }
      } else if (event.type === "content_block_stop") {
        // 도구 호출 완료 - 최종 입력 표시
        if (currentTool) {
          console.log(`Tool ${currentTool} called with: ${toolInput}`);
          currentTool = null;
        }
      }
    }
  }
  ```
</CodeGroup>

## 스트리밍 UI 구축하기

이 예시는 텍스트 및 도구 스트리밍을 결합하여 조화로운 UI를 구축합니다. 에이전트가 현재 도구를 실행 중인지 여부(`in_tool` 플래그 사용)를 추적하여 도구가 실행되는 동안 `[Using Read...]`와 같은 상태 표시기를 표시합니다. 도구 실행 중이 아닐 때는 텍스트가 정상적으로 스트리밍되며, 도구가 완료되면 "done" 메시지가 출력됩니다. 이 패턴은 다단계 에이전트 작업 동안 진행 상황을 표시해야 하는 채팅 인터페이스에 유용합니다.

<CodeGroup>
  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage
  from claude_agent_sdk.types import StreamEvent
  import asyncio
  import sys


  async def streaming_ui():
      options = ClaudeAgentOptions(
          include_partial_messages=True,
          allowed_tools=["Read", "Bash", "Grep"],
      )

      # 현재 도구 호출 중인지 여부 추적
      in_tool = False

      async for message in query(
          prompt="Find all TODO comments in the codebase", options=options
      ):
          if isinstance(message, StreamEvent):
              event = message.event
              event_type = event.get("type")

              if event_type == "content_block_start":
                  content_block = event.get("content_block", {})
                  if content_block.get("type") == "tool_use":
                      # 도구 호출 시작 - 상태 표시기 출력
                      tool_name = content_block.get("name")
                      print(f"\n[Using {tool_name}...]", end="", flush=True)
                      in_tool = True

              elif event_type == "content_block_delta":
                  delta = event.get("delta", {})
                  # 도구를 실행하지 않을 때만 텍스트 스트리밍
                  if delta.get("type") == "text_delta" and not in_tool:
                      sys.stdout.write(delta.get("text", ""))
                      sys.stdout.flush()

              elif event_type == "content_block_stop":
                  if in_tool:
                      # 도구 호출 종료
                      print(" done", flush=True)
                      in_tool = False

          elif isinstance(message, ResultMessage):
              # 에력전트가 모든 작업을 완료함
              print(f"\n\n--- Complete ---")


  asyncio.run(streaming_ui())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 현재 도구 호출 중인지 여부 추적
  let inTool = false;

  for await (const message of query({
    prompt: "Find all TODO comments in the codebase",
    options: {
      includePartialMessages: true,
      allowedTools: ["Read", "Bash", "Grep"]
    }
  })) {
    if (message.type === "stream_event") {
      const event = message.event;

      if (event.type === "content_block_start") {
        if (event.content_block.type === "tool_use") {
          // 도구 호출 시작 - 상태 표시기 출력
          process.stdout.write(`\n[Using ${event.content_block.name}...]`);
          inTool = true;
        }
      } else if (event.type === "content_block_delta") {
        // 도구를 실행하지 않을 때만 텍스트 스트리밍
        if (event.delta.type === "text_delta" && !inTool) {
          process.stdout.write(event.delta.text);
        }
      } else if (event.type === "content_block_stop") {
        if (inTool) {
          // 도구 호출 종료
          console.log(" done");
          inTool = false;
        }
      }
    } else if (message.type === "result") {
      // 에이전트가 모든 작업을 완료함
      console.log("\n\n--- Complete ---");
    }
  }
  ```
</CodeGroup>

## 알려진 제한 사항

* **구조화된 출력**: JSON 결과는 스트리밍 델타가 아니라 최종 `ResultMessage.structured_output`에만 나타납니다. 자세한 내용은 [구조화된 출력](/docs/en/agent-sdk/structured-outputs)을 참조하세요.

## 다음 단계

이제 실시간으로 텍스트 및 도구 호출을 스트리밍할 수 있으므로 다음 관련 항목을 살펴보세요:

* [대화형 vs 단일 실행 쿼리](/docs/en/agent-sdk/streaming-vs-single-mode): 유스케이스에 맞는 입력 모드 선택
* [구조화된 출력](/docs/en/agent-sdk/structured-outputs): 에이전트로부터 타입이 지정된 JSON 응답 수신
* [권한](/docs/en/agent-sdk/permissions): 에이전트가 사용할 수 있는 도구 제어
