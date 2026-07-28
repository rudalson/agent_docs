> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 환경 변수 (Environment variables)

> Claude Code 동작을 제어하는 환경 변수 참조 안내서입니다.

환경 변수는 모델 선택, 인증, 요청 라우팅, 기능 토글 등 Claude Code 동작을 제어할 수 있습니다. 동일한 동작 중 다수는 [설정 파일](/docs/en/settings) 필드, [CLI 플래그](/docs/en/cli-reference), 또는 `/model`과 같은 세션 내 명령을 통해 구성할 수도 있습니다.

이 페이지에서는 다음 내용을 다룹니다:

* 셸이나 설정 파일에서 [환경 변수 설정하기](#set-environment-variables)
* 동일한 동작을 여러 방법으로 설정할 수 있을 때 [어떤 값이 적용되는지 확인하기](#precedence)
* [Claude Code가 읽는 변수 찾아보기](#variables)

## 환경 변수 설정하기

셸에서 설정한 변수는 해당 터미널 세션 동안 지속되며, 설정 파일의 변수는 `claude`가 실행될 때마다 적용됩니다.

### 셸에서 설정하기

`claude`를 실행하기 전에 변수를 설정하세요:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    export API_TIMEOUT_MS="1200000"
    claude
    ```

    모든 세션에 설정하려면 `~/.bashrc`, `~/.zshrc` 또는 셸의 프로필 파일에 `export` 줄을 추가하세요.
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    $env:API_TIMEOUT_MS = "1200000"
    claude
    ```

    모든 세션에 설정하려면 `[Environment]::SetEnvironmentVariable("API_TIMEOUT_MS", "1200000", "User")`를 실행하고 새 터미널을 여세요.
  </Tab>

  <Tab title="Windows CMD">
    ```batch theme={null}
    set API_TIMEOUT_MS=1200000
    claude
    ```

    모든 세션에 설정하려면 `setx API_TIMEOUT_MS "1200000"`을 실행하고 새 터미널을 여세요.
  </Tab>
</Tabs>

할당 줄은 성공 시 아무것도 출력하지 않으므로, `claude`를 실행하기 전에 동일한 셸에서 변수를 출력하여 설정되었는지 확인하세요:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    echo $API_TIMEOUT_MS
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    echo $env:API_TIMEOUT_MS
    ```
  </Tab>

  <Tab title="Windows CMD">
    ```batch theme={null}
    echo %API_TIMEOUT_MS%
    ```
  </Tab>
</Tabs>

### 설정 파일에서 설정하기

`settings.json` 파일의 `env` 키 아래에 변수를 추가하고, 파일이 없는 경우 생성하세요. Claude Code는 파일에서 이를 직접 읽으므로 `claude`가 어떻게 실행되든 효과가 적용됩니다. 실행 중인 세션은 파일을 저장할 때 새 값과 변경된 값을 환경에 적용하지만, [OpenTelemetry 모니터링](/docs/en/monitoring-usage)과 같이 시작 시 변수를 한 번만 읽는 기능은 다시 시작할 때까지 시작 값을 유지합니다. 파일에서 변수를 제거하더라도 실행 중인 세션에서는 설정 해제되지 않으며, 제거는 다음에 `claude`를 실행할 때 적용됩니다.

```json ~/.claude/settings.json theme={null}
{
  "env": {
    "API_TIMEOUT_MS": "1200000",
    "BASH_DEFAULT_TIMEOUT_MS": "300000"
  }
}
```

선택한 파일에 따라 변수가 적용되는 대상이 결정됩니다:

| 파일                          | 적용 대상                                                                    |
| :---------------------------- | :---------------------------------------------------------------------------- |
| `~/.claude/settings.json`     | 모든 프로젝트에서 본인에게 적용                                              |
| `.claude/settings.json`       | 프로젝트에서 작업하는 모든 사람에게 적용되며 버전 제어에 체크인됨            |
| `.claude/settings.local.json` | 이 프로젝트에서만 본인에게 적용 (수동 생성 시 gitignore에 추가하세요)         |
| Managed settings              | 관리자가 배포하여 조직의 모든 사람에게 적용                                 |

각 파일의 위치는 [설정 파일](/docs/en/settings#settings-files)을, 둘 이상의 파일이 동일한 변수를 설정할 때 결합되는 방식은 [설정 우선순위](/docs/en/settings#settings-precedence)를 참조하세요.

## 우선순위

동일한 동작에 환경 변수와 설정 필드가 모두 있는 경우 환경 변수가 우선합니다. 예를 들어 `ANTHROPIC_MODEL`은 `model` 설정을 재정의하고, `CLAUDE_CODE_AUTO_CONNECT_IDE`는 `autoConnectIde`를 재정의합니다. 환경 변수가 설정되지 않은 경우 설정 필드가 적용됩니다.

동일한 변수가 셸과 설정 파일 `env` 블록 모두에 설정된 경우 설정 파일 값이 적용됩니다. Claude Code는 시작할 때와 파일이 변경될 때마다 각 `env` 항목을 프로세스 환경에 작성하여 셸에서 상속된 값을 교체합니다. 몇 가지 변수는 예외 처리되며; [`env` 설정](/docs/en/settings#available-settings)에 예외 사항이 나열되어 있습니다.

설정 파일 간에는 `env` 값이 [설정 우선순위](/docs/en/settings#settings-precedence)를 따르므로, 관리형 설정 항목이 사용자 또는 프로젝트 설정의 동일한 변수를 재정의합니다.

환경 변수가 CLI 플래그 및 세션 내 명령과 상호 작용하는 방식은 기능에 따라 다릅니다: `--model` 및 `/model`은 `ANTHROPIC_MODEL`을 재정의하는 반면, `CLAUDE_CODE_EFFORT_LEVEL`은 `/effort`를 재정의합니다. 변수가 다른 구성 소스와 상호 작용할 때 [변수 목록](#variables)의 해당 행에 우선순위가 명시되거나 이를 설명하는 페이지로 링크됩니다.

Claude Code는 시작 시 셸 환경 변수를 읽으므로 이를 변경하면 다음에 `claude`를 실행할 때 적용됩니다. 설정 파일의 `env` 키 아래에 설정된 변수는 [설정 파일에서 설정하기](#in-settings-files)에 설명된 시작 전용 예외를 제외하고 파일이 변경될 때 실행 중인 세션에 다시 적용됩니다.

## 변수 (Variables)

타임아웃, 토큰 예산, 재시도 횟수와 같은 숫자 변수는 변수 행에 일반 숫자만 사용할 수 있다고 명시된 경우를 제외하고 일반 숫자 외에 지수 표기법(scientific notation) 및 숫자 구분 기호 표기법을 허용합니다. 예를 들어 Claude Code는 `2e3`을 2000으로, `64_000`을 64000으로 읽습니다. v2.1.211 이전에는 이러한 표기법이 `1e6`으로 타임아웃을 1로 설정하는 것과 같이 훨씬 작은 값을 조용히 설정할 수 있었습니다.

| 변수                                                | 목적                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ANTHROPIC_API_KEY`                                     | `X-Api-Key` 헤더로 전송되는 API 키입니다. 설정되면 로그인되어 있더라도 Claude Pro, Max, Team 또는 Enterprise 구독 대신 이 키가 사용됩니다. 비대화형 모드(`-p`)에서는 존재할 때 항상 키가 사용됩니다. 대화형 모드에서는 구독을 재정의하기 전에 키를 한 번 승인하도록 요청받습니다. 대신 구독을 사용하려면 `unset ANTHROPIC_API_KEY`를 실행하세요 |
| `ANTHROPIC_AUTH_TOKEN`                                  | `Authorization` 헤더의 사용자 지정 값입니다 (여기에서 설정한 값 앞에 `Bearer `가 붙습니다) |
| `ANTHROPIC_AWS_API_KEY`                                 | AWS 콘솔에서 생성된 [Claude Platform on AWS](/docs/en/claude-platform-on-aws)용 워크스페이스 API 키입니다. `x-api-key`로 전송되며 AWS SigV4보다 우선합니다 |
| `ANTHROPIC_AWS_BASE_URL`                                | [Claude Platform on AWS](/docs/en/claude-platform-on-aws) 엔드포인트 URL을 재정의합니다. 커스텀 리전이나 [LLM 게이트웨이](/docs/en/llm-gateway)를 통해 라우팅할 때 사용하세요. 기본값은 `https://aws-external-anthropic.{AWS_REGION}.api.aws`입니다 |
| `ANTHROPIC_AWS_WORKSPACE_ID`                            | [Claude Platform on AWS](/docs/en/claude-platform-on-aws)에 필수적입니다. 모든 요청에 `anthropic-workspace-id` 헤더로 전송됩니다 |
| `ANTHROPIC_BASE_URL`                                    | 프록시나 게이트웨이를 통해 요청을 라우팅하기 위해 API 엔드포인트를 재정의합니다. 퍼스트 파티가 아닌 호스트로 설정하면 [MCP 도구 검색](/docs/en/mcp#scale-with-mcp-tool-search)이 기본적으로 비활성화됩니다. 프록시가 `tool_reference` 블록을 전달하는 경우 `ENABLE_TOOL_SEARCH=true`를 설정하세요. v2.1.196부터 [Remote Control](/docs/en/remote-control#requirements)은 Amazon Bedrock, Google Cloud's Agent Platform 및 Microsoft Foundry에서의 동작과 동일하게 `api.anthropic.com` 이외의 호스트를 가리킬 때 비활성화됩니다 |
| `ANTHROPIC_BEDROCK_BASE_URL`                            | Amazon Bedrock 엔드포인트 URL을 재정의합니다. 커스텀 Amazon Bedrock 엔드포인트나 [LLM 게이트웨이](/docs/en/llm-gateway)를 통해 라우팅할 때 사용하세요. [Amazon Bedrock](/docs/en/amazon-bedrock)을 참조하세요 |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL`                     | Amazon Bedrock Mantle 엔드포인트 URL을 재정의합니다. [Mantle 엔드포인트](/docs/en/amazon-bedrock#use-the-mantle-endpoint)를 참조하세요 |
| `ANTHROPIC_BEDROCK_SERVICE_TIER`                        | Amazon Bedrock [서비스 티어](https://docs.aws.amazon.com/bedrock/latest/userguide/service-tiers-inference.html) (`default`, `flex`, 또는 `priority`). `X-Amzn-Bedrock-Service-Tier` 헤더로 전송됩니다. [Amazon Bedrock](/docs/en/amazon-bedrock#service-tiers)을 참조하세요 |
| `ANTHROPIC_BETAS`                                       | API 요청에 포함할 추가 `anthropic-beta` 헤더 값의 쉼표 구획 목록입니다. Claude Code는 이미 필요한 베타 헤더를 보내며, Claude Code가 기본 지원을 추가하기 전에 [Anthropic API 베타](https://platform.claude.com/docs/en/api/beta-headers)에 동의할 때 사용합니다. API 키 인증이 필요한 [`--betas` 플래그](/docs/en/cli-reference#cli-flags)와 달리 이 변수는 Claude.ai 구독을 포함한 모든 인증 방법에서 작동합니다 |
| `ANTHROPIC_CUSTOM_HEADERS`                              | 요청에 추가할 사용자 지정 헤더입니다 (`Name: Value` 형식, 여러 헤더는 줄바꿈으로 구획) |
| `ANTHROPIC_CUSTOM_MODEL_OPTION`                         | `/model` 선택기에 커스텀 항목으로 추가할 모델 ID입니다. 내장 별칭을 바꾸지 않고 비표준 또는 게이트웨이 전용 모델을 선택할 수 있도록 할 때 사용하세요. [모델 구성](/docs/en/model-config#add-a-custom-model-option)을 참조하세요 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION`             | `/model` 선택기의 커스텀 모델 항목에 대한 표시 설명입니다. 설정되지 않은 경우 기본값은 `Custom model (<model-id>)`입니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME`                    | `/model` 선택기의 커스텀 모델 항목에 대한 표시 이름입니다. 설정되지 않은 경우 기본값은 모델 ID입니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES`  | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL`                         | [모델 구성](/docs/en/model-config#environment-variables)을 참조하세요 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_DESCRIPTION`             | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_NAME`                    | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_SUPPORTED_CAPABILITIES`  | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL`                         | [모델 구성](/docs/en/model-config#environment-variables)을 참조하세요 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION`             | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME`                    | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES`  | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL`                          | [모델 구성](/docs/en/model-config#environment-variables)을 참조하세요 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION`              | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME`                     | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES`   | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL`                        | [모델 구성](/docs/en/model-config#environment-variables)을 참조하세요 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION`            | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME`                   | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | [모델 구성](/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_FOUNDRY_API_KEY`                             | Microsoft Foundry 인증용 API 키입니다 ([Microsoft Foundry](/docs/en/microsoft-foundry) 참조) |
| `ANTHROPIC_FOUNDRY_AUTH_TOKEN`                          | Microsoft Entra 액세스 토큰과 같은 Microsoft Foundry 인증용 Bearer 토큰입니다. Claude Code는 이를 `Authorization: Bearer` 헤더로 전송합니다. `ANTHROPIC_FOUNDRY_API_KEY` 및 Azure 기본 자격 증명 체인보다 우선합니다. [Microsoft Foundry](/docs/en/microsoft-foundry)를 참조하세요. Claude Code v2.1.203 이상이 필요합니다 |
| `ANTHROPIC_FOUNDRY_BASE_URL`                            | Microsoft Foundry 리소스의 전체 기본 URL입니다 (예: `https://my-resource.services.ai.azure.com/anthropic`). `ANTHROPIC_FOUNDRY_RESOURCE`에 대한 대안입니다 ([Microsoft Foundry](/docs/en/microsoft-foundry) 참조) |
| `ANTHROPIC_FOUNDRY_RESOURCE`                            | Microsoft Foundry 리소스 이름입니다 (예: `my-resource`). `ANTHROPIC_FOUNDRY_BASE_URL`이 설정되지 않은 경우 필수적입니다 ([Microsoft Foundry](/docs/en/microsoft-foundry) 참조) |
| `ANTHROPIC_MODEL`                                       | 사용할 모델 설정의 이름입니다 ([모델 구성](/docs/en/model-config#environment-variables) 참조) |
| `ANTHROPIC_SMALL_FAST_MODEL`                            | \[사용 중단됨] [백그라운드 작업용 Haiku 클래스 모델](/docs/en/costs)의 이름 |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION`                 | Amazon Bedrock 또는 Amazon Bedrock Mantle 사용 시 Haiku 클래스 모델의 AWS 리전을 재정의합니다. Amazon Bedrock에서는 `ANTHROPIC_DEFAULT_HAIKU_MODEL` 또는 사용 중단된 `ANTHROPIC_SMALL_FAST_MODEL`도 설정된 경우에만 적용됩니다 |
| `ANTHROPIC_VERTEX_BASE_URL`                             | Google Cloud's Agent Platform 엔드포인트 URL을 재정의합니다. 커스텀 Google Cloud's Agent Platform 엔드포인트나 [LLM 게이트웨이](/docs/en/llm-gateway)를 통해 라우팅할 때 사용하세요. [Google Cloud's Agent Platform](/docs/en/google-vertex-ai)을 참조하세요 |
| `ANTHROPIC_VERTEX_PROJECT_ID`                           | Google Cloud's Agent Platform 요청용 GCP 프로젝트 ID입니다. `GCLOUD_PROJECT`, `GOOGLE_CLOUD_PROJECT` 또는 `GOOGLE_APPLICATION_CREDENTIALS` 자격 증명 파일에 있는 프로젝트에 의해 재정의됩니다 |
| `ANTHROPIC_WORKSPACE_ID`                                | [워크로드 정체성 연동](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)용 워크스페이스 ID입니다 |
| `API_FORCE_IDLE_TIMEOUT`                                | 스트리밍 모델 응답에서 바이트가 들어오지 않을 때 응답을 중단하는 5분 유휴 타임아웃을 재정의합니다. `0`으로 설정하면 타임아웃을 비활성화하고 `1`로 설정하면 모든 제공업체에서 타임아웃을 유지합니다 |
| `API_TIMEOUT_MS`                                        | 밀리초 단위의 API 요청 타임아웃입니다 (기본값: 600000 또는 10분; 최대: 2147483647). 느린 네트워크에서 요청 시간이 초과되거나 프록시를 통해 라우팅할 때 이를 늘리세요 |
| `AWS_BEARER_TOKEN_BEDROCK`                              | 인증용 Amazon Bedrock API 키입니다 |
| `BASH_DEFAULT_TIMEOUT_MS`                               | 장시간 실행되는 bash 명령의 기본 타임아웃입니다 (기본값: 120000 또는 2분) |
| `BASH_MAX_OUTPUT_LENGTH`                                | 전체 출력이 파일로 저장되고 Claude가 경로 및 짧은 미리보기를 받기 전까지의 bash 출력 최대 문자 수입니다 |
| `BASH_MAX_TIMEOUT_MS`                                   | 모델이 장시간 실행되는 bash 명령에 설정할 수 있는 최대 타임아웃입니다 (기본값: 600000 또는 10분) |
| `CCR_FORCE_BUNDLE`                                      | GitHub 접근이 가능할 때에도 [`claude --cloud`](/docs/en/claude-code-on-the-web#send-local-repositories-without-github)가 로컬 리포지토리를 번들로 묶어 업로드하도록 강제하려면 `1`로 설정하세요 |
| `CLAUDECODE`                                            | Claude Code가 생성하는 하위 프로세스에 `1`로 설정됩니다 |
| `CLAUDE_AFK_COUNTDOWN_MS`                               | 응답하지 않은 [`AskUserQuestion`](/docs/en/tools-reference) 대화 상자에 자동 계속 화면 카운트다운이 표시되기까지의 밀리초 수입니다 (기본값: `20000`) |
| `CLAUDE_AFK_TIMEOUT_MS`                                 | 응답하지 않은 [`AskUserQuestion`](/docs/en/tools-reference) 대화 상자가 자동으로 계속되기 전까지의 유휴 시간(밀리초)입니다 |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS`               | Explore 및 Plan과 같은 모든 내장 [서브에이전트](/docs/en/sub-agents) 유형을 비활성화하려면 `1`로 설정하세요. 비대화형 모드(`-p` 플래그)에서만 적용됩니다 |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX`                        | SDK에서 생성된 MCP 서버의 도구 이름에서 `mcp__<server>__` 접두사를 건너뛰려면 `1`로 설정하세요 |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS`                   | 백그라운드 서브에이전트의 지연 타임아웃(밀리초)입니다 (기본값 `600000` 또는 10분) |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`                       | 자동 압축이 트리거되는 자동 압축 창의 백분율(1-100)을 설정합니다 |
| `CLAUDE_AUTO_BACKGROUND_TASKS`                          | 장시간 실행되는 에이전트 작업의 자동 백그라운드 전환을 강제 활성화하려면 `1`로 설정하세요 |
| `CLAUDE_AX_SCREEN_READER`                               | 스크린 리더 친화적인 출력을 렌더링하려면 `1`로 설정하세요 |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR`              | 메인 세션의 각 Bash 또는 PowerShell 명령 후에 원래 작업 디렉터리로 돌아갑니다 |
| `CLAUDE_CLIENT_PRESENCE_FILE`                           | 화면을 잠금 해제할 때 외부 도구가 생성하고 잠글 때 삭제하는 파일 경로입니다 |
| `CLAUDE_CODE_ACCESSIBILITY`                             | 네이티브 터미널 커서를 표시 상태로 유지하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD`          | `--add-dir`로 지정된 디렉터리에서 메모리 파일을 로드하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT`                   | [전체 화면 렌더링](/docs/en/fullscreen)에서 모든 프레임마다 전체 화면을 다시 그리려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ALWAYS_ENABLE_EFFORT`                      | 모든 요청과 함께 [effort](/docs/en/model-config#adjust-effort-level) 매개변수를 전송하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`                     | 자격 증명을 새로 고쳐야 하는 간격(밀리초)입니다 |
| `CLAUDE_CODE_ARTIFACT_AUTO_OPEN`                        | 새 [아티팩트](/docs/en/artifacts)가 게시될 때 브라우저를 자동으로 열지 않으려면 `0`으로 설정하세요 |
| `CLAUDE_CODE_ATTRIBUTION_HEADER`                        | 시스템 프롬프트 시작 부분에서 귀속 블록을 생략하려면 `0`으로 설정하세요 |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW`                       | 자동 압축 계산에 사용되는 토큰 단위의 컨텍스트 용량을 설정합니다 |
| `CLAUDE_CODE_AUTO_CONNECT_IDE`                          | 자동 [IDE 연결](/docs/en/vs-code)을 재정의합니다 |
| `CLAUDE_CODE_AWS_CHAIN_RESOLVE_TIMEOUT_MS`              | AWS 기본 자격 증명 제공자 체인이 자격 증명을 생성할 때까지 Claude Code가 기다리는 시간(밀리초)입니다 (기본값: `60000`) |
| `CLAUDE_CODE_BRIDGE_SESSION_ID`                         | 세션에 활성 [Remote Control](/docs/en/remote-control) 연결이 있는 동안 하위 프로세스에 자동으로 설정됩니다 |
| `CLAUDE_CODE_CERT_STORE`                                | TLS 연결을 위한 CA 인증서 소스의 쉼표 구획 목록입니다 (기본값 `bundled,system`) |
| `CLAUDE_CODE_CHILD_SESSION`                             | 하위 프로세스에 `1`로 설정되어 중첩된 세션을 구별합니다 |
| `CLAUDE_CODE_CLIENT_CERT`                               | mTLS 인증을 위한 클라이언트 인증서 파일 경로입니다 |
| `CLAUDE_CODE_CLIENT_KEY`                                | mTLS 인증을 위한 클라이언트 개인 키 파일 경로입니다 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE`                     | 암호화된 CLAUDE_CODE_CLIENT_KEY용 암호입니다 |
| `CLAUDE_CODE_DEBUG_LOGS_DIR`                            | 디버그 로그 파일 경로를 재정의합니다 |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL`                           | 디버그 로그 파일에 기록되는 최소 로그 레벨입니다 |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT`                        | [1M 컨텍스트 창](/docs/en/model-config#extended-context) 지원을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`                 | [적응형 추론](/docs/en/model-config#adjust-effort-level)을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_ADVISOR_TOOL`                      | [advisor 도구](/docs/en/advisor)를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW`                        | [백그라운드 에이전트 및 에이전트 뷰](/docs/en/agent-view)를 끄려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN`                  | [전체 화면 렌더링](/docs/en/fullscreen)을 비활성화하고 클래식 메인 화면 렌더러를 사용하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_ARTIFACT`                          | [Artifact](/docs/en/artifacts) 도구를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS`                       | 첨부 파일 처리를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY`                       | [자동 메모리](/docs/en/memory#auto-memory)를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`                  | 모든 백그라운드 작업 기능을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_GUARD`        | Amazon Bedrock 스트리밍 응답이 특정 content-type을 전달하는지 확인하는 검사를 건너뛰려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_BG_EXIT_HANDOFF`                   | 감독 프로세스가 중지/재시작될 때 백그라운드 세션의 실행 항목 전달을 중지하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP`            | 메모리 압박이 있을 때 백그라운드 셸 명령을 종료하는 것을 중지하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS`                    | Claude Code에 포함된 스킬 및 워크플로를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS`                        | 모든 CLAUDE.md 메모리 파일이 컨텍스트로 로드되지 않도록 하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_CRON`                              | [예약된 작업](/docs/en/scheduled-tasks)을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`                | API 요청에서 베타 헤더 및 도구 스키마 필드를 제거하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS`               | 내장 [Explore 및 Plan 서브에이전트](/docs/en/sub-agents#built-in-subagents)를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_FAST_MODE`                         | [패스트 모드](/docs/en/fast-mode)를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY`                   | 세션 품질 설문 조사를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING`                | 파일 [체크포인팅](/docs/en/checkpointing)을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS`                  | 커밋 및 PR 워크플로 지침을 제거하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP`                | 구형 모델을 현재 모델 버전으로 자동 재매핑하는 것을 방지하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_MOUSE`                             | 전체 화면 렌더링에서 마우스 추적을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_MOUSE_CLICKS`                      | 전체 화면 렌더링에서 클릭, 드래그 및 호버 처리를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`              | 불필요한 네트워크 트래픽을 비활성화하려면 비어 있지 않은 값을 설정하세요 |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK`             | 스트리밍 요청 실패 시 비스트리밍 폴백을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_NOTIFICATION_PRESENCE_CHECK`       | 터미널을 사용하는 동안에도 데스크톱 알림을 보내려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL`  | 첫 실행 시 공식 플러그인 마켓플레이스 자동 추가를 건너뛰려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS`                     | 시스템 차원의 관리형 스킬 디렉터리에서 스킬 로드를 건너뛰려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE`                    | 대화 컨텍스트에 따른 자동 터미널 제목 업데이트를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_THINKING`                          | API 요청에서 `thinking` 매개변수를 완전히 생략하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL`                    | 전체 화면 렌더링에서 가상 스크롤을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_DISABLE_WORKFLOWS`                         | [워크플로](/docs/en/workflows#turn-workflows-off)를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_EFFORT_LEVEL`                              | 지원되는 모델의 노력 수준(effort level)을 설정합니다 (`low`, `medium`, `high`, `xhigh`, `max`, 또는 `auto`) |
| `CLAUDE_CODE_ENABLE_APPEND_SUBAGENT_PROMPT`             | 모든 서브에이전트 프롬프트 끝에 추가 텍스트를 붙이려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ENABLE_AUTO_MODE`                          | 호환성을 위해 수용되며 효과가 없습니다 |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY`                       | 세션 요약(recap) 가용성을 재정의합니다 |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH`          | 비대화형 모드에서 플러그인 상태를 새로 고치려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL`           | 세션 품질 설문 조사를 자체 OpenTelemetry 수집기로 라우팅하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING`        | 도구 호출 입력이 생성될 때 API에서 스트리밍되는지 제어합니다 |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY`            | 게이트웨이의 `/v1/models` 엔드포인트에서 `/model` 선택기를 채우려면 `1`로 설정하세요 |
