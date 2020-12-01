---
title: ' [!DNL Asset Compute Service] 사용자 지정 응용 프로그램을 배포합니다.'
description: ' [!DNL Asset Compute Service] 사용자 지정 응용 프로그램을 배포합니다.'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---


# 사용자 지정 응용 프로그램 {#deploy-custom-application} 배포

응용 프로그램을 배포하려면 [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) 명령을 사용하십시오. 터미널에서 명령은 사용자 지정 응용 프로그램에 액세스하기 위한 URL을 표시합니다. URL 형식은 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`입니다.

응용 프로그램을 다시 배포하지 않고 동일한 URL을 가져오려면 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) 명령을 사용하십시오.

Experience Manager의 [처리 프로필의 URL을 Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)로 사용하여 응용 프로그램을 [!DNL Experience Manager]와 Cloud Service으로 통합합니다.

Firefly 프로젝트 및 작업 영역이 작업을 사용할 Cloud Service 환경으로서 [!DNL Experience Manager]에 해당하는지 확인합니다. 개발, 스테이징 및 프로덕션에 대한 환경이 다릅니다. Firefly 응용 프로그램 루트의 ENV 파일 내에 정의된 자격 증명을 확인하여 환경을 확인할 수 있습니다. `AIO_runtime_*` 예를 들어 `Stage` 작업 영역에 배포하려면 `AIO_runtime_namespace` 형식은 `xxxxxx_xxxxxxxxx_stage`입니다. Cloud Service 프로덕션 환경으로 [!DNL Experience Manager]과 통합하려면 Firefly `Production` 작업 영역의 응용 프로그램 URL을 사용하십시오.

>[!CAUTION]
>
>중요한 [!DNL Experience Manager] 환경에서 개인 작업 영역을 사용하지 마십시오.

>[!MORELIKETHIS]
>
>* [Experience Manager의 환경을 Cloud Service으로 파악하고 관리합니다](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

