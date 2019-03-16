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

# Configuración de las ACL de red mediante la CLI
{ #setting-up-network-acls-using-the-cli}

Mediante la funcionalidad de listas de control de acceso (ACL) disponible en la nube privada virtual de {{site.data.keyword.cloud}}, puede controlar todo el tráfico de entrada y salida relacionado con las cargas de trabajo empresariales críticas en la nube. AL igual que el grupo de seguridad, una ACL es un cortafuegos virtual incorporado. AL contrario que los grupos de seguridad, las reglas de ACL controlan el tráfico procedente de las subredes y el destinado a las mismas, no el procedente de las instancias y el destinado a las mismas.

La tabla siguiente resume algunas diferencias claves entre los grupos de seguridad y las ACL:

|  | Grupos de seguridad | Listas ACL    |
|-------------|-----------------|---------|
| Nivel de control  | Instancia de VSI    | Subred  |
| Estado   | Con estado - El tráfico de retorno está permitido de forma predeterminada | Sin estado - El tráfico de retorno se ha denegado de forma predeterminada y es necesario ofrecer el permiso explícitamente |
| Reglas soportadas | Solo utiliza reglas que permiten | Utiliza reglas que permiten y que deniegan|
| Cómo se aplican las reglas | Se tienen en cuenta todas las reglas | Las reglas se procesan en secuencia |
| Relación con el recurso asociado | Una instancia puede estar asociada a varios grupos de seguridad| Es posible asociar varias subredes a la misma ACL|

En el ejemplo de este documento se muestra cómo crear las ACL de red en la VPC, las cuales protegerán las subredes.

## API y CLI disponibles para las ACL

Puede configurar y gestionar las ACL a través de la API o a través de la CLI, lo que prefiera.

* Revise la [Consulta de API](https://{DomainName}/apidocs/rias) para ver los parámetros, el cuerpo de la solicitud y detalles de la respuesta correspondientes a cada API.

* Consulte este [ejemplo de CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) para ver detalles del mandato, pasos para instalar el plugin de CLI y pasos de requisito previo para utilizar la CLI de VPC.

## Cómo trabajar con ACL y reglas de ACL

Para que las ACL sean efectivas, debe crear reglas que determinen cómo manejar el tráfico de red de entrada y de salida. Puede crear varias reglas de entrada y de salida. Consulte el apartado sobre [Cuotas](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-quotas) para ver información sobre el número de reglas que puede crear.

* Con las reglas de entrada y de salida, puede permitir o denegar tráfico desde un rango de IP de origen o a un rango de IP de destino con los puertos y protocolos especificados.  
* Las reglas ACL se evalúan en secuencia. Las reglas de prioridad baja se evalúan solo cuando las reglas de prioridad más alta no coinciden.
* Las reglas de entrada están separadas de las reglas de salida.
* Los protocolos soportados actualmente son TCP, UDP e ICMP. También puede utilizar la opción **todo** para designar _todos_ los protocolos u _otros_ protocolos (si se especifica una regla con una prioridad superior).

Para ver información sobre cómo utilizar los protocolos ICMP, TCP y UDP en sus reglas de ACL, consulte [Visión general de los protocolos de comunicación de internet](https://{DomainName}/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols).

### Cómo adjuntar una lista de control de acceso (ACL) a una subred

Al adjuntar una ACL a una subred tiene dos opciones:

* Puede crear una nueva subred y especificar la ACL que desee adjuntar. Si no especifica ninguna ACL, se adjuntará una ACL de red predeterminada. La ACL predetermina permite todo el tráfico de entrada y salida de la subred.
* Puede adjuntar una ACL a una subred existente. Si ya se ha adjuntado otra ACL a esta subred, dicha ACL se desconectará antes de que se adjunte la nueva.

## Ejemplo de demostración de ACL

En el ejemplo siguiente podrá crear dos ACL y asociarlas a dos subredes mediante la interfaz de línea de mandatos (CLI). Este es el aspecto que tiene el caso de ejemplo:

![Un ejemplo de caso de ejemplo de ACL](images/vpc-acls.png)

Como muestra la figura, tiene dos servidores web que se ocupan de las solicitudes de Internet y dos servidores que desea ocultar al público. En este ejemplo, debe colocar los servidores en dos subredes separadas, 10.10.10.0/24 y 10.10.20.0/24 respectivamente, y debe permitir que los servidores web intercambien datos con los servidores de fondo. Además, desea permitir un tráfico de salida limitado procedente de los servidores de fondo.

### Reglas de ejemplo

Las reglas de ejemplo siguientes muestran cómo configurar las reglas de ACL en un caso de ejemplo básico, tal y como se ha descrito anteriormente.

Se recomienda otorgar a las reglas precisas una prioridad más alta que a las reglas generales. Por ejemplo, si tiene una regla que bloquea todo el tráfico de la subred 10.10.30.0/24 y esta se compara con una prioridad más alta, la regla precisa con una prioridad más baja para permitir el tráfico de 10.10.30.5 no se aplicará.
{:note}

**Reglas de ejemplo de ACL-1**:

| Entrada/Salida| Permitir/Denegar | IP de origen/destinación | Protocolo | Puerto | Descripción|
|--------------|-----------|------|------|------------------|-------|
| Entrada | Permitir | 0.0.0.0/0 | TCP| 80 | Permitir tráfico HTTP de Internet|
| Entrada | Permitir | 0.0.0.0/0 | TCP | 443 | Permitir tráfico HTTPS de Internet|
| Entrada | Permitir| 10.10.20.0/24 |todos| todos| Permitir todo el tráfico de entrada de la subred 10.10.20.0/24 donde se colocan los servidores de fondo|
| Entrada | Denegar| 0.0.0.0/0|todos| todos| Denegar toda la entrada de tráfico|
| Salida | Permitir | 0.0.0.0/0 | TCP|80 | Permitir tráfico HTTP a Internet|
| Salida | Permitir | 0.0.0.0/0 | TCP|443 | Permitir tráfico HTTPS a Internet|
| Salida | Permitir| 10.10.20.0/24 |todos| todos| Permitir todo el tráfico de salida en la subred 10.10.20.0/24 donde se colocan los servidores de fondo|
| Salida | Denegar| 0.0.0.0/0|todos| todos| Denegar toda la salida de tráfico|


**Reglas de ejemplo de ACL-2**:

| Entrada/Salida| Permitir/Denegar | IP de origen/destinación | Protocolo| Puerto | Descripción|
|--------------|-----------|------|------|------------------|--------|
| Entrada | Permitir| 10.10.10.0/24 |todos| todos| Permitir todo el tráfico de entrada de la subred 10.10.10.0/24 donde se colocan los servidores web|
| Entrada | Denegar| 0.0.0.0/0|todos| todos| Denegar toda la entrada de tráfico|
| Salida | Permitir | 0.0.0.0/0 | TCP| 80 | Permitir tráfico HTTP a Internet|
| Salida | Permitir | 0.0.0.0/0 | TCP| 443 | Permitir tráfico HTTPS a Internet|
| Salida | Permitir| 10.10.10.0/24 |todos| todos| Permitir todo el tráfico de salida en la subred 10.10.10.0/24 donde se colocan los servidores web|
| Salida | Denegar| 0.0.0.0/0|todos| todos| Denegar toda la salida de tráfico|

En este ejemplo solo se muestran casos generales. Es posible que también desee tener un control más detallado sobre el tráfico en sus casos de ejemplo:

* Puede tener un administrador de red que necesite acceso a la subred 10.10.10.0/24 desde una red remota con fines de funcionamiento. En ese caso, deberá permitir el tráfico SSH desde Internet a esta subred.
* Es posible que desee reducir el ámbito de protocolo que permite entre las dos subredes.

### Pasos de ejemplo

En los pasos del ejemplo siguiente se omiten los pasos de requisitos previos de utilizar la CLI para crear una VPC, proceso que debe llevarse a cabo en primer lugar. Para obtener más información, consulte [Creación de una VPC mediante la CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).


#### Paso 1. Crear las ACL

Estos son los mandatos de la CLI que puede utilizar para crear dos ACL llamadas `my_web_subnet_acl` y `my_backend_subnet_acl`:

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

La respuesta incluye los ID de ACL recién creados. Guarde los ID de ambas ACL para utilizarlos en mandatos posteriores. Podría utilizar variables llamadas `webacl` y `bkacl`, como en el siguiente ejemplo:

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### Paso 2. Recuperar las reglas de ACL predeterminadas

Antes de añadir nuevas reglas, recupere las reglas de ACL de entrada y salida predeterminadas para poder insertar nuevas reglas previamente.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

La respuesta muestra las reglas de entrada y salida predeterminadas que permiten todo el tráfico de IPv4 en todos los protocolos.

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

Guarde los ID de ambas reglas de ACL como variables para utilizarlos en mandatos posteriores. Por ejemplo, puede utilizar variables denominadas `inrule` y `outrule`:

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### Paso 3. Añadir nuevas reglas de ACL tal y como se describe

En esta sección se muestra cómo añadir primero las reglas de entrada y, a continuación, las reglas de salida.

Inserte nuevas reglas de entrada antes de la regla de entrada predeterminada.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

En cada paso, guarde el ID de la regla de ACL en una variable, que se utilizará en mandatos posteriores. Por ejemplo, puede utilizar el nombre de variable `acl200`:

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

Ahora añada la regla a `acl200`:

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

Añada más reglas hasta completar la configuración de la ACL, guardando cada ID como una variable.

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

Inserte nuevas reglas de salida antes de la regla de salida predeterminada.

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

#### Paso 4. Crear las dos subredes con la ACL recién creada

Creará dos subredes de modo que cada una de las ACL esté asociada a una de las nuevas subredes. 

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## Hoja de apuntes de la lista de mandatos

Para ver una lista completa de los mandatos de CLI de VPC disponibles para ACL:

```
ibmcloud is network-acls
```
{: pre}

Para ver la ACL y los metadatos, incluidas las reglas:

```
ibmcloud is network-acl $webacl
```
{: pre}

Para obtener la regla de ACL de entrada:

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## Regla `ping` de entrada de ejemplo

Si desea añadir una regla de ACL, utilice el mandato de ejemplo que se muestra a continuación para añadir una regla de entrada `ping` antes de la regla de entrada predeterminada:

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
