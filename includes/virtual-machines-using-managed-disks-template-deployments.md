---
title: File di inclusione
description: File di inclusione
services: storage
author: jboeshart
ms.service: storage
ms.topic: include
ms.date: 06/05/2018
ms.author: jaboes
ms.custom: include file
ms.openlocfilehash: 904bd884bc09c1e2016f55ffc8e1e9f635974ac7
ms.sourcegitcommit: 3e98da33c41a7bbd724f644ce7dedee169eb5028
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/18/2019
ms.locfileid: "67180082"
---
# <a name="using-managed-disks-in-azure-resource-manager-templates"></a>Uso di Managed Disks nei modelli di Azure Resource Manager

Questo documento descrive le differenze tra dischi gestiti e non gestiti quando si usano i modelli di Azure Resource Manager per eseguire il provisioning di macchine virtuali. Gli esempi sono utili per aggiornare i modelli esistenti con dischi non gestiti per l'uso dei dischi gestiti. A titolo di riferimento, come guida viene usato il modello [101-vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows). Per un eventuale confronto diretto, è possibile visualizzare sia il modello per l'uso di [dischi gestiti](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows/azuredeploy.json), sia una versione precedente per [dischi non gestiti](https://github.com/Azure/azure-quickstart-templates/tree/93b5f72a9857ea9ea43e87d2373bf1b4f724c6aa/101-vm-simple-windows/azuredeploy.json).

## <a name="unmanaged-disks-template-formatting"></a>Formattazione del modello per dischi non gestiti

Per iniziare, viene esaminata la modalità di distribuzione dei dischi non gestiti. Quando si creano dischi non gestiti, è necessario un account di archiviazione in cui salvare i file VHD. È possibile creare un nuovo account di archiviazione o usarne uno già esistente. Questo articolo illustra come creare un nuovo account di archiviazione. Creare una risorsa account di archiviazione nel blocco delle risorse, come illustrato di seguito.

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2018-07-01",
    "name": "[variables('storageAccountName')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "kind": "Storage",
    "properties": {}
}
```

All'interno dell'oggetto macchina virtuale aggiungere una dipendenza dall'account di archiviazione, per assicurarsi che venga creato prima della macchina virtuale. All'interno della sezione `storageProfile` specificare l'URI completo del percorso del disco rigido virtuale, che fa riferimento all'account di archiviazione ed è necessario per il disco del sistema operativo ed eventuali dischi dati.

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2018-10-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "name": "osdisk",
                "vhd": {
                    "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
                },
                "caching": "ReadWrite",
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "name": "datadisk1",
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "vhd": {
                        "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/datadisk1.vhd')]"
                    },
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

## <a name="managed-disks-template-formatting"></a>Formattazione del modello per dischi gestiti

Con Azure Managed Disks, il disco diventa una risorsa di primo livello e non richiede più la creazione di un account di archiviazione da parte dell'utente. I dischi gestiti sono stati prima esposti nella versione `2016-04-30-preview` dell'API, sono disponibili in tutte le versioni API successive e sono ora il tipo di disco predefinito. Le sezioni seguenti descrivono in modo dettagliato le impostazioni predefinite e illustrano come personalizzare ulteriormente i dischi.

> [!NOTE]
> È consigliabile usare una versione dell'API successiva rispetto a `2016-04-30-preview` poiché sono state eseguite delle modifiche di rilievo tra `2016-04-30-preview` e `2017-03-30`.
>
>

### <a name="default-managed-disk-settings"></a>Impostazioni predefinite per i dischi gestiti

Per creare una macchina virtuale con dischi gestiti, non è più necessario creare la risorsa account di archiviazione ed è possibile aggiornare la risorsa macchina virtuale come indicato di seguito. Si noti in particolare che `apiVersion` corrisponde a `2017-03-30` e che `osDisk` e `dataDisks` non fanno più riferimento a uno specifico URI per il disco rigido virtuale. Quando si distribuisce senza specificare proprietà aggiuntive, il disco userà il tipo di archiviazione in base alle dimensioni della macchina virtuale. Se, ad esempio, si usa una dimensione della macchina virtuale compatibile con l'archiviazione Premium (dimensioni con "s" nel nome, ad esempio Standard_D2s_v3), il sistema userà l'archiviazione con ridondanza locale Premium. Usare l'impostazione SKU del disco per specificare un tipo di archiviazione. Se non si specifica un nome, assume il formato `<VMName>_OsDisk_1_<randomstring>` per il disco del sistema operativo e `<VMName>_disk<#>_<randomstring>` per ogni disco dati. Per impostazione predefinita, la crittografia dischi di Azure è disattivata; la memorizzazione nella cache è impostata su lettura/scrittura per il disco del sistema operativo e disattivata per i dischi dati. Nell'esempio seguente si può osservare che è ancora presente una dipendenza dall'account di archiviazione, ma è solo per l'archiviazione della diagnostica e non è necessaria per l'archiviazione su disco.

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2018-10-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="using-a-top-level-managed-disk-resource"></a>Uso di una risorsa disco gestito di primo livello

