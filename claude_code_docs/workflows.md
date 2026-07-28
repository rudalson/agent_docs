> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 동적 워크플로우를 사용한 대규모 서브에이전트 조율

> 동적 워크플로우는 Claude가 작성하고 재실행할 수 있는 스크립트를 통해 여러 서브에이전트를 조율합니다. 코드베이스 감사, 대규모 마이그레이션 및 교차 검증 연구에 사용하세요.

{/* plan-availability: feature=workflows plans=pro,max,team,enterprise providers=all */}

<Note>
  동적 워크플로우는 Claude Code v2.1.154 이상이 필요하며 모든 유료 플랜, Anthropic API 액세스, Amazon Bedrock, Google Cloud's Agent Platform 및 Microsoft Foundry에서 사용할 수 있습니다. Pro 플랜에서는 `/config`의 Dynamic workflows 행에서 켤 수 있습니다.
</Note>

동적 워크플로우는 [서브에이전트](/docs/en/sub-agents)를 대규모로 조율하는 JavaScript 스크립트입니다. Claude는 설명한 작업을 위한 스크립트를 작성하고, 런타임은 세션이 응답성을 유지하는 동안 백그라운드에서 해당 스크립트를 실행합니다.

한 대화에서 조율할 수 있는 것보다 더 많은 에이전트가 작업에 필요하거나, 조율 과정을 읽고 재실행할 수 있는 스크립트로 코드화하고 싶을 때 워크플로우를 활용하세요. 예시로는 코드베이스 전체의 버그 수색, 500개 파일 마이그레이션, 출처를 서로 교차 검증해야 하는 연구 질문, 커밋하기 전에 여러 독립적인 각도에서 작성해 볼 가치가 있는 철저한 계획 등이 있습니다.

## 워크플로우를 사용해야 하는 경우

[서브에이전트](/docs/en/sub-agents), [스킬](/docs/en/skills), [에이전트 팀](/docs/en/agent-teams), 워크플로우는 모두 다단계 작업을 실행할 수 있습니다. 차이점은 계획을 누가 가지고 있느냐입니다:

|                                 | 서브에이전트                      | 스킬                       | 에이전트 팀                            | 워크플로우                            |
| :------------------------------ | :----------------------------- | :--------------------------- | :------------------------------------- | :----------------------------------- |
| 개념                            | Claude가 생성하는 워커         | Claude가 따르는 지침          | 동료 세션을 감독하는 리드 에이전트     | 런타임이 실행하는 스크립트           |
| 다음 실행 결정 주체              | Claude (턴 단위)               | Claude (프롬프트 준수)        | 리드 에이전트 (턴 단위)                | 스크립트                             |
| 중간 결과가 저장되는 위치        | Claude의 컨텍스트 윈도우        | Claude의 컨텍스트 윈도우      | 공유 작업 목록                         | 스크립트 변수                        |
| 반복 가능한 요소                | 워커 정의                       | 지침                         | 팀 정의                                | 조율 그 자체                         |
| 규모                            | 턴당 몇 개의 위임된 작업        | 서브에이전트와 동일           | 몇 안 되는 장시간 실행 동료            | 실행당 수십~수백 개의 에이전트       |
| 중단 시 동작                    | 턴 재시작                      | 턴 재시작                    | 팀원은 계속 실행                       | 동일 세션에서 재개 가능               |

워크플로우는 계획을 코드로 이동합니다. 서브에이전트, 스킬, 에이전트 팀의 경우 Claude가 조율자입니다: Claude가 다음에 생성하거나 할당할 항목을 턴 단위로 결정하고 모든 결과가 컨텍스트 윈도우에 놓입니다. 워크플로우 스크립트는 루프, 브랜칭, 중간 결과 자체를 보유하므로 Claude의 컨텍스트에는 최종 답변만 들어갑니다.

계획을 코드로 옮기면 더 많은 에이전트를 실행하는 것뿐만 아니라 워크플로우에 반복 가능한 품질 패턴을 적용할 수 있습니다: 결과가 보고되기 전에 독립적인 에이전트가 서로의 결과를 적대적으로 검토하도록 하거나, 여러 각도에서 플랜을 작성하고 비교하여 단일 패스보다 더 신뢰할 수 있는 결과를 얻을 수 있습니다.

