---
title: Abilitare i dump dell'heap per i servizi Apache Hadoop in HDInsight - Azure
description: Abilitare i dump dell'heap per i servizi Apache Hadoop dai cluster HDInsight basati su Linux per il debug e l'analisi.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: hrasheed
ms.openlocfilehash: 5df6ab47c45a64077a39974a30c65fe13f3c851d
ms.sourcegitcommit: c79aa93d87d4db04ecc4e3eb68a75b349448cd17
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/18/2019
ms.locfileid: "71091488"
---
# <a name="enable-heap-dumps-for-apache-hadoop-services-on-linux-based-hdinsight"></a>Abilitare i dump dell'heap per i servizi Apache Hadoop in HDInsight basato su Linux

[!INCLUDE [heapdump-selector](../../includes/hdinsight-selector-heap-dump.md)]

I dump dell'heap includono uno snapshot della memoria dell'applicazione, ad esempio i valori delle variabili al momento della creazione del dump. Si rivelano quindi utili per diagnosticare i problemi che si verificano in fase di esecuzione.

## <a name="whichServices"></a>Services

È possibile abilitare dump dell'heap per i servizi seguenti:

* **Apache hcatalog** - tempelton
* **Apache hive** - hiveserver2, metastore, derbyserver
* **mapreduce** - jobhistoryserver
* **Apache yarn** - resourcemanager, nodemanager, timelineserver
* **Apache hdfs** - datanode, secondarynamenode, namenode

È inoltre possibile abilitare dump dell'heap per la mappa e ridurre i processi eseguiti da HDInsight.

## <a name="configuration"></a>Informazioni sulla configurazione dei dump dell'heap

I dump dell'heap vengono abilitati mediante il passaggio di opzioni (talvolta noto come parametri) a JVM quando viene avviato un servizio. Per la maggior parte dei servizi [Apache Hadoop](https://hadoop.apache.org/) è possibile modificare lo script della shell usato per avviare il servizio per passare queste opzioni.

In ogni script è presente un'esportazione per **\*\_OPTS**, che contiene le opzioni passate a JVM. Ad esempio, nello script **hadoop-env.sh**, la riga che inizia con `export HADOOP_NAMENODE_OPTS=` contiene le opzioni per il servizio NameNode.

I processi di mapping e riduzione sono leggermente diversi, in quanto queste operazioni sono processi figlio del servizio MapReduce. Ogni processo di mapping o riduzione viene eseguito in un contenitore figlio e ci sono due elementi che contengono le opzioni di JVM. Entrambi sono contenuti in **mapred-site.xml**:

* **mapreduce.admin.map.child.java.opts**
* **mapreduce.admin.reduce.child.java.opts**

