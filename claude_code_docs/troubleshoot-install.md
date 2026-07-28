> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 설치 및 로그인 문제 해결

> Claude Code 설치 또는 로그인 시 발생할 수 있는 command not found, PATH, 권한, 네트워크 및 인증 오류를 해결하세요.

설치에 실패하거나 로그인할 수 없는 경우 아래에서 오류 항목을 찾으세요. Claude Code가 정상 작동한 이후의 런타임 문제는 [Troubleshooting](/docs/en/troubleshooting)을 참조하세요. 설정 미적용이나 훅 미작동과 같은 구성 문제는 [Debug your configuration](/docs/en/debug-your-config)을 참조하세요.

## 오류 찾기

발생한 오류 메시지나 증상을 해결 방법과 일치시키세요:

| 발생 증상 | 해결 방법 |
| :-------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| `command not found: claude` 또는 `'claude' is not recognized` | [PATH 수정하기](#command-not-found-claude-after-installation) |
| `syntax error near unexpected token '<'` | [설치 스크립트가 HTML을 반환함](#install-script-returns-html-instead-of-a-shell-script) |
| `curl: (22) The requested URL returned error: 403` | [설치 스크립트가 403을 반환함](#install-script-returns-html-instead-of-a-shell-script) |
| `curl: (23)` 또는 `curl: (56) Failure writing output to destination` | [연결 확인 또는 대체 설치 프로그램 사용](#curl-56-failure-writing-output-to-destination) |
| Linux 설치 중 `Killed` 또는 `Installation was killed before it could finish (exit code 137)` | [메모리 확보 또는 스왑 공간 추가](#install-killed-on-low-memory-linux-servers) |
| `TLS connect error` 또는 `SSL/TLS secure channel` | [CA 인증서 업데이트](#tls-or-ssl-connection-errors) |
| `Failed to fetch version` 또는 다운로드 서버 접근 불가 | [네트워크 및 프록시 설정 확인](#check-network-connectivity) |
| `irm is not recognized` 또는 `&& is not valid` | [사용 중인 쉘에 맞는 올바른 명령 사용](#wrong-install-command-on-windows) |
| `Cask 'claude-code' is unavailable: No Cask with this name exists` | [Homebrew 업데이트](#homebrew-cask-unavailable-or-outdated) |
| `'bash' is not recognized as the name of a cmdlet` | [Windows용 설치 프로그램 명령 사용](#wrong-install-command-on-windows) |
| `A parameter cannot be found that matches parameter name 'fsSL'` | [Windows용 설치 프로그램 명령 사용](#wrong-install-command-on-windows) |
| `Claude Code on Windows requires either Git for Windows (for bash) or PowerShell` | [쉘 설치](#claude-code-on-windows-requires-either-git-for-windows-for-bash-or-powershell) |
| `Claude Code does not support 32-bit Windows` | [x86 항목이 아닌 Windows PowerShell 열기](#claude-code-does-not-support-32-bit-windows) |
| `The process cannot access the file ... because it is being used by another process` | [다운로드 폴더를 비우고 재시도](#the-process-cannot-access-the-file-during-windows-install) |
| `Error loading shared library` | [시스템에 맞지 않는 바이너리 변형](#linux-musl-or-glibc-binary-mismatch) |
| `Illegal instruction` | [아키텍처 또는 CPU 명령어 세트 불일치](#illegal-instruction) |
| WSL에서 `cannot execute binary file: Exec format error` | [WSL1 네이티브 바이너리 회귀 문제](#exec-format-error-on-wsl1) |
| PowerShell 설치 프로그램이 완료되었으나 `claude`를 찾을 수 없거나 이전 버전을 표시함 | [PATH에 설치 디렉터리 추가](#verify-your-path) 후 새 터미널 열기 |
| macOS에서 `dyld: cannot load`, `dyld: Symbol not found`, 또는 `Abort trap` | [바이너리 호환성 문제](#dyld-cannot-load-on-macos) |
| `Checking for updates` 직후 `claude update`가 멈추거나 `claude doctor`가 출력 없이 멈춤 | [쉘 구성 경로에 위치한 디렉터리 이동](#claude-update-or-claude-doctor-hangs) |
| `Invoke-Expression: Missing argument in parameter list` | [설치 스크립트가 HTML을 반환함](#install-script-returns-html-instead-of-a-shell-script) |
| `App unavailable in region` | 가용 지역이 아닙니다. [지원 국가](https://www.anthropic.com/supported-countries) 참조. |
| `unable to get local issuer certificate` | [기업 CA 인증서 구성](#tls-or-ssl-connection-errors) |
| `OAuth error` 또는 `403 Forbidden` | [인증 문제 해결](#login-and-authentication) |
| `Could not load the default credentials` 또는 `Could not load credentials from any providers` | [Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry 자격 증명](#bedrock-agent-platform-or-foundry-credentials-not-loading) |
| `ChainedTokenCredential authentication failed` 또는 `CredentialUnavailableError` | [Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry 자격 증명](#bedrock-agent-platform-or-foundry-credentials-not-loading) |
| `API Error: 500`, `529 Overloaded`, `429`, 또는 기타 여기에 언급되지 않은 4xx/5xx 오류 | [오류 참조 문서](/docs/en/errors) 참조 |

문제가 나열되어 있지 않은 경우 아래의 진단 검사를 통해 원인을 좁혀보세요.

<Tip>
  터미널을 전혀 사용하지 않고 싶다면 [Claude Code Desktop 앱](/docs/en/desktop-quickstart)을 통해 그래픽 인터페이스로 Claude Code를 설치하고 사용할 수 있습니다. [macOS](https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect?utm_source=claude_code\&utm_medium=docs) 또는 [Windows](https://claude.com/download?utm_source=claude_code\&utm_medium=docs)용 앱을 다운로드하여 명령줄 설정 없이 코딩을 시작하세요. Linux에서는 [Linux 설치 지침](/docs/en/desktop-linux)을 따라 apt로 앱을 설치할 수 있습니다.
</Tip>

## 진단 검사 실행

### 네트워크 연결 확인

설치 프로그램은 `downloads.claude.ai`에서 다운로드합니다. 도달 가능한지 확인하세요:

```bash theme={null}
curl -sI https://downloads.claude.ai/claude-code-releases/latest
```

PowerShell에서는 `curl.exe -sI`를 실행하세요. PowerShell은 `curl`을 `-sI` 플래그를 거부하는 `Invoke-WebRequest`로 별칭(alias) 처리합니다.

`HTTP/2 200` 라인은 서버에 정상적으로 도달했음을 의미합니다. 다른 결과는 원인을 가리킵니다:

* `403`: 보통 프록시나 네트워크 필터가 호스트를 차단 중이거나, 사용자의 지역에서 Claude Code를 [사용할 수 없음](https://www.anthropic.com/supported-countries)을 의미합니다.
* `5xx`: 보통 일시적인 서비스 문제이므로 몇 분 후 재시도하세요.

출력이 없거나 `Could not resolve host`, 연결 시간 초과가 발생하는 경우 네트워크에서 연결을 차단하고 있는 것입니다. 일반적인 원인:

* 기업 방화벽 또는 프록시가 `downloads.claude.ai`를 차단함
* 지역 네트워크 제한: VPN 또는 다른 네트워크를 시도해 보세요.
* TLS/SSL 문제: 시스템의 CA 인증서를 업데이트하거나 `HTTPS_PROXY`가 구성되어 있는지 확인하세요.

기업 프록시 환경인 경우 설치 전 `HTTPS_PROXY` 및 `HTTP_PROXY`를 프록시 주소로 설정하세요. 프록시 URL을 모르는 경우 IT 팀에 문의하거나 브라우저 프록시 설정을 확인하세요.

다음 예시는 두 프록시 변수를 모두 설정한 후 프록시를 통해 설치 프로그램을 실행합니다:

<Tabs>
  <Tab title="macOS/Linux">
    ```bash theme={null}
    export HTTP_PROXY=http://proxy.example.com:8080
    export HTTPS_PROXY=http://proxy.example.com:8080
    curl -fsSL https://claude.ai/install.sh | bash
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    $env:HTTP_PROXY = 'http://proxy.example.com:8080'
    $env:HTTPS_PROXY = 'http://proxy.example.com:8080'
    irm https://claude.ai/install.ps1 | iex
    ```
  </Tab>
</Tabs>

### PATH 확인

설치는 성공했으나 `claude` 실행 시 `command not found` 또는 `not recognized` 오류가 발생하는 경우 설치 디렉터리가 PATH에 포함되어 있지 않은 것입니다. 쉘은 PATH에 나열된 디렉터리에서 프로그램을 찾으며, 설치 프로그램은 `claude`를 macOS/Linux에서는 `~/.local/bin/claude`에, Windows에서는 `%USERPROFILE%\.local\bin\claude.exe`에 배치합니다.

<Note>
  [VS Code 확장 프로그램](/docs/en/vs-code)은 `claude`를 이 위치에 배치하지 않습니다. 확장 프로그램 디렉터리 내에 번들로 제공되는 개인 복사본을 자체 채팅 패널용으로 사용하며 PATH에 추가하지 않습니다. 확장 프로그램만 설치한 경우 `~/.local/bin/claude`가 존재하지 않습니다. 터미널에서 `claude`를 사용하려면 [단독 설치](/docs/en/setup)를 실행한 후 아래 지침을 계속하세요.
</Note>

PATH 항목을 나열하고 `local/bin`으로 필터링하여 설치 디렉터리가 PATH에 있는지 확인하세요:

<Tabs>
  <Tab title="macOS/Linux">
    ```bash theme={null}
    echo $PATH | tr ':' '\n' | grep -Fx "$HOME/.local/bin"
    ```

    `/Users/you/.local/bin` 또는 `/home/you/.local/bin`이 출력되면 디렉터리가 PATH에 있는 것이므로 [충돌하는 설치 확인](#check-for-conflicting-installations)으로 이동하세요. 출력이 없다면 쉘 구성 파일에 추가해야 합니다.

    macOS 기본 쉘인 Zsh의 경우:

    ```bash theme={null}
    echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
    source ~/.zshrc
    ```

    대부분의 Linux 배포판 기본 쉘인 Bash의 경우:

    ```bash theme={null}
    echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    ```

    또는 터미널을 닫고 다시 열 수도 있습니다.

    fish나 Nushell과 같은 다른 쉘의 경우 해당하는 쉘 구성 구문을 사용하여 `~/.local/bin`을 PATH에 추가한 후 터미널을 재시작하세요.

    수정이 완료되었는지 확인합니다:

    ```bash theme={null}
    claude --version
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    ```powershell theme={null}
    $env:PATH -split ';' | Select-String '\.local\\bin'
    ```

    출력이 없다면 사용자 PATH에 설치 디렉터리를 추가하세요:

    ```powershell theme={null}
    $currentPath = [Environment]::GetEnvironmentVariable('PATH', 'User')
    [Environment]::SetEnvironmentVariable('PATH', "$currentPath;$env:USERPROFILE\.local\bin", 'User')
    ```

    변경 사항이 적용되도록 터미널을 재시작하세요.

    수정이 완료되었는지 확인합니다:

    ```powershell theme={null}
    claude --version
    ```
  </Tab>

  <Tab title="Windows CMD">
    ```batch theme={null}
    echo %PATH% | findstr /i "local\bin"
    ```

    출력이 없다면 시스템 설정을 열고 환경 변수로 이동하여 사용자 PATH 변수에 `%USERPROFILE%\.local\bin`을 추가하세요. 터미널을 재시작합니다.

    수정이 완료되었는지 확인합니다:

    ```batch theme={null}
    claude --version
    ```
  </Tab>
</Tabs>

### 충돌하는 설치 확인

여러 버전의 Claude Code가 설치되어 있으면 버전 불일치나 예기치 않은 동작이 발생할 수 있습니다. 설치된 항목을 확인하세요:

<Tabs>
  <Tab title="macOS/Linux">
    PATH에서 찾은 모든 `claude` 바이너리를 나열합니다:

    ```bash theme={null}
    which -a claude
    ```

    아무것도 출력되지 않으면 PATH에 `claude`가 아직 없는 것입니다. [PATH 확인](#verify-your-path)으로 돌아가세요.

    `claude` 바이너리가 존재할 수 있는 세 지점을 확인하세요. `~/.local/bin/claude`는 네이티브 설치 프로그램 위치이고, `~/.claude/local/`은 이전 버전의 Claude Code가 생성한 레거시 로컬 npm 설치 위치이며, npm 글로벌 목록은 `-g` 설치를 보여줍니다:

    ```bash theme={null}
    ls -la ~/.local/bin/claude
    ```

    네이티브 설치는 `~/.local/share/claude/versions/`로 연결되는 심볼릭 링크를 보여줍니다. 이 경로에 직접 만든 스크립트나 심볼릭 링크는 커스텀 런처이며, [자동 업데이트 시 그대로 유지](/docs/en/setup#auto-updates)됩니다.

    `ls` 명령에서 `No such file or directory`가 출력되더라도 오류가 아닙니다. 해당 위치에 설치된 것이 없다는 의미이므로 다음 확인 단계로 이동하세요.

    ```bash theme={null}
    ls -la ~/.claude/local/
    ```

    ```bash theme={null}
    npm -g ls @anthropic-ai/claude-code 2>/dev/null
    ```
  </Tab>

  <Tab title="Windows PowerShell">
    PATH에서 찾은 모든 `claude` 바이너리를 나열합니다:

    ```powershell theme={null}
    where.exe claude
    ```

    네이티브 설치 프로그램이 바이너리를 배치했는지 확인합니다:

    ```powershell theme={null}
    Test-Path "$env:USERPROFILE\.local\bin\claude.exe"
    ```
  </Tab>
</Tabs>

여러 설치가 발견되면 하나만 유지하세요. macOS/Linux의 `~/.local/bin/claude` 또는 Windows의 `%USERPROFILE%\.local\bin\claude.exe`에 있는 네이티브 설치를 권장합니다. 여분의 설치를 제거하세요:

npm 글로벌 설치 삭제:

```bash theme={null}
npm uninstall -g @anthropic-ai/claude-code
```

레거시 로컬 npm 설치 제거:

```bash theme={null}
rm -rf ~/.claude/local
```

Windows에서는 PowerShell을 사용하세요:

```powershell theme={null}
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\local"
```

macOS에서 Homebrew 설치 제거 (`claude-code@latest` 카스크를 설치한 경우 해당 이름으로 대체):

```bash theme={null}
brew uninstall --cask claude-code
```

Windows에서 WinGet 설치 제거:

```powershell theme={null}
winget uninstall Anthropic.ClaudeCode
```

### 디렉터리 권한 확인

설치 프로그램은 macOS 및 Linux의 `~/.local/bin/` 및 `~/.claude/`에 대한 쓰기 권한이 필요합니다. Windows의 설치 위치는 기본적으로 사용자에게 쓰기 권한이 있는 `%USERPROFILE%` 아래이므로 이 섹션이 거의 적용되지 않습니다.

디렉터리에 쓰기 권한이 있는지 확인하세요:

```bash theme={null}
test -w ~/.local/bin && echo "writable" || echo "not writable"
test -w ~/.claude && echo "writable" || echo "not writable"
```

디렉터리에 쓰기 권한이 없다면 설치 디렉터리를 생성하고 사용자를 소유자로 설정하세요:

```bash theme={null}
sudo mkdir -p ~/.local/bin
sudo chown -R $(whoami) ~/.local
```

### 바이너리 작동 확인

`claude --version`이 버전을 출력하지만 시작 시 `claude`가 크래시되거나 멈추는 경우 원인을 좁히기 위해 다음 검사를 실행하세요. `claude --version`에서 command not found가 표시되면 먼저 [PATH 확인](#verify-your-path)으로 이동하세요. 아래 명령은 `claude`가 PATH에 있다고 가정합니다.

바이너리가 존재하고 실행 가능한지 확인합니다:

```bash theme={null}
ls -la "$(command -v claude)"
```

Windows에서는 PowerShell을 사용하세요:

```powershell theme={null}
Get-Command claude | Select-Object Source
```

Linux에서는 누락된 공유 라이브러리를 확인하세요. `ldd`에 누락된 라이브러리가 표시되면 시스템 패키지를 설치해야 할 수 있습니다. Alpine Linux 및 기타 musl 기반 배포판의 경우 [Alpine Linux setup](/docs/en/setup#alpine-linux-and-musl-based-distributions)을 참조하세요.

```bash theme={null}
ldd "$(command -v claude)" | grep "not found"
```

바이너리가 실행될 수 있는지 확인합니다:

```bash theme={null}
claude --version
```

## 일반적인 설치 문제

가장 자주 발생하는 설치 문제 및 해결 방법입니다.

### 설치 스크립트가 쉘 스크립트 대신 HTML을 반환함

설치 명령을 실행할 때 다음과 같은 오류 중 하나가 표시될 수 있습니다:

```text theme={null}
bash: line 1: syntax error near unexpected token `<'
bash: line 1: `<!DOCTYPE html>'
```

PowerShell에서는 동일한 문제가 다음과 같이 표시됩니다:

```text theme={null}
Invoke-Expression: Missing argument in parameter list.
```

요청 라우팅 방식에 따라 HTML 본문이 없는 403 오류가 대신 표시될 수도 있습니다:

```text theme={null}
curl: (22) The requested URL returned error: 403
```

이들은 모두 설치 URL이 설치 스크립트 대신 HTML 페이지나 오류 상태를 반환했음을 의미합니다. HTML 페이지에 "App unavailable in region"이라고 표시되면 해당 국가에서 Claude Code를 사용할 수 없는 것입니다. [지원 국가](https://www.anthropic.com/supported-countries)를 참조하세요.

본문이 없는 순수 403 오류도 종종 동일한 원인을 가지지만, 기업 프록시나 방화벽이 다운로드를 차단하여 발생할 수도 있습니다. 지원되는 국가에 계시면서 403 오류가 계속 표시되면 대체 설치 프로그램을 시도하기 전에 [네트워크 연결 확인](#check-network-connectivity)을 진행하세요.

그 외에는 네트워크 문제, 지역 라우팅 문제 또는 일시적인 서비스 중단으로 인해 발생할 수 있습니다.

**해결 방법:**

1. **대체 설치 방법 사용**:

   macOS에서는 Homebrew를 통해 설치:

   ```bash theme={null}
   brew install --cask claude-code
   ```

   Windows에서는 WinGet을 통해 설치:

   ```powershell theme={null}
   winget install Anthropic.ClaudeCode
   ```

2. **몇 분 후 재시도**: 문제는 일시적인 경우가 많습니다. 잠시 기다린 후 원래 명령을 다시 시도해 보세요.

### 설치 후 `command not found: claude`

설치는 완료되었으나 `claude`가 작동하지 않습니다. 정확한 오류는 플랫폼에 따라 다릅니다:

| 플랫폼 | 오류 메시지 |
| :---------- | :--------------------------------------------------------------------- |
| macOS | `zsh: command not found: claude` |
| Linux | `bash: claude: command not found` |
| Windows CMD | `'claude' is not recognized as an internal or external command` |
| PowerShell | `claude : The term 'claude' is not recognized as the name of a cmdlet` |

이는 설치 디렉터리가 쉘의 검색 경로(PATH)에 포함되지 않았음을 의미합니다. 각 플랫폼별 해결 방법은 [PATH 확인](#verify-your-path)을 참조하세요.

### `curl: (56) Failure writing output to destination`

`curl ... | bash` 명령은 스크립트를 다운로드하고 이를 실행을 위해 Bash에 파이프로 전달합니다. 이 오류와 관련된 `curl: (23) Failure writing output to destination` 오류는 Bash가 완전한 스크립트를 수신하지 못했음을 의미합니다. 종료 코드 56은 다운로드 자체가 중단되었음을 나타내고 종료 코드 23은 curl이 수신한 내용을 파이프에 쓰지 못했음을 의미하며 보통 Bash가 일찍 종료되었기 때문입니다.

**해결 방법:**

1. **네트워크 안정성 확인**: Claude Code 바이너리는 `downloads.claude.ai`에 호스팅됩니다. 도달 가능한지 테스트하세요:

   ```bash theme={null}
   curl -sI https://downloads.claude.ai/claude-code-releases/latest
   ```

   `HTTP/2 200` 라인은 서버에 도달했음을 의미하며 원래의 실패는 간헐적이었을 가능성이 높으므로 설치 명령을 재시도하세요. 다른 결과는 원인을 가리킵니다:

   * `403`: 보통 프록시나 네트워크 필터가 호스트를 차단 중이거나, 사용자의 지역에서 Claude Code를 [사용할 수 없음](https://www.anthropic.com/supported-countries)을 의미합니다.
   * `5xx`: 보통 일시적인 서비스 문제이므로 몇 분 후 재시도하세요.
   * `Could not resolve host` 또는 연결 시간 초과: 네트워크가 다운로드를 차단하고 있습니다.

2. **대체 설치 방법 시도**:

   macOS:

   ```bash theme={null}
   brew install --cask claude-code
   ```

   Windows:

   ```powershell theme={null}
   winget install Anthropic.ClaudeCode
   ```

### Homebrew 카스크를 사용할 수 없거나 만료됨

Homebrew 카스크 인덱스의 로컬 복사본이 카스크 게시 이전의 것인 경우 Homebrew에 `Error: Cask 'claude-code' is unavailable: No Cask with this name exists` 오류가 보고됩니다. 인덱스를 새로 고치고 재시도하세요:

```bash theme={null}
brew update
brew install --cask claude-code
```

Homebrew가 예상보다 이전 버전의 Claude Code를 설치하는 경우에도 동일한 만료된 인덱스가 원인인 경우가 많습니다. `claude-code` 카스크는 안정 채널을 추적하며 일반적으로 최신 릴리스보다 약 1주일 뒤처집니다. 최신 버전을 구하려면 대신 `brew install --cask claude-code@latest`를 실행하세요. 두 카스크 간의 차이점은 [Configure release channel](/docs/en/setup#configure-release-channel)을 참조하세요.

### TLS 또는 SSL 연결 오류

`curl: (35) TLS connect error`, `schannel: next InitializeSecurityContext failed`, 또는 PowerShell의 `Could not establish trust relationship for the SSL/TLS secure channel`과 같은 오류는 TLS 핸드셰이크 실패를 나타냅니다.

**해결 방법:**

1. **시스템 CA 인증서 업데이트**:

   Ubuntu/Debian:

   ```bash theme={null}
   sudo apt-get update && sudo apt-get install ca-certificates
   ```

   macOS의 경우 시스템 curl이 Keychain 신뢰 저장소를 사용합니다. macOS 자체를 업데이트하면 루트 인증서가 업데이트됩니다.

2. **Windows에서 설치 프로그램 실행 전 PowerShell에서 TLS 1.2 활성화**:
   ```powershell theme={null}
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   irm https://claude.ai/install.ps1 | iex
   ```

3. **프록시 또는 방화벽 간섭 확인**: TLS 검사를 수행하는 기업 프록시는 `unable to get local issuer certificate` 및 `SELF_SIGNED_CERT_IN_CHAIN`을 포함하여 이러한 오류를 일으킬 수 있습니다. 설치 단계의 경우 `--cacert`로 기업 CA 번들을 curl에 지시하세요:
   ```bash theme={null}
   curl --cacert /path/to/corporate-ca.pem -fsSL https://claude.ai/install.sh | bash
   ```
   설치된 Claude Code의 경우 API 요청이 동일한 번들을 신뢰하도록 `NODE_EXTRA_CA_CERTS`를 설정하세요:
   ```bash theme={null}
   export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca.pem
   ```
   인증서 파일이 없는 경우 IT 팀에 문의하세요. 프록시가 원인인지 확인하기 위해 직접 연결을 통해 시도해 볼 수도 있습니다.

4. **Windows에서 네트워크가 철회 검사를 차단하는 경우 설치 프로그램 교체**. `CRYPT_E_NO_REVOCATION_CHECK (0x80092012)` 및 `CRYPT_E_REVOCATION_OFFLINE (0x80092013)` 오류는 curl이 서버에 도달했으나 네트워크가 인증서 철회 조회를 차단했음을 의미하며 기업 방화벽 뒤에서 일반적입니다. curl의 `--ssl-revoke-best-effort` 플래그를 추가하는 것으로는 해결되지 않습니다. 플래그는 `install.cmd` 자체의 다운로드에만 적용되고 스크립트 자체 다운로드는 해당 플래그 없이 실행되므로 동일한 오류로 설치에 실패합니다. 대신 차단된 조회를 허용하는 설치 방법을 사용하세요. PowerShell을 열고 .NET을 통해 다운로드되어 철회 서버에 도달할 수 없을 때 실패하지 않는 PowerShell 설치 프로그램을 실행하세요:
   ```powershell theme={null}
   irm https://claude.ai/install.ps1 | iex
   ```
   curl을 완전히 우회하는 `winget install Anthropic.ClaudeCode`로 설치할 수도 있습니다.

### `Failed to fetch version from downloads.claude.ai`

설치 프로그램이 다운로드 서버에 도달할 수 없었습니다. 이는 일반적으로 네트워크에서 `downloads.claude.ai`가 차단되었음을 의미합니다.

**해결 방법:**

1. **직접 연결 테스트**:

   ```bash theme={null}
   curl -sI https://downloads.claude.ai/claude-code-releases/latest
   ```

   `HTTP/2 200` 라인은 서버에 도달할 수 있음을 의미합니다. 다른 결과는 원인을 가리킵니다:

   * `403`: 보통 프록시나 네트워크 필터가 호스트를 차단 중이거나, 사용자의 지역에서 Claude Code를 [사용할 수 없음](https://www.anthropic.com/supported-countries)을 의미합니다.
   * `5xx`: 보통 일시적인 서비스 문제이므로 몇 분 후 재시도하세요.

2. **프록시 환경인 경우** 설치 프로그램이 이를 통해 라우팅할 수 있도록 `HTTPS_PROXY`를 설정하세요. 세부 정보는 [proxy configuration](/docs/en/network-config#proxy-configuration)을 참조하세요.
   ```bash theme={null}
   export HTTPS_PROXY=http://proxy.example.com:8080
   curl -fsSL https://claude.ai/install.sh | bash
   ```

3. **제한된 네트워크인 경우** 다른 네트워크나 VPN을 시도하거나 대체 설치 방법을 사용하세요:

   macOS:

   ```bash theme={null}
   brew install --cask claude-code
   ```

   Windows:

   ```powershell theme={null}
   winget install Anthropic.ClaudeCode
   ```

### Windows에서 잘못된 설치 명령 사용

`'irm' is not recognized`, `The token '&&' is not valid`, `A parameter cannot be found that matches parameter name 'fsSL'`, 또는 `'bash' is not recognized as the name of a cmdlet`이 보인다면 다른 쉘이나 운영체제용 설치 명령을 복사한 것입니다.

* **`irm` 인식되지 않음**: PowerShell이 아닌 CMD 환경입니다. 두 가지 옵션이 있습니다:

  시작 메뉴에서 "PowerShell"을 검색하여 PowerShell을 연 다음 원래 설치 명령을 실행합니다:

  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

  또는 CMD를 그대로 유지하고 CMD 설치 프로그램을 대신 사용합니다:

  ```batch theme={null}
  curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
  ```

* **`&&` 유효하지 않음**: PowerShell 환경이지만 CMD 설치 명령을 실행했습니다. PowerShell 설치 프로그램을 사용하세요:
  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

* **`A parameter cannot be found that matches parameter name 'fsSL'`**: Windows PowerShell에서 macOS/Linux용 `curl -fsSL ... | bash` 설치 프로그램을 실행했습니다. PowerShell에서 `curl`은 `Invoke-WebRequest`에 대한 별칭이며 `-fsSL` 플래그를 거부합니다. PowerShell 설치 프로그램을 대신 사용하세요:
  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

* **`bash` 인식되지 않음**: Windows에서 macOS/Linux 설치 프로그램을 실행했습니다. PowerShell 설치 프로그램을 대신 사용하세요:
  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

### Windows 설치 중 `The process cannot access the file`

PowerShell 설치 프로그램이 `Failed to download binary: The process cannot access the file ... because it is being used by another process` 오류와 함께 실패하는 경우 설치 프로그램이 `%USERPROFILE%\.claude\downloads`에 쓸 수 없는 상태입니다. 이는 일반적으로 이전 설치 시도가 여전히 실행 중이거나 백신 소프트웨어가 해당 폴더에 부분 다운로드된 바이너리를 스캔하고 있기 때문입니다.

설치 프로그램을 실행 중인 다른 PowerShell 창을 모두 닫고 백신 스캔이 파일 잠금을 해제할 때까지 기다리세요. 그런 다음 다운로드 폴더를 삭제하고 설치 프로그램을 다시 실행하세요:

```powershell theme={null}
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\downloads"
irm https://claude.ai/install.ps1 | iex
```

### 메모리가 적은 Linux 서버에서 설치 중 강제 종료(Killed)됨

설치 중 `Killed` 메시지는 일반적으로 시스템의 여유 메모리가 부족하여 Linux OOM(Out-Of-Memory) 킬러가 `claude install` 단계를 종료했음을 의미합니다. 이는 소형 VPS 및 클라우드 인스턴스에서 일반적입니다. 설치 스크립트는 원인을 보고하고 코드 137로 종료됩니다:

```text theme={null}
Setting up Claude Code...
bash: line 142: 34803 Killed    "$binary_path" install ${TARGET:+"$TARGET"}
Installation was killed before it could finish (exit code 137). This usually means the system ran out of memory.
Claude Code needs roughly 512MB of free memory to install. Free up memory, then run this script again.
```

v2.1.200 이전에는 스크립트가 설명 없이 쉘의 순수 `Killed` 라인만 출력하고 종료되었습니다.

설치 시 약 512MB의 여유 메모리가 필요하며 Claude Code를 실행할 때는 더 많이 필요합니다. [system requirements](/docs/en/setup#system-requirements)를 참조하세요.

**해결 방법:**

1. 서버의 RAM이 제한적인 경우 **스왑(swap) 공간 추가**. 스왑은 디스크 공간을 오버플로 메모리로 사용하여 물리적 RAM이 적더라도 설치를 완료할 수 있게 해줍니다.

   2GB 스왑 파일을 생성하고 활성화합니다:

   ```bash theme={null}
   sudo fallocate -l 2G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```

   그런 다음 설치를 재시도합니다:

   ```bash theme={null}
   curl -fsSL https://claude.ai/install.sh | bash
   ```

2. 설치 전에 **다른 프로세스를 닫아** 메모리를 확보합니다.

3. 가능한 경우 **더 큰 인스턴스 사용**. Claude Code는 최소 4GB의 RAM이 필요합니다.

### Docker에서 설치 멈춤

Docker 컨테이너 내에서 Claude Code를 설치할 때 root 사용자로 `/`에 설치하면 멈춤 현상이 발생할 수 있습니다.

**해결 방법:**

1. 설치 프로그램을 실행하기 전에 **작업 디렉터리 설정**. `/`에서 실행할 경우 설치 프로그램이 전체 파일 시스템을 스캔하여 과도한 메모리를 사용합니다. `WORKDIR`을 설정하면 스캔 범위가 소형 디렉터리로 제한됩니다:
   ```dockerfile theme={null}
   WORKDIR /tmp
   RUN curl -fsSL https://claude.ai/install.sh | bash
   ```

2. Docker Desktop을 사용하는 경우 **Docker 메모리 한도 증가**:
   ```bash theme={null}
   docker build --memory=4g .
   ```

### `claude update` 또는 `claude doctor` 멈춤

`claude update` 및 `claude doctor`는 만료된 `claude` 별칭을 찾기 위해 쉘 구성 파일(`~/.zshrc`, `~/.bashrc`, `~/.config/fish/config.fish`, 그리고 macOS의 경우 `~/.bash_profile`, `~/.bash_login`, `~/.profile` 중 존재하는 첫 번째 파일)을 스캔합니다. `ZDOTDIR`을 설정한 경우 Zsh 파일은 `$ZDOTDIR/.zshrc`가 됩니다. {/* min-version: 2.1.214 */} 해당 경로 중 하나가 디렉터리인 경우 Claude Code는 이를 건너뛰고 두 명령 모두 정상적으로 완료됩니다. v2.1.214 이전에는 해당 경로 중 하나에 디렉터리가 있으면 두 명령이 모두 멈추고 `/status` 항목의 시스템 진단 섹션이 빈 상태로 남아있었습니다. `claude doctor`는 출력 없이 멈췄고 `claude update`는 `Checking for updates`를 출력한 직후 멈췄습니다.

이전 버전에서 멈춤 현상이 발생했다면 해당 디렉터리를 찾으세요. 이 명령의 출력에서 `d`로 시작하는 라인은 해당 경로가 디렉터리임을 의미합니다. `No such file or directory` 라인은 해당 경로에 아무것도 존재하지 않음을 의미하므로 원인이 아닙니다:

```bash theme={null}
ls -ld ~/.zshrc ~/.bashrc ~/.bash_profile ~/.bash_login ~/.profile ~/.config/fish/config.fish
```

디렉터리를 다른 곳으로 옮기거나 v2.1.214 이상으로 업데이트하세요. 영향을 받는 버전에서는 `claude update`가 멈추므로 [install script](/docs/en/setup#install-claude-code)를 다시 실행하여 업데이트하세요.

### Windows에서 Claude Desktop이 `claude` 명령을 재정의함

이전 버전의 Claude Desktop을 설치한 경우 PATH에서 Claude Code CLI보다 우선하는 `Claude.exe`를 `WindowsApps` 디렉터리에 등록했을 수 있습니다. `claude`를 실행하면 CLI 대신 Desktop 앱이 열립니다.

이 문제를 해결하려면 Claude Desktop을 최신 버전으로 업데이트하세요.

### Windows에서 Claude Code를 사용하려면 Git for Windows (bash용) 또는 PowerShell이 필요함

Git for Windows는 선택 사항입니다. Claude Code는 Git Bash가 없을 때 [PowerShell tool](/docs/en/tools-reference#powershell-tool)을 사용하므로 이 오류는 두 쉘이 모두 발견되지 않았음을 의미합니다.

**PowerShell이 PATH에서 누락된 경우** 기본 위치는 `C:\Windows\System32\WindowsPowerShell\v1.0\`입니다. 해당 디렉터리를 `PATH`에 추가하거나 `pwsh`를 제공하는 [PowerShell 7](https://aka.ms/powershell)을 설치하세요.

**대신 Git for Windows를 설치하려는 경우** [git-scm.com/downloads/win](https://git-scm.com/downloads/win)에서 다운로드하세요. 설정 중에 "Add to PATH"를 선택합니다. 설치 후 터미널을 재시작하세요. 설치하면 Bash 기반 스크립트 및 도구로 작업할 때 유용한 Bash 도구가 활성화됩니다.

**Git이 이미 설치되어 있으나** Claude Code가 찾지 못하는 경우 [settings.json file](/docs/en/settings)에 경로를 설정하세요:

```json theme={null}
{
  "env": {
    "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
  }
}
```

Git이 다른 위치에 설치되어 있는 경우 PowerShell에서 `where.exe git`을 실행하여 경로를 찾고 해당 디렉터리의 `bin\bash.exe` 경로를 사용하세요.

**경로가 올바르고 파일이 존재함에도** Claude Code가 여전히 찾을 수 없다고 보고하는 경우 AppLocker, 그룹 정책 소프트웨어 제한 정책 또는 EDR 에이전트와 같은 엔드포인트 보안 소프트웨어가 간섭하고 있을 수 있습니다. v2.1.116 이전 버전에서 Claude Code는 경로를 확인하기 위해 자식 프로세스(`cmd.exe`)를 스폰하였으며 이러한 정책이 이를 차단할 수 있었습니다. 일반적인 신호는 PowerShell에서 직접 실행할 때 `cmd.exe /c dir "C:\Program Files\Git\bin\bash.exe"`가 작동하지만 `claude.exe`에서 시작할 때 자동으로 실패하는 것입니다.

Claude Code v2.1.116 이상은 파일 시스템을 직접 확인하므로 먼저 업데이트하세요. 현재 버전에서도 오류가 지속되면 IT 팀에 문의하여 엔드포인트 보호 정책에서 `claude.exe` 및 해당 프로세스가 스폰하는 프로세스(`cmd.exe` 및 `bash.exe` 포함)를 허용 목록에 추가하도록 요청하세요.

### Claude Code는 32비트 Windows를 지원하지 않음

Windows에는 시작 메뉴에 `Windows PowerShell`과 `Windows PowerShell (x86)`이라는 두 가지 PowerShell 항목이 포함되어 있습니다. x86 항목은 32비트 프로세스로 실행되어 64비트 머신에서도 이 오류를 트리거합니다. 어떤 경우인지 확인하려면 오류가 발생한 동일한 창에서 다음을 실행하세요:

```powershell theme={null}
[Environment]::Is64BitOperatingSystem
```

`True`가 출력되면 운영체제는 정상입니다. 창을 닫고 x86 접미사가 없는 `Windows PowerShell`을 연 다음 설치 명령을 다시 실행하세요.

`False`가 출력되면 32비트 버전의 Windows를 사용 중인 것입니다. Claude Code는 64비트 운영체제가 필요합니다. [system requirements](/docs/en/setup#system-requirements)를 참조하세요.

### Linux musl 또는 glibc 바이너리 불일치

설치 후 `libstdc++.so.6` 또는 `libgcc_s.so.1`과 같은 누락된 공유 라이브러리에 대한 오류가 표시되는 경우 설치 프로그램이 시스템에 맞지 않는 잘못된 바이너리 변형을 다운로드했을 수 있습니다.

```text theme={null}
Error loading shared library libstdc++.so.6: No such file or directory
```

이는 musl 교차 컴파일 패키지가 설치된 glibc 기반 시스템에서 발생하여 설치 프로그램이 시스템을 musl로 오인하도록 만들 수 있습니다.

**해결 방법:**

1. **시스템이 사용하는 libc 확인**:
   ```bash theme={null}
   ldd --version 2>&1 | head -1
   ```
   `GNU libc` 또는 `GLIBC`가 언급된 출력은 glibc를 의미합니다. `musl`이 언급된 출력은 musl을 의미합니다.

2. **glibc 환경이지만 musl 바이너리를 받은 경우** 설치를 제거하고 다시 설치하세요. `https://downloads.claude.ai/claude-code-releases/{VERSION}/manifest.json`에 있는 매니페스트를 사용하여 올바른 바이너리를 수동으로 다운로드할 수도 있습니다. `ldd --version` 및 `ls /lib/libc.musl*`의 출력과 함께 [GitHub issue](https://github.com/anthropics/claude-code/issues)를 등록하세요.

3. **Alpine Linux와 같이 실제로 musl 환경인 경우** 필요한 패키지를 설치하세요:
   ```bash theme={null}
   apk add libgcc libstdc++ ripgrep
   ```
   Alpine에서 `ripgrep`은 커뮤니티 리포지토리에 있습니다. `apk`에서 패키지가 누락되었다고 보고하는 경우 [Alpine Linux setup](/docs/en/setup#alpine-linux-and-musl-based-distributions)을 참조하세요.

### `Illegal instruction`

`claude` 또는 설치 프로그램을 실행할 때 `Illegal instruction`이 출력되는 경우 네이티브 바이너리가 프로세서에서 지원하지 않는 CPU 명령어를 사용하는 것입니다. 두 가지 서로 다른 원인이 있습니다.

**아키텍처 불일치.** 설치 프로그램이 잘못된 바이너리를 다운로드했습니다(예: ARM 서버에서 x86 바이너리 다운로드). macOS/Linux에서는 `uname -m`으로, PowerShell에서는 `$env:PROCESSOR_ARCHITECTURE`로 확인하세요. 결과가 받은 바이너리와 일치하지 않으면 출력 내용과 함께 [GitHub issue를 등록](https://github.com/anthropics/claude-code/issues)하세요.

**누락된 AVX 명령어 세트.** 아키텍처가 올바름에도 불구하고 `Illegal instruction`이 계속 보인다면 CPU에 AVX나 바이너리가 요구하는 다른 명령어 세트가 부족할 가능성이 높습니다. 이는 대략 2013년 이전의 Intel 및 AMD 프로세서와 하이퍼바이저가 AVX를 게스트에 전달하지 않는 가상 머신에 영향을 미칩니다.

VPS 또는 VM에서는 `grep -m1 -ow avx /proc/cpuinfo`를 실행하세요. 빈 결과는 게스트에서 AVX를 사용할 수 없음을 의미합니다.

네이티브 바이너리 해결 방법은 존재하지 않습니다. 제보 시 Linux의 `grep -m1 "model name" /proc/cpuinfo` 또는 macOS의 `sysctl -n machdep.cpu.brand_string`에서 확인한 CPU 모델을 포함하여 상태 추적을 위해 [issue #50384](https://github.com/anthropics/claude-code/issues/50384)를 확인하세요.

대체 설치 방법도 동일한 네이티브 바이너리를 다운로드하므로 어느 원인이든 해결되지 않습니다.

### macOS에서 `dyld: cannot load`

설치 중 `dyld: cannot load`, `dyld: Symbol not found`, 또는 `Abort trap: 6`이 보인다면 바이너리가 macOS 버전 또는 하드웨어와 호환되지 않는 것입니다.

```text theme={null}
dyld: cannot load 'claude-2.1.42-darwin-x64' (load command 0x80000034 is unknown)
Abort trap: 6
```

`libicucore`를 참조하는 `Symbol not found` 오류도 macOS 버전이 바이너리가 지원하는 버전보다 이전 버전임을 나타냅니다:

```text theme={null}
dyld: Symbol not found: _ubrk_clone
  Referenced from: claude-darwin-x64 (which was built for Mac OS X 13.0)
  Expected in: /usr/lib/libicucore.A.dylib
```

**해결 방법:**

1. **macOS 버전 확인**: Claude Code는 macOS 13.0 이상이 필요합니다. Apple 메뉴를 열고 이 Mac에 관하여를 선택하여 버전을 확인하세요.

2. **이전 버전인 경우 macOS 업데이트**. 바이너리는 이전 macOS 버전에서 지원하지 않는 로드 명령과 시스템 라이브러리를 사용합니다. Homebrew와 같은 대체 설치 방법도 동일한 바이너리를 다운로드하므로 이 오류가 해결되지 않습니다.

### WSL1에서 `Exec format error`

WSL에서 `claude`를 실행할 때 `cannot execute binary file: Exec format error`가 출력되는 경우 WSL1을 사용 중이며 [issue #38788](https://github.com/anthropics/claude-code/issues/38788)에서 추적 중인 알려진 네이티브 바이너리 회귀 문제에 부딪힌 것입니다. 바이너리의 프로그램 헤더가 WSL1의 로더가 처리할 수 없는 방식으로 변경되었습니다.

가장 깔끔한 해결책은 PowerShell에서 배포판을 WSL2로 변환하는 것입니다:

```powershell theme={null}
wsl --set-version <DistroName> 2
```

WSL1을 유지해야 하는 경우 동적 링커를 통해 바이너리를 호출하세요. WSL 내부의 `~/.bashrc`에 홈 디렉터리가 다른 경우 경로를 대체하여 다음 함수를 추가하세요:

```bash theme={null}
claude() {
  /lib64/ld-linux-x86-64.so.2 "$(readlink -f "$HOME/.local/bin/claude")" "$@"
}
```

그런 다음 `source ~/.bashrc`를 실행하고 `claude`를 재시도하세요.

### WSL에서의 npm install 오류

이러한 문제는 WSL 내부에서 `npm install -g`로 Claude Code를 설치한 경우 적용됩니다. [native installer](/docs/en/setup)를 사용한 경우 이 섹션을 건너뛰세요.

**OS 또는 플랫폼 감지 문제.** 설치 중 npm이 플랫폼 불일치를 보고하는 경우 WSL이 Windows `npm`을 집어 들고 있을 가능성이 높습니다. 먼저 `npm config set os linux`를 실행한 다음 `npm install -g @anthropic-ai/claude-code --force`로 설치하세요. `sudo`를 사용하지 마세요.

**`claude` 실행 시 `exec: node: not found`.** WSL 환경이 Node.js의 Windows 설치본을 사용하고 있을 가능성이 높습니다. `which npm` 및 `which node`로 확인하세요. `/mnt/c/`로 시작하는 경로는 Windows 바이너리이며 Linux 경로는 `/usr/`로 시작합니다. 이를 수정하려면 Linux 배포판의 패키지 관리자 또는 [`nvm`](https://github.com/nvm-sh/nvm)을 통해 Node를 설치하세요.

**nvm 버전 충돌.** WSL과 Windows 모두에 nvm이 설치된 경우 WSL은 기본적으로 Windows PATH를 가져오고 Windows nvm이 우선하므로 WSL에서 Node 버전을 전환하면 오류가 발생할 수 있습니다. 가장 일반적인 원인은 쉘에 nvm이 로드되지 않았기 때문입니다. `~/.bashrc` 또는 `~/.zshrc`에 nvm 로더를 추가하세요:

```bash theme={null}
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

또는 현재 세션에서 로드하세요:

```bash theme={null}
source ~/.nvm/nvm.sh
```

nvm이 로드되었으나 여전히 Windows 경로가 우선하는 경우 Linux Node 경로를 명시적으로 앞에 추가하세요:

```bash theme={null}
export PATH="$HOME/.nvm/versions/node/$(node -v)/bin:$PATH"
```

<Warning>
  `appendWindowsPath = false`를 통해 Windows PATH 가져오기를 비활성화하지 마세요. WSL에서 Windows 실행 파일을 호출하는 기능이 깨집니다. 마찬가지로 Windows 개발용으로 사용하는 경우 Windows에서 Node.js를 제거하지 마세요.
</Warning>

### 설치 중 권한 오류

네이티브 설치 프로그램이 권한 오류로 실패하는 경우 대상 디렉터리에 쓰기 권한이 없을 수 있습니다. [Check directory permissions](#check-directory-permissions)를 참조하세요.

이전에 npm으로 설치했고 npm 전용 권한 오류가 발생하는 경우 네이티브 설치 프로그램으로 전환하세요:

```bash theme={null}
curl -fsSL https://claude.ai/install.sh | bash
```

### npm install 후 네이티브 바이너리를 찾을 수 없음

`@anthropic-ai/claude-code` npm 패키지는 `@anthropic-ai/claude-code-darwin-arm64`와 같은 플랫폼별 옵션 종속성을 통해 네이티브 바이너리를 가져옵니다. 설치 후 `claude` 실행 시 `Could not find native binary package "@anthropic-ai/claude-code-<platform>"`이 출력되면 다음 원인을 확인하세요:

* **옵션 종속성(Optional dependencies)이 비활성화됨.** npm install 명령에서 `--omit=optional`, pnpm에서 `--no-optional`, 또는 yarn에서 `--ignore-optional`을 제거하고 `.npmrc`에 `optional=false`가 설정되어 있지 않은지 확인한 후 다시 설치하세요. 네이티브 바이너리는 옵션 종속성으로만 전달되므로 건너뛰면 JavaScript 폴백이 없습니다.
* **지원되지 않는 플랫폼.** 사전 빌드된 바이너리는 `darwin-arm64`, `darwin-x64`, `linux-x64`, `linux-arm64`, `linux-x64-musl`, `linux-arm64-musl`, `win32-x64`, 및 `win32-arm64`용으로 게시됩니다. Claude Code는 다른 플랫폼용 바이너리를 제공하지 않으므로 [system requirements](/docs/en/setup#system-requirements)를 참조하세요. {/* min-version: 2.1.205 */} FreeBSD에서 설치 프로그램은 플랫폼을 지원되지 않음으로 보고합니다. v2.1.205 이전에는 FreeBSD를 Linux로 취급하여 실행할 수 없는 바이너리를 다운로드했습니다.
* **기업 npm 미러에 플랫폼 패키지가 누락됨.** 레지스트리가 메타 패키지 외에도 8개의 `@anthropic-ai/claude-code-*` 플랫폼 패키지를 모두 미러링하는지 확인하세요.

`--ignore-scripts`로 설치하면 이 오류가 발생하지 않습니다. 바이너리를 연결하는 포스트인스톨 단계가 건너뛰어지므로 Claude Code는 시작 시 플랫폼 바이너리를 찾아 스폰하는 래퍼로 대체됩니다. 작동은 하지만 시작 속도가 느려지므로 직접 실행을 위해 스크립트가 활성화된 상태로 다시 설치하세요.

## 로그인 및 인증

이 섹션에서는 로그인 실패, OAuth 오류 및 토큰 문제를 다룹니다.

### 로그인 재설정

로그인에 실패하고 원인이 분명하지 않은 경우 깨끗한 재인증으로 대부분의 사례가 해결됩니다:

1. 완전히 로그아웃하려면 `/logout` 실행
2. Claude Code 닫기
3. `claude`로 다시 시작하고 인증 프로세스를 다시 완료

로그인 중 브라우저가 자동으로 열리지 않는 경우 `c`를 눌러 OAuth URL을 클립보드에 복사한 후 브라우저에 수동으로 붙여넣으세요. 이는 좁은 터미널이나 SSH 터미널에서 URL이 여러 줄로 줄바꿈되어 직접 클릭할 수 없는 경우에도 작동합니다.

### OAuth 오류: Invalid code

`OAuth error: Invalid code. Please make sure the full code was copied`가 보인다면 로그인 코드가 만료되었거나 복사-붙여넣기 중 잘린 것입니다.

**해결 방법:**

* Enter를 눌러 재시도하고 브라우저가 열린 후 로그인을 빠르게 완료하세요.
* 브라우저가 자동으로 열리지 않는 경우 `c`를 입력하여 전체 URL을 복사하세요.
* 원격/SSH 세션을 사용하는 경우 브라우저가 다른 머신에서 열릴 수 있습니다. 터미널에 표시된 URL을 복사하여 로컬 브라우저에서 대신 여세요.

### 로그인 후 403 Forbidden

로그인 후 `API Error: 403 {"error":{"type":"forbidden","message":"Request not allowed"}}`가 보인다면:

* **Claude Pro/Max 사용자**: [claude.ai/settings](https://claude.ai/settings)에서 구독이 활성 상태인지 확인하세요.
* **Anthropic Console 사용자**: 계정에 "Claude Code" 또는 "Developer" 역할이 있는지 확인하세요. 관리자는 Settings → Members의 Anthropic Console에서 이를 할당합니다.
* **프록시 환경**: 기업 프록시가 API 요청을 방해할 수 있습니다. 프록시 설정은 [network configuration](/docs/en/network-config)을 참조하세요.

### 활성 구독이 있으나 조직이 비활성화됨

활성 Claude 구독이 있음에도 불구하고 `API Error: 400 ... "This organization has been disabled"`가 보인다면 `ANTHROPIC_API_KEY` 환경 변수가 구독을 재정의하고 있는 것입니다. 이는 이전 고용주나 프로젝트의 이전 API 키가 쉘 프로필에 여전히 설정되어 있을 때 일반적으로 발생합니다.

`ANTHROPIC_API_KEY`가 존재하고 이를 승인한 경우 Claude Code는 구독의 OAuth 자격 증명 대신 해당 키를 사용합니다. `-p` 플래그가 있는 비대화형 모드에서는 키가 존재하는 경우 항상 사용됩니다. 전체 해석 순서는 [authentication precedence](/docs/en/authentication#authentication-precedence)를 참조하세요.

대신 구독을 사용하려면 환경 변수를 해제하고 쉘 프로필에서 제거하세요:

```bash theme={null}
unset ANTHROPIC_API_KEY
claude
```

`~/.zshrc`, `~/.bashrc`, 또는 `~/.profile`에서 `export ANTHROPIC_API_KEY=...` 라인을 확인하고 제거하여 변경 사항을 영구적으로 만드세요. Windows에서는 `$PROFILE`의 PowerShell 프로필과 사용자 환경 변수에서 `ANTHROPIC_API_KEY`를 확인하세요. Claude Code 내부에서 `/status`를 실행하여 어떤 인증 방법이 활성화되어 있는지 확인하세요.

### WSL2, SSH 또는 컨테이너에서 OAuth 로그인 실패

Claude Code가 WSL2, SSH를 통한 원격 머신 또는 컨테이너 내부에서 실행될 때 브라우저는 일반적으로 다른 호스트에서 열리며 리디렉션이 Claude Code의 로컬 콜백 서버에 도달할 수 없습니다. 로그인한 후 브라우저는 자동으로 다시 리디렉션하는 대신 로그인 코드를 표시합니다. 로그인을 완료하려면 `Paste code here if prompted` 프롬프트의 터미널에 해당 코드를 붙여넣으세요.

WSL2에서 브라우저가 전혀 열리지 않는 경우 `BROWSER` 환경 변수를 Windows 브라우저 경로로 설정하세요:

```bash theme={null}
export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"
claude
```

또는 대화형 로그인 프롬프트에서 `c`를 눌러 OAuth URL을 복사하거나 `claude auth login`이 출력하는 URL을 복사하여 로컬 머신의 브라우저에서 여세요.

대화형 프롬프트에 코드를 붙여넣어도 아무런 반응이 없다면 터미널의 붙여넣기 바인딩이 입력 필드에 도달하지 못하고 있을 가능성이 높습니다. 터미널의 대체 붙여넣기 단축키(Windows Terminal의 경우 우클릭 또는 Shift+Insert)를 시도하거나 표준 입력에서 붙여넣은 코드를 읽는 `claude auth login`을 대신 사용하세요:

```bash theme={null}
claude auth login
```

이 폴백은 네이티브 Windows 또는 대화형 프롬프트로의 붙여넣기가 실패하는 모든 터미널에도 적용됩니다.

### 로그인되지 않았거나 토큰이 만료됨

세션 후 Claude Code가 다시 로그인하도록 요청하는 경우 OAuth 토큰이 만료되었을 수 있습니다.

다시 인증하려면 `/login`을 실행하세요. 이 현상이 자주 발생하는 경우 토큰 검증이 올바른 타임스탬프에 의존하므로 시스템 시계가 정확한지 확인하세요.

한 머신의 병렬 세션은 저장된 로그인을 공유하며 한 번에 하나의 프로세스만 토큰을 새로 고치도록 갱신을 조정합니다. {/* min-version: 2.1.211 */} v2.1.211 이전에는 머신이 절전 모드에서 깨어날 때 두 세션이 동일한 토큰으로 갱신되어 저장된 로그인이 해지되고 열린 모든 세션이 한 번에 다시 로그인하도록 요청할 수 있었습니다.

macOS에서는 Keychain이 잠겨 있거나 암호가 계정 암호와 동기화되지 않아 Claude Code가 자격 증명을 저장하지 못할 때 로그인이 실패할 수도 있습니다. `claude doctor`를 실행하여 Keychain 접근을 확인하세요. Keychain을 수동으로 잠금 해제하려면 `security unlock-keychain ~/Library/Keychains/login.keychain-db`를 실행하세요. 잠금 해제가 도움이 되지 않는 경우 Keychain Access를 열고 `login` keychain을 선택한 다음 Edit > Change Password for Keychain "login"을 선택하여 계정 암호와 다시 동기화하세요.

### Bedrock, Agent Platform 또는 Foundry 자격 증명이 로드되지 않음

클라우드 제공업체를 사용하도록 Claude Code를 구성했고 Amazon Bedrock에서 `Could not load credentials from any providers`, Google Cloud's Agent Platform에서 `Could not load the default credentials`, 또는 Microsoft Foundry에서 `ChainedTokenCredential authentication failed`가 나타나는 경우 클라우드 제공업체 CLI가 현재 쉘에서 인증되지 않았을 가능성이 높습니다.

Amazon Bedrock의 경우 AWS 자격 증명이 유효한지 확인하세요:

```bash theme={null}
aws sts get-caller-identity
```

Google Cloud's Agent Platform의 경우 쉘에 `ANTHROPIC_VERTEX_PROJECT_ID` 및 `CLOUD_ML_REGION`이 설정되어 있는지 확인한 다음 애플리케이션 기본 자격 증명을 설정하세요:

```bash theme={null}
gcloud auth application-default login
```

Microsoft Foundry의 경우 `ANTHROPIC_FOUNDRY_API_KEY`가 설정되어 있는지 확인하거나 기본 자격 증명 체인이 계정을 찾을 수 있도록 Azure CLI로 로그인하세요:

```bash theme={null}
az login
```

자격 증명이 터미널에서는 작동하지만 VS Code 또는 JetBrains 확장 프로그램에서는 작동하지 않는 경우 IDE 프로세스가 쉘 환경을 상속받지 못했을 가능성이 높습니다. IDE 자체 설정에서 제공업체 환경 변수를 설정하거나 이미 내보낸 터미널에서 IDE를 실행하세요.

전체 제공업체 설정은 [Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](/docs/en/google-vertex-ai), 또는 [Microsoft Foundry](/docs/en/microsoft-foundry)를 참조하세요.

## 여전히 문제가 해결되지 않는 경우

위의 방법으로 문제가 해결되지 않는 경우:

1. [GitHub repository](https://github.com/anthropics/claude-code/issues)에서 알려진 문제를 확인하거나 사용 중인 운영체제, 실행한 설치 명령 및 전체 오류 출력과 함께 새 이슈를 생성하세요.
2. `claude --version`은 작동하지만 다른 문제가 발생하는 경우 자동 진단 보고서를 위해 `claude doctor`를 실행하세요.
3. 세션을 시작할 수 있는 경우 Claude Code 내부의 `/feedback`을 사용하여 문제를 보고하세요.
4. 로그인 루프, 인식되지 않는 구독 또는 비활성화된 조직과 같이 설치보다 계정에 문제가 있는 경우 Anthropic 지원팀에 문의하세요: [claude.ai](https://claude.ai) (Console 사용자는 [platform.claude.com](https://platform.claude.com))에 로그인하고 좌측 하단의 이니셜을 클릭한 후 **Get help**를 선택하세요. 전체 절차는 [How to get support](https://support.claude.com/en/articles/9015913-how-to-get-support)를 참조하세요.
