---
title: Sintassi di query semplice - Ricerca di Azure
description: Informazioni di riferimento sulla sintassi di query semplice usata per le query di ricerca full-text in Ricerca di Azure.
services: search
ms.service: search
ms.topic: conceptual
ms.date: 08/08/2019
author: brjohnstmsft
ms.author: brjohnst
manager: nitinme
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: e6c5ea86534001e0e5de2b02c4151af70631e4ef
ms.sourcegitcommit: bb8e9f22db4b6f848c7db0ebdfc10e547779cccc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/20/2019
ms.locfileid: "69650019"
---
# <a name="simple-query-syntax-in-azure-search"></a>Sintassi di query semplice in Ricerca di Azure
Ricerca di Azure implementa due linguaggi di query basate su Lucene: il [parser di query semplice](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/simple/SimpleQueryParser.html) e il [parser di query Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html). In Ricerca di Azure la sintassi di query semplice esclude le opzioni fuzzy/inclinazione.  

> [!NOTE]  
>  Ricerca di Azure offre come alternativa per le query più complesse la [sintassi di query Lucene](query-lucene-syntax.md). Per altre informazioni sull'architettura di analisi delle query e i vantaggi di ogni sintassi, vedere [Funzionamento della ricerca full-text in Ricerca di Azure](search-lucene-query-architecture.md).

## <a name="how-to-invoke-simple-parsing"></a>Come richiamare l'analisi semplice

La sintassi semplice è quella predefinita. La chiamata è necessaria solo se si sta reimpostando la sintassi da completa a semplice. Per impostare la sintassi in modo esplicito, usare il parametro di ricerca `queryType`. I valori validi includono `simple|full`, con `simple` come impostazione predefinita e `full` per Lucene. 

## <a name="query-behavior-anomalies"></a>Anomalie di comportamento delle query

Qualsiasi testo con uno o più termini è considerato un valido punto di partenza per l'esecuzione di una query. Ricerca di Azure troverà le corrispondenze con i documenti contenenti uno o tutti i termini, incluse eventuali varianti trovate durante l'analisi del testo. 

Per quanto possa sembrare semplice, esiste un aspetto dell'esecuzione delle query in Ricerca di Azure che *potrebbe* generare risultati imprevisti, aumentando invece di ridurre i risultati della ricerca quando altri termini e operatori vengono aggiunti alla stringa di input. Che questa espansione si verifichi o meno dipende dall'inclusione di un operatore NOT, combinato con un'impostazione del parametro `searchMode` che determina come NOT viene interpretato dal punto di vista dei comportamenti di AND o OR. Con l'impostazione predefinita `searchMode=Any` e un operatore NOT, l'operazione viene calcolata come azione OR e quindi `"New York" NOT Seattle` restituisce tutte le città diverse da Seattle.  

