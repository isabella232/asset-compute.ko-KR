---
title: 문제 해결 [!DNL Asset Compute Service]
description: 다음을 사용하여 사용자 정의 응용 프로그램 문제 해결 및 디버그 [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# 문제 해결 {#troubleshoot}

asset compute 서비스 관련 문제를 해결하는 데 도움이 되는 몇 가지 일반적인 문제 해결 팁은 다음과 같습니다.

* 시작 시 JavaScript 응용 프로그램이 충돌하지 않는지 확인하십시오. 이러한 충돌은 일반적으로 라이브러리 누락 또는 종속성과 관련이 있습니다.
* 설치할 모든 종속성이 애플리케이션의 `package.json` 파일.
* 실패 시 정리 시 발생할 수 있는 모든 오류가 원래 문제를 숨기는 자체 오류를 생성하지 않도록 합니다.

* 새 도구를 사용하여 처음으로 개발자 도구를 시작할 때 [!DNL Asset Compute Service] 통합하면 Asset compute 이벤트 저널이 완전히 설정되지 않은 경우 첫 번째 처리 요청에 실패할 수 있습니다. 다른 요청을 보내기 전에 저널이 설정될 때까지 잠시 기다립니다.
* 오류가 발생하면 Asset compute 전송 `/register` 또는 `/process` 요청에서 필요한 모든 API가 [!DNL Adobe I/O] 프로젝트 및 작업 공간(즉, Asset compute) [!DNL Adobe I/O] 이벤트, [!DNL Adobe I/O] 이벤트 관리 및 [!DNL Adobe I/O] 런타임.

## 를 통해 문제 로그인 [!DNL Adobe I/O] CLI {#login-via-aio-cli}

에 로그인하는 데 문제가 있는 경우 [!DNL Adobe Developer Console] [다음을 통해 [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)를 설치한 다음 사용자 정의 응용 프로그램을 개발, 테스트 및 배포하는 데 필요한 자격 증명을 수동으로 추가합니다.

1. 에서 Adobe Developer App Builder 프로젝트 및 작업 영역으로 이동합니다. [Adobe Developer 콘솔](https://console.adobe.io/) 및 누르기 **[!UICONTROL 다운로드]** 오른쪽 상단에서 이 파일을 열고 이 JSON을 컴퓨터의 안전한 위치에 저장합니다.

1. Adobe Developer App Builder 애플리케이션에서 ENV 파일로 이동합니다.

1. 추가 [!DNL Adobe I/O] 런타임 자격 증명. 가져오기 [!DNL Adobe I/O] 다운로드한 JSON의 런타임 자격 증명입니다. 자격 증명은 아래에 있습니다. `project.workspace.services.runtime`. 추가 [!DNL Adobe I/O] 의 런타임 자격 증명 `AIO_runtime_XXX` 변수:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 1단계에서 다운로드한 JSON에 절대 경로를 추가합니다.

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 나머지 를 설정합니다. [필수 자격 증명](develop-custom-application.md) 개발자 도구에 필요합니다.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
