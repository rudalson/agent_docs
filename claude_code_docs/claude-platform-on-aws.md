> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# AWS 상의 Claude Platform에서 Claude Code 사용하기

> AWS 인증, IAM 액세스 제어 및 AWS Marketplace 청구를 사용하도록 Anthropic이 운영하는 Claude API를 Claude Code에 구성합니다.

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
          <strong>조직 전체에 Claude Code를 배포하시나요?</strong> 엔터프라이즈 요금제, SSO 및 중앙 집중식 청구에 대해 영업팀과 상담하세요.
        </div>
        <div className="cc-cs-actions">
          <a href={`https://claude.com/pricing?${utm('view_plans')}#plans-business`} className="cc-cs-btn-ghost">
            요금제 보기
          </a>
          <a href={`https://claude.com/contact-sales?${utm('contact_sales')}`} className="cc-cs-btn-clay">
            영업팀 문의 {iconArrowRight()}
          </a>
        </div>
      </div>
    </div>;
};

export const Experiment = ({flag, treatment, children}) => {
  const VID_KEY = 'exp_vid';
  const CONSENT_COUNTRIES = new Set(['AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI', 'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU', 'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'RE', 'GP', 'MQ', 'GF', 'YT', 'BL', 'MF', 'PM', 'WF', 'PF', 'NC', 'AW', 'CW', 'SX', 'FO', 'GL', 'AX', 'GB', 'UK', 'AI', 'BM', 'IO', 'VG', 'KY', 'FK', 'GI', 'MS', 'PN', 'SH', 'TC', 'GG', 'JE', 'IM', 'CA', 'BR', 'IN']);
  const fnv1a = s => {
    let h = 0x811c9dc5;
    for (let i = 0; i < s.length; i++) {
      h ^= s.charCodeAt(i);
      h += (h << 1) + (h << 4) + (h << 7) + (h << 8) + (h << 24);
    }
    return h >>> 0;
  };
  const bucket = (seed, vid) => fnv1a(fnv1a(seed + vid) + '') % 10000 < 5000 ? 'control' : 'treatment';
  const [decision] = useState(() => {
    const params = new URLSearchParams(location.search);
    const preBucketed = document.documentElement.dataset['gb_' + flag.replace(/-/g, '_')];
    const force = params.get('gb-force');
    if (force) {
      for (const p of force.split(',')) {
        const [k, v] = p.split(':');
        if (k === flag) return {
          variant: v || 'treatment',
          track: false
        };
      }
    }
    if (navigator.globalPrivacyControl) {
      return {
        variant: 'control',
        track: false
      };
    }
    const prefsMatch = document.cookie.match(/(?:^|; )anthropic-consent-preferences=([^;]+)/);
    if (prefsMatch) {
      try {
        if (JSON.parse(decodeURIComponent(prefsMatch[1])).analytics !== true) {
          return {
            variant: 'control',
            track: false
          };
        }
      } catch {
        return {
          variant: 'control',
          track: false
        };
      }
    } else {
      const country = params.get('country')?.toUpperCase() || (document.cookie.match(/(?:^|; )cf_geo=([A-Z]{2})/) || [])[1];
      if (!country || CONSENT_COUNTRIES.has(country)) {
        return {
          variant: 'control',
          track: false
        };
      }
    }
    let vid;
    try {
      const ajsMatch = document.cookie.match(/(?:^|; )ajs_anonymous_id=([^;]+)/);
      if (ajsMatch) {
        vid = decodeURIComponent(ajsMatch[1]).replace(/^"|"$/g, '');
      } else {
        vid = localStorage.getItem(VID_KEY);
        if (!vid) {
          vid = crypto.randomUUID();
        }
        document.cookie = `ajs_anonymous_id=${vid}; domain=.claude.com; path=/; Secure; SameSite=Lax; max-age=31536000`;
      }
      try {
        localStorage.setItem(VID_KEY, vid);
      } catch {}
    } catch {
      return {
        variant: 'control',
        track: false
      };
    }
    const variant = preBucketed === '1' ? 'treatment' : preBucketed === '0' ? 'control' : bucket(flag, vid);
    return {
      variant,
      track: true,
      vid
    };
  });
  useEffect(() => {
    if (!decision.track) return;
    fetch('https://api.anthropic.com/api/event_logging/v2/batch', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-service-name': 'claude_code_docs'
      },
      body: JSON.stringify({
        events: [{
          event_type: 'GrowthbookExperimentEvent',
          event_data: {
            device_id: decision.vid,
            anonymous_id: decision.vid,
            timestamp: new Date().toISOString(),
            experiment_id: flag,
            variation_id: decision.variant === 'treatment' ? 1 : 0,
            environment: 'production'
          }
        }]
      }),
      keepalive: true
    }).catch(() => {});
  }, []);
  return decision.variant === 'treatment' ? treatment : children;
};

