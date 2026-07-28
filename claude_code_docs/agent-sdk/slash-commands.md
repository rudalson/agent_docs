> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# SDK의 Slash Commands (슬래시 명령어)

> SDK를 통해 슬래시 명령어를 사용하여 Claude Code 세션을 제어하는 방법을 알아봅니다.

슬래시 명령어는 `/`로 시작하는 특별한 명령어로 Claude Code 세션을 제어하는 방법을 제공합니다. 이러한 명령어는 SDK를 통해 전송되어 컨텍스트 압축, 컨텍스트 사용량 조회 또는 커스텀 명령어 호출과 같은 작업을 수행할 수 있습니다. 대화형 터미널 없이 작동하는 명령어만 SDK를 통해 디스패치할 수 있으며, `system/init` 메시지에서 세션에서 사용 가능한 명령어를 나열합니다.

## 사용 가능한 슬래시 명령어 탐색하기

Claude Agent SDK는 시스템 초기화 메시지에서 사용 가능한 슬래시 명령어에 대한 정보를 제공합니다. 세션이 시작될 때 이 정보에 접근할 수 있습니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Hello Claude",
    options: { maxTurns: 1 }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      console.log("Available slash commands:", message.slash_commands);
      // 내장 명령어 및 번들 스킬 포함, 예시:
      // ["clear", "compact", "context", "usage", "code-review", "verify", ...]
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage


  async def main():
      async for message in query(prompt="Hello Claude", options=ClaudeAgentOptions(max_turns=1)):
          if isinstance(message, SystemMessage) and message.subtype == "init":
              print("Available slash commands:", message.data["slash_commands"])
              # 내장 명령어 및 번들 스킬 포함, 예시:
              # ["clear", "compact", "context", "usage", "code-review", "verify", ...]


  asyncio.run(main())
  ```
</CodeGroup>

## 슬래시 명령어 전송하기

일반 텍스트와 마찬가지로 프롬프트 문자열에 슬래시 명령어를 포함하여 전송합니다. `/compact`와 같이 대화 기록에 작용하는 명령어는 작업할 이전 메시지가 필요하므로, 아래 예시에서는 먼저 질문을 한 다음 동일한 대화의 후속 요청으로 명령어를 전송합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 먼저 대화 기록 쌓기
  try {
    for await (const message of query({
      prompt: "What does the README in this directory cover?",
      options: { maxTurns: 2 }
    })) {
      if (message.type === "result" && message.subtype === "success") {
        console.log(message.result);
      }
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
    // 아래의 후속 쿼리는 여전히 실행됩니다.
    console.error(`Session ended with an error: ${error}`);
  }

  // 동일한 대화에 대한 후속 요청으로 슬래시 명령어 전송
  for await (const message of query({
    prompt: "/compact",
    options: { continue: true, maxTurns: 1 }
  })) {
    if (message.type === "result") {
      console.log("Command executed, result subtype:", message.subtype);
      // 출력 예시: Command executed, result subtype: success
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      # 먼저 대화 기록 쌓기
      try:
          async for message in query(
              prompt="What does the README in this directory cover?",
              options=ClaudeAgentOptions(max_turns=2),
          ):
              if isinstance(message, ResultMessage) and message.subtype == "success":
                  print(message.result)
      except Exception as error:
          # 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
          # 아래의 후속 쿼리는 여전히 실행됩니다.
          print(f"Session ended with an error: {error}")

      # 동일한 대화에 대한 후속 요청으로 슬래시 명령어 전송
      async for message in query(
          prompt="/compact",
          options=ClaudeAgentOptions(continue_conversation=True, max_turns=1),
      ):
          if isinstance(message, ResultMessage):
              print("Command executed, result subtype:", message.subtype)
              # 출력 예시: Command executed, result subtype: success


  asyncio.run(main())
  ```
</CodeGroup>

