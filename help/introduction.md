---
title: ' [!DNL Asset Compute Service] 소개'
description: '[!DNL Asset Compute Service] 는 복잡성을 줄이고 확장성을 향상시키는 클라우드 기반의 자산 처리 서비스입니다.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '314'
ht-degree: 0%

---

# [!DNL Asset Compute Service] 개요 {#overview}

[!DNL Asset Compute Service] 는 디지털 자산을 처리할 수  [!DNL Adobe Experience Cloud] 있는 확장 가능한 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 축소판, 추출된 텍스트 및 메타데이터, 아카이브 등의 다양한 변환으로 변환할 수 있습니다.

개발자는 사용자 지정 자산 애플리케이션(사용자 지정 작업자라고도 함)을 플러그인으로 사용하여 사용자 지정 사용 사례를 처리할 수 있습니다. 이 서비스는 [!DNL Adobe I/O] 런타임에서 작동합니다. Node.js에 작성된 [!DNL Project Firefly] 헤드리스 앱을 통해 확장 가능합니다. 이러한 작업은 외부 API를 호출하여 이미지 작업을 수행하거나 [!DNL Adobe Sensei] 지원을 활용하는 등의 사용자 지정 작업을 수행할 수 있습니다.

[!DNL Project Firefly] 는  [!DNL Adobe I/O] Adobe Experience Cloud 솔루션을 확장하기 위해 런타임 시 사용자 지정 웹 애플리케이션을 구축 및 배포하는 프레임워크입니다. 사용자 지정 애플리케이션을 만들기 위해 개발자는 [!DNL React Spectrum](Adobe의 UI 툴킷)을 활용하고, 마이크로 서비스를 만들고, 사용자 지정 이벤트를 만들고, API를 오케스트레이션할 수 있습니다. Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)의 [설명서를 참조하십시오.

>[!NOTE]
>
>현재 [!DNL Asset Compute Service]은 [!DNL Experience Manager]을 (를) 통해서만 [!DNL Cloud Service] 를 사용할 수 있습니다. 관리자는 [!DNL Asset Compute Service] 을 호출하여 처리할 자산을 전달할 수 있는 처리 프로필을 만듭니다. [자산 마이크로서비스 및 처리 프로필 사용](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)을 참조하십시오.

## 지원되는 [!DNL Asset Compute Service] {#possible-use-cases-benefits} 사용 사례

[!DNL Asset Compute Service] 에서는 기본 이미지 처리와 같은 몇 가지 일반적인 비즈니스 사용 사례를 지원합니다.Adobe 애플리케이션별 전환복잡한 비즈니스 요구 사항을 조율하는 맞춤형 애플리케이션 제작

[!DNL Asset Compute] 웹 서비스를 사용하여 지원되는 [파일 형식](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)에 대한 고품질 이미지 렌더링과 다양한 파일 형식에 대한 축소판 그림을 생성할 수 있습니다. 사용자 지정 구성](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)을 통해 지원되는 [사용 사례를 참조하십시오.

>[!NOTE]
>
>서비스가 자산 저장소를 제공하지 않습니다. 사용자는 이 파일을 제공하고 클라우드 저장소의 소스 및 표현물 파일 위치에 대한 참조를 제공합니다.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [자산 마이크로서비스를 사용한 자산 처리 개요  [!DNL Adobe Experience Manager] 는  [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)입니다.
>* [Project Firefly 설명서](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [처리에 지원되는 파일 형식](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).
>* [asset compute 서비스의 릴리스 노트](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
