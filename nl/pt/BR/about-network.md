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
{:note: .note}
{:important: .important}
{:download: .download}

# Sobre a rede para a VPC
{: #about-networking-for-vpc}

Uma nuvem particular virtual (VPC) é uma rede virtual que está ligada à sua conta de cliente. Ela fornece segurança em nuvem com a capacidade de escalar dinamicamente, fornecendo controle de baixa granularidade sobre a infraestrutura virtual e sua segmentação de tráfego de rede.

Este documento abrange alguns conceitos de rede da forma como eles se aplicam ao {{site.data.keyword.cloud}} VPC. Os casos de uso e as características descritos incluem:

* regiões
* zonas
* endereços IP reservados
* gateways públicos
* interfaces vNIC
* sub-redes
* endereços IP flutuantes
* conexões VPN

## Visão geral

Para configurar a rede em sua VPC:
* Use um gateway público para o tráfego de Internet da sub-rede.
* Use um IP flutuante para o tráfego de Internet da VSI.
* Use uma VPN para conectividade externa segura.

Lembre-se:
* Alguns intervalos de CIDR de endereço de sub-rede são reservados pela IBM.
* Deve-se criar sua VPC antes de criar sub-redes dentro dessa VPC.
* O suporte ao IPv6 não está disponível.

Opcionalmente, é possível criar uma VPC de acesso Clássico para se conectar à sua Infraestrutura Clássica do IBM Cloud.
{:note}

Conforme mostrado na figura a seguir:
* As sub-redes em sua VPC podem se conectar à Internet pública por meio de um gateway público (PGW) opcional.
* É possível designar um endereço IP flutuante (FIP) a qualquer VSI para atingi-la por meio da Internet ou vice-versa.
* As sub-redes dentro do IBM Cloud VPC oferecem conectividade privada. Elas podem conversar umas com as outras por um link privado, por meio do roteador implícito. A configuração de rotas não é necessária.


**Figura: um cliente pode subdividir uma Nuvem Particular Virtual com sub-redes e cada sub-rede pode atingir a Internet pública, se desejado.**

![Conectividade da VPC](/images/vpc-connectivity-and-security.png)

## Terminologia

Esse [glossário](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary) contém definições e informações sobre termos usados nesse documento para o IBM Cloud VPC. Ao trabalhar com sua VPC, você precisará estar familiarizado com os conceitos básicos de _região_ e _zona_ da forma como eles se aplicam à sua implementação.

### Regiões

Uma região é uma abstração relacionada à área geográfica na qual uma VPC é implementada. Cada região contém múltiplas zonas, que representam domínios de falha independentes. Uma VPC do IBM Cloud pode abranger múltiplas zonas dentro de sua região designada.

### Zonas

Uma zona é uma abstração que se refere ao data center físico que hospeda os recursos de computação, de rede e de armazenamento, assim como o resfriamento e a energia relacionados, que fornece serviços e aplicativos. As zonas são isoladas umas das outras, de modo que nenhum único ponto de falha compartilhado é criado e são geradas tolerância a falhas melhorada e latência diminuída. Uma zona garante as propriedades a seguir:

 * Cada zona é um domínio de falha independente e é extremamente improvável que duas zonas em uma região falhem simultaneamente.
 * O tráfego entre zonas em uma região terá menos de 2 ms de latência.

## Características das sub-redes na VPC

Uma sub-rede consiste em um intervalo de endereços IP especificado (bloco CIDR). As sub-redes são ligadas a uma única zona e não podem abranger múltiplas zonas ou regiões. No entanto, uma sub-rede pode abranger a totalidade das abstrações da zona dentro de sua nuvem particular virtual. As sub-redes no mesmo IBM Cloud VPC estão conectadas umas às outras.


### Endereços IP reservados

Determinados endereços IP são reservados para uso pela IBM ao operar a nuvem particular virtual. Aqui estão os endereços reservados (esses endereços IP assumem que o intervalo de CIDR da sub-rede é 10.10.10.0/24):

  * Primeiro endereço no intervalo CIDR (10.10.10.0): endereço de rede
  * Segundo endereço no intervalo CIDR (10.10.10.1): endereço do gateway
  * Terceiro endereço no intervalo CIDR (10.10.10.2): reservado pela IBM
  * Quarto endereço no intervalo de CIDR (10.10.10.3): reservado pela IBM para uso futuro
  * Último endereço no intervalo CIDR (10.10.10.255): endereço de transmissão de rede

### Usar um gateway público para conectividade externa de uma sub-rede

Um **Gateway público (PGW)** permite que uma sub-rede (com todas as VSIs conectadas à sub-rede) se conecte à Internet. Observe que as sub-redes são privadas por padrão; no entanto, opcionalmente, é possível criar um PGW e conectar uma sub-rede a ele. Depois que uma sub-rede é conectada ao PGW, todas as VSIs nessa sub-rede podem se conectar à Internet.

O PGW usa a _NAT muitos para um_, o que significa que milhares de VSIs com endereços particulares usarão um endereço IP público para conversar com a Internet pública. O PGW não ativa a Internet para iniciar uma conexão com essas instâncias. Use a API para conectar e separar sub-redes para e do PGW.

A figura a seguir resume o escopo atual dos serviços de gateway.

![serviços de gateway](images/scope-of-gateway-services.png)

Um gateway público é criado em uma VPC, mas o gateway não faz nada até que seja anexado a uma sub-rede. É possível criar apenas um gateway público por zona, o que significa, por exemplo, que você pode ter três gateways públicos por VPC em um ambiente com três zonas.
{:note}

### Usar um endereço IP flutuante para conectividade externa de uma VSI 
**Endereços IP flutuantes** são endereços IP fornecidos pelo sistema e podem ser acessados por meio da Internet pública.

É possível reservar um endereço IP flutuante do conjunto de endereços IP flutuantes disponíveis fornecidos pela IBM e é possível associá-lo ou desassociá-lo de qualquer instância na mesma nuvem particular virtual por meio do vNIC dessa instância. Qualquer endereço IP flutuante pode ser associado a vários tipos de instâncias de servidor virtual (VSIs), por exemplo, um balanceador de carga ou um gateway VPN.

O endereço IP flutuante não pode ser associado a múltiplas interfaces. Deve-se especificar a interface na VSI que será associada a esse IP flutuante individual. Essa interface também terá um endereço IP privado. O sistema back-end executa operações _1-to-1 NAT_ entre o IP flutuante e o IP privado dessa interface. Um IP flutuante é liberado apenas quando você solicita especificamente a ação de liberação. 

**Notas:**
* **A associação de um endereço IP flutuante com uma VSI remove a VSI da NAT muitos para um do PGW mencionado anteriormente. **
* **Atualmente, o IP flutuante suporta apenas endereços IPv4.**
* **Não é possível trazer seu próprio endereço IP público para ser usado como um IP flutuante.**

Para obter mais informações sobre as operações NAT, consulte [o documento RFC relacionado à Internet ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Usar o VPN para conectividade externa segura
O serviço Virtual Private Network (VPN) está disponível para que os usuários se conectem a seu IBM Cloud VPC por meio da Internet com segurança.

**Recursos do VPN**
  * Capacidade de CRUD (criar, ler, atualizar e excluir) o serviço VPN (site IPSEC de site para site) para manipulação de transferência de dados.
  * Capacidade de associar o serviço VPN a um IBM Cloud VPC do cliente ou rede virtual.
  * Capacidade de identificar sub-redes no site do cliente, seja pela definição estática ou pelo roteamento dinâmico.
  * Suporte para cifras seguras, tais como SHA256, AES, 3DES, IKEv2
  * Confiabilidade de serviço integrada.
  * Monitoramento contínuo do funcionamento da conexão VPN.
  * Suporte para os modos do inicializador e do respondente; isto é, o tráfego pode ser iniciado por qualquer lado do túnel.

Para obter uma lista de limitações conhecidas e recursos não suportados atualmente, consulte o documento [Limitações conhecidas](/docs/infrastructure/vpc?topic=vpc-known-limitations).

## Aprenda mais

   * [Segurança do IBM VPC](/docs/infrastructure/vpc-network?topic=vpc-network-security-in-your-ibm-cloud-vpc)
   * [Endereços, regiões e sub-redes do IBM VPC](/docs/infrastructure/vpc-network?topic=vpc-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
