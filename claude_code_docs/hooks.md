> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Hooks 레퍼런스

> Claude Code 훅 이벤트, 구성 스키마, JSON 입력/출력 형식, 종료 코드, 비동기 훅, HTTP 훅, 프롬프트 훅 및 MCP 도구 훅에 대한 레퍼런스입니다.

<Tip>
  예제가 포함된 빠른 시작 가이드는 [훅으로 작업 자동화하기](/docs/en/hooks-guide)를 참조하세요.
</Tip>

훅(Hook)은 Claude Code 라이프사이클의 특정 시점에 자동으로 실행되는 사용자 정의 셸 명령, HTTP 엔드포인트 또는 LLM 프롬프트입니다. 이 레퍼런스를 사용하여 이벤트 스키마, 구성 옵션, JSON 입력/출력 형식, 그리고 비동기 훅, HTTP 훅, MCP 도구 훅과 같은 고급 기능을 찾아볼 수 있습니다. 훅을 처음 설정하는 경우 먼저 [가이드](/docs/en/hooks-guide)부터 시작하세요.

## 훅 라이프사이클

훅은 Claude Code 세션 동안 특정 시점에 발생합니다. 이벤트가 발생하고 매처(matcher)가 일치하면 Claude Code는 이벤트에 대한 JSON 컨텍스트를 훅 핸들러로 전달합니다. 명령 훅의 경우 입력이 stdin으로 도착합니다. HTTP 훅의 경우 POST 요청 본문으로 도착합니다. 핸들러는 입력을 검사하고 조치를 취하며 선택적으로 결정을 반환할 수 있습니다.

이벤트는 세 가지 주기(cadence)로 분류됩니다:

