> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 출력 스타일 (Output styles)

> 소프트웨어 엔지니어링 이외의 용도에 맞게 Claude Code 적응시키기

출력 스타일은 Claude가 알고 있는 지식이 아니라 Claude가 응답하는 방식을 변경합니다. 출력 스타일은 시스템 프롬프트를 수정하여 역할, 톤, 출력 형식을 설정합니다. 매 턴마다 동일한 어조나 형식을 계속 재요청해야 하거나 Claude가 소프트웨어 엔지니어가 아닌 다른 역할로 작동하도록 하려는 경우 출력 스타일을 사용하세요.

커스텀 출력 스타일은 시스템 프롬프트에 사용자의 지침을 추가하고 Claude Code의 내장 소프트웨어 엔지니어링 지침을 유지할지 여부를 선택할 수 있도록 합니다. 항상 다이어그램으로 응답하는 것과 같이 Claude가 소통하는 방식만 변경하고 여전히 코딩 작업을 수행할 때는 해당 지침을 유지하세요. 글쓰기 보조나 데이터 분석가와 같이 Claude가 소프트웨어 엔지니어링을 전혀 수행하지 않을 때는 해당 지침을 제외하세요.

프로젝트, 규칙 또는 코드베이스에 관한 지침은 대신 [CLAUDE.md](/docs/en/memory)를 사용하세요.

## 내장 출력 스타일

Claude Code의 **기본(Default)** 출력 스타일은 소프트웨어 엔지니어링 작업을 효율적으로 완료할 수 있도록 설계된 기존 시스템 프롬프트입니다.

세 가지 추가 내장 출력 스타일이 있습니다:

* **주도적(Proactive)**: Claude가 즉시 실행하고, 일상적인 결정을 위해 멈추는 대신 합리적인 가정을 내리며, 계획보다 실행을 우선시합니다. 이는 [자동 모드(auto mode)](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)가 적용하는 것보다 강력한 자율 실행 지침이며, 권한 모드를 변경하지 않고 작동하므로 도구가 실행되기 전에 여전히 권한 프롬프트가 표시됩니다.

* **설명형(Explanatory)**: 소프트웨어 엔지니어링 작업을 완료하도록 돕는 사이에 교육적인 "인사이트(Insights)"를 제공합니다. 구현 선택과 코드베이스 패턴을 이해하는 데 도움을 줍니다.

* **학습형(Learning)**: 협력적인 실습 모드로, Claude가 코딩 중에 "인사이트"를 공유할 뿐만 아니라 소규모의 전략적 코드 조각을 직접 기여하도록 요청합니다. Claude Code는 직접 구현할 수 있도록 코드에 `TODO(human)` 마커를 추가합니다.

## 출력 스타일 변경

`/config`를 실행하고 **Output style**을 선택하여 메뉴에서 스타일을 고릅니다. 선택 사항은 [로컬 프로젝트 수준](/docs/en/settings)의 `.claude/settings.local.json`에 저장됩니다.

<Note>{/* max-version: 2.1.90 */}독립형 `/output-style` 명령은 v2.1.73에서 폐지되었으며 v2.1.91에서 제거되었습니다. `/config`를 사용하거나 `outputStyle` 설정을 직접 편집하세요.</Note>

메뉴 없이 스타일을 설정하려면 설정 파일에서 `outputStyle` 필드를 직접 편집하세요:

```json theme={null}
{
  "outputStyle": "Explanatory"
}
```

