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
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Informazioni sulla rete per VPC
{: #about-networking-for-vpc}

In questo documento, troverai i concetti che descrivono i casi di utilizzo e le funzionalità in {{site.data.keyword.cloud}} VPC per le sottoreti, gli indirizzi IP mobili e le connessioni VPN.

**Figura: puoi suddividere un Virtual Private Cloud con le sottoreti e ognuna di esse può essere raggiunta da internet pubblicamente, se desiderato.**

![Connettività VPC](/images/vpc-connectivity-and-security.png)

Come viene mostrato nell'immagine:

* Le sottoreti possono collegarsi a internet pubblicamente tramite un gateway pubblico (PGW) facoltativo.
* Puoi assegnare un indirizzo IP mobile (FIP) ad ogni VSI per raggiungerla da internet o viceversa.
* Le sottoreti in IBM Cloud VPC offrono la connettività privata; possono comunicare tra loro con un link privato. Non devi configurare alcun instradamento.
* Per ulteriori informazioni, consulta le nostre [Informazioni sull'infrastruttura VPC](docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure).

## Terminologia

Questo [glossario](/docs/infrastructure/vpc?topic=vpc-vpc-glossary) contiene le definizioni e le informazioni sui termini utilizzati in questo documento di IBM Cloud VPC.

## Caratteristiche delle sottoreti nel VPC

Una sottorete è formata da un intervallo di indirizzi IP specificati (blocco CIDR). Le sottoreti sono associate a una sola zona e non possono essere suddivise su più zone o regioni. Tuttavia, una sottorete può estendersi sulla totalità delle astrazioni della zona all'interno del proprio Virtual Private Cloud. Le sottoreti nello stesso IBM Cloud VPC sono collegate tra loro.

### Zone

Una zona è un'astrazione progettata per fornire assistenza con una tolleranza dell'errore migliorata e una latenza ridotta. Una zona garantisce le seguenti proprietà:

 * Ogni zona è un dominio di errore indipendente ed è estremamente improbabile che due zone in una regione abbiamo contemporaneamente un malfunzionamento
 * Il traffico tra le zone in una regione sarà < 2ms di latenza

### Indirizzi IP riservati

Alcuni indirizzi IP sono riservati per l'utilizzo da parte di IBM quando utilizza il Virtual Private Cloud. Di seguito vengono riportati gli indirizzi riservati (gli indirizzi IP forniti presumono che l'intervallo CIDR della sottorete sia 10.10.10.0/24):

  * Primo indirizzo nell'intervallo CIDR (10.10.10.0): indirizzo della rete
  * Secondo indirizzo nell'intervallo CIDR (10.10.10.1): indirizzo del gateway
  * Terzo indirizzo nell'intervallo CIDR (10.10.10.2): riservato da IBM
  * Quarto indirizzo nell'intervallo CIDR (10.10.10.3): riservato da IBM per un utilizzo futuro
  * Ultimo indirizzo nell'intervallo CIDR (10.10.10.255): indirizzo broadcast di rete

### Utilizza un gateway pubblico per la connettività esterna di una sottorete

Un **gateway pubblico (PGW)** abilita il collegamento di una sottorete (con tutte le VSI collegate alla sottorete) a internet. Tieni presente che le sottoreti sono private per impostazione predefinita; tuttavia, facoltativamente, puoi creare un PGW e collegargli una sottorete. Dopo aver collegato una sottorete al PGW, tutte le VSI in tale sottorete possono collegarsi a internet.

Il PGW utilizza _Many-to-1 NAT_, il che significa che migliaia di VSI con indirizzi privati utilizzeranno 1 indirizzo IP pubblico per comunicare con internet pubblicamente.Il PGW non consente a internet di avviare una connessione con tali istanze. Utilizza l'API per collegare e scollegare le sottoreti a/da il tuo PGW.

La seguente figura riepiloga il campo di applicazione dei servizi gateway.

![servizi gateway](images/scope-of-gateway-services.png)

### Utilizza un indirizzo IP mobile per la connettività esterna di una VSI
Gli **indirizzi IP mobili** sono indirizzi IP forniti dal sistema e raggiungibili da internet pubblicamente.

Puoi riservare un indirizzo IP mobile dal pool di indirizzi IP mobili disponibili forniti da IBM e puoi associarlo o annullarne l'associazione a/da ogni istanza nello stesso Virtual Private Cloud, tramite la vNIC di tale istanza. Ogni indirizzo IP mobile può essere associato a vari tipi di istanze del server virtuale (VSI), ad esempio, un programma di bilanciamento del carico o un gateway VPN.

Il tuo indirizzo IP mobile non può essere associato a più interfacce.Devi specificare l'interfaccia sulla VSI che sarà associata a quell'indirizzo IP mobile individuale. Tale interfaccia avrà anche un indirizzo IP privato. Il sistema di backend esegue le operazioni _1-to-1 NAT_ tra l'IP mobile e l'IP privato di tale interfaccia. Un IP mobile viene rilasciato solo quando richiedi specificatamente l'azione di rilascio.

**Note:**
* **L'associazione di un indirizzo IP mobile a una VSI, rimuove la VSI dal Many-to-1 NAT del PGW menzionato precedentemente.**
* **Al momento, l'IP mobile supporta solo indirizzi IPv4.**
* **Non puoi utilizzare il tuo indirizzo IP pubblico come IP mobile.**

Per ulteriori informazioni sulle operazioni NAT, fai riferimento al [documento RFC internet correlato ![Icona link esterno](../../icons/launch-glyph.svg "Icona link esterno")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilizza la VPN per proteggere la connettività esterna
Il servizio VPN (Virtual Private Network) è disponibile per gli utenti per collegare i propri IBM Cloud VPC da internet in modo sicuro. Per istruzioni dettagliate, consulta la nostra [guida dell'interfaccia utente della console IBM](http://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console).

**Funzionalità della VPN**
  * Capacità di eseguire le operazioni CRUD (creare, leggere, aggiornare ed eliminare) per il servizio VPN (VPN IPSEC site-to-site) per gestire il trasferimento di dati.
  * Capacità di associare il servizio VPN all'IBM Cloud VPC o alla rete virtuale di un cliente.
  * Capacità di identificare le sottoreti in loco del cliente tramite la definizione statica o l'instradamento dinamico.
  * Supporto per proteggere le cifrature come ad esempio SHA256, AES, 3DES, IKEv2
  * Affidabilità del servizio integrata.
  * Monitoraggio dell'integrità della connessione VPN.
  * Supporto per le modalità dell'iniziatore e del risponditore; ossia, il traffico può essere avviato da entrambi i lati del tunnel.
