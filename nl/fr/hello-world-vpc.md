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

# VPC Hello World avec la CLI

Ce guide détaillé explique comment mettre à disposition une instance de serveur virtuel dans votre propre instance {{site.data.keyword.cloud}} Virtual Private Cloud, à l'aide de la CLI d'IBM Cloud.

## Prérequis :

1. Assurez-vous que votre compte IBM Cloud est activé pour VPC, voir [comment participer](how-to-participate.html) pour plus d'informations.

2. Assurez-vous de disposer d’une clé SSH publique. 

   Vous avez peut-être déjà une clé SSH publique. Recherchez un fichier appelé ``id_rsa.pub`` dans le répertoire ``.ssh``, sous votre répertoire de base, par exemple, ``/Users/<USERNAME>/.ssh/id_rsa.pub``. Le fichier commence par ``ssh-rsa`` et se termine par votre adresse électronique. 

   Si vous ne possédez pas de clé SSH publique ou si vous avez oublié le mot de passe d'une clé existante, générez-en une nouvelle en exécutant la commande `ssh-keygen` et en suivant les invites. 

3. Installez la [CLI IBM Cloud![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started){: new_window}.

4. Installez le plug-in `infrastructure-service` pour la CLI IBM Cloud.

```
ibmcloud plugin install infrastructure-service
```
{: codeblock}

## Etape 1 : Connectez-vous à IBM Cloud. 

Si vous disposez d'un compte fédéré, utilisez la connexion unique : 

```
ibmcloud login -sso
```
{: codeblock}

Faute de quoi, utilisez cette commande : 

```
ibmcloud login
```
{: codeblock}

## Etape 2 : Créez un VPC et enregistrez l'ID de VPC. 

Utilisez la commande suivante pour créer un VPC nommé _helloworld-vpc_.

```
ibmcloud is vpc-create helloworld-vpc
```
{: codeblock}

La sortie devrait se présenter comme suit : 

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

Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement, par exemple : 

```
vpc="ba9e785a-3e10-418a-811c-56cfe5669676"
```
{: codeblock}

## Etape 3 : Créez un sous-réseau et enregistrez l'ID de sous-réseau.  

Choisissez la zone `us-south-2` correspondant à l'emplacement du sous-réseau et nommez le sous-réseau _helloworld-subnet_.

```
ibmcloud is subnet-create helloworld-subnet $vpc us-south-2 --ipv4_cidr_block "10.0.1.0/24"
```
{: codeblock}

La sortie devrait se présenter comme suit : 

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

Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement, par exemple : 

```
subnet="50ba0da9-279a-4791-b7cb-cd3d7b2bc14"
```
{: codeblock}

Notez que le statut du sous-réseau est `pending` (en attente) lors de sa création. Pour que vous puissiez continuer, le sous-réseau doit passer au statut `available` (disponible), ce qui prend quelques secondes. Pour vérifier le statut du sous-réseau, exécutez la commande suivante : 

```
ibmcloud is subnet $subnet
```
{: codeblock}

## Etape 4 : Créez une clé SSH dans IBM Public Cloud. 

Vous utilisez la clé pour mettre à disposition une instance de serveur virtuel. Vous pouvez utiliser une clé pour mettre à disposition plusieurs instances de serveur virtuel. 

Pour consulter les clés disponibles dans votre compte IBM Cloud, exécutez la commande suivante : 

```
ibmcloud is keys
```
{: codeblock}

Pour créer une clé, exécutez la commande suivante. Remplacez le chemin d'accès à votre fichier `id_rsa.pub`.

```
ibmcloud is key-create helloworld-key @$HOME/.ssh/id_rsa.pub
```
{: codeblock}

La sortie devrait se présenter comme suit : 

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

Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement, par exemple : 

```
key="859b4e97-7540-4337-9c64-384792b85653"
```

## Etape 5 : Sélectionnez un profil et une image pour l'instance de serveur virtuel.  

Choisissez le profil d'instance `b-4x8` et l'image `ubuntu-16.04-amd64`. Pour obtenir l'ID d'image de `ubuntu-16.04-amd64`, exécutez la commande suivante :

```
image=$(ibmcloud is images | grep "ubuntu-16.04-amd64" | cut -d" " -f1)
```
{: codeblock}

## Etape 6 : Mettez à disposition une instance de serveur virtuel.  

```
ibmcloud is instance-create helloworld-vsi $vpc us-south-2 b-4x8 $subnet 1000 --image $image --keys $key
```
{: codeblock}

La sortie devrait se présenter comme suit : 

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

Enregistrez l'ID de l'interface réseau principale `primary intf` dans une variable pour pouvoir l'utiliser ultérieurement, par exemple : 

```
nic="2e850924-b5d7-4386-a778-03556d5850c1"
```
{:codeblock}

Notez que le statut de l'instance est `pending` (en attente) lors de sa création. Pour que vous puissiez continuer, l'instance doit passer au statut `running` (en cours d'exécution), ce qui prend quelques minutes. Pour vérifier le statut de toutes les instances, exécutez la commande suivante : 

```
ibmcloud is instances
```
{: codeblock}


## Etape 7 : Créez une adresse IP flottante. 

L'adresse IP flottante est nécessaire pour la connexion à l'instance de serveur virtuel (VSI) à partir d'Internet. 

```
ibmcloud is floating-ip-reserve helloworld-fip --nic $nic
```
{: codeblock}

La sortie devrait se présenter comme suit : 

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

Enregistrez `Address` dans une variable afin de pouvoir l'utiliser ultérieurement : 

```
address=169.61.181.53
```
{: codeblock}

## Etape 8 : Ajoutez une règle au groupe de sécurité par défaut pour SSH. 

Recherchez le groupe de sécurité pour le VPC : 

```
ibmcloud is vpc-sg $vpc
```
{: codeblock}

La sortie devrait se présenter comme suit : 
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

Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement : 

```
sg=2d364f0a-a870-42c3-a554-000000981149
```
{: codeblock}

A présent, créez la règle pour autoriser SSH : 

```
ibmcloud is sg-rulec $sg ingress tcp --port-min=22 --port-max=22
```
{: codeblock}

La sortie devrait se présenter comme suit : 
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

## Etape 9 : Connectez-vous à votre instance de serveur virtuel à l'aide de votre clé SSH privée. 

Par exemple, vous pouvez utiliser une commande de la forme suivante : 

```
$ ssh -i $HOME/.ssh/id_rsa root@$address
```
{:codeblock}

 Vous recevez une réponse similaire à l'exemple qui suit. Lorsque vous êtes invité à poursuivre la connexion, entrez `yes`.

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

## Etape 10 : Hello, World!

Exécutez la commande suivante dans la fenêtre du terminal : 

```
echo `hostname` says "Hello, World!"
```
{:codeblock}

La sortie devrait se présenter comme suit : 

```
helloworld-vsi says Hello, World!
```
{:screen}

## Etape suivante

Essayez un autre [guide détaillé](step-by-step-pages.html).
