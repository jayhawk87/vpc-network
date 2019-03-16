---



copyright:
  years: 2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (ベータ) IBM Cloud VPC でのロード・バランサーの使用
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

ロード・バランサー・サービスは、{{site.data.keyword.cloud}} VPC の同一リージョン内の複数のサーバー・インスタンス間にトラフィックを分散させます。

**このサービスは、現時点ではベータ版として提供されています。 ベータ期間中に使用したリソースは、ベータ終了以降は使用できなくなり、またサポートは提供されなくなります。 ご意見やご質問は、IBM Cloud 営業担当員にお寄せください。**

## パブリック・ロード・バランサー
{: #public-load-balancer}

ロード・バランサー・サービス・インスタンスには、パブリック・アクセスが可能な完全修飾ドメイン・ネーム (FQDN) が割り当てられます。{{site.data.keyword.cloud}} VPC のロード・バランサー・サービスの背後でホストされているアプリケーションにアクセスする際にはこれを使用する必要があります。 このドメイン・ネームは、1 つ以上のパブリック IP アドレスに登録できます。

パブリック IP アドレスの場所と数は、メンテナンスとスケーリングの作業を行っていく過程で変更される可能性がありますが、お客様はこのような変化を意識せずに操作を続けることができます。 アプリケーションをホストするバックエンド・サーバー・インスタンス (VSI) は、同一 VPC 内の同一リージョンで実行する必要があります。

## フロントエンド・リスナーおよびバックエンド・プール
最大 10 個のフロントエンド・リスナー (アプリケーション・ポート) を定義し、それらをバックエンド・アプリケーション・サーバーの対応するバックエンド・プールにマップできます。 ロード・バランサー・サービス・インスタンスに割り当てられている完全修飾ドメイン・ネーム (FQDN) とフロントエンド・リスナーのポートは、パブリック・インターネットに公開されます。 着信ユーザー要求はこれらのポートで受信されます。

サポートされるフロントエンド・リスナー・プロトコルは次のとおりです。
* HTTP
* HTTPS
* TCP

サポートされるバックエンド・プール・プロトコルは次のとおりです。
* HTTP
* TCP

バックエンド・サーバーとのプレーン・テキスト HTTP 通信を可能にするため、着信 HTTPS トラフィックがロード・バランサーで終了します。

バックエンド・プールには最大 50 個のサーバー・インスタンスを接続できます。 接続される各サーバー・インスタンスではポートを構成しておく必要があります。 このポートは、フロントエンド・リスナー・ポートと同じ場合もあれば同じでない場合もあります。

ポート範囲 56500 から 56520 は管理目的用に予約されています。これらのポートは、フロントエンド・リスナー・ポートとしては使用できません。
{: note}

## ロード・バランシングの方式
{: #load-balancing-methods}

バックエンド・アプリケーション・サーバー間でトラフィックを分散するために、以下の 3 つのロード・バランシング方式が使用可能です。

* **ラウンドロビン:** ラウンドロビンがデフォルトのロード・バランシング方式です。この方式では、ロード・バランサーは、着信クライアント接続をラウンドロビン形式でバックエンド・サーバーに転送します。 その結果として、すべてのバックエンド・サーバーは、ほぼ同数のクライアント接続を受信します。

* **重み付きラウンドロビン:** この方式では、ロード・バランサーは、バックエンド・サーバーに割り当てられた重みに比例して、着信クライアント接続をバックエンド・サーバーに転送します。 各サーバーには、デフォルトの重み 50 が割り当てられます。
この重みは、0 から 100 の間の任意の値にカスタマイズできます。

例えば、3 台のアプリケーション・サーバー A、B、および C の重みがそれぞれ 60、60、および 30 にカスタマイズされている場合、サーバー A と B は同数の接続を受信し、サーバー C はその半数の接続を受信します。

* **最小接続:** この方式では、特定の時間に処理している接続数が最も少ないサーバー・インスタンスが、次のクライアント接続を受信します。

**これらの方式のその他の特性:**

* サーバーの重みを「0」にリセットするということは、そのサーバーには新しい接続が転送されず、既存のトラフィックはすべて、アクティブである限りフローし続けることを意味します。 重み「0」を使用すると、サーバーを正常終了し、サービス・ローテーションからそのサーバーを削除するのに役立ちます。
* サーバーの重みの値は、「重み付きラウンドロビン」方式を使用する場合にのみ適用されます。ラウンドロビンおよび最小接続のロード・バランシング方式では、これらの値は無視されます。

## ヘルス・チェック
{: #health-checks}

バックエンド・プールでは、ヘルス・チェックの定義は必須です。

ロード・バランサーは定期的にヘルス・チェックを実行して、バックエンド・ポートの正常性をモニターし、それに応じてそれらのポートにクライアント・トラフィックを転送します。 正常でないことが検出されたバックエンド・サーバー・ポートには、新しい接続は転送されません。 ロード・バランサーは正常でないポートの正常性を引き続きモニターします。ポートが再び正常な状態になる (つまり、ヘルス・チェックの試行が 2 回連続して合格になる) と、その使用が再開されます。

HTTP ポートおよび TCP ポートのヘルス・チェックは、以下のように実行されます。

* **HTTP:** 事前指定 URL に対する `HTTP GET` 要求がバックエンド・サーバー・ポートに送信されます。 `200 OK` 応答を受信すると、サーバー・ポートに正常のマークが付けられます。 UI ではデフォルトの `GET` ヘルス・パスは「/」ですが、これはカスタマイズできます。

* **TCP:** ロード・バランサーは、指定された TCP ポートでバックエンド・サーバーとの TCP 接続をオープンします。 接続試行が成功するとサーバー・ポートに正常のマークが付けられ、接続がクローズされます。

デフォルトのヘルス・チェック間隔は 5 秒、ヘルス・チェック要求に対するデフォルトのタイムアウトは 2 秒、デフォルトの再試行回数は 2 回です。
{: note}

## SSL オフロードおよび必要な許可

すべての着信 HTTPS 接続について、ロード・バランサー・サービスは SSL 接続を終了し、バックエンド・サーバー・インスタンスとのプレーン・テキスト HTTP 通信を確立します。 この手法によって、バックエンド・サーバー・インスタンスでは、CPU 集中型の SSL ハンドシェークおよび暗号化/復号のタスクを行う必要がなくなり、アプリケーション・トラフィックの処理にすべての CPU サイクルを使用できるようになります。

ロード・バランサーで SSL オフロード・タスクを実行するには、SSL 証明書が必要です。SSL 証明書は、[IBM Certificate Manager ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} で管理できます。

証明書マネージャー・インスタンス内の SSL 証明書にロード・バランサーからアクセスする必要がある場合は、SSL 証明書を格納している証明書マネージャー・インスタンスへのアクセス権をロード・バランサー・サービス・インスタンスに付与する許可を作成してください。このような許可は、[ID およびアクセスの許可 ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](/docs/services/iam/#/authorizations){: new_window} を使用して管理できます。ソース・サービスとして**「インフラストラクチャー・サービス」**、リソース・タイプとして**「Load Balancer for VPC」**、ターゲット・サービスとして**「Certificate Manager」**を選択し、**「ライター」**サービス・アクセス・ロールを割り当ててください。ロード・バランサーを作成するには、ソース・リソース・インスタンスに対して**「すべてのリソース・インスタンス」**許可を付与する必要があります。ターゲット・サービス・インスタンスは**「すべてのインスタンス」**の場合もあれば、特定の証明書マネージャー・リソース・インスタンスの場合もあります。

必要な許可が削除されると、ロード・バランサーでエラーが発生する可能性があります。
{: note}

## ID およびアクセスの管理 (IAM)

**Load Balancer for VPC** インスタンスのアクセス・ポリシーを構成できます。ユーザー・アクセス・ポリシーを管理するには、[ID およびアクセスの許可 ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](/docs/services/iam/#/authorizations){: new_window} を参照してください。

### ユーザーのリソース・グループ・アクセス・ポリシーの構成

ロード・バランサーを作成するには、リソース・グループにアクセスする必要があります。 ロード・バランサーを作成するユーザーには、指定されたリソース・グループに対する適切なアクセス権限が必要です。リソース・グループが指定されていない場合には、デフォルトのリソース・グループに対する適切なアクセス権限が必要です。

1. **「管理」>「アカウント」>「ユーザー」**に移動します。 IBM Cloud アカウントへのアクセス権限を持つユーザーのリストが表示されます。
2. アクセス・ポリシーの割り当て対象ユーザーの名前を選択します。 ユーザーが表示されていない場合は、**「ユーザーの招待」**をクリックして、IBM Cloud アカウントにユーザーを追加します。
3. **「アクセス権限の割り当て」**を選択します。
4. **「リソース・グループ内のアクセス権限の割り当て」**を選択します。
5. **「リソース・グループ」**ドロップダウン・リストから、必要なリソース・グループを選択します。
6. **「リソース・グループへのアクセス権限の割り当て」**ドロップダウン・リストから、必要なアクセス権限を選択します。
7. **「サービス」**ドロップダウン・リストから、必要なサービスを選択します。
9. **「割り当て」**を選択して、ユーザーにリソース・グループ・アクセス・ポリシーを割り当てます。

### ユーザーのリソース・アクセス・ポリシーの構成

| プラットフォームのアクセス役割 | ロード・バランサーのアクション |
|-------------|-----|
| 管理者 | ロード・バランサーの作成/表示/編集/削除 |
| エディター | ロード・バランサーの作成/表示/編集/削除 |
| ビューアー | ロード・バランサーの表示 |

1. **「管理」>「アカウント」>「ユーザー」**に移動します。 IBM Cloud アカウントへのアクセス権限を持つユーザーのリストが表示されます。
2. アクセス・ポリシーの割り当て対象ユーザーの名前を選択します。 ユーザーが表示されていない場合は、**「ユーザーの招待」**をクリックして、IBM Cloud アカウントにユーザーを追加します。
3. **「アクセス権限の割り当て」**を選択します。
4. **「リソースへのアクセス権限の割り当て」**を選択します。
5. **「サービス」**ドロップダウン・リストから**「インフラストラクチャー・サービス」**を選択します。
6. **「リソース・タイプ」**ドロップダウン・リストから**「Load Balancer for VPC」**を選択します。
7. **「ロード・バランサー ID (Load Balancer ID)」**ドロップダウン・リストから、ロード・バランサー・インスタンス ID を選択するか、デフォルト値の**「すべてのロード・バランサー」**を使用します。
8. プラットフォームのアクセス役割をユーザーに割り当てます。
9. **「割り当て」**を選択して、ユーザーにアクセス・ポリシーを割り当てます。

## 使用可能な API

API 呼び出しを行うには、何らかの形式の REST クライアントを使用する必要があります。 例えば、既存のすべてのロード・バランサーを取得するには `curl` コマンドを使用できます。

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

次のセクションで、VPC 環境でロード・バランサーに使用できる API について詳しく説明します。完全な仕様については、[RIAS API リファレンス](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers)を参照してください。

| 説明 | API |
|-------------|-----|
| ロード・バランサーの作成とプロビジョン | POST /load_balancers |
| すべてのロード・バランサーの取得 | GET /load_balancers |
| 1 つのロード・バランサーの取得 | GET /load_balancers/{id} |
| ロード・バランサーの削除 | DELETE /load_balancers/{id} |
| ロード・バランサーの更新 | PATCH /load_balancers/{id} |
| リスナーの作成 | POST /load_balancers/{id}/listeners |
| ロード・バランサーのすべてのリスナーの取得 | GET /load_balancers/{id}/listeners |
| リスナーの取得 | GET /load_balancers/{id}/listeners/{listener_id} |
| リスナーの削除 | DELETE /load_balancers/{id}/listeners/{listener_id} |
| リスナーの更新 | PATCH /load_balancers/{id}/listeners/{listener_id} |
| プールの作成 | POST /load_balancers/{id}/pools |
| ロード・バランサーのすべてのプールの取得 | GET /load_balancers/{id}/pools |
| プールの取得 | GET /load_balancers/{id}/pools/{pool_id} |
| プールの削除 | DELETE /load_balancers/{id}/pools/{pool_id} |
| プールの更新 | PATCH /load_balancers/{id}/pools/{pool_id} |
| メンバーの作成 | POST /load_balancers/{id}/pools/{pool_id}/members |
| プールのすべてのメンバーの取得 | GET /load_balancers/{id}/pools/{pool_id}/members |
| メンバーの取得 |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| プールからのメンバーの削除 | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| メンバーの更新 | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| プールのメンバーの更新 | PUT /load_balancers/{id}/pools/{pool_id}/members |

## ロード・バランサーの例

以下の例では、API を使用してロード・バランサーを作成します。これは、ポート `80` で listen する Web アプリケーションを実行する 2 つの VPC サーバー・インスタンス (`192.168.100.5` および `192.168.100.6`) の前に配置されます。 ロード・バランサーにはフロントエンド・リスナーがあるので、HTTPS を使用して Web アプリケーションに安全にアクセスできます。 次いでその API を使用して、作成されたロード・バランサー・インスタンスの詳細を取得し、そのロード・バランサー・インスタンスを削除できます。

### 手順の例

以下の手順の例では、[IBM Cloud UI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console)、[CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)、または [RIAS API](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) を使用して VPC、サブネット、およびインスタンスをプロビジョンするという前提条件となる手順を省略しています。 


このロード・バランサーの例の手順は、[CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference) を使用して実行することもできます。
{: note}

#### 手順 1. リスナー、プール、および接続するサーバー・インスタンス (メンバー) を指定してロード・バランサーを作成する

```bash
curl -H "Authorization: $iam_token" -X POST
$rias_endpoint/v1/load_balancers?version=2019-01-01 \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

サンプル出力:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d-7ce5-413b-aae4-90a2a88b7872.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

次の手順で使用するために、ロード・バランサーの ID を変数 `lbid` などに保存してください。

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### 手順 2. ロード・バランサーを取得する

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

ロード・バランサーのプロビジョニングにある程度時間がかかることを想定しておいてください。準備ができると、次の出力例に示されているように、`online` および `active` 状況に設定されます。

サンプル出力:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

#### 手順 3. ロード・バランサーを削除する

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## FAQ

このセクションでは、**Load Balancer for VPC** サービスについてよくある質問への回答を記載しています。

### 自分のロード・バランサーに対して別の DNS 名を使用できますか。

ロード・バランサーに自動的に割り当てられる DNS 名はカスタマイズできません。ただし、使用したい DNS 名を、自動的に割り当てられたロード・バランサー DNS 名に差し向ける CNAME (Canonical Name) レコードを追加できます。例えば、ロード・バランサー ID が `dd754295-e9e0-4c9d-bf6c-58fbc59e5727` の場合、自動的に割り当てられるロード・バランサー DNS 名は `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud` です。優先的に `www.myapp.com` という DNS 名を使用したいとします。 この場合、`myapp.com` の管理に使用する DNS プロバイダーを介して CNAME レコードを追加し、その中で、`www.myapp.com` がロード・バランサー DNS 名 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud` をポイントするように指定することができます。

### ロード・バランサー・サービスで定義できるフロントエンド・リスナーの最大数はいくつですか?

10 個です。

### バックエンド・プールに接続できるサーバー・インスタンスの最大数はいくつですか?

50 個です。

### 内部のクライアントだけがアクセスできる、内部専用のプライベート・ロード・バランサーを作成することはできますか?   

いいえ、現時点ではできません。

### ロード・バランサーは水平方向にスケーラブルですか?  

いいえ、現時点では対応していません。

### ロード・バランサーのデプロイに使用するサブネットで ACL またはセキュリティー・グループを使用している場合は、どうしたらよいですか?

構成されているリスナー・ポートおよび管理ポート (ポート 56500 から 56520) の着信トラフィックを許可するために、適切な ACL またはセキュリティー・グループ・ルールが設定されていることを確認する必要があります。 ロード・バランサーとバックエンド・インスタンス間のトラフィックも許可する必要があります。

### エラー・メッセージ`「証明書インスタンスが見つかりませんでした (certificate instance not found)」`を受け取るのはなぜですか?

* 証明書インスタンス CRN が無効である可能性があります。
* サービス間許可を付与していない可能性があります。この資料の **SSL オフロード**のセクションを参照してください。

### `401 Unauthorized Error` コードが表示されるのはなぜですか?

ユーザーの次のアクセス・ポリシーを確認してください。
* ロード・バランサー・リソース・タイプ・アクセス・ポリシー
* リソース・グループ・アクセス・ポリシー

### ロード・バランサーが `maintenance_pending` 状態になるのはなぜですか?

ロード・バランサーは、さまざまなメンテナンス・アクティビティーの実施中に `maintenance_pending` 状態になります。次に例を示します。
* 脆弱性に対処してセキュリティー・パッチを適用するためのローリング・アップグレード
* リカバリー・アクティビティー

### プロビジョン中に複数のサブネットを選択する必要があるのはなぜですか?

ロード・バランサー・アプライアンスは、選択したサブネットにデプロイされます。可用性と冗長性を強化するため、異なるゾーンのサブネットを選択することを強くお勧めします。
