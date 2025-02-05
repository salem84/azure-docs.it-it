---
title: Aggiungere la ricerca full-text all'archivio BLOB di Azure
titleSuffix: Azure Search
description: Estrai il contenuto e Aggiungi la struttura ai BLOB di Azure durante la compilazione di un indice di ricerca full-text in ricerca di Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: search
ms.topic: conceptual
ms.date: 10/09/2019
ms.openlocfilehash: db54c7886f5256dab41325cd12f1b893be732d72
ms.sourcegitcommit: f29fec8ec945921cc3a89a6e7086127cc1bc1759
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/17/2019
ms.locfileid: "72529038"
---
# <a name="add-full-text-search-to-azure-blob-data-using-azure-search"></a>Aggiungere la ricerca full-text ai dati BLOB di Azure con ricerca di Azure

La ricerca in un'ampia gamma di tipi di contenuto inclusi nell'archiviazione BLOB di Azure può rivelarsi un problema complesso da risolvere. Tuttavia, è possibile indicizzare e cercare il contenuto dei BLOB in pochi clic usando [ricerca di Azure](search-what-is-azure-search.md). Ricerca di Azure offre integrazione incorporata per l'indicizzazione dell'archiviazione BLOB tramite un [*indicizzatore BLOB*](search-howto-indexing-azure-blob-storage.md) che aggiunge funzionalità per l'indicizzazione in grado di riconoscere origini dati.

## <a name="what-it-means-to-add-full-text-search-to-blob-data"></a>Cosa significa aggiungere la ricerca full-text ai dati BLOB

Ricerca di Azure è un servizio di ricerca cloud che fornisce motori di indicizzazione e query che operano su indici definiti dall'utente ospitati nel servizio di ricerca. Il percorso di condivisione del contenuto ricercabile con il motore di query nel cloud è necessario per le prestazioni, restituendo i risultati a una velocità prevista dagli utenti dalle query di ricerca.

Ricerca di Azure si integra con archiviazione BLOB di Azure a livello di indicizzazione, importando il contenuto BLOB come documenti di ricerca indicizzati in *indici invertiti* e altre strutture di query che supportano query di testo e espressioni di filtro gratuite. Poiché il contenuto del BLOB è indicizzato in un indice di ricerca, l'accesso al contenuto BLOB può sfruttare la gamma completa di funzionalità di query in ricerca di Azure.

Una volta che l'indice viene creato e popolato, esiste indipendentemente dal contenitore BLOB, ma è possibile rieseguire le operazioni di indicizzazione per aggiornare l'indice con le modifiche apportate al contenitore sottostante. Per il rilevamento delle modifiche vengono usate le informazioni sul timestamp nei singoli BLOB. È possibile optare per l'esecuzione pianificata o l'indicizzazione su richiesta come meccanismo di aggiornamento.

Gli input sono i BLOB, in un singolo contenitore, nell'archivio BLOB di Azure. I BLOB possono essere quasi qualsiasi tipo di dati di testo. Se i BLOB contengono immagini, è possibile aggiungere l' [arricchimento di intelligenza artificiale all'indicizzazione BLOB](search-blob-ai-integration.md) per creare ed estrarre il testo dalle immagini.

L'output è sempre un indice di ricerca di Azure, usato per la ricerca di testo veloce, il recupero e l'esplorazione nelle applicazioni client. In between è l'architettura della pipeline di indicizzazione stessa. La pipeline è basata sulla funzionalità dell' *indicizzatore* , descritta più avanti in questo articolo.

## <a name="start-with-services"></a>Inizia a usare i servizi

Sono necessari ricerca di Azure e l'archiviazione BLOB di Azure. All'interno dell'archiviazione BLOB è necessario un contenitore che fornisca contenuto di origine.

È possibile iniziare direttamente nella pagina del portale dell'account di archiviazione. Nella pagina di spostamento a sinistra, in **servizio BLOB** fare clic su **Aggiungi ricerca di Azure** per creare un nuovo servizio o selezionarne uno esistente. 

Una volta aggiunto ricerca di Azure all'account di archiviazione, è possibile seguire il processo standard per indicizzare i dati BLOB. È consigliabile usare la procedura guidata **Importa dati** in ricerca di Azure per un'introduzione iniziale semplice o chiamare le API REST usando uno strumento come postazione. Questa esercitazione illustra i passaggi per chiamare l'API REST in post, ovvero [indicizzare e cercare i dati semistrutturati (BLOB JSON) in ricerca di Azure](search-semi-structured-data.md). 

## <a name="use-a-blob-indexer"></a>Usare un indicizzatore BLOB

Un *indicizzatore* è un sottoservizio in grado di riconoscere le origini dati dotato di logica interna per il campionamento dei dati, la lettura dei dati dei metadati, il recupero dei dati e la serializzazione dei dati dai formati nativi ai documenti JSON per l'importazione successiva. 

I BLOB in archiviazione di Azure vengono indicizzati usando l' [indicizzatore di archiviazione BLOB di ricerca di Azure](search-howto-indexing-azure-blob-storage.md). È possibile richiamare questo indicizzatore usando la procedura guidata **Importa dati** , un'API REST o .NET SDK. Nel codice, questo indicizzatore viene usato impostando il tipo e fornendo le informazioni di connessione che includono un account di archiviazione di Azure insieme a un contenitore BLOB. È possibile sottoscrivere i BLOB creando una directory virtuale, che può quindi essere passata come parametro o filtrando in base a un'estensione del tipo di file.

Un indicizzatore esegue il "cracking del documento", aprendo un BLOB per esaminare il contenuto. Dopo la connessione all'origine dati, è il primo passaggio della pipeline. Per i dati BLOB, questo è il percorso in cui vengono rilevati PDF, documenti di Office e altri tipi di contenuto. Il cracking dei documenti con estrazione del testo non prevede alcun addebito. Se i BLOB contengono contenuto immagine, le immagini vengono ignorate a meno che non si [aggiunga l'arricchimento di intelligenza artificiale](search-blob-ai-integration.md). L'indicizzazione standard si applica solo al contenuto di testo.

