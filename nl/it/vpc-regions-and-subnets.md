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

# Utilizzo degli intervalli di indirizzi IP, dei prefissi di indirizzo, delle regioni e delle sottoreti 
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}

Questo documento di fornisce una panoramica degli intervalli di indirizzi IP disponibili (o limitati) e delle azioni utente relative alle regioni e alle sottoreti e ai prefissi di indirizzo. Alcuni intervalli di indirizzi IP non sono disponibili perché è possibile che IBM Cloud li stia utilizzando.   

Quando crei una sottorete nel tuo {{site.data.keyword.cloud}} VPC, viene automaticamente assegnato un _prefisso di indirizzo_, basato sulla regione e la zona in cui hai creato il tuo VPC. Puoi utilizzare il prefisso di indirizzo predefinito o puoi modificare manualmente il tuo prefisso di indirizzo, incluso un indirizzo BYOIP, nella maggior parte dei casi. 

## IBM Cloud VPC e regioni

Un {{site.data.keyword.cloud}} VPC viene distribuito in una regione, ma può essere suddiviso in più zone. Ogni regione contiene più zone, che rappresentano dei domini di errore indipendenti.

Alle sottoreti in un VPC viene assegnato un _prefisso di indirizzo_, in base alla regione e alla zona in cui vengono create. La seguente tabella mostra le regioni e le zone disponibili. 

|   Ubicazione     | Regione | Endpoint API | Stato |
| ------- | :------: | :------: |:------: |
| Dallas | us-south | `us-south.iaas.cloud.ibm.com`| Disponibile |
| Francoforte | eu-de | `eu-de.iaas.cloud.ibm.com`| Disponibile |
| Washington DC | us-east | `us-east.iaas.cloud.ibm.com`| Prossimamente |
| Tokyo | jp-tok | `jp-tok.iaas.cloud.ibm.com`| Prossimamente |
| Londra | eu-gb | `eu-gb.iaas.cloud.ibm.com`| Prossimamente |
| Sydney | au-syd | `au-syd.iaas.cloud.ibm.com`| Prossimamente |

## IBM Cloud VPC e sottoreti

I clienti possono (e normalmente lo fanno, come procedura consigliata) dividere un IBM Cloud VPC in sottoreti.Tutte le sottoreti in un IBM Cloud VPC possono raggiungerne un altro tramite l'instradamento L3 privato da un router implicito.Non devi configurare alcun instradamento o router. 

Informazioni utili sulle sottoreti nel VPC:

* Una sottorete è formata da un intervallo di indirizzi IP da te specificato. 
* Una sottorete è associata a una sola zona, che non può essere suddivisa in più zone e regioni.
* Una sottorete può estendersi sulla totalità della zona in un IBM Cloud VPC. 
* Devi creare il tuo VPC prima di creare le tue sottoreti all'interno di tale VPC.
* Il supporto IPv6 non è disponibile.
* Puoi associare o annullare l'associazione di una VSI a/da una sottorete. (Devi aggiungere un vNIC e selezionare una larghezza di banda).

Puoi utilizzare il tuo intervallo di indirizzi IPv4 pubblici (BYOIP) per il tuo account IBM Cloud VPC. Quando utilizzi BYOIP, IBM Cloud deve configurare tali indirizzi IPv4 sulle risorse IBM Cloud, che invieranno i pacchetti da e verso gli indirizzi forniti. Pertanto, in seguito all'utilizzo del tuo intervallo IPv4 fornito su IBM Cloud, tali indirizzi IP possono essere esposti a terze parti e allo staff di supporto di IBM come parte del tuo utilizzo di questo servizio.
{:important}

**Figura: una rappresentazione grafica di un VPC che mostra zone, regioni e sottoreti.**

![IBM Cloud VPC](images/vpc-experience.png)

## Utilizzo dei prefissi di indirizzo per le sottoreti

Ogni sottorete deve essere all'interno di un prefisso di indirizzo.
 * I prefissi dell'indirizzo della sottorete vengono preassegnati per ogni zona.
 * Per la tua nuova sottorete, puoi selezionare il tuo intervallo di indirizzi IP dai prefissi dell'indirizzo esistenti.
 * Se i prefissi dell'indirizzo della zona non sono adatti, può essere modificato uno dei prefissi dell'indirizzo esistenti o ne può essere aggiunto uno nuovo per la zona (utilizzando l'API, la CLI o l'IU).
 
### Prefissi dell'indirizzo VPC predefiniti
 
Quando viene creato un nuovo VPC, i prefissi dell'indirizzo predefiniti vengono assegnati nel seguente modo, in base alla regione e alla zona:
 
* 'us-south-1' = '10.240.0.0/18'
* 'us-south-2' = '10.240.64.0/18'

Altre zone o regioni possono assegnare prefissi predefiniti differenti, inclusi i seguenti:
 
 * 10.240.0.0/18
 * 10.240.64.0/18
 * 10.240.128.0/18
 
### Prefissi dell'indirizzo e IU della console IBM Cloud
 
Quando crei un VPC utilizzando l'IU della console IBM Cloud, il sistema seleziona il tuo prefisso di indirizzo automaticamente e ti richiede di creare una sottorete all'interno di tale prefisso predefinito. Se questo schema di indirizzi non si adatta ai tuoi requisiti, puoi personalizzare i prefissi dell'indirizzo dopo aver creato il VPC. Successivamente, puoi creare delle sottoreti nei tuoi prefissi dell'indirizzo personalizzati ed eliminare la sottorete che hai creato con il prefisso predefinito.
 
Questa soluzione temporanea è necessaria per utilizzare BYOIP tramite l'IU della console IBM Cloud.
{:note}

### Indirizzi IP disponibili

Questi sono gli indirizzi IP disponibili, come definito in **RFC 1918**:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

### I seguenti indirizzi sono riservati in una sottorete

In questo elenco, gli indirizzi IP presumono che l'intervallo CIDR della sottorete sia 10.10.10.0/24:

  * Primo indirizzo nell'intervallo CIDR (10.10.10.0): indirizzo della rete
  * Secondo indirizzo nell'intervallo CIDR (10.10.10.1): indirizzo del gateway
  * Terzo indirizzo nell'intervallo CIDR (10.10.10.2): riservato da IBM
  * Quarto indirizzo nell'intervallo CIDR (10.10.10.3): riservato da IBM per un utilizzo futuro
  * Ultimo indirizzo nell'intervallo CIDR (10.10.10.255): indirizzo broadcast di rete

### Ulteriori informazioni sulla creazione di una sottorete

Esistono due modi per specificare una sottorete per il tuo VPC:
  * Puoi creare una sottorete fornendo la dimensione della sottorete di cui hai bisogno, come ad esempio il numero di indirizzi supportati (ad esempio, 1024)
  * Puoi creare una sottorete fornendo un intervallo CIDR (come ad esempio 10.0.0.1/29)
  
Esempio di come specificare un blocco di 1024 utilizzando CIDR: se stai specificando un intervallo CIDR invece di una dimensione della sottorete, il blocco IPv4 `192.168.100.0/22` rappresenta i 1024 indirizzi IPv4 da `192.168.100.0` a `192.168.103.255`.
{:tip}

### Ulteriori informazioni sull'eliminazione di una sottorete
  * Non puoi eliminare una sottorete se sono in utilizzo delle risorse (come ad esempio una VSI (Virtual Server Instance), un IP mobile o un gateway pubblico) in tale sottorete, devono prima essere eliminate le risorse.
  * NON puoi ridimensionare una sottorete esistente. Ad esempio, 10.10.10.0/24 non può essere ridimensionata con 10.5.1.3/20
