---
slug: ide-hooks
section: Antigravity IDE
title: 훅
path:
  - Antigravity IDE
  - 사용자 정의
  - 훅
---

# 훅 (Hooks)

훅(Hooks)을 사용하면 Antigravity의 실행 루프 중 특정 시점에 사용자 정의 스크립트나 셸 명령을 실행할 수 있습니다. 이는 사용자 정의 규칙을 적용하거나, 린터를 실행하거나, 진단 정보를 자동으로 캡처하는 데 강력한 유용성을 제공합니다.

## 구성

훅은 사용자 정의 디렉터리(예: 워크스페이스의 `.agents/` 또는 `~/.gemini/config/`)에 위치한 `hooks.json` 파일에서 구성됩니다.

## 스키마 및 파일 형식

`hooks.json` 파일은 훅 이름을 해당 이벤트 구성에 매핑합니다.

```json
{
  "my-linter-hook": {
    "PostToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/lint.sh",
            "timeout": 10
          }
        ]
      }
    ]
  },
  "safety-gate": {
    "enabled": false,
    "PreToolUse": [
      {
        "matcher": "run_command",
        "hooks": [
          {
            "command": "./scripts/safety-check.sh"
          }
        ]
      }
    ]
  },
  "reminder": {
    "PreInvocation": [
      {
        "type": "command",
        "command": "./scripts/reminder.sh"
      }
    ]
  }
}
```

### 훅 정의 필드

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `enabled` | boolean | 선택 사항. 제거하지 않고 훅을 비활성화하려면 `false`로 설정합니다. 기본값은 `true`입니다. |
| `PreToolUse` | array | 도구가 실행되기 전에 실행되는 핸들러입니다. |
| `PostToolUse` | array | 도구가 완료된 후에 실행되는 핸들러입니다. |
| `PreInvocation` | array | Antigravity가 모델을 호출하기 전에 실행되는 핸들러입니다. |
| `PostInvocation` | array | 도구 호출이 종료된 후에 실행되는 핸들러입니다. |
| `Stop` | array | 실행 루프가 종료될 때 실행되는 핸들러입니다. |

## 지원되는 이벤트

| 이벤트 | 설명 | 매처 대상 |
| :--- | :--- | :--- |
| `PreToolUse` | 도구가 실행되기 전에 발생합니다. | 도구 이름 (예: `run_command`) |
| `PostToolUse` | 도구가 완료된 후에 발생합니다. | 도구 이름 |
| `PreInvocation` | 모델이 호출되기 전에 발생합니다. | 해당 없음 (매처 무시됨) |
| `PostInvocation` | 도구 호출이 종료된 후에 발생합니다. | 해당 없음 (매처 무시됨) |
| `Stop` | 실행이 종료될 때 발생합니다. | 해당 없음 (매처 무시됨) |

### 매처 (Matcher)

`PreToolUse` 및 `PostToolUse`에 대해 `matcher` 필드에 정규 표현식을 사용하여 어떤 도구가 훅을 트리거할지 지정할 수 있습니다:

* `""` 또는 `"*"`: 모든 도구와 일치합니다.  
* `"run_command"`: 정확히 `run_command`와 일치합니다.  
* `"run_command|view_file"`: 두 도구 중 하나와 일치합니다.  
* `"browser_.*"`: `browser_`로 시작하는 모든 도구와 일치합니다.

<Announcement>
icon: info
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **참고**: `PreInvocation`, `PostInvocation` 및 `Stop` 이벤트의 경우 구조가 더 간단하며(이벤트 키 바로 아래에 핸들러 목록이 위치함), 매처(matcher)는 무시됩니다.
</Announcement>

## 지원되는 도구

`PreToolUse` 및 `PostToolUse` 매처의 경우, 카테고리별로 그룹화된 다음 도구 이름과 일치시킬 수 있습니다:

### 파일 및 디렉터리 작업

* **`view_file`**: 파일 내용 보기  
  * 인수: `AbsolutePath`, `StartLine` (선택 사항), `EndLine` (선택 사항), `IsSkillFile` (선택 사항)  
* **`write_to_file`**: 새 파일 생성  
  * 인수: `TargetFile`, `Overwrite`, `CodeContent`, `Description`, `IsArtifact` (선택 사항), `ArtifactMetadata` (선택 사항)  
