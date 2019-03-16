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

# 虚拟私有云联网入门


从 2019 年 4 月初开始，IBM 将接受有限数量的客户参与“VPC 早期访问计划”，并在接下来的几个月内开放更大的使用量。如果您的组织希望获取 IBM 虚拟私有云的访问权，请填写此[报名表](https://cloud.ibm.com/vpc){: new_window}，IBM 代表将与您联系，就后续步骤进行沟通。
{: important}

开始使用 {{site.data.keyword.cloud}} 虚拟私有云联网

1. 创建虚拟私有云。
2. 在虚拟私有云的一个或多个专区中创建一个或多个子网。
3. 如果希望子网上的资源有权访问因特网或支持通过因特网访问子网上的资源，请在子网上创建公共网关 (PGW)。

## 先决条件

 * **用户许可权**：确保用户具有足够的许可权来创建和管理 VPC 中的资源。有关必需许可权的列表，请参阅[授予 VPC 用户所需的许可权](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)。

## 使用 UI、CLI 或 REST API 来供应 VPC 网络资源

可以通过 UI、CLI 或 REST API 来供应和管理 VPC 网络资源。

* 要通过用户界面进行访问，请登录到 [IBM Cloud 控制台 ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")]( https://{DomainName}/vpc){: new_window}。如果需要帮助，请遵循 [UI 指南](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console)进行操作。
* 要使用命令行界面，请使用 [IBM Cloud CLI](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) 的 [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) 插件，然后遵循 [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) 示例进行操作。
* 对于更高级的用户，可以直接调用 [REST API](https://{DomainName}/apidocs/rias)。请遵循[示例代码](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)教程开始使用 REST API。

## 后续步骤

供应虚拟专用云联网后，请探索更多信息。

* [创建和管理虚拟服务器实例](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [使用安全组](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [使用网络 ACL](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(Beta) 使用 VPN](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(Beta) 使用负载均衡器](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
