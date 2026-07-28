> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# Claude Agent SDK 마이그레이션 가이드

> Claude Code TypeScript 및 Python SDK를 Claude Agent SDK로 마이그레이션하기 위한 가이드입니다.

## 개요

Claude Code SDK의 명칭이 **Claude Agent SDK**로 변경되었으며 관련 문서도 재구성되었습니다. 이번 변경은 단순한 코딩 작업을 넘어 AI 에이전트를 구축하기 위한 SDK의 광범위한 기능을 반영합니다.

## 변경 사항

| 영역 | 이전 | 변경 후 |
| :------------------------- | :-------------------------- | :------------------------------- |
| **패키지 이름 (TS/JS)** | `@anthropic-ai/claude-code` | `@anthropic-ai/claude-agent-sdk` |
| **Python 패키지** | `claude-code-sdk` | `claude-agent-sdk` |
| **문서 위치** | Claude Code 문서 | API 가이드 → Agent SDK 섹션 |

<Note>
  **문서 변경 사항:** Agent SDK 문서가 Claude Code 문서에서 API 가이드 하위의 전용 [Agent SDK](/docs/en/agent-sdk/overview) 섹션으로 이동했습니다. Claude Code 문서는 이제 CLI 도구 및 자동화 기능에 집중합니다.
</Note>

## 마이그레이션 단계

### TypeScript/JavaScript 프로젝트의 경우

**1. 이전 패키지 제거:**

```bash theme={null}
npm uninstall @anthropic-ai/claude-code
```

**2. 새 패키지 설치:**

```bash theme={null}
npm install @anthropic-ai/claude-agent-sdk
```

**3. 임포트 구문 업데이트:**

모든 임포트를 `@anthropic-ai/claude-code`에서 `@anthropic-ai/claude-agent-sdk`로 변경합니다:

```typescript theme={null}
// 변경 전
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-code";

// 변경 후
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
```

**4. package.json 의존성 업데이트:**

`package.json`에 패키지가 나열되어 있는 경우 업데이트하세요:

변경 전:

```json theme={null}
{
  "dependencies": {
    "@anthropic-ai/claude-code": "^0.0.42"
  }
}
```

변경 후:

```json theme={null}
{
  "dependencies": {
    "@anthropic-ai/claude-agent-sdk": "^0.3.0"
  }
}
```