* **`replace_file_content`**: 파일의 단일 연속 텍스트 블록 편집  
  * 인수: `TargetFile`, `Instruction`, `Description`, `AllowMultiple`, `TargetContent`, `ReplacementContent`, `StartLine`, `EndLine`, `TargetLintErrorIds` (선택 사항)  
* **`multi_replace_file_content`**: 동일한 파일에 대한 연속되지 않은 여러 편집 수행  
  * 인수: `TargetFile`, `Instruction`, `Description`, `ReplacementChunks` (청크 배열), `TargetLintErrorIds` (선택 사항), `ArtifactMetadata` (선택 사항)  
* **`list_dir`**: 디렉터리 내용 나열  
  * 인수: `DirectoryPath`  
* **`find_by_name`**: glob 패턴을 사용하여 파일 및 디렉터리 검색  
  * 인수: `SearchDirectory`, `Pattern`, `Type` (선택 사항), `Excludes` (선택 사항), `Extensions` (선택 사항), `FullPath` (선택 사항), `MaxDepth` (선택 사항)

### 검색 및 리서치

* **`grep_search`**: 특정 경로 내에서 빠른 텍스트 검색  
  * 인수: `SearchPath`, `Query`, `IsRegex` (선택 사항), `CaseInsensitive` (선택 사항), `Includes` (선택 사항), `MatchPerLine` (선택 사항)  
* **`search_web`**: 일반 웹 검색 수행  
  * 인수: `query`, `domain` (선택 사항)  
* **`read_url_content`**: 공개 URL의 텍스트 콘텐츠 가져오기  
  * 인수: `Url`

### 시스템 및 실행

* **`run_command`**: 실행할 bash 명령 제안  
  * 인수: `CommandLine`, `Cwd`, `WaitMsBeforeAsync`, `RunPersistent` (선택 사항), `RequestedTerminalID` (선택 사항)  
* **`manage_task`**: 백그라운드 작업과 상호 작용  
  * 인수: `Action` (`'list'`, `'kill'`, `'status'`, `'send_input'`), `TaskId` (선택 사항), `Input` (선택 사항)  
* **`schedule`**: 타이머 또는 반복적인 크론(cron) 작업 설정  
  * 인수: `DurationSeconds` (선택 사항), `CronExpression` (선택 사항), `MaxIterations` (선택 사항), `Prompt`  
* **`list_permissions`**: 현재 리소스 액세스 권한 보기  
  * 인수: 없음  
* **`ask_permission`**: 추가적인 범위의 권한 요청  
  * 인수: `Action`, `Target`, `Reason`

### 에이전트 협업

* **`invoke_subagent`**: 특화된 하위 에이전트 생성  
  * 인수: `Subagents` (`Prompt`, `Role`, `TypeName`, `Workspace` (선택 사항)가 포함된 스펙 배열)  
* **`define_subagent`**: 사용자 정의 하위 에이전트 생성  
  * 인수: `name`, `description`, `system_prompt`, `enable_mcp_tools` (선택 사항), `enable_write_tools` (선택 사항), `enable_subagent_tools` (선택 사항)  
* **`send_message`**: 다른 에이전트와 통신  
  * 인수: `Recipient`, `Message`  
* **`manage_subagents`**: 활성 하위 에이전트 나열 또는 종료  
  * 인수: `Action` (`'list'`, `'kill'`, `'kill_all'`), `ConversationIds` (선택 사항)

### 상호 작용 및 미디어

* **`ask_question`**: 객관식 질문 제공  
  * 인수: `questions` (`question`, `options`, `is_multi_select`가 포함된 질문 배열)  
* **`generate_image`**: 이미지 생성 또는 편집  
  * 인수: `Prompt`, `ImageName`, `ImagePaths` (선택 사항)

## 훅 핸들러 구성

`hooks` 배열의 각 항목은 다음을 지원합니다:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `type` | string | 선택 사항. 현재 `"command"`만 지원됩니다. 기본값은 `"command"`입니다. |
| `command` | string | 필수. 실행할 셸 명령입니다. |
| `timeout` | integer | 선택 사항. 초 단위의 타임아웃 시간입니다. 기본값은 `30`입니다. |

## 입력/출력 규약 (Input/Output Contract)

훅은 **stdin**을 통해 JSON 형식으로 입력을 받고, **stdout**을 통해 JSON 형식으로 출력을 반환해야 합니다. 필드 이름은 camelCase를 사용합니다.

