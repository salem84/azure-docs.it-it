---
title: Configurare l'autenticazione tramite account Microsoft - Servizio app di Azure
description: Informazioni su come configurare l'autenticazione dell'account Microsoft per l'app del servizio app.
author: mattchenderson
services: app-service
documentationcenter: ''
manager: syntaxc4
editor: ''
ms.assetid: ffbc6064-edf6-474d-971c-695598fd08bf
ms.service: app-service
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/08/2019
ms.author: mahender
ms.custom: seodec18
ms.openlocfilehash: 70af534e6bcd0039dbc602a5ebc3fc35fb145e79
ms.sourcegitcommit: 42748f80351b336b7a5b6335786096da49febf6a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/09/2019
ms.locfileid: "72176943"
---
# <a name="configure-your-app-service-app-to-use-microsoft-account-login"></a>Configurare l'app del servizio app per l'uso dell'account di accesso Microsoft

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Questo argomento descrive come configurare il servizio app di Azure per usare l'account Microsoft come provider di autenticazione. 

## <a name="register-microsoft-account"></a>Registrare l'app con l'account Microsoft

1. Passare a [**registrazioni app**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) nel portale di Azure. Se necessario, accedere con l'account Microsoft.
1. Selezionare **nuova registrazione**, quindi immettere un nome di applicazione.
1. In **URI di reindirizzamento**selezionare **Web**, quindi immettere `https://<app-domain-name>/.auth/login/microsoftaccount/callback supply the endpoint for your application`. Sostituire *\<app-Domain-name >* con il nome di dominio dell'app.  Ad esempio `https://contoso.azurewebsites.net/.auth/login/microsoftaccount/callback`. Assicurarsi di usare lo schema HTTPS nell'URL.

1. Selezionare **Registra**.
1. Copiare l' **ID applicazione (client)** . Sarà necessario più avanti.
1. Nel riquadro sinistro selezionare **certificati & secrets** > **New client Secret**. Immettere una descrizione, selezionare la durata della validità e selezionare **Aggiungi**.
1. Copiare il valore visualizzato nella pagina **certificati & segreti** . Una volta lasciata, la pagina non verrà più visualizzata.

    > [!IMPORTANT]
    > La password è una credenziale di sicurezza importante. Non condividerla con altri e non distribuirla all'interno di un'applicazione client.

## <a name="secrets"></a>Aggiungere le informazioni dell'account Microsoft all'applicazione del servizio app

1. Passare all'applicazione nella [portale di Azure].
1. Selezionare **impostazioni** > **autenticazione/autorizzazione**e assicurarsi che **l'autenticazione del servizio app** sia **attiva**.
1. In **provider di autenticazione**selezionare **account Microsoft**. Incollare l'ID dell'applicazione (client) e il segreto client ottenuti in precedenza. Abilitare gli ambiti necessari per l'applicazione.
1. Scegliere **OK**.

   Il servizio app fornisce l'autenticazione ma non limita l'accesso autorizzato al contenuto del sito e alle API. È necessario autorizzare gli utenti nel codice dell'app.

1. Opzionale Per limitare l'accesso agli utenti account Microsoft, impostare l' **azione da eseguire quando la richiesta non viene autenticata** per l'accesso **con l'account Microsoft**. Quando si imposta questa funzionalità, l'app richiede che tutte le richieste vengano autenticate. Reindirizza inoltre tutte le richieste non autenticate a account Microsoft per l'autenticazione.

   > [!CAUTION]
   > La limitazione dell'accesso in questo modo si applica a tutte le chiamate all'app, che potrebbero non essere utili per le app con un home page disponibile pubblicamente, come in molte applicazioni a singola pagina. Per queste applicazioni, **consentire le richieste anonime (nessuna azione)** potrebbe essere preferibile, in modo che l'app avvii manualmente l'autenticazione. Per altre informazioni, vedere [flusso di autenticazione](overview-authentication-authorization.md#authentication-flow).

1. Selezionare **Salva**.

È ora possibile usare l'account Microsoft per l'autenticazione nell'app.

## <a name="related-content"> </a>Passaggi successivi

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- URLs. -->

[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Portale di Azure]: https://portal.azure.com/
