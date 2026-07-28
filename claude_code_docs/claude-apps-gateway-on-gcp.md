> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Google Cloud에 Claude apps gateway 배포하기

> Google Cloud에서 Claude apps gateway를 실행하는 가이드 예제: Cloud Run 또는 GKE, PostgreSQL용 Cloud SQL, Secret Manager, 그리고 Google Cloud Agent Platform에 대한 서비스 계정 인증.

<Note>
  이 페이지에서는 Google Cloud에서 Claude apps gateway를 실행하는 한 가지 방법을 다룹니다. 이 설정은 지원되는 프로덕션 배포가 아닌 고객 관리형 인프라에 대한 실습 예제입니다. 사용자의 환경에 맞게 조정하기 전에 각 요소가 어떻게 결합하는지 확인하는 용도로 활용하세요. 플랫폼에 구애받지 않는 요구 사항은 [배포 가이드](/docs/en/claude-apps-gateway-deploy)를 참조하세요.
</Note>

이 예제에서는 컴퓨팅 환경으로 Cloud Run 또는 GKE를 사용하고 모델 상류(upstream)로 Google Cloud Agent Platform을 사용하는 Google Cloud 기반 Claude apps gateway를 구성합니다. 예제 정체성 제공업체(IdP)로는 Google Workspace를 사용하지만, OpenID Connect (OIDC) 준수 IdP라면 모두 가능하며 `oidc` 블록만 변경됩니다. IdP별 세부 정보는 [정체성 제공업체 설정](/docs/en/claude-apps-gateway-deploy#identity-provider-setup)을 참조하세요.

## 구축할 아키텍처

<Frame>
  <img src="https://mintcdn.com/claude-code/-uq-4JE0W_JO5Er5/images/claude-gateway-gcp-architecture.svg?fit=max&auto=format&n=-uq-4JE0W_JO5Er5/images/claude-gateway-gcp-architecture.svg" alt="Diagram of Claude apps gateway on Google Cloud: Claude Code clients connect over HTTPS to the gateway (Cloud Run or GKE), which runs inside a VPC alongside a private-IP Cloud SQL database for session state. The gateway signs users in via OIDC against Google Workspace, reads config and secrets from Secret Manager, forwards model requests to Google Cloud's Agent Platform, and pulls its image from Artifact Registry at deploy." width="760" height="400" data-path="images/claude-gateway-gcp-architecture.svg" />
</Frame>

참조 설정에서 프로비저닝하는 리소스:

* 게이트웨이 컨테이너를 실행하는 **Cloud Run** 서비스 또는 **GKE** Deployment
* 게이트웨이 이미지를 저장하는 **Artifact Registry** 저장소
* 게이트웨이의 [저장소(store)](/docs/en/claude-apps-gateway-config#store)용 사설 IP 전용 **PostgreSQL용 Cloud SQL** 인스턴스
* `gateway.yaml`, JWT 서명 키, OIDC 클라이언트 비밀 값, Postgres URL을 위한 **Secret Manager** 비밀 값
* `roles/aiplatform.user` 권한을 가진 **서비스 계정** (Cloud Run에 직접 연결되거나 GKE에서 Workload Identity를 통해 바인딩됨)
* HTTPS를 위한 Cloud Run의 **내부 애플리케이션 로드 밸런서** 또는 GKE의 `gce-internal` 클래스 **내부 GKE Ingress**

## 사전 요구 사항

* 결제가 활성화되어 있고 위 리소스를 생성할 권한이 있는 GCP 프로젝트
* `gcloud auth login`으로 인증된 `gcloud` CLI 및 로컬에 설치된 Docker
* GKE 경로의 경우: `kubectl` 및 아래 가이드에서 생성된 VPC에 위치한 GKE 클러스터
* 모델을 게시하는 지역의 Model Garden에서 필요한 Claude 모델에 대한 접근 권한
* 리다이렉트 URI `https://<gateway-host>/oauth/callback`을 가진 Google Workspace OAuth 2.0 웹 애플리케이션 클라이언트 ([정체성 제공업체 설정](/docs/en/claude-apps-gateway-deploy#identity-provider-setup) 참조)
* 로드 밸런서를 가리키는 내부 DNS 이름 형태의 게이트웨이용 TLS 호스트 이름

프로젝트 및 지역을 한 번 설정합니다:

```bash theme={null}
export PROJECT_ID=<your-project>
export REGION=us-east5   # Model Garden에 필요한 Claude 모델이 게시된 지역
gcloud config set project "$PROJECT_ID"
```

## 게이트웨이 배포

아래 단계는 `gcloud` 명령어로 전체 배포 환경을 프로비저닝합니다.

<Steps>
  <Step title="API 활성화">
    가이드에서 사용하는 서비스 API를 활성화합니다:

    ```bash theme={null}
    gcloud services enable \
      aiplatform.googleapis.com \
      artifactregistry.googleapis.com \
      sqladmin.googleapis.com \
      secretmanager.googleapis.com \
      iamcredentials.googleapis.com \
      iam.googleapis.com \
      compute.googleapis.com \
      servicenetworking.googleapis.com \
      run.googleapis.com \
      container.googleapis.com
    ```

    필요한 API는 배포 경로에 따라 다릅니다:

    * `compute` 및 `servicenetworking`: 사설 IP Cloud SQL 경로에 필요
    * `run`: Cloud Run 전용
    * `container`: GKE 전용
  </Step>

  <Step title="서비스 계정 생성 및 IAM 권한 부여">
    게이트웨이는 Google Cloud Agent Platform을 호출할 권한이 있는 전용 서비스 계정으로 실행됩니다. 비밀번호 사용자를 통해 VPC를 넘어 Cloud SQL에 도달하므로 Cloud SQL IAM 역할은 필요하지 않습니다:

    ```bash theme={null}
    gcloud iam service-accounts create claude-gateway --display-name="Claude apps gateway"
    SA="claude-gateway@${PROJECT_ID}.iam.gserviceaccount.com"

    gcloud projects add-iam-policy-binding "$PROJECT_ID" \
      --member="serviceAccount:${SA}" --role="roles/aiplatform.user" --condition=None
    ```

    그런 다음 Model Garden에서 프로젝트에 필요한 Claude 모델을 활성화합니다. 모델은 특정 지역에 게시되므로 각 모델 카드를 확인하세요.
  </Step>

  <Step title="이미지 빌드 및 Artifact Registry에 푸시">
    `linux-x64` glibc 바이너리를 사용하여 [컨테이너 이미지 요구 사항](/docs/en/claude-apps-gateway-deploy#container-image)에 따라 이미지를 빌드하고 푸시합니다:

    ```bash theme={null}
    gcloud artifacts repositories create claude-gateway \
      --repository-format=docker --location="$REGION"
    gcloud auth configure-docker "${REGION}-docker.pkg.dev" --quiet

    # Cloud Run에는 linux/amd64가 필요합니다. --provenance=false는 Cloud Run이 거부하는 buildx OCI
    # 이미지 인덱스를 방지합니다.
    docker build --platform=linux/amd64 --provenance=false \
      -t "${REGION}-docker.pkg.dev/${PROJECT_ID}/claude-gateway/gateway:<version>" .
    docker push "${REGION}-docker.pkg.dev/${PROJECT_ID}/claude-gateway/gateway:<version>"
    ```
  </Step>

  <Step title="PostgreSQL용 Cloud SQL 프로비저닝">
    공개 IP가 없도록 Private Services Access를 통해 VPC에 인스턴스를 생성합니다. 이는 `constraints/sql.restrictPublicIp`가 적용되는 프로젝트에도 부합합니다:

    ```bash theme={null}
    VPC=cc-gateway-vpc
    gcloud compute networks create "$VPC" --subnet-mode=custom
    gcloud compute networks subnets create cc-gateway-subnet \
      --network="$VPC" --region="$REGION" --range=10.0.0.0/24

    # Private Services Access: VPC당 일회성 설정
    gcloud compute addresses create "google-managed-services-${VPC}" \
      --global --purpose=VPC_PEERING --prefix-length=16 --network="$VPC"
    gcloud services vpc-peerings connect \
      --service=servicenetworking.googleapis.com \
      --ranges="google-managed-services-${VPC}" --network="$VPC"

    gcloud sql instances create claude-gateway-db \
      --database-version=POSTGRES_16 --tier=db-g1-small --region="$REGION" \
      --network="projects/${PROJECT_ID}/global/networks/${VPC}" --no-assign-ip
    gcloud sql databases create claude_gateway --instance=claude-gateway-db
    PGPASS="$(openssl rand -hex 24)"
    gcloud sql users create gateway --instance=claude-gateway-db --password="$PGPASS"

    PRIVATE_IP="$(gcloud sql instances describe claude-gateway-db \
      --format='value(ipAddresses[0].ipAddress)')"
    GATEWAY_POSTGRES_URL="postgres://gateway:${PGPASS}@${PRIVATE_IP}:5432/claude_gateway?sslmode=require"
    ```

    Cloud Run 또는 GKE 런타임이 이 VPC에 참여하거나 라우팅되어야 합니다.
  </Step>

  <Step title="gateway.yaml 작성">
    `upstreams` 블록은 `auth: {}`와 함께 Google Cloud Agent Platform을 가리키므로 게이트웨이는 런타임 서비스 계정의 Application Default Credentials를 통해 인증합니다. 모든 필드는 [설정 참조](/docs/en/claude-apps-gateway-config)를 참조하세요.

    두 `listen` 필드는 게이트웨이 앞단 프록시에 따라 다릅니다:

    * `public_url`: Cloud Run 또는 GKE Ingress 뒤에서 필수입니다. 게이트웨이는 `X-Forwarded-*` 헤더가 아닌 오직 이 값에서만 IdP `redirect_uri` 및 디스커버리 문서를 구축합니다.
    * `trusted_proxies`: 프런트엔드의 소스 범위. 게이트웨이는 TCP 피어가 이 목록에 있을 때만 `X-Forwarded-For`를 수락한 후 신뢰할 수 있는 홉을 건너뛰어, IP별 로그인 속도 제한 및 감사 이벤트에 로드 밸런서 대신 개발자의 IP가 기록되도록 합니다.

    프런트엔드에 맞게 `trusted_proxies`를 설정하세요. 클래스가 `gce`인 외부 GKE Ingress는 나열하지 않습니다. 이는 공개 전달 규칙 주소를 프로비저닝하므로 `/login`의 [사설 네트워크 검사](/docs/en/claude-apps-gateway#prerequisites)에서 거부됩니다.

    | 프런트엔드                                              | `trusted_proxies`                                   |
    | ------------------------------------------------------- | --------------------------------------------------- |
    | 로드 밸런서 없이 직접 도달하는 Cloud Run                | `[169.254.0.0/16]`                                  |
    | Cloud Run 앞단의 내부 애플리케이션 로드 밸런서          | `169.254.0.0/16` 및 프록시 전용 서브넷의 CIDR       |
    | `gce-internal` 클래스 GKE 내부 Ingress                  | 프록시 전용 서브넷의 CIDR                           |

    아래 예제는 'Cloud Run 앞단의 내부 애플리케이션 로드 밸런서' 값을 사용합니다.

    ```yaml gateway.yaml theme={null}
    listen:
      host: 0.0.0.0
      port: 8080
      public_url: https://claude-gateway.internal.example.com
      trusted_proxies: [169.254.0.0/16, <your-proxy-only-subnet-cidr>]

    oidc:
      issuer: https://accounts.google.com
      client_id: <your-oauth-client-id>
      client_secret: ${OIDC_CLIENT_SECRET}           # GKE: ${file:/secrets/oidc-client-secret}
      allowed_email_domains: [example.com]
      # Google은 offline_access를 무시합니다. 다음 설정으로 리프레시 토큰을 구합니다:
      scopes: [openid, profile, email]
      extra_auth_params: { access_type: offline, prompt: consent }

    session:
      jwt_secret: ${GATEWAY_JWT_SECRET}              # GKE: ${file:/secrets/jwt-secret}

    store:
      postgres_url: ${GATEWAY_POSTGRES_URL}          # GKE: ${file:/secrets/postgres-url}

    upstreams:
      - provider: vertex
        region: <your-region>                        # $REGION과 일치해야 함
        project_id: <your-project>
        auth: {}                                     # 런타임 서비스 계정을 통한 ADC
    ```

    <Note>
      Google id_token에는 `groups` 클레임이 포함되지 않습니다. Google Workspace를 IdP로 사용하는 경우 [`managed.policies`](/docs/en/claude-apps-gateway-config#managed)에서 그룹 기반 정책을 사용하려면 도메인 전체 위임 권한이 있는 서비스 계정을 사용하여 Admin SDK Directory API를 통해 각 사용자의 그룹을 조회하는 [`oidc.google_groups`](/docs/en/claude-apps-gateway-config#oidc)를 구성하세요. 구성하지 않는 경우 대신 `email_domain`과 일치시키세요.
    </Note>
  </Step>

  <Step title="Secret Manager에 비밀 값 저장">
    4개의 비밀 값을 생성하고 `claude-gateway` 서비스 계정에 `roles/secretmanager.secretAccessor` 권한을 부여하세요:

    | 비밀 값                      | 소스                                            |
    | ---------------------------- | ----------------------------------------------- |
    | `gateway-jwt-secret`         | `openssl rand -base64 32`                       |
    | `gateway-oidc-client-secret` | Google Cloud 콘솔 → OAuth 클라이언트            |
    | `gateway-postgres-url`       | Cloud SQL 단계의 `$GATEWAY_POSTGRES_URL`        |
    | `gateway-config`             | 이전 단계의 전체 `gateway.yaml`                 |

    비밀 값이 컨테이너에 전달되는 방식은 경로에 따라 다릅니다:

    * GKE에서는 Secret Manager CSI 드라이버를 통해 `/secrets`에 파일로 마운트되며, `gateway.yaml`은 `${file:/secrets/...}`를 참조합니다.
    * 하나의 디렉터리에 여러 비밀 값을 마운트할 수 없는 Cloud Run의 경우 `gateway.yaml`이 파일로 마운트되고 다른 3개는 환경 변수로 주입되므로 `gateway.yaml`은 `${GATEWAY_JWT_SECRET}`, `${OIDC_CLIENT_SECRET}`, `${GATEWAY_POSTGRES_URL}`을 대신 참조합니다.
  </Step>

  <Step title="배포">
    <Tabs>
      <Tab title="Cloud Run">
        아래 명령어는 내부 로드 밸런서 뒤에 프로덕션용으로 배포합니다.

        ```bash theme={null}
        gcloud run deploy claude-gateway \
          --image="${REGION}-docker.pkg.dev/${PROJECT_ID}/claude-gateway/gateway:<version>" \
          --region="$REGION" \
          --service-account="claude-gateway@${PROJECT_ID}.iam.gserviceaccount.com" \
          --min-instances=1 \
          --timeout=3600 \
          --ingress=internal-and-cloud-load-balancing \
          --network="$VPC" --subnet=cc-gateway-subnet --vpc-egress=private-ranges-only \
          --set-secrets=/etc/claude/gateway.yaml=gateway-config:latest,GATEWAY_JWT_SECRET=gateway-jwt-secret:latest,OIDC_CLIENT_SECRET=gateway-oidc-client-secret:latest,GATEWAY_POSTGRES_URL=gateway-postgres-url:latest \
          --no-invoker-iam-check
        ```

        `--network`, `--subnet` 및 `--vpc-egress=private-ranges-only`를 통한 직접 VPC 이그레스로 서비스가 Cloud SQL 사설 IP에 직접 도달할 수 있습니다. Google Cloud Agent Platform 엔드포인트 및 `accounts.google.com`으로 향하는 공개 이그레스는 VPC를 통하지 않고 인터넷으로 직접 향하므로 Cloud NAT가 필요하지 않습니다.

        호출자(invoker) IAM 검사는 열려 있거나 비활성화되어야 합니다. 게이트웨이는 자체 OIDC를 실행하고 클라이언트는 GCP 토큰을 가지고 있지 않으므로 Cloud Run의 호출자 검사가 비인증 요청을 허용해야 합니다. 게이트웨이의 OIDC 로그인은 요청이 컨테이너에 도달하면 `allowed_email_domains`로 로그인 가능 도메인을 제한하며 인증을 수행합니다.

        두 가지 플래그가 비인증 요청을 허용합니다:

        * `--no-invoker-iam-check`: 관리할 `allUsers` 바인딩 없이 검사를 비활성화하며 도메인 제한 공유(Domain Restricted Sharing) 상태에서도 작동합니다.
        * `--allow-unauthenticated`: `allUsers`에 `run.invoker` 역할을 부여합니다. 조직에서 `--no-invoker-iam-check`를 허용하지 않는 경우 사용하세요.

        `--ingress`를 통한 인그레스 제한은 호출자 검사와 독립된 별도의 계층입니다. 기업 네트워크로 서비스를 제한하도록 계속 설정해 두세요.

        기본적으로 Cloud Run `*.run.app` URL은 공개 주소로 해소되어 `/login`의 [사설 네트워크 검사](/docs/en/claude-apps-gateway#prerequisites)에서 거부됩니다. 두 가지 토폴로지가 개발자에게 비공개로 해소 가능한 호스트 이름을 제공하며 Cloud Run은 둘 중 어느 것도 대신 프로비저닝해주지 않습니다:

        * **내부 애플리케이션 로드 밸런서** (위의 배포 명령어가 가정하는 토폴로지): `--ingress=internal-and-cloud-load-balancing`으로 배포하고, 내부 DNS 이름 및 인증서를 사용하여 서비스 앞에 내부 애플리케이션 로드 밸런서를 프로비저닝한 다음 `listen.public_url`을 해당 호스트 이름으로 설정합니다.
        * **로드 밸런서가 없는 내부 전용 인그레스**: `--ingress=internal`로 배포하고 `listen.public_url`을 `*.run.app` URL로 둡니다(아래 [참조 자산](#terraform-참조)의 기본값). `*.run.app`이 비공개로 해소되려면 네트워크 팀이 Google API용 Private Service Connect 엔드포인트, 해당 엔드포인트로 `*.run.app`을 해소하는 Cloud DNS 비공개 영역 및 온프레미스 라우팅을 이미 운영 중이어야 합니다.

        Google의 [Cloud Run 비공개 네트워크 가이드](https://cloud.google.com/run/docs/securing/private-networking)에서 두 옵션 모두 필요한 인프라를 다룹니다. 게이트웨이가 사설 호스트 이름에서 작동하기 시작하면 로그인을 검증하세요. 그때까지는 Cloud Run 로그에서 컨테이너 부팅을 확인하세요.

        첫 로그인 전에 OAuth 클라이언트의 승인된 리다이렉트 URI를 `<public_url>/oauth/callback`으로 업데이트하세요. 게이트웨이는 `X-Forwarded-Host` 및 `X-Forwarded-Proto`를 무시하고 오직 설정의 `public_url`에서만 공개 오리진을 구축하므로 `public_url`을 변경한 후 재배포하세요. `X-Forwarded-For`는 `listen.trusted_proxies`가 설정된 경우에만 클라이언트 IP로 적용됩니다.
      </Tab>

      <Tab title="GKE">
        파드가 데이터베이스의 사설 IP에 도달할 수 있도록 클러스터가 Cloud SQL 단계에서 생성된 `$VPC`에 위치해야 합니다. VPC 피어링만으로는 작동하지 않습니다. Cloud SQL 사설 IP 자체가 피어링된 네트워크이고 피어링은 전이되지 않기 때문입니다. 해당 VPC에 새 클러스터를 생성하려면 `gcloud container clusters create`에 `--network="$VPC" --subnetwork=cc-gateway-subnet`을 전달하세요.

        클러스터 및 노드 풀에 Workload Identity를 활성화한 다음 Google 서비스 계정을 Kubernetes 서비스 계정에 바인딩하여 파드가 자격 증명을 상속받도록 합니다:

        ```bash theme={null}
        gcloud container clusters update <cluster> --region="$REGION" \
          --workload-pool="${PROJECT_ID}.svc.id.goog"
        # Standard 클러스터의 경우 기존 노드 풀에 GKE_METADATA도 필요합니다.
        # Autopilot은 이를 기본적으로 활성화합니다.
        gcloud container node-pools update <pool> --cluster=<cluster> \
          --region="$REGION" --workload-metadata=GKE_METADATA

        kubectl create namespace claude-gateway
        kubectl create serviceaccount gateway -n claude-gateway

        gcloud iam service-accounts add-iam-policy-binding \
          "claude-gateway@${PROJECT_ID}.iam.gserviceaccount.com" \
          --role roles/iam.workloadIdentityUser \
          --member "serviceAccount:${PROJECT_ID}.svc.id.goog[claude-gateway/gateway]"

        kubectl annotate serviceaccount gateway -n claude-gateway \
          iam.gke.io/gcp-service-account="claude-gateway@${PROJECT_ID}.iam.gserviceaccount.com"
        ```

        [Kubernetes 배포](/docs/en/claude-apps-gateway-deploy#kubernetes)에 설명된 대로 게이트웨이를 표준 Deployment, Service 및 `gce-internal` 클래스의 내부 Ingress로 배포하세요:

        * `serviceAccountName: gateway`
        * Secret Manager CSI 드라이버가 `/secrets`에 비밀 값을 마운트
        * 준비성 프로브가 `GET /readyz`를 가리킴

        게이트웨이 Service에 상향 조정된 `timeoutSec`를 포함한 BackendConfig를 첨부하세요: GKE Ingress 뒷단의 로드 밸런서 백엔드 서비스는 기본 타임아웃이 30초로 설정되어 있어 긴 스트리밍 응답이 끊길 수 있습니다.

        Workload Identity 클러스터에서 `169.254.169.254`를 차단하는 이그레스 NetworkPolicy를 적용하지 마세요. 파드가 자격 증명을 구하기 위해 메타데이터 서버에 도달해야 합니다. 게이트웨이 내장 [SSRF 방어기능](/docs/en/claude-apps-gateway-deploy#threat-model-summary)이 해당 부분의 방어 역할을 담당합니다.

        게이트웨이는 메타데이터 엔드포인트에 도달 가능함을 알리는 부팅 경고를 기록하고 이그레스 NetworkPolicy 적용을 제안합니다. Workload Identity 환경에서는 파드에 해당 엔드포인트가 필요하므로 이 경고는 예상된 동작입니다.
      </Tab>
    </Tabs>
  </Step>

  <Step title="개발자 머신에 게이트웨이 URL 푸시">
    게이트웨이가 실행 중이지만 게이트웨이 URL이 개발자 머신에 설정될 때까지 개발자는 `/login`을 통해 도달할 수 없습니다. MDM을 통해 `forceLoginMethod`, `forceLoginGatewayUrl` 및 `parentSettingsBehavior: "merge"` 옵트인이 포함된 전체 [관리형 설정 스니펫](/docs/en/claude-apps-gateway#set-the-gateway-url)을 각 기기에 배포하세요. 개발자가 수동으로 선택할 수 있는 로그인 선택기 옵션은 없습니다.
  </Step>
</Steps>

## Terraform 참조

[참조 배포 자산](https://github.com/anthropics/claude-code/tree/main/examples/gateway/gcp)은 이 페이지의 Cloud Run 트랙을 자동화합니다. 설정 및 이미지 자산은 두 트랙 모두에 적용됩니다:

* `setup.sh`: API 활성화부터 첫 배포까지 전체 Cloud Run 경로를 안내하는 멱등한(idempotent) `gcloud` 프로비저너
* `terraform/`: 그린필드 배포를 위한 코드 기반 인프라 배포: Artifact Registry 리포지토리를 생성하는 대상을 지정한 적용 후 이미지 빌드 및 푸시, 이어서 전체 적용
* distroless 런타임 이미지용 `gateway.yaml.example` 및 `Dockerfile`

자산의 기본 Cloud Run 인그레스는 `internal`로 설정되어 있으므로 로드 밸런서가 필요하지 않습니다. 이 페이지의 프로덕션 ALB 뒷단 배포와 맞추려면 `INGRESS=internal-and-cloud-load-balancing`을 설정하여 `setup.sh`를 실행하거나 Terraform 변수 `ingress`를 `INGRESS_TRAFFIC_INTERNAL_LOAD_BALANCER`로 설정하세요. 자산은 또한 호출자 계층을 `--no-invoker-iam-check` 대신 `allUsers` `run.invoker` 부여로 기본 설정하는데, 이는 이 페이지 가이드의 반대 설정입니다. 둘 다 작동하며 선택은 조직의 정책 제약에 따라 다릅니다.

이 자산들은 지원되는 프로덕션 아티팩트가 아닌 작업 예제로 제공되므로 사용자의 환경에 맞게 검토하고 조정하세요.

## 문제 해결

게이트웨이 부팅 및 로그인 오류는 플랫폼에 구애받지 않는 [문제 해결 표](/docs/en/claude-apps-gateway-deploy#troubleshooting)를 참조하세요. 아래 항목은 Google Cloud 전용 내용입니다.

| 증상                                                                                                       | 원인                                                                                                                               | 해결 방법                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 컨테이너에 도달하기 전에 Cloud Run이 `403 Forbidden`을 반환함                                               | 호출자 IAM 검사가 여전히 활성화되어 있음                                                                                           | `--no-invoker-iam-check`로 배포하거나 `--allow-unauthenticated`로 `allUsers`에 `run.invoker` 역할을 부여하세요.                                                                                  |
| `--no-invoker-iam-check`가 `invoker_iam_disabled is not currently available`과 함께 거부됨                | `constraints/run.managed.requireInvokerIam`에 의해 차단됨                                                                          | `--allow-unauthenticated`를 사용하세요. `constraints/iam.allowedPolicyMemberDomains`를 통한 도메인 제한 공유도 이를 차단하는 경우, `allUsers` 바인딩 없이 네트워크 계층에서 게이트웨이를 노출하는 GKE 트랙을 사용하세요.    |
| 배포 시 `Container manifest type … must support amd64/linux` 발생                                          | 이미지가 amd64가 아닌 호스트에서 빌드되었거나 buildx가 OCI 이미지 인덱스를 내보냄                                                 | `--platform=linux/amd64 --provenance=false`로 빌드하세요.                                                                                                                                                                         |
| Cloud Run에서 게이트웨이 부팅이 Postgres 연결 타임아웃 오류와 함께 종료됨                                   | 서비스가 VPC에 연결되어 있지 않거나 Cloud SQL이 해당 VPC에 사설 IP가 없음. 저장소가 5초 후 대기를 중단함                           | Direct VPC 이그레스를 위해 `--network` 및 `--subnet`을 지정하여 배포하고, 동일한 VPC를 가리키는 `--no-assign-ip` 및 `--network`로 Cloud SQL 인스턴스를 생성하세요.                                                               |
| Google Cloud Agent Platform 요청이 `403 PERMISSION_DENIED`를 반환함                                        | 런타임이 `claude-gateway` 서비스 계정을 사용하고 있지 않거나 대상 지역의 프로젝트에 대해 Model Garden에서 모델이 활성화되지 않음  | Cloud Run에서 `--service-account`를 설정하거나 GKE에서 Workload Identity를 바인딩하고, 대상 지역의 Model Garden에서 각 Claude 모델을 활성화하세요.                                                                             |
| 고정된 시간이 지난 후 스트리밍 응답이 끊김                                                                 | 프런트엔드 요청 타임아웃: GKE Ingress 뒷단의 로드 밸런서 백엔드 서비스 기본값은 30초이고 Cloud Run 기본값은 300초임                | GKE에서는 서비스에 상향 조정된 `timeoutSec`를 가진 BackendConfig를 첨부하고, Cloud Run에서는 `--timeout=3600`으로 배포하세요.                                                                                                    |

## 다음 단계

* [설정 참조](/docs/en/claude-apps-gateway-config): `managed.policies` 및 `telemetry`를 포함한 모든 `gateway.yaml` 옵션
* [배포 및 운영](/docs/en/claude-apps-gateway-deploy): IdP 설정, 헬스 체크, JWT 비밀 값 순환, 업그레이드 및 보안 모델
* [Claude apps gateway 개요](/docs/en/claude-apps-gateway): 퀵스타트 및 개발자 연결
