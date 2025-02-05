---
title: 'Creare e modificare un circuito ExpressRoute - portale: Azure | Microsoft Docs'
description: Creazione, provisioning, verifica, aggiornamento, eliminazione e deprovisioning di un circuito ExpressRoute.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: article
ms.date: 10/20/2018
ms.author: cherylmc
ms.reviewer: ganesr
ms.custom: seodec18
ms.openlocfilehash: 25821f60f47b1279e70cba2574901cd5df3d327f
ms.sourcegitcommit: fa45c2bcd1b32bc8dd54a5dc8bc206d2fe23d5fb
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/12/2019
ms.locfileid: "67846593"
---
# <a name="create-and-modify-an-expressroute-circuit"></a>Creare e modificare un circuito ExpressRoute

> [!div class="op_single_selector"]
> * [Portale di Azure](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Interfaccia della riga di comando di Azure](howto-circuit-cli.md)
> * [Modello di Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Video - Portale di Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (classic)](expressroute-howto-circuit-classic.md) (PowerShell (classico))
>

Questo articolo include informazioni utili per creare un circuito ExpressRoute di Azure usando il portale di Azure e il modello di distribuzione di Azure Resource Manager. È anche possibile controllare lo stato, aggiornare, eliminare o effettuare il deprovisioning di un circuito.

## <a name="before-you-begin"></a>Prima di iniziare

