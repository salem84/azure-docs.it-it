---
title: 'Esercitazione: Preparare un registro contenitori per Istanze di Azure Container'
description: Esercitazione di Istanze di Azure Container - Parte 2 di 3 - Preparare un Registro Azure Container ed eseguire il push di un'immagine
services: container-instances
author: dlepow
manager: gwallace
ms.service: container-instances
ms.topic: tutorial
ms.date: 03/21/2018
ms.author: danlep
ms.custom: seodec18, mvc
ms.openlocfilehash: b3c907eacb14ed65410a60fcf22ebe99fd8cc3bb
ms.sourcegitcommit: 4b431e86e47b6feb8ac6b61487f910c17a55d121
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/18/2019
ms.locfileid: "68325614"
---
# <a name="tutorial-deploy-an-azure-container-registry-and-push-a-container-image"></a>Esercitazione: Distribuire un Registro Azure Container ed eseguire il push di un'immagine del contenitore

Questa è la parte due di un'esercitazione in tre parti. Nella [prima parte](container-instances-tutorial-prepare-app.md) dell'esercitazione è stata creata un'immagine del contenitore Docker per un'applicazione Web Node.js. In questa esercitazione si esegue il push dell'immagine in Registro Azure Container. Se l'immagine del contenitore non è ancora stata creata, tornare all'esercitazione [1: Creare un'immagine del contenitore](container-instances-tutorial-prepare-app.md).

Registro Azure Container è il registro Docker privato in Azure. In questa esercitazione si crea un'istanza di Registro Azure Container nella sottoscrizione e quindi si esegue il push dell'immagine del contenitore creata in precedenza in tale istanza. In questo articolo, che corrisponde alla seconda parte della serie, è possibile eseguire queste operazioni:

> [!div class="checklist"]
> * Creare un'istanza di Registro Azure Container
> * Assegnare un tag all'immagine del contenitore per il Registro Azure Container
> * Caricare l'immagine nel registro

Nell'articolo successivo, che corrisponde all'ultimo della serie, il contenitore viene distribuito dal registro privato a Istanze di Azure Container.

## <a name="before-you-begin"></a>Prima di iniziare

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="create-azure-container-registry"></a>Creare un'istanza di Registro Azure Container

Prima di creare il registro contenitori, è necessario un *gruppo di risorse* in cui eseguirne la distribuzione. Un gruppo di risorse è una raccolta logica in cui vengono distribuite e gestite tutte le risorse di Azure.

Creare un gruppo di risorse con il comando [az group create][az-group-create]. Nell'esempio seguente viene creato un gruppo di risorse denominato *myResourceGroup* nell'area *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

