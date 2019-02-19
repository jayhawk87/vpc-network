---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-01-24"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Setting up security groups using the APIs

The following example demonstrates how to create and manage security groups, using the {{site.data.keyword.cloud}} Regional APIs (RIAS).

## Prerequisites

To use security groups, first you must have a running IBM Cloud VPC.

Instructions for creating a VPC and a subnet are available in our [Creating a VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) topic.

## Step 1: Create a security group

Create a security group named `my-security-group` in your IBM Cloud VPC.

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Save the ID in a variable so we can use it later, for example, the variable `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Step 2: Add a rule to allow SSH connections

Create a rule on the security group that will allow inbound connections on port 22.

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

## Step 3: Delete the security group (Optional)

To clean up the security group, it cannot be associated with any network interfaces, and it cannot be referenced by a rule in a different security group.

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
