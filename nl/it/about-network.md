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

# Informazioni sulla rete per VPC
{: #about-networking-for-vpc}

Un Virtual Private Cloud (VPC) è una rete virtuale collegata al tuo account cliente. Ti fornisce la sicurezza cloud, con la capacità di eseguire il ridimensionamento in modo dinamico, fornendo un controllo dettagliato sulla tua infrastruttura virtuale e sulla tua segmentazione del traffico di rete.

Questo documento copre alcuni concetti di rete in quanto vengono applicati all'interno della VPC {{site.data.keyword.cloud}}. I casi di utilizzo e le caratteristiche descritti includono:

* regioni
* zone
* indirizzi IP riservati
* gateway pubblici
* interfacce vNIC
* sottoreti
* indirizzi IP mobili
* connessioni VPN

## Panoramica

Per configurare la rete nel tuo VPC:
* Utilizza un gateway pubblico per il traffico internet della sottorete.
* Utilizza un IP mobile per il traffico internet della VSI.
* Utilizza una VPN per proteggere la connettività esterna.

Ricorda:
* Alcuni intervalli CIDR di indirizzi della sottorete sono riservati a IBM.
* Devi creare il tuo VPC prima di creare le sottoreti all'interno di tale VPC.
* Il supporto IPv6 non è disponibile.

Facoltativamente, puoi creare un VPC di accesso classico per la connessione alla tua infrastruttura classica IBM Cloud.
{:note}

Come viene mostrato nella seguente figura:
* Le sottoreti nel tuo VPC possono connettersi a internet pubblicamente tramite un gateway pubblico (PGW) facoltativo.
* Puoi assegnare un indirizzo IP mobile (FIP) ad ogni VSI per raggiungerla da internet o viceversa.
* Le sottoreti all'interno di IBM Cloud VPC offrono la connettività privata; possono comunicare tra loro con un link privato, tramite il router implicito. La configurazione degli instradamenti non è necessaria.


**Figura: un cliente può suddividere un Virtual Private Cloud con le sottoreti e ognuna di esse può essere raggiunta da internet pubblicamente, se desiderato.**

![Connettività VPC](/images/vpc-connectivity-and-security.png)

## Terminologia

Questo [glossario](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary) contiene le definizioni e le informazioni sui termini utilizzati in questo documento di IBM Cloud VPC. Quando utilizzi il tuo VPC, dovrai avere familiarità con i concetti di base di _regione_ e _zona_ in quanto vengono applicati alla tua distribuzione.

### Regioni

Una regione è un'astrazione correlata all'area geografica in cui viene distribuito un VPC. Ogni regione contiene più zone, che rappresentano dei domini di errore indipendenti. Un IBM Cloud VPC può essere suddiviso su più zone all'interno della sua regione assegnata.

### Zone

Una zona è un'astrazione che fa riferimento al data center fisico che ospita le risorse di calcolo, rete e archiviazione, nonché il raffreddamento e l'alimentazione correlati, che fornisce i servizi e le applicazioni. Le zone sono isolate tra loro, per cui per non creare alcun singolo punto di errore condiviso, è stata migliorata la tolleranza e diminuita la latenza. Una zona garantisce le seguenti proprietà:

 * Ogni zona è un dominio di errore indipendente ed è estremamente improbabile che due zone in una regione abbiamo contemporaneamente un malfunzionamento.
 * Il traffico tra le zone in una regione avrà meno di 2ms di latenza.

## Caratteristiche delle sottoreti nel VPC

Una sottorete è formata da un intervallo di indirizzi IP specificati (blocco CIDR). Le sottoreti sono associate a una sola zona e non possono essere suddivise su più zone o regioni. Tuttavia, una sottorete può estendersi sulla totalità delle astrazioni della zona all'interno del proprio Virtual Private Cloud. Le sottoreti nello stesso IBM Cloud VPC sono collegate tra loro.


### Indirizzi IP riservati

Alcuni indirizzi IP sono riservati per l'utilizzo da parte di IBM quando utilizza il Virtual Private Cloud. Di seguito vengono riportati gli indirizzi riservati (questi indirizzi IP presumono che l'intervallo CIDR della sottorete sia 10.10.10.0/24):

  * Primo indirizzo nell'intervallo CIDR (10.10.10.0): indirizzo della rete
  * Secondo indirizzo nell'intervallo CIDR (10.10.10.1): indirizzo del gateway
  * Terzo indirizzo nell'intervallo CIDR (10.10.10.2): riservato da IBM
  * Quarto indirizzo nell'intervallo CIDR (10.10.10.3): riservato da IBM per un utilizzo futuro
  * Ultimo indirizzo nell'intervallo CIDR (10.10.10.255): indirizzo broadcast di rete

### Utilizza un gateway pubblico per la connettività esterna di una sottorete

Un **gateway pubblico (PGW)** abilita il collegamento di una sottorete (con tutte le VSI collegate alla sottorete) a internet. Tieni presente che le sottoreti sono private per impostazione predefinita; tuttavia, facoltativamente, puoi creare un PGW e collegargli una sottorete. Dopo aver collegato una sottorete al PGW, tutte le VSI in tale sottorete possono collegarsi a internet.

Il PGW utilizza _Many-to-1 NAT_, il che significa che migliaia di VSI con indirizzi privati utilizzeranno 1 indirizzo IP pubblico per comunicare con internet pubblicamente.Il PGW non consente a internet di avviare una connessione con tali istanze. Utilizza l'API per collegare e scollegare le sottoreti a/da il tuo PGW.

La seguente figura riepiloga l'attuale campo di applicazione dei servizi gateway.

![servizi gateway](images/scope-of-gateway-services.png)

Un gateway pubblico viene creato in un VPC, ma non fa nulla finché non viene collegato a una sottorete. Puoi creare solo un gateway pubblico per zona, che significa, ad esempio, che potresti avere 3 gateway pubblici per VPC in un ambiente con 3 zone.
{:note}

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
Il servizio VPN (Virtual Private Network) è disponibile per gli utenti per collegare i propri IBM Cloud VPC da internet in modo sicuro.

**Funzionalità della VPN**
  * Capacità di eseguire le operazioni CRUD (creare, leggere, aggiornare ed eliminare) per il servizio VPN (VPN IPSEC site-to-site) per gestire il trasferimento di dati.
  * Capacità di associare il servizio VPN all'IBM Cloud VPC o alla rete virtuale di un cliente.
  * Capacità di identificare le sottoreti in loco del cliente tramite la definizione statica o l'instradamento dinamico.
  * Supporto per proteggere le cifrature come ad esempio SHA256, AES, 3DES, IKEv2
  * Affidabilità del servizio integrata.
  * Monitoraggio continuo dell'integrità della connessione VPN.
  * Supporto per le modalità dell'iniziatore e del risponditore; ossia, il traffico può essere avviato da entrambi i lati del tunnel.

Per un elenco di funzioni e limitazioni note non al momento supportate, fai riferimento al documento [Limitazioni note](/docs/infrastructure/vpc?topic=vpc-known-limitations).

## Ulteriori informazioni

   * [Sicurezza di IBM VPC](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [Indirizzi, regioni e sottoreti di IBM VPC](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
