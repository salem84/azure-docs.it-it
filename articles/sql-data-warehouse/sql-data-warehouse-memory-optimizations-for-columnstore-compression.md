---
title: Migliorare le prestazioni dell'indice columnstore in Azure SQL Data Warehouse | Microsoft Docs
description: Ridurre i requisiti di memoria o aumentare la memoria disponibile per accrescere al massimo il numero di righe che un indice columnstore comprime in ogni gruppo di righe.
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: load-data
ms.date: 03/22/2019
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: ec85bcc764ba7a7ae6341e0490530c31fdb5a02b
ms.sourcegitcommit: ccb9a7b7da48473362266f20950af190ae88c09b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/05/2019
ms.locfileid: "67595468"
---
# <a name="maximizing-rowgroup-quality-for-columnstore"></a>Ottimizzazione della qualità di un gruppo di righe per columnstore

La qualità di un gruppo di righe è determinata dal numero di righe nel gruppo. Aumentare la memoria disponibile, è possibile ottimizzare il numero di righe di che un indice columnstore comprime ogni rowgroup.  Usare questi metodi per migliorare il tasso di compressione e le prestazioni delle query per gli indici columnstore.

## <a name="why-the-rowgroup-size-matters"></a>Perché sono importanti le dimensioni del gruppo di righe
Poiché un indice columnstore analizza una tabella eseguendo la scansione di segmenti di colonna di singoli gruppi di righe, accrescendo al massimo il numero di righe in ogni gruppo di righe le prestazioni delle query migliorano. Quando i gruppi di righe hanno un numero elevato di righe, la compressione dei dati migliora, il che significa meno dati da leggere dal disco.

