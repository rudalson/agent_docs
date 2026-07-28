> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 프로그래밍 방식으로 Claude Code 실행하기

> Agent SDK를 사용하여 CLI, Python 또는 TypeScript에서 프로그래밍 방식으로 Claude Code를 실행하세요.

[Agent SDK](/docs/en/agent-sdk/overview)는 Claude Code를 구동하는 동일한 도구, 에이전트 루프 및 컨텍스트 관리를 제공합니다. 스크립트 및 CI/CD를 위한 CLI로 사용할 수 있으며, 완전한 프로그래밍 제어를 위한 [Python](/docs/en/agent-sdk/python) 및 [TypeScript](/docs/en/agent-sdk/typescript) 패키지로 제공됩니다.

Claude Code를 비대화형 모드로 실행하려면 프롬프트 및 [CLI 옵션](/docs/en/cli-reference)과 함께 `-p`를 전달하세요:

```bash theme={null}
claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"
```

이 페이지는 CLI (`claude -p`)를 통한 Agent SDK 사용을 다룹니다. 구조화된 출력, 도구 승인 콜백 및 기본 메시지 개체를 제공하는 Python 및 TypeScript SDK 패키지에 대해서는 [전체 Agent SDK 문서](/docs/en/agent-sdk/overview)를 참조하세요.

## 기본 사용법

명령어를 비대화형으로 실행하려면 `claude` 명령에 `-p` (또는 `--print`) 플래그를 추가하세요. 다음을 포함하여 모든 [CLI 옵션](/docs/en/cli-reference)이 `-p`와 함께 작동합니다:

