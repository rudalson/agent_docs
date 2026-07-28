> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 시뮬레이터에서 iOS 앱 테스트하기

> Claude Code Desktop은 Claude가 앱을 빌드, 실행 또는 검사할 때 각 세션마다 별도의 시뮬레이터와 함께 iOS Simulator 창에 앱을 엽니다.

<Note>
  iOS Simulator 창은 macOS의 Claude Code Desktop에서 공개 베타(public beta) 버전으로 제공됩니다. Pro, Max 및 Team 플랜에서 이용할 수 있으며, Enterprise 플랜에서는 이용할 수 없습니다.
</Note>

iOS Simulator 창에는 Claude Code Desktop의 대화 상자 옆에 Apple의 iOS Simulator에서 실행 중인 앱이 표시됩니다. Claude가 시뮬레이터에서 앱을 빌드, 설치, 실행 또는 검사할 때 창이 자동으로 열리고 기기 화면이 라이브로 스트리밍됩니다. 이를 통해 Claude가 앱을 실행하고 테스트하는 모습을 관찰하거나 Claude가 작업을 계속하는 동안 앱을 직접 탭하여 둘러볼 수 있습니다.

시뮬레이터 창은 시뮬레이터를 직접 구동하므로 [컴퓨터 사용](/docs/en/desktop#let-claude-use-your-computer)이 필요하지 않으며 화면을 장악하거나 다른 창을 숨기지 않습니다. CLI에서 Claude는 화면의 시뮬레이터를 마우스로 제어하는 방식인 [컴퓨터 사용](/docs/en/computer-use#test-a-simulator-flow)을 통해 iOS Simulator에 접근합니다.

## 요구 사항

시뮬레이터 창은 데스크톱 앱에 포함되지 않은 Apple의 시뮬레이터 툴링을 사용합니다. 세션을 시작하기 전에 다음 항목이 준비되어 있는지 확인하세요:

* Claude Desktop v1.24012.0 이상
* Apple의 iOS Simulator는 macOS에서만 실행되므로 Mac 컴퓨터 필요
* 시뮬레이터 기기를 제공하는 iOS 플랫폼이 설치된 [Xcode](https://developer.apple.com/xcode/). Xcode에 시뮬레이터가 없다고 나타나는 경우 [시뮬레이터 창에 시뮬레이터를 찾을 수 없다고 표시됨](#the-simulator-pane-says-no-simulators-were-found)을 참조하세요.

<Note>
  이 페이지에서 "기기(device)"란 물리적 하드웨어가 아닌 Xcode의 **Window → Devices and Simulators** 아래에서 관리하는 시뮬레이터 기기 중 하나인 시뮬레이션된 iPhone 또는 iPad를 의미합니다.
</Note>

시뮬레이터 창은 로컬 세션에서만 사용할 수 있습니다. [클라우드](/docs/en/desktop#run-long-running-tasks-remotely) 및 [SSH](/docs/en/desktop#ssh-sessions) 세션에서 Claude는 Mac의 시뮬레이터에 도달할 수 없는 머신에서 실행됩니다.

## 시뮬레이터에서 앱 실행하기

시뮬레이터 창을 열기 위해 별도의 명령이나 설정이 필요하지 않습니다. Claude가 시뮬레이터에서 앱을 실행할 때 창을 엽니다.

<Steps>
  <Step title="iOS 프로젝트 열기">
    Claude Code Desktop에서 **Code** 탭을 열고 앱의 프로젝트를 [프로젝트 폴더](/docs/en/desktop#start-a-session)로 지정하여 세션을 시작하세요. iOS Simulator용 앱을 빌드하는 모든 프로젝트가 작동합니다.
  </Step>

  <Step title="Claude에게 앱 실행 또는 테스트 요청하기">
    앱 실행 또는 검증을 중심으로 작업을 표현하세요. 예:

    ```text theme={null}
    Build the app and run it in the simulator to check the onboarding flow.
    ```
  </Step>

  <Step title="시뮬레이터 창에서 앱 관찰하기">
    시뮬레이터에서 앱이 실행되면 대화 상자 옆에 iOS Simulator 창이 열립니다. Claude가 기기를 처음 사용할 때 데스크톱 앱에서 허용할 것인지 묻습니다; [기기에 대한 Claude의 접근 권한 부여](#grant-claude-access-to-a-device)를 참조하세요. Claude는 앱을 설치하고, 탭하여 둘러보고, 지켜보는 동안 자체 변경 사항을 검증하기 위해 화면을 읽습니다.
  </Step>
</Steps>

세션 중 언제든 Claude가 시뮬레이터에서 앱을 실행할 때마다 시뮬레이터 창이 열립니다. 요청이 앱을 확인하는 것과 관련된 경우(예: "새 화면이 올바르게 보이나요?"), Claude는 작업을 시작하기 전에 시뮬레이터를 시작합니다. Claude가 버그를 수정하거나 화면을 변경한 후 변경 사항을 검증하도록 요청하세요: 앱을 다시 실행하면 창이 열려 있지 않은 경우 다시 열립니다.

시뮬레이터 창에는 앱이 실제로 실행된 기기가 표시됩니다. 특정 기기에서 테스트하려면 요청 시 해당 기기 이름을 지정하세요(예: "run it on the iPhone SE simulator"). 그러면 Claude가 빌드 및 실행 시 해당 기기를 대상으로 지정합니다.

Claude가 부팅한 기기는 Apple의 Simulator 앱에도 나타나며, Claude는 이미 부팅한 기기에 앱을 설치할 수 있습니다.

직접 시뮬레이터 창을 열 수도 있습니다. 세션에 시뮬레이터가 연결되었거나 Swift 파일을 편집한 경우, 세션 툴바의 **Views** 메뉴에 **iOS Simulator** 항목이 표시됩니다. 창에 아직 기기가 표시되지 않은 경우 **Attach simulator**를 클릭하거나 옆에 있는 기기 메뉴에서 특정 기기를 선택하세요; 종료된 기기를 선택하면 부팅됩니다. Xcode나 해당 시뮬레이터가 없는 경우 창에 대신 설정 단계가 표시되며 완료 시 체크 표시됩니다.

## 직접 시뮬레이터 제어하기

시뮬레이터 창은 단순한 뷰어가 아니라 대화형입니다. Claude가 작업하는 동안이나 작업 사이에 다음을 수행할 수 있습니다:

* 기기 화면을 클릭하고 드래그하여 탭 및 스와이프
* Apple의 Simulator 앱과 동일한 단축키로 하드웨어 버튼 누르기: 홈 버튼 **Cmd+Shift+H**, 잠금 **Cmd+L**, 음량 조절 **Cmd+위쪽 화살표** 및 **Cmd+아래쪽 화살표**
* 회전 버튼 또는 **Cmd+오른쪽 화살표**로 기기를 시계 방향으로 90도 회전
* 각 시뮬레이터의 OS 버전과 부팅 여부를 나열하는 기기 메뉴에서 창이 표시하는 기기 전환
* 창의 캡처 버튼이나 단축키를 사용하여 **Cmd+S**로 스크린샷 저장 또는 **Cmd+R**로 화면 녹화 저장; 파일은 데스크톱에 저장됩니다.
* **Detach simulator**를 클릭하여 종료하지 않고 기기 스트리밍 중지(창이 **Attach simulator** 상태로 돌아감)

기기 이름 아래의 행은 시뮬레이터의 비디오 스트림을 조정합니다. 창이 Mac에 부담을 주는 경우 **Frame rate** 또는 **Resolution**을 낮추거나, **Encoding**을 H.264와 JPEG 중에서 전환하거나, **FPS**를 체크하여 창이 수신하는 프레임 레이트를 표시하세요. 이러한 설정은 앱 실행 방식이 아니라 창의 기기 표시 방식을 변경합니다.

사용자와 Claude가 동일한 기기를 구동하므로 사용자의 탭 조작은 Claude가 보는 앱 상태를 변경합니다. Claude에게 특정 화면을 검사하도록 하려면 탭하여 해당 화면으로 이동한 다음 요청하세요. Claude가 기기를 구동하는 동안 창에는 화면 위에 **Claude is using this device** 배지가 표시됩니다; 결과가 사용자의 입력이 아닌 앱 자체를 반영하도록 배지가 사라질 때까지 탭을 자제하세요.

## 세션의 기기 관리 방식

각 기기는 해당 기기를 실행한 세션에 속하므로 [병렬 세션](/docs/en/desktop#work-in-parallel-with-sessions)은 기기를 공유하지 않습니다: 한 세션의 창에 보이는 내용은 다른 세션의 작업이 아닌 해당 세션의 작업을 반영합니다. 사이드바에서 세션을 전환하면 대화와 함께 시뮬레이터 뷰가 전환되며, 다시 돌아오면 동일한 기기가 중단된 지점에서 다시 시작됩니다. Claude가 하나 이상의 기기에서 작업하는 경우 각 기기는 세션당 최대 4개까지 자체 창을 엽니다.

Claude Code Desktop은 부팅한 시뮬레이터가 더 이상 사용되지 않을 때 이를 종료합니다: 앱을 종료할 때, 세션을 아카이브할 때, 또는 창에서 기기를 분리(detach)한 지 10분 후에 종료합니다. 창에서든 Apple의 Simulator 앱에서든 사용자가 직접 부팅한 기기는 자동으로 종료되지 않습니다. 연결된 기기를 즉시 종료하려면 창의 종료 버튼을 사용하세요.

## 기기에 대한 Claude의 접근 권한 부여

Claude는 기기를 제어하기 전에 사용자 동의를 요청하는 반면, 앱을 빌드하거나 URL을 여는 작업은 세션의 권한 모드를 따릅니다. 사용자나 조직은 Claude의 접근 권한을 완전히 끌 수도 있습니다.

### 처음 기기 허용하기

Claude가 시뮬레이터를 처음 사용할 때 데스크톱 앱에서 허용을 요청합니다. 동의 내용은 해당 기기 제어 및 스크린샷 캡처를 포함하며, 세션당 한 번이 아닌 기기당 한 번 부여합니다. 기기에 대한 Claude의 스크린샷은 Anthropic으로 전송되어 일반 대화 보관 설정에 따라 보관되므로 Claude가 사용하는 기기에서 실제 계정에 로그인하지 마세요.

기기를 허용한 후에는 탭하기, 타이핑하기, 앱 실행하기, 스크린샷 찍기와 같은 Claude의 기기에서의 작업이 추가 프롬프트 없이 실행됩니다. 이는 창에서 직접 클릭하는 것과 동일한 신뢰를 가지며 시뮬레이션된 기기만 만지므로, 창에 컴퓨터 사용에 필요한 macOS Accessibility 및 Screen Recording 권한이 필요하지 않습니다.

거부하더라도 기기는 여전히 부팅되고 사용자의 직접 탭 조작에는 창이 계속 작동합니다; Claude의 접근 권한만 꺼진 상태로 유지됩니다. 나중에 마음이 바뀐 경우 창에서 **Let Claude use it**을 클릭하세요.

### 권한 모드를 따르는 작업

두 가지 작업은 일회성 동의 대신 세션의 [권한 모드](/docs/en/permissions#permission-modes)를 따릅니다:

* 딥링크를 테스트하거나 기기의 Safari에 페이지를 로드하는 등 기기에서 URL 열기 (URL이 기기 외부로 데이터를 전달할 수 있기 때문).
* `xcodebuild`가 Mac에서 프로젝트의 빌드 스크립트를 실행하므로 앱 빌드하기. 이미 진행 중인 빌드를 확인하는 것은 프롬프트를 표시하지 않습니다.

### 시뮬레이터 접근 권한 끄기

데스크톱 앱의 설정에서 Claude의 시뮬레이터 접근 권한을 끌 수 있습니다. 조직에는 모든 사람에 대해 이를 끄는 두 가지 방법이 있습니다:

* `disableMobileSimulatorTools` [관리형 설정](/docs/en/desktop#managed-settings)은 Claude의 시뮬레이터 도구를 차단합니다. 시뮬레이터 창은 사용자의 직접 탭에 계속 사용 가능하며, 앱 내부에서 이 설정을 재정의할 수 없습니다.
* 격리된 가상 머신 내부에서 세션을 실행하도록 요구하는 정책은 창과 도구를 완전히 비활성화합니다.

둘 중 하나가 적용되면 Claude가 알려줍니다.

## 한계 사항

Claude는 시뮬레이션된 기기만 구동하며 물리적 iPhone 또는 iPad는 제어할 수 없습니다. 실제 기기에서 테스트하려면 Xcode에서 직접 앱을 실행한 다음 보이는 내용을 설명하거나 대화에 스크린샷을 첨부하여 Claude가 작업하도록 하세요.

## 문제 해결

### Claude가 앱을 실행할 때 시뮬레이터 창이 열리지 않음

Claude가 앱을 실행하거나 테스트하려는 사용자의 의도를 인식하지 못했거나 시뮬레이터 툴링이 누락되었을 수 있습니다. 다음을 확인하세요:

* 목표를 명시적으로 전달하세요(예: "run the app in the iOS Simulator and tap through the signup flow").
* Simulator 앱을 단독으로 실행하여 Xcode 및 iOS Simulator가 설치되어 있는지 확인하세요.
* 조직에서 Claude Code를 관리하는 경우 정책에 의해 [시뮬레이터 도구가 비활성화](#turn-off-simulator-access)되었을 수 있습니다.
* 시뮬레이터 창에는 Claude Desktop v1.24012.0 이상이 필요합니다. **Claude → Check for Updates**를 연 후 앱을 다시 시작하세요.

### 시뮬레이터 창에 시뮬레이터를 찾을 수 없다고 표시됨

Xcode는 설치되어 있지만 나열할 iOS 시뮬레이터가 없습니다. 시뮬레이터 창에 준수해야 할 설정 단계가 표시되며 각 단계가 완료될 때마다 체크 표시됩니다. 누락된 항목을 수동으로 설치하려면 Xcode 설정에서 iOS 시뮬레이터 런타임을 다운로드하거나 `xcodebuild -downloadPlatform iOS`를 실행하세요.

## 관련 항목

* [Desktop에서의 컴퓨터 사용](/docs/en/desktop#let-claude-use-your-computer): 전용 창이 없는 앱을 위한 화면 제어
* [CLI에서의 컴퓨터 사용](/docs/en/computer-use): CLI가 iOS Simulator에 도달하는 방식
* [세션으로 병렬 작업하기](/docs/en/desktop#work-in-parallel-with-sessions): 세션이 변경 사항을 격리하는 방식
* [Claude Code Desktop 시작하기](/docs/en/desktop-quickstart)
