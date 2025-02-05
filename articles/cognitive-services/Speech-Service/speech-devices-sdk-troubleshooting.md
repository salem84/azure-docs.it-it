---
title: Risolvere i problemi relativi all'SDK dei dispositivi vocali-servizio riconoscimento vocale
titleSuffix: Azure Cognitive Services
description: Questo articolo fornisce informazioni che consentono di risolvere i problemi che possono verificarsi quando si usa l'SDK per dispositivi vocali.
services: cognitive-services
author: mswellsi
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: wellsi
ms.openlocfilehash: 9a757402360b4b69e760315a809482be9b4e04d4
ms.sourcegitcommit: 7c4de3e22b8e9d71c579f31cbfcea9f22d43721a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/26/2019
ms.locfileid: "68558909"
---
# <a name="troubleshoot-the-speech-devices-sdk"></a>Risolvere i problemi di Speech Devices SDK

Questo articolo fornisce informazioni che consentono di risolvere i problemi che possono verificarsi quando si usa l'SDK per dispositivi vocali.

## <a name="certificate-failures"></a>Errori di certificato

Se si verificano errori di certificato quando si usa il servizio di riconoscimento vocale, assicurarsi che la data e l'ora del dispositivo siano corrette:

1. Passare a **Impostazioni**. In **System** (Sistema) selezionare **Date & time** (Data e ora).

    ![In Settings (Impostazioni) selezionare Date & time (Data e ora)](media/speech-devices-sdk/qsg-12.png)

1. Mantenere selezionata l'opzione **Automatic date & time** (Data e ora automatiche). In **Select time zone** (Seleziona il fuso orario) selezionare il fuso orario corrente.

    ![Opzioni per la selezione della data e del fuso orario](media/speech-devices-sdk/qsg-13.png)

    Dopo aver verificato che l'ora del kit di sviluppo corrisponde a quella del PC, il kit di sviluppo è connesso a Internet.

## <a name="next-steps"></a>Passaggi successivi

* [Esaminare le note sulla versione](devices-sdk-release-notes.md)
