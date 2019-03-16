---

copyright:
  years: 2017, 2018
lastupdated: "2018-09-25"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}

# VPC Hello World con la CLI

Questa guida dettagliata ti mostra come eseguire il provisioning di un'istanza del server virtuale nel tuo {{site.data.keyword.cloud}} Virtual Private Cloud, utilizzando la CLI IBM Cloud.

## Prerequisiti:

1. Assicurati che il tuo account IBM Cloud sia abilitato per VPC, consulta [come partecipare](how-to-participate.html) per ulteriori informazioni.

2. Assicurati di avere una chiave SSH pubblica disponibile per l'utilizzo.

   Potresti avere già una chiave SSH pubblica. Ricerca un file denominato ``id_rsa.pub`` in una directory ``.ssh`` nella tua directory home, ad esempio, ``/Users/<USERNAME>/.ssh/id_rsa.pub``. Il file inizia con ``ssh-rsa`` e termina con il tuo indirizzo email.

   Se non hai una chiave SSH pubblica o se hai dimenticato la password di una chiave esistente, generane una nuova eseguendo il comando `ssh-keygen` e seguendo le istruzioni. 

3. Installa la [CLI IBM Cloud ![Icona link esterno](../../icons/launch-glyph.svg "Icona link esterno")](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started){: new_window}.

4. Installa il plugin `infrastructure-service` per la CLI IBM Cloud.

```
ibmcloud plugin install infrastructure-service
```
{: codeblock}

## Passo 1: accedi a IBM Cloud.

Se hai un account federato, utilizza SSO (Single Sign On):

```
ibmcloud login -sso
```
{: codeblock}

Altrimenti utilizza questo comando:

```
ibmcloud login
```
{: codeblock}

## Passo 2: crea un VPC e salva l'ID VPC.

Utilizza il seguente comando per creare un VPC denominato _helloworld-vpc_.

```
ibmcloud is vpc-create helloworld-vpc
```
{: codeblock}

Dovresti vedere un output simile a questo:

```
Creating VPC helloworld-vpc in resource group Default under account <Account Name> as user <User>...

ID        ba9e785a-3e10-418a-811c-56cfe5669676   
Name      helloworld-vpc   
Default   no   
Created   1 second ago   
Status    available   
Tags      -   
```
{:screen}

Salva l'ID in una variabile in modo che possiamo utilizzarlo successivamente, ad esempio:

```
vpc="ba9e785a-3e10-418a-811c-56cfe5669676"
```
{: codeblock}

## Passo 3: crea una sottorete e salva l'ID della sottorete. 

Scegli la zona `us-south-2` per l'ubicazione della sottorete e richiama la sottorete _helloworld-subnet_.

```
ibmcloud is subnet-create helloworld-subnet $vpc us-south-2 --ipv4_cidr_block "10.0.1.0/24"
```
{: codeblock}

Dovresti vedere un output simile a questo:

```
Creating Subnet helloworld-subnet in resource group Default under account <Account Name> as user <User>...

ID               50ba0da9-279a-4791-b7cb-cd3d7b2bc14d   
Name             helloworld-subnet   
IPv*             ipv4   
IPv4 CIDR        10.0.1.0/24   
IPv6 CIDR        -   
Addr available   0   
Addr Total       0   
Gen              -   
ACL              allow-all-network-acl-ba9e785a-3e10-418a-811c-56cfe5669676(e9c2790b-cee2-465a-8539-d8cd90d33621)   
Gateway          -   
Created          now   
Status           pending   
Zone             us-south-2   
VPC              helloworld-vpc(ba9e785a-3e10-418a-811c-56cfe5669676)   
Resource Group   -   
Tags             -   
```
{:screen}

Salva l'ID in una variabile in modo che possiamo utilizzarlo successivamente, ad esempio:

```
subnet="50ba0da9-279a-4791-b7cb-cd3d7b2bc14"
```
{: codeblock}

Nota che lo stato della sottorete è `pending` appena viene creata. Prima di poter procedere, la sottorete deve passare allo stato `available`, il che richiede alcuni secondi. Per controllare lo stato della sottorete, immetti questo comando:

