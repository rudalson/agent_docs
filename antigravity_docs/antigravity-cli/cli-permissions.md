---
slug: cli-permissions
section: Antigravity CLI
title: 권한
path:
  - Antigravity CLI
  - 에이전트 기능
  - 권한
---




# 권한

로컬 워크스테이션을 안전하게 보호하고, 절대 파일 경로를 제한하며, 사용자 정의 허용/거부/질문 정책을 구성하고, 대화형 승인을 관리하세요.

## 세부 권한 관리 

자율 워크플로를 구현하는 동시에 워크스테이션을 안전하게 보호하기 위해, Antigravity CLI는 강력한 **세부 권한 관리 엔진(Fine-Grained Permissions Engine)**을 내장하고 있습니다. 에이전트가 수행하는 모든 보안 민감 작업은 `action(target)` 형식의 **권한 리소스**로 표현됩니다.

권한은 글로벌 설정 내에 구성된 세 가지 개별 액세스 목록을 바탕으로 평가됩니다.

```text
~/.gemini/antigravity-cli/settings.json
```

- **`deny` (거부)**: 작업이 즉시 차단됩니다.
- **`ask` (질문)**: 에이전트는 작업을 중단하고 진행하기 전에 명시적인 승인을 요청하는 프롬프트를 표시합니다.
- **`allow` (허용)**: 프롬프트 표시 없이 작업이 자동으로 승인됩니다.

<Announcement>
icon: warning
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **우선순위 규칙**: 규칙이 서로 충돌하는 경우, **Deny > Ask > Allow**의 우선순위로 엄격하게 평가됩니다. 예를 들어, `ask` 목록에 `command(*)`를 구성하고 `allow` 목록에 `command(git)`을 구성한 경우, `ask` 규칙이 우선 적용되어 모든 명령어를 실행하기 전에 확인 프롬프트가 표시됩니다.
</Announcement>

## 지원하는 작업 및 매칭 규칙

세부 권한은 표준 스키마 패턴을 따릅니다.

```text
action(target)
```

지원하는 작업, 대상 형식 사양 및 매칭 알고리즘은 다음과 같습니다.

| 작업 | 대상 형식 | 매칭 동작 | 기본 폴백 |
| :---------------- | :------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------- |
| **`read_file`** | `read_file(/path)`, `read_file(dir)`, 또는 `read_file(*)` | 절대 경로 또는 작업 공간 루트 기준의 상대 경로와 일치합니다. 포함된 모든 파일/폴더에 대한 재귀적 읽기 권한을 부여합니다. `read_file(*)`은 시스템의 모든 파일과 일치합니다. | **질문(Ask)** (작업 공간 내에서는 자동 허용) |
| **`write_file`** | `write_file(/path)` 또는 `write_file(*)` | `read_file`과 동일합니다. 동일한 대상 경로에 대해 암묵적으로 `read_file` 권한을 부여합니다. | **질문(Ask)** (작업 공간 내에서는 자동 허용) |
| **`read_url`** | `read_url(domain)` 또는 `read_url(*)` | 호스트 이름 및 하위 도메인과 일치합니다(예: `google.com`은 `mail.google.com`을 포함). URL 경로 세그먼트는 무시합니다. `read_url(*)`은 모든 도메인과 일치합니다. | **질문(Ask)** |
| **`execute_url`** | `execute_url(domain)` 또는 `execute_url(*)` | 도메인에서 웹 요소 조작(클릭, 입력) 또는 대화형 브라우저 워크플로 실행을 일치시킵니다. | **질문(Ask)** |
| **`command`** | `command(prefix)`, `command(regex)`, 또는 `command(*)` | 정확한 단어/토큰 접두사로 명령어를 매핑합니다. 공백으로 구분된 각 토큰은 앵커 정규식(`^(?:pattern)$`)으로 평가됩니다. 예: `command(npm run (build\|lint\|test))`는 `npm run build` 및 `npm run test`와 일치합니다. | **질문(Ask)** |
| **`unsandboxed`** | `unsandboxed(prefix)` 또는 `unsandboxed(*)` | 정확한 단어/토큰 접두사로 명령어를 매핑합니다. 이 허용 기준과 일치하는 명령어는 컨테이너 격리 외부에서 실행됩니다(터미널 샌드박스가 활성화된 경우에만 적용됨). | **질문(Ask)** |
| **`mcp`** | `mcp(server/tool)` 또는 `mcp(*)` | 정확한 MCP 도구 또는 지정된 서버의 모든 도구와 일치합니다(로컬 `mcpl` 서버 및 원격 연결에 적용됨). `mcp(*)`는 모든 도구와 일치합니다. | **질문(Ask)** |

