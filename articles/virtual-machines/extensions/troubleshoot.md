---
title: Risoluzione degli errori delle estensioni di macchina virtuale Windows | Microsoft Docs
description: Informazioni sulla risoluzione degli errori delle estensioni della macchina virtuale Windows di Azure
services: virtual-machines-windows
documentationcenter: ''
author: kundanap
manager: gwallace
editor: ''
tags: top-support-issue,azure-resource-manager
ms.assetid: 878ab9b6-c3e6-40be-82d4-d77fecd5030f
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/29/2016
ms.author: kundanap
ms.openlocfilehash: 6108b3c8d4f4c5c93bc2543042294bc8167d4dd9
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70084299"
---
# <a name="troubleshooting-azure-windows-vm-extension-failures"></a>Risoluzione degli errori delle estensioni di macchina virtuale Windows di Azure
[!INCLUDE [virtual-machines-common-extensions-troubleshoot](../../../includes/virtual-machines-common-extensions-troubleshoot.md)]

## <a name="viewing-extension-status"></a>Visualizzazione dello stato dell'estensione
I modelli di Azure Resource Manager possono essere eseguiti da Azure PowerShell. Una volta che il modello viene eseguito, sarà possibile visualizzare lo stato dell'estensione da Azure Resource Explorer o dagli strumenti da riga di comando.

Di seguito è fornito un esempio:

Azure PowerShell:

      Get-AzVM -ResourceGroupName $RGName -Name $vmName -Status

Di seguito è riportato l'output di esempio:

      Extensions:  {
      "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
      "Name": "myCustomScriptExtension",
      "SubStatuses": [
        {
          "Code": "ComponentStatus/StdOut/succeeded",
          "DisplayStatus": "Provisioning succeeded",
          "Level": "Info",
          "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
              \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
              test.txt                          \\n\\n",
                      "Time": null
          },
        {
          "Code": "ComponentStatus/StdErr/succeeded",
          "DisplayStatus": "Provisioning succeeded",
          "Level": "Info",
          "Message": "",
          "Time": null
        }
    }
  ]

## <a name="troubleshooting-extension-failures"></a>Risoluzione degli errori delle estensioni
### <a name="rerun-the-extension-on-the-vm"></a>Eseguire nuovamente l'estensione nella macchina virtuale
Se si eseguono gli script nella macchina virtuale usando l'estensione script personalizzato, è possibile riscontrare in alcuni casi un errore in cui la creazione della macchina virtuale è riuscita, ma lo script ha avuto esito negativo. In queste condizioni, il metodo consigliato per risolvere il problema consiste nel rimuovere l'estensione ed eseguire nuovamente il modello.
Nota: in futuro, questa funzionalità potrebbe essere migliorata in modo da eliminare la necessità di disinstallazione dell'estensione.

#### <a name="remove-the-extension-from-azure-powershell"></a>Rimuovere l'estensione da Azure PowerShell
    Remove-AzVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"

Una volta rimossa l'estensione, il modello può essere eseguito nuovamente per eseguire gli script nella macchina virtuale.

