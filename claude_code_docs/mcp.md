> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# MCP를 통해 Claude Code를 도구에 연결하기

> Model Context Protocol(MCP)을 사용하여 Claude Code를 사용자의 도구에 연결하는 방법을 알아봅니다.

Claude Code는 AI 도구 통합을 위한 오픈 소스 표준인 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction)을 통해 수백 개의 외부 도구 및 데이터 소스에 연결할 수 있습니다. MCP 서버는 Claude Code에 사용자 도구, 데이터베이스 및 API에 대한 액세스 권한을 부여합니다.

이슈 트래커나 모니터링 대시보드와 같은 다른 도구에서 채팅창으로 데이터를 복사하여 붙여넣고 있는 자신을 발견한다면 서버를 연결하세요. 연결되면 Claude는 붙여넣은 내용에 의존하지 않고 해당 시스템에서 직접 읽고 조치를 취할 수 있습니다.

첫 번째 서버를 연결하는 경우 단계별 안내가 포함된 [MCP 빠른 시작](/docs/en/mcp-quickstart)부터 시작하세요. 이 페이지는 전체 참조 문서입니다.

## MCP로 할 수 있는 작업

MCP 서버가 연결되면 Claude Code에 다음과 같이 요청할 수 있습니다:

* **이슈 트래커에서 기능 구현**: "JIRA 이슈 ENG-4521에 설명된 기능을 추가하고 GitHub에 PR을 생성해 줘."
* **모니터링 데이터 분석**: "ENG-4521에 설명된 기능의 사용량을 점검하기 위해 Sentry와 Statsig를 확인해 줘."
* **데이터베이스 쿼리**: "PostgreSQL 데이터베이스를 기반으로 ENG-4521 기능을 사용한 무작위 사용자 10명의 이메일을 찾아줘."
* **디자인 통합**: "Slack에 게시된 새 Figma 디자인을 바탕으로 표준 이메일 템플릿을 업데이트해 줘."
* **워크플로 자동화**: "이 10명의 사용자를 새 기능 피드백 세션에 초대하는 Gmail 임시 저장 메일을 생성해 줘."
* **외부 이벤트에 반응**: MCP 서버는 실행 중인 세션으로 메시지를 푸시하는 [채널](/docs/en/channels) 역할도 할 수 있으므로, 자리를 비운 동안 Telegram 메시지, Discord 채팅 또는 웹훅 이벤트에 Claude가 반응할 수 있습니다.

## MCP 서버 찾기 및 빌드

