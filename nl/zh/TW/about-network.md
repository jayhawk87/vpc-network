---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-20"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# 關於 VPC 的網路
{: #about-networking-for-vpc}

「虛擬專用雲端 (VPC)」是一個連結到客戶帳號的虛擬網路。它提供雲端安全，透過對虛擬基礎架構及網路資料流量分段提供細部控制，而具有動態調整的能力。

本文件涵蓋某些網路概念，因為它們會應用在 {{site.data.keyword.cloud}} VPC 內。所說明的使用案例及特徵包括：

* 地區
* 區域
* 保留的 IP 位址
* 公用閘道
* vNIC 介面
* 子網路
* 浮動 IP 位址
* VPN 連線

## 概觀

若要在 VPC 中設定網路，請執行下列動作：
* 針對子網路網際網路資料流量使用公用閘道。
* 對 VSI 網際網路資料流量使用浮動 IP。
* 使用 VPN 建立安全的外部連線。

請記住：
* IBM 保留部分子網路位址 CIDR 範圍。
* 在 VPC 內建立子網路之前，您必須先建立 VPC。
* 不提供 IPv6 支援。

您可以選擇性地建立「標準存取」VPC 來連接至 IBM Cloud「標準基礎架構」。
{:note}

如下圖所示：
* VPC 中的子網路可以透過選用的「公用閘道 (PGW)」連接至公用網際網路。
* 您可以將「浮動 IP 位址 (FIP)」指派給任何 VSI，以從網際網路連接它，反之亦然。
* IBM Cloud VPC 內的子網路提供專用連線功能；它們可以經由隱含的路由器，透過專用鏈結彼此交談。不需要設定路徑。


**圖：客戶可以用子網路來細分「虛擬專用雲端」，每一個子網路都可以視需要連接公用網際網路。**

![VPC 連線功能](/images/vpc-connectivity-and-security.png)

## 術語

[名詞解釋](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)包含本文件針對 IBM Cloud VPC 所使用術語的定義及相關資訊。使用 VPC 時，您需要熟悉_地區_ 和_區域_ 的基本概念，因為它們會應用到您的部署。

### 地區

地區是與 VPC 部署所在之地理區域相關的抽象概念。每個地區包含多個區域，分別代表獨立錯誤網域。IBM Cloud VPC 可以跨越其指派地區內的多個區域。

### 區域

區域是一種抽象概念，它指的是管理計算、網路和儲存空間資源以及相關冷能和電能的實體資料中心，提供各項服務和應用程式。區域彼此隔離，因此會建立不共用的單一失敗點、改良容錯，以及減少延遲。區域可保證下列內容：

 * 每個區域都是一個獨立的錯誤網域，且一個地區中的兩個區域極不可能同時失敗。
 * 一個地區中的兩個區域之間的資料流量，其延遲少於 2 毫秒。

## VPC 中的子網路性質

子網路是由指定的 IP 位址範圍（CIDR 區塊）組成。子網路會連結至單一區域，而且它們不能跨越多個區域或地區。不過，子網路可以跨越其「虛擬專用雲端」內的整個區域抽象項目。同一個 IBM Cloud VPC 中的子網路可以彼此連接。


### 保留的 IP 位址

在操作「虛擬專用雲端」時，某些 IP 位址已保留供 IBM 使用。以下是保留的位址（這些 IP 位址假設子網路的 CIDR 範圍是 10.10.10.0/24）：

  * CIDR 範圍中的第一個位址 (10.10.10.0)：網址
  * CIDR 範圍中的第二個位址 (10.10.10.1)：閘道位址
  * CIDR 範圍中的第三個位址 (10.10.10.2)：由 IBM 保留
  * CIDR 範圍中的第四個位址 (10.10.10.3)：由 IBM 保留供未來使用
  * CIDR 範圍中的最後一個位址 (10.10.10.255)：網路播送位址

### 對子網路的外部連線功能使用「公用閘道」

**公用閘道 (PGW)** 可讓子網路（所有 VSI 都連接至子網路）連接至網際網路。請注意，依預設，子網路是專用的；不過，您可以選擇性地建立 PGW，並將子網路連接至 PGW。子網路連接至 PGW 之後，該子網路中的所有 VSI 就可以連接至網際網路。

PGW 使用_多對一 NAT_，這表示含有專用位址的數千個 VSI 將使用 1 個公用 IP 位址來與公用網際網路進行交談。PGW 不會讓網際網路起始與那些實例的連線。請使用 API 將子網路與 PGW 連接及分離。

下圖彙總閘道服務的現行範圍。

![閘道服務](images/scope-of-gateway-services.png)

在 VPC 中建立公用閘道之後，要等到閘道連接至子網路之後它才能作用。每個區域只能建立一個公用閘道，例如，這表示在具有 3 個區域的環境中，每個 VPC 可以有 3 個公用閘道。
{:note}

### 對 VSI 的外部連線功能使用浮動 IP 位址
**浮動 IP 位址**是系統所提供的 IP 位址，可從公用網際網路連接它們。

您可以從 IBM 所提供的可用「浮動 IP 位址」儲存區中保留一個「浮動 IP 位址」，您可以將它與相同「虛擬專用雲端」中的任何實例建立關聯或取消關聯，方法是透過該實例的 vNIC 來進行。任何「浮動 IP 位址」都可以與各種類型的虛擬伺服器實例 (VSI) 相關聯，例如，負載平衡器或 VPN 閘道。

您的「浮動 IP 位址」無法與多個介面相關聯。您必須在 VSI 上指定要與該個別「浮動 IP」相關聯的介面。該介面也會有專用 IP 位址。後端系統會在「浮動 IP」和該介面的「專用 IP」之間執行_一對一 NAT_ 作業。只有在您特別要求釋放動作時，才會釋放「浮動 IP」。

**附註：**
* **建立「浮動 IP 位址」與 VSI 的關聯，可從前面提及的 PGW「多對一 NAT」中移除 VSI。**
* **目前，「浮動 IP」僅支援 IPv4 位址。**
* **您不能使用自己的公用 IP 位址作為浮動 IP。**

如需 NAT 作業的相關資訊，請參閱[相關的網際網路 RFC 文件![外部鏈結圖示](../../icons/launch-glyph.svg "外部鏈結圖示")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}。

### 使用 VPN 建立安全的外部連線
「虛擬專用網路 (VPN)」服務可讓使用者從網際網路安全地連接至其 IBM Cloud VPC。

**VPN 功能**
  * 對於 VPN 服務（點對點 IPSEC VPN）執行 CRUD（建立、讀取、更新及刪除）來處理資料傳送的能力。
  * 將 VPN 服務與客戶之 IBM Cloud VPC 或虛擬網路相關聯的能力。
  * 依靜態定義或動態遞送來識別客戶的現場子網路的能力。
  * 支援安全密碼，例如 SHA256、AES、3DES、IKEv2。
  * 內建服務可靠性。
  * 持續監視 VPN 連線性能。
  * 同時支援起始器及回應者模式；亦即，可以從通道任一端起始資料流量。

如需已知限制及目前所不支援之特性的清單，請參閱[已知限制](/docs/infrastructure/vpc?topic=vpc-known-limitations)文件。

## 進一步瞭解

   * [IBM VPC 安全](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [IBM VPC 位址、地區和子網路](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
