---
title: Usare la parallelizzazione e il ridimensionamento delle query in Analisi di flusso di Azure
description: Questo articolo descrive come ridimensionare i processi di Analisi di flusso configurando partizioni di input, ottimizzando la definizione delle query e impostando le unità di streaming dei processi.
services: stream-analytics
author: JSeb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/07/2018
ms.openlocfilehash: 5eba5601a50640261fa1b488d959f606d4514737
ms.sourcegitcommit: 6a42dd4b746f3e6de69f7ad0107cc7ad654e39ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/07/2019
ms.locfileid: "67612226"
---
# <a name="leverage-query-parallelization-in-azure-stream-analytics"></a>Sfruttare i vantaggi della parallelizzazione delle query in Analisi di flusso di Azure
Questo articolo illustra come sfruttare i vantaggi della parallelizzazione in Analisi di flusso di Azure. Si apprenderà come ridimensionare i processi di Analisi di flusso configurando partizioni di input e ottimizzando la definizione di query.
Come prerequisito è necessario conoscere la nozione di unità di streaming descritta in [Informazioni e modifica delle unità di streaming](stream-analytics-streaming-unit-consumption.md).

## <a name="what-are-the-parts-of-a-stream-analytics-job"></a>Quali sono le parti di un processo di Analisi di flusso?
Una definizione del processo di Analisi di flusso include input, query e output. Gli input sono le origini da cui il processo legge il flusso di dati, la query viene usata per trasformare il flusso di input dei dati e l'output è la destinazione a cui il processo invia i risultati.

Un processo richiede almeno un'origine di input per il flusso dei dati. L'origine dell'input del flusso dei dati può essere archiviata in un hub eventi di Azure o in una risorsa di archiviazione BLOB di Azure. Per altre informazioni, vedere [Introduzione all'analisi di flusso di Azure](stream-analytics-introduction.md) e [Introduzione all'uso dell'analisi di flusso di Azure](stream-analytics-real-time-fraud-detection.md).

## <a name="partitions-in-sources-and-sinks"></a>Partizioni in origini e sink
Il ridimensionamento di un processo di Analisi di flusso sfrutta i vantaggi offerti dall'uso di partizioni nell'input o nell'output. Il partizionamento consente di suddividere i dati in subset in base a una chiave di partizione. Un processo che utilizza i dati, come avviene per i processi di Analisi di flusso, può utilizzare diverse partizioni e scrivervi in parallelo, aumentando così la velocità effettiva. 

### <a name="inputs"></a>Input
Tutti gli input di Analisi di flusso di Azure possono sfruttare i vantaggi del partizionamento:
-   Hub eventi (è necessario impostare la chiave di partizione in modo esplicito con PARTITION BY)
-   Hub IoT (è necessario impostare la chiave di partizione in modo esplicito con PARTITION BY)
-   Archiviazione BLOB

### <a name="outputs"></a>Output

