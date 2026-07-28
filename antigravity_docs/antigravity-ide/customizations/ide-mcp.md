---
slug: ide-mcp
section: Antigravity IDE
title: MCP
path:
  - Antigravity IDE
  - 사용자 정의
  - MCP
---

# Antigravity 에디터: MCP 연동

Antigravity는 에디터를 로컬 도구, 데이터베이스 및 외부 서비스에 안전하게 연결할 수 있도록 지원하는 표준 프로토콜인 Model Context Protocol(MCP)을 지원합니다. 이 연동을 통해 AI는 에디터에 열려 있는 파일뿐만 아니라 그 이상의 실시간 컨텍스트를 제공받을 수 있습니다.

## MCP란 무엇인가요?

MCP는 Antigravity와 광범위한 개발 환경 사이의 다리 역할을 합니다. 데이터베이스 스키마나 로그와 같은 컨텍스트를 에디터에 수동으로 복사해서 붙여넣는 대신, MCP를 사용하면 Antigravity가 필요할 때 이 정보를 직접 가져올 수 있습니다.

## 핵심 기능

### 1. 컨텍스트 리소스

AI는 연결된 MCP 서버의 데이터를 읽어 제안에 활용할 수 있습니다.

**예시:** SQL 쿼리를 작성할 때, Antigravity는 활성화된 Neon 또는 Supabase 스키마를 검사하여 올바른 테이블 및 열 이름을 제안할 수 있습니다.

**예시:** 디버깅할 때 에디터는 Netlify 또는 Heroku의 최근 빌드 로그를 가져올 수 있습니다.

### 2. 사용자 정의 도구

MCP는 Antigravity가 연결된 서버에 정의된 특정하고 안전한 작업을 실행할 수 있도록 지원합니다.

**예시:** "이 TODO에 대한 Linear 이슈를 생성해 줘."

**예시:** "인증 패턴에 대해 Notion이나 GitHub에서 검색해 줘."

## 연결 방법

연결은 내장된 MCP 스토어를 통해 직접 관리됩니다.

1. **스토어 액세스:** 에디터 사이드 패널 상단의 "..." 드롭다운 내에서 MCP 스토어 패널을 엽니다.
2. **탐색 및 설치:** 목록에서 지원되는 서버 중 하나를 선택하고 Install을 클릭합니다.
3. **인증:** 화면의 안내에 따라 계정을 안전하게 연동합니다(해당되는 경우).

설치되면 서버의 리소스와 도구를 에디터에서 자동으로 사용할 수 있게 됩니다.

## 사용자 정의 MCP 서버 연결

사용자 정의 MCP 서버에 연결하려면 다음 단계를 따르세요:

1. 에디터 에이전트 패널 상단의 "..." 드롭다운을 통해 MCP 스토어를 엽니다.
2. "Manage MCP Servers"를 클릭합니다.
3. "View raw config"를 클릭합니다.
4. 사용자 정의 MCP 서버 구성으로 `mcp_config.json`을 수정합니다.

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

각 서버 항목은 다음 속성을 지원합니다:

**전송 방식 (Transport - 하나 필수):**

- **`command`** (string): stdio 전송을 위한 실행 파일의 경로입니다.
- **`serverUrl`** (string): Streamable HTTP 전송을 위한 원격 서버의 URL입니다.

**선택 사항:**

- **`args`** (string[]): stdio 전송을 위한 명령줄 인수입니다.
- **`env`** (object): stdio 서버 프로세스를 위한 환경 변수입니다.
- **`cwd`** (string): stdio 서버의 작업 디렉터리입니다.
- **`headers`** (object): 원격 서버를 위한 사용자 정의 HTTP 헤더입니다.
- **`authProviderType`** (string): 인증 공급자입니다. ADC의 경우 `"google_credentials"`를 지원합니다.
- **`oauth`** (object): OAuth 클라이언트 자격 증명(`clientId`, `clientSecret`)입니다.
- **`disabled`** (boolean): 구성을 제거하지 않고 서버를 일시적으로 비활성화합니다.
- **`disabledTools`** (string[]): 모델에 제공하지 않을 도구 이름 목록입니다.

## 인증

### Google 자격 증명(Google Credentials)

Google 애플리케이션 기본 자격 증명(ADC)을 사용하려면 `authProviderType`을 `"google_credentials"`로 설정하세요.

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

이를 위해서는 애플리케이션 기본 자격 증명이 구성되어 있어야 합니다. 설정하려면 다음을 실행하세요:

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

서버가 동적 클라이언트 등록을 지원하지 않는 경우, 클라이언트 자격 증명을 수동으로 제공할 수 있습니다:

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

1. `Cmd+,` (Mac) 또는 `Ctrl+,` (Windows/Linux)를 눌러 **에이전트 설정(Agent Settings)**을 엽니다.
2. **Customizations** 탭으로 이동하여 서버 옆의 **Authenticate** 버튼을 클릭합니다.

![Click Authenticate](assets/image/docs/tools/mcp-oauth-authenticate.png)

3. 브라우저에서 인증을 완료하고 승인 코드를 복사합니다.

![Copy authorization code](assets/image/docs/tools/mcp-oauth-copy-code.png)

4. 코드를 설정 패널에 다시 붙여넣고 **Submit**을 클릭합니다.

![Paste auth code](assets/image/docs/tools/mcp-oauth-paste-code.png)

인증이 완료되면 서버가 자동으로 다시 연결됩니다.

![Authenticated server](assets/image/docs/tools/mcp-oauth-authenticated.png)

액세스 토큰은 `~/.gemini/antigravity/mcp_oauth_tokens.json`에 저장됩니다. 만료된 토큰은 자동으로 갱신되며, 유효하지 않은 토큰은 제거됩니다.

### 사용자 정의 헤더(Custom Headers)

사용자 정의 HTTP 헤더(예: API 키 또는 베어러 토큰)가 필요한 서버의 경우, `headers` 객체에 추가합니다. 예시:

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

현재 MCP 스토어는 다음에 대한 연동을 제공합니다:

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
