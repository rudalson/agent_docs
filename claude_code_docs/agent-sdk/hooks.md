> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 후크(hooks)로 에이전트 동작 가로채기 및 제어하기

> 후크를 사용하여 주요 실행 지점에서 에이전트 동작을 가로채고 맞춤화합니다.

후크는 도구 호출, 세션 시작, 실행 중단 등 에이전트 이벤트에 대응하여 코드를 실행하는 콜백 함수입니다. 후크를 사용하여 다음을 수행할 수 있습니다:

* 파괴적인 쉘 명령이나 무단 파일 접근과 같은 **위험한 작업을 실행 전에 차단**
* 규정 준수, 디버깅 또는 분석을 위해 모든 도구 호출을 **로깅 및 감사**
* 데이터 정화, 자격 증명 주석, 파일 경로 리다이렉션을 위해 **입력 및 출력 변환**
* 데이터베이스 쓰기나 API 호출과 같은 민감한 작업에 **사람의 승인 요구**
* 상태 관리, 리소스 정리 또는 알림 전송을 위한 **세션 수명 주기 추적**

이 가이드에서는 도구 차단, 입력 수정, 알림 전달과 같은 일반적인 패턴의 예제와 함께 후크의 작동 방식 및 구성 방법을 다룹니다.

## 후크 작동 방식

