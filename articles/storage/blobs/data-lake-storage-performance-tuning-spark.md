---
title: Linee guida per l'ottimizzazione delle prestazioni di Spark in Azure Data Lake Storage Gen2 | Microsoft Docs
description: Linee guida per l'ottimizzazione delle prestazioni di Spark in Azure Data Lake Storage Gen2
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: feefcf4f9f4448ab2b36c415cb745fd98277eb28
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "64939327"
---
# <a name="performance-tuning-guidance-for-spark-on-hdinsight-and-azure-data-lake-storage-gen2"></a>Materiale sussidiario per l'ottimizzazione delle prestazioni di Spark in HDInsight e Azure Data Lake Storage Gen2

Per l'ottimizzazione delle prestazioni in Spark, è necessario considerare il numero di applicazioni che verranno eseguite nel cluster.  Per impostazione predefinita, nel cluster HDI è possibile eseguire 4 app simultaneamente (nota: l'impostazione predefinita è soggetta a modifiche).  È possibile decidere di usare un numero inferiore di app in modo da sostituire le impostazioni predefinite e usare una parte più ampia del cluster per tali applicazioni.  

## <a name="prerequisites"></a>Prerequisiti

* **Una sottoscrizione di Azure**. Vedere [Ottenere una versione di prova gratuita di Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un account di Azure Data Lake Storage Gen2**. Per istruzioni su come crearne uno, vedere [Guida introduttiva: Creare un account di archiviazione di Azure Data Lake Storage Gen2](data-lake-storage-quickstart-create-account.md).
* Un **cluster Azure HDInsight** con accesso a un account Data Lake Storage Gen2. Vedere [Usare Archiviazione Azure Data Lake Storage Gen2 con cluster Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2). Assicurarsi di abilitare il Desktop remoto per il cluster.
* **Esecuzione di cluster Spark in Data Lake Storage Gen2**.  Per altre informazioni, vedere [Usare il cluster Spark di HDInsight per analizzare i dati in Data Lake Storage Gen2](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-use-with-data-lake-store)
* **Linee guida per l'ottimizzazione delle prestazioni in Data Lake Storage Gen2**.  Per informazioni sui concetti generali relativi alle prestazioni, vedere [Data Lake Storage Gen2 Performance Tuning Guidance](data-lake-storage-performance-tuning-guidance.md) (Linee guida per l'ottimizzazione delle prestazioni in Data Lake Storage Gen2). 

## <a name="parameters"></a>Parametri

Di seguito sono riportate le impostazioni più importanti che possono essere ottimizzate per migliorare le prestazioni in Data Lake Storage Gen2 durante l'esecuzione di processi Spark:

* **Num-executors**: numero di attività che è possibile eseguire simultaneamente.

* **Executor-memory**: quantità di memoria allocata per ogni executor.

* **Executor-cores**: numero di core allocati per ogni executor.                     

**Num-executors**. I Num-executors impostano il numero massimo di attività che è possibile eseguire in parallelo.  Il numero effettivo di attività che possono essere eseguite in parallelo è limitato dalle risorse di memoria e CPU disponibili nel cluster.

**Executor-memory**. Si tratta della quantità di memoria che viene allocata per ogni executor.  La memoria necessaria per ogni executor dipende dal processo.  Per operazioni complesse, la quantità di memoria deve essere maggiore.  Per operazioni semplici come la lettura e la scrittura, la quantità di memoria richiesta è inferiore.  La quantità di memoria necessaria per ogni executor è disponibile in Ambari.  In Ambari passare a Spark e visualizzare la scheda Configs (Configurazioni).  

**Executor-cores**. Definisce il numero di core usati per ogni executor, che determina il numero di thread paralleli che è possibile eseguire per ogni executor.  Ad esempio, se executor-cores = 2, ogni executor può eseguire 2 attività parallele nell'executor.  Gli executor-cores necessari dipendono dal processo.  I processi I/O intensivi non richiedono una grande quantità di memoria per ogni attività. In questo modo, ogni executor può gestire più attività in parallelo.

Per impostazione predefinita, durante l'esecuzione di Spark in HDInsight, vengono definiti due core YARN virtuali per ogni core fisico.  Questo numero fornisce un buon bilanciamento tra concorrenza e quantità di contesto nel passaggio tra più thread.  

## <a name="guidance"></a>Materiale sussidiario

Durante l'esecuzione di carichi di lavoro analitici di Spark per l'elaborazione dei dati in Data Lake Storage Gen2, è consigliabile usare la versione più recente di HDInsight per ottenere prestazioni ottimali con Data Lake Storage Gen2. Quando il processo prevede un I/O più intensivo, è possibile configurare alcuni parametri per migliorare le prestazioni.  Data Lake Storage Gen2 è una piattaforma di archiviazione altamente scalabile in grado di gestire un'elevata velocità effettiva.  Se il processo è costituito principalmente da lettura o scrittura, l'aumento della concorrenza di I/O da e verso Data Lake Storage Gen2 potrebbe migliorare le prestazioni.

Esistono alcuni modi generali per aumentare la concorrenza per i processi con I/O intensivo.

**Passaggio 1: determinare il numero di app in esecuzione nel cluster**. È necessario conoscere il numero di applicazioni in esecuzione nel cluster, inclusa quella corrente.  I valori predefiniti per ogni impostazione Spark presumono che siano presenti 4 applicazioni in esecuzione contemporanea.  Pertanto, si disporrà solo del 25% del cluster per ogni applicazione.  Per ottenere prestazioni migliori, è possibile sostituire le impostazioni predefinite modificando il numero di executor.  