Dopo aver creato il gruppo di risorse, creare un'istanza di Registro Azure Container con il comando [az acr create][az-acr-create]. Il nome del registro contenitori deve essere univoco in Azure e contenere da 5 a 50 caratteri alfanumerici. Sostituire `<acrName>` con un nome univoco per il registro:

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic --admin-enabled true
```

Di seguito è riportato l'output di esempio (qui troncato) per una nuova istanza di Registro Azure Container denominata *mycontainerregistry082*:

```console
$ az acr create --resource-group myResourceGroup --name mycontainerregistry082 --sku Basic --admin-enabled true
...
{
  "adminUserEnabled": true,
  "creationDate": "2018-03-16T21:54:47.297875+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/mycontainerregistry082",
  "location": "eastus",
  "loginServer": "mycontainerregistry082.azurecr.io",
  "name": "mycontainerregistry082",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

Nel resto dell'esercitazione viene usato `<acrName>` come segnaposto per il nome del registro contenitori scelto in questo passaggio.

## <a name="log-in-to-container-registry"></a>Accedere al registro contenitori

È necessario accedere all'istanza di Registro Azure Container prima di eseguire il push di immagini. Usare il comando [az acr login][az-acr-login] per completare l'operazione. È necessario specificare il nome univoco scelto per il registro contenitori al momento della creazione.

```azurecli
az acr login --name <acrName>
```

Al termine, il comando restituisce `Login Succeeded`:

```console
$ az acr login --name mycontainerregistry082
Login Succeeded
```

## <a name="tag-container-image"></a>Assegnare tag all'immagine del contenitore

Per eseguire il push di un'immagine del contenitore in un registro privato come Registro Azure Container, è prima necessario assegnare all'immagine un tag con il nome completo del server di accesso del registro.

Per prima cosa, ottenere il nome completo del server di accesso dell'istanza di Registro Azure Container. Eseguire questo comando [az acr show][az-acr-show], sostituendo `<acrName>` con il nome del registro appena creato:

```azurecli
az acr show --name <acrName> --query loginServer --output table
```

Se il nome del registro è *mycontainerregistry082*, ad esempio:

```console
$ az acr show --name mycontainerregistry082 --query loginServer --output table
Result
------------------------
mycontainerregistry082.azurecr.io
```

Visualizzare quindi l'elenco delle immagini locali con il comando [docker images][docker-images]:

```bash
docker images
```

Insieme a tutte le altre immagini presenti nel computer, verrà visualizzata l'immagine *aci-tutorial-app* creata nell'[esercitazione precedente](container-instances-tutorial-prepare-app.md):

```console
$ docker images
REPOSITORY          TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app    latest    5c745774dfa9    39 minutes ago    68.1 MB
```

Applicare all'immagine *aci-tutorial-app* il tag loginServer del registro contenitori. Aggiungere anche il tag `:v1` alla fine del nome dell'immagine per indicarne il numero di versione. Sostituire `<acrLoginServer>` con il risultato del comando [az acr show][az-acr-show] eseguito in precedenza.

```bash
docker tag aci-tutorial-app <acrLoginServer>/aci-tutorial-app:v1
```

Eseguire di nuovo `docker images` per verificare l'operazione di assegnazione di tag:

```console
$ docker images
REPOSITORY                                            TAG       IMAGE ID        CREATED           SIZE
aci-tutorial-app                                      latest    5c745774dfa9    39 minutes ago    68.1 MB
mycontainerregistry082.azurecr.io/aci-tutorial-app    v1        5c745774dfa9    7 minutes ago     68.1 MB
```

## <a name="push-image-to-azure-container-registry"></a>Eseguire il push dell'immagine in Registro Azure Container

Dopo aver assegnato all'immagine *aci-tutorial-app* il tag con il nome completo del server di accesso del registro privato, è possibile eseguirne il push nel registro con il comando [docker push][docker-push]. Sostituire `<acrLoginServer>` con il nome completo del server di accesso ottenuto nel passaggio precedente.

```bash
docker push <acrLoginServer>/aci-tutorial-app:v1
```

L'operazione `push` dovrebbe richiedere da alcuni secondi a qualche minuto a seconda della connessione Internet e l'output è simile al seguente:

```console
$ docker push mycontainerregistry082.azurecr.io/aci-tutorial-app:v1
The push refers to a repository [mycontainerregistry082.azurecr.io/aci-tutorial-app]
3db9cac20d49: Pushed
13f653351004: Pushed
4cd158165f4d: Pushed
d8fbd47558a8: Pushed
44ab46125c35: Pushed
5bef08742407: Pushed
v1: digest: sha256:ed67fff971da47175856505585dcd92d1270c3b37543e8afd46014d328f05715 size: 1576
```

## <a name="list-images-in-azure-container-registry"></a>Elencare le immagini in Registro Azure Container

Per verificare che l'immagine di cui è appena stato eseguito il push si trovi effettivamente nell'istanza di Registro Azure Container, visualizzare l'elenco delle immagini nel registro con il comando [az acr repository list][az-acr-repository-list]. Sostituire `<acrName>` con il nome del registro contenitori.

```azurecli
az acr repository list --name <acrName> --output table
```

Ad esempio:

```console
$ az acr repository list --name mycontainerregistry082 --output table
Result
----------------
aci-tutorial-app
```

Per visualizzare i *tag* per un'immagine specifica, usare il comando [az acr repository show-tags][az-acr-repository-show-tags].

```azurecli
az acr repository show-tags --name <acrName> --repository aci-tutorial-app --output table
```

L'output dovrebbe essere simile al seguente:

```console
$ az acr repository show-tags --name mycontainerregistry082 --repository aci-tutorial-app --output table
Result
--------
v1
```

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione è stata preparata un'istanza di Registro Azure Container da usare con Istanze di Azure Container ed è stato eseguito il push di un'immagine del contenitore nel registro. Sono stati completati i passaggi seguenti:

> [!div class="checklist"]
> * Distribuzione di un'istanza di Registro Azure Container
> * Assegnazione di un tag all'immagine del contenitore per Registro Azure Container
> * Caricamento di un'immagine in Registro Azure Container

Passare all'esercitazione successiva per apprendere come distribuire il contenitore in Azure usando Istanze di Azure Container:

> [!div class="nextstepaction"]
> [Distribuire un contenitore in Istanze di Azure Container](container-instances-tutorial-deploy-app.md)

<!-- LINKS - External -->
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: https://nodejs.org

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-repository-list]: /cli/azure/acr/repository
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-group-create]: /cli/azure/group#az-group-create
[azure-cli-install]: /cli/azure/install-azure-cli