**5. [하위 호환성을 깨뜨리는 변경 사항(Breaking changes)](#하위-호환성을-깨뜨리는-변경-사항) 검토**

마이그레이션을 완료하기 위해 필요한 코드 변경 사항을 적용하세요.

### Python 프로젝트의 경우

**1. 이전 패키지 제거:**

```bash theme={null}
pip uninstall -y claude-code-sdk
```

이전 패키지가 설치되어 있지 않은 경우 pip가 `WARNING: Skipping claude-code-sdk as it is not installed.`라고 출력할 수 있습니다. 이는 예상된 결과이므로 다음 단계로 진행하시면 됩니다.

**2. 새 패키지 설치:**

```bash theme={null}
pip install claude-agent-sdk
```

**3. 임포트 구문 업데이트:**

모든 임포트를 `claude_code_sdk`에서 `claude_agent_sdk`로 변경합니다:

```python theme={null}
# 변경 전
from claude_code_sdk import query, ClaudeCodeOptions

# 변경 후
from claude_agent_sdk import query, ClaudeAgentOptions
```

**4. 타입 이름 업데이트:**

`ClaudeCodeOptions`를 `ClaudeAgentOptions`로 변경합니다:

```python theme={null}
# 변경 전
from claude_code_sdk import query, ClaudeCodeOptions

options = ClaudeCodeOptions(model="claude-opus-4-7")

# 변경 후
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(model="claude-opus-4-7")
```

**5. [하위 호환성을 깨뜨리는 변경 사항(Breaking changes)](#하위-호환성을-깨뜨리는-변경-사항) 검토**

마이그레이션을 완료하기 위해 필요한 코드 변경 사항을 적용하세요.

## 하위 호환성을 깨뜨리는 변경 사항

<Warning>
  격리 및 명시적 구성을 개선하기 위해 Claude Agent SDK v0.1.0은 Claude Code SDK에서 마이그레이션하는 사용자를 위해 하위 호환성을 깨뜨리는 변경 사항(breaking changes)을 도입했습니다. 마이그레이션 전에 이 섹션을 주의 깊게 검토하세요.
</Warning>

### Python: ClaudeCodeOptions의 명칭이 ClaudeAgentOptions로 변경됨

**변경 내용:** Python SDK 타입 `ClaudeCodeOptions`가 `ClaudeAgentOptions`로 변경되었습니다.

**마이그레이션:**

```python theme={null}
# 변경 전 (claude-code-sdk)
from claude_code_sdk import query, ClaudeCodeOptions

options = ClaudeCodeOptions(model="claude-opus-4-7", permission_mode="acceptEdits")

# 변경 후 (claude-agent-sdk)
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(model="claude-opus-4-7", permission_mode="acceptEdits")
```

**변경 이유:** 타입 이름이 이제 "Claude Agent SDK" 브랜딩과 일치하며 SDK 전체의 명명 규칙에서 일관성을 제공합니다.

### 시스템 프롬프트가 더 이상 기본 적용되지 않음

**변경 내용:** SDK가 더 이상 기본적으로 Claude Code의 시스템 프롬프트를 사용하지 않습니다.

**마이그레이션:**

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 변경 전 (v0.0.x) - 기본적으로 Claude Code의 시스템 프롬프트 사용
  const before = query({ prompt: "Hello" });

  // 변경 후 (v0.1.0) - 기본적으로 최소한의 시스템 프롬프트 사용
  // 이전 동작을 얻으려면 Claude Code의 프리셋을 명시적으로 요청하세요:
  const presetResult = query({
    prompt: "Hello",
    options: {
      systemPrompt: { type: "preset", preset: "claude_code" }
    }
  });

  // 또는 커스텀 시스템 프롬프트를 사용하세요:
  const customResult = query({
    prompt: "Hello",
    options: {
      systemPrompt: "You are a helpful coding assistant"
    }
  });
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions
  import asyncio


  async def main():
      # 변경 전 (v0.0.x) - 기본적으로 Claude Code의 시스템 프롬프트 사용
      async for message in query(prompt="Hello"):
          print(message)

      # 변경 후 (v0.1.0) - 기본적으로 최소한의 시스템 프롬프트 사용
      # 이전 동작을 얻으려면 Claude Code의 프리셋을 명시적으로 요청하세요:
      async for message in query(
          prompt="Hello",
          options=ClaudeAgentOptions(
              system_prompt={"type": "preset", "preset": "claude_code"}  # 프리셋 사용
          ),
      ):
          print(message)

      # 또는 커스텀 시스템 프롬프트를 사용하세요:
      async for message in query(
          prompt="Hello",
          options=ClaudeAgentOptions(system_prompt="You are a helpful coding assistant"),
      ):
          print(message)


  asyncio.run(main())
  ```
</CodeGroup>

**변경 이유:** SDK 애플리케이션에 대한 더 나은 제어 및 격리를 제공합니다. 이제 Claude Code CLI 중심의 지침을 상속받지 않고도 커스텀 동작을 가진 에이전트를 구축할 수 있습니다.

### 설정 소스 기본값

이 기본값은 v0.1.0에서 잠시 변경되었다가 원래대로 복구되었으므로 마이그레이션 조치가 필요하지 않습니다.

**현재 동작:** `query()`에서 `settingSources`를 생략하면 CLI와 동일하게 사용자, 프로젝트 및 로컬 파일 시스템 설정을 로드합니다. 여기에는 `~/.claude/settings.json`, `.claude/settings.json`, `.claude/settings.local.json`, CLAUDE.md 파일 및 커스텀 명령이 포함됩니다.

파일 시스템 설정과 격리되어 실행하려면 빈 배열을 전달하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const isolatedResult = query({
    prompt: "Hello",
    options: {
      settingSources: [] // 파일 시스템 설정 로드 안 함
    }
  });

  // 또는 특정 소스만 로드:
  const projectOnlyResult = query({
    prompt: "Hello",
    options: {
      settingSources: ["project"] // 프로젝트 설정만 로드
    }
  });
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions
  import asyncio


  async def main():
      async for message in query(
          prompt="Hello",
          options=ClaudeAgentOptions(setting_sources=[]),  # 파일 시스템 설정 로드 안 함
      ):
          print(message)

      # 또는 특정 소스만 로드:
      async for message in query(
          prompt="Hello",
          options=ClaudeAgentOptions(
              setting_sources=["project"]  # 프로젝트 설정만 로드
          ),
      ):
          print(message)


  asyncio.run(main())
  ```
</CodeGroup>

격리는 로컬 커스텀 설정이 유출되어서는 안 되는 CI/CD 파이프라인, 배포된 애플리케이션, 테스트 환경 및 멀티 테넌트 시스템에서 특히 중요합니다.

<Note>
  SDK v0.1.0은 잠시 설정을 로드하지 않는 것을 기본값으로 삼았으나, 후속 릴리스에서 복구되었습니다. Python SDK 0.1.59 이하 버전은 빈 목록을 옵션 생략과 동일하게 처리했으므로 `setting_sources=[]`에 의존하기 전에 업그레이드하세요. `settingSources`가 `[]`일 때도 읽히는 입력 사항은 [settingSources가 제어하지 않는 항목](/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control)을 참조하세요.
</Note>

## 이름이 변경된 이유

Claude Code SDK는 원래 코딩 작업을 위해 설계되었으나, 모든 유형의 AI 에이전트를 구축하기 위한 강력한 프레임워크로 발전했습니다. "Claude Agent SDK"라는 새 명칭은 다음과 같은 기능을 더 잘 반영합니다:

* 비즈니스 에이전트 구축 (법률 보조, 재무 어드바이저, 고객 지원)
* 전문 코딩 에이전트 생성 (SRE 봇, 보안 검토자, 코드 리뷰 에이전트)
* 도구 사용, MCP 연동 등을 통해 모든 도메인을 위한 커스텀 에이전트 개발

## 도움말 받기

마이그레이션 중 문제가 발생하는 경우:

**TypeScript/JavaScript의 경우:**

1. 모든 임포트가 `@anthropic-ai/claude-agent-sdk`를 사용하도록 업데이트되었는지 확인하세요
2. package.json에 새 패키지 이름이 반영되었는지 확인하세요
3. `npm install`을 실행하여 의존성이 업데이트되었는지 확인하세요

**Python의 경우:**

1. 모든 임포트가 `claude_agent_sdk`를 사용하도록 업데이트되었는지 확인하세요
2. requirements.txt 또는 pyproject.toml에 새 패키지 이름이 반영되었는지 확인하세요
3. `pip install claude-agent-sdk`를 실행하여 패키지가 설치되었는지 확인하세요

## 다음 단계

* 사용 가능한 기능을 알아보려면 [Agent SDK 개요](/docs/en/agent-sdk/overview)를 둘러보세요
* 상세 API 문서는 [TypeScript SDK 참조](/docs/en/agent-sdk/typescript)를 확인하세요
* Python 전용 문서는 [Python SDK 참조](/docs/en/agent-sdk/python)를 확인하세요
* [커스텀 도구](/docs/en/agent-sdk/custom-tools) 및 [MCP 연동](/docs/en/agent-sdk/mcp)에 대해 알아보세요
