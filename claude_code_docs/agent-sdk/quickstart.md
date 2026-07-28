> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 퀵스타트 (Quickstart)

> Python 또는 TypeScript Agent SDK를 사용하여 수동 개입 없이 자율적으로 작동하는 AI 에이전트를 구축해 보세요.

Agent SDK를 사용하여 수동 작업 없이 코드를 읽고, 버그를 찾고, 수정하는 AI 에이전트를 만들어 봅니다.

**진행 과정:**

1. Agent SDK로 프로젝트 설정하기
2. 버그가 포함된 예제 코드 파일 생성하기
3. 버그를 자동으로 찾아 수정하는 에이전트 실행하기

## 사전 요구 사항

* **Node.js 18+** 또는 **Python 3.10+**
* **Anthropic 계정** ([여기서 가입](https://platform.claude.com/))

## 설정

<Steps>
  <Step title="프로젝트 폴더 생성">
    퀵스타트를 위한 새 디렉토리를 생성합니다:

    ```bash theme={null}
    mkdir my-agent
    cd my-agent
    ```

    자신의 프로젝트의 경우 모든 폴더에서 SDK를 실행할 수 있으며, 기본적으로 해당 디렉토리 및 하위 디렉토리의 파일에 접근할 수 있습니다.
  </Step>

  <Step title="SDK 설치">
    사용하려는 언어에 맞는 Agent SDK 패키지를 설치합니다:

    <Tabs>
      <Tab title="TypeScript (새 프로젝트)">
        ```bash theme={null}
        npm init -y
        npm pkg set type=module
        npm install @anthropic-ai/claude-agent-sdk
        npm install --save-dev tsx
        ```

        `package.json`에 `"type": "module"`을 설정하면 에이전트 스크립트에서 최상위 `await`를 사용할 수 있으며, [tsx](https://tsx.is)를 사용해 TypeScript 파일을 직접 실행할 수 있습니다. 설치가 성공하면 npm이 `added N packages`를 출력합니다.
      </Tab>

      <Tab title="TypeScript (기존 프로젝트)">
        ```bash theme={null}
        npm install @anthropic-ai/claude-agent-sdk
        npm install --save-dev tsx
        ```

        [tsx](https://tsx.is)는 TypeScript 파일을 직접 실행합니다. 프로젝트가 CommonJS를 사용하는 경우 에이전트 스크립트 이름을 `agent.ts` 대신 `agent.mts`로 만드세요. `.mts` 확장자는 tsx가 해당 파일을 ES 모듈로 처리하도록 하므로 전체 프로젝트를 ES 모듈로 변환하지 않고도 최상위 `await`가 동작합니다. 이 퀵스타트의 이후 단계에서 `agent.ts` 대신 `agent.mts`를 사용하세요.
      </Tab>

      <Tab title="Python (uv)">
        [uv](https://docs.astral.sh/uv/)는 가상 환경을 자동으로 관리해 주는 빠른 Python 패키지 관리자입니다:

        ```bash theme={null}
        uv init
        uv add claude-agent-sdk
        ```
      </Tab>

      <Tab title="Python (pip)">
        가상 환경을 생성 및 활성화한 다음 패키지를 설치합니다.

        macOS 또는 Linux:

        ```bash theme={null}
        python3 -m venv .venv
        source .venv/bin/activate
        pip install claude-agent-sdk
        ```

        Windows:

        ```powershell theme={null}
        py -m venv .venv
        .venv\Scripts\Activate.ps1
        pip install claude-agent-sdk
        ```

        PowerShell에서 실행 정책 오류로 `Activate.ps1`이 차단되는 경우 먼저 `Set-ExecutionPolicy -Scope Process RemoteSigned`를 실행하세요.
      </Tab>
    </Tabs>

    <Note>
      TypeScript 및 Python SDK 모두 해당 플랫폼용 네이티브 Claude Code 바이너리를 번들로 제공하므로 Claude Code를 별도로 설치할 필요가 없습니다.
    </Note>
  </Step>

  <Step title="API 키 설정">
    [Claude Console](https://platform.claude.com/)에서 API 키를 발급받은 후 에이전트를 실행할 셸의 환경 변수로 설정합니다:

    <Tabs>
      <Tab title="macOS / Linux">
        ```bash theme={null}
        export ANTHROPIC_API_KEY=your-api-key
        ```
      </Tab>

      <Tab title="Windows (PowerShell)">
        ```powershell theme={null}
        $env:ANTHROPIC_API_KEY = "your-api-key"
        ```
      </Tab>
    </Tabs>

    SDK는 에이전트를 실행하는 프로세스의 환경 변수에서 키를 읽어옵니다. `.env` 파일을 자동으로 로드하지 않습니다. `.env` 파일에 키를 보관하는 경우 SDK를 호출하기 전에 `dotenv` 패키지 등을 사용하여 직접 로드하세요.

    SDK는 써드파티 API 제공자를 통한 인증도 지원합니다:

    * **Amazon Bedrock**: `CLAUDE_CODE_USE_BEDROCK=1` 환경 변수를 설정하고 AWS 자격 증명 구성
    * **Claude Platform on AWS**: `CLAUDE_CODE_USE_ANTHROPIC_AWS=1` 및 `ANTHROPIC_AWS_WORKSPACE_ID`를 설정한 후 AWS 자격 증명 구성
    * **Google Cloud's Agent Platform**: `CLAUDE_CODE_USE_VERTEX=1` 환경 변수를 설정하고 Google Cloud 자격 증명 구성
    * **Microsoft Foundry**: `CLAUDE_CODE_USE_FOUNDRY=1` 환경 변수를 설정하고 Azure 자격 증명 구성

    자세한 내용은 [Amazon Bedrock](/docs/en/amazon-bedrock), [Claude Platform on AWS](/docs/en/claude-platform-on-aws), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry) 설정 가이드를 참고하세요.

    <Note>
      사전 승인을 받은 경우가 아니면, Anthropic은 서드파티 개발자가 Claude Agent SDK로 구축된 에이전트를 포함한 자체 제품에 claude.ai 로그인이나 레이트 리밋(rate limit)을 제공하는 것을 허용하지 않습니다. 본 문서에 설명된 API 키 인증 방법을 대신 사용해 주세요.
    </Note>
  </Step>
</Steps>

## 버그가 있는 파일 생성

이 퀵스타트에서는 코드에서 버그를 찾아 수정할 수 있는 에이전트를 만드는 과정을 안내합니다. 먼저 에이전트가 수정할 수 있도록 의도적인 버그가 포함된 파일이 필요합니다. `my-agent` 디렉토리에 `utils.py`를 생성하고 다음 코드를 붙여넣으세요:

```python theme={null}
def calculate_average(numbers):
    total = 0
    for num in numbers:
        total += num
    return total / len(numbers)


def get_user_name(user):
    return user["name"].upper()
```

이 코드에는 두 가지 버그가 있습니다:

1. `calculate_average([])` 호출 시 0으로 나누기 오류로 런타임 에러 발생
2. `get_user_name(None)` 호출 시 TypeError 발생

## 버그를 찾아 수정하는 에이전트 구축

Python SDK를 사용하는 경우 `agent.py`를, TypeScript를 사용하는 경우 `agent.ts`를 생성하세요. 기존 프로젝트가 CommonJS를 사용하는 경우 대신 `agent.mts`를 사용하세요:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ResultMessage


  async def main():
      # Agentic loop: streams messages as Claude works
      async for message in query(
          prompt="Review utils.py for bugs that would cause crashes. Fix any issues you find.",
          options=ClaudeAgentOptions(
              allowed_tools=["Read", "Edit", "Glob"],  # Auto-approve these tools
              permission_mode="acceptEdits",  # Auto-approve file edits
          ),
      ):
          # Print human-readable output
          if isinstance(message, AssistantMessage):
              for block in message.content:
                  if hasattr(block, "text"):
                      print(block.text)  # Claude's reasoning
                  elif hasattr(block, "name"):
                      print(f"Tool: {block.name}")  # Tool being called
          elif isinstance(message, ResultMessage):
              print(f"Done: {message.subtype}")  # Final result


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // Agentic loop: streams messages as Claude works
  for await (const message of query({
    prompt: "Review utils.py for bugs that would cause crashes. Fix any issues you find.",
    options: {
      allowedTools: ["Read", "Edit", "Glob"], // Auto-approve these tools
      permissionMode: "acceptEdits" // Auto-approve file edits
    }
  })) {
    // Print human-readable output
    if (message.type === "assistant" && message.message?.content) {
      for (const block of message.message.content) {
        if ("text" in block) {
          console.log(block.text); // Claude's reasoning
        } else if ("name" in block) {
          console.log(`Tool: ${block.name}`); // Tool being called
        }
      }
    } else if (message.type === "result") {
      console.log(`Done: ${message.subtype}`); // Final result
    }
  }
  ```
</CodeGroup>

이 코드는 크게 세 부분으로 구성됩니다:

1. **`query`**: 에이전트 루프를 만드는 메인 엔트리 포인트입니다. 비동기 이터레이터를 반환하므로 Claude가 작업하는 동안 `async for`를 사용해 메시지를 스트리밍합니다. 전체 API는 [Python](/docs/en/agent-sdk/python#query) 또는 [TypeScript](/docs/en/agent-sdk/typescript#query) SDK 참조문서를 확인하세요.

2. **`prompt`**: Claude에게 지시할 작업 내용입니다. Claude는 작업 내용에 따라 사용할 도구를 스스로 판단합니다.

3. **`options`**: 에이전트를 위한 구성 옵션입니다. 이 예제에서는 `allowedTools`를 사용하여 `Read`, `Edit`, `Glob`을 미리 승인하고, `permissionMode: "acceptEdits"`를 사용하여 파일 변경을 자동 승인합니다. 기타 옵션으로는 `systemPrompt`, `mcpServers` 등이 있습니다. [Python](/docs/en/agent-sdk/python#claudeagentoptions) 또는 [TypeScript](/docs/en/agent-sdk/typescript#options)의 전체 옵션을 확인해 보세요.

`async for` 루프는 Claude가 생각하고, 도구를 호출하고, 결과를 관찰하며 다음 행동을 결정하는 동안 계속 실행됩니다. 이터레이션마다 메시지(Claude의 추론, 도구 호출, 도구 결과 또는 최종 결과)가 전달됩니다. SDK가 조율 작업(도구 실행, 컨텍스트 관리, 재시도)을 처리하므로 여러분은 스트림을 소비하기만 하면 됩니다. 루프는 Claude가 작업을 마치거나 오류를 만났을 때 종료됩니다.

루프 내부의 메시지 처리 코드는 사람이 읽기 쉬운 출력으로 필터링합니다. 필터링하지 않으면 시스템 초기화 및 내부 상태를 포함한 원시 메시지 객체가 그대로 출력되는데, 이는 디버깅에는 유용하지만 그렇지 않은 경우 어수선할 수 있습니다.

<Note>
  이 예제는 실시간으로 진행 상황을 보여주기 위해 스트리밍 방식을 사용합니다. 실시간 출력이 필요 없는 경우(예: 백그라운드 작업이나 CI 파이프라인) 모든 메시지를 한 번에 수집할 수 있습니다. 자세한 내용은 [스트리밍 vs 단일 턴 모드](/docs/en/agent-sdk/streaming-vs-single-mode)를 참고하세요.
</Note>

### 에이전트 실행

에이전트가 준비되었습니다. 다음 명령으로 실행해 보세요:

<Tabs>
  <Tab title="TypeScript">
    ```bash theme={null}
    npx tsx agent.ts
    ```

    스크립트 이름을 `agent.mts`로 지은 경우 대신 `npx tsx agent.mts`를 실행하세요.
  </Tab>

  <Tab title="Python (uv)">
    ```bash theme={null}
    uv run agent.py
    ```
  </Tab>

  <Tab title="Python (pip)">
    가상 환경이 활성화된 상태에서:

    ```bash theme={null}
    python agent.py
    ```
  </Tab>
</Tabs>

작업이 진행되면서 에이전트는 추론 과정과 호출하는 도구들을 출력하고, 마지막에 `Done: success`를 출력합니다. 실행 후 `utils.py`를 확인해 보세요. 빈 리스트와 null 유저를 처리하는 방어적인 코드가 추가된 것을 볼 수 있습니다. 에이전트가 자율적으로 수행한 작업:

1. `utils.py`를 **읽어서(Read)** 코드를 이해함
2. 로직을 **분석하여** 크래시를 유발할 예외 상황(edge cases)을 식별함
3. 적절한 오류 처리를 추가하도록 파일을 **수정함(Edit)**

이것이 Agent SDK만의 차별점입니다: Claude가 도구 구현을 요구하는 대신 직접 도구를 실행합니다.

<Note>
  "API key not found" 오류가 발생하는 경우 에이전트를 실행하는 셸에 `ANTHROPIC_API_KEY` 환경 변수가 잘 설정되어 있는지 확인하세요. SDK는 `.env` 파일을 자동으로 로드하지 않습니다. 추가 도움이 필요한 경우 [전체 문제 해결 가이드](/docs/en/troubleshooting)를 참고하세요.
</Note>

### 다른 프롬프트 시도해보기

에이전트가 설정되었으니 다른 프롬프트들도 시도해 보세요:

* `"Add docstrings to all functions in utils.py"`
* `"Add type hints to all functions in utils.py"`
* `"Create a README.md documenting the functions in utils.py"`

### 에이전트 커스터마이징

옵션을 변경하여 에이전트의 동작을 수정할 수 있습니다. 몇 가지 예시는 다음과 같습니다:

**웹 검색 기능 추가:**

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      allowed_tools=["Read", "Edit", "Glob", "WebSearch"], permission_mode="acceptEdits"
  )
  ```

  ```typescript TypeScript hidelines={1,-1} theme={null}
  const _ = {
    options: {
      allowedTools: ["Read", "Edit", "Glob", "WebSearch"],
      permissionMode: "acceptEdits"
    }
  };
  ```
</CodeGroup>

**Claude에 커스텀 시스템 프롬프트 부여:**

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      allowed_tools=["Read", "Edit", "Glob"],
      permission_mode="acceptEdits",
      system_prompt="You are a senior Python developer. Always follow PEP 8 style guidelines.",
  )
  ```

  ```typescript TypeScript hidelines={1,-1} theme={null}
  const _ = {
    options: {
      allowedTools: ["Read", "Edit", "Glob"],
      permissionMode: "acceptEdits",
      systemPrompt: "You are a senior Python developer. Always follow PEP 8 style guidelines."
    }
  };
  ```
</CodeGroup>

**터미널에서 명령 실행:**

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      allowed_tools=["Read", "Edit", "Glob", "Bash"], permission_mode="acceptEdits"
  )
  ```

  ```typescript TypeScript hidelines={1,-1} theme={null}
  const _ = {
    options: {
      allowedTools: ["Read", "Edit", "Glob", "Bash"],
      permissionMode: "acceptEdits"
    }
  };
  ```
</CodeGroup>

`Bash`가 활성화된 상태에서 다음을 시도해 보세요: `"Write unit tests for utils.py, run them, and fix any failures"`

## 핵심 개념

**도구(Tools)**는 에이전트가 수행할 수 있는 작업을 제어합니다:

| 도구 | 에이전트가 할 수 있는 작업 |
| -------------------------------------- | ----------------------- |
| `Read`, `Glob`, `Grep` | 읽기 전용 분석 |
| `Read`, `Edit`, `Glob` | 코드 분석 및 수정 |
| `Read`, `Edit`, `Bash`, `Glob`, `Grep` | 완전 자동화 |

**권한 모드(Permission modes)**는 사람의 개입 수준을 제어합니다:

| 모드 | 동작 | 사용 사례 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| `acceptEdits` | 파일 편집 및 일반적인 파일시스템 명령을 자동 승인하고, 다른 작업은 확인을 요청함 | 신뢰할 수 있는 개발 워크플로 |
| `plan` | 읽기 전용 도구만 실행함; 파일 편집은 절대로 자동 승인되지 않으며 `canUseTool` 콜백에 도달함 | 실행을 승인하기 전에 작업 범위를 파악할 때 |
| `dontAsk` | `allowedTools`에 없는 작업은 거부함; 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools) 및 사용자 상호작용이 필요한 도구는 목록에 추가했더라도 거부됨 | 엄격히 제한된 헤드리스 에이전트 |
| `auto` | 모델 분류기가 권한 프롬프트를 승인하거나 거부함 | 안전 가드레일이 적용된 자율 에이전트 |
| `bypassPermissions` | 명시적인 [`ask` 규칙](/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)에 일치하는 도구, 조직에서 [`ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), 사용자 상호작용이 필요한 도구를 제외하고 프롬프트 없이 모든 도구를 실행함. TypeScript SDK의 경우 `options`에 `allowDangerouslySkipPermissions: true` 설정도 필요함 | 샌드박스 처리된 CI, 완전히 신뢰할 수 있는 환경 |
| `default` | 승인 처리를 위해 `canUseTool` 콜백이 필요함 | 커스텀 승인 흐름 |

위 예제는 에이전트가 대화형 프롬프트 없이 실행될 수 있도록 파일 작업을 자동 승인하는 `acceptEdits` 모드를 사용합니다. 사용자에게 승인을 구하고 싶다면 `default` 모드를 사용하고 사용자 입력을 수집하는 [`canUseTool` 콜백](/docs/en/agent-sdk/user-input)을 제공하세요. 더 자세한 제어법은 [권한](/docs/en/agent-sdk/permissions)을 참고하세요.

## 다음 단계

첫 에이전트를 만들었으니, 기능을 확장하고 사용 사례에 맞게 커스터마이징하는 방법을 알아보세요:

* **[권한](/docs/en/agent-sdk/permissions)**: 에이전트가 수행할 수 있는 작업과 승인이 필요한 시점 제어
* **[후크](/docs/en/agent-sdk/hooks)**: 도구 호출 전후에 커스텀 코드 실행
* **[세션](/docs/en/agent-sdk/sessions)**: 컨텍스트를 유지하는 다중 턴 에이전트 구축
* **[MCP 서버](/docs/en/agent-sdk/mcp)**: 데이터베이스, 브라우저, API 및 기타 외부 시스템 연동
* **[호스팅](/docs/en/agent-sdk/hosting)**: Docker, 클라우드 및 CI/CD 환경에 에이전트 배포
* **[에이전트 예제](https://github.com/anthropics/claude-agent-sdk-demos)**: 이메일 어시스턴트, 리서치 에이전트 등 완전한 예제 둘러보기
