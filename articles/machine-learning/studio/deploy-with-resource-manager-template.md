---
title: Distribuire l'area di lavoro di Studio con Azure Resource Manager
titleSuffix: Azure Machine Learning Studio
description: Come distribuire un'area di lavoro per Azure Machine Learning Studio usando il modello di Azure Resource Manager
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: xiaoharper
ms.author: amlstudiodocs
ms.custom: seodec18
ms.date: 02/05/2018
ms.openlocfilehash: 91413aa461261824782717ae4edacc2757ad5405
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "66121313"
---
# <a name="deploy-azure-machine-learning-studio-workspace-using-azure-resource-manager"></a>Distribuire un'area di lavoro di Azure Machine Learning Studio con Azure Resource Manager

L'uso di un modello di distribuzione Azure Resource Manager consente di risparmiare tempo perché è possibile distribuire in modo scalabile i componenti interconnessi con un meccanismo di convalida e di ripetizione dei tentativi. Per configurare le aree di lavoro di Azure Machine Learning Studio, ad esempio, è necessario configurare prima un account di archiviazione di Azure e quindi distribuire l'area di lavoro. Si immagini di doverlo fare manualmente per centinaia di aree di lavoro. Un'alternativa più semplice prevede l'uso di un modello di Azure Resource Manager per distribuire un'area di lavoro di Studio e tutte le dipendenze. Questo articolo illustra il processo in dettaglio. Per una panoramica generale di Azure Resource Manager, vedere [Panoramica di Azure Resource Manager](../../azure-resource-manager/resource-group-overview.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="step-by-step-create-a-machine-learning-workspace"></a>Procedura dettagliata: Creare un'area di lavoro di Machine Learning
Verrà creato un gruppo di risorse di Azure, quindi verranno distribuiti un nuovo account di archiviazione di Azure e una nuova area di lavoro di Azure Machine Learning Studio usando un modello di Resource Manager. Una volta completata la distribuzione, verranno visualizzate importanti informazioni sulle aree di lavoro create (la chiave primaria, l'ID area di lavoro e l'URL dell'area di lavoro).

### <a name="create-an-azure-resource-manager-template"></a>Creare un modello di Azure Resource Manager

Un'area di lavoro di Machine Learning richiede un account di archiviazione di Azure per archiviare il set di dati collegato.
Il modello seguente usa il nome del gruppo di risorse per generare il nome dell'account di archiviazione e il nome dell'area di lavoro.  Usa anche il nome dell'account di archiviazione come proprietà durante la creazione dell'area di lavoro.

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "variables": {
        "namePrefix": "[resourceGroup().name]",
        "location": "[resourceGroup().location]",
        "mlVersion": "2016-04-01",
        "stgVersion": "2015-06-15",
        "storageAccountName": "[concat(variables('namePrefix'),'stg')]",
        "mlWorkspaceName": "[concat(variables('namePrefix'),'mlwk')]",
        "mlResourceId": "[resourceId('Microsoft.MachineLearning/workspaces', variables('mlWorkspaceName'))]",
        "stgResourceId": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "storageAccountType": "Standard_LRS"
    },
    "resources": [
        {
            "apiVersion": "[variables('stgVersion')]",
            "name": "[variables('storageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[variables('location')]",
            "properties": {
                "accountType": "[variables('storageAccountType')]"
            }
        },
        {
            "apiVersion": "[variables('mlVersion')]",
            "type": "Microsoft.MachineLearning/workspaces",
            "name": "[variables('mlWorkspaceName')]",
            "location": "[variables('location')]",
            "dependsOn": ["[variables('stgResourceId')]"],
            "properties": {
                "UserStorageAccountId": "[variables('stgResourceId')]"
            }
        }
    ],
    "outputs": {
        "mlWorkspaceObject": {"type": "object", "value": "[reference(variables('mlResourceId'), variables('mlVersion'))]"},
        "mlWorkspaceToken": {"type": "string", "value": "[listWorkspaceKeys(variables('mlResourceId'), variables('mlVersion')).primaryToken]"},
        "mlWorkspaceWorkspaceID": {"type": "string", "value": "[reference(variables('mlResourceId'), variables('mlVersion')).WorkspaceId]"},
        "mlWorkspaceWorkspaceLink": {"type": "string", "value": "[concat('https://studio.azureml.net/Home/ViewWorkspace/', reference(variables('mlResourceId'), variables('mlVersion')).WorkspaceId)]"}
    }
}

