> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 모니터링

> Claude Code에서 OpenTelemetry를 활성화하고 구성하는 방법을 알아봅니다.

OpenTelemetry(OTel)를 통해 텔레메트리 데이터를 내보냄으로써 조직 전체에서 Claude Code 사용량, 비용 및 도구 활동을 추적할 수 있습니다. Claude Code는 표준 메트릭 프로토콜을 통해 메트릭을 시계열 데이터로, 로그/이벤트 프로토콜을 통해 이벤트를 내보내며, 선택적으로 [추적 프로토콜](#traces-beta)을 통해 분산 추적을 내보냅니다. 모니터링 요구 사항에 맞게 메트릭, 로그 및 추적 백엔드를 구성하세요.

## 빠른 시작

환경 변수를 사용하여 OpenTelemetry를 구성합니다:

```bash theme={null}
# 1. 텔레메트리 활성화
export CLAUDE_CODE_ENABLE_TELEMETRY=1

# 2. 내보내기 선택 (둘 다 선택 사항 - 필요한 것만 구성)
export OTEL_METRICS_EXPORTER=otlp       # 옵션: otlp, prometheus, console, none
export OTEL_LOGS_EXPORTER=otlp          # 옵션: otlp, console, none

# 3. OTLP 엔드포인트 구성 (OTLP 내보내기용)
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317

# 4. 인증 설정 (필요한 경우)
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer your-token"

# 5. 디버깅용: 내보내기 간격 단축
export OTEL_METRIC_EXPORT_INTERVAL=10000  # 10초 (기본값: 60000ms)
export OTEL_LOGS_EXPORT_INTERVAL=5000     # 5초 (기본값: 5000ms)

# 6. Claude Code 실행
claude
```

<Note>
  기본 내보내기 간격은 메트릭의 경우 60초, 로그의 경우 5초입니다. 설정 중에는 디버깅 목적으로 더 짧은 간격을 사용하는 것이 좋습니다. 프로덕션에서 사용할 때는 이 값을 다시 설정해야 합니다.
</Note>

메트릭을 내보내는 설정을 확인하려면 백엔드에서 세션이 시작될 때 Claude Code가 내보내는 `claude_code.session.count` 메트릭을 확인하세요. 로그 전용 설정을 확인하려면 프롬프트를 제출하고 `claude_code.user_prompt` 이벤트를 확인하세요. 아무것도 도착하지 않으면 `claude --debug`를 실행하고 디버그 로그에서 OTel 내보내기 오류를 확인하세요.

전체 구성 옵션은 [OpenTelemetry 사양](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/protocol/exporter.md#configuration-options)을 참조하세요.

## 관리자 구성

관리자는 [관리형 설정 파일](/docs/en/settings#settings-files)을 통해 모든 사용자에 대한 OpenTelemetry 설정을 구성할 수 있습니다. 이를 통해 조직 전체에서 텔레메트리 설정을 중앙에서 제어할 수 있습니다. 설정이 적용되는 방법에 대한 자세한 내용은 [설정 우선순위](/docs/en/settings#settings-precedence)를 참조하세요.

관리형 설정 구성 예시:

```json theme={null}
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.example.com:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer example-token"
  }
}
```

<Note>
  관리형 설정은 MDM(모바일 장치 관리) 또는 기타 장치 관리 솔루션을 통해 배포할 수 있습니다. 관리형 설정 파일에 정의된 환경 변수는 높은 우선순위를 가지며 사용자가 재정의할 수 없습니다.
</Note>

Claude Code는 Bash 도구, 훅, MCP 서버 및 언어 서버를 포함하여 생성하는 하위 프로세스에 `OTEL_*` 환경 변수를 전달하지 않습니다. Bash 도구를 통해 실행하는 OpenTelemetry 계측 애플리케이션은 Claude Code의 내보내기 엔드포인트나 헤더를 상속하지 않으므로 해당 애플리케이션이 자체 텔레메트리를 내보내야 하는 경우 해당 명령에서 직접 변수를 설정하세요.

### 관리형 엔드포인트의 신호별 엔드포인트 제어

관리형 설정에 설정된 일반 `OTEL_EXPORTER_OTLP_ENDPOINT`는 모든 신호의 엔드포인트를 제어합니다. 사용자 설정이나 셸 export와 같이 우선순위가 낮은 소스가 `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`와 같은 신호별 엔드포인트를 설정하는 경우, Claude Code는 시작 시 해당 변수를 제거하고 `claude --debug`로 확인할 수 있는 경고를 기록합니다. 사용자는 한 신호의 OTLP 트래픽을 다른 엔드포인트로 이동할 수 없으므로 이를 방지하기 위해 관리형 설정에서 신호별 엔드포인트 변수를 설정할 필요가 없습니다.

이는 관리자가 텔레메트리가 구성된 관리형 설정을 배포한 머신에만 적용되며, Claude Code가 수집하는 내용이 아니라 텔레메트리가 전달되는 위치를 변경합니다.

관리형 `OTEL_EXPORTER_OTLP_HEADERS`, `OTEL_EXPORTER_OTLP_CLIENT_KEY` 또는 `OTEL_EXPORTER_OTLP_CLIENT_CERTIFICATE` 변수도 엔드포인트 변수를 제어합니다. 그렇지 않으면 해당 자격 증명이 관리형 설정에서 선택하지 않은 수집기로 텔레메트리와 함께 전달될 수 있기 때문입니다.

다음 두 가지 사례에서는 일반적인 키별 우선순위가 유지됩니다:

* 관리형 설정 파일 자체에 설정된 신호별 변수는 여전히 적용되므로 [SIEM 예시](#send-events-to-a-siem)처럼 거기에 설정하여 특정 신호를 다른 수집기로 라우팅할 수 있습니다.
* 내보내기 선택기(`OTEL_METRICS_EXPORTER`, `OTEL_LOGS_EXPORTER` 및 베타 `OTEL_TRACES_EXPORTER`)와 프로토콜 변수는 키별 우선순위를 따르므로 우선순위가 낮은 소스에서 여전히 신호 내보내기 방식을 변경할 수 있습니다. 이러한 설정도 권한을 부여하려면 관리형 설정에서 신호별 키를 직접 설정하세요.

{/* min-version: 2.1.217 */}v2.1.217 이전에는 모든 변수가 독립적으로 키별 설정 우선순위를 따랐기 때문에 사용자 설정이나 셸에 설정된 신호별 엔드포인트가 신호를 관리형 수집기에서 다른 곳으로 리디렉션했습니다.

## 구성 세부 정보

### 공통 구성 변수

이 변수들은 모든 배포에 대한 내보내기, 엔드포인트 및 내보내기 동작을 구성합니다. [관리형 설정](#administrator-configuration)에 설정된 일반 엔드포인트 또는 자격 증명 변수가 모든 신호의 엔드포인트를 제어하고 우선순위가 낮은 소스에서 재정의할 수 없는 경우를 제외하고, 신호별 변수가 일반 변수를 재정의합니다.

| 환경 변수 | 설명 | 예시 값 |
| --- | --- | --- |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 텔레메트리 수집 활성화 (필수) | `1` |
| `OTEL_METRICS_EXPORTER` | 쉼표로 구분된 메트릭 내보내기 유형. 비활성화하려면 `none` 사용 | `console`, `otlp`, `prometheus`, `none` |
| `OTEL_LOGS_EXPORTER` | 쉼표로 구분된 로그/이벤트 내보내기 유형. 비활성화하려면 `none` 사용 | `console`, `otlp`, `none` |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | OTLP 내보내기 프로토콜, 모든 신호에 적용됨 | `grpc`, `http/json`, `http/protobuf` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | 모든 신호에 대한 OTLP 수집기 엔드포인트 | `http://localhost:4317` |
| `OTEL_EXPORTER_OTLP_METRICS_PROTOCOL` | 메트릭 프로토콜, 일반 설정 재정의 | `grpc`, `http/json`, `http/protobuf` |
| `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` | OTLP 메트릭 엔드포인트, 일반 설정 재정의 | `http://localhost:4318/v1/metrics` |
| `OTEL_EXPORTER_OTLP_LOGS_PROTOCOL` | 로그 프로토콜, 일반 설정 재정의 | `grpc`, `http/json`, `http/protobuf` |
| `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT` | OTLP 로그 엔드포인트, 일반 설정 재정의 | `http://localhost:4318/v1/logs` |
| `OTEL_EXPORTER_OTLP_HEADERS` | OTLP 인증 헤더 | `Authorization=Bearer token` |
| `OTEL_METRIC_EXPORT_INTERVAL` | 내보내기 간격 (밀리초) (기본값: 60000) | `5000`, `60000` |
| `OTEL_LOGS_EXPORT_INTERVAL` | 로그 내보내기 간격 (밀리초) (기본값: 5000) | `1000`, `10000` |
| `OTEL_LOG_USER_PROMPTS` | 사용자 프롬프트 내용 로깅 활성화 (기본값: 비활성화) | 활성화 시 `1` |
| `OTEL_LOG_ASSISTANT_RESPONSES` | `assistant_response` 이벤트에서 어시스턴트 응답 텍스트 로깅 활성화 (기본값: 비활성화). 설정되지 않은 경우 `OTEL_LOG_USER_PROMPTS` 값으로 대체됩니다. {/* min-version: 2.1.193 */}Claude Code v2.1.193 이상 필요 | 활성화 시 `1`, 마스킹 유지 시 `0` |
| `OTEL_LOG_TOOL_DETAILS` | 도구 이벤트 및 추적 스팬 속성에 도구 매개변수 및 입력 인수 로깅 활성화: Bash 명령, MCP 서버 및 도구 이름, 스킬 이름, 사용자 작성 워크플로 이름, 도구 입력. 또한 `user_prompt` 이벤트에서 커스텀, 플러그인, MCP 명령 이름 활성화 (기본값: 비활성화). Claude Desktop의 내장 서버의 경우 Claude Desktop이 소유한 세션에서 이 플래그가 꺼져 있더라도 `mcp_server_name`/`mcp_tool_name`이 `tool_decision`/`tool_result`에 내보내집니다. {/* min-version: 2.1.214 */}이 예외에는 Claude Code v2.1.214 이상이 필요합니다. | 활성화 시 `1` |
| `OTEL_LOG_TOOL_CONTENT` | 스팬 이벤트에서 도구 입력 및 출력 내용 로깅 활성화 (기본값: 비활성화). [추적](#traces-beta)이 필요합니다. 내용은 내용 제한(기본값 60KB)에서 잘립니다. | 활성화 시 `1` |
| `OTEL_LOG_RAW_API_BODIES` | 전체 Anthropic Messages API 요청 및 응답 JSON을 `api_request_body` / `api_response_body` 로그 이벤트로 내보냅니다 (기본값: 비활성화). 본문에는 전체 대화 기록이 포함됩니다. 이를 활성화하면 `OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_TOOL_DETAILS`, `OTEL_LOG_TOOL_CONTENT`가 공개하는 모든 내용에 동의함을 의미합니다. | 내용 제한(기본값 60KB)에서 잘린 인라인 본문의 경우 `1`, 이벤트에 `body_ref` 포인터가 포함된 디스크 상의 잘리지 않은 본문의 경우 `file:<dir>` |
| `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH` | 내용 제한: 모델 응답, 도구 내용, 시스템 프롬프트, 원시 API 본문과 같이 내용을 포함하는 속성의 최대 길이(잘림 표시 포함, UTF-16 코드 단위 기준, 기본값: 61440, 즉 60KB). 기본값은 속성 값을 64KB로 제한하는 백엔드에 맞춰 설정됩니다. 백엔드가 더 큰 값을 수용하는 경우에만 높이고, 텔레메트리 볼륨을 줄이려면 낮추세요. OpenTelemetry SDK 속성 제한인 `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT` 또는 해당 logrecord 및 span 변형 중 하나가 더 낮게 설정된 경우, Claude Code는 `[TRUNCATED ...]` 표시가 SDK 제한 내에 유지되도록 더 작은 값에서 잘라냅니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요 | `262144` |
| `OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE` | 메트릭 시간성 선호도 (기본값: `delta`). 백엔드에서 누적 시간성을 기대하는 경우 `cumulative`로 설정 | `delta`, `cumulative` |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | 동적 헤더 갱신 간격 (기본값: 1740000ms / 29분) | `900000` |

`http/protobuf` 및 `http/json` 프로토콜의 경우 Claude Code는 `Content-Length` 헤더와 함께 각 내보내기 요청을 전송합니다. {/* min-version: 2.1.212 */}v2.1.212 이전에는 v2.1.191 이후의 Claude Code 버전에서 청크 전송 인코딩으로 이러한 요청을 전송했으며, 선언된 길이가 필요한 Azure Monitor 및 기타 엔드포인트에서 `411 Length Required` 또는 `400` 오류로 거부되었습니다.

### mTLS 인증

OTLP 내보내기에 대한 클라이언트 인증서를 구성하는 방법은 `OTEL_EXPORTER_OTLP_PROTOCOL` 또는 신호별 재정의를 통해 설정된 해당 신호에 사용 중인 OTLP 프로토콜에 따라 다릅니다. 동일한 구성이 메트릭, 로그 및 추적에 적용됩니다.

| 프로토콜 | 클라이언트 인증서 변수 | 수집기의 CA 신뢰 방식 |
| :--- | :--- | :--- |
| `http/protobuf`, `http/json` | `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`, 선택적으로 `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE`. [네트워크 구성](/docs/en/network-config#mtls-authentication) 참조 | `NODE_EXTRA_CA_CERTS` |
| `grpc` | `OTEL_EXPORTER_OTLP_CLIENT_KEY` 및 `OTEL_EXPORTER_OTLP_CLIENT_CERTIFICATE`, 또는 신호별로 다른 인증서를 사용하기 위한 `OTEL_EXPORTER_OTLP_METRICS_CLIENT_KEY`와 같은 신호별 변형 | `OTEL_EXPORTER_OTLP_CERTIFICATE` |

`grpc`의 경우 OpenTelemetry SDK가 표준 OTLP 변수를 직접 읽으므로 신호별 메트릭 변수를 설정하는 기존 구성이 계속 작동합니다.

### 메트릭 카디널리티 제어

다음 환경 변수는 카디널리티를 관리하기 위해 메트릭에 포함되는 속성을 제어합니다:

| 환경 변수 | 설명 | 기본값 | 비활성화 예시 |
| --- | --- | --- | --- |
| `OTEL_METRICS_INCLUDE_SESSION_ID` | 메트릭에 session.id 속성 포함 | `true` | `false` |
| `OTEL_METRICS_INCLUDE_VERSION` | 메트릭에 app.version 속성 포함 | `false` | `true` |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` | 메트릭에 user.account_uuid 및 user.account_id 속성 포함 | `true` | `false` |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | 메트릭에 app.entrypoint 속성 포함 | `false` | `true` |
| `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES` | `OTEL_RESOURCE_ATTRIBUTES`의 키를 메트릭 데이터포인트의 속성으로 포함 | `true` | `false` |

이 이러한 변수는 메트릭 백엔드의 저장소 요구 사항과 쿼리 성능에 영향을 미치는 메트릭의 카디널리티를 제어하는 데 도움이 됩니다. 카디널리티가 낮으면 일반적으로 성능이 향상되고 저장 비용이 줄어들지만 분석을 위한 세밀한 데이터가 줄어듭니다.

### 추적 (베타)

분산 추적은 각 사용자 프롬프트를 트리거하는 API 요청 및 도구 실행에 연결하는 스팬을 내보내므로 추적 백엔드에서 전체 요청을 단일 추적으로 볼 수 있습니다.

추적은 기본적으로 꺼져 있습니다. 이를 활성화하려면 `CLAUDE_CODE_ENABLE_TELEMETRY=1` 및 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`을 모두 설정한 다음 `OTEL_TRACES_EXPORTER`를 설정하여 스팬이 전송되는 위치를 선택하세요. 추적은 엔드포인트, 프로토콜, 헤더 및 [mTLS](#mtls-authentication)에 대해 [공통 OTLP 구성](#common-configuration-variables)을 재사용합니다. [관리형 설정](#managed-endpoints-govern-signal-specific-endpoints)에 설정된 일반 엔드포인트 또는 자격 증명 변수는 추적 엔드포인트도 제어하므로 아래 표의 `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` 재정의는 관리형 설정이 설정되지 않은 경우에만 적용됩니다.

| 환경 변수 | 설명 | 예시 값 |
| --- | --- | --- |
| `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA` | 스팬 추적 활성화 (필수). `ENABLE_ENHANCED_TELEMETRY_BETA`도 허용됨 | `1` |
| `OTEL_TRACES_EXPORTER` | 쉼표로 구분된 추적 내보내기 유형. 비활성화하려면 `none` 사용 | `console`, `otlp`, `none` |
| `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL` | 추적 프로토콜, `OTEL_EXPORTER_OTLP_PROTOCOL` 재정의 | `grpc`, `http/json`, `http/protobuf` |
| `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` | OTLP 추적 엔드포인트, `OTEL_EXPORTER_OTLP_ENDPOINT` 재정의 | `http://localhost:4318/v1/traces` |
| `OTEL_TRACES_EXPORT_INTERVAL` | 스팬 배치 내보내기 간격 (밀리초) (기본값: 5000) | `1000`, `10000` |

스팬은 기본적으로 사용자 프롬프트 텍스트, 도구 입력 세부 정보 및 도구 내용을 마스킹합니다. 이를 포함하려면 `OTEL_LOG_USER_PROMPTS=1`, `OTEL_LOG_TOOL_DETAILS=1`, `OTEL_LOG_TOOL_CONTENT=1`을 설정하세요.

추적이 활성화되면 Bash 및 PowerShell 하위 프로세스는 활성 도구 실행 스팬의 W3C 추적 컨텍스트를 포함하는 `TRACEPARENT` 환경 변수를 자동으로 상속합니다. 이를 통해 `TRACEPARENT`를 읽는 하위 프로세스가 동일한 추적 아래에 자체 스팬을 상위로 지정할 수 있어 Claude가 실행하는 스크립트 및 명령을 통해 종단 간 분산 추적이 가능해집니다.

추적이 활성화되어 있고 Claude Code가 Anthropic API에 직접 연결된 경우, 각 모델 요청에는 `claude_code.llm_request` 스팬 컨텍스트로 설정된 W3C `traceparent` 헤더가 포함되며, API의 `traceresponse` 헤더는 스팬 링크로 기록됩니다. 함께 이러한 요소는 규격을 준수하는 모든 중계기를 통해 Claude Code의 클라이언트 측 스팬을 서버 측 추적에 연결합니다. 아웃바운드 HTTP MCP 요청도 동일한 방식으로 `traceparent`를 전달합니다. 이 헤더는 타사 공급자에게 전송되지 않습니다.

일부 프록시가 인식할 수 없는 헤더를 거부하므로 기본적으로 모델 및 HTTP MCP 요청의 `traceparent` 헤더는 `ANTHROPIC_BASE_URL`이 설정되지 않았거나 Anthropic API를 가리키는 경우에만 전송됩니다. 하위 프로세스 `TRACEPARENT` 변수는 일관성을 위해 동일한 스위치로 제어됩니다. 커스텀 `ANTHROPIC_BASE_URL` 프록시를 통해 Claude Code를 실행하고 추적 컨텍스트를 전파하려는 경우 `CLAUDE_CODE_PROPAGATE_TRACEPARENT=1`을 설정하세요.

Agent SDK 및 `-p`로 시작된 비대화형 세션에서 Claude Code는 각 상호작용 스팬을 시작할 때 자체 환경에서 `TRACEPARENT` 및 `TRACESTATE`도 읽습니다. 이를 통해 포함하는 프로세스가 활성 W3C 추적 컨텍스트를 하위 프로세스로 전달하여 Claude Code의 스팬이 호출자의 분산 추적의 자식으로 나타나게 할 수 있습니다. 대화형 세션은 CI 또는 컨테이너 환경에서 주변 값을 실수로 상속하지 않도록 인바운드 `TRACEPARENT`를 무시합니다.

인바운드 추적 컨텍스트는 [이벤트](#events)에도 적용됩니다. `TRACEPARENT`가 설정된 Agent SDK 및 `-p` 세션에서 각 OTLP 이벤트 로그 레코드에는 추적 내보내기가 구성되지 않은 경우에도 애플리케이션 추적에 결합하는 `trace_id` 및 `span_id` 값이 포함되므로 로깅 백엔드가 이벤트를 추적의 나머지 부분과 연관시킬 수 있습니다.

상호작용이 활성화된 동안 생성된 레코드는 사용 권한 프롬프트 콜백 또는 시작 시 버퍼링되어 나중에 내보내지는 레코드와 같이 Claude Code가 스팬의 비동기 컨텍스트 외부에서 내보내는 경우에도 상호작용 스팬의 ID를 전달합니다. 활성 상호작용 스팬 없이 내보낸 레코드는 인바운드 `TRACEPARENT` ID를 직접 전달합니다. {/* min-version: 2.1.214 */}v2.1.214 이전에는 스팬의 비동기 컨텍스트 외부에서 내보낸 레코드가 스팬의 ID 대신 인바운드 `TRACEPARENT` ID를 전달했습니다. {/* min-version: 2.1.212 */}v2.1.212 이전에는 활성 스팬 외부에서 내보낸 이벤트 레코드가 `trace_id` 또는 `span_id`를 전달하지 않았습니다.

#### 스팬 계층 구조

각 사용자 프롬프트는 `claude_code.interaction` 루트 스팬을 시작합니다. API 호출, 도구 호출 및 훅 실행은 그 자식으로 기록됩니다. 도구 스팬에는 권한 결정을 기다리는 데 걸린 시간용과 실행 자체용이라는 두 개의 자식 스팬이 있습니다. Agent 도구 또는 레거시 Task 도구가 서브에이전트를 생성할 때 서브에이전트의 API 및 도구 스팬은 부모의 `claude_code.tool` 스팬 아래에 중첩됩니다.

```text theme={null}
claude_code.interaction
├── claude_code.llm_request
├── claude_code.hook                    (상세 베타 추적 필요)
└── claude_code.tool
    ├── claude_code.tool.blocked_on_user
    ├── claude_code.tool.execution
    └── (Agent tool) subagent claude_code.llm_request / claude_code.tool spans
```

Agent SDK 및 `claude -p` 세션에서 환경에 `TRACEPARENT`가 설정되어 있으면 `claude_code.interaction` 자체가 호출자 스팬의 자식이 됩니다.

#### 스팬 속성

모든 스팬에는 [표준 속성](#standard-attributes)과 해당 이름과 일치하는 `span.type` 속성이 포함됩니다. 아래 표에는 각 스팬에 설정된 추가 속성이 나열되어 있습니다. `llm_request`, `tool.execution`, `hook` 스팬은 실패를 기록할 때 OpenTelemetry 상태 `ERROR`를 설정합니다. 다른 스팬은 항상 `UNSET` 상태로 종료됩니다.

**`claude_code.interaction`**

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `user_prompt` | 프롬프트 텍스트. 제어 변수가 설정되지 않으면 값은 `<REDACTED>`입니다 | `OTEL_LOG_USER_PROMPTS` |
| `user_prompt_length` | 문자의 프롬프트 길이 | |
| `interaction.sequence` | 이 세션의 상호작용에 대한 1부터 시작하는 카운터 | |
| `interaction.duration_ms` | 턴의 경과 시간(ms) | |

**`claude_code.llm_request`**

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `model` | 모델 식별자 | |
| `gen_ai.system` | 항상 `anthropic`. OpenTelemetry GenAI 시맨틱 규칙 | |
| `gen_ai.request.model` | `model`과 동일한 값. OpenTelemetry GenAI 시맨틱 규칙 | |
| `query_source` | `repl_main_thread` 또는 서브에이전트 이름과 같이 요청을 보낸 하위 시스템 | |
| `agent_id` | 요청을 보낸 서브에이전트 또는 팀원의 식별자. 메인 세션에서는 없음 | |
| `parent_agent_id` | 이 에이전트를 생성한 에이전트의 식별자. 메인 세션 및 거기서 직접 생성된 에이전트에서는 없음 | |
| `workflow.run_id` | 이 에이전트를 생성한 [Workflow](/docs/en/workflows) 도구 실행의 실행 식별자, `wf_` 접두사 붙음. 워크플로에 의해 생성되지 않은 에이전트에서는 없음 | |
| `workflow.name` | 이 에이전트를 생성한 워크플로의 이름. 사용자 작성 이름은 제어 변수가 설정되지 않으면 `custom`으로 대체됨 | `OTEL_LOG_TOOL_DETAILS` |
| `speed` | `fast` 또는 `normal` | |
| `llm_request.context` | 부모 스팬에 따라 `interaction`, `tool`, `standalone` 중 하나 | |
| `duration_ms` | 재시도를 포함한 전체 경과 시간(ms) | |
| `ttft_ms` | 첫 번째 토큰까지 걸린 시간(ms) | |
| `input_tokens` | API 사용량 블록의 입력 토큰 수 | |
| `output_tokens` | 출력 토큰 수 | |
| `cache_read_tokens` | 프롬프트 캐시에서 읽은 토큰 수 | |
| `cache_creation_tokens` | 프롬프트 캐시에 기록된 토큰 수 | |
| `request_id` | `request-id` 응답 헤더의 Anthropic API 요청 ID | |
| `gen_ai.response.id` | `request_id`와 동일한 값. OpenTelemetry GenAI 시맨틱 규칙 | |
| `client_request_id` | 최종 시도의 클라이언트 생성 `x-client-request-id` | |
| `attempt` | 이 요청에 대해 시도한 총 횟수 | |
| `success` | `true` 또는 `false` | |
| `status_code` | 요청 실패 시 HTTP 상태 코드 | |
| `error` | 요청 실패 시 오류 메시지 | |
| `response.has_tool_call` | 응답에 도구 사용 블록이 포함된 경우 `true` | |
| `stop_reason` | `end_turn`, `tool_use`, `max_tokens`, `stop_sequence`, `pause_turn`, `refusal`과 같은 API 응답 `stop_reason` | |
| `gen_ai.response.finish_reasons` | 문자열 배열로 래핑된 `stop_reason`과 동일한 값. OpenTelemetry GenAI 시맨틱 규칙 | |

각 재시도 시도는 `attempt` 및 `client_request_id` 속성이 포함된 `gen_ai.request.attempt` 스팬 이벤트로도 기록됩니다.

**`claude_code.tool`**

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `tool_name` | 도구 이름 | |
| `duration_ms` | 권한 대기 및 실행을 포함한 경과 시간(ms) | |
| `result_tokens` | 도구 결과의 근사 토큰 크기 | |
| `agent_id` | 도구를 실행한 서브에이전트 또는 팀원의 식별자. 메인 세션에서는 없음 | |
| `parent_agent_id` | 이 에이전트를 생성한 에이전트의 식별자. 메인 세션 및 거기서 직접 생성된 에이전트에서는 없음 | |
| `workflow.run_id` | 이 에이전트를 생성한 Workflow 도구 실행의 실행 식별자, `wf_` 접두사 붙음. 워크플로에 의해 생성되지 않은 에이전트에서는 없음 | |
| `workflow.name` | 이 에이전트를 생성한 워크플로의 이름. 사용자 작성 이름은 제어 변수가 설정되지 않으면 `custom`으로 대체됨 | `OTEL_LOG_TOOL_DETAILS` |
| `tool_use_id` | 이 호출에 대한 모델의 `tool_use` 블록 ID. [tool_result](#tool-result-event) 및 [tool_decision](#tool-decision-event) 이벤트와 훅 페이로드의 `tool_use_id`와 일치하므로 스팬을 해당 레코드에 결합할 수 있습니다 | |
| `gen_ai.tool.call.id` | `tool_use_id`와 동일한 값. OpenTelemetry GenAI 시맨틱 규칙 | |
| `file_path` | Read, Edit, Write 도구의 대상 파일 경로 | `OTEL_LOG_TOOL_DETAILS` |
| `full_command` | Bash 도구의 명령 문자열 | `OTEL_LOG_TOOL_DETAILS` |
| `skill_name` | Skill 도구의 스킬 이름 | `OTEL_LOG_TOOL_DETAILS` |
| `subagent_type` | Agent 도구 또는 레거시 Task 도구의 서브에이전트 유형 | `OTEL_LOG_TOOL_DETAILS` |

`OTEL_LOG_TOOL_CONTENT=1`인 경우 이 스팬은 속성에 도구의 입력 및 출력 본문(속성당 기본 60KB 내용 제한으로 잘림)을 포함하는 `tool.output` 스팬 이벤트도 기록합니다.

**`claude_code.tool.blocked_on_user`**

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `duration_ms` | 권한 결정을 기다리는 데 걸린 시간 | |
| `decision` | `accept` 또는 `reject` | |
| `source` | [도구 결정 이벤트](#tool-decision-event)와 일치하는 결정 소스 | |

**`claude_code.tool.execution`**

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `duration_ms` | 도구 본문을 실행하는 데 걸린 시간 | |
| `tool_use_id` | 상위 `claude_code.tool` 스팬과 동일한 값 | |
| `gen_ai.tool.call.id` | `tool_use_id`와 동일한 값. OpenTelemetry GenAI 시맨틱 규칙 | |
| `success` | `true` 또는 `false` | |
| `error` | 실행 실패 시 `Error:ENOENT` 또는 `ShellError`와 같은 오류 범주 문자열. 제어 변수가 설정되면 전체 오류 메시지가 포함됩니다 | `OTEL_LOG_TOOL_DETAILS` |

**`claude_code.hook`**

이 스팬은 위의 추적 내보내기 구성 외에 `ENABLE_BETA_TRACING_DETAILED=1` 및 `BETA_TRACING_ENDPOINT`가 필요한 상세 베타 추적이 활성화된 경우에만 내보내집니다. 대화형 CLI 세션에서는 해당 기능에 대해 조직이 허용 목록에 등록되어 있어야 합니다. Agent SDK 및 비대화형 `-p` 세션은 제한되지 않습니다. `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA`만 설정된 경우에는 내보내지지 않습니다.

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `hook_event` | `PreToolUse`와 같은 훅 이벤트 유형 | |
| `hook_name` | `PreToolUse:Write`와 같은 전체 훅 이름 | |
| `num_hooks` | 실행된 일치하는 훅 명령 수 | |
| `hook_definitions` | JSON 직렬화된 훅 구성 | `OTEL_LOG_TOOL_DETAILS` |
| `duration_ms` | 일치하는 모든 훅의 경과 시간(ms) | |
| `num_success` | 성공적으로 완료된 훅 수 | |
| `num_blocking` | 차단 결정을 반환한 훅 수 | |
| `num_non_blocking_error` | 차단 없이 실패한 훅 수 | |
| `num_cancelled` | 완료 전에 취소된 훅 수 | |

<Note>
  `new_context`, `system_prompt_preview`, `user_system_prompt`, `tool_input`, `response.model_output`과 같은 추가 내용 포함 속성은 상세 베타 추적이 활성화된 경우에만 내보내집니다. 이들은 안정적인 스팬 스키마의 일부가 아닙니다.

  `user_system_prompt`에는 `OTEL_LOG_USER_PROMPTS=1`도 추가로 필요합니다. 이 속성은 `systemPrompt` SDK 옵션 또는 `--system-prompt` 및 `--append-system-prompt` 플래그를 통해 제공하는 시스템 프롬프트 텍스트만 전달하며(기본 60KB 내용 제한으로 잘림), 요청당 내보내지는 대신 세션당 한 번 내보내집니다.
</Note>

### 동적 헤더

동적 인증이 필요한 엔터프라이즈 환경에서는 헤더를 동적으로 생성하도록 스크립트를 구성할 수 있습니다. 동적 헤더는 `http/protobuf` 및 `http/json` 프로토콜에만 적용됩니다. `grpc` 내보내기는 정적 `OTEL_EXPORTER_OTLP_HEADERS` 값만 사용합니다.

#### 설정 구성

`.claude/settings.json`에 다음을 추가하고 경로를 자체 스크립트로 대체하세요:

```json theme={null}
{
  "otelHeadersHelper": "/path/to/generate-otel-headers.sh"
}
```

값은 공백이 포함된 경로를 포함하여 실행 파일의 경로이거나 인수가 포함된 셸 명령줄일 수 있습니다. Windows에서 값은 항상 셸을 통해 실행되므로 JSON 값 내에 공백이 포함된 경로는 따옴표로 묶으세요.

#### 스크립트 요구 사항

스크립트는 HTTP 헤더를 나타내는 문자열 키-값 쌍이 포함된 유효한 JSON을 출력해야 합니다:

```bash theme={null}
#!/bin/bash
# 예시: 다중 헤더
echo "{\"Authorization\": \"Bearer $(get-token.sh)\", \"X-API-Key\": \"$(get-api-key.sh)\"}"
```

헬퍼가 실패하거나 이러한 요구 사항을 충족하지 않는 결과를 출력하면 Claude Code는 다음 위치에 오류를 보고합니다:

* `/status` 출력
* [`--debug`](/docs/en/cli-reference#cli-flags)로 실행 중이거나 세션에서 `/debug`를 실행한 후의 디버그 로그
* `-p`로 시작된 비대화형 세션의 stderr

#### 갱신 동작

헤더 헬퍼 스크립트는 시작 시 실행되고 그 이후에는 토큰 갱신을 지원하기 위해 주기적으로 실행됩니다. 기본적으로 스크립트는 29분마다 실행됩니다. `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` 환경 변수로 간격을 맞춤 설정하세요.

### 다중 팀 조직 지원

여러 팀이나 부서가 있는 조직은 `OTEL_RESOURCE_ATTRIBUTES` 환경 변수를 사용하여 서로 다른 그룹을 구분하기 위한 커스텀 속성을 추가할 수 있습니다:

```bash theme={null}
# 팀 식별을 위한 커스텀 속성 추가
export OTEL_RESOURCE_ATTRIBUTES="department=engineering,team.id=platform,cost_center=eng-123"
```

이러한 커스텀 속성은 모든 메트릭 및 이벤트에 포함되어 다음을 수행할 수 있습니다:

* 팀 또는 부서별로 메트릭 필터링
* 코스트 센터별 비용 추적
* 팀 전용 대시보드 생성
* 특정 팀에 대한 알림 설정

Claude Code는 이러한 값을 OTLP 리소스 블록에 전송하는 것 외에도 모든 메트릭 데이터포인트 및 이벤트 레코드의 속성으로 첨부합니다. 대부분의 메트릭 백엔드는 데이터포인트 속성을 쿼리 가능한 레이블로 노출하므로 커스텀 키로 메트릭을 직접 그룹화하고 필터링할 수 있습니다. 커스텀 키는 `user.id` 또는 `session.id`와 같은 [표준 속성](#standard-attributes)을 재정의하지 않습니다. 키가 충돌하는 경우 Claude Code는 기본 기본값을 유지합니다.

각 커스텀 키는 모든 메트릭 시리즈의 레이블이 되므로 높은 카디널리티 값은 메트릭 백엔드의 저장 비용을 증가시킵니다. 커스텀 속성을 리소스 블록에만 전송하고 데이터포인트 레이블에서는 생략하려면 `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES=false`를 설정하세요. [메트릭 카디널리티 제어](#metrics-cardinality-control)를 참조하세요.

<Warning>
  `OTEL_RESOURCE_ATTRIBUTES` 환경 변수는 엄격한 형식 지정 요구 사항이 있는 쉼표로 구분된 key=value 쌍을 사용합니다:

  * **공백 불허**: 값에는 공백이 포함될 수 없습니다. 예를 들어 `user.organizationName=My Company`는 잘못된 형식입니다
  * **형식**: 쉼표로 구분된 key=value 쌍이어야 합니다: `key1=value1,key2=value2`
  * **허용되는 문자**: 제어 문자, 공백, 큰따옴표, 쉼표, 세미콜론 및 백슬래시를 제외한 US-ASCII 문자만 허용됩니다
  * **특수 문자**: 허용된 범위 외의 문자는 퍼센트 인코딩되어야 합니다

  공백이 필요한 값의 경우 대신 밑줄이나 camelCase를 사용하세요. 다음 예시는 각 형식으로 `org.name`을 설정합니다:

  ```bash theme={null}
  export OTEL_RESOURCE_ATTRIBUTES="org.name=Johns_Organization"
  export OTEL_RESOURCE_ATTRIBUTES="org.name=JohnsOrganization"
  ```

  제외된 문자뿐만 아니라 모든 문자를 퍼센트 인코딩할 수 있습니다. 이 예시는 공백과 아포스트로피를 모두 인코딩합니다:

  ```bash theme={null}
  export OTEL_RESOURCE_ATTRIBUTES="org.name=John%27s%20Organization"
  ```

  값을 따옴표로 묶어도 공백이 이스케이프되지 않습니다. 예를 들어 `org.name="My Company"`는 `My Company`가 아니라 따옴표가 포함된 문자 그대로의 `"My Company"` 값이 됩니다.
</Warning>

### 예시 구성

`claude`를 실행하기 전에 이 환경 변수들을 설정하세요. 아래 각 시나리오는 완전한 구성을 보여주며 각 변수는 [공통 구성 변수](#common-configuration-variables)에 설명되어 있습니다.

1초 내보내기 간격의 콘솔 디버깅용:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=console
export OTEL_METRIC_EXPORT_INTERVAL=1000
```

gRPC 기반 OTLP용:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

`http://localhost:9464/metrics`에서 수집되는 Prometheus용:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=prometheus
```

여러 내보내기로 메트릭을 전송하려는 경우:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=console,otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=http/json
```

메트릭과 로그를 다른 엔드포인트나 백엔드로 전송하려는 경우:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_METRICS_PROTOCOL=http/protobuf
export OTEL_EXPORTER_OTLP_METRICS_ENDPOINT=http://metrics.example.com:4318
export OTEL_EXPORTER_OTLP_LOGS_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=http://logs.example.com:4317
```

이벤트나 로그 없이 메트릭만 내보내려는 경우:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

메트릭 없이 이벤트 및 로그만 내보내려는 경우:

```bash theme={null}
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

## 사용 가능한 메트릭 및 이벤트

### 표준 속성

모든 메트릭과 이벤트는 다음 표준 속성을 공유합니다:

| 속성 | 설명 | 제어 변수 |
| --- | --- | --- |
| `session.id` | 고유 세션 식별자 | `OTEL_METRICS_INCLUDE_SESSION_ID` (기본값: true) |
| `app.version` | 현재 Claude Code 버전 | `OTEL_METRICS_INCLUDE_VERSION` (기본값: false) |
| `app.entrypoint` | `cli`, `sdk-cli`, `sdk-ts`, `sdk-py`, `claude-vscode` 등 세션이 실행된 방식 | `OTEL_METRICS_INCLUDE_ENTRYPOINT` (기본값: false) |
| `organization.id` | 조직 UUID (인증 시) | 사용 가능한 경우 항상 포함 |
| `user.account_uuid` | 계정 UUID (인증 시) | `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` (기본값: true) |
| `user.account_id` | `user_01BWBeN28...`와 같이 Anthropic 관리 API와 일치하는 태그 지정된 형식의 계정 ID (인증 시) | `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` (기본값: true) |
| `user.id` | 첫 실행 시 생성되어 `~/.claude.json`에 영구 저장되는 무작위 익명 식별자. 개인정보를 포함하지 않으며 Claude 계정에서 파생되지 않습니다. 파일을 삭제하면 다음 실행 시 연관 없는 새로운 값이 생성됩니다. | 항상 포함 |
| `user.email` | 사용자 이메일 주소 (OAuth를 통해 인증된 경우) | 사용 가능한 경우 항상 포함 |
| `terminal.type` | `iTerm.app`, `vscode`, `cursor`, `tmux`와 같은 터미널 유형 | 감지 시 항상 포함 |
| `OTEL_RESOURCE_ATTRIBUTES` 키 | `department` 또는 `team.id`와 같이 사용자가 설정한 커스텀 속성. [다중 팀 조직 지원](#multi-team-organization-support) 참조 | `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES` (기본값: true) |

Claude Code가 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway)에 로그인된 경우 CLI는 게이트웨이 세션의 인증된 신원으로 내보내기에 스탬프를 찍습니다: `user.id`는 익명 설치 식별자가 아닌 IdP 주체이고, `user.email`은 로그인된 이메일이며, `user.groups`는 쉼표로 구분된 문자열로 IdP 그룹 멤버십을 전달합니다. 각 내보내기에는 `identity.source: gateway-oidc`도 포함됩니다. 게이트웨이 신원이 마지막으로 적용되므로 게이트웨이 세션에서는 `OTEL_RESOURCE_ATTRIBUTES`를 통해 설정된 `user.*` 및 `identity.*` 키가 무시됩니다.

이벤트에는 추가로 다음 속성이 포함됩니다. 이러한 속성은 바운드가 없는 카디널리티를 유발할 수 있으므로 메트릭에는 첨부되지 않습니다:

* `prompt.id`: 다음 프롬프트가 나올 때까지 사용자 프롬프트를 모든 후속 이벤트와 연관시키는 UUID. [이벤트 연관 속성](#event-correlation-attributes) 참조.
* `workspace.host_paths`: 데스크톱 앱에서 선택한 호스트 워크스페이스 디렉터리(문자열 배열)
* `workflow.run_id`: [Workflow](/docs/en/workflows) 도구 실행에 속한 에이전트가 내보내는 API 및 도구 이벤트의 `wf_` 접두사가 붙은 실행 식별자. 하나의 `workflow.run_id`로 이벤트를 필터링하면 해당 실행의 API 요청 및 도구 결과를 재구성할 수 있습니다. 이 식별자는 워크플로 스크립트가 생성하는 에이전트와 해당 에이전트가 생성하는 모든 에이전트(예: 스킬 호출)를 처리합니다. Workflow 도구 결과에 보고된 실행 식별자와 일치합니다. 다른 모든 이벤트에는 없습니다. {/* min-version: 2.1.202 */}Claude Code v2.1.202 이상 필요
* `workflow.name`: `workflow.run_id`와 함께 내보내지는 워크플로 이름(해당 스크립트의 `meta.name`). 수정되지 않은 내장 스크립트를 실행할 때 내장 워크플로 이름이 그대로 나타납니다. 내장 스크립트의 수정된 사본을 포함하여 사용자가 작성한 이름은 `OTEL_LOG_TOOL_DETAILS=1`이 설정되지 않은 경우 `custom`으로 대체됩니다. {/* min-version: 2.1.202 */}Claude Code v2.1.202 이상 필요

### 메트릭

Claude Code는 다음 메트릭을 내보냅니다. 단위 열에는 각 메트릭에 첨부된 OpenTelemetry 단위 문자열이 표시됩니다. 카운트 메트릭에는 단위가 없습니다.

| 메트릭 이름 | 설명 | 단위 |
| --- | --- | --- |
| `claude_code.session.count` | 시작된 CLI 세션 수 | 없음 |
| `claude_code.lines_of_code.count` | 수정된 코드 줄 수 | 없음 |
| `claude_code.pull_request.count` | 생성된 풀 리퀘스트 수 | 없음 |
| `claude_code.commit.count` | 생성된 git 커밋 수 | 없음 |
| `claude_code.cost.usage` | Claude Code 세션 비용 | USD |
| `claude_code.token.usage` | 사용된 토큰 수 | tokens |
| `claude_code.code_edit_tool.decision` | 코드 편집 도구 권한 결정 수 | 없음 |
| `claude_code.active_time.total` | 총 활성 시간 | s |

{/* min-version: 2.1.216 */}`OTEL_METRICS_EXPORTER`에 `prometheus`만 나열되어 있는 경우 Claude Code는 내보낸 메트릭에서 `USD`, `tokens`, `s` 단위를 생략하여 스크랩이 유효한 Prometheus 텍스트 형식을 유지하도록 합니다. 메트릭 이름은 변경되지 않으며 `otlp,prometheus`와 같이 내보내기를 조합하는 구성은 단위를 유지합니다. v2.1.216 이전에는 Prometheus 스크랩에 일부 스크레이퍼가 거부하는 OpenMetrics 전용 `# UNIT` 줄이 포함되었습니다.

### 메트릭 세부 정보

각 메트릭에는 위에 나열된 표준 속성이 포함됩니다. 추가적인 컨텍스트 전용 속성이 있는 메트릭은 아래에 기재되어 있습니다.

#### 세션 카운터

각 세션이 시작될 때 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `start_type`: 세션이 시작된 방식. `"fresh"`, `"resume"`, `"continue"`, `"agents_view"` 중 하나. `"agents_view"` 값은 대화형 세션이 아닌 사용자가 실행한 로컬 UI인 `claude agents` 대시보드 프로세스를 식별합니다. 대시보드에서 UI 프로세스 실행을 대화형 세션과 분리하려면 이 값으로 필터링하세요.

#### 코드 줄 수 카운터

코드가 추가되거나 제거될 때 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `type`: (`"added"`, `"removed"`)
* `model`: 변경을 수행한 모델의 모델 식별자 (예: "claude-sonnet-5")

#### 풀 리퀘스트 카운터

Claude Code가 셸 명령이나 MCP 도구를 통해 풀 리퀘스트 또는 머지 리퀘스트를 생성할 때 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)

#### 커밋 카운터

Claude Code를 통해 git 커밋을 생성할 때 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)

#### 비용 카운터

각 API 요청 후에 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `model`: 모델 식별자 (예: "claude-sonnet-5")
* `query_source`: 요청을 발송한 하위 시스템의 범주. `"main"`, `"subagent"`, `"auxiliary"` 중 하나
* `speed`: 요청이 패스트 모드를 사용한 경우 `"fast"`. 그렇지 않은 경우 없음
* `effort`: 요청에 적용된 [노력 수준](/docs/en/model-config#adjust-effort-level): `"low"`, `"medium"`, `"high"`, `"xhigh"`, `"max"`. 모델이 노력을 지원하지 않는 경우 없음.
* `agent.name`: 요청을 발송한 서브에이전트 유형. 내장 에이전트 이름 및 공식 마켓플레이스 플러그인의 에이전트는 그대로 표시됩니다. 기타 사용자 정의 에이전트 이름은 `"custom"`으로 대체됩니다. 이름이 지정된 서브에이전트 유형에 의해 요청이 발송되지 않은 경우 없음.
* `skill.name`: 요청 시 활성화된 스킬. Skill 도구, `/` 명령으로 설정되거나 생성된 서브에이전트가 상속합니다. 내장, 번들, 사용자 정의 및 공식 마켓플레이스 플러그인 스킬 이름은 그대로 표시됩니다. 서드파티 플러그인 스킬 이름은 `"third-party"`로 대체됩니다. 활성화된 스킬이 없는 경우 없음.
* `plugin.name`: 활성 스킬 또는 서브에이전트가 플러그인에서 제공되는 경우 소유 플러그인. 공식 마켓플레이스 플러그인 이름은 그대로 표시됩니다. 서드파티 플러그인 이름은 `"third-party"`로 대체됩니다. 스킬과 서브에이전트 모두 소유 플러그인이 없는 경우 없음.
* `marketplace.name`: 소유 플러그인이 설치된 마켓플레이스. 공식 마켓플레이스 플러그인에 대해서만 내보내집니다. 그렇지 않은 경우 없음.
* `mcp_server.name`: 이 요청을 생성한 턴에서 실행된 도구의 MCP 서버. 내장, claude.ai 프록시 및 공식 레지스트리 서버 이름은 그대로 표시됩니다. 사용자 구성 서버 이름은 `"custom"`으로 대체됩니다. MCP 도구가 실행되지 않은 경우 없음.
* `mcp_tool.name`: 이 요청을 생성한 턴에서 실행된 MCP 도구. `mcp_server.name`과 동일하게 재작성됩니다. MCP 도구가 실행되지 않은 경우 없음.

#### 토큰 카운터

각 API 요청 후에 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `type`: (`"input"`, `"output"`, `"cacheRead"`, `"cacheCreation"`)
* `model`: 모델 식별자 (예: "claude-sonnet-5")
* `query_source`: 요청을 발송한 하위 시스템의 범주. `"main"`, `"subagent"`, `"auxiliary"` 중 하나
* `speed`: 요청이 패스트 모드를 사용한 경우 `"fast"`. 그렇지 않은 경우 없음
* `effort`: 요청에 적용된 [노력 수준](/docs/en/model-config#adjust-effort-level). 자세한 내용은 [비용 카운터](#cost-counter)를 참조하세요.
* `agent.name`, `skill.name`, `plugin.name`, `marketplace.name`, `mcp_server.name`, `mcp_tool.name`: 요청에 대한 스킬, 플러그인, 에이전트 및 MCP 귀속. 정의 및 재작성 동작은 [비용 카운터](#cost-counter)를 참조하세요.

#### 코드 편집 도구 결정 카운터

사용자가 Edit, Write 또는 NotebookEdit 도구 사용을 승인하거나 거부할 때 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `tool_name`: 도구 이름 (`"Edit"`, `"Write"`, `"NotebookEdit"`)
* `decision`: 사용자 결정 (`"accept"`, `"reject"`)
* `source`: 결정 출처. `"config"`, `"hook"`, `"user_permanent"`, `"user_temporary"`, `"user_abort"`, `"user_reject"` 중 하나. 각 값의 의미는 [도구 결정 이벤트](#tool-decision-event)를 참조하세요.
* `language`: `"TypeScript"`, `"Python"`, `"JavaScript"`, `"Markdown"`과 같은 편집된 파일의 프로그래밍 언어. 인식할 수 없는 파일 확장자의 경우 `"unknown"`을 반환합니다.

#### 활성 시간 카운터

유휴 시간을 제외하고 Claude Code를 적극적으로 사용하는 데 보낸 실제 시간을 추적합니다. 이 메트릭은 키보드 입력 및 응답 읽기와 같은 사용자 상호작용 중과 도구 실행 및 AI 응답 생성과 같은 CLI 처리 중에 증가합니다.

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `type`: 키보드 상호작용의 경우 `"user"`, 도구 실행 및 AI 응답의 경우 `"cli"`

### 이벤트

Claude Code는 OpenTelemetry 로그/이벤트를 통해 다음 이벤트를 내보냅니다(`OTEL_LOGS_EXPORTER`가 구성된 경우):

#### 이벤트 연관 속성

사용자가 프롬프트를 제출할 때 Claude Code는 여러 API 호출을 수행하고 여러 도구를 실행할 수 있습니다. `prompt.id` 속성을 사용하면 이러한 모든 이벤트를 해당 이벤트를 트리거한 단일 프롬프트에 다시 연결할 수 있습니다.

| 속성 | 설명 |
| --- | --- |
| `prompt.id` | 단일 사용자 프롬프트를 처리하는 동안 생성된 모든 이벤트를 연결하는 UUID v4 식별자 |
| `message.uuid` | 세션 트랜스크립트(`~/.claude/projects/*/*.jsonl` 파일)에 저장된 메시지의 UUID. `assistant_response` 및 command dispatch를 제외한 `user_prompt`(0개 또는 여러 메시지를 생성할 수 있음)에 존재합니다. `assistant_response`에서 이는 다음 턴의 `parentUuid`가 체이닝되는 응답의 최종 트랜스크립트 항목입니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요 |
| `client_request_id` | `x-client-request-id` 요청 헤더로 전송된 클라이언트 생성 UUID. 퍼스트 파티 API 연결의 `api_request` 및 `api_error`에 존재하며, 서드파티 공급자 백엔드 및 비스트리밍 대체(fallback)를 통해 재시도된 요청에는 없습니다. 요청과 응답을 쌍으로 연결하며 서버 `request_id`를 생성하지 않은 타임아웃과 같은 실패에도 계속 사용할 수 있습니다. `llm_request` 추적 스팬의 동일한 속성과 일치합니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요 |

단일 프롬프트에 의해 트리거된 모든 활동을 추적하려면 특정 `prompt.id` 값으로 이벤트를 필터링하세요. 이렇게 하면 해당 프롬프트를 처리하는 동안 발생한 user_prompt 이벤트, api_request 이벤트 및 tool_result 이벤트가 반환됩니다.

메시지 수준 재구성의 경우 각 이벤트 클래스에는 세션 트랜스크립트의 필드와 일치하는 키가 포함됩니다. 트랜스크립트 항목 형식은 [Claude Code 내부 사양](/docs/en/sessions#where-transcripts-are-stored)이며 버전 간에 변경될 수 있으므로 이러한 필드를 조인하는 파이프라인은 모든 릴리스에서 손상될 수 있습니다. 조인을 안정적인 계약이 아닌 버전 전용으로 취급하세요:

* `user_prompt` 및 `assistant_response`에 대한 `message.uuid`
* API 이벤트의 `request_id` (트랜스크립트의 어시스턴트 항목에 `requestId`로 영구 저장됨)
* `tool_result` 및 `tool_decision` 이벤트의 `tool_use_id`

<Note>
  `prompt.id`는 각 프롬프트가 고유한 ID를 생성하여 끊임없이 증가하는 시계열 수를 생성하므로 메트릭에서 의도적으로 제외됩니다. 이벤트 수준 분석 및 감사 추적에만 사용하세요.
</Note>

#### 사용자 프롬프트 이벤트

사용자가 프롬프트를 제출할 때 기록됩니다.

**이벤트 이름**: `claude_code.user_prompt`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"user_prompt"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `prompt_length`: 프롬프트 길이
* `prompt`: 프롬프트 내용. 기본적으로 마스킹됩니다. 포함하려면 `OTEL_LOG_USER_PROMPTS=1`을 설정하세요
* `message.uuid`: 저장된 트랜스크립트 항목과 일치하는 결과 사용자 메시지의 UUID. 0개 또는 여러 메시지를 생성할 수 있는 명령 발송 시에는 없습니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
* `command_name`: 프롬프트가 명령을 호출할 때의 명령 이름. `compact` 또는 `debug`와 같은 내장 및 번들 명령 이름은 그대로 내보내집니다. `reset`과 같은 별칭은 정식 이름이 아닌 입력된 대로 내보내집니다. 커스텀, 플러그인, MCP 명령 이름은 `OTEL_LOG_TOOL_DETAILS=1`이 설정되지 않은 경우 `custom` 또는 `mcp`로 축소됩니다.
* `command_source`: 명령이 있을 때 명령의 출처: `builtin`, `custom`, `mcp`. 플러그인이 제공하는 명령은 `custom`으로 보고됩니다

#### 어시스턴트 응답 이벤트

모델에서 텍스트 내용을 반환하는 각 API 요청 후에 기록됩니다. 응답의 텍스트 블록만 포함되며 사고 블록 및 도구 사용 블록은 제외됩니다. {/* min-version: 2.1.193 */}Claude Code v2.1.193 이상 필요.

**이벤트 이름**: `claude_code.assistant_response`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"assistant_response"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `response_length`: 문자의 응답 텍스트 길이
* `response`: 응답 텍스트 (내용 제한, 기본 60KB로 잘림). 기본적으로 `<REDACTED>`로 마스킹됩니다. 포함하려면 `OTEL_LOG_ASSISTANT_RESPONSES=1`을 설정하세요. `OTEL_LOG_ASSISTANT_RESPONSES`가 설정되지 않은 경우 `OTEL_LOG_USER_PROMPTS`가 대신 제어하므로 프롬프트 로깅이 켜져 있는 동안 응답을 마스킹 상태로 유지하려면 `OTEL_LOG_ASSISTANT_RESPONSES=0`을 설정하세요.
* `model`: 모델 식별자 (예: "claude-sonnet-5")
* `request_id`: 응답의 `request-id` 헤더에 있는 Anthropic API 요청 ID. API가 하나를 반환할 때만 존재합니다.
* `message.uuid`: 응답의 최종 트랜스크립트 항목의 UUID. API 응답은 콘텐츠 블록당 하나의 트랜스크립트 항목으로 영구 저장됩니다. 이것은 다음 턴의 `parentUuid`가 체이닝되는 마지막 항목입니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
* `query_source`: `"repl_main_thread"`, `"compact"`, 서브에이전트 이름과 같이 요청을 보낸 하위 시스템

#### 도구 결과 이벤트

도구가 실행을 완료할 때 기록됩니다. 도구 호출이 거부된 경우에는 내보내지지 않습니다. 거부 사례는 [도구 결정 이벤트](#tool-decision-event)를 참조하세요.

**이벤트 이름**: `claude_code.tool_result`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"tool_result"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `tool_name`: 도구 이름
* `tool_use_id`: 이 도구 호출에 대한 고유 식별자. 훅에 전달된 `tool_use_id`와 일치하여 OTel 이벤트와 훅 캡처 데이터 간의 연관을 가능하게 합니다.
* `success`: `"true"` 또는 `"false"`
* `duration_ms`: 밀리초 단위의 실행 시간
* `error_type`: 도구가 실패했을 때의 오류 범주 문자열 (예: `"Error:ENOENT"` 또는 `"ShellError"`)
* `error` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): 도구가 실패했을 때의 전체 오류 메시지
* `decision_type`: 이 이벤트는 도구가 실행된 후에만 내보내지므로 항상 `"accept"`입니다. 거부된 호출은 도구 결과를 생성하지 않습니다.
* `decision_source`: 권한 결정이 시작된 출처. `"config"`, `"hook"`, `"user_permanent"`, `"user_temporary"` 중 하나. 각 값의 의미는 [도구 결정 이벤트](#tool-decision-event)를 참조하세요. 거부 전용 소스인 `"user_abort"` 및 `"user_reject"`는 이 이벤트에 나타나지 않습니다.
* `tool_input_size_bytes`: JSON 직렬화된 도구 입력 크기 (바이트)
* `tool_result_size_bytes`: 도구 결과 크기 (바이트)
* `mcp_server_scope`: MCP 서버 범위 식별자 (MCP 도구의 경우)
* `tool_parameters` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): 도구 전용 매개변수를 포함하는 JSON 문자열. Claude Desktop의 내장 서버의 경우 Claude Desktop이 소유한 세션에서 [도구 결정 이벤트](#tool-decision-event)와 동일한 호스트 작성 예외로 플래그가 꺼져 있더라도 `mcp_server_name`/`mcp_tool_name` 쌍이 포함됩니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요. 매개변수는 도구에 따라 다릅니다:
  * Bash 도구의 경우: `bash_command`, `full_command`, `timeout`, `description`, `dangerouslyDisableSandbox`, `git_commit_id`(`git commit` 명령이 성공했을 때의 커밋 SHA)를 포함합니다. 데스크톱 앱의 워크스페이스 bash 도구도 `tool_name`을 `Bash`로 보고하지만 `bash_command`, `full_command`, `timeout`만 포함합니다.
  * MCP 도구의 경우: `mcp_server_name`, `mcp_tool_name`을 포함합니다.
  * Skill 도구의 경우: `skill_name`을 포함합니다.
  * Agent 도구 또는 레거시 Task 도구의 경우: `subagent_type`을 포함합니다.
* `tool_input` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): JSON 직렬화된 도구 인수. 512 자를 초과하는 개별 값은 잘리며 전체 페이로드는 \~4K 자로 제한됩니다. MCP 도구를 포함한 모든 도구에 적용됩니다.

#### API 요청 이벤트

Claude에 대한 각 API 요청 시 기록됩니다.

**이벤트 이름**: `claude_code.api_request`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"api_request"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `model`: 사용된 모델 (예: "claude-sonnet-5")
* `cost_usd`: USD 단위의 추정 비용
* `cost_usd_micros`: 100만 분의 1 달러 단위의 추정 비용 (정수로 내보내짐)
* `duration_ms`: 밀리초 단위의 요청 지속 시간
* `input_tokens`: 입력 토큰 수
* `output_tokens`: 출력 토큰 수
* `cache_read_tokens`: 캐시에서 읽은 토큰 수
* `cache_creation_tokens`: 캐시 생성에 사용된 토큰 수
* `request_id`: 응답의 `request-id` 헤더에 있는 Anthropic API 요청 ID (예: `"req_011..."`). API가 하나를 반환할 때만 존재합니다.
* `client_request_id`: `x-client-request-id` 요청 헤더로 전송된 클라이언트 생성 UUID. `llm_request` 추적 스팬의 동일한 속성과 일치합니다. 서드파티 공급자 백엔드 및 비스트리밍 대체를 통해 재시도된 요청에는 없습니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
* `speed`: 패스트 모드의 활성화 여부를 나타내는 `"fast"` 또는 `"normal"`
* `query_source`: `"repl_main_thread"`, `"compact"`, 서브에이전트 이름과 같이 요청을 보낸 하위 시스템
* `effort`: 요청에 적용된 [노력 수준](/docs/en/model-config#adjust-effort-level): `"low"`, `"medium"`, `"high"`, `"xhigh"`, `"max"`. 모델이 노력을 지원하지 않는 경우 없음.
* `agent.name`, `skill.name`, `plugin.name`, `marketplace.name`, `mcp_server.name`, `mcp_tool.name`: 요청에 대한 스킬, 플러그인, 에이전트 및 MCP 귀속. 정의 및 재작성 동작은 [비용 카운터](#cost-counter)를 참조하세요.

#### API 오류 이벤트

Claude에 대한 API 요청이 실패할 때 기록됩니다.

**이벤트 이름**: `claude_code.api_error`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"api_error"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `model`: 사용된 모델 (예: "claude-sonnet-5")
* `error`: 오류 메시지
* `status_code`: 숫자로 표시된 HTTP 상태 코드. 연결 실패와 같은 비 HTTP 오류에는 없습니다.
* `duration_ms`: 밀리초 단위의 요청 지속 시간
* `attempt`: 초기 요청을 포함하여 시도한 총 횟수 (`1`은 재시도가 발생하지 않음을 의미함)
* `request_id`: 응답의 `request-id` 헤더에 있는 Anthropic API 요청 ID (예: `"req_011..."`). API가 하나를 반환할 때만 존재합니다.
* `client_request_id`: `x-client-request-id` 요청 헤더로 전송된 클라이언트 생성 UUID. 타임아웃이나 연결 오류와 같은 실패로 인해 서버 `request_id`가 생성되지 않은 경우에도 사용 가능합니다. 서드파티 공급자 백엔드 및 비스트리밍 대체를 통해 재시도된 요청에는 없습니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
* `speed`: 패스트 모드의 활성화 여부를 나타내는 `"fast"` 또는 `"normal"`
* `query_source`: `"repl_main_thread"`, `"compact"`, 서브에이전트 이름과 같이 요청을 보낸 하위 시스템
* `effort`: 요청에 적용된 [노력 수준](/docs/en/model-config#adjust-effort-level). 모델이 노력을 지원하지 않는 경우 없음.
* `agent.name`, `skill.name`, `plugin.name`, `marketplace.name`, `mcp_server.name`, `mcp_tool.name`: 요청에 대한 스킬, 플러그인, 에이전트 및 MCP 귀속. 정의 및 재작성 동작은 [비용 카운터](#cost-counter)를 참조하세요.

#### API 거절 이벤트

API 요청이 `stop_reason: "refusal"`을 반환할 때 기록됩니다. 거절은 HTTP 오류가 아닌 성공적인 응답 스트림으로 도착하므로 `api_error` 이벤트가 트리거되지 않습니다. 이 이벤트를 통해 거절 빈도를 추적하고 `api_request` 및 `api_error`와 동일한 속성으로 거절을 그룹화할 수 있습니다.

**이벤트 이름**: `claude_code.api_refusal`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"api_refusal"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `model`: 요청의 모델 식별자
* `request_id`: 응답의 `request-id` 헤더에 있는 Anthropic API 요청 ID (예: `"req_011..."`). API가 하나를 반환할 때만 존재합니다.
* `query_source`: `"repl_main_thread"`, `"compact"`, 서브에이전트 이름과 같이 요청을 보낸 하위 시스템. 정의는 [`api_request`](#api-request-event)를 참조하세요.
* `speed`: [패스트 모드](/docs/en/fast-mode)가 활성화된 경우 `"fast"`, 또는 `"normal"`
* `attempt`: 재시도 시도 횟수. 첫 번째 시도는 `1`입니다.
* `effort`: 요청에 적용된 [노력 수준](/docs/en/model-config#adjust-effort-level). 모델이 노력을 지원하지 않는 경우 없음.
* `server_fallback_hop`: API의 서버 측 모델 대체가 이미 이 거절을 다른 모델에서 재시도하여 사용자가 이 특정 거절을 보지 못한 경우 `true`. 요청이 거절로 끝난 경우 `false`. 대체 모델도 거절하는 경우 단일 턴에서 `true` 홉 이벤트와 이후 `false` 최종 이벤트를 모두 내보낼 수 있습니다.
* `has_category`: API 응답에 `"cyber"`, `"bio"`, `"frontier_llm"`, `"reasoning_extraction"` 중 하나의 `stop_details.category`가 포함된 경우 `true`. 카테고리가 없거나 해당 세트 이외의 값이 포함된 경우 `false`. 홉 블록에는 `stop_details`가 없으므로 `server_fallback_hop`이 `true`인 경우 없습니다.
* `has_explanation`: API 응답에 `stop_details.explanation`이 포함된 경우 `true`, 그렇지 않으면 `false`. `server_fallback_hop`이 `true`인 경우 없습니다.
* `category`: API 응답의 `stop_details.category` 값. `"cyber"`, `"bio"`, `"frontier_llm"`, `"reasoning_extraction"` 중 하나. `OTEL_LOG_TOOL_DETAILS=1`이 설정되어 있고 `has_category`가 `true`인 경우에만 존재합니다.
* `agent.name`, `skill.name`, `plugin.name`, `marketplace.name`, `mcp_server.name`, `mcp_tool.name`: 요청에 대한 스킬, 플러그인, 에이전트 및 MCP 귀속. 정의 및 재작성 동작은 [비용 카운터](#cost-counter)를 참조하세요.

#### API 요청 본문 이벤트

`OTEL_LOG_RAW_API_BODIES`가 설정된 경우 각 API 요청 시도마다 기록됩니다. 시도당 하나의 이벤트가 내보내지므로 조정된 매개변수로 재시도할 때마다 자체 이벤트가 생성됩니다.

**이벤트 이름**: `claude_code.api_request_body`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"api_request_body"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `body`: 시스템 프롬프트, 메시지 및 도구와 같은 JSON 직렬화된 Messages API 요청 매개변수로, 내용 제한(기본값 60KB)에서 잘립니다. 이전 어시스턴트 턴의 확장 사고(extended-thinking) 내용은 마스킹됩니다. 인라인 모드(`OTEL_LOG_RAW_API_BODIES=1`)에서만 내보내집니다.
* `body_ref`: 잘리지 않은 본문이 포함된 `<dir>/<uuid>.request.json` 파일에 대한 절대 경로. 파일 모드(`OTEL_LOG_RAW_API_BODIES=file:<dir>`)에서만 내보내집니다.
* `body_length`: 잘리지 않은 본문 길이. `OTEL_LOG_RAW_API_BODIES=file:<dir>`인 경우 UTF-8 바이트, `=1`인 경우 UTF-16 코드 단위
* `body_truncated`: 인라인 잘림이 발생한 경우 `"true"`. 파일 모드 및 잘림이 발생하지 않은 경우 없습니다.
* `model`: 요청 매개변수의 모델 식별자
* `query_source`: 요청을 보낸 하위 시스템 (예: `"compact"`)

#### API 응답 본문 이벤트

`OTEL_LOG_RAW_API_BODIES`가 설정된 경우 성공적인 각 API 응답에 대해 기록됩니다.

**이벤트 이름**: `claude_code.api_response_body`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"api_response_body"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `body`: id, 콘텐츠 블록, 사용량 및 중단 사유를 포함하는 JSON 직렬화된 Messages API 응답으로, 내용 제한(기본값 60KB)에서 잘립니다. 확장 사고 내용은 마스킹됩니다. 인라인 모드(`OTEL_LOG_RAW_API_BODIES=1`)에서만 내보내집니다.
* `body_ref`: 잘리지 않은 본문이 포함된 `<dir>/<request_id>.response.json` 파일에 대한 절대 경로. 파일 모드(`OTEL_LOG_RAW_API_BODIES=file:<dir>`)에서만 내보내집니다.
* `body_length`: 잘리지 않은 본문 길이. `OTEL_LOG_RAW_API_BODIES=file:<dir>`인 경우 UTF-8 바이트, `=1`인 경우 UTF-16 코드 단위
* `body_truncated`: 인라인 잘림이 발생한 경우 `"true"`. 파일 모드 및 잘림이 발생하지 않은 경우 없습니다.
* `model`: 모델 식별자
* `query_source`: 요청을 보낸 하위 시스템
* `request_id`: 응답의 `request-id` 헤더에 있는 Anthropic API 요청 ID (예: `"req_011..."`). API가 하나를 반환할 때만 존재합니다.

#### 도구 결정 이벤트

도구 권한 결정(승인/거부)이 이루어질 때 기록됩니다.

**이벤트 이름**: `claude_code.tool_decision`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"tool_decision"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `tool_name`: 도구 이름 (예: "Read", "Edit", "Write", "NotebookEdit")
* `tool_use_id`: 이 도구 호출에 대한 고유 식별자. 훅에 전달된 `tool_use_id`와 일치하여 OTel 이벤트와 훅 캡처 데이터 간의 연관을 가능하게 합니다.
* `decision`: `"accept"` 또는 `"reject"`
* `tool_source`: 항상 존재함. CLI 작성 값의 닫힌 세트로서의 도구 출처. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
  * `"builtin"`: CLI 자체의 도구
  * `"mcp"`: 일반 MCP 서버
  * `"sdk_host_builtin_mcp"`: Claude Desktop이 소유한 세션에서 Claude Desktop 자체에 내장된 인프로세스 서버. Claude Desktop은 자식 세션이 아닌 경우 자체 엔트리포인트(`claude-desktop`, `claude-desktop-3p`, `local-agent`) 중 하나에서 시작된 세션을 소유하며, Claude Code 자체가 생성하는 세션을 포함한 중첩 세션은 이러한 서버를 `"mcp"`로 보고합니다.
* `source`: 결정이 시작된 위치:
  * `"config"`: 프로젝트 설정, 사용자 개인 설정의 허용/거부 규칙, 엔터프라이즈 관리 정책, `--allowedTools` 또는 `--disallowedTools` 플래그, 활성 권한 모드, 동일한 대화형 CLI 세션의 이전 프롬프트에서 얻은 세션 범위 권한 또는 도구가 본질적으로 안전하기 때문에 프롬프트 없이 자동 결정됨. 이벤트는 이러한 소스 중 어느 것이 일치하는지 나타내지 않습니다. {/* min-version: 2.1.216 */}Claude Code는 권한 프롬프트 요청 자체가 실패할 때(예: Agent SDK의 [`canUseTool`](/docs/en/agent-sdk/typescript#canusetool) 콜백 또는 [`--permission-prompt-tool`](/docs/en/cli-reference#cli-flags) 도구가 잘못된 결과를 반환하거나 요청이 보류 중인 동안 입력 스트림이 닫힐 때)에도 `"config"`를 보고합니다. v2.1.216 이전에는 이러한 실패를 `"user_reject"`로 보고했습니다.
  * `"hook"`: `PreToolUse` 또는 `PermissionRequest` 훅이 결정을 반환함.
  * `"user_permanent"`: 사용자가 권한 프롬프트에서 "예, ...에 대해 다시 묻지 않음"을 선택하여 개인 설정에 허용 규칙을 저장한 경우 내보내집니다. 대화형 CLI에서는 해당 선택 자체에 대해서만 내보내집니다. 저장된 규칙과 일치하는 이후 호출은 대신 `"config"`를 내보냅니다. Agent SDK 또는 비대화형 `-p` 세션에서는 초기 선택과 이후 규칙 일치 모두 `"user_permanent"`를 내보냅니다. 승인으로 취급됩니다.
  * `"user_temporary"`: 사용자가 1회성 승인을 위해 권한 프롬프트에서 "예"를 선택하거나 파일 편집/읽기 프롬프트에서 "... 이 세션 동안" 옵션 중 하나를 선택한 경우 내보내집니다. 대화형 CLI에서는 선택 자체에 대해서만 내보내집니다. 해당 세션 범위 권한에 의해 허용된 이후 호출은 대신 `"config"`를 내보냅니다. Agent SDK 또는 비대화형 `-p` 세션에서는 선택과 이후 일치 모두 `"user_temporary"`를 내보냅니다. 승인으로 취급됩니다.
  * `"user_abort"`: 사용자가 대답하지 않고 권한 프롬프트를 닫았을 때 내보내집니다. {/* min-version: 2.1.216 */}Agent SDK 및 비대화형 `-p` 세션에서는 `canUseTool` 또는 `--permission-prompt-tool` 권한 요청이 보류 중인 동안 턴을 중단하는 것이 포함됩니다. v2.1.216 이전에는 해당 중단을 `"user_reject"`로 보고했습니다. 거부로 취급됩니다.
  * `"user_reject"`: 사용자가 프롬프트에서 "아니오"를 선택했을 때 내보내집니다. 대화형 CLI에서는 해당 선택 자체에 대해서만 내보내집니다. 사용자의 개인 설정에서 거부 규칙과 일치하는 호출은 대신 `"config"`를 내보냅니다. Agent SDK 또는 비대화형 `-p` 세션에서 개인 설정의 거부 규칙과 일치하는 호출은 `"user_reject"`를 내보냅니다. 거부로 취급됩니다.
* `tool_parameters` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): 도구 전용 매개변수를 포함하는 JSON 문자열. [도구 결과 이벤트](#tool-decision-event)와 동일한 형식이지만 `git_commit_id`와 같은 실행 후 필드는 제외됩니다. 권한 결정이 `updatedInput`을 통해 도구 입력을 다시 작성하는 경우 승인된 호출의 값이 `tool_result`와 다를 수 있습니다. `decision`이 `"reject"`일 때 거부된 명령을 확인하려면 이 속성을 사용하세요.
  * `"sdk_host_builtin_mcp"` 도구의 경우: 호스트 애플리케이션이 이러한 이름을 정의하므로 `OTEL_LOG_TOOL_DETAILS`가 꺼져 있는 경우에도 `mcp_server_name` 및 `mcp_tool_name`이 포함됩니다. 이것이 없으면 내장 서버에 대한 거부된 호출의 출처를 기본 스트림에서 확인할 수 없기 때문입니다. 사용자 구성 MCP 서버의 경우 이벤트의 `tool_name`은 항상 문자 그대로 `"mcp_tool"`이며, 서버 및 도구 이름은 플래그가 켜져 있을 때 `tool_parameters`에만 나타납니다. 인수 내용은 모든 위치에서 플래그가 필요합니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
  * Bash 도구의 경우: `bash_command`, `full_command`, `timeout`, `description`, `dangerouslyDisableSandbox`를 포함합니다. 데스크톱 앱의 워크스페이스 bash 도구도 `tool_name`을 `Bash`로 보고하지만 `bash_command`, `full_command`, `timeout`만 포함합니다.
  * MCP 도구의 경우: `mcp_server_name`, `mcp_tool_name`을 포함합니다.
  * Skill 도구의 경우: `skill_name`을 포함합니다.
  * Agent 도구 또는 레거시 Task 도구의 경우: `subagent_type`을 포함합니다.

#### 권한 모드 변경 이벤트

`Shift+Tab` 순환, 계획 모드 종료 또는 자동 모드 게이트 체크 등으로 인해 권한 모드가 변경될 때 기록됩니다.

**이벤트 이름**: `claude_code.permission_mode_changed`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"permission_mode_changed"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `from_mode`: 이전 권한 모드 (예: `"default"`, `"plan"`, `"acceptEdits"`, `"auto"`, `"bypassPermissions"`)
* `to_mode`: 새 권한 모드
* `trigger`: 변경 원인. `"shift_tab"`, `"exit_plan_mode"`, `"auto_gate_denied"`, `"auto_opt_in"` 중 하나. 전환이 SDK 또는 브리지에서 비롯된 경우 없습니다.

#### 인증 이벤트

`/login` 또는 `/logout`이 완료될 때 기록됩니다.

**이벤트 이름**: `claude_code.auth`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"auth"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `action`: `"login"` 또는 `"logout"`
* `success`: `"true"` 또는 `"false"`
* `auth_method`: `"oauth"`와 같은 인증 방법
* `error_category`: 작업 실패 시의 범주형 오류 종류. 원시 오류 메시지는 포함되지 않습니다.
* `status_code`: 작업이 HTTP 오류로 실패했을 때의 문자열 형태의 HTTP 상태 코드

#### MCP 서버 연결 이벤트

MCP 서버가 연결되거나 연결 해제되거나 연결에 실패할 때 기록됩니다.

**이벤트 이름**: `claude_code.mcp_server_connection`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"mcp_server_connection"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `status`: `"connected"`, `"failed"`, `"disconnected"`
* `transport_type`: `"stdio"`, `"sse"`, `"http"`와 같은 서버 전송 방식
* `server_scope`: `"user"`, `"project"`, `"local"`과 같이 서버가 구성된 범위
* `duration_ms`: 밀리초 단위의 연결 시도 시간
* `error_code`: 연결 실패 시의 오류 코드
* `is_plugin`: 서버가 플래그인에서 제공된 경우 `true`, 그렇지 않은 경우 `false`
* `plugin_id_hash` (`is_plugin`이 `true`인 경우): 이름을 노출하지 않고 플러그인별로 이벤트를 그룹화하기 위한 플러그인 이름 및 마켓플레이스의 안정적인 해시
* `plugin.name` (`is_plugin`이 `true`인 경우): 서버를 제공하는 플러그인의 이름. 서드파티 플러그인의 경우 `OTEL_LOG_TOOL_DETAILS=1`이 아니면 기본적으로 로그에 서드파티 플러그인 이름이 노출되지 않도록 문자 그대로 `"third-party"` 문자열로 표시됩니다. 공식 Anthropic 출처의 플러그인은 항상 이름으로 식별됩니다. `plugin_id_hash` 및 `plugin.name` 속성은 자체 모니터링 백엔드로 흐르며 Anthropic으로 전송되지 않습니다.
* `server_name` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): 구성된 서버 이름
* `error` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): 연결 실패 시의 전체 오류 메시지

#### 내부 오류 이벤트

Claude Code가 예상치 못한 내부 오류를 포착할 때 기록됩니다. 오류 클래스 이름과 errno 스타일의 코드만 기록됩니다. 오류 메시지 및 스택 트레이스는 포함되지 않습니다. 이 이벤트는 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry에서 실행 중이거나 `DISABLE_ERROR_REPORTING`이 설정된 경우에는 내보내지지 않습니다.

**이벤트 이름**: `claude_code.internal_error`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"internal_error"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `error_name`: `"TypeError"` 또는 `"SyntaxError"`와 같은 오류 클래스 이름
* `error_code`: 오류에 존재하는 경우 `"ENOENT"`와 같은 Node.js errno 코드

#### 플러그인 설치 이벤트

`claude plugin install` CLI 명령 및 대화형 `/plugin` UI 모두에서 플러그인 설치가 완료될 때 기록됩니다.

**이벤트 이름**: `claude_code.plugin_installed`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"plugin_installed"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `marketplace.is_official`: 마켓플레이스가 공식 Anthropic 마켓플레이스인 경우 `"true"`, 그렇지 않은 경우 `"false"`
* `install.trigger`: `"cli"` 또는 `"ui"`
* `plugin.name`: 설치된 플러그인의 이름. 서드파티 마켓플레이스의 경우 `OTEL_LOG_TOOL_DETAILS=1`인 경우에만 포함됩니다.
* `plugin.version`: 마켓플레이스 항목에 선언된 플러그인 버전. 서드파티 마켓플레이스의 경우 `OTEL_LOG_TOOL_DETAILS=1`인 경우에만 포함됩니다.
* `marketplace.name`: 플러그인이 설치된 마켓플레이스. 서드파티 마켓플레이스의 경우 `OTEL_LOG_TOOL_DETAILS=1`인 경우에만 포함됩니다.

#### 플러그인 로드 이벤트

세션 시작 시 활성화된 플러그인당 한 번씩 기록됩니다. 이 이벤트를 사용하여 설치 작업 자체를 기록하는 `plugin_installed`를 보완하여 전체 플릿에서 활성화된 플러그인의 재고를 파악하세요.

**이벤트 이름**: `claude_code.plugin_loaded`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"plugin_loaded"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `plugin.name`: 플러그인 이름. 공식 마켓플레이스 및 내장 번들 외의 플러그인의 경우 `OTEL_LOG_TOOL_DETAILS=1`이 아니면 값은 `"third-party"`입니다.
* `marketplace.name`: 알려진 경우 플러그인이 설치된 마켓플레이스. `plugin.name`과 동일한 조건에서 `"third-party"`로 마스킹됩니다.
* `plugin.version`: 플러그인 매니페스트의 버전. 이름이 마스킹되지 않고 매니페스트가 버전을 선언할 때만 포함됩니다.
* `plugin.scope`: 플러그인의 출처 범주: `"official"`, `"org"`, `"user-local"`, `"default-bundle"`
* `enabled_via`: 플러그인이 활성화된 방식: `"default-enable"`, `"org-policy"`, `"seed-mount"`, `"user-install"`
* `plugin_id_hash`: 플러그인 이름과 마켓플레이스의 결정론적 해시로, 구성된 내보내기로만 전송됩니다. 이름을 기록하지 않고도 플릿 전반에 로드된 서로 다른 서드파티 플러그인 수를 계산할 수 있습니다.
* `has_hooks`: 플러그인이 훅을 제공하는지 여부
* `has_mcp`: 플러그인이 MCP 서버를 제공하는지 여부
* `host_owned_mcp`: SDK 호스트가 이 플러그인의 MCP 연결을 관리하고 Claude Code가 플러그인의 MCP 서버 구성 읽기를 건너뛴 경우 `true`, 그렇지 않은 경우 `false`. {/* min-version: 2.1.172 */}Claude Code v2.1.172 이상 필요
* `skill_path_count`: 플러그인이 선언한 스킬 디렉터리 수
* `command_path_count`: 플러그인이 선언한 명령 디렉터리 수
* `agent_path_count`: 플러그인이 선언한 에이전트 디렉터리 수
* `safe_mode`: 세션이 [`--safe-mode`](/docs/en/cli-reference)로 시작된 경우 `"true"`, 그렇지 않은 경우 `"false"`. 안전 모드에서 이 이벤트는 구성된 재고만 보고하며 플러그인의 명령, 스킬, 훅, MCP 서버는 로드되지 않습니다. {/* min-version: 2.1.169 */}Claude Code v2.1.169 이상 필요

#### 스킬 활성화 이벤트

Claude가 Skill 도구를 통해 스킬을 호출하거나 사용자가 `/` 명령으로 스킬을 실행할 때 기록됩니다.

**이벤트 이름**: `claude_code.skill_activated`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"skill_activated"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `skill.name`: 스킬 이름. 사용자 정의 및 서드파티 플러그인 스킬의 경우 `OTEL_LOG_TOOL_DETAILS=1`이 아니면 값은 자리표시자 `"custom_skill"`입니다.
* `invocation_trigger`: 스킬이 트리거된 방식 (`"user-slash"`, `"claude-proactive"`, `"nested-skill"`)
* `skill.source`: 스킬이 로드된 위치 (예: `"bundled"`, `"userSettings"`, `"projectSettings"`, `"plugin"`)
* `skill.kind`: 스킬이 워크플로 스킬인 경우 `"workflow"`. 그렇지 않은 경우 없음
* `plugin.name` (`OTEL_LOG_TOOL_DETAILS=1`이거나 플러그인이 공식 마켓플레이스 출신인 경우): 스킬이 플러그인에서 제공되는 경우 소유 플러그인의 이름
* `marketplace.name` (`OTEL_LOG_TOOL_DETAILS=1`이거나 플러그인이 공식 마켓플레이스 출신인 경우): 스킬이 플러그인에서 제공되는 경우 소유 플러그인이 설치된 마켓플레이스

#### 멘션(@) 이벤트

Claude Code가 프롬프트에서 `@`-멘션을 분석할 때 기록됩니다. 모든 멘션이 이벤트를 내보내는 것은 아닙니다. 권한 거부, 대용량 파일, PDF 참조 첨부파일, 디렉터리 목록 가져오기 실패와 같은 조기 종료 경로는 로깅 없이 반환됩니다.

**이벤트 이름**: `claude_code.at_mention`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"at_mention"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `mention_type`: 멘션 유형 (`"file"`, `"directory"`, `"agent"`, `"mcp_resource"`)
* `success`: 멘션이 성공적으로 분석되었는지 여부 (`"true"` 또는 `"false"`)

#### API 재시도 소진 이벤트

API 요청이 1회 초과의 시도 끝에 실패할 때 한 번 기록됩니다. 최종 `api_error` 이벤트와 함께 내보내집니다.

**이벤트 이름**: `claude_code.api_retries_exhausted`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"api_retries_exhausted"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `model`: 사용된 모델
* `error`: 최종 오류 메시지
* `status_code`: 숫자로 표시된 HTTP 상태 코드. 비 HTTP 오류의 경우 없습니다.
* `total_attempts`: 시도한 총 횟수
* `total_retry_duration_ms`: 모든 시도에 걸친 총 경과 시간
* `speed`: `"fast"` 또는 `"normal"`

#### 훅 등록 이벤트

세션 시작 시 구성된 훅당 한 번씩 기록됩니다. 이 이벤트를 사용하여 실행별 `hook_execution_start` 및 `hook_execution_complete` 이벤트를 보완하여 전체 플릿에서 활성화된 훅의 재고를 파악하세요.

**이벤트 이름**: `claude_code.hook_registered`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"hook_registered"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `hook_event`: `"PreToolUse"` 또는 `"PostToolUse"`와 같은 훅 이벤트 유형
* `hook_type`: 훅 구현 유형: `"command"`, `"prompt"`, `"mcp_tool"`, `"http"`, `"agent"`
* `hook_source`: 훅이 정의된 위치: `"userSettings"`, `"projectSettings"`, `"localSettings"`, `"flagSettings"`, `"policySettings"`, `"pluginHook"`
* `safe_mode`: 세션이 [`--safe-mode`](/docs/en/cli-reference)로 시작된 경우 `"true"`, 그렇지 않은 경우 `"false"`. {/* min-version: 2.1.169 */}Claude Code v2.1.169 이상 필요
* `hook_matcher` (`OTEL_LOG_TOOL_DETAILS=1`인 경우): 훅 구성에서 설정된 경우 일치기(matcher) 문자열
* `plugin.name` (`hook_source`가 `"pluginHook"`인 경우): 기여 플러그인의 이름. 공식 마켓플레이스 및 내장 번들 외의 플러그인의 경우 `OTEL_LOG_TOOL_DETAILS=1`이 아니면 값은 `"third-party"`입니다.
* `plugin_id_hash` (`hook_source`가 `"pluginHook"`인 경우): 플러그인 이름과 마켓플레이스의 결정론적 해시로, 구성된 내보내기로만 전송됩니다. 이름을 기록하지 않고 기여 플러그인 수를 계산할 수 있습니다.

#### 훅 실행 시작 이벤트

하나 이상의 훅이 훅 이벤트 실행을 시작할 때 기록됩니다.

**이벤트 이름**: `claude_code.hook_execution_start`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"hook_execution_start"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `hook_event`: `"PreToolUse"` 또는 `"PostToolUse"`와 같은 훅 이벤트 유형
* `hook_name`: 일치기를 포함한 전체 훅 이름 (예: `"PreToolUse:Write"`)
* `num_hooks`: 일치하는 훅 명령 수
* `managed_only`: 관리형 정책 훅만 허용되는 경우 `"true"`
* `hook_source`: `"policySettings"` 또는 `"merged"`
* `safe_mode`: 세션이 [`--safe-mode`](/docs/en/cli-reference)로 시작된 경우 `"true"`, 그렇지 않은 경우 `"false"`. {/* min-version: 2.1.169 */}Claude Code v2.1.169 이상 필요
* `hook_definitions`: JSON 직렬화된 훅 구성. 상세 베타 추적과 `OTEL_LOG_TOOL_DETAILS=1`이 모두 활성화된 경우에만 포함됩니다.

#### 훅 실행 완료 이벤트

훅 이벤트에 대한 모든 훅이 완료되었을 때 기록됩니다.

**이벤트 이름**: `claude_code.hook_execution_complete`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"hook_execution_complete"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `hook_event`: 훅 이벤트 유형
* `hook_name`: 일치기를 포함한 전체 훅 이름
* `num_hooks`: 일치하는 훅 명령 수
* `num_success`: 성공적으로 완료된 수
* `num_blocking`: 차단 결정을 반환한 수
* `num_non_blocking_error`: 차단 없이 실패한 수
* `num_cancelled`: 완료 전에 취소된 수
* `total_duration_ms`: 일치하는 모든 훅의 경과 시간(ms)
* `managed_only`: 관리형 정책 훅만 허용되는 경우 `"true"`
* `hook_source`: `"policySettings"` 또는 `"merged"`
* `safe_mode`: 세션이 [`--safe-mode`](/docs/en/cli-reference)로 시작된 경우 `"true"`, 그렇지 않은 경우 `"false"`. {/* min-version: 2.1.169 */}Claude Code v2.1.169 이상 필요
* `hook_definitions`: JSON 직렬화된 훅 구성. 상세 베타 추적과 `OTEL_LOG_TOOL_DETAILS=1`이 모두 활성화된 경우에만 포함됩니다.

#### 훅 플러그인 메트릭 이벤트

공식 마켓플레이스 플러그인 훅이 호출별 메트릭을 내보낼 때 기록됩니다. 공식 Anthropic 마켓플레이스에서 설치된 플러그인만 이를 내보낼 수 있습니다. 서드파티 마켓플레이스 플러그인 및 사용자 구성 훅은 이 이벤트로 내보내지 않습니다. 이 이벤트를 사용하여 자체 관찰 가능성 스택에서 검색 비율, 비용, 지속 시간과 같은 플러그인 동작을 모니터링하세요.

**이벤트 이름**: `claude_code.hook_plugin_metrics`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"hook_plugin_metrics"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `plugin_id`: `<name>@<marketplace>` 형식의 플러그인 식별자
* `hook_event`: 메트릭을 내보낸 훅 이벤트 유형
* 최대 20개의 플러그인 내보내기 메트릭 키. 이름은 `^[a-z][a-z0-9_]{0,39}$`와 일치합니다. 값은 불리언 또는 숫자입니다.

#### 압축(Compaction) 이벤트

대화 압축이 완료될 때 기록됩니다.

**이벤트 이름**: `claude_code.compaction`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"compaction"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `trigger`: `"auto"` 또는 `"manual"`
* `success`: `"true"` 또는 `"false"`
* `duration_ms`: 압축 지속 시간
* `pre_tokens`: 압축 전 근사 토큰 수
* `post_tokens`: 압축 후 근사 토큰 수
* `error`: 압축 실패 시의 오류 메시지
* `precompute_reuse`: `trigger`가 `"manual"`일 때만 설정됩니다. 자동 압축은 컨텍스트 창이 채워지기 전에 백그라운드에서 요약을 준비할 수 있으며, 이 속성은 `/compact`가 준비된 요약을 재사용했는지 여부를 기록합니다. `"hit"`은 재사용되었음을 의미하며, `"miss_custom_instructions"`, `"miss_hook"`, `"miss_not_ready"`는 대신 새로 요약을 계산한 이유를 나타냅니다. {/* min-version: 2.1.153 */}Claude Code v2.1.153 이상 필요

#### 피드백 설문조사 이벤트

세션 품질 설문조사가 표시되거나 답변되었을 때 기록됩니다. 설문조사가 수집하는 내용과 이를 제어하는 방법은 [세션 품질 설문조사](/docs/en/data-usage#session-quality-surveys)를 참조하세요.

**이벤트 이름**: `claude_code.feedback_survey`

**속성**:

* 모든 [표준 속성](#standard-attributes)
* `event.name`: `"feedback_survey"`
* `event.timestamp`: ISO 8601 타임스탬프
* `event.sequence`: 세션 내에서 이벤트 순서를 지정하기 위한 단조 증가 카운터
* `event_type`: `"appeared"`, `"responded"`, `"transcript_prompt_appeared"` 등 설문조사 수명주기 이벤트
* `appearance_id`: 하나의 설문조사 인스턴스에 대해 내보낸 이벤트를 연결하는 고유 ID
* `survey_type`: 이벤트를 생성한 설문조사 종류. `"session"`은 "Claude의 성능이 어떠한가요?" 평가 프롬프트입니다
* `response`: `responded` 이벤트에서 사용자의 선택 사항
* `enabled_via_override`: [`CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL`](/docs/en/env-vars)이 설정된 경우 `true`. 문자열이 아닌 불리언으로 내보내집니다. `session` 설문조사 이벤트에 존재합니다. 플릿 전반에 재정의가 적용되었는지 확인하려면 이 속성으로 필터링하세요

## 메트릭 및 이벤트 데이터 해석

내보낸 메트릭과 이벤트는 다양한 분석을 지원합니다:

### 사용량 모니터링

| 메트릭 | 분석 기회 |
| --- | --- |
| `claude_code.token.usage` | `type`(입력/출력), 사용자, 팀, 모델, `skill.name`, `plugin.name`, `agent.name`별 세분화 |
| `claude_code.session.count` | 시간에 따른 채택 및 참여 추적 |
| `claude_code.lines_of_code.count` | 모델별로 구분하여 코드 추가 및 제거를 추적함으로써 생산성 측정 |
| `claude_code.commit.count` & `claude_code.pull_request.count` | 개발 워크플로에 미치는 영향 이해 |

### 비용 모니터링

`claude_code.cost.usage` 메트릭은 다음 작업에 도움이 됩니다:

* 팀 또는 개인 전체의 사용 트렌드 추적
* 최적화를 위해 사용량이 높은 세션 식별
* `skill.name`, `plugin.name`, `agent.name` 속성을 통해 지출을 특정 스킬, 플러그인 또는 서브에이전트 유형에 귀속

<Note>
  비용 메트릭은 근사치입니다. 공식 청구 데이터는 API 공급자(Claude Console, Amazon Bedrock 또는 Google Cloud Agent Platform)를 참조하세요.
</Note>

Claude Code는 `ANTHROPIC_BASE_URL` 뒤의 게이트웨이나 프록시가 여러 프레임에 걸쳐 사용량을 점진적으로 스트리밍하는 경우를 포함하여 각 스트리밍 응답을 비용 및 토큰 메트릭에 정확히 한 번만 반영합니다. {/* min-version: 2.1.214 */}v2.1.214 이전에는 둘 이상의 프레임에 사용량을 포함하는 스트림이 추가 프레임당 약 1회의 전체 요청만큼 `claude_code.cost.usage` 및 `claude_code.token.usage`를 부풀렸습니다.

### 알림 및 세분화

고려해야 할 일반적인 알림:

* 비용 급증
* 비정상적인 토큰 소비
* 특정 사용자의 높은 세션 볼륨

모든 메트릭은 [표준 속성](#standard-attributes)으로 세분화할 수 있습니다. `model` 속성은 `claude_code.token.usage`, `claude_code.cost.usage` 및 {/* min-version: 2.1.172 */}v2.1.172부터 `claude_code.lines_of_code.count`에서 사용할 수 있습니다.

한 세션이 여러 모델에 걸칠 수 있으므로 커밋의 모델별 세분화는 `session.id`로 토큰 또는 비용 메트릭과 조인하여 근사치로만 구할 수 있습니다. 보조 요청과 서브에이전트 요청이 세션의 커밋을 이를 작성하지 않은 모델 탓으로 돌리지 않도록 토큰 또는 비용 측면을 `query_source`가 `"main"`인 행으로 필터링하세요.

### 재시도 소진 감지

Claude Code는 실패한 API 요청을 내부적으로 재시도하며 포기한 후에만 단일 `claude_code.api_error` 이벤트를 내보내므로 이벤트 자체가 해당 요청의 최종 시널입니다. 중간 재시도 시도는 별도의 이벤트로 기록되지 않습니다.

이벤트의 `attempt` 속성은 총 시도 횟수를 기록합니다. `CLAUDE_CODE_MAX_RETRIES`의 기본값은 10이고 최대 15로 제한됩니다. {/* min-version: 2.1.199 */}v2.1.199부터 `CLAUDE_CODE_RETRY_WATCHDOG`가 기본값을 높이고 제한을 제거합니다. 일시적인 오류로 요청이 모든 재시도를 소진하면 `attempt`는 해당 유효 제한보다 1이 더 큽니다(기본적으로 11, 와치독이 설정되지 않은 경우 16 이하). 더 낮은 값은 `400` 응답과 같이 재시도할 수 없는 오류를 나타냅니다.

복구된 세션과 중단된 세션을 구분하려면 이벤트를 `session.id`로 그룹화하고 오류 발생 후에 이후 `api_request` 이벤트가 존재하는지 확인하세요.

### 이벤트 분석

이벤트 데이터는 Claude Code 상호작용에 대한 자세한 통찰력을 제공합니다:

**도구 사용 패턴**: 도구 결과 이벤트를 분석하여 다음을 식별합니다:

* 가장 자주 사용되는 도구
* 도구 성공률
* 평균 도구 실행 시간
* 도구 유형별 오류 패턴

**성능 모니터링**: API 요청 지속 시간과 도구 실행 시간을 추적하여 성능 병목 현상을 식별합니다.

## 보안 이벤트 감사

OpenTelemetry 이벤트는 Claude Code 활동에 대한 감사 데이터 소스입니다. 모든 이벤트에는 도구 호출, MCP 활동 및 권한 결정을 이를 트리거한 사용자에 다시 연결하는 신원 속성이 포함되어 있습니다. OTLP 로그 내보내기는 이러한 이벤트를 OTLP 수신기가 있는 시큐리티 정보 및 이벤트 관리(SIEM) 플랫폼이나 SIEM으로 전달하는 OpenTelemetry Collector로 전달할 수 있습니다.

### 사용자에게 작업 귀속

각 이벤트의 [표준 속성](#standard-attributes)에는 인증된 사용자의 신원이 포함됩니다. Claude 계정으로 로그인한 경우 `user.email`, `user.account_uuid`, `user.account_id`, `organization.id`가 포함되며, `user.id` 및 세션별 `session.id`가 포함됩니다. `user.id`는 설치 범위의 식별자입니다([Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션은 제외하며, 게이트웨이 발급 토큰의 IdP 주체입니다).

따라서 MCP 도구 호출, Bash 명령 및 파일 편집은 세션을 시작한 개발자의 소관입니다. Claude Code는 별도의 서비스 계정으로 작동하지 않습니다. 각 이벤트에 기록된 신원은 개발자 자신의 Claude 계정이거나 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션에서의 개발자의 IdP 신원입니다.

Claude Code가 직접 API 키로 인증하거나 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry에서 인증하는 경우 세션에 Claude 계정이 없으며 `user.id` 및 `session.id`만 채워집니다. 이러한 배포에서는 [관리형 설정](#administrator-configuration) 파일 또는 래퍼를 통해 사용자별로 설정된 `OTEL_RESOURCE_ATTRIBUTES`를 사용하여 사용자 신원을 직접 첨부하세요. Claude 앱 게이트웨이 세션은 이러한 작업이 필요하지 않습니다. [표준 속성](#standard-attributes)에 설명된 대로 CLI가 IdP 신원을 자동으로 스탬프합니다.

```bash theme={null}
export OTEL_RESOURCE_ATTRIBUTES="enduser.id=jdoe@example.com,enduser.directory_id=S-1-5-21-..."
```

### MCP 활동 감사

전체 호출 세부 정보와 함께 MCP 서버 활동을 캡처하려면 로그 내보내기를 활성화하고 `OTEL_LOG_TOOL_DETAILS=1`을 설정하세요. 그러면 각 MCP 작업은 표준 신원 속성과 함께 서버 이름, 도구 이름 및 호출 인수를 전달하는 구조화된 이벤트를 생성합니다:

| 이벤트 | MCP에 대해 기록하는 내용 |
| --- | --- |
| `mcp_server_connection` | `server_name`, `transport_type`, `server_scope`, 오류 세부 정보가 포함된 서버 연결, 연결 해제 및 연결 실패 |
| `tool_result` | `tool_name` 및 `mcp_server_scope`, `mcp_server_name` 및 `mcp_tool_name`이 포함된 `tool_parameters` 페이로드, 호출 인수가 포함된 `tool_input` 페이로드가 있는 각 MCP 도구 호출 |
| `tool_decision` | 호출의 허용/거부 여부, 결정이 설정, 훅, 사용자 중 어디서 유래했는지 여부, `mcp_server_name` 및 `mcp_tool_name`을 포함하는 `tool_parameters` 페이로드 |

`OTEL_LOG_TOOL_DETAILS`가 없으면 이러한 이벤트는 식별 세부 정보를 삭제합니다:

* `tool_result`: 사용자 구성 서버의 경우 `mcp_server_scope` 및 문자 그대로 `"mcp_tool"`로 마스킹된 `tool_name`을 유지하고 인수 내용은 생략합니다. Claude Desktop 내장 서버의 경우 Claude Desktop이 소유한 세션에서 `tool_decision`과 동일한 호스트 작성 예외로 `tool_parameters` 내부의 `mcp_server_name`/`mcp_tool_name` 쌍을 유지합니다. {/* min-version: 2.1.214 */}Claude Code v2.1.214 이상 필요
* `tool_decision`: 사용자 구성 서버의 경우 `tool_source` 및 문자 그대로 `"mcp_tool"`로 마스킹된 `tool_name`을 유지하고 인수 내용은 생략합니다. Claude Desktop 내장 서버의 경우 Claude Desktop이 소유한 세션에서 `tool_parameters` 내부의 `mcp_server_name`/`mcp_tool_name` 쌍도 유지합니다. {/* min-version: 2.1.214 */}`tool_source` 및 이름 쌍 모두 Claude Code v2.1.214 이상 필요
* `mcp_server_connection`: `server_name` 및 오류 메시지는 생략하지만 `is_plugin`, `plugin_id_hash`, `plugin.name`을 유지하며, 비 Anthropic 플러그인 이름은 문자 그대로 `"third-party"`로 마스킹되어 상세 로깅 없이도 플러그인이 제공하는 서버를 구분할 수 있도록 합니다.

### 보안 질문을 이벤트에 매핑

감지 규칙을 구축할 때 모니터링할 신호를 확인하고 백엔드에서 해당 이벤트 및 속성을 쿼리하세요:

| 신호 | 이벤트 | 핵심 속성 |
| --- | --- | --- |
| 도구 호출 허용/거부 및 이유 | `tool_decision` | `decision`, `source`, `tool_name`, `tool_parameters` |
| 권한 모드 에스컬레이션 | `permission_mode_changed` | `from_mode`, `to_mode`, `trigger` |
| 정책 훅이 작업을 차단함 | `hook_execution_complete` | `hook_event`, `num_blocking` |
| 로그인, 로그아웃 및 인증 실패 | `auth` | `action`, `success`, `error_category` |
| MCP 서버 연결 또는 실패 | `mcp_server_connection` | `status`, `server_name`, `is_plugin`, `error_code` |
| 플러그인 설치 및 소스 | `plugin_installed` | `plugin.name`, `marketplace.name`, `marketplace.is_official` |
| 실행된 명령 및 연관 파일 | `OTEL_LOG_TOOL_DETAILS=1` 상태의 `tool_result`(실행됨) 또는 `tool_decision`(거부됨) | `tool_parameters`; `tool_input` (`tool_result`만 해당) |

Claude Code는 원시 이벤트 스트림만 내보냅니다. 이상 감지, 기준선 설정, 세션 간 상관관계 분석 및 알림 설정은 SIEM 또는 관찰 가능성 백엔드의 책임입니다.

### SIEM으로 이벤트 전송

`OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`를 SIEM의 OTLP 수신기 또는 SIEM의 자체 수집 API로 전달하는 OpenTelemetry Collector를 가리키도록 설정하세요. 다음 관리형 설정 예시는 MCP 및 Bash 감사를 위해 전체 도구 세부 정보가 활성화된 상태로 이벤트만 내보냅니다:

```json theme={null}
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_LOG_TOOL_DETAILS": "1",
    "OTEL_EXPORTER_OTLP_LOGS_PROTOCOL": "http/protobuf",
    "OTEL_EXPORTER_OTLP_LOGS_ENDPOINT": "https://siem.example.com:4318/v1/logs",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer your-siem-token"
  }
}
```

## 백엔드 고려 사항

메트릭, 로그 및 추적 백엔드의 선택에 따라 수행할 수 있는 분석 유형이 결정됩니다:

### 메트릭용

* **시계열 데이터베이스 (예: Prometheus)**: 비율 계산, 집계된 메트릭
* **컬럼형 저장소 (예: ClickHouse)**: 복잡한 쿼리, 고유 사용자 분석
* **풀 스택 관찰 가능성 플랫폼 (예: Honeycomb, Datadog, Grafana Cloud)**: 고급 쿼리, 시각화, 알림

### 이벤트/로그용

* **로그 집계 시스템 (예: Elasticsearch, Loki)**: 전문 검색, 로그 분석
* **컬럼형 저장소 (예: ClickHouse)**: 구조화된 이벤트 분석
* **풀 스택 관찰 가능성 플랫폼 (예: Honeycomb, Datadog, Grafana Cloud)**: 메트릭과 이벤트 간 상관관계 분석

### 추적용

분산 추적 저장 및 스팬 상관관계를 지원하는 백엔드를 선택하세요:

* **분산 추적 시스템 (예: Jaeger, Zipkin, Grafana Tempo)**: 스팬 시각화, 요청 폭포수(waterfall) 차트, 지연 시간 분석
* **풀 스택 관찰 가능성 플랫폼 (예: Honeycomb, Datadog, Grafana Cloud)**: 추적 검색 및 메트릭/로그와의 상관관계 분석

일별/주별/월별 활성 사용자(DAU/WAU/MAU) 메트릭이 필요한 조직의 경우 고유 값 쿼리를 효율적으로 지원하는 백엔드를 고려하세요.

## 서비스 정보

모든 메트릭과 이벤트는 다음 리소스 속성과 함께 내보내집니다:

* `service.name`: 터미널 세션의 경우 `claude-code`, [Claude Desktop 앱](/docs/en/desktop)의 Code 탭에서 시작된 세션의 경우 `claude-code-desktop`
* `service.version`: 현재 Claude Code 버전, 또는 Code 탭 세션의 경우 Desktop 앱 버전
* `os.type`: 운영체제 유형 (예: `linux`, `darwin`, `windows`)
* `os.version`: 운영체제 버전 문자열
* `host.arch`: 호스트 아키텍처 (예: `amd64`, `arm64`)
* `wsl.version`: WSL 버전 번호 (Windows Subsystem for Linux에서 실행 중인 경우에만 존재)
* 계측기 이름 (Meter Name): `com.anthropic.claude_code`

수집기 파이프라인이나 대시보드가 `service.name = claude-code`로 필터링하는 경우, Code 탭 세션의 텔레메트리도 캡처하려면 필터에 `claude-code-desktop`을 추가하세요.

## ROI 측정 리소스

텔레메트리 설정, 비용 분석, 생산성 메트릭, 자동화된 보고서를 포함하여 Claude Code의 투자 수익률(ROI) 측정에 대한 포괄적인 가이드는 [Claude Code ROI Measurement Guide](https://github.com/anthropics/claude-code-monitoring-guide)를 참조하세요. 이 리포지토리는 바로 사용 가능한 Docker Compose 구성, Prometheus 및 OpenTelemetry 설정, Linear와 같은 도구와 통합된 생산성 보고서 생성 템플릿을 제공합니다.

## 보안 및 개인정보 보호

* 백엔드로의 OpenTelemetry 내보내기는 옵트인 방식이며 명시적인 구성이 필요합니다. Anthropic의 별도 운영 텔레메트리와 이를 비활성화하는 방법은 [데이터 사용량](/docs/en/data-usage#telemetry-services)을 참조하세요
* 원시 파일 내용 및 코드 스니펫은 메트릭 또는 이벤트에 포함되지 않습니다. 추적 스팬은 별도의 데이터 경로입니다. 아래 `OTEL_LOG_TOOL_CONTENT` 항목을 참조하세요
* OAuth를 통해 인증된 경우 `user.email`이 텔레메트리 속성에 포함됩니다. 이것이 조직의 관심사인 경우 텔레메트리 백엔드와 협력하여 이 필드를 필터링하거나 마스킹하세요
* 사용자 프롬프트 내용은 기본적으로 수집되지 않습니다. 프롬프트 길이만 기록됩니다. 프롬프트 내용을 포함하려면 `OTEL_LOG_USER_PROMPTS=1`을 설정하세요
* 어시스턴트 응답 텍스트는 기본적으로 수집되지 않습니다. 응답 길이만 기록됩니다. 응답 텍스트를 포함하려면 `OTEL_LOG_ASSISTANT_RESPONSES=1`을 설정하세요. Claude Code의 모든 OpenTelemetry 데이터와 마찬가지로 응답 텍스트는 구성한 OTel 엔드포인트로만 전송되며 Anthropic에는 절대 전송되지 않습니다. 이 변수가 설정되지 않은 경우 `OTEL_LOG_USER_PROMPTS`가 대체 수단으로 사용되므로 응답 내용 없이 프롬프트 내용만 원하는 경우 `OTEL_LOG_ASSISTANT_RESPONSES=0`을 설정하세요
* 도구 입력 인수 및 매개변수는 기본적으로 기록되지 않습니다. 포함하려면 `OTEL_LOG_TOOL_DETAILS=1`을 설정하세요. Claude Desktop의 내장 서버의 경우 Claude Desktop이 소유한 세션에서 `tool_decision` 및 `tool_result`가 인수 내용이 아닌 호스트 작성 이름인 `mcp_server_name`/`mcp_tool_name` 쌍을 플래그가 꺼져 있더라도 전달합니다. {/* min-version: 2.1.214 */}이 예외에는 Claude Code v2.1.214 이상이 필요합니다. 이 데이터는 구성한 OTEL 엔드포인트로만 전송되며 Anthropic에는 절대 전송되지 않습니다. 인수에 민감한 값이 계속 포함될 수 있으므로 필요한 경우 이러한 속성을 필터링하거나 마스킹하도록 텔레메트리 백엔드를 구성하세요. 활성화된 경우:
  * `tool_result` 및 `tool_decision` 이벤트에는 Bash 명령, MCP 서버 및 도구 이름, 스킬 이름이 포함된 `tool_parameters` 속성이 포함됩니다. `full_command`와 같은 필드는 잘리지 않고 내보내집니다
  * `tool_result` 이벤트에는 추가로 파일 경로, URL, 검색 패턴 및 기타 인수가 포함된 `tool_input` 속성이 포함됩니다. 512자를 초과하는 개별 값은 잘리며 전체는 \~4K 자로 제한됩니다
  * `user_prompt` 이벤트에는 커스텀, 플러그인, MCP 명령에 대한 문자 그대로의 `command_name`이 포함됩니다
  * 추적 스팬에는 동일한 `tool_input` 속성 및 `file_path`와 같이 입력에서 파생된 속성이 `tool_input`과 동일하게 잘려 포함됩니다
* 도구 입력 및 출력 내용은 기본적으로 추적 스팬에 기록되지 않습니다. 포함하려면 `OTEL_LOG_TOOL_CONTENT=1`을 설정하세요. 활성화되면 스팬 이벤트에 속성당 내용 제한(기본값 60KB)에서 잘린 전체 도구 입력 및 출력 내용이 포함됩니다. 여기에는 Read 도구 결과의 원시 파일 내용과 Bash 명령 출력이 포함될 수 있습니다. 필요한 경우 이러한 속성을 필터링하거나 마스킹하도록 텔레메트리 백엔드를 구성하세요
* 원시 Anthropic Messages API 요청 및 응답 본문은 기본적으로 기록되지 않습니다. 포함하려면 `OTEL_LOG_RAW_API_BODIES`를 설정하세요. `=1`인 경우 각 API 호출은 JSON 직렬화된 페이로드인 `body` 속성이 내용 제한(기본값 60KB)에서 잘린 `api_request_body` 및 `api_response_body` 로그 이벤트를 내보냅니다. `=file:<dir>`인 경우 잘리지 않은 본문은 해당 디렉터리 아래의 `.request.json` 및 `.response.json` 파일에 기록되며 이벤트는 인라인 본문 대신 `body_ref` 경로를 전달합니다. 텔레메트리 스트림을 통하는 대신 로그 수집기나 사이드카로 디렉터리를 전달하세요. 두 모드 모두에서 본문에는 시스템 프롬프트, 이전의 모든 사용자 및 어시스턴트 턴, 도구 결과를 포함한 전체 대화 기록이 포함되므로 이를 활성화하면 다른 `OTEL_LOG_*` 콘텐츠 플래그가 노출하는 모든 내용에 동의함을 의미합니다. Claude의 확장 사고 내용은 다른 설정에 관계없이 이 본문에서 항상 마스킹됩니다

## Amazon Bedrock에서 Claude Code 모니터링

Amazon Bedrock의 상세한 Claude Code 사용 모니터링 가이드는 [Claude Code Monitoring Implementation (Amazon Bedrock)](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock/blob/main/assets/docs/MONITORING.md)을 참조하세요.