* [대화 이어가기](#continue-conversations)를 위한 `--continue`
* [도구 자동 승인](#auto-approve-tools)을 위한 `--allowedTools`
* [구조화된 출력](#get-structured-output)을 위한 `--output-format`

다음 예시는 코드베이스에 대해 Claude에게 질문을하고 응답을 출력합니다:

```bash theme={null}
claude -p "What does the auth module do?"
```

### Bare 모드로 더 빠르게 시작

`--bare`를 추가하면 훅, 스킬, 플러그인, MCP 서버, 자동 메모리 및 CLAUDE.md의 자동 검색을 건너뛰어 시작 시간을 단축할 수 있습니다. 이 플래그가 없으면 `claude -p`는 작업 디렉토리 또는 `~/.claude`에 구성된 내용을 포함하여 대화형 세션이 로드하는 동일한 [컨텍스트](/docs/en/how-claude-code-works#the-context-window)를 로드합니다.

Bare 모드는 모든 시스템에서 동일한 결과가 필요한 CI 및 스크립트에 유용합니다. 동료의 `~/.claude`에 있는 훅이나 프로젝트의 `.mcp.json`에 있는 MCP 서버는 bare 모드가 이를 절대 읽지 않으므로 실행되지 않습니다. 명시적으로 전달한 플래그만 적용됩니다.

다음 예시는 bare 모드에서 일회성 요약 작업을 실행하고 Read 도구를 사전 승인하여 권한 프롬프트 없이 호출이 완료되도록 합니다:

```bash theme={null}
claude --bare -p "Summarize this file" --allowedTools "Read"
```

Bare 모드에서 Claude는 Bash, 파일 읽기 및 파일 편집 도구에 접근할 수 있습니다. 플래그를 사용하여 필요한 컨텍스트를 전달하세요:

| 로드할 대상                 | 사용 방법                                                     |
| ----------------------- | ------------------------------------------------------- |
| 시스템 프롬프트 추가사항 | `--append-system-prompt`, `--append-system-prompt-file` |
| 설정                | `--settings <file-or-json>`                             |
| MCP 서버             | `--mcp-config <file-or-json>`                           |
| 커스텀 에이전트           | `--agents <json>`                                       |
| 플러그인                | `--plugin-dir <path>`, `--plugin-url <url>`             |

Bare 모드는 OAuth 및 키체인 읽기를 건너뜁니다. Anthropic 인증은 `ANTHROPIC_API_KEY` 또는 `--settings`에 전달된 JSON의 `apiKeyHelper`에서 와야 합니다. Amazon Bedrock, Google Cloud Agent Platform 및 Microsoft Foundry는 해당 제공자의 일반적인 자격 증명을 사용합니다.

<Note>
  `--bare`는 스크립트 및 SDK 호출에 권장되는 모드이며 향후 릴리스에서 `-p`의 기본값이 될 예정입니다.
</Note>

### 종료 시 백그라운드 작업

`claude -p` 실행 중에 Claude가 개발 서버나 워치 빌드와 같은 [백그라운드 Bash 작업](/docs/en/tools-reference#bash-tool-behavior)을 시작하는 경우, Claude가 최종 결과를 반환하고 stdin이 닫힌 후 약 5초 후에 해당 셸이 종료됩니다. 유예 기간을 두어 결과 바로 직후에 끝나는 작업이 여전히 출력을 전달할 수 있도록 합니다. v2.1.163 이전에는 절대 종료되지 않는 백그라운드 프로세스로 인해 `claude -p` 호출이 무기한 열려 있었습니다.

백그라운드 [서브에이전트](/docs/en/sub-agents) 및 워크플로는 결과가 최종 출력의 일부이므로 5초 유예 대상에서 제외되며, `claude -p`는 완료될 때까지 기다립니다. v2.1.182부터는 중단된 백그라운드 에이전트가 프로세스를 무기한 붙잡고 있지 않도록 기본적으로 대기 시간이 10분으로 제한됩니다. [`CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS`](/docs/en/env-vars)로 상한을 조정하거나 제한 없이 대기하려면 `0`으로 설정하세요.

`kill`, 프로세스 관리자 또는 세션을 닫는 SDK 호스트 등에서 SIGTERM으로 `claude -p` 실행을 중단하면 Claude Code는 진행 중인 턴을 중단하고 실행 중인 Bash 명령의 프로세스 트리를 종료하며 [`SessionEnd` 훅](/docs/en/hooks#sessionend)을 실행하고 종료 코드 143으로 종료합니다.

## 예시

다음 예시들은 일반적인 CLI 패턴을 보여줍니다. CI 및 기타 스크립트 호출의 경우 로컬에 구성된 내용을 수집하지 않도록 [`--bare`](#start-faster-with-bare-mode)를 추가하세요.

### Claude를 통해 데이터 파이프 연결

비대화형 모드는 stdin을 읽으므로 다른 명령줄 도구처럼 데이터를 파이프로 전달하고 응답을 다른 곳으로 리디렉션할 수 있습니다.

이 예시는 빌드 로그를 Claude로 파이프 연결하고 설명을 파일에 작성합니다:

```bash theme={null}
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt
```

`--output-format json`을 사용하면 응답 페이로드에 `total_cost_usd` 및 모델별 비용 분해 데이터가 포함되므로 스크립트 호출자가 [사용량 대시보드](/docs/en/costs)를 참조하지 않고도 호출당 지출을 추적할 수 있습니다.

<Note>
  Claude Code v2.1.128부터 파이프된 stdin의 최대 크기는 10MB입니다. 제한을 초과하면 Claude Code가 명확한 오류와 함께 0이 아닌 상태 코드로 종료됩니다. 더 큰 입력으로 작업하려면 파이프로 전달하는 대신 내용을 파일에 쓰고 프롬프트에서 해당 파일 경로를 참조하세요.
</Note>

Claude Code를 시작한 프로세스가 해당 종단을 연결 해제하여 Claude Code가 stdin을 읽을 수 없는 경우, Claude Code는 stderr에 경고를 출력하고 명령줄의 프롬프트로 계속 진행합니다. v2.1.211 이전에는 Windows에서 읽을 수 없는 stdin이 세션을 멈추게 하거나 출력 없이 자동으로 종료되도록 만들었습니다.

### 빌드 스크립트에 Claude 추가

비대화형 호출을 스크립트에 래핑하여 Claude를 프로젝트 전용 린터 또는 리뷰어로 사용할 수 있습니다.

이 `package.json` 스크립트는 `main`과의 차이점(diff)을 Claude로 파이프 연결하고 오타를 보고하도록 요청합니다. 차이점을 파이프 연결하면 Claude가 이를 읽기 위해 Bash 권한을 필요로 하지 않으며 이스케이프된 큰따옴표를 통해 스크립트가 Windows에서도 이식성을 유지합니다:

```json theme={null}
{
  "scripts": {
    "lint:claude": "git diff main | claude -p \"you are a typo linter. for each typo in this diff, report filename:line on one line and the issue on the next. return nothing else.\""
  }
}
```

### 구조화된 출력 얻기

`--output-format`을 사용하여 응답 반환 방식을 제어하세요:

* `text` (기본값): 일반 텍스트 출력
* `json`: 결과, 세션 ID 및 메타데이터가 포함된 구조화된 JSON
* `stream-json`: 실시간 스트리밍을 위한 줄바꿈으로 구분된 JSON

다음 예시는 `result` 필드에 텍스트 결과가 포함된 세션 메타데이터와 함께 프로젝트 요약을 JSON으로 반환합니다:

```bash theme={null}
claude -p "Summarize this project" --output-format json
```

특정 스키마를 따르는 출력을 얻으려면 `--output-format json`을 `--json-schema` 및 [JSON Schema](https://json-schema.org/) 정의와 함께 사용하세요. 응답에는 요청에 대한 메타데이터(세션 ID, 사용량 등)와 `structured_output` 필드의 구조화된 출력이 포함됩니다.

다음 예시는 함수 이름을 추출하여 문자열 배열로 반환합니다:

```bash theme={null}
claude -p "Extract the main function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}'
```

값이 유효한 JSON Schema가 아닌 경우 `claude`는 `Error: --json-schema is not a valid JSON Schema` 및 검증기의 진단 내용과 함께 종료됩니다. Claude Code는 `"format": "email"`과 같이 `format` 키워드를 사용하는 스키마를 수락하지만 `format`을 주석으로 처리하고 강제하지는 않습니다. v2.1.205 이전에는 Claude Code가 잘못된 스키마를 자동으로 무시하고 비구조화된 텍스트를 반환했으며 `format`이 포함된 모든 스키마를 잘못된 것으로 취급했습니다.

<Tip>
  응답을 파싱하고 특정 필드를 추출하려면 [jq](https://jqlang.org/)와 같은 도구를 사용하세요:

  ```bash theme={null}
  # 텍스트 결과 추출
  claude -p "Summarize this project" --output-format json | jq -r '.result'

  # 구조화된 출력 추출
  claude -p "Extract function names from auth.py" \
    --output-format json \
    --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
    | jq '.structured_output'
  ```
</Tip>

### 응답 스트리밍

토큰이 생성될 때 받아보려면 `--output-format stream-json`을 `--verbose` 및 `--include-partial-messages`와 함께 사용하세요. 각 줄은 이벤트를 나타내는 JSON 개체입니다:

```bash theme={null}
claude -p "Explain recursion" --output-format stream-json --verbose --include-partial-messages
```

스트림의 마지막 줄은 최종 응답 텍스트, 비용 및 세션 메타데이터가 포함된 `result` 메시지입니다.

소비자가 스트림을 천천히 읽는 경우 Claude Code는 대기 중인 출력이 비워질 때까지 기다린 후 종료하며, 대기 중인 양에 맞춰 대기 시간을 조절하고 최대 30초로 제한합니다. v2.1.214 이전에는 종료 대기가 약 2초로 제한되어 큰 응답의 끝부분이 잘릴 수 있었으며 v2.1.208 이전에는 큰 응답을 파이프 연결할 때 마지막 줄이 잘리고 `result` 메시지가 누락될 수 있었습니다.

[서브에이전트](/docs/en/sub-agents)의 메시지는 `parent_tool_use_id` 필드가 서브에이전트를 생성한 도구 호출의 ID인 `assistant` 및 `user` 메시지로 스트림에 나타납니다. 메인 대화의 메시지는 해당 필드에 `null`을 가집니다.

기본적으로 Claude Code는 서브에이전트의 `tool_use` 및 `tool_result` 블록만 내보냅니다. 서브에이전트 텍스트 및 사고(thinking) 블록도 내보내어 각 서브에이전트의 트랜스크립트를 재구성하려면 [`--forward-subagent-text`](/docs/en/cli-reference#cli-flags)를 전달하거나 [`CLAUDE_CODE_FORWARD_SUBAGENT_TEXT`](/docs/en/env-vars)를 설정하세요. 이 기능은 Claude Code v2.1.211 이상이 필요합니다.

다음 예시는 [jq](https://jqlang.org/)를 사용하여 텍스트 델타를 필터링하고 스트리밍 텍스트만 표시합니다. `-r` 플래그는 원시 문자열(따옴표 없음)을 출력하고 `-j`는 줄바꿈 없이 결합하여 토큰이 연속적으로 스트리밍되도록 합니다:

```bash theme={null}
claude -p "Write a poem" --output-format stream-json --verbose --include-partial-messages | \
  jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

API 요청이 재시도 가능한 오류로 실패하면 Claude Code는 재시도 전에 `system/api_retry` 이벤트를 내보냅니다. 이를 사용하여 재시도 진행 상황을 노출하거나 커스텀 백오프 로직을 구현할 수 있습니다.

| 필드            | 유형            | 설명                                                                                                                                                                                            |
| ---------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `type`           | `"system"`      | 메시지 유형                                                                                                                                                                                           |
| `subtype`        | `"api_retry"`   | 이 이벤트를 재시도 이벤트로 식별                                                                                                                                                                       |
| `attempt`        | integer         | 1부터 시작하는 현재 시도 횟수                                                                                                                                                                  |
| `max_retries`    | integer         | 허용된 총 재시도 횟수                                                                                                                                                                                |
| `retry_delay_ms` | integer         | 다음 시도까지의 밀리초 단위 대기 시간                                                                                                                                                                    |
| `error_status`   | integer or null | HTTP 상태 코드, 또는 HTTP 응답이 없는 연결 오류의 경우 `null`                                                                                                                                |
| `error`          | string          | 오류 범주: `authentication_failed`, `oauth_org_not_allowed`, `billing_error`, `rate_limit`, `overloaded`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, 또는 `unknown` |
| `uuid`           | string          | 고유 이벤트 식별자                                                                                                                                                                                |
| `session_id`     | string          | 이벤트가 속한 세션                                                                                                                                                                           |

`system/init` 이벤트는 모델, 도구, MCP 서버 및 로드된 플러그인을 포함한 세션 메타데이터를 보고합니다. 시작 이벤트가 선행되지 않는 한 스트림의 첫 번째 이벤트입니다:

* [`CLAUDE_CODE_SYNC_PLUGIN_INSTALL`](/docs/en/env-vars)이 설정되어 있을 때의 `plugin_install` 이벤트.
* 설정된 [`SessionStart`](/docs/en/hooks#sessionstart) 또는 [`Setup`](/docs/en/hooks#setup) 훅이 실행되는 동안의 [`hook_started`, `hook_progress` 및 `hook_response` 이벤트](/docs/en/agent-sdk/typescript#sdkhookstartedmessage). 이러한 이벤트는 훅이 생성함에 따라 스트리밍됩니다. Claude Code v2.1.169부터 v2.1.203까지는 훅이 완료된 후 `system/init`에 앞서 한꺼번에 전달되었으나 v2.1.204에서 라이브 전달이 복원되었습니다.

또한 이벤트는 `interrupt_receipt_v1`과 같이 이 Claude Code 버전이 구현하는 프로토콜 동작의 이름을 지정하는 문자열의 선택적 `capabilities` 배열을 전달합니다. 버전 문자열을 비교하는 대신 이를 확인하여 기능을 감지하고 인식하지 못하는 값은 무시하세요. 이 필드는 Claude Code v2.1.205 이상이 필요하며 이전 버전에는 없습니다. 기능 목록은 [`SDKSystemMessage`](/docs/en/agent-sdk/typescript#sdksystemmessage)를 참조하세요.

플러그인이 로드되지 않은 경우 CI를 실패 처리하려면 플러그인 필드를 사용하세요:

| 필드           | 유형  | 설명                                                                                                                                                                                                                                                                                  |
| --------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `plugins`       | array | 성공적으로 로드된 플러그인 목록, 각각 `name` 및 `path` 포함                                                                                                                                                                                                                                |
| `plugin_errors` | array | 플러그인 로드 시 오류 목록, 각각 `plugin`, `type` 및 `message` 포함. 충족되지 않은 종속성 버전 및 누락된 경로나 잘못된 아카이브와 같은 `--plugin-dir` 로드 실패를 포함합니다. 영향 받는 플러그인은 강등되며 `plugins`에서 제외됩니다. 오류가 없으면 키가 생략됩니다 |

[`CLAUDE_CODE_SYNC_PLUGIN_INSTALL`](/docs/en/env-vars)이 설정되어 있으면 Claude Code는 첫 번째 턴 전에 마켓플레이스 플러그인이 설치되는 동안 `system/plugin_install` 이벤트를 내보냅니다. 자체 UI에서 설치 진행 상황을 표시하는 데 사용하세요.

| 필드        | 유형                                                     | 설명                                                                                                    |
| ------------ | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `type`       | `"system"`                                               | 메시지 유형                                                                                                   |
| `subtype`    | `"plugin_install"`                                       | 이 이벤트를 플러그인 설치 이벤트로 식별                                                                      |
| `status`     | `"started"`, `"installed"`, `"failed"`, 또는 `"completed"` | `started` 및 `completed`는 전체 설치의 시작과 끝을 감쌉니다; `installed` 및 `failed`는 개별 마켓플레이스를 보고합니다 |
| `name`       | string, optional                                         | 마켓플레이스 이름 (`installed` 및 `failed`에 존재)                                                          |
| `error`      | string, optional                                         | 실패 메시지 (`failed`에 존재)                                                                           |
| `uuid`       | string                                                   | 고유 이벤트 식별자                                                                                        |
| `session_id` | string                                                   | 이벤트가 속한 세션                                                                                   |

콜백 및 메시지 개체를 사용한 프로그래밍 방식 스트리밍에 대해서는 Agent SDK 문서의 [실시간 응답 스트리밍](/docs/en/agent-sdk/streaming-output)을 참조하세요.

### 도구 자동 승인

Claude가 확인 없이 특정 도구를 사용하도록 허용하려면 `--allowedTools`를 사용하세요. 다음 예시는 테스트 스위트를 실행하고 실패 항목을 수정하면서 Claude가 확인을 구하지 않고 Bash 명령을 실행하고 파일을 읽고/편집할 수 있도록 허용합니다:

```bash theme={null}
claude -p "Run the test suite and fix any failures" \
  --allowedTools "Bash,Read,Edit"
```

개별 도구를 나열하는 대신 전체 세션에 대한 기준선을 설정하려면 [권한 모드](/docs/en/permission-modes)를 전달하세요. `dontAsk`는 `permissions.allow` 규칙이나 [읽기 전용 명령 집합](/docs/en/permissions#read-only-commands)에 없는 항목을 거부하며, 통제된 CI 실행에 유용합니다. `AskUserQuestion`, 커넥터 도구([조직이 `ask`로 설정한 항목](/docs/en/mcp#organization-controls-on-connector-tools)) 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 허용 규칙이 일치하더라도 거부됩니다.

`acceptEdits`를 사용하면 Claude가 확인 없이 파일을 쓸 수 있으며 `mkdir`, `touch`, `mv`, `cp`와 같은 일반적인 파일 시스템 명령도 자동 승인합니다. 다른 셸 명령 및 네트워크 요청에는 여전히 `--allowedTools` 항목이나 `permissions.allow` 규칙이 필요하며, 그렇지 않으면 실행 시 중단됩니다:

```bash theme={null}
claude -p "Apply the lint fixes" --permission-mode acceptEdits
```

### 커밋 생성

이 예시는 스테이징된 변경 사항을 검토하고 적절한 메시지와 함께 커밋을 생성합니다:

```bash theme={null}
claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

`--allowedTools` 플래그는 [권한 규칙 구문](/docs/en/settings#permission-rule-syntax)을 사용합니다. 뒤따르는 ` *`는 접두사 일치를 활성화하므로 `Bash(git diff *)`는 `git diff`로 시작하는 모든 명령을 허용합니다. `*` 앞의 공백이 중요합니다: 공백이 없으면 `Bash(git diff*)`가 `git diff-index`와도 일치하게 됩니다.

<Note>
  사용자가 호출한 [스킬](/docs/en/skills) 및 커스텀 명령은 `-p` 모드에서 작동합니다: 프롬프트 문자열에 `/skill-name`을 포함하면 Claude Code가 실행 전에 이를 확장합니다. `/login`과 같이 터미널 인터페이스에서만 실행되는 내장 명령은 `-p` 모드에서 사용할 수 없습니다. `/model`, `/effort`, `/fast`, `/color`, `/rename`은 `/model sonnet`과 같이 값을 인수로 받아들이고, 인수 없는 `/mcp`는 서버 상태의 텍스트 요약을 출력합니다. 이러한 형태는 Claude Code v2.1.205 이상이 필요하며 각 명령의 [사용 가능 참고사항](/docs/en/commands#all-commands)을 따릅니다. `-p` 호출에서 설정을 변경하려면 `/config`에 `key=value`를 전달하세요 (예: `/config thinking=false`).
</Note>

### 시스템 프롬프트 커스텀

Claude Code의 기본 동작을 유지하면서 지침을 추가하려면 `--append-system-prompt`를 사용하세요. 이 예시는 PR diff를 Claude로 파이프 연결하고 보안 취약점을 검토하도록 지시합니다:

```bash theme={null}
gh pr diff "$1" | claude -p \
  --append-system-prompt "You are a security engineer. Review for vulnerabilities." \
  --output-format json
```

기본 프롬프트를 완전히 대체하는 `--system-prompt`를 포함한 자세한 옵션은 [시스템 프롬프트 플래그](/docs/en/cli-reference#system-prompt-flags)를 참조하세요.

### 대화 이어가기

가장 최근 대화를 이어가려면 `--continue`를 사용하고, 특정 대화를 이어가려면 세션 ID와 함께 `--resume`을 사용하세요. 이 예시는 검토를 실행한 다음 후속 프롬프트를 보냅니다:

```bash theme={null}
# 첫 번째 요청
claude -p "Review this codebase for performance issues"

# 가장 최근 대화 이어가기
claude -p "Now focus on the database queries" --continue
claude -p "Generate a summary of all issues found" --continue
```

여러 대화를 실행 중인 경우 세션 ID를 캡처하여 특정 대화를 재개하세요:

```bash theme={null}
session_id=$(claude -p "Start a review" --output-format json | jq -r '.session_id')
claude -p "Continue that review" --resume "$session_id"
```

동일한 디렉토리에서 두 명령을 모두 실행하세요. 세션 ID 조회 범주는 현재 프로젝트 디렉토리 및 해당 git 워크트리로 제한됩니다. 전체 범위 규칙은 [세션 재개](/docs/en/sessions#resume-a-session)를 참조하세요.

## 다음 단계

* [Agent SDK 빠른 시작](/docs/en/agent-sdk/quickstart): Python 또는 TypeScript로 첫 번째 에이전트 빌드하기
* [CLI 레퍼런스](/docs/en/cli-reference): 모든 CLI 플래그 및 옵션
* [GitHub Actions](/docs/en/github-actions): GitHub 워크플로에서 Agent SDK 사용하기
* [GitLab CI/CD](/docs/en/gitlab-ci-cd): GitLab 파이프라인에서 Agent SDK 사용하기
