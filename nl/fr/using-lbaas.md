---



copyright:
  years: 2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Bêta) Utilisation des équilibreurs de charge dans IBM Cloud VPC
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

Le service d'équilibreur de charge répartit le trafic entre plusieurs instances de serveur dans la même région pour votre VPC {{site.data.keyword.cloud}}. 

**Ce service existe maintenant en version bêta. Les ressources utilisées pendant la période bêta ne seront plus disponibles après la clôture de la version bêta et aucun support ne sera disponible. Veuillez adresser vos commentaires et questions à votre ingénieur commercial IBM Cloud.**

## Equilibreur de charge public
{: #public-load-balancer}

Votre instance de service d'équilibreur de charge se voit attribuer un nom de domaine complet (FQDN) accessible publiquement, que vous devez utiliser pour accéder à vos applications hébergées derrière le service d'équilibreur de charge dans {{site.data.keyword.cloud}} VPC. Vous pouvez enregistrer ce nom de domaine avec une ou plusieurs adresses IP publiques.

Au fil du temps, ces adresses IP publiques et le nombre d'adresses IP publiques peuvent changer en fonction des activités de maintenance et de mise à l'échelle, qui sont transparentes pour les clients. Les instances de serveur de back end (VSI) hébergeant votre application doivent s'exécuter dans la même région et sous le même VPC.

## Programmes d'écoute frontaux et pools de back end
Vous pouvez définir jusqu'à 10 programmes d'écoute frontaux (ports d'application) et les mapper aux pools de back end respectifs sur les serveurs d'applications de back end. Le nom de domaine complet (FQDN) attribué à votre instance de service d'équilibreur de charge et les ports d'écoute frontaux sont exposés à l'Internet public. Les demandes d'utilisateurs entrantes sont reçues sur ces ports.

Les protocoles d'écoute frontaux pris en charge sont les suivants :
* HTTP
* HTTPS
* TCP

Les protocoles de pool de back end pris en charge sont les suivants :
* HTTP
* TCP

Le trafic HTTPS entrant se termine au niveau de l'équilibreur de charge pour permettre une communication HTTP en texte brut avec le serveur de back end. 

Vous pouvez associer jusqu'à 50 instances de serveur à un pool de back end. Chaque instance de serveur connecté doit avoir un port configuré. Le port peut être ou ne pas être identique au port d'écoute frontal. 

La plage de ports de 56500 à 56520 est réservée à des fins de gestion ; ces ports ne peuvent pas être utilisés comme ports d'écoute frontaux.
{: note}

## Méthodes d'équilibrage de charge
{: #load-balancing-methods}

Les trois méthodes d'équilibrage de charge ci-dessous peuvent être utilisées pour répartir le trafic entre les serveurs d'applications de back end :

* **Circulaire :** Il s'agit de la méthode d'équilibrage de charge par défaut. Avec cette méthode, l'équilibreur de charge achemine les connexions client entrantes de façon circulaire vers les serveurs de back end. Chaque serveur de back end reçoit ainsi un nombre équivalent de connexions client.

* **Circulaire pondérée :** Avec cette méthode, l'équilibreur de charge achemine les connexions client entrantes vers les serveurs de back end au prorata du nombre de connexions affectées à ces serveurs. Chaque serveur reçoit une pondération par défaut de 50 connexions, qui peut être personnalisée sur n'importe quelle valeur comprise entre 0 et 100.

Par exemple, si trois serveurs d'applications A, B et C ont des pondérations personnalisées à 60, 60 et 30 respectivement, les serveurs A et B reçoivent un nombre égal de connexions, tandis que le serveur C reçoit la moitié de ce nombre de connexions. 

* **Connexions minimum :** Avec cette méthode, l'instance de serveur qui prend en charge le plus petit nombre de connexions, à un moment donné, reçoit la connexion suivante.

**Caractéristiques supplémentaires de ces méthodes :**

* Si vous réinitialisez le poids d'un serveur sur '0', cela signifie qu'aucune nouvelle connexion n'est transmise à ce serveur, mais que tout le trafic existant continue de circuler tant qu'il est actif. Utiliser un poids égal à '0' peut aider à mettre un serveur hors service sans heurt et à le retirer de la rotation du service.
* Les valeurs de pondération des serveurs ne sont applicables qu'avec la méthode circulaire pondérée. Elles sont ignorées avec les méthodes d'équilibrage de charge Circulaire et Connexions minimum. 

## Diagnostics d'intégrité
{: #health-checks}

Les définitions de diagnostic d'intégrité sont obligatoires pour les pools de back end.

L'équilibreur de charge effectue des diagnostics d'intégrité périodiques pour surveiller l'intégrité des ports de back end et il leur transmet le trafic client en conséquence. Si un port de serveur de back end donné est jugé défaillant, aucune nouvelle connexion ne lui est transmise. L'équilibreur de charge continue de surveiller l'intégrité des ports défectueux et reprend leur utilisation s'ils redeviennent sains, à savoir s'ils ont réussi deux tentatives de diagnostic d'intégrité consécutives.

Les diagnostics d'intégrité des ports HTTP et TCP sont effectués comme suit :

* **HTTP :** Une demande `HTTP GET` fondée sur une URL prédéfinie est envoyée au port du serveur de back end. Le port est marqué comme intègre lorsqu'il reçoit une réponse `200 OK`. Le chemin d'intégrité `GET` par défaut est "/" via l'interface utilisateur et peut être personnalisé.

* **TCP :** L'équilibreur de charge tente d'établir une connexion TCP au serveur de back end sur un port TCP spécifique. Le port du serveur est marqué comme intègre si la tentative de connexion aboutit, puis la connexion est fermée.

L'intervalle de diagnostic d'intégrité par défaut est de 5 secondes, le délai d'attente par défaut pour une demande de diagnostic d'intégrité est de 2 secondes et le nombre de nouvelles tentatives par défaut est de 2.
{: note}

## Déchargement SSL et autorisations requises

Pour toutes les connexions HTTPS entrantes, le service d'équilibreur de charge met fin à la connexion SSL et établit une communication HTTP en texte brut avec l'instance du serveur back end. Avec cette technique, les établissements de liaison SSL et les tâches de chiffrement ou de déchiffrement,consommateurs de ressources processeur, sont déplacés des instances de serveur back end, leur permettant ainsi d'utiliser tous leurs cycles de processeurs pour traiter le trafic des applications.

Un certificat SSL est nécessaire pour que l'équilibreur de charge procède aux tâches de déchargement SSL. Vous pouvez gérer les certificats SSL via [IBM Certificate Manager![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}. 

Si vous avez besoin que l'équilibreur de charge accède au certificat SSL dans votre instance du gestionnaire de certificat, créez une autorisation permettant à l'instance du service de l'équilibreur de charge d'accéder à l'instance du gestionnaire de certificat contenant le certificat SSL. Vous pouvez gérer ce type d'autorisation via [Identity and Access Authorizations ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](/docs/services/iam/#/authorizations){: new_window}. Veillez à sélectionner **Services d'infrastructure** en tant que service source, **Equilibreur de charge pour VPC** en tant que type de ressource, **Certificate Manager** en tant que service cible et attribuez le Rôle Accès au service **Auteur**. Pour créer un équilibreur de charge, vous devez accorder l'autorisation **Toutes les instances de ressource** sur l'instance de ressource source. L'instance de service cible peut être **Toutes les instances**, ou il peut s'agir de votre instance de ressource spécifique du gestionnaire de certificat. 

Si l'autorisation requise est supprimée, des erreurs peuvent survenir sur l'équilibreur de charge.
{: note}

## Identity and Access Management (IAM)

Vous pouvez configurer des règles d'accès pour une instance **Equilibreur de charge pour VPC**. Pour gérer vos règles d'accès utilisateur, rendez-vous sur [Identity and Access Authorizations ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](/docs/services/iam/#/authorizations){: new_window}. 

### Configuration des règles d'accès aux groupes de ressources pour les utilisateurs

Pour créer un équilibreur de charge, vous devez avoir accès à un groupe de ressources. L'utilisateur qui crée un équilibreur de charge doit avoir un accès approprié au groupe de ressources fourni ou au groupe de ressources par défaut si aucun n'est fourni.

1. Accédez à **Gérer > Compte > Utilisateurs**. La liste des utilisateurs ayant accès à votre compte IBM Cloud apparaît.
2. Sélectionnez le nom de l'utilisateur auquel vous souhaitez attribuer une règle d'accès. Si l'utilisateur n'est pas affiché, cliquez sur **Inviter des utilisateurs** pour ajouter l'utilisateur à votre compte IBM Cloud.
3. Sélectionnez **Affecter un accès**.
4. Sélectionnez **Affecter l'accès au sein d'un groupe de ressources**.
5. Dans la liste déroulante **Groupe de ressources**, sélectionnez le groupe de ressources souhaité.
6. Dans la liste déroulante **Affecter l'accès à un groupe de ressources**, sélectionnez l'accès souhaité.
7. Dans la liste déroulante **Services**, sélectionnez les services souhaités.
9. Sélectionnez **Affecter** pour affecter la règle d'accès au groupe de ressources à l'utilisateur.

### Configuration des règles d'accès aux ressources pour les utilisateurs

| Rôle d'accès à la plateforme | Action de l'équilibreur de charge |
|-------------|-----|
| Administrateur | Créer/Afficher/Modifier/Supprimer un équilibreur de charge |
| Editeur | Créer/Afficher/Modifier/Supprimer un équilibreur de charge |
| Afficheur | Afficher l'équilibreur de charge |

1. Accédez à **Gérer > Compte > Utilisateurs**. La liste des utilisateurs ayant accès à votre compte IBM Cloud apparaît.
2. Sélectionnez le nom de l'utilisateur auquel vous souhaitez attribuer une règle d'accès. Si l'utilisateur n'est pas affiché, cliquez sur **Inviter des utilisateurs** pour ajouter l'utilisateur à votre compte IBM Cloud.
3. Sélectionnez **Affecter un accès**.
4. Sélectionnez **Affecter l'accès aux ressources**.
5. Dans la liste déroulante **Services**, sélectionnez **Service d'infrastructure**.
6. Dans la liste déroulante **Type de ressource**, sélectionnez **Equilibreur de charge pour VPC**.
7. Dans la liste déroulante **ID d'équilibreur de charge**, sélectionnez un ID d'instance d'équilibreur de charge ou utilisez la valeur par défaut, **Tous les équilibreurs de charge**.
8. Attribuez un rôle d'accès à la plateforme à l'utilisateur.
9. Sélectionnez **Affecter** pour affecter la règle d'accès à l'utilisateur.

## API disponibles

Pour effectuer des appels d'API, vous devez utiliser une forme de client REST. Par exemple, vous pouvez utiliser la commande `curl` pour récupérer tous les équilibreurs de charge existants :

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

La section suivante fournit des détails sur les API que vous pouvez utiliser pour les équilibreurs de charge dans votre environnement VPC. Pour la spécification complète, consultez la [Référence d'API RIAS](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers). 

| Description | API |
|-------------|-----|
| Crée et met à disposition un équilibreur de charge | POST /load_balancers |
| Extrait tous les équilibreurs de charge | GET /load_balancers |
| Extrait un équilibreur de charge | GET /load_balancers/{id} |
| Supprime un équilibreur de charge | DELETE /load_balancers/{id} |
| Met à jour un équilibreur de charge | PATCH /load_balancers/{id} |
| Crée un programme d'écoute | POST /load_balancers/{id}/listeners |
| Extrait tous les programmes d'écoute de l'équilibreur de charge | GET /load_balancers/{id}/listeners |
| Extrait un programme d'écoute | GET /load_balancers/{id}/listeners/{listener_id} |
| Supprime un programme d'écoute | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Met à jour un programme d'écoute | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Crée un pool | POST /load_balancers/{id}/pools |
| Extrait tous les pools de l'équilibreur de charge | GET /load_balancers/{id}/pools |
| Extrait un pool | GET /load_balancers/{id}/pools/{pool_id} |
| Supprime un pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Met à jour un pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Crée un membre | POST /load_balancers/{id}/pools/{pool_id}/members |
| Extrait tous les membres du pool | GET /load_balancers/{id}/pools/{pool_id}/members |
| Extrait un membre |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Supprime un membre du pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Crée un membre | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Met à jour des membres du pool | PUT /load_balancers/{id}/pools/{pool_id}/members |

## Exemple d'équilibreur de charge

Dans l'exemple suivant, vous utilisez l'API pour créer un équilibreur de charge devant 2 instances de serveur VPC (`192.168.100.5` et `192.168.100.6`) exécutant une application Web à l'écoute sur le port `80`. L'équilibreur de charge dispose d'un programme d'écoute frontal qui permet d'accéder à l'application Web en toute sécurité au moyen de HTTPS. Vous pouvez ensuite utiliser l'API pour obtenir des détails sur l'instance de l'équilibreur de charge après sa création et pour supprimer l'instance de l'équilibreur de charge.

### Exemple d'étapes

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'[interface utilisateur IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), de l'[interface de ligne de commande](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) ou de l'[API RIAS](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) pour mettre à disposition un VPC, des sous-réseaux et des instances.  


Les étapes de l'exemple d'équilibreur de charge peuvent également être exécutées à l'aide de l'[interface de ligne de commande](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference).
{: note}

#### Etape 1. Création d'un équilibreur de charge avec des instances connectées de programme d'écoute, de pool et de serveur (membres)

```bash
curl -H "Authorization: $iam_token" -X POST
$rias_endpoint/v1/load_balancers?version=2019-01-01 \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Exemple de sortie :
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d-7ce5-413b-aae4-90a2a88b7872.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

Sauvegardez l'ID de l'équilibreur de charge à utiliser au cours des étapes suivantes, par exemple, dans la variable `lbid`. 

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### Etape 2. Obtention d'un équilibreur de charge

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

Prévoyez du temps pour la mise à disposition de l'équilibreur de charge. Une fois prêt, il prendra le statut `inline` et `active`, comme dans l'exemple de sortie suivant : 

Exemple de sortie :

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

#### Etape 3. Suppression d'un équilibreur de charge

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## FAQ (Foires aux questions)

Cette section contient des réponses aux questions fréquemment posées sur le service **Equilibreur de charge pour VPC**. 

### Puis-je utiliser un nom DNS différent pour mon équilibreur de charge ?

Le nom DNS affecté automatiquement à l'équilibreur de charge n'est pas personnalisable. Toutefois, vous avez la possibilité d'ajouter un enregistrement CNAME (nom canonique) qui fasse pointer le nom DNS de votre choix vers le nom DNS affecté automatiquement. Par exemple, votre ID d'équilibreur de charge est `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, le nom DNS de l'équilibreur de charge attribué automatiquement est `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud` et vous souhaitez utiliser le nom DNS `www.myapp.com`. Vous pouvez ajouter un enregistrement CNAME (via le fournisseur DNS que vous utilisez pour gérer `myapp.com`) faisant pointer `www.myapp.com` sur le nom DNS de l'équilibreur de charge `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`.

### Quel est le nombre maximal de programmes d'écoute frontaux que je peux définir avec mon équilibreur de charge ?

10.

### Quel est le nombre maximal d'instances de serveur que je peux connecter à mon pool de back end ?

50.

### Puis-je créer un équilibreur de charge privé interne, accessible uniquement par des clients internes ?  

Non, pas pour l'instant.

### L'équilibreur de charge est-il évolutif horizontalement ?  

Non, pas pour l'instant.

### Que dois-je faire si j'utilise des ACL ou des groupes de sécurité sur les sous-réseaux utilisés pour déployer l'équilibreur de charge ?

Vous devez vous assurer que les règles d'ACL ou de groupe de sécurité appropriées sont en place pour autoriser le trafic entrant pour les ports d'écoute configurés et les ports de gestion (ports de 56500 à 56520). Le trafic entre l'équilibreur de charge et les instances de back end doit également être autorisé.

### Pourquoi est-ce que je reçois un message d'erreur : `certificate instance not found` ?

* Le CRN (nom de ressource de cloud) de l'instance de certificat peut être non valide.
* Vous n'avez peut-être pas accordé l'autorisation de service à service. Voir la section **Déchargement SSL** du présent document.

### Pourquoi est-ce que je reçois un code `401 Unauthorized Error` ?

Vérifiez les règles d'accès suivantes pour votre utilisateur :
* Règle d'accès au type de ressource de l'équilibreur de charge
* Règle d'accès au groupe de ressources

### Pourquoi mon équilibreur de charge est-il à l'état `maintenance_pending` ? 

L'équilibreur de charge est à l'état `maintenance_pending` au cours de diverses activités de maintenance telles que :
* La mise à jour en continu pour remédier aux vulnérabilités et appliquer des correctifs de sécurité 
* Les activités de reprise

### Pourquoi dois-je choisir plusieurs sous-réseaux lors de la mise à disposition ?

Les dispositifs d'équilibreur de charge sont déployés sur les sous-réseaux que vous avez sélectionnés. Il est vivement recommandé de choisir des sous-réseaux de différentes zones pour une disponibilité et une redondance plus élevées.
