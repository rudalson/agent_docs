> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 플러그인 생성하기

> 스킬, 에이전트, 훅, MCP 서버로 Claude Code를 확장할 수 있는 커스텀 플러그인을 생성합니다.

플러그인을 사용하면 프로젝트 및 팀 간에 공유할 수 있는 커스텀 기능으로 Claude Code를 확장할 수 있습니다. 이 가이드는 스킬, 에이전트, 훅, MCP 서버가 포함된 플러그인을 직접 제작하는 방법을 다룹니다.

기존 플러그인을 설치하려는 경우 [플러그인 탐색 및 설치](/docs/en/discover-plugins)를 참조하세요. 완전한 기술 명세는 [플러그인 참조 문서](/docs/en/plugins-reference)를 참조하세요.

## 플러그인 vs 독립형(Standalone) 구성 사용 시점

Claude Code는 커스텀 스킬, 에이전트, 훅을 추가하는 두 가지 방식을 지원합니다:

| 접근 방식                                                                                                       | 스킬 이름            | 적합한 대상                                                                                     |
| :-------------------------------------------------------------------------------------------------------------- | :------------------- | :---------------------------------------------------------------------------------------------- |
| **독립형 (Standalone)** (`.claude/` 디렉토리)                                                                   | `/hello`             | 개인 워크플로, 프로젝트 전용 커스텀 설정, 빠른 실험                                             |
| **플러그인 (Plugins)** (스킬, 에이전트, 훅 또는 `.claude-plugin/plugin.json` 매니페스트를 포함하는 독립 디렉토리) | `/plugin-name:hello` | 팀원과의 공유, 커뮤니티 배포, 버전 관리 릴리스, 여러 프로젝트 간 재사용                         |

**다음의 경우 독립형 구성을 사용하세요**:

* 단일 프로젝트에 대해 Claude Code를 커스텀할 때
* 구성이 개인적이고 공유할 필요가 없을 때
* 패키징하기 전에 스킬이나 훅을 실험해 보는 중일 때
* `/hello`나 `/deploy`와 같이 짧은 스킬 이름을 사용하고 싶을 때

**다음의 경우 플러그인을 사용하세요**:

* 팀이나 커뮤니티와 기능을 공유하고 싶을 때
* 여러 프로젝트에 걸쳐 동일한 스킬/에이전트가 필요할 때
* 확장 기능에 대해 버전 제어 및 손쉬운 업데이트가 필요할 때
* 마켓플레이스를 통해 배포할 때
* `/my-plugin:hello`와 같은 네임스페이스 스킬 이름이 괜찮을 때 (네임스페이스 지정은 플러그인 간 이름 충돌을 방지함)

