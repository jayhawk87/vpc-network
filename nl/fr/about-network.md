---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-20"


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

# A propos de la mise en réseau pour VPC
{: #about-networking-for-vpc}

Un cloud privé virtuel (VPC) est un réseau virtuel lié à votre compte client. Il vous offre une sécurité cloud, avec la possibilité d'évoluer de manière dynamique, en fournissant un contrôle fin de votre infrastructure virtuelle et de la segmentation de votre trafic réseau.

Le présent document décrit certains concepts de mise en réseau lorsqu'ils sont appliqués dans un VPC {{site.data.keyword.cloud}}. Les caractéristiques et les cas d'utilisation
décrits sont les suivants : 

* régions
* zones
* adresses IP réservées
* passerelles publiques
* interfaces vNIC
* sous-réseaux
* adresses IP flottantes
* connexions VPN

## Présentation

Pour configurer la mise en réseau dans votre VPC : 
* Utilisez une passerelle publique pour le trafic Internet de sous-réseau. 
* Utilisez une adresse IP flottante pour le trafic Internet VSI. 
* Utilisez un VPN pour une connectivité externe sécurisée. 

Rappel : 
* Certaines plages CIDR d'adresses de sous-réseau sont réservées par IBM. 
* Vous devez créer votre VPC avant de créer des sous-réseaux au sein de ce VPC. 
* La prise en charge d'IPv6 n'est pas disponible. 

Vous pouvez éventuellement créer un VPC en accès classique pour vous connecter à votre infrastructure classique IBM Cloud.
{:note}

Comme le montre la figure suivante : 
* Les sous-réseaux dans votre VPC peuvent se connecter à Internet via une passerelle publique facultative (PGW). 
* Vous pouvez attribuer une adresse IP flottante (FIP) à n'importe quelle VSI pour l'atteindre à partir d'Internet ou inversement. 
* Les sous-réseaux dans le VPC IBM Cloud offrent une connectivité privée ; ils peuvent communiquer via une liaison privée, par le biais du routeur implicite. La configuration de routes n'est pas nécessaire.


**Figure : Un client peut subdiviser un cloud privé virtuel en sous-réseaux, qui peuvent chacun accéder à Internet, si nécessaire.**

![Connectivité VPC](/images/vpc-connectivity-and-security.png)

## Terminologie

Ce [glossaire](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary) contient des définitions et des informations sur les termes utilisés dans le présent document pour IBM Cloud VPC. Lorsque vous utilisez votre VPC, vous devez être familiarisé avec les concepts de base de _région_ et de _zone_, car ils s'appliquent à votre déploiement. 

### Régions

Une région est une abstraction liée à la zone géographique dans laquelle un VPC est déployé. Chaque région contient plusieurs zones, qui représentent des domaines d'erreur indépendants. Un VPC IBM Cloud peut s'étendre sur plusieurs zones dans la région qui lui a été affectée. 

### Zones

Une zone est une abstraction qui fait référence au centre de données physique qui héberge les ressources de calcul, réseau et de stockage, ainsi que les concepts de refroidissement et d'alimentation, qui offrent des services et des applications. Les zones sont isolées les unes des autres, afin de créer un point de défaillance unique non partagé, d'améliorer la tolérance aux pannes et de réduire le temps d'attente. Une zone garantit les propriétés suivantes :

 * Chaque zone est un domaine de panne indépendant et il est extrêmement improbable que deux zones d'une région tombent en panne simultanément. 
 * Le trafic entre les zones d'une région a moins de 2 ms de temps d'attente. 

## Caractéristiques des sous-réseaux dans le VPC

Un sous-réseau est constitué d'une plage d'adresses IP spécifiée (bloc CIDR). Les sous-réseaux sont liés à une seule zone et ils ne peuvent pas couvrir plusieurs zones ou régions. Toutefois, un sous-réseau peut couvrir l'intégralité des abstractions de zone dans leur cloud privé virtuel. Les sous-réseaux du même IBM Cloud VPC sont connectés les uns aux autres.


### Adresses IP réservées

Certaines adresses IP sont réservées à l'utilisation par IBM lors de l'exploitation du cloud privé virtuel. Voici les adresses réservées (ces adresses IP supposent que la plage CIDR du sous-réseau est 10.10.10.0/24) :

  * Première adresse dans la plage CIDR (10.10.10.0) : adresse réseau
  * Deuxième adresse dans la plage CIDR (10.10.10.1) : adresse de passerelle
  * Troisième adresse de la plage CIDR (10.10.10.2) : réservée par IBM
  * Quatrième adresse de la gamme CIDR (10.10.10.3) : réservée par IBM pour une utilisation future
  * Dernière adresse dans la plage CIDR (10.10.10.255) : adresse de diffusion réseau

### Utilisation d'une passerelle publique pour la connectivité externe d'un sous-réseau

Une **passerelle publique (PGW)** permet à un sous-réseau (dont toutes les VSI sont liées au sous-réseau) de se connecter à Internet. Notez que les sous-réseaux sont privés par défaut. Toutefois, vous pouvez éventuellement créer une passerelle PGW et associer un sous-réseau à cette dernière. Une fois qu'un sous-réseau est connecté à la PGW, toutes les VSI de ce sous-réseau peuvent se connecter à Internet.

La PGW utilise la conversion _NAT plusieurs-à-1_, ce qui signifie que des milliers de VSI dotées d'adresses privées utilisent une adresse IP publique pour communiquer avec l'Internet public. La PGW ne permet pas à Internet d'établir une connexion avec ces instances. Utilisez l'API pour connecter et déconnecter des sous-réseaux vers et depuis votre PGW.

La figure suivante résume la portée actuelle des services de passerelle.

![services passerelle](images/scope-of-gateway-services.png)

Une passerelle publique est créée dans un VPC, mais elle ne fait rien tant qu'elle n'est pas connectée à un sous-réseau. Vous ne pouvez créer qu'une seule passerelle publique par zone, ce qui signifie par exemple que vous pouvez avoir 3 passerelles publiques par VPC dans un environnement à 3 zones.
{:note}

### Utilisation d'une adresse IP flottante pour la connectivité externe d'une VSI
Les **adresses IP flottantes** sont les adresses IP fournies par le système et accessibles depuis l'internet public.

Vous pouvez réserver une adresse IP flottante dans le pool d'adresses IP flottantes disponibles fournies par IBM et vous pouvez l'associer ou la dissocier à n'importe quelle instance du même cloud privé virtuel au moyen de la carte vNIC correspondant à cette instance. Toute adresse IP flottante peut être associée à différents types d'instances de serveur virtuel (VSI), par exemple un équilibreur de charge ou une passerelle VPN.

Votre adresse IP flottante ne peut pas être associée à plusieurs interfaces.Vous devez spécifier l'interface sur la VSI qui sera associée à cette IP flottante individuelle. Cette interface possède également une adresse IP privée. Le système de back end effectue des opérations _NAT 1-à-1_ entre l'adresse IP flottante et l'adresse IP privée de cette interface. Une adresse IP flottante est libérée uniquement lorsque vous demandez spécifiquement l'action de libération.

**Remarques :**
* **L'association d'une adresse IP flottante à une VSI supprime la VSI de la conversion NAT plusieurs-à-un de la passerelle PGW mentionnée précédemment.**
* **Actuellement, les adresses IP flottantes ne prennent en charge que les adresses IPv4.**
* **Vous ne pouvez pas utiliser votre propre adresse IP publique comme adresse IP flottante.**

Pour plus d'informations sur les opérations NAT, reportez-vous au [document RFC Internet associé ![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilisation d'un VPN pour une connectivité externe sécurisée
Le service de réseau privé virtuel (VPN) est disponible pour permettre aux utilisateurs de se connecter en toute sécurité à leur IBM Cloud VPC à partir d'Internet.

**Fonctionnalités de VPN**
  * Possibilité d'effectuer une opération CRUD (Create, Read, Update and Delete - Créer, Lire, Mettre à jour et Supprimer) sur un service VPN (VPN IPSEC de site à site) pour gérer le transfert de données.
  * Possibilité d'associer un service VPN à IBM Cloud VPC ou au réseau virtuel d'un client.
  * Possibilité d'identifier les sous-réseaux sur site du client, soit par définition statique, soit par routage dynamique.
  * Prise en charge de chiffrements sécurisés tels que SHA256, AES, 3DES, IKEv2
  * Fiabilité du service intégré.
  * Surveillance continue de la santé de la connexion VPN.
  * Prise en charge des modes initiateur et répondeur ; c'est-à-dire que le trafic peut être lancé de chaque côté du tunnel.

Pour une liste des limitations connues et des fonctionnalités non prises en charge, veuillez vous référer au document [Limitations connues](/docs/infrastructure/vpc?topic=vpc-known-limitations).

## En savoir plus

   * [Sécurité d'un VPC IBM](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [Adresses, régions et sous-réseaux d'un VPC IBM](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
