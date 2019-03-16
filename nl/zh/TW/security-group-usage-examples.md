---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# 使用 API 設定安全群組
{: #setting-up-security-groups-using-the-apis}

下列範例示範如何使用 {{site.data.keyword.cloud}} 區域 API (RIAS) 來建立及管理安全群組。

## 必要條件

若要使用安全群組，首先您必須具有執行中的 IBM Cloud VPC。

如需建立 VPC 及子網路的指示，請參考[建立 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) 主題。

## 步驟 1：建立安全群組

在 IBM Cloud VPC 中，建立名為 `my-security-group` 的安全群組。

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

將 ID 儲存在變數中，以供稍後使用，例如，儲存在 `sg` 變數中：

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## 步驟 2：新增規則以容許 SSH 連線

在安全群組上建立容許在埠 22 建立入埠連線的規則。

```
curl -X POST $rias_endpoint/v1/security_groups/$sg/rules?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## 步驟 3：刪除安全群組（選用）

若要清除安全群組，它不能與任何網路介面相關聯，且不能被不同安全群組中的規則參照。

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
