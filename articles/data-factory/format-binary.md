---
title: Formato binario in Azure Data Factory | Microsoft Docs
description: In questo argomento viene descritto come gestire il formato binario in Azure Data Factory.
author: linda33wj
manager: craigg
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/06/2019
ms.author: jingwang
ms.openlocfilehash: a3b3e4e873497b3b0f726d815a798919ace4f931
ms.sourcegitcommit: bc3a153d79b7e398581d3bcfadbb7403551aa536
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/06/2019
ms.locfileid: "68842543"
---
# <a name="binary-format-in-azure-data-factory"></a>Formato binario in Azure Data Factory

Il formato binario è supportato per i connettori seguenti: [Amazon S3](connector-amazon-simple-storage-service.md), [BLOB di Azure](connector-azure-blob-storage.md), [Azure Data Lake storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake storage Gen2](connector-azure-data-lake-storage.md), [archiviazione file di Azure](connector-azure-file-storage.md), [file System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [http](connector-http.md)e [SFTP](connector-sftp.md).

È possibile usare il set di dati binario nell'attività di [copia](copy-activity-overview.md), l' [attività GetMetadata](control-flow-get-metadata-activity.md)o l' [attività Delete](delete-activity.md). Quando si usa un set di dati binario, ADF non analizza il contenuto del file ma lo considera così com'è. 

>[!NOTE]
>Quando si usa un set di dati binario nell'attività di copia, è possibile copiare solo dal set di dati binario al set di dati binario.

## <a name="dataset-properties"></a>Proprietà del set di dati

Per un elenco completo delle sezioni e delle proprietà disponibili per la definizione dei set di dati, vedere l'articolo [Set di dati](concepts-datasets-linked-services.md). Questa sezione presenta un elenco delle proprietà supportate dal set di dati binario.

| Proprietà         | Descrizione                                                  | Obbligatoria |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La proprietà Type del set di dati deve essere impostata su **Binary**. | Sì      |
| location         | Impostazioni del percorso dei file. Ogni connettore basato su file ha un tipo di percorso e proprietà supportate in `location`. **Per informazioni dettagliate, vedere l'articolo connettore-> sezione Proprietà set di dati**. | Sì      |
| compression | Gruppo di proprietà per configurare la compressione dei file. Configurare questa sezione quando si desidera eseguire la compressione/decompressione durante l'esecuzione dell'attività. | No |
| type | Codec di compressione utilizzato per leggere/scrivere file binari. <br>I valori consentiti sono **bzip2**, **gzip**, deflate, **ZipDeflate**. da usare quando si salva il file. | No       |
| level | Rapporto di compressione. Applicare quando il set di dati viene usato nel sink dell'attività di copia.<br>I valori consentiti sono **ottimali** o più **veloci**.<br>- **Più veloce** l'operazione di compressione deve essere completata il più rapidamente possibile, anche se il file risultante non viene compresso in modo ottimale.<br>- **Ottimale**: l'operazione di compressione deve comprimere il file in modo ottimale, anche se il completamento richiede più tempo. Per maggiori informazioni, vedere l'argomento relativo al [livello di compressione](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) . | No       |

Di seguito è riportato un esempio di set di dati binario nell'archivio BLOB di Azure:

```json
{
    "name": "BinaryDataset",
    "properties": {
        "type": "Binary",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "ZipDeflate"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>Proprietà dell'attività di copia

Per un elenco completo delle sezioni e delle proprietà disponibili per la definizione delle attività, vedere l'articolo sulle [pipeline](concepts-pipelines-activities.md). Questa sezione presenta un elenco delle proprietà supportate dall'origine e dal sink binario.

>[!NOTE]
>Quando si usa un set di dati binario nell'attività di copia, è possibile copiare solo dal set di dati binario al set di dati binario.

### <a name="binary-as-source"></a>Binario come origine

Nella sezione ***\*origine\**** dell'attività di copia sono supportate le proprietà seguenti.

| Proprietà      | Descrizione                                                  | Obbligatoria |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La proprietà Type dell'origine dell'attività di copia deve essere impostata su **BinarySource**. | Sì      |
| storeSettings | Un gruppo di proprietà su come leggere i dati da un archivio dati. Ogni connettore basato su file ha le proprie impostazioni di lettura supportate `storeSettings`in. **Per informazioni dettagliate, vedere l'articolo connettore > sezione proprietà dell'attività di copia**. | No       |

### <a name="binary-as-sink"></a>Binario come sink

Nella sezione ***\*sink\**** dell'attività di copia sono supportate le proprietà seguenti.

| Proprietà      | Descrizione                                                  | Obbligatoria |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La proprietà Type dell'origine dell'attività di copia deve essere impostata su **BinarySink**. | Yes      |
| storeSettings | Gruppo di proprietà su come scrivere dati in un archivio dati. Ogni connettore basato su file ha le proprie impostazioni di scrittura supportate `storeSettings`in. **Per informazioni dettagliate, vedere l'articolo connettore > sezione proprietà dell'attività di copia**. | No       |

## <a name="next-steps"></a>Passaggi successivi

- [Panoramica dell'attività di copia](copy-activity-overview.md)
- [Attività GetMetadata](control-flow-get-metadata-activity.md)
- [Elimina attività](delete-activity.md)
