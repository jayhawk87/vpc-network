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
{:download: .download}

# VPC nos bastidores
{: #vpc-behind-the-curtain}

Esta página apresenta uma figura conceitual mais detalhada do que está acontecendo "nos bastidores" em relação à rede VPC. Espera-se que os leitores tenham alguma experiência de rede.

## Isolamento de rede

O isolamento de rede VPC ocorre em três níveis: 

* **Hypervisor**: as VSIs (instâncias do servidor virtual) são isoladas pelo próprio hypervisor. Uma VSI não poderá atingir diretamente outras VSIs hospedadas pelo mesmo hypervisor se elas não estiverem na mesma VPC.

* **Rede**: o isolamento ocorre no nível de rede por meio do uso de **virtual network identifiers** (VNIs). Esses identificadores são incluídos em todos os pacotes de dados que estão saindo do hypervisor, quando enviados por uma VSI.

* **Roteador**: a _função de roteador implícito_ fornece isolamento para cada VPC, fornecendo uma **virtual routing function** (VRF) e uma VPN com MPLS (multi-protocol label switching) no backbone de nuvem. A VRF de cada VPC tem um identificador exclusivo e esse isolamento permite que cada VPC tenha acesso à sua própria cópia do espaço de endereço IPv4. A VPN MPLS permite federar todas as bordas da nuvem: Infraestrutura clássica, Link direto e VPC.

## Prefixos de endereço

Os prefixos de endereço são as informações de resumo usadas pela função de roteamento implícito de uma VPC para localizar uma _VSI de destino_, independentemente da zona de disponibilidade na qual a VSI de destino está localizada. A função primária dos prefixos de endereço é otimizar o roteamento sobre a VPN MPLS, ao mesmo tempo em que evita casos de roteamento patológico. Todas as sub-redes criadas em uma VPC devem estar contidas em um prefixo de endereço, de modo que todas as VSIs em uma VPC sejam atingíveis por meio de todas as outras VSIs na VPC.

## Fluxos de pacote de dados e o roteador implícito

Seis tipos diferentes de fluxos de pacote de dados de VSI ocorrem em uma VPC. Em ordem crescente de complexidade, esses fluxos são:

* Intrassub-rede, intra-host (mesmo hypervisor)
* Intrassub-rede, inter-host
* Intersub-rede, intrazona
* Intersub-rede, interzona
* Serviço de VPC extra (para acesso ao IaaS ou CSE)
* Internet de VPC extra (para acesso à Internet)

Fluxos de dados **Intrassub-rede, intra-host**: esses são os mais simples. Os pacotes fluem entre as VSIs no hypervisor e nenhum pacote está saindo do hypervisor.

Fluxos de dados **Intrassub-rede, inter-host**: esses envolvem pacotes que saem do hypervisor. Como tal, cada pacote é identificado com o VNI (virtualized network identifier) apropriado para assegurar o isolamento de dados e, em seguida, é enviado para o hypervisor de destino que hospeda a VSI de destino. O hypervisor de destino remove o VNI e encaminha o pacote de dados para a VSI de destino.

Fluxos de dados **Intersub-rede, intrazona**: esses envolvem pacotes que utilizam a função de roteador implícito da VPC, que conecta todas as sub-redes criadas na VPC. Ela roteia o pacote de dados para o hypervisor de destino correto. Se o hypervisor de destino for diferente do hypervisor de origem, o pacote de dados será identificado com o VNI adequado e será enviado para o hypervisor de destino. Lá, o VNI é removido e o pacote de dados é encaminhado para a VSI de destino. (Essas últimas etapas são as mesmas que aqueles descritas no tipo de fluxo de dados anterior.)

Fluxos de dados **Intersub-rede, interzona**: para esses, a função de roteador implícito encaminha o pacote na VPN MPLS da VPC para trânsito ao longo do backbone de nuvem. Na zona de destino, a função de roteador implícito identifica o pacote de dados com o VNI. Em seguida, o pacote é encaminhado para o hypervisor de destino, no qual o VNI é (conforme descrito anteriormente) removido para que o pacote de dados possa ser encaminhado para a VSI de destino.

Fluxos de dados **Serviço de VPC extra**: os pacotes destinados aos serviços IaaS ou CSE (terminal em serviço do IBM Cloud) utilizarão a função de roteador implícito da VPC e também uma função de conversão de endereço de rede (NAT). A função de conversão substitui o endereço de VSI por um endereço IPv4, que identifica a VPC para o serviço IaaS ou CSE que está sendo solicitado.

Fluxos de dados **Internet de VPC extra**: os pacotes destinados à Internet são os mais complexos. Além de utilizar a função de roteador implícito da VPC, cada um desses fluxos também depende de uma das duas funções de conversão de endereço de rede (NAT) do roteador implícito:

  * um NAT explícito de um-para-muitos por meio de uma função de gateway público que atende a todas as sub-redes conectadas a ele.
  * um NAT de um-para-um designado a VSIs individuais.

Após a conversão NAT, o roteador implícito encaminha esses pacotes com destino à Internet para a Internet, usando o backbone de nuvem.

## Acesso clássico
{: #classic-access}

O recurso [**Acesso clássico**](/docs/infrastructure/vpc/classic-access.html) para VPC é realizado reutilizando o identificador VRF da Conta de infraestrutura clássica do {{site.data.keyword.cloud}} como o identificador VRF para VPC. Essa implementação permite que a função de roteador implícito da VPC se associe à mesma VPN MPLS que é usada pela Conta de infraestrutura clássica. Portanto, a VPC tem acesso a recursos clássicos e a qualquer outra coisa que for atingível por meio de conexões existentes de Link direto.
