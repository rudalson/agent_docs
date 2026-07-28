> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# OpenTelemetry를 활용한 관찰 가능성(Observability)

> OpenTelemetry를 사용하여 Agent SDK의 트레이스(trace), 메트릭(metric) 및 이벤트를 관찰 가능성 백엔드로 내보냅니다.

프로덕션 환경에서 에이전트를 실행할 때는 수행된 작업에 대한 시각성이 필요합니다:

* 에이전트가 호출한 도구(tool)
* 각 모델 요청에 소요된 시간
* 소비된 토큰 수
* 오류가 발생한 위치

Agent SDK는 Honeycomb, Datadog, Grafana, Langfuse 또는 자체 호스팅 수집기(collector)와 같이 OpenTelemetry Protocol(OTLP)을 지원하는 모든 백엔드로 이러한 데이터를 OpenTelemetry 트레이스, 메트릭, 로그 이벤트로 내보낼 수 있습니다.

이 가이드에서는 SDK가 원격 측정(telemetry) 데이터를 생성하는 방식, 내보내기 구성 방법, 그리고 백엔드에 데이터가 도달한 후 태깅 및 필터링하는 방법에 대해 설명합니다. 백엔드로 내보내는 대신 SDK 응답 스트림에서 직접 토큰 사용량과 비용을 읽으려면 [비용 및 사용량 추적](/docs/en/agent-sdk/cost-tracking)을 참고하세요.

## SDK에서 원격 측정 데이터가 흐르는 방식

Agent SDK는 Claude Code CLI를 자식 프로세스(child process)로 실행하고 로컬 파이프를 통해 통신합니다. CLI에는 OpenTelemetry 계측(instrumentation) 기능이 내장되어 있습니다: 각 모델 요청 및 도구 실행을 감싸는 스팬(span)을 기록하고, 토큰 및 비용 카운터에 대한 메트릭을 내보내며, 프롬프트 및 도구 결과에 대한 구조화된 로그 이벤트를 발송합니다. SDK 자체는 원격 측정 데이터를 직접 생성하지 않습니다. 대신 CLI 프로세스에 구성을 전달하며, CLI가 수집기로 직접 내보냅니다.

구성 요소는 환경 변수로 전달됩니다. 기본적으로 자식 프로세스는 애플리케이션의 환경 변수를 상속받으므로 두 가지 위치 중 한 곳에서 원격 측정을 구성할 수 있습니다:

* **프로세스 환경 변수:** 애플리케이션이 시작되기 전에 셸, 컨테이너 또는 오케스트레이터에서 변수를 설정합니다. 모든 `query()` 호출이 코드 변경 없이 이를 자동으로 인식합니다. 프로덕션 배포 시 권장되는 방식입니다.
* **호출별 옵션:** `ClaudeAgentOptions.env` (Python) 또는 `options.env` (TypeScript)에 변수를 설정합니다. 동일한 프로세스 내의 서로 다른 에이전트에 서로 다른 원격 측정 설정이 필요한 경우 사용합니다. Python에서는 `env`가 상속된 환경 변수에 병합(merge)됩니다. TypeScript에서는 `env`가 상속된 환경 변수를 완전히 대체하므로 전달하는 객체에 `...process.env`를 포함해야 합니다.

CLI는 세 가지 독립적인 OpenTelemetry 신호를 내보냅니다. 각 신호에는 고유한 활성화 스위치와 내보내기 도구(exporter)가 있어 필요한 신호만 켤 수 있습니다.

