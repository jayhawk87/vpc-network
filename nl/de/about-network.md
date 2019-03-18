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
{:note: .note}
{:important: .important}
{:download: .download}

# Informationen zum Netzbetrieb für VPC
{: #about-networking-for-vpc}

Eine virtuelle private Cloud (VPC, Virtual Private Cloud) ist ein virtuelles Netz, das an Ihr Kundenkonto gebunden ist. Es gibt Ihnen Cloudsicherheit und ermöglicht anhand der bereitgestellten feinkörnigen Steuerung der virtuellen Infrastruktur und der Segmentierung Ihres Datenverkehrs eine dynamische Skalierung.

In diesem Dokument werden einige Konzept des Netzbetriebs behandelt, wie sie in {{site.data.keyword.cloud}} VPC Anwendung gefunden haben. Es werden unter anderem die folgenden Anwendungsfälle und Merkmale beschrieben: 

* Regionen 
* Zonen 
* Reservierte IP-Adressen
* Öffentliche Gateways
* vNIC-Schnittstellen
* Teilnetze
* Variable IP-Adressen
* VPN-Verbindungen

## Übersicht

Gehen Sie wie folgt vor, um den Netzbetrieb in Ihrem VPC einzurichten: 
* Verwenden Sie ein öffentliches Gateway für den Internetdatenverkehr im Teilnetz. 
* Verwenden Sie eine variable IP-Adresse für den Internetdatenverkehr in virtuellen Serverinstanzen (VSIs, Virtual Server Instances). 
* Verwenden Sie ein VPN für sichere externe Konnektivität. 

Berücksichtigen Sie Folgendes: 
* Manche Teilnetzadressbereiche für CIDR sind durch IBM reserviert. 
* Sie müssen Ihre VPC erstellen, bevor Sie Teilnetze in dieser VPC erstellen können. 
* Es ist keine Unterstützung für IPv6 verfügbar. 

Optional können Sie eine VPC mit klassischem Zugriff für die Verbindungsherstellung zur klassischen Infrastruktur Ihrer IBM Cloud erstellen.
{:note}

Die nachstehende Abbildung veranschaulicht Folgendes: 
* Teilnetze in Ihrer virtuellen privaten Cloud (VPC) können über ein optionales öffentliches Gateway (PGW, Public Gateway) eine Verbindung zum öffentlichen Internet herstellen. 
* Jeder virtuellen Serverinstanz (VSI, Virtual Server Instance) kann eine variable IP-Adresse zugewiesen werden, damit sie vom Internet aus erreichbar ist oder damit sie das Internet erreichen kann.
* Teilnetze innerhalb der VPC von IBM Cloud bieten private Konnektivität und können durch den impliziten Router über eine private Verbindung miteinander kommunizieren. Eine Einrichtung von Routen ist nicht erforderlich.


**Abbildung: Kunden können eine virtuelle private Cloud (VPC) mit Teilnetzen unterteilen, von denen jedes das öffentliche Internet erreichen kann, sofern erwünscht. **

![VPC-Konnektivität](/images/vpc-connectivity-and-security.png)

## Terminologie

Dieses [Glossar](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary) enthält Definitionen und Informationen zu den Begriffen, die in diesem Dokument für IBM Cloud VPC verwendet werden. Wenn Sie mit Ihrer virtuellen privaten Cloud (VPC) arbeiten, müssen Sie mit den Grundkonzepten der Begriffe _Region_ und _Zone_ vertraut sein, da diese sich auf Ihre Bereitstellung beziehen. 

### Regionen 

Eine Region ist eine Abstraktion, die sich auf den geografischen Bereich bezieht, in dem ein VPC bereitgestellt wird. Jede Region enthält mehrere Zonen, die jeweils unabhängige Fehlerdomänen darstellen. Eine IBM Cloud-VPC kann sich in der ihr zugewiesenen Region über mehrere Zonen erstrecken. 

### Zonen

Eine Zone ist eine Abstraktion, die sich auf das physische Rechenzentrum bezieht, das die Rechen-, Netz- und Speicherressourcen sowie die damit verbundene Kühlung und Stromversorgung hostet und die Services und Anwendungen bereitstellt. Zonen sind voneinander isoliert, damit kein gemeinsamer Single Point of Failure (SPoF) entsteht, wohl aber eine Verbesserung der Fehlertoleranz und geringere Latenzzeiten erzielt werden können. Eine Zone stellt die folgenden Eigenschaften sicher: 

 * Jede Zone ist eine unabhängige Fehlerdomäne und es ist äußerst unwahrscheinlich, dass zwei Zonen in einer Region gleichzeitig ausfallen. 
 * Für den Datenverkehr zwischen den Zonen in einer Region trifft eine Latenzzeit von weniger als 2 Millisekunden zu. 

## Merkmale von Teilnetzen in der VPC 

Ein Teilnetz besteht aus einem angegebenen IP-Adressbereich (CIDR-Block). Teilnetze sind auf eine einzelne Zone beschränkt und können sich nicht über mehrere Zonen oder Regionen erstrecken. Ein Teilnetz kann jedoch die Gesamtheit der Zonenabstraktionen in ihrer virtuellen privaten Cloud umfassen. Teilnetze in derselben IBM Cloud-VPC sind miteinander verbunden.


### Reservierte IP-Adressen

Bestimmte IP-Adressen sind bei Verwendung der virtuellen privaten Cloud für die Verwendung durch IBM reserviert. Vorausgesetzt, dass der CIDR-Bereich des Subnetzes 10.10.10.0/24 ist, lauten diese reservierten Adressen wie folgt:

  * Erste Adresse im CIDR-Bereich (10.10.10.0): Netzadresse
  * Zweite Adresse im CIDR-Bereich (10.10.10.1): Gateway-Adresse
  * Dritte Adresse im CIDR-Bereich (10.10.10.2): durch IBM reserviert
  * Vierte Adresse im CIDR-Bereich (10.10.10.3): zur späteren Nutzung durch IBM reserviert
  * Letzte Adresse im CIDR-Bereich (10.10.10.255): Netzbroadcastadresse

### Öffentliches Gateway für die externe Konnektivität eines Teilnetzes verwenden

Ein **öffentliches Gateway** ermöglicht einem Teilnetz (und allen an dieses Teilnetz angebundenen virtuellen Serverinstanzen) die Verbindungsherstellung zum Internet. Beachten Sie, dass Teilnetze standardmäßig privat sind. Sie können jedoch optional ein öffentliches Gateway (PGW) erstellen und an dieses ein Teilnetz anhängen. Nachdem ein Teilnetz an das öffentliche Gateway angehängt worden ist, können alle virtuelle Serverinstanzen in diesem Teilnetz eine Verbindung mit dem Internet herstellen.

Das öffentliche Gateway verwendet die _Netzadressumsetzung (NAT, Network Address Translation) vom Typ 'Viele-zu-eins'_, was bedeutet, dass mehrere Tausend virtuelle Serverinstanzen mit privaten Adressen genau eine öffentliche IP-Adresse verwenden, um mit dem öffentlichen Internet zu kommunizieren.Das öffentliche Gateway ermöglicht dem Internet nicht, eine Verbindung zu diesen Instanzen zu starten. Verwenden Sie die Anwendungsprogrammierschnittstelle (API), um Teilnetze an Ihr öffentliches Gateway anzuhängen oder von diesem abzuhängen.

Die folgende Abbildung stellt eine Zusammenfassung des derzeitigen Geltungsbereichs von Gateway-Services dar.

![Gateway-Services](images/scope-of-gateway-services.png)

Ein öffentliches Gateway, das in einer VPC-Instanz erstellt wird, erfüllt so lange keinerlei Funktion, bis es an ein Teilnetz angeschlossen worden ist. Pro Zone kann nur ein öffentliches Gateway erstellt werden. Das bedeutet, dass in einer Umgebung mit drei Zonen drei öffentliche Gateways pro VPC vorhanden sein können.
{:note}

### Variable IP-Adresse für die externe Konnektivität einer virtuellen Servierinstanz (VSI) verwenden
**Variable IP-Adressen** sind IP-Adressen, die vom System bereitgestellt werden und vom öffentlichen Internet aus erreichbar sind.

Sie können eine variable IP-Adresse aus dem Pool der von IBM bereitgestellten verfügbaren variablen IP-Adressen reservieren und diese reservierte IP-Adresse jeder beliebigen Instanz in derselben VPC zuordnen, und zwar mithilfe der virtuellen Netzschnittstellenkarte (vNIC, Virtual Network Interface Card) für diese Instanz. Jede variable IP-Adresse kann verschiedenen Typen von virtuellen Serverinstanzen (VSIs) zugeordnet werden, z. B. einer Lastausgleichsfunktion (Load Balancer) oder einem VPN-Gateway.

Ihre variable IP-Adresse kann nicht mehreren Schnittstellen zugeordnet werden.Sie müssen die Schnittstelle auf der virtuellen Serverinstanz (VSI) angeben, die dieser einzelnen variablen IP-Adresse zugeordnet werden soll. Diese Schnittstelle verfügt ebenfalls über eine private IP-Adresse. Das Back-End-System führt Operationen für die _Netzadressumsetzung (NAT, Network Address Translation) vom Typ 'Eins-zu-eins'_ zwischen der variablen IP-Adresse und der privaten IP-Adresse dieser Schnittstelle aus. Eine variable IP-Adresse wird nur dann freigegeben, wenn Sie die Freigabeaktion explizit anfordern.

**Hinweise:**
* **Durch die Zuordnung einer variablen IP-Adresse zu einer virtuellen Serverinstanz (VSI) wird die VSI aus der zuvor erwähnten Netzadressumsetzung (NAT) vom Typ 'Viele-zu-eins' des öffentlichen Gateways entfernt.**
* **Gegenwärtig werden nur IPv4-Adressen als variable IP-Adressen unterstützt.**
* **Sie können Ihre eigene öffentliche IP-Adresse nicht als variable IP-Adresse verwenden.**

Weitere Informationen zu NAT-Operationen können Sie [im zugehörigen RFC-Dokument im Internet ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://www.faqs.org/rfcs/rfc1631.html){: new_window} nachlesen.

### VPN für sichere externe Konnektivität verwenden
Der VPN-Service ermöglicht Benutzern, sich sicher aus dem Internet mit ihrer IBM Cloud VPC-Instanz zu verbinden.

**VPN-Funktionalität**
  * CRUD-Fähigkeit (Create, Read, Update, and Delete - Erstellen, Lesen, Aktualisieren und Löschen) des VPN-Service (IPsec-VPN für Site-to-Site-Konnektivität) für die Datenübertragung.
  * Fähigkeit, einen VPN-Service zu der IBM Cloud VPC-Instanz oder dem virtuellen Netz eines Kunden zuzuordnen.
  * Fähigkeit, die lokalen Teilnetze des Kunden entweder durch statische Definition oder durch dynamisches Routing zu identifizieren.
  * Unterstützung von sicheren Verschlüsselungscodes wie SHA256, AES, 3DES oder IKEv2.
  * Zuverlässigkeit des integrierten Service.
  * Fortlaufende Überwachung des Status der VPN-Verbindung.
  * Unterstützung des Initiator- wie auch des Respondermodus, was bedeutet, dass der Datenverkehr von beiden Seiten des Tunnels eingeleitet werden kann. 

Eine Liste der bekannten Einschränkungen und der Features, die derzeit nicht unterstützt, enthält das Dokument [Bekannte Einschränkungen](/docs/infrastructure/vpc?topic=vpc-known-limitations).

## Weitere Informationen

   * [Sicherheit bei IBM VPC](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [IBM VPC-Adressen, Regionen und Teilnetze](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
