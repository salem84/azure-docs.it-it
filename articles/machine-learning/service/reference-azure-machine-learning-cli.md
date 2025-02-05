---
title: Estensione dell'interfaccia della riga di comando di Machine Learning
titleSuffix: Azure Machine Learning
description: Informazioni sull'estensione dell'interfaccia della riga di comando di Azure Machine Learning per l'interfaccia della riga di comando di Azure. L'interfaccia della riga di comando di Azure è un'utilità della riga di comando multipiattaforma che consente di usare le risorse nel cloud di Azure. L'estensione Machine Learning consente di usare Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: jordane
author: jpe316
ms.date: 08/20/2019
ms.custom: seodec18
ms.openlocfilehash: 85ebcc36d32b86ec2640ce7ce02190deaab19d6b
ms.sourcegitcommit: 1752581945226a748b3c7141bffeb1c0616ad720
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/14/2019
ms.locfileid: "70997089"
---
# <a name="use-the-cli-extension-for-azure-machine-learning"></a>Usare l'estensione CLI per Azure Machine Learning

L'interfaccia della riga di comando di Azure Machine Learning è un'estensione dell'[interfaccia della riga di comando di Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest), un'interfaccia della riga di comando multipiattaforma per la piattaforma Azure. Questa estensione fornisce i comandi per lavorare con Azure Machine Learning. Consente di automatizzare le attività di machine learning. L'elenco seguente fornisce alcune azioni di esempio che è possibile eseguire con l'estensione CLI:

+ Eseguire gli esperimenti per creare modelli di apprendimento automatico

+ Registrare modelli di Machine Learning per l'uso da parte dei clienti

+ Creare un pacchetto, distribuire e monitorare il ciclo di vita dei modelli di Machine Learning

L'interfaccia della riga di comando non sostituisce Azure Machine Learning SDK. Si tratta di uno strumento complementare ottimizzato per la gestione di attività con parametri molto adatti all'automazione.

## <a name="prerequisites"></a>Prerequisiti

* Per usare l'interfaccia della riga di comando, è necessario avere una sottoscrizione di Azure. Se non è disponibile una sottoscrizione di Azure, creare un account gratuito prima di iniziare. Prova subito la [versione gratuita o a pagamento di Azure Machine Learning](https://aka.ms/AMLFree) .

* [Interfaccia della riga di comando di Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).

## <a name="full-reference-docs"></a>Documentazione di riferimento completa

Trovare la [documentazione completa di riferimento per l'estensione Azure-CLI-ml dell'interfaccia della riga di comando di Azure](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/?view=azure-cli-latest).

## <a name="install-the-extension"></a>Installa l'estensione

Per installare l'estensione dell'interfaccia della riga di comando di Azure Machine Learning, usare il comando seguente:

```azurecli-interactive
az extension add -n azure-cli-ml
```

