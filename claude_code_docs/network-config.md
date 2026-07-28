> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 엔터프라이즈 네트워크 구성

> 프록시 서버, 커스텀 인증 기관(CA) 및 상호 전송 계층 보안(mTLS) 인증을 통해 엔터프라이즈 환경용 Claude Code를 구성합니다.

Claude Code는 환경 변수를 통해 다양한 엔터프라이즈 네트워크 및 보안 구성을 지원합니다. 여기에는 기업 프록시 서버를 통한 트래픽 라우팅, 커스텀 인증 기관(CA) 신뢰, 향상된 보안을 위한 상호 전송 계층 보안(mTLS) 인증서를 통한 인증이 포함됩니다.

<Note>
  이 페이지에 표시된 모든 환경 변수는 [`settings.json`](/docs/en/settings)에서도 구성할 수 있습니다.
</Note>

## 프록시 구성

### 환경 변수

Claude Code는 표준 프록시 환경 변수를 준수합니다:

```bash theme={null}
# HTTPS 프록시 (권장)
export HTTPS_PROXY=https://proxy.example.com:8080

# HTTP 프록시 (HTTPS를 사용할 수 없는 경우)
export HTTP_PROXY=http://proxy.example.com:8080

# 특정 요청에 대해 프록시 우회 - 공백으로 구분된 형식
export NO_PROXY="localhost 192.168.1.1 example.com .example.com"
# 특정 요청에 대해 프록시 우회 - 쉼표로 구분된 형식
export NO_PROXY="localhost,192.168.1.1,example.com,.example.com"
# 모든 요청에 대해 프록시 우회
export NO_PROXY="*"
```

<Note>
  Claude Code는 SOCKS 프록시를 지원하지 않습니다.
</Note>

### 기본 인증

프록시에 기본 인증이 필요한 경우 프록시 URL에 자격 증명을 포함하세요:

```bash theme={null}
export HTTPS_PROXY=http://username:password@proxy.example.com:8080
```

<Warning>
  스크립트에 비밀번호를 하드코딩하지 마세요. 대신 환경 변수나 보안 자격 증명 저장소를 사용하세요.
</Warning>

<Tip>
  고급 인증(NTLM, Kerberos 등)이 필요한 프록시의 경우 사용 중인 인증 방법을 지원하는 LLM Gateway 서비스 사용을 고려하세요.
</Tip>

## CA 인증서 저장소

기본적으로 Claude Code는 번들로 제공되는 Mozilla CA 인증서와 운영체제의 인증서 저장소를 모두 신뢰합니다. OS 저장소를 읽으려면 `tls.getCACertificates`가 포함된 런타임이 필요합니다. 네이티브 설치 프로그램에는 항상 포함되어 있으며, npm 설치에는 Node 22.15 이상이 필요합니다. 이전 Node 버전에서는 번들 세트와 `NODE_EXTRA_CA_CERTS`만 적용됩니다. CrowdStrike Falcon 및 Zscaler와 같은 엔터프라이즈 TLS 검사 프록시는 루트 인증서가 OS 신뢰 저장소에 설치되어 있고 런타임이 이를 읽을 수 있는 경우 추가 구성 없이 작동합니다.

`CLAUDE_CODE_CERT_STORE`는 쉼표로 구분된 소스 목록을 허용합니다. 인식되는 값은 Claude Code와 함께 제공되는 Mozilla CA 세트용 `bundled` 및 운영체제 신뢰 저장소용 `system`입니다. 기본값은 `bundled,system`입니다.

번들로 제공되는 Mozilla CA 세트만 신뢰하려면:

```bash theme={null}
export CLAUDE_CODE_CERT_STORE=bundled
```

OS 인증서 저장소만 신뢰하려면:

```bash theme={null}
export CLAUDE_CODE_CERT_STORE=system
```

<Note>
  `CLAUDE_CODE_CERT_STORE`에는 전용 `settings.json` 스키마 키가 없습니다. `~/.claude/settings.json`의 `env` 블록을 통해 설정하거나 프로세스 환경에서 직접 설정하세요.
</Note>

## 커스텀 CA 인증서

엔터프라이즈 환경에서 커스텀 CA를 사용하는 경우 Claude Code가 이를 직접 신뢰하도록 구성하세요:

```bash theme={null}
export NODE_EXTRA_CA_CERTS=/path/to/ca-cert.pem
```

## mTLS 인증

클라이언트 인증서 인증이 필요한 엔터프라이즈 환경의 경우:

