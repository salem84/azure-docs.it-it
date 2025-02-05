---
title: Eseguire il mapping di un nome DNS personalizzatro esistente - Servizio app di Azure | Microsoft Docs
description: Informazioni su come aggiungere un nome di dominio DNS personalizzato esistente, ad esempio il dominio personale, in un'app Web, nel back-end dell'app per dispositivi mobili o nell'app per le API del servizio app di Azure.
keywords: servizio app, servizio app di Azure, mapping di dominio, nome di dominio, dominio esistente, nome host
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: ''
ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 06/06/2019
ms.author: cephalin
ms.custom: seodec18
ms.openlocfilehash: bbb064c358eba2dd64ba9ae86540a30cb56adb66
ms.sourcegitcommit: ccb9a7b7da48473362266f20950af190ae88c09b
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/05/2019
ms.locfileid: "67595020"
---
# <a name="tutorial-map-an-existing-custom-dns-name-to-azure-app-service"></a>Esercitazione: Eseguire il mapping di un nome DNS personalizzato esistente al Servizio app di Azure

[Servizio app di Azure](overview.md) offre un servizio di hosting Web con scalabilità elevata e funzioni di auto-correzione. Questa esercitazione illustra come eseguire il mapping di un nome DNS personalizzato esistente al Servizio app di Azure.

