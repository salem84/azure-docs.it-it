---
title: Aggiungere una sottoscrizione di Azure esistente al tenant - Azure Active Directory | Microsoft Docs
description: Informazioni su come aggiungere una sottoscrizione di Azure esistente al tenant di Azure Active Directory.
services: active-directory
author: msaburnley
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 03/13/2019
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: fb4fa92d8b3c174cdf9b3695f8564cc11c1ad291
ms.sourcegitcommit: 670c38d85ef97bf236b45850fd4750e3b98c8899
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/08/2019
ms.locfileid: "68851754"
---
# <a name="associate-or-add-an-azure-subscription-to-your-azure-active-directory-tenant"></a>Associare o aggiungere una sottoscrizione di Azure al tenant di Azure Active Directory

Una sottoscrizione di Azure ha una relazione di trust con Azure Active Directory (Azure AD), il che significa che la sottoscrizione considera attendibile Azure AD per autenticare utenti, servizi e dispositivi. Più sottoscrizioni possono considerare attendibile la stessa directory di Azure AD, ma una sottoscrizione può considerare attendibile una sola directory.

Se la sottoscrizione scade, non sarà più possibile accedere alle altre risorse associate alla sottoscrizione. Tuttavia, la directory di Azure AD rimane in Azure e può essere associata o gestita con una sottoscrizione di Azure diversa.

Tutti gli utenti dispongono di una singola *Home* directory per l'autenticazione. Tuttavia, gli utenti possono anche essere guest in altre directory. È possibile visualizzare sia la home directory che la directory guest per ogni utente in Azure AD.

> [!Important]
> Quando si associa una sottoscrizione a una directory diversa, gli utenti che dispongono di ruoli assegnati tramite il controllo degli accessi in [base al ruolo (RBAC)](../../role-based-access-control/role-assignments-portal.md) perderanno l'accesso. Anche gli amministratori della sottoscrizione classica (amministratore del servizio e coamministratori) perderanno l'accesso.
> 
> Inoltre, se si trasferisce il cluster di Azure Kubernetes Service (AKS) a una sottoscrizione diversa o se si trasferisce la sottoscrizione proprietaria del cluster a un nuovo tenant, il cluster perderà la funzionalità a causa delle assegnazioni di ruolo perse e dei diritti delle entità servizio. Per altre informazioni su AKS, vedere [servizio Azure Kubernetes (AKS)](https://docs.microsoft.com/azure/aks/).

## <a name="before-you-begin"></a>Prima di iniziare

Prima di poter associare o aggiungere la sottoscrizione, è necessario eseguire le attività seguenti:

1. Esaminare l'elenco di modifiche seguente e la relativa modalità di impatto:

    - Gli utenti a cui sono stati assegnati ruoli con RBAC perderanno l'accesso
    - L'amministratore del servizio e i coamministratori perderanno l'accesso
    - Se si dispone di insiemi di credenziali delle chiavi, questi saranno inaccessibili e sarà necessario correggerli dopo l'associazione
    - Se si hanno identità gestite per le risorse, ad esempio le macchine virtuali o le app per la logica, sarà necessario riabilitarle o ricrearle dopo l'associazione
    - Se si dispone di un Azure Stack registrato, sarà necessario registrarlo di nuovo dopo l'associazione

1. Accedere con un account che:
    - Dispone di un'assegnazione di ruolo [proprietario](../../role-based-access-control/built-in-roles.md#owner) per la sottoscrizione. Per informazioni su come assegnare il ruolo proprietario, vedere [gestire l'accesso alle risorse di Azure usando RBAC e il portale di Azure](../../role-based-access-control/role-assignments-portal.md).
    - Sia presente sia nella directory corrente associata alla sottoscrizione che nella nuova directory in cui si vuole associare la sottoscrizione in seguito. Per altre informazioni su come ottenere l'accesso a un'altra directory, vedere [Aggiungere utenti di Collaborazione B2B di Azure Active Directory nel portale di Azure](../b2b/add-users-administrator.md).

1. Assicurarsi di non usare una sottoscrizione provider di servizi cloud di Azure (MS-AZR-0145P, MS-AZR-0146P, MS-AZR-159P), una sottoscrizione interna Microsoft (MS-AZR-0015P) o una sottoscrizione Microsoft Imagine (MS-AZR-0144P).
    
