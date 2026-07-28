> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 모델 구성

> `opusplan`과 같은 모델 별칭을 포함하여 Claude Code 모델 구성에 대해 알아봅니다.

## 사용 가능한 모델

Claude Code의 `model` 설정으로 다음 중 하나를 구성할 수 있습니다:

* **모델 별칭 (Model alias)**
* **모델 이름 (Model name)**
  * Anthropic API: 전체 **[모델 이름](https://platform.claude.com/docs/en/about-claude/models/overview)**
  * Amazon Bedrock: 추론 프로필 ARN
  * Microsoft Foundry: 배포 이름
  * Google Cloud's Agent Platform: 버전 이름

서로 다른 작업 유형에 적합한 모델과 노력 수준(effort level)에 대한 가이드는 블로그의 [Claude Code에서 Claude 모델 및 노력 수준 선택하기](https://claude.com/blog/claude-model-and-effort-level-in-claude-code)를 참조하세요.

<Note>
  `ANTHROPIC_BASE_URL`은 요청이 전송되는 위치를 변경하며, 이에 응답하는 모델을 변경하지는 않습니다. LLM 게이트웨이를 통해 Claude를 라우팅하려면 [LLM 게이트웨이](/docs/en/llm-gateway)를 참조하세요.
</Note>

### 모델 별칭

모델 별칭을 사용하면 정확한 버전 번호를 기억하지 않고도 편리하게 모델 설정을 선택할 수 있습니다:

| 모델 별칭        | 동작                                                                                                                                                                                                                                                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`default`**    | 모델 재정의를 지우고 계정 유형에 권장되는 모델로 되돌리거나, 관리자가 설정한 경우 [조직 기본 모델](#organization-default-model)로 되돌리는 특수 값. 자체로 모델 별칭은 아님                                                                                                                                                         |
| **`best`**       | 조직에 권한이 있는 경우 Fable 5를 사용하며, 그렇지 않은 경우 최신 Opus 모델을 사용                                                                                                                                                                                                                                                   |
| **`fable`**      | 가장 어렵고 오래 구동되는 태스크를 위해 Claude Fable 5를 사용                                                                                                                                                                                                                                                                         |
| **`sonnet`**     | 일상적인 코딩 태스크를 위해 최신 Sonnet 모델을 사용                                                                                                                                                                                                                                                                                  |
| **`opus`**       | 복잡한 추론 태스크를 위해 최신 Opus 모델을 사용                                                                                                                                                                                                                                                                                      |
| **`haiku`**      | 간단한 태스크를 위해 빠르고 효율적인 Haiku 모델을 사용                                                                                                                                                                                                                                                                               |
| **`sonnet[1m]`** | 긴 세션을 위해 [100만 토큰 컨텍스트 창](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-window-sizes-by-model)이 포함된 Sonnet을 사용. `sonnet`이 이미 기본 1M 창이 포함된 Sonnet 5로 해석되는 경우 아무런 효과가 없음; [LLM 게이트웨이](/docs/en/llm-gateway) 뒤에서는 Sonnet 5용 1M 창을 선택함 |
| **`opus[1m]`**   | 긴 세션을 위해 [100만 토큰 컨텍스트 창](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-window-sizes-by-model)이 포함된 Opus를 사용                                                                                                                                                                     |
| **`opusplan`**   | 계획 모드(plan mode)에서는 `opus`를 사용하고, 실행을 위해 `sonnet`으로 자동 전환되는 특수 모드                                                                                                                                                                                                                                      |

`opus` 및 `sonnet` 별칭이 해석되는 버전은 프로바이더에 따라 다릅니다:

| 프로바이더                                           | `opus`   | `sonnet`   |
| :--------------------------------------------------- | :------- | :--------- |
| Anthropic API                                        | Opus 4.8 | Sonnet 5   |
| [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws) | Opus 4.8 | Sonnet 4.6 |
| Amazon Bedrock, Google Cloud's Agent Platform        | Opus 4.8 | Sonnet 4.5 |
| Microsoft Foundry                                    | Opus 4.6 | Sonnet 4.5 |

별칭이 이전 모델로 해석되는 경우 명시적으로 전체 모델 이름을 선택하거나 `ANTHROPIC_DEFAULT_OPUS_MODEL` 또는 `ANTHROPIC_DEFAULT_SONNET_MODEL`을 설정하여 더 새로운 모델을 이용할 수 있습니다.

v2.1.207 이전에는 `opus`가 AWS 상의 Claude Platform에서 Opus 4.7로, Amazon Bedrock 및 Google Cloud's Agent Platform에서 Opus 4.6으로 해석되었었습니다.

별칭은 프로바이더에 권장되는 버전을 가리키며 시간이 지남에 따라 업데이트됩니다. 특정 버전에 고정(pin)하려면 `claude-opus-4-8`과 같이 전체 모델 이름을 사용하거나 `ANTHROPIC_DEFAULT_OPUS_MODEL`과 같이 해당 환경 변수를 설정하세요.

<Note>
  Sonnet 5는 Claude Code v2.1.197 이상이 필요합니다. Opus 4.8은 v2.1.154 이상이 필요합니다. 업그레이드하려면 `claude update`를 실행하세요.
</Note>

### Fable 5 활용하기

[Claude Fable 5](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5)는 Claude Code에서 가장 강력한 능력을 발휘하는 모델로, 단일 세션 이상의 대규모 작업에 적합합니다. 긴 자율 세션을 유지하고, 행동하기 전에 조사하며, 더 작은 모델보다 더 자주 작업을 검증합니다.

Fable 5는 기본 모델이 아닙니다. `/model fable`로 선택하세요. 안전 분류기(safety classifiers)가 플래그를 지정하는 요청(사이버 보안 및 생물학 분야에서 가장 흔함)은 [자동 모델 폴백](#automatic-model-fallback)을 트리거합니다.

Fable 5를 최대한 활용하려면:

* **단계가 아닌 결과를 설명하세요**: 원하는 결과를 전달하고 스스로 경로를 계획하게 만드세요. 해당 결과가 유지될 때까지 작업을 계속하도록 하려면 [목표를 설정](/docs/en/goal)하세요.
* **모호한 문제를 맡기세요**: 근본 원인 조사, 장애 디버깅, 아키텍처 결정 시 추가 조사 및 검증 능력이 빛을 발합니다.
* **검증 알림을 건너뛰세요**: 지침을 덜 주어도 스스로 작업을 검증하므로 테스트나 점검에 대한 상기 알림이 대부분 불필요합니다.
* **더 큰 태스크의 규모를 키우세요**: 평소라면 여러 조각으로 나누었을 작업을 부여하세요. 맥락을 잃지 않고 긴 세션을 유지합니다.

<Note>
  Fable 5는 Claude Code v2.1.170 이상이 필요합니다. 이전 버전은 모델 피커에 Fable 5를 표시하지 않으며 선택할 수도 없습니다. 업그레이드하려면 `claude update`를 실행하세요. Fable 5는 [제로 데이터 보존(zero data retention)](/docs/en/zero-data-retention) 환경에서는 사용할 수 없으며, 이 경우 `/model` 피커가 생략하거나 비활성화 상태로 표시합니다.
</Note>

Anthropic API에서 `/model` 피커는 서버가 조직에 Fable 5를 제공할 수 있다고 보고한 후에만 Fable 5를 나열합니다. `/model fable`을 직접 입력하면 Claude Code가 서버와 즉시 직접 가능 여부를 확인하므로 피커가 항목을 나열하기 전이라도 성공적으로 선택할 수 있습니다.

### 모델 설정하기

여러 방법으로 모델을 구성할 수 있으며 우선순위 순으로 나열하면 다음과 같습니다:

1. **세션 도중**: `/model <alias|name>`을 사용하여 즉시 전환하거나 인수 없이 `/model`을 실행하여 피커를 엽니다. 다음 응답이 캐시된 컨텍스트 없이 전체 기록을 다시 읽으므로 대화에 이전 출력이 있는 경우 피커가 확인을 요청합니다.
2. **시작 시**: `claude --model <alias|name>`으로 실행
3. **환경 변수**: `ANTHROPIC_MODEL=<alias|name>` 설정
4. **설정**: `model` 필드를 사용하여 설정 파일에 영구 구성

v2.1.153부터 `/model`은 사용자 설정의 `model` 필드를 작성하여 선택 사항을 새 세션의 기본값으로 저장합니다. 피커 내부에서:

* `Enter`: 모델을 전환하고 기본값으로 저장
* `s`: 이번 세션에 한해서만 모델 전환

`/model <name>`을 직접 입력하면 `Enter`처럼 동작합니다. [비대화형 모드](/docs/en/headless)에서 `-p` 플래그로 `/model`을 통해 설정한 모델은 현재 세션에만 적용되며 기본값으로 저장되지 않습니다. 프로젝트 및 관리형 설정이 여전히 우선 적용되며 다음 실행 시 다시 적용됩니다. 사용자의 선택을 재정의하도록 관리자가 구성한 [조직 기본 모델](#organization-default-model) 역시 다음 실행 시 다시 적용됩니다.

v2.1.144부터 v2.1.152까지는 `/model`이 현재 세션에만 적용되었으며 피커의 `d`로 기본값을 저장했었습니다.

`--model` 플래그와 `ANTHROPIC_MODEL` 환경 변수는 해당 플래그나 변수로 시작한 세션에만 적용됩니다. 서로 다른 터미널에서 동시에 다른 모델을 구동하려면 `/model`로 전환하는 대신 각각 독자적인 `--model` 플래그로 실행하세요.

`/model` 피커의 가격은 Claude Code가 직접 또는 프록시 처리하는 [LLM 게이트웨이](/docs/en/llm-gateway)를 통해 Anthropic API와 통신할 때 표시되며, 각 행의 가격은 해당 행이 선택하는 모델의 가격입니다. Amazon Bedrock과 같은 [서드파티 프로바이더](/docs/en/third-party-integrations) 및 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)에서는 프로바이더나 게이트웨이가 지불 금액을 결정하므로 피커 행에 가격이 표시되지 않습니다. 가격은 표시용 레이블일 뿐이며, 행이 어떤 모델을 선택하는지나 프로바이더의 청구 금액에 영향을 주지 않습니다. v2.1.206 이전에는 [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws) 및 게이트웨이 세션이 Anthropic 리스트 가격을 표시했으며, 행이 선택한 것과 다른 모델의 가격을 표시할 수도 있었습니다.

`claude --resume`, `--continue`, 또는 `/resume` 피커로 다시 시작한 세션은 현재 `model` 설정과 상관없이 트랜스크립트가 저장될 때 사용하던 모델을 유지합니다. 복원된 모델이 사용 중단되었거나 [`availableModels`](#restrict-model-selection)에 의해 제외된 경우 세션은 일반적인 우선순위 순서로 돌아갑니다. 이를 통해 다른 세션의 `/model` 선택이 재개 시 모델을 변경하지 않도록 방지합니다. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 등 Anthropic 모델 ID 대신 프로바이더 고유 배포 ID를 사용하는 프로바이더에서는 트랜스크립트 모델이 복원되지 않으며 세션이 일반적인 우선순위 순서대로 모델을 결정합니다.

`--model` 또는 `ANTHROPIC_MODEL`을 통해 새 실행을 위해 선택한 모델은 복원된 모델보다 여전히 우선 적용됩니다. v2.1.195부터 [`ANTHROPIC_DEFAULT_OPUS_MODEL`](#environment-variables) 계열 변수도 마찬가지로 우선 적용됩니다.

시작 시 활성화된 모델이 사용자의 직접 선택이 아닌 프로젝트나 관리형 설정에서 온 경우 시작 헤더에 해당 설정 파일이 표시됩니다. 재정의하려면 `/model`을 실행하세요; 프로젝트나 관리형 설정은 다음 실행 시 다시 적용됩니다.

[Agent SDK](/docs/en/agent-sdk/overview) `setModel()` 메서드나 Claude Code CLI를 실행하는 [Desktop 앱](/docs/en/desktop)과 같은 앱을 통해 모델 전환이 요청되면 Claude Code는 해당 문자열을 저장하기 전에 인식할 수 있는 값인지 확인합니다. 이 검사는 Claude Code v2.1.200 이상이 필요합니다. Anthropic API에서 Claude Code가 인식하는 대상:

* 모델 별칭
* `/model` 피커의 항목
* `claude-`로 시작하는 모든 이름
* [커스텀 모델 옵션](#add-a-custom-model-option)으로 직접 구성하거나 [`modelOverrides`](#override-model-ids-per-version)에 작성한 값

Claude Code는 인식할 수 없는 문자열을 `Model "<name>" is not a recognized model id.`와 함께 거부하고 세션은 문자열을 저장하고 다음 요청에서 실패하는 대신 현재 모델을 유지합니다. 복구 단계는 [오류 참조](/docs/en/errors#model-is-not-a-recognized-model-id)를 참조하세요.

이 검사는 Anthropic API에서만 실행됩니다. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws), 그리고 [LLM 게이트웨이](/docs/en/llm-gateway)나 커스텀 `ANTHROPIC_BASE_URL` 뒤에서는 사용자의 프로바이더나 게이트웨이가 모델 이름을 정의하므로 Claude Code가 검사 없이 모든 문자열을 전달합니다. 이 검사는 `--model` 플래그, `ANTHROPIC_MODEL` 환경 변수, 또는 `model` 설정에는 적용되지 않습니다; 거기에 잘못 입력된 값은 대신 첫 요청 시 [선택한 모델에 문제가 있음](/docs/en/errors#theres-an-issue-with-the-selected-model) 오류를 발생시킵니다.

요청한 모델에 퇴출 예정 일자가 있거나 더 새로운 버전으로 자동 매핑되는 경우 Claude Code는 요청한 모델의 이름을 명시하는 경고를 표시합니다. 대화형 세션에서는 시작 안내로 나타납니다. v2.1.182부터는 기본 텍스트 출력 형식을 사용할 때 [비대화형 모드](/docs/en/headless)에서도 동일한 경고가 stderr에 작성됩니다. 이 검사는 [subagent 프론트매터](/docs/en/sub-agents)에 설정된 `model`도 다룹니다. `--output-format json` 및 `stream-json`에서는 stderr 경고가 억제됩니다; 대신 [결과 메시지](/docs/en/headless#get-structured-output)의 `modelUsage` 필드에서 실제 모델을 읽으세요.

예를 들어 Opus로 세션을 시작합니다:

```bash theme={null}
claude --model opus
```

그런 다음 세션 내부에서 모델을 전환합니다:

```text theme={null}
/model sonnet
```

설정 파일 예시:

```json theme={null}
{
    "permissions": {
        "allow": ["Bash(npm run lint)"]
    },
    "model": "opus"
}
```

## 모델 선택 제한하기

기업 관리자는 [관리형 또는 정책 설정](/docs/en/settings#settings-files)에서 `availableModels`를 사용하여 사용자가 선택할 수 있는 모델을 제한할 수 있습니다. 항목은 `sonnet`과 같은 모델 계열, `claude-sonnet-4-5`와 같은 버전 접두사, 또는 `claude-sonnet-4-5-20250929`와 같은 전체 모델 ID와 매칭됩니다.

`availableModels`가 설정되면 사용자가 모델을 지정할 수 있는 모든 위치에 허용 목록(allowlist)이 적용됩니다:

* **메인 세션 모델**: `/model`, `--model` 플래그, `ANTHROPIC_MODEL` 환경 변수, `model` 설정, [세션 재개 시](#setting-your-model) 복원된 모델
* **별칭 해석**: `ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`, `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `ANTHROPIC_DEFAULT_FABLE_MODEL` 환경 변수가 허용된 별칭을 목록 외의 모델로 리다이렉트할 수 없음
* **빠른 모드**: `/fast`가 목록 외의 Opus 모델로 암시적으로 전환하려 할 때 "is not in your organization's allowed models" 메시지와 함께 전환을 거부함
* **Subagent 모델**: [subagent](/docs/en/sub-agents#choose-a-model) 프론트매터의 `model` 필드, Agent 도구의 `model` 매개변수, `CLAUDE_CODE_SUBAGENT_MODEL`, 그리고 v2.1.197 및 이전 버전에서는 `/agents` 마법사의 모델 피커
* **스킬 및 명령 모델**: [스킬 및 명령](/docs/en/skills)의 `model` 프론트매터
* **어드바이저 모델**: 구성된 [`advisorModel`](/docs/en/advisor) 설정 및 `--advisor` 플래그
* **백그라운드 에이전트 모델**: [디스패치 피커](/docs/en/agent-view)에서 선택된 모델

Anthropic API 및 [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)에서 모델 계열 별칭(`opus`, `sonnet`, `haiku`, `fable`)은 허용 목록이 허가하는 해당 계열의 가장 최신 버전으로 해석됩니다. 허용 목록이 특정 버전을 고정하는 경우(예: `["sonnet", "claude-opus-4-6"]`), `/model opus` 및 `--model opus`는 모두 허가된 가장 최신 Opus인 Claude Opus 4.6을 선택하고 요청된 모델과 대체된 모델을 모두 명시하는 안내를 표시합니다. v2.1.205 이전에는 최신 릴리스 버전이 목록 외부에 있는 별칭이 다른 차단된 선택처럼 거부되거나 교체되었었습니다.

Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, 및 [Mantle](/docs/en/amazon-bedrock#use-the-mantle-endpoint)은 Anthropic 모델 ID 대신 프로바이더 고유 배포 ID를 사용하므로, 거기서 차단된 별칭은 아래의 거부 및 교체 동작을 따릅니다.

Claude Code는 모델이 설정된 위치에 따라 여타 차단된 선택들을 다음과 같이 처리합니다:

* **`/model`**: 전환이 오류와 함께 거부됨
* **`--model` 플래그, `ANTHROPIC_MODEL`, 또는 `model` 설정**: 시작 시 요청된 모델과 대체된 모델을 모두 명시하는 경고와 함께 값이 교체되며 세션이 기본 모델로 시작됨
* **Subagent, 스킬, 또는 명령 재정의**: 요청이 실패하는 대신 상속받은 모델이나 기본 모델로 폴백됨
* **`advisorModel` 설정**: 세션에 대해 어드바이저가 비활성화됨
* **`--advisor` 플래그**: 시작 시 Claude Code가 오류와 함께 종료됨

제외된 모델은 `/model` 피커에서 숨겨집니다. 목록에 들어 있는 전체 모델 ID 중 내장 피커 행이 없는 ID(목록이 고정하는 이전 버전 등)는 `/model` 피커에 자체 레이블 행으로 표시됩니다. v2.1.199 이전에는 이러한 ID를 `/model <id>`를 직접 입력해야만 선택할 수 있었습니다.

Claude Code가 사용자를 대신해 수행하는 모델 변경사항도 동일한 방식으로 검사됩니다:

* **[폴백 모델 체인](#fallback-model-chains)**: 허용 목록 외의 항목이 제거됨
* **계획 모드 업그레이드**: Anthropic API 및 AWS 상의 Claude Platform에서 제외된 모델로의 업그레이드(예: [`opusplan`](#opusplan-model-setting))는 업그레이드 계열 중 허가된 가장 최신 버전을 사용합니다. 프로바이더 고유 모델 ID를 사용하는 프로바이더와 허가된 버전이 전혀 없는 경우 업그레이드를 건너뛰고 세션의 모델로 계획을 계속합니다.
* **[자동 모델 폴백](#automatic-model-fallback)**: 타깃이 제외된 폴백은 실행되지 않으므로 플래그 지정된 요청이 거절과 함께 종료됨
* **[Auto 모드 분류기](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)**: 분류기의 기본값인 Claude Sonnet 5는 허용 목록이 Sonnet 5를 허가할 때만 적용됩니다. 제외된 경우 허용 목록이 이미 통제 중인 세션의 모델로 실행되거나, 세션이 [Fable 5](#work-with-fable-5)로 실행 중일 때 Opus 모델로 실행됩니다. Anthropic API 이외의 프로바이더에서는 해당 Opus 폴백이 허용 목록 참조 없이 프로바이더의 기본 Opus 모델로 실행됩니다. Claude Code v2.1.210 이상이 필요합니다.
* **[빠른 모드](/docs/en/fast-mode)**: 세션이 이후 구동할 모델이 허용 목록 외부에 있을 때 빠른 모드 활성화가 거부됨

```json theme={null}
{
  "availableModels": ["sonnet", "haiku"]
}
```

### 서페이스 적용 범위

모든 서페이스는 자신이 수신한 허용 목록을 강제 적용합니다. 각 서페이스에 전달되는 전달 메커니즘은 다음과 같습니다:

| 전달 메커니즘                                                                 | CLI 및 IDE | Desktop 로컬 세션 | 웹, 모바일, 클라우드 세션 | Agent SDK 및 비대화형 | Cowork                  |
| :---------------------------------------------------------------------------- | :--------- | :---------------- | :------------------------ | :-------------------- | :---------------------- |
| 관리자 콘솔의 [서버 관리형 설정](/docs/en/server-managed-settings)            | 적용됨     | 적용됨            | 적용됨                    | 적용됨                | 전달 안 됨              |
| [MDM 또는 관리형 설정 파일](/docs/en/settings#settings-files)                 | 적용됨     | 적용됨            | 전달 안 됨                | 적용됨                | 배포된 경우 적용됨      |

* [Claude Code on the web](/docs/en/claude-code-on-the-web) 또는 Desktop 앱의 클라우드 세션은 Anthropic 관리형 VM에서 구동됩니다: 기기에 배포된 설정은 도달하지 않으므로 서버 관리형 설정을 통해 허용 목록을 전달하세요. 요청된 모델이 허용 목록에 의해 제외되면 클라우드 세션의 세션 중간 모델 전환이 거부됩니다. 세션 생성 시 서버 측 거부는 `availableModels` 설정 키가 아닌 [조직 모델 제한](#organization-model-restrictions)에 적용됩니다.
* Claude Desktop 앱의 에이전틱 작업 탭인 Cowork는 Claude Code 서페이스가 아니며 설계상 서버 관리형 설정을 수신하지 않습니다. 관리형 설정 파일은 세션이 구동되는 위치에 파일이 존재할 때 Cowork 세션에 적용됩니다; 원격 Cowork 세션은 Anthropic 관리형 VM에서 구동되므로 기기에 배포된 파일이 존재하지 않습니다.
* Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)과 같은 [서드파티 프로바이더](/docs/en/server-managed-settings#platform-availability) 세션은 서버 관리형 설정을 수신하지 않으므로 거기서는 MDM이나 관리형 설정 파일을 통해 허용 목록을 전달하세요.
* 서버 관리형 전달에는 세션이 조직 로그인이나 직접 구성된 API 키로 인증받는 것도 필요합니다. [`apiKeyHelper`](/docs/en/settings#available-settings) 스크립트를 통해서만 키를 생성하는 플릿은 MDM이나 관리형 설정 파일을 통해 허용 목록을 전달해야 합니다.
* Desktop Code 탭은 [SSH 세션](/docs/en/desktop#ssh-sessions)도 호스팅하며, 이는 구동되는 원격 호스트에서 관리형 설정 파일을 읽습니다. [Desktop 관리형 설정](/docs/en/desktop#managed-settings)을 참조하세요.
* claude.ai 및 Desktop 앱의 모델 피커는 조직의 허용 목록에 의해 제외된 모델을 숨기거나 비활성화 처리합니다. 피커 상태는 사용자를 위한 편의 기능이며; 강제 적용은 세션 내부에서 일어납니다.

### 기본(Default) 모델 동작

[`enforceAvailableModels`](#enforce-the-allowlist-for-the-default-model)도 설정되어 있지 않은 한 모델 피커의 Default 옵션은 `availableModels`의 영향을 받지 않습니다. 단독으로 사용할 때 `availableModels`는 Default를 이용 가능 상태로 두어 계정의 시스템 [런타임 기본값](#default-model-setting)으로 해석되도록 합니다. 해당 기본값이 제한하려는 모델인 경우 `enforceAvailableModels`도 함께 설정하세요.

빈 `availableModels` 배열은 Default 모델 강제 적용을 작동시키지 않습니다: `availableModels: []` 상태에서는 지정된 모델 선택은 차단되지만 계정 유형에 대한 Default 모델은 `enforceAvailableModels`와 상관없이 계속 사용할 수 있습니다.

### Default 모델에 대한 허용 목록 강제 적용

관리형 설정에서 비어 있지 않은 `availableModels`와 함께 `enforceAvailableModels: true`를 설정하여 Default 옵션으로 허용 목록을 확장하세요. Claude Code v2.1.175 이상이 필요합니다.

```json theme={null}
{
  "availableModels": ["sonnet", "haiku"],
  "enforceAvailableModels": true
}
```

Default 옵션은 계정 유형 기본값으로 해석되거나, 관리자가 설정한 경우 [조직 기본 모델](#organization-default-model)로 해석됩니다. 해당 모델이 허용 목록에 없으면 Default 옵션은 허용되고 사용 가능한 모델을 지정하는 첫 번째 `availableModels` 항목으로 해석되며, `/model` 피커의 Default 행에 해당 모델이 표시됩니다. 세션 시작, `/model`에서 Default 선택, [폴백 모델 체인](#fallback-model-chains)의 `"default"` 키워드, 제외된 선택이 삭제될 때 사용되는 폴백 등 기본값에 접근하는 모든 위치에 이것이 적용됩니다.

`availableModels`가 설정되지 않았거나 비어 있을 때는 `enforceAvailableModels`가 아무런 효과가 없습니다: `availableModels: []` 상태에서는 계정 유형의 Default 모델을 계속 사용할 수 있으므로 이 설정으로 인해 사용자가 모든 모델에서 튕겨 나가지는 않습니다. `availableModels`가 비어 있지 않지만 어떠한 항목도 허용되고 사용 가능한 모델로 해석되지 않으면 강제 적용을 건너뛰고 Default가 계정 유형 기본값으로 해석되며 경고는 `--debug` 아래에서만 볼 수 있습니다. 이를 방지하려면 최소 하나 이상의 확실히 사용 가능한 항목을 목록에 유지하세요.

두 키를 모두 [가장 높은 우선순위의 관리형 소스](/docs/en/settings#settings-precedence)에 배포하세요: 관리자가 배포한 관리형 소스는 병합되지 않으므로 서버 관리형 설정이 어떠한 설정이든 전달할 때 관리형 설정 파일에 위치시킨 쌍은 무시됩니다.

### 사용자가 구동하는 모델 제어하기

`model` 설정은 초기 선택 사항일 뿐 강제 적용이 아닙니다. 세션이 시작될 때 활성화되는 모델을 설정하지만 사용자는 여전히 `/model`을 열어 Default를 선택할 수 있으며, 이는 [`enforceAvailableModels`](#enforce-the-allowlist-for-the-default-model)가 이를 리다이렉트하지 않는 한 `model`에 설정된 값과 무관하게 시스템의 [런타임 기본값](#default-model-setting)으로 해석됩니다.

모델 경험을 완전히 통제하려면 다음 설정들을 조합하세요:

* **`availableModels`**: 사용자가 전환할 수 있는 모델 지정 제한
* **`enforceAvailableModels`**: Default 옵션으로 `availableModels` 허용 목록을 확장하여 Default가 목록 외의 모델로 해석되지 않도록 함
* **`model`**: 세션 시작 시 초기 모델 선택 설정
* **`ANTHROPIC_DEFAULT_SONNET_MODEL`** / **`ANTHROPIC_DEFAULT_OPUS_MODEL`** / **`ANTHROPIC_DEFAULT_HAIKU_MODEL`** / **`ANTHROPIC_DEFAULT_FABLE_MODEL`**: Default 옵션과 `sonnet`, `opus`, `haiku`, `fable` 별칭이 해석되는 대상을 제어

다음 예시는 사용자가 Sonnet 4.5에서 시작하도록 하고 피커를 Sonnet과 Haiku로 제한하며, Default가 티어 기본값 대신 허용 목록의 모델로 해석되도록 보장합니다:

```json theme={null}
{
  "model": "claude-sonnet-4-5",
  "availableModels": ["claude-sonnet-4-5", "haiku"],
  "enforceAvailableModels": true,
  "env": {
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "claude-sonnet-4-5"
  }
}
```

`enforceAvailableModels`나 `env` 블록이 없다면 피커에서 Default를 선택한 사용자가 해당 티어의 최신 릴리스를 받게 되어 `model` 및 `availableModels`에 지정한 고정 버전을 우회하게 됩니다. 두 설정은 서로 다른 범위를 다룹니다: `enforceAvailableModels`는 Default가 허용 목록을 따르도록 만들고, `env` 블록은 `sonnet`과 같은 허용된 별칭이 해석되는 특정 버전을 고정합니다. 모델 계열을 제한하는 것으로 충분할 때는 `enforceAvailableModels`를 단독으로 사용하고; 특정 버전까지 고정해야 할 때는 `env` 블록을 추가하세요.

### 병합 동작

[가장 높은 우선순위의 관리형 설정 소스](/docs/en/server-managed-settings#settings-precedence)가 `availableModels`를 정의하면 해당 목록만 적용됩니다: 사용자, 프로젝트, 또는 로컬 설정의 항목이 이를 확장할 수 없으며, 관리자가 배포한 관리형 소스는 서로 병합되지 않으므로 서버 관리형 설정이 어떠한 키든 전달할 때 관리형 설정 파일에 배포된 목록은 무시됩니다. 그렇지 않은 경우 사용자, 프로젝트, 로컬 설정의 목록은 여타 배열 설정처럼 [연결 및 중복 제거](/docs/en/settings#settings-precedence)됩니다. Claude Code v2.1.175부터는 관리형 목록이 더 낮은 우선순위의 항목을 대체하며; 이전 버전은 이들을 병합했습니다.

효용 목록 내부에서 특정 모델을 명시하는 항목(버전 접두사 또는 전체 모델 ID)은 해당 계열의 와일드카드 항목을 비활성화합니다: `["sonnet", "claude-sonnet-4-5"]`는 모든 Sonnet 모델이 아닌 Sonnet 4.5 버전만 허용합니다.

### Mantle 모델 ID

[Amazon Bedrock Mantle 엔드포인트](/docs/en/amazon-bedrock#use-the-mantle-endpoint)가 활성화되어 있을 때 `anthropic.`으로 시작하는 `availableModels` 항목이 커스텀 옵션으로 `/model` 피커에 추가되어 Mantle 엔드포인트로 라우팅됩니다. 이는 [서드파티 배포를 위한 모델 고정](#pin-models-for-third-party-deployments)에 설명된 별칭 매칭의 예외 사항입니다. 설정은 피커를 나열된 항목으로 여전히 제한하며, Mantle ID는 계열 이름을 내포하므로 특정 항목으로 간주되어 해당 계열의 와일드카드를 비활성화합니다: 임의의 Mantle ID와 함께 선택 가능 상태로 유지하려는 버전 접두사나 전체 ID를 나열하세요. [병합 동작](#merge-behavior)을 참조하세요.

### 조직 모델 제한

Claude Enterprise 플랜의 조직 관리자는 claude.ai 관리자 콘솔에서 개별 모델을 비활성화하여 멤버가 구동할 수 있는 모델을 제한할 수 있습니다. 이 제한은 Claude Code가 인증될 때 계정의 권한과 함께 전달되며 설정의 `availableModels` 목록과 별개로 작동하고, 세션이 생성될 때 서버가 동일한 제한을 독립적으로 강제 적용합니다. Claude Code v2.1.187 이상이 필요합니다.

제한은 멤버가 로그인하거나 자신의 API 키를 사용할 때 적용됩니다. 조직 서비스 키와 같이 조직 범위의 자격 증명은 사용자에 귀속되지 않으므로 제한이 적용되지 않습니다.

Claude Console에는 모델 제한 제어 기능이 없습니다. Anthropic API를 통해 멤버가 인증받는 조직을 포함하여 Claude Enterprise 플랜이 없는 조직은 대신 [관리형 설정](/docs/en/settings#settings-files)의 [`availableModels`](#restrict-model-selection)로 모델을 제한하며, Default 옵션을 다루기 위해 [`enforceAvailableModels`](#enforce-the-allowlist-for-the-default-model)를 추가합니다. 이 설정들은 서버가 아닌 Claude Code 자체에 의해 강제 적용됩니다.

제한된 모델은 `/model` 피커에서 숨겨집니다. `--model`, `ANTHROPIC_MODEL` 환경 변수, 또는 `model` 설정으로 이름을 지정하여 선택하면 `Model "<name>" is restricted by your organization's settings. Using <model> instead.` 안내가 표시되고 세션이 허용된 모델로 시작합니다. 제한된 모델에 대해 `/model <name>`을 입력하면 `Model '<name>' is restricted by your organization's settings. Run /model to choose a different model.`과 함께 거부되고 세션은 현재 모델을 유지합니다.

`opus`와 같은 [모델 계열 별칭](#restrict-model-selection)은 조직이 허가하는 해당 계열의 가장 최신 버전으로 해석되며 동일한 대체 안내가 나타납니다. `/model <alias>`는 해당 계열의 모든 버전이 제한될 때만 거부되며; `--model`, `ANTHROPIC_MODEL`, 또는 `model` 설정으로 설정된 별칭은 해당 경우에도 시작 시 여전히 교체됩니다. v2.1.205 이전에는 이전 버전이 허용되었더라도 최신 릴리스 버전만을 기준으로 계열 별칭이 대체되거나 거부되었었습니다.

제한은 조직 전체 또는 역할별로 적용됩니다:

* 조직 수준에서 모델을 비활성화하면 모든 멤버에 대해 모델이 제거됩니다.
* 역할 수준 접근 권한은 서로 다른 커스텀 역할에 서로 다른 모델을 부여하며, 여러 역할을 가진 멤버는 자신의 역할 중 하나라도 부여한 모델을 사용할 수 있습니다.
* Haiku 모델은 항상 사용 가능하며 비활성화할 수 없으므로 모든 멤버가 최소 하나 이상의 사용 가능한 모델을 유지합니다.
* 접근 권한 변경사항은 약 1분 이내에 새 요청에 적용되며; `/model` 피커는 다음에 세션이 시작될 때 이를 반영합니다.

두 제한이 함께 적용됩니다: 모델은 `availableModels`에 의해 허가되고 조직에 의해 제한되지 않을 때만 선택 가능합니다. 조직 제한은 Anthropic API 및 [LLM 게이트웨이](/docs/en/llm-gateway) 배포 세션에 전달됩니다. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, AWS 상의 Claude Platform 세션은 이를 수신하지 않으므로 해당 프로바이더에서는 대신 `availableModels`를 사용하세요.

## 조직 기본 모델

Claude Enterprise 플랜의 조직 관리자는 조직 전체 또는 커스텀 역할별로 claude.ai 관리자 콘솔에서 Claude Code 멤버를 위한 기본 모델을 설정할 수 있습니다. 설정되면 Default 옵션이 [계정 유형 기본값](#default-model-setting) 대신 해당 모델로 해석됩니다. Claude Code v2.1.196 이상이 필요합니다.

`/model` 피커의 Default 행은 Org default 레이아웃과 함께 조직 기본 모델의 이름을 표시합니다. 관리자가 조직 전체에 기본값을 설정했든 역할에 설정했든 레이블은 Org default로 표시됩니다. 역할 기본값은 해당 커스텀 역할의 멤버에게 적용되며 조직 전체 기본값보다 우선 적용됩니다; 여러 역할이 서로 다른 기본값을 설정하는 경우 가장 강력한 성능의 모델이 적용됩니다.

조직 기본값은 출발점일 뿐 제한이 아니며, 여타 다른 모델 선택이 이보다 우선 적용됩니다:

* `--model` 플래그 및 `ANTHROPIC_MODEL` 환경 변수
* [관리형 설정](/docs/en/settings#settings-files) 또는 `--settings`를 통해 제공되는 `model` 값
* `/model`로 저장한 모델을 포함하여 사용자, 프로젝트, 로컬 설정의 `model` 값

관리자는 사용자 선택을 재정의하도록 조직 기본값을 구성할 수도 있습니다. 재정의가 켜져 있으면 사용자, 프로젝트, 로컬 설정의 `model` 값보다 우선 적용되므로 `/model`로 저장한 모델이 현재 세션에 적용되고 다음 실행 시 조직 기본값으로 되돌아갑니다. 선택이 다를 경우 `/model`에 `Your organization's default (<model>) applies on restart`가 표시됩니다. `--model` 플래그, `ANTHROPIC_MODEL`, 관리형 설정, `--settings`는 재정의가 켜져 있어도 여전히 우선 적용됩니다. 재정의 기능은 제한된 조직 세트에 제공됩니다; 이용 가능 여부는 Anthropic 계정 팀에 문의하세요.

멤버가 선택할 수 있는 모델을 제한하려면 대신 [조직 모델 제한](#organization-model-restrictions) 또는 [`availableModels`](#restrict-model-selection)를 사용하세요.

Claude Code는 시작 시 조직 기본값을 한 번 읽으므로 세션 중간에 관리자가 변경한 기본값은 다음 실행 시 적용됩니다.

조직 기본값이 사용자 선택을 재정의하지 않을 때, 관리자가 이를 변경한 후의 첫 번째 대화형 실행은 사용자 설정에서 `model` 키를 한 번 제거하여 새 기본값이 적용되도록 합니다. 파일 내 다른 내용은 변경되지 않으며, 해당 실행 이후 `/model`로 저장한 모델은 유지됩니다.

조직 기본값은 채택되기 전에 다른 Default 모델과 동일한 제한 검사를 거칩니다:

* [`availableModels`](#restrict-model-selection) 자체는 Default 옵션을 구속하지 않으므로 허용 목록 외부의 조직 기본값도 여전히 적용됩니다. [`enforceAvailableModels`](#enforce-the-allowlist-for-the-default-model)도 설정되어 있을 때 허용 목록 외부의 조직 기본값은 다른 Default처럼 첫 번째 허용 목록 항목으로 재매핑됩니다
* [조직 모델 제한](#organization-model-restrictions)이 계정에 대해 거부하는 조직 기본값은 해당 계열의 허용된 가장 최신 모델로 대체되며, 모든 버전이 제한될 경우 더 저렴한 계열로 대체됩니다
* [제로 데이터 보존](/docs/en/zero-data-retention) 환경의 Fable 5처럼 계정에서 전혀 이용할 수 없는 조직 기본값은 건너뛰어지고 Default 옵션은 계정 유형 기본값으로 해석됩니다

v2.1.199부터 조직 기본값이 계정 유형의 일반적인 기본값과 다른 모델 계열일 때 `/model` 피커가 해당 일반 계열에 대한 별도 행을 유지하므로 세션 동안 해당 계열로 전환할 수 있습니다. v2.1.196부터 v2.1.198까지는 해당 행이 피커에 누락되어 있었습니다.

조직 기본값은 Anthropic API로 인증받은 세션에 전달됩니다. [LLM 게이트웨이](/docs/en/llm-gateway) 배포, Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, AWS 상의 Claude Platform 세션은 이를 수신하지 않습니다. 해당 배포에서 기본값을 설정하려면 대신 [관리형 설정](/docs/en/settings#settings-files)의 `model` 키를 사용하세요.

## 조직 노력 제한

Claude Enterprise 플랜의 조직 관리자는 역할 수준의 [조직 모델 제한](#organization-model-restrictions)과 더불어 각 커스텀 역할에 대해 모델별 최대 [노력 수준](#adjust-effort-level)을 설정할 수 있습니다. 상한선 위의 수준은 `/effort` 피커에서 제공되지 않으며 `--effort` 또는 `/effort`로 더 높은 수준을 지정하면 상한선 수준으로 구동됩니다. 대화형 세션 및 일반 텍스트 `--print` 실행에서는 요청된 수준과 적용된 수준을 명시하는 경고가 표시되며; `json`이나 `stream-json` 출력 또는 백그라운드 에이전트에서는 제한이 조용히 적용됩니다. 상한선은 모델별로 적용되므로 모델을 전환하면 사용 가능한 수준이 변경될 수 있습니다. 여러 역할이 동일한 모델을 부여하는 경우 가장 덜 제한적인 상한선이 적용됩니다. Claude Code v2.1.195 이상이 필요합니다.

노력 제한은 [조직 모델 제한](#organization-model-restrictions)과 함께 전달되며 동일한 프로바이더 제공 여부를 따릅니다: Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, AWS 상의 Claude Platform 세션은 이를 수신하지 않습니다.

## 특수 모델 동작

### `default` 모델 설정

`default` 동작은 계정 유형에 따라 다릅니다:

* **Max, Team Premium, Enterprise 사용량 기반 청구(pay-as-you-go), 및 Anthropic API**: 기본값 Opus 4.8
* **AWS 상의 Claude Platform, Amazon Bedrock, 및 Google Cloud's Agent Platform**: 기본값 Opus 4.8
* **Pro, Team Standard, 및 Enterprise 구독 시트**: 기본값 Sonnet 5
* **Microsoft Foundry**: 기본값 Sonnet 4.5

Enterprise 사용량 기반 청구는 구독 시트가 아닌 사용량에 따라 청구되는 Enterprise 조직을 의미합니다.

v2.1.207 이전에는 `default`가 AWS 상의 Claude Platform에서 Opus 4.7로, Amazon Bedrock 및 Google Cloud's Agent Platform에서 Sonnet 4.5로 해석되었었습니다.

관리자가 [조직 기본 모델](#organization-default-model)을 설정한 경우 `default`는 위의 계정 유형 기본값 대신 해당 모델로 해석됩니다. Claude Code v2.1.196 이상이 필요합니다.

관리형 설정이 [Default 모델에 대한 허용 목록을 강제 적용](#enforce-the-allowlist-for-the-default-model)하고 계정 유형 기본값이 `availableModels`에 없을 때 `default`는 위의 계정 유형 기본값 대신 강제 적용된 Default로 해석됩니다. 둘 다 적용되는 경우 조직 기본 모델이 계정 유형 기본값을 먼저 대체하고 강제 적용이 그에 적용됩니다: 허용 목록에 있는 조직 기본 모델은 유지되는 반면 목록 외부에 있는 모델은 강제 적용된 Default로 해석됩니다.

Fable 5는 어떠한 계정 유형에서도 기본 모델이 아닙니다. 세션은 사용자가 `/model fable`, `model` 설정, 또는 Fable 5를 이용할 수 있는 곳에서 `best` 별칭으로 선택한 경우에만 Fable 5를 사용합니다. `/model`로 선택하면 사용자 설정에 선택된 모델로 저장되므로 이후 세션은 사용자가 모델을 변경할 때까지 Fable 5에서 시작합니다.

### `opusplan` 모델 설정

`opusplan` 모델 별칭은 자동화된 하이브리드 접근 방식을 제공합니다:

* **계획 모드(plan mode) 내부**: 복잡한 추론 및 아키텍처 결정을 위해 `opus` 사용
* **실행 모드(execution mode) 내부**: 코드 생성 및 구현을 위해 `sonnet`으로 자동 전환

이는 계획을 위한 Opus의 추론 능력과 실행을 위한 Sonnet의 효율성을 결합합니다.

계획 모드의 Opus 단계는 `opus` 모델 설정과 동일한 컨텍스트 창을 사용합니다. Opus가 [자동으로 1M 컨텍스트로 업그레이드되는](#extended-context) 구독 티어에서는 `opusplan` 역시 계획 모드에서 업그레이드를 받습니다. 자동 업그레이드 티어가 아닐 때 두 단계 모두에 대해 1M 컨텍스트를 강제 적용하려면 모델을 `opusplan[1m]`으로 설정하세요.

[`availableModels`](#restrict-model-selection)가 최신 Opus를 제외하지만 이전 버전을 허가할 때(예: `["sonnet", "claude-opus-4-6"]`), `opusplan`은 계획을 위해 허가된 가장 최신 Opus를 사용하며 모든 Opus가 제외될 때만 Sonnet에 머무릅니다. 계획 모드에서 평소 Sonnet으로 업그레이드될 Haiku 세션도 마찬가지로 허가된 가장 최신 Sonnet을 사용하며 모든 Sonnet이 제외될 때만 Haiku에 머무릅니다. v2.1.205 이전에는 허용 목록이 이전 버전을 허가했더라도 업그레이드 계열의 최신 버전이 제외될 때마다 계획 모드가 세션의 모델에 머물렀었습니다.

이전의 허가된 버전을 대체 적용하는 것은 Anthropic API 및 [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)에 적용됩니다. 배포가 프로바이더 고유 모델 ID를 사용하는 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, Mantle에서는 업그레이드 모델이 제외될 때마다 계획 모드가 세션의 모델에 머뭅니다.

계획 경계에서 전환하지 않고 태스크 중간에 언제 두 번째 모델을 참조할지 Claude가 결정하도록 하는 하이브리드 접근 방식은 [어드바이저 도구](/docs/en/advisor)를 참조하세요.

### 폴백 모델 체인

기본 모델이 과부하 상태이거나, 사용할 수 없거나, 기타 재시도 불가능한 서버 오류를 반환할 때 Claude Code는 요청을 실패시키는 대신 폴백 모델로 전환할 수 있습니다. 인증, 청구, 비율 제한(rate-limit), 요청 크기, 트랜스포트 오류는 전환을 트리거하지 않으며; 정상적인 재시도 및 오류 처리를 따릅니다.

하나 이상의 폴백 모델을 구성하면 Claude Code가 이들을 순서대로 시도하며 전환 시 안내를 표시합니다. 전환은 현재 턴 동안만 유지되므로 다음 메시지는 기본 모델을 먼저 다시 시도합니다. 체인은 중복 제거 후 최대 3개 모델로 제한되며 초과 항목은 무시됩니다.

단일 세션에 대한 체인을 쉼표로 구분된 목록을 수용하는 `--fallback-model` 플래그로 설정합니다:

```bash theme={null}
claude --fallback-model sonnet,haiku
```

세션 전반에 걸쳐 체인을 유지하려면 [설정](/docs/en/settings)에서 `fallbackModel`을 배열로 설정합니다:

```json theme={null}
{
  "fallbackModel": ["claude-sonnet-5", "claude-haiku-4-5"]
}
```

`--fallback-model` 플래그는 `fallbackModel` 설정보다 우선 적용됩니다. 각 항목은 모델 이름이나 별칭을 수용하며 `"default"`는 기본 모델로 확장됩니다.

Claude Code는 시작 시 체인을 확인하지 않으며 `/status`도 이를 표시하지 않습니다. 전환이 일어날 때 표시되는 안내가 폴백이 구성되어 있음을 보여주는 첫 눈에 보이는 표시입니다.

요청이 장애 조치(failover)될 때 Claude Code는 수용하는 모델이 나타날 때까지 각 항목을 순서대로 시도합니다. 설정에 고정된 퇴출된 모델처럼 도달할 수 없는 항목은 동일한 방식으로 다음 항목으로 장애 조치됩니다. 이 순회가 시작되기 전에 두 가지 종류의 항목이 제거됩니다:

* **허용 목록 외부**: [`availableModels`](#restrict-model-selection)가 허용하지 않는 항목은 Claude Code가 체인을 읽을 때 삭제됩니다.
* **압축 중 더 작은 컨텍스트 창**: 체인은 [압축(compaction)](/docs/en/context-window#what-survives-compaction)도 다루지만, 거기서 요약하면 대화의 일부가 먼저 잘려 나가므로 Claude Code는 기본 모델보다 컨텍스트 창이 더 작은 모델로 폴백하지 않습니다. 모든 폴백 모델이 더 작다면 압축에 원본 오류가 표시되며 다시 시도할 수 있습니다.

### 자동 모델 폴백

이 섹션은 Fable 5로부터의 콘텐츠 기반 폴백을 다룹니다. 모델이 과부하 상태이거나 사용할 수 없을 때의 가용성 기반 폴백은 [폴백 모델 체인](#fallback-model-chains)을 참조하세요.

Fable 5는 사이버 보안 및 생물학 콘텐츠에 대한 안전 분류기와 함께 구동됩니다. 분류기가 요청에 플래그를 지정하면 Claude Code는 해당 요청을 프로바이더의 기본 Opus 모델로 다시 실행하고 트랜스크립트에 안내를 표시합니다. Anthropic API, [LLM 게이트웨이](/docs/en/llm-gateway) 배포, [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)에서 해당 모델은 Opus 4.8입니다. [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)에서는 [`opus` 별칭](#environment-variables)을 다른 모델로 가리키지 않는 한 Opus 4.7입니다.

세션은 해당 Opus 모델에서 계속 진행됩니다. Fable 5로 돌아가려면 `/model fable`을 실행하세요.

폴백 타깃은 [`availableModels`](#restrict-model-selection)에 대해 검사됩니다. 차단된 경우 폴백이 일어나지 않습니다. 거절이 일반 오류로 표시되며 세션의 모델은 변경되지 않습니다.

#### 폴백을 트리거한 요인 확인

첫 번째 요청이 CLAUDE.md 내용 및 git 상태와 같은 작업 공간 컨텍스트를 전달하므로 사용자가 이상한 내용을 전송하기 전이라도 세션의 첫 요청에서 폴백이 트리거될 수 있습니다. 보안이나 생물학 자료가 포함된 저장소는 해당 컨텍스트만으로 분류기를 작동시킬 수 있습니다.

커스텀 설정이 원인인지 확인하려면 CLAUDE.md, 스킬, MCP 서버, 훅과 같은 커스텀 설정을 비활성화하는 `claude --safe-mode`로 세션을 시작하세요. Git 상태 및 디렉토리 이름은 커스텀 설정이 아니므로 여전히 포함됩니다.

#### 전환 전 확인하기

자동으로 전환하는 대신 요청에 플래그가 지정될 때마다 어떻게 할지 결정하려면 `/config`를 실행하고 "switch models when a message is flagged"를 끄세요. 플래그 지정된 요청은 세션을 일시 정지하고 두 가지 옵션을 제공합니다: Opus 모델로 전환하거나, 프롬프트를 편집하고 Fable 5에서 다시 시도.

일부 모드는 다르게 동작합니다:

* 두 모델 모두 동일한 요청에 플래그를 지정하면 프롬프트를 편집하여 다시 시도하거나 새 세션을 시작할 수 있습니다.
* 모바일 [Claude Code on the web](/docs/en/claude-code-on-the-web) 세션에서는 편집 및 재시도가 지원되지 않습니다. 모델을 전환하거나 데스크톱 브라우저 또는 데스크톱 앱에서 세션을 계속하세요.
* 프롬프트를 표시할 수 없는 [비대화형 모드](/docs/en/cli-reference#cli-flags) 및 SDK 통합에서는 플래그 지정된 요청이 거절과 함께 턴을 종료합니다.
* 폴백 타깃이 [`availableModels`](#restrict-model-selection)에 의해 차단되면 프롬프트가 표시되지 않습니다. 플래그 지정된 요청은 타깃이 차단되었을 때의 자동 폴백과 동일하게 거절과 함께 종료됩니다.

#### Bedrock, Agent Platform, Foundry에서 폴백 활성화

[Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry)에서 모델 ID는 프로바이더 고유의 것이므로, Claude Code가 관련된 두 모델을 모두 식별할 수 있을 때만 자동 폴백이 작동합니다:

* Claude Code가 현재 모델을 Fable 5로 인식해야 함: 모델 ID가 `claude-fable-5`를 포함하거나, `ANTHROPIC_DEFAULT_FABLE_MODEL` 값과 일치하거나, [`modelOverrides`](#override-model-ids-per-version)로 매핑됨.
* 폴백 타깃이 Opus 모델로 해석되어야 함: 설정되어 있다면 `ANTHROPIC_DEFAULT_OPUS_MODEL` 값, 그렇지 않은 경우 프로바이더 모델 목록의 Opus 4.8 항목.

어느 모델이든 식별할 수 없으면 Claude Code가 자동으로 전환하지 않습니다. 플래그 지정된 요청이 거부 메시지와 함께 종료되며, [`/model`](#setting-your-model)로 모델을 전환하고 다시 시도할 수 있습니다. 이러한 프로바이더에서 자동 폴백을 활성화하려면 `ANTHROPIC_DEFAULT_FABLE_MODEL`을 본인의 Fable 5 모델 ID로 설정하고 `ANTHROPIC_DEFAULT_OPUS_MODEL`을 본인의 Opus 4.8 모델 ID로 설정하세요.

#### 보안 연구 및 생물학 워크로드

침투 테스트, Capture the Flag (CTF) 연습, 생물학 관련 코드베이스를 포함하여 공경적 보안이나 생물학 분야의 워크로드는 첫 번째 요청에서 매우 자주 폴백을 트리거합니다. 실질적인 생물학 작업의 경우 거의 모든 요청이 재라우팅될 것으로 예상해야 합니다.

이는 해당 도메인에 대한 예상된 라우팅이며 계정 플래그가 아닙니다. 조직에 이 작업을 위한 Fable급 성능이 필요한 경우 신뢰할 수 있는 접근(trusted access) 프로그램에 대해 Anthropic 계정 팀에 문의하세요.

### 노력 수준(effort level) 조정

[노력 수준](https://platform.claude.com/docs/en/build-with-claude/effort)은 적응형 추론(adaptive reasoning)을 제어하며, 이를 통해 모델이 태스크 복잡성에 따라 각 단계별로 생각할지 여부와 생각하는 양을 결정할 수 있습니다. 낮은 노력 수준은 직관적인 태스크에 대해 더 빠르고 저렴하며, 높은 노력 수준은 복잡한 문제에 대해 더 깊은 추론을 제공합니다.

사용 가능한 노력 수준은 모델에 따라 다릅니다. 여기에 나열되지 않은 모델은 노력을 지원하지 않습니다:

| 모델                             | 수준                                    |
| :------------------------------- | :-------------------------------------- |
| Fable 5                          | `low`, `medium`, `high`, `xhigh`, `max` |
| Sonnet 5, Opus 4.8, 및 Opus 4.7  | `low`, `medium`, `high`, `xhigh`, `max` |
| Opus 4.6 및 Sonnet 4.6           | `low`, `medium`, `high`, `max`          |

활성화된 모델이 지원하지 않는 수준을 설정하면 Claude Code는 설정한 수준 이하의 지원되는 가장 높은 수준으로 폴백합니다. 예를 들어 Opus 4.6에서 `xhigh`는 `high`로 실행됩니다. 조직에서 모델에 사용할 수 있는 수준의 상한선을 설정할 수도 있습니다; [조직 노력 제한](#organization-effort-limits)을 참조하세요.

기본 노력 수준은 Fable 5, Sonnet 5, Opus 4.8, Opus 4.6, Sonnet 4.6에서 `high`이며, Opus 4.7에서 `xhigh`입니다.

Fable 5, Opus 4.8, 또는 Opus 4.7을 처음 구동할 때 다른 모델에 대해 이전에 다른 수준을 설정했더라도 Claude Code는 해당 모델의 기본 노력 수준을 적용합니다: Fable 5 및 Opus 4.8에서 `high`, Opus 4.7에서 `xhigh`. 전환 후 다른 수준을 선택하려면 `/effort`를 다시 실행하세요. 해당 기본값은 대화형 세션에서 `/effort`를 실행하거나 `--effort`로 실행하는 등 명시적인 노력 선택을 할 때까지 세션 간에 유지됩니다.

대화형 세션에서 설정한 `low`, `medium`, `high`, `xhigh`는 세션 간에 유지됩니다. [비대화형 모드](/docs/en/headless)에서 `-p` 플래그로 `/effort`를 통해 설정한 수준은 현재 세션에만 적용되며 기본값으로 저장되지 않습니다. 비대화형 `/effort`는 위의 모델 기본값 유지 상태를 해제할 수도 없습니다: Fable 5, Opus 4.8, Opus 4.7에서 `Not applied`를 보고하고 세션은 모델의 기본 노력 수준을 유지하므로 시작 시 `--effort`를 전달하세요. `max`는 토큰 지출 제한 없이 가장 깊은 추론을 제공하며 `CLAUDE_CODE_EFFORT_LEVEL` 환경 변수를 통해 설정된 경우를 제외하고 현재 세션에만 적용됩니다.

`/effort` 메뉴는 `ultracode`도 제공합니다. Ultracode는 모델 노력 수준이라기보다 Claude Code 설정입니다: 모델에 `xhigh`를 전송하고 추가로 Claude가 실질적인 태스크에 대해 [동적 워크플로](/docs/en/workflows)를 오케스트레이션하도록 합니다. 현재 세션에만 적용됩니다.

다음 중 하나를 통해 ultracode를 켤 수 있습니다:

* **`/effort`**: `/effort ultracode`를 실행하거나 메뉴에서 선택
* **`--effort` 플래그**: `claude --effort ultracode`로 실행하여 ultracode가 켜진 `xhigh` 노력 수준으로 세션을 시작
* **`--settings` 또는 Agent SDK 제어 요청**: `"ultracode": true` 전달. [`applyFlagSettings()`](/docs/en/agent-sdk/typescript#applyflagsettings) 요청도 `effortLevel: "ultracode"`를 수용함

`--effort` 플래그나 Agent SDK `effortLevel` 값에 `ultracode`를 전달하려면 Claude Code v2.1.203 이상이 필요합니다. v2.1.203 이전에는 `--effort ultracode`가 `Unknown --effort value 'ultracode'`를 출력하고 세션이 기본 노력 수준으로 시작했었습니다.

유지 설정인 `effortLevel` 및 `CLAUDE_CODE_EFFORT_LEVEL` 환경 변수는 `ultracode`를 수용하지 않습니다. `CLAUDE_CODE_EFFORT_LEVEL`이 `xhigh` 이외의 수준으로 설정되어 있으면 요청이 해당 수준으로 구동되고 ultracode의 워크플로 오케스트레이션은 비활성 상태로 유지됩니다. 그때 ultracode를 선택하면 환경 변수가 세션의 노력 수준을 재정의한다는 경고가 표시됩니다.

[워크플로가 꺼져 있는](/docs/en/workflows#turn-workflows-off) 경우처럼 ultracode를 이용할 수 없을 때 `--effort ultracode`는 `xhigh` 노력 수준만 설정합니다.

#### 노력 수준 선택하기

각 수준은 토큰 지출과 성능 간의 균형을 맞춥니다. 기본값은 대부분의 코딩 태스크에 적합하며; 다른 균형을 원할 때 조정하세요.

| 수준        | 사용 시점                                                                                                                                       |
| :---------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| `low`       | 지능에 민감하지 않은 짧고 범위가 지목된 지연 시간 민감 태스크를 위해 예약                                                                     |
| `medium`    | 지능을 일부 희생할 수 있는 비용 민감 작업에 대해 토큰 사용량을 줄임                                                                             |
| `high`      | 토큰 사용량과 지능의 균형. Fable 5, Sonnet 5, Opus 4.8, Opus 4.6, Sonnet 4.6의 기본값                                                          |
| `xhigh`     | 더 높은 토큰 지출에서 더 깊은 추론. Opus 4.7의 기본값                                                                                           |
| `max`       | 요구사항이 가혹한 태스크에서 성능을 향상시킬 수 있으나 수확 체감 현상이 나타날 수 있고 지나친 생각(overthinking) 경향이 있음. 광범위 도입 전 테스트할 것 |
| `ultracode` | 메시지당 `xhigh` 추론과 함께 실질적인 각 태스크에 대해 [동적 워크플로](/docs/en/workflows)를 계획하는 Claude Code 설정. 세션 전용             |

노력 스케일은 모델별로 캘리브레이션되므로 동일한 수준 이름이 모델 간에 동일한 내부 값을 나타내지는 않습니다.

#### 일회성 깊은 추론을 위해 ultrathink 사용하기

세션 노력 설정을 변경하지 않고 해당 턴에 대해 더 깊은 추론을 요청하려면 프롬프트 임의의 위치에 `ultrathink`를 포함시키세요. Claude Code가 해당 키워드를 인식하고 인-컨텍스트(in-context) 지침을 추가합니다. API에 전송되는 노력 수준은 변경되지 않습니다. "think", "think hard", "think more"와 같은 다른 문구는 일반 프롬프트 텍스트로 전달되며 키워드로 인식되지 않습니다.

#### 노력 수준 설정하기

다음 중 하나를 통해 노력 수준을 변경할 수 있습니다:

* **`/effort`**: 대화형 슬라이더를 열려면 인수 없이 `/effort` 실행, 직접 설정하려면 뒤에 수준 이름을 붙여 `/effort` 실행, 모델 기본값으로 재설정하려면 `/effort auto` 실행
* **`/model` 내부**: 모델을 선택할 때 왼쪽/오른쪽 화살표 키를 사용하여 노력 슬라이더 조정
* **`--effort` 플래그**: Claude Code를 시작할 때 단일 세션에 대한 수준 이름을 전달하여 설정
* **환경 변수**: `CLAUDE_CODE_EFFORT_LEVEL`을 수준 이름이나 `auto`로 설정
* **설정**: 설정 파일에서 `effortLevel`을 `low`, `medium`, `high`, `xhigh`로 설정. `max`와 `ultracode`는 [세션 전용](#adjust-effort-level)이므로 여기서 수용되지 않음
* **스킬 및 subagent 프론트매터**: 해당 스킬이나 subagent가 실행될 때 노력 수준을 재정의하도록 [스킬](/docs/en/skills#frontmatter-reference) 또는 [subagent](/docs/en/sub-agents#supported-frontmatter-fields) 마크다운 파일에 `effort` 설정

환경 변수가 여타 다른 모든 방식보다 우선 적용되며, 그다음 사용자가 구성한 수준, 그다음 모델 기본값 순입니다. 프론트매터 노력 수준은 해당 스킬이나 subagent가 활성화되어 있을 때 적용되어 세션 수준을 재정의하지만 환경 변수는 재정의하지 못합니다.

[관리형 설정](/docs/en/settings#settings-precedence)의 `effortLevel` 키는 강제 적용이 아닌 시작 기본값입니다: 사용자가 `/effort`나 `--effort`로 세션에 대한 값을 변경할 수 있으며, 새로 시작하는 세션에서는 관리형 값이 기본값으로 다시 적용됩니다.

지원되는 모델이 선택되었을 때 `/model`에 노력 슬라이더가 표시됩니다. 현재 노력 수준은 모델 이름 옆의 세션 헤더에도 표시되므로(예: "with low effort"), `/model`을 열지 않고도 어떤 설정이 활성화되어 있는지 확인할 수 있습니다. 푸터 역시 시작 시 및 변경 시에 노력 수준을 짧게 보여줍니다.

#### 적응형 추론과 고정 사고 예산

적응형 추론(adaptive reasoning)은 각 단계별 사고를 선택 사항으로 만들어, Claude가 루틴 프롬프트에는 더 빠르게 응답하고 이득이 되는 단계에는 더 깊은 사고를 할당하도록 돕습니다. Claude가 현재 수준이 생성하는 것보다 더 자주 또는 덜 자주 생각하기를 원한다면 프롬프트나 `CLAUDE.md`에서 이를 직접 말할 수 있습니다; 모델은 노력 설정 범위 안에서 해당 지침에 응답합니다.

Fable 5, Sonnet 5, Opus 4.7 및 이후 버전은 항상 적응형 추론을 사용합니다. 고정 사고 예산 모드 및 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`은 여기에 적용되지 않습니다.

Opus 4.6 및 Sonnet 4.6에서는 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`을 설정하여 `MAX_THINKING_TOKENS`에 의해 통제되던 이전의 고정 사고 예산 모드로 되돌릴 수 있습니다. [환경 변수](/docs/en/env-vars)를 참조하세요.

### 확장 사고 (Extended thinking)

확장 사고는 Claude가 응답하기 전에 내보내는 추론 과정입니다. [적응형 추론](#adjust-effort-level)을 지원하는 모델에서는 노력 수준이 사고량에 대한 주된 제어 수단입니다; 아래 설정들은 사고 기능을 켜거나 끄고 표시 방식을 제어합니다.

| 제어                           | 설정 방법                                                                                                                                                                                                                                                                                                                                                                 |
| :----------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 현재 세션 토글                 | macOS에서 `Option+T`, Windows 및 Linux에서 `Alt+T` 누름                                                                                                                                                                                                                                                                                                                   |
| 전역 기본값 설정               | `/config`를 실행하고 thinking mode 토글. `~/.claude/settings.json`에 `alwaysThinkingEnabled`로 저장됨                                                                                                                                                                                                                                                                     |
| 노력과 상관없이 비활성화       | [`MAX_THINKING_TOKENS=0`](/docs/en/env-vars) 설정. Fable 5를 제외하고 Anthropic API에서 사고를 끔. [서드파티 프로바이더](/docs/en/third-party-integrations)에서는 `thinking` 매개변수가 생략되며 적응형 추론 모델은 여전히 생각할 수 있음. 다른 값들은 [고정 사고 예산](#adaptive-reasoning-and-fixed-thinking-budgets) 환경에서만 적용됨                               |

Fable 5에서는 사고 기능을 끌 수 없습니다. 세션 토글, `alwaysThinkingEnabled`, `MAX_THINKING_TOKENS=0`은 거기서 아무런 효과가 없으며, Fable 5는 노력 수준을 바탕으로 단계별로 얼마나 생각할지 스스로 결정합니다.

사고 출력은 기본적으로 접혀 있습니다. 회색 이탤릭체 텍스트로 추론을 확인하려면 `Ctrl+O`를 눌러 verbose 모드를 토글하세요. Anthropic API의 대화형 세션은 기본적으로 요약/가려진(redacted) 사고 블록을 받으므로, 펼쳤을 때 전체 요약을 확인하려면 [설정](/docs/en/settings)에서 `showThinkingSummaries: true`를 설정하세요. 접혀 있거나 가려져 있더라도 생성된 모든 사고 토큰에 대해 요금이 청구됩니다.

### 확장 컨텍스트 (Extended context)

Fable 5, Sonnet 5, Opus 4.6 및 이후 버전, Sonnet 4.6은 대규모 코드베이스와의 긴 세션을 위해 [100만 토큰 컨텍스트 창](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-window-sizes-by-model)을 지원합니다.

제공 여부는 모델 및 요금제에 따라 다릅니다. Anthropic API에서 Fable 5, Sonnet 5, Opus 4.8, Opus 4.7은 항상 1M 창으로 구동됩니다. Max, Team, Enterprise 요금제에서는 별도 구성 없이 Opus가 1M 컨텍스트로 자동 업그레이드됩니다. 이는 Team Standard 및 Team Premium 시트 모두에 적용됩니다. 1M 컨텍스트의 Sonnet 4.6은 자동 업그레이드의 일부가 아니며 Max를 포함한 모든 구독 플랜에서 [사용량 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)이 필요합니다.

| 요금제                    | 1M 컨텍스트 Opus                                                                                            | 1M 컨텍스트 Sonnet 4.6                                                                                      |
| ------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Max, Team, 및 Enterprise  | 구독에 포함됨                                                                                               | [사용량 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans) 필요    |
| Pro                       | [사용량 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans) 필요    | [사용량 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans) 필요    |
| API 및 사용량 기반 청구   | 전체 사용 가능                                                                                              | 전체 사용 가능                                                                                              |

1M 컨텍스트를 완전 비활성화하려면 `CLAUDE_CODE_DISABLE_1M_CONTEXT=1`을 설정하세요. 이는 모델 피커에서 1M 모델 변형을 제거합니다. [환경 변수](/docs/en/env-vars)를 참조하세요.

1M 컨텍스트 창은 200K 이상의 토큰에 할증 없이 표준 모델 가격 정책을 사용합니다. 구독에 확장 컨텍스트가 포함된 플랜의 경우 사용량이 구독 범위 내에 유지됩니다. 사용량 크레딧으로 확장 컨텍스트에 접근하는 플랜의 경우 토큰이 사용량 크레딧으로 청구됩니다.

계정이 1M 컨텍스트를 지원하는 경우 Claude Code 최신 버전의 `/model` 피커에 해당 옵션이 나타납니다. 보이지 않는 경우 세션을 다시 시작해 보세요.

모델 별칭이나 전체 모델 이름에 `[1m]` 접미사를 사용할 수도 있습니다:

```text theme={null}
# opus[1m] 또는 sonnet[1m] 별칭 사용
/model opus[1m]
/model sonnet[1m]

# 또는 전체 모델 이름에 [1m] 추가
/model claude-opus-4-8[1m]
```

#### Sonnet 5 컨텍스트 창

Anthropic API에서 Sonnet 5는 항상 1M 컨텍스트 창으로 구동됩니다. 200K 변형이 없으며, 선택할 `[1m]` 접미사도 없고, 어떠한 플랜에서도 사용량 크레딧이 필요하지 않습니다. 세션은 창이 차기 전(기본값 약 967K 토큰)에 자동 압축됩니다; 다른 임계값을 선택하려면 [`CLAUDE_CODE_AUTO_COMPACT_WINDOW`](/docs/en/env-vars)를 설정하세요.

두 가지 구성은 창 예산을 200K로 책정하고 해당 경계에서 자동 압축합니다:

* **LLM 게이트웨이**: `ANTHROPIC_BASE_URL`이 [게이트웨이](/docs/en/llm-gateway)를 가리킬 때 Claude Code는 1M 지원을 검증할 수 없습니다. 전체 창을 사용하려면 `sonnet[1m]`으로 매핑되는 모델 피커의 Sonnet 5 (1M context)를 선택하세요.
* **`CLAUDE_CODE_DISABLE_1M_CONTEXT=1`**: 컨텍스트 제한이 필요한 배포 환경을 위해 Sonnet 5 세션이 200K 창을 가진 것으로 취급합니다.

## 현재 모델 확인하기

다음 두 위치에서 현재 사용 중인 모델을 확인할 수 있습니다:

* 구성되어 있는 경우 [상태 줄(statusline)](/docs/en/statusline) 내부
* 계정 정보도 함께 표시해 주는 `/status` 내부

## 커스텀 모델 옵션 추가하기

`ANTHROPIC_CUSTOM_MODEL_OPTION`을 사용하여 내장 별칭을 대체하지 않고 단일 커스텀 항목을 `/model` 피커에 추가하세요. 이는 Claude Code가 기본으로 나열하지 않는 모델 ID를 테스트할 때 유용합니다. LLM 게이트웨이 배포의 경우 `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`이 설정되어 있으면 게이트웨이의 `/v1/models` 엔드포인트로부터 Claude Code가 피커를 채울 수 있으므로, 디스커버리가 비활성화되어 있거나 원하는 모델을 반환하지 않을 때만 이 변수가 필요합니다. [게이트웨이 모델 디스커버리](/docs/en/llm-gateway-protocol#model-discovery)를 참조하세요.

이 예시는 게이트웨이 라우팅된 Opus 배포를 선택 가능하게 만들기 위해 세 환경 변수를 모두 설정합니다. Claude Code는 시작 시 환경 변수를 읽으므로 `claude`를 실행하기 전에 export 명령을 구동하거나 기존 세션을 다시 시작하여 적용하세요:

```bash theme={null}
export ANTHROPIC_CUSTOM_MODEL_OPTION="my-gateway/claude-opus-4-8"
export ANTHROPIC_CUSTOM_MODEL_OPTION_NAME="Opus via Gateway"
export ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION="Custom deployment routed through the internal LLM gateway"
```

커스텀 항목은 `/model` 피커 하단에 나타납니다. `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME`과 `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION`은 선택 사항입니다. 생략 시 모델 ID가 이름으로 사용되며 설명은 `Custom model (<model-id>)`로 기본 설정됩니다.

Claude Code는 `ANTHROPIC_CUSTOM_MODEL_OPTION`에 설정된 모델 ID의 검증을 건너뛰므로 API 엔드포인트가 수용하는 모든 문자열을 사용할 수 있습니다. [`availableModels`](#restrict-model-selection)가 설정되어 있을 때는 허용 목록에 커스텀 모델 ID도 포함시키세요: 그렇지 않으면 커스텀 항목이 피커에서 필터링되고 `--model` 선택 시 여타 제외된 모델처럼 거부됩니다. `my-gateway/claude-opus-4-8`과 같이 계열 이름을 내포하는 커스텀 ID는 해당 계열의 특정 항목으로 간주되어 와일드카드를 비활성화하므로 선택 가능 상태로 유지하려는 버전도 함께 나열하세요. [병합 동작](#merge-behavior)을 참조하세요.

## 환경 변수

다음 환경 변수들을 사용하여 별칭이 매핑되는 모델 이름을 제어할 수 있습니다. 각 값은 전체 모델 이름이거나 API 프로바이더에 해당하는 식별자여야 합니다.

| 환경 변수                       | 설명                                                                                                                                                                                                                                                                                                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ANTHROPIC_DEFAULT_FABLE_MODEL`  | `fable`에 사용할 모델이며 서드파티 프로바이더에서 [자동 모델 폴백](#automatic-model-fallback)을 위해 Claude Code가 Fable 5로 인식하는 모델 ID                                                                                                                                                                                                                               |
| `ANTHROPIC_DEFAULT_OPUS_MODEL`   | `opus`에 사용할 모델, 또는 계획 모드가 활성화되어 있을 때 `opusplan`에 사용할 모델                                                                                                                                                                                                                                                                                        |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | `sonnet`에 사용할 모델, 또는 계획 모드가 비활성화되어 있을 때 `opusplan`에 사용할 모델                                                                                                                                                                                                                                                                                     |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL`  | `haiku` 또는 [백그라운드 기능](/docs/en/costs#background-token-usage)에 사용할 모델                                                                                                                                                                                                                                                                                       |
| `CLAUDE_CODE_SUBAGENT_MODEL`     | 모든 [subagent](/docs/en/sub-agents#choose-a-model), [에이전트 팀](/docs/en/agent-teams), [워크플로](/docs/en/workflows)가 실행하는 에이전트에 사용할 모델. `haiku`와 같은 별칭이나 전체 모델 이름을 수용하며, 호출별 `model` 매개변수 및 subagent 정의의 `model` 프론트매터를 재정의함. 일반적인 모델 해상도를 사용하려면 `inherit`으로 설정                                |

참고: `ANTHROPIC_SMALL_FAST_MODEL`은 사용 중단(deprecated)되었으며 `ANTHROPIC_DEFAULT_HAIKU_MODEL` 사용이 권장됩니다.

### 서드파티 배포를 위한 모델 고정

[Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry), 또는 [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)을 통해 Claude Code를 배포할 때 사용자에게 롤아웃하기 전에 모델 버전을 고정하세요.

고정하지 않으면 Claude Code는 프로바이더별 내장 기본 모델 ID로 해석되는 `fable`, `opus`, `sonnet`, `haiku`와 같은 모델 별칭을 사용합니다. 해당 기본값은 최신 Anthropic 릴리스보다 뒤처질 수 있으며, 가리키는 모델을 사용자 계정에서 아직 사용할 수 없을 수도 있습니다. 기본값을 사용할 수 없을 때 Amazon Bedrock 및 Google Cloud's Agent Platform 사용자는 안내를 받게 되고 세션은 기본 모델의 이전 버전으로 폴백되거나, 기본값이 Opus 모델인데 Opus 버전을 이용할 수 없을 때 기본 Sonnet 모델로 폴백됩니다. Microsoft Foundry는 동등한 시작 검사가 없으므로 사용자가 오류를 보게 됩니다.

Amazon Bedrock 및 Google Cloud's Agent Platform에서 `--model`, `ANTHROPIC_MODEL`, 또는 `model` 설정으로 특정 Sonnet이나 Opus 버전에서 세션을 시작한 사용자는 해당 버전을 해당 별칭에 대한 세션 기본값으로 고정합니다: 시작 검사가 대체하는 내장 기본값을 건너뛰고 폴백 안내를 표시하지 않습니다. v2.1.211 이전에는 세션 모델이 명시적으로 구성되어 있어도 검사가 실행되어 안내를 표시할 수 있었습니다.

<Warning>
  초기 설정의 일부로 특정 버전 ID로 모델 환경 변수를 설정하세요. 고정을 통해 사용자가 언제 새 모델로 이동할지 제어할 수 있습니다.
</Warning>

프로바이더에 대한 버전별 모델 ID와 함께 다음 환경 변수들을 사용하세요:

| 프로바이더                    | 예시                                                                 |
| :---------------------------- | :------------------------------------------------------------------- |
| Amazon Bedrock                | `export ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-4-8'` |
| Google Cloud's Agent Platform | `export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'`              |
| Microsoft Foundry             | `export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'`              |

`ANTHROPIC_DEFAULT_FABLE_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`, `ANTHROPIC_DEFAULT_HAIKU_MODEL`에 대해서도 동일한 패턴을 적용하세요. 모든 프로바이더에 걸친 현재 및 이전 모델 ID는 [모델 개요](https://platform.claude.com/docs/en/about-claude/models/overview)를 참조하세요. 사용자를 새 모델 버전으로 업그레이드하려면 이 환경 변수들을 업데이트하고 다시 배포하세요.

고정된 모델에 대해 [확장 컨텍스트](#extended-context)를 활성화하려면 `ANTHROPIC_DEFAULT_OPUS_MODEL` 또는 `ANTHROPIC_DEFAULT_SONNET_MODEL`의 모델 ID 뒤에 `[1m]`을 붙이세요:

```bash theme={null}
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8[1m]'
```

`[1m]` 접미사는 [`opusplan`](#opusplan-model-setting)의 계획 모드 Opus 단계를 포함하여 `opus` 및 `sonnet` 별칭의 모든 사용에 1M 컨텍스트 창을 적용합니다.

* Claude Code는 프로바이더로 모델 ID를 전송하기 전에 접미사를 제거합니다.
* 기반 모델이 [1M 컨텍스트를 지원](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-window-sizes-by-model)할 때만 `[1m]`을 붙이세요.
* 접미사는 모델별이 아닌 변수별로 읽힙니다. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry에서 한 변수의 접미사 없는 모델 ID는 다른 변수가 접미사와 함께 동일한 모델을 설정하더라도 200K 컨텍스트를 사용합니다. Sonnet 5는 이러한 프로바이더에서 항상 1M 창으로 구동되며 접미사가 전혀 필요하지 않습니다.

<Note>
  [MDM이나 관리형 설정 파일](/docs/en/settings#settings-files)을 통해 전달되는 `availableModels` 허용 목록은 서드파티 프로바이더를 사용할 때도 적용됩니다; [서버 관리형 설정은 거기서 전달되지 않습니다](/docs/en/server-managed-settings#platform-availability). 필터링 매칭은 `opus`와 같은 모델 별칭, `claude-opus-4-8`과 같은 버전 접두사, 또는 전체 프로바이더 형태의 모델 ID에 적용됩니다. `us.anthropic.`과 같은 프로바이더 고유 접두사는 제거되지 않으므로 특정 모델을 허용하려면 피커가 보여주는 것과 동일한 프로바이더 형태의 ID를 나열하거나 [`modelOverrides`](#override-model-ids-per-version)를 통해 매핑하세요. `[1m]` 접미사는 매칭 전에 허용 목록 항목과 요청된 모델 모두에서 제거됩니다.
</Note>

### 고정된 모델 표시 및 기능 커스텀

서드파티 프로바이더에서 모델을 고정할 때 프로바이더 고유 ID가 `/model` 피커에 그대로 표시되며 Claude Code가 모델이 지원하는 기능을 인식하지 못할 수 있습니다. 각 고정된 모델에 대해 동반 환경 변수를 통해 표시 이름을 재정의하고 기능을 선언할 수 있습니다.

이 변수들은 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry와 같은 서드파티 프로바이더에서 적용됩니다. `_NAME` 및 `_DESCRIPTION` 변수는 `ANTHROPIC_BASE_URL`이 [LLM 게이트웨이](/docs/en/llm-gateway)를 가리킬 때도 적용됩니다. `api.anthropic.com`에 직접 연결될 때는 아무런 효과가 없습니다.

| 환경 변수                                             | 설명                                                                                                                                       |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME`                   | `/model` 피커에서 고정된 Opus 모델의 표시 이름. 설정되지 않은 경우 모델 ID가 기본값                                                         |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION`            | `/model` 피커에서 고정된 Opus 모델의 표시 설명. 설정되지 않은 경우 `Custom Opus model`이 기본값                                           |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | 고정된 Opus 모델이 지원하는 기능의 쉼표로 구분된 목록                                                                                     |

동일한 `_NAME`, `_DESCRIPTION`, `_SUPPORTED_CAPABILITIES` 접미사를 `ANTHROPIC_DEFAULT_SONNET_MODEL`, `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `ANTHROPIC_DEFAULT_FABLE_MODEL`, `ANTHROPIC_CUSTOM_MODEL_OPTION`에 사용할 수 있습니다.

Claude Code는 모델 ID를 알려진 패턴과 매칭하여 [노력 수준](#adjust-effort-level) 및 [확장 사고](#extended-thinking)와 같은 기능을 활성화합니다. Amazon Bedrock ARN이나 커스텀 배포 이름과 같은 프로바이더 고유 ID는 종종 이러한 패턴에 매칭되지 않아 지원되는 기능이 비활성화 상태로 남게 됩니다. 모델이 실제로 지원하는 기능을 Claude Code에 알리도록 `_SUPPORTED_CAPABILITIES`를 설정하세요:

| 기능 값                | 활성화 대상                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| `effort`               | [노력 수준](#adjust-effort-level) 및 `/effort` 명령                             |
| `xhigh_effort`         | `xhigh` 노력 수준                                                               |
| `max_effort`           | `max` 노력 수준                                                                 |
| `thinking`             | [확장 사고 (Extended thinking)](#extended-thinking)                            |
| `adaptive_thinking`    | 태스크 복잡성에 따라 사고를 동적으로 할당하는 적응형 추론                       |
| `interleaved_thinking` | 도구 호출 사이의 사고                                                           |

`_SUPPORTED_CAPABILITIES`가 설정되면 나열된 기능은 활성화되고 나열되지 않은 기능은 매칭되는 고정 모델에 대해 비활성화됩니다. 변수가 설정되어 있지 않으면 Claude Code는 모델 ID 기반의 내장 감지 방식으로 폴백합니다.

이 예시는 Opus를 Amazon Bedrock 커스텀 모델 ARN에 고정하고 친숙한 이름을 설정하며 기능을 선언합니다:

```bash theme={null}
export ANTHROPIC_DEFAULT_OPUS_MODEL='arn:aws:bedrock:us-east-1:123456789012:custom-model/abc'
export ANTHROPIC_DEFAULT_OPUS_MODEL_NAME='Opus via Bedrock'
export ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION='Opus 4.7 routed through a Bedrock custom endpoint'
export ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES='effort,xhigh_effort,max_effort,thinking,adaptive_thinking,interleaved_thinking'
```

### 버전별 모델 ID 재정의 (modelOverrides)

위의 계열 수준 환경 변수는 계열 별칭당 하나의 모델 ID를 구성합니다. 동일한 계열 내의 여러 버전을 고유한 프로바이더 ID에 매핑해야 하는 경우 대신 `modelOverrides` 설정을 사용하세요.

`modelOverrides`는 개별 Anthropic 모델 ID를 Claude Code가 프로바이더 API로 전송하는 프로바이더 고유 문자열에 매핑합니다. 사용자가 `/model` 피커에서 매핑된 모델을 선택하면 Claude Code가 내장 기본값 대신 구성된 값을 사용합니다.

이를 통해 기업 관리자는 거버넌스, 비용 할당, 또는 리전별 라우팅을 위해 각 모델 버전을 특정 Amazon Bedrock 추론 프로필 ARN, Google Cloud's Agent Platform 버전 이름, 또는 Microsoft Foundry 배포 이름으로 라우팅할 수 있습니다.

[설정 파일](/docs/en/settings#settings-files)에 `modelOverrides`를 설정하세요:

```json theme={null}
{
  "modelOverrides": {
    "claude-opus-4-7": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/opus-prod",
    "claude-opus-4-6": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/opus-46-prod",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-2:123456789012:application-inference-profile/sonnet-prod"
  }
}
```

키는 반드시 [모델 개요](https://platform.claude.com/docs/en/about-claude/models/overview)에 나열된 Anthropic 모델 ID여야 합니다. 날짜가 포함된 모델 ID의 경우 거기에 표시된 그대로 날짜 접미사를 포함시키세요. 알 수 없는 키는 무시됩니다.

재정의문은 `/model` 피커의 각 항목을 뒷받침하는 내장 모델 ID를 대체합니다. Amazon Bedrock에서 `modelOverrides` 항목은 시작 시 Claude Code가 자동으로 탐색하는 모든 추론 프로필보다 우선 적용됩니다. Claude Code는 Amazon Bedrock 추론 프로필 ARN이나 Microsoft Foundry 배포 이름과 같이 이미 프로바이더 기본 형식인 값을 프로바이더에 그대로 전달합니다.

`--model`, `ANTHROPIC_MODEL` 환경 변수, 또는 `ANTHROPIC_DEFAULT_*_MODEL` 환경 변수를 통해 Anthropic 모델 ID를 직접 전달할 때도 재정의가 적용됩니다. Amazon Bedrock, Google Cloud's Agent Platform, 및 [Mantle](/docs/en/amazon-bedrock#use-the-mantle-endpoint)에서 `modelOverrides` 항목이 없는 Anthropic 모델 ID는 프로바이더가 해당 버전을 지원할 때 해당 버전에 대한 `/model` 피커 행과 동일한 프로바이더 고유 ID로 해석됩니다. Mantle은 버전의 일부만을 지원합니다. Mantle이 지원하는 부분 이외의 Anthropic 모델 ID에 대해 `modelOverrides` 항목이 다루지 않는 한 Claude Code는 매핑 없이 원본 ID를 Mantle에 전송합니다. v2.1.200 이전에는 `--model`과 환경 변수 값이 재정의 맵을 거치지 않고 그대로 프로바이더에 전달되었었습니다.

`modelOverrides`는 `availableModels`와 함께 작동합니다. 허용 목록은 재정의 값이 아닌 Anthropic 모델 ID에 대해 평가되므로 `availableModels`의 `"opus"`와 같은 항목은 Opus 버전이 ARN에 매핑된 경우에도 계속 매칭됩니다. 관리형 설정에 `enforceAvailableModels`가 설정되어 있을 때 강제 적용된 Default는 [가장 높은 우선순위의 관리형 소스](/docs/en/server-managed-settings#settings-precedence)의 `modelOverrides`를 통해서만 해석됩니다. 추론 프로필 ARN에 고정된 버전과 같은 관리자의 매핑은 강제 적용된 Default에서 준수됩니다. 사용자나 프로젝트 설정의 재정의는 이에 영향을 주지 않습니다.

`availableModels`가 [관리형 설정](/docs/en/settings#settings-files)에 설정되어 있을 때 해당 관리형 소스의 `modelOverrides`만 `--model`이나 위의 환경 변수를 통해 직접 전달된 Anthropic 모델 ID에 적용됩니다. Claude Code는 해당 ID에 대한 사용자나 프로젝트 설정의 재정의를 무시하며, 관리형 목록이 제외하는 ID를 어떠한 설정 소스의 `modelOverrides`를 통해서도 해석하지 않습니다. 이 관리형 소스 제한은 Claude Code v2.1.200 이상이 필요합니다. 차단된 ID가 어떻게 처리되는지는 [모델 선택 제한하기](#restrict-model-selection)를 참조하세요.

### 프롬프트 캐싱 구성

Claude Code는 성능을 최적화하고 비용을 절감하기 위해 [프롬프트 캐싱](/docs/en/prompt-caching)을 자동으로 사용합니다. 전역으로 또는 특정 모델 티어별로 프롬프트 캐싱을 비활성화할 수 있습니다:

| 환경 변수                       | 설명                                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------- |
| `DISABLE_PROMPT_CACHING`        | 모든 모델에 대해 프롬프트 캐싱을 비활성화하려면 `1`로 설정. 모델별 설정보다 우선 적용됨           |
| `DISABLE_PROMPT_CACHING_HAIKU`  | Haiku 모델에 대해서만 프롬프트 캐싱을 비활성화하려면 `1`로 설정                                    |
| `DISABLE_PROMPT_CACHING_SONNET` | Sonnet 모델에 대해서만 프롬프트 캐싱을 비활성화하려면 `1`로 설정                                   |
| `DISABLE_PROMPT_CACHING_OPUS`   | Opus 모델에 대해서만 프롬프트 캐싱을 비활성화하려면 `1`로 설정                                     |
| `DISABLE_PROMPT_CACHING_FABLE`  | Fable 모델에 대해서만 프롬프트 캐싱을 비활성화하려면 `1`로 설정                                    |

캐시 TTL을 변경하거나 캐시 미스를 유발하는 요인을 알아보려면 [Claude Code의 프롬프트 캐싱 활용 방식](/docs/en/prompt-caching)을 참조하세요.
