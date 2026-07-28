> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 커스텀 서브에이전트 생성하기

> 특정 작업 워크플로와 개선된 컨텍스트 관리를 위해 Claude Code에서 특화된 AI 서브에이전트를 생성하고 사용하세요.

서브에이전트는 특정 유형의 작업을 처리하는 특화된 AI 어시스턴트입니다. 사이드 작업(side task)으로 인해 다시 참조하지 않을 검색 결과, 로그 또는 파일 내용으로 메인 대화가 가득 차게 될 때 서브에이전트를 사용하세요. 서브에이전트는 자체 컨텍스트에서 해당 작업을 수행하고 요약만 반환합니다. 동일한 지침으로 동일한 종류의 워커를 계속 생성하게 될 때 커스텀 서브에이전트를 정의하세요.

각 서브에이전트는 커스텀 시스템 프롬프트, 특정 도구 접근 권한 및 독립적인 권한을 가지고 자체 컨텍스트 창에서 실행됩니다. Claude가 서브에이전트의 설명과 일치하는 작업을 만나면 해당 서브에이전트에 위임하여 독립적으로 작업하고 결과를 반환합니다. 실행 중인 컨텍스트 절약 효과를 실제로 보려면 서브에이전트가 자체 별도 창에서 탐색을 처리하는 세션을 안내하는 [컨텍스트 창 시각화](/docs/en/context-window)를 참조하세요.

<Note>
  서브에이전트는 단일 세션 내에서 작동합니다. 여러 독립 세션을 병렬로 실행하고 한 곳에서 모니터링하려면 [background agents](/docs/en/agent-view)를 참조하세요. 서로 통신하는 세션의 경우 [agent teams](/docs/en/agent-teams)를 참조하세요.
</Note>

서브에이전트의 장점:

* 메인 대화에서 탐색 및 구현 과정을 제외하여 **컨텍스트 보존**
* 서브에이전트가 사용할 수 있는 도구를 제한하여 **제약 조건 강제 적용**
* 사용자 수준 서브에이전트를 통해 프로젝트 간 **구성 재사용**
* 특정 도메인을 위한 집중된 시스템 프롬프트로 **동작 특화**
* Haiku와 같이 더 빠르고 저렴한 모델로 작업을 라우팅하여 **비용 제어**

Claude는 각 서브에이전트의 설명을 사용하여 작업을 위임할 시점을 결정합니다. 서브에이전트를 생성할 때 Claude가 언제 사용할지 알 수 있도록 명확한 설명을 작성하세요.

Claude Code에는 Explore, Plan 및 범용(general-purpose)과 같은 여러 내장 서브에이전트가 포함되어 있습니다. 또한 특정 작업을 처리하기 위해 커스텀 서브에이전트를 생성할 수도 있습니다.

## 내장 서브에이전트 (Built-in subagents)

Claude Code에는 적절할 때 Claude가 자동으로 사용하는 내장 서브에이전트가 포함되어 있습니다. 각 내장 서브에이전트는 추가 도구 제한과 함께 상위 대화의 권한을 상속받습니다.

