---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-14"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 원격 FortiGate 피어와의 보안 연결 작성

이 문서는 FortiGate 300C, 펌웨어 버전 v5.2.13, build762(GA)를 기반으로 합니다.

## 예제 단계
원격 FortiGate 피어에 대한 연결의 토폴로지는 두 VPC 사이에 VPN 연결을 작성하는 것과 유사합니다. 단, 한 측이 FortiGate 300C 장치로 대체됩니다.

![여기에 이미지 설명 입력](./images/vpc-vpn-fg-figure.png)

### 원격 Fortigate 피어와의 보안 연결 작성 방법

**VPN \> IPsec \> 터널**로 이동하여 새 사용자 정의 터널을 작성하거나 기존 터널을 편집하십시오.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

FortiGate 장치가 원격 VPN 피어로부터 연결 요청을 수신하면 IPsec 1단계(Phase) 매개변수를 사용하여 보안 연결을 설정하고 해당 VPN 피어를 인증합니다. 그런 다음, 보안 정책이 연결을 허용하면 FortiGate 장치가 IPsec 2단계(Phase) 매개변수를 사용하여 터널을 설정하고 IPsec 보안 정책을 적용합니다. 키 관리, 인증 및 보안 서비스가 IKE 프로토콜을 통해 동적으로 협상됩니다.

**해당 기능을 지원하려면 FortiGate 장치에 의해 다음과 같은 일반 구성 단계가 수행되어야 합니다.**

* FortiGate 장치가 원격 피어를 인증하고 보안 연결을 설정하는 데 필요한 1단계(Phase) 매개변수를 정의하십시오.

* FortiGate 장치가 원격 피어를 사용하여 VPN 터널을 작성하는 데 필요한 2단계(Phase) 매개변수를 정의하십시오.

* IP 소스 및 목적지 주소 간의 허용되는 트래픽 방향 및 허용되는 서비스를 제어하는 데 필요한 보안 정책을 작성하십시오.

기본적으로 몇 가지 일반적인 1단계(Phase) 및 2단계(Phase) 매개변수는 설정되어 있습니다.

### IBM Cloud VPC VPN에 대한 구성

IBM Cloud VPC의 VPN 기능에 연결하기 위해 권장되는 구성은 다음과 같습니다.

1. 인증에서 IKEv2 선택
2. 1단계(Phase) 제안에서 `DH-group 2`를 사용하도록 설정
3. 1단계(Phase) 제안에서 `lifetime = 36000`을 설정
4. 2단계(Phase) 제안에서 PFS를 사용하지 않도록 설정
5. 2단계(Phase) 제안에서 `lifetime = 10800` 설정
6. 서브넷의 정보를 2단계(Phase) 제안에 입력
7. 나머지 매개변수는 기본값을 유지합니다.

![여기에 이미지 설명 입력](./images/vpc-vpn-fg-network.JPG)

### 네트워크 매개변수

![여기에 이미지 설명 입력](./images/vpc-vpn-fg-authentication.JPG)

### 인증 매개변수

![여기에 이미지 설명 입력](./images/vpc-vpn-fg-phase1.JPG)

### 1단계(Phase) 매개변수

![여기에 이미지 설명 입력](./images/vpc-vpn-fg-phase2.JPG)

### 2단계(Phase) 매개변수

## 로컬 IBM VPC와의 보안 연결 작성

보안 연결을 작성하기 위해 VPC 내에서 VPN 연결을 작성할 것이며 이는 2 VPC 예제와 유사합니다.

* `local_cidrs`를 VPC의 서브넷 값으로 설정하고 `peer_cidrs`를 FortiGate의 서브넷 값으로 설정한 상태로 VPC 및 FortiGate 장치 사이의 VPN 연결과 함께 VPC 서브넷에 VPN 게이트웨이를 작성하십시오.

**참고:** 게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다.

![여기에 이미지 설명 입력](images/vpc-vpn-fg-connection.png)

### 보안 연결의 상태 확인

IBM Cloud 콘솔을 통해 연결 상태를 확인할 수 있습니다. 또한 VSI를 사용하여 사이트에서 사이트로 `ping`을 시도할 수 있습니다.

![여기에 이미지 설명 입력](images/vpc-vpn-fg-status.JPG)

다음 예제 단계에서는 IBM Cloud API 또는 CLI를 사용하여 VPC를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](https://{DomainName}/docs/infrastructure/vpc/getting-started.html) 및 [API로 VPC 설정](https://{DomainName}/docs/infrastructure/vpc/example-code.html)을 참조하십시오.
