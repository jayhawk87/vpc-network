---

copyright:
  years: 2018, 2019
  
lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# Mit IP-Adressbereichen, Adresspräfixen, Regionen und Teilnetzen arbeiten  
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}

Dieses Dokument liefert einen Überblick über die verfügbaren (oder eingeschränkten) IP-Adressbereiche und Benutzeraktionen in Bezug auf Regionen und Teilnetze und vermittelt eine Übersicht über Adresspräfixe. Bestimmte IP-Adressbereiche stehen nicht zur Verfügung, da sie möglicherweise von IBM Cloud verwendet werden.   

Wenn Sie in Ihrer {{site.data.keyword.cloud}}-VPC ein Teilnetz erstellen, wird auf der Grundlage der Region und der Zone, in der Sie die VPC erstellt haben, automatisch ein _Adresspräfix_ zugewiesen. Sie können das Standardadresspräfix verwenden. Sie können aber auch Ihr Adresspräfix einschließlich einer BYOIP-Adresse (Bring Your Own IP) in vielen Fällen manuell ändern. 

## IBM Cloud-VPC und Regionen 

Eine {{site.data.keyword.cloud}}-VPC wird in einer Region bereitgestellt, kann sich aber über mehrere Zonen erstrecken. Jede Region enthält mehrere Zonen, die jeweils unabhängige Fehlerdomänen darstellen.

Den Teilnetzen in einer VPC wird auf der Grundlage der Region und der Zone, in der sie erstellt wurden, ein _Adresspräfix_ zugeordnet. In der folgenden Tabelle werden die verfügbaren Regionen und Zonen aufgeführt.  

| Standort | Region | API-Endpunkt | Status |
| ------- | :------: | :------: |:------: |
| Dallas | us-south | `us-south.iaas.cloud.ibm.com`| Verfügbar |
| Frankfurt | eu-de | `eu-de.iaas.cloud.ibm.com`| Verfügbar |
| Washington D.C. | us-east | `us-east.iaas.cloud.ibm.com`| Demnächst verfügbar |
| Tokio | jp-tok | `jp-tok.iaas.cloud.ibm.com`| Demnächst verfügbar |
| London | eu-gb | `eu-gb.iaas.cloud.ibm.com`| Demnächst verfügbar |
| Sydney | au-syd | `au-syd.iaas.cloud.ibm.com`| Demnächst verfügbar |

## IBM Cloud-VPC und Teilnetze

Kunden können eine IBM Cloud-VPC in Teilnetze aufteilen (und tun dies in der Regel als bewährtes Verfahren). Alle Teilnetze in einer IBM Cloud-VPC können sich gegenseitig über privates L3-Routing mit einem impliziten Router erreichen.Sie müssen keine Router oder Routen einrichten. 

Hier einige praktische Fakten zu Teilnetzen in VPC:

* Ein Teilnetz besteht aus einem IP-Adressbereich, den Sie angeben. 
* Ein Teilnetz ist an eine einzelne Zone gebunden und kann sich nicht über mehrere Zonen oder Regionen erstrecken. 
* Ein Teilnetz kann die gesamte Zone in einer IBM Cloud-VPC umfassen. 
* Sie müssen zuerst Ihre virtuelle private Cloud (VPC) erstellen, bevor Sie Ihr(e) Teilnetz(e) in dieser VPC erstellen können.
* Es ist keine Unterstützung für IPv6 verfügbar. 
* Sie können eine VSI einem Teilnetz zuordnen oder ihre Zuordnung aufheben. (Dazu müssen Sie zuerst eine vNIC hinzufügen und eine Bandbreite auswählen.)

Sie können Ihren eigenen öffentlichen IPv4-Adressbereich (BYOIP) in Ihr IBM Cloud VPC-Konto einbringen. Wenn Sie BYOIP (Bring Your Own IP) verwenden, muss IBM Cloud diese IPv4-Adressen auf IBM Cloud-Ressourcen konfigurieren, die ihrerseits Pakete an und von den bereitgestellten Adressen senden. Die Verwendung Ihres bereitgestellten IPv4-Bereichs in IBM Cloud hat zur Folge, dass diese IP-Adressen möglicherweise gegenüber Mitarbeitern von IBM Support sowie gegenüber Dritten als Teil der Nutzung dieses Service offengelegt werden.
{:important}

**Abbildung: Grafische Darstellung einer VPC mit Anzeige von Zonen, Regionen und Teilnetzen **

