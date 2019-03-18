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

# (Beta) Lastausgleichsfunktionen (Load Balancer) in IBM Cloud VPC verwenden
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

Der Service für den Lastausgleich verteilt den Datenverkehr auf mehrere Serverinstanzen in derselben Region für Ihre {{site.data.keyword.cloud}}-VPC. 

**Dieser Service liegt jetzt als Betaversion vor. Ressourcen, die während der Betaphase verwendet werden, sind nach Beendigung der Betaphase nicht verfügbar, und es wird kein Support zur Verfügung stehen. Bitte richten Sie Kommentare und Fragen an Ihren IBM Cloud-Vertriebsbeauftragten.**

## Öffentliche Lastausgleichsfunktion
{: #public-load-balancer}

Ihrer Instanz des Lastausgleichsservice ist ein öffentlich zugänglicher, vollständig qualifizierter Domänenname (FQDN, Fully Qualified Domain Name) zugewiesen, den Sie für den Zugriff auf Ihre Anwendungen verwenden müssen, die hinter dem Lastausgleichsservice in {{site.data.keyword.cloud}} VPC gehostet werden. Dieser Domänenname kann mit einer oder mit mehreren öffentlichen IP-Adressen registriert sein.

Im Rahmen von Wartungs- und Skalierungsaktivitäten, die gegenüber den Kunden offengelegt werden, können sich diese öffentlichen IP-Adressen und die Anzahl der öffentlichen IP-Adressen im Laufe der Zeit ändern. Die Back-End-Serverinstanzen (VSIs), auf denen Ihre Anwendung per Hosting bereitgestellt wird, müssen in derselben Region und unter derselben VPC ausgeführt werden.

## Front-End-Listener und Back-End-Pools
Sie können bis zu 10 Front-End-Listener (Anwendungsports) definieren und diese den jeweiligen Back-End-Pools auf den Back-End-Anwendungsservern zuordnen. Der vollständig qualifizierte Domänenname (FQDN), der Ihrer Lastausgleichsserviceinstanz zugewiesen ist, und die Front-End-Listener-Ports sind für das öffentliche Internet verfügbar. An diesen Ports werden eingehende Benutzeranforderungen empfangen.

Die folgenden Front-End-Listener-Protokolle werden unterstützt:
* HTTP
* HTTPS
* TCP

Die folgenden Back-End-Pool-Protokolle werden unterstützt:
* HTTP
* TCP

Eingehender HTTPS-Datenverkehr wird an der Lastausgleichsfunktion beendet, um die HTTP-Kommunikation in unverschlüsseltem Text (Klartext) mit dem Back-End-Server zu ermöglichen.

An einen Back-End-Pool können bis zu 50 Serverinstanzen angeschlossen werden. Für jede angeschlossene Serverinstanz muss ein Port konfiguriert sein. Der Port kann mit dem Front-End-Listener-Port übereinstimmen, muss dies aber nicht.

Der Portbereich von 56500 bis 56520 ist für Verwaltungszwecke reserviert. Diese Ports können nicht für Front-End-Listener-Ports verwendet werden.
{: note}

## Lastausgleichsmethoden
{: #load-balancing-methods}

Für die Verteilung des Datenverkehrs zwischen Back-End-Anwendungsservern stehen die folgenden drei Lastausgleichsmethoden zur Verfügung:

* **Umlauf:** Der Umlauf ist die Standardmethode für den Lastausgleich. Bei dieser Methode leitet die Lastausgleichsfunktion eingehende Clientverbindungen im Umlaufverfahren an die Back-End-Server weiter. Als Ergebnis erhalten alle Back-End-Server in etwa die gleiche Anzahl von Clientverbindungen.

* **Gewichteter Umlauf:** Bei dieser Methode leitet die Lastausgleichsfunktion eingehende Clientverbindungen proportional zu der diesen Servern zugewiesenen Gewichtung an die Back-End-Server weiter. Jedem Server ist eine Standardgewichtung von 50 zugewiesen. Diese Gewichtung kann durch die Festlegung eines beliebigen Werts zwischen 0 und 100 angepasst werden. 

Wenn zum Beispiel für drei Anwendungsserver namens A, B und C eine angepasste Gewichtung von 60, 60 und 30 festgelegt wurde, so erhalten die Server A und B jeweils die gleiche Anzahl von Verbindungen, während Server C die Hälfte dieser Verbindungsanzahl erhält. 

* **Wenigste Verbindungen:** Bei dieser Methode erhält diejenige Serverinstanz, die zu einem gegebenen Zeitpunkt die geringste Anzahl von Verbindungen bereitstellt, die nächste Clientverbindung. 

**Weitere Merkmale dieser Methoden: **

* Das Zurücksetzen einer Servergewichtung auf den Wert '0' bedeutet, dass keine neuen Verbindungen an diesen Server weitergeleitet werden. Der vorhandene Datenverkehr wird jedoch weiterhin fortgesetzt, solange er aktiv ist. Die Verwendung einer Gewichtung von '0' kann helfen, einen Server ordnungsgemäß herunterzufahren und ihn aus der Servicerotation zu entfernen.
* Die Servergewichtungswerte gelten nur für die Methode des gewichteten Umlaufs. Sie werden bei den Lastausgleichsmethoden des Umlaufs und der geringsten Anzahl an Verbindungen ignoriert.


## Statusprüfungen
{: #health-checks}

Statusprüfungsdefinitionen sind für Back-End-Pools obligatorisch.

Die Lastausgleichsfunktion führt in regelmäßigen Abständen Statusprüfungen durch, um den Zustand der Back-End-Ports zu überwachen. Danach leitet sie den Clientdatenverkehr entsprechend an die Ports weiter. Wenn sich ein bestimmter Back-End-Server-Port als fehlerhaft herausstellt, werden keine neuen Verbindungen an ihn weitergeleitet. Die Lastausgleichsfunktion überwacht den Zustand fehlerhafter Ports auch weiterhin und nimmt ihre Verwendung wieder auf, wenn sie sich wieder in einwandfreiem Zustand befinden. Dazu müssen sie zwei aufeinanderfolgende Statusprüfungen erfolgreich durchlaufen haben.

Die Statusprüfungen für HTTP- und TCP-Ports werden wie folgt durchgeführt:

* **HTTP:** An den Back-End-Server-Port wird für eine vordefinierte URL eine `HTTP GET`-Anforderung gesendet. Bei Erhalt einer Antwort `200 OK` wird der Server-Port als in einwandfreiem Zustand befindlich markiert. Der Standardpfad für den `GET`-Status lautet '/' über die Benutzerschnittstelle (UI). Er kann jedoch angepasst werden.


* **TCP:** Die Lastausgleichsfunktion versucht, an einem angegebenen TCP-Port eine TCP-Verbindung mit dem Back-End-Server herzustellen. Der Server-Port wird als in einwandfreiem Zustand befindlich markiert, wenn der Verbindungsversuch erfolgreich ist, woraufhin die Verbindung beendet wird.


Das Intervall für die Statusprüfung beträgt standardmäßig 5 Sekunden und als Zeitlimit für eine Zustandsprüfungsanforderung ist standardmäßig ein Wert von 2 Sekunden definiert. Die Standardanzahl der Wiederholungsversuche beläuft sich auf 2.
{: note}

## SSL-Auslagerung und erforderliche Autorisierungen 

Die Lastausgleichsfunktion beendet für alle eingehenden HTTPS-Verbindungen die SSL-Verbindung und stellt mit der Back-End-Serverinstanz eine HTTP-Kommunikation in unverschlüsseltem Text (Klartext) her. Bei diesem Verfahren werden CPU-intensive SSL-Handshakes und Verschlüsselungs- oder Entschlüsselungstasks weg von den Back-End-Serverinstanzen verlagert, sodass diese ihre gesamten CPU-Zyklen für die Verarbeitung von Anwendungsverkehr aufwenden können.

Damit die Lastausgleichsfunktion SSL-Auslagerungstasks ausführen kann, ist ein SSL-Zertifikat erforderlich. Die Verwaltung der SSL-Zertifikate kann über [IBM Certificate Manager ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} vorgenommen werden. 

Wenn die Lastausgleichsfunktion Zugriff auf das SSL-Zertifikat in Ihrer Zertifikatmanagerinstanz benötigt, erstellen Sie eine Berechtigung, die der Serviceinstanz der Lastausgleichsfunktion Zugriff auf die Instanz des Zertifikatmanagers erteilt, die das SSL-Zertifikat enthält. Eine solche Autorisierung können Sie über [Identitäts- und Zugriffsberechtigungen ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](/docs/services/iam/#/authorizations){: new_window} verwalten. Stellen Sie sicher, dass Sie **Infrastrukturservice** als Quellenservice, **Lastausgleichsfunktion für VPC** als Ressourcentyp und **Certificate Manager** als Zielservice auswählen. Weisen Sie **Writer (Schreibberechtigter)** als Servicezugriffsrolle zu. Zum Erstellen einer Lastausgleichsfunktion müssen Sie der Quellenressourceninstanz die Autorisierung **Alle Ressourceninstanzen** erteilen. Für die Zielserviceinstanz kann **Alle Instanzen** festgelegt werden oder sie kann Ihre spezielle Certificate Manager-Ressourceninstanz sein. 

Wenn die erforderliche Autorisierung entfernt wird, können Fehler für Ihre Lastausgleichsfunktion auftreten.
{: note}

## Identity and Access Management (IAM) für das Identitäts- und Zugriffsmanagement

Sie können Zugriffsrichtlinien für eine Instanz der **Lastausgleichsfunktion für VPC** konfigurieren. Wenn Sie Benutzerzugriffsrichtlinien verwalten wollen, besuchen Sie [Identitäts- und Zugriffsberechtigungen ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](/docs/services/iam/#/authorizations){: new_window}. 

### Zugriffsrichtlinien für Ressourcengruppen für Benutzer konfigurieren

Zum Erstellen einer Lastausgleichsfunktion benötigen Sie Zugriff auf eine Ressourcengruppe. Der Benutzer, der einen Lastenausgleich erstellt, muss über ordnungsgemäßen Zugriff auf die bereitgestellte Ressourcengruppe oder aber auf die Standardressourcengruppe verfügen, falls keine bereitgestellt wird.

1. Navigieren Sie zu **Verwalten > Konto > Benutzer**. Es wird eine Liste der Benutzer mit Zugriff auf Ihr IBM Cloud-Konto angezeigt.

2. Wählen Sie den Namen des Benutzers aus, dem Sie eine Zugriffsrichtlinie zuweisen wollen. Falls der gewünschte Benutzer nicht angezeigt wird, klicken Sie auf **Benutzer einladen**, um den Benutzer zu Ihrem IBM Cloud-Konto hinzuzufügen.

3. Wählen Sie **Zugriff zuweisen** aus.
4. Wählen Sie **Zugriff in einer Ressourcengruppe zuweisen** aus.
5. Wählen Sie in der Dropdown-Liste **Ressourcengruppe** die gewünschte Ressourcengruppe aus.
6. Wählen Sie in der Dropdown-Liste **Zugriff für eine Ressourcengruppe zuweisen** den gewünschten Zugriff aus.
7. Wählen Sie in der Dropdown-Liste **Services** die gewünschten Services aus.
9. Wählen Sie **Zuweisen** aus, um dem Benutzer die Zugriffsrichtlinie für die Ressourcengruppe zuzuweisen.

### Zugriffsrichtlinien für Ressourcen für Benutzer konfigurieren

| Plattformzugriffsrolle | Aktion für die Lastausgleichsfunktion |
|-------------|-----|
| Administrator | Erstellen/Anzeigen/Bearbeiten/Löschen der Lastausgleichsfunktion |
| Editor (Bearbeiter) | Erstellen/Anzeigen/Bearbeiten/Löschen der Lastausgleichsfunktion |
| Viewer (Anzeigeberechtigter) | Anzeigen der Lastausgleichsfunktion |

1. Navigieren Sie zu **Verwalten > Konto > Benutzer**. Es wird eine Liste der Benutzer mit Zugriff auf Ihr IBM Cloud-Konto angezeigt.
2. Wählen Sie den Namen des Benutzers aus, dem Sie eine Zugriffsrichtlinie zuweisen wollen. Falls der gewünschte Benutzer nicht angezeigt wird, klicken Sie auf **Benutzer einladen**, um den Benutzer zu Ihrem IBM Cloud-Konto hinzuzufügen.
3. Wählen Sie **Zugriff zuweisen** aus.
4. Wählen Sie **Zugriff auf Ressourcen zuweisen** aus.
5. Wählen Sie in der Dropdown-Liste **Services** den Eintrag **Infrastrukturservice** aus.
6. Wählen Sie in der Dropdown-Liste **Ressourcentyp** den Eintrag **Lastausgleichsfunktion für VPC** aus.
7. Wählen Sie in der Dropdown-Liste **ID der Lastausgleichsfunktion** die ID für eine Lastausgleichsfunktionsinstanz aus oder verwenden Sie den Standardwert **Alle Lastausgleichsfunktionen**.
8. Weisen Sie dem Benutzer einen Plattformzugriff zu.
9. Wählen Sie **Zuweisen** aus, um dem Benutzer die Zugriffsrichtlinie zuzuweisen.

## Verfügbare Anwendungsprogrammierschnittstellen (APIs)

Zum Durchführen von API-Aufrufen müssen Sie eine Form des REST-Clients verwenden. Sie können zum Beispiel den Befehl `curl` verwenden, um alle vorhandenen Lastausgleichsfunktionen abzurufen:

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

Der folgende Abschnitt liefert detaillierte Angaben zu den APIs, die Sie für Lastausgleichsfunktionen in Ihrer VPC-Umgebung verwenden können. Die vollständigen Spezifikationen enthält die [Referenz für die RIAS-API](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers). 

| Beschreibung | API |
|-------------|-----|
| Erstellt eine Lastausgleichsfunktion und stellt sie bereit | POST /load_balancers |
| Ruft alle Lastausgleichsfunktionen ab | GET /load_balancers |
| Ruft eine Lastausgleichsfunktion ab | GET /load_balancers/{id} |
| Löscht eine Lastausgleichsfunktion | DELETE /load_balancers/{id} |
| Aktualisiert eine Lastausgleichsfunktion | PATCH /load_balancers/{id} |
| Erstellt einen Listener | POST /load_balancers/{id}/listeners |
| Ruft alle Listener auf der Lastausgleichsfunktion ab | GET /load_balancers/{id}/listeners |
| Ruft einen Listener ab | GET /load_balancers/{id}/listeners/{listener_id} |
| Löscht einen Listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Aktualisiert einen Listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Erstellt einen Pool | POST /load_balancers/{id}/pools |
| Ruft alle Pools der Lastausgleichsfunktion ab | GET /load_balancers/{id}/pools |
| Ruft eine Pool ab | GET /load_balancers/{id}/pools/{pool_id} |
| Löscht einen Pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Aktualisiert einen Pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Erstellt ein Mitglied (Member) | POST /load_balancers/{id}/pools/{pool_id}/members |
| Ruft alle Mitglieder des Pools ab | GET /load_balancers/{id}/pools/{pool_id}/members |
| Ruft ein Mitglied ab |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Löscht ein Mitglied aus dem Pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aktualisiert ein Mitglied | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aktualisiert Mitglieder des Pools | PUT /load_balancers/{id}/pools/{pool_id}/members |

## Beispiel einer Lastausgleichsfunktion 

Im folgenden Beispiel verwenden Sie die API, um eine Lastausgleichsfunktion vor zwei VPC-Serverinstanzen (`192.168.100.5` und `192.168.100.6`) zu erstellen, die eine Webanwendung ausführen, die an Port `80` empfangsbereit ist. Die Lastausgleichsfunktion verfügt über einen Front-End-Listener, der den sicheren Zugriff auf die Webanwendung mittels HTTPS ermöglicht. Nach der Erstellung der Lastausgleichsinstanz können Sie über die API Details zu der Lastausgleichsinstanz abrufen und die Instanz der Lastausgleichsfunktion löschen.

### Beispielschritte

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Bereitstellen einer VPC, einem oder mehreren Teilnetzen und Instanzen über die [IBM Cloud-UI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), die [CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) oder die [RIAS-API](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) übersprungen.  


Die Beispielschritte für die Lastausgleichsfunktion können auch über die [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference) ausgeführt werden.
{: note}

#### Schritt 1. Lastausgleichsfunktion mit Listener-, Pool- und angehängten Serverinstanzen (Mitgliedern) erstellen

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

Beispielausgabe:
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

Speichern Sie die ID der Lastausgleichsfunktion zur Verwendung in den nachfolgenden Schritten zum Beispiel in der Variablen `lbid`. 

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### Schritt 2. Lastausgleichsfunktion abrufen

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

Rechnen Sie etwas Zeit für die Bereitstellung der Lastausgleichsfunktion ein. Wenn sie bereit ist, wird sie in den Status `Online` und `Aktiv` versetzt, wie in der folgenden Beispielausgabe zu sehen ist: 

Beispielausgabe:

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

#### Schritt 3. Lastausgleichsfunktion löschen

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## Häufig gestellte Fragen (FAQs)

Dieser Abschnitt enthält Antworten auf häufig gestellte Fragen (FAQs) zum Service der **Lastausgleichsfunktion für VPC**. 

### Kann ich einen anderen DNS-Namen für meine Lastausgleichsfunktion verwenden?

Der DNS-Name für die Lastausgleichsfunktion wird automatisch zugeordnet und kann nicht angepasst werden. Sie können jedoch einen CNAME-Datensatz für den kanonischen Namen hinzufügen, der mit dem bevorzugten DNS-Namen auf den automatisch zugewiesenen DNS-Namen der Lastausgleichsfunktion verweist. Wenn Ihre Lastausgleichsfunktion zum Beispiel die ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727` hat, so lautet der ihr automatisch zugewiesene DNS-Name `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`. Nehmen Sie an, dass Ihr bevorzugter DNS-Name `www.myapp.com` wäre. Sie könnten dann (über den DNS-Provider, den Sie zum Verwalten von `myapp.com` verwenden) einen CNAME-Datensatz hinzufügen, mit dem Sie `www.myapp.com` auf den DNS-Namen `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud` der Lastausgleichsfunktion verweisen. 

### Wie viele Front-End-Listener kann ich mit meiner Lastausgleichsfunktion maximal definieren?

10.

### Wie viele Serverinstanzen kann ich maximal meinem Back-End-Pool zuordnen?

50.

### Kann ich eine rein interne, private Lastausgleichsfunktion erstellen, auf die nur interne Clients zugreifen können?   

Nein, zum jetzigen Zeitpunkt ist das nicht möglich.

### Ist die Lastausgleichsfunktion horizontal skalierbar?  

Nein, zum jetzigen Zeitpunkt ist das nicht möglich.

### Was soll ich tun, wenn ich ACLs oder Sicherheitsgruppen für die Teilnetze verwende, die zum Bereitstellen der Lastausgleichsfunktion verwendet werden?

Sie müssen sicherstellen, dass die richtigen ACL- oder Sicherheitsgruppenregeln vorhanden sind, damit eingehender Datenverkehr für konfigurierte Listener-Ports und Management-Ports (Ports 56500-56520) zugelassen wird. Zwischen der Lastausgleichsfunktion und Back-End-Instanzen sollte der Datenverkehr ebenfalls zulässig sein.

### Warum erhalte ich eine Meldung mit dem Inhalt, dass die Zertifikatsinstanz nicht gefunden wurde (`certificate instance not founnot found d`)? 

* Gegebenenfalls ist die Zertifikatsinstanz ungültig.
* Möglicherweise haben Sie keine Service-zu-Service-Autorisierung erteilt. Lesen Sie hierzu den Abschnitt **SSL-Auslagerung** im vorliegenden Dokument. 

### Warum wird ein Fehlercode vom Typ `401 – Nicht autorisiert` zurückgegeben?

Überprüfen Sie die folgenden Zugriffsrichtlinien für Ihren Benutzer:
* Zugriffsrichtlinie für den Ressourcentyp 'Lastausgleich'
* Zugriffsrichtlinie für Ressourcengruppen

### Warum hat meine Lastausgleichsfunktion den Status `maintenance_pending`? 

Die Lastausgleichsfunktion weist den Status `maintenance_pending` bei diversen Verwaltungsaktivitäten auf, wie zum Beispiel den folgenden: 
* Rollierendes Upgrade, bei dem Sicherheitslücken angesprochen und Sicherheitspatches angewendet werden 
* Wiederherstellungsaktivitäten

### Warum muss ich bei der Bereitstellung mehrere Teilnetze auswählen?

Lastausgleichsappliances werden in den von Ihnen ausgewählten Teilnetzen bereitgestellt. Es wird dringend empfohlen, Teilnetze aus verschiedenen Zonen auszuwählen, um eine höhere Verfügbarkeit und Redundanz zu erzielen.
