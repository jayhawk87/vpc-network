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


# 为 VPC 选择 IP 范围
{: #choosing-ip-ranges-for-your-vpc}

使用 CIDR 表示法，例如：

* `<IPv4 address>/number`（VPC 地址示例：10.10.0.0/16）。

您希望将 IPv4 的最后 16 位（65,536 个地址）保留为 0，以便可以将这些地址用于同一 {{site.data.keyword.cloud}} VPC 中的各种子网 IP 地址（子网 IP 地址示例：10.10.1.0/24）。

**注：**RFC 1918 建议遵循此约定。

万一您不熟悉 CIDR 表示法，请记住，斜杠后的数字越小，分配的 IP 地址**越多**，因为斜杠后的数字代表子网前缀掩码中前导 1 的位数。

如果您需要更多信息，可以在线找到有关_无类域间路由_ (CIDR) 的一些优秀的文章。
