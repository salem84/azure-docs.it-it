---
title: Eventi pianificati per macchine virtuali Linux in Azure | Microsoft Docs
description: Pianificare eventi usando il servizio metadati di Azure per le macchine virtuali Linux.
services: virtual-machines-windows, virtual-machines-linux, cloud-services
documentationcenter: ''
author: ericrad
manager: gwallace
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2018
ms.author: ericrad
ms.openlocfilehash: d427544ab9396211e4cbb247527a0eb848f42926
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70091288"
---
# <a name="azure-metadata-service-scheduled-events-for-linux-vms"></a>Servizio metadati di Azure: Eventi pianificati per macchine virtuali Linux

Gli eventi pianificati sono un servizio metadati di Azure che concede all'applicazione il tempo di prepararsi alla manutenzione delle macchine virtuali. Questo servizio secondario offre informazioni sugli eventi di manutenzione imminenti (ad esempio il riavvio), in modo che l'applicazione possa prepararsi di conseguenza e limitare le interruzioni. È disponibile per tutti i tipi di macchine virtuali di Azure, incluse quelle di tipo PaaS e IaaS sia Windows che Linux. 

Per informazioni su Eventi pianificati in Windows, vedere [Eventi pianificati per macchine virtuali Windows](../windows/scheduled-events.md).

