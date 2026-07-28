---
slug: browser
section: Antigravity IDE
title: 개요
path:
  - Antigravity IDE
  - 브라우저
  - 개요
---

# 브라우저 개요

Google Antigravity는 로컬 Chrome 브라우저를 열고, 읽고, 작동시킬 수 있으므로 개발 웹사이트 테스트, 문서 소스 읽기 및 다양한 브라우저 작업 자동화가 가능합니다.

---

## 핵심 메커니즘

전용 [브라우저 하위 에이전트](/docs/subagents)를 사용하여 Antigravity는 필요에 따라 브라우저 탭을 조작하고, 스크린샷을 캡처하고, 작업 비디오를 대화형 아티팩트로 저장합니다.

브라우저 도구를 완전히 비활성화하려면 사용자 설정의 "브라우저" 섹션에서 **브라우저 도구** 설정을 전환할 수 있습니다.

---

## 심층 분석

Antigravity 브라우저 통합의 주요 보안 및 개인정보 보호 기능을 살펴보세요:

<IconCardGroup>
- link: /docs/allowlist-denylist
  icon: security
  title: 허용 목록 및 차단 목록
  description: URL 액세스를 제어하는 데 사용되는 이중 보안 모델(차단 목록 및 허용 목록)에 대해 알아봅니다.
- link: /docs/separate-chrome-profile
  icon: account_box
  title: 격리된 프로필
  description: 개인 데이터를 보호하기 위해 에이전트가 완전히 별도의 Chrome 프로필 내에서 실행되는 방식을 이해합니다.
</IconCardGroup>
