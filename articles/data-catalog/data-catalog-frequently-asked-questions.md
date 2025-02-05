---
title: Domande frequenti sul Catalogo dati di Azure
description: Domande frequenti relative al Catalogo dati di Azure, incluse le funzionalità per la gestione, l'annotazione e l'individuazione dell'origine dati.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: 2e286854b9a38c1189ff85307f3e29454be46fb9
ms.sourcegitcommit: 3e7646d60e0f3d68e4eff246b3c17711fb41eeda
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/11/2019
ms.locfileid: "70898723"
---
# <a name="azure-data-catalog-frequently-asked-questions"></a>Domande frequenti sul Catalogo dati di Azure
Questo articolo fornisce le risposte alle domande frequenti relative al servizio Azure Data Catalog.

## <a name="what-is-azure-data-catalog"></a>Che cos'è il Catalogo dei dati di Azure?
Data Catalog è un servizio completamente gestito, ospitato in Microsoft Azure, che funge da sistema di registrazione e di individuazione per origini dati aziendali. Con Data Catalog, tutti gli utenti, dagli analisti ai data scientist fino agli sviluppatori, possono registrare, trovare, comprendere e utilizzare le origini dati.

## <a name="what-customer-challenges-does-it-solve"></a>Quali sono le problematiche dei clienti che consente di risolvere?
Data Catalog risolve la richiesta di individuazione dell'origine dati e dei cosiddetti "dati oscuri" in modo che gli utenti possano identificare e comprendere le origini dati aziendali.

## <a name="what-are-its-target-audiences"></a>A chi è destinato?
Data Catalog è progettato per gli utenti tecnici e non tecnici, tra cui:

* Sviluppatori di dati e professionisti di analisi e Business Intelligence: Persone responsabili della produzione di dati e contenuti analitici per l'utilizzo da altri utenti.
* Amministratori dei dati: Utenti che conoscono i dati, cosa significa e come sono destinati a essere usati.
* Consumer di dati: Utenti che devono essere in grado di individuare, comprendere e connettersi facilmente ai dati necessari per svolgere il proprio lavoro, usando lo strumento di propria scelta.
* IT centrale: Persone che devono rendere individuabili centinaia di origini dati dagli utenti aziendali e che devono mantenere la supervisione della modalità d'uso dei dati e da chi.

## <a name="what-is-its-availability-by-region"></a>Qual è la disponibilità per area?
Attualmente i servizi di Data Catalog sono disponibili nei data center seguenti:

* Stati Uniti occidentali
* East US
* Europa occidentale
* Europa settentrionale
* Australia orientale
* Asia sud-orientale

## <a name="what-are-its-limits-on-the-number-of-data-assets"></a>Quali sono i limiti relativi al numero di asset di dati?
L'edizione gratuita di Data Catalog è limitata a 5.000 asset di dati registrati.

L'edizione Standard di Date Catalog supporta fino a 100.000 asset di dati registrati.

Qualsiasi oggetto registrato nel Data Catalog, ad esempio tabelle, viste, file e report, viene considerato un asset di dati.

