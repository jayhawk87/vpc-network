---



copyright:
  years: 2018
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

# (베타) IBM Cloud VPC에서 로드 밸런서 사용

로드 밸런서 서비스는 VPC에 대한 동일한 지역 내의 다중 서버 인스턴스 간에 트래픽을 분배합니다.

**이 서비스는 현재 베타 릴리스 버전입니다. 베타 기간 동안 사용되는 리소스는 베타 종료 후에는 사용 가능하지 않으며 지원도 제공되지 않습니다. 의견 및 질문이 있는 경우, IBM Cloud 영업 담당자에게 직접 문의하십시오.**

## 공용 로드 밸런서

로드 밸런서 서비스 인스턴스에는 공용으로 액세스 가능한 완전한 도메인 이름(FQDN)이 지정됩니다. 이 도메인 이름은 {{site.data.keyword.cloud}} VPC에서 로드 밸런서 서비스 뒤에서 호스트되는 애플리케이션에 액세스하기 위해 반드시 사용되어야 합니다. 이 도메인 이름은 하나 이상의 공인 IP 주소로 등록될 수 있습니다.

시간 경과에 따라, 이러한 공인 IP 주소와 공인 IP 주소의 수가 유지보수 및 스케일링 활동을 기반으로 변경될 수 있으며 이는 고객에게 공개됩니다. 사용자의 애플리케이션을 호스트하는 백엔드 서버 인스턴스(VSI)는 동일한 VPC 아래의 동일한 지역에서 실행되어야 합니다.

## 프론트 엔드 리스너 및 백엔드 풀
최대 10개의 프론트 엔드 리스너(애플리케이션 포트)를 정의하고 백엔드 애플리케이션 서버의 각 백엔드 풀에 맵핑할 수 있습니다. 로드 밸런서 서비스 인스턴스 및 프론트 엔드 리스너 포트에 지정된 완전한 도메인 이름(FQDN)은 공용 인터넷에 공개됩니다. 입력되는 사용자 요청은 이러한 포트에서 수신됩니다.

지원되는 프론트 엔드 리스너 프로토콜은 다음과 같습니다.
* HTTP
* HTTPS
* TCP

지원되는 백엔드 풀 프로토콜은 다음과 같습니다.
* HTTP
* TCP

입력 HTTPS 트래픽은 백엔드 서버와의 일반 텍스트 HTTP 통신을 허용하기 위해 로드 밸런서에서 종료됩니다.

최대 50개의 서버 인스턴스를 벡엔드 풀에 연결할 수 있습니다. 연결된 각 서버 인스턴스에는 포트가 구성되어야 합니다. 포트는 프론트 엔드 리스너 포트와 같거나 같지 않을 수 있습니다.

**참고:** 포트 범위 56500에서 56520까지는 관리 용도로 예약됩니다. 해당 범위의 포트는 프론트 엔드 리스너 포트로 사용될 수 없습니다.

## 로드 밸런싱 방법
다음 세 가지 로드 밸런싱 방법으로 벡엔드 애플리케이션 서버 사이에 트래픽을 분배할 수 있습니다.

* **라운드 로빈:** 라운드 로빈은 기본 로드 밸런싱 방법입니다. 이 방법을 사용하면
로드 밸런서가 라운드 로빈 방식으로 입력 클라이언트 연결을 백엔드 서버로 전달합니다. 따라서 모든 백엔드 서버가
대략 동일한 수의 클라이언트 연결을 수신합니다.

* **가중치 라운드 로빈:** 이 방법을 사용하면
로드 밸런서가 해당 서버에 지정된 가중치에 비례하여 입력 클라이언트 연결을 백엔드 서버로 전달합니다. 각 서버에는
기본 가중치인 50이 지정되고 해당 가중치는 0에서 100 사이의 임의의 값으로 사용자 정의될 수 있습니다.

    예를 들어, A, B, C라는 세 개의 애플리케이션 서버가 있으며 가중치가 각각 60, 60, 30으로
사용자 정의되면 서버 A 및 B가 같은 수의 연결을 수신하는 반면 서버 C는 그 수의 반에 해당되는 연결을 수신합니다.

    **참고:**

    * 서버 가중치를 '0'으로 재설정하는 것은 해당 서버에 새 연결이 전달되지 않음을 의미합니다. 단, 모든 기존 트래픽은 활성 상태인 한 계속 플로우됩니다. '0' 가중치를 사용하면 서버를 점진적으로 중단하고 서비스 회전에서 제거하는 데 도움이 됩니다.
    * 서버 가중치 값은 '가중치 라운드 로빈' 방법을 사용하는 경우에만 적용할 수 있습니다. '라운드 로빈' 및 '최소 연결' 로드 밸런싱 방법에서는 무시됩니다.

