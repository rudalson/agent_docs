> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude Code GitLab CI/CD

> GitLab CI/CD를 사용하여 개발 워크플로에 Claude Code를 통합하는 방법에 대해 알아봅니다.

<Info>
  GitLab CI/CD용 Claude Code는 현재 베타 버전입니다. 환경을 개선함에 따라 기능과 동작 방식이 변경될 수 있습니다.

  이 통합은 GitLab에서 유지관리합니다. 지원이 필요한 경우 다음 [GitLab issue](https://gitlab.com/gitlab-org/gitlab/-/issues/573776)를 참조하세요.
</Info>

<Note>
  이 통합은 [Claude Code CLI 및 Agent SDK](/docs/en/agent-sdk/overview)를 기반으로 구축되었으며, CI/CD 작업 및 커스텀 자동화 워크플로에서 Claude를 프로그래밍 방식으로 사용할 수 있도록 지원합니다.
</Note>

## 왜 GitLab과 함께 Claude Code를 사용해야 할까요?

* **즉각적인 MR 생성**: 필요한 내용을 설명하면 Claude가 변경 사항과 설명이 포함된 완전한 MR(Merge Request)을 제안합니다.
* **자동화된 구현**: 단일 명령어나 멘션만으로 이슈를 작동 가능한 코드로 변환합니다.
* **프로젝트 인식**: Claude는 프로젝트의 `CLAUDE.md` 가이드라인과 기존 코드 패턴을 준수합니다.
* **간단한 설정**: `.gitlab-ci.yml`에 하나의 작업을 추가하고 마스킹된 CI/CD 변수 하나만 설정하면 됩니다.
* **엔터프라이즈 지원**: Claude API, Amazon Bedrock 또는 Google Cloud's Agent Platform 중 선택하여 데이터 레지던시 및 조달 요구사항을 충족할 수 있습니다.
* **기본적인 보안성**: 브랜치 보호 및 승인 정책이 적용된 기존 GitLab 러너(runner) 내부에서 실행됩니다.

## 작동 방식

Claude Code는 GitLab CI/CD를 사용하여 격리된 작업에서 AI 태스크를 실행하고 그 결과를 MR을 통해 다시 커밋합니다:

1. **이벤트 기반 오케스트레이션**: GitLab은 지정된 트리거(예: 이슈, MR 또는 리뷰 스레드에서 `@claude`를 언급하는 댓글)를 감지합니다. 작업은 스레드 및 저장소에서 컨텍스트를 수집하고 해당 입력을 기반으로 프롬프트를 구성한 후 Claude Code를 실행합니다.

2. **프로바이더 추상화**: 환경에 맞는 프로바이더를 선택하여 사용할 수 있습니다:
   * Claude API (SaaS)
   * Amazon Bedrock (IAM 기반 액세스, 리전 간 옵션 지원)
   * Google Cloud's Agent Platform (GCP 기본 지원, Workload Identity Federation)

3. **샌드박스 실행**: 각 상호작용은 엄격한 네트워크 및 파일 시스템 규칙이 적용된 컨테이너 내에서 실행됩니다. Claude Code는 작업 공간 범위의 권한을 적용하여 쓰기 작업을 제한합니다. 모든 변경 사항은 MR을 거치므로 리뷰어가 변경 내역(diff)을 확인하고 기존 승인 절차를 그대로 유지할 수 있습니다.

리전별 엔드포인트를 선택하면 기존 클라우드 계약을 활용하는 동시에 지연 시간을 줄이고 데이터 주권 요구사항을 충족할 수 있습니다.

## Claude가 수행할 수 있는 작업

Claude Code는 코드로 작업하는 방식을 혁신하는 강력한 CI/CD 워크플로를 지원합니다:

* 이슈 설명이나 댓글을 바탕으로 MR 생성 및 업데이트
* 성능 저하(regression) 분석 및 최적화 방안 제안
* 브랜치에서 직접 기능을 구현한 뒤 MR 생성
* 테스트나 댓글에서 지적된 버그 및 회귀 문제 수정
* 요청된 변경 사항을 반복 개선하기 위해 추가 댓글에 응답

## 설정

### 빠른 설정

가장 빠르게 시작하는 방법은 `.gitlab-ci.yml`에 최소한의 작업을 추가하고 API 키를 마스킹된 변수로 설정하는 것입니다.

1. **마스킹된 CI/CD 변수 추가**
   * **Settings** → **CI/CD** → **Variables**로 이동합니다.
   * `ANTHROPIC_API_KEY`를 추가합니다(필요에 따라 마스킹 및 보호 설정).

2. **`.gitlab-ci.yml`에 Claude 작업 추가**

```yaml theme={null}
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  # 작업을 트리거할 방식에 맞게 규칙을 조정하세요:
  # - 수동 실행
  # - 병합 요청(MR) 이벤트
  # - 댓글에 '@claude'가 포함될 때 트리거되는 웹/API 트리거
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
  script:
    # 선택 사항: 구성 환경에서 제공하는 경우 GitLab MCP 서버를 시작합니다.
    - /bin/gitlab-mcp-server || true
    # 컨텍스트 페이로드가 있는 웹/API 트리거를 통해 호출할 때 AI_FLOW_* 변수를 사용합니다.
    - echo "$AI_FLOW_INPUT for $AI_FLOW_CONTEXT on $AI_FLOW_EVENT"
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Review this MR and implement the requested changes'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
```

작업 및 `ANTHROPIC_API_KEY` 변수를 추가한 후 **CI/CD** → **Pipelines**에서 작업을 수동으로 실행하여 테스트하거나, MR에서 트리거하여 Claude가 브랜치 변경 사항을 제안하고 필요한 경우 MR을 생성하도록 하세요.

<Note>
  Claude API 대신 Amazon Bedrock 또는 Google Cloud's Agent Platform에서 실행하려면 아래 [Amazon Bedrock 및 Google Cloud 사용](#using-with-amazon-bedrock-and-google-cloud) 섹션의 인증 및 환경 설정을 참조하세요.
</Note>

### 수동 설정 (프로덕션 환경 권장)

더 정밀하게 통제되는 설정이나 엔터프라이즈 프로바이더가 필요한 경우:

1. **프로바이더 액세스 구성**:
   * **Claude API**: `ANTHROPIC_API_KEY`를 생성하고 마스킹된 CI/CD 변수로 저장합니다.
   * **Amazon Bedrock**: **Configure GitLab** → **AWS OIDC** 설정을 진행하고 Amazon Bedrock용 IAM 역할을 생성합니다.
   * **Google Cloud's Agent Platform**: **Configure Workload Identity Federation for GitLab** → **GCP** 설정을 진행합니다.

2. **GitLab API 작업을 위한 프로젝트 자격 증명 추가**:
   * 기본적으로 `CI_JOB_TOKEN`을 사용하거나, `api` 스코프가 있는 Project Access Token을 생성합니다.
   * PAT를 사용하는 경우 `GITLAB_ACCESS_TOKEN`(마스킹 처리)으로 저장합니다.

3. **Claude 작업을 `.gitlab-ci.yml`에 추가** (아래 예제 참조)

4. **(선택 사항) 멘션 기반 트리거 활성화**:
   * 이벤트 리스너(사용하는 경우)에 "Comments (notes)"용 프로젝트 웹훅을 추가합니다.
   * 댓글에 `@claude`가 포함되어 있을 때 리스너가 `AI_FLOW_INPUT` 및 `AI_FLOW_CONTEXT`와 같은 변수를 사용하여 파이프라인 트리거 API를 호출하도록 구성합니다.

## 주요 사용 예시

### 이슈를 MR로 변환

이슈 댓글에서:

```text theme={null}
@claude implement this feature based on the issue description
```

Claude는 이슈와 코드를 분석하고 브랜치에 변경 사항을 작성한 후 검토를 위한 MR을 생성합니다.

### 구현 도움받기

MR 토론에서:

```text theme={null}
@claude suggest a concrete approach to cache the results of this API call
```

Claude는 변경 사항을 제안하고 적절한 캐싱 코드를 추가하여 MR을 업데이트합니다.

### 빠른 버그 수정

이슈 또는 MR 댓글에서:

```text theme={null}
@claude fix the TypeError in the user dashboard component
```

Claude는 버그 위치를 찾아 수정을 수행하고 브랜치를 업데이트하거나 새 MR을 생성합니다.

## Amazon Bedrock 및 Google Cloud 사용

엔터프라이즈 환경에서는 동일한 개발자 경험을 유지하면서 클라우드 인프라 내에서 Claude Code를 완전히 실행할 수 있습니다.

<Tabs>
  <Tab title="Amazon Bedrock">
    ### 사전 요구사항

    Amazon Bedrock과 함께 Claude Code를 설정하기 전에 다음 항목이 필요합니다:

    1. 원하는 Claude 모델에 대한 Amazon Bedrock 액세스 권한이 있는 AWS 계정
    2. AWS IAM에서 OIDC 자격 증명 제공자로 구성된 GitLab
    3. Amazon Bedrock 권한이 있으며 GitLab 프로젝트/참조(ref)로 제한된 신뢰 정책을 가진 IAM 역할
    4. 역할 인수를 위한 GitLab CI/CD 변수:
       * `AWS_ROLE_TO_ASSUME` (역할 ARN)
       * `AWS_REGION` (Amazon Bedrock 리전)

    ### 설정 방법

    GitLab CI 작업이 정적 키 없이 OIDC를 통해 IAM 역할을 맡을(assume) 수 있도록 AWS를 구성합니다.

    **필수 설정:**

    1. Amazon Bedrock을 활성화하고 대상 Claude 모델에 대한 액세스를 요청합니다.
    2. 아직 없는 경우 GitLab용 IAM OIDC 제공자를 생성합니다.
    3. GitLab OIDC 제공자가 신뢰하고 프로젝트 및 보호된 참조로 제한된 IAM 역할을 생성합니다.
    4. Amazon Bedrock 호출 API에 대한 최소 권한을 부여합니다.

    **CI/CD 변수에 저장해야 하는 필수 값:**

    * `AWS_ROLE_TO_ASSUME`
    * `AWS_REGION`

    Settings → CI/CD → Variables에 변수를 추가합니다:

    ```yaml theme={null}
    # Amazon Bedrock 용:
    - AWS_ROLE_TO_ASSUME
    - AWS_REGION
    ```

    위의 Amazon Bedrock 작업 예시를 활용해 실행 시점에 GitLab 작업 토큰을 임시 AWS 자격 증명으로 교환하세요.
  </Tab>

  <Tab title="Google Cloud's Agent Platform">
    ### 사전 요구사항

    Google Cloud's Agent Platform과 함께 Claude Code를 설정하기 전에 다음 항목이 필요합니다:

    1. 다음 설정이 완료된 Google Cloud 프로젝트:
       * Google Cloud's Agent Platform API 활성화
       * GitLab OIDC를 신뢰하도록 설정된 Workload Identity Federation
    2. 필요한 Google Cloud's Agent Platform 역할만 부여된 전용 서비스 계정
    3. WIF용 GitLab CI/CD 변수:
       * `GCP_WORKLOAD_IDENTITY_PROVIDER` (전체 리소스 이름)
       * `GCP_SERVICE_ACCOUNT` (서비스 계정 이메일)

    ### 설정 방법

    GitLab CI 작업이 Workload Identity Federation을 통해 서비스 계정을 가장(impersonate)할 수 있도록 Google Cloud를 구성합니다.

    **필수 설정:**

    1. IAM Credentials API, STS API 및 Google Cloud's Agent Platform API를 활성화합니다.
    2. GitLab OIDC용 Workload Identity Pool 및 제공자를 생성합니다.
    3. Google Cloud's Agent Platform 역할이 부여된 전용 서비스 계정을 생성합니다.
    4. WIF 보안 주체(principal)에 서비스 계정을 가장할 수 있는 권한을 부여합니다.

    **CI/CD 변수에 저장해야 하는 필수 값:**

    * `GCP_WORKLOAD_IDENTITY_PROVIDER`
    * `GCP_SERVICE_ACCOUNT`

    Settings → CI/CD → Variables에 변수를 추가합니다:

    ```yaml theme={null}
    # Google Cloud's Agent Platform 용:
    - GCP_WORKLOAD_IDENTITY_PROVIDER
    - GCP_SERVICE_ACCOUNT
    - CLOUD_ML_REGION (예: us-east5)
    ```

    키를 저장하지 않고 인증하려면 위의 Google Cloud's Agent Platform 작업 예시를 사용하세요.
  </Tab>
</Tabs>

## 구성 예제

아래는 파이프라인에 바로 적용할 수 있는 구성 스니펫입니다.

### 기본 .gitlab-ci.yml (Claude API)

```yaml theme={null}
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
  script:
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Summarize recent changes and suggest improvements'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
  # Claude Code는 CI/CD 변수에서 ANTHROPIC_API_KEY를 사용합니다.
```

### Amazon Bedrock 작업 예시 (OIDC)

**사전 요구사항:**

* 선택한 Claude 모델에 대한 액세스 권한이 부여된 Amazon Bedrock 활성화
* GitLab 프로젝트 및 참조를 신뢰하는 역할로 AWS에 구성된 GitLab OIDC
* Amazon Bedrock 권한이 있는 IAM 역할 (최소 권한 권장)

**필수 CI/CD 변수:**

* `AWS_ROLE_TO_ASSUME`: Amazon Bedrock 액세스를 위한 IAM 역할의 ARN
* `AWS_REGION`: Amazon Bedrock 리전 (예: `us-west-2`)

```yaml theme={null}
claude-bedrock:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
  before_script:
    - apk add --no-cache bash curl jq git python3 py3-pip
    - pip install --no-cache-dir awscli
    - curl -fsSL https://claude.ai/install.sh | bash
    # GitLab OIDC 토큰을 AWS 자격 증명으로 교환
    - export AWS_WEB_IDENTITY_TOKEN_FILE="${CI_JOB_JWT_FILE:-/tmp/oidc_token}"
    - if [ -n "${CI_JOB_JWT_V2}" ]; then printf "%s" "$CI_JOB_JWT_V2" > "$AWS_WEB_IDENTITY_TOKEN_FILE"; fi
    - >
      aws sts assume-role-with-web-identity
      --role-arn "$AWS_ROLE_TO_ASSUME"
      --role-session-name "gitlab-claude-$(date +%s)"
      --web-identity-token "file://$AWS_WEB_IDENTITY_TOKEN_FILE"
      --duration-seconds 3600 > /tmp/aws_creds.json
    - export AWS_ACCESS_KEY_ID="$(jq -r .Credentials.AccessKeyId /tmp/aws_creds.json)"
    - export AWS_SECRET_ACCESS_KEY="$(jq -r .Credentials.SecretAccessKey /tmp/aws_creds.json)"
    - export AWS_SESSION_TOKEN="$(jq -r .Credentials.SessionToken /tmp/aws_creds.json)"
  script:
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Implement the requested changes and open an MR'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
  variables:
    AWS_REGION: "us-west-2"
```

<Note>
  Amazon Bedrock용 모델 ID에는 리전별 접두사(예: `us.anthropic.claude-sonnet-4-6`)가 포함됩니다. 워크플로가 지원하는 경우 작업 구성 또는 프롬프트를 통해 원하는 모델을 전달하세요.
</Note>

### Agent Platform 작업 예시 (Workload Identity Federation)

**사전 요구사항:**

* GCP 프로젝트에서 Google Cloud's Agent Platform API 활성화
* GitLab OIDC를 신뢰하도록 구성된 Workload Identity Federation
* Google Cloud's Agent Platform 권한이 있는 서비스 계정

**필수 CI/CD 변수:**

* `GCP_WORKLOAD_IDENTITY_PROVIDER`: 전체 제공자 리소스 이름
* `GCP_SERVICE_ACCOUNT`: 서비스 계정 이메일
* `CLOUD_ML_REGION`: Google Cloud's Agent Platform 리전 (예: `us-east5`)

```yaml theme={null}
claude-vertex:
  stage: ai
  image: gcr.io/google.com/cloudsdktool/google-cloud-cli:slim
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
  before_script:
    - apt-get update && apt-get install -y git && apt-get clean
    - curl -fsSL https://claude.ai/install.sh | bash
    # WIF를 통해 Google Cloud에 인증 (키 다운로드 없음)
    - >
      gcloud auth login --cred-file=<(cat <<EOF
      {
        "type": "external_account",
        "audience": "${GCP_WORKLOAD_IDENTITY_PROVIDER}",
        "subject_token_type": "urn:ietf:params:oauth:token-type:jwt",
        "service_account_impersonation_url": "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/${GCP_SERVICE_ACCOUNT}:generateAccessToken",
        "token_url": "https://sts.googleapis.com/v1/token"
      }
      EOF
      )
    - gcloud config set project "$(gcloud projects list --format='value(projectId)' --filter="name:${CI_PROJECT_NAMESPACE}" | head -n1)" || true
  script:
    - /bin/gitlab-mcp-server || true
    - >
      CLOUD_ML_REGION="${CLOUD_ML_REGION:-us-east5}"
      claude
      -p "${AI_FLOW_INPUT:-'Review and update code as requested'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
  variables:
    CLOUD_ML_REGION: "us-east5"
```

<Note>
  Workload Identity Federation을 사용하면 서비스 계정 키를 저장할 필요가 없습니다. 저장소별 신뢰 조건 및 최소 권한 서비스 계정을 이용하세요.
</Note>

## 권장 사항

### CLAUDE.md 설정

저장소 루트에 `CLAUDE.md` 파일을 생성하여 코딩 표준, 리뷰 기준 및 프로젝트 규칙을 정의하세요. Claude는 실행 중에 이 파일을 읽고 변경 사항을 제안할 때 규칙을 따릅니다.

### 보안 고려 사항

**API 키나 클라우드 자격 증명을 저장소에 절대로 커밋하지 마세요**. 항상 GitLab CI/CD 변수를 사용하세요:

* `ANTHROPIC_API_KEY`를 마스킹된 변수로 추가 (필요시 보호 설정)
* 가능한 경우 프로바이더 전용 OIDC 사용 (장기 키 지양)
* 작업 권한 및 외부 네트워크 출력 제한
* 다른 기여자의 코드와 마찬가지로 Claude의 MR을 검토

### 성능 최적화

* `CLAUDE.md`를 명확하고 간결하게 유지
* 불필요한 상호작용을 줄이기 위해 명확한 이슈/MR 설명 제공
* 이탈하는 실행을 방지하기 위해 적절한 작업 타임아웃 구성
* 가능한 경우 러너에서 npm 및 패키지 설치 캐싱 사용

### CI 비용

GitLab CI/CD와 함께 Claude Code를 사용할 때 관련 비용을 유의하세요:

* **GitLab Runner 시간**:
  * Claude는 GitLab 러너에서 실행되며 컴퓨팅 시간을 소비합니다.
  * 자세한 내용은 GitLab 요금제의 러너 청구 정책을 참조하세요.

* **API 비용**:
  * 각 Claude 상호작용은 프롬프트 및 응답 크기에 따라 토큰을 소비합니다.
  * 토큰 사용량은 작업 복잡도 및 코드베이스 크기에 따라 달라집니다.
  * 자세한 내용은 [Anthropic 요금 안내](https://platform.claude.com/docs/en/about-claude/pricing)를 참조하세요.

* **비용 절감 팁**:
  * 불필요한 대화를 줄이기 위해 명확한 `@claude` 명령어 사용
  * 적절한 `max_turns` 및 작업 타임아웃 값 설정
  * 병렬 실행을 제어하기 위해 동시 실행 수 제한

## 보안 및 거버넌스

* 각 작업은 제한된 네트워크 액세스를 가진 격리된 컨테이너에서 실행됩니다.
* Claude의 변경 사항은 MR을 거치므로 리뷰어가 모든 diff를 확인할 수 있습니다.
* AI가 생성한 코드에도 브랜치 보호 및 승인 규칙이 적용됩니다.
* Claude Code는 쓰기 권한을 제한하기 위해 작업 공간 범위 권한을 사용합니다.
* 보유한 프로바이더 자격 증명을 직접 사용하므로 비용을 제어할 수 있습니다.

## 문제 해결

### Claude가 @claude 명령에 응답하지 않는 경우

* 파이프라인이 트리거되고 있는지 확인하세요 (수동, MR 이벤트 또는 노트 이벤트 리스너/웹훅 경유).
* CI/CD 변수(`ANTHROPIC_API_KEY` 또는 클라우드 프로바이더 설정)가 존재하며 마스킹 해제되어 있는지 확인하세요.
* 댓글에 (`/claude`가 아닌) `@claude`가 포함되어 있고 멘션 트리거가 구성되어 있는지 확인하세요.

### 작업이 댓글을 남기거나 MR을 생성하지 못하는 경우

* `CI_JOB_TOKEN`에 프로젝트에 대한 충분한 권한이 있는지 확인하거나, `api` 스코프의 Project Access Token을 사용하세요.
* `--allowedTools`에 `mcp__gitlab` 도구가 활성화되어 있는지 확인하세요.
* 작업이 MR의 컨테이너 내에서 실행되거나 `AI_FLOW_*` 변수를 통해 충분한 컨테Context를 확보했는지 확인하세요.

### 인증 오류

* **Claude API 사용 시**: `ANTHROPIC_API_KEY`가 유효하고 만료되지 않았는지 확인하세요.
* **Amazon Bedrock 또는 Google Cloud's Agent Platform 사용 시**: OIDC/WIF 구성, 역할 가장 및 보안 비밀 이름을 검증하고, 리전 및 모델 사용 가능 여부를 확인하세요.

## 고급 구성

### 공통 매개변수 및 변수

Claude Code는 자주 사용되는 다음과 같은 입력을 지원합니다:

* `prompt` / `prompt_file`: 인라인(`-p`) 또는 파일을 통해 지침 제공
* `max_turns`: 주고받는 대화 횟수 제한
* `timeout_minutes`: 전체 실행 시간 제한
* `ANTHROPIC_API_KEY`: Claude API에 필요 (Amazon Bedrock 또는 Google Cloud's Agent Platform에서는 사용 안 함)
* 프로바이더 전용 환경: `AWS_REGION`, Google Cloud's Agent Platform용 프로젝트/리전 변수

<Note>
  정확한 플래그 및 매개변수는 `@anthropic-ai/claude-code` 버전마다 다를 수 있습니다. 작업 환경에서 `claude --help`를 실행하여 지원되는 옵션을 확인하세요.
</Note>

### Claude 동작 맞춤 설정

두 가지 주된 방식으로 Claude를 가이드할 수 있습니다:

1. **CLAUDE.md**: 코딩 표준, 보안 요구사항 및 프로젝트 규칙을 정의합니다. Claude는 실행 중에 이를 읽고 지침을 따릅니다.
2. **커스텀 프롬프트**: 작업의 `prompt`/`prompt_file`을 통해 특정 태스크 관련 지침을 전달합니다. 작업 유형별(예: 리뷰, 구현, 리팩토링)로 서로 다른 프롬프트를 사용하세요.
