---
title: "프로젝트 구조"
weight: 220
---
**Cloud9** 좌측 네비게이션 화면에서 생성된 프로젝트의 구조를 살펴 봅니다.
![CDK project structure](/images/project/structure.png)

다음은 이번 워크샵에서 주로 사용하게 될 몇가지 필수적인 파일들에 대한 설명 입니다.

> **lib/logging-wokrshop-stack.ts**: CDK 어플의 main stack을 정의하는 파일. 
>
> **bin/logging-wokrshop-stack.ts**:  CDK 어플의 entry point. lib/logging-wokrshop-stack.ts 에 정의된 stack을 로드 합니다.
>
> **package.json**: npm 모듈의 manifest 파일. app의 이름, version, dependency 등의 정보가 기록 되어 있다.
>
>**cdk.json**: toolkit이 app을 어떻게 실행할지 command를 지정 하는 곳. 이 워크샵의 경우 `npx ts-node bin/logging-workshop.ts`. 또한 context에 변수를 추가하여 stack에서 해당 context 변수를 사용할 수 있다.
