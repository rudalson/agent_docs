> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 외부 저장소에 세션 보존하기

> S3, Redis 또는 커스텀 백엔드에 세션 트랜스크립트를 미러링하여 모든 호스트에서 재개할 수 있도록 합니다.

기본적으로 SDK는 로컬 파일 시스템의 `~/.claude/projects/` 아래에 있는 JSONL 파일에 세션 트랜스크립트를 기록합니다. `SessionStore` 어댑터를 사용하면 해당 트랜스크립트를 S3, Redis 또는 데이터베이스와 같은 자체 백엔드에 미러링할 수 있으므로 한 호스트에서 생성된 세션을 다른 호스트에서 재개할 수 있습니다.

세션 저장소를 사용하는 일반적인 이유:

* **다중 호스트 배포.** 서버리스 함수, 자동 확장(autoscaled) 워커 및 CI 러너는 파일 시스템을 공유하지 않습니다. 공유 저장소를 사용하면 어떤 복제본(replica)도 모든 세션을 재개할 수 있습니다.
* **지속성(Durability).** 로컬 컨테이너는 일회성(ephemeral)입니다. S3나 데이터베이스 기반의 저장소는 재시작 및 재배포 후에도 유지됩니다.
* **준수 및 감사(Compliance and audit).** 자체 보존 규칙, 암호화 및 접근 제어를 적용하여 이미 관리 중인 저장소에 트랜스크립트를 보관합니다.

## `SessionStore` 인터페이스

