---
slug: build-with-google
section: Antigravity 2.0
title: Google 기술로 빌드하기
path:
  - Antigravity 2.0
  - Google 기술로 빌드하기
---

# Google 기술로 빌드하기

Antigravity 2.0을 사용하면 널리 쓰이는 Google 기술 스택을 활용하여 원활하게 빌드할 수 있습니다. Google 내 여러 팀과 협력하여 스킬(Skills), MCP 서버, 에디터 확장 프로그램을 비롯한 특정 생태계에 맞춰 미리 구성된 엄선된 프리미티브(primitives) 번들을 만들었습니다.

개별 도구를 찾아다닐 필요 없이, 이러한 번들을 활성화하여 에이전트에게 Google 플랫폼에 대한 심도 있는 지식을 즉각 제공할 수 있습니다.

### 활성화 방법

"Google 기술로 빌드하기(Build with Google)" 통합은 다음 두 시점에 활성화할 수 있습니다:

* **온보딩 과정 중**: 사용할 예정인 스택의 체크박스를 선택합니다.  
* **설정에서**: 언제든지 통합을 추가하거나 제거하려면 `Settings > Customizations > Build with Google Plugins`(설정 > 사용자 정의 > Google 빌드 플러그인)로 이동하세요.

### 제공되는 번들

### 현대적 웹 가이드 (Modern Web Guidance)

코딩 에이전트가 최신 웹 개발 모범 사례를 유지할 수 있도록 합니다.

* **포함 내역**: 현대적 웹을 위해 항상 최신 상태로 유지되고 전문가의 검증을 거친 스킬 패키지  
  * **핵심 기능**: 주입된 가이드를 활용하여 에이전트가 접근성 높고 성능이 뛰어나며 안전한 웹 경험을 빌드할 수 있습니다.  
  * **더 알아보기**: [현대적 웹 가이드 읽기](http://goo.gle/modern-web-guidance)

### Firebase 번들

AI 코딩 에이전트를 코드를 작성하고 Firebase 보안 규칙을 구성하며 실시간 리소스를 관리할 수 있는 전문 Firebase 전문가로 변환합니다.

* **포함 내역**:  
  * Firestore, Authentication, App Hosting 등을 포함한 핵심 Firebase 서비스를 위한 에이전트 스킬 패키지  
* **핵심 기능**:  
  * **실제 작업 수행**: 단순히 코드를 작성하는 것 이상을 수행합니다. 에이전트는 서비스를 초기화하고, 인증(Authentication) 사용자를 관리하며, 새로운 Firebase 보안 규칙을 배포하고, Cloud Firestore 데이터로 직접 작업할 수 있습니다.  
  * **최신 상태 유지**: 공식적인 버전 정보가 포함된 프롬프트를 사용하여 에이전트의 설정 작업을 안내합니다.  
  * **정확성 향상**: 프로젝트의 환경 및 스키마에 액세스하여 보다 관련성 높고 정확한 지원을 제공합니다.  
* **더 알아보기**: [Firebase 에이전트 스킬 가이드 살펴보기](https://firebase.google.com/docs/ai-assistance/agent-skills) 

### Google Antigravity SDK

Antigravity Python SDK를 사용한 AI 에이전트 빌드

* **포함 내역**:  
  * Antigravity Python SDK(google-antigravity)에 대한 아키텍처 참조, 시작 예제 및 구성 가이드를 포함하는 에이전트 스킬  
* **핵심 기능**:  
  * **에이전트 빌드**: 에이전트가 새로운 Antigravity 에이전트의 뼈대를 구성하고, 모델을 설정하며, 사용자 정의 Python 도구를 등록하고, SDK 모범 사례를 사용하여 MCP 서버를 연결할 수 있습니다.  
  * **안전 유지**: 기본 거부(deny-by-default) 템플릿 및 인수 수준 서술어(predicates)를 포함하여 선언적 보안 정책을 구현하기 위해 공식 가이드를 사용합니다.  
  * **더 알아보기**: 수명 주기(lifecycle) 훅, 다중 에이전트 위임, 구조화된 출력, 멀티모달 입력 및 토큰 사용량 관찰 가능성(observability)에 대한 참조 자료에 액세스합니다.  
* **더 알아보기**: [Antigravity SDK 리포지토리 방문하기](https://github.com/google-antigravity/antigravity-sdk-python)

### Android CLI

Android 개발에 필요한 핵심 도구 및 지식

* **포함 내역**: UI를 엣지투엣지(edge-to-edge) 및 반응형으로 구성하고, 테스트 프레임워크를 만들고, 성능 최적화를 위해 R8 구성을 분석하는 등의 핵심 개발 작업을 위한 Android CLI 및 Android 스킬.  
* **핵심 기능**:  
  * 원하는 에이전트가 고품질 Android 앱을 빌드할 수 있도록 활성화   
  * 새 프로젝트 생성   
  * Android 가상 기기(AVD)에 앱 배포  
  * 자연어 명령을 사용하여 앱 테스트   
  * 최신 버전의 라이브러리 및 플러그인으로 마이그레이션  
* **더 알아보기**: [Android 에이전트 개발자 도구 살펴보기](http://developer.android.com/tools/agents) 

### 과학 (Science)

과학 연구를 위한 엄선된 에이전트 스킬 모음.

* **포함 내역**: 고품질로 엄선된 과학 분야의 스킬 번들   
* **핵심 기능**: 과학 분야 워크플로우에 대한 에이전트의 기능 향상.  
* **더 알아보기**: [DeepMind 과학 스킬 리포지토리 검토하기](https://github.com/google-deepmind/science-skills) 

### Chrome DevTools

Chrome DevTools 및 Puppeteer를 사용하여 Chrome에서 신뢰할 수 있는 자동화, 심층 디버깅 및 성능 분석 수행

* **포함 내역**: Chrome DevTools 및 Puppeteer를 위한 에이전트 스킬 패키지  
* **핵심 기능**: 웹 접근성(a11y) 이슈 디버깅, 핵심 웹 바이탈(Core Web Vitals - LCP/INP) 감사, 시각적 자동화 테스트 실행 및 메모리 누수 진단.  
* **더 알아보기**: [Chrome DevTools 스킬 리포지토리 살펴보기](https://github.com/google/chrome-devtools-skills)
