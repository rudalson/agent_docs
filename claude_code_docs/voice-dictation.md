> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 음성 받아쓰기 (Voice dictation)

> 눌러서 녹음(hold-to-record) 또는 탭하여 녹음(tap-to-record) 음성 받아쓰기를 사용하여 Claude Code CLI에서 프롬프트를 음성으로 입력하세요.

Claude Code CLI에서 프롬프트를 타이핑하는 대신 음성으로 입력하세요. 음성이 프롬프트 입력창에 실시간으로 받아쓰기 텍스트로 변환되므로 동일한 메시지 내에서 음성과 타이핑을 섞어서 사용할 수 있습니다. `/voice`로 받아쓰기를 활성화한 다음, 말하는 동안 키를 누르고 있거나 한 번 탭하여 시작하고 다시 탭하여 전송하세요.

<Note>
  탭 모드는 Claude Code v2.1.116 이상이 필요합니다. `claude --version`으로 버전을 확인하세요.
</Note>

받아쓰기는 [agent view](/docs/en/agent-view#peek-and-reply)에서도 작동합니다. 백그라운드 세션에 받아쓰기하려면 디스패치 입력이나 픽 패널(peek-panel) 답변에 포커스가 있는 동안 PTT(push-to-talk) 키를 누르거나 탭하세요.

## 요구 사항

음성 받아쓰기는 녹음된 오디오를 수신 대기 방식으로 받아쓰기(transcription)를 위해 Anthropic 서버로 스트리밍합니다. 오디오는 로컬에서 처리되지 않습니다. 다음 조건을 모두 충족해야 합니다:

* **Claude.ai 계정**: 음성-텍스트 변환 서비스는 계정으로 인증할 때만 사용할 수 있으며 Claude Code가 Anthropic API 키 직접 사용, Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry를 사용하도록 구성된 경우에는 사용할 수 없습니다.
* **HIPAA 컴플라이언스가 활성화되지 않은 조직**: 이 제한이 적용되는 경우 `/voice`는 `Voice mode is disabled by your organization's policy`를 표시합니다.
* **로컬 마이크**: 음성 받아쓰기는 [Claude Code on the web](/docs/en/claude-code-on-the-web) 또는 SSH 세션과 같은 원격 환경에서는 작동하지 않습니다.
* **WSL에서 Claude Code를 실행하는 경우 WSLg**: WSLg는 Windows 10 또는 11의 Microsoft Store에서 설치할 때 WSL2에 포함됩니다. WSL1과 같이 WSLg를 사용할 수 없는 경우 네이티브 Windows에서 Claude Code를 대신 실행하세요.

받아쓰기는 Claude 메시지나 토큰을 소비하지 않으며 `/usage`에 표시되는 제한에 합산되지 않습니다. Anthropic이 데이터를 처리하는 방식은 [data usage](/docs/en/data-usage)를 참조하세요.

오디오 녹음은 macOS, Linux 및 Windows의 내장 네이티브 모듈을 사용합니다. Linux에서 네이티브 모듈을 로드할 수 없는 경우 Claude Code는 ALSA utils의 `arecord` 또는 SoX의 `rec`로 대체(fallback)합니다. 둘 다 사용할 수 없는 경우 `/voice`는 패키지 관리자용 설치 명령을 출력합니다.

Claude Code [VS Code extension](/docs/en/vs-code)도 동일한 Claude.ai 계정 요구 사항과 함께 음성 받아쓰기를 지원합니다. 마이크가 로컬 머신에 있고 확장 프로그램은 원격 호스트에서 실행되므로 SSH, Dev Containers 및 Codespaces를 포함한 VS Code Remote 세션에서는 사용할 수 없습니다.

## 음성 받아쓰기 활성화

받아쓰기를 활성화하려면 `/voice`를 실행하세요. 처음 활성화할 때 Claude Code는 마이크 검사를 실행합니다. macOS에서 이는 이전에 승인된 적이 없는 경우 터미널에 대한 시스템 마이크 권한 프롬프트를 트리거합니다.

```
/voice
Voice mode enabled (hold). Hold space to record. Dictation language: en (/config to change).
```

`/voice`는 선택적 모드 인수를 허용합니다:

| 명령어 | 효과 |
| :------------ | :-------------------------------------------- |
| `/voice` | 켜거나 끄기로 토글하며 현재 모드를 유지함 |
| `/voice hold` | [hold mode](#hold-to-record)로 활성화 |
| `/voice tap` | [tap mode](#tap-to-record-and-send)로 활성화 |
| `/voice off` | 비활성화 |

음성 받아쓰기는 세션 간에 유지됩니다. `/voice`를 실행하는 대신 [user settings file](/docs/en/settings)에서 직접 설정하세요:

```json theme={null}
{
  "voice": {
    "enabled": true,
    "mode": "tap"
  }
}
```

음성 받아쓰기가 활성화되어 있는 동안 프롬프트가 비어 있으면 입력 푸터에 `hold space to speak` 힌트가 표시됩니다. 힌트는 현재 `voice:pushToTalk` 바인딩을 반영하며 [dictation key를 다시 바인딩](#rebind-the-dictation-key)하면 업데이트됩니다. 힌트 텍스트는 두 모드에서 동일하며 [custom status line](/docs/en/statusline)이 구성되어 있으면 표시되지 않습니다.

받아쓰기는 두 모드 모두에서 코딩 어휘에 맞게 조정되어 있습니다. `regex`, `OAuth`, `JSON`, `localhost`와 같은 일반적인 개발 용어가 올바르게 인식되며 현재 프로젝트 이름과 git 브랜치 이름이 인식 힌트로 자동으로 추가됩니다.

## 누르고 녹음 (Hold to record)

Hold 모드는 PTT(push-to-talk) 방식입니다. 키를 누르고 있는 동안 녹음이 진행되고 놓으면 중지됩니다. 이것이 기본 모드입니다.

녹음을 시작하려면 `Space`를 누르고 계세요. Claude Code는 터미널로부터의 빠른 키 반복(key-repeat) 이벤트를 감시하여 눌려진 키를 감지하므로 녹음이 시작되기 전에 짧은 웜업(warmup)이 있습니다. 푸터는 웜업 동안 `keep holding…`을 표시한 다음 녹음이 활성화되면 라이브 파형(waveform)으로 전환됩니다.

첫 한두 개의 키 반복 문자는 웜업 중에 입력창에 입력되며 녹음이 활성화되면 자동으로 제거됩니다. 홀드 감지는 빠른 반복 시에만 트리거되므로 단일 `Space` 탭은 여전히 공백을 입력합니다.

<Tip>
  웜업을 건너뛰려면 `/voice tap`으로 [tap mode](#tap-to-record-and-send)로 전환하거나 `meta+k`와 같은 [조합 키(modifier combination)로 다시 바인딩](#rebind-the-dictation-key)하세요. 조합 키는 첫 번째 키 입력 시 바로 녹음을 시작합니다.
</Tip>

말하는 동안 음성이 프롬프트에 표시되며 텍스트가 확정(finalize)될 때까지 흐리게 표시됩니다. 녹음을 중지하고 텍스트를 확정하려면 `Space`를 놓으세요. 받아쓰기 결과는 커서 위치에 삽입되고 커서는 삽입된 텍스트 끝에 남아있으므로 어떤 순서로든 타이핑과 받아쓰기를 섞어서 사용할 수 있습니다. 다른 녹음을 덧붙이려면 `Space`를 다시 누르고 있거나 커서를 먼저 이동하여 프롬프트의 다른 위치에 음성을 삽입하세요:

```
> refactor the auth middleware to ▮
  # space를 누르고 있는 상태에서 "use the new token validation helper" 라고 발화
> refactor the auth middleware to use the new token validation helper▮
```

기본적으로 키를 놓으면 텍스트가 삽입되고 `Enter`를 누를 때까지 기다립니다. 받아쓰기가 최소 3단어 이상인 경우 키를 놓을 때 프롬프트를 자동으로 전송하려면 `voice` 설정 개체에 `"autoSubmit": true`를 설정하세요.

## 탭하여 녹음 및 전송 (Tap to record and send)

Tap 모드는 단일 키 입력으로 녹음을 토글합니다. 한 번 탭하여 시작하고, 말한 다음, 다시 탭하여 프롬프트를 전송합니다. 웜업이 없으며 키를 누르고 있을 필요가 없습니다.

`/voice tap`으로 Tap 모드를 활성화하세요. 프롬프트 입력이 비어 있는 상태에서 `Space`를 탭하여 녹음을 시작합니다. 녹음 중에는 푸터에 라이브 파형이 표시됩니다. 중지하려면 `Space`를 다시 탭하세요.

받아쓰기가 최소 3단어 이상인 경우 Claude Code가 텍스트를 삽입하고 프롬프트를 자동으로 제출합니다. 3단어 미만의 짧은 받아쓰기는 삽입되지만 제출되지 않으므로 실수로 탭하여 엉뚱한 단어가 전송되는 것을 방지합니다.

3단어 임계값은 공백 없이 작성되는 언어의 단어 수를 카운트합니다. v2.1.195부터 일본어, 중국어, 태국어 받아쓰기 결과는 개별 단어를 카운트하므로 Tap 모드 및 `autoSubmit`이 설정된 Hold 모드에서 자동으로 제출됩니다. 이전 버전에서는 공백이 없는 텍스트를 한 단어로 카운트하여 자동으로 제출하지 않았습니다.

첫 번째 탭은 프롬프트 입력이 비어 있을 때만 녹음을 시작하므로 메시지를 작성하는 동안 정상적으로 공백을 타이핑할 수 있습니다. 두 번째 탭은 입력 내용과 관계없이 녹음을 중지합니다. 녹음은 15초 동안 침묵이 흐르거나 총 2분이 지나면 자동으로 중지됩니다.

## 받아쓰기 언어 변경

음성 받아쓰기는 Claude의 응답 언어를 제어하는 동일한 [`language` setting](/docs/en/settings)을 사용합니다. 해당 설정이 비어 있으면 받아쓰기는 기본적으로 영어로 설정됩니다. VS Code 확장 프로그램에서 `language`가 비어 있으면 받아쓰기는 영어로 기본 설정되기 전에 VS Code의 `accessibility.voice.speechLanguage` 설정을 사용합니다.

<Accordion title="지원되는 받아쓰기 언어">
  | 언어 | 코드 |
  | :--------- | :--- |
  | Czech | `cs` |
  | Danish | `da` |
  | Dutch | `nl` |
  | English | `en` |
  | French | `fr` |
  | German | `de` |
  | Greek | `el` |
  | Hindi | `hi` |
  | Indonesian | `id` |
  | Italian | `it` |
  | Japanese | `ja` |
  | Korean | `ko` |
  | Norwegian | `no` |
  | Polish | `pl` |
  | Portuguese | `pt` |
  | Russian | `ru` |
  | Spanish | `es` |
  | Swedish | `sv` |
  | Turkish | `tr` |
  | Ukrainian | `uk` |
</Accordion>

`/config` 또는 설정에서 직접 언어를 설정하세요. [BCP 47 language code](https://en.wikipedia.org/wiki/IETF_language_tag) 또는 언어 이름을 사용할 수 있습니다:

```json theme={null}
{
  "language": "korean"
}
```

`language` 설정이 지원 목록에 없는 경우 `/voice`는 활성화 시 경고를 표시하고 받아쓰기에 대해 영어로 대체합니다. Claude의 텍스트 응답은 이 대체 동작의 영향을 받지 않습니다.

## 받아쓰기 키 다시 바인딩하기

받아쓰기 키는 `Chat` 컨텍스트의 `voice:pushToTalk`에 바인딩되어 있으며 기본값은 `Space`입니다. 동일한 바인딩이 Hold 및 Tap 모드를 모두 제어합니다. [`~/.claude/keybindings.json`](/docs/en/keybindings)에서 다시 바인딩하세요:

```json theme={null}
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "meta+k": "voice:pushToTalk",
        "space": null
      }
    }
  ]
}
```

`voice:pushToTalk` 작업은 한 번에 하나의 키를 사용합니다. 커스텀 키를 바인딩하면 두 번째 트리거를 추가하는 대신 기본 `Space` 바인딩을 대체하므로 이 예시의 `"space": null` 줄은 명확성을 위한 것이며 동작 변경 없이 생략할 수 있습니다.

Hold 모드에서는 홀드 감지가 키 반복에 의존하고 웜업 중에 해당 글자가 프롬프트에 입력되므로 `v`와 같은 단일 문자를 바인딩하는 것을 피하세요. `Space`를 사용하거나 `meta+k`와 같은 조합 키를 사용하여 웜업 없이 첫 번째 키 입력 시 바로 녹음을 시작하세요. Tap 모드에는 웜업이 없으므로 대부분의 키가 작동합니다.

일부 키는 터미널 애플리케이션에 전달되지 않으며 전혀 바인딩할 수 없습니다. 예를 들어 `Caps Lock`은 바인딩을 시도하면 오류가 표시됩니다. 전체 키 바인딩 구문 및 예약된 단축키 목록은 [customize keyboard shortcuts](/docs/en/keybindings)를 참조하세요.

## 문제 해결

음성 받아쓰기가 활성화되지 않거나 녹음되지 않을 때 발생하는 일반적인 문제:

* **`Voice mode requires a Claude.ai account`**: API 키 또는 서드파티 제공업체로 인증된 상태입니다. Claude.ai 계정으로 로그인하려면 `/logout` 및 `/login`을 실행하세요.
* **`Voice mode is disabled by your organization's policy`**: [요구 사항](#requirements)에 설명된 대로 조직의 컴플라이언스 구성에 의해 음성 받아쓰기가 비활성화되었습니다. 조직에서 음성 받아쓰기를 사용할 수 있는지 확인하려면 조직 관리자에게 문의하세요.
* **`Microphone access is denied`**: 시스템 설정에서 터미널에 마이크 권한을 부여하세요. macOS의 경우 시스템 설정 → 개인정보 보호 및 보안 → 마이크로 이동하여 터미널 앱을 활성화한 후 `/voice`를 다시 실행하세요. Windows의 경우 설정 → 개인 정보 및 보안 → 마이크로 이동하여 데스크톱 앱의 마이크 접근을 켠 다음 `/voice`를 다시 실행하세요. 터미널이 macOS 설정에 나열되어 있지 않은 경우 [Terminal not listed in macOS Microphone settings](#terminal-not-listed-in-macos-microphone-settings)를 참조하세요.
* **Linux에서 `No audio recording tool found`**: 네이티브 오디오 모듈을 로드할 수 없으며 대체 도구가 설치되어 있지 않습니다. 오류 메시지에 표시된 명령으로 SoX를 설치하세요(예: `sudo apt-get install sox`).
* **`Voice mode requires a microphone, but SoX could not open an audio capture device`**: SoX가 설치되어 있지만 헤드리스 서버나 컨테이너와 같이 호스트에 오디오 캡처 장치가 없습니다. 마이크가 있는 머신에서 Claude Code를 실행하세요. {/* min-version: 2.1.195 */} v2.1.195부터 Linux의 Claude Code는 해당 상황에서 이 메시지를 보고합니다. 이전 버전에서는 SoX가 이미 설치된 경우에도 SoX를 설치하도록 요청했습니다.
* **`Voice mode could not find a working audio recorder in WSL`**: WSLg는 ALSA 장치가 아닌 PulseAudio를 통해 오디오를 라우팅하므로 SoX의 PulseAudio 백엔드가 명시적으로 설치되어야 합니다. `sudo apt install sox libsox-fmt-pulse`를 실행하세요. `sox`만 설치하면 ALSA 백엔드가 가져와지며, 이는 `/dev/snd` 장치가 없기 때문에 WSL에서 녹음할 수 없습니다.
* **`Voice input is failing repeatedly and has been paused`**: 음성 받아쓰기가 연속으로 여러 번 캡처 실패에 부딪혀 성공할 때까지 새 세션을 시도하지 않고 중지되었습니다. 실패는 마이크 시작에 실패하든 레코더가 시작된 후 오디오를 생성하지 않고 중지하든 상관없이 카운트됩니다. 이는 일반적으로 헤드리스 서버, 오디오 통과가 없는 원격 쉘, 거부된 마이크 권한과 같이 이 호스트의 마이크나 오디오 스택이 오디오를 캡처할 수 없음을 의미합니다. 작동하는 입력 장치를 확인하고 위의 항목에서 근본 원인을 수정한 후 음성을 다시 트리거하세요. {/* min-version: 2.1.202 */} v2.1.202 이전에는 시작 실패만 일시 정지에 반영되었습니다.
* **Hold 모드에서 `Space`를 누르고 있어도 아무 일도 일어나지 않음**: 누르고 있는 동안 프롬프트 입력을 관찰하세요. 공백이 계속 누적되면 음성 받아쓰기가 꺼져 있을 가능성이 높습니다. `/voice hold`를 실행하여 활성화하세요. 공백이 한두 개만 표시되고 아무 반응이 없다면 음성 받아쓰기는 켜져 있지만 홀드 감지가 트리거되지 않는 것입니다. 홀드 감지는 터미널이 키 반복 이벤트를 보내야 하므로 OS 수준에서 키 반복이 비활성화되어 있으면 누려진 키를 감지할 수 없습니다. 키 반복 요구 사항을 피하려면 `/voice tap`으로 Tap 모드로 전환하세요.
* **Tap 모드에서 `Space`를 탭하면 녹음 대신 공백이 입력됨**: 첫 번째 탭은 프롬프트 입력이 비어 있을 때만 녹음을 시작합니다. 먼저 입력을 비우거나 `/voice tap`을 실행하여 Tap 모드에 있는지 확인하세요.
* **`No audio detected from microphone`**: 녹음이 시작되었으나 무음이 캡처되었습니다. 올바른 입력 장치가 시스템 기본값으로 설정되어 있는지, 입력 레벨이 음소거되거나 0에 가깝지 않은지 확인하세요. Windows에서는 설정 → 시스템 → 소리 → 입력으로 이동하여 마이크를 선택하세요. macOS에서는 시스템 설정 → 소리 → 입력으로 이동하세요.
* **`Voice connection failed`**: 연결 실패로 인해 녹음이 받아쓰기 서비스에 도달하지 못했습니다. 네트워크를 확인하고 다시 시도하세요. {/* min-version: 2.1.200 */} 오디오를 캡처하지 않는 녹음은 이 메시지 대신 `No audio detected from microphone`을 보고합니다. v2.1.200 이전에는 조용한 마이크가 연결 실패를 보고할 수 있어 실제 문제는 입력 장치임에도 네트워크 문제를 시사했습니다.
* **`No speech detected`**: 오디오가 받아쓰기 서비스에 도달했으나 단어가 인식되지 않았습니다. 마이크에 더 가깝게 말하고, 주변 소음을 줄이고, [dictation language](#change-the-dictation-language)가 말하는 언어와 일치하는지 확인하세요.
* **받아쓰기 결과가 깨지거나 잘못된 언어로 표시됨**: 받아쓰기는 기본적으로 영어로 설정됩니다. 다른 언어로 받아쓰기하는 경우 먼저 `/config`에서 설정하세요. [Change the dictation language](#change-the-dictation-language)를 참조하세요.

### macOS 마이크 설정에 터미널이 나열되지 않음

터미널 앱이 시스템 설정 → 개인정보 보호 및 보안 → 마이크에 표시되지 않는 경우 활성화할 수 있는 토글이 없는 상태입니다. 다음 `/voice` 실행이 새로운 macOS 권한 프롬프트를 트리거하도록 터미널의 권한 상태를 재설정하세요.

<Steps>
  <Step title="터미널에 대한 마이크 권한 재설정">
    `tccutil reset Microphone <bundle-id>`를 실행하되, `<bundle-id>`를 터미널의 식별자로 대체하세요: 내장 터미널의 경우 `com.apple.Terminal`, iTerm2의 경우 `com.googlecode.iterm2`. 다른 터미널의 경우 `osascript -e 'id of app "AppName"'`으로 식별자를 찾으세요.

    <Warning>
      번들 ID 없이 `tccutil reset Microphone`을 실행할 수도 있지만 Zoom이나 Slack과 같은 앱을 포함하여 Mac의 모든 앱에서 마이크 접근 권한이 취소됩니다. 각 앱은 다음에 사용할 때 접근 권한을 다시 요청해야 하므로 활성 통화 중에는 실행하지 마세요.
    </Warning>
  </Step>

  <Step title="터미널 종료 및 재실행">
    macOS는 이미 실행 중인 프로세스에 대해 프롬프트를 다시 표시하지 않습니다. 창을 닫는 것뿐만 아니라 Cmd+Q로 터미널 앱을 완전히 종료한 다음 다시 여세요.
  </Step>

  <Step title="새로운 프롬프트 트리거">
    Claude Code를 시작하고 `/voice`를 실행하세요. macOS가 마이크 접근을 요청하면 허용하세요.
  </Step>
</Steps>

## 참고 항목

* [Customize keyboard shortcuts](/docs/en/keybindings): `voice:pushToTalk` 및 기타 CLI 키보드 작업 다시 바인딩
* [Configure settings](/docs/en/settings): `voice`, `language` 및 기타 설정 키에 대한 전체 참조
* [Interactive mode](/docs/en/interactive-mode): 키보드 단축키, 입력 모드 및 세션 제어
* [Commands](/docs/en/commands): `/voice`, `/config` 및 기타 모든 명령 참조
