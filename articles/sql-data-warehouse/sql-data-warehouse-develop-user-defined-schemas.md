---
title: Uso di schemi definiti dall'utente in SQL Data Warehouse | Microsoft Docs
description: Suggerimenti per l'uso di schemi T-SQL definiti dall'utente in Azure SQL Data Warehouse per lo sviluppo di soluzioni.
services: sql-data-warehouse
author: XiaoyuMSFT
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: development
ms.date: 04/17/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: e0ae00e0fca5ed4c6fba04444e5c50424462d297
ms.sourcegitcommit: 75a56915dce1c538dc7a921beb4a5305e79d3c7a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/24/2019
ms.locfileid: "68479575"
---
# <a name="using-user-defined-schemas-in-sql-data-warehouse"></a>Uso di schemi definiti dall'utente in SQL Data Warehouse
Suggerimenti per l'uso di schemi T-SQL definiti dall'utente in Azure SQL Data Warehouse per lo sviluppo di soluzioni.

## <a name="schemas-for-application-boundaries"></a>Schemi per i limiti dell'applicazione

I data warehouse tradizionali spesso usano database separati per creare i limiti dell'applicazione in base a carico di lavoro, dominio o di sicurezza. Ad esempio, un data warehouse di SQL Server tradizionale potrebbe includere un database di gestione temporanea, un database del data warehouse e alcuni database del data mart. In questa topologia ogni database funziona come carico di lavoro e limite di sicurezza nell'architettura.

Al contrario, SQL Data Warehouse esegue tutto il carico di lavoro del data warehouse all'interno di un database. I join tra database non sono consentiti. SQL Data Warehouse prevede pertanto che tutte le tabelle usate dal data warehouse siano archiviate all'interno di un database.

> [!NOTE]
> SQL Data Warehouse non supporta query tra database di alcun tipo. Di conseguenza, le implementazioni di data warehouse che usano questo modello dovranno essere modificate.
> 
> 

## <a name="recommendations"></a>Consigli
Si tratta di indicazioni per il consolidamento di carichi di lavoro, sicurezza, dominio e i limiti funzionali mediante l'uso di schemi definiti dall'utente.

1. Usare un database SQL Data Warehouse per eseguire l'intero carico di lavoro del data warehouse.
2. Consolidare l'ambiente data warehouse esistente per l'uso di un solo database di SQL Data Warehouse.
3. Sfruttare gli **schemi definiti dall'utente** per fornire il limite implementato in precedenza tramite database.

Se gli schemi definiti dall'utente non sono stati usati in precedenza, si avrà uno slate pulito. Usare semplicemente il nome del database precedente come base per gli schemi definiti dall'utente nel database di SQL Data Warehouse.

Se gli schemi sono già stati usati, sono disponibili alcune opzioni:

1. Rimuovere i nomi degli schemi legacy e ricominciare dall'inizio.
2. Mantenere i nomi degli schemi legacy premettendo il nome dello schema legacy al nome della tabella.
3. Mantenere i nomi degli schemi legacy implementando viste sulla tabella in uno schema aggiuntivo per ricreare la struttura dello schema precedente.

> [!NOTE]
> A prima vista, l'opzione 3 può sembrare quella più interessante. È tuttavia importante osservare i dettagli. Le viste sono di sola lettura in SQL Data Warehouse. Qualsiasi modifica di dati o tabelle dovrà essere eseguita sulla tabella di base. L'opzione 3 introduce anche un livello di viste nel sistema. È consigliabile valutare attentamente questo aspetto se si usano già viste nell'architettura.
> 
> 

### <a name="examples"></a>Esempi:
Implementare schemi definiti dall'utente in base ai nomi di database.

```sql
CREATE SCHEMA [stg]; -- stg previously database name for staging database
GO
CREATE SCHEMA [edw]; -- edw previously database name for the data warehouse
GO
CREATE TABLE [stg].[customer] -- create staging tables in the stg schema
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[customer] -- create data warehouse tables in the edw schema
(       CustKey BIGINT NOT NULL
,       ...
);
```

Mantenere nomi di schemi legacy premettendoli al nome della tabella. Usare schemi per il limite del carico di lavoro.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- edw defines the data warehouse boundary
GO
CREATE TABLE [stg].[dim_customer] --pre-pend the old schema name to the table and create in the staging boundary
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[dim_customer] --pre-pend the old schema name to the table and create in the data warehouse boundary
(       CustKey BIGINT NOT NULL
,       ...
);
```

Mantenere i nomi di schemi legacy usando viste.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- stg defines the data warehouse boundary
GO
CREATE SCHEMA [dim]; -- edw defines the legacy schema name boundary
GO
CREATE TABLE [stg].[customer] -- create the base staging tables in the staging boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE TABLE [edw].[customer] -- create the base data warehouse tables in the data warehouse boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE VIEW [dim].[customer] -- create a view in the legacy schema name boundary for presentation consistency purposes only
AS
SELECT  CustKey
,       ...
FROM    [edw].customer
;
```

> [!NOTE]
> Qualsiasi modifica nella strategia relativa agli schemi richiede una revisione del modello di sicurezza per il database. In molti casi è possibile semplificare il modello di sicurezza assegnando le autorizzazioni a livello di schema. Se sono necessarie autorizzazioni più granulari, è possibile utilizzare i ruoli del database.
> 
> 

## <a name="next-steps"></a>Passaggi successivi
Per altri suggerimenti sullo sviluppo, vedere la [panoramica dello sviluppo](sql-data-warehouse-overview-develop.md).

