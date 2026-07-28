> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Amazon Bedrock, AWS 기반 Claude Platform, Google Cloud 및 Microsoft Foundry를 위한 Claude apps gateway

> 자체 호스팅 게이트웨이 뒤에서 SSO 로그인, 그룹별 모델 접근 및 OTLP 텔레메트리를 통해 Amazon Bedrock, AWS 기반 Claude Platform, Google Cloud 또는 Microsoft Foundry에서 Claude Code를 실행하세요.

<Note>
  Claude apps gateway는 [데이터 거주성](/docs/en/claude-apps-gateway-deploy#compliance-posture) 요건 준수 등을 위해 자체 클라우드 제공업체를 통해 추론을 라우팅해야 하거나 이를 선호하는 조직을 위해 설계되었습니다. 이러한 요건이 없고 SCIM 프로비저닝이나 웹/모바일에서의 Claude Code 등 기타 기능 접근이 필요한 경우에는 Claude Enterprise가 더 적합할 수 있습니다. 모든 배포 방식의 전체 비교는 [기능 사용 가능 여부](/docs/en/feature-availability) 페이지를 참조하세요.
</Note>

Claude apps gateway는 개발자의 Claude Code 클라이언트와 모델 제공업체 사이에 위치하는 자체 호스팅 서비스입니다. 개발자는 API 키나 클라우드 자격 증명을 보유하는 대신 기업 정체성 제공업체(IdP)로 로그인합니다. 게이트웨이는 상류(upstream) 자격 증명을 보관하고, IdP 그룹별로 모델 접근 및 [관리형 설정](/docs/en/permissions#managed-settings)을 적용하며, 자체 모니터링 스택으로 사용량 텔레메트리를 중계합니다.

이 서비스는 `claude` 바이너리에 포함되어 있으므로 랩톱에서 Claude Code를 실행하는 동일한 실행 파일로 `claude gateway --config gateway.yaml`을 실행하여 게이트웨이 서버를 구동합니다.

이 페이지에서는 다음 내용을 다룹니다:

* [Claude apps gateway를 사용하는 이유](#claude-apps-gateway를-사용하는-이유): 직접 구축 대비 추가되는 기능 및 다른 방법이 더 적합한 경우
* 게이트웨이 설정부터 로그인까지 과정을 다루는 [사전 요구 사항](#사전-요구-사항) 및 [퀵스타트](#퀵스타트)
* 관리형 설정을 통해 게이트웨이 URL을 설정하는 방식을 포함한 [개발자 연결](#개발자-연결)
* 게이트웨이를 통해 작동하는 Claude Code 기능 및 서버 지원 범위를 다루는 [사용 가능 여부 및 제약 사항](#사용-가능-여부-및-제약-사항)

관련 페이지에서 더 자세한 정보를 제공합니다. [설정 참조](/docs/en/claude-apps-gateway-config)는 퀵스타트에서 작성하는 YAML 파일의 모든 옵션을 다루며, [배포 가이드](/docs/en/claude-apps-gateway-deploy)는 IdP별 설정, Kubernetes 및 Cloud Run 배포, 운영 방법을 다룹니다.

## Claude apps gateway를 사용하는 이유

[게이트웨이 개요](/docs/en/gateways)에서 게이트웨이가 수행하는 작업과 운영하는 이유를 다룹니다. Claude apps gateway는 Anthropic 자체 게이트웨이로, `claude` 바이너리에 내장되어 있으며 각 Claude Code 릴리스와 함께 테스트되므로 운영자가 별도의 허용 목록을 유지 관리하지 않아도 Claude Code가 보내는 헤더와 요청 필드를 전달합니다. 배포하면 다음을 제공합니다:

* **자격 증명**: 상류 API 키 또는 클라우드 자격 증명은 조직의 인프라에만 존재합니다. 개발자는 기업 SSO로 인증하고 단기 전달자 토큰(bearer token)을 받으므로 자격 취소(offboarding)는 IdP에서 이루어집니다. 사용자를 비활성화하면 게이트웨이 접근 권한은 세션 수명(기본값 1시간) 내에 만료됩니다.
* **접근 제어**: IdP 그룹은 모델 허용 목록 및 [관리형 설정](/docs/en/permissions#managed-settings) 정책에 매핑됩니다. 게이트웨이는 승인되지 않은 모델 요청을 거부하여 서버 측 모델 접근을 적용하고 각 그룹의 관리형 설정 정책을 선택하며, CLI는 이를 [관리형 설정 계층](/docs/en/settings#settings-precedence)에 적용합니다. 팀마다 서로 다른 모델, 도구, 권한을 받게 되며 개발자는 정책으로 고정된 내용을 재정의할 수 없습니다.
* **설정 전달**: 게이트웨이는 claude.ai 관리자 콘솔의 [서버 관리형 설정](/docs/en/server-managed-settings)을 대신하여 로그인된 클라이언트에 관리형 설정을 직접 전달합니다.
* **텔레메트리**: Datadog, Splunk 또는 ClickHouse와 같이 설정된 각 대상은 토큰 수, 모델, 사용자 정체성 및 지연 시간이 포함된 [OpenTelemetry Protocol (OTLP) 메트릭](/docs/en/monitoring-usage)을 기본적으로 수신하며, 로그 및 트레이스는 대상별 선택 사항으로 제공됩니다.
* **상류 라우팅**: 클라이언트는 Anthropic Messages API를 사용해 게이트웨이와 통신하며, 게이트웨이는 Amazon Bedrock, [AWS 기반 Claude Platform](/docs/en/claude-platform-on-aws), Google Cloud Agent Platform, Microsoft Foundry 또는 Anthropic API 등 구성된 각 상류로 요청을 변환하고 장애 조치(failover)를 수행합니다. 개발자가 눈치채거나 재설정할 필요 없이 지역(region), 제공업체 또는 장애 조치 순서를 변경할 수 있습니다.

<Frame>
  <img src="https://mintcdn.com/claude-code/st9_ZQOFsZa3cKFl/images/claude-gateway-architecture.svg?fit=max&auto=format&n=st9_ZQOFsZa3cKFl&q=85&s=560770d8f49bbd6f1ca7090ed1f13c03" alt="Diagram showing Claude Code clients connecting over HTTPS with bearer tokens to a self-hosted Claude apps gateway inside your infrastructure, which signs users in against your IdP, stores auth state in PostgreSQL, relays telemetry to your OTLP collector, and forwards inference to Amazon Bedrock, Claude Platform on AWS, Google Cloud, Microsoft Foundry, or the Anthropic API" width="760" height="320" data-path="images/claude-gateway-architecture.svg" />
</Frame>

<Note>
  게이트웨이의 데이터 플레인은 Anthropic API가 상류로 설정되지 않은 한 Anthropic 인프라에 아무것도 전송하지 않습니다. 텔레메트리, 감사 로그, 관리형 설정 및 개발자의 IdP 정체성이 전송되는 위치는 사용자가 직접 제어하며 게이트웨이는 이 중 어느 것도 Anthropic으로 전송하지 않습니다. CLI 프로세스가 전송할 수 있는 나머지 트래픽과 이를 차단하는 방법은 [준수 상태(Compliance posture)](/docs/en/claude-apps-gateway-deploy#compliance-posture)를 참조하세요.
</Note>

게이트웨이를 통해 작동하는 Claude Code 기능 및 서버 자체가 지원하는 항목은 아래의 [사용 가능 여부 및 제약 사항](#사용-가능-여부-및-제약-사항)을 참조하세요. 비용, 우회, 여러 게이트웨이 실행, 서버리스 플랫폼 등의 결정사항은 [배포 가이드](/docs/en/claude-apps-gateway-deploy#deployment)를 참조하세요.

### 기타 게이트웨이 구현

요구 사항을 충족하는 LLM 게이트웨이 또는 API 게이트웨이를 이미 구동 중인 경우 계속 사용하세요. [기타 LLM 게이트웨이](/docs/en/llm-gateway)에서 이를 대상으로 Claude Code를 구성하는 방법을 다룹니다.

[게이트웨이 프로토콜 참조](/docs/en/llm-gateway-protocol)는 Claude Code가 모든 게이트웨이에 기대하는 계약(호출하는 엔드포인트, 전달할 헤더 및 본문 필드, 이들이 제거될 때 작동하지 않는 항목)을 제공합니다. 실행 중인 Claude apps gateway는 `GET /protocol`에서 이 계약의 상위 세트를 제공하여 SSO 로그인, 관리형 설정 전달, 텔레메트리를 위한 Claude apps gateway 전용 엔드포인트를 추가합니다. [퀵스타트](#퀵스타트)를 통해 생성된 배포 게이트웨이 등에서 `curl https://claude-gateway.internal.example.com/protocol` 명령으로 이를 가져올 수 있습니다.

프로토콜의 하위 호환성을 중단하는 변경 사항은 사전에 공지되지만 영구적인 이전 버전 호환성이 보장되지는 않습니다.

## 퀵스타트

이 퀵스타트는 최소한의 경로를 제공합니다: IdP에 OAuth 클라이언트 등록, `gateway.yaml` 작성, Docker Compose로 Postgres와 함께 게이트웨이 실행, 엔드투엔드 로그인 검증. 이 예제는 Amazon Bedrock 상류를 사용합니다. [설정 참조](/docs/en/claude-apps-gateway-config#upstreams)에 표시된 대로 `upstreams` 블록을 교체하면 AWS 기반 Claude Platform, Google Cloud Agent Platform, Microsoft Foundry, Anthropic API도 동일하게 지원됩니다. 완료되면 개발자가 `/login`할 수 있는 게이트웨이가 준비됩니다.

<Note>
  **사설 네트워크에 배포하세요.** Claude Code는 주소가 사설인 게이트웨이에만 연결됩니다. 신뢰할 수 있는 게이트웨이가 개발자 머신에서 명령을 실행하는 설정을 푸시할 수 있으므로 이는 보안 보호조치입니다. 내부 로드 밸런서나 VPN 뒤에 게이트웨이를 두고 사설 IP로만 해소되는 호스트 이름을 부여하세요.

  Anthropic이 직접 운영하는 공개 게이트웨이 엔드포인트는 예외입니다: `/login`은 `https://`를 통해 이를 수용합니다. 이들은 Anthropic 자체가 운영하는 소수의 고정된 게이트웨이 세트입니다. 선택하거나 구성할 수 있는 배포 옵션이 아닙니다. 이 목록은 Claude Code에 컴파일되어 들어가므로 어떠한 설정으로도 해당 목록에 호스트 이름을 추가할 수 없으며 자체 호스팅하는 게이트웨이는 예외를 받을 수 없습니다. {/* min-version: 2.1.206 */}v2.1.206 이전에는 `/login`이 다른 공개 주소와 마찬가지로 이러한 엔드포인트를 거부했습니다.
</Note>

### 사전 요구 사항

시작하기 전에 다음 항목을 준비하세요:

| 필요 사항                               | 상세 내용                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code v2.1.195 이상               | `claude gateway` 하위 명령 및 게이트웨이 로그인 흐름은 v2.1.195에 포함되어 있습니다. 이전 공개 빌드에는 포함되어 있지 않습니다. 게이트웨이 서버를 실행하는 머신과 각 개발자 머신 모두 v2.1.195 이상이어야 합니다. 최신 릴리스를 받으려면 `claude update`를 실행하세요. {/* min-version: 2.1.198 */} [AWS 기반 Claude Platform 상류](/docs/en/claude-apps-gateway-config#claude-platform-on-aws)를 사용하려면 게이트웨이 서버에 Claude Code v2.1.198 이상이 필요합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| OpenID Connect (OIDC) 정체성 제공업체   | Okta, Microsoft Entra ID, Google Workspace, Keycloak, Dex 또는 PingFederate와 같은 기타 OIDC 준수 IdP. 게이트웨이는 정식 OIDC 디스커버리 및 인가 코드 흐름(authorization-code flow)을 실행합니다. SAML 및 LDAP은 지원되지 않습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| PostgreSQL 14 이상                      | 브라우저 콜백이 쓰고 CLI가 폴링으로 읽는 디바이스 로그인 흐름과 속도 제한 카운터를 백업합니다. 소형 티어를 포함하여 모든 관리형 Postgres가 작동합니다. 지출 한도가 설정되지 않은 경우 게이트웨이는 수명이 짧은 몇 KB의 인증 상태를 저장합니다. [지출 한도](/docs/en/claude-apps-gateway-spend-limits)가 구성되면 백업해야 하는 영구 지출, 감사, 정체성 테이블도 유지 관리합니다. `?sslmode=require`를 통한 TLS 사용을 권장합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 모델 상류 (Upstream)                    | Amazon Bedrock 자격 증명, AWS 기반 Claude Platform 자격 증명, Google Cloud 자격 증명, Microsoft Foundry 리소스 또는 Anthropic API 키. 장애 조치를 지원하여 여러 상류를 지정할 수 있습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| HTTPS                                   | 게이트웨이는 개발자 랩톱과 로그인에 사용되는 모든 브라우저에서 `https://`를 통해 접근 가능해야 합니다. 게이트웨이는 동일한 수신기에서 디바이스 검증 페이지를 제공합니다. `listen.tls`를 통해 TLS 인증서를 제공하거나, TLS 종료 인그레스 뒤에서 실행하고 `listen.public_url`을 설정하세요. 일반 `http://` 오리진은 로컬 개발을 위한 루프백에서만 허용됩니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 사설 네트워크 주소                      | `/login` 실행 시 Claude Code는 게이트웨이의 호스트 이름 또는 IP 주소가 사설 주소(RFC 1918, 링크 로컬, CGNAT `100.64.0.0/10`, IPv6 ULA `fc00::/7` 또는 로컬 개발용 루프백)로만 해소되도록 요구합니다. 직접 호스팅하는 게이트웨이의 경우 공개 주소는 거부됩니다. 배포 가이드의 [위협 모델 요약](/docs/en/claude-apps-gateway-deploy#threat-model-summary)을 참조하세요. 검사는 해소된 각 IP에서 실행되므로 해소된 주소 중 하나라도 공개 주소이면 `/login`이 URL을 거부합니다. 개발자 머신이 기업 프록시를 통해 HTTPS를 라우팅하는 경우 로그인 시 프록시 호스트도 사설 주소로 해소되어야 합니다. 그렇지 않은 경우 CLI가 직접 연결되도록 `NO_PROXY`에 게이트웨이 호스트를 추가하세요. {/* min-version: 2.1.206 */}Anthropic이 운영하는 공개 게이트웨이 엔드포인트는 사설 주소 및 프록시 검사에서 예외입니다. `/login`은 호스트 이름의 정확한 일치를 통해 `https://`에서 이를 승인하므로 사설 네트워크 요구 사항은 직접 호스팅하는 게이트웨이에만 적용됩니다. v2.1.206 이전에는 `/login`이 Anthropic 운영 엔드포인트를 다른 공개 주소와 마찬가지로 거부했습니다. |
| Linux 런타임                            | 게이트웨이 서버는 기본 Linux 바이너리에서만 실행됩니다. macOS는 로컬 개발용으로 작동합니다. Windows는 서버 플랫폼으로 지원되지 않습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

게이트웨이 서버에는 네이티브 `claude` 바이너리가 필요합니다. [Claude Code 설치](/docs/en/setup)에 설명된 대로 고정(pinned) 릴리스를 다운로드하세요. 서버는 Node 환경에서 Claude Code가 실행될 때는 사용할 수 없는 런타임 기능을 사용합니다. 부팅 시 `requires the native binary` 메시지가 표시되면 단독 설치 방식 중 하나로 전환하세요.

### 단계

<Steps>
  <Step title="IdP에 OAuth 클라이언트 등록">
    리다이렉트 URI가 일치해야 하므로 먼저 게이트웨이의 호스트 이름을 결정하세요. 새 OIDC 웹 애플리케이션을 생성하고 리다이렉트 URI를 `https://claude-gateway.<your-domain>/oauth/callback`으로 설정합니다. 여기서 호스트는 3단계에서 [`listen.public_url`](/docs/en/claude-apps-gateway-config#listen)로 설정하는 값과 동일해야 합니다. `client_id` 및 `client_secret`을 기록해 둡니다. IdP별 지침은 [정체성 제공업체 설정](/docs/en/claude-apps-gateway-deploy#identity-provider-setup)을 참조하세요.
  </Step>

  <Step title="PostgreSQL 데이터베이스 프로비저닝">
    가장 작은 관리형 티어를 포함하여 PostgreSQL 14 이상이면 작동합니다. 게이트웨이는 부팅 시 스키마 마이그레이션을 실행하므로 데이터베이스 사용자에게 `CREATE TABLE` 권한이 필요합니다. 보안 정책상 애플리케이션 역할의 DDL이 금지된 경우 대신 스키마를 미리 생성하세요. [`store`](/docs/en/claude-apps-gateway-config#store)를 참조하세요.
  </Step>

  <Step title="gateway.yaml 작성">
    파일 자체가 버전 제어로 관리될 수 있도록 비밀 값은 `${ENV_VAR}` 확장을 통해 읽어옵니다. `/login`이 공개 주소를 거부하므로 네트워크의 사설 IP로 해소되는 `public_url` 호스트 이름을 사용하세요. 최소 설정에는 5개 섹션이 포함되며 나머지 모든 필드에는 기본값이 있습니다:

    ```yaml gateway.yaml theme={null}
    listen:
      host: 0.0.0.0
      port: 8080
      # TLS 종료 프록시 뒤에서 필수. IdP redirect_uri 및
      # 디스커버리 문서에 사용됩니다.
      public_url: https://claude-gateway.internal.example.com

    oidc:
      issuer: https://login.example.com        # /.well-known/openid-configuration을 제공해야 함
      client_id: 0oa1example2
      client_secret: ${OIDC_CLIENT_SECRET}
      allowed_email_domains: [example.com]        # 조직 외부의 id_token 거부
      userinfo_fallback: true                  # id_token에서 이메일/그룹이 누락된 IdP용 (기타 상황에는 영향 없음)

    session:
      jwt_secret: ${GATEWAY_JWT_SECRET}        # openssl rand -base64 32
      ttl_hours: 1                             # IdP 권한 박탈 시 취소 지연 시간의 상한선 역할도 함

    store:
      postgres_url: ${GATEWAY_POSTGRES_URL}    # 관리형 Postgres의 경우 ?sslmode=require 추가

    upstreams:
      - provider: bedrock
        region: us-east-1
        auth: {}                               # 비어 있음: AWS 기본 자격 증명 체인
                                               # (IRSA, EC2/ECS 태스크 역할, 환경 변수, ~/.aws)

    # 모델은 상류별로 자동 변환됩니다. 기본 제공 카탈로그는
    # claude-opus-4-8을 us.anthropic.claude-opus-4-8 등으로 Bedrock 지원
    # 모든 Claude 모델에 매핑합니다. 특정 모델만 노출하려면 false로 설정하고 `models:` 목록을 추가하세요.
    auto_include_builtin_models: true
    ```

    이 설정은 기본 Amazon Bedrock 모델 카탈로그를 사용하여 작동하는 로그인 루프에 충분합니다. 실행되면 [`managed.policies`](/docs/en/claude-apps-gateway-config#managed)를 통해 그룹별 RBAC 및 관리형 설정을 추가하고, [`telemetry`](/docs/en/claude-apps-gateway-config#telemetry)를 통해 텔레메트리 팬아웃을 추가하고, [`models`](/docs/en/claude-apps-gateway-config#models)를 통해 다중 상류 장애 조치, 프로비저닝된 처리량 ARN 또는 비-US 지역을 추가하세요.

    <Note>
      Amazon Bedrock 상류에는 `inference-profile/us.anthropic.*` ARN과 기본 `foundation-model/anthropic.*` ARN 모두에 대한 `bedrock:InvokeModel` 및 `bedrock:InvokeModelWithResponseStream` 권한과, 사용하려는 Claude 모델에 대해 Amazon Bedrock 콘솔에서 모델 접근이 활성화된 AWS 보안 주체가 필요합니다. 정적 키 대신 EKS의 IRSA, ECS 태스크 역할 또는 EC2 인스턴스 프로필로 자격 증명을 제공하세요. [`upstreams` 참조](/docs/en/claude-apps-gateway-config#upstreams)에서 전체 IAM 정보, 멀티 클라우드 자격 증명 매트릭스 및 다른 제공업체를 위한 `auth` 블록을 확인할 수 있습니다.
    </Note>
  </Step>

  <Step title="실행">
    [이미지 요구 사항](/docs/en/claude-apps-gateway-deploy#container-image)을 충족하는 `claude` 바이너리를 중심으로 컨테이너 이미지를 빌드한 다음 Postgres와 함께 실행합니다. Compose 파일은 이미지를 `registry.example.com/claude-gateway:2.1.198`로 참조합니다. 자체 레지스트리 및 이미지 태그로 대체하세요:

    ```yaml docker-compose.yaml theme={null}
    services:
      gateway:
        image: registry.example.com/claude-gateway:2.1.198
        ports: ["8080:8080"]
        volumes: ["./gateway.yaml:/etc/claude/gateway.yaml:ro"]
        environment:
          OIDC_CLIENT_SECRET: ${OIDC_CLIENT_SECRET}
          GATEWAY_JWT_SECRET: ${GATEWAY_JWT_SECRET}
          GATEWAY_POSTGRES_URL: postgres://gw:pw@postgres/gateway
          # AWS 자격 증명: 프로덕션 환경에서는 이를 생략하고 인스턴스 역할 사용.
          # 로컬 Compose 테스트의 경우 본인 자격 증명 전달:
          AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
          AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
          AWS_SESSION_TOKEN: ${AWS_SESSION_TOKEN}
        depends_on:
          postgres:
            condition: service_healthy
      postgres:
        image: postgres:16-alpine
        environment: { POSTGRES_USER: gw, POSTGRES_PASSWORD: pw, POSTGRES_DB: gateway }
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U gw"]
          interval: 5s
        volumes: ["pgdata:/var/lib/postgresql/data"]
    volumes: { pgdata: }
    ```

    게이트웨이는 설정을 읽고, IdP에 대해 OIDC 디스커버리를 실행하고, Postgres 스키마 마이그레이션을 적용하고, 상류 클라이언트를 빌드하며 수신을 시작하는 단일 Linux 바이너리입니다. 부팅 시 설정, 5초 타임아웃의 Postgres 연결, OIDC 디스커버리 및 상류 클라이언트 생성 실패 시 바로 종료(fail-closed)됩니다. 이들 중 도달할 수 없거나 잘못 구성된 항목이 있으면 게이트웨이는 저하된 상태로 트래픽을 처리하는 대신 에러와 함께 종료됩니다.

    성공적인 부팅이 추론 경로의 유효성을 전적으로 검증하지는 않는데, 이는 Amazon Bedrock 및 Google Cloud Agent Platform 인스턴스 자격 증명이 부팅 시점이 아닌 첫 번째 요청 시 해소되기 때문입니다.

    부팅 시퀀스는 stderr을 통해 확인하세요. 로그 라인은 `[gateway] <timestamp> <level> <message>` 형식을 사용하고, 감사 이벤트는 `evt` 필드가 포함된 단일 행 JSON이며, 아래에서 생략된 시작 배너가 마이그레이션 줄과 수신 줄 사이에 출력됩니다. 순서대로 다음 내용이 표시되어야 합니다:

    ```text theme={null}
    {"ts":"2026-06-10T17:03:21.114Z","evt":"config.load","path":"/etc/claude/gateway.yaml","sha256":"…"}
    [gateway] 2026-06-10T17:03:21.408Z info migration 1 applied
    [gateway] 2026-06-10T17:03:21.512Z info claude gateway listening on http://0.0.0.0:8080
    ```

    `claude gateway listening on` 줄이 표시되기 전에 부팅이 종료되면 stderr의 마지막 줄에 원인이 지정됩니다:

    * 도달할 수 없는 Postgres
    * DDL 권한이 없는 Postgres 역할
    * 도달할 수 없거나 잘못된 OIDC 디스커버리 문서
    * 문제가 있는 필드 경로가 포함된 설정 스키마 위반

    문제를 수정하고 재시작하세요.

    이미 TLS 종료 인그레스가 있는 경우 Compose를 건너뛰고 `claude gateway --config gateway.yaml` 명령으로 바이너리를 직접 실행하세요. `public_url`을 인그레스 오리진으로 설정하고 `listen`을 루프백 또는 클러스터 내부 주소에 바인딩하세요.
  </Step>

  <Step title="인증 표면 검증">
    개발자에게 제공하기 전에 세 가지 검사로 게이트웨이가 실제 사용자를 인증할 수 있는지 확인합니다.

    예제는 게이트웨이의 공개 URL을 사용합니다. 인그레스가 없는 로컬 Compose 설정의 경우 첫 번째 및 두 번째 검사에서 `http://localhost:8080`으로 대체하세요. 세 번째 검사는 `public_url`을 기반으로 생성된 `verification_uri_complete`를 열므로 로컬 Compose의 경우 `gateway.yaml`에 `public_url: http://localhost:8080`을 설정하고 1단계의 OAuth 클라이언트에 두 번째 리다이렉트 URI로 `http://localhost:8080/oauth/callback`을 추가하세요. 게이트웨이가 `public_url`에서 IdP `redirect_uri`를 생성하기 때문입니다. 그러면 로컬 브라우저에서 검증 링크가 열립니다.

    Windows PowerShell에서는 `curl.exe`를 실행하세요. 단순 `curl`은 `Invoke-WebRequest`의 별칭이며 이러한 플래그를 거부합니다.

    첫째, 디스커버리 문서를 가져와 게이트웨이가 작동 중이고 설정을 가져올 수 있으며 모든 부팅 검사를 통과했는지 확인합니다:

    ```bash theme={null}
    curl -s https://claude-gateway.internal.example.com/.well-known/oauth-authorization-server | jq
    ```

    ```json theme={null}
    {
      "issuer": "https://claude-gateway.internal.example.com",
      "device_authorization_endpoint": "…/oauth/device_authorization",
      "token_endpoint": "…/oauth/token",
      "grant_types_supported": ["urn:ietf:params:oauth:grant-type:device_code", "refresh_token"]
    }
    ```

    응답에는 `response_types_supported` 및 `scopes_supported`와 같은 추가 필드가 포함됩니다.

    둘째, 디바이스 인가를 요청하여 디바이스 로그인 흐름이 작동하고 Postgres에 도달할 수 있으며 쓰기가 가능한지 확인합니다:

    ```bash theme={null}
    curl -s -X POST https://claude-gateway.internal.example.com/oauth/device_authorization | jq
    ```

    ```json theme={null}
    {
      "device_code": "…",
      "user_code": "WDJB-MJHT",
      "verification_uri": "https://claude-gateway.internal.example.com/device",
      "verification_uri_complete": "https://claude-gateway.internal.example.com/device?user_code=WDJB-MJHT",
      "expires_in": 600,
      "interval": 5
    }
    ```

    셋째, 브라우저에서 `verification_uri_complete`를 열고 코드를 확인하여 브라우저 단계를 테스트합니다. IdP의 로그인 페이지로 리다이렉트되어야 하며, 로그인한 후 로그인 확인과 함께 게이트웨이로 다시 돌아와야 합니다.

    첫 번째 실패한 검사를 사용하여 문제를 규명하세요:

    * **첫 번째 검사 실패**: 부팅이 완료되지 않음. stderr 확인
    * **두 번째 검사 실패**: 게이트웨이에서 Postgres에 도달할 수 없거나 역할이 쓸 수 없음. 연결 문자열 및 권한 부여 확인
    * **세 번째 검사가 IdP에 도달하지 않음**: IdP의 리다이렉트 URI가 `https://<gateway>/oauth/callback`과 정확히 일치하는지 확인
    * **세 번째 검사가 IdP에 도달하지만 에러와 함께 돌아옴**: `email domain not allowed` 등 거부 이유가 기록된 게이트웨이 감사 로그 확인
  </Step>

  <Step title="개발자 로그인">
    이 마지막 단계는 서버가 아닌 개발자 머신에서 이루어집니다. 해당 머신의 [관리형 설정 파일](/docs/en/settings#settings-files)에서 `forceLoginMethod`를 `"gateway"`로 설정하고 `forceLoginGatewayUrl`을 게이트웨이의 `public_url`로 설정한 다음, `/login`을 실행하고 **Cloud gateway** 화면에서 Enter를 누른 뒤 브라우저 로그인을 완료합니다. 두 키를 대규모로 배포하는 방법은 아래 [게이트웨이 URL 설정](#게이트웨이-url-설정)을 다룹니다.
  </Step>
</Steps>

## 개발자 연결

개발자는 기업 업무용 계정을 사용해 브라우저 한 번의 로그인으로 자신의 랩톱에서 연결합니다. 모델 요청은 조직의 상류 자격 증명을 사용하여 게이트웨이를 통과하므로 claude.ai 계정, API 키 또는 구독이 필요하지 않습니다. 연결은 MDM을 통해 푸시하는 [클라이언트 측 관리형 설정](/docs/en/claude-apps-gateway-config#client-side-managed-settings)에 의해 구동되므로 개발자 측의 수동 설정이 없습니다. 이 섹션에서는 관리자가 설정하는 내용을 다룹니다.

CLI는 첫 연결 시 게이트웨이의 TLS 리프 인증서를 지문으로 추출(fingerprint)하고 호스트 이름별로 고정(pin)합니다. 개발자가 비교할 수 있도록 게이트웨이 URL과 함께 예상되는 SHA-256 지문을 함께 공지하세요. `openssl x509 -noout -fingerprint -sha256 -in cert.pem` 명령으로 인증서 파일에서 지문을 가져올 수 있습니다. `/login` 프롬프트에는 구분 기호 없는 소문자 16진수로 다이제스트의 처음 16자리가 표시됩니다.

인증서가 갱신되면 모든 개발자에게 신뢰 프롬프트가 다시 표시되므로 갱신을 계획된 이벤트로 취급하고 지문을 다시 공지하세요.

로그인하면 [모델 선택기](/docs/en/model-config)에 개발자의 `availableModels` 허용 목록에 있는 모델이 표시되고, 관리형 설정이 시작 시 적용되며 매시간 새로 고쳐지고, 텔레메트리가 수집기로 라우팅됩니다. 세션은 `ttl_hours` 만료 전에 배경에서 자동으로 새로 고쳐지며, IdP 자격 박탈 후 새로 고침에 실패하면 다시 로그인하라는 프롬프트가 표시됩니다.

### 게이트웨이 URL 설정

세 개의 키가 MDM을 통해 배포하거나 디스크에 직접 설치하는 OS별 [관리형 설정 파일](/docs/en/settings#settings-files)에 들어갑니다. `forceLoginMethod` 및 `forceLoginGatewayUrl`은 URL이 채워진 상태로 **Cloud gateway** 화면에서 `/login`을 직접 열며, `parentSettingsBehavior: "merge"`는 [Claude Desktop 세션에 정책 전달](#claude-desktop-세션에-정책-전달)에 설명된 대로 Claude Desktop이 시작하는 Claude Code 세션에 게이트웨이 정책을 전달할 수 있게 합니다:

```json theme={null}
{
  "forceLoginMethod": "gateway",
  "forceLoginGatewayUrl": "https://claude-gateway.internal.example.com",
  "parentSettingsBehavior": "merge"
}
```

개발자는 Enter를 눌러 연결합니다. [첫 연결 TLS 지문 프롬프트](#개발자-연결)는 여전히 표시됩니다.

개발자가 이를 수동으로 설정할 수는 없습니다. 로그인 선택기에는 게이트웨이 옵션이 없으며 개발자 자신의 설정 파일에서는 `forceLoginGatewayUrl`이 무시됩니다. URL 없이 `forceLoginMethod`만 사용하면 개발자에게 "Contact your IT administrator" 메시지가 표시됩니다. 로그인 키는 게이트웨이의 `managed.policies[].cli` 블록(이미 연결된 클라이언트에만 도달함)이 아닌, 머신에 푸시하는 파일에 포함되어야 합니다.

### Claude Desktop 세션에 정책 전달

Claude Desktop은 임베디드 Claude Code 세션을 실행하고 시작하는 각 세션에 게이트웨이 정책을 전달합니다. Claude Desktop은 게이트웨이 자체에서 해당 정책을 받습니다. 자체 관리형 설정을 통해 게이트웨이를 가리키고, [게이트웨이 URL 설정](#게이트웨이-url-설정)의 `forceLoginMethod` 및 `forceLoginGatewayUrl` 키와는 별개로 자체 흐름으로 로그인합니다.

실행 프로세스에 의해 전달되는 설정은 부모 설정(parent settings)입니다. Claude Code는 가장 높은 우선순위 소스가 `parentSettingsBehavior: "merge"`를 설정하지 않는 한, 관리자가 배포한 관리형 소스가 있는 모든 머신에서 부모 설정을 무시합니다.

#### 옵트인이 필요한 머신

부모 설정이 게이트웨이 정책이 임베디드 세션에 도달하는 유일한 방법이므로 Claude Desktop만 실행하는 머신에 이 설정이 필요합니다. 옵트인이 없으면 해당 세션은 게이트웨이의 제약 조건 없이 실행되며 어떠한 경고도 표시되지 않습니다.

개발자가 `/login`을 통해 로그인하는 머신은 이것이 필요하지 않습니다. 모든 Claude Code 호출은 게이트웨이에서 직접 정책을 가져옵니다. [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)를 구성하는 플릿은 헬퍼의 출력이 다른 모든 관리형 소스를 대체하고 헬퍼가 구성되어 있는 동안 부모 설정이 병합되지 않으므로 이를 사용할 수 없습니다.

#### 옵트인 설정

키를 배포하고 각 머신에서 적용되는 소스에 이를 미러링한 다음 검증합니다.

<Steps>
  <Step title="관리형 설정 파일에 옵트인 배포">
    [위의 스니펫](#게이트웨이-url-설정)에 이미 `parentSettingsBehavior: "merge"`가 포함되어 있으므로 머신에 푸시하는 파일에 포함됩니다.
  </Step>

  <Step title="파일보다 상위 순위의 모든 소스에 동일한 키 설정">
    가장 높은 우선순위의 관리자 소스 값만 인정됩니다. macOS의 관리형 환경설정 plist 또는 Windows의 HKLM 정책은 `managed-settings.json` 파일보다 순위가 높으며 게이트웨이의 원격 관리형 설정은 두 가지 모두보다 순위가 높으므로 게이트웨이에 로그인하는 머신의 경우 게이트웨이 정책의 [`cli` 블록](/docs/en/claude-apps-gateway-config#managed)에도 키를 설정하세요.
  </Step>

  <Step title="적용된 소스 확인">
    Agent SDK의 [`resolveSettings()`](/docs/en/agent-sdk/typescript#resolvesettings)를 호출하세요. 결과에 `sources` 목록이 포함되며, 거기 관리형 정책 항목에는 활성 소스 이름을 나타내는 `policyOrigin` 필드가 포함됩니다. `resolveSettings()`는 구성된 `policyHelper`를 실행하지 않으므로 헬퍼 플릿에서는 해당 응답이 라이브 세션을 반영하지 않습니다.
  </Step>
</Steps>

### 부모 설정 제한

`parentSettingsBehavior: "merge"`를 배포하면 Claude Code를 실행하는 모든 호스트 프로세스(Claude Desktop뿐만 아니라 Agent SDK 애플리케이션 또는 IDE 확장 프로그램)가 부모 설정을 제공할 수 있습니다.

Claude Code는 제한적인 키 허용 목록에 대해 부모 설정을 필터링하지만 일부 허용된 키는 제한하기보다는 접근 권한을 부여할 수 있습니다. `allowManaged*Only` 잠금을 설정하지 않으면 호스트가 제공하는 권한 허용 규칙 및 샌드박스 허용 목록이 계속 적용됩니다. 정책의 거부 및 요청 규칙은 어느 쪽이든 그대로 적용됩니다([허용 규칙보다 먼저 평가됨](/docs/en/permissions#manage-permissions)).

#### 잠금 배포

필터가 지원하는 한 부모 설정을 제한 전용에 가깝게 유지하려면 병합 옵트인과 동일한 소스에 5개의 `allowManaged*Only` 잠금 및 이들이 통제하는 허용 목록을 모두 추가하세요:

```json theme={null}
{
  "forceLoginMethod": "gateway",
  "forceLoginGatewayUrl": "https://claude-gateway.internal.example.com",
  "parentSettingsBehavior": "merge",
  "allowManagedPermissionRulesOnly": true,
  "allowManagedMcpServersOnly": true,
  "allowManagedHooksOnly": true,
  "allowedMcpServers": [{ "serverUrl": "https://mcp.internal.example.com/*" }],
  "sandbox": {
    "network": {
      "allowManagedDomainsOnly": true,
      "allowedDomains": ["github.com", "*.npmjs.org"]
    },
    "filesystem": {
      "allowManagedReadPathsOnly": true,
      "denyRead": ["~/"],
      "allowRead": ["~/projects"]
    }
  }
}
```

HKLM 레지스트리 정책이나 관리형 환경설정 plist와 같은 OS 정책은 이 파일보다 순위가 높으므로 파일 대신 해당 정책을 통해 스니펫 전체를 전달하세요. 게이트웨이의 원격 관리형 설정은 OS 정책 및 파일 소스보다 순위가 높지만 연결된 클라이언트에만 도달합니다. 잠금, 허용 목록 및 병합 옵트인을 정책의 [`cli` 블록](/docs/en/claude-apps-gateway-config#managed)에 미러링하고 이 파일을 배포 상태로 유지하세요. 연결하지 않는 머신(Claude Desktop만 실행하는 머신 포함)은 이 파일에서만 정책을 가져오기 때문입니다.

#### 소스 간 잠금 동작

하나의 잠금을 설정한다고 해서 다른 잠금이 제한되는 것은 아닙니다. 각 키는 [설정 참조](/docs/en/settings#available-settings)에 설명되어 있습니다. 적용된 소스 아래의 관리자 소스에서도 두 샌드박스 잠금은 여전히 적용되며 `allowManagedPermissionRulesOnly`는 부모가 제공한 허용 규칙 및 `additionalDirectories`를 차단합니다. 훅 및 MCP 서버 잠금과 개발자 자체 규칙에 대한 `allowManagedPermissionRulesOnly`의 영향은 최상위 관리자 소스가 필요합니다. [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper) 플릿에서는 헬퍼의 출력에서만 잠금을 읽어옵니다.

각 잠금은 Claude Code가 해당 설정에 대한 개발자 자체 항목을 무시하도록 하므로 잠금 옆에 조직의 허용 목록을 포함하세요. 비어 있는 관리형 도메인 목록으로 네트워크 도메인을 잠그면 샌드박스로 보호된 모든 아웃바운드 트래픽이 차단되며, 관리형 또는 부모 제공 `allowedMcpServers` 없이 MCP 서버를 잠그면 `deniedMcpServers`가 차단하지 않는 모든 서버가 로드됩니다. `allowRead` 항목은 `denyRead` 영역 내부의 경로만 다시 허용하므로 관리형 `denyRead`와 쌍을 이루어야 합니다.

#### 잠금이 적용되지 않는 설정

5개의 잠금을 모두 설정하더라도 부모가 제공하는 다음 4개 설정은 준수됩니다:

* **`forceLoginOrgUUID`**: 관리자 소스에서 조직 UUID를 설정하지 않은 경우 Claude Code는 부모가 제공한 값을 적용합니다. 게이트웨이 로그인 시 이 키를 검사하지 않으므로 퍼스트 파티 Anthropic 로그인을 함께 사용하는 플릿에만 의미가 있습니다. 관리자 소스의 조직 UUID는 부모 값을 차단하지만 Claude Code가 적용하는 값은 가장 높은 우선순위 소스에서 가져오므로 거기에 `forceLoginOrgUUID`를 설정하세요.
* **`allowedMcpServers`**: 관리자 소스에서 설정하지 않은 경우 Claude Code는 부모가 제공한 허용 목록을 준수하며 `allowManagedMcpServersOnly`는 이를 차단하지 않습니다. 잠금이 적용되면 어떠한 목록이든 관리형 값으로 인정되기 때문입니다. 관리자 소스의 목록은 부모의 목록을 차단하므로 잠금 옆의 해당 소스에 `allowedMcpServers`를 설정하세요.
* **`availableModels`**: 적용된 관리형 소스에서 설정하지 않은 경우 Claude Code는 부모가 제공한 모델 목록을 준수합니다. 플릿에서 모델을 제한하려면 적용된 소스에 `availableModels`를 설정하세요.
* **`strictPluginOnlyCustomization`**: 이 키는 잠금과 상관없이 필터를 통과하며 Claude Code가 보호 훅을 포함하여 개발자 자체 커스텀 설정을 무시하도록 만듭니다. 어떠한 잠금도 이를 차단하지 않습니다.

### CI 파이프라인 및 원격 머신

무인 파이프라인을 위한 서비스 토큰 흐름은 없습니다. 게이트웨이 로그인은 항상 브라우저 디바이스 흐름을 실행하므로 로그인을 승인할 개발자가 없는 CI 작업은 인증할 수 없습니다. 이러한 작업은 제공업체에 직접 구성하세요.

개발자가 로그인하면 비대화형 `claude -p` 실행 및 Agent SDK에서 시작된 세션을 포함하여 해당 머신의 모든 Claude Code 호출이 게이트웨이 세션을 사용하며, [게이트웨이 정책이 모든 세션에 적용됩니다](/docs/en/claude-apps-gateway-config#managed).

디바이스 흐름은 폴링하는 CLI와 승인하는 브라우저를 분리하므로 디스플레이가 없는 원격 개발 상자에서도 동작합니다. 개발자는 원격 머신의 SSH를 통해 `/login`을 실행하고 랩톱 브라우저에서 검증 링크를 엽니다.

### 개발자에게 적용되는 항목

다음 보장은 로그인된 모든 게이트웨이 세션에 적용됩니다.

* **모델 접근**: 정책이 부여하지 않은 모델 요청은 400을 반환하며 `/model` 선택기는 정책의 `availableModels` 허용 목록으로 필터링됩니다. 정책에 [`enforceAvailableModels: true`](/docs/en/model-config#default-model-behavior)를 설정하여 Default 옵션이 Claude Code의 기본 내장 모델 대신 `availableModels` 내부의 모델로 해소되도록 하세요. 설정하지 않으면 Default가 선택 가능한 상태로 유지되며 부여되지 않은 모델인 경우 요청 시 거부됩니다.
* **텔레메트리 대상**: [텔레메트리 전달](/docs/en/claude-apps-gateway-config#telemetry)이 구성된 경우 OTLP 내보내기 엔드포인트가 게이트웨이에 고정되고 게이트웨이가 푸시한 설정이 로컬에 설정된 `OTEL_*` 변수를 재정의합니다.
* **자격 증명**: 게이트웨이 토큰은 세션의 유일한 자격 증명입니다. 로그인되어 있는 동안 `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_API_KEY`, `apiKeyHelper` 및 이전 claude.ai 로그인은 무시되므로 개발자가 먼저 claude.ai에서 로그아웃할 필요가 없습니다.
* **관리형 설정**: 잠긴 키는 로컬에서 재정의할 수 없습니다. CLI는 시작 시와 매시간 폴링 시 정책을 적용합니다.
* **시작**: 게이트웨이에 도달할 수 없는 경우 로그인된 세션은 설정 없이 시작되는 대신 약 10초 후에 에러와 함께 시작 시 종료됩니다.
* **권한 박탈**: IdP에서 비활성화된 사용자의 세션은 다음 새로 고침이 실패할 때 `ttl_hours` 이내에 만료됩니다.

### 조직이 볼 수 있는 내용

사용량 텔레메트리에는 개발자의 정체성, 토큰 수, 모델 및 지연 시간이 조직의 수집기로 전달됩니다. 게이트웨이는 프롬프트나 완성 내용을 기록하거나 저장하지 않습니다. 명령 및 파일 경로가 포함될 수 있는 로그 및 트레이스와 같은 풍부한 텔레메트리 수집 여부는 조직의 [대상별 선택 사항](/docs/en/claude-apps-gateway-config#telemetry)입니다.

## 사용 가능 여부 및 제약 사항

아래 표는 개발자가 게이트웨이를 통해 연결할 때 작동하는 Claude Code 기능과 게이트웨이 서버 자체가 지원하는 기능을 다룹니다. 지원되지 않는 경우 '참고' 열에 대안을 제공합니다.

게이트웨이는 운영자가 베타 허용 목록을 유지 관리하지 않도록 CLI가 전송하는 [`anthropic-beta`](https://platform.claude.com/docs/en/api/beta-headers) 값을 모든 상류에 전달합니다. 이 헤더를 무시하는 Amazon Bedrock의 경우 게이트웨이는 요청 본문의 `anthropic_beta` 필드로 값을 이동하며, 다른 상류는 전송된 대로 헤더를 수신합니다.

CLI의 게이트웨이 세션 베타 세트에는 퍼스트 파티 전용 베타 및 확장 캐시 TTL 베타가 생략되어 있으므로 아래 표의 해당 행이 '사용할 수 없음'으로 표시됩니다.

| 기능                                                                                                                       | 상태          | 참고                                                                                                                                                                                                                                                                                                                                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 추론 전달 (Amazon Bedrock, AWS 기반 Claude Platform, Google Cloud Agent Platform, Microsoft Foundry, Anthropic)             | 사용 가능     | 상류별 모델 변환 및 장애 조치 제공. Amazon Bedrock 상류는 `bedrock-runtime` 엔드포인트 및 AWS 기본 자격 증명 체인을 사용하며, Amazon Bedrock [Mantle 엔드포인트](/docs/en/amazon-bedrock#use-the-mantle-endpoint)는 지원되는 상류가 아닙니다. [AWS 기반 Claude Platform 상류](/docs/en/claude-apps-gateway-config#claude-platform-on-aws)를 사용하려면 게이트웨이 서버에 Claude Code v2.1.198 이상이 필요합니다. |
| IdP 그룹별 모델 접근 및 관리형 설정                                                                                        | 사용 가능     | 모델 접근은 서버 측에서 적용되며 관리형 설정은 IdP 그룹별로 전달되어 CLI에 의해 [관리형 설정 계층](/docs/en/settings#settings-precedence)에서 적용됩니다.                                                                                                                                                                                                                                                                              |
| 텔레메트리 팬아웃 (OTLP/HTTP)                                                                                              | 사용 가능     | 내보내기별로 정체성 스탬프 적용, protobuf 및 JSON 인코딩 모두 지원                                                                                                                                                                                                                                                                                                                                                               |
| OIDC 정체성 제공업체                                                                                                       | 사용 가능     | 모든 OIDC 준수 IdP 지원. 게이트웨이는 표준 OIDC 디스커버리 및 인가 코드 흐름을 실행합니다. IdP별 설정은 [정체성 제공업체 설정](/docs/en/claude-apps-gateway-deploy#identity-provider-setup)을 참조하세요.                                                                                                                                                                                                                                        |
| 사용자별 및 그룹별 지출 한도                                                                                               | 사용 가능     | [지출 한도](/docs/en/claude-apps-gateway-spend-limits) 참조                                                                                                                                                                                                                                                                                                                                                                      |
| 서버 측 웹 검색                                                                                                            | 사용할 수 없음| CLI가 게이트웨이가 라우팅하는 상류 제공업체를 볼 수 없어 웹 검색 지원을 검증할 수 없으므로 게이트웨이 세션에서 WebSearch를 비활성화합니다.                                                                                                                                                                                                                                                                                       |
| 표준 프롬프트 캐싱                                                                                                         | 사용 가능     | `cache_control` 브레이크포인트가 모든 상류로 전달됩니다.                                                                                                                                                                                                                                                                                                                                                                         |
| 1시간 캐시 TTL                                                                                                             | 사용할 수 없음| 게이트웨이가 라우팅할 수 있는 모든 상류가 1시간 TTL을 지원하지는 않기 때문에 CLI는 게이트웨이 세션에서 extended-cache-ttl 베타를 생략하므로 게이트웨이를 통한 프롬프트 캐싱은 5분 TTL을 사용합니다. 위의 베타 헤더 참고사항 참조.                                                                                                                                                                                                 |
| 전역 캐시 범위 및 토큰 효율적 도구와 같은 퍼스트 파티 전용 최적화                                                          | 사용할 수 없음| CLI가 게이트웨이 세션에서 이를 활성화하지 않습니다. 위의 베타 헤더 참고사항 참조.                                                                                                                                                                                                                                                                                                                                                 |
| OTLP/gRPC                                                                                                                  | 지원되지 않음 | HTTP 기반 OTLP만 지원                                                                                                                                                                                                                                                                                                                                                                                                            |
| SAML, LDAP 및 기타 비 OIDC 인증                                                                                            | 지원되지 않음 | OIDC만 지원. 필요한 경우 앞에 OIDC 브릿지를 두세요.                                                                                                                                                                                                                                                                                                                                                                              |
| 멀티 테넌트 (여러 OIDC 발급자)                                                                                             | 지원되지 않음 | 게이트웨이당 하나의 발급자만 지원. 별도의 인스턴스를 실행하세요.                                                                                                                                                                                                                                                                                                                                                                  |
| Windows 서버                                                                                                               | 지원되지 않음 | Linux에 배포하세요. macOS는 로컬 개발용으로만 사용하세요.                                                                                                                                                                                                                                                                                                                                                                         |
| Helm 차트                                                                                                                  | 사용할 수 없음| 게이트웨이는 표준 상태 없는(stateless) Deployment로 실행됩니다. [배포 가이드](/docs/en/claude-apps-gateway-deploy#kubernetes)를 참조하세요.                                                                                                                                                                                                                                                                                      |
| 관리자 UI                                                                                                                  | 사용할 수 없음| 설정은 YAML 파일로 관리되며 변경하려면 재배포하세요.                                                                                                                                                                                                                                                                                                                                                                             |

## 다음 단계

퀵스타트에서는 Docker Compose 기반의 최소 설정이 완성되었습니다. 이를 더 발전시키려면:

* 그룹별 RBAC, 다중 상류 장애 조치 또는 텔레메트리 대상을 추가하기 위해 `gateway.yaml`을 확장하세요. [설정 참조](/docs/en/claude-apps-gateway-config)에서 모든 옵션을 다룹니다.
* Compose에서 Kubernetes 또는 Cloud Run 기반 프로덕션 배포로 이동하고 IdP를 올바르게 설정하며 보안 모델을 검토하세요. [배포 및 운영 가이드](/docs/en/claude-apps-gateway-deploy)에서 IdP별 설정, 컨테이너 이미지 요구 사항, 헬스 프로브, 문제 해결을 다룹니다.
* 일제히 발생하는 작업이 전체 할당량을 소비하지 않도록 개별 개발자나 그룹에 지출 상한선을 설정하세요. [지출 한도](/docs/en/claude-apps-gateway-spend-limits)에서 관리자 API 및 적용 방식을 다룹니다.
* Cloud Run, Cloud SQL, Secret Manager를 포함한 Google Cloud에서의 전체 구성 예제는 [Google Cloud 배포](/docs/en/claude-apps-gateway-on-gcp)를 참조하세요.
