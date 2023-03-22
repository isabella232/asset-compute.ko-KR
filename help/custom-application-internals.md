---
title: 사용자 정의 애플리케이션의 작업 이해
description: 내부 작업 [!DNL Asset Compute Service] 작동 방식을 이해하는 데 도움이 되는 사용자 지정 애플리케이션입니다.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 2af710443cdc2e5e25e105eca6e779eb58631ae9
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# 사용자 지정 애플리케이션의 내부 {#how-custom-application-works}

클라이언트가 사용자 지정 애플리케이션을 사용하여 디지털 자산을 처리할 때 종단 간 워크플로우를 이해하려면 다음 그림을 사용합니다.

![사용자 지정 애플리케이션 워크플로우](assets/customworker.svg)

*그림: 을 사용하여 자산을 처리하는 절차 [!DNL Asset Compute Service].*

## 등록 {#registration}

클라이언트가 [`/register`](api.md#register) 첫 번째 요청보다 한 번 전에 [`/process`](api.md#process-request) 수신용 분개 URL을 설정하고 검색하기 위해 [!DNL Adobe I/O] Adobe Asset compute 이벤트.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

다음 [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript 라이브러리는 NodeJS 애플리케이션에서 사용하여 등록, 처리에서 비동기 이벤트 처리에 이르기까지 필요한 모든 단계를 처리할 수 있습니다. 필요한 헤더에 대한 자세한 내용은 [인증 및 인증](api.md).

## 처리 중 {#processing}

클라이언트가 [처리](api.md#process-request) 요청.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

클라이언트는 사전 서명된 URL을 사용하여 표현물의 형식을 올바르게 지정할 책임이 있습니다. 다음 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript 라이브러리는 NodeJS 애플리케이션에서 사용하여 URL을 미리 서명할 수 있습니다. 현재 라이브러리는 Azure Blob 저장 공간 및 AWS S3 컨테이너만 지원합니다.

처리 요청은 를 반환합니다 `requestId` 폴링에 사용할 수 있습니다. [!DNL Adobe I/O] 이벤트.

샘플 사용자 지정 애플리케이션 처리 요청은 아래에 나와 있습니다.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

다음 [!DNL Asset Compute Service] 사용자 지정 응용 프로그램 렌디션 요청을 사용자 지정 응용 프로그램으로 보냅니다. 제공된 애플리케이션 URL에 HTTP POST을 사용합니다. URL은 App Builder의 보안 웹 작업 URL입니다. 모든 요청은 HTTPS 프로토콜을 사용하여 데이터 보안을 극대화합니다.

다음 [asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 사용자 지정 애플리케이션에서 사용하는 는 HTTP POST 요청을 처리합니다. 또한 소스 다운로드, 표현물 업로드, 전송을 처리합니다 [!DNL Adobe I/O] 이벤트 및 오류 처리.

<!-- TBD: Add the application diagram. -->

### 애플리케이션 코드 {#application-code}

사용자 지정 코드는 로컬로 사용할 수 있는 소스 파일(`source.path`). 다음 `rendition.path` 은 자산 처리 요청의 최종 결과를 배치할 위치입니다. 사용자 지정 애플리케이션은 콜백을 사용하여 로컬에서 사용할 수 있는 소스 파일을 전달된 이름을 사용하여 변환 파일로 변환합니다(`rendition.path`). 사용자 지정 응용 프로그램은 `rendition.path` 변환을 만들려면 다음을 수행하십시오.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### 소스 파일 다운로드 {#download-source}

사용자 지정 애플리케이션은 로컬 파일만 처리합니다. 소스 파일 다운로드는 [asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### 표현물 작성 {#rendition-creation}

SDK는 비동기 를 호출합니다 [변환 콜백 함수](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) Analytics Premium이 있어야 합니다.

콜백 함수에는 [소스](https://github.com/adobe/asset-compute-sdk#source) 및 [렌디션](https://github.com/adobe/asset-compute-sdk#rendition) 개체. 다음 `source.path` 가 이미 있으며 은 소스 파일의 로컬 복사본 경로입니다. 다음 `rendition.path` 은 처리된 표현물을 저장해야 하는 경로입니다. 다음 경우가 아니라면 [disableSourceDownload 플래그](https://github.com/adobe/asset-compute-sdk#worker-options-optional) 이(가) 설정되면, 응용 프로그램은 `rendition.path`. 그렇지 않으면 SDK가 변환 파일을 찾거나 식별할 수 없으며 실패합니다.

그 예제의 지나치게 단순화는 사용자 정의 응용 프로그램의 구조를 설명하고 집중하기 위해 행해진다. 응용 프로그램은 소스 파일을 변환 대상에 복사합니다.

변환 콜백 매개 변수에 대한 자세한 내용은 [asset compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### 표현물 업로드 {#upload-rendition}

각 변환을 만들고 다음 방법으로 제공된 경로가 있는 파일에 저장 `rendition.path`, [asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 각 렌디션을 클라우드 저장소(AWS 또는 Azure)에 업로드합니다. 사용자 지정 애플리케이션은 동일한 애플리케이션 URL을 가리키는 여러 표현물이 들어오는 경우에만 동시에 여러 표현물을 가져옵니다. 클라우드 스토리지에 업로드하는 작업은 각 변환 후와 다음 변환에 대한 콜백을 실행하기 전에 수행됩니다.

다음 `batchWorker()` 는 모든 표현물을 실제로 처리하고 모든 표현물이 처리된 후에만 이 표현물을 업로드하므로 다른 동작을 수행합니다.

## [!DNL Adobe I/O] 이벤트 {#aio-events}

SDK에서 를 전송합니다 [!DNL Adobe I/O] 각 표현물에 대한 이벤트. 이러한 이벤트는 두 유형 중 하나입니다 `rendition_created` 또는 `rendition_failed` 결과에 따라 자세한 내용은 [비동기 이벤트 asset compute](api.md#asynchronous-events) 이벤트 세부 사항 을 참조하십시오.

## 수신 [!DNL Adobe I/O] 이벤트 {#receive-aio-events}

클라이언트가 폴링을 수행합니다 [[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) 소비 논리에 따라. 초기 분개 URL은 `/register` API 응답. 이벤트는 `requestId` 이벤트에는에 반환하는것과 같습니다 `/process`. 모든 변환에는 렌디션이 업로드되거나 실패하자마자 전송되는 별도의 이벤트가 있습니다. 일치하는 이벤트를 수신하면 클라이언트가 표시하거나 결과 변환을 처리할 수 있습니다.

JavaScript 라이브러리 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 저널이 `waitActivation()` 모든 이벤트를 가져오는 메서드.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

분개 이벤트를 가져오는 방법에 대한 자세한 내용은 [[!DNL Adobe I/O] 이벤트 API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
