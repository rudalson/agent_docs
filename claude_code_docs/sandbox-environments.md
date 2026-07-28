> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 샌드박스 환경 선택하기

> Claude Code 샌드박스 옵션 비교: 내장 샌드박스 Bash 도구, 샌드박스 런타임, 개발 컨테이너, Docker 및 VM. 위협 모델에 맞는 적절한 격리 수준을 선택하세요.

Claude Code를 격리하면 세션이 읽고, 쓰고, 네트워크에 도달할 수 있는 범위가 제한됩니다. 이는 Claude가 적은 권한 프롬프트로 작업하게 하거나, 무인 모드로 실행하거나, 완벽히 신뢰할 수 없는 코드를 가리키도록 설정할 때 가장 중요합니다.

Claude Code는 경량화된 명령별 샌드박스부터 완전히 분리된 가상 머신에 이르기까지 다양한 유형의 격리된 환경에서 실행할 수 있습니다. 이 페이지에서는 격리 대상 및 요구 사항별로 이를 비교하고, 위협 모델에 맞는 환경을 선택하도록 도우며, 조직 전체에서 해당 선택을 강제 적용하는 방법을 안내합니다.

<Info>
  광범위한 보안 모델에 대해서는 [Security](/docs/en/security)를 참조하세요. Agent SDK 배포에 대해서는 [Secure deployment](/docs/en/agent-sdk/secure-deployment)를 참조하세요.
</Info>

## 샌드박스 접근 방식 비교

아래 표의 처음 두 접근 방식은 컨테이너 없이 호스트 운영체제에서 실행됩니다. 나머지 방식은 Claude Code를 컨테이너 또는 가상 머신 내부에 배치합니다.

