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

# Acerca de las redes para VPC
{: #about-networking-for-vpc}

Una nube privada virtual (VPC) es una red virtual que está vinculada a su cuenta de cliente. Le proporciona seguridad en la nube con la capacidad de escalar dinámicamente, proporcionando un control muy completo sobre la infraestructura virtual y la segmentación de tráfico de red.

Este documento cubre algunos conceptos de red a medida que se aplican dentro de la VPC de {{site.data.keyword.cloud}}. Los casos de uso y las características que se describen incluyen:

* regiones
* zonas
* direcciones IP reservadas
* pasarelas públicas
* interfaces vNIC
* subredes
* direcciones IP flotantes
* conexiones VPN

## Visión general

Para configurar la red en su VPC:
* Utilice una pasarela pública para el tráfico de Internet de subred.
* Utilice una IP flotante para el tráfico de Internet de VSI.
* Utilice una VPN para la conectividad externa segura.

Recuerde que:
* Algunos rangos de CIDR de direcciones de subred están reservados por IBM.
* Debe crear la VPC antes de crear subredes en dicha VPC.
* El soporte de IPv6 no está disponible.

Si lo desea, puede crear una VPC de acceso clásico para conectarse a la infraestructura de IBM Cloud clásica.
{:note}

Tal como se muestra en la figura siguiente:
* Las subredes de la VPC se pueden conectar a Internet público a través de una pasarela pública opcional (PGW).
* Puede asignar una dirección IP flotante (FIP) a cualquier VSI para llegar a ella desde Internet o viceversa.
* Las subredes de la VPC de IBM Cloud ofrecen conectividad privada; pueden comunicarse entre sí a través de un enlace privado, a través del direccionador implícito. No es necesario configurar rutas.


**Figura: Un cliente puede subdividir una nube privada virtual con subredes y cada subred puede acceder a Internet público, si lo desea.**

![Conectividad de VPC](/images/vpc-connectivity-and-security.png)

## Terminología

Este [glosario](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary) contiene definiciones e información sobre los términos utilizados en este documento para la VPC de IBM Cloud. Cuando trabaje con la VPC, tendrá que estar familiarizado con los conceptos básicos de _región_ y _zona_ a medida que se apliquen a su despliegue.

### Regiones

Una región es una abstracción relacionada con el área geográfica en la que se despliega una VPC. Cada región contiene varias zonas, que representan dominios de error independientes. Una VPC de IBM Cloud puede abarcar varias zonas dentro de su región asignada.

### Zonas

Una zona es una abstracción que hace referencia al centro de datos físico que aloja los recursos de cálculo, de red y de almacenamiento, así como la refrigeración y la alimentación relacionadas, que proporcionan servicios y aplicaciones. Las zonas están aisladas entre sí para no disponer de un solo punto de anomalía, mejorar la tolerancia a errores y reducir la latencia. Una zona garantiza las propiedades siguientes:

 * Cada zona es un dominio independiente de tolerancia a errores y es muy poco probable que dos zonas de una región fallen simultáneamente.
 * El tráfico entre zona de una región tendrá menos de 2 ms de latencia.

## Características de las subredes en la VPC

Una subred consta de un rango de direcciones IP especificado (bloque CIDR). Las subredes están enlazadas a una única zona y no pueden abarcar varias zonas o regiones. Sin embargo, una subred puede abarcar la totalidad de las abstracciones de la zona dentro de su nube privada virtual. Las subredes en la misma VPC de IBM Cloud están conectadas entre sí.


### Direcciones IP reservadas

Algunas direcciones IP están reservadas para que IBM las utilice al trabajar con la nube privada virtual. A continuación, se muestran las direcciones IP reservadas (estas direcciones IP presuponen que el rango de CIDR de la subred es 10.10.10.0/24):

  * Primera dirección en el rango de CIDR (10.10.10.0): Dirección de red
  * Segunda dirección en el rango de CIDR (10.10.10.1): Dirección de pasarela
  * Tercera dirección en el rango de CIDR (10.10.10.2): reservado por IBM
  * Cuarta dirección en el rango de CIDR (10.10.10.3): reservado por IBM para su uso futuro
  * Última dirección en el rango CIDR (10.10.10.255): Dirección de difusión de red

### Utilizar una pasarela pública para la conectividad externa de una subred

Una **Pasarela pública (PGW)** permite a una subred (con todos los VSI adjuntos a la subred) conectarse a Internet. Tenga en cuenta que las subredes son privadas de forma predeterminada; sin embargo, puede crear opcionalmente una pasarela pública y adjuntar una subred a la misma. Una vez adjuntada la subred a la pasarela pública, todos los VSI de dicha subred se podrán conectar a Internet.

La pasarela pública utiliza _una conversión de direcciones de red de muchas a una_, lo que significa que miles de VSI con direcciones privadas utilizarán 1 dirección IP pública para comunicarse con Internet público. La pasarela pública no permite que Internet inicie una conexión con dichas instancias. Utilice la API para adjuntar y desconectar subredes a y desde la pasarela pública.

La figura siguiente resume el ámbito actual de los servicios de pasarela.

![servicios de pasarela](images/scope-of-gateway-services.png)

Se crea una pasarela pública en VPC, pero esta no hace nada hasta que se adjunta a una subred. Solo puede crear una pasarela pública por zona, lo que significa, por ejemplo, que puede tener 3 pasarelas públicas por VPC en un entorno con 3 zonas.
{:note}

### Utilice una dirección IP flotante para la conectividad externa de una VSI
Las **direcciones IP flotantes** son direcciones IP que proporciona el sistema y son accesibles desde Internet público.

Puede reservar una dirección IP flotante desde la agrupación de direcciones IP flotantes disponibles proporcionada por IBM y puede asociarla o desasociarla con cualquier instancia en la misma nube privada virtual, mediante la vNIC de dicha instancia. Es posible asociar cualquier dirección IP flotante a varios tipos de instancias de servidor virtual (VSI), por ejemplo, un equilibrador de carga o una pasarela VPN.

La dirección IP flotante no se puede asociar con varias interfaces. Debe especificar la interfaz en la VSI que se asociará con dicha IP flotante individual. Esta interfaz también tendrá una dirección IP privada. El sistema de fondo realizar operaciones de _conversión de direcciones de red de una a una_ entre la IP flotante y la IP privada de la interfaz. Solo se libera una IP flotante cuando solicita específicamente la acción de liberación. 

**Notas:**
* **La asociación de una dirección IP flotante con una VSI elimina la VSI de la conversión de direcciones de red de muchas a una de la pasarela pública mencionada anteriormente.**
* **Actualmente, la IP flotante solo da soporte a direcciones IPv4.**
* **No puede traer su propia dirección IP pública para utilizarla como IP flotante.**

Para obtener más información sobre las operaciones de NAT, consulte [el documento RFC de Internet relacionado ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilizar VPN para la conectividad externa segura
El servicio de red privada virtual (VPN) está disponible para que los usuarios se conecten a su VPC de IBM Cloud desde Internet de forma segura.

**Prestaciones de VPN**
  * Capacidad de crear, recuperar, actualizar y suprimir un servicio VPN (VPN de IPSEC de sitio a sitio) para gestionar la transferencia de datos.
  * Capacidad para asociar un servicio VPN a la red virtual o la VPC de IBM Cloud de un cliente.
  * Capacidad de identificar las subredes locales del cliente ya sea mediante definición estática o mediante direccionamiento dinámico.
  * Soporte para cifrados seguros como, por ejemplo, SHA256, AES, 3DES, IKEv2
  * Fiabilidad de servicio incorporada.
  * Supervisión continua del estado de conexión VPN.
  * Soporte para los modos de iniciador y respondedor; es decir, el tráfico se puede iniciar desde cualquier lado del túnel.

Para una lista de limitaciones y características conocidas que no están soportadas actualmente, consulte el documento [Limitaciones conocidas](/docs/infrastructure/vpc?topic=vpc-known-limitations).

## Más información

   * [Seguridad de IBM VPC](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [Direcciones, regiones y subredes de VPC de IBM](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
