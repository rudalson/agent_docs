> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude Code 설정

> 전역 설정, 프로젝트 수준 설정 및 환경 변수를 사용하여 Claude Code를 구성하세요.

Claude Code는 사용자의 요구 사항에 맞춰 동작을 구성할 수 있는 다양한 설정을 제공합니다. 대화형 세션에서 `/config` 명령어를 실행하면 상태 정보를 확인하고 구성 옵션을 수정할 수 있는 탭 형태의 Settings 인터페이스가 열립니다. {/* min-version: 2.1.181 */}v2.1.181부터는 `/config`에 `key=value`를 전달하여 인터페이스를 열지 않고 단일 옵션을 변경할 수 있습니다(예: `/config verbose=true`).

## 구성 스코프 (Configuration scopes)

Claude Code는 구성이 적용되는 위치와 공유 대상을 결정하기 위해 스코프 시스템을 사용합니다. 스코프를 이해하면 개인 용도, 팀 협업 또는 엔터프라이즈 배포를 위해 Claude Code를 구성하는 방식을 결정하는 데 도움이 됩니다.

### 사용 가능한 스코프

| 스코프 | 위치 | 영향을 받는 대상 | 팀과 공유 여부 |
| :--- | :--- | :--- | :--- |
| **관리형(Managed)** | 서버 관리형 설정, plist / 레지스트리, 또는 시스템 수준 `managed-settings.json` | 서버 관리형 전달의 경우 모든 조직 구성원; plist, HKLM 레지스트리 및 파일 전달의 경우 머신의 모든 사용자; HKCU 레지스트리 전달의 경우 현재 사용자 | 예 (IT 부서에서 배포) |
| **사용자(User)** | `~/.claude/` 디렉터리 | 모든 프로젝트에 걸쳐 사용자 본인 | 아니요 |
| **프로젝트(Project)** | 리포지토리 내의 `.claude/` | 이 리포지토리의 모든 협업자 | 예 (git에 커밋됨) |
| **로컬(Local)** | 리포지토리 루트의 `.claude/settings.local.json` | 이 리포지토리에서의 사용자 본인만 | 아니요 (Claude Code 생성 시 gitignore 처리됨) |

### 각 스코프를 사용하는 경우

**관리형 스코프**는 다음 용도에 적합합니다:

* 조직 전체에 적용되어야 하는 보안 정책
* 오버라이드할 수 없는 컴플라이언스 요구 사항
* IT/DevOps에 의해 배포되는 표준화된 구성

**사용자 스코프**는 다음 용도에 가장 적합합니다:

* 모든 곳에서 적용하고자 하는 개인 기본 설정 (테마, 편집기 설정)
* 모든 프로젝트에서 사용하는 도구 및 플러그인
* API 키 및 인증 정보 (안전하게 저장됨)

**프로젝트 스코프**는 다음 용도에 가장 적합합니다:

* 팀 공유 설정 (권한, 훅, MCP 서버)
* 전체 팀이 가져야 하는 플러그인
* 협업자 간의 공통 도구 표준화

**로컬 스코프**는 다음 용도에 가장 적합합니다:

* 특정 프로젝트에 대한 개인 재정의(override)
* 팀과 공유하기 전 구성 테스트
* 다른 사용자에게는 작동하지 않는 머신 특정 설정

### 스코프 간 상호작용 방식

동일한 설정이 여러 스코프에 나타나면 Claude Code는 우선순위 순서대로 적용합니다:

1. **관리형(Managed)** (가장 높음): 어떠한 것으로도 오버라이드할 수 없음
2. **명령줄 인수**: 임시 세션 오버라이드
3. **로컬(Local)**: 프로젝트 및 사용자 설정을 오버라이드함
4. **프로젝트(Project)**: 사용자 설정을 오버라이드함
5. **사용자(User)** (가장 낮음): 다른 스코프에서 설정을 지정하지 않았을 때 적용됨

