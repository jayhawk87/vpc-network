---

copyright:
  years: 2019

lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# CLI を使用したネットワーク ACL のセットアップ
{ #setting-up-network-acls-using-the-cli}

{{site.data.keyword.cloud}} 仮想プライベート・クラウドに用意されているアクセス制御リスト (ACL) 機能を使用して、クラウド上の重要なビジネス・ワークロードに関連するすべての着信/発信トラフィックを制御できます。セキュリティー・グループと同様に、ACL も組み込みの仮想ファイアウォールです。セキュリティー・グループとは異なり、ACL のルールは、インスタンスとの間のトラフィックではなく、サブネットとの間のトラフィックを制御します。

次の表に、セキュリティー・グループと ACL の主な相違点を要約します。

|  | セキュリティー・グループ | ACL    |
|-------------|-----------------|---------|
| 制御レベル  | VSI インスタンス    | サブネット  |
| 状態   | ステートフル - 戻りトラフィックはデフォルトで許可されます | ステートレス - 戻りトラフィックはデフォルトで拒否されるため、明示的に許可する必要があります |
| サポートされているルール | 許可ルールのみ使用 | 許可ルールと拒否ルールを使用|
| ルールの適用方法 | すべてのルールが評価される | ルールが順番に処理される|
| 関連リソースとの関係 | 1 つのインスタンスに複数のセキュリティー・グループを関連付けることができる| 複数のサブネットに同一の ACL を関連付けることができる|

この資料では、VPC にネットワーク ACL を作成してサブネットを保護する方法を示す例を紹介します。

## ACL には API と CLI を使用可能

ACL のセットアップおよび管理は、API または CLI のどちらでも行えます。

* 各 API のパラメーター、要求本体、および応答の詳細については、[API リファレンス](https://{DomainName}/apidocs/rias)を参照してください。

* コマンドの詳細、CLI プラグインのインストール手順、VPC CLI を使用するための前提条件として行うべき手順については、こちらの [CLI の例](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)を参照してください。

## ACL ルールと ACL ルールの処理

ACL を有効にするには、インバウンド/アウトバウンドのネットワーク・トラフィックの処理方法を決定するルールを作成します。複数のインバウンド・ルールとアウトバウンド・ルールを作成できます。作成可能なルールの数に関する具体的な情報については、[割り当て量](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-quotas)を参照してください。

* インバウンド・ルールおよびアウトバウンド・ルールでは、指定したプロトコルとポートを使用する特定の送信元 IP 範囲からのトラフィックおよび特定の宛先 IP 範囲へのトラフィックを許可または拒否できます。  
* ACL ルールは順に処理されます。 優先度が低いルールは、優先度が高いルールが一致しない場合にのみ評価されます。
* インバウンド・ルールは、アウトバウンド・ルールとは分離されています。
* 現在サポートされているプロトコルは、TCP、UDP、および ICMP です。 **all** オプションを使用して_すべての_ プロトコルを指定することも、_その他の_ プロトコル (優先度が高いルールを指定する場合) を指定することもできます。

ACL ルールでの ICMP、TCP、および UDP プロトコルの使用に関する情報は、[インターネット通信プロトコルについて](https://{DomainName}/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols)を参照してください。

### サブネットへの ACL の付加

ACL をサブネットに付加するには、次の 2 つのオプションがあります。

* 新しいサブネットを作成し、付加する ACL を指定できます。ACL を指定しなかった場合は、デフォルトのネットワーク ACL が付加されます。デフォルトの ACL では、このサブネットを宛先とするすべてのインバウンド・トラフィックと、このサブネットから発信されるすべてのアウトバウンド・トラフィックが許可されます。
* 既存のサブネットに ACL を付加できます。このサブネットに別の ACL が既に付加されている場合は、その ACL が切り離されてから新しい ACL が付加されます。

## ACL デモの例

以下の例では、コマンド・ライン・インターフェース (CLI) を使用して、2 つの ACL を作成して 2 つのサブネットに関連付けることができます。シナリオは次のようになります。

![ACL のシナリオの例](images/vpc-acls.png)

図に示されているように、インターネットからの要求を処理する 2 つの Web サーバーと、非公開の 2 つのバックエンド・サーバーがあります。 この例では、これらのサーバーをそれぞれ 2 つの別個のサブネット (10.10.10.0/24 と 10.10.20.0/24) に配置し、Web サーバーがバックエンド・サーバーとデータを交換できるようにする必要があります。 また、バックエンド・サーバーからのアウトバウンド・トラフィックを限定的に許可することもできます。

### ルールの例

以下のルールの例に、前述の基本的なシナリオの ACL ルールをセットアップする方法を示します。

ベスト・プラクティスとして、粒度が細かいルールの優先度を、粒度が粗いルールより高くすることをお勧めします。例えば、サブネット 10.10.30.0/24 からのすべてのトラフィックをブロックするルールを優先度の高いルールにした場合、10.10.30.5 からのトラフィックを許可するという優先度が低くて粒度が細かいルールが適用されることはありません。{:note}

**サンプル・ルール ACL-1**:

| インバウンド/アウトバウンド| 許可/拒否 | 送信元/宛先 IP | プロトコル | ポート | 説明|
|--------------|-----------|------|------|------------------|-------|
| インバウンド | 許可 | 0.0.0.0/0 | TCP| 80 | インターネットからの HTTP トラフィックを許可する|
| インバウンド | 許可 | 0.0.0.0/0 | TCP | 443 | インターネットからの HTTPS トラフィックを許可する|
| インバウンド | 許可| 10.10.20.0/24 |all| すべて| バックエンド・サーバーが配置されているサブネット 10.10.20.0/24 からのすべてのインバウンド・トラフィックを許可する|
| インバウンド | 拒否| 0.0.0.0/0|all| すべて| その他のすべてのインバウンド・トラフィックを拒否する|
| アウトバウンド | 許可 | 0.0.0.0/0 | TCP|80 | インターネットへの HTTP トラフィックを許可する|
| アウトバウンド | 許可 | 0.0.0.0/0 | TCP|443 | インターネットへの HTTPS トラフィックを許可する|
| アウトバウンド | 許可| 10.10.20.0/24 |all| すべて| バックエンド・サーバーが配置されているサブネット 10.10.20.0/24 へのすべてのアウトバウンド・トラフィックを許可する|
| アウトバウンド | 拒否| 0.0.0.0/0|all| すべて| その他のすべてのアウトバウンド・トラフィックを拒否する|


**サンプル・ルール ACL-2**:

| インバウンド/アウトバウンド| 許可/拒否 | 送信元/宛先 IP | プロトコル| ポート | 説明|
|--------------|-----------|------|------|------------------|--------|
| インバウンド | 許可| 10.10.10.0/24 |all| すべて| Web サーバーが配置されているサブネット 10.10.10.0/24 からのすべてのインバウンド・トラフィックを許可する|
| インバウンド | 拒否| 0.0.0.0/0|all| すべて| その他のすべてのインバウンド・トラフィックを拒否する|
| アウトバウンド | 許可 | 0.0.0.0/0 | TCP| 80 | インターネットへの HTTP トラフィックを許可する|
| アウトバウンド | 許可 | 0.0.0.0/0 | TCP| 443 | インターネットへの HTTPS トラフィックを許可する|
| アウトバウンド | 許可| 10.10.10.0/24 |all| すべて| Web サーバーが配置されているサブネット 10.10.10.0/24 へのすべてのアウトバウンド・トラフィックを許可する|
| アウトバウンド | 拒否| 0.0.0.0/0|all| すべて| その他のすべてのアウトバウンド・トラフィックを拒否する|

この例は一般的なケースだけを示しています。 シナリオによっては、トラフィックをより細かく制御する必要があることがあります。

* ネットワーク管理者を任命して、リモート・ネットワークから 10.10.10.0/24 サブネットにアクセスして操作してもらうこともできます。 この場合は、インターネットからこのサブネットへの SSH トラフィックを許可する必要があります。
* 2 つのサブネット間で許可するプロトコル・スコープを絞り込むこともできます。

### 手順の例

以下の例の手順では、CLI を使用して VPC を作成するという前提条件となる手順を省略していますが、この手順を最初に実行しておく必要があります。詳細については、[CLI を使用した VPC の作成](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)を参照してください。


#### 手順 1. ACL を作成する

次の CLI コマンドで、`my_web_subnet_acl` および `my_backend_subnet_acl` という 2 つの ACL を作成できます。

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

新しく作成された ACL の ID が応答として返されます。後でコマンドで使用するので、その両方の ACL の ID を保存します。次のように、`webacl` および `bkacl` という変数を使用すると良いでしょう。

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### 手順 2. ACL のデフォルトのルールを取得する

新しいルールを追加する前に、インバウンドおよびアウトバウンドのデフォルト ACL ルールを取得します。これらの ACL ルールの前に新しいルールを挿入できます。

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

すべてのプロトコルのすべての IPv4 トラフィックを許可するインバウンドとアウトバウンドのデフォルトのルールが応答として返されます。

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

後でコマンドで使用するので、この両方の ACL ルールの ID を変数に保存します。例えば、次のように `inrule` および `outrule` という変数を使用します。

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### 手順 3. 説明に従って新規 ACL ルールを追加する

このセクションでは、最初にインバウンド・ルールを追加し、次にアウトバウンド・ルールを追加します。

デフォルトのインバウンド・ルールの前に新しいインバウンド・ルールを挿入します。

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

この手順を行うたびに、ACL ルールの ID を変数に保存します。後でコマンドで使用します。例えば、次のように `acl200` という変数を使用します。

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

次は、`acl200` にルールを追加します。

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

ACL のセットアップが完了するまでルールを追加していきます。各 ID を変数として保存します。

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

デフォルトのアウトバウンド・ルールの前に新しいアウトバウンド・ルールを挿入します。

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### 手順 4. 新しく作成した ACL を使用して 2 つのサブネットを作成する

サブネットを 2 つ作成し、新規サブネットのそれぞれに ACL を 1 つ関連付けます。 

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## コマンド・リストの説明

ACL に対して使用できるすべての VPC CLI コマンドのリストを表示するには、次のように入力します。

```
ibmcloud is network-acls
```
{: pre}

ACL とそのメタデータ (ルールを含む) を表示するには、次のように入力します。

```
ibmcloud is network-acl $webacl
```
{: pre}

インバウンドのデフォルトの ACL ルールを取得するには、次のように入力します。

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## インバウンドの `ping` ルールの例

ACL ルールの追加に関して、デフォルトのインバウンド・ルールの前に `ping` インバウンド・ルールを追加するサンプル・コマンドを次に示します。

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
