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

# Introduzione alla rete per il VPC (Virtual Private Cloud)
{: #getting-started-with-networking-for-virtual-private-cloud}


IBM accetterà un numero limitato di clienti che parteciperanno a un programma di accesso anticipato al VPC a partire dai primi di aprile 2019, con l'utilizzo completo che verrà aperto nei mesi seguenti. Se la tua organizzazione desidera ottenere l'accesso all'IBM Virtual Private Cloud, completa questo [modulo per nomina](https://cloud.ibm.com/vpc){: new_window} e un rappresentante di IBM ti contatterà per spiegarti i passi successivi.
{: important}

Introduzione alla rete {{site.data.keyword.cloud}} per il VPC (Virtual Private Cloud)

1. Crea un VPC (Virtual Private Cloud).
2. Crea una o più sottoreti nel VPC (Virtual Private Cloud) in una o più zone.
3. Crea un gateway pubblico (PGW) su una sottorete se vuoi che le risorse nella tua sottorete abbiano accesso a internet o viceversa.

## Prerequisiti

 * **Autorizzazioni utente**: assicurati che il tuo utente disponga delle autorizzazioni necessarie per creare e gestire le risorse nel tuo VPC. Per un elenco di autorizzazioni necessarie, consulta [Concessione delle autorizzazioni necessarie agli utenti VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).

## Utilizza l'IU, la CLI o l'API REST per eseguire il provisioning delle risorse di rete VPC

Il provisioning e la gestione delle risorse di rete VPC possono essere eseguiti tramite l'IU, la CLI o l'API REST.

* Per l'accesso tramite l'interfaccia utente, accedi alla [Console IBM Cloud ![Icona link esterno](../../icons/launch-glyph.svg "Icona link esterno")]( https://{DomainName}/vpc){: new_window}. Segui la [Guida dell'interfaccia utente](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console) se hai bisogno di aiuto.
* Per utilizzare l'interfaccia riga di comando, utilizza il plugin [infrastructure-service](/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) della [CLI IBM Cloud](/docs/cli/reference/ibmcloud?topic=cloud-cli-overview) e segui l'esempio [Hello World](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli).
* Per gli utenti più esperti, puoi richiamare direttamente le [API REST](https://{DomainName}/apidocs/rias). Segui l'esercitazione [codice di esempio](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) per iniziare ad utilizzare le API REST.

## Passi successivi

Dopo che è stato eseguito il provisioning della tua rete VPC (Virtual Private Cloud), esplora altri argomenti.

* [Creazione e gestione delle istanze del server virtuale](/docs/infrastructure/vpc?topic=vpc-creating-and-managing-virtual-server-instances)
* [Utilizzo dei gruppi di sicurezza](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-security-groups-using-the-cli)
* [Utilizzo degli ACL di rete](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli)
* [(Beta) Utilizzo della VPN](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-vpn-with-your-vpc)
* [(Beta) Utilizzo dei programmi di bilanciamento del carico](/docs/infrastructure/vpc-network?topic=vpc-network--beta-using-load-balancers-in-ibm-cloud-vpc#-beta-using-load-balancers-in-ibm-cloud-vpc)
