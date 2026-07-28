---
slug: cli-getting-started
section: Antigravity CLI
title: 시작하기
path:
  - Antigravity CLI
  - 시작하기
---

# Antigravity CLI 시작하기

Antigravity CLI에 오신 것을 환영합니다! 이 가이드는 클라이언트를 설치하고, 터미널 사용자 인터페이스(TUI)를 실행하며, 자율 에이전트와의 협업을 시작할 수 있는 직접적이고 핵심적인 개발자 로드맵을 제공합니다.

## 로드맵 체크리스트

첫 번째 세션을 시작하려면 다음 단계를 순서대로 완료하세요.

1. **클라이언트 설치 (빠른 방법)**

   사용 중인 운영 체제에 맞는 빠른 설치 명령을 실행합니다.

   **macOS / Linux**:

   ```bash
   curl -fsSL https://antigravity.google/cli/install.sh | bash
   ```

   **Windows (PowerShell)**:

   ```powershell
   irm https://antigravity.google/cli/install.ps1 | iex
   ```

   **Windows (CMD)**:

   ```cmd
   curl -fsSL https://antigravity.google/cli/install.cmd -o install.cmd && install.cmd && del install.cmd
   ```

   기본적으로 설치 프로그램은 플랫폼에 지정된 디렉토리에 `agy` 바이너리를 등록합니다.
   - **macOS / Linux**: `~/.local/bin/agy`
   - **Windows**: `C:\Users\<Username>\AppData\Local\agy\bin` (여기서 `<Username>`은 활성화된 Windows 프로필 이름을 나타냅니다).

   <Announcement>
   icon: info
   iconColor: var(--theme-primary)
   color: var(--theme-surface-surface-container)
   text: **고급 설정**: 자세한 기업용 자격 증명(Credentials) 구성, 보안 키링 인증 권한, 프록시 설정 또는 설치 오류 해결과 관련된 상세한 내용은 **[설치 및 인증 가이드](/docs/cli-install)**를 참조하세요.
   </Announcement>

2. **프로젝트 내부에서 TUI 실행하기**

   새 터미널 창을 열고, 대상 프로젝트의 코드베이스 디렉토리로 이동한 후 실행 명령을 입력합니다.

   ```bash
   agy
   ```

3. **최초 실행 설정 완료하기**

   최초 실행 시, TUI는 다음과 같은 간단한 대화형 설정을 단계별로 안내합니다.
   - **색상 테마 (Color Scheme)**: 원하는 시각적 테마를 선택합니다 (Solarized, Dark, Solarized Light, 또는 표준 Terminal 색상).
   - **렌더링 모드 (Rendering Mode)**: Alt-Screen 모드(전체 화면 스크롤이 지원되는 대체 버퍼) 또는 Inline 모드(터미널 기록과 연동되는 순차적 스트림)를 선택합니다.
   - **작업 공간 신뢰 (Workspace Trust)**: 리포지토리 디렉토리를 신뢰하는지 확인합니다. 확인이 완료되면 에이전트가 파일을 인덱싱하고 대기 상태에 들어갑니다.

4. **첫 번째 에이전트 작업 실행하기**

   TUI 화면 하단의 프롬프트 입력창에 다음 지침을 입력하고 `Enter`를 누릅니다.

   ```text
   Write a simple python script to fetch web page text
   ```

   에이전트는 작업 공간을 읽고, 작업에 대해 추론한 다음 계획을 제안합니다. TUI 내에서 코드를 검토하고 테스트 명령을 실행하는 방법에 대한 자세한 단계별 튜토리얼은 **[튜토리얼 가이드](/docs/cli-tutorial)**를 참조하세요.

## 관련 리소스

로컬 환경 설정을 최적화하고 고급 협업 도구를 마스터해 보세요.

- **[모범 사례](/docs/cli-best-practices)**: 검증 루프, 계획 단계, 규칙 파일 및 세션 체크포인트를 마스터합니다.
- **[문제 해결](/docs/cli-troubleshooting)**: 일반적인 경로(Path), 키링 또는 SSH 포워딩 오류를 해결합니다.
- **[CLI 레퍼런스](/docs/cli-reference)**: 모든 슬래시 명령, 단축키 및 JSON 키가 나열된 요약 카탈로그를 확인합니다.
