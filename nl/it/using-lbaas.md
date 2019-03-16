---



copyright:
  years: 2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Beta) Utilizzo di Load Balancers in IBM Cloud VPC
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

Il servizio del programma di bilanciamento del carico distribuisce il traffico tra più istanze del server nella stessa regione per il tuo {{site.data.keyword.cloud}} VPC.

**Questo servizio è ora nella versione di release beta. Le risorse utilizzate durante il periodo della beta non saranno disponibili dopo la chiusura della beta e non sarà disponibile il supporto. Invia i commenti e le domande al tuo rappresentante delle vendite di IBM Cloud.**

## Programma di bilanciamento del carico pubblico
{: #public-load-balancer}

Alla tua istanza del servizio del programma di bilanciamento del carico viene assegnato un nome di dominio completo (FQDN) accessibile pubblicamente, che devi utilizzare per accedere alle tue applicazioni ospitate dietro il servizio del programma di bilanciamento del carico in {{site.data.keyword.cloud}} VPC. Questo nome di dominio può essere registrato con uno o più indirizzi IP pubblici.

Nel tempo, questi indirizzi IP pubblici e il loro numero può venire modificato in base alle attività di manutenzione e ridimensionamento, che sono trasparenti ai clienti. Le istanze del servizio di backend (VSI) che ospitano la tua applicazione devono essere in esecuzione nella stessa regione e nello stesso VPC.

## Listener di frontend e pool di backend 
Puoi definire fino a 10 listener di frontend (porte dell'applicazione) e associarli ai rispettivi pool di backend nei server dell'applicazione di backend. Il nome di dominio completo (FQDN) assegnato alla tua istanza del servizio del programma di bilanciamento del carico e le porte del listener di frontend, vengono esposti a internet pubblicamente. Le richieste utente in entrata vengono ricevute su queste porte.

I protocolli del listener di frontend supportati sono:
* HTTP
* HTTPS
* TCP

I protocolli del pool di backend supportati sono:
* HTTP
* TCP

Il traffico HTTPS in entrata viene terminato nel programma di bilanciamento del carico per consentire la comunicazione HTTP di testo semplice con il server di backend.

Puoi collegare fino a 50 istanze del server a un pool di backend. Ogni istanza del server collegata deve avere una porta configurata. La porta può essere la stessa o meno della porta del listener di frontend.

L'intervallo di porte da 56500 a 56520 è riservato per motivi di gestione; queste porte non possono essere utilizzate come porte del listener di frontend.
{: note}

## Metodi di bilanciamento del carico
{: #load-balancing-methods}

Sono disponibili i seguenti tre metodi di bilanciamento del carico per distribuire il traffico tra i server dell'applicazione di backend:

* **Round-robin:** Round-robin è il metodo di bilanciamento del carico predefinito. Con questo metodo, il programma di bilanciamento del carico
inoltra le connessioni client in entrata in modalità round robin ai server di backend. Di conseguenza, tutti
i server di backend ricevono all'incirca un numero uguale di connessioni client.

* **Round-robin ponderato:** con questo metodo, il programma di bilanciamento del carico inoltra
le connessioni client in entrata ai server di backend in base al peso assegnato a tali server. Ad ogni server viene assegnato un peso predefinito
di 50, che può essere personalizzato con qualsiasi valore compreso tra 0 e 100.

Ad esempio, se tre applicazioni server A, B e C, hanno i pesi personalizzati con 60, 60 e 30 rispettivamente, i server A e B ricevono lo stesso numero di connessioni, mentre il server C ne riceve la metà. 

* **Numero minimo di connessioni:** con questo metodo, l'istanza del server che inoltra il numero minimo di connessioni a un ora specificata riceve la successiva connessione client. 

**Ulteriori caratteristiche di questi metodi:**

* Reimpostare il peso su '0' significa che nessuna nuova connessione viene inoltrata a tale server, ma tutto il traffico esistente continuerà a fluire finché rimane attivo. L'utilizzo di un peso di '0' può aiutare a terminare un server correttamente e a rimuoverlo dalla rotazione del servizio.
* I valori del peso del server sono applicabili solo con il metodo round-robin ponderato. Vengono ignorati con i metodi di bilanciamento del carico round-robin e numero minimo di connessioni.

## Controlli di integrità
{: #health-checks}

Le definizioni del controllo di integrità sono obbligatorie per i pool di backend.

Il programma di bilanciamento del carico effettua controlli di integrità periodici per monitorare l'integrità delle porte di backend e inoltra il traffico client ad esse di conseguenza. Se una porta del server di backend specificata viene trovata non integra, non viene più inoltrata alcuna nuova connessione ad essa. Il programma di bilanciamento del carico continua a monitorare l'integrità delle porte non integre e riprende ad utilizzarle se diventano nuovamente integre dopo aver superato due tentativi di controllo di integrità consecutivi.

I controlli di integrità per le porte HTTP e TCP vengono eseguiti nel seguente modo:

* **HTTP:** viene inviata una richiesta `HTTP GET` per un URL pre-specificato alla porta del server di backend. La porta del server viene contrassegnata come integra se viene ricevuta una risposta `200 OK`. Il percorso di integrità `GET` predefinito è "/" tramite l'IU e può essere personalizzato.

* **TCP:** il programma di bilanciamento del carico tenta di aprire una connessione TCP con il server di backend su una porta TCP specificata. La porta del server viene contrassegnata come integra se il tentativo di connessione ha esito positivo e la connessione viene chiusa. 

L'intervallo del controllo di integrità è 5 secondi, il timeout predefinito per una richiesta del controllo di integrità è 2 secondi e il numero di nuovi tentativi predefinito è 2.
{: note}

## Offload SSL e autorizzazioni necessarie

Per tutte le connessioni HTTPS in entrata, il servizio del programma di bilanciamento del carico termina la connessione SSL e stabilisce una comunicazione HTTP di testo semplice con l'istanza del server di backend. Con questa tecnica, gli handshake SSL con uso intensivo di CPU e le attività di crittografia/decrittografia sono stati spostati dalle istanze del server di backend, consentendo loro di utilizzare i propri cicli di CPU per elaborare il traffico dell'applicazione.

Un certificato SSL è obbligatorio al programma di bilanciamento del carico per eseguire le attività di offload SSL. Puoi gestire i certificati SSL tramite il [Gestore certificato di IBM ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Per permettere al programma di bilanciamento del carico di accedere al certificato SSL nella tua istanza del gestore del certificato, crea un'autorizzazione che fornisce all'istanza del servizio del programma di bilanciamento del carico l'accesso all'istanza di gestore del certificato che contiene il certificato SSL. Puoi gestire una simile autorizzazione tramite le [Autorizzazioni di identità e accesso ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](/docs/services/iam/#/authorizations){: new_window}. Assicurati di scegliere **Infrastructure Service** come servizio di origine, **Load Balancer for VPC** come tipo di risorsa, **Certificate Manager** come servizio di destinazione e assegna il ruolo di accesso al servizio di scrittore (**Writer**). Per creare un programma di bilanciamento del carico, devi concedere l'autorizzazione **All resource instances** all'istanza della risorsa di origine. L'istanza del servizio di destinazione può essere **All instances** o può essere la tua istanza della risorsa del gestore del certificato specifica.

Se viene rimossa l'autorizzazione richiesta, si potrebbero verificare degli errori con il tuo programma di bilanciamento del carico.
{: note}

## Identity and access management (IAM)

Puoi configurare le politiche di accesso per un'istanza **Programma di bilanciamento del carico per VPC**. Per gestire le tue politiche di accesso utente, visita [Autorizzazioni di identità e accesso ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](/docs/services/iam/#/authorizations){: new_window}.

### Configurazione delle politiche di accesso al gruppo di risorse per gli utenti

Per creare un programma di bilanciamento del carico, dovrai accedere a un gruppo di risorse. L'utente che sta creando un programma di bilanciamento del carico deve avere l'accesso appropriato al gruppo di risorse fornito o al gruppo di risorse predefinito se non ne viene fornito alcuno.

1. Passa a **Manage > Account > Users**. Visualizzerai un elenco di utenti con accesso al tuo account IBM Cloud.
2. Seleziona il nome dell'utente a cui vuoi assegnare una politica di accesso. Se l'utente non viene visualizzato, fai clic su **Invite users** per aggiungerlo al tuo account IBM Cloud.
3. Seleziona **Assign access**.
4. Seleziona **Assign access within a resource group**.
5. Dall'elenco a discesa **Resource group**, seleziona il gruppo di risorse desiderato.
6. Dall'elenco a discesa **Assign access to a resource group**, seleziona l'accesso desiderato.
7. Dall'elenco a discesa **Services**, seleziona i servizi desiderati.
9. Seleziona **Assign** per assegnare la politica di accesso al gruppo di risorse all'utente.

### Configurazione delle politiche di accesso alla risorsa per gli utenti

| Ruolo di accesso alla piattaforma | Azione del programma di bilanciamento del carico |
|-------------|-----|
| Amministratore | Creare/Visualizzare/Modificare/Eliminare il programma di bilanciamento del carico |
| Editor | Creare/Visualizzare/Modificare/Eliminare il programma di bilanciamento del carico |
| Visualizzatore | Visualizzare il programma di bilanciamento del carico |

1. Passa a **Manage > Account > Users**. Visualizzerai l'elenco di utenti con accesso al tuo account IBM Cloud.
2. Seleziona il nome dell'utente a cui vuoi assegnare una politica di accesso. Se l'utente non viene visualizzato, fai clic su **Invite users** per aggiungerlo al tuo account IBM Cloud.
3. Seleziona **Assign access**.
4. Seleziona **Assign access to resources**.
5. Dall'elenco a discesa **Services**, seleziona **Infrastructure Service**.
6. Dall'elenco a discesa **Resource type**, seleziona **Load Balancer for VPC**.
7. Dall'elenco a discesa **Load Balancer ID**, seleziona un ID dell'istanza del programma di bilanciamento del carico o utilizza il valore predefinito **All load balancers**.
8. Assegna un ruolo di accesso alla piattaforma all'utente.
9. Seleziona **Assign** per assegnare la politica di accesso all'utente.

## API disponibili

Per effettuare le chiamate API, devi utilizzare una sorta di client REST. Ad esempio, puoi utilizzare il comando `curl` per richiamare tutti i programmi di bilanciamento del carico esistenti:

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

La seguente sezione ti fornisce i dettagli sulle API che puoi utilizzare per i programmi di bilanciamento del carico nel tuo ambiente VPC. Per la specifica completa, consulta la [Guida di riferimento API RIAS](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers).

| Descrizione | API |
|-------------|-----|
| Crea ed esegue il provisioning di un programma di bilanciamento del carico | POST /load_balancers |
| Richiama tutti i programmi di bilanciamento del carico | GET /load_balancers |
| Richiama un programma di bilanciamento del carico | GET /load_balancers/{id} |
| Elimina un programma di bilanciamento del carico | DELETE /load_balancers/{id} |
| Aggiorna un programma di bilanciamento del carico | PATCH /load_balancers/{id} |
| Crea un listener | POST /load_balancers/{id}/listeners |
| Richiama tutti i listener del programma di bilanciamento del carico | GET /load_balancers/{id}/listeners |
| Richiama un listener | GET /load_balancers/{id}/listeners/{listener_id} |
| Elimina un listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Aggiorna un listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Crea un pool | POST /load_balancers/{id}/pools |
| Richiama tutti i pool del programma di bilanciamento del carico | GET /load_balancers/{id}/pools |
| Richiama un pool | GET /load_balancers/{id}/pools/{pool_id} |
| Elimina un pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Aggiorna un pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Crea un membro | POST /load_balancers/{id}/pools/{pool_id}/members |
| Richiama tutti i membri del pool | GET /load_balancers/{id}/pools/{pool_id}/members |
| Richiama un membro |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Elimina un membro dal pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aggiorna un membro | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aggiorna i membri del pool | PUT /load_balancers/{id}/pools/{pool_id}/members |

## Esempio di programma di bilanciamento del carico 

Nel seguente esempio, utilizzerai l'API per creare un programma di bilanciamento del carico davanti a 2 istanze del server VPC (`192.168.100.5` e `192.168.100.6`) eseguendo un'applicazione web in ascolto sulla porta `80`. Il programma di bilanciamento del carico ha un listener di frontend che consente l'accesso all'applicazione web in modo sicuro tramite HTTPS. Successivamente, puoi utilizzare l'API per ottenere i dettagli dell'istanza del programma di bilanciamento del carico dopo averla creata e per eliminarla.

### Istruzioni di esempio

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo dell'[IU IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), della [CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) o dell'[API RIAS](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) per eseguire il provisioning di una VPC, delle sottoreti e delle istanze. 


Le istruzioni di esempio del programma di bilanciamento del carico possono essere eseguite anche utilizzando la [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference).
{: note}

#### Passo 1. Crea un programma di bilanciamento del carico con il listener, il pool e le istanze del server collegate (membri)

```bash
curl -H "Authorization: $iam_token" -X POST
$rias_endpoint/v1/load_balancers?version=2019-01-01 \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Output di esempio:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d-7ce5-413b-aae4-90a2a88b7872.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

Salva l'ID del programma di bilanciamento del carico da utilizzare nei prossimi passi, ad esempio, nella variabile `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### Passo 2. Ottieni un programma di bilanciamento del carico

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

Concedi un po' di tempo al provisioning del programma di bilanciamento del carico. Quando pronto, lo stato sarà impostato su `online` e `active`, come vedrai nel seguente output di esempio:

Output di esempio:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

#### Passo 3. Elimina un programma di bilanciamento del carico

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## Domande frequenti

Questa sezione contiene le risposte ad alcune domande frequenti sul servizio **Programma di bilanciamento del carico per VPC**. 

### Posso utilizzare un nome DNS diverso per il mio programma di bilanciamento del carico?

Il nome DNS assegnato automaticamente per il programma di bilanciamento del carico non è personalizzabile. Tuttavia, puoi aggiungere un record del nome canonico (CNAME) che faccia puntare il tuo nome DNS preferito al nome DNS del programma di bilanciamento del carico assegnato automaticamente. Ad esempio, il tuo ID del programma di bilanciamento del carico è `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, il nome DNS del programma di bilanciamento del carico assegnato automaticamente è `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`. Il tuo nome DNS preferito è `www.myapp.com`. Puoi aggiungere un record CNAME (tramite il provider DNS che utilizzi per gestire `myapp.com`) che faccia puntare `www.myapp.com` al nome DNS del programma di bilanciamento del carico `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`.

### Qual è il numero massimo di listener di frontend che posso definire con il mio programma di bilanciamento del carico?

10.

### Qual è il numero massimo di istanze del server che posso collegare al mio pool di backend?

50.

### Posso creare un programma di bilanciamento del carico privato e solo interno a cui possono accedere solo i client interni?  

No, non in questo momento.

### Il programma di bilanciamento del carico è scalabile orizzontalmente?  

No, non in questo momento.

### Cosa devo fare se sto utilizzando degli ACL o dei gruppi di sicurezza sulle sottoreti utilizzate per distribuire il programma di bilanciamento del carico?

Dovrai assicurarti che siano in vigore le regole del gruppo di sicurezza o dell'ACL appropriate per consentire il traffico in entrata per le porte di gestione e del listener configurate (porte da 56500 a 56520). Deve essere consentito anche il traffico tra il programma di bilanciamento del carico e le istanze di backend.

### Perché sto ricevendo un messaggio di errore: `certificate instance not found`?

* Il CRN dell'istanza del certificato potrebbe non essere valido.
* Potresti non aver concesso l'autorizzazione da-servizio-a-servizio. Consulta la sezione **Offload SSL** di questo documento.

### Perché ricevo il codice `401 Unauthorized Error`?

Controlla le seguenti politiche di accesso per il tuo utente:
* Politica di accesso al tipo di risorsa del programma di bilanciamento del carico
* Politica di accesso al gruppo di risorse

### Perché il mio programma di bilanciamento del carico è nello stato `maintenance_pending`?

Il programma di bilanciamento del carico sarà nello stato `maintenance_pending` durante molte attività di manutenzione come ad esempio:
* Ogni aggiornamento progressivo per risolvere le vulnerabilità e applicare le patch di sicurezza
* Attività di ripristino

### Perché devo scegliere più sottoreti durante il provisioning?

Le applicazioni del programma di bilanciamento del carico sono distribuite alle sottoreti che hai selezionato. Si consiglia vivamente di scegliere le sottoreti da zone differenti per l'alta disponibilità e la ridondanza.
