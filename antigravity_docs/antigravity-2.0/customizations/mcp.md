---
slug: mcp
section: Antigravity 2.0
title: MCP
path:
  - Antigravity 2.0
  - 사용자 정의
  - MCP
---

# Antigravity 에디터: MCP 통합

Antigravity는 에디터가 사용자의 로컬 도구, 데이터베이스 및 외부 서비스에 안전하게 연결할 수 있도록 지원하는 표준 규격인 MCP(Model Context Protocol)를 지원합니다. 이 통합은 에디터에 열려 있는 파일뿐만 아니라 그 이상의 실시간 맥락(컨텍스트)을 AI에 제공합니다.

## MCP란 무엇인가요?

MCP는 Antigravity와 광범위한 개발 환경 사이의 다리 역할을 합니다. 데이터베이스 스키마나 로그와 같은 맥락을 에디터에 수동으로 복사해서 붙여넣는 대신, MCP를 사용하면 Antigravity가 필요할 때 이 정보를 직접 가져올 수 있습니다.

## 주요 기능

### 1. 맥락 리소스 (Context Resources)

AI는 연결된 MCP 서버로부터 데이터를 읽어와서 제안에 참고할 수 있습니다.

**예시:** SQL 쿼리를 작성할 때, Antigravity는 활성화된 Neon 또는 Supabase 스키마를 검사하여 올바른 테이블 및 열 이름을 제안할 수 있습니다.

**예시:** 디버깅할 때 에디터가 Netlify 또는 Heroku의 최근 빌드 로그를 가져올 수 있습니다.

### 2. 사용자 정의 도구 (Custom Tools)

MCP를 통해 Antigravity는 연결된 서버에서 정의한 안전하고 특정한 작업을 실행할 수 있습니다.

**예시:** "이 TODO에 대한 Linear 이슈를 생성해 줘."

**예시:** "인증 패턴을 찾기 위해 Notion 또는 GitHub을 검색해 줘."

## 연결 방법

연결은 내장된 MCP 스토어(MCP Store)를 통해 직접 관리됩니다.

1. **스토어 액세스:** 에디터 사이드 패널 상단의 "..." 드롭다운 메뉴에서 MCP Store 패널을 엽니다.
2. **탐색 및 설치:** 목록에서 지원되는 서버 중 하나를 선택하고 Install(설치)을 클릭합니다.
3. **인증:** 화면의 안내에 따라 계정을 안전하게 연결합니다 (해당하는 경우).

설치가 완료되면 서버의 리소스와 도구를 에디터에서 자동으로 사용할 수 있습니다.

## 사용자 정의 MCP 서버 연결하기

사용자 정의 MCP 서버에 연결하려면 다음 단계를 따르세요:

1. 에디터 에이전트 패널 상단의 "..." 드롭다운 메뉴를 통해 MCP 스토어를 엽니다.
2. "Manage MCP Servers"(MCP 서버 관리)를 클릭합니다.
3. "View raw config"(원시 구성 보기)를 클릭합니다.
4. 사용자 정의 MCP 서버 구성을 반영하여 mcp_config.json 파일을 수정합니다.

구성 파일은 `~/.gemini/config/mcp_config.json`에 위치해 있습니다.

### 구성 구조

구성 파일에는 연결하려는 각 서버를 정의하는 단일 `mcpServers` 객체가 포함되어 있습니다.

