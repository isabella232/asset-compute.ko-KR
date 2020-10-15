---
title: '[!DNL Asset Compute Service] HTTP API.'
description: '[!DNL Asset Compute Service] 사용자 정의 애플리케이션을 만드는 HTTP API'
translation-type: tm+mt
source-git-commit: 18e97e544014933e9910a12bc40246daa445bf4f
workflow-type: tm+mt
source-wordcount: '2931'
ht-degree: 2%

---


# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API의 사용은 개발 용도로 제한됩니다. API는 사용자 지정 애플리케이션을 개발할 때 컨텍스트로 제공됩니다. [!DNL Adobe Experience Manager] as a Cloud Service은 API를 사용하여 처리 정보를 사용자 지정 애플리케이션에 전달합니다. 자세한 내용은 자산 마이크로서비스 [및 처리 프로필 사용을 참조하십시오](https://docs.adobe.com/content/help/ko-KR/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 는 Cloud Service [!DNL Experience Manager] 로 사용할 수만 있습니다.

HTTP [!DNL Asset Compute Service] API의 모든 클라이언트는 다음과 같은 높은 수준의 흐름을 따라야 합니다.

1. 클라이언트는 IMS 조직에서 [!DNL Adobe Developer Console] 프로젝트로 제공됩니다. 이벤트 데이터 흐름을 구분하기 위해 각각의 개별 클라이언트(시스템 또는 환경)에는 자체 개별 프로젝트가 필요합니다.

