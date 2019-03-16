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


# Creazione di una connessione sicura con un peer Vyatta remoto
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

Questo documento si basa su Vyatta versione: AT&T vRouter 5600 1801d.

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API {{site.data.keyword.cloud}} per creare i VPC (Virtual Private Cloud). Per ulteriori informazioni, consulta [Introduzione](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) e [Configurazione di VPC con le API](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Istruzioni di esempio
La topologia per la connessione al peer Vyatta remoto è simile alla creazione di una connessione VPN tra due {{site.data.keyword.cloud}} VPC. Tuttavia, un lato viene sostituito dall'unità Vyatta.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-vy-figure.png)

### Per creare una connessione sicura con il peer Vyatta remoto

Quando un peer VPN riceve una richiesta di connessione da un peer VPN remoto, utilizza i parametri IPsec Phase 1 per stabilire una connessione sicura e autenticare tale peer VPN. Successivamente, se la politica di sicurezza consente la connessione, l'unità Vyatta stabilisce il tunnel utilizzando i parametri IPsec Phase 2 e applica la politica di sicurezza IPsec. I servizi di sicurezza, autenticazione e di gestione della chiave vengono negoziati in modo dinamico tramite il protocollo IKE.

Puoi andare alla riga di comando di Vyatta per configurare il tunnel IPsec o puoi caricare un file `*.vcli` per caricare la tua configurazione.

**Per supportare queste funzioni, devono essere eseguiti i seguenti passi di configurazione generale dall'unità Vyatta:**

* Definisci i parametri Phase 1 necessari all'unità Vyatta per autenticare il peer remoto e stabilire una connessione sicura.

* Definisci i parametri Phase 2 necessari all'unità Vyatta per creare un tunnel VPN con il peer remoto.

Per il collegamento alla funzionalità VPN di IBM Cloud VPC, consigliamo la seguente configurazione:

1. Scegli `IKEv2` nell'autenticazione;
2. Abilita `DH-group 2` nella proposta Phase 1
3. Imposta `lifetime = 36000` nella proposta Phase 1
4. Disabilita PFS nella proposta Phase 2
5. Imposta `lifetime = 10800` nella proposta Phase 2
6. Immetti le informazioni sui tuoi peer e sottoreti nella Phase 2

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

Successivamente, puoi caricare questo file `*.vcli` in Vyatta utilizzando SCP, per applicare la configurazione.

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### Per creare una connessione sicura con l'IBM Cloud VPC locale

 Per creare una connessione sicura, creerai la connessione VPN all'interno del tuo VPC, che è simile all'esempio dei 2 VPC.

* Crea un gateway VPN sulla tua sottorete VPC insieme a una connessione VPN tra il VPC e l'unità Vyatta, impostando `local_cidrs` come valore della sottorete nel VPC e `peer_cidrs` come valore della sottorete nell'unità Vyatta.

**Nota:** lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-vy-connection.png)

### Controlla lo stato della connessione sicura

Puoi controllare lo stato della tua connessione tramite la console IBM Cloud. Inoltre, puoi provare ad eseguire il `ping` tra i siti utilizzando le VSI.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-vy-status.png)