> [!Note] 
> Gli eventi pianificati sono disponibili a livello generale in tutte le aree di Azure. Per informazioni sulla versione più recente, vedere [Versione e disponibilità in base all'area geografica](#version-and-region-availability).

## <a name="why-use-scheduled-events"></a>Perché usare gli eventi pianificati

Per molte applicazioni è un vantaggio avere tempo per prepararsi alla manutenzione delle macchine virtuali. Questo tempo può essere impiegato in attività specifiche dell'applicazione per ottenere un miglioramento in termini di disponibilità, affidabilità e gestibilità, ad esempio: 

- Checkpoint e ripristino
- Esaurimento delle connessioni
- Failover della replica primaria
- Rimozione da un pool di bilanciamento del carico
- Registrazione eventi
- Arresto normale

Grazie agli eventi pianificati, l'applicazione è in grado di sapere quando verrà eseguita la manutenzione e di attivare attività specifiche per limitarne l'impatto.  

Gli eventi pianificati informano sugli eventi nei casi d'uso seguenti:

- [Manutenzione avviata dalla piattaforma](https://docs.microsoft.com/azure/virtual-machines/linux/maintenance-and-updates) (ad esempio, riavvio della macchina virtuale, migrazione in tempo reale o aggiornamenti con mantenimento della memoria per l'host)
- Hardware danneggiato
- Manutenzione avviata dall'utente, ad esempio il riavvio o la ridistribuzione di una macchina virtuale eseguita dall'utente
- [Rimozione di macchine virtuali con priorità bassa](https://azure.microsoft.com/blog/low-priority-scale-sets) nei set di scalabilità

## <a name="the-basics"></a>Nozioni di base  

  Il servizio metadati presenta informazioni sulle macchine virtuali in esecuzione usando un endpoint REST accessibile dall'interno della macchina virtuale. Le informazioni sono disponibili tramite un indirizzo IP non instradabile, in modo da non essere esposte al di fuori della macchina virtuale.

### <a name="scope"></a>Ambito
Gli eventi pianificati vengono recapitati a:

- Macchine virtuali autonome.
- Tutte le macchine virtuali in un servizio cloud.
- Tutte le macchine virtuali in un set di disponibilità.
- Tutte le macchine virtuali in un gruppo di posizionamento di un set di scalabilità. 

Di conseguenza, è opportuno controllare il campo `Resources` dell'evento per identificare quali macchine virtuali sono interessate.

### <a name="endpoint-discovery"></a>Individuazione degli endpoint
Per le macchine virtuali abilitate per le reti virtuali, il servizio metadati è disponibile da un indirizzo IP statico non instradabile, `169.254.169.254`. L'endpoint completo per la versione più recente degli eventi pianificati è: 

 > `http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01`

Se la macchina virtuale non viene creata all'interno di una rete virtuale, caso predefinito per i servizi cloud e le macchine virtuali classiche, è necessaria logica aggiuntiva per individuare l'indirizzo IP da usare. Per informazioni su come [individuare l'endpoint dell'host](https://github.com/azure-samples/virtual-machines-python-scheduled-events-discover-endpoint-for-non-vnet-vm), vedere l'esempio seguente.

### <a name="version-and-region-availability"></a>Versione e disponibilità in base all'area geografica
Il servizio eventi pianificati è un servizio con versione. Le versioni sono obbligatorie; la versione corrente è `2017-11-01`.

| Versione | Tipo di versione | Regioni | Note sulla versione | 
| - | - | - | - | 
| 2017-11-01 | Disponibilità generale | Tutti | <li> Aggiunta del supporto per la rimozione di una macchina virtuale con priorità bassa ' preemptive '<br> | 
| 2017-08-01 | Disponibilità generale | Tutti | <li> È stato rimosso il carattere di sottolineatura all'inizio dei nomi delle risorse per le macchine virtuali IaaS<br><li>Requisito dell'intestazione dei metadati applicato per tutte le richieste | 
| 2017-03-01 | Anteprima | Tutti | <li>Versione iniziale


> [!NOTE] 
> Le precedenti versioni di anteprima del servizio eventi pianificati supportano {latest} come api-version. Questo formato non è più supportato e verrà rimosso in futuro.

### <a name="enabling-and-disabling-scheduled-events"></a>Abilitazione e disabilitazione degli eventi pianificati
Gli eventi pianificati vengono abilitati per il servizio la prima volta che si effettua una richiesta di eventi. La prima chiamata potrebbe ricevere una risposta con un ritardo massimo di due minuti.

Gli eventi pianificati vengono disabilitati per il servizio se questo non effettua una richiesta per 24 ore.

### <a name="user-initiated-maintenance"></a>Manutenzione avviata dall'utente
La manutenzione delle macchine virtuali avviata dall'utente tramite il portale, l'API o l'interfaccia della riga di comando di Azure oppure tramite PowerShell restituisce un evento pianificato. È quindi possibile testare la logica di preparazione della manutenzione dell'applicazione e permettere all'applicazione di prepararsi per la manutenzione avviata dall'utente.

Se si riavvia una macchina virtuale, viene pianificato un evento di tipo `Reboot`. Se si ridistribuisce una macchina virtuale, viene pianificato un evento di tipo `Redeploy`.

## <a name="use-the-api"></a>Usare l'API

### <a name="headers"></a>Intestazioni
Quando si eseguono query sul servizio metadati, è necessario specificare l'intestazione `Metadata:true` per garantire che la richiesta non sia stata reindirizzata accidentalmente. L'intestazione `Metadata:true` è obbligatoria per tutte le richieste di eventi pianificati. Se non si include l'intestazione nella richiesta, il servizio metadati restituisce una risposta di richiesta non valida.

### <a name="query-for-events"></a>Query per gli eventi
È possibile eseguire query per eventi pianificati eseguendo la chiamata seguente:

#### <a name="bash"></a>Bash
```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2017-08-01
```

Una risposta contiene una serie di eventi pianificati. Una matrice vuota indica che al momento non sono presenti eventi pianificati.
Nel caso in cui siano presenti eventi pianificati, la risposta contiene una matrice di eventi. 
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Reboot" | "Redeploy" | "Freeze" | "Preempt",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled" | "Started",
            "NotBefore": {timeInUTC},              
        }
    ]
}
```

### <a name="event-properties"></a>Proprietà dell'evento
|Proprietà  |  Descrizione |
| - | - |
| EventId | Identificatore globalmente univoco per l'evento. <br><br> Esempio: <br><ul><li>602d9444-d2cd-49c7-8624-8643e7171297  |
| EventType | Impatto che l'evento causa. <br><br> Valori: <br><ul><li> `Freeze`: È prevista la sospensione della macchina virtuale per alcuni secondi. La connettività CPU e di rete potrebbe essere sospesa, ma non si verifica alcun effetto sulla memoria o sui file aperti.<li>`Reboot`: per la macchina virtuale è pianificato un riavvio (la memoria non permanente andrà persa). <li>`Redeploy`: per la macchina virtuale è pianificato uno spostamento in un altro nodo (i dischi temporanei andranno persi). <li>`Preempt`: È in corso l'eliminazione della macchina virtuale con priorità bassa. i dischi temporanei andranno perduti.|
| ResourceType | Tipo di risorsa interessata dall'evento. <br><br> Valori: <ul><li>`VirtualMachine`|
| Risorse| Elenco di risorse interessate dall'evento. L'elenco contiene sicuramente i computer al massimo di un [dominio di aggiornamento](manage-availability.md), ma potrebbe non contenere tutti i computer del dominio. <br><br> Esempio: <br><ul><li> ["FrontEnd_IN_0", "BackEnd_IN_0"] |
| EventStatus | Stato dell'evento. <br><br> Valori: <ul><li>`Scheduled`: l'avvio dell'evento è pianificato dopo la data e l'ora specificate nella proprietà `NotBefore`.<li>`Started`: l'evento è stato avviato.</ul> Lo stato `Completed` o simile non viene mai restituito. Al termine dell'evento, quest'ultimo non viene più restituito.
| NotBefore| Tempo al termine del quale l'evento può essere avviato. <br><br> Esempio: <br><ul><li> Lun 19 set 2016 18:29:47 GMT  |

### <a name="event-scheduling"></a>Event Scheduling
Ogni evento è pianificato con un ritardo minimo che dipende dal tipo di evento. Questo tempo si riflette in una proprietà `NotBefore` dell'evento. 

|Tipo di evento  | Preavviso minimo |
| - | - |
| Freeze| 15 minuti |
| Riavvio | 15 minuti |
| Ripetere la distribuzione | 10 minuti |
| Hanno | 30 secondi |

### <a name="start-an-event"></a>Avviare un evento 

Dopo aver ricevuto l'avviso di un evento imminente e aver completato la logica per l'arresto normale, è possibile approvare l'evento in sospeso eseguendo una chiamata `POST` al servizio metadati con `EventId`. Questa chiamata segnala ad Azure che è possibile ridurre il tempo minimo di notifica, quando possibile. 

Di seguito è riportato l'esempio JSON con il codice previsto nel corpo della richiesta `POST`. La richiesta deve contenere un elenco di `StartRequests`. Ogni `StartRequest` contiene `EventId` per l'evento che si vuole velocizzare:
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

#### <a name="bash-sample"></a>Esempio Bash
```
curl -H Metadata:true -X POST -d '{"StartRequests": [{"EventId": "f020ba2e-3bc0-4c40-a10b-86575a9eabd5"}]}' http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01
```

> [!NOTE] 
> Il riconoscimento consente a un evento di procedere per tutti gli elementi `Resources` dell'evento, non solo per la macchina virtuale che riconosce l'evento. Di conseguenza, è possibile scegliere un coordinatore che si occupi del riconoscimento, che potrebbe essere semplicemente il primo computer elencato nel campo `Resources`.

## <a name="python-sample"></a>Esempio Python 

L'esempio seguente esegue query sul servizio metadati per gli eventi pianificati e approva ogni evento in sospeso:

```python
#!/usr/bin/python

import json
import urllib2
import socket
import sys

metadata_url = "http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01"
headers = "{Metadata:true}"
this_host = socket.gethostname()

def get_scheduled_events():
   req = urllib2.Request(metadata_url)
   req.add_header('Metadata', 'true')
   resp = urllib2.urlopen(req)
   data = json.loads(resp.read())
   return data

def handle_scheduled_events(data):
    for evt in data['Events']:
        eventid = evt['EventId']
        status = evt['EventStatus']
        resources = evt['Resources']
        eventtype = evt['EventType']
        resourcetype = evt['ResourceType']
        notbefore = evt['NotBefore'].replace(" ","_")
        if this_host in resources:
            print "+ Scheduled Event. This host " + this_host + " is scheduled for " + eventtype + " not before " + notbefore
            # Add logic for handling events here


def main():
   data = get_scheduled_events()
   handle_scheduled_events(data)

if __name__ == '__main__':
  main()
  sys.exit(0)
```

## <a name="next-steps"></a>Passaggi successivi 
- È disponibile una demo sugli [eventi pianificati in Azure Friday](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance). 
- È possibile esaminare gli esempi di codice di Eventi pianificati nel [repository GitHub di Eventi pianificati del servizio metadati dell'istanza di Azure](https://github.com/Azure-Samples/virtual-machines-scheduled-events-discover-endpoint-for-non-vnet-vm).
- Altre informazioni sulle API disponibili nel [servizio metadati dell'istanza](instance-metadata-service.md).
- Informazioni sulla [manutenzione pianificata per le macchine virtuali Linux in Azure](planned-maintenance.md).
