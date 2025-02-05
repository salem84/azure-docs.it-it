---
title: Configurare e accedere ai log di query lente nel database di Azure per MySQL dalla portale di Azure
description: Questo articolo descrive come configurare e accedere ai log lenti in database di Azure per MySQL dalla portale di Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 09/30/2019
ms.openlocfilehash: 7eeeb729973e484e9acb26f3ac8cc42693f72eea
ms.sourcegitcommit: 15e3bfbde9d0d7ad00b5d186867ec933c60cebe6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/03/2019
ms.locfileid: "71841543"
---
# <a name="configure-and-access-slow-query-logs-from-the-azure-portal"></a>Configurare e accedere ai log di query lente dal portale di Azure

È possibile configurare, elencare e scaricare i [log di query lente di database di Azure per MySQL](concepts-server-logs.md) dalla portale di Azure.

## <a name="prerequisites"></a>Prerequisiti
Per la procedura descritta in questo articolo è necessario disporre del [database di Azure per il server MySQL](quickstart-create-mysql-server-database-using-azure-portal.md).

## <a name="configure-logging"></a>Configurare la registrazione
Configurare l'accesso al log di query lente MySQL. 

1. Accedere al [portale di Azure](https://portal.azure.com/).

2. Selezionare il server del Database di Azure per MySQL.

3. Nella sezione **monitoraggio** della barra laterale selezionare log del **Server**. 
   ![Screenshot delle opzioni dei log del server @ no__t-1

4. Per visualizzare i parametri del server, selezionare **fare clic qui per abilitare i log e configurare i parametri del log**.

5. Modificare i parametri che è necessario rettificare. Tutte le modifiche apportate in questa sessione vengono evidenziate in viola. 

   Dopo aver modificato i parametri, selezionare **Salva**. In alternativa, è possibile annullare le modifiche.

   ![Screenshot delle opzioni dei parametri del server](./media/howto-configure-server-logs-in-portal/3-save-discard.png)

Dalla pagina **parametri server** è possibile tornare all'elenco dei log chiudendo la pagina.

## <a name="view-list-and-download-logs"></a>Visualizzare l'elenco e scaricare i log
Una volta iniziata la registrazione, è possibile visualizzare un elenco di log di query lente disponibili e scaricare i singoli file di log.

1. Aprire il Portale di Azure.

2. Selezionare il server del Database di Azure per MySQL.

3. Nella sezione **monitoraggio** della barra laterale selezionare log del **Server**. La pagina Visualizza un elenco dei file di log.

   ![Screenshot della pagina log del server con elenco dei log evidenziato](./media/howto-configure-server-logs-in-portal/4-server-logs-list.png)

   > [!TIP]
   > La convenzione di denominazione del log è **mysql-slow-< nome server>-yyyymmddhh.log**. La data e l'ora utilizzate nel nome del file corrispondono all'ora in cui è stato emesso il log. I file di log vengono ruotati ogni 24 ore o 7,5 GB, a seconda di quale si verifichi per primo. 

4. Se necessario, usare la casella di ricerca per restringere rapidamente a un log specifico, in base alla data e all'ora. La ricerca viene eseguita in base al nome del log.

5. Per scaricare i singoli file di log, selezionare l'icona della freccia rivolta verso il basso accanto a ogni file di log nella riga della tabella.

   ![Screenshot della pagina log del server con icona con la freccia giù evidenziata](./media/howto-configure-server-logs-in-portal/5-download.png)

## <a name="set-up-diagnostic-logs"></a>Configurare i log di diagnostica

1. Nella sezione **monitoraggio** della barra laterale selezionare impostazioni di **diagnostica** > **Aggiungi impostazioni di diagnostica**.

   ![Screenshot delle opzioni delle impostazioni di diagnostica](./media/howto-configure-server-logs-in-portal/add-diagnostic-setting.png)

1. Specificare un nome per l'impostazione di diagnostica.

1. Specificare i sink di dati per l'invio dei log delle query lente (account di archiviazione, Hub eventi o area di lavoro Log Analytics).

1. Selezionare **MySqlSlowLogs** come tipo di log.
![Screenshot delle impostazioni di diagnostica opzioni di configurazione @ no__t-1

1. Dopo aver configurato i sink di dati per inviare tramite pipe i log delle query lente a, selezionare **Salva**.
@no__t 0Screenshot delle opzioni di configurazione delle impostazioni di diagnostica con Save evidenziato @ no__t-1

1. Per accedere ai log di query lente, esplorarli nei sink di dati configurati. Possono essere necessari fino a 10 minuti per la visualizzazione dei log.

## <a name="next-steps"></a>Passaggi successivi
- Vedere [accedere ai log di query lente nell'interfaccia della](howto-configure-server-logs-in-cli.md) riga di comando per informazioni su come scaricare i log di query lente a livello di codice
- Altre informazioni sui [log di query lente](concepts-server-logs.md) nel database di Azure per MySQL.
- Per altre informazioni sulle definizioni di parametri e sulla registrazione di MySQL, vedere la documentazione di MySQL nei [log](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html).