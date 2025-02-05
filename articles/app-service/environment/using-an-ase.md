---
title: Usare un ambiente del servizio app - Azure
description: Come creare, pubblicare e ridimensionare le app in un ambiente del servizio app di Azure
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: a22450c4-9b8b-41d4-9568-c4646f4cf66b
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 05/28/2019
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: cabefcc53106a53459975fc26513dc59ae7d3372
ms.sourcegitcommit: 82499878a3d2a33a02a751d6e6e3800adbfa8c13
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70073218"
---
# <a name="use-an-app-service-environment"></a>Usare un ambiente del servizio app #

Ambiente del servizio app di Azure è una distribuzione di Servizio app di Azure in una subnet nella rete virtuale di Azure. È costituita da:

- **Front-end**: i front-end rappresentano il punto in cui HTTP/HTTPS termina in un ambiente del servizio app (ASE).
- **Ruoli di lavoro**: i ruoli di lavoro sono le risorse che ospitano le app.
- **Database**: il database contiene le informazioni che definiscono l'ambiente.
- **Archiviazione**: Le risorse di archiviazione vengono usate per ospitare le app pubblicate dai clienti.

> [!NOTE]
> Esistono due versioni dell'ambiente del servizio app: ASEv1 e ASEv2. In ASEv1 è necessario gestire le risorse prima di poterle usare. Per informazioni su come configurare e gestire ASEv1, vedere [configurare un ambiente del servizio app V1][ConfigureASEv1]. Il resto di questo articolo è incentrato su ASEv2.
>
>

È possibile distribuire un ambiente del servizio app (sia ASEv1 che ASEv2) con un indirizzo VIP esterno o interno per l'accesso app. La distribuzione con un indirizzo VIP esterno viene comunemente definita come un ambiente del servizio app esterno. La versione interna viene chiamata ambiente del servizio app ILB, poiché usa un servizio di bilanciamento del carico interno (ILB). Per altre informazioni sull'ambiente del servizio app ILB, vedere [creare e usare un][MakeILBASE]ambiente del servizio app ILB.

## <a name="create-an-app-in-an-ase"></a>Creare un'app in un ambiente del servizio app ##

Per creare un'app in un ambiente del servizio app, procedere come di consueto per la creazione di un'app, ma con alcune piccole differenze. Quando si crea un nuovo piano di servizio app:

- Invece di selezionare una località geografica per distribuire l'app, selezionare un ambiente del servizio app come posizione.
- Tutti i piani di servizio app creati in un ambiente del servizio app devono essere in un piano tariffario isolato.

Se non si dispone di un ambiente del servizio app, è possibile crearne uno seguendo le istruzioni riportate in [creare un ambiente del servizio app][MakeExternalASE].

Per creare un'app in un ambiente del servizio app:

1. Selezionare **Crea una risorsa** > **Web e dispositivi mobili** > **App Web**.