예를 들어 사용자 설정에서 `spinnerTipsEnabled`를 `true`로 설정하고 프로젝트 설정에서 `false`로 설정한 경우 프로젝트 값이 적용됩니다. 권한 규칙은 오버라이드 대신 스코프 전반에 걸쳐 병합되므로 다르게 작동합니다. [설정 우선순위](#설정-우선순위)를 참조하세요.

### 스코프를 사용하는 항목

스코프는 다양한 Claude Code 기능에 적용됩니다:

| 기능 | 사용자 위치 | 프로젝트 위치 | 로컬 위치 |
| :--- | :--- | :--- | :--- |
| **설정** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **서브에이전트** | `~/.claude/agents/` | `.claude/agents/` | 없음 |
| **MCP 서버** | `~/.claude.json` | `.mcp.json` | `~/.claude.json` (프로젝트별) |
| **플러그인** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **CLAUDE.md** | `~/.claude/CLAUDE.md` | `CLAUDE.md` 또는 `.claude/CLAUDE.md` | `CLAUDE.local.md` |

Windows에서 `~/.claude`로 표시된 경로는 `%USERPROFILE%\.claude`로 해석됩니다.

---

## 설정 파일

`settings.json` 파일은 계층적 설정을 통해 Claude Code를 구성하는 공식 메커니즘입니다:

* **사용자 설정**은 `~/.claude/settings.json`에 정의되며 모든 프로젝트에 적용됩니다.
* **프로젝트 설정**은 프로젝트 디렉터리에 저장됩니다:
  * `.claude/settings.json`: 소스 제어에 체크인되어 팀과 공유되는 설정
  * `.claude/settings.local.json`: 체크인되지 않는 설정으로, 개인 취향 및 실험에 유용합니다. Claude Code가 `.claude/settings.local.json`을 생성할 때 git이 이 파일을 무시하도록 구성합니다. 직접 파일을 작성하는 경우 gitignore에 수동으로 추가하세요.

    Claude Code는 [작업 트리(worktrees)](/docs/en/worktrees)를 통해 메인 체크아웃으로 확인된 Git 리포지토리 루트에서 이 파일을 읽고 씁니다. 따라서 하나의 파일로 리포지토리의 모든 하위 디렉터리나 작업 트리에서 시작된 세션을 적용합니다. 파일이 시작 디렉터리에 남는 경우는 Git 리포지토리 외부, 리포지토리 루트가 홈 디렉터리인 경우, [Agent SDK](/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources) 세션의 세 가지 예외뿐입니다.

    <Info>v2.1.211 이전에는 파일이 항상 시작 디렉터리에 상주했습니다. Claude Code는 이전 버전이 남겨둔 `.claude/settings.local.json`도 여전히 읽습니다. 두 파일이 동일한 키를 설정하는 경우, 두 파일의 권한 규칙이 모두 유효하다는 점을 제외하고 리포지토리 루트의 값이 승리합니다.</Info>

    또한 Claude Code는 Bash 명령 승인과 같은 영구적인 "다시 묻지 않음" [권한 승인](/docs/en/permissions#permission-system)을 이 파일에 저장합니다.

    이 파일은 리포지토리 소유가 아닌 개인 소유이므로, `.claude/settings.json` 허용 규칙에 필요한 [워크스페이스 신뢰](/docs/en/permissions#project-allow-rules-and-workspace-trust) 단계 없이 권한 `allow` 규칙이 적용됩니다. 예를 들어 커밋을 통해 리포지토리가 이 파일을 제공하는 경우 워크스페이스 신뢰가 여전히 적용됩니다.
* **관리형 설정**: 중앙 관리가 필요한 조직을 위해 Claude Code는 관리형 설정에 대한 여러 배포 메커니즘을 지원합니다. 모두 동일한 JSON 형식을 사용하며 사용자 또는 프로젝트 설정으로 오버라이드할 수 없습니다:

  * **서버 관리형 설정**: 로그인 시 claude.ai 관리 콘솔을 통해 Anthropic 서버에서 원격으로 전달되거나 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)에서 전달됩니다. [서버 관리형 설정](/docs/en/server-managed-settings)을 참조하세요.
  * **MDM/OS 수준 정책**: macOS 및 Windows에서 네이티브 디바이스 관리를 통해 전달됩니다:
    * macOS: `com.anthropic.claudecode` 관리형 환경 설정 도메인. plist의 최상위 키는 `managed-settings.json`을 미러링하며 중첩 설정은 딕셔너리로, 배열은 plist 배열로 제공됩니다. Jamf, Iru (Kandji) 또는 유사한 MDM 도구의 구성 프로필을 통해 배포합니다.
    * Windows: JSON이 포함된 `Settings` 값(REG_SZ 또는 REG_EXPAND_SZ)이 있는 `HKLM\SOFTWARE\Policies\ClaudeCode` 레지스트리 키 (그룹 정책 또는 Intune을 통해 배포)
    * Windows (사용자 수준): `HKCU\SOFTWARE\Policies\ClaudeCode` (가장 낮은 정책 우선순위, 관리자 수준 소스가 없을 때만 사용됨)
  * **파일 기반**: 시스템 디렉터리에 배포되는 `managed-settings.json` 및 `managed-mcp.json`:

    * macOS: `/Library/Application Support/ClaudeCode/`
    * Linux 및 WSL: `/etc/claude-code/`
    * Windows: `C:\Program Files\ClaudeCode\`

    <Warning>
      레거시 Windows 경로인 `C:\ProgramData\ClaudeCode\managed-settings.json`은 v2.1.75부터 더 이상 지원되지 않습니다. 해당 위치에 설정을 배포한 관리자는 파일들을 `C:\Program Files\ClaudeCode\managed-settings.json`으로 이전해야 합니다.
    </Warning>

    파일 기반 관리형 설정은 `managed-settings.json`과 동일한 시스템 디렉터리에 있는 `managed-settings.d/` 드롭인 디렉터리도 지원합니다. 이를 통해 독립적인 팀들이 단일 파일을 번거롭게 수정하지 않고도 개별 정책 조각을 배포할 수 있습니다.

    systemd 관례에 따라 `managed-settings.json`이 기본으로 먼저 병합된 다음 드롭인 디렉터리의 모든 `*.json` 파일이 알파벳순으로 정렬되어 그 위에 병합됩니다. 단일 값에 대해 나중 파일이 이전 파일을 오버라이드하고, 배열은 연결 및 중복 제거되며, 객체는 깊은 병합(deep-merge)됩니다. `.`으로 시작하는 숨김 파일은 무시됩니다.

    병합 순서를 제어하려면 `10-telemetry.json` 및 `20-security.json`과 같은 숫자 접두사를 사용하세요.

  자세한 내용은 [관리형 설정](/docs/en/permissions#managed-only-settings) 및 [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요.

  이 [리포지토리](https://github.com/anthropics/claude-code/tree/main/examples/mdm)에는 Jamf, Iru (Kandji), Intune, 그룹 정책용 시작 배포 템플릿이 포함되어 있습니다. 이를 시작점으로 활용하고 필요에 맞게 조정하세요.

  <Note>
    관리형 배포는 `strictKnownMarketplaces`를 사용하여 **플러그인 마켓플레이스 추가**를 제한할 수도 있습니다. 자세한 내용은 [관리형 마켓플레이스 제한 사항](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)을 참조하세요.
  </Note>
* **기타 구성**은 `~/.claude.json`에 저장됩니다. 이 파일에는 OAuth 세션, 사용자 및 로컬 스코프에 대한 [MCP 서버](/docs/en/mcp) 구성, 프로젝트별 상태(허용된 도구, 신뢰 설정) 및 다양한 캐시가 포함되어 있습니다. 프로젝트 스코프 MCP 서버는 `.mcp.json`에 별도로 저장됩니다.

<Note>
  Claude Code는 데이터 손실을 방지하기 위해 구성 파일의 타임스탬프가 지정된 백업을 자동으로 생성하고 가장 최근의 5개 백업을 유지합니다.
</Note>

다음 예시는 위의 설정 파일 위치 어디서나 작동합니다. 파일을 저장하는 위치에 따라 적용 위치가 결정됩니다:

* 모든 프로젝트에 적용하려면 `~/.claude/settings.json`으로 저장하세요. 이 파일은 프로젝트가 아닌 홈 디렉터리에 존재하므로 어떤 프로젝트를 열든 상관없이 모든 세션에서 Claude Code가 읽습니다.
* 한 프로젝트의 협업자들과 공유하려면 해당 프로젝트의 `.claude/settings.json`으로 저장하세요. Claude Code는 세션이 실행되는 디렉터리에서 이 파일을 읽으므로 해당 프로젝트에만 적용되고 소스 제어에 체크인하면 모든 협업자가 동일한 설정을 갖게 됩니다.

```json Example settings.json theme={null}
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  },
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review our code guidelines at docs.acme.com",
    "Reminder: Code reviews required for all PRs",
    "New security policy in effect"
  ]
}
```

위 예시의 `$schema` 라인은 Claude Code 설정에 대한 [공식 JSON 스키마](https://json.schemastore.org/claude-code-settings.json)를 가리킵니다. 이를 `settings.json`에 추가하면 VS Code, Cursor 및 JSON 스키마 유효성 검사를 지원하는 모든 편집기에서 자동 완성 및 인라인 검증이 활성화됩니다.

게시된 스키마는 주기적으로 업데이트되며 가장 최근 CLI 릴리스에 추가된 설정이 포함되지 않을 수 있으므로 최근 문서화된 필드에 유효성 검사 경고가 발생하더라도 구성이 유효하지 않은 것은 아닙니다.

<Tip>
  설정 파일을 편집한 후 Claude Code 내부에서 `/status`를 실행하여 제대로 로드되었는지 확인하세요. `Setting sources` 라인은 현재 세션에 로드된 각 설정 소스를 나열합니다. 적어도 하나의 설정이 있는 경우 소스가 로드되어 표시되므로 손상된 JSON이 포함된 파일은 설정이 들어있더라도 표시되지 않습니다. [활성 설정 확인하기](#활성-설정-확인하기)를 참조하세요.
</Tip>

### 편집 사항이 적용되는 시점

Claude Code는 설정 파일을 감시하고 변경될 때 다시 로드하므로 대부분의 키 편집은 재시작 없이 실행 중인 세션에 바로 적용됩니다. 여기에는 `permissions`, `hooks`, `apiKeyHelper`와 같은 자격 증명 헬퍼가 포함됩니다. 다시 로드는 사용자, 프로젝트, 로컬 및 관리형 설정을 포함하며 감지된 각 변경 사항에 대해 [`ConfigChange` 훅](/docs/en/hooks#configchange)이 실행됩니다.

몇 가지 키는 세션 시작 시 한 번 읽히며 다음 재시작 시 적용됩니다:

* `model`: 세션 중간에 전환하려면 [`/model`](/docs/en/model-config#setting-your-model)을 사용하세요.
* [`outputStyle`](/docs/en/output-styles): `/clear` 또는 재시작 시 다시 작성되는 시스템 프롬프트의 일부입니다.

### 관리형 설정의 유효하지 않은 항목

관리형 설정은 관대하게 파싱됩니다. 관리형 구성에 스키마 유효성 검사에 실패한 항목이 포함되어 있으면 Claude Code는 해당 항목을 제거하고 경고를 기록하며 나머지 유효한 모든 정책을 강제 적용합니다. 단 하나의 오타로 인해 조직의 나머지 정책이 비활성화되지 않습니다. [`/doctor`](/docs/en/debug-your-config#check-resolved-settings)를 실행하여 소스 파일 및 필드와 함께 제거된 항목을 확인하세요.

이 동작은 [서버 관리형 설정](/docs/en/server-managed-settings), MDM을 통해 배포된 plist 및 레지스트리 정책, `managed-settings.json` 파일 등 세 가지 전달 메커니즘 전체에서 동일합니다. Claude Code v2.1.169 이상이 필요합니다.

보안 강제 필드는 유효하지 않은 항목이 존재할 때 전체를 제거하는 대신 필드별로 다르게 처리됩니다:

| 필드 | 유효하지 않은 항목 존재 시 동작 |
| :--- | :--- |
| `allowedMcpServers` | 값이 수정될 때까지 MCP 서버가 허용되지 않도록 빈 허용 목록으로 강제 적용됩니다. 개별 유효하지 않은 항목은 제거되고 유효한 부분 집합이 강제 적용됩니다. |
| `allowManagedMcpServersOnly` | `true`로 처리됩니다. |
| `availableModels` | {/* min-version: 2.1.175 */}값이 수정될 때까지 기본(Default) 모델만 사용할 수 있도록 빈 허용 목록으로 강제 적용됩니다. 문자열이 아닌 개별 항목은 제거되고 유효한 부분 집합이 강제 적용됩니다. v2.1.175 이상에 적용됩니다. |
| `enforceAvailableModels` | {/* min-version: 2.1.175 */}`true`로 처리됩니다. v2.1.175 이상에 적용됩니다. |
| `forceLoginOrgUUID` | 값이 수정될 때까지 어떤 조직도 로그인할 수 없습니다. |
| `deniedMcpServers` | 개별 유효하지 않은 항목은 제거되고 유효한 부분 집합이 강제 적용됩니다. 완전히 유효하지 않은 값은 경고와 함께 삭제됩니다. 모든 서버를 거부하면 정책에 언급되지 않은 서버까지 차단되기 때문입니다. |
| `sandbox.credentials` | {/* min-version: 2.1.191 */}`files` 또는 `envVars`에서 개별 유효하지 않은 항목은 경고와 함께 제거되고 유효한 부분 집합이 강제 적용됩니다. 완전히 유효하지 않은 `credentials` 값은 경고와 함께 삭제되지만 나머지 `sandbox`는 계속 적용됩니다. v2.1.191 이상에 적용됩니다. |

`requiredMinimumVersion` 및 `requiredMaximumVersion`은 설계상 실패 시 허용(fail-open)입니다. 유효하지 않은 값은 강제 적용되지 않고 제거되므로 잘못된 정책 푸시로 인해 Claude Code 시작이 차단되지 않습니다.

유효성 검사 오류는 세 곳에서 노출됩니다:

* 대화형 세션 시작 시 유효하지 않은 항목을 나열하는 대화 상자가 표시됩니다.
* `-p`를 사용한 비대화형 실행은 stderr에 요약을 출력합니다.
* [`claude doctor`](/docs/en/debug-your-config)는 각 유효하지 않은 항목을 소스 및 필드와 함께 나열합니다.

전사적으로 배포하기 전에 테스트 머신에서 `claude doctor`를 실행하여 정책 변경 사항을 검증하세요.

이 관대한 동작은 관리형 설정에만 적용됩니다. 사용자, 프로젝트 및 로컬 설정 파일은 엄격하게 유지됩니다. 유효성 검사에 실패한 파일은 전체가 거부되어 보고됩니다.

### 사용 가능한 설정

`settings.json`은 다양한 옵션을 지원합니다:

| 키 | 설명 | 예시 |
| :--- | :--- | :--- |
| `advisorModel` | 서버 측 [어드바이저 도구](/docs/en/advisor)를 위한 모델. 모델 별칭 `"opus"`, `"sonnet"` 또는 전체 모델 ID를 허용합니다. `/advisor`를 실행하면 자동으로 기록됩니다. 어드바이저를 비활성화하려면 설정을 해제하세요. {/* min-version: 2.1.210 */}[Claude Code는 Fable 5를 어드바이저로 제공하지 않습니다](/docs/en/advisor#enable-the-advisor): 저장된 `"fable"` 값은 어드바이저를 첨부하지 않으며 오류를 유발하지 않습니다. | `"opus"` |
| `agent` | 메인 스레드를 지정된 이름의 서브에이전트로 실행하고 `claude agents`에서 발송되는 세션의 기본 에이전트를 설정합니다. 해당 서브에이전트의 시스템 프롬프트, 도구 제한 및 모델을 적용합니다. [서브에이전트 명시적 호출](/docs/en/sub-agents#invoke-subagents-explicitly)을 참조하세요. | `"code-reviewer"` |
| `agentPushNotifEnabled` | {/* min-version: 2.1.119 */}**기본값**: `false`. [원격 제어(Remote Control)](/docs/en/remote-control)가 연결되어 있을 때 긴 작업이 끝나는 등 Claude가 휴대폰으로 선제적 푸시 알림을 보내도록 허용합니다. `/config`에서 **Push when Claude decides**로 표시됩니다. [모바일 푸시 알림](/docs/en/remote-control#mobile-push-notifications)을 참조하세요. Claude Code v2.1.119 이상이 필요합니다. | `true` |
| `allowAllClaudeAiMcps` | (관리형 설정 전용) 배포된 `managed-mcp.json`과 함께 claude.ai 커넥터를 로드합니다. 그렇지 않으면 `managed-mcp.json`이 독점 제어권을 갖고 이들을 억제합니다. [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. | `true` |
| `allowedChannelPlugins` | (관리형 설정 전용) 메시지를 푸시할 수 있는 채널 플러그인의 허용 목록. 설정 시 기본 Anthropic 허용 목록을 대체합니다. Undefined = 기본값으로 대체, 빈 배열 = 모든 채널 플러그인 차단. `channelsEnabled: true`가 필요합니다. [채널 플러그인 실행 제한](/docs/en/channels#restrict-which-channel-plugins-can-run)을 참조하세요. | `[{ "marketplace": "claude-plugins-official", "plugin": "telegram" }]` |
| `allowedHttpHookUrls` | HTTP 훅이 타깃으로 지정할 수 있는 URL 패턴의 허용 목록. 와일드카드로 `*`를 지원합니다. 설정 시 일치하지 않는 URL을 가진 훅은 차단됩니다. Undefined = 제한 없음, 빈 배열 = 모든 HTTP 훅 차단. 배열은 설정 소스 전반에 걸쳐 병합됩니다. [훅 구성](#훅-구성)을 참조하세요. | `["https://hooks.example.com/*"]` |
| `allowedMcpServers` | managed-settings.json에 설정된 경우 사용자가 구성할 수 있는 MCP 서버의 허용 목록. Undefined = 제한 없음, 빈 배열 = 완전 차단. 모든 스코프에 적용됩니다. 거부 목록이 우선합니다. [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. | `[{ "serverName": "github" }]` |
| `allowManagedHooksOnly` | (관리형 설정 전용) 관리형 훅, SDK 훅, 그리고 관리형 설정 `enabledPlugins`에서 강제 활성화된 플러그인의 훅만 로드됩니다. 사용자, 프로젝트 및 기타 모든 플러그인 훅은 차단됩니다. [훅 구성](#훅-구성)을 참조하세요. | `true` |
| `allowManagedMcpServersOnly` | (관리형 설정 전용) 관리형 설정의 `allowedMcpServers`만 존중됩니다. `deniedMcpServers`는 여전히 모든 소스에서 병합됩니다. 사용자가 여전히 MCP 서버를 추가할 수 있지만 관리자가 정의한 허용 목록만 적용됩니다. [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. | `true` |
| `allowManagedPermissionRulesOnly` | (관리형 설정 전용) 사용자 및 프로젝트 설정이 `allow`, `ask` 또는 `deny` 권한 규칙을 정의하지 못하도록 방지합니다. 관리형 설정의 규칙만 적용됩니다. [관리 전용 설정](/docs/en/permissions#managed-only-settings)을 참조하세요. | `true` |
| `alwaysThinkingEnabled` | 기본적으로 모든 세션에 대해 [확장 생각(extended thinking)](/docs/en/model-config#extended-thinking)을 활성화합니다. 직접 편집하기보다는 일반적으로 `/config` 명령을 통해 구성됩니다. 이 설정과 상관없이 생각을 완전히 끄려면 `env`에서 [`MAX_THINKING_TOKENS=0`](/docs/en/env-vars)을 설정하세요. 이는 생각을 끌 수 없는 Fable 5를 제외한 Anthropic API에서 생각을 비활성화합니다. [서드파티 제공업체](/docs/en/third-party-integrations)에서는 이 매개변수를 생략하며 적응형 추론 모델이 계속 생각할 수 있습니다. | `true` |
| `apiKeyHelper` | 모델 요청을 위한 `X-Api-Key` 및 `Authorization: Bearer` 헤더로 전송할 인증 값을 생성하기 위해 시스템 셸(macOS/Linux: `/bin/sh`, Windows: `cmd`)을 통해 실행되는 사용자 지정 명령. 새로 고침 간격은 [`CLAUDE_CODE_API_KEY_HELPER_TTL_MS`](/docs/en/env-vars)로 설정하세요. | `/bin/generate_temp_api_key.sh` |
| `askUserQuestionTimeout` | {/* min-version: 2.1.200 */}**기본값**: `"never"`. 응답되지 않은 [`AskUserQuestion`](/docs/en/tools-reference) 대화 상자가 이미 선택한 옵션으로 자동 진행되기 전의 유휴 시간. `"60s"`, `"5m"`, `"10m"` 또는 `"never"`를 허용합니다. 기본값을 사용하면 답변할 때까지 질문이 대기합니다. `/config`에서 **Question auto-continue timeout**으로 표시되며 이 키를 사용자 설정에 기록합니다. 프로젝트나 로컬 설정에서는 읽지 않습니다. Claude Code v2.1.200 이상이 필요합니다. | `"5m"` |
| `attribution` | Git 커밋 및 풀 리퀘스트에 대한 귀속(attribution)을 맞춤 설정합니다. [귀속 설정](#귀속-설정)을 참조하세요. | `{"commit": "🤖 Generated with Claude Code", "pr": ""}` |
| `autoCompactEnabled` | {/* min-version: 2.1.119 */}**기본값**: `true`. 컨텍스트가 한계에 다다를 때 대화를 자동으로 압축합니다. `/config`에서 **Auto-compact**로 표시됩니다. 환경 변수로 비활성화하려면 `env`에 [`DISABLE_AUTO_COMPACT`](/docs/en/env-vars)를 설정하세요. | `false` |
| `autoMemoryDirectory` | [자동 메모리](/docs/en/memory#storage-location) 저장을 위한 사용자 지정 디렉터리. 절대 경로 또는 `~/`로 시작하는 경로를 허용합니다. 프로젝트 또는 로컬 설정에서는 클론된 리포지토리가 이 파일을 제공할 수 있으므로 워크스페이스 신뢰 대화 상자를 수락한 후에만 이 설정이 존중됩니다. | `"~/my-memory-dir"` |
| `autoMemoryEnabled` | **기본값**: `true`. [자동 메모리](/docs/en/memory#enable-or-disable-auto-memory)를 활성화합니다. `false`이면 Claude가 자동 메모리 디렉터리를 읽거나 쓰지 않습니다. 세션 중에 `/memory`로 이를 토글할 수도 있습니다. 환경 변수로 비활성화하려면 `env`에 [`CLAUDE_CODE_DISABLE_AUTO_MEMORY`](/docs/en/env-vars)를 설정하세요. | `false` |
| `autoMode` | [자동 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 분류기가 차단하거나 허용할 대상을 사용자 지정합니다. 문장 규칙의 `environment`, `allow`, `soft_deny`, `hard_deny` 배열을 포함합니다. 해당 위치에서 내장 규칙을 상속하려면 배열에 리터럴 문자열 `"$defaults"`를 포함하세요. [자동 모드 구성하기](/docs/en/auto-mode-config)를 참조하세요. 사용자 설정, `--settings` 플래그 및 관리형 설정에서만 읽습니다. 프로젝트 `.claude/settings.json` 및 로컬 `.claude/settings.local.json`에서는 무시됩니다. | `{"soft_deny": ["$defaults", "Never run terraform apply"]}` |
| `autoMode.classifyAllShell` | {/* min-version: 2.1.193 */}**기본값**: `false`. `true`인 경우 자동 모드가 활성화되어 있는 동안 모든 Bash 및 PowerShell 허용 규칙을 일시 중지하여 임의 코드 실행 패턴과 일치하는 규칙뿐만 아니라 모든 셸 명령이 분류기를 거치도록 합니다. [모든 셸 명령을 분류기로 라우팅](/docs/en/auto-mode-config#route-all-shell-commands-through-the-classifier)을 참조하세요. Claude Code v2.1.193 이상이 필요합니다. | `true` |
| `autoScrollEnabled` | **기본값**: `true`. [전체 화면 렌더링](/docs/en/fullscreen)에서 대화 하단으로 새 출력을 팔로우합니다. `/config`에서 **Auto-scroll**로 표시됩니다. 이 설정이 꺼져 있어도 권한 프롬프트는 화면 뷰로 스크롤됩니다. | `false` |
| `autoUpdatesChannel` | **기본값**: `"latest"`. 업데이트에 대해 팔로우할 릴리스 채널. 보통 약 1주일 이전의 버전이자 주요 회귀를 건너뛰는 버전을 위해 `"stable"`을 사용하세요. | `"stable"` |
| `availableModels` | 사용자가 메인 세션, [서브에이전트](/docs/en/sub-agents), [스킬](/docs/en/skills) 및 [어드바이저](/docs/en/advisor)에 선택할 수 있는 모델을 제한합니다. `enforceAvailableModels`도 설정되어 있지 않으면 Default 옵션에 영향을 주지 않습니다. [모델 선택 제한](/docs/en/model-config#restrict-model-selection)을 참조하세요. | `["sonnet", "haiku"]` |
| `awaySummaryEnabled` | 몇 분 동안 자리를 비운 후 터미널로 돌아왔을 때 한 줄 세션 요약을 표시합니다. 비활성화하려면 `false`로 설정하거나 `/config`에서 Session recap을 끄세요. [`CLAUDE_CODE_ENABLE_AWAY_SUMMARY`](/docs/en/env-vars)와 동일합니다. | `true` |
| `awsAuthRefresh` | `.aws` 디렉터리를 수정하는 사용자 지정 스크립트 ([고급 자격 증명 구성](/docs/en/amazon-bedrock#advanced-credential-configuration) 참조) | `aws sso login --profile myprofile` |
| `awsCredentialExport` | AWS 자격 증명이 포함된 JSON을 출력하는 사용자 지정 스크립트 ([고급 자격 증명 구성](/docs/en/amazon-bedrock#advanced-credential-configuration) 참조) | `/bin/generate_aws_grant.sh` |
| `axScreenReader` | {/* min-version: 2.1.181 */}스크린 리더에 친화적인 출력 렌더링: 장식 테두리나 애니메이션이 없는 평면 텍스트. 스크린 리더 모드는 클래식 렌더러를 사용하므로 활성화되어 있는 동안 `tui` 설정은 효과가 없습니다. 연결된 [백그라운드 세션](/docs/en/agent-view)은 여전히 전체 화면으로 렌더링됩니다. [`CLAUDE_AX_SCREEN_READER`](/docs/en/env-vars) 환경 변수와 [`--ax-screen-reader`](/docs/en/cli-reference#cli-flags) 플래그가 우선합니다. Claude Code v2.1.181 이상이 필요합니다. | `true` |
| `blockedMarketplaces` | (관리형 설정 전용) 마켓플레이스 소스의 차단 목록. 마켓플레이스 추가 및 플러그인 설치, 업데이트, 새로 고침, 자동 업데이트 시 강제 적용되므로 정책이 설정되기 전에 추가된 마켓플레이스를 통해 플러그인을 가져올 수 없습니다. 차단된 소스는 다운로드 전에 확인되므로 파일 시스템에 전혀 닿지 않습니다. [관리형 마켓플레이스 제한 사항](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)을 참조하세요. | `[{ "source": "github", "repo": "untrusted/plugins" }]` |
| `browserExternalPageTools` | (관리형 설정 전용) 데스크톱 앱의 [브라우저 창](/docs/en/desktop#browse-external-sites)에서 Claude가 외부 페이지를 읽거나 조치를 취하는 도구를 사용하지 못하도록 `"disabled"`로 설정합니다. 사용자는 여전히 외부 사이트로 직접 이동할 수 있으며 로컬 개발 서버 미리보기는 영향을 받지 않습니다. | `"disabled"` |
| `channelsEnabled` | (관리형 설정 전용) 조직에 대한 [채널](/docs/en/channels)을 허용합니다. claude.ai Team 및 Enterprise 플랜에서는 이 설정이 설정되어 있지 않거나 `false`일 때 채널이 차단됩니다. API 키 인증을 사용하는 [Anthropic Console](/docs/en/authentication#claude-console-authentication) 계정의 경우 조직에서 관리형 설정을 배포하지 않는 한 기본적으로 채널이 허용되며, 배포 시 이 키를 `true`로 설정해야 합니다. | `true` |
| `claudeMd` | (관리형 설정 전용) 조직 관리 메모리로 주입되는 CLAUDE.md 스타일 지침. 관리형 또는 정책 설정에 설정된 경우에만 존중되며 사용자, 프로젝트 및 로컬 설정에서는 무시됩니다. [조직 전체 CLAUDE.md](/docs/en/memory#deploy-organization-wide-claude-md)를 참조하세요. | `"Always run make lint before committing."` |
| `claudeMdExcludes` | [메모리](/docs/en/memory)를 로드할 때 건너뛸 `CLAUDE.md` 파일의 글롭 패턴 또는 절대 경로. 패턴은 절대 파일 경로와 일치합니다. 사용자, 프로젝트 및 로컬 메모리에만 적용되며 관리형 정책 파일은 제외할 수 없습니다. | `["**/vendor/**/CLAUDE.md"]` |
| `cleanupPeriodDays` | **기본값**: `30`일, 최소 `1`일. Claude Code는 시작 시 이 기간보다 오래된 [세션 파일 및 기타 애플리케이션 데이터](/docs/en/claude-directory#cleaned-up-automatically)를 삭제합니다. `0`으로 설정하면 유효성 검사 오류로 실패합니다. 시작 시 [고아 작업 트리](/docs/en/worktrees#clean-up-worktrees)를 자동 제거하는 데에도 동일한 기간이 적용됩니다. {/* min-version: 2.1.203 */}Claude Code가 설정 파일을 읽거나 파싱할 수 없는 경우, [관리형 설정](/docs/en/server-managed-settings)이 `cleanupPeriodDays`를 제공하여 해당 값으로 정리가 실행되지 않는 한, 설정 파일을 수정할 때까지 보존 정리 작업을 일시 중지하고 `/status`에 경고를 표시합니다. v2.1.203 이전에는 해당 상태에서 30일 기본값으로 정리가 실행되어 더 긴 `cleanupPeriodDays`가 보존하려던 트랜스크립트를 삭제할 수 있었습니다. 30일보다 새 파일은 절대 제거되지 않았습니다. 트랜스크립트 쓰기를 완전히 비활성화하려면 [`CLAUDE_CODE_SKIP_PROMPT_HISTORY`](/docs/en/env-vars) 환경 변수를 설정하세요. 비대화형 모드에서는 `-p`와 함께 `--no-session-persistence`를 전달하거나 Agent SDK에서 `persistSession: false`를 설정하세요. | `20` |
| `companyAnnouncements` | 시작 시 사용자에게 표시할 공지 사항. 여러 공지 사항이 제공된 경우 무작위로 순환하여 표시됩니다. | `["Welcome to Acme Corp! Review our code guidelines at docs.acme.com"]` |
| `defaultShell` | **기본값**: `"bash"`, 또는 Bash를 사용할 수 없을 때 Windows에서 `"powershell"`. 입력 상자 `!` 명령에 대한 기본 셸. `"bash"` 또는 `"powershell"`을 허용합니다. [PowerShell 도구](/docs/en/tools-reference#powershell-tool)가 활성화되어 있을 때 `"powershell"`을 설정하면 대화형 `!` 명령이 PowerShell을 통해 라우팅됩니다. Git Bash가 없는 Windows에서는 기본적으로 켜져 있으며 다른 곳에서는 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`로 활성화합니다. | `"powershell"` |
| `deniedMcpServers` | managed-settings.json에 설정된 경우 명시적으로 차단할 MCP 서버의 거부 목록. 관리형 서버를 포함한 모든 스코프에 적용됩니다. 거부 목록이 허용 목록보다 우선합니다. [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. | `[{ "serverName": "filesystem" }]` |
| `disableAgentView` | [백그라운드 에이전트 및 에이전트 뷰](/docs/en/agent-view)(`claude agents`, `--bg`, `/background` 및 주문형 수퍼바이저)를 끄려면 `true`로 설정하세요. 일반적으로 디바이스별 MDM 강제를 위해 [관리형 설정](/docs/en/permissions#managed-settings)에 설정합니다. `CLAUDE_CODE_DISABLE_AGENT_VIEW`를 `1`로 설정하는 것과 동일합니다. | `true` |
| `disableAllHooks` | 모든 [훅](/docs/en/hooks) 및 모든 사용자 지정 [상태 표시줄](/docs/en/statusline)을 비활성화합니다. | `true` |
| `disableArtifact` | 세션 출력을 claude.ai의 비공개 웹 페이지로 게시하는 [아티팩트](/docs/en/artifacts) 도구를 비활성화하려면 `true`로 설정하세요. `CLAUDE_CODE_DISABLE_ARTIFACT`를 `1`로 설정하는 것과 동일합니다. | `true` |
| `disableAutoMode` | [자동 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)가 활성화되지 않도록 `"disable"`로 설정합니다. `Shift+Tab` 사이클에서 `auto`를 제거하고 시작 시 `--permission-mode auto`를 거부합니다. 사용자가 오버라이드할 수 없는 [관리형 설정](/docs/en/permissions#managed-settings)에서 가장 유용합니다. | `"disable"` |
| `disableBrowserExternalNavigation` | (관리형 설정 전용) 데스크톱 앱의 [브라우저 창](/docs/en/desktop#browse-external-sites)에서 외부 탐색을 끄려면 `true`로 설정합니다. 사용자나 Claude 모두 외부 사이트로 이동할 수 없으며 localhost 개발 서버 미리보기는 영향을 받지 않습니다. 값은 JSON 불리언 `true`여야 하며 문자열 `"true"`는 무시됩니다. | `true` |
| `disableBundledSkills` | Claude Code에 포함된 [스킬](/docs/en/skills) 및 워크플로를 비활성화하려면 `true`로 설정하세요. 번들 스킬과 워크플로가 완전히 제거되는 반면 `/init`과 같은 내장 명령은 계속 입력할 수 있지만 모델로부터 숨겨집니다. `/doctor`는 내장 명령처럼 입력할 수 있습니다. 대신 [`DISABLE_DOCTOR_COMMAND`](/docs/en/env-vars)로 숨기세요. 플러그인, `.claude/skills/` 및 `.claude/commands/`의 스킬은 영향을 받지 않습니다. `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS`를 `1`로 설정하는 것과 동일합니다. | `true` |
| `disableClaudeAiConnectors` | {/* min-version: 2.1.182 */}[claude.ai MCP 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai)를 비활성화하여 자동으로 가져오거나 연결되지 않도록 합니다. 모든 설정 스코프에서 설정할 수 있습니다. 임의의 소스에서 `true`가 우선하므로 체크인된 프로젝트 `.claude/settings.json`이 리포지토리에서 클라우드 커넥터를 사용하지 않도록 선택할 수 있지만 프로젝트 수준의 `false`가 사용자 또는 정책 수준의 `true`를 오버라이드할 수는 없습니다. `--mcp-config`를 통해 명시적으로 전달된 서버는 영향을 받지 않습니다. 모든 커넥터 대신 개별 커넥터를 거부하려면 [`deniedMcpServers`](/docs/en/managed-mcp)를 사용하세요. Claude Code v2.1.182 이상이 필요합니다. | `true` |
| `disableDeepLinkRegistration` | 첫 프롬프트를 전송할 때 운영 체제에 `claude-cli://` 프로토콜 핸들러를 등록하지 않도록 `"disable"`로 설정합니다. [딥 링크](/docs/en/deep-links)를 사용하면 외부 도구가 미리 채워진 프롬프트로 Claude Code 세션을 열 수 있습니다. 프로토콜 핸들러 등록이 제한되거나 별도로 관리되는 환경에서 유용합니다. | `"disable"` |
| `disabledMcpjsonServers` | `.mcp.json` 파일에서 거부할 특정 MCP 서버 목록 | `["filesystem"]` |
| `disableRemoteControl` | {/* min-version: 2.1.128 */}[원격 제어(Remote Control)](/docs/en/remote-control) 비활성화: `claude remote-control`, `--remote-control` 플래그, 자동 시작 및 세션 내 토글을 차단합니다. 일반적으로 디바이스별 MDM 강제를 위해 [관리형 설정](/docs/en/permissions#managed-settings)에 설치하지만 모든 스코프에서 작동합니다. Claude Code v2.1.128 이상이 필요합니다. | `true` |
| `disableSideloadFlags` | {/* min-version: 2.1.193 */}(관리형 설정 전용) 시작 시 사용자가 단일 실행을 위해 [`strictKnownMarketplaces`](#strictknownmarketplaces)를 우회하기 위해 전달할 수 있는 `--plugin-dir`, `--plugin-url`, `--agents`, `--mcp-config` CLI 플래그를 거부합니다. 내부적으로 이러한 플래그로 CLI를 생성하는 모드(현재 데스크톱 앱의 [Cowork](/docs/en/desktop) 로컬 세션)도 거부합니다. 모든 서버가 프로세스 내 `type: "sdk"` 항목인 `--mcp-config`는 여전히 수락되므로 Agent SDK 및 VS Code 확장은 계속 작동합니다. `claude mcp add`, `.mcp.json` 또는 SDK `setMcpServers()`는 차단하지 않습니다. 서버별 MCP 제어를 위해 [`allowedMcpServers`](/docs/en/managed-mcp)와 결합하세요. Claude Code v2.1.193 이상이 필요합니다. | `true` |
| `disableSkillShellExecution` | 사용자, 프로젝트, 플러그인 또는 추가 디렉터리 소스의 [스킬](/en/skills) 및 사용자 지정 명령에서 `` !`...` `` 및 ` ```! ` 블록에 대한 인라인 셸 실행을 비활성화합니다. 명령이 실행되는 대신 `[shell command execution disabled by policy]`로 대체됩니다. 번들 및 관리형 스킬은 영향을 받지 않습니다. 사용자가 오버라이드할 수 없는 [관리형 설정](/en/permissions#managed-settings)에서 가장 유용합니다. | `true` |
| `disableWorkflows` | **기본값**: `false`. [동적 워크플로](/docs/en/workflows#turn-workflows-off) 및 번들 워크플로 명령을 비활성화합니다. `CLAUDE_CODE_DISABLE_WORKFLOWS`를 `1`로 설정하는 것과 동일합니다. | `true` |
| `editorMode` | **기본값**: `"normal"`. 프롬프트 입력의 키 바인딩 모드: `"normal"` 또는 `"vim"`. `/config`에서 **Editor mode**로 표시됩니다. | `"vim"` |
| `effortLevel` | 세션 전반에 걸쳐 [노력 수준(effort level)](/docs/en/model-config#adjust-effort-level)을 지속합니다. `"low"`, `"medium"`, `"high"` 또는 `"xhigh"`를 허용합니다. 이러한 값 중 하나로 `/effort`를 실행하면 자동으로 기록됩니다. `--effort` 및 [`CLAUDE_CODE_EFFORT_LEVEL`](/docs/en/env-vars)은 한 세션 동안 이를 오버라이드합니다. 지원되는 모델은 [노력 수준 조정](/docs/en/model-config#adjust-effort-level)을 참조하세요. | `"xhigh"` |
| `emojiCompletionEnabled` | {/* min-version: 2.1.217 */}**기본값**: `true`. 프롬프트 입력에 `:`와 숏코드를 입력할 때 이모지 제안을 표시하고 완성된 숏코드(예: `:heart:`)를 이모지로 대체합니다. 두 기능을 모두 비활성화하려면 `false`로 설정하세요. [이모지 숏코드](/docs/en/interactive-mode#emoji-shortcodes)를 참조하세요. Claude Code v2.1.217 이상이 필요합니다. | `false` |
| `enableAllProjectMcpServers` | 프로젝트 `.mcp.json` 파일에 정의된 모든 MCP 서버를 자동으로 승인합니다. {/* min-version: 2.1.196 */}v2.1.196부터 신뢰하지 않는 폴더의 `claude mcp list` 및 `claude mcp get`은 [리포지토리에 체크인되지 않은 설정 파일](/docs/en/mcp#managing-your-servers)에서만 이 키를 존중합니다. | `true` |
| `enableArtifact` | {/* min-version: 2.1.196 */}이 사용자에 대해 [아티팩트](/docs/en/artifacts) 도구를 활성화하거나 비활성화합니다. 설정되지 않은 경우 기본값은 계정의 기능 [지원 여부](/docs/en/artifacts#availability)를 따릅니다. `/config` 의 **Artifacts** 행에 이 키가 기록됩니다. 관리형 `disableArtifact` 및 조직의 [관리자 설정](/docs/en/artifacts#manage-artifacts-for-your-organization)이 우선하며, 리포지토리가 커밋할 수 있는 프로젝트 및 로컬 설정(`.claude/settings.json`, `.claude/settings.local.json`)에서는 키가 무시됩니다. Claude Code v2.1.196 이상이 필요합니다. | `true` |
| `enabledMcpjsonServers` | 승인할 `.mcp.json` 파일의 특정 MCP 서버 목록. {/* min-version: 2.1.196 */}v2.1.196부터 신뢰하지 않는 폴더의 `claude mcp list` 및 `claude mcp get`은 [리포지토리에 체크인되지 않은 설정 파일](/docs/en/mcp#managing-your-servers)에서만 이 키를 존중합니다. | `["memory", "github"]` |
| `enforceAvailableModels` | {/* min-version: 2.1.175 */}Default 모델로 `availableModels` 허용 목록을 확장합니다. 관리형 설정에서 `true`이고 `availableModels`가 비어 있지 않은 배열인 경우 Default 옵션은 사용할 수 있는 첫 번째 허용 목록 항목으로 대체되지만, Default가 해석될 모델([조직 기본 모델](/docs/en/model-config#organization-default-model)이 적용되는 경우 해당 모델, 그렇지 않으면 계정 유형 기본값)이 허용 목록에 없을 때만 적용됩니다. 허용 목록에 있는 기본값은 그대로 유지됩니다. `availableModels`가 설정되어 있지 않거나 비어 있으면 효과가 없습니다. [Default 모델에 대한 허용 목록 강제 적용](/docs/en/model-config#enforce-the-list-for-the-default-model)을 참조하세요. Claude Code v2.1.175 이상이 필요합니다. | `true` |
| `env` | 모든 세션 및 Claude Code가 생성하는 자식 프로세스에 적용되는 환경 변수. 변수를 `""`로 설정하면 셸 export가 빈 문자열로 오버라이드되며 Claude Code는 이를 제공업체 선택 시 설정되지 않은 것으로 다룹니다. 자식 프로세스는 여전히 빈 값을 상속합니다. 여기에 설정된 `NO_COLOR` 및 `FORCE_COLOR`는 자식 프로세스에만 도달합니다. Claude Code 고유 인터페이스 색상을 변경하려면 `claude`를 실행하기 전에 셸에서 설정하세요. {/* min-version: 2.1.195 */}v2.1.195부터 Claude Code의 호스팅 환경이 설정하는 식별 변수(예: `CLAUDE_CODE_REMOTE`, `CLAUDE_CODE_ACCOUNT_UUID`)는 여기에 설정하더라도 무시됩니다. | `{"FOO": "bar"}` |
| `fallbackModel` | 주 모델이 과부하되거나 사용할 수 없을 때 순서대로 시도할 대체 모델. Claude Code는 차례의 남은 기간 동안 체인의 다음 사용 가능한 모델로 전환하고 알림을 표시합니다. `"default"`는 기본 모델로 확장됩니다. 체인은 최대 3개의 모델로 제한되며 추가 항목은 무시됩니다. 대부분의 배열 설정과 달리 이 키는 설정 파일 간에 병합되지 않습니다. 이 키를 정의하는 가장 높은 우선순위 파일이 전체 체인을 제공합니다. [`--fallback-model`](/docs/en/cli-reference#cli-flags) 플래그는 한 세션 동안 이를 오버라이드합니다. [대체 모델 체인](/docs/en/model-config#fallback-model-chains)을 참조하세요. | `["claude-sonnet-5", "claude-haiku-4-5"]` |
| `fastMode` | [빠른 모드(fast mode)](/docs/en/fast-mode)를 사용할 수 있는 세션에 대해 빠른 모드를 켭니다. `/fast`로 토글하면 사용자 설정에 `true`가 기록되고 빠른 모드를 끌 때 키가 제거됩니다. | `true` |

---

## 훅 구성 (Hook configuration)

### 관리형 훅 제한

관리자는 관리형 설정을 사용하여 실행할 수 있는 훅을 제한할 수 있습니다:

**`allowManagedHooksOnly`가 `true`일 때의 동작:**

* 관리형 훅 및 SDK 훅이 로드됨
* 관리형 설정 `enabledPlugins`에서 강제 활성화된 플러그인의 훅이 로드됨. 이를 통해 관리자는 다른 모든 항목을 차단하면서 검증된 훅을 조직 마켓플레이스를 통해 배포할 수 있습니다. 신뢰는 전체 `plugin@marketplace` ID로 부여되므로 다른 마켓플레이스에서 동일한 이름을 가진 플러그인은 차단된 상태로 유지됩니다.
* 사용자 훅, 프로젝트 훅 및 기타 모든 플러그인 훅은 차단됨

**HTTP 훅 URL 제한:**

HTTP 훅이 타깃으로 지정할 수 있는 URL을 제한합니다. 일치를 위해 와일드카드로 `*`를 지원합니다. 배열이 정의되면 일치하지 않는 URL을 타깃으로 하는 HTTP 훅은 메시지 없이 차단됩니다. 호스트 이름 일치는 대소문자를 구분하지 않으며 후미 FQDN 점을 무시하여 DNS 시맨틱과 일치합니다.

```json theme={null}
{
  "allowedHttpHookUrls": ["https://hooks.example.com/*", "http://localhost:*"]
}
```

**HTTP 훅 환경 변수 제한:**

HTTP 훅이 헤더 값에 보간할 수 있는 환경 변수 이름을 제한합니다. 각 훅의 유효한 `allowedEnvVars`는 고유 목록과 이 설정의 교집합입니다.

```json theme={null}
{
  "httpHookAllowedEnvVars": ["MY_TOKEN", "HOOK_SECRET"]
}
```

### 정책 헬퍼를 사용하여 관리형 설정 계산하기

`policyHelper` 설정은 시작 시 관리형 설정을 계산하는 실행 파일을 가리키므로 관리자는 정적 파일 대신 디바이스 상태, 식별자 또는 원격 서비스에서 정책을 파생할 수 있습니다. MDM 또는 시스템 `managed-settings.json` 파일에서 이를 구성하세요. Claude Code는 사용자 설정, 프로젝트 설정, HKCU 레지스트리 하이브 및 [서버 관리형 설정](/docs/en/server-managed-settings)을 포함하여 다른 모든 스코프에 `policyHelper`가 나타나면 무시합니다.

이 설정은 다음 키를 수락합니다:

| 키 | 타입 | 설명 |
| :--- | :--- | :--- |
| `path` | string | 헬퍼 실행 파일의 절대 경로 |
| `timeoutMs` | number | 실행을 실패로 처리하기 전에 헬퍼를 기다리는 시간(밀리초) |
| `refreshIntervalMs` | number | 백그라운드에서 헬퍼를 다시 실행하는 주기. 새로 고침을 비활성화하려면 `0`으로 설정하거나 최소 `60000`으로 설정 |

헬퍼는 stdout에 JSON 봉투를 씁니다. 단순 설정 객체는 `managedSettings`가 정의되지 않은 것으로 파싱되어 아무것도 적용되지 않으므로 설정을 최상위 수준이 아닌 `managedSettings` 키 아래에 두세요:

```json theme={null}
{
  "managedSettings": {
    "permissions": { "deny": ["Read(//etc/secrets/**)"] }
  },
  "claudeMd": "# Organization context\n...",
  "appendSystemPrompt": "Always cite the internal style guide."
}
```

헬퍼가 `managedSettings`를 내보내면 해당 객체는 원격, MDM 및 파일 기반 소스보다 우선하여 실행 시의 유일한 관리형 설정 소스가 됩니다. 시작 시 헬퍼가 0이 아닌 상태로 종료되면 Claude Code는 오류를 출력하고 시작을 거부하므로 정전 내구성이 필요한 헬퍼는 자체 캐시에서 제공하고 `0`으로 종료되어야 합니다.

### 설정 우선순위

설정은 우선순위 순서대로 적용됩니다. 가장 높은 순위부터 낮은 순위까지:

1. **관리형 설정** ([서버 관리형](/docs/en/server-managed-settings), [MDM/OS 수준 정책](#구성-스코프-configuration-scopes), 또는 [관리형 설정](/docs/en/settings#settings-files))
   * 서버 전달, MDM 구성 프로필, 레지스트리 정책 또는 관리형 설정 파일을 통해 IT 부서에서 배포한 정책
   * 명령줄 인수를 포함하여 다른 어떠한 수준으로도 오버라이드할 수 없음
   * 관리형 계층 내에서는 하나의 소스만 사용되며 다른 소스는 병합되지 않고 무시됨. 우선순위(높은 순):
     * [`policyHelper`](#정책-헬퍼를-사용하여-관리형-설정-계산하기) 출력: 구성된 경우 이것이 사용되는 유일한 관리형 소스임
     * 원격 (claude.ai [서버 관리형](/docs/en/server-managed-settings) 또는 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 전달)
     * MDM/OS 수준 정책
     * 파일 기반 (`managed-settings.d/*.json` 및 `managed-settings.json`, 함께 병합됨)
     * HKCU 레지스트리 (Windows 전용)
   * 몇 가지 키는 승리한 소스뿐만 아니라 관리자가 제어하는 임의의 관리형 소스에서 설정된 경우 존중됩니다. 사용자 작성 가능한 HKCU 레지스트리 소스는 제외되며, [`policyHelper`](#정책-헬퍼를-사용하여-관리형-설정-계산하기)가 구성되면 그 출력이 이러한 검사가 읽는 유일한 소스가 됩니다. 예외 키는 다음과 같습니다:
     * 샌드박스 잠금 키 `sandbox.network.allowManagedDomainsOnly` 및 `sandbox.filesystem.allowManagedReadPathsOnly`, 그리고 관련된 허용 목록
     * `allowAllClaudeAiMcps`
     * 샌드박스 바이너리 경로 `sandbox.bwrapPath` 및 `sandbox.socatPath`
     * [`forceRemoteSettingsRefresh`](/docs/en/server-managed-settings)
   * Claude Desktop과 같은 임베딩 호스트는 SDK `managedSettings` 옵션을 통해 정책을 제공할 수 있습니다. 기본적으로 관리자가 배포한 관리형 소스(서버 관리형 설정, MDM 또는 OS 수준 정책, 또는 관리형 설정 파일)가 존재하면 이는 무시됩니다. 사용자 작성 가능한 HKCU 레지스트리 대안은 관리자 배포 소스로 카운트되지 않습니다. 관리자는 가장 높은 우선순위의 관리형 소스에서 [`parentSettingsBehavior`](#사용-가능한-설정)를 `"merge"`로 설정하여 참여할 수 있으며, 해당 소스의 값만 읽힙니다. 임베더의 값은 제한 전용 필터를 통과하지만 필터가 엄격하게 조이기만 하는 것은 아닙니다: `allowManaged*Only` 잠금이 설정되어 있지 않은 한 권한 허용 규칙 및 샌드박스 허용 목록과 같은 허용 방향 설정이 여전히 적용됩니다. 잠금에 대해서는 [부모 설정 제한](/docs/en/claude-apps-gateway#restrict-parent-settings)을 참조하세요. [`policyHelper`](#정책-헬퍼를-사용하여-관리형-설정-계산하기)가 구성된 동안에는 이 키와 상관없이 부모 설정이 병합되지 않습니다.
   * 관리자가 제어하는 관리형 소스가 `allowManagedPermissionRulesOnly`를 설정하면 더 높은 우선순위의 소스가 키를 설정하지 않은 채로 두더라도 Claude Code가 읽을 때 [부모가 제공한](/docs/en/claude-apps-gateway#restrict-parent-settings) 권한 허용 규칙 및 `additionalDirectories`를 삭제합니다. 개발자 자신의 권한 규칙에 미치는 효과는 가장 높은 우선순위 소스에서만 옵니다.
   * `forceLoginOrgUUID` 및 `allowedMcpServers`에 대한 부모 설정 갭 채우기 검사도 모든 관리자 제어 관리형 소스를 읽습니다. 어느 소스에든 존재하는 값이 부모 제공 값을 차단하지만 적용되는 값은 여전히 가장 높은 우선순위 소스에서 옵니다.

2. **명령줄 인수**
   * 특정 세션에 대한 임시 오버라이드. `--settings <file-or-json>`을 통해 전달된 JSON은 다른 레이어와 동일한 규칙을 사용하여 파일 기반 설정과 병합됩니다. 여기에 설정된 키는 로컬, 프로젝트 또는 사용자 설정의 동일한 키를 오버라이드하고 키를 생략하면 하위 레이어 값이 그대로 남습니다.

3. **로컬 프로젝트 설정** (`.claude/settings.local.json`)
   * 개인의 프로젝트 특정 설정

4. **공유 프로젝트 설정** (`.claude/settings.json`)
   * 소스 제어의 팀 공유 프로젝트 설정

5. **사용자 설정** (`~/.claude/settings.json`)
   * 개인의 전역 설정

이 계층 구조는 조직 정책이 항상 적용되도록 보장하면서 팀과 개인이 경험을 맞춤 설정할 수 있도록 허용합니다. CLI, [VS Code 확장 프로그램](/docs/en/vs-code) 또는 [JetBrains IDE](/docs/en/jetbrains)에서 Claude Code를 실행하든 관계없이 동일한 우선순위가 적용됩니다.

예를 들어 사용자 설정에서 `permissions.defaultMode`를 `acceptEdits`로 설정하고 프로젝트의 공유 설정에서 `default`로 설정한 경우 프로젝트 값이 적용됩니다. 아래 예시는 권한 규칙과 같은 배열 값 설정이 결합되는 방식을 다룹니다.

<Note>
  **배열 설정은 스코프 전체에서 병합됩니다.** 동일한 배열 값 설정(예: `sandbox.filesystem.allowWrite` 또는 `permissions.allow`)이 여러 스코프에 나타나면 배열이 **연결 및 중복 제거**되며 대체되지 않습니다. 즉, 상위 우선순위 스코프에 설정된 항목을 오버라이드하지 않고 하위 우선순위 스코프가 항목을 추가할 수 있으며 그 반대도 마찬가지입니다. 예를 들어 관리형 설정이 `allowWrite`를 `["/opt/company-tools"]`로 설정하고 사용자가 `["~/.kube"]`를 추가하면 두 경로가 최종 구성에 포함됩니다.

  두 가지 배열 설정은 이러한 방식으로 병합되지 않습니다:

  * [`fallbackModel`](#사용-가능한-설정): 위치가 의미를 갖는 순서 지정 체인이며, 이를 정의하는 가장 높은 우선순위 파일이 전체 값을 제공합니다.
  * [`availableModels`](#사용-가능한-설정): {/* min-version: 2.1.175 */}[가장 높은 우선순위의 관리형 소스](/docs/en/server-managed-settings#settings-precedence)가 이를 정의하면 해당 목록이 있는 그대로 적용되며 사용자, 프로젝트 및 로컬 항목이 이를 확장할 수 없습니다. 비관리형 스코프에서는 배열이 평소대로 병합됩니다. [병합 동작](/docs/en/model-config#merge-behavior)을 참조하세요.
</Note>

### 활성 설정 확인하기

Claude Code 내부에서 `/status`를 실행하여 활성화된 설정 소스를 확인하세요. 메뉴 내부의 **Status** 탭에는 `User settings` 또는 `Project local settings`와 같이 현재 세션에 로드된 각 레이어를 나열하는 `Setting sources` 라인이 포함되어 있습니다. [관리형 설정](/docs/en/admin-setup#decide-how-settings-reach-devices)이 적용 중인 경우 괄호 안에 전달 채널이 표시됩니다(예: `Enterprise managed settings (remote)`, `(plist)`, `(HKLM)`, `(HKCU)`, 또는 `(file)`). `remote` 채널은 claude.ai 서버 관리형 설정과 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 전달 정책을 모두 처리합니다. 레이어는 해당 소스가 적어도 하나의 키와 함께 로드되었을 때만 목록에 나타나므로 빈 목록은 설정 소스를 찾지 못했음을 의미합니다.

`Setting sources` 라인은 어떤 소스가 읽히고 있는지 확인해 줍니다. 각 개별 키를 어떤 레이어가 제공했는지는 보여주지 않습니다. 동일한 대화 상자의 **Config** 탭은 `settings.json` 내용 뷰가 아니라 테마 및 자세한 출력과 같은 고정된 토글 세트에 대한 편집기입니다.

사용자, 프로젝트 또는 로컬 설정 파일에 유효하지 않은 JSON이나 유효성 검사에 실패한 값과 같은 오류가 포함되어 있으면 대화형 세션 시작 시 **Settings Error** 대화 상자가 표시됩니다. 이 대화 상자를 사용하면 Claude의 도움을 받아 파일을 수정하거나, 종료하거나, 손상된 설정 없이 계속 진행할 수 있습니다.

계속 진행한 후 `/status`는 영향을 받은 파일을 나열합니다. 각 오류의 세부 정보를 보려면 `claude doctor`를 실행하세요.

유효성 검사에 실패한 관리형 설정 항목은 [관리형 설정의 유효하지 않은 항목](#관리형-설정의-유효하지-않은-항목)에 설명된 관대한 흐름을 따릅니다. 파일 전체가 거부되지 않으며 나머지 유효한 정책은 계속 적용됩니다.

### 구성 시스템에 대한 핵심 포인트

* **메모리 파일 (`CLAUDE.md`)**: Claude가 시작 시 로드하는 지침 및 컨텍스트 포함
* **설정 파일 (JSON)**: 권한, 환경 변수 및 도구 동작 구성
* **스킬**: `/skill-name`으로 호출하거나 Claude가 자동으로 로드할 수 있는 사용자 지정 프롬프트
* **MCP 서버**: 추가 도구 및 통합으로 Claude Code 확장
* **우선순위**: 상위 수준 구성(Managed)이 하위 수준 구성(User/Project)을 오버라이드함
* **상속**: 설정은 스코프 간에 병합됨. 상위 우선순위 스코프의 스칼라 값이 오버라이드되고 배열은 연결됨([배열 병합 Note](#설정-우선순위)에 설명된 두 가지 예외 제외)

### 시스템 프롬프트

Claude Code의 내부 시스템 프롬프트는 게시되지 않습니다. 사용자 지정 지침을 추가하려면 `CLAUDE.md` 파일이나 `--append-system-prompt` 플래그를 사용하세요.

### 민감한 파일 제외하기

API 키, 시크릿 및 환경 파일과 같은 민감한 정보가 포함된 파일에 Claude Code가 접근하지 못하도록 하려면 `.claude/settings.json` 파일의 `permissions.deny` 설정을 사용하세요:

```json theme={null}
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./build)"
    ]
  }
}
```

이는 더 이상 사용되지 않는 `ignorePatterns` 구성을 대체합니다. 이러한 패턴과 일치하는 파일은 파일 탐색 및 검색 결과에서 제외되며 이러한 파일에 대한 읽기 작업이 거부됩니다.

---

## 서브에이전트 구성 (Subagent configuration)

Claude Code는 사용자 및 프로젝트 수준 모두에서 구성할 수 있는 사용자 지정 AI 서브에이전트를 지원합니다. 이러한 서브에이전트는 YAML 프론트매터가 포함된 Markdown 파일로 저장됩니다:

* **사용자 서브에이전트**: `~/.claude/agents/`, 모든 프로젝트에서 사용 가능
* **프로젝트 서브에이전트**: `.claude/agents/`, 프로젝트에 특화되어 팀과 공유 가능

서브에이전트 파일은 사용자 지정 프롬프트 및 도구 권한을 갖춘 특화된 AI 어시스턴트를 정의합니다. 서브에이전트 생성 및 사용에 대해 자세히 알아보려면 [서브에이전트 문서](/docs/en/sub-agents)를 참조하세요.

---

## 플러그인 구성 (Plugin configuration)

Claude Code는 스킬, 에이전트, 훅 및 MCP 서버로 기능을 확장할 수 있는 플러그인 시스템을 지원합니다. 플러그인은 마켓플레이스를 통해 배포되며 사용자 및 리포지토리 수준 모두에서 구성할 수 있습니다.

### 플러그인 설정

`settings.json`에서의 플러그인 관련 설정:

```json theme={null}
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "analyzer@security-plugins": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  }
}
```

#### `enabledPlugins`

활성화할 플러그인을 제어합니다. 형식: `"plugin-name@marketplace-name": true/false`. 모든 스코프에서 항목이 없는 플러그인은 기본적으로 [`defaultEnabled`](/docs/en/plugins-reference#default-enablement) 값으로 대체됩니다.

**스코프**:

* **사용자 설정** (`~/.claude/settings.json`): 개인 플러그인 기본 설정
* **프로젝트 설정** (`.claude/settings.json`): 팀과 공유되는 프로젝트 전용 플러그인
* **로컬 설정** (`.claude/settings.local.json`): 머신별 오버라이드, Claude Code가 생성 시 gitignore 처리함
* **관리형 설정** (`managed-settings.json`): 모든 스코프에서 설치를 차단하고 마켓플레이스에서 플러그인을 숨기는 조직 전체 정책 오버라이드

<Note>
  프로젝트 설정이 사용자 설정보다 우선하므로 `~/.claude/settings.json`에서 플러그인을 `false`로 설정해도 프로젝트의 `.claude/settings.json`이 활성화한 플러그인은 비활성화되지 않습니다. 머신에서 프로젝트 활성화 플러그인을 거부하려면 대신 `.claude/settings.local.json`에서 `false`로 설정하세요.

  관리형 설정으로 강제 활성화된 플러그인은 관리형 설정이 로컬 설정을 오버라이드하므로 이 방식으로 비활성화할 수 없습니다.

  프로젝트의 `.claude/settings.json`에서 GitHub 리포지토리나 npm 패키지와 같은 외부 소스의 플러그인을 활성화하더라도 다른 사람에게 자동으로 설치되지는 않습니다. Claude Code v2.1.195부터 플러그인을 로드하는 모든 경로는 플러그인이 실행되기 전에 각 사용자에게 [플러그인 설치 및 신뢰](/docs/en/discover-plugins#configure-team-marketplaces)를 요청합니다.
</Note>

**예시**:

```json theme={null}
{
  "enabledPlugins": {
    "code-formatter@team-tools": true,
    "deployment-tools@team-tools": true,
    "experimental-features@personal": false
  }
}
```

#### `pluginConfigs`

플러그인 ID를 키로 하여 플러그인의 [`userConfig`](/docs/en/plugins-reference#user-configuration) 프롬프트가 수집하는 민감하지 않은 옵션 값을 저장합니다. 플러그인의 구성 대화 상자를 채우면 Claude Code가 이 키를 사용자 설정에 수동 편집 없이 기록합니다. 민감한 옵션은 대신 macOS Keychain에 저장되거나 지원되는 키체인이 없는 플랫폼의 경우 `~/.claude/.credentials.json`에 저장됩니다.

다음 예시는 `acme-tools` 마켓플레이스에서 설치된 플러그인에 대한 옵션 하나를 저장합니다:

```json theme={null}
{
  "pluginConfigs": {
    "deployer@acme-tools": {
      "options": {
        "api_endpoint": "https://api.example.com"
      }
    }
  }
}
```

`pluginConfigs`는 사용자 설정, `--settings` 플래그 및 관리형 설정에서만 읽습니다. 프로젝트의 `.claude/settings.json` 또는 `.claude/settings.local.json` 항목은 무시됩니다. 이 값들이 플러그인 훅, MCP 및 LSP 구성에 대입되며 복제된 리포지토리가 이들을 제공해서는 안 되기 때문입니다. v2.1.207 이전에는 프로젝트 및 로컬 설정도 읽혔습니다.

#### `extraKnownMarketplaces`

리포지토리에서 사용할 수 있어야 하는 추가 마켓플레이스를 정의합니다. 일반적으로 팀 구성원이 필요한 플러그인 소스에 접근할 수 있도록 리포지토리 수준 설정에서 사용됩니다.

**리포지토리에 `extraKnownMarketplaces`가 포함되어 있을 때**:

1. 팀 구성원이 폴더를 신뢰할 때 마켓플레이스 설치를 요청받음
2. 그 다음 해당 마켓플레이스에서 플러그인을 설치하도록 요청받음
3. 사용자는 원하지 않는 마켓플레이스나 플러그인을 건너뛸 수 있음 (사용자 설정에 저장됨)
4. 설치 시 신뢰 경계를 존중하며 명시적 동의 필요

**예시**:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "security-plugins": {
      "source": {
        "source": "git",
        "url": "https://git.example.com/security/plugins.git"
      }
    }
  }
}
```

**마켓플레이스 소스 유형**:

* `github`: GitHub 리포지토리 (`repo` 사용)
* `git`: 임의의 Git URL (`url` 사용)
* `directory`: 로컬 파일 시스템 경로 (`path` 사용, 개발 전용)
* `hostPattern`: 마켓플레이스 호스트와 일치시킬 정규식 패턴 (`hostPattern` 사용)
* `settings`: 별도의 호스팅 리포지토리 없이 settings.json에 직접 선언된 인라인 마켓플레이스 (`name` 및 `plugins` 사용)

`git` 소스 유형은 자체 호스팅 GitLab 및 Bitbucket을 포함한 모든 Git 호스팅 서비스에서 작동합니다. Claude Code는 해당 머신에서 `git clone`이 사용할 동일한 인증(구성된 자격 증명 헬퍼 또는 SSH 키)으로 리포지토리를 복제합니다. `GITHUB_TOKEN`과 같은 제공업체 토큰은 이를 읽는 자격 증명 헬퍼를 통해서만 적용됩니다. 설정 세부 정보는 [비공개 리포지토리](/docs/en/plugin-marketplaces#private-repositories)를 참조하세요.

`github` 및 `git` 소스의 경우, Claude Code가 마켓플레이스 리포지토리를 클론하거나 업데이트할 때 Git LFS 다운로드를 건너뛰려면 `source` 객체 내부(`repo` 또는 `url`과 함께)에 `"skipLfs": true`를 설정하세요. LFS 포인터 파일은 콘텐츠를 다운로드하는 대신 포인터로 남아 있습니다. 리포지토리에 플러그인 콘텐츠와 무관한 대용량 LFS 객체가 포함되어 있을 때 사용하세요. {/* min-version: 2.1.153 */}Claude Code v2.1.153 이상이 필요합니다.

각 마켓플레이스 항목은 선택적 `autoUpdate` 불리언도 수락합니다. Claude Code가 시작 후 백그라운드에서 해당 마켓플레이스를 새로 고치고 설치된 플러그인을 업데이트하도록 하려면 `source`와 함께 `"autoUpdate": true`를 설정하세요. 생략하면 공식 Anthropic 마켓플레이스는 기본적으로 `true`로 설정되고 다른 모든 마켓플레이스는 기본적으로 `false`로 설정됩니다. [자동 업데이트 구성](/docs/en/discover-plugins#configure-auto-updates)을 참조하세요.

호스팅된 마켓플레이스 리포지토리를 설정하지 않고 소규모 플러그인 세트를 인라인으로 선언하려면 `source: 'settings'`를 사용하세요. 여기에 나열된 플러그인은 GitHub 또는 npm과 같은 외부 소스를 참조해야 합니다. 여전히 `enabledPlugins`에서 각 플러그인을 별도로 활성화해야 합니다.

```json theme={null}
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "settings",
        "name": "team-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": {
              "source": "github",
              "repo": "acme-corp/code-formatter"
            }
          }
        ]
      }
    }
  }
}
```

#### `strictKnownMarketplaces`

**관리형 설정 전용**: 사용자가 추가하고 플러그인을 설치할 수 있는 플러그인 마켓플레이스를 제어합니다. 이 설정은 [관리형 설정](/docs/en/settings#settings-files)에서만 구성할 수 있으며 관리자에게 마켓플레이스 소스에 대한 엄격한 제어를 제공합니다.

**관리형 설정 파일 위치**:

* **macOS**: `/Library/Application Support/ClaudeCode/managed-settings.json`
* **Linux 및 WSL**: `/etc/claude-code/managed-settings.json`
* **Windows**: `C:\Program Files\ClaudeCode\managed-settings.json`

**주요 특징**:

* 관리형 설정(`managed-settings.json`)에서만 사용할 수 있음
* 사용자 또는 프로젝트 설정으로 오버라이드할 수 없음 (가장 높은 우선순위)
* 네트워크 및 파일 시스템 작업 전에 적용되므로 차단된 소스는 절대 실행되지 않음
* 정규식 일치를 사용하는 `hostPattern` 및 `pathPattern`을 제외하고 소스 사양에 대해 정확한 일치(git 소스의 경우 `ref`, `path` 포함)를 사용함

**허용 목록 동작**:

* `undefined` (기본값): 제한 없음, 사용자가 모든 마켓플레이스를 추가할 수 있음
* 빈 배열 `[]`: 완전한 봉쇄(lockdown), 사용자가 새로운 마켓플레이스를 추가할 수 없음
* 소스 목록: 사용자는 정확히 일치하는 마켓플레이스만 추가할 수 있음

**지원되는 모든 소스 유형**:

허용 목록은 여러 마켓플레이스 소스 유형을 지원합니다. 대부분의 소스는 정확한 일치를 사용하는 반면, `hostPattern` 및 `pathPattern`은 각각 마켓플레이스 호스트 및 파일 시스템 경로에 대해 정규식 일치를 사용합니다.

1. **GitHub 리포지토리**:

```json theme={null}
{ "source": "github", "repo": "acme-corp/approved-plugins" }
{ "source": "github", "repo": "acme-corp/security-tools", "ref": "v2.0" }
{ "source": "github", "repo": "acme-corp/plugins", "ref": "main", "path": "marketplace" }
```

필드: `repo` (필수), `ref` (선택 사항: 브랜치 또는 태그), `path` (선택 사항: 하위 디렉터리)

2. **Git 리포지토리**:

```json theme={null}
{ "source": "git", "url": "https://gitlab.example.com/tools/plugins.git" }
{ "source": "git", "url": "https://bitbucket.org/acme-corp/plugins.git", "ref": "production" }
{ "source": "git", "url": "ssh://git@git.example.com/plugins.git", "ref": "v3.1", "path": "approved" }
```

필드: `url` (필수), `ref` (선택 사항: 브랜치 또는 태그), `path` (선택 사항: 하위 디렉터리)

3. **URL 기반 마켓플레이스**:

```json theme={null}
{ "source": "url", "url": "https://plugins.example.com/marketplace.json" }
{ "source": "url", "url": "https://cdn.example.com/marketplace.json", "headers": { "Authorization": "Bearer ${TOKEN}" } }
```

필드: `url` (필수), `headers` (선택 사항: 인증된 접근을 위한 HTTP 헤더)

<Note>
  URL 기반 마켓플레이스는 `marketplace.json` 파일만 다운로드하며 서버에서 플러그인 파일을 다운로드하지 않습니다. URL 기반 마켓플레이스의 플러그인은 상대 경로 대신 외부 소스(GitHub, npm 또는 Git URL)를 사용해야 합니다. 상대 경로가 포함된 플러그인의 경우 대신 Git 기반 마켓플레이스를 사용하세요. 자세한 내용은 [문제 해결](/docs/en/plugin-marketplaces#plugins-with-relative-paths-fail-in-url-based-marketplaces)을 참조하세요.
</Note>

4. **NPM 패키지**:

```json theme={null}
{ "source": "npm", "package": "@acme-corp/claude-plugins" }
{ "source": "npm", "package": "@acme-corp/approved-marketplace" }
```

필드: `package` (필수, 스코프 지정된 패키지 지원)

5. **파일 경로**:

```json theme={null}
{ "source": "file", "path": "/usr/local/share/claude/acme-marketplace.json" }
{ "source": "file", "path": "/opt/acme-corp/plugins/marketplace.json" }
```

필드: `path` (필수: marketplace.json 파일의 절대 경로)

6. **디렉터리 경로**:

```json theme={null}
{ "source": "directory", "path": "/usr/local/share/claude/acme-plugins" }
{ "source": "directory", "path": "/opt/acme-corp/approved-marketplaces" }
```

필드: `path` (필수: `.claude-plugin/marketplace.json`을 포함하는 디렉터리의 절대 경로)

7. **호스트 패턴 일치**:

```json theme={null}
{ "source": "hostPattern", "hostPattern": "^github\\.example\\.com$" }
{ "source": "hostPattern", "hostPattern": "^gitlab\\.internal\\.example\\.com$" }
```

필드: `hostPattern` (필수: 마켓플레이스 호스트와 일치시킬 정규식 패턴)

각 리포지토리를 개별적으로 나열하지 않고 특정 호스트의 모든 마켓플레이스를 허용하려는 경우 호스트 패턴 일치를 사용하세요. 이는 개발자가 자체 마켓플레이스를 생성하는 사내 GitHub Enterprise 또는 GitLab 서버가 있는 조직에서 유용합니다.

소스 유형별 호스트 추출:

* `github`: 항상 `github.com`과 일치
* `git`: URL에서 호스트 이름을 추출함 (HTTPS 및 SSH 형식 지원)
* `url`: URL에서 호스트 이름을 추출함
* `npm`, `file`, `directory`: 호스트 패턴 일치가 지원되지 않음

8. **경로 패턴 일치**:

```json theme={null}
{ "source": "pathPattern", "pathPattern": "^/opt/approved/" }
{ "source": "pathPattern", "pathPattern": ".*" }
```

필드: `pathPattern` (필수: `file` 및 `directory` 소스의 `path` 필드와 일치시킬 정규식 패턴)

네트워크 소스에 대한 `hostPattern` 제한과 함께 파일 시스템 기반 마켓플레이스를 허용하려면 경로 패턴 일치를 사용하세요. 모든 로컬 경로를 허용하려면 `".*"`를 설정하고 특정 디렉터리로 제한하려면 더 좁은 패턴을 설정하세요.

**구성 예시**:

예시: 특정 마켓플레이스만 허용:

```json theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "github",
      "repo": "acme-corp/security-tools",
      "ref": "v2.0"
    },
    {
      "source": "url",
      "url": "https://plugins.example.com/marketplace.json"
    },
    {
      "source": "npm",
      "package": "@acme-corp/compliance-plugins"
    }
  ]
}
```

예시: 모든 마켓플레이스 추가 비활성화:

```json theme={null}
{
  "strictKnownMarketplaces": []
}
```

예시: 사내 Git 서버의 모든 마켓플레이스 허용:

```json theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "hostPattern",
      "hostPattern": "^github\\.example\\.com$"
    }
  ]
}
```

**정확한 일치 요구 사항**:

사용자의 추가가 허용되려면 마켓플레이스 소스가 정확히 일치해야 합니다. Git 기반 소스(`github` 및 `git`)의 경우 모든 선택적 필드가 포함됩니다:

* `repo` 또는 `url`이 정확히 일치해야 함
* `ref` 필드가 정확히 일치해야 함 (또는 둘 다 정의되지 않음)
* `path` 필드가 정확히 일치해야 함 (또는 둘 다 정의되지 않음)

일치하지 않는 소스의 예:

```json theme={null}
// 이들은 서로 다른 소스입니다:
{ "source": "github", "repo": "acme-corp/plugins" }
{ "source": "github", "repo": "acme-corp/plugins", "ref": "main" }

