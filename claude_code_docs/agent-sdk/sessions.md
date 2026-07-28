> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 세션 작업하기

> 세션이 에이전트 대화 기록을 보존하는 방식과 이전 실행으로 돌아가기 위해 continue, resume, fork를 사용하는 시점

세션은 에이전트가 작업하는 동안 SDK가 누적하는 대화 기록입니다. 프롬프트, 에이전트가 수행한 모든 도구 호출, 모든 도구 결과 및 모든 응답이 포함됩니다. SDK는 나중에 다시 돌아올 수 있도록 이를 디스크에 자동으로 기록합니다.

세션으로 돌아간다는 것은 에이전트가 이미 읽은 파일, 이미 수행한 분석, 이미 내린 결정 등 이전의 전체 컨텍스트를 유지하고 있음을 의미합니다. 후속 질문을 하거나, 중단된 작업을 복구하거나, 다른 접근 방식을 탐색하기 위해 분기(branch off)할 수 있습니다.

<Note>
  세션은 파일 시스템이 아니라 **대화**를 보존합니다. 에이전트가 생성한 파일 변경 사항을 스냅샷으로 저장하고 되돌리려면 [파일 체크포인팅](/docs/en/agent-sdk/file-checkpointing)을 사용하세요.
</Note>

이 가이드에서는 앱에 맞는 최선의 접근 방식을 선택하는 방법, 세션을 자동으로 추적하는 SDK 인터페이스, 세션 ID를 캡처하고 `resume` 및 `fork`를 수동으로 사용하는 방법, 호스트 간에 세션을 재개할 때 알아야 할 사항을 다룹니다.

## 접근 방식 선택

얼마만큼의 세션 처리가 필요한지는 애플리케이션의 형태에 따라 다릅니다. 컨텍스트를 공유해야 하는 여러 프롬프트를 보낼 때 세션 관리가 필요해집니다. 단일 `query()` 호출 내에서 에이전트는 이미 필요한 만큼 턴(turn)을 수행하며, 권한 프롬프트 및 `AskUserQuestion`은 [루프 내부에서 처리](/docs/en/agent-sdk/user-input)됩니다(호출이 종료되지 않음).

