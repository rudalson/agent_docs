> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# TypeScript SDK V2 세션 API (제거됨)

> 다중 대화 세션을 위한 세션 기반 송신/스트림 패턴을 포함하는, 지금은 제거된 V2 TypeScript Agent SDK 세션 API 레퍼런스입니다.

<Warning>
  V2 세션 API는 더 이상 지원되지 않습니다. TypeScript Agent SDK 0.3.142에서 `unstable_v2_createSession`, `unstable_v2_resumeSession`, `unstable_v2_prompt` 및 `SDKSession`, `SDKSessionOptions` 타입이 제거되었습니다.

  마이그레이션하려면 [`query()` API](/docs/en/agent-sdk/typescript)와 해당 API가 수용하는 [세션 옵션](/docs/en/agent-sdk/sessions)을 사용하세요. 다중 대화에는 `AsyncIterable<SDKUserMessage>`를 전달하고, 저장된 세션을 계속하려면 `options.resume`을 전달하세요. 이 페이지는 Agent SDK 0.2.x 이하에서 코드를 유지 관리하는 경우를 위해 참고용으로 보관됩니다.
</Warning>

V2는 비동기 제네레이터 및 yield 조정의 필요성을 제거한 실험적 세션 API였습니다. 여러 차례에 걸쳐 제네레이터 상태를 관리하는 대신 각 차례가 별도의 `send()`/`stream()` 사이클이었습니다. API 인터페이스는 세 가지 개념으로 단순화되었습니다:

* `createSession()` / `resumeSession()`: 대화 시작 또는 계속 진행
* `session.send()`: 메시지 전송
* `session.stream()`: 응답 스트리밍

## 설치 (Installation)

Agent SDK 0.2.x는 V2 인터페이스를 포함하는 마지막 버전입니다. 패키지 버전이 0.2.x에서 0.3.142로 직접 건너뛰었으므로 위에서의 제거 버전과 아래의 설치 고정 버전은 동일한 경계를 설명합니다. V2 호환 가능한 마지막 릴리스를 설치하려면 주 및 부 버전을 고정하세요:

```bash theme={null}
npm install @anthropic-ai/claude-agent-sdk@0.2
```

<Note>
  SDK는 선택적 종속성으로 플랫폼용 네이티브 Claude Code 바이너리를 번들로 제공하므로 Claude Code를 별도로 설치할 필요가 없습니다.
</Note>

## 빠른 시작

### 단일 실행 프롬프트 (One-shot prompt)

세션을 유지할 필요가 없는 간단한 단일 차례 쿼리의 경우 `unstable_v2_prompt()`를 사용하세요. 이 예시는 수학 질문을 보내고 답변을 로깅합니다:

```typescript theme={null}
import { unstable_v2_prompt } from "@anthropic-ai/claude-agent-sdk";

const result = await unstable_v2_prompt("What is 2 + 2?", {
  model: "claude-opus-4-7"
});
if (result.subtype === "success") {
  console.log(result.result);
}
```

<details>
  <summary>V1에서 동일한 작업 확인</summary>

  ```typescript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const q = query({
    prompt: "What is 2 + 2?",
    options: { model: "claude-opus-4-7" }
  });

  for await (const msg of q) {
    if (msg.type === "result" && msg.subtype === "success") {
      console.log(msg.result);
    }
  }
  ```
</details>

### 기본 세션

단일 프롬프트를 넘어서는 상호작용을 위해 세션을 생성합니다. V2는 송신과 스트리밍을 별도의 단계로 분리합니다:

* `send()`는 메시지를 디스패치함
* `stream()`은 응답을 다시 스트리밍함

이러한 명시적 분기를 통해 차례 사이에 로직을 추가(예: 후속 전송 전 응답 처리)하기가 더 쉬워집니다.

