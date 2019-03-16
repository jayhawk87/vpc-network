---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-01-23"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Criando uma conexão segura com um peer FortiGate remoto

Este documento é baseado no FortiGate 300C, versão de firmware v5.2.13, build762 (GA).

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da API ou da CLI do {{site.data.keyword.cloud}} para criar Nuvens Particulares Virtuais. Para obter mais informações, consulte [Introdução](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) e [Configuração de VPC com APIs](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis).

## Etapas de exemplo
A topologia para conexão com o peer do FortiGate remoto é semelhante à criação de uma conexão VPN entre duas VPCs. No entanto, um lado é substituído pela unidade FortiGate 300C.

![inserir descrição de imagem aqui](./images/vpc-vpn-fg-figure.png)

### Para criar uma conexão segura com o peer Fortigate remoto

Acesse **VPN \>IPsec \> Túneis** e crie um novo túnel customizado ou edite um túnel existente.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

Quando uma unidade FortiGate recebe uma solicitação de conexão de um peer VPN remoto, ela usa os parâmetros da Fase 1 do IPsec para estabelecer uma conexão segura e autenticar esse peer VPN. Em seguida, se a política de segurança permitir a conexão, a unidade FortiGate estabelecerá o túnel usando os parâmetros de Fase do IPsec e aplicará a política de segurança IPsec. Os serviços de gerenciamento de chaves, de autenticação e de segurança são negociados dinamicamente por meio do protocolo IKE.

**Para suportar essas funções, as etapas de configuração geral a seguir devem ser executadas pela unidade FortiGate:**

* Defina os parâmetros da Fase 1 que a unidade FortiGate requer para autenticar o peer remoto e estabelecer uma conexão segura.

* Defina os parâmetros da Fase 2 que a unidade FortiGate requer para criar um túnel VPN com o peer remoto.

* Crie políticas de segurança para controlar os serviços permitidos e a direção permitida de tráfego entre os endereços IP de origem e de destino.

Por padrão, alguns parâmetros típicos da Fase 1 e da Fase 2 são configurados.

### Configurando para a VPN do IBM Cloud VPC

Para se conectar ao recurso VPN do IBM Cloud VPC, recomendamos a configuração a seguir:

1. Escolha IKEv2 na autenticação;
2. Ative `DH-group 2` na proposta da Fase 1
3. Configure `lifetime = 36000` na proposta da Fase 1
4. Desativar PFS na proposta da Fase 2
5. Configure `lifetime = 10800` na proposta da Fase 2
6. Insira as informações de sua sub-rede na Fase 2
7. Os parâmetros restantes mantêm seus valores padrão.

![inserir descrição de imagem aqui](./images/vpc-vpn-fg-network.JPG)

### Parâmetros de rede

![inserir descrição de imagem aqui](./images/vpc-vpn-fg-authentication.JPG)

### Parâmetros de autenticação

![inserir descrição de imagem aqui](./images/vpc-vpn-fg-phase1.JPG)

### Parâmetros da Fase 1

![inserir descrição de imagem aqui](./images/vpc-vpn-fg-phase2.JPG)

### Parâmetros da Fase 2

## Para criar uma conexão segura com o IBM VPC local

Para criar uma conexão segura, você criará a conexão VPN dentro de seu VPC, que é semelhante ao exemplo 2 de VPC.

* Crie um gateway VPN em sua sub-rede da VPC junto com uma conexão VPN entre a VPC e a unidade FortiGate configurando `local_cidrs` como o valor de sub-rede na VPC e `peer_cidrs` como o valor de sub-rede no FortiGate.

**Nota:** o status do gateway aparece como `pending` enquanto o gateway VPN está sendo criado, e o status se torna `available` assim que a criação é concluída. A criação pode levar algum tempo.

![inserir descrição de imagem aqui](images/vpc-vpn-fg-connection.png)

### Verifique o status da conexão segura

É possível verificar o status de sua conexão por meio do console do IBM Cloud. Além disso, é possível tentar executar um `ping` de site para site usando os VSIs.

![inserir descrição de imagem aqui](images/vpc-vpn-fg-status.JPG)
