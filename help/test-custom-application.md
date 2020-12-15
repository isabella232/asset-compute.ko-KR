---
title: '사용자 지정 응용 프로그램을 테스트하고 디버깅합니다. [!DNL Asset Compute Service] '
description: '사용자 지정 응용 프로그램을 테스트하고 디버깅합니다. [!DNL Asset Compute Service] '
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# 사용자 지정 응용 프로그램 {#test-debug-custom-worker} 테스트 및 디버그

## 사용자 지정 응용 프로그램 {#test-custom-worker}에 대한 단위 테스트 실행

컴퓨터에 [Docker Desktop](https://www.docker.com/get-started)을(를) 설치합니다. 사용자 지정 작업자를 테스트하려면 응용 프로그램 루트에서 다음 명령을 실행합니다.

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

아래 설명된 대로 프로젝트에서 Asset compute 응용 프로그램 작업에 대한 사용자 정의 단위 테스트 프레임워크를 실행합니다. `package.json` 파일의 구성을 통해 연결됩니다. 또한 Test와 같은 JavaScript 단위 테스트가 있을 수도 있습니다. `aio app test` 둘 다 실행합니다.

빌드/테스트 시스템에 설치하지 않아도 되도록 [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 플러그인이 사용자 지정 응용 프로그램 앱에 개발 종속성으로 포함되어 있습니다.

### 응용 프로그램 단위 테스트 프레임워크 {#unit-test-framework}

asset compute 애플리케이션 단위 테스트 프레임워크를 사용하면 코드를 작성하지 않고도 애플리케이션을 테스트할 수 있습니다. 응용 프로그램의 변환 파일 원리에 대한 소스에 의존합니다. 테스트 소스 파일, 선택적 매개 변수, 예상 변환 및 사용자 정의 유효성 검사 스크립트를 사용하여 테스트 케이스를 정의하려면 특정 파일 및 폴더 구조를 설정해야 합니다. 기본적으로 표현물은 바이트 평등에 대해 비교됩니다. 또한 간단한 JSON 파일을 사용하여 외부 HTTP 서비스를 손쉽게 조롱할 수 있습니다.

### 테스트 추가 {#add-tests}

테스트는 AIO 프로젝트의 루트 수준에 있는 `test` 폴더 내에 있어야 합니다. 각 응용 프로그램에 대한 테스트 케이스는 각 테스트 케이스에 대해 하나의 폴더가 있는 `test/asset-compute/<worker-name>` 경로에 있어야 합니다.

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

[예제 사용자 지정 응용 프로그램](https://github.com/adobe/asset-compute-example-workers/)에서 몇 가지 예를 확인하십시오. 다음은 자세한 참조입니다.

### 테스트 출력 {#test-output}

사용자 정의 응용 프로그램의 로그를 비롯한 자세한 테스트 출력은 `aio app test` 출력에 설명된 대로 Firefly 응용 프로그램 루트의 `build` 폴더에서 사용할 수 있습니다.

### 외부 서비스 모의{#mock-external-services}

HOST_NAME이 시정할 호스트인 테스트 케이스에서 `mock-<HOST_NAME>.json` 파일을 정의하여 액션에서 외부 서비스 호출을 시정할 수 있습니다. 사용 사례는 S3를 별도로 호출하는 응용 프로그램입니다. 새 테스트 구조는 다음과 같습니다.

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

모의 파일은 JSON 형식의 http 응답입니다. 자세한 내용은 [이 설명서](https://www.mock-server.com/mock_server/creating_expectations.html)를 참조하십시오. 시정할 호스트 이름이 여러 개인 경우 여러 `mock-<mocked-host>.json` 파일을 정의합니다. 다음은 `mock-google.com.json`이라는 이름의 `google.com`에 대한 샘플 모의 파일입니다.

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

`worker-animal-pictures` 예에는 상호 작용하는 위키미디어 서비스에 대한 [모의 파일](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)이 포함되어 있습니다.

#### 테스트 케이스 간에 파일 공유 {#share-files-across-test-cases}

여러 테스트에서 `file.*`, `params.json` 또는 `validate` 스크립트를 공유하는 경우 상대 심링크를 사용하는 것이 좋습니다. git을 통해 지원됩니다. 다른 파일이 있을 수 있으므로 공유 파일에 고유한 이름을 지정해야 합니다. 아래 예에서 테스트에서는 몇 개의 공유 파일과 자체 파일을 혼합하고 일치시킵니다.

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### 테스트 예상 오류 {#test-unexpected-errors}

오류 테스트 케이스는 예상 `rendition.*` 파일을 포함할 수 없으며 `params.json` 파일 내에 예상 `errorReason`을 정의해야 합니다.

테스트 케이스 구조 오류:

```json
<error_test_case>/
    file.jpg
    params.json
```

오류 이유가 있는 매개 변수 파일:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

[Asset compute 오류 이유](https://github.com/adobe/asset-compute-commons#error-reasons)의 전체 목록 및 설명을 참조하십시오.

## 사용자 지정 응용 프로그램 {#debug-custom-worker} 디버그

다음 단계에서는 Visual Studio 코드를 사용하여 사용자 지정 응용 프로그램을 디버깅하는 방법을 보여 줍니다. 라이브 로그, 히트 중단점, 코드 스루 및 모든 활성화와 동시에 로컬 코드 변경 사항을 실시간으로 다시 로드할 수 있습니다.

이러한 단계 중 대부분은 일반적으로 `aio`에 의해 자동화됩니다. [Firefly 문서](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)에서 응용 프로그램 디버깅 섹션을 참조하십시오. 현재는 아래 단계에 해결 방법이 포함되어 있습니다.

1. GitHub에서 최신 [wskdebug](https://github.com/apache/openwhisk-wskdebug) 및 선택적 [ngroup](https://www.npmjs.com/package/ngrok)을 설치합니다.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 사용자 설정 JSON 파일에 추가합니다. 이전 VS 코드 디버거를 계속 사용하고 있으며 새 디버그에는 wskdebug와 함께 [몇 가지 문제](https://github.com/apache/openwhisk-wskdebug/issues/74)가 있습니다.`"debug.javascript.usePreview": false`.
1. `aio app run`을(를) 통해 열려 있는 앱의 인스턴스를 모두 닫습니다.
1. `aio app deploy`을(를) 사용하여 최신 코드를 배포합니다.
1. `npx adobe-asset-compute devtool`을(를) 사용하여 Asset compute 장치 도구만 실행합니다. 열어 두어라.
1. VS 코드 편집기에서 아래 디버그 구성을 `launch.json`에 추가합니다.

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   `aio app deploy`의 출력에서 ACTION NAME을 가져옵니다. `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker` 같습니다.

1. 실행/디버그 구성에서 `wskdebug worker`을 선택하고 재생 아이콘을 누릅니다. **[!UICONTROL 디버그 콘솔]** 창에 **[!UICONTROL 활성화 준비]**&#x200B;가 표시될 때까지 기다립니다.

1. Devtool에서 **[!UICONTROL run]**&#x200B;을 클릭합니다. VS 코드 편집기에서 실행되는 동작을 볼 수 있으며 로그가 표시되기 시작합니다.

1. 코드에 중단점을 설정하고 다시 실행하면 히트합니다.

모든 코드 변경 사항은 실시간으로 로드되며 다음 정품 인증이 수행되면 즉시 적용됩니다.

>[!NOTE]
>
>사용자 지정 응용 프로그램의 각 요청에 대해 두 개의 활성화가 있습니다. 첫 번째 요청은 SDK 코드에서 비동기적으로 자신을 호출하는 웹 동작입니다. 두 번째 활성화는 코드를 히트하는 것입니다.
