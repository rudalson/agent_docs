> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 명령 (Commands)

> 내장 명령 및 번들 스킬을 포함하여 Claude Code에서 사용할 수 있는 명령에 대한 완전한 레퍼런스입니다.

명령은 세션 내부에서 Claude Code를 제어합니다. 모델 전환, 권한 관리, 컨텍스트 지우기, 워크플로우 실행 등을 위한 빠른 방법을 제공합니다.

`/`를 입력하면 사용할 수 있는 모든 명령이 표시되며, `/` 뒤에 글자를 입력하여 필터링할 수 있습니다.

명령은 메시지 시작 부분에서만 인식됩니다. 명령 이름 뒤에 오는 텍스트는 해당 명령의 인수가 됩니다. {/* min-version: 2.1.199 */}v2.1.199부터 [스킬](/docs/en/skills#pass-arguments-to-skills)은 예외입니다: `/skill-a /skill-b do XYZ`와 같이 스킬 호출 뒤에 추가 스킬이 이어지면 시작 부분에 지정된 모든 스킬을 로드하고 후속 텍스트를 각 스킬의 인수로 전달합니다. 최대 6개의 스킬을 체인으로 연결할 수 있습니다.

Claude가 응답하는 동안 명령을 전송하면 현재 턴이 완료된 후 대기열에 추가되어 실행됩니다. `/status`, `/tasks`, `/usage`와 같은 일부 명령은 응답을 중단하지 않고 즉시 실행됩니다.

## 일반적인 워크플로우에서의 명령

대부분의 명령은 프로젝트 설정부터 변경 사항 배포까지 세션의 특정 시점에 유용합니다.

**리포지토리에서의 첫 번째 세션.** 스타터 `CLAUDE.md`를 생성하려면 `/init`을 실행한 다음 `/memory`를 실행하여 이를 정교화하세요. `/mcp`를 사용하여 프로젝트에 필요한 서버를 설정하고, Claude에게 원하는 [하위 에이전트](/docs/en/sub-agents)를 생성하도록 요청한 후 `/permissions`를 실행하여 승인 규칙을 설정하세요.

**작업 진행 중.** `/plan`은 대규모 변경을 시작하기 전에 플랜 모드로 전환합니다. `/model` 및 `/effort`는 사용 중인 모델과 적용할 추론 양을 조정합니다. 대화가 길어지면 `/context`는 컨텍스트 창을 채우고 있는 항목을 보여주고 `/compact`는 이를 요약하여 공간을 확보합니다. 대화 기록에 추가되지 않아야 하는 빠른 부가 질문에는 `/btw`를 사용하세요.

**작업 병렬 실행.** Claude는 사이드 작업을 [하위 에이전트](/docs/en/sub-agents)에 위임하며, `/tasks`는 완료된 하위 에이전트를 포함하여 현재 세션의 백그라운드 작업을 나열합니다. `/background`는 전체 세션을 분리하여 [백그라운드 에이전트](/docs/en/agent-view)로 계속 실행되도록 하고 터미널을 자유롭게 합니다. 코드베이스 전체에 걸친 대규모 변경의 경우 `/batch`는 이를 독립적인 단위로 분해하여 각 단위를 자체 [git worktree](/docs/en/worktrees)에서 실행합니다. 이러한 접근 방식 간의 관계는 [에이전트 병렬 실행](/docs/en/agents)을 참조하세요.

**배포 전.** `/diff`는 변경된 내용을 보여주고, `/code-review`는 정확성 버그 및 정리 기회를 확인하며 `--fix`로 발견 항목을 적용할 수 있습니다. `/review`는 GitHub 풀 리퀘스트의 빠른 단일 패스 읽기 전용 리뷰를 제공하고, `/code-review <level> <pr#>`은 PR의 다중 에이전트 리뷰를 실행하며, `/security-review`는 보안 취약점을 확인합니다. `/code-review ultra`는 클라우드에서 다중 에이전트 리뷰를 실행합니다.

**세션 사이.** `/clear`는 프로젝트 메모리를 유지하면서 새 작업에 대해 새로 시작합니다. `/resume`은 이전 대화로 돌아가고, `/branch`는 현재 대화를 분기하여 다른 방향을 시도하며, `/fork`는 이를 새 [백그라운드 세션](/docs/en/agent-view)으로 복사합니다. `/teleport`는 웹 세션을 이 터미널로 가져오고, `/remote-control`을 사용하면 다른 기기에서 이 로컬 세션을 계속할 수 있습니다.

**문제가 발생했을 때.** `/rewind`는 코드와 대화를 체크포인트로 되돌리거나 대화의 일부를 요약합니다. `/doctor`는 설치 및 구성 문제를 진단하고 수정할 수 있는 설정 점검을 실행하고, `/debug`는 런타임 문제를 진단하며, `/feedback`은 세션 컨텍스트가 첨부된 버그를 보고합니다.

## 모든 명령

아래 테이블에는 Claude Code에 포함된 모든 명령이 나열되어 있습니다. 대부분은 CLI에 동작이 코딩된 내장 명령입니다. 다음 두 가지 유형의 항목이 표시됩니다.

* **[Skill](/docs/en/skills#bundled-skills)**: 번들 스킬입니다. 직접 작성하는 스킬처럼 작동합니다(Claude에게 전달되는 프롬프트이며, 관련 있는 경우 Claude가 자동으로 호출할 수도 있음).
  * {/* min-version: 2.1.215 */}`/verify` 및 `/code-review`는 직접 호출할 때만 실행됩니다. v2.1.215 이전에는 Claude가 스스로 실행할 수도 있었습니다.
* **[Workflow](/docs/en/workflows#bundled-workflows)**: 여러 하위 에이전트에 작업을 분산하고 백그라운드에서 실행되는 번들 [동적 워크플로우](/docs/en/workflows)입니다.

자신만의 명령을 추가하려면 [스킬](/docs/en/skills)을 참조하세요.

아래 테이블에서 `<arg>`는 필수 인수를 나타내고 `[arg]`는 선택적 인수를 나타냅니다.

<Note>
  모든 명령이 모든 사용자에게 나타나는 것은 아닙니다. 사용 가능 여부는 플랫폼, 요금제 및 환경에 따라 다릅니다. 예를 들어 `/desktop`은 Claude 구독으로 로그인한 경우 macOS 및 Windows에만 표시되며, `/upgrade`는 Enterprise 요금제에는 표시되지 않습니다.
</Note>

| 명령 | 목적 |
| :--- | :--- |
| `/add-dir <path>` | 현재 세션 동안 파일 접근을 위한 작업 디렉토리를 추가합니다. 경로 일부를 입력하면 일치하는 디렉토리 제안이 표시되며 `Tab`을 눌러 수락합니다. 대부분의 `.claude/` 구성은 추가된 디렉토리에서 [탐색되지 않습니다](/docs/en/permissions#additional-directories-grant-file-access-not-configuration). 나중에 `--continue` 또는 `--resume`을 사용하여 추가된 디렉토리에서 세션을 재개할 수 있습니다. |
| `/advisor [model\|off]` | 작업 중 주요 시점에 두 번째 모델에 조언을 구하는 [어드바이저 도구](/docs/en/advisor)를 활성화하거나 비활성화합니다. `opus`, `sonnet` 또는 전체 모델 ID를 받습니다. {/* min-version: 2.1.210 */}Claude Code는 [Fable 5를 어드바이저로 제공하지 않으며](/docs/en/advisor#enable-the-advisor) `/advisor fable`을 거부합니다. 인수 없이 실행하면 선택기가 열립니다. |
| `/agents` | {/* min-version: 2.1.198 */}v2.1.198부터 `/agents`를 실행하면 Claude에게 [하위 에이전트](/docs/en/sub-agents) 생성/관리를 요청하거나 `.claude/agents/` 또는 `~/.claude/agents/`를 직접 편집하라는 안내문이 출력됩니다. {/* max-version: 2.1.197 */}v2.1.197 이하에서는 하위 에이전트 구성을 생성하고 관리하는 대화형 인터페이스가 열립니다. |
| `/autofix-pr [prompt]` | 현재 브랜치의 PR을 모니터링하고 CI가 실패하거나 리뷰어가 주석을 남길 때 수정 사항을 푸시하는 [Claude Code on the web](/docs/en/claude-code-on-the-web#auto-fix-pull-requests) 세션을 생성합니다. `gh pr view`로 체크아웃된 브랜치에서 열린 PR을 감지합니다. 다른 PR을 모니터링하려면 먼저 해당 브랜치를 체크아웃하세요. 기본적으로 클라우드 세션은 모든 CI 실패 및 리뷰 주석을 수정하도록 안내받습니다. 예: `/autofix-pr only fix lint and type errors`와 같이 프롬프트를 전달하여 다른 지침을 부여하세요. `gh` CLI 및 [Claude Code on the web](/docs/en/claude-code-on-the-web) 접근 권한이 필요합니다. |
| `/background [prompt]` | 현재 세션을 분리하여 [백그라운드 에이전트](/docs/en/agent-view)로 실행되도록 하고 터미널을 자유롭게 합니다. 분리하기 전에 지침을 하나 더 보내려면 프롬프트를 전달하세요. `claude agents`로 세션을 모니터링하세요. 이 세션이 계속 실행되는 동안 대화를 새 백그라운드 세션으로 복사하려면 `/fork`를 사용하세요. 별칭: `/bg` |
| `/batch <instruction>` | **[Skill](/docs/en/skills#bundled-skills).** 코드베이스 전반에 걸친 대규모 변경 사항을 병렬로 조율합니다. 코드베이스를 조사하고 작업을 5~30개의 독립적인 단위로 분해하여 계획을 제시합니다. 승인되면 격리된 [git worktree](/docs/en/worktrees)에서 단위당 하나의 [백그라운드 하위 에이전트](/docs/en/sub-agents#run-subagents-in-foreground-or-background)를 생성합니다. 각 하위 에이전트는 해당 단위를 구현하고 테스트를 실행하며 풀 리퀘스트를 엽니다. git 리포지토리가 필요합니다. 예: `/batch migrate src/ from Solid to React` |
| `/branch [name]` | 이 시점에서 현재 대화의 브랜치를 생성하여 기존 대화를 그대로 유지하면서 다른 방향을 시도할 수 있습니다. 브랜치로 전환하고 원본을 보존하며 `/resume`으로 돌아올 수 있습니다. 그것으로 전환하는 대신 별도의 [백그라운드 세션](/docs/en/agent-view)으로 복사본을 실행하려면 `/fork`를 사용하세요. 결과가 이 대화로 다시 보고되는 사이드 작업을 [하위 에이전트](/docs/en/sub-agents)에 전달하려면 `/subtask`를 사용하세요. |
| `/btw [question]` | 대화에 추가하지 않고 빠른 [사이드 질문](/docs/en/interactive-mode#side-questions-with-%2Fbtw)을 합니다. {/* min-version: 2.1.212 */}질문 없이 실행하면 이 세션의 가장 최근 사이드 질문 오버레이를 다시 열어 이전 답변을 둘러볼 수 있습니다. 사이드 질문이 아직 없으면 질문을 요청합니다. v2.1.212 이전에는 `/btw`에 질문이 필요했습니다. |
| `/bug [report]` | {/* min-version: 2.1.212 */}버그를 보고하거나 대화를 공유합니다. 세션 기록을 얼마나 포함할지 선택하고 아무것도 전송되기 전에 동의 화면에서 확인합니다. 퍼스트 파티 연결로 Anthropic에 로그인한 경우 보고서가 Anthropic으로 이동합니다. 서드파티 공급자이거나 Anthropic 자격 증명이 없는 경우 Claude Code는 사용자가 직접 전달할 수 있는 [`~/.claude/feedback-bundles/` 아래의 로컬 아카이브](/docs/en/data-usage#telemetry-services)에 보고서를 작성합니다. 별칭: `/share`. v2.1.212 이전에는 `/bug` 및 `/share`가 `/feedback`의 별칭이었습니다. |
| `/cd <path>` | {/* min-version: 2.1.169 */}이 세션을 새 작업 디렉토리로 이동합니다. 대화의 프롬프트 캐시는 보존됩니다: 시스템 프롬프트를 다시 작성하는 대신 새 디렉토리의 [`CLAUDE.md`](/docs/en/memory)가 메시지로 추가됩니다. 세션이 새 디렉토리의 프로젝트 저장소로 재배치되므로 `--resume` 및 `--continue`가 거기에서 세션을 찾습니다. 이전에 일해본 적이 없는 디렉토리인 경우 신뢰 여부를 묻는 프롬프트를 표시합니다. {/* min-version: 2.1.206 */}경로 일부를 입력하면 일치하는 디렉토리 제안이 표시되며 `Tab`을 눌러 수락합니다. 제안 기능에는 Claude Code v2.1.206 이상이 필요합니다. 세션을 이동하지 않고 추가 디렉토리에 접근 권한을 부여하려면 `/add-dir`을 사용하세요. [`Cd` 권한 규칙](/docs/en/permissions#cd)으로 `/cd` 대상을 제한하거나 비활성화할 수 있습니다. Claude Code v2.1.169 이상이 필요합니다. |
| `/chrome` | [Claude in Chrome](/docs/en/chrome) 설정을 구성합니다. |
| `/claude-api [migrate\|managed-agents-onboard]` | **[Skill](/docs/en/skills#bundled-skills).** 프로젝트 언어(Python, TypeScript, Java, Go, Ruby, C#, PHP 또는 cURL)에 대한 Claude API 참조 자료 및 Managed Agents 참조 자료를 로드합니다. 도구 사용, 스트리밍, 배치, 구조화된 출력 및 일반적인 함정을 다룹니다. 코드에서 `anthropic` 또는 `@anthropic-ai/sdk`를 가져올 때도 자동으로 활성화됩니다. 기존 Claude API 코드를 최신 모델로 업그레이드하려면 `/claude-api migrate`를 실행하세요. 스캔할 파일과 대상 모델을 묻고 모델 ID, thinking 구성 및 버전 간에 변경된 기타 파라미터를 업데이트합니다. 처음부터 새 Managed Agent를 생성하는 대화형 안내를 보려면 `/claude-api managed-agents-onboard`를 실행하세요. |
| `/clear [name]` | 빈 컨텍스트로 새 대화를 시작합니다. 이전 대화에 라벨을 지정하려면 이름을 전달하세요 (`/resume` 선택기용). 동일한 대화를 계속하면서 컨텍스트 공간을 확보하려면 대신 `/compact`를 사용하세요. `/resume`으로 이전 대화를 재개하거나 동일한 Claude Code 프로세스에서 {/* min-version: 2.1.191 */}[되감기 메뉴의 이전 세션 항목](/docs/en/checkpointing#rewind-past-a-cleared-conversation)에서 복원하세요. 별칭: `/reset`, `/new` |
| `/code-review [low\|medium\|high\|xhigh\|max\|ultra] [--fix] [--comment] [target]` | **[Skill](/docs/en/skills#bundled-skills).** 정확성 버그 및 정리 기회에 대해 현재 diff를 리뷰합니다. 발견 항목을 적용하려면 `--fix`, 인라인 GitHub PR 주석으로 게시하려면 `--comment`, 깊이 있는 [클라우드 리뷰](/docs/en/ultrareview)를 실행하려면 `ultra`를 전달하세요. 작업 투입 수준, 대상 지정 및 `/simplify`와의 관계는 [로컬에서 diff 리뷰하기](/docs/en/code-review#review-a-diff-locally)를 참조하세요. |
| `/color [color\|default]` | 현재 세션의 프롬프트 바 색상을 설정합니다. 사용 가능한 색상: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. 재설정하려면 `default`를 사용하고, 임의의 색상을 선택하려면 인수 없이 실행하세요. [Remote Control](/docs/en/remote-control)이 연결되어 있으면 색상이 claude.ai/code 동기화됩니다. {/* min-version: 2.1.205 */}비대화형 모드(`-p`)에서도 사용 가능하며 Claude Code v2.1.205 이상이 필요합니다. |
| `/compact [instructions]` | 지금까지의 대화를 요약하여 컨텍스트 공간을 확보합니다. 옵션으로 요약에 대한 집중 지침을 전달할 수 있습니다. [축소 시 규칙, 스킬, 메모리 파일 처리 방식](/docs/en/context-window#what-survives-compaction)을 참조하세요. |
| `/config [key=value ...]` | 테마, 모델, [출력 스타일](/docs/en/output-styles) 및 기타 환경설정을 조정할 수 있도록 [설정](/docs/en/settings) 인터페이스를 엽니다. {/* min-version: 2.1.181 */}v2.1.181부터 하나 이상의 `key=value` 쌍을 전달하여 인터페이스를 열지 않고 직접 설정을 변경할 수 있습니다 (예: `/config thinking=false`). {/* min-version: 2.1.182 */}v2.1.182부터 `/config theme=dark` 또는 `/config model=sonnet`과 같은 축약 키 이름도 허용됩니다. `key=value` 형식은 비대화형 모드(`-p`) 및 [Remote Control](/docs/en/remote-control)을 통한 Claude 모바일 앱에서도 작동합니다. 설정 가능한 모든 키와 옵션을 보려면 `/config --help`를 실행하세요. 별칭: `/settings` |
| `/context [all]` | 현재 컨텍스트 사용량을 색상이 지정된 그리드로 시각화합니다. 컨텍스트를 많이 사용하는 도구, 메모리 팽창 및 용량 경고에 대한 최적화 제안을 보여줍니다. {/* min-version: 2.1.216 */}대화가 컨텍스트 창을 초과하면 출력에 공간을 확보하는 명령과 얼마나 제한을 초과했는지 보여주는 [경고](/docs/en/errors#context-exceeds-the-token-limit)가 포함됩니다 (Claude Code v2.1.216 이상 필요). [전체 화면 모드](/docs/en/fullscreen)에서 `/context`는 그리드를 계속 표시하기 위해 항목별 분석을 축소합니다. 이를 펼치려면 `all`을 전달하세요. |
| `/copy [N]` | 마지막 어시스턴트 응답을 클립보드로 복사합니다. N번째 최근 응답을 복사하려면 숫자 `N`을 전달하세요 (`/copy 2`는 두 번째로 최근 응답을 복사함). 코드 블록이 있는 경우 개별 블록 또는 전체 응답을 선택할 수 있는 대화형 선택기를 표시합니다. SSH 환경에서 유용한 기능으로, 클립보드 대신 선택 항목을 파일에 기록하려면 선택기에서 `w`를 누르세요. |
| `/cost` | `/usage`에 대한 별칭입니다. |
| `/dataviz [request]` | **[Skill](/docs/en/skills#bundled-skills).** 차트, 그래프 및 대시보드에 대한 디자인 지침입니다. Claude가 데이터에 맞는 차트 형식을 선택하고, 역할별 색상을 할당하며, 번들 스크립트로 색맹 안전성 및 대비에 대해 팔레트를 검증하고, 마크/상호작용/접근성 규칙을 적용합니다. 브랜드 중립적인 플레이스홀더 팔레트를 사용하므로 자체 팔레트로 교체하세요. {/* min-version: 2.1.198 */}Claude Code v2.1.198 이상이 필요합니다. |
| `/debug [description]` | **[Skill](/docs/en/skills#bundled-skills).** 현재 세션의 디버그 로깅을 활성화하고 세션 디버그 로그를 읽어 문제를 해결합니다. `claude --debug`로 시작하지 않는 한 디버그 로깅은 기본적으로 꺼져 있으므로 세션 중간에 `/debug`를 실행하면 해당 시점부터 로그 캡처가 시작됩니다. 옵션으로 문제를 설명하여 분석을 원활하게 하세요. |
| `/deep-research <question>` | **[Workflow](/docs/en/workflows#bundled-workflows).** 질문에 대해 웹 검색을 분산 실행하고 소스를 가져와 교차 검증하며 인용이 포함된 보고서를 합성합니다. |
| `/design-login` | claude.ai 계정으로 `/design-sync`에 대한 디자인 시스템 접근 권한을 승인합니다. |
| `/design-sync [hint]` | **[Skill](/docs/en/skills#bundled-skills).** 리포지토리의 React 디자인 시스템을 변환하여 [Claude Design](https://claude.ai/design)에 업로드하므로 생성되는 디자인이 실제 컴포넌트를 사용합니다. 옵션으로 디자인 시스템 이름을 지정할 수 있습니다 (예: `/design-sync Acme DS`). 첫 동기화는 모든 컴포넌트를 검증하므로 대규모 리포지토리에서는 몇 시간이 걸릴 수 있습니다. Anthropic API에서 이용 가능하며 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 및 AWS 상의 Claude Platform에서는 기본 도구가 claude.ai에 도달할 수 없어 명령을 사용할 수 없습니다. |
| `/desktop` | Claude Code 데스크톱 앱에서 현재 세션을 계속합니다. macOS 또는 Windows와 Claude 구독이 필요합니다. 별칭: `/app` |
| `/diff` | 커밋되지 않은 변경 사항 및 턴별 diff를 보여주는 대화형 diff 뷰어를 엽니다. 좌/우 화살표를 사용하여 현재 git diff와 개별 Claude 턴 사이를 전환하고, 위/아래 화살표로 파일을 둘러보세요. Enter를 눌러 선택한 파일의 diff를 열고, 위/아래 또는 PageUp/PageDown으로 스크롤하며, Esc를 눌러 파일 목록으로 돌아갑니다. {/* min-version: 2.1.198 */}v2.1.198부터 뷰어가 열려 있는 동안 다른 터미널에서의 브랜치 전환이나 커밋 등 세션 외부에서 리포지토리의 git 상태가 변경되면 자동으로 새로고침됩니다. |
| `/doctor` | **[Skill](/docs/en/skills#bundled-skills).** 문제를 진단하고 수정할 수 있는 설정 점검을 실행합니다. 중복되거나 남은 설치, `PATH` 문제, 구문 분석 불가능한 설정 파일을 포함하여 설치 상태를 점검합니다. 컨텍스트 비용 대비 미사용 스킬, MCP 서버 및 플러그인을 찾고, 느린 [훅](/docs/en/hooks)을 지적하며, 릴리스 채널의 최신 버전을 확인합니다. 로컬 `CLAUDE.md` 파일을 체크인된 파일과 비교하여 중복을 제거하고, Claude가 코드베이스에서 파생할 수 있는 콘텐츠를 잘라내어 체크인된 [`CLAUDE.md`](/docs/en/memory) 파일을 다듬으며, 남아 있는 항상 로드되는 지침을 [스킬](/docs/en/skills) 및 필요할 때 로드되는 중첩 `CLAUDE.md` 파일로 마이그레이션합니다. 다듬기 작업은 디렉토리 레이아웃, 종속성 목록, 아키텍처 개요와 같은 섹션을 축소하고 도구 기본값과 다른 오류/이유/규칙을 보존합니다. 또한 [자동 모드](/docs/en/permissions#permission-modes)를 기본값으로 설정하고 자주 거부되는 읽기 전용 명령을 [사전 승인](/docs/en/permissions)하는 옵션을 제공합니다. 결과를 먼저 보고하고 변경하기 전에 확인을 요청합니다. 터미널에서 `claude doctor`를 실행하면 세션을 시작하지 않고 읽기 전용 설치 진단 정보가 출력됩니다. 별칭: `/checkup`. {/* min-version: 2.1.206 */}`CLAUDE.md` 다듬기 검사에는 Claude Code v2.1.206 이상이 필요합니다. {/* min-version: 2.1.205 */}v2.1.205 이전에는 `/doctor`가 읽기 전용 진단 화면을 열고 `f`를 누르면 보고서를 Claude에게 보냈습니다. |
| `/effort [level\|auto]` | 모델 [작업 투입 수준(effort level)](/docs/en/model-config#adjust-effort-level)을 설정합니다. `low`, `medium`, `high`, `xhigh`, `max` 또는 `ultracode`를 받습니다. 사용 가능한 수준은 모델에 따라 다르며 `max` 및 `ultracode`는 세션 전용입니다. `ultracode`는 `xhigh` 추론과 자동 [워크플로우](/docs/en/workflows#let-claude-decide-with-ultracode) 조율을 결합한 Claude Code 설정입니다. `auto`는 모델 기본값으로 재설정합니다. 인수 없이 실행하면 대화형 슬라이더가 열립니다 (좌/우 화살표로 수준 선택, `Enter`로 적용). 현재 응답이 끝나기를 기다리지 않고 즉시 적용됩니다. {/* min-version: 2.1.205 */}수준 인수와 함께 비대화형 모드(`-p`)에서도 사용 가능하며, 현재 세션에만 적용되고 기본값으로 저장되지 않습니다 (Claude Code v2.1.205 이상 필요). Fable 5, Opus 4.8, Opus 4.7에서는 [모델 기본 작업 투입 유지](/docs/en/model-config#adjust-effort-level)가 적용되는 동안 비대화형 `/effort`가 `Not applied`를 보고하므로 실행 시 `--effort`를 전달하세요. |
| `/exit` | CLI를 종료합니다. 연결된 [백그라운드 세션](/docs/en/agent-view#attach-to-a-session)에서는 분리되고 세션은 계속 실행됩니다. 별칭: `/quit` |
| `/export [filename]` | 현재 대화를 일반 텍스트로 내보냅니다. 파일 이름이 있으면 해당 파일에 직접 기록하고, 없으면 클립보드 복사 또는 파일 저장 다이얼로그를 엽니다. |
| `/fast [on\|off]` | [패스트 모드(fast mode)](/docs/en/fast-mode)를 켜거나 끕니다. {/* min-version: 2.1.205 */}비대화형 모드(`-p`)에서 `/fast`는 [`--settings`](/docs/en/cli-reference#cli-flags) 값에 패스트 모드가 포함된 세션에서만 작동합니다 (예: `claude -p --settings '{"fastMode": true}'`). 해당 전환은 현재 세션에만 적용되고 기본값으로 저장되지 않으며, 다른 비대화형 세션에서는 패스트 모드를 사용할 수 없다고 보고합니다. Claude Code v2.1.205 이상이 필요합니다. |
| `/feedback [report]` | Claude Code에 대한 제품 피드백을 보냅니다. [`/bug`](#all-commands)와 동일한 동의 단계 및 전송 규칙이 포함된 다이얼로그를 엽니다. |
| `/fewer-permission-prompts` | **[Skill](/docs/en/skills#bundled-skills).** 트랜스크립트에서 공통 읽기 전용 Bash 및 MCP 도구 호출을 스캔한 다음, 프로젝트 `.claude/settings.json`에 우선순위 허용 목록을 추가하여 권한 확인 프롬프트를 줄입니다. |
| `/focus` | 마지막 프롬프트, 편집 diffstats가 포함된 한 줄 도구 호출 요약, 최종 응답만 표시하는 포커스 뷰를 전환합니다. {/* min-version: 2.1.198 */}v2.1.198부터 도구 호출 요약은 해당 턴에서 시작된 하위 에이전트 수도 계산하며 완료된 백그라운드 작업 알림을 단일 카운트로 축소합니다. 이 선택은 세션 간에 유지됩니다. 설정에서 [`viewMode`](/docs/en/settings#available-settings)를 설정하여 재정의하세요. [전체 화면 렌더링](/docs/en/fullscreen)에서만 이용 가능합니다. |
| `/fork [prompt]` | {/* min-version: 2.1.212 */}현재 대화를 새 [백그라운드 세션](/docs/en/agent-view#from-inside-a-session)으로 복사하고 여기에서 작업을 계속합니다. 복사본은 지금까지 이 대화의 모든 내용으로 시작하고 [에이전트 뷰](/docs/en/agent-view)에서 자체 행으로 실행됩니다. 그 시점부터 두 세션은 독립적입니다. 프롬프트를 전달하면 복사본이 해당 작업을 즉시 시작하고, 프롬프트가 없으면 에이전트 뷰에서 첫 프롬프트를 기다립니다. 결과가 이 대화로 돌아오는 하위 에이전트에 사이드 작업을 전달하려면 `/subtask`를 사용하세요. 복사본으로 직접 전환하려면 `/branch`를 사용하세요. Claude Code v2.1.212 이상이 필요합니다. v2.1.161부터 v2.1.211까지 이 명령은 [포크된 하위 에이전트](/docs/en/sub-agents#fork-the-current-conversation)를 시작했습니다. |
| `/goal [condition\|clear]` | [목표(goal)](/docs/en/goal)를 설정합니다: Claude는 조건이 충족될 때까지 턴을 거쳐 계속 작업합니다. 인수 없이 실행하면 현재 또는 가장 최근에 달성된 목표를 표시합니다. `clear`, `stop`, `off`, `reset`, `none`, `cancel`은 활성 목표를 조기에 제거합니다. |
| `/heapdump` | 높은 메모리 사용량을 진단하기 위해 JavaScript 힙 스냅샷 및 메모리 내역을 `~/Desktop`(Desktop 폴더가 없는 Linux의 경우 홈 디렉토리)에 만듭니다. `.heapsnapshot` 파일에는 전체 대화 및 자격 증명이 포함되어 있으므로 공유하지 마세요. [문제 해결](/docs/en/troubleshooting#high-cpu-or-memory-usage)을 참조하세요. |
| `/help` | 도움말 및 사용 가능한 명령을 표시합니다. |
| `/hooks` | 도구 이벤트에 대한 [훅](/docs/en/hooks) 구성을 봅니다. |
| `/ide` | IDE 연동을 관리하고 상태를 표시합니다. |
| `/init` | `CLAUDE.md` 가이드로 프로젝트를 초기화합니다. 스킬, 훅, 개인 메모리 파일까지 둘러보는 대화형 흐름을 사용하려면 `CLAUDE_CODE_NEW_INIT=1`을 설정하세요. |
| `/insights` | 프로젝트 영역, 상호작용 패턴, 마찰 지점을 포함하여 Claude Code 세션을 분석하는 보고서를 생성합니다. |
| `/install-github-app` | 옵션 단계로 [GitHub Actions](/docs/en/github-actions) 워크플로우 및 시크릿을 설정하면서 리포지토리에 Claude GitHub 앱을 설치합니다. 리포지토리 선택 및 연동 구성을 안내합니다. |
| `/install-slack-app` | Claude Slack 앱을 설치합니다. OAuth 흐름을 완료하기 위해 브라우저를 엽니다. |
| `/keybindings` | [키보드 단축키](/docs/en/keybindings) 파일을 엽니다. |
| `/login` | Anthropic 계정에 로그인합니다. |
| `/logout` | Anthropic 계정에서 로그아웃합니다. |
| `/loop [interval] [prompt]` | **[Skill](/docs/en/skills#bundled-skills).** 세션이 열려 있는 동안 프롬프트를 반복 실행합니다. 간격을 생략하면 Claude가 반복 간 페이스를 스스로 조절합니다. 프롬프트를 생략하면 [가능한 경우](/docs/en/scheduled-tasks#run-the-built-in-maintenance-prompt) 자율 유지관리 점검이나 `.claude/loop.md`에 있는 프롬프트를 실행합니다. 예: `/loop 5m check if the deploy finished`. [일정에 따라 프롬프트 실행하기](/docs/en/scheduled-tasks)를 참조하세요. 별칭: `/proactive` |
| `/mcp [reconnect <server>\|enable\|disable [<server>\|all]]` | MCP 서버 연결 및 OAuth 인증을 관리합니다. 인수 없이 실행하여 대화형 목록을 열거나, 연결 해제된 서버 하나를 다시 연결하려면 `reconnect <server>`를 전달하거나, 다이얼로그를 열지 않고 연결 상태를 변경하려면 서버 이름 또는 `all`과 함께 `enable`/`disable`을 전달하세요. {/* min-version: 2.1.205 */}비대화형 모드(`-p`)에서도 가능합니다 (Claude Code v2.1.205 이상 필요). |
| `/memory` | `CLAUDE.md` 메모리 파일을 편집하고, [자동 메모리](/docs/en/memory#auto-memory)를 활성화/비활성화하며, 자동 메모리 항목을 봅니다. |
| `/mobile` | Claude 모바일 앱을 다운로드할 수 있는 QR 코드를 표시합니다. 별칭: `/ios`, `/android` |
| `/model [model]` | AI 모델을 전환하고 새 세션의 기본값으로 저장합니다. 지원되는 모델의 경우 좌/우 화살표를 사용하여 [작업 투입 수준을 조정](/docs/en/model-config#adjust-effort-level)하세요. 인수 없이 실행하면 선택기가 열립니다 (행에서 `s`를 눌러 현재 세션에만 적용). 다음 응답이 캐시된 컨텍스트 없이 전체 기록을 다시 읽으므로 대화에 이전 출력이 있는 경우 선택기에서 확인을 요청합니다. 승인되면 현재 응답이 끝나기를 기다리지 않고 즉시 적용됩니다. {/* min-version: 2.1.205 */}비대화형 모드(`-p`)에서도 가능합니다. |
| `/passes` | 친구들에게 Claude Code 1주일 무료 이용권을 공유합니다. 계정이 자격 대상인 경우에만 표시됩니다. |
| `/permissions` | 도구 권한에 대한 허용, 확인, 거부 규칙을 관리합니다. 범위별 규칙 보기, 규칙 추가/제거, 작업 디렉토리 관리, [최근 자동 모드 거부 내역 검토](/docs/en/auto-mode-config#review-denials)를 할 수 있는 대화형 다이얼로그를 엽니다. 별칭: `/allowed-tools` |
| `/plan [description]` | 프롬프트에서 직접 플랜 모드로 진입합니다. 선택적 설명을 전달하여 플랜 모드에 진입하고 즉시 해당 작업을 시작하세요 (예: `/plan fix the auth bug`). |
| `/plugin [subcommand]` | Claude Code [플러그인](/docs/en/plugins)을 관리합니다. 인수 없이 실행하여 플러그인 메뉴를 열거나 `list`, `install`, `enable`, `disable`과 같은 하위 명령을 전달하여 직접 작업하세요. |
| `/powerup` | 애니메이션 데모가 포함된 빠른 대화형 레슨을 통해 Claude Code 기능을 탐색합니다. |
| `/pr-comments [PR]` | {/* max-version: 2.1.90 */}v2.1.91에서 제거되었습니다. 대신 Claude에게 풀 리퀘스트 주석 보기를 직접 요청하세요. |
| `/privacy-settings` | 개인정보 보호설정을 보고 업데이트합니다. Pro 및 Max 요금제 구독자에게만 이용 가능합니다. |
| `/radio` | 브라우저에서 Claude FM lo-fi 라디오를 엽니다. 브라우저를 사용할 수 없을 때는 스트림 URL을 출력합니다. Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 또는 AWS 상의 Claude Platform에서는 사용할 수 없습니다. |
| `/recap` | 요청 시 현재 세션의 한 줄 요약을 생성합니다. 자리를 비운 후 표시되는 자동 요약은 [세션 요약](/docs/en/interactive-mode#session-recap)을 참조하세요. |
| `/release-notes` | 대화형 버전 선택기에서 변경 로그를 봅니다. 특정 버전을 선택하여 출시 노트를 보거나 모든 버전을 표시하도록 선택하세요. {/* min-version: 2.1.208 */}노트는 Claude가 보는 대화에 들어가지 않고 트랜스크립트에 나타납니다. |
| `/reload-plugins [--force]` | 재시작하지 않고 보류 중인 변경 사항을 적용하기 위해 모든 활성 [플러그인](/docs/en/plugins)을 다시 로드합니다. 다시 로드된 각 컴포넌트의 수를 보고하고 로드 오류를 지적합니다. 다시 로드로 인해 로드되는 MCP 도구가 변경되고 프롬프트 캐시가 무효화되는 경우 `--force`를 전달하지 않으면 경고하고 건너끕니다. |
| `/reload-skills` | {/* min-version: 2.1.152 */}세션 중 디스크에서 추가되거나 변경된 스킬을 재시작 없이 사용할 수 있도록 [스킬](/docs/en/skills) 및 명령 디렉토리를 다시 스캔합니다. 사용 가능한 스킬 수와 추가/제거된 스킬 수를 보고합니다. v2.1.152에서 추가되었습니다. |
| `/remote-control` | claude.ai에서 이 세션을 [Remote Control](/docs/en/remote-control)로 이용할 수 있도록 설정합니다. 별칭: `/rc` |
| `/remote-env` | [클라우드 에이전트](/docs/en/claude-code-on-the-web#configure-your-environment)의 기본 환경을 선택합니다. |
| `/rename [name]` | 현재 세션의 이름을 변경하고 프롬프트 바에 이름을 표시합니다. 이름이 없으면 대화 기록에서 자동으로 이름을 생성합니다. 비대화형 모드(`-p`)에서도 사용 가능합니다. |
| `/resume [session]` | ID 또는 이름으로 대화를 재개하거나 세션 선택기를 엽니다. v2.1.144부터 [백그라운드 세션](/docs/en/agent-view)이 `bg`로 표시되어 선택기에 나타납니다. 별칭: `/continue` |
| `/review [PR]` | {/* min-version: 2.1.202 */}번호로 GitHub 풀 리퀘스트의 빠른 단일 패스 읽기 전용 리뷰를 실행합니다. 인수 없이 실행하면 선택할 수 있는 열린 PR을 나열합니다. |
| `/rewind` | 대화 및/또는 코드를 이전 시점으로 되돌리거나 선택한 메시지부터 요약합니다. [체크포인팅](/docs/en/checkpointing)을 참조하세요. 별칭: `/checkpoint`, `/undo` |
| `/run` | **[Skill](/docs/en/skills#bundled-skills).** 테스트를 통과하는 것뿐만 아니라 변경 사항이 제대로 작동하는지 확인하기 위해 프로젝트 앱을 실행하고 구동합니다. [앱 실행 및 검증](/docs/en/skills#run-and-verify-your-app)을 참조하세요. Claude Code v2.1.145 이상이 필요합니다. |
| `/run-skill-generator` | **[Skill](/docs/en/skills#bundled-skills).** 프로젝트별 [스킬](/docs/en/skills#run-and-verify-your-app)을 작성하여 깨끗한 환경에서 프로젝트 앱을 빌드, 실행 및 구동하는 방법을 `/run` 및 `/verify`에 가르칩니다. Claude Code v2.1.145 이상이 필요합니다. |
| `/sandbox` | [샌드박스 모드](/docs/en/sandboxing)를 전환합니다. 지원되는 플랫폼에서만 이용 가능합니다. |
| `/schedule [description]` | Anthropic이 관리하는 클라우드 인프라에서 실행되는 [루틴(routines)](/docs/en/routines)을 생성, 업데이트, 나열 또는 실행합니다. 별칭: `/routines` |
| `/scroll-speed` | 대화상자가 열려 있는 동안 미리보기를 위해 스크롤할 수 있는 자(ruler)를 사용하여 대화형으로 마우스 휠 [스크롤 속도](/docs/en/fullscreen#mouse-wheel-scrolling)를 조정합니다. [전체 화면 렌더링](/docs/en/fullscreen)에서만 이용 가능합니다. |
| `/security-review` | 현재 브랜치의 보류 중인 변경 사항에 대해 보안 취약점을 분석합니다. git diff를 검토하고 인젝션, 인증 문제 및 데이터 노출과 같은 위험을 식별합니다. |
| `/setup-bedrock` | 대화형 위저드를 통해 [Amazon Bedrock](/docs/en/amazon-bedrock) 인증, 리전 및 모델 고정을 구성합니다. `CLAUDE_CODE_USE_BEDROCK=1`이 설정되었을 때만 표시됩니다. |
| `/setup-vertex` | 대화형 위저드를 통해 [Google Cloud Agent Platform](/docs/en/google-vertex-ai) 인증, 프로젝트, 리전 및 모델 고정을 구성합니다. `CLAUDE_CODE_USE_VERTEX=1`이 설정되었을 때만 표시됩니다. |
| `/simplify [target]` | {/* min-version: 2.1.154 */}**[Skill](/docs/en/skills#bundled-skills).** 변경된 코드에서 정리 기회를 리뷰하고 수정 사항을 적용합니다. 4개의 리뷰 [에이전트](/docs/en/sub-agents)가 병렬로 실행됩니다. v2.1.154부터 리뷰는 정확성 버그를 탐색하지 않으며 버그 탐색에는 `/code-review`를 사용하세요. |
| `/skills` | 사용 가능한 [스킬](/docs/en/skills)을 나열합니다. 이름을 입력하여 목록을 필터링하고, `t`를 눌러 토큰 수로 정렬하며, `Space`를 눌러 [Claude 및 `/` 메뉴에 대한 스킬 가시성을 전환](/docs/en/skills#override-skill-visibility-from-settings)한 다음 `Enter`를 눌러 저장합니다. |
| `/stats` | `/usage`에 대한 별칭입니다. Stats 탭에서 열립니다. |
| `/status` | 버전을 비롯하여 모델, 계정 및 연결 상태를 보여주는 Status 탭에서 설정 인터페이스를 엽니다. Claude가 응답하는 동안에도 작동합니다. |
| `/statusline` | Claude Code의 [상태 표시줄](/docs/en/statusline)을 구성합니다. |
| `/stickers` | Claude Code 스티커를 주문합니다. |
| `/stop` | 현재 [백그라운드 세션](/docs/en/agent-view)을 중지합니다. 백그라운드 세션에 연결되어 있는 동안에만 사용할 수 있습니다. |
| `/subtask <task>` | {/* min-version: 2.1.212 */}**[포크된 하위 에이전트](/docs/en/sub-agents#fork-the-current-conversation)**를 생성합니다. 작업이 완료되면 결과가 이 대화로 돌아옵니다. Claude Code v2.1.212 이상이 필요합니다. |
| `/tasks` | 완료된 하위 에이전트를 포함하여 현재 세션의 백그라운드 작업을 보고 관리합니다. `/bashes`로도 이용 가능합니다. |
| `/team-onboarding` | Claude Code 사용 기록에서 팀 온보딩 가이드를 생성합니다. |
| `/teleport` | [Claude Code on the web](/docs/en/claude-code-on-the-web#from-web-to-terminal) 세션을 이 터미널로 가져옵니다. 별칭: `/tp` |
| `/terminal-setup` | Shift+Enter 및 기타 단축키에 대한 터미널 키 바인딩을 구성합니다. |
| `/theme` | 색상 테마를 변경합니다. |
| `/tui [default\|fullscreen]` | 터미널 UI 렌더러를 설정하고 대화를 유지한 채 다시 실행합니다. |
| `/ultraplan <prompt>` | [ultraplan](/docs/en/ultraplan) 세션에서 계획 초안을 작성하고 브랜치에서 검토한 후 원격으로 실행하거나 터미널로 다시 보냅니다. |
| `/ultrareview [PR or branch]` | [ultrareview](/docs/en/ultrareview)로 클라우드 샌드박스에서 심층 다중 에이전트 코드 리뷰를 실행합니다. `/code-review ultra` 권장. |
| `/upgrade` | 상위 요금제로 전환하기 위해 브라우저에서 업그레이드 페이지를 엽니다. |
| `/usage` | 세션 비용, 요금제 사용량 한도 및 활동 통계를 표시합니다. 별칭: `/cost`, `/stats` |
| `/usage-credits` | 한도에 도달했을 때 사용 크레딧을 구성하거나 관리자에게 요청합니다. |
| `/verify` | **[Skill](/docs/en/skills#bundled-skills).** 프로젝트 앱을 빌드, 실행 및 결과를 관찰하여 코드 변경이 제대로 작동하는지 확인합니다. |
| `/vim` | {/* max-version: 2.1.91 */}v2.1.92에서 제거되었습니다. `/config` → Editor mode를 사용하세요. |
| `/voice [hold\|tap\|off]` | [음성 받아쓰기](/docs/en/voice-dictation)를 전환합니다. |
| `/web-setup` | 로컬 `gh` CLI 자격 증명을 사용하여 GitHub 계정을 [Claude Code on the web](/docs/en/web-quickstart#connect-from-your-terminal)에 연결합니다. |
| `/workflows` | 실행 중이거나 완료된 워크플로우를 모니터링하고 일시 중지, 재개 또는 저장할 수 있도록 [워크플로우](/docs/en/workflows#watch-the-run) 진행 상태 뷰를 엽니다. |

## MCP 프롬프트

MCP 서버는 명령으로 표시되는 프롬프트를 표출할 수 있습니다. 이는 `/mcp__<server>__<prompt>` 형식을 사용하며 연결된 서버에서 동적으로 탐색됩니다. 세부 정보는 [MCP 프롬프트](/docs/en/mcp#use-mcp-prompts-as-commands)를 참조하세요.

## 참고 항목

* [스킬](/docs/en/skills): 나만의 명령 만들기
* [대화형 모드](/docs/en/interactive-mode): 키보드 단축키, Vim 모드 및 명령 기록
* [CLI 레퍼런스](/docs/en/cli-reference): 시작 시 플래그
