---
title: Registrazione per i servizi Web - Azure Machine Learning Studio | Microsoft Docs
description: Informazioni su come abilitare la registrazione per i servizi Web di Machine Learning Studio. La registrazione fornisce informazioni aggiuntive per risolvere i problemi relativi alle API.
services: machine-learning
documentationcenter: ''
author: xiaoharper
ms.custom: seodec18
ms.author: amlstudiodocs
editor: cgronlun
ms.assetid: c54d41e1-0300-46ef-bbfc-d6f7dca85086
ms.service: machine-learning
ms.subservice: studio
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/15/2017
ms.openlocfilehash: 727379edb60756ca8cb3e5ebdc29cd38858945e4
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60345636"
---
# <a name="enable-logging-for-azure-machine-learning-studio-web-services"></a>Abilitare la registrazione per i servizi Web di Azure Machine Learning Studio
Questo documento include informazioni sulla capacità di registrazione dei servizi Web di Machine Learning Studio. La registrazione offre informazioni aggiuntive per la risoluzione dei problemi delle API di Machine Learning Studio, non soltanto un semplice codice di errore e un messaggio.  

## <a name="how-to-enable-logging-for-a-web-service"></a>Come abilitare la registrazione per un servizio Web

La registrazione viene abilitata nel portale dei [servizi Web di Azure Machine Learning Studio](https://services.azureml.net). 

1. Accedere al portale dei servizi Web di Azure Machine Learning Studio all'indirizzo [https://services.azureml.net](https://services.azureml.net). Per un servizio Web classico, è possibile anche accedere al portale facendo clic su **New Web Services Experience** (Nuovi servizi Web) nella pagina dei servizi Web di Machine Learning Studio in Machine Learning Studio.

   ![Collegamento New Web Services Experience](./media/web-services-logging/new-web-services-experience-link.png)

2. Nella barra dei menu superiore fare clic su **Servizi Web** per un nuovo servizio Web oppure su **Classic Web Services** (Servizi Web classici) per un servizio Web classico.

   ![Selezione di servizi Web nuovi o classici](./media/web-services-logging/select-web-service.png)

3. Per un nuovo servizio Web, fare clic sul nome del servizio Web. Per un servizio Web classico, fare clic sul nome del servizio Web e nella pagina successiva fare clic sull'endpoint appropriato.

4. Nella barra dei menu superiore fare clic su **Configura**.

5. Impostare l'opzione **Abilita registrazione** su *Errore*, per registrare solo gli errori, o su *Tutti*, per la registrazione completa.

   ![Selezione del livello di registrazione](./media/web-services-logging/enable-logging.png)

6. Fare clic su **Save**.

7. Per i servizi Web classici, creare il contenitore **ml-diagnostics**.

   Tutti i log dei servizi Web vengono mantenuti in un contenitore BLOB denominato **ml-diagnostics** nell'account di archiviazione associato al servizio Web. Per i nuovi servizi Web, questo contenitore viene creato la prima volta in cui si accede al servizio Web. Per i servizi Web classici, è necessario creare il contenitore se non esiste già. 

   1. Nel [portale di Azure](https://portal.azure.com) accedere all'account di archiviazione associato al servizio Web.

   2. In **Servizio BLOB** fare clic su **Contenitori**.

   3. Se il contenitore **ml-diagnostics** non esiste, fare clic su **+ Contenitore**, assegnare al contenitore il nome "ml-diagnostics" e selezionare il **Tipo di accesso** "BLOB". Fare clic su **OK**.

      ![Creare un nuovo contenitore per archiviare i log di diagnostica](./media/web-services-logging/create-ml-diagnostics-container.png)

> [!TIP]
>
> Per un servizio Web classico, anche il dashboard dei servizi Web in Machine Learning Studio ha un'opzione per abilitare la registrazione. Tuttavia, poiché la registrazione ora è gestita tramite il portale dei servizi Web, è necessario abilitarla tramite il portale come descritto in questo articolo. Se la registrazione è già stata abilitata in Studio, nel portale dei servizi Web disabilitare la registrazione e abilitarla di nuovo.


## <a name="the-effects-of-enabling-logging"></a>Effetti dell'abilitazione della registrazione
Quando la registrazione è abilitata, tutti i dati di diagnostica e gli errori originati dall'endpoint del servizio Web vengono registrati nel contenitore BLOB **ml-diagnostics** nell'account di archiviazione di Azure collegato all'area di lavoro dell'utente. Tale contenitore contiene tutte le informazioni di diagnostica per tutti gli endpoint del servizio Web di tutte le aree di lavoro associate all'account di archiviazione.

I log possono essere visualizzati tramite uno dei diversi strumenti disponibili per l'esplorazione degli account di archiviazione di Azure. Il modo più semplice per visualizzarli è passare all'account di archiviazione nel portale di Azure e fare clic su **Contenitori** e quindi sul contenitore **ml-diagnostics**.  

## <a name="log-blob-detail-information"></a>Informazioni dettagliate dei BLOB dei log
Ogni BLOB nel contenitore contiene le informazioni di diagnostica per una delle azioni seguenti:

* Un'esecuzione del metodo Batch-Execution  
* Un'esecuzione del metodo Request-Response  
* Inizializzazione di un contenitore Request-Response

Il nome di ogni BLOB riporta un prefisso nel formato seguente: 


`{Workspace Id}-{Web service Id}-{Endpoint Id}/{Log type}`


Dove _Tipo di log_ corrisponde a uno dei valori seguenti:  

* o batch  
* punteggio/richieste  
* punteggio/init  

