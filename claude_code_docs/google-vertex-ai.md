> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Google Cloud Agent Platform에서의 Claude Code

> 설치, IAM 설정 및 문제 해결을 포함하여 Google Cloud Agent Platform(이전 Vertex AI)을 통해 Claude Code를 구성하는 방법에 대해 알아보세요.

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
          <strong>조직 전반에 Claude Code를 배포하시나요?</strong> 엔터프라이즈 플랜, SSO 및 중앙 집중식 결제에 대해 영업팀에 문의하세요.
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

<ContactSalesCard surface="vertex" />

## 사전 요구 사항

Google Cloud Agent Platform(이전 Vertex AI)과 함께 Claude Code를 구성하기전에 다음 사항을 확인하세요:

* 결제가 활성화된 Google Cloud Platform(GCP) 계정
* Google Cloud Agent Platform API가 활성화된 GCP 프로젝트
* 원하는 Claude 모델(예: Claude Sonnet 4.6)에 대한 접근 권한
* Google Cloud SDK(`gcloud`) 설치 및 구성
* 원하는 GCP 리전에 할당된 쿼터(Quota)

자신의 Google Cloud Agent Platform 자격 증명으로 로그인하려면 아래의 [Agent Platform으로 로그인](#sign-in-with-agent-platform)을 따르세요. 팀 전체에 Claude Code를 배포하려면 [수동 설정](#set-up-manually) 단계를 사용하고 배포 전에 [모델 버전 고정](#5-pin-model-versions)을 수행하세요.

## Agent Platform으로 로그인

Google Cloud 자격 증명이 있고 Google Cloud Agent Platform을 통해 Claude Code를 사용하려는 경우 로그인 마법사가 단계를 안내합니다. 프로젝트당 한 번 GCP 측 사전 요구 사항을 완료하면 마법사가 Claude Code 측 설정을 처리합니다.

<Steps>
  <Step title="GCP 프로젝트에서 Claude 모델 활성화">
    프로젝트에 대해 [Google Cloud Agent Platform API 활성화](#1-enable-agent-platform-api)를 수행한 후 [Google Cloud Agent Platform Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)에서 원하는 Claude 모델에 대한 접근을 요청하세요. 계정에 필요한 권한은 [IAM 구성](#iam-configuration)을 참조하세요.
  </Step>

  <Step title="Claude Code 시작 및 Google Cloud Agent Platform 선택">
    `claude`를 실행합니다. 로그인 프롬프트에서 **3rd-party platform**을 선택한 다음, 로그인 프롬프트에서 Google Cloud Agent Platform 레이블로 여전히 사용되는 **Google Vertex AI**를 선택합니다. 이미 로그인되어 있다면 `/login`을 실행하여 동일한 메뉴를 여세요.
  </Step>

  <Step title="마법사 프롬프트 따르기">
    Google Cloud 인증 방법을 선택합니다: `gcloud`의 애플리케이션 기본 자격 증명(ADC), 서비스 계정 키 파일 또는 환경에 이미 존재하는 자격 증명. 마법사가 프로젝트 및 리전을 감지하고 프로젝트가 호출할 수 있는 Claude 모델을 확인한 다음 이를 고정할 수 있도록 합니다. 결과는 [사용자 설정 파일](/docs/en/settings)의 `env` 블록에 저장되므로 환경 변수를 직접 내보낼 필요가 없습니다.
  </Step>
</Steps>

로그인한 후 언제든지 `/setup-vertex`를 실행하여 마법사를 다시 열고 자격 증명, 프로젝트, 리전 또는 고정된 모델을 변경할 수 있습니다. 모델 고정 단계는 현재 고정된 모델부터 시작합니다. 마법사는 `~/.claude/settings.json`에 작성하거나 [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars#variables)이 설정되어 있을 경우 `$CLAUDE_CONFIG_DIR/settings.json`에 작성합니다.

## 리전 구성

Claude Code는 Google Cloud Agent Platform의 [글로벌](https://cloud.google.com/blog/products/ai-machine-learning/global-endpoint-for-claude-models-generally-available-on-vertex-ai), 멀티 리전 및 리전 엔드포인트를 지원합니다. `CLOUD_ML_REGION`을 `global`, `eu` 또는 `us`와 같은 멀티 리전 위치 또는 `us-east5`와 같은 특정 리전으로 설정하세요. Claude Code는 멀티 리전 위치를 위한 `aiplatform.eu.rep.googleapis.com` 및 `aiplatform.us.rep.googleapis.com` 호스트를 포함하여 각 형태에 맞는 올바른 Google Cloud Agent Platform 호스트 이름을 선택합니다.

<Note>
  Google Cloud Agent Platform은 모든 엔드포인트 유형에서 Claude Code 기본 모델을 지원하지 않을 수 있습니다. 모델 가용성은 [특정 리전](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/locations#genai-partner-models), 멀티 리전 위치 및 [글로벌 엔드포인트](https://cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-partner-models#supported_models)에 따라 다릅니다. 지원되는 위치로 전환하거나 지원되는 모델을 지정해야 할 수 있습니다.
</Note>

## 수동 설정

CI 또는 스크립트 기반 엔터프라이즈 배포와 같이 마법사 대신 환경 변수를 통해 Google Cloud Agent Platform을 구성하려면 아래 단계를 따르세요.

### 1. Agent Platform API 활성화

GCP 프로젝트에서 Google Cloud Agent Platform API를 활성화합니다. 아래 명령과 다음 구성 단계에서 `YOUR-PROJECT-ID`를 사용자의 GCP 프로젝트 ID로 교체하세요:

```bash theme={null}
# 프로젝트 ID 설정
gcloud config set project YOUR-PROJECT-ID

# Agent Platform API 활성화
gcloud services enable aiplatform.googleapis.com
```

### 2. 모델 접근 요청

Google Cloud Agent Platform에서 Claude 모델에 대한 접근을 요청합니다:

1. [Google Cloud Agent Platform Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)으로 이동합니다.
2. "Claude" 모델을 검색합니다.
3. 원하는 Claude 모델(예: Claude Sonnet 4.6)에 대한 접근을 요청합니다.
4. 승인을 기다립니다 (24-48시간 소요될 수 있음).

### 3. GCP 자격 증명 구성

Claude Code는 표준 Google Cloud 인증을 사용합니다.

자세한 내용은 [Google Cloud 인증 문서](https://cloud.google.com/docs/authentication)를 참조하세요.

Claude Code v2.1.121 이상은 동일한 애플리케이션 기본 자격 증명(ADC) 체인을 통해 [X.509 인증서 기반 워크로드 아이덴티티 페더레이션](https://cloud.google.com/iam/docs/workload-identity-federation-with-x509-certificates)을 지원합니다. `GOOGLE_APPLICATION_CREDENTIALS`를 자격 증명 구성 파일 경로로 설정하세요.

<Note>
  Claude Code는 Google Cloud Agent Platform 요청의 프로젝트 ID로 `ANTHROPIC_VERTEX_PROJECT_ID`를 사용합니다. `GCLOUD_PROJECT` 및 `GOOGLE_CLOUD_PROJECT` 환경 변수와 `GOOGLE_APPLICATION_CREDENTIALS`에 의해 참조되는 자격 증명 파일이 우선합니다. 둘 다 설정되어 있지 않은 경우 프로젝트 ID는 `gcloud` 구성 또는 연결된 서비스 계정에서 해제됩니다.
</Note>

#### 고급 자격 증명 구성

Claude Code는 `gcpAuthRefresh` 설정을 통해 GCP 자격 증명의 자동 갱신을 지원합니다. Claude Code [설정 파일](/docs/en/settings)(예: `~/.claude/settings.json`)에 이를 추가하세요. Claude Code가 GCP 자격 증명이 만료되었거나 로드할 수 없음을 감지하면 지연 요청을 다시 시도하기 전에 설정된 명령을 실행하여 새 자격 증명을 획득합니다.

```json theme={null}
{
  "gcpAuthRefresh": "gcloud auth application-default login",
  "env": {
    "ANTHROPIC_VERTEX_PROJECT_ID": "your-project-id"
  }
}
```

명령의 출력은 사용자에게 표시되지만 대화형 입력은 지원되지 않습니다. 이는 CLI가 URL을 보여주고 브라우저에서 인증을 완료하는 브라우저 기반 인증 흐름에 잘 맞습니다. 인증이 완료되지 않으면 갱신 명령은 3분 후 타임아웃됩니다. `.claude/settings.json`과 같은 프로젝트 설정에서 `gcpAuthRefresh`를 설정한 경우, 워크스페이스 신뢰 프롬프트를 수락한 후에만 명령이 실행됩니다.

### 4. Claude Code 구성

다음 환경 변수들을 설정하세요:

```bash theme={null}
# Agent Platform 연동 활성화
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=global
export ANTHROPIC_VERTEX_PROJECT_ID=YOUR-PROJECT-ID

# 선택 사항: 커스텀 엔드포인트 또는 게이트웨이를 위해 Agent Platform 엔드포인트 URL 재정의
# export ANTHROPIC_VERTEX_BASE_URL=https://aiplatform.googleapis.com

# 선택 사항: 필요한 경우 프롬프트 캐싱 비활성화
# export DISABLE_PROMPT_CACHING=1

# 선택 사항: 기본 5분 대신 1시간 프롬프트 캐시 TTL 요청
# export ENABLE_PROMPT_CACHING_1H=1

# CLOUD_ML_REGION=global 설정 시 글로벌 엔드포인트를 지원하지 않는 모델에 대해 리전 재정의
export VERTEX_REGION_CLAUDE_HAIKU_4_5=us-east5
export VERTEX_REGION_CLAUDE_4_6_SONNET=europe-west1
```

대부분의 모델 버전에는 해당되는 `VERTEX_REGION_CLAUDE_*` 변수가 있습니다. 전체 목록은 [환경 변수 레퍼런스](/docs/en/env-vars)를 참조하세요. [Google Cloud Agent Platform Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)을 확인하여 어떤 모델이 글로벌 엔드포인트를 지원하고 어떤 모델이 리전 전용인지 확인하세요.

[프롬프트 캐싱](/docs/en/prompt-caching)은 자동으로 활성화됩니다. 이를 비활성화하려면 `DISABLE_PROMPT_CACHING=1`을 설정하세요. 기본 5분 대신 1시간 캐시 TTL을 요청하려면 `ENABLE_PROMPT_CACHING_1H=1`을 설정하되, 1시간 TTL의 캐시 쓰기는 더 높은 요율로 청구됩니다. 속도 제한(Rate Limit) 증액이 필요한 경우 Google Cloud 지원팀에 문의하세요. Google Cloud Agent Platform을 사용할 때는 Google Cloud 자격 증명을 통해 인증이 처리되므로 `/logout` 명령을 사용할 수 없습니다.

Claude Code는 Google Cloud Agent Platform에서 기본적으로 [MCP 도구 검색](/docs/en/mcp#scale-with-mcp-tool-search)을 비활성화하므로 MCP 도구 정의가 사전에 로드됩니다. Google Cloud Agent Platform은 Claude Sonnet 4.5 이상 및 Claude Opus 4.5 이상에서 도구 검색을 지원합니다. 해당 모델에서 이를 활성화하려면 `ENABLE_TOOL_SEARCH=true`를 설정하세요. Google Cloud Agent Platform의 이전 모델은 필요한 베타 헤더를 수락하지 않으며 도구 검색을 활성화하면 요청이 실패합니다.

### 5. 모델 버전 고정

<Warning>
  여러 사용자에게 배포할 때는 특정 모델 버전을 고정하세요. 고정하지 않으면 `sonnet` 및 `opus`와 같은 모델 별칭이 Google Cloud Agent Platform용 Claude Code 내장 기본값으로 해제되며, 이는 최신 릴리스보다 늦어지거나 프로젝트에서 아직 활성화되지 않았을 수 있습니다. 기본값을 사용할 수 없는 경우 시작 시 Claude Code가 이전 또는 하위 계층 모델로 [폴백(fallback)](#startup-model-checks)하지만, 고정을 통해 사용자가 새 모델로 이동하는 시점을 제어할 수 있습니다.
</Warning>

이 환경 변수들을 특정 Google Cloud Agent Platform 모델 ID로 설정하세요.

`ANTHROPIC_DEFAULT_OPUS_MODEL`이 없으면 Google Cloud Agent Platform의 `opus` 별칭은 Opus 4.8로 해제되고, `ANTHROPIC_DEFAULT_SONNET_MODEL`이 없으면 `sonnet` 별칭은 Sonnet 4.5로 해제됩니다. 이 예시는 각 별칭을 특정 버전에 고정합니다:

```bash theme={null}
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-5'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5@20251001'
```

현재 및 이전 모델 ID는 [모델 개요](https://platform.claude.com/docs/en/about-claude/models/overview)를 참조하세요. 환경 변수의 전체 목록은 [모델 구성](/docs/en/model-config#pin-models-for-third-party-deployments)을 참조하세요.

Claude Code는 고정 변수가 설정되지 않은 경우 다음 기본 모델을 사용합니다:

| 모델 유형       | 기본값                |
| :--------------- | :--------------------------- |
| 기본 모델    | `claude-opus-4-8`            |
| 소형/고속 모델 | `claude-sonnet-4-5@20250929` |

세션 제목 생성과 같은 백그라운드 작업은 소형/고속 모델(일반적으로 Haiku 클래스 모델)을 사용합니다. Google Cloud Agent Platform에서 Claude Code는 백그라운드 작업에 기본 Sonnet 모델을 사용하는데, 그 이유는 모든 프로젝트나 리전에서 Haiku가 활성화되어 있지 않을 수 있기 때문입니다. 다음 두 가지 선택이 해당 작업을 수행하는 모델을 변경합니다:

* `--model`, `ANTHROPIC_MODEL` 또는 `model` 설정으로 기본 모델을 선택하면 백그라운드 작업도 해당 모델을 사용합니다. `ANTHROPIC_DEFAULT_SONNET_MODEL` 없이 `ANTHROPIC_DEFAULT_OPUS_MODEL`을 설정하는 것 역시 선택으로 간주됩니다. 자체 Opus를 지정하는 프로젝트에서는 내장 Sonnet 모델이 활성화되어 있지 않을 수 있기 때문입니다.
* 백그라운드 작업에 Haiku를 사용하려면 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 프로젝트에서 사용 가능한 모델 ID로 설정하세요.

<Warning>
  Opus 모델은 Sonnet 모델보다 토큰당 가격이 높으므로, 기본 모델을 고정하지 않은 배포는 v2.1.207 이상으로 업데이트되면 Opus 요율로 청구됩니다. Sonnet 4.5를 기본 모델로 유지하려면 `ANTHROPIC_MODEL`을 전체 모델 ID로 설정하세요. `ANTHROPIC_DEFAULT_SONNET_MODEL`로 기본값을 지정하고 `ANTHROPIC_DEFAULT_OPUS_MODEL`을 설정하지 않은 배포는 지정된 Sonnet 모델을 기본값으로 유지합니다.
</Warning>

v2.1.207 이전에는 Google Cloud Agent Platform의 기본 모델이 Sonnet 4.5였고, `opus` 별칭은 Opus 4.6으로 해제되었으며, 백그라운드 작업은 항상 기본 모델을 사용했습니다.

모델을 추가로 커스텀하려는 경우:

```bash theme={null}
export ANTHROPIC_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5@20251001'
```

### 6. 구성 검증

Claude Code를 시작하고 `/status`를 실행하여 설정을 확인하세요. `API provider` 행에 `Google Vertex AI`가 표시되고, `GCP project`, `Default region`, `Model` 행에 프로젝트 ID, 리전 및 확인된 모델이 표시됩니다. 제공자 행이 누락된 경우 환경 변수가 프로세스에 전달되지 않는 것입니다. `claude`를 실행한 셸에서 환경 변수가 내보내졌는지 확인하거나 [설정 파일](/docs/en/settings)의 `env` 블록에 설정하세요.

## 시작 시 모델 검사

Google Cloud Agent Platform이 구성된 상태에서 Claude Code가 시작되면 사용하려는 모델을 프로젝트에서 이용할 수 있는지 확인합니다.

현재 Claude Code 기본값보다 이전 모델 버전을 고정했으며 프로젝트가 더 새로운 버전을 호출할 수 있는 경우, Claude Code는 고정 업데이트를 요청합니다. 이를 수락하면 새 모델 ID가 [사용자 설정 파일](/docs/en/settings)에 작성되고 Claude Code가 다시 시작됩니다. 거절하면 다음 기본 버전 변경 시까지 거절 상태가 기억됩니다.

모델을 고정하지 않았고 현재 기본값을 프로젝트에서 사용할 수 없는 경우, Claude Code는 현재 세션에 대해 폴백하고 알림을 표시합니다. 기본 모델의 이전 버전을 먼저 시도하며, 기본값이 Opus 모델이고 사용 가능한 Opus 버전이 없는 경우 기본 Sonnet 모델로 폴백합니다. 폴백은 영구 저장되지 않습니다. 선택을 영구화하려면 [Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)에서 새 모델을 활성화하거나 [버전을 고정](#5-pin-model-versions)하세요.

`--model`, `ANTHROPIC_MODEL` 또는 [`model` 설정](/docs/en/settings)을 사용하여 특정 Sonnet 또는 Opus 버전으로 세션을 시작하면 해당 버전이 일치하는 `sonnet` 또는 `opus` 별칭의 세션 고정 기본값 역할을 합니다. Claude Code는 모델이 대체하는 내장 기본값의 가용성 검사를 건너뛰고 폴백 알림 없이 구성한 모델로 시작합니다.

`opus`와 같은 모델 별칭은 고정 역할을 하지 않으며, Claude Code가 인식하지 못하는 모델 ID도 고정 역할을 하지 않습니다.

<Info>v2.1.211 이전에는 세션 모델이 명시적으로 구성된 경우에도 Claude Code가 기본 모델의 가용성을 검사했으며, 세션이 사용하지 않는 기본값에 대해 폴백 알림을 보여줄 수 있었습니다.</Info>

## IAM 구성

필요한 IAM 권한을 할당하세요:

`roles/aiplatform.user` 역할에는 필요한 권한이 포함되어 있습니다:

* `aiplatform.endpoints.predict` - 모델 호출 및 토큰 계산에 필요함

더 제한적인 권한을 적용하려면 위의 권한만 포함하는 커스텀 역할을 생성하세요.

자세한 내용은 [Google Cloud Agent Platform IAM 문서](https://cloud.google.com/vertex-ai/docs/general/access-control)를 참조하세요.

<Note>
  비용 추적 및 접근 제어를 단순화하기 위해 Claude Code 전용 GCP 프로젝트를 생성하세요.
</Note>

## 1M 토큰 컨텍스트 윈도우

Claude Sonnet 5, Opus 4.6 이상, Sonnet 4.6은 Google Cloud Agent Platform에서 [1M 토큰 컨텍스트 윈도우](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-window-sizes-by-model)를 지원합니다. Sonnet 5는 선택할 `[1m]` 변형 없이 항상 1M 윈도우로 실행됩니다. 다른 모델의 경우 1M 모델 변형을 선택하면 Claude Code가 확장된 컨텍스트 윈도우를 자동으로 활성화합니다.

[설정 마법사](#sign-in-with-agent-platform)는 모델을 고정할 때 1M 컨텍스트 옵션을 제공합니다. 수동으로 고정된 모델에 대해 이를 활성화하려면 모델 ID 뒤에 `[1m]`을 추가하세요. 자세한 내용은 [제3자 배포용 모델 고정](/docs/en/model-config#pin-models-for-third-party-deployments)을 참조하세요.

## 문제 해결

"Could not load the default credentials" 오류가 발생하는 경우:

* `gcloud auth application-default login`을 실행하여 Application Default Credentials를 설정하세요.
* `GOOGLE_APPLICATION_CREDENTIALS`를 서비스 계정 키 파일 경로로 설정하세요.
* 모든 옵션에 대해서는 [GCP 자격 증명 구성](#3-configure-gcp-credentials)을 참조하세요.

쿼터 이슈가 발생하는 경우:

* [Cloud Console](https://cloud.google.com/docs/quotas/view-manage)을 통해 현재 쿼터를 확인하거나 쿼터 인상을 요청하세요.

"model not found" 404 오류가 발생하는 경우:

* [Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)에서 모델이 활성화(Enabled)되어 있는지 확인하세요.
* 지정한 위치에서 모델을 사용할 수 있는지 확인하세요. 일부 모델은 특정 리전이 아닌 `global` 또는 `eu`, `us`와 같은 멀티 리전 위치에서만 제공됩니다.
* `CLOUD_ML_REGION=global`을 사용하는 경우 [Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)의 "Supported features" 아래에서 모델이 글로벌 엔드포인트를 지원하는지 확인하세요. 글로벌 엔드포인트를 지원하지 않는 모델의 경우:
  * `ANTHROPIC_MODEL` 또는 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 통해 지원되는 모델을 지정하거나,
  * `VERTEX_REGION_<MODEL_NAME>` 환경 변수를 사용하여 리전 또는 멀티 리전 위치를 설정하세요.

429 오류가 발생하는 경우:

* 리전 엔드포인트의 경우 선택한 리전에서 기본 모델과 소형/고속 모델이 지원되는지 확인하세요.
* 더 나은 가용성을 위해 `CLOUD_ML_REGION=global`로 전환하는 것을 고려하세요.

## 추가 리소스

* [Google Cloud Agent Platform 문서](https://cloud.google.com/vertex-ai/docs)
* [Google Cloud Agent Platform 요금](https://cloud.google.com/vertex-ai/pricing)
* [Google Cloud Agent Platform 쿼터 및 제한](https://cloud.google.com/vertex-ai/docs/quotas)
