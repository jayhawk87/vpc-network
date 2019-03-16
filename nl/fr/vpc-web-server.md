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

# Configuration de la connectivité entre serveurs Web dans votre VPC
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

Ce document vous fournit des exemples de procédure de configuration de la communication entre deux serveurs Web dans votre VPC {{site.data.keyword.cloud}}, à l'aide de Python. Il vous montre comment créer une communication entre serveurs dans la même zone et dans des zones différentes. 

## Prérequis

Pour pouvoir configurer la communication entre des serveurs Web dans votre VPC, vous devez avoir déjà créé un VPC avec DEUX sous-réseaux et au moins DEUX instances disponibles dans chaque sous-réseau. Vous devrez connaître les ID du VPC, des sous-réseaux et des instances. Suivez les étapes de notre [guide
pour la création de ressources VPC à l'aide de l'interface de ligne de commande](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) si vous avez besoin d'aide pour cette configuration. 

## Scénario 1 : Connectivité entre des serveurs dans la même zone mais dans des sous-réseaux différents

`vpc` holds vpc ID
`subnet1` holds subnet 1 ID 
`subnet2` holds subnet 2 ID


## Partie 1 : Connectivité entre des serveurs de la même zone et du même VPC

Après la création de 2 instances au minimum dans le même sous-réseau, auquel est connectée une passerelle publique, nous pouvons tester leur connectivité en hébergeant un serveur avec du code HTML dans une instance, puis lancer `curl` sur ce serveur et télécharger le fichier HTML sur les autres ou à partir du navigateur de votre ordinateur. 

Supposons qu'`instance_1` et `instance_2` se trouvent dans le même sous-réseau comportant une passerelle publique et le même groupe de sécurité. Vous devez ajouter des règles pour l'instance à l'aide de l'API, de l'interface de ligne de commande ou de l'interface utilisateur dans des groupes de sécurité afin de permettre un accès entrant. Si vous hébergez votre serveur sur `instance_1`, vous devez ajouter une règle pour l'adresse IP flottante d'`instance_2`. Par exemple, supposons qu'IPv4 soit défini sur `169.61.161.161` pour l'`instance_1` hébergeant le serveur et sur `169.61.161.235` pour l'`instance_2` et qu'il tente d'exécuter la commande GET sur le fichier à partir d'`instance_1`, la présentation est la suivante : 

![Nouvelles règles de groupe de sécurité](images/security-group-ui-ex1.png)

* Pour répertorier tous les groupes de sécurité `ibmcloud is sgs`
* Voici une demande d'API qui fonctionne pour obtenir le résultat précédent : 

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. Créez un fichier `index.html` de base sur `instance_1`. Voici un exemple de fichier HTML de base : 

```
<!DOCTYPE html>
<html>
<body>

<h1>You have successfully connected to the instance</h1>
<p>Hello, world!</p>

</body>
</html>
```
{:pre}

2. Dans le répertoire où se trouve le fichier, exécutez la commande `python -m SimpleHTTPServer <port number>`. Vous pouvez utiliser un autre port si le port par défaut 8000 est déjà en cours d'utilisation. 

3. Exécutez SSH sur `instance_2`. 

Pour rechercher une adresse IP flottante, utilisez la commande `ibmcloud is ips` et identifiez le paramètre `floating_ip_id` à l'aide de la commande `ibmcloud is ip
$floating_ip_id`. Vous devez être en mesure d'identifier l'adresse publique. 

4. Identifiez l'adresse IP d'`instance_1`. 

5. Utilisez `curl -X GET http://adresse_IP:numéro_port/index.html` pour demander le téléchargement du fichier `index.html` depuis l'`instance_1` du serveur d'hébergement. (Exemple : `curl -X GET http://169.61.161.161:8000/index.html`)

6. Vous devez être en mesure de voir la sortie du fichier `index.html`. 

7. (Pour accéder au serveur par le biais du navigateur de votre ordinateur local) Ajoutez des règles pour tout le trafic entrant, comme illustré dans la figure suivante :

![Nouvelles règles de groupe de sécurité avec tout le trafic entrant](images/security-group-ui-ex2.png)

* Pour ajouter des règles autorisant TOUS les protocoles et depuis chaque source à vos groupes de sécurité

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**Exemple de demande plus détaillé :**

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "port_max": 22,
  "port_min": 22,
  "protocol": "tcp",
  "remote": {
    "cidr_block": "131.121.0.0/24"
  }
}'
```
{:pre}

8. Ouvrez un navigateur Web, puis saisissez `http://adresse_IP:numéro_port/index.html`

9. Vous devez voir le fichier index affiché au format HTML. 

## Partie 2 : Connectivité entre des serveurs d'une zone et d'un VPC différents

Connectez l'instance de l'autre sous-réseau, sur lequel la passerelle publique n'est pas configurée, à l'instance ayant accès à Internet. Vous souhaitez configurer 2 types d'instances différents : 

* un serveur Web qui traite les demandes provenant d'Internet à l'aide d'une passerelle publique
* un serveur de back end qui exécute une application et stocke les données importantes, en envoyant uniquement le trafic interne – la passerelle publique est désactivée

Les groupes de sécurité s'appliquent aux instances de serveur virtuel (VSI), tandis que les listes de contrôle d'accès réseau s'appliquent aux sous-réseaux. 

Par conséquent, pour effectuer cette tâche globale, vous devez créer des VSI, des sous-réseaux, des règles de groupe de sécurité et de liste de contrôle d'accès, puis connecter le groupe de sécurité à l'interface réseau d'une instance et connecter la liste de contrôle d'accès réseau à un sous-réseau spécifique. 

* Pour répertorier toutes les règles du groupe de sécurité `subnet1`

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. Supprimez la règle `inbound-allow-all-traffic` dans le groupe de sécurité du sous-réseau créée dans la partie 1 de l'exercice. 

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. Créez un VPC, puis créez un sous-réseau sans connecter la passerelle publique. 

3. Créez une `instance_3` dans le nouveau sous-réseau sans passerelle publique (aucune connexion à Internet) - Vous voudrez peut-être affecter davantage de coeurs d'UC et de mémoire à l'instance pour des raisons de back end. 

4. Ajoutez de nouvelles règles pour `instance_3` dans le groupe de sécurité de la partie 1. Exemple : `instance_3` a l'adresse IP flottante `169.61.181.25`

![Nouvelles règles de groupe de sécurité avec une instance provenant d'un autre sous-réseau entrant](images/security-group-ui-ex3.png)

5. Utilisez `curl -X GET http://adresse_IP:numéro_port/index.html` pour demander le téléchargement du fichier `index.html` depuis l'instance_1 du serveur d'hébergement. (Exemple : `curl -X GET http://169.61.161.161:8000/index.html`)

6. Vous devez être en mesure de voir la sortie du fichier `index.html`. 

Cela signifie que vous disposez d'une connexion entre les instances 1 et 3, mais pas 2. 

## Etapes suivantes

Vous souhaitez protéger vos serveurs en créant des règles de liste de contrôle d'accès qui contrôlent le trafic entrant, comme indiqué [dans notre guide d'utilisation des ACL dans un VPC](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli). 
