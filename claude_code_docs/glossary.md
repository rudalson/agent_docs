> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 용어집 (Glossary)

> Claude Code 용어 정의. Agentic loop, compaction, CLAUDE.md, hooks, subagents, MCP 및 기타 핵심 개념의 의미를 알아봅니다.

본 용어집은 Claude Code 용어에 대한 정의를 제공합니다. 각 항목은 해당 개념을 심도 있게 다루는 문서 페이지로 연결됩니다. 토큰(token), 온독(temperature), RAG와 같은 모델 수준의 개념은 [플랫폼 용어집](https://platform.claude.com/docs/en/about-claude/glossary)을 참조하세요. 데스크톱 확장(desktop extension), MCPB, DXT 등 Claude Desktop 용어는 [Claude 고객센터](https://support.claude.com/)를 참조하세요.

## A

### Agent teams (에이전트 팀)

팀 리드(team lead)가 조율하는 여러 개의 독립적인 Claude Code 세션으로, 공유 태스크 목록과 피어 투 피어(peer-to-peer) 메시징을 활용합니다. 단일 세션 내에서 실행되고 부모 세션에만 보고하는 [subagents](#subagent)와 달리, 팀원은 각각 자체 컨텍스트 창(context window)을 가지며 어떤 팀원과도 직접 상호작용할 수 있습니다. Agent teams는 실험적 기능이며 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`을 설정하여 활성화해야 합니다.

자세히 알아보기: [Run agent teams](/docs/en/agent-teams)

### Agentic coding (에이전트 코딩)

직접 코드 변경 사항을 적용해야 하는 텍스트 전용 대화형 보조 도구와 달리, 사용자가 지켜보거나 지시를 변경하거나 자리를 비운 사이 AI가 파일 읽기, 명령 실행, 변경 사항 적용을 자율적으로 수행하는 워크플로입니다. Claude Code는 지시를 따르는 데 그치지 않고 작용할 수 있는 [tools](#tool)를 갖추고 있으므로 Agentic 능력을 보유하고 있습니다.

자세히 알아보기: [How Claude Code works](/docs/en/how-claude-code-works)

### Agentic harness (에이전트 하네스)

언어 모델을 역량 있는 코딩 에이전트로 전환해 주는 도구, 컨텍스트 관리 및 실행 환경입니다. Claude Code가 하네스(harness)이고, 그 안에서 구동되는 모델이 Claude입니다. 하네스는 파일 액세스, 셸 실행, 권한 제어, 메모리 로딩, 그리고 동작을 연쇄적으로 연결하는 루프를 제공합니다.

자세히 알아보기: [How Claude Code works](/docs/en/how-claude-code-works)

### Agentic loop (에이전트 루프)

Claude가 모든 태스크에 대해 수행하는 순환 주기입니다: 컨텍스트 수집, 작업 수행, 결과 검증, 완료될 때까지 반복. 각 도구 사용 결과는 다음 단계에 필요한 정보를 제공합니다. 언제든지 루프를 중단하여 새로운 지시를 내릴 수 있습니다. [hooks](#hook), [skills](#skill), [MCP](#mcp-model-context-protocol)를 포함한 대부분의 확장 포인트는 이 루프의 특정 단계에 연결됩니다.

자세히 알아보기: [How Claude Code works](/docs/en/how-claude-code-works#the-agentic-loop)

### Artifact (아티팩트)

Claude Code가 터미널 텍스트를 읽는 대신 결과를 시각적으로 보거나 공유할 수 있도록, 세션에서 private URL(claude.ai)로 게시하는 라이브 인터랙티브 웹 페이지입니다. 세션이 다시 게시되면 페이지가 그 자리에서 업데이트됩니다. Claude Code에서 생성한 아티팩트는 claude.ai 대화에서 생성된 아티팩트와 동일한 갤러리에 표시됩니다. 공유 권한은 플랜에 따라 다릅니다: Pro 및 Max 플랜에서는 누구나 열 수 있는 공개 링크 제공, Team 및 Enterprise 플랜에서는 조직 내부 공유 기본 적용 (소유자가 활성화하면 공개 링크 제공).

자세히 알아보기: [Share session output as artifacts](/docs/en/artifacts)

### Auto memory (자동 메모리)

사용자의 수정 사항 및 선호도를 바탕으로 Claude가 스스로 작성하는 메모로, git 저장소별로 `~/.claude/projects/`에 저장됩니다. 동일한 저장소의 모든 작업 트리는 하나의 자동 메모리 디렉토리를 공유합니다. 매 세션 시작 시 `MEMORY.md` 인덱스의 처음 200줄 또는 25KB가 로드됩니다. 자동 메모리는 사용자가 직접 작성하는 [CLAUDE.md](#claude-md)와 대비되는, Claude가 작성하는 상대 개념입니다.

자세히 알아보기: [Auto memory](/docs/en/memory#auto-memory)

### Auto mode (자동 모드)

별도의 분류(classifier) 모델이 백그라운드에서 작업 행동을 검토하여 승인 프롬프트 없이 대부분 실행되도록 하는 [permission mode](#permission-mode)입니다. 명시적인 확인(ask) 규칙은 여전히 프롬프트를 띄웁니다. 분류 모델은 범위 확장, 신뢰할 수 없는 인프라 접근 및 [prompt injection](#prompt-injection)을 차단합니다. 분류 모델은 도구 실행 결과를 보지 않으므로 주입된 지시사항이 승인 결정에 영향을 미칠 수 없습니다.

자세히 알아보기: [Eliminate prompts with auto mode](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)

## B

### Bare mode (베어 모드)

hooks, skills, plugins, MCP 서버, auto memory 및 CLAUDE.md의 자동 탐색을 건너뛰는 시작 플래그 `--bare`입니다. 명시적으로 전달한 플래그만 적용됩니다. 로컬 구성과 관계없이 머신 간에 동일한 동작을 보장해야 하는 CI 및 스크립트 실행 환경에서 권장됩니다.

자세히 알아보기: [Start faster with bare mode](/docs/en/headless#start-faster-with-bare-mode)

### Bundled skills (번들 스킬)

`/batch`, `/code-review`, `/debug`, `/loop`와 같이 Claude Code에 기본 포함된 프롬프트 기반 플레이북입니다. 고정된 로직을 실행하는 내장 명령과 달리, 번들 스킬은 Claude에게 상세한 프롬프트를 제공하여 작업을 오케스트레이션하도록 하므로 에이전트를 생성하고 파일을 읽으며 코드베이스에 적응할 수 있습니다.

자세히 알아보기: [Bundled skills](/docs/en/skills#bundled-skills)

## C

### Channel (채널)

터미널을 비운 동안에도 일어나는 이벤트에 Claude가 반응할 수 있도록, 실행 중인 세션에 이벤트를 푸시해 주는 [MCP server](#mcp-model-context-protocol)입니다. 채널은 양방향일 수 있습니다: Claude가 인바운드 이벤트를 읽고 동일한 채널을 통해 응답을 보냅니다. 리서치 프리뷰에는 Telegram, Discord, iMessage가 포함되어 있습니다.

자세히 알아보기: [Channels](/docs/en/channels)

### Checkpoint (체크포인트)

사용자가 프롬프트를 전송할 때마다 생성되는 복원 지점입니다. Claude Code는 체크포인트에서 복원할 수 있도록 변경 전 파일을 스냅샷으로 저장합니다. `Esc` 키를 두 번 누르거나 `/rewind`를 실행하여 코드, 대화 또는 둘 다를 이전 시점으로 복원하거나 선택한 메시지 이후의 대화를 요약할 수 있습니다. 체크포인트는 대화와 함께 저장되므로 재개된 세션에서도 여전히 `/rewind`할 수 있습니다. 체크포인트는 git과 별개이며 Bash 도구를 통해 수행된 변경 사항은 추적하지 않습니다.

자세히 알아보기: [Checkpointing](/docs/en/checkpointing)

### `.claude` 디렉토리

Claude Code가 설정, hooks, skills, subagents, rules, auto memory 등 프로젝트 범위의 구성을 읽어오는 디렉토리입니다. 프로젝트는 루트에 `.claude/`를 갖고, 사용자 수준 기본 설정은 `~/.claude/`에 위치합니다.

자세히 알아보기: [The `.claude` directory](/docs/en/claude-directory)

### CLAUDE.md

사용자가 Claude를 위해 작성하는 영구 지침 마크다운 파일로, 매 세션 시작 시 시스템 프롬프트 다음의 사용자 메시지로 로드됩니다. 프로젝트 관례, 아키텍처 참고 사항, "항상 X를 수행할 것"과 같은 규칙을 여기에 작성합니다. 프로젝트 루트의 CLAUDE.md는 [compaction](#compaction) 후에도 유지되며 디스크에서 새롭게 다시 읽어옵니다.

CLAUDE.md는 프로젝트 범위인 `./CLAUDE.md` 또는 `./.claude/CLAUDE.md`, 사용자 범위인 `~/.claude/CLAUDE.md`, 또는 조직을 위한 [managed policy](#managed-settings) 형태로 위치시킬 수 있습니다. 탐색된 모든 파일은 서로 덮어쓰지 않고 가장 넓은 범위부터 가장 구체적인 범위 순으로 컨텍스트에 연결(concatenation)됩니다.

자세히 알아보기: [CLAUDE.md files](/docs/en/memory#claude-md-files)

### Command (명령)

프롬프트에 `/name`을 입력하여 호출하는 재사용 가능한 지침입니다. `/clear`, `/model`, `/compact`와 같은 내장 명령은 세션을 제어합니다. `.claude/commands/` 디렉토리에 파일 형태로 자체 명령을 정의하거나 [plugin](#plugin)에서 설치할 수 있습니다. 다단계 명령을 패키징할 때는 [Skills](#skill)를 사용하는 것이 권장됩니다.

단어의 다른 두 가지 용도는 무관합니다: `cli-reference`에 나열된 `claude mcp add`와 같은 `claude` CLI 서브키워드, 그리고 스탠다드 I/O(stdio) [MCP server](#mcp-server) 항목의 `command` 필드(Claude Code가 서버를 시작하기 위해 실행하는 실행 파일 지정)입니다.

자세히 알아보기: [Commands](/docs/en/commands) · [Skills](/docs/en/skills)

### Compaction (압축)

[context window](#context-window)가 한계에 다다랐을 때 대화 내역을 자동으로 요약하는 기능입니다. 오래된 도구 출력 결과가 먼저 삭제된 후 대화가 요약됩니다. 프로젝트 루트의 CLAUDE.md 및 auto memory는 압축 후에도 유지되며 디스크에서 다시 로드됩니다. 대화 중에만 전달된 지침은 유실될 수 있습니다. `/compact`를 실행하여 수동으로 트리거할 수 있으며, `/compact focus on the API changes`와 같이 특정 부분에 집중하도록 요청할 수 있습니다.

자세히 알아보기: [What survives compaction](/docs/en/context-window#what-survives-compaction) · [When context fills up](/docs/en/how-claude-code-works#when-context-fills-up)

### Connector (커넥터)

Claude Code 내에서 직접 설정하지 않고 claude.ai 계정에 추가된 [MCP server](#mcp-server)입니다. 해당 계정으로 Claude Code에 로그인하면 로컬에서 추가한 서버와 함께 `/mcp`에 커넥터가 표시됩니다. 조직은 커넥터를 제공하고 도구별 제어를 설정할 수도 있습니다.

자세히 알아보기: [Use MCP servers from claude.ai](/docs/en/mcp#use-mcp-servers-from-claude-ai)

### Context window (컨텍스트 창)

대화 내역, 파일 내용, 명령 출력 결과, CLAUDE.md, auto memory, 로드된 스킬 및 시스템 지침을 보유하는 세션의 작업 메모리입니다. 작업을 진행함에 따라 컨텍스트가 차오르다가 [compaction](#compaction)을 통해 요약됩니다. `/context`를 실행하여 어떤 항목이 공간을 차지하고 있는지 확인할 수 있습니다. 기본 모델 수준의 개념은 [플랫폼 용어집](https://platform.claude.com/docs/en/about-claude/glossary#context-window)을 참조하세요.

자세히 알아보기: [Explore the context window](/docs/en/context-window)

## D

### Dispatch (디스패치)

Claude 모바일 앱에서 코딩 작업 요청을 보낼 때 Desktop 앱에서 Claude Code 세션을 생성해 주는 모바일 시작 작업 라우터입니다. 프롬프트가 적절한 도구로 자동 라우팅됩니다. Pro 및 Max 플랜에서 이용 가능합니다.

자세히 알아보기: [Sessions from Dispatch](/docs/en/desktop#sessions-from-dispatch)

## E

### Effort level (Effort 수준 / 노력 수준)

각 턴마다 Claude가 사용하는 적응형 추론(adaptive-reasoning) 사고 예산(thinking budget)의 양을 제어하는 설정입니다. Effort 수준이 높을수록 사고 토큰이 많아지고 추론이 더 깊어집니다. 낮을수록 빠르고 저렴합니다. Effort 수준은 Fable 5, Opus 4.6 이상, Sonnet 4.6 이상에서 지원됩니다.

자세히 알아보기: [Adjust effort level](/docs/en/model-config#adjust-effort-level)

### Extended thinking (확장 사고)

응답하기 전 모델이 수행하는 눈에 보이는 단계별 추론 과정입니다. [effort level](#effort-level)로 조절하거나 고정 예산을 가진 모델에서 `MAX_THINKING_TOKENS`로 사고 토큰 수를 제한할 수 있습니다. 사고 과정은 터미널에 회색 이탤릭체 텍스트로 표시됩니다.

자세히 알아보기: [Use extended thinking](/docs/en/model-config#extended-thinking)

## H

### Hook (훅)

도구 실행 전, 파일 편집 후, 세션 시작 시 등 Claude Code 라이프사이클의 특정 지점에서 자동으로 실행되는 사용자 정의 핸들러입니다. 핸들러는 셸 명령, HTTP 엔드포인트, MCP 도구, LLM 프롬프트 또는 subagent일 수 있습니다. Hooks는 결정론적입니다: 모델의 재량에 의존하지 않고 고정된 라이프사이클 지점에서 실행됩니다.

Hook 구성은 세 단계로 이루어집니다:

* **Hook event**: 라이프사이클 지점
* **Matcher**: 이벤트 실행 여부를 필터링
* **Hook handler**: 실행될 내용

자세히 알아보기: [Get started with hooks](/docs/en/hooks-guide) · [Hooks reference](/docs/en/hooks)

## M

### Managed settings (관리형 설정)

IT 또는 DevOps 부서에서 조직 전체에 강제 적용하는 설정으로, 관리자 콘솔을 통해 Anthropic 서버에서 전달되거나 `~/.claude` 외부의 OS 수준 경로에 장치별로 배포됩니다. 사용자 및 프로젝트 설정은 managed settings를 재정의(override)할 수 없습니다. 서버 관리형 전달은 [지원 대상 구성](/docs/en/server-managed-settings#platform-availability)에 적용됩니다. [보안 고려 사항](/docs/en/server-managed-settings#security-considerations)을 참조하세요. 보안 정책, 컴플라이언스 요건 또는 여러 기기 전체에 걸친 도구 표준화에 사용됩니다.

자세히 알아보기: [Server-managed settings](/docs/en/server-managed-settings) · [Settings files](/docs/en/settings#settings-files)

### MCP (Model Context Protocol)

AI 도구를 외부 데이터 소스 및 서비스에 연결하기 위한 오픈 표준입니다. MCP 서버는 Claude에게 Slack, Jira, 데이터베이스, 브라우저 및 수백 개의 기타 통합을 위한 새로운 도구를 제공합니다. `/mcp`를 통해 연결하거나 `.mcp.json`에 추가하여 서버를 연결합니다. 프로토콜 자체에 대해서는 [플랫폼 용어집](https://platform.claude.com/docs/en/about-claude/glossary#mcp-model-context-protocol)을 참조하세요.

자세히 알아보기: [Model Context Protocol](/docs/en/mcp)

### MCP server (MCP 서버)

[MCP](#mcp-model-context-protocol)를 통해 Claude에게 도구, 프롬프트, 리소스를 제공하는 프로그램입니다. `claude mcp add`를 실행하거나, `.mcp.json`에 추가하거나, [plugin](#plugin)을 통해, 또는 claude.ai [connector](#connector)로서 서버를 추가합니다. 로컬 stdio 서버는 설정의 `command` 및 `args` 필드에서 지정한 대로 Claude Code가 시작하는 프로세스로 구동되며, 이는 프롬프트에 입력하는 [commands](#command)와는 무관합니다.

자세히 알아보기: [Model Context Protocol](/docs/en/mcp)

### MCP Tool Search (MCP 도구 검색)

필요할 때까지 MCP 도구 스키마 로딩을 지연시키는 컨텍스트 절약 메커니즘입니다. 시작 시에는 도구 이름만 로드되며, Claude가 특정 도구를 사용하기로 결정할 때 요청에 따라 전체 스키마를 가져옵니다. 이를 통해 유휴 상태인 MCP 서버가 컨텍스트를 과도하게 차지하는 것을 방지합니다.

자세히 알아보기: [Scale with MCP Tool Search](/docs/en/mcp#scale-with-mcp-tool-search)

## N

### Non-interactive mode (비대화형 모드)

대화형 프롬프트 없이 단일 프롬프트를 실행하고 종료하는 모드로, `-p` 또는 `--print`로 호출합니다. CI, 스크립트 및 파이프라인 연결에 사용됩니다. `--no-session-persistence`를 전달하지 않는 한 실행 내역은 재개 가능한 세션으로 저장됩니다. Python 및 TypeScript에서의 동등한 형태는 [Agent SDK](/docs/en/agent-sdk/overview)입니다. 이전 명칭은 headless mode였습니다.

자세히 알아보기: [Run Claude Code programmatically](/docs/en/headless)

## O

### Output style (출력 스타일)

응답 동작, 어조 또는 형식을 변경하기 위해 Claude의 시스템 프롬프트를 수정하는 설정입니다. 시스템 프롬프트 다음에 사용자 메시지로 전달되는 [CLAUDE.md](#claude-md)와 달리, output styles는 기본 시스템 프롬프트 중 소프트웨어 공학 관련 항목을 끕니다. 내장 스타일로는 Default, Proactive, Explanatory, Learning이 있습니다.

자세히 알아보기: [Output styles](/docs/en/output-styles)

## P

### Permission mode (권한 모드)

세션의 기본 승인 동작 방식을 의미합니다. CLI에서는 `Shift+Tab`으로 순환하거나 VS Code, Desktop, claude.ai의 모드 선택기를 사용합니다. 사용 가능한 모드는 `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`입니다.

`default` 모드는 CLI, VS Code/JetBrains 확장, 데스크톱 앱에서 Manual로 표시되며, Claude Code는 `manual`을 해당 값의 별칭으로 허용합니다.

자세히 알아보기: [Choose a permission mode](/docs/en/permission-modes)

### Permission rule (권한 규칙)

도구 이름 및 인수 패턴을 기반으로 도구 호출을 허용(allow), 확인(ask), 거부(deny)하는 설정 항목입니다. 규칙은 deny → ask → allow 순으로 평가되며, 먼저 일치하는 규칙이 적용됩니다. 권한 규칙은 넓은 범위의 [permission mode](#permission-mode) 위에 레이어링된 세밀한 제어 수단입니다.

자세히 알아보기: [Configure permissions](/docs/en/permissions)

### Plan mode (플랜 모드)

Claude가 소스 파일을 직접 수정하지 않고 조사를 수행하여 변경 사항을 제안하는 [permission mode](#permission-mode)입니다. 파일 읽기, 검색, 탐색 명령을 실행한 후 코드를 수정하기 전에 승인을 받기 위한 계획(plan)을 제시합니다. `/plan`을 입력하거나 `Shift+Tab`을 눌러 plan mode로 진입합니다.

자세히 알아보기: [Analyze before you edit with plan mode](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)

### Plugin (플러그인)

단일 설치 가능 단위로 패키징된 skills, hooks, subagents 및 MCP 서버의 묶음입니다. 플러그인의 skills는 `plugin-name:skill-name`과 같이 네임스페이스가 지정되므로 여러 플러그인이 함께 존재할 수 있습니다. [marketplace](/docs/en/plugin-marketplaces)를 통해 팀 간에 플러그인을 공유 배포하세요.

자세히 알아보기: [Plugins](/docs/en/plugins)

### Project trust (프로젝트 신뢰)

Claude Code가 구성을 로드하기 전에 해당 디렉토리를 승인하는 대화 상자입니다. 승인 여부는 홈 디렉토리를 제외한 프로젝트 디렉토리별로 저장됩니다. 홈 디렉토리의 경우 현재 세션에 대해서만 신뢰가 유지되며 실행 시마다 프롬프트가 다시 표시됩니다. Project trust는 마켓플레이스 플러그인의 자동 설치 및 프로젝트 정의 hooks의 실행을 제어합니다. 디렉토리를 신뢰하면 해당 디렉토리의 `.claude/settings.json`, `.mcp.json` 및 기타 설정 파일이 적용됩니다.

자세히 알아보기: [The `.claude` directory](/docs/en/claude-directory)

### Prompt injection (프롬프트 주입)

파일, 웹 페이지 또는 도구 결과에 포함된 악의적인 지침으로, 사용자가 요청하지 않은 동작을 수행하도록 Claude를 유인하려는 시도입니다. Claude Code의 방어 요소에는 권한 시스템, 명령 주입 감지, 신뢰 검증이 포함됩니다. [Auto mode](#auto-mode)에는 도구 결과에서 의심스러운 콘텐츠를 검색하는 서버 측 프로브와 도구 결과를 보지 않는 분류 모델이 추가되어, 주입된 텍스트가 승인 결정에 영향을 미치지 못하도록 방지합니다.

자세히 알아보기: [Protect against prompt injection](/docs/en/security#protect-against-prompt-injection)

## R

### Remote Control (원격 제어)

claude.ai를 통해 휴대폰이나 브라우저에서 로컬 Claude Code 세션을 계속 이어하는 방식입니다. 코드 실행과 파일은 사용자의 머신에 남아 있으며 인터페이스만 원격으로 제공됩니다. 클라우드 샌드박스에서 구동되는 Claude Code on the web과는 다릅니다.

자세히 알아보기: [Remote Control](/docs/en/remote-control)

### Rules (규칙)

CLAUDE.md와 함께 로드되는 `.claude/rules/` 내의 모듈식 지침 파일입니다. 규칙은 YAML `paths:` 프론트매터(frontmatter)를 통해 경로 범위를 지정할 수 있으며, Claude가 일치하는 파일을 읽을 때만 로드되어 관련된 순간까지 컨텍스트를 간결하게 유지합니다.

자세히 알아보기: [Organize rules with `.claude/rules/`](/docs/en/memory#organize-rules-with-claude/rules/)

## S

### Sandboxing (샌드박싱)

Bash 도구에 대한 OS 수준의 파일 시스템 및 네트워크 격리 기능입니다. 명령은 미리 정의한 경계 내에서 실행되므로 명령어마다 승인 프롬프트를 띄우지 않고도 경계 내에서 Claude가 자유롭게 작업할 수 있습니다. 샌드박싱은 [permission rules](#permission-rule)와 별개의 레이어입니다.

자세히 알아보기: [Sandboxing](/docs/en/sandboxing)

### Session (세션)

독립적인 자체 [context window](#context-window)를 가지며 현재 디렉토리에 연결된 대화입니다. 세션은 `claude -c`로 재개할 수 있고, `--fork-session`으로 포크하여 새로운 세션 ID 하에 기록을 보존할 수 있으며, 여러 터미널에 걸쳐 병렬로 실행할 수 있습니다. `/clear`를 실행하면 새 세션이 시작되고, 이전 세션은 저장되어 `/resume`을 통해 접근 가능합니다. 각 세션의 트랜스크립트는 `~/.claude/projects/`에 저장됩니다.

자세히 알아보기: [Work with sessions](/docs/en/how-claude-code-works#work-with-sessions)

### Settings layers (설정 레이어)

Claude Code가 구성을 읽어오는 계층 구조로, 우선순위는 가장 높은 것부터 가장 낮은 것 순입니다: [managed policy](#managed-settings), 커맨드라인 인수, `.claude/settings.local.json`의 로컬 설정, `.claude/settings.json`의 프로젝트 설정, `~/.claude/settings.json`의 사용자 설정. 배열(Array)은 레이어 간에 병합되고, 단일 값(Scalar)은 상위 레이어가 하위 레이어를 재정의합니다.

자세히 알아보기: [Settings files](/docs/en/settings#settings-files)

### Skill (스킬)

Claude가 자체 툴킷에 추가하는 지침, 지식 또는 워크플로가 포함된 `SKILL.md` 파일입니다. Claude는 관련이 있을 때 자동으로 스킬을 로드하거나, 사용자가 `/skill-name`으로 직접 호출합니다. Skills는 Agent Skills 오픈 표준을 따르며, Claude Code는 호출 제어 및 subagent 실행 기능으로 이를 확장합니다.

Skills는 사용자 지정 명령(custom commands)을 대체하는 권장 방식입니다. `.claude/commands/deploy.md`와 `.claude/skills/deploy/SKILL.md` 위치의 파일은 둘 다 `/deploy`를 생성하고 동일하게 동작하며, 기존 명령 파일도 계속 작동합니다.

자세히 알아보기: [Extend Claude with skills](/docs/en/skills)

### Subagent (서브에이전트)

커스텀 시스템 프롬프트, 특정 도구 액세스 권한 및 독립적인 권한을 가지고 자체 컨텍스트 창에서 실행되는 전문 AI 보조 도구입니다. 위임된 태스크를 처리한 뒤 메인 대화로 요약 결과를 반환합니다. 대규모 탐색 작업을 메인 컨텍스트 외부에서 유지하거나 병렬 조사를 수행할 때 Subagents를 사용하세요. 각 에이전트가 직접 대화 가능한 완전한 독립 세션인 [agent teams](#agent-teams)와는 다릅니다.

내장 서브에이전트에는 Explore, Plan, 범용(general-purpose) 서브에이전트가 포함되어 있습니다.

자세히 알아보기: [Create custom subagents](/docs/en/sub-agents)

### Surface (서페이스 / 인터페이스)

CLI, VS Code, JetBrains, Desktop 또는 claude.ai 등 Claude Code에 접근하는 모든 지점/환경입니다. 모든 서페이스는 동일한 엔진을 공유합니다. 머신에서 구동되는 세션은 로컬 CLAUDE.md, 설정, 스킬을 읽지만, [cloud sessions](/docs/en/claude-code-on-the-web#what’s-available-in-cloud-sessions)은 저장소의 새로 복제된 사본에서 시작하며 머신의 `~/.claude/`를 읽지 않습니다. Slack 및 Chrome 확장은 서페이스 자체가 아니라 서페이스에 연결되는 통합 기능입니다.

자세히 알아보기: [Platforms and integrations](/docs/en/platforms)

## T

### Teleport (텔레포트)

클라우드상의 Claude Code 세션을 로컬 터미널로 끌어오는 `/teleport` 명령입니다. Claude가 브랜치를 가져오고 대화 내역을 로드하여 웹 세션의 마지막 상태에서 재개합니다. 반대 방향은 `--cloud`로, 로컬 작업 요청을 웹에서 실행하도록 전송합니다.

자세히 알아보기: [From web to terminal](/docs/en/claude-code-on-the-web#from-web-to-terminal)

### Tool (도구)

파일 읽기, 코드 편집, 셸 명령 실행, 웹 검색, 서브에이전트 생성 등 Claude가 수행할 수 있는 작업입니다. Tools는 Claude Code에 Agentic 능력을 부여하는 핵심 요소입니다. 도구가 없다면 Claude는 텍스트로만 응답할 수 있습니다. 각 도구 사용 결과는 [agentic loop](#agentic-loop)에서 Claude의 다음 의사결정에 필요한 정보를 제공합니다.

자세히 알아보기: [Tools available to Claude](/docs/en/tools-reference)

### Turn (턴)

[session](#session) 내에서 Claude가 제공하는 하나의 완전한 응답 단위입니다. 턴은 사용자가 메시지를 전송할 때 시작하여 임의의 수의 [tool](#tool) 호출을 거쳐 Claude가 응답을 완료할 때 끝납니다. 각 턴이 끝날 때 [Stop hooks](#hook)가 실행됩니다. 세션은 여러 턴으로 구성되며, [agentic loop](#agentic-loop)는 한 턴 내부에서 일어나는 일을 기술합니다.

자세히 알아보기: [How Claude Code works](/docs/en/how-claude-code-works#the-agentic-loop)

## V

### Verification loop (검증 루프)

세션이 수행한 작업이 단순히 그럴듯한 수준을 넘어 실제로 완성되었는지 확인하는 방식입니다. 테스트 수트, 빌드 실행, 스크린샷 비교 등 Claude가 실행할 수 있는 검증 수단을 제공하면, Claude는 한 번 시도 후 멈추지 않고 검증이 통과할 때까지 반복 처리합니다. Verification loop는 [`/goal`](/docs/en/goal), 무인 실행(unattended runs) 및 [dynamic workflows](/docs/en/workflows)를 위한 필수 조건입니다. 검증 루프가 없다면 에이전트의 완료 여부를 결정하는 주체는 에이전트 자신밖에 없게 됩니다.

자세히 알아보기: [Give Claude a way to verify its work](/docs/en/best-practices#give-claude-a-way-to-verify-its-work)

## W

### Worktree isolation (작업 트리 격리)

Claude를 `.claude/worktrees/` 아래의 별도 git worktree에서 실행하는 격리 모드로, `-w` 플래그나 subagent 설정의 `isolation: worktree`로 활성화합니다. 변경 사항은 별도 디렉토리의 별도 브랜치에 유지되므로 병렬 에이전트가 서로의 파일을 덮어쓰지 않습니다.

자세히 알아보기: [Run parallel sessions with git worktrees](/docs/en/worktrees)

***

## 변경 및 이전된 용어 (Deprecated and renamed terms)

다음 용어들은 이전 문서, 블로그 게시물 및 커뮤니티 콘텐츠에 나타납니다. 사이트 검색 시에는 현재 사용되는 명칭을 사용하세요.

| 이전 용어       | 현재 명칭                                     | 참고 사항                           |
| --------------- | --------------------------------------------- | ----------------------------------- |
| Headless mode   | [Non-interactive mode](#non-interactive-mode) | 동일한 `-p` 플래그, 동일한 동작 방식|
| Custom commands | [Skills](#skill)                              | `.claude/commands/` 파일 계속 작동함|
| Slash commands  | Commands                                      | 제품 문구에서 "Slash" 표기 삭제됨   |
