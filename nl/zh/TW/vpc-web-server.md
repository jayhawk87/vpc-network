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

# 設定 VPC 中 Web 伺服器之間的連線
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

本文件帶您瀏覽一些範例步驟，利用 Python 來設定 {{site.data.keyword.cloud}} VPC 中的兩個 Web 伺服器之間的通訊。它顯示如何在相同區域及不同區域中建立通訊伺服器。

## 必要條件

在您可以設定 VPC 中 Web 伺服器之間的通訊之前，您必須已建立具有「兩個」子網路的 VPC，且每一個子網路至少有「兩個」實例可用。您需要知道 VPC、子網路和實例的 ID。如果您在設定此配置時需要協助，請遵循[使用 CLI 建立 VPC 資源的手冊](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)中的步驟。

## 情境 1：相同區域但不同子網路的伺服器之間的連線

`vpc` holds vpc ID
`subnet1` holds subnet 1 ID 
`subnet2` holds subnet 2 ID


## 第 1 部分：相同區域與 VPC 的伺服器之間的連線

在已連接公用閘道的相同子網路中建立 2 個以上的實例之後，我們可以利用以下方式來測試其連線：在某個實例中代管一個有 HTML 程式碼的伺服器，然後對該伺服器執行 `curl`，並在其他實例下載該 HTML 檔案，或從您電腦的瀏覽器下載。

假設您在具有公用閘道的相同子網路中以及相同安全群組中具有 `instance1` 和 `instance_2`。您需要使用 API、CLI 或使用者介面，針對安全群組中的實例新增規則，以容許入埠存取。如果您在 `instance_1` 上代管伺服器，則必須新增 `instance_2` 浮動 IP 的規則。例如，假設您有 `instance1` 代管伺服器，其 IPv4 設為 `169.61.161.161` 和 `instance_2`，其 IPv4 設為 `169.61.161.235`，且它嘗試從 `instance_1` 取得該檔案。其看起來如下：

![安全群組新增規則](images/security-group-ui-ex1.png)

* 列出所有安全群組 `ibmcloud is sgs`
* 以下是要取得前一個結果的 API 要求：

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. 在 `instance_1` 上建立基本 `index.html` 檔。以下是基本 HTML 檔的範例：

```
<!DOCTYPE html>
<html>
<body>

<h1>您已順利連接至此實例</h1>
<p>Hello, world!</p>

</body>
</html>
```
{:pre}

2. 在檔案所在的相同目錄中，執行指令 `python -m SimpleHTTPServer <port number>`。如果預設埠 8000 已在使用中，您可以使用其他埠。

3. SSH into `instance_2`

若要尋找浮動 IP 位址，請使用 `ibmcloud is ips` 指令，並使用 `ibmcloud is ip $floating_ip_id` 指令來識別 `floating_ip_id`。您應該能夠識別公用位址。

4. 識別 `instance_1` 的 IP 位址

5. 使用 `curl -X GET http://IP_address:Port_number/index.html` 來要求從管理伺服器 `instance_1` 下載 `index.html`。（例如：`curl -X GET http://169.61.161.161:8000/index.html`）

6. 您應該能夠看到 `index.html` 的輸出

7. （若要透過本端電腦瀏覽器存取伺服器）新增所有入埠資料流量的規則，如下圖所示：

![對於所有入埠的安全群組新增規則](images/security-group-ui-ex2.png)

* 若要新增規則來容許「所有」通訊協定以及從每個來源到安全群組

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**更詳細的要求範例：**

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

8. 開啟 Web 瀏覽器，然後輸入 `http://IP_address:Port_number/index.html`

9. 您應該會看到以 HTML 格式顯示的索引檔。

## 第 2 部分：不同區域與 VPC 的伺服器之間的連線

將其他子網路的實例（沒有設定公用閘道）連接至具有網際網路存取權的實例。您要設定 2 個不同類型的實例：

* 一個 Web 伺服器，其使用公用閘道處理來自網際網路的要求
* 一個後端伺服器，用於執行應用程式並儲存重要資料，但僅傳送內部資料流量 - 已停用公用閘道

「安全群組 (SG)」適用於虛擬伺服器實例 (VSI)，「網路 ACL」適用於子網路。

因此，若要完成此整體作業，您必須建立 VSI、建立子網路、建立 SG 及 ACL 規則，然後將 SG 連接至實例的網路介面，並將網路 ACL 連接至特定子網路。

* 列出 `subnet1` 安全群組中的所有規則

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. 移除第 1 部分的子網路運用的安全群組中的 `inbound-allow-all-traffic` 規則

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. 建立新的 VPC，然後在未連接公用閘道的情況下建立子網路。

3. 在新的子網路中建立不含公用閘道的 `instance_3`（沒有網際網路連線）- 您可能要將較大的 CPU 核心和記憶體配置給該實例，以進行後端管理。

4. 對於第 1 部分的安全群組中的 `instance_3` 新增規則。例如：`instance_3` 有一個浮動 IP `169.61.181.25`

![對於來自其他子網路入埠的安全群組新增規則](images/security-group-ui-ex3.png)

5. 使用 `curl -X GET http://IP_address:Port_number/index.html` 格式的指令，要求從管理伺服器 instance_1 下載 `index.html`。（例如：`curl -X GET http://169.61.161.161:8000/index.html`）

6. 您應該能夠看到 `index.html` 的輸出

這表示您在實例 3 與 1（而不是 2）之間有連線。

## 下一步

您想要建立一些 ACL 規則來控制入埠資料流量以保護伺服器，如[在 VPC 中使用 ACL 的手冊](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)所示。
