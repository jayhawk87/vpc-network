---



copyright:
  years: 2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Beta) Usando balanceadores de carga no IBM Cloud VPC
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

O serviço de balanceador de carga distribui o tráfego entre múltiplas instâncias de servidor dentro da mesma região para a sua VPC do {{site.data.keyword.cloud}}.

**Agora esse serviço está na versão de liberação beta. Os recursos usados durante o período beta não estarão disponíveis após o fechamento de beta e nenhum suporte estará disponível. Direcione comentários e perguntas para o seu representante de vendas do IBM Cloud.**

## Balanceador de carga público
{: #public-load-balancer}

À instância de serviço do balanceador de carga é designado um nome completo do domínio (FQDN) acessível publicamente, que deve ser usado para acessar seus aplicativos hospedados por trás do serviço do balanceador de carga no {{site.data.keyword.cloud}} VPC. Esse nome de domínio pode ser registrado com um ou mais endereços IP públicos.

Ao longo do tempo, esses endereços IP públicos e o número de endereços IP públicos podem mudar com base nas atividades de manutenção e de ajuste de escalação, que são transparentes para os clientes. As instâncias do servidor de back-end (VSIs) que hospedam seu aplicativo devem ser executadas na mesma região e sob a mesma VPC.

## Listeners de front-end e conjuntos de back-end
Você pode definir até 10 listeners de front-end (portas do aplicativo) e mapeá-los para os conjuntos de back-end respectivos nos servidores de aplicativos de back-end. O nome completo do domínio (FQDN) designado para a instância de serviço do balanceador de carga e as portas do listener de front-end são expostos para a Internet pública. As solicitações recebidas de usuário são recebidas nessas portas.

Os protocolos suportados do listener de front-end são:
* HTTP (Protocolo de Transporte de Hipertexto)
* HTTPS
* TCP

Os protocolos suportados do conjunto de back-end são:
* HTTP (Protocolo de Transporte de Hipertexto)
* TCP

O tráfego HTTPS de entrada é finalizado no balanceador de carga para permitir a comunicação HTTP de texto sem formatação com o servidor de back-end.

Você pode conectar até 50 instâncias do servidor a um conjunto de back-end. Cada instância do servidor conectada deve ter uma porta configurada. A porta pode ou não ser a mesma que a porta do listener de front-end.

O intervalo de portas de 56500 a 56520 é reservado para propósitos de gerenciamento; essas portas não podem ser usadas como portas do listener de front-end.
{: note}

## Métodos de balanceamento de carga
{: #load-balancing-methods}

Os três métodos de balanceamento de carga a seguir estão disponíveis para distribuir tráfego entre servidores de aplicativos backend:

* **Round-robin:** round-robin é o método de balanceamento de carga padrão. Com esse método, o balanceador de carga encaminha conexões do cliente recebidas no modo round-robin para os servidores de back-end. Como resultado, todos os servidores de back-end recebem praticamente um número igual de conexões do cliente.

* **Round-robin ponderado:** com esse método, o balanceador de carga encaminha conexões do cliente recebidas para os servidores de
back-end em proporção à ponderação designada a esses servidores. A cada servidor é designado um peso padrão de 50, que pode ser customizado para qualquer valor entre 0 e 100.

Como um exemplo, se três servidores de aplicativos A, B e C tiverem ponderações customizadas para 60, 60 e 30 respectivamente, os servidores A e B receberão um número igual de conexões, enquanto o servidor C receberá metade desse número de conexões.

* **Conexões mínimas:** com esse método, a instância de servidor que atende ao número mínimo de conexões em um determinado momento recebe a próxima conexão do cliente.

**Características adicionais desses métodos:**

* A reconfiguração de um peso do servidor para '0' significa que nenhuma nova conexão é encaminhada para esse servidor, mas qualquer tráfego existente continuará a fluir, desde que esteja ativo. O uso de um peso de '0' pode ajudar a desligar um servidor corretamente e removê-lo da rotação de serviço.
* Os valores de ponderação do servidor são aplicáveis somente com o método round-robin ponderado. Eles são ignorados com métodos de balanceamento de carga de round-robin e de conexões mínimas.

## Verificações de funcionamento
{: #health-checks}

As definições de verificação de funcionamento são obrigatórias para conjuntos de back-end.

O balanceador de carga conduz verificações periódicas de funcionamento para monitorar o funcionamento das portas de back-end e encaminha o tráfego do cliente para elas de acordo. Se uma determinada porta do servidor de back-end for detectada como inoperante, nenhuma nova conexão será encaminhada para ela. O balanceador de carga continuará a monitorar o funcionamento de portas inoperantes e retomará seu uso se elas se tornarem funcionais novamente, o que significa passar com sucesso em duas tentativas de verificação de funcionamento consecutivas.

As verificações de funcionamento para as portas HTTP e TCP são conduzidas da seguinte forma:

* **HTTP:** uma solicitação de `HTTP GET` em uma URL pré-especificada é enviada para a porta do servidor de backend. A porta do servidor é marcada como funcional ao receber uma resposta `200 OK`. O caminho de funcionamento `GET` padrão é "/" por meio da UI e pode ser customizado.

* **TCP:** o Load Balancer tenta abrir uma conexão TCP com o servidor de back-end em uma porta TCP especificada. A porta do servidor será marcada como funcional se a tentativa de conexão for bem-sucedida e a conexão será encerrada.

O intervalo de verificação de funcionamento padrão é 5 segundos, o tempo limite padrão com relação a uma solicitação de verificação de funcionamento é 2 segundos e o número padrão de novas tentativas é 2.
{: note}

## Transferência de SSL e autorizações necessárias

Para todas as conexões HTTPS de entrada, o serviço do balanceador de carga finaliza a conexão SSL e estabelece uma comunicação HTTP de texto sem formatação com a instância do servidor de back-end. Com essa técnica, os handshakes SSL de uso intensivo da CPU e as tarefas de criptografia ou decriptografia são deslocados para longe das instâncias do servidor de back-end, permitindo que eles usem todos os seus ciclos de CPU para processar o tráfego de aplicativos.

Um certificado SSL é necessário para que o balanceador de carga execute tarefas de transferência de SSL. Você pode gerenciar os certificados SSL por meio do [IBM Certificate Manager ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Se você precisar de acesso de balanceador de carga ao certificado SSL na instância do gerenciador de certificados, crie uma autorização que forneça à instância de serviço do balanceador de carga acesso à instância do gerenciador de certificados que contém o certificado SSL. Você pode gerenciar essa autorização por meio de [Autorizações de identidade e acesso ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](/docs/services/iam/#/authorizations){: new_window}. Certifique-se de escolher **Serviço de infraestrutura** como o serviço de origem, **Balanceador de carga para VPC** como o tipo de recurso, **Certificate Manager** como o serviço de destino e designe a função de acesso ao serviço **Gravador**. Para criar um balanceador de carga, deve-se conceder a autorização **Todas as instâncias de recurso** para a instância de recurso de origem. A instância de serviço de destino pode ser **Todas as instâncias** ou pode ser sua instância de recurso de gerenciador de certificado específica.

Se a autorização necessária for removida, erros poderão ocorrer para o balanceador de carga.
{: note}

## Identity and Access Management (IAM)

É possível configurar políticas de acesso para uma instância do **Balanceador de carga para VPC**. Para gerenciar suas políticas de acesso de usuário, visite [Autorizações de identidade e acesso ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](/docs/services/iam/#/authorizations){: new_window}.

### Configurando as políticas de acesso do grupo de recursos para usuários

Para criar um balanceador de carga, você precisará de acesso a um grupo de recursos. O usuário que está criando um balanceador de carga deverá ter acesso apropriado ao grupo de recursos fornecido ou ao grupo de recursos padrão, se nenhum for fornecido.

1. Navegue para  ** Gerenciar > Conta > Usuários **. Você verá uma lista dos usuários com acesso à sua conta do IBM Cloud.
2. Selecione o nome do usuário para quem você deseja designar uma política de acesso. Se o usuário não for mostrado, clique em **Convidar usuários** para incluir o usuário em sua conta do IBM Cloud.
3. Selecione **Designar acesso**.
4. Selecione **Designar acesso em um grupo de recursos**.
5. Na lista suspensa **Grupo de recursos**, selecione o grupo de recursos desejado.
6. Na lista suspensa **Designar acesso a um grupo de recursos**, selecione o acesso desejado.
7. Na lista suspensa **Serviços**, selecione os serviços desejados.
9. Selecione **Designar** para designar a política de acesso ao grupo de recursos para o usuário.

### Configurando políticas de acesso de recurso para usuários

| Função de acesso à plataforma | Ação do balanceador de carga |
|-------------|-----|
| Administrator | Criar/visualizar/editar/excluir balanceador de carga |
| Editor | Criar/visualizar/editar/excluir balanceador de carga |
| Visualizador | Visualizar balanceador de carga |

1. Navegue para  ** Gerenciar > Conta > Usuários **. Você verá a lista de usuários com acesso à sua conta do IBM Cloud.
2. Selecione o nome do usuário para quem você deseja designar uma política de acesso. Se o usuário não for mostrado, clique em **Convidar usuários** para incluir o usuário em sua conta do IBM Cloud.
3. Selecione **Designar acesso**.
4. Selecione **Designar acesso a recursos**.
5. Na lista suspensa **Serviços**, selecione **Serviço de infraestrutura**.
6. Na lista suspensa **Tipo de recurso**, selecione **Balanceador de carga para VPC**.
7. Na lista suspensa **ID do balanceador de carga**, selecione um ID da instância do balanceador de carga ou use o valor padrão, **Todos os balanceadores de carga**.
8. Designe uma função de acesso à plataforma para o usuário.
9. Selecione **Designar** para designar a política de acesso para o usuário.

## APIs disponíveis

Para fazer as chamadas de API, deve-se usar algum formulário de cliente REST. Por exemplo, é possível usar o comando `curl` para recuperar todos os balanceadores de carga existentes:

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

A seção a seguir fornece detalhes sobre as APIs que podem ser usadas para balanceadores de carga em seu ambiente de VPC. Para a especificação integral, veja a [Referência da API do RIAS](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers).

| Descrição | API |
|-------------|-----|
| Cria e fornece um balanceador de carga | POST /load_balancers |
| Recupera todos os balanceadores de carga | GET /load_balancers |
| Recupera um balanceador de carga | GET /load_balancers/{id} |
| Exclui um balanceador de carga | DELETE /load_balancers/{id} |
| Atualiza um balanceador de carga | PATCH /load_balancers/{id} |
| Cria um listener | POST /load_balancers/{id}/listeners |
| Recupera todos os listeners do balanceador de carga | GET /load_balancers/{id}/listeners |
| Recupera um listener | GET /load_balancers/{id}/listeners/{listener_id} |
| Exclui um listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Atualiza um listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Cria um conjunto | POST /load_balancers/{id}/pools |
| Recupera todos os conjuntos do balanceador de carga | GET /load_balancers/{id}/pools |
| Recupera um conjunto | GET /load_balancers/{id}/pools/{pool_id} |
| Exclui um conjunto | DELETE /load_balancers/{id}/pools/{pool_id} |
| Atualiza um conjunto | PATCH /load_balancers/{id}/pools/{pool_id} |
| Cria um membro | POST /load_balancers/{id}/pools/{pool_id}/members |
| Recupera todos os membros do conjunto | GET /load_balancers/{id}/pools/{pool_id}/members |
| Recupera um membro |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Exclui um membro do conjunto | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Atualiza um membro | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Atualiza os membros do conjunto | PUT /load_balancers/{id}/pools/{pool_id}/members |

## Exemplo de balanceador de carga

No exemplo a seguir, você usará a API para criar um balanceador de carga na frente de duas instâncias do servidor da VPC (`192.168.100.5` e `192.168.100.6`) que executam um aplicativo da web atendendo na porta `80`. O balanceador de carga tem um listener de front-end, que permite o acesso ao aplicativo da web com segurança por meio de HTTPS. Em seguida, é possível usar a API para obter detalhes da instância do balanceador de carga após sua criação e para excluir a instância do balanceador de carga.

### Etapas de exemplo

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da [IU do IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), [CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) ou [API do RIAS](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) para provisionar uma VPC, sub-rede e instâncias. 


As etapas de exemplo do balanceador de carga também podem ser executadas usando a [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference).
{: note}

#### Etapa 1. Criar um balanceador de carga com as instâncias de listener, de conjunto e de servidor conectado (membros)

```bash
curl -H "Authorization: $iam_token" -X POST
$rias_endpoint/v1/load_balancers?version=2019-01-01 \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80, "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80, "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Saída de amostra:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d-7ce5-413b-aae4-90a2a88b7872.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004" }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004", "name": "example-pool" }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    }, "subnets": [ {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "name": "example-subnet" }
    ]
}
```
{: screen}

Salve o ID do balanceador de carga para ser usado nas próximas etapas, por exemplo, na variável `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### Etapa 2. Obter um balanceador de carga

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

Permita algum tempo para fornecimento do balanceador de carga. Quando estiver pronto, ele será configurado para os status `online` e `active`, como você verá na saída de amostra a seguir:

Saída de amostra:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004" }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004", "name": "example-pool" }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  }, "subnets": [ {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "name": "example-subnet" }
  ]
}
```
{: screen}

#### Etapa 3. Excluir um balanceador de carga

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## FAQs

Esta seção contém respostas para algumas perguntas mais frequentes sobre o serviço **Balanceador de carga para VPC**.

### Posso usar um nome DNS diferente para meu balanceador de carga?

O nome DNS designado automaticamente para o balanceador de carga não é customizável. No entanto, é possível incluir um registro CNAME (Nome Canônico) que aponte seu nome DNS preferencial para o nome DNS do balanceador de carga designado automaticamente. Por exemplo, o ID do balanceador de carga é `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, o nome DNS do balanceador de carga designado automaticamente é `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`. Seu nome DNS preferencial é `www.myapp.com`. É possível incluir um registro CNAME (por meio do provedor DNS que você usa para gerenciar o `myapp.com`) apontando `www.myapp.com` para o nome DNS do balanceador de carga `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`.

### Qual é o número máximo de listeners de front-end que posso definir com meu balanceador de carga?

10.

### Qual é o número máximo de instâncias do servidor que eu posso conectar ao meu conjunto de back-end?

50.

### Posso criar um balanceador de carga privado somente interno que seja acessível somente para clientes internos?  

Não, não neste momento.

### O balanceador de carga é escalável horizontalmente?  

Não, não neste momento.

### O que deverei fazer se estiver usando ACLs ou grupos de segurança nas sub-redes que são usadas para implementar o balanceador de carga?

Será necessário assegurar que as regras apropriadas da ACL ou do grupo de segurança estejam estabelecidas para permitir o tráfego de entrada para as portas do listener e as portas de gerenciamento configuradas (portas de 56500 a 56520). O tráfego entre o balanceador de carga e as instâncias de back-end também deve ser permitido.

### Por que estou recebendo uma mensagem de erro: `certificate instance not found`?

* O CRN da instância de certificado pode ser inválido.
* Você pode não ter concedido a Autorização de serviço para serviço. Veja a seção **Transferência de SSL** deste documento.

### Por que estou recebendo um código `401 Unauthorized Error`?

Verifique as políticas de acesso a seguir para seu usuário:
* Política de acesso do tipo de recurso do balanceador de carga
* Política de acesso do grupo de recursos

### Por que meu balanceador de carga está no estado `maintenance_pending`?

O balanceador de carga estará no estado `maintenance_pending` durante várias atividades de manutenção, tais como:
* Qualquer upgrade contínuo para tratar vulnerabilidades e aplicar correções de segurança
* Atividades de recuperação

### Por que eu preciso escolher múltiplas sub-redes durante o fornecimento?

Os dispositivos do balanceador de carga são implementados nas sub-redes selecionadas. É altamente recomendado escolher sub-redes de zonas diferentes para maior disponibilidade e redundância.
