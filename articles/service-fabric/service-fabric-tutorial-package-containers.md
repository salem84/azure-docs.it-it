---
title: Creare un pacchetto e distribuire contenitori come un'applicazione di Service Fabric in Azure | Microsoft Docs
description: In questa esercitazione si apprenderà come generare una definizione di applicazione di Azure Service Fabric usando Yeoman e creare il pacchetto dell'applicazione.
services: service-fabric
documentationcenter: ''
author: suhuruli
manager: chackdan
editor: suhuruli
tags: servicefabric
keywords: Docker, contenitori, microservizi, Service Fabric, Azure
ms.assetid: ''
ms.service: service-fabric
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/22/2019
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: b4931b62aaf04db2ee4ff6f4a5e2b96274b8af88
ms.sourcegitcommit: 04ec7b5fa7a92a4eb72fca6c6cb617be35d30d0c
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/22/2019
ms.locfileid: "68385047"
---
# <a name="tutorial-package-and-deploy-containers-as-a-service-fabric-application-using-yeoman"></a>Esercitazione: Creare un pacchetto e distribuire contenitori come un'applicazione di Service Fabric usando Yeoman

Questa è la seconda di una serie di esercitazioni. In questa esercitazione viene usato uno strumento generatore di modelli (Yeoman) per generare una definizione di applicazione di Service Fabric. Questa applicazione può quindi essere usata per distribuire i contenitori in Service Fabric. In questa esercitazione si apprenderà come:

> [!div class="checklist"]
> * Installare Yeoman
> * Creare un pacchetto dell'applicazione usando Yeoman
> * Configurare le impostazioni nel pacchetto dell'applicazione per l'uso con i contenitori
> * Compilare l'applicazione
> * Distribuire ed eseguire l'applicazione
> * Eseguire la pulizia dell'applicazione

## <a name="prerequisites"></a>Prerequisiti

* Vengono usate le immagini del contenitore di cui è stato eseguito il push in Registro Azure Container creato nella [prima](service-fabric-tutorial-create-container-images.md) di questa serie di esercitazioni.
* Viene [configurato](service-fabric-tutorial-create-container-images.md) l'ambiente di sviluppo Linux.

## <a name="install-yeoman"></a>Installare Yeoman

Service Fabric offre gli strumenti di scaffolding per la creazione di applicazioni dal terminale tramite il generatore di modelli Yeoman. Attenersi ai passaggi indicati di seguito per verificare di disporre del generatore di modelli Yeoman.

1. Installare nodejs e NPM nel computer. Si noti che gli utenti Mac OSX dovranno usare la gestione pacchetti Homebrew

    ```bash
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
    nvm install node 
    ```
2. Installare il generatore di modelli Yeoman nel computer da NPM

    ```bash
    npm install -g yo
    ```
3. Installare il generatore di contenitori Yeoman di Service Fabric

    ```bash 
    npm install -g generator-azuresfcontainer
    ```

## <a name="package-a-docker-image-container-with-yeoman"></a>Creare un pacchetto di un contenitore di un'immagine Docker con Yeoman

1. Per creare un'applicazione contenitore di Service Fabric, nella directory 'container-tutorial' del repository clonato eseguire questo comando.

    ```bash
    yo azuresfcontainer
    ```
2. Digitare "TestContainer" per assegnare un nome all'applicazione.
3. Digitare "azurevotefront" per assegnare un nome al servizio dell'applicazione.
4. Specificare il percorso dell'immagine del contenitore in Registro Azure Container del repository frontend, ad esempio '\<acrName&gt;.azurecr.io/azure-vote-front:v1'. Il valore del campo \<acrName> deve essere uguale a quello usato nell'esercitazione precedente.
5. Premere INVIO per lasciare vuota la sezione dei comandi.
6. Specificare 1 come numero di istanze.

Di seguito sono riportati l'input e l'output dell'esecuzione del comando yo:

```bash
? Name your application TestContainer
? Name of the application service: azurevotefront
? Input the Image Name: <acrName>.azurecr.io/azure-vote-front:v1
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/TestContainer/ApplicationManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/ServiceManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/config/Settings.xml
   create TestContainer/TestContainer/azurevotefrontPkg/code/Dummy.txt
   create TestContainer/install.sh
   create TestContainer/uninstall.sh
```

