> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 조직을 위한 Claude Code 설정하기

> API 제공자, 관리형 설정, 정책 적용, 사용량 모니터링, 데이터 처리 등을 다루는 Claude Code 배포 관리자를 위한 결정 로드맵입니다.

Claude Code는 로컬 개발자 구성보다 우선하는 관리형 설정을 통해 조직 정책을 적용합니다. 관리자는 이러한 설정을 Claude 관리 콘솔, 모바일 기기 관리(MDM) 시스템 또는 디스크의 파일 형태로 전달합니다. 이 설정은 Claude가 접근할 수 있는 도구, 명령, 서버 및 네트워크 대상을 제어합니다.

이 페이지에서는 배포 결정 사항을 순서대로 살펴봅니다. 각 행은 아래의 관련 섹션 및 해당 영역의 참조 페이지로 연결됩니다.

<Note>
  SSO, SCIM 프로비저닝 및 시트 할당은 Claude 계정 수준에서 구성됩니다. 해당 단계는 [Claude Enterprise 관리자 가이드](https://claude.com/resources/tutorials/claude-enterprise-administrator-guide) 및 [시트 할당](https://support.claude.com/en/articles/11845131-use-claude-code-with-your-team-or-enterprise-plan)을 참조하세요.
</Note>

| 결정 사항 | 선택 내용 | 참조 |
| :---------------------------------------------------------------------- | :-------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [API 제공자 선택](#api-제공자-선택) | Claude Code 인증 방식 및 청구 방법 | [인증](/docs/en/authentication), [Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry) |
| [기기에 설정을 전달하는 방식 결정](#기기에-설정을-전달하는-방식-결정) | 관리 정책이 개발자 머신에 전달되는 방식 | [서버 관리형 설정](/docs/en/server-managed-settings), [설정 파일](/docs/en/settings#settings-files) |
| [강제 적용할 항목 결정](#강제-적용할-항목-결정) | 허용되는 도구, 명령 및 연동 | [권한](/docs/en/permissions), [샌드박싱](/docs/en/sandboxing) |
| [사용량 가시성 설정](#사용량-가시성-설정) | 비용 및 도입 현황 추적 방식 | [분석](/docs/en/analytics), [모니터링](/docs/en/monitoring-usage), [비용](/docs/en/costs) |
| [데이터 처리 검토](#데이터-처리-검토) | 데이터 보관 및 규정 준수태세 | [데이터 사용](/docs/en/data-usage), [보안](/docs/en/security) |

## API 제공자 선택

Claude Code는 여러 API 제공자 중 하나를 통해 Claude에 연결됩니다. 선택한 제공자에 따라 청구 방식, 인증, 상속받는 규정 준수태세, 개발자가 사용할 수 있는 Claude Code 기능이 달라집니다.

| 제공자 | 선택 권장 대상 |
| :---------------------------- | :------------------------------------------------------------------------------------------------------------------------------------ |
| Claude for Teams / Enterprise | 실행할 인프라 없이 시트당 단일 구독으로 Claude Code와 claude.ai를 함께 사용하고자 하는 경우. 기본 권장 사항입니다. |
| Claude Console | API 우선 방식이거나 종량제(pay-as-you-go) 청구를 원하는 경우 |
| Amazon Bedrock | 기존 AWS 규정 준수 제어 및 청구 체계를 상속받고자 하는 경우 |
| Google Cloud's Agent Platform | 기존 GCP 규정 준수 제어 및 청구 체계를 상속받고자 하는 경우 |
| Microsoft Foundry | 기존 Azure 규정 준수 제어 및 청구 체계를 상속받고자 하는 경우 |

일부 Claude Code 기능은 claude.ai 계정이 필요합니다. [웹용 Claude Code](/docs/en/claude-code-on-the-web), [루틴(Routines)](/docs/en/routines), [코드 리뷰(Code Review)](/docs/en/code-review), [원격 제어(Remote Control)](/docs/en/remote-control) 및 [Chrome 확장 프로그램](/docs/en/chrome)은 Console API 키나 클라우드 제공자 자격 증명만으로는 사용할 수 없습니다. Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry를 통해 배포하는 경우 개발자에게 Claude for Teams 또는 Enterprise 시트가 추가로 필요한지 계획하세요. 각 기능 페이지에 해당 요금제 요구사항이 명시되어 있습니다.

인증, 리전 및 기능 동등성을 다루는 전체 제공자 비교는 [엔터프라이즈 배포 개요](/docs/en/third-party-integrations)를 참조하세요. 각 제공자의 인증 설정은 [인증](/docs/en/authentication)에 있습니다.

[네트워크 구성](/docs/en/network-config)의 프록시 및 방화벽 요구사항은 제공자와 상관없이 적용됩니다. 여러 제공자 앞에 단일 엔드포인트를 두거나 중앙 집중식 요청 로깅을 원하는 경우 [LLM 게이트웨이](/docs/en/llm-gateway)를 참조하세요.

## 기기에 설정을 전달하는 방식 결정

관리형 설정은 로컬 개발자 구성보다 우선하는 정책을 정의합니다. Claude Code는 아래 4가지 소스를 우선순위대로 확인하고 비어 있지 않은 구성을 반환하는 첫 번째 소스를 적용합니다. 샌드박스 허용 목록 잠금과 같은 소형 [교차 소스 잠금 키](/docs/en/settings#settings-precedence) 세트는 관리자 제어 소스 중 하나라도 지정하면 준수되며, [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)가 구성되면 해당 출력만이 이 확인 작업이 읽는 유일한 소스가 됩니다.

| 메커니즘 | 전달 방식 | 우선순위 | 플랫폼 |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- | :------------- |
| 서버 관리형 | claude.ai 관리 콘솔, 또는 게이트웨이 로그인용 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) | 가장 높음 | 전체 |
| plist / 레지스트리 정책 | macOS: `com.anthropic.claudecode` plist<br />Windows: `HKLM\SOFTWARE\Policies\ClaudeCode` | 높음 | macOS, Windows |
| 파일 기반 관리형 | macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`<br />Linux 및 WSL: `/etc/claude-code/managed-settings.json`<br />Windows: `C:\Program Files\ClaudeCode\managed-settings.json` | 중간 | 전체 |
| Windows 사용자 레지스트리 | `HKCU\SOFTWARE\Policies\ClaudeCode` | 가장 낮음 | Windows 전용 |

구성된 [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)는 4가지 소스 모두에 선행합니다. 즉, 그 출력이 해당 실행에 대한 유일한 관리형 구성이 됩니다. [설정 우선순위](/docs/en/settings#settings-precedence)를 참조하세요.

서버 관리형 설정은 별도의 엔드포인트 인프라 없이 인증 시 기기에 전달되며 활성 세션 중 매시간 새로 고침됩니다. claude.ai 관리 콘솔을 통한 전달에는 Claude for Teams 또는 Enterprise 요금제가 필요합니다. Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry 배포는 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)를 실행하여 동일한 원격 전달을 구현하거나, 파일 기반 또는 OS 수준 메커니즘을 대신 사용할 수 있습니다.

조직에서 여러 제공자를 혼용하는 경우, claude.ai 사용자를 위한 [서버 관리형 설정](/docs/en/server-managed-settings)과 다른 사용자가 관리 정책을 계속 받을 수 있도록 [파일 기반 또는 plist/레지스트리 대체(fallback)](/docs/en/settings#settings-files)를 함께 구성하세요.

plist 및 HKLM 레지스트리 위치는 모든 제공자에서 작동하며 쓰기에 관리자 권한이 필요하므로 변조에 강합니다. HKCU의 Windows 사용자 레지스트리는 승인(elevation) 없이 쓸 수 있으므로 강제 적용 채널이라기보다는 편의성 기본값으로 취급하세요.

기본적으로 WSL은 `/etc/claude-code`에 있는 Linux 파일 경로만 읽습니다. 동일한 머신의 Windows 레지스트리 및 `C:\Program Files\ClaudeCode` 정책을 WSL로 확장하려면 두 관리자 전용 Windows 소스 중 하나에 [`wslInheritsWindowsSettings: true`](/docs/en/settings#available-settings)를 설정하세요.

어떤 메커니즘을 선택하든 관리형 값은 사용자 및 프로젝트 설정보다 우선합니다. `permissions.allow` 및 `permissions.deny`와 같은 배열 설정은 모든 소스의 항목을 병합하므로 개발자가 관리형 목록을 확장할 수는 있지만 삭제할 수는 없습니다. [두 가지 예외](/docs/en/settings#settings-precedence)인 `fallbackModel` 및 `availableModels`의 경우, 관리형 값이 하위 계층을 병합하지 않고 대체합니다.

[서버 관리형 설정](/docs/en/server-managed-settings) 및 [설정 파일과 우선순위](/docs/en/settings#settings-files)를 참조하세요.

### Claude Code Desktop에서의 WSL 세션

Windows에서 [Claude Code Desktop은 WSL 2 배포판 내부에서 Code 세션을 실행할 수 있습니다](/docs/en/desktop-wsl). 세션의 Claude Code 프로세스는 배포판 내부에서 실행되므로 위의 WSL 탐색 경로를 통해 관리형 설정을 확인합니다. Windows 전용 소스는 `wslInheritsWindowsSettings: true`가 배포되지 않는 한 도달하지 않습니다.

관리형 설정이 있는 기기에서는 Desktop WSL 세션을 기본적으로 사용할 수 없습니다. 조직에서 이를 활성화하려면 Anthropic 계정 팀에 문의하세요. 활성화된 경우:

* HKLM 레지스트리 또는 `C:\Program Files\ClaudeCode` 파일을 통해 `wslInheritsWindowsSettings: true`를 배포하여 WSL 세션이 호스트 세션과 동일한 정책을 상속받도록 하세요.
* WSL 세션 내부에서 `/status`를 실행하여 확인하세요. `Setting sources` 줄에 배포한 Windows 소스인 `(HKLM)` 또는 `(file)`과 함께 `Enterprise managed settings`가 표시되어야 합니다.

WSL 2 유틸리티 VM 내부의 프로세스는 Windows 측 엔드포인트 탐지 센서에 보이지 않습니다. CrowdStrike Falcon을 사용하는 경우 CrowdStrike의 WSL 문서에서 요구하는 두 가지 제외 사항(WSL 가상 머신 프로세스 및 VM 디스크 이미지)과 함께 WSL 2용 Linux 전용 Falcon 센서를 활성화하여 배포판 내부의 프로세스 및 파일 활동을 관찰 가능하게 만드세요. Claude Code의 [OpenTelemetry 도구 실행 텔레메트리](/docs/en/monitoring-usage)는 WSL 및 네이티브 세션 모두 동일하게 전송됩니다.

## 강제 적용할 항목 결정

관리형 설정을 사용하면 도구를 잠그고, 실행을 샌드박싱하고, MCP 서버 및 플러그인 소스를 제한하고, 실행되는 후크(hook)를 제어할 수 있습니다. 각 행은 해당 제어를 구동하는 설정 키가 있는 제어 영역입니다.

| 제어 항목 | 역할 | 주요 설정 |
| :--------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------- |
| [권한 규칙](/docs/en/permissions) | 특정 도구 및 명령 허용, 확인(ask), 거부 | `permissions.allow`, `permissions.deny` |
| [권한 잠금](/docs/en/permissions#managed-only-settings) | 관리형 권한 규칙만 적용되며 `--dangerously-skip-permissions` 비활성화 | `allowManagedPermissionRulesOnly`, `permissions.disableBypassPermissionsMode` |
| [샌드박싱](/docs/en/sandboxing) | 네트워크 도메인 허용 목록을 통한 OS 수준 파일 시스템 및 네트워크 격리 | `sandbox.enabled`, `sandbox.network.allowedDomains` |
| [관리형 정책 CLAUDE.md](/docs/en/memory#deploy-organization-wide-claude-md) | 모든 세션에 로드되는 조직 차원의 지침, 제외 불가능 | 관리형 정책 경로의 파일 |
| [MCP 서버 제어](/docs/en/managed-mcp) | 사용자가 추가/연결할 수 있는 MCP 서버를 제한하거나 고정 세트 배포 | `allowedMcpServers`, `deniedMcpServers`, `allowManagedMcpServersOnly` 또는 배포된 `managed-mcp.json` 파일 |
| [플러그인 마켓플레이스 제어](/docs/en/plugin-marketplaces#managed-marketplace-restrictions) | 사용자가 추가/설치할 수 있는 마켓플레이스 소스를 제한하고, 단일 실행을 위해 플러그인, 에이전트, MCP 서버를 사이드로드하는 CLI 플래그를 거부하며, 추천 가능한 플러그인 마켓플레이스를 허용 목록화 | `strictKnownMarketplaces`, `blockedMarketplaces`, `disableSideloadFlags`, `pluginSuggestionMarketplaces` |
| [커스텀 요소를 관리형/플러그인 전용으로 제한](/docs/en/settings#strictpluginonlycustomization) | 사용자 및 프로젝트 소스의 스킬, 에이전트, 후크, MCP 서버를 차단하여 플러그인 또는 관리형 설정에서만 가져올 수 있도록 제어 | `strictPluginOnlyCustomization` |
| [후크 제한](/docs/en/settings#hook-configuration) | 관리형 후크만 로드하도록 설정하고 HTTP 후크 URL을 제한 | `allowManagedHooksOnly`, `allowedHttpHookUrls` |
| [로그인 강제 적용](/docs/en/settings#available-settings) | 특정 방식이나 Anthropic 조직으로 로그인을 제한. 방식 제한은 터미널, VS Code 확장 프로그램, Agent SDK, `claude setup-token`, `/install-github-app` 전체에 적용되며, 조직 제한은 Anthropic 조직에 대해 인증하지 않는 [게이트웨이](/docs/en/claude-apps-gateway) 로그인을 제외한 터미널, VS Code 확장 프로그램, Agent SDK를 처리함. {/* min-version: 2.1.212 */}v2.1.212 이전에는 터미널 로그인만 두 키 중 하나를 강제 적용함. 설정 시 `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` 또는 `apiKeyHelper`로 인증된 세션은 시작 시 차단되지만 클라우드 제공자 세션은 영향받지 않음 | `forceLoginMethod`, `forceLoginOrgUUID` |
| [에이전트 뷰 비활성화](/docs/en/agent-view#how-background-sessions-are-hosted) | `claude agents`, `--bg`, `/background` 및 온디맨드 수퍼바이저 비활성화 | `disableAgentView` |
| [사내 론처 구성](/docs/en/corporate-launcher) | 에이전트 뷰를 끄는 대신 [백그라운드 에이전트 수퍼바이저](/docs/en/agent-view#how-background-sessions-are-hosted), 그 워커 및 [기타 대상 백그라운드 프로세스](/docs/en/corporate-launcher#what-the-launcher-covers) 앞에 필수 사내 론처(corporate launcher) 접두사 부과 | `processWrapper` |
| [모델 제한](/docs/en/model-config#restrict-model-selection) | `availableModels`로 선택기에 표시되는 모델을 필터링함. `enforceAvailableModels`를 추가하면 자동 선택되는 기본 모델도 제한됨. 이 설정이 CLI, 웹, IDE에 전달되는 방식은 [표면 영역 범위](/docs/en/model-config#surface-coverage) 참조 | `availableModels`, `enforceAvailableModels` |
| [최소 버전(Version floor)](/docs/en/settings) | 자동 업데이트가 조직 최저 기준 미만으로 설치되는 것을 방지 | `minimumVersion` |
| [필수 버전 범위](/docs/en/settings) | 실행 버전이 조직 승인 범위를 벗어나면 시작을 즉시 거부함. 다운그레이드만 차단하는 `minimumVersion`보다 강력함 | `requiredMinimumVersion`, `requiredMaximumVersion` |

claude.ai 또는 Anthropic API를 통해 인증하는 구성원을 둔 조직은 설정을 배포하지 않고도 모델을 통제할 수 있습니다: [조직 모델 제한](/docs/en/model-config#organization-model-restrictions)은 개별 모델을 비활성화하고, [조직 기본 모델](/docs/en/model-config#organization-default-model)은 새 세션이 시작될 모델을 설정하며, [조직 노력(effort) 제한](/docs/en/model-config#organization-effort-limits)은 역할별 effort 수준을 제한합니다. 3가지 제어 기능 모두 Claude Enterprise 요금제가 필요합니다. 모델 제한 및 effort 제한은 서버 측에서 강제 적용되며, 기본 모델은 조직에서 강제 적용하지 않는 한 사용자가 변경할 수 있는 시작점입니다. 강제 적용은 제한된 일부 조직에서만 사용할 수 있습니다. 사용 가능 여부는 Anthropic 계정 팀에 문의하세요. 이러한 제어 기능은 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 또는 [AWS의 Claude Platform](/docs/en/claude-platform-on-aws) 세션에는 적용되지 않습니다. 해당 제공자의 경우 제한에는 위의 `availableModels`를 사용하고 기본값에는 관리형 설정의 `model` 키를 사용하세요.

[웹용 Claude Code](/docs/en/claude-code-on-the-web)에는 고유한 관리자 표면이 있습니다. 관리 설정의 클라우드 환경 페이지에서 소유자 및 관리자는 구성원의 클라우드 세션을 위한 [네트워크 접근 수준](/docs/en/claude-code-on-the-web#network-access), 환경 변수, 설정 스크립트를 정의하는 [조직 공유 환경](/docs/en/claude-code-on-the-web#organization-shared-environments)을 생성하고 조직의 기본 환경을 선택합니다.

권한 규칙과 샌드박싱은 서로 다른 계층을 다룹니다. WebFetch를 거부하면 Claude의 fetch 도구가 차단되지만 Bash가 허용된 경우 `curl` 및 `wget`은 모든 URL에 계속 접근할 수 있습니다. 샌드박싱은 OS 수준에서 적용되는 네트워크 도메인 허용 목록을 통해 이러한 격차를 줄입니다.

이러한 제어 기능이 방어하는 위협 모델은 [보안](/docs/en/security)을 참조하세요.

## 사용량 가시성 설정

보고해야 하는 항목에 따라 모니터링 방식을 선택하세요. 대시보드, API, 지출 제어 기능은 Claude for Teams / Enterprise 요금제와 Claude Console 조직 간에 차이가 있으므로 기능에 대한 보고 체계를 계획하기 전에 '사용 가능 여부' 열을 확인하세요.

| 기능 | 제공 내용 | 사용 가능 여부 | 시작하기 |
| :--------------------- | :-------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------- |
| 사용량 모니터링 | 세션, 도구, 토큰의 OpenTelemetry 내보내기 | 모든 제공자 | [사용량 모니터링](/docs/en/monitoring-usage) |
| 분석 대시보드 | Teams / Enterprise에서는 리더보드가 포함된 도입 및 기여도 지표, Console에서는 사용자별 사용량 및 지출 지표 | Teams / Enterprise는 [claude.ai/analytics](https://claude.ai/analytics/claude-code), Console은 [platform.claude.com/claude-code](https://platform.claude.com/claude-code) | [분석](/docs/en/analytics) |
| 프로그래밍 방식 보고 | API를 통한 사용자별 사용량 및 비용 데이터 | Enterprise의 경우 [Enterprise Analytics API](https://platform.claude.com/docs/en/api/admin/analytics), Console의 경우 [Claude Code Analytics API](https://platform.claude.com/docs/en/build-with-claude/claude-code-analytics-api) | [비용](/docs/en/costs#manage-costs-for-your-organization) |
| 지출 제어 | 지출 한도 및 속도(rate) 한도 | Teams / Enterprise의 경우 관리자 설정, Console의 경우 워크스페이스 한도. 타사 클라우드에서는 클라우드 예산 제어 또는 사용자별 [지출 한도](/docs/en/claude-apps-gateway-spend-limits)가 있는 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) | [비용](/docs/en/costs#manage-costs-for-your-organization) |

Teams 및 Enterprise의 경우 사용자별 사용량 및 지출 수치는 분석 대시보드가 아닌 조직의 분석 설정에 있는 [지출 보고서](https://support.claude.com/en/articles/12883420-view-usage-analytics-for-team-and-enterprise-plans)에서 제공됩니다. 클라우드 제공자는 AWS Cost Explorer, GCP Billing 또는 Azure Cost Management를 통해 지출을 노출합니다. Claude chat, Claude Code, Cowork 전체에 걸친 엔터프라이즈 예산 계획은 [Claude Enterprise 소비 가이드](https://support.claude.com/en/articles/14782391-claude-enterprise-consumption-guide)를 참조하세요.

## 데이터 처리 검토

Team, Enterprise, Claude API, 클라우드 제공자 요금제에서 Anthropic은 사용자의 코드나 프롬프트로 모델을 학습시키지 않습니다. 데이터 보관 및 규정 준수태세는 사용 중인 API 제공자가 결정합니다.

| 주제 | 주요 내용 | 시작하기 |
| :------------------------ | :--------------------------------------------------------------------------------------------------- | :--------------------------------------------- |
| 데이터 사용 정책 | Anthropic이 수집하는 정보, 보관 기간, 학습에 절대 사용되지 않는 항목 | [데이터 사용](/docs/en/data-usage) |
| 제로 데이터 보관 (ZDR) | 요청이 완료된 후 아무것도 저장되지 않음. Claude for Enterprise의 자격 기준을 충족하는 계정에서 이용 가능 | [제로 데이터 보관](/docs/en/zero-data-retention) |
| 보안 아키텍처 | 네트워크 모델, 암호화, 인증, 감사 추적(audit trail) | [보안](/docs/en/security) |

요청 수준의 감사 로깅이 필요하거나 데이터 민감도에 따라 트래픽을 라우팅해야 하는 경우 개발자와 제공자 사이에 게이트웨이를 배치하세요. 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)는 IdP ID가 포함된 요청별 감사 로그를 기록합니다. 다른 [LLM 게이트웨이](/docs/en/llm-gateway)를 사용할 수도 있습니다. 규제 요구사항 및 인증 정보는 [법적 공지 및 규정 준수](/docs/en/legal-and-compliance)를 참조하세요.

## 검증 및 온보딩

관리형 설정을 구성한 후 개발자에게 Claude Code 내부에서 `/status`를 실행하도록 안내하세요. **Status** 탭의 `Setting sources` 줄에 `Enterprise managed settings`와 함께 괄호 안에 `(remote)`, `(plist)`, `(HKLM)`, `(HKCU)` 또는 `(file)` 중 하나의 소스가 표시됩니다. [활성 설정 검증](/docs/en/settings#verify-active-settings)을 참조하세요.

개발자가 빠르게 시작할 수 있도록 다음 리소스를 공유하세요:

* [빠른 시작](/docs/en/quickstart): 설치부터 프로젝트 작업까지 첫 세션 가이드
* [일반적인 워크플로우](/docs/en/common-workflows): 코드 리뷰, 리팩토링, 디버깅과 같은 일상적인 작업을 위한 패턴
* [Claude 101](https://anthropic.skilljar.com/claude-101) 및 [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action): Anthropic Academy 자습형 강좌

로그인 문제가 발생하는 개발자는 [인증 문제 해결](/docs/en/troubleshoot-install#login-and-authentication)을 참조하도록 하세요. 가장 흔한 해결 방법은 다음과 같습니다:

* 계정을 전환하려면 `/logout` 후 `/login` 실행
* 엔터프라이즈 인증 옵션이 누락된 경우 `claude update` 실행
* 업데이트 후 터미널 재시작

개발자에게 "You haven't been added to your organization yet"이라는 메시지가 표시되면 해당 개발자의 시트에 Claude Code 접근 권한이 포함되어 있지 않은 것이므로 관리 콘솔에서 업데이트해야 합니다.

## 다음 단계

제공자 및 전달 메커니즘을 선택한 후 상세 구성으로 진행하세요:

* [서버 관리형 설정](/docs/en/server-managed-settings): Claude 관리 콘솔에서 관리 정책 전달
* [설정 참조](/docs/en/settings): 모든 설정 키, 파일 위치 및 우선순위 규칙
* [모노레포 및 대형 레포지토리](/docs/en/large-codebases): 모노레포에 배포하는 조직을 위한 디렉토리별 구성 패턴
* [Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry): 제공자별 배포
* [Claude Enterprise 관리자 가이드](https://claude.com/resources/tutorials/claude-enterprise-administrator-guide): SSO, SCIM, 시트 관리 및 롤아웃 플레이북
