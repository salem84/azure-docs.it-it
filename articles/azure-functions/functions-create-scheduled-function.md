---
title: Creare una funzione che viene eseguita in una pianificazione | Microsoft Docs
description: Informazioni su come creare una funzione in Azure eseguita in base a una pianificazione definita.
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.assetid: ba50ee47-58e0-4972-b67b-828f2dc48701
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/28/2018
ms.author: glenga
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: 13aa1f8cd643d579734cc603bbc81519a690fe59
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70085852"
---
# <a name="create-a-function-in-azure-that-is-triggered-by-a-timer"></a>Creare una funzione in Azure attivata da un timer

Informazioni su come usare Funzioni di Azure per creare una funzione [senza server](https://azure.microsoft.com/solutions/serverless/) eseguita in base a una pianificazione definita.

![Creare un'app per le funzioni nel portale di Azure](./media/functions-create-scheduled-function/function-app-in-portal-editor.png)

## <a name="prerequisites"></a>Prerequisiti

Per completare questa esercitazione:

+ Se non si ha una sottoscrizione di Azure, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.

## <a name="create-an-azure-function-app"></a>Creare un'app per le funzioni di Azure

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

![App per le funzioni creata correttamente.](./media/functions-create-first-azure-function/function-app-create-success.png)

Si creerà ora una funzione nella nuova app per le funzioni.

<a name="create-function"></a>

## <a name="create-a-timer-triggered-function"></a>Creare una funzione attivata da un timer

1. Espandere l'app per le funzioni e fare clic sul pulsante **+** accanto a **Funzioni**. Se questa è la prima funzione nell'app per le funzioni, selezionare **Nel portale** e quindi **Continua**. In caso contrario, andare direttamente al passaggio 3.

   ![Pagina della guida introduttiva di Funzioni nel portale di Azure](./media/functions-create-scheduled-function/function-app-quickstart-choose-portal.png)

2. Scegliere **Altri modelli** e quindi **Termina e visualizza i modelli**.

    ![Selezione di altri modelli nella guida introduttiva di Funzioni](./media/functions-create-scheduled-function/add-first-function.png)

3. Nel campo di ricerca digitare `timer` e configurare il nuovo trigger con le impostazioni indicate nella tabella sotto l'immagine.

    ![Creare una funzione attivata da un timer nel portale di Azure.](./media/functions-create-scheduled-function/functions-create-timer-trigger-2.png)

    | Impostazione | Valore consigliato | Descrizione |
    |---|---|---|
    | **Nome** | Predefinito | Definisce il nome della funzione attivata dal timer. |
    | **Pianificare** | 0 \*/1 \* \* \* \* | [Espressione CRON](functions-bindings-timer.md#ncrontab-expressions) a sei campi che pianifica la funzione in modo che venga eseguita ogni minuto. |

4. Fare clic su **Create**(Crea). Viene creata una funzione nel linguaggio scelto che verrà eseguita ogni minuto.

5. Verificare l'esecuzione visualizzando le informazioni di traccia scritte nei log.

    ![Visualizzatore log di Funzioni nel portale di Azure.](./media/functions-create-scheduled-function/functions-timer-trigger-view-logs2.png)

È ora possibile modificare la pianificazione della funzione in modo che venga eseguita ogni ora invece che ogni minuto.

## <a name="update-the-timer-schedule"></a>Aggiornare la pianificazione del timer

1. Espandere la funzione e fare clic su **Integrazione**. È a questo punto che si definiscono i binding di input e di output e si imposta la pianificazione. 

2. Nel campo **Pianificazione** immettere il nuovo valore `0 0 */1 * * *` per la pianificazione oraria e quindi fare clic su **Salva**.  

![Pianificazione del timer di aggiornamento di Funzioni nel portale di Azure.](./media/functions-create-scheduled-function/functions-timer-trigger-change-schedule.png)

È ora disponibile una funzione che viene eseguita ogni ora. 

## <a name="clean-up-resources"></a>Pulire le risorse

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Passaggi successivi

È stata creata una funzione eseguita in base a una pianificazione. Per altre informazioni sui trigger timer, vedere [Pianificare l'esecuzione di codice con Funzioni di Azure](functions-bindings-timer.md).

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]
