> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 오류 레퍼런스 (Error reference)

> Claude Code 런타임 오류 메시지와 각 오류의 의미 및 해결 방법을 찾아보세요.

이 페이지는 Claude Code가 표시하는 런타임 오류와 각 오류에서 복구하는 방법, 오류 메시지 없이 응답이 이상할 때 확인해야 할 사항을 나열합니다. `command not found`나 설정 중 TLS 실패와 같은 설치 오류는 [설치 및 로그인 문제 해결](/docs/en/troubleshoot-install)을 참조하세요.

이러한 오류 및 복구 명령은 CLI, [데스크톱 앱](/docs/en/desktop) 및 [Claude Code on the web](/docs/en/claude-code-on-the-web) 전반에 적용됩니다. 세 가지 환경 모두 동일한 Claude Code CLI를 감싸고 있기 때문입니다. 특정 환경 전용 문제는 해당 페이지의 문제 해결 섹션을 참조하세요.

<Note>
  Claude Code는 모델 응답을 위해 Claude API를 호출하므로 대부분의 런타임 오류는 기본 API 오류 코드에 매핑됩니다. 이 페이지는 Claude Code 내부에서 각 오류가 의미하는 바와 복구 방법을 다룹니다. 원시 HTTP 상태 코드 정의는 [Claude Platform 오류 레퍼런스](https://platform.claude.com/docs/en/api/errors)를 참조하세요.
</Note>

## 오류 찾기

터미널에 표시되는 메시지를 아래 섹션과 매칭하세요.

| 메시지 | 섹션 |
| :--- | :--- |
| `API Error: 500 Internal server error` | [서버 오류](#api-error-500-internal-server-error) |
| `API Error: Repeated 529 Overloaded errors` | [서버 오류](#api-error-repeated-529-overloaded-errors) |
| `Request timed out` | [서버 오류](#request-timed-out) 또는 인터넷 연결 언급 시 [네트워크](#unable-to-connect-to-api) |
| `Server error mid-response. The response above may be incomplete.` | [서버 오류](#the-response-above-may-be-incomplete) |
| `Connection closed mid-response` / `Response stalled mid-stream` | [서버 오류](#the-response-above-may-be-incomplete) |
| `<model> is temporarily unavailable, so auto mode cannot determine the safety of...` | [서버 오류](#auto-mode-cannot-determine-the-safety-of-an-action) |
| `Auto mode could not evaluate this action and is blocking it for safety` | [서버 오류](#auto-mode-cannot-determine-the-safety-of-an-action) |
| `Auto mode classifier transcript exceeded context window` | [서버 오류](#auto-mode-cannot-determine-the-safety-of-an-action) |
| `Agent terminated early due to an API error` | [서버 오류](#agent-terminated-early-due-to-an-api-error) |
| `You've hit your session limit` / `You've hit your weekly limit` | [사용 한도](#youve-hit-your-session-limit) |
| `Usage credits required for 1M context` | [사용 한도](#usage-credits-required-for-1m-context) |
| `Server is temporarily limiting requests` | [사용 한도](#server-is-temporarily-limiting-requests) |
| `Request rejected (429)` | [사용 한도](#request-rejected-429) |
| `Credit balance is too low` | [사용 한도](#credit-balance-is-too-low) |
| `Could not update your spend limit` | [사용 한도](#could-not-update-your-spend-limit) |
| `Not logged in · Please run /login` | [인증 오류](#not-logged-in) |
| `Could not resolve authentication method` | [인증 오류](#could-not-resolve-authentication-method) |
| `Invalid API key` | [인증 오류](#invalid-api-key) |
| `Your apiKeyHelper script is failing` | [인증 오류](#your-apikeyhelper-script-is-failing) |
| `This organization has been disabled` | [인증 오류](#this-organization-has-been-disabled) |
| `Your organization has disabled API key authentication` | [인증 오류](#your-organization-has-disabled-api-key-authentication) |
| `Your organization has disabled Claude subscription access` | [인증 오류](#your-organization-has-disabled-claude-subscription-access) |
| `Routines are disabled by your organization's policy` | [인증 오류](#routines-are-disabled-by-your-organizations-policy) |
| `Remote Control is only available when using Claude via api.anthropic.com` | [인증 오류](#remote-control-requires-the-anthropic-api) |
| `OAuth token revoked` / `OAuth token has expired` | [인증 오류](#oauth-token-revoked-or-expired) |
| `Login expired · Please run /login` | [인증 오류](#login-expired) |
| `Failed to authenticate: OAuth session expired and could not be refreshed` | [인증 오류](#login-expired) |
| `does not meet scope requirement user:profile` | [인증 오류](#oauth-scope-requirement) |
| `AWS credentials expired or invalid` | [인증 오류](#aws-credentials-expired-or-invalid) |
| `AWS authentication failed` | [인증 오류](#aws-authentication-failed) |
| `AWS default-chain credential resolve timed out` | [인증 오류](#aws-default-chain-credential-resolve-timed-out) |
| `Unable to connect to API` | [네트워크 및 연결 오류](#unable-to-connect-to-api) |
| `Socket is closed` | [네트워크 및 연결 오류](#socket-is-closed) |
| `Waiting for API response · will retry in` | [재시도](#automatic-retries) 또는 지속 시 [네트워크](#unable-to-connect-to-api) |
| `Bedrock streaming response has content-type "..."; expected "application/vnd.amazon.eventstream"` | [네트워크 및 연결 오류](#bedrock-streaming-response-has-an-unexpected-content-type) |
| `SSL certificate verification failed` | [네트워크 및 연결 오류](#ssl-certificate-errors) |
| `SSL certificate error (...)` 로그인 또는 시작 시 | [네트워크 및 연결 오류](#ssl-certificate-errors) |
| `403` with `x-deny-reason: host_not_allowed` 클라우드 또는 루틴 세션에서 | [네트워크 및 연결 오류](#host-not-allowed-in-a-cloud-session) |
| `Couldn't reconnect to your Remote Control session` | [네트워크 및 연결 오류](#couldnt-reconnect-to-your-remote-control-session) |
| `Prompt is too long` | [요청 오류](#prompt-is-too-long) |
| `Context exceeds the ...-token limit by ... tokens` `/context` 출력에서 | [요청 오류](#context-exceeds-the-token-limit) |
| `Error during compaction: Conversation too long` | [요청 오류](#error-during-compaction-conversation-too-long) |
| `Request too large` | [요청 오류](#request-too-large) |
| `Image was too large` | [요청 오류](#image-was-too-large) |
| `Unable to resize image` | [요청 오류](#unable-to-resize-image) |
| `PDF too large` / `PDF is password protected` | [요청 오류](#pdf-errors) |
| `Extra inputs are not permitted` | [요청 오류](#extra-inputs-are-not-permitted) |
| `There's an issue with the selected model` | [요청 오류](#theres-an-issue-with-the-selected-model) |
| `Model ... is not a recognized model id` | [요청 오류](#model-is-not-a-recognized-model-id) |
| `Claude Opus is not available with the Claude Pro plan` | [요청 오류](#claude-opus-is-not-available-with-the-claude-pro-plan) |
| `Model ... is restricted by your organization's settings` | [요청 오류](#model-is-restricted-by-your-organizations-settings) |
| `thinking.type.enabled is not supported for this model` | [요청 오류](#thinking-type-enabled-is-not-supported-for-this-model) |
| `max_tokens must be greater than thinking.budget_tokens` | [요청 오류](#thinking-budget-exceeds-output-limit) |
| `API Error: 400 due to tool use concurrency issues` | [요청 오류](#tool-use-or-thinking-block-mismatch) |
| `Claude Code is unable to respond to this request, which appears to violate our Usage Policy` | [요청 오류](#usage-policy-refusal) |
| `<model> has safety measures that flagged this message for a cybersecurity topic` | [요청 오류](#safety-measures-flagged-a-cybersecurity-topic) |
| `Installation was killed before it could finish (exit code 137)` | [설치 오류](#installation-was-killed-before-it-could-finish) |
| `The connection dropped while downloading the update` | [설치 오류](#the-connection-dropped-while-downloading-the-update) |
| `Download timed out: exceeded the total deadline` | [설치 오류](#the-connection-dropped-while-downloading-the-update) |
| `--bg and --print conflict` | [커맨드라인 오류](#command-line-errors) |
| `Error: --json-schema is not a valid JSON Schema` | [커맨드라인 오류](#command-line-errors) |
| `Error: Settings file exceeds the 2MiB limit` | [커맨드라인 오류](#settings-file-exceeds-the-2mib-limit) |
| `Error: Workspace not trusted` Remote Control 시작 시 | [커맨드라인 오류](#workspace-not-trusted-when-starting-remote-control) |
| `Could not import <server>: <reason>` | [커맨드라인 오류](#could-not-import-a-server-from-claude-desktop) |
| `Error: MCP tool <name> (passed via --permission-prompt-tool) not found` | [커맨드라인 오류](#mcp-permission-prompt-tool-not-found) |
| `Diff is too large for ultrareview` / `PR #<N> is too large for ultrareview` | [커맨드라인 오류](#diff-is-too-large-for-ultrareview) |
| `Failed to resume the conversation` | [커맨드라인 오류](#failed-to-resume-the-conversation) |
| `Marketplace "<name>" is registered from an untrusted source` | [플러그인 오류](#marketplace-is-registered-from-an-untrusted-source) |
| `references ${user_config.*} in a shell-form command` | [플러그인 오류](#plugin-command-references-user-config) |
| `Monitor "<name>" from plugin <plugin> references ${user_config.*} in its command` | [플러그인 오류](#plugin-command-references-user-config) |
| `headersHelper for MCP server '<name>' references ${user_config.*}` | [플러그인 오류](#plugin-command-references-user-config) |
| `would be spawned with zero tools — refusing` | [도구 오류](#agent-would-be-spawned-with-zero-tools) |
| `File is covered by a Read deny rule in your permission settings` | [도구 오류](#file-is-covered-by-a-read-deny-rule) |
| `Error: this write left the memory index at MEMORY.md at ..., over its ... read limit` | [도구 오류](#memory-index-is-over-its-read-limit) |
| `pkill: refusing to run` | [도구 오류](#pkill-pattern-matches-the-claude-code-process) |
| `Can't open MCP settings while no terminal is attached to this background session` | [백그라운드 세션 오류](#commands-refused-in-a-background-session) |
| {/* max-version: 2.1.212 */}`Can't open MCP settings in a background session` | [백그라운드 세션 오류](#commands-refused-in-a-background-session) |
| `This session has no saved transcript` | [백그라운드 세션 오류](#this-session-has-no-saved-transcript) |
| `This session was running agent '<name>', which is no longer available` | [백그라운드 세션 오류](#session-agent-no-longer-available) |
| `CLAUDE_CODE_PROCESS_WRAPPER: launcher ...` | [백그라운드 세션 오류](#claude_code_process_wrapper-launcher-errors) |
| `EUNKNOWN: unknown error, uv_spawn` | [백그라운드 세션 오류](#eunknown-when-starting-a-background-session) |
| `Restored the code, but skipped N files` | [되감기 경고](#restored-the-code-but-skipped-files) |
| `Ignoring N permissions.allow entries from ... this workspace has not been trusted` | [구성 경고](#workspace-has-not-been-trusted) |
| 평소보다 응답 품질이 낮게 느껴지는 경우 | [응답 품질 문제](#responses-seem-lower-quality-than-usual) |

## 재시도 (Automatic retries)

Claude Code는 오류를 표시하기 전에 일시적인 실패를 자동으로 재시도합니다. 서버 오류, 과부하 응답, 요청 타임아웃, 일시적인 429 요청 제한 및 끊어진 연결은 지수 백오프(exponential backoff)를 통해 최대 10회까지 재시도됩니다. {/* min-version: 2.1.198 */}v2.1.198부터는 눈에 보이는 출력이 스트리밍되기 전에 응답 중간에 끊어진 연결도 다룹니다. Claude Code는 동일한 백오프로 요청을 다시 전송하며 연결 오류로 중단되는 대신 턴을 계속 이어갑니다. {/* min-version: 2.1.199 */}v2.1.199부터는 claude.ai 구독으로 로그인한 경우 요금제 쿼터 헤더가 포함되지 않은 일시적인 429 제한도 재시도됩니다.

일부 실패 클래스는 재시도가 성공할 수 없기 때문에 재시도되지 않습니다.

* {/* min-version: 2.1.199 */}v2.1.199부터 TLS 인증서 검증 실패(TLS 검사 프록시, 누락된 `NODE_EXTRA_CA_CERTS` 번들, 만료된 인증서 등)는 첫 번째 시도에서 바로 실패하므로 재시도 예산이 다 소모될 때까지 기다리지 않고 수정 사항이 즉시 표시됩니다. [SSL 인증서 오류](#ssl-certificate-errors)를 참조하세요. 핸드셰이크 타임아웃과 같은 일시적인 TLS 상태는 여전히 재시도됩니다.
* {/* min-version: 2.1.199 */}v2.1.199부터 Claude가 이미 표출된 출력을 스트리밍한 후 발생하는 서버 오류는 동일한 도구를 두 번 실행하는 것을 방지하기 위해 재시도하는 대신 부분 응답을 유지하고 [미완성 응답 안내](#the-response-above-may-be-incomplete)를 덧붙입니다.
* {/* min-version: 2.1.208 */}[Amazon Bedrock의 예상치 못한 content-type 스트리밍 응답](#bedrock-streaming-response-has-an-unexpected-content-type)은 응답을 재작성하는 게이트웨이나 프록시가 재시도도 동일하게 재작성하므로 첫 시도에서 실패합니다. Claude Code v2.1.208 이상이 필요합니다.

재시도하는 동안 스피너는 오류 라벨 뒤에 `Retrying in Ns · attempt x/y` 카운트다운을 표시합니다. 라벨에는 바로 조치할 수 있는 실패 원인(네트워크 끊김, TLS 핸드셰이크 실패, Rate limit 도달 등)의 첫 시도 원인이 표시됩니다. 기타 오류의 경우 처음에는 `API error`로 표기됩니다. {/* min-version: 2.1.198 */}v2.1.198부터 세 번째 시도부터 구체적인 원인으로 전환되며, `CLAUDE_CODE_MAX_RETRIES`가 3회 미만으로 설정된 경우 마지막 시도에서 전환됩니다.

{/* min-version: 2.1.198 */}v2.1.198부터 재시도 중에는 일반적인 스피너 팁이 숨겨집니다. 오류 원인이 표출된 후 실패 원인이 529 과부하인 경우 카운트다운 아래 줄에 서비스 상태를 확인할 위치가 이름으로 표기됩니다(Anthropic API의 경우 `status.claude.com`, 기타 구성의 경우 메시지에 이름이 표시된 공급자 또는 게이트웨이 호스트).

{/* min-version: 2.1.185 */}요청이 보류 중인 동안 20초 동안 응답 스트림에 데이터가 도착하지 않으면 스피너는 재시도가 시작되기 전에 `Waiting for API response · will retry in … · check your network`를 표시합니다. 요청이 아직 실패한 것은 아닙니다: Claude Code가 정지된 연결을 중단하고 재시도하는 시점까지 카운트다운이 실행되므로 데이터가 다시 수신되거나 재시도가 성공하면 배너가 저절로 사라집니다. v2.1.185부터 임계값은 20초입니다. 매 시도마다 이것이 다시 나타나면 [네트워크 문제](#unable-to-connect-to-api)로 처리하세요.

{/* min-version: 2.1.214 */}Claude가 [어드바이저](/docs/en/advisor)를 참조하는 동안에는 20초 대신 90초 동안 데이터가 없을 때 배너가 나타납니다. 어드바이저의 긴 검토 과정은 20초 이상 아무것도 보내지 않을 수 있기 때문입니다.

이 페이지의 오류 중 하나가 보이면 해당 재시도가 이미 다 소모된 것입니다(인증서 검증 실패와 같이 재시도되지 않는 클래스에 속하지 않는 한). 다음 환경 변수로 동작을 조정할 수 있습니다.

| 변수 | 기본값 | 효과 |
| :--- | :--- | :--- |
| [`CLAUDE_CODE_MAX_RETRIES`](/docs/en/env-vars) | 10 | 재시도 시도 횟수. {/* min-version: 2.1.186 */}v2.1.186부터 최대 15회로 제한됨; {/* min-version: 2.1.199 */}v2.1.199부터 `CLAUDE_CODE_RETRY_WATCHDOG`가 기본값을 높이고 제한을 제거함. 스크립트에서 실패를 더 빠르게 표출하려면 값을 낮추세요. |
| [`CLAUDE_CODE_RETRY_WATCHDOG`](/docs/en/env-vars) | unset | CI 작업과 같이 무인 세션에서 `1`로 설정하면 `CLAUDE_CODE_MAX_RETRIES` 시도 후 실패하는 대신 `429` 및 `529` 용량 오류를 무기한 재시도함. {/* min-version: 2.1.199 */}v2.1.199부터 서버 오류, 타임아웃, 연결 끊김과 같은 기타 일시적 오류의 기본 재시도 횟수를 300회(약 3시간의 백오프)로 높이고 `CLAUDE_CODE_MAX_RETRIES`의 15회 제한을 제거함. |
| [`API_TIMEOUT_MS`](/docs/en/env-vars) | 600000 | 요청당 타임아웃(밀리초). 느린 네트워크나 프록시 환경에서는 높이세요. |

## 서버 오류 (Server errors)

이러한 오류는 계정이나 요청이 아닌 추론 공급자 측에서 발생합니다. Anthropic API의 경우 Anthropic 인프라를 의미합니다. Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 또는 커스텀 게이트웨이의 경우 해당 공급자의 인프라를 의미합니다.

### API Error: 500 Internal server error

Claude Code는 임의의 5xx 응답에 대해 상태 코드와 API의 오류 메시지를 표시합니다. 아래 예시는 Anthropic API의 500 응답입니다.

```text theme={null}
API Error: 500 Internal server error. This is a server-side issue, usually temporary — try again in a moment. If it persists, check https://status.claude.com.
```

뒷부분 문장은 서비스 상태를 확인할 수 있는 위치를 지정하며 공급자별로 다릅니다.

이 오류는 API 내부의 예기치 않은 실패를 나타냅니다. 프롬프트, 설정 또는 계정 때문이 아닙니다.

**조치 방법:**

* [status.claude.com](https://status.claude.com)이나 메시지에 표기된 공급자 상태 페이지에서 장애가 있는지 확인하세요.
* 잠시 기다린 후 메시지를 다시 전송하세요. 원본 메시지가 대화에 남아 있으므로 긴 프롬프트의 경우 전체를 붙여넣는 대신 `try again`이라고 입력할 수 있습니다.
* 포스팅된 장애 없이 오류가 지속되면 Anthropic이 요청 세부 정보를 조사할 수 있도록 `/feedback`을 실행하세요.

### API Error: Repeated 529 Overloaded errors

API가 전체 사용자에 대해 일시적으로 용량 초과 상태입니다. Claude Code는 이 메시지를 표시하기 전에 이미 여러 번 재시도했습니다.

```text theme={null}
API Error: Repeated 529 Overloaded errors. The API is at capacity — this is usually temporary. Try again in a moment. If it persists, check https://status.claude.com.
```

529 오류는 사용자의 사용 한도가 아니며 쿼터에 반영되지 않습니다.

**조치 방법:**

* [status.claude.com](https://status.claude.com)이나 공급자 상태 페이지에서 용량 알림을 확인하세요.
* 몇 분 후 다시 시도하세요.
* 용량은 모델별로 추적되므로 `/model`을 실행하고 다른 모델로 전환하여 작업을 계속하세요.

### Request timed out

연결 마감 시간 전에 API가 응답하지 않았습니다.

```text theme={null}
Request timed out
```

이는 높은 부하 기간 동안이나 모델이 매우 큰 응답을 생성할 때 발생할 수 있습니다. 기본 요청 타임아웃은 10분입니다.

**조치 방법:**

* 요청을 다시 시도하세요.
* 오래 걸리는 작업의 경우 작업을 더 작은 프롬프트로 분할하세요.
* 느린 네트워크나 프록시가 원인인 경우 `API_TIMEOUT_MS`를 높이세요.

### The response above may be incomplete

Claude가 이미 눈에 보이는 출력을 생성한 후 스트리밍 응답이 실패했습니다. 요청을 다시 전송하면 동일한 도구 호출이 두 번 실행될 수 있으므로 Claude Code는 이미 스트리밍된 내용을 유지하고 이 알림을 덧붙입니다.

```text theme={null}
API Error: Server error mid-response. The response above may be incomplete.
API Error: Connection closed mid-response. The response above may be incomplete.
API Error: Response stalled mid-stream. The response above may be incomplete.
```

**조치 방법:**

* 스트리밍된 응답을 읽어보세요. 잃어버린 내용은 없지만 마지막 문장이나 도구 호출이 누락되었을 수 있습니다.
* `continue`라고 답하여 Claude가 중단된 부분부터 다시 시작하도록 하세요.

### Auto mode cannot determine the safety of an action

[자동 모드(auto mode)](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)가 동작을 분류하기 위해 사용하는 모델이 결정을 생성할 수 없어 자동으로 동작을 승인하지 못했습니다.

작업 디렉토리 내부의 읽기, 검색, 편집은 분류기를 거치지 않으므로 이러한 경우에도 계속 작동합니다.

분류기 모델이 과부하 상태일 때:

```text theme={null}
<model> is temporarily unavailable, so auto mode cannot determine the safety of <tool> right now. Wait briefly and then try this action again.
```

**조치 방법:**

* 몇 초 후 다시 시도하세요. Claude도 동일한 메시지를 확인하고 보통 스스로 재시도합니다.
* 재시도가 계속 실패하면 읽기 전용 작업으로 진행하고 차단된 동작은 나중에 다시 시도하세요.

분류기가 파싱 불가능한 응답을 반환했을 때:

```text theme={null}
Auto mode could not evaluate this action and is blocking it for safety — run with --debug for details
```

**조치 방법:**

* 동작을 다시 시도하세요. 대개 다음 시도에서 성공합니다.
* `claude --debug`를 실행하고 동작을 반복하여 디버그 로그에서 기본 분류기 응답을 확인하세요.

대화 기록이 분류기의 컨텍스트 창보다 커졌을 때:

```text theme={null}
Auto mode classifier transcript exceeded context window — falling back to manual approval (try /compact to reduce conversation size)
```

대화형 세션에서는 수동으로 승인하거나 거부할 수 있도록 정상 권한 프롬프트로 폴백됩니다. [비대화형 모드](/docs/en/headless)에서는 실행이 중단됩니다.

**조치 방법:**

* 표시되는 프롬프트에서 동작을 승인하거나 거부하세요.
* `/compact`를 실행하여 대화 크기를 줄이세요.

### Agent terminated early due to an API error

[하위 에이전트(subagent)](/docs/en/sub-agents)의 API 요청이 영구적으로 실패(사용 한도 도달 또는 서버 오류 재시도 소진 등)하여 작업을 마치기 전에 중단되었습니다.

```text theme={null}
Agent terminated early due to an API error: <error detail>
```

**조치 방법:**

* 콜론 뒤의 오류 세부 정보를 본 페이지의 해당 섹션과 매칭하고 해결 단계를 따르세요.
* 기본 오류가 해결되면 Claude에게 작업을 다시 시도하거나 [하위 에이전트 재개](/docs/en/sub-agents#resume-subagents)를 요청하세요.

## 사용 한도 (Usage limits)

이러한 오류는 계정이나 요금제에 연결된 쿼터에 도달했음을 의미합니다.

<h3 id="youve-hit-your-session-limit">
  You've hit your session limit
</h3>

구독 요금제에는 롤링 사용 할당량이 포함되어 있습니다. 다 소모되면 다음 메시지 중 하나가 표시됩니다.

```text theme={null}
You've hit your session limit · resets 3:45pm
You've hit your weekly limit · resets Mon 12:00am
You've hit your Opus limit · resets 3:45pm
```

Claude Code는 메시지에 표시된 재설정 시간까지 추가 요청을 차단합니다. 세션 및 주간 한도는 모든 모델에서 공유되므로 모델을 전환해도 접근 권한이 복원되지 않습니다. Opus 한도는 Opus 요청에만 적용되므로 `/model`로 다른 모델로 전환하면 계속 작업할 수 있습니다.

**조치 방법:**

* 오류에 표시된 재설정 시간을 기다리세요.
* Opus 한도의 경우 `/model`을 실행하여 다른 모델로 전환하세요.
* `/usage`를 실행하여 요금제 한도 및 재설정 시점을 확인하세요.
* Pro 및 Max의 경우 `/usage-credits`를 실행하여 추가 사용량을 구매하거나, Team 및 Enterprise의 경우 관리자에게 요청하세요.

### Usage credits required for 1M context

선택한 모델이 1M 토큰 확장 컨텍스트 창을 사용하며 요금제에는 사용 크레딧을 통해서만 포함됩니다.

```text theme={null}
API Error: Usage credits required for 1M context · run /usage-credits to turn them on, or /model to switch to standard context
```

**조치 방법:**

* `/model`을 실행하고 `[1m]` 접미사가 없는 변형을 선택하여 표준 컨텍스트 창으로 폴백하세요.
* Pro 및 Max에서 `/usage-credits`를 실행하여 종량제 결제를 켜거나 Team 및 Enterprise에서 관리자에게 요청하세요.

### Server is temporarily limiting requests

API가 요금제 쿼터와 관련 없는 단기 속도 제한(throttle)을 적용했습니다.

```text theme={null}
API Error: Server is temporarily limiting requests (not your usage limit)
```

**조치 방법:**

* 잠시 기다린 후 다시 시도하세요.
* 지속되면 [status.claude.com](https://status.claude.com)을 확인하세요.

### Request rejected (429)

API 키, Amazon Bedrock 프로젝트 또는 Google Cloud 프로젝트에 구성된 처리율 제한(rate limit)에 도달했습니다.

```text theme={null}
API Error: Request rejected (429) · this may be a temporary capacity issue. If it persists, check https://status.claude.com.
```

**조치 방법:**

* `/status`를 실행하여 활성 자격 증명이 예상한 것인지 확인하세요.
* 공급자 콘솔에서 활성 한도를 확인하고 필요한 경우 더 높은 등급을 요청하세요.

### Credit balance is too low

Console 조직의 선불 크레딧이 다 소모되었습니다.

```text theme={null}
Credit balance is too low
```

**조치 방법:**

* [platform.claude.com/settings/billing](https://platform.claude.com/settings/billing)에서 크레딧을 추가하세요.
* Pro, Max, Team 또는 Enterprise 요금제가 있는 경우 `/login`으로 구독 인증으로 전환하세요.

### Could not update your spend limit

지출 한도 도달 시 표시되는 프롬프트에서 변경한 지출 한도가 서버에서 거부되었습니다.

```text theme={null}
Could not update your spend limit: <reason from the server>
```

**조치 방법:**

* 메시지에 이유가 포함되어 있다면 더 낮은 금액 등 이를 만족하는 한도를 선택하세요.
* 일반적인 형태만 표출된다면 재시도하세요.
* 지속적으로 실패하면 브라우저의 [claude.ai 결제 설정](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)에서 변경하세요.

## 인증 오류 (Authentication errors)

이러한 오류는 Claude Code가 API에 사용자의 신원을 증명할 수 없음을 의미합니다. 언제든지 `/status`를 실행하여 현재 활성화된 자격 증명을 확인할 수 있습니다.

### Not logged in

이 세션에 사용 가능한 유효한 자격 증명이 없습니다.

```text theme={null}
Not logged in · Please run /login
```

**조치 방법:**

* `/login`을 실행하여 Claude 구독 또는 Console 계정으로 인증하세요.
* 환경 변수를 예상한 경우 `ANTHROPIC_API_KEY`가 설정되어 있는지 확인하세요.

### Could not resolve authentication method

세션이 자격 증명 없이 API 클라이언트에 도달했습니다. 대화형 로그인 검사가 첫 요청 전 실행되지 않는 백그라운드 세션, 클라우드 세션, Agent SDK 환경에서 나타납니다.

```text theme={null}
Could not resolve authentication method. Expected one of apiKey, authToken, credentials, config, or profile to be set. Or for one of the "X-Api-Key" or "Authorization" headers to be explicitly omitted
```

**조치 방법:**

* 백그라운드 또는 클라우드 세션에서 발생하는 경우 최신 버전으로 업그레이드하세요.
* 워커를 실행하는 환경에 자격 증명이 설정되어 있는지 확인하세요.

### Invalid API key

`ANTHROPIC_API_KEY` 환경 변수 또는 `apiKeyHelper` 스크립트가 API에서 거부된 키를 반환했습니다.

```text theme={null}
Invalid API key · Fix external API key
```

**조치 방법:**

* 오타를 확인하고 키가 취소되지 않았는지 확인하세요.
* `ANTHROPIC_API_KEY`를 해제하고 `/login`을 실행하여 구독 인증을 사용하세요.

### Your apiKeyHelper script is failing

[`apiKeyHelper`](/docs/en/settings#available-settings) 설정에 구성된 명령이 오류로 종료되었거나 타임아웃되거나 stdout에 아무것도 출력하지 않았습니다.

```text theme={null}
Your apiKeyHelper script is failing · This usually means you need to re-authenticate with your provider · Run /status to see the script's error output
```

**조치 방법:**

* 셸에서 `apiKeyHelper` 명령을 직접 실행하여 실패를 재현해 보세요.
* 만료된 세션이 보고되면 자격 증명 공급자(SSO 등)에 다시 인증하세요.

### This organization has been disabled

비활성화된 Console 조직의 오래된 `ANTHROPIC_API_KEY`가 구독 로그인을 재정의하고 있습니다.

```text theme={null}
Your ANTHROPIC_API_KEY belongs to a disabled organization · Unset the environment variable to use your other credentials
API Error: 400 ... This organization has been disabled.
```

**조치 방법:**

* 셸에서 `ANTHROPIC_API_KEY`를 해제하고 `claude`를 다시 실행하세요.

### Your organization has disabled API key authentication

조직의 관리자가 API 키 인증을 비활성화했습니다.

```text theme={null}
Your organization has disabled API key authentication · Run /login to sign in with your claude.ai account
```

**조치 방법:**

* `ANTHROPIC_API_KEY`를 해제하고 `/login`을 실행하여 claude.ai 계정으로 로그인하세요.

### Your organization has disabled Claude subscription access

조직에서 구독 로그인으로 Claude Code에 로그인하는 것을 허용하지 않습니다.

```text theme={null}
Your organization has disabled Claude subscription access for Claude Code · Use an Anthropic API key instead, or ask your admin to enable access
```

**조치 방법:**

* 관리자에게 조직의 Claude Code 접근 권한 활성화를 요청하세요.
* Console API 키로 인증하세요.

<h3 id="routines-are-disabled-by-your-organizations-policy">
  Routines are disabled by your organization's policy
</h3>

조직 정책으로 인해 루틴(Routines)이 비활성화되었습니다.

```text theme={null}
Routines are disabled by your organization's policy.
```

**조치 방법:**

* 소유자에게 [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)에서 **Routines** 토글을 켜달라고 요청하세요.

### Remote Control requires the Anthropic API

세션이 Anthropic API와 직접 통신하지 않아 연동할 claude.ai 백엔드가 없습니다.

```text theme={null}
Remote Control is only available when using Claude via api.anthropic.com.
```

**조치 방법:**

* `ANTHROPIC_BASE_URL`을 해제하고 세션을 재시작하세요.

### OAuth token revoked or expired

저장된 로그인이 더 이상 유효하지 않습니다.

```text theme={null}
OAuth token revoked · Please run /login
OAuth token has expired · Please run /login
```

**조치 방법:**

* `/login`을 실행하여 다시 로그인하세요.

### Login expired

Claude Code가 저장된 로그인을 갱신하려 했으나 OAuth 서비스가 토큰을 거부하여 자격 증명을 지웠습니다.

```text theme={null}
Login expired · Please run /login
```

**조치 방법:**

* `/login`을 실행하여 다시 로그인하세요.

### OAuth scope requirement

저장된 토큰이 최신 기능에 필요한 권한 스코프를 만족하지 못합니다.

```text theme={null}
OAuth token does not meet scope requirement: user:profile
```

**조치 방법:**

* `/login`을 실행하여 최신 스코프로 새 토큰을 받으세요.

### AWS credentials expired or invalid

AWS 세션 토큰이 만료되었거나 거부되었습니다.

```text theme={null}
AWS credentials expired or invalid · run /login and select "Claude Platform on AWS · refresh credentials", or run `aws sso login --profile myprofile` in another terminal · API Error: 401 ...
```

**조치 방법:**

* `aws sso login`을 실행하거나 `/login`에서 **Claude Platform on AWS · refresh credentials**를 선택하세요.

### AWS authentication failed

AWS 공급자가 403을 반환했거나 Amazon Bedrock이 401을 반환했습니다.

```text theme={null}
AWS authentication failed · run /login and select "Claude Platform on AWS · refresh credentials", or run `aws sso login --profile myprofile` in another terminal · if credentials are current, check AWS permissions and model access · API Error: 403 ...
```

**조치 방법:**

* 자격 증명을 갱신하거나 IAM 권한 및 모델 접근 설정을 확인하세요.

### AWS default-chain credential resolve timed out

AWS 기본 자격 증명 체인이 60초 이내에 자격 증명을 생성하지 못했습니다.

```text theme={null}
API Error: AWS default-chain credential resolve timed out
```

**조치 방법:**

* 셸에서 `aws sts get-caller-identity`를 실행하여 자격 증명 프로세스 문제를 확인하세요.

## 네트워크 및 연결 오류 (Network and connection errors)

Claude Code의 네트워크 요청이 목적지에 도달하지 못했거나 응답이 변형되었습니다.

### Unable to connect to API

API에 대한 TCP 연결이 실패했습니다.

```text theme={null}
Unable to connect to API. Check your internet connection
```

**조치 방법:**

* `curl -I https://api.anthropic.com`을 실행하여 통신이 가능한지 확인하세요.
* 프록시 환경인 경우 `HTTPS_PROXY`를 설정하세요.

### Socket is closed

스트리밍 응답 도중 연결이 닫혔습니다.

```text theme={null}
Socket is closed
```

**조치 방법:**

* Claude Code를 최신 버전으로 업데이트하면 자동으로 재시도합니다.

### Bedrock streaming response has an unexpected content-type

프록시가 Amazon Bedrock 스트리밍 응답의 `Content-Type`을 변형했습니다.

```text theme={null}
Bedrock streaming response has content-type "text/event-stream"; expected "application/vnd.amazon.eventstream"...
```

**조치 방법:**

* 게이트웨이가 응답 바이트와 헤더를 변형 없이 전달하도록 설정하거나 `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_GUARD=1`을 설정하세요.

### SSL certificate errors

네트워크의 프록시나 보안 장비가 TLS 트래픽을 가로채고 있으며 Claude Code가 이를 신뢰하지 않습니다.

```text theme={null}
SSL certificate error (UNABLE_TO_GET_ISSUER_CERT_LOCALLY)...
```

**조치 방법:**

* CA 번들을 내보내고 `NODE_EXTRA_CA_CERTS=/path/to/ca-bundle.pem`을 설정하세요.

### Host not allowed in a cloud session

클라우드 세션이나 루틴의 아웃바운드 HTTP 요청이 허용 목록에 의해 차단되었습니다.

```text theme={null}
HTTP 403
x-deny-reason: host_not_allowed
```

**조치 방법:**

* 클라우드 환경 설정에서 해당 도메인을 **Allowed domains**에 추가하세요.

<h3 id="couldnt-reconnect-to-your-remote-control-session">
  Couldn't reconnect to your Remote Control session
</h3>

```text theme={null}
Couldn't reconnect to your Remote Control session. Retry, or start a fresh session without --resume.
```

**조치 방법:**

* `/remote-control`을 실행하여 재연결을 시도하거나 `--resume` 없이 새 세션을 시작하세요.

## 요청 오류 (Request errors)

요청 내용과 관련된 오류입니다.

### Prompt is too long

대화와 첨부 파일이 모델의 컨텍스트 창을 초과했습니다.

```text theme={null}
Prompt is too long
```

**조치 방법:**

* `/compact`를 실행하여 대화를 요약하거나 `/clear`로 새로 시작하세요.

### Context exceeds the token limit

`/context` 출력 상단에 표시되는 경고입니다.

```text theme={null}
Context exceeds the 200k-token limit by 94k tokens — run /compact or /clear to continue.
```

**조치 방법:**

* `/compact` 또는 `/clear`를 실행하세요.

### Error during compaction: Conversation too long

`/compact` 자체가 수행될 여유 공간이 부족하여 실패했습니다.

```text theme={null}
Error during compaction: Conversation too long. Press esc twice to go up a few messages and try again.
```

**조치 방법:**

* Esc를 두 번 눌러 이전 메시지 시점으로 되돌아간 후 `/compact`를 실행하거나 `/clear`를 실행하세요.

### Request too large

원시 요청 바이트가 32MB 제한을 초과했습니다.

```text theme={null}
Request too large (max 32MB)...
```

**조치 방법:**

* `/compact`를 실행하여 누적된 이미지를 삭제하거나 대용량 파일은 직접 붙여넣는 대신 경로로 참조하세요.

### Image was too large

이미지 크기가 제한을 초과했습니다.

```text theme={null}
Image was too large...
```

**조치 방법:**

* 이미지 해상도를 줄이거나 잘라내어 다시 붙여넣으세요.

### Unable to resize image

이미지 리사이징 처리에 실패했습니다.

```text theme={null}
Unable to resize image...
```

**조치 방법:**

* PNG, JPEG, GIF, WebP 형식으로 변환하거나 직접 리사이징하세요.

### PDF errors

PDF를 처리할 수 없습니다.

```text theme={null}
PDF too large (max 100 pages, 20MB)...
```

**조치 방법:**

* 텍스트 추출 도구(`pdftotext`)를 사용하거나 암호를 해제하세요.

### Extra inputs are not permitted

프록시가 `anthropic-beta` 헤더를 제거하여 발생한 오류입니다.

```text theme={null}
API Error: 400 ... Extra inputs are not permitted ...
```

**조치 방법:**

* 게이트웨이가 `anthropic-beta` 헤더를 전달하도록 설정하거나 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`을 설정하세요.

<h3 id="theres-an-issue-with-the-selected-model">
  There's an issue with the selected model
</h3>

구성된 모델 이름을 인식할 수 없거나 접근 권한이 없습니다.

```text theme={null}
There's an issue with the selected model (claude-...).
```

**조치 방법:**

* `/model`을 실행하여 사용 가능한 모델을 선택하세요.

### Model is not a recognized model id

지정한 모델 이름이 유효하지 않습니다.

```text theme={null}
Model "claud-sonnet-5" is not a recognized model id. Did you mean 'claude-sonnet-5'?
```

**조치 방법:**

* 올바른 모델 별칭이나 ID를 입력하세요.

### Claude Opus is not available with the Claude Pro plan

현재 요금제에 선택한 모델이 포함되어 있지 않습니다.

```text theme={null}
Claude Opus is not available with the Claude Pro plan · Select a different model in /model
```

**조치 방법:**

* `/model`을 실행하여 요금제에 포함된 모델을 선택하세요.

<h3 id="model-is-restricted-by-your-organizations-settings">
  Model is restricted by your organization's settings
</h3>

조직 관리자가 해당 모델을 제한했습니다.

```text theme={null}
Model "claude-opus-4-8" is restricted by your organization's settings. Using claude-sonnet-4-6 instead.
```

**조치 방법:**

* `/model`을 실행하여 허용된 모델을 선택하세요.

### thinking.type.enabled is not supported for this model

Claude Code 버전이 너무 오래되어 최신 모델의 thinking 구성을 지원하지 못합니다.

```text theme={null}
API Error: 400 ... "thinking.type.enabled" is not supported for this model...
```

**조치 방법:**

* `claude update`를 실행하여 업데이트하세요.

### Thinking budget exceeds output limit

Thinking 예산이 최대 출력 토큰 수를 초과했습니다.

```text theme={null}
API Error: 400 ... max_tokens must be greater than thinking.budget_tokens
```

**조치 방법:**

* `MAX_THINKING_TOKENS`를 낮추거나 `CLAUDE_CODE_MAX_OUTPUT_TOKENS`를 높이세요.

### Tool use or thinking block mismatch

대화 기록의 도구 호출 및 thinking 블록 순서가 꼬였습니다.

```text theme={null}
API Error: 400 due to tool use concurrency issues. Run /rewind to recover the conversation.
```

**조치 방법:**

* `/rewind`를 실행하거나 Esc를 두 번 눌러 이전 체크포인트로 되돌아가세요.

### Usage Policy refusal

대화 내용이 이용 정책(Usage Policy) 검사를 트리거했습니다.

```text theme={null}
API Error: Claude Code is unable to respond to this request, which appears to violate our Usage Policy...
```

**조치 방법:**

* `/rewind`로 문제의 턴 이전으로 돌아가 프롬프트를 수정하거나 `/clear`로 새 세션을 시작하세요.

### Safety Measures flagged a cybersecurity topic

사이버 보안 주제 안전장치가 메시지를 감지했습니다.

```text theme={null}
API Error: Opus 4.8 has safety measures that flagged this message for a cybersecurity topic...
```

**조치 방법:**

* 정당한 연구인 경우 Cyber Verification Program에 신청하거나 `/feedback`을 보내세요.

## 설치 오류 (Installation errors)

### Installation was killed before it could finish

메모리 부족으로 설치 스크립트 프로세스가 종료되었습니다.

```text theme={null}
Installation was killed before it could finish (exit code 137). This usually means the system ran out of memory.
```

**조치 방법:**

* 메모리를 확보하거나 스왑 공간을 추가한 후 설치를 다시 실행하세요.

### The connection dropped while downloading the update

업데이트 바이너리 다운로드 중 연결이 끊어졌습니다.

```text theme={null}
The connection dropped while downloading the update (attempt 3/3: aborted)...
```

**조치 방법:**

* `claude update`를 다시 실행하세요.

## 커맨드라인 오류 (Command-line errors)

### Conflict between --bg and --print

`--bg`와 `-p`/`--print`를 함께 사용할 수 없습니다.

```text theme={null}
--bg and --print conflict...
```

**조치 방법:**

* `-p`를 제거하고 `claude --bg "<task>"` 형태로 실행하세요.

### The --json-schema value is not a valid JSON Schema

`--json-schema`에 전달된 값이 유효한 JSON Schema가 아닙니다.

```text theme={null}
Error: --json-schema is not a valid JSON Schema...
```

**조치 방법:**

* 진단 메시지에 지정된 스키마 오류를 수정하세요.

### Settings file exceeds the 2MiB limit

`--settings`로 전달한 파일 크기가 2MB를 초과합니다.

```text theme={null}
Error: Settings file exceeds the 2MiB limit: /path/to/settings.json
```

**조치 방법:**

* 2MB 이하의 정규 JSON 설정 파일 경로를 지정하세요.

### Workspace not trusted when starting Remote Control

신뢰하지 않은 디렉토리에서 Remote Control을 시작했습니다.

```text theme={null}
Error: Workspace not trusted. Please run `claude` in /Users/you/project first...
```

**조치 방법:**

* 해당 디렉토리에서 `claude`를 실행하여 워크스페이스 신뢰 다이얼로그를 수락한 후 다시 시작하세요.

### Could not import a server from Claude Desktop

Claude Desktop의 MCP 서버를 가져오지 못했습니다.

```text theme={null}
Could not import my server: Invalid name my server...
```

**조치 방법:**

* 서버 이름에 영문, 숫자, 하이픈, 언더스코어만 사용하도록 수정하세요.

### MCP permission prompt tool not found

`--permission-prompt-tool`에 전달된 MCP 도구를 찾을 수 없습니다.

```text theme={null}
Error: MCP tool mcp__permissions__approve (passed via --permission-prompt-tool) not found...
```

**조치 방법:**

* MCP 서버가 정상 연결되었는지, 도구 이름이 일치하는지 확인하세요.

### Diff is too large for ultrareview

diff 크기가 ultrareview 제한을 초과했습니다.

```text theme={null}
Diff is too large for ultrareview: 812 files, 96,410 lines changed...
```

**조치 방법:**

* 더 가까운 대상 브랜치를 지정하거나 변경 사항을 분할하세요.

### Failed to resume the conversation

저장된 트랜스크립트를 읽지 못했습니다.

```text theme={null}
Failed to resume the conversation.
```

**조치 방법:**

* `claude --resume <session-id>`를 다시 실행하거나 새로 시작하세요.

## 플러그인 오류 (Plugin errors)

### Marketplace is registered from an untrusted source

마켓플레이스 이름이 공식 Anthropic 마켓플레이스용으로 예약된 이름입니다.

```text theme={null}
Marketplace "claude-community" is registered from an untrusted source...
```

**조치 방법:**

* 공식 `github.com/anthropics` 리포지토리에서 마켓플레이스를 다시 추가하세요.

<h3 id="plugin-command-references-user-config">
  Plugin command references user\_config in a shell command
</h3>

플러그인 명령이 셸에 전달되는 형태로 `${user_config.*}`를 참조하고 있습니다.

```text theme={null}
Hook from plugin formatter@acme-tools references ${user_config.*} in a shell-form command...
```

**조치 방법:**

* exec 형식(`"args": [...]`)을 사용하거나 스크립트 내부에서 `$CLAUDE_PLUGIN_OPTION_<KEY>` 환경 변수를 읽도록 수정하세요.

## 도구 오류 (Tool errors)

### Agent would be spawned with zero tools

하위 에이전트의 `tools` 목록이 아무 도구로도 확인되지 않았습니다.

```text theme={null}
Agent 'code-reviewer' would be spawned with zero tools — refusing...
```

**조치 방법:**

* `tools` 목록의 도구 이름을 올바르게 수정하거나 `tools` 필드를 삭제하세요.

### File is covered by a Read deny rule

`Read` 거부 규칙이 적용된 경로에 대해 편집(Edit) 도구가 호출되었습니다.

```text theme={null}
File is covered by a Read deny rule in your permission settings and cannot be edited.
```

**조치 방법:**

* `/permissions`에서 `Read` 거부 규칙을 수정하거나 제거하세요.

### Memory index is over its read limit

`MEMORY.md` 메모리 인덱스가 읽기 한도(200줄 또는 25KB)를 초과했습니다.

```text theme={null}
Error: this write left the memory index at MEMORY.md at 214 lines, over its 200-line read limit...
```

**조치 방법:**

* Claude에게 `MEMORY.md`를 정리하도록 요청하거나 직접 편집하여 200줄 이하로 다듬으세요.

### pkill pattern matches the Claude Code process

`pkill` 패턴이 Claude Code 프로세스 자신과 일치하여 실행이 거부되었습니다.

```text theme={null}
pkill: refusing to run — this pattern matches the Claude CLI process (PID 12345)...
```

**조치 방법:**

* 대상 바이너리의 전체 경로를 지정하거나 `pkill -P $$`를 사용하세요.

## 백그라운드 세션 오류 (Background session errors)

### Commands refused in a background session

터미널이 연결되어 있지 않은 백그라운드 세션에서는 대화형 다이얼로그를 여는 명령을 실행할 수 없습니다.

```text theme={null}
Can't open MCP settings while no terminal is attached to this background session...
```

**조치 방법:**

* 에이전트 뷰에서 해당 세션에 연결(attach)한 후 명령을 다시 실행하거나 인수가 포함된 비대화형 명령(예: `/mcp reconnect <server>`)을 사용하세요.

### This session has no saved transcript

첫 응답이 완료되기 전에 중지된 백그라운드 세션입니다.

```text theme={null}
This session has no saved transcript — it was stopped before its first response finished...
```

**조치 방법:**

* `claude respawn <id>`를 실행하거나 에이전트 뷰에서 해당 행을 선택하고 Enter를 두 번 눌러 새로 시작하세요.

<h3 id="session-agent-no-longer-available">
  Session agent no longer available
</h3>

재개한 세션이 실행 중이던 커스텀 에이전트를 더 이상 찾을 수 없습니다.

```text theme={null}
This session was running agent 'code-reviewer', which is no longer available...
```

**조치 방법:**

* `.claude/agents/<name>.md` 파일에 해당 에이전트를 다시 생성하거나 `--agent <name>`으로 재개하세요.

### CLAUDE\_CODE\_PROCESS\_WRAPPER launcher errors

`CLAUDE_CODE_PROCESS_WRAPPER` 런처 값을 사용할 수 없습니다.

```text theme={null}
CLAUDE_CODE_PROCESS_WRAPPER: launcher `/opt/corp/launcher` is not an executable regular file
```

**조치 방법:**

* `exec "$@"`로 종료되는 실행 가능한 스크립트의 절대 경로를 지정하세요.

### EUNKNOWN when starting a background session

Windows 환경 정책이 프로세스 실행을 차단하여 발생합니다.

```text theme={null}
Couldn't reach the background service (spawn background service: EUNKNOWN: unknown error, uv_spawn)...
```

**조치 방법:**

* Claude Code 바이너리가 관리자 정책에서 허용되도록 관리자에게 요청하거나 최신 버전으로 업데이트하세요.

## 되감기 경고 (Rewind warnings)

<h3 id="restored-the-code-but-skipped-files">
  Restored the code, but skipped files
</h3>

`/rewind` 코드 복원 시 일부 파일이 복원 대상에서 건너뛰어졌습니다.

```text theme={null}
Restored the code, but skipped 2 files...
```

**조치 방법:**

* 건너뛰어진 심볼릭 링크나 변경된 파일의 내용을 직접 검토하세요.

## 구성 경고 (Configuration warnings)

### Workspace has not been trusted

신뢰하지 않은 워크스페이스에서 `.claude/settings.json` 등의 `permissions.allow` 규칙을 무시했습니다.

```text theme={null}
Ignoring 2 permissions.allow entries from .claude/settings.local.json: this workspace has not been trusted...
```

**조치 방법:**

* 해당 디렉토리에서 대화형으로 `claude`를 한 번 실행하여 워크스페이스 신뢰 다이얼로그를 수락하세요.

## 응답 품질 문제 (Response quality)

Claude의 응답 품질이 평소보다 떨어진다고 느껴진다면 다음을 확인하세요.

* **모델 선택**: `/model`을 실행하여 현재 설정된 모델이 의도한 모델인지 확인하세요.
* **작업 투입 수준 (Effort level)**: `/effort`를 실행하여 추론 수준이 적절한지 확인하고 높이세요.
* **컨텍스트 압박**: `/context`를 실행하여 컨텍스트 창 사용량을 확인하고 `/compact` 또는 `/clear`를 실행하세요.
* **오래된 지침**: `CLAUDE.md`가 너무 크거나 오래되었는지 확인하고 `/doctor`로 점검하세요.

응답이 잘못되었을 때는 프롬프트로 수정하는 것보다 `/rewind`로 이전 시점으로 되돌아가 구체적인 지침으로 프롬프트를 다시 작성하는 것이 훨씬 효과적입니다.

## 오류 보고하기 (Report an error)

오류가 지속되거나 해결되지 않는 경우:

* Claude Code 내부에서 `/feedback`을 실행하여 트랜스크립트와 설명을 Anthropic으로 전송하세요.
* 셸에서 `claude doctor`를 실행하거나 대화 중 `/doctor`를 실행하여 설정을 점검하세요.
* [status.claude.com](https://status.claude.com)에서 실시간 장애 여부를 확인하세요.
* GitHub의 [이슈 목록](https://github.com/anthropics/claude-code/issues)을 검색하세요.
