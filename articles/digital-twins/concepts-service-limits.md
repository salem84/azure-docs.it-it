---
title: Limiti del servizio durante l'anteprima pubblica di Gemelli digitali di Azure | Microsoft Docs
description: Informazioni sui limiti del servizio durante l'anteprima pubblica di Gemelli digitali di Azure.
ms.author: alinast
author: alinamstanciu
manager: bertvanhoof
ms.service: digital-twins
services: digital-twins
ms.topic: conceptual
ms.date: 10/03/2019
ms.openlocfilehash: 88a66506b5102a4efaeefe1c7033b337dff9889a
ms.sourcegitcommit: 4f7dce56b6e3e3c901ce91115e0c8b7aab26fb72
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/04/2019
ms.locfileid: "71949251"
---
# <a name="public-preview-service-limits"></a>Limiti del servizio durante l'anteprima pubblica

Durante l'anteprima pubblica, a Gemelli digitali di Azure vengono applicati limiti temporanei relativi a sottoscrizioni, istanze e quantità.

Questi vincoli sono stati definiti per semplificare l'apprendimento del nuovo servizio e delle sue numerose funzionalità.

> [!NOTE]
> Questi limiti verranno aumentati o rimossi in base alla disponibilità generale.

## <a name="per-subscription-limits"></a>Limiti relativi alle sottoscrizioni

Durante l'anteprima pubblica, ogni sottoscrizione di Azure può creare o eseguire una sola istanza di Gemelli digitali di Azure per volta.

> [!TIP]
> Se si elimina l'istanza, sarà possibile crearne una nuova.

## <a name="per-instance-limits"></a>Limiti relativi alle istanze

A sua volta, ogni istanza di Gemelli digitali di Azure può avere:

- Esattamente una risorsa **IoTHub** incorporata che viene creata automaticamente durante il provisioning del servizio.
- Esattamente un endpoint **EventHub** per il tipo di evento **DeviceMessage**.
- Fino a tre endpoint **EventHub**, **ServiceBus** o **EventGrid** per il tipo di evento **SensorChange**, **SpaceChange**, **TopologyOperation** o **UdfCustom**.

> [!NOTE]
> Alcuni parametri che vengono in genere definiti durante la creazione delle entità di Azure IoT indicate sopra non sono necessari durante l'anteprima pubblica.
> - Consultare la [documentazione di riferimento di Swagger](./how-to-use-swagger.md) per le specifiche di API più recenti.

## <a name="azure-digital-twins-management-api-limits"></a>Limiti dell'API di gestione di Gemelli digitali di Azure

I limiti per il numero di richieste per l'API di gestione di Gemelli digitali di Azure sono i seguenti:

- 100 richieste al secondo all'API di gestione di Gemelli digitali di Azure.
- Fino a 1000 oggetti restituiti da una singola query sull'API di gestione di Gemelli digitali di Azure.

> [!IMPORTANT]
> Se si supera il limite di 1.000 oggetti, si riceverà un errore e sarà necessario semplificare la query.

## <a name="user-defined-functions-rate-limits"></a>Limiti al numero di funzioni definite dall'utente

I limiti seguenti definiscono il numero totale di tutte le chiamate delle funzioni definite dall'utente inviate all'istanza di Gemelli digitali di Azure:

- 400 chiamate alla libreria client al secondo
- 100 chiamate **SendNotification** al secondo

> [!NOTE]
> Le operazioni seguenti potrebbero generare l'applicazione temporanea di altri limiti:
> - Modifiche ai metadati di oggetti della topologia
> - Aggiornamenti apportati alla definizione di funzione definita dall'utente
> - Dispositivi che inviano dati di telemetria per la prima volta

## <a name="device-telemetry-limits"></a>Limiti relativi alla telemetria dei dispositivi

I limiti seguenti si applicano al numero totale di tutti i messaggi che i dispositivi possono inviare all'istanza di Gemelli digitali di Azure:

- 100 messaggi al secondo

## <a name="next-steps"></a>Passaggi successivi

- Per provare un esempio di Gemelli digitali di Azure, vedere [Guida introduttiva per la ricerca di stanze disponibili](./quickstart-view-occupancy-dotnet.md).