> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 채널 참조 (Channels reference)

> Claude Code 세션으로 웹훅, 알림, 채팅 메시지를 푸시하는 MCP 서버를 구축하세요. 기능 선언, 알림 이벤트, 답장 도구, 발신자 제어, 권한 릴레이 등 채널 계약에 대한 참조 설명서입니다.

<Note>
  채널은 [리서치 프리뷰](/docs/en/channels#research-preview) 단계에 있습니다. Team 및 Enterprise 조직은 [명시적으로 활성화](/docs/en/channels#enterprise-controls)해야 합니다.
</Note>

채널은 터미널 외부에서 발생하는 일에 Claude가 반응할 수 있도록 Claude Code 세션에 이벤트를 푸시하는 MCP 서버입니다.

단방향 또는 양방향 채널을 만들 수 있습니다. 단방향 채널은 Claude가 조치를 취할 수 있도록 알림, 웹훅, 모니터링 이벤트를 전달합니다. 채팅 브리지와 같은 양방향 채널은 [답장 도구 노출](#답장-도구-노출)을 통해 Claude가 메시지를 다시 보낼 수 있게 합니다. 신뢰할 수 있는 발신자 경로를 사용하는 채널은 [권한 프롬프트 릴레이](#권한-프롬프트-릴레이)를 선택하여 원격으로 도구 사용을 승인하거나 거부할 수도 있습니다.

이 페이지에서는 다음 내용을 다룹니다:

* [개요](#개요): 채널의 작동 방식
* [사전 요구 사항](#사전-요구-사항): 요구 사항 및 일반적인 단계
* [예제: 웹훅 수신기 구축](#예제-웹훅-수신기-구축): 최소한의 단방향 가이드
* [서버 옵션](#서버-옵션): 생성자 필드
* [알림 형식](#알림-형식): 이벤트 페이로드 및 전달 동작
* [답장 도구 노출](#답장-도구-노출): Claude가 메시지를 응답으로 보내도록 허용
* [수신 메시지 제어](#수신-메시지-제어): 프롬프트 주입을 방지하기 위한 발신자 검사
* [권한 프롬프트 릴레이](#권한-프롬프트-릴레이): 원격 채널로 도구 승인 프롬프트 전달

채널을 직접 만드는 대신 기존 채널을 사용하려면 [채널](/docs/en/channels)을 참조하세요. Telegram, Discord, iMessage, fakechat이 리서치 프리뷰에 포함되어 있습니다.

## 개요

채널은 Claude Code와 동일한 머신에서 실행되는 [MCP](https://modelcontextprotocol.io) 서버입니다. Claude Code는 이를 자식 프로세스(subprocess)로 생성하고 stdio를 통해 통신합니다. 채널 서버는 외부 시스템과 Claude Code 세션 사이의 브리지 역할을 합니다:

* **채팅 플랫폼** (Telegram, Discord): 플러그인이 로컬에서 실행되며 새 메시지에 대해 플랫폼의 API를 폴링합니다. 누군가 봇에게 DM을 보내면 플러그인이 메시지를 받아 Claude에게 전달합니다. 외부에 노출할 URL이 필요 없습니다.
* **웹훅** (CI, 모니터링): 서버가 로컬 HTTP 포트에서 대기합니다. 외부 시스템이 해당 포트로 POST를 보내면 서버가 페이로드를 Claude에게 푸시합니다.

<img src="https://mintcdn.com/claude-code/9FG0ZKj9uKYiHmbi/images/channel-architecture.svg?fit=max&auto=format&n=9FG0ZKj9uKYiHmbi&q=85&s=9a037b7da80184ae49015c0256b21a1f" alt="Architecture diagram showing external systems connecting to your local channel server, which communicates with Claude Code over stdio" width="600" height="220" data-path="images/channel-architecture.svg" />

## 사전 요구 사항

필수 요구 사항은 [`@modelcontextprotocol/sdk`](https://www.npmjs.com/package/@modelcontextprotocol/sdk) 패키지와 Node.js 호환 런타임뿐입니다. [Bun](https://bun.sh), [Node](https://nodejs.org), [Deno](https://deno.com) 모두 작동합니다. 리서치 프리뷰의 사전 빌드된 플러그인은 Bun을 사용하지만, 나만의 채널이 Bun을 사용할 필요는 없습니다.

서버에서 수행해야 하는 작업:

1. Claude Code가 알림 수신기(notification listener)를 등록하도록 `claude/channel` 기능 선언
2. 이벤트가 발생할 때 `notifications/claude/channel` 이벤트 전송
3. [stdio 전송](https://modelcontextprotocol.io/docs/concepts/transports#standard-io)을 통해 연결 (Claude Code가 서버를 하위 프로세스로 실행)

[서버 옵션](#서버-옵션) 및 [알림 형식](#알림-형식) 섹션에서 이러한 내용을 자세히 다룹니다. 전체 프로세스는 [예제: 웹훅 수신기 구축](#예제-웹훅-수신기-구축)을 참조하세요.

리서치 프리뷰 동안 사용자 지정 채널은 [승인된 허용 목록](/docs/en/channels#supported-channels)에 포함되지 않습니다. 로컬에서 테스트하려면 `--dangerously-load-development-channels`를 사용하세요. 자세한 내용은 [리서치 프리뷰 동안 테스트](#리서치-프리뷰-동안-테스트)를 참조하세요.

## 예제: 웹훅 수신기 구축

이 안내에서는 HTTP 요청을 대기하고 이를 Claude Code 세션으로 전달하는 단일 파일 서버를 구축합니다. 이 작업이 완료되면 CI 파이프라인, 모니터링 알림, `curl` 명령 등 HTTP POST를 보낼 수 있는 모든 외부 시스템이 Claude에 이벤트를 푸시할 수 있습니다.

이 예제에서는 내장 HTTP 서버와 TypeScript 지원을 위해 [Bun](https://bun.sh)을 런타임으로 사용합니다. 대신 [Node](https://nodejs.org) 또는 [Deno](https://deno.com)를 사용할 수 있으며, 유일한 요구 사항은 [MCP SDK](https://www.npmjs.com/package/@modelcontextprotocol/sdk)입니다.

<Steps>
  <Step title="프로젝트 생성">
    이 페이지의 뒷부분에 나오는 권한 릴레이 예제는 `zod`를 직접 임포트하므로 MCP SDK와 함께 설치합니다. 새 디렉터리를 생성하고 두 패키지를 설치합니다:

    ```bash theme={null}
    mkdir webhook-channel && cd webhook-channel
    bun add @modelcontextprotocol/sdk zod
    ```
  </Step>

  <Step title="채널 서버 작성">
    `webhook.ts`라는 파일을 만듭니다. 이것이 전체 채널 서버 코드입니다. stdio를 통해 Claude Code에 연결하고 포트 8788에서 HTTP POST를 대기합니다. 요청이 도착하면 본문(body)을 채널 이벤트로 Claude에 푸시합니다.

    ```ts title="webhook.ts" theme={null}
    #!/usr/bin/env bun
    import { Server } from '@modelcontextprotocol/sdk/server/index.js'
    import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

    // MCP 서버를 생성하고 채널로 선언
    const mcp = new Server(
      { name: 'webhook', version: '0.0.1' },
      {
        // 이 키를 통해 채널로 지정됨 — Claude Code가 수신기를 등록함
        capabilities: { experimental: { 'claude/channel': {} } },
        // 이러한 이벤트를 처리하는 방법을 알 수 있도록 Claude의 시스템 프롬프트에 추가됨
        instructions: 'Events from the webhook channel arrive as <channel source="webhook" ...>. They are one-way: read them and act, no reply expected.',
      },
    )

    // stdio를 통해 Claude Code에 연결 (Claude Code가 이 프로세스를 생성함)
    await mcp.connect(new StdioServerTransport())

    // 모든 POST 요청을 Claude로 전달하는 HTTP 서버 시작
    Bun.serve({
      port: 8788,  // 사용 가능한 포트 사용
      // localhost 전용: 이 머신 외부에서는 POST할 수 없음
      hostname: '127.0.0.1',
      async fetch(req) {
        const body = await req.text()
        await mcp.notification({
          method: 'notifications/claude/channel',
          params: {
            content: body,  // <channel> 태그의 본문이 됨
            // 각 키는 태그 속성이 됨 (예: <channel path="/" method="POST">)
            meta: { path: new URL(req.url).pathname, method: req.method },
          },
        })
        return new Response('ok')
      },
    })
    ```

    이 파일은 순서대로 세 가지 작업을 수행합니다:

    * **서버 설정**: capabilities에 `claude/channel`을 포함하여 MCP 서버를 생성합니다. 이를 통해 Claude Code에 채널임을 알립니다. [`instructions`](#서버-옵션) 문자열은 Claude의 시스템 프롬프트에 들어갑니다. Claude에게 어떤 이벤트를 기대할지, 답장 여부, 답장 시 라우팅 방식을 알립니다.
    * **Stdio 연결**: stdin/stdout을 통해 Claude Code에 연결합니다. 이는 일반적인 [MCP 서버](https://modelcontextprotocol.io/docs/concepts/transports#standard-io) 방식입니다. Claude Code가 자식 프로세스로 실행합니다.
    * **HTTP 수신기**: 포트 8788에서 로컬 웹 서버를 시작합니다. 모든 POST 본문은 `mcp.notification()`을 통해 채널 이벤트로 Claude에게 전달됩니다. `content`는 이벤트 본문이 되고 각 `meta` 항목은 `<channel>` 태그의 속성이 됩니다. 수신기는 `mcp` 인스턴스에 접근해야 하므로 동일한 프로세스에서 실행됩니다. 대규모 프로젝트의 경우 별도의 모듈로 분리할 수 있습니다.
  </Step>

  <Step title="Claude Code에 서버 등록">
    Claude Code가 서버를 시작하는 방법을 알 수 있도록 MCP 설정에 서버를 추가합니다. 동일한 디렉터리의 프로젝트 수준 `.mcp.json`에는 상대 경로를 사용하고, `~/.claude.json` 사용자 수준 설정에는 어떤 프로젝트에서든 서버를 찾을 수 있도록 절대 경로를 사용합니다:

    ```json title=".mcp.json" theme={null}
    {
      "mcpServers": {
        "webhook": { "command": "bun", "args": ["./webhook.ts"] }
      }
    }
    ```

    Claude Code는 시작 시 MCP 설정을 읽고 각 서버를 자식 프로세스로 생성합니다.
  </Step>

  <Step title="테스트">
    리서치 프리뷰 기간 동안 사용자 지정 채널은 허용 목록에 없으므로 개발 플래그를 사용하여 Claude Code를 시작합니다:

    ```bash theme={null}
    claude --dangerously-load-development-channels server:webhook
    ```

    Claude Code는 로드 중인 개발 채널을 나열하는 전체 화면 경고 대화 상자를 보여줍니다. **I am using this for local development**를 선택하여 계속 진행하거나 **Exit**를 선택하여 종료합니다.

    이 프로젝트에서 세션을 처음 시작할 때 Claude Code는 `.mcp.json`에서 새 서버를 사용하기 전에 동의를 요청합니다. 대화 상자에 "New MCP server found in this project: webhook"이 표시됩니다. **Use this MCP server**를 선택하여 계속합니다.

    수락하면 Claude Code가 `webhook.ts`를 자식 프로세스로 생성하고 설정한 포트(이 예제에서는 8788)에서 HTTP 수신기가 자동으로 시작됩니다. 서버를 직접 실행할 필요가 없습니다.

    시작 배너 아래의 흐릿한 알림을 통해 채널이 등록되었음을 확인할 수 있습니다: `Channels (experimental) messages from server:webhook inject directly in this session · restart without --dangerously-load-development-channels to stop`.

    "blocked by org policy"가 표시되면 조직 관리자가 먼저 [채널을 활성화](/docs/en/channels#enterprise-controls)해야 합니다.

    별도의 터미널에서 서버로 메시지가 포함된 HTTP POST를 전송하여 웹훅을 시뮬레이션합니다. 이 예제에서는 CI 실패 알림을 포트 8788(또는 설정한 포트)로 보냅니다:

    ```bash theme={null}
    curl -X POST localhost:8788 -d "build failed on main: https://ci.example.com/run/1234"
    ```

    페이로드가 `<channel>` 태그 형태로 Claude의 컨텍스트에 도착합니다:

    ```text theme={null}
    <channel source="webhook" path="/" method="POST">build failed on main: https://ci.example.com/run/1234</channel>
    ```

    터미널에는 원시 태그 대신 `← webhook: build failed on main: https://ci.example.com/run/1234`와 같은 한 줄 요약이 표시됩니다. 그런 다음 Claude가 파일 읽기, 명령 실행 등 메시지가 요구하는 응답을 시작하는 것을 볼 수 있습니다. 이것은 단방향 채널이므로 Claude는 세션 내에서 조치를 취하지만 웹훅을 통해 응답을 다시 보내지는 않습니다. 답장 기능을 추가하려면 [답장 도구 노출](#답장-도구-노출)을 참조하세요.

    이벤트가 도착하지 않는 경우 `curl`이 반환한 내용에 따라 진단합니다:

    * **`curl`은 성공하지만 Claude에게 도달하지 않음**: 세션에서 `/mcp`를 실행하여 서버 상태를 확인하세요. "Failed to connect"는 일반적으로 서버 파일의 의존성이나 임포트 오류를 의미합니다. `~/.claude/debug/<session-id>.txt`에서 stderr 트레이스를 확인하세요.
    * **`curl`이 "connection refused"로 실패**: 포트가 아직 바인딩되지 않았거나 이전 실행의 프로세스가 포트를 점유하고 있는 것입니다. `lsof -i :<port>`로 수신 대기 중인 프로세스를 확인하고, 세션을 재시작하기 전에 해당 프로세스를 `kill`하세요.
  </Step>
</Steps>

[fakechat 서버](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/fakechat)는 이 패턴을 웹 UI, 파일 첨부 및 양방향 채팅을 위한 답장 도구로 확장한 예입니다.

## 리서치 프리뷰 동안 테스트

리서치 프리뷰 기간 동안 모든 채널은 등록을 위해 [승인된 허용 목록](/docs/en/channels#research-preview)에 있어야 합니다. 개발 플래그는 확인 프롬프트 후 특정 항목에 대한 허용 목록을 우회합니다. 아래 예는 두 가지 항목 유형을 모두 보여줍니다:

```bash theme={null}
# 개발 중인 플러그인 테스트
claude --dangerously-load-development-channels plugin:yourplugin@yourmarketplace

# 순수 .mcp.json 서버 테스트 (플러그인 래퍼가 아직 없는 경우)
claude --dangerously-load-development-channels server:webhook
```

우회 조치는 항목별로 적용됩니다. 이 플래그를 `--channels`와 함께 사용하더라도 `--channels` 항목으로 우회가 확장되지는 않습니다. 리서치 프리뷰 동안 승인된 허용 목록은 Anthropic이 관리하므로, 제작 및 테스트 중인 채널은 개발 플래그를 사용해 유지됩니다.

<Note>
  이 플래그는 허용 목록만 우회합니다. `channelsEnabled` 조직 정책은 여전히 적용됩니다. 신뢰할 수 없는 소스의 채널을 실행할 때 사용하지 마세요.
</Note>

## 서버 옵션

채널은 [`Server`](https://modelcontextprotocol.io/docs/learn/server-concepts) 생성자에서 이러한 옵션을 설정합니다. `instructions` 및 `capabilities.tools` 필드는 [표준 MCP](https://modelcontextprotocol.io/docs/learn/server-concepts) 방식이며, `capabilities.experimental['claude/channel']` 및 `capabilities.experimental['claude/channel/permission']`은 채널 전용 확장 항목입니다:

| 필드                                                     | 타입     | 설명                                                                                                                                                                                                                                                                  |
| :------------------------------------------------------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `capabilities.experimental['claude/channel']`            | `object` | 필수. 항상 `{}`. 존재 시 알림 수신기가 등록됩니다.                                                                                                                                                                                                                     |
| `capabilities.experimental['claude/channel/permission']` | `object` | 선택 사항. 항상 `{}`. 이 채널이 권한 릴레이 요청을 받을 수 있음을 선언합니다. 선언되면 Claude Code가 도구 승인 프롬프트를 채널로 전달하여 원격으로 승인하거나 거부할 수 있습니다. [권한 프롬프트 릴레이](#권한-프롬프트-릴레이)를 참조하세요.                         |
| `capabilities.tools`                                     | `object` | 양방향 전용. 항상 `{}`. 표준 MCP 도구 기능입니다. [답장 도구 노출](#답장-도구-노출)을 참조하세요.                                                                                                                                                                     |
| `instructions`                                           | `string` | 권장. Claude의 시스템 프롬프트에 추가됩니다. Claude에게 예상되는 이벤트, `<channel>` 태그 속성의 의미, 답장 여부, 답장 시 사용할 도구 및 전달할 속성(예: `chat_id`)을 알립니다.                                                                                           |

단방향 채널을 생성하려면 `capabilities.tools`를 생략하세요. 다음 예제는 채널 기능, 도구 및 지시사항이 설정된 양방향 구성을 보여줍니다:

```ts theme={null}
import { Server } from '@modelcontextprotocol/sdk/server/index.js'

const mcp = new Server(
  { name: 'your-channel', version: '0.0.1' },
  {
    capabilities: {
      experimental: { 'claude/channel': {} },  // 채널 수신기 등록
      tools: {},  // 단방향 채널의 경우 생략
    },
    // Claude가 이벤트를 처리하는 방법을 알 수 있도록 시스템 프롬프트에 추가
    instructions: 'Messages arrive as <channel source="your-channel" ...>. Reply with the reply tool.',
  },
)
```

이벤트를 푸시하려면 메서드가 `notifications/claude/channel`인 `mcp.notification()`을 호출하세요. 파라미터는 다음 섹션에서 설명합니다.

## 알림 형식

서버는 두 개의 파라미터와 함께 `notifications/claude/channel`을 전송합니다:

| 필드      | 타입                     | 설명                                                                                                                                                                                                                                                           |
| :-------- | :----------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `content` | `string`                 | 이벤트 본문. `<channel>` 태그의 본문으로 전달됩니다.                                                                                                                                                                                                                   |
| `meta`    | `Record<string, string>` | 선택 사항. 각 항목은 채팅 ID, 발신자 이름, 알림 심각도와 같은 라우팅 컨텍스트를 위해 `<channel>` 태그의 속성이 됩니다. 키는 식별자(문자, 숫자, 밑줄만 가능)여야 합니다. 하이픈이나 기타 문자가 포함된 키는 무시됩니다.                                                      |

서버는 `Server` 인스턴스에서 `mcp.notification()`을 호출하여 이벤트를 푸시합니다. 다음 예는 두 개의 메타 키와 함께 CI 실패 알림을 푸시합니다:

```ts theme={null}
await mcp.notification({
  method: 'notifications/claude/channel',
  params: {
    content: 'build failed on main: https://ci.example.com/run/1234',
    meta: { severity: 'high', run_id: '1234' },
  },
})
```

이벤트는 `<channel>` 태그로 감싸져 Claude의 컨텍스트에 도착합니다. `source` 속성은 서버의 구성된 이름으로 자동 설정됩니다:

```text theme={null}
<channel source="your-channel" severity="high" run_id="1234">
build failed on main: https://ci.example.com/run/1234
</channel>
```

알림은 확인 응답(ack)을 받지 않습니다. `mcp.notification()`의 `await`는 Claude가 이벤트를 처리했을 때가 아니라 메시지가 전송 계층에 기록되었을 때 확인됩니다. 세션에서 서버를 채널로 로드하지 않았거나 조직 정책에 의해 차단된 경우, 이벤트는 에러 없이 무시됩니다.

전달 확인이 필요한 경우 서버에서 이벤트 상태를 추적하고 Claude가 상태를 다시 보고하기 위해 호출할 수 있는 [답장 도구](#답장-도구-노출)를 노출하세요.

이벤트는 세션 큐에 들어간 후 순서대로 처리됩니다. Claude가 작업 중일 때 여러 알림이 도착하면 다음 턴에 함께 전달되며 Claude가 그룹으로 처리합니다. 독립적인 이벤트 스트림을 동시에 처리하려면 별도의 세션을 실행하세요.

## 답장 도구 노출

채널이 알림 전송용이 아닌 채팅 브리지처럼 양방향인 경우, Claude가 메시지를 응답으로 보낼 수 있도록 표준 [MCP 도구](https://modelcontextprotocol.io/docs/concepts/tools)를 노출하세요. 도구 등록에는 채널 전용의 특수한 부분이 없습니다. 답장 도구에는 세 가지 요소가 있습니다:

1. Claude Code가 도구를 검색할 수 있도록 `Server` 생성자 capabilities에 `tools: {}` 항목 추가
2. 도구 스키마를 정의하고 전송 로직을 구현하는 도구 핸들러
3. `Server` 생성자의 `instructions` 문자열을 통해 Claude가 도구를 언제 어떻게 호출해야 하는지 전달

위의 [웹훅 수신기 예제](#예제-웹훅-수신기-구축)에 이 기능을 추가하는 방법:

<Steps>
  <Step title="도구 검색 활성화">
    `webhook.ts` 파일의 `Server` 생성자에서 capabilities에 `tools: {}`를 추가하여 Claude Code가 서버에서 제공하는 도구를 인식하도록 합니다:

    ```ts theme={null}
    capabilities: {
      experimental: { 'claude/channel': {} },
      tools: {},  // 도구 검색 활성화
    },
    ```
  </Step>

  <Step title="답장 도구 등록">
    `webhook.ts`에 다음을 추가합니다. `import`는 파일 상단의 다른 임포트 문과 함께 위치하고, 두 개의 핸들러는 `Server` 생성자와 `mcp.connect()` 사이에 배치됩니다. 이렇게 하면 Claude가 `chat_id` 및 `text`와 함께 호출할 수 있는 `reply` 도구가 등록됩니다:

    ```ts theme={null}
    // webhook.ts 상단에 이 임포트 추가
    import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

    // Claude가 시작 시 서버가 제공하는 도구를 확인하기 위해 이 요청을 보냄
    mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
      tools: [{
        name: 'reply',
        description: 'Send a message back over this channel',
        // inputSchema는 Claude에게 어떤 인자를 전달해야 하는지 알려줌
        inputSchema: {
          type: 'object',
          properties: {
            chat_id: { type: 'string', description: 'The conversation to reply in' },
            text: { type: 'string', description: 'The message to send' },
          },
          required: ['chat_id', 'text'],
        },
      }],
    }))

    // Claude가 도구를 실행하고자 할 때 이를 호출함
    mcp.setRequestHandler(CallToolRequestSchema, async req => {
      if (req.params.name === 'reply') {
        const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
        // send()는 아웃바운드: 채팅 플랫폼으로 POST를 보내거나,
        // 로컬 테스트의 경우 아래 전체 예제에 표시된 SSE 브로드캐스트 수행
        send(`Reply to ${chat_id}: ${text}`)
        return { content: [{ type: 'text', text: 'sent' }] }
      }
      throw new Error(`unknown tool: ${req.params.name}`)
    })
    ```
  </Step>

  <Step title="지시사항(instructions) 업데이트">
    Claude가 도구를 통해 답장을 라우팅해야 함을 알 수 있도록 `Server` 생성자의 `instructions` 문자열을 업데이트합니다. 이 예제에서는 인바운드 태그의 `chat_id`를 전달하도록 Claude에 안내합니다:

    ```ts theme={null}
    instructions: 'Messages arrive as <channel source="webhook" chat_id="...">. Reply with the reply tool, passing the chat_id from the tag.'
    ```
  </Step>
</Steps>

다음은 양방향 지원이 포함된 전체 `webhook.ts` 코드입니다. 아웃바운드 답장은 [Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) (SSE)를 사용하는 `GET /events`를 통해 스트리밍되어 `curl -N localhost:8788/events`로 라이브로 실시간 모니터링할 수 있으며, 인바운드 채팅은 `POST /`로 도착합니다:

```ts title="Full webhook.ts with reply tool" expandable theme={null}
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

// --- Outbound: write to any curl -N listeners on /events --------------------
// 실제 브리지는 대신 채팅 플랫폼에 POST를 전송합니다.
const listeners = new Set<(chunk: string) => void>()
function send(text: string) {
  const chunk = text.split('\n').map(l => `data: ${l}\n`).join('') + '\n'
  for (const emit of listeners) emit(chunk)
}

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: {
      experimental: { 'claude/channel': {} },
      tools: {},
    },
    instructions: 'Messages arrive as <channel source="webhook" chat_id="...">. Reply with the reply tool, passing the chat_id from the tag.',
  },
)

mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'The conversation to reply in' },
        text: { type: 'string', description: 'The message to send' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})

await mcp.connect(new StdioServerTransport())

let nextId = 1
Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  idleTimeout: 0,  // 유휴 SSE 스트림을 닫지 않음
  async fetch(req) {
    const url = new URL(req.url)

    // GET /events: curl -N이 Claude의 답장을 실시간으로 시청할 수 있는 SSE 스트림
    if (req.method === 'GET' && url.pathname === '/events') {
      const stream = new ReadableStream({
        start(ctrl) {
          ctrl.enqueue(': connected\n\n')  // curl에 즉시 출력되도록 함
          const emit = (chunk: string) => ctrl.enqueue(chunk)
          listeners.add(emit)
          req.signal.addEventListener('abort', () => listeners.delete(emit))
        },
      })
      return new Response(stream, {
        headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
      })
    }

    // POST: 채널 이벤트로 Claude에게 전달
    const body = await req.text()
    const chat_id = String(nextId++)
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,
        meta: { chat_id, path: url.pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

[fakechat 서버](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/fakechat)는 파일 첨부 및 메시지 편집 기능이 포함된 완전한 예제를 제공합니다.

## 수신 메시지 제어

제어되지 않는 채널은 프롬프트 주입(prompt injection)의 통로가 될 수 있습니다. 엔드포인트에 접근할 수 있는 사람이라면 누구든 Claude 앞에 텍스트를 전달할 수 있습니다. 채팅 플랫폼이나 공개 엔드포인트를 수신하는 채널은 이벤트를 발송하기 전에 실제 발신자 검사를 거쳐야 합니다.

`mcp.notification()`을 호출하기 전에 발신자를 허용 목록과 비교하여 검사하세요. 다음 예제는 목록에 없는 발신자의 모든 메시지를 무시합니다:

```ts theme={null}
const allowed = new Set(loadAllowlist())  // access.json 등에서 가져옴

// 메시지 핸들러 내부, 이벤트 발송 전:
if (!allowed.has(message.from.id)) {  // 대화방이 아닌 발신자 ID 기준
  return  // 아무 작업 없이 무시
}
await mcp.notification({ ... })
```

채팅방이나 그룹의 식별자가 아닌 발신자 개인의 식별자를 기준으로 제어하세요(예제에서는 `message.chat.id`가 아닌 `message.from.id`). 그룹 채팅에서는 두 값이 다르며, 그룹 식별자로 제어할 경우 허용 목록에 있는 그룹의 누구라도 세션에 메시지를 주입할 수 있게 됩니다.

[Telegram](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram) 및 [Discord](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord) 채널은 동일한 방식으로 발신자 허용 목록을 관리합니다. 목록은 페어링을 통해 구성됩니다: 사용자가 봇에게 DM을 보내면 봇이 페어링 코드로 답장하고, 사용자가 Claude Code 세션에서 이를 승인하면 해당 플랫폼 ID가 추가됩니다. 전체 페어링 흐름은 각 구현을 참조하세요. [iMessage](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage) 채널은 시작 시 Messages 데이터베이스에서 사용자 자신의 주소를 감지하여 자동으로 통과시키고, 다른 발신자는 핸들을 통해 추가하는 방식을 취합니다.

## 권한 프롬프트 릴레이

Claude가 승인이 필요한 도구를 호출하면 로컬 터미널 대화 상자가 열리고 세션이 대기 상태가 됩니다. 양방향 채널은 수신을 선택하여 다른 기기에서 동시에 동일한 프롬프트를 받아 중계받을 수 있습니다. 두 곳 모두 활성 상태를 유지하므로 터미널이나 휴대폰 중 하나에서 응답할 수 있으며, Claude Code는 먼저도착하는 응답을 적용하고 다른 쪽은 닫습니다.

릴레이는 `Bash`, `Write`, `Edit`와 같은 도구 사용 승인을 다룹니다. 프로젝트 신뢰 및 MCP 서버 동의 대화 상자는 릴레이되지 않으며 로컬 터미널에만 표시됩니다.

### 릴레이 작동 방식

권한 프롬프트가 열리면 릴레이 루프는 다음 4단계로 작동합니다:

1. Claude Code가 짧은 요청 ID(request ID)를 생성하고 서버에 알림을 전송
2. 서버가 채팅 앱에 프롬프트와 요청 ID 전달
3. 원격 사용자가 해당 ID와 함께 예/아니오(yes/no)로 답장
4. 인바운드 핸들러가 답장을 판정(verdict)으로 파싱하고, ID가 열린 요청과 일치할 때만 Claude Code가 이를 적용

로컬 터미널 대화 상자는 이 과정 동안 열려 있습니다. 원격 판정이 도착하기 전에 터미널에서 누군가 응답하면 해당 응답이 먼저 적용되고 보류 중인 원격 요청은 폐기됩니다.

<img src="https://mintcdn.com/claude-code/9FG0ZKj9uKYiHmbi/images/channel-permission-relay.svg?fit=max&auto=format&n=9FG0ZKj9uKYiHmbi&q=85&s=97d57f128f0da55f105ab1e3a7e10240" alt="Sequence diagram: Claude Code sends a permission_request notification to the channel server, the server formats and sends the prompt to the chat app, the human replies with a verdict, and the server parses that reply into a permission notification back to Claude Code" width="600" height="230" data-path="images/channel-permission-relay.svg" />

### 권한 요청 필드

Claude Code의 아웃바운드 알림은 `notifications/claude/channel/permission_request`입니다. [채널 알림](#알림-형식)과 마찬가지로 전송 방식은 표준 MCP이지만 메서드와 스키마는 Claude Code 확장입니다. `params` 객체에는 서버가 보내는 프롬프트에 서식을 지정하는 4개의 문자열 필드가 있습니다:

| 필드            | 설명                                                                                                                                                                                                                                                                                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `request_id`    | 휴대폰에서 입력할 때 `1`이나 `I`로 오인되지 않도록 `l`을 제외한 `a`-`z` 범위의 소문자 5자리로 구성됩니다. 답장에서 그대로 전달받을 수 있도록 아웃바운드 프롬프트에 포함하세요. Claude Code는 발급한 ID를 포함하는 판정만 수락합니다. 로컬 터미널 대화 상자에는 이 ID가 표시되지 않으므로 아웃바운드 핸들러가 이 ID를 알 수 있는 유일한 경로입니다. |
| `tool_name`     | Claude가 사용하고자 하는 도구의 이름(예: `Bash`, `Write`).                                                                                                                                                                                                                                                                                               |
| `description`   | 해당 도구 호출이 수행하는 작업에 대한 설명입니다(명령어 자체는 아님). Bash 호출의 경우 명령에 대한 Claude의 설명이며, 모델이 설명을 제공하지 않으면 필드는 `Run shell command` 상수가 되며 명령어에 대한 상세 정보가 포함되지 않습니다. 공간이 충분한 경우 `input_preview`를 렌더링하세요.                                                              |
| `input_preview` | 최상위 필드별로 키가 지정된, JSON 형식의 디스플레이 텍스트로 표현된 도구 인자입니다. Bash의 경우 명령어이며, Write의 경우 파일 경로 및 내용입니다. 한 줄 메시지만 보낼 수 있는 경우 프롬프트에서 생략하세요. 서버에서 표시할 내용을 결정합니다.                                                                                                        |

Claude Code v2.1.211 이상 클라이언트는 두 필드를 중계하기 전에 정화(sanitize) 작업을 거칩니다: 텍스트 방향 재설정 및 숨김 문자를 무력화하고, 따옴표 및 부등호 유사 문자를 변환하며, 공백 연속을 단일 공백으로 압축하고, `input_preview`의 최상위 필드별로 최대 3,500개 코드 포인트까지 전체 텍스트를 제공하여 JSON의 구조적 따옴표도 유지합니다. 이보다 긴 값은 시작과 끝 부분이 표시되고 중앙에 `⋯ N code points elided ⋯` 마커가 포함되어 긴 명령어의 끝부분을 확인할 수 있습니다. 이전 클라이언트는 `description`을 그대로 전달하고 `input_preview`를 말줄임표와 함께 200 UTF-16 단위로 잘라냅니다. 클라이언트 환경을 직접 제어하지 않는 한 두 필드 모두 신뢰할 수 없는 데이터로 취급하세요.

서버가 다시 보내는 판정은 `notifications/claude/channel/permission`이며, 위의 ID를 전달하는 `request_id`와 `'allow'` 또는 `'deny'`로 설정된 `behavior` 두 가 지 필드를 가집니다. Allow는 도구 호출을 계속 진행하고, Deny는 로컬 대화 상자에서 No를 입력하는 것과 동일하게 거부합니다. 어떤 판정이든 이후 호출에는 영향을 주지 않습니다.

### 채팅 브리지에 릴레이 추가

양방향 채널에 권한 릴레이를 추가하려면 3가지 요소가 필요합니다:

1. Claude Code가 프롬프트를 전달하도록 `Server` 생성자의 `experimental` 기능 아래에 `claude/channel/permission: {}` 항목 추가
2. 프롬프트를 포맷팅하고 플랫폼 API를 통해 전송하는 `notifications/claude/channel/permission_request` 알림 핸들러
3. `yes <id>` 또는 `no <id>`를 인식하고 텍스트를 Claude에 전달하는 대신 `notifications/claude/channel/permission` 판정을 전송하는 인바운드 메시지 핸들러의 검사 로직

채널을 통해 답장할 수 있는 사람은 세션에서 도구 사용을 승인하거나 거부할 수 있으므로, 채널이 [발신자를 인증](#수신-메시지-제어)하는 경우에만 이 기능을 선언하세요.

[답장 도구 노출](#답장-도구-노출)에서 구축한 양방향 채팅 브리지에 이를 추가하려면:

<Steps>
  <Step title="권한 기능 선언">
    `Server` 생성자에서 `experimental` 아래 `claude/channel`과 함께 `claude/channel/permission: {}`를 추가합니다:

    ```ts theme={null}
    capabilities: {
      experimental: {
        'claude/channel': {},
        'claude/channel/permission': {},  // 권한 릴레이 선택
      },
      tools: {},
    },
    ```
  </Step>

  <Step title="수신 요청 처리">
    `Server` 생성자와 `mcp.connect()` 사이에 알림 핸들러를 등록합니다. Claude Code는 권한 대화 상자가 열릴 때 [4개 요청 필드](#권한-요청-필드)와 함께 이를 호출합니다. 핸들러는 플랫폼에 맞게 프롬프트 형식을 지정하고 ID로 답장하기 위한 안내를 포함합니다:

    ```ts theme={null}
    import { z } from 'zod'

    // setNotificationHandler는 method 필드의 z.literal에 따라 라우팅하므로,
    // 이 스키마는 유효성 검사기이자 디스패치 키 역할을 모두 수행합니다.
    const PermissionRequestSchema = z.object({
      method: z.literal('notifications/claude/channel/permission_request'),
      params: z.object({
        request_id: z.string(),     // 소문자 5자리, 프롬프트에 그대로 포함
        tool_name: z.string(),      // 예: "Bash", "Write"
        description: z.string(),    // 이번 호출의 요약. 신뢰할 수 없는 데이터로 취급.
        input_preview: z.string(),  // JSON 형식의 도구 인자. 신뢰할 수 없는 데이터로 취급.
      }),
    })

    mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
      // send()는 아웃바운드: 채팅 플랫폼으로 POST를 보내거나,
      // 로컬 테스트의 경우 아래 전체 예제에 표시된 SSE 브로드캐스트 수행
      send(
        `Claude wants to run ${params.tool_name}: ${params.description}\n` +
        // input_preview는 실제 인자를 전달합니다. 공간이 있으면 렌더링하세요.
        // Bash의 경우 description만으로는 명령어 세부사항 없이 "Run shell command"일 수 있습니다.
        `${params.input_preview}\n\n` +
        // 안내문의 ID는 3단계에서 인바운드 핸들러가 파싱하는 항목입니다.
        `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
      )
    })
    ```
  </Step>

  <Step title="인바운드 핸들러에서 판정 가로채기">
    인바운드 핸들러는 플랫폼에서 메시지를 수신하는 루프 또는 콜백입니다. [발신자를 제어](#수신-메시지-제어)하고 `notifications/claude/channel`을 전송하여 채팅을 Claude로 전달하는 동일한 위치입니다. 채팅 전달 호출 앞에 판정 형식을 인식하고 대신 권한 알림을 보낼 수 있는 검사를 추가하세요.

    정규식은 Claude Code가 생성하는 ID 형식(소문자 5자리, `l` 제외)과 일치합니다. `/i` 플래그는 휴대폰 자동 완성이 대문자로 변경하는 것을 허용하며, 판정을 다시 보내기 전에 수집된 ID를 소문자로 변환합니다.

    ```ts theme={null}
    // "y abcde", "yes abcde", "n abcde", "no abcde" 패턴 일치
    // [a-km-z]는 Claude Code가 사용하는 ID 문자 알파벳입니다 (소문자, 'l' 제외).
    // /i 플래그는 휴대폰 자동 완성을 허용합니다. 보내기 전에 소문자로 변환합니다.
    const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i

    async function onInbound(message: PlatformMessage) {
      if (!allowed.has(message.from.id)) return  // 먼저 발신자 검사

      const m = PERMISSION_REPLY_RE.exec(message.text)
      if (m) {
        // m[1]은 판정 단어, m[2]는 요청 ID
        // 채팅 대신 Claude Code로 판정 알림 전송
        await mcp.notification({
          method: 'notifications/claude/channel/permission',
          params: {
            request_id: m[2].toLowerCase(),  // 대문자 자동 완성을 고려해 정규화
            behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
          },
        })
        return  // 판정으로 처리되었으므로 일반 채팅으로 전송하지 않음
      }

      // 판정 형식과 일치하지 않음: 일반 채팅 경로로 전환
      await mcp.notification({
        method: 'notifications/claude/channel',
        params: { content: message.text, meta: { chat_id: String(message.chat.id) } },
      })
    }
    ```
  </Step>
</Steps>

Claude Code는 로컬 터미널 대화 상자도 계속 열어두므로 어느 위치에서든 응답할 수 있으며, 먼저 도착하는 응답이 적용됩니다. 예상되는 형식과 정확히 일치하지 않는 원격 답장은 두 가지 이유 중 하나로 처리되지 않으며, 두 경우 모두 대화 상자가 계속 열려 있습니다:

* **형식이 다름**: 인바운드 핸들러의 정규식이 일치하지 않아 ID 없는 `approve it` 또는 `yes`와 같은 텍스트는 Claude에 대한 일반 메시지로 전달됩니다.
* **형식은 맞지만 ID가 다름**: 서버가 판정을 전송하지만 Claude Code가 해당 ID의 열린 요청을 찾을 수 없어 무시합니다.

### 전체 예제

아래 구성된 `webhook.ts` 코드는 이 페이지의 3가지 확장 기능인 답장 도구, 발신자 검사, 권한 릴레이를 통합합니다. 이 코드부터 시작하는 경우 초기 설명서의 [프로젝트 설정 및 `.mcp.json` 항목](#예제-웹훅-수신기-구축)도 필요합니다.

curl에서 양방향을 모두 테스트할 수 있도록 HTTP 수신기는 두 개의 경로를 제공합니다:

* **`GET /events`**: SSE 스트림을 열어 두고 각 아웃바운드 메시지를 `data:` 줄로 푸시하므로 `curl -N`으로 Claude의 답장과 권한 프롬프트가도착하는 것을 라이브로 모니터링할 수 있습니다.
* **`POST /`**: 인바운드 측으로, 이전과 동일한 핸들러이지만 이제 채팅 전달 브랜치 이전에 판정 형식 검사가 삽입되었습니다.

```ts title="Full webhook.ts with permission relay" expandable theme={null}
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'
import { z } from 'zod'

// --- Outbound: write to any curl -N listeners on /events --------------------
// 실제 브리지는 대신 채팅 플랫폼에 POST를 전송합니다.
const listeners = new Set<(chunk: string) => void>()
function send(text: string) {
  const chunk = text.split('\n').map(l => `data: ${l}\n`).join('') + '\n'
  for (const emit of listeners) emit(chunk)
}

// 발신자 허용 목록. 로컬 가이드에서는 단일 X-Sender 헤더 값인 "dev"를 신뢰합니다.
// 실제 브리지는 플랫폼의 사용자 ID를 확인해야 합니다.
const allowed = new Set(['dev'])

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: {
      experimental: {
        'claude/channel': {},
        'claude/channel/permission': {},  // 권한 릴레이 선택
      },
      tools: {},
    },
    instructions:
      'Messages arrive as <channel source="webhook" chat_id="...">. ' +
      'Reply with the reply tool, passing the chat_id from the tag.',
  },
)

// --- reply tool: Claude calls this to send a message back -------------------
mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'The conversation to reply in' },
        text: { type: 'string', description: 'The message to send' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})

// --- permission relay: Claude Code (not Claude) calls this when a dialog opens
const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id: z.string(),
    tool_name: z.string(),
    description: z.string(),
    input_preview: z.string(),
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n` +
    `${params.input_preview}\n\n` +
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})

await mcp.connect(new StdioServerTransport())

// --- HTTP on :8788: GET /events streams outbound, POST routes inbound -------
const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i
let nextId = 1

Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  idleTimeout: 0,  // 유휴 SSE 스트림을 닫지 않음
  async fetch(req) {
    const url = new URL(req.url)

    // GET /events: curl -N으로 답장 및 프롬프트를 라이브 모니터링할 수 있는 SSE 스트림
    if (req.method === 'GET' && url.pathname === '/events') {
      const stream = new ReadableStream({
        start(ctrl) {
          ctrl.enqueue(': connected\n\n')  // curl에 즉시 출력되도록 함
          const emit = (chunk: string) => ctrl.enqueue(chunk)
          listeners.add(emit)
          req.signal.addEventListener('abort', () => listeners.delete(emit))
        },
      })
      return new Response(stream, {
        headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
      })
    }

    // 그 외 모든 요청은 인바운드: 먼저 발신자 검사
    const body = await req.text()
    const sender = req.headers.get('X-Sender') ?? ''
    if (!allowed.has(sender)) return new Response('forbidden', { status: 403 })

    // 채팅으로 처리하기 전 판정 형식 검사
    const m = PERMISSION_REPLY_RE.exec(body)
    if (m) {
      await mcp.notification({
        method: 'notifications/claude/channel/permission',
        params: {
          request_id: m[2].toLowerCase(),
          behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
        },
      })
      return new Response('verdict recorded')
    }

    // 일반 채팅: 채널 이벤트로 Claude에게 전달
    const chat_id = String(nextId++)
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: { content: body, meta: { chat_id, path: url.pathname } },
    })
    return new Response('ok')
  },
})
```

세 개의 터미널에서 판정 경로를 테스트합니다. 첫 번째는 `webhook.ts`를 실행하도록 [개발 플래그](#리서치-프리뷰-동안-테스트)로 시작된 Claude Code 세션입니다:

```bash theme={null}
claude --dangerously-load-development-channels server:webhook
```

두 번째 터미널에서는 Claude의 답장과 권한 프롬프트가 발송될 때 확인할 수 있도록 아웃바운드 측을 스트리밍합니다:

```bash theme={null}
curl -N localhost:8788/events
```

세 번째 터미널에서는 Claude가 명령어를 실행하도록 만드는 메시지를 보냅니다:

```bash theme={null}
curl -d "list the files in this directory" -H "X-Sender: dev" localhost:8788
```

파일 목록 검색은 읽기 전용이므로 Claude가 승인 없이 실행합니다. 권한 대화 상자는 Claude가 답변을 다시 보내기 위해 `reply` 도구를 호출할 때 열립니다. Claude Code 터미널에서 로컬 대화 상자가 열리고, 잠시 후 5자리 ID를 포함한 `mcp__webhook__reply` 프롬프트가 `/events` 스트림에 나타납니다. 원격 측에서 승인합니다:

```bash theme={null}
curl -d "yes <id>" -H "X-Sender: dev" localhost:8788
```

로컬 대화 상자가 닫히고 `reply` 도구가 실행되며 Claude의 답변이 스트림에 전송됩니다.

이 파일에 포함된 3가지 채널 전용 항목:

* **Capabilities** (`Server` 생성자 내부): `claude/channel`은 알림 수신기를 등록하고, `claude/channel/permission`은 권한 릴레이를 활성화하며, `tools`는 Claude가 답장 도구를 검색할 수 있도록 합니다.
* **아웃바운드 경로**: `reply` 도구 핸들러는 대화형 응답을 위해 Claude가 호출하는 것이고, `PermissionRequestSchema` 알림 핸들러는 권한 대화 상자가 열릴 때 Claude Code가 호출하는 것입니다. 둘 다 `/events`를 통해 전송하기 위해 `send()`를 호출하지만, 시스템의 서로 다른 부분에 의해 트리거됩니다.
* **HTTP 핸들러**: `GET /events`는 curl이 아웃바운드를 실시간으로 볼 수 있도록 SSE 스트림을 열어 둡니다. `POST`는 인바운드 요청이며 `X-Sender` 헤더로 제어됩니다. `yes <id>` 또는 `no <id>` 본문은 판정 알림으로 Claude Code로 이동하여 Claude에 도달하지 않으며, 그 외의 모든 것은 채널 이벤트로 Claude에게 전달됩니다.

## 플러그인으로 패키징

채널을 설치 및 공유 가능하도록 만들려면 채널을 [플러그인](/docs/en/plugins)으로 감싸서 [마켓플레이스](/docs/en/plugin-marketplaces)에 게시하세요. 사용자는 `/plugin install`을 통해 설치한 뒤 세션별로 `--channels plugin:<name>@<marketplace>`를 통해 활성화합니다.

자체 마켓플레이스에 게시된 채널은 [승인된 허용 목록](/docs/en/channels#supported-channels)에 포함되어 있지 않으므로 실행하려면 여전히 `--dangerously-load-development-channels`가 필요합니다. 기본 허용 목록은 Anthropic이 판단하여 관리하는 `claude-plugins-official`의 채널 플러그인입니다. [앱 내 제출 폼](/docs/en/plugins#submit-your-plugin-to-the-community-marketplace)을 사용하면 커뮤니티 마켓플레이스에 플러그인이 추가되지만 채널 허용 목록에는 포함되지 않습니다.

Anthropic 파트너 담당자와 작업 중인 경우 파트너에게 연락하여 공식 마켓플레이스 등록을 협의하세요. Team 및 Enterprise 플랜에서는 관리자가 기본 Anthropic 허용 목록을 대체하는 조직 자체 [`allowedChannelPlugins`](/docs/en/channels#restrict-which-channel-plugins-can-run) 목록에 해당 플러그인을 포함할 수 있습니다.

## 참고 항목

* [채널](/docs/en/channels): Telegram, Discord, iMessage 또는 fakechat 데모를 설치 및 사용하고 Team/Enterprise 조직에 대한 채널을 활성화하는 방법
* [작동하는 채널 구현 예시](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins): 페어링 흐름, 답장 도구, 파일 첨부가 포함된 완전한 서버 코드
* [MCP](/docs/en/mcp): 채널 서버가 구현하는 기본 프로토콜
* [플러그인](/docs/en/plugins): 사용자가 `/plugin install`로 설치할 수 있도록 채널을 패키징하는 방법