Per aggiungere un altro servizio contenitore a un'applicazione già creata usando Yeoman, seguire questa procedura:

1. Passare alla directory di livello superiore rispetto a **TestContainer**, ad esempio *./TestContainer*
2. Eseguire `yo azuresfcontainer:AddService`
3. Assegnare al servizio il nome 'azurevoteback'
4. Specificare il percorso dell'immagine del contenitore per Redis - 'alpine:redis'
5. Premere Invio per lasciare vuota la sezione dei comandi
6. Specificare "1" come numero di istanze.

Vengono visualizzate tutte le voci per l'aggiunta del servizio usato:

```bash
? Name of the application service: azurevoteback
? Input the Image Name: alpine:redis
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/azurevotebackPkg/ServiceManifest.xml
   create TestContainer/azurevotebackPkg/config/Settings.xml
   create TestContainer/azurevotebackPkg/code/Dummy.txt
```

Per il resto di questa esercitazione, si lavorerà nella directory **TestContainer**. ad esempio *./TestContainer/TestContainer*. Il contenuto di questa directory dovrebbe essere il seguente.

```bash
$ ls
ApplicationManifest.xml azurevotefrontPkg azurevotebackPkg
```

## <a name="configure-the-application-manifest-with-credentials-for-azure-container-registry"></a>Configurare il manifesto dell'applicazione con le credenziali per Registro Azure Container

Affinché Service Fabric possa eseguire il pull delle immagini del contenitore da Registro Azure Container, è necessario fornire le credenziali in **ApplicationManifest.xml**.

Accedere all'istanza di Registro Azure Container. Usare il comando **az acr login** per completare l'operazione. Specificare il nome univoco assegnato al registro contenitori al momento della creazione.

```bash
az acr login --name <acrName>
```

Al termine, il comando restituisce un messaggio di **Accesso riuscito**.

Eseguire quindi questo comando per ottenere la password del registro contenitori. Questa password viene usata da Service Fabric per l'autenticazione al Registro Azure Container per eseguire il pull delle immagini del contenitore.

```bash
az acr credential show -n <acrName> --query passwords[0].value
```

In **ApplicationManifest.xml** aggiungere il frammento di codice sotto l'elemento **ServiceManifestImport** per il servizio front-end. Inserire l'**acrName** per il campo **AccountName**; per il campo **Password** viene usata la password restituita dal comando precedente. Alla fine di questo documento viene fornito un file completo **ApplicationManifest.xml**.

```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code">
    <RepositoryCredentials AccountName="<acrName>" Password="<password>" PasswordEncrypted="false"/>
  </ContainerHostPolicies>
</Policies>
```

## <a name="configure-communication-and-container-port-to-host-port-mapping"></a>Configurare la comunicazione e il mapping tra porta del contenitore e porta dell'host

### <a name="configure-communication-port"></a>Configurare la porta di comunicazione

Configurare un endpoint HTTP in modo che i client possano comunicare con il servizio. Aprire il file *./TestContainer/azurevotefrontPkg/ServiceManifest.xml* e dichiarare una risorsa endpoint nell'elemento **ServiceManifest**.  Aggiungere il protocollo, la porta e il nome. Per questa esercitazione il servizio è in ascolto sulla porta 80. Il frammento codice seguente è inserito sotto il tag *ServiceManifest* nella risorsa.

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
  </Endpoints>
</Resources>

```

Analogamente, modificare il manifesto del servizio per il servizio back-end. Aprire il file *./TestContainer/azurevotebackPkg/ServiceManifest.xml* e dichiarare una risorsa endpoint nell'elemento **ServiceManifest**. Per questa esercitazione, viene mantenuta l'impostazione predefinita redis 6379. Il frammento codice seguente è inserito sotto il tag *ServiceManifest* nella risorsa.

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
  </Endpoints>
</Resources>
```

Se si specifica **UriScheme**, l'endpoint del contenitore viene registrato automaticamente con il servizio Service Fabric Naming per l'individuazione. Alla fine di questo articolo viene fornito un file di esempio completo di ServiceManifest.xml per il servizio back-end.

