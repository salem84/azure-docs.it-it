---
title: Come usare gli argomenti del bus di servizio con PHP | Documentazione Microsoft
description: Informazioni su come usare gli argomenti del bus di servizio con PHP in Azure.
services: service-bus-messaging
documentationcenter: php
author: axisc
manager: timlt
editor: spelluru
ms.assetid: faaa4bbd-f6ef-42ff-aca7-fc4353976449
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 04/15/2019
ms.author: aschhab
ms.openlocfilehash: eba2c0aeb37f2bc2283e7afb108bb4578981120e
ms.sourcegitcommit: b03516d245c90bca8ffac59eb1db522a098fb5e4
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/19/2019
ms.locfileid: "71147230"
---
# <a name="how-to-use-service-bus-topics-and-subscriptions-with-php"></a>Come usare gli argomenti e le sottoscrizioni del bus di servizio con PHP

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

In questo articolo verrà descritto come usare gli argomenti e le sottoscrizioni del bus di servizio. Gli esempi sono scritti in PHP e usano [Azure SDK per PHP](https://github.com/Azure/azure-sdk-for-php). Gli scenari trattati includono:

- Creazione di argomenti e sottoscrizioni 
- Creazione di filtri di sottoscrizione 
- Invio di messaggi a un argomento 
- Ricezione di messaggi da una sottoscrizione
- Eliminazione di argomenti e sottoscrizioni

## <a name="prerequisites"></a>Prerequisiti
1. Una sottoscrizione di Azure. Per completare l'esercitazione, è necessario un account Azure. È possibile attivare i [benefici per gli abbonati di Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) oppure iscriversi per ottenere un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Seguire i [passaggi nella Guida introduttiva: Usare il portale di Azure per creare un argomento del bus di servizio e le sottoscrizioni dell'argomento](service-bus-quickstart-topics-subscriptions-portal.md) per creare uno **spazio dei nomi** del bus di servizio e ottenere la stringa di **connessione**.

    > [!NOTE]
    > In questa Guida introduttiva si creerà un **argomento** e una **sottoscrizione** all'argomento usando **php** . 

## <a name="create-a-php-application"></a>Creare un'applicazione PHP
Per creare un'applicazione PHP che acceda al servizio BLOB di Azure, è sufficiente fare riferimento alle classi in [Azure SDK per PHP](https://github.com/Azure/azure-sdk-for-php) dall'interno del codice. Per creare l'applicazione, è possibile usare qualsiasi strumento di sviluppo o il Blocco note.

