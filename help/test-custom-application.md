---
title: 테스트 및 디버그 [!DNL Asset Compute Service] 사용자 정의 애플리케이션
description: 테스트 및 디버그 [!DNL Asset Compute Service] 사용자 지정 응용 프로그램입니다.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# 사용자 지정 응용 프로그램 테스트 및 디버그 {#test-debug-custom-worker}

## 사용자 정의 애플리케이션에 대한 단위 테스트 실행 {#test-custom-worker}

설치 [Docker 데스크탑](https://www.docker.com/get-started) 컴퓨터에 있습니다. 사용자 지정 작업자를 테스트하려면 응용 프로그램의 루트에서 다음 명령을 실행합니다.

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

이렇게 하면 아래 설명된 대로 프로젝트의 Asset compute 애플리케이션 작업에 대한 사용자 정의 단위 테스트 프레임워크가 실행됩니다. 의 구성을 통해 연결됩니다. `package.json` 파일. Jest와 같은 JavaScript 단위 테스트가 있을 수도 있습니다. `aio app test` 는 둘 다 실행합니다.

다음 [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 플러그인은 사용자 지정 애플리케이션 앱에 개발 종속성으로 포함되므로 빌드/테스트 시스템에 설치할 필요가 없습니다.

### 애플리케이션 단위 테스트 프레임워크 {#unit-test-framework}

asset compute 애플리케이션 단위 테스트 프레임워크를 사용하면 코드를 작성하지 않고도 애플리케이션을 테스트할 수 있습니다. 응용 프로그램의 소스 대 렌디션 파일 원칙에 의존합니다. 테스트 소스 파일, 선택적 매개 변수, 예상 표현물 및 사용자 정의 유효성 검사 스크립트를 사용하여 테스트 사례를 정의하기 위해 특정 파일 및 폴더 구조를 설정해야 합니다. 기본적으로 렌디션은 바이트 동일성과 비교됩니다. 또한 외부 HTTP 서비스는 간단한 JSON 파일을 사용하여 쉽게 조롱할 수 있습니다.

### 테스트 추가 {#add-tests}

내부 테스트가 필요합니다. `test` 폴더의 루트 수준에 있는 폴더 [!DNL Adobe I/O] 프로젝트. 각 애플리케이션의 테스트 사례는 경로에 있어야 합니다. `test/asset-compute/<worker-name>`, 각 테스트 사례에 대해 하나의 폴더 포함:

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

다음을 살펴보십시오 [사용자 정의 애플리케이션 예](https://github.com/adobe/asset-compute-example-workers/) 예를 들어, 아래는 자세한 참조입니다.

### 출력 테스트 {#test-output}

사용자 정의 애플리케이션의 로그를 포함한 자세한 테스트 출력은 `build` 폴더에 나와 있는 대로 Adobe Developer App Builder 앱의 루트에 있는 폴더 `aio app test` 출력.

### 모의 외부 서비스 {#mock-external-services}

다음을 정의하여 작업에서 외부 서비스 호출을 모의할 수 있습니다. `mock-<HOST_NAME>.json` 테스트 사례에 있는 파일입니다. 여기서 HOST_NAME은 모의할 호스트입니다. 사용 사례의 예는 S3에 대해 별도의 호출을 수행하는 애플리케이션입니다. 새 테스트 구조는 다음과 같습니다.

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

모의 파일은 JSON 형식의 http 응답입니다. 자세한 내용은 [이 설명서](https://www.mock-server.com/mock_server/creating_expectations.html). 모의할 호스트 이름이 여러 개 있는 경우 여러 을 정의합니다 `mock-<mocked-host>.json` 파일. 아래는 의 샘플 모의 파일입니다. `google.com` 명명된 `mock-google.com.json`:

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

예 `worker-animal-pictures` 다음 포함: [모의 파일](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) 위키미디어 서비스의 경우 와 상호 작용합니다.

#### 여러 테스트 사례에서 파일 공유 {#share-files-across-test-cases}

를 공유하는 경우 상대 심볼릭 링크를 사용하는 것이 좋습니다 `file.*`, `params.json` 또는 `validate` 여러 테스트에 걸친 스크립트. git에서 지원됩니다. 다른 이름이 있을 수 있으므로 공유 파일에 고유한 이름을 지정해야 합니다. 아래 예에서 테스트는 몇 개의 공유 파일과 자체 파일을 혼합하여 일치시키고 있습니다.

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

오류 테스트 사례에 예상한 내용이 포함되어서는 안 됩니다. `rendition.*` 파일 및 예상 값 정의 `errorReason` 의 내부 `params.json` 파일.

>[!NOTE]
>
>테스트 사례에 예상 값이 포함되지 않은 경우 `rendition.*` 파일 및 예상 값을 정의하지 않음 `errorReason` 의 내부 `params.json` 파일, 다음과 같은 오류 경우로 가정 `errorReason`.

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

의 전체 목록 및 설명 보기 [Asset compute 오류 원인](https://github.com/adobe/asset-compute-commons#error-reasons).

## 사용자 지정 응용 프로그램 디버깅 {#debug-custom-worker}

다음 단계에서는 Visual Studio 코드를 사용하여 사용자 지정 응용 프로그램을 디버깅하는 방법을 보여 줍니다. 이를 통해 라이브 로그, 히트 중단점 및 코드를 단계별로 볼 수 있을 뿐만 아니라 활성화 시마다 로컬 코드 변경 사항을 실시간으로 다시 로드할 수 있습니다.

이러한 단계의 대부분은 일반적으로 다음을 통해 자동화됩니다. `aio` 기본적으로 의 응용 프로그램 디버깅 섹션을 참조하십시오. [Adobe Developer App Builder 설명서](https://developer.adobe.com/app-builder/docs/getting_started/first_app). 현재 아래 단계에는 해결 방법이 포함되어 있습니다.

1. 최신 버전 설치 [wskdebug](https://github.com/apache/openwhisk-wskdebug) GitHub 및 선택 사항에서 [응록](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 를 사용자 설정 JSON 파일에 추가합니다. 이전 VS 코드 디버거를 계속 사용하며, 새 디버거에는 [일부 문제](https://github.com/apache/openwhisk-wskdebug/issues/74) wskdebug 사용: `"debug.javascript.usePreview": false`.
1. 를 통해 열려 있는 앱의 모든 인스턴스 닫기 `aio app run`.
1. 를 사용하여 최신 코드 배포 `aio app deploy`.
1. 다음을 사용하여 Asset compute 개발자 도구만 실행 `aio asset-compute devtool`. 열어 두십시오.
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

   가져오기 `ACTION NAME` 의 출력으로부터 `aio app deploy`.

1. 선택 `wskdebug worker` 실행/디버그 구성에서 재생 아이콘을 누릅니다. 표시될 때까지 시작 대기 **[!UICONTROL 활성화 준비]** 다음에서 **[!UICONTROL 디버그 콘솔]** 창.

1. 클릭 **[!UICONTROL 실행]** 개발자 도구에서. VS 코드 편집기에서 실행되는 작업을 볼 수 있으며 로그가 표시되기 시작합니다.

1. 코드에서 중단점을 설정하고 다시 실행하면 됩니다.

모든 코드 변경 사항은 실시간으로 로드되며 다음 활성화가 발생하는 즉시 유효합니다.

>[!NOTE]
>
>사용자 지정 응용 프로그램의 각 요청에 대해 두 개의 활성화가 존재합니다. 첫 번째 요청은 SDK 코드에서 자신을 비동기적으로 호출하는 웹 작업입니다. 두 번째 활성화는 코드를 히트하는 활성화입니다.
