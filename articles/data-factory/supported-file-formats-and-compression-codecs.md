---
title: Formati di file supportati in Azure Data Factory | Microsoft Docs
description: Questo argomento descrive i formati di file e i codici di compressione supportati dai connettori basati su file in Azure Data Factory.
author: linda33wj
manager: craigg
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: jingwang
ms.openlocfilehash: 2c8983b5d6a44834d0c9659877c857fd73805ce6
ms.sourcegitcommit: fa4852cca8644b14ce935674861363613cf4bfdf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/09/2019
ms.locfileid: "70812316"
---
# <a name="supported-file-formats-and-compression-codecs-in-azure-data-factory"></a>Formati di file e codec di compressione supportati in Azure Data Factory

*Questo articolo si applica ai connettori seguenti: [Amazon S3](connector-amazon-simple-storage-service.md), [BLOB di Azure](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Archiviazione file di Azure](connector-azure-file-storage.md), [File system](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) e [SFTP](connector-sftp.md).*

Per **copiare i file così come sono** tra archivi basati su file (copia binaria), è possibile ignorare la sezione del formato nelle definizioni dei set di dati di input e di output. Se si intende **analizzare o generare file con un formato specifico**, Azure Data Factory supporta i tipi di formato file seguenti:

* [Formato testo](#text-format)
* [Formato JSON](#json-format)
* [Formato Parquet](#parquet-format)
* [Formato ORC](#orc-format)
* [Formato Avro](#avro-format)
* [Formato binario](#binary-format)

> [!TIP]
> Informazioni su come l'attività di copia esegue il mapping dei dati di origine al sink dal [mapping dello schema nell'attività di copia](copy-activity-schema-and-type-mapping.md).

## <a name="text-format"></a>Formato testo

>[!NOTE]
>Data Factory introdotto un nuovo set di dati in formato testo delimitato, vedere l'articolo [formato testo delimitato](format-delimited-text.md) con i dettagli. Le configurazioni seguenti nel set di dati dell'archivio dati basato su file sono ancora supportate così come sono per le Compabitility precedenti. Si consiglia di utilizzare il nuovo modello in futuro.

Se si vuole leggere da un file di testo o scrivere in un file di testo, impostare la proprietà `type` nella sezione `format` del set di dati **TextFormat**. È anche possibile specificare le proprietà **facoltative** seguenti nella sezione `format`. Vedere la sezione [Esempio di TextFormat](#textformat-example) sulla configurazione.

| Proprietà | DESCRIZIONE | Valori consentiti | Obbligatoria |
| --- | --- | --- | --- |
| columnDelimiter |Il carattere usato per separare le colonne in un file. È possibile usare un carattere non stampabile raro che potrebbe non esistere nei dati. Ad esempio, specificare "\u0001", che rappresenta l'inizio intestazione (SOH). |È consentito un solo carattere. Il valore **predefinito** è la **virgola (",")** . <br/><br/>Per usare un carattere Unicode, vedere i [caratteri Unicode](https://en.wikipedia.org/wiki/List_of_Unicode_characters) per ottenere il codice corrispondente. |No |
| rowDelimiter |Il carattere usato per separare le righe in un file. |È consentito un solo carattere. Sono consentiti i seguenti valori **predefiniti** in lettura: **["\r\n", "\r", "\n"]** e **"\r\n"** in scrittura. |No |
| escapeChar |Carattere speciale usato per eseguire l'escape di un delimitatore di colonna nel contenuto del file di input. <br/><br/>Per una tabella, è possibile specificare sia escapeChar che quoteChar. |È consentito un solo carattere. Nessun valore predefinito. <br/><br/>Esempio: se è presente una virgola (,) come delimitatore di colonna, ma si vuole usare il carattere virgola nel testo (ad esempio: "Hello, world"), è possibile definire $ come carattere di escape e usare la stringa "Hello$, world" nell'origine. |No |
| quoteChar |Carattere usato per delimitare tra virgolette un valore stringa. I delimitatori di colonne e righe tra virgolette sono considerati parte del valore stringa. Questa proprietà è applicabile sia ai set di dati di input che a quelli di output.<br/><br/>Per una tabella, è possibile specificare sia escapeChar che quoteChar. |È consentito un solo carattere. Nessun valore predefinito. <br/><br/>Ad esempio, se è presente una virgola (",") come delimitatore di colonna, ma si desidera inserire un carattere virgola nel testo (ad esempio: <Hello, world>), è possibile definire " (virgolette doppie) come carattere di virgolette e usare la stringa "Hello, world" nell'origine. |No |
| nullValue |Uno o più caratteri usati per rappresentare un valore null. |Uno o più caratteri. I valori **predefiniti** sono **"\N" e "NULL"** in lettura e **"\N"** in scrittura. |No |
| encodingName |Specificare il nome della codifica. |Un nome di codifica valido. Vedere [Proprietà Encoding.EncodingName](https://msdn.microsoft.com/library/system.text.encoding.aspx). Esempio: windows-1250 o shift_jis. Il valore **predefinito** è **UTF-8**. |No |
| firstRowAsHeader |Specifica se considerare la prima riga come intestazione. In un set di dati di input Data factory legge la prima riga come intestazione. In un set di dati di output Data factory scrive la prima riga come intestazione. <br/><br/>Vedere [Scenari per l'uso di `firstRowAsHeader` e `skipLineCount`](#scenarios-for-using-firstrowasheader-and-skiplinecount) per gli scenari di esempio. |True<br/><b>False (impostazione predefinita)</b> |No |
| skipLineCount |Indica il numero di righe **non vuote** da ignorare durante la lettura di dati da file di input. Se sono specificati sia skipLineCount che firstRowAsHeader, le righe vengono ignorate e le informazioni di intestazione vengono lette dal file di input. <br/><br/>Vedere [Scenari per l'uso di `firstRowAsHeader` e `skipLineCount`](#scenarios-for-using-firstrowasheader-and-skiplinecount) per gli scenari di esempio. |Integer |No |
| treatEmptyAsNull |Specifica se considerare una stringa vuota o null come valore null durante la lettura di dati da un file di input. |**True (impostazione predefinita)**<br/>False |No |

### <a name="textformat-example"></a>Esempio di TextFormat

Nella definizione JSON seguente per un set di dati sono specificate alcune proprietà facoltative.

```json
"typeProperties":
{
    "folderPath": "mycontainer/myfolder",
    "fileName": "myblobname",
    "format":
    {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";",
        "quoteChar": "\"",
        "NullValue": "NaN",
        "firstRowAsHeader": true,
        "skipLineCount": 0,
        "treatEmptyAsNull": true
    }
},
```

Per usare `escapeChar` invece di `quoteChar`, sostituire la riga con `quoteChar` con l'elemento escapeChar seguente:

```json
"escapeChar": "$",
```

### <a name="scenarios-for-using-firstrowasheader-and-skiplinecount"></a>Scenari di utilizzo di firstRowAsHeader e skipLineCount

* Si vuole copiare da un'origine non basata su file in un file di testo e aggiungere una riga di intestazione contenente i metadati dello schema (ad esempio: schema SQL). Per questo scenario specificare `firstRowAsHeader` come true nel set di dati di output.
* Si desidera copiare da un file di testo contenente una riga di intestazione a un sink non basato su file ed eliminare tale riga. Specificare `firstRowAsHeader` come true nel set di dati di input.
* Si desidera copiare da un file di testo e ignorare alcune righe all'inizio che non contengono né dati né un'intestazione. Specificare `skipLineCount` per indicare il numero di righe da ignorare. Se il resto del file contiene una riga di intestazione, è anche possibile specificare `firstRowAsHeader`. Se sono specificati sia `skipLineCount` che `firstRowAsHeader`, le righe vengono ignorate e le informazioni di intestazione vengono lette dal file di input.

## <a name="json-format"></a>Formato JSON

>[!NOTE]
>Data Factory introdotto un nuovo set di dati in formato JSON, vedere l'articolo [JSON](format-json.md) con i dettagli. Le configurazioni seguenti nel set di dati dell'archivio dati basato su file sono ancora supportate così come sono per le Compabitility precedenti. Si consiglia di utilizzare il nuovo modello in futuro.

Per **importare/esportare un file JSON senza modifiche in/da Azure Cosmos DB**, vedere la sezione relativa all'importazione/esportazione di documenti JSON nell'articolo [Move data to/from Azure Cosmos DB (Spostare dati da e verso Azure Cosmos DB)](connector-azure-cosmos-db.md).

Per analizzare i file JSON o scrivere i dati in formato JSON, impostare la proprietà `type` nella sezione `format` su **JsonFormat**. È anche possibile specificare le proprietà **facoltative** seguenti nella sezione `format`. Vedere la sezione [Esempio JsonFormat](#jsonformat-example) sulla configurazione.

| Proprietà | Descrizione | Obbligatoria |
| --- | --- | --- |
| filePattern |Indicare il modello dei dati archiviati in ogni file JSON. I valori consentiti sono: **setOfObjects** e **arrayOfObjects**. Il valore **predefinito** è **setOfObjects**. Vedere la sezione [Modelli di file JSON](#json-file-patterns) per i dettagli su questi modelli. |No |
| jsonNodeReference | Per eseguire l'iterazione dei dati ed estrarli dagli oggetti presenti nel campo di una matrice con lo stesso modello, specificare il percorso JSON di tale matrice. Questa proprietà è supportata solo quando si copiano dati **da** file JSON. | No |
| jsonPathDefinition | Specificare l'espressione del percorso JSON per ogni mapping colonne con un nome di colonna personalizzato. Iniziare con una lettera minuscola. Questa proprietà è supportata solo quando si copiano dati **da** file JSON ed è possibile estrarre dati dall'oggetto o dalla matrice. <br/><br/> Per i campi sotto l'oggetto radice, iniziare con la radice $. Per i campi nella matrice scelta dalla proprietà `jsonNodeReference`, iniziare dall'elemento matrice. Vedere la sezione [Esempio JsonFormat](#jsonformat-example) sulla configurazione. | No |
| encodingName |Specificare il nome della codifica. Per l'elenco dei nomi di codifica validi, vedere: Proprietà [Encoding.EncodingName](https://msdn.microsoft.com/library/system.text.encoding.aspx). Ad esempio: windows-1250 o shift_jis. Il valore **predefinito** è: **UTF-8**. |No |
| nestingSeparator |Carattere utilizzato per separare i livelli di nidificazione. Il valore predefinito è "." (punto). |No |

>[!NOTE]
>Per il caso di applicazione incrociata di dati in una matrice in più righe (caso 1-> esempio 2 in [JsonFormat esempi](#jsonformat-example)), è possibile scegliere di espandere solo una `jsonNodeReference`singola matrice usando la proprietà.

### <a name="json-file-patterns"></a>Modelli di file JSON

L'attività di copia può eseguire l'analisi dei seguenti modelli di file JSON:

- **Tipo I: setOfObjects**

    Ogni file contiene un solo oggetto o più oggetti con delimitatori di riga/concatenati. Quando si sceglie questa opzione in un set di dati di output, l'attività di copia produce un singolo file JSON con un oggetto per riga (delimitato da riga).

    * **Esempio di JSON a oggetto singolo**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        ```

    * **Esempio di JSON con delimitatori di riga**

        ```json
        {"time":"2015-04-29T07:12:20.9100000Z","callingimsi":"466920403025604","callingnum1":"678948008","callingnum2":"567834760","switch1":"China","switch2":"Germany"}
        {"time":"2015-04-29T07:13:21.0220000Z","callingimsi":"466922202613463","callingnum1":"123436380","callingnum2":"789037573","switch1":"US","switch2":"UK"}
        {"time":"2015-04-29T07:13:21.4370000Z","callingimsi":"466923101048691","callingnum1":"678901578","callingnum2":"345626404","switch1":"Germany","switch2":"UK"}
        ```

    * **Esempio di JSON concatenati**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        }
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
        ```

- **Tipo II: arrayOfObjects**

    Ogni file contiene una matrice di oggetti.

    ```json
    [
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        },
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        },
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
    ]
    ```

### <a name="jsonformat-example"></a>Esempio JsonFormat

**Caso 1: Copia di dati dai file JSON**

**Esempio 1: Estrarre i dati dall'oggetto e dalla matrice**

In questo esempio si prevede che un oggetto JSON radice esegua il mapping a un singolo record in un risultato tabulare. Se si dispone di un file JSON con il contenuto seguente:

```json
{
    "id": "ed0e4960-d9c5-11e6-85dc-d7996816aad3",
    "context": {
        "device": {
            "type": "PC"
        },
        "custom": {
            "dimensions": [
                {
                    "TargetResourceType": "Microsoft.Compute/virtualMachines"
                },
                {
                    "ResourceManagementProcessRunId": "827f8aaa-ab72-437c-ba48-d8917a7336a3"
                },
                {
                    "OccurrenceTime": "1/13/2017 11:24:37 AM"
                }
            ]
        }
    }
}
```

e lo si vuole copiare in una tabella SQL di Azure nel formato seguente, estraendo i dati sia dagli oggetti che dalla matrice:

| id | deviceType | targetResourceType | resourceManagementProcessRunId | occurrenceTime |
| --- | --- | --- | --- | --- |
| ed0e4960-d9c5-11e6-85dc-d7996816aad3 | PC | Microsoft.Compute/virtualMachines | 827f8aaa-ab72-437c-ba48-d8917a7336a3 | 1/13/2017 11:24:37 AM |

Il set di dati di input con il tipo **JsonFormat** è definito come segue (definizione parziale che include solo le parti pertinenti). Più in particolare:

- La sezione `structure` definisce i nomi di colonna personalizzati e il tipo di dati corrispondente durante la conversione in dati tabulari. Questa sezione è **facoltativa** a meno che non sia necessario eseguire il mapping colonne. Per altre informazioni, vedere [Eseguire il mapping delle colonne del set di dati di origine alle colonne del set di dati di destinazione](copy-activity-schema-and-type-mapping.md).
- `jsonPathDefinition` specifica il percorso JSON per ogni colonna indicante da dove estrarre i dati. Per copiare i dati dalla matrice, è possibile usare `array[x].property` per estrarre il valore della proprietà specificata dall'oggetto `xth`, oppure è possibile usare `array[*].property` per trovare il valore in qualsiasi oggetto contenente tale proprietà.

```json
"properties": {
    "structure": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "deviceType",
            "type": "String"
        },
        {
            "name": "targetResourceType",
            "type": "String"
        },
        {
            "name": "resourceManagementProcessRunId",
            "type": "String"
        },
        {
            "name": "occurrenceTime",
            "type": "DateTime"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat",
            "filePattern": "setOfObjects",
            "jsonPathDefinition": {"id": "$.id", "deviceType": "$.context.device.type", "targetResourceType": "$.context.custom.dimensions[0].TargetResourceType", "resourceManagementProcessRunId": "$.context.custom.dimensions[1].ResourceManagementProcessRunId", "occurrenceTime": " $.context.custom.dimensions[2].OccurrenceTime"}
        }
    }
}
```

**Esempio 2: applicazione incrociata di più oggetti con lo stesso modello dalla matrice**

In questo esempio si prevede di trasformare un oggetto JSON radice in più record in risultato tabulare. Se si dispone di un file JSON con il contenuto seguente:

```json
{
    "ordernumber": "01",
    "orderdate": "20170122",
    "orderlines": [
        {
            "prod": "p1",
            "price": 23
        },
        {
            "prod": "p2",
            "price": 13
        },
        {
            "prod": "p3",
            "price": 231
        }
    ],
    "city": [ { "sanmateo": "No 1" } ]
}
```

e lo si vuole copiare in una tabella SQL di Azure nel formato seguente, rendendo flat i dati nella matrice e nel crossjoin con le informazioni radice comuni:

| `ordernumber` | `orderdate` | `order_pd` | `order_price` | `city` |
| --- | --- | --- | --- | --- |
| 01 | 20170122 | P1 | 23 | `[{"sanmateo":"No 1"}]` |
| 01 | 20170122 | P2 | 13 | `[{"sanmateo":"No 1"}]` |
| 01 | 20170122 | P3 | 231 | `[{"sanmateo":"No 1"}]` |


Il set di dati di input con il tipo **JsonFormat** è definito come segue (definizione parziale che include solo le parti pertinenti). Più in particolare:

- La sezione `structure` definisce i nomi di colonna personalizzati e il tipo di dati corrispondente durante la conversione in dati tabulari. Questa sezione è **facoltativa** a meno che non sia necessario eseguire il mapping colonne. Per altre informazioni, vedere [Eseguire il mapping delle colonne del set di dati di origine alle colonne del set di dati di destinazione](copy-activity-schema-and-type-mapping.md).
- `jsonNodeReference` indica di seguire l'iterazione dei dati e di estrarli dagli oggetti con lo stesso modello sotto la **matrice**`orderlines`.
- `jsonPathDefinition` specifica il percorso JSON per ogni colonna indicante da dove estrarre i dati. In questo esempio `ordernumber`, `orderdate` e `city` sono sotto l'oggetto radice con il percorso JSON che inizia con `$.`, mentre `order_pd` e `order_price` sono definiti con il percorso derivato dall'elemento matrice senza `$.`.

```json
"properties": {
    "structure": [
        {
            "name": "ordernumber",
            "type": "String"
        },
        {
            "name": "orderdate",
            "type": "String"
        },
        {
            "name": "order_pd",
            "type": "String"
        },
        {
            "name": "order_price",
            "type": "Int64"
        },
        {
            "name": "city",
            "type": "String"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat",
            "filePattern": "setOfObjects",
            "jsonNodeReference": "$.orderlines",
            "jsonPathDefinition": {"ordernumber": "$.ordernumber", "orderdate": "$.orderdate", "order_pd": "prod", "order_price": "price", "city": " $.city"}
        }
    }
}
```

**Tenere presente quanto segue:**

* Se `structure` e `jsonPathDefinition` non sono definite nel set di dati della data factory, l'attività di copia rileva lo schema dal primo oggetto e rende flat l'intero oggetto.
* Se l'input JSON presenta una matrice, per impostazione predefinita, l'attività di copia converte l'intero valore della matrice in una stringa. È possibile scegliere di estrarre i dati usando `jsonNodeReference` e/o `jsonPathDefinition` oppure di ignorarlo non specificandolo in `jsonPathDefinition`.
* Se ci sono nomi duplicati allo stesso livello, l'attività di copia sceglie quello più recente.
* I nomi delle proprietà distinguono tra maiuscole e minuscole. Due proprietà con lo stesso nome ma con una combinazione differente di maiuscole e minuscole vengono considerate come due proprietà diverse.

**Caso 2: Scrittura dei dati nel file JSON**

Se nel database SQL è presente la tabella seguente:

| id | order_date | order_price | order_by |
| --- | --- | --- | --- |
| 1 | 20170119 | 2000 | David |
| 2 | 20170120 | 3500 | Patrick |
| 3 | 20170121 | 4000 | Jason |

e per ogni record si prevede di scrivere in un oggetto JSON nel formato seguente:

```json
{
    "id": "1",
    "order": {
        "date": "20170119",
        "price": 2000,
        "customer": "David"
    }
}
```

Il set di dati di output con il tipo **JsonFormat** è definito come segue (definizione parziale che include solo le parti pertinenti). Più in particolare, la sezione `structure` definisce i nomi di proprietà personalizzati nel file di destinazione e viene usato `nestingSeparator` (il valore predefinito è ".") per identificare il livello di annidamento dal nome. Questa sezione è **facoltativa** a meno che non si voglia modificare il nome della proprietà confrontandolo con il nome della colonna di origine o annidare alcune delle proprietà.

```json
"properties": {
    "structure": [
        {
            "name": "id",
            "type": "String"
        },
        {
            "name": "order.date",
            "type": "String"
        },
        {
            "name": "order.price",
            "type": "Int64"
        },
        {
            "name": "order.customer",
            "type": "String"
        }
    ],
    "typeProperties": {
        "folderPath": "mycontainer/myfolder",
        "format": {
            "type": "JsonFormat"
        }
    }
}
```

## <a name="parquet-format"></a>Formato Parquet

>[!NOTE]
>Data Factory introdotto un nuovo set di dati in formato parquet, vedere l'articolo relativo al [formato parquet](format-parquet.md) con i dettagli. Le configurazioni seguenti nel set di dati dell'archivio dati basato su file sono ancora supportate così come sono per le Compabitility precedenti. Si consiglia di utilizzare il nuovo modello in futuro.

Per analizzare i file Parquet o scrivere i dati in formato Parquet, impostare la proprietà `format` `type` su **ParquetFormat**. Non è necessario specificare le proprietà nella sezione Format all'interno della sezione typeProperties. Esempio:

```json
"format":
{
    "type": "ParquetFormat"
}
```

Tenere presente quanto segue:

* I tipi di dati complessi non sono supportati (MAP, LIST).
* Spazi vuoti nel nome della colonna non sono supportati.
* Il file Parquet dispone delle opzioni relative alla compressione seguenti: NONE, SNAPPY, GZIP e LZO. Data Factory supporta la lettura dei dati dal file Parquet in uno di questi formati compressi tranne LZO. Per leggere i dati, usa il codec di compressione dei metadati. Tuttavia, durante la scrittura in un file Parquet, Data Factory sceglie SNAPPY, cioè il valore predefinito per il formato Parquet. Al momento non esiste alcuna opzione per ignorare tale comportamento.

> [!IMPORTANT]
> Per le copie attivate dal runtime di integrazione self-hosted, ad esempio tra l'archivio dati locale e quello nel cloud, se non si esegue una copia **identica** dei file Parquet, è necessario installare **JRE 8 (Java Runtime Environment) a 64 bit o OpenJDK** nel computer del runtime di integrazione. Per informazioni più dettagliate, vedere il paragrafo seguente.

Per la copia in esecuzione nel runtime di integrazione self-hosted con la serializzazione/deserializzazione dei file Parquet, il file di definizione dell'applicazione (AFD) individua il runtime Java eseguendo prima una ricerca di JRE nel Registro di sistema *`(SOFTWARE\JavaSoft\Java Runtime Environment\{Current Version}\JavaHome)`* . In caso di esito negativo, esegue una ricerca di OpenJDK nella variabile di sistema *`JAVA_HOME`* .

- **Per usare JRE**: il runtime di integrazione a 64 bit richiede JRE a 64 bit disponibile [qui](https://go.microsoft.com/fwlink/?LinkId=808605).
- **Per usare OpenJDK**: è supportato a partire dalla versione 3.13 del runtime di integrazione. Includere jvm.dll in un pacchetto con tutti gli altri assembly necessari di OpenJDK nel computer del runtime di integrazione self-hosted e impostare di conseguenza la variabile di ambiente di sistema JAVA_HOME.

>[!TIP]
>Se si copiano i dati nel/dal formato Parquet usando il runtime di integrazione self-hosted e si verifica l'errore "An error occurred when invoking java, message: **java.lang.OutOfMemoryError:Java heap space**" (Errore durante la chiamata di Java, messaggio: java.lang.OutOfMemoryError: spazio dell'heap di Java), è possibile aggiungere una variabile di ambiente `_JAVA_OPTIONS` nel computer che ospita il runtime di integrazione self-hosted per regolare le dimensioni min/max dell'heap per JVM e poter ottimizzare la copia, quindi eseguire di nuovo la pipeline.

![Impostare le dimensioni dell'heap JVM nel runtime di integrazione self-hosted](./media/supported-file-formats-and-compression-codecs/set-jvm-heap-size-on-selfhosted-ir.png)

Esempio: impostare la variabile `_JAVA_OPTIONS` con il valore `-Xms256m -Xmx16g`. Il flag `Xms` specifica il pool di allocazione della memoria iniziale per Java Virtual Machine (JVM), mentre `Xmx` specifica il pool di allocazione della memoria massima. JVM verrà quindi avviato con una quantità di memoria pari a `Xms` e potrà usare una quantità massima di memoria pari a `Xmx`. Per impostazione predefinita, Azure Data Factory usa una quantità di memoria minima pari a 64 MB e una quantità di memoria massima pari a 1 GB.

### <a name="data-type-mapping-for-parquet-files"></a>Mapping dei tipi di dati per i file Parquet

| Tipo di dati provvisori di Data Factory | Tipo Parquet primitivo | Tipo Parquet originale (deserializzazione) | Tipo Parquet originale (serializzazione) |
|:--- |:--- |:--- |:--- |
| Boolean | Boolean | N/D | N/D |
| SByte | Int32 | Int8 | Int8 |
| Byte | Int32 | UInt8 | Int16 |
| Int16 | Int32 | Int16 | Int16 |
| UInt16 | Int32 | UInt16 | Int32 |
| Int32 | Int32 | Int32 | Int32 |
| UInt32 | Int64 | UInt32 | Int64 |
| Int64 | Int64 | Int64 | Int64 |
| UInt64 | Int64/Binary | UInt64 | Decimal |
| Single | Float | N/D | N/D |
| Double | Double | N/D | N/D |
| Decimal | Binary | Decimal | Decimal |
| String | Binary | Utf8 | Utf8 |
| DateTime | Int96 | N/D | N/D |
| TimeSpan | Int96 | N/D | N/D |
| DateTimeOffset | Int96 | N/D | N/D |
| ByteArray | Binary | N/D | N/D |
| Guid | Binary | Utf8 | Utf8 |
| Char | Binary | Utf8 | Utf8 |
| CharArray | Non supportate | N/D | N/D |

## <a name="orc-format"></a>Formato ORC

Per analizzare i file ORC o scrivere i dati in formato ORC, impostare la proprietà `format` `type` su **OrcFormat**. Non è necessario specificare le proprietà nella sezione Format all'interno della sezione typeProperties. Esempio:

```json
"format":
{
    "type": "OrcFormat"
}
```

Tenere presente quanto segue:

* I tipi di dati complessi non sono supportati (STRUCT, MAP, LIST, UNION).
* Spazi vuoti nel nome della colonna non sono supportati.
* Il file ORC dispone di tre [opzioni relative alla compressione](https://hortonworks.com/blog/orcfile-in-hdp-2-better-compression-better-performance/): NONE, ZLIB, SNAPPY. Data Factory supporta la lettura dei dati dal file ORC in uno di questi formati compressi. Per leggere i dati, Data Factoy usa la compressione codec dei metadati. Tuttavia, durante la scrittura in un file ORC, Data Factory sceglie ZLIB che è il valore predefinito per ORC. Al momento non esiste alcuna opzione per ignorare tale comportamento.

> [!IMPORTANT]
> Per le copie attivate dal runtime di integrazione self-hosted, ad esempio tra l'archivio dati locale e quello nel cloud, se non si esegue una copia **identica** dei file ORC, è necessario installare **JRE 8 (Java Runtime Environment) a 64 bit o OpenJDK** nel computer del runtime di integrazione. Per informazioni più dettagliate, vedere il paragrafo seguente.

Per la copia in esecuzione nel runtime di integrazione self-hosted con la serializzazione/deserializzazione dei file ORC, il file di definizione dell'applicazione (AFD) individua il runtime Java eseguendo prima una ricerca di JRE nel Registro di sistema *`(SOFTWARE\JavaSoft\Java Runtime Environment\{Current Version}\JavaHome)`* . In caso di esito negativo, esegue una ricerca di OpenJDK nella variabile di sistema *`JAVA_HOME`* .

- **Per usare JRE**: il runtime di integrazione a 64 bit richiede JRE a 64 bit disponibile [qui](https://go.microsoft.com/fwlink/?LinkId=808605).
- **Per usare OpenJDK**: è supportato a partire dalla versione 3.13 del runtime di integrazione. Includere jvm.dll in un pacchetto con tutti gli altri assembly necessari di OpenJDK nel computer del runtime di integrazione self-hosted e impostare di conseguenza la variabile di ambiente di sistema JAVA_HOME.

### <a name="data-type-mapping-for-orc-files"></a>Mapping dei tipi di dati per i file ORC

| Tipo di dati provvisori di Data Factory | Tipi ORC |
|:--- |:--- |
| Boolean | Boolean |
| SByte | Byte |
| Byte | Short |
| Int16 | Short |
| UInt16 | Int |
| Int32 | Int |
| UInt32 | Long |
| Int64 | Long |
| UInt64 | String |
| Single | Float |
| Double | Double |
| Decimal | Decimal |
| String | String |
| DateTime | Timestamp |
| DateTimeOffset | Timestamp |
| TimeSpan | Timestamp |
| ByteArray | Binary |
| Guid | String |
| Char | Char(1) |

## <a name="avro-format"></a>Formato AVRO

>[!NOTE]
>Data Factory introdotto un nuovo set di dati in formato Avro, vedere l'articolo relativo al [formato avri](format-avro.md) con i dettagli. Le configurazioni seguenti nel set di dati dell'archivio dati basato su file sono ancora supportate così come sono per le Compabitility precedenti. Si consiglia di utilizzare il nuovo modello in futuro.

Per analizzare i file Avro o scrivere i dati in formato Avro, impostare la proprietà `format` `type` su **AvroFormat**. Non è necessario specificare le proprietà nella sezione Format all'interno della sezione typeProperties. Esempio:

```json
"format":
{
    "type": "AvroFormat",
}
```

Per usare il formato Avro in una tabella Hive, fare riferimento all' [esercitazione su Apache Hive](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe).

Tenere presente quanto segue:

* I [tipi di dati complessi](https://avro.apache.org/docs/current/spec.html#schema_complex) non sono supportati (record, enumerazioni, matrici, mappe, unioni e dati fissi).

## <a name="binary-format"></a>Formato binario

Per informazioni dettagliate, vedere l'articolo [formato binario](format-binary.md) .

## <a name="compression-support"></a>Supporto della compressione

Azure Data Factory supporta la compressione/decompressione dei dati durante la copia. Quando si specifica una proprietà `compression` in un set di dati di input, l'attività di copia legge i dati compressi dall'origine e li decomprime. Quando si specifica la proprietà in un set di dati di output, l'attività di copia comprime e quindi scrive i dati nel sink. Di seguito vengono forniti alcuni scenari di esempio:

* Leggere i dati compressi GZIP da un BLOB di Azure, decomprimerli e scrivere i dati del risultato in un database SQL di Azure. Definire il set di dati di input del BLOB di Azure con la proprietà `compression` `type` impostata su GZIP.
* Leggere i dati da un file di testo normale dal file system locale, comprimerli usando il formato GZIP e scrivere i dati compressi in un BLOB di Azure. Definire il set di dati di output del BLOB di Azure con la proprietà `compression` `type` impostata su GZIP.
* Leggere il file con estensione zip dal server FTP, decomprimerlo per ottenere i file all'interno e inserire i file in Azure Data Lake Store. Definire un set di dati FTP di input con la proprietà `compression` `type` impostata su ZipDeflate.
* Leggere i dati compressi GZIP da un BLOB di Azure, decomprimerli, comprimerli usando BZIP2 e scrivere i dati del risultato in un BLOB di Azure. Definire il set di dati di input del BLOB di Azure con `compression` `type` impostato su GZIP e il set di dati di output con `compression` `type` impostato su BZIP2.

Per specificare la compressione per un set di dati, usare la proprietà **compression** nel set di dati JSON come illustrato di seguito:

```json
{
    "name": "AzureBlobDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
            "referenceName": "StorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "fileName": "pagecounts.csv.gz",
            "folderPath": "compression/file/",
            "format": {
                "type": "TextFormat"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

La sezione **compression** ha due proprietà:

* **Type:** codec di compressione, che può essere **GZIP**, **Deflate**, **BZIP2** o **ZipDeflate**.
* **Level:** rapporto di compressione, che può essere **Optimal** o **Fastest**.

  * **Fastest:** l'operazione di compressione deve essere completata il più rapidamente possibile, anche se il file risultante non viene compresso in modo ottimale.
  * **Optimal**: l'operazione di compressione deve comprimere il file in modo ottimale, anche se il completamento richiede più tempo.

    Per maggiori informazioni, vedere l'argomento relativo al [livello di compressione](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) .

> [!NOTE]
> Le impostazioni di compressione non sono attualmente supportate per i dati in **AvroFormat**, **OrcFormat** o **ParquetFormat**. Quando si leggono file in questi formati, Data Factory rileva e usa il codec di compressione nei metadati. Quando si scrive in un file che si presenta in uno di questi formati, Data Factory sceglie il codice di compressione predefinito per il formato specifico. ad esempio ZLIB per OrcFormat e SNAPPY per ParquetFormat.

## <a name="unsupported-file-types-and-compression-formats"></a>Tipi di file e formati di compressione non supportati

È possibile utilizzare le funzionalità di estendibilità di Azure Data Factory per trasformare i file non supportati.
Due opzioni includono funzioni di Azure e attività personalizzate usando Azure Batch.

È possibile vedere un esempio che usa una funzione di Azure per [estrarre il contenuto di un file tar](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV2/UntarAzureFilesWithAzureFunction). Per altre informazioni, vedere [attività di funzioni di Azure](https://docs.microsoft.com/azure/data-factory/control-flow-azure-function-activity).

È anche possibile compilare questa funzionalità usando un'attività DotNet personalizzata. Altre informazioni sono disponibili [qui](https://docs.microsoft.com/azure/data-factory/transform-data-using-dotnet-custom-activity)

## <a name="next-steps"></a>Passaggi successivi

Per gli archivi dati basati su file supportati da Azure Data Factory, vedere i seguenti articoli:

- [Connettore di Archiviazione BLOB di Azure](connector-azure-blob-storage.md)
- [Connettore di Azure Data Lake Store](connector-azure-data-lake-store.md)
- [Connettore di Amazon S3](connector-amazon-simple-storage-service.md)
- [Connettore File System](connector-file-system.md)
- [Connettore FTP](connector-ftp.md)
- [Connettore SFTP](connector-sftp.md)
- [Connettore HDFS](connector-hdfs.md)
- [Connettore HTTP](connector-http.md)
