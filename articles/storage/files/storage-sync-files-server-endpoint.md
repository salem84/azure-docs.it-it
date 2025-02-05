---
title: Aggiungere e rimuovere un endpoint server di Sincronizzazione file di Azure | Microsoft Docs
description: Informazioni sugli aspetti da considerare quando si pianifica una distribuzione di File di Azure.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 684b30a24e049722cb531cbc84e3a2cd90912ec8
ms.sourcegitcommit: f3f4ec75b74124c2b4e827c29b49ae6b94adbbb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/12/2019
ms.locfileid: "70932629"
---
# <a name="addremove-an-azure-file-sync-server-endpoint"></a>Aggiungere e rimuovere un endpoint server di Sincronizzazione file di Azure
Sincronizzazione file di Azure consente di centralizzare le condivisioni file dell'organizzazione in File di Azure senza rinunciare alla flessibilità, alle prestazioni e alla compatibilità di un file server locale. Tutto questo avviene trasformando i sistemi Windows Server in una cache rapida della condivisione file di Azure. È possibile usare qualsiasi protocollo disponibile in Windows Server per accedere ai dati in locale (tra cui SMB, NFS e FTPS) ed è possibile scegliere tutte le cache necessarie in tutto il mondo.

Un *endpoint server* rappresenta una posizione specifica in un *server registrato*, ad esempio una cartella in un volume del server o la radice del volume. Possono esistere più endpoint server nello stesso volume se i relativi spazi dei nomi non si sovrappongono, ad esempio F:\sync1 e F:\sync2. È possibile configurare i criteri di suddivisione in livelli nel cloud singolarmente per ogni endpoint server. Se si aggiunge a un gruppo di sincronizzazione un percorso del server con un set di file esistente come endpoint server, i file verranno uniti con qualsiasi altro file già presente in altri endpoint del gruppo di sincronizzazione.

Per informazioni sulla distribuzione di Sincronizzazione file di Azure end-to-end, vedere [Come distribuire Sincronizzazione file di Azure](storage-sync-files-deployment-guide.md).