## <a name="what-are-its-supported-data-source-and-asset-types"></a>Quali sono i tipi di origine dati e di asset supportati?
Per un elenco di origini dati attualmente supportate, vedere [Riferimento per l'origine dati di Azure Data Catalog](data-catalog-dsr.md).

## <a name="how-do-i-request-support-for-another-data-source"></a>Come si richiede il supporto per un'altra origine dati?
Per inviare le richieste di funzionalità e altri commenti, andare su [Data Catalog in Azure Feedback Forums](https://feedback.azure.com/forums/906052-data-catalog/category/320788-data-sources).

## <a name="why-do-i-get-an-error-catalog-already-exists-when-i-try-to-create-a-new-catalog"></a>Perché è *già presente un catalogo* degli errori quando si tenta di creare un nuovo catalogo?

Quando si acquista Office 365 E5 con Power BI Pro licenza, Microsoft crea automaticamente un catalogo predefinito nell'area della sottoscrizione. Questo catalogo usa lo SKU gratuito. La licenza utente di Office 365/Power BI viene gestita nella pagina di amministrazione di Office 365. 

Tuttavia, questo tipo di catalogo dati non dispone di un' **opzione di amministratore** e non è visibile nel **portale di Azure**. Non è possibile eliminare questo tipo di catalogo dati. Analogamente, non è consentito rinominare il Catalogo dati e non è possibile spostarlo in un'altra area. 

Gli account utente a cui è stata assegnata una licenza di Power BI Pro automatica hanno accesso al Catalogo dati a causa del contratto di licenza quando hanno effettuato l'iscrizione a Office 365 E5 con la licenza di Power BI Pro. Questo tipo di utente ha accesso completo agli asset di Data Catalog senza privilegi amministrativi. Questo tipo di utente *non* fa parte del ruolo **utente Catalog** in Azure Data Catalog.


## <a name="how-do-i-get-started-with-data-catalog"></a>Come si inizia a usare Data Catalog?
Il modo migliore per iniziare è vedere l'articolo [Introduzione ad Azure Data Catalog](data-catalog-get-started.md). In questo articolo è riportata una panoramica end-to-end delle funzionalità nel servizio.

## <a name="how-do-i-register-my-data"></a>Come si registrano i dati?
Per registrare i dati in Data Catalog:
1. Nell'area **Pubblica** del portale di Azure Data Catalog avviare lo strumento di registrazione di Azure Data Catalog. 
2. Nello strumento di registrazione dell'origine dati di Data Catalog eseguire l'accesso con le stesse credenziali usate per accedere al portale di Data Catalog.
3. Selezionare l'origine dati e gli asset specifici che si vuole registrare.

## <a name="what-properties-does-it-extract-for-data-assets-that-are-registered"></a>Quali proprietà estrae per gli asset di dati registrati?
Le proprietà specifiche variano in base all'origine dati, ma in genere il servizio di pubblicazione di Data Catalog estrae le informazioni seguenti:

* Nome dell’asset
* Tipo di asset
* Descrizione dell’asset
* Nomi di colonna/attributo
* Tipi di dati di colonna/attributo
* Descrizione di colonna/attributo

> [!IMPORTANT]
> La registrazione degli asset di dati con Data Catalog non sposta o copia i dati nel cloud. Registrando gli asset da un'origine dati si copiano i metadati degli asset in Azure, ma i dati rimangono nel percorso esistente dell'origine dati. L'eccezione a questa regola si rileva quando si sceglie di caricare i record di anteprima o un profilo dei dati quando si registrano gli asset. Quando si include un'anteprima, vengono copiati fino a 20 record da ogni asset e vengono archiviati come snapshot in Data Catalog. Quando si include un profilo dei dati, le informazioni aggregate vengono calcolate e incluse nei metadati archiviati nel catalogo. Le informazioni aggregate possono includere le dimensioni delle tabelle, la percentuale di valori null per colonna o i valori minimi, massimi e medi per le colonne. 
>
>

> [!NOTE]
> Per le origini dati come SQL Server Analysis Services che hanno una proprietà **Description** ottimale, lo strumento di registrazione dell'origine dati di Data Catalog estrae il valore della proprietà. Per i database relazionali di SQL Server, che non hanno una proprietà **Description** ottimale, lo strumento di registrazione dell'origine dati di Data Catalog estrae il valore dalla proprietà estesa **ms_description** per oggetti e colonne. Per altre informazioni, vedere [Uso di proprietà estese su oggetti di database](https://technet.microsoft.com/library/ms190243%28v=sql.105%29.aspx).
>
>

## <a name="how-long-should-it-take-for-newly-registered-assets-to-appear-in-the-catalog"></a>Quanto tempo deve trascorrere prima che gli asset appena registrati vengano visualizzati nel catalogo?
Dopo aver registrato gli asset in Data Catalog è possibile che trascorrano da 5 a 10 secondi prima che vengano visualizzati nel portale di Data Catalog.

## <a name="how-do-i-annotate-and-enrich-the-metadata-for-my-registered-data-assets"></a>Come si annotano e ampliano i metadati per l'asset di dati registrato?
Il modo più semplice per fornire i metadati agli asset registrati consiste nel selezionare l'asset nel portale di Data Catalog e immettere i valori nel riquadro delle proprietà o dello schema per l'oggetto selezionato.

È inoltre possibile fornire alcuni metadati, ad esempio esperti e tag, durante il processo di registrazione. I valori forniti nel servizio di pubblicazione di Data Catalog vengono applicati a tutti gli asset che si trovano in fase di registrazione. Per visualizzare gli oggetti registrati recentemente nel portale per aggiungere annotazioni, selezionare il pulsante **Visualizza portale** nella schermata finale dello strumento di registrazione dell'origine dati di Data Catalog.

## <a name="how-do-i-delete-my-registered-data-objects"></a>Come si eliminano gli oggetti dati registrati?
È possibile eliminare un oggetto da Data Catalog selezionandolo nel portale e quindi scegliendo il pulsante **Elimina**. Rimuovendo l'oggetto vengono rimossi da Data Catalog anche i metadati, ma non l'origine dati sottostante.

## <a name="what-is-an-expert"></a>Che cos'è un esperto?
Un esperto è una persona che ha una prospettiva informata su un oggetto dati. Un oggetto può avere più esperti. Un esperto non deve necessariamente essere il "proprietario" dell'oggetto, ma è una persona che sa come i dati possono e devono essere usati.

## <a name="how-do-i-share-information-with-the-data-catalog-team-if-i-encounter-problems"></a>Come si condividono le informazioni con il team di Data Catalog se si riscontrano problemi?
Per segnalare problemi e condividere informazioni e domande, andare al [forum di Azure Data Catalog](https://go.microsoft.com/fwlink/?LinkID=616424&clcid=0x409).

## <a name="does-the-catalog-work-with-another-data-source-that-im-interested-in"></a>Il catalogo funziona con un'altra origine dati a cui si è interessati?
Grazie al lavoro che si sta svolgendo, in futuro è prevista l'aggiunta di altre origini dati a Data Catalog. Se si è interessati al supporto di un'origine dati specifica, scrivere un suggerimento in proposito (o esprimere il proprio sostegno se già suggerito) andando su [Data Catalog in Azure Feedback Forums](https://feedback.azure.com/forums/906052-data-catalog).

## <a name="what-permissions-do-i-need-to-register-assets-with-data-catalog"></a>Quali autorizzazioni sono necessarie per registrare gli asset in Data Catalog?
Per eseguire lo strumento di registrazione di Data Catalog, sono necessarie autorizzazioni per l'origine dati che consentano di leggere i metadati dall'origine. Per includere anche un'anteprima, è necessario avere autorizzazioni che consentono di leggere i dati dagli oggetti in fase di registrazione.

Data Catalog consente inoltre agli amministratori del catalogo di stabilire quali utenti e gruppi possono aggiungere metadati al catalogo. Per altre informazioni vedere [Come proteggere l'accesso al catalogo dati e agli asset di dati](data-catalog-how-to-secure-catalog.md).

## <a name="will-data-catalog-be-made-available-for-on-premises-deployment-as-well"></a>Data Catalog sarà disponibile anche per la distribuzione locale?
Data Catalog è un servizio cloud che può usare origini dati locali e del cloud per fornire una soluzione ibrida di individuazione dell'origine dati. Al momento non è prevista una versione del servizio Data Catalog da eseguire in locale.

## <a name="can-i-extract-more-or-richer-metadata-from-the-data-sources-i-register"></a>È possibile estrarre metadati più numerosi o migliorati dalle origini dati registrate?
Si sta lavorando attivamente per ampliare le funzionalità di Data Catalog. Se si è interessati all'estrazione di altri metadati dall'origine dati durante la registrazione, scrivere un suggerimento (o esprimere il proprio sostegno se già suggerito) sul [Data Catalog in Azure Feedback Forums](https://feedback.azure.com/forums/906052-data-catalog). 

Se si desidera includere i metadati dello schema o della colonna, le anteprime o i profili di dati, per le origini dati in cui i metadati non vengono estratti dallo strumento di registrazione dell'origine dati, è possibile usare l'API di Data Catalog per aggiungere questi metadati. Per altre informazioni, vedere l'[API REST di Azure Data Catalog](https://docs.microsoft.com/rest/api/datacatalog/).

## <a name="how-do-i-restrict-the-visibility-of-registered-data-assets-so-that-only-certain-people-can-discover-them"></a>Come si limita la visibilità degli asset di dati registrati, in modo che solo determinati utenti possano individuarli?
Selezionare gli asset di dati in Data Catalog e fare clic sul pulsante **Assumi proprietà**. I proprietari degli asset di dati in Data Catalog possono modificare le impostazioni di visibilità per consentire a tutti gli utenti di trovare gli asset di proprietà o limitare la visibilità a utenti specifici. Per altre informazioni, vedere [Gestire gli asset di dati in Azure Data Catalog](data-catalog-how-to-manage.md).

## <a name="how-do-i-update-the-registration-for-a-data-asset-so-that-changes-in-the-data-source-are-reflected-in-the-catalog"></a>Come si aggiorna la registrazione di un asset di dati in modo che le modifiche apportate nell'origine dati vengano riflesse nel catalogo?
Per aggiornare i metadati degli asset di dati già registrati nel catalogo, è sufficiente registrare nuovamente l'origine dati che contiene gli asset. Le modifiche nell'origine dati, ad esempio l'aggiunta o la rimozione di colonne da tabelle o viste, vengono aggiornate nel catalogo, ma tutte le annotazioni fornite dagli utenti vengono conservate.

## <a name="my-question-isnt-answered-here-where-can-i-go-for-answers"></a>Non sono disponibili risposte a una domanda. Dove è possibile trovarle?
Passare al [forum di Azure Data Catalog](https://go.microsoft.com/fwlink/?LinkID=616424&clcid=0x409). dove le domande frequenti trovano risposta.
