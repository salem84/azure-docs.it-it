---
title: 'Spostare circuiti dal modello di distribuzione classica a Resource Manager - ExpressRoute: PowerShell: Azure | Microsoft Docs'
description: Questa pagina illustra come spostare un circuito classico nel modello di distribuzione Resource Manager usando PowerShell.
services: expressroute
author: ganesr
ms.service: expressroute
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: cherylmc
ms.custom: seodec18
ms.openlocfilehash: 11a84d4ced3232102d262352b84abe1f813e2406
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60365182"
---
# <a name="move-expressroute-circuits-from-classic-to-resource-manager-deployment-model-using-powershell"></a>Spostare i circuiti ExpressRoute dal modello di distribuzione classica a quello Resource Manager usando PowerShell

Per usare un circuito ExpressRoute per il modello di distribuzione classica e per Resource Manager,è necessario spostare il circuito nel modello di distribuzione Resource Manager. Le sezioni seguenti descrivono come spostare il circuito tramite PowerShell.

## <a name="before-you-begin"></a>Prima di iniziare

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* Verificare che sono installati sia il modello di distribuzione classica e moduli Az Azure PowerShell in locale nel computer. Per altre informazioni, vedere [Come installare e configurare Azure PowerShell](/powershell/azure/overview).
* Prima di procedere con la configurazione, assicurarsi di avere verificato i [prerequisiti](expressroute-prerequisites.md), i [requisiti di routing](expressroute-routing.md) e i [flussi di lavoro](expressroute-workflows.md).
* Rivedere le informazioni disponibili in [Spostamento dei circuiti ExpressRoute dal modello di distribuzione classica al modello di distribuzione Resource Manager](expressroute-move.md). Assicurarsi di aver compreso pienamente i limiti e le limitazioni.
* Verificare che il circuito sia completamente operativo nel modello di distribuzione classica.
* Assicurarsi che sia disponibile un gruppo di risorse creato nel modello di distribuzione Resource Manager.

## <a name="move-an-expressroute-circuit"></a>Spostare un circuito ExpressRoute

### <a name="step-1-gather-circuit-details-from-the-classic-deployment-model"></a>Passaggio 1: Raccogliere informazioni dettagliate sul circuito dal modello di distribuzione classica

Accedere all'ambiente Azure classico e quindi ottenere la chiave servizio.

1. Accedere all'account Azure.

   ```powershell
   Add-AzureAccount
   ```

2. Selezionare la sottoscrizione di Azure appropriata.

   ```powershell
   Select-AzureSubscription "<Enter Subscription Name here>"
   ```

3. Importare i moduli di PowerShell per Azure ed ExpressRoute.

   ```powershell
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\Azure\Azure.psd1'
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\ExpressRoute\ExpressRoute.psd1'
   ```

4. Usare il cmdlet seguente per ottenere le chiavi servizio per tutti i circuiti ExpressRoute. Dopo avere recuperato le chiavi, copiare la **chiave servizio** del circuito che si desidera spostare nel modello di distribuzione Resource Manager.

   ```powershell
   Get-AzureDedicatedCircuit
   ```

### <a name="step-2-sign-in-and-create-a-resource-group"></a>Passaggio 2: Accedere e creare un gruppo di risorse

Accedere all'ambiente Resource Manager e creare un nuovo gruppo di risorse.

1. Accedere all'ambiente Azure Resource Manager.

   ```powershell
   Connect-AzAccount
   ```

2. Selezionare la sottoscrizione di Azure appropriata.

   ```powershell
   Get-AzSubscription -SubscriptionName "<Enter Subscription Name here>" | Select-AzSubscription
   ```

3. Modificare il frammento seguente per creare un nuovo gruppo di risorse se non si dispone già di un gruppo di risorse.

   ```powershell
   New-AzResourceGroup -Name "DemoRG" -Location "West US"
   ```

### <a name="step-3-move-the-expressroute-circuit-to-the-resource-manager-deployment-model"></a>Passaggio 3: Spostare il circuito ExpressRoute nel modello di distribuzione Resource Manager

