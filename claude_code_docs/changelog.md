> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude Code 변경 이력 (Changelog)

> 버전별 신규 기능, 개선 사항 및 버그 수정 내역을 포함한 Claude Code 릴리스 노트.

이 페이지는 [GitHub CHANGELOG.md](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)에서 생성되었습니다.

설치된 버전을 확인하려면 `claude --version`을 실행하세요.

---

### v2.1.218 (2026년 7월 22일)
* `/code-review`가 백그라운드 서브에이전트로 실행되도록 변경되어 리뷰 작업이 더 이상 대화를 채우지 않도록 개선
* 터미널 및 인코딩 관련 버그 수정
* Windows 경로 처리 관련 버그 수정
* `/ultrareview` 및 `/code-review ultra` 기능 개선

---

### v2.1.217 (2026년 7월 21일)
* 프롬프트 입력창에 이모지 쇼트코드 자동완성 기능 추가 (`:heart:` 등)
* 트랜스크립트 작성 실패 시 경고 추가
* 오디오 및 스크린 리더 기능 관련 버그 수정
* 서브에이전트 동시 실행 제한 추가 (기본 20개)

---

### v2.1.216 (2026년 7월 20일)
* `sandbox.filesystem.disabled` 설정 추가: 네트워크 이그레스 제어는 유지하면서 파일시스템 격리 스킵 가능
* 긴 세션에서의 성능 정체 현상 해결
* 자동 모드 및 MCP 관련 버그 수정
