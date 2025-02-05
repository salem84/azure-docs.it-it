---
title: Requisiti dei dati di SSPR AD Azure - Azure Active Directory
description: Requisiti dei dati per la Reimpostazione self-service delle password e informazioni su come soddisfarli
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahenry
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5a0d7edb6c7faafcad55e827c2d9e3d2eeea40f5
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60358031"
---
# <a name="deploy-password-reset-without-requiring-end-user-registration"></a>Distribuire la reimpostazione della password senza richiedere la registrazione dell'utente finale

Per distribuire la funzionalità di reimpostazione password self-service di Azure Active Directory (Azure AD) devono essere presenti i dati di autenticazione. Alcune organizzazioni richiedono agli utenti di immettere personalmente i dati di autenticazione. Tuttavia, molte organizzazioni preferiscono sincronizzare i dati già presenti in Active Directory. I dati sincronizzati sono resi disponibili ad Azure AD e alla funzionalità di reimpostazione password self-service senza richiedere l'intervento dell'utente se:

* I dati sono formattati correttamente nella directory locale dell'utente.
* È stata eseguita la configurazione di [Azure AD Connect usando le impostazioni rapide](../hybrid/how-to-connect-install-express.md).

Per garantirne il corretto funzionamento, i numeri di telefono devono essere nel formato *+PrefissoInternazionale NumeroTelefonico*, ad esempio +1 4255551234.

> [!NOTE]
> È necessario uno spazio tra il prefisso internazionale e il numero di telefono.
>
> La reimpostazione della password non supporta le estensioni del telefono. Anche nel formato +1 4255551234X12345, le estensioni vengono rimosse prima della chiamata.

## <a name="fields-populated"></a>Campi popolati

Se si usano le impostazioni predefinite in Azure AD Connect, vengono eseguiti i mapping seguenti:

| Active Directory locale | Azure AD |
| --- | --- |
| telephoneNumber | Telefono ufficio |
| mobile | Cellulare |

Quando l'utente verifica il numero di cellulare, anche il campo Telefono in Informazioni di contatto per l'autenticazione in Azure AD viene popolato con il numero.

## <a name="authentication-contact-info"></a>Informazioni di contatto per l'autenticazione

L'amministratore globale può impostare manualmente le informazioni di contatto per l'autenticazione per l'utente come illustrato nello screenshot seguente.

