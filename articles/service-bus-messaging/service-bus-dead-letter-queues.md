---
title: Code dei messaggi non recapitabili del bus di servizio | Documentazione Microsoft
description: Panoramica delle code dei messaggi non recapitabili del bus di servizio Azure
services: service-bus-messaging
documentationcenter: .net
author: axisc
manager: timlt
editor: spelluru
ms.assetid: 68b2aa38-dba7-491a-9c26-0289bc15d397
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/21/2019
ms.author: aschhab
ms.openlocfilehash: afa2e6e46579d9ce2906e2686cf40adf4b65ab2b
ms.sourcegitcommit: f5cc71cbb9969c681a991aa4a39f1120571a6c2e
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/26/2019
ms.locfileid: "68516587"
---
# <a name="overview-of-service-bus-dead-letter-queues"></a>Panoramica delle code dei messaggi non recapitabili del bus di servizio

Le code del bus di servizio di Azure e le sottoscrizioni dell'argomento includono una coda secondaria chiamata *coda di messaggi non recapitabili* (DLQ, Dead-Letter Queue). Non è necessario creare in modo esplicito la coda dei messaggi non recapitabili, che, tra l'altro, non può essere eliminata né altrimenti gestita indipendentemente dall'entità principale.

Questo articolo descrive le code dei messaggi non recapitabili nel bus di servizio. Questo argomento viene in gran parte illustrato nell'[esempio relativo alle code di messaggi non recapitabili](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/DeadletterQueue) su GitHub.
 
## <a name="the-dead-letter-queue"></a>Coda di messaggi non recapitabili

Lo scopo della coda dei messaggi non recapitabili è conservare i messaggi che non possono essere recapitati ai ricevitori o che non possono essere elaborati. I messaggi possono essere rimossi dalla coda e verificati. Con l'aiuto di un operatore, un'applicazione potrebbe correggere i problemi e inviare nuovamente il messaggio, registrare la notizia che si è verificato un errore e intraprendere azioni correttive. 

Dal punto di vista di API e protocolli, la coda DLQ è molto simile a qualsiasi altra coda, ad eccezione del fatto che i messaggi possono essere inviati ad essa solo tramite l'operazione messaggi non recapitabili dell'entità padre. Inoltre, il parametro time-to-live non viene rispettato e non è possibile impostare come non recapitabile un messaggio di una coda DLQ. La coda dei messaggi non recapitabili supporta completamente il recapito con blocco di visualizzazione e le operazioni transazionali.

Si noti che non è prevista alcuna pulizia automatica della coda. I messaggi rimangono nella coda fino a quando non vengono esplicitamente recuperati e non si chiama il metodo [Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) sul messaggio non recapitabile.

## <a name="moving-messages-to-the-dlq"></a>Spostare messaggi nella coda DLQ

Nel bus di servizio sono presenti diverse attività che comportano l'inserimento di messaggi nella coda DLQ dall'interno del motore di messaggistica stesso. Un'applicazione può anche spostare in modo esplicito i messaggi nella coda dei messaggi non recapitabili. 

Quando un messaggio viene spostato dal broker, vengono aggiunte al messaggio due proprietà, `DeadLetterReason` e `DeadLetterErrorDescription`, nel momento in cui il broker chiama la sua versione interna del metodo [DeadLetter](/dotnet/api/microsoft.azure.servicebus.queueclient.deadletterasync) sul messaggio.

Le applicazioni possono definire i propri codici per la proprietà `DeadLetterReason`, ma il sistema imposta i valori seguenti.

| Condizione | DeadLetterReason | DeadLetterErrorDescription |
| --- | --- | --- |
| Sempre |HeaderSizeExceeded |È stata superata la dimensione del flusso. |
| !TopicDescription.<br />EnableFilteringMessagesBeforePublishing e SubscriptionDescription.<br />EnableDeadLetteringOnFilterEvaluationExceptions |exception.GetType().Name |exception.Message |
| EnableDeadLetteringOnMessageExpiration |TTLExpiredException |Il messaggio è scaduto ed è stato configurato come non recapitabile. |
| SubscriptionDescription.RequiresSession |L'ID sessione ha valore null. |L'entità attivata dalla sessione non consente il recapito di un messaggio il cui identificatore di sessione è null. |
| !dead letter queue | MaxTransferHopCountExceeded | Numero massimo di hop consentiti durante l'invio tra le code. Il valore è impostato su 4. |
| Configurazione esplicita di messaggio non recapitabile da parte dell'applicazione |Specificato dall'applicazione |Specificato dall'applicazione |

## <a name="exceeding-maxdeliverycount"></a>Superamento di MaxDeliveryCount

