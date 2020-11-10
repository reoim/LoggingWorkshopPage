---
title: "cdk init"
weight: 210
---

***
 


## 프로젝트 폴더 생성
다음 명령어로 프로젝트 폴더를 생성하고 해당 폴더로 이동 합니다.
```
mkdir logging-workshop && cd logging-workshop
```

&nbsp;
## cdk init
`cdk init` 명령어로 기본 TypeScript cdk 프로젝트를 생성 합니다.
```
cdk init app --language typescript
```
다음과 같은 메세지와 함께 프로젝트가 생성 됩니다. git repository 관련 warning 메세지들은 무시하셔도 됩니다.
```terminal
Applying project template app for typescript
# Welcome to your CDK TypeScript project!

This is a blank project for TypeScript development with CDK.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

## Useful commands

 * `npm run build`   compile typescript to js
 * `npm run watch`   watch for changes and compile
 * `npm run test`    perform the jest unit tests
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk synth`       emits the synthesized CloudFormation template

Initializing a new git repository...
Executing npm install...
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
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
워크샵이 진행 되는 동안 **npm run watch** 명령어가 실행 된 terminal 창을 열어둔 채로 유지 합니다. 해당 terminal 창을 닫으면 TypeScript가 자동 컴파일 되지 않습니다.
{{% /notice %}}

&nbsp;
## 프로젝트 구조
![CDK project structure](/images/project/structure.png)

