> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Amazon Bedrock 기반 Claude Code

> 설정, IAM 구성 및 문제 해결을 포함하여 Amazon Bedrock을 통해 Claude Code를 구성하는 방법에 대해 알아보세요.

## 사전 요구 사항

Amazon Bedrock에서 Claude Code를 구성하기 전에 다음 사항을 확인하세요:

* Amazon Bedrock 접근 권한이 활성화된 AWS 계정
* Amazon Bedrock 내 원하는 Claude 모델(예: Claude Sonnet 4.6)에 대한 접근 권한
* AWS CLI 설치 및 구성 (선택 사항 - 자격 증명을 얻는 다른 방법이 없는 경우에만 필요)
* 적절한 IAM 권한

자신의 Amazon Bedrock 자격 증명으로 로그인하려면 아래 [Bedrock으로 로그인](#sign-in-with-bedrock)을 따르세요. 팀 전체에 Claude Code를 배포하려면 [수동 설정](#set-up-manually) 단계를 사용하고 배포 전에 [모델 버전 고정](#4-pin-model-versions)을 수행하세요.

## Bedrock으로 로그인

AWS 자격 증명이 있고 Amazon Bedrock을 통해 Claude Code를 사용하려는 경우 로그인 마법사가 프로세스를 안내합니다. 계정당 한 번 AWS 측 사전 요구 사항을 완료하면 마법사가 Claude Code 측 설정을 처리합니다.

<Steps>
  <Step title="AWS 계정에서 Anthropic 모델 활성화">
    [Amazon Bedrock 콘솔](https://console.aws.amazon.com/bedrock/)에서 Model catalog를 열고 Anthropic 모델을 선택한 후 유즈케이스 양식을 제출합니다. 제출 즉시 접근 권한이 부여됩니다. AWS Organizations의 경우 [유즈케이스 제출](#1-submit-use-case-details)을, 역할에 필요한 권한은 [IAM 구성](#iam-configuration)을 참고하세요.
  </Step>

  <Step title="Claude Code 시작 및 Amazon Bedrock 선택">
    `claude`를 실행합니다. 로그인 프롬프트에서 **3rd-party platform**을 선택한 다음 **Amazon Bedrock**을 선택합니다. 이미 로그인되어 있어 대화 프롬프트가 보이는 경우 `/setup-bedrock`을 실행하여 마법사를 엽니다. 이 명령어는 Bedrock이 구성될 때까지 명령어 메뉴에 나열되지 않더라도 입력 시 동작합니다.
  </Step>

  <Step title="마법사 지시사항 따르기">
    AWS 인증 방식을 선택합니다: `~/.aws` 디렉토리에서 감지된 AWS 프로필, Amazon Bedrock API 키, 액세스 키 및 시크릿, 또는 환경에 이미 있는 자격 증명. 마법사가 리전을 가져오고 계정에서 호출할 수 있는 Claude 모델을 검증하여 고정(pin)할 수 있도록 합니다. 결과는 [사용자 설정 파일](/docs/en/settings)의 `env` 블록에 저장되므로 환경 변수를 직접 내보낼 필요가 없습니다.
  </Step>
</Steps>

로그인한 후에는 언제든지 `/setup-bedrock`을 실행하여 마법사를 다시 열고 자격 증명, 리전 또는 모델 고정 설정을 변경할 수 있습니다. 마법사는 `~/.claude/settings.json` 또는 [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars#variables)이 설정된 경우 `$CLAUDE_CONFIG_DIR/settings.json`에 내용을 기록합니다.

## 수동 설정

CI 또는 스크립트 기반 엔터프라이즈 배포 등 마법사 대신 환경 변수를 통해 Amazon Bedrock을 구성하려면 아래 단계를 따르세요.

### 1. 유즈케이스 세부 정보 제출

Anthropic 모델을 처음 사용하는 사용자는 모델을 호출하기전에 유즈케이스 세부 정보를 제출해야 합니다. 이는 AWS 계정당 한 번 수행됩니다.

1. 아래 설명된 올바른 IAM 권한이 있는지 확인합니다
2. [Amazon Bedrock 콘솔](https://console.aws.amazon.com/bedrock/)로 이동합니다
3. **Model catalog**에서 Anthropic 모델을 선택합니다
4. 유즈케이스 양식을 작성합니다. 제출 즉시 접근 권한이 부여됩니다.

AWS Organizations를 사용하는 경우 [`PutUseCaseForModelAccess` API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_PutUseCaseForModelAccess.html)를 사용하여 관리 계정에서 한 번만 양식을 제출하면 됩니다. 이 호출에는 `bedrock:PutUseCaseForModelAccess` IAM 권한이 필요합니다. 승인은 하위 계정으로 자동 확장됩니다.

### 2. AWS 자격 증명 구성

Claude Code는 기본 AWS SDK 자격 증명 체인을 사용합니다. 다음 방법 중 하나로 자격 증명을 설정하세요:

**옵션 A: AWS CLI 구성**

```bash theme={null}
aws configure
```

**옵션 B: 환경 변수 (액세스 키)**

```bash theme={null}
export AWS_ACCESS_KEY_ID=your-access-key-id
export AWS_SECRET_ACCESS_KEY=your-secret-access-key
export AWS_SESSION_TOKEN=your-session-token
```

**옵션 C: 환경 변수 (SSO 프로필)**

다음 명령어를 실행하기 전에 `your-profile-name`을 자신의 AWS 프로필 이름으로 교체하세요.

```bash theme={null}
aws sso login --profile=your-profile-name

export AWS_PROFILE=your-profile-name
```

**옵션 D: Amazon Bedrock API 키**

```bash theme={null}
export AWS_BEARER_TOKEN_BEDROCK=your-bedrock-api-key
```

### 3. Claude Code 구성

다음 환경 변수를 설정하여 Amazon Bedrock을 활성화합니다:

```bash theme={null}
# Bedrock 연동 활성화
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1  # AWS 프로필에 리전이 이미 설정된 경우 생략 가능
```

### 4. 모델 버전 고정 (Pinning)

<Warning>
  여러 사용자에게 배포할 때는 특정 모델 버전을 고정하세요. 고정하지 않으면 `sonnet` 및 `opus`와 같은 모델 별칭이 Claude Code의 기본 내장값으로 확인되어 최신 릴리스보다 뒤처지거나 계정에서 아직 사용 불가할 수 있습니다.
</Warning>

이 환경 변수들을 특정 Amazon Bedrock 모델 ID로 설정하세요:

```bash theme={null}
export ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
```

## IAM 구성

Claude Code에 필요한 권한이 포함된 IAM 정책을 생성합니다:

```json theme={null}
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowModelAndInferenceProfileAccess",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:ListInferenceProfiles",
        "bedrock:GetInferenceProfile"
      ],
      "Resource": [
        "arn:aws:bedrock:*:*:inference-profile/*",
        "arn:aws:bedrock:*:*:application-inference-profile/*",
        "arn:aws:bedrock:*:*:foundation-model/*"
      ]
    }
  ]
}
```

## Mantle 엔드포인트 사용하기

Mantle은 Amazon Bedrock Invoke API 대신 네이티브 Anthropic API 형태를 통해 Claude 모델을 제공하는 Amazon Bedrock 엔드포인트입니다.

### Mantle 활성화

AWS 자격 증명이 이미 구성된 상태에서 `CLAUDE_CODE_USE_MANTLE`을 설정하여 요청을 Mantle 엔드포인트로 라우팅합니다:

```bash theme={null}
export CLAUDE_CODE_USE_MANTLE=1
export AWS_REGION=us-east-1
```

Claude Code 내부에서 `/status`를 실행하여 확인합니다. Mantle이 활성화되면 제공자 라인에 `Amazon Bedrock (Mantle)`이 표시됩니다.

## 문제 해결

### 리전 문제

리전 오류가 발생하는 경우:

* 모델 접근 가능 여부 확인: `aws bedrock list-inference-profiles --region your-region`
* 지원되는 리전으로 전환: `export AWS_REGION=us-east-1`

### 추가 리소스

* [Amazon Bedrock documentation](https://docs.aws.amazon.com/bedrock/)
* [Amazon Bedrock pricing](https://aws.amazon.com/bedrock/pricing/)
* [Amazon Bedrock inference profiles](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html)
