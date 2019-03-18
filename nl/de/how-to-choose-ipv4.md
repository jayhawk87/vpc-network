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


# IP-Bereiche für Ihre VPC auswählen
{: #choosing-ip-ranges-for-your-vpc}

Verwenden Sie die CIDR-Notation wie folgt:

* `<IPv4 address>/nummer` (Beispiel einer VPC-Adresse: 10.10.0.0/16)

Sie sollten die letzten 16 Bit (65.536 Adressen) der IPv4-IP als Nullwerte reservieren, damit Sie sie für unterschiedliche Teilnetz-IP-Adressen in derselben {{site.data.keyword.cloud}}-VPC verwenden können (Beispiel einer Teilnetz-ID: 10.10.1.0/24). 

**Hinweis:** Diese Konvention wird von RFC 1918 empfohlen.

Für den Fall, dass Sie mit der CIDR-Notation noch nicht vertraut sind, sollten Sie Folgendes berücksichtigen: Je kleiner die Zahl nach dem Schrägstrich ist, umso **mehr** IP-Adressen werden zugewiesen, da die Zahl nach dem Schrägstrich die Anzahl der führenden 1-Bits in der Präfixmaske des Subnetzes darstellt.

Falls Sie weitere Informationen benötigen, können Sie online eine Reihe hervorragender Artikel zum Thema _Classless Inter-Domain Routing (CIDR)_ finden.
