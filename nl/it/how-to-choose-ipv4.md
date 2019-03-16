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


# Scelta degli intervalli di IP per il tuo VPC
{: #choosing-ip-ranges-for-your-vpc}

Utilizza un'annotazione CIDR come ad esempio:

* `<IPv4 address>/number` (esempio indirizzo VPC: 10.10.0.0/16).

Vorrai riservare gli ultimi 16 bit (65.536 indirizzi) dell'IPv4 come degli 0 in modo da poterli utilizzare per vari indirizzi IP della sottorete all'interno dello stesso VPC {{site.data.keyword.cloud}} (esempio indirizzo IP sottorete: 10.10.1.0/24).

**Nota:** questa convenzione è consigliata dal RFC 1918.

Ricorda, nel caso tu non abbia esperienza con l'annotazione CIDR, più piccolo è il numero dopo la barra e **più** indirizzi IP stai assegnando, perché il numero dopo la barra rappresenta il numero di bit iniziali 1 nella maschera del prefisso della sottorete.

Se hai bisogno di ulteriori informazioni, possono essere trovati online molti eccellenti articoli riguardanti CIDR (_Classless Inter-Domain Routing_).