출력 스타일은 시스템 프롬프트의 일부이며 Claude Code는 세션 시작 시 한 번 읽습니다. 변경 사항은 `/clear`를 실행하거나 새 세션을 시작한 후에 적용됩니다. 출력 스타일 변경이 캐시에 미치는 영향은 [Claude Code가 프롬프트 캐싱을 사용하는 방법](/docs/en/prompt-caching#changing-output-style)을 참조하세요.

## 커스텀 출력 스타일 만들기

커스텀 출력 스타일은 메타데이터용 프론트매터(frontmatter)와 시스템 프롬프트에 추가할 지침이 포함된 Markdown 파일입니다.

<Steps>
  <Step title="Markdown 파일 만들기">
    세 가지 수준 중 하나에 저장합니다. 파일 이름은 프론트매터에 `name`을 설정하지 않는 한 스타일 이름이 됩니다.

    * 사용자: `~/.claude/output-styles`
    * 프로젝트: `.claude/output-styles`
    * 관리 정책: [관리형 설정 디렉터리](/docs/en/settings#settings-files) 내부의 `.claude/output-styles`

    프로젝트 출력 스타일은 작업 디렉터리와 리포지토리 루트 사이의 모든 `.claude/output-styles/`에서 로드됩니다. {/* min-version: 2.1.178 */}v2.1.178부터 이러한 중첩 디렉터리 중 둘 이상이 동일한 이름의 스타일을 정의하는 경우 Claude Code는 작업 디렉터리에 가장 가까운 것을 사용합니다.
  </Step>

  <Step title="프론트매터 및 지침 추가">
    Claude Code의 소프트웨어 엔지니어링 지침을 유지할지 결정합니다. Claude의 소통 방식은 변경하지만 동일한 방식으로 코딩하기를 원하는 경우 `keep-coding-instructions: true`를 설정하세요. Claude가 소프트웨어 엔지니어링을 수행하지 않는 경우 이 설정을 제외하세요.

    다음 예시는 Claude의 코딩 동작을 유지하면서 모든 설명을 다이어그램으로 시작하도록 합니다:

    ```markdown theme={null}
    ---
    name: Diagrams first
    description: Lead every explanation with a diagram
    keep-coding-instructions: true
    ---

    When explaining code, architecture, or data flow, start with a Mermaid diagram showing the structure, then explain in prose.

    ## Diagram conventions

    Use `flowchart TD` for control flow and `sequenceDiagram` for request paths. Keep diagrams under 15 nodes.
    ```
  </Step>

  <Step title="스타일로 전환">
    `/config`를 실행하고 **Output style** 아래에서 만든 스타일을 선택하세요. `/clear` 실행 후 또는 다음 세션을 시작할 때 적용됩니다.
  </Step>
</Steps>

[플러그인](/docs/en/plugins-reference)도 `output-styles/` 디렉터리에 출력 스타일을 포함하여 제공할 수 있습니다.

### 프론트매터

출력 스타일 파일은 다음 프론트매터 필드를 지원합니다:

| 프론트매터 | 목적 | 기본값 |
| :--- | :--- | :--- |
| `name` | 파일 이름과 다른 경우 출력 스타일의 이름 | 파일 이름에서 상속됨 |
| `description` | `/config` 선택기에 표시되는 출력 스타일 설명 | 없음 |
| `keep-coding-instructions` | Claude Code의 내장 소프트웨어 엔지니어링 지침 유지 여부 | `false` |
| `force-for-plugin` | 플러그인 출력 스타일 전용: 사용자가 선택하지 않아도 플러그인이 활성화될 때마다 이 스타일을 자동으로 적용합니다. 사용자의 `outputStyle` 설정을 재정의합니다. 활성화된 여러 플러그인이 이를 설정한 경우 Claude Code는 가장 먼저 로드된 플러그인의 스타일을 사용합니다. | `false` |

## 출력 스타일의 작동 방식

출력 스타일은 Claude Code의 시스템 프롬프트를 직접 수정합니다.

* 모든 출력 스타일은 시스템 프롬프트 끝에 커스텀 지침이 추가됩니다.
* 모든 출력 스타일은 대화 중에 Claude가 출력 스타일 지침을 준수하도록 리마인더를 트리거합니다.
* 커스텀 출력 스타일은 `keep-coding-instructions`가 `true`로 설정되지 않는 한 변경 사항 범위 지정, 주석 작성, 작업 검증 방법과 같은 Claude Code의 내장 소프트웨어 엔지니어링 지침을 제외합니다.

출력 스타일은 메인 대화에만 적용됩니다. [서브에이전트는 자체 시스템 프롬프트 실행](/docs/en/sub-agents#what-loads-at-startup)을 수행하므로 스타일이 서브에이전트의 응답 방식을 변경하지 않습니다. [포크(fork)](/docs/en/sub-agents#fork-the-current-conversation)는 부모의 전체 시스템 프롬프트를 상속하므로 예외입니다.

토큰 사용량은 스타일에 따라 다릅니다. 시스템 프롬프트에 지침을 추가하면 입력 토큰이 증가하지만 프롬프트 캐싱을 통해 세션의 첫 번째 요청 이후에는 이 비용이 감소합니다. 내장된 Explanatory 및 Learning 스타일은 의도적으로 Default보다 긴 응답을 생성하므로 출력 토큰이 증가합니다. 커스텀 스타일의 경우 출력 토큰 사용량은 지침이 Claude에게 무엇을 생성하도록 지시하는지에 따라 달라집니다.

## 관련 기능과의 비교

여러 기능이 Claude Code의 동작을 맞춤 설정합니다. 출력 스타일은 시스템 프롬프트를 직접 수정하여 모든 응답에 적용됩니다. 다른 기능들은 기본 시스템 프롬프트를 변경하지 않고 지침을 추가하거나 특정 작업으로 범위를 한정합니다.

| 기능 | 작동 방식 | 사용해야 하는 경우 |
| :--- | :--- | :--- |
| 출력 스타일 | 시스템 프롬프트를 수정함 | 매 턴마다 다른 역할, 톤 또는 기본 응답 형식을 원할 때 |
| [CLAUDE.md](/docs/en/memory) | 시스템 프롬프트 뒤에 사용자 메시지를 추가함 | Claude가 항상 프로젝트 규칙과 코드베이스 컨텍스트를 알아야 할 때 |
| `--append-system-prompt` | 아무것도 제거하지 않고 시스템 프롬프트에 추가함 | 단일 호출에 대해 1회성 추가를 원할 때 |
| [에이전트](/docs/en/sub-agents) | 자체 시스템 프롬프트, 모델, 도구로 서브에이전트를 실행함 | 집중적인 작업을 위한 별도 범위의 헬퍼를 원할 때 |
| [스킬](/docs/en/skills) | 호출되거나 관련이 있을 때 작업 전용 지침을 로드함 | 재사용 가능한 워크플로가 있을 때 |

## 관련 리소스

* [설정](/docs/en/settings): `outputStyle` 필드가 위치하는 곳 및 설정 우선순위 작동 방식
* [권한 모드](/docs/en/permission-modes): Proactive 스타일과 자동 모드의 비교
* [플러그인](/docs/en/plugins): 스킬, 훅, 에이전트와 함께 출력 스타일을 패키징하고 배포
* [구성 디버깅](/docs/en/debug-your-config): 출력 스타일이 적용되지 않는 이유 진단
