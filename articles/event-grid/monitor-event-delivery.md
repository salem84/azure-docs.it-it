---
title: Monitorare il recapito dei messaggio di Griglia di eventi di Azure
description: Descrive come monitorare il recapito dei messaggi di Griglia di eventi di Azure.
services: event-grid
author: spelluru
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 01/22/2019
ms.author: spelluru
ms.openlocfilehash: fdd18b833794c25cb90188ba8bc418d4785492ba
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60824165"
---
# <a name="monitor-event-grid-message-delivery"></a>Monitorare il recapito dei messaggio di Griglia di eventi 

Questo articolo descrive come usare il portale per controllare i recapiti di eventi.

Griglia di eventi fornisce il recapito durevole. Ogni messaggio viene recapitato almeno una volta per ogni sottoscrizione. Gli eventi vengono inviati immediatamente al webhook registrato di ogni sottoscrizione. Se un webhook non conferma la ricezione di un evento entro 60 secondi dal primo tentativo di recapito, Griglia di eventi esegue un nuovo tentativo di recapito dell'evento.

Per informazioni sul recapito di eventi e sui nuovi tentativi, vedere [Recapito di messaggi di Griglia di eventi e nuovi tentativi](delivery-and-retry.md).

## <a name="delivery-metrics"></a>Metriche di recapito

Il portale visualizza le metriche per lo stato di recapito dei messaggi di evento.

Per gli argomenti le metriche sono:

* **Pubblicazione riuscita**: l'evento è stato inviato all'argomento ed elaborato con una risposta 2xx.
* **Pubblicazione non riuscita**: l'evento è stato inviato all'argomento, ma è stato rifiutato con un codice errore.
* **Senza corrispondenza**: l'evento è stato pubblicato nell'argomento, ma non abbinato a una sottoscrizione di eventi. L'evento è stato eliminato.

Per le sottoscrizioni le metriche sono:

* **Recapito riuscito**: l'evento è stato recapitato all'endpoint della sottoscrizione ed è stata ricevuta una risposta 2xx.
* **Recapito non riuscito**: l'evento è stato inviato all'endpoint della sottoscrizione, ma è stata ricevuta una risposta 4xx o 5xx.
* **Eventi scaduti**: l'evento non è stato recapitato e sono stati inviati tutti i tentativi di ripetizione del recapito. L'evento è stato eliminato.
* **Eventi abbinati**: l'evento nell'argomento è stato abbinato dalla sottoscrizione di eventi.

## <a name="event-subscription-status"></a>Stato della sottoscrizione di eventi

Per visualizzare le metriche per una sottoscrizione dell'evento, è possibile eseguire una ricerca per tipo di sottoscrizione o per sottoscrizioni per una risorsa specifica.

Per eseguire una ricerca per tipo di sottoscrizione dell'evento, selezionare **Tutti i servizi**.

![Selezionare Tutti i servizi](./media/monitor-event-delivery/all-services.png)

Cercare la **griglia di eventi** e selezionare **Sottoscrizioni di Griglia di eventi** tra le opzioni disponibili.

![Cercare sottoscrizioni di eventi](./media/monitor-event-delivery/search-and-select.png)

Filtrare in base al tipo di evento, alla sottoscrizione e alla località. Selezionare **Metrica** per la sottoscrizione da visualizzare.

![Filtrare sottoscrizioni di eventi](./media/monitor-event-delivery/filter-events.png)

Visualizzare le metriche per l'argomento e la sottoscrizione di eventi.

![Visualizzare le metriche degli eventi](./media/monitor-event-delivery/subscription-metrics.png)

Per trovare le metriche per una risorsa specifica, selezionare la risorsa. Selezionare quindi **Eventi**.

![Selezionare gli eventi per una risorsa](./media/monitor-event-delivery/select-events.png)

Vengono visualizzate le metriche per le sottoscrizioni per tale risorsa.

## <a name="custom-event-status"></a>Stato dell'evento personalizzato

Se è stato pubblicato un argomento personalizzato, è possibile visualizzare le relative metriche. Selezionare il gruppo di risorse per l'argomento e selezionare l'argomento.

![Selezionare l'argomento personalizzato](./media/monitor-event-delivery/select-custom-topic.png)

Visualizzare le metriche per l'argomento dell'evento personalizzato.

![Visualizzare le metriche degli eventi](./media/monitor-event-delivery/custom-topic-metrics.png)

## <a name="next-steps"></a>Passaggi successivi

* Per informazioni sul recapito di eventi e sui nuovi tentativi, vedere [Recapito di messaggi di Griglia di eventi e nuovi tentativi](delivery-and-retry.md).
* Per un'introduzione a Griglia di eventi, vedere [Informazioni su Griglia di eventi](overview.md).
* Per iniziare rapidamente a usare Griglia di eventi, vedere [Creare e instradare eventi personalizzati con Griglia di eventi di Azure](custom-event-quickstart.md).