| 신호 | 포함 내용 | 활성화 방법 |
| ---------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| 메트릭 (Metrics) | 토큰, 비용, 세션, 코드 줄 수, 도구 결정에 대한 카운터 | `OTEL_METRICS_EXPORTER` |
| 로그 이벤트 (Log events) | 각 프롬프트, API 요청, API 오류, 도구 결과에 대한 구조화된 기록 | `OTEL_LOGS_EXPORTER` |
| 트레이스 (Traces) | 각 상호작용, 모델 요청, 도구 호출, 후크(hook)에 대한 스팬 (베타) | `OTEL_TRACES_EXPORTER` 및 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1` |

메트릭 이름, 이벤트 이름 및 속성의 전체 목록은 Claude Code [모니터링](/docs/en/monitoring-usage) 참조 문서에서 확인할 수 있습니다. Agent SDK는 동일한 CLI를 실행하므로 동일한 데이터를 내보냅니다. 스팬 이름은 아래의 [에이전트 트레이스 읽기](#read-agent-traces)에 나와 있습니다.

## 원격 측정 내보내기 활성화

원격 측정 기능은 `CLAUDE_CODE_ENABLE_TELEMETRY=1`을 설정하고 하나 이상의 내보내기 도구를 선택할 때까지 비활성화되어 있습니다. 가장 일반적인 구성은 OTLP HTTP를 통해 세 가지 신호를 모두 수집기로 전송하는 것입니다.

다음 예제는 딕셔너리에 변수를 설정하고 `options.env`를 통해 전달합니다. 에이전트는 단일 작업을 실행하고, 루프가 응답 스트림을 소비하는 동안 CLI는 `collector.example.com`에 위치한 수집기로 스팬, 메트릭, 이벤트를 내보냅니다:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import query, ClaudeAgentOptions

  OTEL_ENV = {
      "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
      # Required for traces, which are in beta. Metrics and log events do not need this.
      "CLAUDE_CODE_ENHANCED_TELEMETRY_BETA": "1",
      # Choose an exporter per signal. Use otlp for the SDK; see the Note below.
      "OTEL_TRACES_EXPORTER": "otlp",
      "OTEL_METRICS_EXPORTER": "otlp",
      "OTEL_LOGS_EXPORTER": "otlp",
      # Standard OTLP transport configuration.
      "OTEL_EXPORTER_OTLP_PROTOCOL": "http/protobuf",
      "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.example.com:4318",
      "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer your-token",
  }


  async def main():
      options = ClaudeAgentOptions(env=OTEL_ENV)
      async for message in query(
          prompt="List the files in this directory", options=options
      ):
          print(message)


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  const otelEnv = {
    CLAUDE_CODE_ENABLE_TELEMETRY: "1",
    // Required for traces, which are in beta. Metrics and log events do not need this.
    CLAUDE_CODE_ENHANCED_TELEMETRY_BETA: "1",
    // Choose an exporter per signal. Use otlp for the SDK; see the Note below.
    OTEL_TRACES_EXPORTER: "otlp",
    OTEL_METRICS_EXPORTER: "otlp",
    OTEL_LOGS_EXPORTER: "otlp",
    // Standard OTLP transport configuration.
    OTEL_EXPORTER_OTLP_PROTOCOL: "http/protobuf",
    OTEL_EXPORTER_OTLP_ENDPOINT: "http://collector.example.com:4318",
    OTEL_EXPORTER_OTLP_HEADERS: "Authorization=Bearer your-token",
  };

  for await (const message of query({
    prompt: "List the files in this directory",
    // env replaces the inherited environment in TypeScript, so spread
    // process.env first to keep PATH, ANTHROPIC_API_KEY, and other variables.
    options: { env: { ...process.env, ...otelEnv } },
  })) {
    console.log(message);
  }
  ```
</CodeGroup>

자식 프로세스는 기본적으로 애플리케이션의 환경 변수를 상속받으므로 Dockerfile, Kubernetes 매니페스트 또는 셸 프로필에서 이러한 변수를 내보내고 `options.env`를 완전히 생략해도 동일한 결과를 얻을 수 있습니다.

<Note>
  `console` 내보내기 도구는 원격 측정 데이터를 표준 출력(stdout)으로 작성하며, SDK는 이를 메시지 채널로 사용합니다. SDK를 통해 실행할 때는 내보내기 도구 값으로 `console`을 설정하지 마세요. 로컬에서 원격 측정 데이터를 확인하려면 `OTEL_EXPORTER_OTLP_ENDPOINT`를 로컬 수집기 또는 올인원 Jaeger 컨테이너로 설정하세요.
</Note>

### 단명(short-lived) 호출에서 원격 측정 데이터 플러시

CLI는 원격 측정 데이터를 일정한 간격으로 배치 처리하여 내보냅니다. 정상적인 프로세스 종료 시 보류 중인 데이터를 플러시하려고 시도하지만, 플러시 제한 시간이 짧기 때문에 수집기 응답이 느리면 스팬이 유실될 수 있습니다. CLI가 종료되기 전에 프로세스가 강제 종료되면 배치 버퍼에 남아 있는 모든 내용이 손실됩니다. 내보내기 간격을 줄이면 두 위험 창을 모두 줄일 수 있습니다.

