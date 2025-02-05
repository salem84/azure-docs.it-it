---
title: Gestire risorse di account con la libreria client per .NET - Azure Batch | Microsoft Docs
description: Creare, eliminare e modificare le risorse dell'account Batch tramite la libreria Batch Management .NET.
services: batch
documentationcenter: .net
author: laurenhughes
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 16279b23-60ff-4b16-b308-5de000e4c028
ms.service: batch
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/24/2017
ms.author: lahugh
ms.custom: seodec18
ms.openlocfilehash: f7554993e2e3d8d2f6bce71db57a746a4392ce1a
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70095080"
---
# <a name="manage-batch-accounts-and-quotas-with-the-batch-management-client-library-for-net"></a>Gestire le quote e gli account Batch con la libreria client di gestione Batch per .NET

> [!div class="op_single_selector"]
> * [Portale di Azure](batch-account-create-portal.md)
> * [.NET per la gestione di Batch](batch-management-dotnet.md)
> 
> 

È possibile ridurre il sovraccarico di manutenzione nelle applicazioni Azure Batch usando la libreria [batch Management .NET][api_mgmt_net] per automatizzare la creazione, l'eliminazione, la gestione delle chiavi e l'individuazione delle quote dell'account batch.

* **Creare ed eliminare account Batch** in qualsiasi area. Se, ad esempio, un fornitore di software indipendente (ISV) offre un servizio per cui a ogni cliente viene assegnato un account Batch separato per la fatturazione, è possibile aggiungere funzionalità di creazione ed eliminazione di account al portale per i clienti.
* **Recuperare e rigenerare chiavi di account** a livello di programmazione per qualsiasi account Batch. Questa operazione consente di conformarsi ai criteri di sicurezza che impongono rollover periodici o la scadenza delle chiavi dell'account. In presenza di numerosi account Batch in diverse aree di Azure, l'automazione del processo di rollover permette di aumentare l'efficienza della soluzione.
* **Controllare le quote degli account** e determinare senza errori i limiti dei singoli account Batch. Il controllo delle quote degli account prima di avviare processi, creare pool o aggiungere nodi di calcolo permette di stabilire attivamente dove e quando creare risorse di calcolo. È possibile determinare quali account richiedono un aumento della quota prima dell'allocazione di risorse aggiuntive in tali account.
* **Combina le funzionalità di altri servizi di Azure** per un'esperienza di gestione completa, usando batch Management .net, [Azure Active Directory][aad_about]e il [Azure Resource Manager][resman_overview] insieme nella stessa applicazione. Usando queste funzionalità e le relative API è possibile eliminare i problemi di autenticazione, consentire la creazione e l'eliminazione di gruppi di risorse e fornire le funzionalità descritte in precedenza per una soluzione di gestione end-to-end.

> [!NOTE]
> Sebbene questo articolo sia incentrato sulla gestione a livello di codice degli account, delle chiavi e delle quote di batch, è possibile eseguire molte di queste attività usando il [portale di Azure][azure_portal]. Per altre informazioni, vedere [Creare un account Azure Batch usando il portale di Azure](batch-account-create-portal.md) e [Quote e limiti per il servizio Azure Batch](batch-quota-limit.md).
> 
> 

## <a name="create-and-delete-batch-accounts"></a>Creare ed eliminare account Batch
Come accennato, una delle principali funzionalità dell'API di gestione per Batch è la creazione ed eliminazione di account Batch in un'area di Azure. A tale scopo, usare [BatchManagementClient. account. CreateAsync][net_create] e [DeleteAsync][net_delete]o le relative controparti sincrone.

Il frammento di codice seguente crea un account, ottiene l'account appena creato dal servizio Batch e quindi lo elimina. In questo frammento e gli altri in questo articolo `batchManagementClient` è un'istanza completamente inizializzata di [BatchManagementClient][net_mgmt_client].

```csharp
// Create a new Batch account
await batchManagementClient.Account.CreateAsync("MyResourceGroup",
    "mynewaccount",
    new BatchAccountCreateParameters() { Location = "West US" });

// Get the new account from the Batch service
AccountResource account = await batchManagementClient.Account.GetAsync(
    "MyResourceGroup",
    "mynewaccount");

// Delete the account
await batchManagementClient.Account.DeleteAsync("MyResourceGroup", account.Name);
```

> [!NOTE]
> Le applicazioni che usano la libreria Batch Management .NET e la relativa classe BatchManagementClient necessitano di accesso come **amministratore del servizio** o **coamministratore** alla sottoscrizione a cui appartiene l'account Batch da gestire. Per ulteriori informazioni, vedere la sezione Azure Active Directory e l'esempio di codice [AccountManagement][acct_mgmt_sample] .
> 
> 

## <a name="retrieve-and-regenerate-account-keys"></a>Recuperare e rigenerare chiavi di account
Ottenere chiavi di account primarie e secondarie da qualsiasi account batch all'interno della sottoscrizione usando [ListKeysAsync][net_list_keys]. È possibile rigenerare tali chiavi utilizzando [RegenerateKeyAsync][net_regenerate_keys].