<Note>
  쿼리는 작업이 완료되기 전에 `maxTurns` / `max_turns` 제한에 도달하는 등의 이유로 오류 결과로 종료될 수 있습니다. 이때 최종 결과 메시지는 `success` 대신 `is_error: true` 및 `error_max_turns`와 같은 오류 서브타입을 갖습니다.

  해당 최종 결과 메시지를 반환한 후 CLI 프로세스가 0이 아닌 코드로 종료되므로 SDK는 에러를 발생시킵니다.

  [단일 메시지 입력](/docs/en/agent-sdk/streaming-vs-single-mode#single-message-input)에 나와 있듯이 명령어가 제한에 도달할 수 있는 경우 TypeScript의 `try`/`catch` 또는 Python의 `try`/`except`로 루프를 감싸거나, 작업이 완료될 수 있도록 `maxTurns`를 충분히 크게 설정하세요. Python에서는 `Exception`을 캐치하세요. SDK는 오류 결과를 일반 `Exception`으로 노출합니다.
</Note>

## 주요 슬래시 명령어

### `/compact` - 대화 기록 압축

`/compact` 명령어는 중요한 컨텍스트를 유지하면서 이전 메시지를 요약하여 대화 기록의 크기를 줄입니다. 압축하려면 요약할 최소 2개 이상의 이전 주고받은 메시지가 있는 기존 대화가 필요합니다. 이 예시에서는 먼저 대화를 진행한 후 이를 압축하고 결과를 보고하는 `compact_boundary` 시스템 메시지를 읽습니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 압축에는 기존 기록이 필요하므로 먼저 대화를 진행합니다
  try {
    for await (const message of query({
      prompt: "Explain what this project does",
      options: { maxTurns: 2 }
    })) {
      if (message.type === "result" && message.subtype === "success") {
        console.log(message.result);
      }
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
    // 아래의 후속 쿼리는 여전히 실행됩니다.
    console.error(`Session ended with an error: ${error}`);
  }

  // 동일한 대화 압축
  for await (const message of query({
    prompt: "/compact",
    options: { continue: true, maxTurns: 1 }
  })) {
    if (message.type === "system" && message.subtype === "compact_boundary") {
      console.log("Compaction completed");
      console.log("Pre-compaction tokens:", message.compact_metadata.pre_tokens);
      console.log("Trigger:", message.compact_metadata.trigger);
      // 출력 예시:
      // Compaction completed
      // Pre-compaction tokens: 1842
      // Trigger: manual
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage


  async def main():
      # 압축에는 기존 기록이 필요하므로 먼저 대화를 진행합니다
      try:
          async for message in query(
              prompt="Explain what this project does",
              options=ClaudeAgentOptions(max_turns=2),
          ):
              if isinstance(message, ResultMessage) and message.subtype == "success":
                  print(message.result)
      except Exception as error:
          # 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
          # 아래의 후속 쿼리는 여전히 실행됩니다.
          print(f"Session ended with an error: {error}")

      # 동일한 대화 압축
      async for message in query(
          prompt="/compact",
          options=ClaudeAgentOptions(continue_conversation=True, max_turns=1),
      ):
          if isinstance(message, SystemMessage) and message.subtype == "compact_boundary":
              print("Compaction completed")
              print("Pre-compaction tokens:", message.data["compact_metadata"]["pre_tokens"])
              print("Trigger:", message.data["compact_metadata"]["trigger"])
              # 출력 예시:
              # Compaction completed
              # Pre-compaction tokens: 1842
              # Trigger: manual


  asyncio.run(main())
  ```
</CodeGroup>

<Note>
  `compact_boundary` 메시지는 압축이 실행되었을 때만 전달됩니다. 요약할 항목이 없으면 `/compact`는 에러를 발생하는 대신 이유를 보고합니다. 실행은 여전히 `success` 결과로 끝나고 `compact_boundary` 메시지는 출력되지 않으며, 결과 텍스트에 메시지가 포함됩니다(예: 짧은 대화 1회 후 `Not enough messages to compact.`). 새로 호출하는 단일 실행 `query()`는 빈 컨텍스트로 시작하므로 [스트리밍 입력 모드](/docs/en/agent-sdk/streaming-vs-single-mode)나 세션을 재개할 때와 같이 이전 차례가 있는 세션에서 이 패턴을 사용하세요.
</Note>

### `/clear` - 대화 컨텍스트 초기화

`/clear` 명령어는 대화를 빈 컨텍스트로 초기화하므로 이후 프롬프트는 이전 대화 기록 없이 시작됩니다. 이전 대화는 디스크에 남아 있으며 세션 ID를 [`resume` 옵션](/docs/en/agent-sdk/sessions#resume-by-id)에 전달하여 돌아갈 수 있습니다.

이는 단일 연결을 통해 여러 프롬프트를 보내는 [스트리밍 입력 모드](/docs/en/agent-sdk/streaming-vs-single-mode)에서 유용합니다. 단일 실행 `query()` 호출의 경우 각 호출이 이미 빈 컨텍스트로 시작하므로 `/clear`를 보내는 것은 실질적인 효과가 없으며, 대신 새 `query()`를 시작하세요.

<Note>
  SDK에서 `/clear`를 사용하려면 Claude Code v2.1.117 이상이 필요합니다. 이전 버전에서는 `slash_commands`에서 생략됩니다.
</Note>

## 커스텀 슬래시 명령어 생성하기

내장 슬래시 명령어를 사용하는 것 외에도 SDK를 통해 사용할 수 있는 자체 커스텀 명령어를 생성할 수 있습니다. 커스텀 명령어는 서브에이전트가 구성되는 방식과 유사하게 특정 디렉터리의 markdown 파일로 정의됩니다.

<Note>
  `.claude/commands/` 디렉터리는 기존(legacy) 형식입니다. 권장 형식은 `.claude/skills/<name>/SKILL.md`이며, 동일한 슬래시 명령어 호출(`/name`)뿐만 아니라 Claude의 자율 호출도 지원합니다. 현재 형식은 [Skills](/docs/en/agent-sdk/skills)를 참조하세요. CLI는 두 형식을 모두 계속 지원하며 아래 예시는 `.claude/commands/`에 대해서도 계속 정확하게 적용됩니다.
</Note>

### 파일 위치

커스텀 슬래시 명령어는 스코프에 따라 지정된 디렉터리에 저장됩니다:

* **프로젝트 명령어**: `.claude/commands/` - 현재 프로젝트에서만 사용 가능 (기존 방식, `.claude/skills/` 권장)
* **개인 명령어**: `~/.claude/commands/` - 모든 프로젝트에서 사용 가능 (기존 방식, `~/.claude/skills/` 권장)

### 파일 형식

각 커스텀 명령어는 다음과 같은 markdown 파일입니다:

* 파일 이름(`.md` 확장자 제외)이 명령어 이름이 됩니다.
* 파일 내용은 명령어의 동작을 정의합니다.
* 선택 사항인 YAML 프론트매터는 구성을 제공합니다.

#### 기본 예시

프로젝트에 `.claude/commands` 디렉터리가 없으면 생성한 다음 `.claude/commands/refactor.md`를 생성합니다:

```markdown theme={null}
Refactor the selected code to improve readability and maintainability.
Focus on clean code principles and best practices.
```

이렇게 하면 SDK를 통해 사용할 수 있는 `/refactor` 명령어 가 생성됩니다.

#### 프론트매터 포함 예시

`.claude/commands/security-check.md`를 생성합니다:

```markdown theme={null}
---
allowed-tools: Read, Grep, Glob
description: Run security vulnerability scan
model: claude-opus-4-8
---

Analyze the codebase for security vulnerabilities including:
- SQL injection risks
- XSS vulnerabilities
- Exposed credentials
- Insecure configurations
```

### SDK에서 커스텀 명령어 사용하기

파일 시스템에 정의되면 커스텀 명령어를 SDK를 통해 자동으로 사용할 수 있습니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 커스텀 명령어 사용
  try {
    for await (const message of query({
      prompt: "/refactor src/auth/login.ts",
      options: { maxTurns: 3 }
    })) {
      if (message.type === "assistant") {
        console.log("Refactoring suggestions:", message.message);
      }
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
    // 아래의 두 번째 쿼리는 여전히 실행됩니다.
    console.error(`Session ended with an error: ${error}`);
  }

  // 커스텀 명령어는 slash_commands 목록에 나타납니다
  for await (const message of query({
    prompt: "Hello",
    options: { maxTurns: 1 }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      console.log("Available commands:", message.slash_commands);
      // 내장 명령어, 번들 스킬 및 커스텀 명령어 포함, 예시:
      // ["clear", "compact", "context", "usage", "code-review", "verify", "refactor", "security-check", ...]
    }
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, SystemMessage


  async def main():
      # 커스텀 명령어 사용
      try:
          async for message in query(
              prompt="/refactor src/auth/login.py", options=ClaudeAgentOptions(max_turns=3)
          ):
              if isinstance(message, AssistantMessage):
                  for block in message.content:
                      if hasattr(block, "text"):
                          print("Refactoring suggestions:", block.text)
      except Exception as error:
          # 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
          # 아래의 두 번째 쿼리는 여전히 실행됩니다.
          print(f"Session ended with an error: {error}")

      # 커스텀 명령어는 slash_commands 목록에 나타납니다
      async for message in query(prompt="Hello", options=ClaudeAgentOptions(max_turns=1)):
          if isinstance(message, SystemMessage) and message.subtype == "init":
              print("Available commands:", message.data["slash_commands"])
              # 내장 명령어, 번들 스킬 및 커스텀 명령어 포함, 예시:
              # ["clear", "compact", "context", "usage", "code-review", "verify", "refactor", "security-check", ...]


  asyncio.run(main())
  ```
</CodeGroup>

### 고급 기능

#### 인자 및 플레이스홀더 (Arguments and Placeholders)

커스텀 명령어는 플레이스홀더를 사용하여 동적 인자를 지원합니다:

`.claude/commands/fix-issue.md`를 생성합니다:

```markdown theme={null}
---
argument-hint: [issue-number] [priority]
description: Fix a GitHub issue
---

Fix issue #$0 with priority $1.
Check the issue description and implement the necessary changes.
```

SDK에서 사용하기:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 커스텀 명령어에 인자 전달
  try {
    for await (const message of query({
      prompt: "/fix-issue 123 high",
      options: { maxTurns: 5 }
    })) {
      // 명령어는 $0="123" 및 $1="high"로 처리됩니다
      if (message.type === "result" && message.subtype === "success") {
        console.log("Issue fixed:", message.result);
      }
    }
  } catch (err) {
    // maxTurns 제한에 도달하면 실행이 에러와 함께 종료됩니다
    console.error("Session ended with an error:", err);
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


  async def main():
      # 커스텀 명령어에 인자 전달
      try:
          async for message in query(prompt="/fix-issue 123 high", options=ClaudeAgentOptions(max_turns=5)):
              # 명령어는 $0="123" 및 $1="high"로 처리됩니다
              if isinstance(message, ResultMessage):
                  print("Issue fixed:", message.result)
      except Exception as error:
          # max_turns 제한에 도달하면 실행이 에러와 함께 종료됩니다
          print(f"Session ended with an error: {error}")


  asyncio.run(main())
  ```
</CodeGroup>

#### Bash 명령어 실행

커스텀 명령어는 bash 명령어를 실행하고 그 출력을 포함할 수 있습니다:

`.claude/commands/git-commit.md`를 생성합니다:

```markdown theme={null}
---
allowed-tools: Bash(git add *), Bash(git status *), Bash(git commit *)
description: Create a git commit
---

## Context

- Current status: !`git status`
- Current diff: !`git diff HEAD`

## Task

Create a git commit with appropriate message based on the changes.
```

#### 파일 참조

`@` 접두사를 사용하여 파일 내용을 포함합니다:

`.claude/commands/review-config.md`를 생성합니다:

```markdown theme={null}
---
description: Review configuration files
---

Review the following configuration files for issues:
- Package config: @package.json
- TypeScript config: @tsconfig.json
- Environment config: @.env

Check for security issues, outdated dependencies, and misconfigurations.
```

### 네임스페이스를 활용한 조직화

더 나은 구조를 위해 하위 디렉터리에 명령어를 조직화합니다:

```bash theme={null}
.claude/commands/
├── frontend/
│   ├── component.md      # /component 생성 (project:frontend)
│   └── style-check.md     # /style-check 생성 (project:frontend)
├── backend/
│   ├── api-test.md        # /api-test 생성 (project:backend)
│   └── db-migrate.md      # /db-migrate 생성 (project:backend)
└── review.md              # /review 생성 (project)
```

하위 디렉터리는 명령어 설명에 나타나지만 명령어 이름 자체에는 영향을 주지 않습니다.

### 실용적인 예시

#### Pull Request 리뷰 명령어

`.claude/commands/review-pr.md`를 생성합니다:

```markdown theme={null}
---
allowed-tools: Read, Grep, Glob, Bash(git diff *)
description: Comprehensive code review
---

## Changed Files
!`git diff --name-only HEAD~1`

## Detailed Changes
!`git diff HEAD~1`

## Review Checklist

Review the above changes for:
1. Code quality and readability
2. Security vulnerabilities
3. Performance implications
4. Test coverage
5. Documentation completeness

Provide specific, actionable feedback organized by priority.
```

<Note>
  Claude Code에는 번들로 제공되는 `code-review` 및 `verify` 스킬이 포함되어 있습니다. 커스텀 명령어 이름(예: `.claude/commands/code-review.md`)을 이들과 동일하게 지정하면 커스텀 명령어가 번들 스킬을 오버라이드하며 `slash_commands`에는 해당 이름이 한 번만 나열됩니다.
</Note>

#### 테스트 러너 명령어

`.claude/commands/test.md`를 생성합니다:

```markdown theme={null}
---
allowed-tools: Bash, Read, Edit
argument-hint: [test-pattern]
description: Run tests with optional pattern
---

Run tests matching pattern: $ARGUMENTS

1. Detect the test framework (Jest, pytest, etc.)
2. Run tests with the provided pattern
3. If tests fail, analyze and fix them
4. Re-run to verify fixes
```

SDK를 통해 이러한 명령어를 사용하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 코드 리뷰 실행
  try {
    for await (const message of query({
      prompt: "/review-pr",
      options: { maxTurns: 3 }
    })) {
      // 리뷰 피드백 처리
    }
  } catch (error) {
    // 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
    // 아래의 두 번째 쿼리는 여전히 실행됩니다.
    console.error(`Session ended with an error: ${error}`);
  }

  // 특정 테스트 실행
  for await (const message of query({
    prompt: "/test auth",
    options: { maxTurns: 5 }
  })) {
    // 테스트 결과 처리
  }
  ```

  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions


  async def main():
      # 코드 리뷰 실행
      try:
          async for message in query(prompt="/review-pr", options=ClaudeAgentOptions(max_turns=3)):
              # 리뷰 피드백 처리
              pass
      except Exception as error:
          # 단일 실행 query()는 오류 결과를 반환한 후 에러를 발생시키므로,
          # 아래의 두 번째 쿼리는 여전히 실행됩니다.
          print(f"Session ended with an error: {error}")

      # 특정 테스트 실행
      async for message in query(prompt="/test auth", options=ClaudeAgentOptions(max_turns=5)):
          # 테스트 결과 처리
          pass


  asyncio.run(main())
  ```
</CodeGroup>

## 참고 항목

* [Slash Commands](/docs/en/skills) - 전체 슬래시 명령어 문서
* [SDK의 Subagents](/docs/en/agent-sdk/subagents) - 서브에이전트를 위한 유사한 파일 시스템 기반 구성
* [TypeScript SDK 레퍼런스](/docs/en/agent-sdk/typescript) - 전체 API 문서
* [SDK 개요](/docs/en/agent-sdk/overview) - 일반 SDK 개념
* [CLI 레퍼런스](/docs/en/cli-reference) - 명령줄 인터페이스