<Tip>
  빠른 반복 작업을 위해 `.claude/`의 독립형 구성으로 시작한 후 공유할 준비가 되었을 때 [플러그인으로 전환](#convert-existing-configurations-to-plugins)하세요.
</Tip>

## 빠른 시작

이 빠른 시작 가이드는 커스텀 스킬이 포함된 플러그인을 제작하는 과정을 안내합니다. 매니페스트(플러그인을 정의하는 구성 파일)를 생성하고, 스킬을 추가하며, `--plugin-dir` 플래그를 사용하여 로컬에서 테스트하게 됩니다.

### 사전 요구사항

* Claude Code가 [설치 및 인증](/docs/en/quickstart#step-1-install-claude-code)되어 있어야 함

<Note>
  `/plugin` 명령이 보이지 않으면 Claude Code를 최신 버전으로 업데이트하세요. 업그레이드 안내는 [문제 해결](/docs/en/troubleshooting)을 참조하세요.
</Note>

### 첫 번째 플러그인 제작하기

<Steps>
  <Step title="플러그인 디렉토리 생성">
    모든 플러그인은 스킬, 에이전트, 훅, 그리고 선택적으로 `.claude-plugin/plugin.json` 매니페스트를 담고 있는 자체 디렉토리에 위치합니다. 이 빠른 시작에서는 테스트 단계에서 `--plugin-dir`로 Claude Code에 해당 디렉토리를 지정할 것이므로 위치는 어디든 상관없습니다. 임시 폴더나 프로젝트 디렉토리 등 편한 위치에 생성하세요:

    ```bash theme={null}
    mkdir my-first-plugin
    ```

    남은 단계는 상위 디렉토리에서 실행하며 이에 대한 상대 경로 `my-first-plugin/...`을 참조합니다.
  </Step>

  <Step title="플러그인 매니페스트 생성">
    `.claude-plugin/plugin.json`에 위치한 매니페스트 파일은 플러그인의 정체성(이름, 설명, 버전)을 정의합니다. Claude Code는 이 메타데이터를 활용하여 플러그인 관리자에 플러그인을 표시합니다.

    플러그인 폴더 안에 `.claude-plugin` 디렉토리를 생성하세요:

    ```bash theme={null}
    mkdir my-first-plugin/.claude-plugin
    ```

    그런 다음 아래 내용으로 `my-first-plugin/.claude-plugin/plugin.json`을 생성하세요:

    ```json my-first-plugin/.claude-plugin/plugin.json theme={null}
    {
      "name": "my-first-plugin",
      "description": "A greeting plugin to learn the basics",
      "version": "1.0.0",
      "author": {
        "name": "Your Name"
      }
    }
    ```

    | 필드          | 목적                                                                                                                                                                                                                                                          |
    | :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | `name`        | 고유 식별자 및 스킬 네임스페이스. 스킬 앞에 이 이름이 접두사로 붙습니다 (예: `/my-first-plugin:hello`).                                                                                                                                                       |
    | `description` | 플러그인을 탐색하거나 설치할 때 플러그인 관리자에 표시됩니다.                                                                                                                                                                                                 |
    | `version`     | 선택 사항. 설정 시 사용자는 이 필드를 올려 올릴 때만 업데이트를 받습니다. 생략되고 플러그인이 git을 통해 배포되는 경우 커밋 SHA가 사용되며 모든 커밋이 새 버전으로 간주됩니다. [버전 관리](/docs/en/plugins-reference#version-management)를 참조하세요. |
    | `author`      | 선택 사항. 출처 표시에 유용합니다.                                                                                                                                                                                                                            |

    `homepage`, `repository`, `license`와 같은 추가 필드에 대해서는 [전체 매니페스트 스키마](/docs/en/plugins-reference#plugin-manifest-schema)를 참조하세요.
  </Step>

  <Step title="스킬 추가">
    스킬은 `skills/` 디렉토리에 위치합니다. 각 스킬은 `SKILL.md` 파일을 포함하는 폴더입니다. 폴더 이름에 플러그인의 네임스페이스가 접두사로 붙어 스킬 이름이 됩니다 (`my-first-plugin`이라는 이름의 플러그인 내 `hello/`는 `/my-first-plugin:hello`를 생성함).

    플러그인 폴더 안에 스킬 디렉토리를 생성하세요:

    ```bash theme={null}
    mkdir -p my-first-plugin/skills/hello
    ```

    그런 다음 아래 내용으로 `my-first-plugin/skills/hello/SKILL.md`를 생성하세요:

    ```markdown my-first-plugin/skills/hello/SKILL.md theme={null}
    ---
    description: Greet the user with a friendly message
    disable-model-invocation: true
    ---

    Greet the user warmly and ask how you can help them today.
    ```
  </Step>

  <Step title="플러그인 테스트">
    `--plugin-dir` 플래그와 함께 Claude Code를 실행하여 플러그인을 로드하세요:

    ```bash theme={null}
    claude --plugin-dir ./my-first-plugin
    ```

    Claude Code가 시작되면 새 스킬을 시도해 보세요:

    ```shell theme={null}
    /my-first-plugin:hello
    ```

    Claude가 인사말로 응답하는 것을 보게 될 것입니다. `/help`를 실행하여 플러그인 네임스페이스 아래에 나열된 스킬을 확인하세요.

    <Note>
      **왜 네임스페이스를 지정하나요?** 여러 플러그인이 동일한 이름의 스킬을 가지고 있을 때 충돌을 방지하기 위해 플러그인 스킬은 항상 네임스페이스가 지정됩니다 (예: `/my-first-plugin:hello`).

      네임스페이스 접두사를 변경하려면 `plugin.json` 내부의 `name` 필드를 업데이트하세요.
    </Note>
  </Step>

  <Step title="스킬 인수 추가">
    사용자 입력을 수용하여 스킬을 동적으로 만드세요. `$ARGUMENTS` 플레이스홀더는 사용자가 스킬 이름 뒤에 제공하는 모든 텍스트를 캡처합니다.

    `SKILL.md` 파일을 업데이트하세요:

    ```markdown my-first-plugin/skills/hello/SKILL.md theme={null}
    ---
    description: Greet the user with a personalized message
    ---

    # Hello Skill

    Greet the user named "$ARGUMENTS" warmly and ask how you can help them today. Make the greeting personal and encouraging.
    ```

    `/reload-plugins`를 실행하여 변경 사항을 적용한 후 본인의 이름과 함께 스킬을 시도해 보세요:

    ```shell theme={null}
    /my-first-plugin:hello Alex
    ```

    Claude가 이름으로 인사를 건넬 것입니다. 스킬에 인수를 전달하는 자세한 내용은 [스킬](/docs/en/skills#pass-arguments-to-skills)을 참조하세요.
  </Step>
</Steps>

이러한 핵심 구성 요소로 플러그인을 성공적으로 작성하고 테스트했습니다:

* **플러그인 매니페스트** (`.claude-plugin/plugin.json`): 플러그인의 메타데이터 설명
* **스킬 디렉토리** (`skills/`): 커스텀 스킬 포함
* **스킬 인수** (`$ARGUMENTS`): 동적 동작을 위해 사용자 입력 캡처

<Tip>
  `--plugin-dir` 플래그는 개발 및 테스트에 유용합니다. 다른 사람들과 플러그인을 공유할 준비가 되었다면 [플러그인 마켓플레이스 생성 및 배포](/docs/en/plugin-marketplaces)를 참조하세요.
</Tip>

## skills 디렉토리에서 플러그인 개발하기

매 실행 시 `--plugin-dir`을 전달하는 대신, skills 디렉토리에 플러그인을 보관하여 Claude Code가 이를 자동으로 로드하도록 할 수 있습니다. `claude plugin init` 명령으로 이를 스캐폴딩할 수 있습니다:

```bash theme={null}
claude plugin init my-tool
```

이렇게 하면 `.claude-plugin/plugin.json` 매니페스트와 스타터 `SKILL.md`가 포함된 `~/.claude/skills/my-tool/`이 생성됩니다. 다음 세션에서 마켓플레이스나 설치 단계 없이 `my-tool@skills-dir`로 로드됩니다.

자동 로드 규칙, 개인 대 프로젝트 스코프, 작업 공간 신뢰 요구사항, 업데이트 및 제거 방법은 [Skills 디렉토리 플러그인](/docs/en/plugins-reference#skills-directory-plugins)을 참조하세요.

## 플러그인 구조 개요

스킬이 포함된 플러그인을 작성해 보았지만 플러그인에는 커스텀 에이전트, 훅, MCP 서버, LSP 서버, 백그라운드 모니터 등 훨씬 많은 요소를 포함할 수 있습니다.

<Warning>
  **흔히 하는 실수**: `commands/`, `agents/`, `skills/`, `hooks/`를 `.claude-plugin/` 디렉토리 안에 두지 마세요. `.claude-plugin/` 안에는 `plugin.json`만 들어갑니다. 다른 모든 디렉토리는 플러그인 루트 수준에 위치해야 합니다.

  플러그인 루트는 개별 플러그인 자체의 디렉토리입니다: `--plugin-dir`에 전달하거나 `.claude-plugin/plugin.json`을 포함하는 디렉토리입니다. 절대 `~/.claude/`가 아닙니다. 예를 들어 Claude Code는 `~/.claude/.mcp.json`에 위치한 `.mcp.json`은 읽지 않습니다.
</Warning>

| 디렉토리          | 위치        | 목적                                                                        |
| :---------------- | :---------- | :-------------------------------------------------------------------------- |
| `.claude-plugin/` | 플러그인 루트| `plugin.json` 매니페스트 포함 (구성 요소가 기본 위치를 사용하는 경우 선택) |
| `skills/`         | 플러그인 루트| `<name>/SKILL.md` 디렉토리 형태의 스킬                                       |
| `commands/`       | 플러그인 루트| 단일 마크다운 파일 형태의 스킬. 새 플러그인에는 `skills/` 사용 권장        |
| `agents/`         | 플러그인 루트| 커스텀 에이전트 정의                                                        |
| `hooks/`          | 플러그인 루트| `hooks.json` 내의 이벤트 핸들러                                             |
| `.mcp.json`       | 플러그인 루트| MCP 서버 구성                                                               |
| `.lsp.json`       | 플러그인 루트| 코드 인텔리전스를 위한 LSP 서버 구성                                        |
| `monitors/`       | 플러그인 루트| `monitors.json` 내의 백그라운드 모니터 구성                                 |
| `bin/`            | 플러그인 루트| 플러그인이 활성화되어 있는 동안 Bash 도구의 `PATH`에 추가되는 실행 파일들   |
| `settings.json`   | 플러그인 루트| 플러그인이 활성화될 때 적용되는 기본 [설정](/docs/en/settings)               |

정확히 하나의 스킬만 제공하는 플러그인은 `skills/` 디렉토리를 생성하는 대신 `SKILL.md`를 플러그인 루트에 직접 위치시킬 수 있습니다. Claude Code는 이를 단일 스킬로 로드하며 호출 이름을 위해 프론트매터의 `name` 필드를 사용합니다. 나중에 둘 이상의 스킬로 확장될 수 있는 플러그인에는 `skills/` 레이아웃을 사용하세요.

<Note>
  **다음 단계**: 더 많은 기능을 추가할 준비가 되었나요? 에이전트, 훅, MCP 서버, LSP 서버를 추가하려면 [복잡한 플러그인 개발하기](#develop-more-complex-plugins)로 건너뛰세요. 모든 플러그인 구성 요소에 대한 완전한 기술 명세는 [플러그인 참조 문서](/docs/en/plugins-reference)를 참조하세요.
</Note>

## 복잡한 플러그인 개발하기

기본 플러그인에 익숙해지면 더 정교한 확장 기능을 제작할 수 있습니다.

### 플러그인에 스킬 추가하기

플러그인에 [에이전트 스킬](/docs/en/skills)을 포함하여 Claude의 능력을 확장할 수 있습니다. 스킬은 모델에 의해 호출됩니다: Claude가 태스크 컨텍스트에 따라 스킬을 자동으로 사용합니다.

`SKILL.md` 파일을 담고 있는 스킬 폴더와 함께 플러그인 루트에 `skills/` 디렉토리를 추가하세요:

```text theme={null}
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── code-review/
        └── SKILL.md
```

각 `SKILL.md`는 YAML 프론트매터와 지침을 포함합니다. Claude가 스킬을 언제 사용할지 알 수 있도록 `description`을 포함시키세요:

```yaml theme={null}
---
description: Reviews code for best practices and potential issues. Use when reviewing code, checking PRs, or analyzing code quality.
---

When reviewing code, check for:
1. Code organization and structure
2. Error handling
3. Security concerns
4. Test coverage
```

플러그인을 설치한 후 `/reload-plugins`를 실행하여 스킬을 로드하세요. 단계적 공개 및 도구 제한을 포함하여 스킬 작성에 관한 전체 안내는 [에이전트 스킬](/docs/en/skills)을 참조하세요.

### 플러그인에 LSP 서버 추가하기

<Tip>
  TypeScript, Python, Rust와 같은 일반적인 언어의 경우 공식 마켓플레이스의 빌드된 LSP 플러그인을 설치하세요. 아직 포함되어 있지 않은 언어에 대한 지원이 필요할 때만 커스텀 LSP 플러그인을 제작하세요.
</Tip>

LSP (Language Server Protocol) 플러그인은 Claude에게 실시간 코드 인텔리전스를 제공합니다. 공식 LSP 플러그인이 없는 언어를 지원해야 하는 경우 플러그인에 `.lsp.json` 파일을 추가하여 직접 제작할 수 있습니다:

```json .lsp.json theme={null}
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

플러그인을 설치하는 사용자의 머신에 언어 서버 바이너리가 설치되어 있어야 합니다.

전체 LSP 구성 옵션은 [LSP 서버](/docs/en/plugins-reference#lsp-servers)를 참조하세요.

### 플러그인에 백그라운드 모니터 추가하기

백그라운드 모니터(background monitors)를 사용하면 플러그인이 백그라운드에서 로그, 파일, 외부 상태를 모니터링하고 이벤트가 발생할 때 Claude에게 알릴 수 있습니다. 플러그인이 활성화되면 Claude Code가 각 모니터를 자동으로 시작하므로 사용자가 Claude에게 모니터링을 시작하라고 지시할 필요가 없습니다.

플러그인 루트에 모니터 항목 배열이 포함된 `monitors/monitors.json` 파일을 추가하세요:

```json monitors/monitors.json theme={null}
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log"
  }
]
```

`command`가 낸 stdout의 각 줄은 세션 중에 알림 형태로 Claude에게 전달됩니다. `when` 트리거 및 변수 치환을 포함한 전체 스키마는 [모니터](/docs/en/plugins-reference#monitors)를 참조하세요.

### 플러그인과 함께 기본 설정 제공하기

플러그인은 플러그인이 활성화될 때 기본 구성을 적용하기 위해 플러그인 루트에 `settings.json` 파일을 포함할 수 있습니다. 현재는 `agent` 및 `subagentStatusLine` 키만 지원됩니다.

`agent`를 설정하면 플러그인의 [커스텀 에이전트](/docs/en/sub-agents) 중 하나가 메인 스레드로 활성화되어 시스템 프롬프트, 도구 제한, 모델을 적용하게 됩니다. 이를 통해 플러그인이 활성화되었을 때 Claude Code가 기본적으로 동작하는 방식을 변경할 수 있습니다.

```json settings.json theme={null}
{
  "agent": "security-reviewer"
}
```

이 예시는 플러그인의 `agents/` 디렉토리에 정의된 `security-reviewer` 에이전트를 활성화합니다. `settings.json`에 기재된 설정은 `plugin.json`에 선언된 `settings`보다 우선 적용됩니다. 알 수 없는 키는 조용히 무시됩니다.

### 복잡한 플러그인 조직화하기

많은 구성 요소를 포함하는 플러그인의 경우 기능별로 디렉토리 구조를 구성하세요. 전체 디렉토리 레이아웃 및 구조화 패턴은 [플러그인 디렉토리 구조](/docs/en/plugins-reference#plugin-directory-structure)를 참조하세요.

### 플러그인 로컬 테스트하기

개발 중에 플러그인을 테스트하려면 `--plugin-dir` 플래그를 사용하세요. 이를 통해 설치 과정 없이 플러그인을 직접 로드할 수 있습니다.

```bash theme={null}
claude --plugin-dir ./my-plugin
```

이 플래그는 플러그인 디렉토리의 `.zip` 압축 파일도 수용하며, Claude Code v2.1.128 이상이 필요합니다.

```bash theme={null}
claude --plugin-dir ./my-plugin.zip
```

`--plugin-dir`로 지정된 플러그인이 이미 설치된 마켓플레이스 플러그인과 이름이 같은 경우, 로컬 사본이 해당 세션에 대해 우선 적용됩니다. 이를 통해 이미 설치된 플러그인을 먼저 제거하지 않고도 변경 사항을 테스트할 수 있습니다. 관리형 설정이 강제로 활성화하거나 강제로 비활성화한 플러그인은 예외입니다: `--plugin-dir`이 이들을 재정의할 수는 없습니다.

플러그인을 변경할 때 세션을 다시 시작하지 않고 업데이트 사항을 적용하려면 `/reload-plugins`를 실행하세요. 이렇게 하면 플러그인, 스킬, 에이전트, 훅, 플러그인 MCP 서버, 플러그인 LSP 서버가 다시 로드됩니다. 플러그인 구성 요소 테스트:

* `/plugin-name:skill-name`으로 스킬을 시도해 보기
* Custom Agents 아래의 `/context`에 에이전트가 나타나는지 확인하거나 네임스페이스 이름으로 @-멘션해 보기
* 훅이 예상대로 작동하는지 검증하기

<Tip>
  플래그를 여러 번 지정하여 한 번에 여러 플러그인을 로드할 수 있습니다:

  ```bash theme={null}
  claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
  ```
</Tip>

CI 빌드 아티팩트처럼 이미 `.zip`으로 패키징되어 URL에 호스팅되어 있는 플러그인을 테스트하려면 대신 `--plugin-url`을 사용하세요. Claude Code가 시작 시 압축 파일을 가져와 해당 세션 동안에만 로드합니다. 가져오기가 실패하거나 압축 파일이 유효하지 않은 경우 Claude Code가 플러그인 로드 오류를 보고하고 해당 플러그인 없이 시작합니다. 다른 플러그인 출처와 동일한 [신뢰 고려사항](/docs/en/discover-plugins#security)이 적용됩니다: 이 플래그에는 제어하거나 신뢰하는 압축 파일만을 지정하세요.

여러 플러그인을 로드하려면 각 URL마다 플래그를 반복하세요:

```bash theme={null}
claude --plugin-url https://example.com/my-plugin.zip --plugin-url https://example.com/other.zip
```

또는 따옴표로 감싸진 하나의 인수로 공백으로 구분된 URL들을 전달하세요:

```bash theme={null}
claude --plugin-url "https://example.com/my-plugin.zip https://example.com/other.zip"
```

### 플러그인 문제 디버깅

플러그인이 예상대로 작동하지 않는 경우:

1. **구조 확인**: 디렉토리가 `.claude-plugin/` 내부가 아닌 플러그인 루트에 위치해 있는지 확인
2. **구성 요소 개별 테스트**: 각 스킬, 에이전트, 훅을 따로 점검
3. **검증 및 디버깅 도구 사용**: CLI 명령 및 문제 해결 기법은 [디버깅 및 개발 도구](/docs/en/plugins-reference#debugging-and-development-tools) 참조

### 플러그인 공유하기

플러그인을 공유할 준비가 되었을 때:

1. **문서 추가**: 설치 및 사용 지침이 담긴 `README.md` 포함
2. **버전 관리 전략 선택**: 명시적인 `version`을 설정할지 git 커밋 SHA에 의존할지 결정. [버전 관리](/docs/en/plugins-reference#version-management) 참조
3. **마켓플레이스 생성 또는 사용**: 설치를 위해 [플러그인 마켓플레이스](/docs/en/plugin-marketplaces)를 통해 배포
4. **타인과 함께 테스트**: 더 넓은 배포 전 팀원들이 플러그인을 테스트하도록 지시

플러그인이 마켓플레이스에 올라가면 [플러그인 탐색 및 설치](/docs/en/discover-plugins)의 지침을 사용하여 다른 사람들이 이를 설치할 수 있습니다. 팀 내부로 플러그인을 한정하고 싶다면 [비공개 저장소](/docs/en/plugin-marketplaces#private-repositories)에 마켓플레이스를 호스팅하세요.

### 커뮤니티 마켓플레이스에 플러그인 제출하기

Anthropic은 Claude Code 플러그인을 위한 두 개의 공개 마켓플레이스를 유지 관리합니다:

* **`claude-plugins-official`**: Anthropic이 유지 관리하는 엄선된 플러그인 세트. 대화형으로 Claude Code를 처음 시작할 때 자동으로 등록됩니다. 그 첫 실행 전에 구동되는 비대화형 스크립트는 `claude plugin marketplace add anthropics/claude-plugins-official`로 이를 명시적으로 추가해야 합니다.
* **`claude-community`**: 검토 후 서드파티 제출물이 안착하는 공개 커뮤니티 마켓플레이스. 사용자는 `/plugin marketplace add anthropics/claude-plugins-community`로 이를 추가하고 `@claude-community` 형태로 설치합니다.

커뮤니티 마켓플레이스 검토를 위해 플러그인을 제출하려면 앱 내 서식 중 하나를 사용하세요:

* **claude.ai**: [claude.ai/admin-settings/directory/submissions/plugins/new](https://claude.ai/admin-settings/directory/submissions/plugins/new)
* **Console**: [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)

claude.ai 서식은 Team 또는 Enterprise 조직 및 디렉토리 관리 접근 권한이 필요하며; 조직 소유자(Owner)는 기본적으로 이 접근 권한을 가집니다. Team이나 Enterprise 조직 소속이 아닌 개인 작가는 대신 Console 서식을 사용할 수 있습니다.

제출하기 전에 로컬에서 `claude plugin validate`를 실행하세요. 검토 파이프라인은 자동화된 안전 스크리닝과 함께 모든 제출물에 대해 동일한 검사를 수행합니다.

승인된 플러그인은 [`anthropics/claude-plugins-community`](https://github.com/anthropics/claude-plugins-community) 카탈로그의 특정 커밋 SHA에 고정되며, 사용자가 새 커밋을 저장소에 푸시함에 따라 CI가 고정된 값을 자동으로 올립니다. 공개 카탈로그는 검토 파이프라인으로부터 매일 밤 동기화되므로 승인과 `marketplace.json`에 플러그인이 표시되는 시점 사이에 지연이 있을 수 있습니다. 플러그인을 아직 설치할 수 있는지 확인하려면 [커뮤니티 카탈로그](https://github.com/anthropics/claude-plugins-community/blob/main/.claude-plugin/marketplace.json)에서 이름을 검색하세요.

공식 마켓플레이스인 `claude-plugins-official`은 별도로 큐레이팅됩니다. Anthropic은 재량에 따라 포함할 플러그인을 결정합니다. 신청 절차가 없으며 제출 서식이 공식 마켓플레이스에 플러그인을 추가하지는 않습니다.

Anthropic이 공식 마켓플레이스에 귀하의 플러그인을 나열하면 사용자의 CLI가 Claude Code 사용자에게 플러그인 설치를 제안할 수 있습니다. [CLI에서 플러그인 추천하기](/docs/en/plugin-hints)를 참조하세요.

<Note>
  전체 기술 명세, 디버깅 기법, 배포 전략에 대해서는 [플러그인 참조 문서](/docs/en/plugins-reference)를 참조하세요.
</Note>

## 기존 구성을 플러그인으로 전환하기

`.claude/` 디렉토리에 이미 스킬이나 훅이 있는 경우, 더 쉬운 공유 및 배포를 위해 이들을 플러그인으로 전환할 수 있습니다.

### 마이그레이션 단계

<Steps>
  <Step title="플러그인 구조 생성">
    기존 `.claude/` 폴더 옆의 프로젝트 루트에 새 플러그인 디렉토리를 생성하여 다음 단계의 상대 `cp` 경로가 잘 동작하도록 합니다:

    ```bash theme={null}
    mkdir -p my-plugin/.claude-plugin
    ```

    `my-plugin/.claude-plugin/plugin.json` 위치에 매니페스트 파일을 생성합니다:

    ```json my-plugin/.claude-plugin/plugin.json theme={null}
    {
      "name": "my-plugin",
      "description": "Migrated from standalone configuration",
      "version": "1.0.0"
    }
    ```
  </Step>

  <Step title="기존 파일 복사">
    기존 구성을 플러그인 디렉토리로 복사합니다:

    ```bash theme={null}
    # 명령 복사
    cp -r .claude/commands my-plugin/

    # 에이전트 복사 (있는 경우)
    cp -r .claude/agents my-plugin/

    # 스킬 복사 (있는 경우)
    cp -r .claude/skills my-plugin/
    ```
  </Step>

  <Step title="훅 마이그레이션">
    설정에 훅이 있는 경우 hooks 디렉토리를 생성합니다:

    ```bash theme={null}
    mkdir my-plugin/hooks
    ```

    훅 구성을 담은 `my-plugin/hooks/hooks.json`을 생성합니다. 형식이 동일하므로 `.claude/settings.json` 또는 `settings.local.json`에서 `hooks` 객체를 복사해 오세요. 명령은 stdin을 통해 훅 입력을 JSON으로 받으므로 `jq`를 사용하여 파일 경로를 추출합니다:

    ```json my-plugin/hooks/hooks.json theme={null}
    {
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Write|Edit",
            "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }]
          }
        ]
      }
    }
    ```
  </Step>

  <Step title="마이그레이션된 플러그인 테스트">
    모든 것이 잘 작동하는지 확인하기 위해 플러그인을 로드합니다:

    ```bash theme={null}
    claude --plugin-dir ./my-plugin
    ```

    각 구성 요소 테스트: 명령을 구동하고, `/context` 아래에 에이전트가 표시되는지 확인하고, 훅이 올바르게 트리거되는지 검증합니다.
  </Step>
</Steps>

### 마이그레이션 시 변경되는 사항

| 독립형 (`.claude/`)          | 플러그인                        |
| :--------------------------- | :------------------------------ |
| 단일 프로젝트에서만 사용 가능| 마켓플레이스를 통해 공유 가능   |
| `.claude/commands/` 파일들   | `plugin-name/commands/` 파일들  |
| `settings.json` 내의 훅      | `hooks/hooks.json` 내의 훅      |
| 공유하려면 수동 복사 필요    | `/plugin install`로 설치        |

<Note>
  마이그레이션 후에는 중복을 방지하기 위해 `.claude/`에서 원본 파일들을 제거하세요. 프로젝트 및 사용자 `.claude/agents/` 정의가 동일한 이름의 플러그인 에이전트를 재정의하므로 원본이 제거되어야만 플러그인 버전이 적용됩니다. 플러그인 스킬은 `/plugin-name:skill-name` 형태로 네임스페이스가 지정되므로 하나가 다른 하나를 덮어쓰지 않고 원본 `/skill-name`과 플러그인 복사본이 둘 다 남아있게 됩니다.
</Note>

## 다음 단계

이제 Claude Code의 플러그인 시스템을 이해했으므로 다양한 목표에 맞춘 추천 경로를 제시합니다:

### 플러그인 사용자용

* [플러그인 탐색 및 설치](/docs/en/discover-plugins): 마켓플레이스 탐색 및 플러그인 설치
* [팀 마켓플레이스 구성](/docs/en/discover-plugins#configure-team-marketplaces): 팀을 위한 저장소 수준의 플러그인 설정

### 플러그인 개발자용

* [마켓플레이스 생성 및 배포](/docs/en/plugin-marketplaces): 플러그인 패키징 및 공유
* [플러그인 참조 문서](/docs/en/plugins-reference): 완전한 기술 명세
* 특정 플러그인 구성 요소에 대해 더 깊이 알아보기:
  * [스킬](/docs/en/skills): 스킬 개발 세부사항
  * [Subagents](/docs/en/sub-agents): 에이전트 구성 및 기능
  * [훅](/docs/en/hooks): 이벤트 처리 및 자동화
  * [MCP](/docs/en/mcp): 외부 도구 통합
