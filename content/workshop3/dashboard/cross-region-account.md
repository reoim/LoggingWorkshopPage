---
title: "교차 계정/리전 모니터링"
weight: 440
pre: "<b>6-4.d. </b>"
---
***

Amazon CloudWatch 대시보드를 사용하여 여러 리전/계정에 분산되어 있는 리소스들에 대한 지표 및 경보를 모니터링 할 수 있습니다.

이 페이지에서는 교차 계정/리전 대시보드를 활성화합니다.

{{% notice info %}}
이 실습을 시작하기 위해서는 총 두개의 AWS 계정, 또는 Event Engine 계정이 필요합니다. 편의상 모니터링 용도 첫번째 계정 ID를 `111111111111`, 데이터를 공유하는 두번째 계정 ID를 `222222222222` 라고 가정합니다.
{{% /notice %}}

## CloudWatch 데이터 공유

모니터링 계정(ID: 111111111111)에서 로그아웃 한 뒤 두번째 계정(ID: 222222222222)으로 로그인합니다.

[CloudWatch 콘솔](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#)에서 `Setting` 메뉴를 선택 후 `Cross-account cross-region`의 `configure` 버튼을 클릭합니다.

![cross](/images/workshop3/share1.png)

&nbsp;

`Share data` 버튼을 클릭합니다.

![cross](/images/workshop3/share2.png)

&nbsp;

`Account ID` 필드에 모니터링 계정 ID를 입력하고 `Permissions`는 아래와 같이 `Full read-only..`로 변경합니다.

그 다음 `Launch CloudFormation template`를 클릭합니다.

![cross](/images/workshop3/share-data.png)

&nbsp;

`Confirm`을 입력 후 `Laucnh template`를 클릭합니다. 

![cross](/images/workshop3/share-data2.png)

모니터링 계정 ID가 맞는지 확인 후 스택을 생성합니다.

![cross](/images/workshop3/share-data3.png)

정상적으로 `CloudWatch-CrossAccountSharingRole`이 생성된 것을 확인합니다.

![cross](/images/workshop3/share-data4.png)

## 교차 계정 View

두번째 계정에서 로그아웃 한 뒤 모니터링 계정으로 로그인합니다.

[CloudWatch 콘솔](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#)에서 `Setting` 메뉴를 선택 후 `Cross-account cross-region`의 `configure` 버튼을 클릭합니다.

![cross](/images/workshop3/share1.png)

&nbsp;

`View cross-account cross-region`의 `enable` 버튼을 클릭합니다.

![cross](/images/workshop3/share3.png)

&nbsp;

`Custom account selector`를 선택한 뒤 `<account id> <account label>` 형식에 맞춰 모니터링 계정과 두번째 계정 ID와 라벨을 아래와 같이 입력합니다.

![cross](/images/workshop3/view1.png)

&nbsp;

정상적으로 설정 되었습니다.

![cross](/images/workshop3/view2.png)



## 단일뷰로 교차 계정, 교차 리전 모니터링

[CloudWatch 콘솔](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#) 상단에 다음과 같은 drop box 필드가 생긴것을 확인할 수 있습니다.

![cross](/images/workshop3/cross1.png)

&nbsp;

두번째 계정과 리전을 선택한 뒤 `View account`를 클릭합니다.

![cross](/images/workshop3/cross2.png)

모니터링 계정에서 다른 계정/리전의 CloudWatch 지표와 대시보드, 로그 위젯, 알람을 확인할 수 있습니다.

![cross](/images/workshop3/cross3.png)

![cross](/images/workshop3/cross4.png)

## 다른 계정으로 전환

`View account` 기능으로는 CloudWatch 지표와 대시보드, 로그 위젯, 알람만 접근이 가능했습니다. 로그 그룹이나 `Logs insight`등은 사용이 불가능합니다.

`Switch to this account`를 클릭하여 콘솔에서 계정을 전환하면 Read only 권한으로 모든 서비스들에 접근이 가능합니다.

![cross](/images/workshop3/cross5.png)

&nbsp;

`Display Name`을 입력하고 `Switch Role`을 클릭합니다.

![cross](/images/workshop3/cross6.png)

&nbsp;

Read only 권한으로 다른 계정 콘솔에 접속한 것을 확인할 수 있습니다.

로그 그룹 및 `Logs Insight`는 물론, `CloudWatch`가 아닌 다른 서비스들에도 읽기 권한으로 접근이 가능합니다. (읽기 권한 이상이 필요한 서비스에는 접근이 되지 않습니다.)

![cross](/images/workshop3/cross7.png)

&nbsp;

모니터링 계정으로 다시 돌아가고 싶다면 다음과 같이 `Back to (my account)`를 클릭하여 돌아갈 수 있습니다.

![cross](/images/workshop3/cross8.png)
