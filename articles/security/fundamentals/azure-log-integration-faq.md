---
title: Domande frequenti sull'integrazione dei log di Azure | Microsoft Docs
description: Questo articolo contiene le risponde alle domande sull'integrazione dei log di Azure.
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: TerryLanfear
ms.assetid: d06d1ac5-5c3b-49de-800e-4d54b3064c64
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload8: na
ms.date: 05/28/2019
ms.author: barclayn
ms.custom: azlog
ms.openlocfilehash: 7ed63364a9dc4b96470a23af3a4bff832d42621b
ms.sourcegitcommit: 85b3973b104111f536dc5eccf8026749084d8789
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/01/2019
ms.locfileid: "68727617"
---
# <a name="azure-log-integration-faq"></a>Domande frequenti sull'integrazione dei log di Azure

Questo articolo contiene le risponde alle domande frequenti sull'integrazione dei log di Azure.

>[!IMPORTANT]
> La funzionalità di integrazione dei log di Azure verrà deprecata da 06/15/2019. Il download di AzLog è stato disabilitato il 27 giugno 2018. Per materiale sussidiario su cosa fare dopo, vedere il post [Use Azure monitor to integrate with SIEM tools](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/) (Usare Monitoraggio di Azure per eseguire l'integrazione con gli strumenti per le informazioni di sicurezza e gestione degli eventi) 

L'integrazione dei log di Azure è un servizio del sistema operativo Windows che consente di integrare log non elaborati delle risorse di Azure nei sistemi di gestione di informazioni ed eventi di sicurezza (SIEM) locali. Questa integrazione fornisce un dashboard unificato per tutti gli asset locali o nel cloud. È possibile aggregare, correlare, analizzare e inviare avvisi per gli eventi di sicurezza associati alle applicazioni.

Il metodo preferito per l'integrazione dei log di Azure consiste nell'usare un connettore per Monitoraggio di Azure del fornitore SIEM e seguire queste [istruzioni](/azure/azure-monitor/platform/stream-monitoring-data-event-hubs). Tuttavia, se il fornitore SIEM non offre un connettore per Monitoraggio di Azure, è possibile usare Integrazione log di Azure come soluzione temporanea (se SIEM è supportato da Integrazione log di Azure) fino a quando non sia disponibile un connettore di questo tipo.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="is-the-azure-log-integration-software-free"></a>Il software di integrazione dei log di Azure è gratuito?

Sì. Non è previsto alcun addebito per il software di integrazione dei log di Azure.

## <a name="where-is-azure-log-integration-available"></a>Dove è disponibile l'integrazione dei log di Azure?

Attualmente è disponibile nell'area commerciale di Azure e in Azure per enti pubblici e non è disponibile in Cina né in Germania.

## <a name="how-can-i-see-the-storage-accounts-from-which-azure-log-integration-is-pulling-azure-vm-logs"></a>Come è possibile individuare gli account di archiviazione da cui il servizio di integrazione dei log di Azure estrae i log delle VM di Azure?

Eseguire il comando **AzLog source list**.

## <a name="how-can-i-tell-which-subscription-the-azure-log-integration-logs-are-from"></a>Come è possibile stabilire da quale sottoscrizione provengono i log di integrazione dei log di Azure?

Nel caso dei log di controllo presenti nella directory **AzureResourcemanagerJson**, l'ID della sottoscrizione è il nome del file di log. Questo vale anche per i log nella cartella **AzureSecurityCenterJson**. Ad esempio:

20170407T070805_2768037.0000000023.**1111e5ee-1111-111b-a11e-1e111e1111dc**.json

I log di controllo di Azure Active Directory includono l'ID tenant come parte del nome.

I log di diagnostica che vengono letti da un hub eventi non includono l'ID di sottoscrizione come parte del nome. Includono invece il nome descrittivo specificato nell'ambito della creazione dell'origine dell'hub eventi. 

## <a name="how-can-i-update-the-proxy-configuration"></a>Come è possibile aggiornare la configurazione del proxy?

Se la configurazione del proxy non consente l'accesso di archiviazione di Azure direttamente, aprire il file **AZLOG.EXE.CONFIG** in **c:\Programmi\Integrazione dei log di Microsoft Azure**. Aggiornare il file in modo che includa la sezione **defaultProxy** con l'indirizzo del proxy dell'organizzazione. Al termine dell'aggiornamento, arrestare e avviare il servizio usando i comandi **net stop AzLog** e **net start AzLog**.

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <system.net>
        <connectionManagement>
          <add address="*" maxconnection="400" />
        </connectionManagement>
        <defaultProxy>
          <proxy usesystemdefault="true"
          proxyaddress="http://127.0.0.1:8888"
          bypassonlocal="true" />
        </defaultProxy>
      </system.net>
      <system.diagnostics>
        <performanceCounters filemappingsize="20971520" />
      </system.diagnostics>   

## <a name="how-can-i-see-the-subscription-information-in-windows-events"></a>Come è possibile vedere le informazioni sulla sottoscrizione negli eventi di Windows?

Aggiungere l'ID sottoscrizione al nome descrittivo quando si aggiunge l'origine:

    Azlog source add <sourcefriendlyname>.<subscription id> <StorageName> <StorageKey>  
L'XML dell'evento include i metadati seguenti, compreso l'ID sottoscrizione:

![XML dell'evento](media/azure-log-integration-faq/event-xml.png)

## <a name="error-messages"></a>messaggi di errore
### <a name="when-i-run-the-command-azlog-createazureid-why-do-i-get-the-following-error"></a>Perché viene visualizzato il seguente errore quando si esegue il comando ```AzLog createazureid```?

Errore:

  *Impossibile creare l'applicazione AAD - Tenant 72f988bf-86f1-41af-91ab-2d7cd011db37 - Motivo = "Accesso negato" - Messaggio = "Privilegi insufficienti per completare l'operazione".*

Il comando **azlog createazureid** tenta di creare un'entità servizio in tutti i tenant di Azure AD per le sottoscrizioni a cui l'account di accesso di Azure può accedere. Se l'account di accesso di Azure è solo un utente guest nel tenant di Azure AD, il comando non riesce e mostra l'errore "I privilegi non sono insufficienti per completare l'operazione". Richiedere all'amministratore del tenant di aggiungere l'account come utente nel tenant.

### <a name="when-i-run-the-command-azlog-authorize-why-do-i-get-the-following-error"></a>Durante l'esecuzione del comando **azlog authorize**, perché viene visualizzato il seguente errore?

Errore:

  *Avviso durante la creazione dell'assegnazione del ruolo - Autorizzazione non riuscita: Il client janedo\@Microsoft.com ' con ID oggetto ' fe9e03e4-4DAD-4328-910F-fd24a9660bd2' non è autorizzato a eseguire l'azione ' Microsoft. Authorization/roleAssignments/Write ' sull'ambito '/Subscriptions/ 70d95299-d689-4c97-b971-0d8ff0000000'.*

Il comando **azlog authorize** assegna il ruolo di Lettore all'entità servizio di Azure AD (creata con **azlog createazureid**) per le sottoscrizioni fornite. Se l'account di accesso di Azure non è un coamministratore o un proprietario della sottoscrizione, l'operazione ha esito negativo con il messaggio di errore "Autorizzazione non riuscita". Per completare l'azione è necessario il controllo degli accessi in base al ruolo di coamministratore o proprietario.

## <a name="where-can-i-find-the-definition-of-the-properties-in-the-audit-log"></a>Dove si trova la definizione delle proprietà nel log di controllo?

Vedere:

* [Operazioni di controllo con Azure Resource Manager](/azure/azure-resource-manager/resource-group-audit)
* [Elencare gli eventi di gestione in una sottoscrizione nell'API REST di Monitoraggio di Azure](https://msdn.microsoft.com/library/azure/dn931934.aspx)

## <a name="where-can-i-find-details-on-azure-security-center-alerts"></a>Dove sono disponibili altre informazioni sugli avvisi del Centro sicurezza di Azure?

Vedere [Gestione e risposta agli avvisi di sicurezza nel Centro sicurezza di Azure](/azure/security-center/security-center-managing-and-responding-alerts).

## <a name="how-can-i-modify-what-is-collected-with-vm-diagnostics"></a>Come è possibile scegliere quali informazioni raccogliere con la diagnostica delle VM?

Per informazioni dettagliate su come ottenere, modificare e impostare la configurazione di Diagnostica di Azure vedere [Usare PowerShell per abilitare la Diagnostica di Azure in una macchina virtuale che esegue Windows](/azure/virtual-machines/windows/ps-extensions-diagnostics?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 

L'esempio seguente ottiene la configurazione di Diagnostica di Azure:

    Get-AzVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient
    $publicsettings = (Get-AzVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient).PublicSettings
    $encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
    $xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
    Write-Host $xmlconfig

    $xmlconfig | Out-File -Encoding utf8 -FilePath "d:\WADConfig.xml"

L'esempio seguente modifica la configurazione di Diagnostica di Azure: In questa configurazione, solo l'evento ID 4624 e l'evento ID 4625 vengono raccolti dal log eventi di sicurezza. Gli eventi Microsoft Antimalware per Azure vengono raccolti dal registro eventi di sistema. Per informazioni dettagliate sull'uso di espressioni XPath, vedere [Utilizzo degli eventi](https://msdn.microsoft.com/library/windows/desktop/dd996910(v=vs.85)).

    <WindowsEventLog scheduledTransferPeriod="PT1M">
        <DataSource name="Security!*[System[(EventID=4624 or EventID=4625)]]" />
        <DataSource name="System!*[System[Provider[@Name='Microsoft Antimalware']]]"/>
    </WindowsEventLog>

L'esempio seguente imposta la configurazione di Diagnostica di Azure:

    $diagnosticsconfig_path = "d:\WADConfig.xml"
    Set-AzVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName log3121 -StorageAccountKey <storage key>

Dopo aver apportato modifiche, verificare l'account di archiviazione per assicurarsi che vengano raccolti gli eventi corretti.

Se si verificano problemi durante l'installazione e la configurazione, aprire una [richiesta di supporto](/azure/azure-supportability/how-to-create-azure-support-request). Selezionare **Integrazione log** come servizio per cui richiedere il supporto.

## <a name="can-i-use-azure-log-integration-to-integrate-network-watcher-logs-into-my-siem"></a>È possibile usare l'integrazione dei log di Azure per integrare i log di Network Watcher nel sistema SIEM personale?

Azure Network Watcher genera grandi quantità di informazioni di registrazione. Questi log non sono concepiti per essere inviati a un sistema SIEM. L'unica destinazione supportata per i log di controllo di rete è un account di archiviazione. L'integrazione dei log di Azure non supporta la lettura di questi log e la relativa disponibilità in un sistema SIEM.


