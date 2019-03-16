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

# (Beta) 在 IBM Cloud VPC 中使用负载均衡器
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

Load Balancer 服务在同一区域内的多个服务器实例之间分配 {{site.data.keyword.cloud}} VPC 的流量。

**此服务现在处于 Beta 发行版。Beta 期间使用的资源在 Beta 版结束后将不可用，也不会提供任何支持。如有意见或问题，请直接联系 IBM Cloud 销售代表。**

## 公共负载均衡器
{: #public-load-balancer}

Load Balancer 服务实例分配有可公开访问的标准域名 (FQDN)，必须使用该域名来访问 {{site.data.keyword.cloud}} VPC 中 Load Balancer 服务后面托管的应用程序。此域名可能注册有一个或多个公共 IP 地址。

随着时间的推移，这些公共 IP 地址和公共 IP 地址的数量可能会因维护和伸缩活动而更改，客户可透明了解这些信息。托管应用程序的后端服务器实例 (VSI) 必须在同一 VPC 下的同一区域中运行。

## 前端侦听器和后端池
您最多可定义 10 个前端侦听器（应用程序端口），然后将其映射到后端应用程序服务器上的相应后端池。分配给 Load Balancer 服务实例的标准域名 (FQDN) 和前端侦听器端口会公开给公用因特网。在这些端口上会接收入局用户请求。

支持的前端侦听器协议为：
* HTTP
* HTTPS
* TCP

支持的后端池协议为：
* HTTP
* TCP

入局 HTTPS 流量在负载均衡器处终止，以允许与后端服务器进行明文 HTTP 通信。

最多可将 50 个服务器实例连接到后端池。每个连接的服务器实例必须配置有一个端口。该端口可与前端侦听器端口相同，也可以不同。

端口范围 56500 到 56520 保留用于管理目的；这些端口不能用作前端侦听器端口。
{: note}

## 负载均衡方法
{: #load-balancing-methods}

以下三种负载均衡方法可用于在后端应用程序服务器之间分配流量：

* **循环法：**循环法是缺省负载均衡方法。使用此方法，负载均衡器以循环方式将入局客户机连接转发到后端服务器。因此，所有后端服务器会收到大致相等数量的客户机连接。

* **加权循环法：**使用此方法，负载均衡器会根据分配给后端服务器的权重，成比例地将入局客户机连接转发到这些服务器。每个服务器分配的缺省权重为 50，可以定制为 0 到 100 之间的任何值。

例如，如果三个应用程序服务器 A、B 和 C 的定制权重分别为 60、60 和 30，那么服务器 A 和 B 将接收相等数量的连接，而服务器 C 收到的连接数是 A 和 B 的一半。

* **最少连接数：**使用此方法，在给定时间提供最少连接数的服务器实例会接收下一个客户机连接。

**这些方法的其他特征：**

* 将服务器权重重置为“0”表示新连接不会转发到该服务器，但只要该服务器处于活动状态，任何现有流量都将继续流动。使用权重“0”可帮助正常关闭服务器，并将其从服务循环中除去。
* 服务器权重值仅适用于加权循环法。循环法和最少连接数负载均衡方法将忽略服务器权重值。

## 运行状况检查
{: #health-checks}

对于后端池，运行状况检查定义是必需的。

负载均衡器会定期执行运行状况检查，以监视后端端口的运行状况，并相应地将客户机流量转发给这些端口。如果发现给定的后端服务器端口运行不正常，那么不会向其转发任何新连接。负载均衡器会继续监视未正常运行的端口的运行状况，如果这些端口重新变得运行正常（这意味着这些端口成功通过了两次连续运行状况检查尝试），那么会恢复使用这些端口。

HTTP 和 TCP 端口的运行状况检查按如下所示来执行：

* **HTTP：**针对预先指定的 URL 的 `HTTP GET` 请求会发送到后端服务器端口。在收到 `200 正常`响应时，会将服务器端口标记为正常运行。使用 UI 时，缺省 `GET` 运行状况路径为“/”，此路径可以定制。

* **TCP：**负载均衡器尝试在指定 TCP 端口上打开与后端服务器的 TCP 连接。如果连接尝试成功，那么服务器端口会标记为正常运行，随后该连接会关闭。

缺省运行状况检查时间间隔为 5 秒，针对运行状况检查请求的缺省超时为 2 秒，缺省重试次数为 2。
{: note}

## SSL 卸载和必需的授权

对于所有入局 HTTPS 连接，Load Balancer 服务会终止 SSL 连接，并与后端服务器实例建立明文 HTTP 通信。通过这种方法，CPU 密集型 SSL 握手和加密或解密任务可从后端服务器实例转移到其他位置，从而允许这些实例使用其所有 CPU 周期来处理应用程序流量。

负载均衡器需要 SSL 证书才能执行 SSL 卸载任务。可以通过 [IBM Certificate Manager ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} 来管理 SSL 证书。

如果需要负载均衡器拥有对 Certificate Manager 实例中的 SSL 证书的访问权，请创建授权，以授予 Load Balancer 服务实例对包含 SSL 证书的 Certificate Manager 实例的访问权。可以通过[身份和访问权授权 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](/docs/services/iam/#/authorizations){: new_window} 来管理此类授权。请确保选择**基础架构服务**作为源服务，选择 **Load Balancer for VPC** 作为资源类型，选择 **Certificate Manager** 作为目标服务，并分配**写入者**服务访问角色。要创建负载均衡器，必须授予对源资源实例的**所有资源实例**权限。目标服务实例可以是**所有实例**，也可以是特定 Certificate Manager 资源实例。

如果除去了必需的授权，那么负载均衡器可能会发生错误。
{: note}

## Identity and Access Management (IAM)

您可以为 **Load Balancer for VPC** 实例配置访问策略。要管理用户访问策略，请访问[身份和访问权授权 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](/docs/services/iam/#/authorizations){: new_window}。

### 为用户配置资源组访问策略

要创建负载均衡器，您需要访问资源组。创建负载均衡器的用户必须具有对所提供资源组的正确访问权，或者如果未提供任何资源组，那么必须具有对缺省资源组的正确访问权。

1. 浏览至**管理 > 帐户 > 用户**。您将看到有权访问 IBM Cloud 帐户的用户的列表。
2. 选择要为其分配访问策略的用户的名称。如果未显示所需用户，请单击**邀请用户**以将该用户添加到 IBM Cloud 帐户。
3. 选择**分配访问权**。
4. 选择**在资源组中分配访问权**。
5. 从**资源组**下拉列表中，选择所需的资源组。
6. 从**分配对资源组的访问权**下拉列表中，选择所需的访问权。
7. 从**服务**下拉列表中，选择所需的服务。
9. 选择**分配**以将资源组访问策略分配给用户。

### 为用户配置资源访问策略

|平台访问角色|负载均衡器操作|
|-------------|-----|
|管理员|创建/查看/编辑/删除负载均衡器|
|编辑者|创建/查看/编辑/删除负载均衡器|
|查看者|查看负载均衡器|

1. 浏览至**管理 > 帐户 > 用户**。您将看到有权访问 IBM Cloud 帐户的用户的列表。
2. 选择要为其分配访问策略的用户的名称。如果未显示所需用户，请单击**邀请用户**以将该用户添加到 IBM Cloud 帐户。
3. 选择**分配访问权**。
4. 选择**分配对资源的访问权**。
5. 从**服务**下拉列表中，选择**基础架构服务**。
6. 从**资源类型**下拉列表中，选择 **Load Balancer for VPC**。
7. 从**负载均衡器标识**下拉列表中，选择负载均衡器实例标识，或使用缺省值**所有负载均衡器**。
8. 为用户分配平台访问角色。
9. 选择**分配**以将访问策略分配给用户。

## 可用 API

要进行 API 调用，必须使用某种形式的 REST 客户机。例如，可以使用 `curl` 命令来检索所有现有负载均衡器：

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

以下部分提供了有关可用于 VPC 环境中负载均衡器的 API 的详细信息。有关完整规范，请参阅 [RIAS API 参考](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers)。

|描述|API|
|-------------|-----|
|创建和供应负载均衡器|POST /load_balancers|
|检索所有负载均衡器|GET /load_balancers|
|检索负载均衡器|GET /load_balancers/{id}|
|删除负载均衡器|DELETE /load_balancers/{id}|
|更新负载均衡器|PATCH /load_balancers/{id}|
|创建侦听器|POST /load_balancers/{id}/listeners|
|检索负载均衡器的所有侦听器|GET /load_balancers/{id}/listeners|
|检索侦听器|GET /load_balancers/{id}/listeners/{listener_id}|
|删除侦听器|DELETE /load_balancers/{id}/listeners/{listener_id}|
|更新侦听器|PATCH /load_balancers/{id}/listeners/{listener_id}|
|创建池|POST /load_balancers/{id}/pools|
|检索负载均衡器的所有池|GET /load_balancers/{id}/pools|
|检索池|GET /load_balancers/{id}/pools/{pool_id}|
|删除池|DELETE /load_balancers/{id}/pools/{pool_id}|
|更新池|PATCH /load_balancers/{id}/pools/{pool_id}|
|创建成员|POST /load_balancers/{id}/pools/{pool_id}/members|
|检索池的所有成员|GET /load_balancers/{id}/pools/{pool_id}/members|
|检索成员|GET /load_balancers/{id}/pools/{pool_id}/members/{member_id}|
|从池中删除成员|DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id}|
|更新成员|PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id}|
|更新池的成员|PUT /load_balancers/{id}/pools/{pool_id}/members|

## 负载均衡器示例

在以下示例中，您将使用 API 在运行 Web 应用程序（用于侦听端口 `80`）的 2 个 VPC 服务器实例（`192.168.100.5` 和 `192.168.100.6`）前面创建负载均衡器。负载均衡器具有前端侦听器，用于允许通过 HTTPS 安全地访问 Web 应用程序。然后，可以使用 API 在创建负载均衡器实例后获取其详细信息，也可以删除负载均衡器实例。

### 示例步骤

以下示例步骤跳过了使用 [IBM Cloud UI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console)、[CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) 或 [RIAS API](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) 来供应 VPC、子网和实例的先决条件步骤。 


负载均衡器示例步骤还可以使用 [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference) 来运行。
{: note}

#### 步骤 1. 创建具有侦听器、池和连接的服务器实例（成员）的负载均衡器

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

样本输出：
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

保存负载均衡器的标识以在后续步骤中使用，例如在变量 `lbid` 中使用。

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### 步骤 2. 获取负载均衡器

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

请留出一些时间等待负载均衡器供应。准备就绪后，负载均衡器将设置为 `online` 和 `active` 状态，如以下样本输出中所示：

样本输出：

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

#### 步骤 3. 删除负载均衡器

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## 常见问题

此部分包含有关 **Load Balancer for VPC** 服务的一些常见问题的解答。

### 可以对负载均衡器使用其他 DNS 名称吗？

负载均衡器的 DNS 名称是自动分配的，不可定制。但是，可以添加 CNAME（规范名称）记录，用于将您的首选 DNS 名称指向自动分配的负载均衡器 DNS 名称。例如，负载均衡器标识为 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`，自动分配的负载均衡器 DNS 名称为 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`。您的首选 DNS 名称为 `www.myapp.com`。您可以添加 CNAME 记录（通过用于管理 `myapp.com` 的 DNS 提供程序），将 `www.myapp.com` 指向负载均衡器 DNS 名称 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`。

### 使用负载均衡器最多可以定义多少个前端侦听器？

10 个。

### 最多可以将多少个服务器实例连接到后端池？

50 个。

### 可以创建仅可供内部客户机访问的仅内部专用负载均衡器吗？  

不能，目前不行。

### 负载均衡器可水平伸缩吗？  

不能，目前不行。

### 如果要在用于部署负载均衡器的子网上使用 ACL 或安全组，应该怎么做？

您需要确保落实了正确的 ACL 或安全组规则，以允许配置的侦听器端口和管理端口（从 56500 到 56520 的端口）的入局流量。还应允许负载均衡器与后端实例之间的流量。

### 为什么会收到错误消息：`找不到证书实例`？

* 证书实例 CRN 可能无效。
* 可能尚未向您授予服务到服务权限。请参阅本文档的 **SSL 卸载**部分。

### 为什么会收到 `401 未授权错误`代码？

检查用户的以下访问策略：
* 负载均衡器资源类型访问策略
* 资源组访问策略

### 为什么负载均衡器会处于 `maintenance_pending` 状态？

负载均衡器在各种维护活动期间会处于 `maintenance_pending` 状态，例如以下活动：
* 任何用于解决漏洞并应用安全补丁的卷动升级
* 恢复活动

### 为什么在供应期间需要选择多个子网？

负载均衡器设备会部署到所选的子网。强烈建议选择不同专区中的子网，以提高可用性和冗余性。