| 접근 방식 | 격리되는 대상 | Docker 필요 여부 | 설정 공수 |
| :-------------------------------- | :-------------------------------------------------------------------------- | :-------------- | :---------------------------------------------- |
| [Sandboxed Bash tool](#sandboxed-bash-tool) | Bash 명령 및 그 자식 프로세스 | 아니오 | macOS에서 최소; Linux 및 WSL2에서 낮음 |
| [Sandbox runtime](#sandbox-runtime) | 파일 도구, MCP 서버, 훅을 포함한 Claude Code 프로세스 전체 | 아니오 | 낮음 |
| [Dev container](#dev-containers) | 전체 개발 환경 | 예 | 중간 |
| [Custom container](#custom-container) | 전체 개발 환경 | 예 | 중간 ~ 높음 |
| [Virtual machine](#virtual-machine) | 전체 운영체제 | 아니오 | 높음 |
| [Claude Code on the web](#claude-code-on-the-web) | Anthropic이 호스팅하는 전체 운영체제 | 아니오 | 없음; Claude 구독 및 GitHub 필요 |

[sandboxed Bash tool](/docs/en/sandboxing)은 Claude Code에 내장되어 있으며 Bash 명령만 제한합니다. 내장 파일 도구, MCP 서버 및 훅은 여전히 호스트에서 직접 실행됩니다. 표의 다른 모든 접근 방식은 전체 Claude Code 프로세스를 격리 경계 내부에 두므로 파일 도구, MCP 서버 및 훅도 함께 제한됩니다.

<Warning>
  샌드박스 격리는 침해의 영향을 줄이지만 위험을 완전히 제거하지는 못합니다. 네트워크 아웃바운드(egress)를 허용하는 모든 접근 방식은 에이전트가 읽을 수 있는 데이터를 유출할 수 있으며, 프로젝트 디렉터리를 쓰기 가능하게 마운트하는 모든 접근 방식은 해당 코드를 수정할 수 있습니다. 샌드박스를 하드 제어로 의존하기 전에 [보안 제한 사항](/docs/en/sandboxing#security-limitations)을 검토하세요.

  또한 격리가 모델로 전송되는 내용을 변경하지는 않습니다. 프롬프트와 Claude가 읽는 파일은 샌드박스 여부와 관계없이 Anthropic API 또는 구성된 제공업체로 전송됩니다. Claude Code가 전송하는 내용과 이를 줄이는 방법에 대해서는 [Data usage](/docs/en/data-usage)를 참조하세요.
</Warning>

## 접근 방식 선택하기

목표를 아래 표의 행과 일치시킨 후 이어지는 세부 섹션을 읽어보세요.

| 목표 | 권장 시작 방식 |
| :---------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 본인 머신에서 일상적인 작업 중 권한 프롬프트 줄이기 | `/sandbox`로 활성화되는 [sandboxed Bash tool](/docs/en/sandboxing) |
| `--dangerously-skip-permissions` 또는 오토 모드로 Claude를 무인 실행하기 | 사전 구성된 [dev container](/docs/en/devcontainer), 모든 컨테이너/VM, 또는 [sandbox runtime](#sandbox-runtime) |
| Docker 없이 Bash뿐만 아니라 MCP 서버 및 훅도 격리하기 | [sandbox runtime](#sandbox-runtime) |
| 신뢰할 수 없는 리포지토리에서 작업하기 | 전용 가상 머신, 또는 Claude 구독 및 연결된 GitHub 계정이 있는 경우 [Claude Code on the web](/docs/en/claude-code-on-the-web) |
| 팀 전체에 샌드박스 환경 표준화하기 | 리포지토리에 복사된 사전 구성된 [dev container](/docs/en/devcontainer) |
| 로컬 설정이 없는 기기에서 Claude Code 사용하기 | Claude 구독 및 연결된 GitHub 계정이 필요한 [Claude Code on the web](/docs/en/claude-code-on-the-web) |
| 조직의 모든 개발자에게 격리 강제 적용하기 | [조직 전체에 격리 강제 적용하기](#enforce-isolation-across-an-organization) |
| 네이티브 Windows 호스트에서 작업하기 | 컨테이너 또는 VM, 또는 WSL2 내부에서 Bash 샌드박스 실행 |

### 격리와 권한 모드의 관계

[권한 모드](/docs/en/permission-modes)는 도구 호출을 실행할지 여부와 사전에 프롬프트를 표시할지 여부를 결정합니다. 격리는 명령이 실행된 후 접근할 수 있는 대상을 제한합니다. 두 가지는 함께 작동합니다. 권한 모드가 물어보지 않고 작업을 실행하도록 허용할 때 격리 경계는 해당 작업이 도달할 수 있는 범위를 제한합니다.

`--dangerously-skip-permissions`를 전달하면 Claude는 사전에 묻지 않고 실행됩니다. 명시적인 [ask rules](/docs/en/permissions#manage-permissions), [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구, `/` 또는 홈 디렉터리를 대상으로 하는 삭제 작업에 대해서만 프롬프트가 표시됩니다. 실수를 감지할 프롬프트가 없으므로 선택한 격리 경계가 시스템을 보호합니다. `--dangerously-skip-permissions` 세션은 항상 컨테이너, VM 또는 [sandbox runtime](#sandbox-runtime) 내부에서 실행하여 파일 도구, MCP 서버 및 훅도 경계 내부에 두도록 하세요.

[오토 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)는 프롬프트를 분류기(classifier)로 대체하여 작업을 검토하고 요청 범위를 벗어나거나, 인식되지 않은 인프라를 대상으로 하거나, Claude가 읽은 적대적 콘텐츠에 의해 유도된 것으로 보이는 작업을 차단합니다. 분류기는 작업별 제어 장치이며 격리 경계가 아니므로, 무인 실행 시 격리 경계가 여전히 심층 방어를 제공하지만 `--dangerously-skip-permissions`에서처럼 필수 사항은 아닙니다.

[sandboxed Bash tool](#sandboxed-bash-tool) 자체는 Bash만 제한하므로 두 모드 모두에서 완전한 무인 실행에는 충분하지 않습니다. 여러 방식을 중첩할 수 있습니다. 컨테이너나 VM 내부에서 샌드박스 Bash 도구를 실행하면 외부 환경 경계 위에 OS 수준의 명령 제한을 추가할 수 있습니다. Bash 샌드박스 자체가 권한 규칙 및 모드와 상호작용하는 방식은 [샌드박스와 권한 및 권한 모드의 관계](/docs/en/sandboxing#how-sandboxing-relates-to-permissions-and-permission-modes)를 참조하세요.

## 샌드박스 Bash 도구 (Sandboxed Bash tool)

<Note>
  이 옵션은 네이티브 Windows를 지원하지 않습니다. Windows 호스트의 경우 WSL2 또는 아래의 컨테이너/VM 접근 방식 중 하나를 사용하세요.
</Note>

샌드박스 Bash 도구는 Claude Code에 내장되어 있습니다. 운영체제 프리미티브를 사용하여 Claude가 실행하는 모든 Bash 명령의 파일 시스템 및 네트워크 접근을 제한합니다. macOS에서는 내장 샌드박스인 Seatbelt를, Linux 및 WSL2에서는 [bubblewrap](https://github.com/containers/bubblewrap)을 사용합니다. 기본적으로 작업 디렉터리에 대한 쓰기를 허용하며 명령이 새로운 네트워크 도메인을 필요로 할 때 처음으로 프롬프트를 표시합니다.

`/sandbox` 명령으로 활성화하세요. [Sandboxing](/docs/en/sandboxing) 가이드에서 승인 모드, 기본 경계 및 이를 넓히거나 좁히는 방법을 다룹니다.

명령별 샌드박스는 세션에서 실행되는 모든 것을 포함하지는 않습니다:

* Read, Edit, WebFetch와 같은 다른 [내장 도구](/docs/en/tools-reference)는 Claude Code 프로세스 내부에서 실행되며 임의의 코드를 생성하지 않습니다. 대신 경로 또는 도메인에 대한 [권한 규칙](/docs/en/permissions)이 이를 제어합니다.
* [MCP](/docs/en/mcp) 서버 및 훅은 호스트에서 제한 없이 실행되는 별도 프로세스입니다.

내장 도구, MCP 서버 및 훅을 모두 하나의 OS 경계 뒤에 두려면 [sandbox runtime](#sandbox-runtime), [dev container](#dev-containers) 또는 [custom container](#custom-container) 내부에서 전체 Claude Code 프로세스를 실행하세요.

## 샌드박스 런타임 (Sandbox runtime)

[`@anthropic-ai/sandbox-runtime`](https://github.com/anthropic-experimental/sandbox-runtime) 패키지는 내장 Bash 샌드박스가 사용하는 동일한 Seatbelt 또는 bubblewrap 격리로 프로세스 전체를 감쌉니다. 이를 통해 Claude Code를 실행하면 Bash뿐만 아니라 세션의 모든 도구, 훅 및 MCP 서버가 제한됩니다. 런타임은 베타 리서치 프리뷰 상태이며 패키지가 발전함에 따라 구성 형식이 변경될 수 있습니다.

런타임은 기본적으로 모든 쓰기 및 네트워크 접근을 거부하므로 Claude Code를 실행하기 전에 설정해야 합니다. `~/.srt-settings.json` 또는 `--settings`로 전달하는 파일에서 최소한 프로젝트 디렉터리, Claude Code의 구성 경로(`~/.claude` 및 `~/.claude.json`), 그리고 Claude Code가 런타임 파일을 작성하는 `/tmp`에 대한 쓰기 접근을 허용하세요. `api.anthropic.com` 또는 구성된 제공업체의 엔드포인트를 포함하여 세션에 필요한 네트워크 도메인을 허용합니다. 전체 구성 스키마는 패키지 [README](https://github.com/anthropic-experimental/sandbox-runtime)를 참조하세요.

설정 파일이 준비되면 `npx`로 Claude Code를 시작하고 감쌀 명령으로 `claude`를 전달합니다:

```bash theme={null}
npx @anthropic-ai/sandbox-runtime claude
```

Claude Code는 구성된 파일 시스템 및 네트워크 경계를 가지고 샌드박스 내부에서 시작됩니다. 동일한 명령으로 독립형 MCP 서버나 기타 헬퍼 프로세스를 샌드박스화할 수 있습니다.

## 개발 컨테이너 (Dev containers)

개발 컨테이너(Dev container)는 VS Code나 호환되는 에디터가 관리하는 Docker 컨테이너 내부에서 프로젝트를 마운트한 채로 Claude Code를 실행합니다. 리포지토리의 `.devcontainer/` 디렉터리를 사용하여 자체 컨테이너를 정의할 수 있습니다.

claude-code 리포지토리는 시작점으로 기본 거부 iptables 방화벽이 포함된 [예시 개발 컨테이너](/docs/en/devcontainer)를 게시합니다. 이를 리포지토리에 복사하고 환경에 맞게 방화벽 허용 목록, 베이스 이미지 및 고정된 Claude Code 버전을 조정하세요. 방화벽이 승인되지 않은 아웃바운드를 차단하므로 이러한 구성은 무인 작업을 위해 `--dangerously-skip-permissions`로 Claude Code를 실행하는 것을 지원합니다.

## 커스텀 컨테이너 (Custom container)

자체 네트워크 정책, 마운트된 볼륨 및 seccomp 프로필을 사용하여 모든 Docker 또는 OCI 컨테이너 이미지에서 Claude Code를 실행할 수 있습니다. 이는 기존 컨테이너 인프라나 CI 러너가 있는 조직에서 가장 흔히 사용하는 경로입니다.

여러 관리형 샌드박스 및 원격 실행 서비스에서 컨테이너를 호스팅할 수 있습니다. 직접 운영하는 컨테이너와 동일한 체크리스트가 적용됩니다. 쓰기 가능하게 마운트된 대상, 내부에서 접근 가능한 자격 증명 및 토큰, 네트워크 아웃바운드 정책이 허용하는 대상을 검토하세요.

명령별 제한을 위해 컨테이너 내부에 내장 Bash 샌드박스를 중첩할 수 있습니다. 비권한(unprivileged) 컨테이너에는 [샌드박스 문제 해결](/docs/en/sandboxing#troubleshooting)에 설명된 중첩 샌드박스 설정이 필요합니다.

## 가상 머신 (Virtual machine)

전용 가상 머신은 자체 커널과 (클라우드 또는 microVM 배포의 경우) 자체 가상화된 하드웨어를 통해 가장 강력한 분리를 제공합니다. 옵션에는 클라우드 인스턴스, 로컬 하이퍼바이저 및 Firecracker와 같은 microVM이 포함됩니다.

신뢰할 수 없는 코드를 평가할 때, 보안 정책상 에젠트와 호스트 간에 커널 수준의 분리가 필요할 때, 또는 호스트 수준의 접근 방식이 준수 요구 사항을 충족하지 못할 때 이 방식을 사용하세요. Docker Desktop의 [샌드박스 기능](https://docs.docker.com/ai/sandboxes/)은 자체 Docker 데몬 및 작업 공간 동기화가 있는 microVM을 제공하며, Docker Desktop이 이미 설치된 호스트에서 Claude Code를 실행할 수 있습니다.

## Claude Code on the web

[Claude Code on the web](/docs/en/claude-code-on-the-web)은 격리된 Anthropic 관리 가상 머신에서 각 세션을 실행합니다. 네트워크 프록시가 기본 허용 목록을 강제 적용하고, 별도의 프록시가 샌드박스 외부에 GitHub 토큰을 보관하면서 내부의 리포지토리 접근을 위한 범위 지정 자격 증명을 발급합니다.

인프라를 직접 프로비저닝하지 않고 완전한 VM 격리를 원하거나 로컬 개발 환경이 없는 기기에서 작업을 위임할 때 이 방식을 사용하세요. Claude 구독 및 연결된 GitHub 계정이 필요하며 세션은 GitHub에서 리포지토리를 복제합니다. 플랜 가용성 및 GitHub 인증 옵션은 [Claude Code on the web](/docs/en/claude-code-on-the-web)을 참조하세요.

## 조직 전체에 격리 강제 적용하기

개별 개발자는 위의 접근 방식을 선택하여 사용할 수 있습니다. 조직에서 강제할 수 있는 대상과 사용할 도구는 접근 방식에 따라 다릅니다:

* **내장 Bash 샌드박스**: Claude Code 자체가 강제하는 유일한 접근 방식입니다. MDM에서 관리하는 파일로 또는 Claude.ai의 [서버 관리 설정](/docs/en/server-managed-settings)을 통해 [관리 설정](/docs/en/settings#settings-files)을 통해 `sandbox` 설정 키를 전달합니다. 배포할 키와 개발자가 정책을 넓히지 못하도록 막는 방법은 [관리 설정으로 샌드박스 강제 적용](/docs/en/sandboxing#enforce-sandboxing-with-managed-settings)을 참조하세요.
* **개발 컨테이너**: 팀 전체의 환경을 표준화하기 위해 리포지토리에 [예시 개발 컨테이너](/docs/en/devcontainer)를 커밋합니다. Claude Code가 컨테이너를 필수 요구하지 않으므로 이는 강제 경계라기보다는 관례(convention)입니다. 개발자가 컨테이너 외부에서 Claude Code를 실행할 수 없어야 하는 경우 조직의 기기 관리 또는 소프트웨어 허용 목록 도구로 이를 강제 적용하세요.
* **커스텀 컨테이너 및 VM**: 승인된 이미지를 통해 Claude Code를 배포하고 조직의 기기 관리 또는 소프트웨어 허용 목록 도구를 사용하여 그 외부에서의 설치를 방지합니다.

## 참고 항목

다음 페이지에서 위 접근 방식에 대한 구성 및 정책 세부 정보를 다룹니다.

* [Sandboxing](/docs/en/sandboxing): 내장 샌드박스 Bash 도구 구성
* [Dev container](/docs/en/devcontainer): 사전 구성된 Docker 개발 컨테이너
* [Security](/docs/en/security): 전체 Claude Code 보안 모델
* [Secure deployment](/docs/en/agent-sdk/secure-deployment): Agent SDK 애플리케이션에 대한 격리 안내
* [Settings](/docs/en/settings#sandbox-settings): 관리 설정 전달을 포함한 모든 샌드박스 구성 키