## <a name="prerequisites"></a>Prerequisiti
Per creare un endpoint server, è necessario prima verificare che siano soddisfatti i criteri seguenti: 
- L'agente Sincronizzazione file di Azure è installato nel server e il server è stato registrato. Le istruzioni per l'installazione dell'agente Sincronizzazione file di Azure sono contenute nell'articolo [Registrare e annullare la registrazione di un server con Sincronizzazione file di Azure](storage-sync-files-server-registration.md). 
- Verificare che sia stato distribuito un servizio di sincronizzazione archiviazione. Per informazioni dettagliate sulla distribuzione di un servizio di sincronizzazione archiviazione, vedere [Come distribuire Sincronizzazione file di Azure](storage-sync-files-deployment-guide.md). 
- Verificare che sia stato distribuito un gruppo di sincronizzazione. Leggere le informazioni su [come creare un gruppo di sincronizzazione](storage-sync-files-deployment-guide.md#create-a sync-group-and-a-cloud-endpoint).
- Verificare che il server sia connesso a Internet e che Azure sia accessibile. Per tutte le comunicazioni tra il server e il servizio viene usata la porta 443.

## <a name="add-a-server-endpoint"></a>Aggiungere un endpoint server
Per aggiungere un endpoint server, passare al gruppo di sincronizzazione che interessa e selezionare "Aggiungi endpoint server".

![Aggiungere un nuovo endpoint server nel riquadro del gruppo di sincronizzazione](media/storage-sync-files-server-endpoint/add-server-endpoint-1.png)

Inserire le informazioni seguenti in **Aggiungi endpoint server**:

- **Server registrato**: nome del server o del cluster in cui viene creato l'endpoint server.
- **Percorso**: il percorso in Windows Server da sincronizzare come parte del gruppo di sincronizzazione.
- **Suddivisione in livelli nel cloud**: l'opzione che abilita o disabilita la suddivisione in livelli cloud, Se la suddivisione in livelli cloud è abilitata, i file vengono *suddivisi in livelli* nelle condivisioni file di Azure. Ciò converte le condivisioni file locali in una cache, invece di una copia completa del set di dati, per aiutare a gestire l'efficienza dello spazio nel server.
- **Spazio disponibile nel volume**: quantità di spazio disponibile da riservare nel volume in cui si trova l'endpoint server. Ad esempio, se lo spazio disponibile nel volume è impostato su 50% per un volume con un singolo endpoint server, circa la metà dei dati verrà archiviata a livelli in File di Azure. A prescindere dall'abilitazione o meno della suddivisione in livelli nel cloud, per la condivisione file di Azure è sempre disponibile una copia completa dei dati nel gruppo di sincronizzazione.

Selezionare **Crea** per aggiungere l'endpoint server. I file all'interno di uno spazio dei nomi di un gruppo di sincronizzazione ora rimangono sincronizzati. 

## <a name="remove-a-server-endpoint"></a>Rimuovere un endpoint server
Se si desidera interrompere l'utilizzo di Sincronizzazione file di Azure per un determinato endpoint server, è possibile rimuovere l'endpoint server. 

> [!Warning]  
> Non cercare di risolvere i problemi relativi a sincronizzazione, suddivisione in livelli cloud o qualsiasi altro aspetto di Sincronizzazione file di Azure rimuovendo e ricreando l'endpoint server, a meno che ciò non venga richiesto in modo esplicito da un tecnico Microsoft. La rimozione di un endpoint server è un'operazione distruttiva e i file suddivisi in livelli presenti nell'endpoint server non verranno "riconnessi" ai relativi percorsi nella condivisione file di Azure dopo che l'endpoint server è stato ricreato, causando errori di sincronizzazione. Si noti anche che i file suddivisi in livelli che si trovano all'esterno dello spazio dei nomi dell'endpoint server potrebbero andare definitivamente persi. I file suddivisi in livelli possono essere presenti nell'endpoint server anche se la funzionalità di suddivisione in livelli cloud non è mai stata abilitata.

Per garantire che tutti i file suddivisi in livelli vengano richiamati prima di rimuovere l'endpoint server, disabilitare la suddivisione in livelli cloud nell'endpoint server e quindi eseguire il cmdlet di PowerShell seguente per richiamare tutti i file suddivisi in livelli all'interno dello spazio dei nomi dell'endpoint server:

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint> -Order CloudTieringPolicy
```
Se `-Order CloudTieringPolicy` si specifica, vengono richiamati prima i file modificati più di recente.
Altri parametri facoltativi ma utili da considerare sono:
* `-ThreadCount`determina il numero di file che possono essere richiamati in parallelo.
* `-PerFileRetryCount`determina la frequenza con cui viene eseguito il tentativo di richiamo di un file attualmente bloccato.
* `-PerFileRetryDelaySeconds`determina il tempo in secondi tra i tentativi per richiamare i tentativi e deve essere sempre utilizzato in combinazione con il parametro precedente.

> [!Note]  
> Se il volume locale che ospita il server non ha abbastanza spazio libero per richiamare tutti i dati archiviati a livelli, il cmdlet `Invoke-StorageSyncFileRecall` ha esito negativo.  

Per rimuovere l'endpoint server:

1. Passare al servizio di sincronizzazione archiviazione in cui è registrato il server.
2. Passare al gruppo di sincronizzazione che interessa.
3. Rimuovere l'endpoint server del gruppo di sincronizzazione nel servizio di sincronizzazione archiviazione. L'operazione può essere eseguita facendo clic con il pulsante destro del mouse sull'endpoint server nel riquadro del gruppo di sincronizzazione.

    ![Rimozione di un endpoint server da un gruppo di sincronizzazione](media/storage-sync-files-server-endpoint/remove-server-endpoint-1.png)

## <a name="next-steps"></a>Passaggi successivi
- [Registrare e annullare la registrazione di un server con Sincronizzazione file di Azure](storage-sync-files-server-registration.md)
- [Pianificazione per la distribuzione di Sincronizzazione file di Azure](storage-sync-files-planning.md)
- [Monitorare Sincronizzazione file di Azure](storage-sync-files-monitoring.md)
