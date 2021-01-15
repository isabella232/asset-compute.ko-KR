---
title: '[!DNL Asset Compute Service] HTTP API.'
description: '[!DNL Asset Compute Service] HTTP API를 사용하여 사용자 정의 응용 프로그램을 만듭니다.'
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---


# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API의 사용은 개발 목적으로 제한됩니다. API는 사용자 지정 애플리케이션을 개발할 때 컨텍스트로 제공됩니다. [!DNL Adobe Experience Manager] 를  [!DNL Cloud Service] 사용하여 처리 정보를 사용자 지정 애플리케이션에 전달합니다. 자세한 내용은 [자산 마이크로서비스 및 처리 프로필 사용](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)을 참조하십시오.

>[!NOTE]
>
>[!DNL Asset Compute Service] 은 를 a로 사용할  [!DNL Experience Manager] 때만 사용할 수  [!DNL Cloud Service]있습니다.

[!DNL Asset Compute Service] HTTP API의 모든 클라이언트는 다음 높은 수준의 흐름을 따라야 합니다.

1. 클라이언트는 IMS 조직에서 [!DNL Adobe Developer Console] 프로젝트로 프로비저닝됩니다. 이벤트 데이터 흐름을 구분하기 위해 각각의 개별 클라이언트(시스템 또는 환경)에는 자체 개별 프로젝트가 필요합니다.

