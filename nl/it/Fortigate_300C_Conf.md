---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-01-23"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creazione di una connessione sicura con un peer FortiGate remoto

Questo documento si basa su FortiGate 300C, versione firmware	v5.2.13,build762 (GA).

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API {{site.data.keyword.cloud}} per creare i VPC (Virtual Private Cloud). Per ulteriori informazioni, consulta [Introduzione](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) e [Configurazione di VPC con le API](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Istruzioni di esempio
La topologia per la connessione al peer FortiGate remoto è simile alla creazione di una connessione VPN tra due VPC. Tuttavia, un lato viene sostituito dall'unità FortiGate 300C.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-figure.png)

### Per creare una connessione sicura con il peer Fortigate remoto

Vai a **VPN \> IPsec \> Tunnels** e crea un nuovo tunnel personalizzato o modificane uno esistente.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

Quando un'unità FortiGate riceve una richiesta di connessione da un peer VPN remoto, utilizza i parametri IPsec Phase 1 per stabilire una connessione sicura e autenticare tale peer VPN. Successivamente, se la politica di sicurezza consente la connessione, l'unità FortiGate stabilisce il tunnel utilizzando i parametri IPsec Phase e applica la politica di sicurezza IPsec. I servizi di sicurezza, autenticazione e di gestione della chiave vengono negoziati in modo dinamico tramite il protocollo IKE.

**Per supportare queste funzioni, devono essere eseguiti i seguenti passi di configurazione generale dall'unità FortiGate:**

* Definisci i parametri Phase 1 necessari all'unità FortiGate per autenticare il peer remoto e stabilire una connessione sicura.

* Definisci i parametri Phase 2 necessari all'unità FortiGate per creare un tunnel VPN con il peer remoto.

* Crea le politiche di sicurezza per controllare i servizi e la direzione del traffico consentiti tra gli indirizzi di destinazione e di origine IP.

Per impostazione predefinita, sono stati impostati alcuni parametri Phase 1 e Phase 2 tipici.

### Configurazione della VPN per IBM Cloud VPC

Per il collegamento alla funzionalità VPN di IBM Cloud VPC, consigliamo la seguente configurazione:

1. Scegli IKEv2 nell'autenticazione;
2. Abilita `DH-group 2` nella proposta Phase 1
3. Imposta `lifetime = 36000` nella proposta Phase 1
4. Disabilita PFS nella proposta Phase 2
5. Imposta `lifetime = 10800` nella proposta Phase 2
6. Immetti le informazioni della tua sottorete nella Phase 2
7. I parametri rimanenti conservano i propri valori predefiniti.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-network.JPG)

### Parametri di rete

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-authentication.JPG)

### Parametri di autenticazione

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-phase1.JPG)

### Parametri Phase 1

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-phase2.JPG)

### Parametri Phase 2

## Per creare una connessione sicura con l'IBM VPC locale

Per creare una connessione sicura, creerai la connessione VPN all'interno del tuo VPC, che è simile all'esempio dei 2 VPC.

* Crea un gateway VPN sulla tua sottorete VPC insieme a una connessione VPN tra il VPC e l'unità FortiGate, impostando `local_cidrs` come valore della sottorete nel VPC e `peer_cidrs` come valore della sottorete nel FortiGate.

**Nota:** lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-fg-connection.png)

### Controlla lo stato della connessione sicura

Puoi controllare lo stato della tua connessione tramite la console IBM Cloud. Inoltre, puoi provare ad eseguire il `ping` tra i siti utilizzando le VSI.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-fg-status.JPG)