1. 클라이언트는 [JWT(서비스 계정) 인증을 사용하여 기술 계정에 대한 액세스 토큰을 생성합니다](https://www.adobe.io/authentication/auth-methods.html).

1. 클라이언트는 저널 URL을 검색하기 위해 한 번만 [`/register`](#register) 호출합니다.

1. 클라이언트가 변환을 생성하려는 각 자산에 대해 호출합니다 [`/process`](#process-request) . 비동기 호출입니다.

1. 클라이언트는 정기적으로 저널을 폴링하여 이벤트를 [수신합니다](#asynchronous-events). 변환이 성공적으로 처리되거나(이벤트 유형`rendition_created` ) 오류가 있는 경우(`rendition_failed` 이벤트 유형) 각 요청된 변환에 대한 이벤트를 수신합니다.

@ [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) 모듈을 사용하면 Node.js 코드에서 API를 쉽게 사용할 수 있습니다.

## 인증 및 인증 {#authentication-and-authorization}

모든 API에는 액세스 토큰 인증이 필요합니다. 요청은 다음 헤더를 설정해야 합니다.

1. `Authorization` 헤더(기술 계정 토큰인 bearer 토큰이 있는 경우 Adobe 개발자 콘솔 프로젝트에서 [JWT exchange](https://www.adobe.io/authentication/auth-methods.html) 를 통해 전송됩니다. 범위가 [아래에](#scopes) 기록되어 있습니다.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 헤더를 추가합니다.

1. `x-api-key` 을 [!DNL Adobe Developers Console] 프로젝트에 있는 클라이언트 ID로 채웁니다.

### 범위 {#scopes}

액세스 토큰에 대해 다음 범위를 확인합니다.

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

따라서 [!DNL Adobe Developer Console] 프로젝트 `Asset Compute`, `I/O Events`및 `I/O Management API` 서비스를 신청해야 합니다. 개별 범위의 분류는 다음과 같습니다.

* 기본
   * scopes: `openid,AdobeID`

* 자산 계산
   * metascope: `asset_compute_meta`
   * scopes: `asset_compute,read_organizations`

* Adobe I/O 이벤트
   * metascope: `event_receiver_api`
   * scopes: `event_receiver,event_receiver_api`

* Adobe I/O 관리 API
   * metascope: `ent_adobeio_sdk`
   * scopes: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 등록 {#register}

서비스의 [!DNL Asset Compute service] 각 클라이언트 - 서비스에 가입한 고유 [!DNL Adobe Developer Console] 프로젝트 - 를 처리하기 [전에 등록해야](#register-request) 합니다. 등록 단계는 변환 처리에서 비동기 이벤트를 검색하는 데 필요한 고유한 이벤트 저널을 반환합니다.

라이프사이클이 끝날 때 클라이언트는 [등록을 취소할 수 있습니다](#unregister-request).

### 요청 등록 {#register-request}

이 API 호출은 클라이언트를 설정하고 [!DNL Asset Compute] 이벤트 저널 URL을 제공합니다. 이는 idempose 작업이므로 각 클라이언트에 대해 한 번만 호출하면 됩니다. 저널 URL을 검색하기 위해 다시 호출할 수 있습니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/register` |
| 헤더 `Authorization` | 모든 [인증 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 옵션, 여러 시스템에서 처리 요청의 엔드 투 엔드 고유한 식별자를 위해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 비어 있어야 합니다. |

### 응답 등록 {#register-response}

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | 요청 헤더와 `X-Request-Id` 동일하거나 고유하게 생성된 헤더와 같습니다. 시스템 및/또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | 필드가 `journal`있는 JSON 개체 `ok` `requestId` 로, |

HTTP 상태 코드는 다음과 같습니다.

* **200 성공**:요청이 성공하면 여기에는 이벤트 유형 `journal` (성공 시 또는 실패 시)을 통해 트리거된 비동기 처리 결과에 대해 통지되는 URL이 `/process` `rendition_created` `rendition_failed` 포함되어 있습니다.

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 권한 없음**:요청에 유효한 [인증이 없을 때 발생합니다](#authentication-and-authorization). 예를 들어 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.

* **403 금지**:요청에 유효한 [인증이 없을 때 발생합니다](#authentication-and-authorization). 올바른 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 일부 필수 서비스에 가입되어 있지 않습니다.

* **429 요청이 너무 많습니다**.이 클라이언트에 의해 시스템이 과부하가 발생했거나 다른 경우에 발생합니다. 클라이언트는 [지수 백오프로 다시 시도해야 합니다](https://en.wikipedia.org/wiki/Exponential_backoff). 몸이 비었다.
* **4xx 오류**:다른 클라이언트 오류가 발생하여 등록하지 못했습니다. 일반적으로 모든 오류에 대해 보장되지는 않지만 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx 오류**:다른 서버측 오류가 발생하여 등록하지 못할 때 발생합니다. 일반적으로 모든 오류에 대해 보장되지는 않지만 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 요청 등록 취소 {#unregister-request}

이 API 호출은 클라이언트 등록을 [!DNL Asset Compute] 취소합니다. 그 이후에는 더 이상 전화를 걸 수 없습니다 `/process`. 등록되지 않은 클라이언트 또는 아직 등록되지 않은 클라이언트에 대한 API 호출을 사용하면 오류가 `404` 반환됩니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/unregister` |
| 헤더 `Authorization` | 모든 [인증 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 옵션, 여러 시스템에서 처리 요청의 엔드 투 엔드 고유한 식별자를 위해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 비어 있음. |

### 등록 취소 응답 {#unregister-response}

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | 요청 헤더와 `X-Request-Id` 동일하거나 고유하게 생성된 헤더와 같습니다. 시스템 및/또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | 및 필드가 있는 JSON 개체 `ok``requestId` . |

상태 코드는 다음과 같습니다.

* **200 성공**:등록 및 분기가 발견되고 제거될 때 발생합니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 권한 없음**:요청에 유효한 [인증이 없을 때 발생합니다](#authentication-and-authorization). 예를 들어 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.

* **403 금지**:요청에 유효한 [인증이 없을 때 발생합니다](#authentication-and-authorization). 올바른 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 일부 필수 서비스에 가입되어 있지 않습니다.

* **404 찾을 수 없음**:지정된 자격 증명에 대한 현재 등록이 없을 때 발생합니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 요청이 너무 많습니다**.시스템에 과부하가 발생할 때 발생합니다. 클라이언트는 [지수 백오프로 다시 시도해야 합니다](https://en.wikipedia.org/wiki/Exponential_backoff). 몸이 비었다.

* **4xx 오류**:다른 클라이언트 오류가 발생하여 등록을 취소하지 못할 때 발생합니다. 일반적으로 모든 오류에 대해 보장되지는 않지만 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx 오류**:다른 서버측 오류가 발생하여 등록하지 못할 때 발생합니다. 일반적으로 모든 오류에 대해 보장되지는 않지만 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 프로세스 요청 {#process-request}

작업의 `process` 지침에 따라 소스 자산을 여러 표현물로 변환하는 작업을 제출합니다. 성공적인 완료(이벤트 유형 `rendition_created`) 또는 오류(이벤트 유형 `rendition_failed`)에 대한 알림은 이벤트 저널에 전송되며, 이벤트 분개는 요청 수를 생성하기 전에 한 번 [등록](#register) `/process` /등록하여검색해야합니다. 잘못된 형식의 요청은 400 오류 코드로 인해 즉시 실패합니다.

바이너리는 자산(URL)을 읽고 변환(URL)을 쓰는 데 필요한 Amazon AWS S3 사전 서명된 URL 또는 Azure Blob 저장소 SAS URL과 같은 URL을 사용하여 참조됩니다 `source``GET``PUT` . 클라이언트는 이러한 사전 서명된 URL을 생성합니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/process` |
| MIME 유형 | `application/json` |
| 헤더 `Authorization` | 모든 [인증 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 옵션, 여러 시스템에서 처리 요청의 엔드 투 엔드 고유한 식별자를 위해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 아래의 설명에 따라 프로세스 요청 JSON 형식이어야 합니다. 처리할 자산 및 생성할 변환에 대한 지침을 제공합니다. |

### 프로세스 요청 JSON {#process-request-json}

요청 본문 `/process` 은 다음과 같은 고급 스키마를 사용하는 JSON 개체입니다.

```json
{
    "source": "",
    "renditions" : []
}
```

사용 가능한 필드는 다음과 같습니다.

| 이름 | 유형 | 설명 | 예 |
|--------------|----------|-------------|---------|
| `source` | `string` | 처리할 소스 자산의 URL. 요청된 변환 형식(예: `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | 처리할 소스 자산을 설명합니다. 아래의 [소스 개체 필드](#source-object-fields) 설명을 참조하십시오. 요청된 변환 형식(예: `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 소스 파일에서 생성할 변환입니다. 각 변환 개체는 [변환 명령을 지원합니다](#rendition-instructions). 필수. | `[{ "target": "https://....", "fmt": "png" }]` |

URL로 `source` 보이는 URL이거나 추가 필드가 있는 URL `<string>` `<object>` 이 될 수 있습니다. 다음과 같은 변형이 유사합니다.

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### 소스 객체 필드 {#source-object-fields}

| 이름 | 유형 | 설명 | 예 |
|-----------|----------|-------------|---------|
| `url` | `string` | 처리할 소스 자산의 URL. 필수. | `"http://example.com/image.jpg"` |
| `name` | `string` | 원본 자산 파일 이름. MIME 형식을 찾을 수 없는 경우 이름의 파일 확장자를 사용할 수 있습니다. 바이너리 리소스 헤더의 URL 경로 또는 파일 이름 `content-disposition` 의 파일 이름보다 우선합니다. 기본값은 &quot;file&quot;입니다. | `"image.jpg"` |
| `size` | `number` | 소스 자산 파일 크기(바이트)입니다. 이진 리소스 `content-length` 의 헤더보다 우선합니다. | `10234` |
| `mimetype` | `string` | 원본 자산 파일 MIME 형식입니다. 이진 리소스의 `content-type` 헤더보다 우선합니다. | `"image/jpeg"` |

### 전체 `process` 요청 예 {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## 프로세스 응답 {#process-response}

요청은 `/process` 기본 요청 유효성 검사에 따라 성공 또는 실패로 즉시 반환됩니다. 실제 자산 처리는 비동기적으로 발생합니다.

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | 요청 헤더와 `X-Request-Id` 동일하거나 고유하게 생성된 헤더와 같습니다. 시스템 및/또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | 및 필드가 있는 JSON 개체 `ok``requestId` . |

상태 코드:

* **200 성공**:요청이 성공적으로 제출된 경우. 응답 JSON에는 다음이 포함됩니다 `"ok": true`.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 잘못된 요청**:요청 JSON에 필수 필드가 누락되는 등 요청이 잘못 구성된 경우. 응답 JSON에는 다음이 포함됩니다 `"ok": false`.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 권한 없음**:요청에 유효한 [인증이 없는 경우입니다](#authentication-and-authorization). 예를 들어 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.
* **403 금지**:요청에 유효한 [인증이 없는 경우입니다](#authentication-and-authorization). 올바른 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 일부 필수 서비스에 가입되어 있지 않습니다.
* **429 요청이 너무 많습니다**.시스템이 이 클라이언트 또는 일반적으로 과부하가 발생하는 경우 클라이언트는 [지수 백오프로 다시 시도할 수 있습니다](https://en.wikipedia.org/wiki/Exponential_backoff). 몸이 비었다.
* **4xx 오류**:다른 클라이언트 오류가 발생한 경우 일반적으로 모든 오류에 대해 보장되지는 않지만 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx 오류**:다른 서버측 오류가 발생한 경우 일반적으로 모든 오류에 대해 보장되지는 않지만 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

대부분의 클라이언트는 401이나 403과 같은 구성 문제나 400과 같은 잘못된 요청 [을 제외하고](https://en.wikipedia.org/wiki/Exponential_backoff) 모든 오류에 대해 ** 기하급수적인 백오프로 정확히 동일한 요청을 재시도하는 경향이 있습니다. 429개의 응답을 통해 이루어지는 정기적인 요금 제한 외에도, 일시적인 서비스 중단 또는 제한으로 인해 5xx의 오류가 발생할 수 있습니다. 그런 다음 일정 시간 후에 다시 시도할 것을 권장합니다.

모든 JSON 응답(있는 경우)에는 헤더와 `requestId` 동일한 값이 `X-Request-Id` 포함됩니다. 헤더는 항상 있으므로 읽을 것을 권장합니다. 또한 이 `requestId` 는 처리 요청과 관련된 모든 이벤트에서 반환됩니다 `requestId`. 클라이언트는 이 문자열의 형식을 추측해서는 안 됩니다. 이것은 불투명 문자열 식별자입니다.

## 사후 처리 옵트인 {#opt-in-to-post-processing}

자산 [계산 SDK는](https://github.com/adobe/asset-compute-sdk) 기본 이미지 사후 처리 옵션 집합을 지원합니다. 사용자 지정 작업자는 변환 객체의 필드를 다음으로 설정하여 게시 처리 `postProcess` 에 명시적으로 선택할 수 있습니다 `true`.

지원되는 사용 사례는 다음과 같습니다.

* crop.w, crop.h, crop.x 및 crop.y로 제한이 정의된 사각형으로 변환을 자릅니다.변환 객체 `instructions.crop` 에서 정의됩니다.
* 폭, 높이 또는 둘 다를 사용하여 이미지 크기를 조정할 수 있습니다. 변환 개체 `instructions.width` 에 의해 및 `instructions.height` 에서 정의됩니다. 너비 또는 높이만 사용하여 크기를 조정하려면 하나의 값만 설정합니다. 컴퓨팅 서비스는 종횡비를 유지합니다.
* JPEG 이미지의 품질을 설정합니다. 변환 객체 `instructions.quality` 에서 정의됩니다. 가장 좋은 품질은 낮은 품질로 `100` 표현되며 작은 값은 낮은 품질을 나타냅니다.
* 인터레이스된 이미지를 만듭니다. 변환 객체 `instructions.interlace` 에서 정의됩니다.
* 픽셀에 적용된 비율을 조정하여 데스크탑 퍼블리싱을 위해 렌더링된 크기를 조정하도록 DPI를 설정할 수 있습니다. dpi 해상도를 변경하기 위해 변환 객체 `instructions.dpi` 에서 정의됩니다. 그러나 다른 해상도에서 동일한 크기로 이미지 크기를 조정하려면 지침을 `convertToDpi` 사용하십시오.
* 렌더링된 너비 또는 높이가 지정된 대상 해상도(DPI)의 원본과 동일하게 유지되도록 이미지 크기를 조정합니다. 변환 객체 `instructions.convertToDpi` 에서 정의됩니다.

## 워터마크 에셋 {#add-watermark}

Asset [Compute SDK는](https://github.com/adobe/asset-compute-sdk) PNG, JPEG, TIFF 및 GIF 이미지 파일에 워터마크를 추가할 수 있도록 지원합니다. 변환에 있는 개체의 변환 지침에 따라 워터마크가 `watermark` 추가됩니다.

변환 사후 처리 중에 워터마크가 수행됩니다. 자산에 워터마크를 적용하려면 [사용자 지정 작업자는 변환 개체의 필드를 다음으로 설정하여](#opt-in-to-post-processing) 사후 처리 `postProcess` 에 `true`들어갑니다. 워커가 옵트인을 하지 않으면 워터마크 개체가 요청의 변환 개체에 설정되어 있어도 워터마크가 적용되지 않습니다.

## 변환 지침 {#rendition-instructions}

이러한 옵션은 `renditions` 배열 [/프로세스](#process-request)사용 가능한 옵션입니다.

### 공통 필드 {#common-fields}

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 변환 대상 형식은 텍스트 추출 및 XMP 메타데이터를 xml `text` 로 추출하는 `xmp` 용이기도 합니다. 지원되는 [형식 보기](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | 사용자 [지정 응용 프로그램의 URL](develop-custom-application.md). URL이어야 `https://` 합니다. 이 필드가 있으면 사용자 정의 응용 프로그램에서 변환이 만들어집니다. 그러면 다른 모든 세트 변환 필드가 사용자 정의 응용 프로그램에서 사용됩니다. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | HTTP PUT을 사용하여 생성된 변환을 업로드해야 하는 URL. | `http://w.com/img.jpg` |
| `target` | `object` | 생성된 변환에 대한 여러 부분으로 사전에 서명된 URL 업로드 정보. 이 다중 부분 업로드 비헤이비어가 포함된 [AEM/Oak 직접 바이너리](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) 업로드에 [사용됩니다](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>필드:<ul><li>`urls`:문자열 배열, 사전 서명된 각 부분 URL에 대해 하나씩</li><li>`minPartSize`:한 부분에 사용할 최소 크기 = url</li><li>`maxPartSize`:한 부분에 사용할 최대 크기 = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 옵션으로 제공되는 예약 공간은 클라이언트에 의해 제어되고 변환 이벤트로 전달됩니다. 클라이언트가 변환 이벤트를 식별하는 사용자 지정 정보를 추가할 수 있습니다. 클라이언트는 언제든지 변경할 수 있으므로 사용자 정의 애플리케이션을 수정하거나 의존해서는 안 됩니다. | `{ ... }` |

### 변환 특정 필드 {#rendition-specific-fields}

현재 지원되는 파일 형식 목록은 [지원되는 파일 형식을 참조하십시오](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html).

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 사용자 정의 응용 프로그램이 인식하는 고급 사용자 정의 필드를 [추가할](develop-custom-application.md) 수 있습니다. |  |
| `embedBinaryLimit` | `number` 바이트 | 이 값이 설정되고 변환의 파일 크기가 이 값보다 작은 경우, 변환 생성이 완료된 후 전송되는 이벤트에 변환이 포함됩니다. 임베드할 수 있는 최대 크기는 32KB(32 x 1024바이트)입니다. 변환의 크기가 `embedBinaryLimit` 제한보다 큰 경우 클라우드 스토리지의 위치에 배치되고 이벤트에 임베드되지 않습니다. | `3276` |
| `width` | `number` | 너비(픽셀 단위) 이미지 변환에만 해당됩니다. | `200` |
| `height` | `number` | 높이(픽셀 단위) 이미지 변환에만 해당됩니다. | `200` |
|  |  | 종횡비는 다음과 같은 경우에 항상 유지됩니다. <ul> <li> 둘 다 `width` 와 `height` 를 모두 지정한 다음, 이미지는 종횡비를 유지하면서 크기에 맞춰집니다. </li><li> 지정된 `width` 경우에만 또는 `height` 지정된 결과 이미지는 종횡비를 유지하면서 해당 차원을 사용합니다</li><li> 둘 중 하나 `width` `height` 를 지정하지 않은 경우 원본 이미지 픽셀 크기가 사용됩니다. 소스 유형에 따라 다릅니다. PDF 파일과 같은 일부 포맷의 경우 기본 크기가 사용됩니다. 최대 크기 제한이 있을 수 있습니다.</li></ul> |  |
| `quality` | `number` | 범위 내에서 jpeg 품질 `1` 을 지정합니다 `100`. 이미지 변환에만 해당됩니다. | `90` |
| `xmp` | `string` | XMP 메타데이터 원본에 대해서만 사용되므로, 지정된 변환에 다시 쓰기 위해 base64 인코딩된 XMP입니다. |  |
| `interlace` | `bool` | 인터레이스된 PNG 또는 GIF 또는 점진적 JPEG를 로 설정하여 만듭니다 `true`. 다른 파일 포맷에는 영향을 주지 않습니다. |  |
| `jpegSize` | `number` | JPEG 파일의 대략적인 크기(바이트)입니다. 모든 `quality` 설정을 무시합니다. 다른 포맷에는 영향을 주지 않습니다. |  |
| `dpi` | `number` 또는 `object` | x 및 y DPI를 설정합니다. 간단히 말해, x 및 y 모두에 사용되는 단일 숫자로 설정할 수도 있습니다.이미지 자체에는 영향을 주지 않습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 또는 `object` | x 및 y DPI의 재샘플 값을 유지하면서 실제 크기를 유지할 수 있습니다. 간단히 말해, x 및 y 모두에 사용되는 단일 숫자로 설정할 수도 있습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | ZIP 보관에 포함할 파일 목록(`fmt=zip`). 각 항목은 URL 문자열이거나 필드가 있는 개체일 수 있습니다.<ul><li>`url`:파일을 다운로드할 URL</li><li>`path`:ZIP의 이 경로 아래에 파일 저장</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP 보관에 대한 처리가 중복되었습니다(`fmt=zip`). 기본적으로 ZIP의 동일한 경로 아래에 저장된 여러 파일이 오류를 생성합니다. 설정 `duplicate` 을 `ignore` 설정하면 첫 번째 자산만 저장되고 나머지 자산은 무시됩니다. | `ignore` |
| `watermark` | `object` | 워터마크 [에 대한 지침을 포함합니다](#watermark-specific-fields). |  |

### 워터마크 특정 필드 {#watermark-specific-fields}

PNG 형식은 워터마크로 사용됩니다.

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 워터마크의 크기( `0.0` 와 사이) `1.0`. `1.0` 즉, 워터마크의 원래 크기(1:1)가 있고 값이 낮으면 워터마크 크기가 줄어듭니다. | 값은 원래 크기의 반을 `0.5` 의미합니다. |
| `image` | `url` | 워터마크에 사용할 PNG 파일의 URL. |  |

## 비동기 이벤트 {#asynchronous-events}

변환 처리가 완료되거나 오류가 발생하면 이벤트가 [Adobe I/O 이벤트 저널로 전송됩니다](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). 클라이언트는 [/register를 통해 제공되는 저널 URL을 들어야 합니다](#register). 저널 응답에는 각 이벤트에 대해 하나의 개체로 구성된 `event` 배열이 포함되며, 이 배열에 실제 이벤트 페이로드가 `event` 포함됩니다.

의 모든 이벤트에 대한 Adobe I/O 이벤트 유형은 [!DNL Asset Compute Service] 입니다 `asset_compute`. 분개는 자동으로 이 이벤트 유형에만 가입되며 Adobe I/O 이벤트 유형을 기반으로 더 이상 필터링할 필요가 없습니다. 서비스 특정 이벤트 유형은 이벤트의 `type` 속성에서 사용할 수 있습니다.

### Event types {#event-types}

| 이벤트 | 설명 |
|---------------------|-------------|
| `rendition_created` | 성공적으로 처리되고 업로드된 각 변환에 대해 전송됩니다. |
| `rendition_failed` | 처리 또는 업로드하지 못한 각 변환에 대해 전송됩니다. |

### 이벤트 속성 {#event-attributes}

| 특성 | 유형 | 이벤트 | 설명 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | JavaScript [Date.toISOString()에 정의된 대로, 단순화된 확장 ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 형식으로 이벤트가 전송된 타임스탬프 [](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | 헤더와 같은 원래 요청의 요청 ID `/process`입니다 `X-Request-Id` . |
| `source` | `object` | `*` | 요청 `source` 의 `/process` 내용입니다. |
| `userData` | `object` | `*` | 설정된 경우 요청 `userData` 의 `/process` 변환. |
| `rendition` | `object` | `rendition_*` | 전달된 해당 변환 개체 `/process`. |
| `metadata` | `object` | `rendition_created` | 변환의 [메타데이터](#metadata) 속성입니다. |
| `errorReason` | `string` | `rendition_failed` | 변환 실패 [이유](#error-reasons) (있는 경우) |
| `errorMessage` | `string` | `rendition_failed` | 변환 실패 시 자세한 내용을 제공하는 텍스트입니다. |

### 메타데이터 {#metadata}

| 속성 | 설명 |
|--------|-------------|
| `repo:size` | 변환의 크기(바이트)입니다. |
| `repo:sha1` | 변환의 sha1 다이제스트입니다. |
| `dc:format` | 변환의 MIME 형식입니다. |
| `repo:encoding` | 텍스트 기반 형식인 경우 변환의 문자 인코딩입니다. |
| `tiff:ImageWidth` | 표현물의 너비(픽셀 단위). 이미지 변환에만 표시됩니다. |
| `tiff:ImageLength` | 표현물의 길이(픽셀 단위) 이미지 변환에만 표시됩니다. |

### 오류 이유 {#error-reasons}

| 이유 | 설명 |
|---------|-------------|
| `RenditionFormatUnsupported` | 지정된 소스에는 요청된 변환 형식이 지원되지 않습니다. |
| `SourceUnsupported` | 형식이 지원되더라도 특정 소스는 지원되지 않습니다. |
| `SourceCorrupt` | 소스 데이터가 손상되었습니다. 빈 파일을 포함합니다. |
| `RenditionTooLarge` | 변환은 에 제공된 사전 서명된 URL을 사용하여 업로드할 수 없습니다 `target`. 실제 변환 크기는 에서 메타데이터로 사용할 수 `repo:size` 있으며 클라이언트가 미리 서명된 URL의 정확한 수와 함께 이 변환을 재처리하는 데 사용할 수 있습니다. |
| `GenericError` | 다른 예기치 않은 오류가 있습니다. |
