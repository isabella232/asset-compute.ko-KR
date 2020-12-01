---
title: ' [!DNL Asset Compute Service] 소개.'
description: '[!DNL Asset Compute Service] 은 복잡성을 줄이고 확장성을 개선하는 클라우드 기반의 자산 처리 서비스입니다.'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---


# [!DNL Asset Compute Service] {#overview} 개요

[!DNL Asset Compute Service] 디지털 자산을 처리할 수 있는 확장  [!DNL Adobe Experience Cloud] 가능한 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 축소판, 추출한 텍스트 및 메타데이터, 보관 파일 등 다양한 표현물로 변환할 수 있습니다.

개발자는 맞춤형 에셋 애플리케이션(사용자 정의 작업자라고도 함)을 플러그인으로 사용하여 사용자 정의 사용 사례를 해결할 수 있습니다. 서비스는 [!DNL Adobe I/O] 런타임에서 작동합니다. Node.js로 작성된 [!DNL Project Firefly] 헤드리스 앱을 통해 확장할 수 있습니다. 외부 API를 호출하여 이미지 작업을 수행하거나 [!DNL Adobe Sensei] 지원을 활용하는 등의 사용자 지정 작업을 수행할 수 있습니다.

[!DNL Project Firefly] 는 사용자 정의 웹 애플리케이션을  [!DNL Adobe I/O] 런타임 시 구축 및 배포하여 Adobe Experience Cloud 솔루션을 확장하는 프레임워크입니다. 개발자는 맞춤형 애플리케이션을 제작하기 위해 [!DNL React Spectrum](Adobe의 UI 툴킷)을 활용하고 마이크로 서비스를 제작하며 맞춤형 이벤트를 만들고 API를 조정할 수 있습니다. 프로젝트 Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)의 [설명서를 참조하십시오.

>[!NOTE]
>
>현재 [!DNL Asset Compute Service]은 [!DNL Experience Manager]를 통해서만 Cloud Service으로 사용할 수 있습니다. 관리자는 처리를 위해 자산을 전달할 [!DNL Asset Compute Service]을(를) 호출할 수 있는 처리 프로필을 만듭니다. [자산 마이크로서비스 및 처리 프로필 사용](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)을 참조하십시오.

## 지원되는 [!DNL Asset Compute Service] {#possible-use-cases-benefits} 사용 사례

[!DNL Asset Compute Service] 기본 이미지 처리와 같은 몇 가지 일반적인 비즈니스 사용 사례를 지원합니다.Adobe 애플리케이션별 전환복잡한 비즈니스 요구 사항을 조정하는 맞춤형 애플리케이션 제작

[!DNL Asset Compute] 웹 서비스를 사용하여 다양한 파일 유형에 대한 축소판, 지원되는 [ 파일 포맷에 대한 고품질 이미지 렌더링](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)을 생성할 수 있습니다. 사용자 지정 구성[을 통해 지원되는 사용 사례를 참조하십시오.](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)

>[!NOTE]
>
>서비스에서 자산 저장소를 제공하지 않습니다. 클라우드 스토리지의 소스 및 변환 파일 위치에 대한 참조를 제공하고 있습니다.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use Adobe I/O Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Adobe Experience Manager에서 Cloud Service으로 자산 마이크로 서비스를 사용한 자산 처리 개요](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html) 설명서
>* [파일 형식을 사용하여 처리할 수 있습니다](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).
>* [asset compute 서비스의 릴리스 노트](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
