---
title: File di inclusione
description: File di inclusione
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 09/23/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: ee8a711a867f8abdc831b0d1d9d0b504b1104955
ms.sourcegitcommit: 0486aba120c284157dfebbdaf6e23e038c8a5a15
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/26/2019
ms.locfileid: "71310117"
---
# <a name="creating-an-incremental-snapshot-preview-for-managed-disks"></a>Creazione di uno snapshot incrementale (anteprima) per Managed Disks

Gli snapshot incrementali (anteprima) sono backup temporizzati per i dischi gestiti che, quando vengono eseguiti, sono costituiti solo da tutte le modifiche apportate dopo l'ultimo snapshot. Quando si tenta di scaricare o utilizzare in altro modo uno snapshot incrementale, viene utilizzato il VHD completo. Questa nuova funzionalità per gli snapshot dei dischi gestiti può consentire loro di essere più conveniente, perché non è più necessario archiviare l'intero disco con ogni singolo snapshot, a meno che non si scelga di. Analogamente agli snapshot regolari, è possibile usare gli snapshot incrementali per creare un disco gestito completo o, per creare uno snapshot regolare.

Esistono alcune differenze tra uno snapshot incrementale e uno snapshot regolare. Gli snapshot incrementali utilizzeranno sempre l'archiviazione HDD standard, indipendentemente dal tipo di archiviazione del disco, mentre gli snapshot regolari possono usare unità SSD Premium. Se si usano snapshot regolari nell'archiviazione Premium per la scalabilità verticale delle distribuzioni di macchine virtuali, è consigliabile usare immagini personalizzate nell'archiviazione standard della [raccolta immagini condivise](../articles/virtual-machines/linux/shared-image-galleries.md). Consente di ottenere una scalabilità più massiccia con costi ridotti. Inoltre, gli snapshot incrementali offrono potenzialmente una migliore affidabilità con l' [archiviazione con ridondanza della zona](../articles/storage/common/storage-redundancy-zrs.md) (ZRS). Se ZRS è disponibile nell'area selezionata, verrà usato automaticamente ZRS per uno snapshot incrementale. Se ZRS non è disponibile nell'area, lo snapshot utilizzerà per impostazione predefinita l' [archiviazione con ridondanza locale](../articles/storage/common/storage-redundancy-lrs.md) (con ridondanza locale). È possibile eseguire l'override di questo comportamento e selezionarne uno manualmente, ma non è consigliabile.

Gli snapshot incrementali offrono inoltre una funzionalità differenziale, disponibile in modo univoco per i dischi gestiti. Consentono di ottenere le modifiche tra due snapshot incrementali degli stessi dischi gestiti, fino al livello di blocco. È possibile usare questa funzionalità per ridurre il footprint dei dati durante la copia degli snapshot tra le aree.

Se non è ancora stata effettuata l'iscrizione per l'anteprima e si vuole iniziare a usare gli snapshot incrementali, inviare un AzureDisks@microsoft.com messaggio di posta elettronica all'indirizzo per ottenere l'accesso all'anteprima pubblica.

## <a name="restrictions"></a>Restrizioni

- Gli snapshot incrementali sono attualmente disponibili solo negli Stati Uniti centro-occidentali.
- Attualmente non è possibile creare snapshot incrementali dopo aver modificato le dimensioni di un disco.
- Non è attualmente possibile spostare gli snapshot incrementali tra le sottoscrizioni.
- Attualmente, è possibile generare solo URI SAS di un massimo di cinque snapshot di una particolare famiglia di snapshot in un determinato momento.
- Non è possibile creare uno snapshot incrementale per un disco specifico al di fuori della sottoscrizione del disco.
- È possibile creare fino a sette snapshot incrementali per ogni disco ogni cinque minuti.
- Per un singolo disco è possibile creare un totale di 200 snapshot incrementali.

## <a name="powershell"></a>PowerShell

È possibile utilizzare Azure PowerShell per creare uno snapshot incrementale. Sarà necessaria la versione più recente di Azure PowerShell, il seguente comando lo installerà o aggiornerà l'installazione esistente alla versione più recente:

```PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser
```

Una volta installato, accedere alla sessione di PowerShell con `az login`.

