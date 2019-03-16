---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (ベータ) VPC での VPN の使用
{: #--beta-using-vpn-with-your-vpc}

{{site.data.keyword.cloud}} VPC の VPN サービスを使用すると、プライベート・ネットワークをセキュアな方法で接続できます。VPC とオンプレミスのプライベート・ネットワークとの間、または別の VPC との間で、VPN を使用して IPsec サイト間トンネルをセットアップできます。

現行の {{site.data.keyword.cloud}} VPC リリースでは、ポリシー・ベースのルーティングのみがサポートされています。

**このサービスは、現時点ではベータ版として提供されています。 ベータ期間中に使用したリソースは、ベータ終了以降は使用できなくなり、またサポートは提供されなくなります。 ご意見やご質問は、IBM Cloud 営業担当員にお寄せください。**

## フィーチャー

* IKEv1 および IKEv2
* 認証アルゴリズム: `md5`、`sha1`、`sha256`
* 暗号化アルゴリズム: `3des`、`aes128`、`aes256`
* Diffie-Hellman (DH) グループ: 2、5、14
* IKE ネゴシエーション・モード: main
* IPSec 変換プロトコル: ESP
* IPSec カプセル化モード: tunnel
* Perfect Forward Secrecy (PFS)
* 非活動ピア検出
* ルーティング: ポリシー・ベース
* 認証モード: 事前共有鍵
* HA サポート

## 使用可能な API

次のセクションでは、IBM Cloud VPC 環境で VPN に使用できる API について詳しく説明します。 詳しくは、[VPC REST API](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies) ページを参照してください。

### VPN ゲートウェイと VPN 接続

| 説明 | API |
|----------------------------|-------------|
| VPN ゲートウェイの作成 | POST /vpn_gateways |
| 複数の VPN ゲートウェイの取得 | GET /vpn_gateways |
| 1 つの VPN ゲートウェイの取得 | GET /vpn_gateways/{id} |
| VPN ゲートウェイの削除 | DELETE /vpn_gateways/{id} |
| VPN ゲートウェイの更新 | PATCH /vpn_gateways/{id} |
| 新規 VPN 接続の作成 | POST /vpn_gateways/{vpn_gateway_id}/connections |
| 複数の VPN 接続の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections |
| 1 つの VPN 接続の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 接続の削除 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 接続の更新 | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| VPN 接続のすべてのローカル CIDR の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| VPN 接続からのローカル CIDR の削除 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続に特定のローカル CIDR が存在するかどうかの確認 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続のローカル CIDR の設定 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続のすべてのピア CIDR の取得 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| VPN 接続からのピア CIDR の削除 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続に特定のピア CIDR が存在するかどうかの確認 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| VPN 接続のピア CIDR の設定 | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE ポリシー

| 説明 | API |
|-----------------------------|--------------|
| すべての IKE ポリシーの取得 | GET /ike_policies |
| IKE ポリシーの作成 | POST /ike_policies |
| IKE ポリシーの削除 | DELETE /ike_policies/{id} |
| IKE ポリシーの取得 | GET /ike_policies/{id} |
| IKE ポリシーの更新 | PATCH /ike_policies/{id} |
| 指定された IKE ポリシーを使用するすべての接続の取得 | GET /ike_policies/{id}/connections |

### IPsec ポリシー

| 説明 | API |
|---------------------|-------------|
| すべての IPSec ポリシーの取得 | GET /ipsec_policies |
| IPSec ポリシーの作成 | POST /ipsec_policies |
| IPSec ポリシーの削除 | DELETE /ipsec_policies/{id} |
| IPSec ポリシーの取得 | GET /ipsec_policies/{id} |
| IPSec ポリシーの更新 | PATCH /ipsec_policies/{id} |
| 指定された IPSec ポリシーを使用するすべての接続の取得 | GET /ipsec_policies/{id}/connections |

## VPN の例

以下の例では、VPN を使用して 2 つの VPC を接続できます。つまり、2 つの別個の VPC 内のサブネットを、単一のネットワークであるかのように接続できます。 各サブネットの IP アドレスがオーバーラップしてはなりません。
シナリオは以下のようになります (各 VPC いくつかの VM が追加されています):
![VPN のシナリオの例](images/vpc-vpn.png)

### 手順の例

以下の手順の例では、IBM Cloud API または CLI を使用して VPC を作成するという前提条件となる手順を省略しています。 詳しくは、[概要](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure)および [API を使用した VPC のセットアップ](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)を参照してください。

オプションで、UI を使用して VPN ゲートウェイを作成できます。 この手順については、[コンソール・チュートリアルの資料](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)を参照してください。

#### 手順 1. VPC サブネットに VPN ゲートウェイを作成する

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

サンプル出力:
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

後続のステップのために次のフィールドが保存されていることを確認します。
* `id`。 これが VPN ゲートウェイの ID です。これ以降は、`$gwid1` として参照します。
* `address`。 これが VPN ゲートウェイのパブリック IP アドレスです。これ以降は、`$gwaddress1` として参照します。

注: VPN ゲートウェイの作成中はゲートウェイの状況は `pending` と表示され、作成が完了すると状況が `available` になります。 作成にはしばらく時間がかかる場合があります。 次のコマンドにより、その状況を確認できます。

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### 手順 2. 2 番目の VPN ゲートウェイを別の VPC に作成する

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

サンプル出力:
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

後続のステップのために次のフィールドを必ず保存してください。
* `id`。 これが VPN ゲートウェイの ID です。これ以降は、`$gwid2` として参照します。
* `address`。 これが VPN ゲートウェイのパブリック IP アドレスです。これ以降は、`$gwaddress2` として参照します。


#### 手順 3. 1 番目の VPN ゲートウェイから 2 番目の VPN ゲートウェイへの VPN 接続を作成する。このとき、`local_cidrs` に VPC 1 のサブネットを、`peer_cidrs` に VPC 2 のサブネットを設定する。

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01 \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

サンプル出力:
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 手順 4. 2 番目の VPN ゲートウェイから 1 番目の VPN ゲートウェイへの VPN 接続を作成する。このとき、`local_cidrs` に VPC 2 のサブネットを、`peer_cidrs` に VPC 1 のサブネットを設定する。

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-01-01 \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

サンプル出力:
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### 手順 5. 接続を確認する

VPN 接続が確立されたら、サブネット 1 からサブネット 2 のインスタンスにアクセスできます (逆方向も可能です)。

VPN 接続の状況は、以下のようにして確認できます。
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

サンプル出力:
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## 割り当て量

VPN 割り当て量については、[VPC 割り当て量](/docs/infrastructure/vpc?topic=vpc-quotas#vpn-quotas)のトピックを参照してください。

## ポリシーのオートネゴシエーション

VPN 接続の構成での IKE ポリシーと IPSec ポリシーの使用はオプションです。 ポリシーが選択されていない場合、プロセスに対してデフォルトのプロポーザルが自動的に選択されます。この機能は_オートネゴシエーション_ と呼ばれます。 オートネゴシエーションの一部として使用されるプロポーザルを以下に示します。

IBM Cloud はイニシエーターとして **IKEv2** を使用します。IBM Cloud はレスポンダーとして **IKEv1** と **IKEv2** の両方を受け入れます。
{: note}

### IKE オートネゴシエーション (フェーズ 1)

次に示す暗号化、認証、および DH グループのオプションは、自由に組み合わせて使用できます。

|    | 暗号化 | 認証 | DH グループ |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec オートネゴシエーション (フェーズ 2)

次に示す暗号化と認証のオプションは、自由に組み合わせて使用できます。

|    | 暗号化 | 認証 | DH グループ |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabled  |
| 2  | aes256 | sha256 | disabled  |
| 3  | 3des   | md5    | disabled  |

## FAQ

**VPN ゲートウェイを作成するときに、VPN 接続を同時に作成できますか?**

API または CLI を使用する場合は、VPN ゲートウェイを作成した後で VPN 接続を作成する必要があります。 IBM Cloud コンソールでは、ゲートウェイと接続を同時に作成できます。

**VPN 接続が接続されている VPN ゲートウェイを削除した場合、接続はどうなりますか?**

VPN 接続は VPN ゲートウェイと共に削除されます。

**VPN ゲートウェイまたは VPN 接続を削除した場合、IKE ポリシーまたは IPSec ポリシーは削除されますか?**

削除されません。IKE ポリシーと IPSec ポリシーは複数の接続に適用できます。

**VPN ゲートウェイが配置されているサブネットを削除すると、そのゲートウェイはどうなりますか?**

インスタンス (VPN ゲートウェイを含む) が存在するサブネットは削除できません。

**デフォルトの IKE ポリシーと IPSec ポリシーはありますか?**

ポリシー ID (IKE または IPSec) を参照せずに VPN 接続を作成すると、オートネゴシエーションが使用されます。

**VPN for VPC では HA 構成がサポートされますか?**

はい、サポートされます。
