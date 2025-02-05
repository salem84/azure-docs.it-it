---
title: Gestire i metadati degli elementi degli account di integrazione - App per la logica di Azure | Microsoft Docs
description: Aggiungere o ottenere i metadati degli elementi dagli account di integrazione in App per la logica di Azure con Enterprise Integration Pack
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: bb7d9432-b697-44db-aa88-bd16ddfad23f
ms.date: 01/17/2019
ms.openlocfilehash: 5ebdf45bec4e7cfceb75354af40c7a21c22c6eef
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60846202"
---
# <a name="manage-artifact-metadata-in-integration-accounts-with-azure-logic-apps-and-enterprise-integration-pack"></a>Gestire i metadati degli elementi dagli account di integrazione con App per la logica di Azure ed Enterprise Integration Pack

È possibile definire i metadati personalizzati per gli elementi negli account di integrazione e recuperarli durante il runtime per l'uso da parte dell'app per la logica. Ad esempio, è possibile fornire i metadati per elementi quali partner, contratti, schemi e mappe. Si tratta in ogni caso di metadati archiviati tramite coppie chiave-valore. 

## <a name="prerequisites"></a>Prerequisiti

* Una sottoscrizione di Azure. Se non si ha una sottoscrizione, è possibile <a href="https://azure.microsoft.com/free/" target="_blank">iscriversi per creare un account Azure gratuito</a>.

* Un [account di integrazione](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) di base con gli elementi in cui si vogliono aggiungere metadati, ad esempio: 

  * [Partner](logic-apps-enterprise-integration-partners.md)
  * [Contratto](logic-apps-enterprise-integration-agreements.md)
  * [Schema](logic-apps-enterprise-integration-schemas.md)
  * [Mappa](logic-apps-enterprise-integration-maps.md)

* Un'app per la logica collegata all'account di integrazione e ai metadati dell'elemento che si vogliono usare. Se l'app per la logica non è già collegata, vedere l'articolo su [come collegare app per la logica agli account di integrazione](logic-apps-enterprise-integration-create-integration-account.md#link-account). 

  Se si ha ancora un'app per la logica, vedere l'articolo su [come creare app per la logica](../logic-apps/quickstart-create-first-logic-app-workflow.md). 
  Aggiungere il trigger e le azioni da usare per la gestione dei metadati dell'elemento. In alternativa, per fare semplicemente una prova, aggiungere all'app per la logica un trigger, ad esempio **di richiesta** oppure **HTTP**.

## <a name="add-metadata-to-artifacts"></a>Aggiungere metadati agli elementi

1. Accedere al <a href="https://portal.azure.com" target="_blank">portale di Azure</a> con le credenziali dell'account Azure. Trovare e aprire l'account di integrazione.

1. Selezionare l'elemento in cui si vogliono aggiungere i metadati e scegliere **Modifica**. Immettere i dettagli dei metadati per l'elemento, ad esempio:

   ![Immettere i metadati](media/logic-apps-enterprise-integration-metadata/add-partner-metadata.png)

1. Al termine dell'operazione, scegliere **OK**.

1. Per visualizzare i metadati nella definizione JSON (JavaScript Object Notation) per l'account di integrazione, scegliere **Modifica come JSON** in modo che si apra l'editor JSON: 

   ![JSON per i metadati del partner](media/logic-apps-enterprise-integration-metadata/partner-metadata.png)

## <a name="get-artifact-metadata"></a>Ottenere i metadati degli elementi

1. Nel portale di Azure aprire l'app per la logica collegata all'account di integrazione desiderato. 

1. Se si aggiunge il passaggio per il recupero dei metadati sotto il trigger o l'ultima azione nel flusso di lavoro, in Progettazione App per la logica scegliere **Nuovo passaggio** > **Aggiungi un'azione**. 

1. Nella casella di ricerca, digitare "account di integrazione". Nella casella di ricerca scegliere **Tutti**. Nell'elenco di azioni selezionare questa azione: **Ricerca elemento dell'account di integrazione - Account di integrazione**

   ![Selezionare "Ricerca elemento dell'account di integrazione"](media/logic-apps-enterprise-integration-metadata/integration-account-artifact-lookup.png)

