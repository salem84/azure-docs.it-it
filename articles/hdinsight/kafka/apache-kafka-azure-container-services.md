---
title: Usare il servizio Azure Kubernetes con Kafka in HDInsight
description: Informazioni su come usare Kafka in HDInsight dalle immagini del contenitore ospitate nel servizio Azure Kubernetes.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/07/2018
ms.openlocfilehash: 31eefbad8e8d7cb626d87d53690388d09b85257e
ms.sourcegitcommit: fad368d47a83dadc85523d86126941c1250b14e2
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/19/2019
ms.locfileid: "71122651"
---
# <a name="use-azure-kubernetes-service-with-apache-kafka-on-hdinsight"></a>Usare il servizio Azure Kubernetes con Apache Kafka in HDInsight

Informazioni su come usare il servizio Azure Kubernetes con [Apache Kafka](https://kafka.apache.org/) nel cluster HDInsight. I passaggi illustrati in questo documento usano un'applicazione Node.js ospitata nel servizio Azure Container per verificare la connettività con Kafka. Questa applicazione usa il pacchetto [kafka-node](https://www.npmjs.com/package/kafka-node) per comunicare con Kafka. Usa [Socket.io](https://socket.io/) per la messaggistica basata su eventi tra il client browser e il back-end ospitato nel servizio Azure Container.

[Apache Kafka](https://kafka.apache.org) è una piattaforma di streaming open source distribuita che può essere usata per compilare applicazioni e pipeline di dati in streaming in tempo reale. Il servizio Azure Kubernetes gestisce l'ambiente Kubernetes ospitato e semplifica e velocizza la distribuzione delle applicazioni in contenitori. Usando una rete virtuale di Azure, è possibile connettere i due servizi.

> [!NOTE]  
> Questo documento è incentrato sui passaggi necessari per consentire al servizio Azure Kubernetes di comunicare con Kafka in HDInsight. L'esempio in sé è solo un client Kafka di base con cui si dimostra che la configurazione funziona.

## <a name="prerequisites"></a>Prerequisiti

* [Interfaccia della riga di comando di Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
* Una sottoscrizione di Azure.

Questo documento presuppone che si abbia familiarità con la creazione e l'uso dei servizi di Azure seguenti:

* Kafka in HDInsight
* Servizio Azure Kubernetes
* Reti virtuali di Azure

Questo documento presuppone anche che sia stata eseguita l'[esercitazione sul servizio Azure Kubernetes](../../aks/tutorial-kubernetes-prepare-app.md). In questo articolo viene creato un servizio contenitore, viene creato un cluster Kubernetes, un registro contenitori e viene `kubectl` configurata l'utilità.

## <a name="architecture"></a>Architettura

### <a name="network-topology"></a>Topologia di rete

Sia HDInsight che il servizio Azure Container usano una rete virtuale di Azure come contenitore per le risorse di calcolo. Per abilitare la comunicazione tra HDInsight e il servizio Azure Container, è necessario abilitare la comunicazione tra le rispettive reti. I passaggi illustrati in questo documento usano il peering reti virtuali per le reti. Funzioneranno anche altre connessioni, ad esempio VPN. Per altre informazioni sul peering, vedere il documento [Peering di rete virtuale](../../virtual-network/virtual-network-peering-overview.md).

Il diagramma seguente illustra la topologia di rete usata in questo documento:

![HDInsight in una rete virtuale, AKS in un'altra, usando il peering](./media/apache-kafka-azure-container-services/kafka-aks-architecture.png)

> [!IMPORTANT]  
> Poiché la risoluzione dei nomi non è abilitata tra le reti con peering, vengono usati gli indirizzi IP. Per impostazione predefinita, Kafka in HDInsight è configurato per restituire i nomi host invece degli indirizzi IP quando i client si connettono. I passaggi illustrati in questo documento modificano Kafka per poter usare invece la pubblicità IP.

## <a name="create-an-azure-kubernetes-service-aks"></a>Creare un'istanza del servizio Azure Kubernetes

Se non si ha già un cluster del servizio Azure Container, vedere uno dei documenti seguenti per informazioni su come crearne uno:

* [Distribuire un cluster del servizio Azure Kubernetes - Portale](../../aks/kubernetes-walkthrough-portal.md)
* [Distribuire un cluster del servizio Azure Kubernetes - Interfaccia della riga di comando](../../aks/kubernetes-walkthrough.md)

> [!NOTE]  
> Il servizio Azure Container crea una rete virtuale durante l'installazione. Viene eseguito il peering di questa rete a quella creata per HDInsight nella sezione successiva.

## <a name="configure-virtual-network-peering"></a>Configurare il peering reti virtuali

1. Dal [portale di Azure](https://portal.azure.com) selezionare __Gruppi di risorse__ e quindi trovare il gruppo di risorse contenente la rete virtuale per il cluster del servizio Azure Container. Il nome del gruppo di risorse è `MC_<resourcegroup>_<akscluster>_<location>`. Le voci `resourcegroup` e `akscluster` sono il nome del gruppo di risorse in cui si è creato il cluster e il nome del cluster. `location` è la posizione in cui il cluster è stato creato.

2. Nel gruppo di risorse selezionare la risorsa __Rete virtuale__.

3. Selezionare __Spazio indirizzi__. Prendere nota dello spazio indirizzi elencato.

4. Per creare una rete virtuale per HDInsight, selezionare __+ Crea una risorsa__, __Rete__ e quindi __Rete virtuale__.

    > [!IMPORTANT]  
    > Quando si immettono i valori per la nuova rete virtuale, è necessario usare uno spazio indirizzi che non si sovrapponga a quello usato dalla rete di cluster del servizio Azure Container.

    Usare per la rete virtuale la stessa __posizione__ usata per il cluster del servizio Azure Container.

    Attendere che la rete virtuale sia stata creata prima di andare al passaggio successivo.

5. Per configurare il peering tra la rete di HDInsight e la rete di cluster del servizio Azure Container, selezionare la rete virtuale e quindi selezionare __Peer__. Selezionare __+ Aggiungi__ e usare i valori seguenti per popolare il modulo:

   * __Nome__: immettere un nome univoco per questa configurazione peering.
   * __Rete virtuale__: usare questo campo per selezionare la rete di cluster per il **cluster del servizio Azure Kubernetes**.

     Lasciare tutti gli altri campi impostati sul valore predefinito, quindi fare clic su __OK__ per configurare il peering.

6. Per configurare il peering tra la rete di cluster del servizio Azure Container e la rete di HDInsight, selezionare la __rete virtuale del cluster del servizio Azure Container__ e quindi selezionare __Peer__. Selezionare __+ Aggiungi__ e usare i valori seguenti per popolare il modulo:

   * __Nome__: immettere un nome univoco per questa configurazione peering.
   * __Rete virtuale__: usare questo campo per selezionare la rete di cluster per il __cluster HDInsight__.

     Lasciare tutti gli altri campi impostati sul valore predefinito, quindi fare clic su __OK__ per configurare il peering.

## <a name="install-apache-kafka-on-hdinsight"></a>Installare Apache Kafka in HDInsight

Quando si crea il cluster Kafka in HDInsight, è necessario accedere alla rete virtuale creata prima per HDInsight. Per altre informazioni sulla creazione di un cluster Kafka, vedere il documento [Creare un cluster Apache Kafka](apache-kafka-get-started.md).

> [!IMPORTANT]  
> Quando si crea il cluster, è necessario usare le __impostazioni avanzate__ per accedere alla rete virtuale creata per HDInsight.

## <a name="configure-apache-kafka-ip-advertising"></a>Configurare la pubblicità IP di Apache Kafka

Per configurare Kafka per creare pubblicità per gli indirizzi IP anziché per i nomi di dominio usare la procedura seguente:

1. Usando un Web browser, passare a https://CLUSTERNAME.azurehdinsight.net. Sostituire __CLUSTERNAME__ con il nome di Kafka nel cluster HDInsight.

    Quando richiesto, usare il nome utente HTTPS e la password per il cluster. Viene visualizzata l'interfaccia utente di Ambari Web per il cluster.

2. Per visualizzare le informazioni su Kafka, selezionare __Kafka__ dall'elenco a sinistra.

    ![Elenco di servizio con Kafka evidenziato](./media/apache-kafka-azure-container-services/select-kafka-service.png)

3. Per visualizzare la configurazione di Kafka, selezionare __Configs__ (Configurazioni) nella parte centrale in alto.

    ![Configurazione dei servizi Apache Ambari](./media/apache-kafka-azure-container-services/select-kafka-config1.png)

4. Per trovare la configurazione __kafka-env__, immettere `kafka-env` nel campo __Filtro__ in alto a destra.

    ![Configurazione Kafka per kafka-env](./media/apache-kafka-azure-container-services/search-for-kafka-env.png)

5. Per configurare Kafka affinché generi pubblicità per gli indirizzi IP, aggiungere il testo seguente nella parte inferiore del campo __kafka-env-template__:

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

6. Per configurare l'interfaccia per cui è in ascolto Kafka immettere `listeners` nel campo __Filtro__ in alto a destra.

7. Per configurare tutte le interfacce di rete per cui Kafka è in ascolto, modificare il valore nel campo __Listener__ su `PLAINTEXT://0.0.0.0:9092`.

8. Per salvare le modifiche alla configurazione usare il pulsante __Salva__. Immettere un messaggio di testo che descrive le modifiche. Selezionare __OK__ dopo aver salvato le modifiche.

    ![Configurazione del salvataggio di Apache Ambari](./media/apache-kafka-azure-container-services/save-configuration-button.png)

9. Per evitare errori al riavvio di Kafka, usare il pulsante __Service Actions__ (Azioni del servizio) e selezionare __Attiva modalità di manutenzione__. Per completare questa operazione selezionare OK.

    ![Azioni di servizio, con il pulsante di attivazione manutenzione evidenziato](./media/apache-kafka-azure-container-services/turn-on-maintenance-mode.png)

10. Per riavviare Kafka, utilizzare il pulsante __Riavvia__ e selezionare __Restart All Affected__ (Riavviare tutti gli elementi interessati). Confermare il riavvio, quindi usare il pulsante __OK__ dopo aver completato l'operazione.

    ![Pulsante di riavvio con Restart all affected (Riavviare tutti gli elementi interessati) evidenziato](./media/apache-kafka-azure-container-services/restart-required-button.png)

11. Per disabilitare la modalità di manutenzione, usare il pulsante __Service Actions__ (Azioni del servizio) e selezionare __Disattiva modalità di manutenzione__. Per completare questa operazione selezionare **OK**.

## <a name="test-the-configuration"></a>Testare la configurazione

A questo punto, Kafka e il servizio Azure Kubernetes sono in comunicazione tramite le reti virtuali con peering. Per testare la connessione, seguire questa procedura:

1. Creare un argomento Kafka usato dall'applicazione di test. Per informazioni sulla creazione di argomenti Kafka, vedere il documento [Creare un cluster Apache Kafka](apache-kafka-get-started.md).

2. Scaricare l'applicazione di esempio da [https://github.com/Blackmist/Kafka-AKS-Test](https://github.com/Blackmist/Kafka-AKS-Test).

3. Modificare il file `index.js` e cambiare le righe seguenti:

    * `var topic = 'mytopic'`: sostituire `mytopic` con il nome dell'argomento Kafka usato da questa applicazione.
    * `var brokerHost = '176.16.0.13:9092`: sostituire `176.16.0.13` con l'indirizzo IP interno di uno degli host broker per il cluster.

        Per trovare l'indirizzo IP interno degli host broker (nodi di lavoro) nel cluster, vedere il documento [API REST Apache Ambari](../hdinsight-hadoop-manage-ambari-rest-api.md#example-get-the-internal-ip-address-of-cluster-nodes). Selezionare l'indirizzo IP di una delle voci in cui il nome di dominio inizia con `wn`.

4. Da una riga di comando nella directory `src` installare le dipendenze e usare Docker to per compilare un'immagine per la distribuzione:

    ```bash
    docker build -t kafka-aks-test .
    ```

    > [!NOTE]  
    > I pacchetti necessari per questa applicazione vengono archiviati nel repository, quindi non è necessario usare l'utilità `npm` per installarli.

5. Accedere al Registro Azure Container e trovare il nome loginServer:

    ```bash
    az acr login --name <acrName>
    az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
    ```

    > [!NOTE]  
    > Se non si conosce il nome dell'istanza di Registro Azure Container o non si ha familiarità con l'uso dell'interfaccia della riga di comando di Azure per interagire con il servizio Azure Kubernetes, vedere le [esercitazioni sul servizio Azure Kubernetes](../../aks/tutorial-kubernetes-prepare-app.md).

6. Contrassegnare l'immagine `kafka-aks-test` locale con il server di accesso di Registro Azure Container. Aggiungere anche `:v1` alla fine per indicare la versione dell'immagine:

    ```bash
    docker tag kafka-aks-test <acrLoginServer>/kafka-aks-test:v1
    ```

7. Eseguire il push dell'immagine nel registro:

    ```bash
    docker push <acrLoginServer>/kafka-aks-test:v1
    ```

    Il completamento dell'operazione richiede diversi minuti.

8. Modificare il file manifesto di Kubernetes (`kafka-aks-test.yaml`) e sostituire `microsoft` con il nome del server di accesso di Registro Azure Container recuperato nel passaggio 4.

9. Usare il comando seguente per distribuire le impostazioni applicazione dal manifesto:

    ```bash
    kubectl create -f kafka-aks-test.yaml
    ```

10. Usare il comando seguente per controllare l'elemento `EXTERNAL-IP` dell'applicazione:

    ```bash
    kubectl get service kafka-aks-test --watch
    ```

    Dopo che un indirizzo IP esterno è stato assegnato, usare __CTRL + C__ per uscire dall'espressione di controllo

11. Aprire un Web browser e immettere l'indirizzo IP esterno per il servizio. Si arriva a una pagina simile all'immagine seguente:

    ![Immagine della pagina Web Apache Kafka test](./media/apache-kafka-azure-container-services/test-web-page-image1.png)

12. Immettere il testo nel campo e quindi selezionare il pulsante __Send__ (Invia). I dati vengono inviati a Kafka. Il consumer Kafka nell'applicazione legge quindi il messaggio e lo aggiunge alla sezione __Messages from Kafka__ (Messaggi da Kafka).

    > [!WARNING]  
    > È possibile ricevere più copie di un messaggio. Questo problema si verifica in genere quando si aggiorna il browser dopo la connessione o quando si aprono più connessioni tramite browser all'applicazione.

## <a name="next-steps"></a>Passaggi successivi

Usare i collegamenti seguenti per informazioni su come usare Apache Kafka in HDInsight:

* [Introduzione ad Apache Kafka (anteprima) in HDInsight](apache-kafka-get-started.md)

* [Usare MirrorMaker per creare una replica di Apache Kafka in HDInsight](apache-kafka-mirroring.md)

* [Usare Apache Storm con Apache Kafka in HDInsight](../hdinsight-apache-storm-with-kafka.md)

* [Usare Apache Spark con Apache Kafka in HDInsight](../hdinsight-apache-spark-with-kafka.md)

* [Connettersi ad Apache Kafka tramite una rete virtuale di Azure](apache-kafka-connect-vpn-gateway.md)
