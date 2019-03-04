---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-03-01"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

subcollection: vpc-network


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}

# VPC Behind the Curtain
{: #vpc-behind-the-curtain}

This page presents a more detailed conceptual picture of what's happening "behind the curtain" with respect to VPC networking. Readers are expected to have some networking background.

## Network isolation

VPC network isolation takes place at three levels: 

* **Hypervisor**: The VSIs (virtual server instances) are isolated by the hypervisor itself. A VSI can not directly reach other VSIs hosted by the same hypervisor if they are not in the same VPC.

* **Network**: Isolation occurs at the network level through the use of **virtual network identifiers** (VNIs). These identifiers are added to all data packets that are exiting the hypervisor, when sent by a VSI.

* **Router**: The _implicit router function_ provides isolation to each VPC by providing a **virtual routing function** (VRF) and a VPN with MPLS (multi-protocol label switching) in the cloud backbone. Each VPC's VRF has a unique identifier, and this isolation allows each VPC to have access to its own copy of the IPv4 address space. The MPLS VPN allows for federating all edges of the cloud: Classic Infrastructure, Direct Link, and VPC.

## Address prefixes

Address prefixes are the summary information used by a VPC's implicit routing function to locate a _destination VSI_, regardless of the availability zone in which the destination VSI is located. The primary function of address prefixes is to optimize routing over the MPLS VPN, while avoiding pathological routing cases. All subnets created in a VPC must be contained in an address prefix, so that all VSIs in a VPC are reachable from all other VSIs in the VPC.

## Data packet flows and the implicit router

Six different types of VSI data packet flows occur in a VPC. In ascending order of complexity, these flows are:

* Intra-subnet, intra-host (same hypervisor)
* Intra-subnet, inter-host
* Inter-subnet, intra-zone
* Inter-subnet, inter-zone
* Extra-vpc service (for IaaS or CSE access)
* Extra-vpc Internet (for Internet access)

**Intra-subnet, intra-host** data flows: These are the simplest. Packets flow between the VSIs on the hypervisor, and no packets are leaving the hypervisor.

**Intra-subnet, inter-host** data flows: These involve packets leaving the hypervisor. As such, each packet is tagged with the proper VNI (virtualized network identifier) to ensure data isolation, and then it's sent to the destination hypervisor that hosts the destination VSI. The destination hypervisor strips off the VNI and forwards the data packet to the destination VSI.

**Inter-subnet, intra-zone** data flows: These involve packets that utilize the VPC's implicit router function, which connects all subnets created in the VPC. It routes the data packet to the correct destination hypervisor. If the destination hypervisor is different from the source hypervisor, the data packet is tagged with the proper VNI and sent to the destination hypervisor. There, the VNI is stripped off and the data packet is forwarded to the destination VSI. (These last steps are the same as described in the previous type of data flow.)

**Inter-subnet, inter-zone** data flows: For these, the implicit router function forwards the packet in the VPC's MPLS VPN for transit across the cloud backbone. At the destination zone, the implicit router function tags the data packet with the VNI. Then the packet is forwarded to the destination hypervisor, where the VNI is (as previously described) stripped off so that the data packet can be forwarded to the destination VSI.

**Extra-vpc service** data flows: Packets destined for IaaS or CSE (IBM Cloud service endpoint) services will utilize the VPC's implicit router function, and also a network address translation (NAT) function. The translation function replaces the VSI address with an IPv4 address, which identifies the VPC to the IaaS or CSE service that's being requested.

**Extra-vpc Internet** data flows: Packets destined for the Internet are the most complex. Besides utilizing the VPC's implicit router function, each of these flows also relies on one of the implicit router's two network address translation (NAT) functions:

  * an explicit one-to-many NAT through a public gateway function that serves all subnets connected to it.
  * one-to-one NAT assigned to individual VSIs.

After NAT translation, the implicit router forwards these Internet-destined packets to the Internet, using the cloud backbone.

## Classic access
{: #classic-access}

The [**Classic access**](/docs/infrastructure/vpc/classic-access.html) feature for VPC is accomplished by re-using the VRF identifier from the {{site.data.keyword.cloud}} Classic Infrastructure Account as the VRF identifier for VPC. This implementation allows the VPC's implicit router function to join the same MPLS VPN that is used by the Classic Infrastructure Account. Thus, the VPC has access to classic resources and to anything else that's reachable by means of existing Direct Link connections.