기본적으로 메트릭은 60초마다 내보내지고, 트레이스와 로그는 5초마다 내보내집니다. 다음 예제는 세 가지 간격을 모두 줄여 짧은 작업이 실행되는 동안에도 데이터가 수집기에 도달하도록 합니다:

<CodeGroup>
  ```python Python theme={null}
  OTEL_ENV = {
      # ... exporter configuration from the previous example ...
      "OTEL_METRIC_EXPORT_INTERVAL": "1000",
      "OTEL_LOGS_EXPORT_INTERVAL": "1000",
      "OTEL_TRACES_EXPORT_INTERVAL": "1000",
  }
  ```

  ```typescript TypeScript theme={null}
  const otelEnv = {
    // ... exporter configuration from the previous example ...
    OTEL_METRIC_EXPORT_INTERVAL: "1000",
    OTEL_LOGS_EXPORT_INTERVAL: "1000",
    OTEL_TRACES_EXPORT_INTERVAL: "1000",
  };
  ```
</CodeGroup>

## 에이전트 트레이스 읽기

트레이스는 에이전트 실행에 대한 가장 상세한 보기를 제공합니다. `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`을 설정하면 에이전트 루프의 각 단계가 트레이싱 백엔드에서 검사할 수 있는 스팬으로 변경됩니다:

* **`claude_code.interaction`:** 프롬프트 수신부터 응답 생성까지 에이전트 루프의 단일 턴(turn)을 감쌉니다.
* **`claude_code.llm_request`:** Claude API에 대한 각 호출을 감싸며, 모델 이름, 지연 시간(latency), 토큰 수를 속성으로 가집니다.
* **`claude_code.tool`:** 각 도구 호출을 감싸며, 권한 대기(`claude_code.tool.blocked_on_user`) 및 실제 실행(`claude_code.tool.execution`)에 대한 자식 스팬을 포함합니다.
* **`claude_code.hook`:** 각 [후크(hook)](/docs/en/agent-sdk/hooks) 실행을 감쌉니다. 상기 변수 외에 상세 베타 트레이싱(`ENABLE_BETA_TRACING_DETAILED=1` 및 `BETA_TRACING_ENDPOINT`) 설정이 필요합니다.

`llm_request`, `tool`, `hook` 스팬은 포함하는 `claude_code.interaction` 스팬의 자식입니다. 에이전트가 Task 도구를 통해 서브에이전트를 생성할 때 서브에이전트의 `llm_request` 및 `tool` 스팬은 부모 에이전트의 `claude_code.tool` 스팬 하위에 중첩되므로 전체 위임 체인이 하나의 트레이스로 표시됩니다.

스팬은 기본적으로 `session.id` 속성을 전달합니다. 동일한 [세션](/docs/en/agent-sdk/sessions)에 대해 여러 `query()` 호출을 수행할 때 백엔드에서 `session.id`로 필터링하여 하나의 타임라인으로 볼 수 있습니다. `OTEL_METRICS_INCLUDE_SESSION_ID`가 거짓(falsy) 값으로 설정되면 이 속성은 생략됩니다.

