---
title: Dati da SQL Server a SQL Azure con Azure Data Factory - Processo di data science per i team (TDSP)
description: Impostare una pipeline di ADF che comprenda due attività di migrazione di dati che insieme spostino i dati giornalmente tra database locali e nel cloud.
services: machine-learning
author: marktab
manager: cgronlun
editor: cgronlun
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 11/04/2017
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 59f8b8b253fc914e5723a9c41475ec78bc3f376e
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "61429349"
---
# <a name="move-data-from-an-on-premises-sql-server-to-sql-azure-with-azure-data-factory"></a>Spostare i dati da SQL Server locale a SQL Azure con Azure Data Factory

Questo articolo descrive come spostare i dati da un database di SQL Server locale a un database SQL di Azure tramite Archiviazione BLOB di Azure usando Azure Data Factory (ADF).

Per un tabella che riepiloga le varie opzioni per lo spostamento dei dati in un database SQL di Azure, vedere [Spostare i dati a un database SQL Azure per Azure Machine Learning](move-sql-azure.md).

## <a name="intro"></a>Introduzione: Che cos'è Azure Data Factory e quando deve essere usato per la migrazione dei dati?
Il Data factory è un servizio di integrazione delle informazioni basato sul cloud che permette di automatizzare lo spostamento e la trasformazione dei dati. Il concetto chiave nel modello ADF è pipeline. Una pipeline è un raggruppamento logico di attività, ognuna delle quali definisce le azioni da eseguire sui dati contenuti nel set di dati. I servizi collegati vengono utilizzati per definire le informazioni necessarie affinché il servizio Data factory si connetta a risorse dati esterne.

Con ADF, i servizi di elaborazione dei dati esistenti possono essere composti in pipeline di dati, altamente disponibili e gestiti nel cloud. Queste pipeline di dati possono essere pianificate per inserire, preparare, trasformare, analizzare e pubblicare i dati e l’ADF gestisce e organizza i dati complessi e le dipendenze di elaborazione. Le soluzioni possono essere compilate e distribuite nel cloud, collegando un numero crescente di origini dati locali e cloud.

Considerare l'uso di ADF:

* quando i dati sono soggetti a migrazione continua in uno scenario ibrido che accede alle risorse locali e cloud
* quando i dati sono soggetti a transazioni, devono essere modificati o si vedono aggiungere una logica di business quando vengono migrati.

