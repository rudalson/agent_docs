> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Agent SDK 참조 문서 - Python

> 모든 함수, 타입 및 클래스를 포함하는 Python Agent SDK용 전체 API 참조 문서입니다.

## 설치

가상 환경에 패키지를 설치합니다. 최근 Debian, Ubuntu 및 Homebrew Python 설치에서는 시스템 Python에 대해 `pip install`을 실행하면 `error: externally-managed-environment` 오류와 함께 실패합니다.

```bash theme={null}
python3 -m venv .venv
source .venv/bin/activate
pip install claude-agent-sdk
```

uv, Windows PowerShell 및 API 키 설정에 대해서는 [Agent SDK 개요의 시작하기](/docs/en/agent-sdk/overview#get-started)를 참고하세요.

## `query()`와 `ClaudeSDKClient` 중 선택하기

Python SDK는 Claude Code와 상호작용하는 두 가지 방법을 제공합니다:

### 빠른 비교

| 기능 | `query()` | `ClaudeSDKClient` |
| :------------------ | :--------------------------------------------- | :--------------------------------- |
| **세션** | 기본적으로 새 세션 생성 | 동일한 세션 재사용 |
| **대화** | 단일 주고받기(exchange) | 동일한 컨텍스트 내 다중 주고받기 |
| **연결** | 자동으로 관리됨 | 수동 제어 |
| **스트리밍 입력** | ✅ 지원됨 | ✅ 지원됨 |
| **중단(Interrupt)** | ❌ 지원되지 않음 | ✅ 지원됨 |
| **후크(Hooks)** | ✅ 지원됨 | ✅ 지원됨 |
| **커스텀 도구** | ✅ 지원됨 | ✅ 지원됨 |
| **대화 이어가기** | `continue_conversation` 또는 `resume`을 통해 수동 처리 | ✅ 자동 처리 |
| **사용 사례** | 일회성 작업 | 연속 대화 |

### `query()`를 사용하는 경우 (일회성 작업)

**가장 적합한 사례:**

* 대화 기록이 필요 없는 일회성 질문
* 이전 주고받은 대화의 컨텍스트가 필요 없는 독립적인 작업
* 간단한 자동화 스크립트
* 매번 새로 시작하기를 원할 때

### `ClaudeSDKClient`를 사용하는 경우 (연속 대화)

**가장 적합한 사례:**

* **대화 이어가기** - Claude가 컨텍스트를 기억해야 할 때
* **후속 질문** - 이전 응답을 바탕으로 추가 질문을 할 때
* **대화형 애플리케이션** - 채팅 인터페이스, REPL
* **응답 기반 로직** - 다음 작업이 Claude의 응답에 따라 결정될 때
* **세션 제어** - 대화 수명주기를 명시적으로 관리할 때

## 함수 (Functions)

<Note>이 페이지의 시그니처 블록과 단순 `async for` / `async with` 조각들은 설명용입니다. 실행하려면 본문을 `async def main(): ...`으로 감싸고 `asyncio.run(main())`을 호출하세요.</Note>

### `query()`

기본적으로 Claude Code와의 각 상호작용에 대해 새 세션을 생성합니다. 메시지가 도착하는 대로 생성하는 비동기 이터레이터(async iterator)를 반환합니다. [`ClaudeAgentOptions`](#claudeagentoptions)에 `continue_conversation=True` 또는 `resume`을 전달하지 않는 한, `query()`를 호출할 때마다 이전 상호작용의 기억 없이 새로 시작합니다. [세션](/docs/en/agent-sdk/sessions)을 참고하세요.

```python theme={null}
async def query(
    *,
    prompt: str | AsyncIterable[dict[str, Any]],
    options: ClaudeAgentOptions | None = None,
    transport: Transport | None = None
) -> AsyncIterator[Message]
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 설명 |
| :---------- | :--------------------------- | :------------------------------------------------------------------------- |
| `prompt` | `str \| AsyncIterable[dict]` | 입력 프롬프트(문자열) 또는 스트리밍 모드용 비동기 이터러블 |
| `options` | `ClaudeAgentOptions \| None` | 선택적 구성 객체 (None인 경우 기본값 `ClaudeAgentOptions()`) |
| `transport` | `Transport \| None` | CLI 프로세스와 통신하기 위한 선택적 커스텀 트랜스포트 |

#### 반환값 (Returns)

대화에서 생성된 메시지를 생성하는 `AsyncIterator[Message]`를 반환합니다.

#### 예제 - 옵션 포함

```python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    options = ClaudeAgentOptions(
        system_prompt="You are an expert Python developer",
        permission_mode="acceptEdits",
    )

    async for message in query(prompt="Create a Python web server", options=options):
        print(message)


asyncio.run(main())
```

### `tool()`

타입 안전성을 갖춘 MCP 도구를 정의하기 위한 데코레이터입니다.

```python theme={null}
def tool(
    name: str,
    description: str,
    input_schema: type | dict[str, Any],
    annotations: ToolAnnotations | None = None
) -> Callable[[Callable[[Any], Awaitable[dict[str, Any]]]], SdkMcpTool[Any]]
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 설명 |
| :------------- | :---------------------------------------------- | :------------------------------------------------------------------ |
| `name` | `str` | 도구의 고유 식별자 |
| `description` | `str` | 도구가 하는 일에 대한 사람이 읽을 수 있는 설명 |
| `input_schema` | `type \| dict[str, Any]` | 도구의 입력 매개변수를 정의하는 스키마 (아래 참고) |
| `annotations` | [`ToolAnnotations`](#toolannotations)` \| None` | 클라이언트에 동작 힌트를 제공하는 선택적 MCP 도구 어노테이션 |

#### 입력 스키마 옵션

1. **단순 타입 매핑** (권장):

   ```python theme={null}
   {"text": str, "count": int, "enabled": bool}
   ```

2. **JSON Schema 형식** (복잡한 검증용):
   ```python theme={null}
   {
       "type": "object",
       "properties": {
           "text": {"type": "string"},
           "count": {"type": "integer", "minimum": 0},
       },
       "required": ["text"],
   }
   ```

#### 반환값 (Returns)

도구 구현을 감싸고 `SdkMcpTool` 인스턴스를 반환하는 데코레이터 함수입니다.

#### 예제

```python theme={null}
from claude_agent_sdk import tool
from typing import Any


@tool("greet", "Greet a user", {"name": str})
async def greet(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Hello, {args['name']}!"}]}
```

#### `ToolAnnotations`

`mcp.types`에서 다시 내보낸 타입입니다 (`from claude_agent_sdk import ToolAnnotations`로도 사용 가능). 모든 필드는 선택적 힌트입니다; 클라이언트는 보안 결정을 내릴 때 이 필드에 의존해서는 안 됩니다.

| 필드 | 타입 | 기본값 | 설명 |
| :---------------- | :------------- | :------ | :------------------------------------------------------------------------------------------------------------------- |
| `title` | `str \| None` | `None` | 도구의 사람이 읽을 수 있는 제목 |
| `readOnlyHint` | `bool \| None` | `False` | `True`인 경우 도구가 환경을 수정하지 않음 |
| `destructiveHint` | `bool \| None` | `True` | `True`인 경우 도구가 파괴적인 업데이트를 수행할 수 있음 (`readOnlyHint`가 `False`일 때만 의미가 있음) |
| `idempotentHint` | `bool \| None` | `False` | `True`인 경우 동일한 인자로 반복 호출해도 추가 영향이 없음 (`readOnlyHint`가 `False`일 때만 의미가 있음) |
| `openWorldHint` | `bool \| None` | `True` | `True`인 경우 도구가 외부 엔티티(예: 웹 검색)와 상호작용함. `False`인 경우 도구의 도메인이 닫혀 있음(예: 메모리 도구) |

```python theme={null}
from claude_agent_sdk import tool, ToolAnnotations
from typing import Any


@tool(
    "search",
    "Search the web",
    {"query": str},
    annotations=ToolAnnotations(readOnlyHint=True, openWorldHint=True),
)
async def search(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Results for: {args['query']}"}]}
```

### `create_sdk_mcp_server()`

Python 애플리케이션 내에서 실행되는 프로세스 내(in-process) MCP 서버를 생성합니다.

```python theme={null}
def create_sdk_mcp_server(
    name: str,
    version: str = "1.0.0",
    tools: list[SdkMcpTool[Any]] | None = None
) -> McpSdkServerConfig
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 기본값 | 설명 |
| :-------- | :------------------------------ | :-------- | :---------------------------------------------------- |
| `name` | `str` | - | 서버의 고유 식별자 |
| `version` | `str` | `"1.0.0"` | 서버 버전 문자열 |
| `tools` | `list[SdkMcpTool[Any]] \| None` | `None` | `@tool` 데코레이터로 생성된 도구 함수 목록 |

#### 반환값 (Returns)

`ClaudeAgentOptions.mcp_servers`에 전달할 수 있는 `McpSdkServerConfig` 객체를 반환합니다.

#### 예제

```python theme={null}
from claude_agent_sdk import tool, create_sdk_mcp_server, ClaudeAgentOptions


@tool("add", "Add two numbers", {"a": float, "b": float})
async def add(args):
    return {"content": [{"type": "text", "text": f"Sum: {args['a'] + args['b']}"}]}


@tool("multiply", "Multiply two numbers", {"a": float, "b": float})
async def multiply(args):
    return {"content": [{"type": "text", "text": f"Product: {args['a'] * args['b']}"}]}


calculator = create_sdk_mcp_server(
    name="calculator",
    version="2.0.0",
    tools=[add, multiply],  # Pass decorated functions
)

# Use with Claude
options = ClaudeAgentOptions(
    mcp_servers={"calc": calculator},
    allowed_tools=["mcp__calc__add", "mcp__calc__multiply"],
)
```

### `list_sessions()`

메타데이터와 함께 지난 세션 목록을 조회합니다. 프로젝트 디렉토리별로 필터링하거나 모든 프로젝트의 세션을 나열합니다. 동기식이며 즉시 반환됩니다.

```python theme={null}
def list_sessions(
    directory: str | None = None,
    limit: int | None = None,
    offset: int = 0,
    include_worktrees: bool = True
) -> list[SDKSessionInfo]
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 기본값 | 설명 |
| :------------------ | :------------ | :------ | :----------------------------------------------------------------------------------------------- |
| `directory` | `str \| None` | `None` | 세션을 조회할 디렉토리. 생략하면 모든 프로젝트의 세션을 반환함 |
| `limit` | `int \| None` | `None` | 반환할 세션의 최대 개수 |
| `offset` | `int` | `0` | 정렬된 결과의 시작 부분에서 건너뛸 세션 개수. 페이지네이션을 위해 `limit`과 함께 사용 |
| `include_worktrees` | `bool` | `True` | `directory`가 git 저장소 내부인 경우 모든 워크트리 경로의 세션을 포함함 |

#### 반환 타입: `SDKSessionInfo`

| 속성 | 타입 | 설명 |
| :-------------- | :------------ | :------------------------------------------------------------------- |
| `session_id` | `str` | 고유 세션 식별자 |
| `summary` | `str` | 표시 제목: 커스텀 제목, 자동 생성된 요약 또는 첫 번째 프롬프트 |
| `last_modified` | `int` | 에포크(epoch) 이후 밀리초 단위의 최종 수정 시간 |
| `file_size` | `int \| None` | 바이트 단위 세션 파일 크기 (원격 저장소 백엔드의 경우 `None`) |
| `custom_title` | `str \| None` | 사용자가 설정한 세션 제목 |
| `first_prompt` | `str \| None` | 세션의 첫 번째 의미 있는 사용자 프롬프트 |
| `git_branch` | `str \| None` | 세션 종료 시점의 Git 브랜치 |
| `cwd` | `str \| None` | 세션의 작업 디렉토리 |
| `tag` | `str \| None` | 사용자가 설정한 세션 태그 ([`tag_session()`](#tag_session) 참고) |
| `created_at` | `int \| None` | 에포크(epoch) 이후 밀리초 단위의 세션 생성 시간 |

#### 예제

프로젝트의 가장 최근 세션 10개를 출력합니다. 결과는 `last_modified` 내림차순으로 정렬되므로 첫 번째 항목이 가장 최신 항목입니다. 모든 프로젝트를 검색하려면 `directory`를 생략하세요.

```python theme={null}
from claude_agent_sdk import list_sessions

for session in list_sessions(directory="/path/to/project", limit=10):
    print(f"{session.summary} ({session.session_id})")
```

### `get_session_messages()`

지난 세션에서 메시지를 가져옵니다. 동기식이며 즉시 반환됩니다.

```python theme={null}
def get_session_messages(
    session_id: str,
    directory: str | None = None,
    limit: int | None = None,
    offset: int = 0
) -> list[SessionMessage]
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 기본값 | 설명 |
| :----------- | :------------ | :------- | :---------------------------------------------------------------- |
| `session_id` | `str` | 필수 | 메시지를 가져올 세션 ID |
| `directory` | `str \| None` | `None` | 검색할 프로젝트 디렉토리. 생략하면 모든 프로젝트를 검색함 |
| `limit` | `int \| None` | `None` | 반환할 메시지의 최대 개수 |
| `offset` | `int` | `0` | 시작 부분에서 건너뛸 메시지 개수 |

#### 반환 타입: `SessionMessage`

| 속성 | 타입 | 설명 |
| :------------------- | :----------------------------- | :------------------------ |
| `type` | `Literal["user", "assistant"]` | 메시지 역할 |
| `uuid` | `str` | 고유 메시지 식별자 |
| `session_id` | `str` | 세션 식별자 |
| `message` | `Any` | 원시 메시지 내용 |
| `parent_tool_use_id` | `None` | 향후 용도를 위해 예약됨 |

#### 예제

```python theme={null}
from claude_agent_sdk import list_sessions, get_session_messages

sessions = list_sessions(limit=1)
if sessions:
    messages = get_session_messages(sessions[0].session_id)
    for msg in messages:
        print(f"[{msg.type}] {msg.uuid}")
```

### `get_session_info()`

전체 프로젝트 디렉토리를 스캔하지 않고 ID로 단일 세션의 메타데이터를 읽습니다. 동기식이며 즉시 반환됩니다.

```python theme={null}
def get_session_info(
    session_id: str,
    directory: str | None = None,
) -> SDKSessionInfo | None
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 기본값 | 설명 |
| :----------- | :------------ | :------- | :--------------------------------------------------------------------- |
| `session_id` | `str` | 필수 | 조회할 세션의 UUID |
| `directory` | `str \| None` | `None` | 프로젝트 디렉토리 경로. 생략하면 모든 프로젝트 디렉토리를 검색함 |

[`SDKSessionInfo`](#return-type-sdksessioninfo)를 반환하며, 세션을 찾을 수 없는 경우 `None`을 반환합니다.

#### 예제

프로젝트 디렉토리를 스캔하지 않고 단일 세션의 메타데이터를 조회합니다. 이전 실행에서 이미 세션 ID를 확보하고 있을 때 유용합니다.

```python theme={null}
from claude_agent_sdk import get_session_info

info = get_session_info("550e8400-e29b-41d4-a716-446655440000")
if info:
    print(f"{info.summary} (branch: {info.git_branch}, tag: {info.tag})")
```

### `rename_session()`

커스텀 제목 항목을 추가하여 세션 이름을 변경합니다. 반복 호출해도 안전하며, 가장 최근 제목이 적용됩니다. 동기식입니다.

```python theme={null}
def rename_session(
    session_id: str,
    title: str,
    directory: str | None = None,
) -> None
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 기본값 | 설명 |
| :----------- | :------------ | :------- | :--------------------------------------------------------------------- |
| `session_id` | `str` | 필수 | 이름을 변경할 세션의 UUID |
| `title` | `str` | 필수 | 새 제목. 공백을 제거한 후 비어있지 않아야 함 |
| `directory` | `str \| None` | `None` | 프로젝트 디렉토리 경로. 생략하면 모든 프로젝트 디렉토리를 검색함 |

`session_id`가 유효한 UUID가 아니거나 `title`이 빈 경우 `ValueError`를 발생시키고, 세션을 찾을 수 없는 경우 `FileNotFoundError`를 발생시킵니다.

#### 예제

나중에 쉽게 찾을 수 있도록 가장 최근 세션의 이름을 변경합니다. 새 제목은 이후 읽기 시 [`SDKSessionInfo.custom_title`](#return-type-sdksessioninfo)에 나타납니다.

```python theme={null}
from claude_agent_sdk import list_sessions, rename_session

sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    rename_session(sessions[0].session_id, "Refactor auth module")
```

### `tag_session()`

세션에 태그를 지정합니다. 태그를 지우려면 `None`을 전달하세요. 반복 호출해도 안전하며, 가장 최근 태그가 적용됩니다. 동기식입니다.

```python theme={null}
def tag_session(
    session_id: str,
    tag: str | None,
    directory: str | None = None,
) -> None
```

#### 매개변수 (Parameters)

| 매개변수 | 타입 | 기본값 | 설명 |
| :----------- | :------------ | :------- | :--------------------------------------------------------------------- |
| `session_id` | `str` | 필수 | 태그를 지정할 세션의 UUID |
| `tag` | `str \| None` | 필수 | 태그 문자열, 또는 지우려면 `None`. 저장 전에 유니코드 정제(sanitized) 처리됨 |
| `directory` | `str \| None` | `None` | 프로젝트 디렉토리 경로. 생략하면 모든 프로젝트 디렉토리를 검색함 |

`session_id`가 유효한 UUID가 아니거나 정제 후 `tag`가 빈 경우 `ValueError`를 발생시키고, 세션을 찾을 수 없는 경우 `FileNotFoundError`를 발생시킵니다.

#### 예제

세션에 태그를 지정한 후 나중에 읽을 때 해당 태그로 필터링합니다. 기존 태그를 지우려면 `None`을 전달하세요.

```python theme={null}
from claude_agent_sdk import list_sessions, tag_session

# Tag the most recent session
sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    tag_session(sessions[0].session_id, "needs-review")

# Later: find all sessions with that tag
for session in list_sessions(directory="/path/to/project"):
    if session.tag == "needs-review":
        print(session.summary)
```

## 클래스 (Classes)

### `ClaudeSDKClient`

**여러 대화 주고받기 과정에서 세션을 유지합니다.** 이는 TypeScript SDK의 `query()` 함수가 내부적으로 작동하는 방식과 동일한 Python 방식입니다. 대화를 계속할 수 있는 클라이언트 객체를 생성합니다.

#### 주요 기능

* **세션 연속성**: 여러 `query()` 호출 간에 대화 컨텍스트 유지
* **동일 대화**: 세션에 이전 메시지 유지
* **중단(Interrupt) 지원**: 작업 도중 실행 중단 가능
* **명시적 수명주기**: 세션 시작 및 종료 시점을 사용자가 직접 제어
* **응답 기반 흐름**: 응답에 반응하고 후속 작업 전송 가능
* **커스텀 도구 및 후크**: `@tool` 데코레이터로 생성된 커스텀 도구 및 후크 지원

```python theme={null}
class ClaudeSDKClient:
    def __init__(self, options: ClaudeAgentOptions | None = None, transport: Transport | None = None)
    async def connect(self, prompt: str | AsyncIterable[dict] | None = None) -> None
    async def query(self, prompt: str | AsyncIterable[dict], session_id: str = "default") -> None
    async def receive_messages(self) -> AsyncIterator[Message]
    async def receive_response(self) -> AsyncIterator[Message]
    async def interrupt(self) -> None
    async def set_permission_mode(self, mode: str) -> None
    async def set_model(self, model: str | None = None) -> None
    async def rewind_files(self, user_message_id: str) -> None
    async def get_mcp_status(self) -> McpStatusResponse
    async def reconnect_mcp_server(self, server_name: str) -> None
    async def toggle_mcp_server(self, server_name: str, enabled: bool) -> None
    async def stop_task(self, task_id: str) -> None
    async def get_server_info(self) -> dict[str, Any] | None
    async def disconnect(self) -> None
```

#### 메서드 (Methods)

| 메서드 | 설명 |
| :---------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `__init__(options)` | 선택적 구성으로 클라이언트 초기화 |
| `connect(prompt)` | 선택적 초기 프롬프트 또는 메시지 스트림으로 Claude에 연결 |
| `query(prompt, session_id)` | 스트리밍 모드로 새 요청 전송 |
| `receive_messages()` | 비동기 이터레이터로 Claude의 모든 메시지 수신 |
| `receive_response()` | ResultMessage를 포함할 때까지 메시지 수신 |
| `interrupt()` | 중단 신호 전송 (스트리밍 모드에서만 작동) |
| `set_permission_mode(mode)` | 현재 세션의 권한 모드 변경 |
| `set_model(model)` | 현재 세션의 모델 변경. 기본값으로 재설정하려면 `None` 전달 |
| `rewind_files(user_message_id)` | 파일을 지정된 사용자 메시지 시점의 상태로 복원. `enable_file_checkpointing=True` 필요. [파일 체크포인팅](/docs/en/agent-sdk/file-checkpointing) 참고 |
| `get_mcp_status()` | 구성된 모든 MCP 서버의 상태 조회. [`McpStatusResponse`](#mcpstatusresponse) 반환 |
| `reconnect_mcp_server(server_name)` | 실패했거나 연결 해제된 MCP 서버에 재연결 시도 |
| `toggle_mcp_server(server_name, enabled)` | 세션 중간에 MCP 서버 활성화 또는 비활성화. 비활성화 시 사용 가능한 도구에서 제거됨 |
| `stop_task(task_id)` | 실행 중인 백그라운드 작업 중지. 메시지 스트림에 상태가 `"stopped"`인 [`TaskNotificationMessage`](#tasknotificationmessage)가 뒤따름 |
| `get_server_info()` | 세션 ID 및 기능을 포함한 서버 정보 조회 |
| `disconnect()` | Claude 연결 해제 |

#### 컨텍스트 매니저 지원

자동 연결 관리를 위해 클라이언트를 비동기 컨텍스트 매니저로 사용할 수 있습니다:

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient


async def main():
    async with ClaudeSDKClient() as client:
        await client.query("Hello Claude")
        async for message in client.receive_response():
            print(message)


asyncio.run(main())
```

> **중요:** 메시지를 이터레이션할 때 `break`를 사용하여 조기 종료하는 것은 asyncio 정리 문제를 일으킬 수 있으므로 피하세요. 대신 이터레이션이 자연스럽게 완료되도록 하거나 플래그를 사용하여 필요한 정보를 찾았는지 추적하세요.

#### 예제 - 대화 이어가기

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, AssistantMessage, TextBlock, ResultMessage


async def main():
    async with ClaudeSDKClient() as client:
        # First question
        await client.query("What's the capital of France?")

        # Process response
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

        # Follow-up question - the session retains the previous context
        await client.query("What's the population of that city?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

        # Another follow-up - still in the same conversation
        await client.query("What are some famous landmarks there?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")


asyncio.run(main())
```

#### 예제 - ClaudeSDKClient를 사용한 스트리밍 입력

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient


async def message_stream():
    """Generate messages dynamically."""
    yield {
        "type": "user",
        "message": {"role": "user", "content": "Analyze the following data:"},
    }
    await asyncio.sleep(0.5)
    yield {
        "type": "user",
        "message": {"role": "user", "content": "Temperature: 25°C, Humidity: 60%"},
    }
    await asyncio.sleep(0.5)
    yield {
        "type": "user",
        "message": {"role": "user", "content": "What patterns do you see?"},
    }


async def main():
    async with ClaudeSDKClient() as client:
        # Stream input to Claude
        await client.query(message_stream())

        # Process response
        async for message in client.receive_response():
            print(message)

        # Follow-up in same session
        await client.query("Should we be concerned about these readings?")

        async for message in client.receive_response():
            print(message)


asyncio.run(main())
```

#### 예제 - 중단(Interrupt) 활용하기

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, ResultMessage


async def interruptible_task():
    options = ClaudeAgentOptions(allowed_tools=["Bash"], permission_mode="acceptEdits")

    async with ClaudeSDKClient(options=options) as client:
        # Start a long-running task
        await client.query("Count from 1 to 100 slowly, using the bash sleep command")

        # Let it run for a bit
        await asyncio.sleep(2)

        # Interrupt the task
        await client.interrupt()
        print("Task interrupted!")

        # Drain the interrupted task's messages (including its ResultMessage)
        async for message in client.receive_response():
            if isinstance(message, ResultMessage):
                print(f"Interrupted task finished with subtype={message.subtype!r}")
                # subtype is "error_during_execution" for interrupted tasks

        # Send a new command
        await client.query("Just say hello instead")

        # Now receive the new response
        async for message in client.receive_response():
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(f"New result: {message.result}")


asyncio.run(interruptible_task())
```

<Note>
  **중단 후 버퍼 동작:** `interrupt()`는 정지 신호를 보내지만 메시지 버퍼를 비우지는 않습니다. 중단된 작업에 의해 이미 생성된 메시지(`subtype="error_during_execution"` 형태의 `ResultMessage` 포함)는 스트림에 남아 있습니다. 새 쿼리의 응답을 읽기 전에 `receive_response()`로 이 메시지들을 소비해야 합니다. `interrupt()` 직후 새 쿼리를 전송하고 `receive_response()`를 한 번만 호출하면 새 쿼리의 응답이 아니라 중단된 작업의 메시지를 받게 됩니다.
</Note>

#### 예제 - 고급 권한 제어

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from claude_agent_sdk.types import (
    PermissionResultAllow,
    PermissionResultDeny,
    ToolPermissionContext,
)


async def custom_permission_handler(
    tool_name: str, input_data: dict, context: ToolPermissionContext
) -> PermissionResultAllow | PermissionResultDeny:
    """Custom logic for tool permissions."""

    # Block writes to system directories
    if tool_name == "Write" and input_data.get("file_path", "").startswith("/system/"):
        return PermissionResultDeny(
            message="System directory write not allowed", interrupt=True
        )

    # Redirect sensitive file operations
    if tool_name in ["Write", "Edit"] and "config" in input_data.get("file_path", ""):
        safe_path = f"./sandbox/{input_data['file_path']}"
        return PermissionResultAllow(
            updated_input={**input_data, "file_path": safe_path}
        )

    # Allow everything else
    return PermissionResultAllow(updated_input=input_data)


async def main():
    # Don't also list the gated tools in allowed_tools: allow rules approve calls before can_use_tool runs
    options = ClaudeAgentOptions(can_use_tool=custom_permission_handler)

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Update the system config file")

        async for message in client.receive_response():
            # Will use sandbox path instead
            print(message)


asyncio.run(main())
```

## 타입 (Types)

<Note>
  **`@dataclass` vs `TypedDict`:** 이 SDK는 두 가지 종류의 타입을 사용합니다. `@dataclass`로 장식된 클래스(`ResultMessage`, `AgentDefinition`, `TextBlock` 등)는 런타임에 객체 인스턴스이며 속성 액세스를 지원합니다: `msg.result`. `TypedDict`로 정의된 클래스(`ThinkingConfigEnabled`, `McpStdioServerConfig`, `SyncHookJSONOutput` 등)는 **런타임에 일반 dict**이므로 키 액세스가 필요합니다: `config.budget_tokens`가 아닌 `config["budget_tokens"]`. `ClassName(field=value)` 호출 구문은 두 가지 모두에서 작동하지만 dataclass만 속성이 있는 객체를 생성합니다.
</Note>

### `SdkMcpTool`

`@tool` 데코레이터로 생성된 SDK MCP 도구의 정의입니다.

```python theme={null}
@dataclass
class SdkMcpTool(Generic[T]):
    name: str
    description: str
    input_schema: type[T] | dict[str, Any]
    handler: Callable[[T], Awaitable[dict[str, Any]]]
    annotations: ToolAnnotations | None = None
```

| 속성 | 타입 | 설명 |
| :------------- | :----------------------------------------- | :--------------------------------------------------------------------------------------------------------- |
| `name` | `str` | 도구의 고유 식별자 |
| `description` | `str` | 사람이 읽을 수 있는 설명 |
| `input_schema` | `type[T] \| dict[str, Any]` | 입력 검증을 위한 스키마 |
| `handler` | `Callable[[T], Awaitable[dict[str, Any]]]` | 도구 실행을 처리하는 비동기 함수 |
| `annotations` | `ToolAnnotations \| None` | 선택적 MCP 도구 어노테이션 (예: `readOnlyHint`, `destructiveHint`, `openWorldHint`). `mcp.types`에서 가져옴 |

### `Transport`

커스텀 트랜스포트 구현을 위한 추상 기반 클래스입니다. 커스텀 채널을 통해 Claude 프로세스와 통신할 때 사용합니다(예: 로컬 자식 프로세스 대신 원격 연결).

<Warning>
  이것은 저수준 내부 API입니다. 인터페이스는 향후 릴리스에서 변경될 수 있습니다. 인터페이스 변경 사항에 맞게 커스텀 구현을 업데이트해야 합니다.
</Warning>

```python theme={null}
from abc import ABC, abstractmethod
from collections.abc import AsyncIterator
from typing import Any


class Transport(ABC):
    @abstractmethod
    async def connect(self) -> None: ...

    @abstractmethod
    async def write(self, data: str) -> None: ...

    @abstractmethod
    def read_messages(self) -> AsyncIterator[dict[str, Any]]: ...

    @abstractmethod
    async def close(self) -> None: ...

    @abstractmethod
    def is_ready(self) -> bool: ...

    @abstractmethod
    async def end_input(self) -> None: ...
```

| 메서드 | 설명 |
| :---------------- | :-------------------------------------------------------------------------- |
| `connect()` | 트랜스포트를 연결하고 통신 준비 |
| `write(data)` | 원시 데이터(JSON + 줄바꿈)를 트랜스포트에 작성 |
| `read_messages()` | 파싱된 JSON 메시지를 생성하는 비동기 이터레이터 |
| `close()` | 연결을 닫고 리소스 정리 |
| `is_ready()` | 트랜스포트가 전송 및 수신을 할 수 있는 상태면 `True` 반환 |
| `end_input()` | 입력 스트림 닫기 (예: 자식 프로세스 트랜스포트의 경우 stdin 닫기) |

가져오기: `from claude_agent_sdk import Transport`

### `ClaudeAgentOptions`

Claude Code 쿼리용 구성 dataclass입니다.

```python theme={null}
@dataclass
class ClaudeAgentOptions:
    tools: list[str] | ToolsPreset | None = None
    allowed_tools: list[str] = field(default_factory=list)
    system_prompt: str | SystemPromptPreset | SystemPromptFile | None = None
    mcp_servers: dict[str, McpServerConfig] | str | Path = field(default_factory=dict)
    strict_mcp_config: bool = False
    permission_mode: PermissionMode | None = None
    continue_conversation: bool = False
    resume: str | None = None
    session_id: str | None = None
    max_turns: int | None = None
    max_budget_usd: float | None = None
    disallowed_tools: list[str] = field(default_factory=list)
    model: str | None = None
    fallback_model: str | None = None
    betas: list[SdkBeta] = field(default_factory=list)
    output_format: dict[str, Any] | None = None
    permission_prompt_tool_name: str | None = None
    cwd: str | Path | None = None
    cli_path: str | Path | None = None
    settings: str | None = None
    add_dirs: list[str | Path] = field(default_factory=list)
    env: dict[str, str] = field(default_factory=dict)
    extra_args: dict[str, str | None] = field(default_factory=dict)
    max_buffer_size: int | None = None
    debug_stderr: Any = sys.stderr  # Deprecated
    stderr: Callable[[str], None] | None = None
    can_use_tool: CanUseTool | None = None
    hooks: dict[HookEvent, list[HookMatcher]] | None = None
    user: str | None = None
    include_partial_messages: bool = False
    include_hook_events: bool = False
    fork_session: bool = False
    agents: dict[str, AgentDefinition] | None = None
    setting_sources: list[SettingSource] | None = None
    skills: list[str] | Literal["all"] | None = None
    sandbox: SandboxSettings | None = None
    plugins: list[SdkPluginConfig] = field(default_factory=list)
    max_thinking_tokens: int | None = None  # Deprecated: use thinking instead
    thinking: ThinkingConfig | None = None
    effort: EffortLevel | None = None
    enable_file_checkpointing: bool = False
    session_store: SessionStore | None = None
    session_store_flush: SessionStoreFlushMode = "batched"
    load_timeout_ms: int = 60_000
    task_budget: TaskBudget | None = None
```

| 속성 | 타입 | 기본값 | 설명 |
| :---------------------------- | :-------------------------------------------------------------------- | :--------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tools` | `list[str] \| ToolsPreset \| None` | `None` | 도구 구성. Claude Code의 기본 도구를 사용하려면 `{"type": "preset", "preset": "claude_code"}`를 전달 |
| `allowed_tools` | `list[str]` | `[]` | 프롬프트 없이 자동 승인할 도구들. Claude가 이 도구들만 사용하도록 제한하지는 않음; 나열되지 않은 도구는 `permission_mode` 및 `can_use_tool`로 넘어감. 도구를 차단하려면 `disallowed_tools`를 사용. [권한](/docs/en/agent-sdk/permissions#allow-and-deny-rules) 참고 |
| `system_prompt` | `str \| SystemPromptPreset \| SystemPromptFile \| None` | `None` | 시스템 프롬프트 구성. 커스텀 프롬프트는 문자열, 선택적 `"append"`가 포함된 Claude Code의 시스템 프롬프트는 `{"type": "preset", "preset": "claude_code"}`, 대용량 프롬프트를 디스크에서 로드할 때는 `{"type": "file", "path": "..."}`를 전달. [`SystemPromptPreset`](#systempromptpreset) 및 [`SystemPromptFile`](#systempromptfile) 참고 |
| `mcp_servers` | `dict[str, McpServerConfig] \| str \| Path` | `{}` | MCP 서버 구성 또는 구성 파일 경로 |
| `strict_mcp_config` | `bool` | `False` | `True`인 경우 `mcp_servers`로 전달된 서버만 사용하고 프로젝트 `.mcp.json`, 사용자 설정, 플러그인이 제공하는 MCP 서버 및 [claude.ai 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai)를 무시함. CLI `--strict-mcp-config` 플래그에 매핑됨 |
| `permission_mode` | `PermissionMode \| None` | `None` | 도구 사용 권한 모드 |
| `continue_conversation` | `bool` | `False` | 가장 최근 대화 이어가기 |
| `resume` | `str \| None` | `None` | 재개할 세션 ID |
| `session_id` | `str \| None` | `None` | 자동 생성된 세션 ID 대신 특정 세션 ID 사용. 유효한 UUID여야 함. `fork_session`도 함께 설정되지 않는 한 `continue_conversation` 또는 `resume`과 조합할 수 없음 |
| `max_turns` | `int \| None` | `None` | 최대 에이전트 턴 수 (도구 사용 왕복 회수) |
| `max_budget_usd` | `float \| None` | `None` | 클라이언트 측 예상 비용이 이 USD 값에 도달하면 쿼리 중지. `total_cost_usd`와 동일한 추정치와 비교됨; 정확도 관련 주의사항은 [비용 및 사용량 추적](/docs/en/agent-sdk/cost-tracking) 참고 |
| `disallowed_tools` | `list[str]` | `[]` | 거부할 도구들. `"Bash"`와 같은 단순 이름은 Claude의 컨텍스트에서 도구를 제거함. `"Bash(rm *)"`와 같이 범위가 지정된 규칙은 도구를 사용 가능하게 두고 `bypassPermissions`를 포함한 모든 권한 모드에서 일치하는 호출을 거부함. [권한](/docs/en/agent-sdk/permissions#allow-and-deny-rules) 참고 |
| `enable_file_checkpointing` | `bool` | `False` | 되감기(rewinding)를 위한 파일 변경 사항 추적 활성화. [파일 체크포인팅](/docs/en/agent-sdk/file-checkpointing) 참고 |
| `model` | `str \| None` | `None` | Claude 모델 별칭 또는 전체 모델 이름. [수용 가능한 값 및 제공자별 ID](/docs/en/model-config#available-models) 참고 |
| `fallback_model` | `str \| None` | `None` | 기본 모델 실패 시 사용할 폴백 모델 |
| `betas` | `list[SdkBeta]` | `[]` | 활성화할 베타 기능들. 사용 가능한 옵션은 [`SdkBeta`](#sdkbeta) 참고 |
| `output_format` | `dict[str, Any] \| None` | `None` | 구조화된 응답을 위한 출력 형식 (예: `{"type": "json_schema", "schema": {...}}`). 자세한 내용은 [구조화된 출력](/docs/en/agent-sdk/structured-outputs) 참고 |
| `permission_prompt_tool_name` | `str \| None` | `None` | 권한 프롬프트용 MCP 도구 이름 |
| `cwd` | `str \| Path \| None` | `None` | 현재 작업 디렉토리 |
| `cli_path` | `str \| Path \| None` | `None` | Claude Code CLI 실행 파일의 커스텀 경로 |
| `settings` | `str \| None` | `None` | 설정 파일 경로 |
| `add_dirs` | `list[str \| Path]` | `[]` | Claude가 접근할 수 있는 추가 디렉토리 |
| `env` | `dict[str, str]` | `{}` | 상속된 프로세스 환경 변수 위에 병합되는 환경 변수들. 기본 CLI가 읽는 환경 변수는 [환경 변수](/docs/en/env-vars)를, 타임아웃 관련 변수는 [느리거나 정지된 API 응답 처리](#handle-slow-or-stalled-api-responses)를 참고 |
| `extra_args` | `dict[str, str \| None]` | `{}` | CLI에 직접 전달할 추가 CLI 인자 |
| `max_buffer_size` | `int \| None` | `None` | CLI stdout 버퍼링 시 최대 바이트 수 |
| `debug_stderr` | `Any` | `sys.stderr` | *Deprecated* - 디버그 출력용 파일 유사 객체. 대신 `stderr` 콜백 사용 |
| `stderr` | `Callable[[str], None] \| None` | `None` | CLI의 stderr 출력을 위한 콜백 함수 |
| `can_use_tool` | [`CanUseTool`](#canusetool) ` \| None` | `None` | [권한 평가 흐름](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)이 프롬프트로 넘어갈 때만 호출되는 도구 권한 콜백. `allowed_tools`, 허용 규칙 또는 `permission_mode`에 의해 자동 승인된 호출에는 호출되지 않음. `AskUserQuestion`, 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 허용했더라도 콜백에 도달함; `dontAsk` 모드에서는 이들이 대신 거부됨. 자세한 내용은 [`CanUseTool`](#canusetool) 참고 |
| `hooks` | `dict[HookEvent, list[HookMatcher]] \| None` | `None` | 이벤트를 차단/가로채기 위한 후크 구성 |
| `user` | `str \| None` | `None` | 사용자 식별자 |
| `include_partial_messages` | `bool` | `False` | 부분 메시지 스트리밍 이벤트 포함. 활성화 시 [`StreamEvent`](#streamevent) 메시지가 생김 |
| `include_hook_events` | `bool` | `False` | 메시지 스트림에 후크 수명주기 이벤트를 `HookEventMessage` 객체로 포함 |
| `fork_session` | `bool` | `False` | `resume`으로 재개할 때 기존 세션을 이어가지 않고 새 세션 ID로 분기(fork) |
| `agents` | `dict[str, AgentDefinition] \| None` | `None` | 프로그래밍 방식으로 정의된 서브에이전트 |
| `plugins` | `list[SdkPluginConfig]` | `[]` | 로컬 경로에서 커스텀 플러그인 로드. 자세한 내용은 [플러그인](/docs/en/agent-sdk/plugins) 참고 |
| `sandbox` | [`SandboxSettings`](#sandboxsettings) ` \| None` | `None` | 프로그래밍 방식으로 샌드박스 동작 구성. 자세한 내용은 [샌드박스 설정](#sandboxsettings) 참고 |
| `setting_sources` | `list[SettingSource] \| None` | `None` (CLI 기본값: 모든 소스) | 로드할 파일 시스템 설정을 제어. 사용자, 프로젝트 및 로컬 설정을 비활성화하려면 `[]` 전달. 엔드포인트 관리 정책은 항상 로드됨; 서버 관리 설정은 [지원 대상 구성](/docs/en/server-managed-settings#platform-availability)에서 조직 자격 증명으로 세션 인증 시 가져옴. [Claude Code 기능 사용](/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control) 참고 |
| `skills` | `list[str] \| Literal["all"] \| None` | `None` | 세션에서 사용할 수 있는 스킬 목록. 모든 검색된 스킬을 활성화하려면 `"all"` 전달, 또는 스킬 이름 목록 전달. 설정 시 SDK가 `allowed_tools`에 Skill 도구를 자동으로 추가함. `tools`도 함께 전달하는 경우 해당 목록에 `"Skill"`을 포함. [스킬](/docs/en/agent-sdk/skills) 참고 |
| `max_thinking_tokens` | `int \| None` | `None` | *Deprecated* - 사고 블록의 최대 토큰 수. 대신 `thinking` 사용 |
| `thinking` | [`ThinkingConfig`](#thinkingconfig) ` \| None` | `None` | 확장 사고(extended thinking) 동작 제어. `max_thinking_tokens`보다 우선함 |
| `effort` | [`EffortLevel`](#effortlevel) ` \| None` | `None` | 사고 깊이에 대한 노력 수준. [노력 수준 조정](/docs/en/model-config#adjust-effort-level) 참고 |
| `session_store` | [`SessionStore`](/docs/en/agent-sdk/session-storage#the-sessionstore-interface) ` \| None` | `None` | 모든 호스트가 세션을 재개할 수 있도록 세션 트랜스크립트를 외부 백엔드에 미러링. [외부 저장소에 세션 보존](/docs/en/agent-sdk/session-storage) 참고 |
| `session_store_flush` | `Literal["batched", "eager"]` | `"batched"` | 미러링된 트랜스크립트 항목을 `session_store`로 플러시하는 시점. `"batched"`는 턴당 한 번 또는 버퍼가 찰 때 플러시; `"eager"`는 모든 프레임 이후 백그라운드 플러시 트리거. `session_store`가 `None`인 경우 무시됨 |
| `load_timeout_ms` | `int` | `60000` | 재개 구체화 중 `session_store.load()` 및 `list_subkeys()`의 호출별 타임아웃 (밀리초) |
| `task_budget` | `TaskBudget \| None` | `None` | API 측 토큰 예산. `task-budgets-2026-03-13` 베타 헤더와 함께 `output_config.task_budget`으로 전송됨. `{"total": <int>}` 형태로 전달 |

#### 느리거나 정지된 API 응답 처리

CLI 자식 프로세스는 API 타임아웃 및 정지 감지를 제어하는 여러 환경 변수를 읽습니다. `ClaudeAgentOptions.env`를 통해 전달하세요:

```python theme={null}
from claude_agent_sdk import ClaudeAgentOptions

options = ClaudeAgentOptions(
    env={
        "API_TIMEOUT_MS": "120000",
        "CLAUDE_CODE_MAX_RETRIES": "2",
        "CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS": "120000",
    },
)
```

* `API_TIMEOUT_MS`: Anthropic 클라이언트의 요청별 타임아웃 (밀리초). 기본값 `600000`. 메인 루프 및 모든 서브에이전트에 적용됩니다.
* `CLAUDE_CODE_MAX_RETRIES`: 최대 API 재시도 횟수. 기본값 `10`, 최대 `15`로 제한됨. 각 재시도는 자체 `API_TIMEOUT_MS` 창을 가지므로 최악의 경우 실제 소요 시간은 대략 `API_TIMEOUT_MS × (CLAUDE_CODE_MAX_RETRIES + 1)` + 백오프(backoff)입니다. 긴 장애를 견뎌야 하는 무인 실행의 경우 `CLAUDE_CODE_RETRY_WATCHDOG=1`을 설정하세요: 용량 오류를 무제한 재시도하고, {/* min-version: 2.1.199 */}Claude Code v2.1.199부터는 다른 일시적 오류에 대한 기본값을 `300`으로 올리고 이 변수의 상한을 제거합니다.
* `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS`: `run_in_background`로 실행된 서브에이전트에 대한 정지 감시 장치(watchdog). 기본값 `600000`. 각 스트림 이벤트마다 재설정됩니다; 정지 시 서브에이전트를 중단하고, 작업을 실패로 표시하며, 부분 결과와 함께 부모에게 오류를 노출합니다. 동기식 서브에이전트에는 적용되지 않습니다.
* `CLAUDE_ENABLE_STREAM_WATCHDOG` 및 `CLAUDE_STREAM_IDLE_TIMEOUT_MS`: 헤더가 도착했으나 응답 본문 스트리밍이 멈춘 경우 요청을 중단합니다. 감시 장치는 모든 제공자에 대해 기본적으로 켜져 있습니다; 비활성화하려면 `CLAUDE_ENABLE_STREAM_WATCHDOG=0`을 설정하세요. `CLAUDE_STREAM_IDLE_TIMEOUT_MS` 기본값은 `300000`이며 해당 최소값으로 클램프됩니다. 중단된 요청은 일반 재시도 경로를 거칩니다.

### `OutputFormat`

구조화된 출력 검증을 위한 구성입니다. `ClaudeAgentOptions`의 `output_format` 필드에 `dict`로 전달하세요:

```python theme={null}
# Expected dict shape for output_format
{
    "type": "json_schema",
    "schema": {...},  # Your JSON Schema definition
}
```

| 필드 | 필수 여부 | 설명 |
| :------- | :------- | :------------------------------------------------- |
| `type` | 예 | JSON Schema 검증을 위해 반드시 `"json_schema"`여야 함 |
| `schema` | 예 | 출력 검증을 위한 JSON Schema 정의 |

### `SystemPromptPreset`

선택적 추가 내용과 함께 Claude Code의 프리셋 시스템 프롬프트를 사용하기 위한 구성입니다.

```python theme={null}
class SystemPromptPreset(TypedDict):
    type: Literal["preset"]
    preset: Literal["claude_code"]
    append: NotRequired[str]
    exclude_dynamic_sections: NotRequired[bool]
```

| 필드 | 필수 여부 | 설명 |
| :------------------------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type` | 예 | 프리셋 시스템 프롬프트를 사용하려면 반드시 `"preset"`이어야 함 |
| `preset` | 예 | Claude Code의 시스템 프롬프트를 사용하려면 반드시 `"claude_code"`여야 함 |
| `append` | 아니오 | 프리셋 시스템 프롬프트 뒤에 추가할 지침 |
| `exclude_dynamic_sections` | 아니오 | 작업 디렉토리, git-repo 플래그, 자동 메모리 경로 등 세션별 컨텍스트를 시스템 프롬프트에서 첫 번째 사용자 메시지로 이동. 사용자 및 시스템 간 프롬프트 캐시 재사용성 향상. [시스템 프롬프트 수정](/docs/en/agent-sdk/modifying-system-prompts#improve-prompt-caching-across-users-and-machines) 참고 |

### `SystemPromptFile`

문자열로 전달하는 대신 파일에서 커스텀 시스템 프롬프트를 로드하기 위한 구성입니다. SDK는 이를 CLI [`--system-prompt-file`](/docs/en/cli-reference#system-prompt-flags) 플래그에 매핑합니다. 프롬프트가 큰 경우 파일 형식을 사용하세요: SDK는 CLI 자식 프로세스 argv에 문자열 `system_prompt`를 전달하며, 이는 SDK가 API 요청을 전송하기 전에 OS 명령줄 길이 제한의 영향을 받습니다. Linux에서는 대략 128 KB를 초과하는 단일 인자가 프로세스 생성 시 `Argument list too long`으로 실패합니다. Windows에서는 전체 명령줄 상한이 대략 32 KB이므로 더 낮은 임계값에서 문자열 형식이 실패합니다.

```python theme={null}
class SystemPromptFile(TypedDict):
    type: Literal["file"]
    path: str
```

| 필드 | 필수 여부 | 설명 |
| :----- | :------- | :-------------------------------------------- |
| `type` | 예 | 디스크에서 프롬프트를 로드하려면 반드시 `"file"`이어야 함 |
| `path` | 예 | 시스템 프롬프트가 포함된 파일 경로 |

### `SettingSource`

SDK가 설정을 로드할 파일 시스템 기반 구성 소스를 제어합니다.

```python theme={null}
SettingSource = Literal["user", "project", "local"]
```

| 값 | 설명 | 위치 |
| :---------- | :---------------------------------------------- | :---------------------------- |
| `"user"` | 전역 사용자 설정 | `~/.claude/settings.json` |
| `"project"` | 공유 프로젝트 설정 (버전 관리됨) | `.claude/settings.json` |
| `"local"` | 로컬 프로젝트 설정 (버전 관리되지 않음) | `.claude/settings.local.json` |

#### 기본 동작

`setting_sources`가 생략되거나 `None`인 경우, `query()`는 Claude Code CLI와 동일한 파일 시스템 설정(사용자, 프로젝트 및 로컬)을 로드합니다. 엔드포인트 관리 정책은 모든 경우에 로드됩니다; 서버 관리 설정은 [지원 대상 구성](/docs/en/server-managed-settings#platform-availability)에서 조직 자격 증명으로 세션 인증 시 가져옵니다. 이 옵션과 상관없이 읽히는 입력 항목 및 비활성화 방법은 [settingSources가 제어하지 않는 항목](/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control)을 참고하세요.

#### setting_sources 사용 이유

**파일 시스템 설정 비활성화:**

```python theme={null}
# Do not load user, project, or local settings from disk
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Analyze this code",
        options=ClaudeAgentOptions(
            setting_sources=[]
        ),
    ):
        print(message)


asyncio.run(main())
```

<Note>
  Python SDK 0.1.59 및 이전 버전에서는 빈 목록이 옵션을 생략한 것과 동일하게 처리되었기 때문에 `setting_sources=[]`가 파일 시스템 설정을 비활성화하지 않았습니다. 빈 목록이 적용되도록 하려면 최신 릴리스로 업그레이드하세요. TypeScript SDK는 영향받지 않습니다.
</Note>

**모든 파일 시스템 설정 명시적 로드:**

```python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Analyze this code",
        options=ClaudeAgentOptions(
            setting_sources=["user", "project", "local"]
        ),
    ):
        print(message)


asyncio.run(main())
```

**특정 설정 소스만 로드:**

```python theme={null}
# Load only project settings, ignore user and local
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Run CI checks",
        options=ClaudeAgentOptions(
            setting_sources=["project"]  # Only .claude/settings.json
        ),
    ):
        print(message)


asyncio.run(main())
```

**테스트 및 CI 환경:**

```python theme={null}
# Ensure consistent behavior in CI by excluding local settings
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Run tests",
        options=ClaudeAgentOptions(
            setting_sources=["project"],  # Only team-shared settings
            permission_mode="bypassPermissions",
        ),
    ):
        print(message)


asyncio.run(main())
```

**SDK 전용 애플리케이션:**

```python theme={null}
# Define everything programmatically.
# Pass [] to opt out of filesystem setting sources.
import asyncio
from claude_agent_sdk import AgentDefinition, ClaudeAgentOptions, query


async def main():
    async for message in query(
        prompt="Review this PR",
        options=ClaudeAgentOptions(
            setting_sources=[],
            agents={
                "code-reviewer": AgentDefinition(
                    description="Reviews code changes",
                    prompt="You are a code reviewer. Report issues in the diff.",
                ),
            },
            allowed_tools=["Read", "Grep", "Glob"],
        ),
    ):
        print(message)


asyncio.run(main())
```

### `AssistantMessageError`

어시스턴트 메시지의 가능한 오류 타입들입니다.

```python theme={null}
AssistantMessageError = Literal[
    "authentication_failed",
    "billing_error",
    "rate_limit",
    "invalid_request",
    "server_error",
    "max_output_tokens",
    "unknown",
]
```

### `SystemMessage`

메타데이터가 포함된 시스템 메시지입니다.

```python theme={null}
@dataclass
class SystemMessage:
    subtype: str
    data: dict[str, Any]
```

### `ResultMessage`

비용 및 사용량 정보가 포함된 최종 결과 메시지입니다.

```python theme={null}
@dataclass
class ResultMessage:
    subtype: str
    duration_ms: int
    duration_api_ms: int
    is_error: bool
    num_turns: int
    session_id: str
    stop_reason: str | None = None
    total_cost_usd: float | None = None
    usage: dict[str, Any] | None = None
    result: str | None = None
    structured_output: Any = None
    model_usage: dict[str, Any] | None = None
    permission_denials: list[Any] | None = None
    deferred_tool_use: DeferredToolUse | None = None
    errors: list[str] | None = None
    api_error_status: int | None = None
    uuid: str | None = None
```

`subtype` 필드는 다른 어떤 필드들이 채워지는지를 결정합니다. `"success"`, `"error_during_execution"`, `"error_max_turns"`, `"error_max_budget_usd"`, 또는 `"error_max_structured_output_retries"` 중 하나입니다. Python dataclass는 모든 변형(variant)을 하나의 형태로 단순화하므로 반환된 하위 타입에 해당하지 않는 필드는 `None`입니다.

대화가 오류로 끝날 때 몇몇 필드가 진단 세부 정보를 전달합니다:

* `is_error`: 대화가 오류 상태로 끝났을 때 `True`. `error_*` 하위 타입에서는 항상 `True`. `subtype="success"`에서는 최종 모델 요청이 실패했을 때 `True`(에이전트 루프는 완료되었으나 마지막 API 호출이 오류를 반환함을 의미).
* `api_error_status`: 종료 API 오류의 HTTP 상태 코드. 해당 없이 턴이 종료된 경우 `None`. `subtype="success"`에서만 채워짐.
* `result`: `subtype="success"`일 때 최종 어시스턴트 메시지의 텍스트, 또는 `error_*` 하위 타입일 때 `None`. `subtype="success"`이고 `is_error=True`일 때 사용 가능한 API 오류 문자열을 담고 있지만 비어있을 수 있으므로 자세한 내용은 `api_error_status` 및 이전 `AssistantMessage` 내용을 확인하세요.
* `errors`: max-turns 메시지와 같은 루프 수준 오류 문자열. `error_*` 하위 타입에서만 채워짐.

`usage` 딕셔너리에는 존재하는 경우 다음 키들이 포함됩니다:

| 키 | 타입 | 설명 |
| ----------------------------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input_tokens` | `int` | 최상위 에이전트 루프가 소비한 입력 토큰. [서브에이전트 토큰은 포함되지 않음](/docs/en/agent-sdk/cost-tracking#get-the-total-cost-of-a-query); 전체 트리 계정을 위해 `model_usage`를 사용하세요. |
| `output_tokens` | `int` | 최상위 에이전트 루프가 생성한 출력 토큰. 서브에이전트 토큰은 포함되지 않음. |
| `cache_creation_input_tokens` | `int` | 새 캐시 항목을 만드는 데 사용된 토큰. |
| `cache_read_input_tokens` | `int` | 기존 캐시 항목에서 읽은 토큰. |

`model_usage` 딕셔너리는 모델 이름을 모델별 사용량에 매핑합니다. 내부 딕셔너리 키는 하위 CLI 프로세스에서 변경 없이 전달되어 TypeScript [`ModelUsage`](/docs/en/agent-sdk/typescript#modelusage) 타입과 일치하므로 camelCase를 사용합니다:

| 키 | 타입 | 설명 |
| -------------------------- | ------- | -------------------------------------------------------------------------------------------------------- |
| `inputTokens` | `int` | 이 모델의 입력 토큰. |
| `outputTokens` | `int` | 이 모델의 출력 토큰. |
| `cacheReadInputTokens` | `int` | 이 모델의 캐시 읽기 토큰. |
| `cacheCreationInputTokens` | `int` | 이 모델의 캐시 생성 토큰. |
| `webSearchRequests` | `int` | 이 모델이 수행한 웹 검색 요청 수. |
| `costUSD` | `float` | 클라이언트 측에서 계산된 이 모델의 예상 비용(USD). 요금 관련 주의사항은 [비용 및 사용량 추적](/docs/en/agent-sdk/cost-tracking) 참고. |
| `contextWindow` | `int` | 이 모델의 컨텍스트 창 크기. |
| `maxOutputTokens` | `int` | 이 모델의 최대 출력 토큰 제한. |

### `StreamEvent`

스트리밍 중 부분 메시지 업데이트를 위한 스트림 이벤트입니다. `ClaudeAgentOptions`에서 `include_partial_messages=True`일 때만 수신됩니다. `from claude_agent_sdk.types import StreamEvent`를 통해 가져옵니다.

```python theme={null}
@dataclass
class StreamEvent:
    uuid: str
    session_id: str
    event: dict[str, Any]  # The raw Claude API stream event
    parent_tool_use_id: str | None = None
```

| 필드 | 타입 | 설명 |
| :------------------- | :--------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `uuid` | `str` | 이 이벤트의 고유 식별자 |
| `session_id` | `str` | 세션 식별자 |
| `event` | `dict[str, Any]` | 원시 Claude API 스트림 이벤트 데이터 |
| `parent_tool_use_id` | `str \| None` | 항상 `None`. 스트림 이벤트는 메인 세션에 대해서만 발송됨. 서브에이전트 귀속을 위해서는 [`AssistantMessage`](#assistantmessage)와 같은 전체 메시지를 사용하세요 |

### `RateLimitEvent`

속도 제한(rate limit) 상태가 변경될 때 발송됩니다 (예: `"allowed"`에서 `"allowed_warning"`으로). 하드 제한에 도달하기 전에 사용자에게 경고하거나, 상태가 `"rejected"`일 때 대기(back off)하는 데 사용하세요.

```python theme={null}
@dataclass
class RateLimitEvent:
    rate_limit_info: RateLimitInfo
    uuid: str
    session_id: str
```

| 필드 | 타입 | 설명 |
| :---------------- | :-------------------------------- | :----------------------- |
| `rate_limit_info` | [`RateLimitInfo`](#ratelimitevent) | 현재 속도 제한 상태 |
| `uuid` | `str` | 고유 이벤트 식별자 |
| `session_id` | `str` | 세션 식별자 |

### `RateLimitInfo`

[`RateLimitEvent`](#ratelimitevent)가 전달하는 속도 제한 상태입니다.

```python theme={null}
RateLimitStatus = Literal["allowed", "allowed_warning", "rejected"]
RateLimitType = Literal[
    "five_hour", "seven_day", "seven_day_opus", "seven_day_sonnet", "overage"
]


@dataclass
class RateLimitInfo:
    status: RateLimitStatus
    resets_at: int | None = None
    rate_limit_type: RateLimitType | None = None
    utilization: float | None = None
    overage_status: RateLimitStatus | None = None
    overage_resets_at: int | None = None
    overage_disabled_reason: str | None = None
    raw: dict[str, Any] = field(default_factory=dict)
```

| 필드 | 타입 | 설명 |
| :------------------------ | :------------------------ | :---------------------------------------------------------------------------------------------------- |
| `status` | `RateLimitStatus` | 현재 상태. `"allowed_warning"`은 제한에 접근함을, `"rejected"`는 제한에 도달했음을 의미 |
| `resets_at` | `int \| None` | 속도 제한 창이 재설정되는 Unix 타임스탬프 |
| `rate_limit_type` | `RateLimitType \| None` | 적용되는 속도 제한 창 종류 |
| `utilization` | `float \| None` | 소비된 속도 제한 비율 (0.0 ~ 1.0) |
| `overage_status` | `RateLimitStatus \| None` | 해당되는 경우 종량제 초과 사용 상태 |
| `overage_resets_at` | `int \| None` | 초과 사용 창이 재설정되는 Unix 타임스탬프 |
| `overage_disabled_reason` | `str \| None` | 상태가 `"rejected"`일 때 초과 사용을 이용할 수 없는 이유 |
| `raw` | `dict[str, Any]` | 위에서 모델링되지 않은 필드를 포함하여 CLI에서 온 전체 원시 dict |

### `TaskStartedMessage`

백그라운드 작업이 시작될 때 발송됩니다. 백그라운드 작업은 메인 턴 외부에서 추적되는 모든 작업입니다: 백그라운드 처리된 Bash 명령, [Monitor](#monitor) 감시, Agent 도구를 통해 생성된 서브에이전트, 또는 원격 에이전트. `task_type` 필드가 어떤 종류인지 알려줍니다. 이 명칭은 `Task`에서 `Agent`로의 도구 이름 변경과 무관합니다.

```python theme={null}
@dataclass
class TaskStartedMessage(SystemMessage):
    task_id: str
    description: str
    uuid: str
    session_id: str
    tool_use_id: str | None = None
    task_type: str | None = None
```

| 필드 | 타입 | 설명 |
| :------------ | :------------ | :-------------------------------------------------------------------------------------------------------------------------- |
| `task_id` | `str` | 작업의 고유 식별자 |
| `description` | `str` | 작업에 대한 설명 |
| `uuid` | `str` | 고유 메시지 식별자 |
| `session_id` | `str` | 세션 식별자 |
| `tool_use_id` | `str \| None` | 연관된 도구 사용 ID |
| `task_type` | `str \| None` | 백그라운드 작업 종류: 백그라운드 Bash 및 Monitor 감시의 경우 `"local_bash"`, `"local_agent"`, 또는 `"remote_agent"` |

### `TaskUsage`

백그라운드 작업의 토큰 및 타이밍 데이터입니다.

```python theme={null}
class TaskUsage(TypedDict):
    total_tokens: int
    tool_uses: int
    duration_ms: int
```

### `TaskProgressMessage`

실행 중인 백그라운드 작업의 진행률 업데이트와 함께 주기적으로 발송됩니다.

```python theme={null}
@dataclass
class TaskProgressMessage(SystemMessage):
    task_id: str
    description: str
    usage: TaskUsage
    uuid: str
    session_id: str
    tool_use_id: str | None = None
    last_tool_name: str | None = None
```

| 필드 | 타입 | 설명 |
| :--------------- | :------------ | :---------------------------------- |
| `task_id` | `str` | 작업의 고유 식별자 |
| `description` | `str` | 현재 상태 설명 |
| `usage` | `TaskUsage` | 지금까지 이 작업의 토큰 사용량 |
| `uuid` | `str` | 고유 메시지 식별자 |
| `session_id` | `str` | 세션 식별자 |
| `tool_use_id` | `str \| None` | 연관된 도구 사용 ID |
| `last_tool_name` | `str \| None` | 작업이 마지막으로 사용한 도구 이름 |

### `TaskNotificationMessage`

백그라운드 작업이 완료되거나 실패하거나 중지될 때 발송됩니다. 백그라운드 작업에는 `run_in_background` Bash 명령, Monitor 감시, 및 백그라운드 서브에이전트가 포함됩니다.

```python theme={null}
@dataclass
class TaskNotificationMessage(SystemMessage):
    task_id: str
    status: TaskNotificationStatus  # "completed" | "failed" | "stopped"
    output_file: str
    summary: str
    uuid: str
    session_id: str
    tool_use_id: str | None = None
    usage: TaskUsage | None = None
```

| 필드 | 타입 | 설명 |
| :------------ | :----------------------- | :----------------------------------------------- |
| `task_id` | `str` | 작업의 고유 식별자 |
| `status` | `TaskNotificationStatus` | `"completed"`, `"failed"`, 또는 `"stopped"` 중 하나 |
| `output_file` | `str` | 작업 출력 파일 경로 |
| `summary` | `str` | 작업 결과 요약 |
| `uuid` | `str` | 고유 메시지 식별자 |
| `session_id` | `str` | 세션 식별자 |
| `tool_use_id` | `str \| None` | 연관된 도구 사용 ID |
| `usage` | `TaskUsage \| None` | 작업의 최종 토큰 사용량 |

## 컨텐츠 블록 타입 (Content Block Types)

### `ContentBlock`

모든 컨텐츠 블록의 유니온 타입입니다.

```python theme={null}
ContentBlock = TextBlock | ThinkingBlock | ToolUseBlock | ToolResultBlock
```

### `TextBlock`

텍스트 컨텐츠 블록입니다.

```python theme={null}
@dataclass
class TextBlock:
    text: str
```

### `ThinkingBlock`

사고 컨텐츠 블록입니다 (사고 기능이 있는 모델용).

```python theme={null}
@dataclass
class ThinkingBlock:
    thinking: str
    signature: str
```

### `ToolUseBlock`

도구 사용 요청 블록입니다.

```python theme={null}
@dataclass
class ToolUseBlock:
    id: str
    name: str
    input: dict[str, Any]
```

### `ToolResultBlock`

도구 실행 결과 블록입니다.

```python theme={null}
@dataclass
class ToolResultBlock:
    tool_use_id: str
    content: str | list[dict[str, Any]] | None = None
    is_error: bool | None = None
```

## 오류 타입 (Error Types)

### `ClaudeSDKError`

모든 SDK 오류의 기본 예외 클래스입니다.

```python theme={null}
class ClaudeSDKError(Exception):
    """Base error for Claude SDK."""
```

### `CLINotFoundError`

Claude Code CLI가 설치되지 않았거나 찾을 수 없을 때 발생합니다.

```python theme={null}
class CLINotFoundError(CLIConnectionError):
    def __init__(
        self, message: str = "Claude Code not found", cli_path: str | None = None
    ):
        """
        Args:
            message: Error message (default: "Claude Code not found")
            cli_path: Optional path to the CLI that was not found
        """
```

### `CLIConnectionError`

Claude Code 연결 실패 시 발생합니다.

```python theme={null}
class CLIConnectionError(ClaudeSDKError):
    """Failed to connect to Claude Code."""
```

### `ProcessError`

Claude Code 프로세스가 실패할 때 발생합니다.

```python theme={null}
class ProcessError(ClaudeSDKError):
    def __init__(
        self, message: str, exit_code: int | None = None, stderr: str | None = None
    ):
        self.exit_code = exit_code
        self.stderr = stderr
```

### `CLIJSONDecodeError`

JSON 파싱 실패 시 발생합니다.

```python theme={null}
class CLIJSONDecodeError(ClaudeSDKError):
    def __init__(self, line: str, original_error: Exception):
        """
        Args:
            line: The line that failed to parse
            original_error: The original JSON decode exception
        """
        self.line = line
        self.original_error = original_error
```

## 후크 타입 (Hook Types)

예제 및 일반적인 패턴이 포함된 후크 사용 종합 가이드는 [후크 가이드](/docs/en/agent-sdk/hooks)를 참고하세요.

### `HookEvent`

지원되는 후크 이벤트 타입들입니다.

```python theme={null}
HookEvent = Literal[
    "PreToolUse",  # Called before tool execution
    "PostToolUse",  # Called after tool execution
    "PostToolUseFailure",  # Called when a tool execution fails
    "UserPromptSubmit",  # Called when user submits a prompt
    "Stop",  # Called when stopping execution
    "SubagentStop",  # Called when a subagent stops
    "PreCompact",  # Called before message compaction
    "Notification",  # Called for notification events
    "SubagentStart",  # Called when a subagent starts
    "PermissionRequest",  # Called when a permission decision is needed
]
```

<Note>
  TypeScript SDK는 Python에서 아직 제공되지 않는 추가 후크 이벤트를 지원합니다. SDK별 지원 여부는 [사용 가능한 후크 표](/docs/en/agent-sdk/hooks#available-hooks)를 참고하세요.
</Note>

### `HookCallback`

후크 콜백 함수를 위한 타입 정의입니다.

```python theme={null}
HookCallback = Callable[[HookInput, str | None, HookContext], Awaitable[HookJSONOutput]]
```

매개변수:

* `input`: `hook_event_name` 기반의 식별된 유니온(discriminated unions)을 갖는 강력한 타입의 후크 입력 ([`HookInput`](#hookinput) 참고)
* `tool_use_id`: 선택적 도구 사용 식별자 (도구 관련 후크의 경우)
* `context`: 추가 정보가 포함된 후크 컨텍스트

다음 항목을 포함할 수 있는 [`HookJSONOutput`](#hookjsonoutput)을 반환합니다:

* `decision`: 작업을 차단하려면 `"block"`
* `systemMessage`: 사용자에게 표시되는 경고 메시지
* `hookSpecificOutput`: 후크 전용 출력 데이터

### `HookContext`

후크 콜백에 전달되는 컨텍스트 정보입니다.

```python theme={null}
class HookContext(TypedDict):
    signal: Any | None  # Future: abort signal support
```

### `HookMatcher`

후크를 특정 이벤트나 도구에 매칭하기 위한 구성입니다.

```python theme={null}
@dataclass
class HookMatcher:
    matcher: str | None = (
        None  # Tool name or pattern to match (e.g., "Bash", "Write|Edit")
    )
    hooks: list[HookCallback] = field(
        default_factory=list
    )  # List of callbacks to execute
    timeout: float | None = (
        None  # Timeout in seconds. When omitted, the per-event default applies
    )
```

### `HookInput`

모든 후크 입력 타입의 유니온 타입입니다. 실제 타입은 `hook_event_name` 필드에 따라 결정됩니다.

```python theme={null}
HookInput = (
    PreToolUseHookInput
    | PostToolUseHookInput
    | PostToolUseFailureHookInput
    | UserPromptSubmitHookInput
    | StopHookInput
    | SubagentStopHookInput
    | PreCompactHookInput
    | NotificationHookInput
    | SubagentStartHookInput
    | PermissionRequestHookInput
)
```

### `BaseHookInput`

모든 후크 입력 타입에 존재하는 기본 필드들입니다.

```python theme={null}
class BaseHookInput(TypedDict):
    session_id: str
    transcript_path: str
    cwd: str
    permission_mode: NotRequired[str]
```

| 필드 | 타입 | 설명 |
| :---------------- | :--------------- | :---------------------------------- |
| `session_id` | `str` | 현재 세션 식별자 |
| `transcript_path` | `str` | 세션 트랜스크립트 파일 경로 |
| `cwd` | `str` | 현재 작업 디렉토리 |
| `permission_mode` | `str` (선택적) | 현재 권한 모드 |

### `PreToolUseHookInput`

`PreToolUse` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class PreToolUseHookInput(BaseHookInput):
    hook_event_name: Literal["PreToolUse"]
    tool_name: str
    tool_input: dict[str, Any]
    tool_use_id: str
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| 필드 | 타입 | 설명 |
| :---------------- | :---------------------- | :----------------------------------------------------------------- |
| `hook_event_name` | `Literal["PreToolUse"]` | 항상 "PreToolUse" |
| `tool_name` | `str` | 실행 직전 도구 이름 |
| `tool_input` | `dict[str, Any]` | 도구 입력 매개변수 |
| `tool_use_id` | `str` | 이 도구 사용의 고유 식별자 |
| `agent_id` | `str` (선택적) | 서브에이전트 식별자 (서브에이전트 내부에서 후크가 실행될 때 존재) |
| `agent_type` | `str` (선택적) | 서브에이전트 타입 (서브에이전트 내부에서 후크가 실행될 때 존재) |

### `PostToolUseHookInput`

`PostToolUse` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class PostToolUseHookInput(BaseHookInput):
    hook_event_name: Literal["PostToolUse"]
    tool_name: str
    tool_input: dict[str, Any]
    tool_response: Any
    tool_use_id: str
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| 필드 | 타입 | 설명 |
| :---------------- | :----------------------- | :----------------------------------------------------------------- |
| `hook_event_name` | `Literal["PostToolUse"]` | 항상 "PostToolUse" |
| `tool_name` | `str` | 실행된 도구 이름 |
| `tool_input` | `dict[str, Any]` | 사용된 입력 매개변수 |
| `tool_response` | `Any` | 도구 실행 응답 |
| `tool_use_id` | `str` | 이 도구 사용의 고유 식별자 |
| `agent_id` | `str` (선택적) | 서브에이전트 식별자 (서브에이전트 내부에서 후크가 실행될 때 존재) |
| `agent_type` | `str` (선택적) | 서브에이전트 타입 (서브에이전트 내부에서 후크가 실행될 때 존재) |

### `PostToolUseFailureHookInput`

`PostToolUseFailure` 후크 이벤트를 위한 입력 데이터입니다. 도구 실행이 실패할 때 호출됩니다.

```python theme={null}
class PostToolUseFailureHookInput(BaseHookInput):
    hook_event_name: Literal["PostToolUseFailure"]
    tool_name: str
    tool_input: dict[str, Any]
    tool_use_id: str
    error: str
    is_interrupt: NotRequired[bool]
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| 필드 | 타입 | 설명 |
| :---------------- | :------------------------------ | :----------------------------------------------------------------- |
| `hook_event_name` | `Literal["PostToolUseFailure"]` | 항상 "PostToolUseFailure" |
| `tool_name` | `str` | 실패한 도구 이름 |
| `tool_input` | `dict[str, Any]` | 사용된 입력 매개변수 |
| `tool_use_id` | `str` | 이 도구 사용의 고유 식별자 |
| `error` | `str` | 실패한 실행의 오류 메시지 |
| `is_interrupt` | `bool` (선택적) | 실패 원인이 중단(interrupt)인지 여부 |
| `agent_id` | `str` (선택적) | 서브에이전트 식별자 (서브에이전트 내부에서 후크가 실행될 때 존재) |
| `agent_type` | `str` (선택적) | 서브에이전트 타입 (서브에이전트 내부에서 후크가 실행될 때 존재) |

### `UserPromptSubmitHookInput`

`UserPromptSubmit` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class UserPromptSubmitHookInput(BaseHookInput):
    hook_event_name: Literal["UserPromptSubmit"]
    prompt: str
```

| 필드 | 타입 | 설명 |
| :---------------- | :---------------------------- | :-------------------------- |
| `hook_event_name` | `Literal["UserPromptSubmit"]` | 항상 "UserPromptSubmit" |
| `prompt` | `str` | 사용자가 전송한 프롬프트 |

### `StopHookInput`

`Stop` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class StopHookInput(BaseHookInput):
    hook_event_name: Literal["Stop"]
    stop_hook_active: bool
```

| 필드 | 타입 | 설명 |
| :----------------- | :---------------- | :------------------------------ |
| `hook_event_name` | `Literal["Stop"]` | 항상 "Stop" |
| `stop_hook_active` | `bool` | 정지 후크 활성화 여부 |

### `SubagentStopHookInput`

`SubagentStop` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class SubagentStopHookInput(BaseHookInput):
    hook_event_name: Literal["SubagentStop"]
    stop_hook_active: bool
    agent_id: str
    agent_transcript_path: str
    agent_type: str
```

| 필드 | 타입 | 설명 |
| :---------------------- | :------------------------ | :------------------------------------- |
| `hook_event_name` | `Literal["SubagentStop"]` | 항상 "SubagentStop" |
| `stop_hook_active` | `bool` | 정지 후크 활성화 여부 |
| `agent_id` | `str` | 서브에이전트의 고유 식별자 |
| `agent_transcript_path` | `str` | 서브에이전트 트랜스크립트 파일 경로 |
| `agent_type` | `str` | 서브에이전트 타입 |

### `PreCompactHookInput`

`PreCompact` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class PreCompactHookInput(BaseHookInput):
    hook_event_name: Literal["PreCompact"]
    trigger: Literal["manual", "auto"]
    custom_instructions: str | None
```

| 필드 | 타입 | 설명 |
| :-------------------- | :-------------------------- | :--------------------------------- |
| `hook_event_name` | `Literal["PreCompact"]` | 항상 "PreCompact" |
| `trigger` | `Literal["manual", "auto"]` | 수축(compaction)을 트리거한 원인 |
| `custom_instructions` | `str \| None` | 수축을 위한 커스텀 지침 |

### `NotificationHookInput`

`Notification` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class NotificationHookInput(BaseHookInput):
    hook_event_name: Literal["Notification"]
    message: str
    title: NotRequired[str]
    notification_type: str
```

| 필드 | 타입 | 설명 |
| :------------------ | :------------------------ | :--------------------------- |
| `hook_event_name` | `Literal["Notification"]` | 항상 "Notification" |
| `message` | `str` | 알림 메시지 내용 |
| `title` | `str` (선택적) | 알림 제목 |
| `notification_type` | `str` | 알림 종류 |

### `SubagentStartHookInput`

`SubagentStart` 후크 이벤트를 위한 입력 데이터입니다.

```python theme={null}
class SubagentStartHookInput(BaseHookInput):
    hook_event_name: Literal["SubagentStart"]
    agent_id: str
    agent_type: str
```

| 필드 | 타입 | 설명 |
| :---------------- | :------------------------- | :--------------------------------- |
| `hook_event_name` | `Literal["SubagentStart"]` | 항상 "SubagentStart" |
| `agent_id` | `str` | 서브에이전트의 고유 식별자 |
| `agent_type` | `str` | 서브에이전트 타입 |

### `PermissionRequestHookInput`

`PermissionRequest` 후크 이벤트를 위한 입력 데이터입니다. 후크가 프로그래밍 방식으로 권한 결정을 처리할 수 있도록 합니다.

```python theme={null}
class PermissionRequestHookInput(BaseHookInput):
    hook_event_name: Literal["PermissionRequest"]
    tool_name: str
    tool_input: dict[str, Any]
    permission_suggestions: NotRequired[list[Any]]
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| 필드 | 타입 | 설명 |
| :----------------------- | :----------------------------- | :----------------------------------------------------------------- |
| `hook_event_name` | `Literal["PermissionRequest"]` | 항상 "PermissionRequest" |
| `tool_name` | `str` | 권한을 요청하는 도구 이름 |
| `tool_input` | `dict[str, Any]` | 도구 입력 매개변수 |
| `permission_suggestions` | `list[Any]` (선택적) | CLI의 제안된 권한 업데이트 목록 |
| `agent_id` | `str` (선택적) | 서브에이전트 식별자 (서브에이전트 내부에서 후크가 실행될 때 존재) |
| `agent_type` | `str` (선택적) | 서브에이전트 타입 (서브에이전트 내부에서 후크가 실행될 때 존재) |

### `HookJSONOutput`

후크 콜백 반환값을 위한 유니온 타입입니다.

```python theme={null}
HookJSONOutput = AsyncHookJSONOutput | SyncHookJSONOutput
```

#### `SyncHookJSONOutput`

제어 및 결정 필드가 포함된 동기식 후크 출력입니다.

```python theme={null}
class SyncHookJSONOutput(TypedDict):
    # Control fields
    continue_: NotRequired[bool]  # Whether to proceed (default: True)
    suppressOutput: NotRequired[bool]  # Hide stdout from transcript
    stopReason: NotRequired[str]  # Message when continue is False

    # Decision fields
    decision: NotRequired[Literal["block"]]
    systemMessage: NotRequired[str]  # Warning message for user
    reason: NotRequired[str]  # Feedback for Claude

    # Hook-specific output
    hookSpecificOutput: NotRequired[HookSpecificOutput]
```

<Note>
  Python 코드에서는 `continue_` (밑줄 포함)를 사용하세요. CLI로 전송될 때 `continue`로 자동 변환됩니다.
</Note>

#### `HookSpecificOutput`

후크 이벤트 이름과 이벤트별 필드가 포함된 `TypedDict`입니다. 세부 형태는 `hookEventName` 값에 따라 결정됩니다. 후크 이벤트별 사용 가능한 필드에 대한 자세한 내용은 [후크로 실행 제어하기](/docs/en/agent-sdk/hooks#outputs)를 참고하세요.

이벤트별 출력 타입들의 식별된 유니온입니다. `hookEventName` 필드에 의해 유효한 필드가 결정됩니다.

```python theme={null}
class PreToolUseHookSpecificOutput(TypedDict):
    hookEventName: Literal["PreToolUse"]
    permissionDecision: NotRequired[Literal["allow", "deny", "ask", "defer"]]
    permissionDecisionReason: NotRequired[str]
    updatedInput: NotRequired[dict[str, Any]]
    additionalContext: NotRequired[str]


class PostToolUseHookSpecificOutput(TypedDict):
    hookEventName: Literal["PostToolUse"]
    additionalContext: NotRequired[str]
    updatedToolOutput: NotRequired[Any]
    updatedMCPToolOutput: NotRequired[Any]  # Deprecated: use updatedToolOutput, which works for all tools


class PostToolUseFailureHookSpecificOutput(TypedDict):
    hookEventName: Literal["PostToolUseFailure"]
    additionalContext: NotRequired[str]


class UserPromptSubmitHookSpecificOutput(TypedDict):
    hookEventName: Literal["UserPromptSubmit"]
    additionalContext: NotRequired[str]


class NotificationHookSpecificOutput(TypedDict):
    hookEventName: Literal["Notification"]
    additionalContext: NotRequired[str]


class SubagentStartHookSpecificOutput(TypedDict):
    hookEventName: Literal["SubagentStart"]
    additionalContext: NotRequired[str]


class PermissionRequestHookSpecificOutput(TypedDict):
    hookEventName: Literal["PermissionRequest"]
    decision: dict[str, Any]


HookSpecificOutput = (
    PreToolUseHookSpecificOutput
    | PostToolUseHookSpecificOutput
    | PostToolUseFailureHookSpecificOutput
    | UserPromptSubmitHookSpecificOutput
    | NotificationHookSpecificOutput
    | SubagentStartHookSpecificOutput
    | PermissionRequestHookSpecificOutput
)
```

#### `AsyncHookJSONOutput`

후크 실행을 지연시키는 비동기 후크 출력입니다.

```python theme={null}
class AsyncHookJSONOutput(TypedDict):
    async_: Literal[True]  # Set to True to defer execution
    asyncTimeout: NotRequired[int]  # Timeout in milliseconds
```

<Note>
  Python 코드에서는 `async_` (밑줄 포함)를 사용하세요. CLI로 전송될 때 `async`로 자동 변환됩니다.
</Note>

### 후크 사용 예제

다음 예제는 두 개의 후크를 등록합니다: 하나는 `rm -rf /`와 같이 위험한 bash 명령어를 차단하고, 다른 하나는 감사를 위해 모든 도구 사용을 기록합니다. 보안 후크는 (`matcher`를 통해) Bash 명령에서만 실행되고, 로깅 후크는 모든 도구에서 실행됩니다.

```python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, HookContext
from typing import Any


async def validate_bash_command(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Validate and potentially block dangerous bash commands."""
    if input_data["tool_name"] == "Bash":
        command = input_data["tool_input"].get("command", "")
        if "rm -rf /" in command:
            return {
                "hookSpecificOutput": {
                    "hookEventName": "PreToolUse",
                    "permissionDecision": "deny",
                    "permissionDecisionReason": "Dangerous command blocked",
                }
            }
    return {}


async def log_tool_use(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Log all tool usage for auditing."""
    print(f"Tool used: {input_data.get('tool_name')}")
    return {}


options = ClaudeAgentOptions(
    hooks={
        "PreToolUse": [
            HookMatcher(
                matcher="Bash", hooks=[validate_bash_command], timeout=120
            ),  # 2 min for validation
            HookMatcher(
                hooks=[log_tool_use]
            ),  # Applies to all tools (per-event default timeout)
        ],
        "PostToolUse": [HookMatcher(hooks=[log_tool_use])],
    }
)

async def main():
    async for message in query(prompt="Analyze this codebase", options=options):
        print(message)


asyncio.run(main())
```

## 도구 입력/출력 타입 (Tool Input/Output Types)

내장된 모든 Claude Code 도구의 입력/출력 스키마 문서입니다. Python SDK가 이들을 타입으로 직접 내보내지는 않지만, 메시지 내 도구 입력 및 출력 구조를 나타냅니다.

### Agent

**도구 이름:** `Agent`. 이전 이름인 `Task`도 별칭으로 허용되며, 초기화 [`SystemMessage`](#systemmessage)의 `tools` 목록은 하위 호환성을 위해 이 도구를 `Task`로 보고합니다.

**입력:**

```python theme={null}
{
    "description": str,  # A short (3-5 word) description of the task
    "prompt": str,  # The task for the agent to perform
    "subagent_type": str | None,  # The type of specialized agent to use
    "model": "sonnet" | "opus" | "haiku" | "fable" | None,  # Model override for this agent
    "run_in_background": bool | None,  # Agents run in the background by default; set to False to run synchronously
    "name": str | None,  # Name for the spawned agent
    "team_name": str | None,  # Deprecated; ignored
    "mode": "acceptEdits" | "auto" | "bypassPermissions" | "default" | "dontAsk" | "plan" | None,  # Deprecated; ignored. Subagents inherit the parent session's permission mode; agent-definition frontmatter may override it
    "isolation": "worktree" | "remote" | None,  # Isolation mode for the agent's changes
}
```

복잡한 다단계 작업을 자율적으로 처리하도록 새 에이전트를 시작합니다.

**출력 (상태: `"completed"`):**

```python theme={null}
{
    "status": "completed",
    "agentId": str,  # ID of the agent that ran
    "agentType": str | None,  # The subagent type that handled the task
    "content": [  # Result content blocks
        {
            "type": "text",
            "text": str,
            "citations": list | None,
        }
    ],
    "resolvedModel": str | None,  # Model the subagent started on
    "modelsUsed": list[str] | None,  # Models used in order, with consecutive repeats collapsed
    "totalToolUseCount": int,  # Number of tool calls the agent made
    "totalDurationMs": int,  # Execution duration in milliseconds
    "totalTokens": int,  # Total tokens used
    "usage": {  # Token usage statistics
        "input_tokens": int,
        "output_tokens": int,
        "cache_creation_input_tokens": int | None,
        "cache_read_input_tokens": int | None,
        "server_tool_use": {"web_search_requests": int, "web_fetch_requests": int} | None,
        "service_tier": str | None,
        "cache_creation": {"ephemeral_1h_input_tokens": int, "ephemeral_5m_input_tokens": int} | None,
        "inference_geo": str | None,
        "speed": str | None,
        "iterations": Any | None,
    },
    "toolStats": {  # Aggregate tool activity for the run
        "readCount": int,
        "searchCount": int,
        "bashCount": int,
        "editFileCount": int,
        "linesAdded": int,
        "linesRemoved": int,
        "otherToolCount": int,
        "frameCount": int | None,
    } | None,
    "prompt": str,  # The prompt the agent ran
    "worktreePath": str | None,  # Present for worktree-isolated runs
    "worktreeBranch": str | None,  # Present for worktree-isolated runs
}
```

**출력 (상태: `"async_launched"`):**

```python theme={null}
{
    "status": "async_launched",
    "isAsync": bool | None,  # True on background launches
    "agentId": str,  # ID of the launched agent
    "description": str,  # The task description
    "resolvedModel": str | None,  # Model in use at the backgrounding transition
    "modelsUsed": list[str] | None,  # Models used before backgrounding, in order, with consecutive repeats collapsed
    "prompt": str,  # The prompt the agent runs
    "outputFile": str,  # File path where the agent's output is written
    "canReadOutputFile": bool | None,  # Whether the output file can be read directly
}
```

**출력 (상태: `"remote_launched"`):**

```python theme={null}
{
    "status": "remote_launched",
    "taskId": str,  # ID of the remote task
    "sessionUrl": str,  # Link to the remote cloud session
    "description": str,  # The task description
    "prompt": str,  # The prompt the agent runs
    "outputFile": str,  # File path where the agent's output is written
}
```

서브에이전트의 결과를 반환합니다. 출력은 `status` 필드에 따라 식별됩니다: 완료된 작업은 `"completed"`, 백그라운드 작업은 `"async_launched"`, Claude Code가 원격 클라우드 세션에 발송한 작업은 `"remote_launched"`이며 `sessionUrl`은 해당 세션으로 링크되고 `taskId`는 작업을 식별합니다. 워크트리로 격리된 실행은 `completed` 변형에 `worktreePath` 및 `worktreeBranch`를 포함합니다.

`completed` 변형에서 `resolvedModel`은 서브에이전트가 시작된 모델을 명시하며,이는 [`availableModels`](/docs/en/model-config#restrict-model-selection) 또는 다른 재정의가 적용될 경우 요청된 `model` 입력과 다를 수 있습니다. {/* min-version: 2.1.174 */}이 필드는 Claude Code v2.1.174 이상이 필요합니다. `async_launched` 변형에서 `resolvedModel`은 에이전트가 백그라운드로 전환될 때 사용 중이던 모델을 지정하므로 백그라운드 전환 전에 발생한 스왑이 반영됩니다. 두 변형 모두의 `modelsUsed` 필드는 연이은 중복을 축소하고 순서대로 사용된 모델을 나열합니다; 모델이 실행 도중 교체되었을 때만 설정됩니다. {/* min-version: 2.1.212 */}`modelsUsed` 및 백그라운드 전환 시점 `resolvedModel` 동작에는 Claude Code v2.1.212 이상이 필요합니다.

### AskUserQuestion

**도구 이름:** `AskUserQuestion`

실행 중 사용자에게 명확성을 위한 질문을 합니다. 자세한 용법은 [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input#handle-clarifying-questions)를 참고하세요.

**입력:**

```python theme={null}
{
    "questions": [  # Questions to ask the user (1-4 questions)
        {
            "question": str,  # The complete question to ask the user
            "header": str,  # Very short label displayed as a chip/tag (max 12 chars)
            "options": [  # The available choices (2-4 options)
                {
                    "label": str,  # Display text for this option (1-5 words)
                    "description": str,  # Explanation of what this option means
                    "preview": str | None,  # Preview content rendered when the option is focused
                }
            ],
            "multiSelect": bool,  # Set to true to allow multiple selections
        }
    ],
    "answers": dict[str, str] | None,
    # User answers populated by the permission system. Multi-select
    # answers are a comma-joined string of the selected labels; a
    # list of labels is accepted on input and coerced to that form
    "annotations": dict[str, dict] | None,
    # Per-question annotations from the user, keyed by question text.
    # Each value can carry "preview" (the selected option's preview
    # content) and "notes" (free-text notes on the selection)
    "metadata": dict | None,  # Analytics metadata, such as {"source": "remember"}; not displayed to the user
}
```

**출력:**

```python theme={null}
{
    "questions": [  # The questions that were asked
        {
            "question": str,
            "header": str,
            "options": [{"label": str, "description": str, "preview": str | None}],
            "multiSelect": bool,
        }
    ],
    "answers": dict[str, str],  # Maps question text to answer string
    # Multi-select answers are comma-separated
    "response": str | None,
    # Freeform reply typed instead of answering the questions; when set,
    # Claude receives "The user responded: ..." in place of the answer list
    "annotations": dict[str, dict] | None,  # Per-question "preview" and "notes" from the user's selections
    "afkTimeoutMs": int | None,  # Set when the dialog auto-resolved after this many milliseconds of user inactivity; absent when the user answered
}
```

### Bash

**도구 이름:** `Bash`

**입력:**

```python theme={null}
{
    "command": str,  # The command to execute
    "timeout": int | None,  # Optional timeout in milliseconds (max 600000; higher values are clamped to the max)
    "description": str | None,  # Clear, concise description (5-10 words)
    "run_in_background": bool | None,  # Set to true to run in background
}
```

**출력:**

```python theme={null}
{
    "output": str,  # Combined stdout and stderr output
    "exitCode": int,  # Exit code of the command
    "killed": bool | None,  # Whether command was killed due to timeout
    "shellId": str | None,  # Shell ID for background processes
}
```

### Monitor

**도구 이름:** `Monitor`

백그라운드 소스를 실행하고 각 이벤트를 Claude에 전달하여 폴링 없이 응답할 수 있도록 합니다: `command`는 스크립트를 실행하고 stdout 줄당 하나의 이벤트를 발송하며, `ws`는 WebSocket을 열고 텍스트 프레임당 하나의 이벤트를 발송합니다. `command`나 `ws` 중 정확히 하나만 제공하세요.

Monitor가 명령을 실행할 때는 Bash와 동일한 권한 규칙을 따르며, WebSocket 감시는 별도로 승인 프롬프트를 표시합니다. {/* min-version: 2.1.195 */}`ws` 소스에는 Claude Code v2.1.195 이상이 필요합니다. 동작 및 제공자 이용 가능 여부는 [Monitor 도구 참조 문서](/docs/en/tools-reference#monitor-tool)를 참고하세요.

**입력:**

```python theme={null}
{
    "command": str | None,  # Shell script; each stdout line is an event, exit ends the watch
    "ws": dict | None,  # WebSocket source: {"url": str, "protocols": list[str] | None}; each text frame is an event
    "description": str,  # Short description shown in notifications
    "timeout_ms": int | None,  # Kill after this deadline (default 300000, max 3600000)
    "persistent": bool | None,  # Run for the lifetime of the session; stop with TaskStop
}
```

**출력:**

```python theme={null}
{
    "taskId": str,  # ID of the background monitor task
    "timeoutMs": int,  # Timeout deadline in milliseconds (0 when persistent)
    "persistent": bool | None,  # True when running until TaskStop or session end
}
```

### Edit

**도구 이름:** `Edit`

**입력:**

```python theme={null}
{
    "file_path": str,  # The absolute path to the file to modify
    "old_string": str,  # The text to replace
    "new_string": str,  # The text to replace it with
    "replace_all": bool | None,  # Replace all occurrences (default False)
}
```

**출력:**

```python theme={null}
{
    "message": str,  # Confirmation message
    "replacements": int,  # Number of replacements made
    "file_path": str,  # File path that was edited
}
```

### Read

**도구 이름:** `Read`

**입력:**

```python theme={null}
{
    "file_path": str,  # The absolute path to the file to read
    "offset": int | None,  # The line number to start reading from
    "limit": int | None,  # The number of lines to read
}
```

**출력 (텍스트 파일):**

```python theme={null}
{
    "content": str,  # File contents with line numbers
    "total_lines": int,  # Total number of lines in file
    "lines_returned": int,  # Lines actually returned
}
```

**출력 (이미지):**

```python theme={null}
{
    "image": str,  # Base64 encoded image data
    "mime_type": str,  # Image MIME type
    "file_size": int,  # File size in bytes
}
```

### Write

**도구 이름:** `Write`

**입력:**

```python theme={null}
{
    "file_path": str,  # The absolute path to the file to write
    "content": str,  # The content to write to the file
}
```

**출력:**

```python theme={null}
{
    "message": str,  # Success message
    "bytes_written": int,  # Number of bytes written
    "file_path": str,  # File path that was written
}
```

### Glob

**도구 이름:** `Glob`

**입력:**

```python theme={null}
{
    "pattern": str,  # The glob pattern to match files against
    "path": str | None,  # The directory to search in (defaults to cwd)
}
```

**출력:**

```python theme={null}
{
    "matches": list[str],  # Array of matching file paths
    "count": int,  # Number of matches found
    "search_path": str,  # Search directory used
}
```

### Grep

**도구 이름:** `Grep`

**입력:**

```python theme={null}
{
    "pattern": str,  # The regular expression pattern
    "path": str | None,  # File or directory to search in
    "glob": str | None,  # Glob pattern to filter files
    "type": str | None,  # File type to search
    "output_mode": str | None,  # "content", "files_with_matches", or "count"
    "-i": bool | None,  # Case insensitive search
    "-n": bool | None,  # Show line numbers
    "-B": int | None,  # Lines to show before each match
    "-A": int | None,  # Lines to show after each match
    "-C": int | None,  # Lines to show before and after
    "head_limit": int | None,  # Limit output to first N lines/entries
    "multiline": bool | None,  # Enable multiline mode
}
```

**출력 (content 모드):**

```python theme={null}
{
    "matches": [
        {
            "file": str,
            "line_number": int | None,
            "line": str,
            "before_context": list[str] | None,
            "after_context": list[str] | None,
        }
    ],
    "total_matches": int,
}
```

**출력 (files_with_matches 모드):**

```python theme={null}
{
    "files": list[str],  # Files containing matches
    "count": int,  # Number of files with matches
}
```

### NotebookEdit

**도구 이름:** `NotebookEdit`

**입력:**

```python theme={null}
{
    "notebook_path": str,  # Absolute path to the Jupyter notebook
    "cell_id": str | None,  # The ID of the cell to edit
    "new_source": str,  # The new source for the cell
    "cell_type": "code" | "markdown" | None,  # The type of the cell
    "edit_mode": "replace" | "insert" | "delete" | None,  # Edit operation type
}
```

**출력:**

```python theme={null}
{
    "message": str,  # Success message
    "edit_type": "replaced" | "inserted" | "deleted",  # Type of edit performed
    "cell_id": str | None,  # Cell ID that was affected
    "total_cells": int,  # Total cells in notebook after edit
}
```

### WebFetch

**도구 이름:** `WebFetch`

**입력:**

```python theme={null}
{
    "url": str,  # The URL to fetch content from
    "prompt": str,  # The prompt to run on the fetched content
}
```

**출력:**

```python theme={null}
{
    "bytes": int,  # Size of the fetched content in bytes
    "code": int,  # HTTP response code
    "codeText": str,  # HTTP response code text
    "result": str,  # Processed result from applying the prompt to the content
    "durationMs": int,  # Time to fetch and process the content, in milliseconds
    "url": str,  # URL that was fetched
}
```

### WebSearch

**도구 이름:** `WebSearch`

**입력:**

```python theme={null}
{
    "query": str,  # The search query to use
    "allowed_domains": list[str] | None,  # Only include results from these domains
    "blocked_domains": list[str] | None,  # Never include results from these domains
}
```

**출력:**

```python theme={null}
{
    "query": str,  # The search query
    "results": list[str | {"tool_use_id": str, "content": list[{"title": str, "url": str}]}],
    "durationSeconds": float,  # Search duration in seconds
}
```

### TodoWrite

**도구 이름:** `TodoWrite`

<Note>
  Claude Code v2.1.142부터 `TodoWrite`는 기본적으로 비활성화되어 있습니다. 대신 `TaskCreate`, `TaskGet`, `TaskUpdate` 및 `TaskList`를 사용하세요. 모니터링 코드를 업데이트하려면 [Task 도구로 마이그레이션](/docs/en/agent-sdk/todo-tracking#migrate-to-task-tools)을 참고하거나, `CLAUDE_CODE_ENABLE_TASKS=0`을 설정하여 `TodoWrite`로 돌아가세요.
</Note>

**입력:**

```python theme={null}
{
    "todos": [
        {
            "content": str,  # The task description
            "status": "pending" | "in_progress" | "completed",  # Task status
            "activeForm": str,  # Active form of the description
        }
    ]
}
```

**출력:**

```python theme={null}
{
    "message": str,  # Success message
    "stats": {"total": int, "pending": int, "in_progress": int, "completed": int},
}
```

### TaskCreate

**도구 이름:** `TaskCreate`

**입력:**

```python theme={null}
{
    "subject": str,  # Short task title
    "description": str,  # Detailed task body
    "activeForm": str | None,  # Present-tense label shown while in progress
    "metadata": dict | None,  # Arbitrary caller metadata
}
```

**출력:**

```python theme={null}
{
    "task": {"id": str, "subject": str},  # Created task with assigned ID
}
```

### TaskUpdate

**도구 이름:** `TaskUpdate`

**입력:**

```python theme={null}
{
    "taskId": str,  # ID of the task to patch
    "status": Literal["pending", "in_progress", "completed", "deleted"] | None,
    "subject": str | None,
    "description": str | None,
    "activeForm": str | None,
    "addBlocks": list[str] | None,  # Task IDs this task now blocks
    "addBlockedBy": list[str] | None,  # Task IDs that now block this task
    "owner": str | None,
    "metadata": dict | None,
}
```

**출력:**

```python theme={null}
{
    "success": bool,
    "taskId": str,
    "updatedFields": list[str],  # Names of fields that changed
    "error": str | None,
    "statusChange": {"from": str, "to": str} | None,
}
```

### TaskGet

**도구 이름:** `TaskGet`

**입력:**

```python theme={null}
{
    "taskId": str,  # ID of the task to read
}
```

**출력:**

```python theme={null}
{
    "task": {
        "id": str,
        "subject": str,
        "description": str,
        "status": Literal["pending", "in_progress", "completed"],
        "blocks": list[str],
        "blockedBy": list[str],
    } | None,  # None when the ID is not found
}
```

### TaskList

**도구 이름:** `TaskList`

**입력:**

```python theme={null}
{}
```

**출력:**

```python theme={null}
{
    "tasks": [
        {
            "id": str,
            "subject": str,
            "status": Literal["pending", "in_progress", "completed"],
            "owner": str | None,
            "blockedBy": list[str],
        }
    ],
}
```

### TaskOutput

**도구 이름:** `TaskOutput`. 이전 이름인 `BashOutput`도 별칭으로 허용됩니다.

<Note>`TaskOutput`은 더 이상 권장되지 않으며, 작업의 출력 파일 경로에 대해 `Read`를 사용하는 것을 권장합니다. {/* min-version: 2.1.83 */}Claude Code v2.1.83부터 지원 중단되었습니다. 아래 스키마는 이 도구를 다루는 후크 및 권한 핸들러에서 유효합니다.</Note>

**입력:**

```python theme={null}
{
    "task_id": str,  # The task ID to get output from
    "block": bool,  # Whether to wait for completion (default True)
    "timeout": int,  # Max wait time in ms (default 30000)
}
```

**출력:**

```python theme={null}
{
    "retrieval_status": "success" | "timeout" | "not_ready",  # Whether the output was retrieved
    "task": dict | None,  # Task details: task_id, task_type, status, description, output, plus type-specific fields such as exitCode
}
```

### TaskStop

**도구 이름:** `TaskStop`. 이전 이름인 `KillShell` 및 `KillBash`도 별칭으로 허용됩니다.

**입력:**

```python theme={null}
{
    "task_id": str | None,  # The ID of the background task to stop
    "shell_id": str | None,  # Deprecated: use task_id instead
}
```

**출력:**

```python theme={null}
{
    "message": str,  # Status message about the operation
    "task_id": str,  # The ID of the task that was stopped
    "task_type": str,  # The type of the task that was stopped
    "command": str | None,  # The command or description of the stopped task
}
```

### ExitPlanMode

**도구 이름:** `ExitPlanMode`

**입력:**

```python theme={null}
{
    "plan": str  # The plan to run by the user for approval
}
```

**출력:**

```python theme={null}
{
    "message": str,  # Confirmation message
    "approved": bool | None,  # Whether user approved the plan
}
```

### ListMcpResources

**도구 이름:** `ListMcpResourcesTool`

**입력:**

```python theme={null}
{
    "server": str | None  # Optional server name to filter resources by
}
```

**출력:**

```python theme={null}
{
    "resources": [
        {
            "uri": str,
            "name": str,
            "description": str | None,
            "mimeType": str | None,
            "server": str,
        }
    ],
    "total": int,
}
```

### ReadMcpResource

**도구 이름:** `ReadMcpResourceTool`

**입력:**

```python theme={null}
{
    "server": str,  # The MCP server name
    "uri": str,  # The resource URI to read
}
```

**출력:**

```python theme={null}
{
    "contents": [
        {"uri": str, "mimeType": str | None, "text": str | None, "blob": str | None}
    ],
    "server": str,
}
```

## ClaudeSDKClient를 활용한 고급 기능

### 연속 대화 인터페이스 구축

```python theme={null}
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    TextBlock,
)
import asyncio


class ConversationSession:
    """Maintains a single conversation session with Claude."""

    def __init__(self, options: ClaudeAgentOptions | None = None):
        self.client = ClaudeSDKClient(options)
        self.turn_count = 0

    async def start(self):
        await self.client.connect()
        print("Starting conversation session. Claude will remember context.")
        print(
            "Commands: 'exit' to quit, 'interrupt' to stop current task, 'new' for new session"
        )

        while True:
            user_input = input(f"\n[Turn {self.turn_count + 1}] You: ")

            if user_input.lower() == "exit":
                break
            elif user_input.lower() == "interrupt":
                await self.client.interrupt()
                print("Task interrupted!")
                continue
            elif user_input.lower() == "new":
                # Disconnect and reconnect for a fresh session
                await self.client.disconnect()
                await self.client.connect()
                self.turn_count = 0
                print("Started new conversation session (previous context cleared)")
                continue

            # Send message - the session retains all previous messages
            await self.client.query(user_input)
            self.turn_count += 1

            # Process response
            print(f"[Turn {self.turn_count}] Claude: ", end="")
            async for message in self.client.receive_response():
                if isinstance(message, AssistantMessage):
                    for block in message.content:
                        if isinstance(block, TextBlock):
                            print(block.text, end="")
            print()  # New line after response

        await self.client.disconnect()
        print(f"Conversation ended after {self.turn_count} turns.")


async def main():
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Write", "Bash"], permission_mode="acceptEdits"
    )
    session = ConversationSession(options)
    await session.start()


# Example conversation:
# Turn 1 - You: "Create a file called hello.py"
# Turn 1 - Claude: "I'll create a hello.py file for you..."
# Turn 2 - You: "What's in that file?"
# Turn 2 - Claude: "The hello.py file I just created contains..." (remembers!)
# Turn 3 - You: "Add a main function to it"
# Turn 3 - Claude: "I'll add a main function to hello.py..." (knows which file!)

asyncio.run(main())
```

### 동작 수정을 위한 후크 사용

```python theme={null}
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    HookMatcher,
    HookContext,
)
import asyncio
from typing import Any


async def pre_tool_logger(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Log all tool usage before execution."""
    tool_name = input_data.get("tool_name", "unknown")
    print(f"[PRE-TOOL] About to use: {tool_name}")

    # You can modify or block the tool execution here
    if tool_name == "Bash" and "rm -rf" in str(input_data.get("tool_input", {})):
        return {
            "hookSpecificOutput": {
                "hookEventName": "PreToolUse",
                "permissionDecision": "deny",
                "permissionDecisionReason": "Dangerous command blocked",
            }
        }
    return {}


async def post_tool_logger(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Log results after tool execution."""
    tool_name = input_data.get("tool_name", "unknown")
    print(f"[POST-TOOL] Completed: {tool_name}")
    return {}


async def user_prompt_modifier(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Add context to user prompts."""
    original_prompt = input_data.get("prompt", "")

    # Add a timestamp as additional context for Claude to see
    from datetime import datetime

    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    return {
        "hookSpecificOutput": {
            "hookEventName": "UserPromptSubmit",
            "additionalContext": f"[Submitted at {timestamp}] Original prompt: {original_prompt}",
        }
    }


async def main():
    options = ClaudeAgentOptions(
        hooks={
            "PreToolUse": [
                HookMatcher(hooks=[pre_tool_logger]),
                HookMatcher(matcher="Bash", hooks=[pre_tool_logger]),
            ],
            "PostToolUse": [HookMatcher(hooks=[post_tool_logger])],
            "UserPromptSubmit": [HookMatcher(hooks=[user_prompt_modifier])],
        },
        allowed_tools=["Read", "Write", "Bash"],
    )

    async with ClaudeSDKClient(options=options) as client:
        await client.query("List files in current directory")

        async for message in client.receive_response():
            # Hooks will automatically log tool usage
            pass


asyncio.run(main())
```

### 실시간 진행률 모니터링

```python theme={null}
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    ToolUseBlock,
    ToolResultBlock,
    TextBlock,
)
import asyncio


async def monitor_progress():
    options = ClaudeAgentOptions(
        allowed_tools=["Write", "Bash"], permission_mode="acceptEdits"
    )

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Create 5 Python files with different sorting algorithms")

        # Monitor progress in real-time
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, ToolUseBlock):
                        if block.name == "Write":
                            file_path = block.input.get("file_path", "")
                            print(f"Creating: {file_path}")
                    elif isinstance(block, ToolResultBlock):
                        print("Completed tool execution")
                    elif isinstance(block, TextBlock):
                        print(f"Claude says: {block.text[:100]}...")

        print("Task completed!")


asyncio.run(monitor_progress())
```

## 사용 예제

### 기본 파일 작업 (`query` 사용)

```python theme={null}
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ToolUseBlock
import asyncio


async def create_project():
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Write", "Bash"],
        permission_mode="acceptEdits",
    )

    async for message in query(
        prompt="Create a Python project structure with setup.py", options=options
    ):
        if isinstance(message, AssistantMessage):
            for block in message.content:
                if isinstance(block, ToolUseBlock):
                    print(f"Using tool: {block.name}")


asyncio.run(create_project())
```

### 오류 처리

```python theme={null}
import asyncio

from claude_agent_sdk import query, CLINotFoundError, ProcessError, CLIJSONDecodeError


async def main():
    try:
        async for message in query(prompt="Hello"):
            print(message)
    except CLINotFoundError:
        print(
            "Claude Code CLI not found. Try reinstalling: pip install --force-reinstall claude-agent-sdk"
        )
    except ProcessError as e:
        print(f"Process failed with exit code: {e.exit_code}")
    except CLIJSONDecodeError as e:
        print(f"Failed to parse response: {e}")


asyncio.run(main())
```

### 클라이언트를 사용한 스트리밍 모드

```python theme={null}
from claude_agent_sdk import ClaudeSDKClient
import asyncio


async def interactive_session():
    async with ClaudeSDKClient() as client:
        # Send initial message
        await client.query("What's the weather like?")

        # Process responses
        async for msg in client.receive_response():
            print(msg)

        # Send follow-up
        await client.query("Tell me more about that")

        # Process follow-up response
        async for msg in client.receive_response():
            print(msg)


asyncio.run(interactive_session())
```

### ClaudeSDKClient에서 커스텀 도구 사용하기

```python theme={null}
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    tool,
    create_sdk_mcp_server,
    AssistantMessage,
    TextBlock,
)
import asyncio
from typing import Any


# Define custom tools with @tool decorator
@tool("calculate", "Perform mathematical calculations", {"expression": str})
async def calculate(args: dict[str, Any]) -> dict[str, Any]:
    try:
        result = eval(args["expression"], {"__builtins__": {}})
        return {"content": [{"type": "text", "text": f"Result: {result}"}]}
    except Exception as e:
        return {
            "content": [{"type": "text", "text": f"Error: {str(e)}"}],
            "is_error": True,
        }


@tool("get_time", "Get current time", {})
async def get_time(args: dict[str, Any]) -> dict[str, Any]:
    from datetime import datetime

    current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    return {"content": [{"type": "text", "text": f"Current time: {current_time}"}]}


async def main():
    # Create SDK MCP server with custom tools
    my_server = create_sdk_mcp_server(
        name="utilities", version="1.0.0", tools=[calculate, get_time]
    )

    # Configure options with the server
    options = ClaudeAgentOptions(
        mcp_servers={"utils": my_server},
        allowed_tools=["mcp__utils__calculate", "mcp__utils__get_time"],
    )

    # Use ClaudeSDKClient for interactive tool usage
    async with ClaudeSDKClient(options=options) as client:
        await client.query("What's 123 * 456?")

        # Process calculation response
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Calculation: {block.text}")

        # Follow up with time query
        await client.query("What time is it now?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Time: {block.text}")


asyncio.run(main())
```

## 샌드박스 구성 (Sandbox Configuration)

### `SandboxSettings`

샌드박스 동작을 위한 구성입니다. 프로그래밍 방식으로 명령 샌드박스를 활성화하고 네트워크 제약을 구성할 때 사용하세요.

```python theme={null}
class SandboxSettings(TypedDict, total=False):
    enabled: bool
    autoAllowBashIfSandboxed: bool
    excludedCommands: list[str]
    allowUnsandboxedCommands: bool
    network: SandboxNetworkConfig
    ignoreViolations: SandboxIgnoreViolations
    enableWeakerNestedSandbox: bool
```

| 속성 | 타입 | 기본값 | 설명 |
| :-------------------------- | :---------------------------------------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled` | `bool` | `False` | 명령 실행에 대한 샌드박스 모드 활성화 |
| `autoAllowBashIfSandboxed` | `bool` | `True` | 샌드박스가 활성화되었을 때 bash 명령을 자동 승인 |
| `excludedCommands` | `list[str]` | `[]` | 샌드박스 제약을 항상 우회하는 명령어 목록 (예: `["docker"]`). 모델 개입 없이 자동으로 샌드박스 미적용 실행 |
| `allowUnsandboxedCommands` | `bool` | `True` | 모델이 샌드박스 외부에서 명령을 실행하도록 요청하는 것 허용. `True`일 때 모델은 도구 입력에 `dangerouslyDisableSandbox`를 설정할 수 있으며, 이는 [권한 시스템으로 폴백](#permissions-fallback-for-unsandboxed-commands) |
| `network` | [`SandboxNetworkConfig`](#sandboxnetworkconfig) | `None` | 네트워크 전용 샌드박스 구성 |
| `ignoreViolations` | [`SandboxIgnoreViolations`](#sandboxignoreviolations) | `None` | 무시할 샌드박스 위반 사항 구성 |
| `enableWeakerNestedSandbox` | `bool` | `False` | 호환성을 위해 더 약한 중첩 샌드박스 활성화 |

<Note>
  샌드박스는 플랫폼 지원 여부 및 Linux의 경우 `bubblewrap` 및 `socat`과 같은 도구에 의존합니다. 기본적으로 `enabled`가 `True`이지만 샌드박스가 시작될 수 없는 경우, stderr에 경고를 남기고 명령이 샌드박스 없이 실행됩니다. 이 기본값은 `failIfUnavailable` 기본값이 `true`인 TypeScript SDK와 다릅니다.

  중단하려면 샌드박스 설정에 `"failIfUnavailable": True`를 설정하세요. 해당 키는 아직 `SandboxSettings`에 선언되어 있지 않으나 SDK는 Claude Code에 전달하여 이를 준수하도록 합니다. 그러면 `query()`는 `subtype="error_during_execution"` 및 `errors`에 원인이 포함된 `ResultMessage`를 보고합니다. 일회성 `query()` 호출이므로 SDK는 해당 오류 결과를 반환한 후 예외를 발생시키므로 계속 진행하려면 try 블록으로 루프를 감싸세요. [결과 처리하기](/docs/en/agent-sdk/agent-loop#handle-the-result)의 오류 계약을 참고하세요.
</Note>

#### 사용 예제

```python theme={null}
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions, SandboxSettings

sandbox_settings: SandboxSettings = {
    "enabled": True,
    "autoAllowBashIfSandboxed": True,
    "network": {"allowLocalBinding": True},
}


async def main():
    try:
        async for message in query(
            prompt="Build and test my project",
            options=ClaudeAgentOptions(sandbox=sandbox_settings),
        ):
            print(message)
    except Exception as error:
        # A single-shot query() raises after yielding an error result,
        # such as when failIfUnavailable is set and the sandbox can't start.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

<Warning>
  **Unix 소켓 보안**: `allowUnixSockets` 옵션은 강력한 시스템 서비스에 대한 접근을 허용할 수 있습니다. 예를 들어 `/var/run/docker.sock`을 허용하는 것은 Docker API를 통해 호스트 시스템에 대한 완전한 접근 권한을 허용하여 샌드박스 격리를 우회하는 결과를 낳습니다. 엄격하게 필요한 Unix 소켓만 허용하고 각 소켓의 보안적 함의를 파악하세요.
</Warning>

### `SandboxNetworkConfig`

샌드박스 모드를 위한 네트워크 전용 구성입니다. 이 설정은 부모 [`SandboxSettings`](#sandboxsettings)에서 `enabled`가 `True`일 때 샌드박스가 적용된 Bash 명령에 적용됩니다. 대신 [권한 규칙](/docs/en/permissions#webfetch)을 사용하는 WebFetch 도구를 제한하지는 않습니다.

```python theme={null}
class SandboxNetworkConfig(TypedDict, total=False):
    allowedDomains: list[str]
    deniedDomains: list[str]
    allowManagedDomainsOnly: bool
    allowUnixSockets: list[str]
    allowAllUnixSockets: bool
    allowLocalBinding: bool
    allowMachLookup: list[str]
    httpProxyPort: int
    socksProxyPort: int
```

| 속성 | 타입 | 기본값 | 설명 |
| :------------------------ | :---------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `allowedDomains` | `list[str]` | `[]` | 샌드박스 처리된 프로세스가 접근할 수 있는 도메인 이름 목록 |
| `deniedDomains` | `list[str]` | `[]` | 샌드박스 처리된 프로세스가 접근할 수 없는 도메인 이름 목록. `allowedDomains`보다 우선함 |
| `allowManagedDomainsOnly` | `bool` | `False` | 관리형 설정 전용: 관리형 설정에 지정된 경우 비관리형 설정 소스의 `allowedDomains`를 무시함. SDK 옵션을 통해 설정 시 영향 없음 |
| `allowUnixSockets` | `list[str]` | `[]` | 프로세스가 접근할 수 있는 Unix 소켓 경로 (예: Docker 소켓) |
| `allowAllUnixSockets` | `bool` | `False` | 모든 Unix 소켓에 대한 접근 허용 |
| `allowLocalBinding` | `bool` | `False` | 프로세스가 로컬 포트에 바인딩하는 것을 허용 (예: 개발 서버용) |
| `allowMachLookup` | `list[str]` | `[]` | macOS 전용: 허용할 XPC/Mach 서비스 이름 목록. 후미 와일드카드 지원 |
| `httpProxyPort` | `int` | `None` | 네트워크 요청용 HTTP 프록시 포트 |
| `socksProxyPort` | `int` | `None` | 네트워크 요청용 SOCKS 프록시 포트 |

<Note>
  내장 샌드박스 프록시는 요청된 호스트이름에 기반하여 네트워크 허용 목록을 적용하며 TLS 트래픽을 종료하거나 검사하지 않으므로, [domain fronting](https://en.wikipedia.org/wiki/Domain_fronting)과 같은 기술이 우회할 수 있습니다. 자세한 내용은 [샌드박스 보안 제한사항](/docs/en/sandboxing#security-limitations)을, TLS 종료 프록시 구성은 [안전한 배포](/docs/en/agent-sdk/secure-deployment#traffic-forwarding)를 참고하세요.
</Note>

### `SandboxIgnoreViolations`

특정 샌드박스 위반 사항을 무시하기 위한 구성입니다.

```python theme={null}
class SandboxIgnoreViolations(TypedDict, total=False):
    file: list[str]
    network: list[str]
```

| 속성 | 타입 | 기본값 | 설명 |
| :-------- | :---------- | :------ | :------------------------------------------ |
| `file` | `list[str]` | `[]` | 위반 사항을 무시할 파일 경로 패턴 |
| `network` | `list[str]` | `[]` | 위반 사항을 무시할 네트워크 패턴 |

### 샌드박스 미적용 명령에 대한 권한 폴백

`allowUnsandboxedCommands`가 활성화되면 모델은 도구 입력에 `dangerouslyDisableSandbox: True`를 설정하여 샌드박스 외부에서 명령을 실행하도록 요청할 수 있습니다. 이 요청은 기존 권한 시스템으로 폴백되며, 이는 `can_use_tool` 핸들러가 호출되어 커스텀 승인 로직을 구현할 수 있음을 의미합니다.

<Note>
  **`excludedCommands` vs `allowUnsandboxedCommands`:**

  * `excludedCommands`: 샌드박스를 항상 자동으로 우회하는 명령어의 정적 목록입니다(예: `["docker"]`). 모델은 이에 대해 어떠한 제어 권한도 갖지 못합니다.
  * `allowUnsandboxedCommands`: 모델이 도구 입력에 `dangerouslyDisableSandbox: True`를 설정하여 샌드박스 미적용 실행을 요청할지 여부를 런타임에 직접 결정할 수 있게 합니다.
</Note>

```python theme={null}
import asyncio
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    HookMatcher,
    PermissionResultAllow,
    PermissionResultDeny,
    ToolPermissionContext,
)


def is_command_authorized(command: str | None) -> bool:
    # Replace with your own authorization logic
    return False



async def can_use_tool(
    tool: str, input: dict, context: ToolPermissionContext
) -> PermissionResultAllow | PermissionResultDeny:
    # Check if the model is requesting to bypass the sandbox
    if tool == "Bash" and input.get("dangerouslyDisableSandbox"):
        # The model is requesting to run this command outside the sandbox
        print(f"Unsandboxed command requested: {input.get('command')}")

        if is_command_authorized(input.get("command")):
            return PermissionResultAllow()
        return PermissionResultDeny(
            message="Command not authorized for unsandboxed execution"
        )
    return PermissionResultAllow()


# Required: dummy hook keeps the stream open for can_use_tool
async def dummy_hook(input_data, tool_use_id, context):
    return {"continue_": True}


async def prompt_stream():
    yield {
        "type": "user",
        "message": {"role": "user", "content": "Deploy my application"},
    }


async def main():
    async for message in query(
        prompt=prompt_stream(),
        options=ClaudeAgentOptions(
            sandbox={
                "enabled": True,
                "allowUnsandboxedCommands": True,  # Model can request unsandboxed execution
            },
            permission_mode="default",
            can_use_tool=can_use_tool,
            hooks={"PreToolUse": [HookMatcher(matcher=None, hooks=[dummy_hook])]},
        ),
    ):
        print(message)


asyncio.run(main())
```

이 패턴을 통해 다음이 가능합니다:

* **모델 요청 감사**: 모델이 샌드박스 미적용 실행을 요청하는 시점을 기록
* **허용 목록 구현**: 특정 명령어만 샌드박스 미적용으로 실행되도록 허용
* **승인 워크플로 추가**: 권한이 필요한 작업에 대해 명시적 승인 요구

<Warning>
  `dangerouslyDisableSandbox: True`로 실행되는 명령은 완전한 시스템 접근 권한을 갖습니다. `can_use_tool` 핸들러가 이러한 요청을 신중하게 검증하도록 하세요.

  `permission_mode`가 `bypassPermissions`로 설정되어 있고 `allow_unsandboxed_commands`가 활성화되어 있는 경우, 모델은 승인 프롬프트 없이 샌드박스 외부에서 명령을 자율적으로 실행할 수 있습니다 (명시적인 [`ask` 규칙](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)이 있는 경우는 예외). 이 조합은 결과적으로 모델이 소리 없이 샌드박스 격리를 벗어날 수 있게 만듭니다.
</Warning>

## 관련 항목

* [SDK 개요](/docs/en/agent-sdk/overview) - 일반 SDK 개념
* [TypeScript SDK 참조 문서](/docs/en/agent-sdk/typescript) - TypeScript SDK 문서
* [CLI 참조 문서](/docs/en/cli-reference) - 명령줄 인터페이스
* [일반적인 워크플로](/docs/en/common-workflows) - 단계별 가이드
