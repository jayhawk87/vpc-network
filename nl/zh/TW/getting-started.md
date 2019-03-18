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

# 開始使用虛擬專用雲端的網路
{: #getting-started-with-networking-for-virtual-private-cloud}


IBM 將接受限量客戶參與 2019 年 4 月初開辦的 VPC「搶先體驗」計劃，並在接下來幾個月開放擴大使用。如果您的組織希望體驗「IBM 虛擬專用雲端」，請填寫此[申請表](https://cloud.ibm.com/vpc){: new_window}，IBM 業務代表將聯絡您進行下一步。
{: important}

開始使用虛擬專用雲端的 {{site.data.keyword.cloud}} 網路

1. 建立「虛擬專用雲端」。
2. 在一個以上區域的「虛擬專用雲端」中建立一個以上的子網路。
3. 如果您希望子網路上的資源可以存取網際網路，請在子網路上建立公用閘道 (PGW)，反之亦然。

## 必要條件

 * **使用者許可權**：確保您的使用者具有足夠的許可權可在 VPC 中建立及管理資源。如需必要許可權的清單，請參閱[授與 VPC 使用者所需的許可權](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)。

## 使用使用者介面、CLI 或 REST API 來佈建 VPC 網路資源

透過使用者介面、CLI 或 REST API 可以執行 VPC 網路資源的佈建及管理。

* 若要透過使用者介面存取，請登入 [IBM Cloud 主控台![外部鏈結圖示](../../icons/launch-glyph.svg "外部鏈結圖示")]( https://{DomainName}/vpc){: new_window}。如需協助，請遵循[使用者介面手冊](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console)。
* 如果要使用指令行介面，請使用 [IBM Cloud CLI](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) 的[infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) 外掛程式，並遵循 [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) 範例。
* 若為更高階的使用者，您可以直接呼叫 [REST API](https://{DomainName}/apidocs/rias)。請遵循[程式碼範例](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)指導教學，開始使用 REST API。

## 下一步

在佈建「虛擬專用雲端」網路之後，可進一步探索。

* [建立及管理虛擬伺服器實例](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [使用安全群組](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [使用網路 ACL](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [（測試版）使用 VPN](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [（測試版）使用負載平衡器](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
