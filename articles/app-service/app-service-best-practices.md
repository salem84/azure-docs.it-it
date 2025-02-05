---
title: Procedure consigliate - Servizio app di Azure
description: Informazioni sulle procedure consigliate e la risoluzione dei problemi per il servizio app di Azure.
services: app-service
documentationcenter: ''
author: dariagrigoriu
manager: erikre
editor: mollybos
ms.assetid: f3359464-fa44-4f4a-9ea6-7821060e8d0d
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/01/2016
ms.author: dariac
ms.custom: seodec18
ms.openlocfilehash: c40191c8682d6ff93f70e0853e767c89248ae887
ms.sourcegitcommit: 82499878a3d2a33a02a751d6e6e3800adbfa8c13
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70071608"
---
# <a name="best-practices-for-azure-app-service"></a>Procedure consigliate per il servizio app di Azure
Questo articolo riepiloga le procedure consigliate per l'uso del [servizio app di Azure](https://go.microsoft.com/fwlink/?LinkId=529714). 

## <a name="colocation"></a>Condivisione dell'area
Quando le risorse di Azure che costituiscono una soluzione, ad esempio un'app Web e un database, si trovano in aree diverse, possono verificarsi le situazioni seguenti:

* Aumento della latenza nella comunicazione tra le risorse
* Addebiti per il trasferimento dei dati in uscita tra aree come indicato nella [pagina dei prezzi di Azure](https://azure.microsoft.com/pricing/details/data-transfers).

La condivisione dell'area è ideale per le risorse di Azure che compongono una soluzione per la memorizzazione di dati o contenuti, come un'app Web e un database o un account di archiviazione. Quando si creano risorse, verificare che si trovino nella stessa area di Azure, tranne in casi specifici motivati da ragioni aziendali o di design. È possibile spostare un'app del servizio app nella stessa area del database usando la [funzionalità di clonazione del servizio app](app-service-web-app-cloning.md) attualmente disponibile nel piano di servizio app Premium.   

## <a name="memoryresources"></a>Quando le app usano più memoria del previsto
Se si nota che un'app utilizza più memoria del previsto rispetto a quanto indicato dai servizi di monitoraggio e assistenza, prendere in considerazione l'uso della [funzionalità di correzione automatica del servizio app](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites). Una delle opzioni della funzionalità di correzione automatica consente di eseguire azioni personalizzate in base a una soglia di memoria. Le azioni includono notifiche tramite posta elettronica, analisi con dump di memoria e attenuazione locale tramite riciclo del processo di lavoro. La correzione automatica può essere configurata usando web.config e un'interfaccia utente semplice, come descritto in questo post di blog per l' [estensione del sito di supporto del servizio app](https://azure.microsoft.com/blog/additional-updates-to-support-site-extension-for-azure-app-service-web-apps).   

## <a name="CPUresources"></a>Quando le app usano più CPU del previsto
Quando si nota che un'app utilizza più CPU del previsto o presenta picchi di utilizzo della CPU ripetuti rispetto a quanto indicato dai servizi di monitoraggio e assistenza, può essere opportuno aumentare le prestazioni o il numero di istanze del piano di servizio app. Se si tratta di un'applicazione con stato, l'aumento delle prestazioni è l'unica opzione disponibile, mentre se l'applicazione è senza stato, l'aumento del numero di istanze offre maggiore flessibilità e un potenziale di scalabilità più elevato. 

Per altre informazioni di confronto tra le applicazioni "senza stato" e "con stato" è possibile guardare questo video: [Planning a Scalable End-to-End Multi-Tier Application on Azure App Service](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DEV-B414#fbid=?hashlink=fbid) (Pianificazione di un'applicazione su più livelli end-to-end scalabile in Servizio app di Azure). Per altre informazioni sulle opzioni di scalabilità e scalabilità automatica del servizio app, vedere [Aumentare le prestazioni di un'app Web nel servizio app di Azure](manage-scale-up.md).  

## <a name="socketresources"></a>Quando si esauriscono le risorse socket
Una causa comune dell'esaurimento delle connessioni TCP in uscita è l'impiego di librerie client non implementate per il riutilizzo delle connessioni TCP o il mancato uso di un protocollo di livello superiore, ad esempio keep-alive HTTP. Rivedere la documentazione di ogni libreria a cui fanno riferimento le app nel piano di servizio app per verificare che tali librerie siano configurate o accessibili nel codice per un efficiente riutilizzo delle connessioni in uscita. Seguire anche le indicazioni della documentazione delle librerie per le corrette operazioni di creazione, rilascio o pulizia per evitare la perdita di connessioni. Mentre è in corso l'esame delle librerie client, è possibile attenuare l'impatto sulle prestazioni aumentando il numero di istanze.

### <a name="nodejs-and-outgoing-http-requests"></a>Node.js e richieste HTTP in uscita
Quando si usa Node.js ed è presente un numero rilevante di richieste HTTP in uscita, è importante saper gestire il protocollo keep-alive HTTP. È possibile usare il pacchetto [agentkeepalive](https://www.npmjs.com/package/agentkeepalive) `npm` per semplificarne la gestione nel codice.

È necessario gestire sempre la risposta `http` anche se non si interviene nel gestore. Se la risposta non viene gestita correttamente, l'applicazione alla fine rimane bloccata a causa dell'esaurimento delle risorse socket.

Ad esempio, quando si usa il pacchetto `http` o `https`:

```javascript
const request = https.request(options, function(response) {
    response.on('data', function() { /* do nothing */ });
});
```

In caso di esecuzione del servizio app in Linux in un computer con più core, un'altra procedura consigliata prevede l'uso di PM2 per avviare più processi Node.js per eseguire l'applicazione. A tale scopo, specificare un comando di avvio nel contenitore.

Ad esempio, per avviare quattro istanze:

```
pm2 start /home/site/wwwroot/app.js --no-daemon -i 4
```

## <a name="appbackup"></a>Quando il backup dell'applicazione non viene più eseguito
Le due cause più comuni della mancata esecuzione del backup delle app sono: impostazioni di archiviazione non valide e configurazione del database non valida. In genere, questi errori si verificano in caso di modifiche alle risorse di archiviazione o di database o in caso di modifiche nella modalità di accesso a tali risorse, ad esempio quando vengono aggiornate le credenziali per il database selezionato nelle impostazioni di backup. I backup in genere sono eseguiti in base a una pianificazione e richiedono l'accesso alla risorsa di archiviazione (per l'output dei file di cui è stato eseguito il backup) e ai database (per copiare e leggere il contenuto da includere nel backup). Il risultato dell'incapacità di accedere a queste risorse è una mancata riuscita, costante, del backup. 

Quando si verificano errori di backup, esaminare i risultati più recenti per capire quale tipo di errore si è verificato. In caso di errori di accesso alla risorsa di archiviazione, controllare e aggiornare le impostazioni di archiviazione usate nella configurazione del backup. In caso di errori di accesso ai database, controllare e aggiornare le stringhe di connessione nelle impostazioni dell'applicazione. Procedere quindi ad aggiornare la configurazione del backup al fine di includere correttamente i necessari database. Per altre informazioni sul backup delle app, vedere [Eseguire il backup di un'app Web nel servizio app di Azure](manage-backup.md).

## <a name="nodejs"></a>Quando vengono distribuite nuove app Node.js nel Servizio app di Azure
La configurazione predefinita del Servizio app di Azure per le app Node.js mira a soddisfare al meglio le esigenze delle app più comuni. Se la configurazione dell'app Node.js può trarre vantaggio dall'ottimizzazione personalizzata per migliorare le prestazioni oppure per ottimizzare l'utilizzo delle risorse di rete, della memoria o della CPU, vedere [Procedure consigliate e guida alla risoluzione dei problemi per le applicazioni Node nel servizio app di Azure](app-service-web-nodejs-best-practices-and-troubleshoot-guide.md). Questo articolo descrive le impostazioni di iisnode che può essere necessario configurare per l'app Node.js, illustra i vari scenari o problemi possibili per l'app e indica come risolvere questi problemi.

