---
title: Guida alla sicurezza per Azure multi-Factor Authentication - Azure Active Directory
description: Questo documento include le linee guida relative all'uso di Azure MFA con account Azure
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 436b7899b1a9d4f9cab1ca2581ff9b5b162de8ac
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60357975"
---
# <a name="security-guidance-for-using-azure-multi-factor-authentication-with-azure-ad-accounts"></a>Linee guida sulla sicurezza per l'uso di Azure Multi-Factor Authentication con account Azure AD

La scelta preferita per migliorare il processo di autenticazione nella maggior parte delle organizzazioni è la verifica in due passaggi. Azure Multi-Factor Authentication (MFA) consente alle aziende di soddisfare i requisiti di sicurezza e conformità, offrendo un'esperienza di accesso semplice per gli utenti. Questo articolo illustra alcuni consigli che è consigliabile considerare quando si pianifica l'adozione di Azure MFA.

## <a name="deploy-azure-mfa-in-the-cloud"></a>Distribuire Azure MFA nel cloud

È possibile [abilitare Azure MFA per tutti gli utenti](howto-mfa-getstarted.md) usando due metodi diversi.

* Acquistare licenze per ogni utente (Azure MFA, Azure AD Premium o Enterprise Mobility + Security)
* Creare un provider di Multi-Factor Authentication scegliendo di pagare in base al numero di utenti o di autenticazioni

### <a name="licenses"></a>Licenze

![Applicare le licenze agli utenti, abilitare, inviare una notifica](./media/multi-factor-authentication-security-best-practices/ems.png)

Se si dispone di licenze Azure AD Premium o Enterprise Mobility + Security, si può già usare Azure MFA. L'organizzazione non ha l'esigenza di avere elementi aggiuntivi per estendere la funzionalità di verifica in due passaggi a tutti gli utenti. È sufficiente assegnare una licenza a un utente e quindi è possibile attivare l'autenticazione MFA.

Quando si configura l'autenticazione Multi-Factor Authentication, tenere in considerazione le raccomandazioni seguenti:

* Non creare un provider di Multi-Factor Authentication in base al numero di autenticazioni. Questo perché si rischia di pagare per richieste di verifica provenienti da utenti che dispongono già di licenze.
* Se non si dispone di licenze sufficienti per tutti gli utenti, è possibile creare un provider di Multi-Factor Authentication in base al numero di utenti per coprire il resto dell'organizzazione. 
* Azure AD Connect è un requisito solo se si sincronizza l'ambiente Active Directory locale con una directory di Azure AD. Se si usa una directory di Azure AD non sincronizzata con un'istanza locale di Active Directory, Azure AD Connect non è necessario.

### <a name="multi-factor-auth-provider"></a>Provider di Multi-Factor Authentication

![Provider di multi-Factor Authentication](./media/multi-factor-authentication-security-best-practices/authprovider.png)

In assenza di licenze che includono Azure MFA, è possibile [creare un provider di autenticazione MFA](concept-mfa-authprovider.md).

Quando si crea il provider di autenticazione, è necessario selezionare una directory e considerare quanto segue:

* Non è necessaria una directory di Azure AD per creare un provider Multi-Factor Authentication, ma con una è possibile ottenere più funzionalità. Le seguenti funzionalità vengono abilitate associando il provider di Multi-Factor Authentication a una directory di Azure AD:
  * Estendere la verifica in due passaggi a tutti gli utenti
  * Offrire funzionalità aggiuntive agli amministratori globali, ad esempio il portale di gestione, messaggi di saluto personalizzati e report.
* Se si esegue la sincronizzazione dell'ambiente Active Directory locale con una directory di Azure AD, sono necessarie le funzionalità DirSync o AAD Sync. Se si usa una directory di Azure AD non sincronizzata con un'istanza locale di Active Directory, non sono necessarie le funzionalità DirSync o AAD Sync.
* Scegliere il modello di consumo più adatto alle esigenze aziendali. Dopo aver selezionato il modello non sarà possibile modificarlo. I due modelli sono:
  * Per autenticazione: viene applicato un addebito per ogni verifica. Usare questo modello per usare la verifica in due passaggi per qualsiasi utente che accede a una determinata app, non per utenti specifici.
  * Per utente abilitato: viene applicato un addebito per ogni utente che viene abilitato per Azure MFA. Usare questo modello se non tutti gli utenti dispongono di licenze Azure AD Premium o Enterprise Mobility Suite.

