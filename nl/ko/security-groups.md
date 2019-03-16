---

copyright:
  years: 2018

lastupdated: "2018-12-12"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# VPC CLI 및 API에서 보안 그룹 사용

보안 그룹을 사용하면 IP 주소를 기반으로 하여 가상 서버 인스턴스(VSI)의 각 네트워크 인터페이스에 대한 필터링을 설정하는 규칙을 편리하게 적용할 수 있습니다.

기본적으로 보안 그룹은 모든 트래픽을 거부합니다. 규칙이 보안 그룹에 추가되면서 보안 그룹이 허용하는 트래픽을 정의합니다.

규칙은 _stateful_입니다. 즉, 허용된 트래픽에 대응하는 역 트래픽이 자동으로 허용됩니다. 예를 들어, 포트 80에서 인바운드 TCP 트래픽을 허용하는 규칙은 추가 규칙 없이도 포트 80에서 원래 호스트로 응답하는 아웃바운드 TCP 트래픽도 허용합니다.

보안 그룹의 범위는 단일 VPC입니다. 이 범위는 보안 그룹이 동일한 VPC 내의 VSI의 네트워크 인터페이스에 _한정하여_ 연결될 수 있음을 의미합니다.

VSI가 보안 그룹이 지정되지 않고 작성된 경우, VSI의 기본 네트워크 인터페이스가 해당 VSI의 VPC의 _기본_ 보안 그룹에 배치됩니다. 이 {{site.data.keyword.cloud}} VPC 릴리스에는 특정 트래픽을 허용하는 기본 보안 그룹이 정의되어 있습니다.

보안 그룹을 사용하여 VSI를 프로비저닝하는 방법을 포함하여 자세한 정보를 보려면 [보안 그룹 사용 예제 문서](security-group-usage-examples.html)를 참조하십시오.

### 기본 보안 그룹

각 VPC에는 다음을 허용하는 데 필요한 규칙이 있는 기본 그룹이 있습니다.

* 모든 그룹 멤버로부터의 인바운드 트래픽
* 모든 아웃바운드 트래픽

기본 보안 그룹의 규칙을 편집하면 편집된 해당 규칙이 그룹 내의 모든 현재 및 미래 서버에 적용됩니다.

기본 보안 그룹은 삭제될 수 없다는 점만 제외하면 기타 보안 그룹과 매우 유사합니다.

**참고:** Ping 및 SSH를 허용하기 위한 인바운드 규칙은 기본 보안 그룹에 자동으로 추가되지 않습니다. REST API 또는 `ibmcloud cli`를 사용하여 기본 보안 그룹의 규칙을 수정할 수 있습니다. 예를 들어, 다음과 같습니다.

```
# Add rules allowing SSH and PING

ibmcloud is security-group-rule-add <Security Group ID> inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add <Security Group ID> inbound icmp --icmp-type 8 --icmp-code 0

```

## 사용 가능한 API 및 CLI