![IBM Cloud-VPC](images/vpc-experience.png)

## Verwendung von Adresspräfixen für Teilnetze 

Jedes Teilnetz muss innerhalb eines Adresspräfixes bestehen. 
 * Für jede Zone sind vorab Präfixe für Teilnetzadressen zugeordnet.
 * Für Ihr neues Teilnetz können Sie Ihren IP-Adressbereich aus den vorhandenen Adresspräfixen auswählen.
 * Wenn die Adresspräfixe der Zone nicht geeignet sind, kann eines der vorhandenen Adresspräfixe bearbeitet oder ein neues Adresspräfix für die Zone (über die API, die CLI oder die Benutzerschnittstelle) hinzugefügt werden.
 
### Standardpräfixe für VPC-Adressen
 
Wenn eine neue VPC erstellt wird, werden die Standardadresspräfixe auf der Grundlage der Region und der Zone wie folgt zugewiesen:
 
* 'us-south-1' = '10.240.0.0/18'
* 'us-south-2' = '10.240.64.0/18'

Für andere Zonen oder Regionen können andere Standardpräfixe zugewiesen werden, darunter die folgenden:
 
 * 10.240.0.0/18
 * 10.240.64.0/18
 * 10.240.128.0/18
 
### Adresspräfixe und die Benutzerschnittstelle (UI) der IBM Cloud-Konsole
 
Wenn Sie eine VPC mit der Benutzerschnittstelle (UI) der IBM Cloud-Konsole erstellen, wählt das System automatisch Ihr Adresspräfix aus, innerhalb dessen Sie ein Teilnetz erstellen müssen. Falls dieses Adressschema nicht Ihren Anforderungen entspricht, können Sie die Adresspräfixe nach dem Erstellen der VPC anpassen. Dann können Sie in Ihren benutzerdefinierten Adresspräfixen Teilnetze erstellen und das mit dem Standardpräfix erstellte Teilnetz löschen.
 
Diese Ausweichlösung ist erforderlich, um BYOIP (Bring Your Own IP) über die Benutzerschnittstelle (UI) der IBM Cloud-Konsole nutzen zu können.
{:note}

### Verfügbare IP-Adressen

Die folgenden IP-Adressen sind verfügbar, wie in **RFC 1918** definiert:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

### Reservierte Adressen in einem Teilnetz

Bei den Angaben der IP-Adressen in dieser Liste wird vorausgesetzt, dass der CIDR-Bereich des Teilnetzes 10.10.10.0/24 lautet:

  * Erste Adresse im CIDR-Bereich (10.10.10.0): Netzadresse
  * Zweite Adresse im CIDR-Bereich (10.10.10.1): Gateway-Adresse
  * Dritte Adresse im CIDR-Bereich (10.10.10.2): durch IBM reserviert
  * Vierte Adresse im CIDR-Bereich (10.10.10.3): zur späteren Nutzung durch IBM reserviert
  * Letzte Adresse im CIDR-Bereich (10.10.10.255): Netzbroadcastadresse

### Weitere Informationen zur Erstellung eines Teilnetzes 

Es gibt zwei Möglichkeiten, ein Teilnetz für Ihre VPC anzugeben: 
  * Sie können ein Teilnetz erstellen, indem Sie die Größe des benötigten Teilnetzes angeben, beispielsweise durch die Anzahl der unterstützten Adressen (z. B. 1024).
  * Sie können ein Teilnetz erstellen, indem Sie einen CIDR-Bereich angeben (z. B. 10.0.0.1/29).
  
Das folgende Beispiel verdeutlicht, wie Sie einen Block von 1024 mithilfe von CIDR angeben: Wenn Sie anstatt einer Teilnetzgröße einen CIDR-Bereich angeben, stellt der IPv4-Block `192.168.100.0/22` die 1024 IPv4-Adressen von `192.168.100.0` bis `192.168.103.255` dar.
{:tip}

### Weitere Informationen zur Löschung eines Teilnetzes 
  * Ein Teilnetz kann nicht gelöscht werden, solange es Ressourcen (wie z. B. eine virtuelle Serverinstanz bzw. VSI, eine variable IP-Adresse oder ein öffentliches Gateway) enthält, die verwendet werden. Diese Ressourcen müssen zuerst gelöscht werden. 
  * Die Größe eines vorhandenen Teilnetzes kann NICHT geändert werden. So zum Beispiel kann 10.10.10.0/24 nicht in 10.5.1.3/20 geändert werden. 