```json
{
  "mcpServers": {
    "serverName": {
      "command": "path/to/executable",
      "args": ["--arg1", "value1"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

### 구성 속성

각 서버 항목은 다음 속성들을 지원합니다:

**전송 방식 (Transport - 둘 중 하나 필수):**

- **`command`** (문자열): stdio 전송을 위한 실행 파일 경로.
- **`serverUrl`** (문자열): 스트리밍 가능한 HTTP 전송을 위한 원격 서버 URL.

**선택 사항 (Optional):**

- **`args`** (문자열 배열): stdio 전송을 위한 명령줄 인수.
- **`env`** (객체): stdio 서버 프로세스용 환경 변수.
- **`cwd`** (문자열): stdio 서버용 작업 디렉터리.
- **`headers`** (객체): 원격 서버용 사용자 정의 HTTP 헤더.
- **`authProviderType`** (문자열): 인증 공급자. ADC용으로 `"google_credentials"`를 지원합니다.
- **`oauth`** (객체): OAuth 클라이언트 자격 증명 (`clientId`, `clientSecret`).
- **`disabled`** (부울): 구성을 제거하지 않고 서버를 일시적으로 비활성화합니다.
- **`disabledTools`** (문자열 배열): 모델에 제공하지 않을 도구 이름 목록.

## 인증

### Google 자격 증명 (Google Credentials)

`authProviderType`을 `"google_credentials"`로 설정하여 Google 애플리케이션 기본 자격 증명(ADC)을 사용합니다.

```json
{
  "mcpServers": {
    "my-gcp-service": {
      "serverUrl": "https://example.googleapis.com/mcp/",
      "authProviderType": "google_credentials"
    }
  }
}
```

이 기능을 사용하려면 애플리케이션 기본 자격 증명이 구성되어 있어야 합니다. 이를 설정하려면 다음을 실행하세요:

```bash
gcloud auth application-default login
```

### OAuth

Antigravity는 동적 클라이언트 등록(DCR)을 지원하는 서버의 OAuth를 자동으로 처리할 수 있습니다. 이러한 서버의 경우 추가 구성이 필요하지 않습니다:

```json
{
  "mcpServers": {
    "oauth-server": {
      "serverUrl": "https://api.example.com/mcp/"
    }
  }
}
```

서버가 동적 클라이언트 등록을 지원하지 않는 경우, 다음과 같이 클라이언트 자격 증명을 수동으로 제공할 수 있습니다:

```json
{
  "mcpServers": {
    "oauth-server": {
      "serverUrl": "https://api.example.com/mcp/",
      "oauth": {
        "clientId": "your-client-id",
        "clientSecret": "your-client-secret"
      }
    }
  }
}
```

클라이언트 자격 증명을 수동으로 제공한 경우, OAuth 공급자에 다음이 리디렉션 URI로 등록되어 있는지 확인하세요:

```
https://antigravity.google/oauth-callback
```

OAuth가 활성화된 서버에 연결할 때:

1. `Cmd+,` (Mac) 또는 `Ctrl+,` (Windows/Linux) 키를 눌러 **에이전트 설정(Agent Settings)**을 엽니다.
2. **Customizations**(사용자 정의) 탭으로 이동하여 서버 옆에 있는 **Authenticate**(인증) 버튼을 클릭합니다.

![Click Authenticate](../../.assets/image/mcp-oauth-authenticate.png)

3. 브라우저에서 인증을 완료하고 승인 코드를 복사합니다.

![Copy authorization code](../../.assets/image/mcp-oauth-copy-code.png)

4. 코드를 설정 패널에 다시 붙여넣고 **Submit**(제출)을 클릭합니다.

![Paste auth code](../../.assets/image/mcp-oauth-paste-code.png)

인증되면 서버가 자동으로 다시 연결됩니다.

![Authenticated server](../../.assets/image/mcp-oauth-authenticated.png)

액세스 토큰은 `~/.gemini/antigravity/mcp_oauth_tokens.json`에 저장됩니다. 만료된 토큰은 자동으로 갱신되며, 유효하지 않은 토큰은 제거됩니다.

### 사용자 정의 헤더 (Custom Headers)

사용자 정의 HTTP 헤더(예: API 키 또는 베어러 토큰)가 필요한 서버의 경우, 이를 `headers` 객체에 추가합니다. 예시:

```json
{
  "mcpServers": {
    "my-remote-server": {
      "serverUrl": "https://api.example.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN"
      }
    }
  }
}
```

## 지원되는 서버

MCP 스토어는 현재 다음 항목들에 대한 통합을 제공합니다:

- Airweave
- Arize
- AlloyDB for PostgreSQL
- Atlassian
- BigQuery
- Bigtable Admin remote MCP
- Chrome DevTools
- ClickHouse
- Cloud SQL for MySQL
- Cloud SQL for PostgreSQL
- Cloud SQL for SQL Server
- Cloud SQL Managed MCP
- Dart
- Dataplex
- Figma Dev Mode MCP
- Firebase
- GitHub
- GitLab Orbit
- Harness
- Heroku
- Linear
- Locofy
- Looker
- MCP Toolbox for Databases
- MongoDB
- Neon
- Netlify
- Notion
- PayPal
- Perplexity Ask
- Pinecone
- PostHog
- Postman
- Prisma
- Redis
- Sequential Thinking
- SonarQube
- Spanner
- Stripe
- Supabase
