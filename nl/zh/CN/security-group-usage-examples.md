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

# 使用 API 设置安全组
{: #setting-up-security-groups-using-the-apis}

以下示例说明了如何使用 {{site.data.keyword.cloud}} 区域 API (RIAS) 来创建和管理安全组。

## 先决条件

要使用安全组，首先必须有一个正在运行的 IBM Cloud VPC。

创建 VPC 和子网的指示信息在[创建 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) 主题中提供。

## 步骤 1：创建安全组

在 IBM Cloud VPC 中创建名为 `my-security-group` 的安全组。

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

将标识保存在变量中，以便稍后可以使用，例如保存在变量 `sg` 中：

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## 步骤 2：添加规则以允许 SSH 连接

在安全组中创建规则以允许端口 22 上的入站连接。

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

## 步骤 3：删除安全组（可选）

要清除安全组，该安全组不能与任何网络接口相关联，也不能由其他安全组中的规则引用。

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
