---
title: VPC, 인스턴스 로그
weight: 400
pre: "<b>4-4. </b>"
---

이번 실습에서는 간단한 데모 워크로드를 생성하기 위해 `CDK`를 이용하여 `VPC`를 생성하고 `Apache 웹서버 인스턴스`를 배포합니다.

VPC flow 로그를 활성화하여 reject 로그들을 `CloudWatch Log` 로그 그룹과 로그 버킷으로 전송하는 구성을 만들어볼 것입니다.

또한 `CloudWatch Agent`를 설치하여 인스턴스 세부 지표 및 Apache 웹서버 로그들을 로그 그룹에 수집할 것입니다.

&nbsp;

## 중앙 로그 버킷 공유

VPC flow 로그는 `S3`에 직접 내보낼수 있습니다.

먼저 중앙 로그 버킷 object를 props로 전달 받기 위해 `lib/webserver-stack.ts` 파일을 열어 다음과 같이 수정합니다.

`LogBucketStack`에서 정의한 `interface`를 `import` 합니다.

```typescript
import { BucketProps } from './log-bucket-stack';
```

`constructor`의 `props`를 수정합니다.
```typescript
constructor(scope: cdk.Construct, id: string, props: BucketProps)
```

수정된 코드는 다음과 같습니다.

```typescript
import * as cdk from '@aws-cdk/core';
import { BucketProps } from './log-bucket-stack';

export class WebServerStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props: BucketProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```


