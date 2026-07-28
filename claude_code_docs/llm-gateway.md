> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 기타 LLM 게이트웨이

> 조직에서 이미 운영 중인 LLM 게이트웨이를 통해 Claude Code를 라우팅하세요. 게이트웨이에 Claude Code 연결, 조직을 위한 배포, Claude Code가 게이트웨이로 전송하는 항목을 다룹니다.

이 섹션은 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 대신 조직에서 이미 운영 중인 게이트웨이 제품을 사용하는 방법을 다룹니다. 게이트웨이가 무엇인지, Claude Code와 제공업체 사이에 어떻게 위치하는지, Claude 앱 게이트웨이와 다른 제품 중에서 선택하는 방법은 [게이트웨이 개요](/docs/en/gateways)를 참조하세요.

<Note>
  * 기존 게이트웨이에 연결하는 개발자의 경우: [게이트웨이에 Claude Code 연결](/docs/en/llm-gateway-connect)
  * 조직에 게이트웨이를 배포하는 관리자의 경우: [게이트웨이 배포 및 배포 관리](/docs/en/llm-gateway-rollout)
  * 게이트웨이 제품을 구성하는 사용자의 경우: [게이트웨이 프로토콜 레퍼런스](/docs/en/llm-gateway-protocol)
</Note>

