> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 세션 관리하기

> Claude Code 대화의 이름 지정, 다시 시작, 분기 및 세션 간 전환을 다룹니다. `--continue`, `--resume`, `--from-pr`, `/resume` 선택기, 세션 이름 지정, 트랜스크립트 내보내기, 트랜스크립트 저장 위치가 포함됩니다.

세션은 프로젝트 디렉터리에 연결되어 저장된 대화입니다. Claude Code는 작업하는 동안 세션을 로컬에 지속적으로 저장하므로 작업을 중단한 부분부터 다시 시작하거나, 다른 접근 방식을 시도하기 위해 분기하거나, 작업 간 전환을 수행할 수 있습니다.

[데스크톱 앱](/docs/en/desktop#work-in-parallel-with-sessions), [웹 기반 Claude Code](/docs/en/claude-code-on-the-web), [VS Code 확장 프로그램](/docs/en/vs-code#resume-past-conversations)은 각자 고유한 세션 기록을 유지 관리합니다. 이 페이지에서는 CLI를 다룹니다.

## 세션 다시 시작 (Resume)

세션은 작업하는 동안 [로컬 트랜스크립트 파일](#세션-데이터-내보내기-및-위치-찾기)에 지속적으로 저장되므로 종료하거나 `/clear`를 실행한 후에도 세션으로 돌아올 수 있습니다. 다음 진입점을 사용하세요:

| 명령어 | 기능 |
| :--- | :--- |
| `claude --continue` | 현재 디렉터리에서 가장 최근 세션을 다시 시작합니다. |
| `claude --resume` | [세션 선택기](#세션-선택기-사용하기)를 엽니다. |
| `claude --resume <name>` | 지정된 이름의 세션을 직접 다시 시작합니다. |
| `claude --from-pr <number>` | 해당 풀 리퀘스트에 연결된 세션으로 필터링된 세션 선택기를 엽니다. |
| `/resume` | 활성 세션 내부에서 다른 대화로 전환합니다. |

[`claude -p`](/docs/en/headless) 또는 [Agent SDK](/docs/en/agent-sdk/overview)로 생성된 세션은 세션 선택기에 나타나지 않지만, `claude --resume <session-id>`에 세션 ID를 전달하여 다시 시작할 수 있습니다. 세션이 시작된 디렉터리에서 이를 실행하세요. 세션 ID 조회의 스코프는 현재 프로젝트 디렉터리 및 해당 Git 작업 트리(worktree)로 제한되므로 다른 곳에서 생성된 세션은 `No conversation found with session ID: <session-id>`를 보고합니다.

### 다시 시작된 세션이 복원하는 항목

다시 시작된 세션은 저장된 상태와 함께 대화를 복원합니다:

* 대화 기록: 도구 호출 및 결과를 포함한 전체 기록.
* 모델: 세션은 사용 중이던 모델에서 계속됩니다. 모델이 사용 중단되었거나 `availableModels`에 의해 허용되지 않는 경우, 시작 시 `--model` 플래그 또는 `ANTHROPIC_MODEL` 계열 환경 변수가 하나를 선택하는 경우, 또는 [Amazon Bedrock, Google Cloud's Agent Platform 및 Microsoft Foundry](/docs/en/third-party-integrations)와 같이 제공업체 특정 배포 ID를 사용하는 제공업체에서는 모델이 복원되지 않습니다. 확인 순서는 [모델 구성](/docs/en/model-config#setting-your-model)을 참조하세요.
* 에이전트: [`--agent`](/docs/en/sub-agents#invoke-subagents-explicitly) 또는 `agent` 설정으로 시작된 세션은 해당 에이전트로 계속되어 시스템 프롬프트, 도구 제한 및 모델을 유지합니다. 다른 에이전트를 선택하려면 다시 시작할 때 `--agent`를 전달하세요. {/* min-version: 2.1.216 */}에이전트가 세션의 원래 디렉터리나 다시 시작하는 디렉터리에 더 이상 존재하지 않는 경우 기본 도구 및 시스템 프롬프트로 세션이 다시 시작되고 [에이전트 이름을 지정하는 경고](/docs/en/errors#session-agent-no-longer-available)가 표시됩니다.
* 권한 모드: 세션이 있었던 모드입니다. `plan` 및 `bypassPermissions`는 복원되지 않습니다. [권한 우회](/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode)는 시작 플래그 중 하나 또는 [설정](/docs/en/settings#permission-settings)의 `permissions.defaultMode: "bypassPermissions"`를 사용하여 시작 시 다시 활성화해야 합니다. `auto`는 계정이 여전히 [자동 모드 요구 사항](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)을 충족하는 경우에만 복원됩니다. 복원된 모드를 재정의하려면 `--permission-mode`를 전달하세요.
* 활성 목표: 세션이 종료될 때 여전히 활성화되어 있던 [목표(goal)](/docs/en/goal#resume-with-an-active-goal)가 이월되며, 차례 수, 타이머 및 토큰 지출 기준선이 재설정됩니다.
* 예약된 작업: [만료되지 않은 작업](/docs/en/scheduled-tasks#limitations)이 복원됩니다. 백그라운드 Bash 및 모니터 작업은 복원되지 않습니다.

원래 시작 시의 모든 구성 플래그가 복원되는 것은 아닙니다. 세션이 `--mcp-config`, `--settings`, `--plugin-dir`, `--fallback-model` 또는 `--add-dir`로 추가된 디렉터리에 의존하는 경우 다시 시작할 때 해당 값을 다시 전달하세요. 세션 중간에 `/add-dir`로 추가된 디렉터리도 복원되지 않지만 세션 선택기는 여전히 해당 디렉터리를 사용하여 세션을 찾습니다. `settings.json` 및 `settings.local.json`과 같은 표준 설정 파일은 시작 시 다시 읽히므로 해당 파일에 존재하는 구성은 다시 전달할 필요가 없습니다.

### 세션 선택기가 검색하는 위치

Claude Code는 프로젝트 디렉터리별로 세션을 저장합니다. 기본적으로 세션 선택기는 다음을 표시합니다:

* 목록에서 `bg`로 표시되는 [백그라운드 세션](/docs/en/agent-view)을 포함한 현재 작업 트리의 세션
* 다른 곳에서 시작되었지만 `/add-dir`로 현재 디렉터리를 추가한 세션

`Ctrl+W`를 사용하여 리포지토리의 모든 작업 트리로 확장하거나 `Ctrl+A`를 사용하여 이 머신의 모든 프로젝트로 확장하세요.

{/* min-version: 2.1.211 */}첫 프롬프트가 [`/loop`](/docs/en/scheduled-tasks#run-a-prompt-repeatedly-with-%2Floop) 명령이었던 세션은 선택기에 나타나지 않습니다. 대화 중에 나중에 `/loop`를 실행하더라도 세션이 숨겨지지는 않습니다. v2.1.211 이전에는 대화 초기에 `/loop`를 실행하면 세션 선택기에서 세션이 영구적으로 숨겨졌습니다.

{/* min-version: 2.1.169 */}v2.1.169부터 [`/cd`](/docs/en/commands)로 세션을 이동하면 해당 세션이 새 디렉터리의 프로젝트 저장소로 재배치되므로 이후 해당 디렉터리의 선택기에 나타납니다. {/* min-version: 2.1.196 */}v2.1.196부터 이동된 세션은 비정상 종료나 강제 종료 후에도 이전 디렉터리의 선택기에 나타나지 않습니다. 이전 버전에서는 이전 경로에 밑줄과 같은 특수 문자가 포함된 경우 비정상 종료 후 이전 디렉터리 목록에 다시 나타날 수 있었습니다.

동일한 리포지토리의 다른 작업 트리에서 세션을 선택하면 제자리에서 다시 시작됩니다. 관련 없는 프로젝트에서 세션을 선택하면 대신 `cd` 및 resume 명령이 클립보드에 복사됩니다.

이름으로 다시 시작하면 현재 리포지토리 및 해당 작업 전체에 걸쳐 확인됩니다. 두 형태 모두 정확한 일치를 찾고 다른 작업 트리에 있더라도 직접 다시 시작합니다:

| 명령어 | 정확한 일치 | 모호한 이름 |
| :--- | :--- | :--- |
| `claude --resume <name>` | 직접 다시 시작 | 검색어로 이름이 미리 입력된 상태로 세션 선택기를 엽니다. |
| `/resume <name>` | 직접 다시 시작 | 오류를 보고합니다. 인수 없이 `/resume`을 실행하여 세션 선택기를 엽니다. |

## 세션 이름 지정하기

세션 선택기에서 쉽게 찾고 이름으로 다시 시작할 수 있도록 세션에 설명이 포함된 이름을 부여하세요. 이는 여러 작업을 병렬로 수행할 때 가장 중요합니다.

| 시점 | 이름 설정 방법 |
| :--- | :--- |
| 시작 시 | `claude -n auth-refactor` |
| 세션 진행 중 | `/rename auth-refactor`. 이름은 프롬프트 바에도 나타납니다. |
| 세션 선택기에서 | 세션을 강조 표시하고 `Ctrl+R` 누르기 |
| 플랜 승인 시 | 이미 이름을 설정하지 않은 경우 [플랜 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)에서 플랜을 승인하면 플랜 내용에서 세션 이름이 지정됩니다. |

세션 이름이 지정되면 `claude --resume <name>` 또는 `/resume <name>`을 사용하여 다시 돌아옵니다. 작업 트리 전체에서 이름 확인이 작동하는 방식은 [세션 다시 시작](#세션-다시-시작-resume)을 참조하세요.

{/* min-version: 2.1.196 */}이름을 직접 지정하지 않은 대화형 세션도 시작 시 기본 표시 이름을 받습니다. Claude Code v2.1.196 이상이 필요합니다. 기본값은 작업 디렉터리의 이름과 2자의 접미사를 결합하며(예: `my-app-3f`), [에이전트 뷰](/docs/en/agent-view) 및 `claude agents --json` 출력과 같은 실행 중인 세션 목록에서 세션을 식별합니다.

기본값은 다시 시작 핸들이 아닙니다. `claude --resume <name>`, `/resume <name>` 및 세션 선택기는 설정한 이름과만 일치합니다. 세션 이름을 직접 지정하면 기본값을 대체합니다.

세션 이름을 지정하지 않으면 Claude Code가 세션 제목을 생성합니다. 이는 소형/고속 모델(일반적으로 Haiku 클래스 모델)에 대한 백그라운드 요청을 통해 작성된 첫 번째 프롬프트의 짧은 요약입니다. `--name` 또는 `/rename`으로 세션 이름을 지정하면 생성된 제목이 대체됩니다. 이름이 설정되지 않은 경우 [세션 선택기](#세션-선택기-사용하기) 및 상태 표시줄의 [`session_name`](/docs/en/statusline) 필드에서 생성된 제목을 볼 수 있으며, 기본 표시 이름과 마찬가지로 다시 시작 핸들이 아닙니다.

## 세션 선택기 사용하기

세션 내부에서 `/resume`을 실행하거나 인수 없이 `claude --resume`을 실행하여 대화형 세션 선택기를 엽니다. 다음 키보드 단축키를 사용하여 탐색, 검색 및 목록 확장을 수행하세요:

| 단축키 | 작업 |
| :--- | :--- |
| `↑` / `↓` | 세션 간 탐색 |
| `→` / `←` | 그룹화된 세션 펼치기 또는 접기 |
| `Enter` | 강조 표시된 세션 다시 시작 |
| `Space` | 세션 내용 미리보기. 터미널이 붙여넣기로 캡처하지 않는 경우 `Ctrl+V`도 작동합니다. |
| `Ctrl+R` | 강조 표시된 세션 이름 변경 |
| `/` 또는 `Space` 이외의 인쇄 가능한 문자 | 검색 모드로 진입하여 세션 필터링. GitHub, GitHub Enterprise, GitLab 또는 Bitbucket 풀/머지 리퀘스트 URL을 붙여넣어 생성한 세션을 찾을 수 있습니다. |
| `Ctrl+A` | 이 머신의 모든 프로젝트에서 세션 표시. 다시 누르면 현재 리포지토리로 돌아갑니다. |
| `Ctrl+W` | 현재 리포지토리의 모든 작업 트리에서 세션 표시. 다시 누르면 현재 작업 트리로 돌아갑니다. 다중 작업 트리 리포지토리에서만 표시됩니다. |
| `Ctrl+B` | 현재 Git 브랜치의 세션으로 필터링. 다시 누르면 모든 브랜치를 표시합니다. |
| `Esc` | 세션 선택기 또는 검색 모드 종료 |

각 행에는 이름을 설정한 경우 세션 이름이 표시되고, 그렇지 않으면 AI가 생성한 세션 제목, 대화 요약 또는 첫 번째 프롬프트가 마지막 활동 이후 시간, Git 브랜치 및 파일 크기와 함께 표시됩니다. `Ctrl+A`로 모든 프로젝트로 확장하면 각 세션의 프로젝트 경로도 볼 수 있습니다.

`/branch` 또는 `--fork-session`으로 생성된 세션은 고유한 세션 ID를 받고 별도의 행으로 나타납니다. 선택기가 동일한 세션에 대해 두 개 이상의 항목을 찾으면 단일 행 아래에 그룹화합니다. 그룹을 펼치려면 `→`를 누르세요.

`claude --resume` 선택기에서 선택한 세션을 Claude Code가 로드할 수 없는 경우, 재시도 명령어와 함께 [`Failed to resume the conversation`](/docs/en/errors#failed-to-resume-the-conversation)을 출력하고 코드 1로 종료합니다. 세션 내부의 `/resume` 선택기에서는 Claude Code가 오류를 보고하고 현재 대화가 계속 실행됩니다.

## 세션 분기 (Branch)

분기(branching)는 지금까지의 대화 사본을 생성하고 사용자를 해당 사본으로 전환하여 원래 대화를 손상되지 않은 상태로 둡니다. 가던 길을 잃지 않고 다른 접근 방식을 시도할 때 사용하세요.

세션 내부에서 선택적 이름과 함께 `/branch`를 실행합니다:

```text theme={null}
/branch try-streaming-approach
```

이름을 생략하면 Claude Code는 대화의 첫 번째 프롬프트 이름을 따서 새 브랜치의 이름을 지정합니다. v2.1.198부터는 [압축(compaction)](/docs/en/how-claude-code-works#when-context-fills-up) 후에도 이것이 적용됩니다. 이전 버전은 압축 요약을 지나 원래 첫 프롬프트를 찾는 대신 리터럴 이름인 `Branched conversation`으로 되돌아갔습니다.

명령줄에서 `--continue` 또는 `--resume`과 `--fork-session`을 결합합니다:

```bash theme={null}
claude --continue --fork-session
```

`/branch` 확인 메시지에는 두 개의 세션 ID가 출력됩니다: 현재 진입한 새 브랜치와 원래 세션입니다. 원래 세션은 디스크에서 변경되지 않으며 세션 선택기에 남아 있습니다. `/resume <original-name>`을 사용하거나 해당 ID를 `/resume`에 전달하여 복귀할 수 있습니다.

`/branch`는 트랜스크립트를 복사하고 실행 중인 Claude Code 프로세스를 전환하여 거기에 기록하도록 합니다. 이러한 차이가 브랜치가 상속하는 항목을 결정합니다:

| 상태 | `/branch` 이후 |
| :--- | :--- |
| 대화 기록 | `/branch`를 실행한 시점까지 브랜치로 복사됨 |
| "이 세션 동안 허용" 권한 부여 | 이월됨. 브랜치가 동일한 프로세스에서 실행되므로 기존 부여가 여전히 적용됩니다. `--fork-session`으로 별도의 프로세스로 분기하는 경우 새 프로세스는 권한 부여 없이 시작되며 거기서 다시 승인해야 합니다. |
| 실행 중인 [백그라운드 서브에이전트](/docs/en/sub-agents#run-subagents-in-foreground-or-background) 및 [백그라운드 Bash 명령](/docs/en/interactive-mode#background-bash-commands) | 계속 실행됨. 해당 출력은 원래 세션이 아닌 전환한 새 브랜치에 나타납니다. |

포크(fork) 없이 두 개의 터미널에서 동일한 세션을 다시 시작하면 두 터미널의 메시지가 하나의 트랜스크립트로 인터리빙(interleave)됩니다. 단일 세션 내에서 체크포인트 기반 되돌리기는 [체크포인트(Checkpointing)](/docs/en/checkpointing)를 참조하세요.

## 세션 내 컨텍스트 관리

다음 명령어는 세션을 나가지 않고 컨텍스트 창에 있는 내용을 제어합니다:

* **`/clear`**: 빈 컨텍스트로 새로 시작합니다. Claude Code는 이전 대화를 저장합니다. `/resume`으로 다시 시작하거나 동일한 Claude Code 프로세스 내에서 {/* min-version: 2.1.191 */}[되돌리기 메뉴의 이전 세션 항목](/docs/en/checkpointing#rewind-past-a-cleared-conversation)에서 다시 시작하세요. 새 대화에서도 `--name` 또는 `/rename`으로 설정한 이름은 유지되지만 AI가 생성한 세션 제목은 유지되지 않습니다.
* **`/compact [instructions]`**: 기록을 요약으로 대체하며, 선택적으로 지정한 내용에 집중할 수 있습니다.
* **`/context`**: 현재 컨텍스트를 소비하고 있는 항목을 표시합니다.

압축이 CLAUDE.md, 스킬 및 규칙과 상호작용하는 방식은 [컨텍스트 창 가이드](/docs/en/context-window)를 참조하세요. 지우기(clear)와 압축(compact) 시점에 대한 전략은 [모범 사례](/docs/en/best-practices#manage-your-session)를 참조하세요.

## 세션 데이터 내보내기 및 위치 찾기

`/export`를 실행하면 메시지 및 도구 출력이 읽을 수 있는 텍스트로 렌더링된 상태로 현재 대화를 클립보드에 복사하거나 일반 텍스트 파일로 저장할 수 있는 메뉴가 열립니다. 파일 이름을 전달하면 메뉴를 건너뛰고 해당 파일에 직접 작성합니다.

### 스크립트에서 대화에 액세스하기

`/export`는 사람이 읽을 수 있는 렌더링된 트랜스크립트를 생성합니다. 아래 인터페이스는 스크립트가 파싱할 수 있는 구조화된 데이터를 생성합니다(실행 결과 JSON, 세션의 트랜스크립트 파일 경로, 이벤트 라이브 스트림). 스크립트를 트리거하는 요소에 따라 선택하세요:

* **Claude를 한 번 실행하고 결과 캡처**: 비대화형 실행의 결과, 세션 ID, 사용량 및 비용을 구조화된 JSON으로 캡처하려면 [`--output-format json` 또는 `stream-json`](/docs/en/headless#get-structured-output)과 함께 `claude -p`를 호출합니다.
* **기존 세션에 질문하기**: 세션 ID를 [`claude -p --resume`](/docs/en/headless#continue-conversations)에 전달하여 요약 요청과 같은 후속 프롬프트를 전송하고 구조화된 응답을 캡처합니다.
* **세션 이벤트에 반응하기**: [훅](/docs/en/hooks#common-input-fields) 및 [상태 표시줄 명령](/docs/en/statusline#available-data)이 입력으로 받는 `transcript_path` 필드를 읽습니다. `SessionEnd` 훅은 세션이 끝날 때 트랜스크립트를 보관할 수 있습니다.
* **TypeScript 또는 Python 앱에 Claude 임베딩**: 각 메시지를 프로그래밍 방식으로 수신하려면 [Agent SDK](/docs/en/agent-sdk/overview)를 사용하세요.

아래 예시는 두 번째 인터페이스를 사용합니다. 기존 세션에 후속 프롬프트를 보내고 `jq`로 답변을 읽습니다:

```bash theme={null}
claude -p --resume <session-id> --output-format json "summarize what we changed" | jq -r '.result'
```

### 트랜스크립트가 저장되는 위치

기본적으로 트랜스크립트는 `~/.claude/projects/<project>/<session-id>.jsonl`에 JSONL로 저장되며, 여기서 `<project>`는 영문자가 아닌 문자가 `-`로 대체된 작업 디렉터리 경로입니다. 각 행은 메시지, 도구 사용 또는 메타데이터 항목에 대한 JSON 객체입니다. 항목 형식은 Claude Code 내부 전용이며 버전 간에 변경되므로 이 파일을 직접 파싱하는 스크립트는 릴리스 시 작동하지 않을 수 있습니다. 세션 데이터를 기반으로 구축하려면 대신 `/export` 또는 [스크립트 인터페이스](#스크립트에서-대화에-액세스하기)를 사용하세요.

위치, 보존 및 쓰기 동작을 구성할 수 있습니다:

| 목적 | 설정 | 위치 |
| :--- | :--- | :--- |
| `~/.claude` 외부로 저장소 이동 | [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars) | 환경 변수 |
| 30일 보존 기간 변경 | [`cleanupPeriodDays`](/docs/en/settings#available-settings) | `settings.json` |
| 모든 모드에서 트랜스크립트 쓰기 억제 | [`CLAUDE_CODE_SKIP_PROMPT_HISTORY`](/docs/en/env-vars) | 환경 변수 |
| 단일 비대화형 실행에 대한 쓰기 억제 | [`--no-session-persistence`](/docs/en/cli-reference) | `claude -p`와 함께 사용하는 CLI 플래그 |

## 관련 항목

다음 페이지에서는 관련 세션 및 병렬 처리 메커니즘을 다룹니다:

* [작업 트리(Worktrees)](/docs/en/worktrees): 별도의 브랜치에서 격리된 병렬 세션 실행
* [체크포인트(Checkpointing)](/docs/en/checkpointing): 코드 및 대화를 이전 시점으로 되돌리기
* [컨텍스트 창](/docs/en/context-window): 컨텍스트를 채우는 항목 및 압축 후 살아남는 항목
* [비대화형 모드](/docs/en/headless): `claude -p` 환경에서의 세션 동작
