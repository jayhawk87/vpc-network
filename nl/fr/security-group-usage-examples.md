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
{:download: .download}

# Configuration de groupes de sécurité à l'aide des API
{: #setting-up-security-groups-using-the-apis}

L'exemple suivant montre comment créer et gérer des groupes de sécurité à l'aide des API régionales {{site.data.keyword.cloud}} (RIAS). 

## Prérequis

Pour utiliser des groupes de sécurité, vous devez d'abord disposer d'une instance IBM Cloud VPC en cours d'exécution. 

Les instructions de création d'un VPC et d'un sous-réseau sont disponibles dans notre rubrique [Création d'un VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis). 

## Etape 1 : création d'un groupe de sécurité

Créez un groupe de sécurité nommé `my-security-group` dans votre instance IBM Cloud VPC.

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement, par exemple, dans la variable `sg` : 

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Etape 2 : ajout d'une règle pour autoriser les connexions SSH

Créez une règle sur le groupe de sécurité qui autorise les connexions entrantes sur le port 22.

```
curl -X POST $rias_endpoint/v1/security_groups/$sg/rules?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## Etape 3 : suppression du groupe de sécurité (facultatif)

Le groupe de sécurité peut être nettoyé uniquement s'il n'est pas associé à une interface réseau, ni référencé par une règle d'un autre groupe de sécurité. 

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
