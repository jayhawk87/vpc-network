---

copyright:
  years: 2017, 2018
lastupdated: "2018-10-25"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# API를 사용하는 보안 그룹 사용 예제

다음은 {{site.data.keyword.cloud}} VPC REST API를 사용하여 보안 그룹을 작성하고 관리하는 방법을 보여주는 예제입니다.


## 전제조건

보안 그룹을 사용하려면 먼저 실행 중인 IBM Cloud VPC가 있어야 합니다.

다음 예제에서는 IBM Cloud VPC의 ID를 알아야 합니다. 다음과 같이 ID를 복사하여 변수에 붙여넣으십시오.

```bash
# Something like this: `vpc="35fb0489-7105-41b9-99de-033fae723006"`
vpc="<YOUR_VPC_ID>"
```
{: codeblock}


## 보안 그룹 작성

IBM Cloud VPC에서 `my-security-group`이라는 이름의 보안 그룹을 작성하십시오.

```bash
curl -X POST $rias_endpoint/v1/security_groups \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: codeblock}

VPC와 유사하게 보안 그룹 ID를 변수에 저장하십시오.
```bash
# Something like this: `security_group="35fb0489-7105-41b9-99de-033fae723006"`
security_group="<YOUR_SECURITY_GROUP_ID>"
```
{: codeblock}


## SSH 허용 규칙 추가

포트 22에서 인바운드 연결을 허용할 보안 그룹에 대한 규칙을 작성하십시오.

```bash
curl -X POST $rias_endpoint/v1/security_groups/$security_group/rules \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```


## 보안 그룹 삭제

보안 그룹을 정리하려면 연관된 네트워크 인터페이스가 없어야 하며 다른 보안 그룹 내의 규칙에서 참조되지 않아야 합니다.

```bash
curl -X DELETE $rias_endpoint/v1/security_groups/$security_group \
  -H "Authorization: $iam_token"
```
