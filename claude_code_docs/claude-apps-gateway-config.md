> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude apps gateway 설정 (Configuration)

> 모든 gateway.yaml 옵션에 대한 참조 설명서: 수신기 및 TLS, OIDC, 세션, Postgres 저장소, Amazon Bedrock, AWS 기반 Claude Platform, Google Cloud Agent Platform 및 Microsoft Foundry 상류, 모델 라우팅, 관리형 정책 및 텔레메트리.

Claude apps gateway 배포는 관례적으로 `gateway.yaml`로 명명되는 하나의 YAML 파일로 설정됩니다. 이 파일은 수신 대기 위치, 개발자 로그인 방식, 추론 라우팅 위치, 적용되는 정책 및 텔레메트리 등 게이트웨이가 수행하는 모든 작업을 정의합니다. 이 페이지는 해당 파일의 모든 옵션에 대한 참조 문서입니다.

첫 번째 파일을 작성하려면 최소한의 작동 설정을 구성하고 실행하는 [퀵스타트](/docs/en/claude-apps-gateway#quickstart)에서 시작하세요. 만족스러운 설정이 준비되면 [배포 가이드](/docs/en/claude-apps-gateway-deploy)에서 이를 컨테이너화하여 Kubernetes, Cloud Run 또는 자체 플랫폼에 호스팅하는 방식을 다룹니다.

게이트웨이는 시작 시 `claude gateway --config /path/to/gateway.yaml` 명령을 통해 이 파일을 한 번 읽습니다. 부팅 시 모든 옵션이 스키마와 비교하여 검증되므로, 형식이 잘못된 설정은 첫 사용 시점이 아니라 시작 시점에 필드 수준 오류와 함께 실패합니다.

이 페이지 끝에 있는 [전체 예제](#전체-예제)에서는 모든 섹션을 종합하여 보여줍니다.

## 파일 구조

5개 섹션은 [필수 항목](#필수-섹션)입니다. 다른 모든 섹션은 [선택 사항](#선택-섹션)이며, 생략된 섹션은 기본값을 취합니다. 알 수 없는 키는 부팅 시 실패하므로 오탈자가 발생하는 경우 무시되는 대신 명시적인 에러로 표시됩니다.

**필수 섹션:**

* [`listen`](#listen): 바인드 주소, 공개 URL, TLS 종료
* [`oidc`](#oidc): 발급자(issuer), 클라이언트, 클레임 매핑 및 로그인 가능 대상을 포함한 정체성 제공업체(IdP)
* [`session`](#session): 비밀 값 및 수명을 포함하여 게이트웨이가 발급하는 전달자 토큰(bearer token)
* [`store`](#store): 디바이스 인가 및 속도 제한 카운터용 PostgreSQL
* [`upstreams`](#upstreams): Anthropic, Amazon Bedrock, AWS 기반 Claude Platform, Google Cloud Agent Platform 또는 Microsoft Foundry 등 추론이 라우팅되는 위치

**선택 섹션:**

* [`admin`](#admin): 지출 한도를 위한 관리자 API 인증 및 보존 정책
* [`enforcement`](#enforcement): 지출 한도 감지 시의 fail-open 또는 fail-closed 동작
* [`models`](#models) 및 `auto_include_builtin_models`: 관리자가 구성한 모델 목록 및 상류별 ID
* [`managed`](#managed): IdP 그룹별 관리형 설정 정책
* [`telemetry`](#telemetry): 관측 가능성 스택으로의 OTLP 전달
* [`access_control`, `limits`, `timeouts`, `rate_limits`](#http-조정): IP 허용/거부, 요청 크기 제한, 상류 TTFB(time-to-first-byte) 및 IP별 로그인 제한

## 비밀 값 확장

`client_secret`, `jwt_secret`, `postgres_url`과 같은 비밀 값을 `gateway.yaml`에 직접 작성하지 마세요. 아래 형식 중 하나로 이를 참조하면 게이트웨이가 부팅 시 환경 변수나 파일에서 해당 값을 해소합니다:

| 형식            | 해소 대상                                               | 사용 용도                                                                |
| --------------- | ------------------------------------------------------- | ------------------------------------------------------------------------ |
| `${VAR}`        | 환경 변수 `VAR`. 정의되지 않은 경우 부팅 실패.          | 컨테이너 환경 변수, 환경 변수 주입을 통한 AWS Secrets Manager            |
| `${file:/path}` | 트리밍된 파일 내용                                      | Kubernetes Secret 볼륨 마운트, Vault Agent, SOPS                         |

## 필수 섹션

### `listen`

`listen` 블록은 게이트웨이가 서비스를 제공하는 위치(바인드 주소 및 포트, 외부 노출 오리진, 선택적 TLS 종료)를 제어합니다.

| 필드                   | 필수 여부      | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `host`                 | 아니오         | 바인드 주소. 기본값 `0.0.0.0`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `port`                 | 아니오         | 바인드 포트. 기본값 `8080`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `public_url`           | 프록시 뒤 필수 | IdP `redirect_uri` 및 디스커버리 메타데이터를 구성하는 데 사용되는 외부 노출 `https://` 오리진. 게이트웨이는 자체 오리진을 구성할 때 클라이언트가 위조할 수 있는 `X-Forwarded-*` 헤더를 신뢰하지 않으므로 ALB, Ingress 또는 Cloud Run과 같은 TLS 종료 프록시 뒤에서 필수적입니다. 아래 `trusted_proxies`는 클라이언트 IP 해소만 통제합니다. 게이트웨이가 이 URL에서 클라이언트에 푸시하는 OTLP 엔드포인트를 구축하므로 [텔레메트리](#telemetry)를 활성화하는 데도 필요합니다.                          |
| `tls.cert` / `tls.key` | 아니오         | 게이트웨이가 TLS를 직접 종료하는 경우 PEM 경로                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `trusted_proxies`      | 아니오         | 게이트웨이 앞단 로드 밸런서의 CIDR 또는 IP. 설정 시 게이트웨이는 이들 피어에서만 `X-Forwarded-For`를 신뢰하며 IP별 속도 제한 및 감사를 위해 실제 클라이언트 IP를 기록합니다. nginx `set_real_ip_from`과 동일합니다.                                                                                                                                                                                                                                                                                 |

### `oidc`

`oidc` 블록은 게이트웨이를 정체성 제공업체에 연결하고 로그인할 수 있는 대상을 결정합니다. 발급자 및 OAuth 클라이언트의 이름을 지정하고 이메일과 그룹을 전달하는 클레임을 매핑하며 이메일 도메인이나 그룹별로 로그인을 제한합니다.

OpenID Connect (OIDC)는 게이트웨이가 정체성 제공업체와 사용하는 SSO 프로토콜입니다. IdP 측에 등록할 내용은 [정체성 제공업체 설정](/docs/en/claude-apps-gateway-deploy#identity-provider-setup)을 참조하세요.

| 필드                            | 필수 여부 | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `issuer`                        | 예        | OIDC 디스커버리 베이스. `/.well-known/openid-configuration`에서 디스커버리를 제공해야 합니다. 프로덕션에서는 HTTPS를 사용하세요. 게이트웨이는 `http://` 발급자를 수락하지만, 게이트웨이 환경에 `CLAUDE_GATEWAY_ALLOW_LOOPBACK=1`이 설정되지 않은 한 `http://localhost:8081`과 같은 루프백 발급자는 [SSRF 보호조치](/docs/en/claude-apps-gateway-deploy#threat-model-summary)에 의해 거부됩니다.                                                                                                                                                                                                      |
| `client_id` / `client_secret`   | 예        | OAuth 클라이언트 등록 정보에서 가져옵니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `allowed_email_domains`         | 아니오    | `email` 클레임이 이들 도메인 중 하나에 속하지 않는 id_token을 거부합니다(대소문자 구분 없음). 다중 테넌트 IdP 잘못된 설정에 대한 심층 방어용입니다. 이 설정과 별개로 `email_verified` 클레임이 명시적으로 `false`인 id_token은 항상 거부됩니다.                                                                                                                                                                                                                                                                                                                                                                 |
| `allowed_groups`                | 아니오    | 이들 IdP 그룹의 구성원으로 로그인을 제한합니다(`groups_claim`과 일치). 허용된 이메일 도메인에 속하지만 이들 그룹 중 어디에도 속하지 않는 사용자는 거부됩니다. IdP가 그룹 클레임을 내보내야 합니다.                                                                                                                                                                                                                                                                                                                                                                                                |
| `groups_claim`                  | 아니오    | 어떤 id_token 클레임이 그룹 멤버십을 전달하는지 지정합니다. 기본값 `groups`. Microsoft Entra는 `roles` 아래에 앱 역할을 내보냅니다. 플랫 키 또는 중첩된 클레임에 대한 RFC 6901 JSON Pointer(예: `/resource_access/gateway/roles`)를 수락합니다.                                                                                                                                                                                                                                                                                                                                                         |
| `google_groups`                 | 아니오    | Google의 id_token에는 그룹 클레임이 전달되지 않으므로 Google Workspace Admin SDK Directory API를 통해 로그인한 사용자의 그룹을 조회합니다. `service_account_json_path`를 `https://www.googleapis.com/auth/admin.directory.group.readonly` 범주에 대해 도메인 전체 위임 권한이 있는 서비스 계정 키 파일로 설정하고, `admin_email`을 서비스 계정이 사칭할 Workspace 관리자로 설정하세요. Directory API에는 실제 관리자 주체가 필요합니다. 각 사용자의 그룹 이메일 주소가 그룹 클레임이 되므로 `allowed_groups` 및 `managed.policies.match.groups`는 그룹 이메일과 일치합니다.                       |
| `email_claim`                   | 아니오    | 어떤 id_token 클레임이 사용자의 이메일을 전달하는지 지정합니다. 기본값 `email`. ADFS 및 Entra B2C와 같은 일부 IdP는 대신 `upn` 또는 `preferred_username`을 내보냅니다. 플랫 키, JSON Pointer 또는 첫 번째 존재하는 키가 사용되는 대체 키 목록을 수락합니다.                                                                                                                                                                                                                                                                                                                                               |
| `scopes`                        | 아니오    | 게이트웨이가 요청하는 OIDC 스코프의 전체 재정의. 기본값 `[openid, profile, email, offline_access]`. IdP가 인식하지 못하는 스코프를 거부하거나 그룹 또는 이메일을 내보내기 위해 사용자 지정 스코프가 필요할 때 설정합니다. `openid`를 포함해야 합니다. `offline_access`를 제외하면 리프레시 토큰이 비활성화되므로 개발자가 `session.ttl_hours`마다 브라우저 로그인을 다시 실행하게 됩니다. Google의 리프레시 토큰 흐름과 같은 IdP별 스코프 설정법은 [정체성 제공업체 설정](/docs/en/claude-apps-gateway-deploy#identity-provider-setup)을 참조하세요.                                                 |
| `extra_auth_params`             | 아니오    | IdP 인가 요청에 있는 그대로 추가되는 추가 쿼리 파라미터. Google 리프레시 토큰용 `access_type: offline`, 일부 Entra 테넌트용 `domain_hint`, 단계별 인가 흐름용 `acr_values` 등 IdP 전용 동작을 재정의하는 메커니즘입니다. 게이트웨이가 관리하는 프로토콜 파라미터(`state`, `nonce`, `redirect_uri`, PKCE, `scope`, `response_type`, `response_mode`, `client_id`)는 재정의할 수 없습니다.                                                                                                                                                                                                   |
| `userinfo_fallback`             | 아니오    | id_token에서 이메일이나 그룹이 누락된 경우 `/userinfo`에서 이를 가져옵니다. Keycloak 라이트웨이트 접근 토큰, Okta 조직 서버 및 ADFS 최소 토큰에 필요합니다. id_token이 계속 권한을 가지며 userinfo는 빈 부분만 채웁니다. 기본값 `false`.                                                                                                                                                                                                                                                                                                                                                    |
| `use_pkce`                      | 아니오    | 인가 요청 시 PKCE (S256) 챌린지를 전송합니다. 기본값 `true`. 이 기밀 클라이언트에 대해 IdP가 PKCE를 거부하는 경우에만 `false`로 설정하세요.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `clock_skew_seconds`            | 아니오    | id_token 시간 클레임을 검증할 때 허용할 시계 오차(clock drift). 기본값 `0`(엄격함). 호스트/IdP 시계 오차로 인해 로그인 직후 "token expired / not yet valid" 오류가 발생하는 경우 이 값을 올리세요.                                                                                                                                                                                                                                                                                                                                                                                                    |
| `token_endpoint_auth_method`    | 아니오    | 토큰 엔드포인트 인증 방식을 재정의합니다. `client_secret_basic` 또는 `client_secret_post`를 수락합니다. 기본적으로 자동 협상됩니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `id_token_signed_response_alg`  | 아니오    | 예상되는 id_token 서명 알고리즘. 기본값 `RS256`. ES256, PS256 또는 EdDSA로 서명하는 IdP의 경우 설정하세요.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `additional_authorized_parties` | 아니오    | Keycloak 브로커 및 토큰 교환 흐름을 위해 `client_id` 외에 수락할 추가 `azp` 값                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `discovery_url`                 | 아니오    | 발급자 호스트를 다시 쓰는 프록시 뒤의 IdP를 위해 `issuer`에서 도출하는 대신 이 URL에서 디스커버리 문서를 가져옵니다. 경로에 `/.well-known/`이 포함되어야 합니다.                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `form_action_origins`           | 아니오    | `/device` 페이지의 `Content-Security-Policy: form-action` 지시문에 대한 추가 오리진. 게이트웨이는 이미 `'self'` 및 탐지된 `authorization_endpoint` 오리진을 허용하지만 Chrome은 전체 리다이렉트 체인에 대해 `form-action`을 적용합니다. IdP가 ADFS로 페더레이션된 Azure AD, 허브-스포크 Okta 또는 기업 SSO 인터셉터와 같은 두 번째 호스트를 통해 리다이렉트하는 경우 인가 요청이 리다이렉트될 수 있는 모든 오리진을 나열하세요.                                                                                                                                                 |
| `ca_cert_pem`                   | 아니오    | IdP 요청에 대해서만 시스템 신뢰 저장소를 대체하는 PEM CA 인증서. 기업 PKI 뒤의 Keycloak 또는 Dex에 사용하세요.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

### `session`

`session` 블록은 로그인 후 게이트웨이가 발급하는 전달자 토큰의 형식(서명 비밀 값 및 수명)을 설정합니다.

| 필드        | 필수 여부 | 설명                                                                                                                                                                                                                                                                                                                                                                                                           |
| ----------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `jwt_secret`| 예        | 최소 32바이트 엔트로피(예: `openssl rand -base64 32`). 게이트웨이의 HS256 전달자 토큰에 서명합니다. 서명 순환을 위해 단일 문자열 또는 배열을 수락합니다. 인덱스 0이 서명하고 모든 항목이 검증합니다. 순환하려면 새 비밀 값을 앞에 추가하고 `ttl_hours` 동안 기다린 다음 이전 값을 제거하세요.                                                                                                                   |
| `ttl_hours` | 아니오    | 게이트웨이 전달자 토큰 수명. 기본값 `1`. IdP가 리프레시 토큰을 발급할 때 CLI는 만료 전에 자동으로 배경에서 새로 고칩니다. 수명이 짧을수록 자격 박탈이 빨리 적용되고, 수명이 길수록 IdP 왕복 횟수가 줄어듭니다. `offline_access`를 사용할 수 없어 IdP가 리프레시 토큰을 발급할 수 없는 경우 자동 새로 고침이 없으므로 개발자가 매시간 브라우저 로그인을 다시 실행하지 않도록 이 값을 `8` 또는 `12`로 올리세요. |

### `store`

`store` 블록은 디바이스 인가 및 속도 제한 카운터를 보유하는 PostgreSQL 데이터베이스를 지정합니다.

| 필드             | 필수 여부 | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ----------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `postgres_url`    | 예        | `postgres://` 또는 `postgresql://` URL. 필수: 브라우저 콜백이 쓰고 폴링 CLI가 읽는 디바이스 인가 공유 공간에는 교차 레프리카 상태가 필요합니다. 게이트웨이는 부팅 시 자체 스키마 마이그레이션을 실행하므로 해당 역할에는 대상 스키마에 대한 `CREATE TABLE` 권한이 필요합니다. 보안 정책상 애플리케이션 역할의 DDL이 금지된 경우, 처음 및 마이그레이션이 포함된 새 릴리스가 배포될 때마다 관리자 역할로 마이그레이션을 실행하고 게이트웨이 테이블에 대해 앱 역할에 `SELECT, INSERT, UPDATE, DELETE` 권한을 부여하세요. [업데이트](/docs/en/claude-apps-gateway-deploy#upgrades) 및 [Postgres](/docs/en/claude-apps-gateway-deploy#postgres)를 참조하세요. |
| `username`        | 아니오    | `postgres_url` 내부의 사용자를 재정의합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `password`        | 아니오    | 데이터베이스 자격 증명. 자격 증명이 URL에 노출되지 않도록 `postgres_url` 대신 여기에 설정하세요. 모든 문자를 수락하며 URL 자격 증명보다 우선합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `max_connections` | 아니오    | 레프리카당 Postgres 연결 풀 크기. 기본값 `5`(공유 데이터베이스에 적합한 보수적인 값). [지출 한도](#admin)가 활성화된 경우 핫 패스는 추론 요청당 몇 가지 작업을 수행하므로 부하가 있는 전용 데이터베이스의 경우 이 값을 올리고, 레프리카 수 × 이 값이 데이터베이스의 `max_connections` 미만이 되도록 유지하세요.                                                                                                                                                                                                                                                                                                                                 |

로컬 개발의 경우 `postgres_url`을 일회성 Postgres 컨테이너(예: `docker run --rm -p 5432:5432 -e POSTGRES_HOST_AUTH_METHOD=trust postgres`)로 지정하세요.

### `upstreams`

`upstreams`는 순서가 있는 목록입니다. 게이트웨이는 요청된 모델을 처리하는 첫 번째 상류로 추론을 전달합니다. `5xx`, `429`, `401`, `403`, `404` 또는 타임아웃 발생 시 다음으로 장애 조치(failover)합니다. 기타 `4xx`는 상류가 아닌 요청 자체에 기인한 오류이므로 장애 조치하지 않습니다. `401` 또는 `403`은 해당 상류에 대한 게이트웨이 자체 자격 증명이 실패했음을 의미하며 `404`는 해당 상류가 요청된 모델을 제공하지 않음을 의미하므로 목록의 이후 상류가 이를 처리할 수 있습니다.

`404` 시 장애 조치하려면 게이트웨이 v2.1.198 이상이 필요합니다. 이전 릴리스에서는 목록의 이후 상류가 모델을 제공하더라도 클라이언트에 첫 번째 `404`를 반환했습니다.

동일한 제공업체의 여러 상류에는 고유한 `name:`을 설정해야 합니다.

Amazon Bedrock, AWS 기반 Claude Platform, Google Cloud Agent Platform 및 Microsoft Foundry 클라이언트는 시작 시 한 번 빌드되며 해당 SDK가 내부적으로 자격 증명을 새로 고치므로 클라우드 자격 증명을 순환할 때 재시작이 필요하지 않습니다. 정적 Anthropic API 키 및 전달자 토큰은 시작 시 읽혀집니다. [Anthropic API](#anthropic-api)를 참조하세요.

#### Anthropic API

최소한의 Anthropic 상류 설정은 [Claude Console](https://platform.claude.com)에서 발급한 API 키입니다:

```yaml theme={null}
upstreams:
  - provider: anthropic
    auth:
      api_key: ${ANTHROPIC_API_KEY}
    # 또는 OAuth 전달자 토큰 (예: Workload Identity Federation 교환 토큰):
    #   oauth_token: ${file:/var/run/secrets/anthropic-oauth-token}
    # base_url: https://api.anthropic.com   # 기본값; 포워드 프록시용으로 재정의
```

두 자격 증명 형식은 전송하는 헤더가 다릅니다:

* **`api_key`**: `x-api-key`를 전송합니다. Claude Console에서 순환하고 환경 변수를 업데이트하세요.
* **`oauth_token`**: `Authorization: Bearer`를 전송합니다. 조직이 장기 API 키 대신 단기 토큰을 발급할 때 전달자 형식을 사용하세요. 전달자는 시작 시 한 번 읽히므로 비밀 값을 재마운트하고 재시작하여 새로 고치세요.

정적 키나 전달자 토큰 대신 Workload Identity Federation을 사용할 수 있습니다. [Workload Identity Federation 가이드](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)에 따라 페더레이션 규칙을 생성한 다음 워크로드의 OIDC JWT(Kubernetes 프로젝션 서비스 계정 토큰 또는 CI 플랫폼의 id-token 등)를 파일로 마운트하세요. 게이트웨이는 JWT를 단기 전달자 토큰으로 교환하고 자동으로 새로 고칩니다. 토큰 파일은 매 교환 시마다 다시 읽히므로 순환된 프로젝션 토큰이 재시작 없이 적용됩니다.

```yaml theme={null}
upstreams:
  - provider: anthropic
    auth:
      federation_rule_id: ${ANTHROPIC_FEDERATION_RULE_ID}
      organization_id: ${ANTHROPIC_ORGANIZATION_ID}
      identity_token_file: /var/run/secrets/anthropic/id-token
      # workspace_id: wrkspc_...       # 규칙이 1개 이상의 워크스페이스를 다루는 경우 필수
      # service_account_id: svac_...   # 선택적 대상 검사
```

#### Amazon Bedrock

게이트웨이가 대체하거나 앞단에 위치하는 클라이언트 측 Amazon Bedrock 배포의 경우 [Amazon Bedrock에서의 Claude Code](/docs/en/amazon-bedrock)를 참조하세요. 게이트웨이 측 상류 설정:

```yaml theme={null}
upstreams:
  - provider: bedrock
    region: us-east-1
    auth: {}                           # 권장: AWS 기본 자격 증명 체인
    # 또는 명시적 자격 증명:
    # auth:
    #   aws_access_key_id: ${AWS_AKID}
    #   aws_secret_access_key: ${AWS_SK}
    #   aws_session_token: ${AWS_ST}
    # 또는 Bedrock API 전달자 토큰:
    # auth:
    #   aws_bearer_token: ${AWS_BEARER_TOKEN}
    # FIPS 또는 VPC 엔드포인트 배포를 위한 bedrock-runtime 엔드포인트 재정의:
    # base_url: https://bedrock-runtime-fips.us-east-1.amazonaws.com
```

비어 있는 `auth` 블록은 AWS SDK의 기본 자격 증명 체인(환경 변수, `~/.aws/credentials`, ECS 태스크 역할, EC2 인스턴스 메타데이터 또는 EKS의 IRSA)을 사용합니다. 프로덕션 환경에서는 컨테이너 이미지에 정적 키를 내장하는 대신 게이트웨이 파드에 IAM 역할을 부여하세요.

명시적 자격 증명은 완전해야 합니다: `aws_access_key_id`와 `aws_secret_access_key`가 함께 설정되지 않거나 이들 없이 `aws_session_token`만 설정된 경우 게이트웨이는 부팅 시 실패합니다. v2.1.207 이전에는 부분적인 `auth:` 블록도 검증을 통과했습니다.

| 구성 요소       | 설정 방법                                                                                                                                                                                                                                                                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IAM 권한        | 추론 프로필 ARN 및 기본 파운데이션 모델 ARN 모두에 대해 게이트웨이 보안 주체에 `bedrock:InvokeModel` 및 `bedrock:InvokeModelWithResponseStream` 권한을 부여하세요. 미국 지역의 기본 제공 카탈로그: `arn:aws:bedrock:<region>:<account>:inference-profile/us.anthropic.*` 및 `arn:aws:bedrock:*::foundation-model/anthropic.*`.              |
| 모델 접근       | Amazon Bedrock 콘솔에서 지역별로 원하는 Claude 모델에 대해 모델 접근을 요청하고 활성화하세요. 교차 지역 추론 프로필(`us.anthropic.*`)은 프로필이 적용되는 각 지역에서 모델 접근이 필요합니다.                                                                                                                                                   |
| EKS (IRSA)      | 위의 정책과 게이트웨이 서비스 계정 범위로 설정된 클러스터 OIDC 제공업체용 신뢰 정책을 사용하여 IAM 역할을 생성하세요. 서비스 계정에 `eks.amazonaws.com/role-arn: arn:aws:iam::<acct>:role/claude-gateway` 어노테이션을 추가하면 `auth: {}`가 이를 가져옵니다.                                                                                      |
| ECS / EC2       | 태스크 정의 또는 인스턴스 프로필에 IAM 역할을 연결합니다. `auth: {}`가 이를 가져옵니다.                                                                                                                                                                                                                                                           |
| 기타 환경       | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 및 `AWS_SESSION_TOKEN` 환경 변수를 통해 자격 증명을 전달하거나, `${VAR}` 확장을 사용해 `auth:`에 명시적으로 설정하세요.                                                                                                                                                                            |
| 지역 (Region)   | `region:`은 API 엔드포인트 지역입니다. 교차 지역 추론 프로필은 어느 지역을 선택하든 지리적 영역(US, EU, APAC)에 따라 라우팅됩니다. 미국 외 지역 또는 프로비저닝된 처리량 ARN의 경우 적절한 상류별 ID가 포함된 [`models:`](#models) 블록을 추가하세요.                                                                                          |

#### AWS 기반 Claude Platform

AWS 기반 Claude Platform은 `aws-external-anthropic.<region>.api.aws`에서 AWS 인프라상의 퍼스트 파티 Anthropic API를 제공합니다. 퍼스트 파티 모델 ID를 사용하고전송된 `anthropic-beta` 헤더를 준수하며 `count_tokens`를 제공하므로 Bedrock 전용 변환이 적용되지 않습니다. `anthropicAws` 제공업체에는 Claude Code v2.1.198 이상이 필요합니다. 이전 게이트웨이 릴리스는 부팅 시 이를 거부합니다.

동일한 플랫폼의 클라이언트 측 배포는 [AWS 기반 Claude Platform에서의 Claude Code](/docs/en/claude-platform-on-aws)를 참조하세요. 게이트웨이 측 상류 설정:

```yaml theme={null}
upstreams:
  - provider: anthropicAws
    region: us-east-1
    workspace_id: wrkspc_...
    auth:
      api_key: ${ANTHROPIC_AWS_API_KEY}   # x-api-key로 전송됨
    # 또는 AWS 기본 자격 증명 체인을 통한 SigV4:
    # auth: {}
    # 또는 명시적 SigV4 자격 증명:
    # auth:
    #   aws_access_key_id: ${AWS_ACCESS_KEY_ID}
    #   aws_secret_access_key: ${AWS_SECRET_ACCESS_KEY}
    # 도출된 엔드포인트 재정의:
    # base_url: https://aws-external-anthropic.us-east-1.api.aws
```

플랫폼은 Amazon Bedrock과 별개의 AWS 계정에서 실행되며 자체 서비스 이름인 `aws-external-anthropic`에 대해 SigV4 요청을 서명하므로 Bedrock 범주의 IAM 역할로는 인가되지 않습니다. SigV4 자격 증명도 설정된 경우 `auth.api_key`에 있는 API 키가 우선합니다. 비어 있는 `auth` 블록은 [Amazon Bedrock](#amazon-bedrock) 상류가 사용하는 것과 동일한 AWS SDK의 기본 자격 증명 체인을 사용합니다.

| 필드                                                    | 필수 여부 | 설명                                                                                                                                                |
| ------------------------------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `region`                                                | 예        | AWS 지역 (소문자, 숫자, 하이픈). 게이트웨이는 이를 기반으로 `https://aws-external-anthropic.<region>.api.aws`와 같이 엔드포인트를 도출합니다.     |
| `workspace_id`                                          | 예        | 모든 요청에 헤더로 전송되며 플랫폼에서 필수입니다.                                                                                                  |
| `auth.api_key`                                          | 아니오    | 플랫폼용 API 키로 `x-api-key`로 전송됩니다. 전달자 토큰이 아닙니다. 두 인증 모드는 API 키 또는 SigV4입니다.                                        |
| `auth.aws_access_key_id` / `auth.aws_secret_access_key` | 아니오    | 명시적 SigV4 자격 증명. 하나만 설정하고 다른 하나를 설정하지 않으면 부팅 시 실패합니다. `auth.aws_session_token`도 함께 수락됩니다.              |
| `base_url`                                              | 아니오    | 도출된 엔드포인트 재정의                                                                                                                            |

플랫폼이 퍼스트 파티 모델 ID를 직접 해소하므로 내장 카탈로그는 [`models:`](#models) 블록 없이도 플랫폼으로 라우팅됩니다. `models:` 목록을 직접 구성할 때 항목의 키를 퍼스트 파티 ID와 함께 `anthropicAws:`로 지정하세요.

#### Google Cloud Agent Platform

동일한 클라이언트 측 설정은 [Google Cloud에서의 Claude Code](/docs/en/google-vertex-ai)를 참조하세요. 게이트웨이 측 상류 설정:

```yaml theme={null}
upstreams:
  - provider: vertex
    region: us-east5
    project_id: example-prod
    auth: {}                           # 권장: Application Default Credentials
    # 또는 서비스 계정 키 파일:
    # auth: { service_account_json: /secrets/sa.json }
    # Private Service Connect를 위한 aiplatform 엔드포인트 재정의:
    # base_url: https://us-east5-aiplatform.p.googleapis.com
```

비어 있는 `auth` 블록은 Application Default Credentials(`GOOGLE_APPLICATION_CREDENTIALS`, GCE 메타데이터 또는 GKE Workload Identity)를 사용합니다. 서비스 계정 JSON 키 파일이 지원되지만 권장되지 않습니다. Workload Identity를 사용하거나 GCE 또는 Cloud Run 인스턴스에 서비스 계정을 연결하세요.

지역 엔드포인트 대신 [Google Cloud Agent Platform의 전역 엔드포인트](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/locations)를 사용하려면 `region: global`을 설정하세요. 그러면 Google이 사용 가능한 지역으로 각 요청을 라우팅하므로 지역별 모델 가용성을 추적할 필요가 없습니다. 특정 지역을 설정하면 모든 요청이 해당 지역으로 고정됩니다.

| 구성 요소               | 설정 방법                                                                                                                                                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IAM 권한                | 게이트웨이의 서비스 계정에 프로젝트에 대한 `roles/aiplatform.user` 권한 또는 `aiplatform.endpoints.predict`가 포함된 커스텀 역할을 부여하세요. Google Cloud Agent Platform API(`aiplatform.googleapis.com`)를 활성화하세요.           |
| 모델 접근               | Model Garden에서 프로젝트에 대해 Claude 모델을 활성화하세요. 모델은 특정 지역에 게시되므로 모델 카드에서 지원되는 지역을 확인하세요.                                                                                                      |
| GKE (Workload Identity) | GCP 서비스 계정을 게이트웨이의 Kubernetes 서비스 계정에 바인딩하고 KSA에 `iam.gke.io/gcp-service-account: claude-gateway@<proj>.iam.gserviceaccount.com` 어노테이션을 추가하세요. `auth: {}`가 이를 가져옵니다.                        |
| Cloud Run / GCE         | 서비스의 서비스 계정을 `roles/aiplatform.user` 권한이 있는 계정으로 설정하세요. `auth: {}`가 이를 가져옵니다.                                                                                                                            |
| 기타 환경               | 비밀로 마운트된 JSON 키 파일 경로인 `auth: { service_account_json: /secrets/sa.json }`. 필드는 키 내용이 아닌 파일 경로를 받으므로 `${file:…}` 확장이 필요하지 않습니다.                                                                  |

#### Microsoft Foundry

클라이언트 측 Microsoft Foundry 배포의 경우 [Microsoft Foundry에서의 Claude Code](/docs/en/microsoft-foundry)를 참조하세요. 게이트웨이 측 상류 설정:

```yaml theme={null}
upstreams:
  - provider: foundry
    resource: example-foundry              # https://example-foundry.services.ai.azure.com
    auth: { use_azure_ad: true }        # 권장: DefaultAzureCredential / Managed Identity
    # 또는 API 키:
    # auth:
    #   api_key: ${FOUNDRY_API_KEY}
```

`use_azure_ad: true`는 `DefaultAzureCredential`(AKS, ACI 또는 App Service의 Managed Identity, Azure CLI 또는 환경 자격 증명)을 통해 해소됩니다. API 키도 작동하지만 프로젝트 범위에 한정되며 자동으로 순환되지 않습니다. Microsoft Foundry 엔드포인트는 `resource:`에서 도출됩니다. Azure Government와 같은 소버린 클라우드(sovereign cloud)의 경우 옵션 `base_url`을 설정하여 재정의하세요.

| 구성 요소               | 설정 방법                                                                                                                                                                                                |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RBAC                    | Microsoft Foundry 리소스에 대해 게이트웨이 정체성에 `Azure AI User` 또는 `Cognitive Services User` 권한을 부여하세요.                                                                                    |
| 배포 (Deployments)      | Microsoft Foundry는 정식 모델 ID가 아닌 관리자가 지정한 배포 이름을 사용합니다. 각 정식 ID를 배포 이름에 매핑하는 [`models:`](#models) 블록을 추가하세요.                                               |
| AKS (Workload Identity) | 사용자 할당 관리형 정체성(User-Assigned Managed Identity)을 클러스터의 OIDC 발급자와 페더레이션하고 게이트웨이 서비스 계정에 바인딩하세요. `use_azure_ad: true`가 `WorkloadIdentityCredential`을 통해 이를 가져옵니다. |
| ACI / App Service       | 리소스에 시스템 할당 또는 사용자 할당 관리형 정체성을 활성화하세요. `use_azure_ad: true`가 이를 가져옵니다.                                                                                              |
| 기타 환경               | `auth: { api_key: "${FOUNDRY_API_KEY}" }`. `{ }` 내부의 `${…}`를 따옴표로 감싸세요.                                                                                                                      |

#### 다중 상류 (Multiple upstreams)

동일한 제공업체가 고유한 `name:`을 가지고 2번 이상 나타날 수 있습니다. 이 기능은 서로 다른 지역, 서로 다른 자격 증명 체인을 통한 서로 다른 계정, 프로비저닝된 처리량 대 온디맨드, 교차 제공업체 대체(fallback)를 다룹니다.

게이트웨이는 순서대로 상류를 시도합니다. `5xx`, `429`, `401`, `403`, `404`, 타임아웃 및 미구현 엔드포인트(`501`) 발생 시 장애 조치되며 기타 `4xx` 발생 시에는 하지 않습니다.

`429`는 상류별 용량이므로 프로비저닝된 처리량(PT)이 소진되면 온디맨드로 장애 조치됩니다. `404`는 상류별 모델 가용성이므로 모델을 활성화하지 않은 상류가 모델을 제공하는 이후 상류를 차단하지 않습니다. 요청된 모델을 해소할 수 없는 상류는 네트워크 왕복 없이 건너뜁니다.

이 예제는 프로비저닝된 처리량 Amazon Bedrock 할당을 먼저 라우팅하고, 온디맨드 및 두 번째 계정으로 넘어가며, 마지막 대안으로 Anthropic API로 이동합니다:

```yaml theme={null}
upstreams:
  # 기본: 홈 지역의 프로비저닝된 처리량.
  - name: bedrock-pt
    provider: bedrock
    region: us-east-1
    auth: {}
  # 오버플로: 온디맨드 교차 지역.
  - name: bedrock-od
    provider: bedrock
    region: us-west-2
    auth: {}
  # 다른 계정: 역할 인수(assumed-role) 자격 증명을 통한 별도 Bedrock 할당.
  - name: bedrock-acct2
    provider: bedrock
    region: us-east-1
    auth:
      aws_access_key_id: ${ACCT2_AKID}
      aws_secret_access_key: ${ACCT2_SK}
  # 최종 대안: 직접 Anthropic API.
  - name: anthropic-fallback
    provider: anthropic
    auth:
      api_key: ${ANTHROPIC_API_KEY}

# 상류별 모델 ID는 상류의 `name:`으로 지정됩니다. `name:`이 없는 상류는
# 기본적으로 해당 제공업체 문자열(예: `bedrock`)로 지정됩니다. 모델에 대해
# 나열되지 않은 상류는 건너뛰므로 Opus를 프로비저닝된 처리량으로 라우팅하고
# 나머지는 온디맨드로 유지할 수 있습니다.
models:
  - id: claude-opus-4-8
    label: Claude Opus 4.8
    upstream_model:
      bedrock-pt: arn:aws:bedrock:us-east-1:111111111111:provisioned-model/abcdef
      bedrock-od: us.anthropic.claude-opus-4-8
      bedrock-acct2: us.anthropic.claude-opus-4-8
      anthropic-fallback: claude-opus-4-8
```

| 기법                   | 설정 방법                                                                                                                                                                                                                              |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 서로 다른 지역         | 지역당 하나의 Amazon Bedrock 상류를 두고 각각 고유한 `region:`을 설정합니다. [`auto_include_builtin_models: true`](#models) 설정 시 교차 지역 추론 프로필이 자동으로 라우팅됩니다. 지역 고정 배포의 경우 `models:` 블록을 사용하세요.   |
| 서로 다른 계정         | 계정당 하나의 Amazon Bedrock 상류를 두고 각각 `auth:`에 고유한 자격 증명을 설정합니다. 기본 체인(`auth: {}`)은 파드의 정체성을 사용합니다. 두 번째 계정의 경우 명시적 자격 증명 또는 전달자 토큰을 설정하세요.                         |
| 프로비저닝된 처리량    | 해당 상류 이름의 `models:`에서 모델을 프로비저닝된 처리량 ARN에 매핑합니다. 다른 상류는 온디맨드 ID를 유지하므로 장애 조치 전에 PT 용량이 먼저 소진됩니다.                                                                             |
| VPC / FIPS 엔드포인트  | 상류의 `base_url:`을 VPC 엔드포인트 또는 FIPS 엔드포인트 URL로 설정하세요.                                                                                                                                                             |
| 모델 범위 지정 라우팅  | 모델의 `upstream_model:` 맵에서 상류를 생략하면 해당 모델에 대해 해당 상류가 건너뛰어집니다. 예를 들어 Opus는 프로비저닝된 처리량으로 라우팅하고 Sonnet 및 Haiku는 온디맨드로 라우팅할 수 있습니다.                                   |

클라우드 제공업체 간에 또는 직접 Anthropic API로 장애 조치하면 요청을 통제하는 약관, 지리적 위치 및 기타 조건이 달라집니다.

CLI는 장애 조치로 인해 상류가 거부할 본문 필드가 전송되지 않도록 어떤 상류가 요청을 처리하든 게이트웨이에 동일한 기능 제어를 적용합니다.

## 선택 섹션

### `admin`

선택 사항. Anthropic의 공개 관리자 API를 미러링하는 `/v1/organizations/spend_limits` 및 `/v1/messages`에 대한 개발자별 지출 적용을 활성화합니다. 한도 설정 및 적용 방법은 [지출 한도](/docs/en/claude-apps-gateway-spend-limits)를 참조하세요. 이 섹션에서는 기능을 켜고 조정하는 `gateway.yaml` 키를 다룹니다.

```yaml theme={null}
admin:
  # x-api-key로 전송되는 관리자 엔드포인트용 정적 API 키.
  # id는 audit로그에 admin-key:<id>로 표시되어 각 키를
  # 식별할 수 있습니다. 키 순환을 위해 배열 제공: 새 키 추가,
  # 클라이언트 업데이트, 이전 키 제거.
  write_keys:
    - { id: terraform, key: "${GATEWAY_ADMIN_WRITE_KEY_TF}" }
    - { id: ci,        key: "${GATEWAY_ADMIN_WRITE_KEY_CI}" }
  read_keys:
    - { id: reporting, key: "${GATEWAY_ADMIN_READ_KEY}" }
  # 일반 게이트웨이 JWT를 통해 전체 관리자 권한이 부여된 IdP 그룹 (API 키 없음).
  admin_groups: [platform-finops]
  blocked_message: request an increase at https://go.example.com/claude-limits
```

| 필드                      | 필수 여부 | 설명                                                                                                                                                                                                                                           |
| ------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `write_keys`              | 아니오    | `{id, key}`의 배열. 이들 중 하나와 일치하는 `x-api-key`는 지출 한도를 조회, 설정 및 삭제할 수 있습니다. 키 값은 최소 32자 이상이어야 하며 `id`는 `read_keys` 및 `write_keys` 전반에서 고유해야 합니다.                                        |
| `read_keys`               | 아니오    | `{id, key}`의 배열. 읽기 전용: 한도 목록 조회, ID로 한도 가져오기, [`/effective`](/docs/en/claude-apps-gateway-spend-limits#%2Feffective) 및 [`/audit`](/docs/en/claude-apps-gateway-spend-limits#%2Faudit) 읽기를 포함한 모든 `GET` 엔드포인트.  |
| `admin_groups`            | 아니오    | IdP 그룹 이름. `groups` 클레임에 이들 중 하나가 포함된 게이트웨이 JWT는 읽기 및 쓰기 전체 관리자 접근 권한을 가지며 `oidc:<sub>`로 감사됩니다. 인간 관리자에게 이를 사용하고 머신에는 API 키를 사용하세요.                                      |
| `blocked_message`         | 아니오    | 차단된 개발자에게 표시되는 `429 billing_error`에 그대로 추가됩니다. URL이나 Slack 채널과 같은 전체 안내문을 작성하세요. 설정하지 않으면 에러는 `spend limit reached`입니다.                                                                    |
| `audit_retention_days`    | 아니오    | 기본값 `365`. 더 오래된 `admin_audit` 행이 삭제 정리됩니다.                                                                                                                                                                                    |
| `spend_retention_months`  | 아니오    | 기본값 `13`. 이보다 오래된 `spend` 카운터 행이 삭제 정리됩니다. 기본값은 연간 보고서를 위해 전년 전체 및 현재 부분 월을 유지합니다.                                                                                                            |
| `identity_retention_days` | 아니오    | 기본값 `90`. 각 개발자의 이메일, 표시 이름, 그룹(개인정보)을 보유하는 `principal_emails` 행의 최종 확인 TTL. 자격이 박탈된 정체성이 익명 지출 카운터를 유지하면서 정리되도록 지출 보존 기간보다 의도적으로 짧게 설정됩니다.                 |
| `group_limit_mode`        | 아니오    | `min`(기본값) 또는 `max`. 한도가 있는 여러 그룹에 포함된 개발자의 경우 `min`은 가장 제한적인 한도를 적용하고 `max`는 가장 덜 제한적인 한도를 적용합니다. 적용 로직과 `/effective` 모두에서 사용됩니다.                                      |

### `enforcement`

`enforcement` 블록은 저장소를 사용할 수 없을 때 지출 한도 검사가 작동하는 방식을 제어합니다.

| 필드                   | 필수 여부 | 설명                                                                                                                                                                                                                                                     |
| ---------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fail_closed_on_error` | 아니오    | 기본값 `false`. 지출 적용은 Postgres 장애 발생 시 fail-open 방식으로 작동하므로 추론이 계속 유지됩니다. fail-closed 방식으로 설정하려면 `true`로 지정하세요. 한도를 초과한 개발자는 차단되지만 저장소에 도달할 수 없는 경우 다른 사람도 차단됩니다. [`admin:`](#admin) 블록이 없으면 효과가 없습니다. |

### `models`

`models` 블록은 `/v1/models`에서 제공되고 상류별 모델 ID를 변환하는 데 사용되는 선택적 관리자 제공 모델 목록입니다. 미국 외 Amazon Bedrock 지역, Amazon Bedrock 프로비저닝된 처리량 ARN 및 Microsoft Foundry 배포 이름에 필요합니다.

```yaml theme={null}
auto_include_builtin_models: true   # false: 아래 목록만 노출
models:
  - id: claude-opus-4-8
    label: Claude Opus 4.8
    # description: 이를 표출하는 클라이언트에 표시되는 선택적 텍스트
    upstream_model:
      anthropic: claude-opus-4-8
      bedrock: us.anthropic.claude-opus-4-8   # 또는 추론 프로필 ARN
      foundry: your-opus-deployment-name
```

### `managed`

`managed` 블록은 IdP 그룹 또는 이메일 도메인에 대한 역할 기반 접근 정책을 정의합니다. 정책은 순서대로 평가되며 첫 번째 일치 항목이 선택된 후 아래에 설명된 `match: {}` 전체 적용 베이스에 병합됩니다. 이들은 ETag/304 캐싱과 함께 `GET /managed/settings`에서 사용자별로 제공됩니다.

```yaml theme={null}
managed:
  policies:
    # 특정 그룹을 먼저 배치.
    - match: { groups: [eng-contractors] }
      cli:
        availableModels: [claude-sonnet-4-6]
        permissions: { deny: ["WebFetch", "WebSearch"] }
    # 기본 전체 적용 정책을 마지막에 배치: 인증된 모든 사람과 일치.
    - match: {}
      cli:
        availableModels: [claude-opus-4-8, claude-sonnet-4-6, claude-haiku-4-5]
```

관례적으로 마지막에 나열되는 `match: {}` 전체 적용 항목은 베이스 레이어로 취급됩니다. 다른 모든 정책은 설정하지 않은 모든 키를 이 전체 적용 항목에서 상속하므로 역할별 항목은 조직 기본값과 다른 내용만 나열하면 됩니다. 병합 규칙은 키 유형에 따라 다릅니다:

* **허용 목록(Allow-lists)**: `availableModels` 및 `permissions.allow`. 특정 정책의 목록이 베이스 목록을 완전히 대체합니다.
* **거부 목록 및 훅 배열(Deny-lists and hook arrays)**: `permissions.deny`, `permissions.ask`, `disabledMcpjsonServers`, `deniedMcpServers`, `blockedMarketplaces` 및 모든 `hooks` 이벤트 유형 배열. 이들은 베이스와 정책의 합집합을 취하므로 조직 전체 거부 또는 감사 훅이 역할별 재정의에 의해 실수로 삭제되지 않습니다.
* **레코드 유형 키(Record-typed keys)**: `env`, `modelOverrides` 및 `skillOverrides`. 얕은 병합(shallow-merge)을 수행하므로 역할별 `env` 블록은 설정된 키를 재정의하고 나머지는 베이스에서 상속합니다.

`availableModels`는 `/v1/messages`에서도 서버 측에 적용되므로 거부된 모델은 클라이언트가 전송하는 내용과 상관없이 `400`을 반환합니다.

| 매처 (Matcher)                                      | 동작                                                                                                                              |
| --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `match: {}`                                         | 인증된 모든 사용자와 일치합니다. 이 항목으로 시작하고 나중에 그 위에 그룹 범위 정책을 추가하세요.                                 |
| `match: { groups: [a, b] }`                         | JWT의 `groups` 클레임에 나열된 그룹 중 하나라도 포함된 경우 일치합니다. 대소문자 구분: 그룹은 IdP의 정확한 대소문자와 일치해야 합니다. |
| `match: { email_domain: example.com }`              | JWT의 `email` 클레임에서 마지막 `@` 뒤의 부분과 대소문자 구분 없이 일치합니다. 정책당 하나의 도메인을 수락합니다.                |
| `match: { groups: [a], email_domain: example.com }` | 두 조건이 모두 일치해야 합니다.                                                                                                   |

어떤 정책과도 일치하지 않는 인증된 사용자는 게이트웨이 기본값(카탈로그의 모든 모델이 제공되고 관리형 설정이 적용되지 않음)을 받습니다. 보장된 기본 정책을 원한다면 마지막에 `match: {}` 전체 적용 항목을 추가하세요.

<Note>
  게이트웨이는 자체 사용자 디렉터리를 유지하지 않습니다. 토큰의 `groups` 클레임에서 그룹 멤버십을 읽고 이에 대해 정책을 평가하여 사용자의 IdP 토큰으로 각 요청을 인가합니다. 열거할 명부나 미리 생성할 계정이 없으므로 SCIM이 동기화할 대상이 없어 SCIM 엔드포인트도 존재하지 않습니다.

  사용자 및 그룹 수명주기 관리는 신뢰의 원천(진실의 원천)인 IdP의 기본 SCIM 프로비저닝 또는 전용 정체성 관리 플랫폼에서 실행하세요. 거기서 통제되는 멤버십 및 자격 박탈은 토큰을 통해 게이트웨이에 자동으로 전달됩니다. Claude 계정 자체의 SCIM 프로비저닝을 원하는 경우 이는 [Claude Enterprise](/docs/en/admin-setup) 기능입니다.

  두 가지 전파 주기가 적용됩니다:

  * **정책 내용**: 정책을 수정하고 재배포하면 1시간 이내에 다음 관리형 설정 폴링 시 연결된 클라이언트에 반영됩니다.
  * **그룹 멤버십**: 사용자의 그룹 멤버십을 변경하면 일치하는 정책이 변경됩니다. 이는 다음 세션 재발급(즉, 다음 배경 자동 새로 고침 시, `session.ttl_hours` 상한 적용) 시 적용됩니다.
</Note>

#### `cli`에 들어가는 내용

각 `cli` 값은 완전한 Claude Code `managed-settings.json` 문서로, MDM 또는 `/etc/claude-code/managed-settings.json`을 통해 배포할 동일한 스키마가 여기서는 YAML로 표현됩니다. CLI는 전달된 문서를 사용자 및 프로젝트 설정 위의 관리형 계층에 적용합니다.

게이트웨이는 부팅 시 CLI의 설정 스키마와 비교하여 각 문서를 검증하므로, 인식되지 않는 최상위 키나 형식이 잘못된 값을 가진 인식된 키는 부팅 시 문제 키를 지적하며 실패합니다. 게이트웨이 스키마가 인식하지 못하는 항목을 새 클라이언트가 인식할 수 있으므로 의도적으로 열려 있는 스키마 부분은 임의의 값을 계속 수락합니다. 이러한 열린 키는 `env`, `pluginConfigs` 및 `permissions` 아래에 중첩된 키입니다.

검증이 게이트웨이의 설치된 버전에 번들로 제공되는 스키마를 사용하므로 새 Claude Code 릴리스에서 도입된 최상위 설정 키를 관리형 설정에 추가하려면 게이트웨이를 먼저 업데이트해야 합니다. 조직 전체에 배포하기 전에 하나의 클라이언트에서 새 정책을 테스트해 보세요.

전체 키 참조는 [Claude Code 설정](/docs/en/settings#available-settings)을 참조하세요. 대다수 운영자가 가장 먼저 사용하는 주요 키:

```yaml theme={null}
managed:
  policies:
    - match: {}
      cli:
        # 모델 접근 (서버 측 /v1/messages에서도 적용됨)
        availableModels: [claude-opus-4-8, claude-sonnet-4-6, claude-haiku-4-5]

        # 권한 정책
        permissions:
          deny:
            - "WebFetch"
            - "Read(./.env)"
            - "Read(./secrets/**)"
          disableBypassPermissionsMode: disable   # --dangerously-skip-permissions 차단
        allowManagedPermissionRulesOnly: true     # 사용자/프로젝트 권한 규칙 무시

        # CLI 프로세스로 푸시되는 환경 변수. DISABLE_UPDATES는
        # 백그라운드 및 수동 업데이트를 차단하고 DISABLE_AUTOUPDATER는
        # 백그라운드 업데이트만 중지합니다.
        env:
          DISABLE_UPDATES: "1"                    # 자체 배포 방식으로 버전 고정

        # 조직 전체 훅. 훅 명령은 게이트웨이가 아닌 개발자
        # 머신에서 실행되므로 경로가 정책의 모든 클라이언트 OS에 존재해야 합니다.
        hooks:
          PostToolUse:
            - matcher: "Edit|Write"
              hooks:
                - { type: command, command: /usr/local/bin/audit-edit.sh }
```

| 키                                         | 적용 위치     | 효과                                                                                                                                                                                                                                    |
| ------------------------------------------ | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `availableModels`                          | 게이트웨이 + CLI | 모델 허용 목록. `/v1/messages`에서도 검사되므로 수정된 클라이언트가 우회할 수 없습니다.                                                                                                                                                 |
| `permissions.allow` / `.deny`              | CLI           | 도구 및 명령 규칙. [권한](/docs/en/permissions) 참조.                                                                                                                                                                                   |
| `permissions.disableBypassPermissionsMode` | CLI           | 권한 프롬프트를 건너뛰는 모드인 [`bypassPermissions`](/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode) 및 `--dangerously-skip-permissions` 플래그를 차단하려면 `disable`로 설정하세요.                          |
| `allowManagedPermissionRulesOnly`          | CLI           | `true`일 때 사용자 및 프로젝트 권한 규칙이 무시되며 이 문서의 규칙만 적용됩니다.                                                                                                                                                        |
| `env`                                      | CLI           | CLI 프로세스로 병합되는 환경 변수. 텔레메트리, 자동 업데이트 및 모델 이름 재정의에 사용합니다.                                                                                                                                          |
| `hooks`                                    | CLI           | 조직 전체 [훅(hooks)](/docs/en/hooks)                                                                                                                                                                                                   |

이러한 설정이 네트워크를 통해 전달되므로 CLI는 쉘 명령을 실행하거나 트래픽 목적지를 변경할 수 있는 항목을 적용하기 전에 각 개발자에게 일회성 보안 승인 대화 상자를 보여줍니다. 대화 상자가 다루는 범위:

* `hooks`
* CLI의 내장 안전 목록에 없는 `env` 변수
* `apiKeyHelper` 및 `statusLine`과 같은 셸 실행 설정
* 관리형 CLAUDE.md 내용

안전 목록에 따라 승인 없이 적용되는 `env` 변수가 결정됩니다:

* **안전 목록 포함**: 자동 업데이트 및 모델 이름 변수
* **안전 목록 제외**: 프록시 변수, 베이스 URL 변수 및 `OTEL_EXPORTER_OTLP_ENDPOINT`

게이트웨이의 [텔레메트리](#telemetry) 설정이 `OTEL_EXPORTER_OTLP_ENDPOINT`를 푸시하므로 `telemetry.forward_to`를 설정하면 상호작용 가능한 각 클라이언트에서 대화 상자가 트리거됩니다. 대화 상자는 개발자로부터 조직을 보호하는 것이 아니라 침해되었거나 적대적인 게이트웨이로부터 개발자의 머신을 보호합니다.

`-p` 플래그를 사용한 비대화형 실행은 대화 상자를 표시할 수 없습니다. 비대화형 실행은 해당 실행에만 전송된 설정을 적용하고 이를 승인된 것으로 기록하지 않으므로 개발자의 다음 대화형 세션에서 대화 상자가 계속 표시됩니다. v2.1.207 이전에는 비대화형 실행이 설정을 승인된 것으로 저장하여 이후 대화형 세션에서 대화 상자가 나타나지 않았습니다.

개발자가 거절하면 Claude Code는 정책을 적용하지 않고 종료됩니다. 따라서 새 훅이나 비안전 환경 변수를 광범위한 정책으로 푸시하면 해당되는 모든 개발자의 다음 시작 시 승인 프롬프트가 발생합니다.

`cli` 키는 이전 릴리스에서 `settings`로 명명되었습니다. 해당 표기도 별칭으로 계속 수락되지만 새 배포에서는 `cli`를 사용해야 합니다.

#### 다른 관리형 소스와의 우선순위

기기에 로컬 `managed-settings.json` 또는 MDM이 전달한 정책이 있는 경우 관리형 소스끼리는 병합되지 않습니다. 가장 높은 우선순위 소스가 모든 정책 설정을 제공하며, 순위는 가장 높은 우선순위부터 다음과 같습니다:

1. [정책 헬퍼](/docs/en/settings#compute-managed-settings-with-a-policy-helper)
2. 게이트웨이가 전달한 설정
3. MDM (Windows의 HKLM 레지스트리 또는 macOS의 plist)
4. `managed-settings.json` 파일
5. HKCU 레지스트리 (Windows 전용)

임베딩 호스트는 SDK `managedSettings` 옵션을 통해 정책을 제공할 수 있습니다. 적용 여부는 머신의 관리형 설정에 따라 다릅니다:

* 관리자가 배포한 관리형 소스가 있는 머신에서는 가장 높은 우선순위 소스가 [`parentSettingsBehavior: "merge"`](/docs/en/settings#available-settings)로 옵트인하지 않는 한 무시됩니다.
* [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)가 구성되어 있는 동안에는 절대 병합되지 않습니다.
* 병합될 때 제한 전용 허용 목록을 통과합니다. [부모 설정 제한](/docs/en/claude-apps-gateway#restrict-parent-settings)에서 `allowManaged*Only` 잠금 없이도 계속 적용되는 허용 방향 설정을 나열합니다.

사용자 수정 가능 HKCU 계층 위의 임의 관리자 소스가 설정한 경우 다음 키는 나머지 정책을 제공하는 소스와 상관없이 준수됩니다. [`policyHelper`](/docs/en/settings#compute-managed-settings-with-a-policy-helper)가 구성되면 해당 출력만이 이러한 검사가 읽는 유일한 소스입니다:

* `sandbox.network.allowManagedDomainsOnly` 및 `sandbox.filesystem.allowManagedReadPathsOnly`: 잠겨 있을 때 해당 허용 목록이 소스 전반에서 합집합으로 처리됩니다.
* [`allowAllClaudeAiMcps`](/docs/en/settings#available-settings): claude.ai MCP 서버 허용 목록에 대한 허용 전용 재정의
* `sandbox.bwrapPath` 및 `sandbox.socatPath`: [샌드박스](/docs/en/sandboxing) 헬퍼 바이너리의 파일 시스템 경로
* [`forceRemoteSettingsRefresh`](/docs/en/server-managed-settings): 원격 관리형 설정을 새로 가져올 때까지 시작을 차단하므로, 이를 설정한 MDM 또는 파일 정책은 해당 키가 누락된 캐시된 원격 페이로드가 가장 높은 우선순위 소스이더라도 준수됩니다.

`disableBypassPermissionsMode`를 포함한 다른 모든 키는 가장 높은 우선순위 소스에서만 가져옵니다. 두 가지 [부모 설정](/docs/en/claude-apps-gateway#restrict-parent-settings) 검사는 모든 관리자 소스를 읽습니다:

* 임의의 관리자 소스가 `allowManagedPermissionRulesOnly`를 설정하면 Claude Code는 부모가 제공한 권한 허용 규칙 및 `additionalDirectories`를 삭제합니다. 개발자 자체 규칙에 대한 이 키의 영향은 여전히 가장 높은 우선순위 소스를 따릅니다.
* 임의의 관리자 소스에 있는 `forceLoginOrgUUID` 또는 `allowedMcpServers` 값은 부모가 제공한 값을 차단합니다. 적용되는 값은 여전히 가장 높은 우선순위 소스에서 가져옵니다.

동일한 규칙은 [설정 우선순위](/docs/en/settings#settings-precedence)를 참조하세요.

게이트웨이 정책은 비대화형 `claude -p` 실행 및 Agent SDK가 생성한 세션을 포함하여 머신의 모든 Claude Code 호출에 적용됩니다. 시작 시 게이트웨이에 도달할 수 없는 경우 로그인된 세션은 정책 없이 실행되는 대신 에러와 함께 종료됩니다.

<Warning>
  정책의 `cli` 블록 내부의 `mcpServers`는 게이트웨이 부팅 시 거부됩니다. 그룹별 MCP 배포는 지원되지 않습니다. 각 기기의 파일 기반 `managed-mcp.json`을 통해 MCP 서버를 배포하거나 개발자가 로컬에 추가하도록 하세요.
</Warning>

### `telemetry`

CLI는 OpenTelemetry Protocol (OTLP) over HTTP 메트릭, 로그 및 활성화된 경우 트레이스를 게이트웨이로 전송하고, 게이트웨이는 설정된 각 대상으로 이를 있는 그대로 전달합니다. CLI가 내보내는 메트릭 및 이벤트는 [사용량 모니터링](/docs/en/monitoring-usage)을 참조하세요.

CLI는 게이트웨이가 발급한 JWT에서 읽은 인증된 사용자의 정체성(`user.id`, `user.email` 및 `user.groups` 속성)을 각 내보내기에 스탬프로 기록합니다. 따라서 개발자 측 설정 없이 개발자별 비용 및 사용량 귀속이 작동합니다.

```yaml theme={null}
telemetry:
  forward_to:
    - url: https://otel-collector.internal.example.com
      headers:
        Authorization: ${OTLP_TOKEN}
      # 시그널별 선택. 기본값: 메트릭만.
      metrics: true
      logs: false
      traces: false
    - url: https://api.datadoghq.com/api/v2/otlp
      headers:
        DD-API-KEY: ${DD_API_KEY}
```

<Warning>
  각 대상은 `metrics`, `logs` 및 `traces`를 독립적으로 선택하며 기본값은 메트릭만 해당됩니다. 시그널은 민감도에서 차이가 있습니다:

  * **메트릭**: 토큰 수, 요청 수, 지연 시간과 같은 집계 카운터
  * **로그 및 트레이스**: 개발자 머신에서 Claude Code가 수행하는 모든 작업을 다루는 전체 bash 명령, 도구 입력 및 파일 경로를 전달할 수 있음

  해당 데이터가 요구하는 접근 제어 및 보존 정책이 있는 대상에만 로그 및 트레이스를 활성화하세요.
</Warning>

CLI에서 텔레메트리는 기본적으로 꺼져 있습니다. `telemetry.forward_to`를 `listen.public_url`과 함께 구성하면 켜집니다. 게이트웨이는 `/managed/settings`를 통해 모든 연결된 클라이언트에 5개의 환경 변수를 푸시합니다:

* `CLAUDE_CODE_ENABLE_TELEMETRY=1`
* `OTEL_METRICS_EXPORTER=otlp`
* `OTEL_LOGS_EXPORTER=otlp`
* `OTEL_TRACES_EXPORTER=otlp`
* `OTEL_EXPORTER_OTLP_ENDPOINT=<public_url>`

푸시되는 엔드포인트는 공개 URL에서 작성되므로 메트릭 및 로그에 개발자나 정책의 OTEL 설정이 필요하지 않습니다. 푸시되는 설정은 관리형 계층에 적용되어 개발자가 로컬에 설정한 `OTEL_*` 변수를 재정의합니다.

[트레이스](/docs/en/monitoring-usage#traces-beta)에는 각 클라이언트에 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`이 추가로 필요합니다. 게이트웨이는 해당 변수를 푸시하지 않으므로 관리형 정책의 `env` 블록을 통해 설정하세요. CLI의 안전 목록에 없으므로 정책을 통해 이를 전달하면 푸시된 OTLP 엔드포인트가 이미 트리거하는 것과 동일한 [보안 승인 대화 상자](#managed)가 발생합니다.

protobuf 및 JSON OTLP 인코딩이 모두 중계되며 OpenTelemetry 호환 백엔드라면 모두 대상으로 작동합니다.

### HTTP 조정

네 가지 선택적 최상위 블록인 `access_control`, `limits`, `timeouts` 및 `rate_limits`는 HTTP 표면을 조정합니다. 기본값은 대부분의 배포에 적합합니다.

| 블록              | 키                                             | 기본값   | 설명                                                                                                                                                                                                                                                                                                                         |
| ----------------- | ---------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `access_control`  | `allow_cidrs` / `deny_cidrs`                   | 비어 있음| `trusted_proxies` 해소 후 클라이언트 주소에 의한 수신 IP 허용/거부. `deny_cidrs`가 먼저 검사되어, 일치하는 클라이언트는 `allow_cidrs`와 일치하더라도 거부됩니다. `allow_cidrs`가 비어 있지 않으면 게이트웨이는 기본 거부(default-deny) 방식입니다. `/healthz` 및 `/readyz`는 `allow_cidrs`에서 제외됩니다.                    |
| `limits`          | `max_request_bytes`                            | 32 MiB   | 최대 수신 요청 본문. 크기를 초과하는 요청은 본문이 버퍼링되기 전에 `413`을 받습니다. 대용량 파일이나 이미지 요청의 경우 이 값을 올리세요.                                                                                                                                                                                  |
| `limits`          | `max_request_header_bytes`                     | 미설정   | 설정 시 크기를 초과하는 헤더는 `431`을 반환합니다.                                                                                                                                                                                                                                                                           |
| `limits`          | `max_url_length`                               | 미설정   | 설정 시 너무 긴 URL은 `414`를 반환합니다.                                                                                                                                                                                                                                                                                    |
| `timeouts`        | `upstream_ttfb_ms`                             | 120000   | 상류의 응답 헤더(첫 바이트 도달 시간)에 대한 최대 대기 시간(ms). 그 후 응답 본문은 제한 없이 스트리밍됩니다. 직접 Anthropic 상류 경로에 적용되며, 다른 모든 제공업체는 자체 제공업체 SDK의 타임아웃에 의해 제한됩니다.                                                                                                    |
| `rate_limits`     | `device_authorization.max` / `.window_seconds` | 30 / 600 | 비인증 디바이스 인가 엔드포인트에 대한 IP별 속도 제한. 공유 이그레스 IP 또는 NAT 뒤의 대규모 조직인 경우 이 값을 올리세요. 이 제한은 `/v1/messages` 추론이 아닌 디바이스 인가 로그인 흐름에만 적용됩니다. [사용자 코드 무차별 대입 방어력](/docs/en/claude-apps-gateway-deploy#user-code-brute-force-resistance)을 참조하세요. |
| `rate_limits`     | `device_verify.max` / `.window_seconds`        | 10 / 600 | `/device`에서 `user_code` 제출에 대한 IP별 속도 제한                                                                                                                                                                                                                                                                         |

## 전체 예제

이 전체 참조 설정은 모든 핵심 섹션을 종합하여 다룹니다. [HTTP 조정 블록](#http-조정)은 기본값을 유지합니다. 복사하여 필요 없는 항목을 삭제하고 본인의 값으로 채워 넣으세요. [퀵스타트](/docs/en/claude-apps-gateway#quickstart)의 설정은 이 예제의 최소화 버전입니다.

```yaml gateway.yaml theme={null}
# 실행 명령어:
#   claude gateway --config gateway.yaml
#
# 운영 로그 상세도는 CLAUDE_GATEWAY_LOG_LEVEL 환경 변수로 제어됩니다.
# (info | warn | error; 기본값 info).
# 항상 내보내지는 감사 이벤트에는 영향을 주지 않습니다.

listen:
  host: 0.0.0.0
  port: 8080
  public_url: https://claude-gateway.internal.example.com
  # TLS 종료 인그레스 뒤에서 실행할 때는 tls 블록을 생략하세요.
  # tls:
  #   cert: /certs/gateway.crt
  #   key: /certs/gateway.key
  # trusted_proxies:
  #   - 10.0.0.0/8

oidc:
  issuer: https://example.okta.com
  client_id: 0oa1example2
  client_secret: ${OIDC_CLIENT_SECRET}
  allowed_email_domains:
    - example.com
  # Okta 조직 서버가 발급자인 경우 id_token에서 이메일 및 그룹을 생략할 수 있으므로 필수입니다.
  # 게이트웨이가 /userinfo에서 이를 채웁니다.
  userinfo_fallback: true
  # allowed_groups: [claude-code-users]
  # Okta는 `groups` 스코프가 요청되고 앱의 그룹 클레임 필터가 허용할 때만 그룹을 내보냅니다.
  # 아래의 contractors 정책이 그룹과 일치하므로 스코프가 여기서 요청됩니다.
  scopes: [openid, profile, email, offline_access, groups]
  # extra_auth_params: { access_type: offline, prompt: consent }  # Google
  # groups_claim: groups          # Entra 앱 역할: `roles` 사용
  # email_claim: email

session:
  jwt_secret: ${GATEWAY_JWT_SECRET}   # openssl rand -base64 32
  # ttl_hours: 1

store:
  postgres_url: ${GATEWAY_POSTGRES_URL}
  # max_connections: 5

# /v1/organizations/spend_limits (Anthropic Admin API 미러링) 및
# /v1/messages에 대한 개발자별 지출 적용을 활성화합니다. 비활성화하려면 생략하세요.
# 한도 자체는 여기가 아닌 관리자 API를 통해 설정됩니다.
# admin:
#   write_keys:
#     - { id: terraform, key: "${GATEWAY_ADMIN_WRITE_KEY_TF}" }
#   read_keys:
#     - { id: reporting, key: "${GATEWAY_ADMIN_READ_KEY}" }
#   admin_groups: [platform-finops]
#   blocked_message: request an increase at https://go.example.com/claude-limits
#   # audit_retention_days: 365
#   # spend_retention_months: 13
#   # identity_retention_days: 90
#   # group_limit_mode: min

# enforcement:
#   fail_closed_on_error: false

upstreams:
  - provider: anthropic
    auth:
      api_key: ${ANTHROPIC_API_KEY}

  # - provider: bedrock
  #   region: us-east-1
  #   auth: {}

  # - provider: anthropicAws
  #   region: us-east-1
  #   workspace_id: wrkspc_...
  #   auth:
  #     api_key: ${ANTHROPIC_AWS_API_KEY}

  # - provider: vertex
  #   region: us-east5
  #   project_id: example-prod
  #   auth: {}

  # - provider: foundry
  #   resource: example-foundry
  #   auth: { use_azure_ad: true }

auto_include_builtin_models: true
models:
  - id: claude-opus-4-8
    label: Claude Opus 4.8
    upstream_model:
      anthropic: claude-opus-4-8
      # bedrock: us.anthropic.claude-opus-4-8
      # anthropicAws: claude-opus-4-8
      # vertex: claude-opus-4-8
      # foundry: <your-opus-deployment-name>
  - id: claude-sonnet-4-6
    label: Claude Sonnet 4.6
    upstream_model:
      anthropic: claude-sonnet-4-6
  - id: claude-haiku-4-5
    label: Claude Haiku 4.5
    upstream_model:
      anthropic: claude-haiku-4-5

managed:
  policies:
    - match: { groups: [contractors] }
      cli:
        availableModels: [claude-haiku-4-5]
        # contractors가 기본값에서 400을 받지 않도록 Default 선택기 옵션을
        # 티어 기본값 대신 availableModels로 제한합니다.
        enforceAvailableModels: true
        # allow는 이들 도구를 자동 승인합니다. 나머지 도구를 차단하지는 않습니다.
        # 도구를 제한하려면 deny 규칙을 추가하세요.
        permissions: { allow: [Read, Grep] }
    - match: {}
      cli:
        availableModels: [claude-opus-4-8, claude-sonnet-4-6, claude-haiku-4-5]
        permissions:
          allow: [Read, Grep, Bash, Edit]
          deny: ["WebFetch"]
        env: { HTTP_PROXY: http://proxy.example.com:8080 }

telemetry:
  forward_to:
    - url: https://otel.internal.example.com:4318
      headers:
        Authorization: Bearer ${OTEL_TOKEN}
```

## 클라이언트 측 관리형 설정

위의 모든 내용은 게이트웨이 서버를 설정하는 항목입니다. 개발자 머신을 게이트웨이로 가리키는 작업은 Claude Code의 [관리형 설정](/docs/en/settings#settings-files)을 통해 각 기기에서 별도로 구성됩니다. 로그인 키는 클라이언트에 게이트웨이의 위치를 알리는 키이므로 게이트웨이가 스스로 이를 푸시할 수 없습니다.

CLI의 경우 OS별 `managed-settings.json`에 이러한 키를 설정하세요. 두 개의 로그인 키는 각 개발자의 `/login`을 해당 게이트웨이로 라우팅합니다:

```json theme={null}
{
  "forceLoginMethod": "gateway",
  "forceLoginGatewayUrl": "https://claude-gateway.internal.example.com",
  "parentSettingsBehavior": "merge"
}
```

`parentSettingsBehavior: "merge"`는 임베디드 Claude Code 세션에 대한 Claude Desktop의 정책 전달이 작동하도록 유지합니다. 메커니즘과 옵트인 위치에 대한 설명은 [Claude Desktop 세션에 정책 전달](/docs/en/claude-apps-gateway#deliver-policy-to-claude-desktop-sessions)을 참조하세요.

일반적으로 MDM 플랫폼을 통해 각 기기에 `managed-settings.json` 파일을 배포하세요. 파일 경로는 플랫폼에 따라 다릅니다:

| 플랫폼        | 경로                                                                                                                          |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| macOS         | `/Library/Application Support/ClaudeCode/managed-settings.json` 또는 `com.anthropic.claudecode` 관리형 환경설정 도메인      |
| Linux 및 WSL  | `/etc/claude-code/managed-settings.json`                                                                                      |
| Windows       | `C:\Program Files\ClaudeCode\managed-settings.json` 또는 HKLM 레지스트리를 통한 그룹 정책                                     |

Windows의 레지스트리 정책이나 macOS의 관리형 환경설정 plist는 [위의 예외 키 및 교차 소스 검사](#precedence-with-other-managed-sources)를 제외하고는 `managed-settings.json` 파일과 병합되지 않고 이를 대체합니다. 이 스니펫의 세 키는 모두 가장 높은 우선순위 소스 규칙을 따르므로 그룹 정책 또는 구성 프로필을 통해 정책을 전달하는 플릿은 세 키를 해당 메커니즘에 모두 배치해야 합니다.

`forceLoginGatewayUrl` 및 `forceLoginMethod`의 `"gateway"` 값은 관리자가 제어하는 관리형 계층에서만 인정됩니다. 개발자가 자신의 `~/.claude/settings.json`에 이를 설정해도 아무런 효과가 없습니다.

## 관련 항목

* [Claude apps gateway 개요](/docs/en/claude-apps-gateway): 퀵스타트 및 개발자 연결
* [배포 가이드](/docs/en/claude-apps-gateway-deploy): IdP 설정, 컨테이너 이미지, Kubernetes 및 Cloud Run, 운영 방법
* [지출 한도](/docs/en/claude-apps-gateway-spend-limits): 개발자별 한도 및 관리자 API
