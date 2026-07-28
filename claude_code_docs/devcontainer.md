> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 개발 컨테이너 (Development containers)

> 팀 전체에서 일관되고 격리된 환경을 제공하기 위해 dev container 내부에서 Claude Code를 실행하세요.

[개발 컨테이너](https://containers.dev/)(dev container)를 사용하면 팀의 모든 엔지니어가 실행할 수 있는 동일하고 격리된 환경을 정의할 수 있습니다. 해당 컨테이너에 Claude Code가 설치되면 Claude가 실행하는 명령은 호스트 머신이 아닌 컨테이너 내부에서 실행되는 반면, 작업 시 프로젝트 파일 편집 내용은 로컬 리포지토리에 그대로 나타납니다.

이 페이지에서는 [dev container에 Claude Code 설치하기](#add-claude-code-to-your-dev-container)를 다룬 후, 재빌드 시 인증 유지, 조직 정책 강제 적용, 네트워크 송출 제한, 그리고 권한 프롬프트 없이 실행하기 등의 주제를 다룹니다. 사용자의 환경에 맞는 구성을 읽어보세요.

<Warning>
  dev container가 상당한 보호 기능을 제공하지만, 어떠한 시스템도 모든 공격으로부터 완전히 안전할 수는 없습니다.
  `--dangerously-skip-permissions`로 실행될 때 dev container는 악성 프로젝트가 [`~/.claude`](/docs/en/claude-directory)에 저장된 Claude Code 자격 증명을 포함하여 컨테이너 내부에서 접근 가능한 모든 항목을 유출하는 것을 막지 못합니다.
  신뢰할 수 있는 리포지토리로 개발할 때만 dev container를 사용하고 Claude의 활동을 모니터링하세요.
  `~/.ssh`나 클라우드 자격 증명 파일과 같은 호스트 보안 비밀을 컨테이너에 마운트하지 말고, 리포지토리 범위의 토큰이나 단명(short-lived) 토큰을 사용하세요.
</Warning>

<Accordion title="dev container가 편집기와 작동하는 방식">
  <img src="https://mintcdn.com/claude-code/YvJyjZfd9yMihr0i/images/devcontainer-architecture.svg?fit=max&auto=format&n=YvJyjZfd9yMihr0i&q=85&s=9017b1d16a446c6cc37ba562f35b9aae" className="dark:hidden" alt="호스트의 편집기가 Docker dev container에 연결되는 모습을 보여주는 다이어그램. Claude Code, 터미널 및 빌드 도구는 컨테이너 내부에서 실행됩니다. 호스트 리포지토리는 워크스페이스로서 컨테이너에 바인드 마운트됩니다." width="640" height="300" data-path="images/devcontainer-architecture.svg" />

  <img src="https://mintcdn.com/claude-code/YvJyjZfd9yMihr0i/images/devcontainer-architecture-dark.svg?fit=max&auto=format&n=YvJyjZfd9yMihr0i&q=85&s=ef00c8e25b1ea7a3a152895f1488831b" className="hidden dark:block" alt="호스트의 편집기가 Docker dev container에 연결되는 모습을 보여주는 다이어그램. Claude Code, 터미널 및 빌드 도구는 컨테이너 내부에서 실행됩니다. 호스트 리포지토리는 워크스페이스로서 컨테이너에 바인드 마운트됩니다." width="640" height="300" data-path="images/devcontainer-architecture-dark.svg" />

  dev container는 사용자의 머신이나 GitHub Codespaces와 같은 클라우드 호스트에서 Docker 컨테이너로 실행됩니다. VS Code, GitHub Codespaces, JetBrains IDE 또는 Cursor와 같이 Dev Containers 명세를 지원하는 편집기가 해당 컨테이너에 연결됩니다: 편집기에서 평소처럼 파일을 둘러보고 편집하지만, 통합 터미널, 언어 서버 및 빌드 도구는 모두 호스트가 아닌 컨테이너 내부에서 실행됩니다. 일반 Vim과 같이 dev container 지원이 없는 편집기는 이 워크플로에 포함되지 않습니다.

  Claude Code는 컨테이너 내부에서 실행되므로 프로젝트 툴체인의 나머지 부분과 동일한 파일, 종속성 및 도구를 보게 됩니다. VS Code에서는 [Claude Code 확장 패널](/docs/en/vs-code)을 사용하거나 통합 터미널에서 `claude`를 실행할 수 있습니다. 둘 다 컨테이너 내부에서 실행되며 동일한 `~/.claude` 구성을 공유합니다.
</Accordion>

## dev container에 Claude Code 추가하기

Claude Code는 [Claude Code Dev Container Feature](https://github.com/anthropics/devcontainer-features/tree/main/src/claude-code)를 통해 모든 dev container에 설치됩니다.

이 설정은 VS Code, GitHub Codespaces 또는 JetBrains IDE와 같이 Dev Containers 명세를 지원하는 모든 도구에서 작동합니다. 아래 단계는 VS Code를 예로 사용합니다.

VS Code 또는 Codespaces에서 컨테이너를 열 때 이 기능은 Claude Code VS Code 확장 프로그램도 추가합니다; 다른 편집기는 해당 부분을 무시합니다.

<Tip>
  dev container가 처음이신가요? [VS Code Dev Containers 튜토리얼](https://code.visualstudio.com/docs/devcontainers/tutorial)에서 Docker 설치, 확장 프로그램 설치 및 첫 컨테이너 열기 과정을 안내합니다. 방화벽과 영구 볼륨이 있는 완전하게 강화된 예제는 [참조 컨테이너 사용해 보기](#try-the-reference-container)를 참조하세요.
</Tip>

<Steps>
  <Step title="devcontainer.json 생성 또는 업데이트">
    다음 내용을 리포지토리의 `.devcontainer/devcontainer.json`으로 저장하거나 기존 파일에 `features` 블록을 추가하세요.

    `:1.0`과 같이 끝에 있는 버전 태그는 Claude Code 릴리스가 아니라 해당 기능의 설치 스크립트를 고정합니다. 이 기능은 최신 Claude Code를 설치하며 Claude Code는 기본적으로 컨테이너 내부에서 자동으로 업데이트됩니다.

    CLI 버전을 고정하거나 자동 업데이트를 비활성화하려면 [조직 정책 강제 적용](#enforce-organization-policy)을 참조하세요.

    ```json .devcontainer/devcontainer.json theme={null}
    {
      "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
      "features": {
        "ghcr.io/anthropics/devcontainer-features/claude-code:1.0": {}
      }
    }
    ```

    기존 파일에서 Dockerfile을 사용하는 경우 `image` 줄을 프로젝트의 베이스 이미지로 교체하거나 제거하세요.
  </Step>

  <Step title="컨테이너 다시 빌드">
    Mac에서 `Cmd+Shift+P`, Windows 및 Linux에서 `Ctrl+Shift+P`를 눌러 VS Code Command Palette를 열고 **Dev Containers: Rebuild Container**를 실행하세요.

    다른 도구의 경우 해당 도구의 재빌드 작업을 따르세요: [GitHub Codespaces에서 재빌드](https://docs.github.com/en/codespaces/developing-in-a-codespace/rebuilding-the-container-in-a-codespace), [Dev Containers CLI](https://github.com/devcontainers/cli) 또는 IDE의 dev container 문서를 참조하세요.
  </Step>

  <Step title="Sign in to Claude Code">
    다시 빌드된 컨테이너에서 터미널을 열고 `claude`를 실행한 다음 인증 프롬프트를 따르세요.
  </Step>
</Steps>

인증 프롬프트에 표시되는 내용은 제공업체에 따라 다릅니다:

* **Anthropic**: Claude 또는 Anthropic Console 계정으로 브라우저를 통해 로그인
* **[Amazon Bedrock, Google Cloud's Agent Platform, 또는 Microsoft Foundry](/docs/en/third-party-integrations)**: Claude Code가 브라우저 프롬프트 없이 클라우드 제공업체 자격 증명을 사용함

클라우드 제공업체의 경우 자격 증명 파일을 호스트에서 마운트하기보다는 `containerEnv`, Codespaces 보안 비밀 또는 클라우드의 워크로드 정체성을 통해 환경 변수로 컨테이너에 자격 증명을 전달하세요. Claude Code가 읽는 자격 증명 체인은 [Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), 또는 [Microsoft Foundry](/docs/en/microsoft-foundry)를 참조하세요.

조직에 맞는 경로를 결정하려면 [API 제공업체 선택](/docs/en/admin-setup#choose-your-api-provider)을 참조하세요.

<Note>
  브라우저 로그인은 완료되었지만 콜백이 컨테이너에 도달하지 않는 경우 브라우저에 표시된 코드를 복사하여 터미널의 `Paste code here if prompted` 프롬프트에 붙여넣으세요. 이는 편집기의 포트 포워딩이 localhost 콜백을 라우팅하지 않을 때 발생할 수 있습니다.
</Note>

## 재빌드 시 인증 및 설정 유지

기본적으로 컨테이너의 홈 디렉터리는 재빌드 시 삭제되므로 엔지니어는 매번 다시 로그인해야 합니다. Claude Code는 인증 토큰, 사용자 설정 및 세션 기록을 [`~/.claude`](/docs/en/claude-directory) 아래에 저장합니다. 재빌드 시에도 이 상태를 유지하려면 해당 경로에 명명된 볼륨(named volume)을 마운트하세요.

다음 예제는 `node` 사용자의 홈 디렉터리에 볼륨을 마운트합니다:

```json devcontainer.json theme={null}
"mounts": [
  "source=claude-code-config,target=/home/node/.claude,type=volume"
]
```

`/home/node`를 컨테이너의 `remoteUser` 홈 디렉터리로 교체하세요. `~/.claude` 이외의 위치에 볼륨을 마운트하는 경우 Claude Code가 해당 위치를 읽고 쓸 수 있도록 [`CLAUDE_CONFIG_DIR`](/docs/en/env-vars)을 마운트 경로로 설정하세요.

모든 리포지토리에서 하나의 볼륨을 공유하는 대신 프로젝트별로 상태를 격리하려면 소스 이름에 `${devcontainerId}` 변수를 포함하세요. [참조 구성](https://github.com/anthropics/claude-code/blob/main/.devcontainer/devcontainer.json)은 이 목적으로 `source=claude-code-config-${devcontainerId}`를 사용합니다.

GitHub Codespaces에서 `~/.claude`는 코드스페이스 정지 및 시작 시에는 유지되지만 컨테이너를 재빌드할 때는 여전히 지워지므로 위 볼륨 마운트가 거기도 적용됩니다. 코드스페이스 전반에 걸쳐 인증을 인계하려면 `ANTHROPIC_API_KEY`나 [`claude setup-token`](/docs/en/authentication#generate-a-long-lived-token)으로 생성한 `CLAUDE_CODE_OAUTH_TOKEN`을 [Codespaces secret](https://docs.github.com/en/codespaces/managing-your-codespaces/managing-your-account-specific-secrets-for-github-codespaces)으로 저장하세요. Codespaces는 비밀을 컨테이너 내부의 환경 변수로 자동 제공합니다.

## 조직 정책 강제 적용

dev container는 모든 엔지니어의 머신에서 동일한 이미지와 구성이 실행되므로 조직 정책을 적용하기에 편리한 위치입니다.

Claude Code는 Linux에서 `/etc/claude-code/managed-settings.json`을 읽고 이를 [설정 우선순위](/docs/en/settings#how-scopes-interact)에서 가장 높은 우선순위로 적용하므로, 여기에 있는 값은 엔지니어가 `~/.claude`나 프로젝트의 `.claude/` 디렉터리에 설정한 모든 항목을 재정의합니다. Dockerfile에서 해당 위치로 파일을 복사하세요:

```dockerfile Dockerfile theme={null}
RUN mkdir -p /etc/claude-code
COPY managed-settings.json /etc/claude-code/managed-settings.json
```

Dockerfile이 리포지토리에 상주하므로 쓰기 권한이 있는 사람은 누구나 이 단계를 변경하거나 제거할 수 있습니다. 엔지니어가 리포지토리 파일을 편집하여 우회할 수 없는 정책을 위해 [서버 관리형 설정](/docs/en/server-managed-settings)이나 MDM을 통해 관리형 설정을 전달하세요. 사용 가능한 키 및 기타 전달 경로는 [관리형 설정 파일](/docs/en/settings#settings-files)을 참조하세요.

컨테이너의 모든 Claude Code 세션에 적용되는 [환경 변수](/docs/en/env-vars)를 설정하려면 `devcontainer.json`의 `containerEnv`에 추가하세요. 다음 예제는 텔레메트리 및 오류 보고를 거부하고 설치 후 Claude Code의 자동 업데이트를 방지합니다:

```json devcontainer.json theme={null}
"containerEnv": {
  "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
  "DISABLE_AUTOUPDATER": "1"
}
```

Dev Container Feature는 항상 최신 Claude Code 릴리스를 설치합니다. 재현 가능한 빌드를 위해 특정 Claude Code 버전을 고정하려면 기능을 사용하는 대신 Dockerfile에서 `npm install -g @anthropic-ai/claude-code@X.Y.Z`로 설치하고 위에 표시된 대로 `DISABLE_AUTOUPDATER`를 설정하세요.

권한 규칙, 도구 제한 및 MCP 서버 허용 목록을 포함한 전체 정책 제어 목록은 [조직용 Claude Code 설정](/docs/en/admin-setup)을 참조하세요.

[MCP 서버](/docs/en/mcp)를 컨테이너 내부에서 사용할 수 있도록 하려면 리포지토리 루트의 `.mcp.json` 파일에 [프로젝트 범위](/docs/en/mcp#mcp-installation-scopes)로 정의하여 dev container 구성과 함께 체크인되도록 하세요. 로컬 stdio 서버가 의존하는 실행 파일을 Dockerfile에 설치하고 네트워크 허용 목록에 원격 서버 도메인을 추가하세요.

## 네트워크 송출(egress) 제한

컨테이너의 아웃바운드 트래픽을 Claude Code에 필요한 도메인으로만 제한할 수 있습니다. 추론 및 인증 도메인은 [네트워크 접근 요건](/docs/en/network-config#network-access-requirements)을, 선택적 텔레메트리 및 오류 보고 연결과 이를 비활성화하는 방법은 [텔레메트리 서비스](/docs/en/data-usage#telemetry-services)를 참조하세요.

참조 컨테이너에는 Claude Code 및 개발 도구에 필요한 도메인을 제외한 모든 아웃바운드 트래픽을 차단하는 [`init-firewall.sh`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/init-firewall.sh) 스크립트가 포함되어 있습니다. 컨테이너 내부에서 방화벽을 실행하려면 추가 권한이 필요하므로 참조 구성은 `runArgs`를 통해 `NET_ADMIN` 및 `NET_RAW` 기능을 추가합니다. 방화벽 스크립트와 이러한 기능은 Claude Code 자체에 필수적인 것은 아닙니다: 이를 제외하고 자체 네트워크 제어에 의존할 수 있습니다.

## 권한 프롬프트 없이 실행하기

컨테이너는 Claude Code를 비root 사용자(non-root user)로 실행하고 명령 실행을 컨테이너 내부로 한정하므로, 무인 실행을 위해 `--dangerously-skip-permissions`를 전달할 수 있습니다. CLI는 root로 실행될 때 이 플래그를 거부하므로 `remoteUser`가 비root 계정으로 설정되어 있는지 확인하세요.

권한 프롬프트를 건너뛰면 도구 호출이 실행되기 전에 검토할 수 있는 기회가 없어집니다. Claude는 호스트에 직접 나타나는 바인드 마운트된 워크스페이스의 모든 파일을 수정할 수 있으며 컨테이너의 네트워크 정책이 허용하는 모든 것에 접근할 수 있습니다. 이 플래그를 위 [네트워크 송출 제한](#restrict-network-egress)과 연결하여 바이패스된 세션이 도달할 수 있는 범위를 제한하세요.

안전 검사를 비활성화하지 않고 더 적은 프롬프트를 원한다면 작업 실행 전 분류기가 검토하는 [Auto 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)를 고려해 보세요. 엔지니어가 `--dangerously-skip-permissions`를 전혀 사용하지 못하도록 하려면 [관리형 설정](/docs/en/settings#permission-settings)에서 `permissions.disableBypassPermissionsMode`를 `"disable"`로 설정하세요.

## 참조 컨테이너 사용해 보기

[`anthropics/claude-code`](https://github.com/anthropics/claude-code/tree/main/.devcontainer) 리포지토리에는 CLI, 송출 방화벽, 영구 볼륨 및 Zsh 기반 셸을 결합한 예제 dev container가 포함되어 있습니다. 이는 유지 관리되는 베이스 이미지라기보다는 작동 가능한 예제로 제공됩니다; 자체 구성에 적용하기 전에 각 요소가 어떻게 맞춰지는지 확인하는 용도로 사용하세요.

<Steps>
  <Step title="사전 요건 설치">
    VS Code 및 [Dev Containers 확장 프로그램](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)을 설치하세요.
  </Step>

  <Step title="참조 리포지토리 복제">
    [Claude Code 리포지토리](https://github.com/anthropics/claude-code)를 복제하고 VS Code에서 여세요.
  </Step>

  <Step title="컨테이너에서 다시 열기">
    메시지가 표시되면 **Reopen in Container**를 클릭하거나 Command Palette에서 **Dev Containers: Reopen in Container**를 실행하세요.
  </Step>

  <Step title="Claude Code 시작">
    컨테이너 빌드가 완료되면 `` Ctrl+` ``로 터미널을 열고 `claude`를 실행하여 로그인한 후 첫 세션을 시작하세요.
  </Step>
</Steps>

이 구성을 자신의 프로젝트에 사용하려면 `.devcontainer/` 디렉터리를 리포지토리에 복사하고 툴체인에 맞게 Dockerfile을 조정하거나, 이미 갖고 있는 설정에 이 기능만 추가하려면 [dev container에 Claude Code 추가하기](#add-claude-code-to-your-dev-container)로 돌아가세요.

참조 구성은 세 개의 파일로 구성됩니다. 기능을 통해 자신만의 dev container에 Claude Code를 추가할 때 이러한 파일이 필수적인 것은 아니지만, 구성 요소들을 결합하는 한 가지 방법을 보여줍니다.

| 파일 | 목적 |
| ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [`devcontainer.json`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/devcontainer.json) | 볼륨 마운트, `runArgs` 기능, VS Code 확장 프로그램 및 `containerEnv` |
| [`Dockerfile`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile) | 베이스 이미지, 개발 도구 및 Claude Code 설치 |
| [`init-firewall.sh`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/init-firewall.sh) | 허용된 도메인을 제외한 모든 아웃바운드 네트워크 트래픽 차단 |

## 다음 단계

Claude Code가 dev container에서 실행되면 아래 페이지에서 조직 롤아웃의 나머지 부분(인증 경로 선택, 리포지토리 외부에서 관리형 정책 전달, 사용량 모니터링, Claude Code가 저장하고 전송하는 내용 이해)을 다룹니다.

* [조직용 Claude Code 설정](/docs/en/admin-setup): 인증 제공업체 선택, 기기에 정책이 도달하는 방식 결정 및 롤아웃 계획 수립
* [서버 관리형 설정](/docs/en/server-managed-settings): 엔지니어가 리포지토리 파일을 편집하여 우회할 수 없도록 Claude.ai 관리 콘솔에서 관리형 정책 전달
* [사용량 모니터링 및 활동 감사](/docs/en/monitoring-usage): OpenTelemetry 메트릭 내보내기 및 팀 실행 내역 검토
* [네트워크 접근 요건](/docs/en/network-config#network-access-requirements): 프록시 및 방화벽을 위한 전체 도메인 허용 목록
* [텔레메트리 서비스 및 거부](/docs/en/data-usage#telemetry-services): Claude Code가 기본적으로 보낼 데이터 및 이를 비활성화하는 환경 변수
* [`.claude` 디렉터리 둘러보기](/docs/en/claude-directory): 자격 증명, 설정 및 세션 기록을 포함하여 볼륨 마운트가 보유하는 내용
* [샌드박스 환경](/docs/en/sandbox-environments): dev container와 내장된 Bash 샌드박스, 커스텀 컨테이너 및 VM 비교
* [보안 모델](/docs/en/security): Claude Code의 권한 시스템, 샌드박싱 및 프롬프트 주입 보호가 결합되는 방식
* [권한 모드](/docs/en/permission-modes): Plan 모드부터 Auto 모드, Bypass 모드까지 전체 범위 및 사용 시기
