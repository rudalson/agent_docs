> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 모노레포 또는 대규모 코드베이스에서 Claude Code 설정하기

> 중첩된 CLAUDE.md 파일, 스파스 워크트리(sparse worktrees), 코드 인텔리전스, 패키지별 스킬을 사용하여 모노레포 및 대규모 단일 트립 코드베이스에 대해 Claude Code를 구성함으로써 Claude가 작업 중인 코드에 집중할 수 있도록 하세요.

대규모 코드베이스는 수백만 줄의 코드가 포함된 하나의 저장소이거나 많은 패키지가 포함된 모노레포일 수 있습니다. Claude Code는 모든 규모에서 작동하지만 코드베이스가 커짐에 따라 소규모 프로젝트에 맞춰 조정된 기본 설정이 작업과 무관한 지침과 파일 읽기로 컨텍스트 윈도우를 채워 토큰 비용을 늘리고 Claude의 성능을 저하시킬 수 있습니다.

이 가이드는 개별 개발자와 엔지니어링 팀이 작업이 닿는 코드베이스 범위로 Claude의 범위를 지정하는 방법을 보여줍니다. 각 섹션에는 설정이 로컬 개인 설정인지 저장소에 커밋되는 설정인지 표시되어 있습니다.

## 이 가이드에서 다루는 내용

