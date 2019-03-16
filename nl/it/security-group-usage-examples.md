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

# Configurazione dei gruppi di sicurezza utilizzando le API
{: #setting-up-security-groups-using-the-apis}

Il seguente esempio illustra come creare e gestire i gruppi di sicurezza, utilizzando le API regionali {{site.data.keyword.cloud}} (RIAS).

## Prerequisiti

Per utilizzare i gruppi di sicurezza, devi prima disporre di un IBM Cloud VPC in esecuzione.

Le istruzioni per la creazione di un VPC e di una sottorete sono disponibili nel nostro argomento [Creazione di un VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Passo 1: crea un gruppo di sicurezza

Crea un gruppo di sicurezza denominato `my-security-group` nel tuo IBM Cloud VPC.

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Salva l'ID in una variabile in modo che possiamo utilizzarlo successivamente, ad esempio, la variabile `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Passo 2: aggiungi una regola per consentire le connessioni SSH

Crea una regola sul gruppo di sicurezza che consentirà le connessioni in entrata sulla porta 22.

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

## Passo 3: elimina il gruppo di sicurezza (facoltativo)

Per eliminare il gruppo di sicurezza, non può essere associato ad alcuna interfaccia di rete e non può avere un riferimento da una regola in un gruppo di sicurezza diverso.

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
