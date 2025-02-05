---
title: Server SQL locale
titleSuffix: Azure Machine Learning Studio
description: Usare dati provenienti da un database SQL Server locale per eseguire analisi avanzate con Azure Machine Learning Studio.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: xiaoharper
ms.author: amlstudiodocs
ms.custom: seodec18
ms.date: 03/13/2017
ms.openlocfilehash: 9590728cec663b36c889dc26a6216c3d474244e4
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60735470"
---
# <a name="perform-analytics-with-azure-machine-learning-studio-using-an-on-premises-sql-server-database"></a>Eseguire analisi con Azure Machine Learning Studio usando un database SQL Server locale

In molti casi, le aziende che si avvalgono di dati locali vogliono sfruttare la scalabilità e l'agilità del cloud per i propri carichi di lavoro di Machine Learning. Non intendono tuttavia abbandonare i carichi di lavoro e i processi aziendali correnti spostando nel cloud tutti i propri dati locali. Azure Machine Learning Studio supporta ora la lettura dei dati da un database di SQL Server in locale e quindi eseguire il training e valutazione di un modello con questi dati. Non è più necessario copiare e sincronizzare manualmente i dati tra il cloud e il server locale. Il modulo **Import Data** (Importazione dati) di Azure Machine Learning Studio, infatti, è in grado di leggere i dati direttamente dal server SQL Server locale per i processi di formazione e assegnazione dei punteggi.

Questo articolo fornisce una panoramica di come inserire dati locali di SQL Server in Azure Machine Learning Studio. Si presuppone che abbia familiarità con i concetti di Studio, come le aree di lavoro, moduli, set di dati, esperimenti *via*.