// 이들 역시 서로 다릅니다:
{ "source": "github", "repo": "acme-corp/plugins", "path": "marketplace" }
{ "source": "github", "repo": "acme-corp/plugins" }
```

**`extraKnownMarketplaces`와의 비교**:

| 측면 | `strictKnownMarketplaces` | `extraKnownMarketplaces` |
| :--- | :--- | :--- |
| **목적** | 조직 정책 강제 | 팀 편의성 |
| **설정 파일** | `managed-settings.json` 전용 | 모든 설정 파일 |
| **동작** | 허용 목록에 없는 추가 차단 | 누락된 마켓플레이스 자동 설치 |
| **적용 시점** | 네트워크/파일 시스템 작업 전 | 사용자 신뢰 프롬프트 후 |
| **오버라이드 가능 여부** | 아니요 (가장 높은 우선순위) | 예 (더 높은 우선순위 설정에 의해) |
| **소스 형식** | 직접 소스 객체 | 중첩 소스가 있는 이름 지정된 마켓플레이스 |
| **사용 사례** | 컴플라이언스, 보안 제한 | 온보딩, 표준화 |

**형식 차이점**:

`strictKnownMarketplaces`는 직접 소스 객체를 사용합니다:

```json theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/plugins" }
  ]
}
```

`extraKnownMarketplaces`는 이름 지정된 마켓플레이스가 필요합니다:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "github", "repo": "acme-corp/plugins" }
    }
  }
}
```

