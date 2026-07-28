> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# VS Code에서 Claude Code 사용하기

> VS Code용 Claude Code 확장 프로그램을 설치하고 구성하세요. 인라인 diff, @-멘션, 계획 리뷰 및 키보드 단축키를 통해 AI 코딩 지원을 받으세요.

<img src="https://mintcdn.com/claude-code/-YhHHmtSxwr7W8gy/images/vs-code-extension-interface.jpg?fit=max&auto=format&n=-YhHHmtSxwr7W8gy&q=85&s=300652d5678c63905e6b0ea9e50835f8" alt="VS Code editor with the Claude Code extension panel open on the right side, showing a conversation with Claude" width="2500" height="1155" data-path="images/vs-code-extension-interface.jpg" />

VS Code 확장 프로그램은 IDE에 직접 통합된 Claude Code용 네이티브 그래픽 인터페이스를 제공합니다. 이것이 VS Code에서 Claude Code를 사용하는 권장 방법입니다.

확장 프로그램을 사용하면 Claude의 계획을 수락하기 전에 리뷰 및 편집하고, 수정한 내용을 작성되는 대로 자동 수락하며, 선택 항목에서 특정 줄 범위로 파일을 @-멘션하고, 대화 기록에 접근하며, 별도의 탭이나 창에서 여러 대화를 열 수 있습니다.

## 사전 요구 사항

설치하기 전에 다음 항목을 확인하세요:

