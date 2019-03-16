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

# （測試版）對您的 VPC 使用 VPN
{: #--beta-using-vpn-with-your-vpc}

{{site.data.keyword.cloud}} VPC VPN 服務可讓您以安全的方式連接專用網路。您可以使用 VPN 在您的 VPC 與內部部署的專用網路或另一個 VPC 之間設定 IPsec 端對端通道。

對於現行 {{site.data.keyword.cloud}} VPC 版本，僅支援原則型遞送。

**此服務現在屬於「測試版」。測試版期間使用的資源在「測試版」結束之後將無法使用，且不提供任何支援。如有任何意見和問題，請直接洽詢 IBM Cloud 業務代表。**

## 特性

* IKEv1 和 IKEv2
* 鑑別演算法：`md5`、`sha1`、`sha256`
* 加密演算法：`3des`、`aes128`、`aes256`
* Diffie-Hellman (DH) 群組：2、5、14
* IKE 協議模式：main
* IPSec 轉換通訊協定：ESP
* IPSec 封裝模式：tunnel
* 完整轉遞保密 (PFS)
* 無回應對等節點偵測
* 遞送：原則型
* 鑑別模式：預先共用金鑰
* HA 支援

## 可用的 API

下一節提供有關您可在 IBM Cloud VPC 環境中用於 VPN 之 API 的詳細資料。如需詳細資料，請參閱 [VPC REST API](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies) 頁面。

### VPN 閘道及 VPN 連線

| 說明 | API |
|----------------------------|-------------|
| 建立VPN 閘道 | POST /vpn_gateways |
| 擷取 VPN 閘道 | GET /vpn_gateways |
| 擷取 VPN 閘道 | GET /vpn_gateways/{id} |
| 刪除 VPN 閘道 | DELETE /vpn_gateways/{id} |
| 更新 VPN 閘道 | PATCH /vpn_gateways/{id} |
| 建立新的 VPN 連線 | POST /vpn_gateways/{vpn_gateway_id}/connections |
| 擷取 VPN 連線 | GET /vpn_gateways/{vpn_gateway_id}/connections |
| 擷取 VPN 連線 | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| 刪除 VPN 連線 | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| 更新 VPN 連線 | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| 擷取 VPN 連線的所有本端 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| 刪除 VPN 連線的本端 CIDR | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 檢查 VPN 連線上是否有特定的本端 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 在 VPN 連線上設定本端 CIDR | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| 擷取 VPN 連線的所有對等節點 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| 刪除 VPN 連線的對等節點 CIDR | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| 檢查 VPN 連線上是否有特定的對等節點 CIDR | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| 在 VPN 連線上設定對等節點 CIDR | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE 原則

| 說明 | API |
|-----------------------------|--------------|
| 擷取所有 IKE 原則 | GET /ike_policies |
| 建立 IKE 原則 | POST /ike_policies |
| 刪除 IKE 原則 | DELETE /ike_policies/{id} |
| 擷取 IKE 原則 | GET /ike_policies/{id} |
| 更新 IKE 原則 | PATCH /ike_policies/{id} |
| 擷取所有使用指定 IKE 原則的連線 | GET /ike_policies/{id}/connections |

### IPsec 原則

| 說明 | API |
|---------------------|-------------|
| 擷取所有 IPSec 原則 | GET /ipsec_policies |
| 建立 IPSec 原則 | POST /ipsec_policies |
| 刪除 IPSec 原則 | DELETE /ipsec_policies/{id} |
| 擷取 IPSec 原則 | GET /ipsec_policies/{id} |
| 更新 IPSec 原則 | PATCH /ipsec_policies/{id} |
| 擷取所有使用指定 IPsec 原則的連線 | GET /ipsec_policies/{id}/connections |

## VPN 範例

在下列範例中，您可以使用 VPN 連接兩個 VPC，這表示您可以將兩個不同 VPC 中的子網路連接，彷彿它們是一個單一網路。子網路的 IP 位址不得重疊。
情境範例如下（每個 VPC 新增一些 VM）：![VPN 情境範例](images/vpc-vpn.png)

### 範例步驟

接下來的範例步驟會跳過使用 IBM Cloud API 或 CLI 來建立 VPC 的必要步驟。如需相關資訊，請參閱[開始使用](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure)及[使用 API 的 VPC 設定](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)。

您也可以選擇使用使用者介面來建立 VPN 閘道。您可以在[主控台指導教學文件](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)中找到相關步驟。

#### 步驟 1. 在 VPC 子網路中建立 VPN 閘道

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

輸出範例：
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

請務必儲存下列欄位供後續步驟使用。
* `id`。這是 VPN 閘道 ID，將之稱為 `$gwid1`。
* `address`。這是 VPN 閘道的公用 IP 位址，將稱之為 `$gwaddress1`。

附註：建立 VPN 閘道時，閘道狀態會顯示為 `pending`，建立完成之後，狀態會變成 `available`。建立作業可能需要一些時間。您可以使用下列指令來檢查其狀態：

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### 步驟 2. 在不同的 VPC 上建立第二個 VPN 閘道

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

輸出範例：
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

請務必儲存下列欄位供後續步驟使用。
* `id`。這是 VPN 閘道 ID，將之稱為 `$gwid2`。
* `address`。這是 VPN 閘道的公用 IP 位址，將稱之為 `$gwaddress2`。


#### 步驟 3. 建立從第一個 VPN 閘道到第二個 VPN 閘道的 VPN 連線，將 `local_cidrs` 設為第一個 VPC 上的子網路，並將 `peer_cidrs` 設為第二個 VPC 上的子網路。

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

輸出範例：
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

#### 步驟 4. 建立從第二個 VPN 閘道到第一個 VPN 閘道的 VPN 連線，將 `local_cidrs` 設為第二個 VPC 上的子網路，並將 `peer_cidrs` 設為第一個 VPC 上的子網路。

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

輸出範例：
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

#### 步驟 5. 驗證連線功能

建立 VPN 連線之後，您就可以從第一個子網路連接第二個子網路上的實例，反之亦然。

您可以檢查 VPN 連線的狀態，如下所示：
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

輸出範例：
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

## 配額

請參閱 [VPC 配額](/docs/infrastructure/vpc?topic=vpc-quotas#vpn-quotas)主題，以瞭解 VPN 配額。

## 原則自動協調

可以選擇使用 IKE 和 IPSec 原則來配置 VPN 連線。若未選取任何原則，則會自動針對處理程序選擇預設提案，稱之為_自動協調_。用來作為自動協調一部分的提案如下所示。

作為起始器，IBM Cloud 會使用 **IKEv2**。作為回應者，IBM Cloud 將同時接受 **IKEv1** 和 **IKEv2**。
{: note}

### IKE 自動協調（階段 1）

可以任何組合來使用下列加密、鑑別及 DH 群組選項：

|    | 加密 | 鑑別 | DH 群組 |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec 自動協調（階段 2）

可以任何組合來使用下列加密及鑑別選項：

|    | 加密 | 鑑別 | DH 群組 |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabled  |
| 2  | aes256 | sha256 | disabled  |
| 3  | 3des   | md5    | disabled  |

## 常見問題 (FAQ)

**當我建立 VPN 閘道時，可以同時建立 VPN 連線嗎？**

如果您使用 API 或 CLI，則必須在建立 VPN 閘道之後建立 VPN 連線。在 IBM Cloud 主控台中，您可以同時建立閘道及連線。

**如果我刪除已連接 VPN 連線的 VPN 閘道，連線會發生什麼情況？**

VPN 連線會連同 VPN 閘道一併刪除。

**如果我刪除 VPN 閘道或 VPN 連線，也會一併刪除 IKE 或 IPSec 原則嗎？**

不，IKE 及 IPSec 原則可套用至多個連線。

**如果我嘗試刪除閘道所在的子網路時，VPN 閘道會發生什麼情況？**

如果有任何實例存在（包括 VPN 閘道在內），就無法刪除子網路。

**是否有預設 IKE 和 IPSec 原則？**

當您建立 VPN 連線而未參照原則 ID（IKE 或 IPSec）時，會使用自動協調。

**VPC 的 VPN 是否支援 HA 配置？**

是。
