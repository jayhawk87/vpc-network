---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}

# VPC에서 웹 서버 사이의 연결 설정
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

이 문서는 Python을 사용하여 {{site.data.keyword.cloud}} VPC의 두 웹 서버 간에 통신을 설정하기 위한 몇 가지 예제 단계를 안내합니다. 여기에서는 동일한 구역 및 다른 구역에서 통신 서버를 작성하는 방법을 보여줍니다.

## 선행 조건

VPC에서 웹 서버 간의 통신을 설정하려면 각 서브넷에서 사용할 수 있는 두 개의 서브넷과 최소 두 개의 인스턴스로 VPC를 작성해야 합니다. VPC, 서브넷 및 인스턴스의 ID를 알아야 합니다. 이 구성 설정에 도움이 필요한 경우, [CLI로 VPC 리소스 작성 안내서](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)의 단계를 따르십시오.

## 시나리오 1: 동일한 구역에 있지만 다른 서브넷의 서버 간의 연결

`vpc` holds vpc ID
`subnet1` holds subnet 1 ID 
`subnet2` holds subnet 2 ID


## 파트 1: 동일한 구역 및 VPC에서 서버 간의 연결

공용 게이트웨이가 연결된 동일한 서브넷에서 둘 이상의 인스턴스를 작성하고 나면, 한 인스턴스에서 HTML 코드가 있는 서버를 호스팅하여 연결을 테스트한 후 서버로 `curl`하고 컴퓨터의 브라우저에서나 다른 브라우저에서 HTML 파일을 다운로드할 수 있습니다.

공용 게이트웨이를 가지는 동일한 서브넷 및 동일한 보안 그룹에 `instance_1` 및 `instance_2`가 있다고 가정하겠습니다. 인바운드 액세스를 허용하기 위해 보안 그룹에서 인스턴스에 대한 API, CLI 또는 UI를 사용하여 규칙을 추가해야 합니다. `instance_1`에서 서버를 호스트할 경우, `instance_2` 유동 IP에 대한 규칙을 추가해야 합니다. 예를 들어 IPv4가 `169.61.161.161`로 설정된 `instance_1` 호스팅 서버가 있고 IPv4가 `169.61.161.235`로 설정된 `instance_2`가 있고 `instance_1`에서 파일을 가져오려고 시도한다고 가정하겠습니다. 다음과 같이 표시됩니다. 

![보안 그룹 새 규칙](images/security-group-ui-ex1.png)

* 모든 보안 그룹 나열 `ibmcloud is sgs`
* 다음은 이전 결과를 얻기 위해 작동하는 API 요청입니다. 

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. `instance_1`에 기본 `index.html` 파일을 작성하십시오. 다음은 기본 HTML 파일의 예제입니다. 

```
<!DOCTYPE html>
<html>
<body>

<h1>You have successfully connected to the instance</h1>
<p>Hello, world!</p>

</body>
</html>
```
{:pre}

2. 파일이 설치된 동일한 디렉토리에서 `python -m SimpleHTTPServer<port number>` 명령을 실행하십시오. 기본 포트 8000이 이미 사용 중인 경우 기본 포트를 사용할 수 있습니다.

3. `instance_2`로 SSH 처리하십시오. 

유동 IP 주소를 찾으려면 `ibmcloud is ips` 명령을 사용하고 `ibmcloud is ip $floating_ip_id` 명령을 사용하여 `floating_ip_id`를 식별하십시오. 공용 주소를 식별할 수 있어야 합니다.

4. `instance_1`의 IP 주소를 식별하십시오.

5. `curl -X GET http://IP_address:Port_number/index.html`을 사용하여 호스트 서버 `instance_1`에서 `index.html`을 다운로드하도록 요청하십시오. (예: `curl -X GET http://169.61.161.161:8000/index.html`)

6. `index.html`의 출력을 확인할 수 있어야 합니다. 

7. (로컬 컴퓨터 브라우저를 수단으로 서버에 액세스) 다음 그림에 표시된 대로 모든 인바운드 트래픽에 대한 규칙을 추가하십시오.

![모든 인바운드가 포함된 보안 그룹 새 규칙](images/security-group-ui-ex2.png)

* 모든 프로토콜을 허용하도록 모든 소스에서 보안 그룹까지 규칙을 추가하려면 다음을 사용하십시오.

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**보다 상세한 요청 예제:**

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "port_max": 22,
  "port_min": 22,
  "protocol": "tcp",
  "remote": {
    "cidr_block": "131.121.0.0/24"
  }
}'
```
{:pre}

8. 웹 브라우저를 연 후에 `http://IP_address:Port_number/index.html`을 입력하십시오. 

9. HTML 형식으로 표시된 인덱스 파일을 확인해야 합니다. 

## 파트 2: 다른 구역 및 VPC에서 서버 간의 연결 

공용 게이트웨이가 설정되지 않은 다른 서브넷의 인스턴스를 인터넷 액세스가 있는 인스턴스에 연결하십시오. 다음 두 가지 유형의 인스턴스를 설정할 수 있습니다.

* 공용 게이트웨이를 사용하여 인터넷에서 요청을 처리하는 웹 서버
* 공용 게이트웨이는 사용 안함으로 설정되어 있고 내부 트래픽만 전송하여 애플리케이션을 실행하고 중요 데이터를 저장하는 백엔드 서버

보안 그룹(SG)은 가상 서버 인스턴스(VSI)에 적용되고, 네트워크 ACL은 서브넷에 적용됩니다. 

따라서 이 전체 태스크를 완료하려면 VSI를 작성하고 서브넷을 작성하고 SG 및 ACL 규칙을 작성한 후 SG를 인스턴스의 네트워크 인터페이스에 연결하고 네트워크 ACL을 특정 서브넷에 연결해야 합니다.

* `subnet1` 보안 그룹의 모든 규칙을 나열하려면 다음을 사용하십시오. 

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. 파트 1에서 서브넷 활동의 보안 그룹에서 `inbound-allow-all-traffic` 규칙을 제거하십시오.

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. 새 VPC를 작성한 후, 공용 게이트웨이 연결 없이 서브넷을 작성하십시오.

3. 공용 게이트웨이 없이(인터넷 연결 없이) 새 서브넷에서 `instance_3`을 작성하십시오. 더 큰 CPU 코어 및 메모리를 백엔드를 위해 인스턴스에 할당할 수 있습니다. 

4. 파트 1에서 보안 그룹의 `instance_3`에 대한 새 규칙을 추가하십시오. 예를 들어 `instance_3`은 유동 IP `169.61.181.25`를 가집니다.

![다른 서브넷 인바운드에서 인스턴스가 포함된 보안 그룹 새 규칙](images/security-group-ui-ex3.png)

5. `curl -X GET http://IP_address:Port_number/index.html` 양식의 명령을 사용하여 호스트 서버 instance_1에서 `index.html`을 다운로드하도록 요청하십시오. (예: `curl -X GET http://169.61.161.161:8000/index.html`)

6. `index.html`의 출력을 확인할 수 있어야 합니다. 

이는 인스턴스 2를 제외한 3과 1사이가 연결되었음을 의미합니다.

## 다음 단계

[VPC에서 ACL을 사용하기 위한 안내서](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)에 표시된 대로 인바운드 트래픽을 제어하는 일부 ACL 규칙을 작성하여 서버를 보호할 수 있습니다.
