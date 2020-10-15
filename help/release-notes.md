---
title: 릴리스 노트 [!DNL Asset Compute Service].
description: 새로운 기능, 개선 사항 및 알려진 문제 [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
ht-degree: 0%

---


# 릴리스 노트 [!DNL Asset Compute Service] {#release-notes}

최신 릴리스 [!DNL Asset Compute Service] 는 2020년 7월 30일에 릴리스됩니다.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## What is new {#what-is-new}

첫 번째 릴리스입니다 [!DNL Asset Compute Service]. 디지털 에셋을 처리할 수 있는 확장 가능한 서비스 [!DNL Adobe Experience Cloud] 입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 축소판, 추출한 텍스트 및 메타데이터, 보관 파일 등 다양한 표현물로 변환할 수 있습니다.

현재, 이 [!DNL Asset Compute Service] 는 Cloud Service [!DNL Experience Manager] 로 사용할 수만 있습니다.

## 제한 및 알려진 문제 {#known-limitations}

개발자 도구를 사용하여 사용자 지정 응용 프로그램을 [테스트하려면](https://github.com/adobe/asset-compute-devtool)[클라우드 스토리지 컨테이너에 액세스해야 합니다](https://github.com/adobe/asset-compute-devtool#prerequisites).

* Blob Store와 다른 클라우드 저장소 액세스 권한은 개발자 도구용으로만 필요합니다. [!DNL Experience Manager] 개발자 도구 없이도 사용자 정의 응용 프로그램을 만들고 테스트하고 배포할 수 있습니다.
* 여러 프로젝트에서 여러 개발자가 사용하는 공유 컨테이너일 수 있습니다.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 확장성(extension developer)의 기여도를 [인정하는 github.com/adobe](https://github.com/adobe) 의 개방형 개발 모델을 통해 개발되었습니다. 맞춤형 애플리케이션을 개발, 제작, 테스트 및 배포하기 위해 필요한 모든 구성 요소는 오픈 소스로 제공됩니다. 컴퓨팅 서비스에 기여하는 [방법과 위치를 확인하십시오](contribute-to-compute-service.md).

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
