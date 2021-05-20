---
title: ' [!DNL Asset Compute Service] 릴리스 노트'
description: ' [!DNL Asset Compute Service]의 새로운 기능, 개선 사항 및 알려진 문제입니다.'
exl-id: b348fa8f-0cd6-4ca1-bfe3-f31e8d6583f0
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---

# [!DNL Asset Compute Service] {#release-notes} 릴리스 노트

[!DNL Asset Compute Service] 최신 릴리스는 2020년 7월 30일에 릴리스됩니다.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## 새로운 기능 {#what-is-new}

[!DNL Asset Compute Service]의 첫 번째 릴리스입니다. 디지털 자산을 처리하는 [!DNL Adobe Experience Cloud] 의 확장 가능한 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 축소판, 추출된 텍스트 및 메타데이터, 아카이브 등의 다양한 변환으로 변환할 수 있습니다.

현재 [!DNL Asset Compute Service]은 [!DNL Experience Manager]에서만 [!DNL Cloud Service] 사용할 수 있습니다.

## 제한 사항 및 알려진 문제 {#known-limitations}

[개발자 도구](https://github.com/adobe/asset-compute-devtool)로 사용자 지정 애플리케이션을 테스트하려면 [클라우드 저장소 컨테이너](https://github.com/adobe/asset-compute-devtool#prerequisites)에 액세스해야 합니다.

* 개발자 도구에 대해서만 클라우드 저장소([!DNL Experience Manager] blob 스토어와 다른) 액세스가 필요합니다. 개발자 도구 없이도 여전히 사용자 정의 응용 프로그램을 생성, 테스트 및 배포할 수 있습니다.
* 여러 프로젝트에서 여러 개발자가 사용하는 공유 컨테이너일 수 있습니다.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 확장성은  [github.com/adobe의 개방형 개발 모델](https://github.com/adobe) 에서 개발되었으며, 이 모델은 확장 개발자의 기여를 환영합니다. 사용자 정의 응용 프로그램을 개발, 작성, 테스트 및 배포하는 것과 관련된 모든 구성 요소는 오픈 소스입니다. [계산 서비스에 기여할 방법 및 위치를 참조하십시오](contribute-to-compute-service.md).

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
