---
title: Creare risorse del bus di servizio di Azure tramite modelli di Resource Manager | Microsoft Docs
description: Usare i modelli di Azure Resource Manager per automatizzare la creazione di risorse del bus di servizio
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 24f6a207-0fa4-49cf-8a58-963f9e2fd655
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 09/11/2018
ms.author: spelluru
ms.openlocfilehash: 196b00f1268eada20d0e35473dc6eb43c9e48df6
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "66111125"
---
# <a name="create-service-bus-resources-using-azure-resource-manager-templates"></a>Creare risorse del bus di servizio usando i modelli di Azure Resource Manager

Questo articolo descrive come creare e distribuire risorse del bus di servizio usando i modelli di Azure Resource Manager, PowerShell e il provider di risorse del bus di servizio.

I modelli di Azure Resource Manager aiutano a definire le risorse da distribuire per una soluzione e a specificare i parametri e le variabili che consentono di immettere i valori per diversi ambienti. Il modello, scritto in JSON, è costituito da espressioni che è possibile usare per creare valori per la distribuzione. Per informazioni dettagliate sulla creazione di modelli di Azure Resource Manager e per una descrizione del formato dei modelli, vedere [Comprendere la struttura e la sintassi dei modelli di Azure Resource Manger](../azure-resource-manager/resource-group-authoring-templates.md).

> [!NOTE]
> Gli esempi inclusi in questo articolo spiegano come usare Azure Resource Manager per creare uno spazio dei nomi del bus di servizio e una entità di messaggistica, ovvero una coda. Per altri esempi di modelli, visitare la [raccolta dei modelli di avvio rapido di Azure][Azure Quickstart Templates gallery] e cercare **bus di servizio**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="service-bus-resource-manager-templates"></a>Modelli di Resource Manager per il bus di servizio

I modelli di Azure Resource Manager del bus di servizio sono disponibili per il download e la distribuzione. Fare clic sui collegamenti seguenti per informazioni dettagliate su ognuno di essi, con collegamenti ai modelli su GitHub:

* [Creare uno spazio dei nomi del bus di servizio](service-bus-resource-manager-namespace.md)
* [Creare uno spazio dei nomi del bus di servizio con coda](service-bus-resource-manager-namespace-queue.md)
* [Creare uno spazio dei nomi del bus di servizio con argomento e sottoscrizione](service-bus-resource-manager-namespace-topic.md)
* [Creare uno spazio dei nomi del bus di servizio con coda e regola di autorizzazione](service-bus-resource-manager-namespace-auth-rule.md)
* [Creare uno spazio dei nomi del bus di servizio con argomento, sottoscrizione e regola](service-bus-resource-manager-namespace-topic-with-rule.md)

## <a name="deploy-with-powershell"></a>Distribuire con PowerShell

La procedura seguente illustra come usare PowerShell per distribuire un modello di Azure Resource Manager che crea uno spazio dei nomi del bus di servizio di livello Standard e una coda all'interno di tale spazio dei nomi. Questo esempio è basato sul modello di [Creare uno spazio dei nomi del bus di servizio con coda](https://github.com/Azure/azure-quickstart-templates/tree/master/201-servicebus-create-queue). Il flusso di lavoro è all'incirca il seguente:

1. Installare PowerShell.
2. Creare il modello e, facoltativamente, un file di parametri.
3. In PowerShell accedere all'account Azure.
4. Se non esiste, creare un nuovo gruppo di risorse.
5. Testare la distribuzione.
6. Facoltativamente, è possibile impostare la modalità di distribuzione.
7. Distribuire il modello.

Per informazioni più complete sulla distribuzione dei modelli di Azure Resource Manager, vedere [Distribuire le risorse con i modelli di Azure Resource Manager][Deploy resources with Azure Resource Manager templates].

### <a name="install-powershell"></a>Installare PowerShell

Installare Azure PowerShell seguendo le istruzioni riportate in [Introduzione ai cmdlet di Azure PowerShell](/powershell/azure/get-started-azureps).

### <a name="create-a-template"></a>Creare un modello

Clonare il repository o copiare il modello [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.json) da GitHub:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serviceBusNamespaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Service Bus namespace"
      }
    },
    "serviceBusQueueName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Queue"
      }
    }
  },
  "variables": {
    "defaultSASKeyName": "RootManageSharedAccessKey",
    "authRuleResourceId": "[resourceId('Microsoft.ServiceBus/namespaces/authorizationRules', parameters('serviceBusNamespaceName'), variables('defaultSASKeyName'))]",
    "sbVersion": "2017-04-01"
  },
  "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('serviceBusNamespaceName')]",
      "type": "Microsoft.ServiceBus/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {},
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('serviceBusQueueName')]",
          "type": "Queues",
          "dependsOn": [
            "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
          ],
          "properties": {
            "lockDuration": "PT5M",
            "maxSizeInMegabytes": "1024",
            "requiresDuplicateDetection": "false",
            "requiresSession": "false",
            "defaultMessageTimeToLive": "P10675199DT2H48M5.4775807S",
            "deadLetteringOnMessageExpiration": "false",
            "duplicateDetectionHistoryTimeWindow": "PT10M",
            "maxDeliveryCount": "10",
            "autoDeleteOnIdle": "P10675199DT2H48M5.4775807S",
            "enablePartitioning": "false",
            "enableExpress": "false"
          }
        }
      ]
    }
  ],
  "outputs": {
    "NamespaceConnectionString": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryConnectionString]"
    },
    "SharedAccessPolicyPrimaryKey": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryKey]"
    }
  }
}
```

### <a name="create-a-parameters-file-optional"></a>Creare un file di parametri (facoltativo)

Per usare un file di parametri facoltativo, copiare il file [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.parameters.json). Sostituire il valore di `serviceBusNamespaceName` con il nome dello spazio dei nomi del bus di servizio che si vuole creare in questa distribuzione e sostituire il valore di `serviceBusQueueName` con il nome della coda che si vuole creare.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "serviceBusNamespaceName": {
            "value": "<myNamespaceName>"
        },
        "serviceBusQueueName": {
            "value": "<myQueueName>"
        },
        "serviceBusApiVersion": {
            "value": "2017-04-01"
        }
    }
}
```

