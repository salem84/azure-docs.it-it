---
title: Creare un'area di lavoro Log Analytics usando Azure PowerShell | Microsoft Docs
description: Informazioni su come creare un'area di lavoro Log Analytics per abilitare soluzioni di gestione e la raccolta dei dati dagli ambienti cloud e locali con Azure PowerShell.
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 03/12/2019
ms.author: magoedte
ms.openlocfilehash: b81495f19ce596d689778e6ab75512d744ae4588
ms.sourcegitcommit: 15e3bfbde9d0d7ad00b5d186867ec933c60cebe6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/03/2019
ms.locfileid: "71836563"
---
# <a name="create-a-log-analytics-workspace-with-azure-powershell"></a>Creare un'area di lavoro Log Analytics con Azure PowerShell

Il modulo Azure PowerShell viene usato per creare e gestire le risorse di Azure dalla riga di comando di PowerShell o negli script. Questo argomento di avvio rapido illustra come usare il modulo Azure PowerShell per distribuire un'area di lavoro Log Analytics in Monitoraggio di Azure. Un'area di lavoro Log Analytics è un ambiente univoco per i dati di log di Monitoraggio di Azure. Ogni area di lavoro ha un proprio repository di dati e una propria configurazione, mentre le origini dati e le soluzioni sono configurate per l'archiviazione dei dati in un'area di lavoro specifica. È necessaria un'area di lavoro Log Analytics se si intende raccogliere dati dalle origini seguenti:

* Risorse di Azure nella sottoscrizione  
* Computer locali monitorati tramite System Center Operations Manager  
* Raccolte di dispositivi da System Center Configuration Manager  
* Dati di diagnostica o dei log dall'archiviazione di Azure  
 
Per altre origini, ad esempio macchine virtuali di Azure e macchine virtuali di Windows o Linux presenti nell'ambiente, vedere gli argomenti seguenti:

* [Raccogliere dati dalle macchine virtuali di Azure](../learn/quick-collect-azurevm.md)
* [Raccogliere dati dal computer Linux ibrido](../learn/quick-collect-linux-computer.md)
* [Raccogliere dati dal computer Windows ibrido](quick-collect-windows-computer.md)

Se non si ha una sottoscrizione di Azure, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Se si sceglie di installare e usare PowerShell in locale, per questa esercitazione è necessario il Azure PowerShell AZ Module. Eseguire `Get-Module -ListAvailable Az` per trovare la versione. Se è necessario eseguire l'aggiornamento, vedere [Installare e configurare Azure PowerShell](/powershell/azure/install-az-ps). Se si esegue PowerShell in locale, è anche necessario eseguire `Connect-AzAccount` per creare una connessione con Azure.

## <a name="create-a-workspace"></a>Crea un'area di lavoro
Creare un'area di lavoro con [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). Nell'esempio seguente viene creata un'area di lavoro nella località *eastus* usando un modello di gestione risorse dal computer locale. Il modello JSON è configurato in modo da richiedere solo il nome dell'area di lavoro e specifica un valore predefinito per gli altri parametri che potrebbero essere usati come configurazione standard nell'ambiente in uso. 

Per informazioni sulle aree supportate, vedere [aree log Analytics è disponibile in](https://azure.microsoft.com/regions/services/) e cercare monitoraggio di Azure dal campo **ricerca di un prodotto** . 

I parametri seguenti impostano un valore predefinito:

* location: il valore predefinito è Stati Uniti orientali
* sku: il valore predefinito è il nuovo piano tariffario per GB rilasciato nel modello di prezzi di aprile 2018

>[!WARNING]
>Se si crea o si configura un'area di lavoro Log Analytics in una sottoscrizione basata sul nuovo modello di prezzi di aprile 2018, l'unico piano tariffario di Log Analytics valido è **PerGB2018**. 
>

### <a name="create-and-deploy-template"></a>Creare e distribuire il modello

1. Copiare e incollare nel file la sintassi JSON seguente:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "eastus",
              "westus"
            ],
            "defaultValue": "eastus",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        },
        "sku": {
            "type": "String",
            "allowedValues": [
              "Standalone",
              "PerNode",
              "PerGB2018"
            ],
            "defaultValue": "PerGB2018",
            "metadata": {
            "description": "Specifies the service tier of the workspace: Standalone, PerNode, Per-GB"
        }
          }
    },
    "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "sku": {
                    "Name": "[parameters('sku')]"
                },
                "features": {
                    "searchVersion": 1
                }
            }
          }
       ]
    }
    ```

2. Modificare il modello in base alle esigenze. Rivedere il riferimento del [modello Microsoft.OperationalInsights/workspaces](https://docs.microsoft.com/azure/templates/microsoft.operationalinsights/workspaces) per informazioni sulle proprietà e sui valori supportati. 
3. Salvare questo file come **deploylaworkspacetemplate.json** in una cartella locale.   
4. A questo punto è possibile distribuire il modello. Usare i comandi seguenti dalla cartella che contiene il modello. Quando viene richiesto un nome dell'area di lavoro, specificare un nome univoco a livello globale in tutte le sottoscrizioni di Azure.

    ```powershell
        New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile deploylaworkspacetemplate.json
    ```

Per il completamento della distribuzione sarà necessario attendere alcuni minuti. Al termine, viene visualizzato un messaggio simile al seguente che include il risultato:

![Esempio di risultato al termine della distribuzione](media/quick-create-workspace-posh/template-output-01.png)

## <a name="next-steps"></a>Passaggi successivi
Dopo aver creato un'area di lavoro, è possibile configurare la raccolta di dati di telemetria di monitoraggio, eseguire ricerche nei log per analizzare i dati e aggiungere una soluzione di gestione per fornire informazioni analitiche dettagliate e dati aggiuntivi.  

* Per abilitare la raccolta dati dalle risorse di Azure con Diagnostica di Azure o Archiviazione di Azure, vedere [Raccolta di log e metriche per i servizi di Azure da usare in Monitoraggio di Azure](../platform/collect-azure-metrics-logs.md).  
* Aggiungere [System Center Operations Manager come origine dati](../platform/om-agents.md) per raccogliere i dati da agenti di creazione report per il gruppo di gestione Operations Manager e archiviarli nell'area di lavoro Log Analytics.  
* Connettere [Configuration Manager](../platform/collect-sccm.md) per importare computer che sono membri di raccolte nella gerarchia.  
* Esaminare le [soluzioni di monitoraggio](../insights/solutions.md) disponibili e verificare come aggiungere o rimuovere una soluzione dall'area di lavoro.
