> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# `.claude` 디렉토리 탐색하기

> Claude Code가 CLAUDE.md, settings.json, 후크(hooks), 스킬(skills), 명령어(commands), 서브에이전트(subagents), 워크플로(workflows), 규칙(rules), 자동 메모리(auto memory)를 읽는 위치를 살펴봅니다.

## 프로젝트 수준 구성 (`your-project/`)

* **`CLAUDE.md`**: 프로젝트 지침서. 매 세션마다 Claude가 컨텍스트로 읽습니다.
* **`.mcp.json`**: 프로젝트 범위의 MCP 서버 구성 파일입니다. 팀 전체가 공유합니다.
* **`.worktreeinclude`**: 새로운 git 워크트리에 복사할 gitignored 파일 목록입니다.
* **`.claude/`**: 프로젝트 관련 구성, 규칙, 확장 파일들이 위치하는 디렉토리입니다.
  * `settings.json`: 권한 설정, 후크, 실행 모델 등 프로젝트 기본 설정
  * `settings.local.json`: 개인용 설정 덮어쓰기 파일 (gitignored)
  * `rules/`: 특정 디렉토리나 파일 타입에만 적용되는 주제별 규칙 파일
  * `skills/`: 사용자가 직접 또는 Claude가 호출할 수 있는 재사용 가능한 스킬 폴더
  * `commands/`: 단일 파일 형태의 슬래시 명령어
  * `agents/`: 별도의 컨텍스트 창에서 작동하는 전용 서브에이전트
  * `workflows/`: 다수의 서브에이전트를 조율하는 동적 워크플로 스크립트

## 전역 수준 구성 (`~/`)

* **`~/.claude.json`**: 앱 상태 및 UI 환경설정, 개인용 MCP 서버 관리
* **`~/.claude/`**: 모든 프로젝트에 전역으로 적용되는 구성
  * `CLAUDE.md`: 전역 프로젝트 개인 취향 및 공통 컨벤션
  * `settings.json`: 전역 설정 및 기본 권한
  * `keybindings.json`: 사용자 정의 키보드 단축키
  * `themes/`: 커스텀 컬러 테마
  * `projects/`: 프로젝트별 Claude 자동 메모리 데이터
