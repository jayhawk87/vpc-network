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
{:download: .download}

# Available IP address ranges, regions, and subnets 

When you create a subnet within your {{site.data.keyword.cloud}} VPC, an _address prefix_ is assigned automatically, based on the region and zone within which you've created your VPC. You can use the default address prefix, or you can manually change your address prefix, including a BYOIP address, in many cases. However, certain IP address ranges are unavailable because they may be in use by IBM Cloud. This document gives an overview of the available (or restricted) IP address ranges and user actions pertaining to regions and subnets. 

## IBM Cloud VPC and Regions

An {{site.data.keyword.cloud}} VPC spans multiple zones. Each region contains multiple zones, which represent independent fault domains.

![IBM Cloud VPC](images/vpc-experience.png)


**Notes**

 * **You must create your VPC before you create your subnet(s) within that VPC.**
 * **IPv6 support is not available.**

## IBM Cloud VPC and Subnets

Customers can (and usually do, as a best practice) divide an IBM Cloud VPC into subnets. Handy facts about subnets in VPC:

* A subnet consists of an IP address range that you specify. 
* A subnet is bound to a single zone, which cannot span multiple zones or regions.
* A subnet can span the entirety of the zone in an IBM Cloud VPC. 

All the subnets in an IBM Cloud VPC can reach one another though private L3 routing by an implicit router. You do not need to set up any routers or routes.

### Address prefixes and subnets

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
 
**Note that this workaround is needed to use BYOIP through the IBM Cloud console UI.**

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

### Creating a subnet in a VPC
  * You can create a subnet by providing the size of subnet you need, such as the number of addresses supported (for example, 1024)
  * You can create a subnet by providing a CIDR range (such as 10.0.0.1/29)

As an example of how to specify a 1024 block using CIDR, if you're specifying a CIDR range rather than a subnet size, the IPv4 block `192.168.100.0/22` represents the 1024 IPv4 addresses from `192.168.100.0` to `192.168.103.255`.

### Deleting a subnet
  * You cannot delete a subnet if resources (such as a Virtual Server Instance (VSI)) are in use in that subnet, the resources must be deleted first.
  * You can associate or disassociate a VSI with a subnet (it requires that you add a vNIC and select a bandwidth)
  * You can NOT resize an existing subnet. For example, 10.10.10.0/24 cannot be resized to 10.5.1.3/20

You can bring your own public IPv4 address range (BYOIP) to your IBM Cloud VPC account. When you use BYOIP, IBM Cloud must configure those IPv4 addresses on IBM Cloud resources, which will send packets to and from the provided addresses. Therefore, as a result of using your supplied IPv4 range on IBM Cloud, these IP addresses may be exposed to IBM's support staff and third parties as part of your use of this service.
{:important}