### <a name="map-container-ports-to-a-service"></a>Eseguire il mapping delle porte del contenitore a un servizio

Per esporre i contenitori nel cluster, è necessario anche creare un'associazione delle porte in 'ApplicationManifest.xml'. I criteri **PortBinding** fanno riferimento agli **Endpoint** definiti nei file **ServiceManifest.xml**. Le richieste in ingresso a questi endpoint vengono mappate alle porte del contenitore aperte e associate qui. Nel file **ApplicationManifest.xml** aggiungere il codice seguente per associare la porta 80 e 6379 agli endpoint. Alla fine di questo documento è disponibile un file completo **ApplicationManifest.xml**.

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
</ContainerHostPolicies>
```

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
</ContainerHostPolicies>
```

### <a name="add-a-dns-name-to-the-backend-service"></a>Aggiungere un nome DNS al servizio back-end

Affinché Service Fabric possa assegnare questo nome DNS al servizio back-end, è necessario specificare tale nome nel file **ApplicationManifest.xml**. Aggiungere l'attributo **ServiceDnsName** all'elemento **Service** come illustrato:

```xml
<Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
  <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
    <SingletonPartition/>
  </StatelessService>
</Service>
```

Il servizio front-end legge una variabile di ambiente per conoscere il nome DNS dell'istanza di Redis. Questa variabile di ambiente è già definita nel Dockerfile usato per generare l'immagine Docker, di conseguenza non è necessario eseguire alcuna operazione.

```Dockerfile
ENV REDIS redisbackend.testapp
```

Il frammento di codice seguente illustra in che modo il codice Python front-end preleva la variabile di ambiente descritta nel Dockerfile. Non è necessario eseguire alcuna operazione.

```python
# Get DNS Name
redis_server = os.environ['REDIS']

# Connect to the Redis store
r = redis.StrictRedis(host=redis_server, port=6379, db=0)
```

A questo punto dell'esercitazione il modello di un'applicazione del pacchetto di servizio è disponibile per la distribuzione in un cluster. Nell'esercitazione successiva questa applicazione viene distribuita ed eseguita in un cluster di Service Fabric.

## <a name="create-a-service-fabric-cluster"></a>Creare un cluster di Service Fabric

Per distribuire l'applicazione in Azure, è necessario un cluster di Service Fabric per eseguire l'applicazione. I comandi seguenti creano un cluster con cinque nodi in Azure.  Creano inoltre un certificato autofirmato, lo aggiungono a un insieme di credenziali delle chiavi e lo scaricano in locale come file PEM. Il nuovo certificato viene usato per proteggere il cluster in fase di distribuzione e per l'autenticazione dei client.

```azurecli
#!/bin/bash

# Variables
ResourceGroupName="containertestcluster" 
ClusterName="containertestcluster" 
Location="eastus" 
Password="q6D7nN%6ck@6" 
Subject="containertestcluster.eastus.cloudapp.azure.com" 
VaultName="containertestvault" 
VmPassword="Mypa$$word!321"
VmUserName="sfadminuser"

# Login to Azure and set the subscription
az login

az account set --subscription <mySubscriptionID>

# Create resource group
az group create --name $ResourceGroupName --location $Location 

# Create secure five node Linux cluster. Creates a key vault in a resource group
# and creates a certficate in the key vault. The certificate's subject name must match 
# the domain that you use to access the Service Fabric cluster.  
# The certificate is downloaded locally as a PEM file.
az sf cluster create --resource-group $ResourceGroupName --location $Location \ 
--certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject \ 
--cluster-name $ClusterName --cluster-size 5 --os UbuntuServer1604 --vault-name $VaultName \ 
--vault-resource-group $ResourceGroupName --vm-password $VmPassword --vm-user-name $VmUserName
```

> [!Note]
> Il servizio front-end Web è configurato per l'ascolto del traffico in ingresso sulla porta 80. Per impostazione predefinita, la porta 80 è aperta nelle VM del cluster e nel servizio di bilanciamento del carico di Azure.
>

Per altre informazioni sulla creazione di un cluster, vedere l'articolo su come [creare un cluster di Service Fabric in Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md).

