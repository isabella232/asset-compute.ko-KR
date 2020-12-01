---
title: ' [!DNL Asset Compute Service]에 필요한 개발 환경을 설정합니다.'
description: '사용자 지정 코드 작성 및 테스트를 시작하기 위한 개발자 환경 설정 [!DNL Asset Compute Service] '
translation-type: tm+mt
source-git-commit: c392b8588929f7b13db13e42a3f17bbc4f68a376
workflow-type: tm+mt
source-wordcount: '374'
ht-degree: 0%

---


# 개발자 환경 설정 {#create-dev-environment}

[!DNL Asset Compute Service]용으로 개발할 수 있는 설정을 만들려면 다음 요구 사항과 지침을 따르십시오.

1. [Project Firefox](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) 에 대한 액세스 및 자격 증명을 획득합니다.

1. [로컬 ](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) 환경과 필요한 도구를 설정합니다.

1. 원활한 개발을 시작하는 데 도움이 되는 몇 가지 도구

   * [기트](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10 - v12 LTS, 홀수 버전은 권장되지 않음) 및  [NPM](https://www.npmjs.com). OSX HomeBrew 사용자는 `brew install node`을(를) 사용하여 두 가지를 모두 설치할 수 있습니다. 그렇지 않은 경우 [NodeJS 다운로드 페이지](https://nodejs.org/en/)에서 다운로드하십시오.
   * NodeJS에 적합한 IDE는 디버거에 지원되는 IDE이므로 [Visual Studio 코드(VS 코드)](https://code.visualstudio.com)를 권장합니다. 다른 IDE를 코드 편집기로 사용할 수 있지만 고급 사용(예: 디버거)은 아직 지원되지 않습니다.
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`) - 를 사용하여  `npm install -g @adobe/aio-cli`설치합니다.

1. [사전 요구 사항](/help/understand-extensibility.md#prerequisites-and-provisioning)을 충족해야 합니다.

## Firefox 프로젝트 {#create-firefly-project} 설정

1. 경험 조직에 시스템 관리자 또는 개발자 역할 액세스 권한이 부여됩니다. 이 설정은 [Admin Console](https://adminconsole.adobe.com/overview)에서 시스템 관리자가 설정할 수 있습니다.

1. [Adobe 개발자 콘솔](https://console.adobe.io/)에 로그인합니다. [!DNL Cloud Service] 통합 시 AEM과 동일한 Adobe Experience Cloud 조직에 속해 있는지 확인합니다. Adobe 개발자 콘솔에 대한 자세한 내용은 [콘솔 문서](https://www.adobe.io/apis/experienceplatform/console/docs.html)를 참조하십시오.

1. [Firefox 프로젝트](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md) 만들기 **[!UICONTROL 새 프로젝트 만들기]** > **[!UICONTROL 템플릿]**&#x200B;에서 프로젝트를 클릭합니다. 반딧불이를 선택합니다. 두 개의 작업 영역이 있는 새 Firefox 프로젝트를 만듭니다.`Production` 및 `Stage`. 필요한 경우 `Development`과 같은 추가 작업 영역을 추가합니다.

1. Firefly 프로젝트에서 작업 영역을 선택하고 Asset compute에 필요한 서비스에 가입합니다. **프로젝트에 추가** > **API**&#x200B;를 클릭하고 `Asset Compute`, `IO Events` 및 `IO Events Management` 서비스를 추가합니다. 첫 번째 API를 추가할 때 개인 키를 만들라는 메시지가 표시됩니다. 개발자 도구를 사용하여 사용자 지정 응용 프로그램을 테스트하려면 이 키가 필요하므로 이 정보를 컴퓨터에 저장합니다.

## 다음 단계 {#next-step}

환경이 설정되었으므로 [사용자 지정 응용 프로그램](develop-custom-application.md)을(를) 만들 준비가 되었습니다.

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
