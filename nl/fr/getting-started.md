---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-11"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Initiation à la mise en réseau pour le cloud privé virtuel
{: #getting-started-with-networking-for-virtual-private-cloud}


IBM acceptera un nombre limité de clients pour un programme d'accès en avant-première à VPC qui commencera début avril 2019 ; l'accès élargi ne sera ouvert que dans les mois qui suivront. Si votre organisation souhaite avoir accès à IBM Virtual Private Cloud, veuillez remplir ce [formulaire de nomination](https://cloud.ibm.com/vpc){: new_window} et un interlocuteur IBM entrera en contact avec vous concernant la procédure à suivre.
{: important}

Pour vous initier à la mise en réseau {{site.data.keyword.cloud}} pour un cloud privé virtuel

1. Créez un cloud privé virtuel.
2. Créez un ou plusieurs sous-réseaux dans le cloud privé virtuel dans une ou plusieurs zones.
3. Créez une passerelle publique (PGW) sur un sous-réseau si vous souhaitez que les ressources de votre sous-réseau aient accès à Internet ou inversement.

## Prérequis

 * **Droits utilisateur** : Assurez-vous que votre utilisateur dispose des autorisations suffisantes pour créer et gérer des ressources dans votre VPC. Pour obtenir la liste des droits requis, voir [Granting permissions needed for VPC users](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).

## Utilisation de l'interface utilisateur, de l'interface de ligne de commande ou de l'API REST pour la mise à disposition des ressources réseau du VPC

La mise à disposition et la gestion des ressources réseau du VPC peut s'effectuer via l'interface utilisateur, l'interface de ligne de commande ou l'API REST. 

* Pour un accès via l'interface utilisateur, connectez-vous à la [console IBM Cloud ![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")]( https://{DomainName}/vpc){: new_window}. Suivez le [guide de l'interface utilisateur](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console) si vous avez besoin d'aide. 
* Pour utiliser l'interface de ligne de commande, servez-vous du plug-in [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) de l'[interface de ligne de commande IBM Cloud](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) et suivez l'exemple [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli). 
* Les utilisateurs plus expérimentés peuvent appeler directement les [API REST](https://{DomainName}/apidocs/rias). Suivez le tutoriel [Exemple de code](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) pour vous initier aux API REST. 

## Etapes suivantes

Une fois votre mise en réseau de cloud privé virtuel à disposition, explorez d'autres fonctions. 

* [Création et gestion d'instances de serveur virtuel](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [Utilisation de groupes de sécurité](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Utilisation de listes de contrôle d'accès au réseau](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(Bêta) Utilisation d'un VPN](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(Bêta) Utilisation d'équilibreurs de charge](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