### 공통 입력 필드

모든 훅은 `stdin` 입력 페이로드에서 다음과 같은 시스템 메타데이터 필드를 받습니다:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `conversationId` | string | 활성 에이전트 대화의 고유 UUID입니다. |
| `workspacePaths` | array of strings | 사용자의 마운트된 워크스페이스를 나타내는 절대 디렉터리 경로 목록입니다. |
| `transcriptPath` | string | 지속적인 `transcript.jsonl` 대화 로그의 절대 경로입니다. |
| `artifactDirectoryPath` | string | 모든 대화 아티팩트 및 스크린샷이 포함된 디렉터리의 절대 경로입니다. |

---

### PreToolUse

도구가 실행되기 전에 발생합니다.

#### 스키마

**입력 필드 (stdin)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `toolCall` | object | 제안된 도구 호출의 세부 정보입니다. |
| `toolCall.name` | string | 실행 중인 도구의 이름입니다 (예: `run_command`). |
| `toolCall.args` | object | 도구에 전달된 인수입니다. |
| `stepIdx` | integer | 궤적(trajectory) 상에서의 현재 단계 인덱스(0부터 시작)입니다. |
| *(공통 필드)* | | `conversationId`, `workspacePaths`, `transcriptPath`, `artifactDirectoryPath`를 포함합니다. |

**출력 필드 (stdout)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `decision` | string | **필수.** 도구 호출이 제어되는 방식을 제어합니다:<br>- `"allow"`: 도구 실행을 자동으로 허용합니다.<br>- `"deny"`: 즉시 실행을 강력히 차단합니다.<br>- `"ask"`: 사용자에게 묻되, "항상 허용" 설정을 존중합니다.<br>- `"force_ask"`: 캐시된 권한을 무시하고 항상 사용자에게 확인을 요청합니다. |
| `reason` | string | **선택 사항.** 에이전트 또는 사용자에게 결정 이유를 설명하는 텍스트입니다. |
| `permissionOverrides` | array of strings | **선택 사항.** 기본 도구 권한을 재정의하기 위한 리소스 문자열 목록(예: `["read_file(/path)", "command(args)"]`)입니다. |

#### 예시

* **입력 (stdin)**:

```json
{
  "toolCall": {
    "name": "run_command",
    "args": {
      "CommandLine": "npm test",
      "Cwd": "/workspace/project",
      "WaitMsBeforeAsync": 5000
    }
  },
  "stepIdx": 19,
  "conversationId": "ec33ebf9-0cba-4100-8142-c61503f6c587",
  "workspacePaths": ["/workspace/project"],
  "transcriptPath": "/workspace/project/.gemini/antigravity/transcript.jsonl",
  "artifactDirectoryPath": "/workspace/project/.gemini/antigravity/artifacts"
}
```

* **출력 (stdout)**:

```json
{
  "decision": "ask",
  "reason": "Requires confirmation for test execution.",
  "permissionOverrides": ["command(npm test)"]
}
```

---

### PostToolUse

도구가 완료된 후에 발생합니다.

#### 스키마

**입력 필드 (stdin)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `stepIdx` | integer | 완료된 단계의 인덱스(0부터 시작)입니다. |
| `error` | string | 선택 사항. 도구 호출이 실패한 경우의 상세 런타임 에러 메시지입니다. 성공 시 비어 있습니다. |
| *(공통 필드)* | | `conversationId`, `workspacePaths`, `transcriptPath`, `artifactDirectoryPath`를 포함합니다. |

**출력 필드 (stdout)**: 빈 JSON 객체 `{}`를 반환합니다.

#### 예시

* **입력 (stdin)**:

```json
{
  "stepIdx": 5,
  "error": "exit status 1",
  "conversationId": "ec33ebf9-0cba-4100-8142-c61503f6c587",
  "workspacePaths": ["/workspace/project"],
  "transcriptPath": "/workspace/project/.gemini/antigravity/transcript.jsonl",
  "artifactDirectoryPath": "/workspace/project/.gemini/antigravity/artifacts"
}
```

* **출력 (stdout)**: `{}`

---

### PreInvocation

모델이 호출되기 전에 발생합니다.

#### 스키마

