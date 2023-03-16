---
title: 테스트 및 디버그 [!DNL Asset Compute Service] 사용자 지정 애플리케이션
description: 테스트 및 디버그 [!DNL Asset Compute Service] 사용자 지정 애플리케이션입니다.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# 사용자 지정 애플리케이션 테스트 및 디버그 {#test-debug-custom-worker}

## 사용자 지정 응용 프로그램에 대한 단위 테스트 실행 {#test-custom-worker}

설치 [Docker Desktop](https://www.docker.com/get-started) 사용자 시스템에 있는 사용자 정의 작업자를 테스트하려면 응용 프로그램의 루트에서 다음 명령을 실행합니다.

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

이렇게 하면 아래에서 설명하는 대로 프로젝트에서 Asset compute 응용 프로그램 작업에 대한 사용자 지정 단위 테스트 프레임워크를 실행합니다. 이 파일은 의 구성을 통해 연결됩니다 `package.json` 파일. Jest와 같은 JavaScript 단위 테스트가 있을 수도 있습니다. `aio app test` 는 둘 다 실행합니다.

다음 [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 플러그인은 사용자 지정 애플리케이션 앱에 개발 종속성으로 포함되어 있으므로 빌드/테스트 시스템에 설치할 필요가 없습니다.

### 애플리케이션 단위 테스트 프레임워크 {#unit-test-framework}

asset compute 응용 프로그램 단위 테스트 프레임워크는 코드를 작성하지 않고도 응용 프로그램을 테스트할 수 있다. 응용 프로그램의 렌디션 파일 원칙에 대한 소스를 사용합니다. 테스트 소스 파일, 선택적 매개 변수, 예상 표현물 및 사용자 지정 유효성 검사 스크립트를 사용한 테스트 사례를 정의하려면 특정 파일 및 폴더 구조를 설정해야 합니다. 기본적으로 표현물은 바이트 평등과 비교됩니다. 또한 간단한 JSON 파일을 사용하여 외부 HTTP 서비스를 쉽게 조롱할 수 있습니다.

### 테스트 추가 {#add-tests}

테스트는 `test` 폴더의 루트 수준에 있는 폴더 [!DNL Adobe I/O] 프로젝트. 각 애플리케이션에 대한 테스트 케이스는 경로에 있어야 합니다 `test/asset-compute/<worker-name>`, 각 테스트 사례에 대해 한 개의 폴더가 제공됩니다.

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

한번 보세요 [사용자 지정 애플리케이션 예](https://github.com/adobe/asset-compute-example-workers/) 참조하십시오. 아래는 자세한 참조입니다.

### 테스트 출력 {#test-output}

사용자 정의 애플리케이션의 로그를 포함한 자세한 테스트 출력은 `build` 폴더에 있는에 설명된 대로 Adobe Developer App Builder 앱의 루트에 있는 폴더 `aio app test` 출력.

### 외부 서비스 모의 {#mock-external-services}

다음을 정의하여 작업에서 외부 서비스 호출을 모의할 수 있습니다 `mock-<HOST_NAME>.json` 테스트 사례에 있는 파일로서, 여기서 HOST_NAME은 모의 대상 호스트입니다. 예제 사용 사례는 S3를 별도의 호출로 만드는 애플리케이션입니다. 새 테스트 구조는 다음과 같습니다.

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

mock 파일은 JSON 형식의 http 응답입니다. 자세한 내용은 [이 설명서](https://www.mock-server.com/mock_server/creating_expectations.html). 모의할 호스트 이름이 여러 개 있는 경우 여러 개 정의 `mock-<mocked-host>.json` 파일. 아래는 다음에 대한 샘플 mock 파일입니다. `google.com` 명명된 이름 `mock-google.com.json`:

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

예 `worker-animal-pictures` 다음 포함 [모의 파일](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) Wikimedia 서비스의 경우 이 서비스와 상호 작용합니다.

#### 테스트 사례 간에 파일 공유 {#share-files-across-test-cases}

공유하는 경우 상대 symlink 를 사용하는 것이 좋습니다 `file.*`, `params.json` 또는 `validate` 여러 테스트 간에 스크립트를 작성합니다. Git에서 지원됩니다. 다른 이름을 사용할 수 있으므로 공유 파일에 고유한 이름을 지정해야 합니다. 아래 테스트에서는 몇 개의 공유 파일과 자체 파일을 혼합하여 매칭합니다.

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

### 예상 오류 테스트 {#test-unexpected-errors}

오류 테스트 케이스에 예상된 오류가 없어야 합니다. `rendition.*` 파일 및에 필요한 를 정의해야 합니다. `errorReason` 내부 `params.json` 파일.

>[!NOTE]
>
>테스트 사례에 예상대로 포함되지 않으면 `rendition.*` 파일 및 이(가) 예상한 `errorReason` 내부 `params.json` 파일, 임의의 `errorReason`.

오류 테스트 사례 구조:

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

전체 목록 및 설명 참조 [asset compute 오류 이유](https://github.com/adobe/asset-compute-commons#error-reasons).

## 사용자 지정 애플리케이션 디버깅 {#debug-custom-worker}

다음 단계는 Visual Studio 코드를 사용하여 사용자 지정 응용 프로그램을 디버깅하는 방법을 보여 줍니다. 라이브 로그, 히트 중단점 및 단계별 코드를 확인할 수 있을 뿐만 아니라 모든 활성화 시 로컬 코드 변경 사항을 실시간으로 재로드할 수 있습니다.

이러한 절차 중 대부분은 일반적으로 `aio` 즉시 사용 가능한 경우 [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/getting_started/first_app). 현재, 아래 단계에는 해결 방법이 포함되어 있습니다.

1. 최신 설치 [wskdebug](https://github.com/apache/openwhisk-wskdebug) GitHub 및 옵션 [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 사용자 설정 JSON 파일에 를 추가합니다. 이전 VS 코드 디버거를 계속 사용하며, 새 API는 [일부 문제](https://github.com/apache/openwhisk-wskdebug/issues/74) wskdebug를 사용하여 다음을 수행합니다. `"debug.javascript.usePreview": false`.
1. 를 통해 열려 있는 앱의 인스턴스를 모두 닫습니다. `aio app run`.
1. 를 사용하여 최신 코드 배포 `aio app deploy`.
1. 를 사용하여 Asset compute 장치 도구만 실행 `aio asset-compute devtool`. 계속 열어
1. VS 코드 편집기에서 다음 디버그 구성을 `launch.json`:

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

   가져오기 `ACTION NAME` 의 결과로부터 `aio app deploy`.

1. 선택 `wskdebug worker` run/debug 구성에서 play 아이콘을 누릅니다. 표시될 때까지 기다립니다. **[!UICONTROL 활성화 준비]** 에서 **[!UICONTROL 디버그 콘솔]** 창을 엽니다.

1. 클릭 **[!UICONTROL run]** 를 클릭합니다. VS 코드 편집기에서 실행되는 작업과 로그가 표시됩니다.

1. 코드에 중단점을 설정하고, 다시 실행하면 됩니다.

모든 코드 변경 사항은 실시간으로 로드되며, 다음 활성화가 발생하는 즉시 적용됩니다.

>[!NOTE]
>
>사용자 지정 애플리케이션의 각 요청에 대해 두 개의 활동이 표시됩니다. 첫 번째 요청은 SDK 코드에서 비동기적으로 자신을 호출하는 웹 작업입니다. 두 번째 활성화는 코드를 연결하는 것입니다.
