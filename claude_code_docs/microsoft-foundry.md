> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Microsoft Foundry에서의 Claude Code

> 설정, 구성, 문제 해결을 포함하여 Microsoft Foundry를 통한 Claude Code 구성 방법에 대해 알아봅니다.

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
          <strong>Deploying Claude Code across your organization?</strong> Talk to sales about enterprise plans, SSO, and centralized billing.
        </div>
        <div className="cc-cs-actions">
          <a href={`https://claude.com/pricing?${utm('view_plans')}#plans-business`} className="cc-cs-btn-ghost">
            View plans
          </a>
          <a href={`https://claude.com/contact-sales?${utm('contact_sales')}`} className="cc-cs-btn-clay">
            Contact sales {iconArrowRight()}
          </a>
        </div>
      </div>
    </div>;
};

<ContactSalesCard surface="foundry" />

## 사전 요구사항

Microsoft Foundry를 사용하여 Claude Code를 구성하기 전에 다음 항목이 준비되어 있는지 확인하세요:

* Microsoft Foundry에 액세스할 수 있는 Azure 구독
* Microsoft Foundry 리소스 및 배포를 생성할 수 있는 RBAC 권한
* 설치 및 구성된 Azure CLI (선택 사항 - 자격 증명을 가져오는 다른 메커니즘이 없는 경우에만 필요)

