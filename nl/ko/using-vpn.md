---



copyright:
  years: 2017,2018
lastupdated: "2018-12-12"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (베타) VPC와 함께 VPN 사용

IBM Cloud VPC VPN 서비스를 사용하면 안전하게 사설 네트워크에 연결할 수 있습니다. VPN을 사용하여 VPC 및 온프레미스 사설 네트워크 또는 또 다른 VPC 간의 IPsec 사이트 대 사이트 터널을 설정할 수 있습니다.

현재 {{site.data.keyword.cloud}} VPC 릴리스에 대해서는 정책 기반의 라우팅만 지원됩니다.

**이 서비스는 현재 베타 릴리스 버전입니다. 베타 기간 동안 사용되는 리소스는 베타 종료 후에는 사용 가능하지 않으며 지원도 제공되지 않습니다. 의견 및 질문이 있는 경우, IBM Cloud 영업 담당자에게 직접 문의하십시오.**

## 기능

* IKEv1 및 IKEv2
* 인증 알고리즘: `md5`, `sha1`, `sha256`
* 암호화 알고리즘: `3des`, `aes128`, `aes256`
* DH(Diffie-Hellman) 그룹: 2, 5, 14
* IKE 협상 모드: 기본
* IPSec 변환 프로토콜: ESP
* IPSec 캡슐화 모드: 터널
* PFS(Perfect Forward Secrecy)
* 데드 피어 발견
* 라우팅: 정책 기반
* 인증 모드: 사전 공유 키
* HA 지원

## 사용 가능한 API

다음 절에서는 IBM Cloud VPC 환경 내의 VPN에 대해 사용할 수 있는 API에 관한 세부사항을 제공합니다. 세부사항은 [VPC REST API](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies) 페이지를 참조하십시오.

### VPN 게이트웨이 및 VPN 연결

|설명 | API |
|----------------------------|-------------|
| VPN 게이트웨이 작성 | POST /vpn_gateways |
| VPN 게이트웨이 검색 | GET /vpn_gateways |
| VPN 게이트웨이 검색 | GET /vpn_gateways/{id} |
| VPN 게이트웨이 삭제 | DELETE /vpn_gateways/{id} |
| VPN 게이트웨이 업데이트 | PATCH /vpn_gateways/{id} |
| 새 VPN 연결 작성 | POST /vpn_gateways/{vpn_gateway_id}/connections |
| VPN 연결 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections |
| VPN 연결 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 연결 삭제 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 연결 업데이트 | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 연결에 필요한 모든 로컬 CIDR 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| VPN 연결에서 로컬 CIDR 삭제 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 특정 로컬 CIDR이 VPN 연결에 존재하는지 확인 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 연결에 로컬 CIDR 설정 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 연결에 필요한 모든 피어 CIDR 검색 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| VPN 연결에서 피어 CIDR 삭제 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| 특정 피어 CIDR이 VPN 연결에 존재하는지 확인 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| VPN 연결에 피어 CIDR 설정 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE 정책

|설명 | API |
|-----------------------------|--------------|
| 모든 IKE 정책 검색 | GET /ike_policies |
| IKE 정책 작성 | POST /ike_policies |
| IKE 정책 삭제 | DELETE /ike_policies/{id} |
| IKE 정책 검색 | GET /ike_policies/{id} |
| IKE 정책 업데이트 | PATCH /ike_policies/{id} |
| 지정된 IKE 정책을 사용하는 모든 연결 검색 | GET /ike_policies/{id}/connections |

### IPSec 정책

|설명 | API |
|---------------------|-------------|
| 모든 IPSec 정책 검색 | GET /ipsec_policies |
| IPSec 정책 작성 | POST /ipsec_policies |
| IPSec 정책 삭제 | DELETE /ipsec_policies/{id} |
| IPSec 정책 검색 | GET /ipsec_policies/{id} |
| IPSec 정책 업데이트 | PATCH /ipsec_policies/{id} |
| 지정된 IPsec 정책을 사용하는 모든 연결 검색 | GET /ipsec_policies/{id}/connections |

## VPN 데모 예제

