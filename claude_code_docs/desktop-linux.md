> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Linux에서의 Claude Desktop (beta)

> Ubuntu 및 Debian에서 Claude 데스크톱 앱 설치 및 업데이트하기

<Note>
  Claude 데스크톱 앱의 Linux 지원은 베타(beta) 상태입니다. Chat, Cowork 및 Code 탭을 모두 이용할 수 있습니다.
</Note>

Linux 데스크톱 앱은 macOS 및 Windows와 동일한 Chat, Cowork 및 Claude Code 환경을 제공합니다: 병렬 세션, 시각적 diff 검토, 통합 터미널 및 편집기, 라이브 앱 미리보기. 전체 기능 참조는 [Claude Code Desktop 사용법](/docs/en/desktop)을 참조하세요.

## 요구 사항

* Ubuntu 22.04 이상, 또는 Debian 12 이상
* x86\_64 또는 arm64

이러한 요건을 충족하는 다른 Debian 기반 배포판도 작동할 수 있으나 공식적으로 테스트되지는 않았습니다.

## 설치

시스템의 정기적인 패키지 업데이트를 통해 업데이트가 제공되도록 Anthropic의 apt 리포지토리에서 설치하세요. 터미널을 열고 각 단계의 명령을 실행하세요.

<Steps>
  <Step title="Anthropic의 apt 리포지토리 추가">
    이 단계에서는 신규 Debian 및 Ubuntu 설치 시 포함되어 있지 않을 수 있는 `curl`로 서명 키를 다운로드합니다. 다운로드 명령이 `sudo: curl: command not found` 오류와 함께 실패하면 curl을 먼저 설치하세요:

    ```bash theme={null}
    sudo apt install curl
    ```

    Anthropic의 서명 키를 다운로드합니다:

    ```bash theme={null}
    sudo curl -fsSLo /usr/share/keyrings/claude-desktop-archive-keyring.asc https://downloads.claude.ai/claude-desktop/key.asc
    ```

    리포지토리를 등록합니다:

    ```bash theme={null}
    echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/claude-desktop-archive-keyring.asc] https://downloads.claude.ai/claude-desktop/apt/stable stable main" | sudo tee /etc/apt/sources.list.d/claude-desktop.list
    ```
  </Step>

  <Step title="패키지 설치">
    ```bash theme={null}
    sudo apt update && sudo apt install claude-desktop
    ```
  </Step>

  <Step title="실행 및 로그인">
    애플리케이션 실행기에서 **Claude**를 실행하거나 터미널에서 `claude-desktop`을 실행한 다음 Anthropic 계정으로 로그인하세요.

    Linux 앱은 macOS 및 Windows와 동일한 방식으로 로그인합니다: claude.ai 구독 또는 조직의 SSO를 통해 로그인합니다. Desktop은 Claude Console API 키를 직접 허용하지 않습니다; API 키 인증에는 [CLI](/docs/en/quickstart)를 사용하세요. Desktop을 Google Cloud's Agent Platform 또는 LLM 게이트웨이로 라우팅하는 엔터프라이즈 배포의 경우 [3P에서의 Claude Desktop](https://claude.com/docs/third-party/claude-desktop/overview) 및 [네트워크 구성](/docs/en/network-config)을 참조하세요.
  </Step>
</Steps>

<Accordion title="서명 키 확인">
  다운로드한 서명 키가 Anthropic에 속하는지 확인할 수 있습니다:

  ```bash theme={null}
  gpg --show-keys /usr/share/keyrings/claude-desktop-archive-keyring.asc
  ```

  지문(fingerprint)은 `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`여야 합니다.
</Accordion>

### 다운로드한 파일에서 설치

apt 리포지토리를 통해 설치할 수 없는 경우 리포지토리의 패키지 풀에서 `.deb` 패키지를 직접 다운로드하세요. 이 명령은 리포지토리 색인에서 현재 아키텍처용 최신 패키지를 조회한 다음 현재 디렉터리로 다운로드합니다:

```bash theme={null}
curl -fLO "https://downloads.claude.ai/claude-desktop/apt/stable/$(curl -s "https://downloads.claude.ai/claude-desktop/apt/stable/dists/stable/main/binary-$(dpkg --print-architecture)/Packages" | grep '^Filename: pool/main/c/claude-desktop/claude-desktop_' | sort -V | tail -n 1 | cut -d' ' -f2)"
```

