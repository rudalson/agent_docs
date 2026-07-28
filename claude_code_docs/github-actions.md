> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude Code GitHub Actions

> Claude Code GitHub Actions를 통해 개발 워크플로우에 Claude Code를 연동하는 방법을 알아보세요.

Claude Code GitHub Actions는 AI 기반 자동화를 GitHub 워크플로우에 제공합니다. PR이나 이슈에서 간단히 `@claude`를 멘션하면 Claude가 코드를 분석하고, 풀 리퀘스트(PR)를 생성하며, 기능을 구현하고, 버그를 수정합니다. 이 모든 작업은 프로젝트의 표준을 지키며 수행됩니다. 트리거 없이 모든 PR에 자동으로 게시되는 리뷰를 원하시면 [GitHub Code Review](/docs/en/code-review)를 참조하세요.

<Note>
  Claude Code GitHub Actions는 Claude Code를 애플리케이션에 프로그래밍 방식으로 연동할 수 있는 [Claude Agent SDK](/docs/en/agent-sdk/overview)를 기반으로 구축되었습니다. SDK를 사용하여 GitHub Actions 외에도 커스텀 자동화 워크플로우를 구축할 수 있습니다.
</Note>

## Claude Code GitHub Actions를 사용하는 이유

* **즉각적인 PR 생성**: 필요한 내용을 설명하면 Claude가 필요한 모든 변경 사항이 포함된 완전한 PR을 생성합니다.
* **자동화된 코드 구현**: 단일 명령으로 이슈를 실제 작동하는 코드로 변환합니다.
* **프로젝트 표준 준수**: Claude는 프로젝트의 `CLAUDE.md` 가이드라인과 기존 코드 패턴을 준수합니다.
* **간단한 설정**: 설치 프로그램과 API 키를 사용하여 몇 분 만에 시작하세요.
* **기본적으로 안전함**: 코드는 GitHub 러너 내에 유지됩니다.

## Claude가 할 수 있는 일

Claude Code는 코드 작업 방식을 혁신하는 강력한 GitHub Action을 제공합니다.

### Claude Code Action

이 GitHub Action을 사용하면 GitHub Actions 워크플로우 내에서 Claude Code를 실행할 수 있습니다. 이를 통해 Claude Code를 기반으로 다양한 커스텀 워크플로우를 구축할 수 있습니다.

