---
title: ' [!DNL Asset Compute Service]에 필요한 개발 환경을 설정합니다.'
description: 사용자 지정 코드 만들기 및 테스트를 시작하기 위한  [!DNL Asset Compute Service] 의 개발자 환경 설정입니다.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '358'
ht-degree: 0%

---

# 개발자 환경 설정 {#create-dev-environment}

[!DNL Asset Compute Service]에 대해 개발할 수 있는 설정을 만들려면 다음 요구 사항과 지침을 따르십시오.

1. [에 대한 액세스 및 ](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) 자격 증명을  [!DNL Project Firefly]획득합니다.

1. [로컬 환경 ](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) 및 필요한 도구를 설정합니다.

1. 원활한 개발을 시작하는 데 도움이 되는 몇 가지 추가 도구는 다음과 같습니다.

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10~v12 LTS, 홀수 버전은 권장되지 않음) 및  [NPM](https://www.npmjs.com). OSX HomeBrew 사용자는 `brew install node` 을 사용하여 두 가지를 모두 설치할 수 있습니다. 그렇지 않으면 [NodeJS 다운로드 페이지](https://nodejs.org/en/)에서 다운로드하십시오.
   * NodeJS에 적합한 IDE는 디버거에 지원되는 IDE이므로 [Visual Studio Code(VS Code)](https://code.visualstudio.com)를 사용하는 것이 좋습니다. 다른 IDE를 코드 편집기로 사용할 수는 있지만 고급 사용(예: 디버거)은 아직 지원되지 않습니다.
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)  (`aio`) - 를 사용하여 를 설치합니다  `npm install -g @adobe/aio-cli@7.1.0`.

1. [사전 요구 사항](/help/understand-extensibility.md#prerequisites-and-provisioning)을 충족하는지 확인하십시오.

## Firefly 프로젝트 설정 {#create-firefly-project}

1. [!DNL Experience Cloud] 조직에서 시스템 관리자 또는 개발자 역할을 확인합니다. 이 설정은 [Admin Console](https://adminconsole.adobe.com/overview)에서 시스템 관리자가 설정합니다.

1. [Adobe 개발자 콘솔에 로그인합니다](https://console.adobe.io/). [!DNL Cloud Service] 통합과 동일한 [!DNL Experience Cloud] 조직에 속해 있는지 확인합니다. [!DNL Experience Manager] Adobe 개발자 콘솔에 대한 자세한 내용은 [콘솔 설명서](https://www.adobe.io/apis/experienceplatform/console/docs.html)를 참조하십시오.

1. [Firefly 프로젝트를 만듭니다](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). **[!UICONTROL 새 프로젝트 만들기]** > **[!UICONTROL 템플릿]**&#x200B;에서 프로젝트 를 클릭합니다. Firefly 를 선택합니다. 두 작업 공간으로 새 Firefly 프로젝트를 만듭니다. `Production` 및 `Stage` 필요에 따라 추가 작업 공간(예: `Development`)을 추가합니다.

1. Firefly 프로젝트에서 작업 공간을 선택하고 Asset compute에 필요한 서비스에 가입합니다. **Add to Project** > **API** 를 클릭하고 `Asset Compute`, `IO Events` 및 `IO Events Management` 서비스를 추가합니다. 첫 번째 API를 추가할 때 개인 키를 만들라는 메시지가 표시됩니다. 개발자 도구로 사용자 지정 애플리케이션을 테스트하는 데 이 키가 필요하므로 시스템에 이 정보를 저장합니다.

## 다음 단계 {#next-step}

환경이 설정되었으므로 이제 [사용자 지정 응용 프로그램](develop-custom-application.md)을 만들 준비가 되었습니다.

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
