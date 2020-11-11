---
title: 확장에 대해 [!DNL Asset Compute Service]이해합니다.
description: 사용자 지정 자산 처리를 위해 기능 [!DNL Asset Compute Service] 을 확장하는 시기 및 방법입니다.
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '271'
ht-degree: 0%

---


# 확장성 소개 {#introduction-to-extensibilty}

포맷으로 변환 및 이미지 크기 조정과 같은 많은 변환 요구 사항은 Cloud Service의 [처리 프로필 [!DNL Experience Manager] 에 의해 해결됩니다](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). 복잡한 비즈니스 요구 사항에는 조직의 요구 사항에 맞는 맞춤형 솔루션이 필요할 수 있습니다. [!DNL Asset Compute Service] 의 처리 프로필에서 호출되는 사용자 지정 응용 프로그램을 만들어 확장할 수 있습니다 [!DNL Experience Manager]. 이러한 사용자 지정 응용 프로그램은 [지원되는 사용 사례를](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)충족합니다.

>[!NOTE]
>
>[!DNL Asset Compute Service] 는 Cloud Service [!DNL Experience Manager] 로 사용할 수만 있습니다.

사용자 지정 애플리케이션은 헤드리스 [프로젝트 Firefox](https://github.com/AdobeDocs/project-firefly) 앱입니다. 사용자 정의 애플리케이션 [!DNL Asset Compute Service] 으로 확장할 수 있는 작업은 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) 및 Project Firefly 개발자 도구를 통해 간단합니다. 이를 통해 개발자는 비즈니스 로직에 집중할 수 있습니다. 사용자 정의 응용 프로그램을 만드는 것은 일반 서버를 사용하지 않는 Adobe I/O Runtime 동작을 만드는 것만큼 간단합니다. 단일 Node.js JavaScript 함수입니다. 기본 사용자 [지정 응용 프로그램 예는](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 이를 보여 줍니다.

## 사전 요구 사항 및 프로비저닝 요구 사항 {#prerequisites-and-provisioning}

다음 전제 조건을 충족하는지 확인합니다.

* Project Firefly 도구가 컴퓨터에 설치됩니다.
* 조직 [!DNL Experience Cloud] . 자세한 내용은 [여기를 참조하십시오](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* 경험 조직은 Cloud Service [!DNL Experience Manager] 가 활성화되어 있어야 합니다.
* [!DNL Adobe Experience Cloud] 조직은 개발자 미리 보기 프로그램의 [!DNL Project Firefly] 일부입니다. 액세스 [신청 방법을 참조하십시오](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* 개발자에 대한 조직에서 개발자 역할 또는 관리자 권한을 확인합니다.
* Adobe [I/O CLI가 로컬에](https://github.com/adobe/aio-cli) 설치되어 있는지 확인합니다.

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