**둘 다 함께 사용하기**:

`strictKnownMarketplaces`는 정책 게이트입니다. 사용자가 추가할 수 있는 항목을 제어하지만 마켓플레이스를 등록하지는 않습니다. 모든 사용자에 대해 마켓플레이스를 제한함과 동시에 사전 등록하려면 `managed-settings.json`에 둘 다 설정하세요:

```json theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/plugins" }
  ],
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "github", "repo": "acme-corp/plugins" }
    }
  }
}
```

`strictKnownMarketplaces`만 설정되어 있으면 사용자가 `/plugin marketplace add`를 통해 허용된 마켓플레이스를 수동으로 추가할 수는 있지만 자동으로 사용 가능해지지는 않습니다.

**중요 참고 사항**:

* 네트워크 요청이나 파일 시스템 작업 전에 제한이 확인됩니다.
* 차단되면 사용자에게 관리에 의해 소스가 차단되었음을 알리는 명확한 오류 메시지가 표시됩니다.
* 제한은 마켓플레이스 추가 및 플러그인 설치, 업데이트, 새로 고침, 자동 업데이트 시 적용됩니다. 정책이 설정되기 전에 추가된 마켓플레이스는 해당 소스가 더 이상 허용 목록과 일치하지 않으면 플러그인을 설치하거나 업데이트하는 데 사용할 수 없습니다.
* 관리형 설정은 가장 높은 우선순위를 가지며 오버라이드할 수 없습니다.

