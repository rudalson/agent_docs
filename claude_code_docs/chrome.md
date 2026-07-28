> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Chrome과 함께 Claude Code 사용하기

> Claude Code를 Chrome 브라우저에 연결하여 웹 앱을 테스트하고, 콘솔 로그로 디버깅하며, 폼 입력을 자동화하고, 웹 페이지에서 데이터를 추출하세요.

Claude Code는 [Claude in Chrome 브라우저 확장 프로그램](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn)과 통합되어 CLI 또는 [VS Code 확장 프로그램](/docs/en/vs-code#automate-browser-tasks-with-chrome)에서 브라우저 자동화 기능을 제공합니다. 컨텍스트를 전환하지 않고 코드를 작성한 후 브라우저에서 바로 테스트하고 디버깅할 수 있습니다.

Claude는 브라우저 작업을 위해 새 탭을 열고 브라우저의 로그인 상태를 공유하므로 이미 로그인한 모든 사이트에 접근할 수 있습니다. 브라우저 작업은 화면에 보이는 Chrome 창에서 실시간으로 실행됩니다. Claude가 로그인 페이지나 CAPTCHA를 만나면 일시 중지하고 수동으로 처리하도록 요청합니다.

<Note>
  Chrome 통합은 Google Chrome 및 Microsoft Edge에서 작동합니다. Claude Code는 확장 프로그램을 감지하여 Brave, Arc, Vivaldi, Opera를 포함한 기타 Chromium 기반 브라우저에서도 연결을 설정합니다. WSL(Windows Subsystem for Linux)에서는 Chrome 통합이 지원되지 않습니다.
</Note>

## 기능

Chrome이 연결되면 단일 워크플로에서 브라우저 작업과 코딩 작업을 연쇄적으로 실행할 수 있습니다:

* **라이브 디버깅**: 콘솔 오류와 DOM 상태를 직접 읽고 이를 유발한 코드를 수정합니다.
* **디자인 검증**: Figma 목업을 기반으로 UI를 제작한 다음 브라우저에서 열어 일치하는지 확인합니다.
* **웹 앱 테스트**: 폼 유효성 검사를 테스트하고, 시각적 회귀를 확인하거나, 사용자 흐름을 검증합니다.
* **인증된 웹 앱**: API 커넥터 없이 Google Docs, Gmail, Notion 또는 로그인된 모든 앱과 상호 작용합니다.
* **데이터 추출**: 웹 페이지에서 구조화된 정보를 가져와 로컬에 저장합니다.
* **작업 자동화**: 데이터 입력, 폼 채우기, 다중 사이트 워크플로와 같은 반복적인 브라우저 작업을 자동화합니다.
* **파일 업로드**: 내 머신의 파일을 웹 페이지의 업로드 필드에 첨부합니다.
* **세션 녹화**: 진행된 상황을 문서화하거나 공유하기 위해 브라우저 상호 작용을 GIF로 녹화합니다.

## 사전 요구 사항

Chrome과 함께 Claude Code를 사용하기 전에 다음이 필요합니다:

* [Google Chrome](https://www.google.com/chrome/), [Microsoft Edge](https://www.microsoft.com/edge), 또는 Brave, Arc, Vivaldi, Opera 등 다른 Chromium 기반 브라우저
* Chrome 웹 스토어에서 제공되는 [Claude in Chrome 확장 프로그램](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) 버전 1.0.36 이상
* [Claude Code](/docs/en/quickstart#step-1-install-claude-code)
* 직접 가입한 Anthropic 플랜 (Pro, Max, Team 또는 Enterprise)

Chrome 통합을 사용하려면 `/login`을 통한 로그인도 필요합니다. API 키 또는 [`claude setup-token`](/docs/en/authentication#generate-a-long-lived-token)의 장기 토큰으로 인증하는 경우, 브라우저 확장 프로그램이 해당 자격 증명으로 인증할 수 없기 때문에 `--chrome`을 전달하더라도 Claude Code는 Chrome 통합을 비활성화된 상태로 유지합니다. v2.1.216 이전에는 이러한 세션에서 Chrome 통합을 활성화할 수 있었지만 브라우저 확장 프로그램에 연결하려는 모든 시도가 403 오류로 실패했습니다.

<Note>
  Chrome 통합은 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry와 같은 서드파티 제공업체를 통해서는 사용할 수 없습니다. 서드파티 제공업체를 통해서만 Claude에 접근하는 경우 이 기능을 사용하려면 별도의 claude.ai 계정이 필요합니다.
</Note>

## CLI에서 시작하기

<Steps>
  <Step title="Chrome을 포함하여 Claude Code 실행">
    `--chrome` 플래그를 사용하여 Claude Code를 시작합니다:

    ```bash theme={null}
    claude --chrome
    ```

    Chrome을 포함하여 처음 실행할 때 Claude Code는 통합을 소개하고 사이트 권한이 작동하는 방식을 설명하는 일회성 대화 상자를 보여줍니다. Enter를 눌러 계속 진행합니다.

    플래그 없이 향후 세션에서 Chrome을 기본으로 활성화하려면 [기본적으로 Chrome 활성화](#기본적으로-chrome-활성화)를 참조하세요.
  </Step>

  <Step title="Claude에게 브라우저 사용 요청">
    이 예제는 터미널이나 에디터에서 페이지로 이동하고, 상호 작용하며, 찾은 내용을 보고합니다:

    ```text theme={null}
    Go to code.claude.com/docs, click on the search box,
    type "hooks", and tell me what results appear
    ```

    첫 번째 브라우저 작업은 `claude-in-chrome` 스킬을 사용할 수 있는 권한을 요청합니다. 승인하면 Claude가 새 탭을 열고 작업을 시작합니다.
  </Step>
</Steps>

언제든지 `/chrome`을 실행하여 연결 상태를 확인하고, 권한을 관리하며, 확장 프로그램을 재연결하거나, 연결된 브라우저 중 사용할 브라우저를 선택할 수 있습니다. 상태 패널에 "Status: Enabled" 및 "Extension: Installed"가 표시되면 통합이 작동 중입니다. 브라우저 작업이 시작될 때 두 개 이상의 브라우저가 연결되어 있으면 Claude가 선택을 요청합니다.

VS Code의 경우 [VS Code에서의 브라우저 자동화](/docs/en/vs-code#automate-browser-tasks-with-chrome)를 참조하세요.

### Claude가 요청할 때 확장 프로그램 설치

대화형 세션에서 Claude가 작업을 위해 브라우저를 사용해야 하지만 Claude Code가 확장 프로그램을 감지하지 못하는 경우, 세션당 최대 한 번 "Claude wants to use your browser"라는 제목의 설치 프롬프트가 표시됩니다. 이 프롬프트를 보려면 Claude Code v2.1.206 이상이 필요합니다. Windows에서 **Install extension** 선택 항목을 사용하려면 v2.1.211 이상이 필요합니다. v2.1.211 이전에는 이 항목을 선택해도 설치 페이지를 열 수 없었습니다.

프롬프트는 세 가지 선택 항목을 제공합니다:

* **Install extension**: 브라우저에서 확장 프로그램 설치 페이지를 열고 안내식 설정을 시작합니다. Claude Code는 설치를 기다린 후 확장 프로그램을 연결하고 동일한 세션에서 브라우저 도구를 활성화합니다. 연결이 준비되면 "Continue with browser tools"를 선택하고 Claude가 브라우저에서 작업을 재개합니다. 언제든지 "Continue without browser tools"를 선택하여 설정을 종료하고 나중에 `/chrome`으로 완료할 수 있습니다.
* **Not now**: 브라우저 도구 없이 작업을 계속합니다. 이 프롬프트는 향후 세션에서 다시 나타날 수 있습니다.
* **Don't ask again**: 모든 향후 세션에서 프롬프트를 중지합니다. 언제든지 `/chrome`을 통해 통합을 다시 설정할 수 있습니다.

조직에서 [`deniedMcpServers` 관리형 설정](/docs/en/managed-mcp#policy-based-control-with-allowlists-and-denylists)으로 `claude-in-chrome` MCP 서버를 차단한 경우 Claude Code는 설치 프롬프트를 표시하지 않습니다.

### 기본적으로 Chrome 활성화

매 세션마다 `--chrome`을 전달하지 않으려면 `/chrome`을 실행하고 "Enabled by default"를 선택하세요.

Chrome이 실행 중이지 않을 때 Claude Code는 정상적으로 시작됩니다. v2.1.211 이전에는 Chrome 통합이 활성화되었지만 Chrome이 실행되지 않은 경우 시작 시 중단(hang)될 수 있었습니다.

[VS Code 확장 프로그램](/docs/en/vs-code#automate-browser-tasks-with-chrome)에서는 Chrome 확장 프로그램이 설치되어 있으면 언제든지 Chrome을 사용할 수 있습니다. 추가 플래그가 필요하지 않습니다.

<Note>
  CLI에서 기본적으로 Chrome을 활성화하면 브라우저 도구가 항상 로드되므로 컨텍스트 사용량이 증가합니다. 컨텍스트 소비 증가가 감지되면 이 설정을 비활성화하고 필요할 때만 `--chrome`을 사용하세요.
</Note>

### 사이트 권한 관리

사이트 수준 권한은 Chrome 확장 프로그램에서 상속됩니다. Chrome 확장 프로그램 설정에서 권한을 관리하여 Claude가 탐색, 클릭 및 입력할 수 있는 사이트를 제어하세요.

### 플랜 모드에서의 브라우저 도구

[플랜 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)에서는 페이지나 브라우저 상태만 읽는 브라우저 도구 호출은 권한 프롬프트 없이 실행되고, 상태를 변경하는 호출은 승인 프롬프트를 표시합니다.

* **읽기 전용 호출**: `read_page`, `get_page_text`, `find`, 콘솔 메시지 또는 네트워크 요청 읽기, 스크린샷 캡처
* **상태 변경 호출**: 클릭, 타이핑, 탐색, 탭 및 창 관리, GIF 녹화

v2.1.199부터 `tabs_context_mcp`의 `createIfEmpty`, 콘솔 및 네트워크 리더의 `clear`, 스크린샷의 `save_to_disk`와 같이 상태 변경 입력 플래그를 설정하는 읽기 전용 호출도 승인 프롬프트를 표시합니다. `browser_batch` 호출은 내부의 모든 작업이 읽기 전용인 경우에만 프롬프트 없이 실행됩니다.

## 예제 워크플로

이 예제는 브라우저 작업과 코딩 작업을 결합하는 일반적인 방법을 보여줍니다. `/mcp`를 실행하고 `claude-in-chrome`을 선택한 다음 **View tools**를 선택하여 사용 가능한 브라우저 도구의 전체 목록을 확인하세요.

### 로컬 웹 애플리케이션 테스트

웹 앱을 개발할 때 Claude에게 변경 사항이 올바르게 작동하는지 검증하도록 요청할 수 있습니다:

```text theme={null}
I just updated the login form validation. Can you open localhost:3000,
try submitting the form with invalid data, and check if the error
messages appear correctly?
```

Claude는 로컬 서버로 이동하여 폼과 상호 작용하고 관찰한 내용을 보고합니다.

### 콘솔 로그를 통한 디버깅

Claude는 콘솔 출력을 읽어 문제 진단을 도울 수 있습니다. 로그가 길어질 수 있으므로 모든 콘솔 출력을 요청하기보다 어떤 패턴을 찾아야 하는지 Claude에게 알려주세요:

```text theme={null}
Open the dashboard page and check the console for any errors when
the page loads.
```

Claude는 콘솔 메시지를 읽고 특정 패턴이나 오류 유형을 필터링할 수 있습니다.

### 폼 채우기 자동화

반복적인 데이터 입력 작업의 속도를 높입니다:

```text theme={null}
I have a spreadsheet of customer contacts in contacts.csv. For each row,
go to the CRM at crm.example.com, click "Add Contact", and fill in the
name, email, and phone fields.
```

Claude는 로컬 파일을 읽고, 웹 인터페이스를 탐색하며, 각 레코드의 데이터를 입력합니다.

### 웹 페이지에 파일 업로드

Claude는 내 머신의 파일을 페이지의 업로드 필드에 첨부할 수 있습니다. Claude Code는 파일을 읽고 그 내용을 브라우저로 보내므로 로컬 및 원격 세션 모두에서 업로드가 작동합니다. Claude Code v2.1.211 이상이 필요합니다.

다음 예제는 폼에 로그 파일을 첨부합니다:

```text theme={null}
Open the bug tracker at bugs.example.com, create a new issue,
and attach logs/session.log to it
```

업로드 시 세 가지 제한 사항이 적용됩니다:

* **권한**: Claude는 세션에서 파일 읽기가 허용된 경우에만 파일을 업로드할 수 있으므로, 파일에 대한 `Read` 접근을 거부하는 [권한 설정](/docs/en/settings#permission-settings)이 있는 경우 업로드도 차단됩니다.
* **크기**: 단일 업로드에는 총 최대 10MB의 파일이 포함될 수 있습니다.
* **하드 링크**: Claude는 `node_modules`와 같은 패키지 관리자 저장소 내부에서 흔히 볼 수 있는 여러 하드 링크가 있는 파일을 거부합니다. 파일을 복사한 후 복사본을 업로드하세요.

### Google Docs에서 초안 작성

API 설정 없이 문서에 직접 글을 쓰도록 Claude를 활용하세요:

```text theme={null}
Draft a project update based on the recent commits and add it to my
Google Doc at docs.google.com/document/d/abc123
```

Claude는 문서를 열고 에디터를 클릭한 후 내용을 입력합니다. 이는 Gmail, Notion, Sheets 등 로그인되어 있는 모든 웹 앱에서 작동합니다.

### 웹 페이지에서 데이터 추출

웹사이트에서 구조화된 정보 추출:

```text theme={null}
Go to the product listings page and extract the name, price, and
availability for each item. Save the results as a CSV file.
```

Claude는 페이지로 이동하여 내용을 읽고 데이터를 구조화된 형식으로 컴파일합니다.

### 다중 사이트 워크플로 실행

여러 웹사이트에 걸쳐 작업 조율:

```text theme={null}
Check my calendar for meetings tomorrow, then for each meeting with
an external attendee, look up their company website and add a note
about what they do.
```

Claude는 여러 탭을 통해 정보를 수집하고 워크플로를 완료합니다.

### 데모 GIF 녹화

브라우저 상호 작용의 공유 가능한 녹화본을 생성합니다:

```text theme={null}
Record a GIF showing how to complete the checkout flow, from adding
an item to the cart through to the confirmation page.
```

Claude는 상호 작용 시퀀스를 녹화하고 GIF 파일로 저장합니다. 녹화본에는 로그인된 페이지의 계정 세부정보를 포함하여 브라우저에 표시되는 모든 내용이 캡처되므로 팀 외부에 공유하기 전에 검토하세요.

### 스크린샷을 디스크에 저장

Claude에게 스크린샷을 파일로 보관하도록 요청합니다:

```text theme={null}
Take a screenshot of the checkout page and save it to disk
```

Claude는 이미지를 디스크에 저장하고 파일 경로를 보고합니다. v2.1.211 이전에는 스크린샷 도구의 `save_to_disk` 옵션이 파일을 쓰지 않았습니다.

## 문제 해결

### 확장 프로그램이 감지되지 않음

Claude Code가 Chrome 확장 프로그램을 감지하지 못하는 경우:

1. `chrome://extensions`에서 Chrome 확장 프로그램이 설치되어 있고 활성화되어 있는지 확인하세요.
2. `claude --version`을 실행하여 Claude Code가 최신 상태인지 확인하세요.
3. Chrome이 실행 중인지 확인하세요.
4. `/chrome`을 실행하고 "Reconnect extension"을 선택하여 연결을 다시 설정하세요.
5. 문제가 지속되면 Claude Code와 Chrome을 모두 재시작하세요.

Chrome 통합을 처음 활성화하면 Claude Code는 기본 메시징 호스트 설정 파일을 설치합니다. Chrome은 시작할 때 이 파일을 읽으므로 첫 번째 시도에서 확장 프로그램이 감지되지 않으면 Chrome을 재시작하여 새 설정을 가져오도록 하세요.

v2.1.199부터 Claude Code는 해당 첫 설치 시에만 확장 프로그램 연결을 요청하는 브라우저 탭을 엽니다. Claude Code 빌드나 설정 디렉터리를 전환한 후와 같이 설정 파일을 다시 쓰는 이후 세션에서는 탭이 다시 열리지 않습니다.

여전히 연결에 실패하면 다음 위치에 호스트 설정 파일이 존재하는지 확인하세요:

Chrome의 경우:

* **macOS**: `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json`
* **Linux**: `~/.config/google-chrome/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json`
* **Windows**: Windows 레지스트리에서 `HKCU\Software\Google\Chrome\NativeMessagingHosts\` 확인

Edge의 경우:

* **macOS**: `~/Library/Application Support/Microsoft Edge/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json`
* **Linux**: `~/.config/microsoft-edge/NativeMessagingHosts/com.anthropic.claude_code_browser_extension.json`
* **Windows**: Windows 레지스트리에서 `HKCU\Software\Microsoft\Edge\NativeMessagingHosts\` 확인

기타 Chromium 기반 브라우저는 해당 브라우저 이름이 지정된 자체 설정 디렉터리에서 동일한 파일을 읽습니다. 예를 들어 macOS의 Brave는 `~/Library/Application Support/BraveSoftware/Brave-Browser/NativeMessagingHosts/`를 사용하며, Windows에서 각 브라우저는 `HKCU\Software\BraveSoftware\Brave-Browser\NativeMessagingHosts\`와 같은 자체 레지스트리 키를 가집니다.

### 브라우저가 응답하지 않음

Claude의 브라우저 명령 동작이 중단된 경우:

1. 모달 대화 상자(alert, confirm, prompt)가 페이지를 차단하고 있는지 확인하세요. JavaScript 대화 상자는 브라우저 이벤트를 차단하고 Claude가 명령을 수신하지 못하게 합니다. 대화 상자를 수동으로 닫은 후 Claude에게 계속하도록 전달하세요.
2. Claude에게 새 탭을 만들어 다시 시도하도록 요청하세요.
3. `chrome://extensions`에서 Chrome 확장 프로그램을 비활성화했다가 다시 활성화하여 재시작하세요.

### 장시간 세션 중 연결 끊김

Chrome 확장 프로그램의 서비스 워커는 긴 세션 동안 유휴 상태가 될 수 있으며, 이로 인해 연결이 끊어집니다. 비활동 상태가 지속된 후 브라우저 도구가 작동하지 않으면 `/chrome`을 실행하고 "Reconnect extension"을 선택하세요.

### Windows 전용 문제

Windows에서는 다음과 같은 문제를 만날 수 있습니다:

* **네임드 파이프 충돌 (EADDRINUSE)**: 다른 프로세스가 동일한 네임드 파이프를 사용 중인 경우 Claude Code를 재시작하세요. Chrome을 사용 중일 수 있는 다른 모든 Claude Code 세션을 닫습니다.
* **기본 메시징 호스트 오류**: 시작 시 기본 메시징 호스트가 충돌하는 경우 Claude Code를 다시 설치하여 호스트 설정을 재생성해 보세요.
* **설정 페이지가 열리지 않음**: {/* min-version: 2.1.211 */}Claude Code를 업데이트하세요. v2.1.211 이전에는 Windows에서 확장 프로그램 연결을 유도하는 브라우저 탭이 열리지 않을 수 있었습니다.

### 자주 발생하는 오류 메시지

가장 자주 발생하는 오류와 해결 방법입니다:

| 오류                                        | 원인                                             | 해결 방법                                                       |
| ------------------------------------------- | ------------------------------------------------ | --------------------------------------------------------------- |
| "Browser extension is not connected"        | 기본 메시징 호스트가 확장 프로그램에 도달할 수 없음 | Chrome과 Claude Code를 재시작한 후 `/chrome`을 실행하여 재연결 |
| `/chrome`에서 확장 프로그램이 "Not detected"로 표시됨 | Chrome 확장 프로그램이 설치되지 않았거나 비활성화됨 | `chrome://extensions`에서 확장 프로그램을 설치하거나 활성화   |
| "No tab available"                          | 탭이 준비되기 전에 Claude가 작업을 시도함         | Claude에게 새 탭을 열고 다시 시도하도록 요청                     |
| "Receiving end does not exist"              | 확장 프로그램 서비스 워커가 유휴 상태가 됨       | `/chrome`을 실행하고 "Reconnect extension" 선택                 |

## 참고 항목

* [컴퓨터 사용 (Computer use)](/docs/en/computer-use): 브라우저에서 수행할 수 없는 작업 시 기본 macOS 앱 제어
* [VS Code에서 Claude Code 사용](/docs/en/vs-code#automate-browser-tasks-with-chrome): VS Code 확장 프로그램에서의 브라우저 자동화
* [CLI 참조](/docs/en/cli-reference): `--chrome`을 포함한 명령줄 플래그
* [일반적인 워크플로](/docs/en/common-workflows): Claude Code를 사용하는 더 많은 방법
* [데이터 및 개인정보보호](/docs/en/data-usage): Claude Code가 데이터를 처리하는 방법
* [Chrome에서 Claude 시작하기](https://support.claude.com/en/articles/12012173-getting-started-with-claude-in-chrome): 단축키, 스케줄링 및 권한을 포함한 Chrome 확장 프로그램용 전체 설명서
