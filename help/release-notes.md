---
title: ' [!DNL Asset Compute Service]의 릴리스 노트입니다.'
description: ' [!DNL Asset Compute Service]의 새로운 기능, 개선 사항 및 알려진 문제'
translation-type: tm+mt
source-git-commit: c57867cd896e4ccb9402e6eeb0eea133faaa0e5d
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---


# [!DNL Asset Compute Service] {#release-notes}의 릴리스 노트

[!DNL Asset Compute Service]의 최신 릴리스는 2020년 7월 30일에 출시됩니다.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## 새로운 기능 {#what-is-new}

이 버전은 [!DNL Asset Compute Service]의 첫 번째 릴리스입니다. 디지털 에셋을 처리하기 위해 [!DNL Adobe Experience Cloud]의 확장 가능한 서비스입니다. 이미지, 비디오, 문서 및 기타 파일 형식을 축소판, 추출한 텍스트 및 메타데이터, 보관 파일 등 다양한 표현물로 변환할 수 있습니다.

현재 [!DNL Asset Compute Service]은 [!DNL Experience Manager]에서만 [!DNL Cloud Service]로 사용할 수 있습니다.

## 제한 및 알려진 문제 {#known-limitations}

사용자 지정 응용 프로그램을 [개발자 도구](https://github.com/adobe/asset-compute-devtool)로 테스트하려면 [클라우드 저장소 컨테이너](https://github.com/adobe/asset-compute-devtool#prerequisites)에 액세스해야 합니다.

* 개발자 도구에 대해서만 클라우드 저장소([!DNL Experience Manager] blob 스토어와 다름) 액세스가 필요합니다. 개발자 도구 없이도 사용자 정의 응용 프로그램을 만들고 테스트하고 배포할 수 있습니다.
* 여러 프로젝트에서 여러 개발자가 사용하는 공유 컨테이너일 수 있습니다.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 확장성(extensibility)은 확장 개발자의  [공헌을 ](https://github.com/adobe) 환영합니다. github.com/adobebe의 개방형 개발 모델로 개발되었습니다. 맞춤형 애플리케이션을 개발, 제작, 테스트 및 배포하기 위해 필요한 모든 구성 요소는 오픈 소스로 제공됩니다. 계산 서비스[에 기여할 방법 및 위치를 참조하십시오.](contribute-to-compute-service.md)

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
