---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-01-30"


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

# (Beta) Using VPN with your VPC

The {{site.data.keyword.cloud}} VPC VPN service allows you to connect private networks in a secure fashion. You can use VPN to set up an IPsec site-to-site tunnel between your VPC and your on-premise private network or another VPC.

For the current {{site.data.keyword.cloud}} VPC release, only policy-based routing is supported.

**This service is now in Beta release version. Resources used during the Beta period will not be available after the close of Beta, and no support will be available. Please direct comments and questions to your IBM Cloud sales representative.**

## Features

* IKEv1 and IKEv2
* Authentication algorithms: `md5`, `sha1`, `sha256`
* Encryption algorithms: `3des`, `aes128`, `aes256`
* Diffie-Hellman (DH) groups: 2, 5, 14
* IKE negotiation mode: main
* IPSec transform protocol: ESP
* IPSec encapsulation mode: tunnel
* Perfect Forward Secrecy (PFS)
* Dead Peer Detection
* Routing: Policy-based
* Authentication Mode: Pre-shared key
* HA Support

## APIs available

The following section gives details about APIs you can use for VPN in your IBM Cloud VPC environment. Please see the [VPC REST APIs](https://{DomainName}/docs/infrastructure/vpc/api-doc-wrapper.html) page for more details.

### VPN gateways and VPN connections

| Description | API |
|----------------------------|-------------|
| Creates a VPN gateway | POST /vpn_gateways |
| Retrieves VPN gateways | GET /vpn_gateways |
| Retrieves a VPN gateway | GET /vpn_gateways/{id} |
| Deletes a VPN gateway | DELETE /vpn_gateways/{id} |
| Updates a VPN gateway | PATCH /vpn_gateways/{id} |
| Creates a new VPN connection | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Retrieves VPN connections | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Retrieves a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Deletes a VPN connection | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Updates a VPN connection | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Retrieves all local CIDRs for a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Deletes a local CIDR from a VPN connection | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Checks if a specific local CIDR exists on a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Sets a local CIDR on a VPN connection | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Retrieves all peer CIDRs for a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Deletes a peer CIDR from a VPN connection | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Checks if a specific peer CIDR exists on a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Sets a peer CIDR on a VPN connection | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE policies

| Description | API |
|-----------------------------|--------------|
| Retrieves all IKE policies | GET /ike_policies |
| Creates an IKE policy | POST /ike_policies |
| Deletes an IKE policy | DELETE /ike_policies/{id} |
| Retrieves an IKE policy | GET /ike_policies/{id} |
| Updates an IKE policy | PATCH /ike_policies/{id} |
| Retrieves all the connections that use the specified IKE policy | GET /ike_policies/{id}/connections |

### IPsec policies

| Description | API |
|---------------------|-------------|
| Retrieves all IPSec policies | GET /ipsec_policies |
| Creates an IPSec policy | POST /ipsec_policies |
| Deletes an IPSec policy | DELETE /ipsec_policies/{id} |
| Retrieves an IPSec policy | GET /ipsec_policies/{id} |
| Updates an IPSec policy | PATCH /ipsec_policies/{id} |
| Retrieves all the connections that use the specified IPsec policy | GET /ipsec_policies/{id}/connections |

## VPN example

In the following example, you'll be able to connect two VPCs together using VPN, which means you can connect
subnets in two separate VPCs as if they were a single network. The IP addresses of the subnets must not overlap.
Here is what the scenario looks like (with some VMs added in each VPC):
![An example VPN scenario](images/vpc-vpn.png)

### Example steps

The example steps that follow skip the prerequisite steps of using IBM Cloud API or CLI to create VPCs. For more information, see [Getting Started](getting-started.html) and [VPC setup with APIs](https://{DomainName}/docs/infrastructure/vpc/example-code.html).

Optionally, you can create a VPN gateway using the UI. Steps can be found in the [Console Tutorial document](https://{DomainName}/docs/infrastructure/vpc/console-tutorial.html#creating-a-vpn).

#### Step 1. Create a VPN gateway in your VPC subnet

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Sample output:
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

Ensure the following fields are saved for subsequent steps.
* `id`. This is the VPN gateway ID and it will be referred to as `$gwid1`.
* `address`. This is the public IP address of the VPN gateway, and it will be referred to as `$gwaddress1`.

NOTE: The gateway status appears as `pending` while the VPN gateway is being created, and the status becomes `available` once creation is complete. Creation may take some time. You can check its status with the following command:

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### Step 2. Create a second VPN gateway on a different VPC

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Sample output:
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

Be sure to save the following fields for subsequent steps.
* `id`. This is the VPN gateway ID, and it will be referred to as `$gwid2`.
* `address`. This is the public IP address of the VPN gateway, and it will be referred to as `$gwaddress2`.


#### Step 3. Create a VPN connection from the first VPN gateway to the second VPN gateway, setting `local_cidrs` to the subnet on VPC one and `peer_cidrs` to the subnet on VPC two

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

Sample output:
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

#### Step 4. Create a VPN connection from the second VPN gateway to the first VPN gateway, setting `local_cidrs` to the subnet on VPC two and `peer_cidrs` to the subnet on VPC one

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

Sample output:
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

#### Step 5. Verify connectivity

Once the VPN connection is established, you will be able to reach your instances on subnet two from subnet one, and vice versa.

You can check the status of the VPN connection as follows:
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

Sample output:
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

## Quotas

See our [VPC Quotas](https://{DomainName}/docs/infrastructure/vpc/vpc-quotas.html#vpn-quotas) topic for VPN quotas.

## Policy auto-negotiation

The use of IKE and IPSec policies to configure a VPN connection is optional. When no policy is selected, default proposals are chosen automatically for a process referred to as _auto-negotiation_. The proposals used as part of auto-negotiation are as follows.

As an initiator, IBM Cloud uses **IKEv2**. As a responder, IBM Cloud accepts both **IKEv1** and **IKEv2**.
{: note}

### IKE auto-negotiation (Phase 1)

The following encryption, authentication and DH Group options may be used in any combination:

|    | Encryption | Authentication | DH Group |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec auto-negotiation (Phase 2)

The following encryption and authentication options may be used in any combination:

|    | Encryption | Authentication | DH Group |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabled  |
| 2  | aes256 | sha256 | disabled  |
| 3  | 3des   | md5    | disabled  |

## FAQ

**When I create a VPN gateway, can I create VPN connections at the same time?**

If you use the API or CLI, VPN connections must be created after the VPN gateway is created. In the IBM Cloud console, you can create the gateway and a connection at the same time.

**If I delete a VPN gateway that has VPN connections attached, what will happen to the connections?**

The VPN connections are deleted along with the VPN gateway.

**Will IKE or IPSec policies be deleted if I delete a VPN gateway or VPN connection?**

No, IKE and IPSec policies can apply to multiple connections.

**What happens to a VPN gateway if I try to delete the subnet that the gateway is located on?**

The subnet cannot be deleted if any instances are present, including the VPN gateway.

**Are there default IKE and IPSec policies?**

When you create a VPN connection without referencing a policy ID (IKE or IPSec), auto-negotiation is used.

**Does VPN for VPC support HA configurations?**

Yes, it does.
