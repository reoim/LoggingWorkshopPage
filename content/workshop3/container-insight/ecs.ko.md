---
title: "ECS Container Insight 활용"
weight: 210
---
***

이 페이지에서는 [실습 1, CDK로 로깅 인프라 구축 - ECS 로그](../../../workshop1/ecs) 실습에서 활성화했던 Container Insight에 대해서 알아 봅니다.

Container Insight 는 [`AWS 콘솔`이나 `AWS CLI`를 통해서도 활성화](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-container-insights.html) 할 수 있지만 우리는 앞선 실습에서 CDK로 다음과 같이 클러스터를 정의하여 활성화 했습니다.

```typescript
    // Create an ECS cluster
    const cluster = new ecs.Cluster(this, 'Cluster', { 
      vpc, 
      containerInsights: true
    });
```

[Container Insight 콘솔](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#container-insights:infrastructure)로 이동하여 Cluster 리소스를 확인 합니다.

![resource](/images/workshop3/ci-resource.png)

`Map view`를 클릭하면 클러스터의 리소스들을 트리 형식으로 보실수 있습니다. 리소스를 클릭하면 콘솔 하단부에 몇가지 주요 메트릭을 확인할 수 있습니다.

![dashboard](/images/workshop3/ci-tree.png)

`Performance Monitoring` 메뉴를 ECS 리소스들에 대한 여러가지 메트릭을 확인할 수 있습니다.

![dashboard](/images/workshop3/container.gif)

`ECS Cluster, ECS Instances, ECS Services, ECS Tasks` 를 각각 선택해서 메트릭들을 확인 해봅니다.

![dashboard](/images/workshop3/container2.png)