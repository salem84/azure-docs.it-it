---
title: Azure Marketplace SEO guidance
description: Fornisce indicazioni sull'ottimizzazione dell'uso del motore di ricerca (SEO).
services: Azure, Marketplace, Cloud Partner Portal,
author: v-miclar
ms.service: marketplace
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: pabutler
ms.openlocfilehash: f5b956ed1197e3898c9536bda3a93a41e8ee35c0
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "64935142"
---
# <a name="azure-marketplace-seo-guidance"></a>Azure Marketplace SEO guidance

Questo articolo illustra come ottimizzare l'individuabilità dell'offerta tramite la funzionalità di ricerca nel [Azure Marketplace](https://azuremarketplace.microsoft.com) e [AppSource](https://appsource.microsoft.com). 


## <a name="general-explanation-of-algorithm"></a>Descrizione generale dell'algoritmo

Marketplace Microsoft usano ricerca di Azure per potenziare le funzionalità di ricerca del sito. L'algoritmo si basa sulla frequenza del termine/frequenza inversa del documento ([TF/IDF](https://en.wikipedia.org/wiki/Tf–idf)). Viene usato [l'analizzatore Lucene](https://lucene.apache.org/core/) standard.

In generale, tutto i campi di testo, le categorie e i settori sono inclusi nella ponderazione della pertinenza. Termini specifici che vengono utilizzati raramente dalle app generiche, ma di frequente nella propria app genereranno un punteggio di corrispondenza maggiore nella ricerca. Ad esempio, termini come "VM" offriranno pochi vantaggi, mentre "Ricerca di Azure" potrebbe essere molto più specifico.
Di seguito, sono elencati i campi più importanti da considerare.

 
|  Campo                   | priorità | Materiale sussidiario                                                                                            |
|  --------------------    | ----------                   | ---------------                                                                   |
| Nome offerta               |  Alto      | Un valore esatto o vicino a una corrispondenza completa con la query di ricerca sospenderà la classificazione elevata.                       |
| Nome entità di pubblicazione           |  Alto      | Un valore esatto o vicino a una corrispondenza completa con la query di ricerca sospenderà la classificazione elevata.                       |
| Descrizione breve        |  Media    | I nomi specifici di app e di pubblicazione potrebbero garantire una classificazione elevata, ma non risultare altrettanto pertinenti. In questo caso, una breve descrizione è fondamentale. Creare un testo conciso e rilevante. Le parole chiavi e i termini di ricerca previsti devono essere inclusi per ottenere risultati migliori.  Ad esempio: "Questo è il migliore Retail POS basato completamente su Dynamics 365" è meno efficace rispetto a "Retail POS (punto vendita) per Dynamics 365".  | 
| Descrizione lunga         |  Basso       | La descrizione è un modo per fornire maggiori indicazioni. Le descrizioni più efficaci sono di una lunghezza ragionevole e impiegano parole chiavi.  Una descrizione concisa, completa di parole chiavi, otterrà un maggiore vantaggio rispetto a descrizioni più lunghe. Assicurarsi di includere parole chiavi nella descrizione, ad esempio "IoT".  |
| Categorie di prodotto       | Media     |  Le categorie di prodotti dipendono da una combinazione di opzioni di pubblicazione e Microsoft. Selezionare queste categorie in modo appropriato, in modo che gli utenti possano trovare facilmente le app nella categoria corretta. |
|  |  |  |


## <a name="other-tips"></a>Altri suggerimenti

-   I suggerimenti di Ricerca sono ampiamente usati dall'utente. Assegnano la priorità alle corrispondenze rispetto a quella nome app/server di pubblicazione. Una breve descrizione diventa il campo fondamentale quando il termine di ricerca non corrisponde esattamente con il nome di server di pubblicazione/app.
-   I documenti per il download non sono inclusi nella ponderazione di ricerca.
-   L'acquisizione e l'uso effettivo dell'app influirà sulla ricerca nonché sulla classificazione. Ad esempio, tra due app equivalenti, sarà l'app con un maggiore numero di utenti a ottenere una classificazione migliore.
