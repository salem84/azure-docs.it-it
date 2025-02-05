---
title: Confrontare Azure Data Factory con Data Factory versione 1 | Microsoft Docs
description: Questo articolo mette a confronto Azure Data Factory e Azure Data Factory versione 1.
services: data-factory
documentationcenter: ''
author: kromerm
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: overview
ms.date: 04/09/2018
ms.author: makromer
ms.openlocfilehash: 4cdb517e644d55504bfdafbd3bacdfd4bfa0b36c
ms.sourcegitcommit: 75a56915dce1c538dc7a921beb4a5305e79d3c7a
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/24/2019
ms.locfileid: "68479309"
---
# <a name="compare-azure-data-factory-with-data-factory-version-1"></a>Confrontare Azure Data Factory con Data Factory versione 1
Questo articolo mette a confronto Data Factory e Data Factory versione 1. Per un'introduzione a Data Factory, vedere [Introduzione a Data Factory](introduction.md). Per un'introduzione a Data Factory versione 1, vedere [Introduzione ad Azure Data Factory](v1/data-factory-introduction.md). 

## <a name="feature-comparison"></a>Confronto tra le funzionalità
La tabella seguente mette a confronto le funzionalità di Data Factory con quelle di Data Factory versione 1. 

| Funzionalità | Versione 1 | Versione corrente | 
| ------- | --------- | --------- | 
| Set di dati | Vista denominata di dati che fa riferimento ai dati da usare nelle attività come input e output. I set di dati identificano i dati all'interno dei diversi archivi dati, come tabelle, file, cartelle e documenti. Un set di dati BLOB di Azure, ad esempio, specifica il contenitore BLOB e la cartella nell'archivio BLOB di Azure da cui l'attività dovrà leggere i dati.<br/><br/>La **disponibilità** definisce il modello di sezionamento dell'intervallo di elaborazione del set di dati (ad esempio ogni ora, ogni giorno e così via). | I set di dati sono gli stessi nella versione corrente. Non è tuttavia necessario definire pianificazioni della **disponibilità** per i set di dati. È possibile definire una risorsa trigger che può pianificare le pipeline da un paradigma di utilità di pianificazione basata sul tempo. Per altre informazioni, vedere [Trigger](concepts-pipeline-execution-triggers.md#triggers) e [Set di dati](concepts-datasets-linked-services.md). | 
| Servizi collegati | I servizi collegati sono simili a stringhe di connessione e definiscono le informazioni necessarie per la connessione di Data Factory a risorse esterne. | I servizi collegati sono gli stessi di Data Factory versione 1, ma con una nuova proprietà **connectVia** per l'utilizzo dell'ambiente di calcolo del runtime di integrazione della versione corrente di Data Factory. Per altre informazioni, vedere [Runtime di integrazione in Azure Data Factory](concepts-integration-runtime.md) e le [proprietà del servizio collegato per l'archivio BLOB di Azure](connector-azure-blob-storage.md#linked-service-properties). |
| Pipeline | Una data factory può comprendere una o più pipeline. Una pipeline è un raggruppamento logico di attività che insieme eseguono un compito. Per pianificare ed eseguire le pipeline, si usano startTime, endTime e isPaused. | Le pipeline sono gruppi di attività eseguite sui dati. La pianificazione delle attività nella pipeline è tuttavia stata separata in nuove risorse trigger. È possibile pensare alle pipeline della versione corrente di Data Factory come “unità di flusso di lavoro” che vengono pianificate separatamente tramite trigger. <br/><br/>Le pipeline non hanno “finestre” di esecuzione temporale nella versione corrente di Data Factory. I concetti di startTime, endTime e isPaused di Data Factory versione 1 non sono più presenti nella versione corrente di Data Factory. Per altre informazioni, vedere [Esecuzione e trigger di pipeline](concepts-pipeline-execution-triggers.md) e [Pipeline e attività](concepts-pipelines-activities.md). |
| attività | Le attività definiscono le azioni da eseguire sui dati in una pipeline. Sono supportate attività di spostamento dei dati (attività di copia) e di trasformazione dei dati (ad esempio Hive, Pig e MapReduce). | Nella versione corrente di Data Factory le attività sono ancora azioni definite all'interno di una pipeline. La versione corrente di Data Factory introduce nuove [attività del flusso di controllo](concepts-pipelines-activities.md#control-activities). Queste attività vengono usate in un flusso di controllo (cicli e diramazioni). Le attività di spostamento dati e di trasformazione dati supportate nella versione 1 sono supportate anche nella versione corrente. Nella versione corrente è possibile definire attività di trasformazione senza usare i set di dati. |
| Invio di attività e spostamento dei dati di tipo ibrido | Il [gateway di gestione dati](v1/data-factory-data-management-gateway.md), ora denominato runtime di integrazione, supporta lo spostamento di dati tra ambiente locale e cloud.| Il gateway di gestione dati è ora denominato runtime di integrazione self-hosted e offre le stesse funzionalità disponibili nella versione 1. <br/><br/> Il runtime di integrazione Azure-SSIS nella versione corrente di Data Factory supporta anche la distribuzione e l'esecuzione di pacchetti di SQL Server Integration Services (SSIS) nel cloud. Per altre informazioni, vedere il [Runtime di integrazione in Azure Data Factory](concepts-integration-runtime.md).|
| Parametri | ND | I parametri sono coppie chiave-valore di impostazioni di configurazione di sola lettura definite nelle pipeline. È possibile passare argomenti per i parametri quando si esegue manualmente la pipeline. Se si usa un trigger per l'utilità di pianificazione, anche il trigger può passare valori per i parametri. Le attività all'interno della pipeline usano i valori dei parametri.  |
| Espressioni | Data Factory V1 consente l'uso di funzioni e variabili di sistema nelle query di selezione dei dati e nelle proprietà relative ad attività e set di dati. | Nella versione corrente di Data Factory è possibile usare le espressioni in qualsiasi punto in un valore stringa JSON. Per altre informazioni, vedere [Espressioni e funzioni nella versione corrente di Data Factory](control-flow-expression-language-functions.md).|
| Esecuzioni di pipeline | ND | Una singola istanza di esecuzione di pipeline. Se una pipeline viene eseguita alle ore 8, alle 9 e alle 10, ad esempio, si hanno tre esecuzioni di pipeline separate, ognuna con un ID di esecuzione pipeline univoco. L'ID di esecuzione pipeline è un GUID che definisce in modo univoco la specifica esecuzione della pipeline. Le istanze delle esecuzioni di pipeline vengono in genere create passando argomenti ai parametri definiti nelle pipeline. |
| Esecuzioni attività | ND | Un'istanza di esecuzione di attività in una pipeline. | 
| Esecuzioni di trigger | ND | Un'istanza di esecuzione di trigger. Per altre informazioni, vedere [Trigger](concepts-pipeline-execution-triggers.md). |
| Pianificazione | La pianificazione è basata sull'ora di inizio e di fine della pipeline e sulla disponibilità del set di dati. | Trigger dell'utilità di pianificazione o esecuzione tramite utilità di pianificazione esterna. Per altre informazioni, vedere [Esecuzione e trigger della pipeline](concepts-pipeline-execution-triggers.md). |

Le sezioni seguenti offrono altre informazioni sulle funzionalità della versione corrente. 

## <a name="control-flow"></a>Flusso di controllo  
Per supportare i diversi modelli e flussi di integrazione in un data warehouse moderno, nella versione corrente di Data Factory è stato abilitato un nuovo modello di pipeline di dati flessibile non più associato a dati di serie temporali. Sono ora abilitati alcuni flussi comuni in precedenza non supportati, descritti nelle sezioni seguenti.   

### <a name="chaining-activities"></a>Concatenamento di attività
Nella versione 1, per concatenare le attività è necessario configurare l'output di un'attività come input di un'altra. Nella versione corrente è possibile concatenare le attività in una sequenza all'interno di una pipeline. È possibile usare la proprietà **dependsOn** in una definizione di attività per concatenare l'attività a un'attività upstream. Per altre informazioni e un esempio, vedere [Pipeline e attività](concepts-pipelines-activities.md#multiple-activities-in-a-pipeline) e [Diramazione e concatenamento delle attività](tutorial-control-flow.md). 

### <a name="branching-activities"></a>Diramazione di attività
Nella versione corrente è possibile creare un ramo delle attività all'interno di una pipeline. L'[attività If-condition](control-flow-if-condition-activity.md) svolge la stessa funzione dell'istruzione `if` nei linguaggi di programmazione. Valuta un set di attività se la condizione restituisce `true` e un altro set di attività se la condizione restituisce `false`. Per un esempio di diramazione delle attività, vedere l'esercitazione [Diramazione e concatenamento delle attività](tutorial-control-flow.md).

### <a name="parameters"></a>Parametri 
È possibile definire parametri a livello di pipeline e passare argomenti quando si richiama la pipeline on demand o da un trigger. Le attività possono utilizzare gli argomenti passati alla pipeline. Per altre informazioni, vedere [Pipeline e trigger](concepts-pipeline-execution-triggers.md). 

### <a name="custom-state-passing"></a>Passaggio di stato personalizzato
Gli output delle attività che includono lo stato possono essere usati da un'attività successiva della pipeline. Nella definizione JSON di un'attività, ad esempio, è possibile accedere all'output dell'attività precedente tramite la sintassi seguente: `@activity('NameofPreviousActivity').output.value`. Usando questa funzionalità è possibile creare flussi di lavoro in cui i valori possono essere passati nelle diverse attività.

### <a name="looping-containers"></a>Contenitori di ciclo
L'[attività ForEach](control-flow-for-each-activity.md) definisce un flusso di controllo ripetuto nella pipeline. Questa attività scorre una raccolta ed esegue attività specifiche in un ciclo. L'implementazione in cicli di questa attività è simile alla struttura di esecuzione in cicli Foreach nei linguaggi di programmazione. 

L'attività [Until](control-flow-until-activity.md) svolge la stessa funzione della struttura di ciclo do-until nei linguaggi di programmazione. Esegue infatti un set di attività in un ciclo finché la condizione associata all'attività non restituisce `true`. È possibile specificare un valore di timeout per l'attività Until in Data Factory.  

### <a name="trigger-based-flows"></a>Flussi attivati da trigger
Le pipeline possono essere attivate su richiesta (basate su eventi, ad esempio BLOB Post) o con una pianificazione basata sul tempo. Informazioni dettagliate sui trigger sono disponibili nell'articolo relativo a [pipeline e trigger](concepts-pipeline-execution-triggers.md). 

### <a name="invoking-a-pipeline-from-another-pipeline"></a>Chiamata di una pipeline da un'altra pipeline
L'[attività ExecutePipeline](control-flow-execute-pipeline-activity.md) consente a una pipeline di Data Factory di richiamare un'altra pipeline.

### <a name="delta-flows"></a>Flussi delta
Un caso d'uso chiave nel modello ETL è rappresentato dai "caricamenti differenziali", con cui vengono caricati solo i dati che sono stati modificati dall'ultima iterazione di una pipeline. Le nuove funzionalità della versione corrente, ad esempio l'[attività Lookup](control-flow-lookup-activity.md), la pianificazione flessibile e il flusso di controllo, permettono di attuare questo caso d'uso in modo naturale. Per un'esercitazione con istruzioni dettagliate, vedere [Esercitazione: Copia incrementale](tutorial-incremental-copy-powershell.md).

### <a name="other-control-flow-activities"></a>Altre attività di flusso di controllo
Di seguito sono riportate alcune altre attività di flusso di controllo supportate dalla versione corrente di Data Factory. 

Attività di controllo | DESCRIZIONE
---------------- | -----------
[Attività ForEach](control-flow-for-each-activity.md) | Definisce un flusso di controllo ripetuto nella pipeline. Questa attività viene usata per scorrere una raccolta ed eseguire attività specifiche in un ciclo. L'implementazione di cicli di questa attività è simile alla struttura di esecuzione in ciclo Foreach nei linguaggi di programmazione.
[Attività Web](control-flow-web-activity.md) | Chiama un endpoint REST personalizzato da una pipeline di Data Factory. È possibile passare set di dati e servizi collegati in modo che l'attività possa usarli e accedervi. 
[Attività Lookup](control-flow-lookup-activity.md) | Legge o cerca un valore di nome di record o tabella in qualsiasi origine esterna. Questo output può essere referenziato ulteriormente dalle attività successive. 
[Attività GetMetadata](control-flow-get-metadata-activity.md) | Recupera i metadati di qualsiasi dato in Azure Data Factory. 
[Attività Wait](control-flow-wait-activity.md) | Sospende la pipeline per un periodo di tempo specificato.

## <a name="deploy-ssis-packages-to-azure"></a>Distribuire i pacchetti SSIS in Azure 
Per spostare carichi di lavoro SSIS nel cloud, creare una data factory con la versione corrente ed effettuare il provisioning del runtime di integrazione Azure-SSIS, usare Azure-SSIS.

Azure-SSIS Integration Runtime è un cluster completamente gestito di VM (nodi) di Azure dedicate all'esecuzione di pacchetti SSIS nel cloud. Dopo il provisioning di Azure-SSIS Integration Runtime, è possibile usare gli stessi strumenti usati per distribuire pacchetti SSIS in un ambiente SSIS locale. 

Ad esempio, si può usare SQL Server Data Tools o SQL Server Management Studio per distribuire pacchetti SSIS in questo runtime in Azure. Per istruzioni dettagliate, vedere l'esercitazione [Distribuire pacchetti SQL Server Integration Services in Azure](tutorial-create-azure-ssis-runtime-portal.md). 

## <a name="flexible-scheduling"></a>Pianificazione flessibile
Nella versione corrente di Data Factory non è necessario definire pianificazioni della disponibilità per i set di dati. È possibile definire una risorsa trigger che può pianificare le pipeline da un paradigma di utilità di pianificazione basata sul tempo. È anche possibile passare parametri alle pipeline da un trigger per un modello di pianificazione ed esecuzione flessibile. 

Le pipeline non hanno “finestre” di esecuzione temporale nella versione corrente di Data Factory. I concetti di startTime, endTime e isPaused di Data Factory versione 1 non esistono nella versione corrente di Data Factory. Per altre informazioni su come creare e quindi pianificare una pipeline nella versione corrente di Data Factory, vedere [Esecuzione e trigger di pipeline](concepts-pipeline-execution-triggers.md).

## <a name="support-for-more-data-stores"></a>Supporto per più archivi dati
La versione corrente supporta la copia di dati da e verso un maggior numero di archivi dati rispetto alla versione 1. Per un elenco degli archivi dati supportati, vedere gli articoli seguenti:

- [Versione 1 - Archivi dati supportati](v1/data-factory-data-movement-activities.md#supported-data-stores-and-formats)
- [Versione corrente - Archivi dati supportati](copy-activity-overview.md#supported-data-stores-and-formats)

## <a name="support-for-on-demand-spark-cluster"></a>Supporto per il cluster Spark on demand
La versione corrente supporta la creazione di un cluster Azure HDInsight Spark su richiesta. Per creare un cluster Spark su richiesta, specificare Spark come tipo di cluster nella definizione del servizio collegato HDInsight su richiesta. È quindi possibile configurare l'attività Spark nella pipeline per usare il servizio collegato. 

In fase di esecuzione, quando l'attività viene eseguita, il servizio Data Factory crea automaticamente il cluster Spark. Per altre informazioni, vedere gli articoli seguenti:

- [Attività Spark nella versione corrente di Data Factory](transform-data-using-spark.md)
- [Servizio collegato Azure HDInsight su richiesta](compute-linked-services.md#azure-hdinsight-on-demand-linked-service)

## <a name="custom-activities"></a>Attività personalizzate
Nella versione 1, il codice di un'attività DotNet (personalizzata) viene implementato creando un progetto di libreria di classi .NET con una classe che implementa il metodo Execute dell'interfaccia IDotNetActivity. Il codice personalizzato deve quindi essere scritto in .NET Framework 4.5.2 ed essere eseguito in nodi di pool di Azure Batch basati su Windows. 

In un'attività personalizzata della versione corrente non è necessario implementare un'interfaccia .NET. È possibile eseguire direttamente comandi, script e codice personalizzato compilato come eseguibile. 

Per altre informazioni, vedere [Differenza tra l'attività personalizzata in Data Factory e nella versione 1](transform-data-using-dotnet-custom-activity.md#compare-v2-v1).

## <a name="sdks"></a>SDK
 La versione corrente di Data Factory offre un set più ampio di SDK utilizzabili per creare, gestire e monitorare le pipeline.

- **.NET SDK**: .NET SDK è aggiornato nella versione corrente.

- **PowerShell**: i cmdlet di PowerShell sono aggiornati nella versione corrente. Il nome dei cmdlet della versione corrente contiene **DataFactoryV2**, ad esempio Get-AzDataFactoryV2. 

- **Python SDK**: questo SDK è una novità nella versione corrente.

- **API REST**: l'API REST è aggiornata nella versione corrente. 

Gli SDK aggiornati nella versione corrente non sono compatibili con i client versione 1. 

## <a name="authoring-experience"></a>Esperienza di creazione

| &nbsp; | V2 | V1 |
| ------ | -- | -- | 
| Portale di Azure | [Sì](quickstart-create-data-factory-portal.md) | No |
| Azure PowerShell | [Sì](quickstart-create-data-factory-powershell.md) | [Sì](data-factory-build-your-first-pipeline-using-powershell.md) |
| .NET SDK | [Sì](quickstart-create-data-factory-dot-net.md) | [Sì](data-factory-build-your-first-pipeline-using-vs.md) |
| API REST | [Sì](quickstart-create-data-factory-rest-api.md) | [Sì](data-factory-build-your-first-pipeline-using-rest-api.md) |
| Python SDK | [Sì](quickstart-create-data-factory-python.md) | No |
| Modello di Resource Manager | [Sì](quickstart-create-data-factory-resource-manager-template.md) | [Sì](data-factory-build-your-first-pipeline-using-arm.md) | 

## <a name="roles-and-permissions"></a>Ruoli e autorizzazioni

Il ruolo Collaboratore di Data Factory versione 1 può essere usato per creare e gestire le risorse della versione corrente di Data Factory. Per altre informazioni, vedere [Collaboratore Data Factory](../role-based-access-control/built-in-roles.md#data-factory-contributor).

## <a name="monitoring-experience"></a>Esperienza di monitoraggio
Nella versione corrente è anche possibile eseguire il monitoraggio di data factory con [Monitoraggio di Azure](monitor-using-azure-monitor.md). I nuovi cmdlet di PowerShell supportano il monitoraggio delle istanze di [Integration Runtime](monitor-integration-runtime.md). Sia la versione 1 che la versione 2 supportano il monitoraggio visivo tramite un'applicazione di monitoraggio che può essere avviata dal portale di Azure.


## <a name="next-steps"></a>Passaggi successivi
Per informazioni sulla creazione di una data factory, seguire le istruzioni dettagliate disponibili in queste guide introduttive: [PowerShell](quickstart-create-data-factory-powershell.md), [.NET](quickstart-create-data-factory-dot-net.md), [Python](quickstart-create-data-factory-python.md), [API REST](quickstart-create-data-factory-rest-api.md). 
