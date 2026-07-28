---
slug: cli-troubleshooting
section: Antigravity CLI
title: 문제 해결
path:
  - Antigravity CLI
  - 문제 해결
---

# 문제 해결

설치 PATH, 로컬 자동 업데이트 잠금, 키링 액세스 권한 및 SSH 클립보드 포워딩과 관련된 일반적인 문제를 진단하고 해결합니다.

## 빠른 참조

아래의 조회 테이블을 스캔하여 증상을 식별하고 즉각적인 해결책을 찾으십시오.

| 오류 증상 | 잠재적 원인 | 대상 해결책 |
| :------------------------------------ | :---------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| **`agy: command not found`** | 쉘 환경에서 바이너리 디렉터리가 누락되었습니다. | [쉘 PATH 구성](#configure-your-shell-path) |
| **`keyring: secure lock out`** | 시스템 서비스 권한 누락 또는 활성 잠금입니다. | [키링 권한 승인](#authorize-keyring-permissions) |
| **`SSH Clipboard paste failures`** | 프로토콜 스트림이 차단되었거나 포워드 구성이 누락되었습니다. | [에뮬레이터 클립보드 포워딩 활성화](#enable-emulator-clipboard-forwarding) |
| **`Advisory lock / update failures`** | 자동 업데이트 스레드 잠금 또는 읽기 전용 디렉터리 경로입니다. | [자동 업데이트 잠금 및 실패 해결](#resolve-self-updater-locks-and-failures) |

---

## 쉘 PATH 구성

### 증상

`agy`를 실행하면 다음과 같은 쉘 터미널 오류가 반환됩니다.

```bash
bash: agy: command not found
```

### 원인

설치 유틸리티가 바이너리를 `~/.local/bin`(또는 `C:\Users\<Username>\AppData\Local\agy\bin`)에 다운로드하지만, 쉘의 활성 `$PATH` 환경이 이 디렉터리를 인덱싱하지 않습니다.

### 해결책

터미널 세션이 바이너리 경로를 로드하는지 확인합니다.

**macOS 및 Linux**:

1. 쉘 구성 파일(`~/.bashrc` 또는 `~/.zshrc`)을 엽니다.
2. 파일 끝에 다음 줄을 확인하거나 추가합니다.
   ```bash
   export PATH="~/.local/bin:$PATH"
   ```
3. 프로필 구성을 다시 로드합니다.
   ```bash
   source ~/.zshrc
   ```

**Windows (PowerShell)**:

1. 관리자 권한으로 PowerShell 터미널을 열고 다음을 실행합니다.
   ```powershell
   [System.Environment]::SetEnvironmentVariable("Path", [System.Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Program Files\Google\antigravity-cli", "User")
   ```
2. 시스템 레지스트리 환경을 새로 고치려면 터미널 에뮬레이터를 다시 시작하십시오.

---

## 키링 권한 승인

### 증상

시작할 때 CLI가 중단되거나 DBUS 경고를 출력하거나 키링 액세스 예외를 발생시킵니다.

```text
Error: failed to retrieve token: secret keyring is locked
```

### 원인

Antigravity CLI는 세션 토큰을 암호화하기 위해 보안 키체인 라이브러리(Apple 키체인, dbus를 통한 Linux secret-service 또는 Windows 자격 증명 관리자)를 활용합니다. 백그라운드 데몬이 잠겨 있거나 헤드리스 상태인 경우 CLI가 자격 증명을 읽을 수 없습니다.

### 해결책

**macOS**:

1. **키체인 접근(Keychain Access)** 앱을 엽니다.
2. `Antigravity CLI` 보안 항목을 검색합니다.
3. 마우스 오른쪽 버튼을 클릭하고 **정보 가져오기**를 선택한 다음 **적용 대상(Access Control)** 탭을 선택하고 `agy`가 허용된 애플리케이션 목록에 있는지 확인합니다.
4. Mac의 헤드리스 SSH 세션 내에서 실행 중인 경우 다음 잠금 해제 시퀀스를 실행합니다.
   ```bash
   security unlock-keychain -p "your_keychain_password" login.keychain
   ```

**Linux**:

시스템 키링(예: GNOME Keyring 또는 KWallet)이 잠금 해제되어 있고 액세스할 수 있는지 확인합니다.

헤드리스 환경 또는 SSH를 통해 실행 중인 경우, D-Bus 세션이 활성화되어 있고 키링 데몬이 실행 중인지 확인합니다. 일반적으로 다음을 실행하여 D-Bus 세션을 초기화할 수 있습니다.

```bash
export $(dbus-launch)
```

여전히 액세스 문제가 발생하는 경우 사용자 계정에 키링 서비스에 액세스하는 데 필요한 권한이 있는지 확인하거나 지원 팀에 문의하십시오.

---

## 에뮬레이터 클립보드 포워딩 활성화

### 증상

SSH 터미널 내에서 `Ctrl+V`를 통해 스크린샷이나 미디어 파일을 붙여넣으면 오류 알림이 반환됩니다.

```text
Error: local pasteboard is empty or unreachable over SSH connection
```

### 원인

표준 SSH 스트림은 그래픽 클립보드를 전달하지 않습니다. 그래픽 업로드에는 특정 터미널 멀티플렉서 프로토콜이 필요합니다.

### 해결책

지원되는 터미널 에뮬레이터 및 구성을 활용하고 있는지 확인합니다.

1. **iTerm2 또는 Ghostty 사용**: 이 에뮬레이터들은 고급 클립 채널을 지원합니다.
2. **iTerm2 포워딩 구성**:
   - iTerm2 환경설정(`Cmd+,`)을 엽니다.
   - **General** 탭으로 이동하여 **Selection** 하위 메뉴를 선택합니다.
   - **Applications in terminal may access clipboard**(터미널의 애플리케이션이 클립보드에 액세스할 수 있음)를 체크합니다(OSC 52 쓰기 채널 활성화).
3. **멀티플렉서 우회**: `tmux` 내에서 실행 중인 경우 활성 구성이 표준 붙여넣기 클립을 올바르게 매핑하는지 확인합니다.
   ```text
   set -s set-clipboard on
   ```

---

## 자동 업데이트 잠금 및 실패 해결

### 증상

`agy`를 실행하면 중단되거나, 업그레이드 적용에 실패하거나, 권고 잠금 경고가 반환됩니다.

```text
Warning: another background updater process is already active (update.lock)
```

### 원인

Antigravity CLI에는 백그라운드에서 실행되는 네이티브 정적 링크 자동 업데이트 프로그램이 포함되어 있습니다. 동시 프로세스 충돌을 방지하기 위해 `~/.gemini/antigravity-cli/updater/` 내부에서 15분 TTL(Time-To-Live) 디바운스 마커(`last_check.timestamp`)와 권고 잠금(`update.lock`)을 사용합니다. 백그라운드 업데이트 프로세스가 중단되거나, 잠금을 해제하지 않고 충돌하거나, 실행 파일 디렉터리 내에 사용자 파일 시스템 권한이 부족한 경우 후속 업데이트가 차단됩니다.

### 해결책

- **권고 잠금 해제**: 백그라운드 잠금 파일을 수동으로 제거합니다.
  ```bash
  rm -f ~/.gemini/antigravity-cli/updater/update.lock
  ```
- **자동 업데이트 거부/비활성화**: 쉘 프로필(`~/.bashrc` 또는 `~/.zshrc`) 내에서 `AGY_CLI_DISABLE_AUTO_UPDATE` 환경 변수를 `true`로 설정합니다.
  ```bash
  export AGY_CLI_DISABLE_AUTO_UPDATE=true
  ```
- **디렉터리 쓰기 권한 확인**: 사용자 프로필이 대상 설치 디렉터리(Unix의 경우 `~/.local/bin/`, Windows의 경우 `%LOCALAPPDATA%\agy\bin`)에 대한 소유권 및 쓰기 권한을 가지고 있는지 확인합니다.

---

## 다음 단계

빠른 참조 시트를 보거나 고급 권한을 구성하십시오.

- **[CLI 참조](/docs/cli-reference)**: 모든 슬래시 명령 및 시각적 설정 키를 나열하는 요약 테이블입니다.
- **[권한](/docs/cli-permissions)**: 세분화된 허용 및 거부 작업 정책을 구성합니다.
- **[샌드박스](/docs/cli-sandbox)**: OS 수준의 컨테이너 격리 경계를 적용합니다.
- **[플러그인 및 스킬](/docs/cli-plugins)**: 자신만의 사용자 정의 스킬을 생성합니다.