[지원되는 API 형식](/docs/en/llm-gateway-protocol#api-formats)을 노출하는 모든 게이트웨이가 작동합니다. Anthropic은 서드파티 게이트웨이 제품을 보증, 유지 관리 또는 감사하지 않으며, 임의의 게이트웨이를 통해 Claude Code를 비-Claude 모델로 라우팅하는 것을 지원하지 않습니다. 자체 설명서에 따라 게이트웨이를 배포한 다음 [아래의 배포 단계](#roll-out-a-gateway)를 거쳐 Claude Code 쪽 구성을 완료하세요.

## 게이트웨이가 제공하는 기능

게이트웨이는 조직에 다음을 한곳에서 관리할 수 있는 환경을 제공합니다:

* **자격 증명 (Credentials)**: 제공업체 키는 서버 측에 보관되며 개발자는 게이트웨이 자격 증명을 가집니다.
* **사용량 추적 (Usage tracking)**: 어떤 제공업체가 요청을 처리하든 개발자나 팀별로 사용량을 추적합니다.
* **비용 제어 (Cost controls)**: 한곳에서 예산 및 속도 제한을 강제합니다.
* **감사 로깅 (Audit logging)**: 규정 준수를 위해 모든 모델 요청을 기록합니다.
* **제공업체 전환 (Provider switching)**: 개발자 머신을 건드리지 않고 게이트웨이 구성에서 제공업체를 변경합니다.

제공업체 전환을 제외한 모든 기능은 업스트림이 Anthropic API이든 [클라우드 제공업체](/docs/en/third-party-integrations)이든 상관없이 적용됩니다. 개발자 머신을 다시 구성하지 않고 제공업체를 전환하는 것은 업스트림과 관계없이 단일 [Anthropic 형식 엔드포인트](/docs/en/llm-gateway-protocol#api-formats)를 노출하는 게이트웨이에 따라 달라집니다; 특정 제공업체 고유의 형식을 노출하는 게이트웨이는 클라이언트 구성을 해당 제공업체에 종속시킵니다.

트레이드오프는 게이트웨이가 조직에서 직접 운영하는 인프라가 된다는 점입니다. Claude Code는 릴리스마다 새로운 기능을 추가하며, 이를 전달하지 않는 게이트웨이는 해당 기능을 작동 불능으로 만들므로 Claude Code가 진화함에 따라 게이트웨이 제품도 지속적으로 업데이트해야 합니다. [게이트웨이 프로토콜 레퍼런스](/docs/en/llm-gateway-protocol)에서 전달해야 할 항목을 다룹니다.

## 게이트웨이 배포하기

조직에 LLM 게이트웨이를 배포할 준비가 되었을 때, 선택한 게이트웨이 제품에 관계없이 순서는 동일합니다:

1. 게이트웨이를 배포하고 전달되는 요청을 인증할 수 있도록 제공업체 자격 증명을 부여합니다.
2. 개발자별로 게이트웨이 자격 증명을 발급하여 사용량이 개발자에게 귀속되도록 하고, 온보딩 해제 시 해당 자격 증명 하나만 취소하면 되도록 합니다.
3. [관리형 설정 파일](/docs/en/settings#settings-files) 및 암호화 도구를 통해 구성을 배포하여 모든 머신이 베이스 URL과 자격 증명을 수신하도록 합니다. 둘 다 배포되면 개발자는 아무것도 구성할 필요가 없습니다. 설정 배포 체계가 구축되어 있지 않다면 개발자는 [연결 페이지](/docs/en/llm-gateway-connect)에 따라 변수를 직접 설정합니다.
4. 각 개발자가 [Claude Code에서 구성을 확인](/docs/en/llm-gateway-connect#check-for-an-existing-configuration)하도록 하여 게이트웨이에 의존하기전에 배포 문제가 드러나도록 합니다.

[조직을 위한 LLM 게이트웨이 배포](/docs/en/llm-gateway-rollout)에서 각 단계를 설명하고 각 단계에서 배포할 구성 파일을 보여줍니다. 게이트웨이는 조직 설정의 한 부분일 뿐입니다; 정책 강제, 사용량 가시성 및 데이터 처리 결정에 대해서는 [조직을 위한 Claude Code 설정](/docs/en/admin-setup)을 참조하세요.

## 구독 및 게이트웨이

[게이트웨이 자격 증명 변수](/docs/en/llm-gateway-connect#set-the-credential-variable) 또는 `apiKeyHelper`가 활성화되어 있는 동안에는 개발자의 claude.ai 구독이 사용되지 않습니다: 해당 세션에서 자격 증명이 구독 로그인을 대체하며 구독의 사용 한도가 적용되지 않습니다. 이 트래픽은 게이트웨이가 요청을 라우팅하는 위치에 따라 조직의 Anthropic Console 계정, 또는 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 계정 등 게이트웨이가 전달하는 자격 증명의 소유자에게 토큰당 과금됩니다.

[`ANTHROPIC_BASE_URL`](/docs/en/llm-gateway-connect#set-the-base-url-and-credential)은 Claude Code가 게이트웨이를 가리키도록 설정하는 변수입니다. 게이트웨이 자격 증명 없이 이 변수만 설정하는 것은 구독을 대체하지 않습니다. 요청은 여전히 게이트웨이를 통해 라우팅되지만 저장된 claude.ai 로그인이 활성 자격 증명으로 유지되므로 해당 사용 한도 및 과금이 적용됩니다. 이 트래픽을 Anthropic으로 전달하는 게이트웨이는 `anthropic-beta`에서 OAuth 기능을 전달해야 합니다; [요청 헤더 레퍼런스](/docs/en/llm-gateway-protocol#request-headers)를 참조하세요.

## 관련 페이지

* [게이트웨이 개요](/docs/en/gateways): 게이트웨이 작동 방식 및 Claude 앱 게이트웨이와 다른 제품 간의 선택 가이드
* [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway): SSO 로그인 및 OTLP 텔레메트리를 지원하는 Anthropic 자체 호스팅 게이트웨이
* [LLM 게이트웨이에 Claude Code 연결](/docs/en/llm-gateway-connect): 환경별 구성 및 문제 해결 표와 함께 사용자의 컴퓨터에서 베이스 URL과 자격 증명 설정
* [조직을 위한 LLM 게이트웨이 배포](/docs/en/llm-gateway-rollout): 게이트웨이 배포, 개발자 자격 증명 발급 및 관리형 설정 배포를 위한 관리자 체크리스트
* [게이트웨이 프로토콜 레퍼런스](/docs/en/llm-gateway-protocol): 엔드포인트, 전달할 헤더, 기능 통과를 다루는 운영자용 Claude Code 게이트웨이 전송 데이터 레퍼런스
