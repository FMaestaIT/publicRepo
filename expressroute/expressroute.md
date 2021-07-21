# IMPLEMENTAZIONE DI EXPRESSROUTE: TRA IL ROUTING E LA RESILIENZA

![Federico_resize_autostrada](https://github.com/FMaestaIT/publicRepo/blob/e135aff374c71b2c4c8cbc797167ca12649a9bc2/expressroute/IMG/Federico_resize_autostrada-compressor.jpg?raw=true)

### ExpressRoute-Azure
La **migrazione dei servizi di business in cloud** è un argomento attuale ed estremamente gettonato. Una delle domande che mi sento rivolgere più spesso riguarda la scelta di una **modalità di collegamento all’infrastruttura cloud** che garantisca un accesso stabile, sicuro e ad alte prestazioni.

E il mio consiglio è **ExpressRoute!**

In questo articolo non parlerò di pro e contro di questa soluzione, né della procedura di attivazione: questi argomenti sono largamente approfonditi nella documentazione ufficiale Microsoft (di cui trovate i riferimenti in fondo all’articolo). La mia intenzione, invece, è quella di trattare due aspetti fondamentali con i quali, chiunque si trovi a dover implementare una connettività ExpressRoute, deve fare i conti: il **Routing** e il **Service Level Agreement (SLA)**.

Ora andiamo al succo e vediamo nel dettaglio di cosa stiamo parlando.

### COS'È EXPRESSROUTE?
Il circuito ExpressRoute è un collegamento privato ad alte prestazioni tra la propria infrastruttura ed uno specifico Datacenter di Azure.

Questa soluzione garantisce elevate performance di velocità e offre accessi privati a servizi IaaS collegati alle vNet, a tutti i servizi PaaS che godono del servizio di Private Endpoint e ad alcuni servizi SaaS (come Office 365).

![Federico_Resize_expressroute-connection-overviewedited](https://github.com/FMaestaIT/publicRepo/blob/main/expressroute/IMG/Federico_Resize_expressroute-connection-overviewedited-compressor.jpg?raw=true)


Quando si decide di adottare questa soluzione, la prima decisione da effettuare riguarda il modello di connettività da implementare. ExpressRoute, infatti, offre tre differenti modelli di connettività:

 - Point-to-Point Ethernet Connection
 - Cloud Exchange Co-location
 - Any-to-Any (IPVPN) Connection.

Lo scenario che prenderemo in analisi è **“Any-to-Any Connection“**, in quanto si integra direttamente con la propria connettività MPLS e la sua gestione risulta simile a quella già in uso presso l’IT aziendale.

![Federico_Resize_expressroute-connectivity-models-diagram](https://github.com/FMaestaIT/publicRepo/blob/main/expressroute/IMG/Federico_Resize_expressroute-connectivity-models-diagram-compressor.png?raw=true)


### EXPRESSROUTE E IL ROUTING BGP: UN'IMPLEMENTAZIONE NON DEL TUTTO INDOLORE
Il **Routing** è il protocollo fondamentale per la comunicazione tra componenti collegate tramite una rete. Esistono molte versioni di Protocollo di Routing, ognuna con i propri vantaggi.

Nel caso di ExpressRoute viene utilizzato il **protocollo BGP** (Border Gateway Protocol). Questo, di fatto, è un prerequisito fondamentale che obbliga il comparto aziendale responsabile del Networking a dover affrontare un adattamento strutturale della propria MPLS nel caso in cui siano utilizzati protocolli diversi dal BGP.

Ma i grattacapi del comparto IT non finiscono qui. Un altro aspetto fondamentale, che spesso viene tralasciato fino al giorno in cui viene messa in produzione la connettività ExpressRoute, è la propagazione della **“Default Route”**. Questo significa che, esclusi alcuni scenari particolari “By Design”, le sedi collegate tramite un circuito MPLS raggiungono la rete pubblica per mezzo di una sola tra le sedi fornite dalla MPLS e, generalmente, si tratta di quella dove è situato il principale CED aziendale.

La **“Default Route”** ha il semplice scopo di veicolare tutte le comunicazioni che hanno come destinatario un indirizzamento IP esterno alla propria rete, come tipicamente avviene per le risorse Internet.

Tutti i Datacenter Azure sono provvisti di una propria connettività verso Internet che viene normalmente utilizzata da tutte le risorse allocate nel Cloud, se non diversamente gestite tramite delle specifiche scelte di design. L’applicazione di una “Default Route” interviene sulla comunicazione delle risorse Cloud da e verso Internet. In questo scenario è possibile che, se esposte pubblicamente, non sia possibile raggiungere dall’esterno i servizi che queste erogano.

Per esempio, pensiamo a delle risorse IaaS che erogano i siti istituzionali dell’azienda, catapultandoci in uno scenario di “Routing Asincrono” dove la richiesta effettuata da un generico utente si perde dei meandri della rete, in quanto non percorre lo stesso percorso tra andata e ritorno. Questo tipo di inconveniente si risolve comunicando all’ISP di escludere l’applicazione della “Default Route” dal Datacenter Azure.

### SERVICE LEVEL AGREEMENT: COME RENDERE L'EXPRESSROUTE ESTREMAMENTE RESILIENTE
Il **“Service Level Agreement”**, o SLA, altro non è che la definizione di **canoni di qualità di servizio** che devono essere rispettati dal fornitore di servizi (in questo caso Microsoft) nei confronti dei propri clienti. Lo SLA si valorizza con un numero percentuale che indica la garanzia di disponibilità del servizio in un determinato periodo, per esempio in un mese o in un anno.

**ExpressRoute ha una disponibilità di 99,95% (tre nove)**, quindi il servizio di connettività non è garantito per un massimo di 4,38 ore nell’arco dell’anno. Questo non significa che ExpressRoute non sarà sicuramente disponibile per quasi cinque ore all’anno; ma implica che, se si verificasse un disservizio di durata inferiore a questa soglia, non solo il nostro business ne subirebbe dei danni, ma non avremmo diritto a nessun rimborso da parte di Microsoft.

A questo punto, potreste avere dei dubbi rispetto all’implementazione di ExpressRoute. Ma dovete sapere che, fortunatamente, è possibile intervenire sulla disponibililità **implementando una soluzione di design** che prevede la configurazione di una connessione Site-to-Site a sostegno dell’ExpressRoute.

![Federico_expressroute-vpns2s-compressor](https://github.com/FMaestaIT/publicRepo/blob/main/expressroute/IMG/Federico_expressroute-vpns2s-compressor.jpg?raw=true)

Per fare ciò, è strettamente necessario avere ben a mente alcuni punti fondamentali:

 - è possibile creare un **Virtual Network Gateway** di tipo VPN solo dopo aver creato e implementato il Gateway per l’ExpressRoute;
 - la **Gateway Subnet** deve avere almeno una classe di indirizzi /27 altrimenti non è possibile soddisfare la richiesta di indirizzi IP necessari per l’implementazione dei due Gateway;
 - la configurazione viene effettuata solo tramite **PowerShell**, non è possibile utilizzare il portale;
 - la gestione del traffico tra i due canali di comunicazione va gestita lato on-premises per mezzo di rotte con priorità differenti applicate sugli apparati che gestiscono il traffico locale; è possibile applicare queste rotte direttamente sulla rete MPLS soltanto se è l’ISP stesso ad implementare il canale VPN e ad attestarlo direttamente alla WAN aziendale.

In questo modo, oltre ad implementare una connettività di backup, è possibile veicolare la comunicazione tramite VPN a tutte quelle vNet non servite dall’ExpressRoute. Un esempio potrebbe essere la raggiungibilità della rete di management che per motivi di sicurezza viene isolata dalla rete di erogazione fornita dalla ExpressRoute.

Quindi, ricapitolando: per implementare una connettività ExpressRoute è fondamentale analizzare a fondo la propria network facendo attenzione a tutte quelle applicazioni “By Design” che ci vengono comode per la gestione delle sedi aziendali.

 

Link Utili:
| Argomento | Link |
| ------ | ------ |
| Introduction | https://docs.microsoft.com/it-it/azure/expressroute/expressroute-introduction|
| Connectivity Models | https://docs.microsoft.com/it-it/azure/expressroute/expressroute-connectivity-models |
|High-Availability | https://docs.microsoft.com/it-it/azure/expressroute/designing-for-high-availability-with-expressroute |
| Prerequisites| https://docs.microsoft.com/it-it/azure/expressroute/expressroute-prerequisites |
| Expressroute Coexist| https://docs.microsoft.com/it-it/azure/expressroute/expressroute-howto-coexist-resource-manager |