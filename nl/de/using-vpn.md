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

# (Beta) VPN mit Ihrer VPC-Instanz verwenden
{: #--beta-using-vpn-with-your-vpc}

Der VPN-Service von {{site.data.keyword.cloud}} ermöglicht Ihnen, private Netze auf sichere Weise zu verbinden. Sie können VPN verwenden, um von einem Standort zu einem anderen einen IPsec-Tunnel zwischen Ihrer VPC-Instanz und Ihrem lokalen privaten Netz oder einer anderen VPC-Instanz einzurichten.

Für das aktuelle Release von {{site.data.keyword.cloud}} VPC wird nur die richtlinienbasierte Weiterleitung unterstützt.

**Dieser Service liegt jetzt als Betaversion vor. Ressourcen, die während der Betaphase verwendet werden, sind nach Beendigung der Betaphase nicht verfügbar, und es wird kein Support zur Verfügung stehen. Bitte richten Sie Kommentare und Fragen an Ihren IBM Cloud-Vertriebsbeauftragten.**

## Features

* IKEv1 und IKEv2
* Authentifizierungsalgorithmen: `md5`, `sha1`, `sha256`
* Verschlüsselungsalgorithmen: `3des`, `aes128`, `aes256`
* Diffie-Hellman-Gruppen (DH-Gruppen): 2, 5, 14
* IKE-Vereinbarungsmodus: main
* IPSec-Transformationsprotokoll: ESP
* IPSec-Kapselungsmodus: tunnel
* Absolute vorwärts gerichtete Sicherheit (PFS, Perfect Forward Secrecy)
* Erkennung inaktiver Peers
* Weiterleitung: Richtlinienbasiert
* Authentifizierungsmodus: Vorab verteilter Schlüssel
* Hochverfügbarkeitsunterstützung

## Verfügbare Anwendungsprogrammierschnittstellen (APIs)

Der folgende Abschnitt liefert detaillierte Angaben zu den APIs, die Sie für VPN in Ihrer IBM Cloud VPC-Umgebung verwenden können. Weitere Details können Sie der Seite [VPC-REST-APIs](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies) entnehmen.

### VPN-Gateways und VPN-Verbindungen

| Beschreibung | API |
|----------------------------|-------------|
| Erstellt ein VPN-Gateway | POST /vpn_gateways |
| Ruft VPN-Gateways ab | GET /vpn_gateways |
| Ruft ein VPN-Gateway ab | GET /vpn_gateways/{id} |
| Löscht ein VPN-Gateway | DELETE /vpn_gateways/{id} |
| Aktualisiert ein VPN-Gateway | PATCH /vpn_gateways/{id} |
| Erstellt eine neue VPN-Verbindung | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Ruft VPN-Verbindungen ab | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Ruft eine VPN-Verbindung ab | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Löscht eine VPN-Verbindung | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Aktualisiert eine VPN-Verbindung | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Ruft alle lokalen CIDRs für eine VPN-Verbindung ab | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Löscht eine lokale CIDR aus einer VPN-Verbindung | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Prüft, ob eine bestimmte lokale CIDR in einer VPN-Verbindung vorhanden ist | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Legt eine lokale CIDR für eine VPN-Verbindung fest | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Ruft alle Peer-CIDRs für eine VPN-Verbindung ab | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Löscht eine Peer-CIDR aus einer VPN-Verbindung | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Prüft, ob eine bestimmte Peer-CIDR in einer VPN-Verbindung vorhanden ist | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Legt eine Peer-CIDR für eine VPN-Verbindung fest | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE-Richtlinien

| Beschreibung | API |
|-----------------------------|--------------|
| Ruft alle IKE-Richtlinien ab | GET /ike_policies |
| Erstellt eine IKE-Richtlinie | POST /ike_policies |
| Löscht eine IKE-Richtlinie | DELETE /ike_policies/{id} |
| Ruft eine IKE-Richtlinie ab | GET /ike_policies/{id} |
| Aktualisiert eine IKE-Richtlinie | PATCH /ike_policies/{id} |
| Ruft alle Verbindungen ab, die die angegebene IKE-Richtlinie verwenden | GET /ike_policies/{id}/connections |

### IPsec-Richtlinien

| Beschreibung | API |
|---------------------|-------------|
| Ruft alle IPsec-Richtlinien ab | GET /ipsec_policies |
| Erstellt eine IPsec-Richtlinie | POST /ipsec_policies |
| Löscht eine IPsec-Richtlinie | DELETE /ipsec_policies/{id} |
| Ruft eine IPsec-Richtlinie ab | GET /ipsec_policies/{id} |
| Aktualisiert eine IPsec-Richtlinie | PATCH /ipsec_policies/{id} |
| Ruft alle Verbindungen ab, die die angegebene IPsec-Richtlinie verwenden | GET /ipsec_policies/{id}/connections |

## VPN-Beispiel

Im folgenden Beispiel sind Sie in der Lage, zwei VPCs über VPN miteinander zu verbinden. Das bedeutet, dass Sie Teilnetze in zwei separaten VPCs so verbinden können, als wären sie ein einzelnes Netz. Die IP-Adressen der Teilnetze dürfen sich nicht überschneiden.
Das Szenario sieht wie folgt aus, wobei in jeder VPC-Instanz einige virtuelle Maschinen (VMs) hinzugefügt wurden:
![VPN-Beispielszenario](images/vpc-vpn.png)

### Beispielschritte

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der IBM Cloud-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) und in [VPC-Einrichtung mit APIs](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

Wahlweise können Sie über die Benutzerschnittstelle (UI) ein VPN-Gateway erstellen. Die entsprechenden Schritte sind im [Dokument für das Lernprogramm für die Konsole](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) beschrieben.

#### Schritt 1. VPN-Gateway in Ihrem VPC-Teilnetz erstellen

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Beispielausgabe:
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

Stellen Sie sicher, dass die Angaben für die folgenden Felder für die nachfolgenden Schritte gespeichert werden.
* `id`: Dies ist die VPN-Gateway-ID. Sie wird hier als `$gwid1` bezeichnet.
* `address`: Dies ist die öffentliche IP-Adresse des VPN-Gateways. Sie wird hier als `$gwaddress1` bezeichnet.

**Hinweis:** Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen. Sie können den Status des Gateways mit dem folgenden Befehl überprüfen:

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### Schritt 2. Zweites VPN-Gateway auf einer anderen VPC erstellen

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Beispielausgabe:
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

Stellen Sie sicher, dass Sie die Angaben für die folgenden Felder für die nachfolgenden Schritte speichern.
* `id`: Dies ist die VPN-Gateway-ID. Sie wird hier als `$gwid2` bezeichnet. 
* `address`: Dies ist die öffentliche IP-Adresse des VPN-Gateways. Sie wird hier als `$gwaddress2` bezeichnet. 


#### Schritt 3. VPN-Verbindung vom ersten VPN-Gateway zum zweiten VPN-Gateway erstellen, wobei für `local_cidrs` das Teilnetz auf der 1 und für `peer_cidrs` das Teilnetz auf VPC 2 festgelegt wird 

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

Beispielausgabe:
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

#### Schritt 4. VPN-Verbindung vom zweiten VPN-Gateway zum ersten VPN-Gateway erstellen, wobei für `local_cidrs` das Teilnetz auf VPC 2 und für `peer_cidrs` das Teilnetz auf VPC 1 festgelegt wird 

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

Beispielausgabe:
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

#### Schritt 5. Konnektivität überprüfen

Sobald die VPN-Verbindung hergestellt ist, können Sie Ihre Instanzen in Teilnetz 2 von Teilnetz 1 aus erreichen und umgekehrt. 

Sie können den Status der VPN-Verbindung wie folgt prüfen: 
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

Beispielausgabe:
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

## Beschränkungen bzw. Kontingente

Informationen zu Beschränkungen bzw. Kontingenten für VPNs finden Sie im Abschnitt [VPC-Kontingente](/docs/infrastructure/vpc?topic=vpc-quotas#vpn-quotas). 

## Automatische Vereinbarung von Richtlinien 

Die Verwendung von IKE- und IPsec-Richtlinien zum Konfigurieren einer VPN-Verbindung ist optional. Wenn keine Richtlinie ausgewählt ist, werden automatisch Standardvorschläge für ein Verfahren ausgewählt, das als _automatische Vereinbarung_ bezeichnet wird. Nachfolgend sind die Vorschläge aufgeführt, die im Rahmen der automatischen Vereinbarung verwendet werden.

IBM Cloud verwendet **IKEv2** als Initiator. Als Responder akzeptiert IBM Cloud sowohl **IKEv1** als auch **IKEv2**.
{: note}

### Automatische IKE-Vereinbarung (Phase 1)

Die folgenden Optionen für die Verschlüsselung, die Authentifizierung und die Diffie-Hellman-Gruppe (DH-Gruppe) können in beliebiger Kombination verwendet werden:

|    | Verschlüsselung | Authentifizierung | DH-Gruppe |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Automatische IPsec-Vereinbarung (Phase 2)

Die folgenden Optionen für die Verschlüsselung und die Authentifizierung können in beliebiger Kombination verwendet werden:

|    | Verschlüsselung | Authentifizierung | DH-Gruppe |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | inaktiviert  |
| 2  | aes256 | sha256 | inaktiviert  |
| 3  | 3des   | md5    | inaktiviert  |

## Häufig gestellte Fragen (FAQs)

**Kann ich beim Erstellen eines VPN-Gateways gleichzeitig auch VPN-Verbindungen erstellen?**

Wenn Sie die Anwendungsprogrammierschnittstelle (API) oder die Befehlszeilenschnittstelle (CLI) verwenden, müssen VPN-Verbindungen erstellt werden, nachdem das VPN-Gateway erstellt worden ist. Bei Verwendung der IBM Cloud-Konsole können Sie das Gateway und eine Verbindung zur gleichen Zeit erstellen.

**Was passiert mit den Verbindungen, wenn ich ein VPN-Gateway lösche, an das VPN-Verbindungen angehängt sind?**

Die VPN-Verbindungen werden zusammen mit dem VPN-Gateway gelöscht.

**Werden die IKE- oder IPsec-Richtlinien gelöscht, wenn ich ein VPN-Gateway oder eine VPN-Verbindung lösche?**

Nein, IKE- und IPsec-Richtlinien können für mehrere Verbindungen gelten.

**Was passiert mit einem VPN-Gateway, wenn ich versuche, das Teilnetz zu löschen, in dem sich das Gateway befindet?**

Das Teilnetz kann nicht gelöscht werden, solange noch Instanzen vorhanden sind. Dies gilt auch für das VPN-Gateway.

**Gibt es IKE- und IPsec-Standardrichtlinien?**

Wenn Sie eine VPN-Verbindung erstellen, ohne auf eine Richtlinien-ID (IKE oder IPsec) zu verweisen, wird die automatische Vereinbarung verwendet.

**Unterstützt VPN für VPC HA-Konfigurationen für Hochverfügbarkeit?**

Ja, diese Unterstützung ist gegeben.
