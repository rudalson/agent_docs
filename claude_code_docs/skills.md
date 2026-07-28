> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 스킬로 Claude 기능 확장하기

> Claude Code에서 Claude의 기능을 확장하기 위해 스킬을 생성, 관리 및 공유하세요. 사용자 지정 명령과 번들 스킬이 포함됩니다.

스킬은 Claude가 수행할 수 있는 작업을 확장합니다. 지침이 포함된 `SKILL.md` 파일을 생성하면 Claude가 이를 툴킷에 추가합니다. Claude는 관련이 있을 때 스킬을 사용하거나 사용자가 `/skill-name`으로 직접 호출할 수 있습니다.

동일한 지침, 체크리스트 또는 여러 단계의 절차를 대화창에 계속 붙여넣거나 CLAUDE.md의 한 섹션이 사실(fact)보다는 절차로 커졌을 때 스킬을 생성하세요. CLAUDE.md 콘텐츠와 달리 스킬의 본문은 사용될 때만 로드되므로 필요할 때까지 길다란 참조 자료의 비용이 거의 들지 않습니다.

<Note>
  `/help` 및 `/compact`와 같은 내장 명령과 `/debug` 및 `/code-review`와 같은 번들 스킬은 [명령어 참조](/docs/en/commands)를 참조하세요.

  **사용자 지정 명령이 스킬로 통합되었습니다.** `.claude/commands/deploy.md`에 있는 파일과 `.claude/skills/deploy/SKILL.md`에 있는 스킬은 모두 `/deploy`를 생성하며 동일한 방식으로 작동합니다. 기존의 `.claude/commands/` 파일은 계속 작동합니다. 스킬에는 추가 기능이 포함되어 있습니다: 지원 파일용 디렉터리, [호출 주체를 제어하는 프론트매터](#스킬-호출-주체-제어), 그리고 Claude가 관련이 있을 때 자동으로 로드하는 기능입니다.
</Note>

Claude Code 스킬은 여러 AI 도구에서 작동하는 [Agent Skills](https://agentskills.io) 오픈 표준을 따릅니다. Claude Code는 [호출 제어](#스킬-호출-주체-제어), [서브에이전트 실행](#서브에이전트에서-스킬-실행), [동적 컨텍스트 주입](#동적-컨텍스트-주입)과 같은 추가 기능으로 표준을 확장합니다.

## 번들 스킬

Claude Code에는 `/doctor`, `/code-review`, `/batch`, `/debug`, `/loop`, `/claude-api`와 같은 번들 스킬 세트가 포함되어 있습니다. 번들 스킬은 프롬프트 기반입니다. 즉, Claude에게 상세한 지침을 제공하고 자체 도구를 사용하여 작업을 조율하도록 합니다. 대부분의 내장 명령은 직접 고정된 로직을 실행합니다.

`/` 뒤에 스킬 이름을 입력하여 다른 스킬과 동일한 방식으로 번들 스킬을 호출합니다. Claude는 관련이 있을 때 일부 번들 스킬을 자동으로 호출합니다. {/* min-version: 2.1.215 */}나머지 번들 스킬(`/verify` 및 `/code-review` 포함)은 사용자가 호출할 때만 실행되므로 이러한 장시간 실행 검사가 시간과 토큰을 지출하는 시점을 사용자가 직접 제어할 수 있습니다. v2.1.215 이전에는 Claude가 자체적으로 `/verify` 및 `/code-review`를 실행할 수도 있었습니다.

번들 스킬은 모든 세션에서 사용할 수 있습니다. 이를 끄려면 `/doctor`를 제외한 모든 번들 스킬을 비활성화하는 [`disableBundledSkills`](/docs/en/settings#available-settings) 설정을 사용하세요.

<Note>
  [`/doctor`](/docs/en/commands#all-commands) 설정 점검은 Claude Code v2.1.205 이상에서 `disableBundledSkills`가 켜져 있어도 계속 입력할 수 있습니다. 이를 숨기려면 `DISABLE_DOCTOR_COMMAND` 환경 변수나 `"doctor": "off"`의 [`skillOverrides`](#설정에서-스킬-가시성-재정의) 항목을 설정하세요. v2.1.205 이전에는 `/doctor`가 번들 스킬이 아닌 내장 명령이었습니다.
</Note>

번들 스킬은 [명령어 참조](/docs/en/commands)에서 내장 명령과 함께 나열되어 있으며, 목적(Purpose) 열에 **Skill**로 표시되어 있습니다.

### 앱 실행 및 검증

세 가지 번들 스킬이 함께 작동하여 단순히 테스트를 넘어서 실행 중인 앱에 대해 변경 사항을 시작하고 확인합니다:

| 스킬 | 목적 |
| :--- | :--- |
| `/run` | 변경 사항이 작동하는지 확인하기 위해 앱을 시작하고 구동 |
| `/verify` | 테스트나 타입 검사로 대체하지 않고 코드 변경 사항이 의도대로 작동하는지 확인하기 위해 앱을 빌드하고 실행 |
| `/run-skill-generator` | 프로젝트를 빌드하고 시작하는 방법을 `/run` 및 `/verify`에 교육 |

{/* min-version: 2.1.145 */}세 스킬 모두 Claude Code v2.1.145 이상이 필요합니다. `claude --version` 또는 `/status` 명령으로 버전을 확인하세요.

`/run` 및 `/verify`는 설정 없이 작동합니다. 프로젝트 유형(CLI, 서버, TUI, 브라우저 기반)과 README, `package.json` 또는 `Makefile`에 있는 내용을 통해 시작 방식을 추론합니다. 데이터베이스, env 파일, 그래픽 세션, 다단계 빌드 등 표준 시작을 벗어난 항목이 필요한 프로젝트의 경우 이러한 추론이 불확실해집니다.

`/run-skill-generator`는 대신 레시피를 기록합니다. 깨끗한 환경에서 앱을 실행하고 작동한 항목(설치 명령, env 변수, 시작 스크립트)을 캡처하여 프로젝트별 스킬로 `.claude/skills/run-<name>/`에 커밋합니다. 그 후에는 `/run`, `/verify` 및 리포지토리의 다른 에이전트가 이를 재발견하는 대신 기록된 레시피를 따릅니다. 프로젝트당 한 번 `/run-skill-generator`를 실행하고 빌드 또는 시작 프로세스가 변경되면 다시 실행하세요.

## 시작하기

### 첫 번째 스킬 생성하기

이 예시는 Git 리포지토리에서 커밋되지 않은 변경 사항을 요약하고 위험 요소가 있으면 플래그를 지정하는 스킬을 생성합니다. Claude가 읽기 전에 라이브 diff를 프롬프트로 가져오므로, 응답은 열린 파일에서 Claude가 추측할 수 있는 내용이 아니라 실제 작업 트리에 기반을 둡니다. Claude는 사용자가 변경 사항에 대해 물어볼 때 스킬을 자동으로 로드하거나 사용자가 `/summarize-changes`로 직접 호출할 수 있습니다.

<Steps>
  <Step title="스킬 디렉터리 생성">
    개인 스킬 폴더에 스킬용 디렉터리를 생성합니다. 개인 스킬은 모든 프로젝트에서 사용할 수 있습니다.

    ```bash theme={null}
    mkdir -p ~/.claude/skills/summarize-changes
    ```
  </Step>

  <Step title="SKILL.md 작성">
    모든 스킬에는 두 부분으로 구성된 `SKILL.md` 파일이 필요합니다: Claude에게 스킬을 언제 사용할지 알려주는 `---` 마커 사이의 YAML 프론트매터, 그리고 스킬이 실행될 때 Claude가 따르는 지침이 포함된 Markdown 내용입니다. 디렉터리 이름이 입력하는 명령어가 되고 `description`은 Claude가 스킬을 자동으로 로드할 시점을 결정하는 데 도움을 줍니다.

    이를 `~/.claude/skills/summarize-changes/SKILL.md`에 저장하세요:

    ```yaml theme={null}
    ---
    description: 커밋되지 않은 변경 사항을 요약하고 위험한 요소가 있으면 플래그를 지정합니다. 사용자가 변경된 내용을 묻거나, 커밋 메시지를 원하거나, diff 검토를 요청할 때 사용합니다.
    ---

    ## 현재 변경 사항

    !`git diff HEAD`

    ## 지침

    위의 변경 사항을 2~3개의 글머리 기호로 요약한 다음, 누락된 오류 처리, 하드코딩된 값, 업데이트가 필요한 테스트 등 주의 깊게 살펴봐야 할 위험 요소를 나열하세요. diff가 비어 있으면 커밋되지 않은 변경 사항이 없다고 말하세요.
    ```

    `` !`git diff HEAD` `` 라인은 [동적 컨텍스트 주입](#동적-컨텍스트-주입)을 사용합니다. Claude Code는 Claude가 스킬 내용을 보기 전에 명령을 실행하고 해당 라인을 명령 출력으로 교체하므로 지침이 현재 diff가 이미 인라인된 상태로 도착합니다.
  </Step>

  <Step title="스킬 테스트">
    Git 프로젝트를 열고 파일 하나를 약간 편집한 후 `claude`를 실행하여 Claude Code를 시작하세요. 두 가지 방법으로 스킬을 테스트할 수 있습니다.

    설명과 일치하는 질문을 하여 **Claude가 자동으로 호출하도록 함**:

    ```text theme={null}
    내가 무엇을 변경했지?
    ```

    **또는 스킬 이름으로 직접 호출**:

    ```text theme={null}
    /summarize-changes
    ```

    어느 쪽이든 Claude는 수정 사항에 대한 짧은 요약과 위험 요소를 응답해야 합니다.
  </Step>
</Steps>

### 스킬 저장 위치

스킬을 저장하는 위치에 따라 사용할 수 있는 사람이 결정됩니다:

| 위치 | 경로 | 적용 대상 |
| :--- | :--- | :--- |
| 엔터프라이즈 | [관리형 설정](/docs/en/settings#settings-files) 참조 | 조직의 모든 사용자 |
| 개인 | `~/.claude/skills/<skill-name>/SKILL.md` | 본인의 모든 프로젝트 |
| 프로젝트 | `.claude/skills/<skill-name>/SKILL.md` | 이 프로젝트만 |
| 플러그인 | `<plugin>/skills/<skill-name>/SKILL.md` | 플러그인이 활성화된 곳 |

여러 수준에 걸쳐 동일한 이름의 스킬이 공유되는 경우 엔터프라이즈가 개인을 오버라이드하고 개인이 프로젝트를 오버라이드합니다. 이러한 수준의 스킬은 동일한 이름의 번들 스킬도 오버라이드합니다. 예를 들어 프로젝트의 `.claude/skills/`에 있는 `code-review` 스킬은 번들된 `/code-review`를 대체합니다. 플러그인 스킬은 `plugin-name:skill-name` 네임스페이스를 사용하므로 다른 수준과 충돌하지 않습니다. `.claude/commands/`에 파일이 있는 경우 해당 파일도 동일한 방식으로 작동하지만, 스킬과 명령이 동일한 이름을 공유하는 경우 스킬이 우선합니다.

스킬은 작업 디렉터리 아래의 중첩된 `.claude/skills/` 디렉터리에서도 로드됩니다. Claude가 하위 디렉터리의 파일을 읽거나 편집할 때 해당 하위 디렉터리의 `.claude/skills/`에 있는 스킬을 사용할 수 있게 됩니다. 이를 통해 모노리포 패키지는 세션이 리포지토리 루트에서 시작되었더라도 해당 패키지에서 작업할 때 적용되는 고유한 스킬을 제공할 수 있습니다.

중첩된 스킬이 다른 스킬과 이름을 공유하는 경우 두 스킬 모두 사용할 수 있는 상태로 유지됩니다. 예를 들어 프로젝트 루트에 `deploy` 스킬이 있고 `apps/web/.claude/skills/`에 다른 스킬이 있는 경우:

* 중첩된 스킬은 디렉터리로 한정된 이름인 `apps/web:deploy` 아래에 나타납니다.
* 설명에는 적용되는 디렉터리가 표시됩니다.
* Claude는 작업 중인 파일과 일치하는 변형을 선택합니다.

`/deploy`를 입력하면 프로젝트 루트 스킬이 실행됩니다. 한정된 이름인 `/apps/web:deploy`를 입력하여 중첩된 변형을 명시적으로 실행하세요.

사용자 또는 Claude가 한정되지 않은 이름을 호출하면 프로젝트 루트 스킬이 로드되고, Claude Code는 Claude가 작업 중인 파일이 있는 디렉터리의 모든 변형도 호출하라는 지침과 함께 한정된 변형 목록을 스킬 내용에 추가합니다. 따라서 자격이 한정되지 않은 이름만 호출되더라도 중첩된 스킬은 해당 디렉터리의 작업에 계속 적용됩니다. Claude Code v2.1.203 이상이 필요합니다.

엔터프라이즈, 개인 또는 프로젝트 위치의 `<skill-name>` 항목은 디스크의 다른 위치에 있는 디렉터리에 대한 심볼릭 링크일 수 있습니다. Claude Code는 심볼릭 링크를 따라가서 대상 디렉터리에서 `SKILL.md`를 읽으며, 여러 위치에서 동일한 대상에 도달할 수 있는 경우 Claude Code는 스킬을 한 번만 로드합니다. 플러그인 스킬은 심볼릭 링크를 다르게 처리합니다. [마켓플레이스 내에서 심볼릭 링크로 파일 공유](/docs/en/plugins-reference#share-files-within-a-marketplace-with-symlinks)를 참조하세요.

<Note>
  스킬 폴더에 `.claude-plugin/plugin.json`을 추가하면 `<name>@skills-dir`이라는 [플러그인](/docs/en/plugins-reference#skills-directory-plugins)으로 로드되므로 에이전트, 훅 및 MCP 서버를 연결할 수 있습니다. 프로젝트의 `.claude/skills/`에서는 먼저 워크스페이스 신뢰 대화 상자를 수락해야 합니다.
</Note>

#### 라이브 변경 감지

Claude Code는 파일 변경 사항을 파악하기 위해 스킬 디렉터리를 감시합니다. `~/.claude/skills/`, 프로젝트 `.claude/skills/` 또는 `--add-dir` 디렉터리 내부의 `.claude/skills/` 아래에서 스킬을 추가, 편집 또는 제거하면 재시작 없이 현재 세션 내에서 바로 적용됩니다. 세션이 시작될 때 존재하지 않았던 최상위 스킬 디렉터리를 생성하는 경우 새 디렉터리를 감시할 수 있도록 Claude Code를 재시작해야 합니다.

<Note>
  라이브 변경 감지는 `SKILL.md` 텍스트에만 적용됩니다. [플러그인](/docs/en/plugins-reference#skills-directory-plugins)이기도 한 스킬 폴더의 경우 `hooks/`, `.mcp.json`, `agents/`, `output-styles/`에 대한 변경 사항은 `/reload-plugins`가 적용되어야 합니다.
</Note>

#### 상위 및 중첩 디렉터리에서의 자동 탐색

프로젝트 스킬은 시작 디렉터리와 리포지토리 루트까지의 모든 상위 디렉터리에 있는 `.claude/skills/`에서 로드되므로 하위 디렉터리에서 Claude를 시작해도 루트에 정의된 스킬이 수집됩니다. 시작 디렉터리 아래의 하위 디렉터리에 있는 파일로 작업할 때 Claude Code는 주문형 방식으로 중첩된 `.claude/skills/` 디렉터리에서 스킬을 탐색합니다. 예를 들어 `packages/frontend/`에서 파일을 편집 중인 경우 Claude Code는 `packages/frontend/.claude/skills/`에서도 스킬을 찾습니다. 이는 패키지에 자체 스킬이 있는 모노리포 설정을 지원합니다.

각 스킬은 진입점으로 `SKILL.md`가 있는 디렉터리입니다:

```text theme={null}
my-skill/
├── SKILL.md           # 메인 지침 (필수)
├── template.md        # Claude가 채울 템플릿
├── examples/
│   └── sample.md      # 예상 형식을 보여주는 예시 출력
└── scripts/
    └── validate.sh    # Claude가 실행할 수 있는 스크립트
```

`SKILL.md`에는 메인 지침이 포함되어 있으며 필수 항목입니다. 다른 파일은 선택 사항이며 Claude가 채울 템플릿, 예상 형식을 보여주는 예시 출력, Claude가 실행할 수 있는 스크립트 또는 상세한 참조 문서와 같이 더 강력한 스킬을 구축할 수 있게 해줍니다. Claude가 포함된 내용과 로드할 시점을 알 수 있도록 `SKILL.md`에서 이러한 파일을 참조하세요. 자세한 내용은 [지원 파일 추가](#지원-파일-추가)를 참조하세요.

<Note>
  `.claude/commands/`에 있는 파일은 여전히 작동하며 동일한 [프론트매터](#프론트매터-참조)를 지원합니다. 스킬은 지원 파일과 같은 추가 기능을 지원하므로 스킬 사용을 권장합니다.
</Note>

#### 추가 디렉터리의 스킬

`--add-dir` 플래그 및 `/add-dir` 명령은 구성 탐색보다는 [파일 접근 권한을 부여](/docs/en/permissions#additional-directories-grant-file-access-not-configuration)하지만 스킬은 예외입니다. 추가된 디렉터리 내의 `.claude/skills/`는 자동으로 로드됩니다. 이 예외는 `--add-dir` 및 `/add-dir`에만 적용됩니다. `settings.json`에 있는 `permissions.additionalDirectories` 설정은 파일 접근 권한만 부여하고 스킬을 로드하지 않습니다. 세션 중에 편집 사항이 어떻게 반영되는지는 [라이브 변경 감지](#라이브-변경-감지)를 참조하세요.

명령 및 출력 스타일과 같은 기타 `.claude/` 구성은 추가 디렉터리에서 로드되지 않습니다. 로드되는 항목과 로드되지 않는 항목의 전체 목록, 프로젝트 전반에 걸쳐 구성을 공유하는 권장 방법은 [예외 표](/docs/en/permissions#additional-directories-grant-file-access-not-configuration)를 참조하세요.

<Note>
  `--add-dir` 디렉터리의 CLAUDE.md 파일은 기본적으로 로드되지 않습니다. 이를 로드하려면 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`을 설정하세요. [추가 디렉터리에서 로드](/docs/en/memory#load-from-additional-directories)를 참조하세요.
</Note>

#### Cowork 및 클라우드 세션의 스킬

[루틴](/docs/en/routines)을 포함하여 [Cowork](https://claude.com/product/cowork) 세션 및 [클라우드 세션](/docs/en/claude-code-on-the-web#the-cloud-environment)은 머신의 `~/.claude/skills/`를 읽지 않습니다. 대화형 및 예약된 Cowork 세션은 세션 시작 시 동기화되는 claude.ai 계정에 활성화된 스킬을 로드합니다. 데스크톱 앱 사이드바의 **Customize** 또는 claude.ai의 스킬 설정에서 이를 관리하세요. 클라우드 세션은 클론된 리포지토리의 `.claude/skills/`에 커밋된 프로젝트 스킬을 추가로 로드합니다.

스킬이 머신의 `~/.claude/skills/`에만 존재하는 경우, 각 루틴 실행이 새로운 원격 세션으로 시작되기 때문에 [루틴](/docs/en/routines)이 이를 호출할 때 스킬을 찾을 수 없다고 Claude Code가 보고합니다. 이러한 세션에서 개인 스킬을 사용할 수 있도록 하려면:

* Cowork 및 클라우드 세션의 경우 claude.ai 계정에 스킬을 활성화하세요.
* 클라우드 세션의 경우 대신 리포지토리의 `.claude/skills/`에 스킬을 커밋하거나 리포지토리의 `.claude/settings.json`에 선언된 플러그인으로 출하할 수 있습니다. 리포지토리에 선언된 플러그인은 [세션 시작 시 설치됩니다](/docs/en/claude-code-on-the-web#what’s-available-in-cloud-sessions). 사용자 설정에서만 활성화된 플러그인은 전달되지 않습니다.

[데스크톱 예약 작업](/docs/en/desktop-scheduled-tasks)은 다릅니다: 머신에서 로컬로 실행되고 다른 로컬 세션과 동일한 위치에서 스킬을 로드합니다.

## 스킬 구성

스킬은 `SKILL.md` 상단의 YAML 프론트매터와 그 뒤에 오는 Markdown 내용을 통해 구성됩니다.

### 스킬 내용 유형

스킬 파일에는 모든 지침이 포함될 수 있지만 이를 호출하는 방법을 생각하면 포함할 내용을 결정하는 데 도움이 됩니다:

**참조 내용**은 현재 작업에 Claude가 적용할 지식을 추가합니다. 관례, 패턴, 스타일 가이드, 도메인 지식. 이 내용은 Claude가 대화 컨텍스트와 함께 사용할 수 있도록 인라인으로 실행됩니다.

```yaml theme={null}
---
name: api-conventions
description: 이 코드베이스를 위한 API 디자인 패턴
---

API 엔드포인트를 작성할 때:
- RESTful 명명 규칙을 사용하세요.
- 일관된 오류 형식을 반환하세요.
- 요청 유효성 검사를 포함하세요.
```

**작업 내용**은 배포, 커밋 또는 코드 생성과 같은 특정 작업에 대해 Claude에게 단계별 지침을 제공합니다. 이는 종종 Claude가 실행할 시점을 결정하도록 하는 대신 `/skill-name`으로 직접 호출하려는 작업입니다. Claude가 자동으로 트리거하지 못하도록 하려면 `disable-model-invocation: true`를 추가하세요. 아래 예시는 자체 서브에이전트 컨텍스트에서 스킬을 실행하는 `context: fork`를 추가합니다. [서브에이전트에서 스킬 실행](#서브에이전트에서-스킬-실행)을 참조하세요.

```yaml theme={null}
---
name: deploy
description: 애플리케이션을 프로덕션에 배포
context: fork
disable-model-invocation: true
---

애플리케이션 배포:
1. 테스트 수트 실행
2. 애플리케이션 빌드
3. 배포 타깃으로 푸시
```

`SKILL.md`에는 무엇이든 포함될 수 있지만 스킬이 어떻게 호출되기를 원하는지(사용자에 의해, Claude에 의해, 또는 둘 다에 의해)와 어디서 실행되기를 원하는지(인라인으로 또는 서브에이전트에서)를 잘 생각해보면 포함할 내용을 결정하는 데 도움이 됩니다. 복잡한 스킬의 경우 메인 스킬에 초점을 맞추기 위해 [지원 파일을 추가](#지원-파일-추가)할 수도 있습니다.

본문 자체는 간결하게 유지하세요. 스킬이 로드되면 해당 콘텐츠가 [턴 전체에 걸쳐 컨텍스트에 지속되므로](#스킬-콘텐츠-수명-주기) 모든 라인이 반복적인 토큰 비용입니다. 이유나 방법을 서술하기보다는 수행할 작업을 명시하고 [CLAUDE.md 내용](/docs/en/best-practices#write-an-effective-claude-md)에 적용하는 것과 동일한 간결성 테스트를 적용하세요.

### 프론트매터 참조

Markdown 내용 외에도 `SKILL.md` 파일 상단의 `---` 마커 사이의 YAML 프론트매터 필드를 사용하여 스킬 동작을 구성할 수 있습니다:

```yaml theme={null}
---
name: my-skill
description: 이 스킬이 하는 일
disable-model-invocation: true
allowed-tools: Read Grep
---

스킬 지침을 여기에 작성하세요...
```

모든 필드는 선택 사항입니다. Claude가 스킬을 사용할 시기를 알 수 있도록 `description`만 권장됩니다.

| 필드 | 필수 여부 | 설명 |
| :--- | :--- | :--- |
| `name` | 아니요 | 스킬 목록에 표시되는 이름. 기본값은 디렉터리 이름입니다. 이 필드가 스킬을 호출하기 위해 입력하는 이름과 상호작용하는 방식은 [스킬이 명령 이름을 얻는 방식](#스킬이-명령-이름을-얻는-방식)을 참조하세요. |
| `description` | 권장 | 스킬이 하는 일과 사용 시기. Claude는 이를 사용하여 스킬을 적용할 시기를 결정합니다. 생략하면 Markdown 내용의 첫 번째 단락을 사용합니다. 주요 사용 사례를 먼저 작성하세요: 결합된 `description` 및 `when_to_use` 텍스트는 컨텍스트 사용량을 줄이기 위해 스킬 목록에서 1,536자로 잘립니다. |
| `when_to_use` | 아니요 | 트리거 문구나 예시 요청과 같이 Claude가 스킬을 호출해야 하는 시점에 대한 추가 컨텍스트. 스킬 목록의 `description` 뒤에 추가되며 1,536자 제한에 합산됩니다. |
| `argument-hint` | 아니요 | 예상되는 인수를 나타내기 위해 자동 완성 중에 표시되는 힌트. 예: `[issue-number]` 또는 `[filename] [format]`. |
| `arguments` | 아니요 | 스킬 내용에서 [`$name` 치환](#사용-가능한-문자열-치환)을 위한 이름 지정된 위치 인수. 공백으로 구분된 문자열 또는 YAML 목록을 수락합니다. 이름은 순서대로 인수 위치에 매핑됩니다. |
| `disable-model-invocation` | 아니요 | Claude가 이 스킬을 자동으로 로드하지 못하도록 하려면 `true`로 설정하세요. `/name`으로 수동 트리거하려는 워크플로에 사용합니다. 또한 스킬이 [서브에이전트로 사전 로드](/docs/en/sub-agents#preload-skills-into-subagents)되는 것을 방지합니다. {/* min-version: 2.1.196 */}v2.1.196부터 [예약된 작업](/docs/en/scheduled-tasks)이 프롬프트로 스킬과 함께 발생할 때 스킬이 실행되는 것도 방지합니다. 기본값: `false`. |
| `user-invocable` | 아니요 | `/` 메뉴에서 숨기려면 `false`로 설정하세요. 사용자가 직접 호출하면 안 되는 백그라운드 지식에 사용합니다. 기본값: `true`. |
| `allowed-tools` | 아니요 | 이 스킬을 호출하는 턴 동안 Claude가 승인을 요청하지 않고 사용할 수 있는 도구. 다음 메시지를 보낼 때 부여가 해제됩니다. 공백 또는 쉼표로 구분된 문자열이나 YAML 목록을 수락합니다. [스킬에 대한 도구 사전 승인](#스킬에-대한-도구-사전-승인)을 참조하세요. |
| `disallowed-tools` | 아니요 | 이 스킬이 활성화되어 있는 동안 Claude의 사용 가능한 풀에서 제거되는 도구. 백그라운드 루프에 대해 `AskUserQuestion`과 같이 특정 도구를 절대 호출해서는 안 되는 자율 스킬에 사용합니다. 공백 또는 쉼표로 구분된 문자열이나 YAML 목록을 수락합니다. 다음 메시지를 보낼 때 제한이 해제됩니다. 거부 규칙과 마찬가지로 이 필드는 다른 도구가 남아 있는 동안 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 제거할 수 없습니다. |
| `model` | 아니요 | 이 스킬이 활성화되어 있을 때 사용할 모델. 재지정은 현재 턴의 남은 기간 동안 적용되며 설정에 저장되지 않습니다. 다음 프롬프트에서 세션 모델이 다시 시작됩니다. [`/model`](/docs/en/model-config)과 동일한 값을 허용하거나 활성 모델을 유지하려면 `inherit`를 사용합니다. 조직의 [`availableModels`](/docs/en/model-config#restrict-model-selection) 허용 목록에서 제외된 값은 사용되지 않으며 세션은 현재 모델을 유지합니다. |
| `effort` | 아니요 | 이 스킬이 활성화되어 있을 때의 [노력 수준](/docs/en/model-config#adjust-effort-level). 세션 노력 수준을 오버라이드합니다. 기본값: 세션에서 상속. 옵션: `low`, `medium`, `high`, `xhigh`, `max`. 사용 가능한 수준은 모델에 따라 다릅니다. |
| `context` | 아니요 | 포크된 서브에이전트 컨텍스트에서 실행하려면 `fork`로 설정하세요. [서브에이전트에서 스킬 실행](#서브에이전트에서-스킬-실행)을 참조하세요. |
| `agent` | 아니요 | `context: fork`가 설정되었을 때 사용할 서브에이전트 유형. |
| `hooks` | 아니요 | 이 스킬의 수명 주기에 포함되는 훅. 구성 형식은 [스킬 및 에이전트의 훅](/docs/en/hooks#hooks-in-skills-and-agents)을 참조하세요. |
| `paths` | 아니요 | 이 스킬이 활성화되는 시점을 제한하는 글롭 패턴. 쉼표로 구분된 문자열이나 YAML 목록을 수락합니다. 설정되면 Claude는 패턴과 일치하는 파일로 작업할 때만 스킬을 자동으로 로드합니다. [경로 특정 규칙](/docs/en/memory#path-specific-rules)과 동일한 형식을 사용합니다. |
| `shell` | 아니요 | 이 스킬의 `` !`command` `` 및 ` ```! ` 블록에 사용할 셸. `bash`(기본값) 또는 `powershell`을 수락합니다. `powershell`을 설정하면 [PowerShell 도구](/en/tools-reference#powershell-tool)가 활성화되어 있을 때 PowerShell을 통해 인라인 셸 명령이 실행됩니다: Git Bash가 없는 Windows에서는 기본적으로 켜져 있으며 다른 곳에서는 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`로 활성화합니다. |

#### 스킬이 명령 이름을 얻는 방식

스킬을 호출하기 위해 입력하는 명령어는 스킬 파일이 상주하는 위치에서 오며, 플러그인 스킬의 경우 프론트매터 `name` 필드에서도 옴니다. 개인 또는 프로젝트 스킬에서 `name`은 스킬 목록에 표시되는 표시 레이블만 설정하며 명령은 여전히 디렉터리 또는 파일 이름에서 옵니다. 플러그인 스킬에서 `name`은 명령의 마지막 세그먼트를 설정하고 플러그인 접두사는 유지됩니다.

아래 표는 각 레이아웃에 대해 명령 이름이 나타나는 방식을 보여줍니다:

| 스킬 위치 | 명령 이름 소스 | 예시 |
| :--- | :--- | :--- |
| `~/.claude/skills/` 또는 `.claude/skills/` 아래의 스킬 디렉터리 | 디렉터리 이름 | `.claude/skills/deploy-staging/SKILL.md` → `/deploy-staging` |
| 다른 스킬과 이름이 충돌하는 경우의 [중첩된](#스킬-저장-위치) `.claude/skills/` 디렉터리 | 작업 디렉터리 기준 상대 하위 디렉터리 경로 후 스킬 디렉터리 이름 | `apps/web/.claude/skills/deploy/SKILL.md` → `/apps/web:deploy` |
| `.claude/commands/` 아래의 파일 | 확장자를 제외한 파일 이름 | `.claude/commands/deploy.md` → `/deploy` |
| 플러그인 `skills/` 하위 디렉터리 | 프론트매터 `name` 또는 디렉터리 이름, 플러그인에 의해 네임스페이스 지정됨 | `my-plugin/skills/review/SKILL.md` → `/my-plugin:review`, 또는 `name: fancy`일 때 `/my-plugin:fancy` |
| 플러그인 루트 `SKILL.md` | 프론트매터 `name`, 대체 수단으로 플러그인 디렉터리 이름 | `name: review`인 `my-plugin/SKILL.md` → `/my-plugin:review`. [경로 동작 규칙](/docs/en/plugins-reference#path-behavior-rules) 참조 |

플러그인 스킬에서 프론트매터 `name`은 명령의 마지막 세그먼트에서 디렉터리 이름을 대체하므로 `name: fancy`가 있는 `my-plugin/skills/review/SKILL.md`는 `/my-plugin:fancy`가 됩니다. 다른 명령어가 이미 해당 이름을 사용하지 않는 한 단순한 `/fancy`도 스킬을 호출합니다. v2.1.216 이전에는 프론트매터 이름이 전체 명령 이름을 대체했으므로 메뉴에는 플러그인 접두사 없이 `/fancy`가 표시되었고 `/my-plugin:fancy`는 자동 완성이 되지 않았습니다.

플러그인 루트 `SKILL.md`의 경우 이름을 가져올 스킬 디렉터리가 없으므로 `name`이 전체 최종 세그먼트를 제공합니다. `name` 필드가 없으면 Claude Code는 플러그인의 디렉터리 이름으로 대체합니다.

#### 사용 가능한 문자열 치환

스킬은 스킬 내용의 동적 값에 대한 문자열 치환을 지원합니다:

| 변수 | 설명 |
| :--- | :--- |
| `$ARGUMENTS` | 스킬을 호출할 때 전달된 모든 인수. `$ARGUMENTS`가 내용에 존재하지 않으면 인수가 `ARGUMENTS: <value>`로 추가됩니다. |
| `$ARGUMENTS[N]` | 0 기반 인덱스로 특정 인수에 접근합니다(예: 첫 번째 인수의 경우 `$ARGUMENTS[0]`). |
| `$N` | 첫 번째 인수의 경우 `$0`, 두 번째 인수의 경우 `$1`과 같이 `$ARGUMENTS[N]`의 축약형. |
| `$name` | [`arguments`](#프론트매터-참조) 프론트매터 목록에 선언된 이름 지정된 인수. 이름은 순서대로 위치에 매핑되므로 `arguments: [issue, branch]`를 사용하면 자리 표시자 `$issue`가 첫 번째 인수로 확장되고 `$branch`가 두 번째 인수로 확장됩니다. |
| `${CLAUDE_SESSION_ID}` | 현재 세션 ID. 로깅, 세션 특정 파일 생성 또는 세션과 스킬 출력 연관 시 유용합니다. |
| `${CLAUDE_EFFORT}` | 현재 노력 수준: `low`, `medium`, `high`, `xhigh` 또는 `max`. Ultracode는 별도의 수준이 아니며 `xhigh`로 보고됩니다. 활성 노력 설정에 맞게 스킬 지침을 조정할 때 사용하세요. |
| `${CLAUDE_SKILL_DIR}` | 스킬의 `SKILL.md` 파일이 포함된 디렉터리. 플러그인 스킬의 경우 플러그인 루트가 아니라 플러그인 내의 스킬 하위 디렉터리입니다. 현재 작업 디렉터리와 상관없이 스킬과 함께 번들로 제공되는 스크립트나 파일을 참조하려면 bash 주입 명령에서 이를 사용하세요. |
| `${CLAUDE_PROJECT_DIR}` | 프로젝트 루트 디렉터리. 이는 [훅](/docs/en/hooks#reference-scripts-by-path) 및 MCP 서버가 `CLAUDE_PROJECT_DIR`로 수신하는 것과 동일한 경로입니다. 스킬이 설치된 위치와 상관없이 `${CLAUDE_PROJECT_DIR}/.claude/hooks/helper.sh`와 같이 프로젝트 로컬 스크립트나 파일을 참조할 때 사용하세요. |

Claude Code는 스킬의 Markdown 내용과 [`allowed-tools`](#프론트매터-참조) 프론트매터의 Bash 규칙이라는 두 곳에서 `${CLAUDE_SKILL_DIR}` 및 `${CLAUDE_PROJECT_DIR}`를 치환합니다. 두 장소에서 동일한 변수를 사용하면 스킬이 권한 프롬프트 없이 번들된 스크립트를 실행할 수 있습니다. 다음 스킬은 이 패턴을 보여줍니다:

```yaml theme={null}
---
name: render-chart
description: Render a chart from a CSV file
allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/render.sh *)
---

Run `${CLAUDE_SKILL_DIR}/scripts/render.sh <csv-file>` to render the chart.
```

이 스킬이 `~/.claude/skills/render-chart/`에 설치된 경우 `${CLAUDE_SKILL_DIR}`의 두 발생 항목 모두 해당 디렉터리로 확장됩니다. 그런 다음 `allowed-tools` 규칙은 스킬 본문이 Claude에게 실행하도록 지시하는 정확한 명령과 일치하므로 스크립트가 프롬프트 없이 실행됩니다.

`${CLAUDE_SKILL_DIR}`에 대한 `allowed-tools` 치환에는 Claude Code v2.1.129 이상이 필요합니다. 이전 버전에서는 규칙이 리터럴 `${CLAUDE_SKILL_DIR}` 문자열로 남아 일치하지 않으므로 명령어가 계속 권한을 요청합니다.

`${CLAUDE_PROJECT_DIR}` 치환에는 Claude Code v2.1.196 이상이 필요합니다.

인덱스 지정된 인수는 셸 스타일 따옴표를 사용하므로 여러 단어로 된 값을 따옴표로 감싸 단일 인수로 전달합니다. 예를 들어 `/my-skill "hello world" second`를 사용하면 `$0`이 `hello world`로 확장되고 `$1`이 `second`로 확장됩니다. `$ARGUMENTS` 자리 표시자는 항상 입력된 전체 인수 문자열로 확장됩니다.

단 하나의 인수만 전달되었을 때 `$2`와 같이 해당하는 인수가 없는 인덱스 지정된 자리 표시자는 콘텐츠에 그대로 남습니다. 해당하는 인수가 없는 [`arguments`](#프론트매터-참조) 프론트매터의 이름 지정된 자리 표시자는 빈 문자열로 확장됩니다.

텍스트에서 `$1.00`과 같이 숫자, `ARGUMENTS` 또는 선언된 인수 이름 앞에 리터럴 `$`를 포함하려면 백슬래시로 이스케이프하세요: `\$1.00`. 다른 `$` 앞의 백슬래시는 그대로 유지됩니다. 토큰 바로 앞의 단일 백슬래시만 이스케이프 처리됩니다. `\\$1`과 같이 이중 백슬래시는 두 백슬래시를 모두 남기며 `$1`은 여전히 인수 값으로 확장됩니다.

**치환을 사용하는 예시:**

```yaml theme={null}
---
name: session-logger
description: 이 세션의 활동 기록
---

logs/${CLAUDE_SESSION_ID}.log에 다음을 기록하세요:

$ARGUMENTS
```

### 지원 파일 추가

스킬은 해당 디렉터리에 여러 파일을 포함할 수 있습니다. 이렇게 하면 Claude가 필요할 때만 상세한 참조 자료에 접근할 수 있게 하여 `SKILL.md`를 핵심에 집중시킬 수 있습니다. 대용량 참조 문서, API 사양 또는 예제 모음은 스킬이 실행될 때마다 컨텍스트로 로드할 필요가 없습니다.

```text theme={null}
my-skill/
├── SKILL.md (필수 - 개요 및 내비게이션)
├── reference.md (상세 API 문서 - 필요할 때 로드됨)
├── examples.md (사용 예시 - 필요할 때 로드됨)
└── scripts/
    └── helper.py (유틸리티 스크립트 - 로드되지 않고 실행됨)
```

Claude가 각 파일에 포함된 내용과 로드할 시점을 알 수 있도록 `SKILL.md`에서 지원 파일을 참조하세요:

```markdown theme={null}
## 추가 리소스

- 전체 API 세부 정보는 [reference.md](reference.md) 참조
- 사용 예시는 [examples.md](examples.md) 참조
```

<Tip>`SKILL.md`를 500줄 이하로 유지하세요. 상세한 참조 자료는 별도의 파일로 이동하세요.</Tip>

### 스킬 호출 주체 제어

기본적으로 사용자와 Claude 모두 임의의 스킬을 호출할 수 있습니다. `/skill-name`을 입력하여 직접 호출할 수 있으며 Claude는 대화와 관련이 있을 때 자동으로 로드할 수 있습니다. 두 개의 프론트매터 필드를 통해 이를 제한할 수 있습니다:

* **`disable-model-invocation: true`**: 사용자만 스킬을 호출할 수 있습니다. `/commit`, `/deploy`, `/send-slack-message`와 같이 부작용이 있거나 타이밍을 제어하려는 워크플로에 사용하세요. 코드가 준비되어 보인다고 해서 Claude가 임의로 배포 결정을 내리는 것을 원하지 않을 것입니다.

* **`user-invocable: false`**: Claude만 스킬을 호출할 수 있습니다. 명령으로 실행 가능하지 않은 백그라운드 지식에 사용하세요. `legacy-system-context` 스킬은 이전 시스템의 작동 방식을 설명합니다. Claude는 관련이 있을 때 이를 알아야 하지만 `/legacy-system-context`는 사용자가 취할 수 있는 의미 있는 동작이 아닙니다.

다음 예시는 사용자만 트리거할 수 있는 배포 스킬을 생성합니다. `disable-model-invocation: true`를 설정하면 Claude가 스킬을 자동으로 실행할 수 없습니다:

```yaml theme={null}
---
name: deploy
description: 애플리케이션을 프로덕션에 배포
disable-model-invocation: true
---

$ARGUMENTS를 프로덕션에 배포:

1. 테스트 수트 실행
2. 애플리케이션 빌드
3. 배포 타깃으로 푸시
4. 배포 성공 확인
```

두 필드가 호출 및 컨텍스트 로딩에 미치는 영향은 다음과 같습니다:

| 프론트매터 | 사용자 호출 가능 | Claude 호출 가능 | 컨텍스트로 로드되는 시점 |
| :--- | :--- | :--- | :--- |
| (기본값) | 예 | 예 | 설명은 항상 컨텍스트에 존재, 호출 시 전체 스킬 로드 |
| `disable-model-invocation: true` | 예 | 아니요 | 설명이 컨텍스트에 없음, 사용자 호출 시 전체 스킬 로드 |
| `user-invocable: false` | 아니요 | 예 | 설명은 항상 컨텍스트에 존재, 호출 시 전체 스킬 로드 |

<Note>
  일반 세션에서는 스킬 설명이 컨텍스트에 로드되므로 Claude가 사용 가능한 항목을 알 수 있지만 전체 스킬 내용은 호출될 때만 로드됩니다. [사전 로드된 스킬이 있는 서브에이전트](/docs/en/sub-agents#preload-skills-into-subagents)는 다르게 작동합니다. 시작 시 전체 스킬 내용이 주입됩니다.
</Note>

### 스킬 콘텐츠 수명 주기

사용자나 Claude가 스킬을 호출하면 렌더링된 `SKILL.md` 내용이 단일 메시지로 대화에 진입하고 세션의 남은 기간 동안 그대로 유지됩니다. 이 지속성은 스킬 지침에 적용되며 권한에는 적용되지 않습니다: [`allowed-tools`](#스킬에-대한-도구-사전-승인) 부여는 스킬 내용이 [컨텍스트에 남아 있더라도](#스킬-콘텐츠-수명-주기) 다음 메시지를 보낼 때 지워집니다. 스킬을 다시 호출하면 해당 턴에 대해 권한이 다시 적용됩니다. Claude Code는 나중 턴에서 스킬 파일을 다시 읽지 않으므로 일회성 단계가 아닌 작업 전체에 적용되어야 하는 지침을 지속 지침으로 작성하세요.

Claude가 이미 컨텍스트에 있는 사본과 렌더링된 내용이 동일한 스킬을 다시 호출하면 Claude Code는 콘텐츠 사본을 두 번 추가하지 않고 스킬이 이미 로드되었다는 짧은 참고 사항을 추가합니다. 인수가 변경되었거나 [동적 컨텍스트](#동적-컨텍스트-주입) 명령이 새 출력을 생성하여 렌더링된 내용이 다를 때 Claude Code는 전체 내용을 다시 추가합니다. v2.1.202 이전에는 모든 재호출 시 스킬 지침의 또 다른 전체 사본이 추가되었습니다.

[자동 압축](/docs/en/how-claude-code-works#when-context-fills-up)은 토큰 예산 내에서 호출된 스킬을 앞으로 전달합니다. 대화가 컨텍스트를 비우기 위해 요약되면 Claude Code는 요약 후 각 스킬의 가장 최근 호출을 다시 첨부하여 각 스킬의 처음 5,000개 토큰을 유지합니다. 다시 첨부된 스킬은 25,000개 토큰의 결합 예산을 공유합니다. Claude Code는 가장 최근에 호출된 스킬부터 이 예산을 채우므로 한 세션에서 많은 스킬을 호출한 경우 압축 후 이전 스킬이 완전히 삭제될 수 있습니다.

스킬이 첫 번째 응답 후에 동작에 미치는 영향이 중단된 것처럼 보이면 내용이 여전히 존재하며 모델이 다른 도구나 접근 방식을 선택하는 것입니다. 모델이 계속해서 선호하도록 스킬의 `description` 및 지침을 강화하거나 [훅](/docs/en/hooks)을 사용하여 동작을 확정적으로 강제 적용하세요. 스킬이 크거나 스킬 이후에 다른 여러 스킬을 호출한 경우 압축 후 다시 호출하여 전체 내용을 복원하세요.

### 스킬에 대한 도구 사전 승인

`allowed-tools` 필드는 스킬을 호출하는 턴 동안 나열된 도구에 대한 권한을 부여하므로 Claude가 승인을 요청하지 않고 도구를 사용할 수 있습니다. 스킬 내용이 [컨텍스트에 남아 있더라도](#스킬-콘텐츠-수명-주기) 다음 메시지를 보낼 때 부여가 지워집니다. 스킬을 다시 호출하면 해당 턴에 대해 권한이 다시 적용됩니다. 사용 가능한 도구가 제한되지는 않습니다: 모든 도구를 호출할 수 있으며 나열되지 않은 도구는 [권한 설정](/docs/en/permissions)이 적용됩니다. 단일 턴이 아닌 전체 세션 동안 도구를 사전 승인하려면 해당 권한 설정에 허용 규칙을 추가하세요.

프로젝트의 `.claude/skills/` 디렉터리에 체크인된 스킬의 경우, `.claude/settings.json` 권한 규칙과 마찬가지로 폴더에 대한 워크스페이스 신뢰 대화 상자를 수락한 후 `allowed-tools`가 적용됩니다. 스킬이 광범위한 도구 접근 권한을 자율적으로 부여할 수 있으므로 리포지토리를 신뢰하기 전에 프로젝트 스킬을 검토하세요.

이 스킬을 사용하면 스킬을 호출할 때마다 Claude가 매번 승인받지 않고 Git 명령을 실행할 수 있습니다:

```yaml theme={null}
---
name: commit
description: 현재 변경 사항을 스테이징하고 커밋
disable-model-invocation: true
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
```

스킬이 활성화되어 있는 동안 Claude의 사용 가능한 풀에서 도구를 제거하려면 스킬 프론트매터의 `disallowed-tools`에 나열하세요. 다음 메시지를 보낼 때 제한이 해제됩니다. 거부 규칙과 마찬가지로 이 필드는 다른 도구가 남아 있는 동안 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 제거할 수 없습니다. 모든 스킬과 프롬프트 전반에서 도구를 차단하려면 [권한 설정](/docs/en/permissions)에 거부 규칙을 추가하세요.

### 스킬에 인수 전달하기

사용자와 Claude 모두 스킬을 호출할 때 인수를 전달할 수 있습니다. 인수는 `$ARGUMENTS` 자리 표시자를 통해 사용할 수 있습니다.

이 스킬은 번호로 GitHub 이슈를 수정합니다. `$ARGUMENTS` 자리 표시자는 스킬 이름 뒤에 오는 내용으로 대체됩니다:

```yaml theme={null}
---
name: fix-issue
description: GitHub 이슈 수정
disable-model-invocation: true
---

코딩 표준에 따라 GitHub 이슈 $ARGUMENTS를 수정하세요.

1. 이슈 설명 읽기
2. 요구 사항 이해
3. 수정 사항 구현
4. 테스트 작성
5. 커밋 생성
```

`/fix-issue 123`을 실행하면 Claude는 "코딩 표준에 따라 GitHub 이슈 123을 수정하세요..."를 수신합니다.

인수와 함께 스킬을 호출하지만 스킬에 `$ARGUMENTS`가 포함되어 있지 않은 경우 Claude Code는 Claude가 입력한 내용을 계속 볼 수 있도록 스킬 내용 끝에 `ARGUMENTS: <your input>`을 추가합니다.

한 메시지의 시작 부분에 여러 스킬을 쌓을 수도 있습니다. {/* min-version: 2.1.199 */}v2.1.199부터 `/code-review /fix-issue 123`을 입력하면 두 스킬이 모두 로드되고 후미 텍스트 `123`이 각각에 `$ARGUMENTS`로 전달됩니다. 이전 버전에서는 첫 번째 스킬만 로드되어 `/fix-issue 123`을 리터럴 인수 텍스트로 수신했습니다.

Claude Code는 첫 번째 스킬과 그 뒤에 쌓인 최대 5개까지 확장합니다. 확장은 인라인 사용자 호출 가능 스킬이 아닌 첫 번째 토큰에서 중지되므로 [포크된 서브에이전트](#서브에이전트에서-스킬-실행)로 실행되는 스킬이나 `/loop`와 같이 인수가 슬래시 명령으로 시작될 수 있는 스킬도 거기서 실행을 종료합니다. 해당 토큰과 그 이후의 모든 것은 확장된 모든 스킬의 인수 텍스트가 됩니다.

위치별로 개별 인수에 접근하려면 `$ARGUMENTS[N]` 또는 더 짧은 `$N`을 사용하세요:

```yaml theme={null}
---
name: migrate-component
description: 한 프레임워크에서 다른 프레임워크로 컴포넌트 마이그레이션
---

$ARGUMENTS[0] 컴포넌트를 $ARGUMENTS[1]에서 $ARGUMENTS[2]로 마이그레이션하세요.
기존 모든 동작과 테스트를 보존하세요.
```

`/migrate-component SearchBar React Vue`를 실행하면 `$ARGUMENTS[0]`이 `SearchBar`로, `$ARGUMENTS[1]`이 `React`로, `$ARGUMENTS[2]`가 `Vue`로 대체됩니다. `$N` 축약형을 사용하는 동일한 스킬:

```yaml theme={null}
---
name: migrate-component
description: 한 프레임워크에서 다른 프레임워크로 컴포넌트 마이그레이션
---

$0 컴포넌트를 $1에서 $2로 마이그레이션하세요.
기존 모든 동작과 테스트를 보존하세요.
```

## 고급 패턴

### 동적 컨텍스트 주입

`` !`<command>` `` 구문은 스킬 내용이 Claude에게 전송되기 전에 셸 명령을 실행합니다. 명령 출력이 자리 표시자를 대체하므로 Claude는 명령 자체가 아니라 실제 데이터를 수신합니다.

이 스킬은 GitHub CLI를 통해 라이브 PR 데이터를 가져와 풀 리퀘스트를 요약합니다. `` !`gh pr diff` `` 및 기타 명령이 먼저 실행되고 해당 출력이 프롬프트에 삽입됩니다:

```yaml theme={null}
---
name: pr-summary
description: 풀 리퀘스트의 변경 사항 요약
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## 풀 리퀘스트 컨텍스트
- PR diff: !`gh pr diff`
- PR 댓글: !`gh pr view --comments`
- 변경된 파일: !`gh pr diff --name-only`

## 수행할 작업
이 풀 리퀘스트를 요약하세요...
```

이 스킬이 실행될 때:

1. 각 `` !`<command>` ``가 즉시 실행됩니다(Claude가 보기 전).
2. 출력이 스킬 내용의 자리 표시자를 대체합니다.
3. Claude는 실제 PR 데이터가 포함된 완전 렌더링 프롬프트를 수신합니다.

이는 Claude가 실행하는 것이 아니라 전처리 단계입니다. Claude는 최종 결과만 봅니다.

치환은 원본 파일에 대해 한 번 실행됩니다. 명령 출력은 일반 텍스트로 삽입되며 다른 `` !`<command>` `` 자리 표시자에 대해 다시 스캔되지 않으므로 명령이 나중 패스를 확장하기 위해 자리 표시자를 내보낼 수 없습니다.

인라인 형태는 라인의 시작 부분이나 공백 바로 뒤에 `!`가 나타날 때만 인식됩니다. `` KEY=!`cmd` ``와 같이 `!`가 다른 문자 뒤에 오면 자리 표시자가 리터럴 텍스트로 남고 명령이 실행되지 않습니다.

여러 줄 명령의 경우 인라인 형태 대신 ` ```! `로 시작하는 펜스 코드 블록을 사용하세요:

````markdown theme={null}
## 환경
```!
node --version
npm --version
git status --short
```
````

사용자, 프로젝트, 플러그인 또는 [추가 디렉터리](#추가-디렉터리의-스킬) 소스의 스킬 및 사용자 지정 명령에 대해 이 동작을 비활성화하려면 [설정](/docs/en/settings)에서 `"disableSkillShellExecution": true`를 설정하세요. 각 명령은 실행되는 대신 `[shell command execution disabled by policy]`로 대체됩니다. 번들 및 관리형 스킬은 영향을 받지 않습니다. 이 설정은 사용자가 오버라이드할 수 없는 [관리형 설정](/docs/en/permissions#managed-settings)에서 가장 유용합니다.

<Tip>
  스킬이 실행될 때 더 깊은 추론을 요청하려면 스킬 내용의 어디에든 `ultrathink`를 포함하세요. [일회성 심층 추론을 위해 ultrathink 사용](/docs/en/model-config#use-ultrathink-for-one-off-deep-reasoning)을 참조하세요.
</Tip>

### 서브에이전트에서 스킬 실행

스킬을 격리된 상태로 실행하려면 프론트매터에 `context: fork`를 추가하세요. 스킬 내용은 서브에이전트를 구동하는 프롬프트가 됩니다. 대화 기록에는 접근할 수 없습니다.

<Warning>
  `context: fork`는 명시적 지침이 있는 스킬에만 의미가 있습니다. 작업 없이 "이 API 규칙 사용"과 같은 지침만 스킬에 포함되어 있으면 서브에이전트는 지침은 받지만 실행 가능한 프롬프트는 받지 못하여 의미 있는 출력 없이 반환됩니다.
</Warning>

스킬과 [서브에이전트](/docs/en/sub-agents)는 두 방향으로 함께 작동합니다:

| 접근 방식 | 시스템 프롬프트 | 작업 | 추가 로드 항목 |
| :--- | :--- | :--- | :--- |
| `context: fork`가 있는 스킬 | 에이전트 유형에서 옴 | SKILL.md 내용 | 에이전트가 Explore 또는 Plan일 때를 제외하고 CLAUDE.md |
| `skills` 필드가 있는 서브에이전트 | 서브에이전트의 Markdown 본문 | Claude의 위임 메시지 | 사전 로드된 스킬 + CLAUDE.md |

`context: fork`를 사용하면 스킬에 작업을 작성하고 실행할 에이전트 유형을 선택합니다. 내장된 Explore 및 Plan 에이전트는 컨텍스트를 작게 유지하기 위해 [CLAUDE.md 및 git status를 건너뛰므로](/docs/en/sub-agents#what-loads-at-startup), `agent: Explore`를 사용하는 포크된 스킬은 SKILL.md 내용과 에이전트 자체의 시스템 프롬프트만 봅니다. 반대로 사용자 지정 서브에이전트를 정의하여 참조 자료로 스킬을 사용하는 방식은 [서브에이전트](/docs/en/sub-agents#preload-skills-into-subagents)를 참조하세요.

#### 예시: Explore 에이전트를 사용하는 리서치 스킬

이 스킬은 포크된 Explore 에이전트에서 리서치를 실행합니다. 스킬 내용은 작업이 되고 에이전트는 코드베이스 탐색에 최적화된 읽기 전용 도구를 제공합니다:

```yaml theme={null}
---
name: deep-research
description: 주제를 철저히 조사
context: fork
agent: Explore
---

$ARGUMENTS를 철저히 조사하세요:

1. Glob 및 Grep을 사용하여 관련 파일 찾기
2. 코드 읽기 및 분석
3. 구체적인 파일 참조를 포함하여 결과 요약
```

이 스킬이 실행될 때:

1. 새 격리 컨텍스트가 생성됩니다.
2. 서브에이전트는 스킬 내용을 프롬프트로 받습니다 ("$ARGUMENTS를 철저히 조사하세요...").
3. `agent` 필드가 실행 환경(모델, 도구 및 권한)을 결정합니다.
4. 결과가 요약되어 메인 대화로 반환됩니다.

`agent` 필드는 사용할 서브에이전트 구성을 지정합니다. 옵션에는 내장 에이전트(`Explore`, `Plan`, `general-purpose`) 또는 `.claude/agents/`에 있는 임의의 사용자 지정 서브에이전트가 포함됩니다. 생략하면 `general-purpose`를 사용합니다.

### Claude의 스킬 접근 제한

기본적으로 Claude는 `disable-model-invocation: true`가 설정되어 있지 않은 모든 스킬을 호출할 수 있습니다. `allowed-tools`를 정의하는 스킬은 스킬을 호출하는 턴 동안 매번 승인받지 않고 해당 도구에 접근할 수 있는 권한을 Claude에 부여하며, 다음 메시지를 보낼 때 부여가 지워집니다. [권한 설정](/docs/en/permissions)은 다른 모든 도구에 대한 기준 승인 동작을 계속 다룹니다. `/init`, `/review`, `/security-review`를 포함한 몇 가지 내장 명령도 Skill 도구를 통해 사용할 수 있습니다. `/compact`와 같은 다른 내장 명령은 사용할 수 없습니다.

Claude가 호출할 수 있는 스킬을 제어하는 세 가지 방법:

`/permissions`에서 Skill 도구를 거부하여 **모든 스킬 비활성화**:

```text theme={null}
# 거부 규칙에 추가:
Skill
```

[권한 규칙](/docs/en/permissions)을 사용하여 **특정 스킬 허용 또는 거부**:

```text theme={null}
# 특정 스킬만 허용
Skill(commit)
Skill(review-pr *)

# 특정 스킬 거부
Skill(deploy *)
```

권한 구문: 정확히 일치하려면 `Skill(name)`, 인수에 상관없이 접두사 일치를 하려면 `Skill(name *)`.

스킬 프론트매터에 `disable-model-invocation: true`를 추가하여 **개별 스킬 숨기기**. 이렇게 하면 Claude의 컨텍스트에서 스킬이 완전히 제거됩니다.

<Note>
  `user-invocable` 필드는 메뉴 가시성만 제어하며 Skill 도구 접근을 제어하지 않습니다. 프로그래밍 방식 호출을 차단하려면 `disable-model-invocation: true`를 사용하세요.
</Note>

### 설정에서 스킬 가시성 재정의

`skillOverrides` 설정은 스킬 자체의 프론트매터 대신 [설정](/docs/en/settings)에서 스킬 가시성을 제어합니다. 공유 프로젝트 리포지토리에 체크인된 스킬과 같이 SKILL.md를 편집하고 싶지 않은 스킬에 사용하세요. `/skills` 메뉴가 이를 대신 작성해 줍니다: 스킬을 강조 표시하고 `Space`를 눌러 상태를 순환한 다음 `Enter`를 눌러 `.claude/settings.local.json`에 저장하세요.

각 키는 스킬 이름이고 각 값은 4가지 상태 중 하나입니다:

| 값 | Claude에 나열됨 | `/` 메뉴에 포함 |
| :--- | :--- | :--- |
| `"on"` | 이름 및 설명 | 예 |
| `"name-only"` | 이름만 | 예 |
| `"user-invocable-only"` | 숨김 | 예 |
| `"off"` | 숨김 | 숨김 |

`/skills` 메뉴는 `"user-invocable-only"` 상태를 `user-only`로 표시합니다.

v2.1.199부터 `"off"`는 터미널 `/` 메뉴뿐만 아니라 [원격 제어(Remote Control)](/docs/en/remote-control) 클라이언트 및 [Agent SDK](/docs/en/agent-sdk/slash-commands) 호출자에게 알려진 명령 목록에서도 스킬을 숨깁니다. 전체 이름으로 숨겨진 스킬을 호출하면 스킬을 실행하는 대신 `skillOverrides` 오류가 반환됩니다.

`skillOverrides`에 없는 스킬은 `"on"`으로 처리됩니다. 아래 예시는 한 스킬을 이름으로만 축소하고 다른 스킬은 완전히 끕니다:

```json theme={null}
{
  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  }
}
```

플러그인 스킬은 `skillOverrides`의 영향을 받지 않습니다. 대신 `/plugin`을 통해 관리하세요.

## 스킬 평가 및 반복 개선

스킬이 트리거되는 것을 보는 것은 Claude가 스킬을 찾았다는 것을 의미할 뿐, 의도한 대로 수행되었음을 의미하지는 않습니다. 스킬이 제대로 작동하는지 확인하려면 두 가지를 별도로 측정하세요: Claude가 유효한 프롬프트에서 스킬을 호출하는지와 스킬이 실행될 때의 출력이 예상과 일치하는지입니다.

둘 다에 대한 검사는 기준선 비교입니다. 몇 가지 현실적인 프롬프트를 수집하고, 스킬을 사용할 수 있는 상태의 새 세션에서 각 프롬프트를 실행한 후 스킬이 [비활성화된](#설정에서-스킬-가시성-재정의) 상태에서 다시 실행하여 결과를 비교하세요. 새 세션이 중요한 이유는 스킬을 작성할 때 남아 있는 컨텍스트가 서면 지침의 공백을 가릴 수 있기 때문입니다.

### skill-creator로 평가 실행하기

[`skill-creator` 플러그인](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/skill-creator)은 Claude Code 내부의 비교 루프를 자동화합니다. 공식 마켓플레이스에서 설치하세요:

```text theme={null}
/plugin install skill-creator@claude-plugins-official
```

Claude Code에서 `Marketplace "claude-plugins-official" not found`라고 보고하는 경우 `/plugin marketplace add anthropics/claude-plugins-official` 명령으로 마켓플레이스를 추가하세요. 플러그인을 마켓플레이스에서 찾을 수 없다고 보고하는 경우 로컬 사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고친 후 설치를 다시 시도하세요.

설치 후 `/reload-plugins`를 실행하여 현재 세션에서 플러그인의 스킬을 사용할 수 있게 만드세요. 그런 다음 Claude에게 기존 스킬을 평가하도록 요청하세요(예: `evaluate my summarize-changes skill with skill-creator`). 플러그인이 테스트 케이스 작성 단계를 안내하고 루프를 실행합니다:

* **테스트 케이스**: 프롬프트, 입력 파일 및 예상되는 동작을 스킬 디렉터리 내의 `evals/evals.json`에 저장합니다.
* **격리된 실행**: 테스트 케이스당 [서브에이전트](/docs/en/sub-agents)를 포크하여 각 실행이 깨끗한 컨텍스트로 시작되도록 하고 토큰 수와 기간을 기록합니다.
* **채점**: 출력에 대한 각 어설션을 검사하고 결과 및 증거와 함께 합격 또는 불합격을 `grading.json`에 기록합니다.
* **벤치마크**: 스킬 사용 대 미사용에 대해 합격률, 시간 및 토큰을 `benchmark.json`으로 합산하므로 토큰 및 시간 오버헤드 대비 합격률 향상을 비교할 수 있습니다.
* **버전 비교**: 두 버전의 스킬 간에 맹검 A/B 테스트를 실행하여 수정 사항이 커밋하기 전에 개선되었음을 확인할 수 있습니다.
* **설명 조정**: 트리거되어야 하는 프롬프트와 트리거되어서는 안 되는 프롬프트를 생성하고 히트율을 측정하며 스킬이 잘못된 요청에 활성화될 때 설명 수정을 제안합니다.
* **검토 뷰어**: 각 출력을 검사하고 다음 반복에서 읽을 수 있는 정성적 피드백을 기록하는 HTML 리포트를 엽니다.

평가 파일 형식 및 전체 반복 워크플로는 agentskills.io의 [Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)를 참조하세요. 벤치마크 및 비교 모드에 대한 배경은 [skill-creator 공지사항](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills)을 참조하세요.

## 스킬 공유하기

대상 고객에 따라 다른 스코프로 스킬을 배포할 수 있습니다:

* **프로젝트 스킬**: `.claude/skills/`를 버전 제어에 커밋
* **플러그인**: [플러그인](/docs/en/plugins)에 `skills/` 디렉터리 생성
* **관리형**: [관리형 설정](/docs/en/settings#settings-files)을 통해 조직 전체에 배포

### 시각적 출력 생성

스킬은 모든 언어로 스크립트를 번들로 묶어 실행할 수 있어 단일 프롬프트에서 가능한 것을 넘어서는 기능을 Claude에 제공합니다. 한 가지 강력한 패턴은 시각적 출력을 생성하는 것입니다. 즉, 데이터 탐색, 디버깅 또는 리포트 생성을 위해 브라우저에서 열리는 대화형 HTML 파일입니다.

이 예시는 코드베이스 탐색기를 생성합니다: 디렉터리를 펼치거나 접을 수 있고, 한눈에 파일 크기를 확인하며, 색상별로 파일 유형을 식별할 수 있는 대화형 트리 뷰입니다.

스킬 디렉터리를 생성합니다:

```bash theme={null}
mkdir -p ~/.claude/skills/codebase-visualizer/scripts
```

이를 `~/.claude/skills/codebase-visualizer/SKILL.md`에 저장하세요. 설명은 Claude에게 이 스킬을 활성화할 시점을 알려주고 지침은 Claude에게 번들된 스크립트를 실행하도록 지시합니다. 스크립트 경로는 [`${CLAUDE_SKILL_DIR}`](#사용-가능한-문자열-치환)을 사용하므로 개인, 프로젝트 또는 플러그인 수준 중 어디에 설치되든 정확하게 해석됩니다:

````yaml theme={null}
---
name: codebase-visualizer
description: 코드베이스의 대화형 접이식 트리 시각화를 생성합니다. 새 리포지토리를 탐색하거나, 프로젝트 구조를 이해하거나, 대용량 파일을 식별할 때 사용합니다.
allowed-tools: Bash(python3 *)
---

# Codebase Visualizer

접을 수 있는 디렉터리로 프로젝트의 파일 구조를 보여주는 대화형 HTML 트리 뷰를 생성합니다.

## 사용법

프로젝트 루트에서 시각화 스크립트를 실행하세요:

```bash
python3 ${CLAUDE_SKILL_DIR}/scripts/visualize.py .
```

이렇게 하면 현재 디렉터리에 `codebase-map.html`이 생성되고 기본 브라우저에서 열립니다.

## 시각화가 보여주는 내용

- **접이식 디렉터리**: 폴더를 클릭하여 펼치기/접기
- **파일 크기**: 각 파일 옆에 표시됨
- **색상**: 다양한 파일 유형에 대한 서로 다른 색상
- **디렉터리 합계**: 각 폴더의 집계 크기 표시
````

이를 `~/.claude/skills/codebase-visualizer/scripts/visualize.py`에 저장하세요. 이 스크립트는 디렉터리 트리를 스캔하고 다음을 포함하는 자체 완결형 HTML 파일을 생성합니다:

* 파일 수, 디렉터리 수, 전체 크기 및 파일 유형 수를 보여주는 **요약 사이드바**
* 파일 유형별(크기 기준 상위 8개)로 코드베이스를 분석하는 **막대 차트**
* 색상으로 구분된 파일 유형 표시기와 함께 디렉터리를 펼치고 접을 수 있는 **접이식 트리**

이 스크립트는 Python 3이 필요하지만 내장 라이브러리만 사용하므로 설치할 패키지가 없습니다:

```python expandable theme={null}
#!/usr/usr/bin/env python3
"""Generate an interactive collapsible tree visualization of a codebase."""

import json
import sys
import webbrowser
from html import escape
from pathlib import Path
from collections import Counter

IGNORE = {'.git', 'node_modules', '__pycache__', '.venv', 'venv', 'dist', 'build'}

def scan(path: Path, stats: dict) -> dict:
    result = {"name": path.name, "children": [], "size": 0}
    try:
        for item in sorted(path.iterdir()):
            if item.name in IGNORE or item.name.startswith('.'):
                continue
            if item.is_file():
                size = item.stat().st_size
                ext = item.suffix.lower() or '(no ext)'
                result["children"].append({"name": item.name, "size": size, "ext": ext})
                result["size"] += size
                stats["files"] += 1
                stats["extensions"][ext] += 1
                stats["ext_sizes"][ext] += size
            elif item.is_dir():
                stats["dirs"] += 1
                child = scan(item, stats)
                if child["children"]:
                    result["children"].append(child)
                    result["size"] += child["size"]
    except PermissionError:
        pass
    return result

def generate_html(data: dict, stats: dict, output: Path) -> None:
    ext_sizes = stats["ext_sizes"]
    total_size = sum(ext_sizes.values()) or 1
    sorted_exts = sorted(ext_sizes.items(), key=lambda x: -x[1])[:8]
    colors = {
        '.js': '#f7df1e', '.ts': '#3178c6', '.py': '#3776ab', '.go': '#00add8',
        '.rs': '#dea584', '.rb': '#cc342d', '.css': '#264de4', '.html': '#e34c26',
        '.json': '#6b7280', '.md': '#083fa1', '.yaml': '#cb171e', '.yml': '#cb171e',
        '.mdx': '#083fa1', '.tsx': '#3178c6', '.jsx': '#61dafb', '.sh': '#4eaa25',
    }
    lang_bars = "".join(
        f'<div class="bar-row"><span class="bar-label">{ext}</span>'
        f'<div class="bar" style="width:{(size/total_size)*100}%;background:{colors.get(ext,"#6b7280")}"></div>'
        f'<span class="bar-pct">{(size/total_size)*100:.1f}%</span></div>'
        for ext, size in sorted_exts
    )
    def fmt(b):
        if b < 1024: return f"{b} B"
        if b < 1048576: return f"{b/1024:.1f} KB"
        return f"{b/1048576:.1f} MB"

    html = f'''<!DOCTYPE html>
<html><head>
  <meta charset="utf-8"><title>Codebase Explorer</title>
  <style>
    body {{ font: 14px/1.5 system-ui, sans-serif; margin: 0; background: #1a1a2e; color: #eee; }}
    .container {{ display: flex; height: 100vh; }}
    .sidebar {{ width: 280px; background: #252542; padding: 20px; border-right: 1px solid #3d3d5c; overflow-y: auto; flex-shrink: 0; }}
    .main {{ flex: 1; padding: 20px; overflow-y: auto; }}
    h1 {{ margin: 0 0 10px 0; font-size: 18px; }}
    h2 {{ margin: 20px 0 10px 0; font-size: 14px; color: #888; text-transform: uppercase; }}
    .stat {{ display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #3d3d5c; }}
    .stat-value {{ font-weight: bold; }}
    .bar-row {{ display: flex; align-items: center; margin: 6px 0; }}
    .bar-label {{ width: 55px; font-size: 12px; color: #aaa; }}
    .bar {{ height: 18px; border-radius: 3px; }}
    .bar-pct {{ margin-left: 8px; font-size: 12px; color: #666; }}
    .tree {{ list-style: none; padding-left: 20px; }}
    details {{ cursor: pointer; }}
    summary {{ padding: 4px 8px; border-radius: 4px; }}
    summary:hover {{ background: #2d2d44; }}
    .folder {{ color: #ffd700; }}
    .file {{ display: flex; align-items: center; padding: 4px 8px; border-radius: 4px; }}
    .file:hover {{ background: #2d2d44; }}
    .size {{ color: #888; margin-left: auto; font-size: 12px; }}
    .dot {{ width: 8px; height: 8px; border-radius: 50%; margin-right: 8px; }}
  </style>
</head><body>
  <div class="container">
    <div class="sidebar">
      <h1>📊 Summary</h1>
      <div class="stat"><span>Files</span><span class="stat-value">{stats["files"]:,}</span></div>
      <div class="stat"><span>Directories</span><span class="stat-value">{stats["dirs"]:,}</span></div>
      <div class="stat"><span>Total size</span><span class="stat-value">{fmt(data["size"])}</span></div>
      <div class="stat"><span>File types</span><span class="stat-value">{len(stats["extensions"])}</span></div>
      <h2>By file type</h2>
      {lang_bars}
    </div>
    <div class="main">
      <h1>📁 {escape(data["name"])}</h1>
      <ul class="tree" id="root"></ul>
    </div>
  </div>
  <script>
    const data = {json.dumps(data)};
    const colors = {json.dumps(colors)};
    function fmt(b) {{ if (b < 1024) return b + ' B'; if (b < 1048576) return (b/1024).toFixed(1) + ' KB'; return (b/1048576).toFixed(1) + ' MB'; }}
    function esc(s) {{ return s.replace(/[&<>"']/g, c => ({{"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}}[c])); }}
    function render(node, parent) {{
      if (node.children) {{
        const det = document.createElement('details');
        det.open = parent === document.getElementById('root');
        det.innerHTML = `<summary><span class="folder">📁 ${{esc(node.name)}}</span><span class="size">${{fmt(node.size)}}</span></summary>`;
        const ul = document.createElement('ul'); ul.className = 'tree';
        node.children.sort((a,b) => (b.children?1:0)-(a.children?1:0) || a.name.localeCompare(b.name));
        node.children.forEach(c => render(c, ul));
        det.appendChild(ul);
        const li = document.createElement('li'); li.appendChild(det); parent.appendChild(li);
      }} else {{
        const li = document.createElement('li'); li.className = 'file';
        li.innerHTML = `<span class="dot" style="background:${{colors[node.ext]||'#6b7280'}}"></span>${{esc(node.name)}}<span class="size">${{fmt(node.size)}}</span>`;
        parent.appendChild(li);
      }}
    }}
    data.children.forEach(c => render(c, document.getElementById('root')));
  </script>
</body></html>'''
    output.write_text(html)

if __name__ == '__main__':
    target = Path(sys.argv[1] if len(sys.argv) > 1 else '.').resolve()
    stats = {"files": 0, "dirs": 0, "extensions": Counter(), "ext_sizes": Counter()}
    data = scan(target, stats)
    out = Path('codebase-map.html')
    generate_html(data, stats, out)
    print(f'Generated {out.absolute()}')
    webbrowser.open(f'file://{out.absolute()}')
```

테스트하려면 임의의 프로젝트에서 Claude Code를 열고 "이 코드베이스를 시각화해 줘"라고 요청하세요. Claude는 스크립트를 실행하고 `Generated /path/to/codebase-map.html`과 같이 생성된 파일 경로를 출력하고 브라우저에서 엽니다. 브라우저가 열리지 않는 헤드리스 환경에서 작업하는 경우 출력된 경로로 스크립트 성공 여부를 확인합니다.

이 패턴은 종속성 그래프, 테스트 커버리지 리포트, API 문서 또는 데이터베이스 스키마 시각화 등 모든 시각적 출력에 작동합니다. 번들된 스크립트가 작업을 처리하고 Claude는 조율을 담당합니다.

## 문제 해결

### 스킬이 트리거되지 않음

기대한 시점에 Claude가 스킬을 사용하지 않는 경우:

1. 설명에 사용자가 자연스럽게 사용할 키워드가 포함되어 있는지 확인하세요.
2. `What skills are available?`에 스킬이 나타나는지 확인하세요.
3. 설명과 더 밀접하게 일치하도록 요청 문구를 재구성해 보세요.
4. 스킬이 사용자 호출 가능한 경우 `/skill-name`으로 직접 호출하세요.

프론트매터 YAML의 형식이 잘못된 경우 Claude Code는 빈 메타데이터로 스킬 본문을 로드하므로 `/skill-name`은 계속 작동하지만 Claude가 매칭할 `description`이 없게 됩니다. 파싱 오류를 확인하려면 `--debug`로 실행하세요.

### 스킬이 너무 자주 트리거됨

원치 않을 때 Claude가 스킬을 사용하는 경우:

1. 설명을 더 구체적으로 작성하세요.
2. 수동 호출만 원하는 경우 `disable-model-invocation: true`를 추가하세요.

### 스킬 설명이 잘림

Claude Code는 사용 가능한 항목을 Claude가 알 수 있도록 스킬 이름 및 설명 목록을 컨텍스트로 로드합니다. 목록에는 항상 모든 스킬 이름이 포함되지만 스킬이 많은 경우 Claude Code는 목록의 문자열 예산에 맞추기 위해 설명을 단축하므로 Claude가 요청과 매칭하는 데 필요한 키워드가 잘릴 수 있습니다. 예산은 모델 컨텍스트 창의 1% 수준으로 조정됩니다. 목록이 오버플로되면 Claude Code는 가장 적게 호출한 스킬부터 설명을 삭제하므로 가장 많이 사용하는 스킬은 전체 텍스트를 유지합니다.

목록의 컨텍스트 비용과 가장 기여도가 높은 항목을 추정하려면 `/doctor`를 실행하세요. 목록이 예산을 초과하면 Claude Code는 [`--debug`](/docs/en/cli-reference#cli-flags)로 확인할 수 있는 디버그 로그에도 경고를 기록합니다.

`/context`에 있는 Skills 행은 예산이 적용된 후의 목록 크기를 보고하므로 모델이 수신하는 크기와 일치합니다. v2.1.196 이전에는 이 행이 모든 설명의 전체 텍스트를 카운트하여 구성된 예산보다 몇 배 더 큰 값을 표시할 수 있었습니다.

예산을 늘리려면 [`skillListingBudgetFraction`](/docs/en/settings#available-settings) 설정(예: `0.02` = 2%) 또는 고정 문자열 수로 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 환경 변수를 설정하세요. 다른 스킬을 위한 예산을 확보하려면 우선순위가 낮은 항목을 [`skillOverrides`](#설정에서-스킬-가시성-재정의)에서 `"name-only"`로 설정하여 설명 없이 목록에 노출되도록 하세요. 또한 소스에서 `description` 및 `when_to_use` 텍스트를 다듬을 수 있습니다: 예산과 무관하게 각 항목의 결합 텍스트가 1,536자로 제한되므로 주요 사용 사례를 먼저 작성하세요. 이 제한은 [`skillListingMaxDescChars`](/docs/en/settings#available-settings)로 구성할 수 있습니다.

## 관련 리소스

* **[구성 디버깅](/docs/en/debug-your-config)**: 스킬이 표시되지 않거나 트리거되지 않는 이유 진단
* **[Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)**: agentskills.io의 평가 파일 형식 및 반복 워크플로
* **[Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)**: Claude 제품 전반에 적용되는 작성 지침
* **[서브에이전트](/docs/en/sub-agents)**: 특화된 에이전트에 작업 위임
* **[플러그인](/docs/en/plugins)**: 다른 확장과 함께 스킬 패키징 및 배포
* **[훅](/docs/en/hooks)**: 도구 이벤트 주변 워크플로 자동화
* **[메모리](/docs/en/memory)**: 지속적 컨텍스트를 위해 CLAUDE.md 파일 관리
* **[명령어](/docs/en/commands)**: 내장 명령 및 번들 스킬 참조
* **[권한](/docs/en/permissions)**: 도구 및 스킬 접근 제어
* **[Claude Tag skills](https://claude.com/docs/claude-tag/admins/skills-repo)**: 리포지토리에 커밋된 프로젝트 스킬은 해당 리포지토리가 Claude Tag 채널에서 사용될 때도 로드됨
