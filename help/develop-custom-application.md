---
title: 개발 대상 [!DNL Asset Compute Service]
description: 을 사용하여 사용자 정의 애플리케이션 만들기 [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: a121b48d480b45405259c2061ac86b9ab46b89cb
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# 사용자 지정 애플리케이션 개발 {#develop}

사용자 지정 애플리케이션 개발을 시작하기 전에:

* 모든 [전제 조건](/help/understand-extensibility.md#prerequisites-and-provisioning) 이(가) 충족됩니다.
* 설치 [필수 소프트웨어 도구](/help/setup-environment.md#create-dev-environment).
* 자세한 내용은 [환경 설정](setup-environment.md) 사용자 정의 애플리케이션을 만들 준비가 되었는지 확인합니다.

## 사용자 지정 애플리케이션 만들기 {#create-custom-application}

반드시 을(를) 확인하십시오 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 로컬에 설치되었습니다.

1. 사용자 정의 응용 프로그램을 만들려면 [app Builder 프로젝트 만들기](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). 이렇게 하려면 를 실행합니다 `aio app init <app-name>` 터미널.

   아직 로그인하지 않은 경우 이 명령은 브라우저에 로그인하라는 메시지를 표시합니다 [Adobe Developer 콘솔](https://console.adobe.io/) Adobe ID 사용. 자세한 내용은 [여기](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) cli에서 로그인에 대한 자세한 정보.

   Adobe은 로그인하는 것을 권장합니다. 문제가 있는 경우 지침을 따르십시오 [로그인하지 않고 앱을 만들려면](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. 로그인한 후 CLI의 지시에 따라 다음을 선택합니다 `Organization`, `Project`, 및 `Workspace` 을 입력합니다. 만들 때 만든 프로젝트 및 작업 공간을 선택합니다 [환경 설정](setup-environment.md). 메시지가 표시되면 `Which extension point(s) do you wish to implement ?`, 을 선택해야 합니다. `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. 메시지가 표시되면 `Which Adobe I/O App features do you want to enable for this project?`, 선택 `Actions`. 선택 취소 확인 `Web Assets` 웹 자산으로서의 옵션은 다른 인증 및 권한 부여 검사를 사용합니다.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 메시지가 표시되면 `Which type of sample actions do you want to create?`, 을 선택해야 합니다. `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 화면의 나머지 부분에 따라 Visual Studio 코드(또는 즐겨찾는 코드 편집기)에서 새 응용 프로그램을 엽니다. 사용자 지정 애플리케이션에 대한 스캐폴딩과 샘플 코드가 포함되어 있습니다.

   자세한 내용은 여기 를 참조하십시오. [App Builder 앱의 기본 구성 요소](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   템플릿 응용 프로그램은 [asset compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) 애플리케이션 표현물의 업로드, 다운로드 및 오케스트레이션의 경우, 개발자가 사용자 지정 애플리케이션 로직만 구현하면 됩니다. 내부 `actions/<worker-name>` 폴더, `index.js` 파일은 사용자 지정 애플리케이션 코드를 추가할 위치입니다.

자세한 내용은 [사용자 지정 애플리케이션 예](#try-sample) 사용자 정의 응용 프로그램에 대한 예제 및 아이디어를 참조하십시오.

### 자격 증명 추가 {#add-credentials}

애플리케이션을 만들 때 로그인하면 대부분의 App Builder 자격 증명이 ENV 파일에서 수집됩니다. 하지만 개발자 도구를 사용하려면 추가 자격 증명이 필요합니다.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 개발자 도구 저장소 자격 증명 {#developer-tool-credentials}

실제 [!DNL Asset Compute service] 테스트 파일을 호스팅하고 애플리케이션에서 생성된 렌디션을 받고 표시하기 위해 클라우드 스토리지 컨테이너가 필요합니다.

>[!NOTE]
>
>클라우드 저장소와는 별개입니다 [!DNL Adobe Experience Manager] 로서의 [!DNL Cloud Service]. asset compute 개발자 도구를 사용한 개발 및 테스트에만 적용됩니다.

에 액세스할 수 있는지 확인합니다. [지원되는 클라우드 저장소 컨테이너](https://github.com/adobe/asset-compute-devtool#prerequisites). 필요에 따라 여러 프로젝트에서 여러 개발자가 이 컨테이너를 공유할 수 있습니다.

#### ENV 파일에 자격 증명 추가 {#add-credentials-env-file}

개발자 도구에 대한 다음 자격 증명을 App Builder 프로젝트의 루트의 ENV 파일에 추가합니다.

1. App Builder 프로젝트에 서비스를 추가하는 동안 만든 개인 키 파일에 절대 경로를 추가합니다.

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Adobe Developer 콘솔에서 파일을 다운로드합니다. 프로젝트의 루트로 이동하고 오른쪽 위 모서리에 있는 &quot;Download All&quot;을 클릭합니다. 파일은 로 다운로드됩니다. `<namespace>-<workspace>.json` 를 파일 이름으로 지정합니다. 다음 중 하나를 수행하십시오.

   * 파일 이름을 다음으로 변경합니다. `console.json` 프로젝트의 루트로 이동합니다.
   * 선택적으로 Adobe Developer 콘솔 통합 JSON 파일에 절대 경로를 추가할 수 있습니다. 이것은 같다 [`console.json`](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) project workspace에서 다운로드한 파일입니다.

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

>[!TIP]
>
>다음 `config.json` 파일에 자격 증명이 포함되어 있습니다. 프로젝트 내에서 JSON 파일을 `.gitignore` 파일을 공유하지 않도록 합니다. .env 및 .aio 파일에도 적용됩니다.

## 응용 프로그램 실행 {#run-custom-application}

asset compute 개발자 도구로 애플리케이션을 실행하기 전에 [자격 증명](#developer-tool-credentials).

개발자 도구에서 애플리케이션을 실행하려면 `aio app run` 명령. 작업을 배포합니다. [!DNL Adobe I/O] 런타임 및 로컬 시스템에서 개발 도구를 시작합니다. 이 도구는 개발 중에 응용 프로그램 요청을 테스트하는 데 사용됩니다. 다음은 변환 요청의 예입니다.

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
>를 사용하지 마십시오 `--local` 플래그 `run` 명령. 와 작동하지 않습니다 [!DNL Asset Compute] 사용자 정의 애플리케이션 및 Asset compute 개발자 도구. 사용자 지정 응용 프로그램은 [!DNL Asset Compute Service] 개발자 로컬 시스템에서 실행 중인 작업에 액세스할 수 없습니다.

자세한 내용은 [여기](test-custom-application.md) 애플리케이션을 테스트하고 디버깅하는 방법입니다. 사용자 지정 애플리케이션 개발을 마치면 [사용자 지정 애플리케이션 배포](deploy-custom-application.md).

## Adobe에서 제공하는 샘플 응용 프로그램을 사용해 보십시오 {#try-sample}

다음은 사용자 정의 응용 프로그램의 예입니다.

* [작업자 기본](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [노동자 동물 사진](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 템플릿 사용자 지정 애플리케이션 {#template-custom-application}

다음 [작업자 기본](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 은 템플릿 애플리케이션입니다. 원본 파일을 복사하여 렌디션을 생성합니다. 이 응용 프로그램의 내용은 `Adobe Asset Compute` aio 앱을 만들 때 입니다.

애플리케이션 파일, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 사용 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 소스 파일을 다운로드하려면 각 변환 처리를 오케스트레이션하고 결과 변환을 다시 클라우드 저장소로 업로드합니다.

다음 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 응용 프로그램 코드 내에 정의되어 있습니다. 여기서 모든 응용 프로그램 처리 논리를 수행할 수 있습니다. 의 변환 콜백 `worker-basic` 소스 파일 내용을 변환 파일에 복사하면 됩니다.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 외부 API 호출 {#call-external-api}

애플리케이션 코드에서는 애플리케이션 처리에 도움이 되도록 외부 API 호출을 수행할 수 있습니다. 외부 API를 호출하는 예제 애플리케이션 파일은 아래에 있습니다.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

예: [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 에서는 다음을 사용하여 Wikimedia에서 정적 URL에 가져오기 요청을 만듭니다 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 라이브러리.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 사용자 지정 매개 변수 전달 {#pass-custom-parameters}

사용자정의 매개변수를 변환 객체를 통해 전달할 수 있습니다. URL은 의 애플리케이션 내에서 참조할 수 있습니다 [`rendition` 지침](https://github.com/adobe/asset-compute-sdk#rendition). 변환 개체의 예는 다음과 같습니다.

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

다음 `example-worker-animal-pictures` 사용자 지정 매개 변수를 전달합니다 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) Wikimedia에서 가져올 파일을 결정합니다.

## 인증 및 인증 지원 {#authentication-authorization-support}

기본적으로 Asset compute 사용자 지정 애플리케이션에는 App Builder 프로젝트에 대한 인증 및 인증 검사가 제공됩니다. 이 기능은 `require-adobe-auth` 주석 `true` 에서 `manifest.yml`.

### 다른 Adobe API에 액세스 {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

에 API 서비스 추가 [!DNL Asset Compute] 설치 프로그램에서 만든 콘솔 작업 영역입니다. 이러한 서비스는 [!DNL Asset Compute Service]. 토큰 및 기타 자격 증명은 애플리케이션 작업 내에서 액세스할 수 있습니다 `params` 개체.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 타사 시스템에 대한 자격 증명 전달 {#pass-credentials-for-tp}

다른 외부 서비스에 대한 자격 증명을 처리하려면 작업에서 이러한 자격 증명을 기본 매개 변수로 전달합니다. 이러한 데이터는 전송 중에 자동으로 암호화됩니다. 자세한 내용은 [런타임 개발자 안내서에서 작업 만들기](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). 그런 다음 배포 중에 환경 변수를 사용하여 설정합니다. 이러한 매개 변수는 `params` 작업 내의 개체

내에서 기본 매개 변수를 설정합니다 `inputs` 에서 `manifest.yml`:

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

다음 `$VAR` 표현식은 이름이 지정된 환경 변수에서 값을 읽습니다. `VAR`.

개발 중에 로컬 ENV 파일에서 값을 다음과 같이 설정할 수 있습니다. `aio` 는 호출 쉘에서 설정된 변수 외에 ENV 파일에서 환경 변수를 자동으로 읽습니다. 이 예제에서 ENV 파일은 다음과 같습니다.

```CONF
#...
SECRET_KEY=secret-value
```

프로덕션 배포의 경우 CI 시스템의 환경 변수를 설정할 수 있습니다. 예를 들어 GitHub 작업에서 비밀 사용을 예로 들 수 있습니다. 마지막으로 애플리케이션 내의 기본 매개 변수에 다음과 같이 액세스합니다.

```javascript
const key = params.secretKey;
```

## 애플리케이션 크기 조정 {#sizing-workers}

응용 프로그램은 의 컨테이너에서 실행됩니다 [!DNL Adobe I/O] 런타임 및 [제한](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) 다음을 통해 구성할 수 있습니다. `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

일반적으로 Asset compute 애플리케이션에서 수행하는 보다 광범위한 처리로 인해 최적의 성능(이진 자산을 처리할 수 있을 정도로 크기)과 효율성(사용하지 않는 컨테이너 메모리로 인해 리소스를 낭비하지 않음)을 위해 이러한 제한을 조정해야 할 가능성이 높습니다.

런타임 작업의 기본 시간 제한은 1분이지만 `timeout` 제한(밀리초) 더 큰 파일을 처리할 것으로 예상되면 이 시간을 늘립니다. 소스를 다운로드하고 파일을 처리하고 렌디션을 업로드하는 데 걸리는 총 시간을 고려합니다. 작업이 시간 초과되면 지정된 시간 제한 전에 활성화를 반환하지 않으면 런타임에서 컨테이너를 삭제하고 다시 사용하지 않습니다.

기본적으로 asset compute 응용 프로그램은 네트워크 및 디스크 입력 또는 출력 바인딩인 경향이 있습니다. 소스 파일을 먼저 다운로드해야 하며, 처리가 종종 리소스를 많이 사용하는 경우 결과 변환이 다시 업로드됩니다.

작업 컨테이너에 사용할 수 있는 메모리는 `memorySize` MB입니다. 현재 이는 컨테이너가 받는 CPU 액세스 횟수도 정의하며, 가장 중요한 것은 런타임 사용 비용의 주요 요소입니다(더 큰 컨테이너 비용 증가). 처리에는 더 많은 메모리나 CPU가 필요할 때 여기서 더 큰 값을 사용하되, 컨테이너가 더 커질수록 전체 처리량이 낮아지지 않도록 주의하십시오.

또한, 컨테이너 내의 동작 동시성을 `concurrency` 설정 단일 컨테이너(동일한 작업 중)가 동시에 활성화하는 활동의 수입니다. 이 모델에서 작업 컨테이너는 최대 해당 제한까지 여러 동시 요청을 받는 Node.js 서버와 같습니다. 설정하지 않으면 런타임 시 기본값이 200이지만, 이 값은 더 작은 App Builder 작업에는 좋지만, 일반적으로 Asset compute 응용 프로그램에 대해 너무 커서 보다 많은 로컬 처리 및 디스크 작업이 제공됩니다. 일부 애플리케이션은 구현에 따라 동시 사용 활동에서도 제대로 작동하지 않을 수 있습니다. asset compute SDK를 사용하면 다른 고유한 폴더에 파일을 작성하여 활동을 분리할 수 있습니다.

응용 프로그램을 테스트하여 `concurrency` 및 `memorySize`. 큰 컨테이너 = 높은 메모리 제한은 더 많은 동시 시청 시간을 허용하지만 낮은 트래픽에 낭비될 수도 있습니다.
