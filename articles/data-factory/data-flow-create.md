---
title: Creare un flusso di dati di mapping Azure Data Factory
description: Come creare un flusso di dati di mapping Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/12/2019
ms.openlocfilehash: 31528c0ac7796aafbde74a3814d971138ec4ba2e
ms.sourcegitcommit: bb65043d5e49b8af94bba0e96c36796987f5a2be
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/16/2019
ms.locfileid: "72387255"
---
# <a name="create-azure-data-factory-data-flow"></a>Creare un flusso di dati di Azure Data Factory



I flussi di dati di mapping in Azure Data Factory consentono di trasformare i dati su larga scala senza scrivere codice. È possibile progettare un processo di trasformazione dei dati nella finestra di progettazione del flusso dei dati costruendo una serie di trasformazioni. Iniziare con un numero qualsiasi di trasformazioni di origine seguite da passaggi di trasformazione dei dati. Completare quindi il flusso di dati con sink per indirizzare i risultati a una destinazione.

Per iniziare, creare prima di tutto una nuova versione V2 Data Factory dalla portale di Azure. Dopo aver creato la nuova factory, fare clic sul riquadro "Author & Monitor" (Crea e monitora) per avviare l'interfaccia utente di Data Factory.

![Opzioni flusso di dati](media/data-flow/v2portal.png "creazione flusso di dati")

Nell'interfaccia utente di Data Factory è possibile usare flussi di dati di esempio. Gli esempi sono disponibili dalla raccolta di modelli di Azure Data Factory. In Azure Data Factory creare una pipeline da modello ("Pipeline from Template") e selezionare la categoria del flusso di dati dalla raccolta di modelli.

![Opzioni flusso di dati](media/data-flow/template.png "creazione flusso di dati")

Verrà richiesto di immettere le informazioni sull'account di Archiviazione BLOB di Azure.

![Opzioni flusso di dati](media/data-flow/template2.png "creazione flusso di dati 2")

[I dati usati per questi esempi sono disponibili qui](https://github.com/kromerm/adfdataflowdocs/tree/master/sampledata). Scaricare i dati di esempio e archiviare i file negli account di Archiviazione BLOB di Azure in modo che sia possibile eseguire gli esempi.

## <a name="create-new-data-flow"></a>Crea nuovo flusso di dati

Usare il pulsante Crea risorsa "segno più" nell'interfaccia utente di ADF per creare flussi di dati.

![Opzioni flusso di dati](media/data-flow/newresource.png "Nuova risorsa")

## <a name="next-steps"></a>Passaggi successivi

Iniziare a compilare la trasformazione dei dati con una [trasformazione di origine](data-flow-source.md).
