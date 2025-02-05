---
title: Guida di riferimento per i riquadri di Progettazione viste in Monitoraggio di Azure | Microsoft Docs
description: Utilizzando Progettazione vista in Monitoraggio di Azure, è possibile creare viste personalizzate che vengono visualizzate nel portale di Azure e contengono numerose visualizzazioni sui dati nell'area di lavoro di Log Analitica. Questo articolo è una guida di riferimento per le impostazioni relative ai riquadri disponibili nelle viste personalizzate.
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: 41787c8f-6c13-4520-b0d3-5d3d84fcf142
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 01/17/2018
ms.author: bwren
ms.openlocfilehash: 9c0283081bd7245b1b886ed82ba03130a7a3bf2c
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "61342038"
---
# <a name="reference-guide-to-view-designer-tiles-in-azure-monitor"></a>Guida di riferimento per i riquadri di Progettazione viste in Monitoraggio di Azure
Utilizzando Progettazione vista in Monitoraggio di Azure, è possibile creare un'ampia gamma di visualizzazioni personalizzate nel portale di Azure che consentono di visualizzare i dati nell'area di lavoro di Log Analitica. Questo articolo è una guida di riferimento per le impostazioni relative ai riquadri disponibili nelle viste personalizzate.

Per altre informazioni su Progettazione visualizzazioni, vedere:

* [Progettazione viste](view-designer.md): offre una panoramica di Progettazione viste e delle procedure per la creazione e la modifica di viste personalizzate.
* [Informazioni di riferimento sulle parti di visualizzazione](view-designer-parts.md): guida di riferimento per le impostazioni delle parti di visualizzazione disponibili nelle viste personalizzate.


I riquadri di Progettazione viste disponibili sono descritti nella tabella seguente:  

