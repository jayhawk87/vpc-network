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

# VPC 内の Web サーバー間接続のセットアップ
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

この資料では、Python を使用して {{site.data.keyword.cloud}} VPC にある 2 つの Web サーバー間の通信をセットアップする手順の例を紹介します。これらの例から、同じゾーンおよび別のゾーンに通信サーバーを作成する方法がわかります。

## 前提条件

VPC の Web サーバー間の通信をセットアップする前に、2 つのサブネットを持つ VPC を 1 つ作成し、各サブネットに少なくとも 2 つのインスタンスを用意しておく必要があります。それらの VPC、サブネット、インスタンスの ID が必要になります。この構成のセットアップについての支援が必要な場合は、[CLI で VPC リソースを作成するためのガイド](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)の手順に従ってください。

## シナリオ 1: 同じゾーン内にあるが、サブネットが異なるサーバー間の接続

`vpc` は vpc ID を保持します
`subnet1` はサブネット 1 ID を保持します
`subnet2` はサブネット 2 ID を保持します


## パート 1: ゾーンおよび VPC が同じサーバー間の接続

パブリック・ゲートウェイに接続している同一サブネット内にインスタンスを 2 つ以上作成した後、それらの接続をテストできます。そのためには、いずれかのインスタンスで HTML コードを使用してサーバーをホストし、その他のインスタンスまたはコンピューターのブラウザーから、そのサーバーに対して `curl` を実行して HTML ファイルをダウンロードします。

パブリック・ゲートウェイが存在する同一サブネットに、`instance_1` と `instance_2` が存在し、セキュリティー・グループも同じであるとします。API、CLI、または UI を使用して、インスタンスでインバウンド・アクセスを許可するルールをセキュリティー・グループに追加する必要があります。`instance_1` でサーバーをホストする場合は、`instance_2` の浮動 IP に対するルールを追加する必要があります。例えば、`instance_1` ホスティング・サーバーに IPv4 `169.61.161.161` を、`instance_2` に IPv4 `169.61.161.235` を設定し、instance_2 で `instance_1` からファイルを取得できるか試してみるとします。次のようになります。

![セキュリティー・グループの新しいルール](images/security-group-ui-ex1.png)

* すべてのセキュリティー・グループをリストする場合は、`ibmcloud is sgs` を実行します。
* 前述の結果が得られる API 要求を次に示します。

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. 基礎的な `index.html` ファイルを `instance_1` に作成します。基礎的な HTML ファイルの例を以下に示します。

```
<!DOCTYPE html>
<html>
<body>

<h1>インスタンスに正常に接続しました</h1>
<p>Hello, world!</p>

</body>
</html>
```
{:pre}

2. このファイルを置いたディレクトリーで、次のコマンドを実行します。`python -m SimpleHTTPServer <port number>`. デフォルト・ポート 8000 が既に使用されている場合は、別のポートを使用できます。

3. `instance_2` に SSH で接続します。

浮動 IP アドレスを調べるには、コマンド `ibmcloud is ips` を使用し、コマンド `ibmcloud is ip $floating_ip_id` を使用して `floating_ip_id` を見つけてください。パブリック・アドレスがわかるはずです。

4. `instance_1` の IP アドレスを特定します。

5. `curl -X GET http://IP_address:Port_number/index.html` を使用して、ホスティング・サーバー `instance_1` に `index.html` のダウンロードを要求します (例: `curl -X GET http://169.61.161.161:8000/index.html`)。

6. `index.html` の出力が表示されます。

7. (ローカル・コンピューターのブラウザーを使用してサーバーにアクセスする場合) 以下の図に示すように、すべてのインバウンド・トラフィックに対するルールを追加します。

![すべてのインバウンドに対するセキュリティー・グループの新しいルール](images/security-group-ui-ex2.png)

* すべてのプロトコルおよびすべてのソースを許可するルールをセキュリティー・グループに追加するには、次のようにします

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**詳細な要求例:**

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "port_max": 22,
  "port_min": 22,
  "protocol": "tcp",
  "remote": {
    "cidr_block": "131.121.0.0/24"
  }
}'
```
{:pre}

8. Web ブラウザーを開き、`http://IP_address:Port_number/index.html` と入力します

9. インデックス・ファイルが HTML 形式で表示されます。

## パート 2: ゾーンおよび VPC が異なるサーバー間の接続

パブリック・ゲートウェイをセットアップしていないもう 1 つのサブネットのインスタンスを、インターネット・アクセスを利用して対象インスタンスに接続します。次の 2 種類のインスタンスをセットアップできます。

* パブリック・ゲートウェイを使用して、インターネットからの要求を処理する Web サーバー
* アプリケーションを実行し、重要なデータを保管するバックエンド・サーバー。パブリック・ゲートウェイは無効で、内部トラフィックのみを送信します

セキュリティー・グループ (SG) を仮想サーバー・インスタンス (VSI) に適用し、ネットワーク ACL をサブネットに適用します。

そのため、このタスクをすべて完了するには、VSI の作成、サブネットの作成、SG と ACL のルールの作成を行い、その後 SG をインスタンスのネットワーク・インターフェースに付加し、ネットワーク ACL を特定のサブネットに付加する必要があります。

* `subnet1` セキュリティー・グループ内のすべてのルールをリストするには、次を実行します

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. パート 1 のサブネット操作のセキュリティー・グループの`すべてのインバウンド・トラフィックを許可する`ルールを削除します。

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. 新しい VPC を作成し、パブリック・ゲートウェイに接続しないサブネットを作成します。

3. この新しいサブネットに、パブリック・ゲートウェイを使用せずに (インターネット接続なし) `instance_3` を作成します。バックエンド処理のために、このインスタンスには CPU コアとメモリーを追加で割り当てることもできます。

4. パート 1 のセキュリティー・グループに `instance_3` 用の新しいルールを追加します。例えば、`instance_3` は浮動 IP `169.61.181.25` を持ちます。

![他のサブネット・インバウンドからのインスタンスに関するセキュリティー・グループの新しいルール](images/security-group-ui-ex3.png)

5. `curl -X GET http://IP_address:Port_number/index.html` という形式のコマンドを使用して、ホスティング・サーバー instance_1 に `index.html` のダウンロードを要求します (例: `curl -X GET http://169.61.161.161:8000/index.html`)。

6. `index.html` の出力が表示されます。

つまり、インスタンス 3 はインスタンス 1 との間に接続を確立しましたが、インスタンス 2 との間には確立していません。

## 次のステップ

[VPC で ACL を使用する方法](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)に記載しているように、インバウンド・トラフィックを制御する ACL ルールを作成してサーバーを保護できます。
