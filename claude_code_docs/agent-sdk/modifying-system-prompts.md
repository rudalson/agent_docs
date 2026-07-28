> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 시스템 프롬프트 수정하기

> `claude_code` 프리셋과 커스텀 시스템 프롬프트 중에서 선택하고, CLAUDE.md, 출력 스타일, append 또는 완전한 커스텀 프롬프트로 에이전트 동작을 맞춤화합니다.

시스템 프롬프트는 Claude의 동작, 기능 및 응답 스타일을 정의합니다. 사람이 작업을 지켜보고 제어하는 CLI 또는 IDE와 유사한 코딩 도구의 경우 `claude_code` 프리셋에서 시작하세요. 표면(surface), 정체성 또는 권한 모델이 다른 에이전트의 경우 자신만의 프롬프트를 작성하세요.

이 페이지에서는 다음 내용을 다룹니다:

* 프리셋, `append`가 포함된 프리셋, 그리고 커스텀 프롬프트 중에서 선택하기 위한 결정 표가 포함된 [시스템 프롬프트 작동 방식](#시스템-프롬프트-작동-방식)
* CLAUDE.md 파일, 출력 스타일, `append` 또는 커스텀 문자열로 [에이전트 동작 맞춤화하기](#에이전트-동작-맞춤화하기)
* 보존 가능성, 범위 및 보존 항목에 따른 [4가지 접근 방식 비교](#4가지-접근-방식-비교)
* 맞춤화 방법을 계층화하기 위해 [접근 방식 결합하기](#접근-방식-결합하기)

## 시스템 프롬프트 작동 방식

시스템 프롬프트는 전체 대화 동안 Claude가 행동하는 방식을 형성하는 초기 지침 세트입니다. Agent SDK에는 이를 위한 3가지 출발점이 있습니다:

* **최소 기본값 (Minimal default)**: TypeScript에서 `systemPrompt`, Python에서 `system_prompt`를 설정하지 않으면 SDK는 도구 호출을 다루지만 Claude Code의 코딩 지침, 응답 스타일 및 프로젝트 컨텍스트는 생략하는 최소한의 프롬프트를 사용합니다. 이는 기본적으로 전체 Claude Code 프롬프트를 사용하는 `claude -p`와 다릅니다. CLI에서 마이그레이션 중이고 일치하는 동작을 원한다면 `claude_code` 프리셋을 설정하세요.
* **`claude_code` 프리셋**: 도구 사용 지침, 코드 스타일 및 서식 지침, 응답 톤 및 상세도 규칙, 보안 및 안전 지침, 작업 디렉토리 및 환경에 대한 컨텍스트를 포함하는, Claude Code CLI가 사용하는 전체 시스템 프롬프트입니다. TypeScript에서는 `systemPrompt: { type: "preset", preset: "claude_code" }`, Python에서는 `system_prompt={"type": "preset", "preset": "claude_code"}`를 설정하고, 선택적으로 끝에 자체 지침을 추가하기 위해 `append`를 지정할 수 있습니다.
* **커스텀 문자열 (Custom string)**: 직접 작성하는 프롬프트입니다. SDK는 직접 제공한 내용만 전송합니다.

### 출발점 결정하기

결정 요인은 에이전트가 Claude Code(리포지토리에서 작업하고, 사람이 스트리밍 출력을 지켜보며 작업을 제어하는 코딩 에이전트)와 얼마나 유사한지입니다. 제품이 이와 멀어질수록 자신만의 프롬프트를 작성하는 것이 좋습니다.

| 구축하려는 제품 | 추천 선택 | 얻게 되는 내용 |
| :----------------------------------------------------------------------------------------------------------- | :--------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| 사람이 지켜보며 지시하고 Claude Code의 기본값이 원하는 바와 일치하는 CLI 또는 IDE와 유사한 코딩 도구 | `claude_code` 프리셋 | 전체 Claude Code 프롬프트: 도구 지침, 안전 규칙, 터미널 친화적 응답, 리포지토리 규칙 인식 |
| 동일한 유형의 도구에 코딩 표준, 출력 형식 또는 도메인 컨텍스트와 같은 제품 전용 규칙이 추가된 경우 | `append`가 포함된 `claude_code` 프리셋 | 위의 모든 내용과 함께 프리셋 뒤에 지침이 추가됨. 제거되는 내용이 없으므로 가장 위험이 적은 맞춤화 방식임 |
| 표면, 정체성 또는 권한 모델이 다른 에이전트, 또는 비코딩 에이전트 | 커스텀 프롬프트 문자열 | 직접 작성한 내용만 적용됨. 에이전트에 여전히 필요한 도구 지침 및 안전 지침을 교체하는 책임을 직접 집니다 |
| 에이전트 페르소나 없이 사용자 프롬프트에서 모든 동작을 공급하는 얇은 도구 호출 루프 | `systemPrompt` 옵션 없음 | 최소 기본값: 도구 호출 지원만 제공되고 그 외에는 없음 |

"Claude Code와 다름"은 보통 다음 중 하나를 의미합니다:

* **다른 표면 (Different surface)**: 출력이 작업을 트리거한 사람의 터미널에서 읽히지 않는 경우. 챗 UI, 구조화된 출력 소비 애플리케이션, 비코딩 자동화는 각각 출력이 렌더링되고 검토되는 방식에 맞는 프롬프트가 필요합니다. 린트 오류를 수정하거나 diff를 검토하는 CI 작업과 같은 무인 코딩 자동화는 작업 자체 내용이 프리셋의 작성 목적과 부합하므로 여전히 프리셋에 적합합니다.
* **다른 정체성 (Different identity)**: 에이전트가 스스로를 Claude Code로 표현해서는 안 되는 경우. 지원 봇, 데이터 분석 어시스턴트 또는 임의의 도메인 특화 에이전트는 자체 이름, 범위 및 페르소나가 필요합니다.
* **다른 권한 모델 (Different permission model)**: 에이전트가 각 단계를 승인하는 사람 없이 자율적으로 실행되거나 좁은 리소스 세트에서 작동하는 경우. Claude Code의 프롬프트는 사람이 전체 도구 세트에 대한 접근 권한을 가지고 개입되어 있음을 가정합니다.
* **비코딩 작업 (Non-coding tasks)**: Claude Code 프롬프트의 대부분은 코딩 지침입니다. 조사, 콘텐츠 또는 운영 에이전트의 경우 해당 지침이 실제로 필요한 지침과 경합할 수 있습니다.

[비교 표](#4가지-접근-방식-비교)는 각 맞춤화 방법이 보존하는 내용을 보여줍니다.

## 에이전트 동작 맞춤화하기

출력 스타일, `append`, 그리고 커스텀 프롬프트 문자열은 각각 시스템 프롬프트를 직접 변경합니다. CLAUDE.md는 다른 경로를 취합니다: SDK는 이를 읽어서 시스템 프롬프트가 아닌 대화 내에 프로젝트 컨텍스트로 주입하므로, 선택한 시스템 프롬프트와 함께 동작을 형성합니다. [스킬](/docs/en/agent-sdk/skills), [후크](/docs/en/agent-sdk/hooks), [권한](/docs/en/agent-sdk/permissions) 또한 시스템 프롬프트 외부에서 동작을 형성하며 전용 페이지에서 다룹니다.

### 프로젝트 수준 지침을 위한 CLAUDE.md 파일

CLAUDE.md 파일은 Claude에게 영구적인 프로젝트 컨텍스트와 지침을 제공합니다. SDK는 그 내용을 시스템 프롬프트가 아닌 대화에 주입하므로 모든 시스템 프롬프트 구성과 함께 작동합니다. CLAUDE.md에 포함할 내용, 배치할 위치, 효과적인 지침 작성 방법은 [Claude가 프로젝트를 기억하는 방식](/docs/en/memory)을 참조하세요. 이 섹션에서는 SDK에 특화된 내용인 CLAUDE.md 로딩 방식을 다룹니다.

SDK는 해당 설정 소스가 활성화되었을 때 CLAUDE.md를 읽어옵니다: `'project'`는 작업 디렉토리에서 `CLAUDE.md` 또는 `.claude/CLAUDE.md`를 로드하고, `'user'`는 `~/.claude/CLAUDE.md`를 로드합니다. 기본 `query()` 옵션은 두 소스를 모두 활성화하므로 CLAUDE.md가 자동으로 로드됩니다. TypeScript의 `settingSources` 또는 Python의 `setting_sources`를 명시적으로 설정하는 경우 필요한 소스를 포함하세요. CLAUDE.md 로딩은 `claude_code` 프리셋이 아닌 설정 소스에 의해 제어됩니다.

#### SDK로 CLAUDE.md 로드하기

CLAUDE.md를 로드하려면 `settingSources`를 설정하여 CLAUDE.md가 위치한 레벨을 포함하세요. 아래 예제는 `claude_code` 프리셋과 함께 프로젝트 수준 CLAUDE.md를 로드하므로, Claude는 전체 코딩 에이전트 프롬프트와 프로젝트 규칙을 모두 갖게 됩니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const messages = [];

  for await (const message of query({
    prompt: "Add a new React component for user profiles",
    options: {
      systemPrompt: {
        type: "preset",
        preset: "claude_code" // Claude Code의 시스템 프롬프트 사용
      },
      settingSources: ["project"] // 프로젝트에서 CLAUDE.md 로드
    }
  })) {
    messages.push(message);
  }

  // 이제 Claude는 CLAUDE.md의 프로젝트 지침에 접근할 수 있습니다.
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions

  messages = []

  async for message in query(
      prompt="Add a new React component for user profiles",
      options=ClaudeAgentOptions(
          system_prompt={
              "type": "preset",
              "preset": "claude_code",  # Claude Code의 시스템 프롬프트 사용
          },
          setting_sources=["project"],  # 프로젝트에서 CLAUDE.md 로드
      ),
  ):
      messages.append(message)

  # 이제 Claude는 CLAUDE.md의 프로젝트 지침에 접근할 수 있습니다.
  ```
</CodeGroup>

CLAUDE.md는 프로젝트의 모든 세션에 걸쳐 영구적이며, git을 통해 팀과 공유되고, 코드 변경 없이 자동으로 감지됩니다. 빈 `settingSources` 배열을 전달하면 로드되지 않습니다.

### 영구 구성을 위한 출력 스타일 (Output styles)

출력 스타일은 Claude의 시스템 프롬프트를 수정하는 저장된 구성입니다. 마크다운 파일로 저장되며 세션 및 프로젝트 전반에 걸쳐 재사용할 수 있습니다.

#### 출력 스타일 생성하기

출력 스타일은 메타데이터를 위한 [프론트매터(frontmatter)](/docs/en/output-styles#frontmatter)와 그 뒤에 오는 프롬프트 내용으로 구성된 마크다운 파일입니다. 모든 프로젝트에서 사용할 수 있는 사용자 수준 스타일의 경우 `~/.claude/output-styles/`에 저장하고, 팀과 함께 커밋하고 공유할 수 있는 프로젝트 수준 스타일의 경우 리포지토리의 `.claude/output-styles/`에 저장하세요.

기본적으로 커스텀 출력 스타일은 `claude_code` 프리셋의 소프트웨어 공학 지침을 자신만의 지침으로 교체합니다. 해당 지침을 유지하고 그 위에 지침을 레이어링하려면 프론트매터에 `keep-coding-instructions: true`를 설정하세요. 에이전트가 여전히 소프트웨어 공학 작업을 수행하는 경우 이를 유지하세요. 역할 전체를 교체할 때는 제외하세요.

아래 예제는 코드 리뷰가 여전히 Claude Code의 보안 및 코드 품질 지침의 이점을 얻으므로 코딩 지침을 유지하는 코드 리뷰어 페르소나를 정의합니다. 이를 `~/.claude/output-styles/code-reviewer.md`로 저장하여 여러 프로젝트에서 사용할 수 있도록 하세요:

```markdown ~/.claude/output-styles/code-reviewer.md theme={null}
---
name: Code Reviewer
description: Thorough code review assistant
keep-coding-instructions: true
---

You are an expert code reviewer.

For every code submission:
1. Check for bugs and security issues
2. Evaluate performance
3. Suggest improvements
4. Rate code quality (1-10)
```

#### 출력 스타일 활성화하기

생성된 출력 스타일은 다음을 통해 활성화할 수 있습니다:

* **CLI**: `/config`를 실행하고 출력 스타일을 선택합니다.
* **설정**: `.claude/settings.local.json`에서 `outputStyle`을 설정합니다.
* **TypeScript SDK**: `query()`에 전달되는 인라인 `settings` 객체 내부에서 `outputStyle`을 설정하거나, 이를 설정하는 설정 파일을 `settings`로 지정합니다. `outputStyle`은 최상위 `Options` 필드가 아닙니다:

  ```typescript theme={null}
  const options = { settings: { outputStyle: "Explanatory" } };
  ```

Python SDK에는 프로그래밍 방식으로 출력 스타일을 선택하는 옵션이 없습니다. `.claude/settings.local.json`에 쓸 수 없는 코드 전용 배포의 경우 `append` 또는 커스텀 프롬프트 문자열을 대신 사용하세요.

**SDK 사용자를 위한 참고 사항:** 출력 스타일은 옵션에 TypeScript의 `settingSources: ['user']` 또는 `settingSources: ['project']` / Python의 `setting_sources=["user"]` 또는 `setting_sources=["project"]`를 포함할 때 로드됩니다.

### `claude_code` 프리셋에 추가하기 (`append`)

`append` 속성과 함께 Claude Code 프리셋을 사용하여 모든 내장 기능을 유지하면서 커스텀 지침을 추가할 수 있습니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const messages = [];

  for await (const message of query({
    prompt: "Help me write a Python function to calculate fibonacci numbers",
    options: {
      systemPrompt: {
        type: "preset",
        preset: "claude_code",
        append: "Always include detailed docstrings and type hints in Python code."
      }
    }
  })) {
    messages.push(message);
    if (message.type === "assistant") {
      console.log(message.message.content);
    }
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage

  messages = []

  async for message in query(
      prompt="Help me write a Python function to calculate fibonacci numbers",
      options=ClaudeAgentOptions(
          system_prompt={
              "type": "preset",
              "preset": "claude_code",
              "append": "Always include detailed docstrings and type hints in Python code.",
          }
      ),
  ):
      messages.append(message)
      if isinstance(message, AssistantMessage):
          print(message.content)
  ```
</CodeGroup>

#### 사용자 및 머신 간 프롬프트 캐싱 향상

기본적으로 동일한 `claude_code` 프리셋과 `append` 텍스트를 사용하는 두 세션이라도 서로 다른 작업 디렉토리에서 실행되면 프롬프트 캐시 항목을 공유할 수 없습니다. 이는 프리셋이 `append` 텍스트 앞에 세션별 컨텍스트(작업 디렉토리, git 리포지토리 여부, 플랫폼, 활성 쉘, OS 버전, 자동 메모리 경로)를 포함하기 때문입니다. 해당 컨텍스트의 차이는 다른 시스템 프롬프트를 생성하고 캐시 미스를 유발합니다. CLAUDE.md 내용은 SDK가 시스템 프롬프트가 아닌 대화에 주입하므로 시스템 프롬프트 캐시에 영향을 주지 않습니다.

세션 간에 시스템 프롬프트를 동일하게 만들려면 TypeScript에서 `excludeDynamicSections: true`, Python에서 `"exclude_dynamic_sections": True`를 설정하세요. 세션별 컨텍스트가 첫 번째 사용자 메시지로 이동하여 정적 프리셋과 `append` 텍스트만 시스템 프롬프트에 남으므로 동일한 구성이 사용자 및 머신 간에 캐시 항목을 공유하게 됩니다.

<Note>
  `excludeDynamicSections`는 `@anthropic-ai/claude-agent-sdk` v0.2.98 이상, 또는 Python의 경우 `claude-agent-sdk` v0.1.58 이상이 필요합니다. 프리셋 객체 형식에만 적용되며 `systemPrompt`가 문자열일 때는 효과가 없습니다.
</Note>

다음 예제는 서로 다른 디렉토리에서 실행되는 에이전트 플릿이 동일한 캐시된 시스템 프롬프트를 재사용할 수 있도록 공유 `append` 블록과 `excludeDynamicSections`를 조합합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Triage the open issues in this repo",
    options: {
      systemPrompt: {
        type: "preset",
        preset: "claude_code",
        append: "You operate Acme's internal triage workflow. Label issues by component and severity.",
        excludeDynamicSections: true
      }
    }
  })) {
    // ...
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions

  async for message in query(
      prompt="Triage the open issues in this repo",
      options=ClaudeAgentOptions(
          system_prompt={
              "type": "preset",
              "preset": "claude_code",
              "append": "You operate Acme's internal triage workflow. Label issues by component and severity.",
              "exclude_dynamic_sections": True,
          },
      ),
  ):
      ...
  ```
</CodeGroup>

**트레이드오프:** 작업 디렉토리, git-repo 플래그, 플랫폼, 활성 쉘, OS 버전 및 자동 메모리 경로는 시스템 프롬프트가 아닌 첫 번째 사용자 메시지의 일부로 Claude에 계속 전달됩니다. 사용자 메시지의 지침은 시스템 프롬프트의 동일한 텍스트보다 가중치가 약간 적으므로 Claude가 현재 디렉토리나 자동 메모리 경로에 대해 추론할 때 지침을 덜 엄격하게 의존할 수 있습니다. 세션 간 캐시 재사용이 최대한의 권위 있는 환경 컨텍스트보다 중요할 때 이 옵션을 활성화하세요.

비대화형 CLI 모드에서의 동등한 플래그는 [`--exclude-dynamic-system-prompt-sections`](/docs/en/cli-reference)를 참조하세요.

### 커스텀 시스템 프롬프트

기본값을 자신만의 지침으로 완전히 교체하기 위해 커스텀 문자열을 `systemPrompt`로 제공할 수 있습니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const customPrompt = `You are a Python coding specialist.
  Follow these guidelines:
  - Write clean, well-documented code
  - Use type hints for all functions
  - Include comprehensive docstrings
  - Prefer functional programming patterns when appropriate
  - Always explain your code choices`;

  const messages = [];

  for await (const message of query({
    prompt: "Create a data processing pipeline",
    options: {
      systemPrompt: customPrompt
    }
  })) {
    messages.push(message);
    if (message.type === "assistant") {
      console.log(message.message.content);
    }
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage

  custom_prompt = """You are a Python coding specialist.
  Follow these guidelines:
  - Write clean, well-documented code
  - Use type hints for all functions
  - Include comprehensive docstrings
  - Prefer functional programming patterns when appropriate
  - Always explain your code choices"""

  messages = []

  async for message in query(
      prompt="Create a data processing pipeline",
      options=ClaudeAgentOptions(system_prompt=custom_prompt),
  ):
      messages.append(message)
      if isinstance(message, AssistantMessage):
          print(message.content)
  ```
</CodeGroup>

## 4가지 접근 방식 비교

4가지 맞춤화 방법은 거주 위치, 공유 방식, 그리고 `claude_code` 프리셋에서 보존하는 항목이 다릅니다.

| 기능 | CLAUDE.md | 출력 스타일 | append가 포함된 `systemPrompt` | 커스텀 `systemPrompt` |
| ----------------------- | ---------------- | ------------------------- | -------------------------- | ---------------------- |
| **영속성** | 프로젝트별 파일 | 파일로 저장됨 | 세션 전용 | 세션 전용 |
| **재사용성** | 프로젝트별 | 프로젝트 전반 | 코드 중복 | 코드 중복 |
| **관리 방식** | 파일 시스템 상에서 | CLI + 파일 | 코드 내부 | 코드 내부 |
| **기본 도구** | 보존됨 | 보존됨 | 보존됨 | 손실됨 (포함하지 않는 한) |
| **내장 안전성** | 유지됨 | 유지됨 | 유지됨 | 직접 추가해야 함 |
| **환경 컨텍스트** | 자동 | 자동 | 자동 | 직접 제공해야 함 |
| **맞춤화 수준** | 추가 전용 | 기본값 교체 또는 확장 | 추가 전용 | 완전한 제어 |
| **버전 제어** | 프로젝트와 함께 | 예 | 코드와 함께 | 코드와 함께 |
| **범위** | 프로젝트 특화 | 사용자 또는 프로젝트 | 코드 세션 | 코드 세션 |

"append 포함"은 TypeScript에서 `systemPrompt: { type: "preset", preset: "claude_code", append: "..." }`, Python에서 `system_prompt={"type": "preset", "preset": "claude_code", "append": "..."}`를 사용하는 것을 의미합니다. CLAUDE.md는 시스템 프롬프트 자체를 변경하지 않습니다: SDK는 그 내용을 대화에 프로젝트 컨텍스트로 주입합니다.

## 유즈케이스 및 모범 사례

### CLAUDE.md를 사용하는 경우

세션이 사용하는 시스템 프롬프트에 관계없이 프로젝트의 모든 세션에 적용되어야 하는 지침(코딩 표준, 공통 명령, 아키텍처 컨텍스트, 팀 규칙)에는 CLAUDE.md를 사용하세요. CLAUDE.md는 리포지토리에 커밋되므로 설명하는 코드와 함께 동기화 상태가 유지됩니다. 전체 가이드는 [CLAUDE.md에 추가할 시점](/docs/en/memory#when-to-add-to-claude-md)을 참조하세요.

CLAUDE.md 파일은 기본 `query()` 옵션에서 활성화되어 있는 `project` 설정 소스가 켜져 있을 때 로드됩니다. TypeScript의 `settingSources` 또는 Python의 `setting_sources`를 명시적으로 설정하는 경우 프로젝트 수준 CLAUDE.md를 계속 로드하려면 `'project'`를 포함하세요.

### 출력 스타일을 사용하는 경우

출력 스타일은 애플리케이션 코드를 변경하지 않고 CLI와 SDK 전반에 걸쳐 재사용하려는 페르소나를 위한 것입니다. `.claude/output-styles`에 파일로 존재하므로 CLI의 `/config` 및 일치하는 설정 소스를 로드하는 모든 SDK 세션에서 동일한 페르소나를 이용할 수 있습니다.

**가장 적합한 경우:**

* 세션 간의 영구적인 동작 변경
* 팀 공유 구성
* 코드 리뷰어, 데이터 사이언티스트, DevOps 어시스턴트와 같은 특화된 어시스턴트
* 버전 관리가 필요한 복잡한 프롬프트 수정

**예시:**

* 전용 SQL 최적화 어시스턴트 생성
* 보안 중심 코드 리뷰어 구축
* 특정 교수법을 가진 학습 어시스턴트 개발

### append가 포함된 `systemPrompt`를 사용하는 경우

`claude_code` 프리셋이 제품에 이미 적합하고 추가 지침만 계층화하면 되는 경우 `append`를 사용하세요. 프리셋의 도구 지침, 안전 규칙 및 코딩 규칙을 재구현하지 않고 그대로 유지할 수 있습니다.

**가장 적합한 경우:**

* 특정 코딩 표준 또는 선호 사항 추가
* 출력 서식 맞춤화
* 도메인 특화 지식 추가
* 응답 상세도 수정
* 도구 지침을 잃지 않고 Claude Code의 기본 동작 강화

### 커스텀 `systemPrompt`를 사용하는 경우

[출발점 결정하기](#출발점-결정하기)에서 설명했듯이 에이전트의 표면, 정체성 또는 권한 모델이 Claude Code와 다를 때 커스텀 프롬프트를 사용하세요. 에이전트에 필요한 도구 지침 및 안전 규칙을 포함하여 전체 지침 세트를 직접 정의합니다.

**가장 적합한 경우:**

* Claude 동작에 대한 완전한 제어
* 특화된 단일 세션 작업
* 새 프롬프트 전략 테스트
* 기본 도구가 필요하지 않은 상황
* 고유한 동작을 가진 특화된 에이전트 구축

## 접근 방식 결합하기

이러한 방법들은 서로 결합(compose)할 수 있습니다. 영구 출력 스타일이나 CLAUDE.md가 장기간 유지되는 동작을 설정하고, `append`는 저장된 구성을 건드리지 않고 그 위에 세션 특화 지침을 계층화합니다.

### 출력 스타일과 세션 특화 추가 사항 결합하기

아래 예제는 코드 리뷰어 출력 스타일이 이미 활성화되어 있다고 가정합니다. `append` 블록은 페르소나 위에 세션 특화 집중 영역을 계층화하므로, 단일 리뷰 세션이 저장된 출력 스타일을 변경하지 않고 OAuth 및 토큰 저장소의 우선순위를 지정할 수 있습니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // "Code Reviewer" 출력 스타일이 활성화되어 있다고 가정 (/config 또는 설정 사용)
  // 세션 특화 집중 영역 추가
  const messages = [];

  for await (const message of query({
    prompt: "Review this authentication module",
    options: {
      systemPrompt: {
        type: "preset",
        preset: "claude_code",
        append: `
          For this review, prioritize:
          - OAuth 2.0 compliance
          - Token storage security
          - Session management
        `
      }
    }
  })) {
    messages.push(message);
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions

  # "Code Reviewer" 출력 스타일이 활성화되어 있다고 가정 (/config 또는 설정 사용)
  # 세션 특화 집중 영역 추가
  messages = []

  async for message in query(
      prompt="Review this authentication module",
      options=ClaudeAgentOptions(
          system_prompt={
              "type": "preset",
              "preset": "claude_code",
              "append": """
              For this review, prioritize:
              - OAuth 2.0 compliance
              - Token storage security
              - Session management
              """,
          }
      ),
  ):
      messages.append(message)
  ```
</CodeGroup>

## 참고 항목

* [출력 스타일](/docs/en/output-styles): 파일 형식 및 저장 위치를 포함하여 CLI용 출력 스타일 생성, 관리 및 공유
* [Claude가 프로젝트를 기억하는 방식](/docs/en/memory): CLAUDE.md에 넣을 내용, 배치 위치, 효과적인 프로젝트 지침 작성 방법
* [TypeScript SDK 참조](/docs/en/agent-sdk/typescript): `systemPrompt`, `settingSources`, `settings`를 포함한 전체 `Options` 타입
* [Python SDK 참조](/docs/en/agent-sdk/python): `system_prompt` 및 `setting_sources`를 포함한 전체 `ClaudeAgentOptions` 타입
* [설정](/docs/en/settings): 출력 스타일 및 기타 구성이 저장되는 위치를 포함한 `settings.json` 참조
