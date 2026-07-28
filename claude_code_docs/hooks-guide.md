> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 훅으로 작업 자동화하기

> Claude Code가 파일을 편집하거나 작업을 마치거나 입력을 필요로 할 때 셸 명령을 자동으로 실행하세요. 코드를 포맷팅하고, 알림을 보내고, 명령을 검증하며, 프로젝트 규칙을 적용합니다.

훅(Hook)은 Claude Code의 라이프사이클 중 특정 시점에 실행되는 사용자 정의 셸 명령입니다. 훅은 Claude Code의 동작을 결정론적으로 제어하여, LLM이 해당 실행을 선택할지 여부에 의존하지 않고 특정 작업이 항상 실행되도록 보장합니다. 훅을 사용하여 프로젝트 규칙을 적용하고, 반복 작업을 자동화하며, Claude Code를 기존 도구와 연동하세요.

결정론적 규칙이 아닌 판단이 필요한 결정의 경우, Claude 모델을 사용하여 조건을 평가하는 [프롬프트 기반 훅](#prompt-based-hooks) 또는 [에이전트 기반 훅](#agent-based-hooks)을 사용할 수도 있습니다.

Claude Code를 확장하는 다른 방법에 대해서는 Claude에게 추가 지침 및 실행 가능한 명령을 제공하는 [스킬](/docs/en/skills), 격리된 컨텍스트에서 작업을 실행하는 [서브에이전트](/docs/en/sub-agents), 프로젝트 간에 확장 기능을 공유하도록 패키징하는 [플러그인](/docs/en/plugins)을 참조하세요.

<Tip>
  이 가이드는 일반적인 사용 사례와 시작 방법을 다룹니다. 전체 이벤트 스키마, JSON 입력/출력 형식, 비동기 훅 및 MCP 도구 훅과 같은 고급 기능에 대해서는 [Hooks 레퍼런스](/docs/en/hooks)를 참조하세요.
</Tip>

## 첫 번째 훅 설정하기

훅을 생성하려면 [설정 파일](#configure-hook-location)에 `hooks` 블록을 추가하세요. 이 연습에서는 터미널을 계속 주시하는 대신 Claude가 사용자의 입력을 기다릴 때 알림을 받을 수 있도록 데스크톱 알림 훅을 생성합니다.

<Steps>
  <Step title="설정에 훅 추가">
    `~/.claude/settings.json`을 열고 `Notification` 훅을 추가합니다. 파일이 존재하지 않는 경우 생성하세요. 아래 예시는 macOS용 `osascript`를 사용합니다; Linux 및 Windows 명령은 [Claude가 입력을 필요로 할 때 알림 받기](#get-notified-when-claude-needs-input)를 참조하세요.

    ```json theme={null}
    {
      "hooks": {
        "Notification": [
          {
            "matcher": "",
            "hooks": [
              {
                "type": "command",
                "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
              }
            ]
          }
        ]
      }
    }
    ```

    설정 파일에 이미 `hooks` 키가 있는 경우 전체 개체를 교체하는 대신 기존 이벤트 키의 형제로 `Notification`을 추가하세요. 각 이벤트 이름은 단일 `hooks` 개체 내부의 키입니다:

    ```json theme={null}
    {
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Edit|Write",
            "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }]
          }
        ],
        "Notification": [
          {
            "matcher": "",
            "hooks": [{ "type": "command", "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'" }]
          }
        ]
      }
    }
    ```

    CLI에서 원하는 바를 설명하여 Claude에게 훅을 직접 작성하도록 요청할 수도 있습니다.
  </Step>

  <Step title="구성 확인">
    `/hooks`를 입력하여 훅 브라우저를 엽니다. 이용 가능한 모든 훅 이벤트 목록과 훅이 구성된 각 이벤트 옆에 개수가 표시됩니다. `Notification`을 선택하여 새 훅이 목록에 나타나는지 확인합니다. 훅을 선택하면 해당 상세 정보(이벤트, 매처, 유형, 소스 파일, 명령)가 표시됩니다.
  </Step>

  <Step title="훅 테스트">
    `Esc`를 눌러 CLI로 돌아갑니다. Claude에게 권한이 필요한 작업을 요청한 다음 터미널에서 다른 화면으로 전환하세요. 데스크톱 알림을 받게 됩니다.
  </Step>
</Steps>

<Tip>
  `/hooks` 메뉴는 읽기 전용입니다. 훅을 추가, 수정 또는 제거하려면 설정 JSON을 직접 편집하거나 Claude에게 변경을 요청하세요.
</Tip>

## 자동화할 수 있는 작업

훅을 사용하면 Claude Code 라이프사이클의 핵심 지점에서 코드를 실행할 수 있습니다: 편집 후 파일 포맷팅, 실행 전 명령 차단, Claude가 입력을 필요로 할 때 알림 전송, 세션 시작 시 컨텍스트 주입 등. 훅 이벤트의 전체 목록은 [Hooks 레퍼런스](/docs/en/hooks#hook-lifecycle)를 참조하세요.

각 예시에는 [설정 파일](#configure-hook-location)에 추가할 수 있는 바로 사용 가능한 구성 블록이 포함되어 있습니다.

별도의 모델 리뷰를 실행하고 결과를 세션으로 다시 전달하는 훅의 프로덕션 예시는 [`security-guidance` 플러그인이 Claude Code와 연동되는 방식](/docs/en/security-guidance#how-the-plugin-integrates-with-claude-code)을 참조하세요.

### Claude가 입력을 필요로 할 때 알림 받기

Claude가 작업을 마치고 사용자의 입력을 기다릴 때마다 데스크톱 알림을 받아, 터미널을 주시하지 않고도 다른 작업으로 전환할 수 있습니다.

이 훅은 Claude가 입력이나 권한을 기다릴 때 발생하는 `Notification` 이벤트를 사용합니다. 아래 각 탭은 해당 플랫폼의 네이티브 알림 명령을 사용합니다. 이를 `~/.claude/settings.json`에 추가하세요:

<Tabs>
  <Tab title="macOS">
    ```json theme={null}
    {
      "hooks": {
        "Notification": [
          {
            "matcher": "",
            "hooks": [
              {
                "type": "command",
                "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
              }
            ]
          }
        ]
      }
    }
    ```

    <Accordion title="알림이 나타나지 않는 경우">
      `osascript`는 내장 Script Editor 앱을 통해 알림을 전달합니다. Script Editor에 알림 권한이 없으면 명령이 무반응으로 실패하며 macOS가 권한 부여를 프롬프트하지 않습니다. 알림 설정에 Script Editor가 나타나도록 터미널에서 이를 한 번 실행하세요:

      ```bash theme={null}
      osascript -e 'display notification "test"'
      ```

      아직 아무것도 안 나타날 수 있습니다. **시스템 설정 > 알림**을 열고 목록에서 **Script Editor**를 찾아 **알림 허용**을 켭니다. 명령을 다시 실행하여 테스트 알림이 나타나는지 확인하세요.
    </Accordion>
  </Tab>

  <Tab title="Linux">
    ```json theme={null}
    {
      "hooks": {
        "Notification": [
          {
            "matcher": "",
            "hooks": [
              {
                "type": "command",
                "command": "notify-send 'Claude Code' 'Claude Code needs your attention'"
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>

  <Tab title="Windows (PowerShell)">
    ```json theme={null}
    {
      "hooks": {
        "Notification": [
          {
            "matcher": "",
            "hooks": [
              {
                "type": "command",
                "command": "powershell.exe -Command \"[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('Claude Code needs your attention', 'Claude Code')\""
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>
</Tabs>

비어 있는 `matcher`는 모든 알림 유형에서 실행됩니다. 특정 이벤트에서만 실행되도록 하려면 다음 값 중 하나로 설정하세요:

| 매처                | 실행 시점                                                                                               |
| :--------------------- | :------------------------------------------------------------------------------------------------------- |
| `permission_prompt`    | Claude가 도구 사용 승인을 필요로 할 때                                                                   |
| `idle_prompt`          | Claude가 작업을 마치고 다음 프롬프트를 기다릴 때                                                         |
| `auth_success`         | 인증이 완료될 때                                                                                 |
| `elicitation_dialog`   | MCP 서버가 Elicitation 폼을 열 때                                                                  |
| `elicitation_complete` | MCP Elicitation 폼이 제출되거나 닫힐 때                                                        |
| `elicitation_response` | MCP Elicitation 응답이 서버로 다시 전송될 때                                                   |
| `agent_needs_input`    | 백그라운드 세션이 사용자의 입력을 기다리기 시작할 때. [agent view](/docs/en/agent-view)가 열려 있는 동안에만 실행 |
| `agent_completed`      | 백그라운드 세션이 끝나거나 실패할 때. [agent view](/docs/en/agent-view)가 열려 있는 동안에만 실행            |

`agent_needs_input` 및 `agent_completed` 매처는 Claude Code v2.1.198 이상이 필요합니다.

`/hooks`를 입력하고 `Notification`을 선택하여 훅이 등록되었는지 확인하세요. 전체 이벤트 스키마는 [Notification 레퍼런스](/docs/en/hooks#notification)를 참조하세요.

### 편집 후 코드 자동 포맷팅

Claude가 편집하는 모든 파일에 대해 [Prettier](https://prettier.io/)를 자동으로 실행하여 수동 개입 없이 포맷팅을 일정하게 유지하세요.

이 훅은 `Edit|Write` 매처와 함께 `PostToolUse` 이벤트를 사용하여 파일 편집 도구 이후에만 실행됩니다. 명령은 [`jq`](https://jqlang.org/)로 편집된 파일 경로를 추출하여 Prettier에 전달합니다. 프로젝트 루트의 `.claude/settings.json`에 다음을 추가하세요:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

Claude Code v2.1.191 이상에서는 해당 버전의 도구 이름 매처에서 `|`와 `,`가 서로 교체 가능한 목록 구분 기호이므로 매처를 `Edit,Write`로 쓸 수도 있습니다.

<Note>
  이 페이지의 Bash 예제는 JSON 파싱을 위해 `jq`를 사용합니다. macOS에서는 `brew install jq`, Debian/Ubuntu에서는 `apt-get install jq`로 설치하거나 [`jq` downloads](https://jqlang.org/download/)를 참조하세요.
</Note>

### 보호된 파일 수정 차단

Claude가 `.env`, `package-lock.json` 또는 `.git/` 내부의 파일과 같은 민감한 파일을 수정하지 못하도록 방지하세요. Claude는 편집이 차단된 이유를 설명하는 피드백을 받으므로 접근 방식을 조정할 수 있습니다.

이 예시는 훅이 호출하는 별도의 스크립트 파일을 사용합니다. 스크립트는 대상 파일 경로를 보호된 패턴 목록과 비교하여 일치하면 종료 코드 2로 종료하여 편집을 차단합니다.

<Steps>
  <Step title="훅 스크립트 생성">
    다음 내용을 `.claude/hooks/protect-files.sh`에 저장합니다:

    ```bash theme={null}
    #!/bin/bash
    # protect-files.sh

    INPUT=$(cat)
    FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

    PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

    for pattern in "${PROTECTED_PATTERNS[@]}"; do
      if [[ "$FILE_PATH" == *"$pattern"* ]]; then
        echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
        exit 2
      fi
    done

    exit 0
    ```
  </Step>

  <Step title="macOS 및 Linux에서 스크립트를 실행 가능하도록 설정">
    Claude Code가 실행할 수 있도록 훅 스크립트는 실행 가능해야 합니다:

    ```bash theme={null}
    chmod +x .claude/hooks/protect-files.sh
    ```
  </Step>

  <Step title="훅 등록">
    `Edit` 또는 `Write` 도구 호출 전에 스크립트를 실행하는 `PreToolUse` 훅을 `.claude/settings.json`에 추가합니다:

    ```json theme={null}
    {
      "hooks": {
        "PreToolUse": [
          {
            "matcher": "Edit|Write",
            "hooks": [
              {
                "type": "command",
                "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh"
              }
            ]
          }
        ]
      }
    }
    ```
  </Step>
</Steps>

### 압축 후 컨텍스트 다시 주입

Claude의 컨텍스트 윈도우가 가득 차면 압축(compaction)을 통해 대화를 요약하여 공간을 확보합니다. 이 과정에서 중요한 세부 정보가 손실될 수 있습니다. 매번 압축 후 중요한 컨텍스트를 다시 주입하려면 `compact` 매처가 있는 `SessionStart` 훅을 사용하세요.

명령이 stdout에 쓰는 모든 텍스트는 Claude의 컨텍스트에 추가됩니다. 이 예시는 Claude에게 프로젝트 규칙 및 최근 작업 내용을 상기시킵니다. 프로젝트 루트의 `.claude/settings.json`에 다음을 추가하세요:

```json theme={null}
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing. Current sprint: auth refactor.'"
          }
        ]
      }
    ]
  }
}
```

`echo`를 최근 커밋을 보여주는 `git log --oneline -5`와 같이 동적 출력을 생성하는 명령으로 교체할 수 있습니다. 매 세션 시작 시 컨텍스트를 주입하려면 대신 [CLAUDE.md](/docs/en/memory)를 사용하는 것을 고려하세요. 환경 변수의 경우 레퍼런스의 [`CLAUDE_ENV_FILE`](/docs/en/hooks#persist-environment-variables)를 참조하세요.

### 구성 변경 사항 감사

세션 중 설정이나 스킬 파일이 변경되는 시점을 추적하세요. `ConfigChange` 이벤트는 외부 프로세스나 편집기가 구성 파일을 수정할 때 발생하므로 준수를 위해 변경 사항을 기록하거나 무단 수정을 차단할 수 있습니다.

이 예시는 각 변경 사항을 감사 로그에 추가합니다. 이를 `~/.claude/settings.json`에 추가하세요:

```json theme={null}
{
  "hooks": {
    "ConfigChange": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "jq -c '{timestamp: now | todate, source: .source, file: .file_path}' >> ~/claude-config-audit.log"
          }
        ]
      }
    ]
  }
}
```

매처는 구성 유형(`user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills`)으로 필터링합니다. 변경 사항 적용을 차단하려면 종료 코드 2로 종료하거나 `{"decision": "block"}`을 반환하세요. 전체 입력 스키마는 [ConfigChange 레퍼런스](/docs/en/hooks#configchange)를 참조하세요.

### 디렉토리 또는 파일 변경 시 환경 다시 로드

일부 프로젝트는 어느 디렉토리에 있는지에 따라 서로 다른 환경 변수를 설정합니다. [direnv](https://direnv.net/)와 같은 도구는 셸에서 이를 자동으로 처리하지만, Claude의 Bash 도구는 해당 변경 사항을 스스로 수집하지 않습니다.

`SessionStart` 훅과 `CwdChanged` 훅을 짝지어 사용하면 이 문제가 해결됩니다. `SessionStart`는 시작하는 디렉토리의 변수를 로드하고, `CwdChanged`는 Claude가 디렉토리를 변경할 때마다 해당 변수를 다시 로드합니다. 둘 다 Claude Code가 각 Bash 명령 전 스크립트 도입부로 실행하는 `CLAUDE_ENV_FILE`에 작성합니다. 이를 `~/.claude/settings.json`에 추가하세요:

```json theme={null}
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "direnv export bash > \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ],
    "CwdChanged": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "direnv export bash > \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  }
}
```

`.envrc`가 있는 각 디렉토리에서 `direnv allow`를 한 번 실행하여 direnv가 이를 로드하도록 허용하세요. direnv 대신 devbox나 nix를 사용하는 경우에도 `direnv export bash` 대신 `devbox shellenv` 또는 `devbox global shellenv`를 사용하여 동일한 패턴이 작동합니다.

모든 디렉토리 변경 대신 특정 파일에 반응하려면 `|`로 구분된 감시할 파일 이름을 나열한 `matcher`와 함께 `FileChanged`를 사용하세요. 감시 목록을 작성할 때 Claude Code는 이 값을 정규식으로 평가하지 않고 리터럴 파일 이름으로 분할합니다. 동일한 값이 파일 변경 시 실행할 훅 그룹을 필터링하는 방법은 [FileChanged](/docs/en/hooks#filechanged)를 참조하세요. 이 예시는 작업 디렉토리의 `.envrc` 및 `.env`를 감시합니다:

```json theme={null}
{
  "hooks": {
    "FileChanged": [
      {
        "matcher": ".envrc|.env",
        "hooks": [
          {
            "type": "command",
            "command": "direnv export bash > \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  }
}
```

입력 스키마, `watchPaths` 출력, `CLAUDE_ENV_FILE` 세부 정보는 [CwdChanged](/docs/en/hooks#cwdchanged) 및 [FileChanged](/docs/en/hooks#filechanged) 레퍼런스 항목을 참조하세요.

### 특정 권한 프롬프트 자동 승인

항상 허용하는 도구 호출에 대해서는 승인 대화 상자를 건너뛰세요. 이 예시는 플랜 제시를 마치고 진행 여부를 묻는 Claude의 도구인 `ExitPlanMode`를 자동 승인하여 플랜이 준비될 때마다 프롬프트가 표시되지 않도록 합니다.

위의 종료 코드 예시와 달리 자동 승인에는 훅이 stdout에 JSON 결정을 작성해야 합니다. Claude Code가 권한 대화 상자를 보여주려고 할 때 `PermissionRequest` 훅이 발생하며, `"behavior": "allow"`를 반환하여 사용자를 대신해 응답합니다.

매처는 훅 범주를 `ExitPlanMode`로만 제한하므로 다른 프롬프트에는 영향이 없습니다. 이를 `~/.claude/settings.json`에 추가하세요:

```json theme={null}
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'"
          }
        ]
      }
    ]
  }
}
```

훅이 승인하면 Claude Code는 플랜 모드를 종료하고 플랜 모드에 진입하기 전에 활성화되어 있던 권한 모드를 복원합니다. 대화 상자가 나타났을 위치에 트랜스크립트상으로 "Allowed by PermissionRequest hook"이 표시됩니다. 훅 경로는 항상 현재 대화를 유지합니다: 대화 상자가 할 수 있는 것처럼 컨텍스트를 지우고 새로운 구현 세션을 시작할 수는 없습니다.

대신 특정 권한 모드를 설정하려면 훅의 출력에 `setMode` 항목이 포함된 `updatedPermissions` 배열을 포함할 수 있습니다. `mode` 값은 `default`, `acceptEdits`, `bypassPermissions`와 같은 권한 모드이며, `destination: "session"`은 현재 세션에만 이를 적용합니다.

<Note>
  `bypassPermissions`는 세션이 이미 우회 모드를 사용할 수 있도록 시작된 경우에만 적용됩니다: `--dangerously-skip-permissions`, `--permission-mode bypassPermissions`, `--allow-dangerously-skip-permissions`, 또는 설정의 `permissions.defaultMode: "bypassPermissions"`가 적용되어 있고 [`permissions.disableBypassPermissionsMode`](/docs/en/permissions#managed-settings)에 의해 비활성화되지 않은 경우. 절대 `defaultMode`로 영구 저장되지 않습니다.
</Note>

세션을 `acceptEdits`로 전환하려면 훅이 stdout에 다음 JSON을 작성합니다:

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow",
      "updatedPermissions": [
        { "type": "setMode", "mode": "acceptEdits", "destination": "session" }
      ]
    }
  }
}
```

매처를 가능한 한 좁게 유지하세요. `.*`를 일치시키거나 매처를 비워두면 파일 쓰기 및 셸 명령을 포함한 모든 권한 프롬프트가 자동 승인됩니다. 모든 결정 필드 세트는 [PermissionRequest 레퍼런스](/docs/en/hooks#permissionrequest-decision-control)를 참조하세요.

## 훅 작동 원리

훅 이벤트는 Claude Code의 특정 라이프사이클 지점에서 발생합니다. 이벤트가 발생하면 일치하는 모든 훅이 병렬로 실행되며 동일한 훅 명령은 자동으로 중복 제거됩니다. 아래 표는 각 이벤트와 실행 트리거 시점을 보여줍니다:

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
| `TeammateIdle`        | [에이전트 팀](/docs/en/agent-teams) 팀원이 유휴 상태가 되려고 할 때                                                                                     |
| `InstructionsLoaded`  | CLAUDE.md 또는 `.claude/rules/*.md` 파일이 컨텍스트로 로드될 때. 세션 시작 시 및 세션 도중 지연 로드될 때 발생         |
| `ConfigChange`        | 세션 중 설정 파일이 변경될 때                                                                                                     |
| `CwdChanged`          | Claude가 `cd` 명령을 실행하는 등 작업 디렉토리가 변경될 때. direnv와 같은 도구로 반응형 환경 관리에 유용 |
| `FileChanged`         | 디스크에서 감시 중인 파일이 변경될 때. `matcher` 필드가 감시할 파일 이름을 지정함                                                            |
| `WorktreeCreate`      | `--worktree`, `isolation: "worktree"` 또는 백그라운드 세션을 위해 워크트리가 생성될 때. 기본 git 동작을 대체함                 |
| `WorktreeRemove`      | 세션 종료 시, 서브에이전트 종료 시 또는 백그라운드 세션을 삭제할 때 워크트리가 제거될 때                                    |
| `PreCompact`          | 컨텍스트 압축 전                                                                                                                              |
| `PostCompact`         | 컨텍스트 압축이 완료된 후                                                                                                                     |
| `Elicitation`         | 도구 호출 중 MCP 서버가 사용자 입력을 요청할 때                                                                                              |
| `ElicitationResult`   | 사용자가 MCP 요청에 응답한 후, 서버로 응답이 다시 전송되기 전                                                            |
| `SessionEnd`          | 세션이 종료될 때                                                                                                                              |

각 훅에는 실행 방식을 결정하는 `type`이 있습니다. 대부분의 훅은 셸 명령을 실행하는 `"type": "command"`를 사용합니다. 다른 네 가지 유형을 사용할 수 있습니다:

* `"type": "http"`: URL로 이벤트 데이터 POST. [HTTP 훅](#http-hooks)을 참조하세요.
* `"type": "mcp_tool"`: 이미 연결된 MCP 서버에서 도구 호출. [MCP 도구 훅](/docs/en/hooks#mcp-tool-hook-fields)을 참조하세요.
* `"type": "prompt"`: 단일 턴 LLM 평가. [프롬프트 기반 훅](#prompt-based-hooks)을 참조하세요.
* `"type": "agent"`: 도구 접근 권한을 가진 다중 턴 검증. 에이전트 훅은 실험적 기능이며 변경될 수 있습니다. [에이전트 기반 훅](#agent-based-hooks)을 참조하세요.

### 여러 훅의 결과 병합

여러 훅이 동일한 이벤트와 일치하면 Claude Code가 결과를 병합하기 전에 모든 훅의 명령이 완료될 때까지 실행됩니다. 하나의 훅이 `deny`를 반환한다고 해서 동료 훅의 실행이 중단되는 것은 아닙니다. 다른 훅의 부작용(side effect)을 차단하기 위해 한 훅의 `deny`에 의존하지 마세요.

일치하는 모든 훅이 완료된 후 Claude Code는 해당 출력을 결합합니다. `PreToolUse` 권한 결정의 경우 `deny`, `defer`, `ask`, `allow` 순서로 가장 제한적인 답변이 적용됩니다. `additionalContext`의 텍스트는 모든 훅에서 유지되어 Claude에게 함께 전달됩니다.

아래 예시는 `Bash`에 두 개의 `PreToolUse` 훅을 등록합니다. 첫 번째는 모든 명령을 로그 파일에 추가하고 0으로 종료합니다. 두 번째는 명령에 `rm -rf`가 포함되어 있을 때 2로 종료하여 거부하는 스크립트를 실행합니다:

```json theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r .tool_input.command >> ~/.claude/bash.log"
          },
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm-rf.sh"
          }
        ]
      }
    ]
  }
}
```

Claude가 `rm -rf /tmp/build`를 실행하려고 하면 두 훅이 모두 병렬로 실행됩니다. 로깅 훅은 명령을 `~/.claude/bash.log`에 쓰고 0으로 종료하며 결정 없음을 보고합니다. 가드레일 훅은 2로 종료하여 도구 호출을 거부합니다. 거부가 우선하므로 Claude Code는 명령을 차단하고 Claude에게 가드레일의 stderr을여 보여줍니다. 로깅 훅이 이미 실행되었으므로 로그 항목은 여전히 기록됩니다.

### 입력 읽기 및 출력 반환

훅은 stdin, stdout, stderr, 종료 코드를 통해 Claude Code와 통신합니다. 이벤트가 발생하면 Claude Code는 이벤트 전용 데이터를 JSON 형식으로 스크립트의 stdin에 전달합니다. 스립트는 해당 데이터를 읽고 작업을 수행한 다음 종료 코드를 통해 다음에 수행할 작업을 Claude Code에 전달합니다.

#### 훅 입력

모든 이벤트에는 `session_id` 및 `cwd`와 같은 공통 필드가 포함되어 있지만 각 이벤트 유형은 서로 다른 데이터를 추가합니다. 예를 들어 Claude가 Bash 명령을 실행할 때 `PreToolUse` 훅은 stdin에서 다음과 유사한 내용을 받습니다:

```json theme={null}
{
  "session_id": "abc123",          // 이 세션의 고유 ID
  "cwd": "/Users/sarah/myproject", // 이벤트 발생 시 작업 디렉토리
  "hook_event_name": "PreToolUse", // 이 훅을 트리거한 이벤트
  "tool_name": "Bash",             // Claude가 사용하려는 도구
  "tool_input": {                  // Claude가 도구에 전달한 인수
    "command": "npm test"          // Bash의 경우 셸 명령임
  }
}
```

스크립트는 해당 JSON을 파싱하고 해당 필드에 따라 조치를 취할 수 있습니다. `UserPromptSubmit` 훅은 대신 `prompt` 텍스트를 받고, `SessionStart` 훅은 `startup`, `resume`, `clear`, `compact`, `fork` 중 하나의 `source`를 받는 식입니다. 공유 필드는 레퍼런스의 [공통 입력 필드](/docs/en/hooks#common-input-fields)를, 이벤트별 스키마는 각 이벤트의 섹션을 참조하세요.

#### 훅 출력

스크립트는 stdout 또는 stderr에 쓰고 특정 코드로 종료하여 다음에 수행할 작업을 Claude Code에 안내합니다. 다음 `PreToolUse` 훅은 명령을 차단합니다:

```bash theme={null}
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q "drop table"; then
  echo "Blocked: dropping tables is not allowed" >&2  # stderr이 Claude의 피드백이 됨
  exit 2                                               # exit 2 = 작업 차단
fi

exit 0  # exit 0 = 결정 없음; 일반 권한 흐름 적용
```

종료 코드에 따라 다음에 일어날 일이 결정됩니다:

* **Exit 0**: 훅이 이의를 보고하지 않으며 작업이 정상적으로 진행됩니다. `PreToolUse` 훅의 경우 이것이 도구 호출을 승인하는 것은 아닙니다: 일반적인 [권한 흐름](/docs/en/permissions)이 여전히 적용됩니다. `UserPromptSubmit`, `UserPromptExpansion`, `SessionStart` 훅의 경우 stdout에 작성한 모든 내용이 Claude의 컨텍스트에 추가됩니다.
* **Exit 2**: 작업이 차단됩니다. stderr에 이유를 작성하면 Claude가 조정을 할 수 있도록 피드백으로 수신합니다. 일부 이벤트는 차단할 수 없습니다: `SessionStart`, `Setup`, `Notification` 등의 경우 exit 2는 사용자에게 stderr을 보여주고 실행이 계속됩니다. 전체 목록은 [이벤트별 exit 2 동작](/docs/en/hooks#exit-code-2-behavior-per-event)을 참조하세요.
* **기타 모든 종료 코드**: 작업이 진행됩니다. 트랜스크립트에 `<hook name> hook error` 알림 뒤에 stderr의 첫 번째 줄이 표시되고, 전체 stderr은 [디버그 로그](/docs/en/hooks#debug-hooks)로 전달됩니다.

#### 구조화된 JSON 출력

종료 코드는 차단하거나 조용히 있는 것만 허용합니다. 더 많은 제어를 위해서는 exit 0으로 종료하고 stdout에 JSON 개체를 출력하세요.

<Note>
  stderr 메시지와 함께 차단하려면 exit 2를 사용하고, 구조화된 제어를 위해서는 exit 0과 함께 JSON을 사용하세요. 이를 혼용하지 마세요: exit 2로 종료하면 Claude Code가 JSON을 무시합니다.
</Note>

예를 들어 `PreToolUse` 훅은 도구 호출을 거부하고 Claude에게 이유를 알려주거나 사용자의 승인을 위해 에스컬레이션할 수 있습니다:

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep for better performance"
  }
}
```

`"deny"`를 사용하면 Claude Code는 도구 호출을 취소하고 `permissionDecisionReason`을 Claude에게 다시 전달합니다. 이 `permissionDecision` 값은 `PreToolUse` 전용입니다:

* `"allow"`: 대화형 권한 프롬프트 건너뛰기. 엔터프라이즈 관리형 거부 목록을 포함한 거부 및 질의 규칙은 [조직이 `ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools) 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구에 대한 프롬프트와 마찬가지로 여전히 적용됩니다.
* `"deny"`: 도구 호출을 취소하고 Claude에게 이유 전송
* `"ask"`: 일반적인 권한 프롬프트를 사용자에게 표시

네 번째 값인 `"defer"`는 `-p` 플래그를 사용하는 [비대화형 모드](/docs/en/headless)에서 사용할 수 있습니다. 도구 호출이 보존된 상태로 프로세스를 종료하여 Agent SDK 래퍼가 입력을 수집하고 재개할 수 있도록 합니다. 레퍼런스의 [나중에 도구 호출 연기](/docs/en/hooks#defer-a-tool-call-for-later)를 참조하세요.

`"allow"`를 반환하면 대화형 프롬프트는 건너뛰지만 [권한 규칙](/docs/en/permissions#manage-permissions)을 재정의하지는 않습니다. 거부 규칙이 도구 호출과 일치하면 훅이 `"allow"`를 반환하더라도 호출이 차단됩니다. 질의 규칙이 일치하면 사용자는 여전히 프롬프트를 받으며, [조직이 `ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools) 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구도 마찬가지입니다. 이는 [관리형 설정](/docs/en/settings#settings-files)을 포함한 임의의 설정 범위의 거부 규칙이 항상 훅 승인보다 우선함을 의미합니다.

다른 이벤트는 다른 결정 패턴을 사용합니다. 예를 들어 `PostToolUse` 및 `Stop` 훅은 최상위 `decision: "block"` 필드를 사용하는 반면 `PermissionRequest`는 `hookSpecificOutput.decision.behavior`를 사용합니다. 이벤트별 전체 내역은 레퍼런스의 [요약 표](/docs/en/hooks#decision-control)를 참조하세요.

`UserPromptSubmit` 훅의 경우 대신 `hookSpecificOutput.additionalContext`를 사용하여 Claude의 컨텍스트에 텍스트를 주입하세요. `additionalContext`를 `hookSpecificOutput` 내부 중첩으로 배치하세요; JSON 최상위에 배치하면 Claude Code가 자동으로 무시합니다. 예를 들어 이 출력은 모든 프롬프트에 현재 브랜치 상태를 추가합니다:

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "Current branch: release-42. Deploy freeze until Friday."
  }
}
```

프롬프트 차단 및 세션 제목 설정을 포함한 전체 출력 형태는 [UserPromptSubmit decision control](/docs/en/hooks#userpromptsubmit-decision-control)을 참조하세요.

`type: "prompt"`가 적용된 훅은 출력을르게 처리합니다: [프롬프트 기반 훅](#prompt-based-hooks)을 참조하세요.

### 매처로 훅 필터링

매처가 없으면 훅은 해당 이벤트의 매 발생 시마다 실행됩니다. 매처를 사용하면 이를 좁힐 수 있습니다. 예를 들어 모든 도구 호출이 아닌 파일 편집 후에만 포매터를 실행하려는 경우 `PostToolUse` 훅에 매처를 추가하세요:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "prettier --write ..." }
        ]
      }
    ]
  }
}
```

`"Edit|Write"` 매처는 Claude가 `Bash`, `Read` 또는 기타 도구를 사용할 때가 아니라 `Edit` 또는 `Write` 도구를 사용할 때만 실행됩니다. Claude Code v2.1.191 이상에서는 해당 버전의 도구 이름 매처에서 쉼표가 대안을 동일하게 구분하므로 `"Edit, Write"`도 동일합니다. 일반 이름 및 정규식 평가 방식은 [매처 패턴](/docs/en/hooks#matcher-patterns)을 참조하세요.

<Note>
  Claude는 `Bash` 도구를 통해 셸 명령을 실행하여 파일을 생성하거나 수정할 수도 있습니다. 규정 준수 스캐닝이나 감사 로깅을 위해 훅이 모든 파일 변경 사항을 확인해야 하는 경우 턴당 한 번 작업 트리를 스캐닝하는 [`Stop`](/docs/en/hooks#stop) 훅을 추가하세요. 호출별 검사의 경우 `Bash`도 일치시키고 스크립트가 `git status --porcelain`으로 수정된 파일 및 추적되지 않는 파일을 나열하도록 하세요.
</Note>

각 이벤트 유형은 특정 필드에서 일치시킵니다:

| 이벤트                                                                                                                                                           | 매처가 필터링하는 대상                                              | 매처 값 예시                                                                                                                                                              |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`                                                                      | 도구 이름                                                             | `Bash`, `Edit\|Write`, `mcp__.*`                                                                                                                                                    |
| `SessionStart`                                                                                                                                                  | 세션 시작 방식                                               | `startup`, `resume`, `clear`, `compact`, `fork`                                                                                                                                     |
| `Setup`                                                                                                                                                         | 설정을 트리거한 CLI 플래그                               | `init`, `maintenance`                                                                                                                                                               |
| `SessionEnd`                                                                                                                                                    | 세션 종료 이유                                                 | `clear`, `resume`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other`                                                                                            |
| `Notification`                                                                                                                                                  | 알림 유형                                                     | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response`, `agent_needs_input`, `agent_completed`                    |
| `SubagentStart`                                                                                                                                                 | 에이전트 유형                                                            | `general-purpose`, `Explore`, `Plan` 또는 커스텀 에이전트 이름                                                                                                                         |
| `PreCompact`, `PostCompact`                                                                                                                                     | 압축 트리거 원인                                             | `manual`, `auto`                                                                                                                                                                    |
| `SubagentStop`                                                                                                                                                  | 에이전트 유형                                                            | `SubagentStart`와 동일한 값                                                                                                                                                      |
| `ConfigChange`                                                                                                                                                  | 구성 소스                                                  | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills`                                                                                                  |
| `StopFailure`                                                                                                                                                   | 오류 유형                                                            | `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, `unknown` |
| `InstructionsLoaded`                                                                                                                                            | 로드 이유                                                           | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact`                                                                                                        |
| `Elicitation`                                                                                                                                                   | MCP 서버 이름                                                       | 구성된 MCP 서버 이름                                                                                                                                                    |
| `ElicitationResult`                                                                                                                                             | MCP 서버 이름                                                       | `Elicitation`과 동일한 값                                                                                                                                                        |
| `FileChanged`                                                                                                                                                   | 감시할 리터럴 파일 이름 ([FileChanged](/docs/en/hooks#filechanged) 참조) | `.envrc\|.env`                                                                                                                                                                      |
| `UserPromptExpansion`                                                                                                                                           | 명령 이름                                                          | 스킬 또는 명령 이름                                                                                                                                                         |
| `UserPromptSubmit`, `PostToolBatch`, `Stop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `WorktreeCreate`, `WorktreeRemove`, `CwdChanged`, `MessageDisplay` | 매처 미지원                                                    | 발생할 때마다 항상 실행                                                                                                                                                    |

아래 탭은 서로 다른 이벤트 유형에 대한 몇 가지 매처 예시를 보여줍니다.

<Tabs>
  <Tab title="모든 Bash 명령 기록">
    `Bash` 도구 호출만 일치시키고 각 명령을 파일에 기록합니다. `PostToolUse` 이벤트는 명령이 완료된 후 발생하므로 `tool_input.command`에 실행된 내용이 포함됩니다. 훅은 stdin에서 이벤트 데이터를 JSON으로 수신하고 `jq -r '.tool_input.command'`가 명령 문자열만 추출하여 `>>`가 로그 파일에 추가합니다:

    ```json theme={null}
    {
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Bash",
            "hooks": [
              {
                "type": "command",
                "command": "jq -r '.tool_input.command' >> ~/.claude/command-log.txt"
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>

  <Tab title="MCP 도구 일치">
    MCP 도구는 내장 도구와 다른 명명 규칙을 사용합니다: `mcp__<server>__<tool>` (여기서 `<server>`는 MCP 서버 이름이고 `<tool>`은 제공하는 도구임). 예: `mcp__github__search_repositories` 또는 `mcp__filesystem__read_file`. [플러그인 번들 서버](/docs/en/mcp#plugin-provided-mcp-servers)의 도구는 대신 `mcp__plugin_my-plugin_db__query`와 같이 범위가 지정된 서버 세그먼트를 사용합니다. 특정 서버의 모든 도구를 대상으로 지정하려면 정규식 매처를 사용하거나 `mcp__.*__write.*`와 같은 패턴으로 여러 서버에 걸쳐 일치시키세요. 레퍼런스의 [Match MCP tools](/docs/en/hooks#match-mcp-tools)를 참조하세요.

    아래 명령은 `jq`로 훅의 JSON 입력에서 도구 이름을 추출하여 stderr에 작성합니다. stderr에 작성하면 stdout이 JSON 출력을 위해 깨끗하게 유지되고 메시지는 [디버그 로그](/docs/en/hooks#debug-hooks)로 보내집니다:

    ```json theme={null}
    {
      "hooks": {
        "PreToolUse": [
          {
            "matcher": "mcp__github__.*",
            "hooks": [
              {
                "type": "command",
                "command": "echo \"GitHub tool called: $(jq -r '.tool_name')\" >&2"
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>

  <Tab title="세션 종료 시 정리">
    `SessionEnd` 이벤트는 세션 종료 이유에 대한 매처를 지원합니다. 이 훅은 일반 종료가 아닌 `/clear` 실행 시 설정되는 `clear` 이유에서만 실행됩니다:

    ```json theme={null}
    {
      "hooks": {
        "SessionEnd": [
          {
            "matcher": "clear",
            "hooks": [
              {
                "type": "command",
                "command": "rm -f /tmp/claude-scratch-*.txt"
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>
</Tabs>

전체 매처 구문은 [Hooks 레퍼런스](/docs/en/hooks#configuration)를 참조하세요.

#### `if` 필드로 도구 이름 및 인수별 필터링

`if` 필드는 [권한 규칙 구문](/docs/en/permissions)을 사용하여 도구 호출이 일치할 때만 훅 프로세스가 생성되도록 도구 이름과 인수를 함께 필터링합니다. 이는 도구 이름만으로 그룹 수준에서 필터링하는 `matcher`를 넘어섭니다.

예를 들어 다음 구성은 Claude가 모든 Bash 명령이 아닌 `git` 명령을 사용할 때만 훅을 실행합니다:

```json theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(git *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-git-policy.sh"
          }
        ]
      }
    ]
  }
}
```

훅 명령의 실행 여부는 `if` 패턴의 형태와 Claude가 호출하는 Bash 명령에 따라 다릅니다:

| `if` 패턴       | Bash 명령           | 훅 실행 여부 | 이유                                                                                                 |
| :----------------- | :--------------------- | :--------- | :-------------------------------------------------------------------------------------------------- |
| `Bash(git *)`      | `git push`             | 예         | 명령 이름 일치함                                                                                |
| `Bash(git *)`      | `npm test && git push` | 예         | 각 하위 명령 검사됨; `git push`가 일치함                                                      |
| `Bash(git *)`      | `echo $(git log)`      | 예         | `$()` 및 백틱 내부의 명령 검사됨; `git log`가 일치함                                  |
| `Bash(git *)`      | `echo $(date)`         | 아니오         | `git *`와 일치하는 하위 명령 없음                                                                       |
| `Bash(git push *)` | `echo $(date)`         | 예         | 명령 이름 이상의 요구 사항을 지정하는 패턴은 `$()`, 백틱 또는 `$VAR`에서 어쨌든 훅을 실행함 |

또한 셸 명령을 파싱할 수 없는 경우 필터는 실패하더라도 안전을 위해 패턴과 상관없이 훅을 실행합니다. 필터는 최선의 노력(best-effort) 방식이므로 하드 허용 또는 거부를 강제하려면 훅 대신 [권한 시스템](/docs/en/permissions)을 사용하세요.

`if` 필드는 권한 규칙과 동일한 패턴(`"Bash(git *)"`, `"Edit(*.ts)"` 등)을 수락합니다. 여러 도구 이름을 일치시키려면 각각 고유한 `if` 값을 가진 별도의 핸들러를 사용하거나 파이프 교대가 지원되는 `matcher` 수준에서 일치시키세요.

`if`는 도구 이벤트(`PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`)에서만 작동합니다. 다른 이벤트에 추가하면 훅이 실행되지 않습니다.

### 훅 위치 구성

훅을 추가하는 위치에 따라 범주(scope)가 결정됩니다:

| 위치                                                       | 범주                               | 공유 가능 여부                             |
| :--------------------------------------------------------- | :--------------------------------- | :----------------------------------------- |
| `~/.claude/settings.json`                                  | 모든 프로젝트                      | 아니오, 로컬 컴퓨터 전용                  |
| `.claude/settings.json`                                    | 단일 프로젝트                     | 예, 저장소에 커밋 가능          |
| `.claude/settings.local.json`                              | 단일 프로젝트                     | 아니오, Claude Code가 생성 시 gitignore됨 |
| 관리형 정책 설정(Managed policy settings)                 | 조직 전체                          | 예, 관리자 제어                      |
| [플러그인](/docs/en/plugins) `hooks/hooks.json`           | 플러그인이 활성화되었을 때             | 예, 플러그인과 번들로 제공               |
| [스킬](/docs/en/skills) 또는 [에이전트](/docs/en/sub-agents) 프론트매터 | 스킬 또는 에이전트가 활성화되어 있는 동안 | 예, 구성 요소 파일에 정의됨         |

Claude Code에서 [`/hooks`](/docs/en/hooks#the-%2Fhooks-menu)를 실행하여 이벤트별로 그룹화되어 구성된 모든 훅을 찾아보세요.

훅을 비활성화하려면 설정 파일에 `"disableAllHooks": true`를 설정하세요. 관리형 설정에 구성된 훅은 `disableAllHooks`가 거기서도 설정되지 않는 한 계속 실행됩니다.

Claude Code가 실행 중일 때 설정 파일을 직접 편집하면 파일 감시자가 일반적으로 훅 변경 사항을 자동으로 감지합니다.

## 프롬프트 기반 훅

결정론적 규칙이 아닌 판단이 필요한 결정에는 `type: "prompt"` 훅을 사용하세요. 셸 명령을 실행하는 대신 Claude Code는 결정을 내리기 위해 프롬프트와 훅의 입력 데이터를 Claude 모델(기본값: Haiku)로 보냅니다. 더 많은 능력이 필요한 경우 `model` 필드로 다른 모델을 지정할 수 있습니다.

모델의 유일한 역할은 JSON으로 예/아니오 결정을 반환하는 것입니다:

* `"ok": true`: 작업 진행
* `"ok": false`: 이벤트에 따라 동작 결정:
  * `Stop` 및 `SubagentStop`: `reason`이 Claude에게 전달되어 작업을 계속함
  * `PreToolUse`: 도구 호출 거부됨; 기본적으로 턴이 끝나고 거부 `reason`이 채팅에 경고 줄로 표시됨. 도구 오류로 Claude에게 `reason`을 반환하여 조정 후 계속하도록 하려면 훅에 `continueOnBlock: true` 설정.
  * `PostToolUse`: 기본적으로 턴이 끝나고 `reason`이 채팅에 경고 줄로 표시됨. `continueOnBlock: true`를 설정하면 `reason`을 Claude에게 피드백하고 턴을 계속 진행함
  * `PostToolBatch`, `UserPromptSubmit`, `UserPromptExpansion`: 턴이 끝나고 `reason`이 채팅에 경고 줄로 표시됨

이 예시는 `Stop` 훅을 사용하여 요청된 모든 작업이 완료되었는지 모델에게 확인합니다. 모델이 `"ok": false`를 반환하면 Claude는 계속 작업하고 `reason`을 다음 지침으로 사용합니다:

```json theme={null}
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if all tasks are complete. If not, respond with {\"ok\": false, \"reason\": \"what remains to be done\"}."
          }
        ]
      }
    ]
  }
}
```

전체 구성 옵션은 레퍼런스의 [프롬프트 기반 훅](/docs/en/hooks#prompt-based-hooks)을 참조하세요.

## 에이전트 기반 훅

<Warning>
  에이전트 훅은 실험적 기능입니다. 향후 릴리스에서 동작 및 구성이 변경될 수 있습니다. 프로덕션 워크플로의 경우 [명령 훅](/docs/en/hooks#command-hook-fields)을 사용하세요.
</Warning>

검증 시 파일 검사나 명령 실행이 필요한 경우에는 `type: "agent"` 훅을 사용하세요. 단일 LLM 호출을 수행하는 프롬프트 훅과 달리 에이전트 훅은 결정을 반환하기전에 파일을 읽고 코드를 검색하고 기타 도구를 사용할 수 있는 서브에이전트를 생성합니다.

에이전트 훅은 프롬프트 훅과 동일한 `"ok"` / `"reason"` 응답 형식을 사용하지만 기본 타임아웃이 60초로 더 길고 최대 50회의 도구 사용 턴을 가집니다. 프롬프트의 `$ARGUMENTS` 자리 표시자는 훅의 JSON 입력으로 교체됩니다. [프롬프트 및 에이전트 훅 필드](/docs/en/hooks#prompt-and-agent-hook-fields)를 참조하세요.

이 예시는 Claude가 중단하도록 허용하기 전에 테스트가 통과하는지 검증합니다:

```json theme={null}
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results. $ARGUMENTS",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

훅 입력 데이터만으로 결정을 내리기에 충분할 때는 프롬프트 훅을 사용하세요. 코드베이스의 실제 상태에 대해 검증해야 할 때는 에이전트 훅을 사용하세요.

전체 구성 옵션은 레퍼런스의 [에이전트 기반 훅](/docs/en/hooks#agent-based-hooks)을 참조하세요.

## HTTP 훅

셸 명령을 실행하는 대신 HTTP 엔드포인트로 이벤트 데이터를 POST하려면 `type: "http"` 훅을 사용하세요. 엔드포인트는 명령 훅이 stdin에서 받는 것과 동일한 JSON을 수신하고, 동일한 JSON 형식을 사용하여 HTTP 응답 본문을 통해 결과를 반환합니다.

HTTP 훅은 웹 서버, 클라우드 함수 또는 외부 서비스가 훅 로직을 처리하도록 하려는 경우에 유용합니다(예: 팀 전체에서 도구 사용 이벤트를 기록하는 공유 감사 서비스).

이 예시는 모든 도구 사용을 로컬 로깅 서비스에 게시합니다:

```json theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/tool-use",
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

엔드포인트는 명령 훅과 동일한 [출력 형식](/docs/en/hooks#json-output)을 사용하여 JSON 응답 본문을 반환해야 합니다. 도구 호출을 차단하려면 적절한 `hookSpecificOutput` 필드와 함께 2xx 응답을 반환하세요. HTTP 상태 코드만으로는 작업을 차단할 수 없습니다.

헤더 값은 `$VAR_NAME` 또는 `${VAR_NAME}` 구문을 사용한 환경 변수 보간을 지원합니다. `allowedEnvVars` 배열에 나열된 변수만 해제되며 다른 모든 `$VAR` 참조는 빈 상태로 유지됩니다.

전체 구성 옵션 및 응답 처리는 레퍼런스의 [HTTP 훅](/docs/en/hooks#http-hook-fields)을 참조하세요.

## 제약 사항 및 문제 해결

### 제약 사항

훅을 설계할 때 다음 제약 사항을 염두에 두세요:

* 명령 훅은 stdout, stderr, 종료 코드로만 통신합니다. `/` 명령이나 도구 호출을 트리거할 수 없습니다. `additionalContext`를 통해 반환된 텍스트는 Claude가 일반 텍스트로 읽는 시스템 리마인더로 주입됩니다. HTTP 훅은 대신 응답 본문을 통해 통신합니다.
* 훅 타임아웃은 유형별로 다릅니다. 초 단위의 `timeout` 필드로 훅별로 재정의하세요.
  * `command`, `http`, `mcp_tool`: 10분. `UserPromptSubmit`은 이를 30초로 낮추고 `MessageDisplay`는 10초로 낮춥니다.
  * `prompt`: 30초.
  * `agent`: 60초.
* `PostToolUse` 훅은 도구가 이미 실행되었으므로 작업을 되돌릴 수 없습니다.
* `PermissionRequest` 훅은 `-p` 플래그를 사용하는 [비대화형 모드](/docs/en/headless)에서는 실행되지 않습니다. 자동화된 권한 결정을 위해 `PreToolUse` 훅을 사용하세요.
* `Stop` 훅은 작업 완료 시뿐만 아니라 Claude가 응답을 마칠 때마다 실행됩니다. 사용자가 중단할 때는 실행되지 않습니다. API 오류는 대신 [StopFailure](/docs/en/hooks#stopfailure)를 발생시킵니다.
* 여러 `PreToolUse` 훅이 도구의 인수를 재작성하기 위해 [`updatedInput`](/docs/en/hooks#pretooluse)을 반환할 때, 마지막으로 완료된 훅이 적용됩니다. 훅은 병렬로 실행되므로 순서가 비결정적입니다. 여러 훅이 동일한 도구의 입력을 수정하지 않도록 하세요.

### 훅 및 권한 모드

`PreToolUse` 훅은 `dontAsk`를 포함하여 모든 [권한 모드](/docs/en/permission-modes)에서 임의의 권한 모드 검사 전에 실행됩니다. `permissionDecision: "deny"`를 반환하는 훅은 `bypassPermissions` 모드나 `--dangerously-skip-permissions`에서도 도구를 차단합니다. 이를 통해 사용자가 권한 모드를 변경하여 우회할 수 없는 정책을 강제할 수 있습니다.

그 반대는 성립하지 않습니다: `"allow"`를 반환하는 훅이 설정의 거부 규칙을 우회하지 않으며, [조직이 `ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools)나 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구의 프롬프트를 억제할 수 없습니다. 훅은 제약을 강화할 수는 있지만 권한 규칙이 허용하는 범위를 넘어 완화할 수는 없습니다.

### 훅이 실행되지 않음

훅이 구성되었지만 절대 실행되지 않습니다.

* `/hooks`를 실행하고 훅이 올바른 이벤트 아래에 나타나는지 확인합니다.
* 매처 패턴이 도구 이름과 정확히 일치하는지 확인합니다. 매처는 대소문자를 구분합니다.
* 올바른 이벤트 유형을 트리거하는지 확인합니다: `PreToolUse`는 도구 실행 전에, `PostToolUse`는 도구 실행 후에 실행됩니다.
* 비대화형 모드(-p 플래그)에서 `PermissionRequest` 훅을 사용하는 경우 대신 `PreToolUse`로 전환하세요.

### 출력의 훅 오류

트랜스크립트에 "PreToolUse hook error: ..."와 같은 메시지가 표시됩니다.

* 스크립트가 예기치 않게 0이 아닌 코드로 종료되었습니다. 샘플 JSON을 파이프 연결하여 수동으로 테스트하세요:
  ```bash theme={null}
  echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./my-hook.sh
  echo $?  # 종료 코드 확인
  ```
* "command not found"가 표시되면 절대 경로나 `${CLAUDE_PROJECT_DIR}`을 사용하여 스크립트를 참조하세요. 셸 인용을 완전히 방지하려면 `"args": []`를 추가하여 셸 없이 스크립트를 직접 생성하는 [exec 형태](/docs/en/hooks#exec-form-and-shell-form)로 전환하세요.
* "jq: command not found"가 표시되면 `jq`를 설치하거나 JSON 파싱을 위해 Python/Node.js를 사용하세요.
* 스크립트가 전혀 실행되지 않는 경우 실행 가능하도록 설정하세요: `chmod +x ./my-hook.sh`

### `/hooks`에 구성된 훅이 표시되지 않음

설정 파일을 편집했지만 메뉴에 훅이 나타나지 않습니다.

* 파일 편집은 일반적으로 자동으로 감지됩니다. 몇 초 후에도 나타나지 않으면 파일 감시자가 변경 사항을 놓쳤을 수 있습니다: 세션을 다시 시작하여 강제로 다시 로드하세요.
* JSON이 유효한지 확인하세요: 후행 쉼표 및 주석은 허용되지 않습니다.
* 설정 파일이 올바른 위치에 있는지 확인하세요: 프로젝트 훅의 경우 `.claude/settings.json`, 글로벌 훅의 경우 `~/.claude/settings.json`.

### Stop 훅이 차단 상한에 도달함

Claude가 중단하지 않고 계속 작업하다가 Stop 훅이 연이은 차단을 너무 많이 수행했다는 경고와 함께 턴을 마칩니다.

Claude Code는 진행 없이 연속으로 8번 차단되면 Stop 훅을 재정의합니다. 훅 스크립트는 이미 계속을 트리거했는지 확인해야 합니다. JSON 입력에서 `stop_hook_active` 필드를 파싱하고 `true`이면 조기 종료하세요:

```bash theme={null}
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Claude가 중단되도록 허용
fi
# ... 나머지 훅 로직
```

훅이 수렴하기 위해 8번보다 많은 반복이 정당하게 필요한 경우 [`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`](/docs/en/env-vars)으로 상한을 올리세요.

### JSON 검증 실패

훅 스크립트가 유효한 JSON을 출력함에도 불구하고 Claude Code가 JSON 파싱 오류를 표시합니다.

Claude Code가 `args`가 없는 shell 형태 명령 훅을 실행할 때, 기본적으로 macOS 및 Linux에서는 `sh -c`, Windows에서는 Git Bash를 생성합니다. 이 셸은 비대화형이지만 Git Bash 및 일부 구성(예: `~/.bashrc`를 가리키는 `BASH_ENV`)은 프로필을 소스로 사용합니다. 해당 프로필에 조건 없는 `echo` 문이 포함되어 있으면 해당 출력이 훅의 JSON 앞에 붙습니다:

```text theme={null}
Shell ready on arm64
{"decision": "block", "reason": "Not allowed"}
```

Claude Code는 이를 JSON으로 파싱하려고 시도하다 실패합니다. 이를 수정하려면 셸 프로필의 echo 문을 감싸서 대화형 셸에서만 실행되도록 하세요:

```bash theme={null}
# ~/.zshrc 또는 ~/.bashrc 내부
if [[ $- == *i* ]]; then
  echo "Shell ready"
fi
```

`$-` 변수에는 셸 플래그가 포함되어 있으며 `i`는 대화형을 의미합니다. 훅은 비대화형 셸에서 실행되므로 echo가 건너뛰어집니다.

### 디버그 기법

`Ctrl+O`로 토글되는 트랜스크립트 보기는 발생한 각 훅에 대해 한 줄 요약을 보여줍니다: 성공은 자동, 차단 오류는 stderr 표시, 비차단 오류는 `<hook name> hook error` 알림 뒤에 stderr 첫 줄 표시.

어떤 훅이 일치했는지, 해당 종료 코드, stdout 및 stderr을 포함한 전체 실행 세부 정보는 디버그 로그를 읽으세요. Known 경로에 쓰려면 `claude --debug-file /tmp/claude.log`로 Claude Code를 시작한 뒤 다른 터미널에서 `tail -f /tmp/claude.log`를 실행하세요. 해당 플래그 없이 시작한 경우 세션 중간에 `/debug`를 실행하여 로깅을 활성화하고 로그 경로를 찾으세요.

## 자세히 알아보기

* [Hooks 레퍼런스](/docs/en/hooks): 전체 이벤트 스키마, JSON 출력 형식, 비동기 훅 및 MCP 도구 훅
* [보안 고려사항](/docs/en/hooks#security-considerations): 공유 또는 프로덕션 환경에 훅을 배포하기 전에 검토
* [Bash 명령 검증기 예시](https://github.com/anthropics/claude-code/blob/main/examples/hooks/bash_command_validator_example.py): 완전한 레퍼런스 구현