Explore 및 Plan은 탐색을 빠르고 저렴하게 유지하기 위해 사용자의 CLAUDE.md 파일과 상위 세션의 git status를 건너끕니다. 다른 모든 내장 및 [커스텀 서브에이전트](#configure-subagents)는 둘 다 로드합니다. 시작 시 로드되는 항목에 대한 전체 분석은 [시작 시 로드되는 항목](#what-loads-at-startup)을 참조하세요.

<Tabs>
  <Tab title="Explore">
    코드베이스 검색 및 분석에 최적화된 빠르고 읽기 전용인 에이전트입니다.

    * **Model**: 메인 대화에서 상속받으며, Claude API에서 Opus로 제한되므로 Explore가 세션을 위해 이미 선택한 모델보다 더 비싼 모델에서 실행되지 않습니다.
    * **Tools**: 읽기 전용 도구. Write 및 Edit는 거부됨.
    * **Purpose**: 파일 검색, 코드 검색, 코드베이스 탐색.

    {/* min-version: 2.1.198 */}v2.1.198부터 Explore는 항상 Haiku에서 실행되는 대신 메인 대화의 모델을 상속받습니다. Claude API에서 상속된 모델은 Opus로 상한선이 지정됩니다. 상위 티어의 메인 대화는 Explore를 Opus에서 실행하고, Sonnet 또는 Haiku의 메인 대화는 동일한 모델에서 Explore를 실행합니다. [Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 또는 Claude Platform on AWS](/docs/en/third-party-integrations)와 같은 다른 제공업체에서는 Explore가 메인 대화의 모델을 직접 상속받습니다.

    `Explore`라는 이름의 [사용자 또는 프로젝트 서브에이전트](#choose-the-subagent-scope)는 내장 기능을 재정의하고 자체 `model` 필드를 유지하므로, 탐색을 저비용 모델로 유지하려면 `model: haiku`로 정의하세요.

    Claude는 변경을 수행하지 않고 코드베이스를 검색하거나 이해해야 할 때 Explore에 위임합니다. 이를 통해 탐색 결과가 메인 대화 컨텍스트에 들어가지 않도록 합니다.

    Explore를 호출할 때 Claude는 철저함의 수준을 지정합니다: 대상 조회의 경우 **quick**, 균형 잡힌 탐색의 경우 **medium**, 포괄적인 분석의 경우 **very thorough**.
  </Tab>

  <Tab title="Plan">
    플랜을 제시하기 전에 컨텍스트를 수집하기 위해 [plan mode](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode) 동안 사용되는 리서치 에이전트입니다.

    * **Model**: 메인 대화에서 상속
    * **Tools**: 읽기 전용 도구. Write 및 Edit는 거부됨.
    * **Purpose**: 플래닝을 위한 코드베이스 리서치.

    plan 모드에 있고 Claude가 코드베이스를 이해해야 할 때, 탐색 출력이 별도의 컨텍스트 창에 남아있는 동안 메인 대화가 읽기 전용 상태를 유지하도록 리서치를 Plan 서브에이전트에 위임합니다.
  </Tab>

  <Tab title="General-purpose">
    탐색과 조치가 모두 필요한 복잡한 다단계 작업을 위한 유능한 에이전트입니다.

    * **Model**: 메인 대화에서 상속
    * **Tools**: 모든 도구
    * **Purpose**: 복잡한 리서치, 다단계 작업, 코드 수정.

    작업에 탐색과 수정이 모두 필요하거나, 결과를 해석하기 위한 복잡한 추론이 필요하거나, 여러 종속 단계가 필요할 때 Claude는 general-purpose에 위임합니다.
  </Tab>

  <Tab title="Other">
    Claude Code에는 특정 작업을 위한 추가 헬퍼 에이전트가 포함되어 있습니다. 이러한 에이전트는 일반적으로 자동으로 호출되므로 직접 사용할 필요가 없습니다.

    | 에이전트 | 모델 | Claude가 사용하는 시점 |
    | :---------------- | :----- | :------------------------------------------------------- |
    | statusline-setup | Sonnet | 상태 표시줄을 구성하기 위해 `/statusline`을 실행할 때 |
    | claude-code-guide | Haiku | Claude Code 기능에 대해 질문할 때 |
  </Tab>
</Tabs>

내장 서브에이전트는 대화형 세션에서 기본적으로 등록됩니다. 이를 제한하려면:

* 특정 내장 유형을 차단하려면 [특정 서브에이전트 비활성화](#disable-specific-subagents)에 표시된대로 `permissions.deny`에 추가하세요.
* Claude가 어떤 서브에이전트에도 위임하지 못하도록 하려면 [`permissions.deny`](/docs/en/permissions#tool-specific-permission-rules)를 사용하여 `Agent` 도구 자체를 거부하세요.
* {/* min-version: 2.1.198 */}내장된 `Explore` 및 `Plan` 서브에이전트만 제거하려면 [`CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1`](/docs/en/env-vars)을 설정하세요. Claude는 위임하지 않고 직접 파일을 읽고 탐색합니다. Claude Code v2.1.198 이상이 필요합니다.
* [비대화형 모드](/docs/en/headless) 및 [Agent SDK](/docs/en/agent-sdk/overview)에서는 [`CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS=1`](/docs/en/env-vars)을 설정하여 모든 내장 유형을 제거하고 자체 유형만 제공하세요.

이러한 내장 서브에이전트 외에도 커스텀 프롬프트, 도구 제한, 권한 모드, 훅 및 skill을 사용하여 직접 서브에이전트를 생성할 수 있습니다. 다음 섹션에서는 시작하는 방법과 서브에이전트를 커스텀하는 방법을 보여줍니다.

## 퀵스타트: 첫 번째 서브에이전트 생성하기

서브에이전트는 YAML 프론트매터가 있는 Markdown 파일입니다. 하나를 생성하려면 Claude에게 작성을 요청하거나 [직접 파일을 작성](#write-subagent-files)하세요.

{/* min-version: 2.1.198 */}v2.1.198부터 `/agents` 명령은 더 이상 대화형 생성 마법사를 열지 않습니다. 실행하면 Claude에게 요청하거나 `.claude/agents/`를 직접 편집하라는 알림이 출력됩니다. 서브에이전트 파일, 프론트매터 필드, 그리고 `.claude/agents/` 및 `~/.claude/agents/` 위치는 변경되지 않으며 터미널 마법사만 제거됩니다.

이 워크스루는 코드를 검토하고 개선 사항을 제안하는 사용자 수준 서브에이전트를 생성합니다.

<Steps>
  <Step title="Claude에게 서브에이전트 생성 요청">
    Claude Code에서 원하는 서브에이전트와 저장 위치를 설명합니다:

    ```text wrap theme={null}
    Create a personal code-improver subagent in ~/.claude/agents/ that scans
    files and suggests improvements for readability, performance, and best
    practices. It should explain each issue, show the current code, and
    provide an improved version. Make it read-only and have it use Sonnet.
    ```

    Claude는 `name`, `description`, `tools` 목록, `model`, 그리고 시스템 프롬프트를 사용하여 파일을 작성합니다.
  </Step>

  <Step title="파일 검토">
    `~/.claude/agents/code-improver.md`를 열고 프론트매터가 요청한 내용과 일치하는지 확인합니다. 결과는 다음과 같습니다:

    ```markdown theme={null}
    ---
    name: code-improver
    description: Scans files and suggests improvements for readability, performance, and best practices. Use after writing or modifying code.
    tools: Read, Grep, Glob
    model: sonnet
    ---

    You are a code improvement specialist. For each issue you find, explain
    the problem, show the current code, and provide an improved version.
    ```

    파일이 `~/.claude/agents/`에 존재하므로 이 서브에이전트는 머신의 모든 프로젝트에서 사용할 수 있습니다. 하나의 프로젝트로 범위를 좁히려면 해당 프로젝트의 `.claude/agents/` 디렉터리로 이동하세요. [서브에이전트 범위 선택](#choose-the-subagent-scope)에서 두 가지를 비교합니다.
  </Step>

  <Step title="사용해 보기">
    Claude에게 새 서브에이전트에 위임하도록 요청합니다:

    ```text wrap theme={null}
    Use the code-improver agent to suggest improvements in this project
    ```

    Claude는 새 서브에이전트에 위임하고, 서브에이전트는 코드베이스를 스캔하여 개선 제안을 반환합니다.

    Claude가 새 서브에이전트를 찾을 수 없는 경우 Claude Code를 재시작하고 다시 시도하세요. 이는 세션이 시작되기 전에 `~/.claude/agents/`가 존재하지 않았던 경우에만 발생하며, 실행 중인 세션은 새롭게 생성된 `agents` 디렉터리를 감지하지 못하기 때문입니다.
  </Step>
</Steps>

이제 머신의 모든 프로젝트에서 코드베이스를 분석하고 개선 사항을 제안하는 데 사용할 수 있는 서브에이전트가 준비되었습니다.

손으로 직접 서브에이전트 파일을 작성하거나, CLI 플래그를 통해 정의하거나, 플러그인을 통해 배포할 수도 있습니다. 다음 섹션에서는 모든 구성 옵션을 다룹니다.

<Note>
  Claude Code v2.1.197 이하에서는 `/agents`가 활성 서브에이전트를 나열하는 **Running** 탭과 이를 생성, 편집, 삭제할 수 있는 **Library** 탭이 있는 대화형 마법사를 엽니다. {/* max-version: 2.1.197 */}
</Note>

## 서브에이전트 구성하기

서브에이전트의 파일 위치에 따라 사용할 수 있는 대상이 결정되고, 프론트매터에 따라 수행할 수 있는 작업이 결정됩니다. 이 섹션에서는 서브에이전트 파일이 위치하는 장소와 지원하는 모든 필드를 다룹니다.

### 서브에이전트 범위 선택 (Choose the subagent scope)

범위에 따라 서로 다른 위치에 서브에이전트 파일을 저장하세요. 여러 서브에이전트가 동일한 이름을 공유하는 경우 Claude Code는 더 높은 우선순위 위치의 서브에이전트를 사용합니다.

| 위치 | 범위 | 우선순위 | 생성 방법 |
| :--------------------------- | :---------------------- | :---------- | :-------------------------------------------- |
| 관리 설정 | 조직 전체 | 1 (최상위) | [관리 설정](/docs/en/settings)을 통해 배포 |
| `--agents` CLI 플래그 | 현재 세션 | 2 | Claude Code를 실행할 때 JSON 전달 |
| `.claude/agents/` | 현재 프로젝트 | 3 | Claude에게 요청하거나 수동으로 파일 생성 |
| `~/.claude/agents/` | 모든 사용자 프로젝트 | 4 | Claude에게 요청하거나 수동으로 파일 생성 |
| 플러그인의 `agents/` 디렉터리 | 플러그인이 활성화된 곳 | 5 (최하위) | [플러그인](/docs/en/plugins)과 함께 설치됨 |

**프로젝트 서브에이전트**(`.claude/agents/`)는 코드베이스에 특화된 서브에이전트에 이상적입니다. 버전 제어에 커밋하여 팀이 협업하여 사용하고 개선할 수 있도록 하세요.

프로젝트 서브에이전트는 현재 작업 디렉터리에서 위로 올라가며 검색되므로 해당 위치와 리포지토리 루트 사이의 모든 `.claude/agents/`가 스캔됩니다. {/* min-version: 2.1.178 */}v2.1.178부터 이러한 중첩 디렉터리 중 두 개 이상이 동일한 `name`을 정의하는 경우 Claude Code는 작업 디렉터리에 가장 가까운 정의를 사용합니다.

`--add-dir`로 추가된 디렉터리도 스캔됩니다. 추가된 디렉터리 내부의 `.claude/agents/` 폴더는 프로젝트 서브에이전트와 함께 로드됩니다. 다른 어떤 구성 유형이 `--add-dir`에서 로드되는지는 [추가 디렉터리](/docs/en/permissions#additional-directories-grant-file-access-not-configuration)를 참조하세요. `--add-dir` 없이 프로젝트 간에 서브에이전트를 공유하려면 `~/.claude/agents/` 또는 [플러그인](/docs/en/plugins)을 사용하세요.

**사용자 서브에이전트**(`~/.claude/agents/`)는 모든 프로젝트에서 사용할 수 있는 개인 서브에이전트입니다.

Claude Code는 `.claude/agents/` 및 `~/.claude/agents/`를 재귀적으로 스캔하므로 `agents/review/` 또는 `agents/research/`와 같은 하위 폴더로 정의를 정리할 수 있습니다. 하위 디렉터리 경로는 서브에이전트 식별이나 호출 방식에 영향을 주지 않으며, 식별은 오직 `name` 프론트매터 필드에서만 옵니다.

전체 트리에서 `name` 값을 고유하게 유지하세요. 동일한 `.claude/agents/` 디렉터리(하위 폴더 포함) 아래의 두 파일이 동일한 이름을 선언하는 경우 Claude Code는 문서화된 우선순위가 아닌 파일 시스템 읽기 순서에 따라 하나만 로드합니다. 중첩된 프로젝트 디렉터리 전반에서는 위에 설명된 대로 작업 디렉터리에 가장 가까운 정의가 승리합니다. {/* min-version: 2.1.205 */} [`/doctor`](/docs/en/commands#all-commands) 설정 점검은 동일한 디렉터리에서 이름을 공유하는 파일을 보고하고 하나를 제외한 모든 파일의 이름을 변경하거나 제거할 것을 제안합니다. v2.1.205 이전에는 `/doctor`가 중복 항목을 나열하고 활성 정의를 보여주는 진단 화면을 열었습니다.

플러그인 `agents/` 디렉터리도 재귀적으로 스캔됩니다. 프로젝트 및 사용자 범위와 달리 플러그인의 `agents/` 디렉터리 내부 하위 폴더는 [범위 지정 식별자](#invoke-subagents-explicitly)의 일부가 됩니다. 플러그인 `my-plugin`의 `agents/review/security.md` 파일은 `my-plugin:review:security`로 등록됩니다.

**CLI 정의 서브에이전트**는 Claude Code를 실행할 때 JSON으로 전달됩니다. 해당 세션에만 존재하며 디스크에 저장되지 않으므로 빠른 테스트나 자동화 스크립트에 유용합니다. 단일 `--agents` 호출에서 여러 서브에이전트를 정의할 수 있습니다:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    claude --agents '{
      "code-reviewer": {
        "description": "Expert code reviewer. Use proactively after code changes.",
        "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
        "tools": ["Read", "Grep", "Glob", "Bash"],
        "model": "sonnet"
      },
      "debugger": {
        "description": "Debugging specialist for errors and test failures.",
        "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes."
      }
    }'
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    claude --agents @'
    {
      "code-reviewer": {
        "description": "Expert code reviewer. Use proactively after code changes.",
        "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
        "tools": ["Read", "Grep", "Glob", "Bash"],
        "model": "sonnet"
      },
      "debugger": {
        "description": "Debugging specialist for errors and test failures.",
        "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes."
      }
    }
    '@
    ```
  </Tab>
</Tabs>

`--agents` 플래그는 파일 기반 서브에이전트와 동일한 [프론트매터](#supported-frontmatter-fields) 필드(`description`, `prompt`, `tools`, `disallowedTools`, `model`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`, `skills`, `initialPrompt`, `memory`, `effort`, `background`, `isolation`, `color`)가 포함된 JSON을 받아들입니다. 시스템 프롬프트에는 파일 기반 서브에이전트의 markdown 본문과 동일한 `prompt`를 사용하세요.

**관리형 서브에이전트**는 조직 관리자가 배포합니다. 프로젝트 및 사용자 서브에이전트와 동일한 프론트매터 형식을 사용하여 [관리 설정 디렉터리](/docs/en/settings#settings-files) 내의 `.claude/agents/`에 markdown 파일을 배치하세요. 관리형 정의는 동일한 이름의 프로젝트 및 사용자 서브에이전트보다 우선합니다.

**플러그인 서브에이전트**는 설치한 [플러그인](/docs/en/plugins)에서 옵니다. 커스텀 서브에이전트와 함께 자동으로 로드되며 범위가 지정된 이름으로 @-mention 자동 완성 목록에 표시됩니다. 플러그인 서브에이전트 생성에 대한 자세한 내용은 [플러그인 구성 요소 참조 문서](/docs/en/plugins-reference#agents)를 보세요.

<Note>
  보안상의 이유로 플러그인 서브에이전트는 `hooks`, `mcpServers` 또는 `permissionMode` 프론트매터 필드를 지원하지 않습니다. 이 필드들은 플러그인에서 에이전트를 로드할 때 무시됩니다. 이 필드들이 필요한 경우 에이전트 파일을 `.claude/agents/` 또는 `~/.claude/agents/`로 복사하세요. `settings.json` 또는 `settings.local.json`의 [`permissions.allow`](/docs/en/settings#permission-settings)에 규칙을 추가할 수도 있지만, 이 규칙은 플러그인 서브에이전트뿐만 아니라 세션 전체에 적용됩니다.
</Note>

이러한 모든 범위의 서브에이전트 정의는 [에이전트 팀](/docs/en/agent-teams#use-subagent-definitions-for-teammates)에서도 사용할 수 있습니다. 팀원을 스폰할 때 서브에이전트 유형을 참조할 수 있으며 팀원은 해당 `tools` 및 `model`을 사용하고, 정의의 본문은 추가 지침으로 팀원의 시스템 프롬프트에 덧붙여집니다. 해당 경로에 적용되는 프론트매터 필드는 [agent teams](/docs/en/agent-teams#use-subagent-definitions-for-teammates)를 참조하세요.

### 서브에이전트 파일 작성하기

서브에이전트 파일은 구성을 위해 YAML 프론트매터를 사용하고 그 뒤에 Markdown으로 된 시스템 프롬프트를 작성합니다:

<Note>
  Claude Code는 `~/.claude/agents/` 및 `.claude/agents/`를 감시합니다. 디스크에서 서브에이전트 파일을 추가하거나 편집하거나 Claude에게 작성을 요청하면 Claude Code가 몇 초 내에 변경 사항을 감지하고 재시작 없이 다음 위임 시 업데이트된 정의를 사용합니다.

  다음 두 가지 경우에는 여전히 재시작이 필요합니다:

  * 감시기는 세션이 시작될 때 존재했던 디렉터리만 포함하므로 새로운 `agents` 디렉터리에 범위의 첫 번째 에이전트 파일을 생성한 후에는 재시작하여 로드해야 합니다.
  * `--disable-slash-commands`로 시작된 세션은 이러한 디렉터리를 전혀 감시하지 않습니다.
</Note>

```markdown theme={null}
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

프론트매터는 서브에이전트의 메타데이터와 구성을 정의합니다. 본문은 서브에이전트의 동작을 안내하는 시스템 프롬프트가 됩니다. 서브에이전트는 전체 Claude Code 시스템 프롬프트가 아니라 이 시스템 프롬프트와 작업 디렉터리와 같은 기본 환경 세부 정보만 받습니다.

[비대화형 모드](/docs/en/headless)에서 [`--append-subagent-system-prompt`](/docs/en/cli-reference#cli-flags) 플래그는 중첩된 서브에이전트를 포함한 모든 서브에이전트 시스템 프롬프트 끝에 제공한 텍스트를 덧붙입니다. Claude Code v2.1.205 이상이 필요합니다.

서브에이전트는 메인 대화의 현재 작업 디렉터리에서 시작됩니다. 서브에이전트 내에서 `cd` 명령은 Bash 또는 PowerShell 도구 호출 간에 유지되지 않으며 메인 대화의 작업 디렉터리에 영향을 주지 않습니다. 서브에이전트에 리포지토리의 격리된 사본을 제공하려면 [`isolation: worktree`](#supported-frontmatter-fields)를 설정하세요.

{/* min-version: 2.1.203 */} `isolation: worktree`가 설정된 서브에이전트는 워크트리 내부에서 Bash 및 PowerShell 명령을 실행합니다. 서브에이전트가 실행되는 동안 워크트리 디렉터리가 제거되는 등의 이유로 작업 디렉터리가 메인 체크아웃으로 해석되는 명령은 오류와 함께 실패합니다. v2.1.203 이전에는 이러한 명령이 메인 체크아웃에서 실행될 수 있었습니다.

{/* min-version: 2.1.210 */}이 작업 디렉터리 검사는 Claude Code를 실행한 디렉터리가 포함된 전체 리포지토리를 포함합니다. 세션이 자체 연결된 [워크트리](/docs/en/worktrees)에서 실행되는 경우 이 검사는 해당 워크트리가 연결된 메인 체크아웃도 포함합니다. v2.1.210 이전에는 검사가 실행 디렉터리 자체만 포함했습니다. 모노레포 하위 디렉터리에서 Claude Code를 실행했을 때 리포지토리 루트와 같이 동일한 리포지토리의 다른 곳으로 해석된 작업 디렉터리를 가진 명령은 실패하는 대신 거기서 실행되었습니다.

#### 지원되는 프론트매터 필드

YAML 프론트매터에는 다음 필드를 사용할 수 있습니다. `name`과 `description`만 필수 항목입니다.

| 필드 | 필수 여부 | 설명 |
| :---------------- | :------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name` | 예 | 소문자와 하이픈을 사용하는 고유 식별자. [Hooks](/docs/en/hooks#subagentstart)는 이 값을 `agent_type`으로 받습니다. 파일 이름과 일치할 필요는 없음 |
| `description` | 예 | Claude가 이 서브에이전트에 위임해야 하는 시점 |
| `tools` | 아니오 | 서브에이전트가 사용할 수 있는 [도구](#available-tools). 생략 시 모든 도구를 상속합니다. 목록의 어떤 항목도 도구로 해석되지 않으면 서브에이전트가 해당 항목의 이름을 나타내는 오류와 함께 시작되지 않습니다. Skill을 컨텍스트에 사전 로드하려면 여기에 `Skill`을 나열하지 말고 `skills` 필드를 사용하세요 |
| `disallowedTools` | 아니오 | 거부할 도구. 상속되거나 지정된 목록에서 제거됨 |
| `model` | 아니오 | 사용할 [모델](#choose-a-model): `sonnet`, `opus`, `haiku`, `fable`, 전체 모델 ID(예: `claude-opus-4-8`), 또는 `inherit`. 기본값은 `inherit` |
| `permissionMode` | 아니오 | [권한 모드](#permission-modes): `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`, 또는 {/* min-version: 2.1.200 */} `default`의 별칭으로서 `manual`. `manual` 별칭은 Claude Code v2.1.200 이상이 필요합니다. [플러그인 서브에이전트](#choose-the-subagent-scope)에서는 무시됨 |
| `maxTurns` | 아니오 | 서브에이전트가 중지되기 전 최대 에이전틱 턴 수 |
| `skills` | 아니오 | 시작 시 서브에이전트 컨텍스트에 사전 로드할 [Skills](/docs/en/skills). 설명뿐만 아니라 전체 skill 내용이 주입됩니다. 서브에이전트는 여전히 Skill 도구를 통해 나열되지 않은 프로젝트, 사용자 및 플러그인 skill을 호출할 수 있습니다 |
| `mcpServers` | 아니오 | 이 서브에이전트에서 사용할 수 있는 [MCP 서버](/docs/en/mcp). 각 항목은 이미 구성된 서버를 참조하는 서버 이름(예: `"slack"`)이거나 서버 이름을 키로 하고 전체 [MCP 서버 구성](/docs/en/mcp#installing-mcp-servers)을 값으로 하는 인라인 정의입니다. [플러그인 서브에이전트](#choose-the-subagent-scope)에서는 무시됨 |
| `hooks` | 아니오 | 이 서브에이전트로 범위가 지정된 [생명주기 훅](#define-hooks-for-subagents). [플러그인 서브에이전트](#choose-the-subagent-scope)에서는 무시됨 |
| `memory` | 아니오 | [지속성 메모리 범위](#enable-persistent-memory): `user`, `project`, 또는 `local`. 세션 간 학습 활성화 |
| `background` | 아니오 | Claude가 그 결과를 바로 필요로 하는 경우에도 항상 이 서브에이전트를 [백그라운드 작업](#run-subagents-in-foreground-or-background)으로 실행하려면 `true`로 설정. 설정되지 않은 경우 Claude가 선택하며, {/* min-version: 2.1.198 */}v2.1.198부터는 기본적으로 서브에이전트를 백그라운드에서 실행합니다 |
| `effort` | 아니오 | 이 서브에이전트가 활성화되었을 때의 Effort 수준. 세션 effort 수준을 재정의합니다. 기본값: 세션에서 상속. 옵션: `low`, `medium`, `high`, `xhigh`, `max`. 사용 가능한 수준은 모델에 따라 다릅니다 |
| `isolation` | 아니오 | 서브에이전트를 임시 [git 워크트리](/docs/en/worktrees)에서 실행하려면 `worktree`로 설정. 상위 세션의 `HEAD`가 아닌 [기본 브랜치](/docs/en/worktrees#choose-the-base-branch)에서 기본적으로 분기된 리포지토리의 격리된 사본을 제공합니다. 서브에이전트가 아무런 변경도 하지 않은 경우 워크트리가 자동으로 정리됩니다 |
| `color` | 아니오 | 작업 목록 및 트랜스크립트에서 서브에이전트의 표시 색상. `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`을 허용합니다 |
| `initialPrompt` | 아니오 | 이 에이전트가 메인 세션 에이전트로 실행될 때(`--agent` 또는 `agent` 설정을 통해) 첫 번째 사용자 턴으로 자동 제출됩니다. [Commands](/docs/en/commands) 및 [skills](/docs/en/skills)가 처리됩니다. 사용자가 제공한 프롬프트 앞에 덧붙여집니다 |

### 모델 선택하기

`model` 필드는 서브에이전트가 사용하는 [AI 모델](/docs/en/model-config)을 제어합니다:

* **Model alias (모델 별칭)**: 사용 가능한 별칭 중 하나 사용: `sonnet`, `opus`, `haiku`, 또는 `fable`
* **Full model ID (전체 모델 ID)**: `claude-opus-4-8` 또는 `claude-sonnet-5`와 같은 전체 모델 ID 사용. `--model` 플래그와 동일한 값 허용
* **inherit**: 메인 대화와 동일한 모델 사용
* **Omitted (생략)**: 기본값은 `inherit`이며 메인 대화와 동일한 모델 사용

Claude가 서브에이전트를 호출할 때 해당 특정 호출을 위한 `model` 매개변수를 전달할 수도 있습니다. Claude Code는 다음 순서로 서브에이전트의 모델을 결정합니다:

1. 모델 별칭 또는 모델 ID로 설정된 경우 [`CLAUDE_CODE_SUBAGENT_MODEL`](/docs/en/model-config#environment-variables) 환경 변수
2. 호출별 `model` 매개변수
3. 서브에이전트 정의의 `model` 프론트매터
4. 메인 대화의 모델

{/* min-version: 2.1.196 */}v2.1.196부터 `CLAUDE_CODE_SUBAGENT_MODEL`을 `inherit`으로 설정하는 것은 설정하지 않은 것과 동일합니다. 그 후 호출별 `model` 매개변수, 프론트매터 순으로 계속 확인합니다. 이전 버전에서는 `inherit`이 서브에이전트를 메인 대화의 모델로 강제하고 두 출처를 모두 무시했습니다.

Claude Code는 환경 변수, 호출별 매개변수 및 프론트매터 값을 조직의 [`availableModels`](/docs/en/model-config#restrict-model-selection) 허용 목록과 비교하여 검증합니다. 제외된 모델로 해석되는 값은 건너뛰고 대신 상속된 모델에서 서브에이전트를 실행합니다.

{/* min-version: 2.1.211 */}호출별 `model` 매개변수는 서브에이전트가 [재개되거나 후속 메시지를 받을 때](#resume-subagents)도 적용되므로 서브에이전트가 해당 모델을 유지합니다. v2.1.211 이전에는 재개 시 호출별 값이 삭제되고 서브에이전트가 정의의 `model` 필드로 돌아가거나, 필드가 없으면 메인 대화의 모델로 돌아갔습니다.

{/* min-version: 2.1.198 */}v2.1.198부터 서브에이전트는 메인 대화의 [확장 사고(extended thinking)](/docs/en/model-config#extended-thinking) 구성도 상속받습니다. 세션에서 사고(thinking)가 켜져 있으면 서브에이전트에서도 켜지고 꺼져 있으면 꺼진 상태를 유지합니다. 서브에이전트별 사고 설정은 없습니다. v2.1.198 이전에는 메인 대화 설정과 관계없이 서브에이전트가 확장 사고가 비활성화된 상태로 실행되었습니다.

### 서브에이전트 기능 제어

도구 접근, 권한 모드 및 조건부 규칙을 통해 서브에이전트가 수행할 수 있는 작업을 제어할 수 있습니다.

#### 사용 가능한 도구 (Available tools)

서브에이전트는 기본적으로 메인 대화에서 사용할 수 있는 [내부 도구](/docs/en/tools-reference) 및 MCP 도구를 상속받습니다. 다음 도구들은 메인 대화의 UI 또는 세션 상태에 의존하므로 `tools` 필드에 나열되어 있더라도 서브에이전트에서 사용할 수 없습니다:

* `AskUserQuestion`
* `EndConversation` (메인 대화만 종료할 수 있음. [EndConversation tool behavior](/docs/en/tools-reference#endconversation-tool-behavior) 참조)
* `EnterPlanMode`
* `ExitPlanMode` (서브에이전트의 [`permissionMode`](#permission-modes)가 `plan`인 경우 제외)
* `ScheduleWakeup`
* `WaitForMcpServers`

`Agent` 도구 자체는 상속되므로 서브에이전트가 [중첩된 서브에이전트를 스폰](#spawn-nested-subagents)할 수 있습니다.

도구를 제한하려면 `tools` 필드를 허용 목록으로 사용하거나 `disallowedTools` 필드를 거부 목록으로 사용하세요. 다음 예시는 `tools`를 사용하여 Read, Grep, Glob, Bash만 허용합니다. 서브에이전트는 파일을 편집하거나 작성하거나 MCP 도구를 사용할 수 없습니다:

```yaml theme={null}
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
---
```

이 예시는 `disallowedTools`를 사용하여 Write 및 Edit를 제외한 메인 대화의 모든 도구를 상속받습니다. 서브에이전트는 Bash, MCP 도구 및 기타 모든 도구를 유지합니다:

```yaml theme={null}
---
name: no-writes
description: Inherits every tool except file writes
disallowedTools: Write, Edit
---
```

둘 다 설정된 경우 `disallowedTools`가 먼저 적용된 다음 남은 풀에서 `tools`가 해석됩니다. 둘 다에 나열된 도구는 제거됩니다.

`tools` 목록의 어떤 항목도 도구로 해석되지 않는 경우(예: 모든 항목의 철자가 틀렸거나 서브에이전트에 사용할 수 없는 도구 이름인 경우) Claude Code는 서브에이전트 실행을 거부하고 Agent 도구는 해석되지 않은 항목의 이름을 밝히는 오류를 반환합니다. {/* min-version: 2.1.208 */}v2.1.208 이전에는 해당 서브에이전트가 도구 없이 시작되어 빈 결과나 혼란스러운 결과를 반환할 수 있었습니다.

두 필드 모두 정확한 도구 이름 외에도 MCP 서버 수준 패턴(`mcp__<server>` 또는 `mcp__<server>__*`)을 수용하여 지정된 서버의 모든 도구를 부여하거나 제거합니다. `disallowedTools`에서 `mcp__*`는 모든 서버의 모든 MCP 도구도 제거합니다. 이 예시는 다른 서버의 도구와 모든 내장 도구를 유지하면서 `github` MCP 서버의 모든 도구를 제거합니다:

```yaml theme={null}
---
name: local-only
description: Inherits every tool except those from the github MCP server
disallowedTools: mcp__github
---
```

#### 스폰할 수 있는 서브에이전트 제한

에이전트가 `claude --agent`를 통해 메인 스레드로 실행될 때 Agent 도구를 사용하여 서브에이전트를 스폰할 수 있습니다. 스폰할 수 있는 서브에이전트 유형을 제한하려면 `tools` 필드에서 `Agent(agent_type)` 구문을 사용하세요.

<Note>버전 2.1.63에서 Task 도구의 이름이 Agent로 변경되었습니다. 설정 및 에이전트 정의에 있는 기존 `Task(...)` 참조는 여전히 별칭으로 작동합니다.</Note>

```yaml theme={null}
---
name: coordinator
description: Coordinates work across specialized agents
tools: Agent(worker, researcher), Read, Bash
---
```

이는 허용 목록입니다. `worker` 및 `researcher` 서브에이전트만 스폰할 수 있습니다. 에이전트가 다른 유형을 스폰하려고 하면 요청이 실패하고 에이전트의 프롬프트에는 허용된 유형만 표시됩니다. 다른 모든 에이전트를 허용하면서 특정 에이전트를 차단하려면 대신 [`permissions.deny`](#disable-specific-subagents)를 사용하세요.

제한 없이 모든 서브에이전트 스폰을 허용하려면 괄호 없이 `Agent`를 사용하세요:

```yaml theme={null}
tools: Agent, Read, Bash
```

`tools` 목록에서 `Agent`가 완전히 생략되면 에이전트는 어떤 서브에이전트도 스폰할 수 없습니다.

`Agent(agent_type)` 허용 목록 구문은 `claude --agent`를 사용하여 메인 스레드로 실행되는 에이전트에만 적용됩니다. 서브에이전트 정의에서 `tools`에 `Agent`를 나열하면 해당 서브에이전트가 [중첩된 서브에이전트를 스폰](#spawn-nested-subagents)할 수 있지만 괄호 안의 유형 목록은 무시됩니다.

#### MCP 서버 범위를 서브에이전트로 지정

`mcpServers` 필드를 사용하여 메인 대화에서 사용할 수 없는 [MCP](/docs/en/mcp) 서버에 대한 접근 권한을 서브에이전트에 부여하세요. 여기에 정의된 인라인 서버는 서브에이전트가 시작할 때 연결되고 완료되면 연결 해제됩니다. 문자열 참조는 상위 세션의 연결을 공유합니다.

<Note>
  `mcpServers` 필드는 에이전트 파일이 실행될 수 있는 두 가지 컨텍스트 모두에 적용됩니다:

  * Agent 도구 또는 @-mention을 통해 스폰되는 서브에이전트
  * [`--agent`](#invoke-subagents-explicitly) 또는 `agent` 설정을 통해 실행되는 메인 세션

  에이전트가 메인 세션인 경우 인라인 서버 정의는 [`.mcp.json`](/docs/en/mcp) 및 설정 파일의 서버와 함께 시작 시 연결됩니다.
</Note>

목록의 각 항목은 인라인 서버 정의이거나 세션에 이미 구성된 MCP 서버를 참조하는 문자열입니다:

```yaml theme={null}
---
name: browser-tester
description: Tests features in a real browser using Playwright
mcpServers:
  # Inline definition: scoped to this subagent only
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  # Reference by name: reuses an already-configured server
  - github
---

Use the Playwright tools to navigate, screenshot, and interact with pages.
```

인라인 정의는 서버 이름을 키로 사용하여 `.mcp.json` 서버 항목과 동일한 스키마를 사용하며 `stdio`, `http`, `sse`, `ws` 유형을 지원합니다.

MCP 서버를 메인 대화에서 완전히 제외하고 거기서 도구 설명이 컨텍스트를 소비하지 않도록 하려면 `.mcp.json` 대신 여기에 인라인으로 정의하세요. 서브에이전트는 도구를 얻지만 상위 대화는 얻지 못합니다.

v2.1.153부터 메인 세션에 적용되는 MCP 제한 사항은 서브에이전트 프론트매터에 선언된 서버에도 적용됩니다:

* [`--strict-mcp-config`](/docs/en/cli-reference) 및 [`--bare`](/docs/en/cli-reference)
* [Enterprise managed MCP configuration](/docs/en/managed-mcp)
* [`allowedMcpServers` and `deniedMcpServers` policies](/docs/en/managed-mcp#policy-based-control-with-allowlists-and-denylists)

이 중 하나가 서버를 차단하면 Claude Code는 이를 건너뛰고 차단된 서버 이름을 알리는 경고를 표시합니다.

관리 설정 제한 사항은 정의 방식과 관계없이 모든 서브에이전트에 적용됩니다. `--strict-mcp-config`는 명시적 호출자 입력이므로 `--agents` 또는 SDK `agents` 옵션을 통해 인라인으로 전달하는 서버를 필터링하지 않습니다.

#### 권한 모드 (Permission modes)

`permissionMode` 필드는 서브에이전트가 권한 프롬프트를 처리하는 방식을 제어합니다. 서브에이전트는 메인 대화에서 권한 컨텍스트를 상속받으며 아래 설명된 대로 상위 모드가 우선하는 경우를 제외하고 모드를 재정의할 수 있습니다.

| 모드 | 동작 |
| :------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `default` | 프롬프트가 포함된 표준 권한 검사 |
| `acceptEdits` | 작업 디렉터리 또는 `additionalDirectories`에 있는 경로에 대해 파일 편집 및 일반적인 파일 시스템 명령을 자동 승인 |
| `auto` | [Auto 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode): 백그라운드 분류기가 명령 및 보호된 디렉터리 쓰기를 검토 |
| `dontAsk` | 권한 프롬프트를 자동으로 거부. 명시적으로 허용된 도구는 계속 작동함. `AskUserQuestion`, [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 허용했더라도 거부됨 |
| `bypassPermissions` | 권한 프롬프트 건너뛰기 |
| `plan` | Plan 모드 (읽기 전용 탐색) |

<Warning>
  `bypassPermissions`는 주의해서 사용하세요. 권한 프롬프트를 건너뛰어 `.git`, `.config/git`, `.claude`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`에 대한 쓰기를 포함하여 승인 없이 서브에이전트가 작업을 실행할 수 있도록 허용합니다.

  명시적인 [`ask` 규칙](/docs/en/permissions#manage-permissions), [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구, 그리고 `rm -rf /`와 같은 루트 및 홈 디렉터리 삭제는 여전히 프롬프트를 표시합니다. 세부 정보는 [permission modes](/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode)를 참조하세요.
</Warning>

상위 항목이 `bypassPermissions` 또는 `acceptEdits`를 사용하는 경우 이것이 우선하며 재정의할 수 없습니다. 상위 항목이 [auto 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)를 사용하는 경우 서브에이전트는 auto 모드를 상속받으며 프론트매터의 `permissionMode`는 무시됩니다. 분류기는 상위 세션과 동일한 차단 및 허용 규칙으로 서브에이전트의 도구 호출을 평가합니다.

#### 서브에이전트에 skill 사전 로드

`skills` 필드를 사용하여 시작 시 서브에이전트 컨텍스트에 skill 내용을 주입하세요. 이는 실행 중에 서브에이전트가 skill을 검색하고 로드하도록 요구하지 않고 서브에이전트에 도메인 지식을 제공합니다.

```yaml theme={null}
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

나열된 각 skill의 전체 내용이 시작 시 서브에이전트 컨텍스트에 주입됩니다. 이 필드는 사전 로드되는 skill을 제어하며 서브에이전트가 접근할 수 있는 skill을 제어하지 않습니다. 이 필드가 없더라도 서브에이전트는 실행 중에 Skill 도구를 통해 프로젝트, 사용자 및 플러그인 skill을 계속 검색하고 호출할 수 있습니다. 서브에이전트가 skill을 호출하지 못하게 하려면 [`tools`](#available-tools) 목록에서 `Skill`을 생략하거나 `disallowedTools`에 추가하세요.

사전 로드는 Claude가 호출할 수 있는 동일한 skill 세트에서 선택되므로 [`disable-model-invocation: true`](/docs/en/skills#control-who-invokes-a-skill)를 설정한 skill은 사전 로드할 수 없습니다. 나열된 skill이 누락되었거나 비활성화된 경우 Claude Code는 이를 건너뛰고 디버그 로그에 경고를 기록합니다.

<Note>
  이는 [서브에이전트에서 skill 실행](/docs/en/skills#run-skills-in-a-subagent)의 반대 개념입니다. 서브에이전트의 `skills`를 사용하면 서브에이전트가 시스템 프롬프트를 제어하고 skill 내용을 로드합니다. skill의 `context: fork`를 사용하면 skill 내용이 지정한 에이전트에 주입됩니다. 둘 다 동일한 기본 시스템을 사용합니다.
</Note>

#### 지속성 메모리 활성화

`memory` 필드는 서브에이전트에 대화를 넘어 유지되는 지속적인 디렉터리를 제공합니다. 서브에이전트는 이 디렉터리를 사용하여 코드베이스 패턴, 디버깅 통찰력, 아키텍처 결정과 같은 지식을 시간에 따라 구축합니다.

```yaml theme={null}
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

메모리가 얼마나 광범위하게 적용되어야 하는지에 따라 범위를 선택하세요:

| 범위 | 위치 | 사용 시점 |
| :-------- | :-------------------------------------------- | :----------------------------------------------------------------------------------------- |
| `user` | `~/.claude/agent-memory/<name-of-agent>/` | 서브에이전트가 모든 프로젝트에 걸쳐 학습 내용을 기억해야 할 때 |
| `project` | `.claude/agent-memory/<name-of-agent>/` | 서브에이전트의 지식이 프로젝트 특화되어 있고 버전 제어를 통해 공유 가능할 때 |
| `local` | `.claude/agent-memory-local/<name-of-agent>/` | 서브에이전트의 지식이 프로젝트 특화되어 있지만 버전 제어에 커밋되지 않아야 할 때 |

메모리가 활성화된 경우:

* 서브에이전트의 시스템 프롬프트에는 메모리 디렉터리를 읽고 쓰기 위한 지침이 포함됩니다.
* 서브에이전트의 시스템 프롬프트에는 메모리 디렉터리의 `MEMORY.md`에서 처음 200줄 또는 25KB 중 먼저 도달하는 내용이 포함되며, 이를 초과할 경우 `MEMORY.md`를 기획(curate)하라는 지침이 포함됩니다.
* Read, Write 및 Edit 도구가 자동으로 활성화되어 서브에이전트가 메모리 파일을 관리할 수 있습니다.

##### 지속성 메모리 팁

* `project`가 권장 기본 범위입니다. 서브에이전트 지식을 버전 제어를 통해 공유할 수 있게 만듭니다.
* 작업을 시작하기 전에 서브에이전트에 메모리를 참조하도록 요청하세요: "Review this PR, and check your memory for patterns you've seen before."
* 작업을 완료한 후 서브에이전트에 메모리를 업데이트하도록 요청하세요: "Now that you're done, save what you learned to your memory." 시간이 지남에 따라 서브에이전트를 더 효과적으로 만드는 지식 기반이 구축됩니다.
* 서브에이전트의 markdown 파일에 메모리 지침을 직접 포함하여 지식 기반을 주도적으로 유지 관리하도록 하세요:

  ```markdown theme={null}
  Update your agent memory as you discover codepaths, patterns, library
  locations, and key architectural decisions. This builds up institutional
  knowledge across conversations. Write concise notes about what you found
  and where.
  ```

#### 훅을 이용한 조건부 규칙

도구 사용에 대한 보다 동적인 제어를 위해 `PreToolUse` 훅을 사용하여 작업이 실행되기 전에 검증하세요. 이는 도구의 일부 작업은 허용하고 다른 작업은 차단해야 할 때 유용합니다.

이 예시는 읽기 전용 데이터베이스 쿼리만 허용하는 서브에이전트를 생성합니다. `PreToolUse` 훅은 각 Bash 명령이 실행되기 전에 `command`에 지정된 스크립트를 실행합니다:

```yaml theme={null}
---
name: db-reader
description: Execute read-only database queries
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---
```

Claude Code는 stdin을 통해 [훅 입력을 JSON으로 전달](/docs/en/hooks#pretooluse-input)합니다. 검증 스크립트는 이 JSON을 읽고, Bash 명령을 추출하며, 쓰기 작업을 차단하기 위해 [종료 코드 2로 종료](/docs/en/hooks#exit-code-2-behavior-per-event)합니다:

```bash theme={null}
#!/bin/bash
# ./scripts/validate-readonly-query.sh

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Block SQL write operations (case-insensitive)
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE)\b' > /dev/null; then
  echo "Blocked: Only SELECT queries are allowed" >&2
  exit 2
fi

exit 0
```

전체 입력 스키마는 [Hook input](/docs/en/hooks#pretooluse-input)을, 종료 코드가 동작에 미치는 영향은 [exit codes](/docs/en/hooks#exit-code-output)를 참조하세요. Windows에서는 [running hooks in PowerShell](/docs/en/hooks#windows-powershell-tool)에 설명된 대로 PowerShell로 훅 스크립트를 작성하고 훅 항목에 `shell: powershell`을 추가하세요.

#### 특정 서브에이전트 비활성화

[설정](/docs/en/settings#permission-settings)의 `deny` 배열에 추가하여 Claude가 특정 서브에이전트를 사용하지 못하도록 방지할 수 있습니다. `Agent(subagent-name)` 형식을 사용하세요:

```json theme={null}
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(code-reviewer)"]
  }
}
```

이 규칙은 `Agent` 도구 호출이 수동 또는 자동으로 서브에이전트를 실행하는 것을 차단합니다. `--disallowedTools` CLI 플래그로도 동일하게 수행할 수 있습니다:

```bash theme={null}
claude --disallowedTools "Agent(Explore)"
```

권한 규칙에 대한 자세한 내용은 [Permissions 문서](/docs/en/permissions#tool-specific-permission-rules)를 참조하세요.

### 서브에이전트 훅 정의하기

서브에이전트는 서브에이전트의 생명주기 동안 실행되는 [훅](/docs/en/hooks)을 정의할 수 있습니다. 훅을 구성하는 방법에는 두 가지가 있습니다:

* **서브에이전트 프론트매터 내부**: 해당 서브에이전트가 활성화되어 있는 동안만 실행되는 훅 정의
* **`settings.json` 내부**: 서브에이전트가 시작되거나 중지될 때 메인 세션에서 실행되는 훅 정의

#### 서브에이전트 프론트매터의 훅

서브에이전트의 markdown 파일에 직접 훅을 정의하세요. 이러한 훅은 해당 특정 서브에이전트가 활성화되어 있는 동안에만 실행되며 완료되면 정리됩니다.

<Note>
  프론트매터 훅은 에이전트가 Agent 도구 또는 @-mention을 통해 서브에이전트로 스폰될 때, 그리고 [`--agent`](#invoke-subagents-explicitly) 또는 `agent` 설정을 통해 에이전트가 메인 세션으로 실행될 때 실행됩니다. 메인 세션인 경우 [`settings.json`](/docs/en/hooks)에 정의된 모든 훅과 함께 실행됩니다.
</Note>

모든 [훅 이벤트](/docs/en/hooks#hook-events)가 지원됩니다. 서브에이전트에 가장 일반적인 이벤트는 다음과 같습니다:

| 이벤트 | 매처 입력 | 실행 시점 |
| :------------ | :------------ | :------------------------------------------------------------------ |
| `PreToolUse` | 도구 이름 | 서브에이전트가 도구를 사용하기 전 |
| `PostToolUse` | 도구 이름 | 서브에이전트가 도구를 사용한 후 |
| `Stop` | (없음) | 서브에이전트가 완료될 때(런타임에 `SubagentStop`으로 변환됨) |

이 예시는 `PreToolUse` 훅으로 Bash 명령을 검증하고 `PostToolUse`를 사용하여 파일 편집 후 린터(linter)를 실행합니다:

```yaml theme={null}
---
name: code-reviewer
description: Review code changes with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh $TOOL_INPUT"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

에이전트가 서브에이전트로 호출될 때 프론트매터의 `Stop` 훅은 자동으로 `SubagentStop` 이벤트로 변환됩니다.

#### 서브에이전트 이벤트를 위한 프로젝트 수준 훅

메인 세션에서 서브에이전트 생명주기 이벤트에 반응하는 훅을 `settings.json`에 구성하세요.

| 이벤트 | 매처 입력 | 실행 시점 |
| :-------------- | :-------------- | :------------------------------- |
| `SubagentStart` | 에이전트 유형 이름 | 서브에이전트가 실행을 시작할 때 |
| `SubagentStop` | 에이전트 유형 이름 | 서브에이전트가 완료될 때 |

두 이벤트 모두 이름으로 특정 에이전트 유형을 타겟팅하는 매처를 지원합니다. 매처 값은 프로젝트 수준 및 사용자 수준 서브에이전트의 경우 에이전트의 프론트매터 `name`이고, [플러그인 서브에이전트](/docs/en/plugins)의 경우 `my-plugin:db-agent`와 같은 플러그인 범위 식별자입니다. 범위가 지정된 이름에는 콜론이 포함되어 있으므로 [unanchored regular expression](/docs/en/hooks#matcher-patterns)으로 평가됩니다. 해당 에이전트만 일치시키려면 `^my-plugin:db-agent$`와 같이 `^` 및 `$`로 앵커링하세요.

이 예시는 `db-agent` 서브에이전트가 시작할 때만 설정 스크립트를 실행하고, 임의의 서브에이전트가 중지될 때 정리 스크립트를 실행합니다:

```json theme={null}
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db-connection.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup-db-connection.sh" }
        ]
      }
    ]
  }
}
```

`db-agent`와 같이 하이픈이 들어간 매처는 Claude Code v2.1.195 이상에서 정확히 일치합니다. 이전 버전에서는 비앵커링 정규식으로 평가되어 `prod-db-agent`와 같이 해당 문자열을 포함하는 임의의 에이전트 유형에 대해서도 실행되므로, 해당 버전에서는 `^db-agent$`로 앵커링하세요.

전체 훅 구성 형식은 [Hooks](/docs/en/hooks)를 참조하세요.

## 서브에이전트 활용하기

### 자동 위임 이해하기

Claude는 요청의 작업 설명, 서브에이전트 구성의 `description` 필드, 그리고 현재 컨텍스트를 기반으로 작업을 자동으로 위임합니다. 능동적인 위임을 유도하려면 서브에이전트의 description 필드에 "use proactively"와 같은 문구를 포함하세요.

### 명시적으로 서브에이전트 호출하기

자동 위임으로 충분하지 않을 때 서브에이전트를 직접 요청할 수 있습니다. 일회성 제안부터 세션 전체 기본값까지 3가지 패턴으로 단계가 올라갑니다:

* **자연어**: 프롬프트에 서브에이전트 이름을 지정. Claude가 위임 여부 결정
* **@-mention**: 한 작업에 대해 서브에이전트가 실행됨을 보장
* **세션 전체**: `--agent` 플래그 또는 `agent` 설정을 통해 전체 세션이 해당 서브에이전트의 시스템 프롬프트, 도구 제한 및 모델을 사용

자연어의 경우 특별한 구문이 없습니다. 서브에이전트 이름을 지정하면 Claude가 일반적으로 위임합니다:

```text wrap theme={null}
Use the test-runner subagent to fix failing tests
Have the code-reviewer subagent look at my recent changes
```

**서브에이전트 @-mention.** 파일을 @-mention하는 것과 동일한 방식으로 `@`를 입력하고 자동 완성 목록에서 서브에이전트를 선택합니다. 이를 통해 Claude에게 선택을 맡기지 않고 특정 서브에이전트가 실행되도록 보장합니다:

```text wrap theme={null}
@"code-reviewer (agent)" look at the auth changes
```

전체 메시지는 여전히 Claude에게 전달되며, Claude는 요청한 내용을 기반으로 서브에이전트의 작업 프롬프트를 작성합니다. @-mention은 Claude가 받는 프롬프트가 아니라 호출하는 서브에이전트를 제어합니다.

활성화된 [플러그인](/docs/en/plugins)이 제공하는 서브에이전트는 범위 지정된 이름으로 자동 완성 목록에 표시되며, 플러그인이 [에이전트를 하위 폴더로 정리](#choose-the-subagent-scope)한 경우 `my-plugin:code-reviewer` 또는 `my-plugin:review:security`와 같이 표시됩니다. 세션에서 현재 실행 중인 이름이 있는 백그라운드 서브에이전트도 자동 완성 목록에 나타나며 이름 옆에 상태가 표시됩니다.

선택기를 사용하지 않고 수동으로 아노테이션을 입력할 수도 있습니다. 로컬 서브에이전트의 경우 `@agent-<name>`, 플러그인 서브에이전트의 경우 `@agent-` 뒤에 범위 지정 이름을 입력하세요(예: `@agent-my-plugin:code-reviewer`). 이 형식을 입력하는 동안 자동 완성 목록에는 에이전트 대신 파일 일치 항목이 표시됩니다. 제출하면 에이전트 멘션이 정상적으로 확인됩니다.

**전체 세션을 서브에이전트로 실행.** [`--agent <name>`](/docs/en/cli-reference)을 전달하여 메인 스레드 자체가 해당 서브에이전트의 시스템 프롬프트, 도구 제한 및 모델을 맡는 세션을 시작하세요:

```bash theme={null}
claude --agent code-reviewer
```

서브에이전트의 시스템 프롬프트는 [`--system-prompt`](/docs/en/cli-reference)와 동일한 방식으로 기본 Claude Code 시스템 프롬프트를 완전히 대체합니다. `CLAUDE.md` 파일과 프로젝트 메모리는 여전히 일반 메시지 흐름을 통해 로드됩니다. 활성화되었음을 확인할 수 있도록 시작 헤더에 에이전트 이름이 `@<name>`으로 표시됩니다.

이는 내장 및 커스텀 서브에이전트 모두에서 작동하며 세션을 재개할 때도 선택이 유지됩니다.

플러그인이 제공하는 서브에이전트의 경우 에이전트 이름만 전달해도 Claude Code가 이를 찾습니다:

```bash theme={null}
claude --agent security-reviewer
```

여러 플러그인이 동일한 이름의 에이전트를 제공하는 경우 범위 지정 이름을 전달하여 모호함을 해소하세요:

```bash theme={null}
claude --agent my-plugin:security-reviewer
```

플러그인이 `agents/` 디렉터리의 하위 폴더에 에이전트를 배치한 경우 범위 지정 이름에 하위 폴더를 포함하세요(예: `claude --agent my-plugin:review:security`).

프로젝트의 모든 세션에 대해 이를 기본값으로 설정하려면 `.claude/settings.json`에 `agent`를 설정하세요:

```json theme={null}
{
  "agent": "code-reviewer"
}
```

둘 다 존재하는 경우 CLI 플래그가 설정을 재정의합니다.

### 포그라운드 또는 백그라운드에서 서브에이전트 실행

서브에이전트는 포그라운드 또는 백그라운드에서 실행할 수 있습니다:

* **포그라운드 서브에이전트**: 완료될 때까지 메인 대화를 차단합니다. 권한 프롬프트가 발생하는 대로 전달됩니다.
* **백그라운드 서브에이전트**: 작업을 계속하는 동안 동시 실행됩니다. {/* min-version: 2.1.186 */}v2.1.186부터 백그라운드 서브에이전트가 권한이 필요한 도구 호출에 도달하면 프롬프트가 메인 세션에 표시되고 요청하는 서브에이전트의 이름을 명시합니다. 승인하면 서브에이전트가 계속되고 Esc를 누르면 서브에이전트를 중지하지 않고 해당 단일 도구 호출을 거부합니다. v2.1.186 이전에는 백그라운드 서브에이전트가 프롬프트를 표시할 도구 호출을 자동으로 거부했습니다.

{/* min-version: 2.1.198 */}v2.1.198부터 서브에이전트는 기본적으로 백그라운드에서 실행됩니다. Claude는 계속하기 전에 결과가 필요할 때 포그라운드에서 서브에이전트를 실행합니다. 기본값 변경은 서브에이전트가 실행되는 위치를 변경하는 것이지 허용되는 작업을 변경하는 것이 아닙니다. 백그라운드 서브에이전트는 여전히 메인 세션에 모든 권한 프롬프트를 표시합니다. v2.1.198 이전에는 Claude가 작업에 따라 포그라운드와 백그라운드 중 선택했습니다.

{/* min-version: 2.1.211 */}백그라운드 서브에이전트의 결과는 이후 턴에서 완료 알림으로 Claude에 전달됩니다. Claude는 서브에이전트의 결과를 보고하기 전에 해당 알림을 기다리며, 진행 상황에 대해 먼저 물어보면 서브에이전트가 여전히 실행 중이라고 보고합니다. v2.1.211 이전에는 Claude가 완료되지 않은 백그라운드 서브에이전트에 대한 결과를 보고하기도 했습니다.

이를 직접 제어할 수도 있습니다:

* Claude에게 백그라운드 또는 포그라운드에서 작업을 실행하도록 요청
* 실행 중인 작업을 백그라운드로 보내려면 **Ctrl+B** 누름

{/* min-version: 2.1.208 */}완료된 백그라운드 서브에이전트는 세션이 작업 목록을 정리할 때까지 완료됨으로 표시되고 실행 중인 작업 아래에 정렬되어 [`/tasks`](/docs/en/commands)에 계속 나열됩니다. 해당 세부 정보 뷰는 서브에이전트가 종료되어도 열린 상태로 유지됩니다. 실패했거나 사용자가 중지한 서브에이전트는 목록에서 제외됩니다. v2.1.208 이전에는 완료된 서브에이전트가 완료되는 순간 목록에서 사라지고 세부 정보 뷰가 닫혔습니다.

모든 백그라운드 작업 기능을 비활성화하려면 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` 환경 변수를 `1`로 설정하세요. [Environment variables](/docs/en/env-vars)를 참조하세요.

[`CLAUDE_CODE_FORK_SUBAGENT`](#fork-the-current-conversation)가 `1`로 설정되어 있으면 포크 모드가 `Agent` 도구에서 `run_in_background` 매개변수를 제거하므로 모든 서브에이전트가 백그라운드에서 실행되고 프론트매터 `background` 필드는 효과가 없습니다. `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`가 포크 모드보다 우선하여 서브에이전트를 동기 상태로 유지합니다.

### 서브에이전트에서의 API 오류

{/* min-version: 2.1.199 */}v2.1.199부터 사용량 한도나 반복적인 서버 오류와 같은 API 오류로 실행이 끝난 서브에이전트는 오류 텍스트를 발견 사항인 것처럼 반환하는 대신 Claude에게 해당 실패를 보고합니다. Claude가 받는 내용은 서브에이전트가 실행된 위치에 따라 다릅니다:

* **포그라운드**: 속도 제한, 과부하 또는 서버 오류로 인해 이미 텍스트 출력을 생성한 서브에이전트가 중단된 경우 Agent 도구는 서브에이전트가 중단되어 작업을 완료하지 못했다는 메모와 함께 해당 부분 출력을 반환합니다. {/* min-version: 2.1.200 */}아무것도 생성하지 않았거나 유일한 출력이 도구 호출이었던 서브에이전트는 [`Agent terminated early due to an API error`](/docs/en/errors#agent-terminated-early-due-to-an-api-error) 뒤에 오류 세부 정보가 이어지며 실패합니다. v2.1.199에서는 도구 호출 전용 형태를 중단시킨 속도 제한, 과부하 또는 서버 오류가 중단 메모만 포함된 빈 부분 결과를 반환했습니다.
* **백그라운드**: 서브에이전트가 실패로 표시되고, 종료될 때 Claude가 받는 메시지에 API 오류의 이름이 명시되고 서브에이전트의 마지막 출력이 포함되므로 부분 작업이 손실되지 않습니다.

밑바탕의 API 오류가 해결되면 Claude에게 작업을 재시도하거나 [서브에이전트를 재개](#resume-subagents)하도록 요청하세요.

### 서브에이전트 출력 스캔

Claude Code는 Claude가 읽기 전에 각 서브에이전트의 최종 보고서를 스캔합니다. 서브에이전트는 검토하지 않은 파일, 웹 페이지 또는 명령 출력을 읽었을 수 있으며 해당 출처의 텍스트에는 메인 대화를 겨냥한 지침이 포함될 수 있습니다. 스캔은 내용을 제거하거나 말을 바꾸지 않으며 보고서에서 알아차릴 수 있는 두 가지 종류의 변경을 수행합니다:

* **백슬래시 삽입**: 스캔은 `<system-reminder>` 태그나 `Human:` 또는 `Assistant:`로 시작하는 라인과 같이 Claude Code 자체 출력을 흉내 내는 텍스트에 백슬래시를 삽입하여 대화의 일부로 오인되지 않고 일반 텍스트로 읽히도록 합니다.
* **마커 라인**: 보고서가 `<system-reminder>`와 같은 태그를 흉내 내거나 `bypassPermissions` 또는 `--dangerously-skip-permissions`와 같은 권한 설정을 언급할 때 스캔은 `[harness: subagent output matched instruction-shaped pattern(s):`로 시작하는 라인을 앞에 덧붙입니다. 권한 설정 언급은 마커 라인을 받지만 텍스트 자체는 작성된 대로 유지됩니다.

스캔은 콘텐츠가 악의적인지 여부를 판단하지 않으며 보고서의 지침이 수행할 수 있는 작업을 변경하지 않습니다. 보고서가 Claude로 하여금 실행하게 만드는 도구 호출은 여전히 세션의 [권한 검사](/docs/en/permissions) 및 [샌드박스 적용](/docs/en/sandboxing)을 거칩니다. 이는 [서브에이전트가 접근할 수 있는 범위 제한](#control-subagent-capabilities)의 대체재가 아닙니다.

<Note>
  서브에이전트 출력 스캔은 Claude Code v2.1.210 이상이 필요합니다.
</Note>

### 공통 패턴

#### 대용량 작업 격리

서브에이전트의 가장 효과적인 용도 중 하나는 많은 양의 출력을 생성하는 작업을 격리하는 것입니다. 테스트 실행, 문서 가져오기, 또는 로그 파일 처리는 상당한 컨텍스트를 소비할 수 있습니다. 이를 서브에이전트에 위임함으로써 장황한 출력은 서브에이전트의 컨텍스트에 남아있고 관련 요약만 메인 대화로 반환됩니다.

```text wrap theme={null}
Use a subagent to run the test suite and report only the failing tests with their error messages
```

#### 병렬 리서치 실행

독립적인 조사 작업의 경우 여러 서브에이전트를 스폰하여 동시에 작업하도록 하세요:

```text wrap theme={null}
Research the authentication, database, and API modules in parallel using separate subagents
```

각 서브에이전트는 독립적으로 해당 영역을 탐색한 다음 Claude가 결과를 종합합니다. 이는 리서치 경로가 서로 의존하지 않을 때 가장 잘 작동합니다.

<Warning>
  서브에이전트가 완료되면 그 결과가 메인 대화로 반환됩니다. 각각 상세한 결과를 반환하는 많은 서브에이전트를 실행하면 상당한 컨텍스트가 소비될 수 있습니다.
</Warning>

지속적인 병렬 처리가 필요하거나 컨텍스트 창을 초과하는 작업의 경우 [agent teams](/docs/en/agent-teams)를 사용하여 각 워커에 독립적인 컨텍스트를 제공하세요.

#### 서브에이전트 체이닝 (Chain subagents)

다단계 워크플로의 경우 Claude에게 서브에이전트를 순차적으로 사용하도록 요청하세요. 각 서브에이전트는 작업을 완료하고 결과를 Claude에게 반환하며, Claude는 관련 컨텍스트를 다음 서브에이전트에 전달합니다.

```text wrap theme={null}
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

### 서브에이전트와 메인 대화 중 선택

다음의 경우 **메인 대화**를 사용하세요:

* 자주 주고받아야 하거나 반복적인 개선이 필요한 작업
* 플래닝, 구현, 테스트와 같이 여러 단계가 상당한 컨텍스트를 공유하는 경우
* 빠르고 국소적인 변경을 수행할 때
* 지연 시간이 중요할 때. 서브에이전트는 새로 시작되므로 컨텍스트를 수집하는 데 시간이 걸릴 수 있습니다

다음의 경우 **서브에이전트**를 사용하세요:

* 메인 컨텍스트에 필요하지 않은 장황한 출력을 생성하는 작업
* 특정 도구 제한이나 권한을 강제 적용하고 싶을 때
* 작업이 자체 완결적이고 요약을 반환할 수 있는 경우

격리된 서브에이전트 컨텍스트가 아니라 메인 대화 컨텍스트에서 실행되는 재사용 가능한 프롬프트나 워크플로를 원할 경우 대신 [Skills](/docs/en/skills)를 고려하세요.

대화에 이미 존재하는 내용에 대한 빠른 질문은 서브에이전트 대신 [`/btw`](/docs/en/interactive-mode#side-questions-with-%2Fbtw)를 사용하세요. 전체 컨텍스트를 보지만 도구 접근 권한이 없으며 답변은 히스토리에 추가되지 않고 버려집니다.

### 중첩된 서브에이전트 스폰

{/* min-version: 2.1.172 */}Claude Code v2.1.172부터 서브에이전트가 자체 서브에이전트를 스폰할 수 있습니다. 위임된 작업 자체가 병렬 하위 작업으로 분할될 때(예: 발견 사항별로 검증기를 디스패치하는 리뷰어 서브에이전트) 중간 출력이 메인 대화에 도달하지 않도록 이 기능을 사용하세요. 최상위 서브에이전트의 요약만 본인에게 반환됩니다.

중첩된 서브에이전트는 최상위 서브에이전트와 동일한 방식으로 구성되며 동일한 [범위](#choose-the-subagent-scope)에서 해석됩니다.

프롬프트 입력 아래의 서브에이전트 패널에는 전체 트리가 표시됩니다. 각 행에는 하위 항목의 `(+N)` 개수가 표시되고, {/* min-version: 2.1.193 */}v2.1.193부터는 행을 열면 해당 서브에이전트의 형제 및 직계 자식이 `main`으로 돌아가는 경로와 함께 표시됩니다.

깊이는 각 수준이 [포그라운드 또는 백그라운드](#run-subagents-in-foreground-or-background)에서 실행되는지 여부와 관계없이 메인 대화 아래의 서브에이전트 수준 수로 계산됩니다. 깊이 5에 있는 서브에이전트는 Agent 도구를 받지 못하며 추가 스폰을 할 수 없습니다. 이 제한은 고정되어 있으며 구성할 수 없습니다.

Claude Code v2.1.187부터 백그라운드 서브에이전트의 깊이는 처음 스폰될 때 고정되며 나중에 [재개](#resume-subagents)하더라도 해당 깊이가 변경되지 않습니다. 예를 들어 메인 대화가 서브에이전트 A를 스폰하고 A가 깊이 2에서 백그라운드 서브에이전트 B를 스폰한 경우, 메인 대화에서 B를 직접 재개할 때 B는 여전히 깊이 2에 있습니다. 더 얕은 컨텍스트에서 서브에이전트를 재개하더라도 깊이 제한이 이미 차단한 추가 수준을 스폰할 수는 없습니다.

특정 서브에이전트가 다른 서브에이전트를 스폰하지 못하도록 하려면 [`tools`](#available-tools) 목록에서 `Agent`를 생략하거나 `disallowedTools`에 추가하세요.

[포크](#fork-the-current-conversation)는 여전히 다른 포크를 스폰할 수 없습니다. 포크는 다른 서브에이전트 유형을 스폰할 수 있으며 이는 깊이 제한에 합산됩니다.

### 세션 서브에이전트 제한

기본적으로 Claude는 세션당 최대 200개의 서브에이전트를 스폰할 수 있습니다. 한도를 늘리려면 [`CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION`](/docs/en/env-vars)을 임의의 양의 정수로 설정하세요. 상한선은 없지만 제한을 끌 수는 없습니다. Claude Code v2.1.212 이상이 필요합니다.

Claude가 Agent 도구로 스폰하는 모든 서브에이전트(중첩 서브에이전트, [포크](#fork-the-current-conversation), 백그라운드 서브에이전트, [워크플로](/docs/en/workflows)의 에이전트가 Agent 도구로 스폰하는 서브에이전트 포함)가 이 한도에 합산됩니다. `/subtask`로 직접 시작하는 세션 내 포크도 합산됩니다. 동일한 예산을 소비하지만 제한은 Claude가 Agent 도구로 스폰하는 서브에이전트만 차단하므로 Claude가 한도에 도달한 후에도 본인의 `/subtask`는 시작됩니다. `/fork`로 생성하는 세션은 합산되지 않으며 자체 예산을 가진 별도의 백그라운드 세션으로 실행됩니다. v2.1.212 이전에는 세션 내 포크 이름이 `/fork`였습니다. 워크플로 스크립트가 `agent()`로 스폰하는 에이전트는 합산되지 않으며 워크플로는 별도의 실행당 한도를 갖습니다. 완료된 서브에이전트도 계속 합산됩니다.

Claude가 한도에 도달하면 Agent 도구가 `Subagent spawn limit reached`로 실패하고 오류가 Claude에게 자체 도구로 나머지 작업을 직접 완료하도록 지시합니다.

카운트를 재설정하고 전체 예산으로 새 대화를 시작하려면 [`/clear`](/docs/en/commands#all-commands)를 실행하세요. 실행 중인 워크플로와 같이 서브에이전트를 계속 스폰할 수 있는 작업이 clear 후에도 살아남으면 카운트가 이월됩니다.

이 제한은 서브에이전트의 중첩 깊이를 제한하는 [깊이 제한](#spawn-nested-subagents)과 별개입니다.

### 서브에이전트 컨텍스트 관리

#### 시작 시 로드되는 항목

각 서브에이전트는 새로 만든 격리된 컨텍스트 창에서 시작합니다. 이전 대화 기록, 이미 호출한 skill, Claude가 이미 읽은 파일을 보지 못합니다. Claude는 작업을 요약하는 위임 메시지를 작성하고 서브에이전트는 거기서부터 작업합니다. 예외는 [포크](#fork-the-current-conversation)로, 새로 시작하는 대신 상위 대화를 상속받습니다.

포크가 아닌 서브에이전트의 초기 컨텍스트에는 다음이 포함됩니다:

* **System prompt**: 에이전트 자체의 프롬프트와 Claude Code가 덧붙이는 환경 세부 정보(전체 Claude Code 시스템 프롬프트가 아님). 커스텀 서브에이전트는 [markdown 본문](#write-subagent-files)이나 `prompt` 필드에 자체 프롬프트를 정의합니다. 내장 에이전트에는 사전 정의된 프롬프트가 있습니다.
* **Task message**: 작업을 전달할 때 Claude가 작성하는 위임 프롬프트.
* **CLAUDE.md files**: 메인 대화가 로드하는 [CLAUDE.md 계층 구조](/docs/en/memory#how-claude-md-files-load)의 모든 수준(`~/.claude/CLAUDE.md`, 프로젝트 규칙, `CLAUDE.local.md`, 관리형 정책 파일 포함). 내장된 Explore 및 Plan 에이전트는 이를 건너끕니다.
* **Git status**: 상위 세션 시작 시 촬영한 스냅샷. 작업 디렉터리가 Git 리포지토리가 아니거나 [`includeGitInstructions`](/docs/en/settings#available-settings)가 `false`인 경우 제공되지 않습니다. Explore 및 Plan은 이에 관계없이 건너끕니다.
* **Preloaded skills**: 에이전트의 [`skills` 필드](#preload-skills-into-subagents)에 이름이 명시된 skill의 전체 내용. 내장 에이전트는 skill을 사전 로드하지 않습니다.
* **Sibling roster**: 세션의 `main` 및 다른 모든 명명된 에이전트를 나열하는 시스템 알림으로, 각각은 [`SendMessage`](#resume-subagents)를 위한 유효한 `to` 값입니다. {/* min-version: 2.1.206 */}Claude Code v2.1.206 이상이 필요합니다. 이 목록은 서브에이전트의 도구에 `SendMessage`가 포함되어 있고 최소한 하나의 다른 에이전트에 이름이 있을 때만 표시됩니다(Claude가 스폰할 때 이름을 지정했든 [에이전트 팀](/docs/en/agent-teams) 팀원으로 실행되든 관계없이). 이는 서브에이전트가 시작할 때 촬영한 스냅샷이므로 나중에 이름이 지정된 에이전트는 표시되지 않습니다.

Explore와 Plan은 CLAUDE.md와 git status를 생략하는 유일한 서브에이전트입니다. 어떤 에이전트가 이를 건너뛸지 변경하는 프론트매터 필드나 에이전트별 설정은 없습니다.

메인 대화는 전체 CLAUDE.md 컨텍스트와 함께 Explore 및 Plan 결과를 읽으므로 대부분의 규칙이 서브에이전트 자체에 도달할 필요는 없습니다. "`vendor/` 디렉터리 무시"와 같은 규칙이 반드시 도달해야 하는 경우 위임할 때 Claude에게 제공하는 프롬프트에 이를 재명시하세요.

메인 대화의 일부 상태는 포크가 아닌 서브에이전트에 결코 도달하지 않습니다:

* **Output style**: 서브에이전트는 자체 시스템 프롬프트를 실행하므로 [포크](#fork-the-current-conversation)를 제외하고 사용자의 [출력 스타일](/docs/en/output-styles)이 응답을 형성하지 않습니다.
* **Auto memory**: 메인 대화의 [auto memory](/docs/en/memory#auto-memory)는 로드되지 않습니다. 서브에이전트에 자체 지속성 메모리를 부여하려면 [`memory` 필드](#enable-persistent-memory)를 사용하세요.
* **Context window size**: 서브에이전트의 컨텍스트 창 크기는 상위 창이 아니라 서브에이전트 자체 모델에 의해 결정됩니다. 더 작은 창을 가진 모델에 위임하면 해당 서브에이전트에 더 작은 창이 제공됩니다.

#### 서브에이전트 재개 (Resume subagents)

각 서브에이전트 호출은 새로운 컨텍스트가 있는 새 인스턴스를 생성합니다. 다시 시작하는 대신 기존 서브에이전트의 작업을 계속하려면 Claude에게 재개를 요청하세요.

재개된 서브에이전트는 이전의 모든 도구 호출, 결과 및 추론을 포함하여 전체 대화 기록을 유지합니다. 서브에이전트는 새로 시작하는 대신 정확히 중단된 지점에서 다시 시작합니다.

서브에이전트가 완료되면 Claude는 에이전트 ID를 받습니다. 내장된 Explore 및 Plan 에이전트는 1회성이며 에이전트 ID를 반환하지 않으므로 재개할 수 없습니다. 작업을 계속해야 하는 경우 `general-purpose` 또는 커스텀 서브에이전트를 사용하세요.

Claude는 에이전트의 ID 또는 이름을 `to` 필드로 지정하여 `SendMessage` 도구를 사용해 이를 재개합니다. `SendMessage`는 [에이전트 팀](/docs/en/agent-teams)의 활성화를 요구하지 않으며, `shutdown_request` 및 `plan_approval_response`와 같은 구조화된 팀 프로토콜 메시지만 요구합니다.

서브에이전트를 재개하려면 Claude에게 이전 작업을 계속하도록 요청하세요:

```text wrap theme={null}
Use the code-reviewer subagent to review the authentication module
[Agent completes]

Continue that code review and now analyze the authorization logic
[Claude resumes the subagent with full context from previous conversation]
```

`SendMessage`를 수신한 완료된 서브에이전트는 새로운 `Agent` 호출 없이 백그라운드에서 자동 재개됩니다. Claude가 `TaskStop` 도구로 중지한 서브에이전트에도 동일하게 적용됩니다.

{/* min-version: 2.1.191 */}v2.1.191부터 사용자가 직접 중지한 서브에이전트(`/tasks`에서 `x`를 누르거나 SDK `stop_task` 요청 사용)는 자동 재개되지 않습니다. `SendMessage` 호출은 에이전트가 취소되었음을 Claude에게 알려주는 거부를 반환합니다. 해당 서브에이전트가 다시 자동 재개될 수 있도록 서브에이전트 패널의 해당 서브에이전트 트랜스크립트에 입력하여 직접 재개하면 중지가 해제됩니다.

재개는 동일한 ID로 에이전트의 새 실행을 시작하므로 이미 실패했거나 완료된 서브에이전트가 작업 목록과 Agent SDK의 작업 이벤트에서 다시 실행 중으로 표시됩니다. v2.1.205 이전에는 재개된 실행이 작업되는 동안 이전의 실패 또는 완료 상태를 계속 보여주었습니다.

{/* min-version: 2.1.199 */}v2.1.199부터 `SendMessage`는 이름이 대화의 앞부분에서 도달했던 동일한 에이전트를 여전히 가리키는지 확인합니다. 해당 이름을 재사용한 재스폰된 백그라운드 에이전트와 같이 더 새로운 에이전트가 이름을 차지한 경우, Claude Code는 잘못된 에이전트에 전달하는 대신 전송을 거부하고 오류를 통해 이름이 현재 도달하는 에이전트를 보고하여 Claude가 대상을 다시 지정할 수 있도록 합니다. 여전히 실행 중인 이전 에이전트에 도달하려면 Claude는 해당 에이전트를 스폰했을 때 받은 에이전트 ID로 주소를 지정합니다. 이 검사는 현재 대화로 범위가 지정되며 `/clear` 시 재설정됩니다.

{/* min-version: 2.1.198 */}v2.1.198부터 서브에이전트는 자신을 실행한 에이전트의 메시지를 작업 진행 중 방향 전환을 포함한 일반적인 작업 지시로 취급하고 자체 권한 설정 내에서 이에 따라 작동합니다. 메시지를 보낸 주체와 관계없이 두 가지 제한 사항이 유지됩니다. 어떤 에이전트의 메시지도 보류 중인 권한 프롬프트에 대한 승인으로 간주되지 않으며 어떤 에이전트 메시지도 서브에이전트의 권한 설정, `CLAUDE.md` 또는 구성을 변경할 수 없습니다. 오직 권한 시스템이나 본인의 메시지만 승인을 부여할 수 있습니다.

명시적으로 참조하고 싶은 경우 Claude에게 에이전트 ID를 요청하거나 `~/.claude/projects/{project}/{sessionId}/subagents/`의 트랜스크립트 파일에서 ID를 찾을 수도 있습니다. 각 트랜스크립트는 `agent-{agentId}.jsonl`로 저장됩니다.

서브에이전트 트랜스크립트는 메인 대화와 독립적으로 유지됩니다:

* **메인 대화 축소 (Compaction)**: 메인 대화가 축소되어도 서브에이전트 트랜스크립트는 영향을 받지 않습니다. 별도의 파일에 저장되기 때문입니다.
* **세션 지속성**: 서브에이전트 트랜스크립트는 해당 세션 내에서 유지됩니다. 동일한 세션을 재개하여 Claude Code를 재시작한 후에도 [서브에이전트를 재개](#resume-subagents)할 수 있습니다.
* **자동 정리**: 기본값이 30일인 `cleanupPeriodDays` 설정을 기준으로 트랜스크립트가 정리됩니다.

#### 자동 축소 (Auto-compaction)

서브에이전트는 메인 대화와 동일한 로직을 사용하여 자동 축소를 지원합니다. 축소는 동일한 조건에서 트리거되며 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`도 서브에이전트에 적용됩니다. 오버라이드가 적용되는 시점은 [environment variables](/docs/en/env-vars)를 참조하세요.

축소 이벤트는 서브에이전트 트랜스크립트 파일에 로깅됩니다:

```json theme={null}
{
  "type": "system",
  "subtype": "compact_boundary",
  "compactMetadata": {
    "trigger": "auto",
    "preTokens": 167189
  }
}
```

`preTokens` 값은 축소가 발생하기 전에 사용된 토큰 수를 보여줍니다.

## 현재 대화 포크하기 (Fork the current conversation)

<Note>
  {/* min-version: 2.1.212 */}포크된 서브에이전트는 `/subtask`로 실행하며 Claude Code v2.1.212 이상이 필요합니다. [agent view가 꺼져 있을 때](/docs/en/agent-view#turn-off-agent-view) `/subtask`를 사용할 수 없으며 대신 `/fork`가 포크된 서브에이전트를 시작합니다. 그렇지 않으면 `/fork`는 전체 세션을 새 [백그라운드 세션](/docs/en/agent-view#from-inside-a-session)으로 복사합니다.

  {/* min-version: 2.1.161 */}v2.1.212 이전에는 포크된 서브에이전트 명령이 `/fork`였습니다. v2.1.161 이상에서는 기본적으로 활성화되어 있었으며, v2.1.117부터 v2.1.160까지는 서버 측 롤아웃이 활성화하지 않는 한 [`CLAUDE_CODE_FORK_SUBAGENT`](/docs/en/env-vars) 환경 변수를 `1`로 설정해야 했습니다.

  Claude 자체가 포크를 스폰하도록 허용하는 것은 실험적인 기능이며 향후 릴리스에서 변경될 수 있습니다. 이 기능은 단계적 롤아웃의 일부로 대화형 세션에서도 활성화될 수 있습니다.
</Note>

포크는 새로 시작하는 대신 지금까지의 전체 대화를 상속받는 서브에이전트입니다. 이는 서브에이전트가 제공하는 입력 격리를 해제합니다. 포크는 메인 세션과 동일한 시스템 프롬프트, 도구, 모델 및 메시지 기록을 보므로 상황을 다시 설명하지 않고도 사이드 작업을 전달할 수 있습니다. 포크의 자체 도구 호출은 대화 외부에 유지되고 최종 결과만 반환되므로 메인 컨텍스트 창이 깨끗하게 유지됩니다. 명명된 서브에이전트가 유용하기에 너무 많은 배경 지식이 필요한 경우 또는 동일한 시작점에서 여러 접근 방식을 병렬로 시도하려는 경우 포크를 사용하세요.

단계적 롤아웃과 관계없이 포크 모드를 제어하려면 [`CLAUDE_CODE_FORK_SUBAGENT`](/docs/en/env-vars)를 명시적으로 활성화하려면 `1`로, 비활성화하려면 `0`으로 설정하세요. 이 변수는 대화형 모드와 SDK 또는 `claude -p`를 통해 준수됩니다.

포크 모드를 활성화하면 두 가지 방식으로 Claude Code가 변경됩니다:

* Claude는 `fork` 서브에이전트 유형을 명시적으로 요청하여 포크를 스폰할 수 있습니다. Claude가 유형을 요청하지 않는 경우 여전히 [general-purpose](#built-in-subagents) 서브에이전트를 받으며 Explore와 같은 명명된 서브에이전트는 이전과 같이 스폰됩니다.
* 포크이든 명명된 서브에이전트이든 관계없이 모든 서브에이전트가 [백그라운드](#run-subagents-in-foreground-or-background)에서 실행됩니다. 서브에이전트를 동기화 상태로 유지하려면 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`를 `1`로 설정하세요.

변수 설정 여부와 관계없이 작업 내용 뒤에 `/subtask`를 사용하여 포크를 직접 시작할 수 있습니다. v2.1.161부터 v2.1.211까지는 명령어가 `/fork`였습니다. Claude Code는 작업의 첫 번째 단어들로부터 포크 이름을 지정합니다. 다음 예시는 메인 세션에서 구현을 계속 진행하는 동안 지금까지의 파서 변경 사항에 대한 단위 테스트 초안을 작성하기 위해 대화를 포크합니다:

```text wrap theme={null}
/subtask draft unit tests for the parser changes so far
```

포크는 프롬프트 아래 패널에 표시되며 작업을 계속하는 동안 백그라운드에서 실행됩니다. 완료되면 그 결과가 메인 대화에 메시지로 도착합니다. 다음 섹션에서는 실행 중인 포크를 관찰하고 제어하기 위한 패널 키 컨트롤을 다룹니다.

### 실행 중인 포크 관찰 및 제어

실행 중인 포크는 프롬프트 입력 아래 패널에 표시되며 메인 세션용 행 하나와 각 포크용 행 하나로 구성됩니다. 이 키들을 사용하여 패널과 상호작용하세요:

| 키 | 작업 |
| :-------- | :----------------------------------------------------------------- |
| `↑` / `↓` | 행 간 이동 |
| `Enter` | 선택한 포크의 트랜스크립트를 열고 후속 메시지 보내기 |
| `x` | 완료된 포크를 지우거나 실행 중인 포크 중지 |
| `Esc` | 프롬프트 입력으로 포커스 복귀 |

포크 또는 서브에이전트의 트랜스크립트가 열려 있으면 후속 메시지와 [skills](/docs/en/skills)는 해당 에이전트로 전달되지만 내장 명령은 메인 대화에서 여전히 실행됩니다. {/* min-version: 2.1.199 */}v2.1.199부터 해당 뷰에서 `/model` 또는 `/fast`를 입력하면 뷰에 표시된 에이전트가 아니라 메인 대화의 모델 또는 fast 모드가 변경된다는 알림이 표시됩니다.

### 포크와 명명된 서브에이전트의 차이점

포크는 스폰되는 순간 메인 세션이 가진 모든 것을 상속받습니다. 명명된 서브에이전트는 자체 정의에서 시작합니다.

| | 포크 | 명명된 서브에이전트 |
| :---------------------- | :------------------------------- | :---------------------------------------------------------------------------------------------------------------- |
| 컨텍스트 | 전체 대화 기록 | 전달하는 프롬프트가 포함된 새로운 컨텍스트 |
| 시스템 프롬프트 및 도구 | 메인 세션과 동일 | 서브에이전트의 [정의 파일](#write-subagent-files)에서 가져옴 |
| 모델 | 메인 세션과 동일 | 서브에이전트의 `model` 필드에서 가져옴 |
| 권한 | 프롬프트가 터미널에 표시됨 | 백그라운드 실행 시 [프롬프트가 메인 세션에 표시됨](#run-subagents-in-foreground-or-background) |
| 프롬프트 캐시 | 메인 세션과 공유 | 별도의 캐시 |

포크의 시스템 프롬프트 및 도구 정의는 상위 항목과 동일하므로 첫 번째 요청이 상위 항목의 [프롬프트 캐시](/docs/en/prompt-caching#subagents-and-the-cache)를 재사용합니다. 이로 인해 동일한 컨텍스트가 필요한 작업의 경우 포크가 새로운 서브에이전트를 스폰하는 것보다 비용이 저렴합니다.

Claude가 Agent 도구를 통해 포크를 스폰할 때 `isolation: "worktree"`를 전달할 수 있으므로 포크의 파일 편집 내용이 사용자의 체크아웃 대신 별도의 git 워크트리에 기록됩니다.

### 제한 사항

`CLAUDE_CODE_FORK_SUBAGENT=1`을 설정하면 대화형 세션, [비대화형 모드](/docs/en/headless) 및 Agent SDK에서 포크 모드가 활성화되고, `0`으로 설정하면 서버 측 롤아웃을 포함하여 모든 곳에서 포크 모드가 비활성화됩니다. 포크는 다른 포크를 스폰할 수 없습니다.

## 서브에이전트 예시

이 예시들은 서브에이전트를 구축하기 위한 효과적인 패턴을 보여줍니다. 시작점으로 사용하거나 Claude로 커스텀 버전을 생성하세요.

<Tip>
  **모범 사례:**

  * **집중된 서브에이전트 설계:** 각 서브에이전트는 한 가지 특정 작업에 뛰어나야 합니다
  * **상세한 설명 작성:** Claude는 위임할 시점을 결정하기 위해 설명을 사용합니다
  * **도구 접근 제한:** 보안과 집중을 위해 필요한 권한만 부여하세요
  * **버전 제어에 커밋:** 팀과 프로젝트 서브에이전트를 공유하세요
</Tip>

### Code reviewer

코드를 수정하지 않고 검토하는 읽기 전용 서브에이전트입니다. 이 예시는 Edit 및 Write를 제외한 제한된 도구 접근 권한과 주의 깊게 살펴볼 내용 및 출력 서식 지정 방식을 정확히 지정하는 상세한 프롬프트를 가진 집중된 서브에이전트를 설계하는 방법을 보여줍니다.

```markdown theme={null}
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

### Debugger

이슈를 분석하고 수정할 수 있는 서브에이전트입니다. Code reviewer와 달리 버그를 수정하려면 코드 수정이 필요하므로 Edit가 포함됩니다. 프롬프트는 진단부터 검증까지 명확한 워크플로를 제공합니다.

```markdown theme={null}
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Debugging process:
- Analyze error messages and logs
- Check recent code changes
- Form and test hypotheses
- Add strategic debug logging
- Inspect variable states

For each issue, provide:
- Root cause explanation
- Evidence supporting the diagnosis
- Specific code fix
- Testing approach
- Prevention recommendations

Focus on fixing the underlying issue, not the symptoms.
```

### Data scientist

데이터 분석 작업을 위한 도메인 특화 서브에이전트입니다. 이 예시는 일반적인 코딩 작업 외부의 특화된 워크플로를 위한 서브에이전트를 생성하는 방법을 보여줍니다. 유능한 분석을 위해 `model: sonnet`을 명시적으로 설정합니다.

```markdown theme={null}
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Key practices:
- Write optimized SQL queries with proper filters
- Use appropriate aggregations and joins
- Include comments explaining complex logic
- Format results for readability
- Provide data-driven recommendations

For each analysis:
- Explain the query approach
- Document any assumptions
- Highlight key findings
- Suggest next steps based on data

Always ensure queries are efficient and cost-effective.
```

### Database query validator

Bash 접근을 허용하지만 읽기 전용 SQL 쿼리만 허용하도록 명령을 검증하는 서브에이전트입니다. 이 예시는 `tools` 필드가 제공하는 것보다 정교한 제어가 필요할 때 조건부 검증을 위해 `PreToolUse` 훅을 사용하는 방법을 보여줍니다.

```markdown theme={null}
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data or generating reports.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access. Execute SELECT queries to answer questions about the data.

When asked to analyze data:
1. Identify which tables contain the relevant data
2. Write efficient SELECT queries with appropriate filters
3. Present results clearly with context

You cannot modify data. If asked to INSERT, UPDATE, DELETE, or modify schema, explain that you only have read access.
```

Claude Code는 stdin을 통해 [훅 입력을 JSON으로 전달](/docs/en/hooks#pretooluse-input)합니다. 검증 스크립트는 이 JSON을 읽고, 실행 중인 명령을 추출하며, SQL 쓰기 작업 목록과 비교하여 검사합니다. 쓰기 작업이 감지되면 스크립트는 [종료 코드 2로 종료](/docs/en/hooks#exit-code-2-behavior-per-event)하여 실행을 차단하고 stderr을 통해 Claude에게 오류 메시지를 반환합니다.

프로젝트의 임의의 위치에 검증 스크립트를 생성하세요. 경로는 훅 구성의 `command` 필드와 일치해야 합니다:

```bash theme={null}
#!/bin/bash
# Blocks SQL write operations, allows SELECT queries

# Read JSON input from stdin
INPUT=$(cat)

# Extract the command field from tool_input using jq
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

# Block write operations (case-insensitive)
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT queries only." >&2
  exit 2
fi

exit 0
```

macOS 및 Linux에서는 스크립트를 실행 가능하도록 설정합니다:

```bash theme={null}
chmod +x ./scripts/validate-readonly-query.sh
```

Windows에서는 PowerShell로 검증 스크립트를 작성하고 훅 항목에 `shell: powershell`을 추가합니다. [running hooks in PowerShell](/docs/en/hooks#windows-powershell-tool)을 참조하세요.

훅은 `tool_input.command`에 Bash 명령이 포함된 JSON을 stdin으로 받습니다. 종료 코드 2는 작업을 차단하고 오류 메시지를 Claude에 전달합니다. 종료 코드 세부 정보는 [Hooks](/docs/en/hooks#exit-code-output)를, 전체 입력 스키마는 [Hook input](/docs/en/hooks#pretooluse-input)을 참조하세요.

## 다음 단계

서브에이전트를 이해했으므로 관련 기능을 살펴보세요:

* 팀 또는 프로젝트 간에 서브에이전트를 공유하려면 [Distribute subagents with plugins](/docs/en/plugins) 참조
* CI/CD 및 자동화를 위해 Agent SDK로 프로그래밍 방식으로 Claude Code를 실행하려면 [Run Claude Code programmatically](/docs/en/headless) 참조
* 외부 도구 및 데이터에 대한 서브에이전트 접근 권한을 부여하려면 [Use MCP servers](/docs/en/mcp) 참조
