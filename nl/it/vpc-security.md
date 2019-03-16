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
{:download: .download}

# Sicurezza nel tuo IBM Cloud VPC
{: #security-in-your-ibm-cloud-vpc}

Puoi mantenere sicuri i tuoi carichi di lavoro e il tuo VPC controllando il traffico di rete utilizzando i gruppi di sicurezza (SG), gli elenchi di controllo degli accessi (ACL) o entrambi. I gruppi di sicurezza controllano il traffico su una base per istanza (VSI) e gli elenchi di controllo degli accessi su una base per sottorete.

## Panoramica della sicurezza

* Il traffico a/da una sottorete può essere controllato dagli elenchi di controllo degli accessi (ACL).
* I gruppi di sicurezza (SG) possono controllare il traffico al livello della VSI
* Configura un gateway pubblico per l'accesso della sottorete a internet, protetto dagli ACL
* Configura un IP mobile per l'accesso della VSI a internet, protetto dagli SG


**Figura: i gruppi di sicurezza e gli ACL aggiungono sicurezza alle tue sottoreti e istanze.**

![Sicurezza VPC](/images/vpc-connectivity-and-security.png)


## Definizioni

Questo [glossario](/docs/infrastructure/vpc?topic=vpc-vpc-glossary) fornisce le definizioni e le descrizioni degli ACL e degli SG, e le azioni che puoi effettuare con essi. La seguente sezione descrive le funzionalità di base degli ACL e dei gruppi di sicurezza e come VPC supporta la codifica end-to-end.

### ACL (Access Control List)
Un **ACL (Access Control List)** può gestire (ossia, può consentire o negare) il traffico in entrata e in uscita per una sottorete. Un ACL è senza stato, il che significa che devono essere specificate delle regole in entrata e in uscita separatamente e in modo esplicito. Ogni ACL è composto di regole, basate su un *IP di origine*, una *porta di origine*, un *IP destinazione*, una *porta di destinazione* e un *protocollo*.

In {{site.data.keyword.cloud}} VPC, ogni sottorete viene creata con un ACL predefinito, che consente il traffico in entrata e in uscita, ma i clienti possono creare degli ACL personalizzati. È sempre collegato solo un ACL a una sottorete, ma un ACL può essere collegato a più sottoreti. Per ulteriori informazioni su come utilizzare gli ACL, consulta la nostra [Guida ACL](/docs/infrastructure/vpc-network?topic=vpc-network-setting-up-network-acls-using-the-cli).

### Gruppo di sicurezza
Un **gruppo di sicurezza** si comporta come un firewall virtuale che controlla il traffico per uno o più server (VSI). Un gruppo di sicurezza è una raccolta di regole che specificano se consentire il traffico per una VSI associata.

Quando un cliente crea una VSI, può associare uno o più gruppi di sicurezza a tale VSI. Fornendo le autorizzazioni corrette, i clienti possono modificare le regole del gruppo di sicurezza utilizzando l'API, la CLI o la console IBM.

Per ulteriori informazioni su come creare una VSI che utilizza i gruppi di sicurezza e altre informazioni sulle funzionalità dei gruppi di sicurezza fai riferimento alla nostra [guida per i gruppi di sicurezza](/docs/infrastructure/vpc-network?topic=vpc-network-using-security-groups).

### Codifica end-to-end

Sebbene IBM Cloud VPC non fornisce la codifica end-to-end, la consente. Ad esempio, se hai un endpoint sicuro su un server virtuale (ad esempio, un server HTTPS sulla porta 443), puoi collegare un IP mobile a tale server, dopodiché la tua connessione è codificata end-to-end dal client al server sulla porta 443.  Nulla nel percorso forza una descrizione.

Tieni presente tuttavia, che se utilizzi un protocollo non sicuro come ad esempio HTTP sulla porta 80, i tuoi dati end-to-end sono in chiaro.