[리포지토리 보기 →](https://github.com/anthropics/claude-code-action)

## 설정

## 빠른 설정

Claude Code 터미널에서 `/install-github-app`을 실행하여 대화형으로 연동을 설정하세요. 이 명령은 리포지토리에 Claude GitHub App을 설치하고 GitHub Actions 워크플로우와 API 키 시크릿을 추가하는 과정을 안내합니다.

GitHub App이 설치된 후 명령이 GitHub Actions 설정을 계속할지 묻습니다. Claude Code v2.1.187 이상에서는 **Skip for now**를 선택하여 App만 설치된 상태로 멈추고, 나중에 `/install-github-app`을 다시 실행하여 워크플로우 및 시크릿 단계로 돌아올 수 있습니다. 이전 버전에서는 워크플로우 선택으로 바로 진행됩니다.

<Note>
  * GitHub app을 설치하고 시크릿을 추가하려면 리포지토리 관리자여야 합니다.
  * GitHub app은 Contents, Issues, Pull requests에 대한 읽기 및 쓰기 권한을 요청합니다.
  * 이 퀵스타트 방식은 직접 Claude API를 사용하는 사용자에게만 제공됩니다. Amazon Bedrock이나 Google Cloud Agent Platform을 사용하는 경우 [Amazon Bedrock 및 Google Cloud와 함께 사용하기](#using-with-amazon-bedrock-and-google-cloud) 섹션을 참조하세요.
</Note>

## 수동 설정

`/install-github-app` 명령이 실패하거나 수동 설정을 선호하는 경우 아래의 수동 설정 지침을 따르세요.

1. 리포지토리에 **Claude GitHub app 설치**: [https://github.com/apps/claude](https://github.com/apps/claude)

   Claude GitHub app은 다음 리포지토리 권한이 필요합니다.

   * **Contents**: Read & write (리포지토리 파일 수정용)
   * **Issues**: Read & write (이슈에 응답용)
   * **Pull requests**: Read & write (PR 생성 및 변경 사항 푸시용)

   보안 및 권한에 대한 자세한 내용은 [보안 문서](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)를 참조하세요.
2. 리포지토리 시크릿에 **ANTHROPIC\_API\_KEY 추가** ([GitHub Actions에서 시크릿 사용법 알아보기](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions))
3. [examples/claude.yml](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml)의 **워크플로우 파일을 복사**하여 리포지토리의 `.github/workflows/` 디렉토리에 넣습니다.

<Tip>
  퀵스타트 또는 수동 설정을 완료한 후 이슈나 PR 주석에서 `@claude`를 태그하여 액션을 테스트해 보세요.
</Tip>

## Beta 버전에서 업그레이드하기

<Warning>
  Claude Code GitHub Actions v1.0은 하위 호환성을 깨뜨리는 변경 사항(breaking changes)을 포함하므로 베타 버전에서 v1.0으로 업그레이드하려면 워크플로우 파일을 업데이트해야 합니다.
</Warning>

현재 Claude Code GitHub Actions 베타 버전을 사용하는 경우 GA 버전으로 워크플로우를 업데이트하는 것이 좋습니다. 새 버전은 구성을 단순화하면서 자동 모드 감지와 같은 강력한 기능을 추가합니다.

### 필수 변경 사항

모든 베타 사용자는 업그레이드를 위해 워크플로우 파일에 다음 변경 사항을 적용해야 합니다.

1. **액션 버전 업데이트**: `@beta`를 `@v1`로 변경
2. **모드 구성 제거**: `mode: "tag"` 또는 `mode: "agent"` 삭제 (이제 자동 감지됨)
3. **프롬프트 입력 업데이트**: `direct_prompt`를 `prompt`로 교체
4. **CLI 옵션 이동**: `max_turns`, `model`, `custom_instructions` 등을 `claude_args`로 변환

### 레거시 레퍼런스 및 변경 사항

| 이전 Beta 입력 | 새 v1.0 입력 |
| :--- | :--- |
| `mode` | *(제거됨 - 자동 감지)* |
| `direct_prompt` | `prompt` |
| `override_prompt` | GitHub 변수가 포함된 `prompt` |
| `custom_instructions` | `claude_args: --append-system-prompt` |
| `max_turns` | `claude_args: --max-turns` |
| `model` | `claude_args: --model` |
| `allowed_tools` | `claude_args: --allowedTools` |
| `disallowed_tools` | `claude_args: --disallowedTools` |
| `claude_env` | `settings` JSON 형식 |

### 이전 및 이후 예시

**Beta 버전:**

```yaml theme={null}
- uses: anthropics/claude-code-action@beta
  with:
    mode: "tag"
    direct_prompt: "Review this PR for security issues"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    custom_instructions: "Follow our coding standards"
    max_turns: "10"
    model: "claude-sonnet-5"
```

**GA 버전 (v1.0):**

```yaml theme={null}
- uses: anthropics/claude-code-action@v1
  with:
    prompt: "Review this PR for security issues"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --append-system-prompt "Follow our coding standards"
      --max-turns 10
      --model claude-sonnet-5
```

<Tip>
  이제 액션이 구성에 따라 대화형 모드(`@claude` 멘션에 응답)로 실행할지, 자동화 모드(프롬프트로 즉시 실행)로 실행할지 자동으로 감지합니다.
</Tip>

## 사용 예시

Claude Code GitHub Actions는 다양한 작업을 지원할 수 있습니다. [예시 디렉토리](https://github.com/anthropics/claude-code-action/tree/main/examples)에는 시나리오별로 바로 사용할 수 있는 워크플로우가 포함되어 있습니다.

### 기본 워크플로우

```yaml theme={null}
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # 주석의 @claude 멘션에 응답함
```

### 스킬(Skills) 사용하기

`prompt` 입력은 일반 텍스트뿐만 아니라 [스킬](/docs/en/skills) 호출도 허용합니다.

* 리포지토리의 `.claude/skills/` 디렉토리에 있는 스킬의 경우, 액션 단계 전에 `actions/checkout`을 실행하고 `/skill-name`을 전달하세요.
* 플러그인에 포함된 스킬의 경우, `plugin_marketplaces` 및 `plugins` 입력으로 플러그인을 설치하고 네임스페이스가 지정된 `/plugin-name:skill-name`을 전달하세요.

다음 워크플로우는 `code-review` 플러그인을 설치하고 신규 또는 업데이트된 각 풀 리퀘스트에서 해당 스킬을 실행합니다.

```yaml theme={null}
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          plugin_marketplaces: "https://github.com/anthropics/claude-code.git"
          plugins: "code-review@claude-code-plugins"
          prompt: "/code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

### 프롬프트를 통한 커스텀 자동화

```yaml theme={null}
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"
jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Generate a summary of yesterday's commits and open issues"
          claude_args: "--model opus"
```

### 일반적인 사용 사례

이슈나 PR 주석에서:

```text wrap theme={null}
@claude 이슈 설명을 바탕으로 이 기능을 구현해 줘
@claude 이 엔드포인트에 대한 사용자 인증을 어떻게 구현해야 할까?
@claude 사용자 대시보드 컴포넌트의 TypeError를 수정해 줘
```

Claude가 자동으로 컨텍스트를 분석하고 적절하게 응답합니다.

## 모범 사례

### CLAUDE.md 구성

리포지토리 루트에 `CLAUDE.md` 파일을 작성하여 코드 스타일 가이드, 리뷰 기준, 프로젝트 전용 규칙, 선호하는 패턴을 정의하세요. 이 파일은 Claude가 프로젝트 표준을 이해하는 데 도움이 됩니다.

### 보안 고려 사항

<Warning>API 키를 리포지토리에 직접 커밋하지 마세요.</Warning>

권한, 인증 및 모범 사례를 포함한 종합적인 보안 가이드는 [Claude Code Action 보안 문서](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)를 참조하세요.

API 키에는 항상 GitHub Secrets를 사용하세요.

* `ANTHROPIC_API_KEY`라는 이름의 리포지토리 시크릿으로 API 키를 추가하세요.
* 워크플로우에서 해당 시크릿을 참조하세요: `anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}`
* 액션 권한을 필요한 범위로 제한하세요.
* 병합 전에 Claude의 제안 사항을 검토하세요.

워크플로우 파일에 API 키를 직접 하드코딩하지 말고 항상 GitHub Secrets(예: `${{ secrets.ANTHROPIC_API_KEY }}`)를 사용하세요.

### 성능 최적화

이슈 템플릿을 사용하여 컨텍스트를 제공하고, `CLAUDE.md`를 간결하고 집중되게 유지하며, 워크플로우에 적절한 타임아웃을 구성하세요.

### CI 비용

Claude Code GitHub Actions를 사용할 때 관련된 비용에 유의하세요.

**GitHub Actions 비용:**

* Claude Code는 GitHub 호스팅 러너에서 실행되므로 GitHub Actions 분을 소비합니다.
* 세부 가격 및 사용 가능 분 한도는 [GitHub 결제 문서](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions)를 참조하세요.

**API 비용:**

* 각 Claude 상호작용은 프롬프트와 응답 길이에 따라 API 토큰을 소비합니다.
* 토큰 사용량은 작업 복잡성과 코드베이스 크기에 따라 다릅니다.
* 현재 토큰 요금은 [Claude 가격 페이지](https://claude.com/platform/api)를 참조하세요.

**비용 최적화 팁:**

* 구체적인 `@claude` 명령을 사용하여 불필요한 API 호출을 줄이세요.
* 과도한 반복을 방지하기 위해 `claude_args`에서 적절한 `--max-turns`를 구성하세요.
* 무한 실행을 방지하기 위해 워크플로우 수준의 타임아웃을 설정하세요.
* 병렬 실행을 제한하기 위해 GitHub의 동시성 제어를 사용하는 것을 고려하세요.

## 구성 예시

Claude Code Action v1은 통합된 파라미터로 구성을 단순화합니다.

```yaml theme={null}
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Your instructions here" # 선택 사항
    claude_args: "--max-turns 5" # 선택 사항 CLI 인수
```

주요 기능:

* **통합된 프롬프트 인터페이스** - 모든 지침에 `prompt` 사용
* **스킬(Skills)** - 프롬프트에서 설치된 [스킬](/docs/en/skills)을 직접 호출
* **CLI 패스스루** - `claude_args`를 통한 임의의 Claude Code CLI 인수 전달
* **유연한 트리거** - 임의의 GitHub 이벤트와 작동

전체 워크플로우 파일은 [예시 디렉토리](https://github.com/anthropics/claude-code-action/tree/main/examples)를 방문하세요.

<Tip>
  이슈나 PR 주석에 응답할 때 Claude는 @claude 멘션에 자동으로 응답합니다. 다른 이벤트의 경우 `prompt` 파라미터를 사용하여 지침을 제공하세요.
</Tip>

## Amazon Bedrock 및 Google Cloud와 함께 사용하기

엔터프라이즈 환경에서는 자체 클라우드 인프라와 함께 Claude Code GitHub Actions를 사용할 수 있습니다. 이 방식을 통해 동일한 기능을 유지하면서 데이터 레지던시 및 청구를 완벽하게 제어할 수 있습니다.

### 사전 요구 사항

클라우드 공급자와 함께 Claude Code GitHub Actions를 설정하기 전에 다음 항목이 필요합니다.

#### Google Cloud Agent Platform의 경우:

1. Google Cloud Agent Platform이 활성화된 Google Cloud 프로젝트
2. GitHub Actions용으로 구성된 Workload Identity Federation
3. 필요한 권한이 있는 서비스 계정
4. GitHub App (권장) 또는 기본 GITHUB\_TOKEN 사용

#### Amazon Bedrock의 경우:

1. Amazon Bedrock이 활성화된 AWS 계정
2. AWS에 구성된 GitHub OIDC Identity Provider
3. Amazon Bedrock 권한이 있는 IAM 역할
4. GitHub App (권장) 또는 기본 GITHUB\_TOKEN 사용

<Steps>
  <Step title="커스텀 GitHub App 생성 (3P 공급자에 권장)">
    Google Cloud Agent Platform이나 Amazon Bedrock과 같은 3P 공급자를 사용할 때 최상의 제어력과 보안을 위해 자체 GitHub App을 생성하는 것을 권장합니다.

    1. [https://github.com/settings/apps/new](https://github.com/settings/apps/new)에 접속합니다.
    2. 기본 정보를 입력합니다.
       * **GitHub App name**: 고유한 이름을 선택합니다 (예: "YourOrg Claude Assistant")
       * **Homepage URL**: 조직의 웹사이트나 리포지토리 URL
    3. 앱 설정을 구성합니다.
       * **Webhooks**: "Active" 체크 해제 (이 연동에는 필요하지 않음)
    4. 필요한 권한을 설정합니다.
       * **Repository permissions**:
         * Contents: Read & Write
         * Issues: Read & Write
         * Pull requests: Read & Write
    5. "Create GitHub App"을 클릭합니다.
    6. 생성 후 "Generate a private key"를 클릭하고 다운로드된 `.pem` 파일을 저장합니다.
    7. 앱 설정 페이지에서 App ID를 기록해 둡니다.
    8. 리포지토리에 앱을 설치합니다.
       * 앱 설정 페이지의 왼쪽 사이드바에서 "Install App"을 클릭합니다.
       * 계정이나 조직을 선택합니다.
       * "Only select repositories"를 선택하고 특정 리포지토리를 선택합니다.
       * "Install"을 클릭합니다.
    9. 리포지토리 시크릿에 개인 키를 추가합니다.
       * 리포지토리의 Settings → Secrets and variables → Actions로 이동합니다.
       * `.pem` 파일의 내용으로 `APP_PRIVATE_KEY`라는 새 시크릿을 생성합니다.
    10. App ID를 시크릿으로 추가합니다.

    * GitHub App ID로 `APP_ID`라는 새 시크릿을 생성합니다.

    <Note>
      이 앱은 워크플로우에서 인증 토큰을 생성하기 위해 [actions/create-github-app-token](https://github.com/actions/create-github-app-token) 액션과 함께 사용됩니다.
    </Note>

    **Claude API를 사용하거나 자체 GitHub App을 설정하고 싶지 않은 경우 대안**: 공식 Anthropic 앱을 사용하세요.

    1. 설치 링크: [https://github.com/apps/claude](https://github.com/apps/claude)
    2. 인증에 필요한 추가 구성 없음
  </Step>

  <Step title="클라우드 공급자 인증 구성">
    클라우드 공급자를 선택하고 안전한 인증을 설정하세요.

    <AccordionGroup>
      <Accordion title="Amazon Bedrock">
        **자격 증명을 저장하지 않고 GitHub Actions가 안전하게 인증할 수 있도록 AWS를 구성합니다.**

        > **보안 알림**: 리포지토리 전용 구성을 사용하고 최소한으로 필요한 권한만 부여하세요.

        **필수 설정**:

        1. **Amazon Bedrock 활성화**:
           * Amazon Bedrock에서 Claude 모델 접근 권한 요청
           * 교차 리전 모델의 경우 모든 필요한 리전에서 접근 권한 요청

        2. **GitHub OIDC Identity Provider 설정**:
           * Provider URL: `https://token.actions.githubusercontent.com`
           * Audience: `sts.amazonaws.com`

        3. **GitHub Actions용 IAM 역할 생성**:
           * Trusted entity type: Web identity
           * Identity provider: `token.actions.githubusercontent.com`
           * Permissions: `AmazonBedrockFullAccess` 정책
           * 특정 리포지토리에 맞춰 신뢰 정책 구성

        **필요한 값**:

        설정 후 다음 정보가 필요합니다.

        * **AWS\_ROLE\_TO\_ASSUME**: 생성한 IAM 역할의 ARN

        <Tip>
          OIDC는 자격 증명이 일시적이며 자동으로 순환되므로 고정 AWS 접근 키를 사용하는 것보다 안전합니다.
        </Tip>

        자세한 OIDC 설정 지침은 [AWS 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html)를 참조하세요.
      </Accordion>

      <Accordion title="Google Cloud Agent Platform">
        **자격 증명을 저장하지 않고 GitHub Actions가 안전하게 인증할 수 있도록 Google Cloud를 구성합니다.**

        > **보안 알림**: 리포지토리 전용 구성을 사용하고 최소한으로 필요한 권한만 부여하세요.

        **필수 설정**:

        1. Google Cloud 프로젝트에서 **API 활성화**:
           * IAM Credentials API
           * Security Token Service (STS) API
           * Google Cloud Agent Platform API

        2. **Workload Identity Federation 리소스 생성**:
           * Workload Identity Pool 생성
           * 다음 정보를 사용하여 GitHub OIDC 공급자 추가:
             * Issuer: `https://token.actions.githubusercontent.com`
             * 리포지토리 및 소유자에 대한 속성 매핑
             * **보안 권장 사항**: 리포지토리 전용 속성 조건 사용

        3. **서비스 계정 생성**:
           * `Vertex AI User` 역할만 부여
           * **보안 권장 사항**: 리포지토리마다 전용 서비스 계정 생성

        4. **IAM 바인딩 구성**:
           * Workload Identity Pool이 서비스 계정을 가장할 수 있도록 허용
           * **보안 권장 사항**: 리포지토리 전용 주체 세트 사용

        **필요한 값**:

        설정 후 다음 정보가 필요합니다.

        * **GCP\_WORKLOAD\_IDENTITY\_PROVIDER**: 전체 공급자 리소스 이름
        * **GCP\_SERVICE\_ACCOUNT**: 서비스 계정 이메일 주소

        <Tip>
          Workload Identity Federation을 사용하면 다운로드 가능한 서비스 계정 키가 필요 없어져 보안이 향상됩니다.
        </Tip>

        자세한 설정 지침은 [Google Cloud Workload Identity Federation 문서](https://cloud.google.com/iam/docs/workload-identity-federation)를 참조하세요.
      </Accordion>
    </AccordionGroup>
  </Step>

  <Step title="필수 시크릿 추가">
    리포지토리에 다음 시크릿을 추가하세요 (Settings → Secrets and variables → Actions):

    #### Claude API (직접 연결)의 경우:

    1. **API 인증용**:
       * `ANTHROPIC_API_KEY`: [console.anthropic.com](https://console.anthropic.com)에서 발급받은 Claude API 키

    2. **GitHub App용 (자체 앱을 사용하는 경우)**:
       * `APP_ID`: GitHub App ID
       * `APP_PRIVATE_KEY`: 개인 키 (.pem) 내용

    #### Google Cloud Agent Platform의 경우:

    1. **GCP 인증용**:
       * `GCP_WORKLOAD_IDENTITY_PROVIDER`
       * `GCP_SERVICE_ACCOUNT`

    2. **GitHub App용 (자체 앱을 사용하는 경우)**:
       * `APP_ID`: GitHub App ID
       * `APP_PRIVATE_KEY`: 개인 키 (.pem) 내용

    #### Amazon Bedrock의 경우:

    1. **AWS 인증용**:
       * `AWS_ROLE_TO_ASSUME`

    2. **GitHub App용 (자체 앱을 사용하는 경우)**:
       * `APP_ID`: GitHub App ID
       * `APP_PRIVATE_KEY`: 개인 키 (.pem) 내용
  </Step>

  <Step title="워크플로우 파일 생성">
    클라우드 공급자와 연동되는 GitHub Actions 워크플로우 파일을 생성합니다. 아래 예시는 Amazon Bedrock 및 Google Cloud Agent Platform에 대한 완전한 구성을 보여줍니다.

    <AccordionGroup>
      <Accordion title="Amazon Bedrock 워크플로우">
        **사전 요구 사항:**

        * Claude 모델 권한이 활성화된 Amazon Bedrock 접근 권한
        * AWS에 OIDC identity provider로 구성된 GitHub
        * GitHub Actions를 신뢰하는 Amazon Bedrock 권한이 있는 IAM 역할

        **필요한 GitHub 시크릿:**

        | 시크릿 이름 | 설명 |
        | :--- | :--- |
        | `AWS_ROLE_TO_ASSUME` | Amazon Bedrock 접근용 IAM 역할의 ARN |
        | `APP_ID` | GitHub App ID (앱 설정에서 확인) |
        | `APP_PRIVATE_KEY` | GitHub App용으로 생성한 개인 키 |

        ```yaml theme={null}
        name: Claude PR Action

        permissions:
          contents: write
          pull-requests: write
          issues: write
          id-token: write

        on:
          issue_comment:
            types: [created]
          pull_request_review_comment:
            types: [created]
          issues:
            types: [opened, assigned]

        jobs:
          claude-pr:
            if: |
              (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
              (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
              (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
            runs-on: ubuntu-latest
            env:
              AWS_REGION: us-west-2
            steps:
              - name: Checkout repository
                uses: actions/checkout@v4

              - name: Generate GitHub App token
                id: app-token
                uses: actions/create-github-app-token@v2
                with:
                  app-id: ${{ secrets.APP_ID }}
                  private-key: ${{ secrets.APP_PRIVATE_KEY }}

              - name: Configure AWS Credentials (OIDC)
                uses: aws-actions/configure-aws-credentials@v4
                with:
                  role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
                  aws-region: us-west-2

              - uses: anthropics/claude-code-action@v1
                with:
                  github_token: ${{ steps.app-token.outputs.token }}
                  use_bedrock: "true"
                  claude_args: '--model us.anthropic.claude-sonnet-4-6 --max-turns 10'
        ```

        <Tip>
          Amazon Bedrock의 모델 ID 형식에는 리전 접두사(예: `us.anthropic.claude-sonnet-4-6`)가 포함됩니다.
        </Tip>
      </Accordion>

      <Accordion title="Google Cloud Agent Platform 워크플로우">
        **사전 요구 사항:**

        * GCP 프로젝트에서 Google Cloud Agent Platform API 활성화
        * GitHub용으로 구성된 Workload Identity Federation
        * Google Cloud Agent Platform 권한이 있는 서비스 계정

        **필요한 GitHub 시크릿:**

        | 시크릿 이름 | 설명 |
        | :--- | :--- |
        | `GCP_WORKLOAD_IDENTITY_PROVIDER` | Workload identity provider 리소스 이름 |
        | `GCP_SERVICE_ACCOUNT` | Google Cloud Agent Platform 접근 권한이 있는 서비스 계정 이메일 |
        | `APP_ID` | GitHub App ID (앱 설정에서 확인) |
        | `APP_PRIVATE_KEY` | GitHub App용으로 생성한 개인 키 |

        ```yaml theme={null}
        name: Claude PR Action

        permissions:
          contents: write
          pull-requests: write
          issues: write
          id-token: write

        on:
          issue_comment:
            types: [created]
          pull_request_review_comment:
            types: [created]
          issues:
            types: [opened, assigned]

        jobs:
          claude-pr:
            if: |
              (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
              (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
              (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
            runs-on: ubuntu-latest
            steps:
              - name: Checkout repository
                uses: actions/checkout@v4

              - name: Generate GitHub App token
                id: app-token
                uses: actions/create-github-app-token@v2
                with:
                  app-id: ${{ secrets.APP_ID }}
                  private-key: ${{ secrets.APP_PRIVATE_KEY }}

              - name: Authenticate to Google Cloud
                id: auth
                uses: google-github-actions/auth@v2
                with:
                  workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
                  service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

              - uses: anthropics/claude-code-action@v1
                with:
                  github_token: ${{ steps.app-token.outputs.token }}
                  trigger_phrase: "@claude"
                  use_vertex: "true"
                  claude_args: '--model claude-sonnet-4-5@20250929 --max-turns 10'
                env:
                  ANTHROPIC_VERTEX_PROJECT_ID: ${{ steps.auth.outputs.project_id }}
                  CLOUD_ML_REGION: us-east5
                  VERTEX_REGION_CLAUDE_4_5_SONNET: us-east5
        ```

        <Tip>
          프로젝트 ID는 Google Cloud 인증 단계에서 자동으로 가져오므로 직접 하드코딩할 필요가 없습니다.
        </Tip>
      </Accordion>
    </AccordionGroup>
  </Step>
</Steps>

## 문제 해결

### Claude가 @claude 명령에 응답하지 않음

GitHub App이 올바르게 설치되었는지 확인하고, 워크플로우가 활성화되어 있는지 확인하며, API 키가 리포지토리 시크릿에 설정되었는지 확인하고, 주석에 `/claude`가 아닌 `@claude`가 포함되어 있는지 확인하세요.

### Claude 커밋에서 CI가 실행되지 않음

Actions 사용자가 아닌 GitHub App 또는 커스텀 앱을 사용 중인지 확인하고, 워크플로우 트리거에 필요한 이벤트가 포함되어 있는지 확인하며, 앱 권한에 CI 트리거가 포함되어 있는지 확인하세요.

### 인증 오류

API 키가 유효하고 충분한 권한이 있는지 확인하세요. Amazon Bedrock이나 Google Cloud Agent Platform의 경우 자격 증명 구성을 확인하고 워크플로우에서 시크릿 이름이 올바른지 확인하세요.

## 고급 구성

### Action 파라미터

Claude Code Action v1은 간소화된 구성을 사용합니다.

| 파라미터 | 설명 | 필수 여부 |
| :--- | :--- | :--- |
| `prompt` | Claude용 지침 (일반 텍스트 또는 [스킬](/docs/en/skills) 이름) | 아니요\* |
| `claude_args` | Claude Code에 전달되는 CLI 인수 | 아니요 |
| `plugin_marketplaces` | 줄바꿈으로 구분된 플러그인 마켓플레이스 Git URL 목록 | 아니요 |
| `plugins` | 실행 전 설치할 플러그인 이름의 줄바꿈 구분 목록 | 아니요 |
| `anthropic_api_key` | Claude API 키 | 예\*\* |
| `github_token` | API 접근용 GitHub 토큰 | 아니요 |
| `trigger_phrase` | 커스텀 트리거 문구 (기본값: "@claude") | 아니요 |
| `use_bedrock` | Claude API 대신 Amazon Bedrock 사용 | 아니요 |
| `use_vertex` | Claude API 대신 Google Cloud Agent Platform 사용 | 아니요 |

\*이슈/PR 주석 시 생략되면 Claude는 트리거 문구에 응답하므로 선택 사항입니다.\
\*\*직접 Claude API 시 필수이며, Amazon Bedrock이나 Google Cloud Agent Platform의 경우 불필요합니다.

#### CLI 인수 전달

`claude_args` 파라미터는 임의의 Claude Code CLI 인수를 전달받을 수 있습니다.

```yaml theme={null}
claude_args: "--max-turns 5 --model claude-sonnet-5 --mcp-config /path/to/config.json"
```

주요 인수:

* `--max-turns`: 최대 대화 턴 수 (기본값: 10)
* `--model`: 사용할 모델 (예: `claude-sonnet-5`)
* `--mcp-config`: MCP 구성 경로
* `--allowedTools`: 허용된 도구의 쉼표 구분 목록 (`--allowed-tools` 별칭도 작동함)
* `--debug`: 디버그 출력 활성화

### 대안 연동 방식

`/install-github-app` 명령이 권장되는 방식이지만 다음을 사용할 수도 있습니다.

* **커스텀 GitHub App**: 브랜딩된 사용자 이름이나 커스텀 인증 플로우가 필요한 조직용. 필요한 권한(contents, issues, pull requests)이 있는 자체 GitHub App을 생성하고 actions/create-github-app-token 액션을 사용하여 워크플로우에서 토큰을 생성합니다.
* **수동 GitHub Actions**: 유연성을 극대화하기 위한 직접 워크플로우 구성
* **MCP 구성**: Model Context Protocol 서버 동적 로드

인증, 보안 및 고급 구성에 대한 자세한 가이드는 [Claude Code Action 문서](https://github.com/anthropics/claude-code-action/blob/main/docs)를 참조하세요.

### Claude 동작 커스터마이징

다음 두 가지 방법으로 Claude 동작을 구성할 수 있습니다.

1. **CLAUDE.md**: 리포지토리 루트의 `CLAUDE.md` 파일에 코딩 표준, 리뷰 기준, 프로젝트 전용 규칙을 정의합니다. Claude는 PR을 생성하고 요청에 응답할 때 이러한 가이드라인을 준수합니다. 세부 정보는 [메모리 문서](/docs/en/memory)를 참조하세요.
2. **커스텀 프롬프트**: 워크플로우 파일의 `prompt` 파라미터를 사용하여 워크플로우 전용 지침을 제공합니다. 이를 통해 서로 다른 워크플로우나 작업에 대해 Claude 동작을 커스터마이징할 수 있습니다.

Claude는 PR을 생성하고 요청에 응답할 때 이 가이드라인을 준수합니다.
