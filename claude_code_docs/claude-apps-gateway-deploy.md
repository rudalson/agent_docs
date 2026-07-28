> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude apps gateway 배포 및 운영

> IdP에 게이트웨이를 등록하고, 컨테이너를 빌드하며, Kubernetes 또는 Cloud Run에 배포하고, 헬스 체크, 비밀 값 순환, 업그레이드 및 보안 등 일상적인 운영을 수행하세요.

이 페이지에서는 [Claude apps gateway](/docs/en/claude-apps-gateway) 운영 측면을 다룹니다: 정체성 제공업체(IdP)에 OAuth 클라이언트 등록, 게이트웨이를 컨테이너로 배포 및 일상 운영. 게이트웨이가 부팅 시 읽는 `gateway.yaml` 파일의 모든 옵션에 대해서는 [설정 참조](/docs/en/claude-apps-gateway-config)를 참조하세요.

프로덕션 배포는 순서대로 4단계를 따르며 아래 섹션이 이에 대응합니다. 처음 두 단계는 선택을 내리는 과정이고, 나머지 두 단계는 운영 중 참고할 참조 자료입니다.

1. [정체성 제공업체 설정](#정체성-제공업체-설정): OAuth 클라이언트를 등록하고 Okta, Entra, Google에 대한 IdP별 참고사항 확인
2. [게이트웨이 배포](#게이트웨이-배포): 고정 버전 컨테이너 이미지를 빌드하고 Kubernetes, Cloud Run 또는 자체 플랫폼에서 실행. 이 섹션은 비용, 우회, 다중 게이트웨이 및 서버리스 관련 결정사항도 다룹니다.
3. [운영 설정](#운영): 로그, 헬스 프로브, 장애 시 동작, 비밀 값 순환 및 업그레이드. 모니터링 및 런북(runbook) 구성 시 참조
4. [보안 상태 검토](#보안): 데이터 흐름 경로, 위협 모델 및 준수 답변. 보안 검토 시 참조

진행 과정 중 로그인이나 부팅이 실패하면 표시되는 오류에 맞게 작성된 [문제 해결](#문제-해결)로 바로 이동하세요.

<Note>
  **사설 네트워크에 배포하세요.** Claude Code는 주소가 사설인 게이트웨이에만 연결됩니다. 신뢰할 수 있는 게이트웨이가 개발자 머신에서 명령을 실행하는 설정을 푸시할 수 있으므로 이는 보안 보호조치입니다. 배포하는 게이트웨이를 내부 로드 밸런서나 VPN 뒤에 두고 사설 IP로만 해소되는 호스트 이름을 부여하세요.

  Anthropic이 직접 운영하는 공개 게이트웨이 엔드포인트는 예외입니다: `/login`은 `https://`를 통해 이를 수용합니다. 이들은 Anthropic 자체가 운영하는 소수의 고정된 게이트웨이 세트입니다. 선택하거나 구성할 수 있는 배포 옵션이 아닙니다. 이 목록은 Claude Code에 컴파일되어 들어가므로 어떠한 설정으로도 해당 목록에 호스트 이름을 추가할 수 없으며 직접 호스팅하는 게이트웨이는 예외를 받을 수 없습니다. {/* min-version: 2.1.206 */}v2.1.206 이전에는 `/login`이 다른 공개 주소와 마찬가지로 이러한 엔드포인트를 거부했습니다.
</Note>

## 정체성 제공업체 설정

단일 리다이렉트 URI `https://<gateway>/oauth/callback`을 사용하여 기밀 OAuth/OpenID Connect (OIDC) 웹 애플리케이션을 등록하고 게이트웨이 접근 권한을 가져야 하는 사용자 또는 그룹에 할당하세요.

Okta, Microsoft Entra ID, Google Workspace, Keycloak, Dex, PingFederate 등 모든 OIDC 준수 IdP가 작동합니다. IdP는 세 가지 요구 사항을 충족해야 합니다:

* 프로덕션 환경에서는 HTTPS를 통해 `/.well-known/openid-configuration`을 제공해야 합니다. 게이트웨이는 [`http://` 발급자](/docs/en/claude-apps-gateway-config#oidc)를 수락하며, 루프백 발급자의 경우 추가로 `CLAUDE_GATEWAY_ALLOW_LOOPBACK=1`이 필요합니다.
* 인가 코드 흐름(authorization-code flow)을 지원합니다. PKCE (Proof Key for Code Exchange)는 기본적으로 켜져 있습니다. 이를 지원하지 않는 IdP의 경우 `oidc.use_pkce: false`로 비활성화하세요.
* id_token에 `email` 및 옵션으로 `groups`를 반환하거나 `oidc.userinfo_fallback: true`를 설정하여 userinfo 엔드포인트에서 이를 제공해야 합니다.

사설 PKI의 경우 `oidc.ca_cert_pem`을 설정하세요.

일부 제공업체는 이메일 및 그룹 클레임을 다르게 처리합니다:

* **Okta**: `https://example.okta.com`에 있는 조직 인가 서버는 `email` 및 `groups`가 생략된 경량 id_token을 반환하므로 이를 `issuer`로 사용할 때마다 `oidc.userinfo_fallback: true`를 설정하세요. id_token에 `email` 및 옵션으로 `groups`를 포함하는 `https://example.okta.com/oauth2/default`와 같은 커스텀 인가 서버는 이를 직접 내보내므로 폴백이 필요하지 않습니다. Okta는 `oidc.scopes`에서 `groups` 스코프가 요청되고 앱의 그룹 클레임 필터가 이를 허용할 때만 `groups`를 내보냅니다. `userinfo_fallback`은 IdP에 요청되지 않은 클레임을 채울 수 없습니다.
* **Microsoft Entra ID**: `issuer` = `https://login.microsoftonline.com/<tenant-id>/v2.0`. Entra는 이름 대신 그룹 개체 ID(GUID)를 내보내므로 `managed.policies.match.groups`에서 GUID를 사용하거나 사람이 읽을 수 있는 이름에 앱 역할을 사용하세요. 테넌트가 `groups` 대신 `roles` 아래에 역할을 내보내는 경우 `oidc.groups_claim: roles`를 설정하세요.
* **Google Workspace**: `issuer` = `https://accounts.google.com`. Google의 id_token은 그룹을 전달하지 않습니다. Google을 IdP로 사용하여 그룹 기반 `allowed_groups` 또는 `managed.policies`를 사용하려면 [`oidc.google_groups`](/docs/en/claude-apps-gateway-config#oidc)를 구성하세요. 이는 도메인 전체 위임 권한이 있는 서비스 계정을 사용하여 Admin SDK Directory API를 통해 각 사용자의 그룹을 조회합니다. 구성하지 않는 경우 멤버십 제어에는 `oidc.allowed_email_domains`를 사용하고 정책 할당에는 `managed.policies.match.email_domain`을 사용하세요. Google은 표준 `offline_access` 스코프도 무시합니다. 리프레시 토큰의 경우 `oidc.scopes: [openid, profile, email]` 및 `oidc.extra_auth_params: { access_type: offline, prompt: consent }`를 설정하세요.

위에 언급되지 않은 정체성 제공업체에 대한 지원은 [문제 해결](#문제-해결)을 참조하세요.

<Warning>
  리프레시 토큰을 사용하면 게이트웨이가 개발자를 브라우저로 다시 보내지 않고 배경에서 세션을 자동으로 갱신할 수 있습니다. 또한 IdP에서 사용자를 비활성화하면 다음 새로 고침이 실패하고 세션이 `ttl_hours` 이내에 종료되므로 리프레시 토큰은 자격 박탈(deprovisioning)에도 관여합니다. 게이트웨이는 기본적으로 리프레시 토큰을 받기 위해 `offline_access`를 요청합니다. IdP에서 오프라인 접근에 대한 명시적 동의가 필요한 경우 해당 OAuth 클라이언트가 이를 허용하도록 구성하세요.

  IdP가 리프레시 토큰을 전혀 발급할 수 없는 경우에도 게이트웨이는 작동하지만 자동 갱신이 없으므로 세션이 만료될 때 개발자가 브라우저 로그인을 다시 실행하게 됩니다. 매시간 이런 일이 발생하지 않도록 하려면 [`session.ttl_hours`](/docs/en/claude-apps-gateway-config#session)를 `8` 또는 `12`로 올리세요. 이에 따른 트레이드오프는 자격 박탈 지연 시간입니다. 리프레시 토큰이 없으면 비활성화된 사용자가 더 긴 TTL이 지날 때까지 접근 권한을 유지하게 되기 때문입니다.
</Warning>

## 배포

게이트웨이는 단일 Linux 바이너리입니다. 레프리카는 상태가 없고(stateless) Postgres가 공유 조율 계층 역할을 하므로 수평 수축/확장(horizontal scaling)이 용이합니다. 사용자 환경에서 상태 없는 서비스를 실행하는 방식으로 실행하세요. 이 섹션의 나머지 부분에서는 이미지에 필요한 항목과 Kubernetes 및 Cloud Run에 대한 짧은 참고 사항을 다룹니다.

게이트웨이는 조직의 상류 자격 증명을 보관하고 추론을 위한 단일 이그레스 지점 역할을 하므로 네트워크 내부에서 실행되도록 설계되었습니다. 개발자와 IdP가 HTTPS를 통해 도달할 수 있는 곳이라면 어디서나 실행될 수 있습니다. 프로덕션 자격 증명을 보관하는 다른 서비스와 동일하게 취급하세요.

몇 가지 결정 사항이 어디서 실행할지 외에 배포 형식을 결정합니다:

* **비용**: 게이트웨이에 대한 별도의 라이선스나 사용자당 비용은 없습니다. `claude` 바이너리의 일부입니다. 기존 클라우드 또는 Anthropic 약정을 통해 추론 비용을 지불하고 컨테이너 및 텔레메트리 수집기를 위한 컴퓨팅 비용만 부담합니다.
* **우회**: 게이트웨이는 모델로 향하는 유일한 경로가 게이트웨이를 통과하도록 강제하지 않습니다. 자체 자격 증명을 가진 개발자는 제공업체를 직접 호출할 수 있으므로 해당 경로를 닫는 것은 네트워크 정책상의 결정사항입니다(예: 게이트웨이를 제외한 다른 곳에서 `api.anthropic.com`으로 향하는 이그레스 차단). 해당 이그레스를 차단하면 개발자의 머신에서 `api.anthropic.com`을 호출하는 [WebFetch 도메인 안전 검사](/docs/en/data-usage#webfetch-domain-safety-check)도 중단되므로 이를 비활성화하려면 관리형 정책에서 `skipWebFetchPreflight: true`를 설정하세요.
* **다중 게이트웨이**: 각 게이트웨이는 자체 설정을 가진 별개의 배포체입니다. CLI는 게이트웨이 호스트 이름별로 신뢰 지문과 자격 증명을 저장하므로 충돌 없이 서로 다른 팀이 서로 다른 게이트웨이에 연결할 수 있습니다. 여러 OIDC 발급자를 제공하려면 별도의 인스턴스를 실행하세요.
* **서버리스**: Cloud Run이 작동합니다. 콜드 OIDC 디스커버리를 방지하기 위해 `min-instances: 1`을 설정하세요. Lambda 및 Cloud Functions는 게이트웨이가 상시 실행되는 HTTP 서버이므로 작동하지 않습니다.

여기서 제공하는 모든 프로덕션 토폴로지는 일반 HTTP 레프리카 앞단에 L7 프록시(Ingress, Cloud Run 프런트엔드 또는 ALB 등)를 둡니다. 게이트웨이가 `X-Forwarded-For`에서 클라이언트 IP를 읽을 수 있도록 [`listen.trusted_proxies`](/docs/en/claude-apps-gateway-config#listen)를 프록시의 소스 범위로 설정하세요. 게이트웨이는 TCP 피어를 신뢰할 때만 헤더를 허용합니다. [Google Cloud 예제](/docs/en/claude-apps-gateway-on-gcp)에 토폴로지별 구체적인 값이 나와 있습니다. 신뢰할 수 있는 프록시가 없으면 모든 요청이 프록시의 IP에서 오는 것으로 보여 IP별 속도 제한이 하나의 공유 버킷으로 축소되고 감사 이벤트에 프록시 IP가 기록됩니다.

### 컨테이너 이미지

표준 Claude Code 릴리스의 네이티브 `claude` 바이너리를 중심으로 자체 이미지를 빌드하세요:

1. 이미지 아키텍처에 맞는 Linux 빌드를 고정 릴리스에서 다운로드하세요. 다운로드 URL은 [특정 버전 설치](/docs/en/setup#install-a-specific-version)를 참조하세요.
2. [바이너리 무결성 및 코드 서명](/docs/en/setup#binary-integrity-and-code-signing)에 설명된 대로 릴리스의 GPG 서명된 `manifest.json`과 비교하여 검증하세요.
3. 이를 빌드 컨텍스트로 복사하세요.

빌드 환경에서 릴리스 호스트에 도달할 수 없는 경우 릴리스를 내부 레지스트리로 미러링하고 플릿이 실행할 버전을 고정하세요.

바이너리 외에 이미지에 필요한 항목:

* **glibc 기반 이미지**: glibc 빌드의 유일한 동적 의존성은 glibc 라이브러리입니다. musl 기반 이미지에는 `linux-x64-musl` 또는 `linux-arm64-musl` 빌드와 추가 패키지가 필요합니다. [Alpine Linux 설정](/docs/en/setup#alpine-linux-and-musl-based-distributions)을 참조하세요.
* **쓰기 가능한 상태 디렉터리**: 게이트웨이는 임의의 사용자로 실행되지만 최소 이미지에는 쓰기 가능한 홈이 없습니다. `CLAUDE_CONFIG_DIR`을 `/tmp/.claude`와 같이 쓰기 가능한 경로로 설정하세요.
* **컨테이너 명령어**: `claude gateway --config /etc/claude/gateway.yaml`로, 설정 파일은 읽기 전용으로 마운트하고 비밀 값은 환경 변수로 제공합니다. 게이트웨이는 `listen.port`(기본값 `8080`)에서 수신합니다.

### Kubernetes

상태 없는 서비스처럼 게이트웨이를 Deployment로 실행하세요:

* ConfigMap에서 설정을, Secret에서 비밀 값을 마운트합니다. YAML 내에서 `${file:/path/to/secret}` 또는 환경 변수를 통해 비밀 값을 참조합니다.
* Ingress에서 TLS를 종료하고 `listen.public_url`을 Ingress 호스트 이름으로 설정합니다.
* 준비성 프로브(readiness probe)는 `GET /readyz`를 가리키고 라이브니스 프로브(liveness probe)는 `GET /healthz`를 가리키도록 설정합니다.

<Note>
  **워크로드 정체성 (Workload identity)**

  정적 키 대신 플랫폼의 워크로드 정체성을 권장합니다: Amazon Bedrock 및 AWS 기반 Claude Platform의 경우 EKS의 IRSA, Google Cloud Agent Platform의 경우 GKE의 Workload Identity, Microsoft Foundry의 경우 AKS의 워크로드 정체성을 사용하세요. 상류 블록에 `auth: {}`를 설정하거나 Microsoft Foundry의 경우 `use_azure_ad: true`를 설정하면 게이트웨이가 해당 제공업체의 기본 자격 증명 체인을 통해 파드의 정체성을 가져옵니다. GKE의 Amazon Bedrock 상류와 같은 교차 클라우드 조합의 경우 상류의 `auth` 블록에 명시적 자격 증명을 대신 설정하세요. [`upstreams` 참조](/docs/en/claude-apps-gateway-config#upstreams)에서 플랫폼별 설정 정보를 확인할 수 있습니다.
</Note>

### Cloud Run

서비스를 다음과 같이 구성하세요:

* `listen.port`는 Cloud Run의 기본 `PORT`와 일치하는 기본값 `8080`으로 두거나 `port: ${PORT}`로 설정하세요.
* `public_url`을 외부에서 도달 가능한 오리진으로 설정하세요. `/login`이 [공개 주소를 거부](/docs/en/claude-apps-gateway#prerequisites)하고 `*.run.app` URL이 공개 주소로 해소되므로 프로덕션 환경의 경우 일반적으로 내부 로드 밸런서의 호스트 이름으로 설정해야 합니다. 따라서 Cloud Run URL 자체는 `curl` 또는 브라우저 테스트 용도로만 사용할 수 있습니다. 예외는 `*.run.app`이 Private Service Connect 및 Cloud DNS 비공개 영역을 통해 비공개로 해소되는 네트워크입니다. 해당 토폴로지에서는 Cloud Run URL이 유효한 `public_url`이 됩니다. [Google Cloud 예제](/docs/en/claude-apps-gateway-on-gcp#deploy-the-gateway)에서 두 가지 모두를 다룹니다.
* 설정을 비밀 볼륨으로 마운트합니다.
* 첫 번째 요청 시 콜드 OIDC 디스커버리를 방지하려면 `min-instances: 1`을 설정하세요.

<Note>
  Cloud Run 또는 GKE, Cloud SQL 및 Secret Manager를 다루는 Google Cloud에서의 전체 구성 예제는 [Google Cloud 배포](/docs/en/claude-apps-gateway-on-gcp)를 참조하세요.
</Note>

### 개발자 머신에 게이트웨이 URL 푸시

게이트웨이가 구동되면 MDM을 통해 또는 각 OS별 `managed-settings.json`을 직접 작성하여 관리형 설정을 통해 `forceLoginMethod`, `forceLoginGatewayUrl` 및 `parentSettingsBehavior: "merge"`를 각 개발자의 머신에 푸시하세요. 이 설정이 없으면 `/login`에 게이트웨이 옵션 없이 표준 계정 선택기가 표시됩니다. 파일 경로는 [클라이언트 측 관리형 설정](/docs/en/claude-apps-gateway-config#client-side-managed-settings)을 참조하세요.

## 운영

게이트웨이가 트래픽을 처리하기 시작하면 일상적인 운영은 로그를 읽고, 상태를 조사하며, 일정에 맞춰 비밀 값을 순환시키는 작업입니다. 각 하위 섹션에서는 이러한 작업과 함께 Postgres에 보관되는 내용, 업그레이드 및 롤백 동작 방식을 다룹니다.

### 로그

게이트웨이는 JSON 파싱이 가능한 두 가지 스트림을 stderr에 작성합니다:

* **감사 이벤트**: 보안 관련 이벤트당 단일 행 JSON. stderr를 로그 수집기로 파이프하세요. 내보내지는 이벤트에는 `config.load`, `session.mint`, `session.refresh`, `device.authorize`, `device.verify`, `auth.denied`, `access.denied`, `inference`, `managed.serve`, `spend.blocked` 및 `admin.denied`가 포함됩니다. 필드는 이벤트마다 다릅니다:
  * 성공적인 토큰 발급(mint) 및 새로 고침(refresh) 이벤트에는 `sub`, `email`, `client_ip` 및 결과가 포함됩니다.
  * 거부 이벤트에는 거부 시점에 정체성이 존재하지 않으므로 사유, 경로 및 클라이언트 IP가 포함됩니다.
  * `inference`는 요청을 처리한 상류 및 응답 상태를 기록합니다.
  * `admin.denied`는 제공된 키 자재 없이 사유(`invalid_key` 또는 `no_credentials`), 클라이언트 IP, 메서드 및 경로를 포함하여 거부된 관리자 API 인증 시도를 기록합니다.
* **운영 로그**: 부팅, 경고 및 상류 에러에 대한 사람이 읽을 수 있는 `[gateway]` 접두사가 붙은 행. `CLAUDE_GATEWAY_LOG_LEVEL` 환경 변수는 상세도를 제어하며 `info`, `warn` 또는 `error`를 수락하고 `info`가 기본값입니다. 항상 전송되는 감사 이벤트에는 영향을 주지 않습니다.

### 헬스 상태 (Health)

게이트웨이는 라이브니스 프로브로 `GET /healthz`를, 준비성 프로브로 `GET /readyz`를 제공합니다. `/readyz`는 저장소 도달 가능 여부를 검증합니다. 두 엔드포인트 모두 `access_control.allow_cidrs`에서 제외되므로 잠긴 수신기에서도 프로브가 계속 작동합니다.

`/.well-known/oauth-authorization-server`에 있는 OAuth 디스커버리 문서도 설정 로드, OIDC 디스커버리, 상류 클라이언트 생성 및 Postgres 마이그레이션이 모두 성공한 후에만 `200`을 반환하므로 엔드투엔드 부팅 검사 용도로 겸용할 수 있습니다.

실행 중인 게이트웨이는 실행 중인 버전에 맞춰 `<public_url>/protocol`에서 수락하는 경로 및 요청 형식을 설명하는 정보도 제공합니다. 내용은 릴리스 간에 변경될 수 있습니다.

### 장애 시 동작

Postgres가 다운되더라도 게이트웨이 자체는 이미 로그인한 개발자를 계속 처리하며 새 로그인은 실패합니다. 개발자가 실제로 계속 작업할 수 있는지 여부는 오케스트레이터가 준비성을 처리하는 방식에 따라 다릅니다:

* **기존 세션**: 전달자 토큰이 JWT 비밀 값으로 로컬에서 검증되고 세션 새로 고침이 저장소를 건드리지 않으므로 게이트웨이 프로세스는 계속 추론을 처리할 수 있습니다.
* **새 로그인**: 디바이스 흐름과 속도 제한 카운터가 Postgres에 존재하므로 Postgres가 복구될 때까지 실패합니다.
* **[지출 한도 적용](/docs/en/claude-apps-gateway-spend-limits#postgres-availability)**: 장애 중 기본적으로 fail-open 방식으로 작동하므로 추론이 계속 흐릅니다. 측정되지 않는 것보다 차단하는 것을 원한다면 fail-closed로 전환하세요.
* **준비성**: `/readyz`가 장애 중 준비되지 않음(not-ready)으로 보고하므로 준비성에 따라 트래픽을 제어하는 오케스트레이터는 모든 레프리카를 로테이션에서 동시에 제거합니다. 해당 토폴로지에서는 게이트웨이가 처리할 수 있는 추론을 포함한 모든 트래픽이 Postgres가 복구될 때까지 로드 밸런서에서 실패합니다. `/healthz`의 라이브니스 프로브는 계속 통과하므로 레프리카가 재시작되지는 않습니다. 저장소 장애 중에도 로그인된 개발자가 계속 일할 수 있게 하려면 준비성 프로브가 `/healthz`를 가리키도록 설정하세요. 이때의 트레이드오프는 여전히 준비됨으로 보고하는 레프리카에 대해 새 로그인이 실패한다는 점입니다.

IdP가 다운되면 기존 세션은 `ttl_hours` 동안 작동하며 새 로그인 및 새로 고침은 실패합니다. IdP에 정기 유지 관리 기간이 잦다면 더 긴 `ttl_hours`를 설정하세요.

### JWT 비밀 값 순환

기존 세션이 유효하게 유지되도록 3단계에 거쳐 서명 비밀 값을 순환하세요:

1. 새 비밀 값을 생성합니다. 이를 `session.jwt_secret` 배열의 맨 앞에 추가합니다.
2. 배포를 새로 고칩니다(roll). 새 토큰은 새 비밀 값으로 서명되고 이전 토큰도 여전히 검증됩니다.
3. `ttl_hours`에 여유 시간을 더한 후 이전 비밀 값을 제거하고 다시 새로 고칩니다.

순환은 만료 전에 세션을 강제로 만료시키는 유일한 방법이기도 합니다. 전달자 토큰이 JWT 비밀 값에 대해 로컬에서 검증되므로 세션별 취소가 존재하지 않습니다. 배열에 이전 값을 유지하지 않고 비밀 값을 완전히 교체하면 기존의 모든 세션이 한 번에 무효화됩니다. 개별 자격을 박탈하려면 IdP에서 해당 사용자를 비활성화하세요. 세션은 `ttl_hours` 이내에 종료됩니다.

### Postgres

게이트웨이는 부팅 시 마이그레이션으로 생성된 5개 테이블을 유지 관리합니다:

| 테이블             | 내용                                                                          | 보존 정책                                                       |
| ------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `kv`               | 디바이스 인가 (10분 TTL) 및 속도 제한 카운터                                  | 행별 TTL                                                        |
| `spend`            | 보안 주체별 현재 누적 지출 카운터 (센트 단위)                                 | `admin.spend_retention_months`, 기본값 13                       |
| `spend_limits`     | 구성된 지출 한도 상한선                                                       | API를 통해 삭제될 때까지                                        |
| `admin_audit`      | 관리자 API 변경 이력                                                          | `admin.audit_retention_days`, 기본값 365                        |
| `principal_emails` | 각 보안 주체의 최근 확인된 이메일, 표시 이름 및 IdP 그룹 (개인정보 포함).     | 마지막 활동 후 `admin.identity_retention_days`, 기본값 90       |

30초 주기의 루프가 TTL이 지난 `kv` 행을 만료시키며, 매시간 수행되는 정리가 지출 테이블의 보존 기간을 적용하므로 무제한으로 늘어나지 않습니다. [지출 한도](/docs/en/claude-apps-gateway-spend-limits)가 구성되지 않은 경우 `kv`만 작성됩니다. 보안 정책상 애플리케이션 역할의 DDL이 금지된 경우 관리자 역할로 이 테이블들과 `_migrations`를 미리 생성하고 앱 역할에 각각 `SELECT, INSERT, UPDATE, DELETE` 권한을 부여하세요.

지출 한도를 사용하는 경우 데이터베이스를 분실하면 개발자 재로그인뿐만 아니라 지출 추적 및 한도 정보도 분실되므로 정기적으로 백업을 실행하세요. 보존 기간을 기다리지 않고 퇴사한 개발자 한 명의 정보를 즉시 삭제하려면 `DELETE FROM principal_emails WHERE principal = '<sub>'` 명령을 직접 실행하세요. 그러면 해당 개발자의 이메일, 이름, 그룹을 보유한 유일한 테이블이 삭제됩니다. `spend` 및 `admin_audit` 행은 익명 처리된 OIDC `sub`만 참조합니다.

### 업그레이드

레프리카는 상태가 없으므로 언제든지 롤링 재시작을 수행해도 안전합니다. 게이트웨이는 부팅 시 스키마 마이그레이션을 실행하므로 새 바이너리를 배포하면 데이터베이스가 자동으로 마이그레이션됩니다. 데이터베이스 역할이 DDL을 실행할 수 없는 경우 현재 버전으로 시드된 `_migrations` 테이블을 포함하여 스키마를 미리 생성하세요. 그렇지 않으면 부팅 시 `CREATE TABLE` 시도 중 실패합니다.

마이그레이션은 추가 전용(append-only)이므로 더 적은 마이그레이션을 아는 이전 바이너리로 롤백해도 안전합니다. 추가된 행을 무시하기 때문입니다. 또한 롤백 시 이전 바이너리의 스키마에 대해 YAML을 다시 검증하므로 새 릴리스에서 도입된 키를 적용한 설정은 이전 바이너리에서 부팅 시 실패합니다. 롤백하기 전에 새 키를 제거하세요.

자체 이미지에 게이트웨이 버전을 고정하므로 보안 패치를 포함하여 새 Claude Code 릴리스의 수정 사항은 고정을 업데이트하고 재배포할 때만 적용됩니다. 프로덕션 자격 증명을 보유하는 다른 서비스에 사용하는 것과 동일한 패치 주기에 게이트웨이를 포함하세요.

## 보안

이 섹션에서는 보안 검토에서 묻는 질문에 대한 답변을 다룹니다: 어떤 데이터가 게이트웨이를 통과하고 어디로 가는지, 설계상 어떤 공격을 방어하는지, 규정 준수 설문지에 대한 답변.

### 데이터 흐름

| 데이터                                                                                            | 경로                                                         | 게이트웨이가 Anthropic으로 전송 여부               |
| ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| 추론 (프롬프트, 완성)                                                                             | CLI → 게이트웨이 → 자체 상류                                 | Anthropic API가 상류로 설정된 경우에만 전송       |
| 텔레메트리 (OTLP 메트릭 및 [선택적 로그/트레이스](/docs/en/claude-apps-gateway-config#telemetry)) | CLI → 게이트웨이 → 자체 수집기                               | 전송하지 않음                                      |
| 정체성 (이메일, 그룹, sub)                                                                        | IdP → 게이트웨이 → JWT → CLI; CLI가 OTLP 내보내기에 스탬프   | 전송하지 않음                                      |
| 관리형 설정                                                                                       | 자체 게이트웨이 YAML → CLI                                   | 전송하지 않음                                      |
| 감사 로그                                                                                         | 게이트웨이 stderr → 자체 집계기                              | 전송하지 않음                                      |

### 위협 모델 요약

게이트웨이는 네트워크 경계 내부에 위치하지만 개별 개발자 랩톱을 신뢰할 수 있는 것으로 취급하지 않습니다. 설계상 세 가지 방식으로 이를 반영합니다:

* 개발자는 원시 상류 키 대신 단기 JWT를 보유합니다. CLI-게이트웨이 구간은 RFC 8628 디바이스 인가를 사용하고, 게이트웨이와 IdP 간의 인가 코드 교환은 기본 설정에서 PKCE를 실행하므로 가로챈 IdP 인가 코드는 무용지물이 됩니다.
* 디바이스 검증 페이지는 동일 오리진 POST 및 RFC 8628 §5.1에 따른 IP별 속도 제한을 적용합니다. [사용자 코드 무차별 대입 방어력](#사용자-코드-무차별-대입-방어력)을 참조하세요.
* 아웃바운드 요청은 DNS를 해소하고 링크 로컬, 클라우드 메타데이터 주소 및 기본적으로 루프백을 차단하며 연결을 해소된 IP에 고정하는 서버 측 요청 위조(SSRF) 보호조치를 통과하므로 IdP 및 OTLP 목적지와 같은 운영자 영향 URL이 클라우드 메타데이터 엔드포인트로 리다이렉트될 수 없습니다. IdP 및 OTLP 수집기가 일반적으로 사설 IP에 존재하므로 RFC 1918 사설 범위는 의도적으로 허용됩니다. 루프백 IdP 또는 수집기에 대한 로컬 개발의 경우 게이트웨이 환경에 `CLAUDE_GATEWAY_ALLOW_LOOPBACK=1`을 설정하고, 프로덕션 환경에서는 설정하지 않은 상태로 두세요.

자체 이그레스 제어를 추가하는 경우 인스턴스 메타데이터 자격 증명(워크로드 정체성 등)을 사용할 때마다 게이트웨이가 메타데이터 서버에 도달할 수 있어야 합니다.

두 가지 위협은 사용자의 인프라 보안 범위에 속하므로 게이트웨이 설계 범위를 벗어납니다:

* **탈취된 게이트웨이 호스트**: 호스트가 상류 자격 증명을 보관하고 모든 연결된 개발자에게 [관리형 설정](/docs/en/claude-apps-gateway-config#managed)을 분배하므로, 게이트웨이 설정에 대한 제어는 MDM 제어와 맞먹습니다. 쉘 가능 설정에 대한 CLI의 일회성 승인 대화 상자는 묵시적 변경을 제한하지만 호스트 보안을 대체하지는 않습니다.
* **악의적인 OIDC 제공업체**: 제공업체는 게이트웨이가 신뢰하는 id_token에 서명하므로 임의의 정체성을 주장할 수 있습니다. IdP 검증 및 보안 확보는 사용자의 책임입니다.

### 사용자 코드 무차별 대입 방어력

개발자가 `/device` 검증 페이지에 입력하는 `user_code`는 20개 문자 알파벳에서 도출된 8자(20⁸ 또는 약 2.56×10¹⁰ 조합)로 구성되며 10분 후 만료됩니다.

게이트웨이는 디바이스 인가 엔드포인트에 IP별 속도 제한을 적용하며, [`rate_limits`](/docs/en/claude-apps-gateway-config#http-tuning)를 통해 설정할 수 있습니다. 대규모 개발자가 단일 공유 기업 NAT 주소에서 로그인하는 경우 제한을 올리세요. 이 제한은 추론이 아닌 로그인 흐름에만 적용됩니다.

### 규정 준수 상태

* **데이터 거주성**: Anthropic API가 구성된 상류가 아닌 한 게이트웨이의 자체 데이터 플레인은 Anthropic으로 아무것도 전송하지 않습니다. Anthropic API가 상류인 경우 기존 데이터 처리 계약이 추론 경로에 적용됩니다. 텔레메트리, 감사, 정체성 및 설정은 설정한 목적지로만 전송됩니다.
* **호스트 프로세스 트래픽**: 호스트 프로세스는 Claude Code CLI이며 Anthropic에 시작 분석 및 업데이트 확인을 전송할 수 있습니다. 엄격한 이그레스 배포의 경우 게이트웨이 컨테이너 환경에 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`을 설정하세요.
* **클라이언트 분석**: CLI는 게이트웨이에 로그인되어 있는 동안 자체 사용량 분석을 비활성화하며 서드파티 API 표면에서는 오류 보고가 기본적으로 꺼져 있습니다.
* **클라이언트 머신**: 개발자의 CLI는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1` 및 `skipWebFetchPreflight: true`가 설정되지 않은 한 Anthropic에 WebFetch 호스트 이름 검사 및 버전 검사를 계속 전송합니다. [데이터 사용량](/docs/en/data-usage)을 참조하세요.
* **설문조사 평가**: 게이트웨이 자격 증명은 Anthropic 대상 평가 싱크를 비활성화하므로 평가가 Anthropic으로 전송되지 않습니다.
* **트랜스크립트 공유**: 설문조사의 트랜스크립트 공유 프롬프트에서 '예'를 선택하면 Anthropic에 업로드하는 대신 `~/.claude/feedback-bundles/` 아래에 로컬 파일로 저장됩니다.
* **클라이언트 업데이트**: 업데이트 검사는 게이트웨이 트래픽과 별개입니다. 랩톱에서 릴리스를 가져오지 않아야 하는 경우 자체 배포 방식을 통해 버전을 고정하고 `DISABLE_UPDATES`를 설정하세요. `DISABLE_AUTOUPDATER`는 배경 업데이트만 중지하며 `claude update`는 계속 작동합니다.
* **TLS**: 프로덕션 환경에서 `public_url`을 HTTPS로 제공하세요(`listen.tls`를 통해 게이트웨이 수신기에서 직접 제공하거나 `listen.public_url`이 설정된 일반 HTTP 레프리카 앞단의 TLS 종료 인그레스에서 제공). 게이트웨이는 일반 HTTP를 거부하지 않습니다. IdP는 프로덕션에서 HTTPS를 제공해야 하며 Postgres는 `?sslmode=require`를 지원합니다. 인그레스에서 `Strict-Transport-Security`를 설정하세요.
* **취약점 공개**: [보안 문제 보고](/docs/en/security#reporting-security-issues)를 따르세요.

## 문제 해결

질문 및 피드백은 [Claude Code 지원](https://support.claude.com/en/collections/14445694-claude-code)을 사용하거나 [Claude Code GitHub 리포지토리](https://github.com/anthropics/claude-code/issues)에 이슈를 생성하세요. 문제를 보고할 때 다음을 포함해야 합니다:

* **게이트웨이 문제**: 관련 기간의 게이트웨이 stderr, 비밀 값이 마스킹된 `gateway.yaml`, 게이트웨이 버전(`/` 랜딩 페이지 및 `/managed/settings`의 `x-cc-gateway-version` 응답 헤더에 표시됨), 최근 변경 사항
* **로그인 문제**: 개발자가 `claude --debug-file ./claude-debug.txt`를 실행하여 재현하고 해당 파일과 해당 기간의 게이트웨이 감사 로그를 전송
* **추론 문제**: 요청된 모델, 구성된 상류, 해당 요청을 처리한 상류 및 응답 상태가 기록된 게이트웨이 감사 로그

게이트웨이의 stderr에는 감사 이벤트 스트림이 포함되고 감사 로그에는 개발자 정체성이 기록되며 디버그 파일에는 개발자 머신의 훅 및 MCP 서버 출력이 기록됩니다. 공개 이슈에 게시하기 전에 이를 검토하고 마스킹하세요.

| 증상                                                                                                                                                                        | 원인                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | 해결 방법                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 개발자의 `/login`에 **Cloud gateway** 화면 대신 표준 계정 선택기가 표시됨                                                                                                  | 해당 머신의 관리형 설정에 `forceLoginMethod` 또는 `forceLoginGatewayUrl`이 설정되지 않음                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 기기에 [관리형 설정 파일](/docs/en/claude-apps-gateway#set-the-gateway-url)을 배포하세요. `/login`은 거기서 게이트웨이 URL을 읽습니다.                                                                                                                                                                                                                                                                                               |
| 시작 시 `Gateway login is configured in managed settings, but this Claude Code build does not include Cloud gateway support.` 메시지 표시                                   | 설치된 Claude Code 빌드가 게이트웨이 지원보다 이전 버전임                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 개발자가 Cloud gateway 지원이 포함된 릴리스로 Claude Code를 업데이트하도록 하세요.                                                                                                                                                                                                                                                                                                                                                      |
| CLI `/login`: `Gateway hosts must be on your organization's private network; <host> resolves to the public (or unrecognized) address <ip>`                                  | 게이트웨이 호스트 이름이 하나 이상의 공개 IP 주소로 해소됨. Claude Code는 해소된 각 주소를 검사하고 모든 주소가 사설 주소일 것을 요구합니다. 일반적인 원인은 한 패밀리가 공개 주소로 해소되는 듀얼 스택 이름(공개 범위 AAAA 주소를 반환하는 AWS 내부 듀얼 스택 로드 밸런서 포함)입니다. {/* min-version: 2.1.206 */}Anthropic 운영 공개 게이트웨이 엔드포인트는 이 검사에서 제외되며 `/login`은 `https://`에서 수용합니다. v2.1.206 이전에는 `/login`이 다른 공개 주소와 마찬가지로 거부했습니다. | 개발자 머신에서 게이트웨이 이름이 사설 주소로만 해소되도록 하세요. 듀얼 스택 이름의 경우 공개 범위 레코드를 제거하거나 별도의 내부 전용 DNS 이름을 제공하세요. [사설 네트워크 요구 사항](/docs/en/claude-apps-gateway#prerequisites)을 참조하세요.                                                                                                                    |
| CLI `/login`: `Gateway login requires a direct connection and does not support connecting through an HTTP proxy`                                                            | `HTTPS_PROXY` 또는 `HTTP_PROXY`가 게이트웨이 호스트에 적용되고 프록시의 호스트 이름이 공개 주소로 해소됨. 사설 주소로만 해소되는 프록시는 허용되며 이 오류를 트리거하지 않습니다.                                                                                                                                                                                                                                                                                                                                                                              | 연결이 직접 이루어지도록 개발자 머신의 `NO_PROXY`에 게이트웨이 호스트를 추가하거나 호스트 이름이 사설 주소로 해소되는 프록시를 사용하세요.                                                                                                                                                                                                                                                                                             |
| CLI `/login`: `Could not resolve gateway host <host>`                                                                                                                       | 머신이 게이트웨이의 내부 DNS 이름을 해소할 수 없음 (일반적으로 기업 네트워크에 접속되어 있지 않기 때문)                                                                                                                                                                                                                                                                                                                                                                                                                                            | 개발자가 네트워크 또는 VPN에 연결한 다음 `/login`을 다시 시도하도록 하세요.                                                                                                                                                                                                                                                                                                                                                             |
| `store.postgres_url`을 지적하는 설정 검증 에러와 함께 부팅 종료                                                                                                           | Postgres가 설정되지 않음. 게이트웨이에 Postgres 필수                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `store.postgres_url`을 설정하세요. 로컬 개발의 경우 일회성 컨테이너 `docker run --rm -p 5432:5432 -e POSTGRES_HOST_AUTH_METHOD=trust postgres`를 사용하세요.                                                                                                                                                                                                                                                                            |
| 부팅 종료: `requires the native binary`                                                                                                                                    | 네이티브 바이너리 대신 Node 환경에서 실행 중                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | [단독 설치 방식](/docs/en/setup) 중 하나로 Claude Code를 설치하세요.                                                                                                                                                                                                                                                                                                                                                                    |
| `config.load` 후 OIDC 디스커버리 에러와 함께 부팅 종료                                                                                                                     | `oidc.issuer`에 도달할 수 없거나 TLS 체인을 신뢰할 수 없음                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 파드에서 발급자에 도달할 수 있고 `/.well-known/openid-configuration`을 제공하는지 확인하세요. 사설 PKI의 경우 `ca_cert_pem`을 설정하세요.                                                                                                                                                                                                                                                                                              |
| Postgres 권한 에러와 함께 부팅 종료                                                                                                                                        | 앱 역할에 `CREATE TABLE` 권한 부족                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 관리자 역할로 스키마를 미리 생성하고 앱 역할에 DML 권한을 부여하거나, 새 마이크레이션을 적용하는 부팅 시 임시로 DDL 권한을 부여하세요.                                                                                                                                                                                                                                                                                                  |
| `/oauth/callback`에 "Sign-in could not be completed" 표시                                                                                                                   | 이메일 도메인이 거부되었거나 id_token 검증 실패, 또는 게이트웨이가 재정의 없이 항상 거부하는 `email_verified`가 명시적으로 `false`임                                                                                                                                                                                                                                                                                                                                                                                                       | `allowed_email_domains`를 확인하고 IdP가 검증된 `email` 클레임을 반환하는지 확인하세요. `email_verified: false`의 경우 IdP 측 검증을 수정하세요. IdP가 다른 클레임 이름 아래 이메일을 내보내는 경우 `oidc.email_claim`을 설정하세요.                                                                                                                                                                                                         |
| 로그: `token exchange failed: id_token missing email claim`                                                                                                                 | IdP가 기본적으로 id_token에 `email`을 포함하지 않음. 이 거부는 `allowed_email_domains`가 설정된 경우에만 발생합니다. 설정되지 않은 경우 이메일이 누락되어 이메일이 없는 세션이 발급됩니다.                                                                                                                                                                                                                                                                                                                                                       | id_token에 `email`을 내보내도록 IdP를 구성하세요. Okta: 커스텀 인가 서버의 ID 토큰 클레임에 `email`을 추가하세요. Entra: 앱 등록 시 `email`을 선택적 클레임으로 추가하세요. PingFederate: `email`을 내보내는 OpenID Connect Policy를 활성화하세요. Okta 조직 인가 서버처럼 IdP가 userinfo 엔드포인트에서 `email`을 제공하지만 id_token에는 포함하지 않는 경우 `oidc.userinfo_fallback: true`를 설정하세요.    |
| 모든 Amazon Bedrock 요청이 502를 반환; 로그에 `Could not load credentials from any providers` 표시                                                                          | EC2에서 IMDSv2의 기본 홉 제한(hop limit) 1로 인해 컨테이너 내부에서 인스턴스 메타데이터 요청이 차단됨. AWS SDK가 클라이언트 생성 시점이 아닌 첫 요청 시 인스턴스 자격 증명을 해소하므로 부팅 및 `/readyz`는 통과함                                                                                                                                                                                                                                                                                                                              | `aws ec2 modify-instance-metadata-options --instance-id <id> --http-put-response-hop-limit 2` 명령으로 홉 제한을 올리거나 시작 템플릿에 설정하세요. 변경 사항은 인스턴스의 모든 컨테이너에 적용됩니다. ECS 컨테이너 자격 증명 엔드포인트에서 자격 증명을 읽어 변경을 완전히 피할 수 있는 ECS 태스크 역할을 선호하거나, 전용 게이트웨이 인스턴스에 변경을 적용하여 노출을 제한하세요.                              |
| IdP 에러: unknown or unsupported scope                                                                                                                                     | IdP가 인식하지 못하는 스코프를 거부함                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `oidc.scopes`를 IdP가 수락하는 목록으로 정확히 설정하세요. `openid`를 포함해야 합니다. 기본값은 `openid profile email offline_access`입니다.                                                                                                                                                                                                                                                                                             |
| `oidc.scopes` 설정 후 세션이 배경에서 자동 갱신되지 않음                                                                                                                    | 재정의 시 `offline_access`가 빠짐                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | IdP가 지원하는 경우 `offline_access`를 다시 추가하세요. 리프레시 토큰이 없으면 개발자는 `session.ttl_hours`마다 브라우저 로그인을 다시 실행하게 됩니다.                                                                                                                                                                                                                                                                                        |
| 브라우저에 "This request came from another site and was blocked" 표시                                                                                                       | CSRF 보호로 차단된 교차 사이트 폼 POST. 임베디드 또는 프록시 처리된 페이지에서 예상되는 동작                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 검증 링크를 직접 열으세요.                                                                                                                                                                                                                                                                                                                                                                                                              |
| Chrome에서 승인 버튼이 "Refused to send form data … violates … Content Security Policy directive: form-action"으로 차단되지만 Safari나 Firefox에서는 작동함          | Chrome은 전체 리다이렉트 체인에 대해 `form-action`을 적용함. IdP가 허용 목록에 없는 두 번째 호스트로 리다이렉트함.                                                                                                                                                                                                                                                                                                                                                                                                                                 | 리다이렉트 체인의 각 추가 오리진을 `oidc.form_action_origins`에 추가하세요. 승인 페이지에서 Chrome DevTools → Console을 열어 어떤 오리진이 차단되었는지 확인하세요.                                                                                                                                                                                                                                                                      |
| IdP에서 로그인이 완료되었지만 콜백이 실패하며 Chrome에서 CSP 에러가 나거나 Safari에서 "this sign-in link has expired" 표시                                                  | IdP가 `response_mode=form_post`를 통해 코드를 반환하여 `/oauth/callback`에 POST로 교차 오리진 자동 제출을 시도함. Chrome은 엄격한 CSP 하에 이를 차단하며 Safari는 제출을 허용하지만 콜백이 쿼리 문자열만 읽음.                                                                                                                                                                                                                                                                                                                                   | 게이트웨이가 명시적으로 요청하는 `response_mode=query`를 IdP가 준수하도록 구성하여 콜백이 일반 리다이렉트가 되도록 하세요.                                                                                                                                                                                                                                                                                                              |
| 로컬에서는 로그인이 작동하지만 ALB 뒤에서는 실패함                                                                                                                          | `public_url`이 설정되지 않아 IdP가 내부 `http://` 오리진을 `redirect_uri`로 받음                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | `listen.public_url`을 외부 `https://` 오리진으로 설정하세요.                                                                                                                                                                                                                                                                                                                                                                            |
| 개발자에게 신뢰 프롬프트가 반복해서 표시됨                                                                                                                                  | TLS 인증서가 레프리카별 또는 요청별로 순환함                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 인그레스에서 안정된 인증서를 사용하거나 TLS를 한 번 종료하고 내부에서 일반 HTTP로 레프리카를 실행하세요.                                                                                                                                                                                                                                                                                                                                |
| CLI `/login`: "Could not verify the gateway's TLS certificate" 또는 `SELF_SIGNED_CERT_IN_CHAIN`                                                                            | 게이트웨이의 TLS 체인이 CLI 호스트의 신뢰 저장소에 없는 사설 CA에 의해 서명됨                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Claude Code는 네이티브 바이너리 및 Node 22.15 이상에서 기본적으로 OS 신뢰 저장소를 읽습니다. [`CLAUDE_CODE_CERT_STORE`](/docs/en/network-config#ca-certificate-store)가 이 동작을 제어합니다. CA가 OS 신뢰 저장소에 설치되어 있다면 개발자가 최신 런타임을 사용 중인지 확인하세요. 그렇지 않은 경우 실행 전에 `NODE_EXTRA_CA_CERTS`를 CA 인증서 PEM으로 설정하세요. 첫 연결 신뢰 지문 프롬프트는 여전히 적용됩니다.               |

## 관련 항목

* [Claude apps gateway 개요](/docs/en/claude-apps-gateway): 퀵스타트 및 개발자 연결
* [설정 참조](/docs/en/claude-apps-gateway-config): 모든 `gateway.yaml` 옵션