사용자용 설명 문서는 [관리형 마켓플레이스 제한 사항](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)을 참조하세요.

#### `strictPluginOnlyCustomization`

**관리형 설정 전용**: 사용자 및 프로젝트 소스에서 스킬, 에이전트, 훅 및 MCP 서버를 차단하여 플러그인 또는 관리형 설정에서만 공급될 수 있도록 합니다. `strictKnownMarketplaces`와 결합하여 전체 맞춤 구성 공급망을 제어하세요: 마켓플레이스 허용 목록은 사용자가 설치할 수 있는 플러그인을 제어하고 이 설정은 플러그인이나 관리형 설정에서 오지 않은 모든 것을 차단합니다.

값은 4개 영역 전체를 잠그는 `true`이거나 잠글 영역의 이름을 지정하는 배열입니다:

```json theme={null}
{
  "strictPluginOnlyCustomization": ["skills", "hooks"]
}
```

잠긴 각 영역에 대해 Claude Code는 사용자 및 프로젝트 수준 소스를 건너뛰고 플러그인이 제공하는 소스 및 관리형 소스만 로드합니다:

| 영역 | 잠겼을 때 차단되는 항목 | 여전히 로드되는 항목 |
| :--- | :--- | :--- |
| `skills` | `~/.claude/skills/`, `.claude/skills/` | 플러그인 스킬, 번들 스킬, 관리형 정책 디렉터리의 스킬 |
| `agents` | `~/.claude/agents/`, `.claude/agents/` | 플러그인 에이전트, 내장 에이전트, 관리형 정책 디렉터리의 에이전트 |
| `hooks` | 사용자, 프로젝트 및 로컬 `settings.json`의 훅 | 플러그인 훅, 관리형 설정의 훅 |
| `mcp` | `~/.claude.json` 및 `.mcp.json`의 서버 | 플러그인 MCP 서버, [`managed-mcp.json`](/docs/en/managed-mcp) 서버 |