<Steps>
  <Step title="이벤트 발생">
    에이전트 실행 중 이벤트가 발생하여 SDK가 이벤트를 방출합니다: 도구가 호출되려 함(`PreToolUse`), 도구가 결과를 반환함(`PostToolUse`), 서브에이전트가 시작되거나 중지됨, 에이전트가 유휴 상태임, 또는 실행이 완료됨. [전체 이벤트 목록](#사용-가능한-후크)을 참조하세요.
  </Step>

  <Step title="SDK가 등록된 후크 수집">
    SDK는 해당 이벤트 유형에 대해 등록된 후크를 확인합니다. 여기에는 `options.hooks`로 전달한 콜백 후크와 해당 [`settingSources`](/docs/en/agent-sdk/typescript#settingsource) 또는 [`setting_sources`](/docs/en/agent-sdk/python#settingsource) 항목이 활성화되었을 때(기본 `query()` 옵션에서 활성화됨) 설정 파일에 정의된 쉘 명령 후크가 포함됩니다.
  </Step>

  <Step title="매처(Matchers)가 실행할 후크 필터링">
    후크에 `"Write|Edit"`와 같은 [`matcher`](#매처matchers) 패턴이 있는 경우 SDK는 이벤트의 대상(예: 도구 이름)에 대해 이를 테스트합니다. 매처가 없는 후크는 해당 유형의 모든 이벤트에 대해 실행됩니다.
  </Step>

  <Step title="콜백 함수 실행">
    일치하는 각 후크의 [콜백 함수](#콜백-함수)는 도구 이름, 인자, 세션 ID 및 기타 이벤트별 세부 정보 등 진행 상황에 대한 입력을 수신합니다.
  </Step>

  <Step title="콜백이 결정 반환">
    작업(로깅, API 호출, 검증)을 수행한 후 콜백은 에이전트에게 지침을 전달하는 [출력 객체](#출력outputs)를 반환합니다: 작업 허용, 차단, 입력 수정 또는 대화에 컨텍스트 주입.
  </Step>
</Steps>

다음 예제는 이러한 단계를 결합합니다. 파일 쓰기 도구에 대해서만 콜백이 발생하도록 `"Write|Edit"` 매처(3단계)가 지정된 `PreToolUse` 후크(1단계)를 등록합니다. 트리거되면 콜백은 도구의 입력을 수신하고(4단계), 파일 경로가 `.env` 파일을 타겟팅하는지 확인하고, `permissionDecision: "deny"`를 반환하여 작업을 차단합니다(5단계):

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import (
      AssistantMessage,
      ClaudeSDKClient,
      ClaudeAgentOptions,
      HookMatcher,
      ResultMessage,
  )


  # 도구 호출 세부 정보를 받는 후크 콜백 정의
  async def protect_env_files(input_data, tool_use_id, context):
      # 도구의 입력 인자에서 파일 경로 추출
      file_path = input_data["tool_input"].get("file_path", "")
      file_name = file_path.split("/")[-1]

      # .env 파일을 타겟팅하는 경우 작업 차단
      if file_name == ".env":
          return {
              "hookSpecificOutput": {
                  "hookEventName": input_data["hook_event_name"],
                  "permissionDecision": "deny",
                  "permissionDecisionReason": "Cannot modify .env files",
              }
          }

      # 빈 객체를 반환하여 작업 허용
      return {}


  async def main():
      options = ClaudeAgentOptions(
          hooks={
              # PreToolUse 이벤트에 후크 등록
              # 매처는 Write 및 Edit 도구 호출로만 필터링함
              "PreToolUse": [HookMatcher(matcher="Write|Edit", hooks=[protect_env_files])]
          }
      )

      async with ClaudeSDKClient(options=options) as client:
          await client.query("Create a .env file with the standard local development database configuration")
          async for message in client.receive_response():
              # 어시스턴트 및 결과 메시지 필터링
              if isinstance(message, (AssistantMessage, ResultMessage)):
                  print(message)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query, HookCallback, PreToolUseHookInput } from "@anthropic-ai/claude-agent-sdk";

  // HookCallback 타입으로 후크 콜백 정의
  const protectEnvFiles: HookCallback = async (input, toolUseID, { signal }) => {
    // 타입 안정성을 위해 구체적인 후크 타입으로 입력 캐스팅
    const preInput = input as PreToolUseHookInput;

    // 속성에 접근하기 위해 tool_input 캐스팅 (SDK에서 unknown으로 입력됨)
    const toolInput = preInput.tool_input as Record<string, unknown>;
    const filePath = toolInput?.file_path as string;
    const fileName = filePath?.split("/").pop();

    // .env 파일을 타겟팅하는 경우 작업 차단
    if (fileName === ".env") {
      return {
        hookSpecificOutput: {
          hookEventName: preInput.hook_event_name,
          permissionDecision: "deny",
          permissionDecisionReason: "Cannot modify .env files"
        }
      };
    }

    // 빈 객체를 반환하여 작업 허용
    return {};
  };

  for await (const message of query({
    prompt: "Create a .env file with the standard local development database configuration",
    options: {
      hooks: {
        // PreToolUse 이벤트에 후크 등록
        // 매처는 Write 및 Edit 도구 호출로만 필터링함
        PreToolUse: [{ matcher: "Write|Edit", hooks: [protectEnvFiles] }]
      }
    }
  })) {
    // 어시스턴트 및 결과 메시지 필터링
    if (message.type === "assistant" || message.type === "result") {
      console.log(message);
    }
  }
  ```
</CodeGroup>

어느 스크립트든 실행하면 Claude가 `.env` 파일 생성을 시도하지만 후크가 도구 호출을 거부하고 Claude의 최종 응답에서 `.env` 파일을 생성할 수 없는 이유를 설명합니다.

## 사용 가능한 후크

SDK는 에이전트 실행의 여러 단계에 대한 후크를 제공합니다. 일부 후크는 두 SDK 모두에서 사용할 수 있고, 일부는 TypeScript 전용입니다.

| 후크 이벤트 | Python SDK | TypeScript SDK | 트리거 시점 | 주요 유즈케이스 예시 |
| ------------------------------------------------------ | ---------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `PreToolUse` | 예 | 예 | 도구 호출 요청 (차단 또는 수정 가능) | 위험한 쉘 명령 차단 |
| `PostToolUse` | 예 | 예 | 도구 실행 결과 반환 시 | 모든 파일 변경 사항을 감사 추적(audit trail)에 기록 |
| `PostToolUseFailure` | 예 | 예 | 도구 실행 실패 시 | 도구 오류 처리 또는 로깅 |
| `PostToolBatch` | 아니오 | 예 | 도구 호출의 전체 배치가 완료될 때(다음 모델 호출 전 배치당 1회) | 전체 배치를 위한 규칙/지침 한 번에 주입 |
| `UserPromptSubmit` | 예 | 예 | 사용자 프롬프트 제출 시 | 프롬프트에 추가 컨텍스트 주입 |
| [`UserPromptExpansion`](/docs/en/hooks#userpromptexpansion) | 아니오 | 예 | 사용자 입력 명령이나 MCP 프롬프트가 Claude에 도달하기 전 프롬프트로 확장될 때(Claude가 직접 스킬을 호출할 때는 발화하지 않음) | 직접적인 명령 호출을 차단하거나 스킬 입력 시 컨텍스트 추가 |
| `MessageDisplay` | 아니오 | 예 | 텍스트가 포함된 어시스턴트 메시지가 완료될 때(전체 메시지 텍스트와 함께 메시지당 1회) | 트랜스크립트 변경 없이 표시 텍스트 재구성 또는 민감 정보 마스킹 |
| `Stop` | 예 | 예 | 에이전트 실행 중단 시 | 종료 전 세션 상태 저장 |
| `StopFailure` | 아니오 | 예 | 턴이 정상 정지가 아닌 API 오류로 종료될 때 | 실패 로깅 또는 알림 전송 |
| `SubagentStart` | 예 | 예 | 서브에이전트 초기화 시 | 병렬 작업 생성 추적 |
| `SubagentStop` | 예 | 예 | 서브에이전트 완료 시 | 병렬 작업 결과 집계 |
| `PreCompact` | 예 | 예 | 대화 압축 요청 시 | 요약 전 전체 트랜스크립트 보관 |
| `PostCompact` | 아니오 | 예 | 대화 압축 완료 시 | 생성된 요약 로깅 |
| `PermissionRequest` | 예 | 예 | 권한 대화 상자가 표시되려 할 때 | 커스텀 권한 처리 |
| `PermissionDenied` | 아니오 | 예 | Auto 모드 분류기가 도구 호출을 거부할 때 | 분류기 거부 로깅 또는 모델에 재시도 가능 알림 |
| `SessionStart` | 아니오 | 예 | 세션 초기화 시 | 로깅 및 텔레메트리 초기화 |
| `SessionEnd` | 아니오 | 예 | 세션 종료 시 | 임시 리소스 정리 |
| `Notification` | 예 | 예 | 에이전트 상태 메시지 발생 시 | Slack 또는 PagerDuty로 에이전트 상태 업데이트 전송 |
| `Setup` | 아니오 | 예 | 세션 설정/유지보수 시 | 초기화 작업 실행 |
| `TeammateIdle` | 아니오 | 예 | 동료 에이전트(teammate)가 유휴 상태가 될 때 | 작업 재할당 또는 알림 |
| `TaskCreated` | 아니오 | 예 | `TaskCreate` 도구를 통해 작업이 생성될 때 | 작업 명명 규칙 강제 |
| `TaskCompleted` | 아니오 | 예 | 백그라운드 작업 완료 시 | 병렬 작업 결과 집계 |
| `Elicitation` | 아니오 | 예 | MCP 서버가 작업 중간에 사용자 입력을 요청할 때 | MCP 입력 요청에 프로그래밍 방식으로 응답 |
| `ElicitationResult` | 아니오 | 예 | 사용자가 MCP 이끌어내기(elicitation)에 응답할 때 | 서버로 돌아가기 전에 응답을 수정하거나 차단 |
| `ConfigChange` | 아니오 | 예 | 구성 파일 변경 시 | 설정을 동적으로 새로 고침 |
| `InstructionsLoaded` | 아니오 | 예 | `CLAUDE.md` 또는 규칙 파일이 컨텍스트에 로드될 때 | 로드되는 지침 파일 감사 |
| `WorktreeCreate` | 아니오 | 예 | Git 워크트리 생성 시 | 격리된 워크스페이스 추적 |
| `WorktreeRemove` | 아니오 | 예 | Git 워크트리 제거 시 | 워크스페이스 리소스 정리 |
| `CwdChanged` | 아니오 | 예 | 세션 중 작업 디렉토리가 변경될 때 | 디렉토리별 환경 변수 새로 고침 |
| `FileChanged` | 아니오 | 예 | 감시 대상 파일이 수정, 생성 또는 삭제될 때 | 프로젝트 파일 변경 시 구성 새로 고침 |

## 후크 구성하기

후크를 구성하려면 에이전트 옵션(Python의 `ClaudeAgentOptions`, TypeScript의 `options` 객체)의 `hooks` 필드로 전달하세요. 이 코드 조각은 앞선 예제의 Python `protect_env_files` 또는 TypeScript `protectEnvFiles`와 같은 후크 콜백이 이미 정의되어 있다고 가정합니다:

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      hooks={"PreToolUse": [HookMatcher(matcher="Bash", hooks=[my_callback])]}
  )

  async with ClaudeSDKClient(options=options) as client:
      await client.query("Your prompt")
      async for message in client.receive_response():
          print(message)
  ```

  ```typescript TypeScript theme={null}
  for await (const message of query({
    prompt: "Your prompt",
    options: {
      hooks: {
        PreToolUse: [{ matcher: "Bash", hooks: [myCallback] }]
      }
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

`hooks` 옵션은 Python에서는 딕셔너리, TypeScript에서는 객체이며 다음으로 구성됩니다:

* **키**: `'PreToolUse'`, `'PostToolUse'`, `'Stop'`과 같은 [후크 이벤트 이름](#사용-가능한-후크)
* **값**: 선택적 필터 패턴 및 사용자 [콜백 함수](#콜백-함수)를 포함하는 [매처](#매처matchers) 배열

### 매처(Matchers)

매처를 사용하여 콜백이 실행되는 시점을 필터링하세요. `matcher` 필드는 후크 이벤트 유형에 따라 서로 다른 값에 대해 일치합니다. 예를 들어 도구 기반 후크는 도구 이름에 대해 일치하고, `Notification` 후크는 알림 유형에 대해 일치합니다. 이벤트 유형별 매처 값의 전체 목록은 [Claude Code 후크 참조](/docs/en/hooks#matcher-patterns)를 참조하세요.

SDK 매처는 [설정 파일의 매처](/docs/en/hooks#matcher-patterns)와 동일한 규칙을 따릅니다. 문자의 숫자의 `_`, `-`, 공백, `,`, `|`만 포함하는 매처는 정확한 문자열로 비교되며 `|` 또는 `,`로 구별되는 대안과 선택적인 주변 공백을 포함합니다. 따라서 `Write|Edit`와 `Write, Edit`는 각각 정확히 해당 두 도구에만 일치하고 `code-reviewer`는 해당 에이전트 유형에만 일치합니다. `*`, 빈 문자열, 또는 매처 생략은 해당 이벤트의 모든 발생에 일치합니다.

다른 문자를 포함하는 매처는 앵커되지 않은 정규 표현식으로 평가되므로 `^mcp__`는 모든 MCP 도구에 일치하고 `Edit.*`는 `Edit` 및 `NotebookEdit` 모두에 일치합니다. 전체 문자열 일치가 필요한 경우 정규 표현식을 `^`와 `$`로 감싸세요.

`mcp__memory` 또는 `mcp__brave-search`와 같은 매처는 정확히 일치하는 문자만 포함하므로 정확한 문자열로 비교되어 어떠한 도구에도 일치하지 않습니다. 해당 서버의 모든 도구에 일치시키려면 `mcp__memory__.*`를 사용하세요.

정확한 일치 세트의 하이픈에는 v2.1.195 이상의 Claude Code 런타임이 필요합니다. 이전 버전에서는 `code-reviewer`와 같은 하이픈 이름이 앵커되지 않은 정규 표현식으로 평가되므로 정확히 일치시키려면 `^code-reviewer$`와 같이 앵커링해야 합니다.

`StopFailure` 및 `FileChanged`는 문자의 숫자의 `_`, `|`만으로 구성된 더 좁은 정확한 일치 세트를 사용합니다. 해당 두 이벤트의 매처에 하이픈, 공백, 쉼표가 포함되면 정규 표현식 경로로 유지되며 `|`만 대안을 구분하므로 `rate_limit, overloaded`가 아닌 `rate_limit|overloaded`로 작성하세요. `FileChanged`는 리터럴 파일 이름의 감시 목록을 작성하기 위해 추가로 해당 매처를 사용합니다. [후크 참조의 FileChanged](/docs/en/hooks#filechanged)를 참조하세요.

| 옵션 | 타입 | 기본값 | 설명 |
| --------- | ---------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `matcher` | `string` | `undefined` | 위 비교 규칙에 따라 이벤트의 필터 필드와 일치하는 패턴. 도구 후크의 경우 도구 이름임. 내장 도구에는 `Bash`, `Read`, `Write`, `Edit`, `Glob`, `Grep`, `WebFetch`, `Agent` 등이 포함됨(전체 목록은 [도구 입력 유형](/docs/en/agent-sdk/typescript#tool-input-types) 참조). MCP 도구는 `mcp__<server>__<action>` 패턴을 사용함. |
| `hooks` | `HookCallback[]` | - | 필수. 패턴이 일치할 때 실행할 콜백 함수 배열 |
| `timeout` | `number` | `undefined` | 초 단위 타임아웃. 생략 시 이벤트별 기본값이 적용됨: 대부분의 이벤트는 10분, `UserPromptSubmit`은 30초. `MessageDisplay`는 10초 등 일부 이벤트는 더 짧은 제한으로 실행됨 |

가능한 경우 항상 `matcher` 패턴을 사용하여 특정 도구를 타겟팅하세요. `'Bash'` 매처는 Bash 명령에 대해서만 실행되는 반면 패턴을 생략하면 해당 이벤트의 모든 발생에 대해 콜백이 실행됩니다.

도구 기반 후크의 경우 매처는 파일 경로나 기타 인자가 아닌 도구 이름으로만 필터링합니다. 파일 경로로 필터링하려면 콜백 내부에서 `tool_input.file_path`를 확인하세요.

<Tip>
  **도구 이름 탐색:** 내장 도구 이름의 전체 목록은 [도구 입력 유형](/docs/en/agent-sdk/typescript#tool-input-types)을 참조하거나 매처 없이 후크를 추가하여 세션이 수행하는 모든 도구 호출을 로깅해 보세요.

  **MCP 도구 명명 규칙:** MCP 도구는 항상 `mcp__`로 시작하고 서버 이름과 액션이 뒤따릅니다: `mcp__<server>__<action>`. 예를 들어 `playwright`라는 서버를 구성한 경우 해당 도구 이름은 `mcp__playwright__browser_screenshot`, `mcp__playwright__browser_click` 등이 됩니다. 서버 이름은 `mcpServers` 구성에서 사용하는 키에서 옵니다.
</Tip>

### 콜백 함수

#### 입력

모든 후크 콜백은 3개의 인자를 수신합니다:

* **입력 데이터:** 이벤트 세부 정보가 포함된 타입이 지정된 객체. 각 후크 유형에는 고유한 입력 형상이 있습니다. 예를 들어 `PreToolUseHookInput`에는 `tool_name` 및 `tool_input`이 포함되고, `NotificationHookInput`에는 `message`가 포함됩니다. 전체 타입 정의는 [TypeScript](/docs/en/agent-sdk/typescript#hookinput) 및 [Python](/docs/en/agent-sdk/python#hookinput) SDK 참조를 확인하세요.
  * 모든 후크 입력은 `session_id`, `cwd`, `hook_event_name`을 공유합니다.
  * 서브에이전트 내부에서 후크가 실행될 때 `agent_id` 및 `agent_type`이 채워집니다. TypeScript에서 이들은 기본 후크 입력에 존재하며 모든 후크 유형에서 사용할 수 있습니다. Python에서는 `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`에서는 선택적 필드이고 `SubagentStart` 및 `SubagentStop`에서는 필수 필드입니다.
* **도구 사용 ID** (`str | None` / `string | undefined`): 동일한 도구 호출에 대한 `PreToolUse` 및 `PostToolUse` 이벤트를 상호 연결합니다.
* **컨텍스트:** TypeScript에서는 취소를 위한 `signal` 속성(`AbortSignal`)을 포함합니다. Python에서 이 인자는 향후 사용을 위해 예약되어 있습니다.

#### 출력

콜백은 두 가지 범주의 필드가 포함된 객체를 반환합니다:

* **최상위 필드**는 모든 이벤트에서 동일하게 작동합니다: `systemMessage`는 사용자에게 메시지를 표출하고, `continue` (Python에서는 `continue_`)는 이 후크 실행 후 에이전트가 계속 실행될지를 결정합니다.
* **`hookSpecificOutput`**은 현재 작업을 제어합니다. 내부 필드는 후크 이벤트 유형에 따라 다릅니다. `PreToolUse` 후크의 경우 `permissionDecision` (`"allow"`, `"deny"`, `"ask"`, `"defer"`), `permissionDecisionReason`, `updatedInput`을 설정합니다. `"defer"`를 반환하면 [나중에 다시 시작](/docs/en/hooks#defer-a-tool-call-for-later)할 수 있도록 쿼리가 종료됩니다. `PostToolUse` 후크의 경우 `additionalContext`를 설정하여 도구 결과에 정보를 추가할 수 있습니다. Claude가 보기 전에 도구의 출력을 교체하려면 두 SDK의 모든 도구에서 작동하는 `updatedToolOutput`을 설정하세요. 기존 `updatedMCPToolOutput` 필드는 MCP 도구 출력만 교체하며 더 이상 사용되지 않습니다(deprecated).

변경 없이 작업을 허용하려면 `{}`를 반환하세요. SDK 콜백 후크는 [Claude Code 쉘 명령 후크](/docs/en/hooks#json-output)와 동일한 JSON 출력 형식을 사용하므로 모든 필드 및 이벤트별 옵션이 문서화되어 있습니다. SDK 타입 정의는 [TypeScript](/docs/en/agent-sdk/typescript#synchookjsonoutput) 및 [Python](/docs/en/agent-sdk/python#synchookjsonoutput) SDK 참조를 확인하세요.

<Note>
  여러 후크 또는 권한 규칙이 적용될 때 `deny`는 `defer`보다 우선하며, `defer`는 `ask`보다 우선하고, `ask`는 `allow`보다 우선합니다. 어느 후크 하나라도 `deny`를 반환하면 다른 후크의 반환값과 상관없이 작업이 차단됩니다.
</Note>

#### 비동기 출력

기본적으로 에이전트는 후크가 반환될 때까지 기다린 후 진행합니다. 후크가 로깅이나 웹후크 전송과 같은 사이드 이펙트를 수행하고 에이전트의 동작에 영향을 줄 필요가 없는 경우 비동기 출력을 반환할 수 있습니다. 이는 후크 완료를 기다리지 않고 에이전트가 즉시 계속 진행하도록 지시합니다. 다음 코드 조각에서 Python의 `send_to_logging_service` 및 TypeScript의 `sendToLoggingService`는 직접 정의하는 로깅 함수를 대신합니다:

<CodeGroup>
  ```python Python theme={null}
  async def async_hook(input_data, tool_use_id, context):
      # 백그라운드 작업을 시작하고 즉시 반환
      asyncio.create_task(send_to_logging_service(input_data))
      return {"async_": True, "asyncTimeout": 30000}
  ```

  ```typescript TypeScript theme={null}
  const asyncHook: HookCallback = async (input, toolUseID, { signal }) => {
    // 백그라운드 작업을 시작하고 즉시 반환
    sendToLoggingService(input).catch(console.error);
    return { async: true, asyncTimeout: 30000 };
  };
  ```
</CodeGroup>

| 필드 | 타입 | 설명 |
| -------------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `async` | `true` | 비동기 모드를 알림. 에이전트가 대기 없이 진행함. Python에서는 예약어를 피하기 위해 `async_` 사용. |
| `asyncTimeout` | `number` | 백그라운드 작업의 밀리초 단위 선택적 타임아웃 |

<Note>
  에이전트가 이미 계속 진행했으므로 비동기 출력은 작업을 차단, 수정 또는 컨텍스트 주입할 수 없습니다. 로깅, 지표, 알림과 같은 사이드 이펙트에만 사용하세요.
</Note>

## 예제

이 섹션의 여러 예제는 콜백 함수만 보여줍니다. 이를 실행하려면 [후크 구성하기](#후크-구성하기)에 표시된 대로 옵션의 `hooks` 필드에서 해당하는 이벤트 아래에 콜백을 등록하세요.

### 도구 입력 수정

이 예제는 Write 도구 호출을 가로채서 `file_path` 인자를 바꾸어 앞에 `/sandbox`를 붙임으로써 모든 파일 쓰기를 샌드박스 디렉토리로 리다이렉트합니다. 콜백은 수정된 경로와 수정된 작업을 자동 승인하기 위한 `permissionDecision: 'allow'`가 포함된 `updatedInput`을 반환합니다:

<CodeGroup>
  ```python Python theme={null}
  async def redirect_to_sandbox(input_data, tool_use_id, context):
      if input_data["hook_event_name"] != "PreToolUse":
          return {}

      if input_data["tool_name"] == "Write":
          original_path = input_data["tool_input"].get("file_path", "")
          return {
              "hookSpecificOutput": {
                  "hookEventName": input_data["hook_event_name"],
                  "permissionDecision": "allow",
                  "updatedInput": {
                      **input_data["tool_input"],
                      "file_path": f"/sandbox{original_path}",
                  },
              }
          }
      return {}
  ```

  ```typescript TypeScript theme={null}
  const redirectToSandbox: HookCallback = async (input, toolUseID, { signal }) => {
    if (input.hook_event_name !== "PreToolUse") return {};

    const preInput = input as PreToolUseHookInput;
    const toolInput = preInput.tool_input as Record<string, unknown>;
    if (preInput.tool_name === "Write") {
      const originalPath = toolInput.file_path as string;
      return {
        hookSpecificOutput: {
          hookEventName: preInput.hook_event_name,
          permissionDecision: "allow",
          updatedInput: {
            ...toolInput,
            file_path: `/sandbox${originalPath}`
          }
        }
      };
    }
    return {};
  };
  ```
</CodeGroup>

<Note>
  `updatedInput`을 사용할 때 수정된 입력을 자동 승인하려면 `permissionDecision: 'allow'`, 사용자에게 표시하려면 `permissionDecision: 'ask'`를 반드시 포함해야 합니다. `'defer'`를 사용하면 `updatedInput`이 무시됩니다. 원본 `tool_input`을 변경하기보다는 항상 새 객체를 반환하세요.
</Note>

### 컨텍스트 추가 및 도구 차단

이 예제는 `/etc` 디렉토리에 대한 쓰기를 차단하고 모델과 사용자 모두에게 이유를 설명합니다:

* `permissionDecision: 'deny'`는 도구 호출을 중단시킵니다.
* `permissionDecisionReason`은 모델에게 이유를 알려주어 재시도를 방지합니다.
* `systemMessage`는 사용자에게 발생한 상황을 보여줍니다.

<CodeGroup>
  ```python Python theme={null}
  async def block_etc_writes(input_data, tool_use_id, context):
      file_path = input_data["tool_input"].get("file_path", "")

      if file_path.startswith("/etc"):
          return {
              # 최상위 필드: 사용자에게 표시되는 메시지
              "systemMessage": "Remember: system directories like /etc are protected.",
              # hookSpecificOutput: 작업 차단
              "hookSpecificOutput": {
                  "hookEventName": input_data["hook_event_name"],
                  "permissionDecision": "deny",
                  "permissionDecisionReason": "Writing to /etc is not allowed",
              },
          }
      return {}
  ```

  ```typescript TypeScript theme={null}
  const blockEtcWrites: HookCallback = async (input, toolUseID, { signal }) => {
    const preInput = input as PreToolUseHookInput;
    const toolInput = preInput.tool_input as Record<string, unknown>;
    const filePath = toolInput?.file_path as string;

    if (filePath?.startsWith("/etc")) {
      return {
        // 최상위 필드: 사용자에게 표시되는 메시지
        systemMessage: "Remember: system directories like /etc are protected.",
        // hookSpecificOutput: 작업 차단
        hookSpecificOutput: {
          hookEventName: preInput.hook_event_name,
          permissionDecision: "deny",
          permissionDecisionReason: "Writing to /etc is not allowed"
        }
      };
    }
    return {};
  };
  ```
</CodeGroup>

### 특정 도구 자동 승인

기본적으로 에이전트는 특정 도구를 사용하기 전에 권한을 요청할 수 있습니다. 이 예제는 `permissionDecision: 'allow'`를 반환하여 읽기 전용 파일 시스템 도구(Read, Glob, Grep)를 자동 승인함으로써 사용자 확인 없이 실행할 수 있도록 허용하고, 다른 모든 도구는 일반적인 권한 확인을 거치도록 둡니다:

<CodeGroup>
  ```python Python theme={null}
  async def auto_approve_read_only(input_data, tool_use_id, context):
      if input_data["hook_event_name"] != "PreToolUse":
          return {}

      read_only_tools = ["Read", "Glob", "Grep"]
      if input_data["tool_name"] in read_only_tools:
          return {
              "hookSpecificOutput": {
                  "hookEventName": input_data["hook_event_name"],
                  "permissionDecision": "allow",
                  "permissionDecisionReason": "Read-only tool auto-approved",
              }
          }
      return {}
  ```

  ```typescript TypeScript theme={null}
  const autoApproveReadOnly: HookCallback = async (input, toolUseID, { signal }) => {
    if (input.hook_event_name !== "PreToolUse") return {};

    const preInput = input as PreToolUseHookInput;
    const readOnlyTools = ["Read", "Glob", "Grep"];
    if (readOnlyTools.includes(preInput.tool_name)) {
      return {
        hookSpecificOutput: {
          hookEventName: preInput.hook_event_name,
          permissionDecision: "allow",
          permissionDecisionReason: "Read-only tool auto-approved"
        }
      };
    }
    return {};
  };
  ```
</CodeGroup>

### 여러 후크 등록

이벤트가 발생하면 일치하는 모든 후크가 병렬로 실행됩니다. 권한 결정의 경우 가장 제한적인 결과가 적용됩니다: 다른 후크가 무엇을 반환하든 단 하나의 `deny`가 도구 호출을 차단합니다. 완료 순서가 비결정적이므로 다른 후크가 먼저 실행되었을 것으로 기대하지 말고 각 후크가 독립적으로 작용하도록 작성하세요.

아래 예제는 모든 도구 호출에 대해 3개의 독립적인 확인을 등록합니다:

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      hooks={
          "PreToolUse": [
              HookMatcher(hooks=[authorization_check]),
              HookMatcher(hooks=[input_validator]),
              HookMatcher(hooks=[audit_logger]),
          ]
      }
  )
  ```

  ```typescript TypeScript theme={null}
  const options = {
    hooks: {
      PreToolUse: [
        { hooks: [authorizationCheck] },
        { hooks: [inputValidator] },
        { hooks: [auditLogger] }
      ]
    }
  };
  ```
</CodeGroup>

### 다중 도구 매처로 필터링

관련 도구 전체에서 하나의 콜백을 공유하려면 다중 도구 매처를 사용하세요. 이 예제는 범위가 서로 다른 3개의 매처를 등록합니다:

* 파이프로 구분된 정확한 목록(`Write|Edit|NotebookEdit`)은 파일 수정 도구에 대해서만 `file_security_hook`을 트리거합니다.
* 정규 표현식(`^mcp__`)은 이름이 `mcp__`로 시작하는 모든 MCP 도구에 대해 `mcp_audit_hook`을 트리거합니다.
* 생략된 매처는 이름에 관계없이 모든 도구 호출에 대해 `global_logger`를 트리거합니다.

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      hooks={
          "PreToolUse": [
              # 파일 수정 도구 일치
              HookMatcher(matcher="Write|Edit|NotebookEdit", hooks=[file_security_hook]),
              # 모든 MCP 도구 일치
              HookMatcher(matcher="^mcp__", hooks=[mcp_audit_hook]),
              # 모든 것에 일치 (매처 없음)
              HookMatcher(hooks=[global_logger]),
          ]
      }
  )
  ```

  ```typescript TypeScript theme={null}
  const options = {
    hooks: {
      PreToolUse: [
        // 파일 수정 도구 일치
        { matcher: "Write|Edit|NotebookEdit", hooks: [fileSecurityHook] },

        // 모든 MCP 도구 일치
        { matcher: "^mcp__", hooks: [mcpAuditHook] },

        // 모든 것에 일치 (매처 없음)
        { hooks: [globalLogger] }
      ]
    }
  };
  ```
</CodeGroup>

### 서브에이전트 활동 추적

서브에이전트가 작업을 마치는 시점을 모니터링하려면 `SubagentStop` 후크를 사용하세요. 전체 입력 타입은 [TypeScript](/docs/en/agent-sdk/typescript#hookinput) 및 [Python](/docs/en/agent-sdk/python#hookinput) SDK 참조를 확인하세요. 이 예제는 서브에이전트가 완료될 때마다 요약을 기록합니다:

<CodeGroup>
  ```python Python theme={null}
  async def subagent_tracker(input_data, tool_use_id, context):
      # 완료 시 서브에이전트 세부 정보 로깅
      print(f"[SUBAGENT] Completed: {input_data['agent_id']}")
      print(f"  Transcript: {input_data['agent_transcript_path']}")
      print(f"  Tool use ID: {tool_use_id}")
      print(f"  Stop hook active: {input_data.get('stop_hook_active')}")
      return {}


  options = ClaudeAgentOptions(
      hooks={"SubagentStop": [HookMatcher(hooks=[subagent_tracker])]}
  )
  ```

  ```typescript TypeScript theme={null}
  import { HookCallback, SubagentStopHookInput } from "@anthropic-ai/claude-agent-sdk";

  const subagentTracker: HookCallback = async (input, toolUseID, { signal }) => {
    // 서브에이전트 전용 필드에 접근하기 위해 SubagentStopHookInput으로 캐스팅
    const subInput = input as SubagentStopHookInput;

    // 완료 시 서브에이전트 세부 정보 로깅
    console.log(`[SUBAGENT] Completed: ${subInput.agent_id}`);
    console.log(`  Transcript: ${subInput.agent_transcript_path}`);
    console.log(`  Tool use ID: ${toolUseID}`);
    console.log(`  Stop hook active: ${subInput.stop_hook_active}`);
    return {};
  };

  const options = {
    hooks: {
      SubagentStop: [{ hooks: [subagentTracker] }]
    }
  };
  ```
</CodeGroup>

### 후크에서 HTTP 요청 수행하기

후크는 HTTP 요청과 같은 비동기 작업을 수행할 수 있습니다. 처리되지 않은 예외는 에이전트를 중단시킬 수 있으므로 후크 내부에서 오류를 포착하세요.

이 예제는 각 도구가 완료된 후 웹후크를 전송하여 실행된 도구와 시점을 기록합니다. 웹후크 실패로 인해 에이전트가 중단되지 않도록 오류를 포착합니다:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  import json
  import urllib.request
  from datetime import datetime


  def _send_webhook(tool_name):
      """외부 웹후크로 도구 사용 데이터를 POST하는 동기식 헬퍼 함수."""
      data = json.dumps(
          {
              "tool": tool_name,
              "timestamp": datetime.now().isoformat(),
          }
      ).encode()
      req = urllib.request.Request(
          "https://api.example.com/webhook",
          data=data,
          headers={"Content-Type": "application/json"},
          method="POST",
      )
      urllib.request.urlopen(req)


  async def webhook_notifier(input_data, tool_use_id, context):
      # 도구가 완료된 후(PostToolUse)에만 전송하고 실행 전에는 전송하지 않음
      if input_data["hook_event_name"] != "PostToolUse":
          return {}

      try:
          # 이벤트 루프 차단을 피하기 위해 스레드에서 차단식 HTTP 호출 실행
          await asyncio.to_thread(_send_webhook, input_data["tool_name"])
      except Exception as e:
          # 오류를 기록하되 예외를 던지지 않음. 실패한 웹후크가 에이전트를 멈춰서는 안 됨
          print(f"Webhook request failed: {e}")

      return {}
  ```

  ```typescript TypeScript theme={null}
  import { query, HookCallback, PostToolUseHookInput } from "@anthropic-ai/claude-agent-sdk";

  const webhookNotifier: HookCallback = async (input, toolUseID, { signal }) => {
    // 도구가 완료된 후(PostToolUse)에만 전송하고 실행 전에는 전송하지 않음
    if (input.hook_event_name !== "PostToolUse") return {};

    try {
      await fetch("https://api.example.com/webhook", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          tool: (input as PostToolUseHookInput).tool_name,
          timestamp: new Date().toISOString()
        }),
        // 후크 시간이 초과할 경우 요청이 취소되도록 signal 전달
        signal
      });
    } catch (error) {
      // 취소 오류를 다른 오류와 구별하여 처리
      if (error instanceof Error && error.name === "AbortError") {
        console.log("Webhook request cancelled");
      }
      // 예외를 다시 던지지 않음. 실패한 웹후크가 에이전트를 멈춰서는 안 됨
    }

    return {};
  };

  // PostToolUse 후크로 등록
  for await (const message of query({
    prompt: "Refactor the auth module",
    options: {
      hooks: {
        PostToolUse: [{ hooks: [webhookNotifier] }]
      }
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

### Slack으로 알림 전달하기

`Notification` 후크를 사용하여 에이전트로부터 시스템 알림을 수신하고 이를 외부 서비스로 전달하세요. 알림은 다음과 같은 이벤트 유형에서 발생합니다:

* Claude에게 권한이 필요할 때 `permission_prompt`
* Claude가 입력을 기다릴 때 `idle_prompt`
* 인증이 완료되었을 때 `auth_success`
* 사용자 프롬프트 이끌어내기 흐름을 위한 `elicitation_dialog`, `elicitation_complete`, `elicitation_response`

각 알림에는 사람이 읽을 수 있는 설명이 담긴 `message` 필드와 선택적 `title`이 포함되어 있습니다.

이 예제는 모든 알림을 Slack 채널로 전달합니다. Slack 워크스페이스에 앱을 추가하고 수신 웹후크를 활성화하여 생성하는 [Slack 수신 웹후크 URL](https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks/)이 필요합니다:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  import json
  import urllib.request

  from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, HookMatcher


  def _send_slack_notification(message):
      """수신 웹후크를 통해 Slack으로 메시지를 보내는 동기식 헬퍼 함수."""
      data = json.dumps({"text": f"Agent status: {message}"}).encode()
      req = urllib.request.Request(
          "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
          data=data,
          headers={"Content-Type": "application/json"},
          method="POST",
      )
      urllib.request.urlopen(req)


  async def notification_handler(input_data, tool_use_id, context):
      try:
          # 이벤트 루프 차단을 피하기 위해 스레드에서 차단식 HTTP 호출 실행
          await asyncio.to_thread(_send_slack_notification, input_data.get("message", ""))
      except Exception as e:
          print(f"Failed to send notification: {e}")

      # 빈 객체 반환. 알림 후크는 에이전트 동작을 수정하지 않음
      return {}


  async def main():
      options = ClaudeAgentOptions(
          hooks={
              # Notification 이벤트에 후크 등록 (매처 필요 없음)
              "Notification": [HookMatcher(hooks=[notification_handler])],
          },
      )

      async with ClaudeSDKClient(options=options) as client:
          await client.query("Analyze this codebase")
          async for message in client.receive_response():
              print(message)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query, HookCallback, NotificationHookInput } from "@anthropic-ai/claude-agent-sdk";

  // Slack으로 알림을 보내는 후크 콜백 정의
  const notificationHandler: HookCallback = async (input, toolUseID, { signal }) => {
    // message 필드에 접근하기 위해 NotificationHookInput으로 캐스팅
    const notification = input as NotificationHookInput;

    try {
      // Slack 수신 웹후크로 알림 메시지 POST
      await fetch("https://hooks.slack.com/services/YOUR/WEBHOOK/URL", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          text: `Agent status: ${notification.message}`
        }),
        // 후크 시간이 초과할 경우 요청이 취소되도록 signal 전달
        signal
      });
    } catch (error) {
      if (error instanceof Error && error.name === "AbortError") {
        console.log("Notification cancelled");
      } else {
        console.error("Failed to send notification:", error);
      }
    }

    // 빈 객체 반환. 알림 후크는 에이전트 동작을 수정하지 않음
    return {};
  };

  // Notification 이벤트에 후크 등록 (매처 필요 없음)
  for await (const message of query({
    prompt: "Analyze this codebase",
    options: {
      hooks: {
        Notification: [{ hooks: [notificationHandler] }]
      }
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

## 일반적인 문제 해결

### 후크가 실행되지 않음

* 후크 이벤트 이름이 올바르고 대소문자를 구분하는지 확인하세요 (`preToolUse`가 아닌 `PreToolUse`)
* 매처 패턴이 도구 이름과 정확히 일치하는지 확인하세요
* `options.hooks`에서 올바른 이벤트 유형 아래에 후크가 있는지 확인하세요
* `Notification` 및 `SubagentStop` 같이 매처를 지원하는 비도구 후크의 경우 매처는 서로 다른 필드와 일치하며 `Stop`은 매처를 완전히 무시합니다 ([매처 패턴](/docs/en/hooks#matcher-patterns) 참조)
* 에이전트가 [`max_turns`](/docs/en/agent-sdk/python#claudeagentoptions) 제한에 도달하면 후크가 실행되기 전에 세션이 종료되므로 후크가 안 뜰 수 있습니다

### 매처가 예상대로 필터링되지 않음

매처는 도구 이름만 일치시키며 파일 경로나 기타 인자는 일치시키지 않습니다. 파일 경로로 필터링하려면 후크 내부에서 `tool_input.file_path`를 확인하세요:

```typescript theme={null}
const myHook: HookCallback = async (input, toolUseID, { signal }) => {
  const preInput = input as PreToolUseHookInput;
  const toolInput = preInput.tool_input as Record<string, unknown>;
  const filePath = toolInput?.file_path as string;
  if (!filePath?.endsWith(".md")) return {}; // 마크다운 파일이 아니면 건너뜀
  // 마크다운 파일 처리...
  return {};
};
```

### 후크 타임아웃

* `HookMatcher` 구성에서 `timeout` 값을 늘리세요
* TypeScript에서는 취소를 깔끔하게 처리하기 위해 세 번째 콜백 인자의 `AbortSignal`을 사용하세요

{/* min-version: 2.1.208 */}타임아웃을 초과하는 `UserPromptSubmit` 또는 [`UserPromptExpansion`](/docs/en/hooks#userpromptexpansion) 콜백은 타임아웃 메시지와 함께 해당 프롬프트를 차단하고 세션은 계속 진행됩니다. 콜백이 보류 중인 동안 쿼리를 중단하면 보류 중인 도구 호출이 취소됩니다. v2.1.208 이전에는 해당 이벤트의 콜백 타임아웃이 쿼리를 `error_during_execution`으로 종료했으며 보류 중인 `PreToolUse` 콜백 도중 중단이 발생하면 도구 호출이 진행될 수 있었습니다.

{/* min-version: 2.1.210 */}타임아웃을 초과하는 `PreToolUse` 콜백은 도구 호출을 차단하고 Claude는 타임아웃 명칭이 포함된 오류 결과를 받습니다. 다른 `PreToolUse` 후크가 명시적 거부를 반환한 경우 Claude는 대신 그 거부를 받습니다. v2.1.210 이전에는 사용자가 도구 호출을 거부한 것처럼 Claude Code가 타임아웃을 Claude에게 보고하여 무인 세션이 중단되고 입력을 기다렸습니다.

### 도구가 예기치 않게 차단됨

* 모든 `PreToolUse` 후크에서 `permissionDecision: 'deny'` 반환 여부를 확인하세요
* 후크에 로깅을 추가하여 어떤 `permissionDecisionReason`을 반환하는지 확인하세요
* 매처 패턴이 너무 광범위하지 않은지 확인하세요: 빈 매처는 모든 도구에 일치합니다

### 수정된 입력이 적용되지 않음

* `updatedInput`이 최상위가 아니라 `hookSpecificOutput` 내부에 있는지 확인하세요:

  ```typescript theme={null}
  return {
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "allow",
      updatedInput: { command: "new command" }
    }
  };
  ```

* 수정된 입력을 자동 승인하려면 `permissionDecision: 'allow'`, 사용자에게 표시하여 승인을 받으려면 `'ask'`를 반환하세요

* 출력이 어떤 후크 유형을 위한 것인지 식별하도록 `hookSpecificOutput`에 `hookEventName`을 포함하세요

### Python에서 세션 후크를 사용할 수 없음

`SessionStart` 및 `SessionEnd`는 TypeScript에서 SDK 콜백 후크로 등록할 수 있지만, Python SDK에서는 `HookEvent` 타입에서 생략되어 이용할 수 없습니다. Python에서는 `.claude/settings.json`과 같은 설정 파일에 정의된 [쉘 명령 후크](/docs/en/hooks#hook-events)로만 사용할 수 있습니다. SDK 애플리케이션에서 쉘 명령 후크를 로드하려면 [`setting_sources`](/docs/en/agent-sdk/python#settingsource) 또는 [`settingSources`](/docs/en/agent-sdk/typescript#settingsource)를 통해 적절한 설정 소스를 포함하세요:

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      setting_sources=["project"],  # 후크를 포함한 .claude/settings.json 로드
  )
  ```

  ```typescript TypeScript theme={null}
  const options = {
    settingSources: ["project"] // 후크를 포함한 .claude/settings.json 로드
  };
  ```
</CodeGroup>

Python SDK 콜백으로 초기화 로직을 대신 실행하려면 `client.receive_response()`의 첫 번째 메시지를 트리거로 사용하세요.

### 서브에이전트 권한 프롬프트가 계속 증가함

여러 서브에이전트를 생성할 때 각 서브에이전트가 개별적으로 권한을 요청할 수 있습니다. 서브에이전트는 상위 에이전트 권한을 자동으로 상속하지 않습니다. 반복되는 프롬프트를 방지하려면 `PreToolUse` 후크를 사용하여 특정 도구를 자동 승인하거나 서브에이전트 세션에 적용되는 권한 규칙을 구성하세요.

### 서브에이전트와의 재귀적 후크 루프

서브에이전트를 생성하는 `UserPromptSubmit` 후크는 서브에이전트가 동일한 후크를 트리거하는 경우 무한 루프를 생성할 수 있습니다. 이를 방지하려면:

* 생성하기 전에 후크 입력에서 서브에이전트 표시기를 확인하세요
* 공용 변수나 세션 상태를 사용하여 이미 서브에이전트 내부인지 추적하세요
* 최상위 에이전트 세션에 대해서만 실행되도록 후크의 범위를 지정하세요

### systemMessage가 출력에 표시되지 않음

`systemMessage` 필드는 모델이 아닌 사용자에게 메시지를 보여줍니다. 기본적으로 SDK는 `SessionStart` 및 `Setup` 후크에 대해서만 메시지 스트림에 후크 출력을 표출하므로 `includeHookEvents` (Python의 `include_hook_events`)를 설정하지 않는 한 다른 후크 이벤트의 메시지는 표시되지 않습니다. 모델에 컨텍스트를 대신 전달하려면 [`additionalContext`](/docs/en/hooks#add-context-for-claude)를 반환하세요.

애플리케이션에 후크 결정을 신뢰성 있게 전달해야 하는 경우 별도로 로깅하거나 전용 출력 채널을 사용하세요.

## 관련 리소스

* [Claude Code 후크 참조](/docs/en/hooks): 전체 JSON 입력/출력 스키마, 이벤트 문서 및 매처 패턴
* [Claude Code 후크 가이드](/docs/en/hooks-guide): 쉘 명령 후크 예제 및 설명
* [TypeScript SDK 참조](/docs/en/agent-sdk/typescript): 후크 타입, 입력/출력 정의 및 구성 옵션
* [Python SDK 참조](/docs/en/agent-sdk/python): 후크 타입, 입력/출력 정의 및 구성 옵션
* [권한](/docs/en/agent-sdk/permissions): 에이전트가 할 수 있는 일 제어
* [커스텀 도구](/docs/en/agent-sdk/custom-tools): 에이전트 기능을 확장하는 도구 구축
