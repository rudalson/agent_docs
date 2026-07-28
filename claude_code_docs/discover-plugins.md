> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 마켓플레이스를 통한 사전 제작 플러그인 찾아보기 및 설치하기

> 새 스킬, 에이전트 및 기능으로 Claude Code를 확장하기 위해 마켓플레이스에서 플러그인을 찾아 설치하세요.

플러그인은 스킬, 에이전트, 훅 및 MCP 서버를 사용하여 Claude Code를 확장합니다. 플러그인 마켓플레이스는 이러한 확장 기능을 직접 작성하지 않고 탐색하고 설치할 수 있도록 돕는 카탈로그입니다.

자신만의 마켓플레이스를 생성하고 배포하고 싶으신가요? [플러그인 마켓플레이스 생성 및 배포](/docs/en/plugin-marketplaces)를 참조하세요.

## 마켓플레이스 작동 방식

마켓플레이스는 다른 사람이 작성하고 공유한 플러그인의 카탈로그입니다. 마켓플레이스 사용은 2단계 프로세스로 이루어집니다:

<Steps>
  <Step title="마켓플레이스 추가">
    이렇게 하면 카탈로그가 Claude Code에 등록되어 사용 가능한 항목을 탐색할 수 있습니다. 아직 어떤 플러그인도 설치되지 않습니다.
  </Step>

  <Step title="개별 플러그인 설치">
    카탈로그를 둘러보고 원하는 플러그인을 설치하세요.
  </Step>
</Steps>

앱 스토어를 추가하는 것과 같다고 생각하세요: 스토어를 추가하면 컬렉션을 탐색할 수 있는 접근 권한이 생기지만, 개별적으로 다운로드할 앱은 여전히 직접 선택해야 합니다.

## 공식 Anthropic 마켓플레이스