<Note>
  트레이싱 기능은 베타 상태입니다. 스팬 이름과 속성은 릴리스 간에 변경될 수 있습니다. 트레이스 내보내기 구성 변수에 대해서는 모니터링 참조 문서의 [트레이스 (베타)](/docs/en/monitoring-usage#traces-beta)를 참고하세요.
</Note>

## 애플리케이션에 트레이스 연결

SDK는 W3C 트레이스 컨텍스트를 CLI 자식 프로세스에 자동으로 전파합니다. 애플리케이션에서 OpenTelemetry 스팬이 활성화된 상태에서 `query()`를 호출하면 SDK가 `TRACEPARENT` 및 `TRACESTATE`를 자식 프로세스 환경에 주입하고, CLI가 이를 읽어 `claude_code.interaction` 스팬을 사용자의 스팬의 자식으로 만듭니다. 이렇게 하면 에이전트 실행이 연결되지 않은 루트 스팬이 아닌 애플리케이션의 트레이스 내부에 표시됩니다.

실행 중 발송되는 OTLP 이벤트 로그 기록은 동일한 트레이스 컨텍스트를 전달합니다: `TRACEPARENT`가 설정되면 각 기록의 `trace_id` 및 `span_id`가 애플리케이션의 트레이스와 일치하므로 백엔드에서 [이벤트](/docs/en/monitoring-usage#events)와 스팬을 결합(join)할 수 있습니다. {/* min-version: 2.1.212 */}v2.1.212 이전에는 활성 스팬 외부에서 발송된 이벤트 기록에 `trace_id`나 `span_id`가 포함되지 않았습니다.

트레이스 컨텍스트 전파가 활성화되면 CLI는 실행하는 모든 Bash 및 PowerShell 명령으로도 `TRACEPARENT`를 전달합니다. Bash 도구를 통해 시작된 명령이 자체 OpenTelemetry 스팬을 내보내는 경우 해당 스팬은 명령을 감싸는 `claude_code.tool.execution` 스팬 아래에 중첩됩니다.

`options.env`에 `TRACEPARENT`를 명시적으로 설정하면 자동 주입이 건너뛰어지므로 필요에 따라 특정 부모 컨텍스트를 고정할 수 있습니다. 대화형 CLI 세션은 들어오는 `TRACEPARENT`를 완전히 무시하며, Agent SDK 및 `claude -p` 실행만 이를 준수합니다. 전체 스팬 및 속성 참조는 모니터링 참조 문서의 [트레이스 (베타)](/docs/en/monitoring-usage#traces-beta)를 참고하세요.

## 에이전트의 원격 측정 데이터 태깅

기본적으로 CLI는 `service.name`을 `claude-code`로 보고합니다. 여러 에이전트를 실행하거나 동일한 수집기로 내보내는 다른 서비스와 함께 SDK를 실행하는 경우 서비스 이름을 재정의하고 리소스 속성을 추가하여 백엔드에서 에이전트별로 필터링할 수 있습니다.

다음 예제는 서비스 이름을 변경하고 배포 메타데이터를 첨부합니다. 이 값들은 에이전트가 내보내는 모든 스팬, 메트릭, 이벤트에 OpenTelemetry 리소스 속성으로 적용됩니다:

<CodeGroup>
  ```python Python theme={null}
  options = ClaudeAgentOptions(
      env={
          # ... exporter configuration ...
          "OTEL_SERVICE_NAME": "support-triage-agent",
          "OTEL_RESOURCE_ATTRIBUTES": "service.version=1.4.0,deployment.environment=production",
      },
  )
  ```

  ```typescript TypeScript theme={null}
  const options = {
    env: {
      ...process.env,
      // ... exporter configuration ...
      OTEL_SERVICE_NAME: "support-triage-agent",
      OTEL_RESOURCE_ATTRIBUTES:
        "service.version=1.4.0,deployment.environment=production",
    },
  };
  ```
</CodeGroup>

## 최종 사용자에게 작업 속성 지정

CLI는 Anthropic 호출에 사용하는 자격 증명(credential)을 기반으로 모든 이벤트에 [식별 속성](/docs/en/monitoring-usage#standard-attributes)을 첨부합니다. 하나의 배포에서 여러 최종 사용자에게 서비스를 제공하는 애플리케이션을 구축할 때 이 속성은 에이전트가 대리하여 작업한 최종 사용자가 아닌 서비스의 자격 증명을 식별합니다.

도구 호출 및 MCP 활동을 애플리케이션의 최종 사용자에게 귀속시키려면 각 `query()` 호출에서 리소스 속성으로 최종 사용자 식별 정보를 주입하세요. `OTEL_RESOURCE_ATTRIBUTES`는 [쉼표, 공백, 등호를 예약](/docs/en/monitoring-usage#multi-team-organization-support)하므로 인터폴레이션(보간)하기 전에 값들을 퍼센트 인코딩(Percent-encode)해야 합니다. 다음 예제는 단일 요청의 모든 스팬과 이벤트에 요청 사용자 및 테넌트를 첨부합니다. 이는 사용자 및 테넌트 ID를 전달하는 웹 프레임워크의 `request` 객체를 가정합니다:

<CodeGroup>
  ```python Python theme={null}
  from urllib.parse import quote

  options = ClaudeAgentOptions(
      env={
          # ... exporter configuration ...
          "OTEL_RESOURCE_ATTRIBUTES": f"enduser.id={quote(request.user_id)},tenant.id={quote(request.tenant_id)}",
      },
  )
  ```

  ```typescript TypeScript theme={null}
  const options = {
    env: {
      ...process.env,
      // ... exporter configuration ...
      OTEL_RESOURCE_ATTRIBUTES: `enduser.id=${encodeURIComponent(request.userId)},tenant.id=${encodeURIComponent(request.tenantId)}`,
    },
  };
  ```
</CodeGroup>

최종 사용자 식별 정보가 첨부되면, `claude_code.` 접두사가 붙은 로그 기록으로 내보내지는 `tool_decision`, `tool_result`, `mcp_server_connection`, `permission_mode_changed` 이벤트가 사용자별 감사 추적(audit trail)이 되어 보안 정보 및 이벤트 관리(SIEM) 플랫폼으로 전달될 수 있습니다. 보안 관련 이벤트와 각 이벤트가 전달하는 속성의 전체 목록은 모니터링 참조 문서의 [보안 이벤트 감사](/docs/en/monitoring-usage#audit-security-events)를 참고하세요.

## 내보내기 시 민감한 데이터 제어

원격 측정 데이터는 기본적으로 구조적입니다. 기간, 모델 이름, 도구 이름은 모든 스팬에 기록되며, 토큰 수는 기본 API 요청이 사용량 데이터를 반환할 때 기록되므로 실패하거나 중단된 요청의 스팬에서는 생략될 수 있습니다. 에이전트가 읽고 쓰는 콘텐츠는 기본적으로 기록되지 않습니다. 아래의 옵션 변수들을 통해 내보내는 데이터에 콘텐츠를 추가할 수 있습니다:

| 변수 | 추가 내용 |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OTEL_LOG_USER_PROMPTS=1` | `claude_code.user_prompt` 이벤트 및 `claude_code.interaction` 스팬의 프롬프트 텍스트 |
| `OTEL_LOG_TOOL_DETAILS=1` | `claude_code.tool_result` 이벤트의 도구 입력 인자 (파일 경로, 셸 명령, 검색 패턴) |
| `OTEL_LOG_TOOL_CONTENT=1` | `claude_code.tool` 스팬 이벤트 형태의 전체 도구 입력 및 출력 본문. 기본적으로 60KB로 잘리며, `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH`를 통해 구성 가능. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요. [트레이싱](#read-agent-traces) 활성화 필요 |
| `OTEL_LOG_RAW_API_BODIES` | `claude_code.api_request_body` 및 `claude_code.api_response_body` 로그 이벤트 형태의 전체 Anthropic Messages API 요청 및 응답 JSON. `1`로 설정하면 기본 60KB로 잘린 인라인 본문 저장, `file:<dir>`로 설정하면 이벤트에 `body_ref` 경로가 포함된 디스크 상의 자르지 않은 본문 저장. `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH`로 인라인 자르기 제한 구성 가능, {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요. 본문에는 전체 대화 기록이 포함되며 확장 사고(extended-thinking) 콘텐츠는 편집(redact)됨. 이 변수를 활성화하면 위 세 변수가 공개할 수 있는 모든 내용에 대해 동의하는 것으로 간주됨 |

관찰 가능성 파이프라인이 에이전트가 다루는 데이터를 저장하도록 승인되지 않은 한 이 변수들을 설정하지 않은 상태로 두세요. 속성 및 편집 동작의 전체 목록은 모니터링 참조 문서의 [보안 및 개인정보 보호](/docs/en/monitoring-usage#security-and-privacy)를 참고하세요.

## 관련 문서

에이전트 모니터링 및 배포와 관련된 인접 항목을 다루는 가이드입니다:

* [비용 및 사용량 추적](/docs/en/agent-sdk/cost-tracking): 외부 백엔드 없이 메시지 스트림에서 직접 토큰 및 비용 데이터 읽기.
* [Agent SDK 호스팅](/docs/en/agent-sdk/hosting): 환경 변수 수준에서 OpenTelemetry 변수를 설정할 수 있는 컨테이너에 에이전트 배포.
* [모니터링](/docs/en/monitoring-usage): CLI가 내보내는 모든 환경 변수, 메트릭, 이벤트에 대한 완전한 참조 문서.