L’ADF consente la pianificazione e il monitoraggio dei processi utilizzando semplici script JSON che gestiscono lo spostamento dei dati su base periodica. ADF dispone anche di altre funzionalità quali il supporto di operazioni complesse. Per ulteriori informazioni sul file ADF, vedere la documentazione di [Data factory di Azure (ADF)](https://azure.microsoft.com/services/data-factory/).

## <a name="scenario"></a>Scenario
Si configura una pipeline ADF che compone due attività di migrazione dei dati. Insieme, queste attività spostano i dati giornalmente tra un database SQL locale e un database SQL di Azure nel cloud. Le due attività sono:

* Copiare dati da un database di SQL Server locale in un account dell'archiviazione BLOB di Azure
* Copiare i dati dall'account di archiviazione BLOB di Azure a un database SQL di Azure.

> [!NOTE]
> I passaggi illustrati di seguito sono stati adattati dall'esercitazione più dettagliata fornita dal team di Azure Data Factory: [Copiare dati da un database di SQL Server locale in archiviazione BLOB di Azure](https://docs.microsoft.com/azure/data-factory/tutorial-hybrid-copy-portal/) Se appropriato, vengono indicati Riferimenti alle sezioni pertinenti di quell'argomento.
>
>

## <a name="prereqs"></a>Prerequisiti
Il tutorial presuppone:

* Una **sottoscrizione di Azure**. Se non si ha una sottoscrizione, è possibile iscriversi per provare una [versione di valutazione gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Un **account di archiviazione Azure**. In questa esercitazione si userà un account di archiviazione di Azure per archiviare i dati. Se non si dispone di un account di archiviazione di Azure, vedere l'articolo [Creare un account di archiviazione di Azure](../../storage/common/storage-quickstart-create-account.md) . Dopo avere creato l'account di archiviazione, è necessario ottenere la chiave dell'account usata per accedere alla risorsa di archiviazione. Vedere la sezione [Gestire le chiavi di accesso alle risorse di archiviazione](../../storage/common/storage-account-manage.md#access-keys).
* Accesso a un **database SQL di Azure**. Se è necessario configurare un Database SQL di Azure, l'argomento [Introduzione a Database SQL di Microsoft Azure](../../sql-database/sql-database-get-started.md) fornisce informazioni su come effettuare il provisioning di una nuova istanza di un Database SQL di Azure.
* Installazione e configurazione di **Azure PowerShell** in locale. Per istruzioni, vedere [Come installare e configurare Azure PowerShell](/powershell/azure/overview).

> [!NOTE]
> In questa procedura viene utilizzato il [portale di Azure](https://portal.azure.com/).
>
>

## <a name="upload-data"></a> Caricare i dati in SQL Server locale
Utilizziamo il [set di dati NYC Taxi](https://chriswhong.com/open-data/foil_nyc_taxi/) per illustrare il processo di migrazione. Il set di dati NYC Taxi è disponibile, come indicato nel post, sull'archiviazione BLOB di Azure [Dati NYC Taxi](https://www.andresmh.com/nyctaxitrips/). I dati dispongono di due file, il file trip_data.csv che contiene i dettagli relativi alle corse e il file trip_far.csv che contiene i dettagli della tariffa pagata per ogni corsa. Un esempio e una descrizione di questi file sono inclusi in [Descrizione del set di dati relativo alle corse dei taxi di NYC](sql-walkthrough.md#dataset).

È possibile adattare le procedure riportate di seguito a un set di dati personalizzati o seguire i passaggi come descritto utilizzando il set di dati NYC Taxi. Per caricare il set di dati NYC Taxi nel database di SQL Server locale, seguire la procedura descritta in [Importazione in blocco dei dati nel database SQL Server](sql-walkthrough.md#dbload). Queste istruzioni sono per SQL Server in una macchina virtuale di Azure, ma la procedura per il caricamento in SQL Server locale è la stessa.

## <a name="create-adf"></a> Creare un data factory di Azure
Le istruzioni per la creazione di una nuova data factory di Azure e un gruppo di risorse nel [portale di Azure](https://portal.azure.com/) sono disponibili in [Creazione di un'istanza di Data factory di Azure](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-data-factory). Denominare la nuova istanza ADF *adfdsp*e assegnare il nome *adfdsprg* al gruppo di risorse creato.

## <a name="install-and-configure-azure-data-factory-integration-runtime"></a>Installare e configurare Integration Runtime di Azure Data Factory
Integration Runtime è un'infrastruttura di integrazione dati gestita dal cliente, usata da Azure Data Factory per fornire funzionalità di integrazione dati in ambienti di rete differenti. In precedenza Questo runtime era chiamato "gateway di gestione dati".

Per configurare, [seguire le istruzioni per la creazione di una pipeline](https://docs.microsoft.com/azure/data-factory/tutorial-hybrid-copy-portal#create-a-pipeline)

## <a name="adflinkedservices"></a>Creare servizi collegati per connettersi alle risorse di dati
I servizi collegati definiscono le informazioni necessarie affinché il servizio data factory si connetta a risorse dati. Sono disponibili tre risorse in questo scenario per il quale sono necessari servizi collegati:

1. Server SQL locale
2. Archivio BLOB di Azure
3. Database SQL di Azure

In [Creazione di servizi collegati](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline)viene fornita la procedura dettagliata per la creazione di servizi collegati.


## <a name="adf-tables"></a>Definire e creare tabelle per specificare la modalità di accesso al set di dati
Creare tabelle che specificano la struttura, la posizione e la disponibilità dei set di dati con le seguenti procedure basate su script. I file JSON vengono utilizzati per definire le tabelle. Per ulteriori informazioni sulla struttura di questi file, vedere [Set di dati](../../data-factory/concepts-datasets-linked-services.md).

> [!NOTE]
> È necessario eseguire il cmdlet `Add-AzureAccount` prima di eseguire il cmdlet [New-AzureDataFactoryTable](https://msdn.microsoft.com/library/azure/dn835096.aspx) per verificare che sia selezionata la sottoscrizione di Azure giusta per l'esecuzione del comando. Per la documentazione di questo cmdlet, vedere [Add-AzureAccount](/powershell/module/servicemanagement/azure/add-azureaccount?view=azuresmps-3.7.0).
>
>

Le definizioni basate su JSON nelle tabelle utilizzano i nomi seguenti:

* il **nome della tabella** nel server SQL locale è *nyctaxi_data*
* il **nome del contenitore** nell'account di archiviazione Blob di Azure è *nomecontenitore*

Per questa pipeline ADF sono necessarie tre definizioni di tabella:

1. [Tabella SQL locale](#adf-table-onprem-sql)
2. [Tabella BLOB](#adf-table-blob-store)
3. [Tabella SQL Azure](#adf-table-azure-sql)

> [!NOTE]
> Queste procedure utilizzano Azure PowerShell per definire e creare le attività del file ADF. Tuttavia, queste attività possono inoltre essere eseguite tramite il portale di Azure. Per informazioni dettagliate, vedere [Creare set di dati](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).
>
>

### <a name="adf-table-onprem-sql"></a>Tabella SQL locale
La definizione della tabella per SQL Server locale viene specificata nel file JSON seguente:

```json
{
    "name": "OnPremSQLTable",
    "properties":
    {
        "location":
        {
            "type": "OnPremisesSqlServerTableLocation",
            "tableName": "nyctaxi_data",
            "linkedServiceName": "adfonpremsql"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1,
            "waitOnExternal":
            {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Qui non sono inclusi i nomi di colonna. È possibile sottoselezionare i nomi delle colonne includendoli qui (per ulteriori informazioni consultare l'argomento [Documentazione ADF](../../data-factory/copy-activity-overview.md) ).

Copiare la definizione JSON della tabella in un file denominata *onpremtabledef.json* e salvarlo in una posizione nota (generalmente *C:\temp\onpremtabledef.json*). Creare la tabella nel file ADF con il seguente cmdlet Azure PowerShell:

    New-AzureDataFactoryTable -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp –File C:\temp\onpremtabledef.json


### <a name="adf-table-blob-store"></a>Tabella BLOB
La definizione della tabella per il percorso del BLOB di output è la seguente. I dati inseriti vengono così mappati dal server locale al BLOB di Azure:

```json
{
    "name": "OutputBlobTable",
    "properties":
    {
        "location":
        {
            "type": "AzureBlobLocation",
            "folderPath": "containername",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": "\t"
            },
            "linkedServiceName": "adfds"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Copiare la definizione JSON della tabella in un file denominata *bloboutputtabledef.json* e salvarlo in una posizione nota (generalmente *C:\temp\bloboutputtabledef.json*). Creare la tabella nel file ADF con il seguente cmdlet Azure PowerShell:

    New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\bloboutputtabledef.json

### <a name="adf-table-azure-sql"></a>Tabella SQL Azure
La definizione della tabella per l’output SQL Azure è la seguente (questo schema associa i dati provenienti dal blob):

```json
{
    "name": "OutputSQLAzureTable",
    "properties":
    {
        "structure":
        [
            { "name": "column1", "type": "String"},
            { "name": "column2", "type": "String"}
        ],
        "location":
        {
            "type": "AzureSqlTableLocation",
            "tableName": "your_db_name",
            "linkedServiceName": "adfdssqlazure_linked_servicename"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Copiare la definizione JSON della tabella in un file denominata *AzureSqlTable.json* e salvarlo in una posizione nota (generalmente *C:\temp\AzureSqlTable.json*). Creare la tabella nel file ADF con il seguente cmdlet Azure PowerShell:

    New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\AzureSqlTable.json


## <a name="adf-pipeline"></a>Definire e creare la pipeline
Specificare le attività che appartengono alla pipeline e creare la pipeline con le seguenti procedure basate su script. Un file JSON viene utilizzato per definire le proprietà della pipeline.

* Lo script presuppone che il **nome della pipeline** sia *AMLDSProcessPipeline*.
* Si noti inoltre che abbiamo impostato la periodicità della pipeline per l’esecuzione su base giornaliera e utilizza il tempo di esecuzione predefinito per il processo (12 am UTC).

> [!NOTE]
> Le procedure seguenti utilizzano Azure PowerShell per definire e creare la pipeline del file ADF. Tuttavia, questa attività può inoltre essere eseguita tramite il portale di Azure. Per informazioni dettagliate, vedere [Creare una pipeline](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).
>
>

Se si utilizzano le definizioni di tabella fornite in precedenza, la definizione della pipeline per il file ADF viene specificata come segue:

```json
{
    "name": "AMLDSProcessPipeline",
    "properties":
    {
        "description" : "This pipeline has one Copy activity that copies data from an on-premises SQL to Azure blob",
        "activities":
        [
            {
                "name": "CopyFromSQLtoBlob",
                "description": "Copy data from on-premises SQL server to blob",
                "type": "CopyActivity",
                "inputs": [ {"name": "OnPremSQLTable"} ],
                "outputs": [ {"name": "OutputBlobTable"} ],
                "transformation":
                {
                    "source":
                    {
                        "type": "SqlSource",
                        "sqlReaderQuery": "select * from nyctaxi_data"
                    },
                    "sink":
                    {
                        "type": "BlobSink"
                    }
                },
                "Policy":
                {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            },
            {
                "name": "CopyFromBlobtoSQLAzure",
                "description": "Push data to Sql Azure",
                "type": "CopyActivity",
                "inputs": [ {"name": "OutputBlobTable"} ],
                "outputs": [ {"name": "OutputSQLAzureTable"} ],
                "transformation":
                {
                    "source":
                    {
                        "type": "BlobSource"
                    },
                    "sink":
                    {
                        "type": "SqlSink",
                        "WriteBatchTimeout": "00:5:00",
                    }
                },
                "Policy":
                {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 2,
                    "timeout": "02:00:00"
                }
            }
        ]
    }
}
```

Copiare la definizione JSON della pipeline in un file denominato *pipelinedef.json* e salvarlo in una posizione nota (generalmente *C:\temp\pipelinedef.json*). Creare la pipeline ADF con il seguente cmdlet Azure PowerShell:

    New-AzureDataFactoryPipeline  -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\pipelinedef.json


## <a name="adf-pipeline-start"></a>Avviare la pipeline
La pipeline è avviabile con il comando seguente:

    Set-AzureDataFactoryPipelineActivePeriod -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp -StartDateTime startdateZ –EndDateTime enddateZ –Name AMLDSProcessPipeline

I valori del parametro *startdate* ed *enddate* devono essere sostituiti con le date effettive tra le quali si desidera eseguire la pipeline.

Una volta eseguita la pipeline, si dovrebbe poter visualizzare i dati visualizzati nel contenitore selezionato per il blob, un file al giorno.

Si noti che non abbiamo utilizzato la funzionalità fornita da ADF per dirigere i dati in modo incrementale. Per ulteriori informazioni su come eseguire questa e altre funzionalità fornite da ADF, vedere la [documentazione ADF](https://azure.microsoft.com/services/data-factory/).
