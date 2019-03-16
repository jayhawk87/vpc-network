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

# Hello World VPC con la CLI

Esta guía paso a paso le muestra cómo suministrar una instancia de servidor virtual en su propia nube privada virtual de {{site.data.keyword.cloud}} utilizando la CLI de IBM Cloud.

## Requisitos previos:

1. Asegúrese de que la cuenta de IBM Cloud está habilitada para VPC; consulte [cómo participar](how-to-participate.html) para obtener más información.

2. Asegúrese de que tiene de una clave SSH pública disponible para su uso.

   Es posible que ya tenga una clave SSH pública. Busque un archivo denominado ``id_rsa.pub`` en un directorio ``.ssh`` del directorio de inicio; por ejemplo, ``/Users/<USERNAME>/.ssh/id_rsa.pub``. El archivo empieza por ``ssh-rsa`` y termina con su dirección de correo electrónico.

   Si no tiene una clave SSH pública o si ha olvidado la contraseña de una clave existente, genere una nueva ejecutando el mandato `ssh-keygen` y siguiendo la solicitud siguiente.

3. Instale la [CLI de IBM Cloud ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started){: new_window}.

4. Instale el plugin `infrastructure-service` para la CLI de IBM Cloud.

```
ibmcloud plugin install infrastructure-service
```
{: codeblock}

## Paso 1: Inicie sesión en IBM Cloud.

Si tiene una cuenta federada, utilice el inicio de sesión único:

```
ibmcloud login -sso
```
{: codeblock}

De lo contrario, utilice este mandato:

```
ibmcloud login
```
{: codeblock}

## Paso 2: Cree un VPC y guarde el ID de VPC.

Utilice el mandato siguiente para crear un VPC denominado _helloworld-vpc_.

```
ibmcloud is vpc-create helloworld-vpc
```
{: codeblock}

Debería ver una salida como la siguiente:

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

Guarde el ID en una variable para poder utilizarlo más adelante; por ejemplo:

```
vpc="ba9e785a-3e10-418a-811c-56cfe5669676"
```
{: codeblock}

## Paso 3: Cree una subred y guarde el ID de subred. 

Vamos a seleccionar la zona `us-south-2` para la ubicación de la subred y llamar a la subred _helloworld-subnet_.

```
ibmcloud is subnet-create helloworld-subnet $vpc us-south-2 --ipv4_cidr_block "10.0.1.0/24"
```
{: codeblock}

Debería ver una salida como la siguiente:

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

Guarde el ID en una variable para poder utilizarlo más adelante; por ejemplo:

```
subnet="50ba0da9-279a-4791-b7cb-cd3d7b2bc14"
```
{: codeblock}

Tenga en cuenta que el estado de la subred es `pendiente` cuando se crea por primera vez. Antes de poder continuar, la subred debe pasar al estado `disponible`, proceso que tarda unos segundos. Para comprobar el estado de la subred, ejecute el mandato siguiente:

```
ibmcloud is subnet $subnet
```
{: codeblock}

## Paso 4: Cree una clave SSH en la nube pública de IBM.

Utilizará la clave para suministrar una instancia de servidor virtual. Puede utilizar una clave para suministrar varias instancias de servidor virtual.

Para ver las claves disponibles en la cuenta de IBM Cloud, ejecute este mandato:

```
ibmcloud is keys
```
{: codeblock}

Para crear una clave, ejecute el mandato siguiente. Sustituya la vía de acceso en el archivo `id_rsa.pub`.

```
ibmcloud is key-create helloworld-key @$HOME/.ssh/id_rsa.pub
```
{: codeblock}

Debería ver una salida como la siguiente:

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

Guarde el ID en una variable para poder utilizarlo más adelante; por ejemplo:

```
key="859b4e97-7540-4337-9c64-384792b85653"
```

## Paso 5: Seleccione un perfil e imagen para la instancia de servidor virtual. 

Escojamos el perfil de instancia `b-4x8` y la imagen `ubuntu-16.04-amd64`. Para obtener el ID de imagen de `ubuntu-16.04-amd64`, ejecute el mandato siguiente:

```
image=$(ibmcloud is images | grep "ubuntu-16.04-amd64" | cut -d" " -f1)
```
{: codeblock}

## Paso 6: Proporcione una instancia de servidor virtual. 

```
ibmcloud is instance-create helloworld-vsi $vpc us-south-2 b-4x8 $subnet 1000 --image $image --keys $key
```
{: codeblock}

Debería ver una salida como la siguiente:

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

Guarde el ID de la interfaz de red primaria `primary intf` en una variable para poder utilizarla más adelante; por ejemplo:

```
nic="2e850924-b5d7-4386-a778-03556d5850c1"
```
{:codeblock}

Tenga en cuenta que el estado de la instancia es `pendiente` cuando se crea por primera vez. Antes de poder continuar, la instancia debe pasar al estado `En ejecución`, proceso que tarda unos minutos. Para comprobar el estado de todas las instancias, ejecute este mandato:

```
ibmcloud is instances
```
{: codeblock}


## Paso 7: Cree una dirección IP flotante.

Necesita la dirección IP flotante para poder iniciar sesión en la instancia de servidor virtual (VSI) desde Internet.

```
ibmcloud is floating-ip-reserve helloworld-fip --nic $nic
```
{: codeblock}

Debería ver una salida como la siguiente:

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

Guarde la `Dirección` en una variable para poder utilizarla más adelante:

```
address=169.61.181.53
```
{: codeblock}

## Paso 8: Añada una regla al grupo de seguridad para SSH predeterminado

Busque el grupo de seguridad para el VPC:

```
ibmcloud is vpc-sg $vpc
```
{: codeblock}

Debería ver una salida como la siguiente:
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

Guarde el ID en una variable para poder utilizarlo más adelante:

```
sg=2d364f0a-a870-42c3-a554-000000981149
```
{: codeblock}

Ahora cree la regla para permitir el SSH:

```
ibmcloud is sg-rulec $sg ingress tcp --port-min=22 --port-max=22
```
{: codeblock}

Debería ver una salida como la siguiente:
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

## Paso 9: Inicie sesión en la instancia de servidor virtual utilizando la clave SSH privada.

Por ejemplo, puede utilizar un mandato con este formato:

```
$ ssh -i $HOME/.ssh/id_rsa root@$address
```
{:codeblock}

 Recibirá una respuesta similar a la del ejemplo siguiente. Cuando se le solicite continuar conectándose, escriba `yes`.

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

## Paso 10: Hello, World!

Ejecute el mandato siguiente en la ventana de terminal:

```
echo `hostname` says "Hello, World!"
```
{:codeblock}

Verá la salida siguiente:

```
helloworld-vsi says Hello, World!
```
{:screen}

## Qué hacer a continuación

Pruebe con otra [guía paso a paso](step-by-step-pages.html).