### <a name="supportability"></a>Supporto

Poiché la maggior parte degli utenti è abituata a usare le password solo per l'autenticazione, è importante che l'azienda renda consapevoli tutti gli utenti in merito a questo processo. Questa consapevolezza può ridurre la probabilità che gli utenti contattino l'help desk per problemi di lieve entità riguardanti MFA. Esistono tuttavia alcuni scenari in cui è necessario disabilitare temporaneamente MFA. Per sapere come gestire questi scenari, usare le linee guida seguenti:

* Preparare il personale di supporto tecnico a gestire gli scenari in cui l'utente non è in grado di accedere perché l'app per dispositivi mobili o il telefono non ricevono una notifica o una chiamata telefonica. Il supporto tecnico può abilitare un'[opzione di bypass monouso](howto-mfa-mfasettings.md#one-time-bypass) per consentire all'utente di autenticarsi una sola volta "ignorando" la verifica in due passaggi. Il bypass è temporaneo e scade dopo un numero di secondi specificato.
* La [funzionalità per gli indirizzi IP attendibili](howto-mfa-mfasettings.md#trusted-ips) in Azure MFA può essere considerata come un modo per ridurre al minimo la verifica in due passaggi. Questa funzionalità consente agli amministratori di un tenant gestito o federato di ignorare la verifica in due passaggi per gli utenti che accedono dalla rete Intranet locale dell'azienda. Le funzionalità sono disponibili per i tenant di Azure AD che dispongono di licenze Azure AD Premium, Enterprise Mobility Suite o Azure Multi-Factor Authentication.

## <a name="best-practices-for-an-on-premises-deployment"></a>Procedure consigliate per una distribuzione locale

Se la società decide di sfruttare la propria infrastruttura per abilitare MFA, è necessario [distribuire un server Microsoft Azure Multi-Factor Authentication in locale](howto-mfaserver-deploy.md). I componenti del server MFA sono illustrati nel diagramma seguente:

![I componenti del Server MFA predefinito](./media/multi-factor-authentication-security-best-practices/server.png) \*non è installato per impostazione predefinita \** installato ma non abilitato per impostazione predefinita

Il server Azure Multi-Factor Authentication può proteggere le risorse del cloud e locali usando la federazione. È necessario avere AD FS federato con un tenant di Azure AD.
Quando si configura il server Multi-Factor Authentication, tenere in considerazione quanto segue:

* Se non si proteggono le risorse di Azure AD tramite Active Directory Federation Services (AD FS), il primo passaggio di autenticazione viene eseguito in locale tramite AD FS. Il secondo passaggio viene eseguito in locale rispettando l'attestazione.
* Non è necessario scaricare e installare il server Multi-Factor Authentication nel server federativo AD FS. È tuttavia necessario installare l'adapter Multi-Factor Authentication per AD FS in un computer Windows Server 2012 R2 che esegue AD FS. È possibile installare il server in un computer diverso, purché sia una versione supportata, e installare separatamente l'adapter AD FS nel server federativo di AD FS. 
* L'installazione guidata dell'adapter AD FS di Multi-Factor Authentication crea un gruppo di sicurezza denominato PhoneFactor Admins in Active Directory e quindi aggiunge l'account del servizio AD FS a questo gruppo. Verificare che il gruppo PhoneFactor Admins sia stato creato nel controller di dominio e che l'account del servizio AD FS sia un membro di questo gruppo. Se necessario, aggiungere l'account del servizio ADFS manualmente al gruppo PhoneFactor Admins sul controller di dominio.

### <a name="user-portal"></a>Portale per gli utenti

Il portale per gli utenti consente funzionalità self-service e fornisce un set completo di funzionalità di amministrazione utenti. Viene eseguito in un sito web di Internet Information Server (IIS). Per configurare questo componente, usare le linee guida seguenti:

* Usare IIS 6 o versione successiva
* Installare e registrare ASP.NET v2.0.507207
* Assicurarsi che il server possa essere distribuito in una rete perimetrale

### <a name="app-passwords"></a>Password dell'app

Se l'organizzazione è federata e usa SSO con Azure AD e si prevede di usare Azure MFA, tenere presente i dettagli di seguito:

* La password dell'app viene verificata da Azure AD e di conseguenza ignora la federazione. La federazione viene usata solo quando si configura la password dell'app.
* Per gli utenti federati (SSO) le password vengono archiviate nell'ID dell'organizzazione. Se l'utente lascia l'azienda, tali informazioni devono essere trasmesse nell'ID organizzazione tramite DirSync. La disabilitazione o l'eliminazione dell'account può richiedere fino a 3 ore per la sincronizzazione, ritardando la disabilitazione o l'eliminazione delle password dell'app in Azure AD.
* Le impostazioni locali di Controllo dell'accesso Client non vengono rispettate dalla password dell'app.
* Per le password dell'app non è disponibile alcuna funzionalità di registrazione o controllo dell'autenticazione in locale.
* Alcune architetture avanzate possono richiedere una combinazione di nome utente e password dell'organizzazione e password dell'app quando si usa la verifica in due passaggi con i client, a seconda della posizione in cui viene eseguita l'autenticazione. Per i client che si autenticano con un'infrastruttura locale, verranno usati un nome utente e una password dell'organizzazione. Per i client che eseguono l'autenticazione con AD Azure, verrà usata la password dell'app.
* Per impostazione predefinita, gli utenti non possono creare password dell'app. Se si desidera consentire agli utenti di creare password dell'app, selezionare l'opzione **Consentire agli utenti di creare password dell'app per accedere alle applicazioni non basate su browser**.

## <a name="additional-considerations"></a>Considerazione aggiuntive

Usare questo elenco per leggere alcune considerazioni e procedure consigliate aggiuntive per ogni componente che viene distribuito in locale:

* Configurare Azure Multi-Factor Authentication con [Active Directory Federation Service](multi-factor-authentication-get-started-adfs.md).
* Installare e configurare il server Azure MFA con [Autenticazione RADIUS](howto-mfaserver-dir-radius.md).
* Installare e configurare il server Azure MFA con [Autenticazione IIS](howto-mfaserver-iis.md).
* Installare e configurare il server Azure MFA con l'[autenticazione di Windows](howto-mfaserver-windows.md).
* Installare e configurare il server Azure MFA con [Autenticazione LDAP](howto-mfaserver-dir-ldap.md).
* Installare e configurare il server Azure MFA con [Gateway Desktop remoto e server Azure Multi-Factor Authentication usando RADIUS](howto-mfaserver-nps-rdg.md).
* Installare e configurare la sincronizzazione tra il server Azure MFA e [Windows Server Active Directory](howto-mfaserver-dir-ad.md).
* [Distribuire il servizio Web App Mobile del server Azure Multi-Factor Authentication](howto-mfaserver-deploy-mobileapp.md).
* [Configurazione VPN avanzata con Azure Multi-Factor Authentication](howto-mfaserver-nps-vpn.md) per appliance Cisco ASA, Citrix Netscaler e Juniper/Pulse Secure VPN con LDAP o RADIUS.

## <a name="next-steps"></a>Passaggi successivi

Anche se questo articolo evidenzia alcune procedure consigliate per Azure MFA, sono disponibili anche altre risorse che è possibile usare quando si pianifica la distribuzione di MFA. L'elenco seguente include alcuni articoli importanti che possono risultare utili durante questo processo:

* [Report in Azure Multi-Factor Authentication](howto-mfa-reporting.md)
* [L'esperienza di registrazione della verifica in due passaggi](../user-help/multi-factor-authentication-end-user-first-time.md)
* [Domande frequenti su Azure Multi-Factor Authentication](multi-factor-authentication-faq.md)
