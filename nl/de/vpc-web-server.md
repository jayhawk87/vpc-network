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

# Konnektivität zwischen Web-Servern in Ihrer VPC einrichten
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

In diesem Dokument werden einige Beispielschritte für die Einrichtung der Kommunikation zwischen zwei Web-Servern in Ihrer {{site.data.keyword.cloud}}-VPC mit Python beschrieben. Es veranschaulicht, wie Sie kommunizierende Server in derselben Zone sowie in unterschiedlichen Zonen erstellen. 

## Voraussetzungen

Bevor Sie die Kommunikation zwischen Webservern in Ihrer VPC einrichten können, müssen Sie bereits eine VPC mit ZWEI Teilnetzen erstellt haben, wobei in jedem Teilnetz mindestens ZWEI Instanzen verfügbar sein müssen. Ihnen müssen die IDs der VPC, der Teilnetz und der Instanzen bekannt sein. Wenn Sie Hilfe bei der Einrichtung dieser Konfiguration benötigen, führen Sie den Anweisungen wie im [Leitfaden zur Erstellung von VPC-Ressourcen über die CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) beschrieben. 

## Szenario 1: Konnektivität zwischen Servern, die sich in derselben Zone, aber in unterschiedlichen Teilnetzen befinden

`vpc` enthält die ID für die VPC.
`subnet1` enthält die ID für Teilnetz 1.
`subnet2` enthält die ID für Teilnetz 2. 


## Teil 1: Konnektivität zwischen Servern, die sich in derselben Zone und VPC befinden 

Nachdem Sie mindestens zwei Instanzen in demselben Teilnetz erstellt haben, an das ein öffentliches Gateway angehängt ist, können Sie die Konnektivität dieser Instanzen testen. Dazu stellen Sie in einer Instanz per Hosting einen Server mit HTML-Code bereit, richten sich dann mit `curl` an diesen Server und laden die HTML-Datei auf die anderen Server oder über den Browser Ihres Computers herunter. 

Nehmen Sie an, dass sich Ihre Instanzen `instanz_1` und `instanz_2` in demselben Teilnetz befinden und dieses über ein öffentliches Gateway verfügt, und dass sie derselben Sicherheitsgruppe angehören. Damit der eingehende Zugriff möglich ist, müssen Sie in 'Sicherheitsgruppen' über die API, CLI oder UI Regeln für die Instanz hinzufügen. Wenn Sie Ihren Server auf `instanz_1` hosten, müssen Sie eine Regel für die variable IP-Adresse von `instanz_2` hinzufügen. Nehmen Sie als Beispiel an, dass für den Host-Server von `instanz_1` die IPv4-IP `169.61.161.161` und von `instanz_2` die IPv4-IP `169.61.161.235` festgelegt ist und versucht wird, die Datei mit GET von `instanz_1` abzurufen. Die Regel müsste wie folgt aussehen: 

![Neue Regeln für Sicherheitsgruppe](images/security-group-ui-ex1.png)

* Verwenden Sie zum Abrufen einer Liste aller Sicherheitsgruppen den Befehl `ibmcloud is sgs`.
* Mit der folgenden API-Anforderung kann das vorherige Ergebnis abgerufen werden: 

```
curl -X POST $rias_endpoint/v1/security_groups/{sicherheitsgruppen-ID von instanz_1}/rules?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. Erstellen Sie auf `instanz_1` die Basisversion einer Datei `index.html`. Die Basisversion einer HTML-Datei sieht ungefähr wie im folgenden Beispiel aus: 

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

2. Führen Sie in demselben Verzeichnis, in dem sich die Datei befindet, den Befehl `python -m SimpleHTTPServer <port number>` aus. Falls der Standardport 8000 bereits belegt ist, können Sie einen anderen Port verwenden. 

3. Stellen Sie über SSH eine Verbindung mit `instanz_2`her. 

Wenn Sie nach einer variablen IP-Adresse suchen wollen, führen Sie den Befehl `ibmcloud is ips` aus. Führen Sie zum Identifizieren der ID einer variablen IP-Adresse (`floating_ip_id`) den Befehl `ibmcloud is ip $floating_ip_id` aus. Damit müssten Sie in der Lage sein, die öffentliche Adresse zu ermitteln. 

4. Ermitteln Sie die IP-Adresse für `instanz_1`.

5. Fordern Sie mit `curl -X GET http://IP-adresse:portnummer/index.html` den Download der Datei `index.html` vom Hosting-Server `instanz_1` an. (Beispiel: `curl -X GET http://169.61.161.161:8000/index.html`)

