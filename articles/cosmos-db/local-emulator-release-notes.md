---
title: 'Emulatore Azure Cosmos: download e note sulla versione'
description: Leggere le note sulla versione dell'emulatore Azure Cosmos e scaricarla.
ms.service: cosmos-db
ms.topic: tutorial
author: markjbrown
ms.author: mjbrown
ms.date: 06/20/2019
ms.openlocfilehash: 03163d75681bee7434e2b7576a678b1094d47b94
ms.sourcegitcommit: c2e7595a2966e84dc10afb9a22b74400c4b500ed
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/05/2019
ms.locfileid: "71973703"
---
# <a name="azure-cosmos-emulator---release-notes-and-download-information"></a>Emulatore Azure Cosmos - Note sulla versione e informazioni di download

Questo articolo include le note sulla versione dell'emulatore Azure Cosmos con un elenco di aggiornamenti delle funzionalità apportati in ogni versione. Inoltre, include la versione più recente dell'emulatore disponibile per il download.

## <a name="download"></a>Download

| | |
|---------|---------|
|**Download MSI**|[Area download Microsoft](https://aka.ms/cosmosdb-emulator)|
|**Introduzione**|[Sviluppare in locale con l'emulatore Azure Cosmos](local-emulator.md)|

## <a name="release-notes"></a>Note sulla versione

### <a name="246"></a>2.4.6

- Questa versione offre le stesse funzionalità incluse nella versione di luglio 2019 del servizio Azure Cosmos, con le eccezioni indicate in [Sviluppare in locale con l'emulatore Azure Cosmos](local-emulator.md). Vengono inoltre corretti diversi bug correlati all'arresto dell'emulatore quando viene richiamato tramite la riga di comando e gli override degli indirizzi IP interni per i client SDK che usano la connettività in modalità diretta.

### <a name="243"></a>2.4.3

- Avvio del servizio MongoDB disattivato per impostazione predefinita. Per impostazione predefinita è abilitato solo l'endpoint SQL. L'utente deve avviare l'endpoint manualmente usando l'opzione "/EnableMongoDbEndpoint" della riga di comando dell'emulatore. Adesso è come tutti gli altri endpoint di servizio, ad esempio Gremlin, Cassandra e Table.
- Risolto un bug nell'emulatore quando si inizia con "/AllowNetworkAccess" in cui gli endpoint Gremlin, Cassandra e Table non gestiscono correttamente le richieste provenienti da client esterni.
- Aggiungere porte di connessione diretta alle impostazioni delle regole del Firewall.

### <a name="240"></a>2.4.0

- Risolto un problema con l'emulatore relativo al mancato avvio se nel computer host sono presenti app di monitoraggio di rete, come Pulse Client.