`SessionStore`는 두 가지 필수 메서드인 `append` 및 `load`와 네 가지 선택적 메서드를 가진 객체입니다. SDK는 쿼리 중에 트랜스크립트 항목을 기록하기 위해 `append`를 호출하고, 재개를 위해 이를 읽어오고자 `load`를 호출합니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  // Exported from @anthropic-ai/claude-agent-sdk as
  // SessionStore, SessionKey, SessionStoreEntry, SessionSummaryEntry.

  type SessionKey = {
    projectKey: string;
    sessionId: string;
    subpath?: string;
  };

  type SessionStore = {
    // Required
    append(key: SessionKey, entries: SessionStoreEntry[]): Promise<void>;
    load(key: SessionKey): Promise<SessionStoreEntry[] | null>;

    // Optional
    listSessions?(
      projectKey: string,
    ): Promise<Array<{ sessionId: string; mtime: number }>>;
    listSessionSummaries?(projectKey: string): Promise<SessionSummaryEntry[]>;
    delete?(key: SessionKey): Promise<void>;
    listSubkeys?(key: {
      projectKey: string;
      sessionId: string;
    }): Promise<string[]>;
  };

  type SessionSummaryEntry = {
    sessionId: string;
    mtime: number;
    data: Record<string, unknown>;
  };
  ```

  ```python Python theme={null}
  # Exported from claude_agent_sdk as
  # SessionStore, SessionKey, SessionStoreEntry, SessionSummaryEntry.

  class SessionKey(TypedDict):
      project_key: str
      session_id: str
      subpath: NotRequired[str]

  class SessionStore(Protocol):
      # Required
      async def append(
          self, key: SessionKey, entries: list[SessionStoreEntry]
      ) -> None: ...
      async def load(self, key: SessionKey) -> list[SessionStoreEntry] | None: ...

      # Optional — omit or raise NotImplementedError
      async def list_sessions(
          self, project_key: str
      ) -> list[SessionStoreListEntry]: ...
      async def list_session_summaries(
          self, project_key: str
      ) -> list[SessionSummaryEntry]: ...
      async def delete(self, key: SessionKey) -> None: ...
      async def list_subkeys(self, key: SessionListSubkeysKey) -> list[str]: ...

  class SessionSummaryEntry(TypedDict):
      session_id: str
      mtime: int
      data: dict[str, Any]
  ```
</CodeGroup>

`SessionKey`는 하나의 트랜스크립트를 가리킵니다. `projectKey`는 작업 디렉토리의 안정적이고 파일 시스템 안전한 인코딩이고, `sessionId`는 세션 UUID이며, `subpath`는 항목이 메인 대화가 아닌 서브에이전트 트랜스크립트나 사이드카(sidecar) 파일에 속할 때 설정됩니다. `subpath`는 불투명한 키 접미사로 취급하며, 디스크 레이아웃(예: `subagents/agent-<id>`)을 따릅니다. `subpath`가 정의되지 않은 경우 키는 메인 트랜스크립트를 참조합니다.

| 메서드 | 필수 여부 | 호출 시점 |
| :--------------------- | :------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `append` | 예 | 각 트랜스크립트 항목 배치가 로컬에 기록된 후 호출됨. 항목은 로컬 JSONL의 줄당 하나에 해당하는 JSON 안전 객체임. |
| `load` | 예 | `resume`이 설정되었을 때 자식 프로세스가 생겨나기 전, 그리고 조회가 `listSessionSummaries`에서 폴백될 때 세션당 한 번 호출됨. 알 수 없는 세션인 경우 `null` 반환. |
| `listSessions` | 아니오 | `listSessions({ sessionStore })` 및 `continue: true` 옵션의 `query()`/`startup()`에 의해 호출됨. 정의되지 않은 경우 `continue: true`는 예외를 던지며, `listSessionSummaries`가 구현되지 않은 한 `listSessions({ sessionStore })`도 예외를 던짐. |
| `listSessionSummaries` | 아니오 | 한 번의 호출로 모든 세션의 메타데이터를 읽기 위해 `listSessions({ sessionStore })`에 의해 호출됨. `append` 내부에서 요약을 유지해야 함. 정의되지 않은 경우 조회는 `listSessions`와 세션별 `load`로 폴백됨. |
| `delete` | 아니오 | `deleteSession({ sessionStore })`에 의해 호출됨. 메인 키(`subpath` 없음) 삭제는 해당 세션의 모든 서브키로 연쇄(cascade)되어야 하며 세션의 요약 항목도 제거하여 삭제된 세션이 `listSessionSummaries`에 나타나지 않도록 해야 함. 정의되지 않은 경우 삭제는 무시(no-op)되며, 이는 추가 전용(append-only) 백엔드에 적합함. |
| `listSubkeys` | 아니오 | 재개 중에 서브에이전트 트랜스크립트를 발견하기 위해 호출됨. 정의되지 않은 경우 메인 트랜스크립트만 복원됨. |

`SessionSummaryEntry`에서 `mtime`은 사이드카의 저장소 쓰기 시간이며 `listSessions`가 반환하는 `mtime` 값과 동일한 시계 소스를 공유해야 합니다. `data`는 불투명한 SDK 소유 상태이므로 해석하지 않고 그대로 보존하세요.

`append` 내부의 각 배치에 대해 내보내진 `foldSessionSummary` 헬퍼(Python의 경우 `fold_session_summary`)를 호출하여 항목을 구성하세요. 키에 `subpath`가 있는 배치는 건너뛰어야 합니다; 서브에이전트 트랜스크립트가 메인 세션의 요약에 기여해서는 안 됩니다. 폴드는 `mtime`을 설정하지 않습니다: TypeScript의 `options.mtime` 인자나 Python의 반환 항목 필드를 덮어써서 보존 시점에 타임스탬프를 찍으세요. 동일 세션에 대한 동시 `append` 호출은 사이드카에서 경합(race)을 일으킬 수 있으므로 트랜잭션, compare-and-swap, 또는 세션별 잠금(lock)으로 읽기-폴드-쓰기를 직렬화하세요. 폴드 자체는 순수 함수입니다.

## 빠른 시작

SDK는 개발 및 테스트를 위한 `InMemorySessionStore`를 제공합니다. 아래 예제는 저장소가 첨부된 상태로 쿼리를 실행하고, 결과 메시지에서 세션 ID를 캡처한 다음, 두 번째 `query()` 호출에서 저장소로부터 재개합니다. 두 번째 호출은 동일한 저장소 인스턴스와 `resume`을 전달하므로 SDK가 로컬 파일 시스템 대신 저장소에서 트랜스크립트를 로드합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query, InMemorySessionStore } from "@anthropic-ai/claude-agent-sdk";

  const store = new InMemorySessionStore();

  let sessionId: string | undefined;
  try {
    for await (const message of query({
      prompt: "List the TypeScript files under src/",
      options: { sessionStore: store },
    })) {
      if (message.type === "result") {
        sessionId = message.session_id;
      }
    }
  } catch (error) {
    // A single-shot query() throws after yielding an error result. If the
    // failure was an error result, sessionId was already captured by the loop
    // above; connection or process failures yield no result message.
    console.error(`Session ended with an error: ${error}`);
  }

  // Resume from the store. The agent has full context from the first call.
  for await (const message of query({
    prompt: "Summarize what those files do",
    options: { sessionStore: store, resume: sessionId },
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import (
      ClaudeAgentOptions,
      InMemorySessionStore,
      ResultMessage,
      query,
  )

  store = InMemorySessionStore()


  async def main():
      session_id = None
      try:
          async for message in query(
              prompt="List the Python files under src/",
              options=ClaudeAgentOptions(session_store=store),
          ):
              if isinstance(message, ResultMessage):
                  session_id = message.session_id
      except Exception as error:
          # A single-shot query() raises after yielding an error result. If the
          # failure was an error result, session_id was already captured by the
          # loop above; connection or process failures yield no result message.
          print(f"Session ended with an error: {error}")

      # Resume from the store. The agent has full context from the first call.
      async for message in query(
          prompt="Summarize what those files do",
          options=ClaudeAgentOptions(session_store=store, resume=session_id),
      ):
          if isinstance(message, ResultMessage) and message.subtype == "success":
              print(message.result)


  asyncio.run(main())
  ```
