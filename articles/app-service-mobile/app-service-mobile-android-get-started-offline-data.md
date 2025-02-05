---
title: Abilitare la sincronizzazione offline per l'app per dispositivi mobili di Azure (Android)
description: Informazioni su come usare le app per dispositivi mobili del servizio app per memorizzare nella cache e sincronizzare i dati offline in un'applicazione Android
documentationcenter: android
author: elamalani
manager: crdun
services: app-service\mobile
ms.assetid: 32a8a079-9b3c-4faf-8588-ccff02097224
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: article
ms.date: 06/25/2019
ms.author: emalani
ms.openlocfilehash: 9c845c09c6b05436017f98323dfa78185cd58aa6
ms.sourcegitcommit: bb65043d5e49b8af94bba0e96c36796987f5a2be
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/16/2019
ms.locfileid: "72388989"
---
# <a name="enable-offline-sync-for-your-android-mobile-app"></a>Abilitare la sincronizzazione offline per l'app per dispositivi mobili di Android
[!INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

> [!NOTE]
> Visual Studio App Center supporta servizi end-to-end e integrati fondamentali per lo sviluppo di app per dispositivi mobili. Gli sviluppatori possono usare i servizi **Build**, **Test** e **Distribute** per configurare una pipeline di integrazione e distribuzione continue. Dopo la distribuzione dell'app, gli sviluppatori possono monitorarne lo stato e l'utilizzo tramite i servizi **Analytics** e **Diagnostics** e interagire con gli utenti tramite il servizio **Push**. Gli sviluppatori possono anche usare il servizio **Auth** per autenticare gli utenti e il servizio **Data** per salvare e sincronizzare i dati dell'app nel cloud.
>
> Per integrare i servizi cloud nelle applicazioni per dispositivi mobili, iscriversi ad [App Center](https://appcenter.ms/signup?utm_source=zumo&utm_medium=Azure&utm_campaign=zumo%20doc).

## <a name="overview"></a>Panoramica
Questa esercitazione descrive la funzionalità di sincronizzazione offline delle app per dispositivi mobili di Azure per Android. La sincronizzazione offline consente agli utenti finali di interagire con un'app&mdash;visualizzando, aggiungendo e modificando i dati&mdash;anche se non è disponibile una connessione di rete. Le modifiche vengono archiviate in un database locale. Quando il dispositivo torna online, vengono sincronizzate con il back-end remoto.

Se questa è la prima esperienza con le app per dispositivi mobili di Azure, è consigliabile completare prima l'esercitazione [Creare un'app Android]. Se non si usa il progetto server di avvio rapido scaricato, è necessario aggiungere al progetto il pacchetto di estensione per l'accesso ai dati. Per altre informazioni sui pacchetti di estensione server, vedere l'articolo relativo all' [utilizzo dell'SDK del server back-end .NET per app per dispositivi mobili di Azure](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md).

Per altre informazioni sulla funzionalità di sincronizzazione offline, vedere l'argomento [Sincronizzazione di dati offline nelle app per dispositivi mobili di Azure].

## <a name="update-the-app-to-support-offline-sync"></a>Aggiornare l'app per supportare la sincronizzazione offline
Con la sincronizzazione offline si legge e si scrive da una *tabella di sincronizzazione* (usando l'interfaccia *IMobileServiceSyncTable*), che fa parte di un database **SQLite** nel dispositivo.

Per eseguire il push e il pull delle modifiche tra il dispositivo e Servizi mobili di Azure, si usa un *contesto di sincronizzazione* (*MobileServiceClient.SyncContext*), inizializzato con il database locale per archiviare localmente i dati.

1. In `TodoActivity.java`, impostare come commento la definizione esistente di `mToDoTable` e rimuovere il commento della versione della tabella di sincronizzazione:
   
        private MobileServiceSyncTable<ToDoItem> mToDoTable;
2. Nel metodo `onCreate`, impostare come commento l'inizializzazione esistente di `mToDoTable` e rimuovere il commento di questa definizione:
   
        mToDoTable = mClient.getSyncTable("ToDoItem", ToDoItem.class);
3. In `refreshItemsFromTable`, impostare come commento la definizione di `results` e rimuovere il commento di questa definizione:
   
        // Offline Sync
        final List<ToDoItem> results = refreshItemsFromMobileServiceTableSyncTable();
4. Impostare come commento la definizione di `refreshItemsFromMobileServiceTable`.
5. Rimuovere il commento dalla definizione di `refreshItemsFromMobileServiceTableSyncTable`:
   
        private List<ToDoItem> refreshItemsFromMobileServiceTableSyncTable() throws ExecutionException, InterruptedException {
            //sync the data
            sync().get();
            Query query = QueryOperations.field("complete").
                    eq(val(false));
            return mToDoTable.read(query).get();
        }
6. Rimuovere il commento dalla definizione di `sync`:
   
        private AsyncTask<Void, Void, Void> sync() {
            AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
                @Override
                protected Void doInBackground(Void... params) {
                    try {
                        MobileServiceSyncContext syncContext = mClient.getSyncContext();
                        syncContext.push().get();
                        mToDoTable.pull(null).get();
                    } catch (final Exception e) {
                        createAndShowDialogFromTask(e, "Error");
                    }
                    return null;
                }
            };
            return runAsyncTask(task);
        }

