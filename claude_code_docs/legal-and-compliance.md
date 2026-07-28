> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 법률 및 규정 준수 (Legal and compliance)

> Claude Code에 대한 법적 계약, 규정 준수 인증 및 보안 정보입니다.

## 법적 계약

### 라이선스

Claude Code 사용에는 다음 계약이 적용됩니다:

* [상업용 약관 (Commercial Terms)](https://www.anthropic.com/legal/commercial-terms) - Team, Enterprise 및 Claude API 사용자용
* [소비자 서비스 약관 (Consumer Terms of Service)](https://www.anthropic.com/legal/consumer-terms) - Free, Pro 및 Max 사용자용

### 상업적 계약

Claude API를 직접 사용하든(1P), Amazon Bedrock 또는 Google Cloud's Agent Platform을 통해 접근하든(3P), 상호 달리 합의하지 않는 한 기존 상업적 계약이 Claude Code 사용에 적용됩니다.

## 규정 준수

### 헬스케어 규정 준수 (BAA)

고객이 당사와 BAA(Business Associate Agreement)를 체결하고 Claude Code를 사용하려는 경우, 고객이 BAA를 체결하고 [제외 데이터 보존 (Zero Data Retention; ZDR)](/docs/en/zero-data-retention)을 활성화했다면 BAA가 자동으로 Claude Code까지 확장 적용됩니다. BAA는 Claude Code를 통해 흐르는 해당 고객의 API 트래픽에 적용됩니다. ZDR은 조직 단위로 활성화되므로 BAA 적용을 받으려면 각 조직마다 ZDR을 개별적으로 활성화해야 합니다.

## 사용 정책

### 허용 가능한 사용

Claude Code 사용에는 [Anthropic 사용 정책 (Anthropic Usage Policy)](https://www.anthropic.com/legal/aup)이 적용됩니다. Pro 및 Max 플랜에 대해 공지된 사용 제한은 Claude Code 및 Agent SDK의 일반적인 개별 사용을 가정한 것입니다.

### 인증 및 자격 증명 사용

Claude Code는 OAuth 토큰 또는 API 키를 사용하여 Anthropic 서버와 인증합니다. 이러한 인증 방법은 서로 다른 목적을 가집니다:

* **OAuth 인증**은 Claude Free, Pro, Max, Team 및 Enterprise 구독 플랜 구매자 전용이며, Claude Code 및 기타 네이티브 Anthropic 애플리케이션의 일반적인 사용을 지원하도록 설계되었습니다. 로그인 단계는 [Claude 계정 로그인](https://support.claude.com/en/articles/13189465-logging-in-to-your-claude-account)을 참조하세요; Claude Code가 OAuth 인증을 수행하는 방법은 [인증](/docs/en/authentication)을 참조하세요.
* [Agent SDK](/docs/en/agent-sdk/overview)를 사용하는 제품을 포함하여 Claude의 기능과 상호작용하는 제품이나 서비스를 구축하는 **개발자**는 [Claude Console](https://platform.claude.com/) 또는 지원되는 클라우드 제공업체를 통해 API 키 인증을 사용해야 합니다. Anthropic은 서드파티 개발자가 사용자를 대신하여 Claude.ai 로그인을 제공하거나 Free, Pro 또는 Max 플랜 자격 증명을 통해 요청을 라우팅하는 것을 허용하지 않습니다.

Anthropic은 이러한 제한 사항을 시행하기 위해 조치를 취할 권리를 보유하며 사전 통지 없이 조치할 수 있습니다.

사용 사례에 허용되는 인증 방법에 관한 질문은 [영업팀에 문의](https://www.anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=legal_compliance_contact_sales)하세요.

## 보안 및 신뢰성

### 신뢰 및 안전 (Trust and safety)

[Anthropic Trust Center](https://trust.anthropic.com) 및 [Transparency Hub](https://www.anthropic.com/transparency)에서 자세한 정보를 찾을 수 있습니다.

### 보안 취약점 보고

Anthropic은 HackerOne을 통해 보안 프로그램을 관리합니다. 취약점을 보고하려면 [이 양식을 사용하세요](https://hackerone.com/4f1f16ba-10d3-4d09-9ecc-c721aad90f24/embedded_submissions/new).

***

© Anthropic PBC. All rights reserved. Use is subject to applicable Anthropic Terms of Service.
