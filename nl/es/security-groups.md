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
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Utilización de grupos de seguridad
{: #using-security-groups}

Los grupos de seguridad le proporcionan una forma cómoda de aplicar reglas que establecen el filtrado en cada interfaz de red de una instancia de servidor virtual (VSI) basada en la dirección IP. Cuando cree un nuevo recurso de grupo de seguridad, lo actualizará para crear los patrones de tráfico de red que desee.

Los grupos de seguridad deniegan todo el tráfico por defecto. Puesto que las reglas se añaden a un grupo de seguridad, este define el tráfico que permite el grupo de seguridad.

Las reglas son _con estado_, lo que significa que el tráfico inverso en respuesta al tráfico permitido se autoriza automáticamente. Por lo que, por ejemplo, una regla que permite tráfico TCP de entrada en el puerto 80 también permite responder al tráfico TPC de salida en el puerto 80 al host de origen, sin la necesidad de usar reglas adicionales.

Los grupos de seguridad se circunscriben a un solo VPC. Esto significa que el grupo de seguridad se puede conectar _únicamente_ a interfaces de red de VSI dentro del mismo VPC.

Cuando se crea un VSI sin especificar ningún grupo de seguridad, la interfaz de red primaria de VSI se coloca en el grupo de seguridad _por defecto_ de la VPC de dicha VSI. Este release de VPC de {{site.data.keyword.cloud}} tiene un grupo de seguridad predeterminado definido que permite cierto tráfico. Para obtener más información, consulte [Actualización del grupo de seguridad predeterminado](/docs/infrastructure/vpc-network?topic=vpc-network-updating-the-default-security-group).

Puede configurar grupos de seguridad mediante la API REST, la CLI o la interfaz de usuario:

* [Configuración de grupos de seguridad mediante la API](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-apis)
* [Configuración de grupos de seguridad mediante la CLI](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Configuración de grupos de seguridad mediante la interfaz de usuario](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
