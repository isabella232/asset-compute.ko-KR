---
title: '"[!DNL Asset Compute Service] HTTP API"'
description: '"[!DNL Asset Compute Service] 사용자 지정 애플리케이션을 만들기 위한 HTTP API"'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API의 사용은 개발 용도로 제한됩니다. API는 사용자 지정 애플리케이션을 개발할 때 컨텍스트로 제공됩니다. [!DNL Adobe Experience Manager] 로서의 [!DNL Cloud Service] 는 API를 사용하여 처리 정보를 사용자 지정 애플리케이션에 전달합니다. 자세한 내용은 [자산 마이크로서비스 및 처리 프로필 사용](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 는 과 함께 사용할 수만 있습니다. [!DNL Experience Manager] 로서의 [!DNL Cloud Service].

의 모든 클라이언트 [!DNL Asset Compute Service] HTTP API는 다음 높은 수준의 흐름을 따라야 합니다.

1. 클라이언트는 [!DNL Adobe Developer Console] ims 조직의 프로젝트. 이벤트 데이터 흐름을 구분하기 위해 각각의 개별 클라이언트(시스템 또는 환경)에는 고유한 별도의 프로젝트가 필요합니다.

1. 클라이언트는 를 사용하여 기술 계정에 대한 액세스 토큰을 생성합니다 [JWT(서비스 계정) 인증](https://www.adobe.io/authentication/auth-methods.html).

1. 클라이언트 호출 [`/register`](#register) 저널 URL을 한 번만 검색합니다.

1. 클라이언트 호출 [`/process`](#process-request) 표현물을 생성하려는 각 자산에 대해 입니다. 호출은 비동기적으로 수행됩니다.

1. 클라이언트는 저널을 정기적으로 폴링하여 [이벤트 수신](#asynchronous-events). 표현물이 성공적으로 처리되면 요청된 각 표현물에 대한 이벤트를 수신합니다(`rendition_created` 이벤트 유형) 또는 오류가 있는 경우(`rendition_failed` 이벤트 유형).

다음 [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) 모듈을 사용하면 Node.js 코드에서 API를 쉽게 사용할 수 있습니다.

## 인증 및 인증 {#authentication-and-authorization}

모든 API에는 액세스 토큰 인증이 필요합니다. 요청은 다음 헤더를 설정해야 합니다.

1. `Authorization` bearer 토큰이 있는 헤더로서, 기술 계정 토큰으로서 [JWT 교환](https://www.adobe.io/authentication/auth-methods.html) Adobe 개발자 콘솔 프로젝트에서 참조할 수 있습니다. 다음 [범위](#scopes) 아래에 설명되어 있습니다.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 헤더를 구성합니다.

1. `x-api-key` 사용 [!DNL Adobe Developers Console] 프로젝트.

### 범위 {#scopes}

액세스 토큰에 대해 다음 범위를 확인하십시오.

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

이를 위해서는 [!DNL Adobe Developer Console] 구독할 프로젝트 `Asset Compute`, `I/O Events`, 및 `I/O Management API` 서비스. 개별 범위의 분류는 다음과 같습니다.

* 기본
   * 범위: `openid,AdobeID`

* asset compute
   * meascope: `asset_compute_meta`
   * 범위: `asset_compute,read_organizations`

* [!DNL Adobe I/O] 이벤트
   * meascope: `event_receiver_api`
   * 범위: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 관리 API
   * meascope: `ent_adobeio_sdk`
   * 범위: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 등록 {#register}

의 각 클라이언트 [!DNL Asset Compute service] - 고유 [!DNL Adobe Developer Console] 서비스를 구독한 프로젝트 - 반드시 [등록](#register-request) 처리 요청을 수행하기 전에 등록 단계는 변환 처리에서 비동기 이벤트를 검색하는 데 필요한 고유한 이벤트 저널을 반환합니다.

라이프사이클이 끝나면 클라이언트는 다음을 수행할 수 있습니다 [등록 취소](#unregister-request).

### 요청 등록 {#register-request}

이 API 호출은 [!DNL Asset Compute] 클라이언트 및 은 이벤트 저널 URL을 제공합니다. 이 작업은 idempotent 작업이므로 각 클라이언트에 대해 한 번만 호출해야 합니다. 저널 URL을 검색하기 위해 다시 호출할 수 있습니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/register` |
| 헤더 `Authorization` | 모두 [권한 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 옵션, 은 여러 시스템에서 처리 요청의 고유한 종단 간 식별자에 대해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 비어 있어야 합니다. |

### 응답 등록 {#register-response}

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | 와 동일하거나 `X-Request-Id` 요청 헤더 또는 고유하게 생성된 헤더. 시스템 및/또는 지원 요청에 대한 요청을 식별하는 데 를 사용합니다. |
| 응답 본문 | JSON 개체 `journal`, `ok` 및/또는 `requestId` 필드. |

HTTP 상태 코드는 다음과 같습니다.

* **200년 성공**: 요청이 성공하면. 여기에는 다음 항목이 포함되어 있습니다 `journal` 를 통해 트리거되는 비동기 처리 결과에 대해 알림을 받는 URL `/process` (이벤트 유형 기준) `rendition_created` 성공 시 또는 `rendition_failed` 실패 시).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 권한 없음**: 요청이 유효하지 않은 경우 발생합니다 [인증](#authentication-and-authorization). 예로는 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.

* **403 금지**: 요청이 유효하지 않은 경우 발생합니다 [권한](#authentication-and-authorization). 예를 유효한 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 일부 필수 서비스에 가입되지 않았습니다.

* **429 너무 많은 요청**: 이 클라이언트가 시스템에 오버로드되거나 다른 방식으로 인해 발생합니다. 클라이언트는 를 사용하여 다시 시도해야 합니다. [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff). 몸이 비어 있습니다.
* **4xx 오류**: 다른 클라이언트 오류 및 등록에 실패한 경우 이 모든 오류에 대해 보장되지 않지만 일반적으로 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx 오류**: 다른 서버 측 오류가 발생하여 등록하지 못할 때 발생합니다. 이 모든 오류에 대해 보장되지 않지만 일반적으로 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 요청 등록 취소 {#unregister-request}

이 API 호출은 등록을 취소합니다 [!DNL Asset Compute] 클라이언트. 이후 더 이상 호출할 수 없습니다 `/process`. 등록되지 않은 클라이언트 또는 아직 등록되지 않은 클라이언트에 대한 API 호출을 사용하면 `404` 오류가 발생했습니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/unregister` |
| 헤더 `Authorization` | 모두 [권한 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 옵션, 은 여러 시스템에서 처리 요청의 고유한 종단 간 식별자에 대해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 비어 있음. |

### 응답 등록 취소 {#unregister-response}

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | 와 동일하거나 `X-Request-Id` 요청 헤더 또는 고유하게 생성된 헤더. 시스템 및/또는 지원 요청에 대한 요청을 식별하는 데 를 사용합니다. |
| 응답 본문 | JSON 개체 `ok` 및 `requestId` 필드. |

상태 코드는 다음과 같습니다.

* **200년 성공**: 등록 및 분개가 발견되어 제거될 때 발생합니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 권한 없음**: 요청이 유효하지 않은 경우 발생합니다 [인증](#authentication-and-authorization). 예로는 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.

* **403 금지**: 요청이 유효하지 않은 경우 발생합니다 [권한](#authentication-and-authorization). 예를 유효한 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 일부 필수 서비스에 가입되지 않았습니다.

* **404 찾을 수 없음**: 지정된 자격 증명에 대한 현재 등록이 없는 경우 발생합니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 너무 많은 요청**: 시스템이 오버로드될 때 발생합니다. 클라이언트는 를 사용하여 다시 시도해야 합니다. [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff). 몸이 비어 있습니다.

* **4xx 오류**: 다른 클라이언트 오류가 발생하여 등록을 취소하지 못한 경우 발생합니다. 이 모든 오류에 대해 보장되지 않지만 일반적으로 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx 오류**: 다른 서버 측 오류가 발생하여 등록하지 못할 때 발생합니다. 이 모든 오류에 대해 보장되지 않지만 일반적으로 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 프로세스 요청 {#process-request}

다음 `process` 작업은 요청의 지침에 따라 소스 자산을 여러 변환으로 변환하는 작업을 제출합니다. 성공 완료에 대한 알림(이벤트 유형) `rendition_created`) 또는 모든 오류(이벤트 유형) `rendition_failed`)를 사용하여 검색해야 하는 이벤트 분개로 전송됩니다 [/register](#register) 한 번 전에 `/process` 요청. 잘못된 형식의 요청이 즉시 실패하고 400 오류 코드가 표시됩니다.

바이너리는 Amazon AWS S3 사전 서명된 URL 또는 Azure Blob Storage SAS URL과 같은 URL을 사용하여 참조할 수 있습니다. `source` 자산 (`GET` URL) 및 표현물 쓰기(`PUT` URL). 클라이언트는 이러한 사전 서명된 URL을 생성합니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/process` |
| MIME 유형 | `application/json` |
| 헤더 `Authorization` | 모두 [권한 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 옵션, 은 여러 시스템에서 처리 요청의 고유한 종단 간 식별자에 대해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 아래 설명된 대로 프로세스 요청 JSON 형식이어야 합니다. 처리할 자산 및 생성할 변환에 대한 지침을 제공합니다. |

### 프로세스 요청 JSON {#process-request-json}

의 요청 본문 `/process` 는 이 고급 스키마를 사용하는 JSON 개체입니다.

```json
{
    "source": "",
    "renditions" : []
}
```

사용 가능한 필드는 다음과 같습니다.

| 이름 | 유형 | 설명 | 예 |
|--------------|----------|-------------|---------|
| `source` | `string` | 처리할 소스 자산의 URL입니다. 요청된 변환 형식을 기반으로 하는 옵션(예: `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | 처리할 소스 자산을 설명합니다. 다음 설명 참조 [소스 개체 필드](#source-object-fields) 아래의 제품에서 사용할 수 있습니다. 요청된 변환 형식을 기반으로 하는 옵션(예: `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 소스 파일에서 생성할 표현물. 각 변환 개체는 [변환 명령](#rendition-instructions). 필수. | `[{ "target": "https://....", "fmt": "png" }]` |

다음 `source` 다음 중 하나일 수 있습니다. `<string>` URL로 표시되거나 `<object>` 추가 필드 사용. 다음 변형은 유사합니다.

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### 소스 개체 필드 {#source-object-fields}

| 이름 | 유형 | 설명 | 예 |
|-----------|----------|-------------|---------|
| `url` | `string` | 처리할 소스 자산의 URL입니다. 필수. | `"http://example.com/image.jpg"` |
| `name` | `string` | 소스 자산 파일 이름입니다. MIME 유형을 검색할 수 없는 경우 이름의 파일 확장명을 사용할 수 있습니다. 의 URL 경로나 파일 이름에 있는 파일 이름보다 우선합니다. `content-disposition` 이진 리소스의 헤더입니다. 기본값은 &quot;file&quot;입니다. | `"image.jpg"` |
| `size` | `number` | 소스 자산 파일 크기(바이트)입니다. 우선함 `content-length` 이진 리소스의 헤더입니다. | `10234` |
| `mimetype` | `string` | 소스 자산 파일 MIME 유형입니다. 보다 우선합니다. `content-type` 이진 리소스의 헤더입니다. | `"image/jpeg"` |

### 완료 `process` 요청 예 {#complete-process-request-example}

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

다음 `/process` 요청은 기본 요청 유효성 검사에 따른 성공 또는 오류로 즉시 반환됩니다. 실제 자산 처리가 비동기적으로 수행됩니다.

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | 와 동일하거나 `X-Request-Id` 요청 헤더 또는 고유하게 생성된 헤더. 시스템 및/또는 지원 요청에 대한 요청을 식별하는 데 를 사용합니다. |
| 응답 본문 | JSON 개체 `ok` 및 `requestId` 필드. |

상태 코드:

* **200년 성공**: 요청이 성공적으로 제출된 경우. 응답 JSON에는 다음이 포함됩니다 `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 잘못된 요청**: 요청 JSON에 필수 필드가 누락되는 등 요청이 잘못 구성된 경우. 응답 JSON에는 다음이 포함됩니다 `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 권한 없음**: 요청에 유효하지 않은 경우 [인증](#authentication-and-authorization). 예로는 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.
* **403 금지**: 요청에 유효하지 않은 경우 [권한](#authentication-and-authorization). 예를 유효한 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 일부 필수 서비스에 가입되지 않았습니다.
* **429 너무 많은 요청**: 시스템에 이 클라이언트 또는 일반적으로 오버로드되는 경우입니다. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff). 몸이 비어 있습니다.
* **4xx 오류**: 다른 클라이언트 오류가 발생한 경우. 이 모든 오류에 대해 보장되지 않지만 일반적으로 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx 오류**: 다른 서버 측 오류가 발생한 경우. 이 모든 오류에 대해 보장되지 않지만 일반적으로 이와 같은 JSON 응답이 반환됩니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

대부분의 클라이언트는 를 사용하여 정확히 동일한 요청을 재시도하는 경향이 있습니다. [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff) 오류가 발생한 경우 *제외* 401 또는 403과 같은 구성 문제 또는 400과 같은 잘못된 요청. 429 응답을 통한 정기적인 비율 제한 외에도, 임시 서비스 중단 또는 제한으로 인해 5xx 오류가 발생할 수 있습니다. 그런 다음 일정 시간 후에 다시 시도하는 것이 좋습니다.

모든 JSON 응답(있는 경우)에 `requestId` 와 동일한 값 `X-Request-Id` 헤더. 항상 있으므로 헤더에서 읽는 것이 좋습니다. 다음 `requestId` 는 또한 처리 요청과 관련된 모든 이벤트에서 `requestId`. 클라이언트는 이 문자열의 형식을 가정하지 않아야 하며 불투명 문자열 식별자입니다.

## 사후 처리에 옵트인 {#opt-in-to-post-processing}

다음 [asset compute SDK](https://github.com/adobe/asset-compute-sdk) 에서는 기본 이미지 사후 처리 옵션 세트를 지원합니다. 사용자 지정 작업자는 필드를 설정하여 사후 처리에 명시적으로 옵트인할 수 있습니다 `postProcess` 변환 객체에서 `true`.

지원되는 사용 사례는 다음과 같습니다.

* 자르기.w, crop.h, crop.x 및 crop.y로 제한이 정의된 사각형으로 변환을 자릅니다. 에 의해 정의됩니다 `instructions.crop` 변환 개체
* 폭, 높이 또는 둘 다 사용하여 이미지 크기를 조정할 수 있습니다. 에 의해 정의됩니다 `instructions.width` 및 `instructions.height` 변환 개체 너비나 높이만 사용하여 크기를 조정하려면 값을 하나만 설정하십시오. Compute Service는 종횡비를 유지합니다.
* JPEG 이미지의 품질을 설정합니다. 에 의해 정의됩니다 `instructions.quality` 변환 개체 가장 좋은 품질은 `100` 및 값이 작을수록 품질이 저하됩니다.
* 인터레이스 이미지를 만듭니다. 에 의해 정의됩니다 `instructions.interlace` 변환 개체
* 픽셀에 적용된 비율을 조정하여 데스크톱 게시 목적으로 렌더링된 크기를 조정하도록 DPI를 설정합니다. 에 의해 정의됩니다 `instructions.dpi` 변환 개체에서 dpi 해상도를 변경합니다. 그러나 다른 해상도에서 동일한 크기가 되도록 이미지 크기를 조정하려면 `convertToDpi` 지침
* 렌더링된 너비나 높이가 지정된 대상 해상도(DPI)에서 원본과 동일하게 유지되도록 이미지 크기를 조정합니다. 에 의해 정의됩니다 `instructions.convertToDpi` 변환 개체

## 에셋에 워터마크 추가 {#add-watermark}

다음 [asset compute SDK](https://github.com/adobe/asset-compute-sdk) 에서는 PNG, JPEG, TIFF 및 GIF 이미지 파일에 워터마크 추가를 지원합니다. 워터마크는 `watermark` 개체의 이름을 지정합니다.

워터마크는 변환 후 처리 중에 수행됩니다. 자산에 워터마크 지정하려면 사용자 정의 작업자입니다 [사후 처리로 옵트](#opt-in-to-post-processing) 필드를 설정하여 `postProcess` 변환 객체에서 `true`. 작업자가 옵트인하지 않으면 요청의 표현물 개체에 워터마크 개체가 설정되어 있어도 워터마킹이 적용되지 않습니다.

## 표현물 지침 {#rendition-instructions}

다음은 `renditions` 의 배열 [/process](#process-request).

### 공통 필드 {#common-fields}

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 표현물 대상 형식으로서, `text` 텍스트 추출 및 `xmp` XMP 메타데이터를 xml로 추출하는 데 사용됩니다. 자세한 내용은 [지원되는 형식](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | 의 URL [사용자 지정 애플리케이션](develop-custom-application.md). 이어야 합니다. `https://` URL. 이 필드가 있으면 사용자 지정 애플리케이션에서 렌디션을 만듭니다. 그런 다음 사용자 지정 애플리케이션에서 다른 모든 세트 변환 필드가 사용됩니다. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | HTTP PUT을 사용하여 생성된 변환을 업로드해야 하는 URL입니다. | `http://w.com/img.jpg` |
| `target` | `object` | 생성된 표현물에 대한 여러 부분으로 미리 서명된 URL 업로드 정보입니다. 다음 용도 [AEM/Oak 다이렉트 이진 업로드](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) 이 [다중 부분 업로드 동작](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>필드:<ul><li>`urls`: 사전 서명된 각 부분 URL에 대해 하나씩, 문자열 배열</li><li>`minPartSize`: 한 부분에 사용할 최소 크기 = url</li><li>`maxPartSize`: 한 부분에 사용할 최대 크기 = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 클라이언트에서 제어하고 변환 이벤트로 전달된 선택적 예약 공간. 클라이언트가 변환 이벤트를 식별하는 사용자 지정 정보를 추가할 수 있습니다. 클라이언트가 언제든지 이 설정을 변경할 수 있으므로 사용자 지정 애플리케이션에서 수정하거나 의존하지 않아야 합니다. | `{ ... }` |

### 표현물 특정 필드 {#rendition-specific-fields}

현재 지원되는 파일 형식 목록에 대해서는 [지원되는 파일 형식](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 고급 사용자 지정 필드를 [사용자 지정 애플리케이션](develop-custom-application.md) 이해 |  |
| `embedBinaryLimit` | `number` 바이트 | 이 값이 설정되고 표현물의 파일 크기가 이 값보다 작은 경우, 표현물 생성이 완료되면 전송되는 이벤트에 표현물이 포함됩니다. 포함할 수 있는 최대 크기는 32KB(32 x 1024바이트)입니다. 표현물의 크기가 `embedBinaryLimit` 제한 사항은 클라우드 스토리지의 위치에 배치되며 이벤트에 포함되지 않습니다. | `3276` |
| `width` | `number` | 너비(픽셀 단위)입니다. 이미지 표현물 전용. | `200` |
| `height` | `number` | 높이(픽셀). 이미지 표현물 전용. | `200` |
|  |  | 종횡비는 다음과 같은 경우 항상 유지됩니다. <ul> <li> 둘 다 `width` 및 `height` 를 지정한 다음, 이미지가 종횡비를 유지하면서 크기에 맞게 조정됩니다 </li><li> 전용 `width` 또는 전용 `height` 이 지정되면 결과 이미지는 종횡비를 유지하면서 해당 차원을 사용합니다</li><li> 둘 중 어느 것도 아니면 `width` 아니요 `height` 이 지정되면 원본 이미지 픽셀 크기가 사용됩니다. 소스 유형에 따라 다릅니다. PDF 파일과 같은 일부 형식의 경우 기본 크기가 사용됩니다. 최대 크기 제한이 있을 수 있습니다.</li></ul> |  |
| `quality` | `number` | 범위 내에서 JPEG 품질 지정 `1` to `100`. 이미지 표현물에 대해서만 적용 가능합니다. | `90` |
| `xmp` | `string` | XMP 메타데이터 원본에 쓰기 기능으로만 사용되며, 지정된 표현물에 다시 쓰기 위해 base64 인코딩 XMP입니다. |  |
| `interlace` | `bool` | 로 설정하여 인터레이스 PNG 또는 GIF 또는 점진적 JPEG 만들기 `true`. 다른 파일 형식에는 영향을 주지 않습니다. |  |
| `jpegSize` | `number` | JPEG 파일의 대략적인 크기(바이트)입니다. 다른 항목보다 우선합니다 `quality` 설정 다른 형식에는 영향을 주지 않습니다. |  |
| `dpi` | `number` 또는 `object` | x 및 y DPI를 설정합니다. 간단히 하기 위해 x와 y 모두에 사용되는 단일 숫자로 설정할 수도 있습니다. 이미지 자체에는 영향을 주지 않습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 또는 `object` | x 및 y DPI는 물리적 크기를 유지하면서 다시 샘플링합니다. 간단히 하기 위해 x와 y 모두에 사용되는 단일 숫자로 설정할 수도 있습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | ZIP 보관 위치에 포함할 파일 목록(`fmt=zip`). 각 항목은 URL 문자열이거나, 필드가 있는 개체일 수 있습니다.<ul><li>`url`: 파일을 다운로드할 URL</li><li>`path`: ZIP에서 이 경로 아래에 파일을 저장합니다</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP 아카이브에 대한 중복 처리(`fmt=zip`). 기본적으로 ZIP의 동일한 경로에 저장된 여러 파일은 오류를 생성합니다. 설정 `duplicate` to `ignore` 따라서 저장할 첫 번째 자산과 나머지 자산만 무시됩니다. | `ignore` |
| `watermark` | `object` | 에 대한 지침을 포함합니다 [워터마크](#watermark-specific-fields). |  |

### 워터마크 특정 필드 {#watermark-specific-fields}

PNG 형식은 워터마크로 사용됩니다.

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 워터마크의 크기(사이) `0.0` 및 `1.0`. `1.0` 즉, 워터마크의 원래 배율이 (1:1) 이고 값이 낮을수록 워터마크 크기가 줄어듭니다. | 값 `0.5` 원래 크기의 반을 의미합니다. |
| `image` | `url` | 워터마크에 사용할 PNG 파일의 URL입니다. |  |

## 비동기 이벤트 {#asynchronous-events}

변환 처리가 완료되거나 오류가 발생하면 이벤트가 [[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). 클라이언트는 를 통해 제공된 저널 URL을 수신해야 합니다 [/register](#register). 분개 응답에는 `event` 각 이벤트에 대해 하나의 개체로 구성된 배열 `event` 필드에는 실제 이벤트 페이로드가 포함되어 있습니다.

다음 [!DNL Adobe I/O] 의 모든 이벤트에 대한 이벤트 유형 [!DNL Asset Compute Service] is `asset_compute`. 저널은 이 이벤트 유형에만 자동으로 구독되며, [!DNL Adobe I/O] 이벤트 유형. 서비스별 이벤트 유형은 `type` 이벤트의 속성입니다.

### 이벤트 유형 {#event-types}

| Event | 설명 |
|---------------------|-------------|
| `rendition_created` | 성공적으로 처리되고 업로드된 각 표현물에 대해 전송됩니다. |
| `rendition_failed` | 처리하거나 업로드하지 못한 각 표현물에 대해 전송됩니다. |

### 이벤트 속성 {#event-attributes}

| 특성 | 유형 | 이벤트 | 설명 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 간소화된 확장으로 이벤트를 보낸 타임스탬프 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 형식으로, JavaScript에서 정의한 대로 [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | 에 대한 원래 요청의 요청 ID `/process`와 같음 `X-Request-Id` 헤더. |
| `source` | `object` | `*` | 다음 `source` 의 `/process` 요청. |
| `userData` | `object` | `*` | 다음 `userData` 변환 중 `/process` 설정된 경우 요청합니다. |
| `rendition` | `object` | `rendition_*` | 전달된 해당 변환 개체 `/process`. |
| `metadata` | `object` | `rendition_created` | 다음 [메타데이터](#metadata) 표현물의 속성입니다. |
| `errorReason` | `string` | `rendition_failed` | 변환 실패 [이유](#error-reasons) 있을 경우. |
| `errorMessage` | `string` | `rendition_failed` | 표현물 오류에 대한 자세한 정보를 제공하는 텍스트입니다. |

### 메타데이터 {#metadata}

| 속성 | 설명 |
|--------|-------------|
| `repo:size` | 표현물의 크기(바이트)입니다. |
| `repo:sha1` | 표현물의 sha1 다이제스트입니다. |
| `dc:format` | 표현물의 MIME 유형입니다. |
| `repo:encoding` | 텍스트 기반 형식인 경우 표현물의 문자 인코딩. |
| `tiff:ImageWidth` | 표현물의 픽셀 단위 폭입니다. 이미지 표현물에 대해서만 표시됩니다. |
| `tiff:ImageLength` | 표현물의 길이(픽셀 단위)입니다. 이미지 표현물에 대해서만 표시됩니다. |

### 오류 이유 {#error-reasons}

| 이유 | 설명 |
|---------|-------------|
| `RenditionFormatUnsupported` | 요청한 변환 형식은 지정된 소스에 대해 지원되지 않습니다. |
| `SourceUnsupported` | 형식이 지원되지만 특정 소스는 지원되지 않습니다. |
| `SourceCorrupt` | 원본 데이터가 손상되었습니다. 빈 파일을 포함합니다. |
| `RenditionTooLarge` | 에 제공된 사전 서명된 URL을 사용하여 변환을 업로드할 수 없습니다 `target`. 실제 변환 크기는에서 메타데이터로 사용할 수 있습니다. `repo:size` 및 는 클라이언트가 적절한 수의 사전 서명된 URL로 이 렌디션을 다시 처리하는 데 사용할 수 있습니다. |
| `GenericError` | 다른 예기치 않은 오류가 있습니다. |
