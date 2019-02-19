---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-01-24"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}

# Setting up connectivity between web servers in your VPC

This document takes you through some example steps for setting up communcation between two web servers in your {{site.data.keyword.cloud}} VPC, using Python. It shows you how to create communicating servers in the same zone and in different zones.

## Prerequisites

Before you can set up communication between web servers in your VPC, you must have already created a VPC with TWO subnets and at least TWO instances available in each subnet. You'll need to know the IDs of the VPC, subnets, and instances. Follow the steps in our [guide to creating VPC resources with the CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) if you need help setting up this configuration.

## Scenario 1: Connectivity between servers in the same zone but different subnets

`vpc` holds vpc ID
`subnet1` holds subnet 1 ID 
`subnet2` holds subnet 2 ID


## Part 1: Connectivity between servers from the same zone and VPC

After creating 2 or more instances in the same subnet that has a public gateway attached, we can test their connectivity by hosting a server with HTML code in one instance, then `curl` to that server and download the HTML file on the others or from your computer's browser.

Suppose you have `instance_1` and `instance_2` in the same subnet that has public gateway and same security group. You need to add rules using the API, CLI, or UI for the instance in security groups to allow inbound access. If you host your server on `instance_1` you must add a rule for `instance_2` floating IP. For example, suppose you have `instance_1` hosting server with IPv4 set to `169.61.161.161` and `instance_2` with IPv4 set to `169.61.161.235` and it tries to GET the file from `instance_1`. It should look like this:

![security group new rules](images/security-group-ui-ex1.png)

* To list all security groups `ibmcloud is sgs`
* Here is an API request that works to get the previous result:

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. Create a basic `index.html` file on `instance_1`. Here's an example of a basic HTML file:

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

2. At the same directory where the file is located, run the command `python -m SimpleHTTPServer <port number>`. You can use a different port if the default port 8000 is already in use.

3. SSH into `instance_2`

To look for a floating IP address, use the command `ibmcloud is ips`, and identify the `floating_ip_id` using the command `ibmcloud is ip $floating_ip_id`. You should be able to identify the public address.

4. Identify IP address of `instance_1`

5. Use `curl -X GET http://IP_address:Port_number/index.html` to request to download `index.html` from the hosting server `instance_1`. (For example: `curl -X GET http://169.61.161.161:8000/index.html`)

6. You should be able to see the output of `index.html`

7. (For access to the server by means of your local computer browser) Add rules for all inbound traffic as shown in the following figure:

![security group new rules with all inbound](images/security-group-ui-ex2.png)

* To add rules for allowing ALL protocols and from every source to your security groups

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**More detailed request example:**

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

8. Open a web browser, then enter `http://IP_address:Port_number/index.html`

9. You should see the index file shown in HTML format.

## Part 2: Connectivity between servers from a different zone and VPC

Connect the instance of the other subnet, which does not have the public gateway set up, to the instance with internet access. You want to set up 2 different types of instances:

* a web-server that handles requests from the internet using a public gateway
* a back-end sever that runs an application and stores important data, only sending internal traffic – Public gateway disabled

Security Groups (SG) apply to virtual server instances (VSI), and Network ACLs apply to subnets.

Therefore, to complete this overall task you must create VSIs, create subnets, create SG and ACL rules, then attach the SG to the network interface of an instance and attach the network ACL to a specific subnet.

* To list all rules in the `subnet1` Security Group

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. Remove the `inbound-allow-all-traffic` rule in the security group of the subnet exercise in part 1

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. Create a new VPC, then create a subnet without the public gateway attached.

3. Create an `instance_3` in the new subnet without a public gateway (no internet connection)- You may want to allocate larger CPU Cores and Memory to the instance for back-end reasons.

4. Add new rules for `instance_3` in the security group from part 1. For example: `instance_3` has a floating IP of `169.61.181.25`

![security group new rules with instance from other subnet inbound](images/security-group-ui-ex3.png)

5. Use a command of the form `curl -X GET http://IP_address:Port_number/index.html` to request to download `index.html` from the hosting server instance_1. (For example: `curl -X GET http://169.61.161.161:8000/index.html`)

6. You should be able to see the output of `index.html`

This means you have got a connection between instances 3 and 1, but not 2.

## Next steps

You'll want to protect your servers by making some ACL rules that control inbound traffic, as shown [in our guide to using ACLs in VPC](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli).
