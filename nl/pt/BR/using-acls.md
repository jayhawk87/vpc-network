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

# Configurando ACLs de rede usando a CLI
{ #setting-up-network-acls-using-the-cli}

Usando a funcionalidade da Lista de Controle de Acesso (ACL) disponível no {{site.data.keyword.cloud}} Virtual Private Cloud, é possível controlar todo o tráfego recebido e de saída relacionado às cargas de trabalho de negócios críticos na nuvem. Semelhante ao grupo de segurança, uma ACL é um firewall virtual integrado. Em contraste com grupos de segurança, as regras de ACL controlam o tráfego para e das sub-redes, em vez de para e das instâncias.

A tabela a seguir resume algumas diferenças chave entre grupos de segurança e ACLs:

|  | Grupos de Segurança | ACLs    |
|-------------|-----------------|---------|
| Nível de controle  | Instância da VSI    | Sub-rede  |
| Estado   | Stateful: o tráfego de retorno é permitido por padrão | Stateless: o tráfego de retorno é negado por padrão e precisa ser explicitamente permitido |
| Regras suportadas | Usa somente regras de permissão | Usa regras de permissão e negação|
| Como as regras são aplicadas | Todas as regras são consideradas | As regras são processadas na sequência |
| Relacionamento com o recurso associado | Uma instância pode ser associada a múltiplos grupos de segurança|Múltiplas sub-redes podem ser associadas à mesma ACL|

O exemplo fornecido neste documento mostra como criar ACLs de rede em sua VPC, o que protegerá suas sub-redes.

## APIs e CLIs estão disponíveis para ACLs

É possível configurar e gerenciar ACLs por meio da API ou da CLI, conforme você preferir.

* Veja a [Referência de API](https://{DomainName}/apidocs/rias) para obter os parâmetros, corpo da solicitação e detalhes de resposta para cada API.

* Veja este [exemplo de CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) para obter detalhes do comando, etapas para instalar o plug-in da CLI e as etapas de pré-requisito de uso da CLI da VPC.

## Trabalhando com ACLs e regras de ACL

Para tornar suas ACLs efetivas, você criará regras que determinam como manipular o tráfego de rede de entrada e saída. É possível criar múltiplas regras de entrada e de saída. Veja [Cotas](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-quotas) para obter informações específicas sobre quantas regras é possível criar.

* Com as regras de entrada e saída, é possível permitir ou negar o tráfego de um intervalo de IP de origem ou para um intervalo de IP de destino, com protocolos e portas especificados.  
* As regras de ACL são consideradas em sequência. As regras de baixa prioridade são avaliadas somente quando as regras de prioridade mais alta não correspondem.
* As regras de entrada são separadas das regras de saída.
* Os protocolos suportados atualmente são TCP, UDP e ICMP. Também é possível usar a opção **todos** para designar _todos_ os protocolos ou _outros_ protocolos (caso uma regra com uma prioridade mais alta seja especificada).

Para obter informações relevantes sobre como usar os protocolos ICMP, TCP e UDP em suas regras de ACL, veja [Entendendo protocolos de comunicação da Internet](https://{DomainName}/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols).

### Anexando uma ACL a uma sub-rede

Você tem duas opções ao anexar uma ACL a uma sub-rede:

* É possível criar uma nova sub-rede e especificar uma ACL para anexar. Se você não especificar uma ACL, uma ACL de rede padrão será anexada. A ACL padrão permite todo o tráfego de entrada destinado a essa sub-rede e todo o tráfego de saída dessa sub-rede.
* É possível anexar uma ACL a uma sub-rede existente. Se outra ACL já estiver anexada a essa sub-rede, essa ACL será removida antes que a nova ACL seja anexada.

## Exemplo de demo de ACL

No exemplo a seguir, você poderá criar duas ACLs e associá-las a duas sub-redes, usando a interface da linha de comandos (CLI). Aqui está como o cenário se parece:

![Um cenário de ACL de exemplo](images/vpc-acls.png)

Como a figura ilustra, você tem dois servidores da web lidando com as solicitações da Internet e de dois servidores de back-end que você deseja ocultar do público. Nesse exemplo, você coloca os servidores em duas sub-redes separadas, 10.10.10.0/24 e 10.10.20.0/24, respectivamente, e é necessário permitir que os servidores da web troquem dados com os servidores de back-end. Além disso, você deseja permitir tráfego de saída limitado dos servidores de back-end.

### Regras de exemplo

As regras de exemplo a seguir mostram como configurar as regras de ACL para um cenário básico, conforme descrito anteriormente.

Como melhor prática, recomenda-se fornecer às regras de baixa granularidade uma prioridade mais alta do que as regras de alta granularidade. Por exemplo, se você tiver uma regra bloqueando todo o tráfego da sub-rede 10.10.30.0/24 e ela for correspondida com uma prioridade mais alta, a regra de baixa granularidade com prioridade mais baixa para permitir o tráfego de 10.10.30.5 nunca será aplicada.
{:note}

**Regras de exemplo da ACL-1**:

| Entrada/saída| Permitir/negar | IP de origem/destino | Protocolo | Porta | Descrição|
|--------------|-----------|------|------|------------------|-------|
| Entrada | Permitir | 0.0.0.0/0 | TCP| 80 | Permitir tráfego HTTP da Internet|
| Entrada | Permitir | 0.0.0.0/0 | TCP | 443 | Permitir tráfego HTTPS da Internet|
| Entrada | Permitir| 10.10.20.0/24 |todos| todos| Permitir todo o tráfego de entrada da sub-rede 10.10.20.0/24 em que os servidores de back-end são colocados|
| Entrada | Negar| 0.0.0.0/0|todos| todos| Negar todas as outras entradas de tráfego|
| Saída | Permitir | 0.0.0.0/0 | TCP|80 | Permitir tráfego HTTP para a Internet|
| Saída | Permitir | 0.0.0.0/0 | TCP|443 | Permitir tráfego HTTPS para a Internet|
| Saída | Permitir| 10.10.20.0/24 |todos| todos| Permitir todo o tráfego de saída para a sub-rede 10.10.20.0/24 em que os servidores de back-end são colocados|
| Saída | Negar| 0.0.0.0/0|todos| todos| Negar todas as outras saídas de tráfego|


**Regras de exemplo de ACL-2**:

| Entrada/saída| Permitir/negar | IP de origem/destino | Protocolo| Porta | Descrição|
|--------------|-----------|------|------|------------------|--------|
| Entrada | Permitir| 10.10.10.0/24 |todos| todos| Permitir todo o tráfego de entrada da sub-rede 10.10.10.0/24 em que os servidores da web são colocados|
| Entrada | Negar| 0.0.0.0/0|todos| todos| Negar todas as outras entradas de tráfego|
| Saída | Permitir | 0.0.0.0/0 | TCP| 80 | Permitir tráfego HTTP para a Internet|
| Saída | Permitir | 0.0.0.0/0 | TCP| 443 | Permitir tráfego HTTPS para a Internet|
| Saída | Permitir| 10.10.10.0/24 |todos| todos| Permitir todo o tráfego de saída para a sub-rede 10.10.10.0/24 em que os servidores da web são colocados|
| Saída | Negar| 0.0.0.0/0|todos| todos| Negar todas as outras saídas de tráfego|

Esse exemplo ilustra somente os casos gerais. Em seus cenários, também é possível você querer ter mais controle granular sobre o tráfego:

* Você poderia ter um administrador de rede que precisa acessar a sub-rede 10.10.10.0/24 de uma rede remota para fins de operação. Nesse caso, será necessário permitir o tráfego SSH da Internet para essa sub-rede.
* Talvez você queira limitar o escopo do protocolo permitido entre suas duas sub-redes.

### Etapas de exemplo

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da CLI para criar uma VPC, que deve ser feito primeiro. Para obter mais informações, veja [Criando uma VPC usando a CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).


#### Etapa 1. Criar as ACLs

Aqui estão os comandos da CLI que podem ser usados para criar duas ACLs, denominadas `my_web_subnet_acl` e `my_backend_subnet_acl`:

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

A resposta inclui os IDs de ACL recém-criados. Salve os IDs de ambas as ACLs para serem usados em comandos posteriores. Você poderia usar variáveis denominadas `webacl` e `bkacl`, como esta:

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### Etapa 2. Recuperar as regras de ACL padrão

Antes de incluir novas regras, recupere as regras de ACL de entrada e de saída padrão para que você possa inserir novas regras antes delas.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

A resposta mostra as regras de entrada e saída padrão que permitem todo o tráfego IPv4 em todos os protocolos.

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

Salve os IDs de ambas as regras de ACL como variáveis, você as usará em comandos posteriores. Por exemplo, você poderia usar variáveis denominadas `inrule` e `outrule`:

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### Etapa 3. Incluir novas regras de ACL, conforme descrito

Esta seção mostra a inclusão de regras de entrada primeiro e, em seguida, a inclusão de regras de saída.

Insira novas regras de entrada antes da regra de entrada padrão.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

Em cada etapa, salve o ID da regra de ACL em uma variável, ela será usada em comandos posteriores. Por exemplo, você poderia usar o nome de variável `acl200`:

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

Agora inclua a regra em `acl200`:

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

Inclua mais regras até que a configuração da ACL seja concluída, salvando cada ID como uma variável.

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

Insira novas regras de saída antes da regra de saída padrão.

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

#### Etapa 4. Crie as duas sub-redes com a ACL recém-criada

Você criará duas sub-redes para que cada uma de suas ACLs seja associada a uma das novas sub-redes. 

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## Folha de dicas da lista de comandos

Para mostrar uma lista completa dos comandos da CLI da VPC disponíveis para ACLs:

```
ibmcloud is network-acls
```
{: pre}

Para ver sua ACL e seus metadados, incluindo regras:

```
ibmcloud is network-acl $webacl
```
{: pre}

Para obter a regra de ACL de entrada padrão:

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## Exemplo de regra de entrada `ping`

Para incluir uma regra de ACL, aqui está um exemplo de comando para incluir uma regra de entrada `ping` antes da regra de entrada padrão:

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