![Informazioni su un utente l'autenticazione di contatto in Azure AD][Contact]

Se il campo Telefono è popolato e l'opzione Cellulare è abilitata nei criteri SSPR, il numero verrà visualizzato nella pagina di registrazione per la reimpostazione della password e durante il flusso di lavoro di reimpostazione della password.

Il campo Telefono alternativo non è usato per la reimpostazione della password.

Se il campo Posta elettronica è popolato e l'opzione Posta elettronica è abilitata nei criteri SSPR, l'indirizzo di posta elettronica verrà visualizzato nella pagina di registrazione per la reimpostazione della password e durante il flusso di lavoro di reimpostazione della password.

Se il campo Indirizzo di posta elettronica alternativo è popolato e l'opzione Posta elettronica è abilitata nei criteri SSPR, l'indirizzo di posta elettronica **non** verrà visualizzato nella pagina di registrazione per la reimpostazione della password ma verrà visualizzato durante il flusso di lavoro di reimpostazione della password.

## <a name="security-questions-and-answers"></a>Domande di sicurezza e risposte

Le domande di sicurezza e le risposte sono archiviate in modo sicuro nel tenant di Azure AD e sono accessibili agli utenti esclusivamente tramite il [portale di registrazione SSPR](https://aka.ms/ssprsetup). Gli amministratori non possono vedere, impostare o modificare il contenuto di domande e risposte di altri utenti.

## <a name="what-happens-when-a-user-registers"></a>Cosa accade quando un utente si registra

Quando un utente si registra, i campi seguenti vengono impostati nella pagina di registrazione:

* **Telefono per l'autenticazione**
* **Indirizzo di posta elettronica per l'autenticazione**
* **Domande di sicurezza e risposte**

Se è stato specificato un valore per **Cellulare** o **Indirizzo di posta elettronica alternativo**, gli utenti possono usare immediatamente questi valori per reimpostare le password, anche se non hanno eseguito la registrazione per il servizio. Gli utenti visualizzano e possono modificare tali valori quando si registrano per la prima volta. Dopo aver completato la registrazione, questi valori vengono salvati in modo permanente rispettivamente nei campi **Telefono per l'autenticazione** e **Indirizzo di posta elettronica per l'autenticazione**.

## <a name="set-and-read-the-authentication-data-through-powershell"></a>Impostare e leggere i dati di autenticazione tramite PowerShell

I campi seguenti possono essere impostati tramite PowerShell:

* **Indirizzo di posta elettronica alternativo**
* **Cellulare**
* **Telefono ufficio**: può essere impostato solo se non in sincronizzazione con una directory locale

### <a name="use-powershell-version-1"></a>Usare PowerShell versione 1

Per iniziare, è necessario [scaricare e installare il modulo di Azure AD PowerShell](https://msdn.microsoft.com/library/azure/jj151815.aspx#bkmk_installmodule). Al termine dell'installazione, è possibile seguire la procedura seguente per configurare tutti i campi.

#### <a name="set-the-authentication-data-with-powershell-version-1"></a>Impostare i dati di autenticazione con PowerShell versione 1

```PowerShell
Connect-MsolService

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com")
Set-MsolUser -UserPrincipalName user@domain.com -MobilePhone "+1 1234567890"
Set-MsolUser -UserPrincipalName user@domain.com -PhoneNumber "+1 1234567890"

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com") -MobilePhone "+1 1234567890" -PhoneNumber "+1 1234567890"
```

#### <a name="read-the-authentication-data-with-powershell-version-1"></a>Leggere i dati di autenticazione con PowerShell versione 1

```PowerShell
Connect-MsolService

Get-MsolUser -UserPrincipalName user@domain.com | select AlternateEmailAddresses
Get-MsolUser -UserPrincipalName user@domain.com | select MobilePhone
Get-MsolUser -UserPrincipalName user@domain.com | select PhoneNumber

Get-MsolUser | select DisplayName,UserPrincipalName,AlternateEmailAddresses,MobilePhone,PhoneNumber | Format-Table
```

#### <a name="read-the-authentication-phone-and-authentication-email-options"></a>Leggere i valori delle opzioni Telefono per l'autenticazione e Indirizzo di posta elettronica per l'autenticazione

Per leggere i valori delle opzioni **Telefono per l'autenticazione** e **Indirizzo di posta elettronica per l'autenticazione** se si usa PowerShell versione 1, usare i comandi seguenti:

```PowerShell
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select PhoneNumber
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select Email
```

### <a name="use-powershell-version-2"></a>Usare PowerShell versione 2

Per iniziare, è necessario [scaricare e installare il modulo di Azure AD PowerShell versione 2](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0). Al termine dell'installazione, è possibile seguire la procedura seguente per configurare tutti i campi.

Per eseguire l'installazione rapida da versioni recenti di PowerShell che supportano Install-Module, eseguire i comandi seguenti. La prima riga controlla se il modulo è già installato.

```PowerShell
Get-Module AzureADPreview
Install-Module AzureADPreview
Connect-AzureAD
```

#### <a name="set-the-authentication-data-with-powershell-version-2"></a>Impostare i dati di autenticazione con PowerShell versione 2

```PowerShell
Connect-AzureAD

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("email@domain.com")
Set-AzureADUser -ObjectId user@domain.com -Mobile "+1 2345678901"
Set-AzureADUser -ObjectId user@domain.com -TelephoneNumber "+1 1234567890"

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("emails@domain.com") -Mobile "+1 1234567890" -TelephoneNumber "+1 1234567890"
```

#### <a name="read-the-authentication-data-with-powershell-version-2"></a>Leggere i dati di autenticazione con PowerShell versione 2

```PowerShell
Connect-AzureAD

Get-AzureADUser -ObjectID user@domain.com | select otherMails
Get-AzureADUser -ObjectID user@domain.com | select Mobile
Get-AzureADUser -ObjectID user@domain.com | select TelephoneNumber

Get-AzureADUser | select DisplayName,UserPrincipalName,otherMails,Mobile,TelephoneNumber | Format-Table
```

## <a name="next-steps"></a>Passaggi successivi

* [Come completare l'implementazione della reimpostazione della password self-service per gli utenti](howto-sspr-deployment.md)
* [Reimpostare o modificare la password](../user-help/active-directory-passwords-update-your-own-password.md)
* [Registrarsi per la reimpostazione della password self-service](../user-help/active-directory-passwords-reset-register.md)
* [Domande sulle licenze](concept-sspr-licensing.md)
* [Metodi di autenticazione disponibili per gli utenti](concept-sspr-howitworks.md#authentication-methods)
* [Opzioni dei criteri per la reimpostazione della password self-service](concept-sspr-policy.md)
* [Panoramica del writeback delle password](howto-sspr-writeback.md)
* [Come creare un report sull'attività relativa alla reimpostazione della password self-service](howto-sspr-reporting.md)
* [Informazioni sulle opzioni della reimpostazione della password self-service](concept-sspr-howitworks.md)
* [Come risolvere i problemi di reimpostazione della password self-service](active-directory-passwords-troubleshoot.md)
* [Altre informazioni non illustrate altrove](active-directory-passwords-faq.md)

[Contact]: ./media/howto-sspr-authenticationdata/user-authentication-contact-info.png "Gli amministratori globali possono modificare informazioni di contatto per l'autenticazione di un utente"
