---
title: Rilevamento intelligente - Anomalie degli errori in Application Insights | Documentazione Microsoft
description: Avvisa sui cambiamenti insoliti nella frequenza delle richieste non riuscite all'app Web e offre un'analisi diagnostica. Non è necessaria alcuna configurazione.
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 12/18/2018
ms.reviewer: yossiy
ms.author: mbullwin
ms.openlocfilehash: 46944603fdf45a2a7a14641086959bf61b3f773e
ms.sourcegitcommit: c63e5031aed4992d5adf45639addcef07c166224
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/28/2019
ms.locfileid: "67465875"
---
# <a name="smart-detection---failure-anomalies"></a>Rilevamento intelligente - Anomalie degli errori
[Application Insights](../../azure-monitor/app/app-insights-overview.md) avvisa automaticamente l'utente quasi in tempo reale se si verifica un incremento anomalo della frequenza di richieste non riuscite nell'app Web, rilevando un aumento insolito della percentuale di richieste HTTP o di chiamate alle dipendenze segnalate come non riuscite. Per quanto riguarda le richieste, quelle non riuscite hanno in genere un codice di risposta 400 o superiore. Per poter valutare e diagnosticare il problema, la notifica include un'analisi delle caratteristiche degli errori e i dati di telemetria correlati. Sono disponibili anche collegamenti al portale di Application Insights per un'ulteriore diagnosi. La funzionalità non necessita di alcuna installazione o configurazione, perché usa algoritmi di Machine Learning per stimare la normale frequenza degli errori.

