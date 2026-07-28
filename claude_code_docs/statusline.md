> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 상태 표시줄 맞춤 설정하기

> Claude Code에서 컨텍스트 창 사용량, 비용 및 Git 상태를 모니터링하기 위해 맞춤형 상태 표시줄을 구성하세요.

상태 표시줄은 구성한 임의의 셸 스크립트를 실행하는 Claude Code 하단의 맞춤형 바입니다. stdin을 통해 JSON 세션 데이터를 수신하고 스크립트가 출력하는 모든 내용을 표시하므로 컨텍스트 사용량, 비용, Git 상태 또는 추적하려는 다른 모든 항목에 대한 한눈에 볼 수 있는 영구 뷰를 제공합니다.

상태 표시줄은 다음과 같은 경우에 유용합니다:

* 작업하면서 컨텍스트 창 사용량을 모니터링하려는 경우
* 세션 비용을 추적해야 하는 경우
* 여러 세션에 걸쳐 작업하며 이를 구분해야 하는 경우
* Git 브랜치 및 상태를 항상 확인하고 싶은 경우

상태 표시줄은 내장된 푸터 배지 위의 자체 행에 렌더링되며 이를 대체하지 않습니다. 스크립트를 작성하지 않고 대화에 ID가 나타날 때 푸터에 클릭 가능한 링크 배지를 추가하려면 대신 [`footerLinksRegexes`](/docs/en/settings#footer-link-badges)를 구성하세요.

다음은 첫 번째 줄에는 Git 정보를 표시하고 두 번째 줄에는 색상으로 구분된 컨텍스트 바를 표시하는 [다중 행 상태 표시줄](#여러-줄-표시하기)의 예시입니다.

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-multiline.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=60f11387658acc9ff75158ae85f2ac87" alt="첫 번째 줄에는 모델 이름, 디렉터리, Git 브랜치를 보여주고 두 번째 줄에는 비용 및 기간과 함께 컨텍스트 사용량 진행률 표시줄을 보여주는 다중 행 상태 표시줄" width="776" height="212" data-path="images/statusline-multiline.png" />
</Frame>

이 페이지에서는 [기본 상태 표시줄 설정](#상태-표시줄-설정하기)을 단계별로 설명하고, Claude Code에서 스크립트로 [데이터가 전달되는 방식](#상태-표시줄-작동-방식)을 설명하며, [표시할 수 있는 모든 필드](#사용-가능한-데이터)를 나열하고, Git 상태, 비용 추적 및 진행률 표시줄과 같은 일반적인 패턴에 대해 [바로 사용할 수 있는 예시](#예시)를 제공합니다.

## 상태 표시줄 설정하기

[`/statusline` 명령](#statusline-명령-사용하기)을 사용하여 Claude Code가 대신 스크립트를 생성하도록 하거나, [수동으로 스크립트를 생성](#수동으로-상태-표시줄-구성하기)하여 설정에 추가하세요.

### /statusline 명령 사용하기

`/statusline` 명령은 표시하려는 내용을 설명하는 자연어 지침을 허용합니다. Claude Code는 `~/.claude/`에 스크립트 파일을 생성하고 설정을 자동으로 업데이트합니다:

```text theme={null}
/statusline show model name and context percentage with a progress bar
```

설정 중에 Claude Code가 권한을 요청하는 경우 파일 편집 프롬프트를 승인하세요.

### 수동으로 상태 표시줄 구성하기

사용자 설정(`~/.claude/settings.json`, 여기서 `~`는 홈 디렉터리임) 또는 [프로젝트 설정](/docs/en/settings#settings-files)에 `statusLine` 필드를 추가하세요. `type`을 `"command"`로 설정하고 `command`를 스크립트 경로 또는 인라인 셸 명령으로 지정하세요. 스크립트 생성에 대한 전체 단계별 안내는 [단계별로 상태 표시줄 구축하기](#단계별로-상태-표시줄-구축하기)를 참조하세요.

```json theme={null}
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2
  }
}
```

`command` 필드는 셸에서 실행되므로 스크립트 파일 대신 인라인 명령을 사용할 수도 있습니다. 이 예시는 `jq`를 사용하여 JSON 입력을 파싱하고 모델 이름과 컨텍스트 비율을 표시합니다:

```json theme={null}
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% context\"'"
  }
}
```

선택 사항인 `padding` 필드는 상태 표시줄 내용에 추가 수평 간격(문자 단위)을 추가합니다. 기본값은 `0`입니다. 이 패딩은 인터페이스의 내장 간격에 추가되므로 터미널 가장자리로부터의 절대 거리가 아니라 상대적 들여쓰기를 제어합니다.

선택 사항인 `refreshInterval` 필드는 [이벤트 기반 업데이트](#상태-표시줄-작동-방식) 외에도 N초마다 명령을 다시 실행합니다. 최소값은 `1`입니다. 상태 표시줄에 시계와 같은 시간 기반 데이터를 표시할 때나 메인 세션이 유휴 상태인 동안 백그라운드 서브에이전트가 Git 상태를 변경할 때 이를 설정하세요. 이벤트 발생 시에만 실행하려면 설정하지 않은 상태로 두세요.

선택 사항인 `hideVimModeIndicator` 필드는 프롬프트 아래의 내장 `-- INSERT --` 텍스트를 숨깁니다. 스크립트가 [`vim.mode`](#사용-가능한-데이터) 자체를 렌더링할 때 모드가 두 번 표시되지 않도록 `true`로 설정하세요.

### 상태 표시줄 비활성화

`/statusline`을 실행하고 상태 표시줄을 제거하거나 지우도록 요청하세요(예: `/statusline delete`, `/statusline clear`, `/statusline remove it`). settings.json에서 `statusLine` 필드를 수동으로 삭제할 수도 있습니다.

## 단계별로 상태 표시줄 구축하기

이 가이드는 현재 모델, 작업 디렉터리 및 컨텍스트 창 사용 비율을 표시하는 상태 표시줄을 수동으로 생성하여 내부에서 무슨 일이 일어나는지 보여줍니다.

<Note>원하는 내용에 대한 설명과 함께 [`/statusline`](#statusline-명령-사용하기)을 실행하면 이 모든 것이 자동으로 구성됩니다.</Note>

이 예시에서는 macOS 및 Linux에서 작동하는 Bash 스크립트를 사용합니다. Windows의 경우 PowerShell 및 Git Bash 예시는 [Windows 구성](#windows-구성)을 참조하세요.

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-quickstart.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=696445e59ca0059213250651ad23db6b" alt="모델 이름, 디렉터리 및 컨텍스트 비율을 보여주는 상태 표시줄" width="726" height="164" data-path="images/statusline-quickstart.png" />
</Frame>

<Steps>
  <Step title="JSON을 읽고 출력을 인쇄하는 스크립트 생성">
    Claude Code는 stdin을 통해 스크립트로 JSON 데이터를 전송합니다. 이 스크립트는 명령줄 JSON 파서인 [`jq`](https://jqlang.org/) (설치가 필요할 수 있음)를 사용하여 모델 이름, 디렉터리 및 컨텍스트 비율을 추출한 후 형식이 지정된 줄을 인쇄합니다.

    이를 `~/.claude/statusline.sh`에 저장하세요 (여기서 `~`는 macOS의 `/Users/username` 또는 Linux의 `/home/username`과 같은 홈 디렉터리입니다):

    ```bash theme={null}
    #!/bin/bash
    # Claude Code가 stdin으로 보내는 JSON 데이터 읽기
    input=$(cat)

    # jq를 사용하여 필드 추출
    MODEL=$(echo "$input" | jq -r '.model.display_name')
    DIR=$(echo "$input" | jq -r '.workspace.current_dir')
    # "// 0"은 필드가 null인 경우 기본값을 제공합니다.
    PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

    # 상태 표시줄 출력 - ${DIR##*/}는 폴더 이름만 추출합니다.
    echo "[$MODEL] 📁 ${DIR##*/} | ${PCT}% context"
    ```
  </Step>

  <Step title="실행 가능하게 만들기">
    셸에서 스크립트를 실행할 수 있도록 실행 가능으로 표시합니다:

    ```bash theme={null}
    chmod +x ~/.claude/statusline.sh
    ```
  </Step>

  <Step title="설정에 추가">
    Claude Code에게 상태 표시줄로 스크립트를 실행하도록 지시하세요. `~/.claude/settings.json`에 이 구성을 추가합니다. 이는 `type`을 `"command"`로 설정하고(`"이 셸 명령 실행"`을 의미) `command`를 스크립트로 지정합니다:

    ```json theme={null}
    {
      "statusLine": {
        "type": "command",
        "command": "~/.claude/statusline.sh"
      }
    }
    ```

    인터페이스 하단에 상태 표시줄이 나타납니다. 설정은 자동으로 다시 로드되지만 Claude Code와의 다음 상호작용 전까지는 변경 사항이 나타나지 않습니다.
  </Step>
</Steps>

## 상태 표시줄 작동 방식

Claude Code는 스크립트를 실행하고 stdin을 통해 [JSON 세션 데이터](#사용-가능한-데이터)를 파이프로 전송합니다. 스크립트는 JSON을 읽고 필요한 항목을 추출한 후 stdout으로 텍스트를 인쇄합니다. Claude Code는 스크립트가 인쇄하는 모든 내용을 표시합니다.

**업데이트 시점**

스크립트는 세션을 재개할 때를 포함하여 세션이 시작될 때 한 번 실행됩니다. 그 후에는 다음 경우에 다시 실행됩니다:

* 새 어시스턴트 메시지가 도착할 때
* `/compact`가 완료될 때
* 권한 모드가 변경될 때
* Vim 모드가 토글될 때
* 설정한 경우 [`refreshInterval`](#수동으로-상태-표시줄-구성하기) 타이머가 경과할 때

{/* min-version: 2.1.216 */}v2.1.216 이전에는 세션을 재개하면 명령이 빠르게 연속해서 두 번 실행되었으므로 첫 번째 결과가 교체되기 전에 깜빡일 수 있었습니다.

Claude Code는 300ms에서 업데이트 디바운싱을 수행하므로 빠른 변경 사항이 함께 일괄 처리되고 변경이 중지된 후 스크립트가 한 번 실행됩니다. 스크립트가 아직 실행 중인 동안 새 업데이트가 트리거되면 Claude Code는 실행 중인 스크립트를 취소합니다. 스크립트를 편집하면 업데이트 트리거가 다시 실행할 때 변경 사항이 나타납니다.

이벤트 기반 트리거는 예를 들어 코디네이터가 백그라운드 서브에이전트를 기다리는 동안과 같이 메인 세션이 유휴 상태일 때 조용해질 수 있습니다. 유휴 시간 동안 시간 기반 또는 외부 소스 세그먼트를 최신 상태로 유지하려면 [`refreshInterval`](#수동으로-상태-표시줄-구성하기)을 설정하여 고정 타이머에서도 명령을 다시 실행하도록 하세요.

**스크립트가 출력할 수 있는 내용**

* **여러 줄**: 각 `echo` 또는 `print` 문이 별도의 행으로 표시됩니다. [다중 행 예시](#여러-줄-표시하기)를 참조하세요.
* **색상**: 녹색의 경우 `\033[32m`과 같은 [ANSI 이스케이프 코드](https://en.wikipedia.org/wiki/ANSI_escape_code#Colors)를 사용하세요(터미널이 이를 지원해야 함). [색상이 있는 Git 상태 예시](#색상이-있는-git-상태)를 참조하세요.
* **링크**: [OSC 8 이스케이프 시퀀스](https://en.wikipedia.org/wiki/ANSI_escape_code#OSC)를 사용하여 텍스트를 클릭 가능하게 만드세요(macOS에서는 Cmd+클릭, Windows/Linux에서는 Ctrl+클릭). iTerm2, Kitty 또는 WezTerm과 같이 하이퍼링크를 지원하는 터미널이 필요합니다. [클릭 가능한 링크 예시](#클릭-가능한-링크)를 참조하세요.

**터미널 크기에 맞게 출력 크기 조정**

Claude Code는 스크립트의 출력을 터미널에 직접 연결하는 대신 캡처하므로 `tput cols` 및 언어 수준 너비 감지는 스크립트 내부에서 터미널 크기를 읽을 수 없습니다. {/* min-version: 2.1.153 */}대신 `COLUMNS` 및 `LINES` 환경 변수를 읽으세요. Claude Code는 스크립트를 실행하기 전에 이들을 현재 터미널 차원으로 설정합니다. Claude Code v2.1.153 이상이 필요합니다.

<Note>상태 표시줄은 로컬에서 실행되며 API 토큰을 소비하지 않습니다. 자동 완성 제안, 도움말 메뉴 및 권한 프롬프트를 포함한 특정 UI 상호작용 중에는 임시로 숨겨집니다.</Note>

## 사용 가능한 데이터

Claude Code는 stdin을 통해 스크립트로 다음 JSON 필드를 전송합니다:

| 필드 | 설명 |
| :--- | :--- |
| `model.id`, `model.display_name` | 현재 모델 식별자 및 표시 이름 |
| `cwd`, `workspace.current_dir` | 현재 작업 디렉터리. 두 필드 모두 동일한 값을 포함하며 `workspace.project_dir`과의 일관성을 위해 `workspace.current_dir`을 선호합니다. |
| `workspace.project_dir` | Claude Code가 시작된 디렉터리로 세션 중에 작업 디렉터리가 변경되면 `cwd`와 다를 수 있음 |
| `workspace.added_dirs` | `/add-dir` 또는 `--add-dir`을 통해 추가된 추가 디렉터리. 추가된 항목이 없으면 빈 배열 |
| `workspace.git_worktree` | 현재 디렉터리가 `git worktree add`로 생성된 연결된 워크트리 내부일 때의 Git 워크트리 이름. 메인 작업 트리에는 없음. `--worktree` 세션에만 적용되는 `worktree.*`와 달리 모든 Git 워크트리에 대해 데이터가 채워짐 |
| `workspace.repo.host`, `workspace.repo.owner`, `workspace.repo.name` | `origin` 원격지에서 파싱된 리포지토리 식별자(예: `"github.com"`, `"anthropics"`, `"claude-code"`). Git 리포지토리 외부이거나 `origin` 원격지가 구성되어 있지 않은 경우 없음 |
| `cost.total_cost_usd` | 클라이언트 측에서 계산된 예상 세션 비용(USD). 실제 청구액과 다를 수 있음. {/* min-version: 2.1.211 */}`/clear`로 새 세션을 시작할 때 \$0으로 재설정됨 |
| `cost.total_duration_ms` | 세션이 시작된 이후 총 실시간 경과 시간(밀리초 단위) |
| `cost.total_api_duration_ms` | API 응답을 기다리는 데 소비한 총 시간(밀리초 단위) |
| `cost.total_lines_added`, `cost.total_lines_removed` | 변경된 코드 라인 수 |
| `context_window.total_input_tokens`, `context_window.total_output_tokens` | 가장 최근 API 응답 기준 현재 컨텍스트 창의 토큰 수. 입력에는 캐시 읽기 및 쓰기가 포함됨. {/* min-version: 2.1.132 */}v2.1.132 이전에는 이것들이 누적 세션 합계였습니다. |
| `context_window.context_window_size` | 토큰 단위의 최대 컨텍스트 창 크기. 기본적으로 200000개, 확장 컨텍스트가 있는 모델의 경우 1000000개 |
| `context_window.used_percentage` | 컨텍스트 창 사용 비율(사전 계산됨) |
| `context_window.remaining_percentage` | 남아 있는 컨텍스트 창 비율(사전 계산됨) |
| `context_window.current_usage` | [컨텍스트 창 필드](#컨텍스트-창-필드)에 설명된 마지막 API 호출에서의 토큰 수 |
| `exceeds_200k_tokens` | 가장 최근 API 응답의 전체 토큰 수(입력, 캐시 및 출력 토큰의 합)가 200k를 초과하는지 여부. 이는 실제 컨텍스트 창 크기와 무관한 고정 임계값입니다. |
| `fast_mode` | 세션에 [패스트 모드](/docs/en/fast-mode)가 활성화되어 있는지 여부 |
| `effort.level` | 현재 추론 노력 수준(`low`, `medium`, `high`, `xhigh` 또는 `max`). 세션 중간의 `/effort` 변경 사항을 포함하여 라이브 세션 값을 반영합니다. Ultracode는 별도의 수준이 아니며 `xhigh`로 보고됩니다. 현재 모델이 노력 파라미터를 지원하지 않을 때는 없음 |
| `thinking.enabled` | 세션에 확장 생각(extended thinking)이 활성화되어 있는지 여부 |
| `rate_limits.five_hour.used_percentage`, `rate_limits.seven_day.used_percentage` | 소비된 5시간 또는 7일 속도 제한 비율(0~100) |
| `rate_limits.five_hour.resets_at`, `rate_limits.seven_day.resets_at` | 5시간 또는 7일 속도 제한 창이 재설정되는 Unix 에포크 초 |
| `session_id` | 고유 세션 식별자 |
| `session_name` | 세션 이름. `--name` 플래그 또는 `/rename`으로 설정된 맞춤형 이름이 있는 경우 이를 사용하고, 그렇지 않으면 AI가 생성한 세션 제목을 사용합니다. `my-app-3f`와 같은 [기본 표시 이름](/docs/en/sessions#name-your-sessions)은 이 필드를 채우지 않습니다. 세션에 맞춤형 이름과 AI 생성 제목이 모두 없는 경우 없음 |
| `prompt_id` | 현재 처리 중인 사용자 프롬프트를 식별하는 UUID. [OpenTelemetry 이벤트의 `prompt.id` 속성](/docs/en/monitoring-usage#event-correlation-attributes)과 일치합니다. 첫 사용자 입력 전까지는 없음. {/* min-version: 2.1.196 */}Claude Code v2.1.196 이상이 필요함 |
| `transcript_path` | 대화 트랜스크립트 파일 경로 |
| `version` | Claude Code 버전 |
| `output_style.name` | 현재 출력 스타일의 이름 |
| `vim.mode` | [Vim 모드](/docs/en/interactive-mode#vim-editor-mode)가 활성화되어 있을 때의 현재 Vim 모드(`NORMAL`, `INSERT`, `VISUAL` 또는 `VISUAL LINE`) |
| `agent.name` | `--agent` 플래그를 사용하여 실행하거나 에이전트 설정이 구성되었을 때의 에이전트 이름 |
| `pr.number`, `pr.url` | 현재 브랜치에 대해 열린 풀 리퀘스트. 하단 상태 표시줄의 PR 배지를 미러링합니다. PR이 발견될 때까지, Git 리포지토리에 있지 않을 때, 또는 PR이 머지되거나 닫히면 없음 |
| `pr.review_state` | 열린 PR의 검토 상태: `approved`, `pending`, `changes_requested` 또는 `draft`. `pr`이 존재하는 경우에도 독립적으로 없을 수 있음 |
| `worktree.name` | 활성 워크트리의 이름. `--worktree` 세션 중에만 존재 |
| `worktree.path` | 워크트리 디렉터리의 절대 경로 |
| `worktree.branch` | 워크트리의 Git 브랜치 이름(예: `"worktree-my-feature"`). 훅 기반 워크트리에는 없음 |
| `worktree.original_cwd` | 워크트리에 진입하기 전 Claude가 있던 디렉터리 |
| `worktree.original_branch` | 워크트리에 진입하기 전 체크아웃된 Git 브랜치. 훅 기반 워크트리에는 없음 |

<Accordion title="전체 JSON 스키마">
  상태 표시줄 명령은 stdin을 통해 다음 JSON 구조를 수신합니다:

  ```json theme={null}
  {
    "cwd": "/current/working/directory",
    "session_id": "abc123...",
    "session_name": "my-session",
    "prompt_id": "550e8400-e29b-41d4-a716-446655440000",
    "transcript_path": "/path/to/transcript.jsonl",
    "model": {
      "id": "claude-opus-4-8",
      "display_name": "Opus"
    },
    "workspace": {
      "current_dir": "/current/working/directory",
      "project_dir": "/original/project/directory",
      "added_dirs": [],
      "git_worktree": "feature-xyz",
      "repo": {
        "host": "github.com",
        "owner": "anthropics",
        "name": "claude-code"
      }
    },
    "version": "2.1.90",
    "output_style": {
      "name": "default"
    },
    "cost": {
      "total_cost_usd": 0.01234,
      "total_duration_ms": 45000,
      "total_api_duration_ms": 2300,
      "total_lines_added": 156,
      "total_lines_removed": 23
    },
    "context_window": {
      "total_input_tokens": 15500,
      "total_output_tokens": 1200,
      "context_window_size": 200000,
      "used_percentage": 8,
      "remaining_percentage": 92,
      "current_usage": {
        "input_tokens": 8500,
        "output_tokens": 1200,
        "cache_creation_input_tokens": 5000,
        "cache_read_input_tokens": 2000
      }
    },
    "exceeds_200k_tokens": false,
    "fast_mode": false,
    "effort": {
      "level": "high"
    },
    "thinking": {
      "enabled": true
    },
    "rate_limits": {
      "five_hour": {
        "used_percentage": 23.5,
        "resets_at": 1738425600
      },
      "seven_day": {
        "used_percentage": 41.2,
        "resets_at": 1738857600
      }
    },
    "vim": {
      "mode": "NORMAL"
    },
    "agent": {
      "name": "security-reviewer"
    },
    "pr": {
      "number": 1234,
      "url": "https://github.com/anthropics/claude-code/pull/1234",
      "review_state": "pending"
    },
    "worktree": {
      "name": "my-feature",
      "path": "/path/to/.claude/worktrees/my-feature",
      "branch": "worktree-my-feature",
      "original_cwd": "/path/to/project",
      "original_branch": "main"
    }
  }
  ```

  **누락될 수 있는 필드** (JSON에 없음):

  * `session_name`: `--name` 또는 `/rename`으로 맞춤형 이름을 설정했거나 AI가 생성한 세션 제목이 존재하는 경우에만 나타납니다. `my-app-3f`와 같은 기본 표시 이름은 이를 채우지 않습니다.
  * `prompt_id`: 첫 번째 사용자 입력 후에만 나타납니다.
  * `workspace.git_worktree`: 현재 디렉터리가 연결된 Git 워크트리 내부일 때만 나타납니다.
  * `workspace.repo`: `origin` 원격지가 구성된 Git 리포지토리 내부에서만 나타납니다.
  * `effort`: 현재 모델이 추론 노력 파라미터를 지원할 때만 나타납니다.
  * `vim`: Vim 모드가 활성화되어 있을 때만 나타납니다.
  * `agent`: `--agent` 플래그를 사용하여 실행하거나 에이전트 설정이 구성되었을 때만 나타납니다.
  * `pr`: 현재 브랜치에 대해 열린 PR이 발견된 동안에만 나타나며 PR이 머지되거나 닫히면 제거됩니다. `pr.review_state`는 독립적으로 없을 수 있습니다.
  * `worktree`: `--worktree` 세션 중에만 나타납니다. 존재하는 경우 훅 기반 워크트리에 대해 `branch` 및 `original_branch`가 없을 수도 있습니다.
  * `rate_limits`: 첫 번째 API 응답 이후 Claude.ai 구독자(Pro/Max)에게만 나타납니다. 각 창(`five_hour`, `seven_day`)은 독립적으로 없을 수 있습니다. 부재를 정상적으로 처리하려면 `jq -r '.rate_limits.five_hour.used_percentage // empty'`를 사용하세요.

  **`null`일 수 있는 필드**:

  * `context_window.current_usage`: 세션의 첫 번째 API 호출 전과 다음 API 호출이 다시 채울 때까지 `/compact` 직후에 `null`이 됩니다.
  * `context_window.used_percentage`, `context_window.remaining_percentage`: 세션 초기에 `null`일 수 있습니다.

  스크립트에서 조건부 접근으로 누락된 필드를 처리하고 대체 기본값으로 null 값을 처리하세요.
</Accordion>

### 컨텍스트 창 필드

`context_window` 객체는 가장 최근 API 응답의 라이브 컨텍스트 창을 설명합니다. v2.1.132부터 `total_input_tokens` 및 `total_output_tokens`는 누적 세션 합계가 아닌 현재 컨텍스트 사용량을 반영합니다.

* **결합된 합계** (`total_input_tokens`, `total_output_tokens`): 현재 컨텍스트 창에 있는 토큰 수. `total_input_tokens`는 `input_tokens`, `cache_creation_input_tokens` 및 `cache_read_input_tokens`의 합계이며 `total_output_tokens`는 가장 최근 응답의 출력 토큰 수입니다. 둘 다 첫 번째 API 응답 전에는 `0`입니다.
* **구성 요소별 사용량** (`current_usage`): 범주별로 세분화된 동일한 토큰 수. 새 입력과 별도로 캐시 히트가 필요할 때 사용하세요.

`current_usage` 객체에는 다음이 포함됩니다:

* `input_tokens`: 현재 컨텍스트의 입력 토큰
* `output_tokens`: 생성된 출력 토큰
* `cache_creation_input_tokens`: 캐시에 기록된 토큰
* `cache_read_input_tokens`: 캐시에서 읽은 토큰

캐시 필드가 의미하는 바와 청구 방식은 [프로프라이팅 캐시 성능 확인](/docs/en/prompt-caching#check-cache-performance)을 참조하세요.

`used_percentage` 필드는 입력 토큰만으로 계산됩니다: `input_tokens + cache_creation_input_tokens + cache_read_input_tokens`. `output_tokens`는 포함되지 않습니다.

`current_usage`에서 컨텍스트 비율을 수동으로 계산하는 경우 `used_percentage`와 일치하도록 동일한 입력 전용 공식을 사용하세요.

`current_usage` 객체는 세션의 첫 번째 API 호출 전과 다음 API 호출이 다시 채울 때까지 `/compact` 직후에 `null`입니다.

## 예시

이 예시들은 일반적인 상태 표시줄 패턴을 보여줍니다. 예시를 사용하려면:

1. 스크립트를 `~/.claude/statusline.sh` (또는 `.py`/`.js`)와 같은 파일에 저장합니다.
2. 실행 가능으로 만듭니다: `chmod +x ~/.claude/statusline.sh`
3. 경로를 [설정](#수동으로-상태-표시줄-구성하기)에 추가합니다.

Bash 예시는 JSON을 파싱하기 위해 [`jq`](https://jqlang.org/)를 사용합니다. Python 및 Node.js에는 JSON 파싱이 내장되어 있습니다.

### 컨텍스트 창 사용량

시각적 진행률 표시줄을 사용하여 현재 모델 및 컨텍스트 창 사용량을 표시합니다. 각 스크립트는 stdin에서 JSON을 읽고 `used_percentage` 필드를 추출하며 채워진 블록(▓)이 사용량을 나타내는 10문자 바를 작성합니다:

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-context-window-usage.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=15b58ab3602f036939145dde3165c6f7" alt="모델 이름과 비율이 포함된 진행률 표시줄을 보여주는 상태 표시줄" width="448" height="152" data-path="images/statusline-context-window-usage.png" />
</Frame>

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  # stdin 전체를 변수로 읽기
  input=$(cat)

  # jq로 필드 추출, "// 0"은 null에 대한 대체 제공
  MODEL=$(echo "$input" | jq -r '.model.display_name')
  PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

  # 진행률 표시줄 구성: printf -v는 공백을 생성하고
  # ${var// /▓}는 각 공백을 블록 문자로 대체합니다.
  BAR_WIDTH=10
  FILLED=$((PCT * BAR_WIDTH / 100))
  EMPTY=$((BAR_WIDTH - FILLED))
  BAR=""
  [ "$FILLED" -gt 0 ] && printf -v FILL "%${FILLED}s" && BAR="${FILL// /▓}"
  [ "$EMPTY" -gt 0 ] && printf -v PAD "%${EMPTY}s" && BAR="${BAR}${PAD// /░}"

  echo "[$MODEL] $BAR $PCT%"
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys

  # json.load는 stdin을 한 번에 읽고 파싱합니다.
  data = json.load(sys.stdin)
  model = data['model']['display_name']
  # "or 0"은 null 값을 처리합니다.
  pct = int(data.get('context_window', {}).get('used_percentage', 0) or 0)

  # 문자열 곱셈으로 바를 작성합니다.
  filled = pct * 10 // 100
  bar = '▓' * filled + '░' * (10 - filled)

  print(f"[{model}] {bar} {pct}%")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  // Node.js는 이벤트를 사용하여 비동기로 stdin을 읽습니다.
  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;
      // 옵셔널 체이닝(?.)은 null 필드를 안전하게 처리합니다.
      const pct = Math.floor(data.context_window?.used_percentage || 0);

      // String.repeat()로 바를 작성합니다.
      const filled = Math.floor(pct * 10 / 100);
      const bar = '▓'.repeat(filled) + '░'.repeat(10 - filled);

      console.log(`[${model}] ${bar} ${pct}%`);
  });
  ```
</CodeGroup>

### 색상이 있는 Git 상태

스테이징된 파일과 수정된 파일에 대해 색상으로 구분된 표시기와 함께 Git 브랜치를 표시합니다. 이 스크립트는 터미널 색상용 [ANSI 이스케이프 코드](https://en.wikipedia.org/wiki/ANSI_escape_code#Colors)를 사용합니다: `\033[32m`은 녹색, `\033[33m`은 노란색, `\033[0m`은 기본값으로 리셋합니다.

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-git-context.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=e656f34f90d1d9a1d0e220988914345f" alt="모델, 디렉터리, Git 브랜치 및 스테이징/수정된 파일의 색상 표시기를 보여주는 상태 표시줄" width="742" height="178" data-path="images/statusline-git-context.png" />
</Frame>

각 스크립트는 현재 디렉터리가 Git 리포지토리인지 확인하고 스테이징 및 수정된 파일을 계산하여 색상 표시기를 보여줍니다:

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  input=$(cat)

  MODEL=$(echo "$input" | jq -r '.model.display_name')
  DIR=$(echo "$input" | jq -r '.workspace.current_dir')

  GREEN='\033[32m'
  YELLOW='\033[33m'
  RESET='\033[0m'

  if git rev-parse --git-dir > /dev/null 2>&1; then
      BRANCH=$(git branch --show-current 2>/dev/null)
      STAGED=$(git diff --cached --numstat 2>/dev/null | wc -l | tr -d ' ')
      MODIFIED=$(git diff --numstat 2>/dev/null | wc -l | tr -d ' ')

      GIT_STATUS=""
      [ "$STAGED" -gt 0 ] && GIT_STATUS="${GREEN}+${STAGED}${RESET}"
      [ "$MODIFIED" -gt 0 ] && GIT_STATUS="${GIT_STATUS}${YELLOW}~${MODIFIED}${RESET}"

      echo -e "[$MODEL] 📁 ${DIR##*/} | 🌿 $BRANCH $GIT_STATUS"
  else
      echo "[$MODEL] 📁 ${DIR##*/}"
  fi
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys, subprocess, os

  data = json.load(sys.stdin)
  model = data['model']['display_name']
  directory = os.path.basename(data['workspace']['current_dir'])

  GREEN, YELLOW, RESET = '\033[32m', '\033[33m', '\033[0m'

  try:
      subprocess.check_output(['git', 'rev-parse', '--git-dir'], stderr=subprocess.DEVNULL)
      branch = subprocess.check_output(['git', 'branch', '--show-current'], text=True).strip()
      staged_output = subprocess.check_output(['git', 'diff', '--cached', '--numstat'], text=True).strip()
      modified_output = subprocess.check_output(['git', 'diff', '--numstat'], text=True).strip()
      staged = len(staged_output.split('\n')) if staged_output else 0
      modified = len(modified_output.split('\n')) if modified_output else 0

      git_status = f"{GREEN}+{staged}{RESET}" if staged else ""
      git_status += f"{YELLOW}~{modified}{RESET}" if modified else ""

      print(f"[{model}] 📁 {directory} | 🌿 {branch} {git_status}")
  except:
      print(f"[{model}] 📁 {directory}")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  const { execSync } = require('child_process');
  const path = require('path');

  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;
      const dir = path.basename(data.workspace.current_dir);

      const GREEN = '\x1b[32m', YELLOW = '\x1b[33m', RESET = '\x1b[0m';

      try {
          execSync('git rev-parse --git-dir', { stdio: 'ignore' });
          const branch = execSync('git branch --show-current', { encoding: 'utf8' }).trim();
          const staged = execSync('git diff --cached --numstat', { encoding: 'utf8' }).trim().split('\n').filter(Boolean).length;
          const modified = execSync('git diff --numstat', { encoding: 'utf8' }).trim().split('\n').filter(Boolean).length;

          let gitStatus = staged ? `${GREEN}+${staged}${RESET}` : '';
          gitStatus += modified ? `${YELLOW}~${modified}${RESET}` : '';

          console.log(`[${model}] 📁 ${dir} | 🌿 ${branch} ${gitStatus}`);
      } catch {
          console.log(`[${model}] 📁 ${dir}`);
      }
  });
  ```
</CodeGroup>

### 비용 및 기간 추적

세션의 API 비용 및 경과 시간을 추적합니다. `cost.total_cost_usd` 필드는 현재 세션에서 모든 API 호출의 누적 예상 비용입니다. `cost.total_duration_ms` 필드는 세션이 시작된 후 총 경과 시간을 측정하고, `cost.total_api_duration_ms`는 API 응답을 기다리는 데 소비한 시간만 추적합니다.

각 스크립트는 비용을 통화로 서식 지정하고 밀리초를 분과 초로 변환합니다:

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-cost-tracking.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=e3444a51fe6f3440c134bd5f1f08ad29" alt="모델 이름, 세션 비용 및 기간을 보여주는 상태 표시줄" width="588" height="180" data-path="images/statusline-cost-tracking.png" />
</Frame>

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  input=$(cat)

  MODEL=$(echo "$input" | jq -r '.model.display_name')
  COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
  DURATION_MS=$(echo "$input" | jq -r '.cost.total_duration_ms // 0')

  COST_FMT=$(printf '$%.2f' "$COST")
  DURATION_SEC=$((DURATION_MS / 1000))
  MINS=$((DURATION_SEC / 60))
  SECS=$((DURATION_SEC % 60))

  echo "[$MODEL] 💰 $COST_FMT | ⏱️ ${MINS}m ${SECS}s"
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys

  data = json.load(sys.stdin)
  model = data['model']['display_name']
  cost = data.get('cost', {}).get('total_cost_usd', 0) or 0
  duration_ms = data.get('cost', {}).get('total_duration_ms', 0) or 0

  duration_sec = duration_ms // 1000
  mins, secs = duration_sec // 60, duration_sec % 60

  print(f"[{model}] 💰 ${cost:.2f} | ⏱️ {mins}m {secs}s")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;
      const cost = data.cost?.total_cost_usd || 0;
      const durationMs = data.cost?.total_duration_ms || 0;

      const durationSec = Math.floor(durationMs / 1000);
      const mins = Math.floor(durationSec / 60);
      const secs = durationSec % 60;

      console.log(`[${model}] 💰 $${cost.toFixed(2)} | ⏱️ ${mins}m ${secs}s`);
  });
  ```
</CodeGroup>

### 여러 줄 표시하기

스크립트는 더 풍부한 디스플레이를 생성하기 위해 여러 줄을 출력할 수 있습니다. 각 `echo` 문은 상태 영역에 별도의 행을 생성합니다.

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-multiline.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=60f11387658acc9ff75158ae85f2ac87" alt="첫 번째 줄에는 모델 이름, 디렉터리, Git 브랜치를 보여주고 두 번째 줄에는 비용 및 기간과 함께 컨텍스트 사용량 진행률 표시줄을 보여주는 다중 행 상태 표시줄" width="776" height="212" data-path="images/statusline-multiline.png" />
</Frame>

이 예시는 임계값 기반 색상(70% 미만 녹색, 70-89% 노란색, 90%+ 빨간색), 진행률 표시줄 및 Git 브랜치 정보 등 여러 기술을 결합합니다. 각 `print` 또는 `echo` 문은 별도의 행을 생성합니다:

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  input=$(cat)

  MODEL=$(echo "$input" | jq -r '.model.display_name')
  DIR=$(echo "$input" | jq -r '.workspace.current_dir')
  COST=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
  PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)
  DURATION_MS=$(echo "$input" | jq -r '.cost.total_duration_ms // 0')

  CYAN='\033[36m'; GREEN='\033[32m'; YELLOW='\033[33m'; RED='\033[31m'; RESET='\033[0m'

  # 컨텍스트 사용량에 따라 바 색상 선택
  if [ "$PCT" -ge 90 ]; then BAR_COLOR="$RED"
  elif [ "$PCT" -ge 70 ]; then BAR_COLOR="$YELLOW"
  else BAR_COLOR="$GREEN"; fi

  FILLED=$((PCT / 10)); EMPTY=$((10 - FILLED))
  printf -v FILL "%${FILLED}s"; printf -v PAD "%${EMPTY}s"
  BAR="${FILL// /█}${PAD// /░}"

  MINS=$((DURATION_MS / 60000)); SECS=$(((DURATION_MS % 60000) / 1000))

  BRANCH=""
  git rev-parse --git-dir > /dev/null 2>&1 && BRANCH=" | 🌿 $(git branch --show-current 2>/dev/null)"

  echo -e "${CYAN}[$MODEL]${RESET} 📁 ${DIR##*/}$BRANCH"
  COST_FMT=$(printf '$%.2f' "$COST")
  echo -e "${BAR_COLOR}${BAR}${RESET} ${PCT}% | ${YELLOW}${COST_FMT}${RESET} | ⏱️ ${MINS}m ${SECS}s"
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys, subprocess, os

  data = json.load(sys.stdin)
  model = data['model']['display_name']
  directory = os.path.basename(data['workspace']['current_dir'])
  cost = data.get('cost', {}).get('total_cost_usd', 0) or 0
  pct = int(data.get('context_window', {}).get('used_percentage', 0) or 0)
  duration_ms = data.get('cost', {}).get('total_duration_ms', 0) or 0

  CYAN, GREEN, YELLOW, RED, RESET = '\033[36m', '\033[32m', '\033[33m', '\033[31m', '\033[0m'

  bar_color = RED if pct >= 90 else YELLOW if pct >= 70 else GREEN
  filled = pct // 10
  bar = '█' * filled + '░' * (10 - filled)

  mins, secs = duration_ms // 60000, (duration_ms % 60000) // 1000

  try:
      branch = subprocess.check_output(['git', 'branch', '--show-current'], text=True, stderr=subprocess.DEVNULL).strip()
      branch = f" | 🌿 {branch}" if branch else ""
  except:
      branch = ""

  print(f"{CYAN}[{model}]{RESET} 📁 {directory}{branch}")
  print(f"{bar_color}{bar}{RESET} {pct}% | {YELLOW}${cost:.2f}{RESET} | ⏱️ {mins}m {secs}s")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  const { execSync } = require('child_process');
  const path = require('path');

  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;
      const dir = path.basename(data.workspace.current_dir);
      const cost = data.cost?.total_cost_usd || 0;
      const pct = Math.floor(data.context_window?.used_percentage || 0);
      const durationMs = data.cost?.total_duration_ms || 0;

      const CYAN = '\x1b[36m', GREEN = '\x1b[32m', YELLOW = '\x1b[33m', RED = '\x1b[31m', RESET = '\x1b[0m';

      const barColor = pct >= 90 ? RED : pct >= 70 ? YELLOW : GREEN;
      const filled = Math.floor(pct / 10);
      const bar = '█'.repeat(filled) + '░'.repeat(10 - filled);

      const mins = Math.floor(durationMs / 60000);
      const secs = Math.floor((durationMs % 60000) / 1000);

      let branch = '';
      try {
          branch = execSync('git branch --show-current', { encoding: 'utf8', stdio: ['pipe', 'pipe', 'ignore'] }).trim();
          branch = branch ? ` | 🌿 ${branch}` : '';
      } catch {}

      console.log(`${CYAN}[${model}]${RESET} 📁 ${dir}${branch}`);
      console.log(`${barColor}${bar}${RESET} ${pct}% | ${YELLOW}$${cost.toFixed(2)}${RESET} | ⏱️ ${mins}m ${secs}s`);
  });
  ```
</CodeGroup>

### 클릭 가능한 링크

이 예시는 GitHub 리포지토리에 대한 클릭 가능한 링크를 생성합니다. Git 원격 URL을 읽고 `sed`를 사용하여 SSH 형식을 HTTPS로 변환한 다음 리포지토리 이름을 OSC 8 이스케이프 코드로 감쌉니다. 브라우저에서 링크를 열려면 Cmd(macOS) 또는 Ctrl(Windows/Linux)을 누르고 클릭하세요.

<Frame>
  <img src="https://mintcdn.com/claude-code/nibzesLaJVh4ydOq/images/statusline-links.png?fit=max&auto=format&n=nibzesLaJVh4ydOq&q=85&s=4bcc6e7deb7cf52f41ab85a219b52661" alt="GitHub 리포지토리에 대한 클릭 가능한 링크를 보여주는 상태 표시줄" width="726" height="198" data-path="images/statusline-links.png" />
</Frame>

각 스크립트는 Git 원격 URL을 가져와 SSH 형식을 HTTPS로 변환하고 리포지토리 이름을 OSC 8 이스케이프 코드로 감쌉니다. Bash 버전은 여러 셸에 걸쳐 `echo -e`보다 안정적으로 백슬래시 이스케이프를 해석하는 `printf '%b'`를 사용합니다:

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  input=$(cat)

  MODEL=$(echo "$input" | jq -r '.model.display_name')

  # git SSH URL을 HTTPS로 변환
  REMOTE=$(git remote get-url origin 2>/dev/null | sed 's/git@github.com:/https:\/\/github.com\//' | sed 's/\.git$//')

  if [ -n "$REMOTE" ]; then
      REPO_NAME=$(basename "$REMOTE")
      # OSC 8 형식: \e]8;;URL\a 그 후 텍스트 그 후 \e]8;;\a
      # printf %b는 셸 전반에서 이스케이프 시퀀스를 안정적으로 해석합니다.
      printf '%b' "[$MODEL] 🔗 \e]8;;${REMOTE}\a${REPO_NAME}\e]8;;\a\n"
  else
      echo "[$MODEL]"
  fi
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys, subprocess, re, os

  data = json.load(sys.stdin)
  model = data['model']['display_name']

  # Git 원격 URL 가져오기
  try:
      remote = subprocess.check_output(
          ['git', 'remote', 'get-url', 'origin'],
          stderr=subprocess.DEVNULL, text=True
      ).strip()
      # SSH를 HTTPS 형식으로 변환
      remote = re.sub(r'^git@github\.com:', 'https://github.com/', remote)
      remote = re.sub(r'\.git$', '', remote)
      repo_name = os.path.basename(remote)
      # OSC 8 이스케이프 시퀀스
      link = f"\033]8;;{remote}\a{repo_name}\033]8;;\a"
      print(f"[{model}] 🔗 {link}")
  except:
      print(f"[{model}]")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  const { execSync } = require('child_process');
  const path = require('path');

  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;

      try {
          let remote = execSync('git remote get-url origin', { encoding: 'utf8', stdio: ['pipe', 'pipe', 'ignore'] }).trim();
          // SSH를 HTTPS 형식으로 변환
          remote = remote.replace(/^git@github\.com:/, 'https://github.com/').replace(/\.git$/, '');
          const repoName = path.basename(remote);
          // OSC 8 이스케이프 시퀀스
          const link = `\x1b]8;;${remote}\x07${repoName}\x1b]8;;\x07`;
          console.log(`[${model}] 🔗 ${link}`);
      } catch {
          console.log(`[${model}]`);
      }
  });
  ```
</CodeGroup>

### 속도 제한 사용량

상태 표시줄에 Claude.ai 구독 속도 제한 사용량을 표시합니다. `rate_limits` 객체에는 `five_hour` (5시간 이동 창) 및 `seven_day` (주간) 창이 포함됩니다. 각 창은 `used_percentage` (0-100) 및 `resets_at` (창이 재설정되는 Unix 에포크 초)를 제공합니다.

이 필드는 첫 번째 API 응답 이후 Claude.ai 구독자(Pro/Max)에게만 나타납니다. 각 스크립트는 누락된 필드를 정상적으로 처리합니다:

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  input=$(cat)

  MODEL=$(echo "$input" | jq -r '.model.display_name')
  # "// empty"는 rate_limits가 없을 때 출력을 내보내지 않습니다.
  FIVE_H=$(echo "$input" | jq -r '.rate_limits.five_hour.used_percentage // empty')
  WEEK=$(echo "$input" | jq -r '.rate_limits.seven_day.used_percentage // empty')

  LIMITS=""
  [ -n "$FIVE_H" ] && LIMITS="5h: $(printf '%.0f' "$FIVE_H")%"
  [ -n "$WEEK" ] && LIMITS="${LIMITS:+$LIMITS }7d: $(printf '%.0f' "$WEEK")%"

  [ -n "$LIMITS" ] && echo "[$MODEL] | $LIMITS" || echo "[$MODEL]"
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys

  data = json.load(sys.stdin)
  model = data['model']['display_name']

  parts = []
  rate = data.get('rate_limits', {})
  five_h = rate.get('five_hour', {}).get('used_percentage')
  week = rate.get('seven_day', {}).get('used_percentage')

  if five_h is not None:
      parts.append(f"5h: {five_h:.0f}%")
  if week is not None:
      parts.append(f"7d: {week:.0f}%")

  if parts:
      print(f"[{model}] | {' '.join(parts)}")
  else:
      print(f"[{model}]")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;

      const parts = [];
      const fiveH = data.rate_limits?.five_hour?.used_percentage;
      const week = data.rate_limits?.seven_day?.used_percentage;

      if (fiveH != null) parts.push(`5h: ${Math.round(fiveH)}%`);
      if (week != null) parts.push(`7d: ${Math.round(week)}%`);

      console.log(parts.length ? `[${model}] | ${parts.join(' ')}` : `[${model}]`);
  });
  ```
</CodeGroup>

### 부하가 큰 작업 캐싱

상태 표시줄 스크립트는 활성 세션 중에 자주 실행됩니다. `git status` 또는 `git diff`와 같은 명령은 특히 대규모 리포지토리에서 느릴 수 있습니다. 이 예시는 Git 정보를 임시 파일에 캐싱하고 5초마다만 새로 고칩니다.

캐시 파일 이름은 세션 내의 상태 표시줄 호출 전반에 걸쳐 안정적이어야 하지만, 서로 다른 리포지토리의 동시 세션이 서로의 캐시된 Git 상태를 읽지 않도록 세션 전체에서 고유해야 합니다. `$$`, `os.getpid()` 또는 `process.pid`와 같은 프로세스 기반 식별자는 호출할 때마다 변경되므로 캐시 목적에 부합하지 않습니다. 대신 JSON 입력의 `session_id`를 사용하세요. 이는 세션 수명 동안 안정적이며 세션별로 고유합니다.

각 스크립트는 Git 명령을 실행하기 전에 캐시 파일이 없거나 5초보다 오래되었는지 확인합니다:

<CodeGroup>
  ```bash Bash theme={null}
  #!/bin/bash
  input=$(cat)

  MODEL=$(echo "$input" | jq -r '.model.display_name')
  DIR=$(echo "$input" | jq -r '.workspace.current_dir')
  SESSION_ID=$(echo "$input" | jq -r '.session_id')

  CACHE_FILE="/tmp/statusline-git-cache-$SESSION_ID"
  CACHE_MAX_AGE=5  # 초

  cache_is_stale() {
      [ ! -f "$CACHE_FILE" ] || \
      # stat -c %Y (Linux) 또는 stat -f %m (macOS)는 파일의 마지막 수정
      # 시간을 인쇄합니다. Linux 양식이 먼저 실행되어야 합니다: Linux에서는
      # macOS 양식이 실패하기 전에 stdout에 파일 시스템 리포트를 인쇄하므로
      # 해당 출력이 명령 치환에 포착되어 연산을 중단시킬 수 있습니다.
      [ $(($(date +%s) - $(stat -c %Y "$CACHE_FILE" 2>/dev/null || stat -f %m "$CACHE_FILE" 2>/dev/null || echo 0))) -gt $CACHE_MAX_AGE ]
  }

  if cache_is_stale; then
      if git rev-parse --git-dir > /dev/null 2>&1; then
          BRANCH=$(git branch --show-current 2>/dev/null)
          STAGED=$(git diff --cached --numstat 2>/dev/null | wc -l | tr -d ' ')
          MODIFIED=$(git diff --numstat 2>/dev/null | wc -l | tr -d ' ')
          echo "$BRANCH|$STAGED|$MODIFIED" > "$CACHE_FILE"
      else
          echo "||" > "$CACHE_FILE"
      fi
  fi

  IFS='|' read -r BRANCH STAGED MODIFIED < "$CACHE_FILE"

  if [ -n "$BRANCH" ]; then
      echo "[$MODEL] 📁 ${DIR##*/} | 🌿 $BRANCH +$STAGED ~$MODIFIED"
  else
      echo "[$MODEL] 📁 ${DIR##*/}"
  fi
  ```

  ```python Python theme={null}
  #!/usr/bin/env python3
  import json, sys, subprocess, os, time

  data = json.load(sys.stdin)
  model = data['model']['display_name']
  directory = os.path.basename(data['workspace']['current_dir'])
  session_id = data['session_id']

  CACHE_FILE = f"/tmp/statusline-git-cache-{session_id}"
  CACHE_MAX_AGE = 5  # 초

  def cache_is_stale():
      if not os.path.exists(CACHE_FILE):
          return True
      return time.time() - os.path.getmtime(CACHE_FILE) > CACHE_MAX_AGE

  if cache_is_stale():
      try:
          subprocess.check_output(['git', 'rev-parse', '--git-dir'], stderr=subprocess.DEVNULL)
          branch = subprocess.check_output(['git', 'branch', '--show-current'], text=True).strip()
          staged = subprocess.check_output(['git', 'diff', '--cached', '--numstat'], text=True).strip()
          modified = subprocess.check_output(['git', 'diff', '--numstat'], text=True).strip()
          staged_count = len(staged.split('\n')) if staged else 0
          modified_count = len(modified.split('\n')) if modified else 0
          with open(CACHE_FILE, 'w') as f:
              f.write(f"{branch}|{staged_count}|{modified_count}")
      except:
          with open(CACHE_FILE, 'w') as f:
              f.write("||")

  with open(CACHE_FILE) as f:
      branch, staged, modified = f.read().strip().split('|')

  if branch:
      print(f"[{model}] 📁 {directory} | 🌿 {branch} +{staged} ~{modified}")
  else:
      print(f"[{model}] 📁 {directory}")
  ```

  ```javascript Node.js theme={null}
  #!/usr/bin/env node
  const { execSync } = require('child_process');
  const fs = require('fs');
  const path = require('path');

  let input = '';
  process.stdin.on('data', chunk => input += chunk);
  process.stdin.on('end', () => {
      const data = JSON.parse(input);
      const model = data.model.display_name;
      const dir = path.basename(data.workspace.current_dir);
      const sessionId = data.session_id;

      const CACHE_FILE = `/tmp/statusline-git-cache-${sessionId}`;
      const CACHE_MAX_AGE = 5; // 초

      const cacheIsStale = () => {
          if (!fs.existsSync(CACHE_FILE)) return true;
          return (Date.now() / 1000) - fs.statSync(CACHE_FILE).mtimeMs / 1000 > CACHE_MAX_AGE;
      };

      if (cacheIsStale()) {
          try {
              execSync('git rev-parse --git-dir', { stdio: 'ignore' });
              const branch = execSync('git branch --show-current', { encoding: 'utf8' }).trim();
              const staged = execSync('git diff --cached --numstat', { encoding: 'utf8' }).trim().split('\n').filter(Boolean).length;
              const modified = execSync('git diff --numstat', { encoding: 'utf8' }).trim().split('\n').filter(Boolean).length;
              fs.writeFileSync(CACHE_FILE, `${branch}|${staged}|${modified}`);
          } catch {
              fs.writeFileSync(CACHE_FILE, '||');
          }
      }

      const [branch, staged, modified] = fs.readFileSync(CACHE_FILE, 'utf8').trim().split('|');

      if (branch) {
          console.log(`[${model}] 📁 ${dir} | 🌿 ${branch} +${staged} ~${modified}`);
      } else {
          console.log(`[${model}] 📁 ${dir}`);
      }
  });
  ```
</CodeGroup>

### Windows 구성

Windows에서 Claude Code는 Git Bash가 설치되어 있을 때 Git Bash를 통해 상태 표시줄 명령을 실행하고, Git Bash가 없을 때 PowerShell을 통해 실행합니다.

Git Bash는 따옴표로 감싸지지 않은 백슬래시를 이스케이프 문자로 처리하므로 `C:\Users\username\script.mjs`와 같은 Windows 스타일 경로는 구분 기호가 제거된 상태로 스크립트 러너에 도달하여 볼 수 있는 오류 없이 명령이 실패합니다. 아래 예시에 나와 있듯이 `command` 문자열의 파일 경로는 슬래시로 작성하세요. `~` 축약형도 작동하며 Windows 홈 디렉터리로 확장됩니다.

상태 표시줄로 PowerShell 스크립트를 실행하려면 `powershell`을 통해 호출하세요. 이는 Claude Code가 Git Bash를 통해 명령을 라우팅하든 PowerShell을 통해 라우팅하든 작동합니다:

<CodeGroup>
  ```json settings.json theme={null}
  {
    "statusLine": {
      "type": "command",
      "command": "powershell -NoProfile -File C:/Users/username/.claude/statusline.ps1"
    }
  }
  ```

  ```powershell statusline.ps1 theme={null}
  $input_json = $input | Out-String | ConvertFrom-Json
  $cwd = $input_json.cwd
  $model = $input_json.model.display_name
  $used = $input_json.context_window.used_percentage
  $dirname = Split-Path $cwd -Leaf

  if ($used) {
      Write-Host "$dirname [$model] ctx: $used%"
  } else {
      Write-Host "$dirname [$model]"
  }
  ```
</CodeGroup>

또는 Git Bash가 설치되어 있을 때 Bash 스크립트를 직접 실행하세요:

<CodeGroup>
  ```json settings.json theme={null}
  {
    "statusLine": {
      "type": "command",
      "command": "~/.claude/statusline.sh"
    }
  }
  ```

  ```bash statusline.sh theme={null}
  #!/usr/bin/env bash
  input=$(cat)
  cwd=$(echo "$input" | grep -o '"cwd":"[^"]*"' | cut -d'"' -f4)
  model=$(echo "$input" | grep -o '"display_name":"[^"]*"' | cut -d'"' -f4)
  dirname="${cwd##*[/\\]}"
  echo "$dirname [$model]"
  ```
</CodeGroup>

## 서브에이전트 상태 표시줄

`subagentStatusLine` 설정은 프롬프트 아래의 에이전트 패널에 표시되는 각 [서브에이전트](/docs/en/sub-agents)에 대해 맞춤형 행 본문을 렌더링합니다. 기본 `name · description · token count` 행을 사용자 고유의 서식으로 교체할 때 사용하세요.

```json theme={null}
{
  "subagentStatusLine": {
    "type": "command",
    "command": "~/.claude/subagent-statusline.sh"
  }
}
```

이 명령은 새로 고침 틱당 한 번 실행되며 표시되는 모든 서브에이전트 행을 단일 JSON 객체로 stdin에서 수신합니다. 입력에는 [기본 훅 필드](/docs/en/hooks#common-input-fields), 사용 가능한 행 너비가 포함된 `columns` 필드 및 `tasks` 배열이 포함됩니다. 각 작업에는 `id`, `name`, `type`, `status`, `description`, `label`, `startTime`, `model`, `effort`, `contextWindowSize`, `tokenCount`, `tokenSamples`, `cwd`가 있습니다.

작업별 `model` 필드는 작업이 실행되는 해석된 모델 ID입니다. `contextWindowSize`는 메인 상태 표시줄의 `context_window.context_window_size`와 동일한 방식으로 계산되는 해당 모델의 토큰 단위 컨텍스트 창이므로 `tokenCount`에서 행별 비율을 렌더링할 수 있습니다. 두 필드 모두 Claude Code v2.1.205 이상이 필요하며 아직 모델이 해석되지 않은 작업의 경우 생략됩니다.

작업별 `effort` 필드는 해당 서브에이전트에 설정된 추론 노력으로, [정의 프론트매터](/docs/en/sub-agents#supported-frontmatter-fields) 또는 개별 호출 시에 지정됩니다. 값은 노력 수준 문자열 `low`, `medium`, `high`, `xhigh`, `max` 중 하나이거나 숫자 토큰 예산입니다. 이 필드는 작성된 구성 값을 보고합니다: 모델이 해당 수준을 지원하지 않는 경우 Claude Code가 실제로 적용하는 노력은 다를 수 있습니다. 이 필드는 Claude Code v2.1.214 이상이 필요하며 서브에이전트가 세션의 노력 수준을 상속받을 때는 존재하지 않습니다.

오버라이드하려는 행당 `{"id": "<task id>", "content": "<row body>"}` 형식으로 stdout에 한 줄의 JSON을 작성하세요. `content` 문자열은 ANSI 색상 및 OSC 8 하이퍼링크를 포함하여 있는 그대로 렌더링됩니다. 해당 행의 기본 렌더링을 유지하려면 작업의 `id`를 생략하고, 행을 숨기려면 빈 `content` 문자열을 내보내세요.

`statusLine`에 적용되는 동일한 신뢰 및 `disableAllHooks` 게이트가 여기에도 적용됩니다. 플러그인은 [`settings.json`](/docs/en/plugins-reference#standard-plugin-layout)에 기본 `subagentStatusLine`을 제공할 수 있습니다.

## 팁

* **가짜 입력으로 테스트**: `echo '{"model":{"display_name":"Opus"},"workspace":{"current_dir":"/home/user/project"},"context_window":{"used_percentage":25},"session_id":"test-session-abc"}' | ./statusline.sh`
* **출력 짧게 유지**: 상태 표시줄은 너비가 제한되어 있으므로 긴 출력은 잘리거나 부자연스럽게 줄바꿈될 수 있습니다.
* **느린 작업 캐싱**: 스크립트는 활성 세션 중에 자주 실행되므로 `git status`와 같은 명령은 지연을 유발할 수 있습니다. 이를 처리하는 방법은 [캐싱 예시](#부하가-큰-작업-캐싱)를 참조하세요.

[ccstatusline](https://github.com/sirmalloc/ccstatusline) 및 [starship-claude](https://github.com/martinemde/starship-claude)와 같은 커뮤니티 프로젝트는 테마 및 추가 기능을 포함하는 사전 구축된 구성을 제공합니다.

## 문제 해결

**상태 표시줄이 나타나지 않음**

* 스크립트가 실행 가능한지 확인하세요: `chmod +x ~/.claude/statusline.sh`
* 스크립트가 stderr가 아닌 stdout으로 출력되는지 확인하세요.
* 출력이 생성되는지 확인하기 위해 스크립트를 수동으로 실행해 보세요.
* Git Bash가 설치된 Windows에서는 `command` 경로의 백슬래시가 스크립트가 실행되기 전에 이스케이프 문자로 소비될 가능성이 높습니다. 경로에 슬래시를 사용하세요. [Windows 구성](#windows-구성)을 참조하세요.
* 설정에서 `disableAllHooks`가 `true`로 설정된 경우 상태 표시줄도 비활성화됩니다. 이 설정을 제거하거나 `false`로 설정하여 다시 활성화하세요.
* 세션의 첫 번째 상태 표시줄 호출에서 종료 코드 및 stderr를 로그로 기록하려면 `claude --debug`를 실행하세요.
* 오류를 표시하기 위해 Claude에게 설정 파일을 읽고 `statusLine` 명령을 직접 실행하도록 요청하세요.

**상태 표시줄에 `--` 또는 빈 값이 표시됨**

* 첫 번째 API 응답이 완료되기 전에는 필드가 `null`일 수 있습니다.
* jq의 `// 0`과 같은 대체 기능으로 스크립트에서 null 값을 처리하세요.
* 여러 메시지 후에도 값이 비어 있으면 Claude Code를 재시작하세요.

**컨텍스트 비율에 예상치 못한 값이 표시됨**

* 가장 단순하고 정확한 컨텍스트 상태를 위해 `used_percentage`를 사용하세요.
* 컨텍스트 비율은 각각 계산되는 시점으로 인해 `/context` 출력과 다를 수 있습니다.

**OSC 8 링크를 클릭할 수 없음**

* 터미널이 OSC 8 하이퍼링크를 지원하는지 확인하세요 (iTerm2, Kitty, WezTerm).
* Terminal.app은 클릭 가능한 링크를 지원하지 않습니다.
* 링크 텍스트는 표시되지만 클릭할 수 없는 경우 Claude Code가 터미널에서 하이퍼링크 지원을 감지하지 못했을 수 있습니다. 이는 자동 감지 목록에 없는 Windows Terminal 및 기타 에뮬레이터에 흔히 영향을 미칩니다. Claude Code를 실행하기 전에 감지를 오버라이드하려면 `FORCE_HYPERLINK` 환경 변수를 설정하세요:

  ```bash theme={null}
  FORCE_HYPERLINK=1 claude
  ```

  PowerShell에서는 먼저 현재 세션에서 변수를 설정합니다:

  ```powershell theme={null}
  $env:FORCE_HYPERLINK = "1"; claude
  ```

* SSH 및 tmux 세션은 구성에 따라 OSC 시퀀스를 제거할 수 있습니다.
* 이스케이프 시퀀스가 `\e]8;;`과 같은 리터럴 텍스트로 나타나면 보다 안정적인 이스케이프 처리를 위해 `echo -e` 대신 `printf '%b'`를 사용하세요.

**이스케이프 시퀀스로 인한 디스플레이 문제**

* 복잡한 이스케이프 시퀀스(ANSI 색상, OSC 8 링크)는 다른 UI 업데이트와 겹칠 때 간혹 깨진 출력을 일으킬 수 있습니다.
* 깨진 텍스트가 보이는 경우 스크립트를 일반 텍스트 출력으로 단순화해 보세요.
* 이스케이프 코드가 있는 다중 행 상태 표시줄은 단일 행 일반 텍스트보다 렌더링 문제가 발생하기 쉽습니다.

**워크스페이스 신뢰 필요**

* 상태 표시줄 명령은 현재 디렉터리에 대해 워크스페이스 신뢰 대화 상자를 수락한 경우에만 실행됩니다. `statusLine`은 셸 명령을 실행하므로 훅 및 기타 셸 실행 설정과 동일한 신뢰 수락이 필요합니다.
* 이 폴더에 대해 [워크스페이스 신뢰 대화 상자](/docs/en/security)를 수락하지 않은 경우 상태 표시줄은 비어 있는 상태로 유지되며 `claude --debug`는 `Status line command skipped: workspace trust not accepted`를 로깅합니다. 이를 활성화하려면 Claude Code를 재시작하고 신뢰 대화 상자를 수락하세요.

**스크립트 오류 또는 멈춤**

* 영이 아닌 코드로 종료되거나 출력을 생성하지 않는 스크립트는 상태 표시줄이 빈 상태가 되게 만듭니다.
* 느린 스크립트는 완료될 때까지 상태 표시줄 업데이트를 차단합니다. 오래된 출력을 방지하기 위해 스크립트를 빠르게 유지하세요.
* 느린 스크립트가 실행 중인 동안 새 업데이트가 트리거되면 실행 중인 스크립트가 취소됩니다.
* 구성을 수행하기 전에 가짜 입력으로 스크립트를 독립적으로 테스트하세요.

**알림이 상태 표시줄 행을 공유함**

* MCP 서버 오류 및 자동 업데이트와 같은 시스템 알림은 상태 표시줄과 동일한 행의 오른쪽에 표시됩니다. 컨텍스트 부족 경고와 같은 일시적인 알림도 이 영역을 순환합니다.
* 세부 정보 모드를 활성화하면 이 영역에 토큰 카운터가 추가됩니다.
* 좁은 터미널에서는 이러한 알림으로 인해 상태 표시줄 출력이 잘릴 수 있습니다.
