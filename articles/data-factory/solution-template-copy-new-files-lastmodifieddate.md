---
title: Copia i file nuovi e modificati da LastModifiedDate con Azure Data Factory | Microsoft Docs
description: Informazioni su come usare un modello di soluzione per copiare file nuovi e modificati da LastModifiedDate con Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dearandyxu
ms.author: yexu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 3/8/2019
ms.openlocfilehash: a2a8f0478d1ae4fb19cb911b02572145ff59839b
ms.sourcegitcommit: 11265f4ff9f8e727a0cbf2af20a8057f5923ccda
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/08/2019
ms.locfileid: "72030066"
---
# <a name="copy-new-and-changed-files-by-lastmodifieddate-with-azure-data-factory"></a>Copia i file nuovi e modificati da LastModifiedDate con Azure Data Factory

Questo articolo descrive un modello di soluzione che è possibile usare per copiare i file nuovi e modificati solo da LastModifiedDate da un archivio basato su file in un archivio di destinazione. 

## <a name="about-this-solution-template"></a>Informazioni sul modello di soluzione

Questo modello seleziona innanzitutto i file nuovi e modificati solo in base ai relativi attributi **LastModifiedDate**, quindi copia i file selezionati dall'archivio dell'origine dati all'archivio di destinazione dati.

Il modello contiene un'attività:
- **Copiare** per copiare i file nuovi e modificati solo da LastModifiedDate da un archivio file a un archivio di destinazione.

Il modello definisce quattro parametri:
-  *FolderPath_Source* è il percorso della cartella in cui è possibile leggere i file dall'archivio di origine. È necessario sostituire il valore predefinito con il percorso della cartella.
-  *FolderPath_Destination* è il percorso della cartella in cui si desidera copiare i file nell'archivio di destinazione. È necessario sostituire il valore predefinito con il percorso della cartella.
-  *LastModified_From* viene usato per selezionare i file il cui attributo LastModifiedDate è successivo o uguale a questo valore DateTime.  Per selezionare solo i nuovi file, che non sono stati copiati per la prima volta, questo valore DateTime può essere l'ora in cui è stata attivata la pipeline per l'ultima volta. È possibile sostituire il valore predefinito ' 2019-02-01T00:00:00Z ' con la LastModifiedDate prevista nel fuso orario UTC. 
-  *LastModified_To* viene usato per selezionare i file il cui attributo LastModifiedDate è precedente a questo valore DateTime. Per selezionare solo i nuovi file, che non sono stati copiati l'ultima volta, questo valore DateTime può essere l'ora corrente.  È possibile sostituire il valore predefinito ' 2019-02-01T00:00:00Z ' con la LastModifiedDate prevista nel fuso orario UTC. 

## <a name="how-to-use-this-solution-template"></a>Come usare questo modello di soluzione

1. Passare a modello **Copia nuovi file solo per LastModifiedDate**. Creare una **nuova** connessione all'archivio di archiviazione di origine. L'archivio di archiviazione di origine è il punto da cui si desidera copiare i file.

    ![Creare una nuova connessione all'origine](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate1.png)
    
2. Selezionare prima di tutto il **tipo**di archiviazione. Successivamente, immettere il **nome dell'account** di archiviazione e la **chiave dell'account**. Infine, selezionare **fine**.

    ![Immettere una stringa di connessione](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate2.png)
    
3. Consente di creare una **nuova** connessione all'archivio di destinazione. L'archivio di destinazione è il punto in cui si desidera copiare i file. È anche necessario immettere le informazioni di connessione dell'archivio di destinazione dati Analogamente al passaggio 2.

    ![Creare una nuova connessione alla destinazione](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate3.png)

4. Selezionare **Usa questo modello**.

    ![Usa questo modello](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate4.png)
    
5. Nel pannello verrà visualizzata la pipeline disponibile, come illustrato nell'esempio seguente:

    ![Mostra la pipeline](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate5.png)

6. Selezionare **debug**, scrivere il valore per i **parametri** e fare clic su **fine**.  Nell'immagine seguente vengono impostati i parametri come segue.
   - **FolderPath_Source** =  **/source/**
   - **FolderPath_Destination** =  **/destination/**
   - **LastModified_From** =  **2019-02-01T00:00:00Z**
   - **LastModified_To** = **2019-03-01T00:00:00Z**
    
     L'esempio indica che i file che sono stati modificati l'ultima volta nell'intervallo di tempo compreso tra *2019-02-01T00:00:00Z* e *2019-03-01T00:00:00Z* verranno copiati da una cartella */source/* a una cartella */Destination/* .  È possibile sostituirli con parametri personalizzati.
    
     ![Eseguire la pipeline](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate6.png)

7. Esaminare il risultato. Nell'archivio di destinazione vengono copiati solo i file modificati per ultimi nell'intervallo di tempo configurato.

    ![Esaminare il risultato](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate7.png)
    
8. A questo punto è possibile aggiungere un trigger Windows a cascata per automatizzare questa pipeline, in modo che la pipeline possa sempre copiare i file nuovi e modificati solo da LastModifiedDate periodicamente.  Selezionare **Aggiungi trigger**e selezionare **nuovo/modifica**.

    ![Esaminare il risultato](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate8.png)
    
9. Nella finestra **Add Triggers** (Aggiungi trigger) selezionare **+ Nuovo**.

    ![Esaminare il risultato](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate9.png)

10. Selezionare la **finestra a cascata** per il tipo di trigger, impostare **ogni 15 minuti** come ricorrenza (è possibile passare a qualsiasi intervallo di tempo), quindi selezionare **Avanti**.

    ![Crea trigger](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate10.png)    
    
11. Scrivere il valore per i **parametri di esecuzione del trigger** come segue, quindi fare clic su **fine**.
    - **FolderPath_Source** =  **/source/** .  È possibile sostituire con la cartella nell'archivio dati di origine.
    - **FolderPath_Destination** =  **/Destination/** .  È possibile sostituire con la cartella nell'archivio dati di destinazione.
    - **LastModified_From** =   **\@trigger (). Outputs. windowStartTime**.  Si tratta di una variabile di sistema del trigger che determina l'ora in cui la pipeline è stata attivata l'ultima volta.
    - **LastModified_To** =  **\@trigger (). Outputs. windowEndTime**.  Si tratta di una variabile di sistema del trigger che determina l'ora in cui la pipeline viene attivata questa volta.
    
    ![Parametri di input](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate11.png)
    
12. Selezionare **Pubblica tutti**.
    
    ![Pubblica tutti](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate12.png)

13. Creare nuovi file nella cartella di origine dell'archivio dell'origine dati.  Si è ora in attesa che la pipeline venga attivata automaticamente e solo i nuovi file verranno copiati nell'archivio di destinazione.

14. Selezionare la scheda **monitoraggio** nel riquadro di spostamento a sinistra e attendere circa 15 minuti se la ricorrenza del trigger è stata impostata su ogni 15 minuti. 

    ![Seleziona monitoraggio](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate14.png)

15. Esaminare il risultato. Si noterà che la pipeline verrà attivata automaticamente ogni 15 minuti e solo i file nuovi o modificati dall'archivio di origine verranno copiati nell'archivio di destinazione in ogni esecuzione della pipeline.

    ![Esaminare il risultato](media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate15.png)
    
## <a name="next-steps"></a>Passaggi successivi

- [Introduzione al servizio Azure Data Factory](introduction.md)
