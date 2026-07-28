> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# Agent SDK 호스팅하기

> 프로덕션 환경에서 Agent SDK 배포하기: 서브프로세스 아키텍처, 세션 영속성, 스케일링, 관찰 가능성, 그리고 Docker, Kubernetes, 샌드박스 제공자를 위한 멀티 테넌트 격리.

Agent SDK는 쉘, 작업 디렉토리, 디스크 상의 세션 파일을 소유하는 `claude` CLI 서브프로세스를 생성하고 감독합니다. 이를 호스팅하는 것은 무상태 API 래퍼를 호스팅하는 것과 다릅니다. 실행 중인 모든 에이전트는 로컬 상태에 묶인 장시간 실행 프로세스이며, 이는 리소스 할당, 세션 유지, 테넌트 간 확장 방식에 영향을 줍니다.

이 페이지에서는 자체 인프라에서의 셀프 호스팅을 다룹니다: [서브프로세스 모델](#서브프로세스-모델) 이해, [세션 패턴 선택](#세션-패턴-선택), [컨테이너 프로비저닝](#컨테이너-프로비저닝), 그리고 영속성, 관찰 가능성, 인증, 멀티 테넌트 격리와 같은 [프로덕션 고려 사항 다루기](#프로덕션-고려-사항-다루기). 배포 가능한 Dockerfile 및 Kubernetes 매니페스트는 [호스팅 쿡북](https://github.com/anthropics/claude-cookbooks/tree/main/claude_agent_sdk/hosting)을 참조하세요.

인프라 제어, 커스텀 격리 또는 자체 데이터 플레인이 필요하지 않은 경우 [Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) 사용을 고려해 보세요: Anthropic이 에이전트와 샌드박스를 실행하는 호스팅형 REST API로, 애플리케이션이 호스팅 인프라 운영 없이 이벤트를 보내고 결과를 실시간으로 받을 수 있습니다.

<Info>
  네트워크 제어, 자격 증명 관리, 격리 옵션을 포함하여 기본 샌드박싱을 넘어선 보안 강화에 대해서는 [안전한 배포](/docs/en/agent-sdk/secure-deployment)를 참조하세요.
</Info>

## 서브프로세스 모델

이 페이지의 모든 호스팅 결정은 SDK가 에이전트를 실행하는 방식에서 비롯됩니다. 코드가 `query()`를 호출할 때 SDK는 별도의 `claude` CLI 프로세스를 생성하고 stdio를 통해 통신합니다. 해당 서브프로세스는 쉘, 작업 디렉토리, 로컬 디스크의 JSONL 세션 트랜스크립트를 소유합니다.

<img src="https://mintcdn.com/claude-code/ikqp3_70mqIahteV/images/agent-sdk/hosting-subprocess.svg?fit=max&auto=format&n=ikqp3_70mqIahteV&q=85&s=9dac857ca9d3b1410c3734900c386004" alt="Request flow: client to your app, which spawns a claude CLI subprocess over stdio inside the container; the subprocess writes to local disk and calls api.anthropic.com over HTTPS" width="920" height="220" data-path="images/agent-sdk/hosting-subprocess.svg" />

하나의 에이전트 세션은 하나의 서브프로세스에 매핑됩니다. N개의 동시 세션을 실행한다는 것은 각각 자체 프로세스 트리와 트랜스크립트 파일을 가진 N개의 서브프로세스를 의미합니다. 기본적으로 이들은 모두 애플리케이션의 작업 디렉토리를 상속받으므로, 세션마다 별도의 파일 시스템이 필요한 경우 각 `query()` 호출에서 `cwd`를 전달하세요:

<CodeGroup>
  ```typescript TypeScript theme={null}
  query({ prompt, options: { cwd: "/work/session-a" } })
  ```

  ```python Python theme={null}
  query(prompt=prompt, options=ClaudeAgentOptions(cwd="/work/session-a"))
  ```
</CodeGroup>

### 로컬 디스크에 상주하는 상태

기본적으로 3가지 종류의 에이전트 상태가 컨테이너의 파일 시스템에 상주합니다. 컨테이너 재시작, 스케일 다운 또는 다른 노드로의 이동 시 이 중 어느 것도 보존되지 않습니다.

| 상태 | 기본 위치 |
| --------------------------- | -------------------------------------------------------------------------------- |
| 세션 트랜스크립트 | `~/.claude/projects/`, 또는 설정된 경우 `CLAUDE_CONFIG_DIR` 아래의 `projects/` 디렉토리 |
| `CLAUDE.md` 메모리 파일 | 사용자 계층의 경우 `~/.claude/CLAUDE.md`, 프로젝트 계층의 경우 세션의 작업 디렉토리 |
| 작업 디렉토리 산출물 | 세션의 작업 디렉토리 |

호스트 간에 트랜스크립트를 보존하려면 [`SessionStore` 어댑터](/docs/en/agent-sdk/session-storage)를 구성하세요. 메모리 파일 및 기타 작업 디렉토리 산출물은 마운트된 볼륨이나 오브젝트 스토리지 동기화와 같은 자체 스토리지 전략이 필요합니다.

API 수준에서 세션, 다시 시작(resumption), 포크(forking)가 작동하는 방식은 [세션](/docs/en/agent-sdk/sessions)을 참조하세요.

## 세션 패턴 선택

이 4가지 패턴은 세션 수명 주기를 다룹니다: 서비스하는 세션 대비 컨테이너의 유지 기간. 컨테이너가 실행되는 위치의 경우 [호스팅 쿡북](https://github.com/anthropics/claude-cookbooks/blob/main/claude_agent_sdk/07_Hosting_the_agent.ipynb)에 로컬 Docker, Modal, Kubernetes용 [배포 가능한 코드](https://github.com/anthropics/claude-cookbooks/tree/main/claude_agent_sdk/hosting)가 준비되어 있습니다. 여기서 세션 패턴을 선택하고 쿡북에서 배포 대상을 선택하세요.

### 단명형 세션 (Ephemeral sessions)

각 사용자 작업에 대해 컨테이너를 생성하고 작업이 완료되면 파기합니다. 단발성 작업에 가장 적합합니다. 작업이 완료되는 동안 사용자가 AI와 계속 상호작용할 수 있지만 작업이 완료되면 컨테이너가 파기됩니다.

예시 워크로드로는 버그 조사 및 수정, 송장 및 영수증 추출, 문서 번역, 미디어 변환 등이 있습니다.

컨테이너는 SDK를 호출하고 종료되는 단발성 진입점(entrypoint)을 실행합니다. 아래 예제는 최소한의 TypeScript 버전을 보여줍니다. 최상위 `await`를 사용할 수 있도록 `entrypoint.mts`로 저장하거나 `package.json`에 `"type": "module"`을 설정하세요.

```typescript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

const prompt = process.env.TASK_PROMPT!;
for await (const message of query({ prompt, options: { maxTurns: 20 } })) {
  console.log(message);
}
```

### 장시간 실행 세션 (Long-running sessions)

지속적인 작업을 제공하기 위해 종종 컨테이너당 여러 SDK 프로세스를 호스팅하는 영구 컨테이너 인스턴스를 실행합니다. 자율 조치를 취하거나, 콘텐츠를 제공하거나, 대용량 메시지 스트림을 처리하는 에이전트에 가장 적합합니다.

예시 워크로드로는 수신 메일을 분류하고 응답하는 이메일 에이전트, 컨테이너 포트를 통해 사용자별 편집 가능한 사이트를 호스팅하는 사이트 빌더, Slack과 같은 플랫폼의 지속적인 트래픽을 처리하는 챗봇 등이 있습니다.

컨테이너는 HTTP 또는 WebSocket 엔드포인트를 노출하고 각 활성 세션을 장시간 실행 쿼리와 그 뒤의 서브프로세스에 매핑합니다. TypeScript에서는 [`streamInput()`](/docs/en/agent-sdk/typescript#query-object)을 사용하여 활성 세션에 턴을 추가하고 [`startup()`](/docs/en/agent-sdk/typescript#startup)을 사용하여 수신 트래픽에 앞서 서브프로세스를 웜업(pre-warm)합니다. Python에서는 [`ClaudeSDKClient`](/docs/en/agent-sdk/python#claudesdkclient)를 사용하여 여러 턴에 걸쳐 세션을 열어둡니다. 최대 동시 세션 수를 메모리에 유지할 수 있도록 컨테이너 크기를 조정하세요.

### 하이브리드 세션 (Hybrid sessions)

시작 시 [`SessionStore`](/docs/en/agent-sdk/session-storage)에서 복원(hydrate)되고 업데이트를 다시 지속화하는 단명형 컨테이너입니다. 여러 상호작용에 걸쳐 실행되지만 그 사이에 유휴 상태로 대기하는 세션에 가장 적합합니다. 컨테이너는 유휴 시간 동안 다운되고 사용자가 돌아올 때 다시 실행됩니다.

예시 워크로드로는 간헐적인 점검이 이루어지는 개인 프로젝트 관리자, 수시간에 걸쳐 일시 중지 및 재개되는 심층 조사, 여러 상호작용에 걸쳐 티켓 기록을 로드하는 고객 지원 에이전트 등이 있습니다.

사용자가 얼마나 자주 돌아올 것으로 예상되는지에 따라 제공자의 유휴 타임아웃을 조정하세요. `SessionStore` 구성 없이 컨테이너를 종료하면 트랜스크립트가 함께 손실되므로 이 패턴에서는 스토어가 필수 항목입니다.

이 패턴은 공유 스토어가 연결된 상태에서 ID로 세션을 다시 시작하는 방식을 중심으로 작동합니다:

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query, type SessionStore } from "@anthropic-ai/claude-agent-sdk";

  declare const userInput: string;
  declare const sessionId: string;          // 사용자별로 데이터베이스에서 조회함
  declare const sessionStore: SessionStore; // S3, Redis, Postgres 또는 자체 어댑터

  for await (const message of query({
    prompt: userInput,
    options: { resume: sessionId, sessionStore },
  })) {
    // ...
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions, SessionStore
  import asyncio

  user_input: str = ...
  session_id: str = ...              # 사용자별로 데이터베이스에서 조회함
  session_store: SessionStore = ...  # S3, Redis, Postgres 또는 자체 어댑터


  async def main():
      async for message in query(
          prompt=user_input,
          options=ClaudeAgentOptions(
              resume=session_id,
              session_store=session_store,
          ),
      ):
          ...


  asyncio.run(main())
  ```
</CodeGroup>

전체 `SessionStore` 인터페이스 및 참조 어댑터는 [세션 저장소](/docs/en/agent-sdk/session-storage)를 참조하세요.

### 멀티 에이전트 컨테이너 (Multi-agent container)

하나의 컨테이너 내부에서 여러 SDK 서브프로세스를 실행합니다. 공유 환경에서 에이전트들이 서로 상호작용하는 멀티 에이전트 시뮬레이션과 같이 밀접하게 협력해야 하는 에이전트에 가장 적합합니다.

각 에이전트에 자체 작업 디렉토리를 부여하여 서로의 파일을 덮어쓰지 않도록 하고, 에이전트별 `CLAUDE.md` 파일이 에이전트 간에 누출되지 않도록 설정 로딩을 격리하세요. 구체적인 옵션은 [멀티 테넌트 격리](#멀티-테넌트-격리)를 참조하세요.

## 컨테이너 프로비저닝

### 컨테이너 기반 샌드박싱

프로세스 격리, 리소스 제한, 네트워크 제어, 단명형 파일 시스템을 위해 샌드박스 컨테이너 내부에서 SDK를 실행하세요. 여러 제공자가 Agent SDK 모델에 적합한 샌드박스 컨테이너 환경을 전문으로 제공합니다.

제공자 선택 시 답변해야 할 질문:

* **샌드박스를 주도하는 주체**: 샌드박스 서비스 제공자(Sandbox-as-a-Service)는 인프라를 대신 운영해주고, 셀프 호스팅 옵션은 자체 인프라에서 실행할 소프트웨어를 제공함.
* **콜드 스타트 지연 시간**: "샌드박스 생성"부터 "첫 번째 요청 수신 준비 완료"까지의 시간. 단명형 패턴에는 초 미만의 시작 시간이 필요함. 장시간 실행 패턴은 더 많은 시간을 감수할 수 있음.
* **영구 스토리지**: 제공자가 영구 볼륨을 제공하는지, 단명형 디스크만 제공하는지 여부. 하이브리드 패턴은 샌드박스 내부든 그 옆이든 어딘가에 영구 스토리지가 필요함.
* **가격 모델**: 초당, 요청당, 또는 평탄한 시간당 청구. 초당 가격은 버스트(bursty) 단명형 워크로드에 적합함. 시간당 가격은 장시간 실행 세션에 적합함.
* **네트워킹**: 커스텀 송출(egress) 규칙, 아웃바운드 프록시, 규제 환경을 위한 전용 VPC 피어링 지원.

평가할 제공자 목록:

* [Modal Sandbox](https://modal.com/docs/guide/sandbox), [데모 구현 예시](https://modal.com/docs/examples/claude-slack-gif-creator) 포함
* [Cloudflare Sandboxes](https://github.com/cloudflare/sandbox-sdk)
* [Daytona](https://www.daytona.io/)
* [E2B](https://e2b.dev/)
* [Fly Machines](https://fly.io/docs/machines/)
* [Vercel Sandbox](https://vercel.com/docs/functions/sandbox)

Docker, gVisor, Firecracker 등의 셀프 호스팅 옵션 및 상세 격리 구성은 [격리 기술](/docs/en/agent-sdk/secure-deployment#isolation-technologies)을 참조하세요.

### 런타임 종속성

컨테이너에는 SDK의 언어 런타임만 필요합니다:

* Python SDK의 경우 Python 3.10+, TypeScript SDK의 경우 Node.js 18+
* 두 SDK 패키지 모두 호스트 플랫폼을 위한 네이티브 Claude Code 바이너리를 번들로 제공하므로 생성된 CLI를 위해 별도의 Claude Code 또는 Node.js 설치가 필요하지 않습니다

번들 바이너리는 SDK 패키지 버전에 고정되어 있으므로 SDK를 업데이트하는 것이 CLI를 업데이트하는 방법입니다. SDK는 semver를 따르므로 패치 릴리스를 지속적으로 적용하고 마이너 버전을 적용하기 전 [TypeScript](https://github.com/anthropics/claude-agent-sdk-typescript/blob/main/CHANGELOG.md) 또는 [Python](https://github.com/anthropics/claude-agent-sdk-python/blob/main/CHANGELOG.md) 변경 이력을 검토하세요.

### 리소스

에이전트당 1 GiB RAM, 5 GiB 디스크, 1 CPU는 새로 시작된 인스턴스의 적절한 시작점입니다. 메모리 사용량은 세션 길이 및 도구 활동에 따라 증가하므로 유휴 기준이 아닌 실제로 필요한 세션 길이 및 동시성을 위해 크기를 지정하세요. 호스트당 에이전트 산정 방식은 [스케일링 및 동시성](#스케일링-및-동시성)을 참조하세요.

### 네트워크

SDK는 `api.anthropic.com`으로, 또는 Amazon Bedrock이나 Google Cloud's Agent Platform에서 실행할 때는 제공자의 리전 엔드포인트로 아웃바운드 HTTPS가 필요합니다. 에이전트가 [MCP 서버](/docs/en/agent-sdk/mcp)나 외부 도구를 사용하는 경우 해당 엔드포인트로의 아웃바운드 접근도 필요합니다. 프로덕션 환경의 경우 도메인 허용 목록을 강제하고 자격 증명을 주석 처리하며 요청을 기록하는 송출(egress) 프록시를 통해 아웃바운드 트래픽을 라우팅하세요. 전체 패턴은 [안전한 배포](/docs/en/agent-sdk/secure-deployment)를 참조하세요.

인바운드 트래픽의 경우 컨테이너에서 HTTP 또는 WebSocket 포트를 노출하세요. 애플리케이션은 해당 포트에서 클라이언트 요청을 처리하고 내부적으로 SDK를 호출합니다. 서브프로세스 자체는 네트워크에서 대기(listen)하지 않습니다.

## 프로덕션 고려 사항 다루기

셀프 호스팅 에이전트를 배포하기 전에 이러한 결정 사항을 검토하세요.

### 세션 및 상태 영속성

기본 로컬 디스크는 재시작, 스케일 다운 또는 다른 노드로 이동할 때 손실됩니다. 사용자가 다시 시작할 것으로 예상되는 모든 세션에 대해 [`SessionStore` 어댑터](/docs/en/agent-sdk/session-storage)를 사용하여 트랜스크립트를 영구 스토리지에 미러링하세요. S3, Redis, Postgres 어댑터 및 자체 어댑터를 위한 적합성 스위트는 [참조 구현](/docs/en/agent-sdk/session-storage#reference-implementations)을 참조하세요.

`SessionStore` 동작 방식에 대해 알아야 할 3가지 사항:

* **트랜스크립트 전용**: `SessionStore`는 `CLAUDE.md` 메모리 파일이나 기타 작업 디렉토리 산출물이 아닌 트랜스크립트만 미러링합니다. 공유 볼륨을 마운트하거나 이를 별도로 동기화하세요.
* **대체가 아닌 미러링**: 서브프로세스가 로컬 디스크에 먼저 기록하고 스토어는 각 배치(batch)의 복사본을 받습니다. 로컬 쓰기가 여전히 권한을 가집니다.
* **`mirror_error` 메시지**: 스토어가 거부한 배치는 각 재시도 전 짧은 지연 시간(backoff)과 함께 총 3회까지 다시 전송됩니다. 타임아웃된 호출은 재시도되지 않습니다. 배치가 계속 실패하면 SDK는 이를 버리고 `{ type: "system", subtype: "mirror_error" }` 메시지를 방출한 뒤 쿼리를 계속합니다. 스토어 내구성이 중요한 경우 이에 대해 알림을 설정하세요.

### 관찰 가능성

Agent SDK 에이전트는 여러 API 왕복에 걸쳐 도구 호출을 생성하는 장시간 실행 프로세스입니다. 텔레메트리가 없으면 어떤 도구가 실행되었는지, 얼마나 걸렸는지, 세션이 어디서 지연되었는지 볼 수 없습니다.

SDK는 환경으로부터 OpenTelemetry 구성을 상속받습니다. 모든 `query()` 호출이 스팬, 지표, 로그 이벤트를 수집기로 내보내도록 컨테이너 또는 오케스트레이터 수준에서 OTEL 환경 변수를 설정하세요. 아래 예제는 3가지 신호 모두에 대해 OTLP 내보내기를 활성화합니다. `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA`는 트레이스(trace)에만 필요하므로 지표와 로그만 내보내는 경우 생략하세요.

```bash title=".env" theme={null}
CLAUDE_CODE_ENABLE_TELEMETRY=1
CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_EXPORTER_OTLP_ENDPOINT=http://collector.example.com:4318
```

프롬프트 텍스트 및 도구 입력은 기본적으로 내보내기에 포함되지 않습니다. 옵트인 플래그는 [내보내기에서 민감한 데이터 제어](/docs/en/agent-sdk/observability#control-sensitive-data-in-exports)를, 전체 신호 카탈로그는 [관찰 가능성](/docs/en/agent-sdk/observability)을 참조하세요.

### 인증 및 비밀 정보

호스팅 시 다뤄야 할 3가지 인증 고려 사항:

* **Anthropic API**: 서브프로세스는 환경에서 `ANTHROPIC_API_KEY`를 읽습니다. 비밀 관리자(secret manager)에서 이를 공급하거나, 컨테이너 외부에서 키를 주입하는 프록시를 통해 모델 호출이 라우팅되도록 `ANTHROPIC_BASE_URL`을 설정하세요. 프록시 패턴은 [자격 증명 관리](/docs/en/agent-sdk/secure-deployment#credential-management)를, 지원되는 인증 방식은 [SDK 개요](/docs/en/agent-sdk/overview#get-started)를 참조하세요.
* **인바운드**: 에이전트 컨테이너 앞의 게이트웨이에 인증을 배치하세요. 에이전트는 사전 인증된 요청을 받아야 하며 사용자 토큰을 검증하는 주체가 되어서는 안 됩니다.
* **아웃바운드 도구**: 도구 자격 증명을 에이전트 환경 외부에 두세요. 요청이 컨테이너를 떠난 후 프록시가 API 키를 주입하도록 아웃바운드 호출을 라우팅하세요. 에이전트는 호출을 수행하고 프록시가 자격 증명을 추가합니다.

### 스케일링 및 동시성

각 세션은 자체 서브프로세스에서 실행되므로 호스트에서의 동시성은 해당 RAM이 수용할 수 있는 서브프로세스 수에 따라 제한됩니다.

다음 공식을 사용하여 각 호스트의 크기를 산정하세요:

```text theme={null}
agents per host = (host RAM - overhead) / (per-session RAM ceiling)
```

목표 세션 길이 및 예상 도구 로드 하에서 대표 세션을 실행하고 피크 RSS를 기록하여 세션당 상한(ceiling)을 측정하세요. [리소스](#리소스)의 1 GiB 시작점은 상한이 아닌 하한선입니다.

수평 확장 라우팅은 패턴에 따라 다릅니다. 컨테이너가 많은 세션을 보유하는 장시간 실행 세션의 경우 로드 밸런서 뒤에서 컨테이너 풀을 실행하고 `sessionId`의 일관된 해싱(consistent hashing)을 사용하여 각 세션을 하나의 컨테이너에 고정(pin)하세요. 고정된 세션은 퇴출되거나 컨테이너가 재시작될 때까지 동일한 컨테이너, 즉 동일한 실행 서브프로세스에 계속 연결됩니다.

단일 세션에서 대규모 병렬 [서브에이전트](/docs/en/agent-sdk/subagents) 팬아웃이 발생하면 API 속도 제한에 걸릴 수 있습니다. 광범위한 발송 하나를 수행하기보다는 작업을 소규모 배치로 나누어 처리하세요.

### 비용

Anthropic 토큰 비용은 일반적으로 컨테이너 인프라 비용보다 한 자릿수 이상 큽니다. 최소 구성된 컨테이너는 시간당 약 $0.05로 실행되지만 단일 긴 에이전트 세션은 토큰 비용으로 수 달러를 소비할 수 있습니다. 세션당 토큰 정산은 [비용 추적](/docs/en/agent-sdk/cost-tracking)을 참조하세요.

### 멀티 테넌트 격리

기본 SDK 동작은 파일 시스템에서 설정 및 `CLAUDE.md` 메모리 파일을 읽습니다. 여러 테넌트를 제공하는 공유 컨테이너에서는 이러한 파일로 인해 한 테넌트의 컨테이너 컨텍스트가 다른 테넌트의 세션으로 누출될 수 있습니다.

공유 컨테이너 내부에서 테넌트를 격리하려면:

* 파일 시스템 설정이 로드되지 않도록 TypeScript에서는 `settingSources: []`, Python에서는 `setting_sources=[]`를 전달합니다.
* `env`에 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`을 설정합니다. `~/.claude/projects/<project>/memory/`에 있는 [자동 메모리](/docs/en/memory#auto-memory)는 `settingSources`와 상관없이 시스템 프롬프트에 로드됩니다. 무조건 로드되는 기타 입력은 [settingSources가 제어하지 않는 항목](/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control)을 참조하세요.
* 테넌트가 `~/.claude.json` 전역 구성을 공유하지 않도록 `CLAUDE_CONFIG_DIR`을 테넌트별 디렉토리로 지정합니다.
* 테넌트별 작업 디렉토리를 사용합니다. 모든 `query()` 호출 시 `cwd`를 명시적으로 전달하세요.
* 별도의 아웃바운드 IP, 자격 증명 또는 도메인 허용 목록과 같은 테넌트별 송출 규칙을 프록시에 적용하여 손상된 테넌트가 다른 테넌트의 아웃바운드 정책을 통해 데이터를 탈취할 수 없도록 하세요.

아래 예제는 4가지 SDK 수준 옵션을 함께 적용합니다. 다른 테넌트가 읽을 수 없는 경로를 각 테넌트가 받도록 `tenantDir` 및 `configDir`을 만드세요. TypeScript에서 `env`는 서브프로세스 환경을 교체하므로 `PATH` 및 `ANTHROPIC_API_KEY`와 같은 상속된 변수를 유지하려면 `...process.env`를 펼쳐 넣으세요. Python에서는 `env`가 상속된 환경 위에 병합됩니다.

<CodeGroup>
  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  declare const prompt: string;
  declare const tenantDir: string;
  declare const configDir: string;

  for await (const message of query({
    prompt,
    options: {
      cwd: tenantDir,
      settingSources: [],
      env: {
        ...process.env,
        CLAUDE_CONFIG_DIR: configDir,
        CLAUDE_CODE_DISABLE_AUTO_MEMORY: "1",
      },
    },
  })) {
    // ...
  }
  ```

  ```python Python theme={null}
  from claude_agent_sdk import query, ClaudeAgentOptions
  import asyncio

  prompt: str = ...
  tenant_dir: str = ...
  config_dir: str = ...


  async def main():
      async for message in query(
          prompt=prompt,
          options=ClaudeAgentOptions(
              cwd=tenant_dir,
              setting_sources=[],
              env={
                  "CLAUDE_CONFIG_DIR": config_dir,
                  "CLAUDE_CODE_DISABLE_AUTO_MEMORY": "1",
              },
          ),
      ):
          ...


  asyncio.run(main())
  ```
</CodeGroup>

테넌트별 네트워크 제어는 [안전한 배포](/docs/en/agent-sdk/secure-deployment)를 참조하세요.

## 알려진 제한 사항

배포 설계 시 이를 감안하여 계획하세요.

| 제한 사항 | 처리 방법 |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 최상위 세션 타임아웃 없음 | 세션이 스스로 타임아웃되지 않습니다. 에이전트가 중단되기 전 수행하는 도구 사용 왕복 횟수를 제한하려면 `Options`에서 `maxTurns`를 설정하세요. |
| 긴 세션 동안의 메모리 증가 | 세션 길이를 제한하거나 서브프로세스를 주기적으로 재활용하세요. [스케일링 및 동시성](#스케일링-및-동시성)을 참조하세요. |
| 대규모 병렬 서브에이전트 팬아웃 시 속도 제한 도달 | 광범위한 발송 하나를 수행하기보다는 작업을 소규모 배치로 나누어 처리하세요. |
| 서브에이전트별 실제 시각 데드라인 없음 | 각 [서브에이전트](/docs/en/agent-sdk/subagents)의 `AgentDefinition`에서 `maxTurns`로 제한하세요. 백그라운드 서브에이전트 전용으로 `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS`를 설정하면 `run_in_background` 서브에이전트가 출력을 지양할 때 실행되는 스톨 와치독이 작동하지만, 전체 실행 시간 데드라인은 아닙니다. |

## 다음 단계

* [호스팅 쿡북](https://github.com/anthropics/claude-cookbooks/blob/main/claude_agent_sdk/07_Hosting_the_agent.ipynb): Docker, Modal, Kubernetes를 위한 [배포 가능한 코드](https://github.com/anthropics/claude-cookbooks/tree/main/claude_agent_sdk/hosting)가 포함된 노트북 설명.
* [세션 저장소](/docs/en/agent-sdk/session-storage): `SessionStore` 어댑터로 호스트 간 트랜스크립트 보존.
* [관찰 가능성](/docs/en/agent-sdk/observability): 수집기로 OTEL 트레이스, 지표, 로그 내보내기.
* [안전한 배포](/docs/en/agent-sdk/secure-deployment): 네트워크 제어, 자격 증명 관리, 격리 강화.
* [비용 추적](/docs/en/agent-sdk/cost-tracking): 세션별 토큰 및 비용 정산.
