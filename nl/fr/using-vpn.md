---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Bêta) Utilisation d'un VPN avec votre VPC
{: #--beta-using-vpn-with-your-vpc}

Le service VPN d'{{site.data.keyword.cloud}} VPC vous permet de connecter des réseaux privés en toute sécurité. Vous pouvez utiliser un VPN pour configurer un tunnel IPsec site à site entre votre VPC et votre réseau privé local ou un autre VPC.

Pour la version actuelle d'{{site.data.keyword.cloud}} VPC, seul le routage basé sur des règles est pris en charge.

**Ce service existe maintenant en version bêta. Les ressources utilisées pendant la période bêta ne seront plus disponibles après la clôture de la version bêta et aucun support ne sera disponible. Veuillez adresser vos commentaires et questions à votre ingénieur commercial IBM Cloud.**

## Fonctions

* IKEv1 et IKEv2
* Algorithmes d'authentification : `md5`, `sha1`, `sha256`
* Algorithmes de chiffrement : `3des`, `aes128`, `aes256`
* Groupes Diffie-Hellman (DH) : 2, 5, 14
* Mode de négociation IKE : main
* Protocole de transformation IPSec : ESP
* Mode d'encapsulation IPSec : tunnel
* Perfect Forward Secrecy (PFS)
* Fonction DPD (Dead Peer Detection)
* Routage: basé sur des règles
* Mode d'authentification : clé pré-partagée
* Support haute disponibilité

## API disponibles

La section suivante fournit des détails sur les API que vous pouvez utiliser pour un VPN dans votre environnement IBM Cloud VPC. Veuillez consulter la page [API REST VPC](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies) pour plus de détails.

### Passerelles VPN et connexions VPN 

| Description | API |
|----------------------------|-------------|
| Crée une passerelle VPN | POST /vpn_gateways |
| Extrait des passerelles VPN | GET /vpn_gateways |
| Extrait une passerelle VPN | GET /vpn_gateways/{id} |
| Supprime une passerelle VPN | DELETE /vpn_gateways/{id} |
| Met à jour une passerelle VPN | PATCH /vpn_gateways/{id} |
| Crée une connexion VPN | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Extrait des connexions VPN | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Extrait une connexion VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Supprime une connexion VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Met à jour une connexion VPN | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Extrait tous les routages CIDR locaux pour une connexion VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Supprime un routage CIDR local d'une connexion VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Vérifie si un routage CIDR local spécifique existe sur une connexion VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Définit un routage CIDR local sur une connexion VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Extrait tous les routages CIDR homologues pour une connexion VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Supprime un routage CIDR homologue d'une connexion VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Vérifie si un routage CIDR homologue spécifique existe sur une connexion VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Définit un routage CIDR homologue sur une connexion VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### Stratégies IKE

| Description | API |
|-----------------------------|--------------|
| Extrait toutes les stratégies IKE | GET /ike_policies |
| Crée une stratégie IKE | POST /ike_policies |
| Supprime une stratégie IKE | DELETE /ike_policies/{id} |
| Extrait une stratégie IKE | GET /ike_policies/{id} |
| Met à jour une stratégie IKE | PATCH /ike_policies/{id} |
| Extrait toutes les connexions qui utilisent la stratégie IKE spécifiée | GET /ike_policies/{id}/connections |

### Stratégies IPsec

| Description | API |
|---------------------|-------------|
| Extrait toutes les stratégies IPSec | GET /ipsec_policies |
| Crée une stratégie IPSec | POST /ipsec_policies |
| Supprime une stratégie IPSec | DELETE /ipsec_policies/{id} |
| Extrait une stratégie IPSec | GET /ipsec_policies/{id} |
| Met à jour une stratégie IPSec | PATCH /ipsec_policies/{id} |
| Extrait toutes les connexions qui utilisent la stratégie IPsec spécifiée | GET /ipsec_policies/{id}/connections |

## Exemple de réseau privé virtuel

Dans l'exemple suivant, vous pouvez connecter deux VPC à l'aide d'un VPN, ce qui signifie que vous pouvez connecter des sous-réseaux dans deux VPC distincts comme s'il s'agissait d'un seul réseau. Les adresses IP des sous-réseaux ne doivent pas se chevaucher.
Le scénario (avec quelques machines virtuelles ajoutées dans chaque VPC) se présente comme suit :
![Exemple de scénario VPN](images/vpc-vpn.png)

### Exemple d'étapes

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'API IBM Cloud ou de la CLI pour créer des VPC. Pour plus d'informations, voir [Initiation](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) et [Configuration de VPC avec des API](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

Vous pouvez éventuellement créer une passerelle VPN à l'aide de l'interface utilisateur. Vous trouverez des étapes dans le [document Console Tutorial](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

#### Etape 1. Création d'une passerelle VPN dans votre sous-réseau VPC

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Exemple de sortie :
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Assurez-vous que les zones suivantes sont enregistrées pour les étapes suivantes.
* `id`. Il s'agit de l'ID de passerelle VPN qui sera désigné par `$gwid1`. 
* `address`. Il s'agit de l'adresse IP publique de la passerelle VPN qui sera désignée par `$gwaddress1`. 

REMARQUE : Le statut de la passerelle apparaît en tant qu'`en attente` lors de la création de la passerelle VPN et devient `disponible` une fois la création terminée. La création peut prendre du temps. Vous pouvez vérifier son statut à l'aide de la commande suivante :

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### Etape 2. Création d'une seconde passerelle VPN sur un autre VPC

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Exemple de sortie :
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Veillez à enregistrer les zones suivantes pour les étapes suivantes.
* `id`. Il s'agit de l'ID de passerelle VPN qui sera désigné par `$gwid2`. 
* `address`. Il s'agit de l'adresse IP publique de la passerelle VPN qui sera désignée par `$gwaddress2`. 


#### Etape 3. Création d'une connexion VPN de la première passerelle VPN à la seconde passerelle VPN, en définissant `local_cidrs` sur le sous-réseau de VPC 1 et `peer_cidrs` sur le sous-réseau de VPC 2

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01 \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Exemple de sortie :
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Etape 4. Création d'une connexion VPN de la deuxième passerelle VPN à la première passerelle VPN, en définissant `local_cidrs` sur le sous-réseau de VPC 2 et `peer_cidrs` sur le sous-réseau de VPC 1

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-01-01 \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Exemple de sortie :
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Etape 5. Vérification de la connectivité

Une fois la connexion VPN établie, vous pourrez accéder à vos instances sur le sous-réseau 2 à partir du sous-réseau 1, et inversement.

Vous pouvez vérifier le statut de la connexion VPN comme suit : 
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

Exemple de sortie :
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## Quotas

Consultez notre rubrique [Quotas VPC](/docs/infrastructure/vpc?topic=vpc-quotas#vpn-quotas) pour les quotas VPN. 

## Négociation automatique de stratégies

L'utilisation des stratégies IKE et IPSec pour configurer une connexion VPN est facultative. Lorsqu'aucune stratégie n'est sélectionnée, les propositions par défaut sont choisies automatiquement pour un processus appelé _négociation automatique_. Les propositions utilisées dans le cadre de la négociation automatique sont les suivantes.

En tant qu'initiateur, IBM Cloud utilise **IKEv2**. En tant que répondeur, IBM Cloud accepte **IKEv1** et **IKEv2**.
{: note}

### Négociation automatique IKE (Phase 1)

Les options de chiffrement, d'authentification et de groupe DH suivantes peuvent être utilisées dans n'importe quelle combinaison :

|    | Chiffrement | Authentification | Groupe DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Négociation automatique IPsec (Phase 2)

Les options de chiffrement et d'authentification suivantes peuvent être utilisées dans n'importe quelle combinaison :

|    | Chiffrement | Authentification | Groupe DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | désactivé  |
| 2  | aes256 | sha256 | désactivé  |
| 3  | 3des   | md5    | désactivé  |

## Foire aux questions

**Lorsque je crée une passerelle VPN, puis-je créer des connexions VPN simultanément ?**

Si vous utilisez l'API ou la CLI, des connexions VPN doivent être créées après la création de la passerelle VPN. Dans la console IBM Cloud, vous pouvez créer la passerelle et une connexion simultanément.

**Si je supprime une passerelle VPN comportant des connexions VPN, qu'adviendra-t-il des connexions ?**

Les connexions VPN sont supprimées avec la passerelle VPN.

**Les stratégies IKE ou IPSec seront-elles supprimées si je supprime une passerelle VPN ou une connexion VPN ?**

Non, les stratégies IKE et IPSec peuvent s'appliquer à plusieurs connexions.

**Qu'advient-il d'une passerelle VPN si j'essaie de supprimer le sous-réseau sur lequel se trouve la passerelle ?**

Le sous-réseau ne peut pas être supprimé si des instances sont présentes, y compris la passerelle VPN.

**Existe-t-il des stratégies IKE et IPSec par défaut ?**

Lorsque vous créez une connexion VPN sans référencer un ID de stratégie (IKE ou IPSec), la négociation automatique est utilisée.

**Est-ce que le VPN for VPC prend en charge les configurations à haute disponibilité ?**

Oui.