```csharp
// Get and print the primary and secondary keys
BatchAccountListKeyResult accountKeys =
    await batchManagementClient.Account.ListKeysAsync(
        "MyResourceGroup",
        "mybatchaccount");
Console.WriteLine("Primary key:   {0}", accountKeys.Primary);
Console.WriteLine("Secondary key: {0}", accountKeys.Secondary);

// Regenerate the primary key
BatchAccountRegenerateKeyResponse newKeys =
    await batchManagementClient.Account.RegenerateKeyAsync(
        "MyResourceGroup",
        "mybatchaccount",
        new BatchAccountRegenerateKeyParameters() {
            KeyName = AccountKeyType.Primary
            });
```

> [!TIP]
> È possibile creare un flusso di lavoro di connessione ottimizzato per le applicazioni di gestione. Per prima cosa, ottenere una chiave account per l'account batch che si vuole gestire con [ListKeysAsync][net_list_keys]. Usare quindi questa chiave quando si inizializza la classe [BatchSharedKeyCredentials][net_sharedkeycred] della libreria batch .NET, che viene usata durante l'inizializzazione di [BatchClient][net_batch_client].
> 
> 

## <a name="check-azure-subscription-and-batch-account-quotas"></a>Verificare le quote di sottoscrizioni di Azure e account Batch
Le sottoscrizioni di Azure e i singoli servizi di Azure come Batch hanno tutti quote predefinite che limitano il numero di determinate entità al loro interno. Per informazioni sulle quote predefinite per le sottoscrizioni di Azure, vedere [Sottoscrizione di Azure e limiti, quote e vincoli dei servizi](../azure-subscription-service-limits.md). Per le quote predefinite del servizio Batch, vedere [Quote e limiti per il servizio Azure Batch](batch-quota-limit.md). Tramite la libreria Batch Management .NET è possibile controllare queste quote nelle applicazioni. In questo modo è possibile prendere decisioni relative all'allocazione prima di aggiungere account o risorse di calcolo come pool e nodi di calcolo.

### <a name="check-an-azure-subscription-for-batch-account-quotas"></a>Verificare le quote dell'account Batch di una sottoscrizione di Azure
Prima di creare un account Batch in un'area, è possibile verificare se la sottoscrizione di Azure permette di aggiungere un account in quell'area.

Nel frammento di codice riportato di seguito viene usato prima di tutto [BatchManagementClient. account. ListAsync][net_mgmt_listaccounts] per ottenere una raccolta di tutti gli account batch all'interno di una sottoscrizione. Una volta ottenuta questa raccolta, è possibile determinare il numero di account nell'area di destinazione. Si usa quindi [BatchManagementClient. Subscriptions][net_mgmt_subscriptions] per ottenere la quota dell'account batch e determinare il numero di account che è possibile creare in tale area.

```csharp
// Get a collection of all Batch accounts within the subscription
BatchAccountListResponse listResponse =
        await batchManagementClient.Account.ListAsync(new AccountListParameters());
IList<AccountResource> accounts = listResponse.Accounts;
Console.WriteLine("Total number of Batch accounts under subscription id {0}:  {1}",
    creds.SubscriptionId,
    accounts.Count);

// Get a count of all accounts within the target region
string region = "westus";
int accountsInRegion = accounts.Count(o => o.Location == region);

// Get the account quota for the specified region
SubscriptionQuotasGetResponse quotaResponse = await batchManagementClient.Subscriptions.GetSubscriptionQuotasAsync(region);
Console.WriteLine("Account quota for {0} region: {1}", region, quotaResponse.AccountQuota);

// Determine how many accounts can be created in the target region
Console.WriteLine("Accounts in {0}: {1}", region, accountsInRegion);
Console.WriteLine("You can create {0} accounts in the {1} region.", quotaResponse.AccountQuota - accountsInRegion, region);
```

Nel frammento di codice `creds` precedente, è un'istanza di [TokenCloudCredentials][azure_tokencreds]. Per un esempio di creazione di questo oggetto, vedere l'esempio di codice [AccountManagement][acct_mgmt_sample] su GitHub.

### <a name="check-a-batch-account-for-compute-resource-quotas"></a>Verificare le quote di risorse di calcolo in un account Batch
Prima di aumentare le risorse di calcolo nella soluzione Batch, è possibile verificare che le risorse da allocare non superino le quote dell'account. Nel frammento di codice riportato di seguito si procede alla stampa delle informazioni sulle quote per l'account Batch denominato `mybatchaccount`. Nell'applicazione è possibile usare queste informazioni per stabilire se l'account può gestire le risorse aggiuntive da creare.

```csharp
// First obtain the Batch account
BatchAccountGetResponse getResponse =
    await batchManagementClient.Account.GetAsync("MyResourceGroup", "mybatchaccount");
AccountResource account = getResponse.Resource;

// Now print the compute resource quotas for the account
Console.WriteLine("Core quota: {0}", account.Properties.CoreQuota);
Console.WriteLine("Pool quota: {0}", account.Properties.PoolQuota);
Console.WriteLine("Active job and job schedule quota: {0}", account.Properties.ActiveJobAndJobScheduleQuota);
```

