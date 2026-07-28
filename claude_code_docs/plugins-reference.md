> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 플러그인 참조 문서

> 스키마, CLI 명령, 구성 요소 명세를 포함한 Claude Code 플러그인 시스템의 전체 기술 참조 문서입니다.

<Tip>
  플러그인을 설치하려는 경우 [플러그인 탐색 및 설치](/docs/en/discover-plugins)를 참조하세요. 플러그인을 직접 제작하려는 경우 [플러그인](/docs/en/plugins)을 참조하세요. 플러그인을 배포하려는 경우 [플러그인 마켓플레이스](/docs/en/plugin-marketplaces)를 참조하세요.
</Tip>

이 참조 문서는 구성 요소 스키마, CLI 명령, 개발 도구를 포함하여 Claude Code 플러그인 시스템에 대한 완전한 기술 명세를 제공합니다.

**플러그인**은 커스텀 기능으로 Claude Code를 확장하는 독립된 디렉토리 형태의 구성 요소 모음입니다. 플러그인 구성 요소에는 스킬, 에이전트, 훅, MCP 서버, LSP 서버, 모니터가 포함됩니다.

## 플러그인 구성 요소 참조

### 스킬 (Skills)

플러그인은 Claude Code에 스킬을 추가하여 사용자가 직접 또는 Claude가 자동으로 호출할 수 있는 `/name` 단축키를 만듭니다.

**위치**: 플러그인 루트의 `skills/` 또는 `commands/` 디렉토리, 또는 플러그인 루트의 단일 `SKILL.md` 파일

**파일 형식**: 스킬은 `SKILL.md`를 포함하는 디렉토리이며, 명령(commands)은 단순 마크다운 파일입니다.

**스킬 구조**:

```text theme={null}
skills/
├── pdf-processor/
│   ├── SKILL.md
│   ├── reference.md (선택 사항)
│   └── scripts/ (선택 사항)
└── code-reviewer/
    └── SKILL.md
```

**통합 동작**:

* 플러그인이 설치되면 스킬 및 명령이 자동으로 탐색됩니다.
* Claude는 태스크 컨텍스트에 따라 스킬을 자동으로 호출할 수 있습니다.
* 스킬은 `SKILL.md` 옆에 보조 파일들을 포함할 수 있습니다.

플러그인에 `skills/` 디렉토리와 `skills` 매니페스트 필드가 모두 없는 경우, 플러그인 루트의 `SKILL.md`가 단일 스킬로 로드됩니다. 스킬의 호출 이름을 제어하려면 프론트매터의 `name` 필드를 설정하세요. 설정하지 않으면 Claude Code는 설치 디렉토리 이름으로 폴백하는데, 마켓플레이스에 설치된 플러그인의 경우 업데이트마다 변경되는 버전 문자열이 됩니다. 둘 이상의 스킬을 제공하는 플러그인의 경우 위와 같은 `skills/` 디렉토리 레이아웃을 사용하세요.

자세한 내용은 [스킬](/docs/en/skills)을 참조하세요.

### 에이전트 (Agents)

플러그인은 적절할 때 Claude가 자동으로 호출할 수 있는 특정 태스크용 전문 subagent를 제공할 수 있습니다.

**위치**: 플러그인 루트의 `agents/` 디렉토리

**파일 형식**: 에이전트 기능을 설명하는 마크다운 파일

**에이전트 구조**:

```markdown theme={null}
---
name: agent-name
description: What this agent specializes in and when Claude should invoke it
model: sonnet
effort: medium
maxTurns: 20
disallowedTools: Write, Edit
---

Detailed system prompt for the agent describing its role, expertise, and behavior.
```

플러그인 에이전트는 `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background`, `isolation` 프론트매터 필드를 지원합니다. 유효한 단 하나의 `isolation` 값은 `"worktree"`입니다. 보안상의 이유로 `hooks`, `mcpServers`, `permissionMode`는 플러그인이 제공하는 에이전트에서 지원되지 않습니다.

**통합 포인트**:

