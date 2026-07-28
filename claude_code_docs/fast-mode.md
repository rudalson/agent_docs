> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 패스트 모드(Fast Mode)로 응답 속도 올리기

> 패스트 모드를 전환하여 Claude Code에서 더 빠른 Opus 응답을 받으세요.

<Note>
  패스트 모드는 [리서치 프리뷰(research preview)](#research-preview) 상태입니다. 피드백에 따라 기능, 가격 및 이용 가능 여부가 변경될 수 있습니다.
</Note>

패스트 모드는 Claude Opus를 위한 고속 구성으로, 토큰당 더 높은 비용을 지불하는 대신 모델 응답 속도를 최대 2.5배까지 향상시킵니다. 빠른 반복 작업이나 라이브 디버깅과 같이 대화형 작업에 속도가 필요할 때는 `/fast`로 켜고, 지연 시간보다 비용이 더 중요할 때는 끄세요.

패스트 모드는 서로 다른 별개의 모델이 아닙니다. 비용 효율성보다 속도를 우선시하는 별도의 API 구성으로 Claude Opus를 사용하는 방식입니다. 빠른 응답 속도와 함께 동일한 품질과 기능을 제공받게 됩니다. 패스트 모드는 Opus 4.8 및 Opus 4.7에서 지원됩니다. Sonnet, Haiku 또는 기타 모델에서는 이용할 수 없습니다.

<Warning>
  Opus 4.7의 패스트 모드는 2026년 6월 25일부로 폐지(deprecated)되었으며, 2026년 7월 24일에 제거될 예정입니다. 제거된 후 Opus 4.7의 패스트 모드 요청은 오류를 반환하며 표준 Opus 4.7로 폴백되지 않습니다. 속도 향상을 유지하려면 Opus 4.8로 마이그레이션하세요.
</Warning>

알아두어야 할 사항:

* 대화형 Claude Code CLI에서 패스트 모드를 전환하려면 `/fast`를 사용하세요. 패스트 모드는 VS Code 확장 프로그램에서는 지원되지 않습니다.
* MTok(백만 토큰) 입력/출력당 패스트 모드 가격은 Opus 4.8에서 \$10/\$50, Opus 4.7에서 \$30/\$150입니다.
* 구독 요금제(Pro/Max/Team/Enterprise) 및 Claude Console의 모든 Claude Code 사용자가 이용할 수 있습니다.
* 구독 요금제(Pro/Max/Team/Enterprise) 사용자의 경우 패스트 모드는 사용 크레딧(usage credits)을 통해서만 이용 가능하며 구독 기본 처리율 제한에는 포함되지 않습니다.

## 패스트 모드 전환하기

다음 방법 중 하나로 패스트 모드를 전환하세요.

* `/fast`를 입력하고 Tab을 눌러 켜거나 끄기
* [사용자 설정 파일](/docs/en/settings)에 `"fastMode": true` 설정

기본적으로 대화형 세션에서 패스트 모드를 켜면 다음 세션 전반으로 유지됩니다. {/* min-version: 2.1.205 */}`-p` 플래그를 사용하는 [비대화형 모드](/docs/en/headless)에서 `/fast`는 [`--settings`](/docs/en/cli-reference#cli-flags) 값에 패스트 모드가 포함된 세션(예: `claude -p --settings '{"fastMode": true}'`)에서만 작동합니다. 이때 해당 전환은 해당 세션에만 적용되며 기본값으로 저장되지 않으며, 다른 비대화형 세션에서는 패스트 모드를 사용할 수 없다고 보고합니다. 각 세션마다 초기화되도록 구성할 수 있습니다. 세부 정보는 [세션별 자발적 수동 옵트인 요구하기](#require-per-session-opt-in)를 참조하세요.

최고의 비용 효율성을 위해 세션 중간에 전환하기보다는 세션 시작 부분에서 패스트 모드를 활성화하세요. 세부 정보는 [비용 절충 요소 이해하기](#understand-the-cost-tradeoff)를 참조하세요.

패스트 모드를 활성화할 때:

* 다른 모델을 사용 중인 경우 Claude Code가 자동으로 Opus로 전환합니다.
* "Fast mode ON"이라는 확인 메시지가 표시됩니다.
* 패스트 모드가 활성화되어 있는 동안 프롬프트 옆에 작은 `↯` 아이콘이 나타납니다.
* 언제든지 `/fast`를 다시 실행하여 패스트 모드가 켜져 있는지 꺼져 있는지 확인하세요.

`/fast`로 패스트 모드를 비활성화하더라도 Opus 모델 상태를 유지합니다. 모델이 이전 모델로 되돌아가지 않습니다. 다른 모델로 전환하려면 `/model`을 사용하세요.

패스트 모드를 지원하지 않는 모델로 전환하면 패스트 모드가 꺼집니다. {/* min-version: 2.1.208 */}지원되는 Opus 모델로 다시 전환할 때 저장된 패스트 모드 기본 설정이 켜짐 상태인 경우(새 세션이 기본적으로 시작되는 설정) 패스트 모드가 다시 켜집니다. [세션별 옵트인](#require-per-session-opt-in)이 구성되어 있으면 다시 전환하더라도 패스트 모드가 자동으로 켜지지 않으므로 `/fast`를 실행하여 다시 활성화해야 합니다. 저장된 기본 설정이 꺼짐인 세션에서는 패스트 모드가 활성화되지 않으며, 활성화될 때마다 `↯` 아이콘과 `Fast mode ON` 확인 메시지가 나타납니다. v2.1.208 이전에는 다시 전환한 후 `/fast`를 다시 실행할 때까지 패스트 모드가 켜지지 않았습니다.

Opus 4.8은 Claude Code v2.1.154 이상에서 패스트 모드 기본값입니다. v2.1.142부터 v2.1.153까지 패스트 모드는 Opus 4.7을 기본값으로 사용했습니다.

## 비용 절충 요소 이해하기

패스트 모드는 표준 Opus보다 토큰당 가격이 높으며 배율은 모델마다 다릅니다.

| 모델 | 입력 (MTok) | 출력 (MTok) |
| :--- | :--- | :--- |
| Opus 4.8 | \$10 | \$50 |
| Opus 4.7 | \$30 | \$150 |

패스트 모드 가격은 전체 1M 토큰 컨텍스트 창에 걸쳐 동일하게 적용됩니다. 비교를 위한 표준 Opus 요금은 [Claude 가격 책정 레퍼런스](https://platform.claude.com/docs/en/about-claude/pricing)를 참조하세요.

대화에서 패스트 모드를 처음 활성화할 때 전체 대화 컨텍스트에 대해 캐시되지 않은 입력 토큰 가격 전액을 지불하게 됩니다. 대화가 깊어질수록 이 비용이 증가하므로 처음부터 패스트 모드를 활성화하는 것이 더 저렴합니다. 비용은 대화당 한 번만 적용되므로 나중에 패스트 모드를 껐다 켜도 반복되지 않습니다. 매커니즘은 [패스트 모드가 프롬프트 캐시와 상호작용하는 방식](/docs/en/prompt-caching#turning-on-fast-mode)을 참조하세요.

## 패스트 모드 사용 시점 결정하기

패스트 모드는 비용보다 응답 지연 시간이 중요한 대화형 작업에 가장 적합합니다.

* 코드 변경에 대한 빠른 반복 작업
* 라이브 디버깅 세션
* 촉박한 마감 기한이 있는 시급한 작업

표준 모드는 다음에 더 적합합니다.

* 속도가 덜 중요한 긴 자율 작업
* 배치 처리 또는 CI/CD 파이프라인
* 비용에 민감한 워크로드

### 패스트 모드 vs 작업 투입 수준(Effort level)

패스트 모드와 작업 투입 수준(effort level)은 모두 응답 속도에 영향을 주지만 방식이 다릅니다.

| 설정 | 효과 |
| :--- | :--- |
| **패스트 모드** | 동일한 모델 품질, 더 낮은 지연 시간, 더 높은 비용 |
| **낮은 작업 투입 수준** | 사고(thinking) 시간 감소, 더 빠른 응답, 복잡한 작업 시 품질 저하 가능성 |

둘을 결합할 수도 있습니다: 명확하고 간단한 작업에서 최대 속도를 얻으려면 낮은 [작업 투입 수준](/docs/en/model-config#adjust-effort-level)과 함께 패스트 모드를 사용하세요.

## 요구 사항

패스트 모드를 사용하려면 다음 조건을 모두 충족해야 합니다.

* **Anthropic API 또는 구독 전용**: 패스트 모드는 Anthropic Console API와 사용 크레딧을 이용하는 Claude 구독 요금제에서만 가능합니다. Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry, 또는 Claude Platform on AWS에서는 이용할 수 없습니다.
* **사용 크레딧(Usage credits) 활성화**: 계정에 사용 크레딧이 활성화되어 있어야 요금제 기본 제공 사용량을 넘어 결제할 수 있습니다. 개인 계정의 경우 [Console 결제 설정](https://platform.claude.com/settings/billing)에서 켜세요. Team 및 Enterprise의 경우 관리자가 조직 전체에 대해 사용 크레딧을 켜야 합니다.

<Note>
  요금제 기본 제공 사용량이 남아 있더라도 패스트 모드 사용량은 사용 크레딧에서 직접 차감됩니다. 즉, 패스트 모드 토큰은 요금제 포함 사용량에 반영되지 않으며 첫 번째 토큰부터 패스트 모드 요금으로 청구됩니다.
</Note>

* **Team 및 Enterprise 조직의 소유자 승인**: 패스트 모드는 Team 및 Enterprise 조직에서 기본적으로 비활성화되어 있습니다. 소유자(Owner)가 사용자가 접근하기 전에 직접 [조직의 패스트 모드를 활성화](#enable-fast-mode-for-your-organization)해야 합니다.

<Note>
  조직에서 패스트 모드가 활성화되지 않은 경우 `/fast` 명령을 실행하면 "Fast mode has been disabled by your organization."이 표시됩니다. 조직의 [`availableModels`](/docs/en/model-config#restrict-model-selection) 허용 목록에서 패스트 모드 Opus 모델이 제외되어 있으면 `/fast`는 "is not in your organization's allowed models"로 거부됩니다. 단, 패스트 모드를 지원하고 허용된 Opus 모델에서 이미 실행 중인 세션은 예외로 모델을 전환하지 않고 현재 모델에서 패스트 모드를 활성화합니다.
</Note>

### 조직의 패스트 모드 활성화하기

패스트 모드를 활성화하는 위치는 조직이 사용하는 제품에 따라 다릅니다.

* **Console** (API 고객): 관리자가 [Claude Code 환경설정](https://platform.claude.com/claude-code/preferences)에서 활성화
* **Claude AI** (Team 및 Enterprise): 소유자가 [Admin Settings > Claude Code](https://claude.ai/admin-settings/claude-code)에서 활성화

패스트 모드를 완전히 비활성화하는 또 다른 옵션은 `CLAUDE_CODE_DISABLE_FAST_MODE=1`을 설정하는 것입니다. [환경 변수](/docs/en/env-vars)를 참조하세요.

### 프록시 및 LLM 게이트웨이 환경에서 패스트 모드 사용하기

패스트 모드를 제공하기 전에 Claude Code는 `api.anthropic.com`으로 직접 요청을 보내 조직의 패스트 모드 이용 가능 여부를 확인합니다. 이 점검은 [`ANTHROPIC_BASE_URL`](/docs/en/llm-gateway-connect#set-the-base-url-and-credential)을 따르지 않으므로, Claude 트래픽을 [LLM 게이트웨이](/docs/en/llm-gateway)로 라우팅하고 `api.anthropic.com`으로의 직접 아웃바운드를 차단하는 네트워크에서는 추론 요청이 작동하더라도 이 점검이 실패합니다. 단, 구성된 [HTTP 프록시](/docs/en/network-config#proxy-configuration)를 사용하므로 차단된 점검은 프록시를 통해서도 `api.anthropic.com`에 도달할 수 없는 경우에만 실패합니다.

점검에 실패하면 `/fast`는 "Fast mode unavailable due to network connectivity issues"를 보고하며 조직이 패스트 모드를 활성화한 경우에도 요청이 표준 속도로 실행됩니다. 과거에 성공했던 점검은 캐시된 결과에서 계속 작동하므로 차단된 점검은 주로 신규 설치에 영향을 줍니다.

개방된 네트워크에서도 점검이 `api.anthropic.com`에 도달했지만 Anthropic이 거부하는 자격 증명을 제시하면 동일한 연결 메시지가 나타납니다. 확인된 키가 [`ANTHROPIC_API_KEY`](/docs/en/llm-gateway-connect#set-the-base-url-and-credential)에 있거나 [`apiKeyHelper`](/docs/en/settings#available-settings)가 생성한 게이트웨이 발급 자격 증명인 세션은 해당 키로 점검을 보내며 거부된 요청은 연결 실패로 보고됩니다.

패스트 모드를 복원하려면 네트워크 차단이 원인인 경우 `api.anthropic.com`으로의 직접 출력을 허용 목록에 추가하거나 점검 실패 방식에 맞는 변수를 설정하세요.

* `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS=1`은 실패한 점검을 이용 가능한 것으로 취급하면서도 "disabled by your organization" 응답을 존중합니다. 네트워크가 연결을 거부하거나 Anthropic이 게이트웨이 자격 증명을 거부할 때 사용하세요.
* `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK=1`은 점검을 완전히 건너끕니다. 네트워크가 요청을 거부하지 않고 가로채는 경우에 사용하세요.

조직에서 패스트 모드를 활성화한 경우에도 연결 메시지 대신 "Fast mode has been disabled by your organization"을 보고하는 두 가지 게이트웨이 구성:

* [`ANTHROPIC_AUTH_TOKEN`](/docs/en/llm-gateway-connect#set-the-base-url-and-credential)만으로 인증하는 세션은 점검을 건너끕니다: claude.ai 로그인이나 Anthropic API 키 없이, 성공한 캐시 점검 없이 Claude Code는 요청을 보내지 않고 패스트 모드가 조직에 의해 비활성화된 것으로 처리합니다.
* 점검을 가로채서 자체 페이지로 응답하는 프록시(예: HTTP 200 차단 페이지를 반환하는 TLS 검사 프록시)는 조직이 패스트 모드를 비활성화했음을 나타내는 응답으로 읽힙니다.

두 경우 모두 `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK=1`을 설정하여 패스트 모드를 복원하세요. `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS`는 실패한 점검만 우회하고 두 경우 모두 비활성화 응답을 생성하므로 두 경우에 적용되지 않습니다. 직접 출력을 허용 목록에 추가하는 것은 요청을 전혀 보내지 않는 전달자 토큰(bearer-token) 케이스에 도움이 되지 않습니다.

이 변수들은 클라이언트 측 점검에만 영향을 줍니다. 조직에서 패스트 모드를 비활성화한 경우 변수 설정 여부와 관계없이 API가 패스트 모드 요청을 거부합니다.

`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`을 설정하면 가용성 점검도 억제됩니다. 이전에 캐시된 성공 점검이 없으면 `/fast`는 "Fast mode is currently unavailable"을 보고하며, 두 스킵 변수 모두 해당 구성에서 패스트 모드를 복원합니다.

### 세션별 자발적 수동 옵트인(Opt-in) 요구하기

기본적으로 사용자가 대화형 세션에서 켜는 패스트 모드는 다음 세션으로 유지됩니다. 이를 변경하려면 임의의 [설정 파일](/docs/en/settings#settings-files)에 `fastModePerSessionOptIn`을 `true`로 설정하세요. 그러면 각 세션이 패스트 모드가 꺼진 상태로 시작되며 사용자가 `/fast`를 통해 명시적으로 활성화해야 합니다. [Team](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=fast_mode_teams#team-&-enterprise) 또는 [Enterprise](https://anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=fast_mode_enterprise) 요금제의 소유자는 [서버 관리 대상 설정](/docs/en/server-managed-settings)을 통해 조직 전체에 이를 배포할 수 있습니다.

```json theme={null}
{
  "fastModePerSessionOptIn": true
}
```

이는 사용자가 여러 동시 세션을 실행하는 조직에서 비용을 제어하는 데 유용합니다. 사용자는 속도가 필요할 때 여전히 `/fast`로 패스트 모드를 활성화할 수 있지만 각 새 세션 시작 시 초기화됩니다. 사용자의 패스트 모드 선호도는 여전히 저장되므로 이 설정을 제거하면 기본 유지 동작이 복원됩니다.

## 처리율 제한(Rate Limits) 다루기

패스트 모드는 표준 Opus와 별도의 처리율 제한을 가집니다. Opus 4.8 및 Opus 4.7의 패스트 모드는 동일한 처리율 제한 풀을 공유합니다. 패스트 모드 처리율 제한에 도달하거나 사용 크레딧이 다 소모되면 다음과 같이 작동합니다.

1. 패스트 모드가 표준 속도로 자동 폴백됩니다.
2. 쿨다운(cooldown)을 나타내기 위해 `↯` 아이콘이 회색으로 바뀝니다.
3. 표준 속도 및 요금으로 작업을 계속합니다.
4. 쿨다운이 만료되면 패스트 모드가 자동으로 다시 활성화됩니다.

쿨다운을 기다리지 않고 패스트 모드를 수동으로 비활성화하려면 `/fast`를 다시 실행하세요.

## 리서치 프리뷰 안내

패스트 모드는 리서치 프리뷰 기능입니다. 다음 사항을 참고하세요.

* 피드백에 따라 기능이 변경될 수 있습니다.
* 이용 가능 여부 및 가격은 변경될 수 있습니다.
* 기본 API 구성이 진화할 수 있습니다.

일반적인 Anthropic 지원 채널을 통해 문제나 피드백을 보고해 주세요.

## 참고 항목

* [모델 구성](/docs/en/model-config): 모델 전환 및 작업 투입 수준 조정
* [효율적인 비용 관리하기](/docs/en/costs): 토큰 사용량 추적 및 비용 절감
* [상태 표시줄 구성](/docs/en/statusline): 모델 및 컨텍스트 정보 표시
