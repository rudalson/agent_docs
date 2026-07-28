> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude Code를 LLM 게이트웨이에 연결하기

> 조직의 LLM 게이트웨이로 Claude Code의 방향을 지정합니다. 관리자가 이미 구성했는지 확인하거나 베이스 URL 및 자격 증명을 직접 설정한 후 연결을 검증하고 게이트웨이 오류를 수정합니다.

[LLM 게이트웨이](/docs/en/llm-gateway)는 조직이 Claude Code와 모델 프로바이더 사이에 구동하는 프록시입니다. 조직에서 게이트웨이를 사용하는 경우, Claude Code는 개인 claude.ai 로그인 대신 조직에서 발급한 자격 증명으로 게이트웨이에 인증합니다.

이 페이지는 조직이 운영하는 게이트웨이를 통해 Claude Code를 구동하는 개발자를 위한 문서입니다. 다음 두 가지 경로를 다룹니다: [관리자가 이미 구성했는지 확인하기](#check-for-an-existing-configuration), 그리고 아직 구성되지 않은 경우 [직접 구성하기](#configure-claude-code-yourself).

<Note>
  * 조직을 위한 게이트웨이를 배포하려면 [LLM 게이트웨이 배포하기](/docs/en/llm-gateway-rollout)를 참조하세요.
  * Claude Code가 게이트웨이로 전송하는 내용에 대해서는 [게이트웨이 프로토콜 참조](/docs/en/llm-gateway-protocol)를 참조하세요.
</Note>

## 기존 구성 확인하기

관리자는 [관리형 설정](/docs/en/settings#settings-files), 장치 관리, 또는 [`apiKeyHelper`](#rotate-credentials-with-apikeyhelper)를 통해 게이트웨이 주소와 자격 증명을 배포하여 사용자가 별도로 설정하지 않아도 시작 시 Claude Code가 자동으로 인식하도록 할 수 있습니다. 조직에서 이미 이를 구성했는지 확인하려면 다음 단계를 수행하세요:

<Steps>
  <Step title="Claude Code 시작">
    `claude`를 실행합니다. 세션 대신 로그인 화면이 나타나면 게이트웨이 자격 증명이 배포되지 않은 것이므로, 아래의 [직접 구성하기](#configure-claude-code-yourself)를 진행하세요.
  </Step>

  <Step title="Status 탭 확인">
    로그인 화면 없이 세션이 시작되면 **Status** 탭을 여는 `/status`를 실행하고 두 항목을 확인하세요:

    * `Anthropic base URL`: 이 항목은 게이트웨이 주소가 설정된 경우에만 표시됩니다. 표시되지 않으면 Claude Code가 게이트웨이를 가리키고 있지 않은 것이므로, 아래의 [직접 구성하기](#configure-claude-code-yourself)를 진행하세요.
    * `Auth token` 또는 `API key`: `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_API_KEY` 또는 `apiKeyHelper`가 표시되면 게이트웨이 자격 증명이 활성화되어 있음을 의미합니다. 대신 claude.ai 계정을 지정하는 `Login method` 항목이 보인다면 자격 증명이 배포되지 않은 것이므로 [자격 증명 직접 설정하기](#set-the-credential-variable)를 진행하세요.
  </Step>

  <Step title="테스트 메시지 전송">
    `/status` 메뉴를 닫고 Claude Code에서 임의의 프롬프트를 전송합니다. 오류 없이 정상적인 응답이 오면 게이트웨이 연결이 동작하는 것입니다.
  </Step>
</Steps>

`/status` 메뉴의 두 항목이 제대로 표시되지만 Claude에 메시지를 보낼 때 실패하는 경우 [게이트웨이 문제 해결 표](#troubleshoot-gateway-errors)를 참조하세요.

## Claude Code 직접 구성하기

게이트웨이에 맞춰 Claude Code를 직접 구성하려면 게이트웨이 담당 팀으로부터 다음 정보를 전달받아야 합니다:

* 게이트웨이의 베이스 URL (Base URL)
* 자격 증명: 키/토큰 문자열, 또는 이를 가져오는 명령
  * 게이트웨이 팀에서 자격 증명의 종류를 밝히지 않은 경우 아래의 [자격 증명 변수 설정](#set-the-credential-variable) 섹션을 참조하여 어떤 것을 시도할지 확인하세요.

아래 섹션에서는 구성 순서를 다룹니다:

* [자격 증명 변수 설정](#set-the-credential-variable) 및 [베이스 URL 설정](#set-the-base-url-and-credential): 모든 게이트웨이 연결에 필요한 두 가지 변수
* [연결 검증](#verify-the-connection): 설정을 저장하기 전에 연결이 정상 작동하는지 확인
* [각 서페이스 구성](#configure-each-surface): Claude Code CLI 외에 VS Code 등 다른 서페이스(surface)를 사용하는 경우 게이트웨이 자격 증명을 적용하는 방법
* [추가 구성](#additional-configuration): 베이스 URL과 자격 증명 외에 일부 게이트웨이에서 필요한 변수(커스텀 헤더, 자격 증명 헬퍼, 모델 탐색, 프로바이더 형식 베이스 URL, 게이트웨이 경로 외 트래픽 비활성화 등). 관리자가 지정했거나 네트워크 외부 출력이 제한된 경우에만 설정하세요.

### 자격 증명 변수 설정

Claude Code를 게이트웨이에 인증하려면 환경 변수에 자격 증명을 설정하세요. 게이트웨이 팀에서 가이드한 내용에 따라 사용할 변수가 결정됩니다:

| 설정할 변수 위치                                        | 사용 조건                                                       |
| :------------------------------------------------------ | :-------------------------------------------------------------- |
| `ANTHROPIC_AUTH_TOKEN`                                  | 게이트웨이 팀이 "bearer token" 또는 "Authorization header"라고 지정한 경우 |
| `ANTHROPIC_API_KEY`                                     | 게이트웨이 팀이 "API key" 또는 "x-api-key"라고 지정한 경우        |
| [`apiKeyHelper`](#rotate-credentials-with-apikeyhelper) | 자격 증명이 주기적으로 교체되거나 볼트(vault)에서 로드되는 경우   |

어떤 종류인지 안내받지 못했다면 `ANTHROPIC_AUTH_TOKEN`을 사용하세요. 아래의 [검증 요청](#verify-the-connection) 섹션에서 변경이 필요한지 확인하는 방법을 안내합니다.

### 베이스 URL 및 자격 증명 설정

게이트웨이 베이스 URL과 위에서 선택한 자격 증명 변수를 환경 변수로 설정하세요. 예시에서는 `ANTHROPIC_AUTH_TOKEN`을 사용하였으며, [선택한 변수](#set-the-credential-variable)가 `ANTHROPIC_API_KEY`인 경우 바꾸어 적용하세요. 단일 터미널 세션 동안 유지되는 [셸 환경 변수로 설정](#set-as-shell-environment-variables)하거나, Claude Code가 실행되는 모든 곳에 지속 적용되는 [Claude Code 설정 파일에 설정](#set-in-a-settings-file)할 수 있습니다.

처음 연결할 때는 셸 export로 시작하여 [검증 요청](#verify-the-connection)을 실행한 후 설정 파일로 값을 옮기는 것이 좋습니다.

#### 셸 환경 변수로 설정

게이트웨이 팀에서 제공한 값으로 대체하세요:

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_BASE_URL=https://llm-gateway.example.com
    export ANTHROPIC_AUTH_TOKEN=sk-gateway-key
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_BASE_URL = "https://llm-gateway.example.com"
    $env:ANTHROPIC_AUTH_TOKEN = "sk-gateway-key"
    ```
  </Tab>
</Tabs>

셸 export는 해당 터미널 세션 및 해당 터미널에서 시작된 프로그램에만 적용됩니다. 독(dock)이나 시작 메뉴에서 실행된 에디터에는 전달되지 않습니다. 새 터미널에서도 지속되도록 하려면 `~/.zshrc`, `~/.bashrc` 또는 PowerShell `$PROFILE`과 같은 셸 프로필 파일에 동일한 줄을 추가하세요.

셸에만 게이트웨이를 export하는 경우 [슈퍼바이저](/docs/en/agent-view#how-background-sessions-are-hosted)가 호스팅하는 백그라운드 에이전트에 안정적으로 전달되지 않을 수 있습니다; [각 백그라운드 세션이 게이트웨이를 소싱하는 방법](/docs/en/agent-view#the-supervisor-process)을 참조하세요. 백그라운드 에이전트가 항상 거쳐야 하는 게이트웨이의 경우 설정 파일을 사용하세요.

#### 설정 파일에 설정

[백그라운드 에이전트](/docs/en/agent-view#how-background-sessions-are-hosted)를 포함하여 Claude Code가 실행되는 모든 곳에 적용되도록 하려면 셸에 의존하는 대신 [설정 파일](/docs/en/settings)의 `env` 블록에 변수를 설정하세요. 설정 파일은 범위에 따라 다릅니다:

* `~/.claude/settings.json`은 모든 프로젝트에 적용됩니다. Windows의 경우 경로가 `%USERPROFILE%\.claude\settings.json`입니다.
* `.claude/settings.local.json`은 단일 프로젝트에 적용됩니다. Claude Code가 파일을 생성할 때 자동으로 gitignore에 추가하지만, 직접 생성하는 경우 실수로 자격 증명을 커밋하지 않도록 사전에 gitignore에 추가하세요.

<Warning>
  프로젝트의 `.claude/settings.json`에는 자격 증명을 절대 넣지 마세요. 이 파일은 저장소를 복제하는 모든 사람과 공유 및 커밋됩니다.
</Warning>

두 파일 모두 `env` 블록의 구조는 동일합니다:

```json theme={null}
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://llm-gateway.example.com",
    "ANTHROPIC_AUTH_TOKEN": "sk-gateway-key"
  }
}
```

셸 export와 설정 파일의 `env` 블록에 동일한 변수가 설정된 경우 설정 파일의 값이 적용됩니다. `/status`를 실행하여 어떤 베이스 URL과 자격 증명 소스가 사용 중인지 확인하세요.

### 연결 검증

셸에 변수를 export한 상태에서 게이트웨이에 직접 1토큰 요청을 보내보세요. 이렇게 하면 Claude Code를 열기 전에 URL과 자격 증명이 작동하는지 먼저 확인할 수 있으므로, 실패 시 설정 문제가 아닌 게이트웨이 문제임을 바로 알 수 있습니다. 아래 명령은 셸 변수를 읽으므로 설정 파일에 입력했더라도 [셸 export](#set-as-shell-environment-variables)가 필요합니다.

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    curl -X POST "$ANTHROPIC_BASE_URL/v1/messages" \
      -H "Authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
      -H "anthropic-version: 2023-06-01" \
      -H "content-type: application/json" \
      -d '{"model": "claude-sonnet-4-6", "max_tokens": 1, "messages": [{"role": "user", "content": "."}]}'
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    Invoke-RestMethod -Method Post -Uri "$env:ANTHROPIC_BASE_URL/v1/messages" `
      -Headers @{ "Authorization" = "Bearer $env:ANTHROPIC_AUTH_TOKEN"; "anthropic-version" = "2023-06-01" } `
      -ContentType "application/json" `
      -Body '{"model": "claude-sonnet-4-6", "max_tokens": 1, "messages": [{"role": "user", "content": "."}]}'
    ```
  </Tab>
</Tabs>

게이트웨이가 `x-api-key` 헤더를 요구하는 경우 Bash 명령에서는 `Authorization` 헤더를 `x-api-key: $ANTHROPIC_API_KEY`로 바꾸고, PowerShell 명령에서는 `"Authorization"` 해시테이블 항목을 `"x-api-key" = "$env:ANTHROPIC_API_KEY"`로 변경하세요.

`{"id":"msg_`로 시작하고 `"content":[...]` 필드가 포함된 JSON 응답이 반환되면 게이트웨이에 접근이 가능하며 자격 증명이 올바르게 동작하는 것입니다. 알 수 없는 모델(unknown model) 오류가 발생해도 게이트웨이가 모델 이름을 거부하기 전에 요청을 정상 인증했음을 의미하므로 URL 및 자격 증명이 작동함이 입증됩니다. 테스트를 위해 게이트웨이가 지원하는 모델을 찾아낼 필요는 없습니다. `401` 오류는 자격 증명이 거부된 것이므로, 변수를 임의로 선택했던 경우 다른 변수로 바꾸어 재export하세요.

#### Claude Code에서 확인

동일한 셸에서 `claude`를 시작하여 export된 변수를 상속받은 후 메시지를 전송하고 `/status`를 실행하세요.

**Status** 탭에서 `Anthropic base URL` 항목에 게이트웨이 주소가 표시되면 요청이 게이트웨이로 라우팅되는 것입니다. 이 줄이 없다면 변수가 세션에 전달되지 않은 것입니다. 설정한 변수명이 명시된 `Auth token` 또는 `API key` 항목이 표시되면 저장된 claude.ai 로그인 대신 게이트웨이 자격 증명이 사용 중임을 나타냅니다.

메시지 전송이 실패하거나 `/status`에 게이트웨이 URL이 표시되지 않으면 아래의 [게이트웨이 문제 해결 표](#troubleshoot-gateway-errors)를 참조하세요.

### 자격 증명 변수의 헤더 매핑 방식

각 변수는 다음과 같이 서로 다른 HTTP 헤더로 자격 증명을 전송합니다: `ANTHROPIC_AUTH_TOKEN`은 `Authorization: Bearer`로, `ANTHROPIC_API_KEY`는 `x-api-key`로, 그리고 `apiKeyHelper`는 둘 다 전송합니다. 잘못된 변수에 자격 증명을 지정하면 게이트웨이가 읽지 않는 헤더로 전달되어 `401` 오류가 발생합니다. 검증 요청에서 `401`이 반환되면 다른 변수로 전환하여 시도해 보세요.

### 기존 로그인과의 충돌

게이트웨이 자격 증명 변수는 저장된 claude.ai 로그인이나 Console 키보다 우선 적용됩니다. 변수가 설정되어 있는 동안 저장된 claude.ai 로그인은 유지되지만 사용되지 않으며, 변수 설정을 해제하면 Claude Code가 다시 기존 로그인 방식으로 돌아갑니다. `ANTHROPIC_AUTH_TOKEN`의 경우 변수가 즉시 우선 적용됩니다. `ANTHROPIC_API_KEY`의 경우 인터랙티브 모드에서 키를 승인하라는 프롬프트가 한 번 표시된 후 적용됩니다.

`/status`를 실행하여 어떤 자격 증명 소스가 활성화되어 있는지 확인하세요. 시작 시 두 소스를 명시한 인증 충돌 경고가 나타나면 [게이트웨이 문제 해결 표](#troubleshoot-gateway-errors)의 첫 번째 행을 참조하여 어떤 것을 제거할지 확인하세요. 게이트웨이 자격 증명만 남기도록 저장된 로그인을 지우려면 `/logout`을 실행하세요.

## 각 서페이스 구성

CLI는 위의 환경 변수 및 설정 파일을 읽습니다. 다른 서페이스들(VS Code 확장, 데스크톱 앱, GitHub Actions, Agent SDK, 그리고 Slack 및 웹 등 클라우드 서페이스)에 이러한 설정이 적용되는지 여부는 아래 섹션에서 설명합니다.

### VS Code 확장

[VS Code 확장](/docs/en/vs-code)에 적용할 게이트웨이 변수는 **Preferences: Open User Settings (JSON)** 명령으로 여는 VS Code 전용 사용자 설정의 `claudeCode.environmentVariables`에 설정합니다. 확장은 실행 전 이 설정에서 자격 증명을 확인하므로, 게이트웨이 자격 증명을 설정하는 가장 안정적인 장소입니다. `~/.claude/settings.json` 값은 생성된 프로세스에는 전달되지만 확장 자체의 로그인 확인 단계에는 전달되지 않습니다.

```json theme={null}
{
  "claudeCode.environmentVariables": [
    { "name": "ANTHROPIC_BASE_URL", "value": "https://llm-gateway.example.com" },
    { "name": "ANTHROPIC_AUTH_TOKEN", "value": "sk-gateway-key" }
  ]
}
```

### 데스크톱 앱

데스크톱 앱은 `ANTHROPIC_BASE_URL`이나 `settings.json`이 아닌 [서드파티 추론 설정(third-party inference configuration)](https://claude.com/docs/third-party/claude-desktop/gateway)에서 게이트웨이 라우팅을 읽어옵니다. 해당 설정은 조직에서 배포하거나 앱 내 양식을 통해 입력할 수 있습니다:

* **관리자가 배포한 경우**: 조직에서 [설정을 배포한 경우](/docs/en/llm-gateway-rollout#distribute-through-managed-settings), 별도 설정 없이 데스크톱 앱이 게이트웨이를 거쳐 라우팅됩니다.
* **로컬에서 구성하는 경우**: 관리자가 배포한 설정이 없는 장치에서는 Help → Troubleshooting → Enable Developer Mode를 선택하여 Developer 메뉴가 활성화된 상태로 앱을 다시 시작합니다. 그런 다음 Developer → Configure Third-Party Inference를 열고 게이트웨이 베이스 URL을 입력하세요. 관리자가 배포한 설정이 있는 경우 해당 설정이 우선하며 입력을 수정할 수 없도록 이 읽기 전용으로 표시됩니다.

게이트웨이 구성이 활성화되면 데스크톱 앱은 로컬 머신에서만 세션을 실행합니다. 환경 선택기에 SSH 세션이나 Anthropic 호스팅 클라우드 환경이 나타나지 않으며, [Remote Control](/docs/en/remote-control)을 이용할 수 없습니다. 게이트웨이를 통해 원격 호스트에서 Claude Code를 사용하려면 해당 호스트에서 [`ANTHROPIC_BASE_URL` 및 게이트웨이 자격 증명](#set-the-base-url-and-credential)을 설정한 후 CLI를 실행하세요.

데스크톱 앱에 `Gateway was unreachable` 메시지가 표시되면 시작 시 앱이 구성된 베이스 URL에 접근하지 못한 것입니다. [위의 curl 테스트](#verify-the-connection)로 URL 및 네트워크 경로를 점검하세요.

### GitHub Actions

[Claude Code GitHub Actions](/docs/en/github-actions)는 워크플로의 `env` 블록에서 `ANTHROPIC_BASE_URL` 및 `ANTHROPIC_CUSTOM_HEADERS`를 읽습니다. 자격 증명은 액션의 `anthropic_api_key` 입력값으로 전달하세요. 액션이 이를 `ANTHROPIC_API_KEY`로 설정하여 게이트웨이의 `x-api-key` 헤더로 전달합니다.

`x-api-key` 게이트웨이의 경우 `env`에 베이스 URL을 설정하고 게이트웨이 키를 입력값으로 전달하세요:

```yaml theme={null}
env:
  ANTHROPIC_BASE_URL: https://llm-gateway.example.com

steps:
  - uses: anthropics/claude-code-action@v1
    with:
      anthropic_api_key: ${{ secrets.GATEWAY_API_KEY }}
```

Bearer 토큰 게이트웨이의 경우 동일한 시크릿을 두 번 전달하세요: `anthropic_api_key` 입력값으로 한 번, 워크플로 `env` 블록의 `ANTHROPIC_AUTH_TOKEN`으로 한 번 전달합니다. 액션은 Claude Code를 실행하기 전에 `anthropic_api_key`, `CLAUDE_CODE_OAUTH_TOKEN`, 또는 워크로드 아이덴티티 페더레이션을 필수 조건으로 확인하며 `ANTHROPIC_AUTH_TOKEN`만으로는 시작 확인을 통과할 수 없기 때문입니다. 환경 변수는 게이트웨이가 읽는 `Authorization` 헤더에 키를 넣는 용도이며, `x-api-key`에 들어간 사본은 무시됩니다:

```yaml theme={null}
env:
  ANTHROPIC_BASE_URL: https://llm-gateway.example.com
  ANTHROPIC_AUTH_TOKEN: ${{ secrets.GATEWAY_API_KEY }}

steps:
  - uses: anthropics/claude-code-action@v1
    with:
      anthropic_api_key: ${{ secrets.GATEWAY_API_KEY }}
```

`CLAUDE_CODE_OAUTH_TOKEN` 및 워크로드 아이덴티티 페더레이션을 포함한 액션의 다른 인증 옵션은 [Claude Code GitHub Actions](/docs/en/github-actions) 및 액션 [README](https://github.com/anthropics/claude-code-action#readme)를 참조하세요.

### Agent SDK

[Agent SDK](/docs/en/agent-sdk/overview)에는 별도의 게이트웨이 전용 옵션이 없으며, 자신이 생성하는 Claude Code 프로세스에 환경 변수를 전달합니다. 각 SDK는 생성된 프로세스의 환경을 설정하는 `env` 옵션을 승인하며, TypeScript와 Python SDK는 이를 다르게 처리합니다:

* TypeScript: 기본적으로 생성된 프로세스가 부모 환경을 상속하지만 `options.env`를 설정하면 환경을 완전히 대체합니다. 게이트웨이 변수를 유지하려면 `process.env`를 전개(spread)하여 포함시키세요.
* Python: `ClaudeAgentOptions(env=...)`는 상속된 환경 위에 병합되므로 부모 프로세스에 설정된 게이트웨이 변수가 전개 작업 없이도 그대로 유지됩니다.

<CodeGroup>
  ```ts TypeScript theme={null}
  const result = query({
    prompt: "...",
    options: {
      env: {
        ...process.env,
        ANTHROPIC_BASE_URL: "https://llm-gateway.example.com",
        ANTHROPIC_AUTH_TOKEN: process.env.GATEWAY_KEY,
      },
    },
  })
  ```

  ```python Python theme={null}
  options = ClaudeAgentOptions(
      env={
          "ANTHROPIC_BASE_URL": "https://llm-gateway.example.com",
          "ANTHROPIC_AUTH_TOKEN": os.environ["GATEWAY_KEY"],
      }
  )
  ```
</CodeGroup>

### Slack, 웹 및 Remote Control

[Claude Code in Slack](/docs/en/slack) 및 [Claude Code on the web](/docs/en/claude-code-on-the-web)은 항상 Anthropic API를 사용하는 Anthropic 호스팅 제품이며, 게이트웨이 배포의 대상이 아닙니다. 클라우드 세션 환경 구성에 설정된 게이트웨이 변수는 적용되지 않습니다. 트래픽이 반드시 게이트웨이에 머물러야 하는 경우 해당 사용자에게 이 서페이스들을 활성화하지 마세요.

[Remote Control](/docs/en/remote-control)과 [음성 입력(voice dictation)](/docs/en/voice-dictation)은 모두 claude.ai 아이덴티티에 의존합니다: Remote Control은 라이브 세션을 계정과 페어링하는 데, 음성 입력은 claude.ai 받아쓰기 엔드포인트에 도달하는 데 사용됩니다. `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, 또는 `apiKeyHelper`가 활성화되어 있는 동안에는 두 기능을 사용할 수 없습니다. v2.1.196부터 Remote Control은 `ANTHROPIC_BASE_URL`이 Anthropic 호스트 이외를 가리킬 때도 비활성화되므로, claude.ai로 로그인하는 것만으로는 충분하지 않습니다.

두 기능을 복원하려면 claude.ai로 로그인하고 해당 기능이 확인하는 게이트웨이 변수 설정을 해제하세요. `claude doctor`의 Remote Control 섹션에 해제해야 할 자격 증명 변수명이 안내됩니다.

* 음성 입력: 게이트웨이 자격 증명 해제
* Remote Control: 게이트웨이 자격 증명 및 `ANTHROPIC_BASE_URL` 해제

## 추가 구성

이 설정들은 베이스 URL 및 자격 증명 외의 상황을 다룹니다. 관리자의 지시, 네트워크의 아웃바운드 규칙, 또는 [게이트웨이 문제 해결 표](#troubleshoot-gateway-errors)에서 요구하는 경우에만 설정하세요.

### 추가 헤더 전송

일부 게이트웨이는 테넌트 식별자나 라우팅 키와 같은 자격 증명 외에 커스텀 헤더를 사용하여 요청을 라우팅하거나 태그합니다. 전달하려면 한 줄에 하나의 `Name: Value` 쌍으로 [`ANTHROPIC_CUSTOM_HEADERS`](/docs/en/env-vars)를 설정하세요. 아래 예시는 `X-Org-Route`라는 이름의 라우팅 헤더를 추가합니다:

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_CUSTOM_HEADERS="X-Org-Route: prod"
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_CUSTOM_HEADERS = "X-Org-Route: prod"
    ```
  </Tab>
</Tabs>

설정 파일의 `env` 블록에 `ANTHROPIC_CUSTOM_HEADERS`를 설정할 수도 있습니다. JSON 문자열은 여러 줄에 걸쳐 작성할 수 없으므로 개행 구분을 위해 `\n`을 사용하세요:

```json theme={null}
{
  "env": {
    "ANTHROPIC_CUSTOM_HEADERS": "X-Org-Route: prod\nX-Tenant: example"
  }
}
```

### 게이트웨이 모델을 모델 선택기에 추가

모델 탐색(Model discovery) 기능은 시작 시 게이트웨이에 모델 목록을 쿼리하여 해당 모델 이름을 내장 항목과 함께 `/model` 선택기에 추가합니다.

게이트웨이가 Claude Code의 내장 목록에 없는 모델 이름을 제공하고 이를 선택기에서 지정하고 싶다면 이 기능을 활성화하세요. 내장 모델만 사용하는 경우 모델 탐색을 활성화할 필요가 없으며, 관리자가 이미 관리형 설정을 통해 이를 활성화했을 수도 있습니다.

활성화하려면 셸 또는 `~/.claude/settings.json`의 `env` 블록에 `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`을 설정하세요. 탐색 기능을 사용하려면 Claude Code v2.1.129 이상이 필요합니다.

탐색된 모델은 `From gateway`라는 레이블이 붙어 `/model` 항목에 추가로 나타납니다. 탐색이 실행되었는지 확인하려면 `claude --debug`로 시작하여 `[gatewayDiscovery]` 줄을 확인하세요: 성공 시 캐시된 모델 수가 기록되고, 404, 타임아웃, 리다이렉트 발생 시에도 여기에 기록됩니다. 탐색 실행 시점, 필터링 대상, 게이트웨이가 제공하는 응답 형식은 [모델 탐색 참조](/docs/en/llm-gateway-protocol#model-discovery)를 참조하세요.

### apiKeyHelper로 자격 증명 교체

`apiKeyHelper`는 정적 환경 변수에서 읽는 대신 Claude Code가 게이트웨이 자격 증명을 가져오기 위해 실행하는 명령입니다.

자격 증명이 일정 주기로 만료되거나 볼트(vault) 또는 SSO 명령에서 가져와야 할 때, 혹은 관리자가 이를 구성하도록 가이드한 경우 헬퍼를 사용하세요. 자격 증명이 한 번 설정하는 고정 문자열이라면 [자격 증명 변수](#set-the-credential-variable)로 충분하므로 이 섹션을 건너뛰어도 됩니다.

헬퍼는 표준 출력(stdout)에 현재 자격 증명을 출력하는 임의의 셸 명령입니다. Claude Code는 시스템 셸을 통해 이를 실행하므로 Windows에서는 실행 파일이나 PowerShell 호출 형태일 수 있습니다. 스크립트를 작성하고 실행 권한을 부여한 후 [설정 파일](/docs/en/settings)의 `apiKeyHelper`에서 참조하세요:

<Tabs>
  <Tab title="Bash or Zsh">
    예를 들어 볼트에서 읽어오는 스크립트:

    ```bash theme={null}
    #!/bin/bash
    vault kv get -field=api_key secret/llm-gateway/claude-code
    ```

    `~/.claude/settings.json`에서 해당 경로를 참조합니다:

    ```json theme={null}
    {
      "apiKeyHelper": "~/bin/get-gateway-key.sh"
    }
    ```
  </Tab>

  <Tab title="PowerShell">
    예를 들어 볼트에서 읽어오는 스크립트:

    ```powershell theme={null}
    vault kv get -field=api_key secret/llm-gateway/claude-code
    ```

    `%USERPROFILE%\.claude\settings.json`에서 JSON 문자열 내의 백슬래시를 이스케이프 처리하여 PowerShell 호출을 참조합니다:

    ```json theme={null}
    {
      "apiKeyHelper": "powershell -NoProfile -File C:\\scripts\\get-gateway-key.ps1"
    }
    ```
  </Tab>
</Tabs>

Claude Code는 헬퍼의 출력을 기본 5분 동안 캐시하고, 요청이 HTTP 401을 반환할 때 다시 실행합니다. 캐시 수명을 변경하려면 밀리초 단위로 `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`를 설정하세요(예: 15분의 경우 `CLAUDE_CODE_API_KEY_HELPER_TTL_MS=900000`).

헬퍼의 값은 `Authorization` 및 `x-api-key` 헤더 모두로 전송되므로 게이트웨이가 어느 헤더를 읽든 작동합니다.

### 게이트웨이 경로 외 트래픽 비활성화

게이트웨이는 모델 요청을 전달하지만, Claude Code는 버전 확인, 텔레메트리, 오류 보고서, 릴리스 노트 등 게이트웨이 경로 외의 비필수 백그라운드 트래픽도 Anthropic 및 GitHub 등 서드파티 서비스로 보냅니다. 게이트웨이로의 아웃바운드만 허용하는 네트워크에서는 이러한 요청이 실패하고 아웃바운드 모니터링에 차단된 연결로 나타날 수 있습니다.

해당 트래픽을 차단하려면 셸 export 또는 설정 파일의 `env` 블록에서 게이트웨이 변수와 함께 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`을 설정하세요:

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC = "1"
    ```
  </Tab>
</Tabs>

변수를 설정하면 다음과 같은 영향과 제한 사항이 발생합니다:

* 자동 업데이트가 비활성화되므로 패키지 관리자나 관리형 배포 등 다른 업데이트 방안을 마련해야 합니다.
* [fast mode](/docs/en/fast-mode) 사용 가능 여부 확인이 차단됩니다. 이전 확인을 통해 머신에 fast mode가 이미 활성화되어 있지 않았다면 `/fast` 실행 시 fast mode를 사용할 수 없다고 보고됩니다.
* 게이트웨이 자체에 쿼리하더라도 [게이트웨이 모델 탐색](#add-gateway-models-to-the-model-picker)이 꺼집니다. 이전에 탐색된 모델은 로컬 캐시에 계속 남아 있지만 목록이 새로고침되지 않습니다.
* WebFetch 도구의 [도메인 안전성 확인](/docs/en/data-usage#webfetch-domain-safety-check)은 영향을 받지 않으며 여전히 `api.anthropic.com`을 호출합니다. 네트워크에서 해당 호스트를 차단하는 경우 [설정](/docs/en/settings)에서 `skipWebFetchPreflight: true`로 별도 끄기 설정을 하세요.
* 각 텔레메트리 스트림과 이를 제어하는 변수는 [텔레메트리 서비스](/docs/en/data-usage#telemetry-services)를 참조하세요.

### 게이트웨이를 통해 클라우드 프로바이더로 라우팅

이 구성들은 `ANTHROPIC_BASE_URL` 대신 프로바이더 전용 베이스 URL 변수를 사용하여 Claude Code가 게이트웨이를 가리키도록 합니다. Amazon Bedrock 및 Google Cloud's Agent Platform 게이트웨이는 해당 프로바이더의 네이티브 요청 형식을 수용하며, Microsoft Foundry 및 Claude Platform on AWS 게이트웨이는 Anthropic Messages 형식을 수용하고 호환되는 베이스 URL 변수만 다릅니다.

게이트웨이 팀에서 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, 또는 Claude Platform on AWS를 명시적으로 밝힌 경우에만 선택하여 사용하세요. 위 [검증 요청](#verify-the-connection)에서 정상적인 JSON이 반환되었다면 이 섹션을 건너뛰어도 됩니다.

게이트웨이 팀이 안내한 프로바이더의 블록을 설정하세요. skip-auth 변수는 게이트웨이가 프로바이더 자격 증명을 보유하고 있으므로 Claude Code가 프로바이더 자격 증명으로 요청에 서명하지 않도록 지시합니다. 게이트웨이에 자체 토큰이 필요한 경우 Microsoft Foundry(아래에 표시된 대로 `ANTHROPIC_FOUNDRY_API_KEY` 사용)를 제외하고 블록 뒤에 `ANTHROPIC_AUTH_TOKEN`을 추가하세요. Bearer 토큰을 기대하는 Microsoft Foundry 게이트웨이는 대신 [`ANTHROPIC_FOUNDRY_AUTH_TOKEN`](/docs/en/env-vars)을 사용할 수 있으며, 둘 다 설정된 경우 `ANTHROPIC_FOUNDRY_API_KEY`보다 우선 적용됩니다. `ANTHROPIC_FOUNDRY_AUTH_TOKEN`은 Claude Code v2.1.203 이상이 필요합니다.

#### Amazon Bedrock

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_BEDROCK_BASE_URL=https://llm-gateway.example.com/bedrock
    export CLAUDE_CODE_SKIP_BEDROCK_AUTH=1
    export CLAUDE_CODE_USE_BEDROCK=1
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_BEDROCK_BASE_URL = "https://llm-gateway.example.com/bedrock"
    $env:CLAUDE_CODE_SKIP_BEDROCK_AUTH = "1"
    $env:CLAUDE_CODE_USE_BEDROCK = "1"
    ```
  </Tab>
</Tabs>

#### Google Cloud's Agent Platform

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_VERTEX_BASE_URL=https://llm-gateway.example.com/vertex
    export ANTHROPIC_VERTEX_PROJECT_ID=your-gcp-project-id
    export CLAUDE_CODE_SKIP_VERTEX_AUTH=1
    export CLAUDE_CODE_USE_VERTEX=1
    export CLOUD_ML_REGION=us-east5
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_VERTEX_BASE_URL = "https://llm-gateway.example.com/vertex"
    $env:ANTHROPIC_VERTEX_PROJECT_ID = "your-gcp-project-id"
    $env:CLAUDE_CODE_SKIP_VERTEX_AUTH = "1"
    $env:CLAUDE_CODE_USE_VERTEX = "1"
    $env:CLOUD_ML_REGION = "us-east5"
    ```
  </Tab>
</Tabs>

#### Microsoft Foundry

게이트웨이 자격 증명을 `ANTHROPIC_FOUNDRY_API_KEY`에 입력하면 게이트웨이에 `x-api-key` 헤더로 전송됩니다. Bearer 토큰을 필요로 하는 게이트웨이는 대신 [`ANTHROPIC_FOUNDRY_AUTH_TOKEN`](/docs/en/env-vars)을 받을 수 있습니다. Claude Code는 해당 값을 `Authorization: Bearer` 헤더로 전송하며 둘 다 설정된 경우 `ANTHROPIC_FOUNDRY_API_KEY`보다 우선 적용됩니다. Claude Code v2.1.203 이상이 필요합니다.

자체 `Authorization` 헤더를 주입하는 게이트웨이의 경우 `CLAUDE_CODE_SKIP_FOUNDRY_AUTH=1`을 설정하고 두 자격 증명 변수를 모두 비워 두세요. 그러면 Claude Code는 Azure 자격 증명 없이 요청을 전송하며, `ANTHROPIC_CUSTOM_HEADERS` 등을 통해 제공한 `Authorization` 헤더를 유지합니다. v2.1.203 이전에는 API 키 없이 `CLAUDE_CODE_SKIP_FOUNDRY_AUTH`를 설정하면 Microsoft Foundry 클라이언트가 요청을 전송하지 못했습니다.

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_FOUNDRY_BASE_URL=https://llm-gateway.example.com/foundry
    export ANTHROPIC_FOUNDRY_API_KEY=sk-gateway-key
    export CLAUDE_CODE_USE_FOUNDRY=1
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_FOUNDRY_BASE_URL = "https://llm-gateway.example.com/foundry"
    $env:ANTHROPIC_FOUNDRY_API_KEY = "sk-gateway-key"
    $env:CLAUDE_CODE_USE_FOUNDRY = "1"
    ```
  </Tab>
</Tabs>

#### Claude Platform on AWS

워크스페이스 ID는 [Claude Platform on AWS](/docs/en/claude-platform-on-aws)를 참조하세요.

<Tabs>
  <Tab title="Bash or Zsh">
    ```bash theme={null}
    export ANTHROPIC_AWS_BASE_URL=https://llm-gateway.example.com/anthropic-aws
    export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
    export CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH=1
    export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell theme={null}
    $env:ANTHROPIC_AWS_BASE_URL = "https://llm-gateway.example.com/anthropic-aws"
    $env:ANTHROPIC_AWS_WORKSPACE_ID = "wrkspc_01ABCDEFGHIJKLMN"
    $env:CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH = "1"
    $env:CLAUDE_CODE_USE_ANTHROPIC_AWS = "1"
    ```
  </Tab>
</Tabs>

## 게이트웨이 문제 해결

다음은 게이트웨이를 통해 Claude Code를 실행할 때 발생하는 가장 흔한 오류와 게이트웨이 측 원인 및 해결 방법입니다:

| 오류                                                                                                                                                                                                                                                                                                           | 원인                                                                                                                                                                                                                                                                                                                                     | 해결 방법                                                                                                                                                                                                                                                                                                                                                                                      |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 두 개의 자격 증명 소스를 지정하며 `auth may not work as expected`로 끝나는 시작 경고. (구버전에서는 `Auth conflict: Both a token (SOURCE) and an API key (SOURCE) are set`으로 표시됨)                                                                                                              | 게이트웨이 자격 증명과 저장된 로그인 정보가 모두 활성화되어 있는 상태입니다. 요청에는 환경 변수가 사용되지만, 오래된 로그인 정보로 인해 예기치 않은 인증 동작이 발생할 수 있습니다.                                                                                                                                                                                         | 저장된 로그인을 사용하려면 변수 설정을 해제하고, 게이트웨이 자격 증명을 사용하려면 `/logout`을 실행하세요.                                                                                                                                                                                                                                                                                                |
| 유효하지 않거나 인식할 수 없는 토큰이라는 `401` 오류                                                                                                                                                                                                                                                            | 자격 증명이 게이트웨이에서 발급한 것이 아니거나 게이트웨이가 읽지 않는 헤더에 위치해 있습니다.                                                                                                                                                                                                                                                 | [자격 증명 표](#set-the-credential-variable)에서 변수가 자격 증명 종류와 일치하는지 확인하고, 키가 취소된 경우 게이트웨이에서 키를 재발급받으세요.                                                                                                                                                                                                                       |
| `Your apiKeyHelper script is failing`                                                                                                                                                                                                                                                                           | [`apiKeyHelper`](/docs/en/settings#available-settings) 설정에 지정된 명령이 오류로 종료되었거나, 타임아웃이 발생했거나, 아무것도 출력하지 않아 요청에 더미 키가 전달되었습니다.                                                                                                                                                                     | 명령을 직접 실행하여 실패 원인을 확인하고, 세션 만료가 보고된 경우 자격 증명 제공자에 다시 인증하세요. [오류 참조 문서](/docs/en/errors#your-apikeyhelper-script-is-failing)를 확인하세요.                                                                                                                                                                              |
| 해당 주소에 아무 응답이 없는 `Unable to connect to API (ConnectionRefused)`, 호스트명이 확인되지 않는 `Unable to connect to API (FailedToOpenSocket)`, 또는 npm 설치 시의 `(ECONNREFUSED)` (Claude Code가 [백오프 재시도](/docs/en/errors#automatic-retries)를 수행하는 동안 일시 중지 후 발생) | 베이스 URL에서 아무 응답이 없습니다: 주소가 잘못되었거나 VPN/방화벽이 게이트웨이 경로를 차단하고 있습니다.                                                                                                                                                                                                                               | 동일한 원인으로 즉시 실패하는 [위의 curl 테스트](#verify-the-connection)를 실행하고 게이트웨이 팀을 통해 URL 및 네트워크 경로를 확인하세요.                                                                                                                                                                                                                              |
| `API returned an empty or malformed response (HTTP 200)`                                                                                                                                                                                                                                                        | 게이트웨이 또는 중간 프록시가 API 응답이 아닌 결과(주로 HTML 오류 페이지 또는 로그인 페이지)를 반환했습니다.                                                                                                                                                                                                                                       | [위의 curl 요청](#verify-the-connection)으로 테스트하고, JSON이 아닌 결과를 반환하는 게이트웨이 라우팅을 수정하세요.                                                                                                                                                                                                                                                   |
| `context_management`, `Extra inputs are not permitted` 또는 기타 인식되지 않는 필드를 지목하는 `400` 오류                                                                                                                                                                                                        | 게이트웨이가 Claude Code가 Anthropic 형식 엔드포인트로 전송하는 필드를 거부하는 업스트림으로 요청을 전달하고 있습니다.                                                                                                                                                                                                                          | 사전 공개 필드 대부분을 억제하는 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`을 설정하세요. [기능 전달(feature pass-through)](/docs/en/llm-gateway-protocol#feature-pass-through)을 참조하세요. 일부 베터 기능은 이 플래그로 제어되지 않으므로, 이 경우 해당 프로바이더가 수용하는 내용만 전송하도록 일치하는 `CLAUDE_CODE_USE_*` 프로바이더 변수를 설정하세요.                        |
| `Input tag 'adaptive' found`와 같이 `thinking` 또는 `adaptive`를 지목하는 `400` 오류                                                                                                                                                                                                                              | 업스트림 모델 빌드가 Claude Code가 Claude 4.6 이상 모델에 요청하는 적응형 추론(adaptive reasoning)을 수용하지 않습니다.                                                                                                                                                                                                                    | 게이트웨이의 업스트림을 업그레이드하세요. Opus 4.6 및 Sonnet 4.6의 경우 대신 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`을 설정하면 됩니다. [모델 구성](/docs/en/model-config) 기능 변수는 `ANTHROPIC_BASE_URL` 게이트웨이 뒤가 아닌 `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`와 같은 프로바이더 구성에만 적용됩니다.                                              |
| 게이트웨이의 표현(예: `ContextWindowExceededError` 또는 `prompt token count of N exceeds the limit of M`)으로 컨텍스트나 토큰 한도를 통보하는 `400` 오류                                                                                                                                              | 게이트웨이가 모델의 네이티브 창보다 작은 컨텍스트를 강제하고 업스트림 오류를 재작성하여 Anthropic의 `prompt is too long` 문구에 반응하는 자동 압축 후 재시도가 트리거되지 않습니다.                                                                                                                                                                                         | 세션을 복구하려면 `/compact`를 실행하세요. 예방하려면 `CLAUDE_CODE_AUTO_COMPACT_WINDOW`를 게이트웨이 한도 값으로 설정하세요 (최소 100,000 토큰, 최대 모델의 컨텍스트 창으로 제한되므로 100,000 미만의 게이트웨이 한도는 자동 매칭되지 않으며 `/compact` 수동 실행이 복구 방법으로 남습니다). 또한 `CLAUDE_CODE_MAX_OUTPUT_TOKENS`를 게이트웨이 모델의 출력 한도 미만으로 설정하세요. |
| `/model` 선택기에 모델이 누락됨                                                                                                                                                                                                                                                                         | 게이트웨이 모델 이름이 Claude Code의 내장 목록에 없습니다.                                                                                                                                                                                                                                                                                 | [게이트웨이 모델 탐색](#add-gateway-models-to-the-model-picker)을 활성화하거나 [모델 구성](/docs/en/model-config) 변수를 사용하여 이름을 추가하세요.                                                                                                                                                                                                                                        |
| 추론 요청은 동작하지만 `/fast` 실행 시 `Fast mode unavailable due to network connectivity issues`로 보고됨                                                                                                                                                                                                        | [fast mode](/docs/en/fast-mode) 사용 가능 여부 확인은 `ANTHROPIC_BASE_URL`을 따르지 않고 `api.anthropic.com`으로 직접 요청되므로 아웃바운드가 차단되면 확인이 실패합니다. 네트워크가 열려 있더라도 `ANTHROPIC_API_KEY`나 `apiKeyHelper`에서 게이트웨이 발급 키를 전송하고 Anthropic이 이를 거부할 때 동일한 메시지가 표시됩니다. | 아웃바운드가 차단된 경우 `api.anthropic.com`을 허용 목록에 추가하거나 건너뛰기 변수를 설정하세요. 거부된 게이트웨이 키의 경우 건너뛰기 변수만 효과가 있습니다. [프록시 및 LLM 게이트웨이 뒤에서 fast mode 사용하기](/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways)를 참조하세요.                                                                                                                                    |
| 조직이 fast mode를 활성화했음에도 `ANTHROPIC_AUTH_TOKEN`으로 인증된 세션에서 `/fast`가 `Fast mode has been disabled by your organization`으로 보고됨                                                                                                                                                                  | 사용 가능 여부 확인에는 claude.ai 로그인이나 Anthropic API 키가 필요합니다. Bearer 토큰만 있는 경우 Claude Code는 확인을 보내지 않고 fast mode가 비활성화된 것으로 처리합니다.                                                                                                                                                                   | `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK=1`을 설정하세요. [프록시 및 LLM 게이트웨이 뒤에서 fast mode 사용하기](/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways)를 참조하세요.                                                                                                                                                                                                                           |
| [curl 테스트](#verify-the-connection)가 성공함에도 Claude Code가 반복적으로 로그인을 요구함                                                                                                                                                                                                                     | CLI 자체 자격 증명이 없는 상태입니다: 도달 가능한 베이스 URL만으로는 자격 증명이 되지 않으며, 프로젝트의 `.claude/settings.json` 또는 `.claude/settings.local.json`에 있는 `env` 블록은 최초 실행 마법사 및 신뢰 프롬프트 이후에만 적용됩니다.                                                                                                                 | 셸 export, `~/.claude/settings.json` 내의 `env` 블록, 또는 관리형 설정 등 최초 실행 전에 Claude Code가 읽을 수 있는 위치에 `ANTHROPIC_AUTH_TOKEN`을 설정하세요.                                                                                                                                                                                                                         |
| `ANTHROPIC_API_KEY`가 설정되었으나 프롬프트 없이 무시됨                                                                                                                                                                                                                                                          | 인터랙티브 세션에서 키 승인이 한 번 필요하며, 이전에 거절된 키는 다시 묻지 않고 무시됩니다.                                                                                                                                                                                                                  | `/config` 아래에서 `Use custom API key` 옵션을 활성화하세요.                                                                                                                                                                                                                                                                                                                           |
| `This machine's managed settings require a first-party login`                                                                                                                                                                                                                                                   | 관리형 설정에 `forceLoginMethod` 또는 `forceLoginOrgUUID`가 포함되어 있으며, Claude Code v2.1.146 이상에서는 이 설정이 `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, 또는 `apiKeyHelper`와 공존할 수 없습니다.                                                                                                                                            | 게이트웨이 자격 증명을 사용하려면 관리자가 관리형 설정에서 `forceLoginMethod` 및 `forceLoginOrgUUID`를 제거해야 하고, 퍼스트파티 로그인을 사용하려면 게이트웨이 자격 증명을 제거해야 합니다. 두 방식은 혼용될 수 없습니다.                                                                                                                                                                        |
| 게이트웨이의 자체 로그에는 수신된 요청이 없는데 `403 Forbidden`과 같은 HTML 본문과 함께 `403` 발생                                                                                                                                                                                      | 게이트웨이 앞단의 웹 애플리케이션 방화벽(WAF) 또는 리버스 프록시가 요청 본문이 게이트웨이에 도달하기 전에 차단했습니다. Claude Code 프롬프트에는 XSS 본문 규칙에 걸리는 XML 스타일 태그 및 소스 코드가 포함되므로, 짧은 curl 테스트는 통과하지만 실제 세션은 실패할 수 있습니다.                                               | 게이트웨이의 `/v1/messages` 경로를 요청 본문 검사 대상에서 제외하세요. AWS WAF에서는 `CrossSiteScripting_Body` 관리형 규칙에 해당하며, ModSecurity가 있는 nginx에서는 이에 상응하는 OWASP CRS 본문 규칙에 해당합니다.                                                                                                                                                                                |
| [curl 테스트](#verify-the-connection)가 성공함에도 `SSL certificate verification failed` 또는 `Self-signed certificate detected`와 같은 인증서/TLS 오류 발생                                                                                                                                            | Claude Code의 런타임이 `curl`이 사용하는 것과 동일한 인증 기관(CA)을 신뢰하지 않는 상태입니다. 사내 TLS 검사 프록시 환경에서 흔히 발생합니다.                                                                                                                                                                                                      | CA 번들 경로를 `NODE_EXTRA_CA_CERTS`로 설정하세요. [CA 인증서 저장소](/docs/en/network-config#ca-certificate-store)를 참조하세요.                                                                                                                                                                                                                                                                     |

게이트웨이 구성을 제거한 후에도 Claude Code가 계속 로그인을 요구하는 경우, 원인은 게이트웨이가 아닌 자격 증명 저장소일 가능성이 높습니다. [인증 오류](/docs/en/errors#authentication-errors)를 참조하세요.

## 관련 리소스

* [LLM 게이트웨이 개요](/docs/en/llm-gateway): 게이트웨이란 무엇이며 claude.ai 구독과 어떻게 상호작용하는지
* [조직을 위한 LLM 게이트웨이 배포하기](/docs/en/llm-gateway-rollout): 게이트웨이 구성 배포 및 전달을 위한 관리자용 체크리스트
* [게이트웨이 프로토콜 참조](/docs/en/llm-gateway-protocol): 게이트웨이가 전달해야 하는 헤더 및 필드를 포함하여 Claude Code가 게이트웨이로 전송하는 내용
* [설정](/docs/en/settings): 설정 파일의 위치 및 `env` 블록 읽기 방식
* [인증](/docs/en/authentication): 자격 증명 변수, `apiKeyHelper` 및 OAuth 로그인의 상호작용 방식
