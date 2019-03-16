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

# (Beta) Utilización de la VPN con la VPC
{: #--beta-using-vpn-with-your-vpc}

El servicio de VPN de la VPC de {{site.data.keyword.cloud}} le permite conectar redes privadas de forma segura. Puede utilizar la VPN para configurar un túnel de sitio a sitio de IPsec entre la VPC y la red privada virtual local u otra VPC.

Para el release de la VPC de {{site.data.keyword.cloud}} actual solo se admite el direccionamiento basado en políticas.

**Este servicio ahora dispone de una versión de release beta. Los recursos utilizados durante el período beta no estarán disponibles después del cierre de la versión beta y no se admitirán. Indique sus comentarios y preguntas al representante de ventas de IBM Cloud.**

## Características

* IKEv1 e IKEv2
* Algoritmos de autenticación: `md5`, `sha1`, `sha256`
* Algoritmos de cifrado: `3des`, `aes128`, `aes256`
* Grupos de Diffie-Hellman (DH): 2, 5, 14
* Modalidad de negociación IKE: principal
* Protocolo de transformación IPSec: ESP
* Modalidad de encapsulación IPSec: túnel
* Confidencialidad directa total (PFS)
* Detección de igual inactivo
* Direccionamiento: basado en políticas
* Modalidad de autenticación: clave precompartida
* Soporte de HA

## API disponibles

En la sección siguiente se proporcionan detalles sobre las API que puede utilizar para la VPN en el entorno VPC de IBM Cloud. Consulte la página [API REST de VPC](https://{DomainName}/apidocs/rias#retrieves-all-ike-policies) para obtener más detalles.

### Pasarelas VPN y conexiones VPN

| Descripción | API |
|----------------------------|-------------|
| Crea una pasarela VPN | POST /vpn_gateways |
| Recupera pasarelas VPN | GET /vpn_gateways |
| Recupera una pasarela VPN | GET /vpn_gateways/{id} |
| Suprime una pasarela VPN | DELETE /vpn_gateways/{id} |
| Actualiza una pasarela VPN | PATCH /vpn_gateways/{id} |
| Crea una nueva conexión VPN | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Recupera conexiones VPN | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Recupera una pasarela VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Suprime una conexión VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Actualiza una conexión VPN | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Recupera todos los CIDR locales para una conexión VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Suprime un CIDR local a partir de una conexión VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Comprueba si existe un CIDR local específico en una conexión VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Establece un CIDR local en una conexión VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Recupera todos los CIDR de igual para una conexión VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Suprime un CIDR de igual de una conexión VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Comprueba si existe un CIDR de igual específico en una conexión VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Establece un CIDR de igual en una conexión VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### Políticas de IKE

| Descripción | API |
|-----------------------------|--------------|
| Recupera todas las políticas IKE | GET /ike_policies |
| Crea una política IKE | POST /ike_policies |
| Suprime una política IKE | DELETE /ike_policies/{id} |
| Recupera una política IKE | GET /ike_policies/{id} |
| Actualiza una política IKE | PATCH /ike_policies/{id} |
| Recupera todas las conexiones que utilizan la política IKE especificada | GET /ike_policies/{id}/connections |

### Políticas de IPsec

| Descripción | API |
|---------------------|-------------|
| Recupera todas las políticas de IPSec | GET /ipsec_policies |
| Crea una política de IPSec | POST /ipsec_policies |
| Suprime una política de IPSec | DELETE /ipsec_policies/{id} |
| Suprime una política de IPSec | GET /ipsec_policies/{id} |
| Actualiza una política de IPSec | PATCH /ipsec_policies/{id} |
| Recupera todas las conexiones que utilizan la política de IPsec especificada | GET /ipsec_policies/{id}/connections |

## Ejemplo de VPN

En el ejemplo siguiente, podrá conectar dos VPC utilizando VPN, lo que significa que puede conectar subredes en dos VPC independientes como si fueran una única red. Las direcciones IP de las subredes no deben solaparse.
Este es el aspecto que presenta el caso de ejemplo (con algunas VM añadidas en cada VPC): ![Un caso de ejemplo de VPN de muestra](images/vpc-vpn.png)

### Pasos de ejemplo

Los pasos de ejemplo que siguen omiten los requisitos previos de utilizar la API o CLI de IBM Cloud para crear VPC. Para obtener más información, consulte [Guía de inicio](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) y [Configuración de VPC con API](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

De forma opcional, puede crear una pasarela VPN utilizando la interfaz de usuario. Encontrará los pasos que debe seguir en el documento [Guía de aprendizaje de la consola](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

#### Paso 1. Crear una pasarela VPN en la subred de VPC

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Salida de ejemplo:
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

Asegúrese de que se guarden los campos siguientes para los pasos que aparecen a continuación.
* `id`. Este es el ID de pasarela VPN y se denominará `$gwid1`.
* `address`. Esta es la dirección de IP pública de la pasarela VPN y se denominará `$gwaddress1`.

NOTA: El estado de la pasarela aparece como `pendiente` mientras se crea la pasarela VPN y el estado pasa a ser `disponible` una vez completada la creación. La creación puede tardar un poco. Puede comprobar su estado con el mandato siguiente:

```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-01-01
```
{: codeblock}

#### Paso 2. Crear una segunda pasarela VPN en otra VPC

```bash
curl -H "Authorization: $iam_token" -X POST $rias_endpoint/v1/vpn_gateways?version=2019-01-01 \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Salida de ejemplo:
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

Asegúrese de guardar los campos que aparecen a continuación para los pasos siguientes.
* `id`. Este es el ID de pasarela VPN y se denominará `$gwid2`.
* `address`. Esta es la dirección de IP pública de la pasarela VPN y se denominará `$gwaddress2`.


#### Paso 3. Crear una conexión VPN desde la primera pasarela de VPN a la segunda, estableciendo `local_cidrs` en la subred en la VPC uno y `peer_cidrs` en la subred en la VPC dos

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

Salida de ejemplo:
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

#### Paso 4. Crear una conexión VPN desde la segunda pasarela de VPN a la primera, estableciendo `local_cidrs` en la subred en la VPC dos y `peer_cidrs` en la subred en la VPC uno

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

Salida de ejemplo:
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

#### Paso 5. Verificar la conectividad

Una vez se haya establecido la conexión VPN, podrá acceder a las instancias de la subred dos desde la subred uno y viceversa.

Puede comprobar el estado de la conexión VPN como se indica a continuación:
```bash
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-01-01
```
{: codeblock}

Salida de ejemplo:
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

## Cuotas

Consulte el tema sobre [Cuotas de VPC](/docs/infrastructure/vpc?topic=vpc-quotas#vpn-quotas) para ver las cuotas de VPN.

## Negociación automática de políticas

El uso de las políticas IPSec e IKE para configurar una conexión VPN es opcional. Cuando no se selecciona ninguna política, las propuestas predeterminadas se seleccionan automáticamente para un proceso denominado _negociación automática_. Las propuestas que se utilizan como parte de la negociación automática son las siguientes.

Como iniciador, IBM Cloud utiliza **IKEv2**. Como respondedor, IBM Cloud acepta **IKEv1** e **IKEv2**.
{: note}

### Negociación automática de IKE (fase 1)

Las opciones de grupo DH, autenticación y cifrado se pueden utilizar en cualquier combinación:

|    | Cifrado | Autenticación | Grupo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Negociación automática de IPSec (fase 2)

Las opciones de autenticación y cifrado se pueden utilizar en cualquier combinación:

|    | Cifrado | Autenticación | Grupo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | inhabilitado  |
| 2  | aes256 | sha256 | inhabilitado  |
| 3  | 3des   | md5    | inhabilitado  |

## Preguntas más frecuentes

**Cuando creo una pasarela de VPN, ¿puedo crear al mismo tiempo conexiones de VPN?**

Si utiliza la API o la CLI, las conexiones de VPN se deben crear después de haber creado la pasarela de VPN. En la consola de IBM Cloud, puede crear la pasarela y una conexión al mismo tiempo.

**Si suprimo una pasarela de VPN que tenga conexiones de VPN adjuntas, ¿qué ocurrirá con las conexiones?**

Las conexiones de VPN se suprimen junto con la pasarela de VPN.

**¿Se suprimirán las políticas IKE o IPSec si suprimo una pasarela o conexión de VPN?**

No, las políticas IKE e IPSec se pueden aplicar a varias conexiones.

**¿Qué le ocurre a una pasarela de VPN si intento suprimir la subred en la que se encuentra la pasarela?**

La subred no se puede suprimir si hay instancias presentes, incluida la pasarela VPN.

**¿Existen políticas IPSec e IKE predeterminadas?**

Cuando crea una conexión VPN sin que se haga referencia a un ID de política (IKE o IPSec), se utiliza la negociación automática.

**¿Ofrece suporte la VPN para VPC a las configuraciones de alta disponibilidad?**

Sí, así es.
