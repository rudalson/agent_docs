> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 키보드 단축키 커스텀하기

> 키바인딩 구성 파일을 사용하여 Claude Code의 키보드 단축키를 맞춤 설정하세요.

Claude Code는 커스텀 키보드 단축키를 지원합니다. `/keybindings`를 실행하여 `~/.claude/keybindings.json`에 구성 파일을 생성하거나 열어보세요.

## 구성 파일

키바인딩 구성 파일은 `bindings` 배열을 포함하는 개체입니다. 각 블록은 컨텍스트와 키 입력-작업 매핑을 지정합니다.

<Note>키바인딩 파일의 변경 사항은 Claude Code를 다시 시작하지 않아도 자동으로 감지되고 적용됩니다.</Note>

| 필드      | 설명                                        |
| :--------- | :------------------------------------------------- |
| `$schema`  | 편집기 자동 완성을 위한 선택적 JSON Schema URL |
| `$docs`    | 선택적 설명서 URL                         |
| `bindings` | 컨텍스트별 바인딩 블록 배열                 |

다음 예시는 채팅 컨텍스트에서 `Ctrl+E`를 외부 편집기를 열도록 바인딩하고, `Ctrl+U`의 바인딩을 해제합니다:

```json theme={null}
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

## 컨텍스트 (Contexts)

각 바인딩 블록은 바인딩이 적용되는 **컨텍스트**를 지정합니다:

| 컨텍스트           | 설명                                                  |
| :---------------- | :----------------------------------------------------------- |
| `Global`          | 앱의 모든 곳에 적용                                |
| `Chat`            | 메인 채팅 입력 영역                                         |
| `Autocomplete`    | 자동 완성 메뉴가 열려 있음                                    |
| `Settings`        | 설정 메뉴                                                |
| `Confirmation`    | 권한 및 확인 대화 상자                          |
| `Tabs`            | 탭 탐색 구성 요소                                    |
| `Help`            | 도움말 메뉴가 표시됨                                         |
| `Transcript`      | 트랜스크립트 뷰어                                            |
| `HistorySearch`   | 내역 검색 모드 (Ctrl+R)                                 |
| `Task`            | 백그라운드 작업 실행 중                                   |
| `ThemePicker`     | 테마 선택 대화 상자                                          |
| `Attachments`     | 선택 대화 상자의 이미지 첨부 탐색                |
| `Footer`          | 바닥글 표시기 탐색 (작업, 팀, diff, 아티팩트)  |
| `MessageSelector` | 되돌리기 및 요약 대화 상자 메시지 선택                |
| `DiffDialog`      | Diff 뷰어 탐색                                       |
| `ModelPicker`     | 모델 선택기 effort level                                    |
| `Select`          | 일반 선택/목록 구성 요소                               |
| `Plugin`          | 플러그인 대화 상자 (탐색, 발견, 관리)                     |
| `Scroll`          | 전체 화면 모드의 대화 스크롤 및 텍스트 선택 |

## 사용 가능한 작업 (Actions)

작업은 메시지를 전송하는 `chat:submit`이나 작업 목록을 보여주는 `app:toggleTodos`처럼 `namespace:action` 형식을 따릅니다. 각 컨텍스트에는 사용할 수 있는 특정 작업이 있습니다.

### 앱 작업 (App actions)

`Global` 컨텍스트에서 사용할 수 있는 작업:

| 작업                 | 기본값   | 설명                                                                                                  |
| :--------------------- | :-------- | :----------------------------------------------------------------------------------------------------------- |
| `app:interrupt`        | Ctrl+C    | 현재 작업 취소                                                                                     |
| `app:exit`             | Ctrl+D    | Claude Code 종료. 확인을 위해 800ms 내에 두 번 누름                                                        |
| `app:redraw`           | (미바인딩) | 터미널 화면 강제 다시 그리기                                                                                        |
| `app:toggleTodos`      | Ctrl+T    | Claude의 할 일 체크리스트 표시 여부 토글. [`/tasks`](/docs/en/commands) 백그라운드 작업 보기가 아님 |
| `app:toggleTranscript` | Ctrl+O    | 세부 트랜스크립트 토글                                                                                    |

### 내역 작업 (History actions)

명령 내역 탐색을 위한 작업:

| 작업             | 기본값 | 설명           |
| :----------------- | :------ | :-------------------- |
| `history:search`   | Ctrl+R  | 내역 검색 열기   |
| `history:previous` | Up      | 이전 내역 항목 |
| `history:next`     | Down    | 다음 내역 항목     |

### 채팅 작업 (Chat actions)

`Chat` 컨텍스트에서 사용할 수 있는 작업:

| 작업                | 기본값                           | 설명                                                                                                                                                    |
| :-------------------- | :-------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `chat:cancel`         | Escape                            | 현재 입력 취소                                                                                                                                           |
| `chat:clearInput`     | Ctrl+L                            | 입력을 유지하면서 전체 화면을 다시 그립니다. [전체 화면 렌더링](/docs/en/fullscreen#clear-the-conversation)에서는 2초 내에 두 번 눌러 `/clear`를 실행합니다 |
| `chat:clearScreen`    | Cmd+K                             | [전체 화면 렌더링](/docs/en/fullscreen#clear-the-conversation)에서는 2초 내에 두 번 눌러 `/clear`를 실행합니다                                               |
| `chat:killAgents`     | Ctrl+X Ctrl+K                     | 이 세션에서 실행 중인 모든 [백그라운드 서브에이전트](/docs/en/sub-agents#run-subagents-in-foreground-or-background) 중단                                              |
| `chat:cycleMode`      | Shift+Tab\*                       | 권한 모드 순환                                                                                                                                         |
| `chat:modelPicker`    | Meta+P                            | 모델 선택기 열기                                                                                                                                              |
| `chat:fastMode`       | Meta+O                            | Fast 모드 토글                                                                                                                                               |
| `chat:thinkingToggle` | Meta+T                            | 확장 사고 토글                                                                                                                                       |
| `chat:submit`         | Enter                             | 메시지 제출                                                                                                                                                 |
| `chat:newline`        | Ctrl+J                            | 제출하지 않고 줄바꿈 삽입                                                                                                                            |
| `chat:undo`           | Ctrl+\_, Ctrl+Shift+-             | 마지막 작업 취소                                                                                                                                               |
| `chat:externalEditor` | Ctrl+G, Ctrl+X Ctrl+E             | 외부 편집기에서 열기                                                                                                                                        |
| `chat:stash`          | Ctrl+S                            | 현재 프롬프트 스태시                                                                                                                                           |
| `chat:imagePaste`     | Ctrl+V (Windows 및 WSL에서는 Alt+V) | 클립보드에서 이미지 붙여넣기. WSL에서는 기본적으로 두 단축키 모두 바인딩됨                                                                                        |

\*VT 모드가 없는 Windows(Node \<24.2.0/\<22.17.0, Bun \<1.2.23)에서는 기본값이 Meta+M입니다.

### 자동 완성 작업 (Autocomplete actions)

`Autocomplete` 컨텍스트에서 사용할 수 있는 작업:

| 작업                  | 기본값 | 설명         |
| :---------------------- | :------ | :------------------ |
| `autocomplete:accept`   | Tab     | 제안 수락   |
| `autocomplete:dismiss`  | Escape  | 메뉴 닫기        |
| `autocomplete:previous` | Up      | 이전 제안 |
| `autocomplete:next`     | Down    | 다음 제안     |

### 확인 작업 (Confirmation actions)

`Confirmation` 컨텍스트에서 사용할 수 있는 작업:

| 작업                      | 기본값   | 설명                                                                                                                        |
| :-------------------------- | :-------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| `confirm:yes`               | Y, Enter  | 작업 확인                                                                                                                     |
| `confirm:no`                | N, Escape | 작업 거부                                                                                                                     |
| `confirm:previous`          | Up        | 이전 옵션                                                                                                                    |
| `confirm:next`              | Down      | 다음 옵션                                                                                                                        |
| `confirm:nextField`         | Tab       | 다음 필드                                                                                                                         |
| `confirm:previousField`     | (미바인딩) | 이전 필드                                                                                                                     |
| `confirm:toggle`            | Space     | 선택 토글                                                                                                                   |
| `confirm:cycleMode`         | Shift+Tab | 권한 모드 순환                                                                                                             |
| `confirm:toggleExplanation` | Ctrl+E    | Bash 및 PowerShell 권한 프롬프트에서 모델이 생성한 [명령 설명](/docs/en/permissions#permission-system) 토글 |

### 권한 작업 (Permission actions)

권한 대화 상자의 `Confirmation` 컨텍스트에서 사용할 수 있는 작업:

| 작업                   | 기본값   | 설명                                                                                                         |
| :----------------------- | :-------- | :------------------------------------------------------------------------------------------------------------------ |
| `permission:toggleDebug` | (미바인딩) | 권한 디버그 정보 토글. 이전 기본값 Ctrl+D는 `app:exit`을 가렸기 때문에 v2.1.146에서 제거되었습니다 |

### 트랜스크립트 작업 (Transcript actions)

`Transcript` 컨텍스트에서 사용할 수 있는 작업:

| 작업                     | 기본값           | 설명             |
| :------------------------- | :---------------- | :---------------------- |
| `transcript:toggleShowAll` | Ctrl+E            | 모든 내용 표시 토글 |
| `transcript:exit`          | q, Ctrl+C, Escape | 트랜스크립트 보기 종료    |

`transcript:toggleShowAll`은 기본 렌더러에만 적용됩니다; [전체 화면 렌더링](/docs/en/fullscreen)에서는 트랜스크립트 뷰어가 모두 표시 토글을 제공하지 않습니다.

### 내역 검색 작업 (History search actions)

`HistorySearch` 컨텍스트에서 사용할 수 있는 작업:

| 작업                     | 기본값     | 설명                               |
| :------------------------- | :---------- | :---------------------------------------- |
| `historySearch:next`       | Ctrl+R      | 다음 일치 항목                                |
| `historySearch:accept`     | Escape, Tab | 선택 수락                          |
| `historySearch:cancel`     | Ctrl+C      | 검색 취소                             |
| `historySearch:execute`    | Enter       | 선택한 명령 실행                  |
| `historySearch:cycleScope` | Ctrl+S      | 범위 순환: 세션, 프로젝트, 전체 |

`historySearch:next`, `historySearch:accept`, `historySearch:cancel`, `historySearch:execute` 기본값은 항상 모든 프로젝트의 프롬프트를 검색하는 기본 렌더러의 인라인 내역 검색에 적용됩니다. `historySearch:cycleScope`는 `Ctrl+R`이 검색 대화 상자를 열고 `Ctrl+S`가 범위를 순환하는 [전체 화면 렌더링](/docs/en/fullscreen)에서만 적용됩니다. 대화 상자의 다른 키는 고정되어 다시 바인딩할 수 없습니다: `Enter` 또는 `Tab`은 강조 표시된 일치 항목을 프롬프트 입력에 배치하고 `Esc`는 취소합니다.

### 작업 제어 작업 (Task actions)

`Task` 컨텍스트에서 사용할 수 있는 작업:

| 작업            | 기본값               | Description                                                                                                                                 |
| :---------------- | :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| `task:background` | Ctrl+B, Ctrl+X Ctrl+B | 현재 작업 백그라운드 전환. Ctrl+X Ctrl+B 핑거링은 tmux 접두사 충돌을 방지합니다 |

### 테마 작업 (Theme actions)

`ThemePicker` 컨텍스트에서 사용할 수 있는 작업:

| 작업                           | 기본값 | 설명                |
| :------------------------------- | :------ | :------------------------- |
| `theme:toggleSyntaxHighlighting` | Ctrl+T  | 구문 강조 토글 |

### 도움말 작업 (Help actions)

`Help` 컨텍스트에서 사용할 수 있는 작업:

| 작업         | 기본값 | 설명     |
| :------------- | :------ | :-------------- |
| `help:dismiss` | Escape  | 도움말 메뉴 닫기 |

### 탭 작업 (Tabs actions)

`Tabs` 컨텍스트에서 사용할 수 있는 작업:

| 작업          | 기본값         | 설명  |
| :-------------- | :-------------- | :----------- |
| `tabs:next`     | Tab, Right      | 다음 탭     |
| `tabs:previous` | Shift+Tab, Left | 이전 탭 |

### 첨부 파일 작업 (Attachments actions)

`Attachments` 컨텍스트에서 사용할 수 있는 작업:

| 작업                 | 기본값           | 설명                |
| :--------------------- | :---------------- | :------------------------- |
| `attachments:next`     | Right             | 다음 첨부 파일            |
| `attachments:previous` | Left              | 이전 첨부 파일        |
| `attachments:remove`   | Backspace, Delete | 선택한 첨부 파일 제거 |
| `attachments:exit`     | Down, Escape      | 첨부 파일 탐색 종료 |

### 바닥글 작업 (Footer actions)

`Footer` 컨텍스트에서 사용할 수 있는 작업:

| 작업                  | 기본값           | 설명                                                                                                                                                                                                               |
| :---------------------- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `footer:next`           | Right             | 다음 바닥글 항목                                                                                                                                                                                                          |
| `footer:previous`       | Left              | 이전 바닥글 항목                                                                                                                                                                                                      |
| `footer:up`             | Up                | 바닥글에서 위로 이동 (맨 위에서 선택 해제됨)                                                                                                                                                                                  |
| `footer:down`           | Down              | 바닥글에서 아래로 이동                                                                                                                                                                                                   |
| `footer:openSelected`   | Enter             | 선택한 바닥글 항목 열기                                                                                                                                                                                                 |
| `footer:clearSelection` | Escape            | 바닥글 선택 지우기                                                                                                                                                                                                    |
| `footer:dismiss`        | Backspace, Delete | 바닥글에서 선택한 [아티팩트](/docs/en/artifacts) 링크 닫기; 게시된 아티팩트 자체에는 영향이 없습니다. 다른 바닥글 행에서는 작동하지 않습니다 |

### 메시지 선택기 작업 (Message selector actions)

`MessageSelector` 컨텍스트에서 사용할 수 있는 작업:

| 작업                   | 기본값                                   | 설명       |
| :----------------------- | :---------------------------------------- | :---------------- |
| `messageSelector:up`     | Up, K, Ctrl+P                             | 목록에서 위로 이동   |
| `messageSelector:down`   | Down, J, Ctrl+N                           | 목록에서 아래로 이동 |
| `messageSelector:top`    | Ctrl+Up, Shift+Up, Meta+Up, Shift+K       | 맨 위로 점프       |
| `messageSelector:bottom` | Ctrl+Down, Shift+Down, Meta+Down, Shift+J | 맨 아래로 점프    |
| `messageSelector:select` | Enter                                     | 메시지 선택    |

### Diff 작업 (Diff actions)

`DiffDialog` 컨텍스트에서 사용할 수 있는 작업:

| 작업                | 기본값   | 설명                                                                                                                                         |
| :-------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| `diff:dismiss`        | Escape    | Diff 뷰어 닫기; 세부 보기에서는 대신 파일 목록으로 돌아갑니다                                                                           |
| `diff:previousSource` | Left      | 이전 diff 소스                                                                                                                                |
| `diff:nextSource`     | Right     | 다음 diff 소스                                                                                                                                    |
| `diff:previousFile`   | Up, K     | 파일 목록의 이전 파일; 세부 보기에서는 한 줄 위로 스크롤                                                                               |
| `diff:nextFile`       | Down, J   | 파일 목록의 다음 파일; 세부 보기에서는 한 줄 아래로 스크롤                                                                                 |
| `diff:viewDetails`    | Enter     | Diff 세부 정보 보기                                                                                                                                   |
| `diff:back`           | (미바인딩) | Diff 뷰어에서 뒤로 이동. Escape는 `diff:dismiss`를 통해 뒤로 가기 작업을 수행합니다. 이전 세부 보기의 기본값 Left는 v2.1.203에서 제거되었습니다 |

Diff 세부 보기도 페이저 스타일 키를 표준 [스크롤 작업](#scroll-actions)에 바인딩합니다. 이러한 바인딩은 `DiffDialog` 컨텍스트의 일부이며 세부 보기에서만 적용됩니다; [스크롤 작업](#scroll-actions) 아래 나열된 `Scroll` 컨텍스트 기본값은 변경되지 않습니다.

| 작업                | 기본값        | 설명                 |
| :-------------------- | :------------- | :-------------------------- |
| `scroll:pageUp`       | PageUp         | 뷰포트 절반 위로 스크롤   |
| `scroll:pageDown`     | PageDown       | 뷰포트 절반 아래로 스크롤 |
| `scroll:fullPageUp`   | Shift+Space, B | 전체 뷰포트 위로 스크롤   |
| `scroll:fullPageDown` | Space          | 전체 뷰포트 아래로 스크롤 |
| `scroll:top`          | G, Home        | 맨 위로 점프             |
| `scroll:bottom`       | Shift+G, End   | 맨 아래로 점프          |

### 모델 선택기 작업 (Model picker actions)

`ModelPicker` 컨텍스트에서 사용할 수 있는 작업:

| 작업                        | 기본값 | 설명                                  |
| :---------------------------- | :------ | :------------------------------------------- |
| `modelPicker:decreaseEffort`  | Left    | effort level 감소                        |
| `modelPicker:increaseEffort`  | Right   | effort level 증가                        |
| `modelPicker:thisSessionOnly` | s       | 강조 표시된 모델을 이 세션에만 적용 |

### 선택 작업 (Select actions)

`Select` 컨텍스트에서 사용할 수 있는 작업:

| 작업            | 기본값         | 설명      |
| :---------------- | :-------------- | :--------------- |
| `select:next`     | Down, J, Ctrl+N | 다음 옵션      |
| `select:previous` | Up, K, Ctrl+P   | 이전 옵션  |
| `select:accept`   | Enter           | 선택 수락 |
| `select:cancel`   | Escape          | 선택 취소 |

### 플러그인 작업 (Plugin actions)

`Plugin` 컨텍스트에서 사용할 수 있는 작업:

| 작업            | 기본값 | 설명                                                                |
| :---------------- | :------ | :------------------------------------------------------------------------- |
| `plugin:toggle`   | Space   | 플러그인 선택 토글                                                    |
| `plugin:install`  | I       | 선택한 플러그인 설치                                                   |
| `plugin:favorite` | F       | 선택한 플러그인을 즐겨찾기하여 Installed 탭 상단 근처에 정렬 |

### 설정 작업 (Settings actions)

`Settings` 컨텍스트에서 사용할 수 있는 작업. `select:accept` 및 `confirm:no` 작업은 설정 전용 동작과 함께 [Select](#select-actions) 및 [Confirmation](#confirmation-actions) 컨텍스트에서 재사용됩니다: 변경 사항은 변경하는 즉시 각 설정에 적용되므로 Escape는 변경 사항이 저장된 상태에서 거부 없이 패널을 닫습니다.

| 작업            | 기본값      | 설명                                     |
| :---------------- | :----------- | :---------------------------------------------- |
| `settings:search` | /            | 검색 모드 진입                               |
| `settings:retry`  | R            | 오류 발생 시 사용량 데이터 재로드 시도               |
| `select:accept`   | Enter, Space | 선택한 설정을 변경하거나 하위 메뉴 열기 |
| `confirm:no`      | Escape       | 패널 닫기. 변경 사항은 이미 저장되어 있음      |

### 음성 작업 (Voice actions)

[음성 구술](/docs/en/voice-dictation)이 활성화되어 있을 때 `Chat` 컨텍스트에서 사용할 수 있는 작업:

| 작업             | 기본값 | 설명                                              |
| :----------------- | :------ | :------------------------------------------------------- |
| `voice:pushToTalk` | Space   | 프롬프트 구술. `/voice` 모드에 따라 누르거나 탭 |

### 스크롤 작업 (Scroll actions)

[전체 화면 렌더링](/docs/en/fullscreen)이 활성화되어 있을 때 `Scroll` 컨텍스트에서 사용할 수 있는 작업:

| 작업                      | 기본값              | 설명                                                                                               |
| :-------------------------- | :------------------- | :-------------------------------------------------------------------------------------------------------- |
| `scroll:lineUp`             | (미바인딩)            | 한 줄 위로 스크롤. 마우스 휠 스크롤이 이 작업을 트리거함                                            |
| `scroll:lineDown`           | (미바인딩)            | 한 줄 아래로 스크롤. 마우스 휠 스크롤이 이 작업을 트리거함                                          |
| `scroll:pageUp`             | PageUp               | 뷰포트 높이의 절반 위로 스크롤                                                                        |
| `scroll:pageDown`           | PageDown             | 뷰포트 높이의 절반 아래로 스크롤                                                                      |
| `scroll:top`                | Ctrl+Home            | 대화 시작 위치로 점프                                                                     |
| `scroll:bottom`             | Ctrl+End             | 최신 메시지로 점프하고 자동 팔로우 재활성화                                                      |
| `scroll:halfPageUp`         | (미바인딩)            | 뷰포트 높이의 절반 위로 스크롤. `scroll:pageUp`과 동일한 동작, vi 스타일 바인딩용으로 제공       |
| `scroll:halfPageDown`       | (미바인딩)            | 뷰포트 높이의 절반 아래로 스크롤. `scroll:pageDown`과 동일한 동작, vi 스타일 바인딩용으로 제공   |
| `scroll:fullPageUp`         | (미바인딩)            | 전체 뷰포트 높이 위로 스크롤                                                                        |
| `scroll:fullPageDown`       | (미바인딩)            | 전체 뷰포트 높이 아래로 스크롤                                                                      |
| `selection:copy`            | Ctrl+Shift+C / Cmd+C | 선택한 텍스트를 클립보드에 복사                                                                   |
| `selection:clear`           | (미바인딩)            | 활성 텍스트 선택 지우기                                                                           |
| `selection:extendLeft`      | Shift+Left           | 활성 선택 영역을 왼쪽으로 한 열 확장                                                               |
| `selection:extendRight`     | Shift+Right          | 활성 선택 영역을 오른쪽으로 한 열 확장                                                              |
| `selection:extendUp`        | Shift+Up             | 활성 선택 영역을 위로 한 행 확장. 선택 영역이 상단 가장자리에 도달하면 뷰포트가 스크롤됨      |
| `selection:extendDown`      | Shift+Down           | 활성 선택 영역을 아래로 한 행 확장. 선택 영역이 하단 가장자리에 도달하면 뷰포트가 스크롤됨 |
| `selection:extendLineStart` | Shift+Home           | 활성 선택 영역을 줄 시작 위치까지 확장                                                      |
| `selection:extendLineEnd`   | Shift+End            | 활성 선택 영역을 줄 끝 위치까지 확장                                                        |

## 키 입력 구문 (Keystroke syntax)

### 조합 키 (Modifiers)

`+` 구분 기호와 함께 조합 키를 사용하세요:

* `ctrl` 또는 `control` - Control 키
* `shift` - Shift 키
* `alt`, `opt`, `option` 또는 `meta` - Windows 및 Linux에서는 Alt 키, macOS에서는 Option 키
* `cmd`, `command`, `super` 또는 `win` - macOS에서는 Command 키, Windows에서는 Windows 키, Linux에서는 Super 키

`cmd` 그룹은 Kitty 키보드 프로토콜이나 xterm의 `modifyOtherKeys` 모드를 지원하는 터미널처럼 Super 조합 키를 보고하는 터미널에서만 감지됩니다. 대부분의 터미널은 이를 전송하지 않으므로 모든 곳에서 작동하도록 하려면 `ctrl` 또는 `meta`를 사용하세요.

예시:

```text theme={null}
ctrl+k          Ctrl + K
shift+tab       Shift + Tab
meta+p          macOS에서는 Option + P, 다른 곳에서는 Alt + P
ctrl+shift+c    여러 조합 키
```

### 대문자 (Uppercase letters)

독립형 대문자는 Shift를 의미합니다. 예를 들어 `K`는 `shift+k`와 동일합니다. 이는 대문자와 소문자가 다른 의미를 갖는 vim 스타일 바인딩에 유용합니다.

조합 키가 포함된 대문자(예: `ctrl+K`)는 스타일상 표시된 것으로 간주되어 Shift를 의미하지 **않습니다**: `ctrl+K`는 `ctrl+k`와 동일합니다.

### 핑거링 (Chords)

핑거링은 공백으로 구분된 키 입력 시퀀스입니다:

```text theme={null}
ctrl+k ctrl+s   Ctrl+K를 누르고 뗀 다음 Ctrl+S 누름
```

### 특수 키 (Special keys)

* `escape` 또는 `esc` - Escape 키
* `enter` 또는 `return` - Enter 키
* `tab` - Tab 키
* `space` - 스페이스바
* `up`, `down`, `left`, `right` - 방향키
* `backspace`, `delete` - Delete 키

## 기본 단축키 바인딩 해제

기본 단축키의 바인딩을 해제하려면 작업을 `null`로 설정하세요:

```json theme={null}
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+s": null
      }
    }
  ]
}
```

이는 핑거링 바인딩에도 동일하게 작동합니다. 접두사를 공유하는 모든 핑거링의 바인딩을 해제하면 해당 접두사가 단일 키 바인딩으로 사용되도록 확보됩니다. 활성 컨텍스트의 핑거링은 접두사를 예약된 상태로 유지하므로 해당 컨텍스트에서 정의된 각 핑거링을 해제해야 합니다.

기본 `Ctrl+X` 계열은 `Chat`의 `ctrl+x ctrl+k` 및 `ctrl+x ctrl+e`, 그리고 `Task`의 `ctrl+x ctrl+b`라는 두 컨텍스트에 걸쳐 있습니다. `ctrl+x` 자체를 단일 키 바인딩으로 사용하려면 모든 바인딩을 해제하세요:

```json theme={null}
{
  "bindings": [
    {
      "context": "Task",
      "bindings": {
        "ctrl+x ctrl+b": null
      }
    },
    {
      "context": "Chat",
      "bindings": {
        "ctrl+x ctrl+k": null,
        "ctrl+x ctrl+e": null,
        "ctrl+x": "chat:newline"
      }
    }
  ]
}
```

접두사에서 일부 핑거링의 바인딩만 해제하는 경우 접두사를 누르면 나머지 바인딩에 대해 핑거링 대기 모드로 계속 진입합니다.

## 예약된 단축키 (Reserved shortcuts)

다음 단축키는 다시 바인딩할 수 없습니다:

| 단축키  | 이유                                         |
| :-------- | :--------------------------------------------- |
| Ctrl+C    | 하드코딩된 중단/취소                     |
| Ctrl+D    | 하드코딩된 종료                                 |
| Ctrl+M    | 터미널에서 Enter와 동일함 (둘 다 CR 전송) |
| Caps Lock | 터미널 애플리케이션으로 전달되지 않음         |

## 터미널 충돌 (Terminal conflicts)

일부 단축키는 터미널 멀티플렉서와 충돌할 수 있습니다:

| 단축키 | 충돌                          |
| :------- | :-------------------------------- |
| Ctrl+B   | tmux 접두사 (전송하려면 두 번 누름) |
| Ctrl+A   | GNU screen 접두사                 |
| Ctrl+Z   | Unix 프로세스 일시 중단 (SIGTSTP)    |

## Vim 모드 상호작용

`/config` → Editor mode를 통해 vim 모드가 활성화되면 키바인딩과 vim 모드가 독립적으로 작동합니다:

* **Vim 모드**는 텍스트 입력 수준에서 입력을 처리합니다 (커서 이동, 모드, 동작).
* **키바인딩**은 구성 요소 수준에서 작업을 처리합니다 (할 일 토글, 제출 등).
* Vim 모드의 Escape 키는 INSERT를 NORMAL 모드로 전환합니다; `chat:cancel`을 트리거하지 않습니다.
* 대부분의 Ctrl+키 단축키는 vim 모드를 지나 키바인딩 시스템으로 전달됩니다.
* Vim 키는 키바인딩 파일을 통해 다시 매핑할 수 없습니다. `jj`와 같은 2자 INSERT 모드 시퀀스를 Escape로 매핑하려면 [`vimInsertModeRemaps`](/docs/en/interactive-mode#remap-insert-mode-key-sequences) 설정을 사용하세요.
* Vim NORMAL 모드에서 `?`는 도움말 메뉴를 보여줍니다 (vim 동작).
* Vim NORMAL 모드에서 `/`는 표준 모드의 Ctrl+R과 동일하게 내역 검색을 엽니다.

## 검증 (Validation)

Claude Code는 키바인딩을 검증하고 다음에 대해 경고를 표시합니다:

* 파싱 오류 (유효하지 않은 JSON 또는 구조)
* 유효하지 않은 컨텍스트 이름
* 예약된 단축키 충돌
* 터미널 멀티플렉서 충돌
* 동일한 컨텍스트 내의 중복 바인딩

Claude Code는 파일이 로드될 때 경고를 보고하고 각 경고를 디버그 로그에 씁니다. 세부 정보를 보려면 [`--debug`](/docs/en/cli-reference#cli-flags)로 Claude Code를 시작하세요.
