---
title: Funzionalità e configurazione del Servizio di sincronizzazione Azure AD Connect | Documentazione Microsoft
description: Descrive le funzionalità sul lato del servizio per il Servizio di sincronizzazione Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 213aab20-0a61-434a-9545-c4637628da81
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/25/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: be67a6f287e2d6e77070928cbe12542857696011
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60347542"
---
# <a name="azure-ad-connect-sync-service-features"></a>Funzionalità del servizio di sincronizzazione Azure AD Connect

La funzionalità di sincronizzazione di Azure AD Connect include due componenti:

* Il componente locale, ovvero l'utilità di **sincronizzazione Azure AD Connect** detta anche **motore di sincronizzazione**.
* Il servizio residente in Azure AD, noto anche come **Servizio di sincronizzazione Azure AD Connect**

Questo argomento illustra l'utilizzo delle funzionalità del **Servizio di sincronizzazione Azure AD Connect** seguenti e come è possibile configurarle e usarle con Windows PowerShell.

Queste impostazioni sono configurate tramite il [Modulo di Microsoft Azure Active Directory per Windows PowerShell](https://aka.ms/aadposh). Scaricarlo e installarlo separatamente da Azure AD Connect. I cmdlet documentati in questo argomento sono stati introdotti nella [versione di marzo 2016 (build 9031.1)](https://social.technet.microsoft.com/wiki/contents/articles/28552.microsoft-azure-active-directory-powershell-module-version-release-history.aspx#Version_9031_1). Se i cmdlet documentati in questo argomento non sono disponibili o non producono lo stesso risultato, assicurarsi di eseguire la versione più recente.

Per visualizzare la configurazione nella directory di Azure AD, eseguire `Get-MsolDirSyncFeatures`.  
![Risultato di Get-MsolDirSyncFeatures](./media/how-to-connect-syncservice-features/getmsoldirsyncfeatures.png)

Molte di queste impostazioni possono essere modificate solo da Azure AD Connect.

Di seguito sono riportate le impostazioni che possono essere configurate da `Set-MsolDirSyncFeature`:

| DirSyncFeature | Commento |
| --- | --- |
| [EnableSoftMatchOnUpn](#userprincipalname-soft-match) |Consente l'aggiunta di oggetti a userPrincipalName oltre all'indirizzo SMTP primario. |
| [SynchronizeUpnForManagedUsers](#synchronize-userprincipalname-updates) |Consente al motore di sincronizzazione di aggiornare l'attributo userPrincipalName per gli utenti gestiti/con licenza (non federati). |

Una volta abilitata una funzionalità, non potrà essere disabilitata di nuovo.

> [!NOTE]
> Dal 24 agosto 2016, la funzionalità *Duplicate attribute resiliency* (Resilienza degli attributi duplicati) è abilitata per impostazione predefinita per le nuove directory di Azure AD. Questa funzionalità sarà implementata e abilitata anche nelle directory create prima di tale data. Si riceverà una notifica tramite posta elettronica quando sta per essere abilitata questa funzionalità nella directory dell'utente.
> 
> 

Le impostazioni seguenti vengono configurate da Azure AD Connect e non possono essere modificate da `Set-MsolDirSyncFeature`:

| DirSyncFeature | Commento |
| --- | --- |
| DeviceWriteback |[Azure AD Connect: Abilitazione del writeback dei dispositivi](how-to-connect-device-writeback.md) |
| DirectoryExtensions |[Servizio di sincronizzazione Azure AD Connect: Estensioni della directory](how-to-connect-sync-feature-directory-extensions.md) |
| [DuplicateProxyAddressResiliency<br/>DuplicateUPNResiliency](#duplicate-attribute-resiliency) |Consente di mettere in quarantena un attributo quando è un duplicato di un altro oggetto, invece di causare l'errore dell'intero oggetto durante l'esportazione. |
| Sincronizzazione dell'hash delle password |[Implementazione della sincronizzazione dell'hash delle password con la sincronizzazione di Azure AD Connect](how-to-connect-password-hash-synchronization.md) |
|Autenticazione pass-through|[Accesso utente con l'autenticazione pass-through di Azure Active Directory](how-to-connect-pta.md)|
| UnifiedGroupWriteback |[Anteprima: Writeback dei gruppi](how-to-connect-preview.md#group-writeback) |
| UserWriteback |Attualmente non è supportata. |

## <a name="duplicate-attribute-resiliency"></a>Duplicate attribute resiliency

Invece di causare un errore di provisioning degli oggetti con UPN o proxyAddress duplicati, l'attributo duplicato viene "messo in quarantena" e viene assegnato un valore temporaneo. Una volta risolto il conflitto, l'UPN temporaneo viene modificato automaticamente con il valore appropriato. Per altre informazioni, vedere [Sincronizzazione delle identità e resilienza degli attributi duplicati](how-to-connect-syncservice-duplicate-attribute-resiliency.md).

## <a name="userprincipalname-soft-match"></a>Corrispondenza flessibile di userPrincipalName

Quando questa funzionalità è abilitata, la corrispondenza flessibile viene abilitata per l'UPN, oltre all' [indirizzo SMTP primario](https://support.microsoft.com/kb/2641663)che è sempre abilitato. La corrispondenza flessibile viene usata per associare gli utenti del cloud esistente in Azure AD con gli utenti locali.

Questa funzionalità è utile se gli account AD account locali devono corrispondere agli account esistenti creati nel cloud e non si usa Exchange Online. In questo scenario non esiste in genere un motivo per impostare l'attributo SMTP nel cloud.

Questa funzionalità è attivata per impostazione predefinita per le nuove directory di Azure AD . Per vedere se la funzionalità è abilitata per l'utente corrente, eseguire:  

```powershell
Get-MsolDirSyncFeatures -Feature EnableSoftMatchOnUpn
```

Se questa funzionalità non è abilitata per la directory di Azure AD in uso, è possibile abilitarla eseguendo:  

```powershell
Set-MsolDirSyncFeature -Feature EnableSoftMatchOnUpn -Enable $true
```

## <a name="synchronize-userprincipalname-updates"></a>Sincronizzare gli aggiornamenti di userPrincipalName

In genere, gli aggiornamenti dell'attributo UserPrincipalName usando il servizio di sincronizzazione locale vengono bloccati, a meno che siano rispettate entrambe le condizioni seguenti:

* L'utente è gestito (non federato).
* All'utente non è stata assegnata una licenza.

Per alte informazioni, vedere [I nomi utente in Office 365, Azure o Intune non corrispondono agli ID di accesso o alternativi dell'UPN locale](https://support.microsoft.com/kb/2523192).

L'abilitazione di questa funzionalità consente al motore di sincronizzazione di aggiornare userPrincipalName quando è modificato in locale e si utilizza l'autenticazione pass-through o sincronizzazione hash password. Se si usa la federazione, questa funzionalità non è supportata.

Questa funzionalità è attivata per impostazione predefinita per le nuove directory di Azure AD . Per vedere se la funzionalità è abilitata per l'utente corrente, eseguire:  

```powershell
Get-MsolDirSyncFeatures -Feature SynchronizeUpnForManagedUsers
```

Se questa funzionalità non è abilitata per la directory di Azure AD in uso, è possibile abilitarla eseguendo:  

```powershell
Set-MsolDirSyncFeature -Feature SynchronizeUpnForManagedUsers -Enable $true
```

Dopo aver abilitato questa funzionalità, i valori di userPrincipalName esistenti rimarranno invariati. Alla successiva modifica dell'attributo userPrincipalName locale, la normale sincronizzazione differenziale degli utenti aggiornerà l'UPN.  

## <a name="see-also"></a>Vedere anche

* [Servizio di sincronizzazione Azure AD Connect](how-to-connect-sync-whatis.md)
* [Integrazione delle identità locali con Azure Active Directory](whatis-hybrid-identity.md).
