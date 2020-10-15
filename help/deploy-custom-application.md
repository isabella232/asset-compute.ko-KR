---
title: 사용자 [!DNL Asset Compute Service] 정의 응용 프로그램을 배포합니다.
description: 사용자 [!DNL Asset Compute Service] 정의 응용 프로그램을 배포합니다.
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '201'
ht-degree: 8%

---


# 사용자 정의 응용 프로그램 배포 {#deploy-custom-application}

응용 프로그램을 배포하려면 [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) 명령을 사용합니다. 터미널에서 명령은 사용자 지정 응용 프로그램에 액세스하기 위한 URL을 표시합니다. URL은 형식입니다 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

애플리케이션을 다시 배포하지 않고 동일한 URL을 가져오려면 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) 명령을 사용합니다.

Experience Manager의 [처리 프로필의 URL을 Cloud Service](https://docs.adobe.com/content/help/ko-KR/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 로 사용하여 응용 프로그램을 Cloud Service [!DNL Experience Manager] 로 통합합니다.

Firefly 프로젝트 및 작업 영역이 작업을 사용할 Cloud Service 환경 [!DNL Experience Manager] 과 일치하는지 확인합니다. 개발, 스테이징 및 프로덕션에 대한 환경이 다릅니다. Firefly 응용 프로그램 루트의 ENV 파일 내에 정의된 `AIO_runtime_*` 자격 증명을 확인하여 환경을 확인할 수 있습니다. 예를 들어, 작업 공간에 배포하기 위해 `Stage` 는 형식 `AIO_runtime_namespace` 의 형식입니다 `xxxxxx_xxxxxxxxx_stage`. Cloud Service 프로덕션 환경 [!DNL Experience Manager] 과 통합하려면 Firefly 작업 영역의 애플리케이션 URL을 `Production` 사용합니다.

>[!CAUTION]
>
>중요한 [!DNL Experience Manager] 환경에서 개인 작업 영역을 사용하지 마십시오.

>[!MORELIKETHIS]
>
>* [Experience Manager의 환경을 Cloud Service으로 파악하고 관리합니다](https://docs.adobe.com/content/help/ko-KR/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

