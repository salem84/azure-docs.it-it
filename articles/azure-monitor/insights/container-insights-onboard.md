---
title: Come abilitare monitoraggio di Azure per i contenitori | Microsoft Docs
description: Questo articolo descrive come abilitare e configurare monitoraggio di Azure per i contenitori, in modo da comprendere il modo in cui il contenitore sta eseguendo e quali sono stati identificati i problemi relativi alle prestazioni.
ms.service: azure-monitor
ms.subservice: ''
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 07/12/2019
ms.openlocfilehash: 44cdc2d6b93ac9a62f96875ca6c679fbb97d85a9
ms.sourcegitcommit: ae461c90cada1231f496bf442ee0c4dcdb6396bc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/17/2019
ms.locfileid: "72555393"
---
# <a name="how-to-enable-azure-monitor-for-containers"></a>Come abilitare monitoraggio di Azure per i contenitori

Questo articolo offre una panoramica delle opzioni disponibili per configurare monitoraggio di Azure per i contenitori per monitorare le prestazioni dei carichi di lavoro distribuiti negli ambienti Kubernetes e ospitati nel [servizio Kubernetes di Azure](https://docs.microsoft.com/azure/aks/).

Monitoraggio di Azure per i contenitori può essere abilitato per le nuove distribuzioni oppure per una o più distribuzioni esistenti del servizio Azure Kubernetes usando i metodi supportati seguenti:

* Dal portale di Azure, Azure PowerShell o con l'interfaccia della riga di comando di Azure
* Usando [Terraform e il servizio Azure Kubernetes](../../terraform/terraform-create-k8s-cluster-with-tf-and-aks.md)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerequisiti
Prima di iniziare, verificare di disporre degli elementi seguenti:

* **Area di lavoro Log Analytics.**

    Il monitoraggio di Azure per i contenitori supporta un'area di lavoro Log Analytics nelle aree elencate in prodotti Azure in [base all'area](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor).

    È possibile creare un'area di lavoro quando si Abilita il monitoraggio del nuovo cluster AKS o lasciare che l'esperienza di onboarding crei un'area di lavoro predefinita nel gruppo di risorse predefinito della sottoscrizione del cluster AKS. Se si è scelto di crearla in autonomia, è possibile procedere attraverso [Azure Resource Manager](../platform/template-workspace-configuration.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) o il [portale di Azure](../learn/quick-create-workspace.md). Per un elenco delle coppie di mapping supportate usate per l'area di lavoro predefinita, vedere [mapping delle aree per il monitoraggio di Azure per i contenitori](container-insights-region-mapping.md).

* L'utente è membro del **ruolo di collaboratore log Analytics** per abilitare il monitoraggio dei contenitori. Per altre informazioni su come controllare l'accesso a un'area di lavoro Log Analytics, vedere [Gestire le aree di lavoro](../platform/manage-access.md).

* Si è un membro del ruolo **[proprietario](../../role-based-access-control/built-in-roles.md#owner)** sulla risorsa cluster AKS.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

* Le metriche Prometeo non vengono raccolte per impostazione predefinita. Prima di [configurare l'agente](container-insights-agent-config.md) per raccoglierli, è importante esaminare la [documentazione](https://prometheus.io/) di Prometeo per capire cosa è possibile definire.

## <a name="components"></a>Componenti

La capacità di monitorare le prestazioni si basa su un agente di Log Analytics per Linux incluso in un contenitore sviluppato appositamente per Monitoraggio di Azure per i contenitori. L'agente specializzato raccoglie i dati su prestazioni ed eventi da tutti i nodi del cluster e viene distribuito e registrato automaticamente con l'area di lavoro Log Analytics specificata durante la distribuzione. La versione dell'agente è microsoft/oms:ciprod04202018 o versione successiva ed è rappresentata da una data nel formato seguente: *mmggaaaa*.

>[!NOTE]
>Con la versione di anteprima del supporto di Windows Server per AKS, per un cluster AKS con nodi di Windows Server non è installato un agente per la raccolta di dati e l'invio a monitoraggio di Azure. Al contrario, un nodo Linux distribuito automaticamente nel cluster come parte della distribuzione standard raccoglie e invia i dati a monitoraggio di Azure per conto di tutti i nodi Windows del cluster.  
>

Quando viene rilasciata una nuova versione dell'agente, questo viene aggiornato automaticamente nei cluster Kubernetes gestiti ospitati nel servizio Azure Kubernetes. Per seguire le versioni rilasciate, consultare gli [annunci relativi alla versione dell'agente](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).

>[!NOTE]
>Se è già stato distribuito un cluster servizio Azure Kubernetes, si abilita il monitoraggio usando l'interfaccia della riga di comando di Azure o un modello di Azure Resource Manager fornito, come illustrato più avanti in questo articolo. Non è possibile usare `kubectl` per aggiornare, eliminare, distribuire o ridistribuire l'agente.
>Il modello deve essere distribuito nello stesso gruppo di risorse del cluster.

Per abilitare monitoraggio di Azure per i contenitori, usare uno dei metodi seguenti descritti nella tabella seguente.

| Stato della distribuzione | Metodo | Description |
|------------------|--------|-------------|
| Nuovo cluster AKS | [Creare cluster usando l'interfaccia della riga di comando di Azure](../../aks/kubernetes-walkthrough.md#create-aks-cluster)| È possibile abilitare il monitoraggio di un nuovo cluster AKS creato con l'interfaccia della riga di comando di Azure. |
| | [Creare un cluster usando la bonifica](container-insights-enable-new-cluster.md#enable-using-terraform)| È possibile abilitare il monitoraggio di un nuovo cluster AKS creato usando lo strumento open source per la bonifica. |
| Cluster AKS esistente | [Abilitare l'interfaccia della riga di comando di Azure](container-insights-enable-existing-clusters.md#enable-using-azure-cli) | È possibile abilitare il monitoraggio di un cluster AKS già distribuito usando l'interfaccia della riga di comando di Azure. |
| |[Abilita utilizzo di bonifica](container-insights-enable-existing-clusters.md#enable-using-terraform) | È possibile abilitare il monitoraggio di un cluster AKS già distribuito usando lo strumento open source per la bonifica. |
| | [Abilita da monitoraggio di Azure](container-insights-enable-existing-clusters.md#enable-from-azure-monitor-in-the-portal)| È possibile abilitare il monitoraggio di uno o più cluster AKS già distribuiti dalla pagina del multicluster AKS in monitoraggio di Azure. |
| | [Abilita dal cluster AKS](container-insights-enable-existing-clusters.md#enable-directly-from-aks-cluster-in-the-portal)| È possibile abilitare il monitoraggio direttamente da un cluster AKS nel portale di Azure. |
| | [Abilitare l'uso di un modello di Azure Resource Manager](container-insights-enable-existing-clusters.md#enable-using-an-azure-resource-manager-template)| È possibile abilitare il monitoraggio di un cluster AKS con un modello di Azure Resource Manager preconfigurato. |

## <a name="next-steps"></a>Passaggi successivi

* Con il monitoraggio abilitato per acquisire le metriche di integrità sia per i nodi del cluster servizio Azure Kubernetes che per i pod, queste metriche di integrità sono disponibili nel portale di Azure. Per informazioni su come usare Monitoraggio di Azure per contenitori, vedere [Visualizzare l'integrità del servizio Azure Kubernetes](container-insights-analyze.md).