| Riquadro | Descrizione |
|:--- |:--- |
| [Number](#number-tile) |Numero di record restituiti da una query. |
| [Due numeri](#two-numbers-tile) |Numero di record restituiti da due diverse query. |
| [Anello](#donut-tile) | Grafico basato su una query con un valore di riepilogo al centro. |
| Grafico a linee e callout | Grafico a linee basato su una query e callout con un valore di riepilogo. |
| [Grafico a linee](#line-chart-tile) |Grafico a linee basato su una query. |
| [Due sequenze temporali](#two-timelines-tile) | Istogramma con due serie, ognuna basata su una query separata. |

Le sezioni successive descrivono nei dettagli i tipi di riquadro e le relative proprietà.

> [!NOTE]
> I riquadri nelle visualizzazioni sono basati sui [registrare query](../log-query/log-query-overview.md) nell'area di lavoro di Log Analitica. Attualmente non supportano [query resource tra](../log-query/cross-workspace-query.md) per recuperare dati da Application Insights.

## <a name="number-tile"></a>Riquadro Numero
Il riquadro **Numero** visualizza il numero di record restituiti da una query di log e un'etichetta.

![Riquadro Numero](media/view-designer-tiles/tile-number.png)

| Impostazione | DESCRIZIONE |
|:--- |:--- |
| Name |Testo che viene visualizzato nella parte superiore del riquadro. |
| Descrizione |Testo visualizzato sotto il nome del riquadro. |
| **Riquadro** | |
| Legenda |Testo visualizzato sotto il valore. |
| Query |Query eseguita. Viene visualizzato il numero di record restituiti dalla query. |
| **Funzionalità avanzate** |**&gt; Verifica del flusso di dati** |
| Enabled |Selezionare questo collegamento se è necessario abilitare la verifica del flusso di dati per il riquadro. Questo approccio fornisce un messaggio alternativo se non sono disponibili dati. Questo approccio viene in genere viene usato per fornire un messaggio durante il periodo temporaneo in cui la vista viene installata e i dati diventano disponibili. |
| Query |Query eseguita per determinare se sono disponibili dati per la vista. Se la query non restituisce risultati, viene visualizzato un messaggio al posto del valore dalla query principale. |
| `Message` |Messaggio visualizzato se la query di verifica del flusso di dati non restituisce dati. Se non si specifica alcun messaggio, viene visualizzato il messaggio di stato *Esecuzione della valutazione*. |


## <a name="two-numbers-tile"></a>Riquadro Due numeri
Il riquadro visualizza il numero di record di due diverse query di log e un'etichetta per ognuno.

![Riquadro Due numeri](media/view-designer-tiles/tile-two-numbers.png)

| Impostazione | DESCRIZIONE |
|:--- |:--- |
| Name |Testo che viene visualizzato nella parte superiore del riquadro. |
| Descrizione |Testo visualizzato sotto il nome del riquadro. |
| **Primo riquadro** | |
| Legenda |Testo visualizzato sotto il valore. |
| Query |Query eseguita. Viene visualizzato il numero di record restituiti dalla query. |
| **Secondo riquadro** | |
| Legenda |Testo visualizzato sotto il valore. |
| Query |Query eseguita. Viene visualizzato il numero di record restituiti dalla query. |
| **Funzionalità avanzate** |**&gt; Verifica del flusso di dati** |
| Enabled |Selezionare questo collegamento se è necessario abilitare la verifica del flusso di dati per il riquadro. Questo approccio fornisce un messaggio alternativo se non sono disponibili dati. Questo approccio viene in genere viene usato per fornire un messaggio durante il periodo temporaneo in cui la vista viene installata e i dati diventano disponibili. |
| Query |Query eseguita per determinare se sono disponibili dati per la vista. Se la query non restituisce risultati, viene visualizzato un messaggio al posto del valore dalla query principale. |
| `Message` |Messaggio visualizzato se la query di verifica del flusso di dati non restituisce dati. Se non si specifica alcun messaggio, viene visualizzato il messaggio di stato *Esecuzione della valutazione*. |


## <a name="donut-tile"></a>Riquadro Anello
Il riquadro **Anello** visualizza un singolo numero che riepiloga una colonna di valori in una query di log. L'anello visualizza graficamente i risultati dei tre record principali.

![Riquadro Anello](media/view-designer-tiles/tile-donut.png)

| Impostazione | DESCRIZIONE |
|:--- |:--- |
| Name |Testo che viene visualizzato nella parte superiore del riquadro. |
| Descrizione |Testo visualizzato sotto il nome del riquadro. |
| **Anello** | |
| Query |Query eseguita per il grafico ad anello. La prima proprietà è un valore di testo, mentre la seconda è un valore numerico. Questa query usa in genere la parola chiave *measure* per riepilogare i risultati. |
| **Anello** |**> Centro** |
| Text |Testo che viene visualizzato sotto il valore all'interno del grafico ad anello. |
| Operazione |Operazione eseguita sulla proprietà value per riepilogarla come valore singolo.<ul><li>Somma: sommare i valori di tutti i record con il valore della proprietà.</li><li>Percentuale: percentuale dei valori sommati dai record con il valore di proprietà rispetto ai valori sommati di tutti i record.</li></ul> |
| I valori dei risultati usati nell'operazione centrale |Facoltativamente, fare clic sul segno più (+) per aggiungere uno o più valori. I risultati della query saranno limitati ai record con i valori di proprietà specificati. Se non vengono aggiunti valori, nella query vengono inclusi tutti i record. |
| **Anello** |**> Opzioni aggiuntive** |
| Colori |Colore che viene visualizzato per ognuna delle tre proprietà principali. Per specificare colori alternativi per valori di proprietà specifici, usare *Mappa colori avanzata*. |
| Mappa colori avanzata |Visualizza un colore che rappresenta valori di proprietà specifici. Se il valore specificato è tra i primi tre, viene visualizzato il colore alternativo anziché il colore standard. Se il valore di proprietà non è tra i primi tre, il colore non viene visualizzato. |
| **Funzionalità avanzate** |**&gt; Verifica del flusso di dati** |
| Enabled |Selezionare questo collegamento se è necessario abilitare la verifica del flusso di dati per il riquadro. Questo approccio fornisce un messaggio alternativo se non sono disponibili dati. Questo approccio viene in genere viene usato per fornire un messaggio durante il periodo temporaneo in cui la vista viene installata e i dati diventano disponibili. |
| Query |Query eseguita per determinare se sono disponibili dati per la vista. Se la query non restituisce risultati, viene visualizzato un messaggio al posto del valore dalla query principale. |
| `Message` |Messaggio visualizzato se la query di verifica del flusso di dati non restituisce dati. Se non si specifica alcun messaggio, viene visualizzato il messaggio di stato *Esecuzione della valutazione*. |


## <a name="line-chart-tile"></a>Riquadro Grafico a linee
Questo riquadro contiene un grafico a linee con più serie provenienti da una query di log nel tempo. 

![Riquadro Grafico a linee e callout](media/view-designer-tiles/tile-line-chart.png)

| Impostazione | DESCRIZIONE |
|:--- |:--- |
| NOME |Testo che viene visualizzato nella parte superiore del riquadro. |
| Descrizione |Testo visualizzato sotto il nome del riquadro. |
| **Grafico a linee** | |
| Query |Query eseguita per il grafico a linee. La prima proprietà è un valore di testo, mentre la seconda è un valore numerico. Questa query usa in genere la parola chiave *measure* per riepilogare i risultati. Se la query usa la parola chiave *interval*, l'asse X usa questo intervallo di tempo. Se la query non usa la parola chiave *interval*, l'asse X usa intervalli orari. |
| **Grafico a linee** |**> Asse Y** |
| Usa scala logaritmica |Selezionare questo collegamento per usare una scala logaritmica per l'asse Y. |
| Unità |Specifica le unità per i valori restituiti dalla query. Queste informazioni vengono usate per visualizzare le etichette del grafico che indicano i tipi di valore e, facoltativamente, per convertire i valori. **Tipo di unità** specifica la categoria dell'unità e definisce i valori **Tipo di unità corrente** che sono disponibili. Se si seleziona un valore in **Converti in**, i valori numerici vengono convertiti dal tipo **Unità corrente** al tipo **Converti in**. |
| Etichetta personalizzata |Testo che viene visualizzato per l'asse Y accanto all'etichetta per il tipo *Unità*. Se non viene specificata alcuna etichetta, viene visualizzato solo il tipo *Unità*. |
| **Funzionalità avanzate** |**&gt; Verifica del flusso di dati** |
| Enabled |Selezionare questo collegamento se è necessario abilitare la verifica del flusso di dati per il riquadro. Questo approccio fornisce un messaggio alternativo se non sono disponibili dati. Questo approccio viene in genere viene usato per fornire un messaggio durante il periodo temporaneo in cui la vista viene installata e i dati diventano disponibili. |
| Query |Query eseguita per determinare se sono disponibili dati per la vista. Se la query non restituisce risultati, viene visualizzato un messaggio al posto del valore dalla query principale. |
| `Message` |Messaggio visualizzato se la query di verifica del flusso di dati non restituisce dati. Se non si specifica alcun messaggio, viene visualizzato il messaggio di stato *Esecuzione della valutazione*. |


## <a name="line-chart-and-callout-tile"></a>Riquadro Grafico a linee e callout
Questo riquadro visualizza un grafico a linee con più serie provenienti da una query di log nel tempo e un callout con un valore di riepilogo. 

![Riquadro Grafico a linee e callout](media/view-designer-tiles/tile-line-chart-callout.png)

| Impostazione | DESCRIZIONE |
|:--- |:--- |
| Name |Testo che viene visualizzato nella parte superiore del riquadro. |
| Descrizione |Testo visualizzato sotto il nome del riquadro. |
| **Grafico a linee** | |
| Query |Query eseguita per il grafico a linee. La prima proprietà è un valore di testo, mentre la seconda è un valore numerico. Questa query usa in genere la parola chiave *measure* per riepilogare i risultati. Se la query usa la parola chiave *interval*, l'asse X usa questo intervallo di tempo. Se la query non usa la parola chiave *interval*, l'asse X usa intervalli orari. |
| **Grafico a linee** |**> Callout** |
| Titolo del callout | Testo che viene visualizzato sopra il valore del callout. |
| Nome della serie |Valore della proprietà per la serie da usare come valore di callout. Se non viene fornita alcuna serie, vengono usati tutti i record prodotti dalla query. |
| Operazione |Operazione eseguita sulla proprietà value per riepilogarla come valore singolo per il callout.<ul><li>Media: media dei valori di tutti i record.</li><li>Conteggio: numero di tutti i record restituiti dalla query.</li><li>Ultimo esempio: valore dell'ultimo intervallo incluso nel grafico.</li><li>Max: valore massimo degli intervalli inclusi nel grafico.</li><li>Min: valore minimo degli intervalli inclusi nel grafico.</li><li>Somma: somma dei valori da tutti i record.</li></ul> |
| **Grafico a linee** |**> Asse Y** |
| Usa scala logaritmica |Selezionare questo collegamento per usare una scala logaritmica per l'asse Y. |
| Unità |Specificare le unità per i valori restituiti dalla query. Queste informazioni vengono usate per visualizzare le etichette del grafico che indicano i tipi di valore e, facoltativamente, per convertire i valori. Il tipo di *Unità* specifica la categoria dell'unità e definisce i valori del tipo *Unità corrente* disponibili. Se si seleziona un valore in *Converti in*, i valori numerici vengono convertiti dal tipo *Unità corrente* al tipo *Converti in*. |
| Etichetta personalizzata |Testo che viene visualizzato per l'asse Y accanto all'etichetta per il tipo *Unità*. Se non viene specificata alcuna etichetta, viene visualizzato solo il tipo *Unità*. |
| **Funzionalità avanzate** |**&gt; Verifica del flusso di dati** |
| Enabled |Selezionare questo collegamento se è necessario abilitare la verifica del flusso di dati per il riquadro. Questo approccio fornisce un messaggio alternativo se non sono disponibili dati. Questo approccio viene in genere viene usato per fornire un messaggio durante il periodo temporaneo in cui la vista viene installata e i dati diventano disponibili. |
| Query |Query eseguita per determinare se sono disponibili dati per la vista. Se la query non restituisce risultati, viene visualizzato un messaggio al posto del valore dalla query principale. |
| `Message` |Messaggio visualizzato se la query di verifica del flusso di dati non restituisce dati. Se non si specifica alcun messaggio, viene visualizzato il messaggio di stato *Esecuzione della valutazione*. |


## <a name="two-timelines-tile"></a>Riquadro Due sequenze temporali
Il riquadro **Due sequenze temporali** visualizza come istogramma i risultati di due query di log nel tempo. Viene visualizzato un callout per ogni serie. 

![Riquadro Due sequenze temporali](media/view-designer-tiles/tile-two-timelines.png)

| Impostazione | DESCRIZIONE |
|:--- |:--- |
| NOME |Testo che viene visualizzato nella parte superiore del riquadro. |
| Descrizione |Testo visualizzato sotto il nome del riquadro. |
| Primo grafico | |
| Legenda |Testo che viene visualizzato sotto il callout per la prima serie. |
| Colore |Colore usato per le colonne della prima serie. |
| Query grafico |Query eseguita per la prima serie. Il numero di record in ogni intervallo di tempo è rappresentato dalle colonne del grafico. |
| Operazione |Operazione eseguita sulla proprietà value per riepilogarla come valore singolo per il callout.<ul><li>Media: media dei valori di tutti i record.</li><li>Conteggio: numero di tutti i record restituiti dalla query.</li><li>Ultimo esempio: valore dell'ultimo intervallo incluso nel grafico.</li><li>Max: valore massimo degli intervalli inclusi nel grafico.</li></ul> |
| **Secondo grafico** | |
| Legenda |Testo che viene visualizzato sotto il callout per la seconda serie. |
| Colore |Colore usato per le colonne della seconda serie. |
| Query grafico |Query eseguita per la seconda serie. Il numero di record in ogni intervallo di tempo è rappresentato dalle colonne del grafico. |
| Operazione |Operazione eseguita sulla proprietà value per riepilogarla come valore singolo per il callout.<ul><li>Media: media dei valori di tutti i record.</li><li>Conteggio: numero di tutti i record restituiti dalla query.</li><li>Ultimo esempio: valore dell'ultimo intervallo incluso nel grafico.</li><li>Max: valore massimo degli intervalli inclusi nel grafico. |
| **Funzionalità avanzate** |**&gt; Verifica del flusso di dati** |
| Enabled |Selezionare questo collegamento se è necessario abilitare la verifica del flusso di dati per il riquadro. Questo approccio fornisce un messaggio alternativo se non sono disponibili dati. Questo approccio viene in genere viene usato per fornire un messaggio durante il periodo temporaneo in cui la vista viene installata e i dati diventano disponibili. |
| Query |Query eseguita per determinare se sono disponibili dati per la vista. Se la query non restituisce risultati, viene visualizzato un messaggio al posto del valore dalla query principale. |
| `Message` |Messaggio visualizzato se la query di verifica del flusso di dati non restituisce dati. Se non si specifica alcun messaggio, viene visualizzato il messaggio di stato *Esecuzione della valutazione*. |


## <a name="next-steps"></a>Passaggi successivi
* Scopri [registrare query](../log-query/log-query-overview.md) per supportare le query nei riquadri.
* Aggiungere [parti della visualizzazione](view-designer-parts.md) alla vista personalizzata.
