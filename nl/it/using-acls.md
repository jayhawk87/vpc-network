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
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Configurazione degli ACL di rete utilizzando la CLI 
{ #setting-up-network-acls-using-the-cli}

Utilizzando la funzionalità dell'elenco di controllo degli accessi (ACL) disponibile in {{site.data.keyword.cloud}} Virtual Private Cloud, puoi controllare tutto il traffico in entrata e in uscita relativo ai tuoi carichi di lavoro di business critici sul cloud. Simile al gruppo di sicurezza, un ACL è un firewall virtuale integrato. A differenza dei gruppi di sicurezza, le regole dell'ACL controllano il traffico da e verso le sottoreti, invece che da e verso le istanze.

La seguente tabella riepiloga alcune differenze chiave tra i gruppi di sicurezza e gli ACL:

|  | Gruppi di sicurezza | ACL    |
|-------------|-----------------|---------|
| Livello di controllo  | Istanza VSI    | Sottorete  |
| Stato   | Con stato - La restituzione del traffico è consentita per impostazione predefinita | Senza stato - La restituzione del traffico viene negata per impostazione predefinita e deve essere esplicitamente consentita |
| Regole supportate | Utilizza solo le regole di assenso | Utilizza sia le regole di assenso che di negazione|
| Modalità di applicazione delle regole | Vengono considerate tutte le regole | Le regole vengono elaborate in sequenza |
| Relazione con la risorsa associata | Un'istanza può essere associata a più gruppi di sicurezza | Più sottoreti possono essere associate allo stesso ACL |

L'esempio fornito in questo documento illustra come creare gli ACL di rete nel tuo VPC, che proteggeranno le tue sottoreti.

## Sono disponibili delle API e CLI per gli ACL

Puoi configurare e gestire gli ACL tramite l'API o la CLI, come preferisci.

* Consulta la [Guida di riferimento API](https://{DomainName}/apidocs/rias) per i parametri, il corpo della richiesta e i dettagli della risposta di ogni API.

* Consulta questo [Esempio CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) per ottenere i dettagli del comando, le istruzioni di installazione del plugin CLI e i passi preliminari dell'utilizzo della CLI VPC.

## Utilizzo degli ACL e delle regole dell'ACL

Per rendere i tuoi ACL effettivi, creerai delle regole che determinano come gestire il tuo traffico di rete in entrata e in uscita. Puoi creare più regole in entrata e in uscita. Consulta [Quote](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-quotas) per informazioni specifiche su quante regole puoi creare.

* Con le regole in entrata e in uscita, puoi consentire o negare il traffico da un intervallo di IP di origine o a un intervallo di IP di destinazione con porte e protocolli specificati.  
* Le regole dell'ACL vengono valutate in sequenza. Le regole con priorità bassa vengono valutate solo quando le regole con priorità più alta non corrispondono.
* Le regole in entrata vengono separate dalle regole in uscita.
* I protocolli al momento supportati sono TCP, UDP e ICMP. Puoi utilizzare anche l'opzione **all** per indicare i protocolli _all_ o _other_ (se viene specificata una regola con una priorità più elevata).

Per informazioni importanti sull'utilizzo dei protocolli ICMP, TCP e UDP nelle tue regole ACL, consulta [Descrizione dei protocolli di comunicazione internet](https://{DomainName}/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols).

### Collegamento di un ACL a una sottorete

Hai due opzioni quando colleghi un ACL a una sottorete:

* Puoi creare una nuova sottorete e specificare un ACL da collegare. Se non specifichi un ACL, viene collegato un ACL di rete predefinito. L'ACL predefinito consente tutto il traffico in entrata destinato a questa sottorete e tutto il traffico in uscita da questa sottorete.
* Puoi collegare un ACL a una sottorete esistente. Se un altro ACL è già collegato a questa sottorete, tale ACL viene scollegato prima che venga collegato il nuovo ACL.

## Esempio di demo ACL

Nel seguente esempio, sarai in grado di creare due ACL e associarli a due sottoreti utilizzando l'interfaccia riga di comando (CLI). Ecco come appare lo scenario:

![Uno scenario ACL di esempio](images/vpc-acls.png)

Come illustrato nella figura, hai due server web che si occupano delle richieste da internet e due server di backend che vuoi nascondere al pubblico. In questo esempio, inserisci i server in due sottoreti separate, 10.10.10.0/24 e 10.10.20.0/24 rispettivamente e devi consentire ai server web di scambiare i dati con i server di backend. Inoltre, vuoi consentire del traffico in uscita limitato dai server di backend.

### Regole di esempio

Le seguenti regole di esempio mostrano come configurare le regole dell'ACL per uno scenario di base come descritto precedentemente.

Come procedura ottimale, ti consigliamo di dare alle regole dettagliate una priorità più alta rispetto alle regole poco dettagliate. Ad esempio, se hai una regola che blocca tutto il traffico dalla sottorete 10.10.30.0/24 e viene abbinata a un'elevata priorità, la regola dettagliata con priorità più bassa per consentire il traffico da 10.10.30.5 non sarà mai applicata.
{:note}

**Regole di esempio ACL-1**:

| In entrata/In uscita| Consenti/Nega | IP di origine/destinazione | Protocollo | Porta | Descrizione|
|--------------|-----------|------|------|------------------|-------|
| In entrata | Consenti | 0.0.0.0/0 | TCP| 80 | Consenti il traffico HTTP da internet|
| In entrata | Consenti | 0.0.0.0/0 | TCP | 443 | Consenti il traffico HTTPS da internet|
| In entrata | Consenti| 10.10.20.0/24 |tutti| tutti| Consenti tutto il traffico in entrata dalla sottorete 10.10.20.0/24 dove si trovano i server di backend|
| In entrata | Nega| 0.0.0.0/0|tutti| tutti| Nega tutto il rimanente traffico in entrata|
| In uscita | Consenti | 0.0.0.0/0 | TCP|80 | Consenti il traffico HTTP a internet|
| In uscita | Consenti | 0.0.0.0/0 | TCP|443 | Consenti il traffico HTTPS a internet|
| In uscita | Consenti| 10.10.20.0/24 |tutti| tutti| Consenti tutto il traffico in uscita alla sottorete 10.10.20.0/24 dove si trovano i server di backend|
| In uscita | Nega| 0.0.0.0/0|tutti| tutti| Nega tutto il rimanente traffico in uscita|


**Regole di esempio ACL-2**:

| In entrata/In uscita| Consenti/Nega | IP di origine/destinazione | Protocollo| Porta | Descrizione|
|--------------|-----------|------|------|------------------|--------|
| In entrata | Consenti| 10.10.10.0/24 |tutti| tutti| Consenti tutto il traffico in entrata dalla sottorete 10.10.10.0/24 dove si trovano i server web|
| In entrata | Nega| 0.0.0.0/0|tutti| tutti| Nega tutto il rimanente traffico in entrata|
| In uscita | Consenti | 0.0.0.0/0 | TCP| 80 | Consenti il traffico HTTP a internet|
| In uscita | Consenti | 0.0.0.0/0 | TCP| 443 | Consenti il traffico HTTPS a internet|
| In uscita | Consenti| 10.10.10.0/24 |tutti| tutti| Consenti tutto il traffico in uscita alla sottorete 10.10.10.0/24 dove si trovano i server web|
| In uscita | Nega| 0.0.0.0/0|tutti| tutti| Nega tutto il rimanente traffico in uscita|

Questo esempio illustra solo i casi generali. Nei tuoi scenari, potresti anche voler avere maggior controllo granulare sul traffico:

* Potresti avere un amministratore di rete che deve accedere alla sottorete 10.10.10.0/24 da una rete remota per motivi di gestione. In questo caso, dovrai consentire il traffico SSH da internet a questa sottorete.
* Potresti voler restringere l'ambito del protocollo che consenti tra due sottoreti.

### Istruzioni di esempio

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI per creare un VPC, operazione che deve essere fatta per prima. Per ulteriori informazioni, consulta [Creazione di un VPC utilizzando la CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).


#### Passo 1. Crea gli ACL

Questi sono i comandi CLI che puoi utilizzare per creare due ACL, denominati `my_web_subnet_acl` e `my_backend_subnet_acl`:

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

La risposta include gli ID degli ACL appena creati. Salva gli ID di entrambi gli ACL per utilizzarli nei comandi successivi. Puoi utilizzare le variabili denominate `webacl` e `bkacl`, nel seguente modo: 

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### Passo 2. Richiama le regole dell'ACL predefinite

Prima di aggiungere nuove regole, richiama le regole dell'ACL in entrata e in uscita predefinite in modo da poter inserire le nuove regole prima di esse.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

La risposta mostra le regole in entrata e in uscita predefinite che consentono tutto il traffico IPv4 in tutti i protocolli.

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

Salva gli ID di entrambe le regole dell'ACL come variabili, li utilizzerai nei comandi successivi. Ad esempio, puoi utilizzare le variabili denominate `inrule` e `outrule`:

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### Passo 3. Aggiungi le nuove regole dell'ACL come descritto

Questa sezione mostra prima l'aggiunta delle regole in entrata e poi di quelle in uscita.

Inserisci le nuove regole in entrata prima della regola in entrata predefinita.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

Ad ogni passo, salva l'ID della regola dell'ACL in una variabile, lo utilizzerai nei comandi successivi. Ad esempio, puoi utilizzare il nome della variabile `acl200`:

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

Aggiungi ora la regola a `acl200`:

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

Aggiungi ulteriori regole finché la tua configurazione dell'ACL non è completa, salvando ogni ID come una variabile.

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

Inserisci le nuove regole in uscita prima della regola in uscita predefinita.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 10.10.20.0/24 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### Passo 4. Crea le due sottoreti con l'ACL appena creato

Creerai due sottoreti in modo che ognuno dei tuoi ACL sia associato a una delle nuove sottoreti. 

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## Scheda di riferimento dell'elenco di comandi

Per mostrare un elenco completo di comandi CLI VPC disponibili per gli ACL:

```
ibmcloud is network-acls
```
{: pre}

Per visualizzare il tuo ACL e i suoi metadati incluse le regole:

```
ibmcloud is network-acl $webacl
```
{: pre}

Per ottenere la regola dell'ACL in entrata predefinita: 

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## Regola `ping` in entrata di esempio

Per aggiungere una regola dell'ACL, ecco un comando di esempio per l'aggiunta di una regola in entrata `ping` prima della regola in entrata predefinita:

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