* **최소 연결:** 이 방법을 사용하면 지정된 시간에 가장 적은 수의 연결을 수행하는 서버 인스턴스가 다음 클라이언트 연결을 수신합니다.

## 상태 확인

상태 확인 정의는 백엔드 풀에 필수입니다.

로드 밸런서는 정기적인 상태 확인을 수행하여 백엔드 포트의 상태를 모니터링하고 그 결과에 따라 클라이언트 트래픽을 전달합니다. 지정된 벡엔드 서버 포트가 비정상적인 것으로 판단되면 새 연결이 전달되지 않습니다. 로드 밸런서는 상태가 좋지 않은 포트를 계속 모니터링하여 상태가 다시 회복되면, 즉, 두 가지의 연속적인 상태 확인 시도를 통과하면 사용을 재개합니다.

HTTP 및 TCP 포트에 대한 상태 확인은 다음과 같이 수행됩니다.

* **HTTP:** 미리 지정된 URL에 대한 `HTTP GET` 요청이 백엔드 서버 포트에 전송됩니다. `200 OK` 응답을 수신하면 서버 포트의 상태가 양호한 것으로 표시됩니다. 기본 `GET` 상태 경로는 UI를 통한 "/"이며 이는 사용자 정의될 수 있습니다.

* **TCP:** 로드 밸런서가 지정된 TCP 포트에서 백엔드 서버와의 TCP 연결을 열려고 시도합니다. 연결 시도가 성공하면 서버 포트의 상태가 양호한 것으로 표시되고 연결이 닫힙니다.

**참고:** 기본 상태 확인 간격은 5초이며 상태 확인 요청에 대한 기본 제한시간은 2초이며 기본 재시도 수는 2입니다.

## SSL 오프로드

모든 입력 HTTPS 연결의 경우, 로드 밸런서 서비스가 SSL 연결을 종료하고 백엔드 서버 인스턴스와의 일반 텍스트 HTTP 통신을 설정합니다. 이 기술을 사용하면 CPU 집중적인 SSL 핸드쉐이크 및 암호화 또는 복호화 태스크가 백엔드 서버 인스턴스에서 이동되므로 모든 CPU 사이클을 애플리케이션 트래픽 처리에 사용할 수 있습니다.

로드 밸런서가 SSL 오프로드 태스크를 수행하려면 SSL 인증서가 필요합니다. [IBM 인증서 관리자](../../../catalog/services/certificate-manager)를 통해 SSL 인증서를 관리할 수 있습니다.