```bash theme={null}
# 인증용 클라이언트 인증서
export CLAUDE_CODE_CLIENT_CERT=/path/to/client-cert.pem

# 클라이언트 개인키
export CLAUDE_CODE_CLIENT_KEY=/path/to/client-key.pem

# 선택 사항: 암호화된 개인키의 암호화 구문(Passphrase)
export CLAUDE_CODE_CLIENT_KEY_PASSPHRASE="your-passphrase"
```

Claude Code는 시작 시 인증서 및 키 파일을 읽고 세션 중 설정이 변경되는 경우를 포함하여 설정을 적용할 때마다 파일을 다시 읽습니다. 인증서와 키를 교체하려면 동일한 경로에 있는 파일을 교체하세요.

[클라우드 세션](/docs/en/claude-code-on-the-web)에서는 호스팅 환경이 API 연결을 관리하므로 Claude Code는 설정 파일 `env` 블록에서 시작된 다음 변수들을 무시합니다:

* `CLAUDE_CODE_CLIENT_CERT`
* `CLAUDE_CODE_CLIENT_KEY`
* `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE`
* `NODE_EXTRA_CA_CERTS`
* `NODE_TLS_REJECT_UNAUTHORIZED`
* `CLAUDE_CODE_OAUTH_SCOPES`

Claude Code는 세션의 디버그 로그에 무시된 각 키를 기록합니다.

## 백그라운드 에이전트에 네트워크 설정 적용

