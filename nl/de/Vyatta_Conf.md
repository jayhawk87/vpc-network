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


# Sichere Verbindung mit einem fernen Vyatta-Peer erstellen
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

Dieses Dokument basiert auf Vyatta, Version AT&T vRouter 5600 1801d.

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der {{site.data.keyword.cloud}}-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) und in [VPC-Einrichtung mit APIs](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Beispielschritte
Die Topologie für die Verbindungsherstellung zum fernen Vyatta-Peer hat Ähnlichkeit mit der Erstellung einer VPN-Verbindung zwischen zwei {{site.data.keyword.cloud}}-VPCs. Eine Seite wird jedoch durch die Vyatta-Einheit ersetzt.

![Abbildungsbeschreibung](images/vpc-vpn-vy-figure.png)

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit dem fernen Vyatta-Peer

Wenn ein VPN-Peer eine Verbindungsanforderung von einem fernen VPN-Peer empfängt, verwendet sie IPsec-Parameter für Phase 1, um eine sichere Verbindung herzustellen und diesen VPN-Peer zu authentifizieren. Wenn die Sicherheitsrichtlinie die Verbindung zulässt, richtet die Vyatta-Einheit dann unter Verwendung der IPsec-Parameter für Phase 2 den Tunnel ein und wendet die IPsec-Sicherheitsrichtlinie an. Schlüsselmanagement, Authentifizierung und Sicherheitsservices werden dynamisch über das IKE-Protokoll verhandelt.

Wahlweise können Sie die Befehlszeile von Vyatta öffnen, um den IPsec-Tunnel zu konfigurieren, oder eine Datei mit der Erweiterung `*.vcli` hochladen, um Ihre Konfiguration zu laden.

**Damit diese Funktionen unterstützt werden, müssen die folgenden allgemeinen Konfigurationsschritte von der Vyatta-Einheit ausgeführt werden:**

* Definieren der Parameter für Phase 1, die die Vyatta-Einheit benötigt, um den fernen Peer zu authentifizieren und eine sichere Verbindung herzustellen.

* Definieren der Parameter von Phase 2, die die Vyatta-Einheit benötigt, um einen VPN-Tunnel mit dem fernen Peer zu erstellen.

Um eine Verbindung mit der VPN-Funktionalität von IBM Cloud VPC herzustellen, wird die folgende Konfiguration empfohlen:

1. Wählen Sie im Schritt für die Authentifizierung `IKEv2` aus.
2. Aktivieren Sie im Vorschlag für Phase 1 den Eintrag `DH-group 2`.
3. Legen Sie im Vorschlag für Phase 1 den Wert `lifetime = 36000` fest.
4. Inaktivieren Sie im Vorschlag für Phase 2 den Wert 'PFS'.
5. Legen Sie im Vorschlag für Phase 2 den Wert `lifetime = 10800` fest.
6. Geben Sie beim Vorschlag für Phase 2 die Informationen für Ihren Peer und Ihr Teilnetz ein.

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```

Dann können Sie diese Datei mit der Erweiterung `*.vcli` mit SCP auf die Vyatta-Einheit hochladen, damit die Konfiguration Anwendung findet.

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit der lokalen IBM Cloud-VPC

 Zum Erstellen einer sicheren Verbindung erstellen Sie die VPN-Verbindung innerhalb Ihrer VPC, was dem Beispiel mit den 2 VPCs ähnelt.

* Erstellen Sie ein VPN-Gateway auf Ihrem VPC-Teilnetz sowie eine VPN-Verbindung zwischen der VPC und der Vyatta-Einheit. Legen Sie dabei für `local_cidrs` den Wert für das Teilnetz auf der VPC und für `peer_cidrs` den Wert für das Teilnetz auf der Vyatta-Einheit fest.

**Hinweis:** Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen.

![Abbildungsbeschreibung](images/vpc-vpn-vy-connection.png)

### Status der sicheren Verbindung prüfen

Sie können den Status Ihrer Verbindung über die IBM Cloud-Konsole prüfen. Außerdem können Sie auch versuchen, von Site zu Site ein `Pingsignal` mit den VSIs abzusetzen.

![Abbildungsbeschreibung](images/vpc-vpn-vy-status.png)
