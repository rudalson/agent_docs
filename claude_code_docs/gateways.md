> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 게이트웨이를 통해 Claude Code 실행하기

> 중앙 집중식 자격 증명, 사용량 추적 및 비용 제어를 위해 자체 호스팅 게이트웨이를 통해 Claude Code를 라우팅하세요. 아키텍처, Anthropic의 Claude apps gateway 및 기타 게이트웨이 제품 사용을 다룹니다.

게이트웨이는 조직이 Claude Code와 모델 공급자 사이에서 운영하는 프록시입니다. Claude Code는 API 트래픽을 공급자에게 직접 전송하는 대신 게이트웨이로 전송하며, 게이트웨이는 조직이 보유한 자격 증명을 사용하여 이를 전달합니다. 개발자는 공급자 자격 증명을 직접 보유하는 대신 게이트웨이에 인증하므로 인증, 사용량 추적, 예산 및 감사 로깅이 제어 가능한 한 위치에서 일어납니다.

Claude Code에는 `claude` 바이너리에 자체 호스팅 게이트웨이인 [Claude apps gateway](/docs/en/claude-apps-gateway)가 포함되어 있으므로 별도의 게이트웨이 제품을 도입할 필요 없이 바로 운용할 수 있습니다. 조직이 이미 [LLM 게이트웨이](/docs/en/llm-gateway)를 운영 중인 경우 Claude Code는 해당 게이트웨이와도 함께 작동합니다.

이 페이지에서는 다음 내용을 다룹니다:

