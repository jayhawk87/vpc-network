---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-11"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Einführung in den Netzbetrieb für Virtual Private Cloud
{: #getting-started-with-networking-for-virtual-private-cloud}


IBM wird ab Anfang April 2019 eine begrenzte Anzahl von Kunden zur Teilnahme an einem Programm für VPC mit Vorabzugriff annehmen, wobei die erweiterte Nutzung für die nachfolgenden Monate vorgesehen ist. Wenn Ihr Unternehmen Zugriff auf IBM Virtual Private Cloud erhalten möchte, füllen Sie bitte dieses [Nominierungsformular](https://cloud.ibm.com/vpc){: new_window} aus, damit sich ein IBM Ansprechpartner hinsichtlich der weiteren Vorgehensweise mit Ihnen in Verbindung setzen kann.
{: important}

Führen Sie zum Einstieg in den {{site.data.keyword.cloud}}-Netzbetrieb für Virtual Private Cloud

1. Erstellen Sie eine virtuelle private Cloud.
2. Erstellen Sie in dieser virtuellen privaten Cloud in einer oder mehreren Zonen ein oder mehrere Teilnetze.
3. Erstellen Sie ein öffentliches Gateway (PGW, Public Gateway) in einem Teilnetz, wenn Ressourcen in dem Teilnetz Zugriff auf das Internet erhalten sollen oder umgekehrt.

## Voraussetzungen

 * **Benutzerberechtigungen**: Stellen Sie sicher, dass Ihr Benutzer über ausreichende Berechtigungen zum Erstellen und Verwalten von Ressourcen in Ihrer VPC verfügt. Eine Liste der erforderlichen Berechtigungen finden Sie in [Für Benutzer von VPC benötigte Berechtigungen erteilen](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).

## Verwenden der UI, CLI oder REST-API zur Bereitstellung von VPC-Netzressourcen

Die Einrichtung, Bereitstellung und Verwaltung von VPC-Netzressourcen kann über die UI (User Interface, Benutzerschnittstelle), die CLI (Command Line Interface, Befehlszeilenschnittstelle) oder die REST-API (Application Programming Interface, Anwendungsprogrammierschnittstelle) vorgenommen werden. 

* Um Zugriff über die Benutzerschnittstelle (UI) zu erhalten, melden Sie sich an der [IBM Cloud-Konsole ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")]( https://{DomainName}/vpc){: new_window} an. Folgen Sie den Anweisungen im [Handbuch zur UI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), falls Sie Hilfe benötigen. 
* Wenn Sie die Befehlszeilenschnittstelle (CLI) verwenden möchten, verwenden Sie das Plug-in '[infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html)' der [IBM Cloud-CLI](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) und folgen Sie dem Beispiel für [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli). 
* Fortgeschrittene Benutzer können die [REST-APIs](https://{DomainName}/apidocs/rias) direkt aufrufen. Gehen Sie gemäß dem Lernprogramm für [Beispielcode](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) vor, um den Einstieg in das Arbeiten mit REST-APIs zu finden. 

## Nächste Schritte

Nachdem der Netzbetrieb für Ihre virtuelle private Cloud eingerichtet worden ist, können Sie weitere Aspekte erkunden. 

* [Virtuelle Serverinstanzen (VSIs) erstellen und verwalten](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [Sicherheitsgruppen verwenden](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Netz-ACLs verwenden](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(Beta) VPN verwenden](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(Beta) Lastausgleichsfunktionen (Load Balancer) verwenden](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
