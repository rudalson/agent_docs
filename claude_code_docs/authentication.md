> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 인증 (Authentication)

> Claude Code에 로그인하고 개인, 팀 및 조직을 위한 인증을 구성합니다.

Claude Code는 설정에 따라 다양한 인증 방식을 지원합니다. 개인 사용자는 Claude.ai 계정으로 로그인할 수 있으며, 팀은 Claude for Teams 또는 Enterprise, Claude Console, 또는 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry와 같은 클라우드 제공업체를 사용할 수 있습니다.

## Claude Code에 로그인하기

[Claude Code 설치](/docs/en/setup#install-claude-code) 후 터미널에서 `claude`를 실행합니다. 처음 실행 시 Claude Code는 로그인을 위해 브라우저 창을 엽니다.

로그인이 완료되면 터미널에 `Login successful`이 표시되며 `Enter`를 눌러 계속 진행할 수 있습니다.

다음 계정 유형 중 하나로 인증할 수 있습니다:

* **Claude Pro 또는 Max 구독**: Claude.ai 계정으로 로그인
* **Claude for Teams 또는 Enterprise**: 팀 관리자가 초대한 Claude.ai 계정으로 로그인
* **Claude Console**: Console 자격 증명으로 로그인
* **클라우드 제공업체**: Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry를 사용하는 경우 필수 환경 변수를 설정하고 실행

로그아웃하고 다시 인증하려면 Claude Code 프롬프트에 `/logout`을 입력하세요.

## 자격 증명 관리 및 우선순위

여러 자격 증명이 있는 경우 Claude Code는 다음 순서로 하나를 선택합니다:

1. 클라우드 제공업체 자격 증명 (`CLAUDE_CODE_USE_BEDROCK` 등 설정 시)
2. `ANTHROPIC_AUTH_TOKEN` 환경 변수
3. `ANTHROPIC_API_KEY` 환경 변수
4. `apiKeyHelper` 스크립트 출력
5. `CLAUDE_CODE_OAUTH_TOKEN` 환경 변수
6. `/login`을 통한 구독 OAuth 자격 증명