6. Jetzt müsste die Ausgabe von `index.html` angezeigt werden. 

7. Vorgehensweise, um den Zugriff auf den Server über den Browser Ihres lokalen Computers zu ermöglichen: Fügen Sie für den gesamten eingehenden Datenverkehr Regeln hinzu, wie in der folgenden Abbildung dargestellt: 

![Neue Regeln für Sicherheitsgruppe für gesamten eingehenden Datenverkehr](images/security-group-ui-ex2.png)

* So fügen Sie Regeln, mit denen ALLE Protokolle und von allen Quellen zugelassen werden, zu Ihren Sicherheitsgruppen hinzu: 

```
curl -X POST $rias_endpoint/v1/security_groups/{sicherheitsgruppen-ID}/rules/{instanz-ID}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**Hier das Beispiel der Anforderung im Detail:**

```
curl -X POST $rias_endpoint/v1/security_groups/{sicherheitsgruppen-ID}/rules?version=2019-01-01
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

8. Öffnen Sie einen Web-Browser und geben Sie `http://IP-adresse:portnummer/index.html` ein.

9. Die Indexdatei müsste jetzt in HTML-Format angezeigt werden. 

## Teil 2: Konnektivität zwischen Servern, die sich in verschiedenen Zonen und VPCs befinden

Verbinden Sie die Instanz des anderen Teilnetzes, für das kein öffentliches Gateway eingerichtet ist, mit der Instanz mit Internetzugriff. Dabei sollten Sie zwei verschiedene Arten von Instanzen einrichten: 

* Einen Web-Server, der Anforderungen aus dem Internet über ein öffentliches Gateway abwickelt. 
* Einen Back-End-Server, auf dem eine Anwendung ausgeführt wird und der wichtige Daten speichert, wobei Datenverkehr nur intern stattfindet und das öffentliche Gateway inaktiviert ist. 

Sicherheitsgruppen (SGs) gelten für virtuelle Serverinstanzen (VSIs), während Netz-ACLs für Teilnetze gelten. 

Um diese Task in ihrer Gesamtheit auszuführen, müssen Sie daher VSIs erstellen, Teilnetze erstellen, Regeln für Sicherheitsgruppen (SGs) sowie für Zugriffssteuerungslisten (ACLs) erstellen, dann die Sicherheitsgruppe an die Netzschnittstelle einer Instanz anhängen und die Netz-ACL an ein bestimmtes Teilnetz anhängen. 

* So listen Sie alle Regeln in der Sicherheitsgruppe `subnet1` auf: 

```
curl -X GET $rias_endpoint/v1/security_groups/{sicherheitsgruppen-ID}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. Entfernen Sie die Regel `inbound-allow-all-traffic` in der Sicherheitsgruppe der Teilnetzübung in Teil 1

```
curl -X DELETE $rias_endpoint/v1/security_groups/{sicherheitsgruppen-ID}/rules/{regel-ID}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. Erstellen Sie eine neue VPC und erstellen Sie anschließend ein Teilnetz ohne angehängtes öffentliches Gateway. 

3. Erstellen Sie die Instanz `instance_3` in dem neuen Teilnetz ohne öffentliches Gateway (d. h. ohne Internetverbindung). Dieser Instanz sollten Sie aus Gründen des Back-Ends gegebenenfalls größere CPU-Cores und mehr Hauptspeicher zuordnen. 

4. Fügen Sie neue Regeln für `instanz_3` in der Sicherheitsgruppe aus Teil 1 hinzu. Beispiel: `instanz_3` hat die variable IP-Adresse `169.61.181.25`. 

![Neue Regeln für Sicherheitsgruppe mit Instanz von anderem Teilnetz für eingehenden Datenverkehr](images/security-group-ui-ex3.png)

5. Verwenden Sie einen Befehl im Format `curl -X GET http://IP-adresse:portnummer/index.html`, um den Download der Datei `index.html` vom Hosting-Server `instanz_1` anzufordern. (Beispiel: `curl -X GET http://169.61.161.161:8000/index.html`)

6. Jetzt müsste die Ausgabe von `index.html` angezeigt werden. 

Dies bedeutet, dass eine Verbindung zwischen den Instanzen 3 und 1 besteht, nicht aber zu Instanz 2. 

## Nächste Schritte

Sie sollten Ihre Server schützen, indem Sie ACL-Regeln formulieren, die den eingehenden Datenverkehr steuern. Die entsprechende Vorgehensweise dazu wird im [Leitfaden zur Verwendung von Zugriffssteuerungslisten (ACLs) in VPC](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli) erläutert. 