**입력 필드 (stdin)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `invocationNum` | integer | 현재 모델 호출의 시퀀스 번호입니다. |
| `initialNumSteps` | integer | 현재 궤적(trajectory)에 있는 단계의 수입니다. |
| *(공통 필드)* | | `conversationId`, `workspacePaths`, `transcriptPath`, `artifactDirectoryPath`를 포함합니다. |

**출력 필드 (stdout)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `injectSteps` | array of objects | **선택 사항.** 모델이 호출되기 전에 대화 궤적에 주입할 단계 목록입니다. |

*주입된 단계 스키마(Injected Step Schema)*: `injectSteps` 배열의 각 객체는 다음 필드 중 하나를 가질 수 있습니다:

* `toolCall` (object): 실행할 도구 호출입니다.  
* `userMessage` (string): 사용자가 보낸 메시지입니다.  
* `ephemeralMessage` (string): 일시적인 시스템 메시지입니다.

#### 예시

* **입력 (stdin)**:

```json
{
  "invocationNum": 3,
  "initialNumSteps": 10,
  "conversationId": "ec33ebf9-0cba-4100-8142-c61503f6c587",
  "workspacePaths": ["/workspace/project"],
  "transcriptPath": "/workspace/project/.gemini/antigravity/transcript.jsonl",
  "artifactDirectoryPath": "/workspace/project/.gemini/antigravity/artifacts"
}
```

* **출력 (stdout)**:

```json
{
  "injectSteps": [{"ephemeralMessage": "Remember to lint"}]
}
```

---

### PostInvocation

도구 호출이 종료된 후에 발생합니다.

#### 스키마

**입력 필드 (stdin)**: `PreInvocation` 입력 필드와 동일합니다.

**출력 필드 (stdout)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `injectSteps` | array of objects | **선택 사항.** 호출이 완료된 후 주입할 단계 목록입니다(`PreInvocation` 주입 단계와 동일한 스키마 사용). |
| `terminationBehavior` | string | **선택 사항.** 주입 후의 실행 흐름을 제어합니다:<br>- `"force_continue"`: 루프를 강제로 계속 진행합니다.<br>- `"terminate"`: 루프를 강제로 종료합니다.<br>- `""` (또는 생략됨): 기본 동작으로 작동합니다. |

#### 예시

* **입력 (stdin)**: `PreInvocation`과 동일  
* **출력 (stdout)**:

```json
{
  "injectSteps": [],
  "terminationBehavior": ""
}
```

---

### Stop

실행 루프가 종료될 때 발생합니다.

#### 스키마

**입력 필드 (stdin)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `executionNum` | integer | 실행 시도의 시퀀스 번호입니다. |
| `terminationReason` | string | 실행이 정지하는 이유입니다 (예: `"model_stop"`, `"max_steps_exceeded"`, `"error"`). |
| `error` | string | 선택 사항. 시스템 에러로 인해 종료된 경우의 에러 메시지입니다. |
| `fullyIdle` | boolean | **필수.** 에이전트가 완전히 완료되었고 모든 백그라운드 명령 또는 비동기 작업이 완료된 경우 `true`입니다. 활성 백그라운드 작업이 계속 실행 중인 경우 `false`입니다. |
| *(공통 필드)* | | `conversationId`, `workspacePaths`, `transcriptPath`, `artifactDirectoryPath`를 포함합니다. |

**출력 필드 (stdout)**:

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `decision` | string | **필수.** 에이전트가 중지되는 것을 방지하고 실행 루프에 다시 진입하도록 하려면 `"continue"`로 설정합니다. 다른 모든 값은 중지를 허용합니다. |
| `reason` | string | **선택 사항.** `decision`이 `"continue"`인 경우, 이 메시지가 시스템 메시지로서 대화에 주입됩니다. |

#### 예시

* **입력 (stdin)**:

```json
{
  "executionNum": 1,
  "terminationReason": "model_stop",
  "error": "",
  "fullyIdle": true,
  "conversationId": "ec33ebf9-0cba-4100-8142-c61503f6c587",
  "workspacePaths": ["/workspace/project"],
  "transcriptPath": "/workspace/project/.gemini/antigravity/transcript.jsonl",
  "artifactDirectoryPath": "/workspace/project/.gemini/antigravity/artifacts"
}
```

* **출력 (stdout)**:

```json
{
  "decision": "continue",
  "reason": "Not done yet"
}
```
