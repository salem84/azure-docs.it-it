---
title: Gestione delle credenziali nella libreria client dei database elastici | Documentazione Microsoft
description: Come impostare il livello corretto di credenziali, da amministratore a sola lettura, per le app dei database elastici.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/03/2019
ms.openlocfilehash: d89e83092775828016c2c47a96164319f5474c1e
ms.sourcegitcommit: 7c4de3e22b8e9d71c579f31cbfcea9f22d43721a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/26/2019
ms.locfileid: "68568419"
---
# <a name="credentials-used-to-access-the-elastic-database-client-library"></a>Credenziali usate per accedere alla libreria client dei database elastici

La [libreria client dei database elastici](sql-database-elastic-database-client-library.md) usa tre diversi tipi di credenziali per accedere al [gestore delle mappe partizioni](sql-database-elastic-scale-shard-map-management.md). A seconda delle esigenze, usare le credenziali con il minimo livello possibile di accesso.

* **Credenziali di gestione**: per la creazione o la modifica di un gestore di mappa di partizionamento. (Vedere il [glossario](sql-database-elastic-scale-glossary.md).)
* **Credenziali di accesso**: per accedere a un gestore di mappa di partizione esistente per ottenere informazioni sulle partizioni.
* **Credenziali di connessione**: per connettersi alle partizioni.

Vedere anche [Gestione di database e account di accesso in database SQL di Azure](sql-database-manage-logins.md).

## <a name="about-management-credentials"></a>Informazioni sulle credenziali di gestione

Le credenziali di gestione vengono usate per creare un oggetto **ShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager), [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager)) per applicazioni che modificano le mappe partizioni. Ad esempio, vedere [Aggiunta di una partizione usando gli strumenti di database elastici](sql-database-elastic-scale-add-a-shard.md) e [Routing dipendente dai dati](sql-database-elastic-scale-data-dependent-routing.md). L'utente della libreria client dei database elastici crea gli utenti e gli account di accesso SQL e garantisce che ad ognuno vengano concesse autorizzazioni di lettura/scrittura per il database della mappa globale partizioni, nonché per tutti i database delle partizioni. Queste credenziali vengono usate per la gestione della mappa globale partizioni e delle mappe locali partizioni quando si apportano modifiche alla mappa partizioni. Ad esempio, usare le credenziali di gestione per creare l'oggetto di gestione delle mappe partizioni usando **GetSqlShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.getsqlshardmapmanager), [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager)):

```java
// Obtain a shard map manager.
ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(smmAdminConnectionString,ShardMapManagerLoadPolicy.Lazy);
```

La variabile **smmAdminConnectionString** è una stringa di connessione che contiene le credenziali di gestione. L'ID utente e la password forniscono accesso in lettura/scrittura sia al database della mappa partizioni che alle singole partizioni. La stringa di connessione alla gestione include anche il nome del server e il nome del database per identificare il database della mappa globale partizioni. La seguente è una stringa di connessione tipica usata a tale scopo:

```java
"Server=<yourserver>.database.windows.net;Database=<yourdatabase>;User ID=<yourmgmtusername>;Password=<yourmgmtpassword>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;”
```

Non usare valori nel formato "username@server", ma usare solo il valore "username".  Infatti, le credenziali devono funzionare sia per il database del gestore mappe partizioni sia per le singole partizioni, che possono essere in server diversi.

## <a name="access-credentials"></a>Credenziali di accesso

Quando si crea un gestore mappe partizioni in un'applicazione che non amministra mappe partizioni, usare credenziali a cui sono assegnate autorizzazioni di sola lettura per la mappa globale partizioni. Le informazioni recuperate dalla mappa globale partizioni con queste credenziali vengono usate per il [routing dipendente dai dati](sql-database-elastic-scale-data-dependent-routing.md) e per popolare la cache della mappa di partizionamento orizzontale nel client. Le credenziali vengono fornite in base allo stesso modello di chiamata a **GetSqlShardMapManager**:

```java
// Obtain shard map manager.
ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(smmReadOnlyConnectionString, ShardMapManagerLoadPolicy.Lazy);  
```

Si noti l'uso di **smmReadOnlyConnectionString** per rispecchiare le diverse credenziali per questo accesso usate da utenti **non amministratori**. Queste credenziali non devono infatti fornire autorizzazioni di scrittura per la mappa globale partizioni.

## <a name="connection-credentials"></a>Credenziali di connessione

Sono inoltre necessarie credenziali aggiuntive quando si usa il metodo **OpenConnectionForKey**  ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper.listshardmapper.openconnectionforkey), [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey)) per accedere a una partizione associata a una chiave di partizionamento orizzontale. Queste credenziali devono fornire autorizzazioni per l'accesso di sola lettura alle tabelle della mappa locale partizioni che risiedono nella partizione. Tale comportamento è necessario per eseguire la convalida della connessione per il routing dipendente dai dati nella partizione. Questo frammento di codice consente l'accesso ai dati nel contesto di routing dipendente dai dati:

```csharp
using (SqlConnection conn = rangeMap.OpenConnectionForKey<int>(targetWarehouse, smmUserConnectionString, ConnectionOptions.Validate))
```

In questo esempio **smmUserConnectionString** contiene la stringa di connessione per le credenziali utente. Questa è invece una stringa di connessione tipica per le credenziali utente per il database SQL di Azure:

```java
"User ID=<yourusername>; Password=<youruserpassword>; Trusted_Connection=False; Encrypt=True; Connection Timeout=30;”  
```

Come con le credenziali di amministratore, non usare valori nel formato "username@server". Usare invece solo "username".  Si noti inoltre che la stringa di connessione non contiene un nome di server e un nome di database, in quando la chiamata a **OpenConnectionForKey** reindirizzerà automaticamente la connessione alla partizione corretta in base alla chiave. Di conseguenza, non vengono forniti il nome del database e il nome del server.

## <a name="see-also"></a>Vedere anche

[Gestione di database e account di accesso in database SQL di Azure](sql-database-manage-logins.md)

[Protezione del Database SQL](sql-database-security-overview.md)

[processi di database elastico](elastic-jobs-overview.md)

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]
