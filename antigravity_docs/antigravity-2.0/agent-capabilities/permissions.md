---
slug: permissions
section: Antigravity 2.0
title: 권한
path:
  - Antigravity 2.0
  - 에이전트 기능
  - 권한
---

# 에이전트 권한

Antigravity는 자율적인 워크플로우를 활성화하는 동시에 사용자의 환경을 안전하게 보호하기 위해 강력하고 통합된 권한 엔진을 사용합니다. 에이전트가 수행하는 모든 민감한 작업은 `action(target)` 형식의 **권한 리소스**로 표현됩니다.

권한은 세 가지 고유한 액세스 목록에 따라 평가됩니다.
-   **Deny (거부)**: 작업이 즉시 차단됩니다.
-   **Ask (요청)**: 에이전트가 실행을 일시 중단하고 사용자의 명시적인 승인을 요청합니다.
-   **Allow (허용)**: 별도의 프롬프트 창 없이 작업이 자동으로 승인됩니다.

<Announcement>
icon: warning
color: "#fff8e1"
iconColor: "#f9ab00"
text: "<strong>우선순위 규칙:</strong> 충돌하는 규칙은 <strong>Deny &gt; Ask &gt; Allow</strong>의 우선순위에 따라 엄격하게 평가됩니다. 예를 들어, Ask 목록에 <code>command(*)</code>를 설정하고 Allow 목록에 <code>command(git)</code>를 설정한 경우, Ask 규칙이 우선적으로 적용되어 모든 명령 실행 전에 확인 프롬프트가 표시됩니다."
</Announcement>

## 지원하는 작업 및 매칭 규칙

| 작업 (Action) | 대상 형식 (Target Format) | 매칭 동작 | 기본 폴백 (Default Fallback) |
| :--- | :--- | :--- | :--- |
| `read_file` | `read_file(/path)`, `read_file(dir)`, 또는 `read_file(*)` | 절대 경로 또는 프로젝트 작업 공간 루트에 대한 상대 경로와 일치합니다. 포함된 모든 파일/폴더에 대해 재귀적인 읽기 권한을 부여합니다. `read_file(*)`를 사용하면 시스템의 모든 파일과 매칭됩니다. | **Ask** (작업 공간 내에서는 자동 허용됨) |
| `write_file` | `write_file(/path)` 또는 `write_file(*)` | `read_file`과 동일합니다. 지정된 동일한 대상 경로에 대해 `read_file` 권한을 암묵적으로 부여합니다. | **Ask** (작업 공간 내에서는 자동 허용됨) |
| `read_url` | `read_url(domain)` 또는 `read_url(*)` | 호스트 이름 및 하위 도메인과 일치합니다 (예: `google.com`은 `mail.google.com`을 포함함). URL 경로 세그먼트는 무시합니다. `read_url(*)`를 사용하면 모든 도메인과 매칭됩니다. | **Ask** |
| `execute_url` | `execute_url(domain)` 또는 `execute_url(*)` | 도메인 상에서 웹 요소를 작동(클릭, 입력)시키거나 대화형 브라우저 워크플로우를 구동하는 작업입니다. | **Ask** |
| `command` | `command(prefix)`, `command(regex)`, 또는 `command(*)` | 정확한 단어/토큰 접두사로 매칭합니다. 공백으로 구분된 각 토큰은 고정된 정규 표현식(`^(?:pattern)$`)으로 평가됩니다. 예: `command(npm run (build.*))`는 `npm run build` 및 `npm run build-prod`와 매치됩니다. | **Ask** |
| `unsandboxed` | `unsandboxed(prefix)`, `unsandboxed(regex)`, 또는 `unsandboxed(*)` | 정확한 단어/토큰 접두사로 명령을 매칭합니다. 이 권한과 매칭되는 명령은 컨테이너 격리 외부에서 실행됩니다 (터미널 샌드박싱이 활성화된 경우에만 적용됨). | **Ask** |
| `mcp` | `mcp(server/tool)`, `mcp(server/*)`, 또는 `mcp(*)` | 정확한 MCP 도구 또는 지정된 서버의 모든 도구와 일치합니다 (로컬 `mcpl` 서버 및 원격 연결 모두에 동일하게 적용됨). `mcp(*)`를 사용하면 모든 도구와 매칭됩니다. | **Ask** |

<Announcement>
icon: info
color: "#f5f5f5"
iconColor: "#4285f4"
text: "<strong>전역 와일드카드 구문(<code>*</code>):</strong> 지원되는 모든 작업 유형(예: <code>read_file(*)</code>, <code>command(*)</code>, <code>mcp(*)</code>)에서 전역 와일드카드 <code>*</code>를 전달하면 해당 작업 네임스페이스 내의 모든 대상과 매칭됩니다."
</Announcement>

### 플랫폼 전반에서의 read_url vs execute_url 이해하기
`read_url` 권한은 Antigravity의 세 가지 고유 영역에서 아웃바운드 웹 연결을 제어합니다.
1. **`read_url` 도구:** 에이전트가 리서치를 위해 웹 페이지 마크다운을 가져오기 위해 내부 `read_url_content` 도구를 사용할 때, 사용자의 `read_url` 허용 여부를 확인합니다.
2. **브라우저 서브 에이전트 및 도구:** Chrome 세션을 구동할 때, `read_url`은 대상 도메인을 로드하고 볼 수 있는 권한을 부여합니다. 반면, 대화형 UI 작동(버튼 클릭, 텍스트 입력)은 `execute_url`을 통해 별도로 제어됩니다.
3. **터미널 샌드박싱:** 샌드박스 모드에서 `read_url` 하에 승인된 모든 도메인은 컨테이너의 아웃바운드 네트워크 허용 목록(`AllowedDomains`)에 직접 컴파일되어, `curl` 또는 `npm`과 같은 명령이 승인된 호스트에 연결할 수 있도록 허용합니다.

