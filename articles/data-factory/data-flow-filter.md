---
title: Trasformazione filtro nel flusso di dati del mapping Azure Data Factory | Microsoft Docs
description: Filtrare le righe utilizzando la trasformazione filtro nel flusso di dati del mapping Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/16/2019
ms.openlocfilehash: a4dd53f37a8a963d05a3ad9c49769528e945f6a1
ms.sourcegitcommit: f29fec8ec945921cc3a89a6e7086127cc1bc1759
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/17/2019
ms.locfileid: "72527395"
---
# <a name="filter-transformation-in-mapping-data-flow"></a>Trasformazione filtro nel flusso di dati di mapping

Le trasformazioni di filtro consentono di filtrare le righe in base a una condizione. Il flusso di output include tutte le righe che corrispondono alla condizione di filtro. La trasformazione filtro è simile a una clausola WHERE in SQL.

## <a name="configuration"></a>Configurazione

Utilizzare il generatore di espressioni del flusso di dati per immettere un'espressione per la condizione di filtro. Per aprire il generatore di espressioni, fare clic sulla casella blu. La condizione di filtro deve essere di tipo booleano. Per ulteriori informazioni su come creare un'espressione, vedere la documentazione del [Generatore di espressioni](concepts-data-flow-expression-builder.md) .

![Filtra trasformazione](media/data-flow/filter1.png "Filtra trasformazione")

## <a name="data-flow-script"></a>Script del flusso di dati

### <a name="syntax"></a>Sintassi

```
<incomingStream>
    filter(
        <conditionalExpression>
    ) ~> <filterTransformationName>
```

### <a name="example"></a>Esempio

Nell'esempio seguente viene illustrata una trasformazione Suddivisione condizionale denominata `FilterBefore1960` che accetta `CleanData` flusso in ingresso. La condizione di filtro è l'espressione `year <= 1960`.

In Data Factory UX questa trasformazione è simile all'immagine seguente:

![Filtra trasformazione](media/data-flow/filter1.png "Filtra trasformazione")

Lo script del flusso di dati per questa trasformazione si trova nel frammento di codice seguente:

```
CleanData
    filter(
        year <= 1960
    ) ~> FilterBefore1960

```

## <a name="next-steps"></a>Passaggi successivi

Filtrare le colonne con la [trasformazione Select](data-flow-select.md)