아래 예시는 세션을 생성하고, Claude에게 "Hello!"를 보낸 후 텍스트 응답을 출력합니다. 블록이 종료될 때 세션을 자동으로 닫기 위해 [`await using`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html#using-declarations-and-explicit-resource-management) (TypeScript 5.2+)를 사용합니다. `session.close()`를 수동으로 호출할 수도 있습니다.

```typescript theme={null}
import { unstable_v2_createSession } from "@anthropic-ai/claude-agent-sdk";

await using session = unstable_v2_createSession({
  model: "claude-opus-4-7"
});

await session.send("Hello!");
for await (const msg of session.stream()) {
  // 사람이 읽을 수 있는 출력을 얻기 위해 어시스턴트 메시지 필터링
  if (msg.type === "assistant") {
    const text = msg.message.content
      .filter((block) => block.type === "text")
      .map((block) => block.text)
      .join("");
    console.log(text);
  }
}
```

<details>
  <summary>V1에서 동일한 작업 확인</summary>

  V1에서는 입력과 출력이 모두 단일 비동기 제네레이터를 통해 흐릅니다. 기본 프롬프트의 경우 유사해 보이지만, 다중 대화 로직을 추가하려면 입력 제네레이터를 사용하도록 재구조화해야 합니다.

  ```typescript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const q = query({
    prompt: "Hello!",
    options: { model: "claude-opus-4-7" }
  });

  for await (const msg of q) {
    if (msg.type === "assistant") {
      const text = msg.message.content
        .filter((block) => block.type === "text")
        .map((block) => block.text)
        .join("");
      console.log(text);
    }
  }
  ```
</details>

### 다중 대화 (Multi-turn conversation)

세션은 여러 주고받기 전체에 걸쳐 컨텍스트를 유지합니다. 대화를 계속하려면 동일한 세션에서 `send()`를 다시 호출하세요. Claude는 이전 차례를 기억합니다.

이 예시는 수학 질문을 한 다음 이전 답변을 참조하는 후속 질문을 합니다:

```typescript theme={null}
import { unstable_v2_createSession } from "@anthropic-ai/claude-agent-sdk";

await using session = unstable_v2_createSession({
  model: "claude-opus-4-7"
});

// 차례 1
await session.send("What is 5 + 3?");
for await (const msg of session.stream()) {
  // 사람이 읽을 수 있는 출력을 얻기 위해 어시스턴트 메시지 필터링
  if (msg.type === "assistant") {
    const text = msg.message.content
      .filter((block) => block.type === "text")
      .map((block) => block.text)
      .join("");
    console.log(text);
  }
}

// 차례 2
await session.send("Multiply that by 2");
for await (const msg of session.stream()) {
  if (msg.type === "assistant") {
    const text = msg.message.content
      .filter((block) => block.type === "text")
      .map((block) => block.text)
      .join("");
    console.log(text);
  }
}
```

<details>
  <summary>V1에서 동일한 작업 확인</summary>

  ```typescript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 메시지를 제공하기 위해 비동기 반복 가능 객체를 생성해야 함
  async function* createInputStream() {
    yield {
      type: "user",
      session_id: "",
      message: { role: "user", content: [{ type: "text", text: "What is 5 + 3?" }] },
      parent_tool_use_id: null
    };
    // 다음 메시지를 생성할 시기를 조율해야 함
    yield {
      type: "user",
      session_id: "",
      message: { role: "user", content: [{ type: "text", text: "Multiply by 2" }] },
      parent_tool_use_id: null
    };
  }

  const q = query({
    prompt: createInputStream(),
    options: { model: "claude-opus-4-7" }
  });

  for await (const msg of q) {
    if (msg.type === "assistant") {
      const text = msg.message.content
        .filter((block) => block.type === "text")
        .map((block) => block.text)
        .join("");
      console.log(text);
    }
  }
  ```
</details>

### 세션 재개 (Session resume)

이전 상호작용의 세션 ID가 있는 경우 나중에 재개할 수 있습니다. 이는 오래 실행되는 워크플로우나 애플리케이션 재시작 간에 대화를 유지해야 할 때 유용합니다.

이 예시는 세션을 생성하고, ID를 저장하고, 세션을 닫은 다음 대화를 재개합니다:

```typescript theme={null}
import {
  unstable_v2_createSession,
  unstable_v2_resumeSession,
  type SDKMessage
} from "@anthropic-ai/claude-agent-sdk";

// 어시스턴트 메시지에서 텍스트를 추출하는 헬퍼
function getAssistantText(msg: SDKMessage): string | null {
  if (msg.type !== "assistant") return null;
  return msg.message.content
    .filter((block) => block.type === "text")
    .map((block) => block.text)
    .join("");
}

// 초기 세션 생성 및 대화 진행
const session = unstable_v2_createSession({
  model: "claude-opus-4-7"
});

await session.send("Remember this number: 42");

// 수신된 모든 메시지에서 세션 ID 가져오기
let sessionId: string | undefined;
for await (const msg of session.stream()) {
  sessionId = msg.session_id;
  const text = getAssistantText(msg);
  if (text) console.log("Initial response:", text);
}

console.log("Session ID:", sessionId);
session.close();

// 나중에: 저장된 ID를 사용하여 세션 재개
await using resumedSession = unstable_v2_resumeSession(sessionId!, {
  model: "claude-opus-4-7"
});

await resumedSession.send("What number did I ask you to remember?");
for await (const msg of resumedSession.stream()) {
  const text = getAssistantText(msg);
  if (text) console.log("Resumed response:", text);
}
```

<details>
  <summary>V1에서 동일한 작업 확인</summary>

  ```typescript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 초기 세션 생성
  const initialQuery = query({
    prompt: "Remember this number: 42",
    options: { model: "claude-opus-4-7" }
  });

  // 임의의 메시지에서 세션 ID 가져오기
  let sessionId: string | undefined;
  for await (const msg of initialQuery) {
    sessionId = msg.session_id;
    if (msg.type === "assistant") {
      const text = msg.message.content
        .filter((block) => block.type === "text")
        .map((block) => block.text)
        .join("");
      console.log("Initial response:", text);
    }
  }

  console.log("Session ID:", sessionId);

  // 나중에: 세션 재개
  const resumedQuery = query({
    prompt: "What number did I ask you to remember?",
    options: {
      model: "claude-opus-4-7",
      resume: sessionId
    }
  });

  for await (const msg of resumedQuery) {
    if (msg.type === "assistant") {
      const text = msg.message.content
        .filter((block) => block.type === "text")
        .map((block) => block.text)
        .join("");
      console.log("Resumed response:", text);
    }
  }
  ```
</details>

### 정리 (Cleanup)

자동 리소스 정리를 위한 TypeScript 5.2+ 기능인 [`await using`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html#using-declarations-and-explicit-resource-management)을 사용하여 세션을 수동으로 또는 자동으로 닫을 수 있습니다. 이전 TypeScript 버전을 사용 중이거나 호환성 문제가 발생하는 경우 대신 수동 정리를 사용하세요.

**자동 정리 (TypeScript 5.2+):**

```typescript theme={null}
import { unstable_v2_createSession } from "@anthropic-ai/claude-agent-sdk";

await using session = unstable_v2_createSession({
  model: "claude-opus-4-7"
});
// 블록이 종료될 때 세션이 자동으로 닫힘
```

**수동 정리:**

```typescript theme={null}
import { unstable_v2_createSession } from "@anthropic-ai/claude-agent-sdk";

const session = unstable_v2_createSession({
  model: "claude-opus-4-7"
});
// ... 세션 사용 ...
session.close();
```

## API 레퍼런스

### `unstable_v2_createSession()`

다중 대화 세션을 위해 새 세션을 생성합니다.

```typescript theme={null}
function unstable_v2_createSession(options: {
  model: string;
  // 지원되는 추가 옵션들
}): SDKSession;
```

### `unstable_v2_resumeSession()`

ID로 기존 세션을 재개합니다.

```typescript theme={null}
function unstable_v2_resumeSession(
  sessionId: string,
  options: {
    model: string;
    // 지원되는 추가 옵션들
  }
): SDKSession;
```

### `unstable_v2_prompt()`

단일 차례 쿼리를 위한 단일 실행 편의 함수입니다.

```typescript theme={null}
function unstable_v2_prompt(
  prompt: string,
  options: {
    model: string;
    // 지원되는 추가 옵션들
  }
): Promise<SDKResultMessage>;
```

### SDKSession 인터페이스

```typescript theme={null}
interface SDKSession {
  readonly sessionId: string;
  send(message: string | SDKUserMessage): Promise<void>;
  stream(): AsyncGenerator<SDKMessage, void>;
  close(): void;
}
```

## 기능 가용성 (Feature availability)

V2 세션 API는 모든 V1 기능을 지원하지는 않습니다. 다음은 [V1 SDK](/docs/en/agent-sdk/typescript)가 필요합니다:

* 세션 포크 (`forkSession` 옵션)
* 일부 고급 스트리밍 입력 패턴

## 참고 항목

* [TypeScript SDK 레퍼런스 (V1)](/docs/en/agent-sdk/typescript) - 전체 V1 SDK 문서
* [SDK 개요](/docs/en/agent-sdk/overview) - 일반 SDK 개념
* [GitHub의 V2 예시](https://github.com/anthropics/claude-agent-sdk-demos/tree/main/hello-world-v2) - 작동하는 코드 예시