[백그라운드 에이전트](/docs/en/agent-view)는 에이전트를 보낸 터미널 내부에서 실행되지 않습니다. 사용자별 감독자(supervisor) 프로세스가 요청 시 시작되고 셸보다 오래 유지되며 모든 `claude agents`, `--bg`, `/background` 세션을 호스팅합니다. [백그라운드 세션이 호스팅되는 방식](/docs/en/agent-view#how-background-sessions-are-hosted)을 참조하세요. 이로 인해 이 페이지의 구성이 해당 세션에 도달하는 방식이 달라집니다.

### 셸이 아닌 설정 파일에 네트워크 변수 설정

감독자는 모든 터미널이 공유하는 단일 프로세스입니다. 이를 먼저 시작한 셸의 환경을 상속하며, OS에 설치된 감독자는 셸 환경을 전혀 받지 못합니다. 셸에서만 프록시, CA 경로 또는 mTLS 변수를 export하는 경우, 해당 셸이 감독자를 콜드 스타트했을 때만 백그라운드 에이전트에 전달되고 다른 셸이 스타트했을 때는 전달되지 않습니다.

대신 `~/.claude/settings.json`의 `env` 블록 또는 [관리형 설정](/docs/en/settings)에 동일한 변수를 입력하세요. 이 페이지의 모든 변수는 거기에 설정할 수 있으며, 설정은 모든 머신의 모든 백그라운드 세션에 전달되는 유일한 구성입니다.

### 기업용 런처를 설정으로 구성

일부 조직에서는 샌드박싱, 네트워크 제어 또는 자격 증명 주입을 적용하는 기업용 런처를 통해 모든 Claude Code 프로세스를 시작해야 합니다. 감독자와 그 워커는 `PATH`에서 `claude`를 찾는 대신 고정된 경로에서 Claude Code를 시작하므로 모든 백그라운드 에이전트는 `PATH` 앞쪽에 배치한 래퍼를 우회합니다.

[`processWrapper`](/docs/en/settings#available-settings) 설정을 구성하여 감독자, 그 워커 및 [런처가 다루는 범위](/docs/en/corporate-launcher#what-the-launcher-covers)에 나열된 기타 백그라운드 프로세스의 앞에 런처 접두사를 붙이세요. 동등한 [`CLAUDE_CODE_PROCESS_WRAPPER`](/docs/en/env-vars) 환경 변수는 둘 다 설정되었을 때 우선하며 동일한 규칙의 적용을 받습니다. 셸 export가 아닌 관리형 설정 또는 `~/.claude/settings.json`을 통해 전달해야 합니다. [기업용 런처 뒤에서 Claude Code 실행](/docs/en/corporate-launcher)에서는 런처가 충족해야 하는 계약, 도달하는 영역과 도달하지 않는 영역, 배포 방법을 다룹니다.

<Note>
  이미 실행 중인 감독자는 시작된 실행 구성을 유지합니다. 런처 설정을 배포한 후 [`claude daemon stop --any`](/docs/en/agent-view#the-supervisor-process)를 실행하여 다음 `claude agents` 또는 `--bg`가 이를 준수하는 감독자를 시작하도록 하세요. 설치된 서비스는 `--any` 없이 `claude daemon stop`을 수락합니다.
</Note>

## 네트워크 액세스 요구 사항

Claude Code를 사용하려면 다음 URL에 대한 액세스가 필요합니다. 프록시 구성 및 방화벽 규칙에서(특히 컨테이너화되거나 제한된 네트워크 환경에서) 이 URL들을 허용 목록(Allowlist)에 추가하세요.

| URL | 용도 |
| --- | --- |
| `api.anthropic.com` | WebFetch [도메인 안전 검사](/docs/en/data-usage#webfetch-domain-safety-check), 기능 플래그 가져오기, 텔레메트리 이벤트 로깅을 포함한 Claude API 요청 |
| `claude.ai` | claude.ai 계정 인증 |
| `claude.com` | claude.ai 계정 로그인 시 브라우저에서 `claude.com` 페이지를 열고 `claude.ai`로 리디렉션합니다. 사전 승인된 WebFetch 문서 조회도 CLI에서 이 호스트에 도달합니다 |
| `platform.claude.com` | Anthropic Console 계정 인증. OAuth 토큰 교환, 갱신 및 취소도 claude.ai 계정에 대해 이 호스트로 이동하므로 Console 및 claude.ai 로그인 모두에 필수적입니다 |
| `mcp-proxy.anthropic.com` | 조직 관리자가 구성한 커넥터를 포함한 [claude.ai의 MCP 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai). 커넥터 트래픽은 이 프록시를 통해 라우팅되며 커넥터는 claude.ai 인증 사용자에 대해 기본적으로 활성화되어 있습니다. 비활성화하려면 [`ENABLE_CLAUDEAI_MCP_SERVERS=false`](/docs/en/env-vars) 또는 [`disableClaudeAiConnectors`](/docs/en/settings#available-settings) 설정을 지정하세요 |
| `downloads.claude.ai` | 플러그인 실행 파일 다운로드, 네이티브 설치 프로그램, 네이티브 자동 업데이터 및 업데이트 버전 검사 |
| `storage.googleapis.com` | `/plugin`에 표시되는 설치 수 및 플러그인 메타데이터. 서명된 [아티팩트](/docs/en/artifacts) 업로드는 이 호스트를 먼저 시도하며, 차단되면 게시가 `api.anthropic.com`으로 대체됩니다 |
| `storage.googleapis.com` | {/* max-version: 2.1.115 */}2.1.116 이전 버전의 네이티브 설치 프로그램 및 네이티브 자동 업데이터 |
| `bridge.claudeusercontent.com` | [Claude in Chrome](/docs/en/chrome) 확장 프로그램 WebSocket 브리지 |
| `raw.githubusercontent.com` | [`/release-notes`](/docs/en/commands) 및 업데이트 후 표시되는 릴리스 노트용 변경 이력 피드 |
| `http-intake.logs.us5.datadoghq.com` | 운영 텔레메트리 이벤트. CLI가 Anthropic API를 직접 사용할 때만 전송되며 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry에서는 전송되지 않습니다. 선택 사항: [`DISABLE_TELEMETRY`](/docs/en/data-usage#telemetry-services) 또는 `DO_NOT_TRACK`으로 비활성화 가능 |
| `browser-intake-us5-datadoghq.com` | 운영 오류 보고서. CLI가 Anthropic API를 직접 사용하고 서버 측 롤아웃 게이트가 이를 활성화할 때 전송됩니다. 선택 사항: `DISABLE_ERROR_REPORTING` 또는 `DISABLE_TELEMETRY`로 비활성화 가능. [텔레메트리 서비스](/docs/en/data-usage#telemetry-services) 참조 |
| `formulae.brew.sh` | Homebrew 설치 시 업데이트 버전 검사. 다른 설치 방법은 이 호스트에 연결하지 않습니다 |
| `code.claude.com` | 내장 claude-code-guide 에이전트의 Claude Code 문서 조회 및 사전 승인된 WebFetch 요청. 이 호스트를 차단하면 문서 조회에만 영향을 미칩니다 |

npm을 통해 Claude Code를 설치하거나 자체 바이너리 배포판을 관리하는 경우 최종 사용자에게 `downloads.claude.ai`에 대한 네이티브 설치 프로그램 및 자동 업데이터 용도가 필요하지 않지만, 조직이 미러링하지 않는 한 npm 및 bun 설치 시 해당 패키지 레지스트리인 `registry.npmjs.org`가 필요합니다. 표의 다른 용도는 설치 방법에 상관없이 적용됩니다.

두 개의 Datadog 수신 호스트는 선택적 운영 텔레메트리만 전달하며, [`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`](/docs/en/env-vars)를 설정하면 둘 다 비활성화됩니다. 서드파티 공급자 세션은 플랫폼이 [`CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST`](/docs/en/env-vars)를 설정하고 텔레메트리 메트릭이 기본적으로 켜져 있는 경우에도 이러한 호스트로 전송하지 않습니다. 허용 목록을 확정하기 전에 Claude Code가 전송하는 모든 내용과 이를 비활성화하는 방법은 [텔레메트리 서비스](/docs/en/data-usage#telemetry-services)를 참조하세요.

[Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry) 또는 로그인된 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션을 사용하는 경우 모델 트래픽 및 인증은 `api.anthropic.com`, `claude.ai`, `platform.claude.com` 대신 사용자의 공급자 또는 게이트웨이로 이동합니다. WebFetch 도구는 [설정](/docs/en/settings)에서 `skipWebFetchPreflight: true`를 설정하지 않는 한 여전히 [도메인 안전 검사](/docs/en/data-usage#webfetch-domain-safety-check)를 위해 `api.anthropic.com`을 호출합니다.

[`ANTHROPIC_BASE_URL`](/docs/en/llm-gateway-connect#set-the-base-url-and-credential)을 통해 [LLM 게이트웨이](/docs/en/llm-gateway)로 라우팅할 때 [패스트 모드](/docs/en/fast-mode) 가용성 검사는 게이트웨이 기본 URL이 아닌 `api.anthropic.com`을 계속 호출합니다. 이 검사는 구성된 HTTP 프록시를 준수하므로 네트워크 차단이 원인인 경우 프록시의 `api.anthropic.com`에 대한 허용 목록 항목이 해결책입니다. 네트워크 차단은 프록시를 통해서도 호스트에 도달할 수 없는 경우에만 검사에 실패하며 패스트 모드는 연결 오류를 보고합니다. 검사가 Anthropic이 거부하는 게이트웨이 발급 자격 증명을 제공하는 경우에도 동일한 연결 오류가 나타납니다. 차단된 것이 없기 때문에 허용 목록 추가는 여기서 도움이 되지 않습니다. 이를 복원하는 변수는 [프록시 및 LLM 게이트웨이 뒤에서 패스트 모드 사용](/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways)을 참조하세요.

[웹에서의 Claude Code](/docs/en/claude-code-on-the-web) 및 [Code Review](/docs/en/code-review)는 Anthropic이 관리하는 인프라에서 리포지토리에 연결됩니다. GitHub Enterprise Cloud 조직이 IP 주소별로 액세스를 제한하는 경우 [설치된 GitHub Apps에 대한 IP 허용 목록 상속](https://docs.github.com/en/enterprise-cloud@latest/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-allowed-ip-addresses-for-your-organization#allowing-access-by-github-apps)을 활성화하세요. Claude GitHub App은 IP 범위를 등록하므로 이 설정을 활성화하면 수동 구성 없이 액세스할 수 있습니다. 수동으로 [허용 목록에 범위를 추가](https://docs.github.com/en/enterprise-cloud@latest/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-allowed-ip-addresses-for-your-organization#adding-an-allowed-ip-address)하거나 다른 방화벽을 구성하려면 [Anthropic API IP 주소](https://platform.claude.com/docs/en/api/ip-addresses)를 참조하세요.

방화벽 뒤에서 자체 호스팅되는 [GitHub Enterprise Server](/docs/en/github-enterprise-server) 인스턴스의 경우, Anthropic 인프라가 GHES 호스트에 도달하여 리포지토리를 복제하고 리뷰 코멘트를 게시할 수 있도록 동일한 [Anthropic API IP 주소](https://platform.claude.com/docs/en/api/ip-addresses)를 허용 목록에 추가하세요.

### Desktop 및 claude.ai

이전 표는 독립형 CLI를 다룹니다. 브라우저의 Claude Desktop 앱과 claude.ai는 [아티팩트](/docs/en/artifacts)를 제공하는 `assets-proxy.anthropic.com` 및 `*.claudeusercontent.com` 출처를 포함하여 추가 Anthropic CDN 호스트에서 애플리케이션 코드와 사용자 콘텐츠를 로드합니다. 이러한 호스트를 차단하면서 `claude.ai`를 허용하면 오류 대신 빈 페이지가 생성됩니다. Desktop 페이지의 [네트워크 액세스 요구 사항](/docs/en/desktop#network-access-requirements)을 참조하세요.

## 추가 리소스

* [Claude Code 설정](/docs/en/settings)
* [환경 변수 참조](/docs/en/env-vars)
* [문제 해결 가이드](/docs/en/troubleshooting)