1. 클라이언트는 [JWT(서비스 계정) 인증](https://www.adobe.io/authentication/auth-methods.html)을(를) 사용하여 기술 계정에 대한 액세스 토큰을 생성합니다.

1. 클라이언트는 저널 URL을 검색하려면 [`/register`](#register)만 호출합니다.

1. 클라이언트가 변환을 생성할 각 자산에 대해 [`/process`](#process-request)을 호출합니다. 호출은 비동기 상태입니다.

1. 클라이언트는 저널을 정기적으로 [수신 이벤트](#asynchronous-events)에 폴링합니다. 변환이 성공적으로 처리되거나(`rendition_created` 이벤트 유형) 오류가 있는 경우(`rendition_failed` 이벤트 유형) 요청된 각 변환에 대한 이벤트를 수신합니다.

[@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) 모듈을 사용하면 Node.js 코드에서 API를 쉽게 사용할 수 있습니다.

## 인증 및 인증 {#authentication-and-authorization}

모든 API는 액세스 토큰 인증을 필요로 합니다. 요청은 다음 헤더를 설정해야 합니다.

1. `Authorization` 헤더(기술 계정 토큰인 베어러 토큰)가 Adobe 개발자 콘솔 프로젝트에서  [JWT ](https://www.adobe.io/authentication/auth-methods.html) 교환을 통해 전송됩니다. [범위](#scopes)는 아래에 문서화되어 있습니다.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 헤더를 추가합니다.

1. `x-api-key` 를 프로젝트에 있는 클라이언트 ID로  [!DNL Adobe Developers Console] 설정합니다.

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

이러한 경우 [!DNL Adobe Developer Console] 프로젝트를 `Asset Compute`, `I/O Events` 및 `I/O Management API` 서비스에 가입해야 합니다. 개별 범위의 분류는 다음과 같습니다.

* 기본
   * 범위:`openid,AdobeID`

* asset compute
   * 메타스코프:`asset_compute_meta`
   * 범위:`asset_compute,read_organizations`

* [!DNL Adobe I/O] 이벤트
   * 메타스코프:`event_receiver_api`
   * 범위:`event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 관리 API
   * 메타스코프:`ent_adobeio_sdk`
   * 범위:`adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 등록 {#register}

서비스에 가입한 고유 [!DNL Adobe Developer Console] 프로젝트의 각 클라이언트 - 처리 요청을 만들기 전에 [등록](#register-request)해야 합니다. [!DNL Asset Compute service] 등록 단계에서는 변환 처리에서 비동기 이벤트를 가져오는 데 필요한 고유한 이벤트 저널을 반환합니다.

라이프사이클이 끝나면 클라이언트는 [등록 취소](#unregister-request)할 수 있습니다.

### 등록 요청 {#register-request}

이 API 호출은 [!DNL Asset Compute] 클라이언트를 설정하고 이벤트 저널 URL을 제공합니다. 이것은 idempose 작업이므로 각 클라이언트에 대해 한 번만 호출하면 됩니다. 저널 URL을 검색하기 위해 다시 호출할 수 있습니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/register` |
| 헤더 `Authorization` | 모든 [인증 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 선택 사항으로, 여러 시스템에서 처리 요청의 고유 엔드 투 엔드 식별자에 대해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 비어 있어야 합니다. |

### 응답 등록 {#register-response}

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | `X-Request-Id` 요청 헤더와 동일하거나 고유하게 생성된 헤더와 같습니다. 시스템 및/또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | `journal`, `ok` 및/또는 `requestId` 필드가 있는 JSON 개체. |

HTTP 상태 코드는 다음과 같습니다.

* **200 성공**:요청이 성공하면 이 파일에는 `/process`을 통해 트리거된 비동기 처리 결과에 대한 알림을 받는 `journal` URL이 포함되어 있습니다(성공 시 이벤트 유형 `rendition_created` 또는 실패 시 `rendition_failed`).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 권한 없음**:요청에 유효한 인증이 없을 때  [발생합니다](#authentication-and-authorization). 예에는 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.

* **403 금지**:요청에 유효한 인증이 없는 경우에  [발생합니다](#authentication-and-authorization). 예를 들어 유효한 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 필요한 일부 서비스에 가입되지 않았습니다.

* **429 요청이 너무 많습니다**.이 클라이언트에서 시스템에 과부하가 걸렸을 때 발생합니다. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)로 다시 시도해야 합니다. 시체가 비어 있다.
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

이 API 호출은 [!DNL Asset Compute] 클라이언트의 등록을 취소합니다. 이후에는 더 이상 `/process`을(를) 호출할 수 없습니다. 등록되지 않은 클라이언트 또는 아직 등록되지 않은 클라이언트에 대한 API 호출을 사용하면 `404` 오류가 반환됩니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/unregister` |
| 헤더 `Authorization` | 모든 [인증 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 선택 사항으로, 여러 시스템에서 처리 요청의 고유 엔드 투 엔드 식별자에 대해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 비어 있음. |

### 응답 등록 취소 {#unregister-response}

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | `X-Request-Id` 요청 헤더와 동일하거나 고유하게 생성된 헤더와 같습니다. 시스템 및/또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | `ok` 및 `requestId` 필드가 있는 JSON 개체. |

상태 코드는 다음과 같습니다.

* **200 성공**:등록 및 분기가 발견되어 제거될 때 발생합니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 권한 없음**:요청에 유효한 인증이 없을 때  [발생합니다](#authentication-and-authorization). 예에는 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.

* **403 금지**:요청에 유효한 인증이 없는 경우에  [발생합니다](#authentication-and-authorization). 예를 들어 유효한 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 필요한 일부 서비스에 가입되지 않았습니다.

* **404 찾을 수 없음**:주어진 자격 증명에 대한 현재 등록이 없을 때 발생합니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 요청이 너무 많습니다**.시스템이 오버로드될 때 발생합니다. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)로 다시 시도해야 합니다. 시체가 비어 있다.

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

`process` 작업은 요청의 지침에 따라 소스 자산을 여러 변환으로 변환하는 작업을 제출합니다. 완료 성공(이벤트 유형 `rendition_created`) 또는 오류(이벤트 유형 `rendition_failed`)에 대한 알림은 `/process` 요청을 한 번 수행하기 전에 [/register](#register)을 사용하여 검색해야 하는 이벤트 저널에 전송됩니다. 잘못된 형식의 요청은 400 오류 코드로 인해 즉시 실패하게 됩니다.

바이너리는 `source` 자산(`GET` URL)을 읽고 변환(`PUT` URL)을 쓰기 위해 Amazon AWS S3 사전 서명된 URL 또는 Azure Blob 저장소 SAS URL과 같은 URL을 사용하여 참조됩니다. 클라이언트는 이러한 사전 서명된 URL을 생성합니다.

| 매개 변수 | 값 |
|--------------------------|------------------------------------------------------|
| 메서드 | `POST` |
| 경로 | `/process` |
| MIME 유형 | `application/json` |
| 헤더 `Authorization` | 모든 [인증 관련 헤더](#authentication-and-authorization). |
| 헤더 `x-request-id` | 선택 사항으로, 여러 시스템에서 처리 요청의 고유 엔드 투 엔드 식별자에 대해 클라이언트가 설정할 수 있습니다. |
| 요청 본문 | 아래의 설명에 따라 프로세스 요청 JSON 형식이어야 합니다. 처리할 자산 및 생성할 표현물에 대한 지침을 제공합니다. |

### 프로세스 요청 JSON {#process-request-json}

`/process`의 요청 본문은 다음과 같은 높은 수준의 스키마를 사용하는 JSON 개체입니다.

```json
{
    "source": "",
    "renditions" : []
}
```

사용 가능한 필드는 다음과 같습니다.

| 이름 | 유형 | 설명 | 예 |
|--------------|----------|-------------|---------|
| `source` | `string` | 처리할 소스 자산의 URL. 요청된 변환 형식(예:`fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | 처리할 소스 자산을 설명합니다. 아래의 [소스 개체 필드](#source-object-fields)에 대한 설명을 참조하십시오. 요청된 변환 형식(예:`fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 소스 파일에서 생성할 변환입니다. 각 변환 개체는 [변환 명령](#rendition-instructions)을 지원합니다. 필수. | `[{ "target": "https://....", "fmt": "png" }]` |

`source`은 URL로 표시되는 `<string>`이거나 추가 필드가 있는 `<object>`일 수 있습니다. 다음과 같은 변형이 유사합니다.

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
| `url` | `string` | 처리할 소스 자산의 URL. 필수. | `"http://example.com/image.jpg"` |
| `name` | `string` | 소스 에셋 파일 이름. MIME 형식을 찾을 수 없는 경우 이름의 파일 확장자를 사용할 수 있습니다. 이진 리소스의 `content-disposition` 헤더에 있는 URL 경로의 파일 이름 또는 파일 이름보다 우선합니다. 기본값은 &quot;file&quot;입니다. | `"image.jpg"` |
| `size` | `number` | 소스 에셋 파일 크기(바이트)입니다. 이진 리소스의 `content-length` 헤더보다 우선합니다. | `10234` |
| `mimetype` | `string` | 원본 자산 파일 MIME 유형입니다. 이진 리소스의 `content-type` 헤더보다 우선합니다. | `"image/jpeg"` |

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

`/process` 요청은 즉시 성공 또는 기본 요청 유효성 검사에 따른 실패로 반환됩니다. 실제 자산 처리는 비동기적으로 발생합니다.

| 매개 변수 | 값 |
|-----------------------|------------------------------------------------------|
| MIME 유형 | `application/json` |
| 헤더 `X-Request-Id` | `X-Request-Id` 요청 헤더와 동일하거나 고유하게 생성된 헤더와 같습니다. 시스템 및/또는 지원 요청에서 요청을 식별하는 데 사용합니다. |
| 응답 본문 | `ok` 및 `requestId` 필드가 있는 JSON 개체. |

상태 코드:

* **200 성공**:요청이 성공적으로 제출된 경우. 응답 JSON에는 `"ok": true`이 포함되어 있습니다.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 잘못된 요청**:요청 JSON에 필수 필드가 누락되는 등 요청 형식이 잘못된 경우 응답 JSON에는 `"ok": false`이 포함되어 있습니다.

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 권한 없음**:요청에 유효한 인증이 없는  [경우입니다](#authentication-and-authorization). 예에는 잘못된 액세스 토큰 또는 잘못된 API 키가 있습니다.
* **403 금지**:요청에 유효한 인증이 없는  [경우입니다](#authentication-and-authorization). 예를 들어 유효한 액세스 토큰일 수 있지만 Adobe 개발자 콘솔 프로젝트(기술 계정)가 필요한 일부 서비스에 가입되지 않았습니다.
* **429 요청이 너무 많습니다**.시스템에 이 클라이언트 또는 일반적으로 과부하가 발생하는 경우. 클라이언트는 [지수 백오프](https://en.wikipedia.org/wiki/Exponential_backoff)로 다시 시도할 수 있습니다. 시체가 비어 있다.
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

대부분의 클라이언트는 *오류*&#x200B;에 대해 [지수 백업](https://en.wikipedia.org/wiki/Exponential_backoff), 401 또는 403과 같은 구성 문제 또는 400과 같은 잘못된 요청을 제외하고 에 대해 정확히 동일한 요청을 재시도하는 경향이 있습니다. 429개의 응답을 통해 이루어지는 정기적인 요금 제한 외에도, 일시적인 서비스 중단 또는 제한으로 인해 5xx개의 오류가 발생할 수 있습니다. 그런 다음 일정 기간 후에 다시 시도하는 것이 좋습니다.

모든 JSON 응답(있는 경우)에는 `X-Request-Id` 헤더와 동일한 값인 `requestId`이 포함됩니다. 헤더는 항상 있으므로 헤더에서 읽는 것이 좋습니다. 또한 `requestId`은(는) 처리 요청과 관련된 모든 이벤트에서 `requestId` 반환됩니다. 클라이언트는 이 문자열의 형식을 추측해서는 안 됩니다. 이것은 불투명한 문자열 식별자입니다.

## 사후 처리 옵트인 {#opt-in-to-post-processing}

[Asset compute SDK](https://github.com/adobe/asset-compute-sdk)는 일련의 기본 이미지 사후 처리 옵션을 지원합니다. 사용자 지정 작업자는 변환 개체의 필드 `postProcess`을(를) `true`로 설정하여 명시적으로 후처리 수신을 선택할 수 있습니다.

지원되는 사용 사례는 다음과 같습니다.

* crop.w, crop.h, crop.x 및 crop.y에 의해 제한이 정의된 사각형으로 변환을 자릅니다.변환 개체에서 `instructions.crop`에 의해 정의됩니다.
* 폭, 높이 또는 두 가지 모두를 사용하여 이미지 크기를 조정합니다. 변환 객체에서 `instructions.width` 및 `instructions.height`에 의해 정의됩니다. 폭 또는 높이만 사용하여 크기를 조정하려면 하나의 값만 설정합니다. Compute Service는 종횡비를 유지합니다.
* JPEG 이미지의 품질을 설정합니다. 변환 개체에서 `instructions.quality`에 의해 정의됩니다. 최상의 품질은 `100`으로 표시되고 값이 작을수록 품질이 저하됩니다.
* 인터레이스된 이미지를 만듭니다. 변환 개체에서 `instructions.interlace`에 의해 정의됩니다.
* 픽셀에 적용된 비율을 조정하여 데스크탑 퍼블리싱을 위해 렌더링된 크기를 조정하도록 DPI를 설정합니다. dpi 해상도를 변경하기 위해 변환 개체에서 `instructions.dpi`에 의해 정의됩니다. 그러나 다른 해상도에서 동일한 크기로 이미지 크기를 조정하려면 `convertToDpi` 지침을 사용하십시오.
* 렌더링된 폭 또는 높이가 지정된 대상 해상도(DPI)에서 원본과 동일하게 유지되도록 이미지 크기를 조정합니다. 변환 개체에서 `instructions.convertToDpi`에 의해 정의됩니다.

## 워터마크 자산 {#add-watermark}

[Asset compute SDK](https://github.com/adobe/asset-compute-sdk)에서는 PNG, JPEG, TIFF 및 GIF 이미지 파일에 워터마크를 추가할 수 있습니다. 변환의 `watermark` 개체에 있는 변환 지침에 따라 워터마크가 추가됩니다.

워터마크는 변환 사후 처리 중에 수행됩니다. 자산을 워터마킹하려면 사용자 지정 작업자 [가 변환 개체의 필드 `postProcess`를 `true`로 설정하여 사후 처리](#opt-in-to-post-processing)에 들어갑니다. 워커가 옵트인하지 않으면 워터마크 객체가 요청의 변환 개체에 설정되어 있어도 워터마킹이 적용되지 않습니다.

## 변환 지침 {#rendition-instructions}

다음은 [/process](#process-request)의 `renditions` 배열에 사용할 수 있는 옵션입니다.

### 공통 필드 {#common-fields}

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 변환 대상 형식은 텍스트 추출의 경우 `text`, XMP 메타데이터를 xml로 추출하는 경우 `xmp`일 수도 있습니다. [지원되는 형식](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)을 참조하십시오. | `png` |
| `worker` | `string` | [사용자 지정 응용 프로그램](develop-custom-application.md)의 URL입니다. `https://` URL이어야 합니다. 이 필드가 있으면 사용자 정의 응용 프로그램에 의해 변환이 만들어집니다. 그런 다음 다른 모든 세트 변환 필드가 사용자 정의 응용 프로그램에서 사용됩니다. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | HTTP PUT을 사용하여 생성된 변환을 업로드해야 하는 URL입니다. | `http://w.com/img.jpg` |
| `target` | `object` | 생성된 변환에 대한 여러 부분으로 미리 서명된 URL 업로드 정보. 이것은 [AEM/Oak 직접 이진 업로드](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html)에 대한 것으로, 이 [다중 부분 업로드 동작](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)입니다.<br>필드:<ul><li>`urls`:사전 서명된 각 부분 URL에 대해 하나씩 문자열 배열</li><li>`minPartSize`:한 부분에 사용할 최소 크기 = url</li><li>`maxPartSize`:한 부분에 사용할 최대 크기 = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 옵션으로 제공되는 예약 공간은 클라이언트에 의해 제어되고 변환 이벤트로 전달됩니다. 클라이언트가 변환 이벤트를 식별하는 사용자 정의 정보를 추가할 수 있습니다. 클라이언트는 언제든지 이를 변경할 수 있으므로 사용자 정의 애플리케이션을 수정하거나 의존하지 않아야 합니다. | `{ ... }` |

### 변환 특정 필드 {#rendition-specific-fields}

현재 지원되는 파일 형식 목록은 [지원되는 파일 형식](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)을 참조하십시오.

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `*` | `*` | [사용자 지정 응용 프로그램](develop-custom-application.md)이(가) 이해하는 고급 사용자 지정 필드를 추가할 수 있습니다. |  |
| `embedBinaryLimit` | `number` 바이트 수 | 이 값이 설정되고 변환의 파일 크기가 이 값보다 작은 경우, 변환 생성이 완료되면 전송되는 이벤트에 변환이 포함됩니다. 임베드할 수 있는 최대 크기는 32KB(32 x 1024바이트)입니다. 표현물의 크기가 `embedBinaryLimit` 제한보다 큰 경우 클라우드 스토리지의 위치에 배치되고 이벤트에 포함되지 않습니다. | `3276` |
| `width` | `number` | 너비(픽셀 단위). 이미지 변환에만 해당됩니다. | `200` |
| `height` | `number` | 높이(픽셀 단위). 이미지 변환에만 해당됩니다. | `200` |
|  |  | 종횡비는 다음과 같은 경우에 항상 유지됩니다. <ul> <li> `width` 및 `height` 모두 지정되면 종횡비를 유지하면서 이미지가 크기에 맞게 됩니다. </li><li> `width`만 지정되거나 `height`만 지정되면 결과 이미지는 종횡비를 유지하면서 해당 차원을 사용합니다</li><li> `width` 또는 `height`을 지정하지 않으면 원래 이미지 픽셀 크기가 사용됩니다. 소스 유형에 따라 다릅니다. PDF 파일과 같은 일부 형식의 경우 기본 크기가 사용됩니다. 최대 크기 제한이 있을 수 있습니다.</li></ul> |  |
| `quality` | `number` | `1` ~ `100` 범위의 jpeg 품질을 지정합니다. 이미지 변환에만 적용 가능합니다. | `90` |
| `xmp` | `string` | XMP 메타데이터 원본에 쓰기에만 사용되는 기본 64 인코딩된 XMP으로 지정된 변환에 다시 쓰기됩니다. |  |
| `interlace` | `bool` | 인터레이스 PNG 또는 GIF 또는 점진적 JPEG를 `true`으로 설정하여 만듭니다. 다른 파일 형식에는 영향을 주지 않습니다. |  |
| `jpegSize` | `number` | JPEG 파일의 대략적인 크기(바이트)입니다. 이 설정은 모든 `quality` 설정을 무시합니다. 다른 형식에는 영향을 주지 않습니다. |  |
| `dpi` | `number` 또는 `object` | x 및 y DPI를 설정합니다. 간단히 말해, x 및 y 모두에 사용되는 단일 숫자로 설정할 수도 있습니다.이미지 자체에는 영향을 주지 않습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 또는 `object` | x 및 y DPI는 물리적 크기를 유지하면서 다시 샘플링합니다. 간단히 말해, x 및 y 모두에 사용되는 단일 숫자로 설정할 수도 있습니다. | `96` 또는 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | ZIP 아카이브에 포함할 파일 목록(`fmt=zip`). 각 항목은 URL 문자열이거나 필드가 있는 개체일 수 있습니다.<ul><li>`url`:파일을 다운로드할 URL</li><li>`path`:ZIP의 이 경로 아래에 파일 저장</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP 보관 파일(`fmt=zip`)에 대한 처리가 중복되었습니다. 기본적으로 ZIP의 동일한 경로 아래에 저장된 여러 파일이 오류를 생성합니다. `duplicate`을 `ignore`으로 설정하면 첫 번째 에셋만 저장되고 나머지 에셋은 무시됩니다. | `ignore` |
| `watermark` | `object` | [워터마크](#watermark-specific-fields)에 대한 지침을 포함합니다. |  |

### 워터마크 특정 필드 {#watermark-specific-fields}

PNG 형식은 워터마크로 사용됩니다.

| 이름 | 유형 | 설명 | 예 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 워터마크의 비율( `0.0`과 `1.0` 사이)입니다. `1.0` 즉, 워터마크의 원래 비율(1:1)이 있고 값이 작으면 워터마크 크기가 줄어듭니다. | `0.5` 값은 원래 크기의 절반을 의미합니다. |
| `image` | `url` | 워터마크에 사용할 PNG 파일의 URL. |  |

## 비동기 이벤트 {#asynchronous-events}

변환 처리가 완료되거나 오류가 발생하면 이벤트가 [[!DNL Adobe I/O] 이벤트 저널](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)으로 전송됩니다. 클라이언트는 [/register](#register)를 통해 제공된 저널 URL을 수신해야 합니다. 저널 응답에는 각 이벤트에 대해 하나의 객체로 구성된 `event` 배열이 포함되며 이 배열에 실제 이벤트 페이로드가 포함됩니다.`event`

[!DNL Asset Compute Service]의 모든 이벤트에 대한 [!DNL Adobe I/O] 이벤트 유형은 `asset_compute`입니다. 분기는 이 이벤트 유형에만 자동으로 구독되며 [!DNL Adobe I/O] 이벤트 유형을 기반으로 추가로 필터링할 필요가 없습니다. 서비스 특정 이벤트 유형은 이벤트의 `type` 속성에서 사용할 수 있습니다.

### 이벤트 유형 {#event-types}

| 이벤트 | 설명 |
|---------------------|-------------|
| `rendition_created` | 성공적으로 처리되고 업로드된 각 변환에 대해 전송됨. |
| `rendition_failed` | 처리 또는 업로드하지 못한 각 변환에 대해 전송됨. |

### 이벤트 특성 {#event-attributes}

| 특성 | 유형 | 이벤트 | 설명 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)에 정의된 대로, 단순화된 확장 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 형식으로 이벤트가 전송된 타임스탬프 |
| `requestId` | `string` | `*` | `X-Request-Id` 헤더와 동일한 원래 요청의 요청 ID입니다.`/process` |
| `source` | `object` | `*` | `/process` 요청의 `source`. |
| `userData` | `object` | `*` | 설정된 경우 `/process` 요청의 변환의 `userData`입니다. |
| `rendition` | `object` | `rendition_*` | `/process`에서 전달된 해당 변환 객체입니다. |
| `metadata` | `object` | `rendition_created` | 변환의 [메타데이터](#metadata) 속성. |
| `errorReason` | `string` | `rendition_failed` | 변환 오류가 발생한 경우 [reason](#error-reasons). |
| `errorMessage` | `string` | `rendition_failed` | 변환 실패 시 자세한 내용을 제공하는 텍스트입니다. |

### 메타데이터 {#metadata}

| 속성 | 설명 |
|--------|-------------|
| `repo:size` | 변환의 크기(바이트)입니다. |
| `repo:sha1` | 변환의 sha1 다이제스트입니다. |
| `dc:format` | 변환의 MIME 유형입니다. |
| `repo:encoding` | 텍스트 기반 형식인 경우 변환의 문자 인코딩입니다. |
| `tiff:ImageWidth` | 표현물의 폭(픽셀 단위)입니다. 이미지 변환에만 표시됩니다. |
| `tiff:ImageLength` | 표현물의 길이(픽셀 단위)입니다. 이미지 변환에만 표시됩니다. |

### 오류 원인 {#error-reasons}

| 이유 | 설명 |
|---------|-------------|
| `RenditionFormatUnsupported` | 요청한 변환 형식은 지정된 소스에는 지원되지 않습니다. |
| `SourceUnsupported` | 특정 소스는 지원되는 형식이지만 지원되지 않습니다. |
| `SourceCorrupt` | 소스 데이터가 손상되었습니다. 빈 파일을 포함합니다. |
| `RenditionTooLarge` | `target`에 제공된 사전 서명된 URL을 사용하여 변환을 업로드할 수 없습니다. 실제 변환 크기는 `repo:size`에서 메타데이터로 사용할 수 있으며 클라이언트가 미리 서명된 URL의 정확한 수와 함께 이 변환을 다시 처리하는 데 사용할 수 있습니다. |
| `GenericError` | 기타 예기치 않은 오류가 있습니다. |
