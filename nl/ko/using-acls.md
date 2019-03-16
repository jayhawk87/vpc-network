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

# VPC CLI 및 API에서 네트워크 ACL 사용

{{site.data.keyword.cloud}} 가상 프라이빗 클라우드에서 사용 가능한 액세스 제어 목록(ACL) 기능을 사용하면 클라우드 상의 중요한 비즈니스 워크로드에 대한 입력 및 출력 트래픽을 서브넷 레벨에서 제어할 수 있습니다. ACL은 보안 그룹과 유사하게 세부 단위의 제어가 가능한 기본 제공 가상 방화벽입니다.

다음은 보안 그룹 및 ACL 간의 몇 가지 핵심 차이점을 요약한 표입니다.

|설명 |       보안 그룹         |   ACL  |
|-------------|-----------------|---------|
| 제어 레벨  | VSI 인스턴스    | 서브넷  |
| 상태   | Stateful - 리턴 트래픽이 기본적으로 허용됨 | Stateless - 리턴 트래픽이 기본적으로 거부되고 명시적으로 허용되어야 함|
| 지원되는 규칙 | 허용 규칙만 | 허용 및 거부 규칙 둘 다|
| 규칙이 적용되는 방법 | 모든 규칙이 고려됨 | 규칙이 순서대로 처리됨 |
| 연관된 리소스 간의 관계 | 인스턴스가 다중 보안 그룹과 연관될 수 있음 | 다중 서브넷이 동일한 ACL과 연관될 수 있음 |

이 문서에서 제공된 예제에 따라 VPC에서 네트워크 ACL을 작성하고 서브넷을 보호할 수 있습니다.

## 현재 사용 가능한 API 및 CLI