다음 예제에서는 VPN을 사용하여 두 개의 VPC에 함께 연결할 수 있습니다. 즉, 두 개의 별도의 VPC 내의 서브넷이 단일 네트워크에 존재하는 것처럼 연결할 수 있습니다. 서브넷의 IP 주소가 겹치지 않아야 합니다. 시나리오는 다음과 같습니다(각 VPC에 일부 VM이 추가됨):
![예제 VPN 시나리오](images/vpc-vpn.png)

### 예제 단계

다음 예제 단계에서는 IBM Cloud API 또는 CLI를 사용하여 VPC를 작성하는 전제조건 단계를 건너뜁니다. 자세한 정보는 [시작하기](getting-started.html) 및 [API로 VPC 설정](https://{DomainName}/docs/infrastructure/vpc/example-code.html)을 참조하십시오.

필요에 따라 UI를 사용하여 VPN 게이트웨이를 작성할 수 있습니다. 단계는 [콘솔 튜토리얼 문서](https://{DomainName}/docs/infrastructure/vpc/console-tutorial.html#creating-a-vpn)를 참조하십시오.

**1단계. VPC 서브넷에서 VPN 게이트웨이를 작성하십시오.**

```bash
curl -H "Authorization: Bearer $iam_token" -X POST https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": "<SUBNET-ID-1>"}
        }'
```
{: codeblock}

샘플 출력:
```bash
{
    "created_at": "2018-07-06T19:19:28.694388Z",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "status": "pending",
    "subnet": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58",
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1"
    }
}
```
{: codeblock}

후속 단계를 위해 다음 필드를 저장하십시오.
* `id`. VPN 게이트웨이 ID이며 `VPN-GATEWAY-1-ID`로 참조됩니다.
* `address`. VPN 게이트웨이의 공인 IP 주소이며 `VPN-GATEWAY-1-IP`로 참조됩니다.

참고: 게이트웨이 상태는 VPN 게이트웨이가 작성 중인 동안 `pending`으로 표시되다가 작성이 완료되면 `available`로 변경됩니다. 작성에는 시간이 걸릴 수 있습니다. 다음 명령으로 해당 상태를 확인할 수 있습니다.
```bash
curl -H "Authorization: Bearer $iam_token" -X GET https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/<VPN-GATEWAY-1-ID>
```
{: codeblock}

**2단계. 다른 VPC에서 두 번째 VPN 게이트웨이를 작성하십시오.**

```bash
curl -H "Authorization: Bearer $iam_token" -X POST https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": "<SUBNET-ID-2>"}
            }'
```
{: codeblock}

샘플 출력:
```bash
{
    "created_at": "2018-07-06T19:33:23.789675Z",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "status": "pending",
    "subnet": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2"
    }
}
```
{: codeblock}

후속 단계를 위해 다음 필드를 저장하십시오.
* `id`. VPN 게이트웨이 ID이며 `VPN-GATEWAY-2-ID`로 참조됩니다.
* `address`. VPN 게이트웨이의 공인 IP 주소이며 `VPN-GATEWAY-2-IP`로 참조됩니다.


**3단계. `local_cidrs`를 VPC 1의 서브넷으로 설정하고 `peer_cidrs`를 VPC 2의 서브넷으로 설정한 상태에서 첫 번째 VPN 게이트웨이에서 두 번째 VPN 게이트웨이로 VPN 연결을 작성하십시오.**

```bash
curl -H "Authorization: Bearer $iam_token" -X POST https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/<VPN-GATEWAY-1-ID>/connections \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": "<VPN-GATEWAY-2-IP>",
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

**4단계. `local_cidrs`를 VPC 2의 서브넷으로 설정하고 `peer_cidrs`를 VPC 1의 서브넷으로 설정한 상태에서 두 번째 VPN 게이트웨이에서 첫 번째 VPN 게이트웨이로 VPN 연결을 작성하십시오.**

```bash
curl -H "Authorization: Bearer $iam_token" -X POST https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/<VPN-GATEWAY-2-ID>/connections \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": "<VPN-GATEWAY-1-IP>",
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

**5단계. 연결을 확인하십시오.**
VPN 연결이 설정되고 나면 서브넷 1에서 서브넷 2의 인스턴스에 연결할 수 있으며 그 반대의 경우도 가능합니다.

다음과 같이 VPN 연결의 상태를 확인할 수 있습니다.
```bash
curl -H "Authorization: Bearer $iam_token" -X GET https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/<VPN-GATEWAY-1-ID>/connections
```
{: codeblock}

샘플 출력:
```bash
{
    "connections": [
        {
            "admin_state_up": true,
            "authentication_mode": "psk",
            "created_at": "2018-07-06T19:50:49.252072Z",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "name": "vpn-connection-to-vpn-gateway-2",
            "peer_address": "169.61.161.150",
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "psk": "VPNDemoPassword",
            "route_mode": "policy",
            "status": "up"
        }
    ]
}
```
{: codeblock}

## 할당량

현재의 계정당 리소스 제한사항은 다음과 같습니다.
* 최대 7개의 VPN 게이트웨이
* VPN 게이트웨이당 최대 10개의 VPN 연결
* 지정된 VPN 게이트웨이의 VPN 연결 전체에 걸친 최대 50개의 피어 서브넷
  * 지정된 VPN 연결에 최대 15개의 피어 서브넷
* 지정된 VPN 게이트웨이의 VPN 연결 전체에 걸친 최대 50개의 로컬 서브넷
  * 지정된 VPN 연결에 최대 15개의 로컬 서브넷
* 최대 20개의 IKE 정책
* 최대 20개의 IPSec 정책

## 정책 자동 협상

VPN 연결을 구성하기 위해 IKE 및 IPSec 정책을 사용하는 것은 선택사항입니다. 정책을 선택하지 않으면 _자동 협상_이라 불리는 프로세스에 대한 기본 제안이 자동으로 선택됩니다. 자동 협상의 일부로 사용되는 제안은 다음과 같습니다.

**참고:** IBM Cloud는 개시자로서 **IKEv2**를 사용합니다. IBM Cloud는 응답자로서 **IKEv1** 및 **IKEv2**를 둘 다 사용합니다.

### IKE 자동 협상(1단계(Phase))

다음과 같은 암호화, 인증 및 DH 그룹 옵션을 원하는 대로 조합하여 사용할 수 있습니다.

|    | 암호화 |인증 | DH 그룹 |
|----|------------|----------------|----------|
|1  | aes128 | sha1   |2  |
|2  | aes256 | sha256 |5  |
|3  | 3des   | md5    | 14 |

### IPsec 자동 협상(2단계(Phase))

다음과 같은 암호화 및 인증 옵션을 원하는 대로 조합하여 사용할 수 있습니다.

|    | 암호화 |인증 | DH 그룹 |
|----|------------|----------------|----------|
|1  | aes128 | sha1   | 사용 불가능 |
|2  | aes256 | sha256 | 사용 불가능 |
|3  | 3des   | md5    | 사용 불가능 |

## FAQ

**VPN 게이트웨이를 작성할 때 동시에 VPN 연결을 작성할 수 있습니까?**

API 또는 CLI를 사용하는 경우, VPN 게이트웨이를 작성한 후에 VPN 연결을 작성해야 합니다. IBM Cloud 콘솔에서는 게이트웨이 및 연결을 동시에 작성할 수 있습니다.

**VPN 연결이 접속된 VPN 게이트웨이를 삭제한 경우, 연결은 어떻게 됩니까?**

VPN 연결도 VPN 게이트웨이와 함께 삭제됩니다.

**VPN 게이트웨이 또는 VPN 연결을 삭제하면 IKE 또는 IPSec 정책도 삭제됩니까?**

아니오, IKE 및 IPSec 정책은 다중 연결에 적용될 수 있습니다.

**게이트웨이가 위치한 서브넷을 삭제하려고 시도하면 VPN 게이트웨이는 어떻게 됩니까?**

VPN 게이트웨이를 포함하여 인스턴스가 있으면 서브넷을 삭제할 수 없습니다.

**기본 IKE 및 IPSec 정책이 있습니까?**

정책 ID(IKE 또는 IPSec)를 참조하지 않고 VPN 연결을 작성하면 자동 협상이 사용됩니다.

**VPC용 VPN이 HA 구성을 지원합니까?**

예, 그렇습니다.
