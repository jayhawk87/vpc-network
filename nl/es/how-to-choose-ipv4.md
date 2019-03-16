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


# Selección de rangos de IP para la VPC
{: #choosing-ip-ranges-for-your-vpc}

Utilice la notación CIDR, como por ejemplo:

* `<IPv4 address>/number` (ejemplo de dirección VPC: 10.10.0.0/16).

Es posible que desee reservar los últimos 16 bits (65.536 direcciones) del IPv4 como 0 para poder utilizarlos en varias direcciones IP de subred del mismo VPC de {{site.data.keyword.cloud}} (ejemplo de dirección IP de subred: 10.10.1.0/24).

**Nota:** RFC 1918 recomienda esta convención.

Recuerde que, si es nuevo en lo que respecta a la notación CIDR, cuanto menor sea el número situado tras la barra inclinada, **más** direcciones IP asignará, puesto que dicho número representa el número de 1 bits iniciales en la máscara de prefijo de la subred.

Si necesita más información, encontrará artículos en línea relacionados con el _Direccionamiento entre dominios sin clases_ (CIDR).