## <a name="test-the-app"></a>Testare l'app
In questa sezione viene testato il comportamento dell'app con il WiFi attivato e quindi il WiFi viene disattivato per creare uno scenario offline.

Quando si aggiungono elementi di dati, gli elementi vengono archiviati nell'archivio di SQL Lite, ma vengono sincronizzati con il servizio mobile solo dopo la selezione del pulsante di **aggiornamento** . Altre app potrebbero avere requisiti diversi relativi a quando è necessario sincronizzare i dati, ma per, finalità dimostrative, in questa esercitazione la sincronizzazione viene richiesta esplicitamente dall'utente.

Quando si preme il pulsante, viene avviata una nuova attività in background. Viene effettuato prima di tutto il push di tutte le modifiche apportate all'archivio locale usando il contesto di sincronizzazione, quindi viene effettuato il pull di tutti i dati modificati da Azure alla tabella locale.

### <a name="offline-testing"></a>Test offline
1. Attivare la *Modalità aereo*per il dispositivo o il simulatore. in modo da creare uno scenario offline.
2. Aggiungere alcuni elementi *ToDo* o contrassegnare alcuni elementi come completati. Uscire dal dispositivo o dal simulatore (o forzare la chiusura dell'app) e riavviare. Verificare che le modifiche siano state rese persistenti nel dispositivo, poiché sono incluse nell'archivio SQLite locale.
3. Visualizzare il contenuto della tabella *TodoItem* di Azure mediante uno strumento SQL, come ad esempio *SQL Server Management Studio*, o mediante un client REST, ad esempio *Fiddler* o *Postman*. Verificare che i nuovi elementi *non* siano stati sincronizzati con il server
   
       + Per un back-end Node.js, passare al [portale di Azure](https://portal.azure.com/) e nel back-end dell'app per dispositivi mobili fare clic su **Tabelle semplici** > **TodoItem** per visualizzare il contenuto della tabella `TodoItem`.
       + Per il back-end .NET, visualizzare il contenuto della tabella mediante uno strumento SQL, come ad esempio *SQL Server Management Studio*, o mediante un client REST, ad esempio *Fiddler* o *Postman*.
4. Attivare il WiFi nel dispositivo o nel simulatore. Premere quindi il pulsante di **aggiornamento** .
5. Visualizzare di nuovo i dati TodoItem nel portale di Azure. Dovrebbero essere visualizzati gli elementi TodoItems nuovi e modificati.

## <a name="additional-resources"></a>Risorse aggiuntive
* [Sincronizzazione di dati offline nelle app per dispositivi mobili di Azure]
* [Cloud Cover: Sincronizzazione offline in Servizi mobili di Azure]\(nota: il video è relativo ai Servizi mobili, ma il funzionamento della sincronizzazione offline è simile nelle app per dispositivi mobili di Azure\)

<!-- URLs. -->

[Sincronizzazione di dati offline nelle app per dispositivi mobili di Azure]: app-service-mobile-offline-data-sync.md

[Creare un'app Android]: app-service-mobile-android-get-started.md

[Cloud Cover: Sincronizzazione offline in Servizi mobili di Azure]: https://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Azure Friday: Offline-enabled apps in Azure Mobile Services]: https://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/

