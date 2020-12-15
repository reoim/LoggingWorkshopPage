---
title: 베이스 프로젝트 클론
weight: 220
pre: "<b>5-2.b. </b>"
---

**Cloud9** 터미널 창에서 다음 명령어로 베이스 프로젝트를 클론 합니다.

```
cd ~/environment
git clone https://github.com/reoim/centralized-logging-skeleton.git
```

### 프로젝트 구조
베이스 프로젝트 구조는 다음과 같습니다.
![Base project structure](/images/workshop1/structure.png)

* `/lib`: 이 CDK 앱에서 우리가 작성할 Stack 파일들이 위치 합니다.

* `/bin/centralized-logging-stack.ts`:  CDK 어플의 entry point. **/lib**에 생성한 스택을 실제로 배포하려면 여기에 해당 스택을 추가해줘야 합니다.

* `/resources`: 이 워크샵에서 필요한 몇가지 리소스들 입니다. CloudWatch agent 설정 파일, 샘플용 Lambda function, Log destination 설정 파일, userdata script 등이 정의 되어 있습니다.

* `cdk.json`: toolkit이 app을 어떻게 실행할지 command를 지정 및 context value를 추가하는 곳. 

* `package.json`: npm 모듈의 manifest 파일. app의 이름, version, dependency 등의 정보가 기록 되어 있습니다.

&nbsp;

### 의존성 패키지 설치
워크샵에 필요한 패키지들은 `package.json` 에 정의 되어있습니다.

다음 명령어로 클론 된 베이스 프로젝트 폴더로 이동 후, 필요한 패키지 모듈들을 설치 합니다.

```
cd ~/environment/centralized-logging-skeleton
npm i
```

&nbsp;
## npm run watch
TypeScript는 코드를 수정, 저장시 JavaScript로 컴파일을 해줘야 합니다. 

매번 `npm run build` 명령어로 컴파일 하기에는 번거로우니 Terminal 창에 `npm run watch` 명령어를 실행하여 TypeScript 파일 변경시 자동으로 컴파일 되도록 합니다.
```
npm run watch
```

명령어를 실행하면 터미널 화면에 다음과 같은 메세지가 출력 될 것 입니다.
```terminal
[2:29:53 AM] File change detected. Starting incremental compilation...

[2:29:53 AM] Found 0 errors. Watching for file changes.
``` 

{{% notice warning %}}
워크샵이 진행 되는 동안 `npm run watch` 명령어가 실행 된 `terminal 창을 열어둔 채로 유지` 합니다. 해당 terminal 창을 닫으면 TypeScript가 자동 컴파일 되지 않습니다.
{{% /notice %}}


&nbsp;
## Bootstrap CDK
CDK 앱을 처음 배포한다면 CDK toolkit에 필요한 리소스들을 먼저 배포 해야 합니다. 새 터미널 창을 열어 다음 명령어로 CDK toolkit stack을 배포 합니다.
```
cd ~/environment/centralized-logging-skeleton
cdk bootstrap
```

다음과 같이 CDK Toolkit에 필요한 리소스가 배포 됩니다.
```term
 ⏳  Bootstrapping environment aws://222222222222/us-east-2...
CDKToolkit: creating CloudFormation changeset...
[██████████████████████████████████████████████████████████] (3/3)
```