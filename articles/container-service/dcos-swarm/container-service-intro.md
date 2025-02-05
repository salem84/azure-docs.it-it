---
title: (DEPRECATO) Hosting di contenitore Docker nel cloud di Azure
description: Il servizio Azure Container fornisce un modo per semplificare la creazione, la configurazione e la gestione di un cluster di macchine virtuali che sono preconfigurate per eseguire le applicazioni nei contenitori.
services: container-service
author: rgardler
manager: jeconnoc
ms.service: container-service
ms.topic: overview
ms.date: 03/01/2017
ms.author: rogardle
ms.custom: H1Hack27Feb2017, mvc
ms.openlocfilehash: 957725d40c66ffe9d8d7ce1362279c2bb4d4fded
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 05/16/2019
ms.locfileid: "65788732"
---
# <a name="deprecated-introduction-to-docker-container-hosting-solutions-with-azure-container-service"></a>(DEPRECATO) Introduzione alle soluzioni di hosting di contenitori Docker con il servizio Azure Container 

[!INCLUDE [ACS deprecation](../../../includes/container-service-deprecation.md)]

Il servizio Azure Container fornisce un modo per semplificare la creazione, la configurazione e la gestione di un cluster di macchine virtuali preconfigurate per eseguire le applicazioni nei contenitori. Usa una configurazione ottimizzata di strumenti di pianificazione e orchestrazione open source comuni. Ciò consente di usare le competenze già acquisite o di attingere da un consistente e crescente bagaglio di competenze a livello di community per distribuire e gestire applicazioni basate sul contenitore in Microsoft Azure.

![Il servizio Azure Container fornisce uno strumento per gestire applicazioni in contenitori su più host in Azure.](./media/acs-intro/acs-cluster-new.png)

Il servizio Azure Container si avvale del formato di contenitore Docker per garantire che i contenitori di applicazioni siano completamente portabili. Supporta anche la scelta di Marathon e DC/OS, Docker Swarm o Kubernetes per garantire che queste applicazioni possano essere ridimensionate in migliaia, persino decine di migliaia, di contenitori.

Il servizio Azure Container consente di sfruttare i vantaggi delle funzioni di classe enterprise di Azure mantenendo al tempo stesso la portabilità delle applicazioni, inclusi i livelli di orchestrazione.

## <a name="using-azure-container-service"></a>Utilizzo del servizio Azure Container
L'obiettivo del servizio Azure Container è fornire un ambiente host contenitore tramite tecnologie e strumenti open source, che sono attualmente diffusi fra i nostri clienti. A tal fine vengono esposti gli endpoint API standard per l'agente di orchestrazione scelto: DC/OS, Docker Swarm o Kubernetes. Tramite questi endpoint è possibile usare qualsiasi software in grado di comunicare con essi. Ad esempio, nel caso dell'endpoint Docker Swarm, è possibile scegliere di usare l'interfaccia della riga di comando (CLI) Docker. Per DC/OS è possibile scegliere l'interfaccia della riga di comando DCOS. Per Kubernetes, è possibile scegliere `kubectl`.

## <a name="creating-a-docker-cluster-by-using-azure-container-service"></a>Creazione di un cluster Docker con il servizio Azure Container
Per iniziare a usare il servizio Azure Container, distribuire un cluster del servizio tramite il portale (cercare **servizio Azure Container** nel Marketplace), usando un modello di Azure Resource Manager ([Docker Swarm](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acs-swarm), [Controller di dominio/sistema operativo](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acs-dcos) o [Kubernetes](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acs-kubernetes)) oppure l'[interfaccia della riga di comando di Azure](container-service-create-acs-cluster-cli.md). I modelli di avvio rapido forniti possono essere modificati per includere una configurazione di Azure aggiuntiva o avanzata. Per altre informazioni, vedere [Distribuire un cluster del servizio Azure Container](container-service-deployment.md).

## <a name="deploying-an-application"></a>Distribuzione di un'applicazione
Il servizio Azure Container consente di scegliere tra Docker Swarm, DC/OS o Kubernetes per l'orchestrazione. La modalità di distribuzione dell'applicazione dipende dall'agente di orchestrazione scelto.

