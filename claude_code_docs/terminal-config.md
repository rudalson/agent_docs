> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Claude Code를 위한 터미널 구성하기

> 줄바꿈을 위한 Shift+Enter 수정, Claude가 완료할 때 터미널 벨 받기, tmux 구성, 색상 테마 일치, Claude Code CLI에서 Vim 모드 활성화를 설정하세요.

Claude Code는 특별한 구성 없이도 모든 터미널에서 작동합니다. 이 페이지는 특정 기능이 예상대로 작동하지 않을 때 참조하기 위한 것입니다. 아래에서 관련 증상을 찾으세요. 이미 모든 것이 제대로 작동한다면 이 페이지가 필요하지 않습니다.

* [Shift+Enter를 누르면 줄바꿈 대신 제출됨](#enter-multiline-prompts)
* [macOS에서 Option 키 단축키가 작동하지 않음](#enable-option-key-shortcuts-on-macos)
* [Claude가 완료되어도 소리나 알림이 없음](#get-a-terminal-bell-or-notification)
* [tmux 내부에서 Claude Code를 실행함](#configure-tmux)
* [화면이 깜빡이거나 스크롤백 위치가 튀어 줌](#switch-to-fullscreen-rendering)
* [프롬프트에서 Vim 키를 사용하고 싶음](#edit-prompts-with-vim-keybindings)

이 페이지는 터미널이 Claude Code에 올바른 신호를 보내도록 설정하는 것에 관한 문서입니다. Claude Code 자체가 응답하는 키를 변경하려면 [keybindings](/docs/en/keybindings)를 참조하세요.

## 여러 줄 프롬프트 입력하기

Enter를 누르면 메시지가 제출됩니다. 제출하지 않고 줄바꿈을 추가하려면 Ctrl+J를 누르거나 `\`를 입력한 후 Enter를 누르세요. 둘 다 설정 없이 모든 터미널에서 작동합니다.

대부분의 터미널에서는 Shift+Enter를 누를 수도 있지만 지원 여부는 터미널 에뮬레이터에 따라 다릅니다:

| 터미널 | 줄바꿈을 위한 Shift+Enter |
| :---------------------------------------------------------------------- | :------------------------------------------ |
| Ghostty, Kitty, iTerm2, WezTerm, Warp, Apple Terminal, Windows Terminal | 설정 없이 작동함 |
| VS Code, Cursor, Devin Desktop, Alacritty, Zed | `/terminal-setup`을 한 번 실행하세요 |
| gnome-terminal, PyCharm 및 Android Studio와 같은 JetBrains IDE | 이용 불가. Ctrl+J 또는 `\` 입력 후 Enter 사용 |

VS Code, Cursor, Devin Desktop, Alacritty 및 Zed의 경우 `/terminal-setup`을 실행하면 Shift+Enter 및 기타 키 바인딩이 터미널 구성 파일에 작성됩니다. 처음 실행하면 `Installed VSCode terminal Shift+Enter key binding`과 같은 확인 메시지가 표시됩니다. 기존 바인딩은 그대로 유지되며, `VSCode terminal Shift+Enter key binding already configured`와 같은 메시지가 표시되면 변경되지 않은 것입니다. `/terminal-setup`은 호스트 터미널의 구성에 기록해야 하므로 tmux나 screen 내부가 아닌 호스트 터미널에서 직접 실행하세요.

VS Code, Cursor 및 Devin Desktop에서 `/terminal-setup`은 두 가지 에디터 설정도 업데이트합니다. 통합 터미널에서 텍스트가 깨지는 것을 방지하기 위해 `terminal.integrated.gpuAcceleration`을 `"off"`로 설정하고, [전체 화면 모드](/docs/en/fullscreen)에서 부드러운 스크롤을 위해 `terminal.integrated.mouseWheelScrollSensitivity`를 설정합니다. GPU 가속 변경을 되돌리려면 다시 `"auto"`로 설정하고 에디터 창을 다시 로드하세요.

tmux 내부에서 실행하는 경우 외부 터미널이 이를 지원하더라도 아래의 [tmux 구성](#configure-tmux)이 추가로 필요합니다.

줄바꿈을 다른 키에 바인딩하거나 Enter가 줄바꿈을 삽입하고 Shift+Enter가 제출하도록 동작을 바꾸려면 [키 바인딩 파일](/docs/en/keybindings)에서 `chat:newline` 및 `chat:submit` 작업을 매핑하세요.

## macOS에서 Option 키 단축키 활성화하기

일부 Claude Code 단축키(줄바꿈을 위한 Option+Enter 또는 모델 전환을 위한 Option+P 등)는 Option 키를 사용합니다. macOS에서 대부분의 터미널은 기본적으로 Option을 조합 키(modifier)로 보내지 않으므로 이를 활성화할 때까지 해당 단축키가 작동하지 않습니다. 이와 관련된 터미널 설정은 일반적으로 "Use Option as Meta Key"로 라벨이 지정되어 있습니다. Meta는 현재 Option 또는 Alt로 라벨이 지정된 키의 역사적인 Unix 이름입니다.

<Tabs>
  <Tab title="Apple Terminal">
    Settings → Profiles → Keyboard를 열고 "Use Option as Meta Key"를 체크합니다.

    Claude Code의 첫 실행 시 터미널 설정 프롬프트를 수락했다면 이 작업은 이미 완료된 것입니다. 해당 프롬프트는 Option을 Meta로 활성화하고 Apple Terminal 프로필에서 가음성 벨(audible bell)을 끄는 `/terminal-setup`을 사용자를 위해 실행합니다.

    {/* min-version: 2.1.211 */} [스크린 리더 모드](/docs/en/accessibility)에서 `/terminal-setup`은 벨 설정을 변경하지 않고 유지하여 터미널 벨이 계속 들리도록 합니다. v2.1.211 이전에는 스크린 리더 모드에서도 `/terminal-setup`이 벨을 껐습니다. 이전 실행으로 인해 벨이 꺼진 경우 Settings → Profiles → Advanced → "Audible bell"에서 다시 켜세요.
  </Tab>

  <Tab title="iTerm2">
    Settings → Profiles → Keys → General을 열고 Left Option key 및 Right Option key를 "Esc+"로 설정합니다.

    iTerm2에서 `/terminal-setup`을 실행하면 `/copy` 명령이 시스템 클립보드에 쓸 수 있도록 Settings → General → Selection 아래의 "Applications in terminal may access clipboard"가 활성화됩니다. 이 명령은 tmux 내부에서 실행되더라도 iTerm2를 감지합니다. 변경 사항을 적용하려면 iTerm2를 재시작하세요.
  </Tab>

  <Tab title="VS Code">
    VS Code 설정에 `"terminal.integrated.macOptionIsMeta": true`를 추가합니다.
  </Tab>
</Tabs>

Ghostty, Kitty 및 기타 터미널의 경우 터미널 구성 파일에서 Option-as-Alt 또는 Option-as-Meta 설정을 찾으세요.

## 터미널 벨 또는 알림 받기

Claude가 작업을 완료하거나 권한 프롬프트를 위해 일시 중지될 때 알림 이벤트를 발생시킵니다. 이를 터미널 벨이나 데스크톱 알림으로 표출하면 긴 작업이 실행되는 동안 다른 작업으로 전환할 수 있습니다.

기본적으로 Claude Code는 Ghostty, Kitty 및 iTerm2에서만 데스크톱 알림을 보냅니다. 다른 터미널에서는 [`preferredNotifChannel`](/docs/en/settings#available-settings)을 `"terminal_bell"`로 설정하여 대신 터미널 벨을 울리거나, 사용자 지정 소리 또는 명령을 위한 [Notification 훅](#play-a-sound-with-a-notification-hook)을 구성하세요. 다음 설정 항목은 터미널 벨을 켭니다:

```json ~/.claude/settings.json theme={null}
{
  "preferredNotifChannel": "terminal_bell"
}
```

데스크톱 알림은 SSH를 통해 로컬 머신에 도달하므로 원격 세션에서도 여전히 알림을 보낼 수 있습니다. Ghostty 및 Kitty는 추가 설정 없이 OS 알림 센터로 알림을 전달합니다. iTerm2에서는 전달을 활성화해야 합니다:

<Steps>
  <Step title="iTerm2 알림 설정 열기">
    Settings → Profiles → Terminal로 이동합니다.
  </Step>

  <Step title="알림 활성화">
    "Notification Center Alerts"를 체크한 다음 "Filter Alerts"를 클릭하고 "Send escape sequence-generated alerts"를 활성화합니다.
  </Step>
</Steps>

알림이 여전히 표시되지 않는 경우 OS 설정에서 터미널 애플리케이션에 알림 권한이 있는지 확인하고, tmux 내부에서 실행 중인 경우 [패스스루(passthrough)를 활성화](#configure-tmux)하세요.

### Notification 훅으로 소리 재생하기

모든 터미널에서 [Notification 훅](/docs/en/hooks-guide#get-notified-when-claude-needs-input)을 구성하여 Claude의 주의가 필요할 때 소리를 재생하거나 사용자 지정 명령을 실행할 수 있습니다. 훅은 내장 알림을 대체하는 대신 함께 실행되므로 Warp 또는 VS Code 통합 터미널과 같이 데스크톱 알림을 받지 못하는 터미널은 훅을 사용하거나 `preferredNotifChannel`을 `"terminal_bell"`로 설정할 수 있습니다.

아래 예시는 macOS에서 시스템 소리를 재생합니다. 링크된 가이드에는 macOS, Linux 및 Windows용 데스크톱 알림 명령이 포함되어 있습니다.

```json ~/.claude/settings.json theme={null}
{
  "hooks": {
    "Notification": [
      {
        "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff" }]
      }
    ]
  }
}
```

## tmux 구성하기

Claude Code가 tmux 내부에서 실행될 때 기본적으로 두 가지가 작동하지 않습니다. Shift+Enter가 줄바꿈 대신 제출되고, 데스크톱 알림 및 [진행률 표시줄](/docs/en/settings#available-settings)이 외부 터미널에 전혀 도달하지 않습니다. `~/.tmux.conf`에 다음 줄을 추가한 후 `tmux source-file ~/.tmux.conf`를 실행하여 실행 중인 서버에 적용하세요:

```bash ~/.tmux.conf theme={null}
set -g allow-passthrough on
set -s extended-keys on
set -as terminal-features 'xterm*:extkeys'
```

`allow-passthrough` 줄은 알림 및 진행률 업데이트가 tmux에 의해 삼켜지지 않고 외부 터미널에 도달할 수 있도록 합니다. `extended-keys` 줄은 tmux가 일반 Enter와 Shift+Enter를 구분하도록 하여 줄바꿈 단축키가 작동하게 만듭니다.

## 색상 테마 일치시키기

`/theme` 명령 또는 `/config`의 테마 선택기를 사용하여 터미널과 일치하는 Claude Code 테마를 선택하세요. 자동(auto) 옵션을 선택하면 터미널의 밝은(light) 또는 어두운(dark) 배경을 감지하므로 터미널이 변경될 때마다 테마가 OS 모양 변경을 따릅니다. Claude Code는 터미널 자체의 색상 구성표를 제어하지 않으며, 이는 터미널 애플리케이션에 의해 설정됩니다.

인터페이스 하단에 표시되는 내용을 커스텀하려면 현재 모델, 작업 디렉터리, git 브랜치 또는 기타 컨텍스트를 보여주는 [커스텀 상태 표시줄](/docs/en/statusline)을 구성하세요.

### 커스텀 테마 생성하기

<Note>
  커스텀 테마는 Claude Code v2.1.118 이상이 필요합니다.
</Note>

내장된 프리셋 외에도 `/theme`에는 정의한 커스텀 테마와 설치된 [플러그인](/docs/en/plugins-reference#themes)이 제공하는 테마가 나열됩니다. 목록 끝에서 **New custom theme…**을 선택하여 대화형으로 테마를 생성할 수 있습니다. 테마의 이름을 지정한 다음 오버라이드할 개별 색상 토큰을 선택합니다. 커스텀 테마가 하이라이트된 상태에서 `Ctrl+E`를 누르면 테마를 편집할 수 있습니다.

각 커스텀 테마는 `~/.claude/themes/`에 있는 JSON 파일입니다. `.json` 확장자를 제외한 파일 이름이 테마의 슬러그(slug)가 되며, 테마를 선택하면 테마 환경 설정으로 `custom:<slug>`가 저장됩니다. 이 파일에는 3개의 옵션 필드가 있습니다:

| 필드 | 유형 | 설명 |
| :---------- | :----- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| `name` | string | `/theme`에 표시되는 레이블. 기본값은 파일 이름 슬러그 |
| `base` | string | 테마가 시작되는 내장 프리셋: `dark`, `light`, `dark-daltonized`, `light-daltonized`, `dark-ansi`, 또는 `light-ansi`. 기본값은 `dark` |
| `overrides` | object | 색상 토큰 이름을 색상 값으로 매핑. 여기에 나열되지 않은 토큰은 기본 프리셋을 그대로 사용함 |

색상 값은 `#rrggbb`, `#rgb`, `rgb(r,g,b)`, `ansi256(n)`, 또는 `ansi:<name>`(`red`나 `cyanBright`와 같은 16가지 표준 ANSI 색상 이름 중 하나)을 수용합니다. 알 수 없는 토큰과 잘못된 색상 값은 무시되므로 오타가 렌더링을 망가뜨리지 않습니다.

다음 예시는 어두운(dark) 프리셋을 유지하면서 프롬프트 강조, 오류 텍스트, 성공 텍스트의 색상을 다시 지정하는 테마를 정의합니다:

```json ~/.claude/themes/dracula.json theme={null}
{
  "name": "Dracula",
  "base": "dark",
  "overrides": {
    "claude": "#bd93f9",
    "error": "#ff5555",
    "success": "#50fa7b"
  }
}
```

Claude Code는 `~/.claude/themes/`를 감시하고 파일이 추가되거나 변경되면 다시 로드하므로, 에디터에서 수정한 내용이 재시작 없이 실행 중인 세션에 적용됩니다. Claude Code가 시작할 때 `~/.claude/themes/` 폴더 자체가 존재하지 않았다면 첫 번째 테마 파일을 생성한 후 한 번 재시작하세요. 그 후에는 재시작 없이 변경 사항이 적용됩니다.

아래 참조는 `overrides`에 설정할 수 있는 토큰을 다룹니다. `/theme`의 대화형 에디터는 여기에 생략된 온보딩 화면 색상과 같은 일부 단일 목적 강조 항목과 함께 실시간 미리보기로 동일한 토큰을 보여줍니다.

<Accordion title="색상 토큰 참조">
  다음 예시는 브랜드 강조, 플랜 모드 테두리, diff 배경, 전체 화면 메시지 배경 등 아래의 여러 그룹 토큰을 조합합니다.

  ```json ~/.claude/themes/midnight.json theme={null}
  {
    "name": "Midnight",
    "base": "dark",
    "overrides": {
      "claude": "#a78bfa",
      "planMode": "#38bdf8",
      "diffAdded": "#14532d",
      "diffRemoved": "#7f1d1d",
      "userMessageBackground": "#1e1b4b"
    }
  }
  ```

  #### 텍스트 및 강조 색상

  인터페이스 전체에서 사용되는 주 브랜드 강조 색상과 전경 텍스트 음영을 제어합니다.

  | 토큰 | 제어 대상 |
  | :------------ | :--------------------------------------------------------------- |
  | `claude` | 주 브랜드 강조. 스피너 및 어시스턴트 레이블에 사용됨 |
  | `text` | 기본 전경 텍스트 |
  | `inverseText` | 상태 뱃지와 같이 색상이 지정된 배경 위에 그려지는 텍스트 |
  | `inactive` | 힌트, 타임스탬프, 비활성화된 항목과 같은 보조 텍스트 |
  | `subtle` | 희미한 테두리 및 강조가 해제된 보조 텍스트 |
  | `suggestion` | 자동 완성 제안 및 선택기에서의 선택 하이라이트 |
  | `permission` | 권한 프롬프트 및 선택기를 포함한 대화 상자 테두리 |
  | `remember` | 메모리 및 `CLAUDE.md` 표시기 |

  #### 상태 색상

  메시지 및 표시기 전반에 걸쳐 성공, 실패, 경고 상태를 알립니다.

  | 토큰 | 제어 대상 |
  | :-------- | :--------------------------------------------------- |
  | `success` | 성공 메시지 및 통과된 검사 |
  | `error` | 오류 메시지 및 실패 |
  | `warning` | 경고, 주의 메시지, 오토 모드 테두리 |
  | `merged` | 병합된 풀 리퀘스트 상태 |

  #### 입력 상자 및 모드 표시기

  입력 상자 테두리 색상과 권한 모드 또는 표시기가 활성화되어 있는 동안 표시되는 강조를 설정합니다.

  | 토큰 | 제어 대상 |
  | :------------- | :------------------------------------------------- |
  | `promptBorder` | 기본 권한 모드에서의 입력 상자 테두리 |
  | `planMode` | Plan 모드 강조 및 테두리 |
  | `autoAccept` | Accept-edits 모드 강조 및 테두리 |
  | `bashBorder` | `!` 쉘 명령을 입력할 때의 입력 상자 테두리 |
  | `ide` | IDE 연결 표시기 |
  | `fastMode` | Fast 모드 표시기 |

  #### Diff 렌더링

  파일 편집 및 리뷰에서 추가되거나 제거된 코드의 색상을 지정합니다.

  | 토큰 | 제어 대상 |
  | :------------------ | :------------------------------------------------- |
  | `diffAdded` | 추가된 줄의 배경 |
  | `diffRemoved` | 제거된 줄의 배경 |
  | `diffAddedDimmed` | 추가된 줄 근처의 변경되지 않은 컨텍스트 배경 |
  | `diffRemovedDimmed` | 제거된 줄 근처의 변경되지 않은 컨텍스트 배경 |
  | `diffAddedWord` | 추가된 줄 내 단어 수준 하이라이트 |
  | `diffRemovedWord` | 제거된 줄 내 단어 수준 하이라이트 |

  #### 전체 화면 모드

  메시지에 배경 채우기가 있는 [전체 화면 렌더링 모드](/docs/en/fullscreen)에만 적용됩니다.

  | 토큰 | 제어 대상 |
  | :--------------------------- | :------------------------------------------------------------ |
  | `userMessageBackground` | 트랜스크립트에서 사용자 메시지 뒤의 배경 |
  | `userMessageBackgroundHover` | 마우스를 올리거나 확장했을 때 메시지 뒤의 배경 |
  | `bashMessageBackgroundColor` | 트랜스크립트에서 `!` 쉘 명령 항목 뒤의 배경 |
  | `memoryBackgroundColor` | 트랜스크립트에서 `#` 메모리 항목 뒤의 배경 |
  | `selectionBg` | 마우스로 선택한 텍스트의 배경 |

  #### 사용량 미터 및 화자 레이블

  `/usage` 뷰에 표시되는 막대와 사용자 메시지 및 Claude 메시지를 구분하는 레이블을 조정합니다.

  | 토큰 | 제어 대상 |
  | :----------------- | :------------------------------------------------ |
  | `rate_limit_fill` | 사용량 미터의 채워진 부분 |
  | `rate_limit_empty` | 사용량 미터의 채워지지 않은 부분 |
  | `briefLabelYou` | 사용자 메시지의 `You` 레이블 색상 |
  | `briefLabelClaude` | 어시스턴트 메시지의 `Claude` 레이블 색상 |

  #### Shimmer 변형 및 서브에이전트 색상

  여러 토큰에는 스피너의 애니메이션 그라디언트에 사용되는 더 밝은 색상을 제공하는 페어링된 shimmer 변형이 있습니다. 애니메이션이 잘 맞지 않아 보이면 기본 토큰과 함께 shimmer를 오버라이드하세요.

  * `claude` 및 `claudeShimmer`
  * `warning` 및 `warningShimmer`
  * `permission` 및 `permissionShimmer`
  * `promptBorder` 및 `promptBorderShimmer`
  * `inactive` 및 `inactiveShimmer`
  * `fastMode` 및 `fastModeShimmer`

  각 [서브에이전트](/docs/en/sub-agents) 및 병렬 작업은 트랜스크립트에서 구분할 수 있도록 8가지 지정된 색상 중 하나로 표시됩니다. 토큰 이름은 `<color>_FOR_SUBAGENTS_ONLY` 패턴을 따르며, 여기서 `<color>`는 `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, 또는 `cyan`입니다. 해당 지정 색상이 어떻게 보일지 변경하려면 이 토큰들을 오버라이드하세요. 예를 들어 정의에서 `color: blue`인 서브에이전트는 `blue_FOR_SUBAGENTS_ONLY` 값을 사용하여 그려집니다.

  프롬프트 입력의 [`ultrathink`](/docs/en/model-config#use-ultrathink-for-one-off-deep-reasoning) 및 [`ultraplan`](/docs/en/ultraplan) 키워드는 7색 무지개 그라디언트로 렌더링됩니다. 토큰 이름은 `rainbow_<color>` 및 `rainbow_<color>_shimmer` 패턴을 따르며, 여기서 `<color>`는 `red`, `orange`, `yellow`, `green`, `blue`, `indigo`, 또는 `violet`입니다.
</Accordion>

## 전체 화면 렌더링으로 전환하기

[스크린 리더 모드](/docs/en/accessibility)에서는 이 섹션이 적용되지 않습니다. 연결된 [백그라운드 세션](/docs/en/agent-view)을 제외하고 Claude Code는 항상 일반 스크롤 텍스트로 렌더링되며, 다른 세션에서 `/tui fullscreen`을 실행하면 Claude Code는 전환하는 대신 설명을 출력합니다.

Claude가 작업하는 동안 화면이 깜빡이거나 스크롤 위치가 튀는 경우 [전체 화면 렌더링 모드](/docs/en/fullscreen)로 전환하세요. 이 모드는 일반 스크롤백에 덧붙이는 대신 터미널이 전체 화면 앱을 위해 예약해 둔 별도 화면에 그려지므로 메모리 사용량을 평평하게 유지하고 스크롤 및 선택을 위한 마우스 지원을 추가합니다. 이 모드에서는 터미널의 네이티브 스크롤백이 아닌 Claude Code 내부에서 마우스나 PageUp으로 스크롤합니다. 검색 및 복사 방법은 [전체 화면 페이지](/docs/en/fullscreen#search-and-review-the-conversation)를 참조하세요.

깜빡임이 유일한 문제이고 터미널이 동기화된 출력을 지원하지만 자동 감지되지 않는 경우(예: Emacs `eat`), 렌더러를 변경하지 않고 깜빡임을 중지하려면 [`CLAUDE_CODE_FORCE_SYNC_OUTPUT=1`](/docs/en/env-vars)을 설정하세요.

스위치를 전환하고 환경 설정을 저장하려면 `/tui fullscreen`을 실행하세요. 대화가 온전하게 다시 시작되고 이후 세션은 전체 화면에서 시작됩니다. Claude Code를 시작하기 전에 `CLAUDE_CODE_NO_FLICKER` 환경 변수를 설정할 수도 있습니다:

<CodeGroup>
  ```bash Bash and Zsh theme={null}
  CLAUDE_CODE_NO_FLICKER=1 claude
  ```

  ```powershell PowerShell theme={null}
  $env:CLAUDE_CODE_NO_FLICKER = "1"; claude
  ```

  ```json ~/.claude/settings.json theme={null}
  {
    "env": {
      "CLAUDE_CODE_NO_FLICKER": "1"
    }
  }
  ```
</CodeGroup>

## 대용량 콘텐츠 붙여넣기

프롬프트에 800자 이상 또는 2줄 이상을 붙여넣으면 Claude Code는 입력 상자를 사용 가능한 상태로 유지하기 위해 입력을 `[Pasted text #1 +120 lines]`와 같은 자리 표시자로 축소합니다. 제출할 때 전체 내용이 여전히 Claude에게 전송됩니다.

VS Code 통합 터미널은 매우 큰 붙여넣기 내용이 Claude Code에 도달하기 전에 문자를 삭제할 수 있으므로, 거기서는 파일 기반 워크플로를 사용하는 것을 권장합니다. 전체 파일이나 긴 로그와 같은 매우 큰 입력의 경우 콘텐츠를 파일에 작성하고 붙여넣는 대신 Claude에게 파일 읽기를 요청하세요. 이렇게 하면 대화 트랜스크립트가 읽기 쉽게 유지되고 Claude가 나중에 턴에서 경로로 파일을 참조할 수 있습니다.

## Vim 키 바인딩으로 프롬프트 편집하기

Claude Code에는 프롬프트 입력을 위한 Vim 스타일 편집 모드가 포함되어 있습니다. `/config` → Editor mode를 통해 또는 `~/.claude/settings.json`에서 [`editorMode`](/docs/en/settings#available-settings)를 `"vim"`으로 설정하여 활성화하세요. 이를 끄려면 Editor mode를 다시 `normal`로 설정하세요.

Vim 모드는 `hjkl` 탐색, `v`/`V` 선택, 텍스트 개체를 이용한 `d`/`c`/`y` 등 NORMAL 모드 및 VISUAL 모드 이동(motion) 및 연산자(operator)의 일부를 지원합니다. 전체 키 표는 [Vim editor mode reference](/docs/en/interactive-mode#vim-editor-mode)를 참조하세요.

Vim 이동은 키 바인딩 파일을 통해 다시 매핑할 수 없습니다. `jj`와 같은 2키 INSERT 모드 시퀀스를 Escape에 매핑하려면 사용자 설정에 [`vimInsertModeRemaps`](/docs/en/interactive-mode#remap-insert-mode-key-sequences)를 설정하세요.

표준 Vim과 달리 Enter를 누르면 INSERT 모드에서도 프롬프트가 제출됩니다. 대신 줄바꿈을 삽입하려면 NORMAL 모드에서 `o` 또는 `O`를 누르거나 Ctrl+J를 사용하세요.

## 참고 항목

* [Interactive mode](/docs/en/interactive-mode): 전체 키보드 단축키 참조 및 Vim 키 표
* [Keybindings](/docs/en/keybindings): Enter 및 Shift+Enter를 포함한 모든 Claude Code 단축키 다시 매핑
* [Fullscreen rendering](/docs/en/fullscreen): 전체 화면 모드에서의 스크롤, 검색, 복사에 대한 세부 정보
* [Hooks guide](/docs/en/hooks-guide): Linux 및 Windows용 추가 Notification 훅 예시
* [Troubleshooting](/docs/en/troubleshooting): 터미널 구성 외적인 문제 해결 방법
