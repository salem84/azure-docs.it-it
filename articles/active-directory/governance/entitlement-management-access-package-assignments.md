---
title: Visualizzare, aggiungere e rimuovere assegnazioni per un pacchetto di accesso in Azure AD Management diritti (anteprima)-Azure Active Directory
description: Informazioni su come visualizzare, aggiungere e rimuovere assegnazioni per un pacchetto di accesso in Azure Active Directory gestione dei diritti (anteprima).
services: active-directory
documentationCenter: ''
author: msaburnley
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 10/15/2019
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 694c8866a69d8488511be1670ba3d3013a4a3423
ms.sourcegitcommit: bb65043d5e49b8af94bba0e96c36796987f5a2be
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/16/2019
ms.locfileid: "72392514"
---
# <a name="view-add-and-remove-assignments-for-an-access-package-in-azure-ad-entitlement-management-preview"></a>Visualizzare, aggiungere e rimuovere assegnazioni per un pacchetto di accesso in Azure AD gestione dei diritti (anteprima)

> [!IMPORTANT]
> Gestione entitlement di Azure Active Directory (Azure AD) è attualmente in anteprima pubblica.
> Questa versione di anteprima viene messa a disposizione senza contratto di servizio e non è consigliata per i carichi di lavoro di produzione. Alcune funzionalità potrebbero non essere supportate o potrebbero presentare funzionalità limitate.
> Per altre informazioni, vedere [Condizioni supplementari per l'utilizzo delle anteprime di Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

In Azure AD gestione dei diritti, è possibile visualizzare gli utenti a cui è stato assegnato l'accesso ai pacchetti, i relativi criteri e lo stato. Se un pacchetto di accesso dispone di un criterio appropriato, è anche possibile assegnare direttamente un utente a un pacchetto di accesso. Questo articolo descrive come visualizzare, aggiungere e rimuovere le assegnazioni per i pacchetti di accesso.

## <a name="view-who-has-an-assignment"></a>Visualizza chi ha un'assegnazione

**Ruolo prerequisiti:** Amministratore globale, Amministratore utenti, proprietario del catalogo o gestione pacchetti di accesso

1. Nel portale di Azure fare clic su **Azure Active Directory** e quindi su **Identity Governance**.

1. Nel menu a sinistra fare clic su **Access Packages** , quindi aprire il pacchetto Access.

1. Fare clic su **assegnazioni** per visualizzare un elenco di assegnazioni attive.

    ![Elenco di assegnazioni a un pacchetto di accesso](./media/entitlement-management-access-package-assignments/assignments-list.png)

1. Fare clic su un'assegnazione specifica per visualizzare altri dettagli.

1. Per visualizzare un elenco di assegnazioni in cui non è stato eseguito correttamente il provisioning di tutti i ruoli delle risorse, fare clic sullo stato del filtro e selezionare **distribuzione**.

    È possibile visualizzare ulteriori dettagli sugli errori di recapito individuando la richiesta corrispondente dell'utente nella pagina **richieste** .

1. Per visualizzare le assegnazioni scadute, fare clic sullo stato del filtro e selezionare **scaduto**.

1. Per scaricare un file CSV dell'elenco filtrato, fare clic su **download**.

## <a name="directly-assign-a-user"></a>Assegnare direttamente un utente

In alcuni casi, potrebbe essere necessario assegnare direttamente utenti specifici a un pacchetto di accesso in modo che gli utenti non debbano eseguire il processo di richiesta del pacchetto di accesso. Per assegnare direttamente gli utenti, il pacchetto di accesso deve disporre di un criterio che consenta l'assegnazione diretta dell'amministratore.

**Ruolo prerequisiti:** Amministratore globale, Amministratore utenti, proprietario del catalogo o gestione pacchetti di accesso

1. Nel portale di Azure fare clic su **Azure Active Directory** e quindi su **Identity Governance**.

1. Nel menu a sinistra fare clic su **Access Packages** , quindi aprire il pacchetto Access.

1. Nel menu a sinistra fare clic su **assegnazioni**.

1. Fare clic su **nuova assegnazione** per aprire Aggiungi utente per accedere al pacchetto.

    ![Assegnazioni-Aggiungi utente al pacchetto di accesso](./media/entitlement-management-access-package-assignments/assignments-add-user.png)

1. Fare clic su **Aggiungi utenti** per selezionare gli utenti a cui si desidera assegnare il pacchetto di accesso.

1. Nell'elenco **Seleziona criteri** selezionare un criterio con l'impostazione [Nessuna (solo assegnazione diretta amministratore)](entitlement-management-access-package-request-policy.md#none-administrator-direct-assignments-only) .

    Se il pacchetto di accesso non dispone di questo tipo di criteri, è possibile fare clic su **Crea nuovo criterio** per aggiungerne uno.

1. Consente di impostare la data e l'ora di inizio e fine dell'assegnazione degli utenti selezionati. Se non viene specificata una data di fine, verranno usate le impostazioni del ciclo di vita del criterio.

1. Facoltativamente, fornire una giustificazione per l'assegnazione diretta per la conservazione dei record.

1. Fare clic su **Aggiungi** per assegnare direttamente gli utenti selezionati al pacchetto di accesso.

    Dopo alcuni istanti, fare clic su **Aggiorna** per visualizzare gli utenti nell'elenco assegnazioni.

## <a name="next-steps"></a>Passaggi successivi

- [Modificare la richiesta e le impostazioni per un pacchetto di accesso](entitlement-management-access-package-request-policy.md)
- [Visualizzare report e log](entitlement-management-reports.md)
