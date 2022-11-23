---
title: 배포 [!DNL Asset Compute Service] 사용자 지정 애플리케이션
description: 배포 [!DNL Asset Compute Service] 사용자 지정 애플리케이션입니다.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 129651ba432b75703bc27baa7081da60302f828d
workflow-type: tm+mt
source-wordcount: '184'
ht-degree: 3%

---

# 사용자 지정 애플리케이션 배포 {#deploy-custom-application}

응용 프로그램을 배포하려면 [aio 앱 배포](https://github.com/adobe/aio-cli#aio-appdeploy) 명령. 단말에서 명령은 사용자 정의 응용 프로그램에 액세스할 URL을 표시합니다. URL은 형식입니다 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

애플리케이션을 다시 배포하지 않고 동일한 URL을 가져오려면 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 명령.

URL을 [의 처리 프로필 [!DNL Experience Manager] 로서의 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 응용 프로그램을 [!DNL Experience Manager] 로서의 [!DNL Cloud Service].

App Builder 프로젝트 및 작업 공간이 [!DNL Experience Manager] 로서의 [!DNL Cloud Service] 작업을 사용할 환경입니다. 개발, 스테이징 및 프로덕션에 대한 환경이 다릅니다. 를 확인하여 환경을 확인할 수 있습니다 `AIO_runtime_*` Firefly 애플리케이션 루트의 ENV 파일 내에 정의된 자격 증명입니다. 예를 들어 `Stage` 작업 공간, `AIO_runtime_namespace` 의 형식은 입니다 `xxxxxx_xxxxxxxxx_stage`. 을 사용하여 [!DNL Experience Manager] 로서의 [!DNL Cloud Service] 프로덕션 환경에서 Firefly의 애플리케이션 URL을 사용합니다 `Production` 작업 공간.

>[!CAUTION]
>
>중요한 상황에서 개인 작업 공간을 사용하지 마십시오 [!DNL Experience Manager] 환경.

>[!MORELIKETHIS]
>
>* [의 환경 이해 및 관리 [!DNL Experience Manager] 로서의 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

