---
slug: tab
section: Antigravity IDE
title: Tab
path:
  - Antigravity IDE
  - 기능
  - Tab
---

# Antigravity IDE: Tab 및 탐색

이 가이드는 핵심 탐색 및 자동 완성 도구인 **Supercomplete**, **Tab-to-Jump** 및 **Tab-to-Import**를 다룹니다.

## Supercomplete

Supercomplete는 현재 커서 위치 근처 영역에서 코드 제안을 제공합니다.

![Supercomplete](../../.assets/image/supercomplete.png)

### 작동 방식

- **파일 전체 제안**: 제안 사항은 문서 전체에서 코드를 수정하여 변수 이름을 변경하거나 별도의 함수 정의를 동시에 업데이트하는 것과 같은 작업을 처리할 수 있습니다.
- **수락**: `Tab` 키를 눌러 변경 사항을 수락합니다.

## Tab-to-Jump

Tab-to-Jump는 커서를 이동할 문서의 다음 논리적 위치를 제안하는 유연한 탐색 도구입니다.

![Tab-to-Jump](../../.assets/image/tab_to_jump.png)

### 작동 방식

- 다음 논리적 편집이 수행될 위치로 커서를 이동하라는 제안을 하는 "Tab to jump" 아이콘이 나타납니다. `Tab`을 누르면 커서가 해당 위치로 즉시 이동합니다.
- **수락**: `Tab` 키를 눌러 점프를 수락합니다.

## Tab-to-Import

Tab-to-Import는 흐름을 끊지 않고 누락된 종속성을 처리합니다.

![Tab-to-Import](../../.assets/image/tab_to_import.png)

### 작동 방식

- **감지**: 임포트(import)되지 않은 클래스나 함수를 입력하면 Antigravity가 임포트를 제안합니다.
- **작업**: `Tab`을 누르면 단어가 완성되고 임포트 문이 파일 상단에 즉시 추가됩니다.

## 설정

설정에서 이러한 기능의 동작을 사용자 정의할 수 있습니다.
- **기능 활성화/비활성화**: 자동 완성, Tab-to-Jump, Supercomplete 또는 Tab-to-Import를 개별적으로 끌 수 있습니다.
- **탭 속도**: 제안의 응답성을 제어합니다.
  - `Slow`(느림): 제안을 제안하기 전에 더 많은 컨텍스트를 기다립니다.
  - `Default`(기본값): 균형 잡힌 속도를 제공합니다.
  - `Fast`(빠름): 빠른 연속 제안을 제공합니다.
- **삽입된 텍스트 강조 표시**: 활성화되면 Tab을 통해 삽입된 텍스트가 강조 표시되어 변경 사항을 쉽게 추적할 수 있습니다.
- **클립보드 컨텍스트**: 활성화되면 Antigravity가 클립보드의 내용을 사용하여 완성 정확도를 향상시킵니다.
- **Gitignored 파일 허용**: `.gitignore` 파일에 나열된 파일 내에서 Tab 기능(제안 및 점프)을 활성화합니다. git이 설치된 경우에만 Tab이 gitignored 파일을 무시합니다.