각 API의 매개변수, 요청 본문 및 응답 세부사항을 보려면 [API 참조서](https://{DomainName}/docs/infrastructure/apidocs/rias) 주제를 참조하십시오.

명령 세부사항, CLI 플러그인 설치 및 CLI 사용의 전제조건 단계에 대한 내용은 [CLI 참조서](../../infrastructure-service-cli-plugin/vpc-cli-reference.html#ibm-cloud-vpc-cli-reference) 주제를 참조하십시오.


## ACL 및 ACL 규칙 관련 작업

ACL을 효과적으로 만들기 위해 인바운드 및 아웃바운드 네트워크 트래픽을 처리하는 방법에 대한 규칙을 작성합니다.

### ACL 규칙

네트워크 ACL을 사용하여 다중 인바운드 및 아웃바운드 규칙을 작성할 수 있습니다.

* 인바운드/아웃바운드 규칙을 사용하면 지정된 프로토콜 및 포트를 사용하는 소스 IP 범위 또는 목적지 IP 범위에 대한 트래픽을 허용하거나 거부할 수 있습니다.  
* ACL 규칙은 순서대로 고려됩니다. 낮은 우선순위 규칙은 높은 우선순위 규칙이 일치하지 않는 경우에만 평가됩니다.
* 인바운드 규칙은 아웃바운드 규칙과 별도입니다.
* 현재 지원되는 프로토콜은 TCP, UDP 및 ICMP입니다. 또한 **all** 옵션을 사용하여 _all_ 또는 _other_(높은 우선순위의 규칙이 지정된 경우) 프로토콜을 지정할 수 있습니다.

현재 제한사항에 대한 정보는 [알려진 제한사항 문서](../vpc/known-limitations.html)를 참조하십시오.

### 서브넷에 ACL 연결

ACL을 서브넷에 연결하는 두 가지 옵션이 있습니다.

* 새 서브넷을 작성할 때 연결할 ACL을 지정할 수 있습니다. ACL을 지정하지 않으면 기본 네트워크 ACL이 연결됩니다. 기본 ACL은 이 서브넷으로 향하는 모든 인바운드 트래픽 및 이 서브넷으로부터의 모든 아웃바운드 트래픽을 허용합니다.
* 또한 ACL을 기존 서브넷에 연결할 수 있습니다. 이 서브넷에 다른 ACL이 이미 연결되어 있으면 해당 ACL이 새 ACL이 연결되기 전에 분리됩니다.

## ACL 데모 예제

다음 예제에서는 두 개의 ACL을 작성하여 명령행 인터페이스(CLI)에서 두 개의 서브넷과 연결합니다. 시나리오는 다음과 같습니다.

![예제 ACL 시나리오](images/vpc-acls.png)

그림에서 보듯이 인터넷으로부터의 요청을 처리하는 두 개의 웹 서버와 공개하기를 원하지 않는 두 개의 백엔드 서버가 있습니다. 이 예제에서는 서버를 두 개의 독립된 서브넷, 즉, 10.10.10.0/24 및 10.10.20.0/24에 각각 배치하고 웹 서버가 백엔드 서버와 데이터를 교환할 수 있도록 허용해야 합니다. 또한 백엔드 서버에서 제한된 아웃바운드 트래픽을 허용하려고 합니다.

### 예제 규칙

다음 예제 규칙은 앞에서 설명한 대로 기본 시나리오에 대한 ACL 규칙을 설정하는 방법을 표시합니다.

**참고:** 잘 세분화된 규칙에 덜 세분화된 규칙보다 높은 우선순위를 부여하는 것이 좋습니다. 예를 들어, 서브넷 10.10.30.0/24로부터의 모든 트래픽을 차단하는 규칙이 있으며 이 규칙이 더 높은 우선순위와 일치하는 경우, 10.10.30.5로부터의 트래픽을 허용하는 더 낮은 우선순위의 세분화된 규칙은 결코 적용되지 않습니다.

**ACL-1 예제 규칙**:

| 인바운드/아웃바운드| 허용/거부 | 소스/대상 IP | 프로토콜 | 포트 | 설명|
|--------------|-----------|------|------|------------------|-------|
| 인바운드 | 허용 | 0.0.0.0/0 | TCP| 80 | 인터넷으로부터의 HTTP 트래픽 허용|
| 인바운드 | 허용 | 0.0.0.0/0 | TCP| 443 | 인터넷으로부터의 HTTPS 트래픽 허용|
| 인바운드 | 허용 | 10.10.20.0/24 |모두| 모두| 백엔드 서버가 배치된 서브넷 10.10.20.0/24로부터의 모든 인바운드 트래픽 허용 |
| 인바운드 | 거부 | 0.0.0.0/0|모두| 모두| 모든 기타 인바운드 트래픽 거부|
| 아웃바운드 | 허용 | 0.0.0.0/0 | TCP|80 | 인터넷에 대한 HTTP 트래픽 허용|
| 아웃바운드 | 허용 | 0.0.0.0/0 | TCP| 443 | 인터넷에 대한 HTTPS 트래픽 허용|
| 아웃바운드 | 허용 | 10.10.20.0/24 |모두| 모두| 백엔드 서버가 배치된 서브넷 10.10.20.0/24에 대한 모든 아웃바운드 트래픽 허용 |
| 아웃바운드 | 거부 | 0.0.0.0/0|모두| 모두| 모든 기타 아웃바운드 트래픽 거부|


**ACL-2 예제 규칙**:

|인바운드/아웃바운드| 허용/거부 | 소스/대상 IP | 프로토콜 | 포트 |설명|
|--------------|-----------|------|------|------------------|--------|
| 인바운드 | 허용 | 10.10.10.0/24 |모두|모두| 웹 서버가 배치된 서브넷 10.10.10.0/24로부터의 모든 인바운드 트래픽 허용 |
| 인바운드 | 거부 | 0.0.0.0/0|모두|모두| 모든 기타 인바운드 트래픽 거부|
| 아웃바운드 | 허용 | 0.0.0.0/0 | TCP| 80 | 인터넷에 대한 HTTP 트래픽 허용|
| 아웃바운드 | 허용 | 0.0.0.0/0 | TCP| 443 | 인터넷에 대한 HTTPS 트래픽 허용|
| 아웃바운드 | 허용 | 10.10.10.0/24 |모두|모두| 웹 서버가 배치된 서브넷 10.10.10.0/24에 대한 모든 아웃바운드 트래픽 허용 |
| 아웃바운드 | 거부 | 0.0.0.0/0|모두|모두| 모든 기타 아웃바운드 트래픽 거부|

이 예제에서는 일반 사례에 대해서만 설명합니다. 사용자의 시나리오에서는 트래픽을 좀 더 세분화하여 제어해야 하는 경우가 있을 수 있습니다.

* 운영 목적으로 원격 네트워크에서 10.10.10.0/24 서브넷에 대한 액세스가 필요한 네트워크 관리자가 있습니다. 이 경우, 인터넷에서 이 서브넷으로의 SSH 트래픽을 허용해야 합니다.
* 두 서브넷 사이에 허용할 프로토콜 범위를 좁히고자 할 수 있습니다.

### 예제 단계

다음 예제 단계에서는 IBM Cloud VPC API 또는 CLI를 사용하여 VPC를 작성하는 전제조건 단계를 건너뜁니다. 해당 단계를 먼저 수행해야 합니다. 자세한 정보는 [시작하기](getting-started.html) 및 [API로 VPC 설정](../vpc/example-code.html)을 참조하십시오.


#### 1단계. ACL 작성

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

응답에는 새로 작성된 ACL ID가 포함됩니다. 예를 들어, `a4e28308-8ee7-46ab-8108-9f881f22bdbf`입니다. (사용자의 ID는 다를 수 있습니다.)

#### 2단계. 기본 규칙 검색

새 규칙을 추가하기 전에 그 앞에 새 규칙을 삽입할 수 있도록 기본 인바운드 및 아웃바운드 ACL 규칙을 검색하십시오.

```
ibmcloud is network-acl-rules <my_web_subnet_acl ID>

ibmcloud is network-acl-rules <my_backend_subnet_acl ID>

```
{: codeblock}

응답은 모든 프로토콜에서 모든 IPv4 트래픽을 허용하는 기본 인바운드 및 아웃바운드 규칙을 표시합니다. (사용자의 경우에 기본 규칙의 이름 및 ID가 다를 수 있습니다.)

```
Getting rules of network acl 2d86bc3f-58e4-436a-8c1a-9b0a710556d6 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```

#### 3단계. 설명한 대로 규칙 추가

{: codeblock}

1. 기본 인바운드 규칙 앞에 새 인바운드 규칙을 삽입하십시오.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 <my_web_subnet_acl ID> deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule <allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6 ID>

ibmcloud is network-acl-rule-add my_web_acl_rule100 <my_web_subnet_acl ID> allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule <my_web_acl_rule200 ID>

ibmcloud is network-acl-rule-add my_web_acl_rule20 <my_web_subnet_acl ID> allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule <my_web_acl_rule100 ID>

ibmcloud is network-acl-rule-add my_web_acl_rule10 <my_web_subnet_acl ID> allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule <my_web_acl_rule20 ID>
```
{: codeblock}


2. 기본 아웃바운드 규칙 앞에 새 아웃바운드 규칙을 삽입하십시오.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e <my_web_subnet_acl ID> deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule <allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6 ID>

ibmcloud is network-acl-rule-add my_web_acl_rule100e <my_web_subnet_acl ID> allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule <my_web_acl_rule200e ID>

ibmcloud is network-acl-rule-add my_web_acl_rule20e <my_web_subnet_acl ID> allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 443 --port-min 443 --before-rule <my_web_acl_rule100e ID>

ibmcloud is network-acl-rule-add my_web_acl_rule10e <my_web_subnet_acl ID> allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 80 --port-min 80 --before-rule <my_web_acl_rule20e ID>


```
{: codeblock}

#### 4단계. 새로 작성된 ACL로 두 개의 서브넷 작성

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl <my_web_subnet_acl ID>

ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl <my_web_backend_acl ID>
```
{: codeblock}


## 아직 사용 가능하지 않은 기능

* 서브넷에서 ACL 분리
* ACL 태그에 대한 지원
* ACL에 대한 CRN 지원
* VPC 환경 내의 ACL에 대한 성능 분석


## 명령 목록 치트 시트

ACL에 대해 사용 가능한 VPC CLI 명령의 전체 목록을 보려면 다음을 입력하십시오.

```
ibmcloud is network-acls
```

ACL 및 규칙을 포함한 메타데이터를 보려면 다음을 입력하십시오.

```
ibmcloud is network-acl <acl_ID>
```
사용자의 서브넷의 ACL에 대한 올바른 ACL 이름으로 대체하십시오.

기본 인바운드 ACL 규칙을 가져오려면 다음 명령을 사용하십시오.

```
ibmcloud is network-acl-rules <acl_ID> --direction inbound
```

ACL 규칙을 추가하려면 기본 인바운드 규칙 앞에 `ping` 인바운드 규칙을 추가하는 다음 예제 명령을 참조하십시오.

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: codeblock}
