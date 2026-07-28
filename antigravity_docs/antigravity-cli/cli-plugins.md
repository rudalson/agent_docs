---
slug: cli-plugins
section: Antigravity CLI
title: 플러그인 및 스킬
path:
- Antigravity CLI
- 커스텀 설정
- 플러그인 및 스킬
---




# 플러그인 및 스킬

에이전트 기능을 확장하고, 서드파티 확장 프로그램을 설치하고, 커스텀 워크플로 스킬을 작성하고, Model Context Protocol(MCP) 서버와 연동하세요.

## 확장성 모델 

Antigravity CLI는 한계 없는 맞춤 설정을 지원하도록 설계되었습니다. **플러그인(Plugins)**이라 불리는 구조화된 패키지 모듈을 설치하거나, **스킬(Skills)**이라 불리는 국소적인 마크다운 청사진을 생성하여 공유 에이전트의 기능을 보강할 수 있습니다.

이러한 맞춤 설정을 통해 에이전트는 특수한 사유 명령어를 실행하고, 특정 영역에 특화된 서브에이전트를 호출하며, 맞춤형 스타일 규칙을 참고할 수 있습니다.

## Antigravity 플러그인

플러그인은 커스텀 스킬, 백그라운드 서브에이전트, 린팅(linting) 규칙, Model Context Protocol 정의 및 이벤트 훅을 단일 배포 가능 자산으로 결합한 네임스페이스 번들입니다.

### 플러그인 파일 시스템 구조

플러그인을 설치하거나 가져올 때 CLI는 글로벌 구성 경로 내에 번들 파일을 준비(stage)합니다.

```text
~/.gemini/antigravity-cli/plugins/<plugin_name>/
```

호환되는 플러그인은 다음과 같은 레이아웃을 포함합니다.

```text
~/.gemini/antigravity-cli/plugins/<plugin_name>/
├── plugin.json                 # 필수 패키지 마커 파일
├── mcp_config.json             # 선택적 Model Context Protocol 서버 정의
├── hooks.json                  # 선택적 도구 실행 전/후 이벤트 훅
├── skills/                     # 선택적 특화 스킬 디렉터리
├── agents/                     # 선택적 서브에이전트 정의 템플릿
└── rules/                      # 선택적 커스텀 코드베이스 규칙 파일
```

### CLI 하위 명령어를 통한 플러그인 관리

CLI는 확장 프로그램을 관리하기 위해 `plugin`(또는 복수형 `plugins`) 하위 명령어 파이프라인을 제공합니다.

- **설치된 플러그인 목록 확인**: 활성화된 패키지와 로드된 구성 요소를 보여줍니다.
  ```bash
  agy plugin list
  ```
- **로컬 또는 원격 플러그인 설치**: 패키지 디렉터리를 로컬 프로필로 가져옵니다.
  ```bash
  agy plugin install /path/to/local/plugin
  ```
- **플러그인 활성화/비활성화**: 플러그인의 자산을 삭제하지 않고 해당 도구의 실행만 일시 중단합니다.
  ```bash
  agy plugin disable <plugin_name>
  agy plugin enable <plugin_name>
  ```
- **플러그인 삭제**: 패키지 디렉터리를 삭제하고 레지스트리를 정리합니다.
  ```bash
  agy plugin uninstall <plugin_name>
  ```

## 에이전트 스킬 

스킬은 특수한 엔지니어링 작업을 수행하기 위한 명시적인 지침 프로토콜, 스크립트 및 대상 자산을 기술한 사람이 읽을 수 있는 선언적 마크다운 파일입니다.

등록된 **스킬은 TUI 내에서 자동으로 슬래시 명령어 버전으로 변환**되므로 사용자가 직접 호출할 수 있습니다 (예: `/refactor-ui` 입력).

### 로컬 작업 공간 스킬 생성

git 저장소와 동기화되는 작업 공간 전용 스킬을 빌드하려면 다음과 같이 하십시오.