공식 Anthropic 마켓플레이스(`claude-plugins-official`)는 Claude Code를 시작할 때 자동으로 사용할 수 있습니다. `/plugin`을 실행하고 **Discover** 탭으로 이동하여 사용할 수 있는 항목을 탐색하거나 [claude.com/plugins](https://claude.com/plugins)에서 카탈로그를 확인하세요.

공식 마켓플레이스에서 플러그인을 설치하려면 `/plugin install <name>@claude-plugins-official`을 사용하세요. 예를 들어 GitHub 연동 기능을 설치하려는 경우:

```shell theme={null}
/plugin install github@claude-plugins-official
```

`/plugin`은 터미널 CLI에 대화형 패널을 엽니다. Claude가 이 환경에서 `/plugin`을 사용할 수 없다고 답글을 남기면 Claude 데스크톱 앱의 [플러그인 브라우저](/docs/en/desktop#install-plugins)를 사용하거나 클라우드 세션의 경우 `.claude/settings.json` 내 [`enabledPlugins`](/docs/en/settings#enabledplugins) 아래에 플러그인을 선언하세요.

Claude Code가 `Marketplace "claude-plugins-official" not found`를 보고하는 경우 `/plugin marketplace add anthropics/claude-plugins-official`로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 보고하는 경우 로컬 복사본이 이전 버전입니다: `/plugin marketplace update claude-plugins-official`로 새로 고친 다음 설치를 다시 시도하세요.

<Note>
  공식 마켓플레이스는 Anthropic이 큐레이팅하며 포함 여부는 Anthropic의 재량에 따릅니다. 앱 내 제출 양식은 공식 마켓플레이스가 아닌 [커뮤니티 마켓플레이스](#community-marketplace)에 플러그인을 추가합니다. 플러그인을 독립적으로 배포하려면 [자체 마켓플레이스를 생성](/docs/en/plugin-marketplaces)하여 사용자들과 공유하세요.
</Note>

공식 마켓플레이스에는 여러 카테고리의 플러그인이 포함되어 있습니다:

### 코드 인텔리전스 (Code intelligence)

코드 인텔리전스 플러그인은 Claude Code의 내장 LSP 도구를 활성화하여 편집 직후 Claude가 정의로 이동하고, 참조를 찾고, 타입 오류를 즉시 확인할 수 있도록 합니다. 이러한 플러그인은 VS Code의 코드 인텔리전스를 구동하는 것과 동일한 기술인 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) 연결을 구성합니다.

이러한 플러그인은 언어 서버 실행 파일(binary)이 시스템에 설치되어 있어야 합니다. 언어 서버가 이미 설치되어 있는 경우 프로젝트를 열 때 관련 플러그인을 설치하도록 Claude가 요청할 수 있습니다.

| 언어 | 플러그인 | 필수 실행 파일 |
| :--------- | :------------------ | :--------------------------- |
| C/C++ | `clangd-lsp` | `clangd` |
| C# | `csharp-lsp` | `csharp-ls` |
| Go | `gopls-lsp` | `gopls` |
| Java | `jdtls-lsp` | `jdtls` |
| Kotlin | `kotlin-lsp` | `kotlin-language-server` |
| Lua | `lua-lsp` | `lua-language-server` |
| PHP | `php-lsp` | `intelephense` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |
| Swift | `swift-lsp` | `sourcekit-lsp` |
| TypeScript | `typescript-lsp` | `typescript-language-server` |

다른 언어를 위한 [자신만의 LSP 플러그인을 생성](/docs/en/plugins-reference#lsp-servers)할 수도 있습니다.

<Note>
  플러그인 설치 후 `/plugin` Errors 탭에 `Executable not found in $PATH`가 표시되면 위 표에서 필요한 실행 파일을 설치하세요.
</Note>

#### 코드 인텔리전스 플러그인을 통해 Claude가 얻는 이점

코드 인텔리전스 플러그인이 설치되고 해당 언어 서버 실행 파일을 사용할 수 있게 되면 Claude는 두 가지 기능을 얻습니다:

* **자동 진단(Automatic diagnostics)**: Claude가 수행하는 모든 파일 편집 후 언어 서버가 변경 사항을 분석하고 오류 및 경고를 자동으로 보고합니다. Claude는 컴파일러나 린터를 실행할 필요 없이 타입 오류, 누락된 임포트 및 구문 문제를 확인할 수 있습니다. Claude가 오류를 발생시킨 경우 동일한 턴에서 문제를 감지하고 수정합니다. 이는 플러그인 설치 외에 추가 구성이 필요하지 않습니다. "diagnostics found" 표시기가 나타날 때 **Ctrl+O**를 눌러 인라인 진단 결과를 볼 수 있습니다.
* **코드 탐색(Code navigation)**: Claude는 언어 서버를 사용하여 정의로 이동하고, 참조를 찾고, 호버 시 타입 정보를 가져오고, 기호를 나열하고, 구현을 찾고, 호출 계층 구조를 추적할 수 있습니다. 이러한 작업은 언어 및 환경에 따라 가용성이 다를 수 있지만 grep 기반 검색보다 더 정밀한 탐색을 제공합니다.

문제가 발생하는 경우 [코드 인텔리전스 문제 해결](#code-intelligence-issues)을 참조하세요.

### 외부 연동

이러한 플러그인들은 사전에 구성된 [MCP 서버](/docs/en/mcp)를 번들로 제공하므로 수동 설정 없이 Claude를 외부 서비스에 연결할 수 있습니다:

* **소스 제어**: `github`, `gitlab`
* **프로젝트 관리**: `atlassian` (Jira/Confluence), `asana`, `linear`, `notion`
* **디자인**: `figma`
* **인프라**: `vercel`, `firebase`, `supabase`
* **커뮤니케이션**: `slack`
* **모니터링**: `sentry`

### 자동 보안 검토

`security-guidance` 플러그인은 Claude가 수행하는 각 변경 사항에서 일반적인 취약점을 검토하고 동일한 세션에서 찾은 내용을 수정하도록 Claude에게 지시합니다. 검사 항목과 프로젝트별 규칙 추가 방법은 [Claude가 코드를 작성할 때 보안 문제 포착하기](/docs/en/security-guidance)를 참조하세요.

### 개발 워크플로

일반적인 개발 작업을 위한 스킬과 에이전트를 추가하는 플러그인:

* **commit-commands**: 커밋, 푸시 및 PR 생성을 포함한 Git 커밋 워크플로
* **pr-review-toolkit**: 풀 리퀘스트 검토를 위한 전용 에이전트
* **agent-sdk-dev**: Claude Agent SDK로 구축하기 위한 도구
* **plugin-dev**: 나만의 플러그인을 만들기 위한 도구 모음

### 출력 스타일

Claude의 응답 방식을 설정합니다:

* **explanatory-output-style**: 구현 선택에 대한 교육적 인사이트
* **learning-output-style**: 스킬 향상을 위한 대화형 학습 모드

## 커뮤니티 마켓플레이스

[`anthropics/claude-plugins-community`](https://github.com/anthropics/claude-plugins-community)의 커뮤니티 마켓플레이스는 Anthropic의 자동화된 유효성 검사 및 안전 심사를 통과한 제3자 플러그인을 호스팅합니다. 각 플러그인은 카탈로그의 특정 커밋 SHA에 고정됩니다. 공식 마켓플레이스와 달리 수동으로 추가해야 합니다:

```shell theme={null}
/plugin marketplace add anthropics/claude-plugins-community
```

그런 다음 `claude-community` 마켓플레이스 이름을 사용하여 거기서 플러그인을 설치하세요:

```shell theme={null}
/plugin install <plugin-name>@claude-community
```

커뮤니티 마켓플레이스에 자신만의 플러그인을 제출하려면 플러그인 생성 가이드의 [커뮤니티 마켓플레이스에 플러그인 제출하기](/docs/en/plugins#submit-your-plugin-to-the-community-marketplace)를 참조하세요.

## 시도해 보기: 데모 마켓플레이스 추가

Anthropic은 플러그인 시스템으로 가능한 작업을 보여주는 예제 플러그인이 포함된 [데모 플러그인 마켓플레이스](https://github.com/anthropics/claude-code/tree/main/plugins)(`claude-code-plugins`)도 유지 관리합니다. 공식 마켓플레이스와 달리 이 마켓플레이스는 수동으로 추가해야 합니다.

<Steps>
  <Step title="마켓플레이스 추가">
    Claude Code 내부에서 `anthropics/claude-code` 마켓플레이스에 대한 `plugin marketplace add` 명령을 실행하세요:

    ```shell theme={null}
    /plugin marketplace add anthropics/claude-code
    ```

    이렇게 하면 마켓플레이스 카탈로그가 다운로드되고 해당 플러그인을 사용할 수 있게 됩니다.
  </Step>

  <Step title="사용 가능한 플러그인 탐색">
    `/plugin`을 실행하여 플러그인 관리자를 여세요. 이렇게 하면 **Tab**을 누르거나 **Shift+Tab**으로 되돌아가며 순환할 수 있는 4개의 탭이 포함된 탭 인터페이스가 열립니다:

    * **Discover**: 모든 마켓플레이스에서 사용 가능한 플러그인 탐색
    * **Installed**: 설치된 플러그인 확인 및 관리
    * **Marketplaces**: 추가된 마켓플레이스 추가, 제거 또는 업데이트
    * **Errors**: 플러그인 로딩 오류 확인

    방금 추가한 마켓플레이스의 플러그인을 보려면 **Discover** 탭으로 이동하세요. {/* min-version: 2.1.154 */}관리자가 [`pluginSuggestionMarketplaces`](/docs/en/settings#available-settings) 관리형 설정을 통해 마켓플레이스를 허용 목록에 추가한 경우, 현재 작업 디렉터리와 관련이 있다고 표시된 플러그인이 상단에 **suggested for this directory** 레이블과 함께 고정됩니다.
  </Step>

  <Step title="플러그인 설치">
    플러그인을 선택하여 세부 정보를 확인하세요. 세부 정보 창에는 플러그인에 포함된 항목과 컨텍스트 비용이 표시됩니다:

    * {/* min-version: 2.1.143 */}턴마다 [컨텍스트 창](/docs/en/features-overview#understand-context-costs)에 플러그인이 추가할 토큰 수를 확인할 수 있는 **Context cost** 추정치 (Claude Code v2.1.143 이상)
    * {/* min-version: 2.1.144 */}플러그인의 **Last updated** 날짜 (v2.1.144 이상)
    * {/* min-version: 2.1.145 */}설치하기 전에 추가되는 항목을 정확히 검토할 수 있도록 플러그인의 명령, 에이전트, 스킬, 훅 및 MCP/LSP 서버를 나열하는 **Will install** 섹션 (v2.1.145 이상)

    설치 범위를 선택하세요:

    * **User scope**: 모든 프로젝트에 대해 본인을 위해 설치
    * **Project scope**: 이 리포지토리의 모든 협업자를 위해 설치
    * **Local scope**: 이 리포지토리에서만 협업자와 공유되지 않고 본인을 위해 설치

    예를 들어 git 워크플로 스킬을 추가하는 플러그인인 **commit-commands**를 선택하고 사용자 범위에 설치하세요.

    명령줄에서 설치를 시작할 수도 있습니다:

    ```shell theme={null}
    /plugin install commit-commands@claude-code-plugins
    ```

    범위에 대해 자세히 알아보려면 [구성 범위](/docs/en/settings#configuration-scopes)를 참조하세요.
  </Step>

  <Step title="새 플러그인 사용">
    설치 후 `/reload-plugins`를 실행하여 플러그인을 활성화하세요. 플러그인 스킬은 플러그인 이름으로 네임스페이스가 지정되므로 **commit-commands**는 `/commit-commands:commit`과 같은 스킬을 제공합니다.

    파일을 변경하고 다음을 실행하여 사용해 보세요:

    ```shell theme={null}
    /commit-commands:commit
    ```

    이렇게 하면 변경 사항이 스테이징되고, 커밋 메시지가 생성되며, 커밋이 생성됩니다.

    각 플러그인은 다르게 작동합니다. **Discover** 탭에서 플러그인의 세부 정보를 확인하여 제공하는 명령과 스킬을 확인하거나 사용 지침은 해당 홈페이지를 방문하세요.
  </Step>
</Steps>

이 가이드의 나머지 부분에서는 마켓플레이스를 추가하고, 플러그인을 설치하며, 구성을 관리하는 모든 방법을 다룹니다.

## 마켓플레이스 추가하기

`/plugin marketplace add` 명령을 사용하여 다양한 소스에서 마켓플레이스를 추가하세요.

<Tip>
  **단축키**: `/plugin marketplace` 대신 `/plugin market`을 사용할 수 있으며 `remove` 대신 `rm`을 사용할 수 있습니다.
</Tip>

* **GitHub 리포지토리**: `owner/repo` 형식(예: `anthropics/claude-code`)
* **Git URL**: GitLab, Bitbucket 및 자체 호스팅 서버를 포함한 모든 Git 리포지토리 URL
* **로컬 경로**: 디렉터리 또는 `marketplace.json` 파일로의 직접 경로
* **원격 URL**: 호스팅된 `marketplace.json` 파일로의 직접 URL

### GitHub에서 추가

`owner/repo` 형식을 사용하여 `.claude-plugin/marketplace.json` 파일이 포함된 GitHub 리포지토리를 추가하세요. 여기서 `owner`는 GitHub 사용자 이름 또는 조직이고 `repo`는 리포지토리 이름입니다.

예를 들어 `anthropics/claude-code`는 `anthropics`가 소유한 `claude-code` 리포지토리를 나타냅니다:

```shell theme={null}
/plugin marketplace add anthropics/claude-code
```

### 다른 Git 호스트에서 추가

전체 URL을 제공하여 모든 Git 리포지토리를 추가하세요. 이는 GitLab, Bitbucket 및 자체 호스팅 서버를 포함한 모든 Git 호스트에서 작동합니다. Claude Code가 URL을 호스팅된 `marketplace.json` 파일로의 직접 링크로 취급하지 않고 리포지토리를 복제하도록 `.git` 접미사를 포함하세요.

`https://` 접두사도 포함하세요. Claude Code v2.1.196 이상은 `gitlab.com/company/plugins.git`과 같이 접두사 없이 입력된 호스트를 잘못된 GitHub `owner/repo` 약어로 거부하며 오류 메시지에 접두사를 추가하라고 알려줍니다. 이전 버전은 이를 GitHub 리포지토리 경로로 오인하여 복제 시 실패합니다.

HTTPS 사용:

```shell theme={null}
/plugin marketplace add https://gitlab.com/company/plugins.git
```

SSH 사용:

```shell theme={null}
/plugin marketplace add git@gitlab.com:company/plugins.git
```

특정 브랜치나 태그를 추가하려면 뒤에 `#`과 ref를 붙이세요:

```shell theme={null}
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0
```

### 로컬 경로에서 추가

`.claude-plugin/marketplace.json` 파일이 포함된 로컬 디렉터리를 추가합니다:

```shell theme={null}
/plugin marketplace add ./my-marketplace
```

`marketplace.json` 파일로의 직접 경로를 추가할 수도 있습니다:

```shell theme={null}
/plugin marketplace add ./path/to/marketplace.json
```

### 원격 URL에서 추가

URL을 통해 원격 `marketplace.json` 파일을 추가합니다:

```shell theme={null}
/plugin marketplace add https://example.com/marketplace.json
```

<Note>
  URL 기반 마켓플레이스는 Git 기반 마켓플레이스에 비해 몇 가지 제한 사항이 있습니다. 플러그인을 설치할 때 "path not found" 오류가 발생하는 경우 [문제 해결](/docs/en/plugin-marketplaces#plugins-with-relative-paths-fail-in-url-based-marketplaces)을 참조하세요.
</Note>

## 플러그인 설치하기

마켓플레이스를 추가한 후 플러그인을 직접 설치할 수 있습니다:

```shell theme={null}
/plugin install plugin-name@marketplace-name
```

이 명령은 해당 플러그인의 세부 정보를 열며, 여기서 [설치 범위](/docs/en/settings#configuration-scopes)를 선택합니다. `/plugin`을 실행하고 **Discover** 탭으로 이동한 후 플러그인에서 **Enter**를 누를 때도 동일한 선택이 표시됩니다:

* **User scope**: 모든 프로젝트에 대해 본인을 위해 설치
* **Project scope**: 이 리포지토리의 모든 협업자를 위해 설치되며 `.claude/settings.json`에 플러그인이 추가됨
* **Local scope**: 협업자와 공유되지 않고 이 리포지토리에서만 본인을 위해 설치

대화형 단계 없이 설치하려면 `--scope`를 전달하지 않는 한 사용자 범위에 설치하는 [`claude plugin install`](/docs/en/plugins-reference#plugin-install) 셸 명령을 사용하세요.

**managed** 범위를 가진 플러그인이 표시될 수도 있습니다. 이는 관리자가 [관리형 설정](/docs/en/settings#settings-files)을 통해 설치한 것이며 수정할 수 없습니다.

설치 후 현재 세션에서 플러그인을 활성화하려면 `/reload-plugins`를 실행하세요.

<Warning>
  플러그인을 설치하기 전에 신뢰할 수 있는지 확인하세요. Anthropic은 플러그인에 포함된 MCP 서버, 파일 또는 기타 소프트웨어를 제어하지 않으며 의도대로 작동하는지 검증할 수 없습니다. 자세한 내용은 각 플러그인의 홈페이지를 확인하세요.
</Warning>

## 설치된 플러그인 관리하기

`/plugin`을 실행하고 **Installed** 탭으로 이동하여 플러그인을 확인, 활성화, 비활성화 또는 제거하세요. 목록은 범위별로 그룹화되고 문제를 먼저 볼 수 있도록 정렬됩니다: 로드 오류나 해결되지 않은 종속성이 있는 플러그인이 상단에 나타나고 이어서 즐겨찾기가 표시되며 비활성화된 플러그인은 하단의 접힌 헤더 뒤에 모여 있습니다.

목록에서 다음을 수행할 수 있습니다:

* `f`를 눌러 선택한 플러그인을 즐겨찾기에 추가하거나 해제
* 입력하여 플러그인 이름이나 설명으로 필터링
* Enter를 눌러 플러그인 세부 정보를 열고 활성화, 비활성화 또는 제거

프로젝트의 `.claude/settings.json`에 활성화된 플러그인을 제거할 때 사용할 범위를 묻는 창이 표시됩니다: 사용자 본인만 비활성화(사용자의 `.claude/settings.local.json`에 재정의를 기록하고 프로젝트에 설치된 상태로 유지)하거나, 모두를 위해 제거(공유된 `.claude/settings.json`에서 제거). Claude Code v2.1.203 이상이 필요합니다. v2.1.203 이전에는 대화 상자에서 로컬 비활성화 옵션만 제공했습니다.

세부 정보 뷰는 플러그인이 제공하는 구성 요소인 명령, 스킬, 에이전트, 훅, MCP 서버 및 LSP 서버를 보여줍니다. 동일한 인벤토리는 `claude plugin details` 명령줄에서도 사용할 수 있습니다.

**Installed** 탭에는 직접 설치했지만 최소 10개 세션 동안 2주 이상 사용하지 않은 마켓플레이스 플러그인도 **Not used recently** 헤더 아래에 수집됩니다. 세부 정보 뷰에는 각 플러그인에 대한 **Last used** 줄이 표시됩니다. 더 이상 사용하지 않음에도 시작 및 컨텍스트 비용을 여전히 유발하는 플러그인을 찾는 데 이를 사용한 후 비활성화하거나 제거하세요. Claude Code v2.1.187 이상이 필요합니다.

두 가지 유형의 플러그인은 사용되지 않은 것으로 표시되지 않습니다:

* 조직에서 관리하거나 `--plugin-dir`로 로드하는 플러그인
* 테마, 출력 스타일, 모니터 또는 워크플로를 제공하는 플러그인 (추적할 호출 없이 가치를 제공하기 때문)

조직이 [`strictKnownMarketplaces`](/docs/en/settings#strictknownmarketplaces)로 마켓플레이스를 제한하면 **Not used recently** 헤더와 **Last used** 줄이 모두 숨겨집니다.

플러그인의 [언어 서버](/docs/en/plugins#add-lsp-servers-to-your-plugin)는 진단을 제공하거나 코드 탐색 요청에 응답할 때 사용된 것으로 간주되므로, 세션에서 서버가 활성화된 LSP 플러그인은 미사용으로 표시되지 않습니다. v2.1.203 이전에는 언어 서버 활동을 사용으로 집계할 수 없었으므로 LSP 서버를 제공하는 플러그인은 테마 및 출력 스타일 플러그인과 동일하게 그룹에서 완전히 제외되었습니다.

언어 서버 활동을 집계하는 버전에서의 첫 세션은 사용을 기록하지 않은 각 LSP 플러그인의 사용 기록을 재설정하므로, Claude Code가 이전에 설치한 플러그인을 서버 활동이 추적되기 전에 기록된 데이터를 바탕으로 미사용으로 판단하지 않습니다. v2.1.206 이전에는 첫 세션이 활성 사용 중인 LSP 플러그인을 **Not used recently** 아래에 나열하고 검토를 제안할 수 있었습니다.

종속성을 선언하는 플러그인을 설치하면 설치 출력에 함께 자동 설치된 종속성이 나열됩니다.

직접 명령어로 플러그인을 관리할 수도 있습니다.

메뉴를 열지 않고 설치된 플러그인 나열:

```shell theme={null}
/plugin list
```

해당 상태의 플러그인만 보려면 `--enabled` 또는 `--disabled`를 전달하세요.

제거하지 않고 플러그인 비활성화:

```shell theme={null}
/plugin disable plugin-name@marketplace-name
```

비활성화된 플러그인 다시 활성화:

```shell theme={null}
/plugin enable plugin-name@marketplace-name
```

이러한 식별자에서 `plugin-name`은 [마켓플레이스 항목](/docs/en/plugin-marketplaces#plugin-entries)에서의 플러그인 `name`이며, 이는 플러그인 자체의 `plugin.json`에 있는 `name`과 다를 수 있습니다.

Claude Code v2.1.195부터 `/plugin` 인터페이스의 **Enable** 및 **Disable**은 두 이름이 다른 플러그인에 대해 작동하며, `/plugin enable` 및 `/plugin disable`은 두 이름 모두를 수용합니다. 이전 버전에서 이러한 플러그인을 비활성화하면 Claude Code가 `already disabled`를 보고하고 활성화 상태로 유지합니다.

플러그인 완전히 제거:

```shell theme={null}
/plugin uninstall plugin-name@marketplace-name
```

`--scope` 옵션을 사용하면 CLI 명령으로 특정 범위를 타겟팅할 수 있습니다:

```shell theme={null}
claude plugin install formatter@your-org --scope project
claude plugin uninstall formatter@your-org --scope project
```

### 다시 시작하지 않고 플러그인 변경 사항 적용하기

세션 중에 플러그인을 설치, 활성화 또는 비활성화할 때 재시작 없이 모든 변경 사항을 가져오려면 `/reload-plugins`를 실행하세요:

```shell theme={null}
/reload-plugins
```

Claude Code가 활성 플러그인을 모두 다시 로드하고 플러그인, 스킬, 에이전트, 훅, 플러그인 MCP 서버 및 플러그인 LSP 서버의 개수를 보여줍니다.

다시 로드는 다음 요청에 토큰 비용을 유발합니다: 새롭게 로드된 구성 요소는 대화 뒤에 추가된 콘텐츠에 자신을 알리는 반면, 기존 기록은 여전히 프롬프트 캐시에서 읽습니다. MCP 서버를 제공하는 플러그인은 도구가 [도구 검색(tool search)](/docs/en/mcp#scale-with-mcp-tool-search)에 의해 지연되지 않는 경우 더 많은 비용이 듭니다: 변경 사항으로 인해 캐시가 무효화되고 다음 요청이 전체 대화를 다시 읽습니다. {/* min-version: 2.1.163 */}이 경우 `/reload-plugins`는 경고를 표시하고 다시 로드를 적용하지 않습니다; 그럼에도 적용하려면 `--force`를 전달하세요. 자세한 내용은 [플러그인 활성화 또는 비활성화](/docs/en/prompt-caching#enabling-or-disabling-a-plugin)를 참조하세요.

## 마켓플레이스 관리하기

대화형 `/plugin` 인터페이스나 CLI 명령으로 마켓플레이스를 관리할 수 있습니다.

### 대화형 인터페이스 사용

`/plugin`을 실행하고 **Marketplaces** 탭으로 이동하여 다음을 수행할 수 있습니다:

* 모든 추가된 마켓플레이스를 소스 및 상태와 함께 확인
* 새 마켓플레이스 추가
* 최신 플러그인을 가져오기 위해 마켓플레이스 목록 업데이트
* 더 이상 필요하지 않은 마켓플레이스 제거

### CLI 명령 사용

직접 명령으로 마켓플레이스를 관리할 수도 있습니다.

구성된 모든 마켓플레이스 나열:

```shell theme={null}
/plugin marketplace list
```

마켓플레이스에서 플러그인 목록 새로 고침:

```shell theme={null}
/plugin marketplace update marketplace-name
```

마켓플레이스 제거:

```shell theme={null}
/plugin marketplace remove marketplace-name
```

<Warning>
  마켓플레이스를 제거하면 해당 마켓플레이스에서 설치한 모든 플러그인이 제거됩니다.
</Warning>

### 자동 업데이트 구성

Claude Code는 시작 후 백그라운드에서 마켓플레이스와 설치된 플러그인을 자동으로 업데이트할 수 있습니다. 마켓플레이스에 대해 자동 업데이트가 활성화되면 Claude Code가 마켓플레이스 데이터를 새로 고치고 설치된 플러그인을 디스크의 최신 버전으로 업데이트합니다.

Claude Code는 세션이 시작된 후 최대 10분의 임의 지연시간을 가지고 마켓플레이스 및 플러그인 업데이트를 확인하므로, 실행 중인 세션은 시작 시 로드한 버전을 계속 사용합니다. 플러그인이 업데이트된 경우 `/reload-plugins`를 실행하라는 알림이 표시되거나 다음 실행 시 새 버전이 로드됩니다.

UI를 통해 개별 마켓플레이스의 자동 업데이트 토글:

1. `/plugin`을 실행하여 플러그인 관리자 열기
2. **Marketplaces** 선택
3. 목록에서 마켓플레이스 선택
4. **Enable auto-update** 또는 **Disable auto-update** 선택

공식 Anthropic 마켓플레이스는 기본적으로 자동 업데이트가 활성화되어 있습니다. 제3자 및 로컬 개발 마켓플레이스는 기본적으로 자동 업데이트가 비활성화되어 있습니다.

관리자는 각 사용자가 토글할 필요 없이 조직 마켓플레이스에 대한 자동 업데이트를 활성화하기 위해 관리형 설정의 각 [`extraKnownMarketplaces`](/docs/en/settings#extraknownmarketplaces) 항목에 `"autoUpdate": true`를 설정할 수도 있습니다.

Claude Code와 모든 플러그인의 모든 자동 업데이트를 완전히 비활성화하려면 `DISABLE_AUTOUPDATER` 환경 변수를 설정하세요. 자세한 내용은 [자동 업데이트](/docs/en/setup#auto-updates)를 참조하세요.

Claude Code 자동 업데이트는 비활성화하면서 플러그인 자동 업데이트는 활성화된 상태로 유지하려면 `DISABLE_AUTOUPDATER`와 함께 `FORCE_AUTOUPDATE_PLUGINS=1`을 설정하세요:

```bash theme={null}
export DISABLE_AUTOUPDATER=1
export FORCE_AUTOUPDATE_PLUGINS=1
```

이는 Claude Code 업데이트는 수동으로 관리하면서 플러그인 자동 업데이트는 계속 받고 싶을 때 유용합니다.

## 팀 마켓플레이스 구성

팀 관리자는 `.claude/settings.json`에 마켓플레이스 구성을 추가하여 프로젝트에 대한 자동 마켓플레이스 설치를 설정할 수 있습니다. 팀원이 리포지토리 폴더를 신뢰하면 Claude Code가 이러한 마켓플레이스와 플러그인을 설치하도록 프롬프트를 표시합니다.

Claude Code v2.1.195부터 이 설치 단계는 플러그인을 로드하는 모든 경로에 적용됩니다. 프로젝트의 `.claude/settings.json`만 활성화하고 GitHub 리포지토리나 npm 패키지와 같은 외부 소스에서 가져오는 플러그인은 팀원이 설치할 때까지 로드되지 않습니다. 그때까지 Claude Code는 플러그인이 설치되지 않은 것으로 보고하고 실행할 `claude plugin install` 명령을 보여줍니다.

프로젝트의 `.claude/settings.json`에 `extraKnownMarketplaces`를 추가하세요:

```json theme={null}
{
  "extraKnownMarketplaces": {
    "my-team-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  }
}
```

`extraKnownMarketplaces` 및 `enabledPlugins`를 포함한 전체 구성 옵션은 [플러그인 설정](/docs/en/settings#plugin-settings)을 참조하세요.

## 보안

플러그인과 마켓플레이스는 사용자 권한으로 머신에서 임의의 코드를 실행할 수 있는 높은 수준의 신뢰를 받는 구성 요소입니다. 신뢰하는 소스의 플러그인만 설치하고 마켓플레이스만 추가하세요. 조직은 [관리형 마켓플레이스 제한](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)을 사용하여 사용자가 추가할 수 있는 마켓플레이스를 제한할 수 있습니다.

## 문제 해결

### /plugin 명령이 인식되지 않음

"unknown command"가 표시되거나 `/plugin` 명령이 나타나지 않는 경우:

1. **버전 확인**: `claude --version`을 실행하여 설치된 버전을 확인하세요.
2. **Claude Code 업데이트**:
   * **Homebrew**: `brew upgrade claude-code`, 또는 해당 cask를 설치한 경우 `brew upgrade claude-code@latest`
   * **npm**: `npm install -g @anthropic-ai/claude-code@latest`
   * **네이티브 설치 프로그램**: [설정](/docs/en/setup)에서 설치 명령 다시 실행
3. **Claude Code 다시 시작**: 업데이트 후 터미널을 다시 시작하고 `claude`를 다시 실행하세요.

### 일반적인 문제

* **마켓플레이스가 로드되지 않음**: URL에 접근 가능하며 해당 경로에 `.claude-plugin/marketplace.json`이 존재하는지 확인하세요.
* **플러그인 설치 실패**: 플러그인 소스 URL에 접근 가능하고 리포지토리가 공개되어 있는지, 또는 접근 권한이 있는지 확인하세요.
* **설치 후 파일을 찾을 수 없음**: 플러그인이 캐시로 복사되므로 플러그인 디렉터리 외부의 파일을 참조하는 경로는 작동하지 않습니다.
* **플러그인 스킬이 나타나지 않음**: `rm -rf ~/.claude/plugins/cache`로 캐시를 지우고 Claude Code를 다시 시작한 후 플러그인을 다시 설치하세요.

해결 방법이 포함된 자세한 문제 해결은 마켓플레이스 가이드의 [문제 해결](/docs/en/plugin-marketplaces#troubleshooting)을 참조하세요. 디버깅 도구는 [디버깅 및 개발 도구](/docs/en/plugins-reference#debugging-and-development-tools)를 참조하세요.

### 코드 인텔리전스 문제

* **언어 서버가 시작되지 않음**: 실행 파일이 설치되어 있고 `$PATH`에서 사용할 수 있는지 확인하세요. 자세한 내용은 `/plugin` Errors 탭을 확인하세요.
* **높은 메모리 사용량**: `rust-analyzer` 및 `pyright`와 같은 언어 서버는 대규모 프로젝트에서 상당한 메모리를 소비할 수 있습니다. 메모리 문제가 발생하는 경우 `/plugin disable <plugin-name>`으로 플러그인을 비활성화하고 대신 Claude의 내장 검색 도구에 의존하세요.
* **모노리포에서의 거짓 양성(false positive) 진단**: 워크스페이스가 올바르게 구성되지 않은 경우 언어 서버가 내부 패키지에 대해 미해결 임포트 오류를 보고할 수 있습니다. 이는 Claude의 코드 편집 기능에 영향을 주지 않습니다.

## 다음 단계

* **나만의 플러그인 구축**: 스킬, 에이전트 및 훅을 생성하려면 [플러그인](/docs/en/plugins) 참조
* **마켓플레이스 생성**: 팀이나 커뮤니티에 플러그인을 배포하려면 [플러그인 마켓플레이스 생성](/docs/en/plugin-marketplaces) 참조
* **기술 참조**: 전체 명세는 [플러그인 참조](/docs/en/plugins-reference) 참조
