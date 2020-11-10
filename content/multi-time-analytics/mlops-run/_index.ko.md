---
date: 2020-09-2
title: "MLOps 파이프라인 동작 확인"
chapter: false
weight: 240
pre: "<b>4-4 </b>"
---


[Step Function 콘솔의 State machines 메뉴](https://ap-northeast-2.console.aws.amazon.com/states/home?region=ap-northeast-2#/statemachines)로 이동하여 **DeployStateMachine**로 시작하는 State machine을 클릭 합니다.
![State Machine](/images/mlops/state-machine.png)
현재 실행되고 있는 **workflow**를 확인 하기 위해 **excution**을 클릭 합니다.
![Click Excution](/images/mlops/click-excution.png)
다음과 같이 현재 실행중인 **Step function workflow**를 확인 할 수 있습니다.
![WorkFlow](/images/mlops/workflow.png)
완료 된 step은 녹색으로 표시되며 진행 중인 step은 하늘색으로 표시 됩니다. 이 작업은 약 2시간 정도까지 소요 될 수 있습니다.

모든 작업이 완료되면 다음과 같이 workflow가 녹색으로 변한것을 확인 할 수 있습니다.
![WorkFlow](/images/mlops/workflow2.png)