* Prima di iniziare la configurazione, verificare i [prerequisiti](expressroute-prerequisites.md) e i [flussi di lavoro](expressroute-workflows.md).
* Verificare di avere accesso al [portale di Azure](https://portal.azure.com).
* Verificare di avere le autorizzazioni necessarie per creare nuove risorse di rete. Se non si hanno le autorizzazioni appropriate, contattare l'amministratore dell'account.
* È possibile [visualizzare un video](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit) prima di iniziare, per ottenere una comprensione migliore della procedura.

## <a name="create"></a>Creare un circuito ExpressRoute ed eseguirne il provisioning

### <a name="1-sign-in-to-the-azure-portal"></a>1. Accedere al portale di Azure

In un browser passare al [portale di Azure](https://portal.azure.com) e accedere con l'account Azure.

### <a name="2-create-a-new-expressroute-circuit"></a>2. Creare un nuovo circuito ExpressRoute

> [!IMPORTANT]
> Il circuito ExpressRoute viene addebitato dal momento in cui viene emessa una chiave di servizio. Verificare che l'operazione venga eseguita quando il provider di connettività è pronto a effettuare il provisioning del circuito.

1. È possibile creare un circuito ExpressRoute selezionando l'opzione che consente di creare una nuova risorsa. Fare clic su **Crea una risorsa** > **Rete** > **ExpressRoute**, come illustrato nell'immagine seguente:

   ![Creare un circuito ExpressRoute](./media/expressroute-howto-circuit-portal-resource-manager/createcircuit1.png)
2. Dopo aver fatto clic su **ExpressRoute**, viene visualizzata la pagina **Crea un circuito ExpressRoute**. Quando si inseriscono i valori in questa pagina, assicurarsi di specificare il livello di SKU (Standard o Premium) e il modello di fatturazione della misurazione dei dati (Senza limiti o A consumo) corretti.

   ![Configurazione del livello SKU e misurazione dei dati](./media/expressroute-howto-circuit-portal-resource-manager/createcircuit.png)

   * **livello** determina se è abilitato un componente aggiuntivo ExpressRoute Standard o ExpressRoute Premium. È possibile specificare **Standard** per ottenere lo SKU Standard o **Premium** per il componente aggiuntivo Premium.
   * **misurazione dei dati** determina il tipo di fatturazione. È possibile specificare **A consumo** per un piano dati a consumo e **Senza limiti** per un piano dati illimitato. Si noti che è possibile modificare il tipo di fatturazione da **A consumo** in **Illimitato**.

     > [!IMPORTANT]
     > Non è possibile modificare il tipo da **illimitato** a a **consumo**.

   * **Località peer** è la posizione fisica di peering con Microsoft.

     > [!IMPORTANT]
     > La località peer indica la [posizione fisica](expressroute-locations.md) di peering con Microsoft. Questo percorso **non** è collegato alla proprietà "Location", ovvero all'area geografica in cui si trova il provider di risorse di rete di Azure. Dal momento che non sono collegati, è consigliabile scegliere un provider di risorse di rete geograficamente vicino alla posizione di peering del circuito.

### <a name="3-view-the-circuits-and-properties"></a>3. Visualizzare circuiti e proprietà

**Visualizzare tutti i circuiti**

È possibile visualizzare tutti i circuiti creati selezionando **Tutte le risorse** dal menu a sinistra.

![Visualizzazione dei circuiti](./media/expressroute-howto-circuit-portal-resource-manager/listresource.png)

**Visualizzare le proprietà**

Selezionare il circuito desiderato per visualizzare le relative proprietà. Nella pagina **Panoramica** per il circuito la chiave del servizio viene visualizzata nel campo della chiave del servizio. È necessario copiare la chiave del servizio per il circuito e passarla al provider di servizi per completare il processo di provisioning. La chiave del servizio del circuito è specifica del circuito.

![Visualizza proprietà](./media/expressroute-howto-circuit-portal-resource-manager/servicekey1.png)

### <a name="4-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>4. Inviare la chiave di servizio al provider di connettività per il provisioning

In questa pagina, **Stato provider** offre informazioni sullo stato attuale di provisioning sul lato provider del servizio. **Stato circuito** indica lo stato sul lato Microsoft. Per altre informazioni sullo stato di provisioning dei circuiti, vedere l'articolo relativo ai [flussi di lavoro](expressroute-workflows.md#expressroute-circuit-provisioning-states) .

Quando si crea un nuovo circuito ExpressRoute, il circuito ha lo stato seguente:

Stato provider: Senza provisioning<BR>
Stato circuito: Enabled

![Avvio del processo di provisioning](./media/expressroute-howto-circuit-portal-resource-manager/status.png)

Il circuito passa allo stato seguente quando è in corso l'abilitazione da parte del provider di connettività:

Stato provider: Provisioning<BR>
Stato circuito: Enabled

Per poterlo usare, un circuito ExpressRoute deve avere lo stato seguente:

Stato provider: Sottoposto a provisioning<BR>
Stato circuito: Enabled

### <a name="5-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>5. Controllare periodicamente lo stato e la condizione della chiave del circuito

Selezionare il circuito desiderato per visualizzare le relative proprietà. Prima di continuare, controllare che **Stato provider** sia passato a **Provisioning eseguito**.

![Stato del circuito e del provider](./media/expressroute-howto-circuit-portal-resource-manager/provisioned.png)

### <a name="6-create-your-routing-configuration"></a>6. Creare la configurazione di routing

Per istruzioni dettagliate, vedere l'articolo relativo alla [configurazione del routing per un circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md) per creare e modificare i peering del circuito.

> [!IMPORTANT]
> Queste istruzioni si applicano solo ai circuiti creati con provider di servizi che offrono servizi di connettività di livello 2. Se si usa un provider di servizi che offre servizi gestiti di livello 3 (di solito una VPN IP, come MPLS), il provider di connettività configura e gestisce il routing per conto dell'utente.

### <a name="7-link-a-virtual-network-to-an-expressroute-circuit"></a>7. Collegare una rete virtuale a un circuito ExpressRoute

Collegare quindi una rete virtuale al circuito ExpressRoute. Fare riferimento all'articolo [Collegare una rete virtuale a un circuito ExpressRoute](expressroute-howto-linkvnet-arm.md) quando si usa il modello di distribuzione di Resource Manager.

## <a name="status"></a>Ottenere lo stato di un circuito ExpressRoute

È possibile visualizzare lo stato di un circuito selezionando il circuito e visualizzando la pagina Panoramica.

## <a name="modify"></a>Modifica di un circuito ExpressRoute

È possibile modificare determinate proprietà di un circuito ExpressRoute senza conseguenze per la connettività. È possibile modificare la larghezza di banda, lo SKU e il modello di fatturazione e consentire operazioni classiche nella pagina **Configurazione**. Per informazioni sui limiti e le limitazioni, vedere [Domande frequenti su ExpressRoute](expressroute-faqs.md).

È possibile eseguire le attività seguenti evitando tempi di inattività:

* Abilitare o disabilitare un componente aggiuntivo ExpressRoute Premium per il circuito ExpressRoute.
* Aumentare la larghezza di banda del circuito ExpressRoute, a condizione che nella porta sia disponibile capacità.

  > [!IMPORTANT]
  > Il downgrade della larghezza di banda di un circuito non è supportato.

* Modificare il piano di misurazione da *Dati a consumo* a *Dati senza limiti*.

  > [!IMPORTANT]
  > La modifica del piano di misurazione da Dati senza limiti a Dati a consumo non è supportata.

* È possibile abilitare e disabilitare l'opzione *Consenti operazioni classiche*.
  > [!IMPORTANT]
  > Se la capacità sulla porta esistente non è sufficiente, potrebbe essere necessario ricreare il circuito ExpressRoute. Il circuito non può essere aggiornato se in tale posizione non è disponibile capacità aggiuntiva.
  >
  > Anche se è possibile aggiornare facilmente la banda larga, non è possibile ridurre la larghezza di banda di un circuito ExpressRoute senza interruzioni. Il downgrade della larghezza di banda richiede il deprovisioning del circuito ExpressRoute e quindi il provisioning di un nuovo circuito ExpressRoute.
  >
  > L'operazione di disabilitazione del componente aggiuntivo Premium può avere esito negativo se si usano risorse superiori a quelle consentite per il circuito standard.

Per modificare un circuito ExpressRoute, fare clic su **Configurazione**.

![Modificare il circuito](./media/expressroute-howto-circuit-portal-resource-manager/modifycircuit.png)

## <a name="delete"></a>Deprovisioning ed eliminazione di un circuito ExpressRoute

È possibile eliminare il circuito ExpressRoute selezionando l'icona di **eliminazione** . Tenere presente quanto segue:

* È necessario scollegare tutte le reti virtuali dal circuito ExpressRoute. Se l'operazione non riesce, controllare se sono presenti reti virtuali collegate al circuito.
* Se lo stato di provisioning del provider del servizio del circuito ExpressRoute è **Provisioning in corso** o **Provisioning eseguito**, è necessario collaborare con il provider di servizi per eseguire il deprovisioning del circuito sul lato del provider. Le risorse continuano a essere riservate e la fatturazione continuerà a essere applicata finché il provider di servizi non avrà completato il deprovisioning del circuito e inviato una notifica a Microsoft.
* Se il provider di servizi ha eseguito il deprovisioning del circuito, ovvero lo stato di provisioning del provider di servizi è impostato su **Senza provisioning**, è possibile eliminare il circuito. Viene così interrotta la fatturazione per il circuito.

## <a name="next-steps"></a>Passaggi successivi

Dopo aver creato il circuito, continuare con i passaggi seguenti:

* [Creare e modificare il routing per un circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Collegare la rete virtuale al circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
