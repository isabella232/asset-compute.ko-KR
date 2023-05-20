---
title: 에 필요한 개발 환경 설정 [!DNL Asset Compute Service]
description: 용 개발자 환경 설정 [!DNL Asset Compute Service] 을 클릭하여 사용자 지정 코드를 만들고 테스트하십시오.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 1%

---

# 개발자 환경 설정 {#create-dev-environment}

을(를) 개발할 수 있는 설정을 만들려면 [!DNL Asset Compute Service], 다음 요구 사항 및 지침을 따르십시오.

1. [액세스 및 자격 증명 획득](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) 대상 [!DNL Adobe Developer App Builder].

1. [로컬 환경 설정](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) 및 필수 도구입니다.

1. 원활한 개발을 시작하는 데 도움이 되는 몇 가지 추가 도구는 다음과 같습니다.

   * [Git](https://git-scm.com/)
   * [Docker 데스크탑](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, 홀수 버전은 권장되지 않음) 및 [NPM](https://www.npmjs.com). OSX HomeBrew 사용자는 다음과 같은 작업을 수행할 수 있습니다. `brew install node` 두 가지를 모두 설치합니다. 그렇지 않으면 [NodeJS 다운로드 페이지](https://nodejs.org/en/)
   * NodeJS에 적합한 IDE입니다. [Visual Studio 코드(VS 코드)](https://code.visualstudio.com) 디버거에 지원되는 IDE이므로. 다른 IDE를 코드 편집기로 사용할 수 있지만 고급 사용(예: 디버거)은 아직 지원되지 않습니다
   * 최신 버전 설치[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 다음을 충족해야 합니다. [전제 조건](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## App Builder 프로젝트 설정 {#create-App-Builder-project}

1. 에서 시스템 관리자 또는 개발자 역할을 확인합니다. [!DNL Experience Cloud] 조직. 에서 시스템 관리자가 설정합니다. [Admin Console](https://adminconsole.adobe.com/overview).

1. 에 로그온합니다. [Adobe Developer 콘솔](https://console.adobe.io/). 동일한 그룹에 속해 있는지 확인합니다. [!DNL Experience Cloud] 다음으로 조직 구성 [!DNL Experience Manager] as a [!DNL Cloud Service] 통합. Adobe Developer 콘솔에 대한 자세한 내용은 [콘솔 설명서](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [App Builder 프로젝트 만들기](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). 클릭 **[!UICONTROL 새 프로젝트 만들기]** > **[!UICONTROL 템플릿의 프로젝트]**. App Builder를 선택합니다. 다음 두 작업 공간이 있는 새 App Builder 프로젝트가 만들어집니다. `Production` 및 `Stage`. 추가 작업 공간 추가(예: ) `Development`, 필요에 따라.

1. App Builder 프로젝트에서 작업 영역을 선택하고 Asset compute에 필요한 서비스를 구독합니다. 클릭 **프로젝트에 추가** > **API** 및 추가 `Asset Compute`, `IO Events`, 및 `IO Events Management` 서비스. 첫 번째 API를 추가할 때 개인 키를 만들라는 메시지가 표시됩니다. 개발자 도구를 사용하여 사용자 정의 애플리케이션을 테스트하려면 이 키가 필요하므로 시스템에 이 정보를 저장하십시오.

## 다음 단계 {#next-step}

이제 환경이 설정되었으므로 [사용자 정의 애플리케이션 만들기](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
