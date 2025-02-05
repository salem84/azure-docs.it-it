---
title: Scheda di riferimento rapido per Azure SQL Data Warehouse | Microsoft Docs
description: Collegamenti e procedure consigliate per compilare rapidamente soluzioni Azure SQL Data Warehouse.
services: sql-data-warehouse
author: mlee3gsd
manager: craigg
ms.service: sql-data-warehouse
ms.topic: overview
ms.subservice: design
ms.date: 08/23/2019
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: 1bbb0148e6f4be2afc777960afcda9c727328206
ms.sourcegitcommit: 532335f703ac7f6e1d2cc1b155c69fc258816ede
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 08/30/2019
ms.locfileid: "70195070"
---
# <a name="cheat-sheet-for-azure-sql-data-warehouse"></a>Scheda di riferimento rapido per Azure SQL Data Warehouse
Questa scheda di riferimento fornisce suggerimenti utili e procedure consigliate per creare rapidamente soluzioni Azure SQL Data Warehouse. Prima di iniziare, ottenere altre informazioni dettagliate su ogni passaggio leggendo [Azure SQL Data Warehouse Workload Patterns and Anti-Patterns (Modelli e anti-modelli del carico di lavoro di Azure SQL Data Warehouse)](https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-workload-patterns-and-anti-patterns), che spiega cos'è e cosa non è SQL Data Warehouse.

Il grafico seguente illustra il processo di ideazione di un data warehouse:

![Schema]

## <a name="queries-and-operations-across-tables"></a>Query e operazioni tra tabelle

Se si conoscono in anticipo le operazioni e le query principali da eseguire nel data warehouse, è possibile definire la priorità dell'architettura del data warehouse per queste operazioni. Queste query e operazioni possono includere:
* Join di una o due tabelle dei fatti con tabelle delle dimensioni, applicazione di un filtro alla tabella combinata e aggiunta dei risultati in un data mart.
* Aggiornamenti di lieve o notevole entità alle vendite dei fatti.
* Accodamento di soli dati alle tabelle.

Conoscere in anticipo il tipo di operazioni consente di ottimizzare la progettazione delle tabelle.

## <a name="data-migration"></a>Migrazione dei dati

Prima di tutto, caricare i dati in [Azure Data Lake Store](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-store) o in archiviazione BLOB di Azure. Quindi usare PolyBase per caricare i dati in SQL Data Warehouse in una tabella di staging. Usare la configurazione seguente:

| Progettazione | Recommendation |
|:--- |:--- |
| Distribuzione | Round robin |
| Indicizzazione | Heap |
| Partizionamento | Nessuna |
| Classe di risorse | largerc o xlargerc |

