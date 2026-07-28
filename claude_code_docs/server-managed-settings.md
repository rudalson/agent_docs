> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 서버 관리형 설정 구성하기

> 디바이스 관리 인프라 없이도 서버 전달 설정을 통해 조직 전체의 Claude Code를 중앙에서 구성하세요.

서버 관리형 설정을 사용하면 조직 소유자(Owner)가 claude.ai 콘솔의 [**Admin Settings > Claude Code > Managed settings**](https://claude.ai/admin-settings/claude-code)에서 Claude Code를 중앙에서 구성할 수 있습니다. Claude Code 클라이언트는 서버 관리형 전달이 지원되는 플랫폼에서 사용자가 조직 OAuth 로그인 또는 직접 구성된 API 키로 인증할 때 이러한 설정을 자동으로 가져옵니다. [플랫폼 지원 여부](#플랫폼-지원-여부)를 참조하세요.

이 접근 방식은 디바이스 관리 인프라가 갖추어져 있지 않거나 관리되지 않는 디바이스를 사용하는 사용자의 설정을 관리해야 하는 조직에 적합하도록 설계되었습니다.

<Note>
  서버 관리형 설정은 [Claude for Teams](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=server_settings_teams#team-&-enterprise) 및 [Claude for Enterprise](https://anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=server_settings_enterprise) 고객에게 제공됩니다.
</Note>

## 요구 사항

서버 관리형 설정을 사용하려면 다음이 필요합니다:

* Claude for Teams 또는 Claude for Enterprise 플랜
* 구성을 조회하고 편집할 수 있는 Claude 조직 내 Owner 또는 Primary Owner 역할
* `api.anthropic.com`에 대한 네트워크 액세스

## 서버 관리형 설정과 엔드포인트 관리형 설정 중 선택하기

Claude Code는 중앙 관리형 구성을 위해 두 가지 접근 방식을 지원합니다. 서버 관리형 설정은 Anthropic의 서버에서 구성을 전달합니다. [엔드포인트 관리형 설정](/docs/en/settings#settings-files)은 네이티브 OS 정책(macOS managed preferences, Windows registry) 또는 관리형 설정 파일을 통해 디바이스에 직접 배포됩니다.

| 접근 방식 | 적합한 대상 | 보안 모델 |
| :--- | :--- | :--- |
| **서버 관리형 설정** | MDM이 없는 조직 또는 관리되지 않는 디바이스의 사용자 | 인증 시 Anthropic 서버에서 전달되는 설정 |
| **[엔드포인트 관리형 설정](/docs/en/settings#settings-files)** | MDM 또는 엔드포인트 관리가 갖춰진 조직 | MDM 구성 프로필, 레지스트리 정책 또는 관리형 설정 파일을 통해 디바이스에 배포되는 설정 |

디바이스가 MDM 또는 엔드포인트 관리 솔루션에 등록되어 있는 경우, OS 수준에서 사용자의 설정 파일 수정이 차단될 수 있으므로 엔드포인트 관리형 설정이 더 강력한 보안 보장을 제공합니다. 엔드포인트 관리형 설정은 [클라우드 세션](/docs/en/model-config#surface-coverage)에 도달하지 않으므로 웹 기반 Claude Code를 사용하는 조직은 서버 관리형 설정도 함께 구성해야 합니다.

## 서버 관리형 설정 구성하기

<Steps>
  <Step title="관리 콘솔 열기">
    claude.ai 콘솔에서 [**Admin Settings > Claude Code > Managed settings**](https://claude.ai/admin-settings/claude-code)로 이동합니다.

    링크를 클릭했을 때 Claude Code 페이지 대신 다른 관리자 설정 페이지로 리디렉션되면 해당 계정에 필요한 역할이 없는 것입니다. 관리자(Admin) 및 기타 비소유자 역할은 관리형 설정을 조회하거나 편집할 수 없으므로 조직의 Owner 또는 Primary Owner에게 변경을 요청하세요. [액세스 제어](#액세스-제어)를 참조하세요.
  </Step>

  <Step title="설정 정의하기">
    JSON 형식으로 구성을 추가합니다. OS 수준 정책 전달에만 제한된 일부 설정을 제외하고 [`settings.json`에서 사용 가능한 모든 설정](/docs/en/settings#available-settings)이 지원됩니다. 짧은 제외 목록은 [현재 제한 사항](#현재-제한-사항)을 참조하세요. 이 지원 항목에는 [훅](/docs/en/hooks), [환경 변수](/docs/en/env-vars), 그리고 `allowManagedPermissionRulesOnly`와 같은 [관리 전용 설정](/docs/en/permissions#managed-only-settings)이 포함됩니다.

    다음 예시는 권한 거부 목록을 강제 적용하고, 사용자가 권한을 우회하지 못하도록 하며, 권한 규칙을 관리형 설정에 정의된 규칙으로만 제한합니다:

    ```json theme={null}
    {
      "permissions": {
        "deny": [
          "Bash(curl *)",
          "Read(./.env)",
          "Read(./.env.*)",
          "Read(./secrets/**)"
        ],
        "disableBypassPermissionsMode": "disable"
      },
      "allowManagedPermissionRulesOnly": true
    }
    ```

    훅은 `settings.json`에서와 동일한 형식을 사용합니다.

    다음 예시는 조직 전체에서 파일이 편집될 때마다 감사 스크립트를 실행합니다:

    ```json theme={null}
    {
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Edit|Write",
            "hooks": [
              { "type": "command", "command": "/usr/local/bin/audit-edit.sh" }
            ]
          }
        ]
      }
    }
    ```

    [자동 모드(auto mode)](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 분류기가 조직에서 신뢰하는 리포지토리, 버킷, 도메인을 인식하도록 구성하려면:

    ```json theme={null}
    {
      "autoMode": {
        "environment": [
          "Source control: github.example.com/acme-corp and all repos under it",
          "Trusted cloud buckets: s3://acme-build-artifacts, gs://acme-ml-datasets",
          "Trusted internal domains: *.corp.example.com"
        ]
      }
    }
    ```

    훅은 셸 명령을 실행하므로 적용되기 전에 사용자에게 [보안 승인 대화 상자](#보안-승인-대화-상자)가 표시됩니다. `autoMode` 항목이 분류기의 차단 대상에 미치는 영향과 `environment`, `allow`, `soft_deny`, `hard_deny` 필드에 대한 중요한 경고는 [자동 모드 구성하기](/docs/en/auto-mode-config)를 참조하세요.
  </Step>

  <Step title="저장 및 배포">
    변경 사항을 저장합니다. Claude Code 클라이언트는 다음 시작 시 또는 매시간 폴링 주기마다 업데이트된 설정을 수신합니다.
  </Step>
</Steps>

### 설정 전달 확인

설정이 적용되고 있는지 확인하려면 사용자에게 Claude Code를 재시작하도록 요청하세요. 구성에 [보안 승인 대화 상자](#보안-승인-대화-상자)를 트리거하는 설정이 포함되어 있으면 시작 시 관리형 설정을 설명하는 프롬프트가 사용자에게 표시됩니다. 또한 사용자가 `/permissions`를 실행하여 유효한 권한 규칙을 확인하게 함으로써 관리형 권한 규칙이 활성화되었는지 검증할 수 있습니다.

### 액세스 제어

다음 역할만 서버 관리형 설정을 관리할 수 있습니다:

* **Primary Owner**
* **Owner**

설정 변경 사항은 조직의 모든 사용자에게 적용되므로 신뢰할 수 있는 담당자로만 접근을 제한하세요.

### 관리 전용 설정

대부분의 [설정 키](/docs/en/settings#available-settings)는 모든 스코프에서 작동합니다. 일부 키는 관리형 설정에서만 읽히며 사용자 또는 프로젝트 설정 파일에 위치할 경우 효과가 없습니다. 전체 목록은 [관리 전용 설정](/docs/en/permissions#managed-only-settings)을 참조하세요. 해당 목록에 없는 설정도 관리형 설정에 배치할 수 있으며 가장 높은 우선순위를 갖습니다.

### 현재 제한 사항

서버 관리형 설정에는 다음과 같은 제한 사항이 있습니다:

* 설정은 조직의 모든 사용자에게 균일하게 적용됩니다. 그룹별 구성은 아직 지원되지 않습니다.
* [`managed-mcp.json`](/docs/en/managed-mcp) 파일은 서버 관리형 설정을 통해 배포할 수 없습니다. 대신 `allowedMcpServers` 및 `deniedMcpServers` 정책 키를 전달하세요.
* `policyHelper` 및 `wslInheritsWindowsSettings`와 같이 OS 수준 정책 소스로 제한된 설정은 적용되지 않습니다. 대신 MDM 또는 시스템 `managed-settings.json` 파일을 통해 배포하세요.

## 설정 전달 방식

### 설정 우선순위

서버 관리형 설정과 [엔드포인트 관리형 설정](/docs/en/settings#settings-files)은 모두 Claude Code [설정 계층 구조](/docs/en/settings#settings-precedence)에서 가장 높은 계층을 차지합니다. 명령줄 인수를 포함하여 다른 어떠한 설정 수준도 이를 재정의할 수 없습니다.

관리형 계층 내에서 구성된 [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)는 서버 관리형 설정을 포함하여 다른 모든 관리형 소스를 재지정(preempt)합니다. 해당 출력 결과가 실행 시의 유일한 관리형 구성이 됩니다.

그렇지 않은 경우 Claude Code는 비어 있지 않은 구성을 전달하는 첫 번째 소스를 사용합니다. 서버 관리형 설정을 먼저 확인한 다음 엔드포인트 관리형 설정을 확인합니다. 소스는 병합되지 않습니다. 서버 관리형 설정이 키를 하나라도 전달하면 다른 엔드포인트 관리형 설정은 무시됩니다. 서버 관리형 설정이 아무것도 전달하지 않으면 엔드포인트 관리형 설정이 적용됩니다.

샌드박스 허용 목록 잠금과 같은 소스 간 잠금 키([cross-source lock keys](/docs/en/settings#settings-precedence))의 소규모 집합은 관리자가 제어하는 관리형 소스에서 설정된 경우 존중됩니다. 사용자 작성 가능한 HKCU 레지스트리 계층은 제외되며, [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)가 구성된 경우 해당 출력이 이러한 검사가 읽는 유일한 소스가 됩니다.

엔드포인트 관리형 plist 또는 레지스트리 정책으로 대체할 목적으로 관리 콘솔에서 서버 관리형 구성을 지우는 경우, [가져오기 및 캐싱 동작](#가져오기-및-캐싱-동작)으로 인해 다음 성공적인 가져오기 전까지 클라이언트 머신에 캐시된 설정이 지속된다는 점에 유의하세요. `/status`를 실행하여 현재 활성화된 관리형 소스를 확인하세요.

### 가져오기 및 캐싱 동작

Claude Code는 시작 시 Anthropic 서버에서 설정을 가져오며, 활성 세션 동안 매시간 업데이트를 폴링합니다.

**캐시된 설정이 없는 첫 실행 시:**

* Claude Code가 비동기로 설정을 가져옵니다.
* 가져오기에 실패하면 Claude Code는 관리형 설정 없이 계속 진행합니다.
* 설정을 로드하기 전에 제한 사항이 아직 적용되지 않는 짧은 인터벌이 존재합니다.

**캐시된 설정이 있는 후속 실행 시:**

* 아래 설명된 전송, 라우팅 및 인증 환경 변수를 제외하고 시작 즉시 캐시된 설정이 적용됩니다.
* Claude Code가 백그라운드에서 새로운 설정을 가져옵니다.
* 캐시된 설정은 네트워크 오류 동안에도 유지됩니다. 보류된 환경 변수는 가져오기가 성공할 때까지 보류 상태로 유지됩니다.

v2.1.198부터 Claude Code는 서버가 세션에 대한 페이로드를 확인할 때까지 캐시된 `env` 블록에서 세 가지 범주의 변수를 보류합니다. 이는 캐시된 프록시, 인증서 기관(CA), 엔드포인트 또는 자격 증명 값이 페이로드를 확인하는 설정 가져오기 요청을 리디렉션, 도청 또는 재인증하지 못하도록 방지합니다. 이 강화 조치는 서버에서 가져온 설정 캐시에만 적용되며, MDM 또는 `managed-settings.json`을 통해 배포된 [엔드포인트 관리형 설정](/docs/en/settings#settings-files)은 영향을 받지 않습니다. 보류되는 범주는 다음과 같습니다:

* `HTTPS_PROXY`, `NODE_EXTRA_CA_CERTS`, mTLS 클라이언트 인증서 변수 `CLAUDE_CODE_CLIENT_CERT` 및 `CLAUDE_CODE_CLIENT_KEY`와 같은 프록시 및 TLS 구성
* `ANTHROPIC_BASE_URL`, `CLAUDE_CODE_USE_BEDROCK` 및 `CLAUDE_CODE_USE_VERTEX`와 같은 제공업체 선택 변수, `ANTHROPIC_BEDROCK_BASE_URL`과 같은 제공업체 엔드포인트 URL을 포함한 API 라우팅 및 제공업체 선택
* `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `CLAUDE_CODE_OAUTH_TOKEN`과 같은 인증 자격 증명

텔레메트리 및 OpenTelemetry 구성을 포함하여 캐시된 `env` 블록의 다른 모든 키는 이전과 마찬가지로 시작 시 적용됩니다. 가져오기가 성공하면 보류되었던 변수가 세션의 나머지 기간 동안 적용됩니다.

조직에 `api.anthropic.com`에 도달하기 위한 프록시가 필요한 경우 관리형 `env` 블록에만 설정하지 말고 셸 환경이나 [사용자 설정](/docs/en/settings#settings-files)에 설정하세요. 첫 실행 시에는 캐시가 없으므로 해당 소스가 초기 가져오기에 이미 필요합니다.

Claude Code는 전면 재시작이 필요한 OpenTelemetry 구성과 같은 고급 설정을 제외하고 재시작 없이 설정 업데이트를 자동으로 적용합니다.

### 전달된 설정의 유효하지 않은 항목

전달된 페이로드는 다른 관리형 소스와 동일한 규칙에 따라 관대하게 파싱됩니다. 페이로드에 스키마 유효성 검사에 실패한 항목이 포함되어 있으면 Claude Code는 해당 항목을 제거하고 유효성 검사 오류를 표시하며 나머지 유효한 모든 설정을 적용합니다. 보안 강제 필드가 처리되는 방식을 포함한 필드 수준 동작은 [관리형 설정의 유효하지 않은 항목](/docs/en/settings#invalid-entries-in-managed-settings)을 참조하세요. Claude Code v2.1.169 이상이 필요합니다.

서버 관리형 전달에는 다음과 같은 동작이 추가됩니다:

* `~/.claude/remote-settings.json`의 캐시는 유효하지 않은 항목이 제거된 복구된 페이로드를 저장합니다. 원본의 유효하지 않은 페이로드는 저장되지 않습니다.
* 페이로드의 어떤 필드도 복구할 수 없는 경우 Claude Code는 마지막으로 수락된 캐시 설정을 유지하고 치명적 오류를 기록합니다.
* [보안 승인 대화 상자](#보안-승인-대화-상자)는 복구된 페이로드를 평가하므로 제거된 유효하지 않은 항목은 승인을 위해 제시되지 않으며 실행되지도 않습니다.

전달 문제를 디버깅하려면 `claude --debug-file <path>`를 실행하고 로그에서 `Remote settings`를 검색하세요. 조직에 적용하기 전에 테스트 머신에서 `claude doctor`로 페이로드 변경 사항을 검증하세요.

### 실패 시 종료 시작 강제 적용 (Enforce fail-closed startup)

기본적으로 원격 설정 가져오기가 시작 시 실패하면 CLI는 관리형 설정 없이 진행합니다. 이러한 짧은 미강제 인터벌이 허용되지 않는 환경의 경우 관리형 설정에서 `forceRemoteSettingsRefresh: true`를 설정하세요.

이 설정이 활성화되면 CLI는 원격 설정을 새로 가져올 때까지 시작 시 차단됩니다. 가져오기가 실패하면 CLI는 정책 없이 진행하는 대신 종료됩니다. 이 설정은 자체 지속(self-perpetuate)됩니다. 서버에서 한 번 전달되면 로컬에 캐시되어 다음 세션의 첫 성공적인 가져오기 전에도 후속 시작 시 동일한 동작이 적용됩니다.

이를 활성화하려면 관리형 설정 구성에 해당 키를 추가하세요:

```json theme={null}
{
  "forceRemoteSettingsRefresh": true
}
```

첫 실행 시 서버 페이로드가 전달되기 전에 실패 시 종료 동작을 적용하기 위해 [엔드포인트 관리형](/docs/en/settings#settings-files) MDM 프로필이나 시스템 `managed-settings.json` 파일에 이 키를 설정할 수도 있습니다. v2.1.191부터 이 플래그는 위의 [우선순위 규칙](#설정-우선순위)의 예외입니다. 캐시된 서버 관리형 페이로드가 존재하는 경우에도 관리자가 제어하는 관리형 소스에 설정되면 존중되므로 서버 관리형 설정이 존재하더라도 MDM 전달 값이 무시되지 않습니다. [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)가 구성된 경우 그 출력이 이 키를 포함한 다른 모든 관리형 소스를 대체합니다.

설정 가져오기는 또한 `Cache-Control: no-cache` 헤더를 전송하여 중간 HTTP 프록시가 오래된 응답을 제공하지 않도록 합니다.

이 설정을 활성화하기 전에 네트워크 정책이 `api.anthropic.com`으로의 연결을 허용하는지 확인하세요. 해당 엔드포인트에 도달할 수 없는 경우 시작 시 CLI가 종료되어 사용자가 Claude Code를 시작할 수 없습니다.

v2.1.139부터 `claude auth login`과 같은 `claude auth` 하위 명령은 이 검사에서 제외되므로 자격 증명 만료가 설정 가져오기 실패 원인인 경우 사용자가 다시 인증할 수 있습니다.

### 보안 승인 대화 상자

보안 위험을 초래할 수 있는 특정 설정은 Claude Code가 적용하기 전에 사용자의 명시적인 승인이 필요합니다:

* **셸 명령 설정**: 셸 명령을 실행하는 설정
* **사용자 지정 환경 변수**: 알려진 안전 허용 목록에 없는 변수
* **훅 구성**: 모든 훅 정의
* **관리형 CLAUDE.md 콘텐츠**: 관리형 설정을 통해 전달되는 `claudeMd` 값

이러한 설정이 존재하는 경우 사용자에게 무엇이 구성되는지 설명하는 보안 대화 상자가 표시됩니다. 진행하려면 사용자가 승인해야 합니다. 사용자가 설정을 거부하면 Claude Code가 종료됩니다.

대화형 세션에서 대화 상자를 표시할 수 없는 경우 Claude Code는 전달된 설정을 적용하지 않고 마지막으로 승인된 설정을 유지합니다. 대화 상자는 이를 표시할 수 있는 다음 세션에 나타납니다. Claude Code v2.1.211 이상이 필요합니다.

<Note>
  `claude -p`나 Agent SDK 세션과 같은 비대화형 실행은 대화 상자를 표시할 수 없습니다. 전달된 설정에 승인이 필요한 경우 Claude Code는 해당 실행에만 설정을 적용합니다. 설정을 승인됨으로 기록하거나 [로컬 캐시](#가져오기-및-캐싱-동작)에 쓰지 않으며 다음 대화형 세션에서 대화 상자가 나타납니다. 사용자가 대화형 세션에서 승인할 때까지 각 비대화형 실행은 시작 시 설정을 다시 가져옵니다. v2.1.207 이전에는 비대화형 실행 시 설정을 승인됨으로 저장하여 이후의 대화형 세션에서 대화 상자가 표시되지 않았습니다.
</Note>

## 플랫폼 지원 여부

서버 관리형 설정은 `api.anthropic.com`에 대한 직접 연결이 필요하며, 전달을 위해서는 세션이 조직 OAuth 로그인 또는 직접 구성된 API 키로 인증되어야 합니다. [`apiKeyHelper`](/docs/en/settings#available-settings) 스크립트에 의해 반환된 키는 설정 가져오기를 트리거하지 않습니다.

서버 관리형 설정은 서드파티 모델 제공업체를 사용할 때 사용할 수 없습니다:

* Amazon Bedrock
* Google Cloud's Agent Platform
* Microsoft Foundry
* [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)
* `ANTHROPIC_BASE_URL` 또는 서드파티 [LLM 게이트웨이](/docs/en/llm-gateway)를 통한 사용자 지정 API 엔드포인트

셸에서 `CLAUDE_CODE_USE_*` 제공업체 변수 또는 기본값이 아닌 `ANTHROPIC_BASE_URL`을 내보내는 경우(export), Claude Code는 세션에 대한 설정 가져오기를 건너뜁니다. 내보내기는 가져오기를 차단하므로 서버 관리형 `env` 블록으로 내보내기를 지울 수 없습니다. [엔드포인트 관리형 설정](/docs/en/settings#settings-files) `env` 블록도 가져오기를 복원하지 못합니다. Claude Code는 관리형 `env` 블록을 적용하기 전에 자격을 확인하므로 오버라이드가 세션의 제공업체 선택을 변경하지만 가져오기는 건너뛴 상태로 유지됩니다.

서버 관리형 전달을 복원하려면 셸에서 내보내기를 제거하거나 자격 확인 전에 적용되는 사용자 설정 `env` 블록에서 변수를 `""`로 설정하세요. 사용자가 셸을 변경하는 것에 의존하지 않고 정책을 강제 적용하려면 엔드포인트 관리형 채널을 통해 설정을 배포하세요.

Amazon Bedrock, Google Cloud's Agent Platform 및 Microsoft Foundry 배포의 경우, 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)가 동등한 원격 관리형 설정 전달을 제공합니다. 게이트웨이에 로그인한 클라이언트는 `api.anthropic.com` 대신 게이트웨이에서 관리형 설정을 가져옵니다. 시작 시 실패 시맨틱이 다릅니다. 게이트웨이에 도달할 수 없는 게이트웨이 클라이언트는 캐시된 설정으로 대체되는 대신 오류와 함께 종료되는 반면, 매시간 백그라운드 새로 고침은 두 채널 모두에서 장애 시 허용(fail-open) 방식입니다.

## 감사 로깅

설정 변경에 대한 감사 로그 이벤트는 컴플라이언스 API 또는 감사 로그 내보내기를 통해 확인할 수 있습니다. 액세스에 대해서는 Anthropic 계정 팀에 문의하세요.

감사 이벤트에는 수행된 작업 유형, 작업을 수행한 계정 및 디바이스, 이전 값과 새 값에 대한 참조가 포함됩니다.

## 보안 고려 사항

서버 관리형 설정은 중앙 관리형 정책 강제를 제공하지만 보안 경계가 아닌 클라이언트 측 제어로 작동합니다. 관리되지 않는 디바이스에서 사용자는 이를 우회하기 위해 관리자나 sudo 권한이 필요하지 않습니다.

| 시나리오 | 동작 |
| :--- | :--- |
| 사용자가 캐시된 설정 파일을 편집함 | 변조된 파일이 시작 시 적용되지만 다음 서버 가져오기 시 올바른 설정으로 복원됩니다. {/* min-version: 2.1.198 */}v2.1.198부터 `env` 블록의 전송, API 라우팅 및 인증 환경 변수는 [서버가 페이로드를 확인을 할 때까지 보류됩니다](#가져오기-및-캐싱-동작). |
| 사용자가 캐시된 설정 파일을 삭제함 | 첫 실행 동작이 발생합니다. 설정을 비동기로 가져오며 짧은 미강제 인터벌이 존재합니다. |
| 사용자가 수정된 Claude Code 바이너리를 실행함 | 수정된 클라이언트를 실행할 수 있는 사용자는 클라이언트 측 제어를 우회할 수 있습니다. |
| 사용자가 이전 Claude Code 버전을 실행함 | 서버 관리형 설정보다 오래된 버전은 설정을 가져오거나 적용하지 않습니다. |
| API를 사용할 수 없음 | 사용 가능한 경우 캐시된 설정이 적용되고, 그렇지 않으면 다음 성공적인 가져오기 전까지 관리형 설정이 강제 적용되지 않습니다. {/* min-version: 2.1.198 */}v2.1.198부터 캐시된 `env` 블록의 전송, API 라우팅 및 인증 환경 변수는 [가져오기 실패 시 보류됩니다](#가져오기-및-캐싱-동작). 나머지 캐시는 여전히 적용됩니다. `forceRemoteSettingsRefresh: true`를 사용하면 [`claude auth` 하위 명령](#실패-시-종료-시작-강제-적용-enforce-fail-closed-startup)을 제외하고 CLI가 진행되지 않고 종료됩니다. |
| 사용자가 다른 조직으로 인증함 | 관리되는 조직 외부 계정에는 설정이 전달되지 않습니다. |
| 사용자가 [서드파티 모델 제공업체](#플랫폼-지원-여부)를 구성함 | 서버 관리형 설정이 우회됩니다. 여기에는 `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_MANTLE`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY`, `CLAUDE_CODE_USE_ANTHROPIC_AWS` 또는 기본값이 아닌 `ANTHROPIC_BASE_URL` 설정이 포함됩니다. |
| 네트워크 트래픽이 차단되거나 리디렉션됨 | 비활성화된 TLS 검증 또는 가로채인 트래픽은 클라이언트가 수신하는 설정을 변경할 수 있습니다. |

런타임 구성 변경을 감지하려면 [`ConfigChange` 훅](/docs/en/hooks#configchange)을 사용하여 변경 사항을 로깅하거나 무단 변경이 적용되기 전에 차단하세요.

클라이언트가 제공하는 자격 증명으로 사용자가 액세스할 수 있는 조직을 제한하려면 Claude 도움말 센터의 [테넌트 제한으로 네트워크 수준 액세스 제어 강제 적용](https://support.claude.com/en/articles/13198485-enforce-network-level-access-control-with-tenant-restrictions)을 참조하세요. 더 강력한 강제 보장을 위해 MDM 솔루션에 등록된 디바이스에서 [엔드포인트 관리형 설정](/docs/en/settings#settings-files)을 사용하세요.

## 관련 항목

Claude Code 구성을 관리하기 위한 관련 페이지:

* [설정](/docs/en/settings): 사용 가능한 모든 설정을 포함한 전체 구성 참조
* [엔드포인트 관리형 설정](/docs/en/settings#settings-files): IT 부서에서 디바이스에 배포하는 관리형 설정
* [인증](/docs/en/authentication): Claude Code에 대한 사용자 액세스 설정
* [보안](/docs/en/security): 보안 안전장치 및 모범 사례