Come alternativa alla specifica della configurazione del disco nell'oggetto macchina virtuale, è possibile creare una risorsa disco di primo livello e collegarla nell'ambito della creazione della macchina virtuale. Ad esempio, è possibile creare una risorsa disco come segue per usarla come disco dati.

```json
{
    "type": "Microsoft.Compute/disks",
    "apiVersion": "2018-06-01",
    "name": "[concat(variables('vmName'),'-datadisk1')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "properties": {
        "creationData": {
            "createOption": "Empty"
        },
        "diskSizeGB": 1023
    }
}
```

All'interno dell'oggetto macchina virtuale, fare riferimento all'oggetto disco da collegare. Specificare l'ID risorsa del disco gestito creato nella proprietà `managedDisk` consente di collegare il disco non appena viene creata la macchina virtuale. Il valore di `apiVersion` per la risorsa della macchina virtuale viene impostato su `2017-03-30`. Viene aggiunta una dipendenza dalla risorsa disco per garantire che venga correttamente creata prima della creazione della macchina virtuale. 

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2018-10-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
        "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "lun": 0,
                    "name": "[concat(variables('vmName'),'-datadisk1')]",
                    "createOption": "attach",
                    "managedDisk": {
                        "id": "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
                    }
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="create-managed-availability-sets-with-vms-using-managed-disks"></a>Creare set di disponibilità gestiti con macchine virtuali che usano dischi gestiti

Per creare set di disponibilità gestiti con macchine virtuali che usano dischi gestiti, aggiungere l'oggetto `sku` alla risorsa set di disponibilità e impostare la proprietà `name` su `Aligned`. Questa proprietà garantisce che i dischi per ogni macchina virtuale siano sufficientemente isolati tra loro da evitare singoli punti di guasto. Si noti anche che `apiVersion` per la risorsa del set di disponibilità è impostato su `2018-10-01`.

```json
{
    "type": "Microsoft.Compute/availabilitySets",
    "apiVersion": "2018-10-01",
    "location": "[resourceGroup().location]",
    "name": "[variables('avSetName')]",
    "properties": {
        "PlatformUpdateDomainCount": 3,
        "PlatformFaultDomainCount": 2
    },
    "sku": {
        "name": "Aligned"
    }
}
```

### <a name="standard-ssd-disks"></a>Dischi SSD Standard

Di seguito sono elencati i parametri necessari nel modello di Resource Manager per creare dischi SSD Standard:

* *apiVersion* per Microsoft.Computer deve essere impostato su `2018-04-01` (o versioni successive)
* Specificare *managedDisk.storageAccountType* come `StandardSSD_LRS`

L'esempio seguente mostra la sezione *properties.storageProfile.osDisk* per una macchina virtuale che usa dischi SSD Standard:

```json
"osDisk": {
    "osType": "Windows",
    "name": "myOsDisk",
    "caching": "ReadWrite",
    "createOption": "FromImage",
    "managedDisk": {
        "storageAccountType": "StandardSSD_LRS"
    }
}
```

Per un esempio di modello completo di come creare un disco SSD standard con un modello, vedere [Create a VM from a Windows Image with Standard SSD Data Disks](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/) (Creare una macchina virtuale da un'immagine Windows con dischi dati SSD standard).

### <a name="additional-scenarios-and-customizations"></a>Altri scenari e personalizzazioni

Per informazioni complete sulle specifiche dell'API REST, vedere la [documentazione dell'API REST per la creazione di un disco gestito](/rest/api/manageddisks/disks/disks-create-or-update). La documentazione contiene scenari aggiuntivi e informazioni sui valori predefiniti e accettabili che è possibile inviare all'API tramite distribuzioni di modelli. 

## <a name="next-steps"></a>Passaggi successivi

* Per modelli completi che usano dischi gestiti, visitare i collegamenti seguenti al repository di modelli di avvio rapido di Azure.
    * [Macchina virtuale Windows con disco gestito](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
    * [Macchina virtuale Linux con disco gestito](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
    * [Elenco completo dei modelli con disco gestito](https://github.com/Azure/azure-quickstart-templates/blob/master/managed-disk-support-list.md)
* Per altre informazioni sui dischi gestiti, vedere [Panoramica di Azure Managed Disks](../articles/virtual-machines/windows/managed-disks-overview.md).
* La documentazione di riferimento del modello per le risorse macchina virtuale è disponibile alla pagina [Microsoft.Compute/virtualMachines template reference](/azure/templates/microsoft.compute/virtualmachines).
* La documentazione di riferimento del modello per le risorse disco è disponibile alla pagina [Microsoft.Compute/disks template reference](/azure/templates/microsoft.compute/disks).
* Per informazioni su come usare i dischi gestiti nei set di scalabilità di macchine virtuali di Microsoft Azure, vedere [Usare dischi dati con set di scalabilità](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks).