![Passaggio all'app di Azure nel portale](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

In questa esercitazione si apprenderà come:

> [!div class="checklist"]
> * Eseguire il mapping di un sottodominio (ad esempio, `www.contoso.com`) usando un record CNAME
> * Eseguire il mapping di un dominio radice (ad esempio, `contoso.com`) usando un record A
> * Eseguire il mapping di un dominio con caratteri jolly (ad esempio, `*.contoso.com`) usando un record CNAME
> * Reindirizzare l'URL predefinito a una directory personalizzata
> * Automatizzare il mapping dei domini con script

## <a name="prerequisites"></a>Prerequisiti

Per completare questa esercitazione:

* [Creare un'app del servizio app](/azure/app-service/) oppure usare un'app creata per un'altra esercitazione.
* Acquistare un nome di dominio e assicurarsi di avere accesso al registro DNS per il provider del dominio, ad esempio GoDaddy.

  Ad esempio, per aggiungere le voci DNS per `contoso.com` e `www.contoso.com` è necessario essere autorizzati a configurare le impostazioni DNS per il dominio radice `contoso.com`.

  > [!NOTE]
  > Se non è disponibile un nome di dominio esistente, può essere utile [acquistare un dominio tramite il portale di Azure](manage-custom-dns-buy-domain.md).

## <a name="prepare-the-app"></a>Preparare l'app

Per eseguire il mapping di un nome DNS personalizzato a un'app Web, è necessario che il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) dell'app Web sia un livello a pagamento (**Condiviso**, **Base**, **Standard**, **Premium** o **A consumo** per le funzioni di Azure). In questo passaggio si verifica che l'app del servizio app si trovi nel piano tariffario supportato.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

### <a name="sign-in-to-azure"></a>Accedere ad Azure

Aprire il [portale di Azure](https://portal.azure.com) e accedere con l'account Azure.

### <a name="navigate-to-the-app-in-the-azure-portal"></a>Passare all'app nel portale di Azure

Dal menu a sinistra selezionare **Servizi app** e quindi selezionare il nome dell'app.

![Passaggio all'app di Azure nel portale](./media/app-service-web-tutorial-custom-domain/select-app.png)

Viene visualizzata la pagina di gestione dell'app del servizio app.  

<a name="checkpricing" aria-hidden="true"></a>

### <a name="check-the-pricing-tier"></a>Scegliere il piano tariffario

Nel riquadro di spostamento a sinistra della pagina dell'app scorrere fino alla sezione **Impostazioni** e selezionare **Scala verticalmente (piano di servizio app)** .

![Menu di scalabilità verticale](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

Il livello corrente dell'app è evidenziato da un bordo blu. Verificare che l'app non sia inclusa nel livello **F1**. Il DNS personalizzato non è supportato nel livello **F1**. 

![Controllare il piano tariffario](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

Se il piano di servizio app non è **F1**, chiudere la pagina **Aumenta** e passare a [Esecuzione del mapping di un record CNAME](#cname).

<a name="scaleup" aria-hidden="true"></a>

### <a name="scale-up-the-app-service-plan"></a>Passare a un piano di servizio app di livello superiore

Selezionare uno dei livelli non gratuiti (**D1**, **B1**, **B2**, **B3** o uno qualsiasi dei livelli della categoria **Produzione**). Per altre opzioni, fare clic su **Visualizza opzioni aggiuntive**.

Fare clic su **Apply**.

![Controllare il piano tariffario](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

La visualizzazione della notifica seguente indica che l'operazione di passaggio al livello superiore è stata completata.

![Conferma operazione di scalabilità](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

<a name="cname" aria-hidden="true"></a>

## <a name="map-your-domain"></a>Eseguire il mapping del dominio

È possibile utilizzare un **record CNAME** o un **record A** per eseguire il mapping di un nome DNS personalizzato con un servizio app. Seguire la procedura corrispondente:

- [Esecuzione del mapping di un record CNAME](#map-a-cname-record)
- [Esecuzione del mapping di un record A](#map-an-a-record)
- [Esecuzione del mapping di un dominio con caratteri jolly usando un record CNAME](#map-a-wildcard-domain)

> [!NOTE]
> Usare un record CNAME per tutti i nomi DNS personalizzati tranne i domini radice (ad esempio, `contoso.com`). Per i domini radice, usare i record A.

### <a name="map-a-cname-record"></a>Esecuzione del mapping di un record CNAME

nell'esempio dell'esercitazione si aggiunge un record CNAME per il sottodominio `www`, ad esempio `www.contoso.com`.

#### <a name="access-dns-records-with-domain-provider"></a>Accedere ai record DNS con il provider di dominio

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>Creazione di un record CNAME

Aggiungere un record CNAME per eseguire il mapping di un sottodominio al nome di dominio predefinito dell'app (`<app_name>.azurewebsites.net`, dove `<app_name>` è il nome dell'app).

Per l'esempio di dominio `www.contoso.com`, aggiungere un record CNAME che esegue il mapping del nome `www` a `<app_name>.azurewebsites.net`.

Dopo aver aggiunto il record CNAME, la pagina dei record DNS è simile all'esempio seguente:

![Passaggio all'app di Azure nel portale](./media/app-service-web-tutorial-custom-domain/cname-record.png)

#### <a name="enable-the-cname-record-mapping-in-azure"></a>Abilitare il mapping dei record CNAME in Azure

Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.

![Menu del dominio personalizzato](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

Nella pagina **Domini personalizzati** dell'app aggiungere il nome DNS personalizzato completo (`www.contoso.com`) all'elenco.

Selezionare l'icona **+** accanto ad **Aggiungi dominio personalizzato**.

![Aggiunta del nome host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

Digitare il nome di dominio completo per il quale è stato aggiunto un record CNAME, ad esempio `www.contoso.com`.

Selezionare **Convalida**.

Viene visualizzata la pagina **Aggiungi dominio personalizzato**.

Assicurarsi che **Tipo di record del nome host** sia impostato su **CNAME (www\.example.com o qualsiasi sottodominio**).

Selezionare **Aggiungi dominio personalizzato**.

![Aggiunta del nome DNS all'app](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

La visualizzazione del nuovo dominio personalizzato nella pagina **Domini personalizzati** dell'app potrebbe richiedere qualche minuto. Provare ad aggiornare il browser per visualizzare i dati più recenti.

![Record CNAME aggiunto](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

> [!NOTE]
> L'etichetta **Non sicuro** indica che il dominio personalizzato non è ancora associato a un certificato SSL e che, a seconda del browser in uso, per qualsiasi richiesta HTTPS inviata da un browser al dominio personalizzato verrà restituito un errore o un avviso. Per aggiungere un binding SSL, vedere [Associare un certificato SSL personalizzato esistente al Servizio app di Azure](app-service-web-tutorial-custom-ssl.md).

Se è stato saltato un passaggio o è stato inserito un errore di digitazione, nella parte inferiore della pagina viene visualizzato un errore di verifica.

![Errore di verifica](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

<a name="a" aria-hidden="true"></a>

### <a name="map-an-a-record"></a>Esecuzione del mapping di un record A

Nell'esempio dell'esercitazione si aggiunge un record A per il dominio radice, ad esempio `contoso.com`.

<a name="info"></a>

#### <a name="copy-the-apps-ip-address"></a>Copiare l'indirizzo IP dell'app

Per eseguire il mapping di un record A, è necessario l'indirizzo IP esterno dell'app. È possibile trovare questo indirizzo IP nella pagina **Domini personalizzati** dell'app nel portale di Azure.

Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.

![Menu del dominio personalizzato](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

Nella pagina **Domini personalizzati**, copiare l'indirizzo IP dell'applicazione.

![Passaggio all'app di Azure nel portale](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

#### <a name="access-dns-records-with-domain-provider"></a>Accedere ai record DNS con il provider di dominio

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-a-record"></a>Creazione di un record A

Per eseguire il mapping di un record A a un'app, il servizio app richiede **due** record DNS:

- Un record **A** di cui eseguire il mapping all'indirizzo IP dell'app.
- Un record **TXT** di cui eseguire il mapping al nome di dominio predefinito dell'app `<app_name>.azurewebsites.net`. Il servizio app usa questo record solo in fase di configurazione per verificare che si è proprietari del dominio personalizzato. Dopo che il dominio personalizzato è stato convalidato e configurato nel servizio app, è possibile eliminare il record TXT.

Per l’esempio di dominio `contoso.com`, creare i record A e TXT in base alla tabella seguente (`@` rappresenta in genere il dominio radice).

| Tipo di record | Host | Valore |
| - | - | - |
| Una | `@` | Indirizzo IP ricavato da [Copiare l'indirizzo IP dell'app](#info) |
| TXT | `@` | `<app_name>.azurewebsites.net` |

> [!NOTE]
> Per aggiungere un sottodominio (come `www.contoso.com`) usando un record A al posto di un [record CNAME](#map-a-cname-record) consigliato, il record A e il record TXT devono corrispondere invece alla tabella seguente:
>
> | Tipo di record | Host | Valore |
> | - | - | - |
> | Una | `www` | Indirizzo IP ricavato da [Copiare l'indirizzo IP dell'app](#info) |
> | TXT | `www` | `<app_name>.azurewebsites.net` |
>

Dopo aver aggiunto i record, la pagina dei record DNS è simile all'esempio seguente:

![Pagina dei record DNS](./media/app-service-web-tutorial-custom-domain/a-record.png)

<a name="enable-a" aria-hidden="true"></a>

#### <a name="enable-the-a-record-mapping-in-the-app"></a>Abilitare il mapping del record A nell'app

Nella pagina **Domini personalizzati** dell'app nel portale di Azure aggiungere il nome DNS personalizzato completo, ad esempio `contoso.com`, all'elenco.

Selezionare l'icona **+** accanto ad **Aggiungi dominio personalizzato**.

![Aggiunta del nome host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

Digitare il nome di dominio completo per il quale è stato configurato il record A, ad esempio `contoso.com`.

Selezionare **Convalida**.

Viene visualizzata la pagina **Aggiungi dominio personalizzato**.

Assicurarsi che **Tipo di record del nome host** sia impostato su **Record A (esempio.com)** .

Selezionare **Aggiungi dominio personalizzato**.

![Aggiunta del nome DNS all'app](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

La visualizzazione del nuovo dominio personalizzato nella pagina **Domini personalizzati** dell'app potrebbe richiedere qualche minuto. Provare ad aggiornare il browser per visualizzare i dati più recenti.

![Record A aggiunto](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

> [!NOTE]
> L'etichetta **Non sicuro** indica che il dominio personalizzato non è ancora associato a un certificato SSL e che, a seconda del browser in uso, per qualsiasi richiesta HTTPS inviata da un browser al dominio personalizzato verrà restituito un errore o un avviso. Per aggiungere un binding SSL, vedere [Associare un certificato SSL personalizzato esistente al Servizio app di Azure](app-service-web-tutorial-custom-ssl.md).

Se è stato saltato un passaggio o è stato inserito un errore di digitazione, nella parte inferiore della pagina viene visualizzato un errore di verifica.

![Errore di verifica](./media/app-service-web-tutorial-custom-domain/verification-error.png)

<a name="wildcard" aria-hidden="true"></a>

### <a name="map-a-wildcard-domain"></a>Esecuzione del mapping di un dominio con caratteri jolly

Nell'esempio dell'esercitazione si esegue il mapping di un [nome DNS con caratteri jolly](https://en.wikipedia.org/wiki/Wildcard_DNS_record), ad esempio `*.contoso.com`, all'app del servizio app aggiungendo un record CNAME.

#### <a name="access-dns-records-with-domain-provider"></a>Accedere ai record DNS con il provider di dominio

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>Creazione di un record CNAME

Aggiungere un record CNAME per eseguire il mapping di un nome con caratteri jolly al nome di dominio predefinito dell'app (`<app_name>.azurewebsites.net`).

Per l'esempio di dominio `*.contoso.com`, il record CNAME eseguirà il mapping del nome `*` a `<app_name>.azurewebsites.net`.

Dopo aver aggiunto il record CNAME, la pagina dei record DNS è simile all'esempio seguente:

![Passaggio all'app di Azure nel portale](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)

#### <a name="enable-the-cname-record-mapping-in-the-app"></a>Abilitare il mapping dei record CNAME nell'app

È ora possibile aggiungere qualsiasi sottodominio che corrisponde al nome con caratteri jolly nell'app, ad esempio `sub1.contoso.com` e `sub2.contoso.com` corrispondono a `*.contoso.com`.

Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.

![Menu del dominio personalizzato](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

Selezionare l'icona **+** accanto ad **Aggiungi dominio personalizzato**.

![Aggiunta del nome host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

Digitare un nome di dominio completo corrispondente al dominio con caratteri jolly, ad esempio `sub1.contoso.com`, quindi selezionare **Convalida**.

Il pulsante **Aggiungi dominio personalizzato** viene attivato.

Assicurarsi che **Tipo di record del nome host** sia impostato su **CNAME (www\.example.com o qualsiasi sottodominio**).

Selezionare **Aggiungi dominio personalizzato**.

![Aggiunta del nome DNS all'app](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

La visualizzazione del nuovo dominio personalizzato nella pagina **Domini personalizzati** dell'app potrebbe richiedere qualche minuto. Provare ad aggiornare il browser per visualizzare i dati più recenti.

Fare di nuovo clic sull'icona **+** per aggiungere un altro dominio personalizzato corrispondente al dominio con caratteri jolly. Ad esempio, aggiungere `sub2.contoso.com`.

![Record CNAME aggiunto](./media/app-service-web-tutorial-custom-domain/cname-record-added-wildcard2.png)

> [!NOTE]
> L'etichetta **Non sicuro** indica che il dominio personalizzato non è ancora associato a un certificato SSL e che, a seconda del browser in uso, per qualsiasi richiesta HTTPS inviata da un browser al dominio personalizzato verrà restituito un errore o un avviso. Per aggiungere un binding SSL, vedere [Associare un certificato SSL personalizzato esistente al Servizio app di Azure](app-service-web-tutorial-custom-ssl.md).

## <a name="test-in-browser"></a>Prova nel browser

Passare al nome o ai nomi DNS configurati in precedenza, ad esempio `contoso.com`,  `www.contoso.com`, `sub1.contoso.com` e `sub2.contoso.com`.

![Passaggio all'app di Azure nel portale](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

## <a name="resolve-404-not-found"></a>Risolvere l'errore 404 "Non trovato"

Se si riceve un errore HTTP 404 (non trovato) quando si seleziona l'URL del dominio personalizzato, verificare che il dominio venga risolto nell'indirizzo IP dell'app usando <a href="https://www.whatsmydns.net/" target="_blank">WhatsmyDNS.net</a>. In caso contrario, il problema può essere dovuto a una delle cause seguenti:

- Nel dominio personalizzato configurato mancano un record A e/o un record CNAME.
- Il browser client ha memorizzato nella cache l'indirizzo IP precedente del dominio. Cancellare la cache e testare nuovamente la risoluzione del DNS. In un computer Windows cancellare la cache con `ipconfig /flushdns`.

<a name="virtualdir" aria-hidden="true"></a>

## <a name="migrate-an-active-domain"></a>Eseguire la migrazione di un dominio attivo

Per eseguire la migrazione di un sito in tempo reale e del relativo nome di dominio DNS al Servizio app senza tempi di inattività, vedere [Eseguire la migrazione di un nome DNS attivo al Servizio app di Azure](manage-custom-dns-migrate-domain.md).

## <a name="redirect-to-a-custom-directory"></a>Reindirizzare a una directory personalizzata

Per impostazione predefinita, il servizio app indirizza le richieste Web alla directory radice del codice dell'app. Alcuni framework Web, tuttavia, non vengono avviati nella directory radice. Ad esempio, [Laravel](https://laravel.com/) viene avviato nella sottodirectory `public`. Per continuare l'esempio relativo al DNS `contoso.com`, un'app di questo tipo sarebbe accessibile all'indirizzo `http://contoso.com/public`, ma sarebbe di gran lunga preferibile indirizzare l'URL `http://contoso.com` alla directory `public`. Questa operazione non interessa la risoluzione del DNS, ma implica la personalizzazione della directory virtuale.

A questo scopo, selezionare **Impostazioni applicazione** nel riquadro di spostamento di sinistra della pagina dell'app Web. 

Nella parte inferiore della pagina la directory virtuale radice `/` punta per impostazione predefinita a `site\wwwroot`, che è la directory radice del codice dell'app. Modificare il percorso in modo che la directory punti, ad esempio, a `site\wwwroot\public` e salvare le modifiche.

![Personalizzare la directory virtuale](./media/app-service-web-tutorial-custom-domain/customize-virtual-directory.png)

Al termine dell'operazione, l'app dovrebbe restituire la pagina corretta nel percorso radice (ad esempio http://contoso.com).

## <a name="automate-with-scripts"></a>Automatizzazione con gli script

È possibile automatizzare la gestione dei domini personalizzati con gli script, usando l'[interfaccia della riga di comando di Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/overview). 

### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure 

Il seguente comando aggiunge un nome DNS personalizzato configurato a un'applicazione del servizio app. 

```bash 
az webapp config hostname add \
    --webapp-name <app_name> \
    --resource-group <resource_group_name> \
    --hostname <fully_qualified_domain_name>
``` 

Per altre informazioni, vedere [Eseguire il mapping di un dominio personalizzato a un'app Web](scripts/cli-configure-custom-domain.md).

### <a name="azure-powershell"></a>Azure PowerShell 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Il seguente comando aggiunge un nome DNS personalizzato configurato a un'applicazione del servizio app.

```powershell  
Set-AzWebApp `
    -Name <app_name> `
    -ResourceGroupName <resource_group_name> ` 
    -HostNames @("<fully_qualified_domain_name>","<app_name>.azurewebsites.net")
```

Per ulteriori informazioni, vedere [Assegnazione di un dominio personalizzato a un’app Web](scripts/powershell-configure-custom-domain.md).

## <a name="next-steps"></a>Passaggi successivi

Questa esercitazione illustra come:

> [!div class="checklist"]
> * Eseguire il mapping di un sottodominio usando un record CNAME
> * Eseguire il mapping di un dominio radice usando un record A
> * Eseguire il mapping di un dominio con caratteri jolly usando un record CNAME
> * Reindirizzare l'URL predefinito a una directory personalizzata
> * Automatizzare il mapping dei domini con script

Passare all'esercitazione successiva per apprendere come associare un certificato SSL personalizzato a un'app Web.

> [!div class="nextstepaction"]
> [Associare un certificato SSL personalizzato esistente al servizio app di Azure](app-service-web-tutorial-custom-ssl.md)
