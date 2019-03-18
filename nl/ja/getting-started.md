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

# Virtual Private Cloud でのネットワーキングの概要
{: #getting-started-with-networking-for-virtual-private-cloud}


IBM は、2019 年 4 月上旬に開始する VPC の早期アクセス・プログラムについて、限定数のお客様の参加を受け付けます。その数カ月後に、利用者数を増やす予定です。IBM Virtual Private Cloud へのアクセスを希望するお客様は、この[応募フォーム](https://cloud.ibm.com/vpc){: new_window}にご記入ください。IBM 担当員から次のステップについてお知らせいたします。
{: important}

仮想プライベート・クラウドでの {{site.data.keyword.cloud}} のネットワーキングを開始するには、以下のようにします

1. 仮想プライベート・クラウドを作成します。
2. 1 つ以上のゾーンで仮想プライベート・クラウドに 1 つ以上のサブネットを作成します。
3. サブネット上のリソースがインターネットにアクセスできるようにするには (インターネットからのアクセスの場合も同様)、サブネット上にパブリック・ゲートウェイ (PGW) を作成します。

## 前提条件

 * **ユーザー許可**: VPC でリソースを作成および管理するための十分な許可がユーザーに付与されていることを確認してください。 必要な許可のリストについては、[VPC ユーザーに必要な許可の付与](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)を参照してください。

## UI、CLI、または REST API を使用して VPC ネットワーク・リソースをプロビジョンする

UI、CLI、または REST API を使用して VPC ネットワーク・リソースのプロビジョンや管理を行うことができます。

* ユーザー・インターフェースでアクセスする場合は、[IBM Cloud コンソール ![外部リンク・アイコン](../../icons/launch-glyph.svg "外部リンク・アイコン")]( https://{DomainName}/vpc){: new_window} にログインします。支援が必要な場合は、[UI ガイド](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console) に従ってください。
* コマンド・ライン・インターフェースを使用するには、[IBM Cloud CLI](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) の [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) プラグインを使用して、[Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) の例に従います。
* 上級者ユーザーは、直接 [REST API](https://{DomainName}/apidocs/rias) を呼び出せます。REST API を開始するには、[サンプル・コード](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)のチュートリアルに従ってください。

## 次のステップ

仮想プライベート・クラウドのネットワーキングをプロビジョンしたら、探索を進めてください。

* [仮想サーバー・インスタンスの作成および管理](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [セキュリティー・グループの使用](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [ネットワーク ACL の使用](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(ベータ) VPN の使用](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(ベータ) ロード・バランサーの使用](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
