---
title: Riavviare il Database di Azure per PostgreSQL - singolo Server tramite la CLI di Azure
description: Questo articolo descrive come è possibile riavviare un Database di Azure per PostgreSQL - singolo Server tramite la CLI di Azure
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 0a7cd815724fcebd6311860576e620eb9273523b
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "65068971"
---
# <a name="restart-azure-database-for-postgresql---single-server-using-the-azure-cli"></a>Riavviare il Database di Azure per PostgreSQL - singolo Server tramite la CLI di Azure
In questo argomento viene descritta la procedura per riavviare un server di Database di Azure per PostgreSQL. Potrebbe essere necessario riavviare il server per motivi di manutenzione, causando così una breve interruzione del servizio mentre il server esegue l'operazione.

Il riavvio del server verrà bloccato se il servizio è occupato. È ad esempio possibile che il servizio stia elaborando un'operazione richiesta in precedenza, come il ridimensionamento di vCore.
 
Il tempo necessario per completare un riavvio dipende dal processo di ripristino di PostgreSQL. Per ridurre i tempi di riavvio, è consigliabile ridurre al minimo l'attività in corso nel server prima del riavvio.

## <a name="prerequisites"></a>Prerequisiti
Per completare questa guida, è necessario:
- [Database di Azure per il server PostgreSQL](quickstart-create-server-up-azure-cli.md)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Questa guida dettagliata richiede l'uso dell'interfaccia della riga di comando di Azure 2.0 o versioni successive. Per verificare la versione, al prompt dei comandi dell'interfaccia della riga di comando di Azure immettere `az --version`. Per installare o eseguire l'aggiornamento, vedere [Installare l'interfaccia della riga di comando di Azure]( /cli/azure/install-azure-cli).


## <a name="restart-the-server"></a>Riavviare il server

Riavviare il server con il comando seguente:

```azurecli-interactive
az postgres server restart --name mydemoserver --resource-group myresourcegroup
```

## <a name="next-steps"></a>Passaggi successivi

Informazioni su [come impostare i parametri nel Database di Azure per PostgreSQL](howto-configure-server-parameters-using-cli.md)