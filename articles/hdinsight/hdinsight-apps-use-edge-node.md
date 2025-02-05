---
title: Usare i nodi perimetrali vuoti sui cluster Apache Hadoop in HDInsight - Azure
description: Come aggiungere un nodo perimetrale vuoto al cluster HDInsight che può essere usato come un client, quindi testare oppure ospitare le applicazioni HDInsight.
ms.reviewer: jasonh
author: hrasheed-msft
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 06/13/2019
ms.author: hrasheed
ms.openlocfilehash: cf84d685bf20f1117aa2378cf2e2696cb78a46a5
ms.sourcegitcommit: 8ef0a2ddaece5e7b2ac678a73b605b2073b76e88
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/17/2019
ms.locfileid: "71076156"
---
# <a name="use-empty-edge-nodes-on-apache-hadoop-clusters-in-hdinsight"></a>Usare i nodi perimetrali vuoti sui cluster Apache Hadoop in HDInsight

Informazioni su come aggiungere un nodo perimetrale vuoto a un cluster HDInsight. Un nodo perimetrale vuoto è una macchina virtuale Linux con gli stessi strumenti client installati e configurati nei nodi head, ma senza servizi [Apache Hadoop](https://hadoop.apache.org/) in esecuzione. Il nodo perimetrale può essere usato per accedere al cluster e per testare e ospitare le applicazioni client. 

È possibile aggiungere un nodo perimetrale vuoto a un cluster HDInsight esistente o a un nuovo cluster quando lo si crea. L'aggiunta di un nodo perimetrale vuoto si esegue usando un modello di Azure Resource Manager.  L'esempio seguente illustra come eseguire questa operazione tramite un modello:

    "resources": [
        {
            "name": "[concat(parameters('clusterName'),'/', variables('applicationName'))]",
            "type": "Microsoft.HDInsight/clusters/applications",
            "apiVersion": "2015-03-01-preview",
            "dependsOn": [ "[concat('Microsoft.HDInsight/clusters/',parameters('clusterName'))]" ],
            "properties": {
                "marketPlaceIdentifier": "EmptyNode",
                "computeProfile": {
                    "roles": [{
                        "name": "edgenode",
                        "targetInstanceCount": 1,
                        "hardwareProfile": {
                            "vmSize": "Standard_D3"
                        }
                    }]
                },
                "installScriptActions": [{
                    "name": "[concat('emptynode','-' ,uniquestring(variables('applicationName')))]",
                    "uri": "[parameters('installScriptAction')]",
                    "roles": ["edgenode"]
                }],
                "uninstallScriptActions": [],
                "httpsEndpoints": [],
                "applicationType": "CustomApplication"
            }
        }
    ],

Come illustrato nell'esempio, è possibile chiamare facoltativamente un'[azione script](hdinsight-hadoop-customize-cluster-linux.md) per eseguire configurazioni aggiuntive, ad esempio per installare [Apache Hue](hdinsight-hadoop-hue-linux.md) nel nodo perimetrale. Lo script dell'azione script deve essere pubblicamente accessibile sul web.  Ad esempio, se lo script viene archiviato in Archiviazione di Azure, usare contenitori o BLOB pubblici.

Le dimensioni della macchina virtuale del nodo perimetrale devono soddisfare i requisiti di dimensioni per le macchine virtuali del nodo di lavoro del cluster HDInsight. Per conoscere le dimensioni consigliate per le macchine virtuali dei nodi di lavoro, vedere [Creare cluster Apache Hadoop in HDInsight](hdinsight-hadoop-provision-linux-clusters.md#cluster-types).

Dopo aver creato un nodo perimetrale, è possibile connettersi al nodo stesso tramite SSH ed eseguire gli strumenti client per accedere al cluster Hadoop in HDInsight.

> [!WARNING]   
> I componenti personalizzati che vengono installati nel nodo perimetrale ricevono supporto ragionevole a livello commerciale da Microsoft. Ciò può portare alla risoluzione dei problemi riscontrati. In alternativa è possibile fare riferimento alle risorse della community per ulteriore assistenza. Di seguito sono indicati alcuni dei siti più attivi per ricevere assistenza dalla community:
>
> * [Forum MSDN per HDInsight](https://social.msdn.microsoft.com/Forums/azure/home?forum=hdinsight)
> * [https://stackoverflow.com](https://stackoverflow.com).
>
> Se si usa una tecnologia Apache, è possibile ottenere assistenza tramite i siti di progetto Apache su [https://apache.org](https://apache.org), ad esempio il sito [Apache Hadoop](https://hadoop.apache.org/).

> [!IMPORTANT]
> Le immagini di Ubuntu diventano disponibili per la nuova creazione del cluster HDInsight entro 3 mesi dalla pubblicazione. A partire da gennaio 2019, le patch **non** sono applicate automaticamente ai cluster in esecuzione (inclusi i nodi perimetrali). I clienti devono usare azioni script o altri meccanismi per applicare patch a un cluster in esecuzione.  Per altre informazioni, vedere [Applicazione di patch del sistema operativo per HDInsight](./hdinsight-os-patching.md).

## <a name="add-an-edge-node-to-an-existing-cluster"></a>Aggiungere un nodo perimetrale a un cluster esistente
In questa sezione si userà un modello di Resource Manager per aggiungere un nodo perimetrale a un cluster HDInsight esistente.  Il modello di Resource Manager è disponibile in [GitHub](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-add-edge-node/). Il modello di Resource Manager chiama un'azione script disponibile in https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-add-edge-node/scripts/EmptyNodeSetup.sh. Lo script non esegue alcuna azione.  Per illustrare la chiamata all'azione script da un modello di Resource Manager.

**Per aggiungere un nodo perimetrale a un cluster esistente**

1. Fare clic sull'immagine seguente per accedere ad Azure e aprire il modello di Azure Resource Manager nel portale di Azure. 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-add-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>
3. Configurare le proprietà seguenti:
   
   * **Sottoscrizione** selezionare la sottoscrizione di Azure usata per creare il cluster.
   * **Gruppo di risorse**: selezionare il gruppo di risorse usato per il cluster HDInsight esistente.
   * **Località**: selezionare la località del cluster HDInsight esistente.
   * **Nome del cluster**: immettere il nome di un cluster HDInsight esistente.
   * **Edge Node Size** (Dimensioni nodo perimetrale): selezionare una delle dimensioni di macchina virtuale. Le dimensioni della macchina virtuale devono soddisfare i requisiti di dimensione per le macchine virtuali del nodo di lavoro. Per conoscere le dimensioni consigliate per le macchine virtuali dei nodi di lavoro, vedere [Creare cluster Apache Hadoop in HDInsight](hdinsight-hadoop-provision-linux-clusters.md#cluster-types).
   * **Edge Node Prefix** (Prefisso nodo perimetrale): il valore predefinito è **new**.  Se si usa il valore predefinito, il nome del nodo perimetrale sarà **new-edgenode**.  È possibile personalizzare il prefisso nel portale. Si può anche personalizzare il nome completo nel modello.

4. Selezionare **Accetto le condizioni riportate sopra** e quindi fare clic su **Acquista** per creare il nodo perimetrale.

>[!IMPORTANT]  
> Assicurarsi di selezionare il gruppo di risorse di Azure per il cluster HDInsight esistente.  In caso contrario, verrà visualizzato il messaggio di errore "Non è possibile eseguire l'operazione richiesta su una risorsa annidata. La risorsa padre '&lt;NomeCluster>' non è stata trovata".

## <a name="add-an-edge-node-when-creating-a-cluster"></a>Aggiungere un nodo perimetrale durante la creazione di un cluster
In questa sezione si userà un modello di Resource Manager per creare un cluster HDInsight con un nodo perimetrale.  Il modello di Gestione risorse si trova nella [raccolta di modelli di avvio rapido di Azure](https://azure.microsoft.com/documentation/templates/101-hdinsight-linux-with-edge-node/). Il modello di Resource Manager chiama un'azione script disponibile in https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-with-edge-node/scripts/EmptyNodeSetup.sh. Lo script non esegue alcuna azione.  Per illustrare la chiamata all'azione script da un modello di Resource Manager.

**Creare un cluster HDInsight con un nodo perimetrale**

1. Creare un cluster HDInsight, se non ne è ancora disponibile uno.  Vedere [Introduzione all'uso di Hadoop in HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md).
2. Fare clic sull'immagine seguente per accedere ad Azure e aprire il modello di Azure Resource Manager nel portale di Azure. 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>
3. Configurare le proprietà seguenti:
   
   * **Sottoscrizione** selezionare la sottoscrizione di Azure usata per creare il cluster.
   * **Gruppo di risorse**: creare un nuovo gruppo di risorse usato per il cluster.
   * **Località**: Selezionare una posizione per il gruppo di risorse.
   * **Nome del cluster**: immettere un nome per il nuovo cluster da creare.
   * **Nome utente dell'account di accesso del cluster**: immettere il nome utente HTTP di Hadoop.  Il nome predefinito è **admin**.
   * **Password dell'account di accesso del cluster**: immettere la password utente HTTP di Hadoop.
   * **Nome utente SSH**: immettere il nome utente SSH. Il nome predefinito è **sshuser**.
   * **Password SSH**: immettere la password utente SSH.
   * **Install Script Action** (Installa azione script): Mantieni il valore predefinito per questo articolo.
     
     Alcune proprietà sono state impostate come hardcoded nel modello: il tipo di cluster, il numero di nodi di ruolo di lavoro del cluster, le dimensioni e il nome del nodo perimetrale.
4. Selezionare **Accetto le condizioni riportate sopra** e quindi fare clic su **Acquista** per creare il cluster con il nodo perimetrale.

## <a name="add-multiple-edge-nodes"></a>Aggiungere più nodi perimetrali

È possibile aggiungere più nodi perimetrali a un cluster HDInsight.  La configurazione basata su più nodi perimetrali può essere eseguita solo usando i modelli di Azure Resource Manager.  Vedere l'esempio di modello all'inizio di questo articolo.  È necessario aggiornare il valore **targetInstanceCount** in base al numero di nodi perimetrali da creare.

## <a name="access-an-edge-node"></a>Accedere a un nodo perimetrale
L'endpoint SSH del nodo perimetrale è &lt;NomeNodoPerimetrale>.&lt;NomeCluster>-ssh.azurehdinsight.net:22.  Ad esempio, new-edgenode.myedgenode0914-ssh.azurehdinsight.net:22.

Il nodo perimetrale viene visualizzato come un'applicazione nel portale di Azure.  Il portale fornisce le informazioni per accedere al nodo perimetrale con SSH.

**Per verificare l'endpoint SSH del nodo perimetrale**

1. Accedere al [portale di Azure](https://portal.azure.com).
2. Aprire il cluster HDInsight con un nodo perimetrale.
3. Fare clic su **Applicazioni**. Verrà visualizzato il nodo perimetrale.  Il nome predefinito è **new-edgenode**.
4. Fare clic sul nodo perimetrale. Verrà visualizzato l'endpoint SSH.

**Usare Hive nel nodo perimetrale**

1. Usare SSH per connettersi al nodo perimetrale. Per altre informazioni, vedere [Usare SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Dopo essersi connessi al nodo perimetrale tramite SSH, usare il comando seguente per aprire la console di Hive:
   
        hive
3. Eseguire il comando seguente per visualizzare le tabelle Hive nel cluster:
   
        show tables;

## <a name="delete-an-edge-node"></a>Eliminare un nodo perimetrale
È possibile eliminare un nodo perimetrale dal portale di Azure.

**Per accedere a un nodo perimetrale**

1. Accedere al [portale di Azure](https://portal.azure.com).
2. Aprire il cluster HDInsight con un nodo perimetrale.
3. Fare clic su **Applicazioni**. Verrà visualizzato un elenco di nodi perimetrali.  
4. Fare clic con il pulsante destro del mouse sul nodo perimetrale che si vuole eliminare e quindi scegliere **Elimina**.
5. Fare clic su **Sì** per confermare.

## <a name="next-steps"></a>Passaggi successivi
In questo articolo si è appreso come aggiungere un nodo perimetrale e come accedervi. Per altre informazioni, vedere gli articoli seguenti:

* [Installare applicazioni HDInsight](hdinsight-apps-install-applications.md): informazioni su come installare un'applicazione HDInsight nei cluster.
* [Installare applicazioni HDInsight personalizzate](hdinsight-apps-install-custom-applications.md): informazioni su come distribuire in HDInsight un'applicazione HDInsight non pubblicata.
* [Pubblicare applicazioni HDInsight](hdinsight-apps-publish-applications.md): Informazioni su come pubblicare applicazioni HDInsight personalizzate in Azure Marketplace.
* [MSDN: Installare un'applicazione HDInsight](https://msdn.microsoft.com/library/mt706515.aspx): Informazioni su come definire le applicazioni HDInsight.
* [Personalizzare cluster HDInsight basati su Linux tramite Azioni script](hdinsight-hadoop-customize-cluster-linux.md): informazioni su come usare Azioni script per installare applicazioni aggiuntive.
* [Creare cluster Apache Hadoop basati su Linux in HDInsight tramite modelli ARM](hdinsight-hadoop-create-linux-clusters-arm-templates.md): informazioni su come chiamare i modelli di Azure Resource Manager per creare cluster HDInsight.