2. Immettere un nome per l'app. Se si è già selezionato un piano di servizio app in un ambiente del servizio app, il nome di dominio dell'app rispecchierà il nome di dominio dell'ambiente del servizio app.

    ![Selezione del nome per l'app][1]

1. Selezionare una sottoscrizione.

1. Immettere un nome per un nuovo gruppo di risorse, oppure fare clic su **Usa esistente** e selezionarne uno dall'elenco a discesa.

1. Selezionare il sistema operativo. 

1. Selezionare un piano di servizio app esistente nell'ambiente del servizio app o crearne uno nuovo con la procedura seguente:

    a. Selezionare **Crea nuovo**.

    b. Immettere un nome per il piano di servizio app.

    c. Selezionare l'ambiente del servizio app nel menu a discesa **Località**. 
    
    d. Selezionare un piano tariffario **Isolato**. Scegliere **Seleziona**.

    e. Selezionare **OK**.
    
    ![Piani tariffari isolati][2]

    > [!NOTE]
    > Le app di Linux e Windows non possono essere presenti nello stesso piano di servizio app, ma possono essere presenti nello stesso ambiente del servizio app. 
    >

2. Selezionare **Create**.

## <a name="how-scale-works"></a>Come funziona il ridimensionamento ##

Ogni app del servizio app viene eseguita in un piano di servizio app. Il modello di contenitore è: gli ambienti contengono i piani di servizio app e i piani di servizio app contengono le app. Quando si ridimensiona un'app, si ridimensiona il piano di servizio app e quindi si ridimensionano tutte le app nello stesso piano.

In ASEv2, quando si ridimensiona un piano di servizio app, viene automaticamente aggiunta l'infrastruttura necessaria. Si verifica un ritardo nelle operazioni di ridimensionamento mentre viene aggiunta l'infrastruttura. In ASEv1 l'infrastruttura necessaria deve essere aggiunta prima di poter creare il piano di servizio app o di aumentarne il numero di istanze. 

Nel servizio app multi-tenant il ridimensionamento è in genere immediato perché è già disponibile un pool di risorse che lo supporta. In un ambiente del servizio app tale buffer non è disponibile e le risorse vengono allocate quando se ne presenta la necessità.

In un ambiente del servizio app è possibile ridimensionare fino a 100 istanze. Queste 100 istanze possono essere tutte in un solo piano di servizio app o distribuite in più piani di servizio app.

## <a name="ip-addresses"></a>Indirizzi IP ##

Il servizio app ha la possibilità di allocare un indirizzo IP dedicato a un'app. Questa funzionalità è disponibile dopo la configurazione di un SSL basato su IP, come descritto in [associare un certificato SSL personalizzato esistente al servizio app Azure][ConfigureSSL]. Tuttavia, in un ambiente del servizio app vi è un'eccezione. Non è possibile aggiungere altri indirizzi IP da usare per un certificato SSL basato su IP in un ambiente del servizio app con bilanciamento del carico interno.

In ASEv1 è necessario allocare gli indirizzi IP come risorse prima di poterli usare. In ASEv2 è possibile usarli dall'app proprio come nel servizio app multi-tenant. In ASEv2 è sempre presente un indirizzo di riserva fino a 30 indirizzi IP. Ogni volta che se ne usa uno, ne viene aggiunto un altro, in modo che sia sempre disponibile un indirizzo da usare. Si verifica un ritardo, necessario per allocare un altro indirizzo IP, che impedisce di aggiungere gli indirizzi IP in rapida successione.

## <a name="front-end-scaling"></a>Scalabilità front-end ##

In ASEv2, quando si aumenta il numero di istanze dei piani di servizio app, i ruoli di lavoro vengono automaticamente aggiunti per supportarli. Ogni ambiente del servizio app viene creato con due front-end. Inoltre, il front-end scala automaticamente in orizzontale a una velocità di un front-end per tutte le 15 istanze nei piani di servizio App. Se ad esempio si hanno 15 istanze, si hanno tre front-end. Se si passa a 30 istanze, si avranno quattro front-end e così via.

Questo numero di front-end deve essere più che sufficiente per la maggior parte degli scenari. Tuttavia, è possibile scalare orizzontalmente a una maggiore velocità. È possibile modificare il rapporto minimo a un front-end per ogni cinque istanze. Per modificare il rapporto è previsto un addebito. Per altre informazioni, vedere [prezzi del servizio app Azure][Pricing].

Le risorse front-end sono gli endpoint HTTP/HTTPS per l'ambiente del servizio app. Con la configurazione front-end predefinita, l'uso della memoria per ogni front-end è di circa il 60%. I carichi di lavoro del cliente non vengono eseguiti in un front-end. Il fattore chiave per un front-end dal punto di vista del ridimensionamento è la CPU, che dipende principalmente dal traffico HTTPS.

## <a name="app-access"></a>Accesso all'app ##

In un ambiente del servizio app esterno, il dominio usato per creare le app è diverso dal servizio app multi-tenant. Include il nome dell'ambiente del servizio app. Per altre informazioni su come creare un ambiente del servizio app esterno, vedere [creare un ambiente del servizio app][MakeExternalASE]. Il nome di dominio in un ambiente del servizio app esterno è simile a *.&lt;nomeambientedelservizioapp&gt;.p.azurewebsites.net*. Ad esempio, se l'ambiente del servizio app è denominato _external-ase_ e si ospita un'app denominata _contoso_ in tale ambiente del servizio app, sarà possibile raggiungerlo agli URL seguenti:

- contoso.external-ase.p.azurewebsites.net
- contoso.scm.external-ase.p.azurewebsites.net

L'URL contoso.scm.external-ase.p.azurewebsites.net viene usato per accedere alla console Kudu o per pubblicare l'app usando la distribuzione Web. Per informazioni sulla console Kudu, vedere la [console Kudu per il servizio app Azure][Kudu]. La console Kudu offre un'interfaccia utente Web per il debug, il caricamento di file, la modifica di file e altro ancora.

In un ambiente del servizio app con bilanciamento del carico interno si determina il dominio in fase di distribuzione. Per altre informazioni su come creare un ambiente del servizio app ILB, vedere [creare e usare un][MakeILBASE]ambiente del servizio app ILB. Se si specifica il nome di dominio _ilb-ase.info_, le app in tale ambiente del servizio app usano questo dominio durante la creazione delle app. Per l'app denominata _contoso_, gli URL saranno:

- contoso.ilb-ase.info
- contoso.scm.ilb-ase.info

## <a name="publishing"></a>Pubblicazione ##

Come nel servizio app multi-tenant, in un ambiente del servizio app è possibile pubblicare con:

- Distribuzione Web.
- FTP.
- Integrazione continua.
- Trascinamento della selezione nella console Kudu.
- Un ambiente di sviluppo integrato, ad esempio Visual Studio, Eclipse o Intellij IDEA.

Con un ambiente del servizio app esterno, queste opzioni di pubblicazione hanno lo stesso comportamento. Per ulteriori informazioni, vedere [distribuzione nel servizio app Azure][AppDeploy]. 

La grande differenza dal punto di vista della pubblicazione è negli ambienti del servizio app con bilanciamento del carico interno. Con un ambiente del servizio app con bilanciamento del carico interno gli endpoint di pubblicazione sono tutti disponibili solo tramite il servizio di bilanciamento del carico interno. Il servizio di bilanciamento del carico interno è in un IP privato nella subnet dell'ambiente del servizio app nella rete virtuale. Se non si ha accesso di rete al servizio di bilanciamento del carico interno, non è possibile pubblicare app in tale ambiente del servizio app. Come indicato nella pagina relativa alla [creazione e all'uso di un][MakeILBASE]ambiente del servizio app ILB, è necessario configurare DNS per le app nel sistema. incluso l'endpoint Gestione controllo servizi. Se non definiti correttamente, non è possibile pubblicarli. Gli ambienti di sviluppo integrato devono anche avere accesso di rete al servizio di bilanciamento del carico interno per potervi eseguire direttamente la pubblicazione.

