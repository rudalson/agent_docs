> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 워크트리를 사용해 병렬 세션 실행하기

> 변경 사항이 충돌하지 않도록 별도의 git 워크트리에서 병렬 Claude Code 세션을 격리합니다. `--worktree` 플래그, 서브에이전트 격리, `.worktreeinclude`, 정리 및 비-git VCS 훅에 대해 다룹니다.

A [git worktree](https://git-scm.com/docs/git-worktree)는 기본 체크아웃과 동일한 리포지토리 기록 및 원격 저장소를 공유하면서 고유한 파일과 브랜치를 가지는 별도의 작업 디렉토리입니다. 각 Claude Code 세션을 자체 워크트리에서 실행하면 한 세션의 편집 내용이 다른 세션의 파일에 영향을 주지 않으므로, 한 세션에서 기능을 개발하는 동안 다른 세션에서 버그를 수정할 수 있습니다.

<Note>
  워크트리를 사용하려면 git 리포지토리가 필요합니다; 다른 버전 관리 시스템의 경우 [git 로직을 대체할 훅을 구성](#non-git-version-control)하세요. [데스크톱 앱](/docs/en/desktop#work-in-parallel-with-sessions)에서는 모든 새 세션이 자동으로 자체 워크트리를 받습니다.
</Note>

워크트리는 Claude를 병렬로 실행하는 여러 방법 중 하나입니다. 워크트리는 파일 편집을 격리하는 반면, [서브에이전트](/docs/en/sub-agents) 및 [에이전트 팀](/docs/en/agent-teams)은 작업 자체를 조율합니다. 여러 접근 방식을 비교하려면 [에이전트 병렬 실행](/docs/en/agents)을 참조하거나, 워크트리와 서브에이전트를 함께 사용하려면 [워크트리로 서브에이전트 격리](#isolate-subagents-with-worktrees)로 바로 이동하세요.

대부분의 세션에는 처음 두 섹션만 필요합니다: [워크트리에서 Claude 시작하기](#start-claude-in-a-worktree) 후 [종료 시 정리하기](#clean-up-worktrees). [세션 재개](#resume-a-worktree-session), [워크트리 생성 방식 변경](#customize-worktree-creation), 또는 [오류 디버깅](#troubleshooting)이 필요할 때 나머지 내용을 읽어보세요.

## 워크트리에서 Claude 시작하기

`--worktree` 또는 `-w`와 함께 이름을 전달하여 격리된 워크트리를 생성하고 그 안에서 Claude를 시작하세요. 기본적으로 워크트리는 리포지토리 루트 하위의 `.claude/worktrees/<name>/`에 생성되며 `worktree-<name>`이라는 새 브랜치에 위치합니다:

```bash theme={null}
claude --worktree feature-auth
```

다른 터미널에서 다른 이름으로 이 명령을 다시 실행하여 두 번째 격리된 세션을 시작하세요. 이름을 생략하면 Claude가 `bright-running-fox`와 같은 이름을 생성합니다.

대화형 실행에는 [작업 공간 신뢰](/docs/en/security)가 필요합니다: 이전에 해당 디렉토리에서 Claude를 실행한 적이 없다면 거기에서 `claude`를 한 번 실행하여 신뢰 대화 상자를 승인해야 합니다. 그렇지 않으면 `--worktree`가 승인을 재촉하는 오류와 함께 종료됩니다. `-p`를 사용한 비대화형 실행은 신뢰 검사를 건너뛰므로 `claude -p --worktree`는 신뢰 검사 없이 진행됩니다.

<Tip>
  워크트리 내용이 메인 체크아웃에서 추적되지 않는 파일로 표시되지 않도록 `.gitignore`에 `.claude/worktrees/`를 추가하세요.
</Tip>

### 워크트리 환경 설정

워크트리는 새로 시작하는 체크아웃이므로 거기에서 개발 환경을 초기화하세요: Claude에게 의존성 설치를 요청하거나 `.claude/worktrees/` 하위의 워크트리 디렉토리에서 직접 프로젝트 설정을 실행하세요. `.env`와 같은 gitignore 처리된 파일을 모든 새 워크트리에 자동으로 전달하려면 [`.worktreeinclude` 파일](#copy-gitignored-files-into-worktrees)을 추가하세요.

### Claude에게 워크트리 생성 요청하기

세션 도중 Claude에게 "work in a worktree"(워크트리에서 작업해 줘)라고 요청할 수도 있으며, Claude는 [`EnterWorktree`](/docs/en/tools-reference) 도구를 사용하여 워크트리를 만듭니다. 워크트리에 진입하면 Claude는 대상 경로로 `EnterWorktree`를 호출하여 `.claude/worktrees/` 하위의 다른 워크트리로 직접 전환할 수 있습니다. 이전 워크트리는 디스크에 수정되지 않은 채 남습니다.

Claude가 리포지토리의 `.claude/worktrees/` 디렉토리 외부 경로에 진입할 때 Claude Code는 먼저 사용자의 승인을 요청합니다. 이 이동으로 인해 세션의 작업 디렉토리, 쓰기 권한, `CLAUDE.md` 및 설정과 같은 프로젝트 구성이 해당 위치로 이동하기 때문입니다. `EnterWorktree` [권한 규칙](/docs/en/permissions)이나 "don't ask again" 선택으로도 이 프롬프트를 생략할 수 없으며, `bypassPermissions` 모드만 건너뜁니다. v2.1.206 이전에는 Claude가 묻지 않고 기존의 모든 워크트리 경로에 진입할 수 있었습니다.

## 워크트리 정리

대화형 워크트리 세션을 종료하면 Claude는 삭제 시 없어질 작업(변경된 파일 또는 추적되지 않은 파일, 새 커밋)이 워크트리에 있는지 확인합니다.

* **워크트리가 깨끗함**: 이름 없는 세션의 경우 Claude가 워크트리와 해당 브랜치를 자동으로 제거합니다. [이름이 지정된](/docs/en/sessions#name-your-sessions) 세션은 나중에 워크트리를 유지할 수 있도록 먼저 프롬프트를 표시합니다
* **워크트리에 작업물이 있음**: Claude가 워크트리를 유지할지 제거할지 묻는 프롬프트를 표시합니다. 유지를 선택하면 디렉토리와 브랜치가 보존되어 나중에 돌아올 수 있습니다. 제거를 선택하면 워크트리 디렉토리 및 브랜치와 함께 그 안의 모든 작업물이 삭제됩니다

`-p`를 사용한 비대화형 실행은 종료 프롬프트가 없으므로 Claude가 워크트리를 정리하지 않습니다. `git worktree remove`로 직접 제거하세요.

Windows에서 워크트리를 제거해도 외부 파일은 삭제되지 않습니다. 워크트리 내부의 폴더가 실제로 NTFS 정션이나 디렉토리 심볼릭 링크와 같이 다른 곳을 가리키는 링크인 경우 Claude Code는 링크만 삭제하고 가리키는 폴더는 유지합니다. v2.1.205 이전에는 하위 디렉토리에 중첩된 링크가 있는 워크트리를 제거하면 가리키던 폴더가 삭제될 수 있었습니다.

## 워크트리 세션 재개

워크트리 내부에 있었던 세션을 재개하면 Claude Code는 해당 세션을 해당 워크트리로 다시 돌려놓습니다. 이는 대화형 재개, `-p`를 사용한 [비대화형 모드](/docs/en/headless)에서의 `--continue` 및 `--resume`, Agent SDK 모두에 해당합니다. 워크트리로 돌아온 후에도 Claude는 [`ExitWorktree`](/docs/en/tools-reference) 도구를 사용하여 워크트리를 빠져나올 수 있습니다.

`--fork-session`으로 세션을 포크하여 재개하면 Claude를 실행한 디렉토리에서 시작되며 원래 세션의 워크트리는 유지됩니다. 워크트리 디렉토리가 더 이상 존재하지 않는 경우 세션은 Claude를 실행한 디렉토리에서 재개됩니다.

<Note>
  v2.1.212 이전에는 비대화형 재개가 시작 디렉토리에 유지되었으며 `ExitWorktree`는 빠져나올 활성 워크트리 세션이 없다고 보고했습니다.
</Note>

{/* min-version: 2.1.198 */}Claude가 Claude Code에서 git으로 생성한 워크트리에 진입하거나 빠져나올 때 트랜스크립트도 이를 따릅니다: Claude Code는 [`/cd`](/docs/en/commands)와 동일하게 세션의 새 작업 디렉토리 아래에 세션을 기록하므로 `/desktop` 및 `--resume`이 거기에서 세션을 찾습니다. 빠져나올 때도 동일하게 돌아갑니다. [`WorktreeCreate` 훅](#non-git-version-control)으로 생성된 워크트리는 시작 디렉토리에 트랜스크립트를 유지합니다. Claude Code v2.1.198 이상이 필요합니다.

## 워크트리로 서브에이전트 격리

병렬 편집이 충돌하지 않도록 서브에이전트를 자체 워크트리에서 실행할 수 있습니다. Claude에게 "use worktrees for your agents"(에이전트에 워크트리를 사용해 줘)라고 요청하거나 [커스텀 서브에이전트](/docs/en/sub-agents#supported-frontmatter-fields)의 프론트매터에 `isolation: worktree`를 추가하여 격리를 영구화하세요.

`.claude/agents/` 내의 이 서브에이전트는 항상 자체 워크트리에서 실행됩니다:

```markdown theme={null}
---
name: refactorer
description: Applies mechanical refactors across many files
isolation: worktree
---

Apply the requested refactor across every affected file, then run the tests
and report the results.
```

각 서브에이전트는 Claude Code가 생성하는 임시 워크트리를 받으며, 서브에이전트가 변경 없이 완료되면 자동으로 제거됩니다. 변경 사항이 있는 워크트리는 [아래의 주기적인 정리](#clean-up-subagent-and-background-session-worktrees)에서 작업 손실 없이 제거할 수 있을 때까지 디스크에 유지됩니다.

서브에이전트 워크트리는 `--worktree`와 동일한 [기본 브랜치](#choose-the-base-branch)를 사용하므로 `worktree.baseRef`가 `"head"`로 설정되지 않는 한 리포지토리의 기본 브랜치에서 분기합니다.

### 서브에이전트 및 백그라운드 세션 워크트리 정리

주기적인 정리를 통해 Claude가 서브에이전트 및 [백그라운드 세션](/docs/en/agent-view#how-file-edits-are-isolated)을 위해 생성한 워크트리가 [`cleanupPeriodDays`](/docs/en/settings#available-settings) 설정보다 오래되면 제거됩니다. 아직 작업물(변경된 파일 또는 추적되지 않은 파일, 푸시되지 않은 커밋)이 있는 워크트리는 정리를 건너뜁니다. `--worktree`로 직접 생성한 워크트리는 절대로 제거하지 않습니다.

에이전트가 실행되는 동안 Claude는 워크트리에 `git worktree lock`을 실행하여 동시 정리가 이를 제거하지 못하게 합니다. 잠금은 에이전트가 완료되면 해제됩니다.

{/* min-version: 2.1.210 */}또한 정리는 프로세스가 종료된 세션에 대해 Claude Code가 설정한 잠금을 해제하므로 강제 종료된 백그라운드 세션이 워크트리를 영구적으로 잠근 상태로 두지 않습니다. 정리는 직접 `git worktree lock`으로 설정한 잠금은 해제하지 않습니다. v2.1.210 이전에는 강제 종료된 세션이 남긴 잠금이 `git worktree unlock`을 실행할 때까지 제자리에 남아 있었습니다.

정리가 유지하는 워크트리를 정리하려면 `git worktree remove`를 실행하고, 커밋되지 않은 변경 사항이나 추적되지 않는 파일이 있는 경우 `--force`를 추가하세요.

## 워크트리 생성 맞춤설정

워크트리 생성을 위한 Claude Code의 기본값은 대부분의 세션을 충족합니다: `.claude/worktrees/` 하위에 만들고 리포지토리의 기본 브랜치에서 분기하며 추적 대상 파일만 체크아웃합니다. 이 섹션의 옵션은 해당 기본값을 변경합니다.

### 기본 브랜치 선택

새 워크트리는 리포지토리의 기본 브랜치에서 분기되므로 대부분의 세션에는 이 설정이 필요하지 않습니다. 대신 현재 작업에서 분기하려면 [설정](/docs/en/settings#worktree-settings)에서 `worktree.baseRef`를 설정하세요. 이 설정은 두 가지 값을 받습니다:

* `"fresh"` (기본값): 원격의 리포지토리 기본 브랜치(보통 `main`)에서 분기하여 원격과 일치하는 깨끗한 트리에서 워크트리가 시작됩니다.
* `"head"`: 현재 로컬 `HEAD`에서 분기하므로 워크트리에 푸시되지 않은 커밋과 기능 브랜치 상태가 전달됩니다. 진행 중인 작업에서 작동해야 하는 서브에이전트를 격리할 때 이를 사용하세요. 워크트리 내부에서 `"head"`는 메인 체크아웃의 HEAD가 아니라 해당 워크트리의 `HEAD`로 해석됩니다.

`worktree.baseRef`를 브랜치 이름으로 설정할 수는 없습니다. 기존 특정 브랜치에서 워크트리를 시작하려면 [git을 사용해 직접 만드세요](#manage-worktrees-manually).

`"fresh"` 기본값의 경우 Claude Code는 `origin/HEAD`를 최신 상태로 유지합니다: 지난 24시간 동안 리포지토리를 가져오지 않은 경우 최대 5초 한도로 기본 브랜치를 가져오며 가져오기에 실패하면 로컬에 캐시된 ref를 사용합니다. 원격이 구성되지 않았거나 `origin/HEAD`가 로컬에 캐시되어 있지 않고 가져올 수 없는 경우 워크트리는 현재 로컬 `HEAD`로 대체됩니다. v2.1.208 이전에는 fresh 워크트리가 로컬에 이미 캐시된 `origin/HEAD`를 사용했습니다.

다음 예시는 모든 새 워크트리가 현재 작업에서 분기하도록 설정합니다:

```json theme={null}
{
  "worktree": {
    "baseRef": "head"
  }
}
```

### 풀 리퀘스트에서 분기

특정 풀 리퀘스트에서 분기하려면 PR 번호 앞에 `#`을 붙이거나 전체 GitHub 풀 리퀘스트 URL을 `--worktree`에 전달하세요. Claude Code는 `origin`에서 `pull/<number>/head`를 가져와 `.claude/worktrees/pr-<number>`에 워크트리를 생성합니다. 쉘이 `#`을 주석의 시작으로 취급하지 않도록 인자를 따옴표로 감싸세요:

```bash theme={null}
claude --worktree "#1234"
```

### gitignored 파일을 워크트리에 복사

워크트리는 새로 시작하는 체크아웃이므로 메인 리포지토리의 `.env` 또는 `.env.local`과 같은 추적되지 않은 파일이 존재하지 않습니다. Claude가 워크트리를 생성할 때 이 파일들을 자동으로 복사하려면 프로젝트 루트에 `.worktreeinclude` 파일을 추가하세요.

이 파일은 `.gitignore` 구문을 사용합니다. 패턴과 일치하고 동시에 gitignore에 지정된 파일만 복사되므로 추적 대상 파일은 중복 복사되지 않습니다.

이 `.worktreeinclude` 파일은 2개의 env 파일과 하나의 비밀정보 구성을 새 워크트리마다 복사합니다:

```text .worktreeinclude theme={null}
.env
.env.local
config/secrets.json
```

이는 Claude Code가 git으로 생성하는 모든 워크트리에 적용됩니다: `--worktree` 워크트리, [서브에이전트 워크트리](#isolate-subagents-with-worktrees), [데스크톱 앱](/docs/en/desktop#work-in-parallel-with-sessions)의 병렬 세션. [`WorktreeCreate` 훅](#non-git-version-control)을 사용하는 경우 훅 스크립트 내부에서 파일을 복사하세요.

### 워크트리 이름 재사용

디렉토리가 이미 존재하는 이름을 `--worktree`에 전달하면 새 워크트리를 만드는 대신 기존 워크트리를 엽니다.

기본값인 `"fresh"` [기본 브랜치](#choose-the-base-branch)를 사용할 때 재오픈된 워크트리는 다음 조건이 모두 충족되면 이전 팁에서 계속 진행하는 대신 리포지토리의 기본 브랜치로 리셋됩니다:

* 커밋되지 않은 변경 사항이나 추적되지 않은 파일이 없는 경우.
* Claude Code가 생성한 브랜치에 여전히 있는 경우.
* 자체 커밋이 없거나, 풀 리퀘스트가 병합되고 원격 브랜치가 삭제된 경우.

Claude Code는 git 상태만으로 병합된 경우를 감지합니다: 워크트리가 푸시한 원격 브랜치가 더 이상 존재하지 않으며 워크트리의 모든 커밋이 이미 기본 브랜치에 있는 경우입니다.

다른 경우에는 이전 팁에서 다시 열립니다: 조건 중 하나라도 실패한 워크트리, 상태를 확인할 수 없는 워크트리, `worktree.baseRef`가 `"head"`이거나 이름이 풀 리퀘스트 번호인 재사용 시. v2.1.208 이전에는 재사용된 이름이 항상 이전 워크트리를 이전 팁에서 재오픈했습니다.

### 훅으로 워크트리 생성 대체

기본 `git worktree` 로직을 완전히 대체하도록 [`WorktreeCreate` 훅](/docs/en/hooks#worktreecreate)을 구성하여 `.claude/worktrees/` 이외의 다른 곳에 워크트리를 배치할 수도 있습니다. 완전한 예시는 [비-git 버전 관리](#non-git-version-control)를 참조하세요.

## 워크트리가 메인 체크아웃과 공유하는 항목

워크트리는 고유한 파일과 브랜치를 받지만 리포지토리의 `.git` 디렉토리, 프로젝트 범위의 플러그인, 저장된 권한 승인을 메인 체크아웃과 공유합니다:

* **리포지토리의 `.git` 디렉토리**: 워크트리에서의 git 명령은 메인 리포지토리의 공유 `.git` 디렉토리에 기록되며, [샌드박싱](/docs/en/sandboxing#filesystem-isolation)은 이러한 쓰기를 허용하므로 샌드박스가 활성화된 워크트리 내부에서도 `git commit`과 같은 명령이 작동합니다.
* {/* min-version: 2.1.200 */}**플러그인**: 메인 체크아웃에서 [프로젝트 범위](/docs/en/plugins-reference#plugin-installation-scopes)로 설치된 플러그인은 동일한 리포지토리의 워크트리에서도 로드되므로 워크트리마다 다시 설치할 필요가 없습니다. Claude Code v2.1.200 이상이 필요합니다.
* {/* min-version: 2.1.211 */}**권한 승인**: 워크트리 세션에서 Bash 명령에 대해 "Yes, don't ask again"을 선택하면 승인 규칙이 메인 체크아웃의 `.claude/settings.local.json`에 저장되므로 메인 체크아웃과 리포지토리의 다른 모든 워크트리에 적용되며 워크트리가 제거되어도 유지됩니다. v2.1.211 이전에는 워크트리에서 부여된 승인이 해당 워크트리 내부에 저장되어 다른 곳에 적용되지 않았고 워크트리 제거 시 손실되었습니다. [승인이 저장되는 위치](/docs/en/permissions#permission-system)를 참조하세요.

세 가지 모두 `--worktree`, `git worktree add` 또는 [데스크톱 앱](/docs/en/desktop#work-in-parallel-with-sessions)을 통해 워크트리를 생성하는 경우에 모두 적용됩니다.

## 수동으로 워크트리 관리

기존 특정 브랜치를 체크아웃하거나 워크트리를 리포지토리 외부에 배치해야 할 때는 Git으로 직접 워크트리를 생성하세요.

새 브랜치에 워크트리 생성:

```bash theme={null}
git worktree add ../project-feature-a -b feature-a
```

기존 브랜치에서 워크트리 생성:

```bash theme={null}
git worktree add ../project-bugfix bugfix-123
```

워크트리에서 Claude 시작:

```bash theme={null}
cd ../project-feature-a
claude
```

워크트리 나열:

```bash theme={null}
git worktree list
```

작업이 끝났을 때 제거:

```bash theme={null}
git worktree remove ../project-feature-a
```

전체 명령 참조는 [Git worktree 문서](https://git-scm.com/docs/git-worktree)를 참조하세요.

## 비-git 버전 관리

워크트리 격리는 기본적으로 git을 사용합니다. SVN, Perforce, Mercurial 또는 기타 시스템의 경우 커스텀 생성 및 정리 로직을 제공하도록 [`WorktreeCreate` 및 `WorktreeRemove` 훅](/docs/en/hooks#worktreecreate)을 구성하세요. 훅이 기본 git 동작을 대체하기 때문에 `--worktree`를 사용할 때 [`.worktreeinclude`](#copy-gitignored-files-into-worktrees)는 처리되지 않습니다. 대신 훅 스크립트 내부에서 로컬 구성 파일을 복사하세요.

이 `WorktreeCreate` 훅은 `jq`를 사용해 stdin의 JSON에서 워크트리 이름을 읽고, 새로운 SVN 작업 사본을 체크아웃하며, Claude Code가 이를 세션의 작업 디렉토리로 사용할 수 있도록 디렉토리 경로를 출력합니다. [`settings.json`](/docs/en/settings#settings-files)에 구성을 추가하세요:

```json theme={null}
{
  "hooks": {
    "WorktreeCreate": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'NAME=$(jq -r .name); DIR=\"$HOME/.claude/worktrees/$NAME\"; svn checkout https://svn.example.com/repo/trunk \"$DIR\" >&2 && echo \"$DIR\"'"
          }
        ]
      }
    ]
  }
}
```

세션이 종료될 때 정리할 수 있도록 `WorktreeRemove` 훅과 짝을 이루세요. 입력 스키마 및 제거 예시는 [훅 참조](/docs/en/hooks#worktreecreate)를 참조하세요.

## 문제 해결

아래 오류는 Claude Code가 워크트리를 생성하거나 시작 시 워크트리에 진입할 때 발생합니다.

### Claude Code가 시작 시 워크트리에 진입할 수 없음

Claude Code가 시작할 때 워크트리 디렉토리에 진입할 수 없는 경우 경로를 지정하는 오류를 출력하고 코드 1로 종료합니다. 이는 [`WorktreeCreate` 훅](/docs/en/hooks#worktreecreate)이 생성한 디렉토리가 아닌 다른 것을 출력하거나 설정 후 디렉토리가 삭제된 경우에 발생할 수 있습니다. v2.1.205 이전에는 세션이 충돌했으며 `-p`에서는 약 30초 동안 멈췄다가 코드 0으로 종료되었습니다.

### 심볼릭 링크 경로에서 워크트리 생성 실패

Claude Code는 `.claude`, `.claude/worktrees` 또는 워크트리 디렉토리 자체가 심볼릭 링크인 경우 워크트리 생성을 거부하며 오류는 심볼릭 링크 경로를 나타냅니다. 심볼릭 링크를 제거하고 다시 시도하세요. v2.1.212 이전에는 리포지토리에 해당 경로 중 하나에 커밋된 심볼릭 링크가 이미 포함되어 있는 경우 워크트리 생성이 이를 따라가서 리포지토리 외부에 파일을 생성할 수 있었습니다.

## 참고 항목

워크트리는 파일 격리를 처리합니다. 관련 아래 페이지는 격리된 체크아웃으로 작업을 위임하고 생성한 세션 간을 전환하는 방법을 다룹니다:

* [서브에이전트](/docs/en/sub-agents): 세션 내에서 격리된 에이전트로 작업 위임
* [에이전트 팀](/docs/en/agent-teams): 여러 Claude 세션을 자동으로 조율
* [세션 관리](/docs/en/sessions): 대화 이름 지정, 재개 및 전환
* [Desktop 병렬 세션](/docs/en/desktop#work-in-parallel-with-sessions): 데스크톱 앱에서 워크트리 기반 세션
