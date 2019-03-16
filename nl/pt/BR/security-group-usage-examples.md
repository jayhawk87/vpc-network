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

# Configurando grupos de segurança usando as APIs
{: #setting-up-security-groups-using-the-apis}

O exemplo a seguir demonstra como criar e gerenciar grupos de segurança, usando as APIs Regionais (RIAS) do {{site.data.keyword.cloud}}.

## Pré-requisitos

Para usar grupos de segurança, deve-se primeiro ter uma VPC do IBM Cloud em execução.

As instruções para criar uma VPC e uma sub-rede estão disponíveis em nosso tópico [Criando uma VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Etapa 1: criar um grupo de segurança

Crie um grupo de segurança denominado `my-security-group` em seu IBM Cloud VPC.

```
curl -X POST $rias_endpoint/v1/security_groups?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Salve o ID em uma variável para que possamos usá-lo posteriormente, por exemplo, a variável `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Etapa 2: incluir uma regra para permitir conexões SSH

Crie uma regra sobre o grupo de segurança que permitirá conexões de entrada na porta 22.

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

## Etapa 3: excluir o grupo de segurança (opcional)

Para limpar o grupo de segurança, ele não pode ser associado a nenhuma interface de rede e não pode ser referenciado por uma regra em um grupo de segurança diferente.

```
curl -X DELETE $rias_endpoint/v1/security_groups/$sg?version=2019-01-01 \
  -H "Authorization: $iam_token"
```
{: pre}