**Passaggio 2: impostare executor-memory**. Il primo valore da impostare è la memoria dell'executor.  La memoria dipende dal processo da eseguire.  È possibile aumentare la concorrenza allocando una quantità inferiore di memoria per ogni executor.  Se vengono visualizzate eccezioni di memoria insufficiente quando si esegue il processo, è necessario aumentare il valore di questo parametro.  In alternativa è possibile ottenere una maggiore quantità di memoria usando un cluster con maggiore memoria oppure aumentando le dimensioni del cluster.  Una maggiore quantità di memoria consentirà di usare più executor, ottenendo così più concorrenza.

**Passaggio 3: impostare executor-cores**. Per carichi di lavoro I/O intensivi che non prevedono operazioni complesse, è consigliabile iniziare con un numero elevato di core per ogni executor al fine di aumentare il numero di attività parallele per ognuno di essi.  Per iniziare, è consigliabile impostare executor-cores su 4.   

    executor-cores = 4
L'aumento del numero di core dell'executor offrirà maggior parallelismo in modo che sia possibile sperimentare diversi core dell'executor.  Per i processi che includono operazioni più complesse, è necessario ridurre il numero di core per ogni executor.  Se executor-cores viene impostato su un valore superiore a 4, la Garbage Collection può diventare inefficiente e incidere negativamente sulle prestazioni.

**Passaggio 4: determinare la quantità di memoria YARN nel cluster**. Queste informazioni sono disponibili in Ambari.  Passare a YARN e visualizzare la scheda Configs (Configurazioni).  La memoria YARN è visualizzata in questa finestra.  
Si noti che nella finestra è possibile visualizzare anche le dimensioni predefinite del contenitore YARN.  Le dimensioni del contenitore YARN sono uguali alle dimensioni della memoria per ogni parametro executor.

    Total YARN memory = nodes * YARN memory per node
**Passaggio 5: calcolare num-executors**

**Calcolare il vincolo della memoria**. Il parametro num-executors è vincolato dalla memoria o dalla CPU.  Il vincolo di memoria è determinato dalla quantità di memoria YARN disponibile per l'applicazione.  Prendere il valore della memoria totale di YARN e dividerlo per il valore executor-memory.  Il vincolo deve essere diviso per il numero di applicazioni.

    Memory constraint = (total YARN memory / executor memory) / # of apps   
**Calcolare il vincolo della CPU**. Il vincolo della CPU viene calcolato come il numero totale dei core virtuali diviso per il numero di core per ogni executor.  Per ogni core fisico sono presenti 2 core virtuali.  Come per il vincolo di memoria, questo valore viene diviso per il numero di app.

    virtual cores = (nodes in cluster * # of physical cores in node * 2)
    CPU constraint = (total virtual cores / # of cores per executor) / # of apps
**Impostare num-executors**. Il parametro num-executors viene determinato dal valore minimo del vincolo della memoria e del vincolo della CPU. 

    num-executors = Min (total virtual Cores / # of cores per executor, available YARN memory / executor-memory)   
Impostare un numero maggiore di num-executors non si traduce necessariamente in un miglioramento delle prestazioni.  Si noti che l'aggiunta di esecutori può comportare un carico extra per ciascuno di questi ultimi, con una potenziale diminuzione delle prestazioni.  I num-executors sono limitati dalle risorse del cluster.    

## <a name="example-calculation"></a>Calcolo di esempio

Si supponga di disporre di un cluster costituito da 8 nodi D4v2 che eseguono 2 applicazioni, inclusa quella che si è in procinto di eseguire.  

**Passaggio 1: determinare il numero di app in esecuzione nel cluster**. L'utente sa di avere 2 app nel cluster, inclusa quella corrente.  

**Passaggio 2: impostare executor-memory**. Per questo esempio, si stabilisce che 6 GB di memoria per executor saranno sufficienti per il processo I/O intensivo.  

    executor-memory = 6GB
**Passaggio 3: impostare executor-cores**. Poiché si tratta di un processo I/O intensivo, è possibile impostare su 4 il numero di core per ogni executor.  L'impostazione di un numero di core per executor superiore a 4 può causare problemi alla Garbage Collection.  

    executor-cores = 4
**Passaggio 4: determinare la quantità di memoria YARN nel cluster**. Aprendo Ambari, si scopre che ogni D4v2 ha 25 GB di memoria YARN.  Poiché sono presenti 8 nodi, la memoria di YARN disponibile viene moltiplicata per 8.

    Total YARN memory = nodes * YARN memory* per node
    Total YARN memory = 8 nodes * 25GB = 200GB
**Passaggio 5: impostare num-executors**. Il parametro num-executors viene determinato dal valore minimo del vincolo della memoria e del vincolo della CPU, divisi per il numero di app in esecuzione in Spark.    

**Calcolare il vincolo della memoria**. Il vincolo della memoria viene calcolato come la memoria totale di YARN divisa per la memoria di ogni executor.

    Memory constraint = (total YARN memory / executor memory) / # of apps   
    Memory constraint = (200GB / 6GB) / 2   
    Memory constraint = 16 (rounded)
**Calcolare il vincolo della CPU**. Il vincolo della CPU viene calcolato come il numero totale dei core di YARN diviso per il numero di core per ogni executor.
    
    YARN cores = nodes in cluster * # of cores per node * 2   
    YARN cores = 8 nodes * 8 cores per D14 * 2 = 128
    CPU constraint = (total YARN cores / # of cores per executor) / # of apps
    CPU constraint = (128 / 4) / 2
    CPU constraint = 16
**Impostare num-executors**

    num-executors = Min (memory constraint, CPU constraint)
    num-executors = Min (16, 16)
    num-executors = 16    

