---

copyright:
  years: 2017, 2018
lastupdated: "2018-10-31"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Choosing IP Ranges for your VPC

Use CIDR notation such as:

* `<IPv4 address>/number` (VPC address example: 10.10.0.0/16).

You'll want to reserve the last 16 bits (65,536 addresses) of the IPv4 as 0s so that you can use them for various subnet IP addresses within the same {{site.data.keyword.cloud}} VPC (Subnet IP address example: 10.10.1.0/24).

**Note:** This convention is recommended by RFC 1918.

Remember, just in case you are new to CIDR notation, the smaller the number after the slash, the **more** IP addresses you are allocating, because the number after the slash represents the number of leading 1 bits in the subnet's prefix mask.

If you need more information, a number of excellent articles regarding _Classless Inter-Domain Routing_ (CIDR) can be found online.
