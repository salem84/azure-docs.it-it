---
title: Analisi di utenti, sessioni ed eventi in Azure Application Insights | Microsoft Docs
description: Analisi demografica degli utenti dell'app Web.
services: application-insights
documentationcenter: ''
author: NumberByColors
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 01/24/2018
ms.reviewer: mbullwin
ms.pm_owner: daviste;NumberByColors
ms.author: daviste
ms.openlocfilehash: 7d378c2f72035c3584e1f5cd3c1f0fb9a5d5c2ed
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60372287"
---
# <a name="users-sessions-and-events-analysis-in-application-insights"></a>Analisi di utenti, sessioni ed eventi in Application Insights

Scoprire quando le persone usano l'app Web, a quali pagine sono più interessati, dove si trovano, quali browser e sistemi operativi usano. Analizzare i dati di telemetria sull'utilizzo e sull'azienda con [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md).

![Screenshot della pagina Utenti di Application Insights](./media/usage-segmentation/0001-users.png)

## <a name="get-started"></a>Attività iniziali

Se nei pannelli degli utenti, delle sessioni e degli eventi nel portale di Application Insights non vengono ancora visualizzati i dati, leggere le [informazioni su come iniziare a usare gli strumenti d'uso](usage-overview.md).

## <a name="the-users-sessions-and-events-segmentation-tool"></a>Strumenti di segmentazione di Utenti, Sessioni ed Eventi

Tre pannelli d'uso usano lo stesso strumento per effettuare un'analisi approfondita dei dati di telemetria dell'app Web da tre diverse prospettive. Applicando i filtri e dividendo i dati, si possono scoprire informazioni dettagliate sull'uso di pagine e funzionalità diverse.

* **Strumento Utenti**: numero di persone che hanno usato l'app e le relative funzionalità.  Gli utenti vengono conteggiati tramite ID anonimi memorizzati nei cookie del browser. Una singola persona che usa browser o computer diversi verrà conteggiata più di una volta.
* **Strumento Sessioni**: numero di sessioni delle attività dell'utente che include determinate pagine e funzionalità dell'app. Una sessione viene conteggiata dopo mezz'ora di inattività dell'utente o in seguito a 24 ore di utilizzo continuo.
* **Strumento Eventi**: frequenza con cui vengono usate alcune pagine e funzionalità dell'app. La visualizzazione di una pagina viene conteggiata quando un browser carica la pagina dell'app, purché sia stata [instrumentata](../../azure-monitor/app/javascript.md). 

    Un evento personalizzato indica che nell'app si verifica un'operazione, spesso si tratta di un'interazione dell'utente ad esempio il clic su un pulsante o il completamento di alcune attività. Inserire codice nell'app per [generare eventi personalizzati](../../azure-monitor/app/api-custom-events-metrics.md#trackevent).

## <a name="querying-for-certain-users"></a>Esecuzione di query per determinati utenti

Modificando le opzioni di query nella parte superiore dello strumento Utenti, è possibile esaminare diversi gruppi di utenti:

* Mostra: scegliere una coorte di utenti da analizzare.
* Che ha usato: scegliere le visualizzazioni pagina e gli eventi personalizzati.
* Durante: scegliere un intervallo di tempo.
* Entro: scegliere la modalità di ordinazione dei dati in base a un intervallo di tempo o a un'altra proprietà, ad esempio la città o il browser.
* Dividi per: scegliere una proprietà in base alla quale dividere o segmentare i dati. 
* Aggiungi filtri: limitare le query a determinati utenti, sessioni o eventi in base alle relative proprietà, ad esempio città o browser. 
 
## <a name="saving-and-sharing-reports"></a>Salvataggio e condivisione di report 
È possibile salvare i report Utenti, mantenendoli privati nella sezione My Reports (Report personali) o condividendoli con tutti gli utenti con accesso a questa risorsa di Application Insights nella sezione Report condivisi.

Per condividere un collegamento a un report Utenti, Sessioni o Eventi, fare clic su **Condividi** nella barra degli strumenti e quindi copiare il collegamento.

Per condividere una copia dei dati in un report Utenti, Sessioni o Eventi, fare clic su **Condividi** nella barra degli strumenti e quindi fare clic sull'icona **Word** per creare un documento di Word con i dati. In alternativa, fare clic sull'icona **Word** sopra il grafico principale.

## <a name="meet-your-users"></a>Meet your users (Conosci gli utenti)

La sezione **Meet your users** (Conosci gli utenti) mostra informazioni relative a cinque utenti campione corrispondenti alla query corrente. La valutazione e l'analisi dei comportamenti delle persone, oltre a creare aggregazioni, può offrire informazioni dettagliate sull'uso effettivo dell'app da parte degli utenti.

## <a name="next-steps"></a>Passaggi successivi

- Per abilitare le esperienze di utilizzo, iniziare a inviare [eventi personalizzati](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) o [visualizzazioni pagina](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views).
- Se si inviano già eventi personalizzati o visualizzazioni pagina, è possibile esplorare gli strumenti relativi all'uso per scoprire come gli utenti usano il servizio.
    - [Grafici a imbuto](usage-funnels.md)
    - [Conservazione](usage-retention.md)
    - [Flussi degli utenti](usage-flows.md)
    - [Cartelle di lavoro](../../azure-monitor/app/usage-workbooks.md)
    - [Aggiungere il contesto utente](usage-send-user-context.md)