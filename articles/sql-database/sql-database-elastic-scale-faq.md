---
title: Domande frequenti sulla scalabilità elastica in Azure SQL | Documentazione Microsoft
description: Domande frequenti sulla scalabilità elastica del database SQL di Azure.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 2b101aebd048b94ac95e1dba0f6504446d6d6803
ms.sourcegitcommit: 7c4de3e22b8e9d71c579f31cbfcea9f22d43721a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/26/2019
ms.locfileid: "68568439"
---
# <a name="elastic-database-tools-frequently-asked-questions-faq"></a>Domande frequenti sugli strumenti di database elastici

## <a name="if-i-have-a-single-tenant-per-shard-and-no-sharding-key-how-do-i-populate-the-sharding-key-for-the-schema-info"></a>In presenza di un tenant singolo per partizione senza chiave di partizionamento, in che modo è possibile popolare la chiave di partizionamento per le informazioni sullo schema?

L'oggetto di informazioni sullo schema viene usato solo in scenari di divisione e unione. Se un'applicazione è intrinsecamente single-tenant, non richiede lo strumento di divisione e unione e quindi non è necessario popolare l'oggetto di informazioni sullo schema.

## <a name="ive-provisioned-a-database-and-i-already-have-a-shard-map-manager-how-do-i-register-this-new-database-as-a-shard"></a>Quando è stato eseguito il provisioning di un database e si dispone già di un Gestore mappe partizioni, come è possibile registrare il nuovo database come partizione?

Vedere [Aggiunta di una partizione a un'applicazione usando la libreria dei client di database elastici](sql-database-elastic-scale-add-a-shard.md).

## <a name="how-much-do-elastic-database-tools-cost"></a>Quanto costano gli strumenti di database elastici?

L'uso della libreria di database elastico è gratuito. È previsto un costo solo per i database SQL di Azure che vengono usati per le partizioni e il Gestore mappe partizioni, nonché per i ruoli di lavoro/Web di cui viene eseguito il provisioning per lo strumento di divisione e unione.

## <a name="why-are-my-credentials-not-working-when-i-add-a-shard-from-a-different-server"></a>Perché le credenziali personali non funzionano quando si aggiunge una partizione da un server diverso?

Non usare credenziali in formato "User ID=username@servername", ma usare semplicemente "User ID=nomeutente".  Verificare inoltre che il “nome utente” di accesso disponga di autorizzazioni sulla partizione.

## <a name="do-i-need-to-create-a-shard-map-manager-and-populate-shards-every-time-i-start-my-applications"></a>È necessario creare un Gestore mappe partizioni e popolare le partizioni ogni volta che si avviano le applicazioni?

No. La creazione del Gestore mappe partizioni, ad esempio [ShardMapManagerFactory.CreateSqlShardMapManager](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager), è un'operazione che si esegue una volta sola.  L'applicazione deve usare la chiamata [ShardMapManagerFactory.TryGetSqlShardMapManager()](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager) all'avvio dell'applicazione.  È supportata una sola chiamata di questo tipo per dominio di applicazione.

## <a name="i-have-questions-about-using-elastic-database-tools-how-do-i-get-them-answered"></a>In che modo è possibile ottenere risposte alle domande sugli strumenti di database elastici?

È possibile partecipare al [forum sul database SQL](https://social.msdn.microsoft.com/forums/azure/home?forum=ssdsgetstarted).

## <a name="when-i-get-a-database-connection-using-a-sharding-key-i-can-still-query-data-for-other-sharding-keys-on-the-same-shard--is-this-by-design"></a>Quando si ottiene una connessione di database usando una chiave di partizionamento orizzontale, è possibile eseguire query sui dati per altre chiavi di partizionamento orizzontale sulla stessa partizione.  Si tratta di un comportamento previsto da progettazione?

Le API di Scalabilità elastica offrono una connessione al database corretto per la propria chiave di partizionamento orizzontale, ma non forniscono filtri per le chiavi di partizionamento orizzontale.  Aggiungere la clausola **WHERE** alla query per limitare l'ambito alla chiave di partizionamento orizzontale fornita, se necessario.

## <a name="can-i-use-a-different-sql-database-edition-for-each-shard-in-my-shard-set"></a>È possibile usare un'edizione del database SQL diversa per ogni partizione nel set di partizioni?

Sì, una partizione è un database a sé, per cui è possibile che una partizione sia un'edizione Premium mentre un'altra è un'edizione Standard. Inoltre, l'edizione della partizione può essere aumentata o ridotta più volte durante il ciclo di vita della partizione.

## <a name="does-the-split-merge-tool-provision-or-delete-a-database-during-a-split-or-merge-operation"></a>Lo strumento di divisione e unione esegue il provisioning di un database o lo elimina durante un'operazione di divisione o unione?

No. Per le operazioni di **divisione** è necessario che il database di destinazione sia dotato dello schema appropriato e sia registrato nel Gestore mappe partizioni.  Per le operazioni di **unione** è necessario eliminare la partizione dal Gestore mappe partizioni e quindi eliminare il database.

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]