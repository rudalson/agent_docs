> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 고급 설치 및 구성

> Claude Code의 시스템 요구 사항, 플랫폼별 설치, 버전 관리 및 제거 방법입니다.

이 페이지에서는 시스템 요구 사항, 플랫폼별 설치 세부 정보, 업데이트 및 제거 방법을 다룹니다. 첫 번째 세션에 대한 안내 가이드는 [빠른 시작](/docs/en/quickstart)을 참조하세요. 터미널을 한 번도 사용해 본 적이 없다면 [터미널 가이드](/docs/en/terminal-guide)를 참조하세요.

## 시스템 요구 사항

Claude Code는 다음 플랫폼 및 구성에서 실행됩니다:

* **운영 체제**:
  * macOS 13.0 이상
  * Windows 10 1809 이상 또는 Windows Server 2019 이상
  * Ubuntu 20.04 이상
  * Debian 10 이상
  * Alpine Linux 3.19 이상
* **하드웨어**: 4 GB 이상 RAM, x64 또는 ARM64 프로세서
* **네트워크**: 인터넷 연결 필요. [네트워크 구성](/docs/en/network-config#network-access-requirements)을 참조하세요.
* **셸**: Bash, Zsh, PowerShell 또는 CMD.
* **위치**: [Anthropic 지원 국가](https://www.anthropic.com/supported-countries)

### 추가 종속성

* **ripgrep**: 일반적으로 Claude Code에 포함되어 있습니다. 검색이 실패하는 경우 [검색 문제 해결](/docs/en/troubleshooting#search-and-discovery-issues)을 참조하세요.

## Claude Code 설치

<Tip>
  그래픽 인터페이스를 선호하시나요? [데스크톱 앱](/docs/en/desktop-quickstart)을 사용하면 터미널 없이도 Claude Code를 사용할 수 있습니다. [macOS](https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect?utm_source=claude_code\&utm_medium=docs), [Windows](https://claude.com/download?utm_source=claude_code\&utm_medium=docs), 또는 [Linux](/docs/en/desktop-linux)용으로 다운로드하세요.

  터미널 사용이 처음이신가요? 단계별 지침은 [터미널 가이드](/docs/en/terminal-guide)를 참조하세요.
</Tip>

Claude Code를 설치하려면 다음 방법 중 하나를 사용하세요:

<Tabs>
  <Tab title="네이티브 설치 (권장)">
    **macOS, Linux, WSL:**

    ```bash theme={null}
    curl -fsSL https://claude.ai/install.sh | bash
    ```

    **Windows PowerShell:**

    ```powershell theme={null}
    irm https://claude.ai/install.ps1 | iex
    ```

    **Windows CMD:**

    ```batch theme={null}
    curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
    ```

    `The token '&&' is not a valid statement separator`라는 메시지가 보이면 CMD가 아니라 PowerShell에 있는 것입니다. `'irm' is not recognized as an internal or external command`라는 메시지가 보이면 PowerShell이 아니라 CMD에 있는 것입니다. 프롬프트는 PowerShell에서는 `PS C:\`, CMD에서는 `PS` 없이 `C:\`로 표시됩니다.

    설치 명령이 `syntax error near unexpected token '<'`, `403` 또는 기타 curl 오류로 실패하는 경우 [설치 문제 해결](/docs/en/troubleshoot-install#find-your-error)에서 오류에 맞는 해결 방법 및 대체 설치 방법을 확인하세요.

    네이티브 Windows 환경에서는 Claude Code가 Bash 도구를 사용할 수 있도록 [Git for Windows](https://git-scm.com/downloads/win) 설치를 권장합니다. Git for Windows가 설치되어 있지 않으면 Claude Code는 대신 PowerShell을 셸 도구로 사용합니다. WSL 환경에는 Git for Windows가 필요하지 않습니다.

    <Info>
      네이티브 설치는 최신 버전을 유지하기 위해 백그라운드에서 자동으로 업데이트됩니다.
    </Info>
  </Tab>

  <Tab title="Homebrew">
    ```bash theme={null}
    brew install --cask claude-code
    ```

    Homebrew는 두 가지 cask를 제공합니다. `claude-code`는 일반적으로 약 1주일 정도 뒤처지고 주요 회귀가 있는 릴리스를 건너뛰는 안정적(stable) 릴리스 채널을 팔로우합니다. `claude-code@latest`는 최신(latest) 채널을 팔로우하고 출시되는 즉시 새 버전을 수신합니다.

    <Info>
      Homebrew 설치는 자동으로 업데이트되지 않습니다. 최신 기능과 보안 패치를 받으려면 설치한 cask에 따라 `brew upgrade claude-code` 또는 `brew upgrade claude-code@latest`를 실행하세요.
    </Info>
  </Tab>

  <Tab title="WinGet">
    ```powershell theme={null}
    winget install Anthropic.ClaudeCode
    ```

    <Info>
      WinGet 설치는 자동으로 업데이트되지 않습니다. 최신 기능과 보안 패치를 받으려면 주기적으로 `winget upgrade Anthropic.ClaudeCode`를 실행하세요.
    </Info>
  </Tab>
</Tabs>

Debian, Fedora, RHEL 및 Alpine에서는 [apt, dnf 또는 apk](/docs/en/setup#install-with-linux-package-managers)를 통해 설치할 수도 있습니다.

설치가 완료되면 작업하려는 프로젝트에서 터미널을 열고 Claude Code를 시작하세요:

```bash theme={null}
claude
```

Claude Code가 터미널에서 대화형 세션을 엽니다.

설정 중 문제가 발생하면 [설치 및 로그인 문제 해결](/docs/en/troubleshoot-install)을 참조하세요.

### Windows에서 설정하기

Windows에서 Claude Code를 네이티브로 실행하거나 WSL 내부에서 실행할 수 있습니다. 프로젝트가 위치한 곳과 필요한 기능에 따라 선택하세요:

| 옵션 | 필요 항목 | [샌드박싱](/docs/en/sandboxing) | 사용 시점 |
| :--- | :--- | :--- | :--- |
| 네이티브 Windows | 없음; [Git for Windows](https://git-scm.com/downloads/win)는 선택 사항 | 지원 안 됨 | Windows 네이티브 프로젝트 및 도구 |
| WSL 2 | WSL 2 활성화됨 | 지원됨 | Linux 툴체인 또는 샌드박스 처리된 명령어 실행 |
| WSL 1 | WSL 1 활성화됨 | 지원 안 됨 | WSL 2를 사용할 수 없는 경우 |

**옵션 1: 네이티브 Windows**

PowerShell 또는 CMD에서 설치 명령을 실행하세요. 관리자 권한으로 실행할 필요는 없습니다. [Git for Windows](https://git-scm.com/downloads/win) 설치는 선택 사항입니다. Git Bash를 제공하여 [Bash 도구](/docs/en/tools-reference#bash-tool-behavior)를 활성화합니다.

PowerShell 또는 CMD 중 어느 곳에서 설치하든 실행하는 설치 명령만 달라집니다. 프롬프트는 PowerShell에서 `PS C:\Users\YourName>`, CMD에서 `PS` 없이 `C:\Users\YourName>`으로 표시됩니다. 터미널이 처음이신 경우 [터미널 가이드](/docs/en/terminal-guide#windows)에서 각 단계를 설명합니다.

설치 후 아무 터미널에서나 `claude`를 실행하세요.

* **Git for Windows 미설치 시**, Claude Code는 [PowerShell 도구](/docs/en/tools-reference#powershell-tool)를 통해 셸 명령을 실행합니다.
* **Git for Windows 설치 시**, Claude Code는 [Bash 도구](/docs/en/tools-reference#bash-tool-behavior)용으로 Git Bash를 사용합니다. Claude Code가 Git Bash를 찾지 못하는 경우 [settings.json 파일](/docs/en/settings)에서 경로를 설정하세요:

  ```json theme={null}
  {
    "env": {
      "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
    }
  }
  ```

Git for Windows가 설치되어 있을 때 PowerShell 도구가 Bash와 함께 추가 옵션으로 점진적으로 배포되고 있습니다. 참가를 위해 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`, 제외를 위해 `0`으로 설정하세요. 설정 및 제한 사항은 [PowerShell 도구](/docs/en/tools-reference#powershell-tool)를 참조하세요.

**옵션 2: WSL**

WSL 배포판을 열고 위의 [설치 지침](#claude-code-설치)에서 Linux 인스톨러를 실행하세요. PowerShell이나 CMD가 아닌 WSL 터미널 내부에서 `claude`를 설치하고 실행합니다.

### Alpine Linux 및 musl 기반 배포판

Alpine 및 기타 musl/uClibc 기반 배포판에 Claude Code를 설치하려면 설치 명령에 `bash` 및 `curl`이 필요하고 런타임에 `libgcc`, `libstdc++`, `ripgrep`이 필요합니다. Alpine에는 기본적으로 `bash` 또는 `curl`이 포함되어 있지 않으므로 이를 설치할 때까지 문서화된 설치 명령이 `not found` 오류로 실패합니다. 배포판의 패키지 관리자를 사용하여 이 패키지들을 설치한 다음 `USE_BUILTIN_RIPGREP=0`을 설정하세요.

다음 예시는 Alpine에서 필요한 패키지를 설치합니다:

```bash theme={null}
apk add bash curl libgcc libstdc++ ripgrep
```

Alpine에서 `ripgrep`은 community 리포지토리에 있습니다. `apk`에서 패키지가 누락되었다고 보고하는 경우 Alpine 버전에 맞춰 `/etc/apk/repositories`에 community 리포지토리를 추가하세요:

```bash theme={null}
echo "https://dl-cdn.alpinelinux.org/alpine/v3.22/community" >> /etc/apk/repositories
```

`apk update`를 실행하여 패키지 인덱스를 새로 고치고 `apk add` 명령을 다시 시도하세요.

그런 다음 [`settings.json`](/docs/en/settings#available-settings) 파일에서 `USE_BUILTIN_RIPGREP`을 `0`으로 설정하세요:

```json theme={null}
{
  "env": {
    "USE_BUILTIN_RIPGREP": "0"
  }
}
```

## 설치 검증

설치 후 Claude Code가 작동하는지 확인합니다:

```bash theme={null}
claude --version
```

정상 작동하는 설치는 `2.1.211 (Claude Code)`와 같은 버전 번호를 출력합니다.

`command not found` 또는 기타 오류로 실패하는 경우 [설치 및 로그인 문제 해결](/docs/en/troubleshoot-install)을 참조하세요.

설치 및 구성에 대한 자세한 점검을 위해 [`claude doctor`](/docs/en/troubleshooting#get-more-help)를 실행하세요:

```bash theme={null}
claude doctor
```

`claude doctor`는 세션을 시작하지 않고 설치 상태, 설정 파일 유효성 검사 오류, 권장 해결 방법이 포함된 경고 등 읽기 전용 설치 및 설정 진단 정보를 출력합니다.

## 인증하기

Claude Code를 사용하려면 Pro, Max, Team, Enterprise 또는 Console 계정이 필요합니다. 무료 Claude.ai 플랜에는 Claude Code 접근 권한이 포함되어 있지 않습니다. 또한 [Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai) 또는 [Microsoft Foundry](/docs/en/microsoft-foundry)와 같은 서드파티 API 제공업체와 함께 Claude Code를 사용할 수도 있습니다.

설치 후 `claude`를 실행하고 브라우저 안내를 따라 로그인하세요. `ANTHROPIC_API_KEY` 환경 변수가 설정되어 있으면 Claude Code는 브라우저를 여는 대신 키 승인을 한 번 요청합니다. 모든 계정 유형 및 팀 설정 옵션은 [인증](/docs/en/authentication)을 참조하세요.

## Claude Code 업데이트

네이티브 설치는 백그라운드에서 자동으로 업데이트됩니다. [릴리스 채널을 구성](#릴리스-채널-구성)하여 즉시 업데이트를 받을지 지연된 안정적 일정을 따를지 제어하거나 [자동 업데이트를 완전히 비활성화](#자동-업데이트-비활성화)할 수 있습니다. Homebrew, WinGet 및 [Linux 패키지 관리자](#linux-패키지-관리자로-설치) 설치는 기본적으로 수동 업데이트가 필요합니다.

### 자동 업데이트

Claude Code는 시작 시 및 실행 중에 주기적으로 업데이트를 확인합니다. 업데이트는 백그라운드에서 다운로드 및 설치된 후 다음번에 Claude Code를 시작할 때 적용됩니다.

가장 최근 업데이트 시도의 결과를 보려면 `claude doctor`를 실행하세요.

macOS 및 Linux에서 네이티브 인스톨러는 `~/.local/bin/claude` 런처를 `~/.local/share/claude/versions/` 디렉터리를 가리키는 심볼릭 링크로 관리합니다. 해당 런처를 고유한 스크립트나 심볼릭 링크로 교체하더라도 자동 업데이트 및 `claude update`는 이를 유지합니다. 새 버전은 여전히 `versions/` 디렉터리 아래에 설치되며 런처가 실행할 버전을 결정합니다. v2.1.207 이전에는 자동 업데이트 프로그램이 매 업데이트마다 해당 경로의 사용자 지정 런처를 자체 심볼릭 링크로 교체했습니다.

사용자 지정 런처를 사용하면 Claude Code는 런처에 필요한 버전을 구분할 수 없기 때문에 설치된 모든 버전을 디스크에 유지합니다. `claude doctor`는 네이티브 인스톨러가 생성하지 않은 런처를 보고합니다.

Claude Code가 런처를 다시 관리하도록 하려면 `~/.local/bin/claude`를 제거하고 `claude update`를 실행하세요.

npm 전역 설치가 npm 전역 디렉터리에 쓰기 권한이 없어서 자동 업데이트를 할 수 없는 경우, Claude Code는 시작 시 일회성 알림을 표시하고 `claude doctor`에 가능한 해결 방법을 나열합니다. 자세한 내용은 [설치 중 권한 오류](/docs/en/troubleshoot-install#permission-errors-during-installation)를 참조하세요.

<Note>
  Homebrew, WinGet, apt, dnf 및 apk 설치는 기본적으로 자동 업데이트되지 않습니다. Homebrew 및 WinGet 참여 방법은 아래를 참조하세요. Homebrew를 수동으로 업그레이드하려면 설치한 cask에 따라 `brew upgrade claude-code` 또는 `brew upgrade claude-code@latest`를 실행하세요. WinGet의 경우 `winget upgrade Anthropic.ClaudeCode`를 실행하세요. Linux 패키지 관리자의 경우 [Linux 패키지 관리자로 설치](#linux-패키지-관리자로-설치)의 업그레이드 명령을 참조하세요.

  Claude Code가 Homebrew 또는 WinGet에서 사용자를 대신해 업그레이드 명령을 실행하도록 하려면 [`CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE`](/docs/en/env-vars)를 `1`로 설정하세요. 그러면 새 버전을 사용할 수 있을 때 Claude Code가 백그라운드에서 업그레이드를 실행하고 성공 시 재시작 프롬프트를 표시합니다. 업그레이드는 Claude Code 패키지만 타깃으로 하며 설치된 다른 소프트웨어에는 영향을 주지 않습니다.

  WinGet에서는 Windows가 실행 파일을 잠그기 때문에 Claude Code가 실행 중인 동안 업그레이드가 실패할 수 있습니다. 이 경우 Claude Code는 대신 수동 명령을 표시합니다. apt, dnf 및 apk는 이러한 명령에 상승된 권한이 필요하므로 수동 업그레이드가 계속 필요합니다.

  **알려진 문제:** 이러한 패키지 관리자에 새 버전을 사용할 수 있기 전에 Claude Code가 업데이트를 알릴 수 있습니다. 업그레이드가 실패하면 잠시 기다렸다가 나중에 다시 시도하세요.

  Homebrew는 업그레이드 후 디스크에 이전 버전을 유지합니다. 디스크 공간을 확보하려면 주기적으로 `brew cleanup`을 실행하세요.
</Note>

### 릴리스 채널 구성

`autoUpdatesChannel` 설정으로 자동 업데이트 및 `claude update`가 팔로우하는 릴리스 채널을 제어하세요:

* `"latest"` (기본값): 새 기능이 출시되는 즉시 받기
* `"stable"`: 일반적으로 약 1주일 이전 버전을 사용하며 주요 회귀가 있는 릴리스 건너뛰기

`/config` → **Auto-update channel**을 통해 구성하거나 [settings.json 파일](/docs/en/settings)에 추가하세요:

```json theme={null}
{
  "autoUpdatesChannel": "stable"
}
```

엔터프라이즈 배포의 경우 [관리형 설정](/docs/en/permissions#managed-settings)을 사용하여 조직 전체에 일관된 릴리스 채널을 강제 적용할 수 있습니다.

Homebrew 설치는 이 설정 대신 cask 이름으로 채널을 선택합니다. `claude-code`는 stable을, `claude-code@latest`는 latest를 팔로우합니다.

### 최소 버전 고정

`minimumVersion` 설정은 하한선을 설정합니다. 백그라운드 자동 업데이트 및 `claude update`는 이 값보다 낮은 버전의 설치를 거부하므로 `"stable"` 채널로 이동하더라도 이미 더 새로운 `"latest"` 빌드에 있는 경우 다운그레이드되지 않습니다.

`/config`를 통해 `"latest"`에서 `"stable"`로 전환하면 현재 버전에 머물거나 다운그레이드를 허용하도록 요청하는 프롬프트가 표시됩니다. 머물기를 선택하면 `minimumVersion`이 해당 버전으로 설정됩니다. `"latest"`로 다시 전환하면 설정이 지워집니다.

하한선을 명시적으로 고정하려면 [settings.json 파일](/docs/en/settings)에 추가하세요:

```json theme={null}
{
  "autoUpdatesChannel": "stable",
  "minimumVersion": "2.1.100"
}
```

[관리형 설정](/docs/en/permissions#managed-settings)에서는 이것이 사용자 및 프로젝트 설정으로 오버라이드할 수 없는 조직 전반의 최소 기준을 강제 적용합니다.

`minimumVersion` 고정은 업데이트만 제어합니다. Claude Code가 버전 범위 밖에서 시작하는 것을 거부하도록 하려면 대신 관리형 설정 `requiredMinimumVersion` 및 `requiredMaximumVersion`을 사용하세요. 업데이트도 `requiredMaximumVersion` 상한을 존중합니다. [사용 가능한 설정](/docs/en/settings#available-settings)을 참조하세요.

### 자동 업데이트 비활성화

[`settings.json`](/docs/en/settings#available-settings) 파일의 `env` 키에서 `DISABLE_AUTOUPDATER`를 `"1"`로 설정하세요:

```json theme={null}
{
  "env": {
    "DISABLE_AUTOUPDATER": "1"
  }
}
```

`DISABLE_AUTOUPDATER`는 백그라운드 확인만 중지합니다. `claude update` 및 `claude install`은 여전히 작동합니다. 수동 업데이트를 포함한 모든 업데이트 경로를 차단하려면 대신 [`DISABLE_UPDATES`](/docs/en/env-vars)를 설정하세요. 사용자 지정 채널을 통해 Claude Code를 배포하고 사용자가 제공된 버전에 머물러야 할 때 사용하세요.

### 수동 업데이트

다음 백그라운드 확인을 기다리지 않고 즉시 업데이트를 적용하려면 다음을 실행하세요:

```bash theme={null}
claude update
```

업데이트가 설치되면 명령이 `Successfully updated from <old version> to version <new version>`이라고 보고합니다. 이미 최신 버전에 있는 경우 `Claude Code is up to date (<version>)`이라고 보고합니다. Homebrew, WinGet 또는 apk로 관리되는 설치는 대신 `Claude is up to date!`라고 보고합니다.

## 고급 설치 옵션

이 옵션들은 버전 고정, Linux 패키지 관리자, npm 및 바이너리 무결성 검증을 위한 것입니다.

### 특정 버전 설치

네이티브 인스톨러는 특정 버전 번호 또는 릴리스 채널(`latest` 또는 `stable`)을 허용합니다. 설치 시 선택한 채널이 자동 업데이트의 기본값이 됩니다. 자세한 내용은 [릴리스 채널 구성](#릴리스-채널-구성)을 참조하세요.

최신 버전을 설치하려면 (기본값):

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    curl -fsSL https://claude.ai/install.sh | bash
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    irm https://claude.ai/install.ps1 | iex
    ```
  </Tab>

  <Tab title="Windows CMD">
    ```batch theme={null}
    curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
    ```
  </Tab>
</Tabs>

안정적인 버전을 설치하려면:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    curl -fsSL https://claude.ai/install.sh | bash -s stable
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    & ([scriptblock]::Create((irm https://claude.ai/install.ps1))) stable
    ```
  </Tab>

  <Tab title="Windows CMD">
    ```batch theme={null}
    curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd stable && del install.cmd
    ```
  </Tab>
</Tabs>

특정 버전 번호를 설치하려면:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    curl -fsSL https://claude.ai/install.sh | bash -s 2.1.89
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    & ([scriptblock]::Create((irm https://claude.ai/install.ps1))) 2.1.89
    ```
  </Tab>

  <Tab title="Windows CMD">
    ```batch theme={null}
    curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd 2.1.89 && del install.cmd
    ```
  </Tab>
</Tabs>

### Linux 패키지 관리자로 설치

Claude Code는 서명된 apt, dnf 및 apk 리포지토리를 게시합니다. 각 리포지토리는 두 개의 채널을 제공합니다. `stable`은 일반적으로 약 1주일 이전 버전을 제공하여 주요 회귀가 있는 릴리스를 건너뛰고, `latest`는 출시되는 즉시 모든 릴리스를 제공합니다. 아래 명령은 대부분의 사용자에게 적합한 `stable` 채널을 구성합니다. 각 탭은 `latest` 리포지토리 URL도 보여줍니다. 패키지 관리자 설치는 Claude Code를 통해 자동 업데이트되지 않으며 일반적인 시스템 업그레이드 워크플로를 통해 업데이트가 제공됩니다.

모든 리포지토리는 [Claude Code 릴리스 서명 키](#바이너리-무결성-및-코드-서명)로 서명됩니다. 키를 신뢰하기 전에 각 탭에 설명된 대로 검증하세요.

<Tabs>
  <Tab title="apt">
    Debian 및 Ubuntu용. 아래 설치 명령은 `curl`로 서명 키를 다운로드하며, 새로 설치된 Debian 및 Ubuntu에는 curl이 포함되어 있지 않을 수 있습니다. 다운로드가 `sudo: curl: command not found`로 실패하는 경우 먼저 curl을 설치하세요:

    ```bash theme={null}
    sudo apt install curl
    ```

    다음 명령은 `stable` 채널을 구성합니다:

    ```bash theme={null}
    sudo install -d -m 0755 /etc/apt/keyrings
    sudo curl -fsSL https://downloads.claude.ai/keys/claude-code.asc \
      -o /etc/apt/keyrings/claude-code.asc
    echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" \
      | sudo tee /etc/apt/sources.list.d/claude-code.list
    sudo apt update
    sudo apt install claude-code
    ```

    대신 `latest` 채널을 사용하려면 URL 경로와 제품군 이름이 모두 달라집니다. 다음 `deb` 라인을 사용하세요:

    ```bash theme={null}
    echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/latest latest main" \
      | sudo tee /etc/apt/sources.list.d/claude-code.list
    ```

    신뢰하기 전에 GPG 키 지문을 확인하세요: `gpg --show-keys /etc/apt/keyrings/claude-code.asc`는 `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`를 보고해야 합니다.

    나중에 업그레이드하려면 `sudo apt update && sudo apt upgrade claude-code`를 실행하세요.
  </Tab>

  <Tab title="dnf">
    Fedora 및 RHEL용. 다음 명령은 `stable` 채널을 구성합니다:

    ```bash theme={null}
    sudo tee /etc/yum.repos.d/claude-code.repo <<'EOF'
    [claude-code]
    name=Claude Code
    baseurl=https://downloads.claude.ai/claude-code/rpm/stable
    enabled=1
    gpgcheck=1
    gpgkey=https://downloads.claude.ai/keys/claude-code.asc
    EOF
    sudo dnf install claude-code
    ```

    대신 `latest` 채널을 사용하려면 `baseurl`을 `latest` 리포지토리로 설정하세요:

    ```ini theme={null}
    baseurl=https://downloads.claude.ai/claude-code/rpm/latest
    ```

    dnf는 첫 설치 시 키를 다운로드하고 지문을 확인하라는 프롬프트를 표시합니다. 수락하기 전에 `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`와 일치하는지 확인하세요.

    나중에 업그레이드하려면 `sudo dnf upgrade claude-code`를 실행하세요.
  </Tab>

  <Tab title="apk">
    Alpine Linux용. 다음 명령은 `stable` 채널을 구성합니다:

    ```sh theme={null}
    wget -O /etc/apk/keys/claude-code.rsa.pub \
      https://downloads.claude.ai/keys/claude-code.rsa.pub
    echo "https://downloads.claude.ai/claude-code/apk/stable" >> /etc/apk/repositories
    apk add claude-code
    ```

    `latest` 채널로 전환하려면 `stable` 리포지토리 라인을 제거하고 `latest` 리포지토리를 추가하세요:

    ```sh theme={null}
    sed -i '\|downloads.claude.ai/claude-code/apk/stable|d' /etc/apk/repositories
    echo "https://downloads.claude.ai/claude-code/apk/latest" >> /etc/apk/repositories
    ```

    `sha256sum /etc/apk/keys/claude-code.rsa.pub`로 다운로드한 키를 확인하세요. `395759c1f7449ef4cdef305a42e820f3c766d6090d142634ebdb049f113168b6`을 보고해야 합니다.

    나중에 업그레이드하려면 `apk update && apk upgrade claude-code`를 실행하세요.
  </Tab>
</Tabs>

### npm으로 설치

Claude Code를 전역 npm 패키지로 설치할 수도 있습니다. v2.1.198부터 npm 패키지는 [Node.js 22 이상](https://nodejs.org/en/download)이 필요합니다. 이전 Node.js 버전에서는 설치 중 실패하는 대신 npm이 `EBADENGINE` 경고를 출력하며, 패키지가 런타임에 사용자 Node.js를 사용하지 않는 네이티브 바이너리를 다운로드하므로 설치가 완료되고 `claude`가 계속 실행됩니다.

```bash theme={null}
npm install -g @anthropic-ai/claude-code
```

npm 패키지는 독립형 인스톨러와 동일한 네이티브 바이너리를 설치합니다. npm은 `@anthropic-ai/claude-code-darwin-arm64`와 같은 플랫폼별 선택적 종속성을 통해 바이너리를 가져오며, postinstall 단계가 이를 제자리에 연결합니다. 설치된 `claude` 바이너리 자체가 Node를 호출하지는 않습니다.

지원되는 npm 설치 플랫폼은 `darwin-arm64`, `darwin-x64`, `linux-x64`, `linux-arm64`, `linux-x64-musl`, `linux-arm64-musl`, `win32-x64` 및 `win32-arm64`입니다. 패키지 관리자가 선택적 종속성을 허용해야 합니다. 설치 후 바이너리가 누락된 경우 [문제 해결](/docs/en/troubleshoot-install#native-binary-not-found-after-npm-install)을 참조하세요.

npm 설치를 업그레이드하려면 `npm install -g @anthropic-ai/claude-code@latest`를 실행하세요. 원래 설치의 semver 범위를 존중하여 최신 릴리스로 이동하지 않을 수 있는 `npm update -g`는 사용하지 마세요.

<Warning>
  `sudo npm install -g`를 사용하지 마세요. 권한 문제 및 보안 위험이 발생할 수 있습니다. 권한 오류가 발생하는 경우 [권한 오류 문제 해결](/docs/en/troubleshoot-install#permission-errors-during-installation)을 참조하세요.
</Warning>

### 바이너리 무결성 및 코드 서명

각 릴리스는 모든 플랫폼 바이너리에 대한 SHA256 체크섬이 포함된 `manifest.json`을 게시합니다. 매니페스트는 Anthropic GPG 키로 서명되므로 매니페스트의 서명을 검증하면 나열된 모든 바이너리가 이행적으로 검증됩니다.

#### 매니페스트 서명 검증

1~3단계는 `gpg` 및 `curl`이 있는 POSIX 셸이 필요합니다. Windows의 경우 Git Bash 또는 WSL에서 실행하세요. 4단계에는 PowerShell 옵션이 포함되어 있습니다.

<Steps>
  <Step title="공개 키 다운로드 및 임포트">
    릴리스 서명 키는 고정된 URL에 게시되어 있습니다.

    ```bash theme={null}
    curl -fsSL https://downloads.claude.ai/keys/claude-code.asc | gpg --import
    ```

    임포트된 키의 지문을 표시합니다.

    ```bash theme={null}
    gpg --fingerprint security@anthropic.com
    ```

    출력에 다음 지문이 포함되어 있는지 확인합니다:

    ```text theme={null}
    31DD DE24 DDFA B679 F42D  7BD2 BAA9 29FF 1A7E CACE
    ```
  </Step>

  <Step title="매니페스트 및 서명 다운로드">
    `VERSION`을 검증하려는 릴리스로 설정합니다.

    ```bash theme={null}
    REPO=https://downloads.claude.ai/claude-code-releases
    VERSION=2.1.89
    curl -fsSLO "$REPO/$VERSION/manifest.json"
    curl -fsSLO "$REPO/$VERSION/manifest.json.sig"
    ```
  </Step>

  <Step title="서명 검증">
    매니페스트에 대해 분리된(detached) 서명을 검증합니다.

    ```bash theme={null}
    gpg --verify manifest.json.sig manifest.json
    ```

    유효한 결과는 `Good signature from "Anthropic Claude Code Release Signing <security@anthropic.com>"`을 보고합니다.

    새로 임포트한 키에 대해 `gpg`는 `WARNING: This key is not certified with a trusted signature!`도 출력합니다. 이는 예상된 결과입니다. `Good signature` 라인은 암호화 검사가 통과했음을 확인합니다. 1단계의 지문 비교는 키 자체의 진위 여부를 확인합니다.
  </Step>

  <Step title="매니페스트와 바이너리 비교 점검">
    바이너리의 SHA256 체크섬을 `manifest.json`의 `platforms.<platform>.checksum` 아래 나열된 값과 비교합니다. 아래 명령은 현재 디렉터리에 `claude` 바이너리가 있다고 가정합니다. 설치된 네이티브 바이너리를 대신 검증하려면 `~/.local/share/claude/versions/VERSION`에 대해 명령을 실행하세요(VERSION을 2단계에서 설정한 릴리스로 교체).

    <Tabs>
      <Tab title="Linux">
        ```bash theme={null}
        sha256sum claude
        ```
      </Tab>

      <Tab title="macOS">
        ```bash theme={null}
        shasum -a 256 claude
        ```
      </Tab>

      <Tab title="Windows PowerShell">
        ```powershell theme={null}
        (Get-FileHash claude.exe -Algorithm SHA256).Hash.ToLower()
        ```
      </Tab>
    </Tabs>
  </Step>
</Steps>

<Note>
  매니페스트 서명은 `2.1.89` 이후 릴리스부터 사용할 수 있습니다. 이전 릴리스는 분리된 서명 없이 `manifest.json`에 체크섬을 게시합니다.
</Note>

#### 플랫폼 코드 서명

서명된 매니페스트 외에도 개별 바이너리는 지원되는 경우 플랫폼 네이티브 코드 서명을 전달합니다.

* **macOS**: "Anthropic PBC"에 의해 서명되고 Apple에 의해 공증됨. `codesign --verify --verbose ./claude`로 검증하세요.
* **Windows**: "Anthropic, PBC"에 의해 서명됨. `Get-AuthenticodeSignature .\claude.exe`로 검증하세요.
* **Linux**: 바이너리가 개별적으로 코드 서명되지 않습니다. `claude-code-releases` 버킷에서 직접 다운로드하거나 네이티브 인스톨러를 사용하는 경우 위 매니페스트 서명으로 무결성을 검증하세요. [apt, dnf 또는 apk](#linux-패키지-관리자로-설치)로 설치하는 경우 패키지 관리자가 리포지토리 서명 키를 사용하여 자동으로 서명을 검증합니다.

## Claude Code 제거

Claude Code를 제거하려면 설치 방법에 따른 지침을 따르세요. 그 후에도 `claude`가 계속 실행되면 두 번째 설치가 있거나 이전 인스톨러의 셸 알리아스가 남아 있을 가능성이 높습니다. 찾아 제거하려면 [충돌하는 설치 점검](/docs/en/troubleshoot-install#check-for-conflicting-installations)을 참조하세요.

### 네이티브 설치

Claude Code 바이너리 및 버전 파일을 제거합니다:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    rm -f ~/.local/bin/claude
    rm -rf ~/.local/share/claude
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    Remove-Item -Path "$env:USERPROFILE\.local\bin\claude.exe" -Force
    Remove-Item -Path "$env:USERPROFILE\.local\share\claude" -Recurse -Force
    ```
  </Tab>
</Tabs>

### Homebrew 설치

설치한 Homebrew cask를 제거합니다. stable cask를 설치한 경우:

```bash theme={null}
brew uninstall --cask claude-code
```

latest cask를 설치한 경우:

```bash theme={null}
brew uninstall --cask claude-code@latest
```

### WinGet 설치

WinGet 패키지를 제거합니다:

```powershell theme={null}
winget uninstall Anthropic.ClaudeCode
```

### apt / dnf / apk

패키지 및 리포지토리 구성을 제거합니다:

<Tabs>
  <Tab title="apt">
    ```bash theme={null}
    sudo apt remove claude-code
    sudo rm /etc/apt/sources.list.d/claude-code.list /etc/apt/keyrings/claude-code.asc
    ```
  </Tab>

  <Tab title="dnf">
    ```bash theme={null}
    sudo dnf remove claude-code
    sudo rm /etc/yum.repos.d/claude-code.repo
    ```
  </Tab>

  <Tab title="apk">
    ```sh theme={null}
    apk del claude-code
    sed -i '\|downloads.claude.ai/claude-code/apk|d' /etc/apk/repositories
    rm /etc/apk/keys/claude-code.rsa.pub
    ```
  </Tab>
</Tabs>

### npm

전역 npm 패키지를 제거합니다:

```bash theme={null}
npm uninstall -g @anthropic-ai/claude-code
```

### 구성 파일 제거

<Warning>
  구성 파일을 제거하면 모든 설정, 허용된 도구, MCP 서버 구성 및 세션 기록이 삭제됩니다.
</Warning>

VS Code 확장 프로그램, JetBrains 플러그인 및 데스크톱 앱도 `~/.claude/`에 기록합니다. 이들 중 어느 것이라도 여전히 설치되어 있으면 다음번에 실행될 때 디렉터리가 다시 생성됩니다. Claude Code를 완전히 제거하려면 이 파일들을 삭제하기 전에 [VS Code 확장 프로그램](/docs/en/vs-code#uninstall-the-extension), JetBrains 플러그인 및 데스크톱 앱을 제거하세요.

Claude Code 설정 및 캐시된 데이터를 제거하려면:

<Tabs>
  <Tab title="macOS, Linux, WSL">
    ```bash theme={null}
    # 사용자 설정 및 상태 제거
    rm -rf ~/.claude
    rm ~/.claude.json

    # 프로젝트 특정 설정 제거 (프로젝트 디렉터리에서 실행)
    rm -rf .claude
    rm -f .mcp.json
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    # 사용자 설정 및 상태 제거
    Remove-Item -Path "$env:USERPROFILE\.claude" -Recurse -Force
    Remove-Item -Path "$env:USERPROFILE\.claude.json" -Force

    # 프로젝트 특정 설정 제거 (프로젝트 디렉터리에서 실행)
    Remove-Item -Path ".claude" -Recurse -Force
    Remove-Item -Path ".mcp.json" -Force
    ```
  </Tab>
</Tabs>
