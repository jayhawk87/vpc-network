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


# Sélection des plages d'adresses IP pour votre VPC
{: #choosing-ip-ranges-for-your-vpc}

Utilisez la notation CIDR telle que :

* `<IPv4 address>/number` (Exemple d'adresse VPC : 10.10.0.0/16).

Réservez les 16 derniers bits (65 536 adresses) de l'IPv4 en tant que zéros (0) afin de pouvoir les utiliser pour différentes adresses IP de sous-réseau au sein du même VPC {{site.data.keyword.cloud}} (exemple d'adresse IP de sous-réseau : 10.10.1.0/24). 

**Remarque :** Cette convention est recommandée par RFC 1918.

Au cas où vous ne seriez pas familiarisé avec la notation CIDR, n'oubliez pas que plus le nombre après la barre oblique est petit, **plus** vous allouez d'adresses IP, car le nombre situé après la barre oblique représente le nombre de bits 1 de tête dans le masque de préfixe du sous-réseau.

Pour consulter plus d'informations, vous trouverez d'excellents articles en ligne sur _Classless Inter-Domain Routing_ (CIDR).
