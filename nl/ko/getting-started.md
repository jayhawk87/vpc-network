---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-11"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 가상 사설 클라우드에 대한 네트워킹 시작하기
{: #getting-started-with-networking-for-virtual-private-cloud}


IBM에서는 다음달에 시작될 연장 사용을 포함하여 2019년 4월 초에 시작하는 VPC에 대한 Early Access 프로그램에 참여하는 고객 수를 제한하기로 했습니다. 조직에서 IBM Virtual Private Cloud에 대한 액세스를 얻으려는 경우, 이 [지원 양식](https://cloud.ibm.com/vpc){: new_window}을 완료하십시오. 그러면 IBM 담당자가 연락하여 다음 단계를 안내합니다.
{: important}

가상 사설 클라우드에 대한 {{site.data.keyword.cloud}} 네트워킹 시작하기

1. 가상 사설 클라우드를 작성하십시오.
2. 하나 이상의 구역에서 가상 사설 클라우드에 하나 이상의 서브넷을 작성하십시오.
3. 서브넷의 리소스가 인터넷에 액세스하도록 허용하거나 그 반대를 허용하려면 서브넷에 공용 게이트웨이(PGW)를 작성하십시오.

## 선행 조건

 * **사용자 권한**: 사용자가 VPC에서 리소스를 작성하고 관리하기에 충분한 권한을 보유하고 있어야 합니다. 필수 권한 목록을 보려면 [VPC 사용자에게 필요한 권한 부여](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)를 참조하십시오.

## VPC 네트워크 자원을 프로비저닝하기 위해 UI, CLI 또는 REST API

VPC 네트워크 자원의 프로비저닝 및 관리는 UI, CLI 또는 REST API를 통해 수행할 수 있습니다.

* 사용자 인터페이스를 통해 액세스하려면 [IBM Cloud 콘솔 ![외부 링크 아이콘](../../icons/launch-glyph.svg "외부 링크 아이콘")]( https://{DomainName}/vpc){: new_window}에 로그인하십시오. 도움이 필요한 경우 [UI 안내서](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console)를 따르십시오.
* 명령행 인터페이스를 사용하려면 [IBM Cloud CLI](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview)의 [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) 플러그인을 사용하고 [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) 예제를 따르십시오.
* 고급 사용자의 경우 직접 [REST API](https://{DomainName}/apidocs/rias)를 호출할 수 있습니다. REST API를 시작하려면 [예제 코드](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) 튜토리얼을 따르십시오.

## 다음 단계

가상 사설 클라우드 네트워킹이 프로비저닝되고 나면 자세히 알아보십시오.

* [ 가상 서버 인스턴스 작성 및 관리](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [보안 그룹 사용](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [네트워크 ACL 사용](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(베타) VPN 사용](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(베타) 로드 밸런서 사용](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
