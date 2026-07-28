> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Agent SDK 레퍼런스 - TypeScript

> 모든 함수, 타입 및 인터페이스를 포함한 TypeScript Agent SDK의 전체 API 레퍼런스입니다.

<script src="/docs/components/typescript-sdk-type-links.js" defer />

## 설치 (Installation)

```bash theme={null}
npm install @anthropic-ai/claude-agent-sdk
```

<Note>
  SDK는 `@anthropic-ai/claude-agent-sdk-darwin-arm64`와 같은 플랫폼용 네이티브 Claude Code 바이너리를 선택적 종속성(optional dependency)으로 번들로 제공합니다. Claude Code를 별도로 설치할 필요가 없습니다. SDK 버전은 번들로 제공되는 Claude Code 버전을 따릅니다. SDK v0.3.191은 Claude Code v2.1.191을 번들로 포함하므로, 이 페이지에서 Claude Code 버전을 요구하는 기능은 해당 패치 번호 이상의 SDK 릴리스가 필요합니다. 패키지 관리자가 선택적 종속성을 건너뛰는 경우 SDK는 `Native CLI binary for <platform> not found` 에러를 발생시킵니다. 이 경우 [`pathToClaudeCodeExecutable`](#options)을 별도로 설치된 `claude` 바이너리로 설정하세요.
</Note>

### 단일 실행 파일로 컴파일하기

`bun build --compile`을 사용하여 애플리케이션을 단일 파일 실행 파일로 컴파일하는 경우, SDK는 런타임에 번들로 제공되는 CLI 바이너리를 확인할 수 없습니다. 컴파일된 실행 파일의 `$bunfs` 가상 파일 시스템 내부에서는 `require.resolve`가 작동하지 않으므로 SDK는 `Native CLI binary for <platform> not found`를 발생시킵니다.

이를 우회하려면 플랫폼 바이너리를 파일 자산으로 임베드하고, 시작 시 `extractFromBunfs()`를 사용하여 실제 경로로 추출한 다음 해당 경로를 [`pathToClaudeCodeExecutable`](#options)에 전달하세요.

`extractFromBunfs()` 헬퍼에는 `@anthropic-ai/claude-agent-sdk` v0.3.144 이상이 필요합니다. 아래 예시는 Apple Silicon macOS용 빌드 예시입니다:

```typescript theme={null}
import binPath from "@anthropic-ai/claude-agent-sdk-darwin-arm64/claude" with { type: "file" };
import { extractFromBunfs } from "@anthropic-ai/claude-agent-sdk/extract";
import { query } from "@anthropic-ai/claude-agent-sdk";

const cliPath = extractFromBunfs(binPath);

for await (const message of query({
  prompt: "Hello",
  options: { pathToClaudeCodeExecutable: cliPath },
})) {
  console.log(message);
}
```

`extractFromBunfs()`는 컴파일된 실행 파일의 가상 파일 시스템에서 임베디드 바이너리를 사용자별 임시 디렉터리로 복사하고 실제 경로를 반환합니다. 컴파일된 실행 파일 외부에서는 입력 경로를 변경 없이 반환하므로 동일한 코드가 수정 없이 개발 환경에서 실행됩니다.

각 컴파일된 실행 파일은 단일 플랫폼의 바이너리를 임베드합니다. import의 플랫폼 패키지를 `--target`에 맞추세요:

* 크로스 컴파일하려면 일치하지 않는 플랫폼 패키지를 설치하세요(예: `npm install @anthropic-ai/claude-agent-sdk-linux-x64 --force`).
* Windows에서 바이너리 하위 경로는 `claude.exe`입니다(예: `@anthropic-ai/claude-agent-sdk-win32-x64/claude.exe`).

## 함수 (Functions)

### `query()`

Claude Code와 상호작용하기 위한 주요 함수입니다. 메시지가 도착하는 대로 스트리밍하는 비동기 제네레이터를 생성합니다.

```typescript theme={null}
function query({
  prompt,
  options
}: {
  prompt: string | AsyncIterable<SDKUserMessage>;
  options?: Options;
}): Query;
```

#### 매개변수 (Parameters)

| 매개변수  | 타입                                                             | 설명                                                                   |
| :-------- | :--------------------------------------------------------------- | :--------------------------------------------------------------------- |
| `prompt`  | `string \| AsyncIterable<`[`SDKUserMessage`](#sdkusermessage)`>` | 문자열 또는 스트리밍 모드용 비동기 반복 가능 객체(async iterable) 프롬프트 |
| `options` | [`Options`](#options)                                            | 선택적 구성 객체 (아래 Options 타입 참조)                               |

#### 반환 값 (Returns)

추가 메서드가 있는 `AsyncGenerator<`[`SDKMessage`](#sdkmessage)`, void>`를 확장하는 [`Query`](#query-object) 객체를 반환합니다.

### `startup()`

프롬프트를 사용할 수 있게 되기 전에 하위 프로세스를 생성하고 초기화 핸드셰이크를 완료하여 CLI 하위 프로세스를 사전 워밍(pre-warm)합니다. 반환된 [`WarmQuery`](#warmquery) 핸들은 나중에 프롬프트를 수락하고 이미 준비된 프로세스에 작성하므로, 첫 번째 `query()` 호출이 인라인 하위 프로세스 생성 및 초기화 비용을 지불하지 않고 즉시 확인됩니다.

```typescript theme={null}
function startup(params?: {
  options?: Options;
  initializeTimeoutMs?: number;
}): Promise<WarmQuery>;
```

#### 매개변수

| 매개변수              | 타입                  | 설명                                                                                                                                                                          |
| :-------------------- | :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `options`             | [`Options`](#options) | 선택적 구성 객체. `query()`의 `options` 매개변수와 동일                                                                                                                     |
| `initializeTimeoutMs` | `number`              | 하위 프로세스 초기화를 기다리는 최대 시간(밀리초). 기본값은 `60000`. 초기화가 제시간에 완료되지 않으면 프로미스는 타임아웃 에러와 함께 거부됨                                 |

#### 반환 값

하위 프로세스가 생성되고 초기화 핸드셰이크가 완료되면 확인되는 `Promise<`[`WarmQuery`](#warmquery)`>`를 반환합니다.

#### 예시

애플리케이션 부팅 시 등 조기에 `startup()`을 호출한 다음, 프롬프트가 준비되면 반환된 핸들에서 `.query()`를 호출하세요. 이렇게 하면 하위 프로세스 생성 및 초기화가 임계 경로에서 제외됩니다.

```typescript theme={null}
import { startup } from "@anthropic-ai/claude-agent-sdk";

// 미리 시작 비용 지불
const warm = await startup({ options: { maxTurns: 3 } });

// 나중에 프롬프트가 준비되면 즉시 실행됨
for await (const message of warm.query("What files are here?")) {
  console.log(message);
}
```

### `tool()`

SDK MCP 서버와 함께 사용할 타입 안전 MCP 도구 정의를 생성합니다.

```typescript theme={null}
function tool<Schema extends AnyZodRawShape>(
  name: string,
  description: string,
  inputSchema: Schema,
  handler: (args: InferShape<Schema>, extra: unknown) => Promise<CallToolResult>,
  extras?: { annotations?: ToolAnnotations; searchHint?: string; alwaysLoad?: boolean }
): SdkMcpToolDefinition<Schema>;
```

#### 매개변수

| 매개변수      | 타입                                                                                                   | 설명                                                                                                                                                                                                                                                                                                         |
| :------------ | :----------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        | `string`                                                                                               | 도구 이름                                                                                                                                                                                                                                                                                                    |
| `description` | `string`                                                                                               | 도구가 수행하는 작업에 대한 설명                                                                                                                                                                                                                                                                             |
| `inputSchema` | `Schema extends AnyZodRawShape`                                                                        | 도구의 입력 매개변수를 정의하는 Zod 스키마 (Zod 3 및 Zod 4 모두 지원)                                                                                                                                                                                                                                        |
| `handler`     | `(args, extra) => Promise<`[`CallToolResult`](#calltoolresult)`>`                                      | 도구 로직을 실행하는 비동기 함수                                                                                                                                                                                                                                                                             |
| `extras`      | `{ annotations?: `[`ToolAnnotations`](#toolannotations)`; searchHint?: string; alwaysLoad?: boolean }` | 선택적 엑스트라. `annotations`는 클라이언트에 대한 MCP 동작 힌트를 제공합니다. `searchHint`는 [도구 검색](/docs/en/agent-sdk/tool-search)이 활성화되어 있을 때 지연된 도구 목록에 표시되는 한 줄 기능 구문입니다. `alwaysLoad: true`는 도구를 지연시키는 대신 초기 프롬프트에 도구의 전체 스키마를 유지합니다 |

#### `ToolAnnotations`

`@modelcontextprotocol/sdk/types.js`에서 다시 내보냅니다. 모든 필드는 선택적 힌트이며, 클라이언트는 보안 결정을 위해 이에 의존해서는 안 됩니다.

| 필드               | 타입      | 기본값      | 설명                                                                                                                  |
| :----------------- | :-------- | :---------- | :-------------------------------------------------------------------------------------------------------------------- |
| `title`            | `string`  | `undefined` | 사람이 읽을 수 있는 도구 제목                                                                                          |
| `readOnlyHint`     | `boolean` | `false`     | `true`인 경우 도구가 환경을 수정하지 않음                                                                              |
| `destructiveHint`  | `boolean` | `true`      | `true`인 경우 도구가 파괴적인 업데이트를 수행할 수 있음 (`readOnlyHint`가 `false`일 때만 의미가 있음)                  |
| `idempotentHint`   | `boolean` | `false`     | `true`인 경우 동일한 인자로 반복 호출해도 추가 효과가 없음 (`readOnlyHint`가 `false`일 때만 의미가 있음)                |
| `openWorldHint`    | `boolean` | `true`      | `true`인 경우 도구가 외부 엔티티(예: 웹 검색)와 상호작용함. `false`인 경우 도구 도메인이 닫혀 있음(예: 메모리 도구)     |

```typescript theme={null}
import { tool } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const searchTool = tool(
  "search",
  "Search the web",
  { query: z.string() },
  async ({ query }) => {
    return { content: [{ type: "text", text: `Results for: ${query}` }] };
  },
  { annotations: { readOnlyHint: true, openWorldHint: true } }
);
```

### `createSdkMcpServer()`

애플리케이션과 동일한 프로세스에서 실행되는 MCP 서버 인스턴스를 생성합니다.

```typescript theme={null}
function createSdkMcpServer(options: {
  name: string;
  version?: string;
  instructions?: string;
  tools?: Array<SdkMcpToolDefinition<any>>;
  alwaysLoad?: boolean;
}): McpSdkServerConfigWithInstance;
```

#### 매개변수

| 매개변수               | 타입                          | 설명                                                                                                                                                                                                                                        |
| :--------------------- | :---------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `options.name`         | `string`                      | MCP 서버 이름                                                                                                                                                                                                                               |
| `options.version`      | `string`                      | 선택적 버전 문자열                                                                                                                                                                                                                          |
| `options.instructions` | `string`                      | `initialize`에서 반환되고 MCP 지침 블록으로 모델에 노출되는 선택적 서버 지침                                                                                                                                                                |
| `options.tools`        | `Array<SdkMcpToolDefinition>` | [`tool()`](#tool)로 생성된 도구 정의 배열                                                                                                                                                                                                   |
| `options.alwaysLoad`   | `boolean`                     | `true`인 경우 이 서버의 모든 도구가 초기 프롬프트에 유지되며 [도구 검색](/docs/en/agent-sdk/tool-search) 뒤로 지연되지 않습니다. [`tool()`](#tool)의 도구별 `alwaysLoad`와 조합됩니다                                                    |

### `listSessions()`

경량 메타데이터와 함께 이전 세션을 탐색하고 나열합니다. 프로젝트 디렉터리로 필터링하거나 모든 프로젝트에 걸쳐 세션을 나열합니다.

```typescript theme={null}
function listSessions(options?: ListSessionsOptions): Promise<SDKSessionInfo[]>;
```

#### 매개변수

| 매개변수                   | 타입      | 기본값      | 설명                                                                               |
| :------------------------- | :-------- | :---------- | :--------------------------------------------------------------------------------- |
| `options.dir`              | `string`  | `undefined` | 세션을 나열할 디렉터리. 생략 시 모든 프로젝트의 세션을 반환함                      |
| `options.limit`            | `number`  | `undefined` | 반환할 최대 세션 수                                                                |
| `options.includeWorktrees` | `boolean` | `true`      | `dir`이 git 리포지토리 내부일 때 모든 워크트리 경로의 세션을 포함함               |

#### 반환 타입: `SDKSessionInfo`

| 속성           | 타입                  | 설명                                                                          |
| :------------- | :-------------------- | :---------------------------------------------------------------------------- |
| `sessionId`    | `string`              | 고유 세션 식별자 (UUID)                                                       |
| `summary`      | `string`              | 표시 제목: 커스텀 제목, 자동 생성 요약 또는 첫 번째 프롬프트                 |
| `lastModified` | `number`              | 에포크 이후 밀리초 단위의 최종 수정 시간                                      |
| `fileSize`     | `number \| undefined` | 바이트 단위의 세션 파일 크기. 로컬 JSONL 저장소에 대해서만 채워짐             |
| `customTitle`  | `string \| undefined` | 사용자가 설정한 세션 제목 (`/rename` 사용)                                    |
| `firstPrompt`  | `string \| undefined` | 세션에서 의미 있는 첫 번째 사용자 프롬프트                                    |
| `gitBranch`    | `string \| undefined` | 세션 종료 시점의 Git 브랜치                                                   |
| `cwd`          | `string \| undefined` | 세션의 작업 디렉터리                                                          |
| `tag`          | `string \| undefined` | 사용자가 설정한 세션 태그 ([`tagSession()`](#tagsession) 참조)                |
| `createdAt`    | `number \| undefined` | 첫 번째 항목의 타임스탬프로부터 에포크 이후 밀리초 단위의 생성 시간           |

#### 예시

프로젝트의 가장 최근 세션 10개를 출력합니다. 결과는 `lastModified` 내림차순으로 정렬되므로 첫 번째 항목이 가장 최신입니다. 모든 프로젝트를 검색하려면 `dir`을 생략하세요.

```typescript theme={null}
import { listSessions } from "@anthropic-ai/claude-agent-sdk";

const sessions = await listSessions({ dir: "/path/to/project", limit: 10 });

for (const session of sessions) {
  console.log(`${session.summary} (${session.sessionId})`);
}
```

### `getSessionMessages()`

이전 세션 트랜스크립트에서 사용자 및 어시스턴트 메시지를 읽습니다.

```typescript theme={null}
function getSessionMessages(
  sessionId: string,
  options?: GetSessionMessagesOptions
): Promise<SessionMessage[]>;
```

#### 매개변수

| 매개변수         | 타입     | 기본값      | 설명                                                                          |
| :--------------- | :------- | :---------- | :---------------------------------------------------------------------------- |
| `sessionId`      | `string` | 필수        | 읽을 세션 UUID (`listSessions()` 참조)                                         |
| `options.dir`    | `string` | `undefined` | 세션을 찾을 프로젝트 디렉터리. 생략 시 모든 프로젝트 검색                       |
| `options.limit`  | `number` | `undefined` | 반환할 최대 메시지 수                                                         |
| `options.offset` | `number` | `undefined` | 시작 시 건너뛸 메시지 수                                                       |

#### 반환 타입: `SessionMessage`

| 속성                 | 타입                    | 설명                                                                                                                                                                                                                                        |
| :------------------- | :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `type`               | `"user" \| "assistant"` | 메시지 역할                                                                                                                                                                                                                                 |
| `uuid`               | `string`                | 고유 메시지 식별자                                                                                                                                                                                                                          |
| `session_id`         | `string`                | 이 메시지가 속한 세션                                                                                                                                                                                                                       |
| `message`            | `unknown`               | 트랜스크립트의 원시 메시지 페이로드                                                                                                                                                                                                         |
| `parent_tool_use_id` | `string \| null`        | 서브에이전트 메시지의 경우 호출한 `Agent` 도구 호출의 `tool_use_id`. 메인 세션 메시지 및 이전 세션의 경우 `null`                                                                                                                           |
| `parent_agent_id`    | `string \| null`        | [중첩된 서브에이전트](/docs/en/sub-agents#spawn-nested-subagents) 메시지의 경우 이를 생성한 서브에이전트의 `agentId`. 메인 세션 메시지, 최상위 서브에이전트 메시지 및 이전 세션의 경우 `null`. Claude Code v2.1.202 이상 필요 |

#### 예시

```typescript theme={null}
import { listSessions, getSessionMessages } from "@anthropic-ai/claude-agent-sdk";

const [latest] = await listSessions({ dir: "/path/to/project", limit: 1 });

if (latest) {
  const messages = await getSessionMessages(latest.sessionId, {
    dir: "/path/to/project",
    limit: 20
  });

  for (const msg of messages) {
    console.log(`[${msg.type}] ${msg.uuid}`);
  }
}
```

### `getSessionInfo()`

전체 프로젝트 디렉터리를 스캔하지 않고 ID로 단일 세션의 메타데이터를 읽습니다.

```typescript theme={null}
function getSessionInfo(
  sessionId: string,
  options?: GetSessionInfoOptions
): Promise<SDKSessionInfo | undefined>;
```

#### 매개변수

| 매개변수      | 타입     | 기본값      | 설명                                                    |
| :------------ | :------- | :---------- | :------------------------------------------------------ |
| `sessionId`   | `string` | 필수        | 조회할 세션의 UUID                                      |
| `options.dir` | `string` | `undefined` | 프로젝트 디렉터리 경로. 생략 시 모든 프로젝트 디렉터리 검색 |

[`SDKSessionInfo`](#return-type-sdksessioninfo)를 반환하거나 세션을 찾지 못한 경우 `undefined`를 반환합니다.

### `renameSession()`

커스텀 제목 항목을 추가하여 세션 이름을 변경합니다. 반복 호출해도 안전하며 가장 최근 제목이 적용됩니다.

```typescript theme={null}
function renameSession(
  sessionId: string,
  title: string,
  options?: SessionMutationOptions
): Promise<void>;
```

#### 매개변수

| 매개변수      | 타입     | 기본값      | 설명                                                    |
| :------------ | :------- | :---------- | :------------------------------------------------------ |
| `sessionId`   | `string` | 필수        | 이름을 변경할 세션의 UUID                                |
| `title`       | `string` | 필수        | 새 제목. 공백을 트림한 후 비어 있지 않아야 함            |
| `options.dir` | `string` | `undefined` | 프로젝트 디렉터리 경로. 생략 시 모든 프로젝트 디렉터리 검색 |

### `tagSession()`

세션에 태그를 지정합니다. 태그를 지우려면 `null`을 전달하세요. 반복 호출해도 안전하며 가장 최근 태그가 적용됩니다.

```typescript theme={null}
function tagSession(
  sessionId: string,
  tag: string | null,
  options?: SessionMutationOptions
): Promise<void>;
```

#### 매개변수

| 매개변수      | 타입             | 기본값      | 설명                                                    |
| :------------ | :--------------- | :---------- | :------------------------------------------------------ |
| `sessionId`   | `string`         | 필수        | 태그를 지정할 세션의 UUID                               |
| `tag`         | `string \| null` | 필수        | 태그 문자열 또는 지우기 위한 `null`                      |
| `options.dir` | `string`         | `undefined` | 프로젝트 디렉터리 경로. 생략 시 모든 프로젝트 디렉터리 검색 |

### `resolveSettings()`

Claude CLI를 생성하지 않고 CLI와 동일한 병합 엔진을 사용하여 특정 디렉터리의 유효한 Claude Code 설정을 확인합니다. `query()` 호출을 실행하기 전에 어떤 구성을 보게 될지 검사하는 데 사용합니다.

<Note>
  이 함수는 알파 기능이며 API는 안정화 전에 변경될 수 있습니다. CLI 시작과의 패리티를 위해 macOS plist 및 Windows HKLM/HKCU를 포함한 MDM 소스를 읽지만 관리자가 구성한 `policyHelper` 하위 프로세스는 실행하지 않습니다. `permissions.defaultMode` 필드는 프로젝트 설정을 포함한 모든 계층에서 있는 그대로 반환됩니다. 상향 권한 모드를 수용하기 전에 CLI가 적용하는 신뢰 필터는 적용되지 않습니다.
</Note>

```typescript theme={null}
function resolveSettings(
  options?: ResolveSettingsOptions
): Promise<ResolvedSettings>;
```

#### 매개변수

`resolveSettings()`는 단일 옵션 객체를 받습니다. 모든 필드는 선택 사항입니다.

| 매개변수                        | 타입                                  | 기본값          | 설명                                                                                                                                                                                                                                                                                                                                                               |
| :------------------------------ | :------------------------------------ | :-------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `options.cwd`                   | `string`                              | `process.cwd()` | 프로젝트 및 로컬 설정을 확인하는 기준 디렉터리                                                                                                                                                                                                                                                                                                                     |
| `options.settingSources`        | [`SettingSource`](#settingsource)`[]` | 모든 소스       | 로드할 파일 시스템 소스. 사용자, 프로젝트 및 로컬 설정을 건너뛰려면 `[]` 전달. [엔드포인트 관리 정책](/docs/en/settings#settings-files)은 모든 경우에 로드됨. 서버 관리 설정은 호스트가 전달할 때 `serverManagedSettings`에서 가져오며, 그렇지 않은 경우 CLI의 디스크 캐시에서 읽음 (스냅샷이 네트워크에서 이를 수집하지 않음) |
| `options.managedSettings`       | `Settings`                            | `undefined`     | 임베딩 호스트가 제공하는 정책 계층 설정. [`Options`의 `managedSettings`](#options)와 동일한 규칙을 따르지만, `resolveSettings()`는 구성된 [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)를 실행하지 않으므로 스냅샷에 실시간 세션이 삭제하는 설정이 포함될 수 있음                                                     |
| `options.serverManagedSettings` | `Settings`                            | `undefined`     | `/api/claude_code/settings`의 서버 관리 설정 페이로드. 제한적이지 않은 키는 필터링되지 않고 그대로 전달됨                                                                                                                                                                                                                                                        |

#### 반환 타입: `ResolvedSettings`

`resolveSettings()`는 병합된 설정과 각 키에 기여한 소스를 설명하는 객체를 반환합니다.

| 속성         | 타입                                                | 설명                                                                      |
| :----------- | :-------------------------------------------------- | :------------------------------------------------------------------------ |
| `effective`  | `Settings`                                          | 우선순위 순서대로 모든 활성화된 소스를 적용한 후의 병합된 설정            |
| `provenance` | `Partial<Record<keyof Settings, ProvenanceEntry>>`  | `effective`에 있는 각 최상위 키에 대해 어떤 소스가 값을 제공했는지 나타냄  |
| `sources`    | `Array<{ source, settings, path?, policyOrigin? }>` | 소스별 원시 설정으로, 가장 낮은 우선순위부터 가장 높은 우선순위순으로 정렬됨 |

#### 예시

아래 예시는 프로젝트 디렉터리의 설정을 확인하고 정리 기간을 제어하는 소스를 출력합니다.

```typescript theme={null}
import { resolveSettings } from "@anthropic-ai/claude-agent-sdk";

const { effective, provenance } = await resolveSettings({
  cwd: "/path/to/project",
  settingSources: ["user", "project", "local"],
});

console.log(`Cleanup period: ${effective.cleanupPeriodDays} days`);
console.log(`Set by: ${provenance.cleanupPeriodDays?.source}`);
```

## 타입 (Types)

### `Options`

`query()` 함수를 위한 구성 객체입니다.

| 속성                               | 타입                                                                                                     | 기본값                                      | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| :--------------------------------- | :------------------------------------------------------------------------------------------------------- | :------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `abortController`                 | `AbortController`                                                                                        | `new AbortController()`                     | 작업 취소용 컨트롤러                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `additionalDirectories`           | `string[]`                                                                                               | `[]`                                        | Claude가 접근할 수 있는 추가 디렉터리                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `agent`                           | `string`                                                                                                 | `undefined`                                 | 메인 스레드용 에이전트 이름. 에이전트는 `agents` 옵션 또는 설정에 정의되어 있어야 함                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `agents`                          | `Record<string, [`AgentDefinition`](#agentdefinition)>`                                                  | `undefined`                                 | 프로그래밍 방식으로 서브에이전트 정의                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `agentProgressSummaries`          | `boolean`                                                                                                | `false`                                     | `true`인 경우 서브에이전트에 대한 한 줄 진행 요약을 생성하고 `summary` 필드를 통해 [`task_progress`](#sdktaskprogressmessage) 이벤트로 전달함. 포그라운드 및 백그라운드 서브에이전트에 적용됨                                                                                                                                                                                                                                                                                                                                                                                                           |
| `allowDangerouslySkipPermissions` | `boolean`                                                                                                | `false`                                     | 권한 우회 활성화. `permissionMode: 'bypassPermissions'`를 사용할 때 필요함                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `allowedTools`                    | `string[]`                                                                                               | `[]`                                        | 프롬프트 없이 자동 승인할 도구. 이는 Claude를 이 도구들로만 제한하지 않으며, 목록에 없는 도구는 `permissionMode` 및 `canUseTool`로 넘어감. 도구를 차단하려면 `disallowedTools`를 사용하세요. [권한](/docs/en/agent-sdk/permissions#allow-and-deny-rules) 참조                                                                                                                                                                                                                                                                                                                                        |
| `betas`                           | [`SdkBeta`](#sdkbeta)`[]`                                                                                | `[]`                                        | 베타 기능 활성화                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `canUseTool`                      | [`CanUseTool`](#canusetool)                                                                              | `undefined`                                 | 커스텀 권한 함수. [권한 평가 흐름](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)이 프롬프트로 넘어갈 때만 호출됨. `allowedTools` 항목, 허용 규칙 또는 `acceptEdits`, `bypassPermissions`와 같은 `permissionMode`에 의해 이미 자동 승인된 호출에 대해서는 호출되지 않음. `AskUserQuestion`, [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 허용하더라도 함수에 도달함; `dontAsk` 모드에서는 함수 호출 대신 거부됨. 자세한 내용은 [`CanUseTool`](#canusetool) 참조 |
| `continue`                        | `boolean`                                                                                                | `false`                                     | 가장 최근 대화 계속 진행                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `cwd`                             | `string`                                                                                                 | `process.cwd()`                             | 현재 작업 디렉터리                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `debug`                           | `boolean`                                                                                                | `false`                                     | Claude Code 프로세스에 대한 디버그 모드 활성화                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `debugFile`                       | `string`                                                                                                 | `undefined`                                 | 디버그 로그를 특정 파일 경로에 작성. 암시적으로 디버그 모드 활성화                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `disallowedTools`                 | `string[]`                                                                                               | `[]`                                        | 거부할 도구. `"Bash"`와 같은 단순 이름은 Claude의 컨텍스트에서 도구를 제거함. `"Bash(rm *)"`와 같은 범위 규칙은 도구를 사용할 수 있는 상태로 두고 `bypassPermissions`를 포함한 모든 권한 모드에서 일치하는 호출을 거부함. [권한](/docs/en/agent-sdk/permissions#allow-and-deny-rules) 참조                                                                                                                                                                                                                                                                                                            |
| `effort`                          | `'low' \| 'medium' \| 'high' \| 'xhigh' \| 'max'`                                                        | 모델 기본값                                 | Claude가 응답에 들이는 노력 조절. 추론 깊이를 안내하기 위해 적응형 추론과 함께 작동함. [노력 수준 조절](/docs/en/model-config#adjust-effort-level) 참조                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `enableFileCheckpointing`         | `boolean`                                                                                                | `false`                                     | 되돌리기를 위한 파일 변경 추적 활성화. [파일 체크포인팅](/docs/en/agent-sdk/file-checkpointing) 참조                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `env`                             | `Record<string, string \| undefined>`                                                                    | `process.env`                               | 환경 변수. 설정되면 `process.env`와 병합하는 대신 하위 프로세스 환경을 교체하므로, `PATH`와 같은 상속된 변수를 유지하려면 `{ ...process.env, YOUR_VAR: 'value' }`를 전달하세요. 이 패턴의 예시는 [느리거나 중단된 API 응답 처리](#handle-slow-or-stalled-api-responses)를 참조하고, 기본 CLI가 읽는 변수는 [환경 변수](/docs/en/env-vars)를 참조하세요. User-Agent 헤더에서 앱을 식별하려면 `CLAUDE_AGENT_SDK_CLIENT_APP`을 설정하세요                                                                                                                                                                      |
| `executable`                      | `'bun' \| 'deno' \| 'node'`                                                                              | 자동 감지                                   | 사용할 JavaScript 런타임                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `executableArgs`                  | `string[]`                                                                                               | `[]`                                        | 실행 파일에 전달할 인자                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `extraArgs`                       | `Record<string, string \| null>`                                                                         | `{}`                                        | 추가 인자                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `fallbackModel`                   | `string`                                                                                                 | `undefined`                                 | 주 모델이 실패할 경우 사용할 모델                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

프로그래밍 방식 옵션(`agents`, `allowedTools`, `settings` 등)은 사용자, 프로젝트 및 로컬 파일 시스템 설정을 오버라이드합니다. 관리 정책 설정은 프로그래밍 방식 옵션보다 우선합니다.

### `PermissionMode`

```typescript theme={null}
type PermissionMode =
  | "default" // 표준 권한 동작
  | "acceptEdits" // 파일 편집 자동 수용
  | "bypassPermissions" // 권한 검사 우회; 명시적 ask 규칙은 여전히 프롬프트 표시
  | "plan" // 계획 모드 - 편집 없이 탐색
  | "dontAsk" // 권한 프롬프트 표시 안 함, 사전 승인되지 않은 경우 거부
  | "auto"; // 모델 분류기가 권한 프롬프트 승인 또는 거부
```

### `CanUseTool`

도구 사용을 제어하기 위한 커스텀 권한 함수 타입입니다.

이 함수는 대화형 권한 프롬프트를 대체하는 SDK 용도입니다: [권한 평가 흐름](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)이 프롬프트로 확인되는 경우에만 호출됩니다. `allowedTools` 항목, 설정 허용 규칙, 또는 `acceptEdits` / `bypassPermissions`와 같은 권한 모드에 의해 이미 승인된 도구 호출은 이를 호출하지 않습니다. 모든 도구 호출을 제어하려면 대신 [`PreToolUse` 훅](/docs/en/agent-sdk/hooks)을 사용하세요.

`AskUserQuestion`, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구 및 [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구는 허용 규칙이 일치하더라도 이 함수에 도달합니다. `dontAsk` 모드에서는 이러한 호출이 호출되지 않고 대신 거부됩니다.

```typescript theme={null}
type CanUseTool = (
  toolName: string,
  input: Record<string, unknown>,
  options: {
    signal: AbortSignal;
    suggestions?: PermissionUpdate[];
    blockedPath?: string;
    decisionReason?: string;
    toolUseID: string;
    agentID?: string;
    requestId: string;
  }
) => Promise<PermissionResult | null>;
```

| 옵션             | 타입                                        | 설명                                                                                                                                                                                                                                                                                                  |
| :--------------- | :------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `signal`         | `AbortSignal`                               | 작업을 중단해야 하는 경우 신호 발생                                                                                                                                                                                                                                                                         |
| `suggestions`    | [`PermissionUpdate`](#permissionupdate)`[]` | 사용자가 이 도구에 대해 다시 프롬프트되지 않도록 제안된 권한 업데이트. Bash 프롬프트에는 `localSettings` [대상](#permissionupdatedestination)과의 제안이 포함되어 있어 `updatedPermissions`로 반환하면 규칙이 `.claude/settings.local.json`에 기록되고 세션 간에 지속됨                                |
| `blockedPath`    | `string`                                    | 해당되는 경우 권한 요청을 트리거한 파일 경로                                                                                                                                                                                                                                                                 |
| `decisionReason` | `string`                                    | 이 권한 요청이 트리거된 이유 설명                                                                                                                                                                                                                                                                              |
| `toolUseID`      | `string`                                    | 어시스턴트 메시지 내에서 이 특정 도구 호출에 대한 고유 식별자                                                                                                                                                                                                                                                |
| `agentID`        | `string`                                    | 서브에이전트 내에서 실행 중인 경우 서브에이전트의 ID                                                                                                                                                                                                                                                          |
| `requestId`      | `string`                                    | `control_request` 엔벨로프의 `request_id`. 서명된 HTTP POST와 같이 애플리케이션이 SDK 외부로 전송하는 `control_response`는 이 값을 에코해야 Claude Code 프로세스가 응답을 요청과 매칭할 수 있음                                                                                                            |

콜백은 일반적으로 SDK가 전송을 통해 `control_response`로 다시 작성하는 [`PermissionResult`](#permissionresult)를 반환하여 요청을 해결합니다. 애플리케이션이 `requestId`를 에코하면서 자체 채널을 통해 이 요청에 대한 `control_response`를 이미 전송한 경우에만 `null`을 반환하면, SDK는 전송에 응답을 작성하는 것을 건너끕니다. 다른 경우에 `null`을 반환하면 `control_response`가 전송되지 않고 권한 프롬프트가 타임아웃되지 않으므로 도구 호출이 무기한 차단됩니다.

`requestId` 옵션 및 `null` 반환 값은 Claude Code v2.1.199 이상이 필요합니다.

### `PermissionResult`

권한 검사의 결과입니다.

```typescript theme={null}
type PermissionResult =
  | {
      behavior: "allow";
      updatedInput?: Record<string, unknown>;
      updatedPermissions?: PermissionUpdate[];
      toolUseID?: string;
    }
  | {
      behavior: "deny";
      message: string;
      interrupt?: boolean;
      toolUseID?: string;
    };
```

### `ToolConfig`

내장 도구 동작에 대한 구성입니다.

```typescript theme={null}
type ToolConfig = {
  askUserQuestion?: {
    previewFormat?: "markdown" | "html";
  };
};
```

| 필드                            | 타입                   | 설명                                                                                                                                                                                |
| :------------------------------ | :--------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `askUserQuestion.previewFormat` | `'markdown' \| 'html'` | [`AskUserQuestion`](/docs/en/agent-sdk/user-input#question-format) 옵션의 `preview` 필드를 선택하고 콘텐츠 형식을 설정함. 미설정 시 Claude는 미리보기를 출력하지 않음               |

### `McpServerConfig`

MCP 서버용 구성입니다.

```typescript theme={null}
type McpServerConfig =
  | McpStdioServerConfig
  | McpSSEServerConfig
  | McpHttpServerConfig
  | McpSdkServerConfigWithInstance;
```

#### `McpStdioServerConfig`

```typescript theme={null}
type McpStdioServerConfig = {
  type?: "stdio";
  command: string;
  args?: string[];
  env?: Record<string, string>;
};
```

#### `McpSSEServerConfig`

```typescript theme={null}
type McpSSEServerConfig = {
  type: "sse";
  url: string;
  headers?: Record<string, string>;
};
```

#### `McpHttpServerConfig`

```typescript theme={null}
type McpHttpServerConfig = {
  type: "http";
  url: string;
  headers?: Record<string, string>;
};
```

#### `McpSdkServerConfigWithInstance`

```typescript theme={null}
type McpSdkServerConfigWithInstance = {
  type: "sdk";
  name: string;
  instance: McpServer;
};
```

#### `McpClaudeAIProxyServerConfig`

```typescript theme={null}
type McpClaudeAIProxyServerConfig = {
  type: "claudeai-proxy";
  url: string;
  id: string;
};
```

### `SdkPluginConfig`

SDK에서 플러그인을 로드하기 위한 구성입니다.

```typescript theme={null}
type SdkPluginConfig = {
  type: "local";
  path: string;
  skipMcpDiscovery?: boolean;
};
```

| 필드               | 타입      | 설명                                                                                                                                                                                                                       |
| :----------------- | :-------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`             | `'local'` | `'local'`이어야 함 (현재는 로컬 플러그인만 지원됨)                                                                                                                                                                        |
| `path`             | `string`  | 플러그인 디렉터리의 절대 또는 상대 경로                                                                                                                                                                                    |
| `skipMcpDiscovery` | `boolean` | `true`인 경우 SDK가 이 플러그인에서 스킬, 훅, 에이전트 및 명령어를 로드하지만 해당 `.mcp.json` 또는 매니페스트 `mcpServers`를 읽지 않습니다. 애플리케이션이 플러그인의 MCP 연결을 소유할 때 설정하세요.                  |

**예시:**

```typescript theme={null}
plugins: [
  { type: "local", path: "./my-plugin" },
  { type: "local", path: "/absolute/path/to/plugin" }
];
```

플러그인 생성 및 사용에 대한 완전한 정보는 [플러그인](/docs/en/agent-sdk/plugins)을 참조하세요.

## 메시지 타입 (Message Types)

### `SDKMessage`

쿼리에 의해 반환되는 모든 가능한 메시지의 유니온 타입입니다.

```typescript theme={null}
type SDKMessage =
  | SDKAssistantMessage
  | SDKUserMessage
  | SDKUserMessageReplay
  | SDKResultMessage
  | SDKSystemMessage
  | SDKPartialAssistantMessage
  | SDKCompactBoundaryMessage
  | SDKStatusMessage
  | SDKLocalCommandOutputMessage
  | SDKHookStartedMessage
  | SDKHookProgressMessage
  | SDKHookResponseMessage
  | SDKPluginInstallMessage
  | SDKToolProgressMessage
  | SDKAuthStatusMessage
  | SDKTaskNotificationMessage
  | SDKTaskStartedMessage
  | SDKTaskProgressMessage
  | SDKTaskUpdatedMessage
  | SDKBackgroundTasksChangedMessage
  | SDKThinkingTokensMessage
  | SDKSessionStateChangedMessage
  | SDKWorkerShuttingDownMessage
  | SDKCommandsChangedMessage
  | SDKNotificationMessage
  | SDKFilesPersistedEvent
  | SDKToolUseSummaryMessage
  | SDKMemoryRecallMessage
  | SDKRateLimitEvent
  | SDKElicitationCompleteMessage
  | SDKPermissionDeniedMessage
  | SDKPromptSuggestionMessage
  | SDKAPIRetryMessage
  | SDKMirrorErrorMessage
  | SDKInformationalMessage
  | SDKConversationResetMessage;
```

### `SDKAssistantMessage`

어시스턴트 응답 메시지입니다.

```typescript theme={null}
type SDKAssistantMessage = {
  type: "assistant";
  uuid: UUID;
  session_id: string;
  message: BetaMessage; // Anthropic SDK 출처
  parent_tool_use_id: string | null;
  error?: SDKAssistantMessageError;
  timestamp?: string;
};
```

`message` 필드는 Anthropic SDK의 [`BetaMessage`](https://platform.claude.com/docs/en/api/messages/create)입니다. `id`, `content`, `model`, `stop_reason`, `usage`와 같은 필드가 포함됩니다.

`SDKAssistantMessageError`는 `'authentication_failed'`, `'oauth_org_not_allowed'`, `'billing_error'`, `'rate_limit'`, `'overloaded'`, `'invalid_request'`, `'model_not_found'`, `'server_error'`, `'max_output_tokens'`, 또는 `'unknown'` 중 하나입니다. `'model_not_found'`는 선택한 모델이 존재하지 않거나 계정/배포에서 사용할 수 없음을 의미합니다. `'overloaded'`는 할당량에 대한 429인 `'rate_limit'`과 달리 서버 용량이 초과되어 API가 529를 반환했음을 의미합니다.

`timestamp`는 메시지 콘텐츠 생성이 완료된 ISO 8601 시각입니다. 이 값은 해당 머신의 시계에서 발생하므로 표시용으로만 사용하고 메시지 정렬 기준으로 사용하지 마세요. 하나의 API 차례는 `message.id`를 공유하는 여러 어시스턴트 메시지를 생성할 수 있으며 각각 자체 `timestamp`를 갖습니다. 필드가 없으면 메시지를 수신한 시간으로 폴백합니다.

### `SDKUserMessage`

사용자 입력 메시지입니다.

```typescript theme={null}
type SDKUserMessage = {
  type: "user";
  uuid?: UUID;
  session_id?: string;
  message: MessageParam; // Anthropic SDK 출처
  parent_tool_use_id: string | null;
  isSynthetic?: boolean;
  shouldQuery?: boolean;
  tool_use_result?: unknown;
  origin?: SDKMessageOrigin;
};
```

어시스턴트 차례를 트리거하지 않고 트랜스크립트에 메시지를 추가하려면 `shouldQuery`를 `false`로 설정하세요. 메시지는 대기 상태로 유지되어 차례를 트리거하는 다음 사용자 메시지에 병합됩니다. 모델 호출을 발생시키지 않고 대역 외로 실행한 명령어의 출력과 같은 컨텍스트를 주입할 때 이를 사용하세요.

`tool_result` 블록을 전달하는 메시지에서 `tool_use_result`는 모델에 전송된 텍스트가 아닌 도구의 구조화된 출력 객체입니다. 형상은 일치하는 `tool_use` 블록에 지정된 도구에 따라 달라지므로 필드는 `unknown`으로 타입 지정됩니다. 내장 형상은 [도구 출력 타입](#tool-output-types)에 나열되어 있습니다.

`Agent` 도구의 경우 `tool_use_result`는 [`AgentOutput`](#agent-2)입니다. `completed` 결과에서 `content`는 Claude Code가 `tool_result` 텍스트에 추가하는 에이전트 ID 및 사용량 트레일러 없이 서브에이전트의 보고서를 포함하므로 해당 텍스트를 파싱하는 대신 `tool_use_result`에서 렌더링하세요.

### `SDKUserMessageReplay`

필수 UUID가 포함된 재생된 사용자 메시지입니다.

```typescript theme={null}
type SDKUserMessageReplay = {
  type: "user";
  uuid: UUID;
  session_id: string;
  message: MessageParam;
  parent_tool_use_id: string | null;
  isSynthetic?: boolean;
  tool_use_result?: unknown;
  origin?: SDKMessageOrigin;
  isReplay: true;
};
```

세션 외부에서 주입된 사용자 차례([`origin`](#sdkmessageorigin) 종류가 `peer` 또는 `channel`인 차례)는 활성 차례 중에 전달되었거나 세션이 유휴 상태일 때 새 차례를 시작했는지 여부에 관계없이 리플레이로 스트림에 도달합니다. v2.1.207 이전에는 세션이 유휴 상태일 때 전달된 주입 차례가 스트림에 메시지를 생성하지 않고 트랜스크립트를 다시 읽을 때만 나타났습니다.

### `SDKResultMessage`

최종 결과 메시지입니다.

```typescript theme={null}
type SDKResultMessage =
  | {
      type: "result";
      subtype: "success";
      uuid: UUID;
      session_id: string;
      duration_ms: number;
      duration_api_ms: number;
      is_error: boolean;
      api_error_status?: number | null;
      num_turns: number;
      result: string;
      stop_reason: string | null;
      ttft_ms?: number;
      ttft_stream_ms?: number;
      total_cost_usd: number;
      usage: NonNullableUsage;
      modelUsage: { [modelName: string]: ModelUsage };
      permission_denials: SDKPermissionDenial[];
      structured_output?: unknown;
      deferred_tool_use?: { id: string; name: string; input: Record<string, unknown> };
      terminal_reason?: TerminalReason;
      fast_mode_state?: FastModeState;
      origin?: SDKMessageOrigin;
    }
  | {
      type: "result";
      subtype:
        | "error_max_turns"
        | "error_during_execution"
        | "error_max_budget_usd"
        | "error_max_structured_output_retries";
      uuid: UUID;
      session_id: string;
      duration_ms: number;
      duration_api_ms: number;
      is_error: boolean;
      num_turns: number;
      stop_reason: string | null;
      total_cost_usd: number;
      usage: NonNullableUsage;
      modelUsage: { [modelName: string]: ModelUsage };
      permission_denials: SDKPermissionDenial[];
      errors: string[];
      terminal_reason?: TerminalReason;
      fast_mode_state?: FastModeState;
      origin?: SDKMessageOrigin;
    };
```

결과의 여러 필드는 `subtype` 외의 진단 세부 정보를 수신합니다:

* `api_error_status`: 대화를 종료한 API 에러의 HTTP 상태 코드. 대화 차례가 API 에러 없이 끝난 경우 없거나 `null`임.
* `ttft_ms`: 첫 번째 완벽한 어시스턴트 메시지가 도착했을 때 측정한 밀리초 단위의 첫 토큰 생성 시간. 성공 분기에만 존재함.
* `ttft_stream_ms`: 응답 스트림이 열릴 때 첫 번째 `message_start` 스트림 이벤트까지의 시간(밀리초). `ttft_ms`보다 작음; 둘 사이의 차이는 첫 번째 메시지를 스트리밍하는 데 소비된 시간임. 성공 분기에만 존재함.
* `terminal_reason`: 루프가 종료된 이유. `"completed"`, `"max_turns"`, `"tool_deferred"`, `"aborted_streaming"`, `"aborted_tools"`, `"hook_stopped"`, `"stop_hook_prevented"`, `"background_requested"`, `"blocking_limit"`, `"rapid_refill_breaker"`, `"prompt_too_long"`, `"image_error"`, `"model_error"`, `"api_error"`, `"malformed_tool_use_exhausted"`, `"budget_exhausted"`, `"structured_output_retry_exhausted"`, `"tool_deferred_unavailable"`, 또는 `"turn_setup_failed"` 중 하나.
* `fast_mode_state`: `"on"`, `"off"`, 또는 `"cooldown"` 중 하나.

`origin` 필드는 이 결과를 트리거한 사용자 메시지의 [`SDKMessageOrigin`](#sdkmessageorigin)을 전달합니다. 백그라운드 작업이 완료되고 SDK가 합성 후속 차례를 주입하면 결과 `SDKResultMessage`는 `origin: { kind: "task-notification" }`을 전달합니다. 자체 프롬프트에 응답하는 결과와 백그라운드 작업 후속 조치로 출력된 결과를 구별하려면 이 필드를 확인하여 후자를 라우팅하거나 숨기세요. 사용자 차례 전에 출력된 결과(예: 시작 오류)의 경우 이 필드는 없습니다.

`PreToolUse` 훅이 `permissionDecision: "defer"`를 반환하면 결과는 `stop_reason: "tool_deferred"`를 갖고 `deferred_tool_use`는 대기 중인 도구의 `id`, `name`, `input`을 전달합니다. 자체 UI에서 요청을 표시하려면 이 필드를 읽고 동일한 `session_id`로 재개하여 계속 진행하세요. 전체 왕복은 [나중에 실행하도록 도구 호출 지연](/docs/en/hooks#defer-a-tool-call-for-later)을 참조하세요.

### `SDKSystemMessage`

시스템 초기화 메시지입니다.

```typescript theme={null}
type SDKSystemMessage = {
  type: "system";
  subtype: "init";
  uuid: UUID;
  session_id: string;
  agents?: string[];
  apiKeySource: ApiKeySource;
  betas?: string[];
  claude_code_version: string;
  cwd: string;
  tools: string[];
  mcp_servers: {
    name: string;
    status: string;
  }[];
  model: string;
  permissionMode: PermissionMode;
  slash_commands: string[];
  output_style: string;
  skills: string[];
  plugins: { name: string; path: string }[];
  capabilities?: string[];
};
```

`capabilities` 배열은 이 CLI가 구현하는 프로토콜 동작의 이름을 지정하므로 `claude_code_version` 문자열을 비교하는 대신 기능을 감지할 수 있습니다. 이는 오픈 세트입니다. 인식하지 못하는 값은 무시하고 의존하는 특정 기능이 있는지 확인하세요. 이 필드는 Claude Code v2.1.205 이상이 필요하며 이전 CLI에는 존재하지 않습니다.

| 기능 (Capability)       | 의미                                                                                                                                                                         |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `interrupt_receipt_v1` | [`interrupt()`](#query-object)가 중단에서 살아남은 대기열 메시지의 이름을 지정하는 [`SDKControlInterruptResponse`](#sdkcontrolinterruptresponse) 영수증으로 확인됨 |

### `SDKPartialAssistantMessage`

스트리밍 부분 메시지 (`includePartialMessages`가 true일 때만). `parent_tool_use_id` 필드는 항상 `null`입니다: 스트림 이벤트는 메인 세션에 대해서만 출력됩니다. 서브에이전트 귀속을 위해 `parent_tool_use_id`를 전달하는 완전한 메시지를 사용하거나 [`forwardSubagentText`](#options)를 활성화하여 서브에이전트 텍스트 및 추론을 완전한 메시지로 수신하세요.

```typescript theme={null}
type SDKPartialAssistantMessage = {
  type: "stream_event";
  event: BetaRawMessageStreamEvent; // Anthropic SDK 출처
  parent_tool_use_id: string | null;
  uuid: UUID;
  session_id: string;
  ttft_ms?: number; // ms 단위의 첫 토큰 생성 시간, message_start 이벤트에만 존재
};
```

### `SDKCompactBoundaryMessage`

대화 압축 경계를 나타내는 메시지입니다.

```typescript theme={null}
type SDKCompactBoundaryMessage = {
  type: "system";
  subtype: "compact_boundary";
  uuid: UUID;
  session_id: string;
  compact_metadata: {
    trigger: "manual" | "auto";
    pre_tokens: number;
  };
};
```

### `SDKInformationalMessage`

루프에서 출력되는 일반 텍스트 배너입니다. 비-오류 상태 줄, `UserPromptSubmit` 훅의 차단 이유와 같은 훅 피드백 및 명령어 출력을 전달합니다. 주어진 `level`에 따라 `content`를 일반 텍스트로 렌더링하세요.

```typescript theme={null}
type SDKInformationalMessage = {
  type: "system";
  subtype: "informational";
  content: string;
  level: "info" | "notice" | "suggestion" | "warning";
  tool_use_id?: string;
  prevent_continuation?: boolean;
  uuid: UUID;
  session_id: string;
};
```

### `SDKWorkerShuttingDownMessage`

원격 클라이언트가 하트비트 타임아웃을 기다리는 대신 워커가 종료된 이유를 표시할 수 있도록 정상적인 워커 종료 시 출력됩니다. `reason`은 호스트 CLI가 설정한 짧은 snake\_case 문자열입니다(예: `"host_exit"` 또는 `"remote_control_disabled"`). 라이브 스트리밍 시에만 이에 대응하세요. 재개된 세션은 이 메시지의 이전 인스턴스를 재생하므로 이 경우에는 무시하세요.

```typescript theme={null}
type SDKWorkerShuttingDownMessage = {
  type: "system";
  subtype: "worker_shutting_down";
  reason: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKPluginInstallMessage`

플러그인 설치 진행 이벤트입니다. Agent SDK 애플리케이션이 첫 차례 전에 마켓플레이스 플러그인 설치를 추적할 수 있도록 [`CLAUDE_CODE_SYNC_PLUGIN_INSTALL`](/docs/en/env-vars)이 설정되었을 때 출력됩니다. `started` 및 `completed` 상태는 전체 설치를 중괄호로 묶습니다. `installed` 및 `failed` 상태는 개별 마켓플레이스를 보고하고 `name`을 포함합니다.

```typescript theme={null}
type SDKPluginInstallMessage = {
  type: "system";
  subtype: "plugin_install";
  status: "started" | "installed" | "failed" | "completed";
  name?: string;
  error?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKPermissionDeniedMessage`

권한 시스템이 대화형 프롬프트 없이 도구 호출을 자동 거부할 때 출력되는 스트림 이벤트입니다. 나중에 뒤따르는 `is_error` 도구 결과만 관찰하는 대신 UI에서 거부되는 즉시 렌더링할 때 사용하세요. 대화형 질문 경로는 [`canUseTool`](#canusetool) 콜백을 통해 애플리케이션에 별도로 도달합니다. `PreToolUse` 훅이 발행한 거부는 이 이벤트를 통해 보고되지 않습니다.

이 이벤트는 Claude Code v2.1.136 이상이 필요합니다.

```typescript theme={null}
type SDKPermissionDeniedMessage = {
  type: "system";
  subtype: "permission_denied";
  tool_name: string;
  tool_use_id: string;
  agent_id?: string;
  decision_reason_type?: string;
  decision_reason?: string;
  message: string;
  uuid: UUID;
  session_id: string;
};
```

| 필드                   | 타입     | 설명                                                                                                                      |
| ---------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `tool_name`            | `string` | 거부된 도구의 이름                                                                                                        |
| `tool_use_id`          | `string` | 이 거부가 응답하는 `tool_use` 블록의 ID                                                                                    |
| `agent_id`             | `string` | 거부된 호출이 서브에이전트 내부에서 시작된 경우 서브에이전트 ID. 호스트 측 라우팅을 위한 `can_use_tool` 필드를 미러링함     |
| `decision_reason_type` | `string` | 결정을 내린 구성 요소에 대한 판별자(discriminator)(예: `"rule"`, `"mode"`, `"classifier"`, `"asyncAgent"`)                |
| `decision_reason`      | `string` | 사용 가능한 경우 결정을 내린 구성 요소의 사람이 읽을 수 있는 이유                                                         |
| `message`              | `string` | `tool_result`에서 모델에 반환된 거부 메시지                                                                               |

### `SDKPermissionDenial`

거부된 도구 사용에 대한 정보입니다.

```typescript theme={null}
type SDKPermissionDenial = {
  tool_name: string;
  tool_use_id: string;
  tool_input: Record<string, unknown>;
};
```

### `SDKMessageOrigin`

사용자 역할 메시지의 출처입니다. 이는 [`SDKUserMessage`](#sdkusermessage)의 `origin`으로 나타나며 해당 [`SDKResultMessage`](#sdkresultmessage)로 전달되어 특정 차례를 트리거한 원인을 식별할 수 있습니다.

```typescript theme={null}
type SDKMessageOrigin =
  | { kind: "human" }
  | { kind: "channel"; server: string }
  | {
      kind: "peer";
      from: string;
      name?: string;
      senderTaskId?: string;
      body?: string;
    }
  | { kind: "task-notification" }
  | { kind: "coordinator" }
  | { kind: "auto-continuation" };
```

| `kind`              | 의미                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `human`             | 최종 사용자의 직접적인 입력. 애플리케이션이 사용자가 입력한 내용을 사용자 메시지로 전달하는 경우 `origin`을 `{ kind: "human" }`으로 명시적으로 설정하세요: Claude Code는 `origin`이 없는 사용자 메시지를 출처 미지정으로 처리하며, [`ultracode` 워크플로우 키워드](/docs/en/workflows#ask-for-a-workflow-in-your-prompt)와 같이 사람이 입력한 프롬프트를 요구하는 검사는 이를 수락하지 않습니다. v2.1.210 이전에는 Claude Code가 사용자 메시지의 없는 `origin`을 사람의 입력으로 처리했습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `channel`           | [채널](/docs/en/channels)에 도착하는 메시지. `server`는 소스 MCP 서버 이름입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `peer`              | 다른 에이전트의 메시지. `SendMessage`를 통해 `main`으로 보내는 프로세스 내 [동료 에이전트](/docs/en/agent-teams)의 경우 `from`은 동료의 이름이고 `senderTaskId`는 해당 작업 ID입니다. 다른 로컬 Claude Code 프로세스와 같은 교차 세션 동료의 경우 `from`은 보낸 사람 주소이고 `senderTaskId`는 없습니다. `name`과 `body`는 Claude Code v2.1.205 이상이 필요합니다. `name`은 보낸 사람의 표시 이름이며 Claude Code에 의해 정규화됩니다(유니코드 제어, 형식, 서러게이트, 줄 또는 단락 구분 기호 코드 포인트를 제거한 후 결과를 트림하고 64개 코드 포인트로 캡핑함). `body`는 모델이 보는 것과 바이트 단위로 동일한, 동료 엔벨로프가 제거된 디코딩된 메시지 본문입니다. 동료 메시지의 경우 `body`는 항상 존재합니다. 교차 세션 동료의 경우 차례가 Claude Code가 형성한 하나의 동료 엔벨로프일 때만 존재합니다. 메시지 텍스트를 다시 파싱하는 대신 `name`과 `body`를 렌더링하세요. |
| `task-notification` | 백그라운드 작업이 완료된 후 주입된 합성 차례. [`SDKTaskNotificationMessage`](#sdktasknotificationmessage) 참조.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `coordinator`       | [에이전트 팀](/docs/en/agent-teams)의 팀 코디네이터 메시지.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `auto-continuation` | 후속 프롬프트를 트리거하는 명령어 결과와 같이 새로운 사용자 입력 없이 세션이 계속될 때 주입된 합성 차례.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

## 훅 타입 (Hook Types)

예시 및 일반적인 패턴이 포함된 훅 사용에 대한 포괄적인 가이드는 [훅 가이드](/docs/en/agent-sdk/hooks)를 참조하세요.

### `HookEvent`

사용 가능한 훅 이벤트입니다.

```typescript theme={null}
type HookEvent =
  | "PreToolUse"
  | "PostToolUse"
  | "PostToolUseFailure"
  | "PostToolBatch"
  | "Notification"
  | "UserPromptSubmit"
  | "UserPromptExpansion"
  | "SessionStart"
  | "SessionEnd"
  | "Stop"
  | "SubagentStart"
  | "SubagentStop"
  | "PreCompact"
  | "PermissionRequest"
  | "Setup"
  | "TeammateIdle"
  | "TaskCompleted"
  | "ConfigChange"
  | "WorktreeCreate"
  | "WorktreeRemove"
  | "MessageDisplay";
```

### `HookCallback`

훅 콜백 함수 타입입니다.

```typescript theme={null}
type HookCallback = (
  input: HookInput, // 모든 훅 입력 타입의 유니온
  toolUseID: string | undefined,
  options: { signal: AbortSignal }
) => Promise<HookJSONOutput>;
```

### `HookCallbackMatcher`

선택적 매처가 있는 훅 구성입니다.

```typescript theme={null}
interface HookCallbackMatcher {
  matcher?: string;
  hooks: HookCallback[];
  timeout?: number; // 이 매처의 모든 훅에 대한 초 단위 타임아웃
}
```

### `HookInput`

모든 훅 입력 타입의 유니온 타입입니다.

```typescript theme={null}
type HookInput =
  | PreToolUseHookInput
  | PostToolUseHookInput
  | PostToolUseFailureHookInput
  | PostToolBatchHookInput
  | NotificationHookInput
  | UserPromptSubmitHookInput
  | SessionStartHookInput
  | SessionEndHookInput
  | StopHookInput
  | SubagentStartHookInput
  | SubagentStopHookInput
  | PreCompactHookInput
  | PermissionRequestHookInput
  | SetupHookInput
  | TeammateIdleHookInput
  | TaskCompletedHookInput
  | ConfigChangeHookInput
  | WorktreeCreateHookInput
  | WorktreeRemoveHookInput
  | MessageDisplayHookInput;
```

### `BaseHookInput`

모든 훅 입력 타입이 확장하는 기본 인터페이스입니다.

```typescript theme={null}
type BaseHookInput = {
  session_id: string;
  transcript_path: string;
  cwd: string;
  prompt_id?: string;
  permission_mode?: string;
  effort?: { level: string };
  agent_id?: string;
  agent_type?: string;
};
```

`prompt_id` 필드는 현재 처리 중인 사용자 프롬프트를 식별하는 UUID입니다. 이는 [OpenTelemetry 이벤트의 `prompt.id` 속성](/docs/en/monitoring-usage#event-correlation-attributes)과 일치하며 첫 번째 사용자 입력 전에는 없습니다. Claude Code v2.1.196 이상이 필요합니다.

#### `PreToolUseHookInput`

```typescript theme={null}
type PreToolUseHookInput = BaseHookInput & {
  hook_event_name: "PreToolUse";
  tool_name: string;
  tool_input: unknown;
  tool_use_id: string;
};
```

#### `PostToolUseHookInput`

```typescript theme={null}
type PostToolUseHookInput = BaseHookInput & {
  hook_event_name: "PostToolUse";
  tool_name: string;
  tool_input: unknown;
  tool_response: unknown;
  tool_use_id: string;
  duration_ms?: number;
};
```

#### `PostToolUseFailureHookInput`

```typescript theme={null}
type PostToolUseFailureHookInput = BaseHookInput & {
  hook_event_name: "PostToolUseFailure";
  tool_name: string;
  tool_input: unknown;
  tool_use_id: string;
  error: string;
  is_interrupt?: boolean;
  duration_ms?: number;
};
```

#### `PostToolBatchHookInput`

배치 내의 모든 도구 호출이 해결된 후, 다음 모델 요청 전에 한 번 트리거됩니다. `tool_response`는 모델이 보는 직렬화된 `tool_result` 콘텐츠를 전달합니다. 형상은 `PostToolUseHookInput`의 구조화된 `Output` 객체와 다릅니다.

```typescript theme={null}
type PostToolBatchHookInput = BaseHookInput & {
  hook_event_name: "PostToolBatch";
  tool_calls: PostToolBatchToolCall[];
};

type PostToolBatchToolCall = {
  tool_name: string;
  tool_input: unknown;
  tool_use_id: string;
  tool_response?: unknown;
};
```

#### `NotificationHookInput`

```typescript theme={null}
type NotificationHookInput = BaseHookInput & {
  hook_event_name: "Notification";
  message: string;
  title?: string;
  notification_type: string;
};
```

#### `UserPromptSubmitHookInput`

```typescript theme={null}
type UserPromptSubmitHookInput = BaseHookInput & {
  hook_event_name: "UserPromptSubmit";
  prompt: string;
};
```

#### `SessionStartHookInput`

```typescript theme={null}
type SessionStartHookInput = BaseHookInput & {
  hook_event_name: "SessionStart";
  source: "startup" | "resume" | "clear" | "compact";
  agent_type?: string;
  model?: string;
};
```

#### `SessionEndHookInput`

```typescript theme={null}
type SessionEndHookInput = BaseHookInput & {
  hook_event_name: "SessionEnd";
  reason: ExitReason; // EXIT_REASONS 배열의 문자열
};
```

#### `StopHookInput`

```typescript theme={null}
type StopHookInput = BaseHookInput & {
  hook_event_name: "Stop";
  stop_hook_active: boolean;
  last_assistant_message?: string;
  background_tasks?: BackgroundTaskSummary[];
  session_crons?: SessionCronSummary[];
};
```

#### `SubagentStartHookInput`

```typescript theme={null}
type SubagentStartHookInput = BaseHookInput & {
  hook_event_name: "SubagentStart";
  agent_id: string;
  agent_type: string;
};
```

#### `SubagentStopHookInput`

```typescript theme={null}
type SubagentStopHookInput = BaseHookInput & {
  hook_event_name: "SubagentStop";
  stop_hook_active: boolean;
  agent_id: string;
  agent_transcript_path: string;
  agent_type: string;
  last_assistant_message?: string;
  background_tasks?: BackgroundTaskSummary[];
  session_crons?: SessionCronSummary[];
};

type BackgroundTaskSummary = {
  id: string;
  type: string;
  status: string;
  description: string;
  command?: string;
  agent_type?: string;
  server?: string;
  tool?: string;
  name?: string;
};

type SessionCronSummary = {
  id: string;
  schedule: string;
  recurring: boolean;
  prompt: string;
};
```

#### `PreCompactHookInput`

```typescript theme={null}
type PreCompactHookInput = BaseHookInput & {
  hook_event_name: "PreCompact";
  trigger: "manual" | "auto";
  custom_instructions: string | null;
};
```

#### `PermissionRequestHookInput`

```typescript theme={null}
type PermissionRequestHookInput = BaseHookInput & {
  hook_event_name: "PermissionRequest";
  tool_name: string;
  tool_input: unknown;
  permission_suggestions?: PermissionUpdate[];
};
```

#### `SetupHookInput`

```typescript theme={null}
type SetupHookInput = BaseHookInput & {
  hook_event_name: "Setup";
  trigger: "init" | "maintenance";
};
```

#### `TeammateIdleHookInput`

```typescript theme={null}
type TeammateIdleHookInput = BaseHookInput & {
  hook_event_name: "TeammateIdle";
  teammate_name: string;
  /** @deprecated v2.1.178부터 권장되지 않음. 세션에서 파생된 팀 이름을 전달하며 향후 제거될 예정. */
  team_name: string;
};
```

#### `TaskCompletedHookInput`

```typescript theme={null}
type TaskCompletedHookInput = BaseHookInput & {
  hook_event_name: "TaskCompleted";
  task_id: string;
  task_subject: string;
  task_description?: string;
  teammate_name?: string;
  /** @deprecated v2.1.178부터 권장되지 않음. 세션에서 파생된 팀 이름을 전달하며 향후 제거될 예정. */
  team_name?: string;
};
```

#### `ConfigChangeHookInput`

```typescript theme={null}
type ConfigChangeHookInput = BaseHookInput & {
  hook_event_name: "ConfigChange";
  source:
    | "user_settings"
    | "project_settings"
    | "local_settings"
    | "policy_settings"
    | "skills";
  file_path?: string;
};
```

#### `WorktreeCreateHookInput`

```typescript theme={null}
type WorktreeCreateHookInput = BaseHookInput & {
  hook_event_name: "WorktreeCreate";
  name: string;
};
```

#### `WorktreeRemoveHookInput`

```typescript theme={null}
type WorktreeRemoveHookInput = BaseHookInput & {
  hook_event_name: "WorktreeRemove";
  worktree_path: string;
};
```

#### `MessageDisplayHookInput`

```typescript theme={null}
type MessageDisplayHookInput = BaseHookInput & {
  hook_event_name: "MessageDisplay";
  turn_id: string;
  message_id: string;
  index: number;
  final: boolean;
  delta: string;
};
```

### `HookJSONOutput`

훅 반환 값입니다.

```typescript theme={null}
type HookJSONOutput = AsyncHookJSONOutput | SyncHookJSONOutput;
```

#### `AsyncHookJSONOutput`

```typescript theme={null}
type AsyncHookJSONOutput = {
  async: true;
  asyncTimeout?: number;
};
```

#### `SyncHookJSONOutput`

```typescript theme={null}
type SyncHookJSONOutput = {
  continue?: boolean;
  suppressOutput?: boolean;
  stopReason?: string;
  decision?: "approve" | "block";
  systemMessage?: string;
  reason?: string;
  hookSpecificOutput?:
    | {
        hookEventName: "PreToolUse";
        permissionDecision?: "allow" | "deny" | "ask" | "defer";
        permissionDecisionReason?: string;
        updatedInput?: Record<string, unknown>;
        additionalContext?: string;
      }
    | {
        hookEventName: "UserPromptSubmit";
        additionalContext?: string;
      }
    | {
        hookEventName: "SessionStart";
        additionalContext?: string;
      }
    | {
        hookEventName: "Setup";
        additionalContext?: string;
      }
    | {
        hookEventName: "SubagentStart";
        additionalContext?: string;
      }
    | {
        hookEventName: "PostToolUse";
        additionalContext?: string;
        updatedToolOutput?: unknown;
        /** @deprecated 모든 도구에서 작동하는 `updatedToolOutput`을 사용하세요. */
        updatedMCPToolOutput?: unknown;
      }
    | {
        hookEventName: "PostToolUseFailure";
        additionalContext?: string;
      }
    | {
        hookEventName: "PostToolBatch";
        additionalContext?: string;
      }
    | {
        hookEventName: "Notification";
        additionalContext?: string;
      }
    | {
        hookEventName: "PermissionRequest";
        decision:
          | {
              behavior: "allow";
              updatedInput?: Record<string, unknown>;
              updatedPermissions?: PermissionUpdate[];
            }
          | {
              behavior: "deny";
              message?: string;
              interrupt?: boolean;
            };
      };
};
```

## 도구 입력 타입 (Tool Input Types)

모든 내장 Claude Code 도구의 입력 스키마에 대한 문서입니다. 이러한 타입은 `@anthropic-ai/claude-agent-sdk`에서 내보내지며 타입 안전한 도구 상호작용에 사용할 수 있습니다.

### `ToolInputSchemas`

모든 도구 입력 타입의 유니온 타입으로, `@anthropic-ai/claude-agent-sdk`에서 내보냅니다.

```typescript theme={null}
type ToolInputSchemas =
  | AgentInput
  | AskUserQuestionInput
  | BashInput
  | TaskOutputInput
  | EnterWorktreeInput
  | ExitPlanModeInput
  | FileEditInput
  | FileReadInput
  | FileWriteInput
  | GlobInput
  | GrepInput
  | ListMcpResourcesInput
  | McpInput
  | MonitorInput
  | NotebookEditInput
  | ReadMcpResourceInput
  | SubscribeMcpResourceInput
  | SubscribePollingInput
  | TaskCreateInput
  | TaskGetInput
  | TaskListInput
  | TaskStopInput
  | TaskUpdateInput
  | TodoWriteInput
  | UnsubscribeMcpResourceInput
  | UnsubscribePollingInput
  | WebFetchInput
  | WebSearchInput
  | WorkflowInput;
```

### Agent

**도구 이름:** `Agent` (이전 `Task`, 여전히 에일리어스로 허용됨)

<Note>
  `mode` 필드는 Claude Code v2.1.212 이상에서 권장되지 않으며 무시됩니다: 서브에이전트는 [부모 세션의 권한 모드를 상속](/docs/en/agent-sdk/permissions#available-modes)하며, 서브에이전트 정의의 [`permissionMode`](#agentdefinition)가 부모가 `bypassPermissions`, `acceptEdits`, 또는 `auto`를 사용할 때를 제외하고 이를 오버라이드할 수 있습니다.
</Note>

```typescript theme={null}
type AgentInput = {
  description: string;
  prompt: string;
  subagent_type?: string;
  model?: "sonnet" | "opus" | "haiku" | "fable";
  run_in_background?: boolean;
  name?: string;
  mode?: "acceptEdits" | "auto" | "bypassPermissions" | "default" | "dontAsk" | "plan";
  isolation?: "worktree" | "remote";
};
```

복잡한 다단계 작업을 자율적으로 처리하기 위해 새 에이전트를 시작합니다.

### AskUserQuestion

**도구 이름:** `AskUserQuestion`

```typescript theme={null}
type AskUserQuestionInput = {
  questions: Array<{
    question: string;
    header: string;
    options: Array<{ label: string; description: string; preview?: string }>;
    multiSelect: boolean;
  }>;
};
```

실행 중 사용자에게 명확한 질문을 합니다. 사용법 세부 정보는 [승인 및 사용자 입력 처리](/docs/en/agent-sdk/user-input#handle-clarifying-questions)를 참조하세요.

### Bash

**도구 이름:** `Bash`

```typescript theme={null}
type BashInput = {
  command: string;
  timeout?: number; // 밀리초, 최대 600000; 높은 값은 최대값으로 클램핑됨
  description?: string;
  run_in_background?: boolean;
  dangerouslyDisableSandbox?: boolean;
};
```

선택적 타임아웃 및 백그라운드 실행을 포함하여 Bash 명령어를 실행합니다. 작업 디렉터리는 명령어 간에 유지되며, 내보낸 환경 변수와 같은 쉘 상태는 유지되지 않습니다.

### Monitor

**도구 이름:** `Monitor`

```typescript theme={null}
type MonitorInput = {
  command?: string;
  ws?: {
    url: string;
    protocols?: string[];
  };
  description: string;
  timeout_ms?: number;
  persistent?: boolean;
};
```

백그라운드 소스를 실행하고 각 이벤트를 Claude에 전달하여 폴링 없이 반응할 수 있게 합니다: `command`는 스크립트를 실행하고 stdout 줄당 하나의 이벤트를 출력하며, `ws`는 WebSocket을 열고 텍스트 프레임당 하나의 이벤트를 출력합니다. `command` 또는 `ws` 중 정확히 하나만 제공하세요. `ws` 소스는 Claude Code v2.1.195 이상이 필요합니다.

로그 타일과 같이 세션 길이의 감시를 수행하려면 `persistent: true`로 설정하세요. Monitor가 명령을 실행할 때는 Bash와 동일한 권한 규칙을 따르며, WebSocket 감시는 별도로 승인을 구합니다. 동작 및 공급자 가용성은 [Monitor 도구 레퍼런스](/docs/en/tools-reference#monitor-tool)를 참조하세요.

### TaskOutput

**도구 이름:** `TaskOutput`

```typescript theme={null}
type TaskOutputInput = {
  task_id: string;
  block: boolean;
  timeout: number;
};
```

실행 중이거나 완료된 백그라운드 작업에서 출력을 가져옵니다.

### Edit

**도구 이름:** `Edit`

```typescript theme={null}
type FileEditInput = {
  file_path: string;
  old_string: string;
  new_string: string;
  replace_all?: boolean;
};
```

파일에서 정확한 문자열 교체를 수행합니다.

### Read

**도구 이름:** `Read`

```typescript theme={null}
type FileReadInput = {
  file_path: string;
  offset?: number;
  limit?: number;
  pages?: string;
};
```

텍스트, 이미지, PDF, Jupyter 노트북을 포함하여 로컬 파일 시스템에서 파일을 읽습니다. PDF 페이지 범위에는 `pages`를 사용하세요(예: `"1-5"`).

### Write

**도구 이름:** `Write`

```typescript theme={null}
type FileWriteInput = {
  file_path: string;
  content: string;
};
```

로컬 파일 시스템에 파일을 작성하며, 기존 파일이 있으면 덮어씁니다.

### Glob

**도구 이름:** `Glob`

```typescript theme={null}
type GlobInput = {
  pattern: string;
  path?: string;
};
```

모든 코드베이스 크기에서 작동하는 빠른 파일 패턴 매칭입니다.

### Grep

**도구 이름:** `Grep`

```typescript theme={null}
type GrepInput = {
  pattern: string;
  path?: string;
  glob?: string;
  type?: string;
  output_mode?: "content" | "files_with_matches" | "count";
  "-i"?: boolean;
  "-n"?: boolean;
  "-B"?: number;
  "-A"?: number;
  "-C"?: number;
  context?: number;
  head_limit?: number;
  offset?: number;
  multiline?: boolean;
};
```

정규식 지원이 포함된 ripgrep 기반의 강력한 검색 도구입니다.

### TaskStop

**도구 이름:** `TaskStop`

```typescript theme={null}
type TaskStopInput = {
  task_id?: string;
  shell_id?: string; // 권장되지 않음: task_id 사용
};
```

ID로 실행 중인 백그라운드 작업 또는 쉘을 중지합니다. v2.1.198부터 `task_id`는 에이전트 팀 동료나 에이전트 ID/이름으로 지정된 백그라운드 에이전트도 허용합니다.

### NotebookEdit

**도구 이름:** `NotebookEdit`

```typescript theme={null}
type NotebookEditInput = {
  notebook_path: string;
  cell_id?: string;
  new_source: string;
  cell_type?: "code" | "markdown";
  edit_mode?: "replace" | "insert" | "delete";
};
```

Jupyter 노트북 파일의 셀을 편집합니다.

### WebFetch

**도구 이름:** `WebFetch`

```typescript theme={null}
type WebFetchInput = {
  url: string;
  prompt: string;
};
```

URL에서 콘텐츠를 가져와 AI 모델로 처리합니다.

### WebSearch

**도구 이름:** `WebSearch`

```typescript theme={null}
type WebSearchInput = {
  query: string;
  allowed_domains?: string[];
  blocked_domains?: string[];
};
```

웹을 검색하고 형식화된 결과를 반환합니다.

### Workflow

**도구 이름:** `Workflow`

```typescript theme={null}
type WorkflowInput = {
  script?: string;
  name?: string;
  scriptPath?: string;
  args?: unknown;
  resumeFromRunId?: string;
};
```

[동적 워크플로우](/docs/en/workflows)를 실행합니다. 백그라운드에서 많은 서브에이전트를 조율하고 하나의 통합된 결과를 반환하는 스크립트입니다. `Workflow` 도구는 Agent SDK v0.3.149 이상에서 사용할 수 있습니다. `script`, `name`, 또는 `scriptPath` 중 하나 이상이 필요합니다.

| 필드                | 타입      | 설명                                                                                                                                                                                                                                                                          |
| ------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `script`            | `string`  | 인라인 워크플로우 스크립트. 리터럴로 `export const meta = { name, description }`으로 시작해야 하며, 그 뒤에 `agent()`, `parallel()`, `pipeline()`, `phase()`를 사용하는 스크립트 본문이 옵니다. `meta`에 있는 선택적 `phases` 배열은 진행 상황 뷰에서 이름이 지정된 단계별로 에이전트를 그룹화합니다 |
| `name`              | `string`  | 내장 워크플로우 또는 `.claude/workflows/`에 저장된 워크플로우 이름. 스크립트로 확인됨                                                                                                                                                                                                |
| `scriptPath`        | `string`  | 디스크의 워크플로우 스크립트 파일 경로. `script` 및 `name`보다 우선함. 모든 호출은 해당 스크립트를 지속하고 결과에 경로를 반환하므로 해당 파일을 편집하고 동일한 `scriptPath`로 다시 호출하여 반복할 수 있습니다                                                                    |
| `args`              | `unknown` | 연구 질문 또는 파일 경로 목록과 같은 매개변수화된 이름 있는 워크플로우를 위해 전역 `args`로 스크립트에 노출되는 입력 값. 배열 및 객체는 JSON 인코딩 문자열이 아닌 실제 JSON 값으로 전달하세요                                                                                         |
| `resumeFromRunId`   | `string`  | 재개할 이전 `Workflow` 호출의 실행 ID. 변경되지 않은 입력을 포함하는 완료된 `agent()` 호출은 캐시된 결과를 반환하고, 변경되거나 새로운 호출만 라이브로 실행됩니다. 동일한 세션만 가능                                                                                               |

### TodoWrite

**도구 이름:** `TodoWrite`

```typescript theme={null}
type TodoWriteInput = {
  todos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
};
```

진행 상황 추적을 위한 구조화된 작업 목록을 생성하고 관리합니다.

<Note>
  TypeScript Agent SDK 0.3.142부터 `TodoWrite`는 기본적으로 비활성화되어 있습니다. 대신 `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList`를 사용하세요. 모니터링 코드를 업데이트하려면 [Task 도구로 마이그레이션](/docs/en/agent-sdk/todo-tracking#migrate-to-task-tools)을 참조하고, `TodoWrite`로 되돌리려면 `CLAUDE_CODE_ENABLE_TASKS=0`을 설정하세요.
</Note>

### TaskCreate

**도구 이름:** `TaskCreate`

```typescript theme={null}
type TaskCreateInput = {
  subject: string;
  description: string;
  activeForm?: string;
  metadata?: Record<string, unknown>;
};
```

단일 작업을 생성하고 할당된 ID를 반환합니다.

### TaskUpdate

**도구 이름:** `TaskUpdate`

```typescript theme={null}
type TaskUpdateInput = {
  taskId: string;
  status?: "pending" | "in_progress" | "completed" | "deleted";
  subject?: string;
  description?: string;
  activeForm?: string;
  addBlocks?: string[];
  addBlockedBy?: string[];
  owner?: string;
  metadata?: Record<string, unknown>;
};
```

ID로 하나의 작업을 수정합니다. 제거하려면 `status`를 `"deleted"`로 설정하세요.

### TaskGet

**도구 이름:** `TaskGet`

```typescript theme={null}
type TaskGetInput = {
  taskId: string;
};
```

하나의 작업에 대한 전체 세부 정보를 반환하며, ID를 찾을 수 없는 경우 `null`을 반환합니다.

### TaskList

**도구 이름:** `TaskList`

```typescript theme={null}
type TaskListInput = {};
```

현재 목록에 있는 모든 작업의 스냅샷을 반환합니다.

### ExitPlanMode

**도구 이름:** `ExitPlanMode`

```typescript theme={null}
type ExitPlanModeInput = {
  /** @deprecated 더 이상 사용되지 않음. */
  allowedPrompts?: Array<{
    tool: "Bash";
    prompt: string;
  }>;
};
```

계획 모드를 종료합니다. `allowedPrompts` 필드는 권장되지 않으며 무시됩니다. Claude Code는 기존 호출자 및 트랜스크립트 검증을 위해 여전히 이를 수락합니다. v2.1.205 이전에는 계획 구현을 위해 프롬프트 기반 Bash 권한을 요청했습니다.

### ListMcpResources

**도구 이름:** `ListMcpResourcesTool`

```typescript theme={null}
type ListMcpResourcesInput = {
  server?: string;
};
```

연결된 서버에서 사용 가능한 MCP 리소스를 나열합니다.

### ReadMcpResource

**도구 이름:** `ReadMcpResourceTool`

```typescript theme={null}
type ReadMcpResourceInput = {
  server: string;
  uri: string;
};
```

서버에서 특정 MCP 리소스를 읽습니다.

### EnterWorktree

**도구 이름:** `EnterWorktree`

```typescript theme={null}
type EnterWorktreeInput = {
  name?: string;
  path?: string;
};
```

격리된 작업을 위해 임시 git 워크트리를 생성하고 진입합니다. 새로운 워크트리를 생성하는 대신 기존 워크트리로 전환하려면 `path`를 전달하세요. 첫 번째 진입 시 대상은 현재 리포지토리의 등록된 워크트리이거나 다중 리포지토리 작업 영역의 경우 내부에 중첩된 리포지토리여야 합니다; 워크트리 세션 내부에서는 세션 리포지토리의 `.claude/worktrees/` 아래에 있어야 합니다. `name`과 `path`는 상호 배타적입니다.

## 도구 출력 타입 (Tool Output Types)

모든 내장 Claude Code 도구의 출력 스키마에 대한 문서입니다. 이러한 타입은 `@anthropic-ai/claude-agent-sdk`에서 내보내지며 각 도구가 반환하는 실제 응답 데이터를 나타냅니다.

### `ToolOutputSchemas`

모든 도구 출력 타입의 유니온 타입입니다.

```typescript theme={null}
type ToolOutputSchemas =
  | AgentOutput
  | AskUserQuestionOutput
  | BashOutput
  | EnterWorktreeOutput
  | ExitPlanModeOutput
  | FileEditOutput
  | FileReadOutput
  | FileWriteOutput
  | GlobOutput
  | GrepOutput
  | ListMcpResourcesOutput
  | MonitorOutput
  | NotebookEditOutput
  | ReadMcpResourceOutput
  | TaskCreateOutput
  | TaskGetOutput
  | TaskListOutput
  | TaskStopOutput
  | TaskUpdateOutput
  | TodoWriteOutput
  | WebFetchOutput
  | WebSearchOutput
  | WorkflowOutput;
```

### Agent

**도구 이름:** `Agent` (이전 `Task`, 여전히 에일리어스로 허용됨)

```typescript theme={null}
type AgentOutput =
  | {
      status: "completed";
      agentId: string;
      agentType?: string;
      content: Array<{ type: "text"; text: string; citations?: unknown[] | null }>;
      resolvedModel?: string;
      modelsUsed?: string[];
      totalToolUseCount: number;
      totalDurationMs: number;
      totalTokens: number;
      usage: {
        input_tokens: number;
        output_tokens: number;
        cache_creation_input_tokens: number | null;
        cache_read_input_tokens: number | null;
        server_tool_use: {
          web_search_requests: number;
          web_fetch_requests: number;
        } | null;
        service_tier: string | null;
        cache_creation: {
          ephemeral_1h_input_tokens: number;
          ephemeral_5m_input_tokens: number;
        } | null;
        inference_geo?: string | null;
        speed?: string | null;
        iterations?: unknown;
      };
      toolStats?: {
        readCount: number;
        searchCount: number;
        bashCount: number;
        editFileCount: number;
        linesAdded: number;
        linesRemoved: number;
        otherToolCount: number;
        frameCount?: number;
      };
      prompt: string;
      worktreePath?: string;
      worktreeBranch?: string;
    }
  | {
      status: "async_launched";
      isAsync?: true;
      agentId: string;
      description: string;
      resolvedModel?: string;
      modelsUsed?: string[];
      prompt: string;
      outputFile: string;
      canReadOutputFile?: boolean;
    }
  | {
      status: "remote_launched";
      taskId: string;
      sessionUrl: string;
      description: string;
      prompt: string;
      outputFile: string;
    };
```

서브에이전트로부터 결과를 반환합니다. `status` 필드에 따라 구분됩니다: 완료된 작업의 경우 `"completed"`, 백그라운드 작업의 경우 `"async_launched"`, Claude Code가 원격 클라우드 세션으로 디스패치한 작업의 경우 `"remote_launched"` (여기서 `sessionUrl`은 해당 세션에 링크되고 `taskId`는 이를 식별함).

`completed` 및 `async_launched` 변형의 `resolvedModel` 필드는 서브에이전트가 실제로 실행된 모델의 이름을 지정하며, 이는 [`availableModels`](/docs/en/model-config#restrict-model-selection) 또는 다른 재정의가 적용될 때 요청된 `model` 입력과 다를 수 있습니다. 이 필드는 Claude Code v2.1.174 이상이 필요합니다. `async_launched`에서는 작업이 백그라운드로 이동했을 때 사용 중인 모델의 이름을 지정합니다.

`modelsUsed`는 서브에이전트가 사용한 모델을 순서대로 나열합니다. 이 필드는 실행 중간에 스왑이 발생했을 때만 존재하며, 다시 다시 스왑되었을 때 모델이 재등장합니다. `async_launched`에서 목록은 백그라운드 처리 전에 사용된 모델을 포함합니다. `modelsUsed` 및 `resolvedModel`의 백그라운드 처리 동작은 모두 Claude Code v2.1.212 이상이 필요합니다.

`completed` 변형에서 `worktreePath`는 서브에이전트가 격리된 git 워크트리에서 실행되었을 때 설정되며, `worktreeBranch`는 Claude Code가 생성했을 때 해당 워크트리의 브랜치 이름을 지정합니다. `usage.service_tier`는 서브에이전트 요청에 대해 API가 보고한 서비스 계층 문자열을 전달합니다.

v2.1.207 이전에는 게시된 타입이 더 좁았습니다. `worktreePath`, `worktreeBranch`, `citations`, `toolStats.frameCount`, 및 `inference_geo`, `speed`, `iterations` 사용량 필드를 생략했으며, `service_tier`를 `"standard" | "priority" | "batch"`로 타입 지정했습니다. 타입이 선택 사항으로 표시하는 필드는 이전 버전에서 기록된 결과에 없을 수 있습니다.

### AskUserQuestion

**도구 이름:** `AskUserQuestion`

```typescript theme={null}
type AskUserQuestionOutput = {
  questions: Array<{
    question: string;
    header: string;
    options: Array<{ label: string; description: string; preview?: string }>;
    multiSelect: boolean;
  }>;
  answers: Record<string, string>;
  response?: string;
};
```

질문 내용과 사용자의 답변을 반환합니다. 사용자가 구조화된 질문에 답하는 대신 자유 형식 응답을 입력한 경우 `response`가 설정됩니다. 설정된 경우 Claude는 질문별 답변 목록 대신 "The user responded: …"를 수신합니다.

### Bash

**도구 이름:** `Bash`

```typescript theme={null}
type BashOutput = {
  stdout: string;
  stderr: string;
  rawOutputPath?: string;
  interrupted: boolean;
  isImage?: boolean;
  backgroundTaskId?: string;
  backgroundedByUser?: boolean;
  timedOutAfterMs?: number;
  backgroundCwdHint?: string;
  dangerouslyDisableSandbox?: boolean;
  returnCodeInterpretation?: string;
  structuredContent?: unknown[];
  persistedOutputPath?: string;
  persistedOutputSize?: number;
};
```

stdout/stderr이 분리된 명령어 출력을 반환합니다. 백그라운드 명령어에는 `backgroundTaskId`가 포함됩니다.

`timedOutAfterMs`는 명시적으로 백그라운드에서 시작하는 대신 타임아웃에 도달하여 백그라운드로 이동했을 때의 밀리초 단위 타임아웃입니다. `backgroundCwdHint`는 백그라운드 처리된 명령어에 `cd`, `pushd`, `popd`, 또는 `chdir`과 같은 디렉터리 변경 내장 함수가 포함되어 있을 때 설정되며, 세션 작업 디렉터리가 변경되지 않았음을 기록합니다. 두 필드 모두 Claude Code v2.1.210 이상이 필요합니다.

### Monitor

**도구 이름:** `Monitor`

```typescript theme={null}
type MonitorOutput = {
  taskId: string;
  timeoutMs: number;
  persistent?: boolean;
};
```

실행 중인 모니터의 백그라운드 작업 ID를 반환합니다. 감시를 조기에 취소하려면 이 ID를 `TaskStop`과 함께 사용하세요.

### Edit

**도구 이름:** `Edit`

```typescript theme={null}
type FileEditOutput = {
  filePath: string;
  oldString: string;
  newString: string;
  originalFile: string;
  structuredPatch: Array<{
    oldStart: number;
    oldLines: number;
    newStart: number;
    newLines: number;
    lines: string[];
  }>;
  userModified: boolean;
  replaceAll: boolean;
  gitDiff?: {
    filename: string;
    status: "modified" | "added";
    additions: number;
    deletions: number;
    changes: number;
    patch: string;
  };
};
```

편집 작업의 구조화된 diff를 반환합니다.

### Read

**도구 이름:** `Read`

```typescript theme={null}
type FileReadOutput =
  | {
      type: "text";
      file: {
        filePath: string;
        content: string;
        numLines: number;
        startLine: number;
        totalLines: number;
      };
    }
  | {
      type: "image";
      file: {
        base64: string;
        type: "image/jpeg" | "image/png" | "image/gif" | "image/webp";
        originalSize: number;
        dimensions?: {
          originalWidth?: number;
          originalHeight?: number;
          displayWidth?: number;
          displayHeight?: number;
        };
      };
    }
  | {
      type: "notebook";
      file: {
        filePath: string;
        cells: unknown[];
      };
    }
  | {
      type: "pdf";
      file: {
        filePath: string;
        base64: string;
        originalSize: number;
      };
    }
  | {
      type: "parts";
      file: {
        filePath: string;
        originalSize: number;
        count: number;
        outputDir: string;
      };
    };
```

파일 타입에 적절한 형식으로 파일 내용을 반환합니다. `type` 필드에 따라 구별됩니다.

### Write

**도구 이름:** `Write`

```typescript theme={null}
type FileWriteOutput = {
  type: "create" | "update";
  filePath: string;
  content: string;
  structuredPatch: Array<{
    oldStart: number;
    oldLines: number;
    newStart: number;
    newLines: number;
    lines: string[];
  }>;
  originalFile: string | null;
  gitDiff?: {
    filename: string;
    status: "modified" | "added";
    additions: number;
    deletions: number;
    changes: number;
    patch: string;
  };
};
```

구조화된 diff 정보와 함께 쓰기 결과를 반환합니다.

### Glob

**도구 이름:** `Glob`

```typescript theme={null}
type GlobOutput = {
  durationMs: number;
  numFiles: number;
  filenames: string[];
  truncated: boolean;
  totalMatches?: number;
  countIsComplete?: boolean;
};
```

수정 시간순으로 정렬된 글로브 패턴과 일치하는 파일 경로를 반환합니다.

`totalMatches` 및 `countIsComplete`는 Claude Code v2.1.191 이상이 필요합니다. `totalMatches`는 잘리기 전 일치하는 파일 수를 보고합니다. `countIsComplete`가 false이면 기본 검색 자체가 출력을 잘랐기 때문에 `totalMatches`는 하한값입니다.

### Grep

**도구 이름:** `Grep`

```typescript theme={null}
type GrepOutput = {
  mode?: "content" | "files_with_matches" | "count";
  numFiles: number;
  filenames: string[];
  content?: string;
  numLines?: number;
  numMatches?: number;
  totalFiles?: number;
  totalLines?: number;
  appliedLimit?: number;
  appliedOffset?: number;
};
```

검색 결과를 반환합니다. 형상은 `mode`에 따라 다릅니다: 파일 목록, 일치하는 콘텐츠, 또는 일치 수. `count` 모드에서 `numFiles` 및 `numMatches`는 페이지가 지정된 조각이 아닌 전체 결과 세트에 대한 합계입니다. v2.1.208 이전에는 나열된 항목을 자른 `head_limit` 또는 `offset`이 이러한 합계도 잘랐습니다.

`totalFiles`는 Claude Code v2.1.208 이상이 필요하며 `files_with_matches` 모드에서 `head_limit` 및 `offset` 페이지네이션 전의 전체 결과 파일 수를 보고합니다. `totalLines`는 Claude Code v2.1.210 이상이 필요하며 `content` 모드에서 페이지네이션 전의 전체 줄 수를 보고합니다.

### TaskStop

**도구 이름:** `TaskStop`

```typescript theme={null}
type TaskStopOutput = {
  message: string;
  task_id: string;
  task_type: string;
  command?: string;
};
```

백그라운드 작업을 중지한 후 확인 메시지를 반환합니다.

### NotebookEdit

**도구 이름:** `NotebookEdit`

```typescript theme={null}
type NotebookEditOutput = {
  new_source: string;
  cell_id?: string;
  cell_type: "code" | "markdown";
  language: string;
  edit_mode: string;
  error?: string;
  notebook_path: string;
  original_file: string;
  updated_file: string;
};
```

원본 및 업데이트된 파일 내용과 함께 노트북 편집 결과를 반환합니다.

### WebFetch

**도구 이름:** `WebFetch`

```typescript theme={null}
type WebFetchOutput = {
  bytes: number;
  code: number;
  codeText: string;
  result: string;
  durationMs: number;
  url: string;
};
```

HTTP 상태 및 메타데이터와 함께 가져온 콘텐츠를 반환합니다.

### WebSearch

**도구 이름:** `WebSearch`

```typescript theme={null}
type WebSearchOutput = {
  query: string;
  results: Array<
    | {
        tool_use_id: string;
        content: Array<{ title: string; url: string }>;
      }
    | string
  >;
  durationSeconds: number;
};
```

웹에서 검색 결과를 반환합니다.

### Workflow

**도구 이름:** `Workflow`

```typescript theme={null}
type WorkflowOutput = {
  status: "async_launched";
  taskId: string;
  runId?: string;
  summary?: string;
  transcriptDir?: string;
  scriptPath?: string;
  error?: string;
};
```

도구가 호출을 수락한 후 즉시 반환합니다. 최종 결과는 나중에 작업 완료로 도착합니다. 실행이 시작된 것으로 처리하기 전에 `error`를 확인하세요: 구문 검사에 실패한 스크립트는 `error`가 설정된 `status: "async_launched"`를 반환하며 실행되지 않습니다.

| 필드            | 타입               | 설명                                                                                                                                                                                                   |
| --------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `status`        | `"async_launched"` | 도구가 호출을 수락했음. 이 필드가 취하는 유일한 값임                                                                                                                                                   |
| `taskId`        | `string`           | 실행을 위한 백그라운드 작업 식별자                                                                                                                                                                     |
| `runId`         | `string`           | 나중에 호출할 때 `resumeFromRunId`로 전달할 워크플로우 실행 식별자                                                                                                                                      |
| `summary`       | `string`           | 워크플로우가 수행하는 작업에 대한 한 줄 설명                                                                                                                                                           |
| `transcriptDir` | `string`           | 실행 중 서브에이전트 트랜스크립트가 기록되는 디렉터리                                                                                                                                                  |
| `scriptPath`    | `string`           | 이 실행에 지속되는 워크플로우 스크립트의 경로. 스크립트를 다시 보내지 않고 재실행하려면 이를 편집하고 `scriptPath`로 다시 전달하세요                                                                   |
| `error`         | `string`           | 스크립트 구문 검사 실패 시 설정됨. 존재하는 경우 `async_launched` 상태에도 불구하고 실행이 시작되지 않았음을 의미함                                                                                    |

### TodoWrite

**도구 이름:** `TodoWrite`

```typescript theme={null}
type TodoWriteOutput = {
  oldTodos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
  newTodos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
};
```

이전 및 업데이트된 작업 목록을 반환합니다.

<Note>
  TypeScript Agent SDK 0.3.142부터 `TodoWrite`는 기본적으로 비활성화되어 있습니다. 대신 `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList`를 사용하세요. 모니터링 코드를 업데이트하려면 [Task 도구로 마이그레이션](/docs/en/agent-sdk/todo-tracking#migrate-to-task-tools)을 참조하고, `TodoWrite`로 되돌리려면 `CLAUDE_CODE_ENABLE_TASKS=0`을 설정하세요.
</Note>

### TaskCreate

**도구 이름:** `TaskCreate`

```typescript theme={null}
type TaskCreateOutput = {
  task: {
    id: string;
    subject: string;
  };
};
```

할당된 ID와 함께 생성된 작업을 반환합니다.

### TaskUpdate

**도구 이름:** `TaskUpdate`

```typescript theme={null}
type TaskUpdateOutput = {
  success: boolean;
  taskId: string;
  updatedFields: string[];
  error?: string;
  statusChange?: {
    from: string;
    to: string;
  };
};
```

어떤 필드가 변경되었는지를 포함하여 업데이트 결과를 반환합니다.

### TaskGet

**도구 이름:** `TaskGet`

```typescript theme={null}
type TaskGetOutput = {
  task: {
    id: string;
    subject: string;
    description: string;
    status: "pending" | "in_progress" | "completed";
    blocks: string[];
    blockedBy: string[];
  } | null;
};
```

전체 작업 레코드를 반환하며, ID를 찾을 수 없는 경우 `null`을 반환합니다.

### TaskList

**도구 이름:** `TaskList`

```typescript theme={null}
type TaskListOutput = {
  tasks: Array<{
    id: string;
    subject: string;
    status: "pending" | "in_progress" | "completed";
    owner?: string;
    blockedBy: string[];
  }>;
};
```

현재 목록의 모든 작업에 대한 스냅샷을 반환합니다.

### ExitPlanMode

**도구 이름:** `ExitPlanMode`

```typescript theme={null}
type ExitPlanModeOutput = {
  plan: string | null;
  isAgent: boolean;
  filePath?: string;
  hasTaskTool?: boolean;
  awaitingLeaderApproval?: boolean;
  requestId?: string;
};
```

계획 모드를 종료한 후의 계획 상태를 반환합니다.

### ListMcpResources

**도구 이름:** `ListMcpResourcesTool`

```typescript theme={null}
type ListMcpResourcesOutput = Array<{
  uri: string;
  name: string;
  mimeType?: string;
  description?: string;
  server: string;
}>;
```

사용 가능한 MCP 리소스의 배열을 반환합니다.

### ReadMcpResource

**도구 이름:** `ReadMcpResourceTool`

```typescript theme={null}
type ReadMcpResourceOutput = {
  contents: Array<{
    uri: string;
    mimeType?: string;
    text?: string;
  }>;
};
```

요청된 MCP 리소스의 내용을 반환합니다.

### EnterWorktree

**도구 이름:** `EnterWorktree`

```typescript theme={null}
type EnterWorktreeOutput = {
  worktreePath: string;
  worktreeBranch?: string;
  message: string;
};
```

git 워크트리에 대한 정보를 반환합니다.

## 권한 타입 (Permission Types)

### `PermissionUpdate`

권한 업데이트를 위한 작업들입니다.

```typescript theme={null}
type PermissionUpdate =
  | {
      type: "addRules";
      rules: PermissionRuleValue[];
      behavior: PermissionBehavior;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "replaceRules";
      rules: PermissionRuleValue[];
      behavior: PermissionBehavior;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "removeRules";
      rules: PermissionRuleValue[];
      behavior: PermissionBehavior;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "setMode";
      mode: PermissionMode;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "addDirectories";
      directories: string[];
      destination: PermissionUpdateDestination;
    }
  | {
      type: "removeDirectories";
      directories: string[];
      destination: PermissionUpdateDestination;
    };
```

### `PermissionBehavior`

```typescript theme={null}
type PermissionBehavior = "allow" | "deny" | "ask";
```

### `PermissionUpdateDestination`

```typescript theme={null}
type PermissionUpdateDestination =
  | "userSettings" // 전역 사용자 설정
  | "projectSettings" // 디렉터리별 프로젝트 설정
  | "localSettings" // 로컬 프로젝트 설정
  | "session" // 현재 세션 전용
  | "cliArg"; // CLI 인자
```

### `PermissionRuleValue`

```typescript theme={null}
type PermissionRuleValue = {
  toolName: string;
  ruleContent?: string;
};
```

## 기타 타입 (Other Types)

### `ApiKeySource`

```typescript theme={null}
type ApiKeySource = "user" | "project" | "org" | "temporary" | "oauth";
```

### `SdkBeta`

`betas` 옵션을 통해 활성화할 수 있는 사용 가능한 베타 기능입니다. 자세한 내용은 [베타 헤더](https://platform.claude.com/docs/en/api/beta-headers)를 참조하세요.

```typescript theme={null}
type SdkBeta = "context-1m-2025-08-07";
```

<Warning>
  `context-1m-2025-08-07` 베타는 2026년 4월 30일부로 사용중단되었습니다. Claude Sonnet 4.5 또는 Sonnet 4에서 이 값을 전달해도 효과가 없으며, 표준 200k 토큰 컨텍스트 창을 초과하는 요청은 에러를 반환합니다. 1M 토큰 컨텍스트 창을 사용하려면 베타 헤더 없이 표준 가격으로 1M 컨텍스트가 포함된 [Claude Sonnet 5, Claude Sonnet 4.6, Claude Opus 4.6, Claude Opus 4.7 또는 Claude Opus 4.8](https://platform.claude.com/docs/en/about-claude/models/overview)로 마이그레이션하세요.
</Warning>

### `SlashCommand`

사용 가능한 슬래시 명령어에 대한 정보입니다.

```typescript theme={null}
type SlashCommand = {
  name: string;
  description: string;
  argumentHint: string;
  aliases?: string[];
};
```

### `ModelInfo`

사용 가능한 모델에 대한 정보입니다.

```typescript theme={null}
type ModelInfo = {
  value: string;
  resolvedModel?: string;
  displayName: string;
  description: string;
  supportsEffort?: boolean;
  supportedEffortLevels?: ("low" | "medium" | "high" | "xhigh" | "max")[];
  supportsAdaptiveThinking?: boolean;
  supportsFastMode?: boolean;
  supportsAutoMode?: boolean;
};
```

| 필드                       | 타입                                                               | 설명                                                                                                                                                                                                                                                                                                           |
| :------------------------- | :----------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `value`                    | `string`                                                           | API 호출 시 전달할 모델 식별자                                                                                                                                                                                                                                                                                        |
| `resolvedModel`            | `string \| undefined`                                              | 이 항목의 `value`가 확인되는 정식 유선 모델 ID. `sonnet`과 같은 에일리어스 항목은 `claude-sonnet-5`와 같은 명시적 모델 ID로 확인되므로 호스트는 저장된 명시적 모델 ID를 이를 다루는 에일리어스 항목과 매칭할 수 있습니다. Claude Code v2.1.197 이상 필요                                                              |
| `displayName`              | `string`                                                           | 사람이 읽을 수 있는 표시 이름                                                                                                                                                                                                                                                                                         |
| `description`              | `string`                                                           | 모델의 기능에 대한 설명                                                                                                                                                                                                                                                                                               |
| `supportsEffort`           | `boolean \| undefined`                                             | 이 모델이 노력 수준(effort levels)을 지원하는지 여부                                                                                                                                                                                                                                                                  |
| `supportedEffortLevels`    | `("low" \| "medium" \| "high" \| "xhigh" \| "max")[] \| undefined` | 이 모델이 수용하는 노력 수준                                                                                                                                                                                                                                                                                          |
| `supportsAdaptiveThinking` | `boolean \| undefined`                                             | 이 모델이 생각할 시기와 생각할 양을 Claude가 결정하는 적응형 추론(adaptive thinking)을 지원하는지 여부                                                                                                                                                                                                               |
| `supportsFastMode`         | `boolean \| undefined`                                             | 이 모델이 빠른 모드(fast mode)를 지원하는지 여부                                                                                                                                                                                                                                                                      |
| `supportsAutoMode`         | `boolean \| undefined`                                             | 이 모델이 자동 모드(auto mode)를 지원하는지 여부                                                                                                                                                                                                                                                                      |

### `AgentInfo`

Agent 도구를 통해 호출할 수 있는 사용 가능한 서브에이전트에 대한 정보입니다.

```typescript theme={null}
type AgentInfo = {
  name: string;
  description: string;
  model?: string;
};
```

| 필드          | 타입                  | 설명                                                                 |
| :------------ | :-------------------- | :------------------------------------------------------------------- |
| `name`        | `string`              | 에이전트 타입 식별자(예: `"Explore"`, `"general-purpose"`)           |
| `description` | `string`              | 이 에이전트를 사용할 시기에 대한 설명                                |
| `model`       | `string \| undefined` | 이 에이전트가 사용하는 모델 에일리어스. 생략 시 부모의 모델을 상속함 |

### `McpServerStatus`

연결된 MCP 서버의 상태입니다.

```typescript theme={null}
type McpServerStatus = {
  name: string;
  status: "connected" | "failed" | "needs-auth" | "pending" | "disabled";
  serverInfo?: {
    name: string;
    version: string;
  };
  error?: string;
  config?: McpServerStatusConfig;
  scope?: string;
  tools?: {
    name: string;
    description?: string;
    annotations?: {
      readOnly?: boolean;
      destructive?: boolean;
      openWorld?: boolean;
    };
  }[];
};
```

### `McpServerStatusConfig`

`mcpServerStatus()`에 의해 보고된 MCP 서버의 구성입니다. 이는 모든 MCP 서버 전송 타입의 유니온입니다.

```typescript theme={null}
type McpServerStatusConfig =
  | McpStdioServerConfig
  | McpSSEServerConfig
  | McpHttpServerConfig
  | McpSdkServerConfig
  | McpClaudeAIProxyServerConfig;
```

각 전송 타입의 세부 정보는 [`McpServerConfig`](#mcpserverconfig)를 참조하세요.

### `AccountInfo`

인증된 사용자의 계정 정보입니다.

```typescript theme={null}
type AccountInfo = {
  email?: string;
  organization?: string;
  subscriptionType?: string;
  tokenSource?: string;
  apiKeySource?: string;
};
```

### `ModelUsage`

결과 메시지에 반환되는 모델별 사용량 통계입니다. `costUSD` 값은 클라이언트 측 추정치입니다. 청구 주의 사항은 [비용 및 사용량 추적](/docs/en/agent-sdk/cost-tracking)을 참조하세요.

```typescript theme={null}
type ModelUsage = {
  inputTokens: number;
  outputTokens: number;
  cacheReadInputTokens: number;
  cacheCreationInputTokens: number;
  webSearchRequests: number;
  costUSD: number;
  contextWindow: number;
  maxOutputTokens: number;
};
```

### `ConfigScope`

```typescript theme={null}
type ConfigScope = "local" | "user" | "project";
```

### `NonNullableUsage`

모든 널 허용 필드가 널 비허용으로 지정된 [`Usage`](#usage) 버전입니다.

```typescript theme={null}
type NonNullableUsage = {
  [K in keyof Usage]: NonNullable<Usage[K]>;
};
```

### `Usage`

토큰 사용량 통계입니다. 이는 `@anthropic-ai/sdk`의 `BetaUsage` 타입입니다.

```typescript theme={null}
type Usage = {
  input_tokens: number;
  output_tokens: number;
  cache_creation_input_tokens: number | null;
  cache_read_input_tokens: number | null;
  cache_creation: {
    ephemeral_5m_input_tokens: number;
    ephemeral_1h_input_tokens: number;
  } | null;
  server_tool_use: BetaServerToolUsage | null;
  service_tier: "standard" | "priority" | "batch" | null;
  speed: "standard" | "fast" | null;
  inference_geo: string | null;
  iterations: BetaIterationsUsage | null;
};
```

`BetaServerToolUsage` 및 `BetaIterationsUsage`는 `@anthropic-ai/sdk`에 정의되어 있습니다.

### `CallToolResult`

MCP 도구 결과 타입 (`@modelcontextprotocol/sdk/types.js` 출처)입니다. `structuredContent`는 이미지 블록을 포함하여 `content`와 함께 반환할 수 있는 JSON 객체입니다. [구조화된 데이터 반환](/docs/en/agent-sdk/custom-tools#return-structured-data)을 참조하세요.

```typescript theme={null}
type CallToolResult = {
  content: Array<{
    type: "text" | "image" | "audio" | "resource" | "resource_link";
    // 추가 필드는 타입에 따라 다름
  }>;
  structuredContent?: Record<string, unknown>;
  isError?: boolean;
};
```

### `ThinkingConfig`

Claude의 생각하기/추론 동작을 제어합니다. 권장되지 않는 `maxThinkingTokens`보다 우선합니다.

```typescript theme={null}
type ThinkingDisplay = "summarized" | "omitted";

type ThinkingConfig =
  | { type: "adaptive"; display?: ThinkingDisplay } // 모델이 추론 시기와 양을 결정함 (Opus 4.6+)
  | { type: "enabled"; budgetTokens?: number; display?: ThinkingDisplay } // 고정된 생각하기 토큰 예산
  | { type: "disabled" }; // 확장 추론 사용 안 함
```

선택적 `display` 필드는 생각하기 텍스트가 `"summarized"` 또는 `"omitted"`로 반환되는지 여부를 제어합니다. Claude Opus 4.7 이상에서 API 기본값은 `"omitted"`이므로 `thinking` 블록에서 생각하기 내용을 받으려면 `"summarized"`로 설정하세요.

### `SpawnedProcess`

커스텀 프로세스 생성(spawning)을 위한 인터페이스 (`spawnClaudeCodeProcess` 옵션과 함께 사용됨)입니다. `ChildProcess`는 이미 이 인터페이스를 충족합니다.

```typescript theme={null}
interface SpawnedProcess {
  stdin: Writable;
  stdout: Readable;
  readonly killed: boolean;
  readonly exitCode: number | null;
  kill(signal: NodeJS.Signals): boolean;
  on(
    event: "exit",
    listener: (code: number | null, signal: NodeJS.Signals | null) => void
  ): void;
  on(event: "error", listener: (error: Error) => void): void;
  once(
    event: "exit",
    listener: (code: number | null, signal: NodeJS.Signals | null) => void
  ): void;
  once(event: "error", listener: (error: Error) => void): void;
  off(
    event: "exit",
    listener: (code: number | null, signal: NodeJS.Signals | null) => void
  ): void;
  off(event: "error", listener: (error: Error) => void): void;
}
```

### `SpawnOptions`

커스텀 생성 함수에 전달되는 옵션입니다.

```typescript theme={null}
interface SpawnOptions {
  command: string;
  args: string[];
  cwd?: string;
  env: Record<string, string | undefined>;
  signal: AbortSignal;
}
```

<Note>
  `signal` 필드는 커스텀 생성 함수에 프로세스를 해제할 시기를 알립니다. 이를 Node의 `spawn()`에 `signal` 옵션으로 전달하거나 VM 또는 컨테이너 해제 핸들러에 전달하세요.

  이 신호는 [`Options.abortController`](#options)가 중단되는 즉시 실행되지는 않습니다. SDK는 먼저 프로세스의 stdin을 닫고 CLI가 깔끔하게 종료될 수 있도록 약 2초를 기다린 후 이 신호를 중단합니다. 호출자가 중단하는 즉시 반응하려면 커스텀 생성 함수가 둘러싸는 스코프에서 참조할 수 있는 자체 `Options.abortController.signal`을 관찰하세요.
</Note>

### `McpSetServersResult`

`setMcpServers()` 작업의 결과입니다.

```typescript theme={null}
type McpSetServersResult = {
  added: string[];
  removed: string[];
  errors: Record<string, string>;
};
```

### `RewindFilesResult`

`rewindFiles()` 작업의 결과입니다.

```typescript theme={null}
type RewindFilesResult = {
  canRewind: boolean;
  error?: string;
  filesChanged?: string[];
  insertions?: number;
  deletions?: number;
};
```

### `SDKStatusMessage`

상태 업데이트 메시지 (예: 압축 진행 중)입니다.

```typescript theme={null}
type SDKStatusMessage = {
  type: "system";
  subtype: "status";
  status: "compacting" | null;
  permissionMode?: PermissionMode;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskNotificationMessage`

백그라운드 작업이 완료되거나 실패하거나 중지되었을 때의 알림입니다. 백그라운드 작업에는 `run_in_background` Bash 명령어, [Monitor](#monitor) 감시 및 백그라운드 서브에이전트가 포함됩니다.

```typescript theme={null}
type SDKTaskNotificationMessage = {
  type: "system";
  subtype: "task_notification";
  task_id: string;
  tool_use_id?: string;
  status: "completed" | "failed" | "stopped";
  output_file: string;
  summary: string;
  usage?: {
    total_tokens: number;
    tool_uses: number;
    duration_ms: number;
  };
  uuid: UUID;
  session_id: string;
};
```

### `SDKToolUseSummaryMessage`

대화 내 도구 사용량의 요약입니다.

```typescript theme={null}
type SDKToolUseSummaryMessage = {
  type: "tool_use_summary";
  summary: string;
  preceding_tool_use_ids: string[];
  uuid: UUID;
  session_id: string;
};
```

### `SDKHookStartedMessage`

훅이 실행을 시작할 때 출력됩니다.

Claude Code는 이 메시지, [`SDKHookProgressMessage`](#sdkhookprogressmessage) 및 [`SDKHookResponseMessage`](#sdkhookresponsemessage)를 세션 시작 시 `SessionStart` 또는 `Setup` 훅이 여전히 실행 중일 때를 포함하여 메시지 스트림으로 즉시 전달합니다. Claude Code v2.1.169부터 v2.1.203까지는 `SessionStart` 또는 `Setup` 훅이 완료된 후 하나의 배치로 이 메시지들을 전달했습니다; v2.1.204에서 라이브 전달이 복원되었습니다.

```typescript theme={null}
type SDKHookStartedMessage = {
  type: "system";
  subtype: "hook_started";
  hook_id: string;
  hook_name: string;
  hook_event: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKHookProgressMessage`

stdout/stderr 출력과 함께 훅이 실행되는 동안 출력됩니다.

```typescript theme={null}
type SDKHookProgressMessage = {
  type: "system";
  subtype: "hook_progress";
  hook_id: string;
  hook_name: string;
  hook_event: string;
  stdout: string;
  stderr: string;
  output: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKHookResponseMessage`

훅이 실행을 마쳤을 때 출력됩니다.

```typescript theme={null}
type SDKHookResponseMessage = {
  type: "system";
  subtype: "hook_response";
  hook_id: string;
  hook_name: string;
  hook_event: string;
  output: string;
  stdout: string;
  stderr: string;
  exit_code?: number;
  outcome: "success" | "error" | "cancelled";
  uuid: UUID;
  session_id: string;
};
```

### `SDKToolProgressMessage`

진행 상황을 나타내기 위해 도구가 실행되는 동안 주기적으로 출력됩니다.

```typescript theme={null}
type SDKToolProgressMessage = {
  type: "tool_progress";
  tool_use_id: string;
  tool_name: string;
  parent_tool_use_id: string | null;
  elapsed_time_seconds: number;
  task_id?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKAuthStatusMessage`

인증 흐름 중에 출력됩니다.

```typescript theme={null}
type SDKAuthStatusMessage = {
  type: "auth_status";
  isAuthenticating: boolean;
  output: string[];
  error?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskStartedMessage`

백그라운드 작업이 시작될 때 출력됩니다. `task_type` 필드는 백그라운드 Bash 명령어 및 [Monitor](#monitor) 감시의 경우 `"local_bash"`, 서브에이전트의 경우 `"local_agent"`, 또는 `"remote_agent"`입니다.

```typescript theme={null}
type SDKTaskStartedMessage = {
  type: "system";
  subtype: "task_started";
  task_id: string;
  tool_use_id?: string;
  description: string;
  task_type?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskProgressMessage`

서브에이전트나 백그라운드 작업이 실행 중일 때 주기적으로 출력됩니다. `summary` 필드는 [`agentProgressSummaries`](#options)가 활성화되어 있을 때만 채워집니다.

```typescript theme={null}
type SDKTaskProgressMessage = {
  type: "system";
  subtype: "task_progress";
  task_id: string;
  tool_use_id?: string;
  description: string;
  subagent_type?: string;
  usage: {
    total_tokens: number;
    tool_uses: number;
    duration_ms: number;
  };
  last_tool_name?: string;
  summary?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskUpdatedMessage`

백그라운드 작업의 상태가 변경될 때(예: `running`에서 `completed`로 전환될 때) 출력됩니다. `task_id`를 키로 하는 로컬 작업 맵에 `patch`를 병합하세요. `end_time` 필드는 `Date.now()`와 비교 가능한 밀리초 단위의 Unix 에포크 타임스탬프입니다.

```typescript theme={null}
type SDKTaskUpdatedMessage = {
  type: "system";
  subtype: "task_updated";
  task_id: string;
  patch: {
    status?: "pending" | "running" | "completed" | "failed" | "killed";
    description?: string;
    end_time?: number;
    total_paused_ms?: number;
    error?: string;
    is_backgrounded?: boolean;
  };
  uuid: UUID;
  session_id: string;
};
```

### `SDKBackgroundTasksChangedMessage`

라이브 백그라운드 작업 세트가 변경될 때마다 출력됩니다(작업 시작, 완료, 종료, 또는 포그라운드 에이전트의 백그라운드 처리). `tasks` 배열은 전체 라이브 세트입니다. `task_started` 및 `task_notification` 이벤트를 쌍으로 구성하는 대신 매 페이로드마다 캐시된 세트를 교체하여 다음 멤버십 변경이 놓친 이벤트를 수정하도록 하세요.

이러한 작업별 이벤트와 관련된 순서는 지정되지 않으므로 두 스트림을 관련시키지 마세요.

시작 시에는 아무것도 출력되지 않습니다. 세션의 CLI 프로세스가 시작되거나 다시 시작될 때마다 빈 세트로 재설정하고 다음 멤버십 변경이 이를 다시 로드하도록 하세요.

Claude Code v2.1.203 이상이 필요합니다.

```typescript theme={null}
type SDKBackgroundTasksChangedMessage = {
  type: "system";
  subtype: "background_tasks_changed";
  tasks: {
    task_id: string;
    task_type: string;
    description: string;
  }[];
  uuid: UUID;
  session_id: string;
};
```

### `SDKThinkingTokensMessage`

Claude가 지금까지 생성된 생각하기 토큰의 실행 추정치를 전달하는 가려진(redacted) 블록을 포함하여 생각하기 블록을 생성하는 동안 출력됩니다. `estimated_tokens`는 현재 생각하기 블록의 누적 합계이고 `estimated_tokens_delta`는 이 프레임이 전달하는 증분입니다. 진행 상황 표시에 사용하세요. 최상위 에이전트 루프의 최종 개수는 [서브에이전트 토큰을 포함하지 않는](/docs/en/agent-sdk/cost-tracking#get-the-total-cost-of-a-query) 결과 메시지의 `usage.output_tokens`입니다; 전체 트리 계산을 위해 [`modelUsage`](#modelusage)를 사용하세요.

Claude Code v2.1.153 이상이 필요합니다.

```typescript theme={null}
type SDKThinkingTokensMessage = {
  type: "system";
  subtype: "thinking_tokens";
  estimated_tokens: number;
  estimated_tokens_delta: number;
  uuid: UUID;
  session_id: string;
};
```

### `SDKFilesPersistedEvent`

파일 체크포인트가 디스크에 지속될 때 출력됩니다.

```typescript theme={null}
type SDKFilesPersistedEvent = {
  type: "system";
  subtype: "files_persisted";
  files: { filename: string; file_id: string }[];
  failed: { filename: string; error: string }[];
  processed_at: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKRateLimitEvent`

세션이 속도 제한에 부딪혔을 때 출력됩니다.

```typescript theme={null}
type SDKRateLimitEvent = {
  type: "rate_limit_event";
  rate_limit_info: {
    status: "allowed" | "allowed_warning" | "rejected";
    resetsAt?: number;
    utilization?: number;
    errorCode?: "credits_required";
    canUserPurchaseCredits?: boolean;
    hasChargeableSavedPaymentMethod?: boolean;
  };
  uuid: UUID;
  session_id: string;
};
```

`errorCode`가 `"credits_required"`인 경우 거부는 사용량이 소진된 claude.ai 구독에 의한 것이며, 사용자가 사용 크레딧을 구매할 때까지 세션을 계속할 수 없습니다. `canUserPurchaseCredits`는 인증된 사용자가 계정의 크레딧을 구매할 수 있는지 여부를 나타내며, `hasChargeableSavedPaymentMethod`는 청구 가능한 결제 수단이 등록되어 있는지 여부를 나타냅니다. 세 필드 모두 크레딧 필요 거부가 아닌 속도 제한 이벤트에는 존재하지 않습니다. Claude Code v2.1.181 이상이 필요합니다.

### `SDKLocalCommandOutputMessage`

로컬 슬래시 명령어(예: `/voice` 또는 `/usage`)의 출력입니다. 트랜스크립트에서 어시스턴트 스타일의 텍스트로 표시됩니다.

```typescript theme={null}
type SDKLocalCommandOutputMessage = {
  type: "system";
  subtype: "local_command_output";
  content: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKCommandsChangedMessage`

에이전트가 하위 디렉터리에 진입할 때 스킬이 발견되는 것과 같이 세션 중간에 사용 가능한 명령어 세트가 변경될 때 출력됩니다. `commands` 배열은 업데이트된 전체 목록이므로 캐시된 명령어 목록을 이 페이로드로 교체하세요. `supportedCommands()`를 다시 호출하는 것은 동일하지 않습니다: 해당 메서드는 초기화 시 캡처된 스냅샷을 반환하며 세션 중간 변경 사항을 반영하지 않습니다.

```typescript theme={null}
type SDKCommandsChangedMessage = {
  type: "system";
  subtype: "commands_changed";
  commands: SlashCommand[];
  uuid: UUID;
  session_id: string;
};
```

### `SDKPromptSuggestionMessage`

`promptSuggestions`가 활성화되어 있을 때 각 차례 후에 출력됩니다. 예측된 다음 사용자 프롬프트를 포함합니다.

```typescript theme={null}
type SDKPromptSuggestionMessage = {
  type: "prompt_suggestion";
  suggestion: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKConversationResetMessage`

`/clear` 후, 계획 모드 종료 시, 또는 새 대화가 시작될 때와 같이 세션을 종료하지 않고 세션의 대화가 교체될 때 출력됩니다. `new_conversation_id` 아래에 빈 트랜스크립트를 마운트하고 캐시된 세션 제목을 삭제하세요.

```typescript theme={null}
type SDKConversationResetMessage = {
  type: "conversation_reset";
  new_conversation_id: UUID;
  uuid: UUID;
  session_id: string;
};
```

SDK의 게시된 타이핑은 Claude Code v2.1.203 이상에서 `SDKConversationResetMessage`를 선언합니다. v2.1.203 이전에는 `SDKMessage`가 이를 선언하지 않고 타입을 참조했기 때문에 `skipLibCheck`가 비활성화된 경우 `type === "conversation_reset"`에 대한 서브타이핑 검사가 실패했습니다.

### `AbortError`

중단 작업을 위한 커스텀 오류 클래스입니다.

```typescript theme={null}
class AbortError extends Error {}
```

## 샌드박스 구성 (Sandbox Configuration)

### `SandboxSettings`

샌드박스 동작을 위한 구성입니다. 이를 사용하여 명령어 샌드박싱을 활성화하고 프로그래밍 방식으로 네트워크 제한을 구성할 수 있습니다.

```typescript theme={null}
type SandboxSettings = {
  enabled?: boolean;
  failIfUnavailable?: boolean;
  autoAllowBashIfSandboxed?: boolean;
  excludedCommands?: string[];
  allowUnsandboxedCommands?: boolean;
  network?: SandboxNetworkConfig;
  filesystem?: SandboxFilesystemConfig;
  ignoreViolations?: Record<string, string[]>;
  enableWeakerNestedSandbox?: boolean;
  ripgrep?: { command: string; args?: string[] };
};
```

| 속성                        | 타입                                                  | 기본값      | 설명                                                                                                                                                                                                                                                                       |
| :-------------------------- | :---------------------------------------------------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled`                   | `boolean`                                             | `false`     | 명령어 실행을 위한 샌드박스 모드 활성화                                                                                                                                                                                                                                    |
| `failIfUnavailable`         | `boolean`                                             | `true`      | `enabled`가 `true`이지만 샌드박스를 시작할 수 없는 경우 시작 시 중지함. stderr에 경고와 함께 샌드박스 없이 실행되도록 폴백하려면 `false`로 설정                                                                                                                            |
| `autoAllowBashIfSandboxed`  | `boolean`                                             | `true`      | 샌드박스가 활성화되어 있을 때 bash 명령어 자동 승인                                                                                                                                                                                                                        |
| `excludedCommands`          | `string[]`                                            | `[]`        | 항상 샌드박스 제한을 우회하는 명령어 (예: `['docker']`). 모델 참여 없이 자동으로 샌드박스 없이 실행됨                                                                                                                                                                     |
| `allowUnsandboxedCommands`  | `boolean`                                             | `true`      | 모델이 샌드박스 외부에서 명령어를 실행하도록 요청하는 것을 허용함. `true`인 경우 모델은 도구 입력에서 `dangerouslyDisableSandbox`를 설정할 수 있으며, 이는 [권한 시스템](#permissions-fallback-for-unsandboxed-commands)으로 폴백함                                          |
| `network`                   | [`SandboxNetworkConfig`](#sandboxnetworkconfig)       | `undefined` | 네트워크 전용 샌드박스 구성                                                                                                                                                                                                                                               |
| `filesystem`                | [`SandboxFilesystemConfig`](#sandboxfilesystemconfig) | `undefined` | 읽기/쓰기 제한을 위한 파일 시스템 전용 샌드박스 구성                                                                                                                                                                                                                       |
| `ignoreViolations`          | `Record<string, string[]>`                            | `undefined` | 무시할 위반 카테고리의 맵 (예: `{ file: ['/tmp/*'], network: ['localhost'] }`)                                                                                                                                                                                             |
| `enableWeakerNestedSandbox` | `boolean`                                             | `false`     | 호환성을 위해 더 약한 중첩 샌드박스 활성화                                                                                                                                                                                                                                 |
| `ripgrep`                   | `{ command: string; args?: string[] }`                | `undefined` | 샌드박스 환경을 위한 커스텀 ripgrep 바이너리 구성                                                                                                                                                                                                                          |

<Note>
  샌드박스는 플랫폼 지원 및 Linux의 경우 `bubblewrap` 및 `socat`과 같은 도구에 의존합니다. `enabled`가 `true`이고 샌드박스를 시작할 수 없는 경우 `query()`는 `subtype: "error_during_execution"` 및 `errors`에 이유가 포함된 `result` 메시지를 보고합니다. 단일 메시지 `query()` 호출의 경우 SDK는 해당 에러 결과를 반환한 후 에러를 발생시키므로 루프를 try 블록으로 감싸 이를 지나쳐 진행하세요. 에러 계약은 [결과 처리](/docs/en/agent-sdk/agent-loop#handle-the-result)를 참조하세요.

  대신 샌드박스 없이 실행하려면 `failIfUnavailable: false`를 설정하세요.
</Note>

#### 사용 예시

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({
    prompt: "Build and test my project",
    options: {
      sandbox: {
        enabled: true,
        autoAllowBashIfSandboxed: true,
        network: {
          allowLocalBinding: true
        }
      }
    }
  })) {
    if ("result" in message) console.log(message.result);
  }
} catch (error) {
  // 단일 실행 query()는 샌드박스를 시작할 수 없을 때와 같이
  // 오류 결과를 반환한 후 에러를 발생시킵니다 (failIfUnavailable 기본값 true).
  console.log(`Session ended with an error: ${error}`);
}
```

<Warning>
  **Unix 소켓 보안:** `allowUnixSockets` 옵션은 강력한 시스템 서비스에 대한 접근을 부여할 수 있습니다. 예를 들어 `/var/run/docker.sock`을 허용하면 Docker API를 통해 호스트 시스템에 대한 완전한 접근 권한이 부여되어 샌드박스 격리가 우회됩니다. 엄격히 필요한 Unix 소켓만 허용하고 각 소켓의 보안 영향을 이해하세요.
</Warning>

### `SandboxNetworkConfig`

샌드박스 모드를 위한 네트워크 전용 구성입니다. 이 설정은 부모 [`SandboxSettings`](#sandboxsettings)에서 `enabled`가 `true`일 때 샌드박스 처리된 Bash 명령어에 적용됩니다. 대신 [권한 규칙](/docs/en/permissions#webfetch)을 사용하는 WebFetch 도구는 제한하지 않습니다.

```typescript theme={null}
type SandboxNetworkConfig = {
  allowedDomains?: string[];
  deniedDomains?: string[];
  allowManagedDomainsOnly?: boolean;
  allowLocalBinding?: boolean;
  allowUnixSockets?: string[];
  allowAllUnixSockets?: boolean;
  httpProxyPort?: number;
  socksProxyPort?: number;
};
```

| 속성                      | 타입       | 기본값      | 설명                                                                                                                                                                                                                                        |
| :------------------------ | :--------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `allowedDomains`          | `string[]` | `[]`        | 샌드박스 처리된 프로세스가 접근할 수 있는 도메인 이름                                                                                                                                                                                       |
| `deniedDomains`           | `string[]` | `[]`        | 샌드박스 처리된 프로세스가 접근할 수 없는 도메인 이름. `allowedDomains`보다 우선함                                                                                                                                                          |
| `allowManagedDomainsOnly` | `boolean`  | `false`     | 관리 설정 전용. [관리 설정](/docs/en/permissions#managed-settings)에서 설정된 경우 관리 설정의 `allowedDomains` 항목만 존중되고 사용자, 프로젝트 또는 로컬 설정의 항목은 무시됨. SDK 옵션을 통해 설정된 경우 효과 없음                  |
| `allowLocalBinding`       | `boolean`  | `false`     | 프로세스가 로컬 포트에 바인딩하는 것을 허용함 (예: 개발 서버용)                                                                                                                                                                             |
| `allowUnixSockets`        | `string[]` | `[]`        | 프로세스가 접근할 수 있는 Unix 소켓 경로 (예: Docker 소켓)                                                                                                                                                                                  |
| `allowAllUnixSockets`     | `boolean`  | `false`     | 모든 Unix 소켓에 대한 접근 허용                                                                                                                                                                                                             |
| `httpProxyPort`           | `number`   | `undefined` | 네트워크 요청을 위한 HTTP 프록시 포트                                                                                                                                                                                                       |
| `socksProxyPort`          | `number`   | `undefined` | 네트워크 요청을 위한 SOCKS 프록시 포트                                                                                                                                                                                                      |

<Note>
  내장 샌드박스 프록시는 요청된 호스트 이름을 기반으로 `allowedDomains`를 적용하며 TLS 트래픽을 종료하거나 검사하지 않으므로 [도메인 프론팅](https://en.wikipedia.org/wiki/Domain_fronting)과 같은 기술이 잠재적으로 이를 우회할 수 있습니다. 세부 정보는 [샌드박싱 보안 제한 사항](/docs/en/sandboxing#security-limitations)을 참조하고 TLS 종료 프록시 구성에 대해서는 [보안 배포](/docs/en/agent-sdk/secure-deployment#traffic-forwarding)를 참조하세요.
</Note>

### `SandboxFilesystemConfig`

샌드박스 모드를 위한 파일 시스템 전용 구성입니다.

```typescript theme={null}
type SandboxFilesystemConfig = {
  allowWrite?: string[];
  denyWrite?: string[];
  denyRead?: string[];
};
```

| 속성         | 타입       | 기본값 | 설명                                         |
| :----------- | :--------- | :----- | :------------------------------------------- |
| `allowWrite` | `string[]` | `[]`    | 쓰기 접근을 허용할 파일 경로 패턴            |
| `denyWrite`  | `string[]` | `[]`    | 쓰기 접근을 거부할 파일 경로 패턴            |
| `denyRead`   | `string[]` | `[]`    | 읽기 접근을 거부할 파일 경로 패턴            |

### 샌드박스 처리되지 않은 명령어를 위한 권한 폴백

`allowUnsandboxedCommands`가 활성화되면 모델은 도구 입력에서 `dangerouslyDisableSandbox: true`를 설정하여 샌드박스 외부에서 명령어를 실행하도록 요청할 수 있습니다. 이러한 요청은 기존 권한 시스템으로 폴백하여 `canUseTool` 핸들러가 호출되므로 커스텀 승인 로직을 구현할 수 있습니다. 아래 예시에서 `isCommandAuthorized`는 직접 정의하는 승인 검사를 대행합니다.

<Note>
  **`excludedCommands` vs `allowUnsandboxedCommands`:**

  * `excludedCommands`: 항상 자동으로 샌드박스를 우회하는 정적 명령어 목록 (예: `['docker']`). 모델은 이에 대한 제어권이 없습니다.
  * `allowUnsandboxedCommands`: 모델이 도구 입력에서 `dangerouslyDisableSandbox: true`를 설정하여 샌드박스 미적용 실행을 요청할지 여부를 런타임에 결정하도록 허용합니다.
</Note>

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Deploy my application",
  options: {
    sandbox: {
      enabled: true,
      allowUnsandboxedCommands: true // 모델이 샌드박스 미적용 실행을 요청할 수 있음
    },
    permissionMode: "default",
    canUseTool: async (tool, input) => {
      // 모델이 샌드박스 우회를 요청하는지 확인
      if (tool === "Bash" && input.dangerouslyDisableSandbox) {
        // 모델이 이 명령어를 샌드박스 외부에서 실행하도록 요청하고 있음
        console.log(`Unsandboxed command requested: ${input.command}`);

        if (isCommandAuthorized(input.command)) {
          return { behavior: "allow" as const, updatedInput: input };
        }
        return {
          behavior: "deny" as const,
          message: "Command not authorized for unsandboxed execution"
        };
      }
      return { behavior: "allow" as const, updatedInput: input };
    }
  }
})) {
  if ("result" in message) console.log(message.result);
}
```

이 패턴을 통해 다음을 수행할 수 있습니다:

* **모델 요청 감사:** 모델이 샌드박스 미적용 실행을 요청할 때 로깅
* **허용 목록 구현:** 특정 명령어만 샌드박스 없이 실행되도록 허용
* **승인 워크플로우 추가:** 권한이 필요한 작업에 대해 명시적 승인 요구

<Warning>
  `dangerouslyDisableSandbox: true`로 실행되는 명령어는 전체 시스템 접근 권한을 갖습니다. `canUseTool` 핸들러가 이러한 요청을 신중하게 검증하도록 하세요.

  `permissionMode`가 `bypassPermissions`로 설정되어 있고 `allowUnsandboxedCommands`가 활성화되어 있으면, 모델은 승인 프롬프트 없이 샌드박스 외부에서 명령어를 자율적으로 실행할 수 있습니다 (명시적 [`ask` 규칙](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)은 여전히 프롬프트를 강제함). 이 조합은 효과적으로 모델이 암묵적으로 샌드박스 격리를 탈출할 수 있게 합니다.
</Warning>

## 참고 항목

* [SDK 개요](/docs/en/agent-sdk/overview) - 일반 SDK 개념
* [Python SDK 레퍼런스](/docs/en/agent-sdk/python) - Python SDK 문서
* [CLI 레퍼런스](/docs/en/cli-reference) - 명령줄 인터페이스
* [일반 워크플로우](/docs/en/common-workflows) - 단계별 가이드
