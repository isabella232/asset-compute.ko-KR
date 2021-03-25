---
title: ' [!DNL Asset Compute Service]에 대한 현상'
description: ' [!DNL Asset Compute Service]을(를) 사용하여 사용자 정의 응용 프로그램을 만듭니다.'
translation-type: tm+mt
source-git-commit: 7ae47fdb7ff91e1388d2037d90abe35fe5218216
workflow-type: tm+mt
source-wordcount: '1615'
ht-degree: 0%

---


# 사용자 정의 응용 프로그램 {#develop} 개발

사용자 정의 응용 프로그램을 개발하기 전에 다음을 수행하십시오.

* [사전 요구 사항](/help/understand-extensibility.md#prerequisites-and-provisioning)이 모두 충족되었는지 확인합니다.
* [필수 소프트웨어 도구](/help/setup-environment.md#create-dev-environment)를 설치합니다.
* 사용자 지정 응용 프로그램을 만들 준비가 되었는지 확인하려면 [환경 설정](setup-environment.md)을 참조하십시오.

## 사용자 정의 응용 프로그램 {#create-custom-application} 만들기

[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)를 로컬로 설치하도록 하십시오.

1. 사용자 지정 응용 프로그램을 만들려면 [Firefox 응용 프로그램](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)을 만듭니다. 이렇게 하려면 터미널에서 `aio app init <app-name>`을(를) 실행하십시오.

   아직 로그인하지 않은 경우 이 명령은 Adobe ID을 사용하여 [Adobe 개발자 콘솔](https://console.adobe.io/)에 로그인하라는 브라우저를 표시합니다. 클립에서 로그인하는 방법에 대한 자세한 내용은 [여기](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)를 참조하십시오.

   Adobe에서는 로그인하는 것이 좋습니다. 문제가 있는 경우 [지침에 따라](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)에 로그인하지 않고 앱을 만드십시오.

1. 로그인한 후 CLI의 지시에 따라 `Organization`, `Project` 및 `Workspace`를 선택하여 애플리케이션에 사용합니다. [환경](setup-environment.md)을 설정할 때 만든 프로젝트 및 작업 영역을 선택합니다.

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. `Which Adobe I/O App features do you want to enable for this project?` 메시지가 표시되면 `Actions`을 선택합니다. 웹 에셋은 다른 인증 및 인증 검사를 사용하기 때문에 `Web Assets` 옵션을 선택 취소해야 합니다.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. `Which type of sample actions do you want to create?` 메시지가 표시되면 `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 나머지 프롬프트에 따라 Visual Studio 코드(또는 자주 사용하는 코드 편집기)에서 새 응용 프로그램을 엽니다. 사용자 정의 응용 프로그램에 대한 스캐폴딩 및 샘플 코드가 포함되어 있습니다.

   Firefly 앱](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)의 [기본 구성 요소에 대해 여기에서 확인하십시오.

   템플릿 응용 프로그램은 응용 프로그램 표현물의 업로드, 다운로드 및 오케스트레이션을 위해 [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk)를 활용하므로 개발자는 사용자 정의 응용 프로그램 로직을 구현하기만 하면 됩니다. `actions/<worker-name>` 폴더 내에서 `index.js` 파일은 사용자 지정 응용 프로그램 코드를 추가할 위치입니다.

사용자 정의 응용 프로그램에 대한 예와 아이디어는 [사용자 정의 응용 프로그램 예제](#try-sample)를 참조하십시오.

### 자격 증명 {#add-credentials} 추가

응용 프로그램을 만들 때 로그인하면 대부분의 Firefox 자격 증명이 ENV 파일에 수집됩니다. 그러나 개발자 도구를 사용하려면 추가 자격 증명이 필요합니다.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 개발자 도구 저장소 자격 증명 {#developer-tool-credentials}

실제 [!DNL Asset Compute service]으로 사용자 정의 응용 프로그램을 테스트하는 데 사용되는 개발자 도구는 테스트 파일을 호스팅하고 응용 프로그램에서 생성된 변환을 수신하고 표시하기 위한 클라우드 저장소 컨테이너가 필요합니다.

>[!NOTE]
>
>이것은 [!DNL Adobe Experience Manager]의 클라우드 저장소와 [!DNL Cloud Service]로 별개입니다. 이는 Asset compute 개발자 도구를 사용한 개발 및 테스트에만 적용됩니다.

[지원되는 클라우드 스토리지 컨테이너](https://github.com/adobe/asset-compute-devtool#prerequisites)에 액세스할 수 있는지 확인하십시오. 필요에 따라 여러 다른 프로젝트에서 여러 개발자가 이 컨테이너를 공유할 수 있습니다.

#### ENV 파일 {#add-credentials-env-file}에 자격 증명 추가

Firefly 프로젝트 루트의 ENV 파일에 개발자 도구에 대한 다음 자격 증명을 추가합니다.

1. Firefly 프로젝트에 서비스를 추가하는 동안 만든 개인 키 파일에 절대 경로를 추가합니다.

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Adobe 개발자 콘솔에서 파일을 다운로드합니다. 프로젝트의 루트로 이동하고 오른쪽 위 모서리에서 &quot;모두 다운로드&quot;를 클릭합니다. 파일이 파일 이름으로 `<namespace>-<workspace>.json`로 다운로드됩니다. 다음 중 하나를 수행하십시오.

   * 파일의 이름을 `console.json`으로 변경하고 프로젝트의 루트에서 이동합니다.
   * 선택적으로 Adobe 개발자 콘솔 통합 JSON 파일에 절대 경로를 추가할 수 있습니다. 프로젝트 작업 영역에서 다운로드한 파일과 동일한 [`console.json`](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user) 파일입니다.

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. S3 또는 Azure 저장소 자격 증명을 추가합니다. 하나의 클라우드 스토리지 솔루션만 액세스할 수 있습니다.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>`config.json` 파일에 자격 증명이 있습니다. 프로젝트 내에서 JSON 파일을 `.gitignore` 파일에 추가하여 공유를 방지합니다. .env 및 .aio 파일에도 동일하게 적용됩니다.

## 응용 프로그램 {#run-custom-application} 실행

asset compute 개발자 도구를 사용하여 응용 프로그램을 실행하기 전에 [자격 증명](#developer-tool-credentials)을 올바르게 구성합니다.

개발자 도구에서 응용 프로그램을 실행하려면 `aio app run` 명령을 사용합니다. 동작을 [!DNL Adobe I/O] Runtime에 배포하고 로컬 컴퓨터에서 개발 도구를 시작합니다. 이 도구는 개발 중에 응용 프로그램 요청을 테스트하는 데 사용됩니다. 다음은 변환 요청의 예입니다.

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>`run` 명령과 함께 `--local` 플래그를 사용하지 마십시오. 이 도구는 [!DNL Asset Compute] 사용자 정의 응용 프로그램 및 Asset compute 개발자 도구에서는 작동하지 않습니다. 사용자 지정 응용 프로그램은 개발자의 로컬 컴퓨터에서 실행 중인 작업에 액세스할 수 없는 [!DNL Asset Compute Service]에 의해 호출됩니다.

응용 프로그램을 테스트하고 디버깅하는 방법을 보려면 [여기](test-custom-application.md)를 참조하십시오. 사용자 지정 응용 프로그램 개발이 완료되면 [사용자 지정 응용 프로그램](deploy-custom-application.md)을(를) 배포합니다.

## Adobe {#try-sample}에서 제공하는 샘플 응용 프로그램을 사용해 보십시오.

다음은 사용자 정의 응용 프로그램의 예입니다.

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [노동자 동물 사진](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 템플릿 사용자 지정 응용 프로그램 {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)은 템플릿 응용 프로그램입니다. 소스 파일을 간단히 복사하여 변환을 생성합니다. 이 응용 프로그램의 내용은 aio 앱 제작에서 `Adobe Asset Compute`을 선택하면 받은 템플릿입니다.

응용 프로그램 파일인 [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)은 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview)을 사용하여 소스 파일을 다운로드하고 각 변환 처리를 조정하고 결과 변환을 다시 클라우드 스토리지에 업로드합니다.

응용 프로그램 코드 내에 정의된 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 는 모든 응용 프로그램 처리 논리를 수행할 위치입니다. `worker-basic`의 변환 콜백은 소스 파일 내용을 변환 파일에 복사하는 것입니다.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 외부 API {#call-external-api} 호출

응용 프로그램 코드에서 외부 API 호출을 수행하여 응용 프로그램 처리에 도움을 줄 수 있습니다. 외부 API를 호출하는 예제 응용 프로그램 파일이 아래에 있습니다.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

예를 들어 [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46)은 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 라이브러리를 사용하여 위키미디어에서 정적 URL에 가져오기 요청을 합니다.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 사용자 지정 매개 변수 {#pass-custom-parameters} 전달

사용자 정의 매개 변수를 변환 개체를 통해 전달할 수 있습니다. 이 매개 변수는 [`rendition` 지침](https://github.com/adobe/asset-compute-sdk#rendition)에서 응용 프로그램 내에서 참조할 수 있습니다. 변환 개체의 예는 다음과 같습니다.

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

사용자 지정 매개 변수에 액세스하는 응용 프로그램 파일의 예는 다음과 같습니다.

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures`은 사용자 지정 매개 변수 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39)를 전달하여 위키미디어에서 가져올 파일을 결정합니다.

## 인증 및 인증 지원 {#authentication-authorization-support}

기본적으로 Asset compute 사용자 정의 응용 프로그램은 Firefox 응용 프로그램에 대한 인증 및 인증 검사와 함께 제공됩니다. 이 기능은 `manifest.yml`에서 `require-adobe-auth` 주석을 `true`으로 설정하여 사용할 수 있습니다.

### 다른 Adobe API {#access-adobe-apis} 액세스

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

API 서비스를 설정에서 만든 [!DNL Asset Compute] 콘솔 작업 영역에 추가합니다. 이러한 서비스는 [!DNL Asset Compute Service]에서 생성된 JWT 액세스 토큰의 일부입니다. 토큰 및 기타 자격 증명은 응용 프로그램 작업 `params` 개체 내에서 액세스할 수 있습니다.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 타사 시스템 {#pass-credentials-for-tp}에 대한 자격 증명 전달

다른 외부 서비스에 대한 자격 증명을 처리하려면 작업에 기본 매개 변수로 전달합니다. 이러한 항목은 전송 중에 자동으로 암호화됩니다. 자세한 내용은 런타임 개발자 안내서](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)에서 액션 만들기를 참조하십시오. [ 그런 다음 배포하는 동안 환경 변수를 사용하여 설정합니다. 이러한 매개 변수는 작업 내의 `params` 개체에서 액세스할 수 있습니다.

`manifest.yml`의 `inputs` 내에 기본 매개 변수를 설정합니다.

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

`$VAR` 표현식은 `VAR`이라는 환경 변수의 값을 읽습니다.

개발 중에 이 값을 로컬 ENV 파일에서 설정할 수 있습니다. 이 값은 호출 셸에서 설정된 변수 외에 ENV 파일의 환경 변수를 자동으로 읽습니다. `aio` 이 예에서 ENV 파일은 다음과 같습니다.

```CONF
#...
SECRET_KEY=secret-value
```

제작 배포의 경우 CI 시스템의 환경 변수를 설정할 수 있습니다(예: GitHub 동작의 비밀 사용). 마지막으로 다음과 같이 애플리케이션 내에서 기본 매개 변수에 액세스합니다.

```javascript
const key = params.secretKey;
```

## 애플리케이션 크기 조정 {#sizing-workers}

응용 프로그램은 `manifest.yml`을(를) 통해 구성할 수 있는 [limits](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md)와 함께 [!DNL Adobe I/O] Runtime의 컨테이너에서 실행됩니다.

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

일반적으로 Asset compute 응용 프로그램에서 수행하는 보다 광범위한 처리 때문에 최적의 성능(바이너리 에셋을 처리할 수 있을 만큼 큼)과 효율성(사용되지 않은 컨테이너 메모리로 인해 리소스를 낭비하지 않음)을 위해 이러한 제한을 조정해야 하는 경우가 많습니다.

Runtime의 작업에 대한 기본 시간 제한은 1분이지만 `timeout` 제한(밀리초)을 설정하여 이를 높일 수 있습니다. 더 큰 파일을 처리하려면 이 시간을 늘리십시오. 소스를 다운로드하고 파일을 처리하고 변환을 업로드하는 데 걸리는 총 시간을 고려합니다. 작업이 시간 초과되는 경우, 즉 지정된 시간 제한 전에 활성화를 반환하지 않으면 런타임은 컨테이너를 폐기하고 다시 사용하지 않습니다.

기본적으로 asset compute 애플리케이션은 네트워크 및 디스크 입력 또는 출력 바인딩된 애플리케이션입니다. 소스 파일을 먼저 다운로드해야 하며, 처리 시 종종 리소스가 많이 소모되는 경우가 있고 그 결과 변환이 다시 업로드됩니다.

작업 컨테이너에 사용할 수 있는 메모리는 `memorySize`에서 MB로 지정됩니다. 현재 이 값은 컨테이너가 얻는 CPU 액세스 정도를 정의하며, 가장 중요한 것은 Runtime 사용 비용의 핵심 요소입니다(큰 컨테이너 비용이 더 많음). 처리에는 더 많은 메모리 또는 CPU가 필요하지만 컨테이너가 더 클수록 전체 처리량이 감소하므로 리소스를 낭비하지 않도록 주의하십시오.

또한 `concurrency` 설정을 사용하여 컨테이너 내의 작업 동시 사용을 제어할 수 있습니다. 단일 컨테이너(동일한 작업)가 가져오는 동시 활동 수입니다. 이 모델에서 작업 컨테이너는 최대 해당 제한까지 여러 동시 요청을 받는 Node.js 서버와 같습니다. 설정하지 않은 경우, Runtime의 기본값은 200으로, 이것은 작은 Firefly 작업에는 매우 좋으나, 일반적으로 Asset compute 응용 프로그램이 더욱 집약적인 로컬 처리 및 디스크 작업을 제공했을 때 너무 큽니다. 구현에 따라 일부 응용 프로그램이 동시 활동에 적합하지 않을 수도 있습니다. asset compute SDK에서는 여러 고유한 폴더에 파일을 작성하여 활성화가 구분되도록 합니다.

응용 프로그램을 테스트하여 `concurrency` 및 `memorySize`에 대한 최적의 숫자를 찾습니다. 컨테이너가 더 크면 메모리 제한이 더 많을수록 동시 작업이 많아질 수 있지만 트래픽이 더 적은 경우에도 낭비될 수 있습니다.
