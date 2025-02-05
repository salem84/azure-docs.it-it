---
title: 'Partition and Sample: Riferimento al modulo'
titleSuffix: Azure Machine Learning service
description: Informazioni su come usare il modulo Partition and Sample nel servizio Azure Machine Learning per eseguire il campionamento in un set di dati o per creare partizioni dal set di dati.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ms.openlocfilehash: 79cd6fe9156a785d82e303007d02ce58506dcfcf
ms.sourcegitcommit: 07700392dd52071f31f0571ec847925e467d6795
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70128541"
---
# <a name="partition-and-sample-module"></a>Modulo Partition and Sample

Questo articolo descrive un modulo dell'interfaccia visiva (anteprima) per il servizio Azure Machine Learning.

Usare questo modulo per eseguire il campionamento in un set di dati o per creare partizioni dal set di dati.

Il campionamento è uno strumento importante in Machine Learning perché consente di ridurre le dimensioni di un set di dati mantenendo lo stesso rapporto di valori. Questo modulo supporta diverse attività correlate che sono importanti per Machine Learning: 

- Suddivisione dei dati in più sottosezioni delle stesse dimensioni. 

    È possibile utilizzare le partizioni per la convalida incrociata o per assegnare i case a gruppi casuali.

- Separare i dati in gruppi e quindi usare i dati di un gruppo specifico. 

    Dopo aver assegnato in modo casuale i case a gruppi diversi, potrebbe essere necessario modificare le funzionalità associate a un solo gruppo.

- Campionamento. 

    È possibile estrarre una percentuale dei dati, applicare il campionamento casuale o scegliere una colonna da usare per il bilanciamento del set di dati ed eseguire il campionamento stratificato sui rispettivi valori.

- Creazione di un set di dati più piccolo per il test. 

    Se si dispone di una grande quantità di dati, è possibile utilizzare solo le prime *n* righe durante la configurazione dell'esperimento e quindi passare a utilizzando il set di dati completo quando si compila il modello. È anche possibile usare il campionamento per creare un set di dati più piccolo da usare in fase di sviluppo.

## <a name="configure-partition-and-sample"></a>Configurare Partition ed Sample

Questo modulo supporta più metodi per dividere i dati in partizioni o per il campionamento. Prima scegliere il metodo, quindi impostare le opzioni aggiuntive richieste dal metodo.

- Head
- campionamento
- Assegna a riduzioni
- Selezione

### <a name="get-top-n-rows-from-a-dataset"></a>Ottenere le prime N righe da un set di dati

Usare questa modalità per ottenere solo le prime *n* righe. Questa opzione è utile se si desidera testare un esperimento su un numero ridotto di righe e non è necessario che i dati siano bilanciati o campionati in alcun modo.

1. Aggiungere il modulo **Partition and Sample** all'esperimento nell'interfaccia e connettere il set di dati.  

2. **Partizione o modalità di esempio**: Impostare questa opzione su **Head**.

3. **Numero di righe da selezionare**: Digitare il numero di righe da restituire.

    Il numero di righe specificato deve essere un numero intero non negativo. Se il numero di righe selezionate è maggiore del numero di righe nel set di dati, viene restituito l'intero set di dati.

4. Eseguire l'esperimento.

Il modulo restituisce un singolo set di dati contenente solo il numero di righe specificato. Le righe vengono lette sempre dalla parte superiore del set di dati.

### <a name="create-a-sample-of-data"></a>Creazione di un campione di dati

Questa opzione supporta il campionamento casuale semplice o il campionamento casuale stratificato. Questa opzione è utile se si desidera creare un set di dati di esempio rappresentativo più piccolo per i test.

1. Aggiungere il modulo **Partition and Sample** all'esperimento e connettere il set di dati.

2. **Partizione o modalità di esempio**: Impostare questa impostazionesu campionamento.

3. **Frequenza di campionamento**: Digitare un valore compreso tra 0 e 1. Questo valore specifica la percentuale di righe del set di dati di origine che deve essere inclusa nel set di dati di output.

    Se, ad esempio, si desidera solo metà del set di dati originale `0.5` , digitare per indicare che la frequenza di campionamento deve essere pari al 50%.

    Le righe del set di dati di input vengono mescolate e inserite selettivamente nel set di dati di output, in base al rapporto specificato.

4. Valore **di inizializzazione casuale per**il campionamento: Facoltativamente, digitare un numero intero da utilizzare come valore di inizializzazione.

    Questa opzione è importante se si desidera che le righe vengano divise allo stesso modo ogni volta. Il valore predefinito è 0, vale a dire che un valore di inizializzazione iniziale viene generato in base al clock di sistema. Questo può causare risultati leggermente diversi ogni volta che si esegue l'esperimento.

5. **Suddivisione stratificata per**il campionamento: Selezionare questa opzione se è importante che le righe del set di dati vengano divise in modo uniforme da una colonna chiave prima del campionamento.

    Per **colonna chiave**di stratificazione per il campionamento, selezionare una singola *colonna di strati* da usare quando si divide il set di dati. Le righe nel set di dati vengono quindi divise come segue:

    1. Tutte le righe di input vengono raggruppate (stratificate) in base ai valori nella colonna di strati specificata.

    2. Le righe vengono mescolate all'interno di ogni gruppo.

    3. Ogni gruppo viene aggiunto in modo selettivo al set di dati di output per soddisfare il rapporto specificato.


6. Eseguire l'esperimento.

    Con questa opzione, il modulo restituisce un singolo set di dati che contiene un campionamento dei dati rappresentativo. La parte rimanente non campionata del set di dati non è output. 

## <a name="split-data-into-partitions"></a>Suddividere i dati in partizioni