[Anthropic Directory](https://claude.ai/directory)에서 검토를 거친 커넥터를 살펴보세요. 디렉토리 커넥터는 Claude Code와 동일한 MCP 인프라를 사용하므로, 거기에 포함된 모든 원격 서버를 `claude mcp add`로 추가할 수 있습니다.

<Warning>
  연결하기 전에 각 서버를 신뢰할 수 있는지 검증하세요. 외부 콘텐츠를 가져오는 서버는 [프롬프트 주입 위험](/docs/en/security#protect-against-prompt-injection)에 노출될 수 있습니다.
</Warning>

자체 서버를 구축하려면 프로토콜 기본 사항에 대한 [MCP 서버 가이드](https://modelcontextprotocol.io/docs/develop/build-server) 및 인증, 테스트, 디렉토리 제출에 대한 [Claude 커넥터 구축 문서](https://claude.com/docs/connectors/building)를 참조하세요.

공식 [`mcp-server-dev` 플러그인](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/mcp-server-dev)을 통해 Claude가 대신 서버를 스캐폴딩(scaffold)하도록 할 수도 있습니다.

<Steps>
  <Step title="플러그인 설치">
    Claude Code 세션에서 다음을 실행합니다:

    ```
    /plugin install mcp-server-dev@claude-plugins-official
    ```

    Claude Code가 `Marketplace "claude-plugins-official" not found`를 보고하는 경우 `/plugin marketplace add anthropics/claude-plugins-official`로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 보고하면 로컬 사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로고침 후 설치를 재시도하세요. 설치가 완료되면 `/reload-plugins`를 실행하여 현재 세션에서 활성화하세요.
  </Step>

  <Step title="빌드 스킬 실행">
    ```
    /mcp-server-dev:build-mcp-server
    ```

    Claude가 사용 사례를 묻고 원격 HTTP 또는 로컬 stdio 서버를 스캐폴딩합니다.
  </Step>
</Steps>

## MCP 서버 설치

MCP 서버는 필요에 따라 여러 가지 방식으로 구성할 수 있습니다:

### 옵션 1: 원격 HTTP 서버 추가

원격 MCP 서버에 연결할 때 권한하는 방식은 HTTP 서버입니다. 이는 클라우드 기반 서비스에서 가장 널리 지원되는 트랜스포트(transport) 방식입니다.

```bash theme={null}
# 기본 구문
claude mcp add --transport http <name> <url>

# 실제 예시: Notion 연결
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Bearer 토큰이 포함된 예시
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

`.mcp.json`, `~/.claude.json`, 또는 `claude mcp add-json`을 통해 JSON으로 MCP 서버를 구성할 때 `type` 필드는 `http`에 대한 별칭(alias)으로 `streamable-http`를 수용합니다. MCP 명세는 이 트랜스포트에 대해 `streamable-http`라는 이름을 사용하므로 서버 문서에서 복사한 구성을 수정 없이 그대로 사용할 수 있습니다.

`url`은 있지만 `type`이 없는 JSON 항목은 구성 오류입니다. Claude Code가 `type`이 없는 항목을 stdio 서버로 읽어들이기 때문입니다. Claude Code는 해당 서버를 건너뛰고 `MCP server "<name>" has a "url" but no "type"; add "type": "http" (or "sse" / "ws") to this entry`를 보고합니다. v2.1.202 이전에는 이 잘못된 구성을 `command: expected string, received undefined`로 보고했습니다.

### 옵션 2: 원격 SSE 서버 추가

<Warning>
  SSE (Server-Sent Events) 트랜스포트는 권장되지 않습니다(deprecated). 가능한 경우 대신 HTTP 서버를 사용하세요.
</Warning>

```bash theme={null}
# 기본 구문
claude mcp add --transport sse <name> <url>

# 실제 예시: Asana 연결
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 인증 헤더가 포함된 예시
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: your-key-here"
```

### 옵션 3: 로컬 stdio 서버 추가

Stdio 서버는 머신의 로컬 프로세스로 실행됩니다. 직접적인 시스템 액세스나 커스텀 스크립트가 필요한 도구에 최적입니다.

Claude Code는 생선된 서버 환경의 `CLAUDE_PROJECT_DIR`을 프로젝트 루트로 설정하므로, 서버가 작업 디렉토리에 의존하지 않고 프로젝트 상대 경로를 확인(resolve)할 수 있습니다. 이는 훅(hook)이 `CLAUDE_PROJECT_DIR` 변수로 받는 것과 동일한 디렉토리입니다. Node의 `process.env.CLAUDE_PROJECT_DIR`이나 Python의 `os.environ["CLAUDE_PROJECT_DIR"]`처럼 서버 프로세스 내부에서 이를 읽어 사용할 수 있습니다.

`CLAUDE_PROJECT_DIR`은 고정된 프로젝트 루트로 세션 중간에 작업 디렉토리를 추가하거나 제거해도 변경되지 않습니다. 허용된 디렉토리 세트로 자체 파일 시스템 액세스를 제한하는 서버는 대신 MCP `roots/list` 요청을 구현해야 합니다. Claude Code는 세션의 시작 디렉토리에 `--add-dir`, `/add-dir` 또는 `additionalDirectories` 설정으로 부여한 [추가 작업 디렉토리](/docs/en/permissions#working-directories)를 더해 `roots/list`에 응답합니다. 해당 세트가 변경되면 Claude Code가 `notifications/roots/list_changed`를 보냅니다. v2.1.203 이전에는 `roots/list`가 시작 디렉토리만 반환했으며 Claude Code가 `notifications/roots/list_changed`를 보내지 않았습니다.

이 변수는 Claude Code 자체의 환경이 아닌 서버 환경에 설정되므로, 프로젝트 범위의 `.mcp.json` 항목이나 `~/.claude.json`의 로컬/사용자 범위 서버 항목의 `command` 또는 `args`에서 `${VAR}` 치환으로 이를 참조하려면 `${CLAUDE_PROJECT_DIR:-.}`와 같은 기본값이 필요합니다. 플러그인이 제공하는 MCP 구성은 `${CLAUDE_PROJECT_DIR}`을 직접 대체하므로 기본값이 필요하지 않습니다.

```bash theme={null}
# 기본 구문
claude mcp add [options] <name> -- <command> [args...]

# 실제 예시: Airtable 서버 추가
claude mcp add --env AIRTABLE_API_KEY=YOUR_KEY --transport stdio airtable \
  -- npx -y airtable-mcp-server
```

<Note>
  **중요: 서버 인수는 `--`로 구분하세요**

  stdio 서버의 경우 `--`(이중 대시)는 `--transport`, `--env`, `--scope`와 같은 Claude 자체 옵션을 서버를 실행하는 명령 및 인수와 구분합니다. `--` 뒤에 오는 모든 내용은 서버로 그대로 전달됩니다.

  예시:

  * `claude mcp add --transport stdio myserver -- npx server` → `npx server` 실행
  * `claude mcp add --env KEY=value --transport stdio myserver -- python server.py --port 8080` → `KEY=value` 환경 변수와 함께 `python server.py --port 8080` 실행

  `--`가 없으면 Claude Code는 위의 `--port`와 같은 서버 플래그를 자체 옵션으로 파싱하려고 시도합니다.

  `--env`는 여러 개의 `KEY=value` 쌍을 수용합니다. 서버 이름이 `--env` 바로 뒤에 오면 CLI가 그 이름을 또 다른 쌍으로 읽어 거부하므로 위 예시처럼 `--env`와 서버 이름 사이에 최소 하나 이상의 다른 옵션을 위치시키세요.
</Note>

### 옵션 4: 원격 WebSocket 서버 추가

WebSocket 서버는 영구적인 양방향 연결을 유지하므로, Claude에게 요청 없이 이벤트를 푸시하는 원격 MCP 서버에 적합합니다. 요청에만 응답하는 서버의 경우 HTTP가 OAuth 및 `claude mcp add --transport` 플래그를 지원하는 반면 WebSocket은 둘 다 지원하지 않으므로 대신 HTTP를 사용하세요.

`.mcp.json` 또는 `claude mcp add-json`으로 WebSocket 서버를 구성합니다:

```bash theme={null}
claude mcp add-json events-server \
  '{"type":"ws","url":"wss://mcp.example.com/socket","headers":{"Authorization":"Bearer YOUR_TOKEN"}}'
```

`type: "ws"` 항목은 `http`와 동일한 `url`, `headers`, `headersHelper`, `timeout`, `alwaysLoad` 필드를 수용합니다. 인증은 헤더 전용이므로 `headers`에 정적 토큰을 전달하거나 [`headersHelper`](#use-dynamic-headers-for-custom-authentication)로 연결 시점에 토큰을 생성하세요. `claude mcp add --transport` 플래그는 `ws`를 수용하지 않습니다.

### 서버 관리하기

구성되면 다음 명령어로 MCP 서버를 관리할 수 있습니다:

```bash theme={null}
# 구성된 모든 서버 목록 표시
claude mcp list

# 특정 서버의 상세 정보 가져오기
claude mcp get github

# 서버 제거
claude mcp remove github

# (Claude Code 내부에서) 서버 상태 확인
/mcp
```

승인을 기다리는 `.mcp.json` 파일의 프로젝트 범위 서버는 `claude mcp list` 및 `claude mcp get <name>`에서 ``⏸ Pending approval (run `claude` to approve)``로 표시됩니다. `claude`를 인터랙티브하게 실행하여 검토 및 승인하세요. `claude mcp get <name>`은 거부된 서버를 `✘ Rejected (see disabledMcpjsonServers in settings)`로 표시합니다.

v2.1.196부터 `claude mcp list` 및 `claude mcp get`은 작업 공간에서 `claude`를 실행하고 작업 공간 신뢰 대화 상자를 승인하여 작업 공간을 신뢰하기 전까지 저장소에 커밋되지 않은 설정 파일에서만 `.mcp.json` 승인 항목을 읽습니다. 복제된 저장소는 자체 서버를 승인할 수 없습니다: 프로젝트의 `.claude/settings.json`에 커밋된 [`enableAllProjectMcpServers` 또는 `enabledMcpjsonServers`](/docs/en/settings#available-settings)는 신뢰할 수 없는 폴더에서 무시되며, 서버는 연결되어 헬스 체크를 받는 대신 `⏸ Pending approval` 상태에 머물러 있습니다.

이러한 출처의 승인은 신뢰할 수 없는 폴더에서도 여전히 적용됩니다:

* 사용자 `~/.claude/settings.json`
* 관리형 설정 (managed settings)
* `--settings`로 전달된 설정

추적되지 않은(untracked) `.claude/settings.local.json` 파일의 승인도 적용되지만, 해당 폴더 또는 상위 디렉토리 중 하나에 대한 신뢰 대화 상자를 승인한 후에만 적용됩니다. Claude Code가 git을 실행하여 파일이 추적되는지 확인하고 신뢰할 수 있는 폴더에서만 해당 확인을 수행하기 때문입니다. 한 번도 신뢰하지 않은 폴더에서는 해당 폴더가 사용자의 자체 구성 홈(홈 디렉토리, 또는 `.claude`를 [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars)로 설정한 디렉토리)이 아닌 한 승인이 신뢰 대화 상자를 기다립니다. v2.1.207 이전에는 추적되지 않은 `.claude/settings.local.json`이 신뢰한 적 없는 폴더에서도 서버를 승인했었습니다.

설정 파일의 `disabledMcpjsonServers` 항목은 계속해서 서버를 거부합니다.

`/mcp` 패널은 연결된 각 서버 옆에 도구 개수를 표시하고, 도구 기능을 알리지만 도구를 노출하지 않는 서버를 표시합니다.

구성에 빈 `url`이 포함된 원격 서버는 `/mcp`, `claude mcp list`, 그리고 [`/plugin`](/docs/en/plugins) 관리자에서 `not configured`로 표시되며 Claude Code는 이에 대한 연결을 시도하지 않습니다. 플러그인이 나중에 구성할 커넥터를 위해 이와 같은 더미 항목을 포함할 수 있으므로 Claude Code가 이를 오류나 설정 문제로 보고하지 않습니다. `/mcp`에서 해당 서버의 상세 보기에는 `No URL configured for this server`라고 표시됩니다; 연결하려면 항목의 `url`을 설정하세요. v2.1.208 이전에는 Claude Code가 빈 `url`을 재연결 프롬프트가 포함된 구성 문제로 보고했습니다.

요청에 아직 백그라운드에서 연결 중인 서버의 도구가 필요한 경우, Claude는 계속 진행하기 전에 해당 서버를 기다립니다. 기본적으로 활성화되어 있는 [도구 검색](#scale-with-mcp-tool-search)을 사용하면 `ToolSearch` 호출 내부에서 대기가 일어납니다. Google Cloud's Agent Platform, 커스텀 `ANTHROPIC_BASE_URL`, 또는 `ENABLE_TOOL_SEARCH=false`와 같이 도구 검색이 없는 구성에서는 Claude가 대신 `WaitForMcpServers` 도구를 사용합니다.

일부 서버 이름은 Claude Code의 내장 서버용으로 예약되어 있습니다: `workspace`, `claude-in-chrome`, `computer-use`, `Claude Preview`, `Claude Browser`. 구성에 예약된 이름으로 서버를 정의하면 Claude Code가 로드 시점에 이를 건너뛰고 이름을 변경하라는 경고를 표시합니다. `claude mcp add`는 예약된 이름을 오류와 함께 거부합니다.

`Claude Preview` 및 `Claude Browser`는 모두 [Claude Code 데스크톱 앱의 미리보기 창](/docs/en/desktop#preview-your-app)이 사용하는 내장 서버의 이름입니다. v2.1.205 이전에는 `Claude Browser`가 예약되어 있지 않아 사용자 정의 서버가 해당 이름으로 등록될 수 있었습니다.

### 동적 도구 업데이트

Claude Code는 MCP `list_changed` 알림을 지원하여 끊고 다시 연결할 필요 없이 MCP 서버가 사용 가능한 도구, 프롬프트, 리소스를 동적으로 업데이트할 수 있도록 합니다. MCP 서버가 `list_changed` 알림을 보내면 Claude Code가 해당 서버에서 사용 가능한 기능을 자동으로 새로고침합니다.

새로고침 요청이 실패하는 경우 Claude Code는 나중의 새로고침이 성공할 때까지 서버의 이전에 탐색된 도구, 프롬프트, 리소스를 유지합니다. v2.1.214 이전에는 새로고침 중 일시적인 오류가 발생하면 서버의 도구, 프롬프트, 리소스가 빈 목록으로 대체되었습니다.

### 자동 재연결

HTTP 또는 SSE 서버가 세션 중간에 연결 해제되면 Claude Code는 지수 백오프(exponential backoff) 방식으로 자동 재연결합니다: 1초 지연으로 시작하여 매번 2배로 증가하며 최대 5회 시도합니다. 재연결이 진행되는 동안 서버는 `/mcp`에서 보류 상태로 표시됩니다. 5회 시도가 실패하면 서버가 실패로 표시되며 `/mcp`에서 수동으로 재시도할 수 있습니다. Stdio 서버는 로컬 프로세스이므로 자동으로 재연결되지 않습니다.

시작 시 HTTP 또는 SSE 서버가 초기 연결에 실패할 때도 동일한 백오프가 적용됩니다. v2.1.121부터 Claude Code는 5xx 응답, 연결 거부, 타임아웃과 같은 일시적 오류 발생 시 초기 연결을 최대 3회 재시도한 후에도 연결할 수 없으면 서버를 실패로 표시합니다. 인증 및 404(Not Found) 오류는 해결을 위해 구성 변경이 필요하므로 재시도되지 않습니다.

구성된 서버가 연결에 실패하면 Claude Code는 연결에 실패한 서버와 연결 오류를 Claude에게 알립니다([도구 검색](#scale-with-mcp-tool-search) 결과에서 일치하는 도구를 찾지 못한 경우 포함). 따라서 Claude는 응답에서 연결 실패를 보고합니다. 기본적으로 활성화된 [도구 검색](#scale-with-mcp-tool-search)이 필요합니다. 커스텀 `ANTHROPIC_BASE_URL`, `ENABLE_TOOL_SEARCH=false`, 또는 도구 검색을 지원하지 않는 모델, 그리고 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 환경 등 도구 검색이 없는 구성에서는 Claude Code가 실패한 서버 연결을 Claude에게 보고하지 않습니다. v2.1.205 이전에는 Claude Code가 연결 오류를 Claude에게 전달하지 않아 Claude가 실패한 서버의 도구가 아예 구성되지 않은 것처럼 응답할 수 있었습니다.

v2.1.191부터 성공적인 연결 후 실행되는 기능 탐색 요청(`tools/list`, `prompts/list`, `resources/list` 등)도 짧은 백오프와 함께 일시적인 네트워크 및 서버 오류를 최대 3회 재시도합니다. 인증 오류, 4xx 응답, 요청 타임아웃은 재시도되지 않습니다.

### 채널을 통한 푸시 메시지

MCP 서버는 세션에 직접 메시지를 푸시할 수도 있어 CI 결과, 모니터링 알림, 채팅 메시지와 같은 외부 이벤트에 Claude가 반응할 수 있습니다. 이를 활성화하려면 서버가 `claude/channel` 기능을 선언해야 하며 시작 시 `--channels` 플래그로 옵트인합니다. 공식 지원되는 채널을 사용하려면 [채널](/docs/en/channels)을, 직접 구축하려면 [채널 참조](/docs/en/channels-reference)를 확인하세요.

<Tip>
  팁:

  * `-s` 또는 `--scope` 플래그를 사용하여 구성이 저장되는 위치를 지정하세요:
    * `local` (기본값): 현재 프로젝트에서 본인만 사용 가능. 구버전에서는 이 스코프를 `project`라 불렀음
    * `project`: `.mcp.json` 파일을 통해 프로젝트의 모든 사람과 공유됨
    * `user`: 모든 프로젝트에서 본인이 사용 가능. 구버전에서는 이 스코프를 `global`이라 불렀음
  * `-e` 또는 `--env` 플래그로 환경 변수 설정 (예: `-e KEY=value`)
  * `--transport` 및 `--header` 플래그는 각각 `-t` 및 `-H` 축약형도 수용함
  * `MCP_TIMEOUT` 환경 변수를 사용하여 MCP 서버 시작 타임아웃 구성 (예: `MCP_TIMEOUT=10000 claude`는 10초 타임아웃 설정)
  * 서버의 `.mcp.json` 항목에 밀리초 단위의 `timeout` 필드를 추가하여 서버별 도구 실행 타임아웃을 설정 (예: 10분의 경우 `"timeout": 600000`). 이는 해당 서버에 대해서만 `MCP_TOOL_TIMEOUT` 환경 변수를 재정의함
  * Claude Code는 MCP 도구 출력이 10,000 토큰을 초과할 때 경고를 표시하고 기본적으로 출력을 25,000 토큰으로 제한함. 제한을 높이려면 `MAX_MCP_OUTPUT_TOKENS` 환경 변수를 설정 (예: `MAX_MCP_OUTPUT_TOKENS=50000`); 경고 임계값은 고정되어 있음. [MCP 출력 제한 및 경고](#mcp-output-limits-and-warnings) 참조
  * OAuth 2.0 인증이 필요한 원격 서버로 인증하려면 `/mcp` 사용
</Tip>

서버별 `timeout`은 도구 호출당 하드 경과 시간(wall-clock) 제한이며 서버의 진행 상황 알림으로 연장되지 않습니다. 1000 미만의 값은 무시되어 `MCP_TOOL_TIMEOUT`으로 떨어지거나, 해당 변수가 설정되지 않은 경우 기본값인 약 28시간으로 떨어집니다. HTTP, SSE, 또는 [claude.ai 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai) 서버의 경우 서버의 첫 번째 응답 바이트까지 각 요청을 포함하는 두 번째 요청별 타이머도 있습니다. 해당 타이머는 서버별 `timeout`이나 `MCP_TOOL_TIMEOUT`을 설정하지 않는 한 60초입니다; 둘 중 하나를 60초 이상으로 설정하면 요청별 타이머가 해당 값으로 높아지고, 더 낮은 값은 단축시키지 않으며, 설정되지 않은 `MCP_TOOL_TIMEOUT`의 28시간 기본값은 이 타이머에 공급되지 않습니다. Stdio 및 WebSocket 서버에는 요청별 타이머가 없습니다. v2.1.162 이전에는 1000 미만의 값이 대신 1초로 제한(floored)되었었습니다.

최소 1000 이상의 서버별 `timeout`은 아래 설명된 유휴 타임아웃(idle timeout)의 최저 한도로도 작동합니다: Claude Code는 서버별 `timeout`보다 일찍 유휴 상태로 인해 해당 서버의 도구 호출을 중단하지 않습니다. Claude Code v2.1.203 이상이 필요합니다.

유휴 기간 동안 응답과 진행 알림을 모두 보내지 않는 MCP 서버 도구 호출은 경과 시간 제한을 기다리는 대신 오류와 함께 중단됩니다. 유휴 타임아웃은 Claude Code v2.1.187 이상이 필요합니다. IDE 서버 및 SDK 인프로세스 서버를 제외한 모든 서버 유형에 적용됩니다. 유휴 기간은 HTTP, SSE, WebSocket 및 [claude.ai 커넥터](#use-mcp-servers-from-claude-ai) 서버의 경우 기본 5분, stdio 서버의 경우 30분입니다. v2.1.203 이전에는 stdio 서버가 유휴 타임아웃에서 면제되었습니다.

유휴 기간을 변경하려면 [`CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT`](/docs/en/env-vars) 환경 변수를 밀리초 단위로 설정하거나, 검사를 비활성화하려면 `0`으로 설정하세요.

이 이러한 타임아웃은 호출이 실행될 수 있는 기간을 제한하는 것이며, 세션을 차단하는 기간과 항상 일치하지는 않습니다: Claude Code v2.1.212 이상에서는 메인 대화 호출이 2분을 초과하면 먼저 백그라운드 작업으로 이동합니다. [긴 도구 호출의 자동 백그라운드 전환](#automatic-backgrounding-of-long-tool-calls)을 참조하세요.

### 긴 도구 호출의 자동 백그라운드 전환

2분이 지나도 계속 실행 중인 메인 대화의 MCP 도구 호출은 세션을 차단하는 대신 백그라운드 작업으로 이동합니다. Claude는 즉시 작업 ID를 받고 작업을 계속하며, 호출이 완료되면 작업 알림으로 결과가 도착합니다. 자동 백그라운드 전환은 Claude Code v2.1.212 이상이 필요합니다.

작업은 [`/tasks`](/docs/en/commands#all-commands)에 표시되어 거기서 중지할 수도 있으며, 세션 종료 시 유지되지 않습니다. 호출이 백그라운드에서 실행되는 동안에도 호출당 제한은 계속 적용됩니다: 서버별 `timeout` 또는 [`MCP_TOOL_TIMEOUT`](/docs/en/env-vars)에 의해 설정된 경과 시간 제한, 그리고 [`CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT`](/docs/en/env-vars)에 의해 설정된 유휴 타임아웃.

임계값을 변경하려면 [`CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS`](/docs/en/env-vars) 환경 변수를 밀리초 단위로 설정하거나, 자동 백그라운드 전환을 끄려면 `0`으로 설정하세요. `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`를 `1`로 설정해도 다른 모든 백그라운드 작업 기능과 함께 이 기능이 꺼집니다.

일부 호출은 백그라운드로 이동하지 않습니다:

* [subagent](/docs/en/sub-agents)의 호출; Claude Code는 메인 대화 호출만 백그라운드로 이동시킵니다
* IDE 서버 호출
* 일회성 실행이 결과가 도착하기 전에 끝날 수 있으므로 `CLAUDE_AUTO_BACKGROUND_TASKS`가 `1`로 설정되지 않은 한 [비대화형 모드](/docs/en/headless)에서의 호출

열려 있는 [도구 정보 요청 대화 상자](#respond-to-mcp-elicitation-requests)를 기다리는 호출은 대화 상자가 열려 있는 동안 백그라운드로 전환되지 않습니다. 서버가 사용자 입력을 기다리는 중이지 느린 것이 아니므로 Claude Code는 대화 상자가 닫힐 때까지 전환을 연기합니다.

### 플러그인 제공 MCP 서버

[플러그인](/docs/en/plugins)은 MCP 서버를 번들로 포함할 수 있어 플러그인이 활성화될 때 도구와 통합 기능을 자동으로 제공할 수 있습니다. 플러그인 MCP 서버는 사용자가 구성한 서버와 동일하게 작동합니다.

**플러그인 MCP 서버 작동 방식**:

* 플러그인은 플러그인 루트의 `.mcp.json` 또는 `plugin.json` 내부에 MCP 서버를 정의합니다.
* 플러그인이 활성화되면 해당 MCP 서버가 자동으로 시작됩니다.
* 플러그인 MCP 도구는 수동으로 구성된 MCP 도구와 함께 나타납니다.
* 플러그인 서버는 `/mcp` 명령이 아닌 플러그인 설치를 통해 관리됩니다.

**플러그인 MCP 구성 예시**:

플러그인 루트의 `.mcp.json` 내부:

```json theme={null}
{
  "mcpServers": {
    "database-tools": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_URL": "${DB_URL}"
      }
    }
  }
}
```

또는 `plugin.json` 내부에 포함:

```json theme={null}
{
  "name": "my-plugin",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

**플러그인 MCP 기능**:

* **자동 라이프사이클**: 서버는 다음 시점에 자동으로 연결 및 연결 해제됩니다:
  * 세션 시작 시 활성화된 플러그인의 서버가 자동으로 연결됨
  * 세션 중에 플러그인을 활성화하거나 비활성화한 경우 `/reload-plugins`를 실행하여 해당 MCP 서버를 연결하거나 연결 해제
  * [웹 세션](/docs/en/claude-code-on-the-web)에서 유휴 세션이 깨어난 직후와 같이 아직 연결되지 않은 플러그인 서버로 MCP 호출이 들어오면 요청 시 서버를 시작하고 연결될 때까지 대기함. v2.1.211 이전에는 웹 세션의 플러그인 서버가 다음 메시지가 턴을 시작할 때만 재연결되어 그때까지 유휴 세션이 깨어난 후의 MCP 호출이 실패했었음
* **경로 치환 문자**: `${CLAUDE_PLUGIN_ROOT}`는 플러그인의 설치 디렉토리로, `${CLAUDE_PLUGIN_DATA}`는 [영구 데이터](/docs/en/plugins-reference#persistent-data-directory) 디렉토리로, `${CLAUDE_PROJECT_DIR}`은 고정된 프로젝트 루트로 치환됩니다. 대체가 적용되는 위치:
  * `stdio` 서버: `command`, `args`, `env`
  * `http`, `sse`, `ws` 서버: `url`, `headers`, `headersHelper`. v2.1.195 이전에는 `headersHelper`가 문자 그대로의 치환 문자를 전달했었음
* **사용자 환경 액세스**: 수동으로 구성된 서버와 동일한 환경 변수에 액세스 가능
* **다중 트랜스포트 유형**: stdio, SSE, HTTP, WebSocket 트랜스포트 지원 (단, 서버에 따라 트랜스포트 지원 여부가 다를 수 있음)

**플러그인 MCP 서버 보기**:

```bash theme={null}
# Claude Code 내부에서 플러그인 서버를 포함한 모든 MCP 서버 표시
/mcp
```

플러그인 서버는 목록에 플러그인에서 제공됨을 나타내는 표시와 함께 나타납니다.

**플러그인 MCP 도구 이름**:

플러그인 번들 MCP 서버의 도구는 호출 가능한 이름에 플러그인 이름과 서버 키를 모두 포함합니다. 전체 형식은 `mcp__plugin_<plugin-name>_<server-name>__<tool-name>`이며, `A-Z`, `a-z`, `0-9`, `_`, `-` 이외의 모든 문자는 `_`로 대체됩니다. `my-plugin`이라는 이름의 플러그인에 번들된 `database-tools` 서버의 경우 `query` 도구는 다음과 같이 호출 가능합니다:

```
mcp__plugin_my-plugin_database-tools__query
```

[권한 규칙](/docs/en/permissions), 스킬의 `allowed-tools` 목록, [subagent의 `tools` 필드](/docs/en/sub-agents#available-tools), 또는 [훅 매처](/docs/en/hooks#match-mcp-tools)에서 도구를 참조할 때 이 전체 이름을 사용하세요. `mcp__database-tools__.*`와 같이 순수한 서버 키로 작성된 훅 매처는 플러그인 번들 서버에 대해 절대 실행되지 않습니다.

서버 자체는 `plugin:<plugin-name>:<server-name>`과 같이 범위가 지정된 이름으로 등록됩니다(예: `plugin:my-plugin:database-tools`). [`mcp_tool` 훅의 `server` 필드](/docs/en/hooks#mcp-tool-hook-fields)와 같이 구성된 서버 이름이 예상되는 위치에 해당 이름을 사용하세요.

**플러그인 MCP 서버의 장점**:

* **번들 배포**: 도구와 서버가 함께 패키징됨
* **자동 설정**: 수동 MCP 구성이 필요 없음
* **팀 일관성**: 플러그인을 설치하면 누구나 동일한 도구를 얻음

MCP 서버를 플러그인과 함께 패키징하는 방법에 대한 자세한 내용은 [플러그인 구성 요소 참조](/docs/en/plugins-reference#mcp-servers)를 확인하세요.

## MCP 설치 스코프

MCP 서버는 세 가지 스코프에서 구성할 수 있습니다. 선택한 스코프에 따라 서버가 로드되는 프로젝트와 팀과의 구성 공유 여부가 결정됩니다. 관리자는 [관리형 구성](#managed-mcp-configuration)을 통해 기업 차원에서 서버를 배포할 수도 있습니다.

| 스코프                    | 로드 대상           | 팀 공유 여부              | 저장 위치                   |
| ------------------------- | -------------------- | ------------------------ | --------------------------- |
| [Local](#local-scope)     | 현재 프로젝트만     | 아니오                   | `~/.claude.json`            |
| [Project](#project-scope) | 현재 프로젝트만     | 예 (버전 관리를 통해)    | 프로젝트 루트의 `.mcp.json` |
| [User](#user-scope)       | 본인의 모든 프로젝트 | 아니오                   | `~/.claude.json`            |

### Local 스코프

Local 스코프가 기본값입니다. Local 스코프의 서버는 추가한 프로젝트에서만 로드되며 본인에게만 비공개로 유지됩니다. Claude Code는 해당 프로젝트 경로 아래의 `~/.claude.json`에 이를 저장하므로 동일한 서버가 다른 프로젝트에 나타나지 않습니다. 개인 개발용 서버, 실험적 구성 또는 버전 관리에 포함하고 싶지 않은 자격 증명이 있는 서버에는 Local 스코프를 사용하세요.

<Note>
  MCP 서버의 "Local 스코프"라는 용어는 일반 로컬 설정과 다릅니다. MCP Local 스코프 서버는 `~/.claude.json`(홈 디렉토리)에 저장되는 반면, 일반 로컬 설정은 `.claude/settings.local.json`(프로젝트 디렉토리)을 사용합니다. 설정 파일 위치에 대한 자세한 내용은 [설정](/docs/en/settings#settings-files)을 참조하세요.
</Note>

```bash theme={null}
# Local 스코프 서버 추가 (기본값)
claude mcp add --transport http stripe https://mcp.stripe.com

# 명시적으로 Local 스코프 지정
claude mcp add --transport http stripe --scope local https://mcp.stripe.com
```

이 명령은 `~/.claude.json` 내부의 현재 프로젝트 항목에 서버를 기록합니다. 아래 예시는 `/path/to/your/project`에서 실행했을 때의 결과를 보여줍니다:

```json theme={null}
{
  "projects": {
    "/path/to/your/project": {
      "mcpServers": {
        "stripe": {
          "type": "http",
          "url": "https://mcp.stripe.com"
        }
      }
    }
  }
}
```

### Project 스코프

Project 스코프 서버는 프로젝트 루트 디렉토리의 `.mcp.json` 파일에 구성을 저장하여 팀 협업을 가능하게 합니다. 이 파일은 버전 관리에 커밋되도록 설계되어 모든 팀원이 동일한 MCP 도구 및 서비스에 액세스할 수 있도록 보장합니다. Project 스코프 서버를 추가하면 Claude Code가 적절한 구성 구조로 이 파일을 자동으로 생성하거나 업데이트합니다.

```bash theme={null}
# Project 스코프 서버 추가
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp
```

생성된 `.mcp.json` 파일은 표준화된 형식을 따릅니다:

```json theme={null}
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

보안상의 이유로 Claude Code는 `.mcp.json` 파일의 Project 스코프 서버를 사용하기 전에 승인을 요청합니다. 이러한 승인 선택을 재설정해야 하는 경우 `claude mcp reset-project-choices` 명령을 사용하세요.

### User 스코프

User 스코프 서버는 `~/.claude.json`에 저장되며 프로젝트 간 액세스 가능성을 제공하여 사용자 계정에는 비공개로 유지되면서 머신의 모든 프로젝트에서 사용할 수 있도록 합니다. 이 스코프는 개인 유틸리티 서버, 개발 도구, 또는 여러 프로젝트에서 자주 사용하는 서비스에 유용합니다.

```bash theme={null}
# User 서버 추가
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 스코프 계층 구조 및 우선순위

동일한 서버가 둘 이상의 위치에 정의되어 있는 경우, Claude Code는 가장 우선순위가 높은 출처의 정의를 사용하여 한 번만 연결합니다. 해당 출처의 전체 서버 항목이 사용되며 필드가 스코프 간에 병합되지 않습니다.

1. Local 스코프
2. Project 스코프
3. User 스코프
4. [플러그인 제공 서버](/docs/en/plugins)
5. [claude.ai 커넥터](#use-mcp-servers-from-claude-ai)

세 가지 스코프는 이름을 기준으로 중복을 일치시킵니다. 플러그인과 커넥터는 엔드포인트를 기준으로 일치시키므로 위 서버와 동일한 URL이나 명령을 가리키는 서버는 중복으로 처리됩니다.

### .mcp.json의 환경 변수 치환

Claude Code는 `.mcp.json` 파일에서 환경 변수 치환을 지원하여 팀이 구성을 공유하면서 머신별 경로 및 API 키와 같은 민감한 값에 대해 유연성을 유지할 수 있도록 합니다.

**지원되는 구문:**

* `${VAR}`: 환경 변수 `VAR`의 값으로 치환됨
* `${VAR:-default}`: `VAR`이 설정되어 있으면 해당 값으로, 설정되어 있지 않으면 `default`로 치환됨

**치환 위치:**
환경 변수는 다음 위치에서 치환할 수 있습니다:

* `command`: 서버 실행 파일 경로
* `args`: 커맨드라인 인수
* `env`: 서버에 전달되는 환경 변수
* `url`: HTTP 서버 유형의 URL
* `headers`: HTTP 서버 인증용 헤더

**변수 치환 예시:**

```json theme={null}
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

참조된 환경 변수가 설정되어 있지 않고 기본값도 없는 경우에도 구성은 로드됩니다: Claude Code가 `claude mcp list` 출력에 해당 서버에 대한 변수 누락 경고를 보고하고 치환되지 않은 `${VAR}` 텍스트를 그대로 사용합니다. 의도한 값으로 서버가 시작되도록 변수를 설정하거나 `:-default` 폴백을 추가하세요.

## 실용 예시

### 예시: Sentry로 오류 모니터링

```bash theme={null}
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

Sentry 계정으로 인증합니다:

```text theme={null}
/mcp
```

그런 다음 프로덕션 문제를 디버깅합니다:

```text theme={null}
지난 24시간 동안 가장 많이 발생한 오류는 무엇인가요?
```

```text theme={null}
오류 ID abc123에 대한 스택 트레이스를 보여줘
```

```text theme={null}
어떤 배포에서 이러한 새 오류들이 도입되었나요?
```

### 예시: 코드 리뷰를 위해 GitHub에 연결

GitHub의 원격 MCP 서버는 헤더로 전달되는 GitHub 개인 액세스 토큰(PAT)으로 인증합니다. 토큰을 받으려면 [GitHub 토큰 설정](https://github.com/settings/personal-access-tokens)을 열고, Claude가 작업할 저장소에 대한 액세스 권한이 있는 세밀한(fine-grained) 새 토큰을 생성한 후 서버를 추가하세요:

```bash theme={null}
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer YOUR_GITHUB_PAT"
```

`YOUR_GITHUB_PAT`을 개인 액세스 토큰으로 대체하세요. `claude mcp add` 명령은 자격 증명을 검증하지 않고 구성을 저장하므로 더미 값도 수용되지만 나중에 서버 연결에 실패합니다. 연결을 검증하려면 `/mcp`를 실행하여 서버가 `connected`로 표시되는지 확인하세요. 잘못된 자격 증명을 가진 서버는 `failed`로 표시됩니다.

그런 다음 GitHub으로 작업하세요:

```text theme={null}
PR #456을 검토하고 개선 사항을 제안해 줘
```

```text theme={null}
방금 발견한 버그에 대해 새 이슈를 생성해 줘
```

```text theme={null}
나에게 할당된 열린 PR을 모두 보여줘
```

### 예시: PostgreSQL 데이터베이스 쿼리

```bash theme={null}
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

그런 다음 데이터베이스를 자연스럽게 쿼리하세요:

```text theme={null}
이번 달 우리 총 매출은 얼마인가요?
```

```text theme={null}
orders 테이블의 스키마를 보여줘
```

```text theme={null}
90일 동안 구매하지 않은 고객을 찾아줘
```

## 원격 MCP 서버 인증

많은 클라우드 기반 MCP 서버는 인증을 필요로 합니다. Claude Code는 안전한 연결을 위해 OAuth 2.0을 지원합니다.

Claude Code는 서버가 `401 Unauthorized` 또는 `403 Forbidden`으로 응답할 때 원격 서버에 인증이 필요한 것으로 표시합니다. 로그인하지 않은 서버의 경우 두 상태 코드 모두 `/mcp`에서 이를 표시하여 OAuth 흐름을 완료할 수 있도록 합니다.

이미 로그인한 OAuth 서버에 대한 요청이 `401 Unauthorized`를 반환하면 Claude Code가 저장된 토큰을 새로고침하고 다시 연결하여 요청을 1회 재시도합니다. 해당 재시도도 실패하는 경우에만 `/mcp`에서 서버에 표시를 남깁니다. v2.1.206 이전에는 네트워크 오류와 같이 일시적인 이유로 토큰 새로고침이 실패하면 리프레시 토큰이 여전히 유효함에도 OAuth 서버가 세션의 나머지 기간 동안 인증 필요 상태로 표시되었었습니다.

v2.1.195부터 서버가 저장된 리프레시 토큰을 거부하여 토큰 새로고침이 실패하면 Claude Code가 즉시 `/mcp`를 가리키는 안내를 표시합니다. 거기서 연결된 서버 메뉴가 'Re-authenticate' 옵션을 제공하므로 다음 도구 호출이 실패하기 전에 다시 로그인할 수 있습니다.

인증 서버를 가리키는 `WWW-Authenticate` 헤더를 반환하는 커스텀 서버는 다른 원격 서버와 동일한 자동 탐색을 수행합니다.

Claude Code는 하나 이상의 구성된 서버에 인증이 필요할 때 시작 안내도 표시하므로 어떤 서버에 로그인이 필요한지 확인하기 위해 `/mcp`를 열 필요가 없습니다. 이 안내는 Claude Code v2.1.193 이상이 필요합니다. Claude Code 내에서 로그인할 수 있는 서버만 카운트합니다. v2.1.218 이전에는 claude.ai 설정에서만 연결할 수 있는 연결되지 않은 [claude.ai 커넥터](#use-mcp-servers-from-claude-ai)도 카운트했었습니다.

비대화형 모드에는 `/mcp` 패널이 없으므로 Claude Code가 대신 OAuth 흐름을 실행할 수 없습니다. v2.1.196부터 기본적으로 활성화되어 있는 [도구 검색](#scale-with-mcp-tool-search)을 사용한 `claude -p` 또는 Agent SDK 실행 중에 구성된 서버에 인증이 필요한 경우, Claude Code는 권한을 부여할 때까지 서버의 도구를 사용할 수 없음을 Claude에게 알립니다. 그러면 Claude가 서버가 구성되지 않은 것처럼 응답하는 대신 로그인이 필요한 서버의 이름을 지정할 수 있습니다. 인터랙티브 세션에서 `/mcp` 또는 `claude mcp login <name>`으로 로그인을 완료하세요.

서버에 `headers.Authorization`을 구성했으나 서버가 해당 헤더를 거부하는 경우 Claude Code는 OAuth로 떨어지는 대신 연결 실패로 보고합니다. 토큰이 MCP 엔드포인트에 유효한지 확인하거나 헤더를 제거하여 OAuth 흐름을 사용하세요.

<Steps>
  <Step title="인증이 필요한 서버 추가">
    예시:

    ```bash theme={null}
    claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
    ```
  </Step>

  <Step title="Claude Code 내부에서 /mcp 명령 사용">
    Claude Code에서 다음 명령을 사용합니다:

    ```text theme={null}
    /mcp
    ```

    그런 다음 브라우저의 단계를 따라 로그인합니다.
  </Step>
</Steps>

<Tip>
  팁:

  * 인증 토큰은 안전하게 저장되고 자동으로 새로고침됨
  * 액세스를 회수하려면 `/mcp` 메뉴에서 "Clear authentication" 사용
  * 브라우저가 자동으로 열리지 않으면 제공된 URL을 복사하여 수동으로 열기
  * 인증 후 브라우저 리다이렉트가 연결 오류로 실패하면 브라우저 주소창의 전체 콜백 URL을 Claude Code에 나타나는 URL 프롬프트에 붙여넣기
  * OAuth 인증은 HTTP 서버에서 작동함
</Tip>

### 커맨드라인에서 인증하기

v2.1.186부터 `claude mcp login <name>`은 세션 내부에서 `/mcp` 패널을 열 필요 없이 셸에서 직접 구성된 서버의 OAuth 흐름을 실행합니다.

```bash theme={null}
claude mcp login sentry
```

나중에 저장된 자격 증명을 지우려면 `claude mcp logout <name>`을 실행하세요.

v2.1.191부터 이 명령은 SSH 세션 중이거나 디스플레이 서버가 없는 Linux 등 로컬 브라우저를 사용할 수 없는 경우를 감지하여 브라우저를 열려고 시도하는 대신 인증 URL을 출력합니다. 로컬 머신에서 해당 URL을 연 다음 브라우저 주소창의 전체 리다이렉트 URL을 다시 프롬프트에 붙여넣으세요. 이 명령은 붙여넣기 단계를 위해 대화형 터미널이 필요하므로 `ssh -t`로 연결하세요. 로컬 브라우저가 감지되더라도 URL 프롬프트를 강제하려면 `--no-browser`를 전달하세요.

```bash theme={null}
claude mcp login sentry --no-browser
```

### 고정 OAuth 콜백 포트 사용

일부 MCP 서버는 사전에 등록된 특정 리다이렉트 URI를 요구합니다. 기본적으로 Claude Code는 OAuth 콜백용으로 사용 가능한 임의의 포트를 선택합니다. `http://localhost:PORT/callback` 형식의 사전 등록된 리다이렉트 URI와 일치하도록 포트를 고정하려면 `--callback-port`를 사용하세요.

`--callback-port`를 단독으로(동적 클라이언트 등록과 함께) 사용하거나 `--client-id`(사전 구성된 자격 증명과 함께)와 함께 사용할 수 있습니다.

```bash theme={null}
# 동적 클라이언트 등록과 함께 고정 콜백 포트 사용
claude mcp add --transport http \
  --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### 사전 구성된 OAuth 자격 증명 사용

일부 MCP 서버는 동적 클라이언트 등록(Dynamic Client Registration)을 통한 자동 OAuth 설정을 지원하지 않습니다. "Incompatible auth server: does not support dynamic client registration"과 같은 오류가 보이면 서버가 사전 구성된 자격 증명을 요구하는 것입니다. Claude Code는 동적 클라이언트 등록 대신 CIMD(Client ID Metadata Document)를 사용하는 서버도 지원하며 이를 자동으로 탐색합니다. 자동 탐색이 실패하는 경우 서버의 개발자 포털을 통해 OAuth 앱을 먼저 등록한 다음 서버를 추가할 때 자격 증명을 제공하세요.

<Steps>
  <Step title="서버에 OAuth 앱 등록">
    서버의 개발자 포털을 통해 앱을 생성하고 클라이언트 ID와 클라이언트 시크릿을 확인합니다.

    많은 서버가 리다이렉트 URI도 요구합니다. 그런 경우 포트를 선택하고 `http://localhost:PORT/callback` 형식으로 리다이렉트 URI를 등록하세요. 다음 단계에서 `--callback-port`에 동일한 포트를 사용하세요.
  </Step>

  <Step title="자격 증명과 함께 서버 추가">
    다음 방법 중 하나를 선택하세요. `--callback-port`에 사용되는 포트는 사용 가능한 포트면 됩니다. 이전 단계에서 등록한 리다이렉트 URI와 일치해야 합니다.

    <Tabs>
      <Tab title="claude mcp add">
        `--client-id`를 사용하여 앱의 클라이언트 ID를 전달합니다. `--client-secret` 플래그는 마스킹된 입력으로 시크릿을 요청합니다:

        ```bash theme={null}
        claude mcp add --transport http \
          --client-id your-client-id --client-secret --callback-port 8080 \
          my-server https://mcp.example.com/mcp
        ```
      </Tab>

      <Tab title="claude mcp add-json">
        JSON 구성에 `oauth` 객체를 포함하고 `--client-secret`을 별도 플래그로 전달합니다:

        ```bash theme={null}
        claude mcp add-json my-server \
          '{"type":"http","url":"https://mcp.example.com/mcp","oauth":{"clientId":"your-client-id","callbackPort":8080}}' \
          --client-secret
        ```
      </Tab>

      <Tab title="claude mcp add-json (콜백 포트만 지정)">
        동적 클라이언트 등록을 사용하면서 포트만 고정하려면 클라이언트 ID 없이 `--callback-port`를 사용합니다:

        ```bash theme={null}
        claude mcp add-json my-server \
          '{"type":"http","url":"https://mcp.example.com/mcp","oauth":{"callbackPort":8080}}'
        ```
      </Tab>

      <Tab title="CI / 환경 변수">
        인터랙티브 프롬프트를 건너뛰려면 환경 변수를 통해 시크릿을 설정합니다:

        ```bash theme={null}
        MCP_CLIENT_SECRET=your-secret claude mcp add --transport http \
          --client-id your-client-id --client-secret --callback-port 8080 \
          my-server https://mcp.example.com/mcp
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="Claude Code에서 인증">
    Claude Code에서 `/mcp`를 실행하고 브라우저 로그인 흐름을 따릅니다.
  </Step>
</Steps>

<Tip>
  팁:

  * 클라이언트 시크릿은 구성 파일이 아닌 시스템 키체인(macOS) 또는 자격 증명 파일에 안전하게 저장됨
  * 서버가 시크릿이 없는 공개 OAuth 클라이언트를 사용하는 경우 `--client-secret` 없이 `--client-id`만 사용
  * `--callback-port`는 `--client-id` 유무와 관계없이 사용 가능
  * 이 플래그들은 HTTP 및 SSE 트랜스포트에만 적용되며 stdio 서버에는 영향 없음
  * `claude mcp get <name>`을 사용하여 서버에 OAuth 자격 증명이 구성되어 있는지 검증 가능
</Tip>

### OAuth 메타데이터 탐색 재정의

기본 탐색 체인을 우회하려면 Claude Code가 특정 OAuth 인증 서버 메타데이터 URL을 가리키도록 지정하세요. MCP 서버의 표준 엔드포인트에서 오류가 발생하거나 내부 프록시를 통해 탐색을 라우팅하려는 경우 `authServerMetadataUrl`을 설정합니다. 기본적으로 Claude Code는 먼저 `/.well-known/oauth-protected-resource`에서 RFC 9728 보호 리소스 메타데이터를 확인한 다음 `/.well-known/oauth-authorization-server`에서 RFC 8414 인증 서버 메타데이터로 떨어집니다.

`.mcp.json`의 서버 구성 내 `oauth` 객체에 `authServerMetadataUrl`을 설정합니다:

```json theme={null}
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

URL은 반드시 `https://`를 사용해야 합니다. 메타데이터 URL의 `scopes_supported`는 업스트림 서버가 알리는 스코프를 재정의합니다.

### OAuth 스코프 제한

인가(authorization) 흐름 중 Claude Code가 요청하는 스코프를 고정하려면 `oauth.scopes`를 설정하세요. 이는 업스트림 인가 서버가 부여하고자 하는 것보다 더 많은 스코프를 제공할 때 보안 팀이 승인한 하위 집합으로 MCP 서버를 제한하는 지원 방법입니다. 값은 RFC 6749 §3.3의 `scope` 매개변수 형식과 일치하는 공백 구분 단일 문자열입니다.

```json theme={null}
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": {
        "scopes": "channels:read chat:write search:read"
      }
    }
  }
}
```

`oauth.scopes`는 `authServerMetadataUrl`과 서버가 `/.well-known`에서 탐색한 스코프 모두보다 우선 적용됩니다. 요청된 스코프 세트를 MCP 서버가 결정하도록 하려면 설정하지 않은 상태로 두세요.

v2.1.196부터 `oauth.scopes`가 설정되어 있지 않을 때 Claude Code는 서버의 `WWW-Authenticate` 헤더 또는 보호된 리소스 메타데이터가 제공하는 스코프를 요청하며 둘 다 제공하지 않을 때는 `scope` 매개변수를 보내지 않습니다. 더 이상 자동으로 탐색된 인가 서버 메타데이터에서 전체 `scopes_supported` 카탈로그를 요청하지 않습니다. 해당 카탈로그를 요청하면 관리자 전용 또는 템플릿 스코프를 공지하는 아이덴티티 제공자가 `invalid_scope` 오류와 함께 인가 요청을 거부했었기 때문입니다. 구성된 `authServerMetadataUrl`에서 가져온 메타데이터는 여전히 요청된 스코프로 `scopes_supported`를 제공합니다.

인가 서버가 `scopes_supported`에 `offline_access`를 알리는 경우 Claude Code가 이를 고정된 스코프 뒤에 붙여 새 브라우저 로그인 없이 액세스 토큰을 새로고침할 수 있도록 합니다.

서버가 나중에 도구 호출에 대해 403 `insufficient_scope`를 반환하면 Claude Code가 동일하게 고정된 스코프로 다시 인증합니다. 필요한 도구가 고정된 세트 외부의 스코프를 요구하는 경우 `oauth.scopes`를 확장하세요.

### 커스텀 인증을 위해 동적 헤더 사용

MCP 서버가 OAuth 이외의 인증 체계(Kerberos, 단기 토큰, 내부 SSO 등)를 사용하는 경우 `headersHelper`를 사용하여 연결 시점에 요청 헤더를 생성하세요. Claude Code가 명령을 실행하고 그 출력을 연결 헤더에 병합합니다.

```json theme={null}
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "echo '{\"Authorization\": \"Bearer '\"$(get-token)\"'\"}'"
    }
  }
}
```

**요구사항:**

* 명령은 문자열 키-값 쌍의 JSON 객체를 stdout에 작성해야 함
* 명령은 세션의 현재 작업 디렉토리에서 10초 타임아웃을 가진 셸에서 실행됨. 스크립트에는 절대 경로나 `PATH` 상의 명령을 사용
* 동적 헤더는 동일한 이름의 정적 `headers`를 재정의함

헬퍼는 세션 시작 및 재연결 시 매 연결마다 새로 실행됩니다. 캐싱이 없으므로 토큰 재사용은 스크립트가 담당합니다.

v2.1.193부터 도구 호출이 `401 Unauthorized` 또는 `403 Forbidden`을 반환하면 Claude Code가 자동으로 헬퍼를 다시 실행하고 새 헤더로 다시 연결하여 호출을 1회 재시도합니다. 해당 재시도도 실패하는 경우에만 Claude Code가 `/mcp`에서 서버를 인증 필요 상태로 표시합니다.

Claude Code는 헬퍼 실행 시 다음 환경 변수들을 설정합니다:

| 변수                          | 값                                                                                                           |
| :---------------------------- | :----------------------------------------------------------------------------------------------------------- |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP 서버 이름                                                                                                |
| `CLAUDE_CODE_MCP_SERVER_URL`  | MCP 서버 URL                                                                                                 |
| `CLAUDE_PLUGIN_ROOT`          | 플러그인 루트 디렉토리. [플러그인](/docs/en/plugins-reference#mcp-servers)이 서버를 제공하는 경우에만 설정됨 |

여러 MCP 서버를 처리하는 단일 헬퍼 스크립트를 작성할 때 이를 활용하세요.

플러그인이 제공하는 서버의 경우 헬퍼는 작업 디렉토리가 플러그인 루트로 설정된 상태로 실행되므로 상대 경로 `headersHelper`는 세션의 작업 디렉토리가 아닌 플러그인 디렉토리 내부에서 확인됩니다. Claude Code v2.1.195 이상이 필요합니다.

명령이 셸을 통해 실행되므로 플러그인이 제공하는 `headersHelper`는 플러그인의 [`${user_config.*}`](/docs/en/plugins-reference#user-configuration) 값을 참조할 수 없습니다. Claude Code는 [오류](/docs/en/errors#plugin-command-references-user-config)와 함께 서버가 잘못 구성된 것으로 보고하며 값을 대체하지 않습니다. 셸 파싱되지 않는 서버의 `headers` 필드에 `${user_config.KEY}`를 넣거나 헬퍼 스크립트가 자체 환경 변수나 구성 파일에서 값을 읽도록 하세요. v2.1.207 이전에는 `headersHelper`가 `${user_config.*}` 값을 대체했었습니다.

<Note>
  `headersHelper`는 임의의 셸 명령을 실행합니다. 프로젝트 또는 로컬 스코프에 정의된 경우 작업 공간 신뢰 대화 상자를 승인한 후에만 실행됩니다.
</Note>

## JSON 구성으로 MCP 서버 추가

MCP 서버에 대한 JSON 구성이 있는 경우 직접 추가할 수 있습니다:

<Steps>
  <Step title="JSON으로 MCP 서버 추가">
    ```bash theme={null}
    # 기본 구문
    claude mcp add-json <name> '<json>'

    # 예시: JSON 구성으로 HTTP 서버 추가
    claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

    # 예시: JSON 구성으로 stdio 서버 추가
    claude mcp add-json local-weather '{"type":"stdio","command":"/path/to/weather-cli","args":["--api-key","abc123"],"env":{"CACHE_DIR":"/tmp"}}'

    # 예시: 사전 구성된 OAuth 자격 증명이 포함된 HTTP 서버 추가
    claude mcp add-json my-server '{"type":"http","url":"https://mcp.example.com/mcp","oauth":{"clientId":"your-client-id","callbackPort":8080}}' --client-secret
    ```
  </Step>

  <Step title="서버가 추가되었는지 검증">
    ```bash theme={null}
    claude mcp get weather-api
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 셸에서 JSON이 올바르게 이스케이프 처리되었는지 확인
  * JSON은 MCP 서버 구성 스키마를 준수해야 함
  * 프로젝트 전용 구성 대신 사용자 구성에 서버를 추가하려면 `--scope user`를 사용 가능
</Tip>

## Claude Desktop에서 MCP 서버 가져오기

Claude Desktop에서 이미 MCP 서버를 구성한 경우 이를 가져올 수 있습니다:

<Steps>
  <Step title="Claude Desktop에서 서버 가져오기">
    ```bash theme={null}
    # 기본 구문
    claude mcp add-from-claude-desktop
    ```
  </Step>

  <Step title="가져올 서버 선택">
    명령 실행 후 가져올 서버를 선택할 수 있는 대화형 대화 상자가 나타납니다.
  </Step>

  <Step title="서버가 가져와졌는지 검증">
    ```bash theme={null}
    claude mcp list
    ```
  </Step>
</Steps>

`claude mcp` 명령을 통해 추가되는 서버 이름은 영문자, 숫자, 하이픈, 언더스코어만 포함할 수 있습니다. Claude Desktop은 해당 제한을 적용하지 않으므로 공백과 같이 다른 문자가 포함된 이름의 Claude Desktop 서버는 가져올 수 없습니다. 가져오기 기능은 거부된 각 이름을 보고하며 선택한 다른 서버는 정상적으로 가져옵니다. v2.1.205 이전에는 첫 번째 유효하지 않은 이름으로 인해 가져오기가 중단되고 선택한 어떤 서버도 추가되지 않았었습니다.

<Tip>
  팁:

  * 이 기능은 macOS 및 Windows Subsystem for Linux (WSL)에서만 작동함
  * 해당 플랫폼의 표준 위치에서 Claude Desktop 구성 파일을 읽음
  * 사용자 구성에 서버를 추가하려면 `--scope user` 플래그 사용
  * 가져온 서버는 이름이 영문자, 숫자, 하이픈, 언더스코어만 포함하는 경우 Claude Desktop과 동일한 이름을 유지함. 다른 문자가 포함된 서버는 Claude Code가 보고하고 건너뜀
  * 동일한 이름의 서버가 이미 존재하는 경우 숫자 접미사가 붙음 (예: `server_1`)
</Tip>

## claude.ai의 MCP 서버 사용

[claude.ai](https://claude.ai) 계정으로 Claude Code에 로그인한 경우, claude.ai에서 추가한 MCP 서버([커넥터](https://claude.com/docs/connectors))를 Claude Code에서 자동으로 사용할 수 있습니다:

<Steps>
  <Step title="claude.ai에서 MCP 서버 구성">
    [claude.ai/customize/connectors](https://claude.ai/customize/connectors)에서 서버를 추가합니다. Team 및 Enterprise 플랜에서는 관리자만 서버를 추가할 수 있습니다.
  </Step>

  <Step title="MCP 서버 인증">
    claude.ai에서 필요한 인증 단계를 완료합니다.
  </Step>

  <Step title="Claude Code에서 서버 보기 및 관리">
    Claude Code에서 다음 명령을 사용합니다:

    ```text theme={null}
    /mcp
    ```

    claude.ai의 서버는 목록에 claude.ai에서 제공됨을 나타내는 표시와 함께 나타납니다.
  </Step>
</Steps>

v2.1.161부터 로그인한 적 없는 커넥터는 claude.ai 섹션 끝의 `Show unused connectors` 행 아래로 접혀 조직에 배포된 목록이 패널을 채우지 않도록 합니다. 확장하려면 해당 행을 선택하세요. 이전에 로그인했던 커넥터는 현재 다시 인증이 필요한 경우에도 계속 표시됩니다.

claude.ai의 커넥터는 활성 [인증 방식](/docs/en/authentication#authentication-precedence)이 claude.ai 구독 로그인일 때만 가져옵니다. 이전 이전에 `/login`을 실행했더라도 `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `apiKeyHelper`, 또는 Amazon Bedrock / Google Cloud's Agent Platform과 같은 서드파티 프로바이더가 활성화되어 있는 동안에는 로드되지 않습니다. 또한 모델 요청만 가능한 [`claude setup-token`](/docs/en/authentication#generate-a-long-lived-token)의 토큰을 `CLAUDE_CODE_OAUTH_TOKEN`이 들고 있을 때도 로드되지 않습니다.

`/mcp`에 추가한 커넥터가 표시되지 않으면 `/status`를 실행하여 활성화된 인증 방식을 확인하고 해당 환경 변수의 설정을 해제하거나 `apiKeyHelper` 설정을 제거한 후 `/login`을 실행하여 claude.ai 계정을 선택하세요.

Claude Code에서 추가한 서버는 동일한 URL을 가리키는 claude.ai 커넥터보다 [우선 적용](#scope-hierarchy-and-precedence)됩니다. 이 경우 `/mcp`는 커넥터를 숨김 상태로 표시하고 커넥터를 사용하고 싶은 경우 중복 항목을 제거하는 방법을 보여줍니다.

Microsoft 365, Gmail, Google Calendar와 같이 Anthropic이 호스팅하는 일부 커넥터는 업스트림 아이덴티티 제공자가 claude.ai가 등록한 리다이렉트 URL만 수용하므로 Claude Code의 로컬 OAuth를 지원하지 않습니다. v2.1.162부터 `/mcp`에서 이러한 호스트 중 하나를 인증하면 claude.ai의 Settings → Connectors에서 연결하라는 안내 메시지가 표시됩니다. 거기서 연결되면 커넥터가 Claude Code에 자동으로 나타납니다.

### 커넥터 도구에 대한 조직 제어

조직은 [claude.ai 커넥터](https://claude.com/docs/connectors)에 대해 도구별 제어를 설정할 수 있습니다. Claude Code는 시작 시 이 설정을 읽고 로컬에서 강제 적용합니다. `/mcp`를 실행하여 커넥터의 각 도구에 적용된 설정을 확인하세요.

* **도구가 `ask`로 설정됨**: Claude Code가 `Your organization requires approval for this tool`이라는 이유와 함께 매 호출마다 승인을 요청합니다. `acceptEdits`, `auto`, `bypassPermissions` [권한 모드](/docs/en/permissions#permission-modes)에서도 프롬프트가 표시되며 "다시 묻지 않음" 옵션을 제공하지 않습니다. 도구와 일치하는 [허용 규칙](/docs/en/permissions)도 프롬프트를 건너뛰지 못합니다. 절대로 묻지 않는 `dontAsk` 모드에서는 Claude Code가 호출을 거부합니다.
* **도구가 `blocked`로 설정됨**: Claude Code가 Claude에게 노출되기 전에 도구를 필터링하여 제거하므로 도구 목록에 나타나지 않습니다.

이러한 제어를 강제 적용하려면 Claude Code v2.1.129 이상이 필요합니다. 구버전은 이 설정을 무시하고 표준 권한 흐름을 적용합니다.

### claude.ai 커넥터 비활성화

Claude Code에서 claude.ai MCP 서버를 비활성화하려면 임의의 설정 스코프에서 [`disableClaudeAiConnectors`](/docs/en/settings#available-settings)를 `true`로 설정하세요:

```json theme={null}
{
  "disableClaudeAiConnectors": true
}
```

이 설정은 어디서든 true면 적용되는 의미 구조(any-source-true)를 사용하므로 어떠한 설정 소스에서든 `true`가 우선 적용됩니다. 커밋된 프로젝트의 `.claude/settings.json`에서 저장소를 클라우드 커넥터에서 제외시킬 수 있지만, 프로젝트 수준의 `false`로 사용자 또는 정책 수준의 `true`가 비활성화한 커넥터를 다시 활성화할 수는 없습니다. `--mcp-config`를 통해 명시적으로 전달된 서버는 영향받지 않습니다.

`ENABLE_CLAUDEAI_MCP_SERVERS` 환경 변수를 `false`로 설정할 수도 있으며 현재 셸 세션에 대해 동일한 효과를 냅니다:

```bash theme={null}
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

모든 claude.ai 커넥터 대신 개별 커넥터를 차단하려면 이름 또는 URL 패턴으로 [`deniedMcpServers`](/docs/en/managed-mcp)에 추가하세요. 예를 들어 `"claude.ai Slack"`이라는 `serverName` 항목은 Slack 커넥터를 차단합니다. 현재 프로젝트에 대해서만 커넥터를 토글하려면 `/mcp` 패널을 사용하세요.

<Note>
  이 클라이언트 측 설정은 로컬 Claude Code 세션을 통제합니다. [Claude Code on the web](/docs/en/claude-code-on-the-web) 세션의 경우 원격 호스트에 의해 claude.ai 커넥터가 프로비저닝되고 명시적인 `--mcp-config` 항목으로 도착하므로 `disableClaudeAiConnectors`가 적용되지 않습니다. 커넥터 URL도 세션 프록시를 통해 다시 작성되므로 벤더 URL을 지목하는 `deniedMcpServers` `serverUrl` 패턴이 일치하지 않게 됩니다. 클라우드 세션이 사용할 수 있는 커넥터 관리는 claude.ai 조직 설정에서 수행하세요.
</Note>

## Claude Code를 MCP 서버로 사용

다른 애플리케이션이 연결할 수 있는 MCP 서버 자체로 Claude Code를 사용할 수 있습니다:

```bash theme={null}
# Claude를 stdio MCP 서버로 시작
claude mcp serve
```

claude_desktop_config.json에 다음 구성을 추가하여 Claude Desktop에서 이를 사용할 수 있습니다:

```json theme={null}
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

<Warning>
  **실행 파일 경로 구성**: `command` 필드는 Claude Code 실행 파일을 참조해야 합니다. 시스템 PATH에 `claude` 명령이 없다면 실행 파일의 전체 경로를 지정해야 합니다.

  전체 경로 찾기:

  ```bash theme={null}
  which claude
  ```

  그런 다음 구성에 전체 경로 사용:

  ```json theme={null}
  {
    "mcpServers": {
      "claude-code": {
        "type": "stdio",
        "command": "/full/path/to/claude",
        "args": ["mcp", "serve"],
        "env": {}
      }
    }
  }
  ```

  올바른 실행 파일 경로가 없으면 `spawn claude ENOENT`와 같은 오류가 발생합니다.
</Warning>

<Tip>
  팁:

  * 서버는 View, Edit, LS 등 Claude의 도구에 대한 액세스를 제공함
  * Claude Desktop에서 Claude에게 디렉토리의 파일 읽기, 편집하기 등을 요청해 볼 수 있음
  * 이 MCP 서버는 MCP 클라이언트에 Claude Code의 도구만 노출하므로 개별 도구 호출에 대한 사용자 승인 구현은 클라이언트의 책임임
</Tip>

## MCP 출력 제한 및 경고

MCP 도구가 대용량 출력을 생성할 때 Claude Code는 대화 컨텍스트가 과도해지는 것을 방지하도록 토큰 사용량 관리를 돕습니다:

* **출력 경고 임계값**: 임의의 MCP 도구 출력이 10,000 토큰을 초과할 때 Claude Code가 경고를 표시함
* **구성 가능한 제한**: `MAX_MCP_OUTPUT_TOKENS` 환경 변수를 사용하여 허용되는 최대 MCP 출력 토큰을 조정 가능
* **기본 제한**: 기본 최대값은 25,000 토큰임
* **적용 범위**: 환경 변수는 자체 제한을 선언하지 않은 도구에 적용됨. [`anthropic/maxResultSizeChars`](#raise-the-limit-for-a-specific-tool)를 설정한 도구는 `MAX_MCP_OUTPUT_TOKENS` 설정에 상관없이 텍스트 콘텐츠에 해당 주석 값을 대신 사용함. 이미지 데이터를 반환하는 도구는 여전히 `MAX_MCP_OUTPUT_TOKENS`의 적용을 받음

대용량 출력을 생성하는 도구에 대해 제한을 높이려면:

```bash theme={null}
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

이는 특히 다음과 같은 MCP 서버로 작업할 때 유용합니다:

* 대규모 데이터셋 또는 데이터베이스 쿼리
* 상세 보고서 또는 문서 생성
* 광범위한 로그 파일 또는 디버깅 정보 처리

### 특정 도구의 제한 높이기

MCP 서버를 구축 중이라면 도구의 `tools/list` 응답 항목 내 `_meta["anthropic/maxResultSizeChars"]`를 설정하여 개별 도구가 기본 디스크 저장 임계값보다 큰 결과를 반환하도록 허용할 수 있습니다. Claude Code는 해당 도구의 임계값을 주석 값으로 높이며 최대 500,000자까지 허용합니다.

이는 데이터베이스 스키마나 전체 파일 트리와 같이 본질적으로 크지만 필요한 결과를 반환하는 도구에 유용합니다. 주석이 없으면 기본 임계값을 초과하는 결과가 디스크에 저장되고 대화에서는 파일 참조로 대체됩니다.

```json theme={null}
{
  "name": "get_schema",
  "description": "Returns the full database schema",
  "_meta": {
    "anthropic/maxResultSizeChars": 200000
  }
}
```

이 주석은 텍스트 콘텐츠에 대해 `MAX_MCP_OUTPUT_TOKENS`와 독립적으로 적용되므로 사용자가 이를 선언한 도구를 위해 환경 변수를 높일 필요가 없습니다. 이미지 데이터를 반환하는 도구는 여전히 토큰 제한의 적용을 받습니다.

<Warning>
  직접 제어하지 않는 특정 MCP 서버에서 출력 경고가 자주 발생하는 경우 `MAX_MCP_OUTPUT_TOKENS` 제한을 높이는 것을 고려하세요. 서버 작성자에게 `anthropic/maxResultSizeChars` 주석을 추가하거나 응답을 페이지네이션(paginate)하도록 요청할 수도 있습니다. 주석은 이미지 콘텐츠를 반환하는 도구에는 아무런 효과가 없으며, 이러한 도구의 경우 `MAX_MCP_OUTPUT_TOKENS`를 높이는 것이 유일한 옵션입니다.
</Warning>

## 루트 수준 조합자가 포함된 도구 입력 스키마

일부 MCP 서버는 스키마 최상위 수준에 `anyOf`, `oneOf`, `allOf`를 사용하여 도구의 입력 스키마를 JSON Schema 유니온으로 선언합니다. Claude API는 스키마 루트에서 해당 키워드를 수용하지 않습니다. `properties` 내부에 중첩된 조합자는 수용하며 Claude Code가 이를 변경 없이 전송합니다.

Claude Code v2.1.195부터 루트 수준 조합자가 포함된 도구도 계속 사용할 수 있습니다. API로 도구를 전송하기 전에 Claude Code가 스키마를 단일 객체로 평탄화(flatten)하고 도구 설명 앞에 어떤 매개변수 그룹이 함께 속하는지 Claude에게 알리는 문장을 덧붙입니다:

* `allOf`: 모든 브랜치의 속성이 병합되며 각 브랜치의 `required` 목록이 계속 적용됨
* `anyOf` 및 `oneOf`: 모든 브랜치의 속성이 병합되며 스키마로 강제 적용되는 대신 도구 설명에 각 브랜치의 `required` 목록이 기술됨

서버는 Claude가 선택한 인수를 수신하므로 서버 측에서 조합 검증을 계속 유지하세요.

Claude Code가 API가 수용하는 스키마를 생성할 수 없거나, 오프라인 머신과 같이 재작성을 활성화하는 원격 구성을 받지 못하는 배포 환경에서는 해당 도구 하나를 건너뛰고 서버 로그에 원인을 기록하며 서버의 다른 도구는 사용 가능하게 둡니다. v2.1.195 이전 버전은 입력 스키마에 루트 수준 `anyOf`, `oneOf`, `allOf`가 있는 모든 도구를 건너뛰었었습니다.

## 특정 도구에 대한 승인 요구

MCP 서버를 구축 중이라면 도구의 `tools/list` 응답 항목 내 `_meta["anthropic/requiresUserInteraction"]`을 `true`로 설정하여 호출마다 명시적 승인을 필요로 하도록 도구를 표시할 수 있습니다. 값은 반드시 JSON 부울 `true`여야 하며 다른 모든 값은 무시됩니다.

Claude Code는 `acceptEdits`, `auto`, `bypassPermissions` [권한 모드](/docs/en/permissions#permission-modes)에서도 해당 도구의 권한 프롬프트를 매 호출마다 표시하고 "다시 묻지 않음" 옵션을 제공하지 않습니다. 도구와 일치하는 [허용 규칙](/docs/en/permissions#permission-rule-syntax)도 프롬프트를 건너뛰지 못합니다. 절대로 묻지 않는 `dontAsk` 모드에서는 Claude Code가 호출을 거부합니다.

프롬프트는 사람에게 전달되어야 합니다. [`--permission-prompt-tool`](/docs/en/cli-reference#cli-flags)을 사용하는 비대화형 모드에서 이 표시가 붙은 도구에 대한 프롬프트 도구의 `allow` 결과는 `MCP tool requires user interaction; not supported via --permission-prompt-tool` 메시지와 함께 거부로 변환됩니다. Agent SDK의 [`canUseTool` 콜백](/docs/en/agent-sdk/permissions)은 해당 호출을 수신하고 승인할 수 있는데, SDK 애플리케이션이 사용자에게 이를 표시할 것으로 기대되기 때문입니다.

자동 승인이 사람이 동의하지 않음을 의미하게 되는 동의 또는 액세스 부여 단계와 같이 권한 프롬프트 자체가 목적이 되는 도구에 이를 사용하세요. 동일한 서버의 다른 도구들은 일반적인 권한 동작을 유지합니다.

다음 `tools/list` 항목은 항상 승인을 필요로 하도록 하나의 도구를 표시합니다.

```json theme={null}
{
  "name": "grant_access",
  "description": "Requests access to a protected resource",
  "_meta": {
    "anthropic/requiresUserInteraction": true
  }
}
```

`anthropic/requiresUserInteraction` 주석은 Claude Code v2.1.199 이상이 필요합니다. 구버전은 이를 무시하고 표준 권한 흐름을 적용합니다.

[Remote Control](/docs/en/remote-control) 및 [Agent SDK](/docs/en/agent-sdk/overview)로 구축된 애플리케이션과 같은 일부 환경은 일반적으로 한 번의 탭으로 도구 호출을 승인할 수 있습니다. 이 주석이 표시된 도구의 경우 Claude Code가 원탭 조치를 보류하고 대신 도구의 전체 권한 프롬프트를 표시하므로 탭 대신 프롬프트에 답하는 사람으로부터 승인이 전달되도록 합니다.

Claude Code는 안전 경고가 포함되거나 원격 환경이 표시할 수 없는 "항상 허용" 옵션이 있는 경우처럼 터미널 대화 상자만 전체를 렌더링할 수 있는 모든 권한 요청에 대해 동일한 방식으로 원탭 승인을 보류합니다. 해당 요청은 Remote Control 대신 터미널 대화 상자에서 답해야 합니다. Claude Code v2.1.214 이상이 필요합니다.

## MCP 도구 정보 요청(elicitation) 응답

MCP 서버는 도구 정보 요청(elicitation)을 사용하여 작업 중간에 구조화된 입력을 요청할 수 있습니다. 서버가 스스로 얻을 수 없는 정보가 필요한 경우 Claude Code가 대화형 대화 상자를 표시하고 사용자 응답을 서버로 다시 전달합니다. 사용자 측에 필요한 구성은 없으며 서버가 요청할 때 대화 상자가 자동으로 나타납니다.

서버는 두 가지 방식으로 입력을 요청할 수 있습니다:

* **폼 모드 (Form mode)**: Claude Code가 서버가 정의한 폼 필드(예: 사용자 이름 및 비밀번호 프롬프트)가 있는 대화 상자를 표시함. 필드를 채우고 제출
* **URL 모드 (URL mode)**: Claude Code가 인증 또는 승인을 위한 브라우저 URL을 엶. 브라우저에서 흐름을 완료한 후 CLI에서 확인

대화 상자를 표시하지 않고 도구 정보 요청에 자동 응답하려면 [`Elicitation` 훅](/docs/en/hooks#elicitation)을 사용하세요.

도구 정보 요청을 사용하는 MCP 서버를 구축 중이라면 프로토콜 상세 내용 및 스키마 예시에 대해 [MCP elicitation 명세](https://modelcontextprotocol.io/docs/learn/client-concepts#elicitation)를 참조하세요.

## MCP 리소스 사용

MCP 서버는 파일을 참조하는 것과 유사하게 `@` 멘션을 사용하여 참조할 수 있는 리소스를 노출할 수 있습니다.

### MCP 리소스 참조

<Steps>
  <Step title="사용 가능한 리소스 목록 표시">
    프롬프트에 `@`를 입력하여 연결된 모든 MCP 서버의 사용 가능한 리소스를 확인합니다. 자동 완성 메뉴에서 파일과 함께 리소스가 나타납니다.
  </Step>

  <Step title="특정 리소스 참조">
    `@server:protocol://resource/path` 형식을 사용하여 리소스를 참조합니다:

    ```text theme={null}
    @github:issue://123 을 분석하고 수정을 제안해 줄 수 있나요?
    ```

    ```text theme={null}
    @docs:file://api/authentication 의 API 문서를 검토해 주세요
    ```
  </Step>

  <Step title="다중 리소스 참조">
    단일 프롬프트에서 여러 리소스를 참조할 수 있습니다:

    ```text theme={null}
    @postgres:schema://users 와 @docs:file://database/user-model 을 비교해 줘
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 리소스는 참조될 때 자동으로 가져와져 첨부 파일로 포함됨
  * 리소스 경로는 `@` 멘션 자동 완성에서 퍼지 검색(fuzzy-searchable)이 가능함
  * 서버가 지원하는 경우 Claude Code가 MCP 리소스를 나열하고 읽을 수 있는 도구를 자동으로 제공함
  * 리소스에는 MCP 서버가 제공하는 모든 유형의 콘텐츠(텍스트, JSON, 구조화된 데이터 등)가 포함될 수 있음
</Tip>

## MCP 도구 검색을 통한 스케일링

도구 검색(Tool search)은 Claude가 필요로 할 때까지 도구 정의를 지연시켜 MCP 컨텍스트 사용량을 낮게 유지합니다. 세션 시작 시 도구 이름과 서버 지침만 로드되므로 MCP 서버를 추가해도 컨텍스트 창에 미치는 영향이 최소화됩니다. Claude Code는 고정된 서버별 도구 상한을 강제하지 않습니다; 실제 한계는 사용자의 컨텍스트 창 예산입니다.

### 작동 방식

도구 검색은 기본적으로 활성화되어 있습니다. MCP 도구는 사전에 컨텍스트에 로드되지 않고 지연되며, Claude는 작업에 도구가 필요할 때 검색 도구를 사용하여 관련 도구를 탐색합니다. Claude가 실제로 사용하는 도구만 컨텍스트에 입력됩니다. 사용자의 관점에서 MCP 도구는 이전과 정확히 동일하게 작동합니다.

임계값 기반 로딩을 선호하는 경우 `ENABLE_TOOL_SEARCH=auto`를 설정하여 컨텍스트 창의 10% 이내에 들어오는 스키마는 사전에 로드하고 넘치는 부분만 지연시킬 수 있습니다. 모든 옵션은 [도구 검색 구성](#configure-tool-search)을 참조하세요.

### MCP 서버 작성자를 위한 안내

MCP 서버를 구축 중이라면 도구 검색이 활성화되었을 때 서버 지침(server instructions) 필드가 더 유용해집니다. 서버 지침은 [스킬](/docs/en/skills)이 작동하는 방식과 유사하게 도구를 언제 검색해야 하는지 Claude가 이해하도록 돕습니다.

다음을 설명하는 명확하고 서술적인 서버 지침을 추가하세요:

* 도구가 처리하는 작업 범주
* Claude가 도구를 검색해야 하는 시점
* 서버가 제공하는 핵심 기능

Claude Code는 도구 설명과 서버 지침을 각각 2KB로 자릅니다. 잘림을 방지하기 위해 간결하게 유지하고 시작 부분 근처에 핵심적인 내용을 위치시키세요.

### 도구 검색 구성

도구 검색은 기본적으로 활성화되어 있습니다: MCP 도구가 지연되고 요청 시 탐색됩니다. Claude Code는 Google Cloud's Agent Platform에서 기본적으로 이를 비활성화합니다. 대부분의 프록시가 `tool_reference` 블록을 전달하지 않기 때문에 `ANTHROPIC_BASE_URL`이 퍼스트파티가 아닌 호스트를 가리킬 때도 비활성화됩니다. 폴백을 재정의하려면 `ENABLE_TOOL_SEARCH`를 명시적으로 설정하세요.

[`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`](/docs/en/env-vars)를 설정하면 도구 검색이 비활성화 상태로 유지되며 `ENABLE_TOOL_SEARCH`가 이를 재정의할 수 없습니다. 이 변수는 `defer_loading` 도구 정의 및 `tool_reference` 콘텐츠 블록이 필요한 베타 헤더를 제거하기 때문입니다.

도구 검색을 사용하려면 `tool_reference` 블록을 지원하는 모델이 필요합니다: Claude Sonnet 4.5, Claude Haiku 4.5, Claude Opus 4.5 및 이후 모델. 현재 목록은 [API 문서의 모델 호환성](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool#model-compatibility)을 참조하세요. Google Cloud's Agent Platform에서는 Claude Sonnet 4.5 이상 및 Claude Opus 4.5 이상에서 도구 검색이 지원됩니다.

`ENABLE_TOOL_SEARCH` 환경 변수로 도구 검색 동작을 제어합니다:

| 값       | 동작                                                                                                                                                                                                                                                                 |
| :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| (미설정)  | 모든 MCP 도구가 지연되고 요청 시 로드됨. Google Cloud's Agent Platform이나 `ANTHROPIC_BASE_URL`이 퍼스트파티가 아닌 호스트일 때 사전 로드로 폴백됨                                                                                                                         |
| `true`   | 모든 MCP 도구가 지연됨. Claude Code가 Google Cloud's Agent Platform 및 프록시를 통해서도 베타 헤더를 전송함. Sonnet 4.5나 Opus 4.5 이전의 Google Cloud's Agent Platform 모델이나 `tool_reference` 블록을 지원하지 않는 프록시에서는 요청이 실패함                       |
| `auto`   | 임계값 모드: 컨텍스트 창의 10% 이내에 들어오면 사전에 로드되고, 그렇지 않으면 지연됨                                                                                                                                                                                      |
| `auto:N` | 맞춤 비율의 임계값 모드 (`N`은 0-100). 예: 5%의 경우 `auto:5`                                                                                                                                                                                                            |
| `false`  | 모든 MCP 도구가 사전에 로드되며 지연되지 않음                                                                                                                                                                                                                            |

```bash theme={null}
# 맞춤 5% 임계값 사용
ENABLE_TOOL_SEARCH=auto:5 claude

# 도구 검색 완전히 비활성화
ENABLE_TOOL_SEARCH=false claude
```

또는 [settings.json `env` 필드](/docs/en/settings#available-settings)에 값을 설정하세요.

`ToolSearch` 도구를 특정하여 비활성화할 수도 있습니다:

```json theme={null}
{
  "permissions": {
    "deny": ["ToolSearch"]
  }
}
```

### 서버를 지연에서 제외하기

검색 단계 없이 도구가 항상 Claude에게 노출되어야 하는 서버의 경우 해당 서버 구상에서 `alwaysLoad`를 `true`로 설정하세요. 그러면 해당 서버의 모든 도구가 `ENABLE_TOOL_SEARCH` 설정과 관계없이 세션 시작 시 컨텍스트에 로드됩니다. 각 사전 로드 도구는 대화에 사용할 수 있는 컨텍스트를 소비하므로 Claude가 매 턴마다 필요로 하는 소수의 도구에만 이를 사용하세요.

다음 `.mcp.json` 항목은 다른 서버는 지연 상태로 두면서 하나의 HTTP 서버를 면제합니다:

```json theme={null}
{
  "mcpServers": {
    "core-tools": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "alwaysLoad": true
    }
  }
}
```

`alwaysLoad` 필드는 모든 서버 유형에서 사용 가능하며 Claude Code v2.1.121 이상이 필요합니다. MCP 서버는 도구의 `_meta` 객체에 `"anthropic/alwaysLoad": true`를 포함시켜 개별 도구를 항상 로드되도록 표시할 수도 있으며 해당 도구에 대해서만 동일한 효과를 냅니다.

`alwaysLoad: true`를 설정하면 표준 5초 연결 타임아웃으로 제한되어 서버가 연결될 때까지 시작이 차단됩니다. 첫 번째 프롬프트가 작성될 때 도구가 존재해야 하므로 MCP 시작이 다른 경우 [기본적으로 차단되지 않음](/docs/en/env-vars)에도 불구하고 이 방식이 적용됩니다. 다른 서버들은 계속 백그라운드에서 연결을 진행합니다.

## MCP 프롬프트를 명령으로 사용

MCP 서버는 Claude Code에서 명령으로 사용할 수 있는 프롬프트를 노출할 수 있습니다.

### MCP 프롬프트 실행

<Steps>
  <Step title="사용 가능한 프롬프트 탐색">
    `/`를 입력하여 MCP 서버의 명령을 포함한 사용 가능한 모든 명령을 표시합니다. MCP 프롬프트는 `/mcp__servername__promptname` 형식으로 나타납니다.
  </Step>

  <Step title="인수 없이 프롬프트 실행">
    ```text theme={null}
    /mcp__github__list_prs
    ```
  </Step>

  <Step title="인수와 함께 프롬프트 실행">
    많은 프롬프트가 인수를 수용합니다. 명령 뒤에 공백으로 구분하여 전달하세요:

    ```text theme={null}
    /mcp__github__pr_review 456
    ```

    ```text theme={null}
    /mcp__jira__create_issue "Bug in login flow" high
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * MCP 프롬프트는 연결된 서버에서 동적으로 탐색됨
  * 인수는 프롬프트의 정의된 매개변수에 기반하여 파싱됨
  * 프롬프트 결과는 대화에 직접 주입됨
  * 서버 및 프롬프트 이름은 표준화되어 공백이 언더스코어로 변환됨
</Tip>

## 관리형 MCP 구성

사용자가 연결할 수 있는 MCP 서버를 중앙에서 제어해야 하는 조직의 경우 [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. `managed-mcp.json`으로 고정 서버 세트를 배포하는 방법, `allowedMcpServers` 및 `deniedMcpServers`로 서버를 제한하는 방법, 서버가 차단되었을 때 사용자에게 표시되는 내용을 다룹니다.
