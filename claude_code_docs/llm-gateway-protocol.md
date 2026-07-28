> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 게이트웨이 프로토콜 참조 (Gateway protocol reference)

> Claude Code와 LLM 게이트웨이 간의 API 계약: 전달해야 할 엔드포인트, 헤더 및 본문 필드, 필드가 누락되었을 때의 기능 저하, 비용 추적을 위한 귀속(attribution) 헤더, 모델 탐색(model discovery).

이 페이지는 엔드포인트 호출, 게이트웨이가 전송해야 하는 헤더 및 본문 필드, 그리고 전달되지 않았을 때 작동이 중지되는 기능을 포함하여 Claude Code가 게이트웨이로 보내는 요청을 문서화합니다. Claude Code와 작동하도록 게이트웨이 제품을 구성하는 운영자를 위해 작성되었습니다.

구동 중인 [Claude apps gateway](/docs/en/claude-apps-gateway)는 `GET /protocol`에서 이 계약의 기계 읽기 가능한 버전을 제공합니다. 이는 동일한 전달 요구사항뿐만 아니라 SSO 로그인, 관리형 설정 제공 및 텔레메트리를 위한 Claude apps gateway 전용 엔드포인트까지 취급합니다. Claude apps gateway는 CLI와 동일한 `claude` 바이너리에서 구동되므로, [Claude apps gateway quickstart](/docs/en/claude-apps-gateway#quickstart)는 명세(spec)를 가질 수 있는 실행 인스턴스로 가는 가장 빠른 길입니다.

<Note>
  * 조직을 위해 기존 또는 서드파티 게이트웨이를 배포하려면 [LLM 게이트웨이 배포하기](/docs/en/llm-gateway-rollout)를 참조하세요.
  * 전달받은 자격 증명으로 Claude Code를 게이트웨이에 인증하려는 개별 개발자라면 [Claude Code를 LLM 게이트웨이에 연결하기](/docs/en/llm-gateway-connect)를 참조하세요.
</Note>

이 페이지에서 다루는 내용은 다음과 같습니다:

* [API 형식](#api-formats) 및 각 형식별 제공할 엔드포인트
* [요청 헤더](#request-headers): 업스트림에 전달해야 하는 헤더와 게이트웨이가 자체 소비할 수 있는 헤더
* [시스템 프롬프트 귀속 블록](#system-prompt-attribution-block) 및 프롬프트 캐싱과의 상호작용 방식
* [기능 전달(Feature pass-through)](#feature-pass-through): 헤더나 본문 필드가 누락되었을 때 발생하는 문제
* [모델 탐색(Model discovery)](#model-discovery)

이 페이지에서는 게이트웨이가 각 헤더 및 본문 필드에 대해 수행하는 작업에 대해 두 가지 용어를 사용합니다:

* **변경 없이 전달 (Forward unchanged)**: 바이트 단위 그대로 업스트림에 전달합니다.
* **소비 (Consume)**: 게이트웨이가 라우팅, 귀속, 트레이싱을 위해 읽을 수 있으며 업스트림으로 전달할 필요는 없습니다.

'변경 없이 전달'로 표시되지 않은 모든 항목은 게이트웨이가 자체 소비하거나 무시할 수 있습니다.

## API 형식

게이트웨이는 Claude Code 클라이언트에 최소 하나 이상의 다음 API 형식을 제공해야 합니다. Claude Code가 사용하는 형식은 클라이언트의 구성에 의해 결정됩니다: 아래 표의 "선택 변수" 컬럼에 있는 변수가 Claude Code가 해당 형식으로 게이트웨이를 바라보도록 지정합니다. Google Cloud's Agent Platform은 Google Cloud의 Claude 엔드포인트(구 Vertex AI)이며, 해당 변수명은 `VERTEX` 표기를 유지합니다.

| 형식                                     | 선택 변수                                                     | 엔드포인트                                                               | 변경 없이 전달                                                                                           |
| :--------------------------------------- | :------------------------------------------------------------ | :----------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------- |
| Anthropic Messages                       | `ANTHROPIC_BASE_URL`                                          | `/v1/messages`, `/v1/messages/count_tokens` (선택 사항)                 | `anthropic-beta` 및 `anthropic-version` 요청 헤더                                                        |
| Amazon Bedrock InvokeModel               | `ANTHROPIC_BEDROCK_BASE_URL` with `CLAUDE_CODE_USE_BEDROCK=1` | `/model/{model}/invoke`, `/model/{model}/invoke-with-response-stream`    | `anthropic_beta` 및 `anthropic_version` 요청 본문 필드                                                   |
| Google Cloud's Agent Platform rawPredict | `ANTHROPIC_VERTEX_BASE_URL` with `CLAUDE_CODE_USE_VERTEX=1`   | `:rawPredict`, `:streamRawPredict`, `count-tokens:rawPredict` (선택 사항) | `anthropic-beta` 및 `anthropic-version` 요청 헤더, 그리고 `anthropic_version` 요청 본문 필드            |

### Foundry 및 Claude Platform on AWS

Microsoft Foundry와 [Claude Platform on AWS](/docs/en/claude-platform-on-aws)는 Anthropic Messages 형식을 구현합니다. Claude Code는 각각의 변수인 `ANTHROPIC_FOUNDRY_BASE_URL` 및 `ANTHROPIC_AWS_BASE_URL`을 통해 이들로 라우팅하지만, 이들 앞단에 있는 게이트웨이는 위의 Anthropic Messages 행을 구현하게 됩니다. Claude Platform on AWS 앞단에 있는 게이트웨이는 해당 플랫폼이 [매 요청마다 필수 요구하는](/docs/en/claude-platform-on-aws) `anthropic-workspace-id` 헤더도 함께 전달해야 합니다.

### 선택적 엔드포인트 및 시작 시 트래픽

토큰 카운팅 엔드포인트는 유일한 선택적 엔드포인트입니다: 없으면 Claude Code가 컨텍스트 사용량을 로컬에서 추정합니다. 추론 요청은 `/v1/messages?beta=true`로 POST 전송되므로 전체 URL이 아닌 경로 패턴으로 매칭하세요. Google Cloud's Agent Platform 메소드 접미사는 `/projects/{project}/locations/{location}/publishers/anthropic/models/{model}:streamRawPredict`와 같이 게시자 모델 경로 뒤에 붙습니다.

게이트웨이는 거부하더라도 아무런 문제가 발생하지 않는 시작 시 트래픽도 수신합니다: `HEAD /` 연결성 검사 프로브, 그리고 Amazon Bedrock 형식 게이트웨이의 경우 `GET /inference-profiles?type=SYSTEM_DEFINED` 요청이 이에 해당합니다.

[fast mode](/docs/en/fast-mode) 사용 가능 여부 확인은 게이트웨이 로그에 절대 나타나지 않습니다: `ANTHROPIC_BASE_URL`을 따르지 않고 `api.anthropic.com`을 직접 호출하므로, `api.anthropic.com`으로의 직접 아웃바운드가 차단된 네트워크에서는 게이트웨이를 통한 추론이 잘 동작하더라도 fast mode가 연결 오류를 보고할 수 있습니다. [WebFetch 도메인 안전성 확인](/docs/en/data-usage#webfetch-domain-safety-check) 역시 `api.anthropic.com`을 직접 호출합니다. 이를 복원하는 변수는 [프록시 및 LLM 게이트웨이 뒤에서 fast mode 사용하기](/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways)를 참조하세요.

### 스트리밍 (Streaming)

추론 응답은 반드시 스트리밍(stream) 형태로 제공되어야 합니다. Claude Code는 Server-Sent Events(SSE)가 도착하는 즉시 소비하므로, 응답 전체를 버퍼링한 후 중계하는 게이트웨이는 클라이언트를 지연(stall)시키게 됩니다.

### 업스트림과의 형식 불일치

클라이언트가 사용하는 형식에 따라 게이트웨이가 수신하는 내용이 결정됩니다. 자주 발생하는 실패 유형은 클라이언트가 게이트웨이로 보내는 형식과 그 뒤에 있는 업스트림 프로바이더가 수용하는 형식 간의 불일치입니다.

* 클라이언트가 Amazon Bedrock 또는 Google Cloud's Agent Platform 형식을 사용할 때, Claude Code는 해당 프로바이더가 수용하는 기능 하위 집합만 전송합니다.
* 클라이언트가 Anthropic Messages 형식을 사용할 때, 게이트웨이가 Amazon Bedrock이나 Google Cloud's Agent Platform 업스트림으로 전달하더라도 Claude Code는 전체 기능 집합을 전송합니다.

이러한 차이를 중계하는 것은 게이트웨이의 역할입니다. [기능 전달(Feature pass-through)](#feature-pass-through)에서 이에 실패했을 때 어떤 문제가 발생하는지 설명합니다.

## 요청 헤더

Claude Code는 API 요청 시 다음 헤더를 포함합니다. 헤더 이름은 대소문자를 구분하지 않습니다. `anthropic-version` 및 `anthropic-beta`는 변경 없이 전달하고, 업스트림이 [Claude Platform on AWS](/docs/en/claude-platform-on-aws)인 경우 `anthropic-workspace-id`도 전달하세요. 나머지 헤더는 게이트웨이가 라우팅, 귀속, 트레이싱을 위해 소비할 수 있으며 업스트림으로 전달할 필요는 없습니다.

| 헤더                            | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Authorization`, `x-api-key`    | 개발자의 게이트웨이 자격 증명으로, 설정한 [자격 증명 변수](/docs/en/llm-gateway-connect#set-the-credential-variable)에 따라 한쪽 또는 두 헤더 모두에 들어갑니다.                                                                                                                                                                                                                                                                                  |
| `anthropic-version`             | API 버전으로 현재는 `2023-06-01`입니다. Amazon Bedrock 및 Google Cloud's Agent Platform 형식의 요청에는 `anthropic_version` 본문 필드도 포함되며, 그 값은 이 헤더 값이 아니라 프로바이더 방언(dialect) 문자열입니다.                                                                                                                                                                                                                          |
| `anthropic-beta`                | 요청에 적용되는 쉼표 구분 기능(capability) 값들입니다. 이 헤더는 그래로(verbatim) 전달하세요; Claude Code 릴리스에 따라 값이 계속 추가되므로 개별 값을 허용 목록(allowlist)으로 제한하지 마세요. 개발자가 claude.ai 로그인으로 인증하는 경우(게이트웨이 자격 증명 변수 없이 `ANTHROPIC_BASE_URL`이 설정된 경우 가능), 이 헤더에는 업스트림이 요구하는 OAuth 기능도 포함되며, 이를 제거하면 요청이 `401`로 실패합니다. |
| `x-claude-code-session-id`      | 현재 Claude Code 세션의 고유 식별자입니다. 요청 본문을 파싱하지 않고도 한 세션의 모든 요청을 집계하는 데 사용할 수 있습니다.                                                                                                                                                                                                                                                                                                          |
| `x-claude-code-agent-id`        | 세션 내부에서 생성된 Claude Code 에이전트 요청에 포함되는 [subagent](/docs/en/sub-agents) 식별자입니다. 세션 ID와 함께 사용하여 병렬 에이전트별 비용을 추적하는 데 사용합니다.                                                                                                                                                                                                                                                                                          |
| `x-claude-code-parent-agent-id` | 요청한 에이전트를 생성한 상위 에이전트의 식별자로, 중첩된 에이전트 요청에만 나타납니다.                                                                                                                                                                                                                                                                                                                                                          |

Subagent ID는 생성될 때마다 매번 새롭게 발급됩니다. [agent team](/docs/en/agent-teams)의 이름 지정된 팀원 에이전트는 재연결 시에도 이름 기반의 고정 ID를 재사용합니다. 두 경우 모두 ID는 사람이나 장치가 아닌 에이전트를 식별하는 것이므로 에이전트 ID 헤더를 사용자 식별자로 처리하지 마세요.

개발자가 `ANTHROPIC_CUSTOM_HEADERS`를 설정한 경우 해당 헤더도 요청에 나타납니다.

### 열린 목록(Open list) 형태로 전달

헤더와 본문 필드를 닫힌 목록이 아닌 열린 목록(open list)으로 처리하세요. Claude Code는 릴리스를 거치며 새로운 기능이 추가되고, 이들은 새로운 `anthropic-beta` 값, 새로운 요청 본문 필드, 때로는 새로운 `anthropic-*` 또는 `x-claude-code-*` 헤더 형태로 도착합니다.

Anthropic 형식 업스트림으로 전달할 때는 현재 관찰된 목록만 허용하기보다 `anthropic-*` 요청 헤더와 요청 본문 필드를 변경 없이 그대로 통과시키세요. 관찰된 목록에 고정된 게이트웨이는 다음 기능의 헤더나 필드를 제거하게 되어 해당 기능이 도입된 릴리스에서 동작을 오작동시킵니다.

예외는 Amazon Bedrock이나 Google Cloud's Agent Platform과 같은 비 Anthropic 업스트림으로, 이들 스키마 차이를 중계하는 것은 게이트웨이의 역할입니다; [기능 전달(Feature pass-through)](#feature-pass-through)을 참조하세요.

## 시스템 프롬프트 귀속 블록

Claude Code는 클라이언트 버전과 대화에서 파생된 핑거프린트가 포함된 짧은 귀속(attribution) 블록을 시스템 프롬프트 맨 앞에 붙입니다. `api.anthropic.com` 엔드포인트는 첫 번째 시스템 블록으로 변경 없이 도착할 때 이를 제거한 후 처리하므로 퍼스트파티 프롬프트 캐싱에 영향을 주지 않습니다. 그 외의 모든 업스트림은 이를 프롬프트의 일부로 수신합니다.

제거 작업은 위치 기반(positional)이므로 게이트웨이가 `system` 배열을 변경 없이 전달할 때만 작동합니다. 다른 시스템 콘텐츠를 잃지 않고 블록을 프롬프트에서 제외하려면:

* 수신된 그대로 `system` 배열을 전달하고 블록이 첫 번째에 위치하도록 유지하세요: 다른 시스템 블록을 앞에 붙이거나, 배열 순서를 바꾸거나, 단일 문자열로 변환하면 제거 로직이 작동하지 않아 블록이 모델 및 프롬프트 캐시 키에 다다릅니다.
* 블록을 자체 배열 항목으로 유지하세요: 엔드포인트는 귀속 헤더로 시작하는 병합된 블록을 전체가 귀속 블록인 것으로 처리하여 시스템 프롬프트의 나머지를 포함하여 병합된 모든 내용을 삭제합니다.
* 게이트웨이가 시스템 콘텐츠를 재구성해야 하는 경우, Claude Code가 블록을 생략하도록 [`CLAUDE_CODE_ATTRIBUTION_HEADER=0`](/docs/en/env-vars)을 설정하세요. Anthropic 및 클라우드 프로바이더의 Claude 엔드포인트는 귀속 추적을 위해 블록을 읽으므로 게이트웨이에서 이를 제거하거나 이동하지 말고 클라이언트 측에서 생략하세요.

엔드포인트에 수정 없이 도달하는 요청은 영향을 받지 않습니다.

Claude Code v2.1.181부터는 커스텀 베이스 URL로 요청이 라우팅될 때 이 블록이 대화 수명 동안 고정되므로, 이를 비활성화하지 않고도 전체 요청 본문에 기반한 게이트웨이 측 프롬프트 캐시가 잘 작동합니다. v2.1.181 이전에는 블록에 요청별 토큰이 포함되어 있었으므로, 해당 버전에서는 게이트웨이가 캐시를 구현하는 경우 `CLAUDE_CODE_ATTRIBUTION_HEADER=0`을 설정해야 합니다.

## 기능 전달 (Feature pass-through)

Claude Code는 `ANTHROPIC_BASE_URL` 게이트웨이를 Anthropic 형식 엔드포인트로 처리하고, 아래에서 설명하는 미세한 도구 스트리밍 기본값과 같이 직접 연결에 예약된 일부 진단 및 기본값을 제외하고 `api.anthropic.com`으로 전송하는 동일한 베타 헤더 및 요청 본문 필드를 보냅니다. 이 설정 세트는 릴리스마다 달라질 수 있으므로 구성 내용에 의존하지 마세요.

본문 필드를 추가하는 기능은 베타 헤더와 쌍을 이루며 둘은 함께 이동합니다. 본문은 전달하면서 헤더를 제거하거나 Anthropic 형식 본문을 다른 스키마의 업스트림으로 전달하는 게이트웨이는 `400` 오류를 발생시킵니다. 두 쌍이 모두 함께 제거되었을 때만 기능이 조용히 꺼집니다. 콘텐츠 검사를 위해 요청 본문을 다시 쓰거나 삭제하는 게이트웨이도 헤더 쌍을 깨뜨리므로 수정 없이 검사하세요. 각 기능이 이 쌍 구조에서 벗어나는 경우 표에 참고 사항이 기재되어 있습니다.

미세한 도구 스트리밍(Fine-grained tool streaming)은 직접 연결 기본값 중 하나입니다: 커스텀 베이스 URL로 라우팅될 때 기본적으로 꺼져 있으며, 개발자가 [`CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING=1`](/docs/en/env-vars)을 설정할 때 게이트웨이로 전달됩니다.

| 기능                                                                                                                                                                                                                                          | 헤더 및 본문 쌍                                                                                                                                                                                            | 오작동 시 증상                                                                                                                           | 해결 방법                                                                                                              |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| [적응형 추론(Adaptive reasoning)](/docs/en/model-config#adjust-effort-level)                                                                                                                                                                        | 베타 헤더 없음. Claude Code는 Claude 4.6 이상 모델에 대해 `thinking: {"type": "adaptive"}`를 전송하며, 게이트웨이 별칭 등 인식 불가능한 모델명도 이 필드를 수신하는 최신 모델로 처리함 | 업스트림 모델 빌드가 수용하지 않을 때 `thinking` 필드 또는 `adaptive` 태그를 지목하는 `400` 오류 발생                                  | 업스트림 업그레이드. Opus 4.6 및 Sonnet 4.6의 경우 개발자가 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`을 설정 가능     |
| [컨텍스트 관리(Context management)](https://platform.claude.com/docs/en/build-with-claude/context-editing)                                                                                                                                                      | 컨텍스트 관리 베타 헤더와 `context_management` 본문 필드가 쌍을 이룸                                                                                                                                      | `Extra inputs are not permitted`와 함께 `400` 오류 발생. 게이트웨이가 Anthropic 요청을 받아 Amazon Bedrock으로 전송할 때 흔히 발생        | 둘 다 전달하거나 [`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`](/docs/en/env-vars) 설정                                      |
| [확장 컨텍스트(Extended context)](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-window-sizes-by-model) 및 [인터리브 사고(Interleaved thinking)](https://platform.claude.com/docs/en/build-with-claude/extended-thinking#interleaved-thinking) | 베타 헤더만 존재, 본문 필드 없음                                                                                                                                                                            | 헤더가 제거되면 기능 사용 불가 상태가 됨 (업스트림이 기능 요청을 수신하지 못함)                                                        | `anthropic-beta`를 그대로 전달                                                                                         |
| 베타 [도구 필드(Tool fields)](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)                                                                                                                                                       | 도구 관련 베타 헤더가 `strict` 및 `defer_loading`과 같은 도구 스키마 필드와 쌍을 이룸                                                                                                                       | 본문은 통과하지만 헤더가 누락되었을 때 인식되지 않는 도구 스키마 필드를 지목하는 `400` 오류 발생                                        | 둘 다 전달하거나 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1` 설정                                                      |
| [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) 및 [구조화된 출력(Structured outputs)](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)                                                                        | `output_config` 본문 필드가 effort, 구조화된 출력 형식, 작업 예산 설정을 가지며, 각각 자체 베타 헤더와 쌍을 이룸                                                                                           | Amazon Bedrock 및 Google Cloud's Agent Platform 업스트림에서 주로 `Extra inputs are not permitted` 등 `output_config`를 지목하는 `400` 오류 발생 | 본문 필드와 해당 헤더를 함께 전달                                                                                      |
| [토큰 카운팅(Token counting)](https://platform.claude.com/docs/en/build-with-claude/token-counting)                                                                                                                                                           | 베타 쌍 없음; `count_tokens` 엔드포인트 사용                                                                                                                                                                | Claude Code가 로컬에서 컨텍스트 사용량을 추정하는 방식으로 대체(fallback)됨                                                             | 정확한 토큰 수가 필요한 경우 엔드포인트 노출                                                                           |

`ANTHROPIC_DEFAULT_*_MODEL_SUPPORTED_CAPABILITIES` [변수](/docs/en/model-config)는 `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY`, 그리고 [`CLAUDE_CODE_USE_MANTLE`](/docs/en/amazon-bedrock#use-the-mantle-endpoint)와 같은 프로바이더 구성에서만 모델 기능을 선언합니다. `ANTHROPIC_BASE_URL` 게이트웨이 뒤에서는 아무런 효과가 없습니다.

### 자동 재시도 및 오류 전달

Claude Code는 일부 업스트림 거부 후 자동으로 재시도하며 대화의 나머지 부분 동안 해당 거부된 기능을 비활성화합니다. `thinking` 필드 거부, [thinking signatures](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) 거부, 대화 중간의 시스템 메시지 거부 모두 이 방식으로 복구됩니다. 컨텍스트 관리 및 도구 스키마 필드 거부는 재시도되지 않으며 해당 `400` 오류가 개발자에게 전달됩니다.

재시도 로직은 업스트림의 오류 문구와 매칭되므로 오류 응답 본문을 수정하지 않고 전달하세요. 게이트웨이가 업스트림 오류를 자체 엔벨로프로 감싸면 상태 코드가 유지되더라도 복구 경로가 깨집니다.

### 사전 공개 기능 비활성화

`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`은 컨텍스트 관리 및 베타 도구 필드를 포함하여 모든 프로바이더에서 Claude Code가 사전 공개 기능 및 해당 본문 필드를 전송하지 않도록 정지합니다. 베타가 아닌 모델에 의해 선택되는 적응형 추론에는 영향을 주지 않으며, 구독 인증에 필요한 OAuth 기능도 차단하지 않습니다.

Claude Code가 전송하는 기능 세트는 릴리스에 따라 확장됩니다. 현재 베타 헤더 문자열은 [베타 헤더 참조](https://platform.claude.com/docs/en/api/beta-headers)를 확인하세요; 알려진 목록에 고정하는 대신 새 Claude Code 릴리스에 대해 게이트웨이를 테스트하세요.

## 모델 탐색 (Model discovery)

`ANTHROPIC_BASE_URL`이 Anthropic Messages 형식을 제공하는 게이트웨이를 가리킬 때, Claude Code는 시작 시 게이트웨이의 `/v1/models` 엔드포인트를 쿼리하여 반환된 모델을 `/model` 선택기에 추가할 수 있습니다.

개발자는 자체 환경 변수나 관리형 설정을 통해 [`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`](/docs/en/env-vars)을 설정하여 이를 활성화합니다. 공유 API 키 기반의 게이트웨이가 키로 접근 가능한 모든 모델을 모든 사용자에게 노출하지 않도록 이 기능은 기본적으로 꺼져 있습니다. 사용하려면 Claude Code v2.1.129 이상이 필요합니다.

### 탐색이 실행되는 시점

탐색은 Anthropic Messages 형식에만 적용됩니다. 다음 경우에는 실행되지 않습니다:

* `ANTHROPIC_BASE_URL`이 함께 설정되었더라도 임의의 `CLAUDE_CODE_USE_*` 프로바이더 변수가 설정된 경우
* `ANTHROPIC_BASE_URL`이 설정되지 않았거나 `api.anthropic.com`을 가리키는 경우
* [`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`](/docs/en/env-vars) 또는 조직 정책을 통해 비필수 트래픽이 비활성화된 경우

### 요청 및 응답

요청은 3초 타임아웃이 포함된 `GET /v1/models?limit=1000`이며, 자격 증명이 리다이렉트 대상으로 누출되는 것을 방지하기 위해 모든 리다이렉트는 실패로 처리됩니다. 게이트웨이가 늦게 응답하거나 `/v1/models`를 리다이렉트(`http`에서 `https`로의 리다이렉트 포함)하면 탐색이 조용히 실패합니다; 구성된 베이스 URL에서 엔드포인트를 직접 제공하세요.

탐색 요청은 정확히 하나의 자격 증명 헤더를 전송합니다:

* 설정된 경우 `ANTHROPIC_AUTH_TOKEN`을 bearer 토큰으로 전송
* 그 외의 경우 resolved된 API 키([`apiKeyHelper`](/docs/en/llm-gateway-connect#rotate-credentials-with-apikeyhelper) 값 포함)를 `x-api-key` 헤더로 전송

이는 두 헤더 모두로 헬퍼 값을 전송하는 추론 요청과 다릅니다. `/v1/models`를 인증하는 게이트웨이는 헬퍼 배포를 위해 `x-api-key`를 수용해야 합니다. `ANTHROPIC_CUSTOM_HEADERS`에 정의된 헤더도 함께 포함됩니다.

Claude Code는 응답의 `data` 배열에 있는 각 항목에서 `id` 및 선택적 `display_name`을 읽으며, `id`가 `claude` 또는 `anthropic`으로 시작하지 않는 항목은 무시합니다:

```json theme={null}
{
  "data": [
    { "id": "claude-sonnet-4-6", "display_name": "Claude Sonnet 4.6" },
    { "id": "claude-opus-4-8" }
  ]
}
```

### 선택기 항목 및 캐싱

선택기는 개발자가 Claude Code에서 `/model`을 실행할 때 열리는 대화형 모델 목록입니다. 탐색된 각 항목에는 "From gateway"라는 레이블이 붙으며 `display_name`이 제공된 경우 이를 표시명으로 사용합니다. [`availableModels` 관리형 설정](/docs/en/settings#available-settings)은 탐색이 추가할 수 있는 대상을 제한합니다.

탐색된 ID가 선택기에 이미 존재하는 행과 정확히 일치하거나, 탐색된 ID와 기존 ID가 모두 [Fable](/docs/en/model-config#work-with-fable-5)로 해석될 때 해당 ID는 건너뜁니다. Claude Code v2.1.197부터 탐색된 명시적 ID와 내장 항목이 동일한 모델로 해석될 때 탐색된 ID가 내장 항목으로 병합됩니다. 내장 행은 `sonnet`과 같은 별칭을 키로 사용하므로, `claude-sonnet-5`와 같이 현재 별칭이 가리키는 탐색된 모델 ID는 `sonnet` 행으로 접히는(collapse) 반면, `claude-sonnet-4-6`과 같이 별칭이 가리키지 않는 ID는 내장 항목 옆에 고유한 "From gateway" 행을 추가합니다.

결과는 `~/.claude/cache/gateway-models.json`(Windows의 경우 `%USERPROFILE%\.claude\cache\gateway-models.json`)에 캐시되며 매 시작 시 새로고침됩니다. 요청이 실패하거나 게이트웨이가 `/v1/models`를 구현하지 않은 경우, 선택기는 이전 시작 시의 캐시 목록이나 내장 모델 목록으로 대체(fallback)됩니다. 게이트웨이가 탐색 필터와 일치하지 않는 별칭으로 Claude 모델을 제공하는 경우, 개발자는 [모델 구성](/docs/en/model-config) 변수를 사용하여 해당 별칭을 수동으로 추가할 수 있습니다.

## 관련 리소스

게이트웨이 문서의 나머지 부분 및 기반 API 참조:

* [게이트웨이 개요](/docs/en/gateways): 게이트웨이란 무엇이며 Claude apps gateway와 다른 제품 간에 선택하는 방법
* [기타 LLM 게이트웨이](/docs/en/llm-gateway): 조직이 운영하는 게이트웨이 배포 방식 및 claude.ai 구독과의 상호작용 방식
* [조직을 위한 LLM 게이트웨이 배포하기](/docs/en/llm-gateway-rollout): 이 계약을 활용하는 관리자 체크리스트
* [Claude Code를 LLM 게이트웨이에 연결하기](/docs/en/llm-gateway-connect): 개발자별 구성 및 문제 해결 표
* [베타 헤더 참조](https://platform.claude.com/docs/en/api/beta-headers): 현재 `anthropic-beta` 값 목록
* [Messages API](https://platform.claude.com/docs/en/api/messages): Anthropic 형식 게이트웨이가 구현하는 API 형식
