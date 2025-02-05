---
title: Creare un'offerta di modulo Azure IoT Edge | Azure Marketplace
description: Come pubblicare un nuovo modulo IoT Edge per il Marketplace.
services: Azure, Marketplace, Cloud Partner Portal,
author: dan-wesley
ms.service: marketplace
ms.topic: conceptual
ms.date: 10/18/2018
ms.author: pabutler
ms.openlocfilehash: 98eb537a5c064cd22b995ae214c8d61db7a8b6b5
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "64942489"
---
# <a name="create-a-new-iot-edge-module-offer-with-the-cloud-partner-portal"></a>Creare una nuova offerta di modulo IoT Edge nel portale Cloud Partner

Questo articolo descrive come creare e pubblicare una voce di offerta di modulo IoT Edge per Azure Marketplace. Ogni offerta viene visualizzata come entità a sé stante in Azure Marketplace ed è associata a uno o più SKU.  Un'offerta di modulo IoT Edge è composta dai gruppi di asset e servizi di supporto seguenti:

|  **Gruppo di risorse**   |  **Descrizione**  |
|  ---------------   |  ---------------  |
|    SKU            |  La più piccola unità distribuibile di un'offerta. Una singola offerta (classe di prodotti) può essere associata a più SKU. È possibile usare gli SKU per distinguere tra le funzionalità supportate e i modelli di fatturazione. |
|  Marketplace       | Contiene specifiche e risorse di marketing, legali e di gestione dei clienti potenziali.  <ul><li> Le risorse di marketing includono il nome dell'offerta, la descrizione e i loghi</li> <li> Gli asset legali includono l'informativa sulla privacy, le condizioni per l'utilizzo e altra documentazione legale</li>  <li> I criteri di gestione dei clienti potenziali consentono di specificare come gestire i clienti potenziali dal portale dell'utente finale Azure Marketplace.</li> </ul> |
| Supporto            | Contiene informazioni sui criteri e sui contatti di supporto |


## <a name="new-offer-form"></a>Modulo Nuova offerta 

Accedere al [portale Cloud Partner](https://cloudpartner.azure.com/) e quindi selezionare **+ Nuova offerta** nella barra dei menu a sinistra. Nel menu Nuova offerta selezionare **Moduli IoT Edge** per visualizzare il modulo **Nuova offerta** e avviare il processo di definizione degli asset per la nuova offerta di modulo IoT Edge. 

![Selezione dell'interfaccia utente dell'offerta di nuovo modulo IoT Edge](./media/new-iot-edge-module-offer.png)

## <a name="next-steps"></a>Passaggi successivi

La pagina **Nuova offerta** per il tipo di offerta di modulo IoT Edge include un set di schede e di campi modulo che è possibile usare per creare una nuova offerta. Ognuno degli articoli seguenti illustra come usare la scheda per definire i gruppi di asset e i servizi di supporto per la nuova offerta di modulo IoT Edge.

- [Scheda Impostazioni dell'offerta](./cpp-offer-settings-tab.md)
- [Scheda SKU](./cpp-skus-tab.md)
- [Scheda Marketplace](./cpp-marketplace-tab.md)
- [Scheda Supporto](./cpp-support-tab.md)
