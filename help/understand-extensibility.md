---
title: 확장 [!DNL Asset Compute Service]에 대해 이해
description: 사용자 지정 자산 처리를 위해 [!DNL Asset Compute Service] 기능을 확장하는 시기와 방법.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '259'
ht-degree: 0%

---

# 확장성 소개 {#introduction-to-extensibilty}

형식으로 변환하고 이미지 크기 조정과 같은 많은 변환 요구 사항은 [Processing Profiles in a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)로 해결됩니다.  [!DNL Experience Manager]  조직의 요구 사항에 맞는 맞춤형 솔루션이 필요할 수 있습니다. [!DNL Asset Compute Service] 의 처리 프로필에서 호출되는 사용자 지정 애플리케이션을 만들어 확장할 수 있습니다  [!DNL Experience Manager]. 이러한 사용자 지정 응용 프로그램은 [지원되는 사용 사례](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)에 해당합니다.

>[!NOTE]
>
>[!DNL Asset Compute Service] 는 와  [!DNL Experience Manager] 함께만 사용할 수  [!DNL Cloud Service]있습니다.

사용자 지정 응용 프로그램은 헤드리스 [Project Firefly](https://github.com/AdobeDocs/project-firefly) 앱입니다. [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) 및 Project Firefly 개발자 도구를 통해 사용자 지정 응용 프로그램을 사용하여 [!DNL Asset Compute Service]을(를) 간단하게 확장할 수 있습니다. 이를 통해 개발자는 비즈니스 로직에 집중할 수 있습니다. 사용자 지정 응용 프로그램을 만드는 것은 서버를 사용하지 않는 [!DNL Adobe I/O] 런타임 작업을 만드는 것만큼 간단합니다. 단일 Node.js JavaScript 함수입니다. [기본 사용자 지정 응용 프로그램 예제](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)가 이를 보여 줍니다.

## 사전 요구 사항 및 프로비저닝 요구 사항 {#prerequisites-and-provisioning}

다음 전제 조건을 충족하는지 확인합니다.

* Project Firefly 도구가 컴퓨터에 설치됩니다.
* [!DNL Experience Cloud] 조직. 추가 정보 [여기](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* 경험 조직에는 [!DNL Cloud Service]이 활성화되어 있어야 합니다.[!DNL Experience Manager]
* [!DNL Adobe Experience Cloud] 조직은  [!DNL Project Firefly] 개발자 미리 보기 프로그램의 일부입니다. [액세스](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md)에 적용하는 방법 을 참조하십시오.
* 개발자를 위해 조직에서 개발자 역할이나 관리자 권한을 확인합니다.
* [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)가 로컬로 설치되었는지 확인합니다.

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
