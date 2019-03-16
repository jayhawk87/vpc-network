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
{:download: .download}

# Configuración de grupos de seguridad mediante las API
{: #setting-up-security-groups-using-the-apis}

En el ejemplo siguiente se muestra cómo crear y gestionar grupos de seguridad mediante las API regionales de {{site.data.keyword.cloud}} (RIAS).

## Requisitos previos

Para utilizar grupos de seguridad, primero debe tener una VPC de IBM Cloud en ejecución.

Las instrucciones para la creación de una VPC y una subred están disponibles en nuestro tema sobre [Creación de una VCP](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Paso 1: Crear un grupo de seguridad

Cree un grupo de seguridad denominado `my-security-group` en la VPC de IBM Cloud.

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Guarde el ID en una variable para que se pueda utilizar posteriormente, por ejemplo en la variable `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Paso 2: Añadir una regla para permitir las conexiones SSH

Cree una regla en el grupo de seguridad que permita conexiones de entrada en el puerto 22.

```
curl -X POST $rias_endpoint/v1/security_groups/$sg/rules?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## Paso 3: Suprimir el grupo de seguridad (opcional)

Para limpiar el grupo de seguridad, este no puede estar asociado con ninguna de las interfaces de red ni puede hacer referencia al mismo ninguna regla de otro grupo de seguridad.

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