* 플러그인이 활성화되면 에이전트가 `my-plugin:code-reviewer`와 같이 네임스페이스가 지정된 이름으로 [@-멘션 자동 완성](/docs/en/sub-agents#invoke-subagents-explicitly)에 나타납니다.
* Claude는 태스크 컨텍스트를 바탕으로 에이전트를 자동으로 호출할 수 있습니다.
* 사용자가 에이전트를 수동으로 호출할 수 있습니다.
* 플러그인 에이전트는 Claude의 내장 에이전트와 함께 동작합니다.

자세한 내용은 [Subagents](/docs/en/sub-agents)를 참조하세요.

### 훅 (Hooks)

플러그인은 Claude Code 이벤트에 자동으로 응답하는 이벤트 핸들러를 제공할 수 있습니다.

**위치**: 플러그인 루트의 `hooks/hooks.json`, 또는 plugin.json 내의 인라인

**형식**: 이벤트 매처 및 액션이 포함된 JSON 구성

**훅 구성**:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

플러그인 훅은 [사용자 정의 훅](/docs/en/hooks)과 동일한 라이프사이클 이벤트에 응답합니다:

| 이벤트                | 발생 시점                                                                                                                                                              |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SessionStart`        | 세션이 시작되거나 다시 재개될 때                                                                                                                                       |
| `Setup`               | `--init-only`로 Claude Code를 시작하거나, `-p` 모드에서 `--init` 또는 `--maintenance`로 시작할 때. CI나 스크립트에서의 일회성 준비 작업을 위해 사용                     |
| `UserPromptSubmit`    | 프롬프트를 제출한 직후, Claude가 이를 처리하기 전                                                                                                                      |
| `UserPromptExpansion` | 사용자가 입력한 명령이 프롬프트로 확장될 때, Claude에 전달되기 전. 확장을 차단할 수 있음                                                                              |
| `PreToolUse`          | 도구 호출이 실행되기 전. 호출을 차단할 수 있음                                                                                                                         |
| `PermissionRequest`   | 권한 대화 상자가 나타날 때                                                                                                                                             |
| `PermissionDenied`    | 자동 모드 분류기에 의해 도구 호출이 거부되었을 때. 모델이 거부된 도구 호출을 다시 시도할 수 있음을 알리려면 `{retry: true}`를 반환                                     |
| `PostToolUse`         | 도구 호출이 성공적으로 완료된 후                                                                                                                                       |
| `PostToolUseFailure`  | 도구 호출이 실패한 후                                                                                                                                                  |
| `PostToolBatch`       | 병렬 도구 호출의 전체 배치 처리가 완료된 후, 다음 모델 호출 전                                                                                                         |
| `Notification`        | Claude Code가 알림을 보낼 때                                                                                                                                           |
| `MessageDisplay`      | 어시스턴트 메시지 텍스트가 표시되는 동안                                                                                                                               |
| `SubagentStart`       | subagent가 생성될 때                                                                                                                                                   |
| `SubagentStop`        | subagent가 완료되었을 때                                                                                                                                               |
| `TaskCreated`         | `TaskCreate`를 통해 태스크가 생성되는 중일 때                                                                                                                          |
| `TaskCompleted`       | 태스크가 완료된 것으로 표시되는 중일 때                                                                                                                                |
| `Stop`                | Claude가 응답을 마쳤을 때                                                                                                                                              |
| `StopFailure`        | API 오류로 인해 턴이 끝났을 때. 출력 및 종료 코드는 무시됨                                                                                                             |
| `TeammateIdle`        | [에이전트 팀](/docs/en/agent-teams) 팀원이 유휴 상태로 진입하려 할 때                                                                                                  |
| `InstructionsLoaded`  | CLAUDE.md 또는 `.claude/rules/*.md` 파일이 컨텍스트에 로드될 때. 세션 시작 시 및 세션 도중 파일이 지연 로드될 때 발생                                                  |
| `ConfigChange`        | 세션 도중 구성 파일이 변경될 때                                                                                                                                        |
| `CwdChanged`          | 예를 들어 Claude가 `cd` 명령을 실행하여 작업 디렉토리가 변경될 때. direnv와 같은 도구를 사용하여 사후 반응형 환경 관리에 유용함                                       |
| `FileChanged`        | 감시 중인 파일이 디스크에서 변경될 때. `matcher` 필드가 감시할 파일 이름을 지정함                                                                                     |
| `WorktreeCreate`      | `--worktree`, `isolation: "worktree"`, 또는 백그라운드 세션을 위해 worktree가 생성되는 중일 때. 기본 git 동작을 대체함                                                 |
| `WorktreeRemove`      | 세션 종료 시, subagent가 완료될 때, 또는 백그라운드 세션을 삭제할 때 worktree가 제거되는 중일 때                                                                        |
| `PreCompact`          | 컨텍스트 압축(compaction) 직전                                                                                                                                         |
| `PostCompact`         | 컨텍스트 압축이 완료된 직후                                                                                                                                            |
| `Elicitation`         | MCP 서버가 도구 호출 도중 사용자 입력을 요청할 때                                                                                                                     |
| `ElicitationResult`   | 사용자가 MCP 정보 요청에 응답한 후, 응답이 서버로 다시 전송되기 전                                                                                                    |
| `SessionEnd`          | 세션이 종료될 때                                                                                                                                                       |

**훅 유형**:

* `command`: 셸 명령 또는 스크립트 실행
* `http`: 이벤트 JSON을 POST 요청 형태로 URL에 전송
* `mcp_tool`: 구성된 [MCP 서버](/docs/en/mcp)의 도구 호출
* `prompt`: LLM으로 프롬프트 평가 (컨텍스트를 위해 `$ARGUMENTS` 플레이스홀더 사용)
* `agent`: 복잡한 검증 작업을 위한 도구를 탑재한 에이전틱 검증기 구동

플러그인 자체의 [번들 MCP 서버](#mcp-servers)를 대상으로 하는 훅은 해당하는 네임스페이스 이름을 사용해야 합니다. 도구 매처 및 `if` 필드는 네임스페이스가 지정된 도구 이름 `mcp__plugin_<plugin-name>_<server-name>__<tool>`을 취하고, `mcp_tool` 훅의 `server` 필드는 `plugin:<plugin-name>:<server-name>`을 취합니다. 순수한 서버 키로 작성된 매처는 절대 실행되지 않습니다. [MCP 도구 매칭](/docs/en/hooks#match-mcp-tools) 및 [플러그인 제공 MCP 서버](/docs/en/mcp#plugin-provided-mcp-servers)를 참조하세요.

### MCP 서버

플러그인은 Claude Code를 외부 도구 및 서비스와 연결하는 Model Context Protocol (MCP) 서버를 번들로 포함할 수 있습니다.

**위치**: 플러그인 루트의 `.mcp.json`, 또는 plugin.json 내의 인라인

**형식**: 표준 MCP 서버 구성

**MCP 서버 구성**:

```json theme={null}
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    },
    "plugin-api-client": {
      "command": "npx",
      "args": ["@company/mcp-server", "--plugin-mode"]
    }
  }
}
```

**통합 동작**:

* 플러그인이 활성화되면 플러그인 MCP 서버가 자동으로 시작됩니다.
* 서버는 Claude 도구 세트의 표준 MCP 도구로 나타납니다.
* 서버 기능이 Claude의 기존 도구와 원활하게 통합됩니다.
* 사용자 MCP 서버와 독립적으로 플러그인 서버를 구성할 수 있습니다.

### LSP 서버

<Tip>
  LSP 플러그인을 사용하려는 경우 공식 마켓플레이스에서 설치하세요: `/plugin` Discover 탭에서 "lsp"를 검색합니다. 이 섹션은 공식 마켓플레이스가 다루지 않는 언어에 대한 LSP 플러그인을 만드는 방법을 다룹니다.
</Tip>

플러그인은 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) (LSP) 서버를 제공하여 Claude가 코드베이스 작업 중 실시간 코드 인텔리전스를 얻도록 할 수 있습니다.

LSP 통합이 제공하는 기능:

* **즉각적인 진단**: 각 편집 직후 Claude가 오류 및 경고를 즉시 확인
* **코드 탐색**: 정의로 이동(go to definition), 참조 찾기(find references), 호버 정보
* **언어 인식**: 코드 기호에 대한 타입 정보 및 문서

**위치**: 플러그인 루트의 `.lsp.json`, 또는 `plugin.json` 내의 인라인

**형식**: 언어 서버 이름을 구성으로 매핑하는 JSON 구성

**`.lsp.json` 파일 형식**:

```json theme={null}
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

**`plugin.json` 내의 인라인**:

```json theme={null}
{
  "name": "my-plugin",
  "lspServers": {
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {
        ".go": "go"
      }
    }
  }
}
```

**필수 필드:**

| 필드                  | 설명                                         |
| :-------------------- | :------------------------------------------- |
| `command`             | 실행할 LSP 바이너리 (PATH 상에 존재해야 함)  |
| `extensionToLanguage` | 파일 확장자를 언어 식별자로 매핑             |

**선택 필드:**

| 필드                    | 설명                                                                                                                                                                 |
| :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `args`                  | LSP 서버를 위한 커맨드라인 인수                                                                                                                                     |
| `transport`             | 통신 트랜스포트: `stdio` (기본값) 또는 `socket`                                                                                                                     |
| `env`                   | 서버 시작 시 설정할 환경 변수                                                                                                                                       |
| `initializationOptions` | 초기화 도중 서버로 전달되는 옵션                                                                                                                                    |
| `settings`              | `workspace/didChangeConfiguration`을 통해 전달되는 설정                                                                                                              |
| `workspaceFolder`       | 서버를 위한 워크스페이스 폴더 경로                                                                                                                                  |
| `startupTimeout`        | 서버 시작을 기다리는 최대 시간 (밀리초)                                                                                                                             |
| `shutdownTimeout`       | 정상 종료(graceful shutdown)를 기다리는 최대 시간 (밀리초). 타임아웃이 경과하면 Claude Code가 서버 프로세스를 종료함. unset 상태일 때는 타임아웃이 적용되지 않음   |
| `restartOnCrash`        | 충돌 발생 후 서버를 재시작할지 여부. 기본값은 `true`. 재시작하지 않고 충돌한 서버를 정지된 상태로 두려면 `false`로 설정                                            |
| `maxRestarts`           | 단념하기 전 최대 재시작 시도 횟수                                                                                                                                   |
| `diagnostics`           | 편집 후 Claude의 컨텍스트에 진단 정보를 푸시할지 여부 (기본값 `true`). 코드 탐색은 유지하되 자동 진단 주입을 억제하려면 `false`로 설정                              |

`restartOnCrash`와 `shutdownTimeout`은 Claude Code v2.1.205 이상이 필요합니다. v2.1.205 이전에는 구성 스키마가 두 옵션을 모두 수용했으나 둘 중 하나를 설정하면 시작 시 Claude Code가 해당 LSP 서버를 완전히 건너뛰었고 원인은 `claude --debug` 출력에서만 볼 수 있었습니다.

**동일한 확장자에 대한 다중 서버**: 활성화된 둘 이상의 LSP 서버가 `extensionToLanguage`에서 동일한 파일 확장자를 선언하는 경우, 서버가 하나의 플러그인에서 왔든 다른 플러그인에서 왔든 등록된 첫 번째 서버가 해당 확장자를 가진 파일을 처리하며 나머지는 시작되지 않습니다. `/plugin` 인터페이스는 어떤 플러그인의 서버가 활성화되었는지를 명시하는 경고를 표시합니다.

**초기화에 실패한 서버**: Claude Code는 구성이 유효하지 않은 서버(예: `command` 또는 `extensionToLanguage` 누락)를 건너뛰며, 구성된 다른 서버들은 정상 시작됩니다. 서버가 건너뛰어진 원인을 확인하려면 `claude --debug`를 실행하세요.

건너뛰어진 서버는 해당 파일 확장자를 점유하지 않으므로 동일하거나 다른 플러그인에서 동일한 확장자를 선언하는 다른 유효한 서버가 해당 파일들을 여전히 처리합니다. v2.1.205 이전에는 초기화에 실패한 서버가 여전히 확장자를 점유하여 동일한 확장자에 대한 다른 유효한 서버를 차단했었습니다.

<Warning>
  **언어 서버 바이너리를 별도로 설치해야 합니다.** LSP 플러그인은 Claude Code가 언어 서버에 연결하는 방식을 구성하지만 서버 자체를 포함하지는 않습니다. `/plugin` Errors 탭에서 `Executable not found in $PATH`가 보이면 사용 중인 언어에 필요한 바이너리를 설치하세요.
</Warning>

**사용 가능한 LSP 플러그인:**

| 플러그인            | 언어 서버                  | 설치 명령                                                                                  |
| :------------------ | :------------------------- | :----------------------------------------------------------------------------------------- |
| `pyright-lsp`       | Pyright (Python)           | `pip install pyright` 또는 `npm install -g pyright`                                        |
| `typescript-lsp`    | TypeScript Language Server | `npm install -g typescript-language-server typescript`                                     |
| `rust-analyzer-lsp` | rust-analyzer              | [rust-analyzer 설치 가이드 참조](https://rust-analyzer.github.io/manual.html#installation) |

언어 서버를 먼저 설치한 후 마켓플레이스에서 플러그인을 설치하세요.

### 모니터 (Monitors)

플러그인은 플러그인이 활성화될 때 Claude Code가 자동으로 시작하는 백그라운드 모니터를 선언할 수 있습니다. 각 모니터는 세션 수명 동안 영구 셸 명령을 실행하고 모든 stdout 줄을 알림 형태로 Claude에게 전달하므로, Claude가 직접 관찰을 시작하라는 지시를 받지 않고도 로그 항목, 상태 변경, 폴링된 이벤트에 반응할 수 있습니다.

플러그인 모니터는 [Monitor 도구](/docs/en/tools-reference#monitor-tool)와 동일한 메커니즘을 사용하며 제약 사항도 공유합니다. 대화형 CLI 세션에서만 실행되며, [훅](#hooks)과 동일한 신뢰 수준으로 샌드박싱 없이 실행되고, Monitor 도구를 사용할 수 없는 호스트에서는 건너뜁니다.

**위치**: 플러그인 루트의 `monitors/monitors.json`, 또는 plugin.json 내의 인라인

**형식**: 모니터 항목의 JSON 배열

다음 `monitors/monitors.json`은 배포 상태 엔드포인트와 로컬 오류 로그를 감시합니다:

```json theme={null}
[
  {
    "name": "deploy-status",
    "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/poll-deploy.sh",
    "description": "Deployment status changes"
  },
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log",
    "when": "on-skill-invoke:debug"
  }
]
```

인라인으로 모니터를 선언하려면 `plugin.json` 내부의 `experimental.monitors`를 동일한 배열로 설정하세요. 기본이 아닌 경로에서 로드하려면 `experimental.monitors`를 `"./config/monitors.json"`과 같은 상대 경로 문자열로 설정하세요. 모니터는 [실험적 구성 요소](#experimental-components)입니다.

**필수 필드:**

| 필드          | 설명                                                                                                                   |
| :------------ | :--------------------------------------------------------------------------------------------------------------------- |
| `name`        | 플러그인 내에서 고유한 식별자. 플러그인이 다시 로드되거나 스킬이 다시 호출될 때 중복 프로세스 방지                     |
| `command`     | 세션 작업 디렉토리에서 영구 백그라운드 프로세스로 구동되는 셸 명령                                                     |
| `description` | 감시 중인 대상에 대한 짧은 요약. 태스크 패널 및 알림 요약에 표시됨                                                    |

**선택 필드:**

| 필드    | 설명                                                                                                                                                                                                            |
| :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `when`  | 모니터가 시작되는 시점을 제어. 기본값인 `"always"`는 세션 시작 시 및 플러그인 새로고침 시에 시작함. `"on-skill-invoke:<skill-name>"`은 이 플러그인의 해당 스킬이 처음 디스패치될 때 모니터를 시작함             |

`command` 값은 경로 치환 문자인 `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`, `${CLAUDE_PROJECT_DIR}`과 환경 변수 `${ENV_VAR}`를 지원합니다. 스크립트가 플러그인의 자체 디렉토리에서 구동되어야 하는 경우 명령 앞에 `cd "${CLAUDE_PLUGIN_ROOT}" && `를 붙이세요.

모니터 `command`는 [`${user_config.*}`](#user-configuration) 값을 참조할 수 없습니다. 명령이 셸을 통해 구동되므로 Claude Code가 지목된 값을 치환하는 대신 [오류](/docs/en/errors#plugin-command-references-user-config)와 함께 모니터를 거부합니다. 모니터 프로세스는 `CLAUDE_PLUGIN_OPTION_<KEY>` 환경 변수를 수신하지 않으므로 모니터 스크립트가 자체 구성 파일에서 값을 읽도록 하세요. v2.1.207 이전에는 모니터 명령이 `${user_config.*}` 값을 치환했었습니다.

세션 도중에 플러그인을 비활성화해도 이미 구동 중인 모니터가 정지되지는 않습니다. 세션이 종료될 때 정지됩니다.

### 테마 (Themes)

플러그인은 내장 프리셋 및 사용자의 로컬 테마와 함께 `/theme`에 나타나는 색상 테마를 포함할 수 있습니다. 테마는 `base` 프리셋과 색상 토큰의 듬성듬성한(sparse) `overrides` 맵이 포함된 `themes/` 내의 JSON 파일입니다. 테마는 [실험적 구성 요소](#experimental-components)입니다.

```json theme={null}
{
  "name": "Dracula",
  "base": "dark",
  "overrides": {
    "claude": "#bd93f9",
    "error": "#ff5555",
    "success": "#50fa7b"
  }
}
```

플러그인 테마를 선택하면 사용자 구성에 `custom:<plugin-name>:<slug>`가 유지됩니다. 플러그인 테마는 읽기 전용입니다; `/theme`에서 하나에 `Ctrl+E`를 누르면 `~/.claude/themes/`로 복사되어 사용자가 사본을 편집할 수 있게 됩니다.

***

## 플러그인 설치 스코프

플러그인을 설치할 때 플러그인을 어디서 사용할 수 있는지와 누가 사용할 수 있는지를 결정하는 **스코프(scope)**를 선택합니다:

| 스코프    | 설정 파일                                       | 사용 사례                                               |
| :-------- | :---------------------------------------------- | :------------------------------------------------------ |
| `user`    | `~/.claude/settings.json`                       | 모든 프로젝트에서 사용 가능한 개인 플러그인 (기본값)    |
| `project` | `.claude/settings.json`                         | 버전 관리를 통해 공유되는 팀 플러그인                  |
| `local`   | `.claude/settings.local.json`                   | 프로젝트 전용 플러그인, gitignored                     |
| `managed` | [관리형 설정](/docs/en/settings#settings-files) | 관리형 플러그인 (읽기 전용, 업데이트만 가능)            |

플러그인은 다른 Claude Code 구성과 동일한 스코프 시스템을 사용합니다. 설치 지침 및 스코프 플래그는 [플러그인 설치](/docs/en/discover-plugins#install-plugins)를 참조하세요. 스코프에 대한 완전한 설명은 [구성 스코프](/docs/en/settings#configuration-scopes)를 참조하세요.

***

## Skills 디렉토리 플러그인

`.claude-plugin/plugin.json` 매니페스트를 포함하는 skills 디렉토리 하위의 모든 폴더는 마켓플레이스나 설치 단계 없이 다음 세션에서 `<name>@skills-dir`이라는 이름의 플러그인으로 로드됩니다. [`plugin init`](#plugin-init)으로 스캐폴딩할 수 있습니다. 마켓플레이스 설치와 달리 플러그인이 플러그인 캐시로 복사되지 않고 제자리에서 탐색됩니다.

skills 디렉토리 트리는 세 가지 다른 요소를 지원합니다:

| 보유 형태                                     | 의미                                                                                |
| :-------------------------------------------- | :---------------------------------------------------------------------------------- |
| 매니페스트가 없는 `<skills-dir>/foo/SKILL.md` | `foo`라는 이름의 일반 [스킬](/docs/en/skills)                                       |
| `<skills-dir>/foo/.claude-plugin/plugin.json` | 자체 스킬, 에이전트, 훅 등을 번들할 수 있는 `foo@skills-dir` 플러그인              |
| `<plugin>/skills/bar/SKILL.md`                | 플러그인 내부에 패키징된 `bar` 스킬                                                 |

### 플러그인이 로드되는 위치 선택

| Skills 디렉토리         | 스코프   | 로드 조건                                                                        |
| :---------------------- | :------- | :------------------------------------------------------------------------------- |
| `~/.claude/skills/`     | personal | 위치가 본인 소유이므로 모든 프로젝트에서 로드됨                                  |
| `<cwd>/.claude/skills/` | project  | 해당 폴더에 대한 작업 공간 [신뢰 대화 상자](/docs/en/settings)를 승인한 후에만 로드됨 |

Project 스코프 플러그인은 저장소에 커밋되어 이를 복제하는 모든 협업자에게 도달합니다. 해당 콘텐츠가 사용자가 아닌 저장소로부터 오기 때문에 `.claude/settings.json`을 규정하는 동일한 신뢰 게이트 후에만 로드되며, 코드를 실행하는 구성 요소는 추가로 제한됩니다:

* 선언된 MCP 서버는 프로젝트 `.mcp.json`과 동일하게 [서버별 승인](/docs/en/mcp)을 거침
* LSP 서버는 사용자가 작업 공간을 신뢰한 후에만 시작됨
* [백그라운드 모니터](#monitors)는 로드되지 않음

Personal 스코프 플러그인은 이러한 제한이 전혀 없습니다.

<Warning>
  Project 스코프 `@skills-dir` 플러그인은 Claude Code를 시작한 디렉토리의 `.claude/skills/`에서만 로드됩니다. 일반 스킬이나 명령처럼 [상위 저장소 루트로 순회](/docs/en/skills#automatic-discovery-from-parent-and-nested-directories)하지 않으므로 하위 디렉토리에서 시작하면 저장소 루트에 있는 플러그인을 놓치게 됩니다. 저장소 루트에서 시작하거나 디렉토리를 변경한 후 `/reload-plugins`를 실행하세요.
</Warning>

### skills 디렉토리 플러그인 편집, 새로고침, 비활성화

스킬의 `SKILL.md` 변경 사항은 현재 세션에 즉시 적용됩니다. `hooks/`, `.mcp.json`, `agents/`, `output-styles/` 등 플러그인의 다른 구성 요소 변경 사항은 즉시 적용되지 않습니다. 이를 반영하려면 `/reload-plugins`를 실행하거나 Claude Code를 다시 시작하세요. [실시간 변경 감지](/docs/en/skills#live-change-detection)를 참조하세요.

skills 디렉토리 플러그인의 로드를 정지하려면 해당 폴더를 삭제하거나 이름을 지정하여 비활성화하세요. 마켓플레이스에서 설치된 것이 없으므로 `uninstall` 단계가 없습니다.

```bash theme={null}
claude plugin disable my-tool@skills-dir
```

***

## 플러그인 매니페스트 스키마

`.claude-plugin/plugin.json` 파일은 플러그인의 메타데이터 및 구성을 정의합니다. 이 섹션은 지원되는 모든 필드와 옵션을 다룹니다.

매니페스트는 선택 사항입니다. 생략 시 Claude Code는 [기본 위치](#file-locations-reference)에서 구성 요소를 자동으로 탐색하고 디렉토리 이름에서 플러그인 이름을 파생합니다. 메타데이터나 커스텀 구성 요소 경로를 제공해야 할 때 매니페스트를 사용하세요.

### 전체 스키마

```json theme={null}
{
  "name": "plugin-name",
  "displayName": "Plugin Name",
  "version": "1.2.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "skills": "./custom/skills/",
  "commands": ["./custom/commands/special.md"],
  "agents": ["./custom/agents/reviewer.md"],
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "outputStyles": "./styles/",
  "lspServers": "./.lsp.json",
  "experimental": {
    "themes": "./themes/",
    "monitors": "./monitors.json"
  },
  "dependencies": [
    "helper-lib",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

### 필수 필드

매니페스트를 포함하는 경우 `name`이 유일한 필수 필드입니다.

| 필드   | 타입   | 설명                                                                                                                                                                                                                      | 예시                 |
| :----- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------- |
| `name` | string | 고유 식별자 (kebab-case, 공백 없음). [마켓플레이스 항목](/docs/en/plugin-marketplaces#plugin-entries)이 다른 이름으로 플러그인을 나열할 때 `enabledPlugins` 키와 `/plugin`이 사용하는 것은 마켓플레이스 항목 이름입니다 | `"deployment-tools"` |

이 이름은 구성 요소의 네임스페이스 지정에 사용됩니다. 예를 들어 UI에서 `plugin-dev`라는 이름의 플러그인에 포함된 에이전트 `agent-creator`는 `plugin-dev:agent-creator`로 나타납니다.

### 인식되지 않는 필드

Claude Code는 자신이 인식하지 못하는 최상위 필드를 무시합니다. `plugin.json`에 다른 생태계의 메타데이터를 유지하더라도 플러그인이 정상적으로 로드됩니다. 이를 통해 하나의 매니페스트가 VS Code나 Cursor 확장 프로그램 매니페스트, npm `package.json`, 또는 MCPB/DXT 번들 매니페스트 역할을 겸하도록 만들 수 있습니다.

`claude plugin validate`는 인식되지 않는 필드를 오류가 아닌 경고로 보고합니다. 필드가 인식되는 이름과 한두 글자 차이 나는 경우 경고가 의도했을 가능성이 높은 이름을 제안합니다. 인식되지 않는 필드 경고만 있는 플러그인은 검증을 통과하고 런타임에 로드됩니다.

유효하지 않은 타입을 가진 필드는 실패합니다. 예를 들어 문자열 배열이 아닌 단순 문자열인 `keywords` 값은 로드 오류이며 `claude plugin validate`가 오류로 보고합니다.

경고를 오류로 취급하려면 `--strict`를 전달하세요. 게시하기 전에 철자가 틀린 필드 이름이나 다른 도구의 매니페스트에서 남겨진 필드를 CI에서 포착할 때 이를 사용하세요.

```bash theme={null}
claude plugin validate ./my-plugin --strict
```

### 메타데이터 필드

| 필드             | 타입    | 설명                                                                                                                                                                                                                                                                                                                                      | 예시                                                              |
| :--------------- | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------- |
| `$schema`        | string  | 에디터 자동 완성 및 검증을 위한 JSON Schema URL. Claude Code는 로드 시점에 이 필드를 무시합니다.                                                                                                                                                                                                                                         | `"https://json.schemastore.org/claude-code-plugin-manifest.json"` |
| `displayName`    | string  | `/plugin` 피커 및 기타 UI 서페이스에 표시되는 사람이 읽을 수 있는 이름. 생략 시 `name`으로 폴백됩니다. `name`과 달리 공백 및 임의의 대소문자를 포함할 수 있습니다. 네임스페이스 지정이나 조회에는 사용되지 않습니다. Claude Code v2.1.143 이상이 필요합니다.                                                                             | `"Deployment Tools"`                                              |
| `version`        | string  | 선택 사항. 시맨틱 버전. 이를 설정하면 플러그인이 해당 버전 문자열로 고정되어 사용자가 버전 상승 시에만 업데이트를 받습니다. 생략 시 Claude Code는 git 커밋 SHA로 폴백하므로 모든 커밋이 새 버전으로 취급됩니다. 마켓플레이스 항목에도 설정되어 있는 경우 `plugin.json`이 우선합니다. [버전 관리](#version-management)를 참조하세요.   | `"2.1.0"`                                                         |
| `description`    | string  | 플러그인 목적에 대한 짧은 설명                                                                                                                                                                                                                                                                                                            | `"Deployment automation tools"`                                   |
| `author`         | object  | 작성자 정보                                                                                                                                                                                                                                                                                                                               | `{"name": "Dev Team", "email": "dev@company.com"}`                |
| `homepage`       | string  | 문서 URL                                                                                                                                                                                                                                                                                                                                  | `"https://docs.example.com"`                                      |
| `repository`     | string  | 소스 코드 URL                                                                                                                                                                                                                                                                                                                             | `"https://github.com/user/plugin"`                                |
| `license`        | string  | 라이선스 식별자                                                                                                                                                                                                                                                                                                                           | `"MIT"`, `"Apache-2.0"`                                           |
| `keywords`       | array   | 탐색 태그                                                                                                                                                                                                                                                                                                                                 | `["deployment", "ci-cd"]`                                         |
| `defaultEnabled` | boolean | 사용자가 설정을 하지 않았을 때 플러그인이 활성화된 상태로 시작할지 여부. 기본값은 `true`. [기본 활성화 여부](#default-enablement)를 참조하세요. Claude Code v2.1.154 이상이 필요합니다.                                                                                                                                                    | `false`                                                           |

### 기본 활성화 여부

사용자가 옵트인해야 하는 비용이나 범주를 추가하는 플러그인(외부 서비스에 연결하는 플러그인 등)의 경우 `plugin.json`에 `defaultEnabled: false`를 설정하여 비활성화된 상태로 설치되도록 배포하세요. 사용자는 `claude plugin enable <plugin>`이나 `/plugin` 인터페이스로 이를 켭니다. Claude Code v2.1.154 이상이 필요합니다. 이전 버전은 필드를 무시하고 설치 시 플러그인을 활성화합니다.

`defaultEnabled`는 다른 어떤 것도 플러그인 상태를 결정하지 않았을 때 적용되는 폴백입니다. 두 가지가 이보다 우선합니다:

* **사용자의 설정**: 임의의 설정 스코프에서 `enabledPlugins` 내의 플러그인 항목. 한 번 작성되면 플러그인 업데이트 및 재설치 전반에 걸쳐 유지되므로 추후 릴리스에서 `defaultEnabled`를 변경해도 기존 사용자 상태가 뒤집히지는 않습니다.
* **의존성 요구사항**: 플러그인이 활성화된 다른 플러그인에 의해 필요해질 때 Claude Code는 설치 또는 활성화 시점에 해당 플러그인에 대해 `true`를 씁니다. 이를 통해 명시적 설정이 부여되므로 자체 기본값이 더 이상 적용되지 않습니다. [의존성을 가진 플러그인 활성화 또는 비활성화](/docs/en/plugin-dependencies#enable-or-disable-a-plugin-with-dependencies)를 참조하세요.

동일한 필드가 플러그인의 마켓플레이스 항목에 나타날 수 있으며, 거기서 `plugin.json` 내부의 값보다 우선 적용됩니다. [선택적 플러그인 필드](/docs/en/plugin-marketplaces#optional-plugin-fields)를 참조하세요.

### 구성 요소 경로 필드

| 필드                    | 타입                  | 설명                                                                                                                                                                          | 예시                                                 |
| :---------------------- | :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------- |
| `skills`                | string\|array         | `<name>/SKILL.md`를 포함하는 커스텀 스킬 디렉토리. 기본 `skills/` 스캔에 추가됨. 마켓플레이스 루트 예외는 [경로 동작 규칙](#path-behavior-rules) 참조                       | `"./custom/skills/"`                                 |
| `commands`              | string\|array         | 커스텀 단일 `.md` 스킬 파일 또는 디렉토리 (기본 `commands/`를 대체함)                                                                                                         | `"./custom/cmd.md"` 또는 `["./cmd1.md"]`             |
| `agents`                | string\|array         | 커스텀 에이전트 파일 (기본 `agents/`를 대체함)                                                                                                                                | `"./custom/agents/reviewer.md"`                      |
| `hooks`                 | string\|array\|object | 훅 구성 경로 또는 인라인 구성                                                                                                                                                 | `"./my-extra-hooks.json"`                            |
| `mcpServers`            | string\|array\|object | MCP 구성 경로 또는 인라인 구성                                                                                                                                                | `"./my-extra-mcp-config.json"`                       |
| `outputStyles`          | string\|array         | 커스텀 출력 스타일 파일/디렉토리 (기본 `output-styles/`를 대체함)                                                                                                             | `"./styles/"`                                        |
| `lspServers`            | string\|array\|object | 코드 인텔리전스(정의로 이동, 참조 찾기 등)를 위한 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) 구성                                     | `"./.lsp.json"`                                      |
| `experimental.themes`   | string\|array         | 색상 테마 파일/디렉토리 (기본 `themes/`를 대체함). [테마](#themes) 참조                                                                                                        | `"./themes/"`                                        |
| `experimental.monitors` | string\|array         | 플러그인이 활성화되어 있을 때 자동으로 시작되는 백그라운드 [Monitor](/docs/en/tools-reference#monitor-tool) 구성. [모니터](#monitors) 참조                                   | `"./monitors.json"`                                  |
| `userConfig`            | object                | 활성화 시점에 사용자에게 프롬프트로 요청하는 설정 가능 값들. [사용자 구성](#user-configuration) 참조                                                                          | 아래 참조                                            |
| `channels`              | array                 | 메시지 주입을 위한 채널 선언 (Telegram, Slack, Discord 스타일). [채널](#channels) 참조                                                                                        | 아래 참조                                            |
| `dependencies`          | array                 | 이 플러그인이 필요로 하는 다른 플러그인(선택적 semver 버전 제약 포함). [플러그인 의존성 버전 제약](/docs/en/plugin-dependencies) 참조                                          | `[{ "name": "secrets-vault", "version": "~2.1.0" }]` |

### 실험적 구성 요소

`experimental` 키 아래의 구성 요소인 `themes`와 `monitors`는 안정화되는 동안 릴리스 간에 변경될 수 있는 매니페스트 스키마를 가집니다. 이를 어디에 선언하는가는 별개의 마이그레이션 사항입니다: 최상위 선언도 여전히 작동하며 `claude plugin validate`가 경고하고, 추후 릴리스에서는 `experimental.*`을 요구하게 됩니다.

### 사용자 구성 (userConfig)

`userConfig` 필드는 플러그인이 활성화될 때 Claude Code가 사용자에게 프롬프트로 물어보는 값들을 선언합니다. 사용자가 `settings.json`을 직접 수동 편집하도록 요구하는 대신 이를 사용하세요.

```json theme={null}
{
  "userConfig": {
    "api_endpoint": {
      "type": "string",
      "title": "API endpoint",
      "description": "Your team's API endpoint"
    },
    "api_token": {
      "type": "string",
      "title": "API token",
      "description": "API authentication token",
      "sensitive": true
    }
  }
}
```

키는 반드시 유효한 식별자여야 합니다. 각 옵션은 다음 필드들을 지원합니다:

| 필드          | 필수     | 설명                                                                                     |
| :------------ | :------- | :--------------------------------------------------------------------------------------- |
| `type`        | 예       | `string`, `number`, `boolean`, `directory`, `file` 중 하나                               |
| `title`       | 예       | 구성 대화 상자에 표시되는 레이블                                                         |
| `description` | 예       | 필드 아래에 표시되는 도움말 텍스트                                                       |
| `sensitive`   | 아니오   | `true`인 경우 입력을 마스킹하고 `settings.json` 대신 보안 저장소에 값을 저장함         |
| `required`    | 아니오   | `true`인 경우 필드가 비어 있을 때 검증 실패                                              |
| `default`     | 아니오   | 사용자가 아무것도 제공하지 않을 때 사용되는 값                                           |
| `multiple`    | 아니오   | `string` 타입의 경우 문자열 배열을 허용함                                                |
| `min` / `max` | 아니오   | `number` 타입에 대한 범위 제약                                                           |

각 값은 MCP 및 LSP 서버 구성과 훅 명령에서 `${user_config.KEY}`로 치환될 수 있습니다. 민감하지 않은 값은 스킬 및 에이전트 내용에서도 치환될 수 있습니다. 모든 값은 `<KEY>`가 옵션 키의 대문자인 `CLAUDE_PLUGIN_OPTION_<KEY>` 환경 변수로 훅 프로세스에 내보내집니다.

셸에서 구동되는 필드들은 `${user_config.*}`를 거부합니다: 셸 명령에 구성된 값을 치환하면 해당 값이 포함하는 무엇이든 셸이 구동하게 되므로, 구성 요소가 [오류](/docs/en/errors#plugin-command-references-user-config)와 함께 실패하게 됩니다. 거부되는 각 필드는 값을 전달하는 대체 방식을 제공합니다:

| 거부되는 필드                                                                | 값을 전달하는 방법                                                                                                               |
| :--------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------- |
| 셸 형식 훅 명령                                                              | `args`와 함께 [exec 형식](/docs/en/hooks#exec-form-and-shell-form)을 사용하거나, 훅 환경에서 `CLAUDE_PLUGIN_OPTION_<KEY>`를 읽음|
| [모니터](#monitors) 명령                                                     | 스크립트 내부의 구성 파일에서 값을 읽음                                                                                           |
| MCP [`headersHelper`](/docs/en/mcp#use-dynamic-headers-for-custom-authentication) | 스크립트 내부의 구성 파일에서 값을 읽음                                                                                           |

v2.1.207 이전에는 이러한 필드들이 `${user_config.KEY}` 값을 치환했었습니다; 이에 의존하던 플러그인을 업데이트하세요.

민감하지 않은 값들은 `settings.json` 내의 [`pluginConfigs`](/docs/en/settings#pluginconfigs) 키 아래에 `pluginConfigs[<plugin-id>].options` 형태로 저장됩니다. Claude Code는 사용자 설정에 이 키를 작성하고 사용자 설정, `--settings` 플래그, 관리형 설정에서만 다시 읽어옵니다; 프로젝트의 `.claude/settings.json`이나 `.claude/settings.local.json` 항목은 무시됩니다. v2.1.207 이전에는 Claude Code가 프로젝트 및 로컬 설정도 읽었었습니다.

민감한 값은 macOS Keychain으로 가거나, 지원되는 키체인을 사용할 수 없는 플랫폼에서는 `~/.claude/.credentials.json`으로 갑니다. 키체인 저장소는 OAuth 토큰과 공유되며 총 제한이 약 2KB이므로 민감한 값은 작게 유지하세요.

### 채널 (Channels)

`channels` 필드를 통해 플러그인은 대화에 콘텐츠를 주입하는 하나 이상의 메시지 채널을 선언할 수 있습니다. 각 채널은 플러그인이 제공하는 MCP 서버에 바인딩됩니다.

```json theme={null}
{
  "channels": [
    {
      "server": "telegram",
      "userConfig": {
        "bot_token": {
          "type": "string",
          "title": "Bot token",
          "description": "Telegram bot token",
          "sensitive": true
        },
        "owner_id": {
          "type": "string",
          "title": "Owner ID",
          "description": "Your Telegram user ID"
        }
      }
    }
  ]
}
```

`server` 필드는 필수이며 플러그인의 `mcpServers` 키와 일치해야 합니다. 선택적인 채널별 `userConfig`는 최상위 필드와 동일한 스키마를 사용하여 플러그인이 활성화될 때 봇 토큰이나 소유자 ID를 프롬프트로 요청하게 합니다.

### 경로 동작 규칙

커스텀 경로가 플러그인의 기본 디렉토리를 대체하는지 아니면 확장하는지는 필드에 따라 다릅니다:

* **기본을 대체함**: `commands`, `agents`, `outputStyles`, `experimental.themes`, `experimental.monitors`. 예를 들어 매니페스트가 `commands`를 지정하면 기본 `commands/` 디렉토리는 스캔되지 않습니다. 기본을 유지하면서 추가하려면 명시적으로 나열하세요: `"commands": ["./commands/", "./extras/"]`
* **기본에 추가됨**: `skills`. 기본 `skills/` 디렉토리는 항상 스캔되며 `skills`에 나열된 디렉토리가 옆에 함께 로드됩니다. 예외: [`source`가 마켓플레이스 루트로 해석되는 마켓플레이스 항목](/docs/en/plugin-marketplaces#advanced-plugin-entries)의 경우 특정 하위 디렉토리를 선언하면 기본 `skills/` 스캔을 대체합니다.
* **자체 병합 규칙**: [훅](#hooks), [MCP 서버](#mcp-servers), [LSP 서버](#lsp-servers). 여러 소스가 결합하는 방식은 각 섹션을 참조하세요.

플러그인에 기본 폴더와 일치하는 매니페스트 키가 둘 다 있는 경우, Claude Code v2.1.140 이상은 `claude plugin list` 및 `/plugin` 상세 뷰에서 무시된 폴더에 대한 경고를 출력합니다. 플러그인은 매니페스트 경로를 사용하여 계속 로드됩니다. 매니페스트 키가 `"commands": ["./commands/deploy.md"]`처럼 기본 폴더 내부를 가리킬 때는 경로가 해당 폴더를 명시적으로 지목하므로 Claude Code가 경고를 출력하지 않습니다.

모든 경로 필드에 대해:

* 모든 경로는 플러그인 루트에 상대적이어야 하며 `./`로 시작해야 함
* 커스텀 경로의 구성 요소는 동일한 명명 및 네임스페이스 규칙을 사용함
* 여러 경로는 배열로 지정할 수 있음
* 스킬 경로가 `SKILL.md`를 직접 포함하는 디렉토리를 가리킬 때(예: 플러그인 루트를 가리키는 `"skills": ["./"]`), `SKILL.md` 내부의 프론트매터 `name` 필드가 스킬의 호출 이름을 결정합니다. 이는 설치 디렉토리에 관계없이 안정적인 이름을 제공합니다. 프론트매터에 `name`이 설정되어 있지 않은 경우 디렉토리 기준 이름(basename)이 폴백으로 사용됩니다.

루트에 `SKILL.md`가 있고 `skills/` 하위 디렉토리가 없으며 `skills` 매니페스트 필드도 없는 플러그인은 Claude Code v2.1.142 이상에서 단일 스킬 플러그인으로 자동 로드됩니다. 이 레이아웃을 위해 `plugin.json`에 `"skills": ["./"]`를 설정할 필요가 없습니다. 스킬의 호출 이름은 위와 동일한 규칙을 따릅니다: 프론트매터 `name` 필드, 또는 디렉토리 기준 이름이 폴백.

**경로 예시**:

```json theme={null}
{
  "commands": [
    "./specialized/deploy.md",
    "./utilities/batch-process.md"
  ],
  "agents": [
    "./custom-agents/reviewer.md",
    "./custom-agents/tester.md"
  ]
}
```

### 환경 변수

Claude Code는 경로 참조를 위해 세 변수를 제공합니다:

| 변수                    | 해석 위치                                                                                                  | 사용 용도                                                                                                |
| :---------------------- | :--------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------- |
| `${CLAUDE_PLUGIN_ROOT}` | 플러그인 설치 디렉토리의 절대 경로                                                                         | 플러그인과 함께 번들로 포함된 스크립트, 바이너리, 구성 파일                                             |
| `${CLAUDE_PLUGIN_DATA}` | 첫 참조 시 생성되며 플러그인 업데이트를 거쳐 지속되는 [영구 디렉토리](#persistent-data-directory)          | `node_modules`나 Python 가상 환경 등 설치된 의존성, 생성된 코드, 캐시                                    |
| `${CLAUDE_PROJECT_DIR}` | 프로젝트 루트                                                                                              | 프로젝트 로컬 스크립트 및 구성 파일                                                                      |

세 변수 모두 훅 프로세스 및 MCP/LSP 서버 서브프로세스에 환경 변수로 내보내집니다. 치환 문자가 인라인으로 반영되는 필드는 플러그인 구성 요소에 따라 다릅니다:

| 플러그인 구성 요소              | 플레이스홀더가 해석되는 필드                 |
| :------------------------------ | :------------------------------------------ |
| 스킬 및 에이전트 내용           | 플레이스홀더가 나타나는 모든 곳             |
| 훅 및 모니터 명령               | 플레이스홀더가 나타나는 모든 곳             |
| MCP `stdio` 서버                | `command`, `args`, `env`                    |
| MCP `http`, `sse`, `ws` 서버    | `url`, `headers`, `headersHelper`           |
| LSP 서버                        | `command`, `args`, `env`, `workspaceFolder` |

훅 명령에서는 각 경로가 따옴표 없이 하나의 인수로 전달되도록 `args`가 포함된 [exec 형식](/docs/en/hooks#exec-form-and-shell-form)을 사용하세요. 셸 형식 훅 및 모니터 명령에서는 `"${CLAUDE_PROJECT_DIR}/scripts/server.sh"`와 같이 변수를 큰따옴표로 감싸세요. 다음 셸 형식 훅은 플러그인과 번들된 스크립트를 구동합니다:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/process.sh"
          }
        ]
      }
    ]
  }
}
```

`${CLAUDE_PLUGIN_ROOT}`는 플러그인이 업데이트될 때 변경됩니다. 이전 버전 디렉토리가 정리 전까지 약 2주 동안 디스크에 유지되지만, 일시적인 공간으로 취급하고 거기에 상태를 기록하지 마세요.

세션 도중에 플러그인이 업데이트되어도 훅 명령, 모니터, MCP 서버, LSP 서버는 이전 버전의 경로를 계속 사용합니다. 훅, MCP 서버, LSP 서버를 새 경로로 전환하려면 `/reload-plugins`를 실행하세요; 모니터는 세션 재시작이 필요합니다.

MCP 서버는 런타임 시 세션의 작업 디렉토리를 읽기 위해 `roots/list` 요청을 호출할 수도 있습니다. [roots/list가 반환하는 내용과 Claude Code가 서버에 변경을 알리는 시점](/docs/en/mcp#option-3-add-a-local-stdio-server)을 참조하세요.

#### 영구 데이터 디렉토리 (Persistent data directory)

`${CLAUDE_PLUGIN_DATA}` 디렉토리는 `~/.claude/plugins/data/{id}/`로 해석되며, 여기서 `{id}`는 `a-z`, `A-Z`, `0-9`, `_`, `-` 외의 문자가 `-`로 대체된 플러그인 식별자입니다. `formatter@my-marketplace`로 설치된 플러그인의 경우 디렉토리는 `~/.claude/plugins/data/formatter-my-marketplace/`입니다.

흔한 사용 사례는 언어 의존성을 한 번 설치하고 여러 세션 및 플러그인 업데이트에 걸쳐 재사용하는 것입니다. 데이터 디렉토리가 단일 플러그인 버전보다 길게 유지되므로 디렉토리 존재 여부만으로는 업데이트가 플러그인의 의존성 매니페스트를 변경했을 때를 감지할 수 없습니다. 권장 패턴은 번들된 매니페스트를 데이터 디렉토리의 사본과 비교하여 다를 때 다시 설치하는 것입니다.

이 `SessionStart` 훅은 첫 실행 시 및 플러그인 업데이트에 변경된 `package.json`이 포함될 때마다 `node_modules`를 설치합니다:

```json theme={null}
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "diff -q \"${CLAUDE_PLUGIN_ROOT}/package.json\" \"${CLAUDE_PLUGIN_DATA}/package.json\" >/dev/null 2>&1 || (cd \"${CLAUDE_PLUGIN_DATA}\" && cp \"${CLAUDE_PLUGIN_ROOT}/package.json\" . && npm install) || rm -f \"${CLAUDE_PLUGIN_DATA}/package.json\""
          }
        ]
      }
    ]
  }
}
```

`diff`는 저장된 사본이 없거나 번들된 것과 다를 때 0이 아닌 상태로 종료되어, 첫 실행과 의존성을 변경하는 업데이트를 모두 처리합니다. `npm install`이 실패하면 끝의 `rm`이 복사된 매니페스트를 제거하여 다음 세션이 다시 시도하도록 합니다.

`${CLAUDE_PLUGIN_ROOT}`에 번들된 스크립트는 영구 지속되는 `node_modules`에 대해 다음과 같이 구동될 수 있습니다:

```json theme={null}
{
  "mcpServers": {
    "routines": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"],
      "env": {
        "NODE_PATH": "${CLAUDE_PLUGIN_DATA}/node_modules"
      }
    }
  }
}
```

데이터 디렉토리는 플러그인이 설치된 마지막 스코프에서 언인스톨될 때 자동으로 삭제됩니다. `/plugin` 인터페이스는 디렉토리 크기를 보여주고 삭제 전 프롬프트를 띄웁니다. CLI는 기본적으로 삭제하며; 보존하려면 [`--keep-data`](#plugin-uninstall)를 전달하세요.

***

## 플러그인 캐싱 및 파일 해석

플러그인은 다음 두 가지 방식 중 하나로 지정됩니다:

* 세션 기간 동안 `claude --plugin-dir` 또는 `claude --plugin-url`을 통한 지정.
* 향후 세션을 위해 설치된 마켓플레이스를 통한 지정.

보안 및 검증 목적으로 Claude Code는 *마켓플레이스* 플러그인을 제자리에서 사용하지 않고 사용자의 로컬 **플러그인 캐시** (`~/.claude/plugins/cache`)로 복사합니다. 이 동작을 이해하는 것은 외부 파일을 참조하는 플러그인을 개발할 때 중요합니다.

설치된 각 버전은 캐시 내의 별도 디렉토리입니다. 플러그인을 업데이트하거나 언인스톨하면 이전 버전 디렉토리가 고아(orphaned) 상태로 표시되고 14일 후에 자동으로 제거됩니다. 유예 기간을 통해 이전 버전을 이미 로드한 동시 Claude Code 세션이 오류 없이 계속 구동될 수 있게 합니다.

Claude의 Glob 및 Grep 도구는 검색 중 고아 버전 디렉토리를 건너뛰므로 파일 검색 결과에 오래된 플러그인 코드가 포함되지 않습니다.

### 경로 순회 제약 사항

설치된 플러그인은 디렉토리 외부의 파일을 참조할 수 없습니다. 외부 파일은 캐시로 복사되지 않으므로 플러그인 루트 외부를 순회하는 경로(예: `../shared-utils`)는 설치 후 작동하지 않습니다.

### 심볼릭 링크로 마켓플레이스 내 파일 공유하기

플러그인이 동일한 마켓플레이스의 다른 부분과 파일을 공유해야 하는 경우 플러그인 디렉토리 내에 심볼릭 링크를 생성할 수 있습니다. 플러그인이 캐시로 복사될 때 심볼릭 링크가 처리되는 방식은 타깃이 해석되는 위치에 따라 다릅니다:

* **플러그인의 자체 디렉토리 내부:** 심볼릭 링크가 캐시에 상대 심볼릭 링크로 보존되므로 런타임 시 복사된 타깃으로 계속 해석됩니다.
* **동일한 마켓플레이스 내부의 다른 곳:** 심볼릭 링크가 역참조(dereferenced)됩니다. 타깃의 내용이 그 위치의 캐시로 복사되어 들어갑니다. 이를 통해 메타 플러그인의 `skills/` 디렉토리가 마켓플레이스의 다른 플러그인이 정의한 스킬을 링크할 수 있게 됩니다.
* **마켓플레이스 외부:** 보안을 위해 심볼릭 링크가 건너뛰어집니다. 이는 플러그인이 시스템 경로와 같은 임의의 호스트 파일을 캐시로 끌어오는 것을 방지합니다.

`--plugin-dir`이나 로컬 경로로 설치된 플러그인의 경우 플러그인 자체 디렉토리 내부로 해석되는 심볼릭 링크만 보존됩니다. 다른 모든 심볼릭 링크는 건너뛰어집니다.

다음 명령은 마켓플레이스 플러그인 내부에서 형제 플러그인이 정의한 공유 스킬로의 링크를 생성합니다. Windows에서는 관리자 권한 명령 프롬프트에서 `mklink /D`를 사용하거나 개발자 모드를 활성화하세요:

```bash theme={null}
ln -s ../../shared-plugin/skills/foo ./skills/foo
```

이는 캐싱 시스템의 보안 이점을 유지하면서 유연성을 제공합니다.

***

## 플러그인 디렉토리 구조

### 표준 플러그인 레이아웃

완전한 구조의 플러그인은 다음 레이아웃을 따릅니다:

```text theme={null}
enterprise-plugin/
├── .claude-plugin/           # 메타데이터 디렉토리 (선택 사항)
│   └── plugin.json             # 플러그인 매니페스트
├── skills/                   # 스킬 (Skills)
│   ├── code-reviewer/
│   │   └── SKILL.md
│   └── pdf-processor/
│       ├── SKILL.md
│       └── scripts/
├── commands/                 # 단일 .md 파일 형태의 스킬
│   ├── status.md
│   └── logs.md
├── agents/                   # Subagent 정의
│   ├── security-reviewer.md
│   ├── performance-tester.md
│   └── compliance-checker.md
├── output-styles/            # 출력 스타일 정의
│   └── terse.md
├── themes/                   # 색상 테마 정의
│   └── dracula.json
├── monitors/                 # 백그라운드 모니터 구성
│   └── monitors.json
├── hooks/                    # 훅 구성
│   ├── hooks.json           # 메인 훅 구성
│   └── security-hooks.json  # 추가 훅
├── bin/                      # PATH에 추가되는 플러그인 실행 파일들
│   └── my-tool               # 플러그인이 활성화된 동안 Bash 도구에서 순수 명령으로 호출 가능
├── settings.json            # 플러그인의 기본 설정
├── .mcp.json                # MCP 서버 정의
├── .lsp.json                # LSP 서버 구성
├── scripts/                 # 훅 및 유틸리티 스크립트
│   ├── security-scan.sh
│   ├── format-code.py
│   └── deploy.js
├── LICENSE                  # 라이선스 파일
└── CHANGELOG.md             # 버전 히스토리
```

<Warning>
  `.claude-plugin/` 디렉토리는 `plugin.json` 파일을 포함합니다. 다른 모든 디렉토리(commands/, agents/, skills/, output-styles/, themes/, monitors/, hooks/)는 `.claude-plugin/` 내부가 아닌 플러그인 루트에 위치해야 합니다.
</Warning>

플러그인 루트의 `CLAUDE.md` 파일은 프로젝트 컨텍스트로 로드되지 않습니다. 플러그인은 CLAUDE.md가 아닌 스킬, 에이전트, 훅을 통해 컨텍스트를 기여합니다. Claude의 컨텍스트로 로드되는 지침을 배포하려면 [스킬](#skills)에 추가하세요.

### 파일 위치 참조

| 구성 요소         | 기본 위치                    | 목적                                                                                                                                                                                      |
| :---------------- | :--------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **매니페스트**    | `.claude-plugin/plugin.json` | 플러그인 메타데이터 및 구성 (선택 사항)                                                                                                                                                  |
| **스킬 (Skills)** | `skills/`                    | `<name>/SKILL.md` 구조의 스킬                                                                                                                                                             |
| **명령 (Commands)**| `commands/`                  | 단일 마크다운 파일 형태의 스킬. 새 플러그인에는 `skills/` 사용 권장                                                                                                                       |
| **에이전트**      | `agents/`                    | Subagent 마크다운 파일                                                                                                                                                                   |
| **출력 스타일**   | `output-styles/`             | 출력 스타일 정의                                                                                                                                                                          |
| **테마**          | `themes/`                    | 색상 테마 정의                                                                                                                                                                            |
| **훅**            | `hooks/hooks.json`           | 훅 구성                                                                                                                                                                                   |
| **MCP 서버**      | `.mcp.json`                  | MCP 서버 정의                                                                                                                                                                             |
| **LSP 서버**      | `.lsp.json`                  | 언어 서버 구성                                                                                                                                                                            |
| **모니터**        | `monitors/monitors.json`     | 백그라운드 모니터 구성                                                                                                                                                                    |
| **실행 파일**     | `bin/`                       | Bash 도구의 `PATH`에 추가되는 실행 파일들. 이 안의 파일들은 플러그인이 활성화되어 있는 동안 임의의 Bash 도구 호출 시 순수 명령으로 호출 가능                                              |
| **설정**          | `settings.json`              | 플러그인이 활성화될 때 적용되는 기본 구성. 현재 [`agent`](/docs/en/sub-agents) 및 [`subagentStatusLine`](/docs/en/statusline#subagent-status-lines) 키만 지원됨                             |

***

## CLI 명령 참조

Claude Code는 스크립팅 및 자동화에 유용한 비대화형 플러그인 관리를 위한 CLI 명령을 제공합니다.

### plugin init

`~/.claude/skills/<name>/`에 새 플러그인을 스캐폴딩합니다. 다음 Claude Code 세션에서 별도 설치 단계 없이 `<name>@skills-dir`로 자동 로드되어 `/plugin` 및 `claude plugin list`에 나타납니다.

스코프 및 신뢰 요구사항은 [Skills 디렉토리 플러그인](#skills-directory-plugins)을 참조하세요.

```bash theme={null}
claude plugin init <name> [options]
```

**인수:**

* `<name>`: 플러그인 이름. 스킬 네임스페이스 및 `~/.claude/skills/` 하위의 디렉토리 이름이 되므로 공백이나 경로 구분자를 포함할 수 없음.

**옵션:**

| 옵션                     | 설명                                                                                                                | 기본값                 |
| :----------------------- | :------------------------------------------------------------------------------------------------------------------ | :--------------------- |
| `--description <text>`   | 매니페스트 설명                                                                                                     |                        |
| `--author <name>`        | 작성자 이름                                                                                                         | `git config user.name` |
| `--author-email <email>` | 작성자 이메일                                                                                                       | `git config user.email`|
| `--with <components...>` | 구성 요소 폴더도 함께 스캐폴딩. 유효한 값: `skills`, `agents`, `hooks`, `mcp`, `lsp`, `output-style`, `channel`   |                        |
| `-f, --force`            | 타깃 위치의 기존 `.claude-plugin/`을 덮어씀                                                                         |                        |
| `-h, --help`             | 명령 도움말 표시                                                                                                    |                        |

**별칭:** `new`

각 `--with` 값은 바로 편집할 수 있는 시작 파일을 해당 구성 요소에 대해 추가합니다:

| 구성 요소      | 스캐폴딩되는 내용                                                                                         |
| :------------- | :-------------------------------------------------------------------------------------------------------- |
| `skills`       | 기본 스킬과 함께 네임스페이스가 지정된 추가 `<name>:example` 스킬                                         |
| `agents`       | `agents/` subagent 정의                                                                                   |
| `hooks`        | 예제 이벤트 핸들러가 포함된 `hooks/hooks.json`                                                            |
| `mcp`          | HTTP 및 stdio 서버 예제가 포함된 `.mcp.json`                                                              |
| `lsp`          | `.lsp.json` 언어 서버 예제                                                                                |
| `output-style` | 플러그인이 활성화되어 있는 동안 자동으로 적용되는 `output-styles/<name>.md`                               |
| `channel`      | MCP 기반 [채널](/docs/en/channels): stdio 서버(`server.ts`), `.mcp.json`, `package.json`                  |

스캐폴딩된 플러그인은 마켓플레이스 대신 `@skills-dir` 소스를 사용합니다. 관리자는 `strictKnownMarketplaces`를 통해 또는 [관리형 설정](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)의 `blockedMarketplaces`에 `{"source": "skills-dir"}`을 추가하여 이 소스를 차단할 수 있습니다. 차단되면 `plugin init`이 파일을 쓰기 전에 실패합니다.

**예시:**

```bash theme={null}
# 최소 플러그인 스캐폴딩
claude plugin init my-helper

# skill 및 hook 폴더와 함께 스캐폴딩
claude plugin init my-helper --with skills hooks

# 기존 스캐폴딩 덮어쓰기
claude plugin init my-helper --force
```

### plugin install

사용 가능한 마켓플레이스에서 플러그인을 설치합니다.

```bash theme={null}
claude plugin install <plugin> [options]
```

**인수:**

* `<plugin>`: 특정 마켓플레이스의 경우 플러그인 이름 또는 `plugin-name@marketplace-name`

**옵션:**

| 옵션                   | 설명                                                                                                         | 기본값 |
| :--------------------- | :----------------------------------------------------------------------------------------------------------- | :----- |
| `-s, --scope <scope>`  | 설치 스코프: `user`, `project`, `local`                                                                      | `user` |
| `--config <key=value>` | 플러그인 매니페스트에 선언된 [`userConfig`](#user-configuration) 옵션 설정. 여러 옵션을 설정하려면 플래그를 반복 |        |
| `-h, --help`           | 명령 도움말 표시                                                                                             |        |

스코프에 따라 설치된 플러그인이 추가되는 설정 파일이 결정됩니다. 예를 들어 `--scope project`는 `.claude/settings.json` 내의 `enabledPlugins`에 작성되어 프로젝트 저장소를 복제하는 모든 사람에게 플러그인을 제공합니다.

**예시:**

```bash theme={null}
# user 스코프에 설치 (기본값)
claude plugin install formatter@my-marketplace

# project 스코프에 설치 (팀과 공유)
claude plugin install formatter@my-marketplace --scope project

# local 스코프에 설치 (gitignored)
claude plugin install formatter@my-marketplace --scope local
```

### plugin uninstall

설치된 플러그인을 제거합니다.

```bash theme={null}
claude plugin uninstall <plugin> [options]
```

**인수:**

* `<plugin>`: 플러그인 이름 또는 `plugin-name@marketplace-name`

**옵션:**

| 옵션                | 설명                                                                                                                | 기본값 |
| :------------------ | :------------------------------------------------------------------------------------------------------------------ | :----- |
| `-s, --scope <scope>`| 제거할 스코프: `user`, `project`, `local`                                                                          | `user` |
| `--keep-data`       | 플러그인의 [영구 데이터 디렉토리](#persistent-data-directory) 보존                                                  |        |
| `--prune`           | 다른 플러그인에서 필요로 하지 않는 자동 설치된 의존성도 함께 제거. [plugin prune](#plugin-prune) 참조             |        |
| `-y, --yes`         | `--prune` 확인 프롬프트 건너뛰기. stdin 또는 stdout이 TTY가 아닐 때 필요                                            |        |
| `-h, --help`        | 명령 도움말 표시                                                                                                    |        |

**별칭:** `remove`, `rm`

기본적으로 남아 있는 마지막 스코프에서 언인스톨하면 플러그인의 `${CLAUDE_PLUGIN_DATA}` 디렉토리도 삭제됩니다. 새 버전을 테스트한 후 재설치하는 경우처럼 데이터 디렉토리를 보존하려면 `--keep-data`를 사용하세요.

<Note>
  서로 다른 마켓플레이스에서 설치된 플러그인들이 이름을 공유할 때, `plugin-name@marketplace-name` 형식은 지목된 마켓플레이스의 플러그인만을 언인스톨합니다. v2.1.212 이전에는 정교화된 형식이 다른 마켓플레이스의 동일한 이름의 플러그인을 매칭하여 언인스톨할 수 있었습니다.
</Note>

### plugin prune

설치된 어떤 플러그인에서도 더 이상 필요로 하지 않는 자동 설치된 플러그인 의존성을 제거합니다. 다른 플러그인의 [`dependencies`](/docs/en/plugin-dependencies) 필드를 충족하기 위해 Claude Code가 가져온 의존성이 제거되며; 직접 설치한 플러그인은 절대 손대지 않습니다.

```bash theme={null}
claude plugin prune [options]
```

**옵션:**

| 옵션                | 설명                                                               | 기본값 |
| :------------------ | :----------------------------------------------------------------- | :----- |
| `-s, --scope <scope>`| 정리할 스코프: `user`, `project`, `local`                          | `user` |
| `--dry-run`         | 아무것도 제거하지 않고 제거될 항목 목록 출력                       |        |
| `-y, --yes`         | 확인 프롬프트 건너뛰기. stdin 또는 stdout이 TTY가 아닐 때 필요      |        |
| `-h, --help`        | 명령 도움말 표시                                                   |        |

**별칭:** `autoremove`

이 명령은 고아 의존성을 나열하고 제거하기 전에 확인을 요청합니다. 한 번의 단계로 플러그인을 제거하고 해당 의존성을 정리하려면 `claude plugin uninstall <plugin> --prune`을 실행하세요.

<Note>
  `claude plugin prune`은 Claude Code v2.1.121 이상이 필요합니다.
</Note>

### plugin enable

비활성화된 플러그인을 활성화합니다. 플러그인이 [의존성](/docs/en/plugin-dependencies)을 선언한 경우 Claude Code가 동일한 스코프에서 이들을 전이적으로(transitively) 활성화하며, 의존성이 설치되어 있지 않으면 명령이 실패합니다.

```bash theme={null}
claude plugin enable <plugin> [options]
```

**인수:**

* `<plugin>`: 플러그인 이름 또는 `plugin-name@marketplace-name`

**옵션:**

| 옵션                | 설명                                                                                                               | 기본값     |
| :------------------ | :----------------------------------------------------------------------------------------------------------------- | :--------- |
| `-s, --scope <scope>`| 활성화할 스코프: `user`, `project`, `local`. 생략 시 Claude Code가 플러그인이 설치된 스코프를 감지함             | 자동 감지  |
| `-h, --help`        | 명령 도움말 표시                                                                                                   |            |

### plugin disable

언인스톨하지 않고 플러그인을 비활성화합니다. 활성화된 다른 플러그인이 타깃을 [의존하는 경우](/docs/en/plugin-dependencies#enable-or-disable-a-plugin-with-dependencies) 실패합니다. 오류 메시지에는 각 의존 플러그인을 먼저 비활성화하는 체인 형태의 명령이 포함됩니다.

```bash theme={null}
claude plugin disable [plugin] [options]
```

**인수:**

* `[plugin]`: 플러그인 이름 또는 `plugin-name@marketplace-name`. `--all`을 사용할 때는 선택 사항

**옵션:**

| 옵션                | 설명                                                                                                               | 기본값     |
| :------------------ | :----------------------------------------------------------------------------------------------------------------- | :--------- |
| `-a, --all`         | 활성화된 모든 플러그인 비활성화. `--scope`와 조합할 수 없음                                                        |            |
| `-s, --scope <scope>`| 비활성화할 스코프: `user`, `project`, `local`. 생략 시 Claude Code가 플러그인이 설치된 스코프를 감지함            | 자동 감지  |
| `-h, --help`        | 명령 도움말 표시                                                                                                   |            |

### plugin update

플러그인을 최신 버전으로 업데이트합니다.

```bash theme={null}
claude plugin update <plugin> [options]
```

**인수:**

* `<plugin>`: 플러그인 이름 또는 `plugin-name@marketplace-name`

**옵션:**

| 옵션                | 설명                                                     | 기본값 |
| :------------------ | :------------------------------------------------------- | :----- |
| `-s, --scope <scope>`| 업데이트할 스코프: `user`, `project`, `local`, `managed` | `user` |
| `-h, --help`        | 명령 도움말 표시                                         |        |

***

### plugin list

설치된 플러그인을 버전, 출처 마켓플레이스, 활성화 상태와 함께 나열합니다.

```bash theme={null}
claude plugin list [options]
```

**옵션:**

| 옵션          | 설명                                                              | 기본값 |
| :------------ | :---------------------------------------------------------------- | :----- |
| `--json`      | JSON 형태로 출력                                                  |        |
| `--available` | 마켓플레이스에서 이용 가능한 플러그인도 포함. `--json` 필요       |        |
| `-h, --help`  | 명령 도움말 표시                                                  |        |

대화형 세션 내부에서 `/plugin list`는 인라인으로 유사한 목록을 출력하지만, 마켓플레이스에서 설치된 플러그인만 다룹니다:

* skills 디렉토리에서 로드된 플러그인은 `/plugin` 인터페이스 및 `claude plugin list`에는 나타나지만, 인라인 `/plugin list` 출력에는 나타나지 않습니다.
* `--plugin-dir` 또는 `--plugin-url`로 세션에 로드된 플러그인은 `/plugin` 인터페이스에 나타나며, `claude --plugin-dir <dir> plugin list`와 같이 동일한 플래그가 하위 명령 앞에 올 때만 `claude plugin list`에 나타납니다. 이들은 설치 기록이 없으므로 순수한 `claude plugin list`에서는 나타나지 않습니다.

대화형 형태는 해당 상태의 플러그인만 표시하는 `--enabled` 또는 `--disabled`를 수용하며, `list`에 대한 축약 표현으로 `ls`를 지원합니다.

### plugin details

플러그인의 구성 요소 목록과 예상 토큰 비용을 보여줍니다. 출력에는 플러그인이 기여하는 모든 구성 요소가 Skills, Agents, Hooks, MCP servers, LSP servers로 그룹화되어 나열되며, 매 세션마다 추가하는 예상 토큰 수도 함께 표시됩니다. Skills 그룹에는 `skills/` 및 `commands/` 항목이 모두 포함됩니다.

```bash theme={null}
claude plugin details <name>
```

**인수:**

* `<name>`: 플러그인 이름 또는 `plugin-name@marketplace-name`

**옵션:**

| 옵션        | 설명              | 기본값 |
| :---------- | :---------------- | :----- |
| `-h, --help`| 명령 도움말 표시  |        |

출력에는 각 구성 요소에 대해 두 개의 비용 수치가 표시됩니다:

* **Always-on (항상 적용):** 어떠한 구성 요소가 실행되는지 여부와 상관없이 스킬 설명, 에이전트 설명, 명령 이름 등 플러그인의 목록 텍스트에 의해 매 세션마다 추가되는 토큰.
* **On-invoke (호출 시):** 구성 요소가 구동될 때 소요되는 토큰. 일반적인 세션은 구성 요소의 일부만 호출하므로 플러그인 전체 합계가 아닌 구성 요소별로 표시됩니다.

이 예시는 두 스킬이 있는 플러그인의 출력이 어떻게 보이는지 보여줍니다:

```
dependency-guard 1.2.0
  Dependency analysis for Claude Code sessions
  Source: dependency-guard@example-marketplace

Component inventory
  Skills (2)  scan-dependencies, review-changes
  Agents (0)
  Hooks (1)  SessionStart  (harness-only — no model context cost)
  MCP servers (0)
  LSP servers (0)

Projected token cost
  Always-on:   ~180 tok   added to every session

Per-component (rounded)
  component            always-on  on-invoke
  scan-dependencies        ~100      ~2400
  review-changes            ~80      ~1800

  On-invoke cost is paid each time a skill or agent fires.
  Token counts are estimates and may differ from actual usage.
```

always-on 합계는 사용자의 활성 모델에 대한 `count_tokens` API를 통해 계산됩니다. 구성 요소별 수치는 해당 합계에서 비례하여 스케일링됩니다. API에 도달할 수 없는 경우 명령은 문자 기반 추정치로 폴백합니다.

### plugin tag

플러그인에 대한 릴리스 git 태그를 생성합니다. 기본적으로 명령은 현재 디렉토리의 플러그인에 태그를 지정합니다; 다른 곳의 플러그인에 태그를 지정하려면 경로를 전달하세요. [버전 해상을 위해 플러그인 릴리스에 태그 지정](/docs/en/plugin-dependencies#tag-plugin-releases-for-version-resolution)을 참조하세요.

```bash theme={null}
claude plugin tag [path] [options]
```

**인수:**

* `[path]`: 플러그인 디렉토리 경로. 기본값은 현재 디렉토리.

**옵션:**

| 옵션                  | 설명                                                                       | 기본값   |
| :-------------------- | :------------------------------------------------------------------------- | :------- |
| `--push`              | 태그 생성 후 원격(remote)으로 태그 푸시                                     |          |
| `--dry-run`           | 태그를 생성하지 않고 태그가 붙을 내용을 출력                               |          |
| `-f, --force`         | 작업 트리가 지저분하거나 태그가 이미 존재하더라도 태그 생성                 |          |
| `-m, --message <msg>` | 태그 어노테이션 메시지. 버전에 대한 플레이스홀더로 `%s` 사용               |          |
| `--remote <name>`     | `--push` 사용 시 푸시할 원격                                               | `origin` |
| `-h, --help`          | 명령 도움말 표시                                                           |          |

***

## 디버깅 및 개발 도구

### 디버깅 명령

플러그인 로딩 세부사항을 확인하려면 `claude --debug`를 사용하세요:

다음을 보여줍니다:

* 어떤 플러그인이 로드되고 있는지
* 플러그인 매니페스트의 모든 오류
* 스킬, 에이전트, 훅 등록 상태
* MCP 서버 초기화 상태

### 흔한 문제들

| 문제                                | 원인                            | 해결 방법                                                                                                                                                       |
| :---------------------------------- | :------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 플러그인이 로드되지 않음            | 유효하지 않은 `plugin.json`      | `plugin.json`, 스킬/에이전트/명령 프론트매터, `hooks/hooks.json`의 구문 및 스키마 오류를 검사하기 위해 `claude plugin validate` 또는 `/plugin validate` 실행    |
| 스킬이 보이지 않음                  | 잘못된 디렉토리 구조            | `skills/` 또는 `commands/`가 `.claude-plugin/` 내부가 아닌 플러그인 루트에 위치하는지 확인                                                                      |
| 훅이 실행되지 않음                  | 스크립트가 실행 파일이 아님      | `chmod +x script.sh` 실행                                                                                                                                      |
| MCP 서버 실패                       | `${CLAUDE_PLUGIN_ROOT}` 누락    | 모든 플러그인 경로에 변수 사용                                                                                                                                  |
| 경로 오류                           | 절대 경로 사용됨                | 모든 경로는 상대 경로여야 하며 `./`로 시작해야 함                                                                                                               |
| LSP `Executable not found in $PATH` | 언어 서버가 설치되지 않음       | 바이너리 설치 (예: `npm install -g typescript-language-server typescript`)                                                                                      |

### 오류 메시지 예시

**매니페스트 검증 오류**:

* `Invalid JSON syntax: Unexpected token } in JSON at position 142`: 콤마 누락, 콤마 남발, 또는 따옴표 없는 문자열 확인
* `Plugin <name> has an invalid manifest file at .claude-plugin/plugin.json. Validation errors: name: Invalid input: expected string, received undefined`: 필수 필드 누락
* `Plugin <name> has a corrupt manifest file at .claude-plugin/plugin.json. JSON parse error: ...`: JSON 구문 오류

**플러그인 로딩 오류**:

* `Warning: No commands found in plugin my-plugin custom directory: ./cmds. Expected .md files or SKILL.md in subdirectories.`: 명령 경로가 존재하나 유효한 명령 파일이 포함되어 있지 않음
* `Plugin directory not found at path: ./plugins/my-plugin. Check that the marketplace entry has the correct path.`: marketplace.json의 `source` 경로가 존재하지 않는 디렉토리를 가리킴
* `Plugin my-plugin has conflicting manifests: both plugin.json and marketplace entry specify components.`: 중복된 구성 요소 정의를 제거하거나 marketplace 항목에서 `strict: false` 제거

### 훅 문제 해결

**훅 스크립트가 실행되지 않음**:

1. 스크립트가 실행 파일인지 확인: `chmod +x ./scripts/your-script.sh`
2. 쉬뱅(shebang) 줄 검증: 첫 줄이 `#!/bin/bash` 또는 `#!/usr/bin/env bash`여야 함
3. 경로가 `${CLAUDE_PLUGIN_ROOT}`를 사용하는지 확인: `"command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/your-script.sh"`
4. 스크립트를 수동으로 테스트: `./scripts/your-script.sh`

**예상한 이벤트에서 훅이 트리거되지 않음**:

1. 이벤트 이름이 올바른지 확인 (대소문자 구분): `postToolUse`가 아닌 `PostToolUse`
2. 매처 패턴이 본인의 도구와 일치하는지 확인: 파일 조작의 경우 `"matcher": "Write|Edit"`
3. 훅 유형이 유효한지 확인: `command`, `http`, `mcp_tool`, `prompt`, `agent`

### MCP 서버 문제 해결

**서버가 시작되지 않음**:

1. 명령이 존재하고 실행 파일인지 확인
2. 모든 경로가 `${CLAUDE_PLUGIN_ROOT}` 변수를 사용하는지 검증
3. MCP 서버 로그 확인: `claude --debug`가 초기화 오류를 보여줌
4. Claude Code 외부에서 서버를 수동으로 테스트

**서버 도구가 보이지 않음**:

1. 서버가 `.mcp.json` 또는 `plugin.json`에 올바르게 구성되어 있는지 확인
2. 서버가 MCP 프로토콜을 올바르게 구현하고 있는지 검증
3. 디버그 출력에서 연결 타임아웃 확인

### 디렉토리 구조 실수

**증상**: 플러그인이 로드되지만 구성 요소(스킬, 에이전트, 훅)가 누락됨.

**올바른 구조**: 구성 요소는 `.claude-plugin/` 내부가 아닌 플러그인 루트에 위치해야 합니다. `.claude-plugin/` 내부에는 `plugin.json`만 들어갑니다.

```text theme={null}
my-plugin/
├── .claude-plugin/
│   └── plugin.json      ← 여기에 매니페스트만 존재
├── commands/            ← 루트 수준
├── agents/              ← 루트 수준
└── hooks/               ← 루트 수준
```

구성 요소가 `.claude-plugin/` 내부에 있다면 플러그인 루트로 이동시키세요.

**디버그 체크리스트**:

1. `claude --debug`를 실행하고 "loading plugin" 메시지 찾기
2. 각 구성 요소 디렉토리가 디버그 출력에 나열되어 있는지 확인
3. 파일 권한이 플러그인 파일 읽기를 허용하는지 검증

***

## 배포 및 버전 관리 참조

### 버전 관리

Claude Code는 업데이트가 이용 가능한지를 결정하는 캐시 키로 플러그인의 버전을 사용합니다. `/plugin update`를 실행하거나 자동 업데이트가 발동할 때 Claude Code는 현재 버전을 계산하고 이미 설치된 것과 일치하면 업데이트를 건너뜁니다.

버전은 다음 중 설정된 첫 번째 항목으로부터 해석됩니다:

1. 플러그인의 `plugin.json`에 있는 `version` 필드
2. `marketplace.json`에 있는 플러그인의 마켓플레이스 항목의 `version` 필드
3. git 호스팅 마켓플레이스의 `github`, `url`, `git-subdir`, 상대 경로 소스에 대한 플러그인 소스의 git 커밋 SHA
4. `npm` 소스 또는 git 저장소 내부가 아닌 로컬 디렉토리에 대한 `unknown`

이를 통해 플러그인 버전을 관리하는 두 가지 방법이 제공됩니다:

| 접근 방식             | 방법                                                             | 업데이트 동작                                                                                                                                                                                | 적합한 대상                                       |
| :-------------------- | :--------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------ |
| **명시적 버전**       | `plugin.json`에 `"version": "2.1.0"` 설정                        | 사용자는 이 필드를 올릴 때만 업데이트를 받음. 필드를 올리지 않고 새 커밋을 푸시하는 것은 아무런 효과가 없으며, `/plugin update`가 "already at the latest version"을 보고함                   | 안정적인 릴리스 주기를 갖는 게시된 플러그인       |
| **커밋 SHA 버전**     | `plugin.json` 및 마켓플레이스 항목 모두에서 `version`을 생략함   | 사용자는 플러그인의 git 소스로의 모든 새 커밋 시 업데이트를 받음                                                                                                                              | 활발히 개발 중인 내부 또는 팀 플러그인            |

<Warning>
  `plugin.json`에 `version`을 설정한 경우 사용자가 변경 사항을 받도록 하려면 매번 버전을 올려야 합니다. Claude Code가 동일한 버전 문자열을 보고 캐시된 사본을 유지하기 때문에 새 커밋만 푸시하는 것으로는 부족합니다. 빠른 반복 작업을 진행 중이라면 git 커밋 SHA가 대신 사용되도록 `version`을 설정되지 않은 상태로 두세요.
</Warning>

명시적 버전을 사용하는 경우 [시맨틱 버전 관리](https://semver.org)(`MAJOR.MINOR.PATCH`)를 따르세요: 호환성을 깨는 변경에는 MAJOR, 새 기능에는 MINOR, 버그 수정에는 PATCH를 올립니다. 변경 사항은 `CHANGELOG.md`에 문서화하세요.

***

## 참고 항목

* [플러그인](/docs/en/plugins) - 튜토리얼 및 실용적 사용법
* [플러그인 마켓플레이스](/docs/en/plugin-marketplaces) - 마켓플레이스 생성 및 관리
* [스킬](/docs/en/skills) - 스킬 개발 세부사항
* [Subagents](/docs/en/sub-agents) - 에이전트 구성 및 기능
* [훅](/docs/en/hooks) - 이벤트 처리 및 자동화
* [MCP](/docs/en/mcp) - 외부 도구 통합
* [설정](/docs/en/settings) - 플러그인을 위한 구성 옵션
