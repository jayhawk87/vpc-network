---

copyright:
  years: 2018, 2019
  
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

# Trabalhando com intervalos de endereço IP, prefixos de endereço, regiões e sub-redes 
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}

Este documento fornece uma visão geral dos intervalos de endereço IP disponíveis (ou restritos) e das ações do usuário pertencentes a regiões e sub-redes e a prefixos de endereço. Determinados intervalos de endereço IP estão indisponíveis porque podem estar em uso pelo IBM Cloud.  

Ao criar uma sub-rede na sua VPC do {{site.data.keyword.cloud}}, um _prefixo de endereço_ é designado automaticamente, com base na região e na zona dentro da qual você criou sua VPC. É possível usar o prefixo de endereço padrão ou é possível mudar manualmente seu prefixo de endereço, incluindo um endereço BYOIP, em muitos casos. 

## VPC do IBM Cloud e regiões

Uma VPC do {{site.data.keyword.cloud}} é implementada em uma região, mas pode abranger múltiplas zonas. Cada região contém múltiplas zonas, que representam domínios de falha independentes.

As sub-redes em uma VPC são designadas a um _prefixo de endereço_, com base na região e na zona na qual elas são criadas. A tabela a seguir mostra as regiões e zonas disponíveis. 

|   Local        | Região | Terminal de API | Status |
| ------- | :------: | :------: |:------: |
| Dallas | us-south | `us-south.iaas.cloud.ibm.com`| Disponível |
| Frankfurt | eu-de | `eu-de.iaas.cloud.ibm.com`| Disponível |
| Washington DC | us-east | `us-east.iaas.cloud.ibm.com`| Em breve |
| Tóquio | jp-tok | `jp-tok.iaas.cloud.ibm.com`| Em breve |
| Londres | eu-gb | `eu-gb.iaas.cloud.ibm.com`| Em breve |
| Sydney | au-syd | `au-syd.iaas.cloud.ibm.com`| Em breve |

## VPC do IBM Cloud e sub-redes

Os clientes podem (e, geralmente fazem, como uma melhor prática) dividir um IBM Cloud VPC em sub-redes. Todas as sub-redes em um IBM Cloud VPC podem alcançar umas às outras, embora o roteamento L3 privado seja feito por um roteador implícito. Não é necessário configurar nenhum roteador ou rota. 

Fatos úteis sobre as sub-redes na VPC:

* Uma sub-rede consiste em um intervalo de endereços IP que você especifica. 
* Uma sub-rede está ligada a uma única zona, então ela não pode abranger múltiplas zonas ou regiões.
* Uma sub-rede pode abranger a totalidade da zona em um IBM Cloud VPC. 
* Deve-se criar sua VPC antes de criar suas sub-redes dentro dessa VPC.
* O suporte ao IPv6 não está disponível.
* É possível associar ou desassociar uma VSI a uma sub-rede. (Isso requer que você inclua uma vNIC e selecione uma largura da banda.)

É possível trazer sua própria variação de endereços IPv4 públicos (BYOIP) para a sua conta do IBM Cloud VPC. Ao usar o BYOIP, o IBM Cloud deve configurar esses endereços IPv4 nos recursos do IBM Cloud, que enviarão pacotes para e dos endereços fornecidos. Portanto, como resultado do uso de seu intervalo IPv4 fornecido no IBM Cloud, esses endereços IP podem ser expostos para a equipe de suporte da IBM e terceiros como parte de seu uso desse serviço.
{:important}

**Figura: uma representação gráfica de uma VPC mostrando zonas, regiões e sub-redes.**

![IBM Cloud VPC](images/vpc-experience.png)

## Usando prefixos de endereço para sub-redes

Cada sub-rede deve existir dentro de um prefixo de endereço.
 * Os prefixos de endereço de sub-rede são pré-alocados para cada zona.
 * Para a sua nova sub-rede, é possível selecionar seu intervalo de endereços IP por meio dos prefixos de endereço existentes.
 * Se os prefixos de endereço da zona não forem adequados, um dos prefixos de endereço existentes poderá ser editado ou um novo prefixo de endereço poderá ser incluído para a zona (usando a API, a CLI ou a IU).
 
### Prefixos de endereço padrão da VPC
 
Quando um novo VPC é criado, os prefixos de endereço padrão são designados da seguinte forma, com base na região e na zona:
 
* 'us-south-1' = '10.240.0.0/18'
* 'us-south-2' = '10.240.64.0/18'

Outras zonas ou regiões podem designar prefixos padrão diferentes, incluindo estes:
 
 * 10.240.0.0/18
 * 10.240.64.0/18
 * 10.240.128.0/18
 
### Prefixos de endereço e a IU do console do IBM Cloud
 
Quando você cria uma VPC usando a IU do console do IBM Cloud, o sistema seleciona seu prefixo de endereço automaticamente e requer que você crie uma sub-rede nesse prefixo padrão. Se esse esquema de endereço não se adequar aos seus requisitos, será possível customizar os prefixos de endereço depois de criar a VPC. Então, será possível criar sub-redes em seus prefixos de endereço customizados e excluir a sub-rede criada com o prefixo padrão.
 
Essa solução alternativa é necessária para usar BYOIP por meio da UI do console do IBM Cloud.
{:note}

### Endereços IP disponíveis

Estes são endereços IP disponíveis, conforme definido em **RFC 1918**:

 * 10.0.0.0 a 10.255.255.255
 * 172.16.0.0 a 172.31.255.255
 * 192.168.0.0 a 192.168.255.255

### Os endereços a seguir são endereços reservados em uma sub-rede

Nessa lista, os endereços IP assumem que o intervalo de CIDR da sub-rede é 10.10.10.0/24:

  * Primeiro endereço no intervalo CIDR (10.10.10.0): endereço de rede
  * Segundo endereço no intervalo CIDR (10.10.10.1): endereço do gateway
  * Terceiro endereço no intervalo CIDR (10.10.10.2): reservado pela IBM
  * Quarto endereço no intervalo de CIDR (10.10.10.3): reservado pela IBM para uso futuro
  * Último endereço no intervalo CIDR (10.10.10.255): endereço de transmissão de rede

### Mais sobre a criação de uma sub-rede

Há duas maneiras de especificar uma sub-rede para sua VPC:
  * É possível criar uma sub-rede fornecendo o tamanho da sub-rede que você precisa, como o número de endereços suportados (por exemplo, 1024)
  * É possível criar uma sub-rede fornecendo um intervalo de CIDR (como 10.0.0.1/29)
  
Como um exemplo de como especificar um bloco de 1024 usando CIDR, se você estiver especificando um intervalo de CIDR em vez de um tamanho de sub-rede, o bloco IPv4 `192.168.100.0/22` representará os 1024 endereços IPv4 de `192.168.100.0` a `192.168.103.255`.
{:tip}

### Mais sobre a exclusão de uma sub-rede
  * Não será possível excluir uma sub-rede se os recursos (como uma Instância de Servidor Virtual (VSI), IP flutuante ou gateway público) estiverem em uso nessa sub-rede. Nesse caso, os recursos deverão ser excluídos primeiro.
  * Você NÃO pode redimensionar uma sub-rede existente. Por exemplo, 10.10.10.0/24 não pode ser redimensionada para 10.5.1.3/20
