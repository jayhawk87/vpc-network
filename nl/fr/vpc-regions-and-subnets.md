---

copyright:
  years: 2018, 2019
  
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

# Utilisation de plages d'adresses IP, de préfixes d'adresse, de régions et de sous-réseaux 
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}

Le présent document donne une vue d'ensemble des plages d'adresses IP disponibles (ou restreintes) et des actions utilisateur relatives aux régions, aux sous-réseaux et aux préfixes d'adresse. Certaines plages d'adresses IP ne sont pas disponibles car elles sont peut-être utilisées par IBM Cloud.   

Lorsque vous créez un sous-réseau dans votre VPC {{site.data.keyword.cloud}}, un _préfixe d'adresse_ est attribué automatiquement, en fonction de la région et de la zone dans lesquelles vous avez créé votre VPC. Vous pouvez utiliser le préfixe d'adresse par défaut ou le modifier manuellement, y compris une adresse BYOIP, dans de nombreux cas. 

## VPC et régions IBM Cloud

Un VPC {{site.data.keyword.cloud}} est déployé dans une région, mais peut couvrir plusieurs zones. Chaque région contient plusieurs zones, qui représentent des domaines d'erreur indépendants.

Un _préfixe d'adresse_ est affecté aux sous-réseaux d'un VPC, en fonction de la région et de la zone où ils sont créés. Le tableau suivant présente les régions et zones disponibles.  

|   Emplacement     | Région | Noeud final d'API | Statut |
| ------- | :------: | :------: |:------: |
| Dallas | us-south | `us-south.iaas.cloud.ibm.com`| Disponible |
| Francfort | eu-de | `eu-de.iaas.cloud.ibm.com`| Disponible |
| Washington DC | us-east | `us-east.iaas.cloud.ibm.com`| Bientôt disponible |
| Tokyo | jp-tok | `jp-tok.iaas.cloud.ibm.com`| Bientôt disponible |
| Londres | eu-gb | `eu-gb.iaas.cloud.ibm.com`| Bientôt disponible |
| Sydney | au-syd | `au-syd.iaas.cloud.ibm.com`| Bientôt disponible |

## VPC et sous-réseaux IBM Cloud

Les clients peuvent (et le font généralement, dans le cadre des bonnes pratiques) fractionner une instance IBM Cloud VPC en sous-réseaux.Tous les sous-réseaux d'une instance IBM Cloud VPC peuvent se joindre par le biais d'un routage L3 privé via un routeur implicite.Il est inutile de configurer des routeurs ou des routes. 

Informations utiles sur les sous-réseaux dans le VPC :

* Un sous-réseau consiste en une plage d'adresses IP que vous spécifiez. 
* Un sous-réseau est lié à une zone unique. Il ne peut pas couvrir plusieurs zones ou régions. 
* Un sous-réseau peut couvrir la totalité de la zone dans une instance IBM Cloud VPC.
* Vous devez créer votre VPC avant de créer vos sous-réseaux au sein de ce VPC.
* La prise en charge d'IPv6 n'est pas disponible. 
* Vous pouvez associer ou dissocier une VSI sur un sous-réseau. (Vous devez ajouter une carte vNIC et sélectionner une bande passante.) 

Vous pouvez apporter votre propre plage d'adresses IPv4 publiques (BYOIP) à votre compte IBM Cloud VPC. Lorsque vous utilisez BYOIP, IBM Cloud doit configurer ces adresses IPv4 sur des ressources IBM Cloud, qui envoient des paquets à destination et en provenance des adresses fournies. Par conséquent, en raison de l'utilisation de votre plage d'adresses IPv4 fournie sur IBM Cloud, ces adresses IP peuvent être exposées au personnel d'assistance IBM et à des tiers dans le cadre de votre utilisation de ce service.
{:important}

**Figure : Représentation graphique d'un VPC montrant les zones, les régions et les sous-réseaux**

![IBM Cloud VPC](images/vpc-experience.png)

