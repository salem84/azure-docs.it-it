---
title: Pubblica offerta di immagine i contenitori di Azure | Azure Marketplace
description: Come pubblicare l'offerta di un contenitore di Azure.
services: Azure, Marketplace, Cloud Partner Portal,
author: dan-wesley
ms.service: marketplace
ms.topic: conceptual
ms.date: 11/01/2018
ms.author: pabutler
ms.openlocfilehash: 9433673e464beb2df74eb4f49851e960d2e7f99c
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "64942657"
---
# <a name="publish-container-offer"></a>Pubblicare l'offerta di un contenitore

 Dopo aver creato una nuova offerta tramite la pagina **Nuova offerta**, è possibile pubblicarla. Selezionare **Pubblica** per avviare il processo di pubblicazione.

Il diagramma seguente illustra i passaggi principali del processo di pubblicazione che rende un'offerta disponibile in Azure Marketplace.

![Passaggi per la pubblicazione dell'offerta di un contenitore](./media/offer-publishing-steps.png)

## <a name="detailed-description-of-publishing-steps"></a>Descrizione dettagliata dei passaggi per la pubblicazione

La tabella seguente descrive i passaggi di pubblicazione. Viene inoltre specificato un tempo stimato per il completamento di ogni passaggio.


|  **Passaggio per la pubblicazione**           | **Ora**    | **Descrizione**                                                            |
|  -------------------           | --------    | ---------------                                                            |
| Convalida dei prerequisiti         | 15 min   | Vengono convalidate le informazioni e le impostazioni dell'offerta.                        |
| Certificazione                  | 1 settimana | L'offerta viene analizzata dal team delle certificazioni di Azure. Viene eseguita l'analisi dell'offerta per individuare eventuali virus, malware, conformità alle normative sulla sicurezza e problemi di sicurezza. L'offerta viene controllata per verificare che soddisfi tutti i criteri di idoneità. Per altre informazioni, vedere [Prerequisiti](./cpp-prerequisites.md) e [Prepare your container technical assets](./cpp-create-technical-assets.md) (Preparare gli asset tecnici del contenitore). Se viene rilevato un problema, viene fornito un feedback. |
| Packaging | 1 ora  | Le risorse tecniche dell'offerta vengono incluse in un pacchetto per uso dei clienti e i sistemi di registrazione di clienti potenziali vengono configurati e impostati. |
|  Approvazione del server di pubblicazione             |  -        | Revisione e conferma finali dell'editore prima che l'offerta diventi disponibile in Azure Marketplace. È possibile distribuire l'offerta nelle sottoscrizioni selezionate (nei passaggi relativi alle informazioni sull'offerta) per verificare che soddisfi tutti i requisiti.  Selezionare **Go Live** in modo che l'offerta passi alla fase successiva. |
| Packaging                 | 1 ora | L'offerta completata viene replicata nelle aree e nei sistemi di produzione di Marketplace. | 
| Live                           | 4 giorni |L'offerta viene rilasciata, replicata nelle aree necessarie e resa disponibile al pubblico. |

Occorrono fino a 10 giorni lavorativi perché il processo di pubblicazione venga completato e l'offerta venga rilasciata. Al termine del processo di pubblicazione, l'offerta del contenitore sarà disponibile nell'elenco di [Microsoft Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules).

## <a name="next-steps"></a>Passaggi successivi

[Aggiornare l'offerta di un contenitore esistente in Azure Marketplace](./cpp-update-existing-offer.md)
