---
title: 건축 [!DNL Asset Compute Service].
description: API, [!DNL Asset Compute Service] 애플리케이션 및 SDK가 연동되어 클라우드 기반의 에셋 처리 서비스를 제공합니다.
translation-type: tm+mt
source-git-commit: 0fb256f7d9f83fbae564d9fd52ee6b2f34c5d7e9
workflow-type: tm+mt
source-wordcount: '496'
ht-degree: 0%

---


# 건축 [!DNL Asset Compute Service] {#overview}

그 [!DNL Asset Compute Service] 는 서버를 사용하지 않는 Adobe I/O Runtime 플랫폼 위에 지어졌다. 자산에 대한 Adobe Sensei 콘텐츠 서비스 지원을 제공합니다. 호출 클라이언트(Cloud Service [!DNL Experience Manager] 만 지원됨)는 에셋을 위해 검색한 Adobe Sensei 생성 정보를 제공합니다. 반환된 정보는 JSON 형식으로 되어 있습니다.

[!DNL Asset Compute Service] 는 사용자 지정 응용 프로그램을 기반으로 하여 확장할 수 있습니다 [!DNL Project Firefly]. 이러한 사용자 정의 애플리케이션은 헤드리스 [!DNL Project Firefly] 앱이며 사용자 정의 전환 도구 추가 또는 외부 API를 호출하여 이미지 작업을 수행합니다.

[!DNL Project Firefly] 은 런타임에 사용자 정의 웹 애플리케이션을 구축 및 배포하기 위한 [!DNL Adobe I/O] 프레임워크입니다. 개발자는 맞춤형 애플리케이션을 제작하기 위해 [!DNL React Spectrum] (Adobe의 UI 툴킷)를 활용하고 마이크로 서비스를 제작하며 맞춤형 이벤트를 제작하고 API를 구성할 수 있습니다. 프로젝트 [Firefly 설명서를 참조하십시오](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

아키텍처를 기반으로 하는 기반에는 다음이 포함됩니다.

* 주어진 작업에 필요한 애플리케이션만 포함하는 애플리케이션의 모듈화를 통해 애플리케이션을 서로 분리하고 간단하게 유지할 수 있습니다.

* Adobe I/O Runtime의 서버를 사용하지 않는 개념은 많은 이점을 가져다 준다:비동기적이고 확장성이 뛰어나고 분리된 작업 기반 처리. 이는 자산 처리에 가장 적합합니다.

* 이진 클라우드 스토리지는 사전 서명된 URL 참조를 사용하여 저장소에 대한 전체 액세스 권한 없이도 자산 파일 및 표현물을 개별적으로 저장 및 액세스하는 데 필요한 기능을 제공합니다. 클라우드 스토리지를 통해 컴퓨팅 애플리케이션을 전송 가속화, CDN 캐싱 및 공동 위치를 통해 낮은 지연 컨텐츠 액세스를 최적화할 수 있습니다. AWS와 Azure 클라우드 모두 지원됩니다.

![자산 계산 서비스의 구조](assets/architecture-diagram.png)

*그림:애플리케이션, 스토리지 및 처리 애플리케이션과 통합되는 [!DNL Asset Compute Service] [!DNL Experience Manager]아키텍처입니다.*

아키텍처는 다음 부분으로 구성됩니다.

* **API 및 오케스트레이션 레이어는** 서비스에서 소스 에셋을 여러 변환으로 변형하도록 지시하는 요청(JSON 형식)을 받습니다. 이러한 요청은 비동기식으로 반환되며 활성화 ID(&quot;작업 ID&quot;)로 반환됩니다. 지침은 순전히 선언적인 것이며 모든 표준 처리 작업(예: 축소판 생성, 텍스트 추출) 시 소비자는 원하는 결과만 지정하지만 특정 변환을 처리하는 응용 프로그램은 지정하지 않습니다. 인증, 분석, 속도 제한 등과 같은 일반 API 기능은 서비스 앞의 Adobe API 게이트웨이를 사용하여 처리되고 I/O 런타임으로 이동하는 모든 요청을 관리합니다. 애플리케이션 라우팅은 오케스트레이션 레이어에서 동적으로 수행됩니다. 클라이언트가 특정 변환에 대해 지정하고 사용자 지정 매개 변수를 포함할 수 있습니다. 애플리케이션 실행은 I/O 런타임에서 서버를 사용하지 않는 별도의 함수로 인해 완전히 병렬될 수 있습니다.

* **특정 유형의 파일 포맷이나 대상 표현물에 특화된 자산을** 처리하는 응용 프로그램입니다. 개념적으로는 애플리케이션이 Unix 파이프 개념과 같습니다.입력 파일이 하나 이상의 출력 파일로 변환됩니다.

* **일반적인 [응용 프로그램 라이브러리는 소스 파일 다운로드, 변환 업로드, 오류 보고, 이벤트 전송 및 모니터링과 같은 일반적인 작업을 처리합니다](https://github.com/adobe/asset-compute-sdk)** . 이 기능은 서버를 사용하지 않는 아이디어에 따라 애플리케이션 개발이 가능한 한 간단하게 유지되도록 설계되었으며 로컬 파일 시스템 상호 작용으로 제한될 수 있습니다.

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
