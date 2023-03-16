---
title: 확장에 대한 이해 [!DNL Asset Compute Service]
description: 확장 시기 및 방법 [!DNL Asset Compute Service] 사용자 지정 자산 처리를 수행하는 기능입니다.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 5%

---

# 확장성 소개 {#introduction-to-extensibilty}

형식으로 변환하고 이미지 크기 조정과 같은 많은 변환 요구 사항은 [의 처리 프로필 [!DNL Experience Manager] 로서의 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). 조직의 요구 사항에 맞는 맞춤형 솔루션이 필요할 수 있습니다. [!DNL Asset Compute Service] 의 처리 프로필에서 호출되는 사용자 지정 애플리케이션을 만들어 확장할 수 있습니다 [!DNL Experience Manager]. 이러한 사용자 지정 애플리케이션은 [지원되는 사용 사례](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 는 과 함께 사용할 수만 있습니다. [!DNL Experience Manager] 로서의 [!DNL Cloud Service].

사용자 지정 애플리케이션은 헤드리스 [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) 앱을 사용할 수 없습니다. 확장 [!DNL Asset Compute Service] 사용자 정의 애플리케이션을 사용하면 [asset compute SDK](https://github.com/adobe/asset-compute-sdk) 및 Adobe Developer App Builder 개발자 도구. 이를 통해 개발자는 비즈니스 로직에 집중할 수 있습니다. 사용자 지정 응용 프로그램을 만드는 것은 서버를 사용하지 않는 일반 응용 프로그램을 만드는 것만큼 간단합니다 [!DNL Adobe I/O] 런타임 작업입니다. 단일 Node.js JavaScript 함수입니다. 다음 [기본 사용자 지정 애플리케이션 예](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 설명해드리겠습니다.

## 사전 요구 사항 및 프로비저닝 요구 사항 {#prerequisites-and-provisioning}

다음 전제 조건을 충족하는지 확인합니다.

* Adobe Developer App Builder 도구가 컴퓨터에 설치됩니다.
* An [!DNL Experience Cloud] 조직 추가 정보 [여기](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* 경험 조직은 [!DNL Experience Manager] 로서의 [!DNL Cloud Service] 활성화되었습니다.
* [!DNL Adobe Experience Cloud] 조직이 의 일부입니다 [!DNL Adobe Developer App Builder] 개발자 미리 보기 프로그램. 자세한 내용은 [액세스 신청 방법](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* 개발자를 위해 조직에서 개발자 역할이나 관리자 권한을 확인합니다.
* 확인 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 가 로컬로 설치됩니다.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
