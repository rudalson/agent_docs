> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# CLI 레퍼런스

> 명령 및 플래그를 포함하여 Claude Code 커맨드 라인 인터페이스에 대한 완전한 참조 가이드입니다.

## CLI 명령

다음 명령을 사용하여 세션을 시작하고, 콘텐츠를 파이프 처리하며, 대화를 재개하고, 업데이트를 관리할 수 있습니다.

| 명령 | 설명 | 예시 |
| :--- | :--- | :--- |
| `claude` | 대화형 세션 시작 | `claude` |
| `claude "query"` | 초기 프롬프트와 함께 대화형 세션 시작 | `claude "explain this project"` |
| `claude -p "query"` | SDK를 통해 쿼리 실행 후 종료 | `claude -p "explain this function"` |
| `cat file \| claude -p "query"` | 파이프 처리된 콘텐츠 처리 | `cat logs.txt \| claude -p "explain"` |
| `claude -c` | 현재 디렉토리에서 가장 최근 대화 계속하기 | `claude -c` |
| `claude -c -p "query"` | SDK를 통해 이전 대화 계속하기 | `claude -c -p "Check for type errors"` |
| `claude -r "<session>" "query"` | ID 또는 이름으로 세션 재개 | `claude -r "auth-refactor" "Finish this PR"` |
| `claude update` | 최신 버전으로 업데이트 | `claude update` |
| `claude gateway` | Amazon Bedrock, Google Cloud Agent Platform 또는 Microsoft Foundry의 Claude Code 전면에 SSO 및 정책을 배포하는 관리자를 위해 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 서버를 시작합니다. [`gateway.yaml`](/docs/en/claude-apps-gateway-config)을 가리키는 `--config`가 필요합니다. Claude Code v2.1.195 이상에서 이용 가능합니다. | `claude gateway --config gateway.yaml` |
| `claude install [version]` | 네이티브 바이너리를 설치하거나 재설치합니다. `2.1.118`과 같은 버전 문자열이나 `stable` 또는 `latest`를 인수로 받습니다. [특정 버전 설치](/docs/en/setup#install-a-specific-version)를 참조하세요. | `claude install stable` |
| `claude auth login` | Anthropic 계정에 로그인합니다. `--email`을 사용하여 이메일 주소를 미리 채우고, `--sso`로 SSO 인증을 강제하며, `--console`로 Claude 구독 대신 API 사용량 청구를 위한 Anthropic Console로 로그인합니다. | `claude auth login --console` |
| `claude auth logout` | Anthropic 계정에서 로그아웃합니다. | `claude auth logout` |
| `claude auth status` | 인증 상태를 JSON으로 표시합니다. 사람이 읽을 수 있는 출력을 얻으려면 `--text`를 사용하세요. 로그인 상태이면 종료 코드 0, 그렇지 않으면 1로 종료합니다. | `claude auth status` |
| `claude agents` | 병렬 백그라운드 세션을 모니터링하고 발송할 수 있도록 [에이전트 뷰](/docs/en/agent-view)를 엽니다. 해당 디렉토리에서 시작된 세션만 표시하려면 `--cwd <path>`를 사용하고, 스크립팅을 위해 활성 세션을 JSON 배열로 출력하려면 `--json`을 사용하세요 (`--json --all`은 완료된 백그라운드 세션도 포함). `--permission-mode`, `--model`, `--effort` 또는 `--agent`를 전달하여 [발송되는 세션의 기본값](/docs/en/agent-view#permission-mode-model-and-effort)을 설정할 수 있습니다. 최상위 `claude` 명령과 마찬가지로 `--settings`, `--add-dir`, `--plugin-dir` 및 `--mcp-config`를 허용합니다. 에이전트 뷰를 열려면 대화형 터미널이 필요합니다. | `claude agents --json` |
| `claude attach <id>` | 이 터미널에서 [백그라운드 세션](/docs/en/agent-view#manage-sessions-from-the-shell)에 연결합니다. | `claude attach 7c5dcf5d` |
| `claude auto-mode defaults` | 내장된 [자동 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 분류기 규칙을 JSON으로 출력합니다. 설정이 적용된 유효 구성을 보려면 `claude auto-mode config`를 사용하세요. {/* min-version: 2.1.208 */}`--label <prefix>`는 해당 접두사로 시작하는 라벨이 있는 규칙만 대소문자 구분 없이 일치시켜 출력합니다. Claude Code v2.1.208 이상이 필요합니다. | `claude auto-mode defaults --label 'Git Destructive'` |
| `claude auto-mode reset` | {/* min-version: 2.1.212 */}사용자 설정 파일에서 `autoMode` 섹션을 제거하여 기본 [자동 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 구성을 복원합니다. 작성하기 전에 확인 프롬프트를 표시하며, 프롬프트를 건너뛰려면 `-y`/`--yes`를 전달하세요. [관리형 설정](/docs/en/server-managed-settings) 또는 `--settings` 플래그의 규칙은 여전히 적용됩니다. Claude Code v2.1.212 이상이 필요합니다. [기본값 및 유효 구성 확인](/docs/en/auto-mode-config#inspect-the-defaults-and-your-effective-config)을 참조하세요. | `claude auto-mode reset --yes` |
| `claude daemon status` | 진단을 위해 백그라운드 세션 [수퍼바이저](/docs/en/agent-view#the-supervisor-process)의 상태, 버전, 소켓 디렉토리 및 워커 수를 출력합니다. 수퍼바이저가 실행 중이지 않으면 1로 종료합니다. | `claude daemon status` |
| `claude daemon stop --any` | 백그라운드 세션 [수퍼바이저](/docs/en/agent-view#the-supervisor-process) 및 수퍼바이저가 호스팅하는 세션을 중지합니다. 백그라운드 세션을 계속 실행하여 다음 수퍼바이저가 다시 연결할 수 있도록 하려면 `--keep-workers`를 전달하세요. `--any`는 기본값인 온디맨드 수퍼바이저 중지를 확인합니다. [응답하지 않는 수퍼바이저](/docs/en/agent-view#agent-view-says-the-background-service-did-not-respond)에서 복구할 때 사용하세요. | `claude daemon stop --any --keep-workers` |
| `claude doctor` | 세션을 시작하지 않고 터미널에서 설치 상태, 설정 파일 검증 오류, Remote Control 자격 여부 등 읽기 전용 설치 및 설정 진단 정보를 출력합니다. 수정 사항도 적용할 수 있는 세션 내 설정 점검의 경우 [`/doctor`](/docs/en/commands#all-commands)를 실행하세요. | `claude doctor` |
| `claude logs <id>` | [백그라운드 세션](/docs/en/agent-view#manage-sessions-from-the-shell)의 최근 출력을 출력합니다. | `claude logs 7c5dcf5d` |
| `claude mcp` | Model Context Protocol (MCP) 서버를 구성합니다. | [Claude Code MCP 문서](/docs/en/mcp)를 참조하세요. |
| `claude mcp login <name>` | {/* min-version: 2.1.186 */}대화형 `/mcp` 패널을 열지 않고 구성된 MCP 서버의 OAuth 흐름을 실행합니다. HTTP, SSE 및 claude.ai 커넥터 서버에서 작동합니다. SSH 접속 환경에서는 브라우저를 열지 않고 인증 URL을 출력하려면 `--no-browser`를 추가한 다음 리디렉션 URL을 프롬프트에 다시 붙여넣으세요. Claude Code v2.1.186 이상이 필요합니다. [커맨드 라인에서 인증하기](/docs/en/mcp#authenticate-from-the-command-line)를 참조하세요. | `claude mcp login sentry` |
| `claude mcp logout <name>` | {/* min-version: 2.1.186 */}MCP 서버에 대해 저장된 OAuth 자격 증명을 삭제합니다. Claude Code v2.1.186 이상이 필요합니다. | `claude mcp logout sentry` |
| `claude plugin` | Claude Code [플러그인](/docs/en/plugins)을 관리합니다. 별칭: `claude plugins`. 하위 명령은 [플러그인 레퍼런스](/docs/en/plugins-reference#cli-commands-reference)를 참조하세요. | `claude plugin install code-review@claude-plugins-official` |
| `claude project purge [path]` | 프로젝트에 대한 모든 로컬 Claude Code 상태(트랜스크립트, 작업 목록, 디버그 로그, 파일 편집 기록, 프롬프트 기록 줄 및 `~/.claude.json` 내 프로젝트 항목)를 삭제합니다. 대화형 목록에서 선택하려면 `[path]`를 생략하세요. 플래그: 미리보기 `--dry-run`, 확인 건너뛰기 `-y`/`--yes`, 각 항목 확인 `-i`/`--interactive`, 모든 프로젝트 대상 `--all`. [로컬 데이터 지우기](/docs/en/claude-directory#clear-local-data)를 참조하세요. | `claude project purge ~/work/repo --dry-run` |
| `claude remote-control` | Claude.ai 또는 Claude 앱에서 Claude Code를 제어할 수 있는 [Remote Control](/docs/en/remote-control) 서버를 시작합니다. 서버 모드로 실행됩니다 (로컬 대화형 세션 없음). [서버 모드 플래그](/docs/en/remote-control#start-a-remote-control-session)를 참조하세요. | `claude remote-control --name "My Project"` |
| `claude respawn <id>` | [백그라운드 세션](/docs/en/agent-view#manage-sessions-from-the-shell)을 대화 내용을 유지한 채 실행 중이거나 중지된 상태에서 재시작합니다. 업데이트된 Claude Code 바이너리를 적용하는 등 모든 실행 중인 세션을 재시작하려면 `--all`을 사용하세요. | `claude respawn 7c5dcf5d` |
| `claude rm <id>` | 목록에서 [백그라운드 세션](/docs/en/agent-view#manage-sessions-from-the-shell)을 제거합니다. 대화 트랜스크립트는 로컬 머신에 남아 있어 `claude --resume`을 통해 이용 가능합니다. | `claude rm 7c5dcf5d` |
| `claude setup-token` | CI 및 스크립트용 장기 OAuth 토큰을 생성합니다. 저장하지 않고 터미널에 토큰을 출력합니다. Claude 구독이 필요합니다. [장기 토큰 생성하기](/docs/en/authentication#generate-a-long-lived-token)를 참조하세요. | `claude setup-token` |
| `claude stop <id>` | [백그라운드 세션](/docs/en/agent-view#manage-sessions-from-the-shell)을 중지합니다. `claude kill`도 지원합니다. | `claude stop 7c5dcf5d` |
| `claude ultrareview [target]` | [ultrareview](/docs/en/ultrareview#run-ultrareview-non-interactively)를 비대화형으로 실행합니다. 발견 항목을 stdout으로 출력하며 성공 시 0, 실패 시 1로 종료합니다. 원시 페이로드를 얻으려면 `--json`을 사용하고 기본 30분 한도를 변경하려면 `--timeout <minutes>`를 사용하세요. | `claude ultrareview 1234 --json` |

하위 명령을 오타로 입력하면 Claude Code가 가장 유사한 일치 항목을 제안하고 세션을 시작하지 않고 종료합니다. 예를 들어 `claude udpate`는 `Did you mean claude update?`를 출력합니다.

{/* min-version: 2.1.199 */}v2.1.199부터 `claude --dangerously-skip-permissions daemon <subcommand>`는 `daemon` 하위 명령을 실행합니다. 이전 버전에서는 `daemon <subcommand>`를 새 대화형 세션의 프롬프트로 처리했으므로 플래그가 먼저 나타나는 경우 하위 명령이 실행되지 않았습니다 (`claude`에 플래그가 포함되도록 별칭이 지정된 일반적인 설정). 오직 선두의 `--dangerously-skip-permissions` 또는 `--allow-dangerously-skip-permissions`만 이러한 방식으로 `daemon`으로 라우팅하며, 다른 선두 플래그는 여전히 대화형 세션을 시작합니다.

## CLI 플래그

이러한 커맨드 라인 플래그로 Claude Code의 동작을 커스터마이징하세요. `claude --help`가 모든 플래그를 나열하지는 않으므로 `--help`에 표시되지 않는다고 해서 플래그를 사용할 수 없는 것은 아닙니다.

| 플래그 | 설명 | 예시 |
| :--- | :--- | :--- |
| `--add-dir` | Claude가 파일을 읽고 편집할 수 있도록 추가 작업 디렉토리를 추가합니다. 파일 액세스 권한이 부여됩니다. 대부분의 `.claude/` 구성은 이러한 디렉토리에서 [탐색되지 않습니다](/docs/en/permissions#additional-directories-grant-file-access-not-configuration). 각 경로가 디렉토리로 존재하는지 검증합니다. 세션 간에 이 디렉토리 지정을 유지하려면 설정에서 [`permissions.additionalDirectories`](/docs/en/settings#permission-settings)를 설정하세요. | `claude --add-dir ../apps ../lib` |
| `--advisor <model>` | 모델 별칭(`opus` 또는 `sonnet`) 또는 전체 모델 ID를 사용하여 이 세션에 대해 서버 측 [어드바이저 도구](/docs/en/advisor)를 활성화합니다. 세션의 `advisorModel` 설정보다 우선합니다. {/* min-version: 2.1.210 */}[Claude Code는 Fable 5를 어드바이저로 제공하지 않습니다](/docs/en/advisor#enable-the-advisor): `claude --advisor fable`은 오류와 함께 종료됩니다. | `claude --advisor opus` |
| `--agent` | 현재 세션에 사용할 에이전트를 지정합니다 (`agent` 설정을 재정의합니다). | `claude --agent my-custom-agent` |
| `--agents` | JSON을 통해 커스텀 하위 에이전트를 동적으로 정의합니다. 하위 에이전트 [frontmatter](/docs/en/sub-agents#supported-frontmatter-fields)와 동일한 필드 이름을 사용하며 에이전트 지침을 위한 `prompt` 필드가 추가됩니다. | `claude --agents '{"reviewer":{"description":"Reviews code","prompt":"You are a code reviewer"}}'` |
| `--allow-dangerously-skip-permissions` | 시작 모드로 지정하지 않고 `Shift+Tab` 모드 순환에 `bypassPermissions`를 추가합니다. `plan`과 같은 다른 모드에서 시작하여 나중에 `bypassPermissions`로 전환할 수 있습니다. [권한 모드](/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode)를 참조하세요. | `claude --permission-mode plan --allow-dangerously-skip-permissions` |
| `--allowedTools`, `--allowed-tools` | 권한 확인 프롬프트 없이 실행되는 도구입니다. 패턴 일치는 [권한 규칙 구문](/docs/en/settings#permission-rule-syntax)을 참조하세요. 사용할 수 있는 도구를 제한하려면 대신 `--tools`를 사용하세요. | `"Bash(git log *)" "Bash(git diff *)" "Read"` |
| `--append-subagent-system-prompt` | {/* min-version: 2.1.205 */}중첩된 하위 에이전트를 포함한 모든 [하위 에이전트](/docs/en/sub-agents)의 시스템 프롬프트 끝에 커스텀 텍스트를 추가합니다. `-p`를 사용한 비대화형 모드에서만 적용됩니다. Claude Code v2.1.205 이상이 필요합니다. | `claude -p --append-subagent-system-prompt "Cite file paths in every answer" "query"` |
| `--append-system-prompt` | 기본 시스템 프롬프트 끝에 커스텀 텍스트를 추가합니다. | `claude --append-system-prompt "Always use TypeScript"` |
| `--append-system-prompt-file` | 파일에서 추가 시스템 프롬프트 텍스트를 로드하여 기본 프롬프트 끝에 추가합니다. | `claude --append-system-prompt-file ./extra-rules.txt` |
| `--ax-screen-reader` | {/* min-version: 2.1.181 */}스크린 리더 친화적 출력(장식 테두리나 애니메이션이 없는 플랫 텍스트)을 렌더링합니다. 클래식 렌더러를 강제 적용하므로 [`tui`](/docs/en/settings#available-settings) 설정이 적용되지 않으며, 연결된 [백그라운드 세션](/docs/en/agent-view)은 여전히 전체 화면으로 렌더링됩니다. [`CLAUDE_AX_SCREEN_READER`](/docs/en/env-vars) 및 [`axScreenReader`](/docs/en/settings#available-settings) 설정보다 우선합니다. Claude Code v2.1.181 이상이 필요합니다. | `claude --ax-screen-reader` |
| `--bare` | 최소 모드: 훅, 스킬, 플러그인, MCP 서버, 자동 메모리, CLAUDE.md의 자동 탐색을 건너뛰어 스크립트 호출이 더 빠르게 시작됩니다. Claude는 Bash, 파일 읽기, 파일 편집 도구에 접근할 수 있습니다. [`CLAUDE_CODE_SIMPLE`](/docs/en/env-vars)을 설정합니다. [bare 모드](/docs/en/headless#start-faster-with-bare-mode)를 참조하세요. | `claude --bare -p "query"` |
| `--betas` | API 요청에 포함할 베타 헤더입니다 (API 키 사용자 전용). | `claude --betas interleaved-thinking` |
| `--bg`, `--background` | 세션을 [백그라운드 에이전트](/docs/en/agent-view)로 시작하고 즉시 복귀합니다. 세션 ID 및 관리 명령을 출력합니다. Claude 세션 대신 셸 명령을 백그라운드 작업으로 실행하려면 `--exec`와 결합하고, 특정 하위 에이전트를 실행하려면 `--agent`와 결합하세요. {/* min-version: 2.1.198 */}`-p`/`--print`와 결합할 수 없습니다. [오류 레퍼런스](/docs/en/errors#command-line-errors)를 참조하세요. | `claude --bg "investigate the flaky test"` |
| `--channels` | (리서치 프리뷰) 이 세션에서 Claude가 수신 대기해야 하는 MCP 서버의 [채널](/docs/en/channels) 알림입니다. 공백으로 구분된 `plugin:<name>@<marketplace>` 항목 목록입니다. claude.ai 또는 Console API 키를 통한 Anthropic 인증이 필요합니다. | `claude --channels plugin:my-notifier@my-marketplace` |
| `--chrome` | 웹 자동화 및 테스트를 위해 [Chrome 브라우저 연동](/docs/en/chrome)을 활성화합니다. | `claude --chrome` |
| `--cloud` | 제공된 작업 설명으로 claude.ai에 새 [웹 세션](/docs/en/claude-code-on-the-web)을 생성합니다. | `claude --cloud "Fix the login bug"` |
| `--continue`, `-c` | 현재 디렉토리에서 가장 최근 대화를 로드합니다. `/add-dir`로 이 디렉토리를 추가한 세션도 포함됩니다. | `claude --continue` |
| `--dangerously-load-development-channels` | 로컬 개발을 위해 승인된 허용 목록에 없는 [채널](/docs/en/channels-reference#test-during-the-research-preview)을 활성화합니다. `plugin:<name>@<marketplace>` 및 `server:<name>` 항목을 허용합니다. 확인 프롬프트를 표시합니다. | `claude --dangerously-load-development-channels server:webhook` |
| `--dangerously-skip-permissions` | 권한 프롬프트를 건너끕니다. `--permission-mode bypassPermissions`와 동일합니다. 건너뛰는 항목과 건너뛰지 않는 항목은 [권한 모드](/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode)를 참조하세요. `--bg`로 시작된 세션의 경우 수퍼바이저가 세션을 재시작해도 모드가 [유지됩니다](/docs/en/agent-view#permission-mode-model-and-effort). | `claude --dangerously-skip-permissions` |
| `--debug` | 옵션 카테고리 필터링(예: `"api,hooks"` 또는 `"!statsig,!file"`)과 함께 디버그 모드를 활성화합니다. | `claude --debug "api,mcp"` |
| `--debug-file <path>` | 디버그 로그를 특정 파일 경로에 기록합니다. 암시적으로 디버그 모드를 활성화합니다. `CLAUDE_CODE_DEBUG_LOGS_DIR`보다 우선합니다. | `claude --debug-file /tmp/claude-debug.log` |
| `--disable-slash-commands` | 이 세션의 모든 스킬 및 명령을 비활성화합니다. | `claude --disable-slash-commands` |
| `--disallowedTools`, `--disallowed-tools` | 거부 규칙입니다. 단독 도구 이름은 Claude의 컨텍스트에서 일치하는 도구를 제거합니다 (`"Edit"`는 Edit 제거, `"*"`는 모든 도구 제거, `"mcp__*"`는 모든 MCP 도구 제거). `Bash(rm *)`와 같은 범위 규칙은 도구를 사용 가능한 상태로 두고 일치하는 호출만 거부합니다. [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 지정하는 규칙은 다른 도구가 남아 있는 동안 해당 도구를 제거할 수 없습니다. | `"Bash(git log *)" "Bash(git diff *)" "Edit"` |
| `--effort` | 현재 세션의 [작업 투입 수준(effort level)](/docs/en/model-config#adjust-effort-level)을 설정합니다. 옵션: `low`, `medium`, `high`, `xhigh`, `max` 또는 {/* min-version: 2.1.203 */}`ultracode`. 사용 가능한 수준은 모델에 따라 다릅니다. `ultracode`는 [ultracode](/docs/en/workflows#let-claude-decide-with-ultracode)가 켜진 상태에서 `xhigh` 수준으로 세션을 시작하며 Claude Code v2.1.203 이상이 필요합니다. 이 세션에 대해 [`effortLevel`](/docs/en/settings#available-settings) 설정을 재정의하며 지속되지 않습니다. | `claude --effort high` |
| `--enable-auto-mode` | {/* max-version: 2.1.110 */}v2.1.111에서 제거되었습니다. 자동 모드는 이제 기본적으로 `Shift+Tab` 순환에 포함되어 있습니다. 해당 모드로 시작하려면 `--permission-mode auto`를 사용하세요. | `claude --permission-mode auto` |
| `--exclude-dynamic-system-prompt-sections` | 시스템 프롬프트의 머신별 섹션(작업 디렉토리, 환경 정보, 메모리 경로, git 리포지토리 플래그)을 첫 번째 사용자 메시지로 이동합니다. 동일한 작업을 실행하는 서로 다른 사용자 및 머신 간의 프롬프트 캐시 재사용성을 향상시킵니다. 기본 시스템 프롬프트에서만 적용되며, `--system-prompt` 또는 `--system-prompt-file`이 설정되면 무시됩니다. 스크립트 기반 다중 사용자 워크로드의 경우 `-p`와 함께 사용하세요. | `claude -p --exclude-dynamic-system-prompt-sections "query"` |
| `--exec` | Claude 세션을 시작하는 대신 PTY 기반 백그라운드 작업으로 셸 명령을 실행합니다. 셸에서 시작하려면 `--bg`와 함께 사용하세요. | `claude --bg --exec 'pytest -x'` |
| `--fallback-model` | 주 모델이 과부하 상태이거나 사용할 수 없을 때(예: 더 이상 지원되지 않는 모델) 지정된 모델로의 자동 대체(fallback)를 활성화합니다. 시도할 순서대로 쉼표로 구분된 목록을 받습니다. [대체 모델 체인](/docs/en/model-config#fallback-model-chains)을 참조하세요. 세션 간에 체인을 유지하려면 이 플래그가 재정의하는 [`fallbackModel` 설정](/docs/en/settings#available-settings)을 사용하세요. | `claude --fallback-model sonnet,haiku` |
| `--fork-session` | 대화를 재개할 때 원래 세션 ID를 재사용하는 대신 새 세션 ID를 생성합니다 (`--resume` 또는 `--continue`와 함께 사용). | `claude --resume abc123 --fork-session` |
| `--forward-subagent-text` | {/* min-version: 2.1.211 */}출력 스트림에 [하위 에이전트](/docs/en/sub-agents) 텍스트 및 생각(thinking) 블록을 `parent_tool_use_id`가 설정된 `assistant` 및 `user` 메시지로 내보내 각 하위 에이전트의 트랜스크립트를 재구성할 수 있도록 합니다. 이 플래그가 없으면 Claude Code는 하위 에이전트의 `tool_use` 및 `tool_result` 블록만 내보냅니다. `--print` 및 `--output-format stream-json`이 필요합니다. [`CLAUDE_CODE_FORWARD_SUBAGENT_TEXT`](/docs/en/env-vars) 환경 변수를 통해 동일한 동작을 활성화할 수도 있습니다. Claude Code v2.1.211 이상이 필요합니다. | `claude -p --output-format stream-json --verbose --forward-subagent-text "query"` |
| `--from-pr` | 특정 풀 리퀘스트에 연결된 세션으로 필터링된 세션 선택기를 엽니다. PR 번호, GitHub 또는 GitHub Enterprise PR URL, GitLab 머지 리퀘스트 URL 또는 Bitbucket 풀 리퀘스트 URL을 허용합니다. 세션은 Claude가 풀 리퀘스트를 생성할 때 자동으로 연결됩니다. | `claude --from-pr 123` |
| `--ide` | 유효한 IDE가 정확히 하나만 있는 경우 시작 시 IDE에 자동으로 연결합니다. | `claude --ide` |
| `--init` | 세션 전에 `init` 매처가 포함된 [설정 훅](/docs/en/hooks#setup)을 실행합니다 (출력 모드 전용). | `claude -p --init "query"` |
| `--init-only` | [설정](/docs/en/hooks#setup) 및 `SessionStart` 훅을 실행한 다음 대화를 시작하지 않고 종료합니다. | `claude --init-only` |
| `--include-hook-events` | 출력 스트림에 모든 훅 이벤트의 훅 라이프사이클 이벤트를 포함합니다. `SessionStart` 및 `Setup` 훅 이벤트는 항상 포함되며 이 플래그가 필요하지 않습니다. `--output-format stream-json`이 필요합니다. | `claude -p --output-format stream-json --verbose --include-hook-events "query"` |
| `--include-partial-messages` | 출력에 부분 스트리밍 이벤트를 포함합니다. `--print` 및 `--output-format stream-json`이 필요합니다. | `claude -p --output-format stream-json --verbose --include-partial-messages "query"` |
| `--input-format` | 출력 모드의 입력 형식을 지정합니다 (옵션: `text`, `stream-json`). | `claude -p --output-format json --input-format stream-json` |
| `--json-schema` | 에이전트가 워크플로우를 완료한 후 JSON Schema와 일치하는 검증된 JSON 출력을 가져옵니다 (출력 모드 전용). [구조화된 출력](/docs/en/agent-sdk/structured-outputs)을 참조하세요. {/* min-version: 2.1.205 */}Claude Code는 유효하지 않은 스키마에서 오류와 함께 종료되며 클라이언트 측 검증 없이 `format` 키워드를 어노테이션으로 허용합니다. v2.1.205 이전에는 유효하지 않은 스키마가 오류 없이 비구조화된 출력을 생성했으며 `format`을 사용하는 스키마는 유효하지 않은 것으로 취급되었습니다. | `claude -p --json-schema '{"type":"object","properties":{...}}' "query"` |
| `--maintenance` | 세션 전에 `maintenance` 매처가 포함된 [설정 훅](/docs/en/hooks#setup)을 실행합니다 (출력 모드 전용). | `claude -p --maintenance "query"` |
| `--max-budget-usd` | 중지하기 전 API 호출에 지출할 최대 달러 금액입니다 (출력 모드 전용). | `claude -p --max-budget-usd 5.00 "query"` |
| `--max-turns` | 에이전트 턴 수를 제한합니다 (출력 모드 전용). 한도에 도달하면 오류와 함께 종료합니다. 기본 한도는 없습니다. {/* min-version: 2.1.205 */}`--input-format stream-json`을 사용할 때 Claude가 작업하는 동안 전송된 메시지는 대기열에 유지되며, 한도가 현재 턴을 끝낼 때 자체 한도를 가지고 자체 턴으로 실행됩니다. v2.1.205 이전에는 Claude Code가 해당 메시지를 버렸습니다. | `claude -p --max-turns 3 "query"` |
| `--mcp-config` | JSON 파일 또는 문자열(공백으로 구분)에서 MCP 서버를 로드합니다. | `claude --mcp-config ./mcp.json` |
| `--model` | 최신 모델의 별칭(`sonnet`, `opus`, `haiku`, `fable`) 또는 모델의 전체 이름으로 현재 세션의 모델을 설정합니다. [`model`](/docs/en/settings#available-settings) 설정 및 [`ANTHROPIC_MODEL`](/docs/en/model-config#environment-variables)을 재정의합니다. | `claude --model claude-sonnet-5` |
| `--name`, `-n` | `/resume` 및 터미널 제목에 표시되는 세션의 표시 이름을 설정합니다. `claude --resume <name>`으로 이름 지정된 세션을 재재개할 수 있습니다. <br /><br />[`/rename`](/docs/en/commands)은 세션 중간에 이름을 변경하고 프롬프트 바에도 이름을 표시합니다. | `claude -n "my-feature-work"` |
| `--no-chrome` | 이 세션의 [Chrome 브라우저 연동](/docs/en/chrome)을 비활성화합니다. | `claude --no-chrome` |
| `--no-session-persistence` | 세션이 디스크에 저장되지 않고 재개할 수 없도록 세션 지속성을 비활성화합니다. 출력 모드 전용입니다. [`CLAUDE_CODE_SKIP_PROMPT_HISTORY`](/docs/en/env-vars) 환경 변수도 모든 모드에서 동일하게 작동합니다. | `claude -p --no-session-persistence "query"` |
| `--output-format` | 출력 모드의 출력 형식을 지정합니다 (옵션: `text`, `json`, `stream-json`). | `claude -p "query" --output-format json` |
| `--permission-mode` | 지정된 [권한 모드](/docs/en/permission-modes)로 시작합니다. `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions` 또는 {/* min-version: 2.1.200 */}`default`의 별칭인 `manual`을 허용합니다. `manual` 별칭은 UI에서 Manual로 라벨이 지정된 모드를 선택하며 Claude Code v2.1.200 이상이 필요합니다. `claude --help`는 `default` 대신 나열하며 두 값 모두 동작합니다. 설정 파일의 `defaultMode`를 재정의합니다. | `claude --permission-mode plan` |
| `--permission-prompt-tool` | 비대화형 모드에서 권한 프롬프트를 처리할 MCP 도구를 지정합니다. {/* min-version: 2.1.206 */}Claude Code는 첫 번째 턴을 실행하기 전에 해당 도구의 MCP 서버가 연결될 때까지 최대 [`MCP_TIMEOUT`](/docs/en/env-vars) 시작 타임아웃인 30초 동안 대기합니다. v2.1.206 이전에는 시작이 느린 서버로 인해 실행 시 [MCP 도구를 찾을 수 없다는 오류와 함께 종료](/docs/en/errors#mcp-permission-prompt-tool-not-found)할 수 있었습니다. <br /><br />{/* min-version: 2.1.199 */}프롬프트 도구는 [사용자 상호작용이 필요한](/docs/en/mcp#require-approval-for-a-specific-tool) 것으로 표시된 MCP 도구를 승인할 수 없습니다. Claude Code는 이에 대한 `allow` 결과를 거부(deny)로 변환합니다. 이 제한에는 Claude Code v2.1.199 이상이 필요합니다. | `claude -p --permission-prompt-tool mcp_auth_tool "query"` |
| `--plugin-dir` | 이 세션에 대해서만 디렉토리 또는 `.zip` 아카이브에서 플러그인을 로드합니다. 각 플러그인에 대해 플래그를 하나씩 지정합니다. 여러 플러그인의 경우 플래그를 반복합니다: `--plugin-dir A --plugin-dir B.zip` | `claude --plugin-dir ./my-plugin` |
| `--plugin-url` | 이 세션에 대해서만 URL에서 플러그인 `.zip` 아카이브를 가져옵니다. 여러 플러그인에 대해 플래그를 반복하거나 단일 따옴표 값 내에 공백으로 구분된 URL을 전달하세요. | `claude --plugin-url https://example.com/plugin.zip` |
| `--print`, `-p` | 대화형 모드 없이 응답을 출력합니다 (프로그래밍 방식의 사용법은 [Agent SDK 문서](/docs/en/agent-sdk/overview) 참조). | `claude -p "query"` |
| `--prompt-suggestions` | 다음 사용자 프롬프트 예상이 포함된 `prompt_suggestion` 메시지를 각 턴 이후 내보냅니다. `--print`, `--output-format stream-json` 및 `--verbose`가 필요합니다. [프롬프트 제안](/docs/en/interactive-mode#prompt-suggestions)을 참조하세요. | `claude -p --prompt-suggestions --output-format stream-json --verbose "query"` |
| `--remote` | `--cloud`에 대해 사용 중지 예정(deprecated)된 별칭입니다. | `claude --remote "Fix the login bug"` |
| `--remote-control`, `--rc` | claude.ai 또는 Claude 앱에서도 제어할 수 있도록 [Remote Control](/docs/en/remote-control#start-a-remote-control-session)을 활성화한 상태로 대화형 세션을 시작합니다. 옵션으로 세션 이름을 전달할 수 있습니다. | `claude --remote-control "My Project"` |
| `--remote-control-session-name-prefix <prefix>` | 명시적 이름이 설정되지 않았을 때 자동 생성되는 [Remote Control](/docs/en/remote-control) 세션 이름의 접두사입니다. 기본값은 시스템의 호스트 이름이며 `myhost-graceful-unicorn`과 같은 이름을 생성합니다. 동일한 효과를 얻으려면 `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX`를 설정하세요. | `claude remote-control --remote-control-session-name-prefix dev-box` |
| `--replay-user-messages` | 승인을 위해 stdin의 사용자 메시지를 stdout으로 다시 내보냅니다. `--input-format stream-json` 및 `--output-format stream-json`이 필요합니다. | `claude -p --input-format stream-json --output-format stream-json --verbose --replay-user-messages` |
| `--resume`, `-r` | ID 또는 이름으로 특정 세션을 재개하거나 세션을 선택할 수 있는 대화형 선택기를 엽니다. 선택기 및 이름 검색에는 `/add-dir`로 이 디렉토리를 추가한 세션이 포함되며, 세션 ID를 전달하면 현재 프로젝트 디렉토리 및 작업 트리만 검색합니다. v2.1.144부터 [백그라운드 세션](/docs/en/agent-view)이 `bg`로 표시되어 선택기에 나타납니다. | `claude --resume auth-refactor` |
| `--safe-mode` | {/* min-version: 2.1.169 */}손상된 구성을 문제 해결하기 위해 모든 커스터마이징이 비활성화된 상태로 시작합니다: CLAUDE.md, 스킬, 플러그인, 훅, MCP 서버, 커스텀 명령 및 에이전트, 출력 스타일, 워크플로우, 커스텀 테마, 커스텀 키 바인딩, 상태 표시줄 및 파일 제안 명령, LSP 서버 및 자동 메모리가 로드되지 않습니다. 인증, 모델 선택, 내장 도구 및 권한은 정상 작동하며 이는 [`--bare`](/docs/en/headless#start-faster-with-bare-mode)와 다릅니다. 정책으로 구성된 훅, 상태 표시줄 및 파일 제안 명령을 포함하여 관리형 설정 정책은 여전히 적용되지만, 관리형 플러그인, 관리형 스킬, 관리형 CLAUDE.md 및 정책 구성 MCP 서버는 적용되지 않습니다. 커스터마이징이 [Fable 5의 자동 대체](/docs/en/model-config#automatic-model-fallback)를 트리거하는 요소인지 확인하는 데 유용합니다. [`CLAUDE_CODE_SAFE_MODE`](/docs/en/env-vars)를 설정합니다. | `claude --safe-mode` |
| `--session-id` | 대화에 특정 세션 ID를 사용합니다 (유효한 UUID여야 함). | `claude --session-id "550e8400-e29b-41d4-a716-446655440000"` |
| `--setting-sources` | 로드할 설정 소스의 쉼표로 구분된 목록입니다 (`user`, `project`, `local`). | `claude --setting-sources user,project` |
| `--settings` | 설정 JSON 파일 경로 또는 인라인 JSON 문자열입니다. 여기에 설정한 값은 이 세션 동안 `settings.json` 파일의 동일한 키를 재정의합니다. 생략한 키는 파일 기반 값을 유지합니다. 파일은 2 MiB 이하의 일반 파일이어야 합니다. [설정 우선순위](/docs/en/settings#settings-precedence)를 참조하세요. | `claude --settings ./settings.json` |
| `--strict-mcp-config` | 다른 모든 MCP 구성을 무시하고 `--mcp-config`의 MCP 서버만 사용합니다. | `claude --strict-mcp-config --mcp-config ./mcp.json` |
| `--system-prompt` | 전체 시스템 프롬프트를 커스텀 텍스트로 대체합니다. | `claude --system-prompt "You are a Python expert"` |
| `--system-prompt-file` | 파일에서 시스템 프롬프트를 로드하여 기본 프롬프트를 대체합니다. | `claude --system-prompt-file ./custom-prompt.txt` |
| `--teleport` | 로컬 터미널에서 [웹 세션](/docs/en/claude-code-on-the-web)을 재개합니다. | `claude --teleport` |
| `--teammate-mode` | [에이전트 팀](/docs/en/agent-teams) 팀원의 표시 방식을 설정합니다: `in-process` (기본값), `auto`, `tmux` 또는 {/* min-version: 2.1.186 */}`iterm2` (v2.1.186에서 추가됨). 기본값은 v2.1.179에서 `auto`로부터 변경되었습니다. 이 세션에 대해 [`teammateMode`](/docs/en/settings#available-settings) 설정을 재정의합니다. [표시 모드 선택](/docs/en/agent-teams#choose-a-display-mode)을 참조하세요. | `claude --teammate-mode auto` |
| `--tmux` | 작업 트리에 대한 tmux 세션을 생성합니다. `--worktree`가 필요합니다. 사용 가능한 경우 iTerm2 네이티브 창을 사용하며, 기존 tmux의 경우 `--tmux=classic`을 전달하세요. | `claude -w feature-auth --tmux` |
| `--tools` | Claude가 사용할 수 있는 내장 도구를 제한합니다. 모두 비활성화하려면 `""`, 모두 사용하려면 `"default"`, 또는 `"Bash,Edit,Read"`와 같이 도구 이름을 지정합니다. 이 플래그는 MCP 도구에는 영향을 주지 않으며, 해당 도구도 거부하려면 `--disallowedTools "mcp__*"`를 사용하거나 `--mcp-config` 없이 `--strict-mcp-config`를 전달하여 MCP 서버가 로드되지 않도록 하세요. [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 생략한 목록은 해당 도구를 제거하지 않으며, `""`는 남아 있는 MCP 도구가 없을 때만 제거합니다. | `claude --tools "Bash,Edit,Read"` |
| `--verbose` | 상세 로깅을 활성화하고 턴별 전체 출력을 표시합니다. 이 세션에 대해 [`viewMode`](/docs/en/settings#available-settings) 설정을 재정의합니다. | `claude --verbose` |
| `--version`, `-v` | 버전 번호를 출력합니다. | `claude -v` |
| `--worktree`, `-w` | `<repo>/.claude/worktrees/<name>` 위치의 격리된 [git worktree](/docs/en/worktrees)에서 Claude를 시작합니다. 이름이 지정되지 않으면 자동으로 생성됩니다. `#<number>` 또는 GitHub 풀 리퀘스트 URL을 전달하여 `origin`에서 해당 PR을 가져와 작업 트리를 분기합니다. | `claude -w feature-auth` |

### 시스템 프롬프트 플래그

Claude Code는 시스템 프롬프트를 커스터마이징할 수 있는 4개의 플래그를 제공합니다. 4개 모두 대화형 및 비대화형 모드에서 작동합니다.

| 플래그 | 동작 | 예시 |
| :--- | :--- | :--- |
| `--system-prompt` | 전체 기본 프롬프트를 대체 | `claude --system-prompt "You are a Python expert"` |
| `--system-prompt-file` | 파일 내용으로 대체 | `claude --system-prompt-file ./prompts/review.txt` |
| `--append-system-prompt` | 기본 프롬프트 끝에 추가 | `claude --append-system-prompt "Always use TypeScript"` |
| `--append-system-prompt-file` | 파일 내용을 기본 프롬프트 끝에 추가 | `claude --append-system-prompt-file ./style-rules.txt` |

`--system-prompt` 및 `--system-prompt-file`은 상호 배타적입니다. 추가 플래그는 두 대체 플래그 중 하나와 결합할 수 있습니다.

Claude Code의 기본 정체성이 작업에 여전히 부합하는지 여부에 따라 선택하세요. Claude가 추가 규칙(호출당 지침, 출력 형식 지정 또는 `-p` 스크립트용 도메인 컨텍스트)을 따르는 코딩 어시스턴트로 남아 있어야 할 때는 추가 플래그를 사용하세요. 추가(append) 방식은 기본 도구 지침, 보안 지침 및 코딩 규칙을 보존하므로 다른 부분만 제공하면 됩니다. 파이프라인에서 사람이 지켜보지 않는 비코딩 에이전트와 같이 표면, 정체성 또는 권한 모델이 Claude Code와 다를 때는 대체 플래그를 사용하세요. 대체(replace) 방식은 도구 지침 및 보안 지침을 포함한 모든 기본 프롬프트를 삭제하므로 작업에 필요한 모든 것에 대한 책임을 집니다.

이러한 플래그는 현재 호출에만 적용됩니다. 프로젝트 전체에서 전환하고 공유할 수 있는 지속적인 페르소나의 경우 [출력 스타일](/docs/en/output-styles)을 사용하세요. Claude가 항상 따라야 하는 프로젝트 규칙의 경우 [CLAUDE.md](/docs/en/memory)를 사용하세요. [시스템 프롬프트에 대한 Agent SDK 가이드](/docs/en/agent-sdk/modifying-system-prompts#decide-on-a-starting-point)에서 동일한 결정을 더 깊이 있게 다룹니다.

## 참고 항목

* [Chrome 확장 프로그램](/docs/en/chrome) - 브라우저 자동화 및 웹 테스트
* [대화형 모드](/docs/en/interactive-mode) - 단축키, 입력 모드 및 대화형 기능
* [빠른 시작 가이드](/docs/en/quickstart) - Claude Code 시작하기
* [공통 워크플로우](/docs/en/common-workflows) - 고급 워크플로우 및 패턴
* [설정](/docs/en/settings) - 구성 옵션
* [Agent SDK 문서](/docs/en/agent-sdk/overview) - 프로그래밍 방식 사용법 및 연동
