> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# CLI에서 플러그인 추천하기 (Recommend your plugin from your CLI)

> CLI에서 한 줄 마커를 내보내 Claude Code가 사용자에게 공식 플러그인을 설치하도록 프롬프트를 표시하게 합니다.

CLI 또는 SDK를 유지 관리하고 있으며 공식 Anthropic 마켓플레이스에 플러그인이 있는 경우, 해당 도구가 Claude Code 사용자에게 해당 플러그인을 설치하도록 프롬프트를 표시할 수 있습니다. CLI는 Claude Code 내부에서 실행되고 있음을 감지할 때 stderr에 한 줄 마커를 작성합니다. Claude Code는 마커를 읽고 출력에서 이를 제거한 후 사용자에게 1회성 설치 프롬프트를 표시합니다.

Claude Code는 명령 출력이 모델에 전달되기 전에 힌트 줄을 제거하므로 마커가 대화에 절대 나타나지 않으며 토큰 사용량에 포함되지 않습니다. 이 프로토콜에는 추가 명령이 필요하지 않으며 Claude Code 외부에서 CLI가 사용자에게 출력하는 내용을 변경하지 않습니다.

이 페이지는 CLI 및 SDK 유지 관리자를 위한 것입니다. 플러그인을 설치하려면 [플러그인 탐색 및 설치](/docs/en/discover-plugins)를 참조하세요.

## 작동 방식

Claude Code는 Bash 및 PowerShell 도구를 통해 실행하는 모든 명령과 [훅(hook)](/docs/en/hooks) 명령에 대해 [`CLAUDECODE`](/docs/en/env-vars) 환경 변수를 `1`로 설정합니다. {/* min-version: 2.1.172 */}v2.1.172부터 동일한 하위 프로세스에서 [`CLAUDE_CODE_CHILD_SESSION`](/docs/en/env-vars)도 `1`로 설정합니다. CLI가 이러한 변수 중 하나를 발견하면 stderr에 자체 닫힘 `<claude-code-hint />` 태그를 씁니다. 훅 명령에서는 힌트 태그가 제거되고 무시됩니다. Bash 및 PowerShell 도구 출력만 설치 프롬프트를 트리거합니다.

Claude Code가 명령 출력을 받을 때 수행하는 작업:

1. 힌트 줄을 스캔하고 출력이 모델에 도달하기 전에 제거합니다.
2. 힌트가 공식 Anthropic 마켓플레이스의 플러그인을 타겟팅하는지 확인합니다.
3. 플러그인이 아직 설치되어 있지 않고 이전에 프롬프트가 표시되지 않았는지 확인합니다.
4. 힌트를 내보낸 명령의 이름을 지정하는 설치 프롬프트를 사용자에게 표시합니다.

Claude Code는 플러그인을 자동으로 설치하지 않습니다. 사용자가 항상 확인합니다.

## 힌트 내보내기