Le code e le sottoscrizioni hanno ognuna una proprietà [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) e [SubscriptionDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.maxdeliverycount). Il valore predefinito è 10. Ogni volta che un messaggio viene recapitato in un blocco ([ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode)) ma viene abbandonato in modo esplicito oppure il blocco è scaduto, il valore [BrokeredMessage.DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) del messaggio viene incrementato. Quando il valore [DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) supera [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount), il messaggio viene spostato nella coda DLQ con il codice motivo `MaxDeliveryCountExceeded`.

Non è possibile disattivare questo comportamento, ma è possibile impostare [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) su un numero molto elevato.

## <a name="exceeding-timetolive"></a>Superamento di TimeToLive

Quando la proprietà [QueueDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.queuedescription) o [SubscriptionDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription) viene impostata su **true** (il valore predefinito è **false**), tutti i messaggi in scadenza vengono spostati nella coda DLQ con il codice motivo `TTLExpiredException`.

Si noti che i messaggi scaduti vengono ripuliti e spostati nella coda di messaggi non recapitabili solo quando esiste almeno un ricevitore attivo che effettua il pull dalla sottoscrizione o dalla coda principale. Tale comportamento è predefinito.

## <a name="errors-while-processing-subscription-rules"></a>Errori durante l'elaborazione di regole di sottoscrizione

Quando la proprietà [SubscriptionDescription.EnableDeadLetteringOnFilterEvaluationExceptions](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription) di una sottoscrizione è abilitata, qualsiasi errore si verifichi durante l'esecuzione di una regola di filtro SQL di una sottoscrizione viene acquisito nella coda DLQ con il messaggio che indica l'errore.

## <a name="application-level-dead-lettering"></a>Definizione di messaggi non recapitabili a livello di applicazione

Oltre alle funzionalità di definizione dei messaggi non recapitabili del sistema, le applicazioni possono usare la coda DLQ per rifiutare esplicitamente i messaggi inaccettabili. Ciò può riguardare i messaggi che non possono essere elaborati correttamente a causa diversi problemi del sistema, i messaggi contenenti payload in formato non valido o che non superino il processo di autenticazione quando viene utilizzato un schema di sicurezza a livello di messaggio.

## <a name="dead-lettering-in-forwardto-or-sendvia-scenarios"></a>Messaggi non recapitabili negli scenari ForwardTo o SendVia

I messaggi verranno inviati nella coda dei messaggi non recapitabili di trasferimento nelle condizioni seguenti:

- Un messaggio passa attraverso più di 4 code o argomenti che sono [concatenati](service-bus-auto-forwarding.md).
- L'argomento o la coda di destinazione è disattivato o eliminato.
- L'argomento o la coda di destinazione supera le dimensioni massime dell'entità.

Per recuperare questi messaggi non recapitabili, è possibile creare un destinatario usando il metodo di utilità [FormatTransferDeadletterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formattransferdeadletterpath).

## <a name="example"></a>Esempio

Il frammento di codice seguente crea un ricevitore del messaggio. Nel ciclo di ricezione della coda principale, il codice recupera il messaggio con [Receive(TimeSpan.Zero)](/dotnet/api/microsoft.servicebus.messaging.messagereceiver), che richiede al broker di restituire immediatamente i messaggi disponibili o di restituire nessun risultato. Se riceve un messaggio, il codice lo abbandona immediatamente, incrementando `DeliveryCount`. Dopo che il sistema ha spostato il messaggio alla coda DLQ, la coda principale rimane vuota e il ciclo viene interrotto, perché [ReceiveAsync](/dotnet/api/microsoft.servicebus.messaging.messagereceiver) restituisce **null**.

```csharp
var receiver = await receiverFactory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);
while(true)
{
    var msg = await receiver.ReceiveAsync(TimeSpan.Zero);
    if (msg != null)
    {
        Console.WriteLine("Picked up message; DeliveryCount {0}", msg.DeliveryCount);
        await msg.AbandonAsync();
    }
    else
    {
        break;
    }
}
```

## <a name="path-to-the-dead-letter-queue"></a>Percorso della coda dei messaggi non recapitabili
È possibile accedere alla coda dei messaggi non recapitabili utilizzando la sintassi seguente:

```
<queue path>/$deadletterqueue
<topic path>/Subscriptions/<subscription path>/$deadletterqueue
```

Se si usa .NET SDK, è possibile ottenere il percorso della coda dei messaggi non recapitabili usando il Metodo SubscriptionClient. FormatDeadLetterPath (). Questo metodo accetta il nome dell'argomento e il nome della sottoscrizione e i suffissi con **/$DeadLetterQueue**.


## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sulle code del bus di servizio, vedere gli articoli seguenti:

* [Introduzione alle code del bus di servizio](service-bus-dotnet-get-started-with-queues.md)
* [Analogie e differenze tra le code di Azure e le code del bus di servizio](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