Claude Code 버전이 인식하지 못하는 영역 이름은 설정 파일을 실패 처리하는 대신 무시되므로 모든 클라이언트가 업데이트되기 전에 새 영역 이름을 추가할 수 있습니다.

### 플러그인 관리

대화형으로 플러그인을 관리하려면 `/plugin` 명령을 사용하세요:

* 마켓플레이스에서 사용 가능한 플러그인 둘러보기
* 플러그인 설치/제거
* 플러그인 활성화/비활성화
* 플러그인 상세 정보 확인 (제공되는 스킬, 에이전트, 훅)
* 마켓플레이스 추가/제거

플러그인 시스템에 대해 자세히 알아보려면 [플러그인 문서](/docs/en/plugins)를 참조하세요.

---

## 환경 변수 (Environment variables)

환경 변수를 사용하면 설정 파일을 편집하지 않고도 Claude Code 동작을 제어할 수 있습니다. 모든 변수는 모든 세션에 적용하거나 팀에 배포하기 위해 [`settings.json`](#사용-가능한-설정)의 `env` 키 아래에 구성할 수 있습니다.

전체 목록은 [환경 변수 참조](/docs/en/env-vars)를 참조하세요.

---

## Claude가 사용할 수 있는 도구 (Tools available to Claude)

Claude Code는 읽기, 편집, 검색, 명령 실행 및 서브에이전트 조율을 위한 일련의 도구에 접근할 수 있습니다. 도구 이름은 권한 규칙 및 훅 매처에서 사용하는 정확한 문자열입니다.

전체 목록 및 Bash 도구 동작 세부 정보는 [도구 참조](/docs/en/tools-reference)를 참조하세요.

---

## 관련 항목

* [권한(Permissions)](/docs/en/permissions): 권한 시스템, 규칙 구문, 도구별 패턴 및 관리형 정책
* [인증(Authentication)](/docs/en/authentication): Claude Code에 대한 사용자 접근 설정
* [구성 디버깅](/docs/en/debug-your-config): 설정, 훅 또는 MCP 서버가 적용되지 않는 이유 진단
* [설치 및 로그인 문제 해결](/docs/en/troubleshoot-install): 설치, 인증 및 플랫폼 문제
