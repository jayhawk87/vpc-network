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

# Configurando a conectividade entre servidores da web em sua VPC
{: #setting-up-conectivity-between-wb-servers-in-your-vpc}

Este documento o conduz por algumas etapas de exemplo para configurar a comunicação entre dois servidores da web em sua VPC do {{site.data.keyword.cloud}} usando Python. Ele mostra como criar servidores de comunicação na mesma zona e em zonas diferentes.

## Pré-requisitos

Antes de poder configurar a comunicação entre os servidores da web em sua VPC, deve-se já ter criado uma VPC com DUAS sub-redes e pelo menos DUAS instâncias disponíveis em cada sub-rede. Será necessário saber os IDs da VPC, sub-redes e instâncias. Siga as etapas em nosso [guia para criar recursos VPC com a CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) se precisar de ajuda para definir essa configuração.

## Cenário 1: conectividade entre servidores na mesma zona, mas sub-redes diferentes

`vpc` contém o ID de vpc
`subnet1` contém o ID de sub-rede 1
`subnet2` contém o ID de sub-rede 2


## Parte 1: conectividade entre servidores na mesma zona e VPC

Depois de criar 2 ou mais instâncias na mesma sub-rede que tem um gateway público anexado, podemos testar sua conectividade hospedando um servidor com código HTML em uma instância, em seguida, executar `curl` para esse servidor e fazer download do arquivo HTML no navegador de seu computador ou de outros.

Suponha que você tenha `instance_1` e `instance_2` na mesma sub-rede que tem um gateway público e o mesmo grupo de segurança. É necessário incluir regras usando a API, a CLI ou a IU para a instância em grupos de segurança para permitir acesso de entrada. Se você hospedar seu servidor em `instance_1`, deverá incluir uma regra para o IP flutuante `instance_2`. Por exemplo, suponha que você tenha o servidor host `instance_1` com IPv4 configurado como `169.61.161.161` e `instance_2` com IPv4 configurado como `169.61.161.235` e tente OBTER o arquivo de `instance_1`. Isso deverá ser semelhante a:

![novas regras do grupo de segurança](images/security-group-ui-ex1.png)

* Para listar todos os grupos de segurança `ibmcloud is sgs`
* Aqui está uma solicitação de API que trabalha para obter o resultado anterior:

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id of instance_1}/rules?version=2019-01-01
-H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "protocol": "all",
  "ip_address": "169.61.161.235"
}'
```
{:pre}

1. Crie um arquivo `index.html` básico em `instance_1`. Aqui está um exemplo de um arquivo HTML básico:

```
<!DOCTYPE html>
<html>
<body>

<h1>Você conectou-se com êxito à instância</h1>
<p>Olá, mundo!</p>

</body>
</html>
```
{:pre}

2. No mesmo diretório em que o arquivo está localizado, execute o comando `python -m SimpleHTTPServer <port number>`. Será possível usar uma porta diferente se a porta padrão 8000 já estiver em uso.

3. Execute SSH em `instance_2`

Para procurar um endereço IP flutuante, use o comando `ibmcloud is ips` e identifique o `floating_ip_id` usando o comando `ibmcloud is ip $floating_ip_id`. É necessário ser capaz de identificar o endereço público.

4. Identifique o endereço IP de `instance_1`

5. Use `curl -X GET http://IP_address:Port_number/index.html` para solicitar o download de `index.html` do servidor host `instance_1`. (Por exemplo: `curl -X GET http://169.61.161.161:8000/index.html`)

6. É necessário ser capaz de ver a saída de `index.html`

7. (Para acesso ao servidor por meio de seu navegador do computador local) Inclua regras para todo o tráfego de entrada, conforme mostrado na figura a seguir:

![novas regras do grupo de segurança com toda a entrada](images/security-group-ui-ex2.png)

* Para incluir regras para permitir TODOS os protocolos e de cada origem em seus grupos de segurança

```
curl -X POST $rias_endpoint/v1/security_groups/{security_group_id}/rules/{instance_id}?version=2019-01-01 -H "Authorization:$iam_token" -d '{
  "direction": "inbound",
  "remote": {
    "cidr_block": "0.0.0.0/0"
  }
}'
```
{:pre}

**Exemplo de solicitação mais detalhada:**

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

8. Abra um navegador da web e, em seguida, insira `http://IP_address:Port_number/index.html`

9. É necessário ver o arquivo de índice mostrado no formato HTML.

## Parte 2: conectividade entre servidores de uma zona e uma VPC diferentes

Conecte a instância da outra sub-rede, que não tem o gateway público configurado, à instância com acesso à Internet. Você deseja configurar 2 tipos diferentes de instâncias:

* um servidor da web que manipula solicitações da Internet usando um gateway público
* um servidor de back-end que executa um aplicativo e armazena dados importantes, somente enviando tráfego interno – Gateway público desativado

Os grupos de segurança (SG) se aplicam às instâncias de servidor virtual (VSI) e as ACLs de rede se aplicam a sub-redes.

Portanto, para concluir essa tarefa geral, deve-se criar VSIs, criar sub-redes, criar regras de SG e ACL e, em seguida, conectar o SG à interface de rede de uma instância e conectar a ACL de rede a uma sub-rede específica.

* Para listar todas as regras no Grupo de segurança `subnet1`

```
curl -X GET $rias_endpoint/v1/security_groups/{security_group_id}/rules?version=2019-01-01 -H "Authorization:$iam_token" | json_pp
```
{:pre}

1. Remova a regra `allow-allow-all-traffic` no grupo de segurança do exercício de sub-rede na parte 1

```
curl -X DELETE $rias_endpoint/v1/security_groups/{security_group_id}/rules/{rule_id}?version=2019-01-01 -H "Authorization:$iam_token"
```
{:pre}

2. Crie uma nova VPC e, em seguida, crie uma sub-rede sem o gateway público anexado.

3. Crie um `instance_3` na nova sub-rede sem um gateway público (sem conexão de Internet) - você pode desejar alocar Núcleos de CPU e Memória maiores para a instância por motivos de back-end.

4. Inclua novas regras para `instance_3` no grupo de segurança da parte 1. Por exemplo: `instance_3` tem um IP flutuante de `169.61.181.25`

![novas regras do grupo de segurança com a instância de outra entrada de sub-rede](images/security-group-ui-ex3.png)

5. Use um comando do formato `curl -X GET http://IP_address:Port_number/index.html` para solicitar o download de `index.html` do servidor host instance_1. (Por exemplo: `curl -X GET http://169.61.161.161:8000/index.html`)

6. É necessário ser capaz de ver a saída de `index.html`

Isso significa que você tem uma conexão entre as instâncias 3 e 1, mas não 2.

## Próximas etapas

Você desejará proteger seus servidores fazendo algumas regras de ACL que controlam o tráfego de entrada, conforme mostrado [em nosso guia para usar ACLs na VPC](https://{DomainName}/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli).
