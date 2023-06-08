---
title: 배포 [!DNL Asset Compute Service] 사용자 정의 애플리케이션
description: 배포 [!DNL Asset Compute Service] 사용자 지정 응용 프로그램입니다.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '190'
ht-degree: 3%

---

# 사용자 정의 애플리케이션 배포 {#deploy-custom-application}

응용 프로그램을 배포하려면 [aio 앱 배포](https://github.com/adobe/aio-cli#aio-appdeploy) 명령입니다. 터미널에서 명령은 사용자 정의 애플리케이션에 액세스하기 위한 URL을 표시합니다. URL은 형식입니다 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

응용 프로그램을 다시 배포하지 않고 동일한 URL을 가져오려면 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 명령입니다.

에서 URL 사용 [에서 프로필 처리 중 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 애플리케이션을 와 통합하려면 [!DNL Experience Manager] as a [!DNL Cloud Service].

App Builder 프로젝트 및 작업 영역이 [!DNL Experience Manager] as a [!DNL Cloud Service] 작업을 사용할 환경입니다. 여기에는 개발, 스테이징 및 프로덕션을 위한 다양한 환경이 있습니다. 다음을 확인하여 환경을 확인할 수 있습니다. `AIO_runtime_*` Adobe Developer App Builder 애플리케이션의 루트에 있는 ENV 파일 내에 정의된 자격 증명입니다. 예를 들어 를 로 배포하려면 `Stage` 작업 영역, `AIO_runtime_namespace` 은(는) 형식입니다. `xxxxxx_xxxxxxxxx_stage`. 과 통합하려면 [!DNL Experience Manager] as a [!DNL Cloud Service] 프로덕션 환경에서는 Adobe Developer App Builder의 애플리케이션 URL을 사용합니다. `Production` 작업 영역.

>[!CAUTION]
>
>중요한 업무에 개인 작업 공간 사용 안 함 [!DNL Experience Manager] 환경.

>[!MORELIKETHIS]
>
>* [의 환경 이해 및 관리 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).
