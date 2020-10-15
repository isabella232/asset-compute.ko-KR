---
title: 사용자 정의 애플리케이션의 작동 방식을 파악합니다.
description: 사용자 [!DNL Asset Compute Service] 정의 애플리케이션의 내부 작업을 통해 애플리케이션 작동 방식을 이해할 수 있습니다.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '774'
ht-degree: 0%

---


# 사용자 지정 응용 프로그램의 내부 {#how-custom-application-works}

다음 그림을 사용하여 클라이언트가 사용자 정의 응용 프로그램을 사용하여 디지털 자산을 처리할 때 엔드 투 엔드 워크플로우를 파악합니다.

![맞춤형 애플리케이션 워크플로우](assets/customworker.png)

*그림:자산을 사용하는 데 관련된 절차 [!DNL Asset Compute Service].*

## 등록 {#registration}

Adobe 자산 컴퓨팅에 대한 Adobe I/O 이벤트 수신을 위한 저널 URL을 설정하고 검색하기 위해 클라이언트는 첫 번째 요청 [`/register`](api.md#register) [`/process`](api.md#process-request) 전에 한 번 호출해야 합니다.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

NodeJS 응용 프로그램에서 [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript 라이브러리를 사용하여 등록, 처리에서 비동기 이벤트 처리에 이르기까지 필요한 모든 단계를 처리할 수 있습니다. 필요한 헤더에 대한 자세한 내용은 [인증 및 인증을 참조하십시오](api.md).

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

클라이언트는 미리 서명된 URL을 사용하여 표현물의 서식을 올바르게 지정할 책임이 있습니다. JavaScript [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) 라이브러리는 NodeJS 애플리케이션에서 사용하여 URL을 미리 서명할 수 있습니다. 현재 라이브러리는 Azure Blob 저장소 및 AWS S3 컨테이너만 지원합니다.

처리 요청은 Adobe I/O 이벤트 `requestId` 를 폴링하는 데 사용할 수 있는 항목을 반환합니다.

샘플 사용자 지정 응용 프로그램 처리 요청은 아래에 나와 있습니다.

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

사용자 [!DNL Asset Compute Service] 지정 응용 프로그램 변환 요청을 사용자 지정 응용 프로그램으로 보냅니다. 제공된 응용 프로그램 URL에 HTTP POST을 사용합니다. 이 URL은 Project Firefly의 보안 웹 작업 URL입니다. 모든 요청은 데이터 보안을 최대화하기 위해 HTTPS 프로토콜을 사용합니다.

사용자 [지정 응용 프로그램에서](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 사용하는 자산 계산 SDK는 HTTP POST 요청을 처리합니다. 또한 소스 다운로드, 변환 업로드, I/O 이벤트 전송 및 오류 처리를 처리합니다.

<!-- TBD: Add the application diagram. -->

### Application code {#application-code}

사용자 지정 코드는 로컬에서 사용 가능한 소스 파일(`source.path`)을 가져오는 콜백만 제공해야 합니다. 자산 처리 요청 `rendition.path` 의 최종 결과를 배치할 위치입니다. 사용자 지정 응용 프로그램은 콜백을 사용하여 (`rendition.path`) 전달된 이름을 사용하여 로컬에서 사용 가능한 소스 파일을 변환 파일로 변환합니다. 변환을 만들려면 사용자 정의 응용 프로그램 `rendition.path` 이 작성해야 합니다.

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

사용자 지정 응용 프로그램은 로컬 파일만 취급합니다. 소스 파일 다운로드는 [자산 계산 SDK에서 처리됩니다](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### 변환 만들기 {#rendition-creation}

SDK는 각 변환에 대해 비동기 [변환 콜백 함수를](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 호출합니다.

콜백 함수에는 [소스](https://github.com/adobe/asset-compute-sdk#source) 및 [변환](https://github.com/adobe/asset-compute-sdk#rendition) 개체에 대한 액세스 권한이 있습니다. 소스 파일 `source.path` 의 로컬 복사본 경로입니다. 이 `rendition.path` 는 처리된 표현물을 저장해야 하는 경로입니다. disableSourceDownload [플래그가](https://github.com/adobe/asset-compute-sdk#worker-options-optional) 설정되어 있지 않으면 응용 프로그램은 정확하게 이 플래그를 사용해야 합니다 `rendition.path`. 그렇지 않으면 SDK에서 변환 파일을 찾거나 식별할 수 없으며 오류가 발생합니다.

그 예제의 지나치게 단순화는 사용자 정의 응용 프로그램의 구조에 대해 설명하고 집중하기 위해 행해진다. 이 응용 프로그램은 소스 파일을 변환 대상에 복사하기만 하면 됩니다.

변환 콜백 매개 변수에 대한 자세한 내용은 [자산 계산 SDK API를 참조하십시오](https://github.com/adobe/asset-compute-sdk#api-details).

### 변환 업로드 {#upload-rendition}

각 변환이 작성되어 다음 경로를 가진 파일에 저장되면, `rendition.path`자산 계산 SDK는 [](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 각 변환을 클라우드 스토리지(AWS 또는 Azure)에 업로드합니다. 사용자 지정 응용 프로그램은 동일한 응용 프로그램 URL을 가리키는 여러 표현물을 동시에 가져옵니다. 단, 들어오는 요청에 동일한 응용 프로그램 URL을 가리키는 여러 표현물이 있을 경우에만. 클라우드 스토리지에 업로드는 각 변환 후에 수행되고 다음 변환에 대한 콜백을 실행하기 전에 수행됩니다.

이 `batchWorker()` 는 모든 표현물을 실제로 처리하며 모든 표현물이 처리된 후에만 이러한 표현물을 업로드하므로 다른 비헤이비어가 있습니다.

## Adobe I/O 이벤트 {#aio-events}

SDK는 각 변환에 대한 Adobe I/O 이벤트를 전송합니다. 이러한 이벤트는 결과에 따라 유형 `rendition_created` 또는 `rendition_failed` 달라집니다. 이벤트에 대한 자세한 내용은 [자산 계산 비동기 이벤트](api.md#asynchronous-events) 를 참조하십시오.

## Adobe I/O 이벤트 받기 {#receive-aio-events}

클라이언트는 소비 논리에따라 [Adobe I/O 이벤트 저널을](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) 폴링합니다. 초기 저널 URL은 API 응답에서 제공되는 `/register` URL입니다. 이벤트는 이벤트에 있는 이벤트 `requestId` 를 사용하여 식별할 수 있으며, 반환된 이벤트와 동일합니다 `/process`. 모든 변환에는 변환이 업로드되거나 실패하자마자 전송되는 별도의 이벤트가 있습니다. 일치하는 이벤트를 수신하면 클라이언트가 결과 변환을 표시하거나 처리할 수 있습니다.

JavaScript 라이브러리 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 는 모든 이벤트를 가져오는 방법을 사용하여 저널 폴링을 `waitActivation()` 간단하게 만듭니다.

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

저널 이벤트를 가져오는 방법에 대한 자세한 내용은 [Adobe I/O 이벤트 API를 참조하십시오](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
