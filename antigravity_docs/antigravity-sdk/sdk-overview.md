---
slug: sdk-overview
section: Antigravity SDK
title: 개요 및 빠른 시작
path:
  - Antigravity SDK
  - 개요 및 빠른 시작
---

# Google Antigravity SDK

Antigravity SDK는 자율 AI 에이전트를 빌드, 테스트 및 실행하도록 설계된 프로그래밍 방식의 Python 프레임워크입니다. Antigravity CLI 및 Antigravity 2.0을 구동하는 것과 동일한 핵심 에이전트 하네스(harness)를 확장하여 고급 에이전트 기능을 자체 애플리케이션 및 워크플로에 직접 통합할 수 있도록 지원합니다.

SDK는 에이전트의 논리를 에이전트가 실행되는 위치와 분리하므로 개발자는 에이전트가 수행하는 작업에만 집중할 수 있고, SDK는 실행 방법과 위치를 처리합니다.

## 빠른 시작

pip를 사용하여 SDK를 설치합니다.

```bash
pip install google-antigravity
```

### Hello World 예제

15줄 미만의 Python 코드로 로컬 환경과 상호 작용할 수 있는 기능적 에이전트:

```python
import asyncio
from google.antigravity import Agent, LocalAgentConfig

async def main():
    config = LocalAgentConfig()
    async with Agent(config) as agent:
        response = await agent.chat("현재 디렉터리에 어떤 파일이 있나요?")
        print(await response.text())

if __name__ == "__main__":
    asyncio.run(main())
```

## 핵심 요소

**1. 통제된 확장성 (도구)**  
모든 에이전트는 기본 도구 모음(파일 I/O, 코드 편집, 쉘 실행, 디렉터리 검색)으로 시작하며, 통일된 실행 파이프라인 아래 네 가지 유형의 도구를 사용하여 확장할 수 있습니다.

*   **기본 제공 도구 (Built-in Tools):** 핵심 파일 및 시스템 조작 기능.  
*   **사용자 정의 Python 함수:** 모든 Python 호출 가능 객체(callable)를 에이전트 도구로 등록.  
*   **MCP 서버:** 모든 Model Context Protocol (MCP) 서버(stdio, SSE 또는 HTTP) 연결.  
*   **에이전트 스킬 (Agent Skills):** 재사용 가능한 지침 및 도구 패키지 로드.

**2. 선언적 안전 정책**  
선언적인 "기본 거부(deny by default)" 정책 시스템을 사용하여 에이전트 권한을 구성하고 도구 실행 시기 및 방법을 제어합니다.

```python
from google.antigravity.hooks.policy import deny, allow, ask_user

policies = [
    deny("*"),                                         # 기본적으로 모든 도구 차단
    allow("view_file"),                                # 알림 없이 파일 읽기 허용
    ask_user("run_command", handler=my_handler),       # 쉘 실행을 위해 사람의 승인 요구
]
```

**3. 라이프사이클 훅**  
9개의 구체적인 라이프사이클 시점(예: 세션 시작, 턴 전/후, 도구 호출 전/후)에 걸쳐 세 가지 범주의 훅을 사용하여 에이전트 실행을 세부적으로 제어할 수 있습니다.

*   **Inspect** (읽기 전용, 비차단): 로깅, 감사 추적 및 메트릭 수집용.  
*   **Decide** (읽기 전용, 차단): 사용자 정의 승인/거부 논리(정책)용.  
*   **Transform** (수정, 차단): 전송 중인 데이터 정화(sanitize) 또는 도구 오류 복구용.

---

### 주요 기능

*   **스트리밍:** 실시간 모델 추론 및 출력 청크가 생성되는 대로 액세스합니다.  
*   **멀티모달 입력:** `from_file()`을 사용하여 이미지, PDF, 오디오 및 비디오를 기본적으로 전달합니다.  
*   **서브에이전트:** 독립적인 도구와 컨텍스트를 가진 하위 에이전트를 스폰하여 다중 에이전트 팀을 빌드합니다.  
*   **구조화된 출력:** Pydantic 모델을 사용하여 스키마를 정의하고 검증된 유형의 데이터를 직접 반환합니다.  
*   **Human-in-the-Loop:** 실행을 일시 중지하여 구조화된 질문을 하고 사용자 입력에 따라 분기합니다.  
*   **관찰 가능성:** 턴당 및 누적 토큰 사용량을 추적하고 추론 과정을 확인합니다.

Antigravity 2.0 내에서 SDK를 더 쉽게 사용하려면 Antigravity SDK 스킬을 사용하십시오. Antigravity SDK에 대해 자세히 알아보고 더 많은 사용 예제를 보려면 [**GitHub 저장소**](https://github.com/google-antigravity/antigravity-sdk-python)를 방문하세요.
