---
title: 베이스 프로젝트 클론
weight: 100
pre: "<b>4-1. </b>"
---

터미널 창에서 다음 명령어로 베이스 프로젝트를 클론 합니다.

```
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