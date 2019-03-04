---
copyright:
  years: 2019
lastupdated: "2019-03-03"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-network
---

# Compare Security Groups and Access Control Lists
{: #compare-security-groups-and-access-control-lists}

Security groups and access control lists provide ways to control the traffic across the subnets and instances in your {{site.data.keyword.cloud}} VPC, using rules that you specify.

The following table summarizes some key differences between security groups and ACLs:

|  | Security Groups | ACLs    |
|-------------|-----------------|---------|
| Control level  | VSI instance    | Subnet  |
| State   | Stateful - Return traffic is allowed by default | Stateless - Return traffic is denied by default and needs to be explicitly allowed |
| Supported rules | Uses allow rules only | Uses allow and deny rules|
| How rules are applied | All rules are considered | Rules are processed in sequence |
| Relationship to the associated resource | An instance can be associated with multiple security groups| Multiple subnets can be associated with the same ACL|
