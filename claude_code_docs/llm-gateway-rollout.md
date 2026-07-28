> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 조직을 위한 LLM 게이트웨이 배포하기

> Claude Code를 위한 게이트웨이 제품 배포: Claude Code가 전송하는 내용을 전달하도록 구성하고, 개발자 자격 증명을 발급하며, 관리형 설정을 통해 구성을 배포하고, 배포 결과를 검증합니다.

이 페이지는 관리자가 Claude Code를 위한 LLM 게이트웨이를 배포하는 과정을 안내합니다. 이 안내는 [게이트웨이 요구사항](#gateway-requirements)을 충족하는 게이트웨이 제품이 이미 배포되어 있다고 가정합니다. 특정 제품의 배포 및 운영 자체는 여기서 다루지 않으므로 해당 벤더의 문서를 따라 배포하세요.

<Note>
  * 본인 머신의 Claude Code를 기존 게이트웨이에 연결하려면 [Claude Code를 LLM 게이트웨이에 연결하기](/docs/en/llm-gateway-connect)를 참조하세요.
  * Claude Code가 게이트웨이로 전송하는 내용과 전달해야 할 항목에 대해서는 [게이트웨이 프로토콜 참조](/docs/en/llm-gateway-protocol)를 참조하세요.
</Note>

## 사전 요구사항

배포를 완료하려면 다음 항목이 필요합니다:

* 개발자에게 배포할 정확한 주소(리다이렉트 주소가 아님)에서 HTTPS 서비스를 제공하며, Claude 모델 이름을 프로바이더로 라우팅하도록 설정되어 사용자의 인프라에 배포된 게이트웨이
* 게이트웨이가 요청을 전달할 때 사용할 프로바이더 자격 증명:
  * Anthropic API의 경우: [Claude Console](https://platform.claude.com/settings/keys)에서 발급받은 API 키
  * 클라우드 프로바이더의 경우: 모델 액세스 권한이 있는 클라우드 자격 증명. [Amazon Bedrock](/docs/en/amazon-bedrock#prerequisites), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai#prerequisites), 또는 [Microsoft Foundry](/docs/en/microsoft-foundry#prerequisites) 페이지의 사전 요구사항을 참조하세요.
* MDM이나 구성 관리 도구 등 개발자 머신에 설정 파일을 전달할 수 있는 방법
  * 아직 없다면 [설정이 장치에 도달하는 방법 결정하기](/docs/en/admin-setup#decide-how-settings-reach-devices)에서 옵션들을 비교해 보세요.

### 게이트웨이 요구사항

게이트웨이를 제공하는 제품이 무엇이든 다음 요구사항을 충족해야 합니다:

* **지원되는 API 형식 수용**: [API 형식 표](/docs/en/llm-gateway-protocol#api-formats)에 기재된 형식 중 하나를 지원해야 합니다. 아래 배포 단계는 대부분의 게이트웨이가 제공하는 `POST /v1/messages` 경로의 Anthropic Messages API를 가정합니다.
* **응답 스트리밍(Streaming)**: 전체 응답을 버퍼링하지 않고 도착하는 대로 Server-Sent Events를 전달해야 합니다.
* **Claude 모델 이름 라우팅**: 개발자가 사용하는 각 이름을 업스트림 모델에 매핑해야 합니다. Claude Code는 각 요청 시 `claude-sonnet-4-6`과 같은 모델 이름을 보내며, 대부분의 게이트웨이 제품에서 이 매핑은 게이트웨이 설정의 모델 목록이나 라우팅 테이블로 관리됩니다.
* **헤더 및 본문의 변경 없는 전달**: `anthropic-beta`, `anthropic-version`, 및 요청 본문을 양방향으로 변경 없이 전달해야 합니다. [기능 전달(feature pass-through) 표](/docs/en/llm-gateway-protocol#feature-pass-through)에서 전달되지 않을 때 오작동하는 기능을 확인할 수 있습니다.
* **업스트림 오류의 수정 없는 반환**: Claude Code의 자동 복구 로직은 오류 문구에 기반하므로, 오류를 게이트웨이 자체 엔벨로프로 감싸서 반환하면 복구 기능이 작동하지 않습니다.
* **요청 본문의 WAF 검사 예외 처리**: Claude Code 프롬프트에는 cross-site-scripting 본문 규칙에 걸릴 수 있는 소스 코드 및 XML 스타일 태그가 포함되어 있습니다. 게이트웨이 앞단의 WAF가 이를 검사하면 짧은 테스트 요청은 통과하지만 실제 세션에서는 `403` 오류를 반환합니다.

선택적으로, Claude Code가 [모델 탐색](/docs/en/llm-gateway-protocol#model-discovery)을 통해 게이트웨이로부터 모델 선택기 항목을 구성할 수 있도록 `GET /v1/models`를 제공하세요.

## 배포 단계

배포는 각 단계마다 체크포인트가 포함된 5단계로 진행됩니다:

1. [게이트웨이가 모델을 정상 라우팅하는지 확인](#confirm-the-gateway-routes-your-models)
2. [각 개발자에게 자격 증명 발급](#issue-developer-credentials)
3. [게이트웨이를 대상으로 Claude Code 테스트](#test-claude-code-against-the-gateway)
4. [베이스 URL 및 자격 증명 배포](#distribute-the-configuration)
5. [개발자 머신에서 검증](#verify-the-rollout)

이 과정에는 세 가지 자격 증명이 사용되며, 체크포인트에서는 문제가 발생했을 때 어떤 자격 증명이 원인인지 식별할 수 있도록 다음과 같은 래벨로 구분합니다:

| 자격 증명                         | 소유 주체                                                                                            | 체크포인트 내 플레이스홀더                                   |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------- | :---------------------------------------------------------- |
| 프로바이더 자격 증명 (Provider credential) | 게이트웨이 (업스트림 프로바이더로 요청을 전달할 때 사용)                                              | 게이트웨이에 설정됨 (클라이언트 명령에는 표시 안 됨)       |
| 게이트웨이 관리자 자격 증명 (Gateway administrative credential) | 관리자 (게이트웨이 제품이 관리자/테스트 인터페이스용으로 발급한 경우)                                | `<gateway-key>`                                             |
| 개발자 키 (Developer key)         | 각 개발자 ([개발자 자격 증명 발급](#issue-developer-credentials) 단계에서 게이트웨이가 발급)         | `<developer-key>`                                           |

### 게이트웨이가 모델을 정상 라우팅하는지 확인

게이트웨이에 프로바이더 자격 증명이 구성되어 있고, 베이스 URL에서 수신 대기 중이며, 프로바이더 API로 요청을 정상 전달하고 있는지 확인해야 합니다. 배포 구성에 맞는 두 값을 대체하여 최소한의 요청으로 엔드투엔드 경로를 테스트하세요:

* `<gateway-key>`는 현재 게이트웨이를 호출할 수 있는 자격 증명입니다: 관리자 키, 테스트 키, 또는 이미 발급한 본인의 개발자 키. 별도의 관리자 자격 증명이 없는 게이트웨이 제품이라면 [개발자 자격 증명 발급](#issue-developer-credentials)에서 개발자 키를 먼저 스스로에게 발급하세요.
* `model`은 게이트웨이가 라우팅하도록 설정된 Claude 모델 이름입니다. 예제에서는 `claude-sonnet-4-6`을 사용하며, 설정된 이름으로 대체하세요.

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    curl -X POST "https://llm-gateway.example.com/v1/messages" \
      -H "Authorization: Bearer <gateway-key>" \
      -H "anthropic-version: 2023-06-01" \
      -H "content-type: application/json" \
      -d '{"model": "claude-sonnet-4-6", "max_tokens": 1, "messages": [{"role": "user", "content": "."}]}'
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    Invoke-RestMethod -Method Post -Uri "https://llm-gateway.example.com/v1/messages" `
      -Headers @{ "Authorization" = "Bearer <gateway-key>"; "anthropic-version" = "2023-06-01" } `
      -ContentType "application/json" `
      -Body '{"model": "claude-sonnet-4-6", "max_tokens": 1, "messages": [{"role": "user", "content": "."}]}'
    ```
  </Tab>
</Tabs>

**체크포인트**: `content` 필드가 포함된 `200` 응답이 오면 게이트웨이가 해당 모델 이름으로 프로바이더에 도달했음을 의미합니다. `404`는 게이트웨이에서 해당 이름이 라우팅되지 않음을 의미하고, 프로바이더로부터 수신한 `401`은 게이트웨이의 프로바이더 자격 증명이 잘못되었음을 의미합니다.

게이트웨이 라우팅 설정에 포함된 각 Claude 모델 이름에 대해 테스트를 반복하세요. 게이트웨이가 라우팅하지 않는 이름을 선택하면 개발자에게 `404`가 반환되므로 배포 전에 모든 이름을 테스트하세요.

<Note>
  리다이렉트 뒤에 게이트웨이를 두지 마세요. 리다이렉트는 추론 요청 시 요청 본문을 유실시키거나 자격 증명 헤더를 제거할 수 있으며, [모델 탐색](/docs/en/llm-gateway-protocol#model-discovery)은 자격 증명이 리다이렉트 대상으로 누출되는 것을 막기 위해 모든 리다이렉트를 실패로 처리합니다.
</Note>

### 개발자 자격 증명 발급

각 개발자는 인증을 위한 자체 게이트웨이 키가 필요합니다. 해당 제품의 자격 증명 관리 문서를 참고하여 개발자별로 게이트웨이에 자격 증명을 생성하세요.

`<gateway-key>`를 새로 발급받은 `<developer-key>`로 대체하여 [게이트웨이가 모델을 정상 라우팅하는지 확인](#confirm-the-gateway-routes-your-models)에서 사용한 동일한 요청으로 키가 정상 동작하는지 테스트하세요:

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    curl -X POST "https://llm-gateway.example.com/v1/messages" \
      -H "Authorization: Bearer <developer-key>" \
      -H "anthropic-version: 2023-06-01" \
      -H "content-type: application/json" \
      -d '{"model": "claude-sonnet-4-6", "max_tokens": 1, "messages": [{"role": "user", "content": "."}]}'
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    Invoke-RestMethod -Method Post -Uri "https://llm-gateway.example.com/v1/messages" `
      -Headers @{ "Authorization" = "Bearer <developer-key>"; "anthropic-version" = "2023-06-01" } `
      -ContentType "application/json" `
      -Body '{"model": "claude-sonnet-4-6", "max_tokens": 1, "messages": [{"role": "user", "content": "."}]}'
    ```
  </Tab>
</Tabs>

**체크포인트**: `content` 필드가 포함된 `200` 응답이 오면 개발자 키가 게이트웨이에 도달하고 게이트웨이가 요청을 전달하고 있음을 의미합니다. [이전 단계](#confirm-the-gateway-routes-your-models)가 성공했음에도 여기서 `401`이 발생하면 개발자 키가 잘못되었거나 게이트웨이에 아직 적용되지 않은 것입니다.

공유 키 대신 개발자마다 개별 키를 발급해야 개발자별 사용량 귀속 추적 및 개별 오프보딩(권한 회수)이 정상 작동합니다. 키를 보관하는 환경 변수는 게이트웨이가 읽는 헤더에 따라 다릅니다. `Authorization: Bearer` 헤더에서 자격 증명을 확인하는 게이트웨이의 경우 개발자는 `ANTHROPIC_AUTH_TOKEN`에 키를 설정합니다. `x-api-key` 헤더에서 키를 읽는 게이트웨이의 경우 대신 `ANTHROPIC_API_KEY`를 설정합니다. [자격 증명 표](/docs/en/llm-gateway-connect#set-the-credential-variable)에서 이 매핑을 확인할 수 있습니다.

### 게이트웨이를 대상으로 Claude Code 테스트

전체 배포 시 전달할 동일한 구성을 사용하여, 어떤 것도 배포하기 전에 직접 게이트웨이를 거쳐 Claude Code를 실행해 보세요. `.env`나 설정 파일이 아닌 터미널에 직접 입력하세요. 해당 터미널 세션 동안만 유지되므로 창을 닫으면 기존 정상 구성으로 돌아갑니다. 게이트웨이가 `x-api-key` 헤더를 읽는 경우 `ANTHROPIC_AUTH_TOKEN` 대신 `ANTHROPIC_API_KEY`를 사용하세요:

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_BASE_URL=https://llm-gateway.example.com
    export ANTHROPIC_AUTH_TOKEN="<developer-key>"
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_BASE_URL = "https://llm-gateway.example.com"
    $env:ANTHROPIC_AUTH_TOKEN = "<developer-key>"
    ```
  </Tab>
</Tabs>

그런 다음 게이트웨이를 통해 단발성 프롬프트를 전송하세요:

```bash theme={null}
claude -p "Reply with one word: connected"
```

**체크포인트**: 프롬프트가 응답을 반환하고, 게이트웨이 로그에 `/v1/messages` 경로로 상태 `200`인 `POST` 요청이 나타나야 합니다. Claude Code는 `?beta=true`와 같은 쿼리 문자열을 덧붙이므로 전체 URL이 아닌 경로 패턴으로 확인하세요. 실패 메시지는 원인에 따라 두 가지 형태로 나타납니다:

* `Not logged in`: 게이트웨이 로그를 확인하여 원인을 구별하세요. 로그가 비어 있다면 세션에 자격 증명이 전달되지 않아 머신 외부로 요청이 나가지 않은 것이므로 테스트 중인 셸에서 export를 다시 실행하세요. 로그에 `401` 본문에 `x-api-key`가 명시된 거부된 요청이 보인다면 게이트웨이가 해당 헤더로 키를 받으려는 것이므로 `ANTHROPIC_API_KEY`로 전환하세요.
* `Failed to authenticate. API Error: 401`은 자격 증명이 전송되었으나 거부되었음을 의미합니다. 게이트웨이 로그에 원인이 표시됩니다: `api.anthropic.com` 또는 프로바이더 엔드포인트에서 거부된 `401`은 게이트웨이가 업스트림에 도달했으나 프로바이더 자격 증명이 거부되었음을 나타내므로, 개발자 키는 정상이나 게이트웨이가 가진 프로바이더 자격 증명이 잘못되었거나 더미 값인 경우입니다.

베이스 URL이 잘못되었거나 연결 불가능한 경우 다른 증상이 나타납니다: Claude Code가 [지연 재시도(backoff)를 통해 연결을 다시 시도](/docs/en/errors#automatic-retries)하며 오류를 보고하기 전까지 수분 간 출력이 없을 수 있습니다. 명령이 멈춘 것처럼 보이면 기다리지 말고 게이트웨이 로그를 확인하세요; 수신된 요청이 없다는 것은 `ANTHROPIC_BASE_URL`이 게이트웨이를 가리키고 있지 않음을 의미합니다.

### 베이스 URL 및 자격 증명 배포

모든 개발자 머신에는 게이트웨이 주소와 자격 증명이 필요합니다. [관리형 설정](/docs/en/settings#settings-files)을 통해 중앙에서 배포하여 개발자가 직접 설정할 필요가 없도록 만들거나, 개발자에게 직접 전달하여 스스로 설정하도록 할 수 있습니다.

#### 배포할 항목

어떤 경로를 선택하든 동일한 변수 세트가 적용됩니다. 대부분의 배포에는 `ANTHROPIC_BASE_URL`과 자격 증명만 필요하며, 게이트웨이 설정에 따라 필요한 경우 조건부 행의 항목들을 포함시키세요.

| 변수 또는 설정                                                                                                                                                                                                                  | 역할                                                                                                                                                                                                                           | 포함 조건                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ANTHROPIC_BASE_URL`                                                                                                                                                                                                             | Claude Code의 API 요청을 `api.anthropic.com` 대신 게이트웨이로 전송                                                                                                                                                            | 항상 포함                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `apiKeyHelper`, 또는 `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_API_KEY` 내의 자격 증명                                                                                                                                                 | 게이트웨이에 대한 각 요청을 인증함. 헬퍼는 키를 가져오는 명령을 실행하며; 변수는 각각 `Authorization: Bearer` 및 `x-api-key`로 전송되는 정적 키를 보유함                                                                          | 항상 포함 (셋 중 하나)                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `ANTHROPIC_CUSTOM_HEADERS`                                                                                                                                                                                                       | 모든 API 요청에 추가 HTTP 헤더를 덧붙임                                                                                                                                                                                        | 게이트웨이가 모든 요청에 테넌트 또는 라우팅 헤더를 필요로 하는 경우                                                                                                                                                                                                                                                                                                                                                                                      |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY`                                                                                                                                                                                     | 시작 시 게이트웨이의 `/v1/models`를 쿼리하여 반환된 이름을 `/model` 선택기에 추가                                                                                                                                               | 게이트웨이가 `/v1/models`를 제공하고 개발자의 선택기에 해당 목록을 표시하고 싶은 경우                                                                                                                                                                                                                                                                                                                                                                     |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`                                                                                                                                                                                         | Claude Code가 사전 공개 기능 헤더 및 본문 필드를 전송하지 않도록 함                                                                                                                                                            | 게이트웨이가 베타 필드를 거부하는 Amazon Bedrock 또는 Google Cloud's Agent Platform 업스트림으로 요청을 전달하는 경우; [게이트웨이 요구사항](#gateway-requirements)을 참조                                                                                                                                                                                                                                                                               |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` 또는 `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK`                                                                                                                                            | `ANTHROPIC_BASE_URL`을 따르지 않고 `api.anthropic.com`을 직접 호출하는 사용 가능 여부 확인이 실패/차단되거나 Anthropic 자격 증명 부재로 건너뛰어질 때 [fast mode](/docs/en/fast-mode)를 복원함                                 | 조직이 fast mode를 사용하고 개발자가 `ANTHROPIC_AUTH_TOKEN`만으로 인증하거나, `ANTHROPIC_API_KEY` 또는 `apiKeyHelper`의 게이트웨이 발급 키로 인증하거나, 네트워크가 `api.anthropic.com`으로의 직접 요청을 차단/가로채는 경우; 상황에 맞는 변수는 [프록시 및 LLM 게이트웨이 뒤에서 fast mode 사용하기](/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways)를 참조 |
| `ANTHROPIC_MODEL` 또는 [`ANTHROPIC_DEFAULT_HAIKU_MODEL`](/docs/en/model-config)                                                                                                                                                         | 메인 세션 및 백그라운드 트래픽에 Claude Code가 요청할 모델 이름을 설정                                                                                                                                                          | 게이트웨이가 Claude Code 기본값과 일치하지 않는 모델 이름을 라우팅하거나, [백그라운드 기능](/docs/en/costs#background-token-usage)을 다른 모델로 라우팅하는 경우. 일부 백그라운드 호출은 오버라이드 설정과 무관하게 내장 ID를 요청하므로 오버라이드 이름과 내장 모델 ID 모두를 라우팅해야 함; [모델 구성](/docs/en/model-config)에서 세션의 각 부분이 사용하는 모델을 다룸 |
| `ANTHROPIC_BEDROCK_BASE_URL`, `ANTHROPIC_VERTEX_BASE_URL`, `ANTHROPIC_FOUNDRY_BASE_URL`, 또는 `ANTHROPIC_AWS_BASE_URL` ([해당 프로바이더 변수](/docs/en/llm-gateway-connect#route-to-a-cloud-provider-through-a-gateway)와 함께) | 프로바이더 전용 베이스 URL을 통해 Claude Code가 게이트웨이를 가리키도록 함. Amazon Bedrock 및 Google Cloud's Agent Platform의 경우 해당 프로바이더의 네이티브 요청 형식으로 전환됨                                                             | 게이트웨이가 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, 또는 Claude Platform on AWS 앞단에 위치한 경우; [API 형식](/docs/en/llm-gateway-protocol#api-formats) 참조                                                                                                                                                                                                                                                   |

#### 관리형 설정을 통한 배포

MDM, 레지스트리 정책 또는 구성 관리를 통해 전달되는 [관리형 설정 파일](/docs/en/settings#settings-files)의 `env` 블록을 사용하여 변수를 전송하세요:

```json theme={null}
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://llm-gateway.example.com"
  },
  "apiKeyHelper": "/usr/local/bin/get-gateway-key"
}
```

표에 포함된 조건부 변수들도 동일한 `env` 블록에 추가하세요. Claude Code는 프로세스 환경 및 우선순위가 낮은 설정 위에 이를 적용하므로, 관리형 `ANTHROPIC_BASE_URL`은 강제 적용되며 개발자의 셸 export로 재정의할 수 없습니다.

게이트웨이 자격 증명과 함께 관리형 설정에 `forceLoginMethod` 또는 `forceLoginOrgUUID`를 포함하지 마세요. Claude Code v2.1.146 이상에서는 두 키 중 하나라도 포함되어 있으면(값에 상관없이) 시작 시 `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, 및 `apiKeyHelper`를 차단하므로 개발자에게 `This machine's managed settings require a first-party login`이 표시되고 더 이상 진행할 수 없게 됩니다.

[서버 관리형 설정](/docs/en/server-managed-settings#platform-availability) 배포는 `api.anthropic.com`에 대한 직접 연결이 필요하므로 게이트웨이 라우팅 세션에는 도달하지 않습니다. 게이트웨이 배포는 동일한 키를 강제 적용하는 이 파일 기반 관리형 설정 경로를 사용합니다.

자격 증명의 경우 위에 표시된 대로 관리형 설정 파일에 하나의 [`apiKeyHelper`](/docs/en/llm-gateway-connect#rotate-credentials-with-apikeyhelper) 명령을 배포하세요. 이 명령은 로컬 개발자 권한으로 보안 비밀 저장소에 인증하므로 각 머신에 고유한 키가 전달됩니다. 또는 기존 보안 비밀 공유 프로세스를 통해 개발자에게 각자의 키를 전달하고 직접 `ANTHROPIC_AUTH_TOKEN`을 설정하도록 유도할 수도 있습니다.

일부 환경은 별도의 배포가 필요합니다:

* 데스크톱 앱은 관리형 설정이 아닌 서드파티 추론 설정에서 게이트웨이 라우팅을 읽어오므로, 데스크톱 세션도 게이트웨이를 거쳐 라우팅되도록 MDM을 통해 관리형 설정과 함께 해당 파일을 배포하세요. [데스크톱 서드파티 설정 문서](https://claude.com/docs/third-party/claude-desktop/configuration) 및 [데스크톱 게이트웨이 문서](https://claude.com/docs/third-party/claude-desktop/gateway)를 참조하세요.
* CI 러너는 [러너 환경](/docs/en/llm-gateway-connect#configure-each-surface)에 `ANTHROPIC_BASE_URL`과 자격 증명이 설정되어야 합니다.
* 관리 대상 Windows 머신의 WSL은 [`wslInheritsWindowsSettings`](/docs/en/settings#available-settings)가 `true`일 때만 Windows 관리형 설정을 읽어옵니다.

#### 개발자에게 설정값 직접 안내하기

관리형 설정 배포 환경이 갖추어지지 않은 경우 각 개발자에게 [연결 페이지](/docs/en/llm-gateway-connect#configure-claude-code-yourself)를 따르는 데 필요한 항목을 전달하세요:

* 게이트웨이 URL
* 본인의 자격 증명
* **자격 증명을 넣을 변수**: Bearer 토큰 게이트웨이의 경우 `ANTHROPIC_AUTH_TOKEN`, `x-api-key` 게이트웨이의 경우 `ANTHROPIC_API_KEY`. 어떤 변수인지 명확히 안내하면 개발자가 [연결 페이지](/docs/en/llm-gateway-connect#set-the-credential-variable)에 설명된 시행착오를 줄일 수 있습니다.
* [배포할 항목 표](#what-to-distribute)에 안내된 조건부 변수 및 해당 값

[연결 페이지](/docs/en/llm-gateway-connect#configure-claude-code-yourself)에서 각 항목을 설정하는 단계를 상세히 안내합니다.

**체크포인트**: 개발자 머신에서 `claude`를 실행하면 배포된 자격 증명이 인증을 통과시켜 로그인 화면 없이 세션이 시작됩니다. 그 후 `/status`를 실행하고 **Status** 탭을 엽니다: `Anthropic base URL` 항목에 게이트웨이 주소가 표시되며, 관리형 배포의 경우 `Setting sources` 항목에 managed settings가 포함됩니다. 로그인 화면이 뜨거나 `Anthropic base URL` 항목이 누락된 경우 구성이 해당 머신에 도달하지 않은 것입니다.

### 배포 검증

개발자가 사용하는 네트워크 경로를 테스트할 수 있도록 게이트웨이 호스트가 아닌 개발자 머신에서 모든 것이 정상 작동하는지 확인하세요. 엔드포인트, 스트리밍 전달, 모델 라우팅을 한 번에 검증하는 스트리밍 요청을 전송합니다:

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    curl -N -X POST "https://llm-gateway.example.com/v1/messages" \
      -H "Authorization: Bearer <developer-key>" \
      -H "anthropic-version: 2023-06-01" \
      -H "content-type: application/json" \
      -d '{"model": "claude-sonnet-4-6", "max_tokens": 16, "stream": true, "messages": [{"role": "user", "content": "count to 3"}]}'
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $body = '{"model": "claude-sonnet-4-6", "max_tokens": 16, "stream": true, "messages": [{"role": "user", "content": "count to 3"}]}'
    $body | curl.exe -N -X POST "https://llm-gateway.example.com/v1/messages" `
      -H "Authorization: Bearer <developer-key>" `
      -H "anthropic-version: 2023-06-01" `
      -H "content-type: application/json" `
      --data-binary '@-'
    ```
  </Tab>
</Tabs>

`data:` 라인이 순차적으로 도착해야 합니다. 지연 후 응답 전체가 한 번에 나타난다면 게이트웨이가 버퍼링하고 있는 것이므로 Claude Code를 지연시킵니다; `404`는 해당 모델 이름이 라우팅되지 않음을 의미합니다. 모델 이름마다 반복 테스트하세요.

그런 다음 `claude`를 시작하고 메시지를 전송하세요. 각 증상은 다음과 같은 원인을 가집니다:

* 로그인 프롬프트가 표시되면 자격 증명 누락이 원인입니다. `/status`를 실행하고 **Status** 탭을 확인하세요: `Setting sources` 줄에 managed settings가 포함되어 있지 않다면 배포가 머신에 도달하지 않은 것입니다; 포함되어 있다면 개발자 자격 증명이 전달되지 않은 것이므로 `ANTHROPIC_AUTH_TOKEN` 또는 `apiKeyHelper`를 설정하세요.
* `Failed to authenticate` 오류는 게이트웨이가 요청을 거부함을 의미합니다; 로그에 어떤 자격 증명이 실패했는지 나옵니다. 게이트웨이가 직접 기록한 거부는 개발자 키를 지목하고, `api.anthropic.com` 또는 프로바이더 엔드포인트에서 발생한 `401`은 게이트웨이가 들고 있는 프로바이더 자격 증명이 거부되었음을 나타냅니다.
* 게이트웨이가 `x-api-key` 헤더를 요구하여 `ANTHROPIC_API_KEY`로 설정된 경우 최초 사용 시 1회성 키 승인 프롬프트가 뜨는 것이 정상입니다. `ANTHROPIC_AUTH_TOKEN`의 경우 프롬프트 없이 변수가 자동으로 적용되며, 이전에 저장된 claude.ai 로그인은 해당 세션 동안 비활성화됩니다.

조직에서 [fast mode](/docs/en/fast-mode)를 사용하는 경우 여기서 `/fast`도 실행해 보세요: 사용 가능 여부 확인은 게이트웨이 베이스 URL을 따르지 않고 `api.anthropic.com`을 직접 호출하므로, 게이트웨이 라우팅 세션에서 추론이 동작하더라도 fast mode가 이용 불가 또는 비활성화 상태로 보고될 수 있습니다. [프록시 및 LLM 게이트웨이 뒤에서 fast mode 사용하기](/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways)에서 이를 복원하는 변수를 확인하여 [나머지 구성 항목](#distribute-the-configuration)과 함께 배포하세요.

마지막으로 전송한 메시지에 대해 게이트웨이 로그를 확인하세요: 자격 증명은 개발자를 식별하고, [`x-claude-code-session-id` 헤더](/docs/en/llm-gateway-protocol#request-headers)는 요청을 세션별로 그룹화합니다. [문제 해결 증상](/docs/en/llm-gateway-connect#troubleshoot-gateway-errors)과 함께 기능이 오작동하면 게이트웨이가 헤더를 제거하거나 오류를 다시 쓰고 있는 것입니다; 위의 [게이트웨이 요구사항](#gateway-requirements)을 확인하세요.

## 게이트웨이 유지 관리

배포 후 시간이 지남에 따라 세 가지 유형의 변경사항이 게이트웨이에 영향을 미칩니다. 각 유형별 주시해야 할 증상과 조치 사항은 다음과 같습니다.

| 변경사항                                                                     | 게이트웨이가 대응하지 못했을 때의 증상                                                                                                                    | 조치 사항                                                                                                                                                                                                                                                |
| :--------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 새 Claude Code 릴리스에 `anthropic-beta` 값 및 요청 본문 필드가 추가됨       | 개발자가 Claude Code를 업데이트한 후 새 필드를 지목하는 `400` 오류 발생 보고; [기능 전달](/docs/en/llm-gateway-protocol#feature-pass-through) 참조          | 허용 목록으로 제한하기보다 `anthropic-*` 헤더와 요청 본문을 그대로 전달하세요; 새 Claude Code 릴리스가 개발자에게 도달하기 전에 게이트웨이 대상 테스트를 진행하세요.                                                                                       |
| 새로운 Claude 모델 사용 가능해짐                                             | 새로운 모델 이름을 선택한 개발자에게 `404`가 반환됨; `/model` 선택기에 표시되지 않음                                                                       | 게이트웨이 라우팅 설정에 새 모델 이름을 추가하고 [라우팅 확인](#confirm-the-gateway-routes-your-models)을 다시 실행하세요. `ANTHROPIC_MODEL` 또는 기본 모델 변수를 배포 중이라면 관리형 설정도 업데이트하세요.                                         |
| 자격 증명 만료 또는 교체 필요                                                | 모든 개발자 요청이 업스트림으로부터 `401`을 반환하며 실패함                                                                                                | 게이트웨이의 프로바이더 자격 증명을 자체 일정에 맞춰 교체하세요; 개발자 키는 게이트웨이에서 교체되며, [`apiKeyHelper`](/docs/en/llm-gateway-connect#rotate-credentials-with-apiKeyHelper)를 사용하면 설정 재배포 없이 개발자별 키 교체가 처리됩니다.   |

키별 비율 제한(rate limits)을 설정할 때, 클라이언트가 `Retry-After`를 준수하며 `429` 응답을 포함한 일시적 실패에 대해 지연 재시도를 [최대 10회까지 수행](/docs/en/errors#automatic-retries)할 수 있음을 고려하세요. [프로토콜 참조](/docs/en/llm-gateway-protocol)를 각 Claude Code 릴리스가 전송하는 내용에 대한 표준 계약으로 활용하세요.

## 관련 리소스

* [Claude Code를 LLM 게이트웨이에 연결하기](/docs/en/llm-gateway-connect): 서페이스별 설정 및 개발자에게 제공 가능한 문제 해결 표가 포함된 개발자용 설정 단계
* [게이트웨이 프로토콜 참조](/docs/en/llm-gateway-protocol): 엔드포인트, 전달할 헤더, 기능 전달 표를 다루는 게이트웨이 운영자용 유선 계약
* [설정 파일 및 우선순위](/docs/en/settings#settings-files): 관리형, 프로젝트, 사용자 설정의 결합 방식 및 각 플랫폼별 관리형 파일 위치
* [조직을 위한 Claude Code 설정](/docs/en/admin-setup): 정책 적용, 사용량 가시성, 데이터 처리를 포함하여 게이트웨이가 포함되는 더 넓은 배포 과정
