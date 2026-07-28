> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 엔터프라이즈 배포 개요

> 기업 배포 요구 사항을 충족하기 위해 Claude Code가 다양한 서드파티 서비스 및 인프라와 통합되는 방법에 대해 알아보세요.

export const ContactSalesCard = ({surface}) => {
  const utm = content => `utm_source=claude_code&utm_medium=docs&utm_content=${surface}_${content}`;
  const iconArrowRight = (size = 13) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round" aria-hidden="true">
      <line x1="5" y1="12" x2="19" y2="12" />
      <polyline points="12 5 19 12 12 19" />
    </svg>;
  const STYLES = `
.cc-cs {
  --cs-slate: #141413;
  --cs-clay: #d97757;
  --cs-clay-deep: #c6613f;
  --cs-gray-000: #ffffff;
  --cs-gray-700: #3d3d3a;
  --cs-border-default: rgba(31, 30, 29, 0.15);
  font-family: inherit;
}
.dark .cc-cs {
  --cs-slate: #f0eee6;
  --cs-gray-000: #262624;
  --cs-gray-700: #bfbdb4;
  --cs-border-default: rgba(240, 238, 230, 0.14);
}
.cc-cs-card {
  display: flex; align-items: center; justify-content: space-between;
  gap: 16px; padding: 14px 16px; margin: 0;
  background: var(--cs-gray-000); border: 0.5px solid var(--cs-border-default);
  border-radius: 8px; flex-wrap: wrap;
}
.cc-cs-text { font-size: 13px; color: var(--cs-gray-700); line-height: 1.5; flex: 1; min-width: 240px; }
.cc-cs-text strong { font-weight: 550; color: var(--cs-slate); }
.cc-cs-actions { display: flex; align-items: center; gap: 8px; flex-shrink: 0; }
.cc-cs-btn-clay {
  display: inline-flex; align-items: center; gap: 8px;
  background: var(--cs-clay-deep); color: #fff; border: none;
  border-radius: 8px; padding: 8px 14px;
  font-size: 13px; font-weight: 500;
  transition: background-color 0.15s; white-space: nowrap;
}
.cc-cs-btn-clay:hover { background: var(--cs-clay); }
.cc-cs-btn-ghost {
  display: inline-flex; align-items: center; gap: 8px;
  background: transparent; color: var(--cs-gray-700);
  border: 0.5px solid var(--cs-border-default);
  border-radius: 8px; padding: 8px 14px;
  font-size: 13px; font-weight: 500;
}
.cc-cs-btn-ghost:hover { background: rgba(0, 0, 0, 0.04); }
.dark .cc-cs-btn-ghost:hover { background: rgba(255, 255, 255, 0.04); }
@media (max-width: 720px) {
  .cc-cs-actions { width: 100%; }
}
`;
  return <div className="cc-cs not-prose">
      <style>{STYLES}</style>
      <div className="cc-cs-card">
        <div className="cc-cs-text">
          <strong>조직 전체에 Claude Code를 배포하려고 하시나요?</strong> 엔터프라이즈 플랜, SSO 및 중앙 집중식 청구에 대해 영업팀에 문의하세요.
        </div>
        <div className="cc-cs-actions">
          <a href={`https://claude.com/pricing?${utm('view_plans')}#plans-business`} className="cc-cs-btn-ghost">
            플랜 보기
          </a>
          <a href={`https://claude.com/contact-sales?${utm('contact_sales')}`} className="cc-cs-btn-clay">
            영업팀 문의 {iconArrowRight()}
          </a>
        </div>
      </div>
    </div>;
};

조직은 Anthropic을 통해 직접 또는 클라우드 제공업체를 통해 Claude Code를 배포할 수 있습니다. 이 페이지는 적절한 구성을 선택하도록 돕습니다.

<ContactSalesCard surface="third_party_overview" />

## 배포 옵션 비교

대부분의 조직에는 Claude for Teams 또는 Claude for Enterprise가 가장 좋은 경험을 제공합니다. 팀원은 단일 구독, 중앙 집중식 청구 및 인프라 설정 없이 Claude Code 및 웹의 Claude 모두에 접근할 수 있습니다.

