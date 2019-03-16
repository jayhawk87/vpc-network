---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Beta) Utilizzo della VPN con il tuo VPC
{: #--beta-using-vpn-with-your-vpc}

Il servizio VPN di {{site.data.keyword.cloud}} VPC ti consente di collegare le reti private in modalità sicura. Puoi utilizzare la VPN per configurare un tunnel site-to-site IPsec tra il tuo VPC e la tua rete privata in loco o un altro VPC.

Per la release corrente di {{site.data.keyword.cloud}} VPC, è supportato solo l'instradamento basato sulla politica.

**Questo servizio è ora nella versione di release beta. Le risorse utilizzate durante il periodo della beta non saranno disponibili dopo la chiusura della beta e non sarà disponibile il supporto. Invia i commenti e le domande al tuo rappresentante delle vendite di IBM Cloud.**

## Funzioni

* IKEv1 e IKEv2
* Algoritmi di autenticazione: `md5`, `sha1`, `sha256`
* Algoritmi di codifica: `3des`, `aes128`, `aes256`
* Gruppi DH (Diffie-Hellman): 2, 5, 14
* Modalità di negoziazione IKE: main
* Protocollo di trasformazione IPSec: ESP
* Modalità di incapsulamento IPSec: tunnel
* PFS (Perfect Forward Secrecy)
* Rilevamento peer non attivo
* Instradamento: basato sulla politica
* Modalità di autenticazione: chiave precondivisa
* Supporto HA

## API disponibili

La seguente sezione ti fornisce i dettagli sulle API che puoi utilizzare per la VPN nel tuo ambiente IBM Cloud VPC. Per ulteriori dettagli, consulta la pagina [API REST VPC](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies).

### Gateway e connessioni VPN

| Descrizione | API |
|----------------------------|-------------|
| Crea un gateway VPN | POST /vpn_gateways |
| Richiama i gateway VPN | GET /vpn_gateways |
| Richiama un gateway VPN | GET /vpn_gateways/{id} |
| Elimina un gateway VPN | DELETE /vpn_gateways/{id} |
| Aggiorna un gateway VPN | PATCH /vpn_gateways/{id} |
| Crea una nuova connessione VPN | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Richiama le connessioni VPN | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Richiama una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Elimina una connessione VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Aggiorna una connessione VPN | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Richiama tutti i CIDR locali per una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Elimina un CIDR locale da una connessione VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Verifica se esiste un CIDR locale specifico su una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Imposta un CIDR locale su una connessione VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Richiama tutti i CIDR peer per una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Elimina un CIDR peer da una connessione VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Verifica se esiste un CIDR peer specifico su una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Imposta un CIDR peer su una connessione VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### Politiche IKE

| Descrizione | API |
|-----------------------------|--------------|
| Richiama tutte le politiche IKE | GET /ike_policies |
| Crea una politica IKE | POST /ike_policies |
| Elimina una politica IKE | DELETE /ike_policies/{id} |
| Richiama una politica IKE | GET /ike_policies/{id} |
| Aggiorna una politica IKE | PATCH /ike_policies/{id} |
| Richiama tutte le connessioni che utilizzano la politica IKE specificata | GET /ike_policies/{id}/connections |

### Politiche IPsec

| Descrizione | API |
|---------------------|-------------|
| Richiama tutte le politiche IPSec | GET /ipsec_policies |
| Crea una politica IPSec | POST /ipsec_policies |
| Elimina una politica IPSec | DELETE /ipsec_policies/{id} |
| Richiama una politica IPSec | GET /ipsec_policies/{id} |
| Aggiorna una politica IPSec | PATCH /ipsec_policies/{id} |
| Richiama tutte le connessioni che utilizzano la politica IPsec specificata | GET /ipsec_policies/{id}/connections |

## Esempio VPN

Nel seguente esempio, potrai collegare due VPC tra loro utilizzando la VPN, il che significa che puoi
collegare le sottoreti in due VPC separati come se fossero una singola rete. Gli indirizzi IP delle sottoreti non devono sovrapporsi.
Ecco a cosa somiglia lo scenario (con alcune VM aggiunte in ogni VPC):
![Uno scenario VPN di esempio](images/vpc-vpn.png)

### Istruzioni di esempio

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API IBM Cloud per creare i VPC. Per ulteriori informazioni, consulta [Introduzione](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) e [Configurazione di VPC con le API](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

Facoltativamente, puoi creare un gateway VPN utilizzando l'IU. Le istruzioni possono essere trovate nel [documento dell'esercitazione della console](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

#### Passo 1. Crea un gateway VPN nella tua sottorete VPC

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Assicurati di salvare i seguenti campi per i prossimi passi.
* `id`. Questo è l'ID del gateway VPN e vi sarà fatto riferimento come a `$gwid1`.
* `address`. Questo è l'indirizzo IP pubblico del gateway VPN e vi sarà fatto riferimento come a `$gwaddress1`.

NOTA: lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto. Puoi controllarne lo stato con il seguente comando:

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### Passo 2. Crea un secondo gateway VPN su un VPC differente

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Assicurati di salvare i seguenti campi per i prossimi passi.
* `id`. Questo è l'ID del gateway VPN e vi sarà fatto riferimento come a `$gwid2`.
* `address`. Questo è l'indirizzo IP pubblico del gateway VPN e vi sarà fatto riferimento come a `$gwaddress2`.


#### Passo 3. Crea una connessione VPN dal primo al secondo gateway VPN, impostando `local_cidrs` sulla sottorete sul VPC uno e `peer_cidrs` sulla sottorete sul VPC due

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01 \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Passo 4. Crea una connessione VPN dal secondo al primo gateway VPN, impostando `local_cidrs` sulla sottorete sul VPC due e `peer_cidrs` sulla sottorete sul VPC uno

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-01-01 \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Passo 5. Verifica della connettività

Dopo aver stabilito la connessione VPN, potrai raggiungere le tue istanze sulla sottorete due dalla sottorete uno e viceversa.

Puoi controllare lo stato della connessione VPN nel seguente modo:
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

Output di esempio:
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## Quote

Consulta il nostro argomento [Quote VPC](/docs/infrastructure/vpc?topic=vpc-quotas#vpn-quotas) per le quote VPN.

## Negoziazione automatica della politica

L'utilizzo delle politiche IKE e IPSec per configurare una connessione VPN è facoltativo. Quando non viene selezionata alcuna politica, vengono automaticamente scelte le proposte predefinite per un processo a cui viene fatto riferimento come _negoziazione automatica_. Le proposte utilizzate come parte della negoziazione automatica sono le seguenti.

Come iniziatore, IBM Cloud utilizza **IKEv2**. Come risponditore, IBM Cloud accetta **IKEv1** e **IKEv2**.
{: note}

### Negoziazione automatica IKE (fase 1)

È possibile utilizzare le seguenti opzioni di codifica, autenticazione e gruppo DH in qualsiasi combinazione:

|    | Codifica | Autenticazione | Gruppo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Negoziazione automatica IPsec (fase 2)

È possibile utilizzare le seguenti opzioni di codifica e autenticazione in qualsiasi combinazione:

|    | Codifica | Autenticazione | Gruppo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabilitato  |
| 2  | aes256 | sha256 | disabilitato  |
| 3  | 3des   | md5    | disabilitato  |

## Domande frequenti (FAQ)

**Quando creo un gateway VPN, posso creare le connessioni VPN nello stesso momento?**

Se utilizzi l'API o la CLI, le connessioni VPN devono essere creata dopo aver creato il gateway VPN. Nella console IBM Cloud, puoi creare il gateway e una connessione contemporaneamente.

**Se elimino un gateway VPN con delle connessioni VPN collegate, cosa succede alle connessioni?**

Le connessioni VPN vengono eliminate insieme al gateway VPN.

**Le politiche IKE o IPSec saranno eliminate se elimino una connessione o un gateway VPN?**

No, le politiche IKE e IPSec possono essere applicate a più connessioni.

**Cosa succede a un gateway VPN se provo ad eliminare la sottorete su cui risiede il gateway?**

La sottorete non può essere eliminata se è presente una qualsiasi istanza, incluso il gateway VPN.

**Esistono delle politiche IKE e IPSec predefinite?**

Quando crei una connessione VPN senza il riferimento a un ID della politica (IKE o IPSec), viene utilizzata la negoziazione automatica.

**La VPN per VPC supporta le configurazioni HA?**

Sì.
