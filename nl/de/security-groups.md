---

copyright:
  years: 2019

lastupdated: "2019-02-20"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Sicherheitsgruppen verwenden
{: #using-security-groups}

Sicherheitsgruppen bieten eine bequeme Möglichkeit, Regeln anzuwenden, die auf der Grundlage der IP-Adresse eine Filterung für jede Netzschnittstelle einer virtuellen Serverinstanz (VSI) festlegen. Wenn Sie eine neue Sicherheitsgruppenressource erstellen, ist es erforderlich, sie entsprechend zu aktualisieren, um die gewünschten Netzverkehrsmuster zu erstellen. 

Standardmäßig verweigert eine Sicherheitsgruppe grundsätzlich jeglichen Datenverkehr. Durch Hinzufügen von Regeln zu einer Sicherheitsgruppe wird zunehmend definiert, welchen Datenverkehr die Sicherheitsgruppe zulässt.

Regeln sind _statusabhängig_. Das bedeutet, dass der Datenverkehr in umgekehrter Richtung als Antwort auf zulässigen Datenverkehr automatisch zulässig ist. Eine Regel, die eingehenden TCP-Datenverkehr an Port 80 zulässt, lässt zum Beispiel auch abgehenden TCP-Datenverkehr an Port 80 als Antwort an den Ursprungshost zu, ohne dass hierfür eine zusätzliche Regel erforderlich ist.

Der Gültigkeitsbereich von Sicherheitsgruppen ist auf eine einzelne VPC beschränkt. Diese Einschränkung des Gültigkeitsbereichs impliziert, dass eine Sicherheitsgruppe _nur_ an Netzschnittstellen von virtuellen Serverinstanzen innerhalb derselben virtuellen privaten Cloud (VPC) angehängt werden kann.

Wenn eine virtuelle Serverinstanz (VSI) erstellt wird, ohne dass hierbei Sicherheitsgruppen angegeben werden, wird die primäre Netzschnittstelle der VSI in die _standardmäßige_ Sicherheitsgruppe der VPC dieser virtuellen Serverinstanz platziert. Bei diesem Release von {{site.data.keyword.cloud}} VPC ist eine Standardsicherheitsgruppe definiert, die bestimmten Datenverkehr zulässt. Weitere Informationen enthält [Standardsicherheitsgruppe aktualisieren](/docs/infrastructure/vpc-network?topic=vpc-network-updating-the-default-security-group). 

Sicherheitsgruppen können über die REST-API, die CLI oder die UI eingerichtet werden: 

* [Sicherheitsgruppen über die API einrichten](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-apis)
* [Sicherheitsgruppen über die CLI einrichten](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Sicherheitsgruppen über die UI einrichten](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