1. 프로젝트 루트에 `.agents/skills/`라는 디렉터리를 생성합니다.
2. 디렉터리 내에 `.md` 확장자를 가진 마크다운 파일(예: `format-tests.md`)을 작성합니다.
3. 스킬의 Frontmatter 메타데이터를 정의합니다.
   ```yaml
   ---
   name: format-tests
   description: Python unittest 단언문 표준화 및 재포맷
   ---
   ```
4. 메타데이터 아래에 에이전트를 위한 명시적인 지침을 작성합니다. 해당 디렉터리에서 `agy`를 실행하면 스킬이 컴파일되고 프롬프트 창에서 `/format-tests` 명령어를 사용할 수 있게 됩니다.

### 글로벌 스킬 공유

워크스테이션의 모든 작업 공간에서 스킬을 공유하려면 대상 마크다운 파일을 글로벌 구성 경로 내에 저장합니다.

```text
~/.gemini/antigravity-cli/skills/
```

이 디렉터리에 있는 모든 마크다운 스킬은 임의의 디렉터리에서 `agy`를 실행할 때마다 글로벌 슬래시 명령어로 자동으로 가져와집니다.

## 훅 관리 

훅(Hooks)은 에이전트 작업이 실행되기 직전 또는 직후에 해당 작업을 가로챕니다. 자동 사전 점검을 실행하거나 생성된 파일 포맷을 변경(예: 파일 작성 후 `prettier` 실행)하는 데 유용합니다.

훅은 플러그인의 `hooks.json` 내에 정의되거나 기본 `settings.json` 파일 내에 구성됩니다. TUI 프롬프트 창에 다음을 입력하여 로드되고 활성화된 모든 훅을 확인할 수 있습니다.

```text
/hooks
```

## Model Context Protocol (MCP)

Model Context Protocol은 파운데이션 모델이 로컬 API, 파일 파서 및 커스텀 개발자 도구와 안전하게 연동할 수 있도록 지원하는 개방형 표준입니다.

Antigravity CLI는 로컬 프로세스 및 원격 호스트 MCP 서버 구성을 모두 지원합니다.

### MCP 관리자 액세스

프롬프트 패널에 `/mcp`를 입력하고 `Enter`를 누르면 대화형 **MCP 관리자 오버레이(MCP Manager Overlay)**가 열립니다. 이 패널에서 다음 작업을 수행할 수 있습니다.

- 활성화, 연결 해제 또는 로딩 중인 서버의 실시간 상태 링을 확인합니다.
- 서버 구성을 수동으로 다시 로드하거나 연결 로그를 확인합니다.

### 글로벌 및 작업 공간 서버 설정

기존 설정과 달리, Antigravity는 MCP 정의를 각각 독립된 구성으로 분리합니다.

- **글로벌 서버 설정**: `~/.gemini/antigravity-cli/mcp_config.json`에 구성됩니다.
- **작업 공간 로컬 설정**: 활성 프로젝트의 `.agents/mcp_config.json`에 구성됩니다.

```json
{
  "mcpServers": {
    "sqlite-explorer": {
      "command": "node",
      "args": ["/usr/local/bin/sqlite-mcp-server.js"],
      "env": {
        "SQLITE_DB_PATH": "/var/data/app.db"
      }
    }
  }
}
```

<Announcement>
icon: warning
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **원격 연결 스키마**: 원격 SSE 또는 웹소켓 기반의 MCP 연결을 선언할 때는 반드시 `serverUrl` 필드를 정의해야 합니다. 기존의 `url` 또는 `httpUrl` 필드는 지원되지 않습니다.
</Announcement>

## 다음 단계

Gemini CLI에서 기존 구성을 마이그레이션하고 연결 문제를 해결하는 방법을 알아봅니다.

- **[Gemini CLI에서 마이그레이션하기](/docs/gcli-migration)**: 기존 확장 및 구성을 빠르게 변환합니다.
- **[문제 해결](/docs/cli-troubleshooting)**: 터미널 훅 에러, 잠금 현상 또는 네트워크 연결 오류를 해결합니다.
- **[권한 및 샌드박스](/docs/cli-sandbox)**: 커스텀 플러그인과 MCP 서버 주변에 보안 격리 링을 구성합니다.