## <a name="to-associate-an-existing-subscription-to-your-azure-ad-directory"></a>Per associare una sottoscrizione esistente alla directory di Azure AD

1. Eseguire l'accesso e selezionare una sottoscrizione dalla [pagina Sottoscrizioni del portale di Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

2. Selezionare **Cambia directory**.

    ![Pagina delle sottoscrizioni con l'opzione Cambia directory evidenziata](media/active-directory-how-subscriptions-associated-directory/change-directory-button.png)

3. Esaminare eventuali avvisi che vengono visualizzati e selezionare **Cambia**.

    ![Pagina di modifica della directory con la nuova directory visualizzata](media/active-directory-how-subscriptions-associated-directory/edit-directory-ui.png)

    La directory è stata modificata per la sottoscrizione e viene visualizzato un messaggio di operazione riuscita.

    ![Messaggio di operazione completata sulla modifica della directory](media/active-directory-how-subscriptions-associated-directory/edit-directory-success.png)
4. Usare la selezione **directory** per passare alla nuova directory. La visualizzazione corretta di tutti gli elementi può richiedere diverse ore. Se sembra richiedere troppo tempo, controllare il **filtro della sottoscrizione globale** per la sottoscrizione spostata per assicurarsi che non sia nascosto. Potrebbe essere necessario disconnettersi dal portale di Azure ed eseguire di nuovo l'accesso per visualizzare la nuova directory. 

    ![Pagina Switcher directory con informazioni di esempio](media/active-directory-how-subscriptions-associated-directory/directory-switcher.png)

La modifica della directory della sottoscrizione è un'operazione a livello di servizio, pertanto non influisce sulla proprietà della fatturazione della sottoscrizione. L'amministratore dell'account può comunque modificare l'amministratore del servizio dal [Centro account](https://account.azure.com/subscriptions). Per eliminare la directory originale, è necessario trasferire la proprietà della fatturazione della sottoscrizione a un nuovo amministratore dell'account. Per altre informazioni sul trasferimento della proprietà della fatturazione, vedere [Trasferire la proprietà di una sottoscrizione di Azure a un altro account](../../billing/billing-subscription-transfer.md).

## <a name="post-association-steps"></a>Passaggi successivi all'associazione
Dopo aver associato una sottoscrizione a una directory diversa, potrebbe essere necessario eseguire ulteriori passaggi per riprendere le operazioni.

1. Se sono presenti insiemi di credenziali delle chiavi, è necessario modificare l'ID tenant dell'insieme di credenziali delle chiavi. Per altre informazioni, vedere [modificare un ID tenant di Key Vault dopo uno spostamento della sottoscrizione](../../key-vault/key-vault-subscription-move-fix.md).

2. Se si utilizzano identità gestite assegnate dal sistema per le risorse, è necessario abilitarle di nuovo. Se si utilizzano identità gestite assegnate dall'utente, è necessario ricrearle. Dopo aver riattivato o ricreato le identità gestite, è necessario ristabilire le autorizzazioni assegnate a tali identità. Per altre informazioni, vedere informazioni sulle [identità gestite per le risorse di Azure](../managed-identities-azure-resources/overview.md).

3. Se è stata registrata una Azure Stack usando questa sottoscrizione, è necessario ripetere la registrazione. Per altre informazioni, vedere [registrare Azure stack con Azure](/azure-stack/operator/azure-stack-registration).



## <a name="next-steps"></a>Passaggi successivi

- Per creare un nuovo tenant di Azure AD, vedere [Access Azure Active Directory to create a new tenant](active-directory-access-create-new-tenant.md) (Accedere ad Azure Active Directory per creare un nuovo tenant)

- Per altre informazioni sul modo in cui l'accesso alle risorse viene controllato in Microsoft Azure, vedere [Informazioni sull'accesso alle risorse in Azure](../../role-based-access-control/rbac-and-directory-admin-roles.md)

- Per altre informazioni su come assegnare i ruoli in Azure AD, vedere [Come assegnare ruoli della directory agli utenti con Azure Active Directory](active-directory-users-assign-role-azure-portal.md)