## <a name="build-and-deploy-the-application-to-the-cluster"></a>Compilare e distribuire l'applicazione nel cluster

È possibile distribuire l'applicazione nel cluster di Azure usando l'interfaccia della riga di comando di Service Fabric. Se l'interfaccia della riga di comando di Service Fabric non è installata nel computer, seguire le istruzioni indicate [qui](service-fabric-get-started-linux.md#set-up-the-service-fabric-cli) per installarla.

Connettersi al cluster di Service Fabric in Azure. Sostituire l'endpoint di esempio con il proprio. L'endpoint deve essere un URL completo, simile al seguente.  Il file PEM è il certificato autofirmato creato in precedenza.

```bash
sfctl cluster select --endpoint https://containertestcluster.eastus.cloudapp.azure.com:19080 --pem containertestcluster22019013100.pem --no-verify
```

Usare lo script di installazione fornito nella directory **TestContainer** per copiare il pacchetto dell'applicazione nell'archivio immagini del cluster, registrare il tipo e creare un'istanza dell'applicazione.

```bash
./install.sh
```

Aprire un browser e passare a Service Fabric Explorer all'indirizzo http:\//containertestcluster.eastus.cloudapp.azure.com:19080/Explorer. Espandere il nodo delle applicazioni, nel quale sarà presente una voce per il tipo di applicazione e un'altra per l'istanza.

![Service Fabric Explorer][sfx]

Per connettersi all'applicazione in esecuzione, aprire un Web browser e passare all'URL del cluster, ad esempio http:\//containertestcluster.eastus.cloudapp.azure.com:80. Viene visualizzata l'applicazione di voto nell'interfaccia utente Web.

![votingapp][votingapp]

## <a name="clean-up"></a>Eseguire la pulizia

Usare lo script di disinstallazione incluso nel modello per eliminare l'istanza dell'applicazione dal cluster e annullare la registrazione del tipo di applicazione. Questo comando richiede del tempo per pulire l'istanza e il comando 'install.sh' non può essere eseguito immediatamente dopo questo script.

```bash
./uninstall.sh
```

## <a name="examples-of-completed-manifests"></a>Esempi di manifesti completati

### <a name="applicationmanifestxml"></a>ApplicationManifest.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ApplicationManifest ApplicationTypeName="TestContainerType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotefrontPkg" ServiceManifestVersion="1.0.0"/>
    <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myaccountname" Password="<password>" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
    </ContainerHostPolicies>
        </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotebackPkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
          <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
        </ContainerHostPolicies>
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="azurevotefront">
      <StatelessService ServiceTypeName="azurevotefrontType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
    <Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
      <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

### <a name="front-end-servicemanifestxml"></a>ServiceManifest.xml front-end

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotefrontPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotefrontType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>acrName.azurecr.io/azure-vote-front:v1</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
    </Endpoints>
  </Resources>

 </ServiceManifest>
```

### <a name="redis-servicemanifestxml"></a>ServiceManifest.xml Redis

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotebackPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotebackType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>alpine:redis</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>
     <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
    </Endpoints>
  </Resources>
 </ServiceManifest>
```

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione più contenitori sono stati inseriti nel pacchetto in un'applicazione di Service Fabric usando Yeoman. Questa applicazione è stata quindi distribuita ed eseguita in un cluster di Service Fabric. Sono stati completati i passaggi seguenti:

> [!div class="checklist"]
> * Installare Yeoman
> * Creare un pacchetto dell'applicazione usando Yeoman
> * Configurare le impostazioni nel pacchetto dell'applicazione per l'uso con i contenitori
> * Compilare l'applicazione
> * Distribuire ed eseguire l'applicazione
> * Eseguire la pulizia dell'applicazione

Passare alla prossima esercitazione per informazioni sul failover e il ridimensionamento dell'applicazione in Service Fabric.

> [!div class="nextstepaction"]
> [Learn about failover and scaling applications](service-fabric-tutorial-containers-failover.md) (Informazioni sul failover e il ridimensionamento delle applicazioni)

[votingapp]: ./media/service-fabric-tutorial-deploy-run-containers/votingapp.png
[sfx]: ./media/service-fabric-tutorial-deploy-run-containers/containerspackagetutorialsfx.png