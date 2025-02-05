---
title: Gestire le risorse del bus di servizio di Azure con PowerShell | Microsoft Docs
description: Usare il modulo di PowerShell per creare e gestire le risorse del bus di servizio
services: service-bus-messaging
documentationcenter: .NET
author: axisc
manager: timlt
editor: spelluru
ms.assetid: ''
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/21/2018
ms.author: aschhab
ms.openlocfilehash: 0d15aa4d7b8a922f7606b7c4d1b357a80b3cbfab
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60311047"
---
# <a name="use-powershell-to-manage-service-bus-resources"></a>Gestire le risorse del bus di servizio di Azure con PowerShell

Microsoft Azure PowerShell è un ambiente di scripting che può essere usato per controllare e automatizzare la distribuzione e la gestione dei servizi di Azure. L'articolo descrive come usare il [modulo PowerShell di Resource Manager del bus di servizio](/powershell/module/az.servicebus) per effettuare il provisioning e gestire le entità del bus di servizio (spazi dei nomi, code, argomenti e sottoscrizioni) tramite una console o uno script locale di Azure PowerShell.

È possibile gestire le risorse del bus di servizio anche usando i modelli di Azure Resource Manager. Per altre informazioni, vedere l'articolo [Creare risorse del bus di servizio usando i modelli di Azure Resource Manager](service-bus-resource-manager-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerequisiti

Prima di iniziare, verificare di disporre dei prerequisiti seguenti:

* Una sottoscrizione di Azure. Per altre informazioni su come ottenere una sottoscrizione, vedere le [opzioni di acquisto][purchase options], le [offerte per i membri][member offers] oppure l'[account gratuito][free account].
* Un computer con Azure PowerShell. Per le istruzioni vedere [Get started with Azure PowerShell cmdlets](/powershell/azure/get-started-azureps) (Introduzione ai cmdlet di Azure PowerShell).
* Conoscenza generale degli script di PowerShell, dei pacchetti NuGet e di .NET Framework.

## <a name="get-started"></a>Attività iniziali

Il primo passaggio consiste nell'usare PowerShell per accedere all'account Azure e alla sottoscrizione di Azure. Seguire le istruzioni in [Get started with Azure PowerShell cmdlets](/powershell/azure/get-started-azureps) (Introduzione ai cmdlet di Azure PowerShell) per accedere al proprio account Azure e recuperare e accedere alle risorse nella sottoscrizione di Azure.

## <a name="provision-a-service-bus-namespace"></a>Provisioning di uno spazio dei nomi del bus di servizio

Quando si usano spazi dei nomi del Bus di servizio, è possibile usare la [Get-AzServiceBusNamespace](/powershell/module/az.servicebus/get-azservicebusnamespace), [New-AzServiceBusNamespace](/powershell/module/az.servicebus/new-azservicebusnamespace), [Remove-AzServiceBusNamespace](/powershell/module/az.servicebus/remove-azservicebusnamespace)e [ Set-AzServiceBusNamespace](/powershell/module/az.servicebus/set-azservicebusnamespace) cmdlet.

Questo esempio crea alcune variabili locali nello script: `$Namespace` e `$Location`.

* `$Namespace` è il nome dello spazio dei nomi del bus di servizio che si vuole usare.
* `$Location` identifica il data center in cui si eseguirà il provisioning dello spazio dei nomi.
* `$CurrentNamespace` archivia lo spazio dei nomi di riferimento che viene recuperato (o creato).

In uno script effettivo, `$Namespace` e `$Location` possono essere passati come parametri.

Questa parte dello script esegue le operazioni seguenti:

1. Tenta di recuperare uno spazio dei nomi del bus di servizio con il nome specificato.
2. Se lo spazio dei nomi viene trovato, viene segnalato ciò che viene trovato.
3. Se lo spazio dei nomi non viene trovato, viene creato lo spazio dei nomi e quindi viene recuperato lo spazio dei nomi appena creato.
   
    ``` powershell
    # Query to see if the namespace currently exists
    $CurrentNamespace = Get-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
   
    # Check if the namespace already exists or needs to be created
    if ($CurrentNamespace)
    {
        Write-Host "The namespace $Namespace already exists in the $Location region:"
        # Report what was found
        Get-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
    }
    else
    {
        Write-Host "The $Namespace namespace does not exist."
        Write-Host "Creating the $Namespace namespace in the $Location region..."
        New-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace -Location $Location
        $CurrentNamespace = Get-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
        Write-Host "The $Namespace namespace in Resource Group $ResGrpName in the $Location region has been successfully created."
                
    }
    ```

