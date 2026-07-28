> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# SDK의 Agent Skills (에이전트 스킬)

> Claude Agent SDK에서 Agent Skills를 사용하여 특화된 기능으로 Claude를 확장하세요.

## 개요 (Overview)

Agent Skills는 Claude가 관련이 있을 때 자율적으로 호출하는 특화된 기능으로 Claude를 확장합니다. Skills는 지침(instructions), 설명(descriptions) 및 선택적 지원 리소스를 포함하는 `SKILL.md` 파일로 패키징됩니다.

이점, 아키텍처 및 작성 지침을 포함하여 Skills에 대한 포괄적인 정보는 [Agent Skills 개요](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)를 참조하세요.

## SDK에서 Skills가 작동하는 방식

Claude Agent SDK를 사용할 때 Skills는 다음과 같이 처리됩니다:

1. **파일 시스템 아티팩트로 정의됨**: 특정 디렉터리(`.claude/skills/`)에 `SKILL.md` 파일로 생성됩니다.
2. **파일 시스템에서 로드됨**: `settingSources` (TypeScript) 또는 `setting_sources` (Python)에 의해 제어되는 파일 시스템 위치에서 Skills가 로드됩니다.
3. **자동으로 탐색됨**: 파일 시스템 설정이 로드되면 시작 시 사용자 및 프로젝트 디렉터리에서 Skill 메타데이터가 탐색되며, 트리거될 때 전체 콘텐츠가 로드됩니다.
4. **모델에 의해 호출됨**: Claude는 컨텍스트에 따라 이를 사용할 시기를 자율적으로 선택합니다.
5. **`skills` 옵션을 통해 필터링됨**: 탐색된 스킬은 기본적으로 활성화됩니다. 스킬 이름 목록, `"all"` 또는 `[]`를 전달하여 세션에서 사용할 수 있는 스킬을 제어할 수 있습니다.

프로그래밍 방식으로 정의할 수 있는 서브에이전트(subagents)와 달리 Skills는 파일 시스템 아티팩트로 생성해야 합니다. SDK는 Skills를 등록하기 위한 프로그래밍 방식의 API를 제공하지 않습니다.

<Note>
  Skills는 파일 시스템 설정 소스를 통해 탐색됩니다. 기본 `query()` 옵션을 사용하면 SDK가 사용자 및 프로젝트 소스를 로드하므로 `~/.claude/skills/`, `<cwd>/.claude/skills/` 및 리포지토리 루트까지의 모든 상위 디렉터리의 `.claude/skills/`에 있는 스킬을 사용할 수 있습니다. `settingSources`를 명시적으로 설정하는 경우 `'user'` 또는 `'project'`를 포함하여 스킬 탐색을 유지하거나, [`plugins` 옵션](/docs/en/agent-sdk/plugins)을 사용하여 특정 경로에서 스킬을 로드하세요.
</Note>

## SDK에서 Skills 사용하기

`query()`에서 `skills` 옵션을 설정하여 세션에서 사용할 수 있는 Skills를 제어합니다. 생략하면 탐색된 Skills가 활성화되고 Skill 도구를 사용할 수 있어 CLI 동작과 일치합니다. 모든 탐색된 Skill을 활성화하려면 `"all"`, 해당 스킬만 활성화하려면 Skill 이름 목록, 모두 비활성화하려면 `[]`를 전달합니다. `skills`를 설정하면 SDK가 `allowedTools`에 Skill 도구를 자동으로 추가합니다. 명시적인 `tools` 목록도 전달하는 경우 Claude가 스킬을 호출할 수 있도록 해당 목록에 `"Skill"`을 포함하세요.

