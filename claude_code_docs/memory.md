> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude가 프로젝트를 기억하는 방식

> CLAUDE.md 파일로 Claude에게 지속적인 지침을 제공하고, 자동 메모리(auto memory)를 통해 Claude가 학습 내용을 자동으로 축적하도록 하세요.

각 Claude Code 세션은 깨끗한 컨텍스트 창(context window)으로 시작됩니다. 두 가지 메커니즘이 세션 전체에 걸쳐 지식을 지속시킵니다:

* **CLAUDE.md 파일**: Claude에게 지속적인 컨텍스트를 제공하기 위해 직접 작성하는 지침
* **자동 메모리(Auto memory)**: 사용자의 수정 및 선호도를 바탕으로 Claude가 스스로 작성하는 메모

이 페이지에서는 다음 내용을 다룹니다:

* [CLAUDE.md 파일 작성 및 구성하기](#claude-md-files)
* `.claude/rules/`를 활용해 [특정 파일 유형에 규칙 적용 범위 지정하기](#organize-rules-with-claude/rules/)
* Claude가 자동으로 메모를 작성하도록 [자동 메모리 구성하기](#auto-memory)
* 지침이 잘 적용되지 않을 때 [문제 해결하기](#troubleshoot-memory-issues)

## CLAUDE.md vs 자동 메모리

Claude Code에는 보완 관계를 이루는 두 가지 메모리 시스템이 있습니다. 두 시스템 모두 모든 대화가 시작될 때 로드됩니다. Claude는 이를 강제된 설정이 아닌 컨텍스트로 취급합니다. Claude의 결정과 무관하게 어떠한 동작을 차단하려면 대신 [PreToolUse 훅](/docs/en/hooks-guide)을 사용하세요. 지침이 구체적이고 간결할수록 Claude가 더 일관되게 지침을 준수합니다.

|                      | CLAUDE.md 파일                                    | 자동 메모리 (Auto memory)                                        |
| :------------------- | :------------------------------------------------ | :--------------------------------------------------------------- |
| **작성 주체**        | 사용자                                            | Claude                                                           |
| **포함 내용**        | 지침 및 규칙                                      | 학습한 내용 및 패턴                                              |
| **적용 범위**        | 프로젝트, 사용자 또는 조직                        | 저장소별 (작업 트리 간 공유됨)                                   |
| **로드 위치**        | 모든 세션                                         | 모든 세션 (처음 200줄 또는 25KB)                                 |
| **사용 목적**        | 코딩 표준, 워크플로, 프로젝트 아키텍처            | 빌드 명령, 디버깅 통찰, Claude가 찾아낸 선호 사항               |

Claude의 동작을 가이드하고자 할 때는 CLAUDE.md 파일을 사용하세요. 자동 메모리를 사용하면 수동 작업 없이도 사용자의 수정을 통해 Claude가 스스로 학습하게 됩니다.

Subagent 또한 자체 자동 메모리를 유지할 수 있습니다. 자세한 내용은 [subagent 구성](/docs/en/sub-agents#enable-persistent-memory)을 참조하세요.

## CLAUDE.md 파일

CLAUDE.md 파일은 프로젝트, 개인 워크플로, 또는 전체 조직에 대한 지속적인 지침을 Claude에게 제공하는 마크다운 파일입니다. 사용자는 이 파일을 일반 텍스트로 작성하며, Claude는 모든 세션 시작 시 이 파일을 읽습니다.

### CLAUDE.md에 추가해야 하는 경우

반복해서 다시 설명할 필요가 있는 내용들을 작성하는 공간으로 CLAUDE.md를 활용하세요. 다음과 같은 경우에 추가합니다:

* Claude가 동일한 수정을 두 번째 반복할 때
* 코드 리뷰에서 Claude가 이 코드베이스에 대해 알고 있었어야 할 사항이 지적되었을 때
* 지난 세션에 입력했던 것과 동일한 수정 및 보완사항을 채팅에 다시 입력하고 있을 때
* 신규 팀원이 생산성을 내기 위해 동일한 컨텍스트가 필요할 때

빌드 명령, 규칙, 프로젝트 레이아웃, "항상 X를 수행할 것"과 같은 규칙 등 Claude가 모든 세션에서 유지해야 하는 사실에 집중하세요. 항목이 다단계 절차이거나 코드베이스의 특정 부분에만 적용되는 내용이라면 [스킬(skill)](/docs/en/skills)이나 [경로 범위 지정 규칙](#organize-rules-with-claude/rules/)으로 옮겨 관리하세요. 각 메커니즘을 사용하는 시점에 대해서는 [확장 개요](/docs/en/features-overview#build-your-setup-over-time)를 참조하세요.

### CLAUDE.md 파일 위치 선택

CLAUDE.md 파일은 서로 다른 스코프를 가진 여러 위치에 배치될 수 있습니다. 아래 표는 가장 넓은 스코프부터 가장 구체적인 스코프 순서로 로드되는 순서를 나열하므로, 프로젝트 지침은 컨텍스트 상에서 사용자 지침 뒤에 표시됩니다.

| 스코프                   | 위치                                                                                                                                                                    | 목적                                                       | 사용 사례 예시                                                       | 공유 대상                       |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------- |
| **관리형 정책**          | • macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br />• Linux 및 WSL: `/etc/claude-code/CLAUDE.md`<br />• Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`  | IT/DevOps에서 관리하는 조직 전체 지침                      | 사내 코딩 표준, 보안 정책, 컴플라이언스 요구사항                     | 조직 내 모든 사용자             |
| **사용자 지침**          | `~/.claude/CLAUDE.md`                                                                                                                                                   | 모든 프로젝트에 적용되는 개인 선호 사항                    | 코드 스타일 선호도, 개인 도구 단축키                                 | 본인만 (모든 프로젝트)          |
| **프로젝트 지침**        | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md`                                                                                                                                | 프로젝트를 위해 팀이 공유하는 지침                        | 프로젝트 아키텍처, 코딩 표준, 공통 워크플로                          | 소스 제어를 통해 팀원들과 공유  |
| **로컬 지침**            | `./CLAUDE.local.md`                                                                                                                                                     | 개인적이고 프로젝트에 특화된 선호 사항 (`.gitignore`에 추가)| 본인의 샌드박스 URL, 선호하는 테스트 데이터                          | 본인만 (현재 프로젝트)          |

작업 디렉토리 상위의 디렉토리 계층에 있는 CLAUDE.md 및 CLAUDE.local.md 파일은 시작 시 전체가 로드됩니다. 하위 디렉토리의 파일은 Claude가 해당 디렉토리 내의 파일을 읽을 때 요청 시 로드됩니다. 전체 로드 순서는 [CLAUDE.md 파일 로드 방식](#how-claude-md-files-load)을 참조하세요.

대규모 프로젝트의 경우 [프로젝트 규칙](#organize-rules-with-claude/rules/)을 사용하여 지침을 주제별 파일로 분할할 수 있습니다. 규칙을 활용하면 특정 파일 유형이나 하위 디렉토리로 지침의 적용 범위를 제한할 수 있습니다.

### 프로젝트 CLAUDE.md 설정하기

프로젝트 CLAUDE.md는 `./CLAUDE.md` 또는 `./.claude/CLAUDE.md`에 저장할 수 있습니다. 이 파일을 생성하고 프로젝트에서 작업하는 모든 사람에게 적용되는 지침(빌드 및 테스트 명령, 코딩 표준, 아키텍처 결정 사항, 명명 규칙, 공통 워크플로 등)을 추가하세요. 이 지침은 버전 관리를 통해 팀원들과 공유되므로 개인적 선호보다는 프로젝트 수준의 표준에 집중하세요. 파일이 로드되었는지 확인하려면 세션에서 `/context`를 실행하고 **Memory files** 아래의 목록을 검사하세요.

<Tip>
  `/init`을 실행하여 기본 CLAUDE.md를 자동으로 생성하세요. Claude가 코드베이스를 분석하고 자신이 발견한 빌드 명령, 테스트 지침, 프로젝트 규칙이 담긴 파일을 생성합니다. CLAUDE.md가 이미 존재하는 경우 `/init`은 파일에 대한 개선 사항을 제안합니다. 거기서부터 Claude가 스스로 파악하지 못할 지침들을 덧붙여 다듬어 나가세요.

  대화형 다단계 흐름을 활성화하려면 `CLAUDE_CODE_NEW_INIT=1`을 설정하세요. `/init`이 생성할 아티팩트(CLAUDE.md 파일, 스킬, 훅 등)를 묻습니다. 그런 다음 subagent로 코드베이스를 탐색하고, 추후 질문으로 부족한 부분을 채우며, 파일을 쓰기 전에 검토 가능한 제안을 제시합니다.
</Tip>

### 효과적인 지침 작성하기

CLAUDE.md 파일은 모든 세션 시작 시 컨텍스트 창에 로드되며 대화 내용과 함께 토큰을 소비합니다. [컨텍스트 창 시각화](/docs/en/context-window) 문서에서 시작 시 컨텍스트의 나머지 부분 대비 CLAUDE.md가 로드되는 위치를 확인할 수 있습니다. 강제 적용되는 설정이라기보다 컨텍스트에 해당하므로, 지침을 작성하는 방식이 Claude의 규칙 준수 유효성에 영향을 줍니다. 구체적이고 간결하며 잘 구조화된 지침이 가장 잘 작동합니다.

**크기**: CLAUDE.md 파일당 200줄 미만을 목표로 하세요. 파일이 길어지면 더 많은 컨텍스트를 소비하고 지침 준수율이 저하됩니다. 지침이 점차 커지면 [경로 범위 지정 규칙](#path-specific-rules)을 활용하여 일치하는 파일로 작업할 때만 지침이 로드되도록 하세요. 조직화를 위해 콘텐츠를 [임포트(imports)](#import-additional-files)로 분할할 수도 있지만, 임포트된 파일 역시 시작 시 로드되어 컨텍스트 창에 들어갑니다.

**구조**: 관련 지침을 그룹화하기 위해 마크다운 헤더와 글머리 기호를 사용하세요. Claude는 독자가 읽는 것과 동일한 방식으로 구조를 스캔합니다: 정리된 섹션이 빽빽한 단락보다 따르기 쉽습니다.

**구체성**: 검증 가능한 구체적인 지침을 작성하세요. 예를 들어:

* "코드를 올바르게 포맷할 것" 대신 "2공백 들여쓰기(2-space indentation) 사용"
* "변경 사항을 테스트할 것" 대신 "커밋 전 `npm test` 실행"
* "파일을 잘 정리할 것" 대신 "API 핸들러는 `src/api/handlers/`에 위치함"

**일관성**: 두 규칙이 서로 충돌하면 Claude가 임의로 하나를 선택할 수 있습니다. 주기적으로 CLAUDE.md 파일, 하위 디렉토리의 중첩된 CLAUDE.md 파일, [`.claude/rules/`](#organize-rules-with-claude/rules/)을 검토하여 오래되었거나 충돌하는 지침을 제거하세요. 모노레포에서는 [`claudeMdExcludes`](#exclude-specific-claude-md-files)를 사용하여 자신의 작업과 무관한 다른 팀의 CLAUDE.md 파일을 건너뛰도록 설정할 수 있습니다.

### 추가 파일 임포트하기

CLAUDE.md 파일은 `@path/to/import` 구문을 사용하여 추가 파일을 임포트할 수 있습니다. 임포트된 파일은 이를 참조하는 CLAUDE.md와 함께 시작 시 펼쳐져 컨텍스트에 로드됩니다.

상대 경로와 절대 경로가 모두 허용됩니다. 상대 경로는 작업 디렉토리가 아닌 임포트문이 포함된 파일을 기준 경로로 해석됩니다. 임포트된 파일은 최대 4단계 깊이까지 다른 파일을 재귀적으로 임포트할 수 있습니다.

임포트 파싱은 마크다운 코드 스팬 및 코드 블록을 건너뜁니다. 파일 내용을 임포트하지 않고 CLAUDE.md에서 경로를 언급하려면 백틱으로 감싸세요: `` `@README` ``와 같이 작성하면 문자로 유지되고, 백틱 없는 `@README`는 파일을 임포트합니다.

README, package.json 및 워크플로 가이드를 불러오려면 CLAUDE.md 내 임의의 위치에서 `@` 구문으로 이를 참조하세요:

```text theme={null}
See @README for project overview and @package.json for available npm commands for this project.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

버전 관리에 커밋되면 안 되는 비공개 프로젝트별 선호 사항의 경우 프로젝트 루트에 `CLAUDE.local.md`를 생성하세요. `CLAUDE.md`와 함께 로드되며 동일하게 취급됩니다. 커밋되지 않도록 `.gitignore`에 `CLAUDE.local.md`를 추가하세요. `CLAUDE_CODE_NEW_INIT=1`을 설정한 상태에서 `/init`을 실행하고 개인(personal) 옵션을 선택하면 이 작업이 자동으로 처리됩니다.

동일한 저장소의 여러 git worktree에 걸쳐 작업하는 경우, gitignore 처리된 `CLAUDE.local.md`는 해당 파일을 생성한 worktree에만 존재합니다. 여러 worktree 간에 개인 지침을 공유하려면 대신 홈 디렉토리의 파일을 임포트하세요:

```text theme={null}
# Individual Preferences
- @~/.claude/my-project-instructions.md
```

<Warning>
  프로젝트 수준 메모리 파일 내의 임포트 경로가 위 홈 디렉토리 임포트처럼 작업 디렉토리 외부를 가리키면 외부 임포트(external import)로 처리됩니다. Claude Code가 프로젝트에서 외부 임포트를 처음 만나면 해당 파일들을 나열하는 승인 대화 상자를 띄웁니다. 거부하면 임포트가 비활성화된 상태로 유지되며 대화 상자가 다시 나타나지 않습니다.

  이 대화 상자는 공유 프로젝트에 다른 사람이 커밋한 파일로부터 사용자를 보호하기 위해 존재합니다. `~/.claude/CLAUDE.md` 및 `~/.claude/rules/`와 같은 사용자 스코프 메모리 파일의 임포트는 직접 작성한 파일이므로 대화 상자 없이 로드되며 개인 구성의 나머지 항목과 동일한 신뢰를 가집니다.
</Warning>

지침을 정리하는 보다 구조화된 접근 방식에 대해서는 [`.claude/rules/`](#organize-rules-with-claude/rules/)을 참조하세요.

### AGENTS.md

Claude Code는 `AGENTS.md`가 아닌 `CLAUDE.md`를 읽습니다. 저장소가 다른 코딩 에이전트를 위해 이미 `AGENTS.md`를 사용하고 있다면, `AGENTS.md`를 임포트하는 `CLAUDE.md`를 생성하여 중복 없이 두 도구가 동일한 지침을 읽도록 만드세요. 임포트문 아래에 Claude 전용 지침을 덧붙일 수도 있습니다. Claude는 세션 시작 시 임포트된 파일을 로드한 다음 나머지를 덧붙입니다:

```markdown CLAUDE.md theme={null}
@AGENTS.md

## Claude Code

Use plan mode for changes under `src/billing/`.
```

Claude 전용 내용을 추가할 필요가 없다면 심볼릭 링크(symlink)도 작동합니다:

```bash theme={null}
ln -s AGENTS.md CLAUDE.md
```

성공 시 명령은 아무런 출력을 내지 않습니다. 다음 세션에서 `/context`를 실행하여 **Memory files** 아래에 `CLAUDE.md`가 나타나는지 확인하세요.

Windows에서 심볼릭 링크를 생성하려면 관리자 권한이나 개발자 모드가 필요하므로 대신 `@AGENTS.md` 임포트를 사용하세요.

[`/init`](/docs/en/commands)을 실행하면 `.cursor/rules/` 또는 `.cursorrules`에 있는 Cursor 규칙과 `.github/copilot-instructions.md`에 있는 Copilot 규칙을 읽고 관련 항목을 생성된 `CLAUDE.md`에 포함시킵니다. `CLAUDE_CODE_NEW_INIT=1`을 설정한 경우 `/init`은 `AGENTS.md`, `.devin/rules/`, `.windsurf/rules/` 또는 `.windsurfrules`, `.clinerules`도 함께 읽습니다.

### CLAUDE.md 파일 로드 방식

Claude Code는 현재 작업 디렉토리에서부터 시작하여 상위 디렉토리 트리를 순회하면서 각 디렉토리의 `CLAUDE.md` 및 `CLAUDE.local.md` 파일을 확인하여 읽습니다. 즉, `foo/bar/`에서 Claude Code를 실행하면 `foo/bar/CLAUDE.md`, `foo/CLAUDE.md` 및 이들 옆의 `CLAUDE.local.md` 파일들의 지침이 모두 로드됩니다.

탐색된 모든 파일은 서로를 덮어쓰지 않고 컨텍스트에 연결(concatenation)됩니다. 디렉토리 트리를 통틀어 파일시스템 루트부터 현재 작업 디렉토리 순서대로 내용이 정렬됩니다. `foo/bar/` 예시의 경우, `foo/CLAUDE.md`가 `foo/bar/CLAUDE.md`보다 앞서 컨텍스트에 나타나므로 Claude를 실행한 위치에 가까운 지침이 마지막에 읽히게 됩니다. 각 디렉토리 내부에서는 `CLAUDE.local.md`가 `CLAUDE.md` 뒤에 덧붙여지므로 사용자의 개인 메모가 해당 수준에서 마지막으로 읽히는 내용이 됩니다.

Claude는 현재 작업 디렉토리 아래의 하위 디렉토리에 있는 `CLAUDE.md` 및 `CLAUDE.local.md` 파일도 탐색합니다. 시작 시 로드되는 대신, Claude가 해당 하위 디렉토리의 파일을 읽을 때 포함됩니다.

다른 팀의 CLAUDE.md 파일이 검색되는 대규모 모노레포에서 작업하는 경우 [`claudeMdExcludes`](#exclude-specific-claude-md-files)를 사용하여 해당 파일들을 건너뛰도록 설정하세요. 루트 및 디렉토리별 CLAUDE.md 파일과 규칙의 전체 레이아웃은 [모노레포 및 대규모 저장소](/docs/en/large-codebases)를 참조하세요.

CLAUDE.md 파일 내의 블록 수준 HTML 주석(`<!-- maintainer notes -->`)은 Claude의 컨텍스트에 주입되기 전에 제거됩니다. 컨텍스트 토큰을 소비하지 않고 인간 유지 관리자를 위한 메모를 남길 때 사용하세요. 코드 블록 내부의 주석은 보존됩니다. Read 도구로 CLAUDE.md 파일을 직접 열면 주석이 계속 표시됩니다.

#### 추가 디렉토리로부터 로드

`--add-dir` 플래그는 기본 작업 디렉토리 외부의 추가 디렉토리에 대해 Claude에게 접근 권한을 부여합니다. 기본적으로 이러한 디렉토리의 CLAUDE.md 파일은 로드되지 않습니다.

추가 디렉토리에서도 메모리 파일을 로드하려면 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` 환경 변수를 설정하세요:

```bash theme={null}
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

이렇게 하면 추가 디렉토리에서 `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md`, 및 `CLAUDE.local.md`가 로드됩니다. [`--setting-sources`](/docs/en/cli-reference)에서 `local`을 제외하면 `CLAUDE.local.md`는 건너뜁니다.

### `.claude/rules/`로 규칙 정리하기

대규모 프로젝트의 경우 `.claude/rules/` 디렉토리를 활용하여 지침을 여러 파일로 나누어 정리할 수 있습니다. 이를 통해 지침을 모듈식으로 유지하고 팀이 관리하기 용이하게 만들 수 있습니다. 규칙은 [특정 파일 경로로 적용 범위를 지정](#path-specific-rules)할 수도 있어 Claude가 일치하는 파일로 작업할 때만 컨텍스트에 로드되므로 잡음을 줄이고 컨텍스트 공간을 절약할 수 있습니다.

<Note>
  규칙은 매 세션마다 또는 일치하는 파일이 열릴 때 컨텍스트에 로드됩니다. 항상 컨텍스트에 남아 있을 필요가 없는 작업별 지침의 경우, 직접 호출하거나 Claude가 프롬프트와 관련이 있다고 판단할 때만 로드되는 [스킬(skills)](/docs/en/skills)을 대신 사용하세요.
</Note>

#### 규칙 설정하기

프로젝트의 `.claude/rules/` 디렉토리에 마크다운 파일을 위치시키세요. 각 파일은 `testing.md` 또는 `api-design.md`와 같은 설명적인 파일명과 함께 하나의 주제를 다루어야 합니다. 모든 `.md` 파일은 재귀적으로 탐색되므로 `frontend/`나 `backend/`와 같은 하위 디렉토리로 규칙을 정리할 수 있습니다:

```text theme={null}
your-project/
├── .claude/
│   ├── CLAUDE.md           # 메인 프로젝트 지침
│   └── rules/
│       ├── code-style.md   # 코드 스타일 가이드라인
│       ├── testing.md      # 테스트 규칙
│       └── security.md     # 보안 요구사항
```

[`paths` 프론트매터](#path-specific-rules)가 없는 규칙은 `.claude/CLAUDE.md`와 동일한 우선순위로 시작 시 로드됩니다.

[`--setting-sources`](/docs/en/cli-reference)에서 `project`를 제외하면 프로젝트 규칙이 건너뛰어집니다. v2.1.211 이전에는 경로 범위 규칙 및 중첩된 `.claude/rules/` 디렉토리의 규칙을 포함하여 필요 시 로드되는 규칙들이 `project`가 제외되었을 때도 로드되었었습니다.

#### 경로 지정 규칙

`paths` 필드가 포함된 YAML 프론트매터를 사용하여 규칙의 적용 범위를 특정 파일로 제한할 수 있습니다. 이러한 조건부 규칙은 Claude가 지정된 패턴과 일치하는 파일로 작업할 때만 적용됩니다.

```markdown theme={null}
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All API endpoints must include input validation
- Use the standard error response format
- Include OpenAPI documentation comments
```

`paths` 필드가 없는 규칙은 조건 없이 로드되며 모든 파일에 적용됩니다. 경로 범위 규칙은 모든 도구 사용 시점이 아니라 Claude가 패턴과 일치하는 파일을 읽을 때 트리거됩니다. v2.1.198부터는 심볼릭 링크 첵아웃(checkout) 환경에서처럼 프로젝트 디렉토리로 연결된 심볼릭 링크 경로를 통해 Claude가 파일에 도달할 때도 매칭이 동작합니다.

`paths` 필드에서 글로브(glob) 패턴을 사용하여 확장자, 디렉토리 또는 임의의 조합으로 파일을 매칭하세요:

| 패턴                   | 매칭 대상                                |
| ---------------------- | ---------------------------------------- |
| `**/*.ts`              | 모든 디렉토리의 모든 TypeScript 파일     |
| `src/**/*`             | `src/` 디렉토리 아래의 모든 파일         |
| `*.md`                 | 프로젝트 루트의 마크다운 파일            |
| `src/components/*.tsx` | 특정 디렉토리의 React 컴포넌트           |

여러 패턴을 지정할 수 있으며 브레이스 확장(brace expansion)을 사용하여 단일 패턴에서 여러 확장자를 매칭할 수 있습니다:

```markdown theme={null}
---
paths:
  - "src/**/*.{ts,tsx}"
  - "lib/**/*.ts"
  - "tests/**/*.test.ts"
---
```

각 브레이스 그룹은 확장되는 패턴 수를 곱합니다: `src/*.{ts,tsx}`는 2개 패턴으로 확장되고, `{a,b}/{c,d}/*.{ts,tsx}`는 8개로 확장됩니다. 확장을 적절한 범위 내로 유지하기 위해 규칙의 전체 `paths` 목록은 1,000개의 확장 패턴 및 4MiB라는 하나의 예산을 공유하며, 브레이스가 없는 패턴은 예산 카운트에 포함되지 않습니다.

Claude Code는 예산을 초과하는 패턴은 확장되지 않은 상태로 사용하며, 해당 문자가 포함된 브레이스는 어떠한 파일과도 매칭되지 않습니다. v2.1.217 이전에는 브레이스 그룹이 많은 `paths` 값이 시작 시 CLI를 지연시키거나 충돌을 일으켰었습니다.

글로브 구문은 `[`를 `[abc]`와 같은 브래킷 표현식의 시작으로 처리합니다. `photos [2024/**`와 같이 브래킷 표현식으로 읽을 수 없는 `[`가 포함된 패턴은 유효하지 않으며: 아무것도 매칭하지 않되 규칙의 다른 패턴들은 계속 작동합니다. 파일 이름에서 `[` 문자 자체를 매칭하려면 `photos \[2024/**`와 같이 이스케이프 처리하세요. v2.1.207 이전에는 하나의 유효하지 않은 패턴으로 인해 아무것도 매칭하지 않는 대신 해당 규칙이 평가되는 모든 파일에 대해 Read 도구가 실패했었습니다.

#### 심볼릭 링크로 여러 프로젝트 간 규칙 공유

`.claude/rules/` 디렉토리는 심볼릭 링크를 지원하므로 shared 규칙 세트를 유지하면서 여러 프로젝트에 링크 걸어 사용할 수 있습니다. 심볼릭 링크는 정상적으로 해석되어 로드되며 순환 심볼릭 링크는 감지되어 적절히 처리됩니다.

이 예시는 공유 디렉토리와 개별 파일을 모두 링크합니다:

```bash theme={null}
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

#### 사용자 수준 규칙

`~/.claude/rules/`에 위치한 개인 규칙은 머신의 모든 프로젝트에 적용됩니다. 프로젝트에 특화되지 않은 선호 사항에 사용하세요:

```text theme={null}
~/.claude/rules/
├── preferences.md    # 개인 코딩 선호 사항
└── workflows.md      # 선호하는 워크플로
```

사용자 수준 규칙은 프로젝트 규칙보다 먼저 로드되므로 프로젝트 규칙에 더 높은 우선순위가 부여됩니다.

### 대규모 팀을 위한 CLAUDE.md 관리

팀 전체에 Claude Code를 배포하는 조직의 경우, 지침을 중앙화하고 로드되는 CLAUDE.md 파일을 제어할 수 있습니다.

#### 조직 전체 CLAUDE.md 배포

조직은 머신의 모든 사용자에게 적용되는 중앙 관리형 CLAUDE.md를 배포할 수 있습니다. 이 파일은 개별 설정으로 제외시킬 수 없습니다.

<Steps>
  <Step title="관리형 정책 위치에 파일 생성">
    * macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
    * Linux 및 WSL: `/etc/claude-code/CLAUDE.md`
    * Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`
  </Step>

  <Step title="구성 관리 시스템으로 배포">
    MDM, 그룹 정책, Ansible 또는 유사한 도구를 사용하여 개발자 머신에 파일을 배포하세요. 다른 조직 전체 구성 옵션은 [관리형 설정](/docs/en/permissions#managed-settings)을 참조하세요.
  </Step>
</Steps>

`claudeMd` 키를 사용하면 별도 파일을 배포하는 대신 `managed-settings.json` 내부에 관리형 CLAUDE.md 내용을 직접 넣을 수 있습니다.

**적용 범위**: 모든 저장소, 머신의 모든 Claude Code 세션. 저장소별 지침의 경우 대신 프로젝트 CLAUDE.md를 커밋하세요.

**우선순위**: 관리형 CLAUDE.md 파일과 동일. 사용자 및 프로젝트 CLAUDE.md보다 먼저 로드됨.

**적용 위치**: 관리형 및 정책 설정 전용. 사용자, 프로젝트, 또는 로컬 설정에 `claudeMd`를 설정하는 것은 아무런 효과가 없음.

아래 예시는 관리형 설정 파일에 직접 행동 지침을 추가합니다:

```json theme={null}
{
  "claudeMd": "Always run `make lint` before committing.\nNever push directly to main."
}
```

관리형 CLAUDE.md와 [관리형 설정](/docs/en/settings#settings-files)은 다른 목적을 가집니다. 기술적 강제에는 설정을 사용하고 행동 가이드에는 CLAUDE.md를 사용하세요:

| 관심 분야                                      | 구성 위치                                                 |
| :--------------------------------------------- | :-------------------------------------------------------- |
| 특정 도구, 명령 또는 파일 경로 차단            | 관리형 설정: `permissions.deny`                           |
| 샌드박스 격리 강제                             | 관리형 설정: `sandbox.enabled`                            |
| 환경 변수 및 API 프로바이더 라우팅             | 관리형 설정: `env`                                        |
| 인증 방식 및 조직 잠금                         | 관리형 설정: `forceLoginMethod`, `forceLoginOrgUUID`      |
| 코드 스타일 및 품질 가이드라인                 | 관리형 CLAUDE.md                                          |
| 데이터 처리 및 컴플라이언스 주의사항           | 관리형 CLAUDE.md                                          |
| Claude를 위한 행동 지침                        | 관리형 CLAUDE.md                                          |

설정 규칙은 Claude의 결정과 무관하게 클라이언트에 의해 강제 적용됩니다. CLAUDE.md 지침은 Claude의 동작을 형성하지만 엄격한 강제 계층은 아닙니다.

#### 특정 CLAUDE.md 파일 제외하기

대규모 모노레포에서는 상위 CLAUDE.md 파일에 본인 작업과 무관한 지침이 포함되어 있을 수 있습니다. `claudeMdExcludes` 설정을 사용하면 경로나 글로브 패턴으로 특정 파일을 건너뛸 수 있습니다.

이 예시는 상위 폴더에서 최상위 CLAUDE.md 및 규칙 디렉토리를 제외합니다. 예외 설정이 머신 로컬에 유지되도록 `.claude/settings.local.json`에 추가하세요:

```json theme={null}
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

패턴은 글로브 구문을 사용하여 절대 파일 경로에 대해 매칭됩니다. 모든 [설정 레이어](/docs/en/settings#settings-files)(사용자, 프로젝트, 로컬, 관리형 정책)에서 `claudeMdExcludes`를 구성할 수 있습니다. 배열은 레이어 간에 병합됩니다.

관리형 정책 CLAUDE.md 파일은 제외할 수 없습니다. 이를 통해 조직 전체 지침이 개별 설정에 관계없이 항상 적용되도록 보장합니다.

## 자동 메모리 (Auto memory)

자동 메모리를 사용하면 사용자가 아무것도 쓰지 않아도 Claude가 세션에 걸쳐 지식을 축적할 수 있습니다. Claude는 작업을 진행하면서 빌드 명령, 디버깅 통찰, 아키텍처 메모, 코드 스타일 선호도, 워크플로 습관 등을 스스로 메모로 저장합니다. Claude가 매 세션마다 메모를 저장하는 것은 아닙니다. 해당 정보가 향후 대화에서 유용할지 여부에 따라 기억할 가치가 있는지 결정합니다.

### 자동 메모리 활성화 및 비활성화

자동 메모리는 기본적으로 켜져 있습니다. 토글하려면 세션에서 `/memory`를 열고 자동 메모리 토글을 사용하세요. 이는 `~/.claude/settings.json`의 사용자 설정에 `autoMemoryEnabled`를 저장합니다. 단일 프로젝트에 대해 끄려면 해당 프로젝트의 설정에 `autoMemoryEnabled`를 설정하세요:

```json theme={null}
{
  "autoMemoryEnabled": false
}
```

환경 변수를 통해 자동 메모리를 비활성화하려면 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`을 설정하세요.

### 저장 위치

각 프로젝트는 `~/.claude/projects/<project>/memory/`에 자체 메모리 디렉토리를 가집니다. `<project>` 경로는 git 저장소에서 파생되므로 동일한 저장소 내의 모든 worktree와 하위 디렉토리는 하나의 자동 메모리 디렉토리를 공유합니다. git 저장소 외부에서는 프로젝트 루트가 대신 사용됩니다.

자동 메모리를 다른 위치에 저장하려면 `settings.json`에 `autoMemoryDirectory`를 설정하세요. 사용자, 프로젝트, 로컬, 정책 또는 `--settings` 등 임의의 [설정 스코프](/docs/en/settings#settings-precedence)에서 읽어옵니다.

```json theme={null}
{
  "autoMemoryDirectory": "~/my-custom-memory-dir"
}
```

값은 반드시 절대 경로이거나 `~/`로 시작해야 합니다. 프로젝트의 `.claude/settings.json` 또는 `.claude/settings.local.json`에 설정된 경우, 훅을 제어하는 문과 동일하게 해당 폴더에 대한 작업 공간 신뢰 대화 상자를 승인한 후에만 값이 적용됩니다.

디렉토리에는 `MEMORY.md` 엔트리포인트와 선택적 주제별 파일들이 포함됩니다:

```text theme={null}
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 모든 세션에 로드되는 간결한 인덱스
├── debugging.md       # 디버깅 패턴에 대한 상세 메모
├── api-conventions.md # API 설계 결정 사항
└── ...                # Claude가 생성하는 기타 주제 파일들
```

`MEMORY.md`는 메모리 디렉토리의 인덱스 역할을 합니다. Claude는 어디에 무엇이 저장되어 있는지 추적하기 위해 `MEMORY.md`를 활용하여 세션 전반에 걸쳐 이 디렉토리의 파일들을 읽고 씁니다.

자동 메모리는 머신 로컬에 보관됩니다. 동일한 git 저장소 내부의 모든 worktree와 하위 디렉토리는 하나의 자동 메모리 디렉토리를 공유합니다. 파일은 머신 간이나 클라우드 환경 간에 공유되지 않습니다.

### 작동 방식

처음 200줄 또는 25KB 중 먼저 도달하는 `MEMORY.md` 분량이 모든 대화 시작 시 로드됩니다. 해당 임계값을 초과하는 내용은 세션 시작 시 로드되지 않습니다. Claude는 상세한 메모를 별도의 주제별 파일로 옮김으로써 `MEMORY.md`를 간결하게 유지합니다.

Claude가 `MEMORY.md`에 작성한 후, Claude Code는 200줄 및 25KB 읽기 제한을 기준으로 파일을 측정합니다. 파일이 제한에 가까워지면 Claude Code가 Claude에게 내용을 축소하라고 상기시킵니다: 항목당 한 줄 유지, 세부 사항은 주제별 파일로 이동, 오래된 항목은 병합 또는 삭제. 파일이 제한을 초과하더라도 쓰기는 성공하지만, 다음 로드 시 제한을 초과한 모든 내용이 유실되므로 Claude Code가 [인덱스를 다시 작성하라는 오류](/docs/en/errors#memory-index-is-over-its-read-limit)를 반환합니다.

이 검사는 로드되는 내용만 측정합니다: YAML 프론트매터 및 블록 수준 HTML 주석은 인덱스가 로드되기 전에 제거되므로 제한 계산에 포함되지 않습니다. v2.1.211 이전에는 Claude Code가 원본 파일 자체를 측정하여 로드되는 내용이 기준에 들어맞더라도 프론트매터나 주석으로 인해 오류가 트리거될 수 있었습니다.

이 제한은 `MEMORY.md`에만 적용됩니다. CLAUDE.md 파일은 길이에 관계없이 전체가 로드되지만, 파일이 짧을수록 준수율이 높아집니다.

`debugging.md`나 `patterns.md`와 같은 주제별 파일은 시작 시 로드되지 않습니다. Claude는 정보가 필요할 때 표준 파일 도구를 사용하여 필요한 시점에 읽습니다.

메인 대화의 자동 메모리는 [subagent](/docs/en/sub-agents#what-loads-at-startup)에 로드되지 않습니다; 부모 대화와 시스템 프롬프트를 상속받는 [fork](/docs/en/sub-agents#fork-the-current-conversation)는 예외입니다. subagent `memory` 필드로 활성화되는 subagent의 자체 자동 메모리는 별도의 디렉토리입니다.

Claude는 세션 중에 메모리 파일을 읽고 씁니다. Claude Code 인터페이스에 "Saved 2 memories" 또는 "Recalled 2 memories"와 같은 메시지가 보이면 Claude가 `~/.claude/projects/<project>/memory/`에서 활발히 업데이트하거나 읽고 있는 것입니다.

Claude가 YAML 프론트매터로 시작하는 메모리 파일을 작성할 때, Claude Code는 작성 시간을 ISO 8601 타임스탬프 형태의 `modified` 프론트매터 필드에 기록합니다. 타임스탬프는 사용자 및 메모리를 다시 읽는 Claude 모두에게 정보가 얼마나 최신인지 보여줍니다. 구버전에서 생성된 파일을 포함하여 프론트매터가 있는 파일은 Claude가 다음에 작성할 때 해당 필드를 부여받게 됩니다; Claude Code는 프론트매터가 없는 파일에 프론트매터를 임의로 추가하지는 않습니다. `modified` 필드는 Claude Code v2.1.214 이상이 필요합니다.

### 메모리 감사 및 편집

자동 메모리 파일은 언제든지 편집하거나 삭제할 수 있는 일반 마크다운 파일입니다. 세션 내부에서 메모리 파일을 탐색하고 열려면 [`/memory`](#view-and-edit-with-%2Fmemory)를 실행하세요.

## `/memory`로 보기 및 편집

`/memory` 명령은 아직 존재하지 않는 파일에 대한 사용자 및 프로젝트 CLAUDE.md 항목을 포함하여, 사용자 및 프로젝트 스코프 전반의 CLAUDE.md, CLAUDE.local.md, 기타 메모리 파일 위치를 나열합니다. 또한 자동 메모리를 켜거나 끌 수 있는 토글을 제공하며 자동 메모리 폴더를 여는 옵션을 제공합니다. 임의의 파일을 선택하여 에디터에서 열 수 있으며; 아직 존재하지 않는 파일을 선택하면 먼저 파일이 생성됩니다. 현재 세션에 실제로 로드된 파일들을 확인하려면 `/context`를 실행하세요.

VS Code와 같은 GUI 에디터는 별도 창에서 파일을 열며, 파일이 열려 있는 동안에도 세션을 계속 이용할 수 있습니다. v2.1.216 이전에는 `/memory`가 응답하기 전에 파일이 닫힐 때까지 대기했었습니다. Vim과 같은 터미널 에디터는 종료할 때까지 터미널을 점유합니다.

Claude에게 "항상 npm이 아닌 pnpm을 사용할 것" 또는 "API 테스트에는 로컬 Redis 인스턴스가 필요함을 기억할 것"과 같이 무언가를 기억하도록 요청하면 Claude가 이를 자동 메모리에 저장합니다. 대신 CLAUDE.md에 지침을 추가하려면 "이 내용을 CLAUDE.md에 추가해줘"와 같이 직접 요청하거나 `/memory`를 통해 파일을 직접 편집하세요.

## 메모리 문제 해결

다음은 CLAUDE.md 및 자동 메모리와 관련해 가장 흔히 발생하는 문제들과 디버깅 단계입니다.

### Claude가 CLAUDE.md를 따르지 않음

CLAUDE.md 내용은 시스템 프롬프트 자체가 아니라 시스템 프롬프트 뒤의 사용자 메시지로 전달됩니다. Claude는 이를 읽고 따르려고 시도하지만, 특히 모호하거나 충돌하는 지침에 대해 엄격한 준수를 보증하지는 않습니다.

디버깅 단계:

* `/context`를 실행하고 **Memory files** 아래 목록을 확인하여 CLAUDE.md 및 CLAUDE.local.md 파일이 로드되었는지 검증하세요. 여기에 파일이 없다면 Claude가 볼 수 없는 상태입니다. `/memory`를 사용해 파일을 열고 편집하세요.
* 관련 CLAUDE.md가 세션에서 로드되는 위치에 있는지 확인하세요 ([CLAUDE.md 파일 위치 선택](#choose-where-to-put-claude-md-files) 참조).
* 지침을 더 구체적으로 만드세요. "코드를 예쁘게 포맷할 것"보다 "2공백 들여쓰기 사용"이 더 잘 작동합니다.
* CLAUDE.md 파일 간에 충돌하는 지침이 있는지 확인하세요. 두 파일이 동일한 동작에 대해 다른 가이드를 제공하면 Claude가 임의로 하나를 선택할 수 있습니다.

매 커밋 전이나 매 파일 편집 후와 같이 특정 시점에 반드시 실행되어야 하는 지침이라면, 대신 [훅(hook)](/docs/en/hooks-guide)으로 작성하세요. 훅은 고정된 라이프사이클 이벤트에서 셸 명령으로 실행되며 Claude의 결정과 상관없이 적용됩니다.

시스템 프롬프트 수준에서 제공하고 싶은 지침의 경우 [`--append-system-prompt`](/docs/en/cli-reference#system-prompt-flags)를 사용하세요. 호출 시마다 전달해야 하므로 인터랙티브 사용보다는 스크립트 및 자동화에 더 적합합니다.

<Tip>
  [`InstructionsLoaded` 훅](/docs/en/hooks#instructionsloaded)을 사용하여 어떤 지침 파일이 언제, 왜 로드되는지 정확히 기록하세요. 경로 지정 규칙이나 하위 디렉토리의 지연 로드 파일을 디버깅할 때 유용합니다.
</Tip>

### 자동 메모리에 무엇이 저장되었는지 모름

`/memory`를 실행하고 자동 메모리 폴더를 선택하여 Claude가 저장한 내용을 탐색하세요. 모든 내용은 읽고 편집하고 삭제할 수 있는 일반 마크다운입니다.

### CLAUDE.md가 너무 큼

200줄이 넘는 파일은 더 많은 컨텍스트를 소비하며 지침 준수율을 떨어뜨릴 수 있습니다. [경로 지정 규칙](#path-specific-rules)을 사용하여 Claude가 일치하는 파일로 작업할 때만 지침이 로드되도록 하거나, 매 세션에 필요하지 않은 내용을 잘라내세요. [`@path` 임포트](#import-additional-files)로 분할하는 것은 조직화에는 도움이 되지만 임포트된 파일이 시작 시 로드되므로 컨텍스트를 줄여주지는 못합니다.

[`/doctor`](/docs/en/commands#all-commands) 점검 기능은 커밋된 CLAUDE.md에 대한 다듬기를 제안합니다: 디렉토리 레이아웃, 의존성 목록, 아키텍처 개요와 같이 Claude가 코드베이스에서 파생할 수 있는 콘텐츠는 잘라내고, 함정, 이유, 도구 기본값과 다른 규칙 등은 유지합니다. 다듬기 검사는 Claude Code v2.1.206 이상이 필요합니다.

### `/compact` 후 지침이 사라진 것 같음

프로젝트 루트 CLAUDE.md는 압축(compaction) 후에도 유지됩니다: `/compact` 후 Claude가 디스크에서 이를 다시 읽고 세션에 다시 주입합니다. 하위 디렉토리의 중첩된 CLAUDE.md 파일은 자동으로 다시 주입되지 않으며; Claude가 해당 하위 디렉토리의 파일을 다음에 읽을 때 다시 로드됩니다.

압축 후 지침이 사라졌다면 대화 중에만 전달되었거나 아직 다시 로드되지 않은 중첩 CLAUDE.md에 위치해 있었던 것입니다. 계속 유지되도록 하려면 대화 전용 지침을 CLAUDE.md에 추가하세요. 전체적인 내용은 [압축 후 유지되는 항목](/docs/en/context-window#what-survives-compaction)을 참조하세요.

크기, 구조, 구체성에 대한 가이드는 [효과적인 지침 작성하기](#write-effective-instructions)를 참조하세요.

## 관련 리소스

* [구성 디버깅](/docs/en/debug-your-config): CLAUDE.md 또는 설정이 적용되지 않는 이유 진단
* [스킬 (Skills)](/docs/en/skills): 필요 시 로드되는 재사용 가능한 워크플로 패키징
* [설정 (Settings)](/docs/en/settings): 설정 파일로 Claude Code 동작 구성
* [Subagent 메모리](/docs/en/sub-agents#enable-persistent-memory): Subagent가 자체 자동 메모리를 유지하도록 허용
