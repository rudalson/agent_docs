> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Claude Code 세션의 에이전트 팀 조율 (Orchestrate teams of Claude Code sessions)

> 공유 작업, 에이전트 간 메시징, 중앙 관리 기능을 통해 팀으로서 함께 실행되는 여러 Claude Code 인스턴스를 조율합니다.

<Warning>
  에이전트 팀은 실험적 기능이며 기본적으로 비활성화되어 있습니다. [settings.json](/docs/en/settings) 또는 환경 변수에서 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`을 설정하여 활성화하세요. 이 변수가 없으면 세션 시작 시 팀이 구성되지 않고, 팀 디렉터리가 기록되지 않으며, Claude가 팀 동료를 생성하거나 제안하지 않습니다. 에이전트 팀에는 세션 재개, 작업 조율 및 종료 동작에 대한 [알려진 제한 사항](#limitations)이 있습니다.
</Warning>

에이전트 팀을 사용하면 함께 실행되는 여러 Claude Code 인스턴스를 조율할 수 있습니다. 하나의 세션이 팀 리더 역할을 하여 작업을 조율하고, 과제를 할당하며, 결과를 종합합니다. 팀 동료들은 독립적으로 작동하며 각자 고유한 컨텍스트 창 내에서 서로 직접 통신합니다.

단일 세션 내에서 실행되고 메인 에이전트에만 보고할 수 있는 [서브에이전트](/docs/en/sub-agents)와 달리, 리더를 거치지 않고 개별 팀 동료와 직접 상호작용할 수도 있습니다.

<Note>
  이 페이지는 v2.1.178 시점의 에이전트 팀을 설명합니다. `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`가 설정되면 팀 동료를 생성을 위해 설정 단계가 더 이상 필요하지 않으며 세션 종료 시 정리가 자동으로 수행됩니다. v2.1.178 이전에는 Claude에게 팀을 먼저 생성하고 이름을 지정하도록 요청했으며, Claude는 `TeamCreate` 및 `TeamDelete` 도구를 사용하여 이를 설정하고 제거했습니다. 두 도구는 더 이상 존재하지 않습니다. Agent 도구의 `team_name` 입력은 허용되지만 무시되며, `TaskCreated`, `TaskCompleted`, `TeammateIdle` [훅 페이로드](/docs/en/hooks#taskcreated)의 `team_name` 필드는 세션에서 파생된 이름을 전달하며 사용이 권장되지 않습니다.
</Note>

## 에이전트 팀을 사용할 시기

에이전트 팀은 병렬 탐색이 실질적인 가치를 더하는 작업에 가장 효과적입니다. 전체 시나리오는 [사용 사례 예시](#use-case-examples)를 참조하세요. 가장 권장되는 사용 사례는 다음과 같습니다:

* **연구 및 검토**: 여러 팀 동료가 문제의 다른 측면을 동시에 조사한 후 발견된 사항을 서로 공유하고 이의를 제기할 수 있음
* **새 모듈 또는 기능**: 팀 동료들이 서로를 방해하지 않고 각각 별도의 부분을 소유할 수 있음
* **경쟁 가설을 통한 디버깅**: 팀 동료들이 다른 이론을 병렬로 테스트하고 답변으로 더 빠르게 수렴함
* **계층 간 조율**: 각각 다른 팀 동료가 소유한 프론트엔드, 백엔드 및 테스트 전체에 걸친 변경 사항

에이전트 팀은 조율 오버헤드를 추가하며 단일 세션보다 훨씬 더 많은 토큰을 사용합니다. 팀 동료들이 독립적으로 작동할 수 있을 때 가장 잘 작동합니다. 순차적 작업, 동일 파일 편집, 또는 종속성이 많은 작업의 경우 단일 세션이나 [서브에이전트](/docs/en/sub-agents)가 더 효과적입니다.

### 서브에이전트와의 비교

에이전트 팀과 [서브에이전트](/docs/en/sub-agents) 모두 작업을 병렬화할 수 있게 해주지만 작동 방식이 다릅니다. 작업자가 서로 통신해야 하는지 여부에 따라 선택하세요:

<Frame caption="서브에이전트는 결과를 메인 에이전트에만 보고하며 서로 통신하지 않습니다. 에이전트 팀에서는 팀 동료들이 작업 목록을 공유하고, 작업을 가져가며, 서로 직접 통신합니다.">
  <img src="https://mintcdn.com/claude-code/nsvRFSDNfpSU5nT7/images/subagents-vs-agent-teams-light.png?fit=max&auto=format&n=nsvRFSDNfpSU5nT7&q=85&s=2f8db9b4f3705dd3ab931fbe2d96e42a" className="dark:hidden" alt="Diagram comparing subagent and agent team architectures. Subagents are spawned by the main agent, do work, and report results back. Agent teams coordinate through a shared task list, with teammates communicating directly with each other." width="4245" height="1615" data-path="images/subagents-vs-agent-teams-light.png" />

  <img src="https://mintcdn.com/claude-code/nsvRFSDNfpSU5nT7/images/subagents-vs-agent-teams-dark.png?fit=max&auto=format&n=nsvRFSDNfpSU5nT7&q=85&s=d573a037540f2ada6a9ae7d8285b46fd" className="hidden dark:block" alt="Diagram comparing subagent and agent team architectures. Subagents are spawned by the main agent, do work, and report results back. Agent teams coordinate through a shared task list, with teammates communicating directly with each other." width="4245" height="1615" data-path="images/subagents-vs-agent-teams-dark.png" />
</Frame>

|                   | 서브에이전트 (Subagents)                         | 에이전트 팀 (Agent teams)                           |
| :---------------- | :----------------------------------------------- | :-------------------------------------------------- |
| **컨텍스트**       | 고유 컨텍스트 창; 결과가 호출자에게 반환됨       | 고유 컨텍스트 창; 완전히 독립적                     |
| **통신**           | 결과를 메인 에이전트에만 보고함                  | 팀 동료들이 서로 직접 메시지를 주고받음              |
| **조율**           | 메인 에이전트가 모든 작업을 관리함                | 자체 조율 기능을 갖춘 공유 작업 목록                |
| **최적 사용 사례** | 결과만 중요한 집중적인 작업                       | 토론 및 협업이 필요한 복잡한 작업                   |
| **토큰 비용**      | 낮음: 결과가 메인 컨텍스트로 요약되어 반환됨      | 높음: 각 팀 동료가 별도의 Claude 인스턴스임         |

결과를 보고하는 빠르고 집중적인 작업자가 필요한 경우 서브에이전트를 사용하세요. 팀 동료들이 발견한 사항을 공유하고, 서로 이의를 제기하며, 스스로 조율해야 하는 경우 에이전트 팀을 사용하세요.

## 에이전트 팀 활성화

에이전트 팀은 기본적으로 비활성화되어 있습니다. 쉘 환경이나 [settings.json](/docs/en/settings)을 통해 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` 환경 변수를 `1`로 설정하여 활성화하세요:

