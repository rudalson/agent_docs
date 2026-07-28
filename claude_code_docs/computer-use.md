> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# CLI에서 Claude가 컴퓨터를 사용하도록 허용하기

> Claude Code CLI에서 컴퓨터 사용(computer use)을 활성화하여 Claude가 macOS에서 앱을 열고, 클릭하고, 타이핑하며 화면을 볼 수 있도록 하세요. 터미널을 떠나지 않고 네이티브 앱을 테스트하고 시각적 문제를 디버깅하며 GUI 전용 도구를 자동화하세요.

<Note>
  컴퓨터 사용(computer use)은 Pro 또는 Max 요금제가 필요한 macOS상의 리서치 프리뷰 기능입니다. Team 또는 Enterprise 요금제에서는 이용할 수 없습니다. 대화형 세션이 필요하므로 `-p` 플래그를 사용하는 비대화형 모드에서는 지원되지 않습니다.
</Note>

컴퓨터 사용을 통해 Claude는 앱을 열고, 화면을 제어하며, 사용자가 하는 방식 그대로 컴퓨터에서 작업할 수 있습니다. CLI에서 Claude는 코드를 작성한 동일한 대화 내에서 Swift 앱을 컴파일하고, 실행하고, 모든 버튼을 클릭하며, 결과를 스크린샷으로 캡처할 수 있습니다.

