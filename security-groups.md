---

copyright:
  years: 2018, 2019

lastupdated: "2019-01-15"

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

# Updating the default security group using the CLI

Security groups give you a convenient way to apply rules that establish filtering to each network interface of a virtual server instance (VSI), based on IP address. When you create a new security group resource, you will update it to create the network traffic patterns you desire.

By default, a security group denies all traffic. As rules are added to a security group, it defines the traffic that the security group permits.

Rules are _stateful_, which means that reverse traffic in response to allowed traffic is automatically permitted. So for example, a rule allowing inbound TCP traffic on port 80 also allows replying outbound TCP traffic on port 80 back to the originating host, without the need for an additional rule.

Security groups are scoped to a single VPC. This scoping implies that security group can be attached _only_ to network interfaces of VSIs within the same VPC.

When a VSI is created without any security groups specified, the VSI's primary network interface is put into the _default_ security group of that VSI's VPC. This {{site.data.keyword.cloud}} VPC release has defined a default security group that does allow certain traffic.

## The default security group

The default security group is very similar to any other security group, with the exception that it cannot be deleted.

Each VPC has a default security group, with rules to allow:

* Inbound traffic from all members of the group.
* All outbound traffic.

If you edit the rules of the default security group, those edited rules then apply to all current and future servers in the group.

Inbound rules to allow ping and SSH are not automatically added to the default security group. You can modify the rules of the default security group using the REST API or the `ibmcloud cli`.

## Steps to modify the default security group rules using the CLI

1. Log in to IBM Cloud.

   If you have a federated account:
   ```
   ibmcloud login -sso
   ```
   {: pre}

   Otherwise, use this command:

   ```
   ibmcloud login
   ```
   {: pre}

2. Get the default security group ID and details for the VPC

   Run the following CLI command to list all VPCs:

   ```
   ibmcloud is vpc
   ```
   {: pre}

   The default security group name is shown under the column `Default Security Group`. Note the name of it so that you can find the `ID` when you list the security groups (next). 
   
   Now list all the security groups in the VPC:

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   Save the security group ID (for the default security group) in a variable so you can use it later. For example, using the variable name `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   To get details about the security group, run the following CLI command:

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   Alternatively, you could insert the actual ID value in place of the variable `$sg`.

3. Update the default security group -- add rules allowing SSH and PING

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


Adding and removing security group rules is an asynchronous operation. It usually takes from 1 to 30 seconds for the change to go into effect.
{: note}