```
Salvare questo modello come file mlworkspace.json in c:\temp\.

### <a name="deploy-the-resource-group-based-on-the-template"></a>Distribuire il gruppo di risorse in base al modello

* Aprire PowerShell
* Installare i moduli per Azure Resource Manager e Azure Service Management

```powershell
# Install the Azure Resource Manager modules from the PowerShell Gallery (press “A”)
Install-Module Az -Scope CurrentUser

# Install the Azure Service Management modules from the PowerShell Gallery (press “A”)
Install-Module Azure -Scope CurrentUser
```

   Con questi passaggi vengono scaricati e installati i moduli necessari per completare i passaggi rimanenti. È necessario eseguirli una sola volta nell'ambiente in cui si eseguono i comandi di PowerShell.

* Eseguire l'autenticazione ad Azure

```powershell
# Authenticate (enter your credentials in the pop-up window)
Connect-AzAccount
```
Questo passaggio deve essere ripetuto per ogni sessione. Una volta eseguita l'autenticazione, verranno visualizzate le informazioni sulla sottoscrizione.

![Account Azure](./media/deploy-with-resource-manager-template/azuresubscription.png)

Ora che si ha accesso ad Azure, è possibile creare il gruppo di risorse.

* Creare un gruppo di risorse

```powershell
$rg = New-AzResourceGroup -Name "uniquenamerequired523" -Location "South Central US"
$rg
```

Verificare che il provisioning del gruppo di risorse venga effettuato correttamente. **ProvisioningState** deve essere "Succeeded".
Il nome del gruppo di risorse viene usato dal modello per generare il nome dell'account di archiviazione. Il nome dell'account di archiviazione deve essere di lunghezza compresa tra 3 e 24 caratteri e usare solo numeri e lettere minuscole.

![Gruppo di risorse](./media/deploy-with-resource-manager-template/resourcegroupprovisioning.png)

* Usando la distribuzione del gruppo di risorse, distribuire una nuova area di lavoro di Machine Learning.

```powershell
# Create a Resource Group, TemplateFile is the location of the JSON template.
$rgd = New-AzResourceGroupDeployment -Name "demo" -TemplateFile "C:\temp\mlworkspace.json" -ResourceGroupName $rg.ResourceGroupName
```

Una volta completata la distribuzione, è semplice accedere alle proprietà dell'area di lavoro distribuita. Ad esempio, è possibile accedere al token di chiave primaria.

```powershell
# Access Azure Machine Learning studio Workspace Token after its deployment.
$rgd.Outputs.mlWorkspaceToken.Value
```

Un altro modo per recuperare i token dell'area di lavoro esistente consiste nell'utilizzare il comando Invoke-AzResourceAction. Ad esempio, è possibile elencare i token primari e secondari di tutte le aree di lavoro.

```powershell
# List the primary and secondary tokens of all workspaces
Get-AzResource |? { $_.ResourceType -Like "*MachineLearning/workspaces*"} |ForEach-Object { Invoke-AzResourceAction -ResourceId $_.ResourceId -Action listworkspacekeys -Force}
```
Dopo il provisioning dell'area di lavoro, è anche possibile automatizzare diverse attività di Azure Machine Learning Studio usando il [modulo PowerShell per Azure Machine Learning Studio](https://aka.ms/amlps).

## <a name="next-steps"></a>Passaggi successivi

* Altre informazioni sulla [Creazione di modelli di Azure Resource Manager](../../azure-resource-manager/resource-group-authoring-templates.md).
* Vedere il [repository di modelli di avvio rapido di Azure](https://github.com/Azure/azure-quickstart-templates).
* Guardare questo video su [Azure Resource Manager](https://channel9.msdn.com/Events/Ignite/2015/C9-39).
* Vedere la [guida di riferimento dei modelli di Resource Manager](https://docs.microsoft.com/azure/templates/microsoft.machinelearning/allversions)

<!--Link references-->
