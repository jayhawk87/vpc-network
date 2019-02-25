---

copyright:
  years: 2019

lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Setting up Network ACLs using the CLI
{: #setting-up-network-acls-using-the-cli}

By using the Access Control List (ACL) functionality available in {{site.data.keyword.cloud}} Virtual Private Cloud, you can control all incoming and outgoing traffic related to your critical business workloads on the cloud. Similar to the security group, an ACL is a built-in, virtual firewall. In contrast to security groups, ACL rules control traffic to and from the subnets, rather than to and from the instances.

The following table summarizes some key differences between security groups and ACLs:

|  | Security Groups | ACLs    |
|-------------|-----------------|---------|
| Control level  | VSI instance    | Subnet  |
| State   | Stateful - Return traffic is allowed by default | Stateless - Return traffic is denied by default and needs to be explicitly allowed |
| Supported rules | Uses allow rules only | Uses allow and deny rules|
| How rules are applied | All rules are considered | Rules are processed in sequence |
| Relationship to the associated resource | An instance can be associated with multiple security groups| Multiple subnets can be associated with the same ACL|

The example given in this document shows how to create network ACLs in your VPC, which will protect your subnets.

## APIs and CLIs are available for ACLs

You can set up and manage ACLs through the API or through the CLI, as you prefer.

* See the [API Reference](https://{DomainName}/apidocs/rias) for the parameters, request body, and response details for each API.

* See this [CLI example](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) to get command details, steps for installing the CLI plugin, and the prerequisite steps of using the VPC CLI.

## Working with ACLs and ACL rules

To make your ACLs effective, you'll create rules that determine how to handle your inbound and outbound network traffic. You can create multiple inbound and outbound rules. See [Quotas](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-quotas) for specific information about how many rules you can create.

* With inbound and outbound rules, you can allow or deny traffic from a source IP range or to a destination IP range, with specified protocols and ports.  
* ACL rules are considered in sequence. Low-priority rules are evaluated only when the higher-priority rules do not match.
* Inbound rules are separated from outbound rules.
* The protocols currently supported are TCP, UDP, and ICMP. You also can use the **all** option to designate _all_ or _other_ protocols (if a rule with a higher priority is specified).

For relevant information about using ICMP, TCP, and UDP protocols in your ACL rules, see [Understanding Internet Communication Protocols](https://{DomainName}/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols).

### Attaching an ACL to a subnet

You have two options when attaching an ACL to a subnet:

* You can create a new subnet, and specify an ACL to attach. If you don't specify an ACL, a default network ACL is attached. The default ACL allows all inbound traffic destined for this subnet, and all outbound traffic from this subnet.
* You can attach an ACL to an existing subnet. If another ACL is attached to this subnet already, that ACL is detached before the new ACL is attached.

## ACL demo example

In the example that follows, you'll be able to create two ACLs and associate them with two subnets, using the command line interface (CLI). Here's what the scenario looks like:

![An example ACL scenario](images/vpc-acls.png)

As the figure illustrates, you have two web servers dealing with requests from the Internet and two backend servers that you want to hide from the public. In this example, you place the servers into two separate subnets, 10.10.10.0/24 and 10.10.20.0/24 respectively, and you need to allow the web servers to exchange data with the backend servers. Also, you want to allow limited outbound traffic from the backend servers.

### Example rules

The example rules that follow show how to set up the ACL rules for a basic scenario as described previously.

As a best practice, it's recommended that you give fine-grained rules a higher priority than coarse-grained rules. For example, if you have a rule blocking all traffic from the subnet 10.10.30.0/24, and it is matched with a higher priority, the fine-grained rule with lower priority to allow traffic from 10.10.30.5 will never be applied.
{:note}

**ACL-1 example rules**:

| Inbound/Outbound| Allow/Deny | Source/Destination IP | Protocol | Port | Description|
|--------------|-----------|------|------|------------------|-------|
| Inbound | Allow | 0.0.0.0/0 | TCP| 80 | Allow HTTP traffic from the Internet|
| Inbound | Allow | 0.0.0.0/0 | TCP | 443 | Allow HTTPS traffic from the Internet|
| Inbound | Allow| 10.10.20.0/24 |all| all| Allow all inbound traffic from the subnet 10.10.20.0/24 where the backend servers are placed|
| Inbound | Deny| 0.0.0.0/0|all| all| Deny all other traffic inbound|
| Outbound | Allow | 0.0.0.0/0 | TCP|80 | Allow HTTP traffic to the Internet|
| Outbound | Allow | 0.0.0.0/0 | TCP|443 | Allow HTTPS traffic to the Internet|
| Outbound | Allow| 10.10.20.0/24 |all| all| Allow all outbound traffic to the subnet 10.10.20.0/24 where the backend servers are placed|
| Outbound | Deny| 0.0.0.0/0|all| all| Deny all other traffic outbound|


**ACL-2 example rules**:

| Inbound/Outbound| Allow/Deny | Source/Destination IP | Protocol| Port | Description|
|--------------|-----------|------|------|------------------|--------|
| Inbound | Allow| 10.10.10.0/24 |all| all| Allow all inbound traffic from the subnet 10.10.10.0/24 where the web servers are placed|
| Inbound | Deny| 0.0.0.0/0|all| all| Deny all other traffic inbound|
| Outbound | Allow | 0.0.0.0/0 | TCP| 80 | Allow HTTP traffic to the Internet|
| Outbound | Allow | 0.0.0.0/0 | TCP| 443 | Allow HTTPS traffic to the Internet|
| Outbound | Allow| 10.10.10.0/24 |all| all| Allow all outbound traffic to the subnet 10.10.10.0/24 where the web servers are placed|
| Outbound | Deny| 0.0.0.0/0|all| all| Deny all other traffic outbound|

This example illustrates general cases only. In your scenarios, you also may want to have more granular control over the traffic:

* You could have a network administrator who needs access to the 10.10.10.0/24 subnet from a remote network for operation purposes. In that case, you'll need to allow SSH traffic from the Internet to this subnet.
* You might want to narrow down the protocol scope that you allow between your two subnets.

### Example steps

The following example steps skip the prerequisite steps of using the CLI to create a VPC, which must be done first. For more information, see [Creating a VPC using the CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).


#### Step 1. Create the ACLs

Here are CLI commands you could use to create two ACLs, named `my_web_subnet_acl` and `my_backend_subnet_acl`:

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

The response includes the newly created ACL IDs. Save the IDs of both ACLs to be used in later commands. You could use variables named `webacl` and `bkacl`, like this:

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### Step 2. Retrieve the default ACL rules

Before adding new rules, retrieve the default inbound and outbound ACL rules so that you can insert new rules before them.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

The response shows the default inbound and outbound rules that allow all IPv4 traffic in all protocols.

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

Save the IDs of both ACL rules as variables, you'll use them in later commands. For example, you could use variables named `inrule` and `outrule`:

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### Step 3. Add new ACL rules as described

This section shows adding inbound rules first, then adding outbound rules.

Insert new inbound rules before the default inbound rule.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

At each step, save the ID of the ACL rule in a variable, it will be used in later commands. For example, you could use the variable name `acl200`:

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

Now add the rule to `acl200`:

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

Add more rules until your ACL setup is complete, saving each ID as a variable.

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

Insert new outbound rules before the default outbound rule.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### Step 4. Create the two subnets with the newly created ACL

You'll create two subnets so that each of your ACLs is associated with one of the new subnets. 

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## Command list cheat sheet

To show a complete list of the available VPC CLI commands for ACLs:

```
ibmcloud is network-acls
```
{: pre}

To see your ACL and its metadata including rules:

```
ibmcloud is network-acl $webacl
```
{: pre}

To get the default inbound ACL rule:

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## Example inbound `ping` rule

To add an ACL rule, here's an example command for adding a `ping` inbound rule before the default inbound rule:

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