### 크로스 플랫폼 명령 및 경로 매칭
Antigravity는 macOS, Linux, Windows 중 어떤 환경에서 개발하더라도 권한 규칙이 원활하게 작동하도록 보장합니다. macOS 및 Linux에서 경로는 표준 슬래시(`/`)를 사용합니다. Windows의 경우, Antigravity는 규칙을 평가하기 전에 드라이브 문자(예: `C:`)를 제거하고 모든 백슬래시(`\`)를 슬래시(`/`)로 변환하여 경로를 자동으로 정규화합니다.

## 암묵적 권한 규칙

*   **쓰기는 읽기를 포함함:** 특정 경로에 대해 `write_file`을 허용하면 해당 경로에 대해 `read_file` 권한이 자동으로 부여됩니다.
*   **읽기 거부는 쓰기 거부를 포함함:** 특정 경로에 대해 `read_file`을 거부하면 해당 경로에 대한 `write_file` 권한이 즉시 차단됩니다.

## 대화형 권한 프롬프트

에이전트가 승인이 필요한 작업(**Ask** 모드)을 수행할 때 에디터에 대화형 카드 창이 표시됩니다. 파일, URL 또는 MCP 권한에 대해 **승인(Allow)**을 클릭하기 전에 프롬프트 카드에서 대상 문자열을 직접 편집하여 권한 범위를 확장할 수 있습니다. (예: `/project/file.txt`와 같은 단일 파일 요청을 상위 디렉토리인 `/project`로 확장). Antigravity는 사용자가 편집한 대상이 제안된 작업을 안전하게 포함하는지 확인한 후, 이번 턴의 나머지 기간 동안 확장된 권한을 적용하여 유사한 작업에 대해 프롬프트가 반복해서 나타나는 것을 방지합니다. *(참고: 터미널 명령에 대해서는 범위 편집이 지원되지 않습니다).*

## 터미널 샌드박싱 (미리보기)

샌드박스가 활성화된 경우 권한 부여는 터미널 명령에도 적용됩니다.
*   `read_file`을 통해 승인된 경로는 샌드박스의 읽기 전용 파일 시스템 허용 목록에 동적으로 추가됩니다.
*   `write_file`을 통해 승인된 경로는 샌드박스의 읽기/쓰기 파일 시스템 허용 목록에 동적으로 추가됩니다.
*   `read_url`을 통해 승인된 도메인은 아웃바운드 네트워크 액세스 정책을 정의합니다.

<Announcement>
icon: info
color: "#f5f5f5"
iconColor: "#4285f4"
text: "<strong>샌드박스 지원 여부:</strong> 터미널 샌드박싱은 현재 macOS / Linux에서 미리보기로 제공되며, Windows 지원도 곧 추가될 예정입니다."
</Announcement>

## 시스템 기본 동작 및 가드레일 (Guardrails)

작업이 허용(Allow), 거부(Deny) 또는 요청(Ask) 목록에 명시적으로 나열되지 않은 경우, Antigravity는 안전한 시스템 기본 설정으로 폴백합니다.

1.  **웹 브라우징 기본값은 Ask:** `read_url` 및 `execute_url`에 대한 작업은 기본적으로 **Ask**로 설정됩니다. 허용 규칙이 구성되어 있지 않으면, 에이전트가 웹 페이지로 이동하거나 웹 페이지 상에서 동작을 실행하기 전에 실행을 일시 중단하고 사용자의 명시적인 승인을 요청합니다.
2.  **작업 공간(Workspace) 내 파일은 자동 허용:** 일반적인 작업 시, 활성화된 프로젝트 디렉토리 내부의 파일을 읽고 쓰는 것은 자동으로 허용됩니다. 그 외 설정되지 않은 모든 작업(`command`, `mcp`, `execute_url`, 작업 공간 외부 파일)은 기본적으로 **Ask**로 설정됩니다.

## 설정 예시

**허용 목록 (Allow list)** — 별도의 승인 요청 없이 실행되는 작업들:
```text
command(git)                       # 표준 git 명령
command(npm run (build|lint|test)) # 정규 표현식을 사용하여 안전한 npm 스크립트 허용
unsandboxed(git push)              # 샌드박스 외부에서 git push 허용
read_file(/var/log/app)            # 외부 로그 경로 읽기
write_file(src/)                   # 상대 경로 src/ 폴더 편집
read_url(google.com)               # Google 하위 도메인 데이터 가져오기
mcp(linter/*)                      # 린터 MCP 도구 실행
```

**거부 목록 (Deny list)** — 영구적으로 차단되는 작업들:
```text
command(rm -rf)                    # 파괴적인 삭제 명령 차단
command(curl .*)                   # 검증되지 않은 curl 다운로드 차단
command(sudo)                      # sudo 권한 차단
write_file(.git/)                  # Git 히스토리 보호
write_file(/home/user/.ssh)        # SSH 키 보호
```

**요청 목록 (Ask list)** — 수동 확인을 위해 일시 중단되는 작업들:
```text
command(*)                         # 모든 명령에 대해 확인 요청
execute_url(aws.amazon.com)        # AWS 콘솔 작동에 대해 확인 요청
mcp(sql/execute_mutation)          # SQL 쿼리를 수정하는 작업에 대해 확인 요청
```