Quando si usa Analisi di flusso di Azure, è possibile sfruttare il partizionamento negli output:
-   Archiviazione di Azure Data Lake
-   Funzioni di Azure
-   Tabella di Azure
-   Archiviazione BLOB (è necessario impostare la chiave di partizione in modo esplicito)
-   Cosmos DB (è necessario impostare la chiave di partizione in modo esplicito)
-   Hub eventi (è necessario impostare la chiave di partizione in modo esplicito)
-   Hub IoT (è necessario impostare la chiave di partizione in modo esplicito)
-   Bus di servizio
- Per altre informazioni su SQL e SQL Data Warehouse con il partizionamento facoltativo, vedere la [pagina dell'output in Database SQL di Azure](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-sql-output-perf).

Power BI non supporta il partizionamento. È tuttavia possibile suddividere gli input in partizioni come descritto in [questa sezione](#multi-step-query-with-different-partition-by-values) 

Per altre informazioni sulle partizioni, vedere gli articoli seguenti:

* [Panoramica delle funzionalità di Hub eventi](../event-hubs/event-hubs-features.md#partitions)
* [Partizionamento dei dati](https://docs.microsoft.com/azure/architecture/best-practices/data-partitioning)


## <a name="embarrassingly-parallel-jobs"></a>Processi perfettamente paralleli
Un processo *perfettamente parallelo* è lo scenario più scalabile che può presentarsi in Analisi di flusso di Azure. Connette una partizione dell'input inviato a un'istanza della query a una partizione dell'output. Questo parallelismo presenta i requisiti seguenti:

1. Se la logica di query richiede che la stessa chiave venga elaborata dalla stessa istanza di query, è necessario verificare che gli eventi siano diretti alla stessa partizione dell'input. Per gli hub eventi e l'hub IoT, questo significa che per i dati degli eventi deve essere impostata la proprietà **PartitionKey**. In alternativa, è possibile usare mittenti partizionati. Per l'archiviazione BLOB, questo significa che gli eventi vengono inviati alla stessa cartella di partizione. Se la logica di query non richiede che la stessa chiave venga elaborata dalla stessa istanza di query, è possibile ignorare questo requisito. Un esempio di questa logica è offerto da una query semplice select-project-filter.  

2. Quando i dati sono disposti a livello di input, si deve verificare che la query sia partizionata. A questo scopo, è necessario usare la clausola **PARTITION BY** in tutti i passaggi. È possibile eseguire più passaggi, ma tutti devono essere partizionati con la stessa chiave. Con il livello di compatibilità 1.0 e 1.1, la chiave di partizionamento deve essere impostata su **PartitionId** affinché il processo completamente parallelo. Per i processi con livello di compatibilità dell'1.2 e versioni successiva, colonna personalizzata può essere specificato come chiave di partizione nelle impostazioni di input e il processo sarà automoatically paralellized anche senza clausola PARTITION BY.

3. La maggior parte degli output può sfruttare i vantaggi del partizionamento. Se tuttavia si usa un tipo di output che non supporta il partizionamento, il processo non sarà perfettamente parallelo. Per altri dettagli, vedere la [sezione output](#outputs).

4. Il numero delle partizioni di input deve essere uguale a quello delle partizioni di output. L'output dell'archiviazione BLOB può supportare le partizioni ed eredita lo schema di partizionamento della query a monte. Quando viene specificata una chiave di partizione per l'archiviazione BLOB, i dati vengono partizionati per partizione di input, pertanto il risultato è ancora completamente parallelo. Ecco alcuni esempi di valori di partizioni che consentono un processo perfettamente parallelo:

   * 8 partizioni di input di hub eventi e 8 partizioni di output di hub eventi
   * 8 partizioni di input di hub eventi e output di archiviazione BLOB
   * 8 partizioni di input dell'hub eventi e output di archiviazione BLOB partizionato in base a un campo personalizzato con cardinalità arbitraria
   * 8 partizioni di input di archiviazione BLOB e output di archiviazione BLOB
   * 8 partizioni di input di archiviazione BLOB e 8 partizioni di output di hub eventi

Le sezioni seguenti illustrano alcuni esempi di scenari perfettamente paralleli.

### <a name="simple-query"></a>Query semplice

* Input: hub eventi con 8 partizioni
* Output: hub eventi con 8 partizioni

Query:

```SQL
    SELECT TollBoothId
    FROM Input1 Partition By PartitionId
    WHERE TollBoothId > 100
```

Questa query è un filtro semplice. Non è pertanto necessario preoccuparsi del partizionamento dell'input inviato all'hub eventi. Si noti che i processi con livello di compatibilità prima 1.2 deve includere **PARTITION BY PartitionId** clausola, quindi il requisito #2 illustrato in precedenza. A livello di output, è necessario configurare l'output dell'hub eventi nel processo in modo che la chiave di partizione sia impostata su **PartitionId**. È infine necessario verificare che il numero delle partizioni di input sia uguale a quello delle partizioni di output.

### <a name="query-with-a-grouping-key"></a>Query con chiave di raggruppamento

* Input: hub eventi con 8 partizioni
* Output: Archiviazione BLOB

Query:

```SQL
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
```

Questa query include una chiave di raggruppamento. Gli eventi raggruppati insieme devono quindi essere inviati alla stessa partizione dell'hub eventi. Nell'esempio viene illustrato il raggruppamento per TollBoothID, è quindi necessario assicurarsi che TollBoothID venga usato come chiave di partizione quando gli eventi vengono inviati all'hub eventi. In ASA è possibile usare **PARTITION BY PartitionId** per ereditare da questo schema di partizione e abilitare la parallelizzazione completa. Poiché l'output è costituito dall'archiviazione BLOB, non occorre preoccuparsi di configurare un valore di chiave di partizione, come definito dal requisito 4.

## <a name="example-of-scenarios-that-are-not-embarrassingly-parallel"></a>Esempi di scenari *non* perfettamente paralleli

Nella sezione precedente sono stati presentati alcuni scenari perfettamente paralleli. In questa sezione si illustreranno scenari che non soddisfano tutti i requisiti di parallelismo perfetto. 

### <a name="mismatched-partition-count"></a>Numero di partizioni non corrispondente
* Input: hub eventi con 8 partizioni
* Output: hub eventi con 32 partizioni

In questo caso, la query non è rilevante. Se il numero delle partizioni di input non corrisponde a quello delle partizioni di output, la topologia non è perfettamente parallela, ma è possibile comunque ottenere un certo livello di parallelizzazione.

### <a name="query-using-non-partitioned-output"></a>Query con output non partizionati
* Input: hub eventi con 8 partizioni
* Output: Power BI

L'output Power BI attualmente non supporta il partizionamento. Pertanto, questo scenario non è perfettamente parallelo.

### <a name="multi-step-query-with-different-partition-by-values"></a>Query a più passaggi con valori diversi per PARTITION BY
* Input: hub eventi con 8 partizioni
* Output: hub eventi con 8 partizioni

Query:

```SQL
    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId, PartitionId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1 Partition By TollBoothId
    GROUP BY TumblingWindow(minute, 3), TollBoothId
```

Come è possibile osservare, il secondo passaggio usa **TollBoothId** come chiave di partizionamento, a differenza del primo passaggio. È quindi necessario eseguire una riproduzione in ordine casuale. 

Gli esempi precedenti illustrano alcuni processi di Analisi di flusso che sono o non sono conformi a una topologia perfettamente parallela. Se sono conformi, possono raggiungere il livello massimo di scalabilità. Per i processi che non rientrano in nessuno di questi profili, in futuro saranno disponibili aggiornamenti con le linee guida per il ridimensionamento. Per il momento, seguire le indicazioni generali riportate nelle sezioni seguenti.

### <a name="compatibility-level-12---multi-step-query-with-different-partition-by-values"></a>Livello di compatibilità 1.2 - query a più passaggi con valori diversi per PARTITION BY 
* Input: hub eventi con 8 partizioni
* Output: hub eventi con 8 partizioni

Query:

```SQL
    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
```

Livello di compatibilità 1.2 consente l'esecuzione di query in parallelo per impostazione predefinita. Ad esempio, query nella sezione precedente sarà parttioned, purché la colonna "TollBoothId" è impostata come chiave di partizione di input. PARTIZIONE da ParttionId clausola non è obbligatorio.

## <a name="calculate-the-maximum-streaming-units-of-a-job"></a>Calcolare il numero massimo di unità di streaming di un processo
Il numero totale di unità di streaming che possono essere usate da un processo di Analisi dei flussi dipende dal numero di passaggi nella query definita per il processo e dal numero di partizioni per ogni passaggio.

### <a name="steps-in-a-query"></a>Passaggi in una query
Una query può includere uno o più passaggi. Ogni passaggio è una sottoquery definita mediante la parola chiave **WITH**. Anche la query esterna alla parola chiave **WITH** (una sola query) viene contata come passaggio. Ad esempio, l'istruzione **SELECT** nella query seguente:

Query:

```SQL
    WITH Step1 AS (
        SELECT COUNT(*) AS Count, TollBoothId
        FROM Input1 Partition By PartitionId
        GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )
    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute,3), TollBoothId
```

Questa query include due passaggi.

> [!NOTE]
> Questa query viene illustrata in dettaglio più avanti nell'articolo.
>  

### <a name="partition-a-step"></a>Partizionamento di un passaggio
Il partizionamento di un passaggio richiede le condizioni seguenti:

* L'origine di input deve essere partizionata. 
* L'istruzione **SELECT** della query deve leggere da un'origine di input partizionata.
* La query all'interno del passaggio deve includere la parola chiave **PARTITION BY**.

Quando una query è partizionata, gli eventi di input vengono elaborati e aggregati in gruppi separati di partizioni e per ogni gruppo vengono generati eventi di output. Se si vuole un aggregato combinato, è necessario creare un secondo passaggio non partizionato per l'aggregazione.

### <a name="calculate-the-max-streaming-units-for-a-job"></a>Calcolare il numero massimo di unità di streaming per un processo
Per un processo di Analisi di flusso, l'insieme di tutti i passaggi non partizionati può raggiungere un massimo di sei unità di streaming. È inoltre possibile aggiungere 6 unità di archiviazione per ciascuna partizione in un passaggio partizionato.
Nella tabella seguente sono riportati alcuni **esempi**.

| Query                                               | Numero massimo di unità di streaming per il processo |
| --------------------------------------------------- | ------------------- |
| <ul><li>La query contiene un unico passaggio.</li><li>Il passaggio non è partizionato.</li></ul> | 6 |
| <ul><li>Il flusso di dati di input è suddiviso in 16 partizioni.</li><li>La query contiene un unico passaggio.</li><li>Il passaggio è partizionato.</li></ul> | 96 (6 * 16 partizioni) |
| <ul><li>La query contiene due passaggi.</li><li>Nessuno dei passaggi è partizionato.</li></ul> | 6 |
| <ul><li>Il flusso di dati di input è suddiviso in 3 partizioni.</li><li>La query contiene due passaggi. Il passaggio di input viene partizionato, al contrario del secondo passaggio.</li><li>L'istruzione <strong>SELECT</strong> legge dall'input partizionato.</li></ul> | 24 (18 per i passaggi partizionati + 6 per i passaggi non partizionati) |

### <a name="examples-of-scaling"></a>Esempi di ridimensionamento

La query seguente calcola il numero di automobili che passano per una stazione di pedaggio con tre caselli in un intervallo di tre minuti. Il numero di unità di streaming di questa query può essere aumentato fino a sei.

```SQL
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
```

Per usare più unità di streaming per la query, è necessario che il flusso di dati di input e la query siano entrambi partizionati. Se il partizionamento del flusso di dati è impostato su 3, la query modificata seguente può essere ridimensionata fino a un massimo di 18 unità di streaming:

```SQL
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
```

Quando una query è partizionata, gli eventi di input vengono elaborati e aggregati in gruppi di partizioni separati e vengono anche generati eventi di output per ognuno dei gruppi. Quando il campo **GROUP BY** non corrisponde alla chiave di partizione nel flusso di dati di input, il partizionamento può causare risultati imprevisti. Ad esempio, il campo **TollBoothId** nella query precedente non corrisponde alla chiave di partizione **Input1**. I dati provenienti dal casello 1 possono pertanto essere distribuiti in più partizioni.

Le singole partizioni di **Input1** verranno elaborate separatamente da Analisi di flusso. Verranno pertanto creati più record del conteggio relativo al passaggio di automobili dallo stesso casello nella stessa finestra a cascata. Se la chiave di partizione di input non può essere modificata, è possibile risolvere questo problema aggiungendo un altro passaggio non di partizione per aggregare i valori tra le partizioni, come nell'esempio seguente:

```SQL
    WITH Step1 AS (
        SELECT COUNT(*) AS Count, TollBoothId
        FROM Input1 Partition By PartitionId
        GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
```

Per questa query è possibile aumentare il numero di unità di streaming fino a 24.

> [!NOTE]
> Se si uniscono due flussi, verificare che tali flussi siano partizionati in base alla chiave di partizione della colonna usata per le unioni. Controllare inoltre che in entrambi i flussi sia presente lo stesso numero di partizioni.
> 
> 

## <a name="achieving-higher-throughputs-at-scale"></a>Come raggiungere una maggiore velocità effettiva su larga scala

Un' [perfettamente paralleli](#embarrassingly-parallel-jobs) processo è necessaria ma non sufficiente a sostenere una velocità effettiva superiore su larga scala. Ogni sistema di archiviazione e il relativo output Stream Analitica corrispondente include varianti su come ottenere la velocità effettiva migliore possibile scrittura. Come con tutti gli scenari su larga scala, esistono alcuni problemi che possono essere risolti con le configurazioni a destra. In questa sezione illustra le configurazioni per alcuni output comuni e vengono forniti esempi per garantire alti tassi di inserimento di 1 KB, pari a 5 KB e 10 k rpm eventi al secondo.

Le osservazioni seguenti usano un processo di Stream Analitica con query senza stato (pass-through), una basic UDF di JavaScript che scrive in Hub eventi, Azure SQL DB o Cosmos DB.

#### <a name="event-hub"></a>Hub eventi

|Frequenza di inserimento (eventi al secondo) | Unità di streaming | Risorse di output  |
|--------|---------|---------|
| 1K     |    1    |  2 UNITÀ ELABORATE   |
| PARI A 5 KB     |    6    |  6 UNITÀ ELABORATE   |
| 10.000    |    12   |  10 UNITÀ ELABORATE  |

Il [Hub eventi](https://github.com/Azure-Samples/streaming-at-scale/tree/master/eventhubs-streamanalytics-eventhubs) soluzione scalate in modo lineare in termini di unità (unità di streaming) e velocità effettiva, rendendo più efficiente e ad alte prestazioni modo di analisi di flusso di dati all'esterno di Stream Analitica di streaming. I processi possono essere ridimensionati fino a 192 unità di streaming, si traduce approssimativamente in elaborazione fino a 200 MB/s o 19 miliardi di eventi al giorno.

#### <a name="azure-sql"></a>SQL di Azure
|Frequenza di inserimento (eventi al secondo) | Unità di streaming | Risorse di output  |
|---------|------|-------|
|    1K   |   3  |  S3   |
|    PARI A 5 KB   |   18 |  P4   |
|    10.000  |   36 |  P6   |

[SQL Azure](https://github.com/Azure-Samples/streaming-at-scale/tree/master/eventhubs-streamanalytics-azuresql) supporta la scrittura in parallelo, chiamato ereditare partizione, ma non è abilitato per impostazione predefinita. Tuttavia, l'abilitazione di ereditare partizione, insieme a una query perfettamente parallela, potrebbe non essere sufficiente per ottenere una maggiore velocità effettiva. Produttività scrittura SQL dipendono in modo significativo lo schema di configurazione e la tabella di database di SQL Azure. Il [le prestazioni di Output SQL](./stream-analytics-sql-output-perf.md) articolo è più in dettaglio i parametri che è possibile ottimizzare la velocità effettiva di scrittura. Come indicato nella [output Analitica Stream di Azure al Database SQL di Azure](./stream-analytics-sql-output-perf.md#azure-stream-analytics) articolo, questa soluzione non supporta la scalabilità in modo lineare come pipeline perfettamente parallela oltre 8 partizioni e potrebbe essere necessario ripartizionare prima di output SQL (vedere [ IN](https://docs.microsoft.com/stream-analytics-query/into-azure-stream-analytics#into-shard-count)). SKU Premium sono necessari a supportare frequenze dei / o elevate con overhead dal backup del log che accade ogni pochi minuti.

#### <a name="cosmos-db"></a>Cosmos DB
|Frequenza di inserimento (eventi al secondo) | Unità di streaming | Risorse di output  |
|-------|-------|---------|
|  1K   |  3    | 20K RU  |
|  PARI A 5 KB   |  24   | 60K UR  |
|  10.000  |  48   | 120K RU |

[COSMOS DB](https://github.com/Azure-Samples/streaming-at-scale/tree/master/eventhubs-streamanalytics-cosmosdb) output Stream Analitica è stata aggiornata per usare l'integrazione nativa con [livello di compatibilità 1.2](./stream-analytics-documentdb-output.md#improved-throughput-with-compatibility-level-12). Livello di compatibilità 1.2 consente una velocità effettiva notevolmente superiore e riduce il consumo di UR rispetto a 1.1, che è il livello di compatibilità predefinito per i nuovi processi. La soluzione Usa i contenitori di COSMOS DB partizionati in base alla /deviceId e il resto della soluzione è configurato in modo identico.

Tutti i [lo Streaming in esempi di azure scala](https://github.com/Azure-Samples/streaming-at-scale) usare un Hub eventi di alimentazione dal carico che simulano i client di test come input. Ogni evento di input è un documento JSON da 1KB, che viene convertita facilmente le frequenze di inserimento configurato per la velocità effettiva (1MB/s, 5MB/s e 10MB/s). Gli eventi di simulano un dispositivo IoT invia i dati JSON seguenti (in una forma abbreviata) per i dispositivi di fino a 1 KB:

```
{
    "eventId": "b81d241f-5187-40b0-ab2a-940faf9757c0",
    "complexData": {
        "moreData0": 51.3068118685458,
        "moreData22": 45.34076957651598
    },
    "value": 49.02278128887753,
    "deviceId": "contoso://device-id-1554",
    "type": "CO2",
    "createdAt": "2019-05-16T17:16:40.000003Z"
}
```

> [!NOTE]
> Le configurazioni sono soggette a modifiche a causa di vari componenti usati nella soluzione. Per una stima più accurata, personalizzare gli esempi per adattarlo allo scenario.

### <a name="identifying-bottlenecks"></a>Identificare i colli di bottiglia

Utilizzare il riquadro metriche nel processo di Azure Stream Analitica per identificare i colli di bottiglia nella pipeline. Revisione **eventi di Input/Output** per la velocità effettiva e ["Ritardo della filigrana"](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/) oppure **eventi backlog sta** per vedere se il processo è stato aggiornato con la frequenza di input. Per le metriche di Hub eventi, cercare **richieste limitate** e modificare di conseguenza le unità di soglia. Per le metriche di Cosmos DB, esaminare **numero massimo di unità richiesta al secondo utilizzate per ogni intervallo di chiavi di partizione** sotto la velocità effettiva per verificare gli intervalli di chiavi di partizione vengono utilizzati in modo uniforme. Per il database SQL di Azure, monitorare **i/o Log** e **CPU**.

## <a name="get-help"></a>Ottenere aiuto

Per ulteriore assistenza, provare il [Forum di Analisi dei flussi di Azure](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics).

## <a name="next-steps"></a>Passaggi successivi
* [Introduzione ad Analisi dei flussi di Azure](stream-analytics-introduction.md)
* [Introduzione all'uso di Analisi dei flussi di Azure](stream-analytics-real-time-fraud-detection.md)
* [Informazioni di riferimento sul linguaggio di query di Analisi dei flussi di Azure](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Informazioni di riferimento sulle API REST di gestione di Analisi di flusso di Azure](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor-NewPortal.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg
[img.stream.analytics.preview.portal.settings.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsPreviewPortalJobSettings-NewPortal.png   

<!--Link references-->

[microsoft.support]: https://support.microsoft.com
[azure.event.hubs.developer.guide]: https://msdn.microsoft.com/library/azure/dn789972.aspx

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301

