---
title: Associazioni di Azure Cosmos DB per Funzioni 1.x
description: Informazioni su come usare trigger e associazioni di Azure Cosmos DB in Funzioni di Azure.
services: functions
author: craigshoemaker
ms.author: cshoe
manager: gwallace
keywords: Funzioni di Azure, Funzioni, elaborazione eventi, calcolo dinamico, architettura senza server
ms.service: azure-functions
ms.topic: reference
ms.date: 11/21/2017
ms.custom: seodec18
ms.openlocfilehash: 0e6782c48543723438ee332313de268117dee3e9
ms.sourcegitcommit: 9b80d1e560b02f74d2237489fa1c6eb7eca5ee10
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/01/2019
ms.locfileid: "67480731"
---
# <a name="azure-cosmos-db-bindings-for-azure-functions-1x"></a>Associazioni di Azure Cosmos DB per Funzioni di Azure 1.x

> [!div class="op_single_selector" title1="Selezionare la versione del runtime di funzioni di Azure in uso: "]
> * [Versione 1](functions-bindings-cosmosdb.md)
> * [Versione 2](functions-bindings-cosmosdb-v2.md)

Questo articolo descrive come usare le associazioni di [Azure Cosmos DB](../cosmos-db/serverless-computing-database.md) in Funzioni di Azure. Funzioni di Azure supporta i trigger e le associazioni di input e output per Azure Cosmos DB.

> [!NOTE]
> Questo articolo riguarda Funzioni di Azure 1.x. Per informazioni su come usare le associazioni in Funzioni 2.x, vedere [Azure Cosmos DB bindings for Azure Functions 2.x](functions-bindings-cosmosdb-v2.md) (Associazioni di Azure Cosmos DB per Funzioni di Azure 2.x).
>
>Questa associazione è stata originariamente denominata DocumentDB. In Funzioni versione 1.x, solo il trigger viene rinominato Cosmos DB, mentre l'associazione di input, l'associazione di output e il pacchetto NuGet conservano il nome di DocumentDB.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE]
> Le associazioni di Azure Cosmos DB sono supportate solo per l'uso con l'API SQL. Per tutte le altre API di Azure Cosmos DB è necessario accedere al database dalla funzione usando il client statico per l'API, incluse l'[API MongoDB](../cosmos-db/mongodb-introduction.md), l'[API Cassandra](../cosmos-db/cassandra-introduction.md), l'[API Gremlin](../cosmos-db/graph-introduction.md) e l'[API Tabella](../cosmos-db/table-introduction.md).

## <a name="packages---functions-1x"></a>Pacchetti: Funzioni 1.x

Le associazioni di Azure Cosmos DB per Funzioni versione 1.x sono incluse nel pacchetto NuGet [Microsoft.Azure.WebJobs.Extensions.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DocumentDB) versione 1.x. Il codice sorgente per le associazioni si trova nel repository GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/v2.x/src/WebJobs.Extensions.DocumentDB).

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="trigger"></a>Trigger

Il trigger di Azure Cosmos DB usa il [feed di modifiche di Azure Cosmos DB](../cosmos-db/change-feed.md) per rimanere in ascolto degli inserimenti e degli aggiornamenti nelle partizioni. Il feed delle modifiche pubblica aggiunte e aggiornamenti, non eliminazioni.

## <a name="trigger---example"></a>Trigger - esempio

Vedere l'esempio specifico per ciascun linguaggio:

