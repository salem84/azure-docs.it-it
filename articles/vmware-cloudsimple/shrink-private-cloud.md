---
title: Compatta la soluzione VMware di Azure di CloudSimple cloud privato
description: Viene descritto come compattare un cloud privato CloudSimple.
author: sharaths-cs
ms.author: b-shsury
ms.date: 07/01/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 00d0067fe51efef8efffe31860e16f4530adf3ac
ms.sourcegitcommit: 47b00a15ef112c8b513046c668a33e20fd3b3119
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/22/2019
ms.locfileid: "69972360"
---
# <a name="shrink-a-cloudsimple-private-cloud"></a>Compattare un cloud privato CloudSimple

CloudSimple offre la flessibilità necessaria per compattare dinamicamente un cloud privato.  Un cloud privato è costituito da uno o più cluster vSphere. Ogni cluster può avere da 3 a 16 nodi. Quando si compatta un cloud privato, si rimuove un nodo dal cluster esistente o si elimina un intero cluster. 

## <a name="before-you-begin"></a>Prima di iniziare

Per la compattazione di un cloud privato devono essere soddisfatte le condizioni seguenti.  Non è possibile eliminare il cluster di gestione (primo cluster) creato quando è stato creato un cloud privato.

* Un cluster vSphere deve avere tre nodi.  Un cluster con solo tre nodi non può essere compattato.
* Lo spazio di archiviazione totale utilizzato non deve superare la capacità totale dopo la compattazione del cluster. 

## <a name="sign-in-to-azure"></a>Accedi ad Azure

Accedere al portale di Azure all'indirizzo [https://portal.azure.com](https://portal.azure.com).

## <a name="shrink-a-private-cloud"></a>Compattare un cloud privato

1. [Accedere al portale di CloudSimple](access-cloudsimple-portal.md).

2. Aprire la pagina **risorse** .

3. Fare clic sul cloud privato che si vuole compattare

4. Nella pagina Riepilogo fare clic su **compatta**.

    ![Compatta cloud privato](media/shrink-private-cloud.png)

5. Selezionare il cluster che si desidera compattare o eliminare. 

    ![Compattazione del cloud privato: selezionare un cluster](media/shrink-private-cloud-select-cluster.png)

6. Selezionare **Rimuovi un nodo** o **Elimina l'intero cluster**. 

7. Verificare la capacità del cluster

8. Fare clic su **Invia** per compattare il cloud privato.

Viene avviata la compattazione del cloud privato.  È possibile monitorare lo stato di avanzamento nelle attività.  Il processo di compattazione può richiedere alcune ore a seconda dei dati, che devono essere risincronizzati in rete VSAN.

## <a name="next-steps"></a>Passaggi successivi

* [Utilizzare macchine virtuali VMware in Azure](quickstart-create-vmware-virtual-machine.md)
* Altre informazioni sui [cloud privati](cloudsimple-private-cloud.md)
