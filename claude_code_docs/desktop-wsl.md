> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# WSL에서의 Claude Code Desktop

> Windows의 WSL 2 배포판 내부에서 Code 세션 실행하기

Windows에서 Code 탭은 Windows 자체가 아닌 WSL 2 배포판 내부에서 세션을 실행할 수 있습니다. 세션의 Claude Code 프로세스, 관련 도구 및 git이 모두 배포판 내부에서 실행되며, 프로젝트가 목표로 하는 동일한 환경인 Linux 툴체인 및 네이티브 Linux 경로를 사용합니다.

리포지토리가 배포판의 파일 시스템 내에 존재할 때 WSL 세션을 사용하세요. Windows에서 해당 파일로 작업하면 네트워크 파일 시스템을 거치게 되어 속도가 느려지고 파일 감시(file watching)가 중단됩니다; 배포판 내부에서 세션을 실행하면 이 두 가지 문제가 모두 회피됩니다.

## 요구 사항

* [WSL 2](https://learn.microsoft.com/windows/wsl/install)가 포함된 Windows 10 또는 11. WSL 1은 지원되지 않습니다.
* 하나 이상의 설치된 배포판(예: Ubuntu).
* 배포판 내부에 설치된 `git`.

## WSL 세션 시작하기

<Steps>
  <Step title="배포판 선택">
    Code 탭에서 새 세션을 시작하고 환경 선택기를 여세요. 설치된 WSL 2 배포판이 **WSL** 섹션에 나타납니다. 하나를 선택하세요.
  </Step>

  <Step title="폴더 선택">
    세션은 배포판의 홈 디렉터리에서 시작됩니다. 폴더 선택기를 사용하여 프로젝트 폴더를 선택하세요. 둘러보기는 `/home/you/project`와 같은 Linux 경로와 함께 배포판 내부에서 수행됩니다.
  </Step>

  <Step title="폴더 신뢰">
    폴더에서의 첫 세션에는 워크스페이스 신뢰 대화 상자가 표시됩니다. 신뢰는 배포판 및 폴더별로 부여됩니다; 한 배포판에서 폴더를 신뢰하는 것은 다른 배포판이나 Windows의 동일한 경로에는 적용되지 않습니다.
  </Step>
</Steps>

배포판의 첫 세션은 Claude가 내부에 설정되는 동안 약간 더 오래 걸립니다. 일반 폴더 선택기에서 `\\wsl.localhost\...` 폴더를 열 수도 있으며, 해당 배포판 내부에서 다시 열립니다.

최근에 사용한 폴더는 배포판별로 선택기에 나타나므로 한 번의 클릭으로 프로젝트에 다시 연결할 수 있습니다.

## WSL 세션에서 작동하는 기능

병렬 세션, 사이드 챗, 시각적 diff 검토, 브랜치 및 풀 리퀘스트 상태, 워크트리가 모두 배포판 내부의 git 및 툴체인에 의해 뒷받침되어 작동합니다. "Open in editor"는 [Remote - WSL](https://code.visualstudio.com/docs/remote/wsl)을 통해 배포판에 연결된 VS Code를 엽니다.

몇 가지 기능은 WSL 세션에서 아직 이용할 수 없습니다: 통합 터미널, 커넥터 및 플러그인, 세션 포크(forking), 파일 브라우저 창, 입력 창에서 `@`를 입력할 때의 파일 제안.

## 관리되는 기기

조직에서 관리하는 기기에서는 WSL 세션을 사용하지 못할 수 있습니다. 세션 시작 시 기기가 관리된다는 메시지와 함께 실패하는 경우 이는 관리자에 의해 제어되는 것입니다. 관리자: 배포 가이드의 [기기에 설정이 도달하는 방식 결정](/docs/en/admin-setup#decide-how-settings-reach-devices)을 참조하세요.