| 구축하려는 항목 | 사용할 기능 |
| :-------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 일회성 작업: 단일 프롬프트, 후속 작업 없음 | 추가 작업 필요 없음. 단일 `query()` 호출로 처리됨. |
| 단일 프로세스 내 다중 턴 대화 | [`ClaudeSDKClient` (Python) 또는 `continue: true` (TypeScript)](#automatic-session-management). SDK가 ID 처리 없이 세션을 자동으로 추적함. |
| 프로세스 재시작 후 중단된 위치에서 계속 진행 | `continue_conversation=True` (Python) / `continue: true` (TypeScript). ID 없이 디렉토리 내 가장 최근 세션을 재개함. |
| 특정 과거 세션 재개 (가장 최근 세션이 아닌 경우) | 세션 ID를 캡처하여 `resume`에 전달. |
| 원본을 잃지 않고 다른 접근 방식 시도 | 세션을 분기(fork)함. |
| 무상태(Stateless) 작업, 디스크 기록 원치 않음 (TypeScript 전용) | [`persistSession: false`](/docs/en/agent-sdk/typescript#options) 설정. 세션은 호출 기간 동안 메모리에만 존재함. Python은 항상 디스크에 저장함. |

### Continue, resume, fork 비교

Continue, resume, fork는 `query()`에서 설정하는 옵션 필드입니다 (Python의 경우 [`ClaudeAgentOptions`](/docs/en/agent-sdk/python#claudeagentoptions), TypeScript의 경우 [`Options`](/docs/en/agent-sdk/typescript#options)).

**Continue**와 **resume**은 모두 기존 세션을 가져와 내용을 추가합니다. 차이점은 해당 세션을 찾는 방식입니다:

* **Continue**는 현재 디렉토리에서 가장 최근 세션을 찾습니다. 별도로 추적할 필요가 없습니다. 앱이 한 번에 하나의 대화를 실행할 때 잘 맞습니다.
* **Resume**은 특정 세션 ID를 받습니다. ID를 직접 추적해야 합니다. 대화가 여러 개 있거나(예: 다중 사용자 앱의 사용자별 대화) 가장 최근 세션이 아닌 세션으로 돌아가고자 할 때 필요합니다.

**Fork**는 다릅니다: 원본 기록의 복사본으로 시작하는 새 세션을 생성합니다. 원본은 변경되지 않은 상태로 유지됩니다. 돌아갈 수 있는 여지를 남겨둔 채 다른 방향을 시도해 볼 때 fork를 사용하세요.

## 자동 세션 관리

두 SDK 모두 호출 간에 세션 상태를 자동으로 추적하는 인터페이스를 제공하므로 ID를 수동으로 전달할 필요가 없습니다. 단일 프로세스 내에서 다중 턴 대화를 진행할 때 사용하세요.

### Python: `ClaudeSDKClient`

[`ClaudeSDKClient`](/docs/en/agent-sdk/python#claudesdkclient)는 내부적으로 세션 ID를 처리합니다. `client.query()`를 호출할 때마다 동일한 세션이 자동으로 계속됩니다. [`client.receive_response()`](/docs/en/agent-sdk/python#claudesdkclient)를 호출하여 현재 쿼리의 메시지를 순회하세요. 연결 설정 및 해제가 자동으로 처리되도록 클라이언트를 비동기 컨텍스트 매니저로 사용하거나, `connect()` 및 `disconnect()`를 수동으로 호출하세요.

다음 예제는 동일한 `client`에 대해 두 개의 쿼리를 실행합니다. 첫 번째 쿼리는 에이전트에 모듈 분석을 요청하고, 두 번째 쿼리는 해당 모듈의 리팩토링을 요청합니다. 두 호출 모두 동일한 클라이언트 인스턴스를 거치므로, 두 번째 쿼리는 명시적인 `resume`이나 세션 ID 없이도 첫 번째 쿼리의 전체 컨텍스트를 갖습니다:

```python Python theme={null}
import asyncio
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    ResultMessage,
    TextBlock,
)


def print_response(message):
    """Print only the human-readable parts of a message."""
    if isinstance(message, AssistantMessage):
        for block in message.content:
            if isinstance(block, TextBlock):
                print(block.text)
    elif isinstance(message, ResultMessage):
        cost = (
            f"${message.total_cost_usd:.4f}"
            if message.total_cost_usd is not None
            else "N/A"
        )
        print(f"[done: {message.subtype}, cost: {cost}]")


async def main():
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Glob", "Grep"],
    )

    async with ClaudeSDKClient(options=options) as client:
        # First query: client captures the session ID internally
        await client.query("Analyze the auth module")
        async for message in client.receive_response():
            print_response(message)

        # Second query: automatically continues the same session
        await client.query("Now refactor it to use JWT")
        async for message in client.receive_response():
            print_response(message)


asyncio.run(main())
```

각 쿼리는 에이전트의 텍스트 응답을 출력한 후 `[done: success, cost: $0.0042]`와 같이 결과 메시지의 상태 라인을 출력합니다.

`ClaudeSDKClient`와 독립형 `query()` 함수 중 선택하는 기준은 [Python SDK 참조 문서](/docs/en/agent-sdk/python#choosing-between-query-and-claudesdkclient)를 참고하세요.

### TypeScript: `continue: true`

TypeScript SDK에는 Python의 `ClaudeSDKClient`와 같은 세션 유지 클라이언트 객체가 없습니다. 대신 이후의 각 `query()` 호출 시 `continue: true`를 전달하면 SDK가 현재 디렉토리에서 가장 최근 세션을 선택합니다. ID 추적이 필요하지 않습니다.

다음 예제는 두 개의 별도 `query()` 호출을 수행합니다. 첫 번째는 새 세션을 만들고, 두 번째는 `continue: true`를 설정하여 SDK가 디스크에서 가장 최근 세션을 찾아 재개하도록 합니다. 에이전트는 첫 번째 호출의 전체 컨텍스트를 갖습니다:

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

// First query: creates a new session
try {
  for await (const message of query({
    prompt: "Analyze the auth module",
    options: { allowedTools: ["Read", "Glob", "Grep"] }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  // A single-shot query() throws after yielding an error result,
  // so the follow-up query below still runs.
  console.error(`Session ended with an error: ${error}`);
}

// Second query: continue: true resumes the most recent session
for await (const message of query({
  prompt: "Now refactor it to use JWT",
  options: {
    continue: true,
    allowedTools: ["Read", "Edit", "Write", "Glob", "Grep"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

<Note>
  `send` / `stream` 패턴으로 `createSession()`을 제공했던 실험적 [V2 세션 API](/docs/en/agent-sdk/typescript-v2-preview)는 TypeScript Agent SDK 0.3.142에서 제거되었습니다. 대신 `query()` 함수와 이 페이지에 설명된 세션 옵션을 사용하세요.
</Note>

## `query()`에서 세션 옵션 사용하기

### 세션 ID 캡처하기

Resume 및 fork에는 세션 ID가 필요합니다. 결과 메시지(Python의 경우 [`ResultMessage`](/docs/en/agent-sdk/python#resultmessage), TypeScript의 경우 [`SDKResultMessage`](/docs/en/agent-sdk/typescript#sdkresultmessage))의 `session_id` 필드에서 읽어올 수 있으며, 성공이나 오류에 상관없이 모든 결과에 존재합니다. TypeScript에서는 초기화 `SystemMessage`에 직접 필드로 존재하며, Python에서는 `SystemMessage.data` 내부에 중첩되어 있습니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      session_id = None

      try:
          async for message in query(
              prompt="Analyze the auth module and suggest improvements",
              options=ClaudeAgentOptions(
                  allowed_tools=["Read", "Glob", "Grep"],
              ),
          ):
              if isinstance(message, ResultMessage):
                  session_id = message.session_id
                  if message.subtype == "success":
                      print(message.result)
      except Exception as error:
          # A single-shot query() raises after yielding an error result.
          # If the failure was an error result, session_id was already
          # captured by the loop above; connection or process failures
          # yield no result message.
          print(f"Session ended with an error: {error}")

      print(f"Session ID: {session_id}")
      return session_id


  session_id = asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  let sessionId: string | undefined;

  try {
    for await (const message of query({
      prompt: "Analyze the auth module and suggest improvements",
      options: { allowedTools: ["Read", "Glob", "Grep"] }
    })) {
      if (message.type === "result") {
        sessionId = message.session_id;
        if (message.subtype === "success") {
          console.log(message.result);
        }
      }
    }
  } catch (error) {
    // A single-shot query() throws after yielding an error result.
    // If the failure was an error result, sessionId was already captured
    // by the loop above; connection or process failures yield no result
    // message.
    console.error(`Session ended with an error: ${error}`);
  }

  console.log(`Session ID: ${sessionId}`);
  ```
</CodeGroup>

쿼리가 완료되면 스크립트는 에이전트의 응답을 출력한 후 `Session ID: 5b3f2c1a-8d4e-4f6b-9a7c-2e1d0f9b8a6c`와 같은 라인을 출력합니다. 다음 섹션에서는 이 ID를 `resume`에 전달합니다.

### ID로 재개하기 (Resume)

특정 세션으로 돌아가려면 해당 세션 ID를 `resume`에 전달하세요. 에이전트는 세션이 중단된 위치의 전체 컨텍스트를 유지한 채 이어서 진행합니다. 재개하는 일반적인 이유:

* **완료된 작업의 후속 조치.** 에이전트가 이미 무언가를 분석했습니다; 이제 파일들을 다시 읽지 않고 해당 분석을 토대로 작업을 실행하도록 하고 싶습니다.
* **제한(Limit) 복구.** 첫 실행이 `error_max_turns` 또는 `error_max_budget_usd`로 종료되었습니다 ([결과 처리하기](/docs/en/agent-sdk/agent-loop#handle-the-result) 참고); 더 높은 제한을 설정하여 재개합니다. 단일 `query()` 호출 시 SDK는 해당 오류 결과를 반환한 후 예외를 발생시키므로, 재개하기 전에 오류를 포착하세요.
* **프로세스 재시작.** 종료 전에 ID를 캡처해 두었고 대화를 복원하려 합니다.

다음 예제는 후속 프롬프트를 사용해 [세션 ID 캡처하기](#capture-the-session-id)의 세션을 재개합니다. 재개 중이므로 에이전트는 이미 이전 분석 내용을 컨텍스트에 담고 있습니다:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

  session_id = "..."  # The ID you captured in the previous example


  async def main():
      # Earlier session analyzed the code; now build on that analysis
      async for message in query(
          prompt="Now implement the refactoring you suggested",
          options=ClaudeAgentOptions(
              resume=session_id,
              allowed_tools=["Read", "Edit", "Write", "Glob", "Grep"],
          ),
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const sessionId = "..."; // The ID you captured in the previous example

  // Earlier session analyzed the code; now build on that analysis
  for await (const message of query({
    prompt: "Now implement the refactoring you suggested",
    options: {
      resume: sessionId,
      allowedTools: ["Read", "Edit", "Write", "Glob", "Grep"]
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```
</CodeGroup>

처음부터 다시 시작하는 대신 이전 분석을 토대로 작성된 응답을 볼 수 있습니다. 이를 통해 에이전트가 이전 컨텍스트를 그대로 유지한 채 세션을 재개했음을 확인할 수 있습니다.

<Tip>
  `resume` 호출 시 예상되는 대화 기록 대신 새 세션이 반환된다면, 가장 흔한 원인은 `cwd` 불일치입니다. 세션은 로컬 디스크의 `~/.claude/projects/<encoded-cwd>/*.jsonl` 아래에 저장되며 (`CLAUDE_CONFIG_DIR` 환경 변수를 설정한 경우 `$CLAUDE_CONFIG_DIR/projects/<encoded-cwd>/*.jsonl` 아래), 여기서 `<encoded-cwd>`는 영문 숫자가 아닌 모든 문자가 `-`로 대체된 절대 작업 디렉토리입니다 (예: `/Users/me/proj`는 `-Users-me-proj`가 됨). 재개 호출이 다른 디렉토리에서 실행되면 SDK는 엉뚱한 위치를 찾게 됩니다. 또한 세션 파일이 현재 머신에 존재해야 합니다.
</Tip>

서버리스 환경이나 여러 머신 간에 세션을 재개하려면 [`SessionStore` 어답터](/docs/en/agent-sdk/session-storage)를 사용해 세션 트랜스크립트를 공유 저장소에 미러링하세요.

### 대안 탐색을 위한 분기 (Fork)

분기(Fork)는 원본 기록의 복사본으로 시작하지만 그 시점부터 분기되는 새 세션을 생성합니다. 분기된 세션은 고유한 세션 ID를 가지며, 원본의 ID와 기록은 변경되지 않습니다. 결과적으로 별도로 재개할 수 있는 두 개의 독립된 세션을 얻게 됩니다.

<Note>
  분기(Forking)는 파일 시스템이 아니라 대화 기록을 분기합니다. 분기된 에이전트가 파일을 편집하면 해당 변경 사항은 실제 파일에 적용되어 동일 디렉토리에서 작업하는 모든 세션에 보이게 됩니다. 파일 변경 사항을 분기하고 되돌리려면 [파일 체크포인팅](/docs/en/agent-sdk/file-checkpointing)을 사용하세요.
</Note>

다음 예제는 [세션 ID 캡처하기](#capture-the-session-id)를 토대로 작성되었습니다: 이미 `session_id`에서 인증 모듈을 분석했고, JWT 중심 스레드를 잃지 않고 OAuth2를 탐색하고자 합니다. 첫 번째 블록은 세션을 분기하고 분기된 세션의 ID(`forked_id`)를 캡처합니다; 두 번째 블록은 기존 `session_id`를 재개하여 JWT 경로를 계속 진행합니다. 이제 두 개의 독립된 기록을 가리키는 두 개의 세션 ID가 생깁니다:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

  session_id = "..."  # The ID you captured in the previous example


  async def main():
      # Fork: branch from session_id into a new session
      forked_id = None
      try:
          async for message in query(
              prompt="Instead of JWT, outline how OAuth2 would work for the auth module",
              options=ClaudeAgentOptions(
                  resume=session_id,
                  fork_session=True,
                  max_turns=5,
              ),
          ):
              if isinstance(message, ResultMessage):
                  forked_id = message.session_id  # The fork's ID, distinct from session_id
                  if message.subtype == "success":
                      print(message.result)
      except Exception as error:
          # A single-shot query() raises after yielding an error result. If the
          # failure was an error result, forked_id was already captured by the
          # loop above; connection or process failures yield no result message.
          print(f"Session ended with an error: {error}")

      print(f"Forked session: {forked_id}")

      # Original session is untouched; resuming it continues the JWT thread
      try:
          async for message in query(
              prompt="Continue with the JWT approach",
              options=ClaudeAgentOptions(resume=session_id),
          ):
              if isinstance(message, ResultMessage) and message.subtype == "success":
                  print(message.result)
      except Exception as error:
          # A single-shot query() raises after yielding an error result.
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const sessionId = "..."; // The ID you captured in the previous example

  // Fork: branch from sessionId into a new session
  let forkedId: string | undefined;

  try {
    for await (const message of query({
      prompt: "Instead of JWT, outline how OAuth2 would work for the auth module",
      options: {
        resume: sessionId,
        forkSession: true,
        maxTurns: 5
      }
    })) {
      if (message.type === "system" && message.subtype === "init") {
        forkedId = message.session_id; // The fork's ID, distinct from sessionId
      }
      if (message.type === "result" && message.subtype === "success") {
        console.log(message.result);
      }
    }
  } catch (error) {
    // A single-shot query() throws after yielding an error result. If the
    // failure was an error result, forkedId was already captured by the loop
    // above; connection or process failures yield no result message.
    console.error(`Session ended with an error: ${error}`);
  }

  console.log(`Forked session: ${forkedId}`);

  // Original session is untouched; resuming it continues the JWT thread
  try {
    for await (const message of query({
      prompt: "Continue with the JWT approach",
      options: { resume: sessionId }
    })) {
      if (message.type === "result" && message.subtype === "success") {
        console.log(message.result);
      }
    }
  } catch (error) {
    // A single-shot query() throws after yielding an error result.
    console.error(`Session ended with an error: ${error}`);
  }
  ```
</CodeGroup>

`forkedId`가 원본 세션 ID와 다른 것을 확인할 수 있습니다. 원본 세션을 재개하면 여전히 JWT 스레드가 이어지므로, 분기 작업이 원본 기록을 수정하지 않았음을 확인할 수 있습니다.

## 호스트 간 세션 재개

세션 파일은 해당 파일을 생성한 머신에 저장됩니다. 다른 호스트(CI 워커, 일회성 컨테이너, 서버리스)에서 세션을 재개하려면 두 가지 옵션이 있습니다:

* **세션 파일 이동.** 첫 실행 후 `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`을 보존하고, `resume`을 호출하기 전에 새 호스트의 동일한 경로에 복원합니다. `cwd`가 일치해야 합니다.
* **세션 재개에 의존하지 않기.** 필요한 결과(분석 출력, 결정 사항, 파일 변경 내역)를 애플리케이션 상태로 캡처하여 새 세션의 프롬프트에 전송합니다. 이 방식이 트랜스크립트 파일을 주고받는 것보다 더 견고할 때가 많습니다.

두 SDK 모두 디스크에 있는 세션을 나열하고 메시지를 읽는 함수들을 제공합니다: TypeScript의 경우 [`listSessions()`](/docs/en/agent-sdk/typescript#listsessions) 및 [`getSessionMessages()`](/docs/en/agent-sdk/typescript#getsessionmessages), Python의 경우 [`list_sessions()`](/docs/en/agent-sdk/python#list_sessions) 및 [`get_session_messages()`](/docs/en/agent-sdk/python#get_session_messages). 이를 사용하여 커스텀 세션 선택기, 정리 로직 또는 트랜스크립트 뷰어를 만드세요.

또한 개별 세션을 조회하고 수정하는 함수들도 제공합니다: Python의 경우 [`get_session_info()`](/docs/en/agent-sdk/python#get_session_info), [`rename_session()`](/docs/en/agent-sdk/python#rename_session), [`tag_session()`](/docs/en/agent-sdk/python#tag_session), TypeScript의 경우 [`getSessionInfo()`](/docs/en/agent-sdk/typescript#getsessioninfo), [`renameSession()`](/docs/en/agent-sdk/typescript#renamesession), [`tagSession()`](/docs/en/agent-sdk/typescript#tagsession). 이를 사용하여 세션을 태그별로 정리하거나 사람이 읽기 쉬운 제목을 부여하세요.

## 관련 리소스

* [에이전트 루프 동작 방식](/docs/en/agent-sdk/agent-loop): 세션 내 턴, 메시지 및 컨텍스트 누적 이해하기
* [파일 체크포인팅](/docs/en/agent-sdk/file-checkpointing): 세션 내에서 에이전트가 수행한 파일 변경 사항을 스냅샷으로 저장하고 되돌리기
* [Python `ClaudeAgentOptions`](/docs/en/agent-sdk/python#claudeagentoptions): Python 세션 옵션 전체 참조
* [TypeScript `Options`](/docs/en/agent-sdk/typescript#options): TypeScript 세션 옵션 전체 참조
