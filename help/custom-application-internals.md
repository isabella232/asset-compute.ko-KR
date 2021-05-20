---
title: 사용자 정의 애플리케이션의 작업 이해
description: 작동 방식을 이해하는 데 도움이 되도록 [!DNL Asset Compute Service] 사용자 지정 응용 프로그램의 내부 작업.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# 사용자 지정 응용 프로그램 {#how-custom-application-works} 내부

클라이언트가 사용자 지정 애플리케이션을 사용하여 디지털 자산을 처리할 때 종단 간 워크플로우를 이해하려면 다음 그림을 사용합니다.

![사용자 지정 애플리케이션 워크플로우](assets/customworker.png)

*그림:을 사용하여 자산을 처리하는 절차  [!DNL Asset Compute Service].*

## 등록 {#registration}

Adobe Asset compute에 대한 [!DNL Adobe I/O] 이벤트를 받기 위한 저널 URL을 설정하고 검색하려면 클라이언트가 [`/process`](api.md#process-request)에 대한 첫 번째 요청 전에 [`/register`](api.md#register)을 한 번 호출해야 합니다.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

NodeJS 애플리케이션에서 [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript 라이브러리를 사용하여 등록, 처리에서 비동기 이벤트 처리에 이르기까지 필요한 모든 단계를 처리할 수 있습니다. 필요한 헤더에 대한 자세한 내용은 [인증 및 인증](api.md)을 참조하십시오.

## 처리 중 {#processing}

클라이언트가 [처리](api.md#process-request) 요청을 보냅니다.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

클라이언트는 사전 서명된 URL을 사용하여 표현물의 형식을 올바르게 지정할 책임이 있습니다. [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript 라이브러리를 NodeJS 애플리케이션에서 사용하여 URL을 사전 서명할 수 있습니다. 현재 라이브러리는 Azure Blob 저장 공간 및 AWS S3 컨테이너만 지원합니다.

처리 요청은 [!DNL Adobe I/O] 이벤트를 폴링하는 데 사용할 수 있는 `requestId`을 반환합니다.

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

[!DNL Asset Compute Service]은 사용자 정의 응용 프로그램 변환 요청을 사용자 정의 응용 프로그램으로 보냅니다. 제공된 응용 프로그램 URL에 HTTP POST을 사용합니다. 이 URL은 Project Firefly의 보안 웹 작업 URL입니다. 모든 요청은 HTTPS 프로토콜을 사용하여 데이터 보안을 극대화합니다.

사용자 지정 애플리케이션에서 사용하는 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)는 HTTP POST 요청을 처리합니다. 또한 소스 다운로드, 표현물 업로드, [!DNL Adobe I/O] 이벤트 보내기 및 오류 처리를 처리합니다.

<!-- TBD: Add the application diagram. -->

### 응용 프로그램 코드 {#application-code}

사용자 지정 코드는 로컬로 사용할 수 있는 소스 파일(`source.path`)을 가져오는 콜백만 제공해야 합니다. `rendition.path`은 자산 처리 요청의 최종 결과를 배치할 위치입니다. 사용자 지정 응용 프로그램은 콜백을 사용하여 로컬에서 사용할 수 있는 소스 파일을 전달된 이름을 사용하여 변환 파일로 변환합니다(`rendition.path`). 사용자 지정 응용 프로그램은 변환을 만들려면 `rendition.path`에 작성해야 합니다.

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

사용자 지정 애플리케이션은 로컬 파일만 처리합니다. 소스 파일 다운로드는 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)에 의해 처리됩니다.

### 변환 만들기 {#rendition-creation}

SDK는 각 변환에 대해 비동기 [변환 콜백 함수](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)를 호출합니다.

콜백 함수에는 [source](https://github.com/adobe/asset-compute-sdk#source) 및 [rendition](https://github.com/adobe/asset-compute-sdk#rendition) 개체에 액세스할 수 있습니다. `source.path`이(가) 이미 있으며 이 경로는 소스 파일의 로컬 복사본 경로입니다. `rendition.path` 은 처리된 표현물을 저장해야 하는 경로입니다. [disableSourceDownload 플래그](https://github.com/adobe/asset-compute-sdk#worker-options-optional)가 설정되지 않은 경우, 애플리케이션은 `rendition.path`를 정확히 사용해야 합니다. 그렇지 않으면 SDK가 변환 파일을 찾거나 식별할 수 없으며 실패합니다.

그 예제의 지나치게 단순화는 사용자 정의 응용 프로그램의 구조를 설명하고 집중하기 위해 행해진다. 응용 프로그램은 소스 파일을 변환 대상에 복사합니다.

표현물 콜백 매개 변수에 대한 자세한 내용은 [Asset compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details)를 참조하십시오.

### 표현물 업로드 {#upload-rendition}

각 렌디션이 만들어지고 `rendition.path`에서 제공하는 경로를 사용하는 파일에 저장되면 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)는 각 렌디션을 클라우드 저장소(AWS 또는 Azure)에 업로드합니다. 사용자 지정 애플리케이션은 동일한 애플리케이션 URL을 가리키는 여러 표현물이 들어오는 경우에만 동시에 여러 표현물을 가져옵니다. 클라우드 스토리지에 업로드하는 작업은 각 변환 후와 다음 변환에 대한 콜백을 실행하기 전에 수행됩니다.

`batchWorker()` 은 실제로 모든 표현물을 처리하고, 모든 표현물이 처리된 후에만 이러한 업로드가 수행되므로 다른 동작이 있습니다.

## [!DNL Adobe I/O] 이벤트 {#aio-events}

SDK는 각 표현물에 대해 [!DNL Adobe I/O] 이벤트를 보냅니다. 이러한 이벤트는 결과에 따라 `rendition_created` 또는 `rendition_failed` 유형입니다. 이벤트 세부 사항은 [비동기 이벤트 Asset compute](api.md#asynchronous-events)를 참조하십시오.

## [!DNL Adobe I/O] 이벤트 {#receive-aio-events} 수신

클라이언트는 해당 소비 논리에 따라 [[!DNL Adobe I/O] 이벤트 저널](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling)을 폴링합니다. 초기 저널 URL은 `/register` API 응답에 제공된 URL입니다. 이벤트에 있고 `/process`에 반환되는 이벤트와 동일한 `requestId` 를 사용하여 이벤트를 식별할 수 있습니다. 모든 변환에는 렌디션이 업로드되거나 실패하자마자 전송되는 별도의 이벤트가 있습니다. 일치하는 이벤트를 수신하면 클라이언트가 표시하거나 결과 변환을 처리할 수 있습니다.

JavaScript 라이브러리 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage)를 사용하면 `waitActivation()` 메서드를 사용하여 저널 폴링을 간단하게 수행하여 모든 이벤트를 가져옵니다.

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

분개 이벤트를 가져오는 방법에 대한 자세한 내용은 [[!DNL Adobe I/O] 이벤트 API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml)를 참조하십시오.

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
