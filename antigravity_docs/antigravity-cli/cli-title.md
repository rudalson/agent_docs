---
slug: cli-title
section: Antigravity CLI
title: 터미널 제목 사용자 정의
path:
  - Antigravity CLI
  - 사용자 정의
  - 창 제목
---

# 터미널 제목 사용자 정의

동적 창 제목을 구성하고, 사용자 정의 스크립팅 구성을 매핑하고, JSON 상태 출력을 형식화하여 터미널 헤더를 사용자 정의합니다.

## 개요

터미널 창 제목 기능은 터미널 에뮬레이터의 제목 표시줄 내에 에이전트 세부 정보, 활성 워크스페이스 기본 이름 및 활성 대화 매개변수를 표시합니다. 이를 통해 터미널 창이 최소화되거나 포커스가 해제되어 있을 때도 에이전트 진행 상황을 모니터링할 수 있습니다.

## 대화형 토글링

- **구문**:
  ```text
  /title [on|off]
  ```
- 프롬프트 상자에 `/title`을 입력하고 `Enter`를 눌러 기능을 켜거나 끕니다.
- 기능을 명시적으로 설정하려면 `/title on` 또는 `/title off`를 입력합니다.

## 사용자 정의 제목 스크립팅

사용자 정의된 창 제목 형식을 지정하기 위해 활성 TUI 상태 세부 정보를 사용자 정의 쉘 스크립트로 라우팅할 수 있습니다.

### 구성

`~/.gemini/antigravity-cli/settings.json` 파일에 `title` 구성 블록을 추가합니다.

```json
{
  "title": {
    "type": "command",
    "command": "~/.gemini/antigravity-cli/title.sh"
  }
}
```

에이전트 상태가 변경될 때마다 TUI는 커맨드 스크립트를 실행하고 세부 상태 JSON 페이로드를 스크립트의 `stdin`으로 직접 파이핑한 다음, `stdout`에서 형식화된 문자열을 읽고 터미널 창 제목을 업데이트합니다. 인쇄할 수 없는 문자와 ANSI 이스케이프 시퀀스는 렌더링 전에 자동으로 제거됩니다.

### JSON 상태 페이로드 스키마

JSON 상태 페이로드는 사용자 정의 상태 표시줄 스크립트로 전송되는 페이로드와 동일합니다. 여기에는 `cwd`, `conversation_id`, `agent_state`, `vcs` 세부 정보 등을 나타내는 세부 속성이 포함됩니다. 전체 속성 목록은 **[상태 표시줄 스키마](/docs/cli-statusline)**를 참조하세요.

### 예제 스크립트

공식 [GitHub의 title.sh 예제](https://github.com/google-antigravity/antigravity-cli/blob/main/examples/title/title.sh)에서 전체 레이아웃 적응형 스크립트를 다운로드할 수 있습니다. 이 스크립트는 활성 워크스페이스 폴더 기본 이름을 추출하고 실시간 에이전트 상태 및 대화 세션 접두사가 포함된 구조화된 터미널 제목을 렌더링합니다.

스크립트를 `~/.gemini/antigravity-cli/title.sh`에 저장하고 실행 가능하게 만듭니다.

```bash
chmod +x ~/.gemini/antigravity-cli/title.sh
```

## 참고 항목

- **[상태 표시줄 사용자 정의](/docs/cli-statusline)**: 동적 TUI 상태 표시줄을 사용자 정의합니다.
- **[설정, 렌더링 및 키 바인딩](/docs/cli-settings)**: 키보드 단축키 및 버퍼를 사용자 정의합니다.
- **[권한 및 샌드박스](/docs/cli-sandbox)**: 보안 디렉터리 권한을 관리합니다.