명령이 `Remote file name has no length` 오류와 함께 실패하면 조회가 패키지 경로를 반환하지 않은 것입니다. 이는 네트워크가 `downloads.claude.ai`를 차단하는 등 리포지토리 색인을 가져올 수 없거나, 해당 아키텍처용 패키지가 없음을 의미할 수 있습니다. 네트워크가 `downloads.claude.ai`에 도달할 수 있는지, 그리고 `dpkg --print-architecture`가 `amd64` 또는 `arm64`를 출력하는지 확인하세요; 리포지토리는 다른 아키텍처용 패키지를 게시하지 않습니다.

그런 다음 GNOME Software와 같은 소프트웨어 설치 프로그램으로 다운로드한 파일을 열거나 다운로드한 파일이 포함된 디렉터리에서 apt로 설치하세요:

```bash theme={null}
sudo apt install ./claude-desktop_*.deb
```

apt가 `E: Unsupported file ./claude-desktop_*.deb given on commandline`을 보고하는 경우 패턴이 현재 디렉터리의 `.deb` 파일과 일치하지 않는 것입니다. 다운로드가 완료되었는지 확인한 다음 파일이 포함된 디렉터리에서 명령을 다시 실행하세요.

이 방식으로 설치된 `.deb`는 업데이트를 받지 않습니다. apt를 통해 업데이트를 받으려면 [Anthropic의 apt 리포지토리 추가](#install) 단계에 따라 리포지토리를 등록하세요. 해당 패키지는 주석 처리된 리포지토리 항목을 `/etc/apt/sources.list.d/claude-desktop.list`에 작성하기도 하므로 해당 파일의 `deb` 줄 주석을 해제하는 것도 동일합니다.

## 업데이트

데스크톱 앱은 Linux에서 자체 업데이트되지 않습니다. 업데이트는 시스템의 정기 패키지 업데이트와 함께 제공됩니다:

```bash theme={null}
sudo apt update && sudo apt upgrade
```

배포판의 그래픽 소프트웨어 업데이터도 새 버전을 가져옵니다.

## 제거

```bash theme={null}
sudo apt remove claude-desktop
```

이렇게 하면 앱과 함께 서명 키가 제거되므로 설치 중에 리포지토리 항목을 추가한 경우 해당 항목도 제거하세요:

```bash theme={null}
sudo rm /etc/apt/sources.list.d/claude-desktop.list
```

## 문제 해결

### claude-desktop 패키지를 찾을 수 없음 (Unable to locate package)

`sudo apt install claude-desktop`이 `E: Unable to locate package claude-desktop` 오류와 함께 실패하면 apt가 추가한 리포지토리를 찾지 못한 것입니다. 다음을 확인하세요:

* 리포지토리 항목이 작성되었는지 확인하세요. `cat /etc/apt/sources.list.d/claude-desktop.list` 명령은 [Anthropic의 apt 리포지토리 추가](#install) 단계의 `deb` 줄을 보여줘야 합니다. 파일이 비어 있거나 누락된 경우 해당 단계를 다시 실행하세요.
* 본인의 아키텍처가 지원되는지 확인하세요. `dpkg --print-architecture`는 `amd64` 또는 `arm64`를 출력해야 합니다. 리포지토리는 다른 아키텍처용 패키지를 게시하지 않습니다.
* `sudo apt update`를 다시 실행하고 `downloads.claude.ai`와 관련된 오류가 있는지 출력을 확인하세요. 거기서 발생하는 네트워크 또는 키 오류는 리포지토리가 추가되었지만 도달하거나 검증할 수 없음을 의미합니다.

리포지토리가 올바르게 설정되어 있고 연결 가능하지만 패키지를 여전히 찾을 수 없는 경우 [다운로드한 파일에서 설치](#install-from-a-downloaded-file)를 대신 수행하세요.

## Linux 베타에 아직 포함되지 않은 사항

* **Computer Use**: [앱 및 화면 제어](/docs/en/desktop#let-claude-use-your-computer)는 Linux에서 아직 사용할 수 없습니다.
* **음성 입력(Dictation)**: 음성 입력은 Linux 데스크톱 앱에서 이용할 수 없습니다. 대신 CLI에서 [음성 입력](/docs/en/voice-dictation)을 사용하세요.
* **빠른 입력 전역 단축키(Quick Entry global hotkey)**: X11에서 작동합니다. 네이티브 Wayland에서는 데스크톱 환경의 GlobalShortcuts 포털이 필요합니다.
* **Fedora 및 RHEL**: 현재 Debian 기반 배포판만 지원됩니다. 향후 추가 배포판에 대한 지원이 제공될 예정입니다.

데스크톱 앱에서 아직 사용할 수 없는 기능의 경우 [CLI](/docs/en/quickstart)가 동일한 Claude Code 엔진을 실행하며 더 넓은 범위의 Linux 배포판을 지원합니다; [시스템 요구 사항](/docs/en/setup#system-requirements)을 참조하세요.