</CodeGroup>

두 번째 쿼리는 첫 번째 쿼리의 파일 요약을 출력하며,이는 에이전트가 저장소의 전체 컨텍스트를 유지한 채 재개되었음을 보여줍니다.

## 커스텀 어답터 작성하기

백엔드에 맞게 `append` 및 `load`를 구현하세요. `listSessions()`, 단일 호출 메타데이터 읽기, `deleteSession()`, 그리고 서브에이전트 재개가 저장소에서 작동하도록 하려면 `listSessions`, `listSessionSummaries`, `delete`, 및 `listSubkeys`를 추가하세요.

`append`에 전달되는 항목은 `SessionStoreEntry` (`{ type: string; ... }` 객체) 타입입니다. 이들을 불투명한 JSON 안전 값으로 취급하세요: 순서대로 보존하고 `load`에서 동일한 순서로 반환하세요. `load`는 추가된 내용과 동등(deep-equal)한 항목을 반환해야 합니다; 바이트 단위의 일치 직렬화가 요구되지는 않으므로 객체 키의 순서를 바꾸는 Postgres `jsonb` 같은 백엔드도 괜찮습니다.

## 참조 구현

TypeScript SDK 저장소에는 [`examples/session-stores/`](https://github.com/anthropics/claude-agent-sdk-typescript/tree/main/examples/session-stores) 아래에 S3, Redis, Postgres용 실행 가능한 참조 어답터가 포함되어 있습니다. npm에는 게시되지 않으므로 필요한 `src/` 파일을 프로젝트에 복사하고 해당 백엔드 클라이언트를 설치하세요.

| 어답터 | 백엔드 클라이언트 | 저장소 모델 |
| :----------------------------------------------------------------------------------------------------------------------------- | :------------------- | :--------------------------------------------------------------------------- |
| [`S3SessionStore`](https://github.com/anthropics/claude-agent-sdk-typescript/tree/main/examples/session-stores/s3) | `@aws-sdk/client-s3` | `append()`당 하나의 JSONL 파트 파일; `load()`는 이를 나열, 정렬 및 결합함. |
| [`RedisSessionStore`](https://github.com/anthropics/claude-agent-sdk-typescript/tree/main/examples/session-stores/redis) | `ioredis` | 트랜스크립트당 `RPUSH`/`LRANGE` 리스트, 그리고 정렬된 집합(sorted-set) 세션 인덱스. |
| [`PostgresSessionStore`](https://github.com/anthropics/claude-agent-sdk-typescript/tree/main/examples/session-stores/postgres) | `pg` | `BIGSERIAL`로 정렬되는 `jsonb` 테이블 내 항목당 하나의 행(row). |

각 어답터는 사전 구성된 클라이언트 인스턴스를 사용하므로 자격 증명, TLS, 리전 및 풀링을 직접 제어할 수 있습니다. 예를 들어 S3의 경우:

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";
import { S3Client } from "@aws-sdk/client-s3";
import { S3SessionStore } from "./S3SessionStore"; // copied from examples/session-stores/s3

const store = new S3SessionStore({
  bucket: "my-claude-sessions",
  prefix: "transcripts",
  client: new S3Client({ region: "us-east-1" }),
});

for await (const message of query({
  prompt: "Hello!",
  options: { sessionStore: store },
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}

// Later, possibly on a different host:
for await (const message of query({
  prompt: "Continue where we left off",
  options: { sessionStore: store, resume: "previous-session-id" },
})) {
  // ...
}
```

### 어답터 검증하기

두 SDK 모두 `append`, `load` 및 선택적 메서드가 충족해야 하는 동작 계약을 검증하는 적합성 테스트 수트(conformance suite)를 제공합니다. 선택적 메서드에 대한 테스트는 해당 메서드가 구현되지 않은 경우 자동으로 건너뜁니다.

TypeScript에서는 예제 디렉토리에서 테스트 수트로 [`shared/conformance.ts`](https://github.com/anthropics/claude-agent-sdk-typescript/blob/main/examples/session-stores/shared/conformance.ts)를 복사하세요. Python에서는 수트가 패키지에 번들로 제공됩니다:

```python Python theme={null}
import pytest
from claude_agent_sdk.testing import run_session_store_conformance


@pytest.mark.asyncio
async def test_my_store_conformance():
    await run_session_store_conformance(MyRedisStore)  # Your adapter class
```

## 동작 관련 참고 사항

### 이중 쓰기 아키텍처

저장소는 대체재가 아닌 미러입니다. Claude Code 자식 프로세스는 항상 로컬 디스크에 먼저 씁니다; 그런 다음 SDK가 각 배치를 `append()`로 전달합니다. 로컬 사본을 일회성으로 만들려면 `options.env`에서 `CLAUDE_CONFIG_DIR`을 임시 디렉토리로 설정하세요.

미러링이 로컬 쓰기에 의존하므로 TypeScript SDK에서 `sessionStore`와 `persistSession: false`를 결합하면 예외가 발생합니다. 파일 이력 백업 블롭은 로컬 디스크에 직접 작성되고 저장소로 미러링되지 않으므로, 두 SDK 모두 저장소와 파일 체크포인팅(TypeScript의 `enableFileCheckpointing`, Python의 `enable_file_checkpointing`)을 결합하면 예외를 던집니다.

### 미러 쓰기는 최선 노력(Best-effort) 방식임

`append()`가 거부되면 SDK는 짧은 백오프와 함께 배치를 최대 2회 더 재시도하여 총 3회까지 시도합니다. 제한 시간이 초과된 호출은 원래 호출이 여전히 도달할 수 있으므로 재시도되지 않습니다. 배치가 계속 실패하면 오류가 기록되고, `{ type: "system", subtype: "mirror_error" }` 메시지가 이터레이터로 발송되고, 배치는 삭제되며, 쿼리는 계속 진행됩니다. 로컬 트랜스크립트는 이미 디스크에 안전하게 저장되어 있으므로 저장소 장애가 에이전트를 중단시키거나 로컬 데이터를 손실시키지는 않습니다. 저장소 데이터 손실을 감지해야 하는 경우 `mirror_error`를 모니터링하세요. 재시도된 배치는 이미 도달한 항목을 다시 전달할 수 있으므로 `append()` 구현 시 `entry.uuid`로 중복을 제거하세요.

### `getSessionMessages`는 수축 후 체인을 반환함

`getSessionMessages({ sessionStore })`는 에이전트가 재개 시 보게 될 연결된 메시지 체인을 반환합니다. 자동 수축(auto-compaction) 후에는 이전 턴이 요약으로 대체되므로, 저장소에 503개의 원시 항목이 보존된 세션이라도 `getSessionMessages`에서 18개의 메시지만 반환될 수 있습니다. 수축 전 턴 및 메타데이터 항목을 포함한 전체 원시 이력을 확인하려면 `store.load(key)`를 직접 호출하세요.

### `forkSession`은 바이트 복사가 아님

`forkSession({ sessionStore })`는 소스 항목을 읽고, 모든 `sessionId` 필드를 다시 작성하고, 메시지 UUID를 재매핑한 다음, 변환된 항목을 새 키 아래에 추가합니다. 어답터 수준의 복사나 `CopyObject` 단축키는 이전 세션 ID를 계속 참조하는 트랜스크립트를 생성하므로 SDK는 이를 사용하지 않습니다.

### 서브에이전트 트랜스크립트

서브에이전트 트랜스크립트는 `subpath: "subagents/agent-<id>"` 아래에 미러링됩니다. `listSubagents({ sessionStore })`를 사용하려면 어답터가 `listSubkeys`를 구현해야 합니다; `getSubagentMessages({ sessionStore })`는 사용 가능한 경우 이를 사용하지만 정의되지 않은 경우 직접적인 하위 경로로 폴백합니다. 재개 시에도 서브에이전트 파일을 복원하기 위해 `listSubkeys`를 호출합니다; 이 메서드가 없으면 메인 트랜스크립트만 구체화됩니다.

### 보존 기간 (Retention)

SDK는 저장소의 내용을 스스로 삭제하지 않습니다. 보존 관리는 어답터의 책임입니다: 준수 요건에 따라 TTL, S3 라이프사이클 정책, 또는 예약된 정리를 구현하세요. `CLAUDE_CONFIG_DIR` 아래의 로컬 트랜스크립트는 `cleanupPeriodDays` 설정에 의해 독립적으로 정리됩니다.

## 지원 현황

다음 TypeScript SDK 함수들은 `sessionStore` 옵션을 전달받으며, 옵션이 제공되면 로컬 파일 시스템 대신 저장소를 대상으로 동작합니다:

* [`query()`](/docs/en/agent-sdk/typescript#query)
* [`startup()`](/docs/en/agent-sdk/typescript#startup)
* [`listSessions()`](/docs/en/agent-sdk/typescript#listsessions)
* [`getSessionInfo()`](/docs/en/agent-sdk/typescript#getsessioninfo)
* [`getSessionMessages()`](/docs/en/agent-sdk/typescript#getsessionmessages)
* [`renameSession()`](/docs/en/agent-sdk/typescript#renamesession)
* [`tagSession()`](/docs/en/agent-sdk/typescript#tagsession)
* [`deleteSession()`](/docs/en/agent-sdk/typescript)
* [`forkSession()`](/docs/en/agent-sdk/typescript)
* [`listSubagents()`](/docs/en/agent-sdk/typescript)
* [`getSubagentMessages()`](/docs/en/agent-sdk/typescript)

Python SDK에서는 [`ClaudeAgentOptions`](/docs/en/agent-sdk/python#claudeagentoptions)에 `session_store`를 설정하여 저장소를 대상으로 `query()`를 실행합니다. 나머지 작업들에는 각각 저장소를 인자로 받는 저장소 기반 Python 함수가 존재합니다: `list_sessions_from_store()`, `get_session_info_from_store()`, `get_session_messages_from_store()`, `list_subagents_from_store()`, `get_subagent_messages_from_store()`, `rename_session_via_store()`, `tag_session_via_store()`, `delete_session_via_store()`, 및 `fork_session_via_store()`. `startup()`에 해당하는 Python 동등물은 없습니다. [Python SDK 참조 문서](/docs/en/agent-sdk/python#functions)에 기록된 독립형 함수(예: `list_sessions()`)는 로컬 세션 파일을 읽습니다.

## 관련 리소스

* [세션 작업하기](/docs/en/agent-sdk/sessions): 커스텀 저장소 없이 이어가기, 재개 및 분기
* [SDK 호스팅하기](/docs/en/agent-sdk/hosting): 다중 호스트 환경을 위한 배포 패턴
* [TypeScript `Options`](/docs/en/agent-sdk/typescript#options): 전체 옵션 참조
* [`examples/session-stores/`](https://github.com/anthropics/claude-agent-sdk-typescript/tree/main/examples/session-stores): 실행 가능한 S3, Redis, Postgres 참조 어답터