Per creare uno snapshot incrementale con Azure PowerShell, impostare la configurazione con [New-AzSnapShotConfig](https://docs.microsoft.com/en-us/powershell/module/az.compute/new-azsnapshotconfig?view=azps-2.7.0) con il `-Incremental` parametro e quindi passarlo come variabile a [New-AzSnapshot](https://docs.microsoft.com/en-us/powershell/module/az.compute/new-azsnapshot?view=azps-2.7.0) tramite il `-Snapshot` parametro.

Sostituire `<yourDiskNameHere>`, `<yourResourceGroupNameHere>` e`<yourDesiredSnapShotNameHere>` con i valori, è possibile usare lo script seguente per creare uno snapshot incrementale:

```PowerShell
# Get the disk that you need to backup by creating an incremental snapshot
$yourDisk = Get-AzDisk -DiskName <yourDiskNameHere> -ResourceGroupName <yourResourceGroupNameHere>

# Create an incremental snapshot by setting the SourceUri property with the value of the Id property of the disk
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental 
New-AzSnapshot -ResourceGroupName <yourResourceGroupNameHere> -SnapshotName <yourDesiredSnapshotNameHere> -Snapshot $snapshotConfig 
```

È possibile identificare gli snapshot incrementali dallo stesso disco con `SourceResourceId` e le `SourceUniqueId` proprietà degli snapshot. `SourceResourceId`ID della risorsa Azure Resource Manager del disco padre. `SourceUniqueId`valore ereditato dalla `UniqueId` proprietà del disco. Se si elimina un disco e quindi si crea un nuovo disco con lo stesso nome, il valore della `UniqueId` proprietà cambia.

È possibile utilizzare `SourceResourceId` e `SourceUniqueId` per creare un elenco di tutti gli snapshot associati a un disco specifico. Sostituire `<yourResourceGroupNameHere>` con il valore, quindi è possibile usare l'esempio seguente per elencare gli snapshot incrementali esistenti:

```PowerShell
$snapshots = Get-AzSnapshot -ResourceGroupName <yourResourceGroupNameHere>

$incrementalSnapshots = New-Object System.Collections.ArrayList
foreach ($snapshot in $snapshots)
{
    
    if($snapshot.Incremental -and $snapshot.CreationData.SourceResourceId -eq $yourDisk.Id -and $snapshot.CreationData.SourceUniqueId -eq $yourDisk.UniqueId){

        $incrementalSnapshots.Add($snapshot)
    }
}

$incrementalSnapshots
```

## <a name="cli"></a>Interfaccia della riga di comando

È possibile creare uno snapshot incrementale con l'interfaccia della riga di comando di Azure. sarà necessaria la versione più recente dell'interfaccia della riga di comando di Azure. Il comando seguente consente di installare o aggiornare l'installazione esistente alla versione più recente:

```PowerShell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'
```

Per creare uno snapshot incrementale, usare [AZ snapshot create](https://docs.microsoft.com/cli/azure/snapshot?view=azure-cli-latest#az-snapshot-create) con il `--incremental` parametro.

Nell'esempio seguente viene creato uno snapshot incrementale, `<yourDesiredSnapShotNameHere>`vengono `<yourResourceGroupNameHere>`sostituiti`<exampleDiskName>`, `<exampleLocation>` , e con valori personalizzati, quindi viene eseguito l'esempio:

```bash
sourceResourceId=$(az disk show -g <yourResourceGroupNameHere> -n <exampleDiskName> --query '[id]' -o tsv)

az snapshot create -g <yourResourceGroupNameHere> \
-n <yourDesiredSnapShotNameHere> \
-l <exampleLocation> \
--source "$sourceResourceId" \
--incremental
```

È possibile identificare gli snapshot incrementali dallo stesso disco con `SourceResourceId` e le `SourceUniqueId` proprietà degli snapshot. `SourceResourceId`ID della risorsa Azure Resource Manager del disco padre. `SourceUniqueId`valore ereditato dalla `UniqueId` proprietà del disco. Se si elimina un disco e quindi si crea un nuovo disco con lo stesso nome, il valore della `UniqueId` proprietà cambia.

È possibile utilizzare `SourceResourceId` e `SourceUniqueId` per creare un elenco di tutti gli snapshot associati a un disco specifico. Nell'esempio seguente vengono elencati tutti gli snapshot incrementali associati a un disco specifico, ma è necessario un certo programma di installazione.

Questo esempio USA JQ per eseguire query sui dati. Per eseguire l'esempio, è necessario [installare JQ](https://stedolan.github.io/jq/download/).

Sostituire `<yourResourceGroupNameHere>` e`<exampleDiskName>` con i valori, quindi è possibile usare l'esempio seguente per elencare gli snapshot incrementali esistenti, purché sia stato installato anche JQ:

```bash
sourceUniqueId=$(az disk show -g <yourResourceGroupNameHere> -n <exampleDiskName> --query '[uniqueId]' -o tsv)

 
sourceResourceId=$(az disk show -g <yourResourceGroupNameHere> -n <exampleDiskName> --query '[id]' -o tsv)

az snapshot list -g <yourResourceGroupNameHere> -o json \
| jq -cr --arg SUID "$sourceUniqueId" --arg SRID "$sourceResourceId" '.[] | select(.incremental==true and .creationData.sourceUniqueId==$SUID and .creationData.sourceResourceId==$SRID)'
```

## <a name="resource-manager-template"></a>Modello di Resource Manager

È inoltre possibile utilizzare Azure Resource Manager modelli per creare uno snapshot incrementale. È necessario assicurarsi che apiVersion sia impostato su **2019-03-01** e che anche la proprietà incrementale sia impostata su true. Il frammento di codice seguente è un esempio di creazione di uno snapshot incrementale con i modelli di Gestione risorse:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "contosodisk1"
    },
  "diskResourceId": {
    "defaultValue": "<your_managed_disk_resource_ID>",
    "type": "String"
  }
  }, 
  "resources": [
  {
    "type": "Microsoft.Compute/snapshots",
    "name": "[concat( parameters('diskName'),'_snapshot1')]",
    "location": "[resourceGroup().location]",
    "apiVersion": "2019-03-01",
    "properties": {
      "creationData": {
        "createOption": "Copy",
        "sourceResourceId": "[parameters('diskResourceId')]"
      },
      "incremental": true
    }
  }
  ]
}
```

## <a name="next-steps"></a>Passaggi successivi

Se non è ancora stata effettuata l'iscrizione per l'anteprima e si vuole iniziare a usare gli snapshot incrementali, inviare un AzureDisks@microsoft.com messaggio di posta elettronica all'indirizzo per ottenere l'accesso all'anteprima pubblica.
