---
slug: allowlist-denylist
section: Antigravity IDE
title: 허용 목록 / 차단 목록
path:
  - Antigravity IDE
  - 브라우저
  - 허용 목록 / 차단 목록
---

# 허용 목록 / 차단 목록

브라우저는 어떤 URL에 액세스할 수 있는지 제어하기 위해 이중 레이어 보안 시스템을 사용합니다:
- **차단 목록(Denylist)** - 위험하거나 악성인 URL 거부
- **허용 목록(Allowlist)** - 신뢰할 수 있는 URL을 명시적으로 허용

## 작동 방식

### 차단 목록
차단 목록은 Google Superroots의 BadUrlsChecker 서비스를 사용하여 유지 관리 및 적용됩니다(문서 참조).
브라우저가 URL로 이동을 시도하면, RPC를 통해 호스트 이름이 서버 측 차단 목록에 대해 검사됩니다.

**참고:** 서버를 사용할 수 없는 경우 기본적으로 액세스가 거부됩니다.

### 허용 목록

허용 목록은 특정 URL을 명시적으로 신뢰하도록 편집할 수 있는 로컬 텍스트 파일입니다.

![허용 목록](assets/image/docs/browser-allowlist.png)

허용 목록은 localhost로만 초기화되며 언제든지 편집할 수 있습니다.

브라우저가 허용 목록에 없는 URL로 이동하려고 하면 "항상 허용" 버튼이 있는 프롬프트가 표시되며, 이를 클릭하면 해당 URL이 허용 목록에 추가되어 브라우저가 웹 페이지를 열고 상호 작용할 수 있습니다. 예시 상황은 아래와 같습니다:

![항상 허용](assets/image/docs/always-allow-url.png)

허용 목록에서 직접 URL을 수동으로 추가/제거할 수도 있습니다. 하지만 항상 차단 목록이 우선합니다. 차단 목록에 있는 URL은 허용 목록에 추가할 수 없습니다.