* [C#](#trigger---c-example)
* [Script C# (file con estensione csx)](#trigger---c-script-example)
* [JavaScript](#trigger---javascript-example)

[Ignora esempi trigger](#trigger---attributes)

### <a name="trigger---c-example"></a>Trigger - esempio in C#

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che viene richiamata in caso di inserimenti o aggiornamenti nel database e nella raccolta specificati.

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;

namespace CosmosDBSamplesV1
{
    public static class CosmosTrigger
    {
        [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "ToDoItems",
            collectionName: "Items",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]IReadOnlyList<Document> documents,
            TraceWriter log)
        {
            if (documents != null && documents.Count > 0)
            {
                log.Info($"Documents modified: {documents.Count}");
                log.Info($"First document Id: {documents[0].Id}");
            }
        }
    }
}
```

[Ignora esempi trigger](#trigger---attributes)

### <a name="trigger---c-script-example"></a>Trigger - esempio di script C#

L'esempio seguente mostra un'associazione di trigger di Cosmos DB in un file *function.json* e una [funzione script C#](functions-reference-csharp.md) che usa l'associazione. La funzione scrive i messaggi di log quando vengono modificati i record di Cosmos DB.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Ecco il codice script C#:

```cs
    #r "Microsoft.Azure.Documents.Client"
    
    using System;
    using Microsoft.Azure.Documents;
    using System.Collections.Generic;
    

    public static void Run(IReadOnlyList<Document> documents, TraceWriter log)
    {
        log.Info("Documents modified " + documents.Count);
        log.Info("First document Id " + documents[0].Id);
    }
```

[Ignora esempi trigger](#trigger---attributes)

### <a name="trigger---javascript-example"></a>Trigger - esempio JavaScript

L'esempio seguente mostra un'associazione di trigger di Cosmos DB in un file *function.json* e una [funzione JavaScript](functions-reference-node.md) che usa l'associazione. La funzione scrive i messaggi di log quando vengono modificati i record di Cosmos DB.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Ecco il codice JavaScript:

```javascript
    module.exports = function (context, documents) {
        context.log('First document Id modified : ', documents[0].id);

        context.done();
    }
```

## <a name="trigger---attributes"></a>Trigger - attributi

Nelle [librerie di classi C#](functions-dotnet-class-library.md) usare l'attributo [CosmosDBTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/Trigger/CosmosDBTriggerAttribute.cs).

Il costruttore dell'attributo accetta il nome del database e il nome della raccolta. Per informazioni su tali impostazioni e altre proprietà che è possibile configurare, vedere [Trigger - configurazione](#trigger---configuration). Di seguito è riportato un esempio dell'attributo `CosmosDBTrigger` in una firma del metodo:

```csharp
    [FunctionName("DocumentUpdates")]
    public static void Run(
        [CosmosDBTrigger("database", "collection", ConnectionStringSetting = "myCosmosDB")]
    IReadOnlyList<Document> documents,
        TraceWriter log)
    {
        ...
    }
```

Per un esempio completo, vedere [Trigger - esempio in C#](#trigger---c-example).

## <a name="trigger---configuration"></a>Trigger - configurazione

Nella tabella seguente sono illustrate le proprietà di configurazione dell'associazione impostate nel file *function.json* e nell'attributo `CosmosDBTrigger`.

|Proprietà di function.json | Proprietà dell'attributo |DESCRIZIONE|
|---------|---------|----------------------|
|**type** || Il valore deve essere impostato su `cosmosDBTrigger`. |
|**direction** || Il valore deve essere impostato su `in`. Questo parametro viene impostato automaticamente quando si crea il trigger nel portale di Azure. |
|**name** || Il nome della variabile usato nel codice funzione che rappresenta l'elenco di documenti con le modifiche. |
|**connectionStringSetting**|**ConnectionStringSetting** | Il nome di un'impostazione dell'app che contiene la stringa di connessione usata per connettersi all'account di Azure Cosmos DB monitorato. |
|**databaseName**|**DatabaseName**  | Il nome del database di Azure Cosmos DB con la raccolta monitorata. |
|**collectionName** |**CollectionName** | Il nome della raccolta monitorata. |
|**leaseConnectionStringSetting** | **LeaseConnectionStringSetting** | (Facoltativo) Il nome di un'impostazione app che contiene la stringa di connessione al servizio in cui si trova la raccolta del lease. Se non impostato, viene usato il valore `connectionStringSetting`. Questo parametro viene impostato automaticamente al momento della creazione dell'associazione nel portale. La stringa di connessione per la raccolta di lease deve avere autorizzazioni di scrittura.|
|**leaseDatabaseName** |**LeaseDatabaseName** | (Facoltativo) Il nome del database in cui si trova la raccolta usata per archiviare i lease. Se non impostato, viene usato il valore dell'impostazione `databaseName`. Questo parametro viene impostato automaticamente al momento della creazione dell'associazione nel portale. |
|**leaseCollectionName** | **LeaseCollectionName** | (Facoltativo) Il nome della raccolta usata per archiviare i lease. Se non impostato, viene usato il valore `leases`. |
|**createLeaseCollectionIfNotExists** | **CreateLeaseCollectionIfNotExists** | (Facoltativo) Se impostato su `true`, la raccolta di lease viene creata automaticamente se non esiste già. Il valore predefinito è `false`. |
|**leasesCollectionThroughput**| **LeasesCollectionThroughput**| (Facoltativo) Definisce la quantità di unità richiesta da assegnare quando viene creata la raccolta di lease. Questa impostazione viene usata solo quando `createLeaseCollectionIfNotExists` è impostato su `true`. Questo parametro viene impostato automaticamente al momento della creazione dell'associazione tramite il portale.
|**leaseCollectionPrefix**| **LeaseCollectionPrefix**| (Facoltativo) Se impostato, aggiunge un prefisso ai lease creati nella raccolta Lease per questa funzione, consentendo in modo efficace a due funzioni distinte di Azure di condividere la stessa raccolta Lease usando prefissi diversi.
|**feedPollDelay**| **FeedPollDelay**| (Facoltativo) Se impostato, definisce, in millisecondi, il ritardo tra il polling di una partizione alla ricerca di nuove modifiche al feed, dopo la rimozione di tutte le modifiche correnti. Il valore predefinito è 5000 (5 secondi).
|**leaseAcquireInterval**| **LeaseAcquireInterval**| (Facoltativo) Se impostato, definisce, in millisecondi, l'intervallo per l'avvio di un'attività che consente di calcolare se le partizioni sono distribuite equamente tra le istanze di host note. Il valore predefinito è 13000 (13 secondi).
|**leaseExpirationInterval**| **LeaseExpirationInterval**| (Facoltativo) Se impostato, definisce, in millisecondi, l'intervallo che indica la durata di un lease su un lease che rappresenta una partizione. Se il lease non viene rinnovato entro questo intervallo, ne comporterà la scadenza e la proprietà della partizione passerà a un'altra istanza. Il valore predefinito è 60000 (60 secondi).
|**leaseRenewInterval**| **LeaseRenewInterval**| (Facoltativo) Se impostato, definisce, in millisecondi, l'intervallo di rinnovo per tutti i lease per le partizioni attualmente occupate da un'istanza. Il valore predefinito è 17000 (17 secondi).
|**checkpointFrequency**| **CheckpointFrequency**| (Facoltativo) Se impostato, definisce, in millisecondi, l'intervallo tra i checkpoint dei lease. Il valore predefinito è sempre dopo ogni chiamata di funzione.
|**maxItemsPerInvocation**| **MaxItemsPerInvocation**| (Facoltativo) Se impostato, personalizza il numero massimo di elementi ricevuti per ogni chiamata di funzione.
|**startFromBeginning**| **StartFromBeginning**| (Facoltativo) Se impostato, indica al trigger di iniziare a leggere le modifiche dall'inizio della cronologia della raccolta anziché dall'ora corrente. Questo funziona solo al primo avvio del trigger, perché per le esecuzioni successive i checkpoint sono già archiviati. L'impostazione su `true` in presenza di lease già creati non ha alcun effetto.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="trigger---usage"></a>Trigger - uso

Il trigger richiede una seconda raccolta usata per archiviare i _lease_ nelle partizioni. Sia la raccolta monitorata che la raccolta che contiene i lease deve essere disponibile affinché il trigger funzioni.

>[!IMPORTANT]
> Se più funzioni sono configurate per l'uso di un trigger di Cosmos DB per la stessa raccolta, ogni funzione deve usare una raccolta di lease dedicata oppure è necessario specificare un diverso `LeaseCollectionPrefix` per ogni funzione. In caso contrario verrà attivata solo una delle funzioni. Per informazioni sul prefisso, vedere la [sezione relativa alla configurazione](#trigger---configuration).

Il trigger non indica se un documento è stato aggiornato o aggiunto, fornisce solo il documento stesso. Se è necessario gestire aggiornamenti e aggiunte in modo diverso, è possibile implementare campi di timestamp per le aggiunte o gli aggiornamenti.

## <a name="input"></a>Input

L'associazione di input di Azure Cosmos DB usa l'API SQL per recuperare uno o più documenti di Azure Cosmos DB e li passa al parametro di input della funzione. L'ID documento o i parametri di query possono essere determinati in base al trigger che richiama la funzione.

## <a name="input---examples"></a>Input: esempi

Vedere gli esempi specifici di ogni linguaggio per leggere un singolo documento specificando un valore di ID:

* [C#](#input---c-examples)
* [Script C# (file con estensione csx)](#input---c-script-examples)
* [JavaScript](#input---javascript-examples)
* [F#](#input---f-examples)

[Ignora esempi input](#input---attributes)

### <a name="input---c-examples"></a>Input: esempi in C#

Questa sezione contiene gli esempi seguenti:

* [Trigger della coda e ricerca dell'ID da JSON](#queue-trigger-look-up-id-from-json-c)
* [Trigger HTTP e ricerca dell'ID da una stringa di query](#http-trigger-look-up-id-from-query-string-c)
* [Trigger HTTP e ricerca dell'ID dai dati della route](#http-trigger-look-up-id-from-route-data-c)
* [Trigger HTTP e ricerca dell'ID dai dati della route con SqlQuery](#http-trigger-look-up-id-from-route-data-using-sqlquery-c)
* [Trigger HTTP e recupero di più documenti con SqlQuery](#http-trigger-get-multiple-docs-using-sqlquery-c)
* [Trigger HTTP e recupero di più documenti con DocumentClient](#http-trigger-get-multiple-docs-using-documentclient-c)

Gli esempi fanno riferimento a un tipo semplice `ToDoItem`:

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="queue-trigger-look-up-id-from-json-c"></a>Trigger della coda e ricerca dell'ID da JSON (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che recupera un singolo documento. La funzione viene attivata da un messaggio in coda contenente un oggetto JSON. Il trigger della coda analizza il codice JSON in un oggetto denominato `ToDoItemLookup` contenente l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItemLookup
    {
        public string ToDoItemId { get; set; }
    }
}
```

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromJSON
    {
        [FunctionName("DocByIdFromJSON")]
        public static void Run(
            [QueueTrigger("todoqueueforlookup")] ToDoItemLookup toDoItemLookup,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{ToDoItemId}")]ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info($"C# Queue trigger function processed Id={toDoItemLookup?.ToDoItemId}");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
        }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-query-string-c"></a>Trigger HTTP e ricerca dell'ID da una stringa di query (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa una stringa di query per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromQueryString
    {
        [FunctionName("DocByIdFromQueryString")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{Query.id}")] ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-c"></a>Trigger HTTP e ricerca dell'ID dai dati della route (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa i dati della route per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromRouteData
    {
        [FunctionName("DocByIdFromRouteData")]
        public static HttpResponseMessage Run(
            [HttpTrigger(
                AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems/{id}")]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{id}")] ToDoItem toDoItem,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.Info($"ToDo item not found");
            }
            else
            {
                log.Info($"Found ToDo item, Description={toDoItem.Description}");
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery-c"></a>Trigger HTTP e ricerca dell'ID dai dati della route con SqlQuery (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa i dati della route per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocByIdFromRouteDataUsingSqlQuery
    {
        [FunctionName("DocByIdFromRouteDataUsingSqlQuery")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems2/{id}")]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "select * from ToDoItems r where r.id = {id}")] IEnumerable<ToDoItem> toDoItems,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.Info(toDoItem.Description);
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-sqlquery-c"></a>Trigger HTTP e recupero di più documenti con SqlQuery (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che recupera un elenco di documenti. La funzione viene attivata da una richiesta HTTP. La query è specificata nella proprietà dell'attributo `SqlQuery`.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using System.Net;
using System.Net.Http;

namespace CosmosDBSamplesV1
{
    public static class DocsBySqlQuery
    {
        [FunctionName("DocsBySqlQuery")]
        public static HttpResponseMessage Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "SELECT top 2 * FROM c order by c._ts desc")]
                IEnumerable<ToDoItem> toDoItems,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.Info(toDoItem.Description);
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-documentclient-c"></a>Trigger HTTP e recupero di più documenti con DocumentClient (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che recupera un elenco di documenti. La funzione viene attivata da una richiesta HTTP. Il codice usa un'istanza di `DocumentClient` fornita dall'associazione di Azure Cosmos DB per leggere un elenco di documenti. L'istanza di `DocumentClient` potrebbe essere usata anche per operazioni di scrittura.

```cs
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;

namespace CosmosDBSamplesV1
{
    public static class DocsByUsingDocumentClient
    {
        [FunctionName("DocsByUsingDocumentClient")]
        public static async Task<HttpResponseMessage> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
            string searchterm = req.GetQueryNameValuePairs()
                .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
                .Value;

            if (searchterm == null)
            {
                return req.CreateResponse(HttpStatusCode.NotFound);
            }

            log.Info($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
            IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
                .Where(p => p.Description.Contains(searchterm))
                .AsDocumentQuery();

            while (query.HasMoreResults)
            {
                foreach (ToDoItem result in await query.ExecuteNextAsync())
                {
                    log.Info(result.Description);
                }
            }
            return req.CreateResponse(HttpStatusCode.OK);
        }
    }
}
```

[Ignora esempi input](#input---attributes)

### <a name="input---c-script-examples"></a>Input: esempi di script C#

Questa sezione contiene gli esempi seguenti:

* [Trigger della coda e ricerca dell'ID dalla stringa](#queue-trigger-look-up-id-from-string-c-script)
* [Trigger della coda e recupero di più documenti con SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-c-script)
* [Trigger HTTP e ricerca dell'ID da una stringa di query](#http-trigger-look-up-id-from-query-string-c-script)
* [Trigger HTTP e ricerca dell'ID dai dati della route](#http-trigger-look-up-id-from-route-data-c-script)
* [Trigger HTTP e recupero di più documenti con SqlQuery](#http-trigger-get-multiple-docs-using-sqlquery-c-script)
* [Trigger HTTP e recupero di più documenti con DocumentClient](#http-trigger-get-multiple-docs-using-documentclient-c-script)

Gli esempi di trigger HTTP fanno riferimento a un tipo `ToDoItem` semplice:

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

[Ignora esempi input](#input---attributes)

#### <a name="queue-trigger-look-up-id-from-string-c-script"></a>Trigger della coda e ricerca dell'ID dalla stringa (script C#)

L'esempio seguente mostra un'associazione di input di Cosmos DB in un file *function.json* e una [funzione script C#](functions-reference-csharp.md) che usa l'associazione. La funzione legge un singolo documento e aggiorna il relativo valore di testo.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "inputDocument",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "partitionKey": "{partition key value}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "in"
}
```
Queste proprietà sono descritte nella sezione [configuration](#input---configuration).

Ecco il codice script C#:

```cs
    using System;

    // Change input document contents using Azure Cosmos DB input binding
    public static void Run(string myQueueItem, dynamic inputDocument)
    {
        inputDocument.text = "This has changed.";
    }
```

[Ignora esempi input](#input---attributes)

#### <a name="queue-trigger-get-multiple-docs-using-sqlquery-c-script"></a>Trigger della coda e recupero di più documenti con SqlQuery (script C#)

L'esempio seguente mostra un'associazione di input di Azure Cosmos DB in un file *function.json* e una [funzione script C#](functions-reference-csharp.md) che usa l'associazione. La funzione recupera più documenti specificati da una query SQL mediante un trigger di coda per personalizzare i parametri di query.

Il trigger di coda fornisce un parametro `departmentId`. Un messaggio nella coda di `{ "departmentId" : "Finance" }` restituirà tutti i record per il reparto finanziario.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "documents",
    "type": "documentdb",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connection": "CosmosDBConnection"
}
```

Queste proprietà sono descritte nella sezione [configuration](#input---configuration).

Ecco il codice script C#:

```csharp
    public static void Run(QueuePayload myQueueItem, IEnumerable<dynamic> documents)
    {
        foreach (var doc in documents)
        {
            // operate on each document
        }
    }

    public class QueuePayload
    {
        public string departmentId { get; set; }
    }
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-query-string-c-script"></a>Trigger HTTP e ricerca dell'ID da una stringa di query (script C#)

L'esempio seguente illustra una [funzione script C#](functions-reference-csharp.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa una stringa di query per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

Ecco il file *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}"
    }
  ],
  "disabled": true
}
```

Ecco il codice script C#:

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
        log.Info($"ToDo item not found");
    }
    else
    {
        log.Info($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-c-script"></a>Trigger HTTP e ricerca dell'ID dai dati della route (script C#)

L'esempio seguente illustra una [funzione script C#](functions-reference-csharp.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa i dati della route per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

Ecco il file *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}"
    }
  ],
  "disabled": false
}
```

Ecco il codice script C#:

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
        log.Info($"ToDo item not found");
    }
    else
    {
        log.Info($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-sqlquery-c-script"></a>Trigger HTTP e recupero di più documenti con SqlQuery (script C#)

L'esempio seguente illustra una [funzione script C#](functions-reference-csharp.md) che recupera un elenco di documenti. La funzione viene attivata da una richiesta HTTP. La query è specificata nella proprietà dell'attributo `SqlQuery`.

Ecco il file *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "sqlQuery": "SELECT top 2 * FROM c order by c._ts desc"
    }
  ],
  "disabled": false
}
```

Ecco il codice script C#:

```cs
using System.Net;

public static HttpResponseMessage Run(HttpRequestMessage req, IEnumerable<ToDoItem> toDoItems, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    foreach (ToDoItem toDoItem in toDoItems)
    {
        log.Info(toDoItem.Description);
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-get-multiple-docs-using-documentclient-c-script"></a>Trigger HTTP e recupero di più documenti con DocumentClient (script C#)

L'esempio seguente illustra una [funzione script C#](functions-reference-csharp.md) che recupera un elenco di documenti. La funzione viene attivata da una richiesta HTTP. Il codice usa un'istanza di `DocumentClient` fornita dall'associazione di Azure Cosmos DB per leggere un elenco di documenti. L'istanza di `DocumentClient` potrebbe essere usata anche per operazioni di scrittura.

Ecco il file *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "client",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "inout"
    }
  ],
  "disabled": false
}
```

Ecco il codice script C#:

```cs
#r "Microsoft.Azure.Documents.Client"

using System.Net;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, DocumentClient client, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
    string searchterm = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
        .Value;

    if (searchterm == null)
    {
        return req.CreateResponse(HttpStatusCode.NotFound);
    }

    log.Info($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
    IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
        .Where(p => p.Description.Contains(searchterm))
        .AsDocumentQuery();

    while (query.HasMoreResults)
    {
        foreach (ToDoItem result in await query.ExecuteNextAsync())
        {
            log.Info(result.Description);
        }
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

[Ignora esempi input](#input---attributes)

### <a name="input---javascript-examples"></a>Input: esempi JavaScript

Questa sezione contiene gli esempi seguenti:

* [Trigger della coda e ricerca dell'ID da JSON](#queue-trigger-look-up-id-from-json-javascript)
* [Trigger HTTP e ricerca dell'ID da una stringa di query](#http-trigger-look-up-id-from-query-string-javascript)
* [Trigger HTTP e ricerca dell'ID dai dati della route](#http-trigger-look-up-id-from-route-data-javascript)
* [Trigger della coda e recupero di più documenti con SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-javascript)

[Ignora esempi input](#input---attributes)

#### <a name="queue-trigger-look-up-id-from-json-javascript"></a>Trigger della coda e ricerca dell'ID da JSON (JavaScript)

L'esempio seguente mostra un'associazione di input di Cosmos DB in un file *function.json* e una [funzione JavaScript](functions-reference-node.md) che usa l'associazione. La funzione legge un singolo documento e aggiorna il relativo valore di testo.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "inputDocumentIn",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "in"
},
{
    "name": "inputDocumentOut",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```
Queste proprietà sono descritte nella sezione [configuration](#input---configuration).

Ecco il codice JavaScript:

```javascript
    // Change input document contents using Azure Cosmos DB input binding, using context.bindings.inputDocumentOut
    module.exports = function (context) {
    context.bindings.inputDocumentOut = context.bindings.inputDocumentIn;
    context.bindings.inputDocumentOut.text = "This was updated!";
    context.done();
    };
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-query-string-javascript"></a>Trigger HTTP e ricerca dell'ID da una stringa di query (JavaScript)

L'esempio seguente illustra una [funzione JavaScript](functions-reference-node.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa una stringa di query per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

Ecco il file *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}"
    }
  ],
  "disabled": true
}
```

Ecco il codice JavaScript:

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

[Ignora esempi input](#input---attributes)

#### <a name="http-trigger-look-up-id-from-route-data-javascript"></a>Trigger HTTP e ricerca dell'ID dai dati della route (JavaScript)

L'esempio seguente illustra una [funzione JavaScript](functions-reference-node.md) che recupera un singolo documento. La funzione viene attivata da una richiesta HTTP che usa una stringa di query per specificare l'ID da cercare. Questo ID viene usato per recuperare un documento `ToDoItem` dal database e dalla raccolta specificati.

Ecco il file *function.json*:

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "documentDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}"
    }
  ],
  "disabled": false
}
```

Ecco il codice JavaScript:

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

[Ignora esempi input](#input---attributes)



#### <a name="queue-trigger-get-multiple-docs-using-sqlquery-javascript"></a>Trigger della coda e recupero di più documenti con SqlQuery (JavaScript)

L'esempio seguente mostra un'associazione di input di Azure Cosmos DB in un file *function.json* e una [funzione JavaScript](functions-reference-node.md) che usa l'associazione. La funzione recupera più documenti specificati da una query SQL mediante un trigger di coda per personalizzare i parametri di query.

Il trigger di coda fornisce un parametro `departmentId`. Un messaggio nella coda di `{ "departmentId" : "Finance" }` restituirà tutti i record per il reparto finanziario.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "documents",
    "type": "documentdb",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connection": "CosmosDBConnection"
}
```

Queste proprietà sono descritte nella sezione [configuration](#input---configuration).

Ecco il codice JavaScript:

```javascript
    module.exports = function (context, input) {
        var documents = context.bindings.documents;
        for (var i = 0; i < documents.length; i++) {
            var document = documents[i];
            // operate on each document
        }
        context.done();
    };
```

[Ignora esempi input](#input---attributes)

<a name="infsharp"></a>

### <a name="input---f-examples"></a>Input: esempi in F#

L'esempio seguente mostra un'associazione di input di Cosmos DB in un file *function.json* e una [funzione F#](functions-reference-fsharp.md) che usa l'associazione. La funzione legge un singolo documento e aggiorna il relativo valore di testo.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "inputDocument",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "connection": "MyAccount_COSMOSDB",
    "direction": "in"
}
```

Queste proprietà sono descritte nella sezione [configuration](#input---configuration).

Ecco il codice F#:

```fsharp
    (* Change input document contents using Azure Cosmos DB input binding *)
    open FSharp.Interop.Dynamic
    let Run(myQueueItem: string, inputDocument: obj) =
    inputDocument?text <- "This has changed."
```

Per questo esempio è necessario un file `project.json` che specifichi le dipendenze NuGet `FSharp.Interop.Dynamic` e `Dynamitey`:

```json
{
    "frameworks": {
        "net46": {
            "dependencies": {
                "Dynamitey": "1.0.2",
                "FSharp.Interop.Dynamic": "3.0.0"
            }
        }
    }
}
```

Per aggiungere un file `project.json`, vedere l'argomento relativo alla [gestione dei pacchetti F #](functions-reference-fsharp.md#package).

## <a name="input---attributes"></a>Input - attributi

Nelle [librerie di classi C#](functions-dotnet-class-library.md) usare l'attributo [DocumentDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.DocumentDB/DocumentDBAttribute.cs).

Il costruttore dell'attributo accetta il nome del database e il nome della raccolta. Per informazioni su tali impostazioni e altre proprietà che è possibile configurare, vedere la [sezione seguente relativa alla configurazione](#input---configuration).

## <a name="input---configuration"></a>Input - configurazione

Nella tabella seguente sono illustrate le proprietà di configurazione dell'associazione impostate nel file *function.json* e nell'attributo `DocumentDB`.

|Proprietà di function.json | Proprietà dell'attributo |DESCRIZIONE|
|---------|---------|----------------------|
|**type**     || Il valore deve essere impostato su `documentdb`.        |
|**direction**     || Il valore deve essere impostato su `in`.         |
|**name**     || Nome del parametro di binding che rappresenta il documento nella funzione.  |
|**databaseName** |**DatabaseName** |Database che contiene il documento.        |
|**collectionName** |**CollectionName** | Nome della raccolta che contiene il documento. |
|**id**    | **Id** | ID del documento da recuperare. Questa proprietà supporta le [espressioni di associazione](./functions-bindings-expressions-patterns.md). Non impostare entrambe le proprietà **id** e **sqlQuery**. Se non si imposta una delle due proprietà, verrà recuperato l'intera raccolta. |
|**sqlQuery**  |**SqlQuery**  | Query SQL di Azure Cosmos DB usata per recuperare più documenti. La proprietà supporta le associazioni del runtime, come nell'esempio seguente: `SELECT * FROM c where c.departmentId = {departmentId}`. Non impostare entrambe le proprietà **id** e **sqlQuery**. Se non si imposta una delle due proprietà, verrà recuperato l'intera raccolta.|
|**connessione**     |**ConnectionStringSetting**|Nome dell'impostazione app contenente la stringa di connessione di Azure Cosmos DB.        |
|**partitionKey**|**PartitionKey**|Specifica il valore della chiave di partizione per la ricerca. Può includere i parametri di associazione.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="input---usage"></a>Input - uso

Nelle funzioni C# e F#, quando la funzione termina correttamente, le modifiche apportate al documento di input tramite i parametri di input denominati vengono rese automaticamente persistenti.

Nelle funzioni di JavaScript gli aggiornamenti non vengono eseguiti automaticamente al termine della funzione. Per eseguire gli aggiornamenti usare invece `context.bindings.<documentName>In` e `context.bindings.<documentName>Out`. Vedere l'[esempio JavaScript](#input---javascript-examples).

## <a name="output"></a>Output

L'associazione di output di Azure Cosmos DB consente di scrivere un nuovo documento in un database di Azure Cosmos DB tramite l'API SQL.

## <a name="output---examples"></a>Output - esempi

Vedere gli esempi specifici per ciascun linguaggio:

* [C#](#output---c-examples)
* [Script C# (file con estensione csx)](#output---c-script-examples)
* [JavaScript](#output---javascript-examples)
* [F#](#output---f-examples)

Vedere anche l'[esempio di input](#input---c-examples) che usa `DocumentClient`.

[Ignora esempi output](#output---attributes)

### <a name="output---c-examples"></a>Output - Esempi in C#

Questa sezione contiene gli esempi seguenti:

* Trigger della coda e scrittura di un documento
* Trigger della coda e scrittura di documenti con IAsyncCollector

Gli esempi fanno riferimento a un tipo semplice `ToDoItem`:

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

[Ignora esempi output](#output---attributes)

#### <a name="queue-trigger-write-one-doc-c"></a>Trigger della coda e scrittura di un documento (C#)

L'esempio seguente mostra una [funzione in C#](functions-dotnet-class-library.md) che aggiunge un documento a un database usando i dati forniti nel messaggio dall'archiviazione code.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System;

namespace CosmosDBSamplesV1
{
    public static class WriteOneDoc
    {
        [FunctionName("WriteOneDoc")]
        public static void Run(
            [QueueTrigger("todoqueueforwrite")] string queueMessage,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]out dynamic document,
            TraceWriter log)
        {
            document = new { Description = queueMessage, id = Guid.NewGuid() };

            log.Info($"C# Queue trigger function inserted one row");
            log.Info($"Description={queueMessage}");
        }
    }
}
```

[Ignora esempi output](#output---attributes)

#### <a name="queue-trigger-write-docs-using-iasynccollector-c"></a>Trigger della coda e scrittura di documenti con IAsyncCollector (C#)

L'esempio seguente illustra una [funzione C#](functions-dotnet-class-library.md) che aggiunge una raccolta di documenti a un database usando i dati forniti nel codice JSON di un messaggio in coda.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Threading.Tasks;

namespace CosmosDBSamplesV1
{
    public static class WriteDocsIAsyncCollector
    {
        [FunctionName("WriteDocsIAsyncCollector")]
        public static async Task Run(
            [QueueTrigger("todoqueueforwritemulti")] ToDoItem[] toDoItemsIn,
            [DocumentDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]
                IAsyncCollector<ToDoItem> toDoItemsOut,
            TraceWriter log)
        {
            log.Info($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

            foreach (ToDoItem toDoItem in toDoItemsIn)
            {
                log.Info($"Description={toDoItem.Description}");
                await toDoItemsOut.AddAsync(toDoItem);
            }
        }
    }
}
```

[Ignora esempi output](#output---attributes)

### <a name="output---c-script-examples"></a>Output: esempi di script C#

Questa sezione contiene gli esempi seguenti:

* Trigger della coda e scrittura di un documento
* Trigger della coda e scrittura di documenti con IAsyncCollector

[Ignora esempi output](#output---attributes)

#### <a name="queue-trigger-write-one-doc-c-script"></a>Trigger della coda e scrittura di un documento (script C#)

L'esempio seguente mostra un'associazione di output di Azure Cosmos DB in un file *function.json* e una [funzione script C#](functions-reference-csharp.md) che usa l'associazione. La funzione usa un'associazione di input di coda per una coda che riceve JSON nel formato seguente:

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La funzione crea documenti di Azure Cosmos DB nel formato seguente per ogni record:

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "employeeDocument",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

Queste proprietà sono descritte nella sezione [configuration](#output---configuration).

Ecco il codice script C#:

```cs
    #r "Newtonsoft.Json"

    using Microsoft.Azure.WebJobs.Host;
    using Newtonsoft.Json.Linq;

    public static void Run(string myQueueItem, out object employeeDocument, TraceWriter log)
    {
        log.Info($"C# Queue trigger function processed: {myQueueItem}");

        dynamic employee = JObject.Parse(myQueueItem);

        employeeDocument = new {
            id = employee.name + "-" + employee.employeeId,
            name = employee.name,
            employeeId = employee.employeeId,
            address = employee.address
        };
    }
```

#### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Trigger della coda e scrittura di documenti con IAsyncCollector

Per creare più documenti, è possibile definire l'associazione a `ICollector<T>` o `IAsyncCollector<T>` dove `T` è uno dei tipi supportati.

Questo esempio si riferisce a un tipo `ToDoItem` semplice:

```cs
namespace CosmosDBSamplesV1
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

Ecco il file function.json:

```json
{
  "bindings": [
    {
      "name": "toDoItemsIn",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "todoqueueforwritemulti",
      "connection": "AzureWebJobsStorage"
    },
    {
      "type": "documentDB",
      "name": "toDoItemsOut",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Ecco il codice script C#:

```cs
using System;

public static async Task Run(ToDoItem[] toDoItemsIn, IAsyncCollector<ToDoItem> toDoItemsOut, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

    foreach (ToDoItem toDoItem in toDoItemsIn)
    {
        log.Info($"Description={toDoItem.Description}");
        await toDoItemsOut.AddAsync(toDoItem);
    }
}
```

[Ignora esempi output](#output---attributes)

### <a name="output---javascript-examples"></a>Output: esempi JavaScript

L'esempio seguente mostra un'associazione di output di Azure Cosmos DB in un file *function.json* e una [funzione script JavaScript](functions-reference-node.md) che usa l'associazione. La funzione usa un'associazione di input di coda per una coda che riceve JSON nel formato seguente:

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La funzione crea documenti di Azure Cosmos DB nel formato seguente per ogni record:

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "employeeDocument",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

Queste proprietà sono descritte nella sezione [configuration](#output---configuration).

Ecco il codice JavaScript:

```javascript
    module.exports = function (context) {

      context.bindings.employeeDocument = JSON.stringify({
        id: context.bindings.myQueueItem.name + "-" + context.bindings.myQueueItem.employeeId,
        name: context.bindings.myQueueItem.name,
        employeeId: context.bindings.myQueueItem.employeeId,
        address: context.bindings.myQueueItem.address
      });

      context.done();
    };
```

[Ignora esempi output](#output---attributes)

### <a name="output---f-examples"></a>Output: esempi in F#

L'esempio seguente mostra un'associazione di output di Azure Cosmos DB in un file *function.json* e una [funzione script F#](functions-reference-fsharp.md) che usa l'associazione. La funzione usa un'associazione di input di coda per una coda che riceve JSON nel formato seguente:

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La funzione crea documenti di Azure Cosmos DB nel formato seguente per ogni record:

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Ecco i dati di associazione nel file *function.json*:

```json
{
    "name": "employeeDocument",
    "type": "documentDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connection": "MyAccount_COSMOSDB",
    "direction": "out"
}
```
Queste proprietà sono descritte nella sezione [configuration](#output---configuration).

Ecco il codice F#:

```fsharp
    open FSharp.Interop.Dynamic
    open Newtonsoft.Json

    type Employee = {
      id: string
      name: string
      employeeId: string
      address: string
    }

    let Run(myQueueItem: string, employeeDocument: byref<obj>, log: TraceWriter) =
      log.Info(sprintf "F# Queue trigger function processed: %s" myQueueItem)
      let employee = JObject.Parse(myQueueItem)
      employeeDocument <-
        { id = sprintf "%s-%s" employee?name employee?employeeId
          name = employee?name
          employeeId = employee?employeeId
          address = employee?address }
```

Per questo esempio è necessario un file `project.json` che specifichi le dipendenze NuGet `FSharp.Interop.Dynamic` e `Dynamitey`:

```json
{
    "frameworks": {
        "net46": {
          "dependencies": {
            "Dynamitey": "1.0.2",
            "FSharp.Interop.Dynamic": "3.0.0"
           }
        }
    }
}
```

Per aggiungere un file `project.json`, vedere l'argomento relativo alla [gestione dei pacchetti F #](functions-reference-fsharp.md#package).

## <a name="output---attributes"></a>Output - attributi

Nelle [librerie di classi C#](functions-dotnet-class-library.md) usare l'attributo [DocumentDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.DocumentDB/DocumentDBAttribute.cs).

Il costruttore dell'attributo accetta il nome del database e il nome della raccolta. Per informazioni su tali impostazioni e altre proprietà che è possibile configurare, vedere [Output - configurazione](#output---configuration). Di seguito è riportato un esempio dell'attributo `DocumentDB` in una firma del metodo:

```csharp
    [FunctionName("QueueToDocDB")]
    public static void Run(
        [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
        [DocumentDB("ToDoList", "Items", Id = "id", ConnectionStringSetting = "myCosmosDB")] out dynamic document)
    {
        ...
    }
```

Per un esempio completo, vedere [Output - esempio in C#](#output---c-examples).

## <a name="output---configuration"></a>Output - configurazione

Nella tabella seguente sono illustrate le proprietà di configurazione dell'associazione impostate nel file *function.json* e nell'attributo `DocumentDB`.

|Proprietà di function.json | Proprietà dell'attributo |DESCRIZIONE|
|---------|---------|----------------------|
|**type**     || Il valore deve essere impostato su `documentdb`.        |
|**direction**     || Il valore deve essere impostato su `out`.         |
|**name**     || Nome del parametro di binding che rappresenta il documento nella funzione.  |
|**databaseName** | **DatabaseName**|Database contenente la raccolta in cui viene creato il documento.     |
|**collectionName** |**CollectionName**  | Nome della raccolta in cui viene creato il documento. |
|**createIfNotExists**  |**CreateIfNotExists**    | Valore booleano che indica se la raccolta viene creata quando non esiste. Il valore predefinito è *false* perché le nuove raccolte vengono create con una velocità effettiva riservata, che ha implicazioni in termini di costi. Per altre informazioni vedere la [pagina dei prezzi](https://azure.microsoft.com/pricing/details/documentdb/).  |
|**partitionKey**|**PartitionKey** |Se `CreateIfNotExists` è true, definisce il percorso della chiave di partizione per la raccolta creata.|
|**collectionThroughput**|**CollectionThroughput**| Se `CreateIfNotExists` è true, definisce la [velocità effettiva](../cosmos-db/set-throughput.md) della raccolta creata.|
|**connessione**    |**ConnectionStringSetting** |Nome dell'impostazione app contenente la stringa di connessione di Azure Cosmos DB.        |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="output---usage"></a>Output - uso

Per impostazione predefinita, quando si scrive nel parametro di output della funzione, viene creato un documento nel database. L'ID di questo documento è un GUID generato automaticamente. È possibile specificare l'ID del documento di output specificando la proprietà `id` nell'oggetto JSON passato al parametro di output.

> [!Note]
> Quando si specifica l'ID di un documento esistente, questo viene sovrascritto dal nuovo documento di output.

## <a name="exceptions-and-return-codes"></a>Eccezioni e codici restituiti

| Associazione | Riferimenti |
|---|---|
| Cosmos DB | [Codici di errore di CosmosDB](https://docs.microsoft.com/rest/api/cosmos-db/http-status-codes-for-cosmosdb) |

## <a name="next-steps"></a>Passaggi successivi

* [Altre informazioni sull'elaborazione di database senza server con Cosmos DB](../cosmos-db/serverless-computing-database.md)
* [Altre informazioni su trigger e associazioni di Funzioni di Azure](functions-triggers-bindings.md)

<!---
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Cosmos DB trigger](functions-create-cosmos-db-triggered-function.md)
--->
