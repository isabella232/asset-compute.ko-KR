---
title: 문제 해결 [!DNL Asset Compute Service].
description: 사용자 정의 애플리케이션 문제 해결 및 디버그를 [!DNL Asset Compute Service]사용합니다.
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 1%

---


# 문제 해결 {#troubleshoot}

자산 계산 서비스 문제를 해결하는 데 도움이 되는 몇 가지 일반적인 문제 해결 팁은 다음과 같습니다.

* 시작 시 JavaScript 응용 프로그램이 중단되지 않도록 합니다. 이러한 충돌은 일반적으로 누락된 라이브러리 또는 종속성이 관련된 것입니다.
* 설치할 모든 종속성이 응용 프로그램의 `package.json` 파일에서 참조되는지 확인합니다.
* 실패 시 정리 시 발생할 수 있는 모든 오류는 원래 문제를 숨기는 고유한 오류를 생성하지 않도록 합니다.

* 새 [!DNL Asset Compute Service] 통합으로 처음으로 개발자 도구를 시작할 때, 자산 계산 이벤트 분기가 완전히 설정되지 않을 수 있으므로 첫 번째 처리 요청이 실패할 수 있습니다. 다른 요청을 보내기 전에 분기가 설정될 때까지 잠시 기다립니다.
* 에셋 계산 `/register` 또는 `/process` 요청을 전송하는 오류가 발생하는 경우 필요한 모든 API가 Adobe I/O 프로젝트 및 작업 공간(즉, 에셋 계산, IO 이벤트, IO 이벤트 관리 및 런타임)에 추가되어 있는지 확인합니다.

## Adobe I/O CLI를 통해 로그인 문제 {#login-via-aio-cli}

Adobe I/O CLI [!DNL Adobe Developer Console] 를 [](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)통해 애플리케이션에 로그인하는 데 문제가 있는 경우 사용자 정의 애플리케이션을 개발, 테스트 및 배포하는 데 필요한 자격 증명을 수동으로 추가합니다.

1. Adobe 개발자 콘솔의 Firefly 프로젝트 및 작업 영역으로 [이동하고](https://console.adobe.io/) 오른쪽 상단 모서리에서 **[!UICONTROL 다운로드를]** 누릅니다. 이 파일을 열고 이 JSON을 시스템의 안전한 장소에 저장합니다.

1. Firefly 응용 프로그램에서 ENV 파일로 이동합니다.

1. Adobe I/O Runtime 자격 증명을 추가합니다. 다운로드한 JSON에서 Adobe I/O Runtime 자격 증명을 받으십시오. 자격 증명이 아래에 있습니다 `project.workspace.services.runtime`. 변수에 입출력 런타임 자격 증명을 `AIO_runtime_XXX` 추가합니다.

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 1단계에서 다운로드한 JSON에 절대 경로를 추가합니다.

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 개발자 도구에 [필요한 나머지 자격](develop-custom-application.md) 증명을 설정합니다.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