In genere, è più probabile osservare questi comportamenti nei modelli di interazione utente per le applicazioni che eseguono ricerche sul contenuto, dove è più probabile che gli utenti includano un operatore in una query, a differenza dei siti di e-commerce che hanno strutture di esplorazione più integrate. Per altre informazioni, vedere [Operatore NOT](#not-operator). 

## <a name="boolean-operators-and-or-not"></a>Operatori booleani (AND, OR, NOT) 

È possibile incorporare gli operatori in una stringa di query per creare un set completo di criteri in base al quale vengono trovati i documenti corrispondenti. 

### <a name="and-operator-"></a>Operatore AND `+`

L'operatore AND è un segno più. Ad esempio, `wifi+luxury` cercherà i documenti contenenti sia `wifi` che `luxury`.

### <a name="or-operator-"></a>Operator OR `|`

L'operatore OR è un carattere barra verticale o pipe. Ad esempio, `wifi | luxury` cercherà i documenti contenenti `wifi` o `luxury` oppure entrambi.

<a name="not-operator"></a>

### <a name="not-operator--"></a>Operatore NOT `-`

L'operatore NOT è un segno meno. Ad esempio, `wifi –luxury` cercherà i documenti che contengono il termine `wifi` e/o non contengono `luxury` (e/o sono controllati da `searchMode`).

> [!NOTE]  
>  L'opzione `searchMode` controlla se un termine con l'operatore NOT è associato tramite l'operatore AND o OR agli altri termini nella query in assenza di un operatore `+` o `|`. Si ricordi che `searchMode` può essere impostato su `any` (impostazione predefinita) o su `all`. Se si usa `any`, il livello di richiamo delle query aumenta includendo più risultati e per impostazione predefinita il segno `-` verrà interpretato come "OR NOT". Ad esempio, `wifi -luxury` troverà la corrispondenza con documenti contenenti il termine `wifi` o quelli non contenenti il termine `luxury`. Se si usa `all`, il livello di precisione delle query aumenta includendo meno risultati e per impostazione predefinita il segno - verrà interpretato come "AND NOT". Ad esempio, `wifi -luxury` troverà la corrispondenza con documenti contenenti il termine `wifi` e quelli non contenenti il termine "luxury". Si tratta di un comportamento verosimilmente più intuitivo per l'operatore `-`. Valutare quindi l'opportunità di usare `searchMode=all` invece di `searchMode=any` se si vuole ottimizzare il livello di precisione delle ricerche invece che quello di richiamo *e* gli utenti usano spesso l'operatore `-` nelle ricerche.

## <a name="suffix-operator"></a>Operatore suffisso

L'operatore suffisso è un `*`asterisco. `lux*`, ad esempio, cercherà i documenti contenenti un termine che inizia con `lux`, ignorando le lettere maiuscole/minuscole.  

## <a name="phrase-search-operator"></a>Operatore di ricerca frasi

L'operatore phrase racchiude una frase tra virgolette `" "`. Ad esempio, mentre `Roach Motel` (senza virgolette) cerca i documenti contenenti `Roach` e/o `Motel` ovunque e in qualsiasi ordine, `"Roach Motel"` (con le virgolette) troverà solo i documenti contenenti l'intera frase in quell'ordine specifico (l'analisi del testo è comunque applicabile).

## <a name="precedence-operator"></a>Operatore di precedenza

L'operatore di precedenza racchiude la stringa tra parentesi `( )`. Ad esempio, `motel+(wifi | luxury)` cercherà i documenti che contengono il termine del Motel `wifi` e `luxury` o (o entrambi).  

## <a name="escaping-search-operators"></a>Escape degli operatori di ricerca  

 Per usare i simboli elencati sopra come parte effettiva del testo di ricerca, è necessario che siano preceduti da una barra rovesciata come carattere di escape. Ad esempio, `luxury\+hotel` restituirà il termine `luxury+hotel`. Per semplificare i casi più comuni, sono previste due eccezioni a questa regola, in cui i caratteri di escape non sono necessari:  

- L'operatore NOT `-` deve essere preceduto da un carattere di escape solo se è il primo carattere dopo lo spazio vuoto, non se è all'interno di un termine. `wi-fi`, ad esempio, è un termine singolo, mentre i GUID (ad esempio `3352CDD0-EF30-4A2E-A512-3B30AF40F3FD`) vengono trattati come un singolo token.
- L'operatore suffisso `*` deve essere preceduto da un carattere di escape solo se è l'ultimo carattere prima dello spazio vuoto, non se è all'interno di un termine. `wi*fi`, ad esempio, viene considerato come un singolo token.

> [!NOTE]  
>  Anche se i caratteri di escape mantengono uniti i token, l'analisi del testo potrebbe dividerli, a seconda della modalità di analisi. Per informazioni dettagliate, vedere [Supporto per la lingua &#40;API REST per il servizio Ricerca di Azure&#41;](index-add-language-analyzers.md).  

## <a name="see-also"></a>Vedere anche  

+ [Search Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (Cercare documenti - API REST per il servizio Ricerca di Azure) 
+ [Sintassi di query Lucene](query-lucene-syntax.md)
+ [Sintassi delle espressioni OData](query-odata-filter-orderby-syntax.md) 