Per altre informazioni sui gruppi di righe, vedere [Descrizione degli indici columnstore](https://msdn.microsoft.com/library/gg492088.aspx).

## <a name="target-size-for-rowgroups"></a>Dimensioni di destinazione per i gruppi di righe
Per ottimizzare le prestazioni delle query, l'obiettivo è accrescere al massimo il numero di righe per ogni gruppo di righe in un indice columnstore. Un gruppo di righe può avere un massimo di 1.048.576 righe. È accettabile non avere il numero massimo di righe per gruppo di righe. Gli indici columnstore ottengono buone prestazioni quando i gruppi di righe hanno almeno 100.000 righe.

## <a name="rowgroups-can-get-trimmed-during-compression"></a>I gruppi di righe possono essere tagliati durante la compressione

Durante un caricamento bulk o la ricompilazione di un indice columnstore, talvolta non è disponibile memoria sufficiente per comprimere tutte le righe per ogni gruppo di righe. Quando la memoria disponibile è scarsa, gli indici columnstore riducono le dimensioni del gruppo di righe in modo da consentire la compressione nel columnstore. 

Quando la memoria è insufficiente per la compressione di almeno 10.000 righe in ogni gruppo di righe, SQL Data Warehouse genera un errore.

Per altre informazioni sul caricamento bulk, vedere [Caricamento bulk in un indice columnstore cluster](https://msdn.microsoft.com/library/dn935008.aspx#Bulk ).

## <a name="how-to-monitor-rowgroup-quality"></a>Come monitorare la qualità di un gruppo di righe

La DMV pdw_nodes_db_column_store_row_group_physical_stats ([DM db_column_store_row_group_physical_stats](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-db-column-store-row-group-physical-stats-transact-sql) contiene la definizione della vista corrispondenti database SQL in SQL Data Warehouse) che espone informazioni utili ad esempio il numero di righe nel rowgroup e il motivo per cui è stato tagliato. Per effettuare una query su questa DMV allo scopo di ottenere informazioni sul trimming di un gruppo di righe, è possibile creare la vista seguente.

```sql
create view dbo.vCS_rg_physical_stats
as 
with cte
as
(
select   tb.[name]                    AS [logical_table_name]
,        rg.[row_group_id]            AS [row_group_id]
,        rg.[state]                   AS [state]
,        rg.[state_desc]              AS [state_desc]
,        rg.[total_rows]              AS [total_rows]
,        rg.[trim_reason_desc]        AS trim_reason_desc
,        mp.[physical_name]           AS physical_name
FROM    sys.[schemas] sm
JOIN    sys.[tables] tb               ON  sm.[schema_id]          = tb.[schema_id]                             
JOIN    sys.[pdw_table_mappings] mp   ON  tb.[object_id]          = mp.[object_id]
JOIN    sys.[pdw_nodes_tables] nt     ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[dm_pdw_nodes_db_column_store_row_group_physical_stats] rg      ON  rg.[object_id]     = nt.[object_id]
                                                                            AND rg.[pdw_node_id]   = nt.[pdw_node_id]
                                        AND rg.[distribution_id]    = nt.[distribution_id]                                          
)
select *
from cte;
```

trim_reason_desc specifica se il gruppo di righe è stato tagliato (trim_reason_desc = NO_TRIM indica che il gruppo di righe è di qualità ottimale e non è stato tagliato). I motivi seguenti indicano che il gruppo di righe è stato tagliato prematuramente:
- BULKLOAD: questo motivo viene usato se il batch di righe in ingresso per il caricamento è inferiore a 1 milione di righe. Il motore creerà gruppi di righe compressi se devono essere inserite più di 100.000 righe (a differenza dell'inserimento nell'archivio differenziale), ma imposta il motivo per cui il gruppo è stato tagliato su BULKLOAD. In questo scenario, provare ad aumentare il carico di batch in modo da includere altre righe. Rivalutare inoltre lo schema di partizionamento per verificare che non sia troppo granulare e che i gruppi di righe non possano quindi estendersi oltre i limiti della partizione.
- MEMORY_LIMITATION: per creare gruppi di righe con 1 milione di righe, il motore richiede una certa quantità di memoria di lavoro. Se la memoria disponibile nella sessione di caricamento è inferiore alla memoria di lavoro necessaria, i gruppi di righe vengono tagliati in modo prematuro. Le sezioni seguenti illustrano come stimare la memoria necessaria e allocare memoria aggiuntiva.
- DICTIONARY_SIZE: questo motivo indica che il gruppo di righe è stato tagliato perché era presente almeno una colonna di stringhe con stringhe wide e/o a cardinalità elevata. Le dimensioni del dizionario sono limitate a 16 MB di memoria e, al raggiungimento di questo limite, il gruppo di righe viene compresso. Se si verifica questa situazione, valutare l'opportunità di isolare la colonna problematica in una tabella separata.

## <a name="how-to-estimate-memory-requirements"></a>Come stimare i requisiti di memoria

<!--
To view an estimate of the memory requirements to compress a rowgroup of maximum size into a columnstore index, download and run the view [dbo.vCS_mon_mem_grant](). This view shows the size of the memory grant that a rowgroup requires for compression in to the columnstore.
-->

La memoria massima necessaria per comprimere un gruppo di righe è circa

- 72 MB +
- \#righe \* \#colonne \* 8 byte +
- \#righe \*\#colonne stringa breve \* 32 byte +
- \#colonne stringa lunga \* 16 MB per il dizionario di compressione

dove le colonne stringa breve usano tipi di dati stringa < = 32 byte e le colonne stringa lunga usano tipi di dati stringa > 32 byte.

Le stringhe lunghe vengono compresse con un metodo di compressione progettato per la compressione del testo. Questo metodo di compressione usa un *dizionario* per archiviare i modelli di testo. La dimensione massima di un oggetto dictionary è 16 MB. Esiste un solo dizionario per ogni colonna stringa lunga nel gruppo di righe.

Per un'analisi approfondita dei requisiti di memoria columnstore, vedere il video [Azure SQL Data Warehouse scaling: configuration and guidance](https://channel9.msdn.com/Events/Ignite/2016/BRK3291) (Scalabilità di Azure SQL Data Warehouse: configurazione e linee guida).

## <a name="ways-to-reduce-memory-requirements"></a>Modi per ridurre i requisiti di memoria

Usare le tecniche seguenti per ridurre i requisiti di memoria per la compressione dei gruppi di righe in indici columnstore.

### <a name="use-fewer-columns"></a>Usare meno colonne
Se possibile, progettare la tabella con meno colonne. Quando un gruppo di righe viene compresso nel columnstore, l'indice columnstore comprime ogni segmento di colonna separatamente. Pertanto i requisiti di memoria per comprimere un gruppo di righe aumentano con l'aumentare del numero di colonne.


### <a name="use-fewer-string-columns"></a>Usare meno colonne di stringhe
Le colonne di dati di tipo stringa richiedono una quantità di memoria maggiore rispetto ai tipi di dati numerici. Per ridurre i requisiti di memoria, prendere in considerazione la rimozione delle colonne di tipo stringa dalle tabelle di dati e il loro inserimento in tabelle di dimensioni minori.

Requisiti di memoria aggiuntivi per la compressione di stringhe:

- I tipi di dati stringa fino a 32 caratteri possono richiedere 32 byte aggiuntivi per valore.
- I tipi di dati stringa con più di 32 caratteri vengono compressi mediante metodi di dizionario.  Ogni colonna del gruppo di righe può richiedere fino a 16 MB aggiuntivi per creare il dizionario.

### <a name="avoid-over-partitioning"></a>Evitare il partizionamento eccessivo

Gli indici columnstore creano uno o più gruppi di righe per partizione. In SQL Data Warehouse il numero di partizioni aumenta rapidamente perché i dati vengono distribuiti e ogni distribuzione è partizionata. Se la tabella ha troppe partizioni, potrebbero esserci abbastanza righe per riempire i gruppi di righe. La mancanza di righe non crea richiesta di memoria durante la compressione, ma alcuni gruppi di righe soffriranno di scarse prestazioni delle query columnstore.

Un altro motivo per evitare l'eccessivo partizionamento è che il caricamento di righe in un indice columnstore in una tabella partizionata comporta un sovraccarico della memoria. Durante il caricamento molte partizioni potrebbero ricevere le righe in ingresso, che vengono mantenute in memoria finché ogni partizione dispone di un numero di righe sufficiente da comprimere. Con un numero eccessivo di partizioni vengono create richieste di memoria aggiuntive.

### <a name="simplify-the-load-query"></a>Semplificare la query di caricamento

Il database condivide la concessione di memoria per una query tra tutti gli operatori della query. Quando una query di caricamento contiene ordinamenti complessi e join, la memoria disponibile per la compressione è ridotta.

Progettare la query di caricamento concentrandosi solo sul caricamento. Se è necessario eseguire trasformazioni sui dati, eseguirle separatamente dalla query di caricamento. Ad esempio, collocare temporaneamente i dati in una tabella heap, eseguire le trasformazioni e quindi caricare la tabella di gestione temporanea nell'indice columnstore. È possibile anche caricare prima i dati e poi usare il sistema MPP per trasformarli.

### <a name="adjust-maxdop"></a>Regolare MAXDOP

Ogni distribuzione comprime i gruppi di righe nel columnstore in parallelo quando c'è più di un core CPU disponibile per distribuzione. Il parallelismo richiede risorse di memoria aggiuntive che possono portare a richieste di memoria pesanti e al taglio del gruppo di righe.

Per ridurre le richieste di memoria, è possibile usare l'hint di query MAXDOP per forzare l'esecuzione seriale dell'operazione di caricamento in ogni distribuzione.

```sql
CREATE TABLE MyFactSalesQuota
WITH (DISTRIBUTION = ROUND_ROBIN)
AS SELECT * FROM FactSalesQuota
OPTION (MAXDOP 1);
```

## <a name="ways-to-allocate-more-memory"></a>Modi per allocare altra memoria

La dimensione delle DWU e la classe della risorsa utente insieme determinano la quantità di memoria disponibile per una query dell'utente. Per aumentare la concessione di memoria per una query di caricamento, è possibile aumentare il numero di DWU o aumentare la classe risorsa.

- Per aumentare le DWU, vedere [Ridimensionare le prestazioni](quickstart-scale-compute-portal.md)
- Per cambiare la classe risorsa per una query, vedere [Esempio di modifica della classe di risorse di un utente](resource-classes-for-workload-management.md#change-a-users-resource-class).

## <a name="next-steps"></a>Passaggi successivi

Per trovare altri modi con cui migliorare le prestazioni in SQL Data Warehouse, vedere la sezione [Panoramica](sql-data-warehouse-overview-manage-user-queries.md) relativa alle prestazioni.