**Claude for Teams**는 셀프 서비스 방식이며 협업 기능, 관리자 도구 및 청구 관리를 포함합니다. 빠르게 시작해야 하는 소규모 팀에 가장 적합합니다.

**Claude for Enterprise**는 SSO 및 도메인 캡처, 역할 기반 권한, 컴플라이언스 API 접근, 그리고 조직 전체에 Claude Code 구성을 배포하기 위한 관리형 정책 설정을 추가합니다. 보안 및 컴플라이언스 요구 사항이 있는 대규모 조직에 가장 적합합니다.

[Team plans](https://support.claude.com/en/articles/9266767-what-is-the-team-plan) 및 [Enterprise plans](https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan)에 대해 자세히 알아보세요.

조직에 특정 인프라 요구 사항이 있는 경우 아래 옵션을 비교하세요:

<table>
  <thead>
    <tr>
      <th>기능</th>
      <th>Claude for Teams/Enterprise</th>
      <th>Anthropic Console</th>
      <th>Amazon Bedrock</th>
      <th>Claude Platform on AWS</th>
      <th>Google Cloud's Agent Platform (구 Vertex AI)</th>
      <th>Microsoft Foundry</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>적합한 대상</td>
      <td>대부분의 조직 (권장)</td>
      <td>개별 개발자</td>
      <td>AWS 네이티브 배포</td>
      <td>Claude API 기능을 갖춘 AWS Marketplace 청구</td>
      <td>GCP 네이티브 배포</td>
      <td>Azure 네이티브 배포</td>
    </tr>

    <tr>
      <td>청구</td>
      <td><strong>Teams:</strong> \$150/석 (Premium, 종량제 사용 가능)<br /><strong>Enterprise:</strong> <a href="https://claude.com/contact-sales?utm_source=claude_code&utm_medium=docs&utm_content=third_party_enterprise">영업팀 문의</a></td>
      <td>종량제 (PAYG)</td>
      <td>AWS를 통한 종량제</td>
      <td>AWS Marketplace를 통한 종량제</td>
      <td>GCP를 통한 종량제</td>
      <td>Azure를 통한 종량제</td>
    </tr>

    <tr>
      <td>리전</td>
      <td>지원되는 [국가](https://www.anthropic.com/supported-countries)</td>
      <td>지원되는 [국가](https://www.anthropic.com/supported-countries)</td>
      <td>다중 AWS [리전](https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html)</td>
      <td>다중 AWS 리전</td>
      <td>다중 GCP [리전](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/locations)</td>
      <td>다중 Azure [리전](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/)</td>
    </tr>

    <tr>
      <td>프롬프트 캐싱</td>
      <td>기본적으로 활성화됨</td>
      <td>기본적으로 활성화됨</td>
      <td>기본적으로 활성화됨</td>
      <td>기본적으로 활성화됨</td>
      <td>기본적으로 활성화됨</td>
      <td>기본적으로 활성화됨</td>
    </tr>

    <tr>
      <td>인증</td>
      <td>Claude.ai SSO 또는 이메일</td>
      <td>API 키</td>
      <td>API 키 또는 AWS 자격 증명</td>
      <td>API 키 또는 AWS 자격 증명</td>
      <td>GCP 자격 증명</td>
      <td>API 키 또는 Microsoft Entra ID</td>
    </tr>

    <tr>
      <td>비용 추적</td>
      <td>사용량 대시보드</td>
      <td>사용량 대시보드</td>
      <td>AWS Cost Explorer</td>
      <td>AWS Cost Explorer</td>
      <td>GCP Billing</td>
      <td>Azure Cost Management</td>
    </tr>

    <tr>
      <td>웹 버전 Claude 포함</td>
      <td>예</td>
      <td>아니오</td>
      <td>아니오</td>
      <td>아니오</td>
      <td>아니오</td>
      <td>아니오</td>
    </tr>

    <tr>
      <td>엔터프라이즈 기능</td>
      <td>팀 관리, SSO, 사용량 모니터링</td>
      <td>없음</td>
      <td>IAM 정책, CloudTrail</td>
      <td>IAM 정책, CloudTrail</td>
      <td>IAM 역할, Cloud Audit Logs</td>
      <td>RBAC 정책, Azure Monitor</td>
    </tr>
  </tbody>
</table>

각 옵션에서 사용할 수 있는 기능별 분석은 [Feature availability](/docs/en/feature-availability)를 참조하세요.

설정 지침을 보려면 배포 옵션을 선택하세요:

* [Claude for Teams or Enterprise](/docs/en/authentication#claude-for-teams-or-enterprise)
* [Anthropic Console](/docs/en/authentication#claude-console-authentication)
* [Claude apps gateway](/docs/en/claude-apps-gateway): Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform, Microsoft Foundry 또는 Anthropic API 앞에 IdP 로그인을 추가하는 자체 호스팅 게이트웨이
* [Amazon Bedrock](/docs/en/amazon-bedrock)
* [Claude Platform on AWS](/docs/en/claude-platform-on-aws)
* [Google Cloud's Agent Platform](/docs/en/google-vertex-ai)
* [Microsoft Foundry](/docs/en/microsoft-foundry)

Amazon Bedrock 및 Google Vertex AI의 경우 로그인 프롬프트에서 `claude`를 실행하고 **3rd-party platform**을 선택하여 대화형 설정 마법사를 시작할 수도 있습니다.

## 프록시 및 게이트웨이 구성

대부분의 조직은 추가 구성 없이 클라우드 제공업체를 직접 사용할 수 있습니다. 그러나 조직에 특정 네트워크 또는 관리 요구 사항이 있는 경우 기업 프록시 또는 LLM 게이트웨이를 구성해야 할 수 있습니다. 이들은 함께 사용할 수 있는 다른 구성입니다:

* **Corporate proxy (기업 프록시)**: HTTP/HTTPS 프록시를 통해 트래픽을 라우팅합니다. 보안 모니터링, 컴플라이언스 또는 네트워크 정책 강제를 위해 모든 아웃바운드 트래픽이 프록시 서버를 통과해야 하는 조직의 경우 이를 사용하세요. `HTTPS_PROXY` 또는 `HTTP_PROXY` 환경 변수로 구성합니다. 자세한 내용은 [Enterprise network configuration](/docs/en/network-config)을 참조하세요.
* **LLM Gateway (LLM 게이트웨이)**: 인증 및 라우팅을 처리하기 위해 Claude Code와 클라우드 제공업체 사이에 위치하는 서비스입니다. 팀 전반의 중앙 집중식 사용량 추적, 커스텀 속도 제한 또는 예산, 중앙 집중식 인증 관리가 필요한 경우 이를 사용하세요. `ANTHROPIC_BASE_URL`, `ANTHROPIC_BEDROCK_BASE_URL`, `ANTHROPIC_AWS_BASE_URL`, `ANTHROPIC_VERTEX_BASE_URL` 또는 `ANTHROPIC_FOUNDRY_BASE_URL` 환경 변수로 구성합니다. 자세한 내용은 [LLM gateways](/docs/en/llm-gateway)를 참조하세요.

다음 예시는 쉘 또는 쉘 프로필(`.bashrc`, `.zshrc`)에 설정할 환경 변수를 보여줍니다. 다른 구성 방법은 [Settings](/docs/en/settings)를 참조하세요.

### Amazon Bedrock

<Tabs>
  <Tab title="Corporate proxy">
    다음 [환경 변수](/docs/en/env-vars)를 설정하여 기업 프록시를 통해 Amazon Bedrock 트래픽을 라우팅합니다:

    ```bash theme={null}
    # Enable Bedrock
    export CLAUDE_CODE_USE_BEDROCK=1
    export AWS_REGION=us-east-1

    # Configure corporate proxy
    export HTTPS_PROXY='https://proxy.example.com:8080'
    ```
  </Tab>

  <Tab title="LLM Gateway">
    다음 [환경 변수](/docs/en/env-vars)를 설정하여 LLM 게이트웨이를 통해 Amazon Bedrock 트래픽을 라우팅합니다:

    ```bash theme={null}
    # Enable Bedrock
    export CLAUDE_CODE_USE_BEDROCK=1

    # Configure LLM gateway
    export ANTHROPIC_BEDROCK_BASE_URL='https://your-llm-gateway.com/bedrock'
    export CLAUDE_CODE_SKIP_BEDROCK_AUTH=1  # If gateway handles AWS auth
    ```
  </Tab>
</Tabs>

### Microsoft Foundry

<Tabs>
  <Tab title="Corporate proxy">
    다음 [환경 변수](/docs/en/env-vars)를 설정하여 기업 프록시를 통해 Microsoft Foundry 트래픽을 라우팅합니다:

    ```bash theme={null}
    # Enable Microsoft Foundry
    export CLAUDE_CODE_USE_FOUNDRY=1
    export ANTHROPIC_FOUNDRY_RESOURCE=your-resource
    export ANTHROPIC_FOUNDRY_API_KEY=your-api-key  # Or omit for Entra ID auth

    # Configure corporate proxy
    export HTTPS_PROXY='https://proxy.example.com:8080'
    ```
  </Tab>

  <Tab title="LLM Gateway">
    다음 [환경 변수](/docs/en/env-vars)를 설정하여 LLM 게이트웨이를 통해 Microsoft Foundry 트래픽을 라우팅합니다:

    ```bash theme={null}
    # Enable Microsoft Foundry
    export CLAUDE_CODE_USE_FOUNDRY=1

    # Configure LLM gateway
    export ANTHROPIC_FOUNDRY_BASE_URL='https://your-llm-gateway.com'
    export ANTHROPIC_FOUNDRY_API_KEY=your-gateway-key  # Sent as x-api-key
    ```
  </Tab>
</Tabs>

### Google Cloud's Agent Platform

<Tabs>
  <Tab title="Corporate proxy">
    다음 [환경 변수](/docs/en/env-vars)를 설정하여 기업 프록시를 통해 Google Cloud's Agent Platform 트래픽을 라우팅합니다:

    ```bash theme={null}
    # Enable Agent Platform
    export CLAUDE_CODE_USE_VERTEX=1
    export CLOUD_ML_REGION=us-east5
    export ANTHROPIC_VERTEX_PROJECT_ID=your-project-id

    # Configure corporate proxy
    export HTTPS_PROXY='https://proxy.example.com:8080'
    ```
  </Tab>

  <Tab title="LLM Gateway">
    다음 [환경 변수](/docs/en/env-vars)를 설정하여 LLM 게이트웨이를 통해 Google Cloud's Agent Platform 트래픽을 라우팅합니다:

    ```bash theme={null}
    # Enable Agent Platform
    export CLAUDE_CODE_USE_VERTEX=1

    # Configure LLM gateway
    export ANTHROPIC_VERTEX_BASE_URL='https://your-llm-gateway.com/vertex'
    export CLAUDE_CODE_SKIP_VERTEX_AUTH=1  # If gateway handles GCP auth
    export ANTHROPIC_VERTEX_PROJECT_ID=your-gcp-project-id
    export CLOUD_ML_REGION=us-east5
    ```
  </Tab>
</Tabs>

<Tip>
  Claude Code에서 `/status`를 사용하여 프록시 및 게이트웨이 구성이 올바르게 적용되었는지 확인하세요. 예를 들어 위의 Bedrock 게이트웨이 구성을 사용하면 출력에 다음과 같은 줄이 포함됩니다:

  ```
  API provider: Amazon Bedrock
  Bedrock base URL: https://your-llm-gateway.com/bedrock
  AWS region: us-east-1
  AWS auth skipped
  ```

  기업 프록시를 구성한 경우 `/status`에는 프록시 URL이 포함된 `Proxy` 줄도 표시됩니다.
</Tip>

## 조직을 위한 모범 사례

### 문서 및 메모리에 투자

Claude Code가 코드베이스를 이해할 수 있도록 문서에 투자할 것을 강력히 권장합니다. 조직은 여러 수준에서 CLAUDE.md 파일을 배포할 수 있습니다:

* **조직 전체**: 전사 표준을 위해 `/Library/Application Support/ClaudeCode/CLAUDE.md`(macOS), `/etc/claude-code/CLAUDE.md`(Linux 및 WSL) 또는 `C:\Program Files\ClaudeCode\CLAUDE.md`(Windows)와 같은 시스템 디렉터리에 배포
* **리포지토리 수준**: 프로젝트 아키텍처, 빌드 명령 및 기여 지침이 포함된 `CLAUDE.md` 파일을 리포지토리 루트에 생성. 모든 사용자가 혜택을 볼 수 있도록 소스 제어에 커밋

자세한 내용은 [Memory and CLAUDE.md files](/docs/en/memory)를 참조하세요.

### 배포 단순화

커스텀 개발 환경이 있는 경우 Claude Code를 설치하는 "원클릭" 방법을 만드는 것이 조직 전체의 채택을 늘리는 핵심이라는 것을 확인했습니다.

### 가이드 기반 사용으로 시작

새로운 사용자가 코드베이스 Q\&A 또는 소규모 버그 수정/기능 요청에 Claude Code를 사용해 보도록 권장하세요. Claude Code에 플랜을 작성하도록 요청하세요. Claude의 제안을 확인하고 궤도를 벗어난 경우 피드백을 제공하세요. 시간이 지남에 따라 사용자가 이 새로운 패러다임을 더 잘 이해하게 되면 Claude Code가 더 자율적으로 실행되도록 하는 데 더욱 효과적이 될 것입니다.

### 클라우드 제공업체의 모델 버전 고정

[Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry) 또는 [Claude Platform on AWS](/docs/en/claude-platform-on-aws)를 통해 배포하는 경우 `ANTHROPIC_DEFAULT_FABLE_MODEL`, `ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL` 및 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 사용하여 특정 모델 버전을 고정하세요. 고정하지 않으면 모델 별칭이 해당 제공업체에 대한 Claude Code의 내장 기본값으로 해석되므로 최신 릴리스보다 뒤처질 수 있고 계정에서 아직 활성화되지 않았을 수 있습니다. 고정을 사용하면 사용자가 새 모델로 이동하는 시점을 제어할 수 있습니다. 기본값을 사용할 수 없을 때 각 제공업체가 수행하는 작업은 [Model configuration](/docs/en/model-config#pin-models-for-third-party-deployments)을 참조하세요.

### 보안 정책 구성

보안 팀은 Claude Code가 허용하거나 허용하지 않는 작업에 대해 로컬 구성으로 재정의할 수 없는 관리형 권한을 구성할 수 있습니다. [자세히 알아보기](/docs/en/security).

### 통합을 위해 MCP 활용

MCP는 티켓 관리 시스템이나 오류 로그에 연결하는 등 Claude Code에 더 많은 정보를 제공하는 훌륭한 방법입니다. 모든 사용자가 혜택을 볼 수 있도록 한 중앙 팀이 MCP 서버를 구성하고 `.mcp.json` 구성을 코드베이스에 커밋하는 것을 권장합니다. [자세히 알아보기](/docs/en/mcp).

Anthropic에서는 모든 Anthropic 코드베이스에 걸쳐 개발에 Claude Code를 사용하고 있습니다. 여러분도 저희만큼 Claude Code를 즐겁게 사용하시기를 바랍니다.

## 다음 단계

배포 옵션을 선택하고 팀의 접근 권한을 구성했다면:

1. **팀에 배포**: 설치 지침을 공유하고 팀원이 [Claude Code를 설치](/docs/en/setup)하고 자격 증명으로 인증하도록 합니다.
2. **공유 구성 설정**: Claude Code가 코드베이스 및 코딩 표준을 이해하는 데 도움이 되도록 리포지토리에 [CLAUDE.md 파일](/docs/en/memory)을 생성합니다.
3. **권한 구성**: [보안 설정](/docs/en/security)을 검토하여 해당 환경에서 Claude Code가 할 수 있는 작업과 할 수 없는 작업을 정의합니다.
