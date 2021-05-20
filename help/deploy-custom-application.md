---
title: 사용자 지정 응용 프로그램 배포 [!DNL Asset Compute Service] 사용자 지정
description: '사용자 지정 응용 프로그램을 배포합니다. [!DNL Asset Compute Service] '
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 0%

---

# 사용자 지정 응용 프로그램 {#deploy-custom-application} 배포

응용 프로그램을 배포하려면 [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) 명령을 사용합니다. 단말에서 명령은 사용자 정의 응용 프로그램에 액세스할 URL을 표시합니다. URL은 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]` 형식입니다.

응용 프로그램을 다시 배포하지 않고 동일한 URL을 가져오려면 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) 명령을 사용합니다.

 [!DNL Experience Manager] 의 [처리 프로필의 URL을 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)로 사용하여 [!DNL Experience Manager]와 응용 프로그램을 [!DNL Cloud Service]로 통합합니다.

Firefly 프로젝트 및 작업 공간이 작업을 사용할 [!DNL Experience Manager] 환경으로 일치하는지 확인합니다. [!DNL Cloud Service] 개발, 스테이징 및 프로덕션에 대한 환경이 다릅니다. Firefly 응용 프로그램의 루트에서 ENV 파일 내에 정의된 `AIO_runtime_*` 자격 증명을 확인하여 환경을 확인할 수 있습니다. 예를 들어 `Stage` 작업 공간에 배포하려면 `AIO_runtime_namespace` 형식은 `xxxxxx_xxxxxxxxx_stage`입니다. [!DNL Experience Manager]과 을 [!DNL Cloud Service] 프로덕션 환경으로 통합하려면 Firefly `Production` 작업 공간의 애플리케이션 URL을 사용합니다.

>[!CAUTION]
>
>중요한 [!DNL Experience Manager] 환경에서 개인 작업 공간을 사용하지 마십시오.

>[!MORELIKETHIS]
>
>* [환경을 이해 및  [!DNL Experience Manager] 관리합니다 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