* [Claude Code와 공급자 사이에서 게이트웨이가 위치하는 방식](#how-a-gateway-works)
* [Claude apps gateway와 기존에 운영 중인 게이트웨이 간 선택](#choose-a-gateway)
* [게이트웨이가 claude.ai 구독과 상호작용하는 방식](#subscriptions-and-gateways)
* [게이트웨이와 별도로 구성되는 항목](#configure-separately-from-the-gateway)

## 게이트웨이 작동 방식

각 개발자의 Claude Code는 게이트웨이 주소를 가리키며 게이트웨이가 발급한 자격 증명으로 인증합니다.

게이트웨이는 개발자를 인증하고, 구성된 접근 및 예산 규칙을 적용하며, 조직의 자격 증명을 사용하여 요청을 공급자에게 전달합니다. 공급자는 Anthropic API이거나 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry와 같은 [클라우드 공급자](/docs/en/third-party-integrations)일 수 있으며, 게이트웨이의 구성에 따라 결정됩니다. Claude apps gateway나 단일 Anthropic 형식 엔드포인트를 노출하는 기타 게이트웨이를 사용하면 공급자를 변경하더라도 개발자 머신을 수정할 필요가 없습니다.

두 가지 유형의 자격 증명이 사용됩니다.

* **개발자 자격 증명**: 게이트웨이가 발급하며 각 개발자가 개별적으로 보유합니다. 개발자를 게이트웨이에 인증하고 사용량 추적 시 신원을 식별합니다.
* **공급자 자격 증명**: 게이트웨이가 공급자 계정용으로 단 하나 보유하며, 전달되는 모든 트래픽이 이를 공유합니다.

## 게이트웨이 선택하기

Claude Code는 Anthropic 자체 게이트웨이 또는 조직에서 이미 운영 중인 게이트웨이와 함께 작동합니다.

### Claude apps gateway

Claude apps gateway는 `claude` 바이너리에 포함된 Anthropic의 자체 호스팅 게이트웨이입니다. 업스트림으로 Amazon Bedrock, Claude Platform on AWS, Google Cloud, Microsoft Foundry 또는 Anthropic API로 라우팅합니다. 개발자는 `/login`을 통해 기업 자격 증명 공급자(IdP)로 로그인하고, 게이트웨이는 IdP 그룹별로 모델 접근 및 [관리 대상 설정](/docs/en/permissions#managed-settings)을 적용하며, [OpenTelemetry Protocol (OTLP)](/docs/en/monitoring-usage) 사용량 메트릭을 고유한 모니터링 스택으로 내보냅니다.

Claude Code 릴리스와 함께 빌드되고 테스트되므로 Claude Code가 전송하는 헤더와 요청 필드를 유실 없이 전달합니다. 개별적으로 유지 관리되는 게이트웨이는 각 릴리스마다 헤더와 필드가 변경됨에 따라 [전달 규칙을 업데이트](/docs/en/llm-gateway-protocol#forward-as-open-lists)해야 합니다. Claude apps gateway는 CLI와 함께 릴리스되므로 지속적으로 관리할 목록이 없습니다. 게이트웨이 세션에서 다르게 작동하는 일부 기능은 [이용 가능 여부 및 제한 사항](/docs/en/claude-apps-gateway#availability-and-limitations)을 참조하세요.

게이트웨이 로그인은 브라우저 SSO 단계이며 서비스 토큰 플로우가 없으므로 로그인을 승인할 개발자가 없는 CI 파이프라인은 게이트웨이를 통해 인증할 수 없습니다. 해당 파이프라인은 공급자에게 직접 구성하세요. 개발자가 로그인한 머신에서의 Agent SDK 세션 및 `claude -p` 실행은 해당 머신의 게이트웨이 세션을 사용하며 해당 정책의 규제를 받습니다. [CI 파이프라인 및 원격 머신](/docs/en/claude-apps-gateway#ci-pipelines-and-remote-machines)을 참조하세요.

배포하려면 [Claude apps gateway](/docs/en/claude-apps-gateway)를 참조하세요.

### 기타 게이트웨이

조직에서 이미 LLM 게이트웨이나 API 게이트웨이를 운영 중인 경우 대신 사용할 수 있습니다. Anthropic은 다른 게이트웨이 제품을 보증, 유지 관리 또는 감사하지 않으며, 게이트웨이를 통한 비-Claude 모델로의 Claude Code 라우팅을 지원하지 않습니다. 관리자 배포 체크리스트, 게이트웨이가 구현해야 하는 사항 및 Claude Code를 이를 향하도록 설정하는 방법은 [기타 LLM 게이트웨이](/docs/en/llm-gateway)를 참조하세요.

## 구독과 게이트웨이

개발자가 게이트웨이 자격 증명으로 게이트웨이를 통해 연결하면 사용량은 API 요금으로 조직의 공급자 계정에 청구되며 개발자의 claude.ai 구독은 사용되거나 청구되지 않습니다. 직접 운영하는 게이트웨이에 대해 [`ANTHROPIC_AUTH_TOKEN`](/docs/en/env-vars)을 설정하거나 `/login`으로 Claude apps gateway에 로그인하면 해당 세션에 대해 구독 로그인이 꺼집니다. 해당 자격 증명 하에 전달되는 모든 요청은 게이트웨이의 공급자 자격 증명 뒤에 있는 계정에 청구됩니다.

게이트웨이 자격 증명 없이 `ANTHROPIC_BASE_URL`만 설정하는 것은 예외입니다. 요청이 여전히 게이트웨이를 통해 라우팅되지만 저장된 claude.ai 로그인이 활성 자격 증명으로 유지되므로 구독의 사용 한도 및 청구가 적용됩니다. [기타 LLM 게이트웨이](/docs/en/llm-gateway#subscriptions-and-gateways)에서 이 구성과 작동을 위해 게이트웨이가 전달해야 하는 사항을 다룹니다.

## 게이트웨이와 별도로 구성하는 항목

게이트웨이는 모델 API 요청을 라우팅합니다. 게이트웨이가 처리할 것으로 예상할 수 있는 몇 가지 사항은 다른 곳에서 구성됩니다.

* **응답할 모델 결정**: `/model` 명령이나 [모델 환경 변수](/docs/en/model-config#setting-your-model)로 모델을 선택합니다. 게이트웨이는 개발자가 선택한 모델이 아닌 요청이 이동할 위치를 결정합니다. Claude apps gateway는 그룹별 `availableModels` 허용 목록으로 선택 범위를 제한할 수 있지만 개발자는 여전히 그 안에서 선택합니다.
* **기타 네트워크 트래픽**: Claude Code 자체는 게이트웨이 경로와 별개로 버전을 확인하고 Anthropic에서 직접 다운로드를 전송합니다. 선택적 클라이언트 텔레메트리 스트림의 활성화 여부는 공급자에 따라 다릅니다. [텔레메트리 기본값 표](/docs/en/data-usage#telemetry-services)에서 각 케이스를 다룹니다. 로그인된 Claude apps gateway 세션에서 게이트웨이 자격 증명은 Anthropic을 향하는 분석을 비활성화하며, [텔레메트리 전달](/docs/en/claude-apps-gateway-config#telemetry)이 구성된 경우 OTLP 내보내기를 게이트웨이에 고정합니다. 네트워크에 [필수 도메인](/docs/en/network-config)으로의 출력이 여전히 필요하거나, 선택적 스트림을 끄려면 [`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`](/docs/en/env-vars)을 설정하세요.
* **기업 HTTP 프록시**: `HTTPS_PROXY`는 게이트웨이를 포함하여 통신하는 모든 서버와 Claude Code 사이에 위치합니다. 네트워크에 프록시가 필요한 경우 게이트웨이 외에 [프록시를 구성](/docs/en/network-config)하세요. 직접 호스팅하는 Claude apps gateway의 경우 [로그인 시 프록시 호스트도 사설 네트워크에 있는지 확인](/docs/en/claude-apps-gateway#prerequisites)합니다. 그렇지 않은 경우 CLI가 게이트웨이에 직접 연결되도록 `NO_PROXY`에 게이트웨이 호스트를 추가하세요.

## 다음 단계

다음 페이지는 누가 게이트웨이를 운영하는지에 따라 다릅니다. Anthropic의 게이트웨이는 `claude` 바이너리에서 실행되며 자체 설정 가이드가 있습니다; 조직에서 이미 운영 중인 게이트웨이는 구현할 프로토콜과 관리자 배포 체크리스트를 포함합니다.

* [Claude apps gateway](/docs/en/claude-apps-gateway): SSO 로그인 및 OTLP 텔레메트리를 포함한 Anthropic의 자체 호스팅 게이트웨이 배포
* [기타 LLM 게이트웨이](/docs/en/llm-gateway): 조직에서 이미 운영 중인 게이트웨이가 구현해야 하는 사항 및 Claude Code 지정 방법
* [조직을 위한 Claude Code 설정](/docs/en/admin-setup): 게이트웨이가 일부분을 차지하는 광범위한 배포 결정 사항