## Utilisation de préfixes d'adresse pour les sous-réseaux

Chaque sous-réseau doit se trouver dans un préfixe d'adresse.
 * Les préfixes d'adresse de sous-réseau sont pré-alloués pour chaque zone.
 * Pour votre nouveau sous-réseau, vous pouvez sélectionner la plage d'adresses IP parmi les préfixes d'adresse existants.
 * Si les préfixes d'adresse de la zone ne conviennent pas, vous pouvez modifier l'un des préfixes d'adresse existants ou en ajouter un nouveau pour la zone (à l'aide de l'API, de la CLI ou de l'interface utilisateur).
 
### Préfixes d'adresse VPC par défaut
 
Lorsqu'un VPC est créé, les préfixes d'adresse par défaut sont attribués comme suit, en fonction de la région et de la zone :
 
* 'us-south-1' = '10.240.0.0/18'
* 'us-south-2' = '10.240.64.0/18'

D'autres zones ou régions peuvent attribuer différents préfixes par défaut, notamment :
 
 * 10.240.0.0/18
 * 10.240.64.0/18
 * 10.240.128.0/18
 
### Préfixes d'adresse et interface utilisateur de la console IBM Cloud
 
Lorsque vous créez un VPC à l'aide de l'interface utilisateur de la console IBM Cloud, le système sélectionne automatiquement votre préfixe d'adresse et vous oblige à créer un sous-réseau avec ce préfixe par défaut. Si ce schéma d'adresse ne correspond pas à vos besoins, vous pouvez personnaliser les préfixes d'adresse après avoir créé le VPC. Vous pouvez ensuite créer des sous-réseaux dans vos préfixes d'adresses personnalisés et supprimer le sous-réseau que vous avez créé avec le préfixe par défaut.
 
Cette solution de contournement est nécessaire pour utiliser BYOIP via l'interface utilisateur de la console IBM Cloud. {:note}

### Adresses IP disponibles

Voici les adresses IP disponibles, telles que définies dans **RFC 1918** :

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

### Les adresses suivantes sont des adresses réservées dans un sous-réseau

Dans cette liste, les adresses IP supposent que la plage CIDR du sous-réseau est 10.10.10.0/24 :

  * Première adresse dans la plage CIDR (10.10.10.0) : adresse réseau
  * Deuxième adresse dans la plage CIDR (10.10.10.1) : adresse de passerelle
  * Troisième adresse de la plage CIDR (10.10.10.2) : réservée par IBM
  * Quatrième adresse de la gamme CIDR (10.10.10.3) : réservée par IBM pour une utilisation future
  * Dernière adresse dans la plage CIDR (10.10.10.255) : adresse de diffusion réseau

### En savoir plus sur la création d'un sous-réseau

Il existe deux façons de spécifier un sous-réseau pour votre VPC : 
  * Vous pouvez créer un sous-réseau en indiquant la taille du sous-réseau dont vous avez besoin, tel que le nombre d'adresses prises en charge (par exemple, 1024).
  * Vous pouvez créer un sous-réseau en indiquant une plage CIDR (telle que 10.0.0.1/29)
  
Par exemple, pour indiquer un bloc 1024 à l'aide de CIDR, si vous spécifiez une plage CIDR plutôt qu'une taille de sous-réseau, le bloc IPv4 `192.168.100.0/22` représente les 1024 adresses IPv4 de `192.168.100.0` à `192.168.103.255`.
{:tip}

### En savoir plus sur la suppression d'un sous-réseau
  * Vous ne pouvez pas supprimer un sous-réseau si des ressources (telle qu'une instance de serveur virtuel (VSI), une adresse IP flottante, ou une passerelle publique) sont utilisées dans ce sous-réseau : les ressources doivent d'abord être supprimées. 
  * Vous NE pouvez PAS redimensionner un sous-réseau existant. Par exemple, 10.10.10.0/24 ne peut pas être redimensionné à 10.5.1.3/20