* VS Code 1.94.0 이상
* Anthropic 계정: 모든 유료 Claude 구독(Pro, Max, Team 또는 Enterprise) 또는 Claude Console 계정이 작동하며 API 키가 필요하지 않습니다. 확장 프로그램을 처음 열 때 이 계정으로 [로그인](/docs/en/authentication#log-in-to-claude-code)합니다. Amazon Bedrock이나 Google Cloud's Agent Platform과 같은 서드파티 제공업체를 통해 Claude에 접근하는 경우 설정 지침은 [Use third-party providers](#use-third-party-providers)를 참조하세요.

<Tip>
  확장 프로그램은 채팅 패널용 CLI(명령줄 인터페이스)의 자체 복사본을 번들로 제공합니다. VS Code의 통합 터미널에서 `claude`를 실행하려면 [단독 CLI 설치](/docs/en/setup)도 필요합니다. 세부 정보는 [VS Code extension vs. Claude Code CLI](#vs-code-extension-vs-claude-code-cli)를 참조하세요.
</Tip>

## 확장 프로그램 설치

직접 설치하려면 IDE에 해당하는 링크를 클릭하세요:

* [Install for VS Code](vscode:extension/anthropic.claude-code)
* [Install for Cursor](cursor:extension/anthropic.claude-code)

또는 VS Code에서 `Cmd+Shift+X`(Mac) 또는 `Ctrl+Shift+X`(Windows/Linux)를 눌러 확장 프로그램 뷰를 열고 "Claude Code"를 검색한 다음 **Install**을 클릭하세요.

확장 프로그램은 Devin Desktop이나 Kiro와 같은 다른 VS Code 포크(fork)에도 설치할 수 있습니다. 에디터의 확장 프로그램 뷰에서 "Claude Code"를 검색하거나 [Open VSX registry](https://open-vsx.org/extension/Anthropic/claude-code)에서 설치하세요. 에디터가 확장 프로그램을 설치할 수 없는 경우 [CLI를 설치](/docs/en/quickstart)하고 통합 터미널에서 `claude`를 대신 실행하세요. CLI는 모든 터미널에서 작동합니다.

<Note>설치 후 확장 프로그램이 표시되지 않으면 VS Code를 재시작하거나 커맨드 팔레트에서 "Developer: Reload Window"를 실행하세요.</Note>

## 시작하기

설치 후 VS Code 인터페이스를 통해 Claude Code 사용을 시작할 수 있습니다:

<Steps>
  <Step title="Claude Code 패널 열기">
    VS Code 전체에서 번개(Spark) 아이콘은 Claude Code를 나타냅니다: <img src="https://mintcdn.com/claude-code/c5r9_6tjPMzFdDDT/images/vs-code-spark-icon.svg?fit=max&auto=format&n=c5r9_6tjPMzFdDDT&q=85&s=3ca45e00deadec8c8f4b4f807da94505" alt="Spark icon" style={{display: "inline", height: "0.85em", verticalAlign: "middle"}} width="16" height="16" data-path="images/vs-code-spark-icon.svg" />

    Claude를 가장 빠르게 여는 방법은 **Editor Toolbar**(에디터 우측 상단 구석)에서 Spark 아이콘을 클릭하는 것입니다. 아이콘은 파일을 열어 두었을 때만 표시됩니다.

    <img src="https://mintcdn.com/claude-code/mfM-EyoZGnQv8JTc/images/vs-code-editor-icon.png?fit=max&auto=format&n=mfM-EyoZGnQv8JTc&q=85&s=eb4540325d94664c51776dbbfec4cf02" alt="VS Code editor showing the Spark icon in the Editor Toolbar" width="2796" height="734" data-path="images/vs-code-editor-icon.png" />

    Claude Code를 여는 다른 방법:

    * **Activity Bar**: 좌측 사이드바에서 Spark 아이콘을 클릭하여 세션 목록을 엽니다. 모든 세션을 클릭하여 전체 에디터 탭으로 열거나 새 세션을 시작할 수 있습니다. 이 아이콘은 Activity Bar에 항상 표시됩니다.
    * **Command Palette**: `Cmd+Shift+P`(Mac) 또는 `Ctrl+Shift+P`(Windows/Linux)를 누르고 "Claude Code"를 입력한 다음 "Open in New Tab"과 같은 옵션을 선택합니다.
    * **Status Bar**: 창 우측 하단 구석의 **✱ Claude Code**를 클릭합니다. 이는 파일을 열어 두지 않았을 때도 작동합니다.

    Claude 패널을 드래그하여 VS Code 내의 어느 위치로든 재배치할 수 있습니다. 세부 정보는 [Customize your workflow](#customize-your-workflow)를 참조하세요.
  </Step>

  <Step title="로그인">
    패널을 처음 열 때 로그인 화면이 표시됩니다. **Sign in**을 클릭하고 브라우저에서 인증을 완료하세요.

    나중에 **Not logged in · Please run /login**이 표시되면 확장 프로그램이 로그인 화면을 자동으로 다시 엽니다. 표시되지 않는 경우 커맨드 팔레트에서 **Developer: Reload Window**로 창을 다시 로드하세요.

    쉘에 `ANTHROPIC_API_KEY`가 설정되어 있음에도 로그인 프롬프트가 계속 표시되면 VS Code가 쉘 환경을 상속받지 못했을 수 있습니다. 터미널에서 `code .`로 VS Code를 실행하여 환경 변수를 상속받거나 대신 Claude 계정으로 로그인하세요.

    로그인 후 **Learn Claude Code** 체크리스트가 표시됩니다. **Show me**를 클릭하여 각 항목을 진행하거나 X로 닫을 수 있습니다. 나중에 다시 열려면 Extensions → Claude Code 아래의 VS Code 설정에서 **Hide Onboarding**을 해제하세요.
  </Step>

  <Step title="프롬프트 보내기">
    작동 방식 설명, 문제 디버깅, 변경 사항 적용 등 코드나 파일에 대한 도움을 Claude에게 요청하세요.

    <Tip>Claude는 선택한 텍스트를 자동으로 볼 수 있습니다. 프롬프트에 @-멘션 참조(`@file.ts#5-10`과 같은)도 함께 삽입하려면 `Option+K`(Mac) / `Alt+K`(Windows/Linux)를 누르세요.</Tip>

    파일의 특정 줄에 대해 묻는 예시입니다:

    <img src="https://mintcdn.com/claude-code/FVYz38sRY-VuoGHA/images/vs-code-send-prompt.png?fit=max&auto=format&n=FVYz38sRY-VuoGHA/images/vs-code-send-prompt.png" alt="VS Code editor with lines 2-3 selected in a Python file, and the Claude Code panel showing a question about those lines with an @-mention reference" width="3288" height="1876" data-path="images/vs-code-send-prompt.png" />
  </Step>

  <Step title="변경 사항 리뷰">
    Claude가 파일을 편집하고자 할 때 원본과 제안된 변경 사항을 나란히(side-by-side) 비교하여 보여준 다음 승인을 요청합니다. 수락하거나 거절하거나 Claude에게 대신 수행할 작업을 알려줄 수 있습니다. 수락하기 전에 diff 뷰에서 제안된 내용을 직접 편집하면 Claude가 원본 제안과 일치한다고 가정하지 않도록 수정한 사실이 알려집니다.

    <img src="https://mintcdn.com/claude-code/FVYz38sRY-VuoGHA/images/vs-code-edits.png?fit=max&auto=format&n=FVYz38sRY-VuoGHA/images/vs-code-edits.png" alt="VS Code showing a diff of Claude's proposed changes with a permission prompt asking whether to make the edit" width="3292" height="1876" data-path="images/vs-code-edits.png" />
  </Step>
</Steps>

Claude Code로 수행할 수 있는 작업에 대한 더 많은 아이디어는 [Common workflows](/docs/en/common-workflows)를 참조하세요.

<Tip>
  기본 사항에 대한 가이드 투어를 보려면 커맨드 팔레트에서 "Claude Code: Open Walkthrough"를 실행하세요.
</Tip>

## 프롬프트 상자 사용하기

프롬프트 상자는 여러 기능을 지원합니다:

* **권한 모드(Permission modes)**: 프롬프트 상자 하단의 모드 표시기를 클릭하여 모드를 전환하거나 `claudeCode.initialPermissionMode` 아래의 VS Code 설정에서 기본값을 설정하세요. 표시기가 제공하는 모든 모드는 [permission modes](/docs/en/permission-modes#switch-permission-modes)를 참조하세요.
  * **Manual**: Claude는 파일 편집 및 대부분의 쉘 명령 전에 승인을 요청합니다.
  * **Plan**: Claude는 수행할 작업을 설명하고 변경을 적용하기 전에 승인을 기다립니다. VS Code는 Claude가 시작하기 전에 주석을 추가하여 피드백을 줄 수 있도록 플랜을 전체 Markdown 문서로 자동으로 엽니다.
  * **Edit automatically**: Claude가 묻지 않고 편집을 수행합니다.
* **명령어 메뉴(Command menu)**: `/`를 클릭하거나 `/`를 입력하여 명령어 메뉴를 엽니다. 파일 첨부, 모델 전환, 확장 사고(extended thinking) 토글, 플랜 사용량 보기(`/usage`), [Remote Control](/docs/en/remote-control) 세션 시작(`/remote-control`) 등의 옵션이 포함되어 있습니다. 커스터마이즈 섹션에서는 MCP 서버, 훅, 메모리, 권한 및 플러그인에 접근할 수 있습니다. 터미널 아이콘이 있는 항목은 통합 터미널에서 열립니다.
  * {/* min-version: 2.1.203 */}설정 섹션에는 [`remoteControlAtStartup`](/docs/en/settings#available-settings)을 설정하여 [모든 새로운 대화형 세션이 Remote Control에 자동으로 연결되도록 설정](/docs/en/remote-control#enable-remote-control-for-all-sessions)하는 **Enable Remote Control for all sessions**가 포함되어 있습니다. Claude Code v2.1.203 이상이 필요합니다.
* **컨텍스트 표시기(Context indicator)**: 프롬프트 상자는 사용 중인 Claude의 컨텍스트 창 용량을 보여줍니다. Claude는 필요에 따라 자동으로 요약(compact)을 진행하며, 수동으로 `/compact`를 실행할 수도 있습니다.
* **확장 사고(Extended thinking)**: Claude가 복잡한 문제를 추론하는 데 더 많은 시간을 할애할 수 있도록 합니다. 명령어 메뉴(`/`)를 통해 활성화하세요. Claude의 추론 과정은 대화 상에 접힌 블록으로 표시됩니다. 블록을 클릭하여 읽거나 `Ctrl+O`를 눌러 세션의 모든 추론 블록을 펼치거나 접을 수 있습니다. 세부 정보는 [Extended thinking](/docs/en/model-config#extended-thinking)을 참조하세요.
* **여러 줄 입력**: 전송하지 않고 새 줄을 추가하려면 `Shift+Enter`를 누르세요. 이는 질문 대화 상자의 "Other" 자유 텍스트 입력에서도 작동합니다.

### 파일 및 폴더 참조

특정 파일이나 폴더에 대한 컨텍스트를 Claude에게 제공하려면 @-멘션을 사용하세요. `@` 뒤에 파일이나 폴더 이름을 입력하면 Claude가 해당 내용을 읽고 그에 대해 질문에 답변하거나 변경을 수행할 수 있습니다. Claude Code는 모호한 일치(fuzzy matching)를 지원하므로 부분적인 이름을 입력하여 필요한 항목을 찾을 수 있습니다:

```text theme={null}
> Explain the logic in @auth (auth.js, AuthService.ts 등에 모호하게 일치함)
> What's in @src/components/ (폴더의 경우 슬래시로 끝냄)
```

대형 PDF의 경우 전체 파일 대신 특정 페이지를 읽도록 Claude에게 요청할 수 있습니다: 단일 페이지, 페이지 1-10과 같은 범위, 또는 페이지 3 이후와 같은 열린 범위.

에디터에서 텍스트를 선택하면 Claude가 하이라이트된 코드를 자동으로 볼 수 있습니다. 프롬프트 상자 푸터에는 선택된 줄 수가 표시됩니다. 파일 경로 및 줄 번호와 함께 @-멘션 참조(예: `@app.ts#5-10`)를 삽입하려면 `Option+K`(Mac) / `Alt+K`(Windows/Linux)를 누르세요. 선택 표시기를 클릭하여 Claude가 하이라이트된 텍스트를 볼 수 있는지 여부를 토글할 수 있습니다. 눈 모양에 사선이 그어진 아이콘은 선택 항목이 Claude에게 숨겨져 있음을 의미합니다.

또한 `Shift`를 누른 채 파일을 프롬프트 상자로 드래그하여 첨부 파일로 추가할 수 있습니다. 컨텍스트에서 제거하려면 첨부 파일의 X를 클릭하세요.

### 이전 대화 재개하기

대화 기록에 접근하려면 Claude Code 패널 상단의 **Session history** 버튼을 클릭하세요. 키워드로 검색하거나 시간별(Today, Yesterday, Last 7 days 등)로 탐색할 수 있습니다. 전체 메시지 기록과 함께 대화를 재개하려면 세션을 클릭하세요. 새 세션은 첫 번째 메시지를 기반으로 AI가 생성한 제목을 받습니다. 세션 위에 마우스를 올려 이름 변경 및 제거 작업을 나타내세요: 설명이 포함된 제목으로 이름을 변경하거나 목록에서 삭제할 수 있습니다. 세션 재개에 대한 자세한 내용은 [Manage sessions](/docs/en/sessions)를 참조하세요.

### Claude.ai에서 클라우드 세션 재개하기

[Claude Code on the web](/docs/en/claude-code-on-the-web)을 사용하는 경우 해당 클라우드 세션을 VS Code에서 직접 재개할 수 있습니다. 이를 위해서는 Anthropic Console이 아닌 **Claude.ai Subscription**으로 로그인해야 합니다.

<Steps>
  <Step title="세션 기록 열기">
    Claude Code 패널 상단의 **Session history** 버튼을 클릭합니다.
  </Step>

  <Step title="Web 탭 선택">
    대화 상자에 Local과 Web의 두 탭이 표시됩니다. claude.ai의 세션을 보려면 **Web**을 클릭합니다.
  </Step>

  <Step title="재개할 세션 선택">
    클라우드 세션을 탐색하거나 검색합니다. 다운로드하여 로컬에서 대화를 계속하려면 세션을 클릭합니다.
  </Step>
</Steps>

<Note>
  GitHub 리포지토리로 시작된 웹 세션만 Web 탭에 표시됩니다. 재개하면 대화 기록이 로컬로 로드되며 변경 사항은 claude.ai에 다시 동기화되지 않습니다.
</Note>

### 계정 및 사용량 확인

Account & usage 대화 상자를 열려면 명령어 메뉴에서 `/usage`를 실행하세요. 로그인한 계정, 플랜, 현재 세션 및 주간 단위의 사용량 막대와 각 제한이 재설정될 때까지 남은 시간이 표시됩니다.

또한 대화 상자는 플랜 제한에 기여하는 요소를 분석합니다. 캐시 미스, 긴 컨텍스트, 서브에이전트가 많거나 병렬 처리가 심한 세션과 같이 최근 사용량의 10% 이상을 차지하는 동작에 플래그를 지정하고 각각에 대해 사용량을 줄이기 위한 팁을 제공합니다. 기여도 테이블은 각 skill, 서브에이전트, 플로그인 및 MCP 서버에서 얼마나 많은 사용량이 발생했는지 보여줍니다. Claude Code v2.1.174 이상이 필요합니다.

지난 24시간과 지난 7일 사이를 전환하려면 Day 및 Week 토글을 사용하세요. 수치는 대략적이며 이 머신의 로컬 세션에서 계산되므로 다른 기기나 claude.ai의 사용량은 포함되지 않습니다. 사용량 추적 및 절감에 대한 자세한 내용은 [Track your costs](/docs/en/costs#track-your-costs)를 참조하세요.

## 워크플로 커스터마이즈

설정을 완료하고 실행했으면 Claude 패널을 재배치하거나, 여러 세션을 실행하거나, 터미널 모드로 전환할 수 있습니다.

### Claude의 위치 선택

Claude 패널을 드래그하여 VS Code 내의 어느 위치로든 재배치할 수 있습니다. 패널의 탭이나 타이틀 바를 잡고 다음과 같이 드래그하세요:

* **Secondary sidebar**: 창의 우측. 코딩하는 동안 Claude를 계속 볼 수 있게 합니다.
* **Primary sidebar**: Explorer, Search 등의 아이콘이 있는 좌측 사이드바.
* **Editor area**: 파일과 함께 탭으로 Claude를 엽니다. 하위 작업에 유용합니다.

<Tip>
  주 Claude 세션에는 사이드바를 사용하고 하위 작업에는 추가 탭을 두세요. Claude는 선호하는 위치를 기억합니다. Activity Bar의 세션 목록 아이콘은 Claude 패널과 별개입니다. 세션 목록은 Activity Bar에 항상 표시되지만, Claude 패널 아이콘은 패널이 좌측 사이드바에 도킹되어 있을 때만 거기에 표시됩니다.
</Tip>

### 여러 대화 실행하기

추가 대화를 시작하려면 커맨드 팔레트에서 **Open in New Tab** 또는 **Open in New Window**를 사용하세요. 각 대화는 자체 기록과 컨텍스트를 유지하므로 서로 다른 작업을 병렬로 진행할 수 있습니다.

탭을 사용할 때 Spark 아이콘의 작은 색상 점은 상태를 나타냅니다: 파란색은 승인 요청이 보류 중임을 의미하고, 주황색은 탭이 숨겨진 동안 Claude가 작업을 완료했음을 의미합니다.

### 터미널 모드로 전환하기

기본적으로 확장 프로그램은 그래픽 채팅 패널을 엽니다. CLI 스타일 인터페이스를 선호하는 경우 [Use Terminal setting](vscode://settings/claudeCode.useTerminal)을 열고 상자를 체크하세요.

`Cmd+,`(Mac) 또는 `Ctrl+,`(Windows/Linux)로 VS Code 설정을 열고 Extensions → Claude Code로 이동하여 **Use Terminal**을 체크할 수도 있습니다.

## 플러그인 관리

VS Code 확장 프로그램에는 [plugins](/docs/en/plugins)을 설치하고 관리하기 위한 그래픽 인터페이스가 포함되어 있습니다. **Manage plugins** 인터페이스를 열려면 프롬프트 상자에 `/plugins`를 입력하세요.

### 플러그인 설치

플러그인 대화 상자에는 **Plugins**와 **Marketplaces**의 두 탭이 표시됩니다.

Plugins 탭에서:

* **Installed plugins**가 토글 스위치와 함께 상단에 표시되어 사용 또는 비활성화할 수 있습니다
* 구성된 마켓플레이스의 **Available plugins**가 아래에 표시됩니다
* 이름이나 설명으로 플러그인을 필터링하려면 검색하세요
* 사용할 수 있는 플러그인에서 **Install**을 클릭하세요

플러그인을 설치할 때 설치 범위(scope)를 선택하세요:

* **Install for you**: 모든 프로젝트에서 사용 가능 (사용자 범위)
* **Install for this project**: 프로젝트 협업자와 공유됨 (프로젝트 범위)
* **Install locally**: 본인에게만, 이 리포지토리에서만 사용 가능 (로컬 범위)

### 마켓플레이스 관리

플러그인 소스를 추가하거나 제거하려면 **Marketplaces** 탭으로 전환하세요:

* 새 마켓플레이스를 추가하려면 GitHub 리포지토리, URL 또는 로컬 경로를 입력하세요
* 마켓플레이스의 플러그인 목록을 업데이트하려면 새로 고침 아이콘을 클릭하세요
* 마켓플레이스를 제거하려면 휴지통 아이콘을 클릭하세요

 변경을 수행한 후 배너가 표시되어 업데이트를 적용하기 위해 Claude Code를 재시작하도록 안내합니다.

<Note>
  VS Code의 플러그인 관리는 내부적으로 동일한 CLI 명령을 사용합니다. 확장 프로그램에서 구성한 플러그인 및 마켓플레이스는 CLI에서도 사용할 수 있으며 그 반대도 마찬가지입니다.
</Note>

플러그인 시스템에 대한 자세한 내용은 [Plugins](/docs/en/plugins) 및 [Plugin marketplaces](/docs/en/plugin-marketplaces)를 참조하세요.

## Chrome으로 브라우저 작업 자동화

VS Code를 떠나지 않고도 웹 앱을 테스트하고, 콘솔 로그로 디버깅하며, 브라우저 워크플로를 자동화하려면 Claude를 Chrome 브라우저에 연결하세요. 이를 위해서는 버전 1.0.36 이상의 [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn)이 필요합니다.

프롬프트 상자에 `@browser`를 입력한 후 Claude가 수행할 작업을 작성하세요:

```text theme={null}
@browser go to localhost:3000 and check the console for errors
```

첨부 파일 메뉴를 열어 새 탭 열기나 페이지 내용 읽기와 같은 특정 브라우저 도구를 선택할 수도 있습니다.

Claude는 브라우저 작업에 대한 새 탭을 열고 브라우저의 로그인 상태를 공유하므로 이미 로그인되어 있는 모든 사이트에 접근할 수 있습니다.

설치 지침, 기능 전체 목록 및 문제 해결은 [Use Claude Code with Chrome](/docs/en/chrome)을 참조하세요.

## VS Code 명령 및 단축키

커맨드 팔레트(`Cmd+Shift+P`(Mac) 또는 `Ctrl+Shift+P`(Windows/Linux))를 열고 "Claude Code"를 입력하여 Claude Code 확장 프로그램에서 사용할 수 있는 모든 VS Code 명령을 확인하세요.

일부 단축키는 어느 패널이 "포커스"(키보드 입력을 수신)되어 있는지에 따라 달라집니다. 커서가 코드 파일에 있으면 에디터가 포커스됩니다. 커서가 Claude의 프롬프트 상자에 있으면 Claude가 포커스됩니다. 둘 사이를 전환하려면 `Cmd+Esc` / `Ctrl+Esc`를 사용하세요.

<Note>
  이들은 확장 프로그램을 제어하기 위한 VS Code 명령입니다. 내장된 모든 Claude Code 명령을 확장 프로그램에서 사용할 수 있는 것은 아닙니다. 세부 정보는 [VS Code extension vs. Claude Code CLI](#vs-code-extension-vs-claude-code-cli)를 참조하세요.
</Note>

| 명령 | 단축키 | 설명 |
| -------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Focus Input | `Cmd+Esc` (Mac) / `Ctrl+Esc` (Windows/Linux) | 에디터와 Claude 간 포커스 토글 |
| Open in Side Bar | - | 좌측 사이드바에서 Claude 열기 |
| Open in Terminal | - | 터미널 모드에서 Claude 열기 |
| Open in New Tab | `Cmd+Shift+Esc` (Mac) / `Ctrl+Shift+Esc` (Windows/Linux) | 에디터 탭으로 새 대화 열기 |
| Open in New Window | - | 별도의 창에서 새 대화 열기 |
| New Conversation | `Cmd+N` (Mac) / `Ctrl+N` (Windows/Linux) | 새 대화 시작. Claude에 포커스가 맞춰져 있어야 하며 `enableNewConversationShortcut`이 `true`로 설정되어 있어야 함 |
| Reopen Closed Session | `Cmd+Shift+T` (Mac) / `Ctrl+Shift+T` (Windows/Linux) | 가장 최근에 닫은 Claude 세션 탭 다시 열기. 마지막으로 닫은 탭이 Claude 세션이 아닌 경우 VS Code의 일반 닫힌 에디터 다시 열기 기능으로 떨어짐. `enableReopenClosedSessionShortcut`으로 비활성화 가능 |
| Insert @-Mention Reference | `Option+K` (Mac) / `Alt+K` (Windows/Linux) | 현재 파일 및 선택 항목에 대한 참조 삽입 (에디터에 포커스가 맞춰져 있어야 함) |
| Show Logs | - | 확장 프로그램 디버그 로그 보기 |
| Logout | - | Anthropic 계정에서 로그아웃 |

### 다른 도구에서 VS Code 탭 시작하기

확장 프로그램은 `vscode://anthropic.claude-code/open`에 URI 핸들러를 등록합니다. 쉘 별칭, 브라우저 북마크릿, 또는 URL을 열 수 있는 모든 스크립트 등 자체 도구 모음에서 새 Claude Code 탭을 열 때 이를 사용하세요. VS Code가 아직 실행 중이지 않은 경우 URL을 열면 먼저 실행됩니다. VS Code가 이미 실행 중인 경우 현재 포커스된 창에서 URL이 열립니다.

운영체제의 URL 오프너로 핸들러를 호출하세요.

<Tabs>
  <Tab title="macOS">
    ```bash theme={null}
    open "vscode://anthropic.claude-code/open"
    ```
  </Tab>

  <Tab title="Linux">
    ```bash theme={null}
    xdg-open "vscode://anthropic.claude-code/open"
    ```

    `xdg-open` 명령은 `xdg-utils` 패키지에서 제공합니다. 쉘에 찾을 수 없다고 보고되는 경우 [xdg-open is not found on Linux](/docs/en/deep-links#xdg-open-is-not-found-on-linux)를 참조하세요.
  </Tab>

  <Tab title="Windows">
    PowerShell에서:

    ```powershell theme={null}
    Start-Process "vscode://anthropic.claude-code/open"
    ```

    `cmd.exe`에서 `start`는 따옴표로 둘러싸인 첫 번째 인수를 창 제목으로 취급하므로 URL 앞에 빈 제목을 전달하세요:

    ```cmd theme={null}
    start "" "vscode://anthropic.claude-code/open"
    ```
  </Tab>
</Tabs>

핸들러는 두 개의 선택적 쿼리 매개변수를 받습니다:

| 매개변수 | 설명 |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `prompt` | 프롬프트 상자에 사전에 채울 텍스트. URL 인코딩되어야 함. 프롬프트가 미리 채워지지만 자동으로 제출되지는 않음. |
| `session` | 새 대화를 시작하는 대신 재개할 세션 ID. 세션은 현재 VS Code에 열려 있는 작업 공간에 속해야 함. 세션을 찾을 수 없으면 대신 새 대화가 시작됨. 세션이 이미 탭에 열려 있는 경우 해당 탭이 포커스됨. 프로그래밍 방식으로 세션 ID를 캡처하려면 [Continue conversations](/docs/en/headless#continue-conversations)를 참조하세요. |

예를 들어 "review my changes"가 사전에 채워진 탭을 열려면:

```text theme={null}
vscode://anthropic.claude-code/open?prompt=review%20my%20changes
```

VS Code 탭 대신 터미널 세션을 시작하려면 CLI의 `claude-cli://` 핸들러를 사용하세요. [Launch sessions from links](/docs/en/deep-links)를 참조하세요.

## 설정 구성하기

확장 프로그램에는 두 가지 유형의 설정이 있습니다:

* VS Code의 **Extension settings**: VS Code 내에서 확장 프로그램의 동작을 제어합니다. `Cmd+,`(Mac) 또는 `Ctrl+,`(Windows/Linux)로 연 후 Extensions → Claude Code로 이동합니다. `/`를 입력하고 **General Config**를 선택하여 설정을 열 수도 있습니다.
* `~/.claude/settings.json`의 **Claude Code settings**: 확장 프로그램과 CLI 간에 공유됩니다. 허용된 명령, 환경 변수, 훅 및 MCP 서버에 사용합니다. 세부 정보는 [Settings](/docs/en/settings)를 참조하세요.

<Tip>
  VS Code에서 사용할 수 있는 모든 설정에 대해 자동 완성 및 인라인 검증을 직접 받으려면 `settings.json`에 `"$schema": "https://json.schemastore.org/claude-code-settings.json"`을 추가하세요.
</Tip>

### 확장 프로그램 설정 (Extension settings)

| 설정 | 기본값 | 설명 |
| ----------------------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useTerminal` | `false` | 그래픽 패널 대신 터미널 모드로 Claude 시작 |
| `initialPermissionMode` | `default` | 새 대화에 대한 승인 프롬프트를 제어: `default`, `plan`, `acceptEdits`, 또는 `bypassPermissions`. {/* min-version: 2.1.200 */}`manual`은 `default`에 대한 별칭이며 모드 표시기에서 **Manual**로 라벨 지정된 모드를 선택함. Claude Code v2.1.200 이상 필요. [permission modes](/docs/en/permission-modes) 참조 |
| `preferredLocation` | `panel` | Claude가 열리는 위치: `sidebar` (우측) 또는 `panel` (새 탭) |
| `autosave` | `true` | Claude가 읽거나 쓰기 전에 파일 자동 저장 |
| `useCtrlEnterToSend` | `false` | 프롬프트를 보낼 때 Enter 대신 Ctrl/Cmd+Enter 사용 |
| `enableNewConversationShortcut` | `false` | 새 대화를 시작하기 위해 Cmd/Ctrl+N 활성화 |
| `enableReopenClosedSessionShortcut` | `true` | 가장 최근에 닫은 Claude 세션 탭을 다시 열기 위해 Cmd/Ctrl+Shift+T 사용. 마지막으로 닫은 탭이 Claude 세션이 아닌 경우 단축키는 VS Code의 일반 닫힌 에디터 다시 열기 명령을 실행함 |
| `hideOnboarding` | `false` | 온보딩 체크리스트(학사모 아이콘) 숨기기 |
| `respectGitIgnore` | `true` | 파일 검색에서 .gitignore 패턴 제외 |
| `usePythonEnvironment` | `true` | Claude를 실행할 때 작업 공간의 Python 환경 활성화. Python 확장 프로그램 필요 |
| `environmentVariables` | `[]` | Claude 프로세스에 대한 환경 변수 설정. 공유 구성에는 Claude Code 설정을 대신 사용 |
| `disableLoginPrompt` | `false` | 인증 프롬프트 건너뛰기 (서드파티 제공업체 설정용) |
| `allowDangerouslySkipPermissions` | `false` | 모드 선택기에 Bypass 권한 추가. 인터넷 연결이 없는 샌드박스에서만 사용 |
| `claudeProcessWrapper` | - | Claude 프로세스를 실행하는 데 사용되는 실행 파일. 존재할 때 번들로 제공되는 바이너리 경로가 인수로 전달됨. 확장 프로그램 빌드에 플랫폼용 바이너리가 포함되어 있지 않은 경우 별도로 설치된 `claude` 바이너리로 설정하세요. 활성화 시 "Unsupported platform" 오류가 발생하는 것은 플랫폼에 대한 바이너리가 번들로 제공되지 않음을 의미합니다. [which platforms have prebuilt binaries](/docs/en/troubleshoot-install#native-binary-not-found-after-npm-install)를 참조하세요 |

## VS Code 확장 프로그램 vs. Claude Code CLI

Claude Code는 VS Code 확장 프로그램(그래픽 패널)과 CLI(터미널의 명령줄 인터페이스) 모두로 사용할 수 있습니다. 일부 기능은 CLI에서만 사용할 수 있습니다. CLI 전용 기능이 필요한 경우 VS Code의 통합 터미널에서 `claude`를 실행하세요. 이를 위해서는 [단독 CLI 설치](/docs/en/setup)가 필요합니다. 확장 프로그램은 `claude`를 PATH에 추가하지 않습니다. [Run CLI in VS Code](#run-cli-in-vs-code)를 참조하세요.

| 기능 | CLI | VS Code Extension |
| ------------------- | ------------------- | ------------------------------------------------------------------------------------ |
| 명령 및 skill | [전체](/docs/en/commands) | 부분 집합 (사용 가능한 항목을 보려면 `/` 입력) |
| MCP 서버 구성 | 예 | 부분적 (CLI를 통해 서버 추가, 채팅 패널에서 `/mcp`로 기존 서버 관리) |
| 체크포인트 | 예 | 예 |
| `!` bash 단축키 | 예 | 아니오 |
| 탭 자동 완성 | 예 | 아니오 |

### 체크포인트로 되감기

VS Code 확장 프로그램은 Claude의 파일 편집을 추적하고 이전 상태로 되돌릴 수 있는 체크포인트를 지원합니다. 세 가지 옵션 중 선택하려면 메시지 위에 마우스를 올려 되감기 버튼을 나타내세요:

* **Fork conversation from here**: 모든 코드 변경 사항을 유지한 채 이 메시지로부터 새로운 대화 브랜치 시작
* **Rewind code to here**: 전체 대화 기록을 유지한 채 파일 변경 사항을 대화의 이 지점으로 되돌리기
* **Fork conversation and rewind code**: 새로운 대화 브랜치를 시작하고 파일 변경 사항을 이 지점으로 되돌리기

체크포인트 작동 방식과 제한 사항에 대한 전체 세부 정보는 [Checkpointing](/docs/en/checkpointing)을 참조하세요.

### VS Code에서 CLI 실행하기

VS Code에 머물면서 CLI를 사용하려면 통합 터미널(Windows/Linux의 경우 `` Ctrl+` ``, Mac의 경우 `` Cmd+` ``)을 열고 `claude`를 실행하세요. CLI는 diff 보기 및 진단 공유와 같은 기능을 위해 IDE와 자동으로 통합됩니다.

확장 프로그램을 설치해도 쉘 PATH에 `claude`가 추가되지는 않습니다. 확장 프로그램은 채팅 패널용 CLI의 개인 복사본을 번들로 제공하지만, 터미널에서 `claude`를 입력하려면 [단독 CLI 설치](/docs/en/setup)가 필요합니다. 설치를 한 번 실행하면 `claude mcp add` 및 `claude --resume`을 포함하여 이 페이지의 명령어가 모든 터미널에서 작동합니다. 설치 후에도 `claude`를 찾을 수 없는 경우 [PATH를 확인](/docs/en/troubleshoot-install#verify-your-path)하세요.

외부 터미널을 사용하는 경우 Claude Code 내부에서 `/ide`를 실행하여 VS Code에 연결하세요.

### 확장 프로그램과 CLI 간 전환

확장 프로그램과 CLI는 동일한 대화 기록을 공유합니다. CLI에서 확장 프로그램 대화를 계속하려면 터미널에서 `claude --resume`을 실행하세요. 대화를 검색하고 선택할 수 있는 대화형 선택기가 열립니다.

### 프롬프트에 터미널 출력 포함하기

`name`이 터미널 제목인 `@terminal:name`을 사용하여 프롬프트에서 터미널 출력을 참조하세요. 이를 통해 Claude는 복사-붙여넣기 없이 명령 출력, 오류 메시지 또는 로그를 볼 수 있습니다.

### 백그라운드 프로세스 모니터링

Claude가 장시간 실행되는 명령을 실행할 때 확장 프로그램은 상태 표시줄에 진행 상황을 보여줍니다. 그러나 백그라운드 작업에 대한 가시성은 CLI에 비해 제한적입니다. 더 나은 가시성을 위해 Claude가 명령을 출력하도록 하여 VS Code의 통합 터미널에서 실행할 수 있도록 하세요.

### MCP로 외부 도구에 연결

MCP(Model Context Protocol) 서버는 Claude에게 외부 도구, 데이터베이스 및 API에 대한 접근 권한을 부여합니다.

MCP 서버를 추가하려면 통합 터미널(`` Ctrl+` `` 또는 `` Cmd+` ``)을 열고 `claude mcp add`를 실행하세요. 아래 예시는 헤더로 전달된 [personal access token](https://github.com/settings/personal-access-tokens)으로 인증하는 GitHub의 원격 MCP 서버를 추가합니다:

```bash theme={null}
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer YOUR_GITHUB_PAT"
```

`YOUR_GITHUB_PAT`을 개인 액세스 토큰으로 대체하세요. `claude mcp add` 명령은 자격 증명을 검증하지 않고 구성을 저장하므로 여기서는 자리 표시자 값이 수용되지만 나중에 서버 연결에 실패합니다. 연결을 확인하려면 채팅 패널에 `/mcp`를 입력하고 서버에 `connected`가 표시되는지 확인하세요. 잘못된 자격 증명이 있는 서버에는 `failed`가 표시됩니다.

구성되면 Claude에게 도구를 사용하도록 요청하세요(예: "Review PR #456").

VS Code를 떠나지 않고 MCP 서버를 관리하려면 채팅 패널에 `/mcp`를 입력하세요. MCP 관리 대화 상자를 통해 서버를 사용 또는 비활성화하고, 서버에 다시 연결하며, OAuth 인증을 관리할 수 있습니다. 사용 가능한 서버는 [MCP documentation](/docs/en/mcp)를 참조하세요.

## Git 활용하기

Claude Code는 VS Code에서 직접 버전 관리 워크플로를 도울 수 있도록 git과 통합됩니다. Claude에게 변경 사항을 커밋하거나, 풀 리퀘스트를 생성하거나, 브랜치 간에 작업하도록 요청하세요.

### 커밋 및 풀 리퀘스트 생성

Claude는 작업을 기반으로 변경 사항을 스테이징하고, 커밋 메시지를 작성하며, 풀 리퀘스트를 생성할 수 있습니다:

```text theme={null}
> commit my changes with a descriptive message
> create a pr for this feature
> summarize the changes I've made to the auth module
```

풀 리퀘스트를 생성할 때 Claude는 실제 코드 변경 사항을 기반으로 설명을 생성하며 테스트나 구현 결정에 대한 컨텍스트를 추가할 수 있습니다.

### 병렬 작업을 위한 git worktrees 사용

자체 파일과 브랜치를 가진 격리된 워크트리에서 Claude를 시작하려면 `--worktree` (`-w`) 플래그를 사용하세요:

```bash theme={null}
claude --worktree feature-auth
```

각 워크트리는 git 기록을 공유하면서 독립적인 파일 상태를 유지합니다. 이는 다른 작업을 진행할 때 Claude 인스턴스가 서로 간섭하는 것을 방지합니다. 자세한 내용은 [Run parallel sessions with Git worktrees](/docs/en/worktrees)를 참조하세요.

## 서브파티 제공업체 사용

기본적으로 Claude Code는 Anthropic의 API에 직접 연결됩니다. 조직에서 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry를 사용하여 Claude에 접근하는 경우 대신 해당 제공업체를 사용하도록 확장 프로그램을 구성하세요:

<Steps>
  <Step title="로그인 프롬프트 비활성화">
    [Disable Login Prompt setting](vscode://settings/claudeCode.disableLoginPrompt)을 열고 상자를 체크합니다.

    `Cmd+,`(Mac) 또는 `Ctrl+,`(Windows/Linux)로 VS Code 설정을 열고 "Claude Code login"을 검색한 다음 **Disable Login Prompt**를 체크할 수도 있습니다.
  </Step>

  <Step title="제공업체 구성">
    해당 제공업체의 설정 가이드를 따르세요:

    * [Claude Code on Amazon Bedrock](/docs/en/amazon-bedrock)
    * [Claude Code on Google Cloud's Agent Platform](/docs/en/google-vertex-ai)
    * [Claude Code on Microsoft Foundry](/docs/en/microsoft-foundry)

    이 가이드에는 VS Code 확장 프로그램과 CLI 간에 설정이 공유되도록 하는 `~/.claude/settings.json`에서의 제공업체 구성이 포함되어 있습니다.
  </Step>
</Steps>

## 보안 및 개인정보 보호

코드는 비공개로 유지됩니다. Claude Code는 지원을 제공하기 위해 코드를 처리하지만 모델 학습에는 사용하지 않습니다. 데이터 처리 및 로깅 제외 방법에 대한 세부 정보는 [Data and privacy](/docs/en/data-usage)를 참조하세요.

자동 편집 권한이 활성화되어 있으면 Claude Code가 VS Code가 자동으로 실행할 수 있는 VS Code 구성 파일(`settings.json` 또는 `tasks.json` 등)을 수정할 수 있습니다. 신뢰할 수 없는 코드로 작업할 때 위험을 줄이려면:

* 신뢰할 수 없는 작업 공간에 대해 [VS Code Restricted Mode](https://code.visualstudio.com/docs/editor/workspace-trust#_restricted-mode) 활성화
* 편집에 대해 자동 수락 대신 수동 승인 모드 사용
* 변경 사항을 수락하기 전에 신중하게 리뷰

### 내장 IDE MCP 서버

확장 프로그램이 활성화되면 CLI가 자동으로 연결되는 로컬 MCP 서버를 실행합니다. 이것이 CLI가 VS Code의 네이티브 diff 뷰어에서 diff를 열고, `@`-멘션을 위해 현재 선택 항목을 읽으며, Jupyter 노트북에서 작업할 때 VS Code에 셀 실행을 요청하는 방식입니다.

서버 이름은 `ide`이며 구성할 내용이 없기 때문에 `/mcp`에서 숨겨져 있습니다. 그러나 조직에서 `PreToolUse` 훅을 사용하여 MCP 도구를 허용 목록에 추가하는 경우 존재하는지 알아둘 필요가 있습니다.

**선택 항목 및 열린 파일 컨텍스트.** 연결되어 있는 동안 CLI는 보내는 각 프롬프트에 현재 에디터 선택 항목과 활성 파일 경로를 컨텍스트로 포함합니다. 이 상황이 발생할 때 트랜스크립트는 `⧉ Selected N lines from <file>` 라인을 보여줍니다. `.env`와 같은 민감한 파일을 제외하려면 해당 경로에 대한 [`Read` 거부 규칙](/docs/en/permissions#read-and-edit)을 추가하세요. 일치하는 거부 규칙은 선택한 텍스트와 해당 파일에 대한 열린 파일 알림이 모두 Claude에 도달하는 것을 방지합니다.

**전송 및 인증.** 서버는 10000–65535 범위의 임의의 포트에서 `127.0.0.1`에 바인딩되며 포트는 구성할 수 없습니다. 전송 방식은 암호화되지 않은 `ws://`입니다. 소켓이 루프백 전용이므로 트래픽을 캡처할 수 있는 모든 프로세스는 잠금 파일에서 토큰을 읽을 수도 있으므로 TLS가 추가 보호를 제공하지 않기 때문입니다. 각 확장 프로그램 활성화는 새로운 임의의 인증 토큰을 생성하여 `~/.claude/ide/<port>.lock`의 잠금 파일에 기록하며, CLI는 연결하기 위해 이를 `X-Claude-Code-Ide-Authorization` 헤더로 제시해야 합니다. 잠금 파일은 `0700` 디렉터리 내에 `0600` 권한을 가지므로 VS Code를 실행하는 사용자만 읽을 수 있습니다. `CLAUDE_CONFIG_DIR`이 설정되어 있으면 잠금 파일이 `$CLAUDE_CONFIG_DIR/ide/`에 대신 작성됩니다.

**모델에 노출되는 도구.** 서버는 12개 정도의 도구를 호스팅하지만 두 개만 모델에 표시됩니다. 나머지는 diff 열기, 선택 항목 읽기, 파일 저장 등 자체 UI를 위해 CLI가 사용하는 내부 RPC이며 도구 목록이 Claude에 도달하기 전에 필터링됩니다.

| 도구 이름 (훅에 보이는 이름) | 기능 | 읽기 전용 여부 |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- | --------- |
| `mcp__ide__getDiagnostics` | 언어 서버 진단(VS Code의 Problems 패널에 있는 오류 및 경고)을 반환. 선택적으로 하나의 파일로 범위 지정 가능. | 예 |
| `mcp__ide__executeCode` | 활성 Jupyter 노트북의 커널에서 Python 코드를 실행. 아래의 확인 절차 참조. | 아니오 |

**Jupyter 실행은 항상 먼저 확인합니다.** `mcp__ide__executeCode`는 은밀하게 아무것도 실행할 수 없습니다. 각 호출 시 코드는 활성 노트북 끝에 새 셀로 삽입되고, VS Code가 해당 셀을 스크롤하여 보여주며, 네이티브 Quick Pick이 **Execute** 또는 **Cancel**을 요청합니다. 취소하거나 `Esc`로 선택기를 닫으면 Claude에 오류가 반환되고 아무것도 실행되지 않습니다. 또한 활성 노트북이 없거나, Jupyter 확장 프로그램(`ms-toolsai.jupyter`)이 설치되어 있지 않거나, 커널이 Python이 아닌 경우 도구가 바로 거부합니다.

<Note>
  Quick Pick 확인은 `PreToolUse` 훅과 별개입니다. `mcp__ide__executeCode`에 대한 허용 목록 항목은 Claude가 셀 실행을 *제안*하도록 허용하며, VS Code 내부의 Quick Pick이 *실제로* 실행하도록 만드는 것입니다.
</Note>

<a id="troubleshooting" />

## 일반적인 문제 해결

### 확장 프로그램이 설치되지 않음

* 호환되는 버전의 VS Code(1.94.0 이상)를 사용 중인지 확인하세요
* VS Code에 확장 프로그램을 설치할 수 있는 권한이 있는지 확인하세요
* [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)에서 직접 설치해 보세요

### Spark 아이콘이 보이지 않음

Spark 아이콘은 파일을 열어 두었을 때 **Editor Toolbar**(에디터 우측 상단)에 표시됩니다. 보이지 않는 경우:

1. **파일 열기**: 아이콘은 파일이 열려 있어야 합니다. 폴더만 열려 있는 것으로는 부족합니다.
2. **VS Code 버전 확인**: 1.94.0 이상 필요 (Help → About)
3. **VS Code 재시작**: 커맨드 팔레트에서 "Developer: Reload Window" 실행
4. **충돌하는 확장 프로그램 비활성화**: 다른 AI 확장 프로그램(Cline, Continue 등)을 임시로 비활성화
5. **작업 공간 신뢰 확인**: 확장 프로그램은 Restricted Mode에서 작동하지 않음

또는 **Status Bar**(우측 하단 구석)에서 "✱ Claude Code"를 클릭하세요. 파일이 열려 있지 않아도 작동합니다. 커맨드 팔레트(`Cmd+Shift+P` / `Ctrl+Shift+P`)를 사용하고 "Claude Code"를 입력할 수도 있습니다.

### macOS에서 Cmd+Esc가 동작하지 않음

macOS Tahoe 이상에서는 시스템 Game Overlay 단축키가 기본적으로 `Cmd+Esc`로 바인딩되어 VS Code에 도달하기 전에 키 입력을 가로채갑니다. 단축키를 해제하려면:

1. 시스템 설정을 엽니다
2. 키보드로 이동한 다음, 키보드 단축키, 게임 컨트롤러로 이동합니다
3. 게임 오버레이 체크박스를 해제합니다

또는 확장 프로그램을 다른 키로 다시 바인딩하세요: VS Code [Keyboard Shortcuts editor](https://code.visualstudio.com/docs/configure/keybindings)(`Cmd+K Cmd+S`)를 열고 `Claude Code: Focus input`을 검색하여 새 바인딩을 할당하세요.

### Claude Code가 전혀 응답하지 않음

Claude Code가 프롬프트에 응답하지 않는 경우:

1. **인터넷 연결 확인**: 인터넷 연결이 안정적인지 확인하세요
2. **새 대화 시작**: 문제가 지속되는지 확인하기 위해 새 대화를 시작해 보세요
3. **CLI 시도**: 더 자세한 오류 메시지가 표시되는지 확인하기 위해 터미널에서 `claude`를 실행해 보세요

문제가 지속되면 오류 세부 정보와 함께 [GitHub에 이슈를 등록](https://github.com/anthropics/claude-code/issues)하세요.

## 확장 프로그램 제거

Claude Code 확장 프로그램을 제거하려면:

1. 확장 프로그램 뷰를 엽니다 (`Cmd+Shift+X`(Mac) 또는 `Ctrl+Shift+X`(Windows/Linux))
2. "Claude Code"를 검색합니다
3. **Uninstall**을 클릭합니다

VS Code 통합 터미널에서 `claude`를 실행하면 확장 프로그램이 자동으로 다시 설치됩니다. 제거 상태를 유지하려면 `/config`에서 **Auto-install IDE extension**을 끄거나 [`autoInstallIdeExtension`](/docs/en/settings#global-config-settings)을 `false`로 설정하세요. [`CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL`](/docs/en/env-vars) 환경 변수를 `1`로 설정할 수도 있습니다.

확장 프로그램 데이터를 제거하고 모든 설정을 재설정하려면 플랫폼에 해당하는 확장 프로그램의 저장소 디렉터리를 삭제하세요.

macOS의 경우:

```bash theme={null}
rm -rf ~/Library/"Application Support"/Code/User/globalStorage/anthropic.claude-code
```

Linux의 경우:

```bash theme={null}
rm -rf ~/.config/Code/User/globalStorage/anthropic.claude-code
```

Windows의 PowerShell에서:

```powershell theme={null}
Remove-Item -Recurse -Force "$env:APPDATA\Code\User\globalStorage\anthropic.claude-code"
```

추가적인 도움말은 [troubleshooting guide](/docs/en/troubleshooting)를 참조하세요.

## 다음 단계

이제 VS Code에서 Claude Code 설정을 완료했습니다:

* Claude Code를 최대한 활용하기 위해 [Explore common workflows](/docs/en/common-workflows) 탐색
* 외부 도구로 Claude의 기능을 확장하기 위해 [Set up MCP servers](/docs/en/mcp) 설정. CLI를 사용하여 서버를 추가한 다음 채팅 패널에서 `/mcp`로 관리하세요.
* 허용된 명령, 훅 등을 커스터마이즈하기 위해 [Configure Claude Code settings](/docs/en/settings) 구성. 이 설정은 확장 프로그램과 CLI 간에 공유됩니다.
