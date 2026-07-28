---
title: Firebase Studio 마이그레이션
slug: firebase-studio-migration
section: 마이그레이션
path:
  - 마이그레이션
  - Firebase Studio 마이그레이션
---

# Firebase Studio 마이그레이션

Antigravity는 Google의 차세대 에이전트 퍼스트(agent-first) 플랫폼입니다. 고속의 자율적인 개발 워크플로를 위한 기본적인 공간이 되도록 설계되었습니다. 클라우드 기반 웹 에디터에만 의존하는 대신, Antigravity는 AI의 힘을 로컬 개발 환경으로 직접 가져옵니다.

## 왜 Antigravity인가요?

Antigravity는 Firebase Studio의 웹 기반 코드 뷰(Code view)에 비해 다음과 같은 대폭적인 개선 사항을 제공합니다:

- **로컬 환경 제어**: Antigravity는 머신에서 로컬로 실행되므로 파일 시스템, 버전 및 터미널을 완벽하게 제어할 수 있습니다.
- **진정한 에이전트 기반 개발**: 기본적인 코드 완성을 뛰어넘습니다. Antigravity는 코드베이스 전체에서 독립적으로 포맷팅, 테스트 및 전체 작업을 구현할 수 있는 에이전트 기반 개발 워크플로를 제공합니다.
- **원활한 Firebase 지원**: 여전히 프로젝트를 Firebase에 쉽게 배포하고, Firebase CLI를 통해 Firebase 서비스와 통신하며, 이전과 마찬가지로 로컬에서 함수를 테스트할 수 있습니다.

## Antigravity 둘러보기

적응을 돕기 위해, Firebase Studio의 주요 기능들이 Antigravity의 어디에 위치해 있는지 알려드립니다:

<IconCardGroup>
- link: /docs/editor
  icon: code
  title: 클라우드 IDE 코드 뷰
  description: 익숙한 VS Code 인터페이스를 하드웨어에서 로컬로 실행하며 AI의 힘을 빌려 더욱 강력하게 활용하세요.
- link: /docs/agent
  icon: auto_awesome
  title: 에이전트 채팅
  description: Firebase Studio에서 사용하던 AI 채팅이 이제 IDE에 기본적으로 통합되어 로컬 파일에 대해 자율적인 작업을 수행할 수 있습니다.
- link: /docs/tools
  icon: rocket_launch
  title: App Hosting 배포
  description: Firebase CLI 또는 내장된 IDE 통합을 사용하여 App Hosting URL을 원활하게 배포하고 계속 관리할 수 있습니다.
- link: /docs/browser
  icon: important_devices
  title: 브라우저 기반 에뮬레이터
  description: 더 빠르고 오프라인에서도 가능한 테스트를 위해 Firebase 로컬 에뮬레이터 세트(Local Emulator Suite)를 머신에서 직접 실행하세요.
</IconCardGroup>

## Firebase Studio 프로젝트를 Antigravity로 마이그레이션하기

Antigravity는 로컬 개발 환경에 AI의 힘을 제공하는 로컬 및 에이전트 퍼스트 IDE입니다.

### 사전 요구 사항

로컬에 다음이 설치되어 있고 최신 버전으로 업데이트되어 있는지 확인하세요:

