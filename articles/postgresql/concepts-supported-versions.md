---
title: Versioni supportate nel database di Azure per PostgreSQL-server singolo
description: Descrive le versioni supportate nel database di Azure per PostgreSQL-server singolo.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/06/2019
ms.custom: fasttrack-edit
ms.openlocfilehash: b77fd082be43b8cbdedf7cbe5875a8931eb0474a
ms.sourcegitcommit: bc3a153d79b7e398581d3bcfadbb7403551aa536
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/06/2019
ms.locfileid: "68837900"
---
# <a name="supported-postgresql-database-versions"></a>Versione supportate del database PostgreSQL
Microsoft mira a supportare le versioni n-2 del motore PostgreSQL nel database di Azure per PostgreSQL: singolo server. Le versioni sono la versione principale attuale in Azure (n) e le due versioni principali precedenti (-2).

Database di Azure per PostgreSQL supporta attualmente le versioni principali seguenti:

## <a name="postgresql-version-11"></a>PostgreSQL versione 11
La versione secondaria corrente è 11,4. Vedere la [documentazione di PostgreSQL](https://www.postgresql.org/docs/11/static/release-11-4.html) per ottenere altre informazioni sulle correzioni e sui miglioramenti apportati in questa versione secondaria.

## <a name="postgresql-version-10"></a>PostgreSQL versione 10
La versione secondaria corrente è 10,9. Vedere la [documentazione di PostgreSQL](https://www.postgresql.org/docs/10/static/release-10-9.html) per ottenere altre informazioni sulle correzioni e sui miglioramenti apportati in questa versione secondaria.

## <a name="postgresql-version-96"></a>PostgreSQL versione 9,6
La versione secondaria corrente è 9.6.14. Vedere la [documentazione di PostgreSQL](https://www.postgresql.org/docs/9.6/static/release-9-6-14.html) per ottenere altre informazioni sulle correzioni e sui miglioramenti apportati in questa versione secondaria.

## <a name="postgresql-version-95"></a>PostgreSQL versione 9,5
La versione secondaria corrente è 9.5.18. Vedere la [documentazione di PostgreSQL](https://www.postgresql.org/docs/9.5/static/release-9-5-18.html) per ottenere altre informazioni sulle correzioni e sui miglioramenti apportati in questa versione secondaria.

## <a name="managing-upgrades"></a>Gestione degli aggiornamenti
Database di Azure per PostgreSQL gestisce automaticamente gli aggiornamenti della versione secondaria. 

L'aggiornamento automatico della versione principale non è supportato. Ad esempio, non è disponibile un aggiornamento automatico da PostgreSQL 9,5 a PostgreSQL 9,6. Per eseguire l'aggiornamento alla versione principale successiva, creare un [dump del database ed](./howto-migrate-using-dump-and-restore.md) eseguire il ripristino in un server creato con la nuova versione del motore.

### <a name="version-syntax"></a>Sintassi della versione
Prima di PostgreSQL versione 10, i [criteri di controllo delle versioni di PostgreSQL](https://www.postgresql.org/support/versioning/) consideravano un aggiornamento della _versione principale_ come un aumento del primo _o_ del secondo numero. Ad esempio, 9,5 a 9,6 è stato considerato un aggiornamento della versione _principale_ . A partire dalla versione 10, solo una modifica nel primo numero viene considerata un aggiornamento della versione principale. Ad esempio, 10,0 a 10,1 è un aggiornamento della versione _secondaria_ . La versione da 10 a 11 è un aggiornamento della versione _principale_ .

## <a name="next-steps"></a>Passaggi successivi
Per informazioni sulle estensioni PostgreSQL supportate, vedere [il documento sulle estensioni](concepts-extensions.md).
