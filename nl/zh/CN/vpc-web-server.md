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

# 设置 VPC 中 Web 服务器之间的连接
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

本文档将引导您逐步完成一些示例步骤，以使用 Python 设置 {{site.data.keyword.cloud}} VPC 中两个 Web 服务器之间的通信。其中将说明如何在同一专区中以及在不同专区中创建通信服务器。

## 先决条件

必须已创建具有 2 个子网的 VPC，并且每个子网中至少有 2 个实例可用后，才能设置该 VPC 中 Web 服务器之间的通信。您需要知道 VPC、子网和实例的标识。如果需要有关设置此配置的帮助，请遵循[通过 CLI 创建 VPC 资源的指南](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli)中的步骤。

## 场景 1：同一专区内不同子网中服务器之间的连接

`vpc` holds vpc ID
`subnet1` holds subnet 1 ID 
`subnet2` holds subnet 2 ID


## 第 1 部分：同一专区和 VPC 中服务器之间的连接

在连接了公共网关的同一子网中创建 2 个或更多实例后，可以通过使用 HTML 代码在一个实例中托管服务器来测试其连接，接着对该服务器执行 `curl` 命令，然后在其他实例上或通过计算机的浏览器下载该 HTML 文件。

假设您在具有公共网关和相同安全组的同一子网中有 `instance_1` 和 `instance_2`。您需要使用 API、CLI 或 UI 在安全组中添加用于实例的规则以允许入站访问。如果在 `instance_1` 上托管服务器，那么必须添加用于 `instance_2` 浮动 IP 的规则。例如，假设您有 `instance_1`（用于托管服务器，IPv4 设置为 `169.61.161.161`）和 `instance_2`（IPv4 设置为 `169.61.161.235`），后者尝试从 `instance_1` 中通过 GET 操作获取该文件。规则应该类似于以下内容：

![安全组新规则](images/security-group-ui-ex1.png)

* 列出所有安全组：`ibmcloud is sgs`
* 下面是用于获取先前结果的 API 请求：

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. 在 `instance_1` 上创建基本 `index.html` 文件。下面是基本 HTML 文件的示例：

```
<!DOCTYPE html>
<html>
<body>

<h1>You have successfully connected to the instance</h1>
<p>Hello, world!</p>

</body>
</html>
```
{:pre}

2. 在该文件所在的目录中，运行 `python -m SimpleHTTPServer <port number>` 命令。如果缺省端口 8000 已在使用，那么可以使用其他端口。

3. 通过 SSH 登录到 `instance_2`

要查找浮动 IP 地址，请使用 `ibmcloud is ips` 命令，然后使用 `ibmcloud is ip $floating_ip_id` 命令来识别 `floating_ip_id`。您应该能够识别公共地址。

4. 识别 `instance_1` 的 IP 地址

5. 使用 `curl -X GET http://IP_address:Port_number/index.html` 来请求从托管服务器 `instance_1` 下载 `index.html`。（例如：`curl -X GET http://169.61.161.161:8000/index.html`）

6. 您应该能够看到 `index.html` 的输出

7. （通过本地计算机浏览器访问服务器）添加用于所有入站流量的规则，如下图所示：

![用于所有入站流量的安全组新规则](images/security-group-ui-ex2.png)

* 添加规则以允许所有协议以及从每个源到安全组的流量

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**更详细的请求示例：**

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

8. 打开 Web 浏览器，然后输入 `http://IP_address:Port_number/index.html`

9. 您应该会看到显示为 HTML 格式的索引文件。

## 第 2 部分：不同专区和 VPC 中服务器之间的连接

将其他子网的实例（未设置公共网关）连接到具有因特网访问权的实例。您希望设置 2 种不同类型的实例：

* Web 服务器，用于使用公共网关处理来自因特网的请求
* 后端服务器，用于运行应用程序并存储重要数据，并且仅发送内部流量 - 禁用了公共网关

安全组 (SG) 应用于虚拟服务器实例 (VSI)，网络 ACL 应用于子网。

因此，要完成此总体任务，必须创建 VSI，创建子网，创建 SG 和 ACL 规则，然后将 SG 连接到实例的网络接口，并将网络 ACL 连接到特定子网。

* 列出 `subnet1` 安全组中的所有规则

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. 除去第 1 部分子网练习的安全组中的 `inbound-allow-all-traffic` 规则

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. 创建新的 VPC，然后创建不连接公共网关的子网。

3. 在没有公共网关（无因特网连接）的新子网中创建 `instance_3` - 出于后端原因，您可能希望为该实例分配更大的 CPU 核心数和内存。

4. 在第 1 部分的安全组中添加用于 `instance_3` 的规则。例如：`instance_3` 具有浮动 IP `169.61.181.25`

![用于具有来自其他子网出站流量的实例的安全组新规则](images/security-group-ui-ex3.png)

5. 使用格式为 `curl -X GET http://IP_address:Port_number/index.html` 的命令来请求从托管服务器 instance_1 下载 `index.html`。（例如：`curl -X GET http://169.61.161.161:8000/index.html`）

6. 您应该能够看到 `index.html` 的输出

这说明已在实例 3 和实例 1 之间建立了连接，但未与实例 2 建立连接。

## 后续步骤

您将希望通过创建用于控制入站流量的某些 ACL 规则来保护服务器，如[在 VPC 中使用 ACL 的指南](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)中所示。
