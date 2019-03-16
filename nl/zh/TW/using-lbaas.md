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

# （測試版）在 IBM Cloud VPC 中使用負載平衡器
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

負載平衡器服務會配送您 {{site.data.keyword.cloud}} VPC 的相同地區內的多個伺服器實例之間的資料流量。

**此服務現在屬於「測試版」。測試版期間使用的資源在「測試版」結束之後將無法使用，且不提供任何支援。如有任何意見和問題，請直接洽詢 IBM Cloud 業務代表。**

## 公用負載平衡器
{: #public-load-balancer}

您的負載平衡器服務實例被指派一個可公開存取的完整網域名稱 (FQDN)，您必須使用此名稱來存取在 {{site.data.keyword.cloud}} VPC 中的負載平衡器服務背後代管的應用程式。這個網域名稱可能已登錄一個以上的公用 IP 位址。

經過一段時間之後，這些公用 IP 位址及公用 IP 位址數目可能會根據對客戶透明化的維護和調整活動而變更。代管應用程式的後端伺服器實例 (VSI) 必須在相同地區及相同的 VPC 下執行。

## 前端接聽器及後端儲存區
您最多可以定義 10 個前端接聽器（應用程式埠），並將它們對映至後端應用程式伺服器上的個別後端儲存區。將指派給負載平衡器服務實例的完整網域名稱 (FQDN) 以及前端系統接聽器埠公開至公用網際網路。會在這些埠上接收送入的使用者要求。

支援的前端接聽器通訊協定如下：
* HTTP
* HTTPS
* TCP

支援的後端儲存區通訊協定如下：
* HTTP
* TCP

送入的 HTTPS 資料流量會在負載平衡器終止，以容許與後端伺服器進行純文字 HTTP 通訊。

您最多可以將 50 個伺服器實例連接至後端儲存區。每個連接的伺服器實例都必須配置一個埠。這個埠不一定與前端接聽器埠相同。

保留 56500 至 56520 的埠範圍作為管理用途；這些埠無法作為前端接聽器埠。
{: note}

## 負載平衡方法
{: #load-balancing-methods}

下列三種負載平衡方法可用來配送後端應用程式伺服器之間的資料流量：

* **循環式：**循環式是預設的負載平衡方法。使用此方法，負載平衡器會以循環方式將送入的用戶端連線轉遞至後端伺服器。因此，所有後端伺服器都會接收到約略相等數目的用戶端連線。

* **加權循環式：**使用此方法時，負載平衡器將送入的用戶端連線轉遞至後端伺服器時會與指派給這些伺服器的加權成正比。每部伺服器都會獲指派預設加權 50，您可以將此值自訂為 0 到 100 之間的任何值。

舉例來說，如果三個應用程式伺服器 A、B 和 C 分別自訂為 60、60 和 30 的加權，則伺服器 A 和 B 會收到相等數目的連線，而伺服器 C 會收到此連線數目的一半。

* **最少連線數：**如果使用此方法，則在給定的時間內提供最少連線數目的伺服器實例會接收下一個用戶端連線。

**這些方法的其他特徵：**

* 將伺服器加權重設為 '0'，表示沒有新的連線會轉遞至該伺服器，但只要它保持作用中，任何現有資料流量都會繼續傳送。使用加權 '0' 可協助溫和關閉伺服器，並將它從服務循環中移除。
* 伺服器加權值僅適用於加權循環式方法。循環式和最少連線數的負載平衡方法會忽略它們。

## 性能檢查
{: #health-checks}

對於後端儲存區而言，性能檢查定義是必要的。

負載平衡器會執行定期性能檢查，以監視後端埠的性能，並據此將用戶端資料流量轉遞給它們。如果發現給定的後端伺服器埠性能不佳，就不會轉遞任何新連線。負載平衡器會持續監視不健全埠的性能，如果它們的性能恢復，就會繼續使用，這表示它們會順利通過兩次連續性能檢查。

HTTP 和 TCP 埠的性能檢查執行如下：

* **HTTP：**會將針對預先指定之 URL 的 `HTTP GET` 要求傳送至後端伺服器埠。收到 `200 OK` 回應時，會將伺服器埠標示為性能良好。預設 `GET` 性能路徑是憑藉使用者介面的 "/"，可以自訂。

* **TCP：**「負載平衡器」嘗試在指定的 TCP 埠開啟與後端伺服器的 TCP 連線。如果連線嘗試成功，且連線已關閉，伺服器埠會被標示為健全。

預設性能檢查間隔為 5 秒，性能檢查要求的預設逾時為 2 秒，而預設的重試次數為 2。
{: note}

## SSL 卸載及必要的授權

對於所有送入的 HTTPS 連線，負載平衡器服務會終止 SSL 連線，並建立與後端伺服器實例的純文字 HTTP 通訊。使用此技術，會將 CPU 密集 SSL 信號交換及加密或解密作業移出後端伺服器實例，從而容許它們使用所有 CPU 週期來處理應用程式資料流量。

負載平衡器需要有 SSL 憑證才能執行 SSL 卸載作業。您可以透過 [IBM Certificate Manager ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} 來管理 SSL 憑證。

如果您需要憑證管理程式實例中 SSL 憑證的負載平衡器存取權，請建立一個授權，讓負載平衡器服務實例存取包含該 SSL 憑證的憑證管理程式實例。您可以透過[身分和存取授權 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](/docs/services/iam/#/authorizations){: new_window} 來管理這類授權。請務必選擇**基礎架構服務**作為來源服務，選擇 **VPC 的負載平衡器**作為資源類型，選擇**憑證管理程式**作為目標服務，並指派**寫出器**服務存取角色。若要建立負載平衡器，您必須針對來源資源實例授與**所有資源實例**授權。目標服務實例可以是**所有實例**，也可以是您的特定憑證管理程式資源實例。

如果移除了必要授權，您的負載平衡器可能會發生錯誤。
{: note}

## Identity and Access Management (IAM)

您可以配置 **VPC 的負載平衡器**實例的存取原則。若要管理您的使用者存取原則，請造訪[身分和存取授權 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](/docs/services/iam/#/authorizations){: new_window}。

### 配置使用者的資源群組存取原則

若要建立負載平衡器，您需要存取資源群組。建立負載平衡器的使用者必須對所提供的資源群組或對預設資源群組（如果沒有提供的話）具有適當的存取權。

1. 導覽至**管理 > 帳戶 > 使用者**。您會看到一份有權存取 IBM Cloud 帳戶的使用者清單。
2. 選取您要對其指派存取原則的使用者名稱。如果未顯示使用者，請按一下**邀請使用者**，將使用者新增至您的 IBM Cloud 帳戶。
3. 選取**指派存取權**。
4. 選取**在資源群組內指派存取權**。
5. 從**資源群組**下拉清單中，選取想要的資源群組。
6. 從**指派對資源群組的存取權**下拉清單中，選取想要的存取權。
7. 從**服務**下拉清單中，選取想要的服務。
9. 選取**指派**，將資源群組存取原則指派給使用者。

### 配置使用者的資源存取原則

| 平台存取角色 | 負載平衡器動作 |
|-------------|-----|
| 管理者 | 建立/檢視/編輯/刪除負載平衡器 |
| 編輯者 | 建立/檢視/編輯/刪除負載平衡器 |
| 檢視者 | 檢視負載平衡器 |

1. 導覽至**管理 > 帳戶 > 使用者**。您會看到有權存取 IBM Cloud 帳戶的使用者清單。
2. 選取您要對其指派存取原則的使用者名稱。如果未顯示使用者，請按一下**邀請使用者**，將使用者新增至您的 IBM Cloud 帳戶。
3. 選取**指派存取權**。
4. 選取**指派對資源的存取權**。
5. 從**服務**下拉清單中，選取**基礎架構服務**。
6. 從**資源類型**下拉清單中，選取 **VPC 的負載平衡器**。
7. 從**負載平衡器 ID** 下拉清單中，選取「負載平衡器」實例 ID，或使用預設值**所有負載平衡器**。
8. 指派平台存取角色給使用者。
9. 選取**指派**，將存取原則指派給使用者。

## 可用的 API

若要進行 API 呼叫，您必須使用某種 REST 用戶端格式。例如，您可以使用 `curl` 指令來擷取所有現有負載平衡器：

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

下一節提供有關您可在 VPC 環境中用於負載平衡器之 API 的詳細資料。如需完整規格，請參閱 [RIAS API 參考資料](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers)。

| 說明 | API |
|-------------|-----|
| 建立及佈建負載平衡器 | POST /load_balancers |
| 擷取所有負載平衡器 | GET /load_balancers |
| 擷取負載平衡器 | GET /load_balancers/{id} |
| 刪除負載平衡器 | DELETE /load_balancers/{id} |
| 更新負載平衡器 | PATCH /load_balancers/{id} |
| 建立接聽器 | POST /load_balancers/{id}/listeners |
| 擷取負載平衡器的所有接聽器 | GET /load_balancers/{id}/listeners |
| 擷取接聽器 | GET /load_balancers/{id}/listeners/{listener_id} |
| 刪除接聽器 | DELETE /load_balancers/{id}/listeners/{listener_id} |
| 更新接聽器 | PATCH /load_balancers/{id}/listeners/{listener_id} |
| 建立儲存區 | POST /load_balancers/{id}/pools |
| 擷取負載平衡器的所有儲存區 | GET /load_balancers/{id}/pools |
| 擷取儲存區 | GET /load_balancers/{id}/pools/{pool_id} |
| 刪除儲存區 | DELETE /load_balancers/{id}/pools/{pool_id} |
| 更新儲存區 | PATCH /load_balancers/{id}/pools/{pool_id} |
| 建立成員 | POST /load_balancers/{id}/pools/{pool_id}/members |
| 擷取儲存區的所有成員 | GET /load_balancers/{id}/pools/{pool_id}/members |
| 擷取成員 |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 刪除儲存區的成員 | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 更新成員 | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 更新儲存區的成員 | PUT /load_balancers/{id}/pools/{pool_id}/members |

## 負載平衡器範例

在下列範例中，您將使用 API 在 2 個 VPC 伺服器實例（`192.168.100.5` 和 `192.168.100.6`）前面建立負載平衡器，其在埠 `80` 執行 Web應用程式接聽。負載平衡器具有前端接聽器，它容許透過 HTTPS 安全地存取 Web 應用程式。然後，您可以使用 API 在建立負載平衡器實例之後取得該實例的詳細資料，以及刪除負載平衡器實例。

### 範例步驟

接下來的範例步驟會跳過使用 [IBM Cloud 使用者介面](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console)、[CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) 或 [RIAS API](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) 來佈建 VPC、子網路及實例的必要步驟。 


負載平衡器範例步驟也可以使用 [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference) 來執行。
{: note}

#### 步驟 1. 建立具有接聽器、儲存區及連接伺服器實例（成員）的負載平衡器

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

輸出範例：
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

儲存下一步要使用的負載平衡器 ID，例如，儲存為變數 `lbid`。

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### 步驟 2. 取得負載平衡器

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

給負載平衡器佈建一點時間。完成時，它將設為 `online` 和 `active` 狀態，如同您在下列輸出範例中所見：

輸出範例：

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

#### 步驟 3. 刪除負載平衡器

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## 常見問題 (FAQ)

這一節包含有關 **VPC 的負載平衡器**服務的一些常見問題的回答。

### 我的負載平衡器可以使用不同的 DNS 名稱嗎？

負載平衡器的自動指派 DNS 名稱不可自訂。不過，您可以新增 CNAME（標準名稱）記錄，將您偏好的 DNS 名稱指向自動指派的負載平衡器 DNS 名稱。例如，您的負載平衡器 ID 是 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`，自動指派的負載平衡器 DNS 名稱是 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`。您偏好的 DNS 名稱是 `www.myapp.com`。您可以新增 CNAME 記錄（透過您用來管理 `myapp.com` 的 DNS 提供者），將 `www.myapp.com` 指向負載平衡器 DNS 名稱 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`。

### 我可以使用負載平衡器定義的前端接聽器數目上限是多少？

10。

### 我可以連接至後端儲存區的伺服器實例數目上限是多少？

50。

### 我可以建立僅限內部用戶端存取的內部專用負載平衡器嗎？  

不，現在不行。

### 負載平衡器可以橫向擴充嗎？  

不，現在不行。

### 如果我在用來部署負載平衡器的子網路上使用 ACL 或安全群組，該怎麼辦？

您需要確定已設定適當的 ACL 或安全群組規則，來容許針對所配置的接聽器埠及管理埠的送入資料流量（56500 - 56520 範圍內的埠）。也應該容許負載平衡器與後端實例之間的資料流量。

### 為什麼我會收到錯誤訊息：`找不到憑證實例`？

* 憑證實例 CRN 可能無效。
* 您可能未授與「服務對服務授權」。請參閱本文件的 **SSL 卸載**一節。

### 為什麼我會收到 `401 未獲授權的錯誤`碼？

檢查使用者的下列存取原則：
* 負載平衡器資源類型存取原則
* 資源群組存取原則

### 為什麼我的負載平衡器處於 `maintenance_pending` 狀態？

在各種維護活動期間，負載平衡器會處於 `maintenance_pending` 狀態，例如：
* 有任何漸進式升級正在處理漏洞及套用安全修補程式
* 回復活動

### 為何需要在佈建期間選擇多個子網路？

負載平衡器應用裝置會部署到您所選取的子網路。強烈建議您選擇不同區域中的子網路，以獲得更高的可用性及備援。
