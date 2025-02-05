---
title: 'Azure Active Directory Domain Services: Distribuire Azure AD proxy di applicazione | Microsoft Docs'
description: Usare il proxy di applicazione di Azure AD nei domini gestiti di Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: iainfoulds
manager: daveba
editor: curtand
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/14/2019
ms.author: iainfou
ms.openlocfilehash: 21693926bae681cf15d31dca06344dfa5d865e3b
ms.sourcegitcommit: e42c778d38fd623f2ff8850bb6b1718cdb37309f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/19/2019
ms.locfileid: "69613006"
---
# <a name="deploy-azure-ad-application-proxy-on-an-azure-ad-domain-services-managed-domain"></a>Distribuire il proxy di applicazione di Azure AD in un dominio gestito di Azure AD Domain Services
Il proxy dell'applicazione di Azure Active Directory (AD) consente di supportare lavoratori remoti pubblicando applicazioni locali in modo che siano accessibili tramite Internet. Azure AD Domain Services ora consente di trasferire in modalità lift-and-shift le applicazioni legacy in esecuzione in locale nei servizi di infrastruttura di Azure. È quindi possibile pubblicare queste applicazioni con il proxy di applicazione di Azure AD per garantire l'accesso remoto sicuro agli utenti dell'organizzazione.

Se non si ha familiarità con il proxy di applicazione di Azure AD e per altre informazioni su questa funzionalità, vedere il seguente articolo: [Come fornire l'accesso remoto sicuro alle applicazioni locali](../active-directory/manage-apps/application-proxy.md).

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>Prima di iniziare
Per eseguire le attività elencate in questo articolo sono necessari gli elementi seguenti:

1. Una **sottoscrizione di Azure**valida.
2. Una **directory di Azure AD** sincronizzata con una directory locale o con una directory solo cloud.
3. Per usare il proxy di applicazione Azure AD, è necessaria una **licenza di Azure ad Premium** .
4. **Servizi di dominio Azure AD** devono essere abilitati per la directory di Azure AD. Se non è stato fatto, eseguire tutte le attività descritte nella [guida introduttiva](tutorial-create-instance.md).

<br>

## <a name="task-1---enable-azure-ad-application-proxy-for-your-azure-ad-directory"></a>Attività 1: Abilitare il proxy di applicazione di Azure AD per la directory di Azure AD
Per abilitare il proxy di applicazione di Azure AD per la directory di Azure AD, seguire questa procedura.

1. Accedere come amministratore al [portale di Azure](https://portal.azure.com).

2. Fare clic su **Azure Active Directory** per visualizzare la panoramica della directory. Fare clic su **Applicazioni aziendali**.

3. Fare clic su **Proxy di applicazione**.

4. Per scaricare il connettore, fare clic sul pulsante **Connettore**.

5. Nella pagina di download accettare le condizioni di licenza e l'informativa sulla privacy e fare clic sul pulsante **Scarica**.

    ![Confermare il download](./media/app-proxy/app-proxy-enabled-confirm-download.png)


## <a name="task-2---provision-domain-joined-windows-servers-to-deploy-the-azure-ad-application-proxy-connector"></a>Attività 2: Effettuare il provisioning di server Windows aggiunti a un dominio per distribuire il connettore del proxy di applicazione di Azure AD
È necessario avere a disposizione macchine virtuali Windows Server aggiunte a un dominio in cui installare il connettore del proxy di applicazione di Azure AD. Per alcune applicazioni è possibile scegliere di effettuare il provisioning di più server in cui è installato il connettore. Questa opzione di distribuzione offre maggiore disponibilità e permette di gestire carichi di lavoro di autenticazione più gravosi.

Effettuare il provisioning di server del connettore nella stessa rete virtuale o in una rete virtuale connessa/associata in cui è stato abilitato il dominio gestito di Azure AD Domain Services. Analogamente, i server che ospitano le applicazioni pubblicate tramite il proxy di applicazione devono essere installati nella stessa rete virtuale di Azure.

Per effettuare il provisioning dei server del connettore, seguire la procedura illustrata nell'articolo [Aggiungere una macchina virtuale Windows Server a un dominio gestito](active-directory-ds-admin-guide-join-windows-vm.md).


## <a name="task-3---install-and-register-the-azure-ad-application-proxy-connector"></a>Attività 3: Installare e registrare il connettore del proxy di applicazione di Azure AD
In precedenza è stato effettuato il provisioning di una macchina virtuale Windows Server, poi aggiunta al dominio gestito. In questa attività il connettore del proxy di applicazione di Azure AD viene installato in tale macchina virtuale.

1. Copiare il pacchetto di installazione di connettore nella macchina virtuale in cui va installato il connettore del proxy applicazione Web di Azure AD.

2. Eseguire **AADApplicationProxyConnectorInstaller.exe** nella macchina virtuale. Accettare le condizioni di licenza software.

    ![Accettare le condizione per l'installazione](./media/app-proxy/app-proxy-install-connector-terms.png)
3. Durante l'installazione viene richiesto di registrare il connettore con il proxy di applicazione della directory di Azure AD.
   * Fornire le **credenziali di amministratore dell'applicazione Azure ad**. Il tenant dell'amministratore dell'applicazione può essere diverso dalle credenziali Microsoft Azure.
   * L'account amministratore usato per registrare il connettore deve trovarsi nella stessa directory in cui è stato abilitato il servizio proxy dell'applicazione. Se il dominio del tenant è contoso.com, ad esempio, l'amministratore deve essere admin@contoso.com o qualsiasi altro alias valido di tale dominio.
   * Se la Sicurezza avanzata di Internet Explorer è abilitata per il server in cui si vuole installare il connettore, la schermata di registrazione potrebbe essere bloccata. Per consentire l'accesso, seguire le istruzioni contenute nel messaggio di errore. Verificare che Internet Explorer Enhanced Security Context sia disabilitato.
   * Se la registrazione del connettore non riesce, vedere [Risolvere i problemi del Proxy applicazione](../active-directory/manage-apps/application-proxy-troubleshoot.md).

     ![Connettore installato](./media/app-proxy/app-proxy-connector-installed.png)
4. Per garantire il corretto funzionamento del connettore, eseguire lo strumento di risoluzione dei problemi del connettore del proxy di applicazione di Azure AD. Dopo l'esecuzione dello strumento di risoluzione dei problemi verrà visualizzato un report relativo all'esito positivo dell'operazione.

    ![Esito positivo dello strumento di risoluzione dei problemi](./media/app-proxy/app-proxy-connector-troubleshooter.png)
5. Il connettore appena installato viene elencato nella pagina del proxy di applicazione nella directory di Azure AD.

    ![Nel portale di Azure il connettore installato risulta disponibile](./media/app-proxy/app-proxy-connector-page.png)

> [!NOTE]
> È possibile scegliere di installare i connettori in più server per garantire la disponibilità elevata per l'autenticazione di applicazioni pubblicate tramite il proxy di applicazione di Azure AD. Seguire la stessa procedura appena illustrata per installare il connettore in altri server aggiunti al dominio gestito.
>
>

## <a name="next-steps"></a>Fasi successive
È stato configurato il proxy di applicazione di Azure AD ed è stato integrato con il dominio gestito di Azure AD Domain Services.

* **Migrare applicazioni in macchine virtuali di Azure:** è possibile trasferire in modalità lift-and-shift le applicazioni da server locali in macchine virtuali di Azure aggiunte al dominio gestito. Questo permette di eliminare i costi di infrastruttura legati all'esecuzione di server locali.

* **Pubblicare applicazioni mediante il proxy di applicazione Azure AD:** è possibile pubblicare le applicazioni in esecuzione nelle macchine virtuali di Azure usando il proxy di applicazione di Azure AD. Per altre informazioni, vedere l'articolo relativo alla [pubblicazione di applicazioni con il proxy di applicazione di Azure AD](../active-directory/manage-apps/application-proxy-publish-azure-portal.md).


## <a name="deployment-note---publish-iwa-integrated-windows-authentication-applications-using-azure-ad-application-proxy"></a>Nota di distribuzione: pubblicare applicazioni con l'autenticazione integrata di Windows usando il proxy di applicazione di Azure AD
Abilitare l'accesso Single Sign-On alle applicazioni che usano l'autenticazione integrata di Windows concedendo ai connettori del proxy di applicazione l'autorizzazione a rappresentare gli utenti e a inviare e ricevere token per loro conto. Configurare la delega vincolata Kerberos (KCD) per il connettore per concedere le autorizzazioni necessarie per accedere alle risorse nel dominio gestito. Usare il meccanismo della delega vincolata Kerberos basata su risorse nei domini gestiti per una maggiore sicurezza.


### <a name="enable-resource-based-kerberos-constrained-delegation-for-the-azure-ad-application-proxy-connector"></a>Abilitare la delega vincolata Kerberos basata su risorse per il connettore del proxy di applicazione di Azure AD
Il connettore del proxy di applicazione Azure deve essere configurato per la delega vincolata Kerberos (KCD), in modo che possa rappresentare gli utenti nel dominio gestito. In un dominio gestito di Azure AD Domain Services non si hanno privilegi di amministratore di dominio. Di conseguenza, **non è possibile configurare la delega vincolata Kerberos tradizionale a livello di account in un dominio gestito**.

Usare la delega vincolata Kerberos basata su risorse, come illustrato in [questo articolo](deploy-kcd.md).

> [!NOTE]
> Per poter amministrare il dominio gestito usando i cmdlet di Azure AD PowerShell, è necessario essere un membro del gruppo "AAD DC Administrators".
>
>

Usare il cmdlet di PowerShell Get-ADComputer per recuperare le impostazioni del computer in cui è installato il connettore del proxy di applicazione di Azure AD.
```powershell
$ConnectorComputerAccount = Get-ADComputer -Identity contoso-proxy.contoso.com
```

Successivamente, usare il cmdlet Set-ADComputer per impostare la delega vincolata Kerberos basata su risorse per il server delle risorse.
```powershell
Set-ADComputer contoso-resource.contoso.com -PrincipalsAllowedToDelegateToAccount $ConnectorComputerAccount
```

Se sono stati distribuiti più connettori del proxy di applicazione nel dominio gestito, è necessario configurare la delega vincolata Kerberos basata su risorse per ogni istanza di tale connettore.


## <a name="related-content"></a>Contenuto correlato
* [Servizi di dominio Azure AD: introduzione](tutorial-create-instance.md)
* [Configurare la delega vincolata Kerberos in un dominio gestito](deploy-kcd.md)
* [Panoramica della delega vincolata Kerberos](https://technet.microsoft.com/library/jj553400.aspx)