구성이 완료되면 Claude는 파일 시스템에서 Skills를 자동으로 탐색하고 사용자의 요청과 관련이 있을 때 이를 호출합니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions


  async def main():
      options = ClaudeAgentOptions(
          cwd="/path/to/project",  # .claude/skills/가 있는 프로젝트
          setting_sources=["user", "project"],  # 파일 시스템에서 Skills 로드
          skills="all",  # 탐색된 모든 Skill 활성화
          allowed_tools=["Read", "Write", "Bash"],
      )

      async for message in query(
          prompt="Help me process this PDF document", options=options
      ):
          print(message)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  for await (const message of query({
    prompt: "Help me process this PDF document",
    options: {
      cwd: "/path/to/project", // .claude/skills/가 있는 프로젝트
      settingSources: ["user", "project"], // 파일 시스템에서 Skills 로드
      skills: "all", // 탐색된 모든 Skill 활성화
      allowedTools: ["Read", "Write", "Bash"]
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

특정 Skills만 활성화하려면 해당 이름을 전달하세요. 이름은 `SKILL.md`의 `name` 필드 또는 Skill의 디렉터리 이름과 일치합니다. 플러그인이 제공하는 Skills의 경우 `plugin:skill`을 사용하세요.

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(skills=["pdf", "docx"])
  ```

  ```typescript TypeScript theme={null}
  const options = { skills: ["pdf", "docx"] };
  ```
</CodeGroup>

`skills` 옵션은 샌드박스가 아닌 컨텍스트 필터입니다. 목록에 없는 Skills는 모델로부터 숨겨지고 Skill 도구에 의해 거부되지만, 해당 파일은 디스크에 그대로 남아 있으며 Read 및 Bash를 통해 접근할 수 있습니다.

## Skill 위치 (Skill Locations)

Skills는 `settingSources`/`setting_sources` 구성에 따라 파일 시스템 디렉터리에서 로드됩니다:

* **프로젝트 Skills** (`.claude/skills/`): git을 통해 팀과 공유 - `setting_sources`에 `"project"`가 포함된 경우 로드됨
* **사용자 Skills** (`~/.claude/skills/`): 모든 프로젝트에 걸친 개인 Skills - `setting_sources`에 `"user"`가 포함된 경우 로드됨
* **플러그인 Skills**: 설치된 Claude Code 플러그인과 함께 번들로 제공됨

## Skills 생성하기

Skills는 YAML 프론트매터 및 Markdown 콘텐츠가 포함된 `SKILL.md` 파일을 포함하는 디렉터리로 정의됩니다. `description` 필드는 Claude가 Skill을 호출하는 시점을 결정합니다.

**디렉터리 구조 예시**:

```bash theme={null}
.claude/skills/processing-pdfs/
└── SKILL.md
```

SKILL.md 구조, 다중 파일 Skills 및 예시를 포함하여 Skills 생성에 대한 완전한 지침은 다음을 참조하세요:

* [Claude Code의 Agent Skills](/docs/en/skills): 예시가 포함된 전체 가이드
* [Agent Skills 모범 사례](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices): 작성 지침 및 명명 규칙

## 도구 제한 사항 (Tool Restrictions)

<Note>
  SKILL.md의 `allowed-tools` 프론트매터 필드는 Claude Code CLI를 직접 사용할 때만 지원됩니다. **SDK를 통해 Skills를 사용할 때는 적용되지 않습니다.**

  SDK를 사용할 때는 쿼리 구성의 기본 `allowedTools` 옵션을 통해 도구 접근을 제어하세요.
</Note>

SDK 애플리케이션에서 Skills에 대한 도구 접근을 제어하려면 `allowedTools`를 사용하여 특정 도구를 사전 승인하세요. `canUseTool` 콜백이 없으면 목록에 없는 모든 도구가 거부됩니다:

<Note>
  다음 코드 스니펫에서는 첫 번째 예시의 import 문이 이미 포함되어 있다고 가정합니다.
</Note>

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      setting_sources=["user", "project"],  # 파일 시스템에서 Skills 로드
      skills="all",
      allowed_tools=["Read", "Grep", "Glob"],
  )

  async for message in query(prompt="Analyze the codebase structure", options=options):
      print(message)
  ```

  ```typescript TypeScript theme={null}
  for await (const message of query({
    prompt: "Analyze the codebase structure",
    options: {
      settingSources: ["user", "project"], // 파일 시스템에서 Skills 로드
      skills: "all",
      allowedTools: ["Read", "Grep", "Glob"],
      permissionMode: "dontAsk" // allowedTools에 없는 것은 모두 거부
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

## 사용 가능한 Skills 탐색하기

SDK 애플리케이션에서 어떤 Skills를 사용할 수 있는지 확인하려면 Claude에게 질문하기만 하면 됩니다:

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      setting_sources=["user", "project"],  # 파일 시스템에서 Skills 로드
      skills="all",
  )

  async for message in query(prompt="What Skills are available?", options=options):
      print(message)
  ```

  ```typescript TypeScript theme={null}
  for await (const message of query({
    prompt: "What Skills are available?",
    options: {
      settingSources: ["user", "project"], // 파일 시스템에서 Skills 로드
      skills: "all"
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

Claude는 현재 작업 디렉터리 및 설치된 플러그인을 기반으로 사용 가능한 Skills 목록을 출력합니다.

## Skills 테스트하기

설명과 일치하는 질문을 하여 Skills를 테스트합니다:

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      cwd="/path/to/project",
      setting_sources=["user", "project"],  # 파일 시스템에서 Skills 로드
      skills="all",
      allowed_tools=["Read", "Bash"],
  )

  async for message in query(prompt="Extract text from invoice.pdf", options=options):
      print(message)
  ```

  ```typescript TypeScript theme={null}
  for await (const message of query({
    prompt: "Extract text from invoice.pdf",
    options: {
      cwd: "/path/to/project",
      settingSources: ["user", "project"], // 파일 시스템에서 Skills 로드
      skills: "all",
      allowedTools: ["Read", "Bash"]
    }
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

설명이 요청과 일치하는 경우 Claude가 관련 Skill을 자동으로 호출합니다.

## 문제 해결 (Troubleshooting)

### Skills를 찾을 수 없음

**settingSources 구성 확인**: Skills는 `user` 및 `project` 설정 소스를 통해 탐색됩니다. `settingSources`/`setting_sources`를 명시적으로 설정하고 해당 소스를 생략하면 스킬이 로드되지 않습니다:

<CodeGroup>
  ```python Python theme={null}
  # Skills 로드 안 됨: setting_sources에서 user 및 project 제외됨
  options = ClaudeAgentOptions(setting_sources=[], skills="all")

  # Skills 로드됨: user 및 project 소스 포함됨
  options = ClaudeAgentOptions(
      setting_sources=["user", "project"],
      skills="all",
  )
  ```

  ```typescript TypeScript theme={null}
  // Skills 로드 안 됨: settingSources에서 user 및 project 제외됨
  const options = {
    settingSources: [],
    skills: "all"
  };

  // Skills 로드됨: user 및 project 소스 포함됨
  const options = {
    settingSources: ["user", "project"],
    skills: "all"
  };
  ```
</CodeGroup>

`settingSources`/`setting_sources`에 대한 자세한 내용은 [TypeScript SDK 레퍼런스](/docs/en/agent-sdk/typescript#settingsource) 또는 [Python SDK 레퍼런스](/docs/en/agent-sdk/python#settingsource)를 참조하세요.

**작업 디렉터리 확인**: SDK는 `cwd` 옵션 및 리포지토리 루트까지의 모든 상위 디렉터리에 있는 `.claude/skills/`에서 Skills를 로드합니다. 동일한 리포지토리 내에서 `cwd`가 `.claude/skills/`를 포함하는 디렉터리 또는 그 하위를 가리키고 있는지 확인하세요:

<CodeGroup>
  ```python Python theme={null}
  # cwd가 .claude/skills/를 포함하는 디렉터리를 가리키는지 확인
  options = ClaudeAgentOptions(
      cwd="/path/to/project",  # 여기에 있거나 상위 디렉터리에 있는 .claude/skills/
      setting_sources=["user", "project"],  # 이 소스들에서 스킬 로드
      skills="all",
  )
  ```

  ```typescript TypeScript theme={null}
  // cwd가 .claude/skills/를 포함하는 디렉터리를 가리키는지 확인
  const options = {
    cwd: "/path/to/project", // 여기에 있거나 상위 디렉터리에 있는 .claude/skills/
    settingSources: ["user", "project"], // 이 소스들에서 스킬 로드
    skills: "all"
  };
  ```
</CodeGroup>

전체 패턴은 위의 "SDK에서 Skills 사용하기" 섹션을 참조하세요.

**파일 시스템 위치 검증**:

```bash theme={null}
# 프로젝트 Skills 확인
ls .claude/skills/*/SKILL.md

# 개인 Skills 확인
ls ~/.claude/skills/*/SKILL.md
```

### Skill이 사용되지 않음

**`skills` 옵션 확인**: `skills` 목록을 전달한 경우 스킬 이름이 포함되어 있는지 확인하세요. `[]`를 전달하면 모든 스킬이 비활성화됩니다.

**설명 확인**: 구체적이고 관련 키워드가 포함되어 있는지 확인하세요. 효과적인 설명 작성 지침은 [Agent Skills 모범 사례](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices#writing-effective-descriptions)를 참조하세요.

### 추가 문제 해결

일반적인 Skills 문제 해결(YAML 구문, 디버깅 등)에 대해서는 [Claude Code Skills 문제 해결 섹션](/docs/en/skills#troubleshooting)을 참조하세요.

## 관련 문서

### Skills 가이드

* [Claude Code의 Agent Skills](/docs/en/skills): 생성, 예시 및 문제 해결이 포함된 전체 Skills 가이드
* [Agent Skills 개요](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview): 개념 개요, 이점 및 아키텍처
* [Agent Skills 모범 사례](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices): 효과적인 Skills 작성을 위한 지침
* [Agent Skills 쿡북](https://platform.claude.com/cookbook/skills-notebooks-01-skills-introduction): 예시 Skills 및 템플릿

### SDK 리소스

* [SDK의 Subagents](/docs/en/agent-sdk/subagents): 프로그래밍 방식 옵션이 포함된 유사한 파일 시스템 기반 에이전트
* [SDK의 Slash Commands](/docs/en/agent-sdk/slash-commands): 사용자 호출 명령어
* [SDK 개요](/docs/en/agent-sdk/overview): 일반 SDK 개념
* [TypeScript SDK 레퍼런스](/docs/en/agent-sdk/typescript): 전체 API 문서
* [Python SDK 레퍼런스](/docs/en/agent-sdk/python): 전체 API 문서