Utilizzare questa opzione quando si desidera suddividere il set di dati in subset di dati. Questa opzione è utile anche quando si desidera creare un numero personalizzato di riduzioni per la convalida incrociata oppure suddividere le righe in più gruppi.

1. Aggiungere il modulo **Partition and Sample** all'esperimento e connettere il set di dati.

2. Per la **modalità partizione o di esempio**, selezionare **assegna a riduzioni**.

3. **Usare la sostituzione nel partizionamento**: Selezionare questa opzione se si desidera che la riga campionata venga rimessa nel pool di righe per il possibile riutilizzo. Di conseguenza, la stessa riga potrebbe essere assegnata a diverse riduzioni.

    Se non si utilizza la sostituzione (opzione predefinita), la riga campionata non viene rimessa nel pool di righe per il possibile riutilizzo. Di conseguenza, ogni riga può essere assegnata a una sola riduzioni.

4. **Suddivisione casuale**:  Selezionare questa opzione se si desidera che le righe vengano assegnate in modo casuale alle riduzioni.

    Se non si seleziona questa opzione, le righe vengono assegnate a riduzioni utilizzando il metodo Round Robin.

5. Valore di **inizializzazione casuale**: Facoltativamente, digitare un numero intero da utilizzare come valore di inizializzazione. Questa opzione è importante se si desidera che le righe vengano divise allo stesso modo ogni volta. In caso contrario, il valore predefinito 0 indica che verrà utilizzato un valore di inizializzazione iniziale casuale.

6. **Specificare il metodo Partitioner**: Indicare come si desidera che i dati vengano ripartiti a ogni partizione utilizzando le opzioni seguenti:

    - **Partizionamento uniforme**: Usare questa opzione per inserire un numero uguale di righe in ogni partizione. Per specificare il numero di partizioni di output, digitare un numero intero nella casella di testo **specificare il numero di riduzioni da dividere** in modo uniforme.

    - **Partizione con proporzioni personalizzate**: Usare questa opzione per specificare le dimensioni di ogni partizione come un elenco delimitato da virgole.

        Se, ad esempio, si desidera creare tre partizioni, con la prima partizione che contiene il 50% dei dati e le due partizioni rimanenti ognuna contenente il 25% dei dati, fare clic sull' **elenco di proporzioni separate da virgola** e digitare i numeri seguenti:`.5, .25, .25`

        La somma di tutte le dimensioni della partizione deve essere sommata esattamente a 1.

        - Se si immettono numeri che si aggiungono a un valore **minore di 1**, viene creata una partizione aggiuntiva che conterrà le righe rimanenti. Se, ad esempio, si digitano i valori 0,2 e,3, viene creata una terza partizione che include il 50% di tutte le righe rimanenti.

        - Se si immettono numeri che si aggiungono a **più di 1**, viene generato un errore quando si esegue l'esperimento.

7. **Suddivisione**stratificata: Selezionare questa opzione se si desidera che le righe vengano stratificate quando si divide, quindi scegliere la _colonna strati_.

8. Eseguire l'esperimento.

    Con questa opzione, il modulo restituisce più set di impostazioni partizionati usando le regole specificate.

### <a name="use-data-from-a-predefined-partition"></a>Usare i dati di una partizione predefinita  

Questa opzione viene usata quando si divide un set di dati in più partizioni e ora si vuole caricare ogni partizione a sua volta per un'ulteriore analisi o elaborazione.

1. Aggiungere il modulo **Partition and Sample** all'esperimento.

2. Connetterlo all'output di un'istanza precedente di **Partition ed Sample**. Per generare un certo numero di partizioni, è necessario che nell'istanza sia stata utilizzata l'opzione **assegna a riduzioni** .

3. **Partizione o modalità di esempio**: Selezionare **pick fold**.

4. **Consente di specificare**la sezione da campionare: Selezionare una partizione da utilizzare digitando il relativo indice. Gli indici di partizione sono basati su 1. Se, ad esempio, il set di dati è stato diviso in tre parti, le partizioni avrebbero gli indici 1, 2 e 3.

    Se si digita un valore di indice non valido, viene generato un errore in fase di progettazione: "Errore 0018: Il set di dati contiene dati non validi. "

    Oltre a raggruppare il set di dati in base alle riduzioni, è possibile separare il set di dati in due gruppi: una sezione di destinazione e tutto il resto. A tale scopo, digitare l'indice di una singola sezione, quindi selezionare l'opzione seleziona **complemento della sezione selezionata**per ottenere tutto tranne i dati nella sezione specificata.

5. Se si utilizzano più partizioni, è necessario aggiungere altre istanze della **partizione e** del modulo di esempio per gestire ogni partizione.

    Si immagini, ad esempio, i pazienti partizionati in precedenza in quattro riduzioni utilizzando Age. Per utilizzare ogni singola sezione, è necessario disporre di quattro copie del modulo **Partition and Sample** e, in ognuno di essi, selezionare una sezione diversa, come illustrato di seguito. Non è corretto usare l' **assegnazione per** ripiegare l'output direttamente.  

    [![Partition ed Sample](./media/partition-and-sample/partition-and-sample.png)](./media/partition-and-sample/partition-and-sample-lg.png#lightbox)

5. Eseguire l'esperimento.

    Con questa opzione, il modulo restituisce un singolo set di dati contenente solo le righe assegnate a tale riduzioni.

> [!NOTE]
>  Non è possibile visualizzare direttamente le designazioni di riduzioni; sono presenti solo nei metadati.

## <a name="next-steps"></a>Passaggi successivi

Vedere il [set di moduli disponibili](module-reference.md) per Azure Machine Learning servizio. 