> [!NOTE]  
> È consigliabile usare [Apache Ambari](https://ambari.apache.org/) per modificare gli script e le impostazioni di mapred-site.xml, in quanto Ambari gestisce la replica delle modifiche tra i nodi del cluster. Per i passaggi specifici, vedere la sezione [Uso di Apache Ambari](#using-apache-ambari).

### <a name="enable-heap-dumps"></a>Abilitare i dump dell'heap

L'opzione seguente abilita il dump dell'heap quando si verifica un OutOfMemoryError:

    -XX:+HeapDumpOnOutOfMemoryError

**+** indica che l'opzione è abilitata. L'impostazione predefinita è disabilitata.

> [!WARNING]  
> I dump dell'heap non sono abilitati per i servizi Hadoop in HDInsight per impostazione predefinita, perché i file di dump possono essere di grandi dimensioni. Se li si abilita per la risoluzione dei problemi, ricordarsi di disabilitarli dopo aver riprodotto il problema e raccolto i file di dump.

### <a name="dump-location"></a>Percorso dei dump

Il percorso predefinito per il file di dump è la directory di lavoro corrente. È possibile controllare dove viene archiviato il file usando l'opzione seguente:

    -XX:HeapDumpPath=/path

Ad esempio, se si usa `-XX:HeapDumpPath=/tmp`, il dump viene archiviato nella directory /tmp.

### <a name="scripts"></a>Script

È anche possibile generare uno script quando si verifica un **OutOfMemoryError** . Ad esempio, attivando una notifica per sapere che si è verificato l'errore. Usare l'opzione seguente per attivare uno script su un __OutOfMemoryError__:

    -XX:OnOutOfMemoryError=/path/to/script

> [!NOTE]  
> Poiché Apache Hadoop è un sistema distribuito, tutti gli script usati devono essere posizionati in tutti i nodi del cluster in cui viene eseguito il servizio.
> 
> Lo script deve anche trovarsi in un percorso accessibile dall'account con cui viene eseguito il servizio e deve fornire autorizzazioni di esecuzione. Ad esempio, è possibile archiviare gli script in `/usr/local/bin` e usare `chmod go+rx /usr/local/bin/filename.sh` per concedere autorizzazioni di lettura ed esecuzione.

## <a name="using-apache-ambari"></a>Uso di Apache Ambari

Per modificare la configurazione di un servizio, attenersi alla procedura seguente:

1. Aprire l'interfaccia utente Web di Ambari per il cluster. L'URL è https://YOURCLUSTERNAME.azurehdinsight.net.

    Quando richiesto, eseguire l'autenticazione al sito usando il nome dell'account HTTP (impostazione predefinita: admin) e la password per il cluster.

   > [!NOTE]  
   > È possibile che Ambari richieda il nome utente e la password una seconda volta. In questo caso, immettere lo stesso nome e la stessa password dell'account.

2. Usando l'elenco a sinistra, selezionare l'area del servizio da modificare. Ad esempio, **HDFS**. Nell'area centrale selezionare la scheda **Configs** .

    ![Immagine dell'interfaccia Web di Ambari con la scheda HDFS Configs selezionata](./media/hdinsight-hadoop-collect-debug-heap-dump-linux/hdi-service-config-tab.png)

3. Usando la voce **Filter...** immettere **opts**. Vengono visualizzati solo gli elementi che contengono questo testo.

    ![Elenco filtrato di configurazione di Apache Ambari](./media/hdinsight-hadoop-collect-debug-heap-dump-linux/hdinsight-filter-list.png)

4. Trovare la voce **\*\_OPTS** per il servizio per cui si vogliono abilitare i dump dell'heap e aggiungere le opzioni da abilitare. Nella figura seguente è stato aggiunto `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/` alla voce **HADOOP\_NAMENODE\_OPTS**:

    ![Apache Ambari Hadoop-NameNode-opz](./media/hdinsight-hadoop-collect-debug-heap-dump-linux/hadoop-namenode-opts.png)

   > [!NOTE]  
   > Quando si abilitato i dump dell'heap per il processo figlio di mapping o riduzione, si cercano i campi denominati **mapreduce.admin.map.child.java.opts** e **mapreduce.admin.reduce.child.java.opts**.

    Fare clic sul pulsante **Save** per salvare le modifiche apportate. Sarà possibile immettere una breve nota che descrive le modifiche.

5. Dopo avere applicato le modifiche, viene visualizzata l'icona **Restart required** accanto a uno o più servizi.

    ![icona restart required e pulsante restart](./media/hdinsight-hadoop-collect-debug-heap-dump-linux/restart-required-icon.png)

6. Selezionare ogni servizio che richiede un riavvio e usare il pulsante **Service Actions** per **Disattivare la modalità di manutenzione**. La modalità di manutenzione impedisce che vengano generati avvisi quando si riavvia questo servizio.

    ![Attivare il menu modalità di manutenzione HDI](./media/hdinsight-hadoop-collect-debug-heap-dump-linux/hdi-maintenance-mode.png)

7. Dopo aver abilitato la modalità di manutenzione, usare il pulsante **Restart** per il servizio per **Restart All Effected**

    ![Voce di riavvio tutti interessati di Apache Ambari](./media/hdinsight-hadoop-collect-debug-heap-dump-linux/hdi-restart-all-button.png)

   > [!NOTE]  
   > Le voci per il pulsante **Restart** (Riavvia) possono essere diverse per altri servizi.

8. Dopo il riavvio dei servizi, usare il pulsante **Service Actions** per **Disattivare la modalità di manutenzione**. Questa opzione indica ad Ambari di riprendere il monitoraggio per gli avvisi relativi al servizio.