힌트 프롬프트는 공식 Anthropic 마켓플레이스에 나열된 플러그인에 대해서만 트리거됩니다. 통합을 출시하기 전에 [공식 마켓플레이스에 플러그인 등재하기](#get-your-plugin-into-the-official-marketplace)를 참조하세요.

사람이 CLI를 직접 실행할 때 마커가 나타나지 않도록 환경 변수로 게이트를 지정한 다음 자체 줄의 stderr에 태그를 쓰세요. 검사할 변수를 선택하세요:

* `CLAUDECODE`: 모든 Claude Code 버전에서 설정되므로 가장 많은 세션에 도달합니다. Claude Code가 시작하는 tmux 세션 및 stdio MCP 서버 하위 프로세스에도 설정됩니다. IDE 확장 프로그램도 사람이 CLI를 직접 실행할 수 있는 통합 터미널에서 이를 설정합니다.
* {/* min-version: 2.1.172 */}`CLAUDE_CODE_CHILD_SESSION`: 도구 호출, 훅 명령, [상태 줄](/docs/en/statusline) 명령과 같이 Claude Code 자체가 생성하는 하위 프로세스에서만 설정되므로 태그가 일반적으로 인간 터미널에 도달하지 않습니다. tmux 서버와 같이 세션 내부에서 시작된 오래 지속되는 프로세스는 변수를 캡처하므로 해당 프로세스에서 나중에 시작된 셸에는 여전히 원시 태그가 표시됩니다. Claude Code v2.1.172 이상이 필요하므로 이전 버전의 세션에서는 힌트가 누락됩니다.

다음 예시는 최대 도달 범위를 위해 `CLAUDECODE`로 게이트를 지정하고 공식 마켓플레이스에서 `example-cli`라는 플러그인에 대한 힌트를 내보냅니다:

<CodeGroup>
  ```javascript Node.js theme={null}
  if (process.env.CLAUDECODE) {
    process.stderr.write(
      '<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />\n',
    )
  }
  ```

  ```python Python theme={null}
  import os, sys

  if os.environ.get("CLAUDECODE"):
      print(
          '<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />',
          file=sys.stderr,
      )
  ```

  ```go Go theme={null}
  if os.Getenv("CLAUDECODE") != "" {
      fmt.Fprintln(os.Stderr,
          `<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />`)
  }
  ```

  ```shell Shell theme={null}
  if [ -n "$CLAUDECODE" ]; then
    printf '%s\n' '<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />' >&2
  fi
  ```
</CodeGroup>

`example-cli`를 공식 마켓플레이스에 있는 플러그인 이름으로 교체하세요.

## 내보낼 위치 선택

힌트를 내보낼 코드 경로를 제어합니다. Claude Code는 플러그인별로 중복을 제거하므로 모든 호출 시 내보내는 데 단점이 없습니다. 잘 작동하는 접점은 다음과 같습니다:

| 위치 | 작동하는 이유 |
| :--- | :--- |
| `--help` 출력 | Claude는 낯선 CLI를 탐색할 때 도움말을 자주 실행합니다 |
| 알 수 없는 하위 명령 오류 | 인터페이스에 대해 Claude가 혼란스러워하는 순간에 도달합니다 |
| 로그인 또는 인증 성공 | 사용자가 이미 설정 마인드셋에 있습니다 |
| 첫 실행 환영 메시지 | 자연스러운 온보딩 순간입니다 |

## 사용자가 보는 내용

힌트가 모든 검사를 통과하면 Claude Code에 다음과 같은 프롬프트가 표시됩니다:

```text theme={null}
─────────────────────────────────────────────────────────────
  Plugin recommendation

    The example-cli command suggests installing a plugin.

    Plugin: example-cli
    Marketplace: claude-plugins-official
    Official integration for example-cli deployments

    Would you like to install it?
    ❯ 1. Yes, install example-cli
      2. No
      3. No, and don't show plugin installation hints again

─────────────────────────────────────────────────────────────
```

프롬프트는 힌트를 생성한 명령의 이름을 지정하므로 사용자가 도구와 도구가 추천하는 플러그인 간의 불일치를 찾아낼 수 있습니다. 사용자가 30초 이내에 응답하지 않으면 프롬프트가 **No**로 닫힙니다.

프롬프트 빈도는 제한되어 있으며 일부 세션에서는 프롬프트가 전혀 표시되지 않습니다:

* **플러그인당 한 번**: 프롬프트가 표시된 후 Claude Code는 플러그인을 기록하고 사용자의 응답에 관계없이 해당 플러그인에 대한 프롬프트를 다시는 표시하지 않습니다.
* **세션당 한 번**: 머신의 모든 CLI에 걸쳐 Claude Code 세션당 최대 하나의 힌트 프롬프트가 나타납니다.
* **텔레메트리 옵트아웃**: 분석이 비활성화된 세션에서는 힌트 프롬프트가 표시되지 않습니다. 여기에는 `DISABLE_TELEMETRY` 또는 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`이 설정된 세션과 Amazon Bedrock 또는 Google Cloud Agent Platform과 같이 [자동 텔레메트리 옵트아웃](/docs/en/data-usage#default-behaviors-by-api-provider)이 적용되는 서드파티 공급자 세션이 포함됩니다.

**Yes**를 선택하면 사용자 범위에 플러그인이 설치됩니다. **No, and don't show plugin installation hints again**을 선택하면 사용자에 대해 향후 모든 힌트 프롬프트가 비활성화됩니다.

## 힌트 형식

힌트는 3개의 필수 속성이 있는 자체 닫힘 태그입니다.

```text theme={null}
<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />
```

| 속성 | 필수 여부 | 설명 |
| :--- | :--- | :--- |
| `v` | 예 | 프로토콜 버전. `1`이 지원되는 유일한 값입니다 |
| `type` | 예 | 힌트 종류. `plugin`이 지원되는 유일한 값입니다 |
| `value` | 예 | `name@marketplace` 형태의 플러그인 식별자 |

속성 값은 큰따옴표로 따옴표를 묶거나 따옴표 없이 남겨둘 수 있습니다. 따옴표가 없는 값에는 공백이 포함될 수 없습니다. 이스케이프 시퀀스는 지원되지 않습니다.

## 요구 사항

Claude Code는 힌트에 따라 조치를 취하기 전에 두 가지 조건을 강제 적용합니다. 어느 검사든 실패한 힌트는 삭제됩니다:

* **자체 줄**: 태그는 자체 줄을 차지해야 합니다. 예를 들어 로그 문 내부와 같이 줄 중간에 포함된 태그는 무시됩니다. 줄의 선두 및 미행 공백은 허용됩니다.
* **공식 마켓플레이스**: `value`는 `claude-plugins-official`과 같이 Anthropic이 제어하는 마켓플레이스의 플러그인을 참조해야 합니다. 다른 마켓플레이스를 가리키는 힌트는 자동으로 삭제됩니다.

버전이나 유형을 인식할 수 없는 경우에도 힌트 줄은 항상 모델에 도달하기 전에 출력에서 제거되므로 마커가 토큰 사용량에 카운트되지 않습니다.

남은 지침은 권장 사항이지만 강제 적용되지는 않습니다. Claude Code는 CLI가 이를 따르는지 여부를 관찰할 수 없습니다:

* **stderr에 쓰기**: stderr는 `example-cli deploy | jq`와 같은 셸 파이프라인에서 태그를 제외합니다. Claude Code는 두 스트림을 모두 스캔하므로 stdout도 작동합니다.
* **환경 변수로 게이트 지정**: `CLAUDECODE` 또는 `CLAUDE_CODE_CHILD_SESSION`이 설정된 경우에만 내보냅니다. 두 변수의 차이점은 [힌트 내보내기](#emit-the-hint)를 참조하세요.

## 공식 마켓플레이스에 플러그인 등재하기

힌트 프로토콜은 공식 Anthropic 마켓플레이스인 `claude-plugins-official`에 나열된 플러그인에 대해서만 효력을 발휘합니다. Anthropic은 재량에 따라 해당 마켓플레이스를 관리하며, 앱 내 제출 양식은 힌트 프로토콜이 검사하지 않는 [커뮤니티 마켓플레이스](/docs/en/plugins#submit-your-plugin-to-the-community-marketplace)에 플러그인을 추가합니다. Anthropic 파트너 담당자와 협력하는 경우 해당 담당자에게 연락하여 공식 마켓플레이스 등재를 조율하세요.

## 관련 정보

* [플러그인 만들기](/docs/en/plugins): CLI가 추천하는 플러그인 구축
* [플러그인 마켓플레이스 생성 및 배포](/docs/en/plugin-marketplaces): 공식 마켓플레이스 외부에서 플러그인 호스팅
* [환경 변수](/docs/en/env-vars): `CLAUDECODE` 및 관련 변수에 대한 전체 참조
