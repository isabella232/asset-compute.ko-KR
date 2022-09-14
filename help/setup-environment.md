---
title: 에 필요한 개발 환경 설정 [!DNL Asset Compute Service]
description: 에 대한 개발자 환경 설정 [!DNL Asset Compute Service] 사용자 지정 코드 만들기 및 테스트를 시작하려면 다음을 수행하십시오.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: a50a3bdb520cbe608c5710716df80ac6e3b486e5
workflow-type: tm+mt
source-wordcount: '364'
ht-degree: 1%

---

# 개발자 환경 설정 {#create-dev-environment}

을 개발할 수 있는 설정을 만들려면 [!DNL Asset Compute Service], 다음 요구 사항 및 지침을 따르십시오.

1. [액세스 및 자격 증명 획득](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) 대상 [!DNL Project Firefly].

1. [로컬 환경 설정](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) 필요한 도구를 제공합니다.

1. 원활한 개발을 시작하는 데 도움이 되는 몇 가지 추가 도구는 다음과 같습니다.

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v12~v14 LTS, 홀수 버전은 권장되지 않음) 및 [NPM](https://www.npmjs.com). OSX HomeBrew 사용자는 다음을 수행할 수 있습니다 `brew install node` 둘 다 설치하려면 그렇지 않으면 [NodeJS 다운로드 페이지](https://nodejs.org/en/)
   * NodeJS에 적합한 IDE를 사용하는 것이 좋습니다 [Visual Studio 코드(VS 코드)](https://code.visualstudio.com) 디버거에 지원되는 IDE이므로 다른 IDE를 코드 편집기로 사용할 수 있지만 고급 사용(예: 디버거)은 아직 지원되지 않습니다
   * 최신 설치[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 을(를) 충족하는지 확인합니다. [전제 조건](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## App Builder 프로젝트 설정 {#create-App-Builder-project}

1. 에서 시스템 관리자 또는 개발자 역할을 확인합니다. [!DNL Experience Cloud] 조직 이 설정은 [Admin Console](https://adminconsole.adobe.com/overview).

1. 에 로그인합니다. [Adobe Developer 콘솔](https://console.adobe.io/). 동일한 항목에 포함되는지 확인 [!DNL Experience Cloud] 조직 [!DNL Experience Manager] 로서의 [!DNL Cloud Service] 통합. Adobe Developer 콘솔에 대한 자세한 내용은 [콘솔 설명서](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [App Builder 프로젝트 만들기](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). 클릭 **[!UICONTROL 새 프로젝트 만들기]** > **[!UICONTROL 템플릿의 프로젝트]**. 앱 빌더를 선택합니다. 두 작업 공간으로 새 App Builder 프로젝트를 만듭니다. `Production` 및 `Stage`. 예를 들어 추가 작업 공간 추가 `Development`필요한 경우 ).

1. App Builder 프로젝트에서 작업 공간을 선택하고 Asset compute에 필요한 서비스에 구독합니다. 클릭 **프로젝트에 추가** > **API** 및 추가 `Asset Compute`, `IO Events`, 및 `IO Events Management` 서비스. 첫 번째 API를 추가할 때 개인 키를 만들라는 메시지가 표시됩니다. 개발자 도구로 사용자 지정 애플리케이션을 테스트하는 데 이 키가 필요하므로 시스템에 이 정보를 저장합니다.

## 다음 단계 {#next-step}

환경이 설정되었으므로 이제 다음 작업을 수행할 준비가 되었습니다 [사용자 지정 애플리케이션 만들기](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