- [Google Antigravity IDE](https://antigravity.google/download)
- [Node.js](https://nodejs.org/en) (버전 20 이상)
- [Firebase CLI](https://firebase.google.com/docs/cli) (버전 15.10.0 이상)

### 1단계: 앱 내보내기 및 초기화

**옵션 1: 자동 마이그레이션**

이 워크플로는 Antigravity 에이전트를 사용하여 프로젝트 변환을 자율적으로 처리합니다.

1. Firebase Studio의 워크스페이스 상단에 있는 **지금 이동(Move now)** 버튼을 클릭합니다.
2. 나타나는 창에 따라 내보내기 방법을 따릅니다:
   - **압축 및 다운로드(Zip and Download)** 버튼이 보이면 클릭합니다.
   - 그렇지 않은 경우 커맨드 팔레트(Mac의 경우 `Cmd` + `Shift` + `P`, ChromeOS/Windows/Linux의 경우 `Ctrl` + `Shift` + `P`)를 열고 **Firebase Studio: 압축 및 다운로드(Firebase Studio: Zip & Download)** 명령을 실행합니다.
3. 압축된 폴더를 로컬에 해제하고 Antigravity에서 엽니다.
4. Antigravity의 에이전트(Agent) 창에 다음 프롬프트를 입력합니다. 워크플로를 최적화하고 토큰을 절약하기 위해 **Gemini Flash** 모델을 선택하는 것을 권장합니다. 파일 변환과 같은 대량의 변환 작업에서 속도와 효율성을 위해 설계된 모델입니다.

```
@fbs-to-agy-export
```

   그러면 Antigravity 에이전트가 프로젝트 마이그레이션을 시작하며, 진행 과정에서 사용자에게 지원을 요청할 것입니다. 에이전트의 안내에 따라 마이그레이션 프로세스를 완료하세요. 오류가 발생하면 에이전트에게 다시 시도하도록 프롬프트를 입력하세요.

<Announcement>
icon: info
color: "#f5f5f5"
iconColor: "#4285f4"
text: "다운로드 창이 나타나지 않으면 브라우저 주소창에서 팝업 차단 아이콘을 확인하고 팝업이 허용되어 있는지 확인하세요."
</Announcement>

**옵션 2: 수동 내보내기**

AI 토큰을 사용하지 않고 마이그레이션을 직접 관리하려는 경우 Firebase CLI를 사용하여 프로젝트를 수동으로 내보낼 수 있습니다. 이 방법은 직관적이며 에이전트 상호 작용이 필요하지 않습니다.

터미널을 열고 압축을 푼 프로젝트 폴더 또는 원래의 `.zip` 파일 경로로 `<path>`를 대체하여 다음 명령을 실행합니다:

```bash
npx firebase-tools@latest studio:export <path>
```

<Announcement>
icon: warning
color: "#fff8e1"
iconColor: "#f9ab00"
text: "<code>studio:export</code> 명령은 현재 Next.js, Flutter 및 Angular 워크스페이스에 최적화되어 있습니다. 다른 워크스페이스 유형에서도 이 명령을 사용할 수 있지만 마이그레이션이 완전히 성공하지 못할 수 있습니다. 현재 마이그레이션 흐름을 개선하기 위해 활발히 작업 중입니다."
</Announcement>

### 2단계: 앱 미리보기

프로젝트를 추출하고 Antigravity에서 열었다면 로컬에서 애플리케이션을 확인할 수 있습니다:

1. Antigravity의 왼쪽 사이드바에 있는 **실행 및 디버그(Run and Debug)** 메뉴로 이동합니다.
2. 재생 버튼을 클릭하여 로컬 개발 서버를 시작합니다.
3. 터미널의 지침에 따라 앱을 미리 봅니다.

<Announcement>
icon: lightbulb
color: "#f5f5f5"
iconColor: "#4285f4"
text: "앱을 세부 조정하거나 문제를 해결하려면 자연어를 사용하여 에이전트와 채팅하세요. 에이전트 창이 숨겨져 있는 경우 창 상단의 에이전트 토글(Toggle Agent) 아이콘을 클릭하여 다시 열 수 있습니다."
</Announcement>

### 3단계: 앱 게시하기

Antigravity는 에이전트 스킬을 사용하여 Firebase 모범 사례에 따라 앱을 게시합니다.

1. 채팅 패널에 다음 프롬프트를 입력하여 에이전트에게 지시합니다:

```
Publish my app
```

2. `firebase deploy`를 실행하라는 메시지가 표시되면 **예(Yes)**를 선택합니다. 이전에 App Hosting에 게시한 적이 있다면 에이전트가 기존 URL에 게시합니다. App Hosting에 처음으로 게시하는 경우 에이전트가 단계를 안내해 드립니다.
3. 향후 업데이트 시에는 Antigravity 채팅 패널에서 에이전트에게 단순히 `publish my app`이라고 지시하면 됩니다.

## 작업 계속하기

Antigravity에서 개발을 계속하는 방법에는 여러 가지가 있습니다.

- **워크플로 실행:** Antigravity 에이전트 채팅 패널에 `@workflows <workflow_name>`을 입력하여 원활하게 워크플로를 실행하고 모델과 작업을 계속 진행할 수 있습니다.
- **App Hosting 배포:** 에이전트 스킬을 활용하여 에이전트를 통해 직접 앱을 원활하게 배포하거나, 플랫폼과 무관한 Firebase CLI 및 GitHub을 사용할 수 있습니다.
- **문제 해결:** 배포 문제가 발생하면 Firebase CLI에 다시 인증하거나 프로젝트 비밀(secrets) 값을 확인해 보세요.

Firebase Studio 여정에 함께해 주셔서 감사합니다. 여러분의 프로토타입과 피드백은 Google의 AI 도구를 형성하는 데 직접적인 영향을 미쳤으며, Antigravity에서 다음에 무엇을 만드실지 기대됩니다!

## 도움이 필요하신가요?

마이그레이션 관련 버그는 [GitHub Issues](https://github.com/firebase/firebase-tools/issues)에 등록해 주세요.
