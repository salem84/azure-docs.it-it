---
title: Inserire l'applicazione nella raccolta di applicazioni di Azure Active Directory | Microsoft Docs
description: Informazioni su come inserire un'applicazione che supporta l'accesso Single Sign-On nella raccolta di app di Azure Active Directory
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/16/2019
ms.author: ryanwi
ms.reviewer: elisol, bryanla
ms.custom: aaddev, seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: c77657101f5cd8a117b2163386f6d551b7985458
ms.sourcegitcommit: 0576bcb894031eb9e7ddb919e241e2e3c42f291d
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/15/2019
ms.locfileid: "72374082"
---
# <a name="list-your-application-in-the-azure-active-directory-application-gallery"></a>Inserire l'applicazione nella raccolta di applicazioni di Azure Active Directory

Questo articolo illustra come elencare un'applicazione nella raccolta di applicazioni Azure Active Directory (Azure AD), implementare Single Sign-On (SSO) e gestire l'elenco.

## <a name="what-is-the-azure-ad-application-gallery"></a>Definizione della raccolta di applicazioni di Azure AD

- Esperienza Single Sign-On ottimale per i clienti.
- Configurazione minima e semplice dell'applicazione.
- Ricerca rapida per l'individuazione dell'applicazione nella raccolta.
- Uso di questa integrazione per tutti i clienti con account Free, Basic e Premium di Azure AD.
- Esercitazione dettagliata per la configurazione per i clienti reciproci.
- I clienti che usano il sistema per la gestione delle identità tra domini ([scim](https://techcommunity.microsoft.com/t5/Identity-Standards-Blog/Provisioning-with-SCIM-getting-started/ba-p/880010)) possono usare il provisioning per la stessa app.

## <a name="prerequisites"></a>Prerequisiti

- Per le applicazioni federate (Open ID e SAML/WS-Fed), l'applicazione deve supportare il modello SaaS (software-as-a-Service) per essere elencato nella raccolta di app Azure AD. Le applicazioni della raccolta Enterprise devono supportare più configurazioni dei clienti e non un cliente specifico.
- Per Open ID Connect, l'applicazione deve essere multi-tenant e il [Framework di consenso Azure ad](consent-framework.md) deve essere implementato correttamente per l'applicazione. L'utente può inviare la richiesta di accesso a un endpoint comune in modo che qualsiasi cliente possa fornire il consenso all'applicazione. È possibile controllare l'accesso utente in base all'ID del tenant e all'UPN dell'utente ricevuti nel token.
- Per SAML 2.0/WS-Fed, l'applicazione deve avere la possibilità di eseguire l'integrazione SSO SAML/WS-Fed in modalità SP o IDP. Verificare che questa funzionalità funzioni correttamente prima di inviare la richiesta.
- Per l'accesso Single Sign-on basato su password, assicurarsi che l'applicazione supporti l'autenticazione basata su form, Single Sign-On in modo che l'insieme di credenziali delle password possa essere eseguito come previsto.
- È necessario un account permanente per i test con almeno due utenti registrati.

## <a name="submit-the-request-in-the-portal"></a>Inviare la richiesta nel portale

Dopo aver verificato che l'integrazione dell'applicazione funzioni con Azure AD, inviare la richiesta di accesso nel [portale di rete dell'applicazione](https://microsoft.sharepoint.com/teams/apponboarding/Apps). Se si dispone di un account Office 365, usarlo per accedere a questo portale. In caso contrario, usare la account Microsoft, ad esempio Outlook o Hotmail, per accedere.

Se viene visualizzata la pagina seguente dopo l'accesso, contattare il [team di integrazione di Azure ad SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>). Fornire l'account di posta elettronica che si vuole usare per l'invio della richiesta. Il team di Azure AD aggiungerà l'account nel portale di rete delle applicazioni Microsoft.

![Messaggio di richiesta di accesso nel portale di SharePoint](./media/howto-app-gallery-listing/errorimage.png)

Dopo aver aggiunto l'account, è possibile accedere al portale di rete delle applicazioni Microsoft.

Se viene visualizzata la pagina seguente dopo l'accesso, fornire una giustificazione aziendale per la necessità di accesso nella casella di testo. Selezionare quindi **Richiedi accesso**.

  ![Casella relativa alla motivazione aziendale nel portale di SharePoint](./media/howto-app-gallery-listing/accessrequest.png)

Il team esamina i dettagli e fornisce l'accesso di conseguenza. Dopo che la richiesta è stata approvata, è possibile accedere al portale e inviare la richiesta selezionando il riquadro **Invia richiesta (ISV)** nella Home page.

![Riquadro Invia richiesta (ISV) in home page](./media/howto-app-gallery-listing/homepage.png)

> [!NOTE]
> In caso di problemi con l'accesso, contattare il [team di integrazione di Azure ad SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="implement-sso-by-using-the-federation-protocol"></a>Implementare SSO tramite il protocollo federativo

Per inserire un'applicazione nella raccolta di app di Azure AD, è innanzitutto necessario implementare uno dei protocolli di federazione seguenti supportati da Azure AD. È anche necessario accettare i termini e le condizioni della raccolta di applicazioni Azure AD. Leggere i termini e le condizioni della raccolta di applicazioni Azure AD nel [sito Web](https://azure.microsoft.com/support/legal/active-directory-app-gallery-terms/).

- **OpenID Connect**: per integrare l'applicazione con Azure ad usando il protocollo Open ID Connect, seguire le istruzioni per gli [sviluppatori](v1-authentication-scenarios.md).

    ![Elenco di un'applicazione OpenID Connect nella raccolta](./media/howto-app-gallery-listing/openid.png)

    * Per aggiungere l'applicazione all'elenco nella raccolta usando OpenID Connect, selezionare **OpenID connect & OAuth 2,0** come illustrato.
    * In caso di problemi con l'accesso, contattare il [team di integrazione di Azure ad SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

- **Saml 2,0** o **WS-Fed**: se l'app supporta SAML 2,0, è possibile integrarla direttamente con un tenant di Azure ad seguendo le [istruzioni per l'aggiunta di un'applicazione personalizzata](../active-directory-saas-custom-apps.md).

  ![Elenco di un'applicazione SAML 2,0 o WS-Fed nella raccolta](./media/howto-app-gallery-listing/saml.png)

  * Se si vuole aggiungere l'applicazione all'elenco nella raccolta con **saml 2,0** o **WS-Fed**, selezionare **SAML 2.0/WS-Fed** come illustrato.
  * In caso di problemi con l'accesso, contattare il [team di integrazione di Azure ad SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="implement-sso-by-using-the-password-sso"></a>Implementare SSO tramite la password SSO

Creare un'applicazione Web con una pagina di accesso HTML per configurare l'[accesso Single Sign-On basato su password](../manage-apps/what-is-single-sign-on.md). L'SSO basato su password, definito anche insieme di credenziali delle password, consente di gestire l'accesso degli utenti e le password per le applicazioni Web che non supportano la federazione delle identità. È utile anche per gli scenari in cui più utenti devono condividere un singolo account, ad esempio gli account di app di social media dell'organizzazione.

![Elenco di un'applicazione con password SSO nella raccolta](./media/howto-app-gallery-listing/passwordsso.png)

* Per aggiungere l'applicazione all'elenco della raccolta usando l'accesso SSO basato su password, selezionare **SSO con password** come illustrato.
* In caso di problemi con l'accesso, contattare il [team di integrazione di Azure ad SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="request-for-user-provisioning"></a>Richiesta di provisioning utenti

Per richiedere il provisioning degli utenti, seguire la procedura illustrata nell'immagine seguente.

   ![Richiesta di provisioning utenti](./media/howto-app-gallery-listing/user-provisioning.png)

## <a name="update-or-remove-an-existing-listing"></a>Aggiornare o rimuovere un elenco esistente

Per aggiornare o rimuovere un'applicazione esistente nella raccolta Azure AD app, è necessario prima inviare la richiesta nel [portale di rete dell'applicazione](https://microsoft.sharepoint.com/teams/apponboarding/Apps). Se si dispone di un account Office 365, usarlo per accedere a questo portale. In caso contrario, usare la account Microsoft, ad esempio Outlook o Hotmail, per accedere.

- Selezionare l'opzione appropriata, come illustrato nella figura seguente.

    ![Elenco di un'applicazione SAML nella raccolta](./media/howto-app-gallery-listing/updateorremove.png)

    * Per aggiornare un'applicazione esistente, selezionare l'opzione appropriata in base ai requisiti.
    * Per rimuovere un'applicazione esistente dalla raccolta di app Azure AD, selezionare **Rimuovi l'elenco di applicazioni dalla raccolta**.
    * In caso di problemi con l'accesso, contattare il [team di integrazione di Azure ad SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

## <a name="list-requests-by-customers"></a>Elenca richieste per clienti

I clienti possono inviare una richiesta per elencare un'applicazione selezionando **richieste di app da clienti** > **inviare una nuova richiesta**.

![Mostra il riquadro app richieste dal cliente](./media/howto-app-gallery-listing/customer-submit-request.png)

Ecco il flusso delle applicazioni richieste dai clienti.

![Mostra il flusso delle app richieste dal cliente](./media/howto-app-gallery-listing/customer-request.png)

## <a name="timelines"></a>Tempistica

La sequenza temporale per il processo di visualizzazione di un'applicazione SAML 2,0 o WS-Fed nella raccolta è da 7 a 10 giorni lavorativi.

  ![Sequenza temporale per l'elenco di un'applicazione SAML nella raccolta](./media/howto-app-gallery-listing/timeline.png)

La sequenza temporale per il processo di visualizzazione di un'applicazione OpenID Connect nella raccolta è di 2 a 5 giorni lavorativi.

  ![Sequenza temporale per l'elenco di un'applicazione OpenID Connect nella raccolta](./media/howto-app-gallery-listing/timeline2.png)

## <a name="escalations"></a>Escalation

Per le escalation, inviare un messaggio di posta elettronica al [team di integrazione di Azure ad SSO](mailto:SaaSApplicationIntegrations@service.microsoft.com) all'SaaSApplicationIntegrations@service.microsoft.com e risponderemo il prima possibile.
