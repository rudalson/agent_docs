> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 조직에 플러그인 추천하기

> 마켓플레이스 플러그인 항목에 relevancia(관련성) 블록을 추가하여 사용자의 작업이 일치할 때 Claude Code가 해당 플러그인을 제안하도록 만듭니다.

조직용 플러그인 마켓플레이스를 운영하는 경우, 사용자가 작업 중인 내용에 따라 Claude Code가 특정 플러그인을 사용자에게 추천하도록 할 수 있습니다. `marketplace.json`의 플러그인 항목에 `relevance` 블록을 추가한 다음, 관리형 설정에서 마켓플레이스를 허용 목록(allowlist)에 등록하세요. 사용자의 세션이 선언된 신호 중 하나와 일치하면 Claude Code가 해당 플러그인에 대한 설치 제안을 표시합니다.

마켓플레이스 선언 제안은 [관리형 설정](/docs/en/settings#settings-files)을 통해 마켓플레이스별로 옵트인됩니다. 관리자가 허용 목록에 추가하기 전까지는 공식 Anthropic 마켓플레이스를 포함하여 어떤 마켓플레이스의 `relevance` 선언도 제안을 생성하지 않습니다. Claude Code에는 이 허용 목록과 독립적인 내장 제안 하나도 포함되어 있습니다; 이 팁과 마켓플레이스 선언 팁 모두 [`spinnerTipsEnabled`](/docs/en/settings#available-settings)가 `false`로 설정되면 비활성화됩니다.

이 기능은 Claude Code v2.1.152 이상이 필요합니다. 이전 클라이언트는 `relevance` 필드를 무시합니다.

이 페이지는 마켓플레이스 운영자 및 기업 관리자를 위한 문서입니다. 플러그인을 설치하려는 경우 [플러그인 탐색 및 설치](/docs/en/discover-plugins)를 참조하세요.

## 작동 방식

`marketplace.json` 내의 각 플러그인 항목은 `relevance` 객체를 포함할 수 있습니다. 해당 객체는 주제(topic) 및 하나 이상의 신호(signals)를 명시합니다. 신호는 작업 디렉토리나 Claude가 읽은 파일 등 Claude Code가 현재 세션에 맞춰 테스트하는 패턴입니다.

신호 매칭은 사용자의 머신에서 로컬로 발생합니다. 매칭 시 네트워크 트래픽이 추가되지 않으며, 어떤 신호가 매칭되었는지나 해당 값을 Anthropic 또는 마켓플레이스 운영자에게 보고하지 않습니다.

신호가 일치하고 플러그인이 아직 설치되지 않은 경우, Claude Code는 세 곳에서 플러그인을 표시합니다:

* **스피너 팁 (Spinner tip)**: Claude가 응답하는 동안 스피너 아래에 `/plugin install` 명령과 함께 "Working with *topic*? Install the *plugin* plugin" 메시지가 나타납니다.
* **세션 시작 제안**: `cwd` 신호가 작업 디렉토리와 일치하는 경우, 첫 턴이 시작되기 전에 한 줄짜리 `plugin suggestion: <name>@<marketplace> · /plugin` 알림이 나타납니다. 이 서페이스는 Claude Code v2.1.153 이상이 필요합니다.
* **`/plugin` Discover 탭**: 플러그인이 "suggested for this directory" 또는 "suggested for stripe commands"와 같은 주석과 함께 Discover 목록 상단에 고정됩니다. 이 서페이스는 Claude Code v2.1.154 이상이 필요합니다.

스피너 팁과 세션 시작 알림은 스피너 팁 시스템의 일부입니다. 둘 다 사용자나 프로젝트가 `spinnerTipsEnabled`를 `false`로 설정하거나 커스텀 `spinnerTipsOverride`가 `excludeDefault`와 함께 구성되었을 때 비활성화됩니다. Discover 탭 고정 기능은 팁 설정과 독립적입니다.

Claude Code는 절대로 플러그인을 자동으로 설치하지 않습니다. 사용자가 항상 이를 확인합니다.

## 플러그인 항목에 relevance 추가하기

`marketplace.json` 파일의 플러그인 항목에 `relevance` 객체를 추가하세요. 다음 예시는 Claude가 `.tf` 파일을 읽거나 `terraform`을 실행할 때 `terraform-helpers` 플러그인이 관련이 있음을 선언합니다:

```json theme={null}
{
  "name": "acme-corp-plugins",
  "owner": { "name": "Acme Platform Team" },
  "plugins": [
    {
      "name": "terraform-helpers",
      "source": "./plugins/terraform-helpers",
      "description": "Acme conventions and helpers for Terraform",
      "relevance": {
        "topic": "Terraform",
        "signals": {
          "cli": ["terraform"],
          "filesRead": ["**/*.tf"]
        }
      }
    }
  ]
}
```

`relevance` 블록은 있지만 일치하는 신호가 없는 플러그인은 다른 여타 마켓플레이스 항목처럼 동작합니다. Discover 목록의 일반 위치에 표시되며 스피너 팁으로 나타나지 않습니다.

## 필드 참조

### `relevance`

| 필드      | 타입   | 설명                                                                                                                                                                                                                                                                                                                                                               |
| :-------- | :----- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `topic`   | string | 선택 사항. 스피너 팁에서 "Working with *topic*?"을 채우는 문구입니다. `Stripe`와 같이 제품 이름인 경우가 많습니다. 플러그인 이름이 주제로 자연스럽게 읽히지 않을 때는 `design`과 같은 도메인을 사용하세요. 기본값은 각 하이픈 세그먼트가 대문자화된 플러그인 이름입니다. 세션 시작 알림은 이 값을 사용하지 않습니다. 최대 64자.                                |
| `signals` | object | 플러그인이 관련이 있는지를 결정하는 매처(matchers). 플러그인을 제안 가능하게 만들려면 하나 이상의 신호가 필요합니다. 아래 표를 참조하세요.                                                                                                                                                                                                                        |

### `relevance.signals`

| 필드           | 타입             | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :------------- | :--------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cwd`          | array of strings | 세션의 작업 디렉토리에 맞춰 테스트되는 글로브(glob) 패턴. 절대 경로 및 git 저장소 내부일 때 저장소 루트 기준 상대 경로로 매칭됩니다. 슬래시(/) 정규화되며 대소문자를 구분하지 않습니다. 모든 패턴은 디렉토리 자체 및 그 아래의 모든 항목과 매칭되므로 `infra`, `infra/`, `infra/**`는 동일하게 동작합니다. 첫 턴 전인 세션 시작 시에 매칭될 수 있는 유일한 신호입니다. 각각 최대 256자의 패턴을 최대 10개까지 지정 가능.                                                                                                                                                                                                                                                                                                                                                                 |
| `cli`          | array of strings | 이번 세션에서 Claude가 실행한 셸 명령의 명령 이름(예: `["stripe"]`). 모든 플랫폼에 적용됩니다: Windows의 PowerShell이나 Git Bash에서 실행된 명령도 동일한 방식으로 기록됩니다. Claude Code는 셸 도구 호출당 하나의 명령 이름을 기록합니다: 선행 환경 변수 할당이나 `sudo` 뒤의 첫 토큰. 복합 명령은 선행 명령에만 기여하므로 `cd infra && terraform plan`은 `terraform`이 아닌 `cd`를 기록합니다. 완전 일치(exact match). 각각 최대 64자의 항목을 최대 10개까지 지정 가능.                                                                                                                                                                                                                                                                                                                          |
| `hosts`        | array of strings | 이번 세션에서 Bash 명령의 `http://` 또는 `https://` URL에서 포착된 호스트 이름(예: `["api.stripe.com"]`). 소문자 호스트 이름만 포함: 스키마, 포트, 경로 없음. 대소문자 구분 없는 완전 일치. 각각 최대 128자의 항목을 최대 20개까지 지정 가능.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `filesRead`    | array of strings | 이번 세션에서 Claude가 읽은 파일의 경로에 맞춰 테스트되는 글로브 패턴(예: `["**/*.tf"]`). 슬래시(/) 정규화되며 대소문자를 구분하지 않습니다. 각각 최대 256자의 패턴을 최대 10개까지 지정 가능.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `manifestDeps` | array of objects | 이번 세션에서 Claude가 읽은 패키지 매니페스트에 선언된 의존성. 각 항목은 `{ "file": "...", "pattern": "..." }`이며, `file`은 세션 상태에 기록된 매니페스트 파일 경로(일반적으로 절대 경로)에 맞춰 테스트되는 정규식이고, `pattern`은 해당 파일 내용에 맞춰 테스트되는 정규식입니다. 시작 앵커 처리된 패턴은 절대 경로와 매칭되지 않으므로 `file`을 끝 부분에 앵커 지정하세요(예: JSON 이스케이프 처리된 형태의 `[/\\\\]package\\.json$`). 이 신호에서는 경로의 구분자가 정규화되지 않으므로 Windows 경로는 백슬래시를 사용합니다. 512KB를 초과하는 매니페스트 파일은 건너뜁니다. 두 값 모두 최대 256자의 JavaScript `RegExp` 소스 문자열입니다. `file`은 대소문자를 구분하지 않고 매칭되며 `pattern`은 대소문자를 구분합니다. 최대 10개 항목. |

`cli`, `hosts`, `filesRead`, `manifestDeps` 신호는 세션 기록이 필요하므로 스피너 팁과 Discover 탭에서만 매칭될 수 있습니다. `cwd` 신호만이 세션 시작 시 매칭될 수 있습니다. `filesRead` 및 `manifestDeps` 신호는 세션에 기록된 파일 상태를 테스트하며, 여기에는 Claude가 쓰거나 편집한 파일과 지연 로드된 `CLAUDE.md` 메모리 파일도 포함됩니다.

다음 예시는 Claude가 `stripe`에 의존하는 `package.json`을 읽었을 때 Stripe 플러그인을 추천하도록 `manifestDeps`를 사용하는 방법을 보여줍니다. `file` 패턴은 `[/\\\\]`를 사용하여 슬래시 및 백슬래시 경로 구분자 모두와 매칭되며, `\\.`를 사용하여 점(dot) 문자를 문자로 매칭합니다. JSON에서 정규식의 각 백슬래시는 두 번 작성됩니다.

```json theme={null}
{
  "name": "stripe-helpers",
  "source": "./plugins/stripe-helpers",
  "relevance": {
    "topic": "Stripe",
    "signals": {
      "manifestDeps": [
        {
          "file": "[/\\\\]package\\.json$",
          "pattern": "\"stripe\"\\s*:"
        }
      ]
    }
  }
}
```

<Note>
  `relevance` 및 `relevance.signals` 아래의 알 수 없는 필드는 이전 Claude Code 클라이언트가 마켓플레이스를 계속 로드할 수 있도록 로드 시점에 무시됩니다. 이들을 경고로 표시하려면 `claude plugin validate`를 실행하세요.
</Note>

## 관리형 설정에서 제안 활성화하기

`marketplace.json`에 `relevance`를 선언하는 것만으로는 충분하지 않습니다. 사용자에게 제안이 나타나기 전에 관리자가 [관리형 설정](/docs/en/settings#settings-files)에서 해당 마켓플레이스를 허용 목록에 등록해야 합니다.

`pluginSuggestionMarketplaces`에 마켓플레이스 이름을 추가하세요. 공식 Anthropic 마켓플레이스 이외의 마켓플레이스의 경우 동일한 관리형 설정에서 `extraKnownMarketplaces` 항목 또는 `strictKnownMarketplaces` 항목으로 마켓플레이스 소스도 선언해야 합니다. 머신에 등록된 마켓플레이스가 다른 소스에서 온 경우 허용 목록에 오른 이름이 무시됩니다. 이는 무관한 소스가 허용 목록에 오른 이름으로 등록되어 조직 전체에 플러그인을 제안하는 것을 방지합니다.

다음 `managed-settings.json`은 GitHub 저장소로부터 조직 마켓플레이스를 등록하고 해당 제안을 활성화합니다:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "acme-corp-plugins": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  },
  "pluginSuggestionMarketplaces": ["acme-corp-plugins"]
}
```

공식 마켓플레이스는 해당 이름이 공식 Anthropic 소스로부터만 등록될 수 있으므로 소스 선언 요구사항에서 면제됩니다. 이름만 허용 목록에 등록하는 것으로 충분합니다:

```json theme={null}
{
  "pluginSuggestionMarketplaces": ["claude-plugins-official"]
}
```

전체 구성에 대해서는 `pluginSuggestionMarketplaces` 및 [`extraKnownMarketplaces`](/docs/en/settings#extraknownmarketplaces)에 대한 [설정 참조 문서](/docs/en/settings)를 확인하세요.

## 사용자에게 보이는 내용

세션 중 신호가 일치하면 스피너 팁이 다음과 같이 표시됩니다:

```text theme={null}
Working with Terraform? Install the terraform-helpers plugin:
/plugin install terraform-helpers@acme-corp-plugins
```

세션 시작 시 `cwd` 신호가 일치하면 한 줄짜리 알림이 나타납니다:

```text theme={null}
plugin suggestion: terraform-helpers@acme-corp-plugins · /plugin
```

특정 플러그인의 제안은 스피너 팁과 세션 시작 알림을 통틀어 최대 3 세션당 1회 표시되며, 플러그인이 설치되면 둘 다 반복되지 않습니다. 세션 시작 알림은 제안이 2회 표시된 후 추가로 나타나지 않습니다.

`/plugin` Discover 탭에서 해당 플러그인은 `suggested for this directory` 또는 `suggested for terraform commands`와 같이 일치하는 신호를 명시하는 주석과 함께 다른 결과들 상단에 고정됩니다. Discover 탭은 해당 플러그인을 한 번만 고정하며; 이후 방문에서는 일반 순서로 나열됩니다. Discover 탭 고정 기능은 Claude Code v2.1.154 이상이 필요합니다. v2.1.152에서는 스피너 팁만 나타나며; 세션 시작 알림은 v2.1.153에서 추가되었습니다.

## 마켓플레이스 검증하기

게시하기 전에 `relevance` 블록을 확인하려면 마켓플레이스 디렉토리를 대상으로 `claude plugin validate`를 실행하세요:

```
claude plugin validate ./my-marketplace
```

검증 도구(validator)는 `relevance` 및 `relevance.signals` 아래의 알 수 없는 키를 경고로 보고하고, 객체가 아닌 `relevance` 값을 지적하며, 스키마/포트/경로가 포함된 `signals.hosts` 항목을 거부합니다.

## 참고 항목

* [플러그인 마켓플레이스 생성 및 배포](/docs/en/plugin-marketplaces): 플러그인을 호스팅하는 마켓플레이스 구축
* [CLI에서 플러그인 추천하기](/docs/en/plugin-hints): Claude Code 세션 신호 대신 사용자의 자체 CLI에서 사용자에게 프롬프트 표시
* [설정](/docs/en/settings): `pluginSuggestionMarketplaces` 및 `extraKnownMarketplaces`에 대한 전체 참조 문서
