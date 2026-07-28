> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 도구 참조 문서 (Tools reference)

> 권한 요구 사항 및 도구별 동작을 포함하여 Claude Code가 사용할 수 있는 도구에 대한 포괄적인 참조 문서입니다.

Claude Code는 코드베이스를 이해하고 수정하는 데 도움이 되는 일련의 내장 도구(built-in tools)에 접근할 수 있습니다. 도구 이름은 [권한 규칙](/docs/en/permissions#tool-specific-permission-rules), [서브에이전트 도구 목록](/docs/en/sub-agents), 및 [훅 매처](/docs/en/hooks)에서 사용하는 정확한 문자열입니다.

Claude가 사용할 수 있는 도구와 처음에 승인을 요청하는 시점을 제어하려면 설정의 [권한 규칙](/docs/en/permissions#tool-specific-permission-rules), [훅](/docs/en/hooks), 또는 [서브에이전트의 도구 목록](/docs/en/sub-agents#supported-frontmatter-fields)을 구성하세요. 도구 이름을 수용하는 각 위치는 [권한 규칙 및 훅으로 도구 구성하기](#configure-tools-with-permission-rules-and-hooks)를 참조하세요.

커스텀 도구를 추가하려면 [MCP 서버](/docs/en/mcp)를 연결하세요. 재사용 가능한 프롬프트 기반 워크플로로 Claude를 확장하려면 [skill](/docs/en/skills)을 작성하세요. 이는 새 도구 항목을 추가하는 대신 기존 `Skill` 도구를 통해 실행됩니다.

<Info>
  `권한 필요` 열은 작업 디렉터리 내부의 경로에 대해 기본 권한 모드에서 도구가 프롬프트를 표시하는지 여부를 보여줍니다. `Read`, `Grep`, `Glob`을 포함하여 '아니오'로 표시된 파일 접근 도구는 [작업 디렉터리 및 추가 디렉터리](/docs/en/permissions#working-directories) 외부의 경로에 대해서는 여전히 프롬프트를 표시합니다. `Bash`는 '예'로 표시되어 있지만 프롬프트 없이 내장된 [읽기 전용 명령](/docs/en/permissions#read-only-commands) 세트를 실행합니다.
</Info>

| 도구 | 설명 | 권한 필요 |
| :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------ |
| `Agent` | 작업을 처리하기 위해 자체 컨텍스트 창이 있는 [서브에이전트](/docs/en/sub-agents)를 스폰합니다. [Agent tool behavior](#agent-tool-behavior) 참조 | 아니오 |
| `Artifact` | HTML 또는 Markdown 파일을 [아티팩트](/docs/en/artifacts)(claude.ai의 비공개 대화형 페이지)로 게시합니다. 공개 링크로 공유하거나, 소유자가 [공개 공유를 활성화](/docs/en/artifacts#control-public-sharing)해야 하는 Team 및 Enterprise 플랜의 조직 내부에서 공유할 수 있습니다. {/* plan-availability: feature=artifacts plans=pro,max,team,enterprise providers=anthropic */} Pro, Max, Team 또는 Enterprise 플랜 및 `/login` 인증이 필요합니다. [Availability](/docs/en/artifacts#availability) 참조 | 예 |
| `AskUserQuestion` | 요구 사항을 수집하거나 모호함을 명확히 하기 위해 객관식 질문을 합니다. 질문은 기본적으로 답변할 때까지 열려 있습니다. [AskUserQuestion tool behavior](#askuserquestion-tool-behavior) 참조 | 아니오 |
| `Bash` | 환경에서 쉘 명령을 실행합니다. [Bash tool behavior](#bash-tool-behavior) 참조 | 예 |
| `CronCreate` | 현재 세션 내에서 반복 또는 1회성 프롬프트를 예약합니다. 작업은 세션 범위로 제한되며 만료되지 않은 경우 `--resume` 또는 `--continue` 시 복원됩니다. [scheduled tasks](/docs/en/scheduled-tasks) 참조 | 아니오 |
| `CronDelete` | ID로 예약된 작업을 취소합니다 | 아니오 |
| `CronList` | 세션의 모든 예약된 작업을 나열합니다 | 아니오 |
| `Edit` | 특정 파일을 대상으로 수정합니다. [Edit tool behavior](#edit-tool-behavior) 참조 | 예 |
| `EndConversation` | 지속적인 악성 입력이 발생하는 드문 경우나 사용자가 도구 시연을 요청할 때 세션을 종료합니다. {/* min-version: 2.1.213 */} Claude Code v2.1.213 이상 필요. [EndConversation tool behavior](#endconversation-tool-behavior) 참조 | 아니오 |
| `EnterPlanMode` | 코딩 전 접근 방식을 설계하기 위해 plan 모드로 전환합니다 | 아니오 |
| `EnterWorktree` | 격리된 [git 워크트리](/docs/en/worktrees)를 생성하고 해당 워크트리로 전환합니다. 새 워크트리를 생성하는 대신 기존 워크트리로 전환하려면 `path`를 전달하세요. {/* min-version: 2.1.203 */} 처음 진입 시 대상은 현재 리포지토리의 워크트리이거나 다중 리포지토리 작업 공간의 경우 내부에 중첩된 리포지토리의 워크트리일 수 있습니다. v2.1.203 이전에는 중첩된 리포지토리의 워크트리가 거부되었습니다. {/* min-version: 2.1.206 */} `.claude/worktrees/` 외부의 `path`는 세션의 작업 디렉터리와 쓰기 접근 권한을 해당 위치로 이동시키므로 진입 전 승인을 요청합니다. 새 워크트리 생성 및 `.claude/worktrees/` 아래의 경로는 프롬프트를 표시하지 않습니다. v2.1.206 이전에는 Claude가 프롬프트 없이 `.claude/worktrees/` 외부 경로에 진입했습니다. 워크트리 세션 내부에서 또는 [`isolation: worktree`](/docs/en/sub-agents#supported-frontmatter-fields)와 같이 고정된 작업 디렉터리가 있는 서브에이전트에서는 `path` 형식만 사용할 수 있으며 대상은 세션 리포지토리의 `.claude/worktrees/` 아래여야 합니다 | 예 |
| `ExitPlanMode` | 승인을 위해 플랜을 제시하고 plan 모드를 종료합니다 | 예 |
| `ExitWorktree` | 워크트리 세션을 종료하고 원래 디렉터리로 돌아갑니다. [`isolation: worktree`](/docs/en/sub-agents#supported-frontmatter-fields)와 같이 이미 자체 작업 디렉터리에서 실행 중인 서브에이전트에는 사용할 수 없습니다 | 아니오 |
| `Glob` | 패턴 일치를 기반으로 파일을 찾습니다. [Glob tool behavior](#glob-tool-behavior) 참조 | 아니오 |
| `Grep` | 파일 내용에서 패턴을 검색합니다. [Grep tool behavior](#grep-tool-behavior) 참조 | 아니오 |
| `ListMcpResourcesTool` | 연결된 [MCP 서버](/docs/en/mcp)가 노출하는 리소스를 나열합니다 | 아니오 |
| `LSP` | 언어 서버를 통한 코드 인텔리전스: 정의로 이동, 참조 찾기, 타입 오류 및 경고 보고. [LSP tool behavior](#lsp-tool-behavior) 참조 | 아니오 |
| `Monitor` | 백그라운드에서 명령을 실행하고 각 출력 라인을 Claude에게 다시 제공하여 대화 중간에 로그 항목, 파일 변경 사항 또는 폴링된 상태에 반응할 수 있게 합니다. WebSocket을 열고 들어오는 각 메시지를 이벤트로 취급할 수도 있습니다. [Monitor tool](#monitor-tool) 참조 | 예 |
| `NotebookEdit` | Jupyter 노트북 셀을 수정합니다. [NotebookEdit tool behavior](#notebookedit-tool-behavior) 참조 | 예 |
| `PowerShell` | 네이티브로 PowerShell 명령을 실행합니다. 가용성은 [PowerShell tool](#powershell-tool) 참조 | 예 |
| `PushNotification` | 이동 시 장시간 실행되는 작업이나 [예약된 작업](/docs/en/scheduled-tasks)이 사용자에게 도달할 수 있도록 데스크톱 알림을 보내고 [Remote Control](/docs/en/remote-control)이 연결되어 있을 때 휴대폰 푸시를 보냅니다. {/* plan-availability: feature=push-notifications providers=anthropic */} 푸시 전송은 Anthropic 호스팅 인프라를 통해 실행되며 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry에서는 접근할 수 없습니다 | 아니오 |
| `Read` | 파일 내용을 읽습니다. [Read tool behavior](#read-tool-behavior) 참조 | 아니오 |
| `ReadMcpResourceTool` | URI로 특정 MCP 리소스를 읽습니다 | 아니오 |
| `RemoteTrigger` | claude.ai에서 [Routines](/docs/en/routines)를 생성, 업데이트, 실행 및 나열합니다. `/schedule` 명령의 바탕이 됩니다. {/* plan-availability: feature=routines plans=pro,max,team,enterprise providers=anthropic */} 루틴은 claude.ai에 존재하며 Pro, Max, Team 또는 Enterprise 플랜이 필요하므로 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry에서는 이 도구에 접근할 수 없습니다 | 아니오 |
| `ReportFindings` | Claude Code가 코드 리뷰 결과를 텍스트로 출력하는 대신 렌더링할 수 있도록 파일, 요약 및 실패 시나리오가 포함된 구조화된 목록으로 보고합니다. 활성 코드 리뷰 지침이 지시할 때 Claude가 이를 호출합니다. {/* min-version: 2.1.196 */} Claude Code v2.1.196 이상 필요. {/* min-version: 2.1.199 */} v2.1.199부터 발견 사항은 렌더링된 목록의 파일 위치 옆에 표시되는 `correctness` 또는 `test-coverage`와 같은 선택적 `category` 슬러그를 가질 수도 있습니다 | 아니오 |
| `ScheduleWakeup` | [자율 주행 `/loop`](/docs/en/scheduled-tasks#let-claude-choose-the-interval)의 다음 반복을 재예약합니다. Claude는 다음 반복이 1분에서 1시간 사이 중 언제 실행될지 선택하기 위해 각 반복의 끝에 이를 호출하며, 사용자가 직접 호출하지 않습니다. 대신 루프를 종료하기 위해 Claude는 `stop: true`로 호출하여 보류 중인 대기를 취소합니다. {/* min-version: 2.1.202 */} `stop` 필드는 Claude Code v2.1.202 이상이 필요합니다. 보류 중인 대기는 [Stop hook input](/docs/en/hooks#stop-input)의 `session_crons`에 표시됩니다. {/* plan-availability: feature=loop-dynamic providers=anthropic */} Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform 또는 Microsoft Foundry에서는 사용할 수 없으며, 이러한 플랫폼에서는 간격이 없는 `/loop` 프롬프트가 대신 고정 일정으로 실행됩니다 | 아니오 |
| `SendMessage` | [에이전트 팀](/docs/en/agent-teams) 팀원에게 메시지를 보내거나 에이전트 ID 또는 이름으로 [서브에이전트를 재개](/docs/en/sub-agents#resume-subagents)합니다. 완료된 서브에이전트는 백그라운드에서 자동 재개되며, `/tasks`에서 중지한 서브에이전트는 재개되지 않고 호출이 거부를 반환합니다. 구조화된 팀 프로토콜 메시지는 에이전트 팀이 필요합니다. 수신자는 다른 에이전트의 메시지를 사용자의 동의나 승인으로 취급하지 않습니다. {/* min-version: 2.1.198 */} v2.1.198부터 서브에이전트는 자신을 실행한 에이전트의 메시지를 동료 요청이 아닌 일반 작업 지시로 취급합니다. {/* min-version: 2.1.199 */} v2.1.199부터 대화 앞부분에서 도달했던 것과 다른 에이전트로 해석되는 이름에 대한 전송은 전달되지 않고 거부됩니다. [Resume subagents](/docs/en/sub-agents#resume-subagents) 참조 | 아니오 |
| `SendUserFile` | 메시지 트랜스크립트에 언급되는 것 외에도 생성된 보고서, 다이어그램, 스크린샷 또는 빌드된 아티팩트가 사용자의 기기에 도달하도록 선택적 캡션과 함께 세션에서 사용자에게 파일을 보냅니다. {/* min-version: 2.1.196 */} v2.1.196부터 선택적 `display` 입력이 프레젠테이션을 제어합니다. `render`는 클라이언트에서 파일을 인라인으로 열고, `attach`는 다운로드 카드만 보여주며, 설정되지 않은 경우 클라이언트가 파일 유형에 따라 결정합니다. [Remote Control](/docs/en/remote-control) 클라이언트가 연결되어 있거나 세션이 [Claude Code on the web](/docs/en/claude-code-on-the-web)과 같은 관리형 클라우드 환경에서 실행될 때 사용할 수 있습니다. 전송은 Anthropic 호스팅 인프라를 통해 이루어지므로 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry에서는 이 도구를 사용할 수 없습니다 | 아니오 |
| `ShareOnboardingGuide` | {/* plan-availability: feature=onboarding-guide-share plans=pro,max,team,enterprise providers=anthropic */} `ONBOARDING.md`를 업로드하고 팀원이 Claude Code에서 열 수 있는 공유 링크를 반환합니다. 가이드가 작성된 후 `/team-onboarding`에서 호출됩니다. Pro, Max, Team 및 Enterprise 플랜의 claude.ai 구독자가 사용할 수 있습니다 | 예 |
| `Skill` | 메인 대화 내에서 [skill](/docs/en/skills#control-who-invokes-a-skill)을 실행합니다 | 예 |
| `TaskCreate` | 작업 목록에 새 작업을 생성합니다 | 아니오 |
| `TaskGet` | 특정 작업에 대한 전체 세부 정보를 가져옵니다 | 아니오 |
| `TaskList` | 현재 상태와 함께 모든 작업을 나열합니다 | 아니오 |
| `TaskOutput` | 백그라운드 작업에서 출력을 가져옵니다. 작업의 출력 파일 경로에 대한 `Read`를 사용하도록 권장되어 단종(deprecated)되었습니다. {/* min-version: 2.1.203 */} ID와 일치하는 작업이 없으면 오류에 ID 및 설명별로 실행 중인 백그라운드 에이전트가 나열됩니다. v2.1.203 이전에는 오류에 누락된 ID만 명시되었습니다 | 아니오 |
| `TaskStop` | ID로 실행 중인 백그라운드 작업을 중지합니다. {/* min-version: 2.1.198 */} 에이전트 ID 또는 이름으로 [에이전트 팀 팀원](/docs/en/agent-teams) 또는 명명된 백그라운드 에이전트도 허용합니다. v2.1.198 이전에는 백그라운드 작업 ID만 허용했습니다. {/* min-version: 2.1.203 */} ID와 일치하는 작업이 없으면 오류에 다른 에이전트가 스폰한 에이전트를 포함하여 ID 및 설명별로 실행 중인 백그라운드 에이전트가 나열됩니다. v2.1.203 이전에는 오류에 실행 중인 팀원과 명명된 에이전트가 나열되었지만 다른 에이전트가 스폰한 백그라운드 에이전트는 나열되지 않아 메인 대화에서 이를 식별하거나 중지할 수 없었습니다 | 아니오 |
| `TaskUpdate` | 작업 상태, 종속성, 세부 정보를 업데이트하거나 작업을 삭제합니다 | 아니오 |
| `TodoWrite` | {/* min-version: 2.1.142 */} 세션 작업 체크리스트를 관리합니다. `TaskCreate`, `TaskGet`, `TaskList`, `TaskUpdate`를 권장함에 따라 v2.1.142부터 기본적으로 비활성화되었습니다. 다시 활성화하려면 `CLAUDE_CODE_ENABLE_TASKS=0`을 설정하세요 | 아니오 |
| `ToolSearch` | [tool search](/docs/en/mcp#scale-with-mcp-tool-search)가 활성화되어 있을 때 지연된 도구를 검색하고 로드합니다 | 아니오 |
| `WaitForMcpServers` | 세션을 재시작하지 않고도 해당 도구를 사용할 수 있도록 백그라운드에서 연결 중인 하나 이상의 [MCP 서버](/docs/en/mcp)를 기다립니다. 필요한 서버가 아직 연결되지 않았을 때 Claude가 이를 호출합니다. `ToolSearch`가 활성화되어 있을 때 대기를 처리하므로 [tool search](/docs/en/mcp#scale-with-mcp-tool-search)가 비활성화되어 있을 때만 표시됩니다 | 아니오 |
| `WebFetch` | 지정된 URL에서 콘텐츠를 가져옵니다. [WebFetch tool behavior](#webfetch-tool-behavior) 참조 | 예 |
| `WebSearch` | 웹 검색을 수행합니다. [WebSearch tool behavior](#websearch-tool-behavior) 참조 | 예 |
| `Workflow` | 백그라운드에서 많은 서브에이전트를 오케스트레이션하고 하나의 통합된 결과를 반환하는 스크립트인 [동적 워크플로](/docs/en/workflows)를 실행합니다 | 예 |
| `Write` | 파일을 생성하거나 덮어씁니다. [Write tool behavior](#write-tool-behavior) 참조 | 예 |

## 권한 규칙 및 훅으로 도구 구성하기

대부분의 경우 Claude가 이러한 도구를 사용할 시점을 결정하므로 Claude와 상호작용할 때 직접 이름을 지정할 필요가 없습니다. 권한 및 기타 구성을 정의할 때 도구 이름을 직접 참조합니다:

* 설정의 [`permissions.allow` 및 `permissions.deny`](/docs/en/settings#available-settings) 및 `/permissions` 인터페이스에서
* `--allowedTools` 및 `--disallowedTools` [CLI 플래그](/docs/en/cli-reference)에서
* Agent SDK의 [`allowedTools` 및 `disallowedTools`](/docs/en/agent-sdk/permissions#allow-and-deny-rules) 옵션에서
* [서브에이전트의 `tools` 또는 `disallowedTools`](/docs/en/sub-agents#supported-frontmatter-fields) 프론트매터에서
* [skill의 `allowed-tools`](/docs/en/skills#frontmatter-reference) 프론트매터에서
* 훅의 [`if` 조건](/docs/en/hooks-guide#filter-by-tool-name-and-arguments-with-the-if-field)에서

이들 모두 동일한 규칙 형식인 `ToolName(specifier)`를 받아들입니다. 지시자(specifier)는 도구에 따라 달라지며 여러 도구가 형식을 공유합니다:

| 규칙 형식 | 적용 대상 | 세부 정보 |
| :----------------------------- | :------------------------ | :--------------------------------------------------------------- |
| `Bash(npm run *)` | Bash, Monitor | [명령 패턴 일치](/docs/en/permissions#bash) |
| `PowerShell(Get-ChildItem *)` | PowerShell | [명령 패턴 일치](/docs/en/permissions#powershell) |
| `Read(~/secrets/**)` | Read, Grep, Glob, LSP | [경로 패턴 일치](/docs/en/permissions#read-and-edit) |
| `Edit(/src/**)` | Edit, Write, NotebookEdit | [경로 패턴 일치](/docs/en/permissions#read-and-edit) |
| `Skill(deploy *)` | Skill | [Skill 이름 일치](/docs/en/skills#restrict-claude’s-skill-access) |
| `Agent(Explore)` | Agent | [서브에이전트 유형 일치](/docs/en/permissions#agent-subagents) |
| `WebFetch(domain:example.com)` | WebFetch | [도메인 일치](/docs/en/permissions#webfetch) |
| `WebSearch` | WebSearch | 지시자 없음. 도구 전체를 허용하거나 거부 |

`ExitPlanMode` 또는 `ShareOnboardingGuide`와 같이 여기에 나열되지 않은 도구는 지시자 없이 순수 도구 이름만 받습니다.

`Edit(...)` 허용 규칙은 동일한 경로에 대한 읽기 접근 권한도 부여하므로 일치하는 `Read(...)` 규칙이 필요하지 않습니다. {/* min-version: 2.1.208 */} `Read(...)` 거부 규칙은 편집 결과를 다시 읽어야 하므로 거기에 새 파일을 생성하는 것을 포함하여 동일한 경로에서 Edit 도구도 차단합니다. 편집 시 `Read` 거부 검사는 Claude Code v2.1.208 이상이 필요합니다.

훅 `matcher` 필드는 괄호 형태의 규칙 형식이 아닌 순수 도구 이름을 사용합니다. 일치 규칙은 [matcher patterns](/docs/en/hooks#matcher-patterns)를 참조하세요. 각 도구가 훅의 `tool_input`으로 전달하는 필드 이름은 [PreToolUse input reference](/docs/en/hooks#pretooluse-input)를 참조하세요.

## Agent 도구 동작

Agent 도구는 별도의 컨텍스트 창에서 서브에이전트를 스폰합니다. 서브에이전트는 자율적으로 작업을 진행한 다음 상위 대화에 단일 텍스트 결과를 반환합니다. 상위 대화에는 서브에이전트의 중간 도구 호출이나 출력이 표시되지 않고 최종 결과만 표시됩니다.

서브에이전트가 실행되는 턴 수를 제한하려면 [서브에이전트 정의](/docs/en/sub-agents#supported-frontmatter-fields)에 `maxTurns`를 설정하세요.

동일한 Agent 도구는 포크 모드가 활성화되어 있을 때 [포크된 서브에이전트](/docs/en/sub-agents#fork-the-current-conversation)도 시작합니다. 포크는 새로 시작하는 대신 전체 상위 대화를 상속받고, 항상 백그라운드에서 실행되며, 터미널에 권한 프롬프트를 표출합니다. 이 섹션의 나머지 부분에서는 명명된 서브에이전트에 대해 설명합니다.

명명된 서브에이전트가 사용할 수 있는 도구는 [서브에이전트 정의](/docs/en/sub-agents)의 `tools` 및 `disallowedTools` 필드에 따라 다릅니다:

* **두 필드 모두 설정되지 않음**: 서브에이전트는 상위 대화에서 사용할 수 있는 모든 도구를 상속받습니다.
* **`tools`만 설정됨**: 서브에이전트는 나열된 도구만 받습니다.
* **`disallowedTools`만 설정됨**: 서브에이전트는 나열된 도구를 제외한 모든 상위 도구를 받습니다.
* **둘 다 설정됨**: `disallowedTools`가 우선합니다. 둘 다에 나열된 도구는 제거됩니다.

서브에이전트의 `tools` 목록이 도구로 해석되지 않는 경우(예: 모든 항목의 철자가 틀렸거나 서브에이전트에 사용할 수 없는 도구 이름인 경우) Agent 도구는 서브에이전트를 시작하는 대신 해당 항목을 나열하는 오류를 반환합니다. {/* min-version: 2.1.208 */} v2.1.208 이전에는 서브에이전트가 도구 없이 시작되어 빈 결과나 혼란스러운 결과를 반환할 수 있었습니다.

서브에이전트를 실행하는 것 자체가 권한 프롬프트를 표시하지는 않습니다. Claude Code는 서브에이전트가 실행될 때 서브에이전트 자체의 도구 호출을 권한 규칙과 비교하여 검사합니다.

{/* min-version: 2.1.198 */} v2.1.198부터 서브에이전트는 기본적으로 백그라운드에서 실행됩니다. Claude는 계속하기 전에 결과가 필요할 때 포그라운드에서 실행합니다.

* **포그라운드 서브에이전트**: 각 도구 호출이 발생하는 순간 메인 대화에서 볼 수 있는 것과 동일한 권한 프롬프트를 표시합니다.
* **백그라운드 서브에이전트**: {/* min-version: 2.1.186 */} v2.1.186부터 메인 세션에 권한 프롬프트를 표출합니다. 프롬프트는 요청하는 서브에이전트의 이름을 명시하며, Esc를 누르면 서브에이전트를 중지하지 않고 해당 단일 도구 호출을 거부합니다. v2.1.186 이전에는 백그라운드 서브에이전트가 그렇지 않으면 프롬프트를 표시할 도구 호출을 자동으로 거부하고 해당 도구 없이 계속 진행했습니다.

서브에이전트가 접근할 수 있는 범위를 사전에 제한하려면 [Control subagent capabilities](/docs/en/sub-agents#control-subagent-capabilities)에 설명된 대로 `tools` 필드를 좁히거나, 목록에서 Bash를 제외하거나, 설정에서 deny 규칙을 설정하세요. 포그라운드와 백그라운드 중 선택하는 방법에 대한 자세한 내용은 [Run subagents in foreground or background](/docs/en/sub-agents#run-subagents-in-foreground-or-background)를 참조하세요.

## AskUserQuestion 도구 동작

Claude는 결정이나 명확화가 필요할 때 객관식 질문을 하기 위해 `AskUserQuestion`을 사용합니다. 옵션을 선택하여 답변하거나, `Other` 행이나 메모 필드를 통해 직접 텍스트를 입력하세요.

직접 텍스트를 입력하여 답변하는 경우 Claude Code는 Claude가 먼저 기다리거나 설명해 달라는 요청을 포함하여 작성한 내용을 따르도록 중립적인 표현으로 답변을 전달합니다.

### 질문 자동 진행 시간 초과

질문은 답변할 때까지 열려 있습니다. 답변하지 않고 방치한 질문이 결국 닫혀서 Claude가 사용자 없이 계속 진행되도록 하려면 사용자 `settings.json` 또는 `/config`의 **Question auto-continue timeout** 행에서 [`askUserQuestionTimeout`](/docs/en/settings#available-settings) 설정을 `60s`, `5m`, 또는 `10m`으로 설정하세요.

질문이 입력 없이 해당 시간 동안 방치되면 대화 상자가 자체적으로 닫힙니다. 이미 선택한 옵션을 제출하고 Claude에게 키보드에서 멀어져 있을 수 있다고 알려주므로 Claude는 자체 판단에 따라 진행하며 나중에 다시 물어볼 수 있습니다. 마지막 20초 동안 카운트다운이 표시됩니다. 타이머를 재시작하려면 아무 키나 누르세요. 포커스를 보고하는 터미널에서는 해당 창으로 전환해도 재시작됩니다.

시간 초과는 `AskUserQuestion`의 객관식 질문에만 적용되며, 플랜 승인을 포함한 권한 프롬프트는 유휴 상태에서 자동으로 해결되지 않습니다.

## Bash 도구 동작

Bash 도구는 별도의 프로세스에서 각 명령을 실행합니다.

### 명령 간 유효하게 유지되는 항목

* Claude가 메인 세션에서 `cd`를 실행할 때, 프로젝트 디렉터리 내부에 있거나 `--add-dir`, `/add-dir` 또는 설정의 `additionalDirectories`로 추가한 [추가 작업 디렉터리](/docs/en/permissions#working-directories) 내에 있는 한 새로운 작업 디렉터리가 이후의 Bash 명령으로 이월됩니다. 서브에이전트 세션은 작업 디렉터리 변경 사항을 이월하지 않습니다.
  * `cd`가 해당 디렉터리 외부에 도달하는 경우 Claude Code는 프로젝트 디렉터리로 재설정하고 도구 결과에 `Shell cwd was reset to <dir>`을 덧붙입니다.
  * 모든 Bash 명령이 프로젝트 디렉터리에서 시작하도록 이 이월 기능을 비활성화하려면 `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1`을 설정하세요.
* 환경 변수는 유지되지 않습니다. 한 명령의 `export`는 다음 명령에서 사용할 수 없습니다.
* 쉘 시작 파일에 정의된 별칭(alias) 및 쉘 함수를 사용할 수 있습니다. 세션 시작 시 Claude Code는 쉘에 따라 `~/.zshrc`, `~/.bashrc`, 또는 `~/.profile`을 소싱하고, 결과로 생성된 별칭, 함수 및 쉘 옵션을 캡처하여 모든 Bash 명령에 적용합니다.

Claude Code를 실행하기 전에 virtualenv 또는 conda 환경을 활성화하세요. 환경 변수를 Bash 명령 간에 유지하려면 Claude Code를 실행하기 전에 [`CLAUDE_ENV_FILE`](/docs/en/env-vars)을 쉘 스크립트로 설정하거나 [SessionStart hook](/docs/en/hooks#persist-environment-variables)을 사용하여 동적으로 채우세요.

### 시간 초과 및 출력 제한

각 명령에는 두 가지 제한이 적용됩니다:

* **시간 초과(Timeout)**: 기본 2분. Claude는 `timeout` 매개변수로 명령당 최대 10분을 요청할 수 있습니다. [`BASH_DEFAULT_TIMEOUT_MS` 및 `BASH_MAX_TIMEOUT_MS`](/docs/en/env-vars)로 기본값과 상한선을 재정의하세요.
* **출력 길이(Output length)**: 기본 30,000자. 명령이 그보다 많은 출력을 생성할 때 Claude Code는 전체 출력을 세션 디렉터리의 파일에 저장하고 시작 부분의 짧은 미리보기와 함께 Claude에게 파일 경로를 제공합니다. Claude는 나머지가 필요할 때 해당 파일을 읽거나 검색합니다. [`BASH_MAX_OUTPUT_LENGTH`](/docs/en/env-vars)로 한도를 최대 150,000자의 하드 상한선까지 늘릴 수 있습니다.

### 백그라운드 명령

개발 서버나 감시 빌드와 같은 장시간 실행 프로세스의 경우 Claude는 `run_in_background: true`를 설정하여 명령을 백그라운드 작업으로 시작하고 실행되는 동안 작업을 계속할 수 있습니다. `/tasks`로 백그라운드 작업을 나열하고 중지하세요. `-p` 플래그가 있는 비대화형 모드에서는 [실행의 최종 결과 직후에 백그라운드 작업이 종료됩니다](/docs/en/headless#background-tasks-at-exit).

명령이 완료되지 않고 시간 초과에 도달하면 Claude Code는 명령을 중지하는 대신 백그라운드로 이동하므로 명령이 완료될 때까지 실행되는 동안 Claude는 작업을 계속합니다. Claude Code는 `sleep`으로 시작하는 명령을 절대로 자동 백그라운드 처리하지 않으며, [`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`](/docs/en/env-vars#variables)을 설정하면 다른 백그라운드 작업 기능과 함께 자동 백그라운드 처리가 비활성화됩니다. 이 방식으로 이동된 명령의 결과는 일어난 상황을 명시합니다:

* {/* min-version: 2.1.210 */} 시간 초과로 이동이 트리거되면 결과에 이를 명시적으로 보고합니다: `Command did not complete within its 120s timeout and was moved to the background`(초는 적용된 시간 초과와 일치함). 그 뒤에 작업 ID와 출력이 작성되는 파일 경로가 이어집니다.
* {/* min-version: 2.1.210 */} 백그라운드로 이동된 명령 내부의 `cd`, `pushd`, `popd`, 또는 `chdir`은 절대로 이월되지 않습니다. 결과는 `Session cwd remains <dir>; directory changes made by the backgrounded command do not apply to subsequent commands.`를 명시하므로 Claude는 일어나지 않은 디렉터리 변경에 대해 조치를 취하지 않습니다.

## Edit 도구 동작

Edit 도구는 정확한 문자열 교체를 수행합니다. `old_string`과 `new_string`을 받아 첫 번째 문자열을 두 번째 문자열로 교체합니다. 정규식이나 모호한 일치(fuzzy matching)를 사용하지 않습니다.

편집이 적용되려면 세 가지 검사를 통과해야 합니다. {/* min-version: 2.1.208 */} 이들보다 먼저, [`Read` 거부 규칙](/docs/en/permissions#tool-specific-permission-rules)과 일치하는 경로는 거부되며, 거기에 새 파일을 생성하는 것도 포함됩니다. 이 거부는 Claude Code v2.1.208 이상이 필요합니다.

* **편집 전 읽기**: Claude는 편집하기 전에 현재 대화에서 파일을 읽으며, [`PARTIAL view` 알림](#read-tool-behavior)으로 짧게 잘린 읽기는 카운트되지 않습니다. Claude Opus 4.6, Claude Haiku 4.5 및 이전 모델은 항상 읽기를 요구합니다. 최신 모델은 파일을 읽는 데 권한 프롬프트가 필요하지 않고 Read 도구를 사용할 수 있을 때 읽지 않은 파일을 편집할 수 있습니다.
* **일치**: `old_string`이 파일에 작성된 그대로 정확히 나타나야 합니다. 공백이나 들여쓰기의 단 한 문자 차이로도 일치에 실패할 수 있습니다.
* **고유성**: `old_string`이 정확히 한 번 나타나야 합니다. 두 번 이상 나타나는 경우 Claude는 하나의 발생을 고정할 수 있도록 충분한 주변 컨텍스트가 포함된 더 긴 문자열을 제공하거나, 모두 교체하기 위해 `replace_all: true`를 설정합니다.

Claude가 마지막으로 읽은 후 디스크에서 변경된 파일이라도 `old_string`이 현재 내용과 정확히 일치하고 명확하며 Claude Code가 프롬프트 없이 파일을 읽을 수 있는 경우에는 여전히 편집할 수 있습니다. 파일의 현재 내용과 일치시키면 안전하게 유지되며, 결과는 파일에 다른 변경 사항이 포함되어 있음을 알려주므로 Claude는 주변 내용에 의존하는 편집 전에 파일은 다시 읽습니다. 만료된 `old_string` 또는 `replace_all` 없이 두 번 이상 일치하는 경우와 같은 기타 모든 사례에서 Claude는 편집 전에 파일을 다시 읽습니다. {/* min-version: 2.1.208 */} 읽지 않은 파일 및 변경된 파일에 대한 완화된 처리는 Claude Code v2.1.208 이상이 필요합니다. 그 전에는 Claude Code가 대화에서 읽지 않았거나 읽은 후 디스크에서 변경된 파일에 대한 모든 편집을 거부했습니다.

Bash로 파일을 보는 것도 하나의 파일에서 파이프나 리디렉션 없이 `cat`, `head`, `tail`, `sed -n 'X,Yp'`, `grep`, `egrep`, 또는 `fgrep` 명령인 경우 편집 전 읽기 요구 사항을 충족합니다. 파이프 연결된 출력 및 기타 Bash 명령은 편집 전 읽기 검사에 카운트되지 않습니다.

 이는 편집 자격에만 영향을 주며 권한에는 영향을 주지 않습니다. [Read 및 Edit 거부 규칙](/docs/en/permissions#tool-specific-permission-rules)은 `cat`, `head`, `tail`, `sed`, `grep`과 같이 Claude Code가 Bash에서 인식하는 파일 명령에도 적용되지만, 파일을 직접 여는 Python 또는 Node 스크립트처럼 파일을 간접적으로 읽거나 쓰는 임의의 하위 프로세스에는 적용되지 않습니다. 거부 규칙에 대해 인식되는 명령 세트는 위의 편집 전 읽기 목록과 동일하지 않습니다. 예를 들어 `egrep` 및 `fgrep`은 편집 전 읽기에는 카운트되지만 Read 거부 규칙에 대해서는 검사되지 않습니다. 모든 프로세스를 포함하는 OS 수준 강제 적용을 위해서는 [샌드박스를 활성화](/docs/en/sandboxing)하세요.

## EndConversation 도구 동작

EndConversation 도구는 현재 세션을 종료합니다. Claude는 다음 두 가지 상황에서만 이 도구를 사용합니다:

* 대화를 전환하려는 시도가 실패하고 이전 메시지에서 명확한 경고를 보낸 후 지속적인 악성 입력에 대응하기 위한 최종 수단
* 도구 시연을 명시적으로 요청하고 세션 종료를 확인한 경우

일반적인 좌절감, 욕설 또는 작업이 잘 진행되지 않는 것은 자격이 되지 않으며, 유해한 콘텐츠에 대한 요청도 자격이 되지 않으므로 Claude는 세션을 종료하는 대신 거절합니다. Claude Code는 [드문 대화 세트를 종료](https://www.anthropic.com/research/end-subset-conversations)할 수 있는 claude.ai와 동일한 접근 방식을 따릅니다.

Claude가 대화형 세션을 종료한 후 세션이 잠깁니다. 새로운 프롬프트 및 대부분의 명령은 `Claude ended this conversation. Start a new session (or /clear) to continue.`를 반환하며 `/clear`, `/resume`, `/help`, `/exit`, `/feedback`만 계속 실행됩니다. Claude Code는 세션 트랜스크립트에 종료를 기록하므로 종료된 세션을 재개하면 잠금이 복원되며, 세션 기록은 삭제되지 않습니다.

[-p` 플래그를 사용하여 비대화형 모드](/docs/en/headless)에서 종료된 세션을 재개하면 스크립트가 종료된 실행을 성공으로 읽지 않도록 오류가 발생하고 코드 1로 종료됩니다.

이 도구는 결코 권한을 요청하지 않으며 [PreToolUse hooks](/docs/en/hooks#pretooluse)도 이 도구에 대해 실행되지 않습니다. 다른 도구가 남아있는 동안에는 이 도구를 차단할 수도 없습니다. `EndConversation`을 지정하는 [거부 및 요청 규칙](/docs/en/permissions#tool-specific-permission-rules)은 효과가 없으며, `--disallowedTools`나 `--tools` 목록도 이를 제거할 수 없습니다. 이 면제는 의도적인 것입니다. 도구는 파일이나 데이터를 읽거나 수정하지 않고 대화를 종료하는 작업만 수행하며, 이러한 종류의 보호 조치는 그것이 적용되는 세션이 끄지 못해야만 유지됩니다. `"*"`와 같이 거부 목록이 다른 모든 도구를 제거하고 `EndConversation`과도 일치하는 경우, 허용 규칙이 `EndConversation`을 명시적으로 지정하지 않는 한 Claude Code는 이를 유일한 도구로 남겨두지 않고 함께 제거합니다. `EndConversation`과 일치하지 않고 다른 모든 도구를 제거하는 거부 목록은 이 도구를 그대로 둡니다.

[서브에이전트](/docs/en/sub-agents)는 이 도구를 결코 받지 못합니다. 메인 대화의 도구 목록을 공유하는 백그라운드 작업에는 이 도구가 표시되지만 거기서 호출해도 아무것도 종료되지 않습니다.

이 도구는 다음 사항이 모두 충족될 때만 표시됩니다:

* **버전**: {/* min-version: 2.1.213 */} Claude Code v2.1.213 이상.
* **모델**: 세션 모델이 Claude Opus 4.8, Claude Sonnet 5, Claude Fable 5, 또는 해당 패밀리의 최신 버전.
* **인터페이스**: IDE의 통합 터미널에 있는 `claude` 세션([JetBrains 플러그인](/docs/en/jetbrains)이 실행하는 방식)을 포함한 대화형 터미널 세션. 다음과 같은 다른 인터페이스에는 이 도구가 포함되지 않습니다:
  * 비대화형 `-p` 실행
  * [Agent SDK](/docs/en/agent-sdk/overview) TypeScript 및 Python 패키지를 통한 세션
  * 자체 CLI를 번들로 제공하는 [VS Code 확장 프로그램](/docs/en/vs-code) 패널
  * [GitHub Actions](/docs/en/github-actions)
  * [Claude Code on the web](/docs/en/claude-code-on-the-web)
* **시작 모드**: [`--bare`](/docs/en/headless#start-faster-with-bare-mode) 세션이 아님. Bare 모드는 쉘 및 파일 도구만 로드하므로 도구가 거기에 등록되지 않습니다.
* **제공업체**: {/* plan-availability: feature=end-conversation providers=anthropic */} [Amazon Bedrock](/docs/en/amazon-bedrock), [Claude Platform on AWS](/docs/en/claude-platform-on-aws), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry), 또는 [클라우드 게이트웨이](/docs/en/claude-apps-gateway)를 통해 로그인한 세션에서는 사용할 수 없습니다.

## Glob 도구 동작

Glob 도구는 이름 패턴으로 파일을 찾습니다. 재귀 디렉터리 일치를 위한 `**`를 포함하여 표준 glob 구문을 지원합니다:

* `**/*.js`는 모든 깊이의 모든 `.js` 파일과 일치합니다
* `src/**/*.ts`는 `src/` 아래의 모든 `.ts` 파일과 일치합니다
* `*.{json,yaml}`은 현재 디렉터리의 `.json` 및 `.yaml` 파일과 일치합니다

결과는 수정 시간별로 정렬되며 100개 파일로 제한됩니다. 한도에 도달하면 Claude는 결과에서 잘림 플래그를 보고 패턴을 좁힐 수 있습니다.

Glob은 기본적으로 `.gitignore`를 준수하지 않으므로 추적 대상 파일과 함께 gitignore 처리된 파일을 찾습니다. 이는 gitignore 처리된 파일을 건너뛰는 [Grep](#grep-tool-behavior)과 다릅니다. Glob이 `.gitignore`를 준수하도록 하려면 Claude Code를 실행하기 전에 `CLAUDE_CODE_GLOB_NO_IGNORE=false`를 설정하세요.

널 바이트(null byte)가 포함된 `pattern` 또는 `path` 값은 이를 제거하도록 요청하는 오류를 반환합니다. {/* min-version: 2.1.208 */}

## Grep 도구 동작

Grep 도구는 파일 내용에서 패턴을 검색합니다. [Glob](#glob-tool-behavior)이 이름으로 파일을 찾는 것과 달리 Grep은 파일 내부의 줄을 찾습니다.

Grep은 [ripgrep](https://github.com/BurntSushi/ripgrep)을 기반으로 구축되었으며 POSIX grep이 아닌 ripgrep의 정규식 구문을 사용합니다. 정규식 메타문자가 포함된 패턴은 이스케이프 처리가 필요합니다. 예를 들어 Go 코드에서 `interface{}`를 찾으려면 `interface\{\}` 패턴을 사용해야 합니다.

ripgrep이 거부하는 패턴, glob 또는 파일 유형은 ripgrep의 진단이 포함된 오류를 반환하므로 Claude가 입력을 수정하고 다시 검색할 수 있습니다. {/* min-version: 2.1.208 */} v2.1.208 이전에는 검색된 텍스트가 대상 파일에 존재하는 경우에도 Claude Code가 거부된 입력을 오류 대신 `No files found`로 보고했습니다.

세 가지 출력 모드가 반환되는 내용을 제어합니다:

* `files_with_matches`: 파일 경로만 표시하고 줄 내용은 표시하지 않음. 이것이 기본값입니다.
* `content`: 파일 및 줄 번호와 함께 일치하는 줄 표시. {/* min-version: 2.1.210 */} 도구의 `offset` 매개변수가 일치 항목이 있는 패턴의 마지막 일치 항목을 지나는 위치를 가리킬 때, Grep은 `No entries at this offset`을 반환하므로 Claude가 패턴이 일치하지 않는다고 결론 내리는 대신 오프셋을 넓히거나 재설정합니다.
* `count`: 파일당 일치 횟수 표시 후 모든 일치 파일에 걸친 합계 표시. {/* min-version: 2.1.208 */} 합계는 도구의 `head_limit` 또는 `offset` 매개변수가 나열된 파일별 항목을 잘라내는 경우에도 모든 일치 항목을 포함합니다. v2.1.208 이전에는 합계가 나열된 항목만 합산했습니다.

Claude는 `**/*.tsx`와 같은 `glob` 매개변수로 결과 범위를 파일별로 지정하거나 `py` 또는 `rust`와 같은 `type` 매개변수로 언어별로 지정할 수 있습니다. 기본적으로 패턴은 단일 줄 내에서 일치합니다. Claude는 줄 경계를 넘어 일치하도록 `multiline: true`를 설정할 수 있습니다.

Grep은 `.gitignore`를 준수하므로 gitignore 처리된 파일은 건너뜁니다. gitignore 처리된 파일을 검색하려면 Claude가 해당 경로를 직접 전달합니다.

## LSP 도구 동작

LSP 도구는 실행 중인 언어 서버로부터 코드 인텔리전스를 Claude에게 제공합니다. 각 파일 편집 후 별도의 빌드 단계 없이 문제를 수정할 수 있도록 타입 오류와 경고를 자동으로 보고합니다. Claude는 코드를 탐색하기 위해 직접 호출할 수도 있습니다:

* 심볼의 정의로 이동
* 심볼에 대한 모든 참조 찾기
* 특정 위치에서 타입 정보 가져오기
* 파일 내 심볼 나열
* 작업 공간 전체에서 이름으로 심볼 검색
* 인터페이스의 구현체 찾기
* 호출 계층 구조 추적

해당 언어를 위한 [코드 인텔리전스 플러그인](/docs/en/discover-plugins#code-intelligence)을 설치할 때까지 도구는 비활성화 상태입니다. 플러그인은 언어 서버 구성을 번들로 제공하며, 언어 서버 바이너리는 별도로 설치합니다.

## Monitor 도구

Monitor 도구를 사용하면 대화를 일시 중지하지 않고도 Claude가 백그라운드에서 무언가를 감시하고 변경될 때 반응할 수 있습니다. Claude에게 다음을 요청하세요:

* 로그 파일을 추적(tail)하고 오류가 나타날 때 플래그 지정
* PR 또는 CI 작업을 폴링하고 상태가 변경될 때 보고
* 디렉터리의 파일 변경 사항 감시
* 지정한 장시간 실행 스크립트의 출력 추적
* WebSocket 피드에 연결하고 각 메시지가 도착할 때 보고

대부분의 감시 작업의 경우 Claude는 소형 스크립트를 작성하고 백그라운드에서 실행하며 각 출력 라인이 도착할 때 이를 받습니다. 이미 이벤트를 푸시하는 서버의 경우 Claude는 스크립트를 실행하는 대신 [WebSocket](#websocket-source)을 열 수 있습니다.

동일한 세션에서 작업을 계속하고 이벤트가 도착하면 Claude가 끼어듭니다. Claude에게 취소를 요청하거나 세션을 종료하여 모니터를 중지하세요.

Monitor가 명령을 실행할 때 [Bash와 동일한 권한 규칙](/docs/en/permissions#tool-specific-permission-rules)을 사용하므로 Bash에 대해 설정한 `allow` 및 `deny` 패턴이 여기에도 적용됩니다. [WebSocket source](#websocket-source)에는 자체 승인 프롬프트가 있습니다.

이 도구는 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry에서는 사용할 수 없습니다. `DISABLE_TELEMETRY` 또는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`이 설정되어 있을 때도 사용할 수 없습니다.

플러그인은 Claude에게 시작을 요청하는 대신 플러그인이 활성화될 때 자동으로 시작되는 모니터를 선언할 수 있습니다. [plugin monitors](/docs/en/plugins-reference#monitors)를 참조하세요.

### WebSocket 소스 (WebSocket source)

<Note>
  WebSocket 소스는 Claude Code v2.1.195 이상이 필요합니다.
</Note>

서버가 이미 WebSocket을 통해 이벤트를 푸시할 때 Claude는 폴링 스크립트를 작성하는 대신 서버에 직접 연결할 수 있습니다. 각 소켓 활동 유형은 이벤트가 되거나 감시를 종료합니다:

* **텍스트 메시지**: 메시지가 여러 줄에 걸쳐 있더라도 각각 하나의 이벤트가 됩니다.
* **바이너리 메시지**: 통과되지 않습니다. Claude는 대신 `[binary frame, 512 bytes]`와 같은 자리 표시자 라인을 받습니다.
* **1 MiB보다 큰 메시지**: 감시가 종료되므로 존재하는 경우 필터링된 피드에 구독하세요.
* **소켓 닫힘**: 감시가 종료되고 Claude가 종료 코드를 받습니다.

WebSocket 감시는 `command` 대신 `ws` 입력을 사용하며 단일 Monitor 호출은 두 가지를 결합할 수 없습니다. `ws` 입력에는 두 개의 필드가 있습니다:

| 필드 | 필수 여부 | 설명 |
| :---------- | :------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| `url` | 예 | 연결할 엔드포인트. 퍼베이시브 자격 증명이나 공백이 없고 ASCII 문자만 사용하는 `ws://` 또는 `wss://` URL이어야 함 |
| `protocols` | 아니오 | 핸드셰이크 중에 제공할 WebSocket 서브프로토콜 이름. 각 항목은 유효한 서브프로토콜 토큰이어야 하며 목록에 중복이 포함될 수 없음 |

`timeout_ms` 및 `persistent` 입력은 명령에서와 동일하게 동작합니다. `persistent`가 설정되어 있지 않으면 기한에 감시가 종료되고 `TaskStop`이 조기에 취소합니다.

WebSocket을 열면 승인을 요청하는 프롬프트가 표시되며 해당 프롬프트는 동일한 호스트에 대한 향후 프롬프트를 건너뛰는 옵션을 제공하지 않습니다.

Claude Code는 사설, 링크 로컬, 또는 클라우드 메타데이터 주소를 가리키는 URL(해당 주소로 해석되는 호스트 이름 포함)을 거부합니다. 또한 `sandbox.network.deniedDomains`에 있는 호스트를 거부하며, 관리 설정에 [`allowManagedDomainsOnly`](/docs/en/settings#sandbox-settings)가 설정된 경우 관리형 허용 목록 외의 모든 호스트를 거부합니다.

## NotebookEdit 도구 동작

NotebookEdit는 `cell_id`로 셀을 타겟팅하여 Jupyter 노트북을 한 번에 한 셀씩 수정합니다. 일반 파일에서 [Edit](#edit-tool-behavior)가 수행하는 방식으로 노트북 전체에 걸쳐 문자열 교체를 수행하지 않습니다.

세 가지 편집 모드가 대상 셀에 발생하는 상황을 제어합니다:

* `replace`: 셀의 소스를 덮어씁니다. 이것이 기본값입니다.
* `insert`: 대상 뒤에 새 셀을 추가합니다. `cell_id`가 없으면 새 셀이 노트북 시작 부분에 추가됩니다. `cell_type`이 `code` 또는 `markdown`으로 설정되어야 합니다.
* `delete`: 대상 셀을 제거합니다.

권한 규칙은 `Edit(...)` 경로 형식을 사용합니다. `Edit(notebooks/**)`와 같은 규칙은 해당 디렉터리의 파일에 대한 NotebookEdit 호출을 처리합니다.

## PowerShell 도구

PowerShell 도구를 사용하면 Claude가 네이티브로 PowerShell 명령을 실행할 수 있습니다. Windows에서 이는 명령이 Git Bash를 거치지 않고 PowerShell에서 실행됨을 의미합니다. 도구를 사용할 수 있게 되는 방식은 플랫폼에 따라 다릅니다:

* **Git Bash가 없는 Windows**: 도구가 자동으로 활성화됩니다.
* **Git Bash가 설치된 Windows**: 도구가 점진적으로 배포 중입니다.
* **Linux, macOS, 및 WSL**: 도구가 옵트인(opt-in) 방식입니다.

### PowerShell 도구 활성화

환경 변수나 `settings.json`에서 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`을 설정하세요:

```json theme={null}
{
  "env": {
    "CLAUDE_CODE_USE_POWERSHELL_TOOL": "1"
  }
}
```

Windows에서는 롤아웃에서 제외(opt-out)되도록 변수를 `0`으로 설정하세요. Linux, macOS 및 WSL에서 이 도구는 PowerShell 7 이상이 필요합니다. `pwsh`를 설치하고 `PATH`에 있는지 확인하세요.

Windows에서 Claude Code는 PowerShell 5.1에 대한 대체(fallback)와 함께 PowerShell 7+용 `pwsh.exe`를 자동 감지합니다. 도구가 활성화되면 Claude는 PowerShell을 기본 쉘로 취급합니다. Git Bash가 설치되어 있으면 POSIX 스크립트에 대해 Bash 도구를 계속 사용할 수 있습니다.

Claude Code는 프로세스 범위에서만 `-ExecutionPolicy Bypass`를 사용하여 PowerShell을 실행하므로 머신의 정책을 변경하지 않고도 기본 Windows 설치에서 `.ps1` 스크립트 및 모듈 가져오기가 작동합니다. 프로세스 범위 우회는 그룹 정책 `MachinePolicy` 또는 `UserPolicy`를 재정의하지 않으므로 기업 정책이 여전히 적용됩니다. 머신의 유효한 실행 정책을 존중하려면 `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY=1`을 설정하세요.

### 설정, 훅 및 skill에서의 쉘 선택

세 가지 추가 설정이 PowerShell이 사용되는 위치를 제어합니다:

* [`settings.json`](/docs/en/settings#available-settings)의 `"defaultShell": "powershell"`: 대화형 `!` 명령을 PowerShell을 통해 라우팅합니다. PowerShell 도구가 활성화되어 있어야 합니다.
* 개별 [명령 훅](/docs/en/hooks#command-hook-fields)의 `"shell": "powershell"`: 해당 훅을 PowerShell에서 실행합니다. 훅은 PowerShell을 직접 스폰하므로 `CLAUDE_CODE_USE_POWERSHELL_TOOL`과 관계없이 작동합니다.
* [skill 프롬프트](/docs/en/skills#frontmatter-reference)의 `shell: powershell`: PowerShell에서 `` !`command` `` 블록을 실행합니다. PowerShell 도구가 활성화되어 있어야 합니다.

Bash 도구 섹션에 설명된 동일한 메인 세션 작업 디렉터리 재설정 동작이 `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` 환경 변수를 포함하여 PowerShell 명령에 적용됩니다.

{/* min-version: 2.1.196 */} v2.1.196부터 PowerShell 도구는 검색 및 diff 종료 코드에 대한 Bash 도구의 처리와 일치합니다. `grep`, `egrep`, `fgrep`, `git grep`의 종료 코드 1은 일치 항목이 없음을 의미하고 `git diff`의 종료 코드 1은 차이점이 존재함을 의미하므로 이러한 결과는 Claude에 명령 실패로 보고되지 않습니다.

### Windows 인코딩 및 종료 코드

Windows에서 다음 PowerShell 인코딩 및 종료 코드 동작은 Claude Code v2.1.214 이상이 필요합니다:

* `>` 및 `>>`를 통한 리디렉션은 PowerShell 5.1에서 UTF-8 파일을 작성합니다
* Claude Code는 네이티브 명령의 표준 입력으로 파이프 연결된 텍스트를 UTF-8로 인코딩합니다
* Claude Code는 ANSI 이스케이프 시퀀스 없이 오류 출력을 캡처합니다
* 자식 프로세스가 표준 입력을 기다리는 명령은 멈추는(hang) 대신 파일 끝(EOF)을 받습니다
* `where.exe`에서 종료 코드 1은 일치 항목 없음을 의미하고 `fc.exe` 및 `diff.exe`에서는 파일이 다름을 의미하므로 명령이 출력을 생성할 때 Claude Code는 해당 종료 코드를 명령 오류가 아닌 유효한 부정적인 답변으로 취급합니다. Claude Code는 `where.exe /Q` 또는 `$null`로의 리디렉션과 같은 묵음 처리된 형식을 종료 코드 1에서 실패로 여전히 보고합니다

v2.1.214 이전에는 PowerShell 5.1의 `>`가 UTF-16LE 파일을 작성했고, 비-ASCII 파이프 입력이 `?`로 도착했으며, 비-ASCII 문자를 출력할 때 Python 스크립트가 `UnicodeEncodeError`로 크래시될 수 있었습니다.

### 프리뷰 제한 사항

PowerShell 도구에는 프리뷰 동안 다음과 같은 알려진 제한 사항이 있습니다:

* PowerShell 프로필이 로드되지 않음
* Windows에서는 샌드박싱이 지원되지 않음

## Read 도구 동작

Read 도구는 파일 경로를 받아 줄 번호와 함께 내용을 반환합니다. Claude는 항상 절대 경로를 전달하도록 지시받습니다.

기본적으로 Read는 시작 부분부터 파일을 반환합니다. 전체 파일 읽기가 토큰 한도를 초과할 때 Read는 Claude에게 얼마나 많은 파일 내용을 받았는지와 `offset` 및 `limit`로 더 읽는 방법을 알려주는 `PARTIAL view` 알림과 함께 첫 페이지를 반환합니다. 명시적인 `offset` 또는 `limit`를 전달하고도 토큰 한도를 초과하는 읽기는 오류를 반환합니다.

명시적인 `limit`가 있는 읽기는 선택한 줄이 토큰 한도에 맞춰질 수 있는 수준을 초과하는 즉시 중지되고 범위의 나머지를 로드하지 않고 오류를 반환합니다. 단일 줄이 그만큼 클 때 오류는 Claude에게 더 작은 `limit`를 사용하거나 대신 [Grep](#grep-tool-behavior)으로 특정 내용을 검색하도록 알려줍니다. {/* min-version: 2.1.208 */} v2.1.208 이전에는 Claude Code가 이를 거부하기 전에 전체 범위를 메모리로 로드했으므로 극도로 긴 단일 줄이 있는 파일을 읽으면 메모리가 부족해질 수 있었습니다.

빈 파일을 읽으면 파일은 존재하지만 내용이 비어 있다는 알림을 반환하며, 마지막 줄을 지나는 `offset`은 파일의 줄 수를 알려주는 알림을 반환합니다. {/* min-version: 2.1.208 */} v2.1.208 이전에는 빈 파일을 읽으면 대신 끝을 지났다는 알림을 반환했습니다.

Read는 일반 텍스트 외에 여러 파일 유형을 처리합니다:

* **이미지**: PNG, JPG 및 기타 이미지 형식은 날것의 바이트가 아니라 Claude가 볼 수 있는 시각적 콘텐츠로 반환됩니다. Claude Code는 이미지를 보내기 전에 모델의 이미지 크기 한도에 맞게 대형 이미지의 크기를 조정하고 다시 압축하므로 Claude는 대형 스크린샷의 축소된 버전을 볼 수 있습니다. {/* min-version: 2.1.196 */} v2.1.196부터 해당 크기 조정 후에도 여전히 500KB보다 큰 이미지는 픽셀 치수를 변경하지 않고 줄어든 품질의 JPEG로 다시 인코딩됩니다. Claude가 대형 이미지에서 미세한 픽셀 수준 세부 정보를 놓치는 경우, 예를 들어 Bash를 통해 ImageMagick을 사용하여 관심 영역을 먼저 자르도록 요청하세요.
* **PDF**: Claude는 짧은 `.pdf` 파일을 전체적으로 읽습니다. 10페이지보다 긴 PDF의 경우 한 번에 최대 20페이지까지 `"1-5"`와 같은 `pages` 매개변수로 범위를 지정하여 읽습니다.
* **Jupyter 노트북**: `.ipynb` 파일은 코드, markdown 및 시각화를 포함하여 모든 셀을 출력과 함께 반환합니다.

Read는 파일만 읽으며 디렉터리는 읽지 않습니다. Claude는 디렉터리 내용을 나열하기 위해 Bash 도구를 통해 `ls`를 사용합니다.

## WebFetch 도구 동작

WebFetch는 URL과 추출할 내용을 설명하는 프롬프트를 받아들입니다. 페이지를 가져오고, 서버가 HTML을 반환할 때 응답을 Markdown으로 변환하며, 작고 빠른 모델을 사용하여 콘텐츠에 대해 프롬프트를 실행합니다. 대부분의 가져오기에 대해 Claude는 날것의 페이지가 아니라 해당 모델의 답변을 받습니다. 변환 단계는 구성할 수 없습니다.

이로 인해 WebFetch는 설계를 통해 손실(lossy)이 발생합니다. 추출 프롬프트가 Claude에 도달하는 내용을 결정하므로 페이지가 무언가를 언급하지 않는다는 결과는 프롬프트가 그것에 대해 묻지 않았음을 의미할 수 있습니다. Claude에게 더 구체적인 프롬프트로 다시 가져오도록 요청하거나 미처리된 페이지의 경우 Bash를 통해 `curl`을 사용하세요.

몇 가지 동작이 Claude가 받는 응답을 형성합니다:

* HTTP URL은 자동으로 HTTPS로 업그레이드됩니다.
* 대형 페이지는 처리 전에 고정된 문자 수 한도로 잘립니다.
* 응답은 15분 동안 캐시되므로 동일한 URL을 반복해서 가져오면 빠르게 반환됩니다.
* URL이 다른 호스트로 리디렉션될 때 WebFetch는 이를 따르는 대신 원래 URL과 리디렉션 대상을 명시하는 텍스트 결과를 반환합니다. Claude는 두 번째 WebFetch 호출로 새 URL을 가져옵니다.
* {/* min-version: 2.1.212 */} 추출 단계가 과부하된 API에 도달하면 Claude Code는 지연(backoff) 후 재시도하며, 계속 실패하는 가져오기는 오류 결과를 반환합니다. v2.1.212 이전에는 API 오류 텍스트가 추출된 페이지 내용인 것처럼 Claude에 도달할 수 있었습니다.

기본 및 `acceptEdits` 권한 모드에서 WebFetch는 프롬프트 없이 가져오는 사전 승인된 내장 문서 도메인 세트를 제외하고 새로운 도메인에 처음 도달할 때 프롬프트를 표시합니다. 프롬프트 없이 사전에 다른 도메인을 허용하려면 `WebFetch(domain:example.com)`과 같은 권한 규칙을 추가하세요. `auto` 및 `bypassPermissions` [권한 모드](/docs/en/permissions#permission-modes)는 프롬프트를 완전히 건너끕니다.

`deny`, `ask`, 또는 `allow`에 명시된 `WebFetch(domain:...)` 규칙은 사전 승인된 세트보다 우선하므로 사전 승인된 도메인을 차단하거나 프롬프트를 요구할 수 있습니다.

WebFetch는 `Claude-User`로 시작하는 `User-Agent` 헤더와 콘텐츠 협상을 지원하는 서버가 Markdown을 직접 반환할 수 있도록 HTML보다 Markdown을 선호하는 `Accept` 헤더를 설정합니다.

[샌드박스](/docs/en/sandboxing) 네트워크 규칙은 별도로 구성하므로 샌드박스 처리된 프로세스가 도달하기를 원하는 도메인에는 여전히 명시적인 샌드박스 권한 규칙이 필요합니다.

## WebSearch 도구 동작

WebSearch는 Anthropic의 [web search](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool) 백엔드에 대해 쿼리를 실행하고 결과 제목과 URL을 반환합니다. 결과 페이지를 가져오지는 않습니다. Claude가 검색 결과에서 찾은 페이지를 읽으려면 [WebFetch](#webfetch-tool-behavior)로 후속 조치를 취합니다.

도구는 결과를 반환하기 전에 내부적으로 검색을 다듬으면서 호출당 최대 8개의 백엔드 검색을 수행할 수 있습니다. Claude는 특정 호스트만 포함하도록 `allowed_domains`로 결과 범위를 지정하거나 제외하도록 `blocked_domains`로 지정할 수 있습니다. 두 목록은 단일 호출에서 결합할 수 없습니다.

{/* min-version: 2.1.212 */} 검색 요청이 과부하된 API에 도달하면 Claude Code는 지연 후 재시도하며, 계속 실패하는 호출은 오류 결과를 반환합니다. v2.1.212 이전에는 API 오류 텍스트가 검색 결과인 것처럼 Claude에 도달할 수 있었습니다.

WebSearch 권한 규칙은 지시자를 받지 않습니다. `allow` 또는 `deny`에 순수 `WebSearch` 항목을 두는 것이 유일한 형식입니다.

검색 백엔드는 구성할 수 없습니다. 다른 제공업체로 검색하려면 검색 도구를 노출하는 [MCP 서버](/docs/en/mcp)를 추가하세요.

<Note>
  WebSearch는 Claude API, [Claude Platform on AWS](/docs/en/claude-platform-on-aws), 및 Microsoft Foundry에서 사용할 수 있습니다. Google Cloud's Agent Platform에서는 Opus, Sonnet, Haiku를 포함한 Claude 4 이상의 모델에서 작동합니다. Amazon Bedrock은 서버 측 웹 검색 도구를 노출하지 않습니다.
</Note>

### 세션 검색 제한

세션은 메인 대화와 스폰하는 모든 [서브에이전트](/docs/en/sub-agents)에 걸쳐 집계하여 최대 200회의 WebSearch 호출을 수행할 수 있으므로 병렬 리서치 팬아웃에 의해 수행된 검색이 동일한 한도에 합산됩니다. 이 제한은 Claude Code v2.1.212 이상이 필요합니다. Claude가 한도에 도달하면 추가 호출은 재시도를 유도하는 오류 대신 이미 수집한 정보로 계속 진행하라는 알림을 Claude에게 반환합니다. 사용자는 알림을 보지 못합니다. 제한된 호출은 아무것도 하지 않은 검색으로 대화에 표시되며 Claude가 진정으로 더 많은 검색이 필요한 경우 알림은 Claude에게 한도를 늘려달라고 사용자에게 요청하도록 지시합니다.

한도를 변경하려면 [`CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION`](/docs/en/env-vars) 환경 변수를 설정하세요. 양의 정수를 받으므로 한도를 늘릴 수는 있지만 끄는 것은 불가능합니다. [`/clear`](/docs/en/commands#all-commands)를 실행하면 [세션 서브에이전트 제한](/docs/en/sub-agents)과 동일한 규칙에 따라 카운트가 재설정됩니다.

## Write 도구 동작

Write 도구는 새 파일을 생성하거나 제공된 전체 내용으로 기존 파일을 덮어씁니다. 덧붙이거나 병합하지 않습니다.

대상 경로가 이미 존재하는 경우 Claude는 덮어쓰기 전에 현재 대화에서 해당 파일을 최소 한 번 이상 읽었어야 합니다. 읽지 않은 기존 파일에 대한 Write는 오류와 함께 실패합니다. 이 제약 조건은 새 파일에는 적용되지 않습니다.

Bash로 파일을 보는 것도 [Edit tool behavior](#edit-tool-behavior)에 설명된 것과 동일한 규칙 하에서 이 요구 사항을 충족합니다.

기존 파일에 대한 부분 변경의 경우 Claude는 Write 대신 Edit를 사용합니다.

## 사용 가능한 도구 확인하기

정확한 도구 세트는 제공업체, 플랫폼 및 설정에 따라 다릅니다. 실행 중인 세션에 로드된 도구를 확인하려면 Claude에게 직접 물어보세요:

```text theme={null}
What tools do you have access to?
```

Claude가 대화형 요약을 제공합니다. 정확한 MCP 도구 이름의 경우 `/mcp`를 실행하세요.

<Note>
  [advisor 도구](/docs/en/advisor)는 Claude Code가 구현하는 도구가 아니라 API가 실행하는 [서버 도구](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool)입니다. 권한 규칙이나 훅 매처에서 참조할 수 있는 이름이 없습니다.
</Note>

## 참고 항목

* [MCP servers](/docs/en/mcp): 외부 서버를 연결하여 커스텀 도구 추가
* [Permissions](/docs/en/permissions): 권한 시스템, 규칙 구문 및 도구별 패턴
* [Subagents](/docs/en/sub-agents): 서브에이전트를 위한 도구 접근 구성
* [Hooks](/docs/en/hooks-guide): 도구 실행 전후에 커스텀 명령 실행
