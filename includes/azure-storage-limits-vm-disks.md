---
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/18/2019
ms.author: rogarana
ms.openlocfilehash: 8b25d2395811a2197aff6d653c5038a4380021e9
ms.sourcegitcommit: cd70273f0845cd39b435bd5978ca0df4ac4d7b2c
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/18/2019
ms.locfileid: "68669703"
---
È possibile aggiungere un numero di dischi dati a una macchina virtuale di Azure. In base agli obiettivi di scalabilità e prestazioni per i dischi dati di una macchina virtuale, è possibile determinare il numero e il tipo di disco necessari per soddisfare i requisiti di prestazioni e capacità.

> [!IMPORTANT]
> Per prestazioni ottimali, limitare il numero di dischi a elevato uso collegati alla macchina virtuale per evitare una possibile limitazione. Se tutti i dischi collegati non vengono utilizzati allo stesso tempo, la macchina virtuale può supportare un numero maggiore di dischi.

**Per Azure Managed disks:**

La tabella seguente illustra i limiti predefiniti e massimi del numero di risorse per area per sottoscrizione. Non esiste alcun limite per il numero di Managed Disks, snapshot e immagini per gruppo di risorse.  

> | Risorsa | Limite predefinito  | Limite massimo |
> | --- | --- | --- |
> | Dischi gestiti Standard | 50,000 | 50,000 |
> | Managed Disks SSD Standard | 50,000 | 50,000 |
> | Managed Disks Premium | 50,000 | 50,000 |
> | Snapshot Standard_LRS | 50,000 | 50,000 |
> | Snapshot Standard_ZRS | 50,000 | 50,000 |
> | Immagine gestita | 50,000 | 50,000 |

* **Per gli account di archiviazione standard:** Un account di archiviazione standard ha una frequenza di richiesta totale massima di 20.000 IOPS. Il numero totale di IOPS in tutti i dischi delle macchine virtuali in un account di archiviazione standard non deve superare questo limite.
  
    È possibile calcolare approssimativamente il numero di dischi a elevato utilizzo supportato da un singolo account di archiviazione standard in base al limite di frequenza delle richieste. Per una VM di livello Basic, ad esempio, il numero massimo di dischi a elevato utilizzo è circa 66, ovvero 20.000/300 IOPS per disco. Il numero massimo di dischi a elevato utilizzo per una macchina virtuale di livello standard è pari a circa 40, ovvero 20.000/500 IOPS per disco. 

* **Per gli account di archiviazione Premium:** Un account di archiviazione Premium ha una velocità effettiva totale massima di 50 Gbps. La velocità effettiva totale in tutti i dischi della macchina virtuale non deve superare questo limite.

