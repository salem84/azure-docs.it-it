---
title: Effettuare il provisioning di un'app Web con Cache Redis di Azure
description: Usare il modello di Azure Resource Manager per distribuire l'app Web con Cache Redis di Azure.
services: app-service
documentationcenter: ''
author: yegu-ms
manager: jhubbard
editor: ''
ms.assetid: 6e99c71f-ef8e-4570-a307-e4c059e60c35
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/06/2017
ms.author: yegu
ms.openlocfilehash: b28ed58159545bca10ec89375b82b9c97ae38630
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70098229"
---
# <a name="create-a-web-app-plus-azure-cache-for-redis-using-a-template"></a>Creare un'app Web e Cache Redis di Azure usando un modello

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

In questo argomento viene illustrato come creare un modello di Azure Resource Manager che consente di distribuire un'app Web di Azure con Cache Redis di Azure. Verrà illustrato come definire le risorse da distribuire e i parametri specificati quando viene eseguita la distribuzione. È possibile usare questo modello per le proprie distribuzioni o personalizzarlo in base alle esigenze.

Per altre informazioni sulla creazione dei modelli, vedere [Creazione di modelli di Gestione risorse di Azure](../azure-resource-manager/resource-group-authoring-templates.md). Per informazioni sulla sintassi e le proprietà JSON per i tipi di risorse cache, vedere [Tipi di risorse Microsoft.Cache](/azure/templates/microsoft.cache/allversions).

Per il modello completo, vedere [Web App with Azure Cache for Redis template](https://github.com/Azure/azure-quickstart-templates/blob/master/201-web-app-with-redis-cache/azuredeploy.json) (Modello di app Web con Cache Redis di Azure).

## <a name="what-you-will-deploy"></a>Elementi distribuiti
In questo modello, verrà distribuito quanto segue:

* App Web di Azure
* Cache Redis di Azure

Per eseguire automaticamente la distribuzione, fare clic sul pulsante seguente:

[![Distribuzione in Azure](./media/cache-web-app-arm-with-redis-cache-provision/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-web-app-with-redis-cache%2Fazuredeploy.json)

## <a name="parameters-to-specify"></a>Parametri da specificare
[!INCLUDE [app-service-web-deploy-web-parameters](../../includes/app-service-web-deploy-web-parameters.md)]

[!INCLUDE [cache-deploy-parameters](../../includes/cache-deploy-parameters.md)]

## <a name="variables-for-names"></a>Variabili per i nomi
Questo modello si serve di variabili per costruire i nomi delle risorse. Tramite la funzione [uniqueString](../azure-resource-manager/resource-group-template-functions-string.md#uniquestring) viene creato un valore in base all'ID del gruppo di risorse.

    "variables": {
      "hostingPlanName": "[concat('hostingplan', uniqueString(resourceGroup().id))]",
      "webSiteName": "[concat('webSite', uniqueString(resourceGroup().id))]",
      "cacheName": "[concat('cache', uniqueString(resourceGroup().id))]"
    },


## <a name="resources-to-deploy"></a>Risorse da distribuire
[!INCLUDE [app-service-web-deploy-web-host](../../includes/app-service-web-deploy-web-host.md)]

### <a name="azure-cache-for-redis"></a>Cache Redis di Azure
Crea l'istanza di Cache Redis di Azure che viene usata con l'app Web. Il nome della cache è specificato nella variabile **cacheName** .

Il modello crea la cache nella stessa posizione in cui si trova il gruppo di risorse.

    {
      "name": "[variables('cacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "dependsOn": [ ],
      "tags": {
        "displayName": "cache"
      },
      "properties": {
        "sku": {
          "name": "[parameters('cacheSKUName')]",
          "family": "[parameters('cacheSKUFamily')]",
          "capacity": "[parameters('cacheSKUCapacity')]"
        }
      }
    }


### <a name="web-app"></a>App Web
Crea l'app Web con il nome specificato nella variabile **webSiteName** .

Si noti che l'app Web è configurata con proprietà di impostazione dell'app che consentono di usare Cache Redis di Azure. Queste impostazioni dell'app vengono create dinamicamente in base ai valori forniti durante la distribuzione.

    {
      "apiVersion": "2015-08-01",
      "name": "[variables('webSiteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Web/serverFarms/', variables('hostingPlanName'))]",
        "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', variables('hostingPlanName'))]": "empty",
        "displayName": "Website"
      },
      "properties": {
        "name": "[variables('webSiteName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "type": "config",
          "name": "appsettings",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', variables('webSiteName'))]",
            "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
          ],
          "properties": {
            "CacheConnection": "[concat(variables('cacheName'),'.redis.cache.windows.net,abortConnect=false,ssl=true,password=', listKeys(resourceId('Microsoft.Cache/Redis', variables('cacheName')), '2015-08-01').primaryKey)]"
          }
        }
      ]
    }

## <a name="commands-to-run-deployment"></a>Comandi per eseguire la distribuzione
[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### <a name="powershell"></a>PowerShell
    New-AzResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-with-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup

### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure
    azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-with-redis-cache/azuredeploy.json -g ExampleDeployGroup
