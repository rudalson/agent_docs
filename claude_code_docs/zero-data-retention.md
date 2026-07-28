> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 제로 데이터 보존

> 범위, 비활성화된 기능, 활성화 요청 방법을 포함하여 Claude for Enterprise의 자격이 있는 계정에서 사용할 수 있는 Claude Code용 제로 데이터 보존(Zero Data Retention, ZDR)에 대해 알아봅니다.

Claude Code용 제로 데이터 보존(Zero Data Retention, ZDR)은 Claude for Enterprise의 자격을 갖춘 계정에서 사용할 수 있습니다. ZDR이 활성화되면 법률 준수 또는 오용 대응에 필요한 경우를 제외하고 Claude Code 세션 중에 생성된 프롬프트 및 모델 응답이 실시간으로 처리되며 응답이 반환된 후 Anthropic에 저장되지 않습니다.

<Note>
  ZDR은 표준 Claude for Enterprise 플랜에 포함되어 있지 않으며 관리자 설정에서 활성화할 수 없습니다. 자격을 갖춘 계정에서 이용할 수 있으며 Anthropic의 별도 활성화가 필요합니다. 조직에 ZDR이 필요한 경우 [영업팀에 문의](https://www.anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=zero_data_retention_request)하거나 Anthropic 계정 팀에 문의하여 자격을 확인하세요.
</Note>

Claude for Enterprise의 ZDR을 통해 기업 고객은 제로 데이터 보존으로 Claude Code를 사용하고 관리 기능을 이용할 수 있습니다:

* 사용자별 비용 제어
* [분석(Analytics)](/docs/en/analytics) 대시보드
* [서버 관리형 설정](/docs/en/server-managed-settings)
* 감사 로그

Claude for Enterprise의 Claude Code ZDR은 Anthropic의 직접 플랫폼에만 적용됩니다. Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry의 Claude 배포에 대해서는 해당 플랫폼의 데이터 보존 정책을 참조하세요.

## ZDR 범위

ZDR은 Claude for Enterprise의 Claude Code 추론에 적용됩니다.

<Warning>
  ZDR은 조직별로 활성화됩니다. 새 조직마다 Anthropic 계정 팀이 ZDR을 별도로 활성화해야 합니다. 동일한 계정 하위에 생성된 새 조직에 ZDR이 자동으로 적용되지는 않습니다. 새 조직에 대해 ZDR을 활성화하려면 계정 팀에 문의하세요.
</Warning>

### Claude Code 트래픽을 ZDR 조직으로 라우팅

ZDR은 ZDR이 활성화된 조직에 인증하는 요청에 적용됩니다. 개발자가 개인 계정이나 다른 조직의 API 키로 Claude Code에 로그인하는 경우 해당 세션은 적용되지 않습니다. ZDR 조직으로 로그인을 제한하려면 `forceLoginMethod` 및 `forceLoginOrgUUID` 관리형 설정을 배포하세요; [조직으로 로그인 제한](/docs/en/authentication#restrict-login-to-your-organization)을 참조하세요.

### ZDR이 적용되는 대상

ZDR은 Claude for Enterprise의 Claude Code를 통해 이루어지는 모델 추론 호출에 적용됩니다. 터미널에서 Claude Code를 사용할 때 전송하는 프롬프트와 Claude가 생성하는 응답은 Anthropic에 보존되지 않습니다. 이는 ZDR 조직에 제공되는 모든 모델에 적용됩니다. 일부 모델은 데이터 보존이 필요하여 ZDR 하에서 이용할 수 없습니다; [ZDR 하에서의 모델 제공 여부](#model-availability-under-zdr)를 참조하세요.

### ZDR이 적용되지 않는 대상

ZDR이 활성화된 조직이라도 다음 항목에는 ZDR이 확장되지 않습니다. 이 기능들은 [표준 데이터 보존 정책](/docs/en/data-usage#data-retention)을 따릅니다:

| 기능                  | 세부 정보                                                                                                                                                                                                                                                     |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Chat on claude.ai        | Claude for Enterprise 웹 인터페이스를 통한 채팅 대화는 ZDR이 적용되지 않습니다.                                                                                                                                                                  |
| Cowork                   | Cowork 세션은 ZDR이 적용되지 않습니다.                                                                                                                                                                                                                     |
| Claude Code Analytics    | 프롬프트나 모델 응답은 저장하지 않지만 계정 이메일 및 사용 통계와 같은 생산성 메타데이터를 수집합니다. ZDR 조직에서는 기여도 메트릭을 사용할 수 없으며, [분석 대시보드](/docs/en/analytics)에는 사용량 메트릭만 표시됩니다. |
| 사용자 및 시트 관리 | 계정 이메일 및 시트 할당과 같은 관리 데이터는 표준 정책에 따라 보존됩니다.                                                                                                                                                                                       |
| 써드파티 통합 | 써드파티 도구, MCP 서버 또는 기타 외부 통합에 의해 처리되는 데이터는 ZDR이 적용되지 않습니다. 해당 서비스의 데이터 처리 관행을 독립적으로 검토하세요.                                                                                       |

## ZDR 하에서 비활성화되는 기능

Claude for Enterprise의 Claude Code 조직에 ZDR이 활성화되면 프롬프트 또는 완료 저장이 필요한 특정 기능이 백엔드 수준에서 자동으로 비활성화됩니다:

| 기능                                                           | 이유                                                                                      |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [웹상 Claude Code](/docs/en/claude-code-on-the-web)              | 대화 기록의 서버 측 저장이 필요합니다.                                       |
| Desktop 앱의 [클라우드 세션](/docs/en/desktop#cloud-sessions) | 프롬프트 및 완료를 포함하는 영구적인 세션 데이터가 필요합니다.                     |
| [아티팩트](/docs/en/artifacts)                                        | Anthropic이 운영하는 인프라에 게시된 페이지 콘텐츠를 저장해야 합니다.               |
| 피드백 제출 (`/feedback`, `/bug`, `/share`)               | 피드백을 제출하면 대화 데이터가 Anthropic으로 전송됩니다.                                   |
| [Remote Control](/docs/en/remote-control)                              | 장치 간 대화를 동기화하기 위해 Anthropic 서버에 세션 트랜스크립트를 저장합니다. |

이러한 기능은 클라이언트 측 표시와 무관하게 백엔드에서 차단됩니다. 시작 시 Claude Code 터미널에 비활성화된 기능이 표시되는 경우, 이를 사용하려고 시도하면 조직의 정책이 해당 작업을 허용하지 않음을 나타내는 오류가 반환됩니다.

향후 기능도 프롬프트나 완료 저장이 필요한 경우 비활성화될 수 있습니다.

### ZDR 하에서의 모델 제공 여부

Claude Fable 5는 제로 데이터 보존이 활성화된 조직에서는 사용할 수 없습니다. 이 모델 클래스는 [데이터 보존이 필요](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)하므로 ZDR 조직의 요청은 제공할 수 없습니다. ZDR 조직의 경우 이 모델은 `/model` 선택기에 없거나 ZDR 비활성화가 필요하다는 안내와 함께 비활성화되어 표시되며, 서버는 클라이언트 구성과 상관없이 요청을 거부합니다.

다른 모델은 ZDR 하에서도 계속 이용할 수 있습니다. Fable 5는 기본 모델이 아니며, Fable 5가 제공되는 곳에서 Fable 5로 확인되는 `best` 별칭은 ZDR 조직을 포함하여 제공되지 않는 조직에서는 Opus로 확인됩니다.

## 정책 위반 시 데이터 보존

ZDR이 활성화되어 있더라도 Anthropic은 법률에 의해 요구되거나 사용 정책 위반을 해결해야 하는 경우 데이터를 보존할 수 있습니다. 세션이 정책 위반으로 플래그 지정된 경우, Anthropic의 표준 ZDR 정책에 따라 Anthropic은 연결된 입력 및 출력을 최대 2년 동안 보존할 수 있습니다.

## ZDR 요청하기

Claude for Enterprise에서 Claude Code용 ZDR을 요청하려면 [영업팀에 문의](https://www.anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=zero_data_retention_request)하거나 Anthropic 계정 팀에 문의하세요. 계정 팀이 내부적으로 요청을 제출하며, Anthropic은 자격을 확인한 후 조직에 ZDR을 검토 및 활성화합니다. 모든 활성화 작업은 감사 로그에 기록됩니다.

현재 종량제 API 키를 통해 Claude Code용 ZDR을 사용 중인 경우, Claude for Enterprise로 전환하여 Claude Code용 ZDR을 유지하면서 관리 기능을 사용할 수 있습니다. 마이그레이션을 조율하려면 계정 팀에 문의하세요.
