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

# Iniciación a la gestión de redes en la nube privada virtual


IBM aceptará la participación de un número limitado de clientes en un programa de acceso temprano a VPC a partir de principios de abril de 2019 y en los meses siguientes se ampliará su uso. Si su organización desea obtener acceso a IBM Virtual Private Cloud, complete este [formulario de nominación](https://cloud.ibm.com/vpc){: new_window} y un representante de IBM se pondrá en contacto con usted para indicarle los siguientes pasos a seguir.
{: important}

Para comenzar a trabajar con {{site.data.keyword.cloud}} Networking for Virtual Private Cloud

1. Cree una nube privada virtual.
2. Cree una o varias subredes en la nube privada virtual en una o varias zonas.
3. Cree una pasarela pública (PGW) en una subred si desea que los recursos de la subred tengan acceso a Internet o viceversa.

## Requisitos previos

 * **Permisos de usuario**: Asegúrese de que el usuario tiene permisos suficientes para crear y gestionar recursos en la VPC. Para obtener una lista de los permisos necesarios, consulte [Cómo otorgar los permisos necesarios para usuarios de VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).

## Utilice la interfaz de usuario, la CLI o la API REST para suministrar recursos de red de VPC

El suministro y la gestión de los recursos de red de VPC se pueden realizar a través de la interfaz de usuario, la CLI o la API REST.

* Para acceder a través de la interfaz de usuario, inicie una sesión en la [consola de IBM Cloud ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")]( https://{DomainName}/vpc){: new_window}. Siga la [guía de la interfaz de usuario](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console) si necesita ayuda.
* Para utilizar la interfaz de línea de mandatos, utilice el plugin [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) de la [CLI de IBM Cloud](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) y siga el ejemplo [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).
* Los usuarios más avanzados pueden llamar directamente a las [API REST](https://{DomainName}/apidocs/rias). Siga la guía de aprendizaje de [código de ejemplo](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) para comenzar a utilizar las API REST.

## Siguientes pasos

Después de suministrar la red VPC (Virtual Private Cloud), siga explorando.

* [Creación y gestión de instancias de servidor virtual](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [Utilización de grupos de seguridad](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Utilización de ACL de red](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(Beta) Utilización de VPN](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(Beta) Utilización de equilibradores de carga](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