1. Specificare queste informazioni per l'elemento da trovare:

   | Proprietà | Obbligatorio | Value | Descrizione | 
   |----------|---------|-------|-------------| 
   | **Tipo di elemento** | Yes | **Schema**, **Mappa**, **Partner**, **Contratto** o un tipo personalizzato | Tipo dell'elemento desiderato | 
   | **Nome elemento** | Yes | <*artifact-name*> | Nome dell'elemento desiderato | 
   ||| 

   Ad esempio, si supponga di voler ottenere i metadati per un elemento partner commerciale:

   ![Selezionare il tipo di elemento e specificare un nome elemento](media/logic-apps-enterprise-integration-metadata/artifact-lookup-information.png)

1. Aggiungere l'azione desiderata per la gestione dei metadati, ad esempio:

   1. Sotto l'azione **Ricerca elemento dell'account di integrazione** scegliere **Passaggio successivo**e selezionare **Aggiungi un'azione**. 

   1. Nella casella di ricerca immettere "http". Sotto la casella di ricerca scegliere **Predefiniti**e selezionare questa azione: **HTTP - HTTP**

      ![Aggiungere azione HTTP](media/logic-apps-enterprise-integration-metadata/http-action.png)

   1. Fornire informazioni per i metadati dell'elemento da gestire. 

      Ad esempio, si supponga di voler ottenere i metadati `routingUrl` aggiunti in precedenza in questo argomento. Ecco i valori delle proprietà che è possibile specificare: 

      | Proprietà | Obbligatorio | Value | Descrizione | 
      |----------|----------|-------|-------------| 
      | **Metodo** | Yes | <*operation-to-run*> | Operazione HTTP da eseguire sull'elemento. Ad esempio, questa azione HTTP usa il metodo **GET**. | 
      | **URI** | Yes | <*metadata-location*> | Per accedere al valore dei metadati `routingUrl` dall'elemento recuperato si può usare un'espressione, ad esempio: <p>`@{outputs('Integration_Account_Artifact_Lookup')['properties']['metadata']['routingUrl']}` | 
      | **Intestazioni** | No | <*header-values*> | Qualsiasi intestazione di output del trigger da passare nell'azione HTTP. Ad esempio, per passare il valore della proprietà `headers` del trigger è possibile usare un'espressione, ad esempio: <p>`@triggeroutputs()['headers']` | 
      | **Corpo** | No | <*body-content*> | Qualsiasi altro contenuto che si vuole passare tramite la proprietà `body` dell'azione HTTP. Questo esempio passa i valori `properties` dell'elemento nell'azione HTTP: <p>1. Fare clic all'interno della proprietà **Corpo** in modo che venga visualizzato l'elenco di contenuto dinamico. Se non compaiono proprietà, scegliere **Altre informazioni**. <br>2. Nell'elenco del contenuto dinamico, sotto **Ricerca elemento dell'account di integrazione** selezionare **Proprietà**. | 
      |||| 

      Ad esempio:

      ![Specificare i valori e le espressioni per un'azione HTTP](media/logic-apps-enterprise-integration-metadata/add-http-action-values.png)

   1. Per controllare le informazioni fornite per l'azione HTTP, visualizzare la definizione JSON dell'app per la logica. Sulla barra degli strumenti di Progettazione App per la logica scegliere **Visualizzazione Codice** in modo che compaia la definizione JSON dell'app, ad esempio:

      ![Definizione JSON dell'app per la logica](media/logic-apps-enterprise-integration-metadata/finished-logic-app-definition.png)

      Tornando a Progettazione app per la logica, le espressioni usate vengono ora visualizzate come risolte, ad esempio:

      ![Espressioni risolte in Progettazione app per la logica](media/logic-apps-enterprise-integration-metadata/resolved-expressions.png)

## <a name="next-steps"></a>Passaggi successivi

* [Altre informazioni sui contratti](logic-apps-enterprise-integration-agreements.md)
