> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# GitHub Enterprise Server에서 Claude Code 사용하기

> 웹 세션, 코드 리뷰 및 플러그인 마켓플레이스를 위해 자체 호스팅 GitHub Enterprise Server 인스턴스에 Claude Code를 연결하세요.

<Note>
  GitHub Enterprise Server 지원은 Team 및 Enterprise 요금제에서 사용할 수 있습니다.
</Note>

GitHub Enterprise Server(GHES) 지원을 통해 조직은 github.com 대신 자체 관리형 GitHub 인스턴스에 호스팅된 리포지토리에서 Claude Code를 사용할 수 있습니다. 소유자(Owner)가 GHES 인스턴스를 연결하면 개발자는 리포지토리별 추가 구성 없이 웹 세션을 실행하고 자동화된 코드 리뷰를 받을 수 있습니다. 해당 인스턴스에 호스팅된 플러그인 마켓플레이스도 지원됩니다. 자격 증명 요구 사항은 [GHES상의 플러그인 마켓플레이스](#plugin-marketplaces-on-ghes)에 설명된 대로 영역마다 다릅니다.

github.com에 있는 리포지토리는 [Claude Code on the web](/docs/en/claude-code-on-the-web) 및 [Code Review](/docs/en/code-review)를 참조하세요. 자체 CI 인프라에서 Claude를 실행하려면 [GitHub Actions](/docs/en/github-actions)를 참조하세요.

## GitHub Enterprise Server에서 지원되는 기능

아래 표는 GHES를 지원하는 Claude Code 기능과 github.com 동작과의 차이점을 보여줍니다.

| 기능 | GHES 지원 여부 | 비고 |
| :--- | :--- | :--- |
| Claude Code on the web | ✅ 지원됨 | 소유자가 GHES 인스턴스를 한 번 연결하면 개발자는 평소처럼 `claude --cloud`나 [claude.ai/code](https://claude.ai/code)를 사용함 |
| Code Review | ✅ 지원됨 | github.com과 동일한 자동화된 PR 리뷰 |
| Claude Security | ✅ 지원됨 | Enterprise 요금제 사용자를 위해 [claude.ai/security](https://claude.ai/security)에서 퍼블릭 베타로 제공됨 |
| Teleport 세션 | ✅ 지원됨 | `--teleport`를 통해 웹과 터미널 간 세션 이동 |
| 플러그인 마켓플레이스 | ✅ 지원됨 | 자격 증명 요구 사항이 영역별로 다름. [GHES상의 플러그인 마켓플레이스](#plugin-marketplaces-on-ghes) 참조 |
| 기여 메트릭 | ✅ 지원됨 | 웹훅을 통해 [분석 대시보드](/docs/en/analytics)로 전달됨 |
| GitHub Actions | ✅ 지원됨 | 수동 워크플로우 설정 필요; `/install-github-app`은 github.com 전용 |
| GitHub MCP 서버 | ❌ 미지원 | GitHub MCP 서버는 GHES 인스턴스에서 작동하지 않음 |

## 관리자 설정

소유자가 Claude Code에 GHES 인스턴스를 한 번 연결합니다. 이후 조직의 개발자는 별도의 추가 구성 없이 GHES 리포지토리를 사용할 수 있습니다. Claude 조직의 Owner 또는 Primary Owner 역할과 GHES 인스턴스에 GitHub App을 생성할 수 있는 권한이 필요합니다.

가이드 설정 프로세스는 GitHub App 매니페스트를 생성하며, 한 번의 클릭으로 앱을 생성할 수 있도록 GHES 인스턴스로 리디렉션합니다. 환경이 리디렉션 플로우를 차단하는 경우 [대안 수동 설정](#manual-setup)을 사용할 수 있습니다.

<Steps>
  <Step title="Claude Code 관리자 설정 열기">
    [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)로 이동하여 GitHub Enterprise Server 섹션을 찾습니다.
  </Step>

  <Step title="가이드 설정 시작">
    **Connect**를 클릭합니다. 연결의 표시 이름과 GHES 호스트 이름(예: `github.example.com`)을 입력합니다. GHES 인스턴스가 자체 서명된 인증서나 사설 인증 기관(CA)을 사용하는 경우 선택 필드에 CA 인증서를 붙여넣습니다.
  </Step>

  <Step title="GitHub App 생성">
    **Continue to GitHub Enterprise**를 클릭합니다. 브라우저가 사전 입력된 앱 매니페스트와 함께 GHES 인스턴스로 리디렉션됩니다. 구성을 검토하고 **Create GitHub App**을 클릭합니다. GHES가 앱 자격 증명을 자동으로 저장하면서 Claude로 다시 리디렉션합니다.
  </Step>

  <Step title="리포지토리에 앱 설치">
    GHES 인스턴스의 GitHub App 페이지에서 Claude가 접근하길 원하는 리포지토리나 조직에 앱을 설치합니다. 일부로 시작하여 나중에 추가할 수 있습니다.
  </Step>

  <Step title="기능 활성화">
    [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)로 돌아가 github.com과 동일한 구성을 사용하여 GHES 리포지토리에 대해 [Code Review](/docs/en/code-review#set-up-code-review), Claude Security, [기여 메트릭](/docs/en/analytics#enable-contribution-metrics)을 활성화합니다.
  </Step>
</Steps>

### GitHub App 권한

매니페스트는 웹 세션, Code Review, Claude Security 및 기여 메트릭 전반에 걸쳐 Claude가 필요로 하는 권한과 웹훅 이벤트를 갖추도록 GitHub App을 구성합니다.

| 권한 | 접근 권한 | 용도 |
| :--- | :--- | :--- |
| Contents | Read and write | 리포지토리 클론 및 브랜치 푸시 |
| Pull requests | Read and write | PR 생성 및 리뷰 주석 포스팅 |
| Issues | Read and write | 이슈 멘션에 응답 |
| Checks | Read and write | Code Review 체크 실행 포스팅 |
| Actions | Read | 자동 수정을 위한 CI 상태 읽기 |
| Repository hooks | Read and write | 기여 메트릭용 웹훅 수신 |
| Metadata | Read | 모든 앱에 대해 GitHub이 요구하는 권한 |

앱은 `pull_request`, `issue_comment`, `pull_request_review_comment`, `pull_request_review`, `check_run` 이벤트를 구독합니다.

### 수동 설정

네트워크 구성으로 인해 가이드 리디렉션 플로우가 차단된 경우 Connect 대신 **Add manually**를 클릭하세요. [위의 권한 및 이벤트](#github-app-permissions)를 갖춘 GitHub App을 GHES 인스턴스에 생성한 후 양식에 앱 자격 증명(호스트 이름, OAuth 클라이언트 ID 및 시크릿, GitHub App ID, 클라이언트 ID, 클라이언트 시크릿, 웹훅 시크릿, 개인 키)을 입력하세요.

### 네트워크 요구 사항

Claude가 리포지토리를 클론하고 리뷰 주석을 포스팅할 수 있도록 GHES 인스턴스가 Anthropic 인프라에서 도달 가능해야 합니다. GHES 인스턴스가 방화벽 뒤에 있는 경우 [Anthropic API IP 주소](https://platform.claude.com/docs/en/api/ip-addresses)를 허용 목록에 추가하세요.

## 개발자 워크플로우

소유자가 GHES 인스턴스를 연결하면 개발자 측 구성이 필요하지 않습니다. Claude Code는 작업 디렉토리의 git 원격(remote)에서 GHES 호스트 이름을 자동으로 감지합니다.

`github.example.com`과 리포지토리 경로를 본인의 GHES 호스트 이름과 리포지토리로 교체하여 평소처럼 GHES 인스턴스에서 리포지토리를 클론하세요.

```bash theme={null}
git clone git@github.example.com:platform/api-service.git
cd api-service
```

그런 다음 웹 세션을 시작합니다. Claude가 git 원격에서 GHES 호스트를 감지하고 조직에 구성된 인스턴스를 통해 세션을 라우팅합니다.

```bash theme={null}
claude --cloud "Add retry logic to the payment webhook handler"
```

세션은 Anthropic 인프라에서 실행되고 GHES에서 리포지토리를 클론하며 변경 사항을 브랜치로 다시 푸시합니다. `/tasks` 또는 [claude.ai/code](https://claude.ai/code)에서 진행 상황을 모니터링하세요. diff 검토, 자동 수정 및 루틴을 포함한 전체 클라우드 세션 워크플로우는 [Claude Code on the web](/docs/en/claude-code-on-the-web)을 참조하세요.

### 세션을 터미널로 텔레포트(Teleport)하기

`claude --teleport`를 사용하여 웹 세션을 로컬 터미널로 가져오세요. Teleport는 브랜치를 가죠오고 세션 기록을 로드하기 전에 동일한 GHES 리포지토리가 체크아웃되어 있는지 확인합니다. 세부 정보는 [teleport 요구 사항](/docs/en/claude-code-on-the-web#teleport-requirements)을 참조하세요.

## Plugin marketplaces on GHES

GHES 인스턴스에 플러그인 마켓플레이스를 호스팅하여 조직 전체에 내부 툴링을 배포하세요. 마켓플레이스 구조는 github.com 호스팅 마켓플레이스와 동일하지만, 어디에 마켓플레이스를 추가하는지에 따라 설치 작동 방식이 달라지며 자격 증명 요구 사항도 영역마다 다릅니다.

| 영역 | 설치 작동 방식 | 각 사용자에게 필요한 사항 |
| :--- | :--- | :--- |
| Claude Code CLI 및 데스크톱 | Claude Code가 머신의 기존 git 자격 증명을 사용하여 마켓플레이스 리포지토리를 클론함 | 해당 머신에서 GHES 호스트로의 Git 접근 권한 |
| 관리 대상 설정 (`extraKnownMarketplaces`) | Claude Code가 항목을 등록하고 머신의 기존 git 자격 증명을 사용하여 리포지토리를 클론함 | 해당 머신에서 GHES 호스트로의 Git 접근 권한 |
| claude.ai 조직 플러그인 설정 | 소유자가 GHES 인스턴스를 소스로 선택함; Anthropic 백엔드가 [관리자 설정](#admin-setup)의 GitHub App을 사용하여 리포지토리를 가져오고 동기화함 | 추가된 후에는 사용자별 요구 사항 없음. 마켓플레이스를 추가하는 소유자는 접근 확인을 위해 자체 GitHub Enterprise 계정이 연결되어 있어야 하며, 마켓플레이스 리포지토리에 GitHub App이 설치되어 있어야 함 |
| claude.ai 사용자 설정 | Anthropic 백엔드가 제출 사용자의 GitHub Enterprise 연결을 사용하여 리포지토리를 가져옴 | Claude에 연결된 본인의 GitHub Enterprise 계정 |
| Claude Code on the web | 클라우드 세션이 세션 샌드박스 내부에서 마켓플레이스를 클론함. 샌드박스는 세션의 리포지토리가 동일한 인스턴스에 있을 때만 GHES 인스턴스에 접근할 수 있으며 git 자격 증명은 세션의 리포지토리로 범위가 한정됨 | GHES 호스팅 마켓플레이스에는 신뢰할 수 없음: 세션 리포지토리와 다른 호스트는 도달할 수 없으며 동일 인스턴스 설치도 실패할 수 있음. 대신 CLI, 관리 대상 설정 또는 claude.ai를 사용하세요 |

<Warning>
  사용자 설정에서 마켓플레이스를 추가할 때 claude.ai의 GitHub Enterprise 연결은 사용자별로 적용됩니다. [관리자 설정](#admin-setup)은 GHES 인스턴스를 조직에 연결하지만 개별 사용자 계정을 연결하지는 않습니다: 본인의 설정에서 GHES 마켓플레이스를 추가하는 각 사용자는 먼저 자체 GitHub Enterprise 계정을 연결해야 하며, 소유자를 포함한 한 사용자의 연결이 다른 사용자를 커버하지 않습니다. 조직 플러그인 설정에서 소유자가 추가한 마켓플레이스는 지속적인 패치에 조직의 GitHub App을 사용하므로 사용자에게 이 요구 사항을 적용하지 않습니다. 마켓플레이스를 추가하는 소유자는 추가 시점에 본인의 GitHub Enterprise 계정이 연결되어 있어야 합니다.
</Warning>

### GHES 마켓플레이스 추가하기

`owner/repo` 축약어는 항상 github.com으로 확인됩니다. GHES 호스팅 마켓플레이스의 경우 `github.example.com`과 리포지토리 경로를 본인의 경로로 교체하여 전체 git URL을 사용하세요. HTTPS URL을 권장합니다.

```bash theme={null}
/plugin marketplace add https://github.example.com/platform/claude-plugins.git
```

머신이 이미 GHES 호스트를 신뢰하는 경우 SSH URL이 작동합니다.

```bash theme={null}
/plugin marketplace add git@github.example.com:platform/claude-plugins.git
```

Claude Code는 비대화형으로 git을 실행하며 머신의 `known_hosts` 파일에 없는 호스트에 대한 SSH 연결을 거부합니다. git 자격 증명 헬퍼와 함께 HTTPS URL을 사용하면 `known_hosts` 요구 사항을 피할 수 있습니다.

마켓플레이스 구축에 대한 전체 가이드는 [플러그인 마켓플레이스 생성 및 배포](/docs/en/plugin-marketplaces)를 참조하세요.

### 관리 대상 설정으로 GHES 마켓플레이스 사전 등록하기

`extraKnownMarketplaces` 설정은 마켓플레이스를 사전 등록하여 개발자가 수동 설정 없이 받을 수 있게 합니다. 리포지토리의 `.claude/settings.json`을 포함하여 [모든 설정 파일](/docs/en/settings#extraknownmarketplaces)에서 작동하며, 관리 대상 설정은 조직 전체에 배포합니다.

```json theme={null}
{
  "extraKnownMarketplaces": {
    "internal-tools": {
      "source": {
        "source": "git",
        "url": "https://github.example.com/platform/claude-plugins.git"
      }
    }
  }
}
```

Claude Code는 이러한 마켓플레이스를 로컬에 설치합니다: 각 항목을 등록하고 머신의 기존 git 자격 증명으로 리포지토리를 클론합니다. 이 경로는 claude.ai를 거치지 않으므로 사용자별 GitHub Enterprise 연결이 필요하지 않습니다. 성공적인 배포를 위해:

* **전체 git URL을 사용하세요.** `owner/repo` 축약어는 항상 github.com으로 확인되며 GHES 호스트를 참조할 수 없습니다.
* **HTTPS URL을 선호하세요.** SSH 클론은 GHES 호스트 키를 아직 신뢰하지 않는 머신에서 실패합니다. 조직의 표준 git 자격 증명 헬퍼가 적용된 HTTPS URL은 자격 증명이 구성된 모든 머신에서 작동합니다.
* **각 머신이 GHES 호스트에서 클론할 수 있는지 확인하세요.** 머신에 자격 증명이 없으면 마켓플레이스가 등록되지만 설치되지 않으며 자격 증명을 묻는 대신 해당 플러그인을 찾을 수 없다고 보고합니다.
* **설정이 각 머신에 도달하는지 확인하세요.** 관리 대상 설정 파일은 기기 관리 시스템 등을 통해 배포된 머신에만 적용됩니다. 파일 위치는 [관리 대상 설정](/docs/en/settings#settings-files)을 참조하세요.

### 관리 대상 설정에서 GHES 마켓플레이스 허용 목록 작성하기

조직이 [관리 대상 설정](/docs/en/settings)을 사용하여 개발자가 추가할 수 있는 마켓플레이스를 제한하는 경우 `hostPattern` 소스 유형을 사용하여 각 리포지토리를 열거하지 않고 GHES 인스턴스의 모든 마켓플레이스를 허용할 수 있습니다.

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

전체 스키마는 [strictKnownMarketplaces](/docs/en/settings#strictknownmarketplaces) 및 [extraKnownMarketplaces](/docs/en/settings#extraknownmarketplaces) 설정 레퍼런스를 참조하세요.

## 제한 사항

몇 가지 기능은 GHES에서 github.com과 다르게 작동합니다. [기능 표](#what-works-with-github-enterprise-server)에 요약되어 있으며 이 섹션에서는 우회 방법을 다룹니다.

* **`/install-github-app` 명령**: claude.ai의 [관리자 설정](#admin-setup) 플로우를 대신 따르세요. GHES에서 GitHub Actions 워크플로우를 사용하려는 경우 [예시 워크플로우](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml)를 수동으로 조정하세요.
* **GitHub MCP 서버**: GHES 호스트용으로 구성된 `gh` CLI를 대신 사용하세요. `gh auth login --hostname github.example.com`을 실행하여 인증하면 Claude가 세션에서 `gh` 명령을 사용할 수 있습니다.

## 문제 해결

### 웹 세션의 리포지토리 복제 실패

`claude --cloud`가 클론 오류로 실패하면 소유자가 GHES 인스턴스 설정을 완료했는지, 작업 중인 리포지토리에 GitHub App이 설치되어 있는지 확인하세요. 인스턴스를 연결한 소유자에게 Claude 설정에 등록된 호스트 이름이 git 원격의 호스트 이름과 일치하는지 확인하도록 요청하세요.

### 마켓플레이스 추가 시 정책 오류 발생

GHES URL에 대해 `/plugin marketplace add`가 차단되는 경우 조직에서 마켓플레이스 소스를 제한한 것입니다. 관리자에게 [관리 대상 설정](#allowlist-ghes-marketplaces-in-managed-settings)에서 GHES 호스트 이름에 대한 `hostPattern` 항목을 추가하도록 요청하세요.

### claude.ai에서 마켓플레이스 추가 시 GitHub 접근 오류 발생

사용자 설정에서 GHES 마켓플레이스를 추가할 때 "Marketplace couldn't be added"와 같은 일반적인 오류로 실패하는 경우 먼저 GitHub Enterprise 연결을 확인하세요. 이는 조직의 GHES 인스턴스가 구성되어 있고 다른 사용자가 연결되어 있더라도 본인의 GitHub Enterprise 계정이 Claude에 연결되어 있지 않을 때 나타납니다. 다이얼로그가 GitHub Enterprise 연결 플로우를 안내하지 않으며 Browse 탭의 "Connect to GitHub" 옵션은 GHES 리포지토리에 대한 접근 권한을 부여하지 않는 github.com에 로그인합니다.

GitHub Enterprise 계정을 연결하려면: [claude.ai/code](https://claude.ai/code)의 리포지토리 선택기가 구성된 각 GHES 인스턴스에 대한 연결 옵션을 제공하며, 소유자는 [Claude Code 관리자 설정](https://claude.ai/admin-settings/claude-code)의 GitHub Enterprise 섹션에서도 연결할 수 있습니다. 그 후 마켓플레이스를 다시 추가하세요. 대안으로 소유자에게 조직 플러그인 설정에 마켓플레이스를 추가하도록 요청하면 사용자별 연결 요구 사항이 제거됩니다.

다른 claude.ai 영역에서 GHES 마켓플레이스에 대한 "Repository not found. If it's private, GitHub access is required" 오류는 일반적으로 동일하게 누락된 연결을 나타냅니다. 위의 경로 중 하나를 통해 GitHub Enterprise 계정을 연결한 후 다시 시도하세요.

### GHES 인스턴스에 접근할 수 없음

리뷰나 웹 세션이 타임아웃되면 Anthropic 인프라에서 GHES 인스턴스에 도달할 수 없는 것일 수 있습니다. 방화벽이 [Anthropic API IP 주소](https://platform.claude.com/docs/en/api/ip-addresses)에서의 인바운드 연결을 허용하는지 확인하세요.

### `Unable to get organization UUID`로 세션 시작 실패

웹 세션에는 Team 또는 Enterprise 조직이 필요합니다. 조직 계정으로 `/login`을 진행하세요. 대신 API 키로 인증하면 웹 세션이 `/login`을 실행하라는 메시지와 함께 더 일찍 실패합니다.

## 관련 리소스

이 가이드 전체에서 참조된 기능에 대해 더 자세히 다루는 페이지입니다.

* [Claude Code on the web](/docs/en/claude-code-on-the-web): 클라우드 인프라에서 Claude Code 세션 실행
* [Code Review](/docs/en/code-review): 자동화된 PR 리뷰
* [플러그인 마켓플레이스](/docs/en/plugin-marketplaces): 플러그인 카탈로그 구축 및 배포
* [분석](/docs/en/analytics): 사용량 및 기여 메트릭 추적
* [관리 대상 설정](/docs/en/settings): 조직 전반의 정책 구성
* [네트워크 구성](/docs/en/network-config): 방화벽 및 IP 허용 목록 요구 사항