Questa funzionalità funziona per qualsiasi app web, ospitata nel cloud o nei tuoi server, che genera dati di telemetria richieste o delle dipendenze: ad esempio, se si dispone di un ruolo di lavoro che chiama [trackrequest ()](../../azure-monitor/app/api-custom-events-metrics.md#trackrequest) o [trackdependency ()](../../azure-monitor/app/api-custom-events-metrics.md#trackdependency).

Al termine della configurazione di [Application Insights per il progetto](../../azure-monitor/app/app-insights-overview.md), non appena l'app genera una quantità minima di dati di telemetria la funzionalità di rilevamento intelligente delle anomalie degli errori impiega 24 ore per apprendere il comportamento normale dell'app prima di essere attivato e inviare avvisi.

Ecco un avviso di esempio.

![Esempio di avviso di rilevamento intelligente che mostra l'analisi del cluster riguardo all'errore](./media/proactive-failure-diagnostics/013.png)

> [!NOTE]
> Per impostazione predefinita, si riceve un messaggio di posta elettronica più breve rispetto a quello dell’esempio. Tuttavia, è possibile [passare a questo formato dettagliato](#configure-alerts).
>
>

Le informazioni fornite includono:

* La frequenza degli errori confrontata al comportamento normale dell’applicazione.
* Il numero di utenti interessati, per far comprendere la portata del problema.
* Un modello caratteristico associato agli errori. In questo esempio sono presenti un codice di risposta, un nome di richiesta (operazione) e una versione dell'applicazione specifici. che indica immediatamente in quale punto del codice cercare. In alternativa si può usare un browser o un sistema operativo client specifico.
* Eccezioni, tracce di log ed errori di dipendenza (database o altri componenti esterni) che sembrano associati ai particolari errori.
* Collegamenti diretti alle ricerche rilevanti nei dati di telemetria in Application Insights.

## <a name="failure-anomalies-v2"></a>Errore anomalie v2
È ora disponibile una nuova versione della regola di avviso anomalie degli errori. Questa nuova versione è in esecuzione nella nuova piattaforma avvisi Azure e introduce numerosi miglioramenti rispetto alla versione esistente.

### <a name="whats-new-in-this-version"></a>Novità in questa versione?
- Rilevamento più veloce dei problemi
- Viene creata un set più completo di azioni - la regola di avviso con un oggetto associato [gruppo di azioni](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups) denominato "Application Insights rilevamento intelligente" che contiene le azioni webhook e posta elettronica e possono essere estese per attivare azioni aggiuntive quando l'avviso viene attivato.
- Più incentrato notifiche - notifiche tramite posta elettronica inviate da questa regola di avviso vengono ora inviate per impostazione predefinita per gli utenti associati ai ruoli di monitoraggio di lettore e collaboratore al monitoraggio della sottoscrizione. Per ulteriori informazioni su questo [qui](https://docs.microsoft.com/azure/azure-monitor/app/proactive-email-notification).
- Configurazione semplificata tramite modelli ARM, vedere l'esempio [qui](https://docs.microsoft.com/azure/azure-monitor/app/proactive-arm-config).
- Seguono le notifiche inviate da questa regola di avviso di supporto dello schema di avviso comuni - i [schema comune degli avvisi](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-common-schema).
- Unified modello di posta elettronica - messaggio di posta elettronica delle notifiche da questa regola di avviso hanno un aspetto coerente & sentono con altri tipi di avviso. Con questa modifica, la possibilità di ricevere avvisi di anomalie degli errori con le informazioni di diagnostica dettagliate non è più disponibile.

### <a name="how-do-i-get-the-new-version"></a>Come ottenere la nuova versione?
- Vengono ora effettuato il provisioning di risorse di Application Insights appena create con la nuova versione della regola di avviso anomalie degli errori.
- Le risorse con la versione classica delle anomalie degli errori di avviso regola otterrà la nuova versione di una volta la sottoscrizione di hosting esistente di Application Insights viene eseguita la migrazione alla nuova piattaforma avvisi come parte di [processo di ritiro degli avvisi classici ](https://docs.microsoft.com/azure/azure-monitor/platform/monitoring-classic-retirement).

> [!NOTE]
> La nuova versione della regola di avviso anomalie degli errori rimane disponibile. Inoltre, le azioni webhook e posta elettronica attivata da associato "Rilevamento intelligente di Application Insights" gruppo di azione è gratuita.
> 
> 

## <a name="benefits-of-smart-detection"></a>Vantaggi del rilevamento intelligente
I normali [avvisi relativi alla metrica](../../azure-monitor/app/alerts.md) indicano che potrebbe essersi verificato un problema. Tuttavia, la funzionalità di rilevamento intelligente avvia automaticamente i processi di diagnostica ed esegue diverse analisi che altrimenti l'utente dovrebbe eseguire manualmente. Si ottengono risultati ben strutturati che consentono individuare rapidamente le cause del problema.

## <a name="how-it-works"></a>Funzionamento
Il rilevamento intelligente monitora i dati di telemetria ricevuti dall'app e, in particolare, le frequenze degli errori. Questa regola conta il numero di richieste per cui la proprietà `Successful request` è false e il numero di chiamate alle dipendenze per cui la proprietà `Successful call` è false. Per le richieste, per impostazione predefinita, `Successful request == (resultCode < 400)` a meno che sia stato scritto codice personalizzato per [filtrare](../../azure-monitor/app/api-filtering-sampling.md#filtering) o generare particolari chiamate [TrackRequest](../../azure-monitor/app/api-custom-events-metrics.md#trackrequest). 

Le prestazioni dell'applicazione hanno un modello di comportamento tipico. Alcune richieste o chiamate alle dipendenze saranno più soggette a errori di altre e la frequenza degli errori generale potrebbe aumentare proporzionalmente al carico. Il rilevamento intelligente si avvale del servizio Machine Learning per trovare queste anomalie.

Man mano che i dati di telemetria vengono inviati dall'app Web ad Application Insights, la funzionalità di rilevamento intelligente confronta il comportamento corrente con i modelli rilevati negli ultimi giorni. Se viene osservato un incremento anomalo della frequenza degli errori rispetto alle prestazioni precedenti, viene attivata un’analisi.

Quando l’analisi viene attivata, il servizio esegue un'analisi cluster della richiesta non riuscita per provare a identificare un modello dei valori che caratterizzano gli errori. Nell'esempio precedente l'analisi ha rivelato che la maggior parte degli errori riguardano un codice risultato, un nome richiesta, un URL del server host e un’istanza del ruolo specifici. L’analisi ha anche determinato che la proprietà sistema operativo client è distribuita su più valori e perciò non è presente nell’elenco.

Quando il servizio viene instrumentato con queste chiamate ai dati di telemetria, l'analizzatore cerca un'eccezione e un errore di dipendenza associati alle richieste identificate nel cluster, con un esempio di qualsiasi log di traccia associato a tali richieste.

I risultati dell'analisi vengono inviati all'utente come avviso, a meno che la configurazione attiva preveda di non eseguire questa operazione.

Analogamente agli [avvisi impostati manualmente](../../azure-monitor/app/alerts.md), è possibile esaminare lo stato dell'avviso e configurarlo nel pannello Avvisi della risorsa di Application Insights. A differenza di altri avvisi, non è però necessario impostare o configurare il rilevamento intelligente. Se necessario, è possibile disabilitarlo o modificare gli indirizzi di posta elettronica di destinazione.

### <a name="alert-logic-details"></a>Dettagli della logica degli avvisi

Gli avvisi vengono attivati da un algoritmo di apprendimento automatico proprietario che impedisce di condividere i dettagli di implementazione esatti. D'altra parte, è comprensibile che talvolta si rende necessario conoscere altri dettagli su come funziona la logica sottostante. Per determinare se deve essere attivato un avviso, vengono valutati i fattori principali seguenti: 

* Analisi della percentuale di errore delle richieste o delle dipendenze in una finestra temporale scorrevole di 20 minuti.
* Confronto tra la percentuale di errore degli ultimi 20 minuti rispetto alla percentuale registrata negli ultimi 40 minuti e negli ultimi sette giorni e individuazione delle deviazioni significative che superano di X volte la deviazione standard.
* Uso di un limite adattivo per la percentuale di errore minima, che varia in base al volume di richieste/dipendenze dell'app.

## <a name="configure-alerts"></a>Configurare gli avvisi
È possibile disabilitare il rilevamento intelligente, modificare i destinatari di posta elettronica, creare un webhook o scegliere messaggi di avviso più dettagliati.

Aprire la pagina degli avvisi. Le anomalie degli errori sono disponibili con gli eventuali avvisi impostati manualmente e si può vedere se attualmente sono in uno stato di avviso.

![Nella pagina Panoramica fare clic sul riquadro Avvisi. In alternativa, in qualsiasi pagina di Metrica fare clic su pulsante Avvisi.](./media/proactive-failure-diagnostics/021.png)

Fare clic sull'avviso per configurarlo.

![Configurazione](./media/proactive-failure-diagnostics/032.png)

Si noti che è possibile disabilitare l'avviso di rilevamento intelligente, ma non eliminarlo (o crearne un altro).

#### <a name="detailed-alerts"></a>Avvisi dettagliati
Se si seleziona "È possibile ottenere informazioni di diagnostica più dettagliate", il messaggio di posta elettronica conterrà un maggior numero di informazioni di diagnostica. A volte è possibile individuare il problema basandosi solo sui dati contenuti nel messaggio di posta elettronica.

Esiste un rischio limitato che un avviso più dettagliato possa contenere informazioni riservate poiché include i messaggi di eccezione e di traccia. Ciò si verifica tuttavia soltanto se il codice permette l'inserimento di informazioni riservate in tali messaggi.

## <a name="triaging-and-diagnosing-an-alert"></a>Valutazione e diagnosi di un avviso
Un avviso indica che è stato rilevato un incremento anomalo nella frequenza delle richieste non riuscite. È probabile che si sia verificato un problema con l'app o il relativo ambiente.

Dalla percentuale delle richieste e dal numero di utenti interessati è possibile decidere il livello di priorità del problema. Nell'esempio precedente si ha una percentuale di errori del 22,5% rispetto alla percentuale normale dell'1% e ciò evidenzia sicuramente un'anomalia. D'altra parte, solo 11 utenti sono stati interessati. Se si fosse trattato di un'app personale, sarebbe stato più semplice valutare la gravità del problema.

In molti casi è possibile diagnosticare rapidamente il problema in base al nome della richiesta, alle eccezioni, agli errori di dipendenza e ad altri dati di traccia forniti.

Esistono alcune altre indicazioni. Ad esempio, la percentuale di errori di dipendenza in questo esempio è uguale alla percentuale di eccezioni (89,3%). Ne consegue che l'eccezione è collegata direttamente all'errore di dipendenza e questo indica chiaramente il punto di partenza per la ricerca dell'errore nel codice.

Per approfondire l'analisi, i collegamenti in ogni sezione consentono di passare direttamente a una [pagina di ricerca](../../azure-monitor/app/diagnostic-search.md) filtrata in base alle richieste, all'eccezione, alla dipendenza o alla traccia pertinenti. In alternativa è possibile aprire il [portale di Azure](https://portal.azure.com), passare alla risorsa di Application Insights per l'app e aprire il pannello Errori.

In questo esempio, facendo clic sul collegamento "View dependency failures details" (Visualizza i dettagli degli errori di dipendenza), si apre il pannello di ricerca di Application Insights, Mostra l'istruzione SQL che include un esempio della causa radice: Nei campi obbligatori sono stati inseriti valori null che non hanno superato la convalida durante l'operazione di salvataggio.

![Ricerca diagnostica](./media/proactive-failure-diagnostics/051.png)

## <a name="review-recent-alerts"></a>Esaminare gli avvisi recenti

Fare clic su **Rilevamento intelligente** per accedere all'avviso più recente:

![Riepilogo degli avvisi](./media/proactive-failure-diagnostics/070.png)


## <a name="whats-the-difference-"></a>Qual è la differenza
Il rilevamento intelligente delle anomalie degli errori integra altre funzionalità simili ma distinte di Application Insights.

* Gli [avvisi metrica](../../azure-monitor/app/alerts.md) vengono impostati dall'utente e possono monitorare un'ampia gamma di metriche, ad esempio utilizzo della CPU, frequenza delle richieste, tempi di caricamento delle pagine e così via. È possibile usarli per avvisare l'utente, ad esempio, se è necessario aggiungere altre risorse. Al contrario, il rilevamento intelligente delle anomalie degli errori copre una gamma ristretta di metriche critiche (attualmente solo la percentuale di richieste non riuscite), progettate per inviare notifiche quasi in tempo reale dopo l'aumento significativo della percentuale di richieste non riuscite dell'app Web rispetto al comportamento normale della stessa app Web.

    Il rilevamento intelligente adatta automaticamente la soglia in risposta alle condizioni attuali.

    Il rilevamento intelligente avvia automaticamente le attività di diagnostica.
* Il [rilevamento intelligente delle anomalie delle prestazioni](proactive-performance-diagnostics.md) usa l'intelligenza artificiale per individuare modelli insoliti nelle metriche, senza richiedere alcuna configurazione. A differenza del rilevamento intelligente delle anomalie degli errori, tuttavia, lo scopo del rilevamento intelligente delle anomalie delle prestazioni è trovare i segmenti dei molteplici modi di utilizzo che potrebbero essere gestiti in modo non corretto, ad esempio da pagine specifiche in un tipo specifico di browser. L'analisi viene eseguita ogni giorno e se viene trovato un risultato, è probabilmente molto meno urgente di un avviso. Al contrario, l'analisi per le anomalie degli errori viene eseguita continuamente sui dati di telemetria in ingresso e l'utente riceverà una notifica entro pochi minuti se la percentuale di errori del server è superiore a quella prevista.

## <a name="if-you-receive-a-smart-detection-alert"></a>Se si riceve un avviso di rilevamento intelligente
*Perché ho ricevuto questo avviso?*

* È stato rilevato un incremento anomalo delle richieste non riuscite rispetto al valore normale del periodo precedente. Dopo aver analizzato errori e dati di telemetria associati, si consiglia di esaminare più approfonditamente l'eventuale problema.

*La notifica indica la sicura presenza di un problema?*

* Gli avvisi vengono emessi in caso di interruzione dell'app o di riduzione delle prestazioni, ma solo l'utente può comprendere completamente la semantica e l'impatto del problema sull'app o gli utenti.

*Il servizio implica l'accesso manuale ai dati da parte di Microsoft?*

* No. Il servizio è completamente automatico. L'utente riceve le notifiche, ma i dati restano [privati](../../azure-monitor/app/data-retention-privacy.md).

*Va eseguita una sottoscrizione a questo avviso?*

* No. In ogni applicazione che invia dati di telemetria della richiesta è presente la regola di avviso per il rilevamento intelligente.

*È possibile annullare la sottoscrizione oppure ottenere le notifiche inviate ai colleghi?*

* Sì. In Regole di avviso fare clic sulla regola per il rilevamento intelligente per configurarla. È possibile disabilitare l'avviso o modificare i destinatari dell'avviso.

*Non trovo più il messaggio di posta elettronica. Dove trovo le notifiche nel portale?*

* In Log attività. In Azure aprire la risorsa di Application Insights per l'app e quindi selezionare Log attività.

*Alcuni avvisi segnalano problemi noti che è inutile ricevere.*

* Nel backlog è disponibile una funzionalità per eliminare gli avvisi.

## <a name="next-steps"></a>Passaggi successivi
Gli strumenti di diagnostica seguenti consentono di controllare la telemetria dall'app:

* [Esplora metriche](../../azure-monitor/app/metrics-explorer.md)
* [Esplora ricerche](../../azure-monitor/app/diagnostic-search.md)
* [Linguaggio avanzato di query di Analisi](../../azure-monitor/log-query/get-started-portal.md)

Gli avvisi di rilevamento intelligente sono completamente automatici, tuttavia è possibile configurare avvisi aggiuntivi, se necessario.

* [Configurare manualmente gli avvisi relativi alle metriche](../../azure-monitor/app/alerts.md)
* [Test Web di disponibilità](../../azure-monitor/app/monitor-web-app-availability.md)
