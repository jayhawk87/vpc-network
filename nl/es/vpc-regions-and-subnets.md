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

# Cómo trabajar con rangos de direcciones IP, prefijos de direcciones, regiones y subredes 
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}

Este documento proporciona una visión general de los rangos de direcciones IP disponibles (o restringidos) y las acciones de usuario relacionadas con regiones y subredes y con prefijos de direcciones. Determinados rangos de direcciones IP no están disponibles porque es posible que IBM Cloud los esté utilizando.  

Cuando crea una subred en la VPC de {{site.data.keyword.cloud}}, se asigna automáticamente un _prefijo de dirección_ en función de la región y la zona en la que ha creado la VPC. Puede utilizar el prefijo de dirección predeterminado o puede modificarlo manualmente, incluida una dirección BYOIP, en muchos de los casos. 

## VPC y regiones de IBM Cloud

Una VPC de {{site.data.keyword.cloud}} se despliega en una región, pero puede abarcar varias zonas. Cada región contiene varias zonas, que representan dominios de error independientes.

A las subredes de una VPC se les asigna un _prefijo de dirección_, basado en la región y en la zona en la que se crean. En la tabla siguiente se muestran las regiones y las zonas disponibles. 

|   Ubicación     | Región | Punto final de API | Estado |
| ------- | :------: | :------: |:------: |
| Dallas | us-south | `us-south.iaas.cloud.ibm.com`| Disponible |
| Frankfurt | eu-de | `eu-de.iaas.cloud.ibm.com`| Disponible |
| Washington DC | us-east | `us-east.iaas.cloud.ibm.com`| Próximamente |
| Tokio | jp-tok | `jp-tok.iaas.cloud.ibm.com`| Próximamente |
| Londres | eu-gb | `eu-gb.iaas.cloud.ibm.com`| Próximamente |
| Sídney | au-syd | `au-syd.iaas.cloud.ibm.com`| Próximamente |

## VPC y subredes de IBM Cloud

Los clientes pueden (y suelen hacerlo, como método recomendado) dividir una VPC de IBM Cloud en subredes. Todas las subredes de una VPC de IBM Cloud pueden alcanzar a otro mediante el direccionamiento L3 privado de un direccionador implícito. No es necesario configurar ningún direccionador o ruta. 

Estos son algunos hechos sobre las subredes en VPC:

* Una subred consta de un rango de dirección IP que especifique. 
* Una subred está enlazada a una sola zona y no puede abarcar varias zonas o regiones.
* Una subred puede abarcar la totalidad de la zona en una VPC de IBM Cloud.
* Debe crear la VPC antes de crear las subredes en dicha VPC.
* El soporte de IPv6 no está disponible.
* Puede asociar o desasociar una VSI a una subred. (Requiere que añada una vNIC y seleccione un ancho de banda.)

Puede utilizar su propio rango de dirección IPv4 público (BYOIP) en su cuenta de VPC de IBM Cloud. Cuando utilice BYOIP, IBM Cloud deberá configurar las direcciones IPv4 en los recursos de IBM Cloud, que enviarán paquetes a y desde las direcciones proporcionadas. Por lo tanto, como resultado de utilizar el rango IPv4 proporcionado en IBM Cloud, las direcciones IP pueden estar expuestas al personal de soporte de IBM y a terceros como parte de su uso de este servicio.
{:important}

**Figura: Representación gráfica de una VPC que muestra zonas, regiones y subredes.**

![VPC de IBM Cloud ](images/vpc-experience.png)

## Utilización de prefijos de direcciones para subredes

Cada subred debe existir dentro de un prefijo de dirección.
 * Los prefijos de dirección de subred se asignan previamente para cada zona.
 * Para la nueva subred, puede elegir el rango de direcciones IP a partir de los prefijos de dirección existentes.
 * Si los prefijos de dirección de la zona no son adecuados, es posible editar uno de los prefijos de dirección existentes o añadir uno nuevo a la zona (utilizando la API, CLI o IU).
 
### Prefijos de dirección VPC predeterminados
 
Cuando se crea un nuevo VPC, los prefijos de dirección predeterminados se asignan como se indica a continuación, en función de la región y la zona:
 
* 'us-south-1' = '10.240.0.0/18'
* 'us-south-2' = '10.240.64.0/18'

Otras zonas o regiones pueden asignar distintos prefijos predeterminados, incluidos los siguientes:
 
 * 10.240.0.0/18
 * 10.240.64.0/18
 * 10.240.128.0/18
 
### Direccionar prefijos y la interfaz de usuario de la consola de IBM Cloud
 
Cuando crea una VPC utilizando la interfaz de usuario de la consola de IBM Cloud, el sistema selecciona el prefijo de dirección de manera automática y  le solicita que cree una subred en el prefijo predeterminado. Si el esquema de dirección no se adapta a sus requisitos, puede personalizar los prefijos de dirección después de crear la VPC. A continuación, puede crear subredes en los prefijos de direcciones personalizados y suprimir la subred que ha creado con el prefijo predeterminado.
 
Este método alternativo es necesario para utilizar BYOIP a través de la interfaz de usuario de la consola de IBM Cloud.
{:note}

### Direcciones IP disponibles

Estas son las direcciones IP disponibles, tal como se definen en **RFC 1918**:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

### Las direcciones siguientes son direcciones reservadas en una subred

En la lista, las direcciones IP presuponen que el rango de CIDR de la subred es 10.10.10.0/24:

  * Primera dirección en el rango de CIDR (10.10.10.0): Dirección de red
  * Segunda dirección en el rango de CIDR (10.10.10.1): Dirección de pasarela
  * Tercera dirección en el rango de CIDR (10.10.10.2): reservado por IBM
  * Cuarta dirección en el rango de CIDR (10.10.10.3): reservado por IBM para su uso futuro
  * Última dirección en el rango CIDR (10.10.10.255): Dirección de difusión de red

### Más información sobre la creación de una subred

Hay dos maneras de especificar una subred para la VPC:
  * Puede crear una subred proporcionando el tamaño de la subred que necesita, como el número de direcciones soportadas (por ejemplo 1024)
  * Puede crear una subred proporcionando un rango de CIDR (como 10.0.0.1/29)
  
A modo de ejemplo de cómo especificar un bloque 1024 utilizando CIDR, si especifica un rango de CIDR en lugar de un tamaño de subred, el bloque IPv4 `192.168.100.0/22` representa las direcciones IPv4 1024 de `192.168.100.0` a `192.168.103.255`.
{:tip}

### Más información sobre la supresión de una subred
  * No puede suprimir una subred si los recursos (como por ejemplo instancia de servidor virtual (VSI), IP flotante o pasarela pública) se están utilizando en dicha subred; primero debe suprimir los recursos.
  * No puede redimensionar una subred existente. Por ejemplo, 10.10.10.0/24 no se puede redimensionar a 10.5.1.3/20
