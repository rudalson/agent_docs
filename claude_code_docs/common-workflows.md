> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 공통 워크플로우 (Common Workflows)

> Claude Code를 사용하여 코드베이스 탐색, 버그 수정, 리팩토링, 테스트 및 기타 일상적인 작업을 수행하기 위한 단계별 가이드입니다.

이 페이지는 일상적인 개발을 위한 짧은 레시피를 모아놓았습니다. 프롬프팅 및 컨텍스트 관리에 대한 상위 수준의 지침은 [모범 사례(Best practices)](/docs/en/best-practices)를 참조하세요.

이 페이지에서 다루는 내용:

* 코드 탐색, 버그 수정, 리팩토링, 테스트, PR 및 문서 작성을 위한 [프롬프트 레시피](#prompt-recipes)
* 작업이 여러 번의 세션에 걸쳐 이어질 수 있도록 [이전 대화 재개하기](#resume-previous-conversations)
* 동시 편집이 충돌하지 않도록 [작업 트리(worktree)로 병렬 세션 실행하기](#run-parallel-sessions-with-worktrees)
* 변경 사항이 디스크에 반영되기 전에 검토할 수 있도록 [편집 전 계획 수립하기](#plan-before-editing)
* 메인 컨텍스트를 깨끗하게 유지하기 위해 [하위 에이전트에 조사 위임하기](#delegate-research-to-subagents)
* CI 및 배치 처리를 위해 [Claude를 스크립트로 파이프 전송하기](#pipe-claude-into-scripts)

## 프롬프트 레시피

생소한 코드 탐색, 디버깅, 리팩토링, 테스트 작성, PR 생성과 같은 일상적인 작업을 위한 프롬프트 패턴입니다. 각 레시피는 모든 Claude Code 환경에서 작동합니다. 표현을 프로젝트에 맞게 변경하세요.

### 새로운 코드베이스 이해하기

모노리포 또는 대규모 코드베이스에서 Claude Code를 구성하는 방법은 [모노리포 및 대규모 리포지토리](/docs/en/large-codebases)를 참조하세요.

#### 빠른 코드베이스 개요 파악하기

새 프로젝트에 막 참여하여 구조를 빠르게 이해해야 하는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="프로젝트 루트 디렉토리로 이동">
    ```bash theme={null}
    cd /path/to/project 
    ```

    `/path/to/project`를 프로젝트 경로로 교체하세요.
  </Step>

  <Step title="Claude Code 시작">
    ```bash theme={null}
    claude 
    ```
  </Step>

  <Step title="상위 수준 개요 요청">
    ```text theme={null}
    give me an overview of this codebase
    ```
  </Step>

  <Step title="특정 컴포넌트 더 깊이 파악하기">
    ```text theme={null}
    explain the main architecture patterns used here
    ```

    ```text theme={null}
    what are the key data models?
    ```

    ```text theme={null}
    how is authentication handled?
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 광범위한 질문으로 시작한 다음 특정 영역으로 범위를 좁히세요.
  * 프로젝트에서 사용되는 코딩 규칙 및 패턴에 대해 물어보세요.
  * 프로젝트 특화 용어집을 요청하세요.
</Tip>

#### 관련 코드 찾기

특정 기능과 관련된 코드를 찾아야 하는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="Claude에게 관련 파일 찾기 요청">
    ```text theme={null}
    find the files that handle user authentication
    ```
  </Step>

  <Step title="컴포넌트 상호작용 방식에 대한 컨텍스트 파악">
    ```text theme={null}
    how do these authentication files work together?
    ```
  </Step>

  <Step title="실행 흐름 이해">
    ```text theme={null}
    trace the login process from front-end to database
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 찾고자 하는 내용에 대해 구체적으로 설명하세요.
  * 프로젝트의 도메인 용어를 사용하세요.
  * Claude에게 정확한 "정의로 이동(go to definition)" 및 "참조 찾기(find references)" 탐색 기능을 제공하려면 언어에 맞는 [코드 인텔리전스 플러그인](/docs/en/discover-plugins#code-intelligence)을 설치하세요.
</Tip>

***

### 버그를 효율적으로 수정하기

오류 메시지가 발생하여 소스를 찾아 수정해야 하는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="Claude에게 오류 공유">
    ```text theme={null}
    I'm seeing an error when I run npm test
    ```
  </Step>

  <Step title="수정 방안 권장 요청">
    ```text theme={null}
    suggest a few ways to fix the @ts-ignore in user.ts
    ```
  </Step>

  <Step title="수정 사항 적용">
    ```text theme={null}
    update user.ts to add the null check you suggested
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 문제를 재현하고 스택 트레이스를 얻을 수 있는 명령을 Claude에게 알려주세요.
  * 오류를 재현하기 위한 단계를 언급하세요.
  * 오류가 간헐적인지 지속적인지 Claude에게 알려주세요.
</Tip>

***

### 코드 리팩토링하기

오래된 코드를 최신 패턴과 관행으로 업데이트해야 하는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="리팩토링할 레거시 코드 식별">
    ```text theme={null}
    find deprecated API usage in our codebase
    ```
  </Step>

  <Step title="리팩토링 권장 사항 가져오기">
    ```text theme={null}
    suggest how to refactor utils.js to use modern JavaScript features
    ```
  </Step>

  <Step title="안전하게 변경 사항 적용">
    ```text theme={null}
    refactor utils.js to use ES2024 features while maintaining the same behavior
    ```
  </Step>

  <Step title="리팩토링 검증">
    ```text theme={null}
    run tests for the refactored code
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * Claude에게 최신 접근 방식의 이점을 설명하도록 요청하세요.
  * 필요한 경우 변경 사항이 하위 호환성을 유지하도록 요청하세요.
  * 리팩토링은 테스트 가능한 단위로 작게 진행하세요.
</Tip>

***

### 테스트 작업하기

커버되지 않은 코드에 테스트를 추가해야 하는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="테스트되지 않은 코드 식별">
    ```text theme={null}
    find functions in NotificationsService.swift that are not covered by tests
    ```
  </Step>

  <Step title="테스트 스캐폴딩 생성">
    ```text theme={null}
    add tests for the notification service
    ```
  </Step>

  <Step title="의미 있는 테스트 케이스 추가">
    ```text theme={null}
    add test cases for edge conditions in the notification service
    ```
  </Step>

  <Step title="테스트 실행 및 검증">
    ```text theme={null}
    run the new tests and fix any failures
    ```
  </Step>
</Steps>

Claude는 프로젝트의 기존 패턴 및 규칙을 따르는 테스트를 생성할 수 있습니다. 테스트를 요청할 때 검증하려는 동작에 대해 구체적으로 설명하세요. Claude는 기존 테스트 파일을 검사하여 이미 사용 중인 스타일, 프레임워크 및 어설션 패턴을 일치시킵니다.

포괄적인 커버리지를 위해 놓쳤을 수 있는 엣지 케이스를 식별하도록 Claude에게 요청하세요. Claude는 코드 경로를 분석하고 오류 조건, 경계값, 간과하기 쉬운 예상치 못한 입력에 대한 테스트를 제안할 수 있습니다.

***

### 풀 리퀘스트 생성하기

Claude에게 직접 요청("create a pr for my changes")하여 풀 리퀘스트를 생성하거나 단계를 안내할 수 있습니다.

<Steps>
  <Step title="변경 사항 요약">
    ```text theme={null}
    summarize the changes I've made to the authentication module
    ```
  </Step>

  <Step title="풀 리퀘스트 생성">
    ```text theme={null}
    create a pr
    ```
  </Step>

  <Step title="검토 및 다듬기">
    ```text theme={null}
    enhance the PR description with more context about the security improvements
    ```
  </Step>
</Steps>

`gh pr create`를 사용하여 PR을 생성하면 세션이 해당 PR에 자동으로 연결됩니다. 나중에 찾으려면 자신의 PR 번호와 함께 `claude --from-pr 1234`를 실행하여 해당 PR에 연결된 세션으로 필터링된 선택기를 열거나, [`/resume` 선택기](/docs/en/sessions#use-the-session-picker) 검색에 PR URL을 붙여넣으세요.

<Tip>
  제출하기 전에 Claude가 생성한 PR을 검토하고 잠재적인 위험이나 고려 사항을 강조하도록 Claude에게 요청하세요.
</Tip>

### 문서 처리하기

코드에 대한 문서를 추가하거나 업데이트해야 하는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="문서화되지 않은 코드 식별">
    ```text theme={null}
    find functions without proper JSDoc comments in the auth module
    ```
  </Step>

  <Step title="문서 생성">
    ```text theme={null}
    add JSDoc comments to the undocumented functions in auth.js
    ```
  </Step>

  <Step title="검토 및 보완">
    ```text theme={null}
    improve the generated documentation with more context and examples
    ```
  </Step>

  <Step title="문서 검증">
    ```text theme={null}
    check if the documentation follows our project standards
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 원하는 문서 스타일(JSDoc, docstrings 등)을 지정하세요.
  * 문서에 예시를 포함하도록 요청하세요.
  * 공개 API, 인터페이스 및 복잡한 로직에 대한 문서를 요청하세요.
</Tip>

***

### 노트 및 비코드 폴더에서 작업하기

Claude Code는 모든 디렉토리에서 작동합니다. 노트 볼트, 문서 폴더 또는 마크다운 파일 모음 내부에서 실행하여 코드를 수정하는 것과 동일한 방식으로 콘텐츠를 검색, 편집 및 재구성하세요.

`.claude/` 디렉토리와 `CLAUDE.md`는 충돌 없이 다른 도구의 구성 디렉토리와 함께 존재합니다. Claude는 각 도구 호출 시 파일을 새로 읽으므로 다음 번에 파일을 읽을 때 다른 애플리케이션에서 수행한 편집 내용을 그대로 확인합니다.

***

### 이미지 작업하기

코드베이스의 이미지로 작업해야 하고 Claude의 도움을 받아 이미지 콘텐츠를 분석하려는 경우를 예로 들어보겠습니다.

<Steps>
  <Step title="대화에 이미지 추가">
    다음 방법 중 하나를 사용할 수 있습니다.

    1. Claude Code 창으로 이미지를 드래그 앤 드롭
    2. 이미지를 복사하여 Ctrl+V로 CLI에 붙여넣기 (macOS의 경우 iTerm2에서 Cmd+V도 가능)
    3. Claude에게 이미지 경로 제공 (예: "Analyze this image: /path/to/your/image.png")
  </Step>

  <Step title="Claude에게 이미지 분석 요청">
    ```text theme={null}
    What does this image show?
    ```

    ```text theme={null}
    Describe the UI elements in this screenshot
    ```

    ```text theme={null}
    Are there any problematic elements in this diagram?
    ```
  </Step>

  <Step title="컨텍스트를 위해 이미지 사용">
    ```text theme={null}
    Here's a screenshot of the error. What's causing it?
    ```

    ```text theme={null}
    This is our current database schema. How should we modify it for the new feature?
    ```
  </Step>

  <Step title="시각적 콘텐츠에서 코드 제안 가져오기">
    ```text theme={null}
    Generate CSS to match this design mockup
    ```

    ```text theme={null}
    What HTML structure would recreate this component?
    ```
  </Step>
</Steps>

<Tip>
  팁:

  * 텍스트 설명이 명확하지 않거나 번거로울 때 이미지를 사용하세요.
  * 더 나은 컨텍스트를 위해 오류 스크린샷, UI 디자인 또는 다이어그램을 포함하세요.
  * 대화에서 여러 이미지로 작업할 수 있습니다.
  * 이미지 분석은 다이어그램, 스크린샷, 목업 등에서 동작합니다.
  * Claude가 이미지를 참조할 때(예: `[Image #1]`) `Cmd+Click`(Mac) 또는 `Ctrl+Click`(Windows/Linux)하여 링크를 누르면 기본 뷰어에서 이미지가 열립니다.
</Tip>

***

### 파일 및 디렉토리 참조하기

Claude가 읽을 때까지 기다리지 않고 파일이나 디렉토리를 빠르게 포함하려면 @를 사용하세요.

<Steps>
  <Step title="단일 파일 참조">
    ```text theme={null}
    Explain the logic in @src/utils/auth.js
    ```

    대화에 파일의 전체 내용이 포함됩니다.
  </Step>

  <Step title="디렉토리 참조">
    ```text theme={null}
    What's the structure of @src/components?
    ```

    파일 정보가 포함된 디렉토리 목록이 제공됩니다.
  </Step>

  <Step title="MCP 리소스 참조">
    ```text theme={null}
    Show me the data from @github:repos/owner/repo/issues
    ```

    @server:resource 형식을 사용하여 연결된 MCP 서버에서 데이터를 가져옵니다. 세부 정보는 [MCP 리소스](/docs/en/mcp#use-mcp-resources)를 참조하세요.
  </Step>
</Steps>

<Tip>
  팁:

  * 파일 경로는 상대 경로 또는 절대 경로가 될 수 있습니다.
  * `@`를 입력하여 경로 제안 메뉴를 연 다음 Enter 또는 Tab을 눌러 강조 표시된 경로를 수락하고 Enter를 다시 눌러 메시지를 보내세요.
  * @ 파일 참조는 해당 파일의 디렉토리 및 상위 디렉토리에 있는 `CLAUDE.md`를 컨텍스트에 추가합니다.
  * 디렉토리 참조는 파일 내용이 아닌 파일 목록을 보여줍니다.
  * 단일 메시지에서 여러 파일을 참조할 수 있습니다 (예: "@file1.js and @file2.js").
</Tip>

***

### 일정에 따라 Claude 실행하기

매일 아침 열린 PR을 검토하거나 매주 종속성을 감사하거나 밤새 CI 실패를 확인하는 것처럼 주기적으로 작업을 자동으로 처리하도록 Claude를 설정하려는 경우를 예로 들어보겠습니다.

작업을 실행할 위치에 따라 일정 옵션을 선택하세요.

| 옵션 | 실행 위치 | 용도 |
| :--- | :--- | :--- |
| [루틴 (Routines)](/docs/en/routines) | Anthropic 관리형 인프라 | 컴퓨터가 꺼져 있을 때도 실행되어야 하는 작업. 일정 외에도 API 호출이나 GitHub 이벤트로 트리거 가능. [claude.ai/code/routines](https://claude.ai/code/routines)에서 구성. |
| [데스크톱 예약 작업](/docs/en/desktop-scheduled-tasks) | 로컬 머신 (데스크톱 앱) | 로컬 파일, 도구 또는 미커밋 변경 사항에 대한 직접 접근이 필요한 작업. |
| [GitHub Actions](/docs/en/github-actions) | CI 파이프라인 | 열린 PR과 같은 리포지토리 이벤트나 워크플로우 구성과 함께 위치해야 하는 크론 일정 작업. |
| [`/loop`](/docs/en/scheduled-tasks) | 현재 CLI 세션 | 세션이 열려 있는 동안의 빠른 폴링. 새 대화를 시작하면 작업이 중지되며 `--resume` 및 `--continue`로 만료되지 않은 작업을 복원합니다. |

<Tip>
  예약된 작업에 대한 프롬프트를 작성할 때 성공적인 결과가 어떤 모습인지, 결과로 무엇을 해야 하는지 명확히 설명하세요. 작업이 자율적으로 실행되므로 추가 질문을 할 수 없습니다. 예: "`needs-review` 라벨이 지정된 열린 PR을 검토하고, 문제에 인라인 주석을 남기며, `#eng-reviews` Slack 채널에 요약을 게시하세요."
</Tip>

***

### Claude 기능에 대해 물어보기

Claude는 자체 문서에 대한 접근 권한이 내장되어 있어 자신의 기능 및 제한 사항에 대한 질문에 답변할 수 있습니다.

#### 예시 질문

```text theme={null}
can Claude Code create pull requests?
```

```text theme={null}
how does Claude Code handle permissions?
```

```text theme={null}
what skills are available?
```

```text theme={null}
how do I use MCP with Claude Code?
```

```text theme={null}
how do I configure Claude Code for Amazon Bedrock?
```

```text theme={null}
what are the limitations of Claude Code?
```

<Note>
  Claude는 이러한 질문에 대해 문서 기반의 답변을 제공합니다. 실제 시연을 원하시면 대화형 레슨이 포함된 `/powerup`을 실행하거나 상단의 특정 워크플로우 섹션을 참조하세요.
</Note>

<Tip>
  팁:

  * Claude는 사용 중인 버전에 관계없이 항상 최신 Claude Code 문서에 접근할 수 있습니다.
  * 구체적인 질문을 하면 상세한 답변을 얻을 수 있습니다.
  * Claude는 MCP 연동, 엔터프라이즈 구성 및 고급 워크플로우와 같은 복잡한 기능을 설명할 수 있습니다.
</Tip>

***

## 이전 대화 재개하기

작업이 여러 번의 세션에 걸쳐 이어질 때 컨텍스트를 다시 설명하는 대신 중단했던 부분부터 계속하세요. Claude Code는 모든 대화를 로컬에 저장합니다.

```bash theme={null}
claude --continue
```

이는 현재 디렉토리에서 가장 최근 세션을 재개합니다. 아직 세션이 없으면 `No conversation found to continue`를 출력하고 종료합니다. 목록에서 선택하려면 `claude --resume`을 사용하고 실행 중인 세션 내부에서는 `/resume`을 사용하세요. 이름 지정, 브랜치 생성 및 전체 선택기 레퍼런스는 [세션 관리](/docs/en/sessions)를 참조하세요.

## 작업 트리(worktree)로 병렬 세션 실행하기

편집 사항이 충돌하지 않고 Claude가 다른 터미널에서 버그를 수정하는 동안 한 터미널에서 기능 작업을 진행하세요. 각 [git worktree](https://git-scm.com/docs/git-worktree)는 기존 커밋에서 생성된 자체 브랜치의 별도 체크아웃이므로 리포지토리에 먼저 하나 이상의 커밋이 필요합니다.

```bash theme={null}
claude --worktree feature-auth
```

격리된 병렬 세션을 시작하려면 두 번째 터미널에서 다른 이름으로 동일한 명령을 실행하세요. 커밋이 없는 리포지토리에서는 `Failed to resolve base branch "HEAD": git rev-parse failed` 오류와 함께 명령이 실패합니다. 정리, `.worktreeinclude` 및 non-git VCS 지원은 [Worktrees](/docs/en/worktrees)를 참조하세요. 별도의 터미널 대신 한 화면에서 병렬 세션을 모니터링하려면 [백그라운드 에이전트](/docs/en/agent-view)를 참조하세요.

## 편집 전 계획 수립하기

디스크에 반영되기 전에 검토하려는 변경 사항의 경우 플랜 모드로 전환하세요. Claude는 파일을 읽고 계획을 제안하지만 사용자가 승인할 때까지 편집하지 않습니다. 플랜 모드가 활성화되어 있는 동안 상태 표시줄에 `⏸ plan mode on`이 표시됩니다.

```bash theme={null}
claude --permission-mode plan
```

세션 중간에 `Shift+Tab`을 눌러 플랜 모드로 순환 전환할 수도 있습니다. 순환 순서는 `default` → `acceptEdits` → `plan`입니다. 승인 흐름 및 텍스트 편집기에서 계획을 편집하는 방법은 [플랜 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)를 참조하세요.

## 하위 에이전트에 조사 위임하기

대규모 코드베이스를 탐색하면 파일 읽기로 컨텍스트가 가득 찹니다. 결과만 돌아오도록 조사를 위임하세요.

```text theme={null}
use a subagent to investigate how our auth system handles token refresh
```

하위 에이전트는 자체 컨텍스트 창에서 파일을 읽고 요약을 보고합니다. 자체 도구와 프롬프트를 사용하는 커스텀 에이전트 정의는 [하위 에이전트](/docs/en/sub-agents)를 참조하세요.

## Claude를 스크립트로 파이프 전송하기

CI, pre-commit 훅 또는 배치 처리를 위해 비대화형으로 Claude를 실행하세요. Stdin 및 stdout은 Unix 도구처럼 작동합니다.

```bash theme={null}
git log --oneline -20 | claude -p "summarize these recent commits"
```

출력 형식, 권한 플래그 및 분산 처리 패턴은 [비대화형 모드](/docs/en/headless)를 참조하세요.

## 다음 단계

<CardGroup cols={2}>
  <Card title="모범 사례" icon="lightbulb" href="/docs/en/best-practices">
    Claude Code를 최대한 활용하기 위한 패턴
  </Card>

  <Card title="세션 관리" icon="rotate-left" href="/docs/en/sessions">
    대화 재개, 이름 지정 및 브랜치 생성
  </Card>

  <Card title="작업 트리 (Worktrees)" icon="code-branch" href="/docs/en/worktrees">
    격리된 병렬 세션 실행
  </Card>

  <Card title="Claude Code 확장" icon="puzzle-piece" href="/docs/en/features-overview">
    스킬, 훅, MCP, 하위 에이전트 및 플러그인 추가
  </Card>
</CardGroup>