L'indicizzatore BLOB viene fornito con i parametri di configurazione e supporta il rilevamento delle modifiche se i dati sottostanti forniscono informazioni sufficienti. È possibile ottenere altre informazioni sulle funzionalità di base nell' [indicizzatore di archiviazione BLOB di ricerca di Azure](search-howto-indexing-azure-blob-storage.md).

### <a name="supported-content-types"></a>Tipi di contenuto supportati

Eseguendo un indicizzatore BLOB su un contenitore, è possibile estrarre testo e metadati dai tipi di contenuto seguenti con una singola query:

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

### <a name="indexing-blob-metadata"></a>Indicizzazione di metadati BLOB

Uno scenario comune che semplifica l'ordinamento nei BLOB di qualsiasi tipo di contenuto consiste nell'indicizzare i metadati personalizzati e le proprietà di sistema per ogni BLOB. In questo modo, le informazioni per tutti i BLOB vengono indicizzate indipendentemente dal tipo di documento, archiviate in un indice in ricerca di Azure. Usando il nuovo indice, è possibile procedere con l'ordinamento, il filtro e il facet in tutto il contenuto dell'archiviazione BLOB.

### <a name="indexing-json-blobs"></a>Indicizzazione di BLOB JSON
È possibile configurare Ricerca di Azure in modo da estrarre il contenuto strutturato dei BLOB che contengono JSON. Ricerca di Azure può leggere BLOB JSON e analizzare il contenuto strutturato nei campi appropriati di un documento di Ricerca di Azure. Ricerca di Azure può anche usare BLOB che contengono una matrice di oggetti JSON ed eseguire il mapping di ogni elemento a un documento di Ricerca di Azure separato. È possibile impostare una modalità di analisi per influenzare il tipo di oggetto JSON creato dall'indicizzatore.

## <a name="search-blob-content-in-a-search-index"></a>Cerca contenuto BLOB in un indice di ricerca 

L'output di un'indicizzazione è un indice di ricerca, usato per l'esplorazione interattiva con testo libero e query filtrate in un'app client. Per l'esplorazione iniziale e la verifica del contenuto, è consigliabile iniziare con [Esplora ricerche](search-explorer.md) nel portale per esaminare la struttura del documento. In Esplora ricerche è possibile utilizzare [sintassi di query semplice](query-simple-syntax.md), [sintassi di query complete](query-lucene-syntax.md)e [sintassi delle espressioni di filtro](query-odata-filter-orderby-syntax.md) .

Una soluzione più permanente consiste nel raccogliere gli input della query e presentare la risposta come risultati della ricerca in un'applicazione client. L'esercitazione C# seguente illustra come compilare un'applicazione di ricerca: [creare la prima applicazione in ricerca di Azure](tutorial-csharp-create-first-app.md).

## <a name="next-steps"></a>Passaggi successivi

> [!div class="nextstepaction"]
> [Configurare un indicizzatore BLOB](search-howto-indexing-azure-blob-storage.md) 
