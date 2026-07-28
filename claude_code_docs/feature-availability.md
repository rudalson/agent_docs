> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 기능 제공 현황 (Feature availability)

> Anthropic 구독 요금제, Anthropic Console, Amazon Bedrock, Claude Platform on AWS, Google Cloud Agent Platform, Microsoft Foundry 간에 제공되는 Claude Code 기능을 비교하세요.

Claude Code CLI 및 로컬에서 실행되는 모든 기능은 모든 공급자에서 동일하게 작동합니다. 공급자별 설정 지침은 [엔터프라이즈 배포 개요](/docs/en/third-party-integrations)를 참조하세요. 사용 중인 공급자에서 누락된 기능으로 바로 이동하려면 [공급자별 요약](#summary-by-provider) 탭을 확인하세요.

아래 표에서 ✓는 이용 가능, ✗는 이용 불가, "각주 참조"는 일부 지원에 대한 각주 링크를 의미합니다. ✓ 뒤의 수식어는 해당 하위 집합으로 제공 범위를 제한하며, "관리자 활성화"는 조직 관리자가 켜기 전까지 꺼져 있음을 의미합니다.

## 모델 공급자별 제공 현황

인증 방식에 따라 Claude Code가 도달할 수 있는 기능이 결정됩니다. 사용 중인 공급자에서 누락된 항목을 단일 목록으로 보려면 [공급자별 요약](#summary-by-provider) 탭을 참조하세요. 아래 표의 열을 찾으려면:

* **Claude 구독**: Pro, Max, Team 또는 Enterprise 요금제의 claude.ai 계정으로 로그인함
* **Anthropic Console**: Anthropic API 키로 인증함
* **Amazon Bedrock**: Amazon Bedrock 모델 카탈로그의 Claude 모델을 사용하고 `CLAUDE_CODE_USE_BEDROCK`을 설정함. [Mantle 엔드포인트](/docs/en/amazon-bedrock#use-the-mantle-endpoint)(`CLAUDE_CODE_USE_MANTLE`)도 이 열에 포함됨
* **Claude Platform on AWS**: AWS Marketplace를 통해 Claude를 구매했지만 Anthropic API를 호출하고 `CLAUDE_CODE_USE_ANTHROPIC_AWS`를 설정함
* **Google Cloud Agent Platform**: Google이 운영하며 `CLAUDE_CODE_USE_VERTEX`를 설정함
* **Microsoft Foundry**: Azure 상의 Anthropic 운영 환경이며 `CLAUDE_CODE_USE_FOUNDRY`를 설정함

### 모든 공급자에서 제공되는 기능

이 기능들은 모든 공급자에서 작동합니다.

* [CLI](/docs/en/quickstart) 및 [Agent SDK](/docs/en/agent-sdk/overview)
* [VS Code](/docs/en/vs-code) 및 [JetBrains](/docs/en/jetbrains) 확장 프로그램
* [하위 에이전트](/docs/en/sub-agents), [훅](/docs/en/hooks-guide), [명령어](/docs/en/commands), [스킬](/docs/en/skills)
* [CLAUDE.md 메모리](/docs/en/memory), [플러그인](/docs/en/plugins), [MCP 서버](/docs/en/mcp)
* [체크포인트](/docs/en/checkpointing), [샌드박싱](/docs/en/sandboxing), [워크플로우](/docs/en/workflows)
* [OpenTelemetry 메트릭](/docs/en/monitoring-usage) 및 [관리 대상 설정 파일](/docs/en/settings#settings-files)

이 중 세 가지에는 공급자별 차이점이 있습니다.

* **MCP 서버**: [claude.ai의 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai)는 claude.ai 구독이 활성 인증 방식일 때만 로드되며, [도구 검색](/docs/en/mcp#configure-tool-search)은 Google Cloud Agent Platform 및 `ANTHROPIC_BASE_URL`이 퍼스트 파티가 아닌 호스트를 가리킬 때 기본적으로 비활성화됩니다.
* **하위 에이전트**: 내장된 [Explore 하위 에이전트](/docs/en/sub-agents#built-in-subagents)는 Claude API에서 상속받는 모델을 Opus로 제한하며, Claude Platform on AWS를 포함한 다른 공급자에서는 메인 대화의 모델을 직접 상속받습니다.
* **[명령어](/docs/en/commands#all-commands)**: `/design-sync` 및 `/radio`는 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry, Claude Platform on AWS에서 이용할 수 없으며, `/voice`는 claude.ai 계정이 필요합니다.

### Claude 구독이 필요한 기능

이 기능들은 claude.ai 계정으로 로그인해야 하며 Anthropic Console API 키나 서드파티 공급자에서는 접근할 수 없습니다.

* [Claude Code on the web](/docs/en/claude-code-on-the-web), Claude Code 모바일 앱, [Claude Code in Slack](/docs/en/slack)
* [Claude Code 데스크톱 앱](/docs/en/desktop)
* [루틴](/docs/en/routines) (`/schedule`)
* [Ultraplan](/docs/en/ultraplan) 및 [Ultrareview](/docs/en/ultrareview)
* [Code Review](/docs/en/code-review): Team 및 Enterprise 요금제
* [Remote Control](/docs/en/remote-control)
* [Chrome 확장 프로그램](/docs/en/chrome)
* [컴퓨터 사용(Computer use)](/docs/en/computer-use): Pro 및 Max 요금제
* [아티팩트(Artifacts)](/docs/en/artifacts): Pro, Max, Team, Enterprise 요금제
* [음성 받아쓰기](/docs/en/voice-dictation)

데스크톱 앱은 부분적 예외입니다: [게이트웨이 라우팅을 앱 내부나 관리자가 구성](/docs/en/llm-gateway-connect#desktop-app)할 수 있으며, 엔터프라이즈 배포는 [관리 대상 설정](https://claude.com/docs/third-party/claude-desktop/configuration)을 통해 데스크톱을 Google Cloud Agent Platform이나 게이트웨이 공급자로 라우팅할 수 있고, [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview)는 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 또는 자체 호스팅 LLM 게이트웨이에서 Code 탭을 실행합니다. 이러한 기능의 요금제별 이용 가능 여부는 [구독 요금제별 제공 현황](#availability-by-subscription-plan)을 참조하세요.

### 공급자별로 다른 CLI 기능

이 기능들은 로컬 CLI에서 작동하지만 공급자가 모두 표출하지는 않는 서버 측 기능에 의존합니다.

<table>
  <thead>
    <tr>
      <th>기능</th>
      <th>Claude 구독</th>
      <th>Anthropic Console</th>
      <th>Amazon Bedrock</th>
      <th>Claude Platform on AWS</th>
      <th>Google Cloud Agent Platform</th>
      <th>Microsoft Foundry</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>[웹 검색](/docs/en/tools-reference#websearch-tool-behavior)</td>
      <td>✓</td>
      <td>✓</td>
      <td>✗</td>
      <td>✓</td>
      <td>각주 참조 <sup><a href="#fn1">1</a></sup></td>
      <td>✓</td>
    </tr>

    <tr>
      <td>[패스트 모드](/docs/en/fast-mode)</td>
      <td>✓</td>
      <td>✓</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
    </tr>

    <tr>
      <td>[자동 모드](/docs/en/auto-mode-config)</td>
      <td>✓</td>
      <td>✓</td>
      <td>각주 참조 <sup><a href="#fn2">2</a></sup></td>
      <td>✓</td>
      <td>각주 참조 <sup><a href="#fn2">2</a></sup></td>
      <td>각주 참조 <sup><a href="#fn2">2</a></sup></td>
    </tr>

    <tr>
      <td>[어드바이저](/docs/en/advisor)</td>
      <td>✓</td>
      <td>✓</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
    </tr>

    <tr>
      <td>[채널 (Channels)](/docs/en/channels)</td>
      <td>✓</td>
      <td>✓</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
    </tr>

    <tr>
      <td>[`/loop` 예약 작업](/docs/en/scheduled-tasks)</td>
      <td>✓</td>
      <td>✓</td>
      <td>각주 참조 <sup><a href="#fn3">3</a></sup></td>
      <td>각주 참조 <sup><a href="#fn3">3</a></sup></td>
      <td>각주 참조 <sup><a href="#fn3">3</a></sup></td>
      <td>각주 참조 <sup><a href="#fn3">3</a></sup></td>
    </tr>

    <tr>
      <td>[GitHub Actions](/docs/en/github-actions) 및 [GitLab CI/CD](/docs/en/gitlab-ci-cd)</td>
      <td>✓</td>
      <td>✓</td>
      <td>✓</td>
      <td>✓</td>
      <td>✓</td>
      <td>✗</td>
    </tr>
  </tbody>
</table>

### 관리자 및 분석

조직 수준의 제어 및 사용량 가시성입니다.

<table>
  <thead>
    <tr>
      <th>기능</th>
      <th>Claude 구독</th>
      <th>Anthropic Console</th>
      <th>Amazon Bedrock</th>
      <th>Claude Platform on AWS</th>
      <th>Google Cloud Agent Platform</th>
      <th>Microsoft Foundry</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>[분석 대시보드 및 API](/docs/en/analytics)</td>
      <td>✓ (대시보드: Team/Enterprise; API: Enterprise)</td>
      <td>✓ <sup><a href="#fn5">5</a></sup></td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
    </tr>

    <tr>
      <td>[서버 관리 대상 설정](/docs/en/server-managed-settings)</td>
      <td>✓ (Team 및 Enterprise)</td>
      <td>✓ (Team 및 Enterprise)</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
      <td>✗</td>
    </tr>

    <tr>
      <td>[Zero Data Retention (ZDR)](/docs/en/zero-data-retention)</td>
      <td>✓ (자격 대상 Enterprise 계정)</td>
      <td>✓ (자격 대상 계정)</td>
      <td>각주 참조 <sup><a href="#fn4">4</a></sup></td>
      <td>✓ (자격 대상 계정)</td>
      <td>각주 참조 <sup><a href="#fn4">4</a></sup></td>
      <td>각주 참조 <sup><a href="#fn4">4</a></sup></td>
    </tr>
  </tbody>
</table>

<span id="fn1" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>1</sup> Google Cloud Agent Platform의 경우 웹 검색은 Claude 4 모델 이상에서 제공됩니다.<br />
<span id="fn2" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>2</sup> 해당 공급자에서 자동 모드는 Claude Sonnet 5, Opus 4.7, Opus 4.8, Fable 5만 지원합니다. [자동 모드 구성](/docs/en/auto-mode-config)을 참조하세요.<br />
<span id="fn3" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>3</sup> `/loop every 2 hours`와 같은 명시적 간격은 모든 공급자에서 작동합니다. Amazon Bedrock, Claude Platform on AWS, Google Cloud Agent Platform, Microsoft Foundry에서는 `/loop`가 간격을 직접 선택하거나 기본 유지관리 프롬프트를 제공할 수 없어 간격 없는 프롬프트는 10분마다 실행되며 인수 없는 `/loop`는 사용 안내를 표시합니다. [예약 작업](/docs/en/scheduled-tasks)을 참조하세요.<br />
<span id="fn4" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>4</sup> 클라우드 공급자와의 계약에 따릅니다.<br />
<span id="fn5" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>5</sup> 대시보드 및 API 전용입니다. [기여 메트릭](/docs/en/analytics#enable-contribution-metrics)에는 claude.ai Team 또는 Enterprise 조직이 필요합니다.

<Note>
  [LLM 게이트웨이](/docs/en/llm-gateway)를 통해 인증하는 경우 기능 가시성은 게이트웨이가 전달하는 기본 공급자와 일치합니다. [어드바이저](/docs/en/advisor)와 같은 일부 Anthropic 전용 기능은 게이트웨이가 Anthropic API로 요청을 유실 없이 전달할 때만 작동합니다.
</Note>

### 공급자별 요약

각 탭에는 해당 공급자에서 이용할 수 없거나 일부만 지원되는 항목이 대안과 함께 나열되어 있습니다. 나열되지 않은 다른 모든 기능은 위에 언급된 [공급자별 차이점](#features-available-on-every-provider)을 제외하고 Claude 구독과 동일하게 작동합니다. Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry, Claude Platform on AWS에서는 Anthropic으로의 오류 보고 및 텔레메트리가 기본적으로 꺼져 있습니다. Anthropic에 도달하는 트래픽 및 수동 옵트아웃 방법은 [API 공급자별 기본 동작](/docs/en/data-usage#default-behaviors-by-api-provider)을 참조하세요.

<Tabs>
  <Tab title="Amazon Bedrock">
    **이용 불가:** [Claude 구독이 필요한 모든 기능](#features-that-require-a-claude-subscription), [웹 검색](/docs/en/tools-reference#websearch-tool-behavior), [패스트 모드](/docs/en/fast-mode), [어드바이저](/docs/en/advisor), [채널](/docs/en/channels), [분석 대시보드](/docs/en/analytics), [서버 관리 대상 설정](/docs/en/server-managed-settings), [`/design-sync` 및 `/radio` 명령](/docs/en/commands#all-commands).

    **일부 지원:**

    * [데스크톱 앱](/docs/en/desktop): [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview)를 통해서만 가능
    * [자동 모드](/docs/en/auto-mode-config): Sonnet 5, Opus 4.7, Opus 4.8, Fable 5만 지원
    * [`/loop`](/docs/en/scheduled-tasks): 명시적 간격 지정 시에만 가능
    * [Zero Data Retention](/docs/en/zero-data-retention): AWS 계약에 따름

    **대안:** 일정을 예약하려면 `/schedule` 대신 명시적 간격이 포함된 [`/loop`](/docs/en/scheduled-tasks)을 사용하세요. 클라우드 세션에는 [GitHub Actions](/docs/en/github-actions)나 [GitLab CI/CD](/docs/en/gitlab-ci-cd)를 사용하세요. 웹 조회의 경우 특정 URL과 함께 [WebFetch 도구](/docs/en/tools-reference#webfetch-tool-behavior)를 사용하세요.
  </Tab>

  <Tab title="Claude Platform on AWS">
    **이용 불가:** [Claude 구독이 필요한 모든 기능](#features-that-require-a-claude-subscription), [패스트 모드](/docs/en/fast-mode), [어드바이저](/docs/en/advisor), [채널](/docs/en/channels), [분석 대시보드](/docs/en/analytics), [서버 관리 대상 설정](/docs/en/server-managed-settings), [`/design-sync` 및 `/radio` 명령](/docs/en/commands#all-commands).

    **Amazon Bedrock과 달리 제공되는 기능:** [웹 검색](/docs/en/tools-reference#websearch-tool-behavior).

    **일부 지원:**

    * [`/loop`](/docs/en/scheduled-tasks): 명시적 간격 지정 시에만 가능

    **대안:** 일정을 예약하려면 `/schedule` 대신 명시적 간격이 포함된 [`/loop`](/docs/en/scheduled-tasks)을 사용하세요. 클라우드 세션에는 [GitHub Actions](/docs/en/github-actions)나 [GitLab CI/CD](/docs/en/gitlab-ci-cd)를 사용하세요.
  </Tab>

  <Tab title="Google Cloud Agent Platform">
    **이용 불가:** [Claude 구독이 필요한 모든 기능](#features-that-require-a-claude-subscription), [패스트 모드](/docs/en/fast-mode), [어드바이저](/docs/en/advisor), [채널](/docs/en/channels), [분석 대시보드](/docs/en/analytics), [서버 관리 대상 설정](/docs/en/server-managed-settings), [`/design-sync` 및 `/radio` 명령](/docs/en/commands#all-commands).

    **일부 지원:**

    * [데스크톱 앱](/docs/en/desktop): [관리 대상 설정](https://claude.com/docs/third-party/claude-desktop/configuration) 또는 [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview)를 통해 가능
    * [웹 검색](/docs/en/tools-reference#websearch-tool-behavior): Claude 4 모델 이상에서 제공
    * [자동 모드](/docs/en/auto-mode-config): Sonnet 5, Opus 4.7, Opus 4.8, Fable 5만 지원
    * [`/loop`](/docs/en/scheduled-tasks): 명시적 간격 지정 시에만 가능
    * [Zero Data Retention](/docs/en/zero-data-retention): Google Cloud 계약에 따름

    **대안:** 일정을 예약하려면 `/schedule` 대신 명시적 간격이 포함된 [`/loop`](/docs/en/scheduled-tasks)을 사용하세요. 클라우드 세션에는 [GitHub Actions](/docs/en/github-actions)나 [GitLab CI/CD](/docs/en/gitlab-ci-cd)를 사용하세요.
  </Tab>

  <Tab title="Microsoft Foundry">
    **이용 불가:** [Claude 구독이 필요한 모든 기능](#features-that-require-a-claude-subscription), [패스트 모드](/docs/en/fast-mode), [어드바이저](/docs/en/advisor), [채널](/docs/en/channels), [GitHub Actions](/docs/en/github-actions) 및 [GitLab CI/CD](/docs/en/gitlab-ci-cd), [분석 대시보드](/docs/en/analytics), [서버 관리 대상 설정](/docs/en/server-managed-settings), [`/design-sync` 및 `/radio` 명령](/docs/en/commands#all-commands).

    **일부 지원:**

    * [데스크톱 앱](/docs/en/desktop): [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview)를 통해서만 가능
    * [자동 모드](/docs/en/auto-mode-config): Sonnet 5, Opus 4.7, Opus 4.8, Fable 5만 지원
    * [`/loop`](/docs/en/scheduled-tasks): 명시적 간격 지정 시에만 가능
    * [Zero Data Retention](/docs/en/zero-data-retention): Azure 계약에 따름

    **대안:** 일정을 예약하려면 `/schedule` 대신 명시적 간격이 포함된 [`/loop`](/docs/en/scheduled-tasks)을 사용하세요.
  </Tab>

  <Tab title="Anthropic Console">
    **이용 불가:** [Claude 구독이 필요한 모든 기능](#features-that-require-a-claude-subscription).

    [공급자별로 다른 CLI 기능](#cli-capabilities-that-vary-by-provider)의 모든 항목이 제공되며, API 키가 Team 또는 Enterprise 조직에 속한 경우 [서버 관리 대상 설정](/docs/en/server-managed-settings)도 제공됩니다.
  </Tab>
</Tabs>

## 구독 요금제별 제공 현황

Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry, Anthropic Console API 키를 통해 인증하는 경우 이 섹션은 적용되지 않습니다. claude.ai 계정으로 로그인할 때 사용 중인 요금제에 따라 아래 기능의 이용 가능 여부가 결정됩니다.

| 기능 | Pro | Max | Team | Enterprise |
| :--- | :--- | :--- | :--- | :--- |
| [Claude Code on the web](/docs/en/claude-code-on-the-web) | ✓ | ✓ | ✓ | ✓ <sup><a href="#fn6">6</a></sup> |
| [루틴](/docs/en/routines) | ✓ | ✓ | ✓ | ✓ |
| [Remote Control](/docs/en/remote-control) | ✓ | ✓ | 관리자 활성화 | 관리자 활성화 |
| [채널 (Channels)](/docs/en/channels) | ✓ | ✓ | 관리자 활성화 | 관리자 활성화 |
| [컴퓨터 사용](/docs/en/computer-use) | ✓ | ✓ | ✗ | ✗ |
| 디스패치 ([데스크톱](/docs/en/desktop#sessions-from-dispatch)) | ✓ | ✓ | ✗ | ✗ |
| [Code Review](/docs/en/code-review) | ✗ | ✗ | ✓ | ✓ |
| [아티팩트 (Artifacts)](/docs/en/artifacts) | ✓ | ✓ | ✓ | 관리자 활성화 |
| [분석 대시보드 및 기여 메트릭](/docs/en/analytics) | ✗ | ✗ | ✓ | ✓ |
| [Enterprise Analytics API](/docs/en/analytics#access-data-programmatically) | ✗ | ✗ | ✗ | ✓ |
| [서버 관리 대상 설정](/docs/en/server-managed-settings) | ✗ | ✗ | ✓ | ✓ |
| [SSO](https://support.claude.com/en/articles/9266767-what-is-the-team-plan) | ✗ | ✗ | ✓ | ✓ |
| SCIM | ✗ | ✗ | ✗ | ✓ |
| [Compliance API](https://platform.claude.com/docs/en/api/compliance) | ✗ | ✗ | ✗ | ✓ |
| [Zero Data Retention](/docs/en/zero-data-retention) | ✗ | ✗ | ✗ | ✓ <sup><a href="#fn7">7</a></sup> |

<span id="fn6" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>6</sup> Enterprise에서는 프레미엄 시트 또는 Chat + Claude Code 시트가 필요합니다. [Claude Code on the web](/docs/en/claude-code-on-the-web)을 참조하세요.<br />
<span id="fn7" style={{display: 'block', position: 'relative', top: '-120px'}} /><sup>7</sup> 표준 Enterprise 요금제에는 포함되지 않습니다. 자격 대상 계정에 대해 Anthropic의 별도 활성화가 필요합니다. [Zero Data Retention](/docs/en/zero-data-retention)을 참조하세요.

가격 및 전체 요금제 비교는 [Team 요금제](https://support.claude.com/en/articles/9266767-what-is-the-team-plan) 및 [Enterprise 요금제](https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan)를 참조하세요.

## 모델 제공 현황

공급자 및 리전별로 제공되는 Claude 모델 및 컨텍스트 창 크기는 [모델 구성](/docs/en/model-config) 및 [모델 개요](https://platform.claude.com/docs/en/about-claude/models/overview)를 참조하세요. 비전(Vision), PDF 입력 및 extended thinking은 Claude Code 기능이라기보다 모델 기능에 해당하며 해당 모델을 제공하는 모든 공급자에서 작동합니다. [프롬프트 캐싱](/docs/en/prompt-caching)은 대부분의 공급자에서 동일하게 작동합니다; Amazon Bedrock에서는 모델별로 지원이 다릅니다.

## 관련 리소스

* [엔터프라이즈 배포 개요](/docs/en/third-party-integrations): 공급자 간 인증, 청구 및 리전 비교
* 공급자 설정 가이드: [Amazon Bedrock](/docs/en/amazon-bedrock), [Claude Platform on AWS](/docs/en/claude-platform-on-aws), [Google Cloud Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry)
* [플랫폼 및 연동](/docs/en/platforms): CLI, 데스크톱, IDE 확장 프로그램, 웹, 모바일 및 CI/CD를 포함하여 Claude Code가 실행되는 위치
