---
slug: cli-tutorial
section: Antigravity CLI
title: 튜토리얼
path:
  - Antigravity CLI
  - 튜토리얼
---

# Antigravity CLI 튜토리얼

Antigravity CLI를 실행하고, 자율적인 로컬 에이전트와 협업하며, 생성된 파일을 검토하고, 터미널 테스트 명령을 실행하는 방법을 배웁니다.

## 개요

이 가이드는 빠른 온보딩 실습 과정을 안내합니다. 자율 에이전트에게 Python 유틸리티 스크립트를 생성하도록 지시하고, 변경 사항을 검토하며, 실행 결과를 확인해 봅니다.

## 단계별 안내

1. 새로운 프로젝트 디렉토리를 생성하고 Antigravity TUI를 실행합니다.

   ```bash
   mkdir agy-demo && cd agy-demo
   agy
   ```

   <Announcement>
   icon: info
   iconColor: var(--theme-primary)
   color: var(--theme-surface-surface-container)
   text: **최초 실행**: `agy`를 처음 실행하는 경우, 터미널의 안내에 따라 자동(silent) 인증을 완료하세요. 문제 해결과 관련된 세부 정보는 [설치 및 인증](/docs/cli-install)을 참조하세요.
   </Announcement>

2. 에이전트에게 Python 스크래핑 스크립트를 생성하도록 프롬프트를 입력합니다.

   화면 하단의 프롬프트 입력창에 다음 지침을 입력하고 `Enter`를 누릅니다.

   ```text
   Write a simple python script to fetch web page text
   ```

   에이전트는 작업 공간을 읽고, 파일이 존재하지 않음을 확인한 뒤 스크립트를 작성하기 위한 계획을 수립합니다. 에이전트가 추론을 수행하고 작업을 예약함에 따라 실시간 업데이트가 화면에 표시됩니다.

3. 아티팩트 검토 화면을 열어 제안된 코드를 검사합니다.

   에이전트가 파일 생성을 완료하면 알림이 나타납니다. `ctrl+r`을 눌러 **아티팩트 검토 (Artifact Review)** 화면으로 이동합니다.
   - `↑`/`↓` 키를 사용하여 새로 생성된 `main.py`로 이동합니다.
   - 전체 파일 내용과 차이점(diff)을 검토합니다.
   - `main.py` 생성을 승인하려면 `y`를 누릅니다.
   - `Esc`를 눌러 검토 패널을 닫고 메인 프롬프트로 돌아갑니다.

4. 에이전트와 함께 테스트 명령을 실행하여 출력을 검증합니다.

   에이전트에게 Python 스크립트를 실행하여 동작을 확인하도록 지시합니다. 프롬프트 창에 다음 명령을 입력하고 `Enter`를 누릅니다.

   ```text
   Run the python script and show me the output
   ```

   에이전트가 `python3 main.py` 실행을 제안합니다. `y`를 눌러 명령 실행을 확인하고 승인합니다. 에이전트는 로컬에서 스크립트를 실행하고 표준 출력을 터미널 화면에 직접 스트리밍합니다.

5. Antigravity 세션 종료하기

   작업을 완료한 후, 프롬프트 입력창에서 `ctrl+d`를 누르거나 `/exit`를 입력하여 TUI를 닫고 원래의 쉘 세션으로 돌아갑니다.

## 다음 단계

에이전트 지원을 받는 첫 번째 워크플로우를 실행했으므로, 이제 CLI를 구성하고 핵심 개념을 마스터하는 방법을 배워보세요.

- **[설치 및 인증](/docs/cli-install)**: `agy` 설치 및 SSH 프로필 설정에 대한 자세한 안내입니다.
- **[프롬프트 및 상호작용](/docs/cli-prompting)**: 다중 라인 입력, 미디어 파일 붙여넣기 및 활성 중단 제어에 대한 모범 사례입니다.
- **[아티팩트 검토](/docs/cli-artifacts)**: "투명성을 통한 신뢰(Trust through Transparency)" 아키텍처 패턴에 대해 자세히 알아봅니다.
