---
title: Accedere ai log del server in Database di Azure per MariaDB usando l'interfaccia della riga di comando di Azure
description: Questo articolo descrive come accedere ai log del server in Database di Azure per MariaDB usando l'utilità dell'interfaccia della riga di comando di Azure.
author: rachel-msft
ms.author: raagyema
ms.service: mariadb
ms.devlang: azurecli
ms.topic: conceptual
ms.date: 06/12/2019
ms.openlocfilehash: ffc724ef5133ee25643a966d2b6d8448a4c3a920
ms.sourcegitcommit: 11265f4ff9f8e727a0cbf2af20a8057f5923ccda
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/08/2019
ms.locfileid: "72023607"
---
# <a name="configure-and-access-server-logs-by-using-azure-cli"></a>Configurare e accedere ai log del server usando l'interfaccia della riga di comando di Azure
È possibile scaricare i log del server di Database di Azure per MariaDB usando l'interfaccia della riga di comando di Azure, l'utilità della riga di comando di Azure.

## <a name="prerequisites"></a>Prerequisiti
Per proseguire con questa guida, si richiedono:
- [Database di Azure per il server MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md)
- L'[interfaccia della riga di comando di Azure](/cli/azure/install-azure-cli) o Azure Cloud Shell nel browser

## <a name="configure-logging-for-azure-database-for-mariadb"></a>Configurare la registrazione per Database di Azure per MariaDB
Per configurare il server per l'accesso al log delle query lente di MariaDB, seguire la procedura seguente:
1. Attivare la registrazione impostando il parametro **slow\_query\_log** su ON.
2. Regolare gli altri parametri, ad esempio **long\_query\_time** e **log\_slow\_admin\_statements**.

Per informazioni su come impostare il valore di questi parametri tramite l'interfaccia della riga di comando di Azure, vedere [Personalizzare i parametri di configurazione server usando l'interfaccia della riga di comando di Azure](howto-configure-server-parameters-cli.md).

Il comando dell'interfaccia della riga di comando seguente, ad esempio, attiva il log delle query lente, imposta la durata di una query prolungata su 10 secondi e quindi disattiva la registrazione dell'istruzione per un amministratore lento. Infine elenca le opzioni di configurazione da verificare.
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mariadb server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mariadb server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mariadb server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mariadb-server"></a>Elencare i log del server per Database di Azure per MariaDB
Per elencare i file di log di query lente disponibili per il server, eseguire il comando [AZ mariadb server-logs list](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-list) .

È possibile elencare i file di log per il server **mydemoserver.mariadb.database.azure.com** nel gruppo di risorse **myresourcegroup**. Quindi indirizzare l'elenco dei file di log a un file di testo denominato **log\_files\_list.txt**.
```azurecli-interactive
az mariadb server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Scaricare i log dal server
Tramite il comando [az mariadb server-logs download](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-download) è possibile scaricare i singoli file di log per il server.

Usare l'esempio seguente per scaricare il file di log specifico per il server **mydemoserver.mariadb.database.azure.com** nel gruppo di risorse **myresourcegroup** nell'ambiente locale.
```azurecli-interactive
az mariadb server-logs download --name mysql-slow-mydemoserver-2018110800.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Passaggi successivi
- Informazioni sui [log di query lente nel database di Azure per MariaDB](concepts-server-logs.md).
