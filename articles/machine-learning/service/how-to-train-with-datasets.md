---
title: Eseguire il training con azureml-DataSets
titleSuffix: Azure Machine Learning service
description: Informazioni su come usare i set di impostazioni nel training
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 09/25/2019
ms.openlocfilehash: 9ccc5f5721d1ddc8459918913a4f3ce707766dea
ms.sourcegitcommit: 9fba13cdfce9d03d202ada4a764e574a51691dcd
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/26/2019
ms.locfileid: "71316700"
---
# <a name="train-with-datasets-preview-in-azure-machine-learning"></a>Eseguire il training con DataSet (anteprima) in Azure Machine Learning

In questo articolo vengono illustrati i due modi per utilizzare [Azure Machine Learning set](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py) di dati in un training di esperimento remoto senza preoccuparsi delle stringhe di connessione o dei percorsi di dati.

- Opzione 1: Se sono presenti dati strutturati, creare un TabularDataset e usarlo direttamente nello script di training.

- Opzione 2: Se si dispone di dati non strutturati, creare un file filedataset e montarli o scaricarli in un computer remoto per il training.

Azure Machine Learning [set di impostazioni](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.hyperdrive?view=azure-ml-py)forniscono un'integrazione perfetta con Azure machine learning prodotti di formazione come [scriptrun](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrun?view=azure-ml-py), [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py) e iperguida.

## <a name="prerequisites"></a>Prerequisiti

Per creare ed eseguire il training con i set di impostazioni, è necessario:

