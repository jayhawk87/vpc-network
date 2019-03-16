---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-02-11"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Introdução à rede para a nuvem particular virtual


A IBM aceitará um número limitado de clientes para participar de um programa de Acesso Antecipado à VPC no início de abril de 2019, com o uso expandido sendo aberto nos meses seguintes. Se a sua organização gostaria de obter acesso à Nuvem Particular Virtual da IBM, preencha este [formulário de nomeação](https://cloud.ibm.com/vpc){: new_window} e um representante IBM entrará em contato com você com relação às próximas etapas.
{: important}

Para introdução à rede do {{site.data.keyword.cloud}} para nuvem particular virtual

1. Crie uma nuvem particular virtual.
2. Crie uma ou mais sub-redes na nuvem particular virtual em uma ou mais zonas.
3. Crie um gateway público (PGW) em uma sub-rede se você desejar que os recursos em sua sub-rede tenham acesso à Internet ou vice-versa.

## Pré-requisitos

 * **Permissões do usuário**: certifique-se de que seu usuário tenha permissões suficientes para criar e gerenciar recursos na VPC. Para obter uma lista de permissões necessárias, consulte [Concedendo permissões necessárias para usuários da VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).

## Usar a IU, a CLI ou a API de REST para provisionar os recursos de rede VPC

O fornecimento e o gerenciamento de recursos de rede VPC podem ser feitos por meio da IU, da CLI ou da API de REST.

* Para acesso por meio da interface com o usuário, efetue login no [Console do IBM Cloud ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")]( https://{DomainName}/vpc){: new_window}. Siga o [guia da IU](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console) se você precisar de ajuda.
* Para usar a interface da linha de comandos, use o plug-in [serviços de infraestrutura](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) da [CLI do IBM Cloud](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) e siga o exemplo [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).
* Para usuários mais avançados, é possível chamar as [APIs de REST](https://{DomainName}/apidocs/rias) diretamente. Siga o tutorial [código de exemplo](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) para introdução às APIs de REST.

## Próximas etapas

Depois que sua rede Nuvem Particular Virtual tiver sido provisionada, explore mais.

* [Criando e gerenciando instâncias de servidor virtual](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [Usando grupos de segurança](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Usando ACLs de rede](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(Beta) Usando VPN](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(Beta) Usando balanceadores de carga](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
