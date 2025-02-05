---
title: Aggiungere la tolleranza di errore all'attività di copia di Azure Data Factory ignorando le righe incompatibili | Microsoft Docs
description: Informazioni su come aggiungere la tolleranza di errore all'attività di copia di Azure Data Factory ignorando le righe incompatibili
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 03/27/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 3a255b21e8bfd7d78954603e9aa6e5ca39cee95b
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60566084"
---
# <a name="add-fault-tolerance-in-copy-activity-by-skipping-incompatible-rows"></a>Aggiungere la tolleranza di errore all'attività di copia ignorando le righe incompatibili

> [!div class="op_single_selector" title1="Selezionare la versione del servizio Data Factory in uso:"]
> * [Versione 1](data-factory-copy-activity-fault-tolerance.md)
> * [Versione 2 (corrente)](../copy-activity-fault-tolerance.md)

> [!NOTE]
> Le informazioni di questo articolo sono valide per la versione 1 di Data Factory. Se si usa la versione corrente del servizio Data Factory, vedere le informazioni sulla [tolleranza di errore nell'attività di copia di Data Factory](../copy-activity-fault-tolerance.md).

L'[attività di copia](data-factory-data-movement-activities.md) di Azure Data Factory offre due modi per gestire le righe incompatibili durante la copia di dati tra gli archivi dati di origine e sink:

- È possibile interrompere l'attività di copia quando vengono rilevati dati incompatibili (comportamento predefinito).
- È possibile continuare a copiare tutti i dati aggiungendo la tolleranza di errore e ignorando le righe di dati incompatibili. È anche possibile registrare le righe incompatibili nell'archivio BLOB di Azure. È quindi possibile esaminare il log per conoscere la causa dell'errore, correggere i dati nell'origine dati e ripetere l'attività di copia.

## <a name="supported-scenarios"></a>Scenari supportati
L'attività di copia supporta tre scenari per rilevare, ignorare e registrare i dati incompatibili:

- **Incompatibilità tra il tipo di dati di origine e il tipo nativo sink**

    Ad esempio: si vogliono copiare dati da un file CSV nell'archiviazione BLOB a un database SQL con una definizione di schema che contiene tre colonne di tipo **INT**. Le righe del file CSV contenenti dati numerici, ad esempio `123,456,789`, vengono copiate nell'archivio sink. Tuttavia, le righe che contengono valori non numerici, ad esempio `123,456,abc`, vengono rilevate come incompatibili e vengono ignorate.

- **Mancata corrispondenza nel numero di colonne tra l'origine e il sink**

    Ad esempio: si vogliono copiare dati da un file CSV nell'archivio BLOB a un database SQL con una definizione di schema che contiene sei colonne. Le righe del file CSV che contengono sei colonne vengono copiate nell'archivio sink. Le righe del file CSV che contengono più o meno di sei colonne vengono rilevate come incompatibili e vengono ignorate.

- **Violazione della chiave primaria per la scrittura in SQL Server/database SQL di Azure/Azure Cosmos DB**

    Ad esempio:  si vogliono copiare dati da un'istanza di SQL Server a un database SQL. Il database SQL del sink contiene la definizione di una chiave primaria, che invece manca nell'istanza di SQL Server di origine. Non è possibile copiare nel sink le righe duplicate presenti nell'origine. L'attività di copia copierà nel sink solo la prima riga dei dati di origine. Le righe di origine successive che contengono il valore della chiave primaria duplicato vengono rilevate come incompatibili e vengono ignorate.

>[!NOTE]
>Questa funzionalità non si applica quando l'attività di copia è configurata per chiamare un meccanismo di caricamento di dati esterni fra cui [Azure SQL Data Warehouse PolyBase](data-factory-azure-sql-data-warehouse-connector.md#use-polybase-to-load-data-into-azure-sql-data-warehouse) o [Amazon Redshift Unload](data-factory-amazon-redshift-connector.md#use-unload-to-copy-data-from-amazon-redshift). Per caricare dati in SQL Data Warehouse mediante PolyBase, usare il supporto nativo della tolleranza di errore di PolyBase specificando "[polyBaseSettings](data-factory-azure-sql-data-warehouse-connector.md#sqldwsink)" nell'attività di copia.

## <a name="configuration"></a>Configurazione
L'esempio seguente offre la definizione JSON per specificare di ignorare le righe incompatibili nell'attività di copia:

```json
"typeProperties": {
    "source": {
        "type": "BlobSource"
    },
    "sink": {
        "type": "SqlSink",
    },
    "enableSkipIncompatibleRow": true,
    "redirectIncompatibleRowSettings": {
        "linkedServiceName": "BlobStorage",
        "path": "redirectcontainer/erroroutput"
    }
}
```

| Proprietà | DESCRIZIONE | Valori consentiti | Obbligatorio |
| --- | --- | --- | --- |
| **enableSkipIncompatibleRow** | Specificare se ignorare o meno le righe incompatibili durante la copia. | True<br/>False (impostazione predefinita) | No |
| **redirectIncompatibleRowSettings** | Un gruppo di proprietà che può essere specificato quando si vuole registrare le righe incompatibili. | &nbsp; | No |
| **linkedServiceName** | Servizio collegato di Archiviazione di Azure con cui archiviare il log che contiene le righe ignorate. | Nome di un servizio collegato [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service) o [AzureStorageSas](data-factory-azure-blob-connector.md#azure-storage-sas-linked-service) che fa riferimento all'istanza di archiviazione da usare per archiviare il file di log. | No |
| **path** | Percorso del file di log che contiene le righe ignorate. | Specificare il percorso dell'archivio BLOB da usare per registrare i dati incompatibili. Se non si specifica un percorso, il servizio crea automaticamente un contenitore. | No |

## <a name="monitoring"></a>Monitoraggio
Al termine dell'esecuzione dell'attività di copia, è possibile visualizzare il numero di righe ignorate nella sezione di monitoraggio:

![Monitorare le righe incompatibili ignorate](./media/data-factory-copy-activity-fault-tolerance/skip-incompatible-rows-monitoring.png)

Se si configura la registrazione delle righe incompatibili, vedere il file di log al percorso seguente: `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/[copy-activity-run-id]/[auto-generated-GUID].csv` Il file di log contiene le righe ignorate e la causa radice dell'incompatibilità.

Nel file vengono registrati sia i dati originali che l'errore corrispondente. Di seguito è riportato un esempio del contenuto del file di log:
```
data1, data2, data3, UserErrorInvalidDataValue,Column 'Prop_2' contains an invalid value 'data3'. Cannot convert 'data3' to type 'DateTime'.,
data4, data5, data6, Violation of PRIMARY KEY constraint 'PK_tblintstrdatetimewithpk'. Cannot insert duplicate key in object 'dbo.tblintstrdatetimewithpk'. The duplicate key value is (data4).
```

## <a name="next-steps"></a>Passaggi successivi
Per altre informazioni sull'attività di copia di Azure Data Factory, vedere [Spostare dati con l'attività di copia](data-factory-data-movement-activities.md).