```json settings.json theme={null}
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## 첫 번째 에이전트 팀 시작

에이전트 팀을 활성화한 후 자연어로 작업과 원하는 팀 동료를 설명하세요. Claude가 이들을 생성하고 프롬프트를 바탕으로 작업을 조율합니다.

세 가지 역할이 독립적이며 서로 기다리지 않고 문제를 탐색할 수 있으므로 다음 예시가 잘 작동합니다:

```text theme={null}
I'm designing a CLI tool that helps developers track TODO comments across
their codebase. Spawn three teammates to explore this from different angles:
one on UX, one on technical architecture, one playing devil's advocate.
```

거기서부터 Claude는 [공유 작업 목록](/docs/en/interactive-mode#task-list)을 채우고, 각 관점별로 팀 동료를 생성하며, 이들이 문제를 탐색하도록 하고, 완료되면 결과를 종합합니다.

Claude는 팀을 만드는 대신 [서브에이전트](/docs/en/sub-agents)를 사용할 수도 있습니다. 서브에이전트는 팀 동료와 동일한 에이전트 패널에 나타나므로 패널만으로는 팀 형성을 확인할 수 없습니다. Claude가 서브에이전트를 생성한 경우 다시 질문하고 에이전트 팀을 명시적으로 요청하세요.

리더의 터미널 프롬프트 입력 아래의 에이전트 패널에 팀 동료가 나열됩니다. 패널에서:

* **위쪽 및 아래쪽 화살표**: 팀 동료 선택
* **Enter**: 선택한 팀 동료의 트랜스크립트를 열고 직접 메시지 전송
* **Escape**: 선택한 팀 동료의 현재 차례 중단

v2.1.199부터 팀 동료나 서브에이전트가 계속 작업하는 동안 유휴 상태의 팀 동료 행이 패널에 남아 있으므로 해당 행을 선택하여 트랜스크립트를 검토하거나 더 많은 작업을 전송할 수 있습니다. 패널의 모든 에이전트가 유휴 상태가 되면 30초 후에 유휴 행이 숨겨지고 팀 동료의 다음 차례에 다시 나타납니다. 숨겨져 있는 동안에도 팀 동료는 계속 실행 상태를 유지하며 주소 지정이 가능합니다. v2.1.181부터 v2.1.198까지는 다른 팀 동료가 여전히 작업 중이더라도 자체 차례가 끝난 후 30초 후에 유휴 행이 숨겨졌습니다. v2.1.181 이전 버전에서는 유휴 행이 숨겨지지 않습니다.

3명 이상의 팀 동료가 동시에 유휴 상태일 때 처음 3개를 넘어서는 행은 5개가 유휴 상태일 때 `2 idle agents`와 같이 접힌 팀 동료의 수를 집계하는 단일 행으로 접힙니다. 이를 선택하고 Enter를 눌러 접힌 행을 펼치거나 Esc를 눌러 다시 접으세요. 작업 중인 팀 동료, 실패한 팀 동료, 그리고 현재 보고 있는 팀 동료는 항상 자체 행을 유지합니다.

각 팀 동료를 별도의 분할 창에 표시하려면 [표시 모드 선택](#choose-a-display-mode)을 참조하세요.

## 에이전트 팀 제어

자연어로 리더에게 원하는 바를 전달하세요. 리더는 지침에 따라 팀 조율, 작업 할당 및 위임을 처리합니다.

### 표시 모드 선택

에이전트 팀은 두 가지 표시 모드를 지원합니다:

* **In-process**: 모든 팀 동료가 메인 터미널 내부에서 실행됩니다. 에이전트 패널에서 위쪽 및 아래쪽 화살표 키를 사용하여 팀 동료를 선택한 다음 Enter를 눌러 확인하고 입력하여 직접 메시지를 보냅니다. 모든 터미널에서 작동하며 추가 설정이 필요하지 않습니다.
* **Split panes**: 각 팀 동료가 자체 창(pane)을 갖습니다. 모든 사람의 출력을 한 번에 확인하고 창을 클릭하여 직접 상호작용할 수 있습니다. tmux 또는 iTerm2가 필요합니다.

<Note>
  `tmux`는 특정 운영 체제에서 알려진 제한 사항이 있으며 전통적으로 macOS에서 가장 잘 작동합니다. iTerm2에서 `tmux -CC`를 사용하는 것이 `tmux`로의 권장 진입점입니다.
</Note>

기본값은 `"in-process"`입니다. v2.1.179 이전의 기본값은 `"auto"`였으므로 이전에 분할 창을 열었던 업그레이드된 세션은 모드를 명시적으로 설정하지 않는 한 하나의 터미널에 유지됩니다. 이미 tmux 세션 내부에서 실행 중이거나 terminal이 `it2` CLI가 설치된 iTerm2일 때 분할 창을 활성화하고 그렇지 않은 경우 in-process로 폴백하려면 `"auto"`로 설정하세요. `"tmux"` 설정은 분할 창 모드를 활성화하고 터미널에 따라 tmux 또는 iTerm2를 사용할지 여부를 자동 감지합니다.

v2.1.186부터 iTerm2 기본 분할 창을 명시적으로 사용하려면 `"iterm2"`로 설정하세요. 이 모드는 [`it2` CLI](https://github.com/mkusaka/it2)가 필요하며 `it2`가 없는 경우 설치 명령어와 함께 에러를 표시합니다. `it2` 설치를 제공하거나 tmux로 전환하는 설정 프롬프트는 터미널이 iTerm2이고 tmux를 폴백으로 사용할 수 있을 때 `"auto"` 또는 `"tmux"` 아래에 나타납니다.

기본값을 오버라이드하려면 `~/.claude/settings.json`에서 [`teammateMode`](/docs/en/settings#available-settings)를 설정하세요:

```json theme={null}
{
  "teammateMode": "auto"
}
```

단일 세션에 대한 모드를 설정하려면 플래그로 전달하세요:

```bash theme={null}
claude --teammate-mode auto
```

`--teammate-mode` 플래그는 실험적 기능이며 `claude --help`에 나타나지 않습니다.

분할 창 모드에는 [tmux](https://github.com/tmux/tmux/wiki) 또는 [`it2` CLI](https://github.com/mkusaka/it2)가 포함된 iTerm2가 필요합니다. 수동으로 설치하려면:

* **tmux**: 시스템의 패키지 관리자를 통해 설치하세요. 플랫폼별 지침은 [tmux wiki](https://github.com/tmux/tmux/wiki/Installing)를 참조하세요.
* **iTerm2**: [`it2` CLI](https://github.com/mkusaka/it2)를 설치한 다음 **iTerm2 → Settings → General → Magic → Enable Python API**에서 Python API를 활성화하세요.

### 팀 동료 및 모델 지정

Claude는 작업에 따라 생성할 팀 동료 수를 결정하거나, 원하는 바를 정확히 지정할 수 있습니다:

```text theme={null}
Spawn 4 teammates to refactor these modules in parallel. Use Sonnet for
each teammate.
```

팀 동료는 기본적으로 리더의 `/model` 선택을 상호 상속하지 않습니다. 프롬프트에서 지정하지 않은 경우 사용되는 모델을 변경하려면 `/config`에서 **Default teammate model**을 설정하세요. 팀 동료가 리더의 현재 모델을 따르도록 하려면 **Default (leader's model)**을 선택하세요.

팀 동료는 리더의 [노력 수준](/docs/en/model-config#adjust-effort-level)을 상속합니다. 분할 창 모드에서는 v2.1.186부터 적용되며 이전 버전은 리더 세션의 노력을 분할 창 팀 동료에게 전달하지 않았습니다.

### 팀 동료의 계획 승인 요구

복잡하거나 위험한 작업의 경우 구현하기 전에 팀 동료가 계획하도록 요구할 수 있습니다. 팀 동료는 리더가 접근 방식을 승인할 때까지 읽기 전용 계획 모드에서 작업합니다:

```text theme={null}
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
```

팀 동료가 계획을 마치면 리더에게 계획 승인 요청을 보냅니다. 리더는 계획을 검토하고 이를 승인하거나 피드백과 함께 거부합니다. 거부되면 팀 동료는 계획 모드에 남아 피드백을 바탕으로 수정하고 다시 제출합니다. 승인되면 팀 동료는 계획 모드를 종료하고 구현을 시작합니다.

리더는 자율적으로 승인 결정을 내립니다. 리더의 판단에 영향을 주려면 프롬프트에서 "테스트 커버리지가 포함된 계획만 승인" 또는 "데이터베이스 스키마를 수정하는 계획은 거부"와 같은 기준을 제공하세요.

### 팀 동료와 직접 대화

각 팀 동료는 완전하고 독립적인 Claude Code 세션입니다. 팀 동료에게 직접 메시지를 보내 추가 지침을 제공하거나, 후속 질문을 하거나, 접근 방식을 전환할 수 있습니다.

* **In-process 모드**: 에이전트 패널에서 위쪽 및 아래쪽 화살표 키를 사용하여 팀 동료를 선택한 다음 Enter를 눌러 해당 세션을 보고 입력하여 메시지를 전송합니다. 선택한 팀 동료에서 `x`를 눌러 중지합니다. Ctrl+T를 눌러 작업 목록을 토글합니다.
* **Split-pane 모드**: 팀 동료의 창을 클릭하여 해당 세션과 직접 상호작용합니다. 각 팀 동료는 자체 터미널의 전체 뷰를 갖습니다.

in-process 팀 동료를 보고 있는 동안 일반 텍스트와 [스킬](/docs/en/skills)은 해당 팀 동료에게 전송되지만, 내장 명령어는 여전히 리더 세션에서 실행됩니다.

팀 동료의 모델과 빠른 모드는 생성 시 고정되므로 `/model` 및 `/fast`는 리더의 설정만 변경합니다. v2.1.199부터 팀 동료를 보고 있는 동안 두 명령어 중 하나를 입력하면 변경 사항이 리더에 적용된다는 알림이 표시되며, 이전 버전에서는 표식 없이 리더에 적용되었습니다. 팀 동료는 리더의 [노력 수준](/docs/en/model-config#adjust-effort-level)을 따르므로 `/effort`는 보고 있는 팀 동료의 이후 차례에 여전히 적용됩니다.

### 작업 할당 및 가져오기

공유 작업 목록은 팀 전체의 작업을 조율합니다. 리더가 작업을 생성하고 팀 동료가 이를 수행합니다. 작업에는 보류(pending), 진행 중(in progress), 완료(completed)의 세 가지 상태가 있습니다. 작업은 다른 작업에 의존할 수도 있습니다. 해결되지 않은 종속성이 있는 보류 중인 작업은 해당 종속성이 완료될 때까지 가져올 수 없습니다.

리더가 작업을 명시적으로 할당하거나 팀 동료가 직접 가져올 수 있습니다:

* **리더가 할당**: 리더에게 어떤 작업을 어떤 팀 동료에게 줄지 전달
* **자체 가져오기**: 작업을 마친 후 팀 동료가 할당되지 않고 차단되지 않은 다음 작업을 스스로 선택

작업 가져오기는 파일 잠금을 사용하여 여러 팀 동료가 동일한 작업을 동시에 가져오려고 할 때 경합 조건을 방지합니다.

### 팀 동료 종료

팀 동료의 세션을 정상적으로 종료하려면 이름을 참조하세요. 예를 들어 researcher라는 이름의 팀 동료가 있는 경우:

```text theme={null}
Ask the researcher teammate to shut down
```

리더가 종료 요청을 보냅니다. 팀 동료는 정상적으로 종료되어 승인하거나 설명과 함께 거부할 수 있습니다.

팀의 공유 디렉터리는 세션이 끝날 때 자동으로 정리되므로 별도의 정리 단계가 없습니다. 어떤 디렉터리가 제거되고 어떤 디렉터리가 재개된 세션을 위해 유지되는지는 [아키텍처](#architecture)를 참조하세요.

### 훅으로 품질 게이트 강제 적용

팀 동료가 작업을 마치거나 작업이 생성 또는 완료될 때 [훅](/docs/en/hooks)을 사용하여 규칙을 강제 적용하세요:

* [`TeammateIdle`](/docs/en/hooks#teammateidle): 팀 동료가 유휴 상태가 되려고 할 때 실행됨. 피드백을 보내고 팀 동료가 계속 작업하도록 하려면 종료 코드 2로 종료하세요.
* [`TaskCreated`](/docs/en/hooks#taskcreated): 작업이 생성될 때 실행됨. 생성을 방지하고 피드백을 보내려면 종료 코드 2로 종료하세요.
* [`TaskCompleted`](/docs/en/hooks#taskcompleted): 작업이 완료로 표시될 때 실행됨. 완료를 방지하고 피드백을 보내려면 종료 코드 2로 종료하세요.

## 에이전트 팀의 작동 방식

이 섹션에서는 에이전트 팀 뒤의 아키텍처와 메커니즘을 다룹니다. 에이전트 팀을 바로 시작하려면 위 [에이전트 팀 제어](#control-your-agent-team)를 참조하세요.

### Claude가 에이전트 팀을 시작하는 방식

첫 번째 팀 동료가 생성될 때 메인 세션이 리더 역할을 하면서 에이전트 팀이 형성됩니다. 팀 동료가 생성되는 방법에는 두 가지가 있습니다:

* **팀 동료 요청**: 병렬 작업의 이점이 있는 작업을 Claude에게 제공하고 팀 동료를 명시적으로 요청합니다. Claude가 지침을 바탕으로 팀 동료를 생성합니다.
* **Claude의 팀 동료 제안**: Claude가 작업에 병렬 작업이 유용하다고 판단하면 팀 동료 생성을 제안할 수 있습니다. 계속 진행하기전에 승인합니다.

두 경우 모두 사용자가 직접 제어권을 갖습니다. Claude는 승인 없이 팀 동료를 생성하지 않습니다.

### 아키텍처 (Architecture)

에이전트 팀은 다음으로 구성됩니다:

| 구성 요소     | 역할                                                                    |
| :------------ | :---------------------------------------------------------------------- |
| **팀 리더**   | 팀 동료를 생성하고 작업을 조율하는 메인 Claude Code 세션                |
| **팀 동료**   | 할당된 작업을 처리하는 별도의 Claude Code 인스턴스                      |
| **작업 목록** | 팀 동료들이 가져가고 완료하는 공유 작업 항목 목록                       |
| **사물함**     | 에이전트 간 통신을 위한 메시징 시스템                                   |

표시 구성 옵션은 [표시 모드 선택](#choose-a-display-mode)을 참조하세요. 팀 동료 메시지는 리더에게 자동으로 도착합니다.

각 에이전트의 사물함(mailbox)은 `~/.claude/teams/{team-name}/inboxes/{agent-name}.json`에 있는 JSON 파일입니다. Claude Code는 사물함 파일을 읽을 때 모든 항목을 검증합니다. 메시지 형식과 일치하지 않는 항목은 에어로 보고되고 파일에서 제거되며, 유효한 메시지는 정상 전달됩니다. v2.1.207 이전에는 하나의 잘못된 형식의 사물함 항목으로 인해 매초마다 오류가 반복되고 파일은 수동으로 삭제할 때까지 사물함 전달이 차단되었습니다.

시스템은 작업 종속성을 자동으로 관리합니다. 팀 동료가 다른 작업이 의존하는 작업을 완료하면 수동 개입 없이 차단된 작업의 차단이 해제됩니다.

팀과 작업은 세션에서 파생된 이름 아래 로컬로 저장됩니다. 이름은 `session-` 뒤에 세션 ID의 처음 8자입니다:

* **팀 구성**: `~/.claude/teams/{team-name}/config.json`
* **작업 목록**: `~/.claude/tasks/{team-name}/`

Claude Code는 세션 시작 시 이 두 가지를 모두 자동으로 생성하고 팀 동료가 합류하거나 유휴 상태가 되거나 떠날 때 업데이트합니다. 팀 구성 디렉터리는 세션이 끝날 때 제거됩니다. 작업 목록 디렉터리는 로컬에 지속되며 업로드되지 않으므로 재개된 세션이 작업을 유지합니다. 보유 기간은 세션 트랜스크립트에 대해 이미 제어하는 것과 동일한 [`cleanupPeriodDays`](/docs/en/settings#available-settings)에 의해 관리됩니다.

팀 구성에는 세션 ID 및 tmux 창 ID와 같은 런타임 상태가 포함되므로 수동으로 편집하거나 미리 작성하지 마세요. 변경 사항은 다음 상태 업데이트 시 덮어써집니다.

재사용 가능한 팀 동료 역할을 정의하려면 대신 [팀 동료에 서브에이전트 정의 사용](#use-subagent-definitions-for-teammates)을 사용하세요.

팀 구성에는 각 멤버의 이름과 에이전트 ID가 포함된 `members` 배열이 포함됩니다. 리더 항목은 항상 에이전트 타입 `team-lead`를 전달하며, 팀 동료 항목은 팀 동료가 [서브에이전트 정의](#use-subagent-definitions-for-teammates)에서 생성된 경우에만 에이전트 타입을 포함합니다. 팀 동료는 다른 팀 멤버를 발견하기 위해 이 파일을 읽을 수 있습니다.

팀 구성의 프로젝트 수준 동등물은 없습니다. 프로젝트 디렉터리의 `.claude/teams/teams.json`과 같은 파일은 구성으로 인식되지 않으며, Claude는 이를 일반 파일로 처리합니다.

### 팀 동료에 서브에이전트 정의 사용

팀 동료를 생성할 때 프로젝트, 사용자, 플러그인 또는 CLI 정의와 같은 모든 [서브에이전트 스코프](/docs/en/sub-agents#choose-the-subagent-scope)에서 [서브에이전트](/docs/en/sub-agents) 타입을 참조할 수 있습니다. 이를 통해 security-reviewer 또는 test-runner와 같은 역할을 한 번 정의하고 위임된 서브에이전트 및 에이전트 팀 동료 모두로 재사용할 수 있습니다.

서브에이전트 정의를 사용하려면 팀 동료를 생성하도록 Claude에게 요청할 때 이름을 언급하세요:

```text theme={null}
Spawn a teammate using the security-reviewer agent type to audit the auth module.
```

팀 동료는 해당 정의의 `tools` 허용 목록과 `model`을 존중하며, 정의의 본문은 팀 동료의 시스템 프롬프트를 교체하는 대신 추가 지침으로 추가됩니다. `SendMessage` 및 작업 관리 도구와 같은 팀 조율 도구는 `tools`가 다른 도구를 제한하더라도 팀 동료가 항상 사용할 수 있습니다.

<Note>
  서브에이전트 정의의 `skills` 및 `mcpServers` 프론트매터 필드는 해당 정의가 팀 동료로 실행될 때 적용되지 않습니다. 팀 동료는 일반 세션과 마찬가지로 프로젝트 및 사용자 설정에서 스킬과 MCP 서버를 로드합니다.
</Note>

### 권한 (Permissions)

팀 동료는 리더의 권한 설정으로 시작합니다. 리더가 `--dangerously-skip-permissions`로 실행되면 모든 팀 동료도 동일하게 실행됩니다. 생성 후에는 개별 팀 동료 모드를 변경할 수 있지만, 생성 시점에 팀 동료별 모드를 설정할 수는 없습니다.

한 에이전트가 `SendMessage`를 통해 다른 에이전트에게 메시지를 보낼 때, 수신 에이전트는 해당 메시지가 사용자가 아닌 다른 Claude 세션에서 왔다는 것을 알게 됩니다. 팀 동료는 사용자 대신 권한 프롬프트를 승인하거나 동의를 제공할 수 없으며, 작업을 거부당한 팀 동료가 해당 작업을 검사를 우회하기 위해 다른 팀 동료에게 전달할 수 없습니다. [자동 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)에서 분류기는 다른 에이전트로부터 전달된 승인 주장을 사용자의 확인이 아닌 신뢰할 수 없는 입력으로 처리합니다.

팀 동료 권한 프롬프트는 리더 세션에 나타나므로 거기서 직접 승인하세요. [계획 승인](#require-plan-approval-for-teammates)은 설계된 예외입니다. 리더 세션은 사용자에게 별도의 프롬프트 없이 팀 동료 계획 승인을 부여합니다.

### 컨텍스트 및 통신

각 팀 동료는 자체 컨텍스트 창을 갖습니다. 생성 시 팀 동료는 일반 세션과 동일한 프로젝트 컨텍스트(CLAUDE.md, MCP 서버, 스킬)를 로드합니다. 또한 리더로부터 생성 프롬프트를 받습니다. 리더의 대화 기록은 이월되지 않습니다.

**팀 동료가 정보를 공유하는 방식:**

* **자동 메시지 전달**: 팀 동료가 메시지를 보낼 때 수신자에게 자동으로 전달됩니다. 리더가 업데이트를 폴링할 필요가 없습니다.
* **유휴 알림**: 팀 동료가 완료하고 중지되면 리더에게 자동으로 알립니다. v2.1.198부터 API 오류로 차례가 끝난 팀 동료는 리더에게 실패했음을 알리고 정상적으로 완료된 것으로 나타나는 대신 오류 텍스트를 포함합니다.
* **공유 작업 목록**: 모든 에이전트가 작업 상태를 보고 사용 가능한 작업을 가져올 수 있습니다.
* **팀 동료 메시징**: 이름으로 특정 팀 동료 한 명에게 메시지를 보냅니다. 모든 사람에게 전달하려면 수신자당 하나의 메시지를 전송하세요.

리더는 팀 동료를 생성할 때 모든 팀 동료에게 이름을 할당하며, 모든 팀 동료는 해당 이름으로 다른 팀 동료에게 메시지를 보낼 수 있습니다. 나중에 프롬프트에서 참조할 수 있는 예측 가능한 이름을 얻으려면 생성 지침에서 각 팀 동료를 부를 이름을 리더에게 전하세요.

### 토큰 사용량

에이전트 팀은 단일 세션보다 훨씬 더 많은 토큰을 사용합니다. 각 팀 동료는 고유한 컨텍스트 창을 가지며, 토큰 사용량은 활성 팀 동료 수에 따라 비례합니다. 연구, 검토 및 새 기능 작업의 경우 추가 토큰의 가치가 있는 경우가 많습니다. 일상적인 작업의 경우 단일 세션이 더 비용 효율적입니다. 사용량 지침은 [에이전트 팀 토큰 비용](/docs/en/costs#agent-team-token-costs)을 참조하세요.

## 사용 사례 예시

이 예시들은 에이전트 팀이 병렬 탐색이 가치를 더하는 작업을 처리하는 방법을 보여줍니다.

### 병렬 코드 리뷰 실행

단일 리뷰어는 한 번에 한 가지 유형의 문제에 끌리는 경향이 있습니다. 리뷰 기준을 독립적인 도메인으로 분할하면 보안, 성능 및 테스트 커버리지 모두 동시에 철저하게 주의를 기울일 수 있습니다. 프롬프트는 겹치지 않도록 각 팀 동료에게 서로 다른 렌즈를 할당합니다:

```text theme={null}
Spawn three teammates to review PR #142:
- One focused on security implications
- One checking performance impact
- One validating test coverage
Have them each review and report findings.
```

각 리뷰어는 동일한 PR에서 작업하지만 다른 필터를 적용합니다. 리더는 세 플레이어가 모두 마친 후 발견된 사항을 종합합니다.

### 경쟁 가설을 통한 조사

근본 원인이 불분명할 때 단일 에이전트는 하나의 그럴듯한 설명을 찾고 조사를 중단하는 경향이 있습니다. 프롬프트는 팀 동료를 명시적으로 대립하게 만듦으로써 이에 맞섭니다: 각자의 역할은 자신의 이론을 조사하는 것뿐만 아니라 다른 사람의 이론에 이의를 제기하는 것입니다.

```text theme={null}
Users report the app exits after one message instead of staying connected.
Spawn 5 agent teammates to investigate different hypotheses. Have them talk to
each other to try to disprove each other's theories, like a scientific
debate. Update the findings doc with whatever consensus emerges.
```

여기서 토론 구조가 핵심 메커니즘입니다. 순차적 조사는 앵커링(anchoring) 현상을 겪습니다: 하나의 이론이 탐색되고 나면 이후의 조사는 그쪽으로 편향됩니다.

서로의 이론을 적극적으로 반박하려는 여러 독립적인 조사자가 있으면 살아남는 이론이 실제 근본 원인일 가능성이 훨씬 높아집니다.

## 모범 사례

### 팀 동료에게 충분한 컨텍스트 제공

팀 동료는 CLAUDE.md, MCP 서버 및 스킬을 포함한 프로젝트 컨텍스트를 자동으로 로드하지만 리더의 대화 기록은 상속하지 않습니다. 세부 정보는 [컨텍스트 및 통신](#context-and-communication)을 참조하세요. 생성 프롬프트에 작업별 세부 정보를 포함하세요:

```text theme={null}
Spawn a security reviewer teammate with the prompt: "Review the authentication module
at src/auth/ for security vulnerabilities. Focus on token handling, session
management, and input validation. The app uses JWT tokens stored in
httpOnly cookies. Report any issues with severity ratings."
```

### 적절한 팀 크기 선택

팀 동료 수에 대한 하드 리밋은 없지만 실질적인 제약이 적용됩니다:

* **토큰 비용 선형 증가**: 각 팀 동료는 자체 컨텍스트 창을 가지며 독립적으로 토큰을 소비합니다. 세부 정보는 [에이전트 팀 토큰 비용](/docs/en/costs#agent-team-token-costs)을 참조하세요.
* **조율 오버헤드 증가**: 팀 동료가 많을수록 통신, 작업 조율 및 충돌 가능성이 증가합니다.
* **수익 감소**: 특정 시점을 넘어서면 팀 동료가 추가되어도 작업 속도가 비례하여 빨라지지 않습니다.

대부분의 워크플로우에서는 3~5명의 팀 동료로 시작하세요. 이렇게 하면 과도하지 않은 조율과 병렬 작업의 균형이 유지됩니다. 이 가이드의 예시에서는 3~5명의 팀 동료를 사용하는데, 그 범위가 다양한 작업 유형에 걸쳐 잘 작동하기 때문입니다.

팀 동료당 5~6개의 [작업](/docs/en/agent-teams#architecture)을 보유하면 지나친 컨텍스트 스위칭 없이 모든 사람이 생산성을 유지할 수 있습니다. 15개의 독립적인 작업이 있는 경우 3명의 팀 동료가 좋은 출발점입니다.

동시에 일하는 팀 동료의 이점이 진정으로 있을 때만 규모를 확장하세요. 집중적인 3명의 팀 동료가 분산된 5명보다 뛰어난 경우가 많습니다.

### 적절한 크기의 작업 구성

* **너무 작음**: 조율 오버헤드가 이점보다 큼
* **너무 큼**: 중간 점검 없이 팀 동료가 너무 오래 작업하여 낭비되는 노력이 커짐
* **적절함**: 함수, 테스트 파일 또는 리뷰와 같이 명확한 결과물을 생성하는 자급자족형 단위

<Tip>
  리더는 작업을 나누고 팀 동료에게 자동으로 할당합니다. 작업이 충분히 생성되지 않는 경우 더 작은 조각으로 나누도록 요청하세요. 팀 동료당 5~6개의 작업을 보유하면 모든 사람이 생산성을 유지하고 누군가 막혔을 때 리더가 작업을 다시 할당할 수 있습니다.
</Tip>

### 팀 동료가 마칠 때까지 대기

때때로 리더는 팀 동료를 기다리지 않고 스스로 작업을 구현하기 시작합니다. 이러한 현상이 관찰되면 다음과 같이 지시하세요:

```text theme={null}
Wait for your teammates to complete their tasks before proceeding
```

### 연구 및 검토부터 시작

에이전트 팀이 처음이라면 경계가 명확하고 코드를 작성할 필요가 없는 작업(PR 리뷰, 라이브러리 조사, 버그 조사)부터 시작하세요. 이러한 작업은 병렬 구현 시 수반되는 조율 문제 없이 병렬 탐색의 가치를 보여줍니다.

### 파일 충돌 방지

두 팀 동료가 동일한 파일을 편집하면 덮어쓰기가 발생합니다. 각 팀 동료가 서로 다른 파일 세트를 소유하도록 작업을 나누세요.

### 모니터링 및 방향 조율

팀 동료의 진행 상황을 점검하고, 작동하지 않는 접근 방식을 변경하며, 발견된 사항이 나오는 대로 종합하세요. 팀이 너무 오래 방치되도록 두면 노력이 낭비될 위험이 커집니다.

## 문제 해결

### 팀 동료가 나타나지 않음

Claude에게 팀 동료 생성을 요청한 후에도 팀 동료가 나타나지 않는 경우:

* in-process 모드에서는 프롬프트 입력 아래의 에이전트 패널에 팀 동료가 나타납니다. 위쪽 및 아래쪽 화살표 키를 사용하여 선택한 다음 Enter를 눌러 확인하세요.
* 유휴 상태로 있다가 사라진 팀 동료 행은 중지된 것이 아니라 숨겨진 것입니다. 유휴 행은 전체 패널이 유휴 상태가 된 후 30초 후에 숨겨지고 팀 동료의 다음 차례에 다시 나타납니다. 3명 이상의 팀 동료가 유휴 상태일 때 초과 행은 Enter로 펼쳐지는 단일 `N idle agents` 행으로 접힙니다. 숨겨진 행을 다시 가져오려면 이름으로 해당 팀 동료에게 메시지를 보내세요.
* Claude에게 부여한 작업이 팀을 구성할 만큼 복잡했는지 확인하세요. Claude는 작업에 따라 팀 동료 생성 여부를 결정합니다.
* 분할 창을 명시적으로 요청한 경우 tmux가 설치되어 있고 PATH에서 사용 가능한지 확인하세요:
  ```bash theme={null}
  which tmux
  ```
* iTerm2의 경우 `it2` CLI가 설치되어 있고 iTerm2 환경설정에서 Python API가 활성화되어 있는지 확인하세요.

### 과도한 권한 프롬프트

팀 동료 권한 요청은 리더에게 에스컬레이션되어 마찰을 유발할 수 있습니다. 중단을 줄이려면 팀 동료를 생성하기 전에 [권한 설정](/docs/en/permissions)에서 일반적인 작업을 사전 승인하세요.

### 에러 시 팀 동료 중단

팀 동료는 복구하는 대신 에러를 만난 후 중단될 수 있습니다. in-process 모드에서는 에이전트 패널에서 팀 동료를 선택하고 Enter를 누르거나, 분할 모드에서는 창을 클릭하여 이들의 출력을 확인한 후 다음 중 하나를 수행하세요:

* 직접 추가 지침 제공
* 작업을 계속하도록 대체 팀 동료 생성

v2.1.198부터 실패한 API 요청을 재시도하기 위해 대기 중인 in-process 팀 동료는 리더나 다른 팀 동료의 메시지에 의해 깨어나므로 전체 재시도 지연을 기다리지 않고 즉시 재시도합니다.

### 작업이 끝나기 전에 리더 종료

모든 작업이 실제로 완료되기 전에 리더가 팀 작업이 끝났다고 판단할 수 있습니다. 이런 일이 발생하면 계속 진행하도록 지시하세요. 또한 리더가 위임하지 않고 직접 작업을 시작하는 경우 팀 동료가 마칠 때까지 기다렸다가 진행하도록 지시할 수도 있습니다.

### 고립된 tmux 세션 (Orphaned tmux sessions)

Claude Code 세션이 끝난 후에도 tmux 세션이 유지된다면 완전히 정리되지 않았을 수 있습니다. 세션을 나열하고 팀에서 생성한 세션을 종료하세요:

```bash theme={null}
tmux ls
tmux kill-session -t <session-name>
```

## 제한 사항 (Limitations)

에이전트 팀은 실험적 기능입니다. 주의해야 할 현재 제한 사항은 다음과 같습니다:

* **in-process 팀 동료의 세션 재개 불가**: `/resume` 및 `/rewind`는 in-process 팀 동료를 복원하지 않습니다. 세션을 재개한 후 리더는 더 이상 존재하지 않는 팀 동료에게 메시지를 시도할 수 있습니다. 이런 일이 발생하면 리더에게 새 팀 동료를 생성하도록 지시하세요.
* **작업 상태 지연 가능**: 팀 동료가 때때로 작업을 완료로 표시하지 못하여 의존적인 작업이 차단될 수 있습니다. 작업이 막힌 것으로 보이면 작업이 실제로 끝났는지 확인하고 작업 상태를 수동으로 업데이트하거나 리더에게 팀 동료를 독촉하도록 지시하세요.
* **종료가 느릴 수 있음**: 팀 동료는 종료하기 전에 현재 요청 또는 도구 호출을 완료하므로 시간이 걸릴 수 있습니다.
* **세션당 하나의 팀**: 세션에는 해당 세션에 스코프가 지정된 정확히 하나의 팀이 있습니다. 추가로 이름이 지정된 팀을 생성하거나 여러 세션에 걸쳐 팀을 공유할 수 없습니다.
* **중첩 팀 불가**: 팀 동료가 자체 팀 동료를 생성할 수 없습니다. 리더만 팀을 관리할 수 있습니다.
* **in-process 팀 동료의 백그라운드 서브에이전트 불가**: in-process 팀 동료의 자체 서브에이전트는 포그라운드에서 실행됩니다. `run_in_background`를 사용하든 `background: true`를 설정하는 서브에이전트 정의를 사용하든 백그라운드 작업을 요청하면 에러가 반환됩니다. 팀 동료의 백그라운드 작업은 리더의 프로세스보다 오래 지속될 수 없기 때문입니다. 메인 대화에서 실행된 서브에이전트는 [백그라운드 기본값](/docs/en/sub-agents#run-subagents-in-foreground-or-background)을 따릅니다.
* **리더 고정**: 메인 세션은 수명 동안 리더입니다. 팀 동료를 리더로 승격하거나 리더십을 이전할 수 없습니다.
* **생성 시 설정되는 권한**: 모든 팀 동료는 리더의 권한 모드로 시작합니다. 생성 후 개별 팀 동료 모드를 변경할 수 있지만, 생성 시점에 팀 동료별 모드를 설정할 수는 없습니다.
* **분할 창은 tmux 또는 iTerm2 필요**: 기본 in-process 모드는 모든 터미널에서 작동합니다. 분할 창 모드는 VS Code의 통합 터미널, Windows Terminal 또는 Ghostty에서 지원되지 않습니다.

<Tip>
  **`CLAUDE.md`는 정상 작동함**: 팀 동료는 작업 디렉터리에서 `CLAUDE.md` 파일을 읽습니다. 이를 사용하여 모든 팀 동료에게 프로젝트 전용 지침을 제공하세요.
</Tip>

## 다음 단계

병렬 작업 및 위임에 대한 관련 접근 방식을 탐색하세요:

* **경량 위임**: [서브에이전트](/docs/en/sub-agents)는 세션 내에서 연구나 검증을 위해 헬퍼 에이전트를 생성하며, 에이전트 간 조율이 필요 없는 작업에 더 좋습니다
* **수동 병렬 세션**: [Git worktrees](/docs/en/worktrees)를 사용하면 자동화된 팀 조율 없이 직접 여러 Claude Code 세션을 실행할 수 있습니다
* **접근 방식 비교**: 나란히 비교 분석한 [서브에이전트 vs 에이전트 팀](/docs/en/features-overview#compare-similar-features) 비교를 참조하세요
