---
slug: gcli-migration
section: Antigravity CLI
title: 마이그레이션
path:
- Antigravity CLI
- Gemini 마이그레이션
---




# Gemini CLI에서 마이그레이션하기

기존 구성을 변환하고, Gemini CLI 확장을 네이티브 플러그인으로 가져오고, 사용자 정의 스킬 경로를 조정하고, Model Context Protocol 구성을 재포맷합니다.

## 개요

Antigravity CLI는 Gemini CLI로 널리 알려진 핵심 개발자 경험 구조와의 하위 호환성을 유지합니다. 원활한 업그레이드를 보장하기 위해 CLI는 명시적인 CLI 마이그레이션 명령어 시퀀스와 함께 자동 온보딩 변환을 제공합니다.

## 첫 실행 온보딩

기존 구성이 포함된 환경에서 `agy`를 처음 실행하면 CLI가 기존 프로필을 자동으로 감지합니다. 대화형 체크리스트가 표시되어 마이그레이션할 자산을 선택하라는 메시지가 나타납니다.

1. **자동 변환**: 변환할 확장 프로그램 및 글로벌 구성을 선택합니다.
2. **키링 저장소**: CLI가 활성 세션 토큰을 운영 체제의 네이티브 키링 저장소로 안전하게 마이그레이션합니다.
3. **설정 정렬**: 기본 시각적 매개변수와 렌더링 버퍼가 새 설정 프로필에 자동으로 매핑됩니다.

<Announcement>
icon: info
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **부분적인 호환성**: 작업 공간 스킬, 규칙 및 MCP 서버에 대한 지원은 유지되지만, Gemini CLI의 특정 사용자 정의 터미널 테마 또는 실험적인 시각적 오버레이는 지원되지 않을 수 있습니다.
</Announcement>

## 확장을 플러그인으로 변환하기

Gemini CLI가 출시된 이후, 업계에서는 **플러그인(plugins)**이라는 용어로 표준화되었습니다. 다음 명령을 실행하여 레거시 Gemini 확장을 네이티브 Antigravity 플러그인으로 수동 변환할 수 있습니다.

```bash
agy plugin import gemini
```

이 유틸리티는 기존 로컬 디렉터리를 검색하고, 확장 매니페스트를 파싱하고, 파일을 네이티브 레이아웃 블록으로 변환합니다.

### 예상 가져오기 출력

```text
[ok]   conductor-tools
       - skills     : skipped (none detected)
       - agents     : skipped (none detected)
       ✔ commands   : 4 legacy commands converted to skills
       - mcpServers : skipped (none detected)
[ok]   google-workspace
       ✔ skills     : 5 skills processed
       - agents     : skipped (none detected)
       ✔ commands   : 2 legacy commands converted to skills
       ✔ mcpServers : 1 server definition migrated to mcp_config.json
```

## 컨텍스트 파일 및 작업 공간 규칙

두 CLI 플랫폼 모두 동일한 작업 공간 컨텍스트 규칙을 사용합니다. 기존 규칙 문서에 대한 수정은 필요하지 않습니다.

- **작업 공간 로컬 컨텍스트**: 에이전트는 활성 디렉터리의 `GEMINI.md` 및 `AGENTS.md` 파일 내에 정의된 규칙 제약 조건을 계속 파싱하고 적용합니다.
- **글로벌 개발자 컨텍스트**: 에이전트는 `~/.gemini/GEMINI.md`에 위치한 글로벌 제약 조건을 자동으로 참고하고 적용합니다.

## 업데이트된 스킬 경로

글로벌 공유 스킬은 사용자 홈 디렉터리에 그대로 유지되지만, 로컬 작업 공간 전용 스킬의 대상 폴더 경로가 업데이트되었습니다.

| 구성 | Gemini CLI | Antigravity CLI |
| :------------------------- | :------------------ | :---------------------------------- |
| **글로벌 공유 경로** | `~/.gemini/skills/` | `~/.gemini/antigravity-cli/skills/` |
| **작업 공간 프로젝트 경로** | `.gemini/skills/` | `.agents/skills/` |

<Announcement>
icon: warning
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **필수 조치**: 프로젝트에 `.gemini/skills/`에 정의된 사용자 정의 작업 공간 스킬이 포함되어 있는 경우, Antigravity 에이전트가 이를 활성 슬래시 명령어로 인식하도록 폴더 이름을 `.agents/skills/`로 수동으로 변경하거나 위치를 이동해야 합니다.
</Announcement>

## MCP 설정 형식 변경 사항

Antigravity CLI는 Model Context Protocol 서버를 기본 설정 구성 내에 중첩하는 대신 독립적인 경량 JSON 프로필로 분리합니다.

### 디렉터리 매핑

- **레거시 Gemini 구성**: 서버가 `~/.gemini/settings.json` 내에 인라인으로 선언되었습니다.
- **Antigravity CLI 구성**: 서버가 독립 실행형 `mcp_config.json` 프로필 내에 정의됩니다.
  - 글로벌 서버: `~/.gemini/config/mcp_config.json`
  - 작업 공간 서버: `.agents/mcp_config.json`

### 필수 스키마 업데이트

원격 웹소켓 또는 SSE 서버 정의를 수동으로 마이그레이션할 때, 현재 표준에 맞게 URI 키 매개변수를 업데이트하세요.

- **레거시 스키마 키**: `url` 또는 `httpUrl`
- **최신 스키마 키**: `serverUrl`

```json
{
  "mcpServers": {
    "remote-indexer": {
      "serverUrl": "https://mcp.internal.enterprise.com/sse",
      "env": {
        "AUTH_TOKEN": "secure_alpha_token"
      }
    }
  }
}
```

## 다음 단계

새로운 시각적 매개변수를 구성하고 설정 시의 비정상적인 문제를 해결해 보세요.

- **[설정, 렌더링 및 단축키](/docs/cli-settings)**: 키보드 단축키, 테마 및 화면 버퍼를 커스텀 설정합니다.
- **[문제 해결](/docs/cli-troubleshooting)**: 인증 잠금 또는 경로 문제를 해결하는 방법을 알아봅니다.
- **[CLI 참조](/docs/cli-reference)**: 표준 매개변수 목록 및 슬래시 명령어 매핑에 액세스합니다.
