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

# Configurazione della connettività tra i server web nel tuo VPC
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

Questo documento ti guida attraverso alcune istruzioni di esempio per la configurazione della comunicazione tra due server web nel tuo {{site.data.keyword.cloud}} VPC, utilizzando Python. Ti mostra come creare dei server di comunicazione nella stessa zona e in zone differenti.

## Prerequisiti

Prima di poter configurare la comunicazione tra i server web nel tuo VPC, devi aver già creato un VPC con DUE sottoreti ed almeno DUE istanze disponibili in ogni sottorete. Dovrai conoscere gli ID del VPC, le sottoreti e le istanze. Segui le istruzioni nella nostra [guida per la creazione delle risorse VPC con la CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) se hai bisogno di aiuto con questa configurazione.

## Scenario 1: connettività tra i server nella stessa zona ma con sottoreti diverse

`vpc` contiene l'ID vpc
`subnet1` contiene l'ID della sottorete 1
`subnet2` contiene l'ID della sottorete 2


## Parte 1: connettività tra i server dalla stessa zona e dallo stesso VPC

Dopo aver creato 2 o più istanze nella stessa sottorete che dispone di un gateway pubblico collegato, possiamo verificare la loro connettività ospitando un server con il codice HTML in un'istanza, poi eseguire `curl` su tale server e scaricare il file HTML sugli altri o dal browser del tuo computer.

Supponi di avere `instance_1` e `instance_2` nella stessa sottorete con il gateway pubblico e lo stesso gruppo di sicurezza. Devi aggiungere delle regole utilizzando l'API, la CLI o l'IU per l'istanza nei gruppi di sicurezza per consentire l'accesso in entrata. Se ospiti il tuo server sull'`instance_1` devi aggiungere una regola per l'IP mobile dell'`instance_2`. Ad esempio, supponi che l'`instance_1` che ospita il server abbia l'IPv4 impostato su `169.61.161.161` e l'`instance_2` abbia l'IPv4 impostato su `169.61.161.235` e tenti di OTTENERE il file dall'`instance_1`. Dovrebbe essere simile a questo:

![nuove regole del gruppo di sicurezza](images/security-group-ui-ex1.png)

* Per elencare tutti i gruppi di sicurezza `ibmcloud is sgs`
* Questa è una richiesta API per ottenere il risultato precedente:

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. Crea un file `index.html` di base sull'`instance_1`. Questo è un esempio di un file HTML di base:

```
<!DOCTYPE html>
<html>
<body>

<h1>You have successfully connected to the instance</h1>
<p>Hello, world!</p>

</body>
</html>
```
{:pre}

2. Nella stessa directory in cui è ubicato il file, esegui il comando `python -m SimpleHTTPServer <port number>`. Puoi utilizzare una porta diversa se la porta predefinita 8000 è già in uso.

3. Esegui l'SSH nell'`instance_2`

Per cercare un indirizzo IP mobile, utilizza il comando `ibmcloud is ips` e identifica il `floating_ip_id` utilizzando il comando `ibmcloud is ip $floating_ip_id`. Dovresti poter identificare l'indirizzo pubblico.

4. Identifica l'indirizzo IP dell'`instance_1`

5. Utilizza `curl -X GET http://IP_address:Port_number/index.html` per richiedere di scaricare `index.html` dal server che ospita l'`instance_1`. (Ad esempio: `curl -X GET http://169.61.161.161:8000/index.html`)

6. Dovresti poter visualizzare l'output di `index.html`

7. (Per l'accesso al server tramite il tuo browser del computer locale) Aggiungi delle regole a tutto il traffico in entrata come descritto nella seguente figura:

![nuove regole del gruppo di sicurezza con tutto il traffico in entrata](images/security-group-ui-ex2.png)

* Per aggiungere delle regole per consentire TUTTI i protocolli e da tutte le origini ai tuoi gruppi di sicurezza

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**Esempio di richiesta più dettagliato:**

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "port_max": 22,
  "port_min": 22,
  "protocol": "tcp",
  "remote": {
    "cidr_block": "131.121.0.0/24"
  }
}'
```
{:pre}

8. Apri un browser web e immetti `http://IP_address:Port_number/index.html`

9. Dovresti visualizzare il file index mostrato nel formato HTML.

## Parte 2: connettività tra i server da una zona e un VPC diversi

Connetti l'istanza dell'altra sottorete, che non ha il gateway pubblico configurato, all'istanza con l'accesso a internet. Vuoi configurare 2 diversi tipi di istanze:

* un server web che gestisce le richieste da internet utilizzando un gateway pubblico
* un server di back-end che esegue un'applicazione e archivia i dati importanti, inviando soltanto il traffico interno – gateway pubblico disabilitato.

I gruppi di sicurezza (SG) si applicano alle VSI (virtual server instance) e gli ACL di rete alle sottoreti.

Pertanto, per completare questa attività complessiva devi creare le VSI, le sottoreti, le regole dell'ACL e del gruppo di sicurezza (SG) e poi collegare l'SG all'interfaccia di rete di un'istanza e collegare l'ACL di rete a una sottorete specifica.

* Per elencare tutte le regole nel gruppo di sicurezza `subnet1` 

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. Rimuovi la regola `inbound-allow-all-traffic` nel gruppo di sicurezza dell'esempio di sottorete nella parte 1.

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. Crea un nuovo VPC, poi crea una sottorete senza il gateway pubblico collegato.

3. Crea un'`instance_3` nella nuova sottorete senza un gateway pubblico (senza alcuna connessione internet)- Potresti voler assegnare maggiori core CPU e memoria all'istanza per motivi di back-end.

4. Aggiungi le nuove regole per l'`instance_3` nel gruppo di sicurezza dalla parte 1. Ad esempio: l'`instance_3` ha un IP mobile di `169.61.181.25`

![nuove regole del gruppo di sicurezza con l'istanza da altro traffico in entrata della sottorete](images/security-group-ui-ex3.png)

5. Utilizza un comando del formato `curl -X GET http://IP_address:Port_number/index.html` per richiedere di scaricare `index.html` dal server che ospita l'instance_1. (Ad esempio: `curl -X GET http://169.61.161.161:8000/index.html`)

6. Dovresti poter visualizzare l'output di `index.html`

Questo significa che hai effettuato una connessione tra le istanze 3 e 1, ma non la 2.

## Passi successivi

Vorrai proteggere i tuoi server creando alcune regole ACL che controllano il traffico in entrata, come mostrato [nella nostra guida sull'utilizzo degli ACL nel VPC](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli).