### <a name="using-dcos"></a>Uso di DC/OS
DC/OS è un sistema operativo distribuito basato sul kernel dei sistemi distribuiti Apache Mesos. Apache Mesos fa parte di Apache Software Foundation e annovera tra gli utenti e i collaboratori alcuni dei [nomi più importanti del panorama IT](https://mesos.apache.org/documentation/latest/powered-by-mesos/) .

![servizio Azure Container configurato per DC/OS che mostra agenti e schemi.](media/acs-intro/dcos.png)

DC/OS e Apache Mesos offrono un set di funzionalità molto ampio:

* Scalabilità collaudata
* Server primario e server secondari replicati a tolleranza di errore che usano Apache ZooKeeper
* Supporto per i contenitori formattati Docker
* Isolamento nativo tra le attività con i contenitori Linux
* Pianificazione di più risorse (memoria, CPU, disco e porte)
* Java, Python e API C++ per lo sviluppo di nuove applicazioni parallele
* Interfaccia utente Web per la visualizzazione dello stato del cluster

Per impostazione predefinita, DC/OS in esecuzione nel servizio Azure Container include la piattaforma di orchestrazione Marathon per la pianificazione dei carichi di lavoro. La distribuzione di DC/OS del servizio ACS include tuttavia il pacchetto Mesosphere Universe di servizi che possono essere aggiunti al servizio, tra cui Spark, Hadoop, Cassandra e molto altro ancora.

![Universe DC/OS nel servizio Azure Container](media/dcos/universe.png)

#### <a name="using-marathon"></a>Uso di Marathon
Marathon è un sistema di init e controllo di tutto il cluster per i servizi in cgroups o, nel caso del servizio Azure Container, in contenitori formattati Docker. Marathon offre un'interfaccia utente Web da cui è possibile distribuire le applicazioni. È possibile accedervi tramite un URL simile a `http://DNS_PREFIX.REGION.cloudapp.azure.com`
dove DNS\_PREFIX e REGION vengono definiti in fase di distribuzione. Naturalmente, è anche possibile fornire il proprio nome DNS. Per altre informazioni sull'esecuzione di un contenitore tramite l'interfaccia utente Web di Marathon, vedere [Gestione di contenitori DC/OS tramite l'interfaccia utente Web di Marathon](container-service-mesos-marathon-ui.md).

![Elenco di applicazioni Marathon](media/dcos/marathon-applications-list.png)

È anche possibile usare le API REST per la comunicazione con Marathon. Esistono una serie di librerie client disponibili per ogni strumento. Coprono un'ampia gamma di linguaggi e, naturalmente, è possibile utilizzare il protocollo HTTP in qualsiasi linguaggio. Inoltre, molti strumenti comuni di DevOps forniscono il supporto per Marathon. Ciò fornisce flessibilità massima per il team operativo quando si lavora con un cluster del servizio Azure Container. Per altre informazioni sull'esecuzione di un contenitore tramite l'API REST di Marathon, vedere [Gestione di contenitori DC/OS tramite l'API REST di Marathon](container-service-mesos-marathon-rest.md).

### <a name="using-docker-swarm"></a>Utilizzo di Docker Swarm
Docker Swarm fornisce clustering nativo per Docker. Poiché Docker Swarm serve l'API Docker standard, gli strumenti che già comunicano con un daemon Docker possono usare Swarm per la scalabilità trasparente a più host nel servizio Azure Container.

![servizio Azure Container configurato per l'uso di Swarm.](media/acs-intro/acs-swarm2.png)

[!INCLUDE [container-service-swarm-mode-note](../../../includes/container-service-swarm-mode-note.md)]

Gli strumenti supportati per la gestione dei contenitori in un cluster Swarm includono in particolare quanto segue:

* Dokku
* Docker CLI e Docker Compose
* Krane
* Jenkins

### <a name="using-kubernetes"></a>Uso di Kubernetes
Kubernetes è un diffuso strumento open source di livello di produzione per l'orchestrazione di contenitori. Kubernetes automatizza la distribuzione, il ridimensionamento e la gestione delle applicazioni nei contenitori. Trattandosi di una soluzione open source che dipende dalla community open source, Kubernetes funziona perfettamente nel servizio Azure Container e permette di distribuirvi contenitori su larga scala.

![servizio Azure Container configurato per l'uso di Kubernetes.](media/acs-intro/kubernetes.png)

Dispone di un set completo di funzionalità tra cui:
* Scalabilità orizzontale
* Bilanciamento del carico e rilevamento del servizio
* Gestione della configurazione e segreti
* Implementazioni e ripristini automatizzati basati sull'API
* Riparazione automatica

## <a name="videos"></a>Video
Introduzione al servizio Azure Container (101):  

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Azure-Container-Service-101/player]
>
>

Compilazione di applicazioni tramite il servizio Azure Container (build 2016)

> [!VIDEO https://channel9.msdn.com/Events/Build/2016/B822/player]
>
>

## <a name="next-steps"></a>Passaggi successivi

Distribuire un cluster del servizio contenitore usando il [portale](container-service-deployment.md) o l'[interfaccia della riga di comando di Azure](container-service-create-acs-cluster-cli.md).
