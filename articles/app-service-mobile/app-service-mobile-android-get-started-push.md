---
title: Aggiungere notifiche push all'app Android con App per dispositivi mobili | Documentazione Microsoft
description: Informazioni su come usare App per dispositivi mobili per inviare notifiche push all'app Android.
services: app-service\mobile
documentationcenter: android
manager: crdun
editor: ''
author: elamalani
ms.assetid: 9058ed6d-e871-4179-86af-0092d0ca09d3
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: article
ms.date: 06/25/2019
ms.author: emalani
ms.openlocfilehash: 539859ba73c8a26d6f7e8f25b9e7453d987a52bd
ms.sourcegitcommit: bb65043d5e49b8af94bba0e96c36796987f5a2be
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/16/2019
ms.locfileid: "72389019"
---
# <a name="add-push-notifications-to-your-android-app"></a>Aggiungere notifiche push all'app Android

[!INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

> [!NOTE]
> Visual Studio App Center supporta servizi end-to-end e integrati fondamentali per lo sviluppo di app per dispositivi mobili. Gli sviluppatori possono usare i servizi **Build**, **Test** e **Distribute** per configurare una pipeline di integrazione e distribuzione continue. Dopo la distribuzione dell'app, gli sviluppatori possono monitorarne lo stato e l'utilizzo tramite i servizi **Analytics** e **Diagnostics** e interagire con gli utenti tramite il servizio **Push**. Gli sviluppatori possono anche usare il servizio **Auth** per autenticare gli utenti e il servizio **Data** per salvare e sincronizzare i dati dell'app nel cloud.
>
> Per integrare i servizi cloud nelle applicazioni per dispositivi mobili, iscriversi ad [App Center](https://appcenter.ms/signup?utm_source=zumo&utm_medium=Azure&utm_campaign=zumo%20doc).

## <a name="overview"></a>Panoramica

In questa esercitazione vengono aggiunte notifiche push al progetto [avvio rapido di Android], in modo che una notifica push venga inviata al dispositivo a ogni inserimento di record.

Se non si usa il progetto server di avvio rapido scaricato, è necessario aggiungere il pacchetto di estensione di notifica push. Per altre informazioni, vedere [Usare l'SDK del server back-end .NET per App per dispositivi mobili di Azure](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md).

## <a name="prerequisites"></a>Prerequisiti

Sono necessari gli elementi seguenti:

* Un IDE, in base al back-end del progetto:

  * [Android Studio](https://developer.android.com/sdk/index.html) se questa app ha un back-end Node.js.
  * [Visual Studio Community 2013](https://go.microsoft.com/fwLink/p/?LinkID=391934) o versione successiva se questa app ha un back-end Microsoft .NET.
* Android 2.3 o versione successiva, Google Repository 27 o versione successiva e Google Play Services 9.0.2 o versione successiva per Firebase Cloud Messaging.
* Completare l'[avvio rapido di Android].

## <a name="create-a-project-that-supports-firebase-cloud-messaging"></a>Creare un progetto che supporta Google Cloud Messaging

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]

## <a name="configure-a-notification-hub"></a>Configurare un hub di notifica

[!INCLUDE [app-service-mobile-configure-notification-hub](../../includes/app-service-mobile-configure-notification-hub.md)]

## <a name="configure-azure-to-send-push-notifications"></a>Configurare Azure per l'invio di notifiche push

[!INCLUDE [app-service-mobile-android-configure-push](../../includes/app-service-mobile-android-configure-push-for-firebase.md)]

## <a name="enable-push-notifications-for-the-server-project"></a>Abilitare le notifiche push per il progetto server

[!INCLUDE [app-service-mobile-dotnet-backend-configure-push-google](../../includes/app-service-mobile-dotnet-backend-configure-push-google.md)]

## <a name="add-push-notifications-to-your-app"></a>Aggiungere notifiche push all'app

In questa sezione l'app del client Android viene aggiornata in modo da gestire le notifiche push.

### <a name="verify-android-sdk-version"></a>Verificare la versione di Android SDK

[!INCLUDE [app-service-mobile-verify-android-sdk-version](../../includes/app-service-mobile-verify-android-sdk-version.md)]

Il passaggio successivo comporta l'installazione di Google Play Services. Firebase Cloud Messaging prevede alcuni requisiti minimi a livello di API per lo sviluppo e i test. È necessario che la proprietà **minSdkVersion** nel manifesto sia conforme a tali requisiti.

In caso di test con un dispositivo meno recente, vedere [Aggiungere Firebase al progetto Android] (Aggiungere Firebase a un progetto Android) per determinare il livello minimo su cui è possibile impostare tale valore e applicare l'impostazione corretta.

### <a name="add-firebase-cloud-messaging-to-the-project"></a>Aggiungere Firebase Cloud Messaging al progetto

[!INCLUDE [Add Firebase Cloud Messaging](../../includes/app-service-mobile-add-firebase-cloud-messaging.md)]

### <a name="add-code"></a>Aggiungere codice

[!INCLUDE [app-service-mobile-android-getting-started-with-push](../../includes/app-service-mobile-android-getting-started-with-push.md)]

## <a name="test-the-app-against-the-published-mobile-service"></a>Eseguire il test dell'app sul servizio mobile pubblicato

È possibile eseguire il test dell'app collegando direttamente un telefono Android con un cavo USB oppure usando un dispositivo virtuale nell'emulatore.

## <a name="next-steps"></a>Passaggi successivi

Dopo aver completato questa esercitazione, continuare con una delle esercitazioni seguenti:

* [Aggiungere l'autenticazione all'app Android](app-service-mobile-android-get-started-users.md).
  Questa esercitazione spiega come aggiungere l'autenticazione al progetto introduttivo TodoList in Android tramite un provider di identità supportato.
* [Attivare la sincronizzazione offline per l'app Android](app-service-mobile-android-get-started-offline-data.md).
  Informazioni su come aggiungere il supporto offline all'app usando il back-end di App per dispositivi mobili. Con la sincronizzazione offline è possibile interagire con un'app per dispositivi mobili &mdash;visualizzando, aggiungendo e modificando i dati&mdash; anche quando non è disponibile una connessione di rete.

<!-- URLs -->
[avvio rapido di Android]: app-service-mobile-android-get-started.md
[Aggiungere Firebase al progetto Android]: https://firebase.google.com/docs/android/setup