È ora possibile spostare il circuito ExpressRoute dal modello di distribuzione classica al modello di distribuzione Resource Manager. Prima di continuare, verificare le informazioni disponibili in [Spostamento di un circuito ExpressRoute dal modello di distribuzione classica al modello di distribuzione Resource Manager](expressroute-move.md).

Per spostare il circuito, modificare ed eseguire il frammento seguente:

```powershell
Move-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "DemoRG" -Location "West US" -ServiceKey "<Service-key>"
```

Nella modalità classica di un circuito ExpressRoute non esiste il concetto di vincolo a un'area. In Resource Manager, tuttavia, ogni risorsa deve essere mappata a un'area di Azure. L'area specificato nel cmdlet Move-AzExpressRouteCircuit tecnicamente può essere qualsiasi area. A fini organizzativi, è possibile scegliere un'area che rappresenta in modo affidabile il percorso di peering.

> [!NOTE]
> Al termine dello spostamento, il nuovo nome elencato nel cmdlet precedente verrà usato per fare riferimento alla risorsa. Il circuito verrà essenzialmente rinominato.
> 

## <a name="modify-circuit-access"></a>Modificare l'accesso al circuito

### <a name="to-enable-expressroute-circuit-access-for-both-deployment-models"></a>Per abilitare il circuito ExpressRoute accedere ad entrambi i modelli di distribuzione

Dopo aver spostato il circuito ExpressRoute classico nel modello di distribuzione Resource Manager, è possibile abilitare l'accesso a entrambi i modelli di distribuzione. Eseguire i cmdlet seguenti per abilitare l'accesso a entrambi i modelli di distribuzione:

1. Ottenere i dettagli del circuito.

   ```powershell
   $ckt = Get-AzExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"
   ```

2. Impostare "Allow Classic Operations" (Consenti operazioni classiche) su VERO.

   ```powershell
   $ckt.AllowClassicOperations = $true
   ```

3. Aggiornamento del circuito. Al termine di questa operazione, sarà possibile visualizzare il circuito nel modello di distribuzione classica.

   ```powershell
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

4. Eseguire il cmdlet seguente per ottenere i dettagli del circuito ExpressRoute. Si dovrebbe essere in grado di vedere la chiave servizio elencata.

   ```powershell
   get-azurededicatedcircuit
   ```

5. È ora possibile gestire i collegamenti al circuito ExpressRoute usando i comandi del modello di distribuzione classica per le reti virtuali classiche e i comandi di Resource Manager per le reti virtuali di Resource Manager. Gli articoli seguenti descrivono come gestire i collegamenti al circuito ExpressRoute:

    * [Collegare la rete virtuale al circuito ExpressRoute nel modello di distribuzione Resource Manager](expressroute-howto-linkvnet-arm.md)
    * [Collegare la rete virtuale al circuito ExpressRoute nel modello di distribuzione classica](expressroute-howto-linkvnet-classic.md)

### <a name="to-disable-expressroute-circuit-access-to-the-classic-deployment-model"></a>Per disabilitare l'accesso del circuito ExpressRoute al modello di distribuzione classica

Eseguire i cmdlet seguenti per disabilitare l'accesso al modello di distribuzione classica.

1. Ottenere i dettagli del circuito ExpressRoute.

   ```powershell
   $ckt = Get-AzExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"
   ```

2. Impostare "Allow Classic Operations" (Consenti operazioni classiche) su FALSO.

   ```powershell
   $ckt.AllowClassicOperations = $false
   ```

3. Aggiornamento del circuito. Al termine di questa operazione, non sarà possibile visualizzare il circuito nel modello di distribuzione classica.

   ```powershell
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

## <a name="next-steps"></a>Passaggi successivi

* [Creare e modificare il routing per un circuito ExpressRoute](expressroute-howto-routing-arm.md)
* [Collegare la rete virtuale al circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
