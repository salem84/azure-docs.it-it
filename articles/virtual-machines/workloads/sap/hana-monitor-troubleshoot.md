---
title: Monitoraggio e risoluzione dei problemi dal lato HANA in SAP HANA di Azure (Istanze large) | Microsoft Docs
description: Monitoraggio e risoluzione dei problemi dal lato HANA in SAP HANA di Azure (Istanze large).
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: gwallace
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e2c596a876817f0a501025c37e463a7eebb55cf2
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70099830"
---
# <a name="monitoring-and-troubleshooting-from-hana-side"></a>Monitoraggio e risoluzione dei problemi dal lato HANA

Per analizzare efficacemente i problemi correlati a SAP HANA in Azure (istanze di grandi dimensioni), è utile circoscrivere la causa principale di un problema. SAP ha pubblicato molta documentazione di supporto.

Domande frequenti relative alle prestazioni di SAP HANA sono reperibili nelle note SAP seguenti:

- [Note SAP #2222200 – domande frequenti: Rete SAP HANA](https://launchpad.support.sap.com/#/notes/2222200)
- [Note SAP #2100040 – domande frequenti: CPU SAP HANA](https://launchpad.support.sap.com/#/notes/0002100040)
- [Note SAP #199997 – domande frequenti: Memoria SAP HANA](https://launchpad.support.sap.com/#/notes/2177064)
- [Note SAP #200000 – domande frequenti: Ottimizzazione delle prestazioni SAP HANA](https://launchpad.support.sap.com/#/notes/2000000)
- [Note SAP #199930 – domande frequenti: Analisi di I/O SAP HANA](https://launchpad.support.sap.com/#/notes/1999930)
- [Note SAP #2177064 – domande frequenti: Riavvio e arresti anomali del servizio SAP HANA](https://launchpad.support.sap.com/#/notes/2177064)

## <a name="sap-hana-alerts"></a>Avvisi SAP HANA

Innanzitutto controllare i log di avviso correnti di SAP HANA. In SAP Hana Studio passare alla console **di amministrazione: Avvisi Mostra: tutti gli**avvisi. Questa scheda mostra tutti gli avvisi di SAP HANA per valori specifici (memoria fisica disponibile, utilizzo della CPU e così via) che si trovano all'esterno delle soglie minima e massima stabilite. Per impostazione predefinita i controlli vengono aggiornati automaticamente ogni 15 minuti.

![In SAP HANA Studio passare alla console di amministrazione: Avvisi Mostra: tutti gli avvisi](./media/troubleshooting-monitoring/image1-show-alerts.png)

## <a name="cpu"></a>CPU

Per un avviso generato a causa di un'impostazione non corretta della soglia, una possibile soluzione è reimpostare il valore predefinito o un valore di soglia accettabile.

![Reimpostare il valore predefinito o un valore di soglia accettabile](./media/troubleshooting-monitoring/image2-cpu-utilization.png)

Gli avvisi seguenti possono indicare problemi di risorse della CPU:

- Utilizzo della CPU dell'host (avviso 5)
- Funzionamento del punto di salvataggio più recente (avviso 28)
- Durata del punto di salvataggio (avviso 54)

È possibile notare un elevato utilizzo della CPU sul database SAP HANA da uno dei seguenti elementi:

- L'avviso 5 (utilizzo della CPU dell'host) viene generato per l'utilizzo della CPU corrente o precedente
- L'utilizzo della CPU visualizzato nella schermata della panoramica

![Utilizzo della CPU visualizzato nella schermata della panoramica](./media/troubleshooting-monitoring/image3-cpu-usage.png)

Il grafico di carico potrebbe mostrare un elevato utilizzo della CPU o un elevato utilizzo precedente:

![Il grafico di carico potrebbe mostrare un elevato utilizzo della CPU o un elevato utilizzo precedente](./media/troubleshooting-monitoring/image4-load-graph.png)

Un avviso generato a causa di un utilizzo elevato della CPU potrebbe essere causato da vari motivi, tra cui, a sua scelta, l'esecuzione di determinate transazioni, il caricamento di dati, processi che non rispondono, istruzioni SQL con esecuzione prolungata e prestazioni di query non valide, ad esempio con BW nei cubi HANA).

Vedere la [SAP Hana risoluzione dei problemi: Cause e soluzioni](https://help.sap.com/saphelp_hanaplatform/helpdata/en/4f/bc915462db406aa2fe92b708b95189/content.htm?frameset=/en/db/6ca50424714af8b370960c04ce667b/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=46&amp;show_children=false) correlate alla CPU per i passaggi dettagliati della risoluzione dei problemi.

## <a name="operating-system"></a>Sistema operativo

Uno dei controlli più importanti per SAP HANA in Linux consiste nel verificare che siano disattivate le transparent huge page: vedere la [nota SAP #2131662 dedicata alle transparent huge page (THP) sui server SAP HANA](https://launchpad.support.sap.com/#/notes/2131662).

- È possibile controllare se le transparent huge page sono attivate tramite il seguente comando di Linux: **cat /sys/kernel/mm/transparent\_hugepage/enabled**
- Se _always_ è racchiuso tra parentesi quadre come mostrato di seguito, significa che le transparent huge page sono attivate: [always] madvise never; se _never_ è racchiuso tra parentesi quadre come mostrato di seguito, significa che le transparent huge page sono disattivate: always madvise [never]

Il seguente comando di Linux non deve restituire alcun valore: **rpm -qa | grep ulimit**. Nel caso sembrasse che _ulimit_ è installato, disinstallarlo immediatamente.

## <a name="memory"></a>Memoria

È possibile osservare che la quantità di memoria allocata per il database SAP HANA è maggiore del previsto. Gli avvisi seguenti indicano problemi con un utilizzo elevato della memoria:

- Utilizzo di memoria fisica dell'host (avviso 1)
- Utilizzo di memoria del server dei nomi (avviso 12)
- Utilizzo di memoria totale delle tabelle columnstore (avviso 40)
- Utilizzo di memoria dei servizi (avviso 43)
- Utilizzo di memoria totale della risorsa di archiviazione principale delle tabelle columnstore (avviso 45)
- File di dump di runtime (avviso 46)

Vedere la [SAP Hana risoluzione dei problemi: Sito problemi](https://help.sap.com/saphelp_hanaplatform/helpdata/en/db/6ca50424714af8b370960c04ce667b/content.htm?frameset=/en/59/5eaa513dde43758b51378ab3315ebb/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=26&amp;show_children=false) di memoria per i passaggi dettagliati della risoluzione dei problemi.

## <a name="network"></a>Rete

Vedere la [nota SAP #2081065 dedicata alla risoluzione dei problemi di rete di SAP HANA](https://launchpad.support.sap.com/#/notes/2081065) e svolgere i passaggi di risoluzione dei problemi riportati in questa nota SAP.

1. Analisi del tempo di round trip tra client e server.
  R. Eseguire lo script SQL [_HANA\_Network\_Clients_](https://launchpad.support.sap.com/#/notes/1969700) _._
  
2. Analizzare le comunicazioni internodo.
  R. Eseguire lo script SQL [_HANA\_Network\_Services_](https://launchpad.support.sap.com/#/notes/1969700) _._

3. Eseguire il comando di Linux **ifconfig** (l'output mostra se si verificano perdite di pacchetti).
4. Eseguire il comando di Linux **tcpdump**.

Usare inoltre lo strumento open source [IPERF](https://iperf.fr/) (o simile) per misurare le prestazioni reali della rete applicazioni.

Vedere la [SAP Hana risoluzione dei problemi: Sito problemi](https://help.sap.com/saphelp_hanaplatform/helpdata/en/a3/ccdff1aedc4720acb24ed8826938b6/content.htm?frameset=/en/dc/6ff98fa36541e997e4c719a632cbd8/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=142&amp;show_children=false) di connettività e prestazioni di rete per procedure dettagliate per la risoluzione dei problemi.

## <a name="storage"></a>Archiviazione

Dal punto di vista dell'utente finale, un'applicazione (o il sistema nel suo complesso) viene eseguita lentamente, non risponde o può sembrare non rispondere se si verificano problemi con le prestazioni di I/O. Nella scheda **Volumes** (Volumi) di SAP HANA Studio è possibile visualizzare i volumi collegati e quali volumi sono utilizzati da ciascun servizio.

![Nella scheda Volumes (Volumi) di SAP HANA Studio è possibile visualizzare i volumi collegati e quali volumi sono utilizzati da ciascun servizio](./media/troubleshooting-monitoring/image5-volumes-tab-a.png)

Nella parte inferiore della schermata sono visualizzati i dettagli dei volumi, ad esempio i file e le statistiche di I/O.

![Nella parte inferiore della schermata sono visualizzati i dettagli dei volumi, ad esempio i file e le statistiche di I/O](./media/troubleshooting-monitoring/image6-volumes-tab-b.png)

Vedere la [SAP Hana risoluzione dei problemi: Cause e soluzioni](https://help.sap.com/saphelp_hanaplatform/helpdata/en/dc/6ff98fa36541e997e4c719a632cbd8/content.htm?frameset=/en/47/4cb08a715c42fe9f7cc5efdc599959/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=55&amp;show_children=false) principali correlate all'I/O [e SAP Hana risoluzione dei problemi: Sito per le cause e le](https://help.sap.com/saphelp_hanaplatform/helpdata/en/47/4cb08a715c42fe9f7cc5efdc599959/content.htm?frameset=/en/44/3e1db4f73d42da859008df4f69e37a/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=53&amp;show_children=false) soluzioni radice correlate al disco per procedure dettagliate per la risoluzione dei problemi.

## <a name="diagnostic-tools"></a>Strumenti di diagnostica

Eseguire un controllo di integrità di SAP HANA tramite HANA\_Configuration\_Minichecks. Questo strumento restituisce problemi tecnici critici che dovrebbero essere già stati notificati mediante avvisi in SAP HANA Studio.

Vedere la [nota SAP #1969700 dedicata alla raccolta di istruzioni SQL per SAP HANA](https://launchpad.support.sap.com/#/notes/1969700) e scaricare il file Statements.zip SQL associato alla nota. Salvare il file .zip sul disco rigido locale.

In SAP HANA Studio, sulla scheda **System Information** (Informazioni di sistema), fare clic con il pulsante destro del mouse nella colonna **Name** (Nome) e selezionare **Import SQL Statements** (Importa istruzioni SQL).

![In SAP HANA Studio, sulla scheda System Information (Informazioni di sistema), fare clic con il pulsante destro del mouse nella colonna Nome (Nome) e selezionare Import SQL Statements (Importa istruzioni SQL)](./media/troubleshooting-monitoring/image7-import-statements-a.png)

Selezionare il file SQL Statements.zip salvato in locale per importare una cartella con le istruzioni SQL corrispondenti. A questo punto possono essere eseguiti i molti diversi controlli di diagnostica con queste istruzioni SQL.

Ad esempio, per testare i requisiti di larghezza di banda di replica di sistema SAP Hana, fare **clic con il pulsante destro del mouse sull'istruzione bandwidth in replica Larghezza** di banda e selezionare **Apri** nella console di SQL.

Verrà visualizzata l'istruzione SQL completa e sarà possibile modificare i parametri di input (sezione di modifica) e poi eseguirla.

![Verrà visualizzata l'istruzione SQL completa e sarà possibile modificare i parametri di input (sezione di modifica) e poi eseguirla](./media/troubleshooting-monitoring/image8-import-statements-b.png)

Un altro esempio è fare clic con il pulsante destro **del mouse sulle istruzioni in replica: Panoramica**. Selezionare **Run** (Esegui) dal menu di scelta rapida:

![Un altro esempio è fare clic con il pulsante destro del mouse sulle istruzioni in replica: Panoramica. Selezionare Run (Esegui) dal menu di scelta rapida](./media/troubleshooting-monitoring/image9-import-statements-c.png)

Vengono visualizzate informazioni utili per la risoluzione dei problemi:

![Vengono visualizzate informazioni utili per la risoluzione dei problemi](./media/troubleshooting-monitoring/image10-import-statements-d.png)

Eseguire la stessa operazione per HANA\_Configuration\_Minichecks e cercare eventuali _X_ nella colonna _C_ (livello critico).

Output di esempio:

**HANA\_Configuration\_MiniChecks\_Rev102.01+1** per controlli SAP HANA generali.

![HANA\_Configuration\_MiniChecks\_Rev102.01+1 per controlli SAP HANA generali](./media/troubleshooting-monitoring/image11-configuration-minichecks.png)

**HANA\_Services\_Overview** per una panoramica di quanto stanno eseguendo attualmente i servizi SAP HANA.

![HANA\_Services\_Overview per una panoramica di quanto stanno eseguendo attualmente i servizi SAP HANA](./media/troubleshooting-monitoring/image12-services-overview.png)

**HANA\_Services\_Statistics** per informazioni sul servizio SAP HANA (CPU, memoria, ecc.).

![HANA\_Services\_Statistics per informazioni sul servizio SAP HANA](./media/troubleshooting-monitoring/image13-services-statistics.png)

**HANA\_Configuration\_Overview\_Rev110+** per informazioni generali sull'istanza di SAP HANA.

![HANA\_Configuration\_Overview\_Rev110+ per informazioni generali sull'istanza di SAP HANA](./media/troubleshooting-monitoring/image14-configuration-overview.png)

**HANA\_Configuration\_Parameters\_Rev70+** per controllare i parametri di SAP HANA.

![HANA\_Configuration\_Parameters\_Rev70+ per controllare i parametri di SAP HANA](./media/troubleshooting-monitoring/image15-configuration-parameters.png)

**Passaggi successivi**

- Consultare[Configurazione della disponibilità elevata di SUSE usando STONITH](ha-setup-with-stonith.md).