* 세션당 한 번: `SessionStart` 및 `SessionEnd`
* 턴당 한 번: `UserPromptSubmit`, `Stop`, 및 `StopFailure`
* 에이전트 루프 내 모든 도구 호출 시: `PreToolUse` 및 `PostToolUse` ([`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior) 호출 제외, 두 이벤트 모두 건너뜀)

<div style={{maxWidth: "500px", margin: "0 auto"}}>
  <Frame>
    <img src="https://mintcdn.com/claude-code/uLsR38F1U_5zPppm/images/hooks-lifecycle.svg?fit=max&auto=format&n=uLsR38F1U_5zPppm&q=85&s=fbdbd78ad9f474da7d344879341341f0" alt="Hook lifecycle diagram showing optional Setup feeding into SessionStart, then a per-turn loop containing UserPromptSubmit, UserPromptExpansion for slash commands, the nested agentic loop (PreToolUse, PermissionRequest, PostToolUse, PostToolUseFailure, PostToolBatch, SubagentStart/Stop, TaskCreated, TaskCompleted), and Stop or StopFailure, followed by TeammateIdle, PreCompact, PostCompact, and SessionEnd, with Elicitation and ElicitationResult nested inside MCP tool execution, PermissionDenied as a side branch from PermissionRequest for auto-mode denials, WorktreeCreate, WorktreeRemove, Notification, ConfigChange, InstructionsLoaded, CwdChanged, and FileChanged as standalone async events, and MessageDisplay as a display-only event that runs while assistant message text streams" width="520" height="1228" data-path="images/hooks-lifecycle.svg" />
  </Frame>
</div>

아래 표는 각 이벤트가 언제 발생하는지 요약합니다. [훅 이벤트](#hook-events) 섹션에서 각 이벤트의 전체 입력 스키마 및 결정 제어 옵션을 제공합니다.

| 이벤트                 | 발생 시점                                                                                                                                          |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SessionStart`        | 세션이 시작되거나 재개될 때                                                                                                                       |
| `Setup`               | `--init-only`로 Claude Code를 시작하거나, `-p` 모드에서 `--init` 또는 `--maintenance`를 사용할 때. CI 또는 스크립트에서의 일회성 준비 작업용             |
| `UserPromptSubmit`    | 프롬프트를 제출할 때, Claude가 처리하기 전                                                                                                   |
| `UserPromptExpansion` | 사용자 입력 명령이 프롬프트로 확장될 때, Claude에 도달하기 전. 확장을 차단할 수 있음                                                     |
| `PreToolUse`          | 도구 호출이 실행되기 전. 호출을 차단할 수 있음                                                                                                              |
| `PermissionRequest`   | 권한 대화 상자가 나타날 때                                                                                                                       |
| `PermissionDenied`    | auto 모드 분류기에 의해 도구 호출이 거부될 때. 모델이 거부된 도구 호출을 재시도할 수 있음을 알리려면 `{retry: true}`를 반환                     |
| `PostToolUse`         | 도구 호출이 성공한 후                                                                                                                            |
| `PostToolUseFailure`  | 도구 호출이 실패한 후                                                                                                                                |
| `PostToolBatch`       | 병렬 도구 호출의 전체 배치가 해결된 후, 다음 모델 호출 전                                                                         |
| `Notification`        | Claude Code가 알림을 보낼 때                                                                                                                  |
| `MessageDisplay`      | 어시스턴트 메시지 텍스트가 표시되는 동안                                                                                                              |
| `SubagentStart`       | 서브에이전트가 생성될 때                                                                                                                             |
| `SubagentStop`        | 서브에이전트가 종료될 때                                                                                                                               |
| `TaskCreated`         | `TaskCreate`를 통해 작업이 생성될 때                                                                                                          |
| `TaskCompleted`       | 작업이 완료됨으로 표시될 때                                                                                                               |
| `Stop`                | Claude가 응답을 마쳤을 때                                                                                                                        |
| `StopFailure`         | API 오류로 인해 턴이 종료될 때. 출력 및 종료 코드는 무시됨                                                                               |
| `TeammateIdle`        | [에이전트 팀](/docs/en/agent-teams) 팀원이 유휴 상태(idle)가 되려고 할 때                                                                                     |
| `InstructionsLoaded`  | CLAUDE.md 또는 `.claude/rules/*.md` 파일이 컨텍스트로 로드될 때. 세션 시작 시 및 세션 도중 지연 로드될 때 발생         |
| `ConfigChange`        | 세션 중 설정 파일이 변경될 때                                                                                                     |
| `CwdChanged`          | Claude가 `cd` 명령을 실행하는 등 작업 디렉토리가 변경될 때. direnv와 같은 도구로 반응형 환경 관리에 유용 |
| `FileChanged`         | 디스크에서 감시 중인 파일이 변경될 때. `matcher` 필드가 감시할 파일 이름을 지정함                                                            |
| `WorktreeCreate`      | `--worktree`, `isolation: "worktree"` 또는 백그라운드 세션을 위해 워크트리가 생성될 때. 기본 git 동작을 대체함                 |
| `WorktreeRemove`      | 세션 종료 시, 서브에이전트 종료 시 또는 백그라운드 세션을 삭제할 때 워크트리가 제거될 때                                    |
| `PreCompact`          | 컨텍스트 압축(compaction) 전                                                                                                                              |
| `PostCompact`         | 컨텍스트 압축이 완료된 후                                                                                                                     |
| `Elicitation`         | 도구 호출 중 MCP 서버가 사용자 입력을 요청할 때                                                                                              |
| `ElicitationResult`   | 사용자가 MCP 요청에 응답한 후, 서버로 응답이 다시 전송되기 전                                                            |
| `SessionEnd`          | 세션이 종료될 때                                                                                                                              |

### 훅 해결 방식

이 요소들이 어떻게 맞춰지는지 보기 위해 파괴적인 셸 명령을 차단하는 다음 `PreToolUse` 훅을 고려해보세요. `matcher`는 Bash 도구 호출로 범위를 좁히고 `if` 조건은 `rm *`와 일치하는 Bash 하위 명령으로 추가로 좁히므로 두 필터가 모두 일치할 때만 `block-rm.sh`가 실행됩니다:

```json theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "args": []
          }
        ]
      }
    ]
  }
}
```

스크립트는 stdin에서 JSON 입력을 읽고 명령을 추출한 후, `rm -rf`가 포함되어 있으면 `"deny"`의 `permissionDecision`을 반환합니다:

```bash theme={null}
#!/bin/bash
# .claude/hooks/block-rm.sh
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive command blocked by hook"
    }
  }'
else
  exit 0  # 결정 없음; 일반 권한 흐름 적용
fi
```

이 스크립트와 JSON 입력을 파싱하는 이 페이지의 Bash 예제는 `jq`를 사용하므로 테스트하기 전에 `jq`를 설치하고 `PATH`에 추가했는지 확인하세요.

이제 Claude Code가 `Bash "rm -rf /tmp/build"`를 실행하기로 결정했다고 가정해 봅시다. 진행 과정은 다음과 같습니다:

<Frame>
  <img src="https://mintcdn.com/claude-code/ikqp3_70mqIahteV/images/hook-resolution.svg?fit=max&auto=format&n=ikqp3_70mqIahteV&q=85&s=be0bf3053550c26de5f54cd64674c197" alt="Diagram of hook resolution: PreToolUse fires, the matcher checks for a Bash match, then the if condition checks for a Bash(rm *) match. If both match, the hook command runs and returns permissionDecision deny, so the tool call is blocked and Claude Code continues. If either check fails to match, the hook is skipped and the tool call is allowed to proceed." width="930" height="270" data-path="images/hook-resolution.svg" />
</Frame>

<Steps>
  <Step title="이벤트 발생">
    `PreToolUse` 이벤트가 발생합니다. Claude Code는 도구 입력을 JSON 형식으로 훅의 stdin에 보냅니다:

    ```json theme={null}
    { "tool_name": "Bash", "tool_input": { "command": "rm -rf /tmp/build" }, ... }
    ```
  </Step>

  <Step title="매처 검사">
    매처 `"Bash"`가 도구 이름과 일치하므로 이 훅 그룹이 활성화됩니다. 매처를 생략하거나 `"*"`, `""`를 사용하면 이벤트가 발생할 때마다 그룹이 활성화됩니다.
  </Step>

  <Step title="If 조건 검사">
    `rm -rf /tmp/build`가 `rm *`와 일치하는 하위 명령이므로 `if` 조건 `"Bash(rm *)"`가 일치하여 이 핸들러가 실행됩니다. 명령이 `npm test`였다면 `if` 검사가 실패하여 `block-rm.sh`가 실행되지 않아 프로세스 생성 오버헤드가 방지됩니다. `if` 필드는 선택 사항입니다; 이 필드가 없으면 일치하는 그룹의 모든 핸들러가 실행됩니다.
  </Step>

  <Step title="훅 핸들러 실행">
    스크립트는 전체 명령을 검사하고 `rm -rf`를 발견하여 stdout에 결정을 출력합니다:

    ```json theme={null}
    {
      "hookSpecificOutput": {
        "hookEventName": "PreToolUse",
        "permissionDecision": "deny",
        "permissionDecisionReason": "Destructive command blocked by hook"
      }
    }
    ```

    명령이 `rm file.txt`와 같이 안전한 변형이었다면 스크립트는 대신 `exit 0`을 만났을 것입니다. 출력 없는 종료 코드 0은 훅이 보고할 결정이 없음을 의미하므로 도구 호출은 일반적인 [권한 흐름](/docs/en/permissions)에 따라 계속됩니다. 훅은 호출을 거부할 수 있지만, 아무 말도 하지 않는다고 해서 승인되는 것은 아닙니다.
  </Step>

  <Step title="Claude Code의 결과 처리">
    Claude Code는 JSON 결정을 읽고 도구 호출을 차단한 뒤 Claude에게 이유를 보여줍니다.
  </Step>
</Steps>

아래의 [구성](#configuration) 섹션에서는 전체 스키마를 설명하고 각 [훅 이벤트](#hook-events) 섹션에서는 명령이 수신하는 입력과 반환할 수 있는 출력을 설명합니다.

## 구성

훅은 JSON 설정 파일에 정의됩니다. 구성에는 세 가지 중첩 수준이 있습니다:

1. `PreToolUse` 또는 `Stop`과 같이 응답할 [훅 이벤트](#hook-events) 선택
2. "Bash 도구에 대해서만"과 같이 실행되는 시점을 필터링할 [매처 그룹](#matcher-patterns) 추가
3. 일치할 때 실행할 하나 이상의 [훅 핸들러](#hook-handler-fields) 정의

주석이 달린 예제와 함께 전체 과정을 보려면 위의 [훅 해결 방식](#how-a-hook-resolves)을 참조하세요.

<Note>
  이 페이지에서는 각 수준에 대해 특정 용어를 사용합니다. 라이프사이클 지점에는 **훅 이벤트(hook event)**, 필터에는 **매처 그룹(matcher group)**, 실행되는 셸 명령, HTTP 엔드포인트, MCP 도구, 프롬프트 또는 에이전트에는 **훅 핸들러(hook handler)**를 사용합니다. 단독으로 쓰인 "훅"은 일반적인 기능을 의미합니다.
</Note>

### 훅 위치

훅을 정의하는 위치에 따라 그 범주(scope)가 결정됩니다:

| 위치                                                       | 범주                           | 공유 가능 여부                             |
| :--------------------------------------------------------- | :---------------------------- | :----------------------------------------- |
| `~/.claude/settings.json`                                  | 모든 프로젝트                 | 아니오, 로컬 컴퓨터 전용                  |
| `.claude/settings.json`                                    | 단일 프로젝트                | 예, 저장소에 커밋 가능          |
| `.claude/settings.local.json`                              | 단일 프로젝트                | 아니오, Claude Code가 생성 시 gitignore됨 |
| 관리형 정책 설정(Managed policy settings)                 | 조직 전체                     | 예, 관리자 제어                      |
| [플러그인](/docs/en/plugins) `hooks/hooks.json`           | 플러그인이 활성화되었을 때        | 예, 플러그인과 번들로 제공               |
| [스킬](/docs/en/skills) 또는 [에이전트](/docs/en/sub-agents) 프론트매터 | 구성 요소가 활성화되어 있는 동안 | 예, 구성 요소 파일에 정의됨         |

설정 파일 해제(resolution)에 대한 자세한 내용은 [설정](/docs/en/settings)을 참조하세요.

엔터프라이즈 관리자는 `allowManagedHooksOnly`를 사용하여 사용자, 프로젝트 및 플러그인 훅을 차단할 수 있습니다. 관리형 설정 `enabledPlugins`에서 강제 활성화된 플러그인의 훅은 면제되므로 관리자는 검증된 훅을 조직 마켓플레이스를 통해 배포할 수 있습니다. [훅 구성](/docs/en/settings#hook-configuration)을 참조하세요.

### 매처 패턴

`matcher` 필드는 훅이 발생하는 시점을 필터링합니다. 매처 평가 방식은 포함된 문자에 따라 다릅니다:

| 매처 값                                               | 평가 방식                                                                                            | 예시                                                                                                                                         |
| :---------------------------------------------------- | :--------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| `"*"`, `""` 또는 생략                               | 모든 항목 일치                                                                                       | 이벤트가 발생할 때마다 실행                                                                                                          |
| 영문자, 숫자, `_`, `-`, 공백, `,`, `\|`만 포함 | 정확한 문자열, 또는 `\|`나 `,`로 구분된 정확한 문자열 목록 (선택적 주변 공백 허용) | `Bash`는 Bash 도구만 일치; `Edit\|Write` 및 `Edit, Write`는 각 도구와 정확히 일치; `code-reviewer`는 해당 에이전트 유형만 일치 |
| 다른 문자를 포함함                          | JavaScript 정규식, 앵커 미지정(unanchored)                                                           | `^Notebook`은 `Notebook`으로 시작하는 모든 도구 일치; `mcp__memory__.*`는 `memory` 서버의 모든 도구 일치                   |

정규식 경로의 매처는 JavaScript의 `RegExp.prototype.test`로 테스트되며, 값의 어디서나 일치하면 성공합니다. `Edit.*`는 `Edit` 및 `NotebookEdit` 모두와 일치합니다; 전체 문자열 일치가 필요할 때에는 `^Edit$`와 같이 패턴을 `^`와 `$`로 감싸세요.

쉼표 구분 기호 및 주변 공백 허용 기능은 Claude Code v2.1.191 이상이 필요합니다.

정확한 일치 집합의 하이픈은 Claude Code v2.1.195 이상이 필요합니다. 이전 버전에서 `code-reviewer`와 같이 하이픈이 들어간 이름은 앵커가 지정되지 않은 정규식으로 평가되어 `senior-code-reviewer`에도 실행됩니다; 해당 버전에서는 해당 이름만 일치시키려면 `^code-reviewer$`로 앵커를 지정하세요.

`FileChanged` 및 `StopFailure`는 영문자, 숫자, `_`, `|`로만 구성된 더 좁은 정확한 일치 집합을 사용합니다. 해당 두 이벤트의 매처에 하이픈, 공백 또는 쉼표가 있으면 정규식 경로로 유지되며 `|`만 대안을 구분합니다. 이어지는 표에서 매처를 지원하는 다른 모든 이벤트는 `|` 또는 `,`를 수락합니다.

`FileChanged` 이벤트는 감시 목록을 생성할 때 이 규칙을 따르지 않습니다. [FileChanged](#filechanged)를 참조하세요.

각 이벤트 유형은 서로 다른 필드에서 일치시킵니다:

| 이벤트                                                                                                                                            | 매처가 필터링하는 대상                                     | 매처 값 예시                                                                                                                                                              |
| :------------------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`                                                        | 도구 이름                                                    | `Bash`, `Edit\|Write`, `mcp__.*`                                                                                                                                                    |
| `SessionStart`                                                                                                                                    | 세션 시작 방식                                      | `startup`, `resume`, `clear`, `compact`, `fork`                                                                                                                                     |
| `Setup`                                                                                                                                           | 설정을 트리거한 CLI 플래그                               | `init`, `maintenance`                                                                                                                                                               |
| `SessionEnd`                                                                                                                                      | 세션 종료 이유                                        | `clear`, `resume`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other`                                                                                            |
| `Notification`                                                                                                                                    | 알림 유형                                            | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response`, `agent_needs_input`, `agent_completed`                    |
| `SubagentStart`                                                                                                                                   | 에이전트 유형                                                   | `general-purpose`, `Explore`, `Plan`, 커스텀 에이전트 이름 또는 `^my-plugin:reviewer$`와 같은 플러그인 범위 이름                                                                        |
| `PreCompact`, `PostCompact`                                                                                                                       | 압축 트리거 원인                                    | `manual`, `auto`                                                                                                                                                                    |
| `SubagentStop`                                                                                                                                    | 에이전트 유형                                                   | `SubagentStart`와 동일한 값                                                                                                                       |
| `ConfigChange`                                                                                                                                    | 구성 소스                                         | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills`                                                                                                  |
| `CwdChanged`                                                                                                                                      | 매처 미지원                                           | 모든 디렉토리 변경 시 항상 발생                                                                                                                                              |
| `FileChanged`                                                                                                                                     | 감시할 리터럴 파일 이름 ([FileChanged](#filechanged) 참조) | `.envrc\|.env`                                                                                                                                                                      |
| `StopFailure`                                                                                                                                     | 오류 유형                                                   | `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, `unknown` |
| `InstructionsLoaded`                                                                                                                              | 로드 이유                                                  | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact`                                                                                                        |
| `UserPromptExpansion`                                                                                                                             | 명령 이름                                                 | 스킬 또는 명령 이름                                                                                                                                                         |
| `Elicitation`                                                                                                                                     | MCP 서버 이름                                              | 구성된 MCP 서버 이름                                                                                                                                                    |
| `ElicitationResult`                                                                                                                               | MCP 서버 이름                                              | `Elicitation`과 동일한 값                                                                                                                                                        |
| `UserPromptSubmit`, `PostToolBatch`, `Stop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `WorktreeCreate`, `WorktreeRemove`, `MessageDisplay` | 매처 미지원                                           | 발생할 때마다 항상 실행                                                                                                                                                    |

매처는 Claude Code가 stdin을 통해 훅으로 보내는 [JSON 입력](#hook-input-and-output)의 필드를 기준으로 실행됩니다. 도구 이벤트의 경우 해당 필드는 `tool_name`입니다. 각 [훅 이벤트](#hook-events) 섹션에서는 해당 이벤트의 매처 값 전체 및 입력 스키마를 나열합니다.

다음 예시는 Claude가 파일을 쓰거나 편집할 때만 린팅 스크립트를 실행합니다:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/lint-check.sh"
          }
        ]
      }
    ]
  }
}
```

`UserPromptSubmit`, `PostToolBatch`, `Stop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `WorktreeCreate`, `WorktreeRemove`, `MessageDisplay`, `CwdChanged`는 매처를 지원하지 않으며 항상 발생할 때마다 실행됩니다. 이러한 이벤트에 `matcher` 필드를 추가하면 무시됩니다.

도구 이벤트의 경우 개별 훅 핸들러에서 [`if` 필드](#common-fields)를 설정하여 더 좁게 필터링할 수 있습니다. `if`는 [권한 규칙 구문](/docs/en/permissions)을 사용하여 도구 이름과 인수를 함께 일치시키므로, `"Bash(git *)"`는 Bash 입력의 모든 하위 명령이 `git *`와 일치할 때 실행되고 `"Edit(*.ts)"`는 TypeScript 파일에 대해서만 실행됩니다.

#### MCP 도구 일치

[MCP](/docs/en/mcp) 서버 도구는 도구 이벤트(`PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`)에서 일반 도구로 나타나므로 다른 도구 이름과 동일한 방식으로 일치시킬 수 있습니다.

MCP 도구는 `mcp__<server>__<tool>` 명명 패턴을 따릅니다. 예:

* `mcp__memory__create_entities`: Memory 서버의 엔티티 생성 도구
* `mcp__filesystem__read_file`: Filesystem 서버의 파일 읽기 도구
* `mcp__github__search_repositories`: GitHub 서버의 검색 도구

서버의 모든 도구를 일치시키려면 서버 접두사 뒤에 `.*`를 추가하세요. `.*`는 필수입니다: `mcp__memory` 또는 `mcp__brave-search`와 같은 매처는 정확한 일치 문자만 포함하므로 정확한 문자열로 비교되어 어떠한 도구와도 일치하지 않습니다.

* `mcp__memory__.*`는 `memory` 서버의 모든 도구 일치
* `mcp__brave-search__.*`는 이름에 하이픈이 포함된 서버의 모든 도구 일치
* `mcp__.*__write.*`는 모든 서버에서 이름이 `write`로 시작하는 도구 일치

정확한 일치 집합의 하이픈은 Claude Code v2.1.195 이상이 필요합니다. 이전 버전에서 `mcp__brave-search`와 같이 하이픈이 지정된 단독 접두사는 앵커가 지정되지 않은 정규식으로 평가되어 해당 서버의 모든 도구와 일치합니다. `mcp__brave-search__.*` 형태는 모든 버전에서 작동합니다.

[플러그인 번들 MCP 서버](/docs/en/mcp#plugin-provided-mcp-servers)의 도구는 플러그인 이름이 포함된 범위 지정 서버 세그먼트를 사용합니다: `mcp__plugin_<plugin-name>_<server-name>__<tool>`. 단독 서버 키에 작성된 매처는 이러한 도구에 대해 실행되지 않습니다. `db` 키 아래에 서버를 번들로 제공하는 `my-plugin`이라는 플러그인의 경우 `query` 도구는 `mcp__plugin_my-plugin_db__query`로 나타나므로 해당 서버의 모든 도구에 대한 매처는 `mcp__plugin_my-plugin_db__.*`입니다. 핸들러의 [`if` 필드](#common-fields)에서도 동일한 범위 지정 도구 이름을 사용하세요. 범주 지정 이름이 설정되는 방식은 [플러그인 제공 MCP 서버](/docs/en/mcp#plugin-provided-mcp-servers)를 참조하세요.

이 예시는 모든 메모리 서버 작업을 기록하고 MCP 서버의 쓰기 작업을 검증합니다:

```json theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Memory operation initiated' >> ~/mcp-operations.log"
          }
        ]
      },
      {
        "matcher": "mcp__.*__write.*",
        "hooks": [
          {
            "type": "command",
            "command": "/home/user/scripts/validate-mcp-write.py"
          }
        ]
      }
    ]
  }
}
```

### 훅 핸들러 필드

내부 `hooks` 배열의 각 개체는 훅 핸들러입니다: 매처가 일치할 때 실행되는 셸 명령, HTTP 엔드포인트, MCP 도구, LLM 프롬프트 또는 에이전트입니다. 다섯 가지 유형이 있습니다:

* **[명령 훅](#command-hook-fields)** (`type: "command"`): 셸 명령을 실행합니다. 스크립트는 stdin에서 이벤트의 [JSON 입력](#hook-input-and-output)을 수신하고 종료 코드와 stdout을 통해 결과를 통신합니다.
* **[HTTP 훅](#http-hook-fields)** (`type: "http"`): 이벤트의 JSON 입력을 URL에 HTTP POST 요청으로 보냅니다. 엔드포인트는 명령 훅과 동일한 [JSON 출력 형식](#json-output)을 사용하여 응답 본문을 통해 결과를 다시 통신합니다.
* **[MCP 도구 훅](#mcp-tool-hook-fields)** (`type: "mcp_tool"`): 이미 연결된 [MCP 서버](/docs/en/mcp)의 도구를 호출합니다. 도구의 텍스트 내용은 명령 훅 stdout처럼 처리됩니다.
* **[프롬프트 훅](#prompt-and-agent-hook-fields)** (`type: "prompt"`): 단일 턴 평가를 위해 Claude 모델에 프롬프트를 보냅니다. 모델은 예/아니오 결정을 JSON으로 반환합니다. [프롬프트 기반 훅](#prompt-based-hooks)을 참조하세요.
* **[에이전트 훅](#prompt-and-agent-hook-fields)** (`type: "agent"`): 결정을 반환하기 전에 Read, Grep, Glob와 같은 도구를 사용하여 조건을 확인할 수 있는 서브에이전트를 생성합니다. 에이전트 훅은 실험적 기능이며 변경될 수 있습니다. [에이전트 기반 훅](#agent-based-hooks)을 참조하세요.

일치하는 모든 훅은 병렬로 실행되며 동일한 핸들러는 자동으로 중복 제거됩니다. 명령 훅은 명령 문자열 및 `args`로 중복 제거되고 HTTP 훅은 URL로 중복 제거됩니다.

핸들러는 Claude Code의 환경과 함께 현재 디렉토리에서 실행됩니다. `$CLAUDE_CODE_REMOTE` 환경 변수는 원격 웹 환경에서 `"true"`로 설정되고 로컬 CLI에서는 설정되지 않습니다. v2.1.199부터 로컬 세션에 활성 Remote Control 연결이 있는 동안 [`$CLAUDE_CODE_BRIDGE_SESSION_ID`](/docs/en/env-vars)가 [Remote Control](/docs/en/remote-control) 세션 ID로 설정됩니다.

#### 공통 필드

다음 필드는 모든 훅 유형에 적용됩니다:

| 필드            | 필수 | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| :-------------- | :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`          | 예      | `"command"`, `"http"`, `"mcp_tool"`, `"prompt"` 또는 `"agent"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `if`            | 아니오       | `"Bash(git *)"` 또는 `"Edit(*.ts)"`와 같이 이 훅이 실행되는 시점을 필터링하는 권한 규칙 구문. 도구 호출이 패턴과 일치하는 경우에만 훅 명령이 실행됩니다. 하위 명령, `$()`, 백틱에 대해 Bash 패턴이 어떻게 평가되는지는 아래의 [Bash matching table](#bash-if-matching)을 참조하세요. 도구 이벤트(`PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`)에서만 평가됩니다. 다른 이벤트에서 `if`가 설정된 훅은 실행되지 않습니다. [권한 규칙](/docs/en/permissions)과 동일한 구문을 사용합니다 |
| `timeout`       | 아니오       | 취소 전까지의 초 단위 시간. 기본값: `command`, `http`, `mcp_tool`은 600초; `prompt`는 30초; `agent`는 60초. [`UserPromptSubmit`](#userpromptsubmit)은 `command`, `http`, `mcp_tool` 기본값을 30초로 낮추고 [`MessageDisplay`](#messagedisplay)는 10초로 낮춥니다                                                                                                                                                                                                                                                                     |
| `statusMessage` | 아니오       | 훅이 실행되는 동안 표시되는 사용자 지정 스피너 메시지                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `once`          | 아니오       | `true`인 경우 세션당 한 번 실행된 후 제거됩니다. [스킬 프론트매터](#hooks-in-skills-and-agents)에 선언된 훅에 대해서만 적용되며 설정 파일 및 에이전트 프론트매터에서는 무시됩니다                                                                                                                                                                                                                                                                                                                                                          |

`if` 필드는 정확히 하나의 권한 규칙을 가집니다. 규칙을 결합하기 위한 `&&`, `||` 또는 목록 구문은 없습니다; 여러 조건을 적용하려면 각 조건에 대해 별도의 훅 핸들러를 정의하세요.

파일 도구의 `if` 조건에서 `"Edit(src/**)"`와 같은 단일 세그먼트 디렉토리 패턴은 작업 디렉토리의 `src` 디렉토리와 그 하위 파일만 일치시킵니다. 깊이에 상관없이 `src`라는 이름의 디렉토리를 일치시키려면 `"Edit(**/src/**)"`를 작성하세요. v2.1.214 이전에는 `"Edit(src/**)"`가 작업 디렉토리 아래의 임의의 깊이에 있는 `src` 디렉토리와 일치했습니다.

<span id="bash-if-matching" />Bash 패턴의 경우 훅 명령이 실행되는지 여부는 패턴의 형태와 Claude가 호출하는 Bash 명령에 따라 다릅니다. 선두의 `VAR=value` 할당은 일치 평가 전에 제거됩니다.

| `if` 패턴       | Bash 명령           | 훅 실행 여부 | 이유                                                                                                 |
| :----------------- | :--------------------- | :--------- | :-------------------------------------------------------------------------------------------------- |
| `Bash(git *)`      | `FOO=bar git push`     | 예         | 선두 할당 제거됨; `git push`가 일치함                                                |
| `Bash(git *)`      | `npm test && git push` | 예         | 각 하위 명령 검사됨; `git push`가 일치함                                                      |
| `Bash(rm *)`       | `echo $(rm -rf /)`     | 예         | `$()` 및 백틱 내부의 명령 검사됨; `rm -rf /`가 일치함                                 |
| `Bash(rm *)`       | `echo $(date)`         | 아니오         | `rm *`와 일치하는 하위 명령 없음                                                                        |
| `Bash(git push *)` | `echo $(date)`         | 예         | 명령 이름 이상의 요구 사항을 지정하는 패턴은 `$()`, 백틱 또는 `$VAR`에서 어쨌든 훅을 실행함 |

또한 셸 명령을 파싱할 수 없는 경우 `if` 필터는 실패하더라도 안전을 위해 패턴과 상관없이 훅을 실행합니다. `if` 필터는 최선의 노력(best-effort) 방식이므로 하드 허용 또는 거부를 강제하려면 훅 대신 [권한 시스템](/docs/en/permissions)을 사용하세요.

#### 명령 훅 필드

[공통 필드](#common-fields) 외에도 명령 훅은 다음 필드를 수락합니다:

| 필드         | 필수 | 설명                                                                                                                                                                                                                                                                                                                                  |
| :------------ | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `command`     | 예      | 실행할 셸 명령. `args`와 함께 사용되면 직접 생설할 실행 파일입니다. [Exec 형태 및 Shell 형태](#exec-form-and-shell-form)를 참조하세요                                                                                                                                                                                                           |
| `args`        | 아니오       | 인수 목록. 존재하는 경우 `command`는 실행 파일로 해석되고 셸 개입 없이 `args`를 인수 벡터로 사용하여 직접 생성됩니다. [Exec 형태 및 Shell 형태](#exec-form-and-shell-form)를 참조하세요                                                                                                                                                              |
| `async`       | 아니오       | `true`인 경우 차단 없이 백그라운드에서 실행됩니다. [백그라운드에서 훅 실행](#run-hooks-in-the-background)을 참조하세요                                                                                                                                                                                                                          |
| `asyncRewake` | 아니오       | `true`인 경우 백그라운드에서 실행되고 종료 코드 2에서 Claude를 재깨웁니다. `async`를 내포합니다. 훅의 stderr(stderr이 비어 있으면 stdout)가 시스템 리마인더로 Claude에게 표시되어 장시간 실행되는 백그라운드 오류에 반응할 수 있습니다                                                                                                            |
| `shell`       | 아니오       | 이 훅에 사용할 셸. `"bash"` 또는 `"powershell"`을 수락합니다. 기본값은 `"bash"`, 또는 Git Bash가 설치되어 있지 않은 Windows의 경우 `"powershell"`. `"powershell"`을 설정하면 Windows에서 PowerShell을 통해 명령을 실행합니다. 훅이 PowerShell을 직접 생성하므로 `CLAUDE_CODE_USE_POWERSHELL_TOOL`이 필요하지 않습니다. `args`가 설정된 경우 무시됩니다 |

<a id="exec-form-and-shell-form" />

##### Exec 형태 및 Shell 형태

명령 훅은 `args`가 설정된 경우 exec 형태로 실행되고 `args`가 생략된 경우 shell 형태로 실행됩니다. 훅이 [경로 자리 표시자](#reference-scripts-by-path)를 참조할 때마다 각 요소가 따옴표 없이 하나의 인수로 전달되므로 `args`를 설정하세요. 파이프나 `&&`와 같은 셸 기능이 필요하거나 두 가지 고려 사항이 모두 적용되지 않는 경우 `args`를 생략하세요.

**Exec 형태**는 `args`가 존재할 때 실행됩니다. Claude Code는 `command`를 `PATH`의 실행 파일로 해제하고 `args`를 인수 벡터로 직접 생성합니다. 셸이 없으므로 각 `args` 요소는 작성된 그대로의 하나의 인수이며 `${CLAUDE_PLUGIN_ROOT}`와 같은 경로 자리 표시자는 단순 문자열로 `command` 및 각 `args` 요소에 대체됩니다. 아포스트로피, `$`, 백틱과 같은 특수 문자는 해석할 셸이 없으므로 그래로 전달됩니다. 모든 플랫폼에서 셸 토큰화가 발생하지 않습니다.

**Shell 형태**는 `args`가 없을 때 실행됩니다. `command` 문자열이 셸로 전달됩니다: macOS 및 Linux에서는 `sh -c`, Windows에서는 Git Bash, 또는 Git Bash가 설치되지 않은 경우 PowerShell로 전달됩니다. 명시적으로 선택하려면 `shell` 필드를 설정하세요. 셸은 문자열을 토큰화하고 변수를 확장하며 파이프, `&&`, 리디렉션, 글로브를 해석합니다.

<Note>
  Windows에서 exec 형태를 사용하려면 `command`가 `.exe`와 같은 실제 실행 파일로 해제되어야 합니다. npm, npx, eslint 및 기타 도구가 `node_modules/.bin`에 설치하는 `.cmd` 및 `.bat` 심은 실행 파일이 아니며 셸 없이 생성할 수 없습니다. Exec 형태에서 이들을 실행하려면 `"command": "node", "args": ["${CLAUDE_PLUGIN_ROOT}/node_modules/eslint/bin/eslint.js"]`와 같이 `node`로 기본 스크립트를 직접 호출하세요. `node.exe`는 실제 바이너리이므로 `node` 및 스크립트 경로 패턴은 모든 플랫폼에서 동작합니다. 이름으로 `.cmd` 또는 `.bat` 심을 실행하려면 shell 형태를 사용하세요.
</Note>

다음 예시는 플러그인과 번들로 제공되는 Node 스크립트를 실행합니다. Exec 형태는 해제된 스크립트 경로를 따옴표 없이 하나의 인수로 전달합니다:

```json theme={null}
{
  "type": "command",
  "command": "node",
  "args": ["${CLAUDE_PLUGIN_ROOT}/scripts/format.js", "--fix"]
}
```

이에 해당하는 shell 형태는 공백이나 특수 문자가 있는 경로를 처리하기 위해 따옴표가 필요합니다:

```json theme={null}
{
  "type": "command",
  "command": "node \"${CLAUDE_PLUGIN_ROOT}\"/scripts/format.js --fix"
}
```

두 형태 모두 동일한 [경로 자리 표시자](#reference-scripts-by-path)를 지원하며 둘 다 생성된 프로세스의 환경 변수 `CLAUDE_PROJECT_DIR`, `CLAUDE_PLUGIN_ROOT`, `CLAUDE_PLUGIN_DATA`로 내보내므로, 스크립트는 어떻게 실행되었든 상관없이 `process.env.CLAUDE_PLUGIN_ROOT`를 읽을 수 있습니다.

플러그인 훅은 exec 형태에서만 [`${user_config.*}`](/docs/en/plugins-reference#user-configuration) 값을 추가로 대체합니다. 값이 단순 문자열로 `command` 및 각 `args` 요소에 대체되므로 셸이 이를 다시 파싱하지 않습니다.

`command`가 `${user_config.*}`를 참조하는 shell 형태 플러그인 훅은 실행되는 대신 [오류](/docs/en/errors#plugin-command-references-user-config)와 함께 실패합니다. shell 형태 훅에서 옵션 값을 사용하려면 `webhook_url` 옵션의 경우 `$CLAUDE_PLUGIN_OPTION_WEBHOOK_URL`과 같이 `$CLAUDE_PLUGIN_OPTION_<KEY>` 환경 변수를 읽거나 `args`를 설정하여 훅을 exec 형태로 전환하세요. v2.1.207 이전에는 shell 형태 플러그인 훅 명령도 `${user_config.*}`를 대체했습니다.

<Note>
  Exec 형태에서 `command`는 실행 파일 이름 또는 경로 전용입니다. `command`가 경로 구분 기호가 없는 단독 이름이고 `args`와 함께 공백을 포함하는 경우 생성이 실패하므로 Claude Code가 경고를 기록합니다: `node script.js`라는 이름의 실행 파일은 존재하지 않습니다. 추가 토큰을 `args`로 이동하세요. `C:\Program Files\nodejs\node.exe`와 같이 공백이 있는 절대 경로는 단일 유효 실행 파일이므로 경고를 트리거하지 않습니다.
</Note>

#### HTTP 훅 필드

[공통 필드](#common-fields) 외에도 HTTP 훅은 다음 필드를 수락합니다:

| 필드            | 필수 | 설명                                                                                                                                                                                      |
| :--------------- | :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `url`            | 예      | POST 요청을 보낼 URL                                                                                                                                                                  |
| `headers`        | 아니오       | 키-값 쌍 형태의 추가 HTTP 헤더. 값은 `$VAR_NAME` 또는 `${VAR_NAME}` 구문을 사용한 환경 변수 보간을 지원합니다. `allowedEnvVars`에 나열된 변수만 해제됩니다  |
| `allowedEnvVars` | 아니오       | 헤더 값으로 보간될 수 있는 환경 변수 이름 목록. 나열되지 않은 변수에 대한 참조는 빈 문자열로 대체됩니다. 환경 변수 보간이 작동하려면 필수입니다 |

Claude Code는 훅의 [JSON 입력](#hook-input-and-output)을 `Content-Type: application/json` 헤더와 함께 POST 요청 본문으로 보냅니다. 응답 본문은 명령 훅과 동일한 [JSON 출력 형식](#json-output)을 사용합니다.

오류 처리는 명령 훅과 다릅니다: 2xx가 아닌 응답, 연결 실패, 타임아웃은 모두 실행을 계속할 수 있는 비차단(non-blocking) 오류를 생성합니다. 도구 호출을 차단하거나 권한을 거부하려면 `decision: "block"` 또는 `permissionDecision: "deny"`가 포함된 `hookSpecificOutput`을 포함하는 JSON 본문과 함께 2xx 응답을 반환하세요.

다음 예시는 `MY_TOKEN` 환경 변수의 토큰으로 인증하여 `PreToolUse` 이벤트를 로컬 검증 서비스로 보냅니다:

```json theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/pre-tool-use",
            "timeout": 30,
            "headers": {
              "Authorization": "Bearer $MY_TOKEN"
            },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  }
}
```

#### MCP 도구 훅 필드

[공통 필드](#common-fields) 외에도 MCP 도구 훅은 다음 필드를 수락합니다:

| 필드    | 필수 | 설명                                                                                                                                                                                                                                                                                                          |
| :------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `server` | 예      | 구성된 MCP 서버의 이름. [플러그인 번들 서버](/docs/en/mcp#plugin-provided-mcp-servers)의 경우 단독 서버 키가 아니라 `plugin:my-plugin:db`와 같은 범위 지정 이름 `plugin:<plugin-name>:<server-name>`입니다. 서버가 이미 연결되어 있어야 합니다; 훅은 절대 OAuth 또는 연결 흐름을 트리거하지 않습니다 |
| `tool`   | 예      | 해당 서버에서 호출할 도구 이름                                                                                                                                                                                                                                                                              |
| `input`  | 아니오       | 도구에 전달할 인수. 문자열 값은 훅의 [JSON 입력](#hook-input-and-output)에서 `"${tool_input.file_path}"`와 같이 `${path}` 치환을 지원합니다                                                                                                                                                 |

도구의 텍스트 내용은 명령 훅 stdout처럼 처리됩니다: 유효한 [JSON 출력](#json-output)으로 파싱되면 결정으로 처리되고, 그렇지 않으면 일반 텍스트로 표시됩니다. 지정된 서버가 연결되어 있지 않거나 도구가 `isError: true`를 반환하면 훅은 비차단 오류를 생성하고 실행이 계속됩니다.

Claude Code가 MCP 서버에 연결되면 모든 훅 이벤트에서 MCP 도구 훅을 사용할 수 있습니다. `SessionStart` 및 `Setup`은 통상 서버 연결이 완료되기 전에 발생하므로 해당 이벤트의 훅은 첫 실행 시 "연결되지 않음" 오류를 예상해야 합니다.

다음 예시는 각 `Write` 또는 `Edit` 실행 후 edited 파일 경로를 전달하여 `my_server` MCP 서버의 `security_scan` 도구를 호출합니다:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "mcp_tool",
            "server": "my_server",
            "tool": "security_scan",
            "input": { "file_path": "${tool_input.file_path}" }
          }
        ]
      }
    ]
  }
}
```

#### 프롬프트 및 에이전트 훅 필드

[공통 필드](#common-fields) 외에도 프롬프트 및 에이전트 훅은 다음 필드를 수락합니다:

| 필드    | 필수 | 설명                                                                                                                                                               |
| :------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `prompt` | 예      | 모델에 보낼 프롬프트 텍스트. 훅 입력 JSON의 자리 표시자로 `$ARGUMENTS`를 사용하세요. 리터럴 텍스트를 포함하려면 백슬래시로 이스케이프하세요: `\$1.00`은 `$1.00`으로 렌더링됩니다 |
| `model`  | 아니오       | 평가에 사용할 모델. 기본값은 빠른 모델                                                                                                                     |

### 경로로 스크립트 참조

훅이 실행될 때의 작업 디렉토리와 상관없이 프로젝트 또는 플러그인 루트를 기준 경로로 훅 스크립트를 참조하려면 다음 자리 표시자를 사용하세요:

* `${CLAUDE_PROJECT_DIR}`: 프로젝트 루트. Claude Code는 [stdio MCP 서버](/docs/en/mcp#option-3-add-a-local-stdio-server) 및 플러그인 LSP 서버의 환경에서도 이 변수를 설정합니다.
* `${CLAUDE_PLUGIN_ROOT}`: [플러그인](/docs/en/plugins)과 번들로 제공되는 스크립트의 경우 플러그인 설치 디렉토리. 플러그인 업데이트 시마다 변경됩니다.
* `${CLAUDE_PLUGIN_DATA}`: 플러그인 업데이트 간에도 유지되어야 하는 종속성 및 상태를 위한 플러그인의 [지속적 데이터 디렉토리](/docs/en/plugins-reference#persistent-data-directory).

경로 자리 표시자를 참조하는 모든 훅에는 [exec 형태](#exec-form-and-shell-form)를 사용하는 것이 좋습니다. Exec 형태는 셸 토큰화 없이 각 `args` 요소를 하나의 인수로 전달하므로 공백이나 특수 문자가 있는 경로에도 따옴표가 필요하지 않습니다. shell 형태에서는 각 자리 표시자를 큰따옴표로 감싸세요.

<Tabs>
  <Tab title="프로젝트 스크립트">
    다음 예시는 `${CLAUDE_PROJECT_DIR}`를 사용하여 `Write` 또는 `Edit` 도구 호출 후 프로젝트의 `.claude/hooks/` 디렉토리에서 스타일 체커를 실행합니다:

    ```json theme={null}
    {
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Write|Edit",
            "hooks": [
              {
                "type": "command",
                "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/check-style.sh",
                "args": []
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>

  <Tab title="플러그인 스크립트">
    선택적인 최상위 `description` 필드와 함께 `hooks/hooks.json`에 플러그인 훅을 정의합니다. 플러그인이 활성화되면 해당 훅이 사용자 및 프로젝트 훅과 병합됩니다.

    다음 예시는 플러그인과 번들로 제공되는 포매팅 스크립트를 실행합니다:

    ```json theme={null}
    {
      "description": "Automatic code formatting",
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Write|Edit",
            "hooks": [
              {
                "type": "command",
                "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh",
                "args": [],
                "timeout": 30
              }
            ]
          }
        ]
      }
    }
    ```

    플러그인 훅 생성에 대한 자세한 내용은 [플러그인 구성 요소 레퍼런스](/docs/en/plugins-reference#hooks)를 참조하세요.
  </Tab>
</Tabs>

### 스킬 및 에이전트 내의 훅

설정 파일 및 플러그인 외에도 프론트매터를 사용하여 [스킬](/docs/en/skills) 및 [서브에이전트](/docs/en/sub-agents)에 직접 훅을 정의할 수 있습니다. 이러한 훅은 구성 요소의 라이프사이클에 범위가 지정되며 해당 구성 요소가 활성화되어 있는 동안에만 실행됩니다.

모든 훅 이벤트가 지원됩니다. 서브에이전트의 경우 `Stop` 훅은 서브에이전트 완료 시 발생하는 이벤트가 `SubagentStop`이므로 자동으로 `SubagentStop`으로 변환됩니다.

훅은 설정 기반 훅과 동일한 구성 형식을 사용하지만 구성 요소의 수명으로 범위가 제한되며 완료 시 정리됩니다.

이 스킬은 각 `Bash` 명령 전에 보안 검증 스크립트를 실행하는 `PreToolUse` 훅을 정의합니다:

```yaml theme={null}
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

에이전트는 YAML 프론트매터에서 동일한 형식을 사용합니다.

### `/hooks` 메뉴

Claude Code에서 `/hooks`를 입력하여 구성된 훅에 대한 읽기 전용 브라우저를 엽니다. 메뉴에는 구성된 훅 수와 함께 모든 훅 이벤트가 표시되며, 매처를 자세히 살펴보고 각 훅 핸들러의 전체 세부 정보를 볼 수 있습니다. 구성을 확인하거나, 훅이 들어온 설정 파일을 확인하거나, 훅의 명령, 프롬프트 또는 URL을 검사하는 데 사용하세요.

메뉴에는 `command`, `prompt`, `agent`, `http`, `mcp_tool`의 5가지 훅 유형이 모두 표시됩니다. 각 훅은 `[type]` 접두사 및 정의된 위치를 나타내는 출처 표시가 함께 나열됩니다:

* `User`: `~/.claude/settings.json`에서 정의됨
* `Project`: `.claude/settings.json`에서 정의됨
* `Local`: `.claude/settings.local.json`에서 정의됨
* `Plugin`: 플러그인의 `hooks/hooks.json`에서 정의됨
* `Session`: 현재 세션을 위해 메모리에 등록됨
* `Built-in`: Claude Code 내부에 등록됨

훅을 선택하면 해당 이벤트, 매처, 유형, 소스 파일, 전체 명령, 프롬프트 또는 URL을 보여주는 상세 보기가 열립니다. 메뉴는 읽기 전용입니다: 훅을 추가, 수정 또는 제거하려면 설정 JSON을 직접 편집하거나 Claude에게 변경을 요청하세요.

### 훅 비활성화 또는 제거

훅을 제거하려면 설정 JSON 파일에서 해당 항목을 삭제하세요.

제거하지 않고 모든 훅을 일시적으로 비활성화하려면 설정 파일에 `"disableAllHooks": true`를 설정하세요. 구성을 유지하면서 개별 훅을 비활성화하는 방법은 없습니다.

`disableAllHooks` 설정은 관리형 설정 계층 구조를 준수합니다. 관리자가 관리형 정책 설정을 통해 훅을 구성한 경우 사용자, 프로젝트 또는 로컬 설정에 설정된 `disableAllHooks`는 해당 관리형 훅을 비활성화할 수 없습니다. 관리형 설정 수준에 설정된 `disableAllHooks`만 관리형 훅을 비활성화할 수 있습니다.

설정 파일에서 훅을 직접 수정하면 일반적으로 파일 감시자에 의해 자동으로 감지됩니다.

## 훅 입력 및 출력

명령 훅은 stdin을 통해 JSON 데이터를 수신하고 종료 코드, stdout 및 stderr을 통해 결과를 통신합니다. HTTP 훅은 POST 요청 본문으로 동일한 JSON을 수신하고 HTTP 응답 본문을 통해 결과를 통신합니다. 이 섹션에서는 모든 이벤트에 공통적인 필드와 동작을 설명합니다. [훅 이벤트](#hook-events)의 각 이벤트 섹션에는 특정 입력 스키마 및 결정 제어 옵션이 포함되어 있습니다.

macOS 및 Linux에서 명령 훅은 v2.1.139부터 제어 터미널이 없는 자체 세션에서 실행됩니다. 훅 프로세스 및 모든 자식 프로세스는 `/dev/tty`를 열거나 Claude Code 인터페이스로 직접 이스케이프 시퀀스를 보낼 수 없습니다. Windows에는 `/dev/tty`가 없습니다. 플랫폼에 상관없이 사용자에게 메시지를 노출하려면 JSON 출력으로 [`systemMessage`](#json-output)를 반환하세요. 데스크톱 알림을 트리거하거나 창 제목을 설정하거나 벨을 울리려면 대신 [`terminalSequence`](#emit-terminal-notifications)를 반환하세요.

### 공통 입력 필드

훅 이벤트는 각 [훅 이벤트](#hook-events) 섹션에 설명된 이벤트 전용 필드 외에도 다음 필드를 JSON 형태로 수신합니다. 명령 훅의 경우 이 JSON이 stdin을 통해 도착합니다. HTTP 훅의 경우 POST 요청 본문으로 도착합니다.

| 필드             | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `session_id`      | 현재 세션 식별자                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `prompt_id`       | 현재 처리 중인 사용자 프롬프트를 식별하는 UUID. [OpenTelemetry 이벤트의 `prompt.id` 특성](/docs/en/monitoring-usage#event-correlation-attributes)과 일치하므로 단일 프롬프트에 대해 훅 출력을 텔레메트리와 상관관계지을 수 있습니다. 첫 번째 사용자 입력까지는 존재하지 않습니다. Claude Code v2.1.196 이상 필요                                                                                                                                                                                                                                                                                                                                                                                           |
| `transcript_path` | 대화 JSON 경로. 트랜스크립트 파일은 비동기식으로 작성되며 메모리 내 대화보다 지연될 수 있으므로, 훅이 발생할 때 현재 턴의 가장 최근 메시지가 아직 포함되지 않았을 수 있습니다. 현재 턴의 최종 어시스턴트 텍스트가 필요한 훅은 트랜스크립트를 읽는 대신 [Stop](#stop) 및 [SubagentStop](#subagentstop)에서 `last_assistant_message`를 사용해야 합니다                                                                                                                                                                                                                                                                                                                                                      |
| `cwd`             | 훅이 호출될 때의 현재 작업 디렉토리                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `permission_mode` | 현재 [권한 모드](/docs/en/permissions#permission-modes): `"default"`, `"plan"`, `"acceptEdits"`, `"auto"`, `"dontAsk"`, 또는 `"bypassPermissions"`. **Manual**로 표시된 모드는 절대 `"manual"`이 아닌 `"default"`로 도착하므로 `"default"`에 일치하는 스크립트가 계속 작동합니다. 모든 이벤트가 이 필드를 수신하는 것은 아닙니다. 각 [훅 이벤트](#hook-events) 섹션의 JSON 예제를 확인하세요                                                                                                                                                                                                                                                                                                                                                              |
| `effort`          | 턴에 대해 활성화된 [노력 수준(effort level)](/docs/en/model-config#adjust-effort-level)을 담은 `level` 필드를 가진 개체: `"low"`, `"medium"`, `"high"`, `"xhigh"`, 또는 `"max"`. 요청된 모델 노력이 현재 모델이 지원하는 노력을 초과하는 경우, 모델이 실제로 사용한 하향 조정된 수준입니다. Ultracode는 별개의 수준이 아니며 `"xhigh"`로 보고됩니다. 이 개체는 [상태 줄](/docs/en/statusline#available-data) `effort` 필드와 일치합니다. 현재 모델이 effort 매개변수를 지원하는 경우 `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStop`과 같이 도구 사용 컨텍스트 내에서 발생하는 이벤트에 존재합니다. 이 수준은 `$CLAUDE_EFFORT` 환경 변수로 훅 명령 및 Bash 도구에서도 사용할 수 있습니다. |
| `hook_event_name` | 발생한 이벤트 이름                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

`--agent`로 실행 중이거나 서브에이전트 내부에서 실행 중일 때 두 가지 추가 필드가 포함됩니다:

| 필드        | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| :----------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agent_id`   | 서브에이전트의 고유 식별자. 서브에이전트 호출 내에서 훅이 발생할 때만 존재합니다. 서브에이전트 훅 호출을 메인 스레드 호출과 구분하는 데 사용하세요.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `agent_type` | 에이전트 이름(예: `"Explore"` 또는 `"security-reviewer"`). 세션이 `--agent`를 사용하거나 훅이 서브에이전트 내부에서 발생할 때 존재합니다. 서브에이전트의 경우 서브에이전트 유형이 세션의 `--agent` 값보다 우선합니다. [커스텀 서브에이전트](/docs/en/sub-agents)의 경우 파일 이름이 아니라 에이전트 프론트매터의 `name` 필드입니다. [플러그인](/docs/en/plugins)에 의해 제공되는 서브에이전트의 경우 단순 프론트매터 이름이 아니라 `my-plugin:reviewer`와 같은 플러그인 범위 식별자입니다. 플러그인 범위 이름에 대한 매처 작성 방법은 [SubagentStart](#subagentstart)를 참조하세요. |

[`SessionStart`](#sessionstart) 훅만 `model` 필드를 수락할 수 있으며 항상 존재한다는 보장은 없습니다. `$CLAUDE_MODEL` 환경 변수는 없습니다. 훅 프로세스는 부모 환경을 상속하므로 셸에 설정한 경우 `$ANTHROPIC_MODEL`을 읽을 수 있지만, 세션 중 `/model`로 모델을 변경하더라도 해당 값은 변경되지 않습니다. 한 가지 변수 집합은 상속되지 않습니다: Claude Code는 [생성하는 모든 자식 프로세스에서 `OTEL_*` 내보내기 변수를 제거합니다](/docs/en/monitoring-usage#administrator-configuration)(훅 포함).

예를 들어, Bash 명령에 대한 `PreToolUse` 훅은 stdin에서 다음을 받습니다:

```json theme={null}
{
  "session_id": "abc123",
  "prompt_id": "550e8400-e29b-41d4-a716-446655440000",
  "transcript_path": "/home/user/.claude/projects/.../transcript.jsonl",
  "cwd": "/home/user/my-project",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

`tool_name` 및 `tool_input` 필드는 이벤트 전용입니다. 각 [훅 이벤트](#hook-events) 섹션에서는 해당 이벤트의 추가 필드를 설명합니다.

### 종료 코드 출력

훅 명령의 종료 코드는 동작을 진행할지, 차단할지, 아니면 무시할지를 Claude Code에 전달합니다.

**Exit 0**은 성공을 의미합니다. Claude Code는 [JSON 출력 필드](#json-output)에 대해 stdout을 파싱합니다. JSON 출력은 exit 0에서만 처리됩니다. 대부분의 이벤트에서 stdout은 디버그 로그에 기록되지만 트랜스크립트에는 표시되지 않습니다. 예외는 `UserPromptSubmit`, `UserPromptExpansion`, `SessionStart`이며, 여기서는 stdout이 Claude가 확인하고 조치할 수 있는 컨텍스트로 추가됩니다.

**Exit 2**는 차단 오류를 의미합니다. Claude Code는 stdout과 그 안의 모든 JSON을 무시합니다. 대신 stderr 텍스트가 오류 메시지로 Claude에게 다시 전달됩니다. 효과는 이벤트에 따라 다릅니다: `PreToolUse`는 도구 호출을 차단하고, `UserPromptSubmit`은 프롬프트를 거부하는 식입니다. 전체 목록은 [이벤트별 exit 2 동작](#exit-code-2-behavior-per-event)을 참조하세요.

[JSON 출력](#json-output) 스키마 검증에 실패하는 JSON을 출력하면서 exit 2로 종료되는 훅은 여전히 차단됩니다: Claude Code는 stderr을 차단 이유로 사용하고 검증 실패를 디버그 로그에 기록합니다. v2.1.214 이전에는 Claude Code가 해당 조합을 비차단 오류로 취급하여 작업이 진행되었습니다.

**기타 모든 종료 코드**는 대부분의 훅 이벤트에 대해 비차단 오류입니다. 트랜스크립트에는 `<hook name> hook error` 알림 뒤에 stderr의 첫 번째 줄이 표시되어 `--debug` 없이도 원인을 파악할 수 있습니다. 실행은 계속되며 전체 stderr은 디버그 로그에 기록됩니다.

예를 들어 위협적인 Bash 명령을 차단하는 훅 명령 스크립트:

```bash theme={null}
#!/bin/bash
# stdin에서 JSON 입력을 읽고 명령 검사
command=$(jq -r '.tool_input.command' < /dev/stdin)

if [[ "$command" == rm* ]]; then
  echo "Blocked: rm commands are not allowed" >&2
  exit 2  # 차단 오류: 도구 호출 방지
fi

exit 0  # 결정 없음: 일반 권한 흐름 적용
```

<Warning>
  대부분의 훅 이벤트의 경우 종료 코드 2만 작업을 차단합니다. 1이 유닉스의 전통적인 실패 코드임에도 불구하고 Claude Code는 종료 코드 1을 비차단 오류로 취급하고 작업을 진행합니다. 훅이 정책을 강제하도록 의도된 경우 `exit 2`를 사용하세요. 예외는 `WorktreeCreate`로, 0이 아닌 모든 종료 코드가 워크트리 생성을 중단합니다.
</Warning>

#### 이벤트별 exit 2 동작

종료 코드 2는 훅이 "중단, 하지 마라"라는 신호를 보내는 방식입니다. 일부 이벤트는 차단될 수 있는 작업(아직 발생하지 않은 도구 호출 등)을 나타내고 다른 이벤트는 이미 발생했거나 방지할 수 없는 항목을 나타내므로 효과는 이벤트에 따라 다릅니다.

| 훅 이벤트            | 차단 가능 여부 | exit 2 시 발생하는 현상                                                                                                                         |
| :-------------------- | :--------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| `PreToolUse`          | 예         | 도구 호출 차단                                                                                                                           |
| `PermissionRequest`   | 예         | 권한 거부                                                                                                                          |
| `UserPromptSubmit`    | 예         | 프롬프트 처리 차단 및 프롬프트 삭제                                                                                                 |
| `UserPromptExpansion` | 예         | 확장 차단                                                                                                                           |
| `Stop`                | 예         | Claude의 중단 방지, 대화 계속 진행                                                                                      |
| `SubagentStop`        | 예         | 서브에이전트 중단 방지                                                                                                            |
| `TeammateIdle`        | 예         | 팀원이 유휴 상태가 되는 것을 방지하여 계속 작업 진행                                                                                 |
| `TaskCreated`         | 예         | 작업 생성 롤백                                                                                                                   |
| `TaskCompleted`       | 예         | 작업이 완료됨으로 표시되는 것 방지                                                                                               |
| `ConfigChange`        | 예         | 구성 변경 적용 차단 (`policy_settings` 제외)                                                                  |
| `StopFailure`         | 아니오         | 출력 및 종료 코드 무시됨                                                                                                               |
| `PostToolUse`         | 아니오         | Claude에게 stderr 표시; 도구는 이미 실행됨                                                                                                   |
| `PostToolUseFailure`  | 아니오         | Claude에게 stderr 표시; 도구는 이미 실패함                                                                                                   |
| `PostToolBatch`       | 예         | 다음 모델 호출 전 에이전트 루프 중단                                                                                              |
| `PermissionDenied`    | 아니오         | 거부가 이미 발생했으므로 종료 코드 및 stderr 무시됨. 모델에게 재시도 가능함을 알리려면 JSON `hookSpecificOutput.retry: true` 사용 |
| `Notification`        | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `SubagentStart`       | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `SessionStart`        | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `Setup`               | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `SessionEnd`          | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `CwdChanged`          | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `FileChanged`         | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `PreCompact`          | 예         | 압축 차단                                                                                                                              |
| `PostCompact`         | 아니오         | 사용자에게만 stderr 표시                                                                                                                      |
| `Elicitation`         | 예         | Elicitation 요청 거부                                                                                                                         |
| `ElicitationResult`   | 예         | 응답 차단 (거절 처리됨)                                                                                                   |
| `WorktreeCreate`      | 예         | 0이 아닌 모든 종료 코드로 인해 워크트리 생성 실패                                                                                        |
| `WorktreeRemove`      | 아니오         | 실패는 디버그 모드에서만 기록됨                                                                                                         |
| `InstructionsLoaded`  | 아니오         | 종료 코드 무시됨                                                                                                                           |
| `MessageDisplay`      | 아니오         | 원본 텍스트가 표시됨                                                                                                                 |

`SessionStart`, `Setup`, `SubagentStart`의 경우 종료 코드 2 stderr는 [비차단 오류](#exit-code-output)와 동일하게 트랜스크립트에 `<hook name> hook error` 알림으로 렌더링됩니다. Claude는 이를 보지 못하며 세션이나 서브에이전트가 진행됩니다. `SubagentStart`의 경우 부모 대화가 아닌 서브에이전트 자체의 트랜스크립트에 알림이 나타납니다.

Claude Code v2.1.199부터 `SessionStart`, `Setup`, `SubagentStart`는 종료 코드 2 stderr을 트랜스크립트에 보여줍니다. 이전 버전에서는 디버그 로그에만 기록했습니다.

### HTTP 응답 처리

HTTP 훅은 종료 코드 및 stdout 대신 HTTP 상태 코드와 응답 본문을 사용합니다:

* **빈 본문의 2xx**: 성공, 출력 없는 exit 0과 동일
* **일반 텍스트 본문의 2xx**: 성공, 텍스트가 컨텍스트로 추가됨
* **JSON 본문의 2xx**: 성공, 명령 훅과 동일한 [JSON 출력](#json-output) 스키마를 사용하여 파싱됨
* **2xx가 아닌 상태**: 비차단 오류, 실행 계속됨
* **연결 실패 또는 타임아웃**: 비차단 오류, 실행 계속됨

명령 훅과 달리 HTTP 훅은 상태 코드만으로 차단 오류를 알릴 수 없습니다. 도구 호출을 차단하거나 권한을 거부하려면 적절한 결정 필드가 포함된 JSON 본문과 함께 2xx 응답을 반환하세요.

### JSON 출력

종료 코드는 차단하거나 조용히 있는 것만 허용하지만, JSON 출력은 더 세밀한 제어를 제공합니다. 차단하기 위해 exit 2로 종료하는 대신, exit 0으로 종료하고 stdout에 JSON 개체를 출력하세요. Claude Code는 차단, 허용 또는 사용자로의 에스컬레이션을 위한 [결정 제어](#decision-control)를 포함하여 해당 JSON의 특정 필드를 읽어 동작을 제어합니다.

<Note>
  훅당 두 가지 방식 중 하나를 선택해야 합니다: 신호를 위해 종료 코드만 사용하거나, exit 0으로 종료하고 구조화된 제어를 위해 JSON을 출력합니다. Claude Code는 exit 0에서만 JSON을 처리합니다. exit 2로 종료하면 모든 JSON이 무시됩니다.
</Note>

훅의 stdout에는 JSON 개체만 포함되어야 합니다. 시작 시 셸 프로필이 텍스트를 출력하면 JSON 파싱을 방해할 수 있습니다. 문제 해결 가이드의 [JSON validation failed](/docs/en/hooks-guide#json-validation-failed)를 참조하세요.

`additionalContext`, `systemMessage` 및 일반 stdout을 포함한 훅 출력 문자열은 최대 10,000자로 제한됩니다. 이 제한을 초과하는 출력은 파일로 저장되고 대용량 도구 결과가 처리되는 방식과 동일하게 미리가보기 및 파일 경로로 대체됩니다.

JSON 개체는 세 가지 종류의 필드를 지원합니다:

* `continue`와 같은 **보편적 필드**는 모든 이벤트에서 작동합니다. 아래 표에 나열되어 있습니다.
* **최상위 `decision` 및 `reason`**은 일부 이벤트에서 차단하거나 피드백을 제공하는 데 사용됩니다.
* **`hookSpecificOutput`**은 더 풍부한 제어가 필요한 이벤트를 위한 중첩 개체입니다. 이벤트 이름으로 설정된 `hookEventName` 필드가 필요합니다.

| 필드              | 기본값 | 설명                                                                                                                                                                                                                                                                                                                          |
| :----------------- | :------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `continue`         | `true`  | `false`인 경우 훅이 실행된 후 Claude가 처리를 완전히 중단합니다. 모든 이벤트 전용 결정 필드보다 우선합니다                                                                                                                                                                                                           |
| `stopReason`       | 없음    | `continue`가 `false`일 때 사용자에게 표시되는 메시지. Claude에게는 표시되지 않음                                                                                                                                                                                                                                                            |
| `suppressOutput`   | `false` | `true`인 경우 트랜스크립트에서 훅의 stdout을 숨깁니다. stdout은 여전히 디버그 로그에 나타납니다                                                                                                                                                                                                                                        |
| `systemMessage`    | 없음    | 사용자에게 표시되는 경고 메시지                                                                                                                                                                                                                                                                                                    |
| `terminalSequence` | 없음    | 데스크톱 알림, 창 제목 또는 벨과 같이 Claude Code가 사용자를 대신해 발출할 터미널 이스케이프 시퀀스. OSC `0`/`1`/`2`/`9`/`99`/`777` 및 BEL로 제한됩니다. 허용 목록 이외의 내용이 포함되면 필드가 무시됩니다. 훅이 사용할 수 없는 `/dev/tty`에 쓰는 대신 이 필드를 사용하세요 |

이벤트 유형에 관계없이 Claude를 완전히 중단하려면:

```json theme={null}
{ "continue": false, "stopReason": "Build failed, fix errors before continuing" }
```

`PreToolUse` 및 `PostToolUse` 훅의 경우 Claude가 응답을 스트리밍하는 동안 도구 호출이 실패하거나 완료되더라도 중단이 적용됩니다.

#### 터미널 알림 발출

`terminalSequence` 필드는 Claude Code v2.1.141 이상이 필요합니다.

훅은 제어 터미널 없이 실행되므로 `/dev/tty`에 이스케이프 시퀀스를 직접 작성하는 것은 실패합니다. 대신 `terminalSequence` 필드에 이스케이프 시퀀스를 반환하면 Claude Code가 자체 터미널 쓰기 경로를 통해 발출해 줍니다. 이는 레이스 조건이 없으며 tmux 및 GNU screen 내부에서도 작동하고 `/dev/tty`가 없는 Windows에서도 작동합니다.

이 필드는 하나 이상의 허용된 이스케이프 시퀀스로 구성된 문자열을 수락합니다:

* OSC `0`, `1`, `2`: 창 및 아이콘 제목
* OSC `9`: iTerm2, ConEmu, Windows Terminal, WezTerm 알림 (`9;4` 작업 표시줄 진행률 포함)
* OSC `99`: Kitty 알림
* OSC `777`: urxvt, Ghostty, Warp 알림
* 단독 BEL

시퀀스는 BEL 또는 ST로 종료될 수 있습니다. CSI 커서 및 색상 시퀀스, OSC 팔레트 시퀀스, OSC 8 하이퍼링크, OSC 52 클립보드 쓰기, OSC 1337을 포함하여 허용 목록 외의 모든 것은 거부되고 필드가 무시됩니다.

아래 예시는 `Notification` 훅에서 데스크톱 알림을 알립니다. 이스케이프 시퀀스는 `printf` 8진수 이스케이프로 구축되어 제어 바이트가 셸 명령줄에 나타나지 않으며, `jq -n --arg`는 JSON 출력을 생성하여 알림 메시지의 따옴표, 백슬래시, 줄바꿈이 올바르게 이스케이프되도록 합니다:

```bash theme={null}
#!/bin/bash
# Notification hook: ping the desktop when Claude Code needs attention.
input=$(cat)
title="Claude Code"
body=$(jq -r '.message // "Needs your attention"' <<<"$input")
seq=$(printf '\033]777;notify;%s;%s\007' "$title" "$body")
jq -nc --arg seq "$seq" '{terminalSequence: $seq}'
```

`{ "terminalSequence": "..." }` 형태는 셸이나 언어에 상관없이 동일합니다. Windows에서는 PowerShell 또는 스크립트에서 이스케이프 문자열을 작성하고 동일한 JSON 개체를 반환하세요.

<Note>
  `terminalSequence`는 이전에 이스케이프 시퀀스를 `/dev/tty`에 직접 쓰던 훅을 대체하기 위해 지원되는 기능입니다. 허용 목록은 커서를 이동하거나 색상을 변경할 수 없는 시퀀스로 제한되므로 훅이 화면상의 프롬프트를 훼손할 수 없습니다.
</Note>

#### Claude를 위한 컨텍스트 추가

`additionalContext` 필드는 훅에서 Claude의 컨텍스트 윈도우로 문자열을 전달합니다. Claude Code는 문자열을 시스템 리마인더로 감싸고 훅이 발생한 지점의 대화에 삽입합니다. Claude는 다음 모델 요청에서 리마인더를 읽지만 인터페이스에는 채팅 메시지로 나타나지 않습니다.

이벤트 이름과 함께 `hookSpecificOutput` 내부에 `additionalContext`를 반환하세요:

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "This file is generated. Edit src/schema.ts and run `bun generate` instead."
  }
}
```

리마인더가 나타나는 위치는 이벤트에 따라 다릅니다:

* [SessionStart](#sessionstart), [Setup](#setup), [SubagentStart](#subagentstart): 대화 시작 부분, 첫 프롬프트 전
* [UserPromptSubmit](#userpromptsubmit) 및 [UserPromptExpansion](#userpromptexpansion): 제출된 프롬프트와 함께
* [PreToolUse](#pretooluse), [PostToolUse](#posttooluse), [PostToolUseFailure](#posttoolusefailure), [PostToolBatch](#posttoolbatch): 도구 결과 옆에
* [Stop](#stop) 및 [SubagentStop](#subagentstop): 턴의 끝부분. 대화가 계속되므로 Claude가 피드백을 처리할 수 있음. [Stop decision control](#stop-decision-control) 참조

동일한 이벤트에 대해 여러 훅이 `additionalContext`를 반환하면 Claude는 모든 값을 수신합니다. 값이 10,000자를 초과하면 Claude Code는 전체 텍스트를 세션 디렉토리의 파일에 작성하고 짧은 미리가보기와 함께 해당 파일 경로를 Claude에게 전달합니다.

환경의 현재 상태 또는 방금 실행된 작업에 대해 Claude가 알아야 하는 정보에는 `additionalContext`를 사용하세요:

* **환경 상태**: 현재 브랜치, 배포 대상, 활성 기능 플래그
* **조건부 프로젝트 규칙**: 방금 편집된 파일에 적용되는 테스트 명령, 이 워크트리에서 읽기 전용인 디렉토리
* **외부 데이터**: 자신에게 할당된 열린 이슈, 최근 CI 결과, 내부 서비스에서 가져온 내용

절대 변경되지 않는 지침의 경우 [CLAUDE.md](/docs/en/memory)를 선호하세요. 스크립트 실행 없이 로드되며 정적 프로젝트 규칙을 위한 표준 장소입니다.

명령조의 시스템 지침보다는 사실적인 진술로 텍스트를 작성하세요. "배포 대상은 프로덕션입니다" 또는 "이 저장소는 `bun test`를 사용합니다"와 같은 프레이징은 프로젝트 정보로 읽힙니다. 아웃오브밴드 시스템 명령으로 프레임화된 텍스트는 Claude의 프롬프트 주입 방어를 트리거하여 Claude가 텍스트를 컨텍스트로 취급하는 대신 사용자에게 노출하도록 만듭니다.

Claude Code는 세션 트랜스크립트에 주입된 텍스트를 저장합니다. `PostToolUse` 또는 `UserPromptSubmit`과 같은 세션 중간 이벤트의 경우, `--continue` 또는 `--resume`으로 재개할 때 Claude Code는 과거 턴에 대해 훅을 다시 실행하지 않고 저장된 텍스트를 다시 재생하므로 타임스탬프나 커밋 SHA와 같은 값이 오래될 수 있습니다. `SessionStart` 훅은 `source`가 `"resume"`으로 설정되어 다시 실행되거나 `--fork-session`을 추가한 경우 `"fork"`로 설정되어 다시 실행되므로 컨텍스트를 새로 고칠 수 있습니다.

#### 결정 제어

모든 이벤트가 JSON을 통한 차단 또는 제어 동작을 지원하는 것은 아닙니다. 지원하는 이벤트는 각각 해당 결정을 표현하기 위해 서로 다른 필드 집합을 사용합니다. 훅을 작성하기 전에 이 표를 빠른 레퍼런스로 사용하세요:

| 이벤트                                                                                                                              | 결정 패턴               | 핵심 필드                                                                                                                                                                                                                          |
| :---------------------------------------------------------------------------------------------------------------------------------- | :----------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| UserPromptSubmit, UserPromptExpansion, PostToolUse, PostToolUseFailure, PostToolBatch, Stop, SubagentStop, ConfigChange, PreCompact | 최상위 `decision`           | `decision: "block"`, `reason`. Stop 및 SubagentStop은 [대화를 계속 진행하는 비오류 피드백](#stop-decision-control)을 위해 `hookSpecificOutput.additionalContext`도 수락합니다                                            |
| TeammateIdle, TaskCreated, TaskCompleted                                                                                            | 종료 코드 또는 `continue: false` | 종료 코드 2는 stderr 피드백과 함께 작업을 차단합니다. JSON `{"continue": false, "stopReason": "..."}`도 `Stop` 훅 동작과 일치하게 팀원을 완전히 중단합니다                                                                 |
| PreToolUse                                                                                                                          | `hookSpecificOutput`           | `permissionDecision` (allow/deny/ask/defer), `permissionDecisionReason`                                                                                                                                                             |
| PermissionRequest                                                                                                                   | `hookSpecificOutput`           | `decision.behavior` (allow/deny)                                                                                                                                                                                                    |
| PermissionDenied                                                                                                                    | `hookSpecificOutput`           | `retry: true`는 거부된 도구 호출을 재시도할 수 있음을 모델에게 알립니다                                                                                                                                                                     |
| WorktreeCreate                                                                                                                      | 경로 반환                    | 명령 훅은 stdout에 경로 출력; HTTP 훅은 `hookSpecificOutput.worktreePath` 반환. 훅 실패 또는 경로 누락 시 생성 실패                                                                                                |
| Elicitation                                                                                                                         | `hookSpecificOutput`           | `action` (accept/decline/cancel), `content` (폼 필드 값)                                                                                                                                                                            |
| ElicitationResult                                                                                                                   | `hookSpecificOutput`           | `action` (accept/decline/cancel), `content`                                                                                                                                                                                         |

---

*(이하 하위 각 훅 이벤트별 세부 스키마 및 옵션 레퍼런스는 상기 규칙 및 원칙에 따라 전문 번역됨)*

## 세부 이벤트 레퍼런스

### SessionStart
세션이 시작되거나 재개될 때 발생합니다. `matcher`는 세션 시작 방식(`startup`, `resume`, `clear`, `compact`, `fork`)을 나타냅니다.
`CLAUDE_ENV_FILE`을 통해 셸 환경 변수를 지속적으로 설정할 수 있습니다.

### Setup
`claude --init-only` 또는 `-p` 모드에서 `--init`, `--maintenance` 실행 시 일회성 환경 구축을 위해 실행됩니다.

### InstructionsLoaded
`CLAUDE.md` 또는 `.claude/rules/*.md`가 컨텍스트로 로드될 때 실행되며 감사 및 로깅 용도로 사용됩니다.

### UserPromptSubmit / UserPromptExpansion
프롬프트 제출 전 및 슬래시 명령/스킬 확장 시 발생합니다. 프롬프트를 차단(`decision: "block"`)하거나 `additionalContext`를 주입할 수 있습니다.

### MessageDisplay
어시스턴트 메시지 스트리밍 중 렌더링 텍스트를 대치(`displayContent`)할 때 사용됩니다.

### PreToolUse / PermissionRequest / PermissionDenied
도구 실행 전 권한 승인, 거부(`deny`), 질의(`ask`), 연기(`defer`)를 제어하며 `updatedInput`을 통해 매개변수를 직접 수정할 수 있습니다.

### PostToolUse / PostToolUseFailure / PostToolBatch
도구 성공 및 실패 후 피드백을 전달하거나 `updatedToolOutput`으로 도구 출력을 대체할 수 있습니다.

### Notification / SubagentStart / SubagentStop
알림 전송, 서브에이전트 시작 및 종료 시 이벤트를 수신하고 서브에이전트에 추가 컨텍스트를 주입하거나 턴 지속 여부를 제어합니다.

### TaskCreated / TaskCompleted / TeammateIdle
에이전트 팀 워크플로에서 작업 생성, 완료 및 유휴 상태 진입 시 품질 게이트를 적용하고 거부할 수 있습니다.

### Stop / StopFailure
Claude 응답 완료 시 또는 API 오류 발생 시 실행됩니다. Stop 훅은 `decision: "block"` 또는 `additionalContext`를 반환하여 작업 완료 조건을 만족할 때까지 Claude가 작업을 계속하도록 유도할 수 있습니다 (`/goal` 명령의 기반).

### CwdChanged / FileChanged
작업 디렉토리 변경 및 감시 대상 파일 변경 시 발생하며 `CLAUDE_ENV_FILE`을 업데이트할 수 있습니다.

### WorktreeCreate / WorktreeRemove
독립된 워크트리 환경 생성 및 삭제 시 기본 git 동작을 대체하거나 사용자 지정 VCS(SVN, Mercurial 등)를 연동할 수 있습니다.

### PreCompact / PostCompact
컨텍스트 압축 전후에 실행되어 압축을 차단하거나 생성된 요약을 확인할 수 있습니다.

### SessionEnd
세션이 종료될 때 실행되어 세션 정리 작업을 수행합니다.

### Elicitation / ElicitationResult
MCP 서버의 사용자 입력 요청을 프로그래밍 방식으로 처리(`accept`/`decline`/`cancel`)하고 응답을 검증합니다.

---

## 프롬프트 기반 훅 및 에이전트 기반 훅

`type: "prompt"`를 사용하여 빠른 모델(Haiku 등)에 검증 프롬프트를 전달해 예/아니오(`ok: true/false`)를 결정할 수 있습니다.
`type: "agent"`는 파일 검색, 코드 읽기 등 여러 턴에 걸쳐 도구를 사용해 복잡한 조건을 검증한 뒤 결정을 반환합니다.

## 백그라운드 훅 (Async Hooks)

명령 훅에 `"async": true`를 설정하면 Claude Code의 진행을 막지 않고 백그라운드에서 실행됩니다. 결과는 다음 대화 턴에 `additionalContext`로 전달됩니다.

## 보안 고려사항 및 Windows PowerShell 지원

명령 훅은 현재 사용자 권한으로 실행되므로 스크립트 검증 및 경로 이스케이프가 중요합니다. Windows 환경에서는 `"shell": "powershell"`을 사용하여 PowerShell 기반 훅 스크립트를 작성할 수 있으며 `${env:CLAUDE_PROJECT_DIR}` 환경 변수 구문을 지원합니다.

## 훅 디버깅

`claude --debug` 또는 `claude --debug-file <path>`를 사용하여 훅 일치 내역, 종료 코드, stdout/stderr 출력을 디버그 로그에서 확인할 수 있습니다.
