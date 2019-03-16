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

# Configuración de la conectividad entre servidores web en la VPC
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

Este documento le guía por algunos pasos de ejemplo para configurar la comunicación entre dos servidores web de la VPC {{site.data.keyword.cloud}}, mediante Python. Le muestra cómo crear servidores de comunicación en la misma zona y en zonas diferentes.

## Requisitos previos

Para poder configurar la comunicación entre los servidores web de la VPC, debe haber creado una VPC con DOS subredes y al menos DOS instancias disponibles en cada subred. Deberá conocer los ID de la VPC, de las subredes y de las instancias. Siga los pasos de nuestra [guía para crear recursos de VPC con la CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) si necesita ayuda para implementar esta configuración.

## Caso de ejemplo 1: conectividad entre servidores de la misma zona pero en distintas subredes

`vpc` aloja el ID vpc
`subnet1` aloja el ID 1 de subred
`subnet2` aloja el ID 2 de subred


## Parte 1: conectividad entre servidores de la misma zona y VPC

Después de crear 2 o más instancias en la misma subred que tiene una pasarela pública conectada, podemos probar su conectividad alojando un servidor con código HTML en una instancia y ejecutando `curl` sobre dicho servidor y descargando el archivo HTML en los demás o desde el navegador del sistema.

Supongamos que tiene `instance_1` e `instance_2` en la misma subred que tiene la pasarela pública y el mismo grupo de seguridad. Debe añadir reglas mediante la API, la CLI o la interfaz de usuario para la instancia en los grupos de seguridad para permitir el acceso de entrada. Si aloja el servidor en `instance_1`, debe añadir una regla para la IP flotante `instance_2`. Por ejemplo, supongamos que tiene el servidor de alojamiento `instance_1` con IPv4 establecido en `169.61.161.161` e `instance_2` con IPv4 establecido en `169.61.161.235` y trata de obtener (GET) el archivo de `instance_1`. Debería quedar así:

![reglas de nuevo grupo de seguridad](images/security-group-ui-ex1.png)

* Para obtener una lista de todos los grupos de seguridad: `ibmcloud is sgs`
* A continuación se muestra una solicitud de API para obtener el resultado anterior:

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. Cree un archivo `index.html` básico en `instance_1`. A continuación se muestra un ejemplo de un archivo HTML básico:

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

2. En el directorio en el que se encuentra el archivo, ejecute el mandato `python -m SimpleHTTPServer <port number>`. Puede utilizar otro puerto si el puerto predeterminado 8000 ya se está utilizando.

3. SSH en `instance_2`

Para buscar una dirección IP flotante, utilice el mandato `ibmcloud is ips` e identifique `floating_ip_id` con el mandato `ibmcloud is ip $floating_ip_id`. Debería poder identificar la dirección pública.

4. Identifique la dirección IP de `instance_1`

5. Utilice `curl -X GET http://IP_address:Port_number/index.html` para solicitar que se descargue `index.html` desde el servidor de alojamiento `instance_1`. (Por ejemplo: `curl -X GET http://169.61.161.161:8000/index.html`)

6. Debería poder ver la salida de `index.html`

7. (Para acceder al servidor por medio del navegador del sistema local) Añada reglas para todo el tráfico de entrada, tal como se muestra en la figura siguiente:

![reglas del nuevo grupo de seguridad con todo entrada](images/security-group-ui-ex2.png)

* Para añadir reglas que permitan TODOS los protocolos y desde cada origen a los grupos de seguridad

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**Ejemplo más detallado de la solicitud:**

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

8. Abra un navegador web y especifique `http://IP_address:Port_number/index.html`

9. Debería ver el archivo de índice que se muestra en formato HTML.

## Parte 2: conectividad entre servidores de distintas zonas y VPC

Conecte la instancia de la otra subred, que no tiene configurada la pasarela pública, a la instancia con acceso a Internet. Debe configurar 2 tipos distintos de instancias:

* un servidor web que gestione las solicitudes procedentes de Internet mediante una pasarela pública
* un servidor de fondo que ejecuta una aplicación y almacena datos importantes, enviando solo tráfico interno; pasarela pública inhabilitada

Los grupos de seguridad (SG) se aplican a las instancias de servidor virtual (VSI) y las ACL de red se aplican a las subredes.

Por lo tanto, para completar esta tarea global, debe crear VSI, crear subredes, crear reglas de SG y de ACL y, a continuación, adjuntar el SG a la interfaz de red de una instancia y adjuntar la ACL de red a una subred específica.

* Para obtener una lista de todas las reglas del grupo de seguridad `subnet1`

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. Elimine la regla `inbound-allow-all-traffic` del grupo de seguridad del ejercicio de subred de la parte 1

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. Cree una nueva VPC y luego cree una subred sin la pasarela pública adjunta.

3. Cree una `instance_3` en la nueva subred sin una pasarela pública (sin conexión a Internet); desea asignar núcleos de CPU mayores y más memoria a la instancia para que gestione los procesos internos.

4. Añada reglas para `instance_3` en el grupo de seguridad de la parte 1. Por ejemplo: `instance_3` tiene la IP flotante `169.61.181.25`

![reglas de nuevo grupo de seguridad con instancia de otra entrada de subred](images/security-group-ui-ex3.png)

5. Utilice el mandato `curl -X GET http://IP_address:Port_number/index.html` para solicitar que se descargue `index.html` desde el servidor de alojamiento instance_1. (Por ejemplo: `curl -X GET http://169.61.161.161:8000/index.html`)

6. Debería poder ver la salida de `index.html`

Esto significa que tiene una conexión entre las instancias 3 y 1, pero no con la 2.

## Siguientes pasos

Desea proteger sus servidores creando reglas de ACL que controlen el tráfico de entrada, tal como se muestra en [nuestra guía sobre cómo utilizar ACL en VPC](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli).
