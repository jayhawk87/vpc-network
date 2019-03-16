---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-12"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# 원격 Vyatta 피어와의 보안 연결 작성

이 문서는 Vyatta 버전: AT&T vRouter 5600 1801d를 기반으로 합니다.

## 예제 단계
원격 Vyatta 피어에 대한 연결의 토폴로지는 두 VPC 사이에 VPN 연결을 작성하는 것과 유사합니다. 단, 한 측이 Vyatta 장치로 대체됩니다.

![여기에 이미지 설명 입력](images/vpc-vpn-vy-figure.png)

### 원격 Vyatta 피어와의 보안 연결 작성 방법

VPN 피어가 원격 VPN 피어로부터 연결 요청을 수신하면 IPsec 1단계(Phase) 매개변수를 사용하여 보안 연결을 설정하고 해당 VPN 피어를 인증합니다. 그런 다음, 보안 정책이 연결을 허용하면 Vyatta 장치가 IPsec 2단계(Phase) 매개변수를 사용하여 터널을 설정하고 IPsec 보안 정책을 적용합니다. 키 관리, 인증 및 보안 서비스가 IKE 프로토콜을 통해 동적으로 협상됩니다.

Vyatta 명령행으로 이동하여 IPsec 터널을 구성하거나 `*.vcli` 파일을 업로드하여 사용자의 구성을 로드할 수 있습니다.

**해당 기능을 지원하려면 Vyatta 장치에 의해 다음과 같은 일반 구성 단계가 수행되어야 합니다.**

* Vyatta 장치가 원격 피어를 인증하고 보안 연결을 설정하는 데 필요한 1단계(Phase) 매개변수를 정의하십시오.

* Vyatta 장치가 원격 피어를 사용하여 VPN 터널을 작성하는 데 필요한 2단계(Phase) 매개변수를 정의하십시오.

IBM Cloud VPC의 VPN 기능에 연결하기 위해 권장되는 구성은 다음과 같습니다.

1. 인증에서 `IKEv2` 선택
2. 1단계(Phase) 제안에서 `DH-group 2`를 사용하도록 설정
3. 1단계(Phase) 제안에서 `lifetime = 36000`을 설정
4. 2단계(Phase) 제안에서 PFS를 사용하지 않도록 설정
5. 2단계(Phase) 제안에서 `lifetime = 10800` 설정
6. 피어 및 서브넷의 정보를 2단계(Phase) 제안에 입력

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```

그런 다음 SCP를 사용하여 이 `*.vcli` 파일을 Vyatta에 업로드하여 구성을 적용할 수 있습니다.

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### 로컬 IBM Cloud VPC와의 보안 연결 작성

 보안 연결을 작성하기 위해 VPC 내에서 VPN 연결을 작성할 것이며 이는 2 VPC 예제와 유사합니다.

* `local_cidrs`를 VPC의 서브넷 값으로 설정하고 `peer_cidrs`를 Vyatta 장치의 서브넷 값으로 설정한 상태로 VPC 및 Vyatta 장치 사이의 VPN 연결과 함께 VPC 서브넷에 VPN 게이트웨이를 작성하십시오.

**참고:** 게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다.

![여기에 이미지 설명 입력](images/vpc-vpn-vy-connection.png)

### 보안 연결의 상태 확인

IBM Cloud 콘솔을 통해 연결 상태를 확인할 수 있습니다. 또한 VSI를 사용하여 사이트에서 사이트로 `ping`을 시도할 수 있습니다.

![여기에 이미지 설명 입력](images/vpc-vpn-vy-status.png)

다음 예제 단계에서는 IBM Cloud API 또는 CLI를 사용하여 VPC를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](../vpc/getting-started.html) 및 [API로 VPC 설정](../vpc/example-code.html)을 참조하십시오.
