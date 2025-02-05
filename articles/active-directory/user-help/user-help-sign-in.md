---
title: Eseguire l'accesso con la verifica in due passaggi o le informazioni di sicurezza-Azure Active Directory | Microsoft Docs
description: Informazioni su come eseguire l'accesso usando i vari metodi di verifica dell'identità disponibili nelle informazioni di sicurezza.
services: active-directory
author: eross-msft
manager: daveba
ms.reviewer: librown
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: conceptual
ms.date: 07/18/2019
ms.author: lizross
ms.custom: user-help
ms.collection: M365-identity-device-management
ms.openlocfilehash: b20ea1131ceda0527ed35d1a1082d05f25da6bac
ms.sourcegitcommit: 04ec7b5fa7a92a4eb72fca6c6cb617be35d30d0c
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/22/2019
ms.locfileid: "68382324"
---
# <a name="sign-in-using-two-step-verification-or-security-info"></a>Eseguire l'accesso usando la verifica in due passaggi o le informazioni di sicurezza

Dopo aver impostato la verifica in due passaggi o le informazioni di sicurezza, sarà possibile accedere all'account usando il metodo di autenticazione specificato.

> [!Note]
> Se si usa ancora l'esperienza di verifica in due passaggi, è necessario configurare i metodi di autenticazione seguendo le istruzioni riportate nell'articolo [Configurare l'account per la verifica in due passaggi](multi-factor-authentication-end-user-first-time.md).
>
> Se l'amministratore ha attivato l'esperienza per le informazioni di sicurezza, è necessario impostare i metodi di autenticazione usando questi articoli di procedure dettagliate:<ul><li>[Configurare le informazioni di sicurezza per l'uso di un'app di autenticazione](security-info-setup-auth-app.md)</li><li>[Configurare le informazioni di sicurezza per l'uso di SMS](security-info-setup-text-msg.md)</li><li>[Configurare le informazioni di sicurezza per l'uso di una chiamata telefonica](security-info-setup-phone-number.md)</li><li>[Configurare le informazioni di sicurezza per l'uso di una chiave di sicurezza](security-info-setup-security-key.md)</li></ul>

## <a name="sign-in-using-an-authenticator-app-notification-on-your-mobile-device"></a>Eseguire l'accesso usando una notifica dell'app di autenticazione nel dispositivo mobile

1. Accedere all'account con il nome utente e la password.

2. Selezionare **Approva** dalla notifica di approvazione inviata al dispositivo mobile.

## <a name="sign-in-using-an-authenticator-app-code-on-your-mobile-device"></a>Eseguire l'accesso usando un codice dell'app di autenticazione nel dispositivo mobile

1. Accedere all'account con il nome utente e la password.

2. Aprire l'app di autenticazione e digitare il codice generato in modo casuale per l'account nella casella **Immettere il codice**.

## <a name="sign-in-using-your-phone-number"></a>Eseguire l'accesso con il numero di telefono

1. Accedere all'account con il nome utente e la password.

2. Rispondere alla chiamata e seguire le istruzioni.

## <a name="sign-in-using-a-text-message"></a>Eseguire l'accesso con un SMS

1. Accedere all'account con il nome utente e la password.

2. Aprire il messaggio SMS e digitare il codice contenuto nel messaggio nella casella **Immettere il codice**.

## <a name="sign-in-using-a-security-key-at-the-lock-screen"></a>Accedere con una chiave di sicurezza nella schermata di blocco

1. Dopo aver registrato la chiave di sicurezza, selezionare l'immagine della chiave di sicurezza dalla schermata di blocco di Windows 10.

2. Inserire la chiave di sicurezza nella porta USB del dispositivo e accedere a Windows usando il PIN della chiave di sicurezza.

    ![Accesso alla chiave di sicurezza nella schermata di blocco di Windows 10](./media/security-info/security-info-windows-10-lock-screen-security-key.png)

## <a name="sign-in-using-a-security-key-and-the-microsoft-edge-browser"></a>Accedere con una chiave di sicurezza e il browser Microsoft Edge

1. Dopo aver registrato la chiave di sicurezza, aprire il browser Microsoft Edge.

2. Quando viene richiesto di eseguire l'accesso, inserire la chiave di sicurezza nella porta USB del dispositivo e accedere a Windows con il PIN della chiave di sicurezza.

    ![Accesso alla chiave di sicurezza tramite il browser Microsoft Edge](./media/security-info/security-info-edge-security-key.png)

    >[!NOTE]
    >Per informazioni sull'accesso con l'app Microsoft Authenticator, vedere l'articolo [accedere agli account usando l'app Microsoft Authenticator](user-help-auth-app-sign-in.md).

## <a name="sign-in-using-another-verification-method"></a>Accedere con un altro metodo di verifica

Se per qualche motivo non si è in grado di usare il metodo di accesso principale, è possibile usare un altro metodo di verifica impostato in precedenza.

1. Accedere al proprio account normalmente, quindi scegliere il collegamento **Accedi a un altro modo** nella pagina di **verifica in due passaggi** .

    ![Modificare il metodo di verifica dell'accesso](media/security-info/two-factor-auth-signin-another-way.png)

    >[!Note]
    >Se non viene visualizzato il collegamento **Accedi a un altro modo** , significa che non sono stati configurati altri metodi di verifica e che sarà necessario contattare l'amministratore per ottenere assistenza per l'accesso all'account. Quando l'amministratore consente di eseguire l'accesso, assicurarsi di aggiungere ulteriori metodi di verifica. Per altre informazioni sui metodi di verifica, vedere l'articolo [Gestire le impostazioni per la verifica in due passaggi](multi-factor-authentication-end-user-manage-settings.md).
    >
    >Se viene visualizzato il collegamento **Accedi a un altro modo** , ma non vengono ancora visualizzati altri metodi di verifica, è necessario contattare l'amministratore per richiedere assistenza per l'accesso al proprio account.

2. Scegliere il metodo di verifica alternativo e continuare il processo di verifica in due passaggi.

3. Dopo aver eseguito l'accesso al proprio account, è possibile aggiornare i metodi di verifica (se necessario). Per altre informazioni su come aggiungere o modificare i propri metodi, vedere l'articolo [Gestire le impostazioni per la verifica in due passaggi](multi-factor-authentication-end-user-manage-settings.md).

## <a name="next-steps"></a>Passaggi successivi

- Altre informazioni sulle informazioni di sicurezza sono disponibili nell'articolo [Panoramica delle informazioni di sicurezza (anteprima)](user-help-security-info-overview.md).

- Informazioni sulla verifica in due passaggi sono disponibili nell'articolo [Panoramica della verifica in due passaggi](user-help-two-step-verification-overview.md).

- Se la password è stata persa o dimenticata, è possibile reimpostarla dal [portale per la reimpostazione della password](https://passwordreset.microsoftonline.com/)

- Ottenere informazioni e suggerimenti per la risoluzione dei problemi di accesso nell'articolo [Quando non riesci ad accedere al tuo account Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).
