---
title: Impostare gli URL di reindirizzamento su b2clogin.com-Azure Active Directory B2C
description: Informazioni sull'uso di b2clogin.com negli URL di reindirizzamento per Azure Active Directory B2C.
services: active-directory-b2c
author: mmacy
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/17/2019
ms.author: marsma
ms.subservice: B2C
ms.openlocfilehash: dbc366daac89f44d4b084081590124f81ff9cc9c
ms.sourcegitcommit: 040abc24f031ac9d4d44dbdd832e5d99b34a8c61
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/16/2019
ms.locfileid: "69533753"
---
# <a name="set-redirect-urls-to-b2clogincom-for-azure-active-directory-b2c"></a>Impostare gli URL di reindirizzamento su b2clogin.com per Azure Active Directory B2C

Quando si configura un provider di identità per l'iscrizione e l'accesso nell'applicazione Azure Active Directory B2C (Azure AD B2C), è necessario specificare un URL di reindirizzamento. Non è più necessario fare riferimento a *login.microsoftonline.com* nelle applicazioni e nelle API. Usare invece *b2clogin.com* per tutte le nuove applicazioni ed eseguire la migrazione delle applicazioni esistenti da *login.microsoftonline.com* a *b2clogin.com*.

## <a name="benefits-of-b2clogincom"></a>Vantaggi di b2clogin.com

Quando si usa *b2clogin.com* come URL di reindirizzamento:

* Lo spazio usato nell'intestazione cookie dei servizi Microsoft viene ridotto.
* Gli URL di reindirizzamento non devono più includere un riferimento a Microsoft.
* Il codice lato client JavaScript è supportato (attualmente in [Anteprima](user-flow-javascript-overview.md)) in pagine personalizzate. A causa delle restrizioni di sicurezza, gli elementi del codice JavaScript e del modulo HTML vengono rimossi dalle pagine personalizzate se si utilizza *login.microsoftonline.com*.

## <a name="overview-of-required-changes"></a>Panoramica delle modifiche necessarie

Per eseguire la migrazione delle applicazioni a *b2clogin.com*, potrebbe essere necessario apportare diverse modifiche:

* Modificare l'URL di reindirizzamento nelle applicazioni del provider di identità in modo che faccia riferimento a *b2clogin.com*.
* Aggiornare le applicazioni Azure AD B2C in modo da usare *b2clogin.com* nei riferimenti all'endpoint del flusso utente e del token.
* Aggiornare le **origini** consentite definite nelle impostazioni CORS per la [personalizzazione dell'interfaccia utente](active-directory-b2c-ui-customization-custom-dynamic.md).

## <a name="change-identity-provider-redirect-urls"></a>Modificare gli URL di reindirizzamento del provider di identità

Nel sito Web del provider di identità in cui è stata creata un'applicazione, modificare tutti gli URL attendibili `your-tenant-name.b2clogin.com` per il reindirizzamento al anziché *login.microsoftonline.com*.

Esistono due formati che è possibile usare per gli URL di reindirizzamento di b2clogin.com. Il primo consente di non visualizzare "Microsoft" in qualsiasi punto dell'URL usando l'ID tenant (GUID) al posto del nome di dominio del tenant:

```
https://{your-tenant-name}.b2clogin.com/{your-tenant-id}/oauth2/authresp
```

La seconda opzione Usa il nome di dominio del tenant nel formato `your-tenant-name.onmicrosoft.com`. Ad esempio:

```
https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp
```

Per entrambi i formati:

* Sostituire `{your-tenant-name}` con il nome del tenant di Azure AD B2C.
* Rimuovere `/te` se è presente nell'URL.

## <a name="update-your-applications-and-apis"></a>Aggiornare le applicazioni e le API

Il codice nelle applicazioni e nelle API abilitate per `login.microsoftonline.com` Azure ad B2C può riferirsi a in diversi punti. Ad esempio, il codice potrebbe avere riferimenti a flussi utente ed endpoint token. Aggiornare il codice seguente in modo `your-tenant-name.b2clogin.com`che faccia riferimento a:

* Endpoint autorizzazione
* Endpoint token
* Autorità emittente di token

Ad esempio, l'endpoint dell'autorità per i criteri di iscrizione/accesso di Contoso ora sarà:

```
https://contosob2c.b2clogin.com/00000000-0000-0000-0000-000000000000/B2C_1_signupsignin1
```

## <a name="microsoft-authentication-library-msal"></a>Microsoft Authentication Library (MSAL)

### <a name="validateauthority-property"></a>Proprietà ValidateAuthority

Se si usa [MSAL.NET][msal-dotnet] v2 o versioni precedenti, impostare la proprietà `false` ValidateAuthority su on creazione di un'istanza del client per consentire reindirizzamenti a *b2clogin.com*. Questa impostazione non è necessaria per MSAL.NET V3 e versioni successive.

```CSharp
ConfidentialClientApplication client = new ConfidentialClientApplication(...); // Can also be PublicClientApplication
client.ValidateAuthority = false; // MSAL.NET v2 and earlier **ONLY**
```

Se si usa [MSAL per JavaScript][msal-js]:

```JavaScript
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: false
  }
);
```

<!-- LINKS - External -->
[msal-dotnet]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[msal-dotnet-b2c]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics
[msal-js]: https://github.com/AzureAD/microsoft-authentication-library-for-js
[msal-js-b2c]: ../active-directory/develop/msal-b2c-overview.md