[아래 표](#settings-on-this-page)는 각 설정과 달성하는 목표를 나열합니다. [뒤따르는 파일 트리](#the-example-monorepo)는 이 페이지의 모든 코드 예제가 참조하는 모노레포 예시입니다.

### 이 페이지의 설정 목록

아래의 각 설정은 독립적입니다. 서로를 대체하기보다 계층화되므로 저장소에 맞는 설정을 적용하세요. [Claude 시작 위치 선택하기](#choose-where-to-start-claude)는 설정 파일이 저장되는 위치를 결정하므로 먼저 읽어보세요. [종합하기](#put-it-together)에는 이 모든 설정이 결합된 형태가 나와 있습니다.

| 원하는 작업                                                                                         | 사용하는 설정                                                                                        |
| :-------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| 전체 서브시스템을 포함하는 하나의 루트 파일 대신 작업 대상 코드의 규칙만 로드 | 디렉토리별 [CLAUDE.md 파일](#layer-claude-md-files-by-directory)                       |
| 작업하지 않는 패키지의 CLAUDE.md 파일 제외                                              | [`claudeMdExcludes`](#exclude-irrelevant-claude-md-files)                                  |
| 빌드 출력, 생성된 코드 및 벤더링된 종속성을 Claude가 열지 못하도록 차단                   | `permissions.deny`의 [`Read` 거부 규칙](#block-reads-of-generated-and-vendored-code)     |
| 파일을 스캔하는 대신 언어 서버를 통해 심볼 정의나 호출자 찾기         | [코드 인텔리전스 플러그인](#reduce-file-reads-with-code-intelligence)                    |
| Claude가 워크트리를 생성할 때 작업에 필요한 디렉토리만 체크아웃                          | [`worktree.sparsePaths`](#check-out-only-the-directories-you-need)                         |
| 동일한 세션에서 형제 패키지나 다른 저장소를 읽고 편집                         | [`--add-dir`](#grant-access-across-packages-or-repositories) 또는 `additionalDirectories`    |
| 관련될 때만 로드되는 특정 영역에 특화된 절차를 Claude에게 제공                            | 디렉토리별 [스킬](#add-per-directory-skills)                                          |
| 많은 디렉토리별 CLAUDE.md 파일을 모두가 설치하는 하나의 공통 규칙으로 대체            | 내부 마켓플레이스의 [플러그인](#centralize-conventions-when-layering-stops-scaling) |

<Tip>
  파일 읽기가 메인 대화에 들어가지 않도록 [서브에이전트에서 탐색 실행](/docs/en/best-practices#use-subagents-for-investigation)하는 것과 같이 모든 저장소에서 컨텍스트를 작게 유지하는 워크플로 기법은 [Claude Code 모범 사례](/docs/en/best-practices)를 참조하세요. 조직의 모든 개발자에게 기준 구성을 배포하려면 [조직을 위한 Claude Code 설정](/docs/en/admin-setup)을 참조하세요.
</Tip>

### 모노레포 예시

이 페이지 전체의 예시는 3개의 패키지가 있는 모노레포를 참조합니다. 동일한 패턴이 대규모 단일 트리 코드베이스에서도 적용됩니다: 예시에서 `packages/api/`를 사용하는 곳에 `src/backend/` 또는 `lib/core/`와 같은 자체 서브시스템 디렉토리로 대체하세요.

```text theme={null}
monorepo/
  CLAUDE.md                     # 루트 지침
  packages/
    api/
      CLAUDE.md                 # API 전용 지침
      .claude/skills/
      src/
    web/
      CLAUDE.md                 # 프론트엔드 전용 지침
      .claude/skills/
      src/
    shared/
      CLAUDE.md                 # 공유 라이브러리 지침
      src/
```

## Claude 시작 위치 선택하기

`claude`를 실행하는 위치에 따라 추가 권한 허가 없이 Claude가 읽고 편집할 수 있는 파일, 시작 시 컨텍스트로 로드되는 CLAUDE.md 파일, 적용되는 프로젝트 설정이 결정됩니다.

| 시작 위치      | 파일 접근 권한                             | 시작 시 로드되는 CLAUDE.md                                           | 사용 시점                                   |
| :-------------- | :-------------------------------------- | :------------------------------------------------------------------- | :----------------------------------------- |
| 저장소 루트 | 모든 파일                              | 루트 전용; 하위 디렉토리 파일은 Claude가 거기서 읽을 때 요청 시 로드 | 작업이 여러 패키지나 서브시스템에 걸쳐 있을 때 |
| 하위 디렉토리  | 해당 하위 트리에만 한정 (추가 허가 전까지) | 해당 디렉토리 및 모든 상위 디렉토리 파일                               | 작업이 하나의 패키지나 서브시스템으로 제한될 때 |

`.claude/settings.json`의 프로젝트 설정은 시작 디렉토리에서만 로드되며 CLAUDE.md 파일처럼 상위 디렉토리에서 상속되지 않습니다: 저장소 루트의 `.claude/settings.json`은 루트에서 시작할 때만 적용됩니다.

아래의 각 섹션에는 해당 설정 파일이 저장소 루트에 속하는지 또는 시작하는 하위 디렉토리에 속하는지, 그리고 커밋되는지 로컬로 유지되는지 명시되어 있습니다.

## 디렉토리별 CLAUDE.md 파일 계층화

대규모 코드베이스에서 저장소 루트의 단일 CLAUDE.md는 모든 서브시스템의 규칙을 담도록 커져 현재 작업과 무관한 지침으로 컨텍스트 비용을 지불하거나, 너무 일반적이어서 쓸모가 없어지는 경향이 있습니다. 지침을 디렉토리별 파일로 분할하면 Claude가 저장소 전체 규칙에 더해 작업 중인 코드의 규칙만 로드하게 됩니다.

Claude Code는 시작 시 작업 디렉토리 및 모든 상위 디렉토리에서 모든 [CLAUDE.md](/docs/en/memory) 파일을 로드한 다음, 해당 위치의 파일을 읽을 때 요청 시 각 하위 디렉토리의 파일을 로드합니다. 루트 파일은 저장소 전체 규칙을 설정하고 각 하위 디렉토리는 자체 규칙을 추가합니다.

일반적인 분할은 2단계 방식입니다:

* **루트 `CLAUDE.md`**: 코딩 표준, 커밋 규칙, 저장소 레이아웃과 같이 모든 곳에 적용되는 지침
* **하위 디렉토리별 `CLAUDE.md`**: 해당 영역 스택에 특화된 규칙. 모노레포에서는 패키지당 1개. 대규모 단일 트리에서는 `src/db/` 또는 `src/api/`와 같은 서브시스템당 1개

팀원이 상속받을 수 있도록 이 파일들을 저장소에 커밋하세요. 일반적으로 각 디렉토리 소유자가 해당 파일을 관리합니다.

루트 `CLAUDE.md`는 Claude에게 저장소 구조를 오리엔테이션합니다:

```markdown CLAUDE.md theme={null}
This is a monorepo with three packages under packages/:

- packages/api: Node.js REST API with Express, TypeScript, and PostgreSQL
- packages/web: React frontend with Vite, TypeScript, and TailwindCSS
- packages/shared: shared TypeScript utilities used by both api and web

Run commands from the package directory, not the monorepo root.
Each package has its own tsconfig.json, package.json, and test suite.
```

각 하위 디렉토리의 `CLAUDE.md` (여기서는 `packages/api/CLAUDE.md`)는 해당 영역 스택에 특화된 컨텍스트를 추가합니다:

```markdown packages/api/CLAUDE.md theme={null}
This package is the REST API server.

- Run tests: `npm test` (uses Vitest)
- Run dev server: `npm run dev` (port 3001)
- Database migrations: `npm run migrate`
- Environment variables: copy `.env.example` to `.env`

API routes are in src/routes/. Each route file exports an Express router.
Database queries use Knex in src/db/. Never write raw SQL strings in route handlers.
```

`packages/api/`에서 Claude를 시작하면 `packages/api/CLAUDE.md`와 루트 `CLAUDE.md`가 모두 로드됩니다. Claude는 저장소 전체 규칙과 함께 로컬 지침을 보며, `packages/web/`에서의 지침은 컨텍스트에 포함되지 않습니다. 모노레포가 아닌 트리의 모든 하위 디렉토리에도 동일하게 적용됩니다. 로드된 파일을 확인하려면 `/context`를 실행하고 **Memory files** 아래의 목록을 확인하세요.

코드베이스와 모델이 변경됨에 따라 파일을 최신 상태로 유지하는 몇 가지 방법:

* **풀 리퀘스트 시 검토**: 규칙이 코드를 따르도록 CLAUDE.md 편집을 다른 문서 변경 사항처럼 취급하세요
* **주요 모델 출시 후 재검토**: 이전 모델의 한계를 우회하던 지침은 새 모델이 자체적으로 처리하게 되면 오버헤드가 될 수 있습니다. 예를 들어, 단일 파일 리팩토링을 강제하는 규칙은 제한이 사라지면 삭제할 수 있습니다.
* **업데이트를 제안하는 Stop 훅 추가**: [`Stop` 훅](/docs/en/hooks#stop)은 Claude가 응답을 마쳤을 때 세션 트랜스크립트 경로를 받으므로, 스크립트가 세션을 검토하고 노출된 공백이 생생할 때 CLAUDE.md 업데이트를 제안할 수 있습니다.

CLAUDE.md 파일이 로드되고 상호작용하는 방식에 대한 자세한 내용은 [메모리 및 프로젝트 지침](/docs/en/memory)을 참조하세요.

### 디렉토리별 CLAUDE.md와 경로 범위 규칙 비교

디렉토리별 `CLAUDE.md` 파일과 `.claude/rules/` 아래의 [경로 범위 규칙](/docs/en/memory#path-specific-rules)은 모두 지침을 트리의 일부에 타겟팅할 수 있게 해줍니다. 이 둘은 파일이 저장되는 위치와 로드되는 시점에서 차이가 있습니다.

| 접근 방식                             | 파일 위치                            | 로드 시점                                                                              | 사용 시점                                                                                  |
| :----------------------------------- | :--------------------------------------- | :-------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------- |
| 디렉토리별 `CLAUDE.md`            | 해당 디렉토리 내부, 코드와 함께 | 해당 디렉토리에서 시작할 때 시작 시, 또는 Claude가 거기서 파일을 읽을 때 요청 시 | 디렉토리 소유자가 자체 규칙을 관리할 때; 지침이 코드와 함께 버전 관리될 때 |
| `.claude/rules/`의 경로 범위 규칙 | 저장소 루트의 중앙 `.claude/`      | Claude가 규칙의 `paths:` 글로브와 일치하는 파일로 작업할 때                         | 모든 규칙을 한곳에 유지하고 싶거나 동일한 규칙이 흩어진 여러 경로에 적용될 때   |

스킬까지 포함된 비교는 [유사한 기능 비교](/docs/en/features-overview#compare-similar-features)를 참조하세요.

### 무관한 CLAUDE.md 파일 제외

저장소 루트에서 Claude를 시작하면 하위 디렉토리의 CLAUDE.md는 Claude가 해당 디렉토리에서 파일을 읽는 즉시 로드됩니다. `claudeMdExcludes` 설정은 특정 파일의 경로나 글로브 패턴을 건너뛰어 로드되지 않도록 합니다.

다른 팀의 패키지, 레거시 코드, 벤더링된 하위 트리 등 전혀 작업하지 않는 디렉토리에 이를 사용하세요. 제외 목록은 정적이며 작업별 스위치가 아닙니다. 오늘 한 패키지에 집중하고 내일 다른 패키지에 집중하려면 제외 목록을 편집하는 대신 [해당 패키지 디렉토리에서 Claude를 시작](#choose-where-to-start-claude)하세요.

자신에게만 이러한 제외를 적용하려는 경우 `.claude/settings.local.json`에 설정을 추가하세요. Claude Code가 생성할 때 해당 파일을 gitignore합니다; 직접 생성하는 경우 gitignore에 추가하세요. 패턴은 절대 파일 경로와 비교되는 글로브 구문을 사용하므로 트리 어느 곳에서나 일치시키려면 상대적 스타일 패턴을 `**/`로 시작하세요. 아래 예시는 다른 팀이 소유한 패키지를 제외합니다:

```json .claude/settings.local.json theme={null}
{
  "claudeMdExcludes": [
    "**/packages/web/**"
  ]
}
```

이렇게 하면 해당 패키지 아래의 모든 CLAUDE.md 및 규칙 파일이 건너뛰어집니다. 루트 CLAUDE.md 및 실제 작업하는 패키지는 정상적으로 로드됩니다.

이 패턴들은 다른 일반적인 케이스를 다룹니다:

* `"**/packages/*/CLAUDE.md"`: 루트는 유지하면서 모든 패키지의 CLAUDE.md를 제외
* `"**/packages/legacy-*/**"`: 규칙을 포함하여 이름이 글로브와 일치하는 모든 패키지 제외
* `"/home/user/monorepo/legacy/CLAUDE.md"`: 절대 경로로 특정 파일 하나 제외

관리형 정책 CLAUDE.md 파일은 제외할 수 없으므로 조직 전체 지침이 항상 적용됩니다. 사용자, 프로젝트, 로컬, 관리형과 같은 모든 [설정 범위](/docs/en/settings#configuration-scopes)에서 `claudeMdExcludes`를 설정할 수 있습니다. 배열은 여러 범위 간에 병합되므로 팀이 프로젝트 수준 기본값을 설정하고 개인별로 로컬 재정의를 추가할 수 있습니다.

전체 제외 문서는 [특정 CLAUDE.md 파일 제외](/docs/en/memory#exclude-specific-claude-md-files)를 참조하세요.

## Claude가 읽는 양 줄이기

지침은 Claude의 컨텍스트에 들어가는 항목의 일부일 뿐입니다. 파일 읽기는 코드베이스와 함께 커지는 또 다른 비용입니다. 아래 설정은 무관한 경로의 읽기를 차단하고 소모적인 파일 스캔을 언어 서버 조회로 대체합니다.

### 생성된 코드 및 벤더링된 코드의 읽기 차단

Claude의 콘텐츠 검색은 기본적으로 `.gitignore`를 준수하므로 `node_modules/`, `dist/`, `build/` 등 거기에 등록된 경로는 추가 구성 없이 검색 결과에서 제외됩니다.

벤더링된 SDK나 커밋된 생성 코드와 같이 체크인되어 있는 경로의 경우 `permissions.deny`에 `Read` 거부 규칙을 추가하여 검색 결과에 해당 파일이 나열되더라도 Claude가 열지 못하도록 차단하세요.

거부 규칙은 설정 파일을 어디에 넣는지에 따라 저장소에서 작업하는 모든 사람, 자신만, 또는 컴퓨터의 모든 세션에 적용될 수 있습니다:

* **저장소에서 작업하는 모든 사람**: `.claude/settings.json`에 규칙을 커밋합니다. 이 페이지의 다른 프로젝트 설정과 마찬가지로 해당 파일은 시작 디렉토리에서만 로드되므로, 루트에서 시작하는 경우 저장소 루트에 넣거나 하위 디렉토리에서 시작하는 경우 각 패키지의 `.claude/`에 넣으세요.
* **자신만**: 시작 디렉토리와 관계없이 저장소 내부의 모든 CLI 세션에서 로드되는 저장소 루트의 `.claude/settings.local.json`을 사용합니다. 예시의 `Read(./vendor/**)`와 같은 상대 패턴은 여전히 [Claude Code를 시작한 디렉토리에 고정](/docs/en/permissions#read-and-edit)되므로 하위 디렉토리에서 세션을 시작하는 경우 `Read(//absolute/path/to/repo/vendor/**)`와 같이 `//`-절대 경로로 작성하세요.
* **모든 사람에게 강제**: 사용자 및 프로젝트 설정이 재정의할 수 없는 [관리형 설정](/docs/en/settings#settings-files)에 규칙을 설정합니다.

아래 예시는 빌드 아티팩트 및 벤더링된 SDK를 차단합니다:

```json .claude/settings.json theme={null}
{
  "permissions": {
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)",
      "Read(./**/*.generated.*)",
      "Read(./vendor/**)"
    ]
  }
}
```

거부 규칙은 인수로 차단된 경로가 전달될 때 Claude의 내장 파일 도구 및 `cat`, `head`, `grep`, `find` 등 인식된 Bash 파일 명령을 적용 대상으로 합니다. 재귀 검색의 출력에서 차단된 경로를 필터링하지는 않으며, 직접 파일을 열어보는 임의의 하위 프로세스는 다루지 않습니다. 전체 패턴 구문은 [Read 및 Edit 권한 규칙](/docs/en/permissions#read-and-edit)을 참조하세요.

### 코드 인텔리전스로 파일 읽기 줄이기

대규모 코드베이스에서 심볼이 정의되었거나 사용된 위치를 찾는 것은 많은 파일 읽기 및 grep 호출을 필요로 할 수 있습니다. [코드 인텔리전스 플러그인](/docs/en/discover-plugins#code-intelligence)은 Claude를 언어 서버에 연결하여 트리를 스캔하는 대신 정의로 바로 이동하고 참조를 찾으며 타입 오류를 직접 표면화할 수 있도록 합니다.

공식 마켓플레이스에는 TypeScript, Python, Go, Rust 및 기타 일반적인 언어용 플러그인이 있습니다. Claude Code 세션 내부에서 아래 명령을 실행하여 TypeScript 플러그인을 설치하세요:

```shell theme={null}
/plugin install typescript-lsp@claude-plugins-official
```

Claude Code가 `Marketplace "claude-plugins-official" not found`를 보고하는 경우 `/plugin marketplace add anthropics/claude-plugins-official`로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 보고하면 로컬 사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고치세요. 그 후 설치를 재시도하세요.

직접 설치하는 대신 저장소의 모든 사용자에 대해 플러그인을 활성화하려면 [`enabledPlugins` 프로젝트 설정](/docs/en/settings#plugin-settings)에 이를 추가하세요.

코드 인텔리전스 플러그인은 각 개발자의 컴퓨터에 해당 언어의 언어 서버 바이너리가 필요합니다. [각 언어에 필요한 바이너리](/docs/en/discover-plugins#code-intelligence)를 확인하세요. 공식 마켓플레이스에서 설치하려면 마켓플레이스가 호스팅되는 GitHub에 대한 네트워크 접근이 필요합니다. 제한된 네트워크의 경우 [내부 Git 호스트 또는 로컬 경로에서 마켓플레이스 추가](/docs/en/discover-plugins#add-from-other-git-hosts)를 대신 사용하세요.

이는 `claudeMdExcludes` 및 위의 `Read` 거부 규칙과 잘 조합됩니다. 이러한 규칙은 무관한 콘텐츠를 컨텍스트에서 오려내고, 코드 인텔리전스는 Claude가 정의를 찾기 위해 남은 파일들을 뒤지는 것을 막아 줍니다.

## 워크트리 및 파일 접근 범위 제한

이 설정들은 워크트리의 디스크 상태와 시작점 외에 Claude가 읽고 쓸 수 있는 디렉토리를 제어합니다.

### 필요한 디렉토리만 체크아웃

`--worktree` 플래그는 사용자의 메인 체크아웃과 변경 사항을 격리하여 새 git 워크트리에서 세션을 시작합니다. 기본적으로 전체 저장소를 체크아웃합니다. 대규모 저장소에서 `worktree.sparsePaths` 설정은 git sparse-checkout을 사용하여 나열된 디렉토리와 루트 수준 파일만 디스크에 쓰므로 워크트리가 더 빨리 시작되고 공간을 덜 사용합니다.

이 디렉토리에서 작업하는 모든 사용자에게 동일한 경로가 필요한 경우 `.claude/settings.json`에 설정을 커밋하세요. 자신만의 경로를 추가하려면 `.claude/settings.local.json`을 사용하세요: 목록은 여러 범위에 걸쳐 병합되므로 로컬 파일이 커밋된 목록에 경로를 추가할 수는 있지만 제거할 수는 없습니다.

이 페이지의 JSON 예시는 한 번에 하나의 설정을 보여줍니다. `.claude/settings.json`에 위의 `permissions.deny` 규칙과 같은 다른 키가 이미 포함되어 있는 경우 파일을 교체하지 말고 그 옆에 `worktree` 키를 추가하세요. [종합하기](#put-it-together)는 결합된 결과를 보여줍니다.

아래 예시는 커밋된 파일을 보여줍니다:

```json .claude/settings.json theme={null}
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ]
  }
}
```

Claude가 워크트리를 생성할 때 전체 트리 대신 `.claude/`, `packages/api/`, `packages/shared/`만 체크아웃합니다. `sparsePaths`의 경로는 Claude를 시작하는 하위 디렉토리와 관계없이 저장소 루트에 상대적입니다. 패키지 루트뿐만 아니라 모든 디렉토리 경로가 여기서 작동합니다.

이는 [서브에이전트 워크트리 격리](/docs/en/worktrees#isolate-subagents-with-worktrees)에 특히 유용합니다. 서브에이전트는 하위 작업을 위해 생성되는 병렬 Claude 인스턴스이며, 워크트리에서 실행되는 각 인스턴스는 전체 트리 대신 경량 체크아웃을 받습니다. 세션의 모든 워크트리는 동일한 `sparsePaths`를 공유하므로 한 서브에이전트에 `packages/api/`가 필요하고 다른 서브에이전트에 `packages/web/`이 필요한 경우 둘 다 나열하세요.

개별 파일이 아닌 디렉토리를 `sparsePaths`에 나열하세요. `package.json`, `tsconfig.base.json`, 잠금 파일과 같은 루트 수준 파일은 나열된 디렉토리와 함께 항상 체크아웃됩니다. 루트 수준 디렉토리는 그렇지 않으므로 워크트리 내부에서 저장소 루트의 `.claude/settings.json`, `.claude/rules/` 또는 `.claude/skills/`를 사용하려면 목록에 `.claude`를 포함하세요.

스파스 워크트리가 존재하는 동안 스파스 체크아웃을 하려면 git이 저장소의 공유 `.git/config`에서 `extensions.worktreeConfig`를 활성화해야 합니다. Claude Code는 마지막 워크트리가 제거된 후 해당 항목을 제거하지만 Claude Code가 추가한 경우에만 해당합니다. 사용자가 직접 설정한 값은 절대 제거하지 않습니다.

워크트리 전체에서 `node_modules`와 같은 대용량 디렉토리가 중복 생성되는 것을 방지하려면 동일한 `.claude/settings.json`에서 `sparsePaths`와 `symlinkDirectories`를 짝지어 사용하세요:

```json .claude/settings.json theme={null}
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ],
    "symlinkDirectories": [
      "node_modules"
    ]
  }
}
```

이렇게 하면 디스크에 중복 생성되는 대신 각 워크트리의 `node_modules/`에서 메인 저장소 사본으로의 심볼릭 링크가 생성됩니다.

<Note>
  `sparsePaths` 및 `symlinkDirectories` 설정은 워크트리가 생성되기 전에 시작 디렉토리에서 읽습니다. 생성 후 세션의 작업 디렉토리는 실행한 하위 디렉토리가 아니라 워크트리 루트가 됩니다. 따라서 워크트리 내부의 프로젝트 설정은 워크트리 루트의 `.claude/settings.json`(저장소 루트 파일의 체크아웃 사본)에서 로드됩니다. 권한 규칙이나 훅과 같이 워크트리 내부에서 필요한 기타 설정은 저장소 루트의 `.claude/settings.json`에 배치하세요.
</Note>

전체 워크트리 설정 레퍼런스는 [워크트리 설정](/docs/en/settings#worktree-settings)을 참조하세요.

### 패키지 또는 저장소 간 접근 권한 허가

이 섹션은 하위 디렉토리에서 Claude를 시작하거나 작업이 여러 체크아웃에 걸쳐 있을 때 적용됩니다. 단일 대규모 트리에서 저장소 루트부터 시작하는 경우 Claude는 이미 모든 파일에 대한 접근 권한을 가지고 있으므로 이 단계를 건너뛸 수 있습니다.

`packages/api/`에서 Claude를 시작하면 해당 디렉토리 내의 파일을 읽고 쓸 수 있습니다. 작업에서 `api`와 `web`이 모두 가져오는 공유 타입을 업데이트하는 등 패키지 간 변경이 필요한 경우 형제 디렉토리에 접근 권한을 허가해야 합니다. 동일한 메커니즘으로 별도로 체크아웃된 저장소에 대한 접근 권한을 허가합니다.

`.claude/settings.json`에 있는 `additionalDirectories` 설정은 Claude에게 작업 디렉토리 외부 디렉토리에 대한 접근 권한을 부여합니다. 아래 예시는 두 형제 패키지에 대한 접근 권한을 허가합니다:

```json packages/api/.claude/settings.json theme={null}
{
  "permissions": {
    "additionalDirectories": [
      "../shared",
      "../web"
    ]
  }
}
```

상대 경로는 Claude를 시작하는 디렉토리를 기준으로 해제됩니다. 이 구성을 사용하면 Claude가 `packages/api/`에서 작업하는 동안 `packages/shared/` 및 `packages/web/`의 파일을 읽고 편집할 수 있습니다.

설정을 편집하지 않고 시작할 때 `--add-dir`을 전달하여 런타임에 접근 권한을 부여할 수도 있습니다:

```bash theme={null}
claude --add-dir ../shared
```

디렉토리를 어떻게 추가하든 Claude는 디렉토리의 파일을 읽고 편집할 수 있습니다. 해당 디렉토리의 CLAUDE.md, `.claude/rules/` 파일 및 스킬도 함께 로드되는지 여부는 추가한 방법에 따라 다릅니다:

| 추가 방법                             | CLAUDE.md 및 규칙 로드 여부                | 스킬 로드 여부 |
| :------------------------------------- | :--------------------------------------- | :----------- |
| `additionalDirectories` 설정        | 로드 안 함                                    | 로드 안 함        |
| `--add-dir` 플래그 또는 `/add-dir` 명령 | 아래 환경 변수가 있을 때만 | 로드함          |

`--add-dir` 또는 `/add-dir`로 추가된 디렉토리에서 CLAUDE.md 및 규칙 파일을 로드하려면 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` 환경 변수를 설정하세요:

```bash theme={null}
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared
```

환경 변수는 `additionalDirectories` 설정에 나열된 디렉토리에는 영향을 미치지 않습니다. 자세한 내용은 [추가 디렉토리에서 로드](/docs/en/memory#load-from-additional-directories)를 참조하세요.

이 영역의 모든 사람에게 필요한 형제 디렉토리의 경우 `additionalDirectories`를 `.claude/settings.json`에 커밋하세요. 개인적인 선택이나 일회성 접근의 경우 `.claude/settings.local.json`을 사용하거나 시작 시 `--add-dir`을 전달하세요.

## 디렉토리별 스킬 추가

모든 하위 디렉토리는 자체 스택으로 범위가 지정된 [스킬](/docs/en/skills)을 정의할 수 있습니다. 스킬은 Claude가 관련이 있다고 판단할 때 요청 시 로드되므로 API 전용 도구가 프론트엔드 작업 중에 컨텍스트를 소비하지 않습니다.

스킬은 디렉토리 내부의 `.claude/skills/` 아래에 위치합니다. 해당 영역의 코드와 함께 커밋하여 저장소를 복제하는 누구나 스킬을 얻을 수 있도록 하세요. 모노레포에서는 패키지당 하나의 스킬 세트가 될 수 있습니다. 대규모 단일 트리 코드베이스에서는 `src/db/.claude/skills/`와 같이 서브시스템당 하나의 세트가 됩니다.

하위 디렉토리 내에 스킬 디렉토리를 만듭니다:

```bash theme={null}
mkdir -p packages/api/.claude/skills/api-testing
```

그런 다음 해당 디렉토리(여기서는 `packages/api/.claude/skills/api-testing/SKILL.md`)에 `SKILL.md`를 작성합니다. 이 예시는 API 패키지의 테스트 패턴을 Claude에게 가르칩니다:

```markdown packages/api/.claude/skills/api-testing/SKILL.md theme={null}
---
name: api-testing
description: Testing patterns for the API package. Use when writing or modifying tests in packages/api/.
---

## Test structure

Tests are in `src/__tests__/` mirroring the `src/` directory structure.
Each route file has a corresponding `.test.ts` file.

## Running tests

- All tests: `npm test`
- Single file: `npm test -- src/__tests__/routes/users.test.ts`
- Watch mode: `npm test -- --watch`

## Test utilities

- `src/__tests__/helpers/db.ts`: provides `setupTestDb()` and `teardownTestDb()` for database tests
- `src/__tests__/helpers/auth.ts`: provides `createTestUser()` and `getAuthToken()` for authenticated endpoints

## Patterns

- Use `supertest` for HTTP assertions, not raw fetch
- Always wrap database tests in a transaction that rolls back
- Mock external services in `src/__tests__/mocks/`
```

서로 다른 하위 디렉토리는 동일한 방식으로 서로 다른 스킬을 가집니다: `packages/web/.claude/skills/component-patterns/`는 테스트 대신 프론트엔드의 컴포넌트 규칙을 설명합니다. Claude가 `packages/api/`의 파일에서 작업할 때 api-testing 스킬이 로드됩니다. `packages/web/`에서 작업할 때는 대신 component-patterns가 로드됩니다. 서로의 작업 중에는 상대 디렉토리의 스킬이 로드되지 않습니다.

위치 대신 파일 패턴으로 스킬의 범위를 지정할 수도 있습니다. [`paths` 프론트매터 필드](/docs/en/skills#frontmatter-reference)는 글로브 패턴을 지원하며, Claude는 일치하는 파일로 작업할 때만 스킬을 자동으로 로드합니다. 저장소 루트의 `.claude/skills/`에 위치하지만 `**/migrations/**`로 범위가 지정된 데이터베이스 마이그레이션 스킬처럼 위치에 상관없이 특정 파일에만 적용되는 스킬에 이 방식을 사용하세요.

스킬 작성 및 구성에 대한 자세한 내용은 [스킬](/docs/en/skills)을 참조하세요.

### 스킬 검색 가능성 유지

스킬이 많은 디렉토리에 퍼져 있으면 Claude가 선택하는 목록이 커질 수 있습니다. Claude는 발견된 모든 스킬의 이름과 설명을 읽어 스킬을 선택하고, 선택된 스킬의 전체 내용만 컨텍스트로 로드됩니다. 이 섹션에서는 해당 목록을 작게 유지하고 축약 시에도 살아남는 설명을 작성하는 방법을 다룹니다.

어떤 스킬이 범위 내에 있는지는 Claude를 시작하는 위치에 따라 달라집니다:

* **`packages/api/`와 같은 하위 디렉토리에서**: 해당 디렉토리, 저장소 루트까지의 모든 상위 디렉토리, 사용자 및 엔터프라이즈 수준의 스킬
* **저장소 루트에서**: 세션 중 Claude가 만지는 모든 하위 디렉토리의 스킬 (수백 개로 누적될 수 있음)
* **[`--add-dir`](#grant-access-across-packages-or-repositories)로 형제를 추가한 후**: 해당 형제의 스킬도 로드됩니다. `additionalDirectories` 설정은 파일 접근 권한만 부여하며 스킬은 로드하지 않습니다.

이름은 항상 로드되지만 [스킬이 많으면 설명이 축약](/docs/en/skills#skill-descriptions-are-cut-short)되므로, 스킬 적용 여부를 결정할 때 Claude가 사용하는 키워드가 잘려 나갈 수 있습니다. 설명을 짧게 유지하고 "writing or modifying tests in `packages/api/`"와 같이 요청에 포함될 법한 단어로 시작하세요.

PR 규칙이나 배포 체크리스트처럼 많은 디렉토리가 공유하는 스킬의 경우 어떤 시작 디렉토리에서든 로드될 수 있도록 저장소 루트의 `.claude/skills/`에 배치하세요. 공유 스킬에 자체 버전 내역이 필요하거나 여러 저장소에 걸쳐 작동해야 하는 경우 대신 [플러그인](/docs/en/plugins)으로 패키징하세요. 플러그인 스킬은 `plugin-name:skill-name` 네임스페이스를 사용하므로 디렉토리별 스킬과 충돌하지 않습니다. 플랫폼 팀이 한곳에서 버전을 관리하고 업데이트할 수 있습니다.

사용되지 않는 스킬을 찾으려면 OpenTelemetry [로그 내보내기](/docs/en/monitoring-usage)를 활성화하고 `OTEL_LOG_TOOL_DETAILS=1`을 설정하여 스킬 이름이 수정되지 않고 있는 그대로 기록되도록 하세요. [`skill_activated` 이벤트](/docs/en/monitoring-usage#skill-activated-event)는 `skill.name` 속성에 모든 호출을 기록하며, `invocation_trigger`는 명령, Claude, 또는 중첩된 스킬이 이를 호출했는지 여부를 기록하므로 통합하거나 은퇴시킬 항목을 파악할 수 있습니다.

## 계층화가 한계에 다다랐을 때 규칙 중앙화

코드베이스가 커짐에 따라 디렉토리별 CLAUDE.md 파일을 관리하기가 어려워질 수 있습니다. 규칙이 표류하고 파일이 오래되며 아무도 루트를 소유하지 않게 됩니다. 이 문제를 해결하는 일은 일반적으로 해당 영역에서 작업하는 각 개발자가 아닌 저장소의 Claude Code 설정을 유지 관리하는 팀의 몫이 됩니다.

항상 로드되는 CLAUDE.md에서 규정 및 참조 콘텐츠를 제거하고 요청 시 로드되는 메인 메커니즘으로 이동하세요:

* [스킬](/docs/en/skills): 작업과 관련이 있을 때만 Claude가 로드하는 참조 자료
* [플러그인](/docs/en/plugins): 플랫폼 팀이 중앙에서 소유하는 스킬, 훅, 명령의 버전 관리되는 번들
* [MCP 서버](/docs/en/mcp): 조직에서 이미 저장소에 대해 코드 검색 또는 RAG 인덱스를 운영 중인 경우, 이를 MCP 도구로 노출하여 Claude가 파일을 직접 읽는 대신 쿼리하도록 설정

플랫폼 팀이 이를 중앙에서 적용하는 방법은 [서버 관리형 또는 엔드포인트 관리형 설정](/docs/en/server-managed-settings#choose-between-server-managed-and-endpoint-managed-settings)을 참조하세요.

### 세션 시작 시 올바른 플러그인 추천

규칙이 플러그인으로 이동하면 생소한 트리의 일부에서 Claude를 시작하는 팀원은 해당 영역 소유자가 유지 관리하는 플러그인에 대한 신호를 받지 못합니다. 훅이 stdout에 출력하는 모든 내용은 첫 프롬프트 전 Claude의 컨텍스트에 추가되므로 [`SessionStart` 훅](/docs/en/hooks#sessionstart)을 사용하여 이 신호 공백을 메울 수 있습니다.

예를 들어 [훅 입력](/docs/en/hooks#common-input-fields)에서 시작 디렉토리를 읽고, 저장소에 커밋된 경로-플러그인 맵에서 이를 조회한 뒤, Claude가 첫 응답에 이를 전달할 수 있도록 추천 텍스트를 출력하는 스크립트를 작성할 수 있습니다. 훅 작성 및 등록은 [훅으로 작업 자동화하기](/docs/en/hooks-guide)를 참조하세요.

## 종합하기

아래의 결합된 구성은 모노레포 레이아웃을 사용합니다. 대규모 단일 트리의 모든 하위 디렉토리에도 동일한 파일이 작동합니다. 프로젝트 설정은 Claude를 시작하는 디렉토리에서만 로드되므로 각 하위 디렉토리의 `.claude/settings.json`은 루트 파일에 계층화되기보다 자체 완결적이어야 합니다.

이 예시는 `.claude/settings.json`에 `worktree`, `additionalDirectories`, `Read` 거부 규칙을 커밋하여 `packages/api/`에서 작업하는 모든 개발자가 동일한 형제 접근 권한, 스파스 경로 및 제외 설정을 받도록 합니다. 아래 파일은 `packages/api/`에 대해 커밋된 영역별 설정입니다:

```json packages/api/.claude/settings.json theme={null}
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ],
    "symlinkDirectories": [
      "node_modules"
    ]
  },
  "permissions": {
    "additionalDirectories": [
      "../shared"
    ],
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)"
    ]
  }
}
```

이 세션은 `packages/api/`에서 시작하므로 형제 패키지의 CLAUDE.md 파일은 이미 범위 밖에 있어 `claudeMdExcludes`가 여기서는 필요하지 않습니다. 루트에서도 세션을 시작하는 경우 저장소 루트의 `.claude/settings.local.json`에 추가하세요.

`additionalDirectories` 항목은 `packages/api/`에서 Claude를 직접 시작할 때 적용됩니다. 이 세션에서 생성된 워크트리 내부의 작업 디렉토리는 워크트리 루트가 되므로 이 설정 파일은 로드되지 않습니다. 형제 패키지는 추가 조치 없이도 워크트리 내부에서 이미 도달 가능하지만, [워크트리 설정 노트](#check-out-only-the-directories-you-need)에서 설명했듯이 워크트리 세션이 읽어올 수 있도록 저장소 루트의 `.claude/settings.json`에 거부 규칙의 사본을 하나 더 두어야 합니다:

```json .claude/settings.json theme={null}
{
  "permissions": {
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)"
    ]
  }
}
```

설정 후 저장소는 다음 레이아웃을 가집니다:

```text theme={null}
monorepo/
  CLAUDE.md
  .claude/settings.json                           # 워크트리 세션용 거부 규칙
  packages/
    api/
      CLAUDE.md
      .claude/settings.json                       # worktree, additionalDirectories, 거부 규칙
      .claude/skills/api-testing/SKILL.md
    web/
      CLAUDE.md
      .claude/skills/component-patterns/SKILL.md
    shared/
      CLAUDE.md
```

이 설정을 통해 `packages/api/`에서 Claude를 시작할 때:

* 루트 CLAUDE.md 및 `packages/api/CLAUDE.md`를 로드하고 `packages/web/CLAUDE.md`는 건너뜁니다
* `packages/api/` 및 `packages/shared/`에서 파일을 읽고 편집할 수 있습니다
* `packages/api/` 내부의 `dist/` 및 `build/` 아래 빌드 출력 읽기를 건너뜁니다
* api-testing 스킬을 요청 시 사용할 수 있습니다
* `.claude/`, `packages/api/`, `packages/shared/` 및 루트 수준 파일을 포함하는 워크트리를 생성하며, 루트 설정 파일의 거부 규칙이 워크트리 전체에 적용됩니다

## 패키지에 걸친 범위 및 플랜 변경 사항

위의 구성은 Claude가 보는 대상을 제어합니다. 단일 변경 사항이 해당 공유 타입을 사용하는 모든 호출 사이트와 함께 공유 타입을 업데이트하는 것처럼 여러 패키지에 손을 대는 경우, 작업의 범위와 순서를 결정하는 방식도 결과에 영향을 줍니다.

패키지 간 변경 사항을 일관되게 유지하는 데 유용한 두 가지 기법:

* **단일 세션에서 Claude에게 전체 변경 부여**: 공유 편집 내용과 호출 사이트를 함께 전달하면 패키지별로 재도출하는 대신 각 편집 뒤의 결정을 일관되게 유지할 수 있습니다
* **편집 전에 플랜을 파일에 저장**: [먼저 플랜을 세우고](/docs/en/best-practices#explore-first-then-plan-then-code) Claude에게 플랜을 저장소의 마크다운 파일에 쓰도록 요청하세요. 긴 패키지 간 세션은 진행에 따라 [컨텍스트가 압축](/docs/en/context-window#what-survives-compaction)되므로, 저장된 플랜은 대화 내역이 남아있지 못하는 상황에서도 살아남습니다

## 다음 단계

이 구성을 갖춘 후 다음 사항을 구체화할 수 있습니다:

* Claude가 파일을 편집한 후 디렉토리별 린터나 타입 체커를 실행하려면 [훅](/docs/en/hooks-guide)을 사용하세요
* 코드베이스 크기가 토큰 사용량에 미치는 영향 및 폭넓은 배포 전에 지출 한도를 설정하는 방법을 이해하려면 [비용을 효과적으로 관리하기](/docs/en/costs)를 검토하세요
* 이 페이지의 저장소별 구성을 넘어서는 조직 배포 패턴 및 소유권 모델에 대해 알아보려면 Claude 블로그에서 [대규모 코드베이스에서 Claude Code가 작동하는 방식](https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start)을 읽어보세요
