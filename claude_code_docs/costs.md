> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 효율적인 비용 관리하기

> 토큰 사용량을 추적하고, 팀 지출 한도를 설정하며, 컨텍스트 관리, 모델 선택, extended thinking 설정 및 전처리 훅을 사용하여 Claude Code 비용을 줄이세요.

Claude Code는 API 토큰 소비량에 따라 요금을 부과합니다. 구독 요금제 가격(Pro, Max, Team, Enterprise)은 [claude.com/pricing](https://claude.com/pricing)을 참조하세요. 개발자당 비용은 모델 선택, 코드베이스 크기, 다중 인스턴스 실행이나 자동화 같은 사용 패턴에 따라 크게 다릅니다.

엔터프라이즈 배포 전체에서 개발자 1인당 평균 비용은 활성 일수당 약 \$13, 월별 \$150~250 수준이며, 전체 사용자의 90%에 대해 활성 일수당 \$30 미만을 유지합니다. 자체 팀의 지출을 추정하려면 소규모 파일럿 그룹으로 시작하여 아래의 추적 도구를 사용해 전체 배포 전에 기준선(baseline)을 설정하세요.

이 페이지에서는 [비용 추적하기](#track-your-costs), [조직의 비용 관리하기](#manage-costs-for-your-organization), [토큰 사용량 줄이기](#reduce-token-usage) 방법을 다룹니다.

## 비용 추적하기

### `/usage` 명령 사용하기

<Note>
  `/usage` 상단의 Session 블록은 API 토큰 사용량을 보여주며 API 사용자를 위한 것입니다. Claude Max 및 Pro 구독자는 구독에 사용량이 포함되어 있으므로 세션 비용 수치는 청구 목적으로 관련이 없습니다. 구독자는 동일한 화면에서 요금제 사용량 바, 활동 통계 및 사용 내역을 확인합니다.
</Note>

`/usage` 상단의 Session 블록은 현재 세션의 상세 토큰 사용량 통계를 보여줍니다. 달러 금액은 토큰 수에서 로컬로 계산된 추정치이며 실제 청구서와 다를 수 있습니다. 권위 있는 청구 정보는 [Claude Console](https://platform.claude.com/usage)의 Usage 페이지를 참조하세요.

```text theme={null}
Total cost:            $0.55
Total duration (API):  6m 20s
Total duration (wall): 6h 33m 10s
Total code changes:    0 lines added, 0 lines removed
Usage by model:
   claude-sonnet-4-6:  1.2k input, 5.3k output, 940.0k cache read, 50.0k cache write ($0.55)
```

이 합계는 `/clear`로 새 세션을 시작할 때 재설정되므로 다음 세션의 총비용은 \$0부터 시작합니다. v2.1.211 이전에는 Claude Code 프로세스의 수명 동안 `/clear` 실행 후에도 비용이 계속 누적되었습니다.

Pro, Max, Team 또는 Enterprise 요금제에서 `/usage`는 요금제 한도에 반영되는 세부 내역도 보여줍니다. 최근 사용량을 스킬, 하위 에이전트, 플러그인 및 개별 MCP 서버별로 귀속시켜 각각을 전체의 비율(%)로 보여줍니다. 또한 10% 이상을 차지하는 긴 컨텍스트나 캐시 미스(cache miss) 같은 동작도 지적합니다. `d` 또는 `w`를 눌러 최근 24시간과 최근 7일 간 전환할 수 있습니다. 수치는 이 머신의 로컬 세션 기록에서 계산된 추정치이므로 다른 기기나 claude.ai의 사용량은 포함되지 않습니다.

요금제 한도 요청이 실패할 때(주로 사용량 엔드포인트에 대한 요청 수가 제한되었을 때), `/usage`는 지난 60분 이내에 이 머신에 로드했던 마지막 사용량 바를 해당 데이터의 가져온 시점을 명시하는 `Showing last-known usage` 참고 문구와 함께 보여줍니다. `r`을 눌러 다시 시도할 수 있으며 재시도가 성공하면 마지막 사용량 바가 새 데이터로 교체됩니다. 지난 60분 이내의 스냅샷이 없으면 `/usage`는 사용량 엔드포인트의 요청 수가 제한되었음을 보고하고 동일한 재시도 단축키를 제공합니다. v2.1.208 이전에는 아직 사용량을 로드하지 않은 세션에서 요청 수가 제한되면 사용량 바 없이 오류만 표시되었습니다.

[VS Code 확장 프로그램](/docs/en/vs-code#check-account-and-usage)에서는 동일한 세부 내역이 Day 및 Week 전환 기능과 함께 Account & usage 다이얼로그에 나타납니다. Claude Code v2.1.174 이상이 필요합니다.

### 구독에 사용 크레딧 추가하기

[사용 크레딧(Usage credits)](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)을 사용하면 요금제의 사용 한도를 초과하여 계속 작업할 수 있습니다. 크레딧을 관리하려면 `/login`을 통해 claude.ai 구독으로 로그인한 후 `/usage-credits`를 실행하세요. 이 명령은 API 키 인증에서는 이용할 수 없습니다. 열리는 화면은 역할에 따라 다릅니다.

| 사용자 역할 | `/usage-credits` 수행 내용 |
| :--- | :--- |
| Pro 또는 Max 구독자 | 브라우저에서 결제 설정 페이지를 엽니다 |
| 결제 접근 권한이 있는 Team 또는 Enterprise 멤버 | 브라우저에서 조직의 사용량 설정 페이지를 엽니다 |
| 결제 접근 권한이 없는 Team 또는 Enterprise 멤버 | {/* min-version: 2.1.211 */}확인을 요청한 후 조직의 관리자에게 요청을 전송합니다. v2.1.211 이전에는 확인 단계 없이 요청을 전송했습니다 |

결제 접근 권한이 없는 Team 및 Enterprise 멤버의 경우 확인 창은 대화형 세션에서만 나타납니다. `-p` 플래그를 사용하는 비대화형 모드 및 [Remote Control](/docs/en/remote-control)에서는 요청을 보내지 않고 대화형 세션에서 실행하라는 안내문이 출력됩니다.

Pro 및 Max 요금제에서 사용 크레딧이 남아 있는 상태로 지출 한도에 도달하면 CLI를 벗어나지 않고 한도를 올리거나 제거할 수 있는 프롬프트가 표시됩니다. 서버가 변경을 거부하는 경우 [Could not update your spend limit](/docs/en/errors#could-not-update-your-spend-limit)을 참조하세요.

## 조직의 비용 관리하기

제공되는 제어 기능은 조직이 Claude Code에 접근하는 방식(Claude Teams/Enterprise 요금제, Claude Console, 또는 클라우드 공급자)에 따라 달라집니다. Teams 및 Enterprise 요금제에서는 각 멤버의 시트 할당량에서 사용량이 차감됩니다. Console 및 클라우드 공급자에서는 토큰당 비용이 조직에 청구됩니다. 조직에서 로그인 방식을 혼용하는 경우 개발자별로 인증받은 방식에 따라 계량됩니다.

다음 표는 각 설정과 지출 확인 위치, 지출 제한 방법 및 사용자별 수치 추출 방법을 매핑합니다.

| 설정 방식 | 지출 확인 위치 | 지출 한도 제한 | 사용자별 보고 기능 |
| :--- | :--- | :--- | :--- |
| [Claude Teams 및 Enterprise](#claude-for-teams-and-enterprise) | [조직 분석의 지출 보고서](https://support.claude.com/en/articles/12883420-view-usage-analytics-for-team-and-enterprise-plans) | 관리자 설정의 지출 한도 | [지출 보고서 CSV](https://support.claude.com/en/articles/12883420-view-usage-analytics-for-team-and-enterprise-plans); Enterprise의 [Enterprise Analytics API](https://platform.claude.com/docs/en/api/admin/analytics) |
| [Claude Console (API)](#claude-console) | [Console 사용량 페이지](https://platform.claude.com/usage) | 워크스페이스 지출 한도 | [Console 대시보드](https://platform.claude.com/claude-code), [Claude Code Analytics API](https://platform.claude.com/docs/en/build-with-claude/claude-code-analytics-api) |
| [Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry](#cloud-providers) | 해당 클라우드의 결제 콘솔 | 클라우드 서비스의 예산 제어 기능 | [OpenTelemetry](/docs/en/monitoring-usage) 또는 [LLM 게이트웨이](/docs/en/llm-gateway) |

[OpenTelemetry 내보내기](/docs/en/monitoring-usage)는 모든 설정에서 작동하며 거의 실시간으로 자체 모니터링 스택에 사용자별 토큰 및 비용 메트릭을 스트리밍할 수 있는 유일한 옵션입니다.

### Claude Teams 및 Enterprise

Claude Teams 및 Enterprise 요금제에서 각 멤버의 Claude Code 사용량은 5시간 롤링 창 및 주간 창으로 재설정되는 시트별 할당량에서 차감됩니다. 할당량은 Claude 대화 및 Cowork와 공유되며 크기는 멤버의 [시트 등급](https://support.claude.com/en/articles/11845131-use-claude-code-with-your-team-or-enterprise-plan)(Standard 또는 Premium)에 따라 다릅니다. 제어 기능은 Claude Console이 아닌 claude.ai 관리자 콘솔에 있습니다.

* **지출 확인**: [조직 분석의 지출 보고서](https://support.claude.com/en/articles/12883420-view-usage-analytics-for-team-and-enterprise-plans)는 사용자별 및 모델별 추정 지출을 CSV 내보내기와 함께 매일 업데이트하여 보여줍니다. 보고서는 사용 크레딧 지출을 포함하며 사용 크레딧이 활성화되면 나타납니다. 시트 할당량 내부의 사용량은 달러 단위로 계량되지 않습니다.
* **도입 현황 확인**: [분석 대시보드](https://claude.ai/analytics/claude-code)는 기여 데이터 CSV 내보내기와 함께 일일 활성 사용자, 세션 및 기여 메트릭을 보여줍니다. [분석으로 팀 사용량 추적하기](/docs/en/analytics)를 참조하세요.
* **지출 한도 설정**: 시트 할당량이 기본 한도입니다. 멤버들이 이를 초과해 작업할 수 있게 하려면 [사용 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)을 켜고 조직, 그룹 또는 개별 멤버 수준에서 지출 한도를 설정하세요.
* **사용자별 수치 추출**: Enterprise 요금제에서는 [Enterprise Analytics API](https://platform.claude.com/docs/en/api/admin/analytics)가 Claude Code를 포함한 Claude 제품 전반의 사용자별 사용량 및 비용 보고서를 반환합니다. 기본 소유자(Primary Owner)가 [claude.ai/analytics/api-keys](https://claude.ai/analytics/api-keys)에서 `read:analytics` 스코프로 키를 생성합니다. Teams 요금제에서는 사용자별 및 모델별 토큰 사용량과 추정 지출을 나열하는 [지출 보고서 CSV](https://support.claude.com/en/articles/12883420-view-usage-analytics-for-team-and-enterprise-plans)를 내보내세요.

[Claude Enterprise 소비 가이드](https://support.claude.com/en/articles/14782391-claude-enterprise-consumption-guide)는 관리자를 위한 계획 참조 자료입니다. Claude 대화, Claude Code 및 Cowork 간에 소비 방식이 어떻게 다른지 설명하고 예산 작성을 위한 사용자별 달러 시작 지점을 제공합니다. 코딩 시트는 일반 대화 시트보다 더 많은 예산을 측정하세요: Claude Code의 한 턴에는 파일 내용, 도구 호출, 다단계 추론이 포함되므로 한 번의 디버깅 세션이 하루 치 일반 대화보다 더 많이 소비될 수 있습니다.

### Claude Console

API 조직은 [워크스페이스](https://platform.claude.com/docs/en/build-with-claude/workspaces)를 통해 Claude Code 지출을 관리합니다. 전체 Claude Code 지출에 대해 [워크스페이스 지출 한도를 설정](https://platform.claude.com/docs/en/build-with-claude/workspaces#workspace-limits)하고 Console에서 [비용 및 사용량 보고를 확인](https://platform.claude.com/docs/en/build-with-claude/workspaces#usage-and-cost-tracking)할 수 있습니다.

<Note>
  Claude Console 계정으로 Claude Code를 처음 인증하면 "Claude Code"라는 워크스페이스가 자동으로 생성됩니다. 이 워크스페이스는 조직 내 모든 Claude Code 사용에 대한 중앙 집중식 비용 추적 및 관리를 제공합니다. 이 워크스페이스에는 API 키를 생성할 수 없으며 Claude Code 인증 및 사용 전용입니다.

  커스텀 처리율 제한(rate limit)이 있는 조직의 경우 이 워크스페이스의 Claude Code 트래픽이 조직의 전체 API 처리율 제한에 집계됩니다. Claude Console의 Limits 페이지에서 이 워크스페이스에 대한 [워크스페이스 처리율 제한](https://platform.claude.com/docs/en/api/rate-limits#setting-lower-limits-for-workspaces)을 설정하여 Claude Code의 점유율을 제어하고 다른 프로덕션 워크로드를 보호할 수 있습니다.
</Note>

사용자별 보고의 경우 [Console 대시보드](https://platform.claude.com/claude-code)는 멤버별 지출 및 수락된 줄 수를 보여주며, [Claude Code Analytics API](https://platform.claude.com/docs/en/build-with-claude/claude-code-analytics-api)는 [Admin API 키](https://platform.claude.com/settings/admin-keys)로 동일한 일일 사용자별 메트릭을 제공합니다. [API 고객을 위한 분석](/docs/en/analytics#access-analytics-for-api-customers)을 참조하세요.

#### 처리율 제한(Rate Limit) 권장 사항

팀용으로 Claude Code를 설정할 때 조직 크기에 따른 사용자별 TPM(Token Per Minute) 및 RPM(Request Per Minute) 권장 사항입니다.

| 팀 크기 | 사용자당 TPM | 사용자당 RPM |
| :--- | :--- | :--- |
| 1-5명 | 200k-300k | 5-7 |
| 5-20명 | 100k-150k | 2.5-3.5 |
| 20-50명 | 50k-75k | 1.25-1.75 |
| 50-100명 | 25k-35k | 0.62-0.87 |
| 100-500명 | 15k-20k | 0.37-0.47 |
| 500명 이상 | 10k-15k | 0.25-0.35 |

예를 들어 200명의 사용자가 있는 경우 각 사용자당 20k TPM, 즉 총 4백만 TPM (200 \* 20,000 = 4,000,000)을 요청할 수 있습니다.

팀 크기가 커질수록 동시 사용하는 사용자의 비율이 낮아지므로 사용자당 TPM이 감소합니다. 이러한 처리율 제한은 개별 사용자별이 아닌 조직 수준에서 적용되므로 다른 사람들이 서비스를 활발히 사용하지 않을 때 개별 사용자가 계산된 지분보다 일시적으로 더 많이 소비할 수 있습니다.

<Note>
  대규모 그룹의 라이브 교육 세션과 같이 동시 사용률이 이례적으로 높은 시나리오가 예상되는 경우 사용자당 더 높은 TPM 할당이 필요할 수 있습니다.
</Note>

### 클라우드 공급자

Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry에서 Claude Code는 클라우드 계정에 토큰당 비용으로 청구되며 지출 제어 기능은 클라우드 공급자의 결제 콘솔에 있습니다. Claude Code는 클라우드의 메트릭을 Anthropic으로 다시 보내지 않으므로 [분석 대시보드](/docs/en/analytics) 및 Claude Code Analytics API는 이 사용량을 다루지 않습니다.

사용자별 비용 귀속을 위한 세 가지 옵션이 있습니다.

* **OpenTelemetry**: 개발자 머신에서 자체 모니터링 스택으로 [메트릭을 내보냅니다](/docs/en/monitoring-usage). 이는 공급자에 관계없이 사용자별 토큰 수, 비용 및 도구 활동을 제공합니다.
* **Claude 앱 게이트웨이**: 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)는 사용자별 사용 귀속, 토큰 수가 포함된 OTLP 메트릭 및 해당 공급자상의 [사용자별 지출 한도](/docs/en/claude-apps-gateway-spend-limits)를 제공합니다.
* **LLM 게이트웨이**: 모든 Claude Code 트래픽을 키별 지출을 추적하는 프록시로 라라우팅합니다. 여러 대기업에서 [키별 지출을 추적](https://docs.litellm.ai/docs/proxy/virtual_keys#tracking-spend)하는 오픈소스 도구인 [LiteLLM](/docs/en/llm-gateway)을 사용한다고 보고했습니다. 이 프로젝트는 Anthropic과 관련이 없으며 보안 감사를 받지 않았습니다.

### 개발자가 한도에 대해 문의할 때

개발자는 보통 관리자에게 한도 질문을 하므로 도달한 한도가 어떤 것인지 아는 것이 도움이 됩니다. 다음 세 가지 상황은 서로 다른 의미를 갖습니다.

* **"You've hit your session limit" 또는 "You've hit your weekly limit"**: 구독 요금제의 시트 기반 사용 창입니다. 이러한 창은 모든 모델에서 공유되므로 `/model`로 모델을 전환해도 접근 권한이 복원되지 않습니다(단, 모델별 "You've hit your Opus limit" 메시지 이후에 작업을 계속할 수는 있음). 메시지는 창이 재설정되는 시점을 보여주며, [사용 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)이 켜져 있는 경우 개발자는 `/usage-credits`를 실행하여 할당량 이상의 사용을 요청할 수 있습니다. [사용 한도 오류](/docs/en/errors#youve-hit-your-session-limit)를 참조하세요.
* **컨텍스트 또는 자동 축소 경고**: 사용 한도가 아닙니다. 대화가 모델의 최대 입력 크기에 가까워져 Claude Code가 공간을 확보하기 위해 이전 기록을 요약하는 것입니다. 개발자에게 [토큰 사용량 줄이기](#reduce-token-usage)를 안내하세요.
* **API 또는 클라우드 공급자 요금제에서 예상을 벗어난 높은 지출**: 일반적으로 정리되지 않고 오랫동안 열려 있던 세션이나 기본 모델로 남겨진 Opus로 역추적됩니다. 가장 효과적인 습관은 관련 없는 작업 간의 대화 초기화와 작업에 맞는 모델 선택입니다. 둘 다 [토큰 사용량 줄이기](#reduce-token-usage)에서 다룹니다.

### 에이전트 팀 토큰 비용

[에이전트 팀](/docs/en/agent-teams)은 각각 자체 컨텍스트 창을 갖는 여러 Claude Code 인스턴스를 생성합니다. 토큰 사용량은 활성화된 팀원 수와 각 팀원의 실행 시간 및 길이에 따라 비례합니다.

에이전트 팀 비용을 관리가능한 수준으로 유지하려면 다음을 수행하세요.

* 팀원용 모델로 Sonnet을 사용하세요. 조율 작업에 필요한 기능과 비용 간의 균형을 잡아줍니다.
* 팀 크기를 작게 유지하세요. 각 팀원이 자체 컨텍스트 창을 실행하므로 토큰 사용량은 팀 크기에 대략 비례합니다.
* 생성 프롬프트를 구체적으로 유지하세요. 팀원은 CLAUDE.md, MCP 서버 및 스킬을 자동으로 로드하지만 생성 프롬프트의 모든 내용이 시작부터 컨텍스트에 추가됩니다.
* 작업이 끝나면 팀원을 종료하세요. 활성 상태인 각 팀원은 종료되거나 세션이 끝날 때까지 토큰을 계속 소비합니다.
* 에이전트 팀은 기본적으로 비활성화되어 있습니다. [settings.json](/docs/en/settings) 또는 환경 변수에서 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`을 설정하여 활성화하세요. [에이전트 팀 활성화](/docs/en/agent-teams#enable-agent-teams)를 참조하세요.

## 토큰 사용량 줄이기

토큰 비용은 컨텍스트 크기에 비례합니다: Claude가 더 많은 컨텍스트를 처리할수록 더 많은 토큰을 사용합니다. Claude Code는 시스템 프롬프트와 같은 반복 콘텐츠의 비용을 줄여주는 [프롬프트 캐싱](/docs/en/prompt-caching)과 컨텍스트 한도에 도달할 때 대화 기록을 요약하는 자동 축소(auto-compaction)를 통해 비용을 자동으로 최적화합니다.

다음 전략은 컨텍스트를 작게 유지하고 메시지당 비용을 줄이는 데 도움이 됩니다.

### 컨텍스트 선제적 관리

`/usage`를 사용하여 현재 토큰 사용량을 확인하거나 [상태 표시줄을 구성](/docs/en/statusline#context-window-usage)하여 지속적으로 표시하세요.

* **작업 간 대화 초기화**: 관련 없는 작업으로 전환할 때 `/clear`를 사용하여 새로 시작하세요. 오래된 컨텍스트는 이후 모든 메시지에서 토큰을 낭비합니다. 초기화하기 전에 `/rename`을 사용하면 나중에 세션을 쉽게 찾을 수 있으며, `/resume`으로 다시 돌아갈 수 있습니다.
* **커스텀 축소 지침 추가**: `/compact Focus on code samples and API usage`는 요약하는 동안 보존할 내용을 Claude에게 알려줍니다. 세션 시작 시에는 요약할 대화 기록이 없으므로 `/compact`를 실행하면 `Not enough messages to compact.`가 출력됩니다.

프로젝트 루트의 CLAUDE.md 파일에서 축소 동작을 맞춤 설정할 수도 있습니다.

```markdown theme={null}
# Compact instructions

When you are using compact, please focus on test output and code changes
```

### 적절한 모델 선택

Sonnet은 대부분의 코딩 작업을 잘 처리하며 Opus보다 비용이 적게 듭니다. 심도 있는 아키텍처 결정이나 다단계 추론에는 Opus를 활용하세요. 세션 중간에 모델을 전환하려면 `/model`을 사용하고 `/config`에서 기본값을 설정하세요. 간단한 하위 에이전트 작업의 경우 [하위 에이전트 구성](/docs/en/sub-agents#choose-a-model)에 `model: haiku`를 지정하세요.

### MCP 서버 오버헤드 줄이기

MCP 도구 정의는 [기본적으로 지열 처리](/docs/en/mcp#scale-with-mcp-tool-search)되므로 Claude가 특정 도구를 사용할 때까지 도구 이름만 컨텍스트에 들어옵니다. `/context`를 실행하여 어떤 항목이 공간을 차지하는지 확인하세요.

* **가능한 경우 CLI 도구 선호**: `gh`, `aws`, `gcloud`, `sentry-cli`와 같은 도구는 도구 목록을 추가하지 않기 때문에 MCP 서버보다 컨텍스트 효율적입니다. Claude는 CLI 명령을 직접 실행할 수 있습니다.
* **사용하지 않는 서버 비활성화**: `/mcp`를 실행하여 구성된 서버를 확인하고 사용하지 않는 서버를 비활성화하세요.

### 타입 언어용 코드 인텔리전스 플러그인 설치

[코드 인텔리전스 플러그인](/docs/en/discover-plugins#code-intelligence)은 텍스트 기반 검색 대신 정확한 심볼 탐색을 제공하여 생소한 코드를 탐색할 때 불필요한 파일 읽기를 줄여줍니다. 단 한 번의 "정의로 이동" 호출이 여러 후보 파일을 읽는 grep 검색을 대체할 수 있습니다. 설치된 언어 서버는 편집 후 타입 오류를 자동으로 보고하므로 컴파일러를 실행하지 않고도 오류를 잡아냅니다.

### 훅과 스킬에 처리 위임하기

커스텀 [훅(Hooks)](/docs/en/hooks)은 Claude가 보기 전에 데이터를 전처리할 수 있습니다. Claude가 오류를 찾기 위해 10,000줄 분량의 로그 파일을 읽는 대신, 훅이 `ERROR`를 grep하여 일치하는 줄만 반환하면 컨텍스트를 수만 토큰에서 수백 토큰으로 줄일 수 있습니다.

[스킬(Skill)](/docs/en/skills)은 Claude가 탐색할 필요가 없도록 도메인 지식을 제공할 수 있습니다. 예를 들어 "codebase-overview" 스킬은 프로젝트의 아키텍처, 주요 디렉토리, 명명 규칙을 설명할 수 있습니다. Claude가 스킬을 호출하면 구조를 이해하기 위해 여러 파일을 읽느라 토큰을 쓰는 대신 이 컨텍스트를 즉시 얻게 됩니다.

예를 들어 다음 PreToolUse 훅은 실패한 내역만 보이도록 테스트 출력을 필터링합니다.

<Tabs>
  <Tab title="settings.json">
    모든 Bash 명령 전에 훅을 실행하려면 [settings.json](/docs/en/settings#settings-files)에 다음을 추가하세요.

    ```json theme={null}
    {
      "hooks": {
        "PreToolUse": [
          {
            "matcher": "Bash",
            "hooks": [
              {
                "type": "command",
                "command": "~/.claude/hooks/filter-test-output.sh"
              }
            ]
          }
        ]
      }
    }
    ```
  </Tab>

  <Tab title="filter-test-output.sh">
    훅이 이 스크립트를 호출합니다. `mkdir -p ~/.claude/hooks`로 폴더를 생성하고, 아래 스크립트를 `~/.claude/hooks/filter-test-output.sh`로 저장한 후, `chmod +x ~/.claude/hooks/filter-test-output.sh`로 실행 권한을 부여하세요. 스크립트는 명령이 테스트 러너인지 확인하고 실패한 내역만 표시하도록 수정합니다.

    ```bash theme={null}
    #!/bin/bash
    input=$(cat)
    cmd=$(echo "$input" | jq -r '.tool_input.command')

    # 테스트 실행 시 실패 내역만 필터링
    if [[ "$cmd" =~ ^(npm test|pytest|go test) ]]; then
      filtered_cmd="$cmd 2>&1 | grep -A 5 -E '(FAIL|ERROR|error:)' | head -100"
      echo "{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\",\"updatedInput\":{\"command\":\"$filtered_cmd\"}}}"
    else
      echo "{}"
    fi
    ```
  </Tab>
</Tabs>

설정을 검증하려면 `/hooks`를 실행하고 훅이 PreToolUse 아래에 나타나는지 확인하세요. `claude --debug`로 Claude Code를 시작하고 `npm test`와 같은 테스트 명령을 실행해 볼 수도 있습니다. 디버그 로그는 훅이 명령을 재작성할 때 `modified tool input keys: [command]`를 보여줍니다.

### CLAUDE.md에서 스킬로 지침 이동하기

[CLAUDE.md](/docs/en/memory) 파일은 세션 시작 시 컨텍스트에 로드됩니다. 여기에 특정 워크플로우(PR 리뷰나 데이터베이스 마이그레이션 등)에 대한 상세한 지침이 포함되어 있으면 관련 없는 작업을 수행할 때도 해당 토큰이 존재하게 됩니다. [스킬](/docs/en/skills)은 호출할 때만 온디맨드로 로드되므로 특화된 지침을 스킬로 이동하면 기본 컨텍스트가 작아집니다. 필수 요소만 포함하여 CLAUDE.md를 200줄 이하로 유지하세요.

### extended thinking 조정

Extended thinking은 복잡한 계획 수립 및 추론 작업에서 성능을 대폭 향상시키므로 기본적으로 활성화되어 있습니다. Thinking 토큰은 출력 토큰으로 청구되며 모델에 따라 기본 예산이 요청당 수만 토큰이 될 수 있습니다. 깊은 추론이 필요 없는 간단한 작업의 경우 `/effort`나 `/model`에서 [작업 투입 수준](/docs/en/model-config#adjust-effort-level)을 낮추거나, `/config`에서 thinking을 비활성화하거나, [고정 thinking 예산](/docs/en/model-config#adaptive-reasoning-and-fixed-thinking-budgets)을 사용하는 모델에서 `MAX_THINKING_TOKENS` [환경 변수](/docs/en/env-vars)(예: `MAX_THINKING_TOKENS=8000`)를 설정하여 예산을 줄여 비용을 절감할 수 있습니다. 자율 적응형 추론(adaptive reasoning) 모델은 0이 아닌 예산을 무시하므로 거기서는 작업 투입 수준을 조절하세요. 항상 extended thinking을 사용하는 Fable 5에서는 thinking 비활성화가 제공되지 않습니다.

### 출력이 긴 작업을 하위 에이전트에 위임

테스트 실행, 문서 가져오기, 로그 파일 처리는 상당한 컨텍스트를 소비할 수 있습니다. 이러한 작업들을 [하위 에이전트](/docs/en/sub-agents#isolate-high-volume-operations)에 위임하여 출력이 긴 정보는 하위 에이전트 컨텍스트에 두고 요약만 메인 대화로 반환받으세요.

### 에이전트 팀 비용 관리

에이전트 팀은 각 팀원이 자체 컨텍스트 창을 유지하고 별도의 Claude 인스턴스로 실행되므로 플랜 모드에서 실행할 때 표준 세션보다 약 7배 더 많은 토큰을 사용합니다. 팀원당 토큰 사용량을 제한하려면 팀 작업을 작고 자체 완결적으로 유지하세요. 세부 정보는 [에이전트 팀](/docs/en/agent-teams)을 참조하세요.

### 구체적인 프롬프트 작성

"이 코드베이스를 개선해 줘"와 같은 모호한 요청은 광범위한 스캔을 유발합니다. "auth.ts의 로그인 함수에 입력 검증을 추가해 줘"와 같이 구체적인 요청을 통해 Claude가 최소한의 파일 읽기로 효율적으로 작업하게 하세요.

### 복잡한 작업에서 효율적으로 일하기

길거나 복잡한 작업의 경우 다음 습관을 통해 잘못된 방향으로 진행되어 토큰이 낭비되는 것을 방지할 수 있습니다.

* **복잡한 작업에는 플랜 모드 사용**: 구현 전에 Shift+Tab을 눌러 [플랜 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)로 전환하세요. Claude가 코드베이스를 탐색하고 승인을 위한 접근 방식을 제안하여 초기 방향이 잘못되었을 때 비싼 재작업이 일어나는 것을 방지합니다.
* **조기에 방향 수정**: Claude가 잘못된 방향으로 이동하기 시작하면 Escape를 눌러 즉시 중지하세요. 대화와 코드를 이전 체크포인트로 복원하려면 `/rewind`를 사용하거나 Escape를 두 번 누르세요.
* **검증 목표 제공**: 프롬프트에 테스트 케이스를 포함하거나, 스크린샷을 붙여넣거나, 예상 출력을 정의하세요. Claude가 스스로 작업을 검증할 수 있으면 수정 요청을 하기 전에 문제를 포착합니다.
* **점진적 테스트**: 파일 하나를 작성하고 테스트한 후 계속 진행하세요. 이렇게 하면 수정 비용이 적을 때 문제를 조기에 포착할 수 있습니다.

## 백그라운드 토큰 사용량

Claude Code는 유휴 상태일 때도 일부 백그라운드 기능에 토큰을 사용합니다.

* **대화 요약**: `claude --resume` 기능을 위해 이전 대화를 요약하는 백그라운드 작업
* **명령 처리**: `/usage`와 같은 일부 명령은 상태 확인을 위한 요청을 생성할 수 있습니다.

이러한 백그라운드 프로세스는 활발한 상호작용이 없더라도 소량의 토큰(일반적으로 세션당 \$0.04 미만)을 소비합니다.

## 긴 세션에서 사용량이 증가하는 이유

수천 시간 동안 열려 있던 세션은 작업량보다 훨씬 더 많은 요금제 한도를 사용할 수 있습니다. 주된 이유는 다음과 같습니다.

* **긴 컨텍스트**: Claude Code는 매 메시지마다 전체 대화를 다시 전송하므로, 하루 종일 열려 있던 세션의 한 줄짜리 질문도 질문 한 줄만이 아닌 전체 대화의 토큰을 소비합니다. 컨텍스트를 작게 유지하는 방법은 [컨텍스트 선제적 관리](#manage-context-proactively)를 참조하세요.
* **캐시 미스 (Cache misses)**: [캐시 수명](/docs/en/prompt-caching#cache-lifetime)보다 길게 쉰 후의 첫 메시지는 캐시 미스를 발생시키고 전체 컨텍스트를 다시 처리합니다. 수명은 구독에서 1시간이며, [사용 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)을 사용하는 순간 5분으로 줄어듭니다. API 키나 클라우드 공급자에서는 기본 5분입니다.
* **예약된 작업**: [예약된 작업](/docs/en/scheduled-tasks)은 세션이 유휴 상태일 때도 지정된 간격에 실행되어 매번 전체 컨텍스트를 전송합니다.
* **에이전트 팀원**: 활성 상태의 각 [팀원](#agent-team-token-costs)은 종료될 때까지 토큰을 계속 소비합니다.
* **축소(Compaction)**: `/compact`는 요약 대상 대화를 읽으므로 [대용량 컨텍스트를 축소하는 것](/docs/en/prompt-caching#compacting-the-conversation) 자체가 큰 요청이 됩니다. 대화를 계속 이어가는 대신 새로 시작하고자 할 때는 `/clear`를 사용하면 비용이 들지 않습니다.

Pro, Max, Team 또는 Enterprise 요금제에서 `/usage` 세부 내역은 긴 컨텍스트나 캐시 미스처럼 최근 사용량의 10% 이상을 차지하는 동작을 각각 줄이기 위한 팁과 함께 지적해 줍니다.

## Claude Code 동작 변경에 대한 이해

Claude Code는 비용 보고를 포함하여 기능 작동 방식을 변경하는 업데이트를 정기적으로 받습니다. 현재 버전을 확인하려면 `claude --version`을 실행하세요.

특정 계정에 대한 청구 관련 문의는 제품 내 메시징을 통해 Anthropic 지원팀에 문의하세요.

* **구독 요금제** (Pro, Max, Team, Enterprise): [claude.ai](https://claude.ai)에 로그인하고 좌측 하단의 이름 이니셜을 클릭한 후 **Get help**를 선택하세요.
* **Console (API) 청구**: [platform.claude.com](https://platform.claude.com)에 로그인하고 이니셜을 클릭한 후 **Get help**를 선택하세요.

각 요금제에서 사람 상담원에게 연결할 수 있는 대상을 포함한 전체 흐름은 [How to get support](https://support.claude.com/en/articles/9015913-how-to-get-support)를 참조하세요.
