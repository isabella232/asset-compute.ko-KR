---
title: ' [!DNL Asset Compute Service]의 아키텍처.'
description: ' [!DNL Asset Compute Service] API, 응용 프로그램 및 SDK가 함께 작동하여 클라우드 기본 에셋 처리 서비스를 제공하는 방법입니다.'
translation-type: tm+mt
source-git-commit: c392b8588929f7b13db13e42a3f17bbc4f68a376
workflow-type: tm+mt
source-wordcount: '494'
ht-degree: 0%

---


# [!DNL Asset Compute Service] {#overview} 아키텍처

[!DNL Asset Compute Service]은(는) 서버를 사용하지 않는 Adobe I/O Runtime 플랫폼 위에 구축되었습니다. 자산에 대한 Adobe Sensei 컨텐츠 서비스 지원을 제공합니다. 호출 클라이언트([!DNL Experience Manager]은(는) 지원됨)(A0/>만 지원됨)에는 에셋을 위해 찾던 Adobe Sensei 생성 정보가 제공됩니다. [!DNL Cloud Service] 반환된 정보는 JSON 형식으로 되어 있습니다.

[!DNL Asset Compute Service] 을 확장할 수 있습니다.  [!DNL Project Firefly] 이러한 사용자 정의 응용 프로그램은 [!DNL Project Firefly] 헤드리스 앱이며 사용자 정의 전환 도구 추가 또는 이미지 작업을 수행하기 위해 외부 API를 호출하는 등의 작업을 수행합니다.

[!DNL Project Firefly] 는 런타임에 사용자 정의 웹 애플리케이션을 구축 및 배포하는  [!DNL Adobe I/O] 프레임워크입니다. 개발자는 맞춤형 애플리케이션을 제작하기 위해 [!DNL React Spectrum](Adobe의 UI 툴킷)을 활용하고 마이크로서비스를 제작하며 맞춤형 이벤트를 만들고 API를 조정할 수 있습니다. [프로젝트 Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)의 설명서를 참조하십시오.

아키텍처의 기반이 되는 기반에는 다음이 포함됩니다.

* 주어진 작업에 필요한 것만 포함하는 애플리케이션의 모듈성으로 인해 애플리케이션을 서로 분리하고 크기를 줄일 수 있습니다.

* Adobe I/O Runtime을 서버를 사용하지 않는 개념은 다음과 같은 다양한 이점을 제공합니다.비동기적이고 확장성이 뛰어나고 분리된 작업 기반 처리를 통해 자산 처리에 완벽하게 적합합니다.

* 이진 클라우드 스토리지는 사전 서명된 URL 참조를 사용하여 저장소에 대한 전체 액세스 권한이 없어도 자산 파일 및 표현물을 개별적으로 저장 및 액세스하는 데 필요한 기능을 제공합니다. 낮은 지연 컨텐츠 액세스를 최적화할 수 있는 클라우드 스토리지와 함께 전송 가속, CDN 캐싱 및 컴퓨팅 애플리케이션을 함께 배치할 수 있습니다. AWS 및 Azure 클라우드 모두 지원됩니다.

![asset compute 서비스의 구조](assets/architecture-diagram.png)

*그림:아키텍처  [!DNL Asset Compute Service] 및  [!DNL Experience Manager]스토리지 및 처리 애플리케이션과 통합되는 방식*

아키텍처는 다음 부분으로 구성됩니다.

* **API 및 오케스트레이션** 레이어는 서비스에서 소스 에셋을 여러 변환으로 변형하도록 지시하는 요청(JSON 형식)을 받습니다. 이러한 요청은 비동기식으로 반환되며 활성화 ID(&quot;작업 ID&quot;)로 반환됩니다. 지침은 순전히 선언적인 것이며, 모든 표준 처리 작업(예: 축소판 생성, 텍스트 추출) 시 소비자는 원하는 결과만 지정하지만 특정 변환을 처리하는 애플리케이션은 지정하지 않습니다. 인증, 분석, 속도 제한 등과 같은 일반 API 기능은 서비스 앞에 있는 Adobe API 게이트웨이를 사용하여 처리되고 I/O 런타임으로 이동하는 모든 요청을 관리합니다. 응용 프로그램 라우팅은 오케스트레이션 레이어에서 동적으로 수행됩니다. 사용자 지정 응용 프로그램은 특정 변환에 대해 클라이언트가 지정할 수 있으며 사용자 지정 매개 변수를 포함할 수 있습니다. 애플리케이션 실행은 입출력 런타임에서 서버를 사용하지 않는 별도의 함수이므로 완전히 병렬화할 수 있습니다.

* **특정 유형의 파일** 형식이나 대상 표현물을 전문으로 처리하는 응용 프로그램. 개념적으로 애플리케이션은 Unix 파이프 개념과 같습니다.입력 파일이 하나 이상의 출력 파일로 변환됩니다.

* **일반적인  [응용 프로그램 ](https://github.com/adobe/asset-compute-sdk)** 라이브러리는 소스 파일 다운로드, 변환 업로드, 오류 보고, 이벤트 전송 및 모니터링과 같은 일반적인 작업을 처리합니다. 이 방식은 서버를 사용하지 않는 아이디어를 따라 애플리케이션 개발이 가능한 한 간단하게 유지되도록 설계되었으며 로컬 파일 시스템 상호 작용으로 제한될 수 있습니다.

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
