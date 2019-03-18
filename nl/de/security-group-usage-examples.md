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

# Sicherheitsgruppen über die APIs einrichten 
{: #setting-up-security-groups-using-the-apis}

Das folgende Beispiel veranschaulicht, wie Sie über den Regional Infrastructure API Service (RIAS) REST-APIs von {{site.data.keyword.cloud}} Sicherheitsgruppen erstellen und verwalten. 

## Voraussetzungen

Um Sicherheitsgruppen verwenden zu können, müssen Sie zuerst über eine aktive IBM Cloud-VPC verfügen. 

Anweisungen zum Erstellen einer VPC und eines Teilnetzes finden Sie im Abschnitt [VPC erstellen](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis). 

## Schritt 1: Sicherheitsgruppe erstellen

Erstellen Sie in Ihrer IBM Cloud-VPC eine Sicherheitsgruppe namens `my-security-group`. 

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Speichern Sie die ID in einer Variablen, sodass sie zu einem späteren Zeitpunkt verwendet werden kann, zum Beispiel in der Variablen namens `sg`: 

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Schritt 2: Regel hinzufügen, die SSH-Verbindungen zulässt 

Erstellen Sie eine Regel für die Sicherheitsgruppe, die eingehende Verbindungen an Port 22 zulässt.

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

## Schritt 3: Sicherheitsgruppe löschen (optional)

Damit die Sicherheitsgruppe gelöscht werden kann, darf sie keinen Netzschnittstellen zugeordnet sein und darf auch nicht von Regeln in anderen Sicherheitsgruppen referenziert werden. 

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
