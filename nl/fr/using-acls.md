---

copyright:
  years: 2019

lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Configuration de listes de contrôle d'accès au réseau à l'aide de l'interface de ligne de commande
{ #setting-up-network-acls-using-the-cli}

A l'aide de la fonctionnalité Liste de contrôle d'accès (ACL) disponible dans {{site.data.keyword.cloud}} Virtual Private Cloud, vous pouvez contrôler l'ensemble du trafic entrant et sortant lié aux charges de travail métier critiques dans le cloud. De même que le groupe de sécurité, une ACL est un pare-feu virtuel intégré. Contrairement aux groupes de sécurité, les règles de liste de contrôle d'accès contrôlent le trafic vers et depuis les sous-réseaux, plutôt que vers et depuis les instances. 

Le tableau suivant récapitule certaines des principales différences entre les groupes de sécurité et les ACL :

|  | Groupes de sécurité | ACL    |
|-------------|-----------------|---------|
| Niveau de contrôle  | Instance VSI    | Sous-réseau  |
| Etat   | Avec état - Le trafic de retour est autorisé par défaut | Sans état - Le trafic de retour est refusé par défaut et doit être explicitement autorisé. |
| Règles prises en charge | Utilise des règles d'autorisation uniquement | Utilise des règles d'autorisation et de refus|
| Mode d'application des règles | Toutes les règles sont prises en compte | Les règles sont traitées dans l'ordre |
| Relation avec la ressource associée | Une instance peut être associée à plusieurs groupes de sécurité| Plusieurs sous-réseaux peuvent être associés à la même ACL|

L'exemple donné dans ce document montre comment créer des ACL de réseau dans votre VPC, afin de protéger vos sous-réseaux. 

## API et interfaces de ligne de commande disponibles pour les listes de contrôle d'accès

Vous pouvez configurer et gérer les listes de contrôle d'accès via l'API ou l'interface de ligne de commande, comme vous préférez. 

* Consultez la [Référence d'API](https://{DomainName}/apidocs/rias) pour les paramètres, le corps de la demande et les détails de la réponse pour chaque API. 

* Consultez cet [exemple d'interface de ligne de commande](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) pour obtenir des détails sur les commandes, la procédure d'installation du plug-in de l'interface de ligne de commande et les étapes prérequises pour l'utilisation de l'interface de ligne de commande du VPC. 

## Utilisation des listes ACL et des règles d'ACL

Pour que vos ACL soient efficaces, créez des règles qui déterminent la gestion du trafic réseau entrant et sortant. Vous pouvez créer plusieurs règles entrantes et sortantes. Consultez la rubrique [Quotas](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-quotas) pour obtenir des informations spécifiques sur le nombre de règles que vous pouvez créer. 

* Avec les règles entrantes et sortantes, vous pouvez autoriser ou refuser le trafic à partir d'une plage d'adresses IP source ou vers une plage d'adresses IP de destination, à l'aide de protocoles et de ports spécifiés.   
* Les règles d'ACL sont traitées dans l'ordre. Les règles de faible priorité ne sont évaluées que lorsque les règles de priorité supérieure ne correspondent pas. 
* Les règles entrantes sont séparées des règles sortantes.
* Les protocoles actuellement pris en charge sont TCP, UDP et ICMP. Vous pouvez également utiliser l'option **all** pour désigner _tous les_ ou _d'autres_ protocoles (si une règle avec une priorité plus élevée est spécifiée). 

Pour obtenir des informations pertinentes sur l'utilisation des protocoles ICMP, TCP et UDP dans vos règles de liste de contrôle d'accès, voir [Présentation des protocoles de communication Internet](https://{DomainName}/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols). 

### Connexion d'une liste ACL à un sous-réseau

Vous disposez de deux options pour associer une ACL à un sous-réseau :

* Vous pouvez créer un sous-réseau et spécifier une ACL à connecter. Si vous ne spécifiez aucune ACL, une ACL réseau par défaut est connectée. L'ACL par défaut autorise tout le trafic entrant destiné à ce sous-réseau et tout le trafic sortant de ce sous-réseau.
* Vous pouvez connecter une ACL à un sous-réseau existant. Si une autre ACL est déjà connectée à ce sous-réseau, cette ACL est déconnectée avant que la nouvelle ACL ne soit connectée.

## Exemple de démonstration d'ACL

Dans l'exemple qui suit, vous créez deux ACL et les associez à deux sous-réseaux, à l'aide de l'interface de ligne de commande (CLI). Le scénario se présente comme suit :

![Exemple de scénario ACL](images/vpc-acls.png)

Comme l'illustre la figure, deux serveurs Web traitent des demandes provenant d'Internet et deux serveurs de back end sont masqués pour le public. Dans cet exemple, vous placez les serveurs dans deux sous-réseaux distincts, respectivement 10.10.10.0/24 et 10.10.20.0/24, et vous devez autoriser les serveurs Web à échanger des données avec les serveurs de back end. En outre, vous souhaitez autoriser un trafic sortant limité depuis les serveurs de back end. 

### Exemples de règle

Les exemples de règles ci-après montrent comment configurer les règles d'ACL pour un scénario de base, comme décrit précédemment.

Il est recommandé d'accorder une priorité plus élevée aux règles à granularité fine qu'aux règles à granularité grossière. Par exemple, si une règle bloque tout le trafic du sous-réseau 10.10.30.0/24 et qu'elle correspond à une priorité plus élevée, la règle à granularité fine avec une priorité inférieure pour autoriser le trafic à partir de 10.10.30.5 n'est jamais appliquée.
{:note}

**Exemples de règles ACL-1** :

| Entrant/Sortant| Autoriser/Interdire | IP source/cible | Protocole | Port | Description|
|--------------|-----------|------|------|------------------|-------|
| Entrant | Autoriser | 0.0.0.0/0 | TCP| 80 | Autoriser le trafic HTTP à partir d'Internet|
| Entrant | Autoriser | 0.0.0.0/0 | TCP | 443 | Autoriser le trafic HTTPS à partir d'Internet|
| Entrant | Autoriser| 10.10.20.0/24 |tous| tous| Autoriser tout le trafic entrant provenant du sous-réseau 10.10.20.0/24 où sont placés les serveurs de back end|
| Entrant | Interdire| 0.0.0.0/0|tous| tous| Interdire tout autre trafic entrant|
| Sortant | Autoriser | 0.0.0.0/0 | TCP|80 | Autoriser le trafic HTTP vers Internet|
| Sortant | Autoriser | 0.0.0.0/0 | TCP|443 | Autoriser le trafic HTTPS vers Internet|
| Sortant | Autoriser| 10.10.20.0/24 |tous| tous| Autoriser tout le trafic sortant vers le sous-réseau 10.10.20.0/24 où sont placés les serveurs de back end|
| Sortant | Interdire| 0.0.0.0/0|tous| tous| Interdire tout autre trafic sortant|


**Exemples de règles ACL-2** :

| Entrant/Sortant| Autoriser/Interdire | IP source/cible | Protocole| Port | Description|
|--------------|-----------|------|------|------------------|--------|
| Entrant | Autoriser| 10.10.10.0/24 |tous| tous| Autoriser tout le trafic entrant provenant du sous-réseau 10.10.10.0/24 où sont placés les serveurs web|
| Entrant | Interdire| 0.0.0.0/0|tous| tous| Interdire tout autre trafic entrant|
| Sortant | Autoriser | 0.0.0.0/0 | TCP| 80 | Autoriser le trafic HTTP vers Internet|
| Sortant | Autoriser | 0.0.0.0/0 | TCP| 443 | Autoriser le trafic HTTPS vers Internet|
| Sortant | Autoriser| 10.10.10.0/24 |tous| tous| Autoriser tout le trafic sortant vers le sous-réseau 10.10.10.0/24 où sont placés les serveurs web|
| Sortant | Interdire| 0.0.0.0/0|tous| tous| Interdire tout autre trafic sortant|

Cet exemple illustre uniquement des cas généraux. Dans vos scénarios, vous pouvez également avoir un contrôle plus granulaire du trafic :

* Un administrateur réseau peut avoir besoin d'accéder au sous-réseau 10.10.10.0/24 à partir d'un réseau distant pour des raisons d'exploitation. Dans ce cas, vous devez autoriser le trafic SSH d'Internet vers ce sous-réseau.
* Vous pouvez limiter la portée du protocole que vous autorisez entre vos deux sous-réseaux.

### Exemple d'étapes

Les exemples d'étapes suivants omettent les étapes prérequises d'utilisation de l'interface de ligne de commande pour créer un VPC, ce qui doit être fait en premier lieu. Pour plus d'informations, voir [Création d'un VPC à l'aide de l'interface de ligne de commande](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli). 


#### Etape 1. Création des ACL

Voici les commandes de l'interface de ligne de commande que vous pouvez utiliser pour créer deux ACL appelées `my_web_subnet_acl` et `my_backend_subnet_acl` : 

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

La réponse inclut les ID des ACL récemment créées. Sauvegardez les ID des deux ACL à utiliser dans des commandes ultérieures. Vous pouvez utiliser des variables appelées `webacl` et `bkacl`, comme suit : 

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### Etape 2. Extraction des règles de liste de contrôle d'accès par défaut

Avant d'ajouter de nouvelles règles, extrayez les règles d'ACL entrantes et sortantes par défaut afin de pouvoir insérer les nouvelles règles avant.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

La réponse affiche les règles entrantes et sortantes par défaut qui autorisent tout le trafic IPv4 dans tous les protocoles.

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

Sauvegardez les ID des deux règles d'ACL sous forme de variables, car vous les utiliserez dans des commandes ultérieures. Par exemple, vous pouvez utiliser des variables appelées `inrule` et `outrule` : 

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### Etape 3. Ajout de nouvelles règles d'ACL conformément à la description

Cette section montre d'abord l'ajout des règles entrantes, puis celui des règles sortantes. 

Insérez les nouvelles règles entrantes avant la règle entrante par défaut.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

A chaque étape, sauvegardez l'ID de la règle d'ACL dans une variable, car elle sera utilisée dans des commandes ultérieures. Par exemple, vous pouvez utiliser le nom de variable `acl200` : 

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

Ajoutez maintenant la règle à `acl200` : 

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

Ajoutez des règles supplémentaires jusqu'à ce que la configuration de votre liste de contrôle d'accès soit terminée, en sauvegardant chaque ID sous forme de variable. 

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

Insérez les nouvelles règles sortantes avant la règle sortante par défaut.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### Etape 4. Créez les deux sous-réseaux avec la liste ACL créée.

Vous allez créer deux sous-réseaux afin que chacune de vos listes de contrôle d'accès soit associée à l'un d'eux.  

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## Aide-mémoire sur la liste de commandes

Pour afficher la liste complète des commandes d'interface de ligne de commande du VPC disponibles pour les ACL, entrez : 

```
ibmcloud is network-acls
```
{: pre}

Pour afficher votre ACL et ses métadonnées, y compris les règles, entrez : 

```
ibmcloud is network-acl $webacl
```
{: pre}

Pour obtenir la règle d'ACL entrante par défaut, saisissez : 

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## Exemple de règle `ping` entrante

Pour ajouter une règle d'ACL, voici un exemple de commande permettant d'ajouter une règle entrante `ping` avant la règle entrante par défaut :

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
