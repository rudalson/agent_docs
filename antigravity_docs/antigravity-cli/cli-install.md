---
slug: cli-install
section: Antigravity CLI
title: 설치 및 인증
path:
  - Antigravity CLI
  - 설치 및 인증
---

# 설치 및 인증

Antigravity CLI를 설치하고, 기업용 요구 사항을 구성하며, 안전한 인증 세션을 설정합니다.

## 설치

Antigravity CLI는 macOS, Linux 및 Windows에서 기본적으로 실행됩니다. 아래의 플랫폼별 스크립트를 사용하여 시스템에 바이너리를 설치하거나 업그레이드하세요.

### macOS 및 Linux

기본 설치 스크립트를 실행하여 실행 파일을 다운로드하고 `~/.local/bin/agy`에 설치합니다.

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

### Windows

설치 스크립트는 로컬 사용자 디렉토리에 `agy` 바이너리를 등록합니다.
`C:\Users\<Username>\AppData\Local\agy\bin` (여기서 `<Username>`은 사용 중인 Windows 사용자 프로필을 나타냅니다).

**PowerShell**:
PowerShell을 열고 다음 설치 스크립트를 실행합니다.

```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

**CMD (명령 프롬프트)**:
표준 명령 프롬프트를 열고 다음을 실행합니다.

```cmd
curl -fsSL https://antigravity.google/cli/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### 설치 플래그 (Flags)

설치 스크립트를 실행할 때 다음과 같은 커스텀 플래그를 추가할 수 있습니다.

- `--skip-aliases`: 쉘 프로필의 별칭(alias) 제거를 건너뜁니다 (스크립트가 기존 `agy` 또는 `antigravity` 쉘 별칭을 제거하거나 업데이트하는 것을 방지합니다).
- `--skip-path`: 쉘 프로필 `PATH` 추가를 건너뜁니다 (스크립트가 쉘 프로필의 동적 환경 변수를 수정하는 것을 방지합니다).

## 인증 워크플로우

Antigravity CLI는 공유 에이전트 하네스와 통신하기 위해 안전한 자격 증명(Credentials) 및 토큰 프로필을 사용합니다.

### 로컬 자동(Silent) 키링 로그인

로컬 머신에서 `agy`를 실행할 때, CLI는 운영 체제의 네이티브 보안 키링(예: Apple Keychain, Linux Secret Service/dbus, 또는 Windows 자격 증명 관리자)에 접근을 시도합니다. 유효한 토큰 프로필이 발견되면, CLI는 브라우저를 열지 않고 자동으로 세션을 인증합니다.

저장된 세션이 발견되지 않은 경우:

1. CLI가 자동으로 로컬 기본 웹 브라우저를 시작합니다.
2. 승인된 계정 자격 증명을 사용하여 로그인합니다.

### 원격 SSH OAuth 흐름

SSH를 통해 실행할 때, CLI는 원격 연결 환경을 감지합니다. 로컬 웹 브라우저를 띄울 수 없으므로, CLI는 수동 URL 루프를 시작합니다.

1. 원격 터미널 세션에서 `agy`를 실행합니다.
2. CLI가 SSH 환경을 감지하고 고유하고 안전한 인증 URL을 표시합니다.
3. 이 URL을 복사하여 로컬 머신의 웹 브라우저에 붙여넣습니다.
4. 승인된 자격 증명으로 로그인하여 인증을 완료합니다.
5. 브라우저에 고유한 영숫자 인증 코드가 표시됩니다.
6. 이 코드를 복사하여 원격 SSH 터미널로 돌아온 뒤 프롬프트에 붙여넣습니다.

## 세션 관리

세션을 종료하면 활성 자격 증명과 로컬 캐시 디렉토리가 지워집니다.

### 로그아웃하기

계정 연결을 해제하고 운영 체제 키링에서 저장된 인증 프로필을 제거하려면 CLI 프롬프트 입력창에 다음 명령을 실행합니다.

```text
/logout
```

## 다음 단계

설치 및 인증이 완료되면 로컬 에이전트와 상호 작용을 시작하세요.

- **[튜토리얼](/docs/cli-tutorial)**: 에이전트와 함께 기본적인 Python 프로젝트를 생성하고 실행해 봅니다.
- **[프롬프트 및 상호작용](/docs/cli-prompting)**: 다중 라인 텍스트 편집, 명령 중단(interrupt) 및 터미널 미디어 붙여넣기 기능을 살펴봅니다.
- **[권한 및 샌드박스](/docs/cli-sandbox)**: 안전한 파일 시스템 디렉토리와 명령 제한을 구성합니다.
