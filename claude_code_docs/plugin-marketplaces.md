> ## 문서 색인
> 전체 문서 색인은 다음에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 자세히 살펴보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 플러그인 마켓플레이스 생성 및 배포 (Create and distribute a plugin marketplace)

> 플러그인 마켓플레이스를 구축하고 호스팅하여 팀 및 커뮤니티 전반에 Claude Code 확장 프로그램을 배포합니다.

**플러그인 마켓플레이스**는 다른 사람에게 플러그인을 배포할 수 있도록 하는 카탈로그입니다. 마켓플레이스는 중앙 집중식 탐색, 버전 추적, 자동 업데이트, git 리포지토리 및 로컬 경로를 포함한 여러 소스 유형 지원을 제공합니다. 이 가이드는 팀이나 커뮤니티와 플러그인을 공유하기 위해 자체 마켓플레이스를 생성하는 방법을 보여줍니다.

기존 마켓플레이스에서 플러그인을 설치하려는 경우 [사전 구축된 플러그인 탐색 및 설치](/docs/en/discover-plugins)를 참조하세요.

## 개요

마켓플레이스를 생성하고 배포하는 단계:

1. **플러그인 생성**: 스킬, 에이전트, 훅, MCP 서버 또는 LSP 서버가 포함된 하나 이상의 플러그인을 구축합니다. 이 가이드는 배포할 플러그인이 이미 있다고 가정합니다. 플러그인 생성 방법에 대한 자세한 내용은 [플러그인 만들기](/docs/en/plugins)를 참조하세요.
2. **마켓플레이스 파일 생성**: 플러그인과 플러그인을 찾을 수 있는 위치를 나열하는 `marketplace.json`을 정의합니다. [마켓플레이스 파일 생성](#create-the-marketplace-file)을 참조하세요.
3. **마켓플레이스 호스팅**: GitHub, GitLab 또는 다른 git 호스트에 푸시합니다. [마켓플레이스 호스팅 및 배포](#host-and-distribute-marketplaces)를 참조하세요.
4. **사용자와 공유**: 사용자는 `/plugin marketplace add`로 마켓플레이스를 추가하고 개별 플러그인을 설치합니다. [플러그인 탐색 및 설치](/docs/en/discover-plugins)를 참조하세요.

마켓플레이스가 게시되면 리포지토리에 변경 사항을 푸시하여 업데이트할 수 있습니다. 사용자는 `/plugin marketplace update`를 통해 로컬 사본을 새로 고칩니다.

## 튜토리얼: 로컬 마켓플레이스 생성

이 예시는 하나의 플러그인(코드 리뷰용 `quality-review` 스킬)이 있는 마켓플레이스를 생성합니다. 디렉터리 구조를 만들고, 스킬을 추가하고, 플러그인 매니페스트 및 마켓플레이스 카탈로그를 생성한 다음 설치하여 테스트해 봅니다.

<Steps>
  <Step title="디렉터리 구조 생성">
    ```bash theme={null}
    mkdir -p my-marketplace/.claude-plugin
    mkdir -p my-marketplace/plugins/quality-review-plugin/.claude-plugin
    mkdir -p my-marketplace/plugins/quality-review-plugin/skills/quality-review
    ```
  </Step>

  <Step title="스킬 생성">
    `quality-review` 스킬이 수행하는 작업을 정의하는 `SKILL.md` 파일을 생성합니다.

    ```markdown my-marketplace/plugins/quality-review-plugin/skills/quality-review/SKILL.md theme={null}
    ---
    description: Review code for bugs, security, and performance
    ---

    Review the code I've selected or the recent changes for:
    - Potential bugs or edge cases
    - Security concerns
    - Performance issues
    - Readability improvements

    Be concise and actionable.
    ```
  </Step>

  <Step title="플러그인 매니페스트 생성">
    플러그인을 설명하는 `plugin.json` 파일을 생성합니다. 매니페스트는 `.claude-plugin/` 디렉터리에 들어갑니다.

    ```json my-marketplace/plugins/quality-review-plugin/.claude-plugin/plugin.json theme={null}
    {
      "name": "quality-review-plugin",
      "description": "Adds a quality-review skill for quick code reviews",
      "version": "1.0.0",
      "author": {
        "name": "Your Name"
      }
    }
    ```

    <Note>
      `version`을 설정하면 이 필드를 변경할 때만 사용자가 업데이트를 받으므로 릴리스마다 버전을 올리세요. `version`을 생략하고 이 마켓플레이스를 git에 호스팅하면 모든 커밋이 자동으로 새 버전으로 카운트됩니다. 올바른 접근 방식을 선택하려면 [버전 확인](#version-resolution-and-release-channels)을 참조하세요.
    </Note>
  </Step>

  <Step title="마켓플레이스 파일 생성">
    플러그인을 나열하는 마켓플레이스 카탈로그를 생성합니다.

    ```json my-marketplace/.claude-plugin/marketplace.json theme={null}
    {
      "name": "my-plugins",
      "owner": {
        "name": "Your Name"
      },
      "plugins": [
        {
          "name": "quality-review-plugin",
          "source": "./plugins/quality-review-plugin",
          "description": "Adds a quality-review skill for quick code reviews"
        }
      ]
    }
    ```
  </Step>

  <Step title="추가 및 설치">
    `my-marketplace`가 포함된 디렉터리에서 Claude Code를 시작하고 다음 명령을 실행합니다. 설치 명령은 설치를 확인하기 위해 설치 범위를 선택하는 플러그인 세부 정보 뷰를 열고, `/reload-plugins`는 현재 세션에서 플러그인을 활성화합니다.

    ```shell theme={null}
    /plugin marketplace add ./my-marketplace
    /plugin install quality-review-plugin@my-plugins
    /reload-plugins
    ```
  </Step>

  <Step title="사용해 보기">
    에디터에서 코드를 선택하고 새 스킬을 실행합니다. 플러그인 스킬은 플러그인 이름으로 네임스페이스가 지정됩니다.

    ```shell theme={null}
    /quality-review-plugin:quality-review
    ```
  </Step>
</Steps>

훅, 에이전트, MCP 서버, LSP 서버를 포함하여 플러그인이 수행할 수 있는 작업에 대해 자세히 알아보려면 [플러그인](/docs/en/plugins)을 참조하세요.

<Note>
  **플러그인 설치 방식**: 사용자가 플러그인을 설치하면 Claude Code는 플러그인 디렉터리를 캐시 위치에 복사합니다. 즉, 복사되지 않는 `../shared-utils`와 같은 경로를 사용하여 플러그인 디렉터리 외부의 파일을 참조할 수 없습니다.

  플러그인 간에 파일을 공유해야 하는 경우 심볼릭 링크를 사용하세요. 자세한 내용은 [플러그인 캐싱 및 파일 확인](/docs/en/plugins-reference#plugin-caching-and-file-resolution)을 참조하세요.
</Note>

## 마켓플레이스 파일 생성

리포지토리 루트에 `.claude-plugin/marketplace.json`을 생성합니다. 이 파일은 마켓플레이스 이름, 소유자 정보 및 출처와 함께 플러그인 목록을 정의합니다.

각 플러그인 항목에는 최소한 Claude Code에 가져올 위치를 알리는 `name`과 `source`가 필요합니다. 사용 가능한 모든 필드는 아래의 [전체 스키마](#marketplace-schema)를 참조하세요.

```json theme={null}
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "./plugins/formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0",
      "author": {
        "name": "DevTools Team"
      }
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

## 마켓플레이스 스키마

### 필수 필드

| 필드 | 유형 | 설명 | 예시 |
| :--- | :--- | :--- | :--- |
| `name` | string | 마켓플레이스 식별자 (kebab-case, 공백 없음). 사용자에게 공개되는 이름입니다. 플러그인을 설치할 때 보게 됩니다 (예: `/plugin install my-tool@your-marketplace`). 각 사용자는 이름당 하나의 마켓플레이스만 등록할 수 있습니다. 동일한 이름으로 두 번째 마켓플레이스를 추가하면 첫 번째 마켓플레이스가 대체됩니다. 하나의 마켓플레이스 이름 아래에 여러 플러그인을 게시하려면 [단일 `marketplace.json`](#create-the-marketplace-file)에 모두 나열하세요. | `"acme-tools"` |
| `owner` | object | 마켓플레이스 유지 관리자 정보 ([아래 필드 참조](#owner-fields)) | |
| `plugins` | array | 사용 가능한 플러그인 목록 | 아래 참조 |

<Note>
  **예약된 이름**: 다음 마켓플레이스 이름은 공식 Anthropic 전용으로 예약되어 있으며 서드파티 마켓플레이스에서 사용할 수 없습니다: `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `claude-plugins-community`, `claude-community`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `anthropic-agent-skills`, `knowledge-work-plugins`, `life-sciences`, `claude-for-legal`, `claude-for-financial-services`, `financial-services-plugins`, `first-party-plugins`, `healthcare`. `official-claude-plugins` 또는 `anthropic-plugins-v2`와 같이 공식 마켓플레이스를 사칭하는 이름도 차단됩니다. 이러한 이름을 예약하면 서드파티 마켓플레이스가 Anthropic 게시 소스로 위장하는 것을 방지할 수 있습니다.

  Claude Code는 마켓플레이스를 추가할 때뿐만 아니라 로드할 때마다 예약된 이름을 다시 확인합니다. 이름이 예약되기 전에 해당 이름 중 하나로 등록된 마켓플레이스는 로드를 중지하고 [신뢰할 수 없는 소스에서 등록됨](/docs/en/errors#marketplace-is-registered-from-an-untrusted-source) 오류를 보고합니다. 해당 마켓플레이스를 제거하고 공식 Anthropic 소스에서 다시 추가하세요. 새로 예약된 이름의 영향을 받는 서드파티 마켓플레이스는 다른 이름으로 다시 추가하는 즉시 로드됩니다. v2.1.205 이전에는 `first-party-plugins` 및 `healthcare`가 예약되지 않았으며 이미 예약된 이름으로 등록된 마켓플레이스는 계속 로드되었습니다.
</Note>

### Owner 필드

| 필드 | 유형 | 필수 여부 | 설명 |
| :--- | :--- | :--- | :--- |
| `name` | string | 예 | 유지 관리자 또는 팀 이름 |
| `email` | string | 아니오 | 유지 관리자의 연락처 이메일 |

### 선택적 필드

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `$schema` | string | 에디터 자동 완성 및 검증을 위한 JSON Schema URL. Claude Code는 로드 시 이 필드를 무시합니다. |
| `description` | string | 짧은 마켓플레이스 설명 |
| `version` | string | 마켓플레이스 매니페스트 버전 |
| `metadata.pluginRoot` | string | 상대 플러그인 소스 경로 앞에 붙는 기본 디렉터리 (예: `"./plugins"`를 설정하면 `"source": "./plugins/formatter"` 대신 `"source": "formatter"` 작성 가능) |
| `allowCrossMarketplaceDependenciesOn` | array | 이 마켓플레이스의 플러그인이 종속될 수 있는 다른 마켓플레이스. 여기에 나열되지 않은 마켓플레이스의 종속성은 설치 시 차단됩니다. [다른 마켓플레이스의 플러그인에 종속](/docs/en/plugin-dependencies#depend-on-a-plugin-from-another-marketplace)을 참조하세요. |
| `renames` | object | {/* min-version: 2.1.193 */}이전 플러그인 `name`에서 현재 이름으로, 또는 플러그인이 제거된 경우 `null`로 매핑. `plugins`에서 항목의 이름을 바꾸거나 제거할 때 기존 사용자가 자동으로 마이그레이션되도록 합니다. [플러그인 이름 변경 또는 제거](#rename-or-remove-a-plugin)를 참조하세요. Claude Code v2.1.193 이상이 필요합니다. |

`description` 및 `version`은 하위 호환성을 위해 `metadata` 아래에서도 허용됩니다.

## 플러그인 항목

`plugins` 배열의 각 플러그인 항목은 플러그인 및 찾을 수 있는 위치를 설명합니다. `description`, `version`, `author`, `commands`, `hooks`와 같은 [플러그인 매니페스트 스키마](/docs/en/plugins-reference#plugin-manifest-schema)의 모든 필드에 마켓플레이스 전용 필드인 `source`, `category`, `tags`, `strict`, `relevance`를 추가할 수 있습니다.

### 필수 필드

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `name` | string | 플러그인 식별자 (kebab-case, 공백 없음). 사용자에게 공개되는 이름입니다. 설치 시 보게 됩니다 (예: `/plugin install my-plugin@marketplace`). |
| `source` | string\|object | 플러그인을 가져올 위치 (아래 [플러그인 소스](#plugin-sources) 참조) |

### 선택적 플러그인 필드

**표준 메타데이터 필드:**

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `displayName` | string | {/* min-version: 2.1.143 */}UI 화면에 표시되는 읽기 쉬운 이름. 생략하면 `name`으로 폴백됩니다. 공백과 대소문자를 포함할 수 있습니다. 네임스페이스나 조회에는 사용되지 않습니다. Claude Code v2.1.143 이상이 필요합니다. |
| `description` | string | 짧은 플러그인 설명 |
| `version` | string | 플러그인 버전. 설정된 경우(여기 또는 `plugin.json`에서) 플러그인이 이 문자열에 고정되며 해당 문자열이 변경될 때만 사용자가 업데이트를 받습니다. git 커밋 SHA로 폴백하려면 생략하세요. [버전 확인](#version-resolution-and-release-channels)을 참조하세요. |
| `author` | object | 플러그인 작성자 정보 (`name` 필수, `email` 선택) |
| `homepage` | string | 플러그인 홈페이지 또는 문서 URL |
| `repository` | string | 소스 코드 리포지토리 URL |
| `license` | string | SPDX 라이선스 식별자 (예: MIT, Apache-2.0) |
| `keywords` | array | 플러그인 탐색 및 범주화를 위한 태그 |
| `category` | string | 조직을 위한 플러그인 범주 |
| `tags` | array | 검색 가능성을 위한 태그 |
| `strict` | boolean | `plugin.json`이 구성 요소 정의의 권한인지 여부를 제어합니다(기본값: true). 아래 [Strict 모드](#strict-mode)를 참조하세요. |
| `relevance` | object | {/* min-version: 2.1.152 */}Claude Code에게 이 플러그인을 사용자에게 추천할 시기를 알리는 신호. 관리자가 관리형 설정에 허용 목록으로 지정한 마켓플레이스에 대해서만 적용됩니다. [조직을 위한 플러그인 추천](/docs/en/plugin-relevance)을 참조하세요. Claude Code v2.1.152 이상이 필요합니다. |
| `defaultEnabled` | boolean | {/* min-version: 2.1.154 */}설치 후 플러그인이 활성화되는지 여부(기본값: true). 사용자가 동의할 때까지 플러그인을 비활성화된 상태로 설치하려면 `false`로 설정하세요. 플러그인의 `plugin.json`에 있는 동일한 필드보다 우선 적용됩니다. [기본 활성화 여부](/docs/en/plugins-reference#default-enablement)를 참조하세요. Claude Code v2.1.154 이상이 필요합니다. |

**구성 요소 구성 필드:**

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `skills` | string\|array | `<name>/SKILL.md`를 포함하는 스킬 디렉터리의 커스텀 경로 |
| `commands` | string\|array | 플랫 `.md` 스킬 파일 또는 디렉터리의 커스텀 경로 |
| `agents` | string\|array | 에이전트 파일의 커스텀 경로 |
| `hooks` | string\|object | 커스텀 훅 구성 또는 훅 파일 경로 |
| `mcpServers` | string\|object | MCP 서버 구성 또는 MCP 구성 경로 |
| `lspServers` | string\|object | LSP 서버 구성 또는 LSP 구성 경로 |

## 플러그인 소스

플러그인 소스는 Claude Code에 마켓플레이스에 나열된 각 개별 플러그인을 가져올 위치를 알려줍니다. 이는 `marketplace.json` 내의 각 플러그인 항목의 `source` 필드에 설정됩니다.

Claude Code가 플러그인을 로컬 머신에 복제하거나 다운로드한 후, `~/.claude/plugins/cache`의 로컬 버전 지정 플러그인 캐시에 플러그인을 복사합니다.

| 소스 | 유형 | 필드 | 참고 사항 |
| :--- | :--- | :--- | :--- |
| 상대 경로 | `string` (예: `"./my-plugin"`) | 없음 | 마켓플레이스 리포지토리 내의 로컬 디렉터리. `./`로 시작해야 합니다. `.claude-plugin/` 디렉터리가 아닌 마켓플레이스 루트를 기준으로 해석됩니다. |
| `github` | object | `repo`, `ref?`, `sha?` | |
| `url` | object | `url`, `ref?`, `sha?` | Git URL 소스 |
| `git-subdir` | object | `url`, `path`, `ref?`, `sha?` | git 리포지토리 내부의 하위 디렉터리. 모노리포의 대역폭을 최소화하기 위해 희소 복제(sparse clone)합니다. |
| `npm` | object | `package`, `version?`, `registry?` | `npm install`을 통해 설치됨 |

<Note>
  **마켓플레이스 소스 vs 플러그인 소스**: 둘은 서로 다른 것을 제어하는 별개의 개념입니다.

  * **마켓플레이스 소스**: `marketplace.json` 카탈로그 자체를 가져올 위치. 사용자가 `/plugin marketplace add`를 실행할 때나 `extraKnownMarketplaces` 설정에 설정됩니다. `ref`(브랜치/태그)를 지원하지만 `sha`는 지원하지 않습니다.
  * **플러그인 소스**: `marketplace.json` 내부에 나열된 개별 플러그인을 가져올 위치. `marketplace.json` 내부의 각 플러그인 항목의 `source` 필드에 설정됩니다. `ref`(브랜치/태그)와 `sha`(정확한 커밋)를 모두 지원합니다.

  예를 들어 `acme-corp/plugin-catalog`(마켓플레이스 소스)에 호스팅된 마켓플레이스는 `acme-corp/code-formatter`(플러그인 소스)에서 가져온 플러그인을 나열할 수 있습니다. 마켓플레이스 소스와 플러그인 소스는 서로 다른 리포지토리를 가리키며 독립적으로 고정됩니다.
</Note>

아래의 git 기반 소스 유형은 `github`, `url`, `git-subdir`입니다. 이들 중 어느 것에서든 `ref`와 `sha`가 모두 설정된 경우 `sha`가 유효한 고정(pin)이 됩니다. Claude Code는 고정된 커밋을 직접 가져와 체크아웃합니다.

GitHub, GitLab, Bitbucket을 포함한 대부분의 git 호스트에서 이는 고정된 커밋이 리포지토리에서 도달 가능한 한 `ref`에 지정된 브랜치나 태그가 업스트림에서 삭제된 경우에도 설치가 성공함을 의미합니다. AWS CodeCommit과 같은 일부 서버는 SHA로 커밋을 가져오는 것을 지원하지 않습니다. 해당 서버에서는 `ref`가 여전히 존재해야 하며 고정된 커밋에 도달할 수 있어야 합니다.

### 상대 경로

동일한 리포지토리에 있는 플러그인의 경우 `./`로 시작하는 경로를 사용하세요:

```json theme={null}
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

경로는 `.claude-plugin/`을 포함하는 디렉터리인 마켓플레이스 루트를 기준으로 해석됩니다. 위 예시에서 `marketplace.json`이 `<repo>/.claude-plugin/marketplace.json`에 있더라도 `./plugins/my-plugin`은 `<repo>/plugins/my-plugin`을 가리킵니다. 마켓플레이스 루트 외부의 경로를 참조할 때 `../`를 사용하지 마세요.

<Note>
  상대 경로는 마켓플레이스의 로컬 복사본에 대해 해석되므로 사용자가 git 소스나 로컬 디렉터리에서 마켓플레이스를 추가할 때 작동합니다. 사용자가 `marketplace.json` 파일에 대한 직접 URL을 통해 마켓플레이스를 추가하는 경우 해당 파일만 다운로드되므로 상대 경로가 해석되지 않습니다. URL 기반 배포의 경우 대신 GitHub, npm 또는 git URL 소스를 사용하세요. 자세한 내용은 [문제 해결](#plugins-with-relative-paths-fail-in-url-based-marketplaces)을 참조하세요.
</Note>

### GitHub 리포지토리

```json theme={null}
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  }
}
```

특정 브랜치, 태그 또는 커밋으로 고정할 수 있습니다:

```json theme={null}
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `repo` | string | 필수. `owner/repo` 형식의 GitHub 리포지토리 |
| `ref` | string | 선택. Git 브랜치 또는 태그 (기본값: 리포지토리 기본 브랜치) |
| `sha` | string | 선택. 정확한 버전에 고정하기 위한 전체 40자 git 커밋 SHA |

### Git 리포지토리

```json theme={null}
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  }
}
```

특정 브랜치, 태그 또는 커밋으로 고정할 수 있습니다:

```json theme={null}
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git",
    "ref": "main",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `url` | string | 필수. 전체 git 리포지토리 URL (`https://` 또는 `git@`). `.git` 접미사는 선택 사항이므로 접미사 없는 Azure DevOps 및 AWS CodeCommit URL이 작동합니다. |
| `ref` | string | 선택. Git 브랜치 또는 태그 (기본값: 리포지토리 기본 브랜치) |
| `sha` | string | 선택. 정확한 버전에 고정하기 위한 전체 40자 git 커밋 SHA |

### Git 하위 디렉터리

`git-subdir`을 사용하여 git 리포지토리의 하위 디렉터리 내에 있는 플러그인을 가리킵니다. Claude Code는 희소 파설 복제를 사용하여 하위 디렉터리만 가져오므로 대규모 모노리포의 대역폭을 최소화합니다.

```json theme={null}
{
  "name": "my-plugin",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/acme-corp/monorepo.git",
    "path": "tools/claude-plugin"
  }
}
```

특정 브랜치, 태그 또는 커밋으로 고정할 수 있습니다:

```json theme={null}
{
  "name": "my-plugin",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/acme-corp/monorepo.git",
    "path": "tools/claude-plugin",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

`url` 필드는 GitHub 축약형(`owner/repo`) 또는 SSH URL(`git@github.com:owner/repo.git`)도 수용합니다.

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `url` | string | 필수. Git 리포지토리 URL, GitHub `owner/repo` 축약형, 또는 SSH URL |
| `path` | string | 필수. 플러그인을 포함하는 리포지토리 내의 하위 디렉터리 경로 (예: `"tools/claude-plugin"`) |
| `ref` | string | 선택. Git 브랜치 또는 태그 (기본값: 리포지토리 기본 브랜치) |
| `sha` | string | 선택. 정확한 버전에 고정하기 위한 전체 40자 git 커밋 SHA |

### npm 패키지

npm 패키지로 배포된 플러그인은 `npm install`을 사용하여 설치됩니다. 이는 공개 npm 레지스트리 또는 팀이 호스팅하는 사설 레지스트리의 모든 패키지에서 작동합니다.

```json theme={null}
{
  "name": "my-npm-plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin"
  }
}
```

특정 버전에 고정하려면 `version` 필드를 추가하세요:

```json theme={null}
{
  "name": "my-npm-plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin",
    "version": "2.1.0"
  }
}
```

사설 또는 내부 레지스트리에서 설치하려면 `registry` 필드를 추가하세요:

```json theme={null}
{
  "name": "my-npm-plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin",
    "version": "^2.0.0",
    "registry": "https://npm.example.com"
  }
}
```

| 필드 | 유형 | 설명 |
| :--- | :--- | :--- |
| `package` | string | 필수. 패키지 이름 또는 스코프 패키지 (예: `@org/plugin`) |
| `version` | string | 선택. 버전 또는 버전 범위 (예: `2.1.0`, `^2.0.0`, `~1.5.0`) |
| `registry` | string | 선택. 커스텀 npm 레지스트리 URL. 시스템 npm 레지스트리(일반적으로 npmjs.org)로 기본 설정됨 |

### 고급 플러그인 항목

이 예시는 명령, 에이전트, 훅 및 MCP 서버에 대한 커스텀 경로를 포함하여 많은 선택적 필드를 사용하는 플러그인 항목을 보여줍니다:

```json theme={null}
{
  "name": "enterprise-tools",
  "source": {
    "source": "github",
    "repo": "company/enterprise-plugin"
  },
  "description": "Enterprise workflow automation tools",
  "version": "2.1.0",
  "author": {
    "name": "Enterprise Team",
    "email": "enterprise@example.com"
  },
  "homepage": "https://docs.example.com/plugins/enterprise-tools",
  "repository": "https://github.com/company/enterprise-plugin",
  "license": "MIT",
  "keywords": ["enterprise", "workflow", "automation"],
  "category": "productivity",
  "commands": [
    "./commands/core/",
    "./commands/enterprise/",
    "./commands/experimental/preview.md"
  ],
  "agents": ["./agents/security-reviewer.md", "./agents/compliance-checker.md"],
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "enterprise-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  },
  "strict": false
}
```

주요 참고 사항:

* **`commands` 및 `agents`**: 여러 디렉터리 또는 개별 파일을 지정할 수 있습니다. 경로는 플러그인 루트를 기준으로 합니다.
* **`${CLAUDE_PLUGIN_ROOT}`**: 훅 명령 및 MCP 서버 구성에서 플러그인의 설치 디렉터리 내에 있는 파일을 참조하려면 이 변수를 사용하세요. 플러그인이 설치될 때 캐시 위치로 복사되기 때문에 이것이 필요합니다.
  * 서버 유형별로 치환되는 구성 필드는 [치환 표](/docs/en/plugins-reference#environment-variables)를 참조하세요.
  * 플러그인 업데이트에도 유지되어야 하는 종속성이나 상태의 경우 대신 [`${CLAUDE_PLUGIN_DATA}`](/docs/en/plugins-reference#persistent-data-directory)를 사용하세요.
* **`strict: false`**: false로 설정되어 있으므로 플러그인에 자체 `plugin.json`이 필요하지 않습니다. 마켓플레이스 항목이 모든 것을 정의합니다. 아래의 [Strict 모드](#strict-mode)를 참조하세요.

기본적으로 플러그인의 스킬은 `source` 아래의 `skills/` 디렉터리에서 로드됩니다. `skills` 필드에 나열된 경로는 해당 스캔에 추가됩니다:

```json theme={null}
"skills": ["./skills/", "./extra-skills/"]
```

여러 플러그인 항목이 마켓플레이스 루트(`source: "./"`)에서 하나의 `skills/` 폴더를 공유하는 경우 각 항목이 해당 스킬만 로드하도록 특정 하위 디렉터리를 나열하세요:

```json theme={null}
"source": "./",
"skills": ["./skills/code-review", "./skills/docs"]
```

마켓플레이스 루트 `source`를 사용하면 나열된 경로가 해당 항목의 완전한 세트가 되며 공유된 `skills/` 폴더의 다른 디렉터리는 로드되지 않습니다. `./skills/` 자체 또는 플러그인 루트를 나열하면 전체 스캔이 유지됩니다. 나열된 경로 중 어느 것도 존재하지 않는 경우 기본 스캔이 대신 실행됩니다.

### Strict 모드

`strict` 필드는 `plugin.json`이 구성 요소 정의(스킬, 에이전트, 훅, MCP 서버, 출력 스타일)의 권한인지 여부를 제어합니다.

| 값 | 동작 |
| :--- | :--- |
| `true` (기본값) | `plugin.json`이 권한입니다. 마켓플레이스 항목은 추가 구성 요소로 이를 보완할 수 있으며 두 소스가 병합됩니다. |
| `false` | 마켓플레이스 항목이 전체 정의입니다. 플러그인에 구성 요소를 선언하는 `plugin.json`도 있는 경우 이는 충돌이며 플러그인 로드에 실패합니다. |

**각 모드를 사용하는 시기:**

* **`strict: true`**: 플러그인에 자체 `plugin.json`이 있고 자체 구성 요소를 관리합니다. 마켓플레이스 항목은 위에 추가 스킬이나 훅을 추가할 수 있습니다. 이것이 기본값이며 대부분의 플러그인에서 작동합니다.
* **`strict: false`**: 마켓플레이스 운영자가 전체 제어를 원합니다. 플러그인 리포지토리는 원시 파일을 제공하고 마켓플레이스 항목은 해당 파일 중 어떤 파일이 스킬, 에이전트, 훅 등으로 노출되는지 정의합니다. 마켓플레이스가 플러그인 작성자가 의도한 것과 다르게 플러그인의 구성 요소를 구조화하거나 큐레이팅할 때 유용합니다.

## 마켓플레이스 호스팅 및 배포

### GitHub에 호스팅 (권장)

GitHub는 마켓플레이스를 호스팅하고 배포하는 권장되는 방법입니다:

1. **리포지토리 생성**: 마켓플레이스를 위한 새 리포지토리 설정
2. **마켓플레이스 파일 추가**: 플러그인 정의로 `.claude-plugin/marketplace.json` 생성
3. **팀과 공유**: 사용자는 `/plugin marketplace add owner/repo`로 마켓플레이스를 추가함

**이점**: 내장된 버전 제어, 이슈 추적 및 팀 협업 기능.

### 기타 git 서비스에 호스팅

GitLab, Bitbucket 및 자체 호스팅 서버와 같은 모든 git 호스팅 서비스가 작동합니다. 사용자는 전체 리포지토리 URL로 추가합니다:

```shell theme={null}
/plugin marketplace add https://gitlab.com/company/plugins.git
```

### 사설 리포지토리

Claude Code는 사설 리포지토리에서 플러그인을 설치하는 것을 지원합니다. 수동 설치 및 업데이트의 경우 Claude Code는 기존 git 자격 증명 도우미를 사용하므로 `gh auth login`, macOS Keychain 또는 `git-credential-store`를 통한 HTTPS 액세스가 터미널에서와 동일하게 작동합니다. SSH 액세스는 호스트가 이미 `known_hosts` 파일에 있고 키가 `ssh-agent`에 로드되어 있는 한 작동합니다. Claude Code가 호스트 지문 및 키 암호에 대한 대화형 SSH 프롬프트를 억제하기 때문입니다. GitHub `owner/repo` 축약형 소스는 기본적으로 SSH를 통해 복제됩니다. HTTPS를 통해 복제하려면 [`CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1`](/docs/en/env-vars#variables)을 설정하세요.

백그라운드 자동 업데이트는 다르게 작동합니다. 기본적으로 백그라운드 새로 고침은 `git pull`에 대해 git 자격 증명 도우미를 비활성화하므로 도우미가 구성된 경우에도 풀이 HTTPS를 통해 사설 리포지토리에 인증할 수 없습니다. SSH 원격은 영향을 받지 않습니다. `ssh-agent`에 로드된 키는 수동 작업과 동일한 방식으로 백그라운드 풀을 인증합니다. 백그라운드 풀이 실패하면 Claude Code는 마켓플레이스를 처음부터 다시 복제하는 방식으로 폴백합니다. 재복제는 저장된 git 자격 증명을 사용하지만 [대규모 리포지토리에서는 시간 초과가 발생할 수 있으므로](#git-operations-time-out) 사설 마켓플레이스 자동 업데이트가 간헐적으로 실패할 수 있습니다.

두 가지 설정으로 사설 마켓플레이스가 예측 가능하게 동작하도록 합니다:

* `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1`을 설정하여 백그라운드 풀이 실패할 때 삭제 및 재복제하는 대신 기존 복제본을 유지하세요. 플러그인은 마지막 동기화 상태에서 계속 작동하며 `/plugin marketplace update`를 통한 수동 업데이트는 여전히 자격 증명으로 가져옵니다.
* 재복제 폴백이 프롬프트 없이 인증할 수 있도록 GitHub의 경우 `gh auth setup-git`과 같은 git 자격 증명 도우미를 구성하세요.

환경에 `GITHUB_TOKEN`과 같은 공급자 토큰을 설정하는 것만으로는 백그라운드 인증이 활성화되지 않습니다. 토큰은 구성된 자격 증명 도우미(예: `GH_TOKEN` 및 `GITHUB_TOKEN`을 읽는 `gh` CLI 도우미)를 통해서만 적용됩니다.

백그라운드 풀 자체가 HTTPS를 통해 인증하도록 하려면 전역 git URL 재작성을 구성하세요. 재작성은 원격 URL에 토큰을 포함하므로 백그라운드 풀이 자격 증명 도우미를 비활성화하더라도 적용되며 성공적인 풀은 재복제 폴백을 건너끕니다. 다음 예시는 액세스 토큰을 포함하도록 마켓플레이스 리포지토리의 URL을 재작성합니다:

```bash theme={null}
git config --global url."https://x-access-token:YOUR_TOKEN@github.com/acme-corp/plugins".insteadOf "https://github.com/acme-corp/plugins"
```

재작성의 범위를 마켓플레이스 리포지토리 또는 조직 경로로 제한하세요. 기본 위치가 호스트뿐인 재작성은 해당 머신의 해당 호스트에 대한 모든 가져오기 및 푸시에 적용되며 본인의 리포지토리에 푸시하는 것을 포함하여 일반적인 자격 증명을 재정의합니다.

각 공급자는 재작성된 URL에 서로 다른 사용자 이름을 기대하며, 동일한 경로 범위 지정이 모든 공급자에 적용됩니다. 자체 호스팅 서버의 경우 호스트 이름을 서버의 호스트 이름으로 교체하세요:

| 공급자 | 재작성된 URL 형식 |
| :--- | :--- |
| GitHub | `https://x-access-token:YOUR_TOKEN@github.com/acme-corp/plugins` |
| GitLab | `https://oauth2:YOUR_TOKEN@gitlab.com/acme-corp/plugins` |
| Bitbucket | `https://x-token-auth:YOUR_TOKEN@bitbucket.org/acme-corp/plugins` |

재작성은 gitconfig에 토큰을 일반 텍스트로 저장하므로 마켓플레이스 리포지토리에 대한 읽기 전용 액세스 권한이 있는 토큰을 사용하세요.

<Note>
  CI/CD 환경에서는 사설 리포지토리에서 플러그인을 설치하기 전에 git 자격 증명 도우미를 구성하세요. GitHub Actions에서는 마켓플레이스 리포지토리에 대한 읽기 접근 권한이 있는 토큰을 `GH_TOKEN`으로 내보낸 다음 `gh auth setup-git`을 실행하세요. 기본 워크플로 토큰은 워크플로 자체 리포지토리에만 접근할 수 있으므로 다른 리포지토리의 사설 마켓플레이스에는 개인 액세스 토큰 또는 앱 토큰이 필요합니다. 파이프라인에 구성된 전역 URL 재작성도 백그라운드 풀을 직접 인증합니다.
</Note>

### 배포 전 로컬에서 테스트

공유하기 전에 마켓플레이스를 로컬에서 테스트하세요:

```shell theme={null}
/plugin marketplace add ./my-marketplace
/plugin install quality-review-plugin@my-plugins
```

추가 명령의 전체 범위(GitHub, Git URL, 로컬 경로, 원격 URL)는 [마켓플레이스 추가](/docs/en/discover-plugins#add-marketplaces)를 참조하세요.

### 팀에 마켓플레이스 요구

팀원이 프로젝트 폴더를 신뢰할 때 마켓플레이스를 설치하도록 자동으로 프롬프트를 표시하도록 리포지토리를 구성할 수 있습니다. `.claude/settings.json`에 마켓플레이스를 추가하세요:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  }
}
```

기본적으로 활성화되어야 하는 플러그인을 지정할 수도 있습니다:

```json theme={null}
{
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

전체 구성 옵션은 [플러그인 설정](/docs/en/settings#plugin-settings)을 참조하세요.

<Note>
  상대 경로가 있는 로컬 `directory` 또는 `file` 소스를 사용하면 경로는 리포지토리의 메인 체크아웃을 기준으로 해석됩니다. git worktree에서 Claude Code를 실행할 때 경로는 여전히 메인 체크아웃을 가리키므로 모든 워크트리가 동일한 마켓플레이스 위치를 공유합니다. 마켓플레이스 상태는 프로젝트별이 아닌 사용자당 한 번 `~/.claude/plugins/known_marketplaces.json`에 저장됩니다.
</Note>

### 컨테이너용 플러그인 사전 채우기

컨테이너 이미지 및 CI 환경의 경우 런타임에 아무것도 복제하지 않고도 마켓플레이스 및 플러그인을 이미 사용할 수 있도록 빌드 시 플러그인 디렉터리를 사전 채울 수 있습니다. `CLAUDE_CODE_PLUGIN_SEED_DIR` 환경 변수를 설정하여 이 디렉터리를 가리키도록 하세요.

여러 시드 디렉터리를 계층화하려면 Unix에서는 `:`로, Windows에서는 `;`로 경로를 구분하세요. Claude Code는 각 디렉터리를 순서대로 검색하고 지정된 마켓플레이스 또는 플러그인 캐시를 포함하는 첫 번째 시드를 사용합니다.

시드 디렉터리는 `~/.claude/plugins` 구조를 미러링합니다:

```
$CLAUDE_CODE_PLUGIN_SEED_DIR/
  known_marketplaces.json
  marketplaces/<name>/...
  cache/<marketplace>/<plugin>/<version>/...
```

시드 디렉터리를 구축하려면 이미지 빌드 중에 Claude Code를 한 번 실행하고 필요한 플러그인을 설치한 다음 결과 `~/.claude/plugins` 디렉터리를 이미지에 복사하고 `CLAUDE_CODE_PLUGIN_SEED_DIR`이 이를 가리키도록 설정합니다.

복사 단계를 건너뛰려면 빌드 중에 `CLAUDE_CODE_PLUGIN_CACHE_DIR`을 대상 시드 경로로 설정하여 플러그인이 거기에 직접 설치되도록 하세요:

```bash theme={null}
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin marketplace add your-org/plugins
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin install my-tool@your-plugins
```

그런 다음 컨테이너의 런타임 환경에서 `CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed`를 설정하여 시작 시 Claude Code가 시드에서 읽도록 하세요.

시작 시 Claude Code는 시드의 `known_marketplaces.json`에 있는 마켓플레이스를 주 구성에 등록하고 다시 복제하지 않고 `cache/` 아래에 있는 플러그인 캐시를 제자리에 사용합니다. 이는 대화형 모드와 `-p` 플래그가 있는 비대화형 모드 모두에서 작동합니다.

동작 세부 정보:

* **읽기 전용**: 시드 디렉터리에는 절대 쓰지 않습니다. 읽기 전용 파일 시스템에서는 git pull이 실패하므로 시드 마켓플레이스에 대해 자동 업데이트가 비활성화됩니다.
* **시드 항목 우선 적용**: 시드에 선언된 마켓플레이스는 시작할 때마다 사용자의 구성에 일치하는 항목을 덮어씁니다. 시드 플러그인을 옵트아웃하려면 마켓플레이스를 제거하는 대신 `/plugin disable`을 사용하세요.
* **경로 해석**: Claude Code는 시드의 JSON 내부 디렉터리에 저장된 경로를 신뢰하지 않고 런타임에 `$CLAUDE_CODE_PLUGIN_SEED_DIR/marketplaces/<name>/`을 탐색하여 마켓플레이스 콘텐츠를 찾습니다. 즉, 빌드된 위치와 다른 경로에 마운트되더라도 시드가 올바르게 작동합니다.
* **변경 차단됨**: 시드 관리 마켓플레이스에 대해 `/plugin marketplace remove` 또는 `/plugin marketplace update`를 실행하면 시드 이미지를 업데이트하도록 관리자에게 요청하라는 지침과 함께 실패합니다.
* **설정과 합성**: `extraKnownMarketplaces` 또는 `enabledPlugins`가 시드에 이미 존재하는 마켓플레이스를 선언하는 경우 Claude Code는 복제하는 대신 시드 사본을 사용합니다.

### 관리형 마켓플레이스 제한

플러그인 소스에 대한 엄격한 제어가 필요한 조직의 경우 관리자는 관리형 설정의 [`strictKnownMarketplaces`](/docs/en/settings#strictknownmarketplaces) 설정을 사용하여 사용자가 추가할 수 있는 플러그인 마켓플레이스를 제한할 수 있습니다. 단일 실행을 위해 플러그인, 에이전트 및 MCP 서버를 사이드로드하는 CLI 플래그도 거부하려면 [`disableSideloadFlags`](/docs/en/settings#available-settings)와 함께 사용하세요. 어떤 마켓플레이스의 플러그인이 문맥상 설치 제안으로 나타날 수 있는지 허용 목록으로 지정하려면 [`pluginSuggestionMarketplaces`](/docs/en/settings#available-settings)를 설정하세요.

관리형 설정에서 `strictKnownMarketplaces`가 구성된 경우 제한 동작은 값에 따라 다릅니다:

| 값 | 동작 |
| :--- | :--- |
| 미정의 (기본값) | 제한 없음. 사용자가 모든 마켓플레이스를 추가할 수 있음 |
| 빈 배열 `[]` | 완전 잠금. 사용자가 새로운 마켓플레이스를 추가할 수 없음 |
| 소스 목록 | 사용자가 허용 목록과 정확히 일치하는 마켓플레이스만 추가할 수 있음 |

#### 일반적인 구성

모든 마켓플레이스 추가 비활성화:

```json theme={null}
{
  "strictKnownMarketplaces": []
}
```

특정 마켓플레이스만 허용:

```json theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "github",
      "repo": "acme-corp/security-tools",
      "ref": "v2.0"
    },
    {
      "source": "url",
      "url": "https://plugins.example.com/marketplace.json"
    }
  ]
}
```

호스트의 정규식 패턴 일치를 사용하여 내부 git 서버의 모든 마켓플레이스 허용. 이는 [GitHub Enterprise Server](/docs/en/github-enterprise-server#plugin-marketplaces-on-ghes) 또는 자체 호스팅 GitLab 인스턴스에 권장되는 접근 방식입니다:

```json theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "hostPattern",
      "hostPattern": "^github\\.example\\.com$"
    }
  ]
}
```

경로의 정규식 패턴 일치를 사용하여 특정 디렉터리의 파일 시스템 기반 마켓플레이스 허용:

```json theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "pathPattern",
      "pathPattern": "^/opt/approved/"
    }
  ]
}
```

`hostPattern`으로 네트워크 소스를 계속 제어하면서 모든 파일 시스템 경로를 허용하려면 `pathPattern`으로 `".*"`를 사용하세요.

<Note>
  `strictKnownMarketplaces`는 사용자가 추가할 수 있는 항목을 제한하지만 자체적으로 마켓플레이스를 등록하지는 않습니다. 사용자가 `/plugin marketplace add`를 실행하지 않고도 허용된 마켓플레이스를 자동으로 사용할 수 있도록 하려면 동일한 `managed-settings.json`에서 [`extraKnownMarketplaces`](/docs/en/settings#extraknownmarketplaces)와 조합하세요. [둘 다 함께 사용하기](/docs/en/settings#strictknownmarketplaces)를 참조하세요.
</Note>

#### 제한 작동 방식

제한 사항은 네트워크 또는 파일 시스템 작업 전에 검사됩니다. 검사는 마켓플레이스 추가 및 플러그인 설치, 업데이트, 새로 고침 및 자동 업데이트 시 실행됩니다. 정책이 구성되기 전에 마켓플레이스가 추가되었고 해당 소스가 더 이상 허용 목록과 일치하지 않는 경우 Claude Code는 해당 마켓플레이스에서 플러그인을 설치하거나 업데이트하는 것을 거부합니다. 동일한 시행이 `blockedMarketplaces`에도 적용됩니다.

허용 목록은 대부분의 소스 유형에 대해 정확한 일치를 사용합니다. 마켓플레이스가 허용되려면 지정된 모든 필드가 정확히 일치해야 합니다:

* GitHub 소스의 경우: `repo`가 필수이며 허용 목록에 지정된 경우 `ref` 또는 `path`도 일치해야 함
* URL 소스의 경우: 전체 URL이 정확히 일치해야 함
* `hostPattern` 소스의 경우: 마켓플레이스 호스트가 정규식 패턴과 일치함
* `pathPattern` 소스의 경우: 마켓플레이스의 파일 시스템 경로가 정규식 패턴과 일치함

정확한 일치는 URL을 정규화하지 않습니다: 끝 슬래시, `.git` 접미사 또는 `ssh://` 대 `https://` 형식은 서로 다른 값으로 취급됩니다. 조직의 마켓플레이스를 여러 URL 형식으로 복제할 수 있는 경우 모든 형식이 일치하도록 리터럴 URL 대신 `hostPattern` 항목을 선호하세요.

`strictKnownMarketplaces`는 [관리형 설정](/docs/en/settings#settings-files)에 설정되어 있으므로 개별 사용자 및 프로젝트 구성이 이러한 제한을 재정의할 수 없습니다.

지원되는 모든 소스 유형 및 `extraKnownMarketplaces`와의 비교를 포함한 완전한 구성 세부 정보는 [strictKnownMarketplaces 참조](/docs/en/settings#strictknownmarketplaces)를 참조하세요.

### 버전 확인 및 릴리스 채널

플러그인 버전은 캐시 경로 및 업데이트 감지를 결정합니다. 확인된 버전이 사용자가 이미 가지고 있는 버전과 일치하는 경우 `/plugin update` 및 자동 업데이트가 플러그인을 건너끕니다.

Claude Code는 다음 중 설정된 첫 번째 항목에서 플러그인 버전을 확인합니다:

1. 플러그인의 `plugin.json`에 있는 `version`
2. 플러그인의 마켓플레이스 항목에 있는 `version`
3. 플러그인 소스의 git 커밋 SHA

git 기반 소스 유형인 `github`, `url`, `git-subdir` 및 git 호스팅 마켓플레이스 내부의 상대 경로의 경우 `version`을 완전히 생략할 수 있으며 매 새 커밋이 새 버전으로 취급됩니다. 이는 내부 플러그인 또는 활발히 개발 중인 플러그인을 위한 가장 간단한 설정입니다.

<Warning>
  `version`을 설정하면 플러그인이 고정됩니다. `plugin.json`이 `"version": "1.0.0"`을 선언하는 경우 해당 문자열을 변경하지 않고 새 커밋을 푸시해도 기존 사용자에게는 아무것도 발생하지 않습니다. Claude Code가 동일한 버전을 보고 캐시된 복사본을 유지하기 때문입니다. 릴리스마다 버전을 늘리거나 생략하여 커밋 SHA를 사용하세요.

  `plugin.json`과 마켓플레이스 항목 모두에 `version`을 설정하지 마세요. Claude Code는 항상 경고 없이 `plugin.json` 값을 사용하므로 오래된 매니페스트 버전이 `marketplace.json`에 설정한 버전을 마스킹할 수 있습니다.
</Warning>

#### 릴리스 채널 설정

플러그인에 대해 "stable" 및 "latest" 릴리스 채널을 지원하기 위해 동일한 리포지토리의 서로 다른 ref 또는 SHA를 가리키는 두 개의 마켓플레이스를 설정할 수 있습니다. 그런 다음 [관리형 설정](/docs/en/settings#settings-files)을 통해 두 마켓플레이스를 다른 사용자 그룹에 할당할 수 있습니다.

<Warning>
  각 채널은 서로 다른 버전으로 해석되어야 합니다. 명시적 버전을 사용하는 경우 `plugin.json`은 고정된 각 ref에서 서로 다른 `version`을 선언해야 합니다. `version`을 생략하면 고유한 커밋 SHA가 이미 채널을 구별합니다. 두 ref가 동일한 버전 문자열로 해석되는 경우 Claude Code는 이를 동일하게 취급하고 업데이트를 건너끕니다.
</Warning>

##### 예시

```json theme={null}
{
  "name": "stable-tools",
  "plugins": [
    {
      "name": "code-formatter",
      "source": {
        "source": "github",
        "repo": "acme-corp/code-formatter",
        "ref": "stable"
      }
    }
  ]
}
```

```json theme={null}
{
  "name": "latest-tools",
  "plugins": [
    {
      "name": "code-formatter",
      "source": {
        "source": "github",
        "repo": "acme-corp/code-formatter",
        "ref": "latest"
      }
    }
  ]
}
```

##### 사용자 그룹에 채널 할당

관리형 설정을 통해 각 마켓플레이스를 적절한 사용자 그룹에 할당합니다. 예를 들어 안정화 그룹 수신:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "stable-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/stable-tools"
      }
    }
  }
}
```

얼리 액세스 그룹은 대신 `latest-tools`를 받습니다:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "latest-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/latest-tools"
      }
    }
  }
}
```

#### 종속성 버전 고정

플러그인은 종속성 업데이트가 종속 플러그인을 손상시키지 않도록 종속성을 semver 범위로 제약할 수 있습니다. `{plugin-name}--v{version}` git 태그 규칙, 범위 구문 및 동일한 종속성에 대한 여러 제약 조건이 결합되는 방식은 [플러그인 종속성 버전 제약](/docs/en/plugin-dependencies)을 참조하세요.

### 플러그인 이름 변경 또는 제거

플러그인의 `name`은 안정적인 식별자입니다. 사용자는 `enabledPlugins`, `pluginConfigs` 및 `/plugin install` 명령에서 이를 참조하므로 이를 변경하면 기존의 모든 설치가 손상됩니다. 설치를 손상시키지 않고 UI에 표시되는 레이블을 변경하려면 [`displayName`](#optional-plugin-fields)을 설정하고 `name`을 변경되지 않은 상태로 유지하세요.

플러그인의 `name`을 변경해야 하거나 `plugins` 배열에서 플러그인을 제거하는 경우 기존 사용자가 `plugin-not-found` 오류를 보는 대신 마이그레이션되도록 최상위 `renames` 항목을 추가하세요. 자동 마이그레이션에는 Claude Code v2.1.193 이상이 필요합니다. 각 이전 이름을 현재 이름으로 매핑하거나 플러그인이 더 이상 존재하지 않는 경우 `null`로 매핑하세요. 다음 예시는 `formatter`의 이름을 `code-formatter`로 바꾸고 `legacy-linter`가 제거되었음을 기록합니다:

```json theme={null}
{
  "name": "acme-tools",
  "owner": { "name": "Acme" },
  "plugins": [
    { "name": "code-formatter", "source": "./plugins/code-formatter" }
  ],
  "renames": {
    "formatter": "code-formatter",
    "legacy-linter": null
  }
}
```

사용자가 여전히 설정에 이전 이름이 있는 상태로 Claude Code를 시작하면 Claude Code는 `renames` 맵을 따릅니다:

* 항목이 새 이름을 가리키는 경우 Claude Code는 새 이름 아래에서 플러그인을 로드하고 `Renamed to "code-formatter" in the "acme-tools" marketplace`와 같은 한 줄 알림을 표시합니다. 그런 다음 사용자, 프로젝트 및 로컬 설정 범위에서 `enabledPlugins` 및 `pluginConfigs` 모두에 대해 이전 키를 새 키로 다시 작성하므로 알림이 한 번 표시됩니다.
* `null` 항목의 경우 Claude Code는 이전 키를 삭제하고 마켓플레이스에서 플러그인이 제거되었음을 알립니다.
* 이름이 변경된 플러그인이 `github` 또는 `npm`과 같은 원격 소스를 사용하는 경우 Claude Code는 이름 변경 후 `plugin-cache-miss`를 보고하며 사용자는 새 이름으로 가져오기 위해 `/plugin install`을 한 번 실행해야 합니다.

`renames`를 추가 전용 기록으로 취급하세요: 모든 사용자가 마이그레이션했다고 생각한 후에도 이전 항목을 제자리에 유지하세요. Claude Code는 체인을 따르므로 나중에 `code-formatter` 이름을 `formatter-pro`로 변경하려면 첫 번째 항목을 편집하는 대신 두 번째 항목을 추가하세요. 그런 다음 원래 `formatter`가 활성화된 사용자는 두 항목을 통해 `formatter-pro`로 해석됩니다.

맵을 편집한 후 `claude plugin validate .`를 실행하세요. 체인이 사이클을 형성하거나 `null` 또는 `plugins`에 나열된 이름으로 종료되지 않는 항목을 거부합니다.

<Note>
  관리형 및 정책 설정은 Claude Code에 대해 읽기 전용이므로 거기서 활성화된 플러그인은 자동으로 다시 작성될 수 없습니다. 이름이 변경된 플러그인은 각 세션마다 여전히 로드되지만 관리자가 새 이름을 사용하도록 관리형 설정 파일의 `enabledPlugins`를 업데이트할 때까지 이름 변경 알림이 반복됩니다. `--add-dir`과 같은 다른 읽기 전용 소스를 통해 활성화된 플러그인에도 동일하게 적용됩니다.
</Note>

이전 버전의 Claude Code는 `renames` 필드를 무시하고 이전 이름에 대해 `plugin-not-found`를 보고합니다.

## 검증 및 테스트

공유하기 전에 마켓플레이스를 테스트하세요.

마켓플레이스 디렉터리에서 JSON 구문을 검증합니다:

```bash theme={null}
claude plugin validate .
```

또는 Claude Code 내부에서:

```shell theme={null}
/plugin validate .
```

테스트를 위해 마켓플레이스를 추가합니다:

```shell theme={null}
/plugin marketplace add ./path/to/marketplace
```

모든 것이 작동하는지 확인하기 위해 테스트 플러그인을 설치합니다:

```shell theme={null}
/plugin install test-plugin@marketplace-name
```

전체 플러그인 테스트 워크플로는 [로컬에서 플러그인 테스트](/docs/en/plugins#test-your-plugins-locally)를 참조하세요. 기술적인 문제 해결은 [플러그인 참조](/docs/en/plugins-reference)를 참조하세요.

## CLI에서 마켓플레이스 관리

Claude Code는 스크립팅 및 자동화를 위해 비대화형 `claude plugin marketplace` 하위 명령을 제공합니다. 이들은 대화형 세션 내부에서 사용할 수 있는 `/plugin marketplace` 명령과 동등합니다.

### Plugin marketplace add

GitHub 리포지토리, git URL, 원격 URL 또는 로컬 경로에서 마켓플레이스를 추가합니다.

```bash theme={null}
claude plugin marketplace add <source> [options]
```

**인수:**

* `<source>`: GitHub `owner/repo` 축약형, git URL, `marketplace.json` 파일에 대한 원격 URL, 또는 로컬 디렉터리 경로. 브랜치나 태그로 고정하려면 GitHub 축약형에 `@ref`를 추가하거나 git URL에 `#ref`를 추가하세요.

URL에는 스킴이 포함되어야 합니다. Claude Code v2.1.196부터 `gitlab.example.com/team/plugins`와 같이 스킴 없이 입력한 호스트는 잘못된 `owner/repo` 축약형으로 거부되며 오류는 `https://`를 추가하거나 로컬 경로에 `./`를 사용하도록 알려줍니다. 이전 버전은 이를 GitHub 리포지토리 경로로 오독하고 복제 시 GitHub 찾을 수 없음 오류로 실패합니다.

**옵션:**

| 옵션 | 설명 | 기본값 |
| :--- | :--- | :--- |
| `--scope <scope>` | 마켓플레이스를 선언할 위치: `user`, `project`, 또는 `local`. [플러그인 설치 범위](/docs/en/plugins-reference#plugin-installation-scopes)를 참조하세요 | `user` |
| `--sparse <paths...>` | git sparse-checkout을 통해 체크아웃을 특정 디렉터리로 제한합니다. 모노리포에 유용합니다 | |

`owner/repo` 축약형을 사용하여 GitHub에서 마켓플레이스 추가:

```bash theme={null}
claude plugin marketplace add acme-corp/claude-plugins
```

`@ref`로 특정 브랜치 또는 태그에 고정:

```bash theme={null}
claude plugin marketplace add acme-corp/claude-plugins@v2.0
```

GitHub가 아닌 호스트의 git URL에서 추가:

```bash theme={null}
claude plugin marketplace add https://gitlab.example.com/team/plugins.git
```

`marketplace.json` 파일을 직접 제공하는 원격 URL에서 추가:

```bash theme={null}
claude plugin marketplace add https://example.com/marketplace.json
```

테스트를 위해 로컬 디렉터리에서 추가:

```bash theme={null}
claude plugin marketplace add ./my-marketplace
```

`.claude/settings.json`을 통해 팀과 공유되도록 프로젝트 범위에서 마켓플레이스를 선언:

```bash theme={null}
claude plugin marketplace add acme-corp/claude-plugins --scope project
```

모노리포의 경우 플러그인 콘텐츠가 포함된 디렉터리로 체크아웃 제한:

```bash theme={null}
claude plugin marketplace add acme-corp/monorepo --sparse .claude-plugin plugins
```

### Plugin marketplace list

구성된 모든 마켓플레이스를 나열합니다.

```bash theme={null}
claude plugin marketplace list [options]
```

**옵션:**

| 옵션 | 설명 |
| :--- | :--- |
| `--json` | JSON으로 출력 |

`--json`을 사용하면 각 항목에 `name`, `source`, 마켓플레이스가 저장되는 로컬 캐시 경로가 있는 `installLocation` 필드 및 소스별 필드(GitHub 소스의 경우 `repo`, git 및 URL 소스의 경우 `url`, 로컬 소스의 경우 `path`)가 포함됩니다. GitHub 및 git 소스에는 마켓플레이스가 고정된 브랜치나 태그로 추가된 경우 `ref` 필드도 포함됩니다.

### Plugin marketplace remove

구성된 마켓플레이스를 제거합니다. 별칭 `rm`도 허용됩니다.

```bash theme={null}
claude plugin marketplace remove <name> [options]
```

**인수:**

* `<name>`: `claude plugin marketplace list`에 표시된 제거할 마켓플레이스 이름. 이는 `add`에 전달한 소스가 아니라 `marketplace.json`에 있는 `name`입니다.

**옵션:**

| 옵션 | 설명 | 기본값 |
| :--- | :--- | :--- |
| `--scope <scope>` | 단일 설정 범위로 제거 제한: `user`, `project`, 또는 `local`. [플러그인 설치 범위](/docs/en/plugins-reference#plugin-installation-scopes)를 참조하세요. 생략된 경우 편집 가능한 모든 범위에서 선언이 제거됩니다. 주어진 경우 해당 범위의 선언만 제거됩니다. 마켓플레이스가 다른 범위에 여전히 선언되어 있으면 공유 상태, 캐시 및 설치된 플러그인 데이터가 보존됩니다 | (모든 범위) |

<Warning>
  마지막 남은 범위에서 마켓플레이스를 제거하면 거기서 설치한 모든 플러그인도 삭제됩니다. 설치된 플러그인을 잃지 않고 마켓플레이스를 새로 고치려면 대신 `claude plugin marketplace update`를 사용하세요.
</Warning>

### Plugin marketplace update

새로운 플러그인 및 버전 변경 사항을 가져오기 위해 소스에서 마켓플레이스를 새로 고칩니다. 브랜치나 태그 `ref`로 추가된 마켓플레이스는 리포지토리의 기본 브랜치가 아니라 해당 ref의 최신 커밋으로 업데이트됩니다.

```bash theme={null}
claude plugin marketplace update [name]
```

**인수:**

* `[name]`: `claude plugin marketplace list`에 표시된 업데이트할 마켓플레이스 이름. 생략하면 모든 마켓플레이스를 업데이트합니다.

읽기 전용인 시드 관리 마켓플레이스에 대해 실행하면 `remove`와 `update`가 모두 실패합니다. 모든 마켓플레이스를 업데이트할 때 시드 관리 항목은 건너뛰고 다른 마켓플레이스는 계속 업데이트됩니다. 시드 제공 플러그인을 변경하려면 시드 이미지를 업데이트하도록 관리자에게 요청하세요. [컨테이너용 플러그인 사전 채우기](#pre-populate-plugins-for-containers)를 참조하세요.

## 문제 해결

### 마켓플레이스가 로드되지 않음

**증상**: 마켓플레이스를 추가할 수 없거나 마켓플레이스의 플러그인을 볼 수 없음

**해결 방법**:

* 마켓플레이스 URL에 접근할 수 있는지 확인
* 지정된 경로에 `.claude-plugin/marketplace.json`이 존재하는지 검사
* `claude plugin validate` 또는 `/plugin validate`를 사용하여 JSON 구문이 유효한지 확인. 스킬, 에이전트, 명령 프론트매터를 확인하려면 각 플러그인 디렉터리에 대해 명령을 실행합니다.
* 사설 리포지토리의 경우 접근 권한이 있는지 확인

### 마켓플레이스 검증 오류

마켓플레이스 디렉터리에서 `claude plugin validate .` 또는 `/plugin validate .`를 실행하여 문제를 검사합니다. 마켓플레이스 디렉터리를 가리킬 때 검증기는 `marketplace.json`에 스키마 오류, 중복 플러그인 이름 및 소스 경로 순회가 있는지 확인합니다. `source`가 로컬 경로인 각 항목의 경우 플러그인 자체의 `plugin.json`도 검증하고 항목의 `version`이 `plugin.json`에 있는 버전과 일치하지 않을 때 경고합니다. 플러그인의 `plugin.json`에서 발견된 문제에는 `plugins[2] plugin.json →` 형식으로 항목 인덱스가 접두사로 붙습니다.

Claude Code v2.1.196부터 항목별 패스도 다음 작업을 수행합니다:

* `source`가 `.`인 플러그인 포함
* `marketplace.json`이 `.claude-plugin` 디렉터리 외부에 있을 때 실행되어 파일 자체의 디렉터리에 대해 소스 해석
* 파일의 다른 부분에 스키마 오류가 있는 경우에도 각 항목의 문제 보고

이전 버전은 마켓플레이스 루트에 있는 플러그인을 건너뛰고 `.claude-plugin/marketplace.json`에서만 내려갑니다.

개별 플러그인의 `plugin.json` 및 스킬, 에이전트, 명령, 훅 파일을 검증하려면 플러그인 디렉터리 자체에 대해 명령을 실행하세요(예: `claude plugin validate ./plugins/my-plugin`). 일반적인 오류:

| 오류 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| `File not found: .claude-plugin/marketplace.json` | 매니페스트 누락 | 필수 필드가 있는 `.claude-plugin/marketplace.json` 생성 |
| `Invalid JSON syntax: Unexpected token...` | marketplace.json의 JSON 구문 오류 | 누락된 쉼표, 추가 쉼표 또는 따옴표가 없는 문자열 확인 |
| `Duplicate plugin name "x" found in marketplace` | 두 플러그인이 동일한 이름 공유 | 각 플러그인에 고유한 `name` 값 부여 |
| `plugins[0].source: Path contains ".."` | 소스 경로에 `..` 포함 | `..` 없이 마켓플레이스 루트에 상대적인 경로 사용. [상대 경로](#relative-paths) 참조 |
| `YAML frontmatter failed to parse: ...` | 스킬, 에이전트, 명령 파일의 잘못된 YAML | 프론트매터 블록의 YAML 구문 수정. 런타임에 이 파일은 메타데이터 없이 로드됨. 플러그인 디렉터리를 검증할 때만 보고됨 |
| `Invalid JSON syntax: ...` (hooks.json) | 잘못된 형식의 `hooks/hooks.json` | JSON 구문 수정. 잘못된 형식의 `hooks/hooks.json`은 전체 플러그인이 로드되는 것을 방지함. 플러그인 디렉터리를 검증할 때만 보고됨 |

**경고** (비차단):

* `Marketplace has no plugins defined`: `plugins` 배열에 하나 이상의 플러그인 추가
* `No marketplace description provided`: 사용자가 마켓플레이스를 이해하는 데 도움이 되도록 최상위 `description` 추가
* `Plugin name "x" is not kebab-case`: 플러그인 이름에 대문자, 공백 또는 특수 문자가 포함됨. 소문자, 숫자 및 하이픈만으로 이름 변경 (예: `my-plugin`). Claude Code는 다른 형식을 허용하지만 claude.ai 마켓플레이스 동기화는 이를 거부함.

### 플러그인 설치 실패

**증상**: 마켓플레이스는 나타나지만 플러그인 설치가 실패함

**해결 방법**:

* 플러그인 소스 URL에 접근할 수 있는지 확인
* 플러그인 디렉터리에 필수 파일이 포함되어 있는지 확인
* GitHub 소스의 경우 리포지토리가 공개되어 있거나 접근 권한이 있는지 확인
* 복제/다운로드를 통해 플러그인 소스를 수동으로 테스트
* 소스가 `ref`와 `sha`를 모두 고정하는 경우 업스트림 브랜치나 태그가 삭제되어도 GitHub, GitLab, Bitbucket을 포함한 대부분의 git 호스트에서 설치가 차단되지 않습니다. SHA로 커밋을 가져오는 것을 지원하지 않는 서버(예: AWS CodeCommit)에서는 `ref`가 여전히 존재해야 하며 고정된 커밋에 도달할 수 있어야 합니다. 여전히 설치에 실패하는 경우 고정된 커밋이 리포지토리에 계속 존재하는지 확인하세요.

### 사설 리포지토리 인증 실패

**증상**: 사설 리포지토리에서 플러그인을 설치할 때 인증 오류 발생

**해결 방법**:

수동 설치 및 업데이트의 경우:

* git 공급자에 인증되어 있는지 확인 (예: GitHub의 경우 `gh auth status` 실행)
* 자격 증명 도우미가 구성되어 있는지 확인: `git config --global credential.helper`
* git이 스스로 인증할 수 있는지 테스트하기 위해 `git ls-remote <marketplace-url>` 실행. git이 사용자 이름이나 비밀번호를 요청하면 자격 증명을 먼저 저장하세요: HTTPS를 통한 GitHub의 경우 `gh auth setup-git`을 실행하고 SSH 원격의 경우 키를 `ssh-agent`에 로드하세요.

백그라운드 자동 업데이트의 경우:

* 기본적으로 백그라운드 새로 고침은 풀에 대해 git 자격 증명 도우미를 비활성화하므로 풀이 HTTPS를 통해 인증할 수 없습니다. `ssh-agent`에 키가 로드된 SSH 원격은 여전히 인증합니다. 풀이 실패하면 처음부터 재복제가 트리거되며, 이는 저장된 자격 증명을 사용하지만 대규모 리포지토리에서는 시간 초과가 발생할 수 있습니다.
* 백그라운드 풀이 실패할 때 기존 복제본을 유지하려면 `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1`을 설정하세요.
* 재복제 폴백이 인증할 수 있도록 `gh auth setup-git`과 같은 git 자격 증명 도우미를 구성하세요.
* 대규모 리포지토리에서 재복제가 시간 초과되는 경우 [`CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS`](#git-operations-time-out)로 제한을 늘리세요.
* 백그라운드 풀이 직접 인증하도록 마켓플레이스 리포지토리로 범위를 지정하여 [git URL 재작성](#private-repositories)을 구성하세요.
* 또는 자격 증명을 사용하는 `/plugin marketplace update <name>`으로 사설 마켓플레이스를 수동으로 업데이트하세요.

### 오프라인 환경에서 마켓플레이스 업데이트 실패

**증상**: 백그라운드에서 마켓플레이스 `git pull`이 실패하고 Claude Code가 성공할 수 없는 재복제를 반복적으로 시도합니다.

**원인**: 기본적으로 `git pull`이 실패하면 Claude Code는 처음부터 재복제를 시도합니다. 오프라인 또는 에어갭 환경에서는 동일한 방식으로 재복제가 실패하며, 그 후 이전 캐시의 복구는 최선의 노력으로 이루어집니다. 새로 고침은 시작 후 백그라운드에서 실행되므로 시작이 지연되지는 않지만 각 세션은 실패한 시도를 반복하고 각 git 작업은 [120초 시간 초과](#git-operations-time-out)까지 기다릴 수 있습니다.

**해결 방법**: 재복제 시도를 건너뛰고 풀이 실패할 때 기존 캐시를 계속 사용하도록 `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1`을 설정하세요:

```bash theme={null}
export CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1
```

이 변수가 설정되면 Claude Code는 `git pull` 실패 시 오래된 마켓플레이스 복제본을 유지하고 마지막으로 알려진 정상 상태를 계속 사용합니다. 리포지토리에 도달할 수 없는 완전한 오프라인 배포의 경우 대신 [`CLAUDE_CODE_PLUGIN_SEED_DIR`](#pre-populate-plugins-for-containers)를 사용하여 빌드 시 플러그인 디렉터리를 사전 채우세요.

### Git 작업 시간 초과

**증상**: 플러그인 설치 또는 마켓플레이스 업데이트가 "Git clone timed out after 120s" 또는 "Git pull timed out after 120s"와 같은 시간 초과 오류로 실패합니다.

**원인**: Claude Code는 플러그인 리포지토리 복제 및 마켓플레이스 업데이트 가져오기를 포함한 모든 git 작업에 대해 120초 시간 초과를 사용합니다. 대규모 리포지토리나 느린 네트워크 연결은 이 제한을 초과할 수 있습니다.

**해결 방법**: `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` 환경 변수를 사용하여 시간 초과를 늘리세요. 값은 밀리초 단위입니다:

```bash theme={null}
export CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=300000  # 5 minutes
```

### URL 기반 마켓플레이스에서 상대 경로가 있는 플러그인 실패

**증상**: URL(예: `https://example.com/marketplace.json`)을 통해 마켓플레이스를 추가했지만 `"./plugins/my-plugin"`과 같은 상대 경로 소스가 있는 플러그인이 "path not found" 오류로 설치에 실패함.

**원인**: URL 기반 마켓플레이스는 `marketplace.json` 파일 자체만 다운로드합니다. 서버에서 플러그인 파일을 다운로드하지 않습니다. 마켓플레이스 항목의 상대 경로는 다운로드되지 않은 원격 서버의 파일을 참조합니다.

**해결 방법**:

* **외부 소스 사용**: 플러그인 항목을 상대 경로 대신 GitHub, npm 또는 git URL 소스를 사용하도록 변경합니다:
  ```json theme={null}
  { "name": "my-plugin", "source": { "source": "github", "repo": "owner/repo" } }
  ```
* **Git 기반 마켓플레이스 사용**: Git 리포지토리에 마켓플레이스를 호스팅하고 git URL로 추가합니다. Git 기반 마켓플레이스는 전체 리포지토리를 복제하므로 상대 경로가 올바르게 작동합니다.

### 설치 후 파일을 찾을 수 없음

**증상**: 플러그인이 설치되지만 파일 참조가 실패함(특히 플러그인 디렉터리 외부 파일)

**원인**: 플러그인은 제자리에 사용되지 않고 캐시 디렉터리로 복사됩니다. 플러그인 디렉터리 외부의 파일을 참조하는 경로(예: `../shared-utils`)는 해당 파일이 복사되지 않기 때문에 작동하지 않습니다.

**해결 방법**: 심볼릭 링크 및 디렉터리 구조 조정을 포함한 우회 방법은 [플러그인 캐싱 및 파일 확인](/docs/en/plugins-reference#plugin-caching-and-file-resolution)을 참조하세요.

추가 디버깅 도구 및 일반적인 문제는 [디버깅 및 개발 도구](/docs/en/plugins-reference#debugging-and-development-tools)를 참조하세요.

## 관련 정보

* [사전 구축된 플러그인 탐색 및 설치](/docs/en/discover-plugins) - 기존 마켓플레이스에서 플러그인 설치
* [플러그인](/docs/en/plugins) - 자체 플러그인 만들기
* [플러그인 참조](/docs/en/plugins-reference) - 기술 사양 및 스키마 전문
* [플러그인 설정](/docs/en/settings#plugin-settings) - 플러그인 구성 옵션
* [strictKnownMarketplaces 참조](/docs/en/settings#strictknownmarketplaces) - 관리형 마켓플레이스 제한