Sono disponibili altre informazioni sulla [migrazione dei dati], sul [caricamento dei dati] e sul [processo di estrazione, caricamento e trasformazione (ELT - Extract, Load, and Transform)](https://docs.microsoft.com/azure/sql-data-warehouse/design-elt-data-loading). 

## <a name="distributed-or-replicated-tables"></a>Tabelle distribuite o replicate

A seconda delle proprietà della tabella usare le strategie seguenti:

| Type | Ideale per...| Prestare attenzione se...|
|:--- |:--- |:--- |
| Replicata | • Tabelle delle dimensioni ridotte in uno schema star con meno di 2 GB di spazio di archiviazione dopo la compressione (compressione ~5x) |• Molte transazioni di scrittura sono sulla tabella (ad esempio, insert, upsert, delete, update)<br></br>• Modifica frequente del provisioning delle unità Data Warehouse (DWU)<br></br>• Vengono usate solo 2-3 colonne ma la tabella contiene molte colonne<br></br>• Viene indicizzata una tabella replicata |
| Round robin (predefinita) | • Tabella di staging/temporanea<br></br> • Nessuna chiave di join ovvia o colonna candidata ottimale |• Rallentamento delle prestazioni a causa dello spostamento dei dati |
| Hash | • Tabelle dei fatti<br></br>• Tabelle delle dimensioni estese |• La chiave di distribuzione non può essere aggiornata |

**Suggerimenti:**
* Iniziare con round robin ma mirare a una strategia di distribuzione hash per sfruttare i vantaggi di un'architettura parallela elevata.
* Assicurarsi che le chiavi hash comuni abbiano lo stesso formato di dati.
* Non eseguire la distribuzione sul formato varchar.
* Le tabelle delle dimensioni con chiave hash comune in una tabella dei fatti con operazioni di join frequenti possono essere distribuite tramite hash.
* Usare *[sys.dm_pdw_nodes_db_partition_stats]* per analizzare eventuali asimmetrie nei dati.
* Usare *[sys.dm_pdw_request_steps]* per analizzare gli spostamenti dei dati protetti da query e monitorare il tempo richiesto dalle operazioni di trasmissione e casuali. È consigliabile rivedere la strategia di distribuzione.

Sono disponibili altre informazioni sulle [tabelle replicate] e le [tabelle distribuite].

## <a name="index-your-table"></a>Indicizzare la tabella

L'indicizzazione è utile per leggere rapidamente le tabelle. È disponibile un set univoco di tecnologie che è possibile usare in base alle proprie esigenze:

| Type | Ideale per... | Prestare attenzione se...|
|:--- |:--- |:--- |
| Heap | • Tabella di staging/temporanea<br></br>• Tabelle ridotte con ricerche limitate |• Qualsiasi ricerca analizza la tabella completa |
| Indice cluster | • Tabelle con meno di 100 milioni di righe<br></br>• Tabelle estese (più di 100 milioni di righe) con solo 1-2 colonne usate di frequente |• Usato su una tabella replicata<br></br>• Query complesse che includono più operazioni di Join, Group By<br></br>• Aggiornamenti nelle colonne indicizzate: richiede memoria |
| Indice columnstore cluster (CCI) (predefinito) | • Tabelle estese (più di 100 milioni di righe) | • Usato su una tabella replicata<br></br>• Operazioni di aggiornamento estremamente elevate nella tabella<br></br>• Creazione di un numero eccessivo di partizioni: i gruppi di righe non si estendono su partizioni e nodi di distribuzione diversi |

**Suggerimenti:**
* Su un indice cluster si potrebbe voler aggiungere un indice non cluster a una colonna usata di frequente per il filtro. 
* Prestare attenzione a come viene gestita la memoria in una tabella con indice columnstore cluster. Quando si caricano i dati, è opportuno che l'utente (o la query) tragga vantaggio da una classe di risorse di grandi dimensioni. Assicurarsi di evitare il trimming e la creazione di molti gruppi di righe compressi di piccole dimensioni.
* In Gen2, le tabelle con indice ColumnStore cluster vengono memorizzati nella cache in locale sui nodi di calcolo per ottimizzare le prestazioni.
* Per l'indice columnstore cluster può verificarsi un rallentamento delle prestazioni a causa della scarsa compressione dei gruppi di righe. In questo caso, ricompilare o riorganizzare l'indice columnstore cluster. Sono necessarie almeno 100.000 righe per gruppi di righe compressi. La soluzione ideale è 1 milione di righe in un gruppo di righe.
* In base alle dimensioni e alla frequenza di caricamento incrementale, è possibile procedere all'automatizzazione quando si riorganizzano o si ricompilano gli indici. È sempre utile fare pulizia.
* Adottare una strategia per il trimming di un gruppo di righe. Quanto devono essere grandi i gruppi di righe aperti? Quanti dati si prevedere di caricare nei prossimi giorni?

Sono disponibili altre informazioni sugli [indici].

## <a name="partitioning"></a>Partizionamento
È possibile partizionare la tabella in presenza di tabelle dei fatti estese (con oltre 1 miliardo di righe). Nel 99% dei casi la chiave di partizione deve essere basata sulla data. Prestare attenzione a non creare un numero eccessivo di partizioni, specialmente se si dispone di un indice columnstore cluster.

Con le tabelle di staging che richiedono ELT, è possibile trarre vantaggio dal partizionamento, che facilita la gestione del ciclo di vita dei dati.
Prestare attenzione a non creare un numero eccessivo di dati, in particolare in un indice columnstore cluster.

Sono disponibili altre informazioni sulle [partizioni].

## <a name="incremental-load"></a>Carico incrementale

Se si intende caricare i dati in modo incrementale, è necessario assicurarsi di allocare le classi di risorse di dimensioni maggiori al caricamento dei dati.  Questa operazione è particolarmente importante quando si esegue il caricamento in tabelle con indici columnstore cluster.  Per altri dettagli, vedere le [classi di risorse](https://docs.microsoft.com/azure/sql-data-warehouse/resource-classes-for-workload-management).  

È consigliabile usare PolyBase e ADF V2 per automatizzare le pipeline di ELT in SQL Data Warehouse.

Per un grande batch di aggiornamenti nei dati cronologici, prendere in considerazione l'uso di [CTAS](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-develop-ctas) per scrivere i dati da mantenere in una tabella, invece di usare INSERT, UPDATE e DELETE.

## <a name="maintain-statistics"></a>Gestire le statistiche
 Finché le statistiche automatiche non saranno disponibili a livello generale, SQL Data Warehouse richiede la manutenzione manuale delle statistiche. È importante aggiornare le statistiche quando vengono apportate modifiche *significative* ai dati. Ciò consente di ottimizzare i piani di query. Se si ritiene che la gestione di tutte le statistiche richieda troppo tempo, scegliere in modo più selettivo le colonne con le statistiche. 

È anche possibile definire la frequenza degli aggiornamenti. Potrebbe ad esempio essere consigliabile aggiornare le colonne di data, in cui potrebbero venire aggiunti nuovi valori su base giornaliera. Il massimo vantaggio è offerto dalle statistiche sulle colonne usate nei join, su quelle usate nella clausola WHERE e sulle colonne presenti in GROUP BY.

Sono disponibili altre informazioni sulle [statistiche].

## <a name="resource-class"></a>classe di risorse
SQL Data Warehouse usa i gruppi di risorse per allocare la memoria per le query. Se si necessita di maggiore memoria per migliorare la velocità di caricamento o delle query, è consigliabile allocare più classi di risorse. D'altra parte, l'uso di classi di risorse di dimensioni maggiori incide sulla concorrenza. È consigliabile prendere in considerazione questo aspetto prima di spostare tutti gli utenti in una classe di risorse di grandi dimensioni.

Se si nota che le query richiedono troppo tempo, verificare che gli utenti non vengano eseguiti in classi di risorse di grandi dimensioni. Le classi di risorse estese usano molti slot di concorrenza e possono comportare l'accodamento di altre query.

Infine, tramite Gen2 di SQL Data Warehouse, ogni classe di risorse ottiene 2,5 volte più memoria rispetto a Gen1.

Sono disponibili altre informazioni su come lavorare con [classi di risorse e concorrenza].

## <a name="lower-your-cost"></a>Ridurre i costi
Una funzionalità chiave di SQL Data Warehouse è la possibilità di [gestire risorse di calcolo](sql-data-warehouse-manage-compute-overview.md). È possibile sospendere Data Warehouse quando non è in uso, interrompendo così la fatturazione delle risorse di calcolo. È possibile ridimensionare le risorse per soddisfare le richieste di prestazioni. Per sospendere, usare il [portale di Azure](pause-and-resume-compute-portal.md) o [PowerShell](pause-and-resume-compute-powershell.md). Per ridimensionare, usare il [portale di Azure](quickstart-scale-compute-portal.md), [Powershell](quickstart-scale-compute-powershell.md), [T-SQL](quickstart-scale-compute-tsql.md) o un'[API REST](sql-data-warehouse-manage-compute-rest-api.md#scale-compute).

Procedere ora al ridimensionamento automatico in corrispondenza del momento desiderato con Funzioni di Azure:

<a href="https://ms.portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwTimerScaler%2Fazuredeploy.json" target="_blank">
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png"/>
</a>

## <a name="optimize-your-architecture-for-performance"></a>Ottimizzare l'architettura per le prestazioni

È consigliabile prendere in considerazione il database SQL e Azure Analysis Services in un'architettura hub-spoke. Questa soluzione può fornire l'isolamento del carico di lavoro tra gruppi di utenti diversi, usando anche alcune funzionalità di protezione avanzate del database SQL e di Azure Analysis Services. Anche questo è un modo per fornire concorrenza illimitata agli utenti.

Sono disponibili altre informazioni sulle [architetture tipiche che sfruttano i vantaggi di SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/20../../common-isv-application-patterns-using-azure-sql-data-warehouse/).

Distribuire con un clic del mouse gli spoke nei database SQL da SQL Data Warehouse:

<a href="https://ms.portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwSpokeDbTemplate%2Fazuredeploy.json" target="_blank">
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png"/>
</a>


<!--Image references-->
[Schema]:media/sql-data-warehouse-cheat-sheet/picture-flow.png

<!--Article references-->
[caricamento dei dati]:design-elt-data-loading.md
[deeper guidance]:guidance-for-loading-data.md
[indici]:sql-data-warehouse-tables-index.md
[partizioni]:sql-data-warehouse-tables-partition.md
[Statistiche]:sql-data-warehouse-tables-statistics.md
[classi di risorse e concorrenza]:resource-classes-for-workload-management.md
[tabelle replicate]:design-guidance-for-replicated-tables.md
[tabelle distribuite]:sql-data-warehouse-tables-distribute.md

<!--MSDN references-->


<!--Other Web references-->
[typical architectures that take advantage of SQL Data Warehouse]: https://blogs.msdn.microsoft.com/sqlcat/20../../common-isv-application-patterns-using-azure-sql-data-warehouse/
[is and is not]:https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-workload-patterns-and-anti-patterns/
[migrazione dei dati]: https://blogs.msdn.microsoft.com/sqlcat/20../../migrating-data-to-azure-sql-data-warehouse-in-practice/

[Azure Data Lake Storage]: ../data-factory/connector-azure-data-lake-store.md
[sys.dm_pdw_nodes_db_partition_stats]: /sql/relational-databases/system-dynamic-management-views/sys-dm-db-partition-stats-transact-sql
[sys.dm_pdw_request_steps]:/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql
