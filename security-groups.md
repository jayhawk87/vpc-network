---

copyright:
  years: 2019

lastupdated: "2019-03-01"

keywords: security groups, traffic, firewall, stateful, filtering

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
{:DomainName: data-hd-keyref="DomainName"}

# Using security groups
{: #using-security-groups}

Security groups give you a convenient way to apply rules that establish filtering to each network interface of a virtual server instance (VSI), based on IP address. When you create a new security group resource, you will update it to create the network traffic patterns you desire.

For a comparison of the characteristics of security groups and ACLs, see the [comparison table](/docs/infrastructure/vpc-network?topic=vpc-network-compare-security-groups-and-access-control-lists).

By default, a security group denies all traffic. As rules are added to a security group, it defines the traffic that the security group permits.

Rules are _stateful_, which means that reverse traffic in response to allowed traffic is automatically permitted. So for example, a rule allowing inbound TCP traffic on port 80 also allows replying outbound TCP traffic on port 80 back to the originating host, without the need for an additional rule.

Security groups are scoped to a single VPC. This scoping implies that security group can be attached _only_ to network interfaces of VSIs within the same VPC.

When a VSI is created without any security groups specified, the VSI's primary network interface is put into the _default_ security group of that VSI's VPC. This {{site.data.keyword.cloud}} VPC release has defined a default security group that does allow certain traffic. For more information, see [Updating the default security group](/docs/infrastructure/vpc-network?topic=vpc-network-updating-the-default-security-group).

You can set up security groups by using the REST API, CLI, or UI:

* [Setting up security groups using the API](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-apis)
* [Setting up security groups using the CLI](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Setting up security groups using the UI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
