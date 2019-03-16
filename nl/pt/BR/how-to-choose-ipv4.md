---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Escolhendo intervalos de IP para sua VPC
{: #choosing-ip-ranges-for-your-vpc}

Use a notação CIDR, tal como:

* `<IPv4 address>/number` (exemplo de endereço da VPC: 10.10.0.0/16).

Você desejará reservar os últimos 16 bits (65.536 endereços) do IPv4 como 0s para que seja possível usá-los para vários endereços IP de sub-rede dentro da mesma VPC do {{site.data.keyword.cloud}} (exemplo de endereço IP de sub-rede: 10.10.1.0/24).

**Nota:** essa convenção é recomendada pelo RFC 1918.

Lembre-se de que, caso você seja novo na notação CIDR, quanto menor o número após a barra, **mais** endereços IP você estará alocando, pois o número após a barra representa o número de bits 1 iniciais na máscara de prefixo da sub-rede.

Se você precisa de mais informações, vários artigos excelentes referentes a _Classless Inter-Domain Routing_ (CIDR) podem ser localizados on-line.
