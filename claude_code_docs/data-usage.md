> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 데이터 사용법

> Claude에 대한 Anthropic의 데이터 사용 정책에 대해 알아보세요.

## 데이터 정책

### 데이터 학습 정책

**개인 사용자 (Free, Pro 및 Max 플랜)**:
이 설정이 켜져 있을 때(이 계정에서 Claude Code를 사용할 때 포함) 새 모델을 학습시키는 데 Free, Pro 및 Max 계정의 데이터를 사용할 수 있도록 허용하는 옵션을 제공합니다.

**상업용 사용자**: (Team 및 Enterprise 플랜, API, 제3자 플랫폼, Claude Gov) 기존 정책을 유지합니다: Anthropic은 고객이 모델 개선을 위해 데이터를 제공하기로 선택한 경우(예: [개발 파트너 프로그램](https://support.claude.com/en/articles/11174108-about-the-development-partner-program))를 제외하고, 상업용 약관에 따라 Claude Code로 전송된 코드나 프롬프트를 사용하여 생성형 모델을 학습시키지 않습니다.

### 개발 파트너 프로그램

[개발 파트너 프로그램](https://support.claude.com/en/articles/11174108-about-the-development-partner-program) 등을 통해 학습에 사용될 자료를 당사에 제공하는 방법에 명시적으로 동의(opt-in)한 경우, 당사는 제공된 자료를 사용하여 모델을 학습시킬 수 있습니다. 조직 관리자는 조직을 위해 개발 파트너 프로그램에 명시적으로 동의할 수 있습니다. 이 프로그램은 Anthropic 1차 파티 API에서만 사용할 수 있으며, Amazon Bedrock이나 Google Cloud의 Agent Platform 사용자는 사용할 수 없습니다.

### `/feedback` 명령을 사용한 피드백

`/feedback` 명령을 사용하여 Claude Code에 대한 피드백을 보내기로 선택한 경우, 당사는 귀하의 피드백을 제품 및 서비스 개선에 사용할 수 있습니다. `/feedback`을 통해 공유되거나 동일한 경로로 보고되는 `/bug` 및 `/share`를 통해 공유된 트랜스크립트는 5년 동안 보관됩니다.

### 세션 품질 설문조사

Claude Code에서 "How is Claude doing this session?"(이번 세션에서 Claude의 수행 능력이 어떠한가요?)이라는 프롬프트가 표시될 때, "Dismiss"를 선택하는 것을 포함하여 이 설문에 응답하면 귀하의 평점만 기록됩니다. 평점 프롬프트 자체의 일부로서 대화 트랜스크립트, 입력, 출력 또는 기타 세션 데이터를 수집하거나 저장하지 않습니다. 좋아요/싫어요 피드백이나 `/feedback` 보고서와 달리, 이 세션 품질 설문조사는 단순한 제품 만족도 지표입니다.

평점 프롬프트 후에 "Can Anthropic look at your session transcript to help us improve Claude Code?"(Anthropic이 Claude Code 개선을 돕기 위해 귀하의 세션 트랜스크립트를 살펴보아도 될까요?)라는 별도의 후속 질문이 표시될 수 있습니다. 이는 평점과 구분되는 선택적 2단계입니다:

* **Yes**: 대화 트랜스크립트, 서브에이전트 트랜스크립트, 디스크의 원본 세션 로그 파일을 Anthropic에 업로드합니다. 알려진 API 키 및 토큰 패턴은 업로드 전에 가려집니다(redact). 소스 코드, 파일 내용 및 기타 대화 내용은 있는 그대로 업로드됩니다. 공유된 트랜스크립트는 최대 6개월 동안 보관됩니다. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 및 로그인된 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션에서는 Yes를 선택하면 업로드 대신 동일한 페이로드를 `~/.claude/feedback-bundles/` 아래의 로컬 아카이브에 기록하며, 사용자가 해당 파일을 전달하기 전까지는 아무것도 시스템 밖으로 나가지 않습니다.
* **No**: 아무것도 전송하지 않고 거부합니다.
* **Don't ask again**: 거부하고 향후 세션에서 이 후속 프롬프트가 표시되지 않도록 합니다.

명시적으로 **Yes**를 선택하지 않는 한 아무것도 업로드되지 않습니다. [데이터 보관 기간이 0인(Zero Data Retention)](/docs/en/zero-data-retention) 조직, 조직 정책에 의해 제품 피드백이 비활성화된 조직, 또는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`이 설정된 조직에는 이 후속 질문이 표시되지 않습니다. 평점 프롬프트 제출 후 세션 트랜스크립트를 제출하는 것을 포함하여 이 설문조사에 대한 귀하의 응답은 데이터 학습 기본 설정에 영향을 미치지 않으며 AI 모델 학습에 사용될 수 없습니다.

이 설문조사를 비활성화하려면 `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`로 설정하세요. 또한 `DISABLE_TELEMETRY`, `DO_NOT_TRACK` 또는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`이 설정되어 있으면 설문조사가 비활성화됩니다. 필수적이지 않은 트래픽을 차단하지만 자체 [OpenTelemetry 수집기](/docs/en/monitoring-usage)를 통해 설문 응답을 캡처하는 조직은 `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL=1`을 설정하여 설문조사를 다시 활성화할 수 있습니다. 그러면 설문조사는 구성된 수집기에만 평점을 기록합니다. 트랜스크립트 공유 후속 질문 및 기타 모든 Anthropic 바운드 피드백 트래픽은 비활성화 상태를 유지합니다. 비활성화하는 대신 빈도를 제어하려면 설정 파일의 [`feedbackSurveyRate`](/docs/en/settings#available-settings)를 `0`과 `1` 사이의 확률로 설정하세요.

### 데이터 보관 (Data retention)

Anthropic은 계정 유형 및 기본 설정에 따라 Claude Code 데이터를 보관합니다.

**개인 사용자 (Free, Pro 및 Max 플랜)**:

* 모델 개선을 위한 데이터 사용을 허용한 사용자: 모델 개발 및 안전성 향상을 지원하기 위해 5년 보관 기간 적용
* 모델 개선을 위한 데이터 사용을 허용하지 않은 사용자: 30일 보관 기간 적용
* 개인정보 설정은 [claude.ai/settings/data-privacy-controls](https://claude.ai/settings/data-privacy-controls)에서 언제든지 변경할 수 있습니다.

**상업용 사용자 (Team, Enterprise 및 API)**:

* 표준: 30일 보관 기간
* [데이터 보관 기간 0 (Zero data retention)](/docs/en/zero-data-retention): Claude for Enterprise의 Claude Code 사용 자격이 있는 계정에 제공됩니다. ZDR은 표준 Enterprise 플랜에 포함되어 있지 않으며, 자격 확인 후 계정 담당 팀에 의해 조직별로 활성화됩니다.
* 로컬 캐싱: Claude Code 클라이언트는 세션 재개를 지원하기 위해 기본적으로 30일 동안 `~/.claude/projects/` 아래에 세션 트랜스크립트를 일반 텍스트로 로컬 저장합니다. `cleanupPeriodDays`로 이 기간을 조정할 수 있습니다. 저장되는 항목과 삭제 방법은 [애플리케이션 데이터](/docs/en/claude-directory#application-data)를 참조하세요.

웹상의 개별 Claude Code 세션은 언제든지 삭제할 수 있습니다. 세션을 삭제하면 해당 세션의 이벤트 데이터가 영구적으로 제거됩니다. 세션 삭제 방법에 대한 자세한 내용은 [세션 삭제](/docs/en/claude-code-on-the-web#delete-sessions)를 참조하세요.

데이터 보관 관행에 대한 자세한 내용은 [개인정보 보호 센터](https://privacy.anthropic.com/)에서 확인하세요.

자세한 내용은 [상업용 서비스 약관](https://www.anthropic.com/legal/commercial-terms) (Team, Enterprise 및 API 사용자용) 또는 [소비자 약관](https://www.anthropic.com/legal/consumer-terms) (Free, Pro 및 Max 사용자용) 및 [개인정보 처리방침](https://www.anthropic.com/legal/privacy)을 검토하세요.

## 데이터 액세스

모든 1차 파티 사용자는 [로컬 Claude Code](#local-claude-code-data-flow-and-dependencies) 및 [원격 Claude Code](#cloud-execution-data-flow-and-dependencies)에 대해 기록되는 데이터에 대해 자세히 알아볼 수 있습니다. [Remote Control](/docs/en/remote-control) 세션은 모든 실행이 사용자의 머신에서 이루어지므로 로컬 데이터 흐름을 따릅니다. 연결되어 있는 동안 세션 트랜스크립트는 [연결 및 보안](/docs/en/remote-control#connection-and-security)에 설명된 대로 기기 간 대화를 동기화하기 위해 Anthropic 서버에도 저장됩니다. 원격 Claude Code의 경우, Claude는 Claude Code 세션을 시작한 리포지토리에만 액세스합니다. Claude는 연결되어 있지만 세션을 시작하지 않은 리포지토리에는 액세스하지 않습니다.

## 로컬 Claude Code: 데이터 흐름 및 종속성

아래 다이어그램은 설치 및 일반 실행 중에 Claude Code가 외부 서비스에 연결되는 방식을 보여줍니다. 실선은 필수 연결을 나타내고, 점선은 선택 사항이거나 사용자가 시작한 데이터 흐름을 나타냅니다.

<img src="https://mintcdn.com/claude-code/YR4DRZyI3CdsXkiT/images/claude-code-data-flow.svg?fit=max&auto=format&n=YR4DRZyI3CdsXkiT&q=85&s=2846ea92cfc2297b8620c31c82b482ad" alt="Claude Code의 외부 연결을 보여주는 다이어그램: 설치/업데이트는 배포 서버에 연결되고, 사용자 요청은 Anthropic의 Console auth 및 public-api에 연결되며, 선택적 텔레메트리 흐름은 Anthropic 및 제3자 서비스로 메트릭 및 오류 보고서를 전달합니다. /feedback으로 전송된 피드백은 Google Cloud Storage로 전송되며 선택적으로 GitHub 이슈를 생성합니다." width="720" height="520" data-path="images/claude-code-data-flow.svg" />

Claude Code는 로컬에서 실행됩니다. LLM과 상호작용하기 위해 Claude Code는 네트워크를 통해 데이터를 보냅니다. 이 데이터에는 TLS 1.2+를 통해 전송 중에 암호화된 모든 사용자 프롬프트 및 모델 출력이 포함됩니다. Claude Code는 대부분의 인기 있는 VPN 및 LLM 프록시와 호환됩니다.

저장 시 암호화(Encryption at rest)는 모델 제공업체에 따라 다릅니다:

| 제공업체 | 저장 시 암호화 (Encryption at rest) |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Anthropic API | 인프라 수준 디스크 암호화 (AES-256). 서버 측 영구 저장을 방지하려면 [Zero Data Retention](/docs/en/zero-data-retention)을 활성화하세요. |
| Amazon Bedrock | AWS 관리형 키를 사용한 AES-256. AWS KMS를 통해 고객 관리형 키 이용 가능. |
| Google Cloud's Agent Platform | Google 관리형 암호화 키. CMEK 이용 가능. |
| Microsoft Foundry | 배포의 [호스팅 옵션](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry#hosting-options)에 따라 다릅니다. Azure에서 호스팅되는 배포의 경우 프롬프트와 완성 결과가 Azure 내에 유지됩니다. 사용량 메트릭과 Anthropic의 안전 시스템에서 플래그가 지정된 콘텐츠만 Anthropic으로 전송됩니다. Anthropic에서 호스팅되는 배포의 경우 요청이 AES-256 디스크 암호화가 적용된 Anthropic 인프라로 라우팅됩니다. |

Claude Code는 Anthropic의 API를 기반으로 구축되었습니다. API 로깅 절차를 포함하여 API 보안 제어에 대한 자세한 내용은 [Anthropic Trust Center](https://trust.anthropic.com)의 준수 아티팩트를 참조하세요.

### 클라우드 실행: 데이터 흐름 및 종속성

[웹에서의 Claude Code](/docs/en/claude-code-on-the-web)를 사용할 때 세션은 로컬 대신 Anthropic이 관리하는 가상 머신에서 실행됩니다. 클라우드 환경에서:

* **코드 및 데이터 저장소:** 리포지토리가 격리된 VM에 복제됩니다. 코드 및 세션 데이터는 계정 유형의 보관 및 사용 정책이 적용됩니다 (위의 데이터 보관 섹션 참조).
* **자격 증명:** GitHub 인증은 보안 프록시를 통해 처리됩니다. GitHub 자격 증명은 샌드박스 내부로 절대 들어가지 않습니다.
* **네트워크 트래픽:** 모든 아웃바운드 트래픽은 감사 로깅 및 남용 방지를 위해 보안 프록시를 거칩니다.
* **세션 데이터:** 프롬프트, 코드 변경 사항 및 출력은 로컬 Claude Code 사용과 동일한 데이터 정책을 따릅니다.

클라우드 실행에 대한 보안 자세한 내용은 [보안](/docs/en/security#cloud-execution-security)을 참조하세요.

## 텔레메트리 서비스

Claude Code는 사용량 메트릭 및 오류 보고서라는 두 가지 유형의 운영 텔레메트리를 보냅니다. 아래의 환경 변수를 사용하여 각 항목을 개별적으로 끌 수 있으며, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`을 설정하여 필수적이지 않은 모든 트래픽을 한 번에 비활성화할 수도 있습니다.

**메트릭 (Metrics)**: 지연 시간, 안정성 및 사용 패턴이 TLS를 통해 Anthropic 및 제3자 로깅 인프라로 전송됩니다. 메트릭에는 코드, 프롬프트 또는 파일 경로가 포함되지 않습니다. 거부하려면 `DISABLE_TELEMETRY=1`을 설정하세요.

**오류 보고서 (Error reports)**: Claude Code 자체 내부의 오류 메시지 및 스택 트레이스가 TLS를 통해 제3자 오류 추적 서비스로 전송됩니다. Claude Code는 어떠한 데이터도 머신 밖으로 나가기 전에 알려진 패턴의 보안 비밀, 파일 경로, 이메일 주소 및 기타 개인정보를 가립니다(redact). 거부하려면 `DISABLE_ERROR_REPORTING=1`을 설정하세요.

오류 보고 기능은 다음 조건이 모두 충족될 때만 켜집니다:

* Claude Pro 또는 Max 구독으로 로그인한 경우
* Claude Code v2.1.198 이상을 실행 중인 경우
* Claude API에 직접 연결 중인 경우
* 조직에 zero data retention(데이터 보관 기간 0) 또는 HIPAA 계약이 없는 경우

`/feedback` 명령을 실행하면 코드를 포함한 대화 기록 사본이 Anthropic으로 전송됩니다. `/bug` 및 `/share` 명령도 동일한 경로를 통해 제출됩니다. 제출하기 전에 포함할 기록의 양을 선택할 수 있습니다: 기본값인 현재 세션만 포함하거나, 지난 24시간 또는 7일 동안 동일한 프로젝트에서 진행된 다른 세션도 포함할 수 있습니다. 데이터는 전송 중 TLS를 통해 암호화되고 Google Cloud Storage에 저장되며, 기본적으로 저장된 데이터는 암호화됩니다. 선택적으로 공개 리포지토리에 GitHub 이슈가 생성됩니다. 거부하려면 `DISABLE_FEEDBACK_COMMAND` 환경 변수를 `1`로 설정하세요.

Amazon Bedrock 또는 Google Cloud's Agent Platform과 같은 제3자 제공업체를 사용하거나 Anthropic 자격 증명이 구성되어 있지 않은 경우, `/feedback`은 Anthropic으로 보고서를 전송하는 대신 `~/.claude/feedback-bundles/` 아래의 로컬 아카이브에 기록합니다. 아카이브가 기록되기 전에 알려진 API 키 및 토큰 패턴이 가려집니다. 해당 파일을 Anthropic 계정 담당자에게 보내거나 지원 요청에 첨부할 때까지는 귀하의 머신에서 아무것도 나가지 않습니다.

## API 제공업체별 기본 동작

기본적으로 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 또는 AWS의 Claude Platform을 사용할 때는 오류 보고, 텔레메트리 및 버그 보고 기능이 비활성화됩니다. 세션 품질 설문조사 및 WebFetch 도메인 안전 검사는 예외이며 제공업체에 관계없이 실행됩니다. 로그인된 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션에서는 사용 분석, 오류 보고 및 Anthropic으로의 설문 평점이 게이트웨이 자격 증명 자체에 의해 비활성화되며, 이를 다시 활성화하는 설정은 없습니다. `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`을 설정하여 설문 조사를 포함한 모든 필수적이지 않은 트래픽을 한 번에 거부할 수 있습니다. 이 변수는 WebFetch 검사나 공식 플러그인 마켓플레이스 자동 설치에는 영향을 미치지 않습니다. 각 기능에는 자체 거부 옵션이 있습니다: WebFetch의 경우 [설정](/docs/en/settings)의 `skipWebFetchPreflight`, 마켓플레이스의 경우 `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL`. 다음은 전체 기본 동작입니다:

| 서비스 | Claude API | Google Cloud's Agent Platform API | Amazon Bedrock API | Microsoft Foundry API | Claude Platform on AWS |
| ------------------------------------ | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **메트릭** | 기본 켜짐.<br />비활성화하려면 `DISABLE_TELEMETRY=1`. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_VERTEX`가 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_BEDROCK`이 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_FOUNDRY`가 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_ANTHROPIC_AWS`가 1이어야 함. |
| **오류 보고서** | v2.1.198+에서 Pro 및 Max 로그인 시 켜짐, 그렇지 않으면 꺼짐.<br />비활성화하려면 `DISABLE_ERROR_REPORTING=1`. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_VERTEX`가 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_BEDROCK`이 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_FOUNDRY`가 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_ANTHROPIC_AWS`가 1이어야 함. |
| **Claude API (`/feedback` 보고서)** | 기본 켜짐.<br />비활성화하려면 `DISABLE_FEEDBACK_COMMAND=1`. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_VERTEX`가 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_BEDROCK`이 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_FOUNDRY`가 1이어야 함. | 기본 꺼짐.<br />`CLAUDE_CODE_USE_ANTHROPIC_AWS`가 1이어야 함. |
| **세션 품질 설문조사** | 기본 켜짐.<br />비활성화하려면 `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`. | 기본 켜짐.<br />비활성화하려면 `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`. | 기본 켜짐.<br />`CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`. | 기본 켜짐.<br />`CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`. | 기본 켜짐.<br />`CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`. |
| **WebFetch 도메인 안전 검사** | 기본 켜짐.<br />비활성화하려면 [설정](/docs/en/settings)에 `skipWebFetchPreflight: true`. | 기본 켜짐.<br />비활성화하려면 [설정](/docs/en/settings)에 `skipWebFetchPreflight: true`. | 기본 켜짐.<br />비활성화하려면 [설정](/docs/en/settings)에 `skipWebFetchPreflight: true`. | 기본 켜짐.<br />비활성화하려면 [설정](/docs/en/settings)에 `skipWebFetchPreflight: true`. | 기본 켜짐.<br />비활성화하려면 [설정](/docs/en/settings)에 `skipWebFetchPreflight: true`. |

모든 환경 변수는 `settings.json`에 포함될 수 있습니다 ([설정 참조](/docs/en/settings) 확인).

v2.1.126부터 호스트 플랫폼이 `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST`를 설정하면 Google Cloud's Agent Platform, Amazon Bedrock 및 Microsoft Foundry에 대해 메트릭이 기본적으로 켜지고 표준 `DISABLE_TELEMETRY` 거부를 따릅니다. 해당 제공업체에서 오류 보고 및 `/feedback` 보고서는 기본적으로 꺼진 상태로 유지됩니다.

### WebFetch 도메인 안전 검사

URL을 가져오기 전에 WebFetch 도구는 요청된 호스트 이름을 Anthropic이 유지 관리하는 안전 차단 목록과 비교하기 위해 `api.anthropic.com`으로 전송합니다. 전체 URL, 경로 또는 페이지 내용이 아닌 호스트 이름만 전송됩니다. 결과는 호스트 이름별로 5분 동안 캐시됩니다.

이 검사는 사용하는 모델 제공업체에 관계없이 실행되며 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`의 영향을 받지 않습니다. 네트워크가 `api.anthropic.com`을 차단하는 경우 도메인을 허용 목록에 추가하거나 [설정](/docs/en/settings)에서 `skipWebFetchPreflight: true`를 설정할 때까지 WebFetch 요청이 실패합니다. 검사를 비활성화하면 WebFetch가 차단 목록을 확인하지 않고 URL을 가져오려고 시도하므로, Claude가 접근할 수 있는 도메인을 제한해야 하는 경우 [`WebFetch` 권한 규칙](/docs/en/permissions#webfetch)과 함께 사용하세요.