## 번들로 제공되는 워크플로우 실행

워크플로우가 작동하는 것을 보는 가장 빠른 방법은 여러 출처에 걸쳐 질문을 조사하기 위해 Claude Code에 포함된 [내장 워크플로우](#bundled-workflows)인 `/deep-research`를 실행해 보는 것입니다. 세션이 자유로운 상태를 유지하는 동안 에이전트가 백그라운드에서 일련의 단계를 거쳐 작업하는 것을 볼 수 있으며, 턴 바이 턴 트랜스크립트 대신 하나의 보고서를 받게 됩니다.

<Steps>
  <Step title="워크플로우 실행">
    조사하려는 질문과 함께 `/deep-research`를 실행하세요. 여러 각도에서 웹 검색을 전개하고, 찾은 출처를 가져와 교차 검증하며, 출처가 인용된 보고서를 종합합니다.

    ```text theme={null}
    /deep-research What changed in the Node.js permission model between v20 and v22?
    ```
  </Step>

  <Step title="워크플로우 허용">
    Claude Code가 워크플로우 허용 여부를 묻습니다. 계속하려면 **Yes**를 선택하세요. 정확한 프롬프트는 권한 모드에 따라 다릅니다. 모드별 옵션은 [실행 전 플랜 승인](#approve-the-plan-before-it-runs)을 참조하세요.
  </Step>

  <Step title="진행 상황 관찰">
    실행이 백그라운드에서 시작됩니다. `/workflows`를 실행하고 화살표 키를 사용하여 실행 항목을 선택한 다음 Enter를 눌러 진행 상황 뷰를 여세요:

    ```text theme={null}
    /workflows
    ```

    이 뷰는 에이전트 수, 토큰 합계, 경과 시간과 함께 각 단계를 보여줍니다. 임의의 단계를 드릴다운하여 해당 에이전트와 각 에이전트가 발견한 내용을 확인할 수 있습니다. 전체 제어 키는 [실행 관찰](#watch-the-run)을 참조하세요.

    입력 상자 아래의 작업 패널에서도 관찰할 수 있습니다: 실행이 진행되는 동안 한 줄 진행 요약이 표시됩니다. 아래쪽 화살표를 눌러 포커스를 맞춘 다음 Enter를 눌러 펼치세요.
  </Step>

  <Step title="보고서 읽기">
    실행이 끝나면 보고서가 세션에 들어옵니다. 각 주장의 출처를 인용하며, 교차 검증을 통과하지 못한 주장은 이미 필터링되어 제외됩니다.

    {/* min-version: 2.1.196 */}v2.1.196부터 속도 제한이나 API 오류 등으로 인해 검증 에이전트가 주장을 검사할 수 없는 경우, 해당 주장을 반박된 것으로 간주하는 대신 미검증(unverified)으로 나열합니다.
  </Step>
</Steps>

자신만의 작업을 위해 워크플로우를 실행하려면 [Claude가 작성하도록 하고](#have-claude-write-a-workflow), 원하는 대로 작업이 수행되면 나만의 명령으로 [저장](#save-the-workflow-for-reuse)할 수 있습니다.

### 번들로 제공되는 워크플로우

Claude Code에는 `/deep-research`가 내장 워크플로우로 포함되어 있습니다:

| 명령어                     | 기능                                                                                                                                                                                                                                                                                                      |
| :-------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/deep-research <question>` | 여러 각도에서 질문에 대한 웹 검색을 전개하고, 찾은 출처를 가져와 교차 검증하며, 각 주장에 대해 투표하고, 교차 검증을 통과하지 못한 주장은 필터링하여 인용된 보고서를 반환합니다. [WebSearch tool](/docs/en/tools-reference#websearch-tool-behavior)이 제공되어야 합니다 |

직접 [저장한 워크플로우](#save-the-workflow-for-reuse)는 동일한 방식으로 명령어가 되며 번들 명령어와 함께 `/` 자동 완성에 표시됩니다.

### 실행 관찰

워크플로우는 백그라운드에서 실행되므로 에이전트가 작업하는 동안에도 세션이 응답성을 유지합니다. 언제든지 `/workflows`를 실행하여 실행 중이거나 완료된 워크플로우를 나열한 다음, 하나를 선택하여 진행 상황 뷰를 여세요.

```text theme={null}
/workflows
```

진행 상황 뷰는 에이전트 수, 토큰 합계, 경과 시간과 함께 각 단계를 보여줍니다. 푸터에는 각 작업의 키가 나열됩니다:

| 키            | 작업                                                                                                                      |
| :------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| `↑` / `↓`      | 단계 또는 에이전트 선택                                                                                                     |
| `Enter` 또는 `→` | 선택한 단계로 드릴다운한 다음 에이전트로 드릴다운하여 해당 프롬프트, 최근 도구 호출 및 결과 읽기                         |
| `Esc` 또는 `←`   | 한 단계 뒤로 이동. v2.1.203~v2.1.205에서는 `←`가 단계나 에이전트에서 뒤로 빠져나오지 않았으므로 해당 버전에서는 `Esc`를 사용하세요 |
| `j` / `k`      | 에이전트 세부 정보가 넘칠 때 스크롤                                                                            |
| `f`            | {/* min-version: 2.1.186 */}선택한 단계의 에이전트 목록을 상태별로 필터링. 다시 눌러 순환                     |
| `p`            | 실행 일시 중지 또는 재개                                                                                                     |
| `x`            | 선택한 에이전트 중지, 포커스가 실행에 있을 때는 전체 워크플로우 중지                                                |
| `r`            | 선택한 실행 중인 에이전트 재시작                                                                                          |
| `s`            | 실행 스크립트를 명령으로 [저장](#save-the-workflow-for-reuse)                                                          |

## Claude가 워크플로우를 작성하도록 하기

두 가지 방법으로 Claude가 작업을 위한 워크플로우를 작성하도록 할 수 있습니다:

* 프롬프트에서 [워크플로우 요청하기](#ask-for-a-workflow-in-your-prompt): 자신의 단어로 또는 `ultracode` 키워드를 포함하여 프롬프트에서 요청하면 Claude가 해당 작업을 위한 워크플로우를 작성합니다.
* [ultracode로 Claude가 결정하게 하기](#let-claude-decide-with-ultracode): `/effort ultracode`를 설정하면 Claude가 세션의 모든 실질적인 작업에 대해 워크플로우를 계획합니다.

[번들 워크플로우](#bundled-workflows)(예: `/deep-research`) 또는 [저장한 워크플로우](#save-the-workflow-for-reuse) 등 이미 존재하는 워크플로우 명령을 실행할 수도 있습니다.

### 프롬프트에서 워크플로우 요청하기

세션의 노력 수준을 변경하지 않고 단일 작업을 워크플로우로 실행하려면 프롬프트에 `ultracode` 키워드를 포함하세요. "use a workflow"나 "run a workflow"처럼 자신의 단어로 요청하는 것도 동일하게 동작합니다: Claude는 직접적인 요청을 동일한 옵트인으로 취급합니다. v2.1.160 이전의 리터럴 트리거 키워드는 `workflow`였으며 자연어 요청은 두 버전 모두에서 작동합니다.

```text theme={null}
ultracode: audit every API endpoint under src/routes/ for missing auth checks
```

Claude Code는 입력에서 키워드를 강조 표시하고 Claude는 턴 바이 턴으로 작업하는 대신 해당 작업을 위한 워크플로우 스크립트를 작성합니다. 키워드는 Claude가 작업을 구성하는 방식만 선택합니다: 이 방식으로 시작된 워크플로우는 세션의 기존 [권한 모드](/docs/en/permission-modes) 내에서 실행되며, 에이전트의 도구 호출은 세션의 다른 도구 호출과 동일한 권한 검사 및 [샌드박싱](/docs/en/sandboxing)을 받습니다.

실행 결과가 원하는 대로 나오면 나중에 [명령으로 저장](#save-the-workflow-for-reuse)할 수 있습니다. 이미 서브에이전트 프롬프트 폴더나 작업을 분산시키는 스킬 등 다른 방식으로 구축된 오케스트레이터가 있는 경우 Claude에게 이를 가리키며 동일한 작업을 수행하는 워크플로우를 요청할 수 있습니다.

#### 키워드 해제 또는 끄기

워크플로우를 시작하려던 것이 아니라면 macOS에서 `Option+W`, Windows 및 Linux에서 `Alt+W`를 눌러 이 프롬프트의 하이라이트를 해제하거나, 강조 표시된 키워드 바로 뒤에 커서가 있을 때 백스페이스를 누르세요. 키워드 트리거를 완전히 정지하려면 `/config`에서 Ultracode keyword trigger를 끄세요.

#### 키워드가 작동하는 위치

키워드는 직접 입력하는 프롬프트에서만 옵트인으로 작동합니다: 대화형 프롬프트, IDE 확장 프로그램 패널, [Remote Control](/docs/en/remote-control) 클라이언트, 또는 키보드 입력의 [`origin`](/docs/en/agent-sdk/typescript#sdkmessageorigin)을 `{ kind: "human" }`으로 날인하는 Agent SDK 애플리케이션. 다른 경로로 세션에 도달할 때는 워크플로우를 시작하지 않습니다:

* `-p`로 전달된 프롬프트
* 사람 입력으로 날인되지 않은 Agent SDK 애플리케이션 전달 프롬프트
* 예약된 작업 프롬프트
* 대화로 전달된 웹훅 페이로드 또는 풀 리퀘스트 댓글

<Note>
  v2.1.210 이전에는 웹훅 페이로드나 대화로 전달된 풀 리퀘스트 댓글을 포함해 이러한 모든 경로에서도 키워드가 워크플로우를 시작했습니다.
</Note>

### ultracode로 Claude가 결정하게 하기

Ultracode는 `xhigh` [추론 노력(reasoning effort)](/docs/en/model-config#adjust-effort-level)과 자동 워크플로우 조율을 결합하는 Claude Code 설정입니다. 이 설정을 켜면 Claude는 사용자가 요청할 때까지 기다리지 않고 실질적인 각 작업에 대해 워크플로우를 계획합니다.

```text theme={null}
/effort ultracode
```

ultracode가 켜진 상태로 세션을 시작하려면 `claude --effort ultracode`로 실행하세요. Claude Code v2.1.203 이상이 필요합니다.

ultracode가 활성화되면 Claude는 작업에 워크플로우가 필요한지 스스로 결정합니다. 단일 요청이 여러 워크플로우의 연속으로 바뀔 수 있습니다: 코드를 이해하기 위한 워크플로우 하나, 변경을 수행하기 위한 워크플로우 하나, 이를 검증하기 위한 워크플로우 하나. 이는 세션의 모든 작업에 적용되므로 각 요청은 낮은 노력 수준보다 더 많은 토큰을 사용하고 시간이 더 오래 걸립니다.

Ultracode는 현재 세션 동안 유지되며 새 세션을 시작하면 리셋됩니다. 일상적인 작업으로 돌아갈 때는 `/effort high`로 낮추세요. `xhigh` [노력](/docs/en/model-config#adjust-effort-level)을 지원하는 모델에서 사용할 수 있으며, 다른 모델에서는 `/effort` 메뉴에 표시되지 않습니다.

### 실행 전 플랜 승인

CLI에서 실행별 프롬프트에는 계획된 단계와 다음 옵션이 표시됩니다:

* **Yes, run it**: 실행 시작
* **Yes, and don't ask again for `<name>` in `<path>`**: 시작하며, 이후 이 프로젝트에서 이 워크플로우에 대한 이 프롬프트를 건너뜁니다
* **View raw script**: 결정하기 전에 원시 스크립트 읽기
* **No**: 취소

`Ctrl+G`는 에디터에서 스크립트를 엽니다. `Tab`을 사용하면 실행이 시작되기 전에 프롬프트를 조정할 수 있습니다.

이 프롬프트가 표시되는지 여부는 [권한 모드](/docs/en/permission-modes)에 따라 다릅니다:

| 권한 모드                            | 프롬프트 표시 시점                                                                                                                                    |
| :----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Default, accept edits                      | 매 실행 시 (단, 해당 프로젝트에서 해당 워크플로우에 대해 **Yes, and don't ask again**을 선택한 경우 제외)                                                        |
| Auto                                       | 최초 실행 시에만. **Yes** 선택 시 사용자 설정에 동의가 기록되며, 이후 실행은 프롬프트 없이 시작됩니다. ultracode가 켜져 있으면 완전히 건너뜁니다 |
| Bypass permissions, `claude -p`, Agent SDK | 프롬프트를 표시하지 않음. 실행이 즉시 시작됩니다                                                                                                                       |

데스크톱 앱에서는 워크플로우 이름, 단계 목록, 토큰 사용 주의사항과 함께 **Once**, **Always**, **Deny** 작업이 포함된 승인 카드가 표시됩니다. 진행 상황 뷰는 백그라운드 작업 사이드 창에 나타납니다.

권한 모드는 위의 실행 프롬프트만 제어합니다. 워크플로우가 생성하는 서브에이전트는 세션 모드와 상관없이 항상 `acceptEdits` 모드로 실행되며 사용자의 [도구 허용 목록](/docs/en/settings#permission-settings)을 상속받습니다. 파일 편집은 자동 승인됩니다.

허용 목록에 없는 쉘 명령, 웹 가져오기, MCP 도구는 실행 중간에 프롬프트를 표시할 수 있습니다. 긴 실행에서 이를 방지하려면 시작하기전에 에이전트에 필요한 명령을 허용 목록에 추가하세요.

`claude -p` 및 Agent SDK에는 프롬프트를 받을 사람이 없으므로 도구 호출은 대화형 확인 없이 구성된 권한 규칙을 따릅니다.

### 재사용을 위해 워크플로우 저장

Claude가 반복할 작업을 위한 워크플로우를 작성할 때 해당 실행의 스크립트를 명령으로 저장할 수 있습니다. 모든 브랜치에서 실행하는 리뷰와 같은 프로세스는 매번 동일한 조율을 실행하게 됩니다.

`/workflows`를 실행하고 유지하려는 실행 항목을 선택한 후 `s`를 누르세요. 저장 대화 상자에서 Tab 키를 통해 두 저장 위치를 전환할 수 있습니다:

* 프로젝트 내 `.claude/workflows/`: 리포지토리를 클론하는 모든 사람과 공유됨
* 홈 디렉토리 내 `~/.claude/workflows/`: 모든 프로젝트에서 사용할 수 있으며 본인에게만 보임. [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars)을 설정한 경우 이 위치는 해당 경로 하위의 `workflows/` 디렉토리입니다.

{/* min-version: 2.1.208 */}저장 대화 상자에는 개인 위치의 확인된 경로가 표시됩니다. v2.1.208 이전에는 `CLAUDE_CONFIG_DIR`이 설정되어 있어도 `~/.claude/workflows/`를 표시했습니다; 파일은 여전히 구성된 디렉토리 하위에 저장되었습니다.

Enter를 눌러 저장하세요. 워크플로우는 두 위치 모두에서 향후 세션 시 `/<name>`으로 실행됩니다.

{/* min-version: 2.1.178 */}여러 `.claude/` 디렉토리가 있는 모노리포에서는 적용 대상 패키지와 함께 워크플로우를 보관할 수 있습니다. v2.1.178부터 프로젝트 위치에 저장하면 작업 디렉토리와 리포지토리 루트 사이에 이미 존재하는 가장 가까운 `.claude/workflows/` 디렉토리에 작성되며, 아직 없으면 리포지토리 루트에 작성됩니다. 프로젝트 워크플로우는 해당 경로를 따른 모든 `.claude/workflows/`에서도 로드되며, 둘 이상이 동일한 이름을 정의하면 Claude Code는 작업 디렉토리에 가장 가까운 워크플로우를 실행합니다.

프로젝트 워크플로우와 개인 워크플로우가 이름을 공유하는 경우 프로젝트 워크플로우가 실행됩니다.

### 저장된 워크플로우에 입력 전달

저장된 워크플로우는 `args` 매개변수를 통해 입력을 수락할 수 있습니다. 스크립트는 이를 `args`라는 전역 이름으로 읽습니다. 매 실행마다 스크립트를 편집하는 대신 호출 시 연구 질문, 대상 경로 목록, 구성 개체를 제공하는 데 사용하세요.

다음 프롬프트는 이슈 번호 목록과 함께 저장된 워크플로우를 실행합니다:

```text theme={null}
> Run /triage-issues on issues 1024, 1025, and 1030
```

Claude는 목록을 구조화된 데이터로 전달하므로 스크립트는 먼저 파싱하지 않고도 `args`에서 직접 배열 및 개체 메서드를 호출할 수 있습니다. `args`가 생략되면 스크립트 내부에서 전역 값은 `undefined`입니다.

## 워크플로우 프롬프트 예시

워크플로우는 한 에이전트가 컨텍스트에 담기에는 작업이 너무 크거나 동일한 단계를 여러 항목에 걸쳐 실행해야 할 때 가장 적합합니다. 아래 프롬프트는 일반적인 형태를 보여줍니다. 각 프롬프트는 Claude에게 해당 작업을 위한 워크플로우를 작성하고 실행하도록 요청합니다; 스크립트를 직접 작성할 필요는 없습니다.

### 동일한 문제에 대해 여러 파일 감사

파일당 하나의 에이전트를 배치한 다음 결과를 수집하고 검증합니다.

```text theme={null}
> use a workflow to audit every route handler under src/routes/ for missing authentication checks, and adversarially verify each finding before reporting it
```

### 검사가 통과할 때까지 계속 수정

검사기를 실행하고 실패한 항목을 수정한 다음, 통과하거나 더 이상 진전이 없을 때까지 반복합니다.

```text theme={null}
> use a workflow to run npx tsc --noEmit and keep fixing the reported errors until the type check passes or two rounds in a row make no progress
```

### 병렬로 여러 파일 마이그레이션

마이그레이션할 파일을 찾고, 편집 충돌이 없도록 격리된 사본에서 각 파일을 변환한 후 각 결과를 검증합니다.

```text theme={null}
> use a workflow to migrate every component under src/components/ from styled-components to Tailwind, working on each file in its own isolated copy
```

### 변경된 모든 파일 리뷰 및 하나의 요약 작성

파일당 리뷰어를 실행한 다음, 모든 결과를 정렬하고 중복을 제거하는 한 에이전트에 전달합니다.

```text theme={null}
> use a workflow to review every file changed in this PR for correctness issues, then merge the per-file findings into one ranked summary
```

### 여러 출처에 걸쳐 주제 연구

변경 로그, 이슈, 문서 전반에 걸쳐 읽기 에이전트를 배치한 다음 종합합니다. 번들로 제공되는 `/deep-research` 워크플로우가 이 작업을 수행하며, 더 좁은 버전을 설명할 수도 있습니다.

```text theme={null}
> use a workflow to research how our three competitors handle rate limiting: read their public docs and recent changelog entries in parallel, then compare the approaches
```

### 목록 작성이 지연될 때까지 문제 탐색

라운드별로 계속 검색하고 새 라운드에서 새로운 내용이 나오지 않으면 중지합니다.

```text theme={null}
> use a workflow to find flaky tests in this repo: run the suite repeatedly, record which tests fail intermittently, and stop once two rounds in a row find nothing new
```

### 저장된 스크립트의 모습

[워크플로우를 저장](#save-the-workflow-for-reuse)하면 `.claude/workflows/` 내의 파일에는 서브에이전트를 조율하는 스크립트 본문 앞에 `meta` 블록이 포함됩니다. 수동으로 편집할 필요는 없지만 Claude가 생성한 내용을 알아볼 수 있도록 소형 구조를 보여줍니다:

```javascript theme={null}
export const meta = {
  name: 'audit-routes',
  description: 'Audit every route handler for missing auth checks',
}

const found = await agent('List every .ts file under src/routes/.', {
  schema: { type: 'object', required: ['files'], properties: { files: { type: 'array', items: { type: 'string' } } } },
})

const audits = await pipeline(found.files, file =>
  agent(`Audit ${file} for missing authentication checks.`, { label: file }),
)

return audits.filter(Boolean)
```

본문은 최상위 `await`를 포함하는 일반 JavaScript입니다. `agent()`는 서브에이전트 하나를 생성하고 `pipeline()`은 목록의 항목당 하나씩 실행합니다. 스크립트를 직접 편집하려면 Claude에게 변경 과정을 시연해 달라고 요청하거나 [Agent SDK reference](/docs/en/agent-sdk/typescript)의 Workflow 도구 항목을 참조하세요.

## 워크플로우 실행 방식

워크플로우 런타임은 대화와 격리된 환경에서 스크립트를 실행합니다. 중간 결과는 Claude의 컨텍스트에 포함되는 대신 스크립트 변수에 유지됩니다.

모든 실행은 `~/.claude/projects/`에 있는 세션 디렉토리 하위의 파일에 스크립트를 기록합니다. 실행이 시작될 때 Claude가 해당 경로를 받으므로 이를 요청할 수 있습니다. 해당 파일을 열어 Claude가 작성한 조율 내용을 읽거나 이전 실행 스크립트와의 차이점을 파악하거나 편집한 버전에서 Claude에게 재실행하도록 요청할 수 있습니다.

런타임은 진행됨에 따라 각 에이전트의 결과를 추적하므로 동일 세션 내에서 실행을 [재개](#resume-after-a-pause)할 수 있습니다.

### 동작 및 제한

런타임은 다음 제약 조건을 적용합니다:

| 제약 조건                                                           | 이유                                                                                                            |
| :------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------- |
| 실행 중 사용자 입력 불가                                                | 에이전트 권한 프롬프트만 실행을 일시 중지할 수 있습니다. 단계 간 승인을 위해서는 각 단계를 자체 워크플로우로 실행하세요 |
| 워크플로우 자체에서 파일 시스템 또는 쉘에 직접 액세스 불가        | 에이전트가 명령을 읽고 쓰며 실행합니다. 스크립트는 에이전트를 조율합니다                                        |
| 동시 에이전트 최대 16개 (CPU 코어가 제한된 머신에서는 더 적음) | 로컬 리소스 사용 제한                                                                                      |
| 실행당 총 1,000개 에이전트                                           | 제어 불능 루프 방지                                                                                         |

## 실행 관리

실행이 시작되면 `/workflows` 뷰에서 관리하거나 입력 상자 아래 작업 패널의 진행 상황 줄을 확장하여 관리합니다.

### 일시 정지 후 재개

실행을 중지한 경우 재개할 수 있습니다: 이미 완료된 에이전트는 캐시된 결과를 반환하고 나머지는 라이브로 실행됩니다. 중지할 때 여전히 실행 중이었던 에이전트는 저장되지 않고 재개 시 처음부터 다시 시작되므로, 작업을 많은 소형 에이전트에 분산하는 워크플로우가 하나의 긴 에이전트보다 더 많은 진행 상황을 보존합니다. `/workflows`에서 일시 정지된 실행을 선택하고 `p`를 눌러 재개하거나 동일한 스크립트로 워크플로우를 다시 실행하도록 Claude에 요청하세요.

재개는 동일한 Claude Code 세션 내에서 동작합니다. 워크플로우가 실행 중일 때 Claude Code를 종료하면 다음 세션은 워크플로우를 새로 시작합니다.

### 비용

워크플로우는 많은 에이전트를 생성하므로 단일 실행이 대화에서 동일한 작업을 수행하는 것보다 의미 있게 더 많은 토큰을 사용할 수 있습니다. 실행은 다른 세션과 마찬가지로 플랜의 사용량 및 속도 제한에 포함됩니다.

대규모 작업에 임하기 전에 소모량을 측정하려면 먼저 작은 조각에서 워크플로우를 실행하세요: 전체 리포지토리 대신 하나의 디렉토리, 또는 광범위한 질문 대신 좁은 질문. `/workflows` 뷰는 실행이 진행됨에 따라 각 에이전트의 토큰 사용량을 보여주며, 완료된 작업을 잃지 않고 언제든지 거기에서 실행을 중지할 수 있습니다. 런타임의 [에이전트 한도](#behavior-and-limits)는 단일 실행이 생성할 수 있는 에이전트 수를 제한하므로 제어 불능 스크립트의 비용을 제한합니다. 기본적으로 모든 실행을 더 작게 유지하려면 `/config`에서 [크기 지침을 설정](#set-a-size-guideline)하세요.

Claude Code는 지나치게 커지는 실행에 대해서도 플래그를 지정합니다. 워크플로우가 25개 이상의 에이전트를 예약하거나 예상 토큰 합계가 150만 개를 초과하면 입력 상자 아래 작업 패널의 진행 상황 줄에 `Large workflow` 경고가 표시됩니다. 경고는 실행을 중지할 수 있는 [`/workflows`](#watch-the-run)를 가리킵니다. Claude Code v2.1.203 이상이 필요합니다.

이 경고는 권고 사항입니다: 실행을 일시 중지하거나 제한하지 않습니다. 두 가지 설정으로 표시 시점을 변경할 수 있습니다:

* [크기 지침을 설정](#set-a-size-guideline)하면 지침의 에이전트 수가 25개 에이전트 임계값을 대체합니다.
* [ultracode](#let-claude-decide-with-ultracode)가 켜진 세션에서는 대규모 실행을 이미 옵트인한 것이므로 경고를 표시하지 않습니다.

스크립트가 다른 모델에 단계를 라우팅하거나 둘 다 오버라이드하는 [`CLAUDE_CODE_SUBAGENT_MODEL`](/docs/en/model-config#environment-variables) 환경 변수가 설정되어 있지 않은 한 워크플로우의 모든 에이전트는 세션의 모델을 사용합니다. 모델 비용을 제어하려면:

* 일상적인 작업을 위해 보통 더 작은 모델로 전환하는 경우 대규모 실행 전에 `/model`을 확인하세요
* 작업을 설명할 때 가장 강력한 모델이 필요하지 않은 단계에는 더 작은 모델을 사용하도록 Claude에 요청하세요

### 크기 지침 설정

`/config`에서 Dynamic workflow size 설정은 Claude가 작성하는 워크플로우를 기본적으로 더 작은 규모로 유지합니다. Claude Code는 설정을 권고로 Claude에 전달하므로 다른 규모를 요구하는 프롬프트가 이를 오버라이드합니다. Claude Code v2.1.202 이상이 필요합니다.

각 값은 Claude가 작성하는 스크립트에서 목표로 하는 에이전트 수를 설정합니다.

| 값          | Claude에 전달되는 지침            |
| :------------- | :--------------------------------- |
| `unrestricted` | 지침 없음. 기본값입니다. |
| `small`        | 에이전트 5개 미만 목표.       |
| `medium`       | 에이전트 15개 미만 목표.      |
| `large`        | 에이전트 50개 미만 목표.      |

변경 사항은 다음 프롬프트부터 적용됩니다. 설정과 상관없이 [런타임 에이전트 한도](#behavior-and-limits)는 여전히 적용됩니다.

### 워크플로우 끄기

워크플로우는 CLI, 데스크톱 앱, IDE 확장 프로그램, `claude -p`를 사용한 [비대화형 모드](/docs/en/headless) 및 [Agent SDK](/docs/en/agent-sdk/overview)에서 사용할 수 있습니다. 모든 표면에서 동일한 비활성화 설정이 적용됩니다.

자신을 위해 워크플로우를 끄려면:

* `/config`에서 Dynamic workflows를 비활성화로 전환합니다. 세션 전반에 유지됩니다.
* `~/.claude/settings.json`에서 `"disableWorkflows": true`를 설정합니다. 세션 전반에 유지됩니다.
* `CLAUDE_CODE_DISABLE_WORKFLOWS=1`을 설정합니다. 시작 시 읽히므로 설정된 모든 곳에 적용됩니다.

전체 조직에 대해 워크플로우를 끄려면 [관리형 설정](/docs/en/server-managed-settings)에서 `"disableWorkflows": true`를 설정하거나 [Claude Code 관리자 설정](https://claude.ai/admin-settings/claude-code) 페이지의 토글을 사용하세요.

워크플로우가 비활성화되면 번들 워크플로우 명령을 사용할 수 없으며 `ultracode` 키워드가 더 이상 실행을 트리거하지 않고 `/effort` 메뉴에서 `ultracode`가 제거됩니다.

## 관련 리소스

* [에이전트 병렬 실행](/docs/en/agents): 서브에이전트, 에이전트 뷰, 에이전트 팀 및 워크플로우 비교
* [커스텀 서브에이전트 만들기](/docs/en/sub-agents): 워크플로우가 조율하는 워커 원형
* [비용 관리](/docs/en/costs): 멀티 에이전트 실행이 사용 한도에 반영되는 방식
