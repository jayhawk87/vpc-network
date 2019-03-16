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

# Utilizzo dei gruppi di sicurezza
{: #using-security-groups}

I gruppi di sicurezza ti forniscono un modo pratico per applicare le regole che stabiliscono il filtro di ogni interfaccia di rete di un'istanza del server virtuale (VSI), in base all'indirizzo IP. Quando crei una nuova risorsa del gruppo di sicurezza, la aggiornerai per creare i modelli di traffico di rete che desideri.

Per impostazione predefinita, un gruppo di sicurezza nega tutto il traffico. Come vengono aggiunte delle regole a un gruppo di sicurezza, viene definito il traffico consentito dal gruppo di sicurezza.

Le regole sono _con stato_, il che significa che viene automaticamente consentito il traffico inverso in risposta al traffico consentito. Quindi, ad esempio, una regola che consente il traffico TCP in entrata sulla porta 80 consente anche il traffico TCP in uscita in risposta sulla porta 80 all'host di origine, senza la necessità di un'ulteriore regola.

I gruppi di sicurezza sono dedicati a un solo VPC. Questo ambito implica che il gruppo di sicurezza può essere collegato _solo_ alle interfacce di rete delle VSI all'interno dello stesso VPC.

Quando una VSI viene creata senza che venga specificato alcun gruppo di sicurezza, l'interfaccia di rete primaria della VSI viene inserita nel gruppo di sicurezza _predefinito_ di tale VPC della VSI. Questa release di {{site.data.keyword.cloud}} VPC ha definito un gruppo di sicurezza predefinito che consente del traffico determinato. Per ulteriori informazioni, consulta [Aggiornamento del gruppo di sicurezza predefinito](/docs/infrastructure/vpc-network?topic=vpc-network-updating-the-default-security-group).

Puoi configurare i gruppi di sicurezza utilizzando l'API REST, la CLI o l'IU:

* [Configurazione dei gruppi di sicurezza utilizzando l'API](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-apis)
* [Configurazione dei gruppi di sicurezza utilizzando la CLI](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Configurazione dei gruppi di sicurezza utilizzando l'IU](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
