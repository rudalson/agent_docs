---
slug: enterprise
section: 엔터프라이즈
title: 엔터프라이즈
path:
  - 엔터프라이즈
---

# Antigravity 및 Gemini 엔터프라이즈 에이전트 플랫폼 시작하기

이 가이드는 Antigravity와 Gemini 엔터프라이즈 에이전트 플랫폼(Gemini Enterprise Agent Platform)의 연동을 활성화하기 위해 Google Cloud 환경을 설정하는 관리자를 위한 안내서입니다. 이 연동을 통해 엔터프라이즈 개발자는 Google Cloud 서비스 약관에 따라 자체 Google Cloud 프로젝트에 호스팅된 모델과 함께 Antigravity를 사용할 수 있으며, 프라이빗 네트워킹 및 데이터 상주 요구 사항을 충족하고 사용량 기반 결제를 활용할 수 있습니다.

## 기본 설정

### 사전 요구 사항

시작하기 전에 다음 사항을 준비하세요:

* Google Cloud 계정  
* Google Cloud 콘솔에 대한 액세스 권한

### 1단계: Google Cloud 프로젝트 선택 또는 생성

Google Cloud 콘솔의 프로젝트 선택기 페이지에서 Google Cloud 프로젝트를 선택하거나 생성합니다.

### 프로젝트 선택 또는 생성에 필요한 역할

* **프로젝트 선택**: 프로젝트를 선택하는 데는 특정 IAM 역할이 필요하지 않으며, 역할이 부여된 모든 프로젝트를 선택할 수 있습니다.
  
  <Announcement>
  icon: info
  iconColor: var(--theme-primary)
  color: var(--theme-surface-surface-container)
  text: **참고**: 다른 Google Cloud 프로젝트 또는 위치로 전환하려면 먼저 Antigravity CLI 또는 허브(Hub)에서 로그아웃한 다음, 다시 로그인하여 새 프로젝트/위치를 선택해야 합니다. 로그인한 상태에서 직접 프로젝트나 위치를 변경하는 것은 현재 지원되지 않습니다.
  </Announcement>
  
* **프로젝트 생성**: 프로젝트를 생성하려면 `resourcemanager.projects.create` 권한이 포함된 **프로젝트 생성자(Project Creator)** 역할(`roles/resourcemanager.projectCreator`)이 필요합니다. [역할을 부여하는 방법 알아보기](https://cloud.google.com/iam/docs/granting-changing-revoking-access).

<Announcement>
icon: info
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **참고**: 이 절차에서 생성한 리소스를 유지할 계획이 없다면 기존 프로젝트를 선택하는 대신 새 프로젝트를 생성하세요. 단계를 완료한 후 프로젝트를 삭제하여 관련된 모든 리소스를 제거할 수 있습니다.
</Announcement>

[프로젝트 선택기로 이동](https://console.cloud.google.com/projectselector2)

### 2단계: 결제 확인

Google Cloud 프로젝트에 대해 결제가 활성화되어 있는지 확인합니다. 결제 상태는 [Google Cloud 결제 콘솔(Google Cloud Billing Console)](https://console.cloud.google.com/billing)에서 확인할 수 있습니다. 자세한 지침은 [프로젝트의 결제 활성화 여부 확인](https://cloud.google.com/billing/docs/how-to/verify-billing-enabled)을 참조하세요.

### 3단계: 에이전트 플랫폼 API 활성화

Gemini 엔터프라이즈 에이전트 플랫폼과 함께 Antigravity를 사용하려면 에이전트 플랫폼 API(`aiplatform.googleapis.com`)를 활성화해야 합니다.

### API 활성화에 필요한 역할

API를 활성화하려면 `serviceusage.services.enable` 권한이 포함된 **서비스 사용 관리자(Service Usage Admin)** IAM 역할(`roles/serviceusage.serviceUsageAdmin`)이 필요합니다. [역할을 부여하는 방법 알아보기](https://cloud.google.com/iam/docs/granting-changing-revoking-access).

### API 활성화하기

[API 라이브러리에서 에이전트 플랫폼 API 활성화하기](https://console.cloud.google.com/apis/library/aiplatform.googleapis.com)

### 사용자 권한

Gemini 엔터프라이즈 에이전트 플랫폼을 사용하는 데 필요한 권한을 얻으려면 관리자에게 프로젝트에 대한 **에이전트 플랫폼 사용자(Agent Platform User)** IAM 역할(`roles/aiplatform.user`)을 부여해 달라고 요청하세요. 역할 부여에 대한 자세한 내용은 [프로젝트, 폴더 및 조직에 대한 액세스 관리](https://cloud.google.com/iam/docs/granting-changing-revoking-access)를 참조하세요.

또한 [사용자 정의 역할(custom roles)](https://cloud.google.com/iam/docs/creating-custom-roles) 또는 기타 [사전 정의된 역할(predefined roles)](https://cloud.google.com/iam/docs/roles-overview#predefined)을 통해 필요한 권한을 얻을 수도 있습니다.

## 고급 구성

### 요청 및 응답 로깅

Gemini 엔터프라이즈 에이전트 플랫폼에 대한 요청 및 응답 로깅을 활성화하고 구성하는 방법에 대한 자세한 지침은 공식 문서를 참조하세요:

[요청 및 응답 로깅 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/capabilities/request-response-logging)

### VPC 서비스 제어 (VPC Service Controls - VPC-SC)

조직에 서비스 경계가 있는 경우 다음 리소스를 경계에 추가해야 합니다:

* 에이전트 플랫폼 API

VPC 서비스 제어 구성에 대한 자세한 지침은 공식 문서를 참조하세요:

[VPC 서비스 제어 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/machine-learning/general/vpc-service-controls)

## 보완 리소스

### 사용 옵션 (Consumption Options)

Gemini 엔터프라이즈 에이전트 플랫폼은 요구 사항에 맞는 다양한 사용 옵션을 제공합니다.

사용 옵션에 대한 자세한 내용은 공식 문서를 참조하세요:

[사용 옵션 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/deploy/consumption-options)

### 배포 및 엔드포인트 위치

현재 Antigravity CLI 및 2.0은 global, eu(멀티 리전), us(멀티 리전)의 3가지 엔드포인트를 제공합니다.

<Announcement>
icon: info
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **참고**: 현재 `eu` 및 `us` 위치에서는 이미지 생성을 사용할 수 없습니다.
</Announcement>

사용 가능한 위치 및 배포 엔드포인트의 전체 목록은 공식 문서를 참조하세요:

[배포 엔드포인트 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/resources/locations#global)
