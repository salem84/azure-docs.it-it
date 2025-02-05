---
title: 'Flat file di codifica o decodifica: App per la logica di Azure | Microsoft Docs'
description: Codificare o decodificare file flat per l'integrazione aziendale con App per la logica di Azure e Enterprise Integration Pack
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 82152dab-c7ad-43df-b721-596559703be8
ms.date: 07/08/2016
ms.openlocfilehash: d0ef61b94d7bd604b6c0062341224510f3048c57
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "61467262"
---
# <a name="encode-or-decode-flat-files-with-azure-logic-apps-and-enterprise-integration-pack"></a>Codificare o decodificare file flat con App per la logica di Azure e Enterprise Integration Pack

In uno scenario B2B è consigliabile codificare i contenuti XML prima di inviarli a un partner commerciale. In un'app per la logica, è possibile usare a tale scopo il connettore di codifica file flat. L'app per la logica che viene creata può ottenere il relativo contenuto XML da varie origini, come un trigger di richiesta HTTP, un'altra applicazione o uno dei numerosi [connettori](../connectors/apis-list.md). Per altre informazioni sulle app per la logica, vedere la [documentazione relativa alle app per la logica](logic-apps-overview.md "Altre informazioni sulle app per la logica").  

## <a name="create-the-flat-file-encoding-connector"></a>Come creare il connettore di codifica flat file
Per aggiungere un connettore di codifica file flat all'app per la logica, seguire questa procedura.

1. Creare un'app per la logica e [collegarla all'account di integrazione](logic-apps-enterprise-integration-accounts.md "Informazioni su come collegare un account di integrazione a un'app per la logica"). Questo account contiene lo schema che verrà usato per codificare i dati XML.  
1. Aggiungere un trigger **Richiesta - Alla ricezione di una richiesta HTTP** all'app per la logica.  
   ![Screenshot del trigger da selezionare](./media/logic-apps-enterprise-integration-b2b/flatfile-1.png)    
1. Aggiungere l'azione di codifica file flat, come illustrato di seguito:
   
    a. Selezionare il segno **più** .
   
    b. Selezionare il collegamento **Aggiungi un'azione** che viene visualizzato dopo aver selezionato il segno più.
   
    c. Immettere *Flat* nella casella di ricerca per filtrare tutte le azioni su quella che si desidera usare.
   
    d. Selezionare l'opzione **Codifica file flat** nell'elenco.   
   ![Screenshot dell'opzione Codifica file flat](media/logic-apps-enterprise-integration-flatfile/flatfile-2.png)   
1. Nella finestra di dialogo **Codifica file flat** selezionare la casella di testo **Contenuto**.  
   ![Screenshot della casella di testo Contenuto](media/logic-apps-enterprise-integration-flatfile/flatfile-3.png)  
1. Selezionare il tag del corpo come contenuto che verrà codificato. Il tag body popolerà il campo del contenuto.     
   ![Screenshot del tag body](media/logic-apps-enterprise-integration-flatfile/flatfile-4.png)  
1. Selezionare la casella di riepilogo **Nome schema** e scegliere lo schema da usare per codificare il contenuto di input.    
   ![Screenshot della casella di riepilogo Nome schema](media/logic-apps-enterprise-integration-flatfile/flatfile-5.png)  
1. Salvare il lavoro.   
   ![Screenshot dell'icona Salva](media/logic-apps-enterprise-integration-flatfile/flatfile-6.png)  

A questo punto la configurazione del connettore di codifica del file flat è completa. Nel caso di un'applicazione reale è consigliabile archiviare i dati codificati in un'applicazione line-of-business, ad esempio Salesforce. In alternativa è possibile inviare i dati codificati a un partner commerciale. È possibile aggiungere facilmente un'azione per inviare l'output dell'azione di codifica a Salesforce oppure a un partner commerciale usando uno degli altri connettori forniti.

Ora è possibile testare il connettore. A tale scopo, inviare una richiesta all'endpoint HTTP e includere il contenuto XML nel corpo della richiesta.  

## <a name="create-the-flat-file-decoding-connector"></a>Come creare il connettore di decodifica flat file

> [!NOTE]
> Per completare questi passaggi è necessario un file di schema già caricato nell'account di integrazione.

1. Aggiungere un trigger **Richiesta - Alla ricezione di una richiesta HTTP** all'app per la logica.  
   ![Screenshot del trigger da selezionare](./media/logic-apps-enterprise-integration-b2b/flatfile-1.png)    
1. Aggiungere l'azione di decodifica file flat, come illustrato di seguito:
   
    a. Selezionare il segno **più** .
   
    b. Selezionare il collegamento **Aggiungi un'azione** che viene visualizzato dopo aver selezionato il segno più.
   
    c. Immettere *Flat* nella casella di ricerca per filtrare tutte le azioni su quella che si desidera usare.
   
    d. Selezionare l'opzione **Decodifica file flat** nell'elenco.   
   ![Screenshot dell'opzione Decodifica file flat](media/logic-apps-enterprise-integration-flatfile/flatfile-2.png)   
1. Selezionare il controllo **Contenuto** . Verrà restituito un elenco del contenuto dai passaggi precedenti che è possibile usare come contenuto da decodificare. Si noti che il *Corpo* della richiesta HTTP in ingresso è disponibile per l'uso come contenuto da decodificare. Si noti che è possibile immettere anche il contenuto da decodificare direttamente nel controllo **Contenuto** .     
1. Selezionare il tag *body* . Si noti che il tag body è ora nel controllo **Contenuto** .
1. Selezionare il nome dello schema da usare per decodificare il contenuto. La schermata seguente mostra che *OrderFile* è il nome di schema selezionato. Questo nome di schema era stato caricato in precedenza nell'account di integrazione.
   
   ![Screenshot della finestra di dialogo Decodifica file flat](media/logic-apps-enterprise-integration-flatfile/flatfile-decode-1.png)    
1. Salvare il lavoro.  
   ![Screenshot dell'icona Salva](media/logic-apps-enterprise-integration-flatfile/flatfile-6.png)    

A questo punto la configurazione del connettore di decodifica file flat è completa. Nel caso di un'applicazione reale è consigliabile archiviare i dati decodificati in un'applicazione line-of-business, ad esempio Salesforce. È possibile eseguire facilmente questa azione inviando l'output della decodifica a Salesforce.

È ora possibile testare il connettore effettuando una richiesta all'endpoint HTTP e includendo il contenuto XML da decodificare nel corpo della richiesta.  

## <a name="next-steps"></a>Passaggi successivi
* [Altre informazioni su Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md "Informazioni su Enterprise Integration Pack").  