```
ibmcloud is subnet $subnet
```
{: codeblock}

## Passo 4: crea una chiave SSH nel cloud pubblico di IBM.

Utilizzerai la chiave per eseguire il provisioning di un'istanza del server virtuale. Puoi utilizzare una chiave per eseguire il provisioning di più istanze del server virtuale.

Per visualizzare le chiavi disponibili nel tuo account IBM Cloud, immetti questo comando:

```
ibmcloud is keys
```
{: codeblock}

Per creare una chiave, immetti il seguente comando. Sostituisci il percorso al tuo file `id_rsa.pub`.

```
ibmcloud is key-create helloworld-key @$HOME/.ssh/id_rsa.pub
```
{: codeblock}

Dovresti vedere un output simile a questo:

```
Creating key helloworld-key in resource group Default under account <Account Name> as user <User>...

ID               859b4e97-7540-4337-9c64-384792b85653   
Name             helloworld-key   
Type             rsa   
Length           2048   
FingerPrint      SHA256:hkcAOGB5z7QXqZLHd0kGqhj735qrfMjZLH9PxS/42vA   
Key              ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC9i2joQ8eiFVdJ7pOlC6h5+IoBL6wFFygkk9Na3gV8Bi52dv44YOAjSJ2oduguHEtLp5r4eh4+5jiEBkFYkHNUhE0MxlcVZABTYWePXx4QnlmGr99xyOfi6DAHhSRQiSBhmhjcGjbADavDuIgoyKpVXbU9O1If3P0miNpaouaZmr+d68OHt4yPvNnztlluV3JBISJgqJ7pzg6wFF0SrjqtEYKBd8oxwogHu+rmRgT7IF09oSiKpKZRF0VfeLFz+REh9RuKa4Jh63aa2PRIgDKq6HO7MEdfOtGzCzoqqlFKgpl+VgGyT0b5BjQEnWv13cwx02bv5QCwma/GeAOpW0CD user@email.com   

Created          now   
Resource Group   -   
Tags             -   
```
{:screen}

Salva l'ID in una variabile in modo che possiamo utilizzarlo successivamente, ad esempio:

```
key="859b4e97-7540-4337-9c64-384792b85653"
```

## Passo 5: seleziona un profilo e un'immagine per l'istanza del server virtuale. 

Scegli il profilo `b-4x8` e l'immagine `ubuntu-16.04-amd64` per l'istanza. Per ottenere l'ID dell'immagine di `ubuntu-16.04-amd64`, immetti il seguente comando:

```
image=$(ibmcloud is images | grep "ubuntu-16.04-amd64" | cut -d" " -f1)
```
{: codeblock}

## Passo 6: esegui il provisioning di un'istanza del server virtuale. 

```
ibmcloud is instance-create helloworld-vsi $vpc us-south-2 b-4x8 $subnet 1000 --image $image --keys $key
```
{: codeblock}

Dovresti vedere un output simile a questo:

```
Creating instance helloworld-vsi in resource group Default under account <Account Name> as user <User>...

ID                4562c5c0-9cf7-4406-bc90-ab4baea91057   
Name              helloworld-vsi   
Gen                  
Profile           b-4x8   
CPU Arch          amd64   
CPU Cores         4   
CPU Frequency     2000   
Memory            8   
Primary Intf      primary(2e850924-b5d7-4386-a778-03556d5850c1)   
Primary Address   10.0.1.5   
Image             ubuntu-16.04-amd64(7eb4e35b-4257-56f8-d7da-326d85452591)   
Status            pending   
Created           5 seconds ago   
VPC               helloworld-vpc(ba9e785a-3e10-418a-811c-56cfe5669676)   
Zone              us-south-2   
Resource Group    -   
Tags              -   
```
{:screen}

Salva l'ID dell'interfaccia di rete primaria `primary intf` in una variabile in modo che possiamo utilizzarlo successivamente, ad esempio:

```
nic="2e850924-b5d7-4386-a778-03556d5850c1"
```
{:codeblock}