I sistemi CI basati su Internet, ad esempio GitHub e Azure DevOps, non funzionano con un ambiente del servizio app ILB perché l'endpoint di pubblicazione non è accessibile tramite Internet. Per Azure DevOps, è possibile risolvere il problema installando un agente di rilascio self-hosted nella rete interna in cui è possibile raggiungere il ILB. In alternativa, è anche possibile usare un sistema CI che usa un modello pull, ad esempio Dropbox.

Gli endpoint di pubblicazione per le app in un ambiente del servizio app con bilanciamento del carico interno usano il dominio con cui l'ambiente del servizio app con bilanciamento del carico interno è stato creato, che può essere visualizzato nel profilo di pubblicazione dell'app e nel pannello del portale dell'app (in **Panoramica** > **Informazioni di base** e anche in **Proprietà**). 

## <a name="pricing"></a>Prezzi ##

Lo SKU di prezzi chiamato **Isolato** è stato creato solo per l'uso con ASEv2. Tutti i piani di servizio app ospitati in ASEv2 fanno parte dello SKU di prezzi Isolato. I costi del piano di servizio app Isolato possono variare in base all'area. 

Oltre al prezzo dei piani di servizio app, è disponibile una tariffa fissa solo per l'ambiente del servizio app. Questa tariffa fissa non cambia in base alle dimensioni dell'ambiente del servizio app e si applica all'infrastruttura dell'ambiente del servizio app a un rapporto di ridimensionamento predefinito di un front-end aggiuntivo per ogni 15 istanze del piano di servizio app.  

Se il rapporto predefinito di un front-end per ogni 15 istanze del piano di servizio app non è sufficientemente veloce, è possibile modificarlo in base ai front-end aggiunti o alle dimensioni dei front-end.  Quando si modificano il rapporto o le dimensioni, si paga per i core front-end che non verrebbero aggiunti per impostazione predefinita.  

Se ad esempio si imposta la proporzione di ridimensionamento su 10, verrà aggiunto un front-end ogni 10 istanze nei piani di servizio app. La tariffa fissa copre una proporzione di ridimensionamento pari a un front-end ogni 15 istanze. Con una proporzione di ridimensionamento di 10, il costo verrà addebitato al terzo front-end aggiunto per le 10 istanze del piano di servizio app. Non è necessario pagare quando si raggiungono le 15 istanze, perché il front-end è stato aggiunto automaticamente.

Se si modificano le dimensioni dei front-end in base a due core, ma non si modifica il rapporto, si paga per i core aggiuntivi.  Poiché viene creato un ambiente del servizio app con due front-end, anche al di sotto della soglia di ridimensionamento automatica si paga per due core aggiuntivi se le dimensioni sono state aumentate a due core front-end.

Per altre informazioni, vedere [prezzi del servizio app Azure][Pricing].

## <a name="delete-an-ase"></a>Eliminare un ambiente del servizio app ##

Per eliminare un ambiente del servizio app: 

1. Usare **Elimina** nella parte superiore del pannello **Ambiente del servizio app**. 

1. Immettere il nome dell'ambiente del servizio app per confermare l'eliminazione. Quando si elimina un ambiente del servizio app, viene eliminato anche tutto il relativo contenuto. 

    ![Eliminazione dell'ambiente del servizio app][3]

<!--Image references-->
[1]: ./media/using_an_app_service_environment/usingase-appcreate.png
[2]: ./media/using_an_app_service_environment/usingase-pricingtiers.png
[3]: ./media/using_an_app_service_environment/usingase-delete.png


<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../web-sites-purchase-ssl-web-site.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md