### <a name="create-a-namespace-authorization-rule"></a>Crea una regola di autorizzazione dello spazio dei nomi

Nell'esempio seguente illustra come gestire le regole di autorizzazione dello spazio dei nomi usando il [New-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/new-azservicebusauthorizationrule), [Get-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/get-azservicebusauthorizationrule), [ Set-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/set-azservicebusauthorizationrule), e [Remove-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/remove-azservicebusauthorizationrule) cmdlet.

```powershell
# Query to see if rule exists
$CurrentRule = Get-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

# Check if the rule already exists or needs to be created
if ($CurrentRule)
{
    Write-Host "The $AuthRule rule already exists for the namespace $Namespace."
}
else
{
    Write-Host "The $AuthRule rule does not exist."
    Write-Host "Creating the $AuthRule rule for the $Namespace namespace..."
    New-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule -Rights @("Listen","Send")
    $CurrentRule = Get-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule
    Write-Host "The $AuthRule rule for the $Namespace namespace has been successfully created."

    Write-Host "Setting rights on the namespace"
    $authRuleObj = Get-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

    Write-Host "Remove Send rights"
    $authRuleObj.Rights.Remove("Send")
    Set-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

    Write-Host "Add Send and Manage rights to the namespace"
    $authRuleObj.Rights.Add("Send")
    Set-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj
    $authRuleObj.Rights.Add("Manage")
    Set-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

    Write-Host "Show value of primary key"
    $CurrentKey = Get-AzServiceBusKey -ResourceGroup $ResGrpName -NamespaceName $Namespace -Name $AuthRule
        
    Write-Host "Remove this authorization rule"
    Remove-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -Name $AuthRule
}
```

## <a name="create-a-queue"></a>Creare una coda

Per creare una coda o un argomento, eseguire una verifica dello spazio dei nomi usando lo script indicato nella sezione precedente. Creare quindi la coda:

```powershell
# Check if queue already exists
$CurrentQ = Get-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName

if($CurrentQ)
{
    Write-Host "The queue $QueueName already exists in the $Location region:"
}
else
{
    Write-Host "The $QueueName queue does not exist."
    Write-Host "Creating the $QueueName queue in the $Location region..."
    New-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -EnablePartitioning $True
    $CurrentQ = Get-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName
    Write-Host "The $QueueName queue in Resource Group $ResGrpName in the $Location region has been successfully created."
}
```

### <a name="modify-queue-properties"></a>Modificare le proprietà della coda

Dopo avere eseguito lo script nella sezione precedente, è possibile usare la [Set-AzServiceBusQueue](/powershell/module/az.servicebus/set-azservicebusqueue) cmdlet per aggiornare le proprietà di una coda, come nell'esempio seguente:

```powershell
$CurrentQ.DeadLetteringOnMessageExpiration = $True
$CurrentQ.MaxDeliveryCount = 7
$CurrentQ.MaxSizeInMegabytes = 2048
$CurrentQ.EnableExpress = $True

Set-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -QueueObj $CurrentQ
```

## <a name="provisioning-other-service-bus-entities"></a>Provisioning di altre entità del bus di servizio

È possibile usare il [modulo PowerShell del bus di servizio](/powershell/module/az.servicebus) per effettuare il provisioning di altre entità, ad esempio argomenti e sottoscrizioni. Questi cmdlet sono sintatticamente simili a quelli per la creazione della coda illustrati nella sezione precedente.

## <a name="next-steps"></a>Passaggi successivi

- Vedere la documentazione completa del modulo PowerShell di Resource Manager del bus di servizio [qui](/powershell/module/az.servicebus). Questa pagina elenca tutti i cmdlet disponibili.
- Per informazioni sull'uso dei modelli di Azure Resource Manager, vedere l'articolo [Creare risorse del bus di servizio usando i modelli di Azure Resource Manager](service-bus-resource-manager-overview.md).
- Informazioni sulle [librerie di gestione .NET del bus di servizio](service-bus-management-libraries.md).

Esistono alcune soluzioni alternative per la gestione delle entità del bus di servizio, come descritto in questi post di blog:

* [Come creare code, argomenti e sottoscrizioni del bus di servizio tramite uno script di PowerShell](https://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
* [Come creare uno spazio dei nomi del bus di servizio e un hub eventi tramite uno script PowerShell](https://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)
* [Script PowerShell del bus di servizio](https://code.msdn.microsoft.com/Service-Bus-PowerShell-a46b7059)

<!--Anchors-->

[purchase options]: https://azure.microsoft.com/pricing/purchase-options/
[member offers]: https://azure.microsoft.com/pricing/member-offers/
[free account]: https://azure.microsoft.com/pricing/free-trial/
