---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-03-01"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creating a secure connection with a remote StrongSwan peer
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

This document is based on Strongswan, version Linux StrongSwan U5.3.5/K4.4.0-133-generic.

The example steps that follow skip the prerequisite steps of using {{site.data.keyword.cloud}} API or CLI to create Virtual Private Clouds. For more information, see [Getting Started](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) and [VPC setup with APIs](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Example steps
The topology for connecting to the remote StrongSwan peer is similar to creating a VPN connection between two VPCs. However, one side of the connection is replaced by the StrongSwan unit.

![enter image description here](./images/vpc-vpn-sw-figure.png)

### To create a secure connection with a remote StrongSwan peer

Go to **/etc** and create your new custom tunnel configuration file with a name similar to **ipsec.abc.conf**. Edit **/etc/ipsec.conf** to include **ipsec.abc.conf** by adding this line:

    include /etc/ipsec.abc.conf

When a VPN peer receives a connection request from a remote VPN peer, it uses IPsec Phase 1 parameters to establish a secure connection and authenticate that VPN peer. Then, if the security policy permits the connection, the StrongSwan unit establishes the tunnel using IPsec Phase 2 parameters and applies the IPsec security policy. Key management, authentication, and security services are negotiated dynamically through the IKE protocol.

**To support these functions, the following general configuration steps must be performed by the StrongSwan unit:**

* Define the Phase 1 parameters that the StrongSwan requires to authenticate the remote peer and establish a secure connection.

* Define the Phase 2 parameters that the StrongSwan requires to create a VPN tunnel with the remote peer.
To connect to the IBM Cloud VPC's VPN capability, we recommend the following configuration:

1. Choose `IKEv2` in authentication;
2. Enable `DH-group 2` in the Phase 1 proposal
3. Set `lifetime = 36000` in the Phase 1 proposal
4. Disable PFS in the Phase 2 proposal
5. Set `lifetime = 10800` in the Phase 2 proposal
6. Input your peer's and subnet's information in the Phase 2 proposal

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: codeblock}

Set the preshared key in `/etc/ipsec.secrets`

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```

After the configuration file is finished executing, restart the StrongSwan unit.

```
 ipsec restart
```
### To create a secure connection with the local IBM Cloud VPC

 +To create a secure connection, you'll create the VPN connection within your VPC, which is similar to the 2 VPC example.

* Create a VPN gateway on your VPC subnet  along with a VPN connection between the VPC and the StrongSwan, setting `local_cidrs` to the subnet value on the VPC, and `peer_cidrs` to the subnet value on the StrongSwan.

NOTE: The gateway status appears as `pending` while the VPN gateway is being created, and the status becomes `available` once creation is complete. Creation may take some time.

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### Check the status for a secure connection

You can check the status of your connection through the IBM Cloud console. Also, you could try to do a `ping` from site to site using the VSIs.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
