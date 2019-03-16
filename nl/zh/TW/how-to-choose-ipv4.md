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


# 選擇 VPC 的 IP 範圍
{: #choosing-ip-ranges-for-your-vpc}

使用 CIDR 表示法，例如：

* `<IPv4 address>/number`（VPC 位址範例：10.10.0.0/16）。

您想要將 IPv4 的最後 16 個位元（65,536 位址）保留為 0，這樣您就可以在相同的 {{site.data.keyword.cloud}} VPC（子網路 IP 位址範例：10.10.1.0/24）內，將它們用於各種子網路 IP 位址。

**附註：**RFC 1918 建議採用這個慣例。

萬一您是第一次使用 CIDR 表示法，請記住，斜線之後的數字越小，您配置的 IP 位址**越多**，因為斜線之後的數字代表子網路的字首遮罩前面的 1 位元數目。

如果您需要更多資訊，可以在線上找到關於_無類別內部網域遞送_ (CIDR) 的許多優秀文章。