Nota che lo stato dell'istanza è `pending` appena viene creata. Prima di poter procedere, l'istanza deve passare allo stato `running`, il che richiede alcuni minuti. Per controllare lo stato di tutte le istanze, immetti questo comando:

```
ibmcloud is instances
```
{: codeblock}


## Passo 7: crea un indirizzo IP mobile.

Hai bisogno dell'indirizzo IP mobile per accedere all'istanza del server virtuale (VSI) da internet.

```
ibmcloud is floating-ip-reserve helloworld-fip --nic $nic
```
{: codeblock}

Dovresti vedere un output simile a questo:

```
Creating floating ip helloworld-fip in resource group Default under account <Account Name> as user <User>...

ID               b9d1cc1f-67db-40e3-81de-9228465170a5   
Address          169.61.181.53   
Name             helloworld-fip   
Target           primary(2e850924-.)   
Target Type      intf   
Target IP        10.0.1.5   
Created          now   
Status           available   
Zone             us-south-2   
Resource Group   -   
Tags             -   
```
{:screen}

Salva l'indirizzo (`Address`) in una variabile in modo che possiamo utilizzarlo successivamente:

```
address=169.61.181.53
```
{: codeblock}

## Passo 8: aggiungi una regola al gruppo di sicurezza predefinito per SSH.

Trova il gruppo di sicurezza per il VPC:

```
ibmcloud is vpc-sg $vpc
```
{: codeblock}

Dovresti vedere un output simile a questo:
```
Getting default security group of vpc ba9e785a-3e10-418a-811c-56cfe5669676 under account <Account Name> as user <User>...

ID               2d364f0a-a870-42c3-a554-000000981149
Name             errand-drastic-imperial-retail-unlocked-jester
Created          1 week ago
VPC              helloworld-vpc(ba9e785a-3e10-418a-811c-56cfe5669676)
Resource Group   -
Tags             -

Rules
ID                                     Direction   IPv*   Protocol                  Remote
b597cff2-38e8-4e6e-999d-000002031345   inbound     ipv4   all                       errand-drastic-imperial-retail-unlocked-jester(2d364f0a-.)
b597cff2-38e8-4e6e-999d-000002031527   outbound    ipv4   all                       -

```
{:screen}

Salva l'ID in una variabile in modo che possiamo utilizzarlo successivamente:

```
sg=2d364f0a-a870-42c3-a554-000000981149
```
{: codeblock}

Crea ora la regola per consentire SSH:

```
ibmcloud is sg-rulec $sg ingress tcp --port-min=22 --port-max=22
```
{: codeblock}

Dovresti vedere un output simile a questo:
```
Creating rule for security group 2d364f0a-a870-42c3-a554-000000981149 under account <Account Name> as user <User>...

ID          b597cff2-38e8-4e6e-999d-000001949921
Direction   inbound
IPv*        ipv4
Protocol    tcp
Min Port    22
Max Port    22
Remote      -
```
{:screen}

## Passo 9: accedi alla tua istanza del server virtuale utilizzando la tua chiave SSH privata.

Ad esempio, puoi utilizzare un comando di questo formato:

```
$ ssh -i $HOME/.ssh/id_rsa root@$address
```
{:codeblock}

 Riceverai una risposta simile al seguente esempio. Quando ti viene richiesto di continuare con la connessione, immetti `yes`.

```
The authenticity of host '169.61.181.53 (169.61.181.53)' can't be established.
ECDSA key fingerprint is SHA256:9MczXIwJq892DYwu0sZpQZOXORdvNXeP1aFioZpQDsM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '169.61.181.53' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-133-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@helloworld-vsi:~#
```
{:screen}

## Passo 10: Hello, World!

Immetti il seguente comando nella finestra del terminale:

```
echo `hostname` says "Hello, World!"
```
{:codeblock}

Dovresti vedere il seguente output:

```
helloworld-vsi says Hello, World!
```
{:screen}

## Operazioni successive

Prova un'altra [guida dettagliata](step-by-step-pages.html).
