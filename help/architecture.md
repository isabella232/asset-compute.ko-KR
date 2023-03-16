---
title: 아키텍처 [!DNL Asset Compute Service]
description: 방법 [!DNL Asset Compute Service] API, 애플리케이션 및 SDK는 함께 작동하여 클라우드 기반의 자산 처리 서비스를 제공합니다.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '486'
ht-degree: 0%

---

# 아키텍처 [!DNL Asset Compute Service] {#overview}

다음 [!DNL Asset Compute Service] 서버를 사용하지 않고 빌드됨 [!DNL Adobe I/O] 런타임 플랫폼. 자산에 대한 Adobe Sensei 컨텐츠 서비스 지원을 제공합니다. 호출하는 클라이언트(만) [!DNL Experience Manager] 로서의 [!DNL Cloud Service] 가 지원됨)에는 자산에 대해 검색한 Adobe Sensei 생성 정보가 포함되어 있습니다. 반환된 정보는 JSON 형식으로 제공됩니다.

[!DNL Asset Compute Service] 을 기반으로 사용자 지정 애플리케이션을 만들어 확장 가능 [!DNL Project Adobe Developer App Builder]. 이러한 사용자 지정 애플리케이션은 [!DNL Project Adobe Developer App Builder] 헤드리스 앱과 사용자 지정 전환 도구 추가 또는 외부 API를 호출하여 이미지 작업을 수행합니다.

[!DNL Project Adobe Developer App Builder] 은 웹 환경에서 사용자 정의 웹 애플리케이션을 작성하고 배포하는 프레임워크입니다 [!DNL Adobe I/O] 런타임. 개발자는 사용자 정의 응용 프로그램을 만들기 위해 [!DNL React Spectrum] (Adobe의 UI 툴킷), 마이크로 서비스 만들기, 사용자 지정 이벤트 만들기 및 API 오케스트레이션. 자세한 내용은 [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/overview).

아키텍처가 기반으로 하는 기반은 다음과 같습니다.

* 특정 작업에 필요한 것만 포함하는 응용 프로그램의 모듈화를 사용하면 응용 프로그램을 서로 분리하고 경량화할 수 있습니다.

* 서버리스 개념 [!DNL Adobe I/O] 런타임에는 다음과 같은 다양한 이점이 있습니다. 자산 처리에 가장 적합한 비동기, 확장성이 뛰어난 고립된 작업 기반 처리.

* 이진 클라우드 저장소는 사전 서명된 URL 참조를 사용하여 저장소에 대한 전체 액세스 권한 없이 자산 파일 및 표현물을 개별적으로 저장 및 액세스하는 데 필요한 기능을 제공합니다. 클라우드 스토리지와 함께 전송 가속, CDN 캐싱 및 컴퓨팅 애플리케이션 위치를 함께 지정하면 지연 시간이 짧은 컨텐츠 액세스를 최적화할 수 있습니다. AWS 및 Azure 클라우드 모두 지원됩니다.

![asset compute 서비스 아키텍처](assets/architecture-diagram.png)

*그림: 아키텍처 [!DNL Asset Compute Service] 그리고 [!DNL Experience Manager], 저장 및 처리 애플리케이션을 포함합니다.*

아키텍처는 다음 부분으로 구성됩니다.

* **API 및 오케스트레이션 레이어** 서비스에서 소스 자산을 여러 변환으로 변형하도록 지시하는 요청(JSON 형식)을 받습니다. 요청은 비동기적으로 반환되고, 작업 ID인 활성화 ID가 반환됩니다. 지침은 순전히 선언적이며, 모든 표준 처리 작업(예: 축소판 생성, 텍스트 추출)에 대해 소비자는 원하는 결과만 지정하고 특정 변환을 처리하는 응용 프로그램은 지정하지 않습니다. 인증, 분석, 속도 제한 등의 일반 API 기능은 서비스 앞에 있는 Adobe API 게이트웨이를 사용하여 처리되고, [!DNL Adobe I/O] 런타임. 응용 프로그램 라우팅은 오케스트레이션 레이어에 의해 동적으로 수행됩니다. 사용자 지정 애플리케이션은 클라이언트가 특정 표현물에 대해 지정하고 사용자 지정 매개 변수를 포함할 수 있습니다. 에서 별도의 서버리스 기능이므로 애플리케이션 실행은 완전히 병렬화할 수 있습니다 [!DNL Adobe I/O] 런타임.

* **자산을 처리하는 애플리케이션** 특정 유형의 파일 형식이나 target 표현물을 전문으로 합니다. 개념적으로 응용 프로그램은 Unix 파이프 개념과 같습니다. 입력 파일은 하나 이상의 출력 파일로 변형됩니다.

* **A [공통 애플리케이션 라이브러리](https://github.com/adobe/asset-compute-sdk)** 소스 파일 다운로드, 변환 업로드, 오류 보고, 이벤트 전송 및 모니터링 과 같은 일반적인 작업을 처리합니다. 이 기능은 서버를 사용하지 않는 아이디어를 따라 애플리케이션 개발이 최대한 단순하게 유지되도록 설계되었으며 로컬 파일 시스템 상호 작용으로 제한될 수 있습니다.

<!-- TBD:

* About the YAML file?
* See [https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
