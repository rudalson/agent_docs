> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 권한 모드 선택 (Choose a permission mode)

> Claude가 파일을 편집하거나 명령을 실행하기 전에 물어볼지 여부를 제어합니다. CLI에서 Shift+Tab을 사용하여 모드를 순환하거나 VS Code, 데스크톱, claude.ai의 모드 선택기를 사용하세요.

Claude가 파일을 편집하거나 셸 명령을 실행하거나 네트워크 요청을 수행하려고 할 때 작업을 승인하도록 요청하며 일시 정지합니다. 권한 모드는 이러한 일시 정지가 얼마나 자주 발생하는지 제어합니다. 선택한 모드는 세션의 흐름을 결정합니다. 수동(Manual) 모드에서는 각 작업이 발생할 때마다 검토하는 반면, 느슨한 모드에서는 Claude가 끊김 없이 더 오래 작업하고 완료되면 다시 보고하도록 합니다. 민감한 작업에는 더 많은 감독을 선택하고, 방향을 신뢰할 때는 중단을 줄이세요.

## 사용 가능한 모드

각 모드는 편의성과 감독 간에 서로 다른 절충안을 제시합니다. 아래 표는 각 모드에서 권한 프롬프트 없이 실행되는 항목을 보여줍니다.

| 모드 | 물어보지 않고 실행되는 항목 | 최적의 용도 |
| :--- | :--- | :--- |
| `default` | 읽기 전용 | 시작하기, 민감한 작업 |
| [`acceptEdits`](#auto-approve-file-edits-with-acceptedits-mode) | 읽기, 파일 편집 및 일반 파일 시스템 명령(`mkdir`, `touch`, `mv`, `cp` 등) | 검토 중인 코드 반복 수정 |
| [`plan`](#analyze-before-you-edit-with-plan-mode) | 읽기 전용 | 코드베이스를 변경하기 전에 탐색 |
| [`auto`](#eliminate-prompts-with-auto-mode) | 모든 작업 (백그라운드 안전 검사 포함) | 장기 작업, 프롬프트 피로 감소 |
| [`dontAsk`](#allow-only-pre-approved-tools-with-dontask-mode) | 사전 승인된 도구만 허용 | 잠긴 CI 및 스크립트 |
| [`bypassPermissions`](#skip-all-checks-with-bypasspermissions-mode) | 모든 작업 | 격리된 컨테이너 및 VM 전용 |

모든 작업을 검토하는 모드는 CLI, `claude --help`, VS Code 및 JetBrains 확장 프로그램, 데스크톱 앱에서 **Manual(수동)**로 명명됩니다. 해당 구성 값은 `default`이며, 훅과 SDK 통합에서 사용하는 값입니다. CLI는 `claude --permission-mode manual` 또는 `"defaultMode": "manual"`과 같이 값을 입력하는 모든 곳에서 `manual`을 별칭으로 허용합니다. Manual 레이블 및 `manual` 별칭에는 Claude Code v2.1.200 이상이 필요합니다. 데스크톱 앱의 레이블은 CLI 버전에 의존하지 않습니다.

`bypassPermissions`를 제외한 모든 모드에서 [보호된 경로](#protected-paths)에 대한 쓰기는 절대 자동 승인되지 않으므로 리포지토리 상태와 Claude 자체 구성이 실수로 손상되는 것을 방지합니다.

모드는 기준을 설정합니다. 특정 도구를 사전 승인하거나 차단하려면 그 위에 [권한 규칙](/docs/en/permissions#manage-permissions)을 레이어링하세요. 이러한 제어 기능은 `bypassPermissions`를 포함한 모든 모드에 적용됩니다:

* 모든 도구에 적용되지만 다른 도구가 남아있는 동안 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 차단할 수 없는 거부(deny) 규칙 및 명시적 질문(ask) 규칙
* [커넥터 도구에 대한 조직 `ask` 설정](/docs/en/mcp#organization-controls-on-connector-tools)
* [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool) 마커

다른 모든 것이 이미 승인되어 있으므로 허용(allow) 규칙은 `bypassPermissions`에서 효과가 없습니다.

## 권한 모드 전환

세션 중간, 시작 시 또는 영구 기본값으로 모드를 전환할 수 있습니다. 모드는 채팅에서 Claude에게 물어보는 것이 아니라 이러한 제어 장치를 통해 설정됩니다. 변경 방법을 보려면 아래에서 인터페이스를 선택하세요.

<Tabs>
  <Tab title="CLI">
    **세션 중에**: `Shift+Tab`을 눌러 `default` → `acceptEdits` → `plan` 순으로 순환합니다. 상태 표시줄에는 활성 모드가 `⏸ plan mode on`, `⏵⏵ accept edits on`, `⏵⏵ auto mode on`, `⏵⏵ don't ask on`, `⏵⏵ bypass permissions on`으로 표시됩니다. {/* min-version: 2.1.203 */}수동 모드(해당 순환의 `default`)에는 회색 `⏸ manual mode on` 배지가 표시됩니다. v2.1.203 이전에는 수동 모드에서 상태 표시줄에 배지가 표시되지 않았습니다.

    모든 모드가 기본 순환에 포함되는 것은 아닙니다:

    * `auto`: 계정이 [자동 모드 요구 사항](#eliminate-prompts-with-auto-mode)을 충족할 때 나타납니다. 이 모드로 순환하면 확인 프롬프트 없이 모드가 전환됩니다.
    * `bypassPermissions`: `--permission-mode bypassPermissions`, `--dangerously-skip-permissions`, `--allow-dangerously-skip-permissions` 또는 [설정](/docs/en/settings#permission-settings)에서 `permissions.defaultMode: "bypassPermissions"`로 시작한 후 나타납니다. `--allow-` 변형은 모드를 활성화하지 않고 순환에 추가합니다.
    * `dontAsk`: 순환에 절대 나타나지 않으며, `--permission-mode dontAsk`로 설정합니다.

    활성화된 옵션 모드는 `plan` 뒤에 배치되며, `bypassPermissions`가 먼저 나오고 `auto`가 마지막에 나옵니다. 둘 다 활성화한 경우 `auto`로 가는 도중에 `bypassPermissions`를 순환하게 됩니다.

    **시작 시**: 플래그로 모드를 전달합니다.

    ```bash theme={null}
    claude --permission-mode plan
    ```

    **기본값으로**: `~/.claude/settings.json`과 같은 [설정 파일](/docs/en/settings#settings-files)에서 `defaultMode`를 설정합니다:

    ```json theme={null}
    {
      "permissions": {
        "defaultMode": "acceptEdits"
      }
    }
    ```

    동일한 `--permission-mode` 플래그는 [비대화형 실행](/docs/en/headless)을 위한 `-p`와 함께 사용할 수 있습니다.
  </Tab>

  <Tab title="VS Code">
    **세션 중에**: 프롬프트 상자 하단의 모드 표시기를 클릭합니다.

    **기본값으로**: VS Code 설정에서 `claudeCode.initialPermissionMode`를 설정하거나 Claude Code 확장 프로그램 설정 패널을 사용합니다.

    모드 표시기에는 적용되는 각 모드에 매핑된 다음 레이블이 표시됩니다:

    | UI 레이블 | 모드 |
    | :--- | :--- |
    | Manual | `default` |
    | Edit automatically | `acceptEdits` |
    | Plan | `plan` |
    | Auto | `auto` |
    | Bypass permissions | `bypassPermissions` |

    v2.1.205 이전에는 확장 프로그램에서 `plan`을 Plan 모드로, `auto`를 Auto 모드로 표시했습니다.

    자동 모드는 계정이 [자동 모드 섹션](#eliminate-prompts-with-auto-mode)에 나열된 모든 요구 사항을 충족할 때 모드 표시기에 나타납니다. `claudeCode.initialPermissionMode` 설정은 `auto`를 허용하지 않습니다. 기본적으로 자동 모드로 시작하려면 [사용자 설정](/docs/en/settings#settings-files)에서 `defaultMode`를 설정하세요. Claude Code는 프로젝트 및 로컬 설정의 `defaultMode: "auto"`를 무시합니다.

    권한 우회(Bypass permissions)는 모드 표시기에 나타나기 전에 확장 프로그램 설정에서 **Allow dangerously skip permissions** 토글이 필요합니다.

    확장 프로그램별 세부 정보는 [VS Code 가이드](/docs/en/vs-code)를 참조하세요.
  </Tab>

  <Tab title="JetBrains">
    JetBrains 플러그인은 IDE 터미널에서 Claude Code를 실행하므로 모드 전환은 CLI와 동일하게 작동합니다: `Shift+Tab`을 눌러 순환하거나 실행 시 `--permission-mode`를 전달합니다.
  </Tab>

  <Tab title="데스크톱">
    **세션 중에**: 전송 버튼 옆의 모드 선택기를 사용합니다. 선택기에 모든 모드가 나타나는 것은 아닙니다:

    * **Auto**: 계정이 [자동 모드 요구 사항](#eliminate-prompts-with-auto-mode)을 충족할 때 나타납니다.
    * **Bypass permissions**: Pro 및 Max 플랜의 데스크톱 설정에서 **Allow bypass permissions mode** 토글이 필요합니다. Team 및 Enterprise 플랜에서는 조직 정책이 대신 이를 제어합니다.

    데스크톱 전용 세부 정보는 데스크톱 가이드의 [권한 모드 선택](/docs/en/desktop#choose-a-permission-mode)을 참조하세요.

    **기본값으로**: [설정](/docs/en/settings#settings-files)에서 `defaultMode`를 설정합니다. 데스크톱 앱은 CLI와 동일한 설정 파일을 읽고 새 로컬 세션에 모드를 적용합니다.

    모드 선택기에서 선택한 모드는 폴더별로 기억되며 해당 폴더에 대해 `defaultMode`보다 우선 적용됩니다. Plan은 예외입니다. 이를 선택하면 현재 세션에만 적용됩니다.

    다음 예시는 새 로컬 세션의 기본값으로 Plan 모드를 설정합니다:

    ```json theme={null}
    {
      "permissions": {
        "defaultMode": "plan"
      }
    }
    ```
  </Tab>

  <Tab title="웹 및 모바일">
    [claude.ai/code](https://claude.ai/code) 또는 모바일 앱의 프롬프트 상자 옆에 있는 모드 드롭다운을 사용합니다. 승인을 위한 권한 프롬프트가 claude.ai에 나타납니다. 표시되는 모드는 세션이 실행되는 위치에 따라 다릅니다:

    * [웹에서의 Claude Code](/docs/en/claude-code-on-the-web)의 **클라우드 세션**: Accept edits, Plan 및 Auto. Accept edits는 `default` 모드에 대응합니다. 클라우드 환경은 모드에 관계없이 파일 편집을 사전 승인하므로 드롭다운에 Manual 대신 Accept edits가 표시됩니다. 클라우드 세션은 여전히 설정의 `defaultMode: "acceptEdits"`를 준수합니다. Auto 모드는 조직에서 허용하고 선택한 모델이 지원하는 경우에만 나타납니다. Bypass permissions는 사용할 수 없습니다.
    * 로컬 머신의 **[Remote Control](/docs/en/remote-control) 세션**: Manual, Accept edits 및 Plan. 앱에서 Auto 또는 Bypass permissions를 선택할 수 없습니다. {/* min-version: 2.1.202 */}드롭다운에는 터미널에서 설정한 모드를 포함하여 로컬 세션이 위치한 모드가 표시되며 앱이나 터미널에서 모드가 변경될 때 업데이트됩니다. 유일한 예외는 Bypass permissions입니다. 세션은 해당 모드를 claude.ai에 보고하지 않으므로 터미널에서 해당 모드로 전환해도 드롭다운 표시 내용은 변경되지 않습니다. v2.1.202 이전에는 `/remote-control` 또는 `claude --remote-control`로 연결된 세션이 모드를 전혀 보고하지 않아 claude.ai 및 모바일 앱에 세션이 위치하지 않은 모드가 표시될 수 있었습니다. 이러한 불일치는 레이블에만 영향을 미쳤습니다. Claude Code는 세션의 실제 모드에서 권한 프롬프트를 생성했으며 승인을 위해 앱에 나타났습니다.

    Remote Control의 경우 호스트가 claude.ai 계정으로 로그인되어 있어야 하며, API 키는 지원되지 않습니다. 호스트를 실행할 때 시작 모드를 설정할 수도 있습니다:

    ```bash theme={null}
    claude remote-control --permission-mode acceptEdits
    ```
  </Tab>
</Tabs>

## acceptEdits 모드로 파일 편집 자동 승인

`acceptEdits` 모드를 사용하면 Claude가 프롬프트 없이 작업 디렉터리에 파일을 생성하고 편집할 수 있습니다. 이 모드가 활성화되어 있는 동안 상태 표시줄에 `⏵⏵ accept edits on`이 표시됩니다.

파일 편집 외에도 `acceptEdits` 모드는 일반적인 파일 시스템 Bash 명령인 `mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed`를 자동 승인합니다. 이러한 명령은 `LANG=C` 또는 `NO_COLOR=1`과 같은 안전한 환경 변수나 `timeout`, `nice`, `nohup`과 같은 프로세스 래퍼가 앞에 붙은 경우에도 자동 승인됩니다. 파일 편집과 마찬가지로 자동 승인은 작업 디렉터리 또는 `additionalDirectories` 내부의 경로에만 적용됩니다. 해당 범위를 벗어난 경로, [보호된 경로](#protected-paths)에 대한 쓰기, [내장 읽기 전용 세트](/docs/en/permissions#read-only-commands)를 제외한 기타 모든 Bash 명령은 여전히 프롬프트를 표시합니다.

[PowerShell 도구](/docs/en/tools-reference#powershell-tool)가 활성화된 경우 `acceptEdits` 모드는 범위 내 경로에서 `Set-Content`, `Add-Content`, `Clear-Content`, `Remove-Item` 및 이들의 공통 별칭도 자동 승인합니다. 동일한 범위 및 보호된 경로 규칙이 적용됩니다.

인라인으로 각 편집을 승인하는 대신 나중에 에디터나 `git diff`를 통해 변경 사항을 검토하려는 경우 `acceptEdits`를 사용하세요.

수동 모드에서 `Shift+Tab`을 한 번 눌러 이 모드로 진입하거나 직접 시작하세요:

```bash theme={null}
claude --permission-mode acceptEdits
```

## plan 모드로 편집 전 분석

Plan 모드는 Claude에게 변경 사항을 적용하지 않고 조사하고 제안하도록 지시합니다. Claude는 파일을 읽고, 셸 명령을 실행하여 탐색하고, 계획을 작성하지만 소스를 편집하지는 않습니다. 권한 프롬프트는 [자동 모드](/docs/en/auto-mode-config)를 사용할 수 있고 기본값인 `useAutoModeDuringPlan`이 켜져 있지 않는 한 수동 모드와 동일하게 적용됩니다. 자동 모드가 활성화되면 분류기(classifier)가 프롬프트 없이 검색 및 파일 읽기와 같은 읽기 전용 명령을 승인합니다. 계획을 승인할 때까지는 어느 쪽이든 편집이 차단된 상태로 유지됩니다.

[내장 읽기 전용 세트](/docs/en/permissions#read-only-commands) 이외의 셸 명령(계획 수립 중 자동 모드가 활성화되어 있거나 샌드박스의 [자동 허용 모드](/docs/en/sandboxing#sandbox-modes)가 활성화되어 있는 경우를 포함하여 `touch` 및 `rm`과 같은 파일 수정 명령 포함)은 plan 모드에서 승인 프롬프트를 표시합니다.

`Shift+Tab`을 누르거나 단일 프롬프트 앞에 `/plan`을 붙여 plan 모드로 진입하세요. CLI에서 plan 모드로 시작할 수도 있습니다:

```bash theme={null}
claude --permission-mode plan
```

계획을 승인하지 않고 plan 모드를 종료하려면 `Shift+Tab`을 다시 누르세요.

### 계획 검토 및 승인

계획이 준비되면 Claude가 이를 제시하고 진행 방법을 묻습니다. 해당 프롬프트에서 다음을 선택할 수 있습니다:

* **Yes, and use auto mode**: 승인하고 [자동 모드](#eliminate-prompts-with-auto-mode)로 시작합니다. 자동 모드를 사용할 수 없는 경우 이 옵션은 **Yes, auto-accept edits**로 읽힙니다. 권한 우회가 활성화된 상태로 시작된 세션에는 대신 **Yes, and bypass permissions**가 표시됩니다.
* **Yes, manually approve edits**: 승인하고 각 편집을 개별적으로 검토합니다.
* **No, refine with Ultraplan on Claude Code on the web**: 브라우저 기반 검토를 위해 계획을 [Ultraplan](/docs/en/ultraplan)으로 전송합니다.
* **No, keep planning**: plan 모드에 남아 Claude에게 변경할 내용을 전달합니다.

계획을 승인하면 plan 모드가 종료되고 세션이 각 승인 옵션이 설명하는 권한 모드로 전환되므로 Claude가 편집을 시작합니다. 다시 계획하려면 `Shift+Tab`으로 plan 모드로 돌아가거나 다음 프롬프트 앞에 `/plan`을 붙이세요.

Claude가 진행하기 전에 기본 텍스트 에디터에서 제안된 계획을 열고 직접 편집하려면 `Ctrl+G`를 누르세요. [`showClearContextOnPlanAccept`](/docs/en/settings#available-settings)가 활성화되면 목록에 계획을 승인하고 계획 컨텍스트를 지우는 첫 번째 옵션이 추가됩니다.

계획을 승인하면 `--name` 또는 `/rename`으로 이미 이름을 설정하지 않은 한 계획 내용에서 세션 이름이 자동으로 지정됩니다.

### plan 모드를 기본값으로 설정

프로젝트의 기본값을 plan 모드로 설정하려면 `.claude/settings.json`에서 `defaultMode`를 설정하세요:

```json theme={null}
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

<h2 id="eliminate-prompts-with-auto-mode">
  auto 모드로 권한 프롬프트 제거
</h2>

자동 모드를 사용하면 일상적인 권한 프롬프트 없이 Claude가 실행할 수 있습니다. 별도의 분류기(classifier) 모델이 실행 전 작업을 검토하여 요청 수준을 넘어서거나, 인식되지 않는 인프라를 대상으로 하거나, Claude가 읽은 적대적 콘텐츠에 의한 것으로 보이는 작업을 차단합니다. 명시적인 [질문(ask) 규칙](/docs/en/permissions#manage-permissions)은 여전히 프롬프트를 강제합니다.

`rm -rf /` 및 `rm -rf ~`와 같이 파일 시스템 루트 또는 홈 디렉터리를 대상으로 하는 삭제 작업은 분류기로 이동하는 대신 승인 프롬프트를 표시합니다. {/* min-version: 2.1.208 */}이 프롬프트는 명령에 `$(...)`나 백틱을 사용한 명령 치환 또는 `<(...)`를 사용한 프로세스 치환이 포함되어 있을 때에도 실행됩니다(`echo "$(rm -rf ~)"`와 같이 치환 내부에 삭제가 위치하거나 동일한 명령의 다른 위치에 위치하는지 여부에 관계없음). v2.1.208 이전에는 이러한 형식을 포함하는 명령이 프롬프트를 표시하는 대신 분류기로 이동했습니다.

또한 자동 모드는 확인 질문을 위해 멈추지 않고 Claude가 계속 작업하도록 유도하지만, 프롬프트나 스킬이 이에 명시적으로 의존하는 경우 Claude는 여전히 질문합니다. 권한 프롬프트를 유지하면서 더 강력한 자율 동작을 원할 경우 대신 [주도적(Proactive) 출력 스타일](/docs/en/output-styles)을 설정하세요.

<Warning>
  자동 모드는 권한 프롬프트를 줄여주지만 안전을 보장하지는 않습니다. 민감한 작업에 대한 검토를 대체하는 것이 아니라 일반적인 방향을 신뢰하는 작업에 사용하세요.
</Warning>

자동 모드는 계정이 다음 모든 요구 사항을 충족하는 경우에만 사용할 수 있습니다:

* **플랜**: 모든 플랜.
* **소유자**: Team 및 Enterprise에서는 사용자가 켜기 전에 소유자가 [Claude Code 관리자 설정](https://claude.ai/admin-settings/claude-code)에서 이를 활성화해야 합니다. 관리자는 [관리형 설정](/docs/en/permissions#managed-settings)에서 `permissions.disableAutoMode`를 `"disable"`로 설정하여 자동 모드를 끌 수도 있습니다. 데스크톱 앱의 Code 탭의 경우 `disableAutoMode`가 조직 수준 제어 항목이며 관리자 설정 토글은 적용되지 않습니다.
* **모델**: Anthropic API 및 [AWS의 Claude Platform](/docs/en/claude-platform-on-aws)에서는 Claude Opus 4.6 이상, Sonnet 4.6 이상 또는 [Fable 5](/docs/en/model-config#work-with-fable-5). Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 및 로그인한 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션에서는 Claude Sonnet 5, Opus 4.7, Opus 4.8 및 Fable 5만 해당됩니다. Sonnet 4.5, Opus 4.5, Haiku, claude-3 모델을 포함한 이전 모델은 어떤 공급자에서도 지원되지 않습니다.
* **공급자**: Anthropic API, AWS의 Claude Platform, Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 및 로그인한 Claude 앱 게이트웨이 세션에서 기본적으로 사용할 수 있습니다. {/* min-version: 2.1.207 */}v2.1.158부터 v2.1.206까지는 `CLAUDE_CODE_ENABLE_AUTO_MODE=1`을 설정할 때까지 Anthropic API 및 AWS의 Claude Platform을 제외한 모든 공급자에서 자동 모드가 꺼져 있었습니다. v2.1.207에서 이 요구 사항이 제거되었습니다.

Claude Code가 자동 모드를 사용할 수 없다고 보고하면 이러한 요구 사항 중 하나가 충족되지 않은 것이며, 이는 일시적인 장애가 아닙니다. 모델 이름을 포함하고 자동 모드가 작업의 "안전성을 판단할 수 없다"고 하는 별도의 메시지는 일시적인 분류기 장애입니다. [오류 참조](/docs/en/errors#auto-mode-cannot-determine-the-safety-of-an-action)를 참조하세요.

[설정](/docs/en/settings#available-settings)에서 `defaultMode: "auto"`를 설정했지만 세션이 오류 없이 `default` 모드로 시작하는 경우 해당 설정이 `.claude/settings.json` 또는 `.claude/settings.local.json`에 있을 가능성이 높습니다. Claude Code v2.1.142 이상은 리포지토리가 자체적으로 자동 모드를 부여할 수 없도록 해당 파일의 `auto`를 무시합니다. 해당 설정을 `~/.claude/settings.json`으로 이동하세요.

<h3 id="enable-auto-mode-on-bedrock-agent-platform-or-foundry">
  Bedrock, Agent Platform 또는 Foundry에서의 auto 모드
</h3>

[Amazon Bedrock](/docs/en/amazon-bedrock), [Google Cloud Agent Platform](/docs/en/google-vertex-ai), [Microsoft Foundry](/docs/en/microsoft-foundry) 및 로그인한 [Claude 앱 게이트웨이](/docs/en/claude-apps-gateway) 세션에서 자동 모드는 기본적으로 `Shift+Tab` 순환에 나타납니다. 순환에 나타난다고 해서 세션이 시작되는 모드가 변경되지는 않습니다. 변경하지 않는 한 세션은 여전히 수동 모드인 [`defaultMode`](/docs/en/settings#available-settings)에서 시작됩니다. 이러한 공급자에서는 Claude Sonnet 5, Opus 4.7, Opus 4.8 및 Fable 5만 지원됩니다.

자동 모드를 기본 시작 모드로 만들려면 사용자 또는 관리형 설정에서 `"permissions": {"defaultMode": "auto"}`를 설정하세요.

{/* min-version: 2.1.210 */}[`/doctor`](/docs/en/commands#all-commands) 점검은 Anthropic API에서와 동일한 방식으로 이러한 공급자에서 사용자 설정 기본값을 제안합니다.

개발자가 자동 모드를 사용하지 못하도록 하려면 [관리형 설정](/docs/en/permissions#managed-settings)에서 `disableAutoMode`를 `"disable"`로 설정하세요. 이렇게 하면 `Shift+Tab` 순환에서 `auto`가 제거되고 시작 시 `--permission-mode auto`가 거부됩니다.

v2.1.158부터 v2.1.206까지는 `CLAUDE_CODE_ENABLE_AUTO_MODE=1`을 설정할 때까지 이러한 공급자에서 자동 모드가 꺼져 있었으며, Claude Code는 변수도 설정되지 않는 한 이러한 공급자에서 `defaultMode: "auto"`를 무시했습니다. 변수는 호환성을 위해 계속 수용되며 v2.1.207 이후부터는 효과가 없습니다.

### 분류기가 기본적으로 차단하는 항목

분류기는 세션이 시작될 때 구성된 원격 저장소와 작업 디렉터리를 신뢰합니다. {/* min-version: 2.1.200 */}세션 중에 `git remote add` 또는 `git remote set-url`로 추가되거나 다시 지정된 원격 저장소는 신뢰할 수 없으며 [신뢰할 수 있는 인프라를 구성](/docs/en/auto-mode-config)할 때까지 다른 모든 것은 외부 항목으로 취급됩니다. v2.1.200 이전에는 세션 중간에 추가된 원격 저장소도 신뢰되었습니다.

**기본적으로 차단됨**:

* `curl | bash`와 같은 코드 다운로드 및 실행
* 외부 엔드포인트로 민감한 데이터 전송
* 프로덕션 배포 및 마이그레이션
* 클라우드 저장소의 대량 삭제
* IAM 또는 리포지토리 권한 부여
* 공유 인프라 수정
* 세션 이전에 존재했던 파일의 비가역적 파괴
* 강제 푸시 (Force push)
* {/* min-version: 2.1.211 */}실행 시 비밀이나 민감한 데이터를 리포지토리 외부로 전송하거나 배포가 노출하는 범위를 넓히는 변경 사항의 커밋 또는 푸시. 이는 이미 수신하지 않은 대상에 비밀을 전달하는 CI 워크플로 또는 배포 구성, 비밀 저장소를 읽어 데이터를 외부로 전송하는 스크립트 또는 설정 단계, 레지스트리, 가시성, 아티팩트 또는 소스맵 설정과 같이 배포가 게시하는 범위를 넓히는 구성 변경을 포함합니다. 이 검사는 모든 브랜치에 적용되며 리포지토리가 공개 상태인 경우에도 적용되고, 해당 방류가 파이프라인을 트리거하는지 여부에 관계없이 변경 사항이 적용될 때 실행됩니다. 이를 통과하려면 커밋이나 푸시뿐만 아니라 실행 효과의 이름을 지정해야 합니다. v2.1.211 이전에는 이 검사가 기본 브랜치로 범위가 지정되었습니다. 즉, 민감한 내용, 요청한 내용에 비해 숨겨지거나 잘못 설명된 변경 사항, 리포지토리 외부에서 가져온 내용이 포함되어 있거나 요청한 검토를 우회한 경우 푸시가 차단되었으며 v2.1.203 이전에는 기본 브랜치에 대한 임의의 직접 푸시가 차단되었습니다.
* {/* min-version: 2.1.182 */}분류기가 커밋되지 않은 변경 사항을 삭제할 것으로 간주하는 `git reset --hard`, `git checkout -- .`, `git restore .`, `git clean -fd`, `git stash drop`, `git stash clear`
* HEAD의 커밋이 이 세션에서 생성되지 않았을 때의 `git commit --amend`
* {/* min-version: 2.1.198 */}v2.1.198부터 HEAD의 커밋이 이미 푸시되었을 때의 `git commit --amend`. Claude가 이 세션 중에 생성한 커밋에 대해 새로 스테이징된 내용이 없는 `--amend -m`과 같은 메시지 전용 문구 수정은 차단되지 않습니다.
* `terraform destroy`, `pulumi destroy`, `cdk destroy`, `terragrunt destroy` 및 리소스를 파괴하는 계획 적용

Claude Code v2.1.195 이상은 기본적으로 더 많은 범주를 차단합니다. 구체적인 이름으로 범위를 좁힐 수 있는 민감한 원격 대상 및 보호된 IaC 범위와 같은 구체적인 항목은 [환경(environment)](/docs/en/auto-mode-config#define-trusted-infrastructure) 항목에 의존합니다.

* 비밀 관리자에 쓰기, 또는 DNS 레코드나 TLS 인증서 변경
* 사람이 승인하지 않은 풀 리퀘스트 머지, Claude 자체의 풀 리퀘스트 승인, 또는 CI 검사 비활성화
* `atlantis apply`나 봇의 `/deploy` 또는 `/merge`와 같이 자동화에 대한 명령 자체인 댓글 게시
* 프로덕션 기능 플래그 전환, 램핑 또는 삭제
* 보호된 IaC 범위에 인프라 변경 사항 적용, 또는 클러스터 노드 비우기 및 제거
* 라벨 선택기나 다른 사용자의 작업을 포착하는 `--all`과 같이 지정한 리소스를 벗어나는 공유 컴퓨팅 클러스터에 대한 쓰기
* DaemonSet 및 어드미션 웹훅과 같이 모든 노드에서 실행되거나 클러스터 트래픽을 가로채는 Kubernetes 리소스 생성
* 민감한 원격 대상에 대한 대화형 셸 또는 포트 포워딩
* 공용 인터넷에서 로컬 서비스에 액세스할 수 있도록 하는 터널 또는 리버스 셸 열기
* 트랜스크립트나 파일에 라이브 자격 증명 또는 토큰 출력
* [환경](/docs/en/auto-mode-config#define-trusted-infrastructure)에서 민감한 데이터 위치로 나열된 위치에 접근하거나 해당 위치에서 데이터 복사. {/* min-version: 2.1.198 */}v2.1.198부터는 항목이 제외하는 대상에게 데이터를 전송하는 것도 차단됩니다.
* 내부 패키지 레지스트리를 우회하여 공용 레지스트리로 패키지 설치 라우팅. {/* min-version: 2.1.198 */}v2.1.198부터는 환경에 목록이 있는 경우뿐만 아니라 대화 중에 내부 레지스트리나 미러가 존재한다고 Claude에게 말한 경우에도 적용됩니다.
* `--insecure`와 같이 안전 가드를 해제하는 플래그로 명령 실행
* `--dangerously-skip-permissions` 또는 `--no-sandbox`로 시작된 것과 같이 사람의 승인이나 샌드박스 없이 실행되는 자율 에이전트 루프 실행. {/* min-version: 2.1.198 */}v2.1.198부터는 `--yes-always`로 시작된 러너와 같이 격리 및 작업별 승인이 비활성화된 서드파티 에이전트 또는 평가 하네스 실행도 포함됩니다.
* 페이지 내용, 쿠키 또는 자격 증명을 다른 출처로 보낼 수 있는 [Claude in Chrome](/docs/en/chrome) 브라우저 작업

Claude Code v2.1.198 이상은 기본적으로 다음 항목도 차단합니다:

* 특정 지정된 경로가 아닌 와일드카드, 글로브 또는 생성 시기 필터에 의해 `/tmp`, `$TMPDIR` 또는 기타 공유 임시/캐시 디렉터리의 파일 삭제
* 자체 메시지가 해당 수신자에 대해 해당 세부 정보를 승인하지 않은 경우, 전송, 업로드, 게시 또는 다른 사람이나 공유 시스템에 작성된 콘텐츠에 민감한 세부 정보 포함. {/* min-version: 2.1.200 */}조직의 자체 공개 리포지토리를 포함하여 리포지토리가 신뢰 경계 외부에 있거나 공개된 경우 PR 및 이슈 본문, 커밋 메시지, 댓글이 이러한 아웃바운드 콘텐츠에 해당합니다. 내부 파일 경로, 코드명, 이메일이나 계정 식별자와 같은 라이브 API 응답 데이터, 인프라 식별자가 민감한 세부 정보에 해당합니다. PR, 이슈, 커밋 메시지 범위 지정에는 Claude Code v2.1.200 이상이 필요합니다. {/* min-version: 2.1.203 */}PR이나 이슈 본문에 있는 이메일 주소, 계정 또는 조직 식별자, 사용량 메트릭과 같은 API 응답의 라이브 개인 데이터는 리포지토리의 가시성이나 신뢰 경계에 관계없이 해당 세부 정보와 수신자의 이름을 지정해야 합니다. 해당 검사에는 Claude Code v2.1.203 이상이 필요합니다.
* 자체 인터페이스를 구동하기 위해 Claude Code 자체의 tmux 창에 키 입력 전송(분류기는 이를 Claude가 자체 권한이나 감독을 변경하는 것으로 취급함)

Claude Code v2.1.200 이상은 기본적으로 다음 항목도 차단합니다:

* 인증, 액세스 제어, 입력 검증 또는 샌드박싱과 같은 보안 동작을 보호하는 테스트나 단정문(assertion) 주석 처리, 삭제 또는 강제 통과
* 더 구체적인 삭제 규칙이 적용되지 않고 해당 리소스를 지정하지 않은 경우 세션에서 Claude가 생성하지 않은 상태 저장 리소스 삭제 또는 제거
* 예제 파일(`.env.example` 등)을 포함하여 작업에 맞지 않는 서드파티 호스트에 API 기본 URL, 프록시 엔드포인트, 웹훅 수신기 또는 레지스트리 미러 다시 지정
* 새 원격 저장소의 이름을 지정하지 않은 경우 `git remote set-url` 또는 `git remote add`로 푸시 위치 변경
* 공개된 것으로 알려진 리포지토리에 비밀이나 개인/위탁 데이터를 푸시하거나, 해당 리포지토리의 자체 작업 일부가 아닌 기밀 자료를 거기에 푸시. {/* min-version: 2.1.203 */}닷파일(dotfiles) 리포지토리의 자체 주제는 개인 또는 위탁 데이터에 대한 한 가지 예외이며 비공개 리포지토리의 콘텐츠가 공개 영역에 도달하는 것도 동일한 방식으로 차단됩니다. 두 가지 개선 사항 모두 Claude Code v2.1.203 이상이 필요합니다. v2.1.203 이전에는 개인 데이터가 기밀 자료와 함께 그룹화되어 해당 리포지토리 자체 작업의 일부가 아닌 경우에만 차단되었습니다. 리포지토리의 가시성이 설정되지 않은 경우 분류기는 그것만으로 차단하지 않고 대신 다른 규칙에 따라 내용을 판단합니다.
* 외부 대상을 지정하지 않은 경우 다른 리포지토리나 조직을 향해 풀 리퀘스트 열기, `gh repo fork`로 포크하기, 서드파티 리포지토리에 푸시하기

Claude Code v2.1.203 이상은 기본적으로 다음 항목도 차단합니다:

* 민감한 로컬 저장소 또는 이름, 경로, 유형으로 민감함이 표시된 파일의 콘텐츠가 원천과 대상을 모두 지정하지 않는 한 커밋, 푸시, PR/이슈 텍스트, Gist/Paste 또는 패키지 게시에 포함되는 것. 세션 트랜스크립트 및 대화 로그, SSH 키, 클라우드 자격 증명, 브라우저 프로필, 셸 기록과 같은 자격 증명 및 구성 닷-폴더, 사용자 데이터 내보내기가 모두 포함되며 리포지토리가 비공개라는 사실로 해결되지 않습니다.

Claude Code v2.1.205 이상은 기본적으로 다음 항목도 차단합니다:

* 직접적으로든 셸 명령을 통해서든 `~/.claude/projects/` 아래의 `.jsonl` 기록 파일이나 구성된 설정 디렉터리에 있는 Claude Code 세션 트랜스크립트에 쓰기. 이 규칙은 Claude Code가 자체 검사를 위해 각 트랜스크립트 항목에 추가하는 메타데이터 줄도 포함합니다. 트랜스크립트는 작업 파일이 아니라 Claude Code가 작성하는 세션 상태이며, 변조된 항목은 세션을 재개하면 이후 모든 검사에 도달하므로 심층 방어 차원에서 자동 모드가 이러한 쓰기를 차단합니다. 트랜스크립트 읽기는 차단되지 않습니다.
* 분류기가 보는 대화의 어느 곳에서도 할당되지 않은 셸 변수를 대상으로 하는 `rm -rf "$VAR"` 또는 `Remove-Item -Recurse -Force $dir`와 같은 재귀적 강제 삭제, 또는 이를 루트로 하는 글로브. 값은 분류기가 수신하지 않는 이전 명령 출력에서만 나왔으므로 분류기는 다른 삭제 규칙에 대해 삭제 대상을 검증할 수 없습니다. 분류기는 의도적으로 명령 출력이 아닌 대화를 읽으므로 대상 추측 대신 호출을 차단합니다. 차단은 삭제되는 정확한 경로의 이름을 지정하거나 Claude가 명령에 작성된 확인된 리터럴 경로로 삭제를 다시 실행할 때 해제됩니다. 분류기가 확인할 수 있는 대상의 삭제는 영향을 받지 않습니다.

**기본적으로 허용됨**:

* 작업 디렉터리의 로컬 파일 작업
* 잠금 파일 또는 매니페스트에 선언된 종속성 설치
* `.env` 읽기 및 해당 API로 자격 증명 전송
* 읽기 전용 HTTP 요청
* {/* min-version: 2.1.211 */}기본 브랜치를 포함하여 작업 중인 리포지토리의 모든 브랜치에 푸시. `production`이나 `gh-pages`와 같이 이름에 배포 또는 게시 대상임이 표시된 비기본 브랜치는 포함되지 않습니다. 분류기는 해당 브랜치로의 푸시를 그 자체 조건에 따라 판단합니다. 푸시의 내용은 다른 규칙에 대해 여전히 검사되며, [`permissions.deny` 규칙](/docs/en/permissions#manage-permissions)은 모든 모드에서 특정 브랜치로의 푸시를 완전히 차단할 수 있으며 원격 자체의 브랜치 보호는 여전히 적용됩니다. v2.1.211 이전에는 시작한 브랜치, Claude가 생성한 브랜치, 기본 브랜치로의 일상적인 푸시만 기본적으로 허용되었으며 v2.1.203 이전에는 기본 브랜치에 대한 임의의 직접 푸시가 차단되었습니다.

Claude Code v2.1.195 이상은 기본적으로 다음 항목도 허용합니다:

* 동일한 세션에서 이전에 Claude가 생성한 정확한 작업 삭제
* 작업의 일부로 보안 관련 코드, 구성 및 위협 모델 읽기, 검토 또는 작성
* 동일한 다중 에이전트 세션에서 함께 작업하는 에이전트 간의 메시지
* [`environment`](/docs/en/auto-mode-config#define-trusted-infrastructure)에 나열한 신뢰할 수 있는 도메인, 버킷 및 서비스로 데이터 전송. 이는 데이터 흐름만 다루며 동일한 인프라에 대한 파괴적 작업이나 자격 증명 작업은 다루지 않습니다.
* 신뢰할 수 있는 내부 도메인, localhost 또는 지정한 URL로의 [Claude in Chrome](/docs/en/chrome) 탐색

샌드박스 네트워크 액세스 요청은 기본적으로 허용되지 않고 분류기를 통해 라우팅됩니다. {/* min-version: 2.1.198 */}v2.1.198부터 분류기는 모든 연결에 대해 다시 실행하는 대신 네트워크 호스트 및 포트에 대한 판정을 재사용합니다:

* 허용 항목은 대화에 새 콘텐츠가 입력될 때까지 재사용되며, 해당 시점에 해당 호스트가 다시 검사됩니다.
* 대화형 CLI에서는 턴이 끝날 때 거부 항목이 삭제됩니다.
* [비대화형 모드](/docs/en/headless) 및 Agent SDK 세션에는 턴 경계가 없으므로 남아있는 실행 시간 동안 거부 항목이 재사용됩니다.
* 권한 모드나 규칙을 변경하면 캐시된 모든 판정이 삭제됩니다.

전체 규칙 목록을 JSON으로 출력하려면 `claude auto-mode defaults`를 실행하세요. 일상적인 작업이 차단되면 관리자는 `autoMode.environment` 설정을 통해 신뢰할 수 있는 리포지토리, 버킷 및 서비스를 추가할 수 있습니다: [자동 모드 구성](/docs/en/auto-mode-config)을 참조하세요.

{/* min-version: 2.1.211 */}작업 중인 리포지토리의 모든 브랜치에 푸시하고 요청과 일치하는 풀 리퀘스트를 생성하는 것은 위 목록에서 다루는 두 가지 예외를 제외하고 프롬프트 없이 실행됩니다: 분류기는 `production`이나 `gh-pages`와 같은 배포 이름의 브랜치로의 푸시를 자체 조건에 따라 판단하며, 내용에 위험이 포함된 푸시는 여전히 차단합니다. 자동 모드를 유지하면서 이러한 작업 전에 사람의 확인 단계를 요구하려면 `permissions.ask` 규칙을 추가하세요: [일반적인 경계](/docs/en/auto-mode-config#common-boundaries)를 참조하세요.

### 대화에서 언급하는 경계

분류기는 대화에서 작성자가 언급하는 경계를 차단 시그널로 취급합니다. Claude에게 "푸시하지 마라" 또는 "배포하기 전에 내가 검토할 때까지 기다려라"라고 말하면 기본 규칙이 허용하더라도 분류기는 일치하는 작업을 차단합니다. 경계는 이후 메시지에서 해제할 때까지 유효하게 유지됩니다. 조건이 충족되었다는 Claude 자체의 판단으로 해제되지는 않습니다.

경계는 규칙으로 저장되지 않습니다. 분류기는 각 검사 시 트랜스크립트에서 경계를 다시 읽으므로 [컨텍스트 압축](/docs/en/costs#reduce-token-usage)으로 인해 경계를 설명한 메시지가 제거되면 경계가 손실될 수 있습니다. 확실한 보장을 위해서는 대신 [거부(deny) 규칙](/docs/en/permissions#permission-rule-syntax)을 추가하세요.

### auto 모드가 폴백(fallback)될 때

거부된 각 작업은 알림을 표시하며 `/permissions` 아래의 Recently denied 탭에 나타나며, 여기서 `r`을 눌러 수동 승인으로 재시도할 수 있습니다.

분류기가 연달아 3번 또는 총 20번 작업을 차단하면 자동 모드가 일시 정지되고 Claude Code가 프롬프트 표시를 재개합니다. 프롬프트가 표시된 작업을 승인하면 자동 모드가 재개됩니다. 이러한 임계값은 구성할 수 없습니다. 허용된 작업은 연속 카운터를 재설정하며, 총 카운터는 세션 동안 유지되고 자체 제한이 폴백을 트리거할 때만 재설정됩니다.

`-p` 플래그가 있는 [비대화형 모드](/docs/en/headless)에서는 프롬프트를 표시할 사용자가 없기 때문에 반복적인 차단이 발생하면 세션이 중단됩니다.

반복적인 차단은 일반적으로 분류기에 인프라에 대한 컨텍스트가 부족함을 의미합니다. 잘못된 탐지(오탐)를 보고하려면 `/feedback`을 사용하거나 관리자가 [신뢰할 수 있는 인프라를 구성](/docs/en/auto-mode-config)하도록 하세요.

<AccordionGroup>
  <Accordion title="분류기가 작업을 평가하는 방법">
    각 작업은 고정된 결정 순서를 거칩니다. 가장 먼저 일치하는 단계가 적용됩니다:

    1. [허용, 질문 또는 거부 규칙](/docs/en/permissions#manage-permissions)과 일치하는 작업은 즉시 해결됩니다. [보호된 경로](#protected-paths)에 대한 쓰기는 허용 규칙이 일치하더라도 분류기로 라우팅됩니다. [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 허용 규칙이 일치하더라도 사용자에게 직접 프롬프트를 표시합니다. 내용 범위의 질문 규칙은 권한 프롬프트로 폴백됩니다.
    2. 작업 디렉터리의 읽기 전용 작업 및 파일 편집은 [보호된 경로](#protected-paths)에 대한 쓰기를 제외하고 자동 승인됩니다.
    3. 그 외의 모든 작업은 분류기로 이동합니다. [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구는 분류기를 건너뛰고 사용자에게 직접 프롬프트를 표시하므로 조직에서 요구하는 승인은 절대 자동 승인되지 않습니다. {/* min-version: 2.1.199 */}v2.1.199부터 [`_meta["anthropic/requiresUserInteraction"]`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구도 분류기를 건너뛰고 직접 프롬프트를 표시하므로 도구 작성자를 대신한 동의 단계가 절대 자동 승인되지 않습니다.
    4. 분류기가 차단하는 경우 Claude는 사유를 받아 대안을 시도합니다.

    자동 모드에 들어갈 때 임의의 코드 실행을 허용하는 광범위한 허용 규칙이 해제됩니다:

    * 포괄적인 `Bash(*)` 또는 `PowerShell(*)`
    * `Bash(python*)`과 같이 와일드카드가 적용된 인터프리터
    * 패키지 관리자 실행 명령
    * `Agent` 허용 규칙

    `Bash(npm test)`와 같은 세부 규칙은 유지됩니다. 해제된 규칙은 자동 모드를 이탈할 때 복원됩니다.

    분류기는 사용자 메시지, 도구 호출, CLAUDE.md 내용을 봅니다. 도구 결과는 제거되므로 파일이나 웹 페이지의 적대적 콘텐츠가 분류기를 직접 조작할 수 없습니다. 별도의 서버 측 프로브가 들어오는 도구 결과를 스캔하고 Claude가 읽기 전에 의심스러운 콘텐츠에 플래그를 지정합니다. 이러한 레이어들이 함께 작동하는 방식에 대한 자세한 내용은 [자동 모드 발표](https://claude.com/blog/auto-mode) 및 [엔지니어링 딥 다이브](https://www.anthropic.com/engineering/claude-code-auto-mode)를 참조하세요.
  </Accordion>

  <Accordion title="auto 모드가 서브에이전트를 처리하는 방법">
    분류기는 세 시점에서 [서브에이전트](/docs/en/sub-agents) 작업을 검사합니다:

    1. 서브에이전트가 시작되기 전에 위임된 작업 설명이 평가되므로 위험해 보이는 작업은 생성(spawn) 시점에 차단됩니다.
    2. 서브에이전트가 실행되는 동안 각 작업은 부모 세션과 동일한 규칙에 따라 분류기를 거치며 서브에이전트의 프론트매터에 있는 `permissionMode`는 무시됩니다.
    3. 서브에이전트가 완료되면 분류기가 전체 작업 기록을 검토합니다. 이 반환 검사에서 우려 사항에 플래그가 지정되면 보안 경고가 서브에이전트 결과 앞에 추가됩니다.

    1단계에는 Claude Code v2.1.178 이상이 필요합니다. 이전 버전은 2단계와 3단계에서 분류기를 적용했지만 서브에이전트가 시작되기 전에 작업 설명을 평가하지는 않았습니다.
  </Accordion>

  <Accordion title="비용 및 지연 시간">
    {/* min-version: 2.1.210 */}분류기는 `/model` 선택이 아닌 기본적으로 Claude Sonnet 5에서 실행됩니다. Anthropic이 서버 측에서 구성하는 분류기 모델이 이 기본값보다 우선합니다. 세션 모델이 Claude Sonnet 4.6이거나 [`availableModels`](/docs/en/model-config#restrict-model-selection)에서 Sonnet 5를 제외하는 경우 분류기는 대신 세션 모델에서 실행되거나 세션이 [Fable 5](/docs/en/model-config#work-with-fable-5)에서 실행될 때 Opus 모델에서 실행됩니다. Anthropic API 이외의 공급자에서 해당 Opus 폴백은 공급자의 기본 Opus 모델입니다.

    세션의 첫 번째 자동 모드 요청은 Sonnet 5 기본값을 검증합니다. 요청이 성공하면 Sonnet 5가 세션의 분류기 모델로 유지되고, 모델을 사용할 수 없어 실패하면 세션은 대신 폴백을 사용합니다. 해당 검증이 결정된 후에는 세션에 대해 분류기의 모델이 변경되지 않습니다.

    분류기 호출은 토큰 사용량에 포함됩니다. 각 검사는 트랜스크립트의 일부와 보류 중인 작업을 전송하여 실행 전에 왕복을 추가합니다. 보호된 경로 이외의 읽기 및 작업 디렉터리 편집은 분류기를 건너뛰므로 오버헤드는 주로 셸 명령 및 네트워크 작업에서 발생합니다.

    {/* min-version: 2.1.198 */}분류기는 호스트 및 포트에 대한 샌드박스 네트워크 판정을 재사용하므로 동일한 호스트에 대한 반복 연결이 매번 검사를 추가하지 않습니다. [분류기가 기본적으로 차단하는 항목](#what-the-classifier-blocks-by-default)에서는 허용 및 거부의 유효 기간을 설명합니다.
  </Accordion>
</AccordionGroup>

## dontAsk 모드로 사전 승인된 도구만 허용

`dontAsk` 모드를 설정하면 Claude Code는 프롬프트를 표시할 수 있는 모든 도구 호출을 자동으로 거부합니다. Claude는 `permissions.allow` 규칙, [읽기 전용 Bash 명령](/docs/en/permissions#read-only-commands), [PreToolUse 훅](/docs/en/permissions#extend-permissions-with-hooks)에 의해 승인된 호출과 일치하는 작업만 실행합니다. Claude가 수행할 수 있는 작업을 정확히 사전 정의하는 CI 파이프라인이나 제한된 환경에서 이 모드를 사용하세요. 세션은 입력을 기다리지 않습니다. 이 모드가 활성화되어 있는 동안 상태 표시줄에 `⏵⏵ don't ask on`이 표시됩니다.

Claude Code는 프롬프트를 표시하는 대신 명시적인 [`ask` 규칙](/docs/en/permissions#manage-permissions)과 일치하는 호출을 거부합니다. 또한 허용 규칙이 일치하더라도 내장 `AskUserQuestion` 도구 및 [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구를 거부합니다. {/* min-version: 2.1.199 */}마찬가지로 [`_meta["anthropic/requiresUserInteraction"]`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구도 승인 카드에 이 모드가 수집하지 않는 답변이 필요하기 때문에 거부합니다. 여기에 Claude Code v2.1.199 이상이 필요합니다.

[웹에서의 Claude Code](/docs/en/claude-code-on-the-web)의 클라우드 세션은 `defaultMode: "dontAsk"`를 무시합니다. 자세한 내용은 [bypassPermissions](#skip-all-checks-with-bypasspermissions-mode)를 참조하세요.

시작 시 플래그로 설정하세요:

```bash theme={null}
claude --permission-mode dontAsk
```

## bypassPermissions 모드로 모든 검사 건너뛰기

`bypassPermissions` 모드는 권한 프롬프트 및 안전 검사를 비활성화하므로 [보호된 경로](#protected-paths)에 대한 쓰기를 포함하여 도구 호출이 즉시 실행됩니다. v2.1.126 이전에는 보호된 경로 쓰기가 이 모드에서도 프롬프트를 표시했습니다.

명시적인 [질문(ask) 규칙](/docs/en/permissions#manage-permissions) 및 [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구는 이 모드에서도 프롬프트를 강제합니다. {/* min-version: 2.1.199 */}마찬가지로 [`_meta["anthropic/requiresUserInteraction"]`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구도 프롬프트를 강제합니다. 여기에 Claude Code v2.1.199 이상이 필요합니다.

`rm -rf /` 및 `rm -rf ~`와 같이 파일 시스템 루트 또는 홈 디렉터리를 대상으로 하는 삭제 작업은 모델 오류에 대한 서킷 브레이커로서 여전히 프롬프트를 표시합니다. {/* min-version: 2.1.208 */}서킷 브레이커는 `echo "$(rm -rf ~)"`와 같이 치환 내부에 삭제가 위치하거나 동일한 명령의 다른 위치에 위치하는지 여부에 관계없이 명령에 `$(...)`나 백틱을 사용한 명령 치환 또는 `<(...)`를 사용한 프로세스 치환이 포함되어 있을 때에도 실행됩니다. 고유 명령으로 작성된 일반적인 형태는 서킷 브레이커가 도입된 이래 이 모드에서 프롬프트를 표시했습니다. v2.1.208 이전에는 이러한 형식을 포함하는 명령이 프롬프트를 표시하지 않았습니다.

<Warning>
  Claude Code가 호스트 시스템을 손상시킬 수 없는 컨테이너, VM, 인터넷 액세스가 없는 개발 컨테이너와 같이 격리된 환경에서만 이 모드를 사용하세요.
</Warning>

이 모드가 활성화되지 않고 시작된 세션에서는 `bypassPermissions`로 진입할 수 없습니다. 시작 시 [설정](/docs/en/settings#permission-settings)의 `permissions.defaultMode: "bypassPermissions"` 또는 활성화 플래그를 사용하여 이를 활성화하세요:

```bash theme={null}
claude --permission-mode bypassPermissions
```

`--dangerously-skip-permissions` 플래그도 동일합니다.

이 모드가 활성화된 대화형 세션을 처음 시작할 때 Claude Code는 권한 검사 없이 취해진 조치에 대한 책임을 수용할 것을 요청하는 경고 대화 상자를 표시합니다. Claude Code는 수용 여부를 사용자 설정에 저장하므로 대화 상자는 한 번만 나타납니다. 거부하면 Claude Code가 종료됩니다. [비대화형 모드](/docs/en/headless)에서는 대화 상자가 표시되지 않으며 대화형 세션에서 대화 상자를 수용할 때까지 `--bg`로 시작된 [백그라운드 세션](/docs/en/agent-view)이 거부됩니다.

Linux 및 macOS에서 Claude Code는 root 또는 `sudo` 아래에서 실행 중일 때 이 모드로 시작되는 것을 거부합니다:

```text theme={null}
--dangerously-skip-permissions cannot be used with root/sudo privileges for security reasons
```

인식된 샌드박스 내부에서는 검사가 자동으로 건너뛰어집니다. 컨테이너에서 자율적으로 실행하려면 비 root 사용자로 Claude Code를 실행하는 [dev container](/docs/en/devcontainer) 구성을 사용하세요.

[웹에서의 Claude Code](/docs/en/claude-code-on-the-web)는 설정 파일의 `defaultMode: "bypassPermissions"` 또는 `"dontAsk"`를 준수하지 않으므로 리포지토리의 체크인된 설정으로 권한 우회 모드의 클라우드 세션을 시작할 수 없습니다. 이 설정은 자동으로 무시되며 세션은 대신 모드 드롭다운에 표시된 모드로 시작됩니다. 클라우드 세션이 제공하는 모드는 [권한 모드 전환](#switch-permission-modes)을 참조하세요.

<Warning>
  `bypassPermissions`는 프롬프트 주입이나 의도하지 않은 작업에 대한 보호를 제공하지 않습니다. 권한 프롬프트가 훨씬 적은 백그라운드 안전 검사의 경우 대신 [자동 모드](#eliminate-prompts-with-auto-mode)를 사용하세요. 관리자는 [관리형 설정](/docs/en/permissions#managed-settings)에서 `permissions.disableBypassPermissionsMode`를 `"disable"`로 설정하여 이 모드를 차단할 수 있습니다.
</Warning>

## 보호된 경로

적은 수의 경로에 대한 쓰기는 `bypassPermissions`를 제외한 모든 모드에서 절대 자동 승인되지 않습니다. 이렇게 하면 리포지토리 상태와 Claude 자체 구성이 실수로 손상되는 것을 방지합니다.

| 모드 | 보호된 경로 쓰기 |
| :--- | :--- |
| `default`, `acceptEdits`, `plan` | 프롬프트 표시 |
| `auto` | 분류기로 라우팅 |
| `dontAsk` | 거부됨 |
| `bypassPermissions` | 허용됨 |

설정 파일의 [`permissions.allow`](/docs/en/permissions#manage-permissions) 규칙은 보호된 경로 쓰기를 사전 승인하지 않습니다. 안전 검사는 Claude Code가 설정의 허용 규칙을 평가하기 전에 실행되므로 `~/.claude/settings.json` 또는 `.claude/settings.json`에 `Edit(.claude/**)`와 같은 항목이 있어도 위 표의 모드별 결과가 변경되지 않습니다. 프롬프트를 표시하는 모드에서 `.claude/` 쓰기에 대한 프롬프트는 **Yes, and allow Claude to edit its own settings for this session**을 제공하여 다시 묻지 않고 해당 세션에서 이후의 `.claude/` 쓰기를 승인합니다.

보호된 디렉터리:

* `.git`
* `.config/git`
* `.vscode`
* `.idea`
* `.husky`
* `.cargo`
* `.devcontainer`
* `.yarn`
* `.mvn`
* `.claude` (Claude가 자체 git worktree를 저장하는 `.claude/worktrees` 제외)

보호된 파일:

* `.gitconfig`, `.gitmodules`
* `.bashrc`, `.bash_profile`, `.bash_login`, `.bash_aliases`, `.bash_logout`, `.zshrc`, `.zprofile`, `.zshenv`, `.zlogin`, `.zlogout`, `.profile`, `.envrc`
* `.npmrc`, `.yarnrc`, `.yarnrc.yml`, `.pnp.cjs`, `.pnp.loader.mjs`, `.pnpmfile.cjs`, `bunfig.toml`, `.bunfig.toml`
* `.bazelrc`, `.bazelversion`, `.bazeliskrc`
* `.pre-commit-config.yaml`, `lefthook.yml`, `lefthook.yaml`, `.lefthook.yml`, `.lefthook.yaml`
* `gradle-wrapper.properties`, `maven-wrapper.properties`
* `.devcontainer.json`
* `.ripgreprc`, `pyrightconfig.json`
* `.mcp.json`, `.claude.json`

## 관련 정보

* [권한](/docs/en/permissions): allow, ask, deny 규칙 및 관리형 정책
* [자동 모드 구성](/docs/en/auto-mode-config): 조직이 신뢰하는 인프라를 분류기에 전달
* [훅](/docs/en/hooks): `PreToolUse` 및 `PermissionRequest` 훅을 통한 커스텀 권한 로직
* [Ultraplan](/docs/en/ultraplan): 브라우저 기반 검토가 포함된 웹 세션의 Claude Code에서 plan 모드 실행
* [보안](/docs/en/security): 안전장치 및 모범 사례
* [샌드박싱](/docs/en/sandboxing): Bash 명령에 대한 파일 시스템 및 네트워크 격리
* [비대화형 모드](/docs/en/headless): `-p` 플래그로 Claude Code 실행
