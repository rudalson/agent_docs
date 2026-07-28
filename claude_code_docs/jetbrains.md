> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# JetBrains IDEs

> IntelliJ, PyCharm, WebStorm 등을 포함한 JetBrains IDE에서 Claude Code 사용하기

Claude Code는 전용 플러그인을 통해 JetBrains IDE와 연동되어 대화형 차이점(diff) 보기, 선택 항목 컨텍스트 공유 등의 기능을 제공합니다.

## 지원되는 IDE

Claude Code 플러그인은 다음을 포함한 대부분의 JetBrains IDE에서 작동합니다:

* IntelliJ IDEA
* PyCharm
* Android Studio
* WebStorm
* PhpStorm
* GoLand

## 기능

* **빠른 실행**: `Cmd+Esc` (Mac) 또는 `Ctrl+Esc` (Windows/Linux)를 사용하여 편집기에서 직접 Claude Code를 열거나 UI의 Claude Code 버튼을 클릭합니다.
* **차이점(Diff) 보기**: 코드 변경 사항이 터미널 대신 IDE diff 뷰어에 직접 표시될 수 있습니다.
* **선택 항목 컨텍스트**: IDE의 현재 선택 영역 또는 탭이 Claude Code와 자동으로 공유됩니다. [`Read` 거부 규칙](/docs/en/permissions#read-and-edit)은 일치하는 파일에 대해 이러한 공유를 차단합니다.
* **파일 참조 단축키**: `Cmd+Option+K` (Mac) 또는 `Alt+Ctrl+K` (Linux/Windows)를 사용하여 `@src/auth.ts#L1-99`와 같은 파일 참조를 삽입합니다.
* **진단 공유**: 린트 및 구문 오류와 같은 IDE의 진단 오류가 작업 시 Claude와 자동으로 공유됩니다.

## 설치

플러그인은 IDE의 통합 터미널에서 `claude` 명령을 실행하고 이에 연결합니다. CLI의 가체 사본을 번들로 제공하지 않으므로 두 구성 요소를 모두 설치하세요:

<Steps>
  <Step title="Claude Code CLI 설치">
    아직 설치하지 않았다면 [빠른 시작](/docs/en/quickstart)에 따라 CLI를 설치하세요. `claude`가 PATH에 없으면 플러그인에 "Cannot launch Claude Code" 알림이 표시됩니다.
  </Step>

  <Step title="JetBrains 플러그인 설치">
    JetBrains Marketplace에서 [Claude Code 플러그인](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-)을 설치하고 IDE를 다시 시작합니다.
  </Step>
</Steps>

IDE가 찾을 수 없는 위치에 `claude`가 설치되어 있는 경우, 플러그인의 [Claude 명령 설정](#general-settings)에서 전체 경로를 설정하세요.

Claude Code는 모든 유료 Claude 구독(Pro, Max, Team 또는 Enterprise) 또는 Claude Console 계정과 작동하며 API 키가 필요하지 않습니다. `claude`를 처음 실행할 때 [로그인](/docs/en/authentication#log-in-to-claude-code)하라는 메시지가 표시됩니다.

<Note>
  플러그인을 설치한 후 적용되려면 IDE를 완전히 다시 시작해야 할 수 있습니다.
</Note>

## 사용법

### IDE 내부에서

IDE의 통합 터미널에서 `claude`를 실행하면 모든 연동 기능이 활성화됩니다.

### 외부 터미널에서

외부 터미널에서 `/ide` 명령을 사용하여 Claude Code를 JetBrains IDE에 연결하고 모든 기능을 활성화하세요:

```bash theme={null}
claude
```

```text theme={null}
/ide
```

연결이 성공하면 Claude Code는 `Connected to IntelliJ IDEA.`와 같은 메시지로 확인합니다. Claude Code가 플러그인이 설치되지 않은 실행 중인 IDE를 감지하면 `/ide`가 플러그인을 설치하고 IDE를 다시 시작하도록 요청합니다.

Claude가 IDE 프로젝트 루트와 동일한 파일에 접근하도록 하려면 IDE 프로젝트 루트와 동일한 디렉토리에서 Claude Code를 시작하세요.

## 구성

### Claude Code 설정

Claude Code의 설정을 통해 IDE 연동을 구성하세요:

1. `claude` 실행
2. `/config` 명령 입력
3. IDE에 차이점을 보여주려면 **Diff tool**을 `auto`로 설정하고, 터미널에 유지하려면 `terminal`로 설정

**Diff tool** 항목은 Claude Code가 IDE에 연결되어 있을 때만 `/config`에 나타나므로 JetBrains 터미널에서 `claude`를 실행하거나 외부 터미널에서 먼저 [`/ide`](/docs/en/commands)를 실행하세요. 기본 설정은 [`diffTool`](/docs/en/settings#global-config-settings)을 참조하세요.

### 플러그인 설정

**Settings → Tools → Claude Code \[Beta]**로 이동하여 Claude Code 플러그인을 구성하세요:

#### 일반 설정

* **Claude command**: Claude를 실행할 사용자 지정 명령 지정 (예: `claude`, `/usr/local/bin/claude` 또는 `npx @anthropic-ai/claude-code`)
* **Suppress notification for Claude command not found**: Claude 명령을 찾을 수 없다는 알림 건너뛰기
* **Enable using Option+Enter for multi-line prompts**: macOS 전용. 활성화되면 Option+Enter가 Claude Code 프롬프트에 줄바꿈을 삽입합니다. Option 키가 예기치 않게 캡처되는 경우 비활성화하세요. 터미널 재시작이 필요합니다.
* **Enable automatic updates**: 플러그인 업데이트를 자동으로 확인하고 설치하며, 재시작 시 적용됩니다.

<Tip>
  WSL 사용자의 경우: `wsl -d Ubuntu -- bash -lic "claude"`를 Claude 명령으로 설정하세요 (`Ubuntu`를 사용 중인 WSL 배포판 이름으로 교체)
</Tip>

#### ESC 키 구성

JetBrains 터미널에서 ESC 키가 Claude Code 작업을 중단하지 않는 경우:

1. **Settings → Tools → Terminal**로 이동합니다.
2. 다음 중 하나를 수행합니다:
   * "Move focus to the editor with Escape" 체크 해제, 또는
   * "Configure terminal keybindings"를 클릭하고 "Switch focus to Editor" 단축키 삭제
3. 변경 사항 적용

이렇게 하면 ESC 키가 Claude Code 작업을 올바르게 중단할 수 있습니다.

## 특별 구성

### 원격 개발 (Remote development)

<Warning>
  JetBrains Remote Development를 사용할 때는 로컬 클라이언트 머신이 아닌 **Settings → Plugin (Host)**를 통해 원격 호스트에 플러그인을 설치해야 합니다.
</Warning>

### WSL 구성

WSL2에서 JetBrains IDE와 함께 Claude Code를 사용하고 "No available IDEs detected"가 표시되는 경우, 원인은 보통 WSL2의 NAT 네트워킹이나 Windows 방화벽이 WSL2와 Windows 호스트에서 실행 중인 IDE 간의 연결을 차단하기 때문입니다. WSL1은 호스트의 네트워크를 직접 사용하므로 영향받지 않습니다.

#### Windows 방화벽을 통해 WSL2 트래픽 허용

기존 WSL2 네트워킹 모드를 유지할 수 있으므로 권장되는 해결 방법입니다.

<Steps>
  <Step title="WSL2 IP 주소 찾기">
    WSL 셸 내부에서 다음을 실행합니다:

    ```bash theme={null}
    hostname -I
    ```

    서브넷을 확인하세요: 주소의 처음 두 세그먼트를 가져와 뒤에 `.0.0/16`을 붙입니다. 예를 들어 주소가 `172.21.123.45`이면 서브넷은 `172.21.0.0/16`입니다.
  </Step>

  <Step title="방화벽 규칙 생성">
    관리자 권한으로 PowerShell을 열고 사용자의 서브넷에 맞게 IP 범위를 조정하여 다음을 실행합니다:

    ```powershell theme={null}
    New-NetFirewallRule -DisplayName "Allow WSL2 Internal Traffic" -Direction Inbound -Protocol TCP -Action Allow -RemoteAddress 172.21.0.0/16 -LocalAddress 172.21.0.0/16
    ```
  </Step>

  <Step title="IDE 및 Claude Code 다시 시작">
    새 규칙이 적용되도록 둘 다 닫았다가 다시 엽니다.
  </Step>
</Steps>

#### WSL2를 미러링된 네트워킹(mirrored networking)으로 전환

미러링된 네트워킹은 Windows 11 22H2 이상이 필요합니다. Windows 10을 사용하는 경우 위의 방화벽 규칙을 대신 사용하세요.

Windows 사용자 디렉토리의 `.wslconfig`에 다음을 추가합니다:

```ini theme={null}
[wsl2]
networkingMode=mirrored
```

그런 다음 PowerShell에서 `wsl --shutdown`으로 WSL을 다시 시작하세요.

## 문제 해결

### 플러그인이 작동하지 않음

플러그인이 설치되어 있지만 IDE에 Claude Code 기능이 나타나지 않는 경우:

* 프로젝트 루트 디렉토리에서 Claude Code를 실행하고 있는지 확인하세요.
* IDE 설정에서 JetBrains 플러그인이 활성화되어 있는지 확인하세요.
* IDE를 완전히 다시 시작하세요 (여러 번 수행해야 할 수 있음).
* Remote Development의 경우 원격 호스트에 플러그인이 설치되어 있는지 확인하세요.

### IDE가 감지되지 않음

`/ide` 명령에 "No available IDEs detected"가 표시되는 경우:

* 플러그인이 설치되어 있고 활성화되어 있는지 확인하세요.
* IDE를 완전히 다시 시작하세요.
* `/ide`를 실행하지 않고 자동 연결을 기대한 경우 IDE의 통합 터미널에서 `claude`를 실행했는지 확인하세요.
* WSL 사용자의 경우 위의 [WSL 구성](#wsl-configuration)을 참조하세요.

### Command not found

Claude 아이콘을 클릭했을 때 "command not found"가 표시되는 경우:

1. 터미널에서 `claude --version`을 실행하여 Claude Code가 설치되어 있는지 확인합니다.
2. 플러그인 설정에서 Claude 명령 경로를 구성합니다.
3. WSL 사용자의 경우 구성 섹션에 언급된 WSL 명령 형식을 사용합니다.

## 보안 고려사항

Claude Code가 [`acceptEdits` 권한 모드](/docs/en/permission-modes#auto-approve-file-edits-with-acceptedits-mode)에서 JetBrains IDE로 실행 중일 때, IDE가 자동으로 실행할 수 있는 IDE 구성 파일을 수정할 수 있습니다. 이는 `acceptEdits` 모드에서 Claude Code를 실행할 때의 위험을 증가시키고 bash 실행에 대한 Claude Code의 권한 프롬프트를 우회할 수 있습니다.

JetBrains IDE에서 실행할 때는 다음을 고려하세요:

* 편집 시 수동 승인 모드 사용
* 신뢰할 수 있는 프롬프트에만 Claude가 사용되도록 각별히 주의
* Claude Code가 수정 권한을 가진 파일을 인지

IDE 외부의 Claude Code 설치 또는 로그인 문제는 [설치 및 로그인 문제 해결](/docs/en/troubleshoot-install)을 참조하세요.

### 내장 IDE MCP 서버

플러그인이 활성화되어 있으면 CLI가 자동으로 연결하는 로컬 MCP 서버가 실행됩니다. CLI가 IDE의 네이티브 diff 뷰어에서 차이점을 열고, `@`-멘션을 위해 현재 선택 항목을 읽고, 검사 진단을 대화로 가져오는 방식이 바로 이 때문입니다.

서버 이름은 `ide`이며 구성할 내용이 없으므로 `/mcp`에서 숨겨집니다. 조직에서 MCP 도구를 허용 목록에 올리기 위해 [`PreToolUse` 훅](/docs/en/hooks#pretooluse)을 사용하는 경우에는 이 존재를 알아두어야 합니다.

**선택 항목 및 열린 파일 컨텍스트.** 연결되어 있는 동안 CLI는 프롬프트를 보낼 때마다 현재 편집기 선택 항목과 활성 파일 경로를 컨텍스트로 포함합니다. 이 상황이 발생하면 트랜스크립트에 `⧉ Selected N lines from <file>` 행이 표시됩니다. `.env`와 같은 민감한 파일을 제외하려면 해당 경로에 대한 [`Read` 거부 규칙](/docs/en/permissions#read-and-edit)을 추가하세요. 일치하는 거부 규칙은 선택한 텍스트와 해당 파일의 열린 파일 알림이 Claude에게 전달되는 것을 방지합니다.

**전송 및 인증.** 서버는 OS가 할당한 임시 포트에서 수신 대기하며 포트는 구성할 수 없습니다. 전송 방식은 암호화되지 않은 `ws://`입니다. 루프백에서는 트래픽을 캡처할 수 있는 모든 프로세스가 잠금 파일에서 토큰을 읽을 수도 있으므로 TLS가 로컬 공격자에 대한 추가 보호를 제공하지 않습니다. IDE가 시작될 때마다 무작위 인증 토큰이 생성되어 `~/.claude/ide/<port>.lock`의 잠금 파일에 기록되며, CLI가 연결하려면 `X-Claude-Code-Ide-Authorization` 헤더로 이를 제시해야 합니다. `CLAUDE_CONFIG_DIR`이 설정되어 있으면 잠금 파일이 `$CLAUDE_CONFIG_DIR/ide/`에 작성됩니다.

**모델에 노출되는 도구.** 서버는 여러 도구를 호스팅하지만 하나만 모델에 표시됩니다. 나머지는 차이점 열기 및 선택 영역 읽기와 같이 CLI가 자체 UI에 사용하는 내부 RPC이며, 도구 목록이 Claude에게 도달하기 전에 필터링됩니다.

| 도구 이름 (훅에서 보이는 이름) | 역할                                                                                                          | 읽기 전용 여부 |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------- | --------- |
| `mcp__ide__getDiagnostics`   | IDE의 검사 진단(편집기에 표시되는 오류 및 경고)을 반환합니다. 선택적으로 한 파일로 범위를 지정할 수 있습니다. | 예        |

JetBrains 플러그인은 모델에 코드 실행 도구를 노출하지 않습니다.

**수신 인터페이스.** 서버가 바인딩하는 네트워크 인터페이스는 **Settings → Tools → Claude Code \[Beta] → Networking (Advanced)** 아래의 **Accept connections from all network interfaces**에 의해 제어됩니다. 이 설정이 비활성화되면 서버는 `127.0.0.1`에서만 수신 대기하며 다른 호스트에서는 연결할 수 없습니다. 활성화되면 로컬 네트워크에서 포트에 접근할 수 있습니다. 이 설정은 기본 NAT 네트워킹을 사용하는 WSL2나 원격 IDE 설정과 같이 CLI가 루프백을 통해 IDE에 연결할 수 없는 경우를 위해 존재합니다. 해당 시나리오는 [WSL 구성](#wsl-configuration)을 참조하세요.

<Warning>
  **Accept connections from all network interfaces**를 활성화하면 로컬 네트워크에서 IDE MCP 포트에 접근할 수 있게 됩니다. 연결 시 여전히 잠금 파일의 인증 토큰이 필요하지만, 전송 방식이 암호화되지 않은 `ws://`이므로 설정이 켜져 있으면 세션 트래픽과 토큰이 네트워크를 평문으로 건너갑니다. 루프백이 정말 작동할 수 없는 경우에만 켜세요. WSL2의 경우 Windows 루프백 인터페이스가 Linux VM과 공유되어 소켓이 루프백에 유지될 수 있도록 [미러링된 네트워킹으로 전환](#switch-wsl2-to-mirrored-networking)하는 것을 권장합니다.
</Warning>
