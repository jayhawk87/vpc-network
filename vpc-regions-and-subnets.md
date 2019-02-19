---

copyright:
  years: 2018, 2019
  
lastupdated: "2019-02-12"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# Working with IP address ranges, address prefixes, regions, and subnets 

This document gives an overview of the available (or restricted) IP address ranges and user actions pertaining to regions and subnets, and to address prefixes. Certain IP address ranges are unavailable because they may be in use by IBM Cloud.  

When you create a subnet within your {{site.data.keyword.cloud}} VPC, an _address prefix_ is assigned automatically, based on the region and zone within which you've created your VPC. You can use the default address prefix, or you can manually change your address prefix, including a BYOIP address, in many cases. 

## IBM Cloud VPC and regions

An {{site.data.keyword.cloud}} VPC is deployed in a region, but may span multiple zones. Each region contains multiple zones, which represent independent fault domains.

Subnets in a VPC are assigned an _address prefix_, based on the region and zone in which they are created. The following table shows the available regions and zones. 

|   Location     | Region | API Endpoint | Status |
| ------- | :------: | :------: |:------: |
| Dallas | us-south | `us-south.iaas.cloud.ibm.com`| Available |
| Frankfurt | eu-de | `eu-de.iaas.cloud.ibm.com`| Available |
| Washington DC | us-east | `us-east.iaas.cloud.ibm.com`| Coming Soon |
| Tokyo | jp-tok | `jp-tok.iaas.cloud.ibm.com`| Coming Soon |
| London | eu-gb | `eu-gb.iaas.cloud.ibm.com`| Coming Soon |
| Sydney | au-syd | `au-syd.iaas.cloud.ibm.com`| Coming Soon |

## IBM Cloud VPC and subnets

Customers can (and usually do, as a best practice) divide an IBM Cloud VPC into subnets. All the subnets in an IBM Cloud VPC can reach one another though private L3 routing by an implicit router. You do not need to set up any routers or routes. 

Handy facts about subnets in VPC:

* A subnet consists of an IP address range that you specify. 
* A subnet is bound to a single zone, it cannot span multiple zones or regions.
* A subnet can span the entirety of the zone in an IBM Cloud VPC.
* You must create your VPC before you create your subnet(s) within that VPC.
* IPv6 support is not available.
* You can associate or disassociate a VSI to a subnet. (It requires that you add a vNIC and select a bandwidth.)

You can bring your own public IPv4 address range (BYOIP) to your IBM Cloud VPC account. When you use BYOIP, IBM Cloud must configure those IPv4 addresses on IBM Cloud resources, which will send packets to and from the provided addresses. Therefore, as a result of using your supplied IPv4 range on IBM Cloud, these IP addresses may be exposed to IBM's support staff and third parties as part of your use of this service.
{:important}

**Figure: A graphical representation of a VPC showing zones, regions, and subnets.**

![IBM Cloud VPC](images/vpc-experience.png)

## Using address prefixes for subnets

Each subnet must exist within an address prefix.
 * Subnet address prefixes are pre-allocated for each zone.
 * For your new subnet, you can pick your range of IP addresses from the existing address prefixes.
 * If the zone's address prefixes are not suitable, one of the existing address prefixes can be edited or a new address prefix can be added for the zone (using the API, CLI, or UI).
 
### Default VPC address prefixes
 
When a new VPC is created, the default address prefixes are assigned as follows, based on the region and zone:
 
* 'us-south-1' = '10.240.0.0/18'
* 'us-south-2' = '10.240.64.0/18'

Other zones or regions may assign different default prefixes, including these:
 
 * 10.240.0.0/18
 * 10.240.64.0/18
 * 10.240.128.0/18
 
### Address prefixes and the IBM Cloud console UI
 
When you create a VPC using the IBM Cloud Console UI, the system selects your address prefix automatically and requires you to create a  subnet within that default prefix. If this address scheme does not suit your requirements, you can customize the address prefixes after you create the VPC. You can then create subnets in your customized address prefixes, and delete the subnet you created with the default prefix.
 
This workaround is needed to use BYOIP through the IBM Cloud console UI.
{:note}

### Available IP addresses

These are available IP addresses, as defined in **RFC 1918**:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

### The following addresses are reserved addresses in a subnet

In this list, the IP addresses assume that the subnet's CIDR range is 10.10.10.0/24:

  * First address in the CIDR range (10.10.10.0): Network address
  * Second address in the CIDR range (10.10.10.1): Gateway address
  * Third address in the CIDR range (10.10.10.2): reserved by IBM
  * Fourth address in the CIDR range (10.10.10.3): reserved by IBM for future use
  * Last address in the CIDR range (10.10.10.255): Network broadcast address

### More about creating a subnet

There are two ways to specify a subnet for your VPC:
  * You can create a subnet by providing the size of subnet you need, such as the number of addresses supported (for example, 1024)
  * You can create a subnet by providing a CIDR range (such as 10.0.0.1/29)
  
As an example of how to specify a 1024 block using CIDR, if you're specifying a CIDR range rather than a subnet size, the IPv4 block `192.168.100.0/22` represents the 1024 IPv4 addresses from `192.168.100.0` to `192.168.103.255`.
{:tip}

### More about deleting a subnet
  * You cannot delete a subnet if resources (such as a Virtual Server Instance (VSI), floating IP, or public gateway) are in use in that subnet, the resources must be deleted first.
  * You can NOT resize an existing subnet. For example, 10.10.10.0/24 cannot be resized to 10.5.1.3/20
