> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 보안 (Security)

> Claude Code의 보안 보호 조치와 안전한 사용을 위한 모범 사례에 대해 알아보세요.

## 보안 접근 방식

### 보안 기반

사용자 코드의 보안은 최우선 사항입니다. Claude Code는 보안을 핵심에 두고 구축되었으며 Anthropic의 포괄적인 보안 프로그램에 따라 개발되었습니다. 자세한 내용과 관련 리소스(SOC 2 Type 2 보고서, ISO 27001 인증서 등)는 [Anthropic Trust Center](https://trust.anthropic.com)에서 확인할 수 있습니다.

### 권한 기반 아키텍처

Claude Code는 기본적으로 엄격한 읽기 전용 권한을 사용합니다. 추가 작업(파일 편집, 테스트 실행, 명령 실행)이 필요한 경우 Claude Code는 명시적인 권한을 요청합니다. 사용자는 작업을 매번 승인할지 아니면 자동으로 허용할지 제어합니다.

Claude Code는 시스템을 수정할 수 있는 Bash 명령을 실행하기 전에 승인을 요구합니다. `ls`, `cat`, `git status`와 같은 내장된 [읽기 전용 명령](/docs/en/permissions#read-only-commands) 세트는 프롬프트 없이 실행됩니다. 이 접근 방식을 통해 사용자 및 조직이 직접 권한을 구성할 수 있습니다.

자세한 권한 구성은 [Permissions](/docs/en/permissions)를 참조하세요.

### 내장 보호 기능

에이전틱 시스템의 위험을 완화하기 위한 기능:

* **샌드박스 Bash 도구**: 파일 시스템 및 네트워크 격리를 통해 Bash 명령을 [샌드박스화](/docs/en/sandboxing)하여 보안을 유지하면서 권한 프롬프트를 줄입니다. `/sandbox`로 활성화하여 Claude Code가 자율적으로 작업할 수 있는 경계를 정의하세요.
* **작업 디렉터리 경계**: Claude Code는 실행된 폴더 및 그 하위 폴더에만 쓸 수 있으며, 명시적인 권한 없이는 부모 디렉터리의 파일을 수정할 수 없습니다. 승인 프롬프트를 거친 후 Read, Grep 및 Glob 도구를 사용하여 이 경계 외부의 경로를 읽을 수 있습니다. 프롬프트를 건너뛰려면 [추가 디렉터리](/docs/en/permissions#working-directories)로 경계를 확장하거나, 샌드박스가 활성화된 경우에만 적용되는 [샌드박스 `denyRead` 규칙](/docs/en/sandboxing#filesystem-isolation)으로 읽기 전용 Bash 명령에 제공되는 광범위한 읽기 접근을 제한하세요.
* **프롬프트 피로 완화**: 사용자별, 코드베이스별 또는 조직별로 자주 사용되는 안전한 명령을 허용 목록에 추가할 수 있도록 지원합니다.
* **Accept Edits 모드**: 작업 디렉터리의 경로에 대해 파일 편집 및 `mkdir`, `touch`, `rm`, `mv`, `cp`, `sed`와 같은 고정된 파일 시스템 Bash 명령 세트를 자동 승인합니다. 기타 Bash 명령 및 범위를 벗어난 경로는 여전히 프롬프트를 표시합니다.

### 사용자 책임

Claude Code는 사용자가 부여한 권한만 가집니다. 승인 전 제안된 코드와 명령의 안전성을 검토할 책임은 사용자에게 있습니다.

## 프롬프트 주입 방어

프롬프트 주입(Prompt injection)은 악의적인 텍스트를 삽입하여 AI 어시스턴트의 지침을 무력화하거나 조작하려는 공격 기법입니다. Claude Code에는 이러한 공격에 대비한 여러 보호 조치가 포함되어 있습니다:

### 핵심 보호 기능

* **권한 시스템**: 민감한 작업에는 명시적인 승인이 필요합니다.
* **컨텍스트 인식 분석**: 전체 요청을 분석하여 잠재적으로 유해한 지침을 감지합니다.
* **입력 정제(Input sanitization)**: 사용자 입력을 처리하여 명령 주입을 방지합니다.
* **네트워크 명령 승인**: `curl` 및 `wget`과 같이 웹에서 콘텐츠를 가져오는 명령은 기본적으로 자동 승인되지 않습니다. 이러한 명령은 다른 비-읽기 전용 Bash 명령과 마찬가지로 프롬프트를 표시하므로, 한 번 승인하거나 `Bash(curl *)`와 같이 명시적인 허용 규칙을 추가할 수 있습니다. 완전히 차단하려면 [`permissions.deny`](/docs/en/permissions#tool-specific-permission-rules)에 추가하세요.

### 개인정보 보호 보호 조치

데이터를 보호하기 위해 다음과 같은 여러 보호 조치를 구현했습니다:

* 민감한 정보에 대한 제한된 보존 기간(자세한 내용은 [Privacy Center](https://privacy.anthropic.com/en/articles/10023548-how-long-do-you-store-my-data) 참조)
* 사용자 세션 데이터에 대한 접근 제한
* 데이터 학습 기본 설정에 대한 사용자 제어 권한. 일반 사용자는 언제든지 [개인정보 설정](https://claude.ai/settings/privacy)을 변경할 수 있습니다.

전체 세부 정보는 [Commercial Terms of Service](https://www.anthropic.com/legal/commercial-terms)(Team, Enterprise, API 사용자용) 또는 [Consumer Terms](https://www.anthropic.com/legal/consumer-terms)(Free, Pro, Max 사용자용) 및 [Privacy Policy](https://www.anthropic.com/legal/privacy)를 검토하세요.

### 추가 보호 조치

* **네트워크 요청 승인**: 네트워크 요청을 수행하는 도구는 기본적으로 사용자 승인이 필요합니다.
* **격리된 컨텍스트 창**: 웹 가져오기(Web fetch)는 잠재적으로 악의적인 프롬프트가 주입되는 것을 방지하기 위해 별도의 컨텍스트 창을 사용합니다.
* **신뢰 검증**: 처음 실행하는 코드베이스 및 새로운 MCP 서버는 신뢰 검증이 필요합니다.
  * 참고: `-p` 플래그로 비대화형 실행을 할 때는 신뢰 검증이 비활성화됩니다.
  * 참고: 홈 디렉터리에서 Claude Code를 직접 시작하는 경우 신뢰 수락은 현재 세션에만 유지되고 디스크에 기록되지 않으므로 시작할 때마다 프롬프트가 다시 표시됩니다. 이를 유지하는 설정은 없습니다. 대신 디렉터리별로 신뢰 수락이 저장되는 프로젝트 하위 디렉터리에서 Claude Code를 시작하세요.
* **명령 주입 감지**: 의심스러운 Bash 명령은 이전에 허용 목록에 추가되었더라도 수동 승인이 필요합니다.
* **Fail-closed 일치**: 일치하지 않는 명령은 수동 승인을 필요로 하는 것으로 기본 처리됩니다.
* **자연어 설명**: 복잡한 Bash 명령에는 사용자의 이해를 돕기 위한 설명이 포함됩니다.
* **안전한 자격 증명 저장**: API 키 및 토큰은 사용 가능한 경우 macOS Keychain에 저장되며 Windows 및 Linux에서는 파일 권한으로 보호됩니다. [Credential Management](/docs/en/authentication#credential-management)를 참조하세요.

<Warning>
  **Windows WebDAV 보안 위험**: Windows에서 Claude Code를 실행할 때 WebDAV를 활성화하거나 WebDAV 하위 디렉터리가 포함될 수 있는 `\\*`와 같은 경로에 Claude Code가 접근하도록 허용하는 것을 권장하지 않습니다. [WebDAV는 보안 위험으로 인해 Microsoft에 의해 지원 중단(deprecated)되었습니다](https://learn.microsoft.com/en-us/windows/whats-new/deprecated-features#:~:text=The%20Webclient%20\(WebDAV\)%20service%20is%20deprecated). WebDAV를 활성화하면 Claude Code가 권한 시스템을 우회하여 원격 호스트로 네트워크 요청을 보내도록 허용될 수 있습니다.
</Warning>

**신뢰할 수 없는 콘텐츠 작업 시 모범 사례**:

1. 승인 전 제안된 명령 검토
2. 신뢰할 수 없는 콘텐츠를 Claude에게 직접 파이프 연결하는 것 피하기
3. 중요한 파일에 대한 제안된 변경 사항 검토 및 확인
4. 스크립트 실행 및 도구 호출 시(특히 외부 웹 서비스와 상호작용할 때) 가상 머신(VM) 사용
5. 의심스러운 동작은 `/feedback`으로 보고

<Warning>
  이러한 보호 조치가 위험을 크게 줄이지만 어떤 시스템도 모든 공격으로부터 완전히 면역일 수는 없습니다. AI 도구로 작업할 때는 항상 올바른 보안 관행을 유지하세요.
</Warning>

## MCP 보안

Claude Code를 사용하면 Model Context Protocol (MCP) 서버를 구성할 수 있습니다. 허용된 MCP 서버 목록은 엔지니어가 소스 제어에 체크인하는 Claude Code 설정의 일부로 소스 코드에 구성됩니다.

직접 MCP 서버를 작성하거나 신뢰할 수 있는 제공업체의 MCP 서버를 사용할 것을 권장합니다. MCP 서버에 대한 Claude Code 권한을 구성할 수 있습니다. Anthropic은 [Anthropic Directory](https://claude.ai/directory)에 커넥터를 추가하기 전에 [등재 기준](https://claude.com/docs/connectors/building/review-criteria)에 맞춰 검토하지만, MCP 서버를 보안 감사하거나 직접 관리하지는 않습니다.

## IDE 보안

IDE에서 Claude Code 실행에 대한 자세한 내용은 [VS Code 보안 및 개인정보 보호](/docs/en/vs-code#security-and-privacy)를 참조하세요.

## 클라우드 실행 보안

[Claude Code on the web](/docs/en/claude-code-on-the-web)을 사용할 때는 추가 보안 제어가 적용됩니다:

* **격리된 가상 머신**: 각 클라우드 세션은 격리된 Anthropic 관리 VM에서 실행됩니다.
* **네트워크 접근 제어**: 네트워크 접근은 기본적으로 제한되며 비활성화하거나 특정 도메인만 허용하도록 구성할 수 있습니다.
* **자격 증명 보호**: 인증은 샌드박스 내부에서 범위 지정 자격 증명을 사용하는 보안 프록시를 통해 처리되며, 이는 실제 GitHub 인증 토큰으로 변환됩니다.
* **브랜치 제한**: Git push 작업은 현재 작업 브랜치로 제한됩니다.
* **감사 로깅**: 클라우드 환경의 모든 작업은 준수 및 감사 목적으로 로깅됩니다.
* **자동 정리**: 세션 완료 후 클라우드 환경이 자동으로 종료됩니다.

클라우드 실행에 대한 자세한 내용은 [Claude Code on the web](/docs/en/claude-code-on-the-web)을 참조하세요.

[Remote Control](/docs/en/remote-control) 세션은 다르게 작동합니다. 웹 인터페이스가 로컬 머신에서 실행 중인 Claude Code 프로세스에 연결됩니다. 모든 코드 실행 및 파일 접근은 로컬에 유지되며, 세션 트래픽은 TLS를 통해 Anthropic API를 거쳐 이동합니다. 연결되어 있는 동안 [Connection and security](/docs/en/remote-control#connection-and-security)에 설명된 대로 기기 간 대화를 동기화하기 위해 세션 트랜스크립트가 Anthropic 서버에 저장됩니다. 클라우드 VM이나 샌드박스는 관여하지 않습니다. 연결은 수명이 짧고 범위가 좁게 지정된 여러 자격 증명을 사용하며, 각 자격 증명은 특정 목적에만 국한되고 독립적으로 만료되어 단일 자격 증명 유출 시의 피해 범위를 제한합니다.

## 보안 모범 사례

### 민감한 코드 작업 시

* 승인 전 제안된 모든 변경 사항 검토
* 민감한 리포지토리에 프로젝트별 권한 설정 사용
* 추가 격리를 위해 [개발 컨테이너](/docs/en/devcontainer) 사용 고려
* `/permissions`로 권한 설정을 정기적으로 감사

### 팀 보안

* [관리 설정](/docs/en/settings#settings-files)을 사용하여 조직 표준 강제 적용
* 버전 제어를 통해 승인된 권한 구성 공유
* 보안 모범 사례에 대해 팀원 교육
* [OpenTelemetry 메트릭](/docs/en/monitoring-usage)을 통해 Claude Code 사용량 모니터링
* [`ConfigChange` 훅](/docs/en/hooks#configchange)을 사용하여 세션 중 설정 변경을 감사하거나 차단

### 보안 문제 보고

Claude Code에서 보안 취약점을 발견한 경우:

1. 공개적으로 공개하지 마세요.
2. [HackerOne 프로그램](https://hackerone.com/4f1f16ba-10d3-4d09-9ecc-c721aad90f24/embedded_submissions/new)을 통해 보고하세요.
3. 상세한 재현 단계를 포함하세요.
4. 공개 발표 전 당사가 문제를 처리할 수 있도록 시간을 주세요.

## 참고 항목

* [Security guidance plugin](/docs/en/security-guidance): Claude가 세션 중 자신의 코드 변경 사항에서 취약점을 검토하고 수정하도록 함
* [Sandbox environments](/docs/en/sandbox-environments): 격리 접근 방식을 비교하고 위협 모델에 맞는 항목 선택
* [Sandboxing](/docs/en/sandboxing): Bash 명령에 대한 파일 시스템 및 네트워크 격리
* [Permissions](/docs/en/permissions): 권한 및 접근 제어 구성
* [Monitoring usage](/docs/en/monitoring-usage): Claude Code 활동 추적 및 감사
* [Development containers](/docs/en/devcontainer): 안전하고 격리된 환경
* [Anthropic Trust Center](https://trust.anthropic.com): 보안 인증 및 준수 사항