> [!TIP]
> È possibile trovare i file di esempio che è possibile usare con i [comandi riportati di seguito.](https://aka.ms/azml-deploy-cloud)

Quando richiesto, selezionare `y` per installare l'estensione.

Per verificare che l'estensione sia stata installata, usare il comando seguente per visualizzare un elenco di sottocomandi specifici di ML:

```azurecli-interactive
az ml -h
```

## <a name="update-the-extension"></a>Aggiornare l'estensione

Per aggiornare l'estensione dell'interfaccia della riga di comando Machine Learning, usare il comando seguente:

```azurecli-interactive
az extension update -n azure-cli-ml
```


## <a name="remove-the-extension"></a>Rimuovere l'estensione

Per rimuovere l'estensione dell'interfaccia della riga di comando, usare il comando seguente:

```azurecli-interactive
az extension remove -n azure-cli-ml
```

## <a name="resource-management"></a>Resource management

I comandi seguenti illustrano come usare l'interfaccia della riga di comando per gestire le risorse usate da Azure Machine Learning.

+ Se non ne è già presente uno, creare un gruppo di risorse:

    ```azurecli-interactive
    az group create -n myresourcegroup -l westus2
    ```

+ Creare un'area di lavoro Azure Machine Learning:

    ```azurecli-interactive
    az ml workspace create -w myworkspace -g myresourcegroup
    ```

    Per ulteriori informazioni, vedere [AZ ml Workspace create](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-create).

+ Alleghi la configurazione di un'area di lavoro a una cartella per abilitare la consapevolezza contestuale CLI.

    ```azurecli-interactive
    az ml folder attach -w myworkspace -g myresourcegroup
    ```

    Questo comando crea una `.azureml` sottodirectory che contiene i file dell'ambiente runconfig e conda di esempio. Contiene anche un `config.json` file usato per comunicare con l'area di lavoro Azure Machine Learning.

    Per ulteriori informazioni, vedere [AZ ml Folder Connetti](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/folder?view=azure-cli-latest#ext-azure-cli-ml-az-ml-folder-attach).

+ Alleghi un contenitore BLOB di Azure come archivio dati.

    ```azurecli-interactive
    az ml datastore attach-blob  -n datastorename -a accountname -c containername
    ```

    Per altre informazioni, vedere [AZ ml datastore allegato-BLOB](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/datastore?view=azure-cli-latest#ext-azure-cli-ml-az-ml-datastore-attach-blob).

+ Caricare i file in un archivio dati.

    ```azurecli-interactive
    az ml datastore upload  -n datastorename -p sourcepath
    ```

    Per altre informazioni, vedere [AZ ml datastore upload](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/datastore?view=azure-cli-latest#ext-azure-cli-ml-az-ml-datastore-upload).

+ Alleghi un cluster AKS come destinazione di calcolo.

    ```azurecli-interactive
    az ml computetarget attach aks -n myaks -i myaksresourceid -g myresourcegroup -w myworkspace
    ```

    Per altre informazioni, vedere [AZ ml computetarget allegato AKS](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-attach-aks)

+ Creare una nuova destinazione AMLcompute.

    ```azurecli-interactive
    az ml computetarget create amlcompute -n cpu --min-nodes 1 --max-nodes 1 -s STANDARD_D3_V2
    ```

    Per altre informazioni, vedere [AZ ml computetarget create amlcompute](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest#ext-azure-cli-ml-az-ml-computetarget-create-amlcompute).

## <a id="experiments"></a>Eseguire esperimenti

* Avviare un'esecuzione dell'esperimento. Quando si usa questo comando, specificare il nome del file runconfig (il testo precedente \*a. runconfig se si sta osservando la file System) rispetto al parametro-c.

    ```azurecli-interactive
    az ml run submit-script -c sklearn -e testexperiment train.py
    ```

    > [!TIP]
    > Il `az ml folder attach` comando crea una `.azureml` sottodirectory che contiene due file runconfig di esempio. 
    >
    > Se si dispone di uno script Python che crea un oggetto di configurazione di esecuzione a livello di codice, è possibile usare [runconfig. Save ()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py#save-path-none--name-none--separate-environment-yaml-false-) per salvarlo come file runconfig.
    >
    > Per ulteriori esempi di file runconfig, [https://github.com/MicrosoftDocs/pipelines-azureml/tree/master/.azureml](https://github.com/MicrosoftDocs/pipelines-azureml/tree/master/.azureml)vedere.

    Per ulteriori informazioni, vedere [AZ ml Run Submit-script](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/run?view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-submit-script).

* Visualizzare un elenco di esperimenti:

    ```azurecli-interactive
    az ml experiment list
    ```

    Per altre informazioni, vedere [AZ ml Experiment list](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/experiment?view=azure-cli-latest#ext-azure-cli-ml-az-ml-experiment-list).

## <a name="environment-management"></a>Gestione dell'ambiente

I comandi seguenti illustrano come creare, registrare ed elencare Azure Machine Learning [ambienti](how-to-configure-environment.md) per l'area di lavoro:

+ Creare file di impalcatura per un ambiente:

    ```azurecli-interactive
    az ml environment scaffold -n myenv -d myenvdirectory
    ```

    Per altre informazioni, vedere [AZ ml Environment patibolo](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest#ext-azure-cli-ml-az-ml-environment-scaffold).

+ Registrare un ambiente:

    ```azurecli-interactive
    az ml environment register -d myenvdirectory
    ```

    Per ulteriori informazioni, vedere [AZ ml Environment Register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest#ext-azure-cli-ml-az-ml-environment-register).

+ Elencare gli ambienti registrati:

    ```azurecli-interactive
    az ml environment list
    ```

    Per ulteriori informazioni, vedere [AZ ml Environment list](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest#ext-azure-cli-ml-az-ml-environment-list).

+ Scaricare un ambiente registrato:

    ```azurecli-interactive
    az ml environment download -n myenv -d downloaddirectory
    ```

    Per altre informazioni, vedere [AZ ml Environment download](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest#ext-azure-cli-ml-az-ml-environment-download).

## <a name="model-registration-profiling-deployment"></a>Registrazione del modello, profilatura, distribuzione

I comandi seguenti illustrano come registrare un modello con training e quindi distribuirlo come servizio di produzione:

+ Registrare un modello con Azure Machine Learning:

    ```azurecli-interactive
    az ml model register -n mymodel -p sklearn_regression_model.pkl
    ```

    Per ulteriori informazioni, vedere [AZ ml Model Register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-register).

+ **Facoltativo** Profilare il modello per ottenere valori di CPU e memoria ottimali per la distribuzione.
    ```azurecli-interactive
    az ml model profile -n myprofile -m mymodel:1 --ic inferenceconfig.json -d "{\"data\": [[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}" -t myprofileresult.json
    ```

    Per altre informazioni, vedere [AZ ml Model Profile](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-profile).

+ Distribuire il modello in AKS
    ```azurecli-interactive
    az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json --ct akscomputetarget
    ```
    
    Per altre informazioni sullo schema del file di configurazione dell'inferenza, vedere [schema di configurazione di inferenza](#inferenceconfig).
    
    Per ulteriori informazioni sullo schema del file di configurazione della distribuzione, vedere [schema di configurazione della distribuzione](#deploymentconfig).

    Per altre informazioni, vedere [AZ ml Model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

<a id="inferenceconfig"></a>

## <a name="inference-configuration-schema"></a>Schema di configurazione dell'inferenza

[!INCLUDE [inferenceconfig](../../../includes/machine-learning-service-inference-config.md)]

<a id="deploymentconfig"></a>

## <a name="deployment-configuration-schema"></a>Schema di configurazione della distribuzione

### <a name="local-deployment-configuration-schema"></a>Schema di configurazione della distribuzione locale

[!INCLUDE [deploymentconfig](../../../includes/machine-learning-service-local-deploy-config.md)]

### <a name="azure-container-instance-deployment-configuration-schema"></a>Schema di configurazione della distribuzione dell'istanza di contenitore di Azure 

[!INCLUDE [deploymentconfig](../../../includes/machine-learning-service-aci-deploy-config.md)]

### <a name="azure-kubernetes-service-deployment-configuration-schema"></a>Schema di configurazione della distribuzione del servizio Kubernetes di Azure

[!INCLUDE [deploymentconfig](../../../includes/machine-learning-service-aks-deploy-config.md)]

## <a name="next-steps"></a>Passaggi successivi

* [Riferimento ai comandi per l'estensione CLI Machine Learning](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml?view=azure-cli-latest).

* [Eseguire il training e distribuire modelli di apprendimento automatico usando Azure Pipelines](/azure/devops/pipelines/targets/azure-machine-learning)
