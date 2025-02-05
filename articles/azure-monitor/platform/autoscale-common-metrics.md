---
title: Metriche comuni per la scalabilità automatica
description: Informazioni su quali metriche vengono comunemente usate per la scalabilità automatica di servizi cloud, macchine virtuali e app Web.
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 12/6/2016
ms.author: ancav
ms.subservice: autoscale
ms.openlocfilehash: 9da8e5fb88ff34e561b579b760973ecd23c884a3
ms.sourcegitcommit: 13d5eb9657adf1c69cc8df12486470e66361224e
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/31/2019
ms.locfileid: "66129740"
---
# <a name="azure-monitor-autoscaling-common-metrics"></a>Metriche comuni per la scalabilità automatica di Monitoraggio di Azure

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

La scalabilità automatica di Monitoraggio di Azure consente di aumentare o ridurre il numero delle istanze in esecuzione in base ai dati di telemetria (metriche). Questo documento descrive le metriche comuni che è possibile usare. Nel portale di Azure è possibile scegliere la metrica della risorsa in base alla quale eseguire il ridimensionamento. È tuttavia possibile scegliere metriche da risorse diverse.

La scalabilità automatica di Monitoraggio di Azure si applica solo a [set di scalabilità di macchine virtuali](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Servizi cloud](https://azure.microsoft.com/services/cloud-services/), [app Web del servizio app](https://azure.microsoft.com/services/app-service/web/) e [servizi di gestione API](https://docs.microsoft.com/azure/api-management/api-management-key-concepts). Altri servizi Azure usano metodi di ridimensionamento diversi.

## <a name="compute-metrics-for-resource-manager-based-vms"></a>Metriche di calcolo per le macchine virtuali basate su Resource Manager
Per impostazione predefinita, le macchine virtuali e i set di scalabilità di macchine virtuali basati su Resource Manager generano metriche di base (a livello di host). Quando si configura la raccolta dei dati di diagnostica per una VM di Azure e VMSS, inoltre, l'estensione Diagnostica di Azure genera anche i contatori delle prestazioni del sistema operativo guest (noti comunemente come "metriche del sistema operativo guest").  Tutte queste metriche vengono usate nelle regole di scalabilità automatica.

Per visualizzare le metriche disponibili per la risorsa del set di scalabilità di macchine virtuali è possibile usare l'interfaccia della riga di comando, PowerShell o l'API `Get MetricDefinitions`.

Se si usano set di scalabilità di macchine virtuali e una metrica specifica non è riportata nell'elenco, è probabile che sia *disabilitata* nell'estensione della diagnostica.

Se una metrica specifica non viene campionata o trasferita con la frequenza prevista, è possibile aggiornare la configurazione della diagnostica.

Se si verificano entrambe le condizioni precedenti, vedere [Usare PowerShell per abilitare la Diagnostica di Azure in una macchina virtuale che esegue Windows](../../virtual-machines/extensions/diagnostics-windows.md) per informazioni su come usare PowerShell per configurare e aggiornare l'estensione Diagnostica delle VM di Azure per abilitare la metrica. Questo articolo include anche un file di configurazione della diagnostica di esempio.

### <a name="host-metrics-for-resource-manager-based-windows-and-linux-vms"></a>Metriche host per le VM Windows e Linux basate su Resource Manager
Le metriche a livello di host seguenti vengono generate per impostazione predefinita per le VM di Azure e VMSS nelle istanze di Windows e Linux. Queste metriche descrivono la VM di Azure, ma vengono raccolte dall'host della VM di Azure anziché tramite l'agente installato nella VM guest. Queste metriche possono essere usate nelle regole di scalabilità automatica.

- [Metriche host per le VM Windows e Linux basate su Resource Manager](../../azure-monitor/platform/metrics-supported.md#microsoftcomputevirtualmachines)
- [Metriche host per i set di scalabilità di macchine virtuali Windows e Linux basati su Resource Manager](../../azure-monitor/platform/metrics-supported.md#microsoftcomputevirtualmachinescalesets)

### <a name="guest-os-metrics-resource-manager-based-windows-vms"></a>Metriche del sistema operativo guest per le VM Windows basate su Resource Manager
Quando si crea una nuova VM in Azure, la diagnostica viene abilitata con l'uso dell'estensione Diagnostica. L'estensione Diagnostica genera un set di metriche recuperate dall'interno della VM. Questo significa che è possibile gestire la scalabilità automatica con metriche non generate per impostazione predefinita.

Per generare un elenco delle metriche è possibile usare il comando seguente in PowerShell.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

È possibile creare un avviso per le metriche seguenti:

| Nome della metrica | Unità |
| --- | --- |
| \Processor(_Total)\% Processor Time |Percent |
| \Processor(_Total)\% Privileged Time |Percent |
| \Processor(_Total)\% User Time |Percent |
| \Processor Information(_Total)\Processor Frequency |Conteggio |
| \System\Processes |Conteggio |
| \Process(_Total)\Thread Count |Conteggio |
| \Process(_Total)\Handle Count |Conteggio |
| \Memory\% Committed Bytes In Use |Percent |
| \Memory\Available Bytes |Byte |
| \Memory\Committed Bytes |Byte |
| \Memory\Commit Limit |Byte |
| \Memory\Pool Paged Bytes |Byte |
| \Memory\Pool Nonpaged Bytes |Byte |
| \PhysicalDisk(_Total)\% Disk Time |Percent |
| \PhysicalDisk(_Total)\% Disk Read Time |Percent |
| \PhysicalDisk(_Total)\% Disk Write Time |Percent |
| \DiscoFisico(_Totale)\Trasferimenti disco/secondo |CountPerSecond |
| \PhysicalDisk(_Total)\Disk Reads/sec |CountPerSecond |
| \PhysicalDisk(_Total)\Disk Writes/sec |Conteggio al secondo |
| \PhysicalDisk(_Total)\Disk Bytes/sec |Byte al secondo |
| \PhysicalDisk(_Total)\Disk Read Bytes/sec |Byte al secondo |
| \PhysicalDisk(_Total)\Disk Write Bytes/sec |Byte al secondo |
| \PhysicalDisk(_Total)\Avg. Lunghezza coda disco |Conteggio |
| \PhysicalDisk(_Total)\Avg. Disk Read Queue Length |Conteggio |
| \PhysicalDisk(_Total)\Avg. Disk Write Queue Length |Conteggio |
| \LogicalDisk(_Total)\% Free Space |Percent |
| \LogicalDisk(_Total)\Free Megabytes |Conteggio |

### <a name="guest-os-metrics-linux-vms"></a>Metriche del sistema operativo guest per le VM Linux
Quando si crea una nuova VM in Azure, la diagnostica viene abilitata per impostazione predefinita con l'uso dell'estensione Diagnostica.

Per generare un elenco delle metriche è possibile usare il comando seguente in PowerShell.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

 È possibile creare un avviso per le metriche seguenti:

| Nome della metrica | Unità |
| --- | --- |
| \Memory\AvailableMemory |Byte |
| \Memory\PercentAvailableMemory |Percent |
| \Memory\UsedMemory |Byte |
| \Memory\PercentUsedMemory |Percent |
| \Memory\PercentUsedByCache |Percent |
| \Memory\PagesPerSec |CountPerSecond |
| \Memory\PagesReadPerSec |CountPerSecond |
| \Memory\PagesWrittenPerSec |CountPerSecond |
| \Memory\AvailableSwap |Byte |
| \Memory\PercentAvailableSwap |Percent |
| \Memory\UsedSwap |Byte |
| \Memory\PercentUsedSwap |Percent |
| \Processor\PercentIdleTime |Percent |
| \Processor\PercentUserTime |Percent |
| \Processor\PercentNiceTime |Percent |
| \Processor\PercentPrivilegedTime |Percent |
| \Processor\PercentInterruptTime |Percent |
| \Processor\PercentDPCTime |Percent |
| \Processor\PercentProcessorTime |Percent |
| \Processor\PercentIOWaitTime |Percent |
| \PhysicalDisk\BytesPerSecond |Byte al secondo |
| \PhysicalDisk\ReadBytesPerSecond |Byte al secondo |
| \PhysicalDisk\WriteBytesPerSecond |Byte al secondo |
| \PhysicalDisk\TransfersPerSecond |CountPerSecond |
| \PhysicalDisk\ReadsPerSecond |CountPerSecond |
| \PhysicalDisk\WritesPerSecond |CountPerSecond |
| \PhysicalDisk\AverageReadTime |Secondi |
| \PhysicalDisk\AverageWriteTime |Secondi |
| \PhysicalDisk\AverageTransferTime |Secondi |
| \PhysicalDisk\AverageDiskQueueLength |Conteggio |
| \NetworkInterface\BytesTransmitted |Byte |
| \NetworkInterface\BytesReceived |Byte |
| \NetworkInterface\PacketsTransmitted |Conteggio |
| \NetworkInterface\PacketsReceived |Conteggio |
| \NetworkInterface\BytesTotal |Byte |
| \NetworkInterface\TotalRxErrors |Conteggio |
| \NetworkInterface\TotalTxErrors |Conteggio |
| \NetworkInterface\TotalCollisions |Conteggio |

## <a name="commonly-used-web-server-farm-metrics"></a>Metriche Web (server farm) usate comunemente
È possibile eseguire la scalabilità automatica anche in base a metriche di server Web comuni, come la lunghezza della coda HTTP. Il nome della metrica è **HttpQueueLength**.  La sezione seguente riporta un elenco delle metriche di server farm (app Web) disponibili.

### <a name="web-apps-metrics"></a>Metriche di app Web
Per generare un elenco delle metriche di app Web è possibile usare il comando seguente in PowerShell.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

È possibile generare avvisi o ridimensionare in base a queste metriche.

| Nome della metrica | Unità |
| --- | --- |
| CpuPercentage |Percent |
| MemoryPercentage |Percent |
| DiskQueueLength |Conteggio |
| HttpQueueLength |Conteggio |
| BytesReceived |Byte |
| BytesSent |Byte |

## <a name="commonly-used-storage-metrics"></a>Metriche di archiviazione usate comunemente
È possibile eseguire il ridimensionamento in base alla lunghezza della coda di archiviazione, ovvero il numero di messaggi nella coda di archiviazione. La lunghezza della coda di archiviazione è una metrica speciale e la soglia corrisponde al numero di messaggi per ogni istanza. Ad esempio, se sono presenti due istanze e la soglia è impostata su 100, il ridimensionamento viene eseguito quando il numero totale di messaggi nella coda è 200. Può trattarsi di 100 messaggi per ogni istanza, 120 e 80 o qualsiasi altra combinazione corrispondente a un totale di 200 o più.

Questa impostazione può essere configurata nel pannello **Impostazioni** del portale di Azure. Per i set di scalabilità di macchine virtuali, è possibile aggiornare l'impostazione di scalabilità automatica nel modello di Resource Manager in modo da usare *metricName* come *ApproximateMessageCount* e passare l'ID della coda di archiviazione come *metricResourceUri*.

Per un account di archiviazione classico, ad esempio, la classe metricTrigger per la scalabilità automatica includerà:

```
"metricName": "ApproximateMessageCount",
 "metricNamespace": "",
 "metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ClassicStorage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
 ```

Per un account di archiviazione non classico, metricTrigger includerà:

```
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.Storage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
```

## <a name="commonly-used-service-bus-metrics"></a>Metriche del bus di servizio usate comunemente
È possibile eseguire il ridimensionamento in base alla lunghezza della coda del bus di servizio, ovvero il numero di messaggi nella coda del bus di servizio. La lunghezza della coda del bus di servizio è una metrica speciale e la soglia corrisponde al numero di messaggi per ogni istanza. Ad esempio, se sono presenti due istanze e la soglia è impostata su 100, il ridimensionamento viene eseguito quando il numero totale di messaggi nella coda è 200. Può trattarsi di 100 messaggi per ogni istanza, 120 e 80 o qualsiasi altra combinazione corrispondente a un totale di 200 o più.

Per i set di scalabilità di macchine virtuali, è possibile aggiornare l'impostazione di scalabilità automatica nel modello di Resource Manager in modo da usare *metricName* come *ApproximateMessageCount* e passare l'ID della coda di archiviazione come *metricResourceUri*.

```
"metricName": "MessageCount",
 "metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ServiceBus/namespaces/SB_NAMESPACE/queues/QUEUE_NAME"
```

> [!NOTE]
> Per il bus di servizio il concetto di gruppo di risorse non esiste, ma Azure Resource Manager consente di creare un gruppo di risorse predefinito per ogni area. Il gruppo di risorse è in genere nel formato "Default-ServiceBus-[area]". Ad esempio, "Default-ServiceBus-EastUS", "Default-ServiceBus-WestUS", "Default-ServiceBus-AustraliaEast" e così via.
>
>