> [!NOTE]
> Questa funzionalità non è disponibile per le aree di lavoro gratuite. Per altre informazioni sui prezzi e sui piani tariffari di Machine Learning, vedere [Azure Machine Learning Pricing](https://azure.microsoft.com/pricing/details/machine-learning/)(Prezzi di Azure Machine Learning).
>
>

<!-- -->



## <a name="install-the-data-factory-self-hosted-integration-runtime"></a>Installare il runtime di integrazione self-hosted di Data Factory
Per accedere a un database di SQL Server in locale in Azure Machine Learning Studio, è necessario scaricare e installare il Data Factory Self-Hosted Integration Runtime, precedentemente noto come Gateway di gestione dati. Quando si configura la connessione in Machine Learning Studio, è possibile scaricare e installare il runtime di integrazione (IR) usando la finestra di dialogo **Scarica e registra il gateway dati** descritta di seguito.


È anche possibile installare il runtime di integrazione in anticipo scaricando ed eseguendo il pacchetto di installazione MSI dal [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=39717). Il file MSI è anche utilizzabile per eseguire l'aggiornamento di un runtime di integrazione esistente alla versione più recente conservando tutte le impostazioni.

Il runtime di integrazione self-hosted di Data Factory prevede i prerequisiti seguenti:

* L'integrazione self-hosted di Data Factory richiede un sistema operativo a 64 bit con .NET Framework 4.6.1 o versioni successive.
* Sono supportati i sistemi operativi Windows 10, Windows 2012, Windows Server 2012, Windows Server 2012 R2, Windows Server 2016. 
* La configurazione consigliata per la machine IR è di almeno 2 GHz, CPU 4 core, 8 GB di RAM e un disco da 80 GB.
* Se il computer host entra in stato di ibernazione, il runtime di integrazione non risponde alle richieste di dati. Pertanto, configurare una combinazione per il risparmio di energia appropriata nel computer prima di installare il runtime di integrazione. L'installazione del runtime di integrazione visualizza un messaggio se il computer è configurato per l'ibernazione.
* Poiché le attività di copia seguono una frequenza specifica, l'utilizzo delle risorse nel computer (CPU e memoria) segue lo stesso ciclo costituito da periodi di massimo utilizzo alternati a periodi di inattività. L'utilizzo delle risorse dipende molto anche dalla quantità di dati da spostare. Quando sono in corso più processi di copia, l'utilizzo delle risorse aumenta durante i periodi di picco. Sebbene la configurazione minima sopra elencata sia tecnicamente sufficiente, a seconda del carico previsto per lo spostamento dei dati è possibile adottare una configurazione con più risorse rispetto alla configurazione minima.

Tenere presente quanto segue durante l'installazione e l'uso di un runtime di integrazione self-hosted di Data Factory:

* In un computer può essere installata una sola istanza di IR.
* È possibile usare un singolo runtime di integrazione per più origini dati locali.
* Alla stessa origine dati locale è possibile collegare più runtime di integrazione su computer diversi.
* Configurare un runtime di integrazione per la sola area di lavoro alla volta. Attualmente i runtime di integrazione non possono essere condivisi tra più aree di lavoro.
* È possibile configurare più runtime di integrazione per una singola area di lavoro. Ad esempio, è possibile usare un runtime di integrazione che è connesso alle origini dati di test durante lo sviluppo e un runtime di integrazione di produzione, quando si è pronti per l'operazionalizzazione.
* Il runtime di integrazione non deve trovarsi sullo stesso computer dell'origine dati. Tuttavia, se i gateway sono posizionati in prossimità dell'origine dati, il tempo necessario alla connessione del gateway all'origine dati si riduce. Si consiglia di installare il runtime di integrazione in un computer diverso da quello che ospita l'origine dati locale in modo che il gateway non si contenda le risorse con l'origine dati.
* Se si dispone già di un runtime di integrazione installato nel computer agente gli scenari di Power BI o Azure Data Factory, installare un runtime di integrazione separato per Azure Machine Learning Studio in un altro computer.

  > [!NOTE]
  > Non è possibile eseguire runtime di integrazione self-hosted di Data Factory e Power BI Gateway nello stesso computer.
  >
  >
* È necessario usare il Runtime di integrazione Self-Hosted di Data Factory di Azure Machine Learning Studio, anche se si usa Azure ExpressRoute per altri dati. Considerare l'origine dati come origine dati locale, ovvero protetta da firewall, anche quando si usa ExpressRoute. Usare il runtime di integrazione self-hosted Data Factory per stabilire la connettività tra Machine Learning e l'origine dati.

Informazioni dettagliate sui prerequisiti di installazione e sulla procedura di installazione, oltre a suggerimenti sulla risoluzione dei problemi, sono disponibili nell'articolo [Runtime di integrazione in Data Factory](../../data-factory/concepts-integration-runtime.md).

## <a name="span-idusing-the-data-gateway-step-by-step-walk-classanchorspan-idtoc450838866-classanchorspanspaningress-data-from-your-on-premises-sql-server-database-into-azure-machine-learning"></a><span id="using-the-data-gateway-step-by-step-walk" class="anchor"><span id="_Toc450838866" class="anchor"></span></span>Inserire dati del database SQL Server locale in Azure Machine Learning
In questa procedura dettagliata, si verrà impostato un Runtime di integrazione di Azure Data Factory in un'area di lavoro di Azure Machine Learning, configurarlo e quindi leggere i dati da un database di SQL Server in locale.

> [!TIP]
> Prima di iniziare, disabilitare il blocco popup del browser per `studio.azureml.net`. Se si usa il browser Google Chrome, scaricare e installare uno dei diversi plug-in disponibili nella sezione dell' [estensione per app ClickOnce](https://chrome.google.com/webstore/search/clickonce?_category=extensions)in Google Chrome WebStore.
>
> [!NOTE]
> Il runtime di integrazione self-hosted Data Factory di Azure era conosciuto in precedenza come Gateway di gestione dati. L'esercitazione dettagliata continuerà a indicarlo come gateaway.  

### <a name="step-1-create-a-gateway"></a>Passaggio 1: Creare un gateway
Il primo passaggio consiste nel creare e configurare il gateway per accedere al database SQL locale.

1. Accedere ad [Azure Machine Learning Studio](https://studio.azureml.net/Home/) e selezionare l'area di lavoro in cui si vuole lavorare.
2. Fare clic sul pannello **SETTINGS** (IMPOSTAZIONI) a sinistra e quindi sulla scheda **DATA GATEWAYS** (GATEWAY DATI) in alto.
3. Fare clic su **NEW DATA GATEWAY** (NUOVO GATEWAY DATI) nella parte inferiore della schermata.

    ![Nuovo gateway dati](./media/use-data-from-an-on-premises-sql-server/new-data-gateway-button.png)
4. Nella finestra di dialogo **New data gateway** (Nuovo gateway dati) compilare il campo **Gateway Name** (Nome gateway) e, facoltativamente, il campo **Description** (Descrizione). Fare clic sulla freccia nell'angolo inferiore destro per passare al passaggio successivo della configurazione.

    ![Immettere un nome e una descrizione per il gateway](./media/use-data-from-an-on-premises-sql-server/new-data-gateway-dialog-enter-name.png)
5. Nella finestra di dialogo Download and register data gateway (Scarica e registra il gateway dati) copiare negli Appunti il valore del campo GATEWAY REGISTRATION KEY (CHIAVE REGISTRAZIONE GATEWAY).

    ![Scaricare e registrare il gateway dati](./media/use-data-from-an-on-premises-sql-server/download-and-register-data-gateway.png)
6. <span id="note-1" class="anchor"></span>Se Gateway di gestione dati di Microsoft non è ancora stato scaricato e installato, fare clic su **Download data management gateway**(Scarica Gateway di gestione dati). Si verrà reindirizzati all'Area download Microsoft in cui sarà possibile selezionare la versione del gateway necessaria, in modo da poterla scaricare e installare. Informazioni dettagliate sui prerequisiti di installazione e sulla procedura di installazione, oltre a suggerimenti sulla risoluzione dei problemi, sono disponibili nelle sezioni iniziali dell'articolo [Spostare dati tra origini locali e il cloud con Gateway di gestione dati](../../data-factory/tutorial-hybrid-copy-portal.md).
7. Dopo aver installato il gateway, verrà aperto Gestione configurazione di Gateway di gestione dati e verrà visualizzata la finestra di dialogo **Registra gateway** . Incollare la **chiave di registrazione del gateway** copiata negli Appunti e fare clic su **Registra**.
8. Se si dispone già di un gateway installato, eseguire Gestione configurazione di Gateway di gestione dati. Fare clic su **Cambia chiave**, incollare la **chiave di registrazione del gateway** copiata negli Appunti nel passaggio precedente e fare clic su **OK**.
9. Al termine dell'installazione verrà visualizzata la finestra di dialogo **Registra gateway** di Gestione configurazione di Gateway di gestione dati. Incollare la CHIAVE DI REGISTRAZIONE DEL GATEWAY copiata negli Appunti e fare clic su **Registra**.

    ![Registrare il gateway](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-register-gateway.png)
10. La configurazione del gateway è completa quando nella scheda **Home** di Gestione configurazione di Gateway di gestione dati i valori seguenti sono impostati secondo quanto descritto di seguito:

    * **Nome gateway** e **Nome istanza** sono impostati sul nome del gateway.
    * **Registrazione** è impostato su **Registrato**.
    * **Stato** è impostato su **Avviato**.
    * La barra di stato in fondo visualizza **Connesso al servizio cloud Gateway di gestione dati** insieme a un segno di spunta verde.

      ![Gestione del gateway di gestione dati](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-registered.png)

      Al termine della registrazione viene aggiornato anche Azure Machine Learning Studio.

    ![Registrazione del gateway completata](./media/use-data-from-an-on-premises-sql-server/gateway-registered.png)
11. Nella finestra di dialogo **Download and register data gateway** (Scarica e registra gateway dati) fare clic sul segno di spunta per completare l'installazione. Nella pagina **Settings** (Impostazioni) lo stato del gateway risulta impostato su "Online". Nel riquadro di destra sono disponibili informazioni sullo stato e altre informazioni utili.

    ![Impostazioni del gateway](./media/use-data-from-an-on-premises-sql-server/gateway-status.png)
12. In Gestione configurazione di Gateway di gestione dati passare alla scheda **Certificato** . Il certificato specificato in questa scheda viene usato per crittografare e decrittografare le credenziali per l'archivio dati locale specificato nel portale. Questo certificato è quello predefinito. Si consiglia di sostituirlo con il certificato personale di cui è stato eseguito il backup nel sistema di gestione dei certificati. Fare clic su **Modifica** per usare il proprio certificato.

    ![Cambiare il certificato del gateway](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-certificate.png)
13. (Facoltativo) Se si vuole abilitare la registrazione dettagliata per la risoluzione dei problemi del gateway, in Gestione configurazione di Gateway di gestione dati passare alla scheda **Diagnostica** e selezionare l'opzione **Abilita la registrazione dettagliata per la risoluzione dei problemi**. Le informazioni sulla registrazione si trovano nel Visualizzatore eventi di Windows, nel nodo **Registri applicazioni e servizi** -&gt; **Gateway di gestione dati**. È possibile usare la scheda **Diagnostica** anche per testare la connessione a un'origine dati locale usando il gateway.

    ![Abilitare la registrazione dettagliata](./media/use-data-from-an-on-premises-sql-server/data-gateway-configuration-manager-verbose-logging.png)

In questo passaggio si completa il processo di installazione del gateway in Azure Machine Learning Studio.
ed è quindi possibile iniziare a usare i dati locali.

In Studio è possibile creare e configurare più gateway per ogni area di lavoro. Può essere utile, ad esempio, creare un gateway da connettere alle origini dati di test in fase di sviluppo e un gateway per le origini dati di produzione. Azure Machine Learning Studio offre la flessibilità necessaria per configurare più gateway, a seconda dell'ambiente aziendale. Attualmente, tuttavia, non è possibile condividere un gateway tra più aree di lavoro e in un computer è possibile installare un solo gateway. Per altre informazioni, vedere [Spostare dati tra origini locali e il cloud con Gateway di gestione dati](../../data-factory/tutorial-hybrid-copy-portal.md).

### <a name="step-2-use-the-gateway-to-read-data-from-an-on-premises-data-source"></a>Passaggio 2: Usare il gateway per leggere dati da un'origine dati locale
Dopo aver configurato il gateway è possibile aggiungere un modulo **Import Data** (Importa dati) a un esperimento in cui si inseriscono i dati letti dal database SQL Server locale.

1. In Machine Learning Studio selezionare la scheda **EXPERIMENTS** (ESPERIMENTI), fare clic su **+NEW** (+NUOVO) nell'angolo inferiore sinistro e selezionare **Blank Experiment** (Esperimento vuoto) oppure selezionare uno degli esperimenti di esempio disponibili.
2. Trovare il modulo **Import data** (Importa dati) e trascinarlo nell'area di disegno dell'esperimento.
3. Fare clic su **Save as** (Salva con nome) sotto l'area di disegno. Immettere "Azure Machine Learning Studio On-Premises SQL Server Tutorial" come nome dell'esperimento, selezionare l'area di lavoro e scegliere il **OK** segno di spunta.

   ![Salvare l'esperimento con un nuovo nome](./media/use-data-from-an-on-premises-sql-server/experiment-save-as.png)
4. Fare clic sul modulo **Import Data** (Importazione dati) per selezionarlo, quindi sul pannello **Properties** (Proprietà) a destra dell'area di disegno e selezionare "On-Premises SQL Database" (Database SQL locale) dall'elenco a discesa **Data source** (Origine dati).
5. Selezionare il **gateway dati** precedentemente installato e registrato. È possibile configurare un altro gateway selezionando l'opzione che consente di aggiungere un nuovo gateway dati.

   ![Selezionare il gateway dati per il modulo Import Data](./media/use-data-from-an-on-premises-sql-server/import-data-select-on-premises-data-source.png)
6. Compilare i campi **Database server name** (Nome server database) e **Database name** (Nome database) insieme a **Database query** (Query database) e specificare la query di database SQL che si vuole eseguire.
7. Fare clic su **Enter values** (Immetti valori) in **User name and password** (Nome utente e password) e specificare le credenziali del database. È possibile usare Autenticazione integrata di Windows o Autenticazione di SQL Server, in base al tipo di configurazione del database SQL Server locale.

   ![Immettere le credenziali del database](./media/use-data-from-an-on-premises-sql-server/database-credentials.png)

   Il messaggio "values required" (valori richiesti) verrà modificato in "values set" (valori impostati) con un segno di spunta verde. Se la password o le informazioni del database non vengono modificate, è sufficiente immettere le credenziali una sola volta. Azure Machine Learning Studio Usa il certificato fornito quando è stato installato il gateway per crittografare le credenziali nel cloud. Azure non archivia mai credenziali locali senza crittografia.

   ![Proprietà del modulo Import Data](./media/use-data-from-an-on-premises-sql-server/import-data-properties-entered.png)
8. Fare clic su **RUN** (ESEGUI) per eseguire l'esperimento.

Al termine dell'esecuzione dell'esperimento è possibile visualizzare i dati importati dal database facendo clic sulla porta di output del modulo **Import Data** (Importa dati) e selezionando **Visualize** (Visualizza).

Dopo aver completato lo sviluppo dell'esperimento, è possibile distribuire il modello e renderlo operativo. I dati del database SQL Server locale configurati nel modulo **Import Data** (Importa dati) verranno letti e usati per l'assegnazione dei punteggi tramite il servizio Esecuzione batch. Sebbene per l'assegnazione dei punteggi ai dati locali sia possibile usare il servizio di richiesta/risposta, Microsoft consiglia l'uso del [componente aggiuntivo di Excel](excel-add-in-for-web-services.md) . La scrittura in un database SQL Server locale tramite **Export data** (Esporta dati) non è attualmente supportata, né negli esperimenti né nei servizi Web pubblicati.
