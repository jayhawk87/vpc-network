---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-01-23"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Création d'une connexion sécurisée avec un homologue FortiGate distant

Ce document est basé sur FortiGate 300C, version de microprogramme v5.2.13, build762 (disponibilité générale). 

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'API ou de l'interface de ligne de commande {{site.data.keyword.cloud}} pour créer des clouds privés virtuels. Pour plus d'informations, voir [Initiation](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) et [Configuration de VPC avec des API](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Exemple d'étapes
La topologie de connexion à l'homologue FortiGate distant est similaire à la création d'une connexion VPN entre deux VPC. Cependant, un côté est remplacé par l'unité FortiGate 300C.

![entrer la description de l'image ici](./images/vpc-vpn-fg-figure.png)

### Pour créer une connexion sécurisée avec l'homologue Fortigate distant

Accédez à **VPN \> IPsec \> Tunnels** et créez un nouveau tunnel personnalisé ou modifiez un tunnel existant.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

Lorsqu'une unité FortiGate reçoit une demande de connexion d'un homologue VPN distant, elle utilise les paramètres IPsec Phase 1 pour établir une connexion sécurisée et authentifier cet homologue VPN. Ensuite, si la stratégie de sécurité autorise la connexion, l'unité FortiGate établit le tunnel à l'aide des paramètres de phase IPsec et applique la stratégie de sécurité IPsec. Les services de gestion de clés, d'authentification et de sécurité sont négociés de manière dynamique via le protocole IKE.

**Pour prendre en charge ces fonctions, les étapes de configuration générales suivantes doivent être effectuées par l'unité FortiGate :**

* Définissez les paramètres de phase 1 requis par l'unité FortiGate pour authentifier l'homologue distant et établir une connexion sécurisée.

* Définissez les paramètres de phase 2 nécessaires à l'unité FortiGate pour créer un tunnel VPN avec l'homologue distant.

* Créez des stratégies de sécurité pour contrôler les services autorisés et la direction autorisée du trafic entre les adresses source et de destination IP.

Par défaut, certains paramètres types des phases 1 et 2 ont été définis.

### Configuration pour le VPN d'IBM Cloud VPC

Pour vous connecter à la fonctionnalité VPN d'IBM Cloud VPC, nous vous recommandons la configuration suivante :

1. Choisissez IKEv2 dans l'authentification ;
2. Activez `DH-group 2` dans la proposition de phase 1
3. Définissez `lifetime = 36000` dans la proposition de phase 1
4. Désactivez PFS dans la proposition de phase 2
5. Définissez `lifetime = 10800` dans la proposition de phase 2
6. Entrez les informations de votre sous-réseau en phase 2
7. Les paramètres restants conservent leurs valeurs par défaut.

![entrer la description de l'image ici](./images/vpc-vpn-fg-network.JPG)

### Paramètres réseau

![entrer la description de l'image ici](./images/vpc-vpn-fg-authentication.JPG)

### Paramètres d'authentification

![entrer la description de l'image ici](./images/vpc-vpn-fg-phase1.JPG)

### Paramètres de la phase 1

![entrer la description de l'image ici](./images/vpc-vpn-fg-phase2.JPG)

### Paramètres de la phase 2

## Pour créer une connexion sécurisée avec le VPC IBM local

Pour créer une connexion sécurisée, créez une connexion VPN au sein de votre VPC, similaire à l'exemple VPC 2.

* Créez une passerelle VPN sur votre sous-réseau VPC avec une connexion VPN entre le VPC et l'unité FortiGate, en définissant `local_cidrs` à la valeur de sous-réseau sur le VPC et `peer_cidrs` à la valeur de sous-réseau sur l'unité FortiGate.

**Remarque :** Le statut de la passerelle apparaît en tant qu'`en attente` lors de la création de la passerelle VPN et devient `disponible` une fois la création terminée. La création peut prendre du temps.

![entrer la description de l'image ici](images/vpc-vpn-fg-connection.png)

### Vérification du statut de la connexion sécurisée

Vous pouvez vérifier le statut de votre connexion via la console IBM Cloud. En outre, vous pouvez tenter de créer un fichier `ping` d'un site à l'autre à l'aide des VSI.

![entrer la description de l'image ici](images/vpc-vpn-fg-status.JPG)
