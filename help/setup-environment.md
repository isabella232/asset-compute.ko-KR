---
title: 에 필요한 개발 환경을 [!DNL Asset Compute Service]설정합니다.
description: 사용자 지정 코드 [!DNL Asset Compute Service] 를 만들고 테스트하는 개발자 환경 설정
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 0%

---


# 개발자 환경 설정 {#create-dev-environment}

개발할 수 있는 설정을 만들려면 다음 요구 [!DNL Asset Compute Service]사항과 지침을 따르십시오.

1. [Project Firefox에 대한 액세스 및 자격](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) 증명을 획득합니다.

1. [로컬 환경](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) 및 필요한 도구를 설정합니다.

1. 원활한 개발을 시작하는 데 도움이 되는 몇 가지 도구

   * [기트](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10 - v12 LTS, 홀수 버전은 권장되지 않음) 및 [NPM](https://www.npmjs.com). OSX HomeBrew 사용자는 두 가지 모두를 설치할 수 `brew install node` 있습니다. 그렇지 않은 경우 NodeJS [다운로드 페이지에서 다운로드하십시오](https://nodejs.org/en/).
   * NodeJS에 적합한 IDE는 디버거에 지원되는 IDE이므로 [Visual Studio 코드(VS 코드)](https://code.visualstudio.com) 를 사용하는 것이 좋습니다. 다른 IDE를 코드 편집기로 사용할 수 있지만 고급 사용(예: 디버거)은 아직 지원되지 않습니다.
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`) - 를 사용하여 `npm install -g @adobe/aio-cli`설치합니다.

1. 필수 [요구 사항을 충족하는지 확인하십시오](/help/understand-extensibility.md#prerequisites-and-provisioning).

## Firefox 프로젝트 설정 {#create-firefly-project}

1. 경험 조직에 시스템 관리자 또는 개발자 역할 액세스 권한이 부여됩니다. Admin Console의 시스템 관리자가 설정할 수 [있습니다](https://adminconsole.adobe.com/overview).

1. Adobe 개발자 [콘솔에 로그인합니다](https://console.adobe.io/). Cloud Service 통합으로 AEM과 동일한 Adobe Experience Cloud 조직에 속하는지 확인합니다. Adobe 개발자 콘솔에 대한 자세한 내용은 [콘솔 설명서를 참조하십시오](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Firefox 프로젝트](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)만들기 새 프로젝트 **[!UICONTROL 만들기]** > **[!UICONTROL 템플릿에서 프로젝트 만들기를 클릭합니다]**. 반딧불이를 선택합니다. 두 개의 작업 영역이 있는 새 Firefox 프로젝트를 만듭니다. `Production` 및 `Stage`. 필요한 경우 작업 영역을 추가할 수 `Development`있습니다.

1. Firefly 프로젝트에서 작업 영역을 선택하고 자산 계산에 필요한 서비스에 가입합니다. 프로젝트 **에 추가** > **API를** 클릭하고 `Asset Compute`서비스 `IO Events`를 추가 `IO Events Management` ,및추가합니다. 첫 번째 API를 추가할 때 개인 키를 만들라는 메시지가 표시됩니다. 개발자 도구를 사용하여 사용자 지정 응용 프로그램을 테스트하려면 이 키가 필요하므로 이 정보를 컴퓨터에 저장합니다.

## Next step {#next-step}

환경이 설정되면 사용자 지정 응용 프로그램 [을 만들 준비가 됩니다](develop-custom-application.md).

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