> [!IMPORTANT]
> Sebbene esistano quote predefinite per le sottoscrizioni e i servizi di Azure, molti di questi limiti possono essere generati inviando una richiesta nel [portale di Azure][azure_portal]. Per istruzioni su come aumentare le quote dell'account Batch, vedere ad esempio [Quote e limiti per il servizio Azure Batch](batch-quota-limit.md) .
> 
> 

## <a name="use-azure-ad-with-batch-management-net"></a>Usare Azure AD con la gestione .NET per Batch

La libreria di gestione .NET per batch è un client del provider di risorse di Azure e viene usata insieme a [Azure Resource Manager][resman_overview] per gestire le risorse dell'account a livello di codice. Azure AD è necessario per autenticare le richieste effettuate tramite qualsiasi client del provider di risorse di Azure, tra cui la libreria .NET per la gestione di batch e la [Azure Resource Manager][resman_overview]. Per informazioni sull'uso di Azure AD con la libreria della gestione .NET per Batch, vedere [Usare Azure Active Directory per autenticare le soluzioni Batch](batch-aad-auth.md). 

## <a name="sample-project-on-github"></a>Progetto di esempio su GitHub

Per visualizzare batch Management .NET in azione, consultare il progetto di esempio [AccountManagement][acct_mgmt_sample] su GitHub. L'applicazione di esempio AccountManagement illustra le operazioni seguenti:

1. Acquisire un token di sicurezza da Azure AD usando [adal][aad_adal]. Se l'utente non ha già eseguito l'accesso, gli viene richiesto di fornire le credenziali di Azure.
2. Con il token di sicurezza ottenuto da Azure AD, creare una [SubscriptionClient][resman_subclient] per eseguire una query su Azure per ottenere un elenco di sottoscrizioni associate all'account. L'utente può selezionare una sottoscrizione dall'elenco se contiene più di una sottoscrizione.
3. Ottenere le credenziali associate alla sottoscrizione selezionata.
4. Creare un oggetto [ResourceManagementClient][resman_client] utilizzando le credenziali.
5. Usare un oggetto [ResourceManagementClient][resman_client] per creare un gruppo di risorse.
6. Usare un oggetto [BatchManagementClient][net_mgmt_client] per eseguire diverse operazioni dell'account batch:
   * Creare un account Batch nel nuovo gruppo di risorse.
   * Ottenere l'account appena creato dal servizio Batch.
   * Stampare le chiavi dell'account per il nuovo account.
   * Rigenerare una nuova chiave primaria per l'account.
   * Stampare le informazioni sulla quota per l'account.
   * Stampare le informazioni sulla quota per la sottoscrizione.
   * Stampare tutti gli account all'interno della sottoscrizione.
   * Eliminare l'account appena creato.
7. Eliminare il gruppo di risorse.

Prima di eliminare l'account batch e il gruppo di risorse appena creati, è possibile visualizzarli nel [portale di Azure][azure_portal]:

Per eseguire correttamente l'applicazione di esempio, è innanzitutto necessario registrarla con il tenant di Azure AD nel portale di Azure e concedere le autorizzazioni all'API di Azure Resource Manager. Seguire la procedura indicata in [Autenticare le soluzioni di gestione Batch con Active Directory](batch-aad-auth-management.md).


[aad_about]:../active-directory/fundamentals/active-directory-whatis.md "Informazioni su Azure Active Directory"
[aad_adal]: ../active-directory/active-directory-authentication-libraries.md
[aad_auth_scenarios]:../active-directory/develop/authentication-scenarios.md "Scenari di autenticazione per Azure AD"
[aad_integrate]:../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md "Integrazione di applicazioni con Azure Active Directory"
[acct_mgmt_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement
[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_mgmt_net]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[azure_portal]: https://portal.azure.com
[azure_storage]: https://azure.microsoft.com/services/storage/
[azure_tokencreds]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.tokencloudcredentials.aspx
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_list_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.listkeysasync.aspx
[net_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.createasync.aspx
[net_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.deleteasync.aspx
[net_regenerate_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.regeneratekeyasync.aspx
[net_sharedkeycred]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.auth.batchsharedkeycredentials.aspx
[net_mgmt_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.aspx
[net_mgmt_subscriptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.subscriptions.aspx
[net_mgmt_listaccounts]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.iaccountoperations.listasync.aspx
[resman_api]: https://msdn.microsoft.com/library/azure/mt418626.aspx
[resman_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx
[resman_subclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.subscriptions.subscriptionclient.aspx
[resman_overview]: ../azure-resource-manager/resource-group-overview.md

[1]: ./media/batch-management-dotnet/portal-01.png
[2]: ./media/batch-management-dotnet/portal-02.png
[3]: ./media/batch-management-dotnet/portal-03.png