* Una sottoscrizione di Azure. Se non è disponibile una sottoscrizione di Azure, creare un account gratuito prima di iniziare. Provare subito la [versione gratuita o a pagamento del servizio Azure Machine Learning](https://aka.ms/AMLFree).

* Un' [area di lavoro del servizio Azure Machine Learning](how-to-manage-workspace.md).

* [SDK Azure Machine Learning per Python installato](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py), che include il pacchetto azureml-DataSets.

> [!Note]
> Alcune classi del set di dati (anteprima) presentano dipendenze dal pacchetto [azureml-dataprep](https://docs.microsoft.com/python/api/azureml-dataprep/?view=azure-ml-py) . Per gli utenti Linux queste classi sono supportate solo nelle distribuzioni seguenti:  Red Hat Enterprise Linux, Ubuntu, Fedora e CentOS.

## <a name="option-1-use-datasets-directly-in-training-scripts"></a>Opzione 1: Usare i set di impostazioni direttamente negli script di training

In questo esempio si crea un [TabularDataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py) e lo si usa come input diretto all'oggetto per `estimator` il training. 

### <a name="create-a-tabulardataset"></a>Creare un TabularDataset

Il codice seguente crea un TabularDataset non registrato da un URL Web. È anche possibile creare set di dati da file o percorsi locali negli archivi dati. Altre informazioni su [come creare set di impostazioni](https://aka.ms/azureml/howto/createdatasets).

```Python
from azureml.core.dataset import Dataset

web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path)
```

### <a name="access-the-input-dataset-in-your-training-script"></a>Accedere al set di dati di input nello script di training

Gli oggetti TabularDataset offrono la possibilità di caricare i dati in un frame di dati Pandas o Spark in modo che sia possibile lavorare con le librerie di formazione e preparazione dei dati note. Per sfruttare questa funzionalità, è possibile passare un TabularDataset come input nella configurazione di training e quindi recuperarlo nello script.

A tale scopo, accedere al set di dati di [`Run`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run.run?view=azure-ml-py) input tramite l'oggetto nello script di training [`to_pandas_dataframe()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py#to-pandas-dataframe--) e usare il metodo. 

```Python
%%writefile $script_folder/train_titanic.py

from azureml.core import Dataset, Run

run = Run.get_context()
# get the input dataset by name
dataset = run.input_datasets['titanic_ds']
# load the TabularDataset to pandas DataFrame
df = dataset.to_pandas_dataframe()
```

### <a name="configure-the-estimator"></a>Configurare lo strumento di stima

Viene usato un oggetto [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py) per inviare l'esecuzione dell'esperimento. Azure Machine Learning dispone di estimatori preconfigurati per i Framework di apprendimento automatico comuni, oltre a uno strumento di stima generico.

Questo codice crea un oggetto estimatore generico `est`,, che specifica

* Directory di script per gli script. Tutti i file in questa directory vengono caricati nei nodi del cluster per l'esecuzione.
* Script di training, *train_titanic. py*.
* Set di dati di input per `titanic`il training.
* Destinazione di calcolo per l'esperimento.
* Definizione dell'ambiente per l'esperimento.

```Python
est = Estimator(source_directory=script_folder,
                entry_script='train_titanic.py',
                # pass dataset object as an input with name 'titanic'
                inputs=[titanic_ds.as_named_input('titanic')],
                compute_target=compute_target,
                environment_definition= conda_env)

# Submit the estimator as part of your experiment run
experiment_run = experiment.submit(est)
experiment_run.wait_for_completion(show_output=True)
```

## <a name="option-2--mount-files-to-a-remote-compute-target"></a>Opzione 2:  Montare i file in una destinazione di calcolo remota

Se si desidera rendere disponibili i file di dati nella destinazione di calcolo per il training, utilizzare [filedataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.file_dataset.filedataset?view=azure-ml-py) per montare o scaricare i file a cui fa riferimento.

Quando si monta una file system, si collega tale file system a una directory (punto di montaggio) e la si rende disponibile nella destinazione di calcolo. Il montaggio è istantaneo perché i file vengono caricati solo al momento dell'elaborazione. Il montaggio è supportato e consigliato per i calcoli basati su Linux, tra cui Azure Machine Learning calcolo, macchine virtuali e HDInsight. Per il calcolo non basato su Linux, è supportato solo il download.  

>[!WARNING]
> Se le dimensioni dei dati superano il limite di archiviazione della destinazione di calcolo, il download avrà esito negativo.

### <a name="create-a-filedataset"></a>Creare un filedataset

Nell'esempio seguente viene creato un oggetto filedataset non registrato da URL Web. Altre informazioni su [come creare set di impostazioni](https://aka.ms/azureml/howto/createdatasets) da altre origini.

```Python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
mnist_ds = Dataset.File.from_files(path = web_paths)
```

### <a name="configure-the-estimator"></a>Configurare lo strumento di stima

Anziché passare il set di dati tramite `inputs` il parametro nello strumento di stima, è anche possibile passare il `script_params` set di dati tramite e ottenere il percorso dati (punto di montaggio) nello script di training tramite argomenti. In questo modo, è possibile accedere ai dati e usare uno script di training esistente.

Un oggetto [SKLearn](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.sklearn.sklearn?view=azure-ml-py) Estimator viene usato per inviare l'esecuzione per gli esperimenti Scikit-learn. Scopri di più sul training con [SKlearn Estimator](how-to-train-scikit-learn.md).

```Python
from azureml.train.sklearn import SKLearn

script_params = {
    # mount the dataset on the remote compute and pass the mounted path as an argument to the training script
    '--data-folder': mnist_ds.as_named_input('mnist').as_mount(),
    '--regularization': 0.5
}

est = SKLearn(source_directory=script_folder,
              script_params=script_params,
              compute_target=compute_target,
              environment_definition=env,
              entry_script='train_mnist.py')

# Run the experiment
run = experiment.submit(est)
run.wait_for_completion(show_output=True)
```

### <a name="retrieve-the-data-in-your-training-script"></a>Recuperare i dati nello script di training

Dopo aver inviato l'esecuzione, i file di dati a `mnist` cui fa riferimento il set di dati verranno montati nella destinazione di calcolo. Nel codice seguente viene illustrato come recuperare i dati nello script.

```Python
%%writefile $script_folder/train_mnist.py

import argparse
import os
import numpy as np
import glob

from utils import load_data

# retrieve the 2 arguments configured through script_params in estimator
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# get the file paths on the compute
X_train_path = glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0]
X_test_path = glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0]
y_train_path = glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0]
y_test = glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0]

# load train and test set into numpy arrays
X_train = load_data(X_train_path, False) / 255.0
X_test = load_data(X_test_path, False) / 255.0
y_train = load_data(y_train_path, True).reshape(-1)
y_test = load_data(y_test, True).reshape(-1)
```

## <a name="notebook-examples"></a>Esempi di notebook

I [notebook del set di dati](https://aka.ms/dataset-tutorial) dimostrano ed espandono i concetti in questo articolo. 

## <a name="next-steps"></a>Passaggi successivi

* Eseguire il [training automatico dei modelli di Machine Learning](how-to-auto-train-remote.md) con TabularDatasets.

* Eseguire il [training dei modelli di classificazione delle immagini](https://aka.ms/filedataset-samplenotebook) con i set di dati.

