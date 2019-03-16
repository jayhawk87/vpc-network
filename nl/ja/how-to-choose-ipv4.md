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


# VPC の IP 範囲の選択
{: #choosing-ip-ranges-for-your-vpc}

次のような CIDR 表記を使用します。

* `<IPv4 address>/number` (VPC アドレスの例: 10.10.0.0/16)。

IPv4 の最後の 16 ビット (アドレス 65,536 個分) を 0 として予約しておけば、同じ {{site.data.keyword.cloud}} VPC 内のさまざまなサブネット IP アドレスに使用できます (サブネット IP アドレスの例: 10.10.1.0/24)。

**注:** このきまりは、RFC 1918 で推奨されています。

CIDR 表記を初めて使用するユーザーへの注記として、スラッシュの後の数字が小さいほど、割り振る IP アドレスの数が**増える**点に注意してください。これは、スラッシュの後の数字がサブネットのプレフィックス・マスクの先行 1 ビットの数を表すためです。

詳細情報が必要な場合は、_クラスレス・ドメイン間ルーティング_ (CIDR) に関する多数の優れた記事をオンラインで見つけることができます。