<Note>
  여러 사용자에게 Claude Code를 배포하는 경우 배포 전에 [모델 버전을 고정(pin)](#4-pin-model-versions)하세요.
</Note>

## 설정

### 1. Microsoft Foundry 리소스 생성

먼저 Azure에서 Claude 리소스를 생성합니다:

1. [Microsoft Foundry 포털](https://ai.azure.com/)로 이동합니다.
2. 새 리소스를 생성하고 리소스 이름을 기록해 둡니다.
3. Claude 모델에 대한 배포를 생성하고 각 배포에 부여한 이름을 기록해 둡니다. 단계 4에서 이 이름들을 모델 변수로 설정합니다:

   * Claude Opus
   * Claude Sonnet
   * Claude Haiku

   배포를 구성할 때 추론이 Azure에서 실행되는지 아니면 Anthropic 인프라에서 실행되는지를 결정하는 [호스팅 옵션](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry#hosting-options)도 함께 선택합니다.

### 2. Azure 자격 증명 구성

Claude Code는 Microsoft Foundry에 대해 세 가지 인증 방식을 지원합니다. 보안 요구사항에 가장 잘 맞는 방식을 선택하세요.

**옵션 A: API 키 인증**

1. Microsoft Foundry 포털에서 해당 리소스로 이동합니다.
2. **Endpoints and keys** 섹션으로 이동합니다.
3. **API Key**를 복사합니다.
4. 환경 변수를 설정하고 `your-azure-api-key`를 복사한 키로 변경합니다:

```bash theme={null}
export ANTHROPIC_FOUNDRY_API_KEY=your-azure-api-key
```

**옵션 B: Microsoft Entra ID 인증**

`ANTHROPIC_FOUNDRY_API_KEY`와 `ANTHROPIC_FOUNDRY_AUTH_TOKEN`이 모두 설정되어 있지 않은 경우, Claude Code는 자동으로 Azure SDK의 [기본 자격 증명 체인(default credential chain)](https://learn.microsoft.com/en-us/azure/developer/javascript/sdk/authentication/credential-chains#defaultazurecredential-overview)을 사용합니다.
이는 로컬 및 원격 워크로드를 인증하기 위한 다양한 방식을 지원합니다.

로컬 환경에서는 일반적으로 Azure CLI를 사용할 수 있습니다:

```bash theme={null}
az login
```

**옵션 C: Bearer 토큰 인증**

Claude Code는 모든 요청 시 `Authorization: Bearer` 헤더로 `ANTHROPIC_FOUNDRY_AUTH_TOKEN` 값을 보냅니다. 호스트 애플리케이션이나 로그인 스크립트 등 다른 프로세스가 이미 액세스 토큰을 획득한 경우 이 옵션을 사용하세요. Claude Code v2.1.203 이상이 필요합니다.

Microsoft Entra ID가 리소스에 대해 발급한 bearer 토큰으로 변수를 설정합니다:

```bash theme={null}
export ANTHROPIC_FOUNDRY_AUTH_TOKEN=your-entra-access-token
```

`ANTHROPIC_FOUNDRY_AUTH_TOKEN`은 `ANTHROPIC_FOUNDRY_API_KEY` 및 기본 자격 증명 체인보다 우선 적용됩니다.

<Note>
  Microsoft Foundry를 사용할 때는 인증이 Azure 자격 증명을 통해 처리되므로 `/logout` 명령을 이용할 수 없습니다.
</Note>

### 3. Claude Code 구성

Microsoft Foundry를 활성화하려면 다음 환경 변수들을 설정하세요:

```bash theme={null}
# Microsoft Foundry 통합 활성화
export CLAUDE_CODE_USE_FOUNDRY=1

# Azure 리소스 이름 ({resource}를 본인의 리소스 이름으로 대체)
export ANTHROPIC_FOUNDRY_RESOURCE={resource}
# 또는 전체 베이스 URL 제공:
# export ANTHROPIC_FOUNDRY_BASE_URL=https://{resource}.services.ai.azure.com/anthropic
```

### 4. 모델 버전 고정

<Warning>
  모든 배포에 대해 특정 모델 버전을 고정(pin)하세요. 고정하지 않으면 `sonnet` 및 `opus`와 같은 모델 별칭이 Microsoft Foundry용 Claude Code의 내장 기본값으로 해석되는데, 이는 최신 릴리스보다 뒤처질 수 있으며 계정에서 아직 사용할 수 없을 수도 있습니다. Microsoft Foundry는 시작 시 모델 검사가 없으므로 기본값을 사용할 수 없으면 요청이 실패합니다. Azure 배포를 생성할 때 "최신 버전으로 자동 업데이트" 대신 특정 모델 버전을 선택하세요.
</Warning>

단계 1에서 생성한 배포 이름과 일치하도록 모델 변수를 설정합니다.

`ANTHROPIC_DEFAULT_OPUS_MODEL`이 없으면 Microsoft Foundry의 `opus` 별칭이 Opus 4.6으로 해석됩니다. 최신 모델을 사용하려면 Opus 4.8 ID로 설정하세요:

```bash theme={null}
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-5'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5'
```

세션 제목 생성과 같은 백그라운드 작업은 작고 빠른 모델(보통 Haiku급 모델)을 사용합니다. 모든 계정에 Haiku 배포가 있는 것은 아니므로 Microsoft Foundry에서 Claude Code는 이 모델의 기본값을 기본 모델로 사용합니다. 백그라운드 작업에 Haiku를 사용하려면 위와 같이 계정에서 사용할 수 있는 Haiku 배포로 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 설정하세요.

현재 및 이전 모델 ID는 [모델 개요](https://platform.claude.com/docs/en/about-claude/models/overview)를 참조하세요. 환경 변수의 전체 목록은 [모델 구성](/docs/en/model-config#pin-models-for-third-party-deployments)을 참조하세요.

[프롬프트 캐싱](/docs/en/prompt-caching)은 자동으로 활성화됩니다. 기본 5분 대신 1시간 캐시 TTL을 요청하려면 다음 변수를 설정하세요; 1시간 TTL을 적용한 캐시 쓰기는 더 높은 요율로 청구됩니다:

```bash theme={null}
export ENABLE_PROMPT_CACHING_1H=1
```

### 5. Claude Code 실행

환경 변수가 설정되면 프로젝트 디렉토리에서 Claude Code를 시작하세요:

```bash theme={null}
claude
```

Claude Code는 환경 변수에서 `CLAUDE_CODE_USE_FOUNDRY` 및 기타 Microsoft Foundry 변수를 읽고 첫 프롬프트에서 Azure 리소스에 연결합니다. Amazon Bedrock 및 Google Cloud's Agent Platform과 달리 Microsoft Foundry에는 대화형 설정 마법사가 없으므로 단계 3과 4의 환경 변수가 유일한 구성 경로입니다.

설정을 검증하려면 Claude Code 내부에서 `/status`를 실행하세요. API 프로바이더 행에 `Microsoft Foundry`와 함께 구성한 리소스 이름 또는 베이스 URL이 표시됩니다.

## Azure RBAC 구성

`Azure AI User` 및 `Cognitive Services User` 기본 역할에는 Claude 모델을 호출하는 데 필요한 모든 권한이 포함되어 있습니다.

더 제한적인 권한을 설정하려면 다음 권한이 포함된 커스텀 역할을 생성하세요:

```json theme={null}
{
  "permissions": [
    {
      "dataActions": [
        "Microsoft.CognitiveServices/accounts/providers/*"
      ]
    }
  ]
}
```

자세한 내용은 [Microsoft Foundry RBAC 문서](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/rbac-azure-ai-foundry)를 참조하세요.

## 문제 해결

"Failed to get token from azureADTokenProvider: ChainedTokenCredential authentication failed" 오류가 발생하는 경우:

* 환경에 Entra ID를 구성하거나 `ANTHROPIC_FOUNDRY_API_KEY`를 설정하세요.

첫 프롬프트 시 연속된 연결 오류와 함께 요청이 실패하는 경우:

* `ANTHROPIC_FOUNDRY_RESOURCE`가 플레이스홀더가 아닌 실제 리소스 이름으로 설정되어 있는지 확인하세요. Claude Code는 이 값으로 엔드포인트 URL을 작성하므로 잘못된 이름은 존재하지 않는 호스트를 가리키게 됩니다.

## 추가 리소스

* [Microsoft Foundry 문서](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry)
* [Microsoft Foundry 모델](https://ai.azure.com/explore/models)
* [Microsoft Foundry 요금 안내](https://azure.microsoft.com/en-us/pricing/details/ai-foundry/)