<Experiment flag="docs-contact-sales-cta" treatment={<ContactSalesCard surface="claude_platform_on_aws" />} />

AWS 상의 Claude Platform은 AWS 인증, IAM 액세스 제어 및 AWS Marketplace 청구를 제공하는 Anthropic 운영 Claude API입니다. 요청이 Anthropic의 API로 직접 전달되므로 동일한 출시 일정에 맞춰 [Claude API](https://platform.claude.com/docs)와 동일한 모델 및 API 기능을 이용할 수 있습니다. Anthropic의 기능 플래그 서비스를 통해 Claude Code가 활성화하는 클라이언트 측 기능(예: [`/loop` 자율 페이싱](/docs/en/scheduled-tasks#let-claude-choose-the-interval))은 기본적으로 비활성화되어 있으며, [어드바이저 도구](/docs/en/advisor)는 사용할 수 없습니다. 전체 목록은 [기능 지원 여부 매트릭스](/docs/en/feature-availability#summary-by-provider)를 참조하세요. AWS 자격 증명 또는 워크스페이스 API 키로 인증하고, AWS Marketplace를 통해 비용을 지불합니다.

이 가이드를 사용하여 Claude Code가 AWS 상의 Claude Platform을 통해 이미 프로비저닝한 워크스페이스를 가리키도록 구성하세요. 이전에 수행하는 AWS 구독 및 워크스페이스 설정은 [AWS 상의 Claude Platform 문서](https://platform.claude.com/docs/en/build-with-claude/claude-platform-on-aws)를 참조하세요.

<Note>
  AWS Marketplace를 통해 구독하면 AWS 계정과 연결된 새 Anthropic 조직이 프로비저닝됩니다. 이 조직은 Anthropic에 이미 가지고 있는 기존 조직과 분리되어 있으며 자격 증명이 서로 이전되지 않습니다. 기존 Claude Console 계정이 아닌 AWS 연결 조직의 워크스페이스 ID와 API 키를 사용하세요.
</Note>

## 사전 요구 사항

Claude Code를 구성하기 전에 다음 사항이 필요합니다.

* AWS Marketplace를 통한 활성 AWS 상의 Claude Platform 구독
* AWS 연결 Anthropic 조직 내의 워크스페이스 및 해당 워크스페이스 ID
* Anthropic 서비스를 호출할 수 있는 권한이 있는 IAM 주체 또는 해당 워크스페이스로 범위가 지정된 API 키
* SigV4 인증을 사용하려는 경우 환경 변수, `~/.aws/credentials` 또는 연결된 IAM 역할의 AWS 자격 증명. AWS CLI는 SSO 로그인 흐름에만 필요합니다.

## 설정

### 1. AWS 자격 증명 구성

Claude Code는 AWS 상의 Claude Platform에 대해 두 가지 인증 방법을 지원합니다. 팀의 액세스 관리 방식에 맞는 방법을 선택하세요.

**옵션 A: SigV4를 사용한 AWS 자격 증명**

Claude Code는 환경 변수, `~/.aws/credentials`의 공유 자격 증명, IAM 역할, AWS SSO 세션 및 AWS SDK가 지원하는 기타 모든 소스와 같은 표준 AWS 자격 증명 체인을 사용하여 SigV4로 요청에 서명합니다.

로컬에서 사용하는 경우 Claude Code를 시작하기 전에 AWS CLI로 로그인하세요. 아래 예시는 SSO 프로필을 사용하지만, 표준 위치에 자격 증명을 생성하는 모든 방식이 작동합니다.

```bash theme={null}
aws sso login --profile my-profile
export AWS_PROFILE=my-profile
```

CI 및 자동화의 경우 실행자(runner)에 Anthropic 서비스를 호출할 권한이 있는 IAM 역할을 부여하고 `AWS_REGION`을 설정하세요. 자격 증명 체인이 역할을 자동으로 가져옵니다.

세션 도중에 SSO 자격 증명이 만료되는 경우 [`awsAuthRefresh`](/docs/en/amazon-bedrock#advanced-credential-configuration)를 구성하여 Claude Code가 실패하는 대신 로그인 명령을 재실행하고 다시 시도하도록 설정하세요. AWS 상의 Claude Platform에서 자동 새로고침을 사용하려면 Claude Code v2.1.198 이상이 필요합니다. 이전 버전은 `/login` 실행 안내와 함께 중단되며, 이는 AWS 자격 증명을 새로 고칠 수 없습니다. `~/.claude/settings.json`과 같은 [설정 파일](/docs/en/settings)에 명령을 추가하세요:

```json theme={null}
{
  "awsAuthRefresh": "aws sso login --profile my-profile"
}
```

Claude Code는 기존 AWS 자격 증명을 검증할 수 없을 때 시작 시 이 명령을 실행하고, 로그인이 완료될 때까지 `Authentication` 패널에 명령 출력을 표시합니다. {/* min-version: 2.1.212 */}v2.1.212 이전의 패널 제목은 `Cloud authentication`이었습니다.

`awsAuthRefresh`가 구성되면 `/login`은 **3rd-party 플랫폼 사용** 아래에 **Claude Platform on AWS · 자격 증명 새로고침** 옵션을 표시합니다. 이를 선택하면 구성된 명령이 실행되고 Claude Code를 재시작하지 않고 AWS 자격 증명을 다시 읽어옵니다.

**옵션 B: 워크스페이스 API 키**

워크스페이스 API 키는 연동된 AWS 자격 증명을 관리하고 싶지 않을 때 유용한 장기 실행 보안 비밀입니다. AWS 콘솔의 **Claude Platform on AWS → API keys**에서 생성하고 `ANTHROPIC_AWS_API_KEY`로 설정하세요.

```bash theme={null}
export ANTHROPIC_AWS_API_KEY=sk-ant-xxxxx
```

키는 `x-api-key`로 전송되며 SigV4보다 우선 적용되므로 환경에 있는 모든 AWS 자격 증명이 무시됩니다. 별도의 Claude Console 조직에서 생성된 API 키는 여기서 작동하지 않습니다.

워크스페이스 API 키를 다른 프로덕션 자격 증명과 동일하게 취급하세요. [사용자 설정 파일](/docs/en/settings) `env` 블록은 키를 전역으로 내보내지 않고 사용자의 시스템에 범위를 지정하는 편리한 방법입니다.

<Note>
  `/login` 및 `/logout` 명령은 AWS 상의 Claude Platform에 대한 Claude.ai 구독에 로그인시키지 않습니다. 인증은 AWS 자격 증명 또는 워크스페이스 API 키를 통해 수행됩니다. 예외는 `awsAuthRefresh`가 구성되었을 때 `/login`이 표시하는 **자격 증명 새로고침** 옵션으로, 위에 설명된 대로 AWS 자격 증명을 다시 읽어옵니다.
</Note>

### 2. Claude Code 구성

기본 Anthropic API 대신 AWS 상의 Claude Platform을 통해 Claude Code를 라우팅하는 환경 변수를 설정합니다.

```bash theme={null}
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
export AWS_REGION=us-east-1
```

`ANTHROPIC_AWS_WORKSPACE_ID`는 필수 항목이며 모든 요청에서 `anthropic-workspace-id` 헤더로 전송됩니다. 예시 `wrkspc_01ABCDEFGHIJKLMN` 값을 AWS 상의 Claude Platform 설정에서 가져온 자신의 워크스페이스 ID로 교체하세요. 기본 URL은 `AWS_REGION`에서 `https://aws-external-anthropic.{region}.api.aws`로 계산됩니다. URL을 직접 재정의하려면 `ANTHROPIC_AWS_BASE_URL`을 설정하세요.

AWS 상의 Claude Platform은 환경에 AWS 자격 증명이 있는 경우에도 명시적으로 선택(opt-in)해야 합니다. 공급자 라우팅에서 Amazon Bedrock 및 Microsoft Foundry가 우선하므로 `CLAUDE_CODE_USE_BEDROCK` 및 `CLAUDE_CODE_USE_FOUNDRY`가 설정되어 있으면 해제하세요.

### 3. 모델 버전 고정

AWS 상의 Claude Platform은 직접 Claude API와 동일한 모델 ID를 사용합니다.

기본 별칭인 `fable`, `opus`, `sonnet`, `haiku`는 AWS 상의 Claude Platform에 대한 Claude Code의 내장 기본값으로 확인되며, 이는 최신 릴리스보다 뒤처질 수 있습니다. `ANTHROPIC_DEFAULT_OPUS_MODEL`이 없으면 `opus` 별칭은 Opus 4.8로 확인됩니다. {/* min-version: 2.1.207 */}v2.1.207 이전에는 Opus 4.7로 확인되었습니다.

팀에 Claude Code를 배포하는 경우 새 릴리스로 인해 모든 사용자가 한 번에 이동하지 않도록 모델 ID를 명시적으로 고정하세요.

```bash theme={null}
export ANTHROPIC_DEFAULT_FABLE_MODEL=claude-fable-5
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-8
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-5
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5
```

모델 ID 및 별칭의 전체 목록은 [모델 개요](https://platform.claude.com/docs/en/about-claude/models/overview)를 참조하세요. 기타 모델 관련 변수는 [모델 구성](/docs/en/model-config)을 참조하세요.

[프롬프트 캐싱](/docs/en/prompt-caching)은 자동으로 활성화됩니다. 기본 5분 대신 1시간 캐시 TTL을 요청하려면 `ENABLE_PROMPT_CACHING_1H=1`을 설정하세요. API는 1시간 캐시 쓰기에 대해 더 높은 요금을 청구합니다. 요금은 [프롬프트 캐싱 가격](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#pricing)을 참조하세요.

### 4. 실행 및 검증

Claude Code를 시작하고 라우팅을 확인합니다.

```bash theme={null}
claude
```

공급자가 활성화되면 시작 배너에 `Claude Platform on AWS`가 표시됩니다. `/status`를 실행하여 세부 정보를 확인하세요. `API provider` 줄에 `Claude Platform on AWS`가 읽히고, 출력에는 `Workspace ID`, `AWS region` 및 재정의를 설정한 경우 `Claude Platform on AWS base URL`이 포함됩니다.

## Agent SDK 사용

[Agent SDK](/docs/en/agent-sdk/overview)는 CLI와 동일한 환경 변수를 읽으므로, Claude Code 하위 프로세스를 생성하는 모든 프로그램은 호출 전에 `CLAUDE_CODE_USE_ANTHROPIC_AWS`, `ANTHROPIC_AWS_WORKSPACE_ID` 및 `ANTHROPIC_AWS_API_KEY` 또는 AWS 자격 증명을 내보내어 AWS 상의 Claude Platform을 대상으로 지정할 수 있습니다.

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

process.env.CLAUDE_CODE_USE_ANTHROPIC_AWS = "1";
process.env.ANTHROPIC_AWS_WORKSPACE_ID = "wrkspc_01ABCDEFGHIJKLMN";
process.env.AWS_REGION = "us-east-1";

for await (const msg of query({ prompt: "What's in this repo?" })) {
  console.log(msg);
}
```

이 예시는 SigV4에 대해 주변 AWS 자격 증명 체인에 의존합니다. 대신 워크스페이스 API 키로 인증하려면 동일한 방식으로 `ANTHROPIC_AWS_API_KEY`를 설정하세요. 광범위한 Agent SDK 영역에 대해서는 [Agent SDK 개요](/docs/en/agent-sdk/overview)를 참조하세요.

## 프록시를 통한 라우팅

프록시 또는 [LLM 게이트웨이](/docs/en/llm-gateway)를 통해 트래픽을 라우팅하려면 `ANTHROPIC_AWS_BASE_URL`을 프록시 주소로 설정하세요. Claude Code는 동일한 워크스페이스 및 인증 헤더를 사용하여 해당 URL로 요청을 보내므로, 변경 없이 다시 전달하는 모든 게이트웨이가 작동합니다.

```bash theme={null}
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
export ANTHROPIC_AWS_BASE_URL=https://anthropic-proxy.example.com
```

게이트웨이가 요청에 직접 서명하는 경우 `CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH=1`을 설정하여 Claude Code가 서명되지 않은 요청을 보내도록 하고 게이트웨이가 AWS로 전달하기 전에 SigV4 헤더를 추가하도록 하세요. 게이트웨이에 자체 토큰이 필요한 경우 `ANTHROPIC_AUTH_TOKEN`에 설정하세요.

```bash theme={null}
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
export CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH=1
export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
export ANTHROPIC_AWS_BASE_URL=https://anthropic-proxy.example.com
```

## 문제 해결

`/status`를 실행하여 확인된 공급자와 명시적으로 구성된 워크스페이스 ID, 리전, 기본 URL 재정의 및 인증 건너뛰기 설정을 확인하세요. 이는 Claude Code가 AWS 상의 Claude Platform을 타겟팅하고 있는지 확인하는 가장 빠른 방법입니다.

### 모든 요청에서 `403 Forbidden` 또는 `AccessDenied` 발생

Claude Code가 확인한 IAM 주체에 워크스페이스에서 Anthropic 서비스를 호출할 권한이 없을 가능성이 높습니다. AWS 프로필에 연결된 역할 또는 Claude Code를 시작한 실행자(runner)를 확인하고, [IAM 작업 참조](https://platform.claude.com/docs/en/api/claude-platform-on-aws-iam-actions)에 문서화된 `aws-external-anthropic` 작업 권한이 있는지 확인하세요.

`ANTHROPIC_AWS_API_KEY`를 설정한 경우 해당 키가 SigV4보다 우선 적용되며, 만료되거나 잘못된 키도 동일한 오류를 발생시킵니다. AWS 콘솔의 **Claude Platform on AWS → API keys**에서 키를 재생성하거나 변수를 해제하여 AWS 자격 증명으로 대체하세요.

### 워크스페이스 누락 오류로 요청 실패

`ANTHROPIC_AWS_WORKSPACE_ID`가 설정되지 않았거나 비어 있을 가능성이 높습니다. 모든 AWS 상의 Claude Platform 요청에는 워크스페이스 ID가 포함되어야 합니다. AWS 자격 증명에 암시되어 있지 않습니다. AWS 상의 Claude Platform 설정에서 ID를 찾아 Claude Code를 시작하기 전에 내보내세요.

### 요청이 여전히 `api.anthropic.com`으로 전송됨

`CLAUDE_CODE_USE_ANTHROPIC_AWS`가 설정되지 않았거나 참(truthy)으로 구문 분석되지 않는 값으로 설정되었을 가능성이 높습니다. `1`로 설정하고 `/status`를 실행하여 확인된 공급자를 구성을 확인하세요. `CLAUDE_CODE_USE_BEDROCK` 또는 `CLAUDE_CODE_USE_FOUNDRY`도 설정되어 있으면 AWS 상의 Claude Platform보다 해당 항목이 우선합니다.

## 추가 리소스

Claude Code를 구성하기 전에 수행하는 AWS 상의 Claude Platform 구독, 워크스페이스 및 IAM 설정은 플랫폼 문서에서 다룹니다.

* [AWS 상의 Claude Platform 개요](https://platform.claude.com/docs/en/build-with-claude/claude-platform-on-aws): 구독, 워크스페이스 설정 및 제품 참조
* [IAM 작업 참조](https://platform.claude.com/docs/en/api/claude-platform-on-aws-iam-actions): 권한 및 관리형 정책