### 전역 와일드카드 구문

지원되는 모든 작업 유형에서 전역 와일드카드 `*`(`read_file(*)`, `command(*)`, `mcp(*)` 등)를 전달하면 해당 작업 네임스페이스 내의 모든 대상과 매치됩니다.

### 암묵적 권한 규칙

- **쓰기는 읽기를 포함함 (Write implies Read)**: 특정 경로에 대해 `write_file`을 허용하면 해당 경로에 대해 `read_file` 권한이 자동으로 부여됩니다.
- **읽기 거부는 쓰기 거부를 포함함 (Deny Read implies Deny Write)**: 특정 경로에 대해 `read_file`을 거부하면 해당 경로에 대한 `write_file` 권한도 즉시 차단됩니다.

### 크로스 플랫폼 경로 정규화

Antigravity는 macOS, Linux, Windows 중 어떤 환경에서 개발하더라도 권한 규칙이 원활하게 작동하도록 보장합니다. macOS와 Linux에서 경로는 표준 슬래시(`/`)를 사용합니다. Windows에서 Antigravity는 규칙 평가 전에 드라이브 문자(예: `C:`)를 제거하고 모든 백슬래시(`\`)를 슬래시(`/`)로 변환하여 경로를 자동으로 정규화합니다.

---

## 기본 시스템 동작 및 안전장치 

작업이 `allow`, `deny` 또는 `ask` 목록에 명시적으로 나열되지 않은 경우, 시스템은 안전한 기본 시스템값으로 대체(fallback)됩니다:

1. **작업 공간 자동 허용**: 표준 작동 모드에서 활성 프로젝트 디렉터리 내의 파일을 읽고 쓰는 것은 자동으로 허용됩니다.
2. **웹 브라우징 기본값은 질문(Ask)**: `read_url` 및 `execute_url` 동작은 기본적으로 **질문(Ask)**으로 설정됩니다. 에이전트가 허용 규칙이 구성되지 않은 웹 페이지로 이동하거나 조작하기 전에, 작업을 일시 중단하고 승인을 요청하는 프롬프트를 표시합니다.
3. **미구성 작업 기본값은 질문(Ask)**: 기타 구성되지 않은 모든 작업(`command`, `mcp`, `execute_url`, 작업 공간 외부 파일)의 기본값은 **질문(Ask)**입니다.

---

## 대화형 권한 프롬프트

에이전트가 승인이 필요한 작업(**Ask** 모드)을 실행할 때, TUI에 대화형 프롬프트 카드가 나타납니다.

파일, URL 또는 MCP 권한에 대해 **허용**을 확인하기 전에, 프롬프트 카드의 대상 문자열을 직접 편집하여 권한이 부여된 범위를 확장할 수 있습니다. (예: `/project/file.txt`와 같은 단일 파일 요청을 부모 디렉터리인 `/project`로 확장). CLI는 편집된 대상이 작업을 안전하게 커버하는지 검증하고 해당 턴의 남은 기간 동안 확장된 권한을 적용하여 관련 작업에 대해 프롬프트가 반복 표시되는 것을 방지합니다. _(참고: 터미널 명령어에 대해서는 범위 편집이 지원되지 않습니다.)_

---

## 구성 예시

`~/.gemini/antigravity-cli/settings.json` 파일에 다음 규칙을 추가합니다.

```json
{
  "permissions": {
    "allow": [
      "command(git)",
      "command(npm run (build|lint|test))",
      "unsandboxed(git push)",
      "read_file(/var/log/app)",
      "write_file(src/)",
      "read_url(google.com)",
      "mcp(linter/*)"
    ],
    "deny": [
      "command(rm -rf)",
      "command(curl .*)",
      "command(sudo)",
      "write_file(.git/)",
      "write_file(/home/user/.ssh)"
    ],
    "ask": ["command(*)", "execute_url(aws.amazon.com)", "mcp(sql/execute_mutation)"]
  }
}
```

## 참고 항목

- **[샌드박스 커스텀 설정](/docs/cli-sandbox)**: OS 수준의 컨테이너 격리 경계를 강제 적용합니다.
- **[플러그인 및 스킬](/docs/cli-plugins)**: 나만의 커스텀 스킬 슬래시 명령어를 생성합니다.
- **[설정, 렌더링 및 단축키](/docs/cli-settings)**: 키보드 단축키 및 버퍼를 구성합니다.
