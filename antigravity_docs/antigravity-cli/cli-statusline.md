---
slug: cli-statusline
section: Antigravity CLI
title: 상태 표시줄 커스텀 설정
path:
  - Antigravity CLI
  - 커스텀 설정
  - 상태 표시줄
---




# 상태 표시줄 커스텀 설정

기본 상태 표시줄 구성 요소를 토글하고, 커스텀 스크립트 설정을 정의하며, 동적 JSON 상태 페이로드를 조작(format)합니다.

## 개요

상태 표시줄은 TUI 프롬프트 패널의 하단에 위치합니다. 활성 에이전트 사이클, 작업 공간 환경, 컨텍스트 토큰 창 사용량, 백그라운드 실행 태스크에 대한 컨텍스트를 한눈에 보여줍니다.

## 대화형 토글

1. 프롬프트 창에 `/statusline`을 입력하고 `Enter`를 누릅니다.
2. 대화형 **상태 선택기 패널(Status Picker Panel)** 오버레이가 열립니다.
3. `↑`/`↓`를 사용하여 특정 메트릭 요소(예: 활성 모델, 태스크 카운터, 컨텍스트 백분율)를 토글하여 켜고 끕니다.
4. `Enter`를 누르면 선택 사항이 적용되고 종료됩니다. 취소하려면 `Esc`를 누릅니다.

## 커스텀 상태 표시줄 스크립트 작성

고급 터미널 레이아웃이나 커스텀 상태 표시줄 디스플레이의 경우, 활성 에이전트 메타데이터를 커스텀 스크립트로 전달할 수 있습니다.

### 구성

`~/.gemini/antigravity-cli/settings.json` 파일에 `statusLine` 구성 블록을 추가합니다.

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.gemini/antigravity-cli/statusline.sh"
  }
}
```

에이전트 상태가 변경될 때마다 TUI는 사용자 정의 명령 스크립트를 실행하고 상세 상태 JSON 페이로드를 스크립트의 `stdin`에 직접 전달한 후, `stdout`에서 포맷된 문자열을 읽어 프롬프트의 상태 표시줄에 렌더링합니다. 전체 ANSI 색상 코드가 지원됩니다.

### 사용 가능한 JSON 필드

스크립트에 전달되는 JSON 페이로드에는 다음과 같은 최상위 필드가 포함됩니다.

| 필드 | 유형 | 설명 |
| :-------------------------- | :----- | :----------------------------------------------------------------------------------------------------------------------------- |
| `cwd`                       | string | 현재 작업 디렉터리 |
| `conversation_id`           | string | 현재 대화 ID |
| `model`                     | object | 활성 모델의 `id` 및 `display_name` |
| `product`                   | string | 애플리케이션 이름 (예: `antigravity-cli`) |
| `workspace`                 | object | `current_dir` 및 `project_dir` 경로 |
| `version`                   | string | CLI 버전 문자열 |
| `plan_tier`                 | string | 인증된 사용자의 구독 등급 |
| `email`                     | string | 인증된 사용자의 LDAP/이메일 |
| `agent`                     | object | 활성 에이전트 프로필 이름 |
| `context_window`            | object | `total_input_tokens`, `total_output_tokens`, `context_window_size`, `used_percentage`, `remaining_percentage`, `current_usage` |
| `agent_state`               | string | 현재 상태: `idle`, `thinking`, `working`, `tool_use`, `initializing` |
| `vcs`                       | object | 버전 관리 정보: `type` (git/jj/fig), `branch`, `client`, `dirty` |
| `sandbox`                   | object | 샌드박스 구성: `enabled`, `allow_network` |
| `subagents`                 | array  | `name`, `role`, `status`가 포함된 활성 서브에이전트 세션 |
| `artifacts`                 | array  | `uri`, `status`, `type`이 포함된 생성된 아티팩트 |
| `pending_input_count`       | int    | 큐에 대기 중인 사용자 메시지 수 |
| `background_tasks`          | array  | `name`, `status`, `index`가 포함된 실행 중인 태스크 |
| `tool_confirmation_pending` | bool   | 도구 확인 대화 상자 표시 여부 |
| `terminal_width`            | int    | 대화형 터미널의 실시간 너비 |

### JSON 페이로드 예시

상태 표시줄 스크립트로 전달되는 일반적인 실제 JSON 페이로드의 예시입니다.

```json
{
  "cwd": "/home/user/my-project",
  "conversation_id": "12345678-abcd-ef01-2345-6789abcdef01",
  "model": {
    "id": "Gemini",
    "display_name": "Gemini"
  },
  "workspace": {
    "current_dir": "/home/user/my-project",
    "project_dir": "file:///home/user/my-project"
  },
  "version": "2026.04.15",
  "context_window": {
    "total_input_tokens": 88244,
    "total_output_tokens": 61074,
    "context_window_size": 1048576,
    "used_percentage": 8.415603637695312,
    "remaining_percentage": 91.58439636230469,
    "current_usage": {
      "input_tokens": 63382,
      "output_tokens": 346,
      "cache_creation_input_tokens": 0,
      "cache_read_input_tokens": 20857
    }
  },
  "product": "antigravity-cli",
  "agent_state": "idle",
  "vcs": {
    "type": "git",
    "client": "my-project",
    "branch": "dev",
    "dirty": false
  },
  "sandbox": {
    "enabled": false
  },
  "plan_tier": "Pro",
  "email": "developer@email.com",
  "terminal_width": 111
}
```

### 스크립트 예시

공식 [GitHub의 statusline.sh 예시](https://github.com/google-antigravity/antigravity-cli/blob/main/examples/statusline/statusline.sh)에서 레이아웃에 적응하는 전체 스크립트를 다운로드할 수 있습니다. 이 스크립트는 상태 배지를 렌더링하고, 활성 브랜치를 처리하며, 컨텍스트 창 진행률 표시줄을 동적으로 표시합니다.

스크립트를 `~/.gemini/antigravity-cli/statusline.sh`에 저장하고 실행 파일로 만듭니다.

```bash
chmod +x ~/.gemini/antigravity-cli/statusline.sh
```

## 참고 항목

- **[터미널 타이틀 커스텀 설정](/docs/cli-title)**: 동적 창 타이틀을 구성합니다.
- **[설정, 렌더링 및 단축키](/docs/cli-settings)**: 키보드 단축키 및 버퍼를 구성합니다.
- **[권한 및 샌드박스](/docs/cli-sandbox)**: 보안 디렉터리 권한을 관리합니다.
