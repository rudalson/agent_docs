> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 권한 구성 (Configure permissions)

> 세세한 권한 규칙, 모드 및 관리형 정책을 통해 Claude Code가 접근하고 수행할 수 있는 작업을 제어합니다.

Claude Code는 에이전트에 허용되는 작업과 허용되지 않는 작업을 정확하게 지정할 수 있도록 세분화된 권한을 지원합니다. 권한 설정은 버전 제어에 체크인하고 조직의 모든 개발자에게 배포할 수 있으며, 개별 개발자가 맞춤 설정할 수도 있습니다.

## 권한 시스템

Claude Code는 성능과 안전성의 균형을 유지하기 위해 계층화된 권한 시스템을 사용합니다:

| 도구 유형 | 예시 | 필요한 승인 | "예, 다시 묻지 않음" 동작 |
| :--- | :--- | :--- | :--- |
| 읽기 전용 | 파일 읽기, Grep | [작업 디렉터리 및 추가 디렉터리](#working-directories) 내에서는 승인 불필요 | 해당 없음 |
| Bash 명령 | 셸 실행 | 내장된 [읽기 전용 명령 세트](#read-only-commands)를 제외하고 승인 필요 | 리포지토리 및 명령별로 영구 저장 |
| 파일 수정 | 파일 편집/쓰기 | 승인 필요 | 세션 종료 시까지 유효 |

"예, 다시 묻지 않음"을 선택하고 Bash 명령과 같이 승인이 영구적으로 저장되는 경우, Claude Code는 [워크트리(worktrees)](/docs/en/worktrees)를 통해 메인 체크아웃으로 분석되는 git 리포지토리 루트의 `.claude/settings.local.json`에 규칙을 저장합니다. 규칙은 하위 디렉터리 및 워크트리에서 시작된 세션을 포함하여 해당 리포지토리의 모든 위치에서 향후 세션에 적용됩니다. 파일 수정 승인은 파일에 저장되지 않습니다. 표에 표시된 대로 세션이 끝날 때까지만 유지됩니다. git 리포지토리 외부 및 리포지토리 루트가 홈 디렉터리인 경우, Claude Code는 시작한 디렉터리에 규칙을 저장합니다.

v2.1.211 이전에는 Claude Code가 항상 시작 디렉터리에 규칙을 저장했기 때문에 워크트리나 하위 디렉터리에서 부여된 승인이 리포지토리의 나머지 부분에 적용되지 않았습니다. 이전 버전이 하위 디렉터리나 워크트리에 저장한 규칙은 여전히 거기서 시작된 세션에 적용됩니다.

Bash 또는 PowerShell 권한 프롬프트에서 `Ctrl+E`를 누르면 명령에 대한 설명(명령이 수행하는 작업, Claude가 이를 실행하는 이유, 발생할 수 있는 문제)이 **Low risk**, **Med risk**, **High risk**로 표시됩니다. Claude Code는 매 프롬프트마다 전송하는 것이 아니라 사용자가 `Ctrl+E`를 누를 때만 명령과 호출에 대한 Claude 자체의 설명을 모델로 전송하여 설명을 생성합니다. 설명을 표시해도 명령이 실행되지는 않습니다. 숨기려면 `Ctrl+E`를 다시 누르세요.

단축키를 끄려면 `~/.claude.json`에서 [`permissionExplainerEnabled`](/docs/en/settings#global-config-settings)를 `false`로 설정하세요.

## 권한 관리

`/permissions`를 사용하여 Claude Code의 도구 권한을 확인하고 관리할 수 있습니다. 이 UI에는 모든 권한 규칙과 각 규칙이 파생된 `settings.json` 파일이 나열됩니다.

* **허용(Allow)** 규칙은 Claude Code가 수동 승인 없이 지정된 도구를 사용할 수 있도록 합니다.
* **질문(Ask)** 규칙은 Claude Code가 지정된 도구를 사용하려고 할 때마다 확인 프롬프트를 표시합니다.
* **거부(Deny)** 규칙은 Claude Code가 지정된 도구를 사용하지 못하도록 방지합니다.

규칙은 거부(deny), 질문(ask), 허용(allow) 순서대로 평가됩니다. 해당 순서의 첫 번째 일치 항목에 따라 결과가 결정되며 규칙의 구체성(specificity)은 순서를 변경하지 않습니다.

`Bash(aws *)`와 같은 광범위한 거부 규칙은 `Bash(aws s3 ls)`와 같이 더 좁은 허용 규칙과 일치하는 호출을 포함하여 일치하는 모든 호출을 차단하므로 거부 규칙에 허용 목록 예외를 포함할 수 없습니다. 질문과 허용 간에도 동일한 우선순위가 적용됩니다. 더 구체적인 허용 규칙이 동일한 호출과 일치하는 경우에도 일치하는 질문 규칙이 프롬프트를 표시합니다.

거부 규칙은 도구 이름을 지정하는지 아니면 도구 내의 패턴 범위를 지정하는지에 따라 다르게 작동합니다. `Bash`와 같은 단순한 도구 이름은 Claude의 컨텍스트에서 해당 도구를 완전히 제거하므로 Claude는 이를 전혀 보지 못합니다. 단순 이름 제거는 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 제외한 모든 도구에 적용됩니다. 다른 도구가 남아있는 동안 거부 규칙은 이를 제거할 수 없으며 질문 규칙은 이를 프롬프트하지 않습니다. `Bash(rm *)`와 같은 범위 지정 규칙은 도구를 사용할 수 있는 상태로 두고 Claude가 해당 호출을 시도할 때 일치하는 호출을 차단합니다.

<Note>
  권한 규칙은 모델이 아닌 Claude Code에 의해 강제 적용됩니다. 프롬프트나 `CLAUDE.md`에 있는 지침은 Claude가 시도하려는 작업을 조형하지만 Claude Code가 허용하는 내용을 변경하지는 않습니다. 액세스를 부여하거나 취소하려면 `/permissions`, 여기에 설명된 규칙, [권한 모드](/docs/en/permission-modes) 또는 [PreToolUse 훅](#extend-permissions-with-hooks)을 사용하세요.
</Note>

## 권한 모드

Claude Code는 도구 호출 승인 방식을 제어하는 여러 권한 모드를 지원합니다. 각각을 사용하는 시기는 [권한 모드](/docs/en/permission-modes)를 참조하세요. [설정 파일](/docs/en/settings#settings-files)에서 `defaultMode`를 설정하세요:

| 모드 | 설명 |
| :--- | :--- |
| `default` | 표준 동작: 각 도구를 처음 사용할 때 권한을 묻습니다. {/* min-version: 2.1.200 */}CLI, VS Code 및 JetBrains 확장 프로그램, 데스크톱 앱에서 Manual로 표시되며 Claude Code는 `manual`을 별칭으로 허용합니다. 레이블 및 별칭에는 Claude Code v2.1.200 이상이 필요합니다. 데스크톱 앱의 레이블은 CLI 버전에 의존하지 않습니다. |
| `acceptEdits` | 작업 디렉터리 또는 `additionalDirectories`에 있는 경로에 대한 파일 편집 및 `mkdir`, `touch`, `mv`, `cp`와 같은 일반적인 파일 시스템 명령을 자동으로 승인합니다. |
| `plan` | Claude가 탐색을 위해 파일을 읽고 읽기 전용 셸 명령을 실행하지만 소스 파일을 편집하지는 않습니다. CLI 및 VS Code 확장 프로그램에서 Plan으로 표시됩니다. |
| `auto` | 작업이 요청과 일치하는지 확인하는 백그라운드 안전 검사를 통해 도구 호출을 자동 승인합니다. |
| `dontAsk` | `/permissions` 또는 `permissions.allow` 규칙을 통해 사전 승인되지 않은 경우 도구를 자동 거부합니다. `AskUserQuestion`, [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 승인했더라도 거부됩니다. |
| `bypassPermissions` | 명시적 `ask` 규칙, [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구에 의해 강제된 프롬프트를 제외하고 권한 프롬프트를 건너끕니다. `rm -rf /` 및 `rm -rf ~`와 같은 루트 및 홈 디렉터리 삭제도 서킷 브레이커로 계속 프롬프트를 표시합니다. |

<Warning>
  `bypassPermissions` 모드는 `.git`, `.config/git`, `.claude`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`에 대한 쓰기를 포함하여 권한 프롬프트를 건너끕니다. Claude Code가 손상을 입힐 수 없는 컨테이너나 VM과 같이 격리된 환경에서만 이 모드를 사용하세요.

  이 모드에서도 일부 프롬프트는 계속 발생합니다. 명시적인 `ask` 규칙, [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 여전히 프롬프트를 표시합니다. `rm -rf /` 및 `rm -rf ~`와 같이 파일 시스템 루트 또는 홈 디렉터리를 대상으로 하는 삭제도 모델 오류에 대한 서킷 브레이커로서 계속 프롬프트를 표시합니다. {/* min-version: 2.1.208 */}명령에 `$(...)`나 백틱을 사용한 명령 치환 또는 `<(...)`를 사용한 프로세스 치환이 포함된 경우도 포함됩니다. v2.1.208 이전에는 자체 명령으로 작성된 `rm -rf ~`와 같은 일반적인 형태만 프롬프트를 표시했으며, 치환을 통해 삭제에 도달한 명령은 프롬프트를 표시하지 않았습니다.
</Warning>

`bypassPermissions` 또는 `auto` 모드가 사용되지 않도록 하려면 모든 [설정 파일](/docs/en/settings#settings-files)에서 `permissions.disableBypassPermissionsMode` 또는 `permissions.disableAutoMode`를 `"disable"`로 설정하세요. 이러한 모드는 재정의할 수 없는 [관리형 설정](#managed-settings)에서 가장 유용합니다.

## 권한 규칙 구문

권한 규칙은 `Tool` 또는 `Tool(specifier)` 형식을 따릅니다.

### 도구의 모든 사용 일치

도구의 모든 사용과 일치시키려면 괄호 없이 도구 이름만 사용하세요:

| 규칙 | 효과 |
| :--- | :--- |
| `Bash` | 모든 Bash 명령과 일치 |
| `WebFetch` | 모든 웹 가져오기 요청과 일치 |
| `Read` | 모든 파일 읽기와 일치 |

`Bash(*)`는 `Bash`와 동등하며 모든 Bash 명령과 일치합니다. 거부 규칙의 경우 두 형식 모두 Claude의 컨텍스트에서 도구를 제거합니다.

### 세분화된 제어를 위한 지정자(specifiers) 사용

특정 도구 사용과 일치시키려면 괄호 안에 지정자를 추가하세요:

| 규칙 | 효과 |
| :--- | :--- |
| `Bash(npm run build)` | 정확한 명령 `npm run build`와 일치 |
| `Read(./.env)` | 현재 디렉터리의 `.env` 파일 읽기와 일치 |
| `WebFetch(domain:example.com)` | example.com에 대한 가져오기 요청과 일치 |

### 입력 매개변수별 일치

거부 및 질문 규칙은 `Tool(param:value)`를 통해 도구의 최상위 입력 매개변수와 일치할 수 있습니다. 매개변수가 정확히 해당 값으로 설정되어 Claude가 도구를 호출할 때 규칙이 일치합니다. 하나의 매개변수 값에 대한 허용 규칙은 전체적으로 호출이 안전하다는 점을 입증하지 못하므로 허용 규칙은 각 도구 자체의 지정자 구문을 계속 사용합니다. 이는 도구가 수용하는 모든 스칼라 매개변수에 적용됩니다:

| 규칙 | 일치 대상 |
| :--- | :--- |
| `Agent(model:opus)` | Opus 모델 티어를 요청하는 Agent 호출 |
| `Agent(isolation:worktree)` | git worktree를 요청하는 Agent 호출 |
| `Bash(run_in_background:true)` | 백그라운드에서 실행되는 Bash 호출 |

매개변수 일치는 다음 규칙을 따릅니다:

* 매개변수 이름은 Agent 도구의 `model`과 같이 도구 입력의 직접 필드여야 합니다. 객체나 배열 내부에 중첩된 필드는 일치시킬 수 없습니다.
* 각 규칙은 하나의 매개변수를 지정합니다. `model`과 `isolation`을 모두 제한하려면 하나의 규칙에 결합하는 대신 `Agent(model:opus)` 및 `Agent(isolation:worktree)` 두 규칙을 작성하세요.
* 값은 모든 문자 시퀀스와 일치하는 와일드카드로 `*`를 지원하므로 `Agent(isolation:*)`는 명시적인 격리 값과 일치합니다. `*`가 없으면 일치는 정확해야 합니다.
* 모델이 생략한 매개변수는 일치하지 않으므로 `Agent(model:*)`는 `model`을 설정하지 않은 호출과 일치하지 않습니다.
* 정규화 전 Claude가 전송하는 리터럴 입력과 값을 비교합니다. `Agent(model:opus)`는 별칭 `opus`와 일치하지만 전체 모델 ID와는 일치하지 않습니다. 각 도구 호출의 정확한 매개변수 이름과 값을 보려면 [`--verbose`](/docs/en/cli-reference)로 실행하세요.
* 콜론 주변의 공백은 무시됩니다.

도구가 이미 자체 표준화 규칙과 일치하는 필드는 이러한 방식으로 일치시킬 수 없습니다. Bash 및 PowerShell의 경우 `command`, Read, Edit, Write의 경우 `file_path`, Grep 및 Glob의 경우 `path`, NotebookEdit의 경우 `notebook_path`, WebFetch의 경우 `url`이 여기에 해당합니다. `Bash(command:rm *)`와 같은 규칙은 복합 명령에 의해 우회될 수 있으므로 Claude Code는 이를 무시하고 시작 경고를 내보냅니다. 대신 `Bash(rm *)`, `Read(./path)`, `WebFetch(domain:host)`를 사용하세요.

### 와일드카드 패턴

Bash 규칙은 `*`를 통한 글로브 패턴을 지원합니다. 와일드카드는 명령의 어느 위치에나 나타날 수 있습니다. 이 구성은 git push를 차단하면서 npm 및 git commit 명령을 허용합니다:

```json theme={null}
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Bash(git * main)",
      "Bash(* --version)",
      "Bash(* --help *)"
    ],
    "deny": [
      "Bash(git push *)"
    ]
  }
}
```

`*` 앞의 공백이 중요합니다: `Bash(ls *)`는 `ls -la`와 일치하지만 `lsof`와는 일치하지 않으며, `Bash(ls*)`는 둘 다와 일치합니다. `:*` 접미사는 미행 와일드카드를 작성하는 동등한 방법이므로 `Bash(ls:*)`는 `Bash(ls *)`와 동일한 명령과 일치합니다.

권한 대화 상자는 명령 접두사에 대해 "예, 다시 묻지 않음"을 선택할 때 공백으로 구분된 형식을 작성합니다. `:*` 형식은 패턴의 끝에서만 인식됩니다. `Bash(git:* push)`와 같은 패턴에서 콜론은 리터럴 문자로 취급되며 git 명령과 일치하지 않습니다.

### 도구 이름 와일드카드

거부 및 질문 규칙은 도구 이름 위치에서도 글로브 패턴을 허용합니다. 패턴은 전체 도구 이름과 일치해야 합니다: `"*"`는 모든 도구와 일치하며 `"mcp__*"`는 모든 서버에 걸쳐 모든 MCP 도구와 일치합니다. 단순 이름 글로브 거부 규칙과 일치하는 도구는 단순 도구 이름과 마찬가지로 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior) 예외를 포함하여 Claude의 컨텍스트에서 제거됩니다. 글로브 거부는 다른 도구가 남아있는 동안 이를 제거할 수 없으며 글로브 질문은 이에 대해 프롬프트를 표시하지 않습니다. 이 구성은 모든 MCP 도구를 거부합니다:

```json theme={null}
{
  "permissions": {
    "deny": [
      "mcp__*"
    ]
  }
}
```

허용 규칙은 리터럴 `mcp__<server>__` 접두사 뒤에서만 도구 이름 글로브를 허용합니다. 규칙이 구성한 특정 서버의 이름을 지정하도록 서버 세그먼트에 글로브가 없어야 합니다. `mcp__puppeteer__*`는 `puppeteer` 서버의 모든 도구와 일치하고 `mcp__github__get_*`는 `get_` 도구와 일치합니다. `"*"`, `"B*"`, `"mcp__*"`와 같이 고정되지 않은 허용 글로브는 경고와 함께 건너뛰어지며 아무것도 자동 승인하지 않습니다.

알려진 도구와 일치하지 않는 도구 이름이 있는 거부 또는 질문 규칙은 오타를 포착하기 위해 시작 경고를 생성합니다. `_` 또는 `*`가 포함된 도구 이름은 이 검사에서 제외됩니다.

트랜스크립트 및 권한 대화 상자에서 도구에 표시되는 레이블은 정식 이름과 다를 수 있습니다. 예를 들어 트랜스크립트에서 `Stop Task`로 라벨 지정된 도구의 정식 이름은 `TaskStop`입니다. 권한 규칙 및 [훅 일치기(hook matchers)](/docs/en/hooks)는 정식 이름만 일치하므로 `Stop Task`로 작성된 규칙은 일치하지 않습니다. 거부 및 질문 규칙의 경우 위의 시작 경고가 불일치를 포착합니다. [도구 참조](/docs/en/tools-reference)에 나열된 정식 이름을 사용하세요.

## 도구별 권한 규칙

### Bash

Bash 권한 규칙은 `*`를 사용한 와일드카드 일치를 지원합니다. 와일드카드는 시작, 중간, 끝을 포함하여 명령의 어느 위치에나 나타날 수 있습니다:

* `Bash(npm run build)`는 정확한 Bash 명령 `npm run build`와 일치합니다.
* `Bash(npm run test *)`는 `npm run test`로 시작하는 Bash 명령과 일치합니다.
* `Bash(npm *)`는 `npm `으로 시작하는 모든 명령과 일치합니다.
* `Bash(* install)`는 ` install`로 끝나는 모든 명령과 일치합니다.
* `Bash(git * main)`는 `git checkout main` 및 `git log --oneline main`과 같은 명령과 일치합니다.

단일 `*`는 공백을 포함하여 모든 문자 시퀀스와 일치하므로 하나의 와일드카드가 여러 인수에 걸칠 수 있습니다. `Bash(git *)`는 `git log --oneline --all`과 일치하고 `Bash(git * main)`는 `git push origin main` 및 `git merge main`과 일치합니다.

`*`가 공백이 앞에 붙은 상태로 끝에 나타나면(`Bash(ls *)`처럼) 단어 경계를 강제하므로 접두사 뒤에 공백이 오거나 문자열의 끝이어야 합니다. 예를 들어 `Bash(ls *)`는 `ls -la`와 일치하지만 `lsof`와는 일치하지 않습니다. 반대로 공백이 없는 `Bash(ls*)`는 단어 경계 제약 조건이 없기 때문에 `ls -la` 및 `lsof` 모두와 일치합니다.

#### 복합 명령

<Tip>
  Claude Code는 셸 연산자를 인식하므로 `Bash(safe-cmd *)`와 같은 규칙이 `safe-cmd && other-cmd` 명령을 실행할 수 있는 권한을 부여하지 않습니다. 인식되는 명령 구분 기호는 `&&`, `||`, `;`, `|`, `|&`, `&` 및 줄바꿈입니다. 규칙은 각 하위 명령과 독립적으로 일치해야 합니다.
</Tip>

"예, 다시 묻지 않음"으로 복합 명령을 승인하면 Claude Code는 전체 복합 문자열에 대한 단일 규칙이 아닌 승인이 필요한 각 하위 명령에 대해 별도의 규칙을 저장합니다. 예를 들어 `git status && npm test`를 승인하면 `npm test`에 대한 규칙이 저장되므로 향후 `npm test` 호출은 `&&` 앞에 무엇이 오는지에 관계없이 인식됩니다. 하위 디렉터리로 이동하는 `cd`와 같은 하위 명령은 해당 경로에 대한 자체 Read 규칙을 생성합니다. 하나의 복합 명령에 대해 최대 5개의 규칙을 저장할 수 있습니다.

<h4 id="process-wrappers">
  래퍼 (Wrappers)
</h4>

Bash 규칙을 일치시키기 전에 Claude Code는 고정된 래퍼 세트를 제거하므로 `Bash(npm test *)`와 같은 규칙도 `timeout 30 npm test`와 일치합니다. 제거되는 래퍼는 `timeout`, `time`, `nice`, `nohup`, `stdbuf`와 셸 내장 기능인 `command` 및 `builtin`, zsh의 `noglob`입니다. 각각은 인수를 실제 명령으로 실행합니다. 관련 없는 두 가지 형식은 제거되지 않습니다. 하나는 명령을 실행하지 않고 조회하는 쿼리 형식 `command -v`이고, 다른 하나는 zsh의 `nocorrect`입니다.

또한 Claude Code는 특정 알려진 안전한 환경 변수의 선두 할당을 제거하므로 `Bash(npm test *)`는 `NODE_ENV=test npm test`와 일치합니다. 허용 규칙은 다른 변수의 할당을 지나 일치하지 않습니다. 거부 또는 질문 규칙은 선두 할당을 지나 일치하므로 거부 항목의 `Bash(rm *)`는 여전히 `FOO=bar rm -rf tmp/`와 일치합니다.

단순 `xargs`도 제거되므로 `Bash(grep *)`는 `xargs grep pattern`과 일치합니다. 제거는 `xargs`에 플래그가 없을 때만 적용됩니다. `xargs -n1 grep pattern`과 같은 호출은 `xargs` 명령으로 일치하므로 내부 명령용으로 작성된 규칙은 이를 포함하지 않습니다.

이 래퍼 목록은 내장되어 있으며 구성할 수 없습니다. `direnv exec`, `devbox run`, `mise exec`, `npx`, `docker exec`와 같은 개발 환경 러너는 목록에 없습니다. 이러한 도구는 인수를 명령으로 실행하므로 `Bash(devbox run *)`와 같은 규칙은 `devbox run rm -rf .`를 포함하여 `run` 뒤에 오는 모든 것과 일치합니다. 환경 러너 내에서 작업을 승인하려면 `Bash(devbox run npm test)`와 같이 러너와 내부 명령을 모두 포함하는 구체적인 규칙을 작성하세요. 허용하려는 내부 명령당 하나의 규칙을 추가하세요.

`watch`, `setsid`, `ionice`, `flock`과 같은 Exec 래퍼는 항상 프롬프트를 표시하며 `Bash(watch *)`와 같은 접두사 규칙으로 자동 승인될 수 없습니다. `-exec` 또는 `-delete`가 있는 `find`에도 동일하게 적용됩니다. `Bash(find *)` 규칙은 이러한 형식을 포함하지 않습니다. 특정 호출을 승인하려면 전체 명령 문자열에 대해 정확히 일치하는 규칙을 작성하세요.

#### 읽기 전용 명령

Claude Code는 내장된 Bash 명령 세트를 읽기 전용으로 인식하고 모든 모드에서 권한 프롬프트 없이 실행합니다. 여기에는 `ls`, `cat`, `echo`, `pwd`, `head`, `tail`, `grep`, `find`, `wc`, `which`, `diff`, `stat`, `du`, `cd` 및 `git` 읽기 전용 형식이 포함됩니다. 이 세트는 구성할 수 없습니다. 이러한 명령 중 하나에 대해 프롬프트를 요구하려면 해당 명령에 대한 `ask` 또는 `deny` 규칙을 추가하세요.

모든 플래그가 읽기 전용인 명령에는 따옴표로 묶이지 않은 글로브 패턴이 허용되므로 `ls *.ts` 및 `wc -l src/*.py`는 프롬프트 없이 실행됩니다. `find`, `sort`, `sed`, `git`과 같이 쓰기가 가능하거나 실행 가능한 플래그가 있는 명령은 따옴표로 묶이지 않은 글로브가 있는 경우 글로브가 `-delete`와 같은 플래그로 확장될 수 있으므로 여전히 프롬프트를 표시합니다.

작업 디렉터리 또는 [추가 디렉터리](#working-directories) 내의 경로로 이동하는 `cd`도 읽기 전용입니다. `cd packages/api && ls`와 같은 복합 명령은 각 부분이 자체적으로 자격을 갖춘 경우 프롬프트 없이 실행됩니다. 하나의 복합 명령에서 `cd`와 `git`을 조합하면 `cd`가 다른 디렉터리로 변경될 때 프롬프트를 표시합니다. 새 디렉터리에서 `git`을 실행하면 해당 디렉터리의 훅이 실행될 수 있기 때문입니다. 대상이 현재 작업 디렉터리로 해석되는 `cd`는 아무 작업도 하지 않으므로 이 프롬프트를 트리거하지 않습니다.

하나의 복합 명령에서 `cd`와 출력 리디렉션을 조합하면 `cd`가 실행된 후 Claude Code가 리디렉션 대상이 해석되는 디렉터리를 확인할 수 없는 경우에도 프롬프트를 표시합니다. `cd app; grep -r pattern . 2>/dev/null`과 같이 리디렉션 대상이 `/dev/null` 뿐인 명령은 `/dev/null`이 작업 디렉터리에 의존하지 않기 때문에 이 프롬프트를 트리거하지 않습니다. {/* min-version: 2.1.207 */}v2.1.207 이전에는 `cd`를 포함하는 복합 명령이 유일한 대상이 `/dev/null`인 경우를 포함하여 모든 출력 리디렉션에 대해 프롬프트를 표시했습니다.

<Warning>
  명령 인수를 제한하려는 Bash 권한 패턴은 취약합니다. 예를 들어 `Bash(curl http://github.com/ *)`는 curl을 GitHub URL로 제한하려는 의도이지만 다음과 같은 변형에는 일치하지 않습니다:

  * URL 앞의 옵션: `curl -X GET http://github.com/...`
  * 다른 프로토콜: `curl https://github.com/...`
  * 리디렉션: GitHub로 리디렉션되는 `curl -L http://short.example.com/xyz`
  * 변수: `URL=http://github.com && curl $URL`
  * 추가 공백: `curl  http://github.com`

  더 안정적인 URL 필터링을 위해 다음을 고려하세요:

  * **Bash 네트워크 도구 제한**: 거부 규칙을 사용하여 `curl`, `wget` 및 유사한 명령을 차단한 다음 허용된 도메인에 대한 `WebFetch(domain:github.com)` 권한과 함께 WebFetch 도구를 사용합니다.
  * **PreToolUse 훅 사용**: Bash 명령의 URL을 검증하고 허용되지 않는 도메인을 차단하는 훅을 구현합니다.
  * **CLAUDE.md 지침 추가**: `CLAUDE.md`에 허용된 curl 패턴을 설명합니다. 이는 Claude가 시도하는 작업을 결정하지만 경계를 강제하지는 않으므로 위의 옵션 중 하나와 조합하세요.

  WebFetch만 사용해도 네트워크 액세스가 차단되는 것은 아닙니다. Bash가 허용된 경우 Claude는 여전히 `curl`, `wget` 또는 기타 도구를 사용하여 모든 URL에 도달할 수 있습니다.
</Warning>

### PowerShell

PowerShell 권한 규칙은 Bash 규칙과 동일한 형태를 사용합니다. `*`를 사용한 와일드카드는 모든 위치에서 일치하고, `:*` 접미사는 미행 ` *`와 동등하며, 단순 `PowerShell` 또는 `PowerShell(*)`은 모든 명령과 일치합니다. 이 구성은 `Remove-Item`을 차단하면서 `Get-ChildItem` 및 `git commit` 명령을 허용합니다:

```json theme={null}
{
  "permissions": {
    "allow": [
      "PowerShell(Get-ChildItem *)",
      "PowerShell(git commit *)"
    ],
    "deny": [
      "PowerShell(Remove-Item *)"
    ]
  }
}
```

일반적인 별칭은 일치하기 전에 표준화됩니다. cmdlet 이름용으로 작성된 규칙은 해당 별칭과도 일치하므로 `PowerShell(Get-ChildItem *)`은 `gci`, `ls`, `dir`과도 일치합니다. 일치는 대소문자를 구분하지 않습니다.

Claude Code는 PowerShell AST를 구문 분석하고 복합 명령의 각 명령을 독립적으로 검사합니다. 파이프라인 연산자 `|`, 문 구분 기호 `;` 및 PowerShell 7+에서는 체인 연산자 `&&` 및 `||`가 복합 명령을 하위 명령으로 분할합니다. 복합 명령이 허용되려면 규칙이 모든 하위 명령과 일치해야 합니다.

### Read 및 Edit

`Edit` 규칙은 파일을 편집하는 모든 내장 도구에 적용됩니다. Claude는 Grep 및 Glob과 같이 파일을 읽는 모든 내장 도구, 프롬프트의 `@file` 멘션, 연동된 [IDE](/docs/en/vs-code#the-built-in-ide-mcp-server)가 Claude와 공유하는 선택 영역 및 열린 파일 컨텍스트에 `Read` 규칙을 적용하기 위해 최선의 노력을 다합니다.

{/* min-version: 2.1.208 */}`Read` 거부 규칙은 동일한 경로의 [Edit 도구](/docs/en/errors#file-is-covered-by-a-read-deny-rule)도 차단합니다(거기에 새 파일을 만드는 것 포함). Write 및 NotebookEdit는 포함되지 않으므로 어떠한 도구도 변경할 수 없는 경로의 경우 `Edit` 거부 규칙을 추가하세요. Claude Code v2.1.208 이상이 필요합니다.

{/* min-version: 2.1.210 */}파일 권한 검사는 `Edit(path)` 및 `Read(path)` 규칙만 일치합니다. `Write(path)`, `NotebookEdit(path)`, `Glob(path)` 규칙은 수용되지만 해당 검사와는 일치하지 않으므로 Claude Code는 해당 일치하지 않는 형태의 각 허용, 거부, 질문 규칙에 대해 시작 시 경고합니다. `Write(docs/**)` 또는 `NotebookEdit(docs/**)` 대신 `Edit(docs/**)`를 사용하고 `Glob(docs/**)` 대신 `Read(docs/**)`를 사용하세요. `Write`에 대한 거부 규칙과 같이 경로가 없는 도구 이름 규칙은 영향을 받지 않습니다. 모든 위치에서 도구와 일치하며 경고를 생성하지 않습니다. Claude Code v2.1.210 이상이 필요합니다.

프로젝트 설정의 거부 규칙 `Write(docs/**)`는 다음 시작 경고를 생성합니다:

```text theme={null}
Permission deny rule (.claude/settings.json): Write(docs/**) is not matched by file permission checks — only Edit(path) rules are. Use Edit(docs/**) instead (Edit rules cover all file-editing tools).
```

<Warning>
  Read 및 Edit 거부 규칙은 Claude의 내장 파일 도구와 Claude Code가 Bash에서 인식하는 파일 명령(`cat`, `head`, `tail`, `sed` 등)에 적용됩니다. 자체적으로 파일을 여는 Python 또는 Node 스크립트와 같이 간접적으로 파일을 읽거나 쓰는 임의의 하위 프로세스에는 적용되지 않습니다. 모든 프로세스가 경로에 접근하지 못하도록 하는 OS 수준의 시행을 위해 [샌드박스를 활성화](/docs/en/sandboxing)하세요.
</Warning>

Read 및 Edit 규칙은 모두 네 가지 구체적인 패턴 유형이 있는 [gitignore](https://git-scm.com/docs/gitignore) 사양을 따릅니다:

| 패턴 | 의미 | 예시 | 일치 대상 |
| --- | --- | --- | --- |
| `//path` | 파일 시스템 루트로부터의 절대 경로 | `Read(//Users/alice/secrets/**)` | `/Users/alice/secrets/**` |
| `~/path` | 홈 디렉터리로부터의 경로 | `Read(~/Documents/*.pdf)` | `/Users/alice/Documents/*.pdf` |
| `/path` | 설정 소스에 상대적인 경로 | `Edit(/src/**/*.ts)` | 프로젝트 설정의 `<project root>/src/**/*.ts` |
| `path` 또는 `./path` | 현재 디렉터리에 상대적인 경로 | `Read(*.env)` | `<cwd>/*.env` |

<Warning>
  `/Users/alice/file`과 같은 패턴은 절대 경로가 아닙니다. 단일 선두 슬래시는 파일 시스템 루트가 아닌 설정 소스에 고정됩니다. 절대 경로에는 `//Users/alice/file`을 사용하세요.
</Warning>

`/path` 패턴은 이를 정의하는 설정 소스와 연결된 디렉터리에 고정되므로 저장 위치에 따라 동일한 규칙이 다른 위치와 일치합니다:

| 규칙 정의 위치 | `/path` 해석 결과 |
| :--- | :--- |
| `.claude/settings.json`의 프로젝트 설정 | `<project root>/path` |
| `.claude/settings.local.json`의 로컬 설정 | `<original cwd>/path` |
| `~/.claude/settings.json`의 사용자 설정 | `~/.claude/path` |
| `--settings <file>`로 전달된 파일 | `<directory of file>/path` |
| CLI 플래그, `/permissions`, 또는 세션 규칙 | `<original cwd>/path` |

로컬 설정 규칙은 v2.1.211 이상에서 Claude Code가 [파일을 저장](#permission-system)하는 리포지토리 루트가 아닌 Claude Code를 시작한 디렉터리에 고정됩니다. 리포지토리 루트에서 시작된 세션에서는 두 디렉터리가 동일합니다. [워크트리(worktrees)](/docs/en/worktrees) 세션에서 `Edit(/src/**)`와 같은 공유 규칙은 해당 워크트리 자체의 `src/` 디렉터리와 일치합니다.

사용자 설정의 `Read(/secrets/**)`와 같은 거부 규칙은 프로젝트의 `secrets` 디렉터리가 아닌 `~/.claude/secrets/**`를 차단합니다. 모든 프로젝트 내부에서 적용되는 사용자 설정 규칙을 작성하려면 대신 `//` 절대 경로 또는 `~/` 홈 상대 경로를 사용하세요.

Windows에서는 패턴을 일치시키기 전에 경로가 POSIX 형식으로 표준화됩니다. `C:\Users\alice`는 `/c/Users/alice`가 되므로 해당 드라이브의 모든 위치에서 `.env` 파일과 일치시키려면 `//c/**/.env`를 사용하세요. 모든 드라이브에서 일치시키려면 `//**/.env`를 사용하세요.

예시:

* `Edit(/docs/**)`: `/docs/` 또는 `<project>/.claude/docs/`가 아닌 `<project>/docs/` 내부 편집
* `Read(~/.zshrc)`: 홈 디렉터리의 `.zshrc` 읽기
* `Edit(//tmp/scratch.txt)`: 절대 경로 `/tmp/scratch.txt` 편집
* `Read(src/**)`: `<current-directory>/src/`에서 읽기

규칙은 고정된 위치 아래의 파일과만 일치하므로 고정 위치에 따라 거부 규칙이 미치는 범위가 결정됩니다. 단순 파일 이름은 gitignore 시맨틱을 따르며 모든 깊이에서 일치하므로 `Read(.env)` 및 `Read(**/.env)`는 동등합니다:

| 거부 규칙 | 차단 대상 | 차단하지 않는 대상 |
| --- | --- | --- |
| `Read(.env)` 또는 `Read(**/.env)` | 현재 디렉터리 또는 그 하위의 모든 `.env` | 상위 디렉터리나 다른 프로젝트의 `.env` |
| `Read(//**/.env)` | 파일 시스템의 모든 위치에 있는 모든 `.env` | 없음. 규칙이 파일 시스템 루트에 고정됨 |

<Note>
  gitignore 패턴에서 `*`는 단일 경로 세그먼트 내에서 일치하고 패턴의 모든 위치에 나타날 수 있는 반면 `**`는 디렉터리를 가로질러 일치합니다. 모든 파일 접근을 허용하려면 괄호 없이 도구 이름만 사용하세요: `Read`, `Edit`, `Write`.
</Note>

"예, 다시 묻지 않음"으로 파일 경로를 승인하면 Claude Code는 해당 경로에서 `[`, `]`, `*`와 같은 gitignore 패턴 문자를 이스케이프하므로 생성된 규칙은 승인한 리터럴 경로와만 일치합니다. 직접 작성한 규칙은 이스케이프되지 않습니다. v2.1.202 이전에는 Claude Code가 이스케이프되지 않은 경로를 저장했기 때문에 `[2024-06] Reports`라는 디렉터리에 대해 생성된 규칙이 자체 경로와 일치하지 않거나 의도하지 않은 형제 디렉터리와 일치할 수 있었습니다.

Claude가 심볼릭 링크에 접근할 때 권한 규칙은 심볼릭 링크 자체와 심볼릭 링크가 가리키는 파일이라는 두 가지 경로를 검사합니다. 허용 및 거부 규칙은 해당 쌍을 다르게 취급합니다. 허용 규칙은 프롬프트를 표시하는 방식으로 폴백하는 반면 거부 규칙은 즉시 차단합니다.

* **허용 규칙**: 심볼릭 링크 경로와 그 대상이 모두 일치할 때만 적용됩니다. 허용된 디렉터리 내의 심볼릭 링크가 그 외부를 가리키는 경우에도 여전히 프롬프트를 표시합니다.
* **거부 규칙**: 심볼릭 링크 경로나 그 대상 중 하나라도 일치할 때 적용됩니다. 거부된 파일을 가리키는 심볼릭 링크는 그 자체로 거부됩니다.

예를 들어 `Read(./project/**)`가 허용되고 `Read(~/.ssh/**)`가 거부된 경우 `~/.ssh/id_rsa`를 가리키는 `./project/key` 위치의 심볼릭 링크는 차단됩니다. 대상이 허용 규칙을 통과하지 못하고 거부 규칙과 일치하기 때문입니다.

### WebFetch

WebFetch 규칙은 `domain:` 접두사를 사용하고 요청된 URL의 호스트 이름과 일치합니다. 일치는 대소문자를 구분하지 않고 `*` 와일드카드를 지원하며 규칙과 호스트 이름 모두에서 끝의 `.`을 제거하므로 `example.com.`과 `example.com`은 동일하게 취급됩니다.

* `WebFetch(domain:example.com)`은 `example.com`에 대한 요청과 일치합니다.
* `WebFetch(domain:*.example.com)`은 `api.example.com` 또는 `a.b.example.com`과 같이 모든 깊이의 하위 도메인과 일치하지만 `example.com` 자체와는 일치하지 않습니다.
* `WebFetch(domain:*)`은 모든 도메인과 일치하며 단순 `WebFetch` 규칙과 동등합니다.

선두 `*.` 또는 단순 `*` 이외의 위치에서 와일드카드는 두 점 사이의 텍스트와만 일치합니다. `WebFetch(domain:example.*)`는 `*`가 `org`가 되는 `example.org`와 일치하지만 `*`가 `evil.com`이 되어 점을 넘어가야 하는 `example.evil.com`과는 일치하지 않습니다. 이렇게 하면 공격자가 등록할 수 있는 도메인과 미행 와일드카드가 일치하는 것을 방지합니다.

### MCP

MCP 규칙은 Claude Code에 구성된 서버 이름을 사용하며 선택적으로 해당 서버의 도구 이름이 뒤따릅니다.

* `mcp__puppeteer`는 `puppeteer` 서버가 제공하는 모든 도구와 일치합니다.
* `mcp__puppeteer__*`는 와일드카드 구문을 사용하며 `puppeteer` 서버의 모든 도구와도 일치합니다.
* `mcp__puppeteer__puppeteer_navigate`는 `puppeteer` 서버가 제공하는 `puppeteer_navigate` 도구와 일치합니다.

조직에서 [claude.ai 커넥터](/docs/en/mcp#organization-controls-on-connector-tools) 도구를 `ask`로 설정한 경우 해당 도구에 대한 허용 규칙이 적용되지 않습니다. Claude Code는 `auto` 및 `bypassPermissions` 모드에서도 매 호출 시 프롬프트를 표시합니다. 프롬프트를 전혀 표시하지 않는 `dontAsk` 모드에서는 Claude Code가 대신 호출을 거부합니다. 커넥터 도구는 `mcp__claude_ai_<server>__<tool>`로 나타납니다.

### Agent (서브에이전트)

Claude가 사용할 수 있는 [서브에이전트](/docs/en/sub-agents)를 제어하려면 `Agent(AgentName)` 규칙을 사용하세요:

* `Agent(Explore)`는 Explore 서브에이전트와 일치합니다.
* `Agent(Plan)`는 Plan 서브에이전트와 일치합니다.
* `Agent(my-custom-agent)`는 `my-custom-agent`라는 이름의 커스텀 서브에이전트와 일치합니다.

설정의 `deny` 배열에 이러한 규칙을 추가하거나 `--disallowedTools` CLI 플래그를 사용하여 특정 에이전트를 비활성화하세요. Explore 에이전트를 비활성화하려면:

```json theme={null}
{
  "permissions": {
    "deny": ["Agent(Explore)"]
  }
}
```

### Cd

`Cd` 규칙은 [`/cd` 명령](/docs/en/commands)이 세션을 이동할 수 있는 디렉터리를 제어합니다. `Cd`는 모델이 호출할 수 있는 도구가 아닙니다. Claude는 이를 호출할 수 없으며 규칙은 직접 `/cd`를 실행할 때만 적용됩니다.

단순 `Cd` 거부 규칙은 `/cd`를 완전히 비활성화합니다. `Cd(<path-pattern>)` 거부 규칙은 일치하는 대상을 차단합니다. 거부 규칙은 해당 경로로 분석되는 모든 심볼릭 링크 홉을 포함하여 대상의 모든 표기를 검사하므로 하나의 경로용으로 작성된 규칙은 해당 경로로 분석되는 대상도 차단합니다.

`Cd` 허용 규칙을 추가하면 `/cd`가 허용 목록 모드로 전환됩니다. 해석된 대상 디렉터리가 허용 규칙 중 하나와 일치해야 하며 그렇지 않으면 `/cd`가 거부됩니다. `Cd` 규칙이 구성되어 있지 않으면 `/cd`는 기본 동작을 유지하고 생소한 디렉터리를 신뢰하도록 프롬프트를 표시합니다.

경로 패턴은 [Read 및 Edit 규칙](#read-and-edit)의 `//`, `~/`, `/` 앵커를 공유하지만 일치는 gitignore 스타일이 아닌 전체 디렉터리 경로에 고정됩니다. `*`는 정확히 하나의 경로 세그먼트와 일치하고 `**`는 세그먼트를 가로질러 일치합니다. 끝의 `/**`도 해당 지정된 루트와 일치합니다.

| 규칙 | 일치 대상 | 일치하지 않는 대상 |
| --- | --- | --- |
| `Cd(~/code/*)` | `~/code/app` | `~/code/app/src`, `~/code` |
| `Cd(~/code/**)` | `~/code` 및 그 하위의 모든 디렉터리 | `~/code` 외부의 디렉터리 |
| `Cd(**/node_modules)` | 모든 깊이의 모든 `node_modules` 디렉터리 | `node_modules/pkg` |

## 훅으로 권한 확장

[Claude Code 훅](/docs/en/hooks-guide)을 사용하면 런타임에 권한을 평가하는 커스텀 셸 명령을 등록할 수 있습니다. Claude Code가 도구를 호출할 때 PreToolUse 훅은 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 제외한 모든 도구에 대해 권한 프롬프트 전에 실행됩니다. 훅 출력은 도구 호출을 거부하거나, 프롬프트를 강제하거나, 프롬프트를 건너뛰어 호출을 진행하도록 할 수 있습니다.

훅 결정은 권한 규칙을 우회하지 않습니다. Claude Code는 PreToolUse 훅이 반환하는 내용에 관계없이 거부 및 질문 규칙을 평가합니다. 일치하는 거부 규칙은 호출을 차단하고, 일치하는 질문 규칙은 훅이 `"allow"` 또는 `"ask"`를 반환하더라도 여전히 프롬프트를 표시합니다. 이렇게 하면 관리형 설정에 설정된 거부 규칙을 포함하여 [권한 관리](#manage-permissions)에 설명된 거부 우선 우선순위가 유지됩니다.

[조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구도 훅이 `"allow"`를 반환할 때 여전히 프롬프트를 표시합니다.

차단 훅도 허용 규칙보다 우선합니다. 종료 코드 2로 종료되는 훅은 권한 규칙이 평가되기 전에 도구 호출을 정지하므로 허용 규칙이 그렇지 않고 호출을 진행하도록 하는 경우에도 차단이 적용됩니다. 프롬프트 거부 대상 몇 개를 제외하고 모든 Bash 명령을 프롬프트 없이 실행하려면 허용 목록에 `"Bash"`를 추가하고 해당 특정 명령을 거부하는 PreToolUse 훅을 등록하세요. 조정할 수 있는 훅 스크립트는 [보호된 파일에 대한 편집 차단](/docs/en/hooks-guide#block-edits-to-protected-files)을 참조하세요.

## 작업 디렉터리

기본적으로 Claude는 이를 실행한 디렉터리의 파일에 접근할 수 있습니다. 이 접근 권한을 확장할 수 있습니다:

* **시작 시**: `--add-dir <path>` CLI 인수 사용
* **세션 중에**: `/add-dir` 명령 사용
* **영구 구성**: [설정 파일](/docs/en/settings#settings-files)의 `additionalDirectories`에 추가

추가 디렉터리의 파일은 원래 작업 디렉터리와 동일한 권한 규칙을 따릅니다. 프롬프트 없이 읽을 수 있게 되며 파일 편집 권한은 현재 권한 모드를 따릅니다.

macOS의 백그라운드 세션에서 세션 호스트는 Claude가 거기에 있는 파일을 읽거나 써야 할 때 터미널과 별도로 `~/Desktop`, `~/Documents`, `~/Downloads`와 같은 보호된 폴더에 대한 접근을 요청합니다. 거기서 읽기가 `Operation not permitted`로 실패하는 경우 [백그라운드 세션에 폴더 접근 권한을 부여하는 방법](/docs/en/agent-view#background-sessions-can’t-read-desktop-documents-or-downloads-on-macos)을 참조하세요.

다른 디렉터리를 추가하는 대신 세션의 기본 작업 디렉터리를 변경하려면 [`/cd`](/docs/en/commands)를 사용하세요. `/cd` 명령에는 Claude Code v2.1.169 이상이 필요합니다. `/add-dir`과 달리 세션을 이동합니다: 새 디렉터리의 `CLAUDE.md`가 로드되고 `--resume`이 거기서 세션을 찾습니다.

### 추가 디렉터리는 파일 접근만 부여하며 구성을 부여하지 않음

디렉터리를 추가하면 Claude가 파일을 읽고 편집할 수 있는 위치가 확장됩니다. 해당 디렉터리가 전체 구성 루트가 되는 것은 아닙니다: 예외로 로드되는 몇 가지 유형을 제외하고는 대부분의 `.claude/` 구성이 추가 디렉터리에서 검색되지 않습니다.

이러한 예외는 `--add-dir` 플래그 또는 `/add-dir` 명령으로 추가된 디렉터리에만 적용됩니다. 설정 파일의 `permissions.additionalDirectories`에 나열된 디렉터리는 파일 접근만 부여하며 아래 구성은 로드하지 않습니다.

다음 구성 유형은 `--add-dir` 디렉터리에서 로드됩니다:

| 구성 | `--add-dir`에서 로드 여부 |
| :--- | :--- |
| `.claude/skills/`의 [스킬](/docs/en/skills) | 예 (실시간 다시 로드 포함) |
| `.claude/agents/`의 [서브에이전트](/docs/en/sub-agents) | 예 |
| `.claude/settings.json` 및 `.claude/settings.local.json`의 [설정](/docs/en/settings) | `enabledPlugins` 및 `extraKnownMarketplaces` 키만 해당 |
| [CLAUDE.md](/docs/en/memory) 파일, `.claude/rules/`, `CLAUDE.local.md` | `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`이 설정된 경우에만 해당. `CLAUDE.local.md`에는 기본적으로 활성화되어 있는 `local` 설정 소스가 추가로 필요함 |

Claude Code는 현재 작업 디렉터리 및 그 상위 디렉터리, `~/.claude/` 사용자 디렉터리, 관리형 설정에서 명령 및 출력 스타일을 검색합니다. 훅 및 기타 `.claude/settings.json` 키는 상위 디렉터리 폴백 없이 현재 작업 디렉터리의 `.claude/` 폴더에서 사용자 `~/.claude/settings.json` 및 관리형 설정과 함께 로드됩니다. {/* min-version: 2.1.211 */}하위 디렉터리에서 Claude Code를 시작하더라도 `.claude/settings.local.json`은 대신 git 리포지토리 루트에서 로드됩니다. v2.1.211 이전에는 이 역시 현재 작업 디렉터리에서만 로드되었습니다. [Agent SDK](/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources) 세션은 모든 버전의 작업 디렉터리에서 이를 로드합니다.

프로젝트 전반에 걸쳐 해당 구성을 공유하려면 다음 접근 방식 중 하나를 사용하세요:

* **사용자 수준 구성**: 파일들을 `~/.claude/agents/`, `~/.claude/output-styles/` 또는 `~/.claude/settings.json`에 배치하여 모든 프로젝트에서 사용할 수 있도록 합니다.
* **플러그인**: 팀이 설치할 수 있는 [플러그인](/docs/en/plugins)으로 구성을 패키징하고 배포합니다.
* **구성 디렉터리에서 실행**: 원하는 `.claude/` 구성이 포함된 디렉터리에서 Claude Code를 실행합니다.

## 권한과 샌드박싱의 상호작용 방식

권한과 [샌드박싱](/docs/en/sandboxing)은 상호 보완적인 보안 레이어입니다:

* **권한**은 Claude Code가 사용할 수 있는 도구와 접근할 수 있는 파일 또는 도메인을 제어합니다. 거부 또는 질문 규칙이 다른 도구가 남아있는 동안 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 차단할 수 없다는 점을 제외하고 Bash, Read, Edit, WebFetch, MCP 및 기타 모든 도구에 적용됩니다.
* **샌드박싱**은 Bash 도구의 파일 시스템 및 네트워크 접근을 제한하는 OS 수준의 시행을 제공합니다. 이는 Bash 명령 및 그 하위 프로세스에만 적용됩니다.

심층 방어를 위해 둘 다 사용하세요:

* 권한 거부 규칙은 Claude가 제한된 리소스에 접근을 시도하는 것조차 차단합니다.
* 샌드박스 제한은 프롬프트 주입이 Claude의 의사 결정을 우회하더라도 Bash 명령이 정의된 경계를 벗어난 리소스에 도달하는 것을 방지합니다.
* 샌드박스의 파일 시스템 제한은 [`sandbox.filesystem`](/docs/en/sandboxing) 설정과 Read 및 Edit 거부 규칙을 조합합니다. 둘 다 최종 샌드박스 경계에 병합됩니다.
* 네트워크 제한은 WebFetch 권한 규칙과 샌드박스의 `allowedDomains` 및 `deniedDomains` 목록을 조합합니다.

샌드박싱을 활성화하고 `autoAllowBashIfSandboxed`를 기본값인 `true`로 두면 권한에 단순 `Bash` 질문 규칙이나 [동등한 `Bash(*)` 형식](#match-all-uses-of-a-tool)이 포함되어 있더라도 샌드박스 처리된 Bash 명령은 프롬프트 없이 실행됩니다. 샌드박스 경계가 해당 전체 도구 프롬프트를 대체합니다.

[plan 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)에서 Claude Code는 이 대체를 건너끕니다. 질문 규칙이 없으면 내장 읽기 전용 명령은 여전히 프롬프트 없이 실행되고 기타 다른 셸 명령은 계획을 세우는 동안 승인 프롬프트를 표시합니다. 단순 `Bash` 질문 규칙을 사용하면 샌드박스 바깥과 동일하게 샌드박스 처리된 읽기 전용 명령을 포함하여 모든 Bash 명령이 프롬프트를 표시합니다. v2.1.212 이전에는 대체가 plan 모드에서도 적용되었습니다.

이러한 검사는 여전히 적용됩니다:

* `Bash(git push *)`와 같은 내용 범위의 질문 규칙은 여전히 프롬프트를 강제합니다.
* 명시적인 거부 규칙은 여전히 적용됩니다.
* `/`, 홈 디렉터리 또는 기타 중요한 시스템 경로를 대상으로 하는 `rm` 또는 `rmdir` 명령은 여전히 프롬프트를 트리거합니다.

제외된 명령과 같이 샌드박스 처리되어 실행되지 않는 명령은 평소와 같이 단순 `Bash` 질문 규칙을 준수합니다. 이 동작을 변경하려면 [샌드박스 모드](/docs/en/sandboxing#sandbox-modes)를 참조하세요.

## 관리형 설정

Claude Code 구성에 대한 중앙 집중식 제어가 필요한 조직의 경우 관리자는 사용자나 프로젝트 설정으로 재정의할 수 없는 관리형 설정을 배포할 수 있습니다. 이러한 정책 설정은 일반 설정 파일과 동일한 형식을 따르며 MDM/OS 수준 정책, 관리형 설정 파일, [서버 관리형 설정](/docs/en/server-managed-settings) 또는 자체 호스팅 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)를 통해 전달될 수 있습니다. 전달 메커니즘 및 파일 위치는 [설정 파일](/docs/en/settings#settings-files)을 참조하세요.

### 관리 전용 설정

다음 설정은 관리형 설정에서만 읽힙니다. 사용자나 프로젝트 설정 파일에 배치해도 효과가 없습니다.

| 설정 | 설명 |
| :--- | :--- |
| `allowAllClaudeAiMcps` | `true`인 경우 독점 제어에 의해 억제되는 대신 배포된 `managed-mcp.json`과 함께 claude.ai 커넥터가 로드됩니다. [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. |
| `allowedChannelPlugins` | 메시지를 푸시할 수 있는 채널 플러그인의 허용 목록입니다. 설정 시 기본 Anthropic 허용 목록을 대체합니다. `channelsEnabled: true`가 필요합니다. [실행할 수 있는 채널 플러그인 제한](/docs/en/channels#restrict-which-channel-plugins-can-run)을 참조하세요. |
| `allowManagedHooksOnly` | `true`인 경우 관리형 훅, SDK 훅 및 관리형 설정 `enabledPlugins`에서 강제 활성화된 플러그인의 훅만 로드됩니다. 사용자, 프로젝트 및 기타 모든 플러그인 훅은 차단됩니다. |
| `allowManagedMcpServersOnly` | `true`인 경우 관리형 설정의 `allowedMcpServers`만 수용됩니다. `deniedMcpServers`는 모든 소스에서 여전히 병합됩니다. [관리형 MCP 구성](/docs/en/managed-mcp)을 참조하세요. |
| `allowManagedPermissionRulesOnly` | `true`인 경우 사용자 및 프로젝트 설정이 `allow`, `ask`, `deny` 권한 규칙을 정의하지 못하도록 방지합니다. 관리형 설정의 규칙만 적용됩니다. MCP 서버 허용 목록에는 영향을 미치지 않으며 이를 위해 `allowManagedMcpServersOnly`를 설정하세요. |
| `blockedMarketplaces` | 마켓플레이스 소스의 차단 목록입니다. 차단된 소스는 다운로드 전에 검사되므로 파일 시스템에 전혀 닿지 않습니다. [관리형 마켓플레이스 제한](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)을 참조하세요. |
| `channelsEnabled` | 조직에 대해 [채널](/docs/en/channels)을 허용합니다. 각 플랜의 기본값은 [엔터프라이즈 제어](/docs/en/channels#enterprise-controls)를 참조하세요. |
| `disableSideloadFlags` | {/* min-version: 2.1.193 */}시작 시 `--plugin-dir`, `--plugin-url`, `--agents`, `--mcp-config` CLI 플래그를 거부합니다. 이것이 없으면 사용자는 이러한 플래그를 전달하여 단일 실행에 대해 `strictKnownMarketplaces`를 우회할 수 있습니다. [`disableSideloadFlags`](/docs/en/settings#available-settings)를 참조하세요. Claude Code v2.1.193 이상이 필요합니다. |
| `forceRemoteSettingsRefresh` | `true`인 경우 원격 관리형 설정을 새로 가져올 때까지 CLI 시작을 차단하고 가져오기에 실패하면 종료합니다. [Fail-closed 시행](/docs/en/server-managed-settings#enforce-fail-closed-startup)을 참조하세요. |
| `pluginTrustMessage` | 설치 전에 표시되는 플러그인 신뢰 경고에 추가되는 커스텀 메시지입니다. |
| `sandbox.filesystem.allowManagedReadPathsOnly` | `true`인 경우 관리형 설정의 `filesystem.allowRead` 경로만 수용됩니다. `denyRead`는 모든 소스에서 여전히 병합됩니다. |
| `sandbox.network.allowManagedDomainsOnly` | `true`인 경우 관리형 설정의 `allowedDomains` 및 `WebFetch(domain:...)` 허용 규칙만 수용됩니다. 허용되지 않은 도메인은 사용자에게 묻지 않고 자동으로 차단됩니다. 거부된 도메인은 모든 소스에서 여전히 병합됩니다. |
| `strictKnownMarketplaces` | 사용자가 플러그인을 추가하고 설치할 수 있는 플러그인 마켓플레이스 소스를 제어합니다. [관리형 마켓플레이스 제한](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)을 참조하세요. |
| `strictPluginOnlyCustomization` | 사용자 및 프로젝트 소스에서 스킬, 에이전트, 훅, MCP 서버를 차단하여 플러그인이나 관리형 설정에서만 올 수 있도록 합니다. `true`는 네 가지 영역을 모두 잠그고 `["skills", "hooks"]`와 같은 배열은 지정된 영역만 잠급니다. [`strictPluginOnlyCustomization`](/docs/en/settings#strictpluginonlycustomization)을 참조하세요. |
| `wslInheritsWindowsSettings` | Windows HKLM 레지스트리 키 또는 `C:\Program Files\ClaudeCode\managed-settings.json`에서 `true`인 경우 WSL은 `/etc/claude-code` 외에도 Windows 정책 체인에서 관리형 설정을 읽습니다. [설정 파일](/docs/en/settings#settings-files)을 참조하세요. |

`disableBypassPermissionsMode`는 일반적으로 조직 정책을 시행하기 위해 관리형 설정에 배치되지만 모든 범위에서 작동합니다. 사용자는 우회 모드에서 스스로를 차단하도록 자체 설정에 이를 설정할 수 있습니다.

<Note>
  Team 및 Enterprise 플랜에서 소유자는 [Claude Code 관리자 설정](https://claude.ai/admin-settings/claude-code)에서 조직 전체의 [Remote Control](/docs/en/remote-control) 및 [웹 세션](/docs/en/claude-code-on-the-web)을 활성화하거나 비활성화합니다. Remote Control은 [`disableRemoteControl`](/docs/en/settings#available-settings) 설정으로 장치별로 비활성화할 수도 있습니다. 웹 세션에는 장치별 관리형 설정 키가 없습니다.
</Note>

## 설정 우선순위

권한 규칙은 다른 모든 Claude Code 설정과 동일한 [설정 우선순위](/docs/en/settings#settings-precedence)를 따릅니다:

1. **관리형 설정**: 명령줄 인수를 포함하여 다른 어떤 수준에서도 재정의할 수 없습니다.
2. **명령줄 인수**: 임시 세션 재정의
3. **로컬 프로젝트 설정** (`.claude/settings.local.json`)
4. **공유 프로젝트 설정** (`.claude/settings.json`)
5. **사용자 설정** (`~/.claude/settings.json`)

어떤 수준에서든 도구가 거부되면 다른 어떤 수준에서도 이를 허용할 수 없습니다. 예를 들어 관리형 설정 거부는 `--allowedTools`로 재정의할 수 없으며 `--disallowedTools`는 관리형 설정이 정의한 수준 이상의 제한을 추가할 수 있습니다.

설정 범위 전반에서도 마찬가지입니다. 사용자 설정이 권한을 허용하고 프로젝트 설정이 이를 거부하는 경우 거부 규칙이 이를 차단합니다. 그 반대도 마찬가지입니다. 모든 범위의 거부 규칙이 허용 규칙보다 먼저 평가되므로 사용자 수준 거부는 프로젝트 수준 허용을 차단합니다.

임베딩 호스트는 관리자가 `allowManaged*Only` 잠금을 설정하지 않는 한 권한 허용 규칙을 포함하여 SDK `managedSettings` 옵션을 통해 추가 관리 정책을 제공할 수 있습니다. [Claude Desktop 세션에 정책 전달](/docs/en/claude-apps-gateway#deliver-policy-to-claude-desktop-sessions)에서는 임베더 정책이 적용되는 시기를 다룹니다.

## 프로젝트 허용 규칙 및 워크스페이스 신뢰

프로젝트의 `.claude/settings.json`에 있는 `permissions.allow` 규칙 및 `permissions.additionalDirectories` 항목은 기능을 부여하므로 Claude Code는 해당 워크스페이스에 대한 [워크스페이스 신뢰 대화 상자](/docs/en/security#additional-safeguards)를 수용한 후에만 이를 적용합니다. 그전까지 Claude Code는 규칙을 읽지만 적용하지는 않습니다. 신뢰 대화 상자에는 승인 전에 검토할 수 있도록 폴더가 부여할 허용 규칙과 추가 디렉터리가 나열됩니다. `deny` 및 `ask` 규칙은 제한만 하므로 영향을 받지 않습니다.

Claude Code는 git 리포지토리 루트 또는 리포지토리 외부의 시작 디렉터리를 키로 지정하여 워크스페이스별로 신뢰를 저장합니다. 홈 디렉터리에서 시작할 때 신뢰는 현재 세션에 대해서만 유지되고 디스크에 기록되지 않습니다. [추가 안전장치](/docs/en/security#additional-safeguards) 노트를 참조하세요. 상위 디렉터리를 신뢰한다고 해서 중첩된 프로젝트의 허용 규칙이 적용되지는 않습니다.

`.claude/settings.local.json`은 사용자 고유의 파일이므로 일반적으로 워크스페이스 신뢰 검사가 적용되지 않습니다. git에 커밋되어 있거나 `.claude`가 심볼릭 링크인 경우와 같이 리포지토리가 해당 파일을 제공했을 수 있는 경우 해당 허용 규칙 및 추가 디렉터리는 프로젝트 설정과 마찬가지로 신뢰 검사를 거칩니다.

Claude Code는 git을 실행하여 리포지토리가 파일을 제공했는지 여부를 검사하고 해당 폴더 또는 해당 상위 디렉터리 중 하나에 대해 수용된 신뢰 대화 상자가 다루는 폴더에서만 해당 검사를 실행합니다. 아직 신뢰하지 않은 폴더의 대화형 세션에서 `.claude/settings.local.json`이 아래에 설명된 대로 자체 구성 홈에서 실행되지 않는 한 수용할 때까지 허용 규칙 및 추가 디렉터리는 프로젝트 설정과 마찬가지로 신뢰 검사를 거칩니다. 아래의 두 가지 예외 중 git을 실행할 필요가 없기 때문에 대화 상자 전에 구성 홈 예외만 적용됩니다. 디렉터리가 git 리포지토리 내부에 있지 않음을 판단하는 것은 동일한 git 검사를 사용하므로 폴더를 다루는 신뢰 대화 상자가 수용되면 리포지토리 내부가 아님 예외가 적용됩니다. v2.1.207 이전에는 추적되지 않는 `.claude/settings.local.json`이 대화 상자를 수용하기 전에 해당 폴더에 허용 규칙을 적용했습니다.

`.claude/settings.local.json`의 허용 규칙 및 추가 디렉터리는 두 가지 경우에 워크스페이스 신뢰 없이도 적용됩니다:

* Claude Code를 시작한 디렉터리가 git 리포지토리 내부에 있지 않은 경우.
* 세션이 사용자 고유의 구성 홈(홈 디렉터리 또는 `.claude` 하위 디렉터리를 [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars)로 설정한 디렉터리)에서 실행되는 경우.

두 경우 모두 파일은 리포지토리가 제공할 수 있는 파일이 아니라 직접 만든 파일이며 리포지토리에 커밋된 `.claude/settings.local.json`에는 여전히 워크스페이스 신뢰가 필요합니다. 2.1.196부터 2.1.199까지의 버전은 해당 워크스페이스에서 해당 파일을 리포지토리가 제공한 것으로 취급하고 허용 규칙을 무시하며 stderr에 [`this workspace has not been trusted`](/docs/en/errors#workspace-has-not-been-trusted) 경고를 출력했습니다. 위의 두 예외는 v2.1.195 및 이전 버전과 일치하며 v2.1.200에서 복원되었습니다.

또한 v2.1.200부터 허용 규칙이나 추가 디렉터리가 여전히 적용되지 않지만 상위 디렉터리가 이미 신뢰되었기 때문에 신뢰 대화 상자를 표시하지 않은 워크스페이스는 다음에 대화형으로 Claude Code를 시작할 때 대화 상자를 표시합니다. 대화 상자는 두 가지 선택을 제공합니다:

* **Yes, I trust this folder**: 해당 워크스페이스에 대한 신뢰를 저장하고 동일한 세션에 규칙을 적용합니다.
* **No, continue without these permissions**: 해당 규칙을 무시한 채 작업을 계속합니다. 대화 상자는 다음 세션에 다시 나타납니다.

`-p`가 있는 [비대화형 모드](/docs/en/headless)에서는 대화 상자가 나타나지 않으며 규칙은 무시된 상태로 유지됩니다.

## 예시 구성

이 [리포지토리](https://github.com/anthropics/claude-code/tree/main/examples/settings)에는 일반적인 배포 시나리오를 위한 기본 설정 구성이 포함되어 있습니다. 이를 시작점으로 사용하여 필요에 맞게 조정하세요.

## 관련 정보

* [설정](/docs/en/settings): 권한 설정 표를 포함한 전체 구성 참조
* [자동 모드 구성](/docs/en/auto-mode-config): 조직이 신뢰하는 인프라를 자동 모드 분류기에 전달
* [샌드박싱](/docs/en/sandboxing): Bash 명령에 대한 OS 수준의 파일 시스템 및 네트워크 격리
* [인증](/docs/en/authentication): Claude Code에 대한 사용자 접근 설정
* [보안](/docs/en/security): 보안 안전장치 및 모범 사례
* [훅](/docs/en/hooks-guide): 워크플로 자동화 및 권한 평가 확장