> [!NOTE]
> L'installazione di PHP deve avere anche l'[estensione OpenSSL](https://php.net/openssl) installata e abilitata.
> 
> 

Questo articolo illustra come usare le funzionalità del servizio che possono essere chiamate in un'applicazione PHP in locale o nel codice in esecuzione in un ruolo Web, in un ruolo di lavoro o in un sito Web di Azure.

## <a name="get-the-azure-client-libraries"></a>Acquisire le librerie client di Azure

### <a name="install-via-composer"></a>Installazione tramite Composer
1. Creare un file denominato **composer.json** nella radice del progetto, quindi aggiungere nel file il codice seguente:
   
    ```json
    {
      "require": {
        "microsoft/windowsazure": "*"
      }
    }
    ```
2. Scaricare **[Composer. phar] [Composer-Phar]** nella radice del progetto.
3. Aprire un prompt dei comandi ed eseguire il comando seguente nella radice del progetto
   
    ```
    php composer.phar install
    ```

## <a name="configure-your-application-to-use-service-bus"></a>Configurare l'applicazione per l'uso del bus di servizio
Per usare le API del bus di servizio:

1. Fare riferimento al file autoloader mediante l'istruzione [require_once][require-once] .
2. Fare riferimento a tutte le eventuali classi utilizzabili.

Nell'esempio seguente viene indicato come includere il file autoloader e fare riferimento alla classe **ServiceBusService**.

> [!NOTE]
> Questo esempio (e in altri esempi in questo articolo) presuppone che siano state installate le librerie client PHP per Azure tramite Composer. Se le librerie sono state installate manualmente o come pacchetto PEAR, è necessario fare riferimento al file autoloader **WindowsAzure.php**.
> 
> 

```php
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

Negli esempi seguenti l'istruzione `require_once` viene sempre visualizzata, ma si fa riferimento solo alle classi necessarie per eseguire l'esempio.

## <a name="set-up-a-service-bus-connection"></a>Configurare una stringa di connessione per il bus di servizio
Per creare un'istanza di un client del bus di servizio, è necessario innanzitutto disporre di una stringa di connessione valida nel seguente formato:

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

Dove `Endpoint` è in genere nel formato `https://[yourNamespace].servicebus.windows.net`.

Per creare un client del servizio Azure, è necessario usare la classe `ServicesBuilder`. È possibile:

* Passare la stringa di connessione direttamente.
* utilizzare **CloudConfigurationManager (CCM)** per cercare la stringa di connessione in più origini esterne:
  * Per impostazione predefinita viene fornito con il supporto per un'origine esterna, ovvero le variabili ambientali.
  * È possibile aggiungere nuove origini estendendo la classe `ConnectionStringSource`.

Per gli esempi illustrati in questo articolo, la stringa di connessione viene passata direttamente.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-topic"></a>Crea un argomento
È possibile eseguire operazioni di gestione per gli argomenti del bus di servizio usando la classe `ServiceBusRestProxy`. Un oggetto `ServiceBusRestProxy` è costruito tramite il metodo factory `ServicesBuilder::createServiceBusService` con una stringa di connessione appropriata che incapsula le autorizzazioni di token per gestirlo.

L'esempio seguente mostra come creare un'istanza di `ServiceBusRestProxy` e chiamare il metodo `ServiceBusRestProxy->createTopic` per creare un argomento denominato `mytopic` in uno spazio dei nomi `MySBNamespace`:

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\TopicInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create topic.
    $topicInfo = new TopicInfo("mytopic");
    $serviceBusRestProxy->createTopic($topicInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> È possibile usare il metodo `listTopics` negli oggetti `ServiceBusRestProxy` per verificare se in uno spazio dei nomi del servizio esiste già un argomento con il nome specificato.
> 
> 

## <a name="create-a-subscription"></a>Crea una sottoscrizione
È possibile creare le sottoscrizioni di un argomento con il metodo `ServiceBusRestProxy->createSubscription`. Le sottoscrizioni sono denominate e possono includere un filtro facoltativo che limita l'insieme dei messaggi passati alla coda virtuale della sottoscrizione.

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>Creare una sottoscrizione con il filtro (MatchAll) predefinito
Se non vengono specificati altri filtri durante la creazione di una nuova sottoscrizione, viene usato il filtro (predefinito) **MatchAll**. Quando si usa il filtro **MatchAll**, tutti i messaggi pubblicati nell'argomento vengono inseriti nella coda virtuale della sottoscrizione. L'esempio seguente crea una sottoscrizione denominata `mysubscription` e usa il filtro **MatchAll** predefinito.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\SubscriptionInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create subscription.
    $subscriptionInfo = new SubscriptionInfo("mysubscription");
    $serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

### <a name="create-subscriptions-with-filters"></a>Creare sottoscrizioni con i filtri
È anche possibile configurare filtri che consentono di specificare quali messaggi inviati a un argomento devono essere presenti in una specifica sottoscrizione dell'argomento. Il tipo di filtro più flessibile tra quelli supportati dalle sottoscrizioni è [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter), che implementa un subset di SQL92. I filtri SQL agiscono sulle proprietà dei messaggi pubblicati nell'argomento. Per altre informazioni su sqlfilters, vedere [Proprietà SqlFilter. SqlExpression][sqlfilter].

> [!NOTE]
> Ogni regola in una sottoscrizione elabora i messaggi in arrivo indipendentemente, aggiungendo i messaggi risultanti alla sottoscrizione stessa. Ogni nuova sottoscrizione presenta anche un oggetto **Rule** predefinito con un filtro che aggiunge tutti i messaggi dall'argomento alla sottoscrizione. Per ricevere solo messaggi corrispondenti al filtro in uso, è necessario rimuovere la regola predefinita. È possibile rimuovere la regola predefinita usando il metodo `ServiceBusRestProxy->deleteRule`.
> 
> 

L'esempio seguente crea una sottoscrizione denominata `HighMessages` con un filtro **SqlFilter** che seleziona solo i messaggi in cui il valore della proprietà `MessageNumber` personalizzata è maggiore di 3. Per informazioni su come aggiungere proprietà personalizzate ai messaggi, vedere [Inviare messaggi a un argomento](#send-messages-to-a-topic).

```php
$subscriptionInfo = new SubscriptionInfo("HighMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "HighMessages", '$Default');

$ruleInfo = new RuleInfo("HighMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber > 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "HighMessages", $ruleInfo);
```

Questo codice richiede l'uso di uno spazio dei nomi aggiuntivo: `WindowsAzure\ServiceBus\Models\SubscriptionInfo`.

Analogamente, l'esempio seguente crea una sottoscrizione denominata `LowMessages` con un filtro `SqlFilter` che seleziona solo i messaggi in cui il valore della proprietà `MessageNumber` è minore o uguale a 3.

```php
$subscriptionInfo = new SubscriptionInfo("LowMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "LowMessages", '$Default');

$ruleInfo = new RuleInfo("LowMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber <= 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "LowMessages", $ruleInfo);
```

A questo punto, quando viene inviato un messaggio all'argomento `mytopic`, viene sempre recapitato ai ricevitori con sottoscrizione `mysubscription` e recapitato selettivamente ai ricevitori con sottoscrizioni `HighMessages` e `LowMessages`, a seconda del contenuto del messaggio.

## <a name="send-messages-to-a-topic"></a>Inviare messaggi a un argomento
Per inviare un messaggio a un argomento del bus di servizio, l'applicazione chiama il metodo `ServiceBusRestProxy->sendTopicMessage`. Il codice seguente illustra come inviare un messaggio all'argomento `mytopic` creato prima nello spazio dei nomi del servizio `MySBNamespace`.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

I messaggi inviati ad argomenti del bus di servizio sono istanze della classe [BrokeredMessage][BrokeredMessage]. Gli oggetti [BrokeredMessage][BrokeredMessage] includono un set di proprietà e metodi standard, nonché proprietà che possono essere usate per contenere proprietà personalizzate specifiche dell'applicazione. L'esempio seguente illustra come inviare cinque messaggi di prova all'argomento `mytopic` creato in precedenza. Il metodo `setProperty` viene usato per aggiungere una proprietà personalizzata (`MessageNumber`) a ogni messaggio. Il valore della proprietà `MessageNumber` varia per ogni messaggio, consentendo di determinare quali sottoscrizioni lo riceveranno, come descritto nella sezione [Creare una sottoscrizione](#create-a-subscription):

```php
for($i = 0; $i < 5; $i++){
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message ".$i);

    // Set custom property.
    $message->setProperty("MessageNumber", $i);

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
```

Gli argomenti del bus di servizio supportano messaggi di dimensioni massime fino a 256 KB nel [livello Standard](service-bus-premium-messaging.md) e fino a 1 MB nel [livello Premium](service-bus-premium-messaging.md). Le dimensioni massime dell'intestazione, che include le proprietà standard e personalizzate dell'applicazione, non possono superare 64 KB. Non esiste alcun limite al numero di messaggi mantenuti in un argomento, mentre è prevista una limitazione alla dimensione totale dei messaggi di un argomento. Il limite massimo delle dimensioni di un argomento è 5 GB. Per altre informazioni sulle quote, vedere [Quote del bus di servizio][Service Bus quotas].

## <a name="receive-messages-from-a-subscription"></a>Ricevere messaggi da una sottoscrizione
Il modo ideale per ricevere i messaggi da una sottoscrizione prevede l'uso di un metodo `ServiceBusRestProxy->receiveSubscriptionMessage`. I messaggi possono essere ricevuti in due modalità diverse: [*ReceiveAndDelete* e *PeekLock*](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode). **PeekLock** è la modalità predefinita.

Quando si usa la modalità [ReceiveAndDelete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode), l'operazione di ricezione viene eseguita in un'unica fase. Quando infatti il bus di servizio riceve una richiesta di lettura relativa a un messaggio in una sottoscrizione, lo contrassegna come usato e lo restituisce all'applicazione. La modalità [ReceiveAndDelete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) rappresenta il modello più semplice ed è adatta per scenari in cui un'applicazione può tollerare la mancata elaborazione di un messaggio in caso di errore. Per comprendere meglio questo meccanismo, si consideri uno scenario in cui il consumer invia la richiesta di ricezione e viene arrestato in modo anomalo prima dell'elaborazione. Poiché il bus di servizio ha contrassegnato il messaggio come utilizzato, quando l'applicazione viene riavviata e inizia a usare nuovamente i messaggi, il messaggio usato prima dell'arresto anomalo del sistema risulta perso.

Nella modalità [PeekLock](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) predefinita l'operazione di ricezione viene suddivisa in due fasi, in modo da consentire il supporto di applicazioni che non possono tollerare messaggi mancanti. Quando il bus di servizio riceve una richiesta, individua il messaggio successivo da usare, lo blocca per impedirne la ricezione da parte di altri consumer e quindi lo restituisce all'applicazione. Dopo aver elaborato il messaggio, o averlo archiviato in modo affidabile per una successiva elaborazione, l'applicazione completa la seconda fase del processo di ricezione passando il messaggio ricevuto a `ServiceBusRestProxy->deleteMessage`. Quando il bus di servizio vede la chiamata `deleteMessage`, contrassegna il messaggio come utilizzato e lo rimuove dalla coda.

L'esempio seguente illustra come ricevere ed elaborare un messaggio usando la modalità [PeekLock](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) (predefinita). 

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Set receive mode to PeekLock (default is ReceiveAndDelete)
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Get message.
    $message = $serviceBusRestProxy->receiveSubscriptionMessage("mytopic", "mysubscription", $options);

    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Deleting message...<br />";
    $serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Procedura: Gestire arresti anomali e messaggi illeggibili dell'applicazione
Il bus di servizio fornisce funzionalità per il ripristino gestito automaticamente in caso di errori nell'applicazione o di problemi di elaborazione di un messaggio. Se un'applicazione ricevente non è in grado di elaborare il messaggio per un qualsiasi motivo, può chiamare il metodo `unlockMessage` anziché `deleteMessage` sul messaggio ricevuto. In questo modo, il bus di servizio sblocca il messaggio nella coda rendendolo nuovamente disponibile per la ricezione da parte della stessa o di un'altra applicazione consumer.

Al messaggio bloccato nella coda è anche associato un timeout. Se l'applicazione non riesce a elaborare il messaggio prima della scadenza del timeout, ad esempio a causa di un arresto anomalo, il bus di servizio sbloccherà automaticamente il messaggio rendendolo nuovamente disponibile per la ricezione.

In caso di arresto anomalo dell'applicazione dopo l'elaborazione del messaggio, ma prima dell'invio della richiesta `deleteMessage`, il messaggio viene nuovamente recapitato all'applicazione al riavvio del sistema. Questo tipo di elaborazione viene spesso definita elaborazione *At-Least-Once*, per indicare che ogni messaggio verrà elaborato almeno una volta ma che in determinate situazioni potrà essere recapitato una seconda volta. Se lo scenario non tollera la doppia elaborazione, gli sviluppatori dovranno aggiungere logica aggiuntiva alle applicazioni per gestire il secondo recapito del messaggio. Viene spesso usato il metodo `getMessageId` del messaggio, che rimane costante in tutti i tentativi di recapito.

## <a name="delete-topics-and-subscriptions"></a>Eliminare argomenti e sottoscrizioni
Per eliminare un argomento o una sottoscrizione, usare rispettivamente il metodo `ServiceBusRestProxy->deleteTopic` o `ServiceBusRestProxy->deleteSubscripton`. Se si elimina un argomento, vengono eliminate anche tutte le sottoscrizioni registrate con l'argomento.

L'esempio seguente illustra come eliminare un argomento denominato `mytopic` e le relative sottoscrizioni registrate.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\ServiceBus\ServiceBusService;
use WindowsAzure\ServiceBus\ServiceBusSettings;
use WindowsAzure\Common\ServiceException;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Delete topic.
    $serviceBusRestProxy->deleteTopic("mytopic");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

È possibile eliminare una sottoscrizione in modo indipendente usando il metodo `deleteSubscription`:

```php
$serviceBusRestProxy->deleteSubscription("mytopic", "mysubscription");
```

> [!NOTE]
> È possibile gestire le risorse del bus di servizio con [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/). Service Bus Explorer consente agli utenti di connettersi a uno spazio dei nomi del bus di servizio e di amministrare le entità di messaggistica in modo semplice. Lo strumento offre caratteristiche avanzate, tra cui funzionalità di importazione/esportazione o la possibilità di testare argomenti, code, sottoscrizioni, servizi di inoltro, hub di notifica e hub eventi. 

## <a name="next-steps"></a>Passaggi successivi
Per ulteriori informazioni, vedere [code, argomenti e sottoscrizioni][Queues, topics, and subscriptions].

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[sqlfilter]: /dotnet/api/microsoft.servicebus.messaging.sqlfilter
[require-once]: https://php.net/require_once
[Service Bus quotas]: service-bus-quotas.md
