---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-01-16"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Getting started with Networking for Virtual Private Cloud

The Beta program has closed, access is no longer available. Thank you for your interest, your participation, and your feedback. Stay tuned for product availability announcements.
{: important}


To get started with {{site.data.keyword.cloud}} Networking for Virtual Private Cloud

1. Create a Virtual Private Cloud.
2. Create one or more subnets in the Virtual Private Cloud in one or more zones.
3. Create a public gateway (PGW) on a subnet if you want resources on your subnet to have access to the internet or vice-versa.

## Prerequisites

 * **User Permissions**: Be sure that your user has sufficient permissions to create and manage resources in your VPC. For a list of required permissions, see [Granting permissions needed for VPC users](../vpc/vpc-user-permissions.html).

### Use the UI, CLI, or REST API to provision VPC Network Resources

The provisioning and managing of VPC network resources can be done through the UI, CLI or REST API.

* For access through the user interface, log into the [IBM Cloud Console ![External link icon](../../icons/launch-glyph.svg "External link icon")]( https://{DomainName}/vpc){: new_window}. Follow the [UI guide](../vpc/console-tutorial.html) if you need help.
* To use the command line interface, use the [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) plugin of the [IBM Cloud CLI ![External link icon](../../icons/launch-glyph.svg "External link icon")](/docs/cli/reference/bluemix_cli/get_started.html#getting-started){: new_window} and follow the [Hello World](../vpc/hello-world-vpc.html) example.
* For more advanced users, you can call the [REST APIs](https://{DomainName}/apidocs/rias) directly. Follow the [example code](../vpc/example-code.html) tutorial to get started with the REST APIs.

## Next steps

After your Virtual Private Cloud networking has been provisioned, explore more.

* [Creating and managing virtual server instances](../vpc/create-manage-vsi.html)
* [Using Security Groups](security-groups.html)
* [Using Network ACLs](using-acls.html)
* [(Beta) Using VPN](using-vpn.html)
* [(Beta) Using Load Balancers](using-lbaas.html)
