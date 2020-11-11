---
title: 개발 대상 [!DNL Asset Compute Service].
description: 를 사용하여 사용자 정의 응용 프로그램을 만듭니다 [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 6de4e3cde9c38f2e23838f5d728dae23e15d2147
workflow-type: tm+mt
source-wordcount: '1559'
ht-degree: 0%

---


# 맞춤형 애플리케이션 개발 {#develop}

사용자 지정 애플리케이션을 개발하기 전에 다음을 수행하십시오.

* 모든 [사전 요구 사항을](/help/understand-extensibility.md#prerequisites-and-provisioning) 만족하는지 확인합니다.
* 필요한 [소프트웨어 도구를 설치합니다](/help/setup-environment.md#create-dev-environment).
* 사용자 [지정 애플리케이션을 만들 준비가 되었는지 확인하려면 환경](setup-environment.md) 설정을 참조하십시오.

## 사용자 정의 응용 프로그램 만들기 {#create-custom-application}

Adobe [I/O CLI를 로컬로](https://github.com/adobe/aio-cli) 설치했는지 확인하십시오.

1. 사용자 정의 응용 프로그램을 만들려면 Firefox 응용 프로그램 [을 만듭니다](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli). 이렇게 하려면 터미널 `aio app init <app-name>` 에서 실행합니다.

   아직 로그인하지 않은 경우 이 명령은 Adobe ID과 함께 [Adobe 개발자 콘솔에](https://console.adobe.io/) 로그인하라는 브라우저를 표시합니다. 클립에서 로그인에 대한 자세한 내용은 [여기를](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) 참조하십시오.

   Adobe은 로그인하는 것을 권장합니다. 문제가 있는 경우 지침에 따라 로그인하지 않고 앱 [을 제작하십시오](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. 로그인한 후 CLI의 지시에 따라 애플리케이션에 대해 `Organization`, `Project`및 `Workspace` 를 선택합니다. 환경을 설정할 때 만든 프로젝트 및 작업 공간 [을 선택합니다](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. 메시지가 표시되면 다음 `Which Adobe I/O App features do you want to enable for this project?`을 적어도 `Actions`선택합니다.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 메시지가 `Which type of sample actions do you want to create?`표시되면 다음을 선택해야 합니다 `Adobe Asset Compute Worker`.

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 화면의 나머지 부분에 따라 Visual Studio 코드(또는 자주 사용하는 코드 편집기)에서 새 응용 프로그램을 엽니다. 사용자 지정 응용 프로그램의 스캐폴딩 및 샘플 코드가 포함되어 있습니다.

   Firefly 앱의 [주요 구성 요소에 대해 여기를 참조하십시오](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

   템플릿 애플리케이션은 애플리케이션 표현물의 업로드, 다운로드 및 [편성을 위해 Adobe](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) Asset compute SDK를 활용하므로 개발자는 맞춤형 애플리케이션 로직만 구현하면 됩니다. 폴더 `actions/<worker-name>` 내에서 `index.js` 사용자 지정 응용 프로그램 코드를 추가할 위치입니다.

사용자 정의 응용 프로그램에 대한 예제 및 아이디어는 [사용자 정의 응용 프로그램](#try-sample) 예를 참조하십시오.

### 자격 증명 추가 {#add-credentials}

응용 프로그램을 만들 때 로그인하면 대부분의 Firefox 자격 증명이 ENV 파일에 수집됩니다. 그러나 개발자 도구를 사용하려면 추가 자격 증명이 필요합니다.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 개발자 도구 저장소 자격 증명 {#developer-tool-credentials}

사용자 정의 응용 프로그램을 실제 버전으로 테스트하는 데 사용되는 개발자 도구는 테스트 파일을 호스팅하고 응용 프로그램에서 생성된 변환을 수신하고 표시하기 위한 클라우드 저장소 컨테이너가 필요합니다. [!DNL Asset Compute service]

>[!NOTE]
>
>이는 Cloud Service의 클라우드 스토리지와 [!DNL Adobe Experience Manager] 별개입니다. asset compute 개발자 도구를 사용한 개발 및 테스트에만 적용됩니다.

[지원되는 클라우드 스토리지 컨테이너에 액세스할 수 있는지 확인하십시오](https://github.com/adobe/asset-compute-devtool#prerequisites). 필요에 따라 여러 프로젝트에서 여러 개발자가 이 컨테이너를 공유할 수 있습니다.

#### ENV 파일에 자격 증명 추가 {#add-credentials-env-file}

Firefly 프로젝트 루트의 ENV 파일에 개발자 도구에 대한 다음 자격 증명을 추가합니다.

1. Firefly 프로젝트에 서비스를 추가하는 동안 만든 개인 키 파일에 절대 경로를 추가합니다.

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Firefox 앱 `console.json` 의 루트에 없는 경우 Adobe 개발자 콘솔 통합 JSON 파일에 절대 경로를 추가합니다. 프로젝트 작업 영역에서 다운로드한 파일과 동일한 [`console.json`](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user) 파일입니다. 또는 ENV 파일에 경로를 추가하는 `aio app use <path_to_console_json>` 대신 명령을 사용할 수도 있습니다.

   ```conf
   ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. S3 또는 Azure 저장소 자격 증명을 추가합니다. 하나의 클라우드 스토리지 솔루션만 액세스하면 됩니다.

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

## 애플리케이션 실행 {#run-custom-application}

asset compute 개발자 도구를 사용하여 응용 프로그램을 실행하기 전에 자격 증명을 [올바르게 구성합니다](#developer-tool-credentials).

개발자 도구에서 응용 프로그램을 실행하려면 `aio app run` 명령을 사용합니다. 동작을 Adobe I/O Runtime에 배포하고 로컬 시스템에서 개발 도구를 시작합니다. 이 도구는 개발 중에 응용 프로그램 요청을 테스트하는 데 사용됩니다. 다음은 변환 요청의 예입니다.

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
>명령에는 `--local` 플래그를 사용하지 `run` 마십시오. 사용자 정의 응용 프로그램 및 Asset compute 개발자 도구에서는 작동하지 않습니다. [!DNL Asset Compute] 사용자 지정 응용 프로그램은 개발자의 로컬 시스템에서 실행되는 작업에 액세스할 수 없는 사용자 지정 응용 프로그램 [!DNL Asset Compute Service] 에 의해 호출됩니다.

애플리케이션을 테스트하고 디버그하는 [방법은 여기를](test-custom-application.md) 참조하십시오. 사용자 정의 애플리케이션 개발이 끝나면 사용자 정의 응용 프로그램을 [배포합니다](deploy-custom-application.md).

## Adobe에서 제공하는 샘플 응용 프로그램 사용해 보기 {#try-sample}

다음은 사용자 지정 응용 프로그램의 예입니다.

* [노동자 기본](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [노동자 동물 사진](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 템플릿 사용자 지정 애플리케이션 {#template-custom-application}

worker-basic [은](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 템플릿 애플리케이션입니다. 소스 파일을 복사하여 변환을 생성합니다. 이 응용 프로그램의 콘텐츠는 aio 앱 생성 `Adobe Asset Compute` 에서 선택할 때 받은 템플릿입니다.

응용 프로그램 파일 [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 은 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 을 사용하여 소스 파일을 다운로드하고 각 변환 처리를 구성하며 결과 변환을 클라우드 스토리지에 다시 업로드합니다.

응용 프로그램 코드 내에 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 정의된 이 위치에서 모든 응용 프로그램 처리 논리를 수행합니다. 변환 콜백은 소스 파일 내용을 변환 파일에 `worker-basic` 간단히 복사합니다.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 외부 API 호출 {#call-external-api}

응용 프로그램 코드에서 외부 API 호출을 만들어 응용 프로그램 처리에 도움을 줄 수 있습니다. 외부 API를 호출하는 예제 응용 프로그램 파일이 아래에 있습니다.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

예를 들어, 라이브러리 [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 를 사용하여 위키미디어에서 정적 URL로 가져오기 요청을 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 만듭니다.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 사용자 지정 매개 변수 전달 {#pass-custom-parameters}

사용자 정의 매개 변수를 변환 개체를 통해 전달할 수 있습니다. 애플리케이션 내에서 지침에 따라 참조할 수 [`rendition` 있습니다](https://github.com/adobe/asset-compute-sdk#rendition). 변환 개체의 예는 다음과 같습니다.

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

이 `example-worker-animal-pictures` 는 사용자 지정 매개 변수 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 를 전달하여 위키미디어에서 가져올 파일을 결정합니다.

## 인증 및 인증 지원 {#authentication-authorization-support}

기본적으로 Asset compute 사용자 정의 응용 프로그램은 Firefox 응용 프로그램에 대한 인증 및 인증 검사와 함께 제공됩니다. 주석을 에서 `require-adobe-auth` 로 설정하여 이 `true` 를 활성화할 수 `manifest.yml`있습니다.

### 다른 Adobe API 액세스 {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

API 서비스를 설정에서 만든 콘솔 작업 영역에 [!DNL Asset Compute] 추가합니다. 이러한 서비스는 에서 생성된 JWT 액세스 토큰의 일부입니다 [!DNL Asset Compute Service]. 토큰 및 기타 자격 증명은 응용 프로그램 작업 개체 내에서 액세스할 수 `params` 있습니다.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 타사 시스템에 대한 자격 증명 전달 {#pass-credentials-for-tp}

다른 외부 서비스에 대한 자격 증명을 처리하려면 작업에 기본 매개 변수로 전달합니다. 이러한 것들은 전송 중에 자동으로 암호화됩니다. 자세한 내용은 런타임 개발자 안내서에서 [작업 만들기를 참조하십시오](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). 그런 다음 배포하는 동안 환경 변수를 사용하여 설정합니다. 이러한 매개 변수는 작업 내의 `params` 개체에서 액세스할 수 있습니다.

의 기본 매개 변수 `inputs` 를 `manifest.yml`설정합니다.

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

표현식은 `$VAR` 이름이 지정된 환경 변수의 값을 읽습니다 `VAR`.

개발 중에 이 값은 로컬 ENV 파일에서 설정할 수 있습니다. ENV 파일의 환경 변수와 호출 셸에서 설정된 변수를 `aio` 자동으로 읽습니다. 이 예에서 ENV 파일은 다음과 같습니다.

```CONF
#...
SECRET_KEY=secret-value
```

프로덕션 배포의 경우 CI 시스템의 환경 변수를 설정할 수 있습니다. 예를 들어 GitHub 작업의 비밀을 사용합니다. 마지막으로 다음과 같이 애플리케이션 내의 기본 매개 변수에 액세스합니다.

```javascript
const key = params.secretKey;
```

## 애플리케이션 크기 조정 {#sizing-workers}

애플리케이션은 다음 링크를 통해 구성할 수 있는 [제한](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) 으로 Adobe I/O Runtime의 컨테이너에서 실행됩니다 `manifest.yml`.

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

일반적으로 Asset compute 애플리케이션에서 수행하는 보다 광범위한 처리 기능으로 인해 최적의 성능(바이너리 에셋을 처리할 수 있을 정도로 크기)과 효율성(사용되지 않은 컨테이너 메모리로 인해 리소스를 낭비하지 않음)을 위해 이러한 제한을 조정해야 할 가능성이 높습니다.

Runtime의 작업에 대한 기본 시간 제한은 1분이지만, 제한(밀리초)을 설정하여 `timeout` 늘릴 수 있습니다. 더 큰 파일을 처리하려면 이 시간을 늘리십시오. 소스를 다운로드하고 파일을 처리하고 변환을 업로드하는 데 걸리는 총 시간을 고려합니다. 작업 시간이 초과되는 경우, 즉 지정된 제한 시간 이전에 활성화를 반환하지 않는 경우, 런타임은 컨테이너를 삭제하고 다시 사용하지 않습니다.

기본적으로 asset compute 애플리케이션은 네트워크 및 디스크 IO 바인딩되는 경향이 있습니다. 소스 파일을 먼저 다운로드해야 하고, IO가 많은 경우가 많으며, 따라서 변환이 다시 업로드됩니다.

작업 컨테이너에 사용할 수 있는 메모리는 MB `memorySize` 로 지정됩니다. 현재 이 설정은 컨테이너가 받는 CPU 액세스 수량도 정의하며, 가장 중요한 것은 Runtime 사용 비용의 핵심 요소입니다(더 큰 컨테이너 비용 증가). 처리 시 더 많은 메모리 또는 CPU가 필요할 때 여기에 더 큰 값을 사용하되, 컨테이너가 더 큰 만큼 리소스를 낭비하지 않도록 주의하십시오.

또한 설정을 사용하여 컨테이너 내의 작업 동시 사용을 제어할 수 `concurrency` 있습니다. 단일 컨테이너(동일한 작업)가 가져오는 동시 활성화 수입니다. 이 모델에서 작업 컨테이너는 최대 해당 제한까지 여러 개의 동시 요청을 받는 Node.js 서버와 같습니다. 설정하지 않으면 런타임의 기본값은 200으로, 작은 Firefly 액션에는 아주 좋으나, 일반적으로 Asset compute 응용 프로그램이 보다 집약적인 로컬 처리 및 디스크 작업을 제공했을 때는 너무 큽니다. 구현에 따라 일부 응용 프로그램이 동시 활동에서 제대로 작동하지 않을 수도 있습니다. asset compute SDK에서는 여러 고유한 폴더에 파일을 작성하여 활성화가 구분되도록 합니다.

응용 프로그램을 테스트하여 `concurrency` 및 `memorySize`. 대용량 컨테이너 = 메모리 용량이 클수록 동시 시청 횟수가 늘어날 수 있지만, 트래픽 저하를 위한 낭비도 발생할 수 있습니다.