이 페이지는 CLI에서 컴퓨터 사용이 작동하는 방식을 다룹니다. macOS 또는 Windows용 데스크톱 앱의 경우 [데스크톱에서의 컴퓨터 사용](/docs/en/desktop#let-claude-use-your-computer)을 참조하세요.

## 컴퓨터 사용(Computer use)으로 할 수 있는 작업

컴퓨터 사용은 GUI가 필요한 작업(일반적으로 터미널을 벗어나 직접 손으로 수행해야 하는 작업)을 처리합니다.

* **네이티브 앱 빌드 및 검증**: Claude에게 macOS 메뉴 바 앱 빌드를 요청하세요. Claude는 앱을 직접 열어보기도 전에 Swift 코드를 작성하고, 컴파일하고, 실행하며, 모든 컨트롤을 클릭하여 잘 작동하는지 검증합니다.
* **엔드투엔드 UI 테스트**: Claude에게 로컬 Electron 앱을 가리키며 "온보딩 흐름을 테스트해 줘"라고 말해보세요. Claude는 앱을 열고, 가입 절차를 클릭해 진행하며, 각 단계를 스크린샷으로 남깁니다. Playwright 구성이나 테스트 하네스가 필요 없습니다.
* **시각적 및 레이아웃 문제 디버깅**: Claude에게 "작은 창에서 모달이 잘려 보여"라고 전달하세요. Claude는 창 크기를 조정하고, 버그를 재현하고, 스크린샷을 찍고, CSS를 수정하며, 수정 사항을 검증합니다. Claude는 사용자가 보는 것을 그대로 봅니다.
* **GUI 전용 도구 구동**: 디자인 도구, 하드웨어 제어판, iOS 시뮬레이터 또는 CLI나 API가 없는 전용 앱과 상호작용합니다.

## 컴퓨터 사용이 적용되는 경우

Claude는 앱 또는 서비스와 상호작용하는 여러 가지 방법을 가지고 있습니다. 컴퓨터 사용은 가장 범위가 넓고 느리므로 Claude는 가장 정밀한 도구를 먼저 시도합니다.

* 서비스에 대한 [MCP 서버](/docs/en/mcp)가 있는 경우 Claude는 해당 서버를 사용합니다.
* 작업이 셸 명령인 경우 Claude는 Bash를 사용합니다.
* 작업이 브라우저 작업이고 [Claude in Chrome](/docs/en/chrome)이 설정되어 있는 경우 Claude는 해당 기능을 사용합니다.
* 위의 어떤 것도 해당하지 않는 경우 Claude는 컴퓨터 사용을 활용합니다.

화면 제어는 다른 도구가 도달할 수 없는 대상(네이티브 앱, 시뮬레이터, API가 없는 도구)을 위해 예약되어 있습니다. 데스크톱 앱에서 iOS 앱을 실행하거나 테스트하면 화면 제어 대신 전용 [iOS 시뮬레이터 패널](/docs/en/desktop-ios-simulator)이 열립니다. CLI에서는 컴퓨터 사용을 통해 Claude가 iOS 시뮬레이터에 접근합니다.

## 컴퓨터 사용 활성화하기

컴퓨터 사용은 `computer-use`라는 내장 MCP 서버로 제공됩니다. 직접 활성화할 때까지 기본적으로 꺼져 있습니다.

<Steps>
  <Step title="MCP 메뉴 열기">
    대화형 Claude Code 세션에서 다음을 실행합니다.

    ```text theme={null}
    /mcp
    ```

    서버 목록에서 `computer-use`를 찾습니다. 비활성화 상태로 표시됩니다.
  </Step>

  <Step title="서버 활성화">
    `computer-use`를 선택하고 **Enable**을 선택합니다. 이 설정은 프로젝트별로 유지되므로 컴퓨터 사용을 원하는 각 프로젝트에 대해 한 번만 설정하면 됩니다.
  </Step>

  <Step title="macOS 권한 부여">
    Claude가 처음 컴퓨터를 사용하려고 할 때 두 가지 macOS 권한을 부여하라는 프롬프트가 표시됩니다.

    * **손쉬운 사용 (Accessibility)**: Claude가 클릭, 타이핑, 스크롤을 할 수 있게 합니다.
    * **화면 기록 (Screen Recording)**: Claude가 화면에 있는 내용을 볼 수 있게 합니다.

    프롬프트에는 관련 시스템 설정 창을 열 수 있는 링크가 포함되어 있습니다. 두 권한을 모두 허용한 다음 프롬프트에서 **Try again**을 선택하세요. macOS에서 화면 기록 권한을 부여한 후 Claude Code를 재시작해야 할 수도 있습니다.
  </Step>
</Steps>

설정이 완료된 후 GUI가 필요한 작업을 Claude에게 요청해 보세요.

```text theme={null}
Build the app target, launch it, and click through each tab to make
sure nothing crashes. Screenshot any error states you find.
```

## 세션별 앱 승인하기

`computer-use` 서버를 활성화한다고 해서 컴퓨터의 모든 앱에 대한 접근 권한이 Claude에게 자동으로 부여되는 것은 아닙니다. 세션에서 Claude가 특정 앱을 처음 필요로 할 때 터미널에 다음 내용이 포함된 프롬프트가 나타납니다.

* Claude가 제어하고자 하는 앱
* 클립보드 접근과 같은 요청된 추가 권한
* Claude가 작업하는 동안 숨겨질 기타 앱 수

**Allow for this session** 또는 **Deny**를 선택하세요. 승인은 현재 세션 동안 유지됩니다. Claude가 여러 앱을 함께 요청할 때 한 번에 승인할 수 있습니다.

접근 권한 범위가 넓은 앱은 어떤 권한이 부여되는지 알 수 있도록 프롬프트에 추가 경고를 표시합니다.

| 경고 | 적용 대상 |
| :--- | :--- |
| Equivalent to shell access (셸 접근과 동등) | Terminal, iTerm, VS Code, Warp 및 기타 터미널 및 IDE |
| Can read or write any file (모든 파일 읽기/쓰기 가능) | Finder |
| Can change system settings (시스템 설정 변경 가능) | 시스템 설정 (System Settings) |

이러한 앱이 차단되지는 않습니다. 경고를 보고 작업이 해당 수준의 접근을 정당화하는지 직접 결정할 수 있습니다.

또한 Claude의 제어 수준은 앱 카테고리별로 다릅니다. 브라우저 및 트레이딩 플랫폼은 읽기 전용(view-only), 터미널 및 IDE는 클릭 전용(click-only), 그 외 모든 것은 전체 제어 권한을 얻습니다. 전체 등급 내역은 [데스크톱에서의 앱 권한](/docs/en/desktop#app-permissions)을 참조하세요.

## Claude가 화면에서 작동하는 방식

흐름을 이해하면 Claude가 무엇을 할지 예측하고 개입하는 데 도움이 됩니다.

### 한 번에 하나의 세션만 실행

컴퓨터 사용은 첫 번째 컴퓨터 사용 동작부터 해당 세션이 종료될 때까지 머신 전체의 잠금(lock)을 유지합니다. {/* min-version: 2.1.195 */}v2.1.195부터 작업을 완료해도 잠금이 해제되지 않으며 세션을 종료해야 해제됩니다. 다른 Claude Code 세션이 이미 컴퓨터를 사용 중인 경우 어떤 세션이 잠금을 보유하고 있는지 알려주는 메시지와 함께 새 시도가 실패합니다. 해당 세션을 먼저 종료하세요.

### Claude가 작업하는 동안 앱이 숨겨짐

Claude가 화면 제어를 시작하면 승인된 앱과만 상호작용하도록 다른 모든 보이는 앱이 숨겨집니다. 터미널 창은 계속 보이며 스크린샷에서 제외되므로 세션을 지켜볼 수 있으며 Claude는 자신의 출력을 보지 못합니다.

Claude가 턴을 마지면 숨겨진 앱이 자동으로 복원됩니다.

### 스크린샷 자동 축소

Claude Code는 스크린샷을 모델로 보내기 전에 자동으로 축소합니다. Retina 또는 기타 고해상도 디스플레이에서 디스플레이 해상도를 낮추거나 창 크기를 조절할 필요가 없습니다. 기본 Retina 해상도의 16인치 MacBook Pro는 3456×2234로 캡처되고 가로세로 비율을 유지하면서 약 1372×887로 축소됩니다.

목표 크기를 변경하는 설정은 없습니다. 축소 후 화면의 텍스트나 컨트롤이 너무 작아 Claude가 읽을 수 없는 경우 디스플레이 해상도를 변경하는 대신 앱 내부에서 크기를 키우세요.

### 언제든지 중지 가능

Claude가 잠금을 획득하면 macOS 알림이 나타납니다: "Claude is using your computer · press Esc to stop." 어디서나 `Esc`를 누르면 현재 동작이 즉시 중단되며, 터미널에서 `Ctrl+C`를 눌러도 됩니다. 두 경우 모두 Claude가 중지되고, 앱 숨김이 해제되며, 제어권이 반환됩니다. 세션은 종료될 때까지 [컴퓨터 사용 잠금](#one-session-at-a-time)을 유지합니다.

Claude 작업이 끝나면 두 번째 알림이 나타납니다.

## 보안 및 신뢰 경계

<Warning>
  [샌드박스 처리된 Bash 도구](/docs/en/sandboxing)와 달리 컴퓨터 사용은 사용자가 승인한 앱 접근 권한을 가지고 실제 데스크톱에서 실행됩니다. Claude는 각 동작을 확인하고 화면 콘텐츠에서 발생할 수 있는 잠재적 프롬프트 인젝션을 지적하지만 신뢰 경계가 다릅니다. 모범 사례는 [컴퓨터 사용 보안 가이드](https://support.claude.com/en/articles/14128542)를 참조하세요.
</Warning>

내장된 안전장치는 설정 없이도 위험을 줄여줍니다.

* **앱별 승인**: Claude는 현재 세션에서 승인한 앱만 제어할 수 있습니다.
* **파수꾼 경고**: 셸, 파일시스템 또는 시스템 설정 접근 권한을 부여하는 앱은 승인 전에 경고가 표시됩니다.
* **스크린샷에서 터미널 제외**: Claude는 터미널 창을 보지 못하므로 세션 화면의 프롬프트가 모델로 피드백되지 않습니다.
* **전역 이스케이프**: `Esc` 키는 어디서나 컴퓨터 사용을 중단시키며 키 입력이 소비되므로 프롬프트 인젝션이 대화상자를 닫는 데 사용할 수 없습니다.
* **잠금 파일**: 한 번에 하나의 세션만 머신을 제어할 수 있습니다.

## 예시 워크플로우

다음 예시는 컴퓨터 사용을 코딩 작업과 결합하는 일반적인 방법을 보여줍니다.

### 네이티브 빌드 검증

macOS 또는 iOS 앱을 변경한 후 Claude가 한 번에 컴파일하고 검증하도록 하세요.

```text theme={null}
Build the MenuBarStats target, launch it, open the preferences window,
and verify the interval slider updates the label. Screenshot the
preferences window when you're done.
```

Claude는 `xcodebuild`를 실행하고, 앱을 시작하고, UI와 상호작용하며, 찾은 결과를 보고합니다.

### 레이아웃 버그 재현

시각적 버그가 특정 창 크기에서만 나타날 때 Claude가 찾도록 하세요.

```text theme={null}
The settings modal clips its footer on narrow windows. Resize the app
window down until you can reproduce it, screenshot the clipped state,
then check the CSS for the modal container.
```

Claude는 창 크기를 조정하고, 손상된 상태를 캡처하며, 관련 스타일시트를 읽습니다.

### 시뮬레이터 흐름 테스트

XCTest를 작성하지 않고 iOS 시뮬레이터 구동:

```text theme={null}
Open the iOS Simulator, launch the app, tap through the onboarding
screens, and tell me if any screen takes more than a second to load.
```

Claude는 마우스를 사용하는 것과 동일하게 시뮬레이터를 제어합니다. 이 흐름은 CLI에 적용됩니다. 데스크톱 앱에서는 동일한 요청 시 화면 제어 대신 [iOS 시뮬레이터 패널](/docs/en/desktop-ios-simulator)이 열립니다.

## 데스크톱 앱과의 차이점

CLI 및 데스크톱 환경은 몇 가지 차이점을 제외하고 동일한 컴퓨터 사용 엔진을 공유합니다.

| 기능 | 데스크톱 | CLI |
| :--- | :--- | :--- |
| 플랫폼 | macOS 및 Windows | macOS 전용 |
| 활성화 | **Settings > General**에서 전환 (**Desktop app** 아래) | `/mcp`에서 `computer-use` 활성화 |
| 거부된 앱 목록 | 설정에서 구성 가능 | 아직 제공되지 않음 |
| 자동 숨김 해제 전환 | 선택 가능 | 항상 켜짐 |
| 디스패치 연동 | 디스패치로 생성된 세션이 컴퓨터 사용 가능 | 해당 없음 |

## 문제 해결

### "Computer use is in use by another Claude session"

다른 Claude Code 세션이 잠금을 보유하고 있으며 해당 세션이 종료될 때까지 유지됩니다. 해당 세션을 종료하세요. 다른 세션이 충돌로 종료된 경우 Claude가 프로세스가 더 이상 실행되지 않음을 감지하면 잠금이 자동으로 해제됩니다.

### macOS 권한 프롬프트가 계속 다시 나타남

macOS는 화면 기록 권한을 부여한 후 요청 프로세스를 재시작해야 하는 경우가 있습니다. Claude Code를 완전히 종료하고 새 세션을 시작하세요. 프롬프트가 지속되면 **시스템 설정 > 개인정보 보호 및 보안 > 화면 기록**을 열고 터미널 앱이 나열되어 있고 활성화되어 있는지 확인하세요.

### `/mcp`에 `computer-use`가 나타나지 않음

서버는 대상 자격 환경에서만 나타납니다. 다음을 확인하세요.

* macOS 환경인지 확인하세요. CLI에서의 컴퓨터 사용은 Linux나 Windows에서 이용할 수 없습니다. Windows에서는 대신 [데스크톱에서의 컴퓨터 사용](/docs/en/desktop#let-claude-use-your-computer)을 사용하세요.
* Pro 또는 Max 요금제인지 확인하세요. `/status`를 실행하여 구독을 확인하세요.
* claude.ai를 통해 인증되었는지 확인하세요. 컴퓨터 사용은 Amazon Bedrock, Google Cloud Agent Platform 또는 Microsoft Foundry와 같은 서드파티 공급자에서는 사용할 수 없습니다. 서드파티 공급자를 통해서만 Claude에 접근하는 경우 이 기능을 사용하려면 별도의 claude.ai 계정이 필요합니다.
* 대화형 세션인지 확인하세요. 컴퓨터 사용은 `-p` 플래그를 사용하는 비대화형 모드에서는 이용할 수 없습니다.

## 참고 항목

* [데스크톱에서의 컴퓨터 사용](/docs/en/desktop#let-claude-use-your-computer): 그래픽 설정 페이지가 제공되는 동일한 기능
* [Claude in Chrome](/docs/en/chrome): 웹 기반 작업을 위한 브라우저 자동화
* [MCP](/docs/en/mcp): 구조화된 도구 및 API에 Claude 연결
* [샌드박싱](/docs/en/sandboxing): Claude의 Bash 도구가 파일시스템 및 네트워크 접근을 격리하는 방식
* [컴퓨터 사용 보안 가이드](https://support.claude.com/en/articles/14128542): 안전한 컴퓨터 사용을 위한 모범 사례