로드 밸런서가 인증서 관리자 인스턴스 내의 SSL 인증서에 액세스하려면 로드 밸런서 서비스 인스턴스가 SSL 인증서를 포함하는 인증서 관리자 인스턴스에 액세스할 수 있도록 허용하는 권한을 작성해야 합니다. [ID 및 액세스 권한](../../../iam/#/authorizations)을 통해 해당 권한을 관리할 수 있습니다. 소스 서비스로 **인프라 서비스**를 선택하고 리소스 유형으로 **VPC용 로드 밸런서**를 선택하고 대상 서비스로 **인증서 관리자**를 선택한 다음 **쓰기 권한** 서비스 액세스 역할을 지정하십시오. 로드 밸런서를 작성하려면 소스 리소스 인스턴스에 대해 **모든 리소스 인스턴스** 권한을 부여해야 합니다. 대상 서비스 인스턴스는 **모든 인스턴스**이거나 사용자의 특정 인증서 관리자 리소스 인스턴스일 수 있습니다.

**참고:** 필수 권한이 제거되면, 로드 밸런서에 오류가 발생할 수 있습니다.

## ID 및 액세스 관리(IAM)

VPC 인스턴스용 로드 밸런서에 대해 액세스 정책을 구성할 수 있습니다. 사용자 액세스 정책을 관리하려면 [ID 및 사용자 액세스](../../../iam/#/users)를 방문하십시오.

### 사용자에 대한 리소스 그룹 액세스 정책 구성

로드 밸런서를 작성하려면 리소스 그룹에 액세스해야 합니다. 로드 밸런서를 작성 중인 사용자는 제공된 리소스 그룹 또는 기본 리소스 그룹(제공된 리소스 그룹이 없는 경우)에 대한 적절한 액세스 권한이 필요합니다.

1. **관리 > 계정 > 사용자**로 이동하십시오. IBM Cloud 계정에 대한 액세스 권한이 있는 사용자 목록이 표시됩니다.
2. 액세스 정책을 지정할 사용자 이름을 선택하십시오. 사용자가 표시되지 않으면 **사용자 초대**를 클릭하여 사용자를 IBM Cloud 계정에 추가하십시오.
3. **액세스 권한 지정**을 선택하십시오.
4. **리소스 그룹 내의 액세스 권한 지정**을 선택하십시오.
5. **리소스 그룹** 드롭 다운 목록에서 원하는 리소스 그룹을 선택하십시오.
6. **리소스 그룹에 액세스 권한 지정** 드롭 다운 목록에서 원하는 액세스 권한을 선택하십시오.
7. **서비스** 드롭 다운 목록에서 원하는 서비스를 선택하십시오.
9. **지정**을 선택하여 사용자에게 리소스 그룹 액세스 정책을 지정하십시오.

### 사용자에 대한 리소스 액세스 정책 구성

| 플랫폼 액세스 역할 | 로드 밸런서 조치 |
|-------------|-----|
|관리자| 로드 밸런서 작성/보기/편집/삭제 |
|편집자| 로드 밸런서 작성/보기/편집/삭제 |
|뷰어| 로드 밸런서 보기 |

1. **관리 > 계정 > 사용자**로 이동하십시오. IBM Cloud 계정에 대한 액세스 권한이 있는 사용자 목록이 표시됩니다.
2. 액세스 정책을 지정할 사용자 이름을 선택하십시오. 사용자가 표시되지 않으면 **사용자 초대**를 클릭하여 사용자를 IBM Cloud 계정에 추가하십시오.
3. **액세스 권한 지정**을 선택하십시오.
4. **리소스에 대한 액세스 권한 지정**을 선택하십시오.
5. **서비스** 드롭 다운 목록에서 **인프라 서비스**를 선택하십시오.
6. **리소스 유형** 드롭 다운 목록에서 **VPC용 로드 밸런서**를 선택하십시오.
7. **로드 밸런서 ID** 드롭 다운 목록에서 로드 밸런서 인스턴스 ID를 선택하거나 기본 값인 **모든 로드 밸런서**를 사용하십시오.
8. 사용자에게 플랫폼 액세스 역할을 지정하십시오.
9. **지정**을 선택하여 사용자에게 액세스 정책을 지정하십시오.

## 사용 가능한 API

API 호출을 작성하려면 REST 클라이언트의 일부 양식을 사용해야 합니다. 예를 들어, `curl` 명령을 사용하여 모든 기존 로드 밸런서를 검색할 수 있습니다.

```
bash
curl -X GET $rias_endpoint/v1/load_balancers -H "Authorization: $iam_token"
```

다음 절에서는 VPC 환경 내의 로드 밸런서에 대해 사용할 수 있는 API에 관한 세부사항을 제공합니다.

|설명 | API |
|-------------|-----|
| 로드 밸런서 작성 및 프로비저닝 | POST /load_balancers |
| 모든 로드 밸런서 검색 | GET /load_balancers |
| 로드 밸런서 검색 | GET /load_balancers/{id} |
| 로드 밸런서 삭제 | DELETE /load_balancers/{id} |
| 로드 밸런서 업데이트 | PATCH /load_balancers/{id} |
| 리스너 작성 | POST /load_balancers/{id}/listeners |
| 로드 밸런서의 모든 리스너 검색 | GET /load_balancers/{id}/listeners |
| 리스너 검색 | GET /load_balancers/{id}/listeners/{listener_id} |
| 리스너 삭제 | DELETE /load_balancers/{id}/listeners/{listener_id} |
| 리스너 업데이트 | PATCH /load_balancers/{id}/listeners/{listener_id} |
| 풀 작성 | POST /load_balancers/{id}/pools |
| 로드 밸런서의 모든 풀 검색 | GET /load_balancers/{id}/pools |
| 풀 검색 | GET /load_balancers/{id}/pools/{pool_id} |
| 풀 삭제 | DELETE /load_balancers/{id}/pools/{pool_id} |
| 풀 업데이트 | PATCH /load_balancers/{id}/pools/{pool_id} |
| 멤버 작성 | POST /load_balancers/{id}/pools/{pool_id}/members |
| 풀의 모든 멤버 검색 | GET /load_balancers/{id}/pools/{pool_id}/members |
| 멤버 검색 |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 풀에서 멤버 삭제 | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 멤버 업데이트 | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 풀의 멤버 업데이트 | PUT /load_balancers/{id}/pools/{pool_id}/members |

## 로드 밸런서 데모 예제

다음 예제에서는 API를 사용하여 포트 `80`에서 청취 중인 웹 애플리케이션을 실행하는 2 VPC 서버 인스턴스(`192.168.100.5` 및 `192.168.100.6`)의 앞에 로드 밸런서를 작성합니다. 로드 밸런서에는 프론트 엔드 리스너가 있으며 이로 인해 HTTPS를 통해 안전하게 웹 애플리케이션에 액세스할 수 있습니다. 그런 다음 API를 사용하여 로드 밸런서 인스턴스가 작성된 후의 세부사항을 가져올 수 있으며 로드 밸런서 인스턴스를 삭제할 수 있습니다.

### 예제 단계

전제조건:
* IBM Cloud VPC API 또는 CLI로 VPC 작성
* IBM Cloud VPC API 또는 CLI로 서브넷 작성
* IBM Cloud VPC API 또는 CLI로 서버 인스턴스 작성

자세한 정보는 [시작하기](getting-started.html), [API로 VPC 설정](vpc-setup-with-apis.md), [VPC API 참조서](https://{DomainName}/apidocs/rias) 및 [VPC CLI 참조서](../../infrastructure-service-cli-plugin/vpc-cli-reference.html#ibm-cloud-vpc-cli-reference)를 참조하십시오.

**1단계. 리스너, 풀 및 연결된 서버 인스턴스(멤버)가 있는 로드 밸런서를 작성하십시오.**

```bash
curl -H "Authorization: Bearer$iam_token" -X POST
https://us-south.iaas.cloud.ibm.com/v1/load_balancers \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

샘플 출력:
```bash
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d-7ce5-413b-aae4-90a2a88b7872.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: codeblock}

**2단계. 로드 밸런서를 가져오십시오.**

```bash
curl -H "Authorization: Bearer $iam_token" -X GET https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```
{: codeblock}

로드 밸런서가 프로비저닝하는 동안 잠시 대기하면 온라인 및 활성으로 설정됩니다.

샘플 출력:
```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: codeblock}

**3단계. 로드 밸런서를 삭제하십시오.**

```bash
curl -H "Authorization: Bearer $iam_token" -X DELETE https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```
{: codeblock}

## FAQ

이 절에서는 VPC용 로드 밸런서 서비스에 대해 자주 묻는 몇 가지 질문에 대한 답변을 소개합니다.

### 내 로드 밸런서에 다른 DNS 이름을 사용할 수 있습니까?

로드 밸런서에 자동 지정된 DNS 이름을 사용자 정의할 수는 없지만 선호하는 DNS 이름을 가리키는 CNAME(표준 이름) 레코드를 자동 지정된 로드 밸런서 DNS 이름에 추가할 수 있습니다. 예를 들어, 로드 밸런서 ID가 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`이며 자동 지정된 DNS 이름이 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`입니다. 선호하는 DNS 이름은 `www.myapp.com`입니다. `myapp.com`을 관리하기 위해 사용하는 DNS 제공자를 통해 `www.myapp.com`을 가리키는 CNAME 레코드를 로드 밸런서 DNS 이름 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`에 추가할 수 있습니다.

### 내 로드 밸런서로 정의할 수 있는 프론트 엔드 리스너는 최대 몇 개입니까?

10.

### 내 백엔드 풀에 연결할 수 있는 서버 인스턴스는 최대 몇 개입니까?

50.

### 내부 클라이언트만 액세스할 수 있는 내부 전용의 사설 로드 밸런서를 작성할 수 있습니까?  

아니오, 지금은 허용되지 않습니다.

### 로드 밸런서를 수평으로 규모 조절할 수 있습니까?  

아니오, 지금은 허용되지 않습니다.

### 로드 밸런서를 배치하기 위해 사용되는 서브넷에서 ACL 또는 보안 그룹을 사용 중인 경우, 필요한 작업은 무엇입니까?

구성된 리스너 포트 및 관리 포트(56500 - 56520 사이의 포트)에 대한 입력 트래픽을 허용하기 위해 적절한 ACL 또는 보안 그룹 규칙이 적소에 배치되었는지 확인해야 합니다. 로드 밸런서 및 백엔드 인스턴스 간의 트래픽도 허용되어야 합니다.

### 인증서 인스턴스를 찾을 수 없다는 메시지가 표시되는 이유는 무엇입니까?

* 인증서 인스턴스 CRN이 올바르지 않을 수 있습니다.
* 서비스에 서비스 권한을 부여하지 않았을 수 있습니다. 이 문서의 **SSL 오프로드** 절을 참조하십시오.

### `401 Unauthorized Error` 코드를 수신하는 이유는 무엇입니까?

사용자에 대한 다음 액세스 정책을 확인하십시오.
* 로드 밸런서 리소스 유형 액세스 정책
* 리소스 그룹 액세스 정책

### 내 로드 밸런서가 maintenance_pending 상태인 이유는 무엇입니까?

로드 밸런서는 다음과 같은 다양한 유지보수 활동 동안 `maintenance_pending` 상태가 됩니다.
* 취약점을 처리하고 보안 패치를 적용하기 위한 롤링 업그레이드
* 복구 활동

### 프로비저닝 동안 다중 서브넷을 선택해야 하는 이유는 무엇입니까?

로드 밸런서 어플라이언스는 사용자가 선택한 서브넷에 배치됩니다. 고가용성 및 중복성을 확보하기 위해 다른 구역의 서브넷을 선택하도록 강력히 권장합니다.