보안 그룹에 사용 가능한 API에 대한 문서는 [API 참조서](https://{DomainName}/apidocs/rias)에서 찾을 수 있습니다.
보안 그룹에 대한 작업에 사용 가능한 CLI 명령에 대한 문서는 [CLI 참조서](https://{DomainName}/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html)에서 찾을 수 있습니다.

이러한 엔드포인트는 기본 보안 그룹을 포함하여, VPC의 보안 그룹을 관리하는 데 사용될 수 있습니다.

API 호출을 사용하려면 REST 클라이언트의 일부 양식을 사용해야 합니다. 예를 들어, `curl`을 사용하여 모든 현재 보안 그룹을 검색할 수 있습니다.

```bash
curl -X GET $rias_endpoint/v1/security_groups -H "Authorization: $iam_token"
```

## 보안 그룹 데모 예제

다음 예제에서는 명령행 인터페이스(CLI)를 사용하여 보안 그룹을 사용하도록 설정하고 VSI를 작성합니다. 시나리오는 다음과 같습니다.

![이미지3](/images/security-groups-schematic.png)

그림에서는 **SG4**라는 이름의 VSI에 내부 VPC 주소 `10.0.0.5` 외에 유동 IP `169.60.208.144`가 지정되어 있음을 볼 수 있으며 따라서 공용 인터넷에 연결할 수 있습니다. VSI **SG4**에 지정된 보안 그룹의 이름은 "demosg"입니다.

이름이 **SG8**로 지정된 VSI는 사설 IP 주소를 사용하여 VPC에 대해 내부적으로만 사용됩니다. VSI **SG8**에 지정된 보안 그룹의 이름은 "my_vpc_sg"입니다. 이러한 VSI 둘 다 `sgvpc`라는 이름의 VPC 및 동일한 서브넷 `10.0.0.0/28`에 있으므로 서로 통신할 수 있습니다.

## 예제 코드

다음 예제 코드는 기본 기능인 SSH, PING 및 아웃바운드 TCP를 포함하도록 "my_vpc_sg"에 대한 보안 그룹 규칙을 설정하는 방법을 설명합니다.

`ibmcloud is sgc` 명령으로 보안 그룹을 먼저 작성한 다음 새로 작성한 이 보안 그룹을 포함할 VSI를 작성해야 합니다. 기본적으로 VSI 작성 명령의 끝에 (방금 작성한) 보안 그룹의 ID를 지정하는 매개변수만 추가하면 됩니다. 아래 예제에서는 보안 그룹 ID가 `2d364f0a-a870-42c3-a554-000000632953`입니다.

이 예제 코드에서는 몇 단계를 건너뜁니다. 아래에서 자세한 정보를 찾을 수 있습니다.

 * VPC 및 서브넷 작성에 대한 지시사항은 [예제 코드 문서](example-code.html#example-code-tutorial)에 나와 있습니다.

이 예제 CLI 코드에서 명령을 복사하여 붙여넣는 방법으로 보안 그룹이 있는 VSI 작성을 시작할 수 있습니다. 이 샘플 코드에서 시스템 응답은 모두 표시되지 않습니다. _VPC_, _서브넷_, _이미지_ 및 _키_에 대한 올바른 리소스 ID 및 올바른 _보안 그룹 ID 번호_로 명령을 업데이트해야 합니다.

**1단계. 보안 그룹 "my_vpc_sg" 작성**

```
Format: ibmcloud is security-group-create <Security Group Name> <VPC ID>
Example: ibmcloud is security-group-create my_vpc_sg e636a5de-391d-456d-badb-9a9570259a51
```
{: codeblock}

응답에는 새로 작성한 SG ID: **2d364f0a-a870-42c3-a554-000000632953**(이 경우, 사용자의 ID는 다를 수 있음)이 포함됩니다. 나중에 사용할 수 있도록 변수 `sg`에 저장하십시오.

**2단계. SSH, PING 및 아웃바운드 TCP를 허용하는 규칙 추가**

```
ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
ibmcloud is security-group-rule-add $sg outbound tcp

```
{: codeblock}

**3단계. 새로 작성한 SG를 사용하여 VSI 작성**

```
Format: ibmcloud is instance-create <Instance Name> <VPC ID> <Zone> <Profile> <Subnet ID> <PORT_SPEED> --image <Image ID> --keys <Key ID> --security-groups <Security Group ID>

Example: ibmcloud is instance-create test-instance 4063e74c-eb1d-4994-ab3d-2c3f1fe64416  us-south-2 B1_1X2X25 5ac380b4-ad2f-429d-97ea-0a7d3bb4930e 1000 --image 7eb4e35b-4257-56f8-d7da-326d85452591 --keys 329d8a9f-2f5e-49c8-9777-523addd67734 --security-groups $sg
```
{: codeblock}

## 명령 목록 치트 시트

보안 그룹에 대해 사용 가능한 VPC CLI 명령의 전체 목록을 보려면 다음을 입력하십시오.

```
ibmcloud is help | grep sg
```

보안 그룹 및 규칙을 포함한 메타데이터를 보려면 다음을 입력하십시오(앞의 예의 경우).

```
ibmcloud is sg $sg
```

**참고:** 일반적인 경우, 사용자의 VSI의 보안 그룹에 대한 올바른 보안 그룹 ID로 대체하십시오.

보안 그룹 규칙을 추가하려면 보안 그룹에 PING 인바운드 규칙을 추가하는 다음 예제 명령을 참조하십시오.

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: codeblock}

## FAQ
이 절에서는 IBM Cloud VPC용 보안 그룹에 대해 자주 묻는 몇 가지 질문에 대한 답변을 소개합니다.

### 보안 그룹에 대한 할당량이 무엇입니까?

보안 그룹 및 연관된 규칙에 대해 몇 가지 계정 기반 할당량(한계)이 있습니다.

|리소스  |할당량|
|--------|-----|
|네트워크 인터페이스당 보안 그룹|5|
|계정당 보안 그룹               |500|
|보안 그룹당 규칙               |50|
|보안 그룹당 원격 규칙          |5|
|보안 그룹당 네트워크 인터페이스|100|
