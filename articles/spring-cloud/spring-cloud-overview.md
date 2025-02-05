---
title: Introduzione ad Azure Spring Cloud
description: Informazioni sulle funzionalità e i vantaggi di Azure Spring Cloud per la distribuzione e la gestione di applicazioni Java Spring in Azure.
author: jpconnock
ms.service: spring-cloud
ms.topic: overview
ms.date: 10/04/2019
ms.author: jeconnoc
ms.openlocfilehash: c58e6b2a0c0de8295df65b44fbdeaeb2d4461948
ms.sourcegitcommit: aef6040b1321881a7eb21348b4fd5cd6a5a1e8d8
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/09/2019
ms.locfileid: "72166525"
---
# <a name="what-is-azure-spring-cloud"></a>Cos'è Azure Spring Cloud?

Azure Spring Cloud semplifica la distribuzione di applicazioni di microservizi basate su Spring Boot in Azure, senza modificare il codice.  Azure Spring Cloud gestisce il ciclo di vita delle applicazioni Spring Cloud lasciando gli sviluppatori liberi di dedicarsi al loro codice.  Spring Cloud fornisce la gestione del ciclo di vita tramite funzionalità complete di monitoraggio e diagnostica, gestione della configurazione, individuazione dei servizi, integrazione di CI/CD, distribuzioni blu/verde e altro ancora.

Facendo parte dell'ecosistema di Azure, Azure Spring Cloud si può associare facilmente ad altri servizi di Azure, come archiviazione, database, monitoraggio e così via.

Azure Spring Cloud è attualmente disponibile in anteprima.  Per accedere a questo servizio durante la fase di anteprima, [compilare questo modulo](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR-LA2geqX-ZLhi-Ado1LD3tUNDk2VFpGUzYwVEJNVkhLRlcwNkZFUFZEUS4u
).

Per iniziare, completare l'argomento di avvio rapido su Spring Cloud usando l'[interfaccia della riga di comando di Azure](spring-cloud-quickstart-launch-app-cli.md), il [portale di Azure](spring-cloud-quickstart-launch-app-portal.md) oppure [Maven](spring-cloud-quickstart-launch-app-maven.md).

## <a name="application-configuration"></a>Configurazione dell'applicazione

### <a name="spring-cloud-config-server"></a>Spring Cloud Config Server

Spring Cloud Config Server fornisce la configurazione esternalizzata in un sistema distribuito con il supporto sia lato server che lato client.  Offre una posizione centrale da cui gestire le proprietà dell'applicazione in tutti gli ambienti.  Per altre informazioni, vedere le [informazioni di riferimento su Spring Cloud Config Server](https://spring.io/projects/spring-cloud-config.md) e completare l'esercitazione.

### <a name="enable-bluegreen-deployments"></a>Abilitare distribuzioni blu/verde

Azure Spring Cloud supporta distribuzioni blu/verde per il rilascio e l'aggiornamento di codice negli ambienti di produzione.  Sfruttando questo modello di gestione del cambiamento, gli sviluppatori possono implementare funzionalità e modifiche del codice con la sicurezza di un fallback immediato, se necessario.  Azure consente agli sviluppatori di concentrarsi sulla scrittura di codice gestendo più ambienti di produzione e semplificando l'aggiornamento o il rollback delle modifiche del codice senza interrompere l'applicazione.  Per altre informazioni sugli ambienti di staging e le distribuzioni blu/verde, vedere questa [guida pratica](spring-cloud-howto-staging-environment.md).

### <a name="automate-cicd-pipelines"></a>Automatizzare le pipeline CI/CD

Azure Spring Cloud prevede l'integrazione con Azure DevOps tramite l'interfaccia della riga di comando di Azure.  Usando Azure DevOps, è possibile automatizzare l'integrazione e la distribuzione del codice nell'applicazione Spring.  Per altre informazioni, vedere questo [articolo](spring-cloud-howto-cicd.md).

### <a name="scale-your-application"></a>Ridimensionare l'applicazione

Azure Spring Cloud consente di ridimensionare i microservizi nel dashboard di Azure Spring Cloud.  È possibile aumentare o ridurre sia il numero di vCPU che la quantità di memoria disponibile per i microservizi, in base a specifici requisiti.  Il ridimensionamento diventa effettivo dopo pochi secondi e non richiede modifiche del codice né la ridistribuzione.  Per altre informazioni, completare questa [esercitazione](spring-cloud-tutorial-scale-manual.md).

## <a name="application-monitoring"></a>Monitoraggio dell'applicazione

### <a name="monitor-your-application-using-distributed-tracing-and-azure-app-insights"></a>Monitorare l'applicazione con la traccia distribuita e Azure Application Insights

Gli strumenti di traccia distribuita di Spring Cloud consentono agli sviluppatori di eseguire il debug e il monitoraggio delle complesse interconnessioni tra i microservizi nella rispettiva applicazione.  Integrando [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) con Azure [Application Insights](../azure-monitor/insights/insights-overview.md), Azure offre potenti funzionalità di traccia distribuita dal portale di Azure.  Per altre informazioni, completare questa [esercitazione](spring-cloud-tutorial-distributed-tracing.md).

## <a name="next-steps"></a>Passaggi successivi

- [Avviare l'app Spring Cloud dall'interfaccia della riga di comando](spring-cloud-quickstart-launch-app-cli.md)
