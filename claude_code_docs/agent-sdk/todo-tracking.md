> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 할 일 목록 (Todo Lists)

> 체계적인 작업 관리를 위해 Claude Agent SDK를 사용하여 할 일을 추적하고 표시합니다.

할 일 추적(Todo tracking)은 작업을 관리하고 진행 상황을 사용자에게 표시하는 구조화된 방법을 제공합니다. Claude Agent SDK에는 복잡한 워크플로우를 정리하고 작업 진행 상황을 사용자에게 계속 알리는 데 도움이 되는 내장 할 일 기능이 포함되어 있습니다.

<Note>
  TypeScript Agent SDK 0.3.142 및 Claude Code v2.1.142부터 세션은 `TodoWrite` 대신 구조화된 Task 도구인 `TaskCreate`, `TaskUpdate`, `TaskGet`, `TaskList`를 사용합니다. Python SDK는 Python 패키지 버전이 아니라 시작하는 Claude Code CLI에서 이 변경 사항을 가져옵니다. pip 패키지 내부에 번들로 제공되는 복사본 또는 `cli_path`로 가리키는 CLI가 v2.1.142 이상이 되면 이 전환이 적용됩니다. 모니터링 코드가 어떻게 변경되는지는 [Task 도구로 마이그레이션](#migrate-to-task-tools)을 참조하세요. 이 페이지의 예시는 아직 마이그레이션되지 않은 세션에 대해 `TodoWrite`를 계속 표시하도록 `CLAUDE_CODE_ENABLE_TASKS=0`을 설정합니다.
</Note>

### 할 일 생애주기 (Todo Lifecycle)

할 일은 예측 가능한 생애주기를 따릅니다:

1. **생성(Created)**: 작업이 식별되면 `pending`으로 생성됨
2. **활성화(Activated)**: 작업이 시작되면 `in_progress`로 변경됨
3. **완료(Completed)**: 작업이 성공적으로 완료되면 `completed`로 변경됨
4. **제거(Removed)**: 그룹의 모든 작업이 완료되면 제거됨

### 할 일이 사용되는 경우

SDK는 다음과 같은 대부분의 다단계 작업에 대해 할 일을 생성합니다:

* 3개 이상의 고유한 작업을 요구하는 **복잡한 다단계 작업**
* 여러 항목이 언급된 **사용자 제공 작업 목록**
* 진행 상황 추적의 이점이 있는 **중요한 작업**
* 사용자가 할 일 정리를 요청하는 **명시적 요청**

매우 짧거나 단일 단계 요청의 경우 할 일을 건너뛸 수 있습니다.

## 예시 (Examples)

이 예시를 실행하기 전에 [빠른 시작](/docs/en/agent-sdk/quickstart)에 따라 Claude Agent SDK를 설치하세요.

각 예시는 에이전트가 완료되고 최종 결과 메시지를 출력할 때까지 실행됩니다. 세션이 먼저 차례 제한에 도달하면 해당 결과 메시지는 `error_max_turns` 서브타입을 갖습니다. 종료를 감지하려면 `subtype`을 확인하세요.

이 예시들은 단일 실행 `query()` 호출을 사용합니다. `error_max_turns` 결과를 반환한 후 `query()`는 `Reached maximum number of turns`가 포함된 에러를 발생시킵니다. 각 예시는 해당 상황이 발생할 때 깔끔하게 종료되도록 루프를 try 블록으로 감쌉니다.

결과 서브타입에 대해서는 [결과 처리](/docs/en/agent-sdk/agent-loop#handle-the-result)를 참조하세요.

### 할 일 변경 사항 모니터링

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  try {
    for await (const message of query({
      prompt: "Optimize my React app performance and track progress with todos",
      // 이 예시가 모니터링하는 TodoWrite를 다시 활성화합니다. 그렇지 않으면 SDK가
      // 대신 Task 도구를 사용하여 이 tool_use 블록이 전혀 나타나지 않습니다.
      options: { maxTurns: 15, env: { ...process.env, CLAUDE_CODE_ENABLE_TASKS: "0" } }
    })) {
      // 메시지 스트림에 반영되는 할 일 업데이트
      if (message.type === "assistant") {
        for (const block of message.message.content) {
          if (block.type === "tool_use" && block.name === "TodoWrite") {
            const todos = block.input.todos;

            console.log("Todo Status Update:");
            todos.forEach((todo, index) => {
              const status =
                todo.status === "completed" ? "✅" : todo.status === "in_progress" ? "🔧" : "❌";
              console.log(`${index + 1}. ${status} ${todo.content}`);
            });
          }
        }
      }
    }
  } catch (error) {
    // 단일 실행 query()는 maxTurns 제한에 도달했을 때와 같이
    // 오류 결과를 반환한 후 에러를 발생시킵니다.
    console.log(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio

  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ToolUseBlock


  async def main():
      try:
          async for message in query(
              prompt="Optimize my React app performance and track progress with todos",
              # 이 예시가 모니터링하는 TodoWrite를 다시 활성화합니다. 그렇지 않으면 SDK가
              # 대신 Task 도구를 사용하여 이 tool_use 블록이 전혀 나타나지 않습니다.
              options=ClaudeAgentOptions(max_turns=15, env={"CLAUDE_CODE_ENABLE_TASKS": "0"}),
          ):
              # 메시지 스트림에 반영되는 할 일 업데이트
              if isinstance(message, AssistantMessage):
                  for block in message.content:
                      if isinstance(block, ToolUseBlock) and block.name == "TodoWrite":
                          todos = block.input["todos"]

                          print("Todo Status Update:")
                          for i, todo in enumerate(todos):
                              status = (
                                  "✅"
                                  if todo["status"] == "completed"
                                  else "🔧"
                                  if todo["status"] == "in_progress"
                                  else "❌"
                              )
                              print(f"{i + 1}. {status} {todo['content']}")
      except Exception as error:
          # 단일 실행 query()는 max_turns 제한에 도달했을 때와 같이
          # 오류 결과를 반환한 후 에러를 발생시킵니다.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

### 실시간 진행 상황 표시

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  class TodoTracker {
    private todos: any[] = [];

    displayProgress() {
      if (this.todos.length === 0) return;

      const completed = this.todos.filter((t) => t.status === "completed").length;
      const inProgress = this.todos.filter((t) => t.status === "in_progress").length;
      const total = this.todos.length;

      console.log(`\nProgress: ${completed}/${total} completed`);
      console.log(`Currently working on: ${inProgress} task(s)\n`);

      this.todos.forEach((todo, index) => {
        const icon =
          todo.status === "completed" ? "✅" : todo.status === "in_progress" ? "🔧" : "❌";
        const text = todo.status === "in_progress" ? todo.activeForm : todo.content;
        console.log(`${index + 1}. ${icon} ${text}`);
      });
    }

    async trackQuery(prompt: string) {
      try {
        for await (const message of query({
          prompt,
          // 이 트래커가 감시하는 TodoWrite를 다시 활성화합니다.
          options: { maxTurns: 20, env: { ...process.env, CLAUDE_CODE_ENABLE_TASKS: "0" } }
        })) {
          if (message.type === "assistant") {
            for (const block of message.message.content) {
              if (block.type === "tool_use" && block.name === "TodoWrite") {
                this.todos = block.input.todos;
                this.displayProgress();
              }
            }
          }
        }
      } catch (error) {
        // 단일 실행 query()는 maxTurns 제한에 도달했을 때와 같이
        // 오류 결과를 반환한 후 에러를 발생시킵니다.
        console.log(`Session ended with an error: ${error}`);
      }
    }
  }

  // 사용법
  const tracker = new TodoTracker();
  await tracker.trackQuery("Build a complete authentication system with todos");
  ```

  ```python Python theme={null}
  import asyncio

  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ToolUseBlock
  from typing import List, Dict


  class TodoTracker:
      def __init__(self):
          self.todos: List[Dict] = []

      def display_progress(self):
          if not self.todos:
              return

          completed = len([t for t in self.todos if t["status"] == "completed"])
          in_progress = len([t for t in self.todos if t["status"] == "in_progress"])
          total = len(self.todos)

          print(f"\nProgress: {completed}/{total} completed")
          print(f"Currently working on: {in_progress} task(s)\n")

          for i, todo in enumerate(self.todos):
              icon = (
                  "✅"
                  if todo["status"] == "completed"
                  else "🔧"
                  if todo["status"] == "in_progress"
                  else "❌"
              )
              text = (
                  todo["activeForm"]
                  if todo["status"] == "in_progress"
                  else todo["content"]
              )
              print(f"{i + 1}. {icon} {text}")

      async def track_query(self, prompt: str):
          try:
              async for message in query(
                  prompt=prompt,
                  # 이 트래커가 감시하는 TodoWrite를 다시 활성화합니다.
                  options=ClaudeAgentOptions(max_turns=20, env={"CLAUDE_CODE_ENABLE_TASKS": "0"}),
              ):
                  if isinstance(message, AssistantMessage):
                      for block in message.content:
                          if isinstance(block, ToolUseBlock) and block.name == "TodoWrite":
                              self.todos = block.input["todos"]
                              self.display_progress()
          except Exception as error:
              # 단일 실행 query()는 max_turns 제한에 도달했을 때와 같이
              # 오류 결과를 반환한 후 에러를 발생시킵니다.
              print(f"Session ended with an error: {error}")


  # 사용법
  async def main():
      tracker = TodoTracker()
      await tracker.track_query("Build a complete authentication system with todos")


  asyncio.run(main())
  ```
</CodeGroup>

## Task 도구로 마이그레이션

Task 도구는 단일 `TodoWrite` 호출을 각각의 새 항목에 대한 `TaskCreate` 및 각 상태 변경에 대한 `TaskUpdate`로 분할하며, 모델이 현재 목록을 다시 읽을 수 있도록 `TaskList` 및 `TaskGet`을 사용할 수 있습니다. 모니터링 코드는 여전히 어시스턴트 스트림에서 `tool_use` 블록을 검사하지만 매 호출마다 전체 목록을 교체하는 대신 작업 ID를 키로 하는 맵을 유지합니다. Task 도구는 TypeScript Agent SDK 0.3.142 및 Claude Code v2.1.142부터 기본값이므로 별도의 `options.env` 변경이 필요하지 않습니다.

| `TodoWrite` 사용 시                           | Task 도구 사용 시                                                                                                                                                                                                                                                                                   |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 하나의 도구 호출이 전체 `todos` 배열을 다시 작성함 | `TaskCreate`가 하나의 항목을 추가하고, `TaskUpdate`가 `taskId`로 하나의 항목을 수정함                                                                                                                                                                                                              |
| `block.name === "TodoWrite"` 매칭             | `block.name === "TaskCreate"` 또는 `"TaskUpdate"` 매칭                                                                                                                                                                                                                                              |
| 항목 형상: `{ content, status, activeForm }`   | `TaskCreate` 입력: `{ subject, description, activeForm?, metadata? }`. `TaskUpdate` 입력: `{ taskId, status?, subject?, description?, activeForm?, addBlocks?, addBlockedBy?, owner?, metadata? }`. `status`는 `"pending"`, `"in_progress"`, `"completed"`; 삭제하려면 `status: "deleted"`로 설정 |
| `block.input.todos`를 직접 렌더링함            | 여러 호출에 걸쳐 항목을 누적하거나 `TaskList` 도구 결과에서 스냅샷을 읽음                                                                                                                                                                                                                           |

할당된 작업 ID는 `TaskCreate` 입력에 존재하지 않습니다. 일치하는 `tool_result`에 `{ task: { id, subject } }`로 돌아오므로 결과 블록에서 캡처하여 맵의 키로 지정하세요. 다음 예시는 [할 일 변경 사항 모니터링](#monitoring-todo-changes) 루프에 대한 최소한의 변경 사항을 보여줍니다. `tool_use` 입력만 읽고 `tool_result` 블록에서 ID 캡처는 건너뜁니다. 전체 목록을 렌더링하려면 스트림에서 `TaskList` 도구 결과를 관찰하거나 `TaskCreate` 결과 및 `TaskUpdate` 입력을 맵에 누적하세요.

스트리밍된 `tool_use` 입력은 모델이 출력한 원시 형상입니다. Claude Code는 실행 전에 유사하지만 잘못된 일부 키 이름을 수정하여 `id` 또는 `task_id`를 `taskId`로, `active_form`을 `activeForm`으로 매핑하지만 해당 수정 사항이 스트림에 반영되지는 않습니다. 아래 샘플처럼 정식 이름이 항상 존재한다고 가정하지 않고 `TaskUpdate` 입력 필드를 안전하게 읽으세요.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  try {
    for await (const message of query({
      prompt: "Optimize my React app performance and track progress with todos",
      options: { maxTurns: 15 },
    })) {
      if (message.type !== "assistant") continue;
      for (const block of message.message.content) {
        if (block.type !== "tool_use") continue;
        if (block.name === "TaskCreate") {
          const input = block.input as { subject: string };
          console.log(`+ ${input.subject}`);
        } else if (block.name === "TaskUpdate") {
          const input = block.input as {
            taskId?: string;
            id?: string;
            task_id?: string;
            status?: string;
          };
          const taskId = input.taskId ?? input.id ?? input.task_id;
          if (taskId && input.status) console.log(`  ${taskId} -> ${input.status}`);
        }
      }
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과 반환 후 에러를 발생시킵니다.
    console.log(`Session ended with an error: ${error}`);
  }
  ```

  ```python Python theme={null}
  import asyncio

  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ToolUseBlock

  async def main():
      try:
          async for message in query(
              prompt="Optimize my React app performance and track progress with todos",
              options=ClaudeAgentOptions(max_turns=15),
          ):
              if not isinstance(message, AssistantMessage):
                  continue
              for block in message.content:
                  if not isinstance(block, ToolUseBlock):
                      continue
                  if block.name == "TaskCreate":
                      print(f"+ {block.input['subject']}")
                  elif block.name == "TaskUpdate" and block.input.get("status"):
                      task_id = (
                          block.input.get("taskId")
                          or block.input.get("id")
                          or block.input.get("task_id")
                      )
                      if task_id:
                          print(f"  {task_id} -> {block.input['status']}")
      except Exception as error:
          # 단일 실행 query()는 오류 결과 반환 후 에러를 발생시킵니다.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

## 관련 문서

* [TypeScript SDK 레퍼런스](/docs/en/agent-sdk/typescript)
* [Python SDK 레퍼런스](/docs/en/agent-sdk/python)
* [스트리밍 vs 단일 모드](/docs/en/agent-sdk/streaming-vs-single-mode)
* [커스텀 도구](/docs/en/agent-sdk/custom-tools)