Per altre informazioni, vedere l'articolo [Parametri](../azure-resource-manager/resource-group-template-deploy.md#parameter-files).

### <a name="log-in-to-azure-and-set-the-azure-subscription"></a>Accedere ad Azure e impostare la sottoscrizione di Azure

Al prompt di PowerShell, eseguire il comando seguente:

```powershell
Connect-AzAccount
```

Il sistema chiede di accedere all'account Azure. Dopo l'accesso, eseguire il comando seguente per visualizzare le sottoscrizioni disponibili:

```powershell
Get-AzSubscription
```

Questo comando restituisce un elenco delle sottoscrizioni di Azure disponibili. Scegliere una sottoscrizione per la sessione corrente eseguendo il comando seguente. Sostituire `<YourSubscriptionId>` con il GUID della sottoscrizione di Azure che si vuole usare:

```powershell
Set-AzContext -SubscriptionID <YourSubscriptionId>
```

### <a name="set-the-resource-group"></a>Impostare il gruppo di risorse

Se non è una risorsa esistente di gruppo, creare un nuovo gruppo di risorse con il **New-AzResourceGroup** comando. Specificare il nome del gruppo di risorse e la posizione che si vuole usare, Ad esempio:

```powershell
New-AzResourceGroup -Name MyDemoRG -Location "West US"
```

Se il nuovo gruppo di risorse è stato creato correttamente, viene visualizzato il relativo riepilogo.

```powershell
ResourceGroupName : MyDemoRG
Location          : westus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
```

### <a name="test-the-deployment"></a>Test della distribuzione

Convalidare la distribuzione eseguendo il cmdlet `Test-AzResourceGroupDeployment`. Durante il test della distribuzione, specificare esattamente gli stessi parametri di quando si esegue la distribuzione.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="create-the-deployment"></a>Creare la distribuzione

Per creare la nuova distribuzione, eseguire il cmdlet `New-AzResourceGroupDeployment` e specificare i parametri necessari quando viene richiesto. I parametri includono il nome della distribuzione, il nome del gruppo di risorse e il percorso o l'URL del file di modello. Se il parametro **Mode** non è specificato, viene usato il valore predefinito **Incremental**. Per altre informazioni, vedere [Distribuzioni incrementali e complete](../azure-resource-manager/deployment-modes.md).

Il comando seguente richiede tre parametri nella finestra di PowerShell:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

Per specificare invece un file di parametri, usare il comando seguente:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
```

È anche possibile usare i parametri inline quando si esegue il cmdlet di distribuzione. Il comando è il seguente:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
```

Per eseguire una distribuzione [completa](../azure-resource-manager/deployment-modes.md), impostare il parametro **Mode** su **Complete**:

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="verify-the-deployment"></a>Verificare la distribuzione
Se le risorse vengono distribuite correttamente, nella finestra di PowerShell viene visualizzato il riepilogo della distribuzione:

```powershell
DeploymentName    : MyDemoDeployment
ResourceGroupName : MyDemoRG
ProvisioningState : Succeeded
Timestamp         : 4/19/2017 10:38:30 PM
Mode              : Incremental
TemplateLink      :
Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    serviceBusNamespaceName  String             <namespaceName>
                    serviceBusQueueName  String                 <queueName>
                    serviceBusApiVersion  String                2017-04-01

```

## <a name="next-steps"></a>Passaggi successivi
Finora sono stati illustrati il flusso di lavoro di base e i comandi per la distribuzione di un modello di Azure Resource Manager. Per informazioni più dettagliate, visitare i collegamenti seguenti:

* [Panoramica di Azure Resource Manager][Azure Resource Manager overview]
* [Distribuire le risorse con i modelli di Azure Resource Manager e Azure PowerShell][Deploy resources with Azure Resource Manager templates]
* [Creazione di modelli di Azure Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md)
* [Tipi di risorsa Microsoft.ServiceBus](/azure/templates/microsoft.servicebus/allversions)

[Azure Resource Manager overview]: ../azure-resource-manager/resource-group-overview.md
[Deploy resources with Azure Resource Manager templates]: ../azure-resource-manager/resource-group-template-deploy.md
[Azure Quickstart Templates gallery]: https://azure.microsoft.com/documentation/templates/?term=service+bus
