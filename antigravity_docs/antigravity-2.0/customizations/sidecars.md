---
slug: sidecars
section: Antigravity 2.0
title: 사이드카
path:
  - Antigravity 2.0
  - 커스텀 설정
  - 사이드카
---

# 사이드카 (Sidecars)

사이드카는 Antigravity와 함께 실행되는 백그라운드 프로세스입니다. Antigravity는 사이드카의 라이프사이클을 관리하며, 이를 자동으로 시작하고 충돌이나 오류가 발생할 경우 재시작합니다.
이들은 지속적인 백그라운드 스크립트 실행, 예약된 반복 작업 처리, 이벤트 반응 등에 유용합니다.

## 설정

사이드카는 `sidecar.json` 설정 파일을 검색하여 감지합니다. 사이드카는 다음 두 위치에서 정의할 수 있습니다.

- 전역 사이드카: `~/.gemini/config/sidecars/` 아래
- 플러그인 사이드카: `~/.gemini/config/plugins/<pluginName>/sidecars/` 아래

각 사이드카는 고유의 디렉토리를 가지며, 디렉토리 이름이 사이드카의 ID로 사용됩니다. 플러그인에서 로드된 사이드카의 ID는 `<pluginName>/<sidecarName>` 형식을 따릅니다.

사이드카 디렉토리에는 `sidecar.json` 파일이 포함되어 있어야 하며, 실행할 스크립트와 같은 다른 헬퍼 파일들이 포함될 수 있습니다. 또한 사이드카 디렉토리는 사이드카 명령의 현재 작업 디렉토리(CWD)로 작동합니다.

예시 디렉토리 구조:

```
~/.gemini/config/sidecars/
├── sidecar1/
│   ├── sidecar.json
│   └── script.py
└── sidecar2/
    └── sidecar.json

~/.gemini/config/plugins/
└── my-plugin/
      └── sidecars/
            └── plugin-sidecar/
                  └── sidecar.json
```

### 설정 스키마 (sidecar.json)

*   **`command`** (string): 실행할 명령/실행 파일 (예: `python3` 또는 `/bin/bash`). `builtin`과 상호 배타적입니다.
*   **`builtin`** (string): 실행할 내장(builtin) 명령입니다. 현재는 `schedule`을 지원합니다. `command`와 상호 배타적입니다.
*   **`args`** (string[]): 선택 사항. 명령 또는 내장 함수에 전달할 인수 목록입니다.
*   **`restart_policy`** (string): 선택 사항. 재시작 동작 방식입니다. `always`, `on-failure`, 또는 `never` 중 하나를 지정합니다. 기본값은 `always`입니다.
*   **`description`** (string): 선택 사항. 사이드카의 역할에 대한 설명입니다.
*   **`env`** (object): 선택 사항. 사이드카 프로세스에 설정할 환경 변수 맵입니다.
*   **`display_name`** (string): 선택 사항. UI에 표시될 이름입니다.

`command` 또는 `builtin` 중 하나는 반드시 설정해야 합니다.

예시:

```json
{
  "description": "Background worker",
  "command": "python3",
  "args": ["worker.py"],
  "restart_policy": "on-failure"
}
```

```json
{
  "description": "Hourly agent to triage review requests.",
  "builtin": "schedule",
  "args": [
    "0 * * * *",
    "agentapi",
    "new-conversation",
    "Give me a summary of incoming review requests."
  ]
}
```

### 사용자 설정 (config.json)

사이드카는 사용자가 `~/.gemini/config/config.json`에 위치한 전역 설정 파일에서 명시적으로 활성화하지 않으면 비활성화 상태로 유지됩니다.

*   **`enabled`** (boolean): 사이드카의 활성화 여부입니다.
*   **`projectId`** (string): 선택 사항. `agentapi`가 새 대화를 생성할 프로젝트의 ID입니다.

예시:

```json
{
  "sidecars": {
    "sidecar1": {
      "enabled": true
    },
    "my-plugin/plugin-sidecar": {
      "enabled": true,
      "projectId": "<projectId>"
    }
  }
}
```

### 런타임 데이터

사이드카가 생성하는 런타임 데이터는 `~/.gemini/antigravity/sidecar_data/<sidecarId>/`에 저장됩니다.

여기에는 다음이 포함됩니다.
*   **`data/`**: 영구 데이터를 저장하기 위한 하위 디렉토리입니다. 이 경로는 `ANTIGRAVITY_EXECUTABLE_DATA_DIR` 환경 변수를 통해 사용할 수 있습니다.
*   **`logs/`**: stdout 및 stderr로부터 자동 생성되는 타임스탬프가 기록된 로그입니다.
*   **`events/`**: `agentapi` 호출을 위해 기록된 JSON 파일들입니다.

### `schedule` 내장 명령 (Builtin)

`schedule`은 반복적으로 명령을 실행하기 위한 단순한 내장 스케줄러입니다.

```json
{
  "builtin": "schedule",
  "args": [
    "* * * * *",
    "<command>",
    "<arg1>",
    "<arg2>"
  ]
}
```

첫 번째 인수는 표준 5개 필드로 구성된 크론(cron) 표현식입니다. 나머지 인수들은 지정된 일정에 따라 실행할 명령과 해당 인수들입니다.

### `agentapi`

사이드카는 `agentapi` CLI를 사용하여 Antigravity와 프로그래밍 방식으로 상호 작용할 수 있습니다. 실행 파일은 사이드카의 경로(Path)에 자동으로 추가되어 `agentapi`로 직접 사용할 수 있습니다.

*   `agentapi new-conversation <prompt>`
    새 대화를 만드는 사이드카는 반드시 `projectId`가 설정되어 있어야 합니다.
*   `agentapi send-message <conversation_id> <prompt>`
