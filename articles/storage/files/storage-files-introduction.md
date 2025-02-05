---
title: Introduzione a File di Azure | Microsoft Docs
description: Panoramica di File di Azure, un servizio che consente di creare e usare condivisioni file di rete nel cloud usando il protocollo SMB standard di settore.
author: roygara
ms.service: storage
ms.topic: overview
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 7fe95a35a6b78bae66521a09d63d376ec8757479
ms.sourcegitcommit: 800f961318021ce920ecd423ff427e69cbe43a54
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/31/2019
ms.locfileid: "68699686"
---
# <a name="what-is-azure-files"></a>Informazioni su File di Azure
File di Azure offre condivisioni file completamente gestite sul cloud, accessibili tramite il [protocollo SMB (Server Message Block)](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx) standard di settore. Le condivisioni file di Azure possono essere montate simultaneamente da distribuzioni cloud o locali di Windows, macOS e Linux. Le condivisioni file di Azure possono essere anche memorizzate nella cache in Windows Server con Sincronizzazione file di Azure per l'accesso rapido nelle vicinanze del punto in cui vengono usati i dati.

## <a name="videos"></a>Video
| Introduzione a Sincronizzazione file di Azure (2 minuti) | File di Azure con sincronizzazione (Ignite 2017, 85 minuti)  |
|-|-|
| [![Screencast del video di introduzione a Sincronizzazione file di Azure (fare clic per riprodurre)](./media/storage-files-introduction/azure-file-sync-video-snapshot.png)](https://www.youtube.com/watch?v=Zm2w8-TRn-o) | [![Screencast della presentazione su File di Azure e la sincronizzazione (fare clic per riprodurre)](./media/storage-files-introduction/ignite-2018-video.png)](https://www.youtube.com/watch?v=GMzh2M66E9o) |

## <a name="why-azure-files-is-useful"></a>Vantaggi offerti da File di Azure
Le condivisioni file di Azure possono essere usate per:

* **Sostituire o integrare file server locali**:  
    È possibile usare File di Azure per sostituire completamente o integrare i dispositivi NAS o i file server locali tradizionali. I sistemi operativi più diffusi, come Windows, macOS e Linux, possono montare direttamente condivisioni file di Azure ovunque nel mondo. Le condivisioni file di Azure possono anche essere replicate nei server Windows con Sincronizzazione file di Azure, in locale o nel cloud, per ottenere prestazioni elevate e per un caching efficiente e distribuito dei dati nella posizione in cui vengono usati.

* **Applicazioni "lift-and-shift"** :  
    File di Azure semplifica il trasferimento nel cloud in modalità lift-and-shift di applicazioni che prevedono una condivisione file per archiviare i dati delle applicazioni o degli utenti. File di Azure consente sia lo scenario lift-and-shift "classico", in cui l'applicazione e i dati vengono trasferiti in Azure, che lo scenario lift-and-shift "ibrido", in cui i dati dell'applicazione vengono trasferiti in File di Azure e l'applicazione continua a essere eseguita in locale. 

* **Semplificare lo sviluppo per il cloud**:  
    È possibile usare File di Azure in molti modi per semplificare i nuovi progetti di sviluppo per il cloud. Ad esempio:
    * **Impostazioni delle applicazioni condivise**:  
        Un modello comune per le applicazioni distribuite consiste nell'inserire i file di configurazione in una posizione centralizzata in cui siano accessibili da numerose istanze dell'applicazione. Le istanze dell'applicazione possono caricare la propria configurazione tramite l'API REST di File e gli utenti possono accedere alle istanze in base alle esigenze mondando la condivisione SMB in locale.

    * **Condivisione di diagnostica**:  
        Una condivisione file di Azure è un pratico strumento che permette alle applicazioni cloud di scrivere log, metriche e dump di arresto anomalo del sistema. I log possono essere scritti dalle istanze dell'applicazione tramite l'API REST di File e gli sviluppatori possono accedervi montando la condivisione file nel computer locale. Ciò consente una notevole flessibilità perché gli sviluppatori possono dedicarsi allo sviluppo cloud senza dover abbandonare gli strumenti che già conoscono e preferiscono.

    * **Sviluppo/test/debug**  
        Quando gli sviluppatori o gli amministratori usano VM nel cloud, spesso necessitano di un set di strumenti o di utilità. La copia di tali strumenti e utilità in ogni macchina virtuale può richiedere molto tempo. Montando una condivisione file di Azure in locale nelle macchine virtuali, sviluppatori e amministratori possono accedere rapidamente a strumenti e utilità, senza che siano necessarie attività di copia.

## <a name="key-benefits"></a>Vantaggi principali
* **Accesso condiviso**. Le condivisioni file di Azure supportano il protocollo SMB standard di settore. Di conseguenza, è possibile sostituire facilmente le condivisioni file locali con condivisioni file di Azure senza preoccuparsi della compatibilità delle applicazioni. La possibilità di condividere un file system tra più computer, applicazioni e istanze è un vantaggio importante di File di Azure per le applicazioni che richiedono condivisibilità. 
* **Soluzione completamente gestita**. È possibile creare condivisioni file di Azure senza dover gestire l'hardware o un sistema operativo. Di conseguenza, non è necessario applicare patch del sistema operativo server con aggiornamenti critici della sicurezza o sostituire dischi rigidi difettosi.
* **Script e strumenti**. È possibile usare i cmdlet di PowerShell e l'interfaccia della riga di comando di Azure per creare, montare e gestire le condivisioni file di Azure nell'ambito dell'amministrazione delle applicazioni Azure. Le condivisioni file di Azure possono essere create e gestite con il portale di Azure e Azure Storage Explorer. 
* **Resilienza**. File di Azure è stato progettato per garantire disponibilità costante. Sostituendo le condivisioni file locali con File di Azure non sarà più necessario occuparsi di problemi di rete o interruzioni dell'alimentazione a livello locale. 
* **Programmabilità nota**. Le applicazioni eseguite in Azure possono accedere ai dati nella condivisione tramite le [API di I/O del file system](https://msdn.microsoft.com/library/system.io.file.aspx). Gli sviluppatori possono quindi riutilizzare il codice esistente e le competenze acquisite per eseguire la migrazione delle applicazioni esistenti. Oltre alle API di I/O di sistema, è possibile usare [librerie client di archiviazione di Azure](https://msdn.microsoft.com/library/azure/dn261237.aspx) o l'[API REST di Archiviazione di Azure](/rest/api/storageservices/file-service-rest-api).

## <a name="next-steps"></a>Passaggi successivi
* [Creare una condivisione file di Azure](storage-how-to-create-file-share.md)
* [Eseguire la connessione e il montaggio in Windows](storage-how-to-use-files-windows.md)
* [Eseguire la connessione e il montaggio in Linux](storage-how-to-use-files-linux.md)
* [Eseguire la connessione e il montaggio in macOS](storage-how-to-use-files-mac.md)
