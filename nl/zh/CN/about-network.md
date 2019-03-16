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

# 关于 VPC 的联网
{: #about-networking-for-vpc}

虚拟私有云 (VPC) 是与客户帐户绑定的虚拟网络。VPC 通过提供对虚拟基础架构和网络流量分段的细颗粒度控制，为您提供了云安全性以及动态伸缩能力。

本文档涵盖在 {{site.data.keyword.cloud}} VPC 中应用的一些联网概念。描述的用例和特征包括：

* 区域
* 专区
* 保留的 IP 地址
* 公共网关
* vNIC 接口
* 子网
* 浮动 IP 地址
* VPN 连接

## 概述

要在 VPC 中设置联网，请执行以下操作：
* 使用公共网关处理子网因特网流量。
* 使用浮动 IP 处理 VSI 因特网流量。
* 使用 VPN 建立安全外部连接。

请记住：
* 某些子网地址 CIDR 范围被 IBM 保留。
* 必须先创建 VPC，然后才能在该 VPC 中创建子网。
* IPv6 支持不可用。

（可选）可以创建经典访问 VPC 以连接到 IBM Cloud 经典基础架构。
{:note}

如下图所示：
* VPC 中的子网可以通过可选的公共网关 (PGW) 连接到公用因特网。
* 可以将浮动 IP 地址 (FIP) 分配给任何 VSI，以便通过因特网访问 VSI，或通过 VSI 访问因特网。
* IBM Cloud VPC 中的子网提供专用连接；子网之间可以利用专用链接通过隐式路由器相互对话。不需要设置路径。


**图：客户可以根据需要通过子网对虚拟私有云进行细分，并且每个子网都可以访问公用因特网。**

![VPC 连接](/images/vpc-connectivity-and-security.png)

## 术语

此[词汇表](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)包含有关本文档中用于 IBM Cloud VPC 的术语的定义和信息。使用 VPC 时，您需要熟悉基本概念_区域_和_专区_，这些概念将应用于部署。

### 区域 (Regions)

区域是与部署 VPC 的地理区域相关的抽象概念。每个区域可包含多个专区，专区表示独立的故障区。IBM Cloud VPC 可以跨其分配的区域内的多个专区。

### 专区 (Zones)

专区是一个抽象概念，指托管计算、网络和存储资源以及相关散热设备和电源的物理数据中心，用于提供服务和应用程序。专区彼此隔离，以便不会造成共享的单点故障，同时提高了容错性，缩短了等待时间。专区可保证以下特性：

 * 每个专区都是一个独立的故障区，一个区域中的两个专区同时发生故障的可能性极低。
 * 一个区域中各专区之间的流量的等待时间短于 2 毫秒。

## VPC 中子网的特征

子网由指定的 IP 地址范围（CIDR 块）组成。子网绑定到单个专区，无法跨多个专区或区域。但是，子网可以跨虚拟私有云内的专区抽象整体。同一 IBM Cloud VPC 中的子网可以相互连接。


### 保留的 IP 地址

运行虚拟私有云时，某些 IP 地址会保留供 IBM 使用。下面是保留的地址（这些 IP 地址假定子网的 CIDR 范围为 10.10.10.0/24）：

  * CIDR 范围中的第一个地址 (10.10.10.0)：网络地址
  * CIDR 范围中的第二个地址 (10.10.10.1)：网关地址
  * CIDR 范围中的第三个地址 (10.10.10.2)：被 IBM 保留
  * CIDR 范围中的第四个地址 (10.10.10.3)：被 IBM 保留供未来使用
  * CIDR 范围中的最后一个地址 (10.10.10.255)：网络广播地址

### 使用公共网关建立子网外部连接

**公共网关 (PGW)** 支持子网（以及连接到该子网的所有 VSI）连接到因特网。请注意，缺省情况下，子网是专用的；但是，可以选择创建 PGW 并将子网连接到该 PGW。将子网连接到 PGW 后，该子网中的所有 VSI 都可以连接到因特网。

PGW 使用_多对一 NAT_，这意味着数千个使用专用地址的 VSI 将使用 1 个公共 IP 地址与公用因特网对话。PGW 不支持因特网发起与这些实例的连接。使用 API 可将子网连接到 PGW 以及从 PGW 拆离子网。

下图概述了网关服务的当前作用域。

![网关服务](images/scope-of-gateway-services.png)

公共网关在 VPC 中创建，但该网关在连接到子网之前不会执行任何操作。每个专区只能创建一个公共网关，例如，在具有 3 个专区的环境中，这意味着每个 VPC 可能有 3 个公共网关。
{:note}

### 使用浮动 IP 地址建立 VSI 外部连接
**浮动 IP 地址**是系统提供且可从公用因特网访问的 IP 地址。

您可以从 IBM 提供的可用浮动 IP 地址池中保留一个浮动 IP 地址，并且可以将该地址与同一虚拟私有云中的任何实例相关联或解除关联（借助于该实例的 vNIC）。任何浮动 IP 地址都可以关联到各种类型的虚拟服务器实例 (VSI)，例如负载均衡器或 VPN 网关。

浮动 IP 地址不能与多个接口相关联。必须在 VSI 上指定将与单个浮动 IP 关联的接口。该接口还将具有专用 IP 地址。后端系统在浮动 IP 与该接口的专用 IP 之间执行_一对一 NAT_ 操作。仅当特别请求释放操作时，才会释放浮动 IP。

**注：**
* **将浮动 IP 地址与 VSI 相关联会从前述 PGW 的多对一 NAT 中除去该 VSI。**
* **目前，浮动 IP 仅支持 IPv4 地址。**
* **您无法将自己的公共 IP 地址用作浮动 IP。**

有关 NAT 操作的更多信息，请参阅[相关因特网 RFC 文档 ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}。

### 使用 VPN 建立安全外部连接
虚拟专用网 (VPN) 服务可供用户通过因特网安全地连接到其 IBM Cloud VPC。

**VPN 功能**
  * 能够对 VPN 服务（站点到站点 IPSEC VPN）执行 CRUD（创建、读取、更新和删除）操作，以处理数据传输。
  * 能够将 VPN 服务关联到客户的 IBM Cloud VPC 或虚拟网络。
  * 能够通过静态定义或动态路由来识别客户的现场子网。
  * 支持安全密码，例如 SHA256、AES、3DES 和 IKEv2
  * 内置服务可靠性。
  * 持续监视 VPN 连接运行状况。
  * 支持发起者和响应者方式；也就是说，可以从隧道的任一侧发起流量。

有关当前不支持的已知限制和功能的列表，请参阅[已知限制](/docs/infrastructure/vpc?topic=vpc-known-limitations)文档。

## 了解更多

   * [IBM VPC 安全性](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [IBM VPC 地址、区域和子网](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
