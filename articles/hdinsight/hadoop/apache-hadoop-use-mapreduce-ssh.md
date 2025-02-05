---
title: Connessione di MapReduce e SSH con Apache Hadoop in HDInsight - Azure
description: Informazioni su come usare SSH per eseguire processi MapReduce con Apache Hadoop in HDInsight.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/10/2018
ms.author: hrasheed
ms.openlocfilehash: 5eac6cd60b9e58dea8901fad076bb481d5f2eb06
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "67078313"
---
# <a name="use-mapreduce-with-apache-hadoop-on-hdinsight-with-ssh"></a>Usare MapReduce con Apache Hadoop in HDInsight con SSH

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Informazioni su come inviare processi MapReduce da una connessione Secure Shell (SSH) a HDInsight.

> [!NOTE]
> Se si ha già familiarità con l'uso di server Apache Hadoop basati su Linux ma non si è esperti di HDInsight, vedere [Informazioni sull'uso di HDInsight in Linux](../hdinsight-hadoop-linux-information.md).

## <a id="prereq"></a>Prerequisiti

* Un cluster HDInsight (Hadoop in HDInsight) basato su Linux.

* Un client SSH. Per altre informazioni, vedere [Usare SSH con HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="ssh"></a>Connettersi con SSH

Connettersi al cluster tramite SSH. Il comando seguente, ad esempio, stabilisce la connessione a un cluster denominato **myhdinsight** con l'account **sshuser**:

```bash
ssh sshuser@myhdinsight-ssh.azurehdinsight.net
```

**Se si usa una chiave del certificato per l'autenticazione SSH**, è possibile che sia necessario specificare il percorso della chiave privata nel sistema client, ad esempio:

```bash
ssh -i ~/mykey.key sshuser@myhdinsight-ssh.azurehdinsight.net
```

**Se si usa una password per l'autenticazione SSH**, è necessario fornire la password quando viene richiesta.

Per altre informazioni sull'uso di SSH con HDInsight, vedere l'articolo [Usare SSH con HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="hadoop"></a>Usare i comandi Hadoop

1. Dopo essersi connessi al cluster HDInsight, usare il comando seguente per avviare un processo MapReduce:

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput
    ```

    Questo comando avvia la classe `wordcount`, contenuta nel file `hadoop-mapreduce-examples.jar`. Usa il documento `/example/data/gutenberg/davinci.txt` come input, mentre l'output viene archiviato in `/example/data/WordCountOutput`.

    > [!NOTE]
    > Per altre informazioni su questo processo MapReduce e per i dati di esempio, vedere [Usare MapReduce in Apache Hadoop in HDInsight](hdinsight-use-mapreduce.md).

2. Il processo genera dettagli durante l'elaborazione e, al termine, restituisce informazioni simili alle seguenti:

        File Input Format Counters
        Bytes Read=1395666
        File Output Format Counters
        Bytes Written=337623

3. Al termine del processo, usare il comando seguente per elencare i file di output:

    ```bash
    hdfs dfs -ls /example/data/WordCountOutput
    ```

    Con questo comando vengono visualizzati due file: `_SUCCESS` e `part-r-00000`. Il file `part-r-00000` contiene l'output del processo.

    > [!NOTE]  
    > Alcuni processi MapReduce possono dividere i risultati in più file **part-r-#####** . In questo caso, usare il suffisso ##### per indicare l'ordine dei file.

4. Per visualizzare l'output, usare il seguente comando:

    ```bash
    hdfs dfs -cat /example/data/WordCountOutput/part-r-00000
    ```

    Viene visualizzato un elenco di parole contenute nel file **wasb://example/data/gutenberg/davinci.txt** e il numero di occorrenze di ogni parola. Di seguito è riportato un esempio dei dati contenuti nel file:

        wreathed        3
        wreathing       1
        wreaths         1
        wrecked         3
        wrenching       1
        wretched        6
        wriggling       1

## <a id="summary"></a>Riepilogo

Come è possibile notare, i comandi Hadoop forniscono un modo semplice per eseguire processi MapReduce in un cluster HDInsight e visualizzare l'output del processo.

## <a id="nextsteps"></a>Passaggi successivi

Per informazioni generali sui processi MapReduce in HDInsight:

* [Usare MapReduce in HDInsight Hadoop](hdinsight-use-mapreduce.md)

Per informazioni su altre modalità d'uso di Hadoop in HDInsight:

* [Usare Apache Hive con Apache Hadoop su HDInsight](hdinsight-use-hive.md)
* [Usare Apache Pig con Apache Hadoop su HDInsight](hdinsight-use-pig.md)
