---
title: Panoramica degli insiemi di credenziali di Servizi di ripristino
description: Una panoramica e un confronto tra gli insiemi di credenziali di Servizi di ripristino e gli insiemi di credenziali di Backup di Azure.
author: dcurwin
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 08/10/2018
ms.author: dacurwin
ms.openlocfilehash: 3c451d1bd77fa1cec85a9d42b5d4d3d819b2d19a
ms.sourcegitcommit: d470d4e295bf29a4acf7836ece2f10dabe8e6db2
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/02/2019
ms.locfileid: "70210354"
---
# <a name="recovery-services-vaults-overview"></a>Panoramica di insiemi di credenziali di Servizi di ripristino

In questo articolo vengono descritte le funzionalità di un insieme di credenziali di Servizi di ripristino. Un insieme di credenziali di Servizi di ripristino è un'entità di archiviazione di Azure che ospita i dati. I dati sono in genere copie di dati o informazioni di configurazione per macchine virtuali, carichi di lavoro, server o workstation. È possibile usare gli insiemi di credenziali di Servizi di ripristino per contenere dati di backup per vari servizi di Azure, ad esempio database SQL di Azure e macchine virtuali IaaS, Linux o Windows. Gli insiemi di credenziali di Servizi di ripristino supportano System Center DPM, Windows Server, server di Backup di Azure e altro ancora. Gli insiemi di credenziali di Servizi di ripristino semplificano l'organizzazione dei dati di backup, riducendo al minimo l'overhead di gestione.

Nell'ambito di una sottoscrizione di Azure è possibile creare un massimo di 500 insiemi di credenziali di Servizi di ripristino per ogni sottoscrizione in una determinata area.

## <a name="comparing-recovery-services-vaults-and-backup-vaults"></a>Confronto tra gli insiemi di credenziali di Servizi di ripristino e gli insiemi di credenziali di Backup

Se si dispone ancora di insiemi di credenziali di Backup, questi vengono aggiornati automaticamente in base agli insiemi di credenziali di Servizi di ripristino. A novembre 2017 è stato eseguito l'aggiornamento di tutti gli insiemi di credenziali di Backup agli insiemi di credenziali di Servizi di ripristino.

Gli insiemi di credenziali di Servizi di ripristino si basano sul modello di Azure Resource Manager, mentre gli insiemi di credenziali di Backup si basano sul modello di Service Manager di Azure. Quando si aggiorna un insieme di credenziali di Backup a un insieme di credenziali di Servizi di ripristino, i dati di backup rimangono intatti durante e dopo il processo di aggiornamento. Gli insiemi di credenziali di Servizi di ripristino offrono le funzionalità non disponibili per gli insiemi di credenziali di Backup, ad esempio:

- **Funzionalità avanzate per proteggere i dati di backup**: con gli insiemi di credenziali di Servizi di ripristino, Backup di Azure offre funzionalità di sicurezza per proteggere i backup nel cloud. Queste funzionalità di sicurezza garantiscono la protezione dei backup e ripristinano i dati in modo sicuro anche se i server di produzione e di backup vengono compromessi. [Altre informazioni](backup-azure-security-feature.md)

- **Monitoraggio centralizzato per l'ambiente IT ibrido**: con gli insiemi di credenziali di Servizi di ripristino, è possibile monitorare non solo le [macchine virtuali IaaS di Azure](backup-azure-manage-vms.md) ma anche le [risorse locali](backup-azure-manage-windows-server.md#manage-backup-items) da un portale centrale. [Altre informazioni](https://azure.microsoft.com/blog/alerting-and-monitoring-for-azure-backup)

- **Controllo degli accessi in base al ruolo o RBAC**: il controllo degli accessi in base al ruolo consente un controllo della gestione degli accessi con granularità fine in Azure. [Azure offre diversi ruoli predefiniti](../role-based-access-control/built-in-roles.md) mentre Backup di Azure dispone di tre [ruoli predefiniti per la gestione dei punti di ripristino](backup-rbac-rs-vault.md). Gli insiemi di credenziali di Servizi di ripristino sono compatibili con il controllo degli accessi in base al ruolo, che consente di limitare il backup e ripristinare l'accesso a insiemi definiti di ruoli utente. [Altre informazioni](backup-rbac-rs-vault.md)

- **Proteggere tutte le configurazioni delle macchine virtuali Azure**: gli insiemi di credenziali di Servizi di ripristino proteggono le macchine virtuali basate su Resource Manager tra cui i dischi Premium, i dischi gestiti e le macchine virtuali crittografate. L'aggiornamento di un insieme di credenziali di Backup a un insieme di credenziali di Servizi di ripristino offre l'opportunità di aggiornare le macchine virtuali basate su Service Manager alle macchine virtuali basate su Resource Manager. Durante l'aggiornamento dell'insieme di credenziali, è possibile mantenere i punti di ripristino della macchina virtuale basata su Service Manager e configurare la protezione per le macchine virtuali aggiornate abilitate per Resource Manager. [Altre informazioni](https://azure.microsoft.com/blog/azure-backup-recovery-services-vault-ga)

- **Ripristino immediato delle macchine virtuali IaaS**: con gli insiemi di credenziali di Servizi di ripristino, è possibile ripristinare file e cartelle da una macchina virtuale IaaS senza ripristinare l'intera macchina virtuale, il che consente di avere tempi di ripristino più rapidi. Il ripristino immediato per le macchine virtuali IaaS è disponibile sia per le macchine virtuali Windows che Linux. [Altre informazioni](https://azure.microsoft.com/blog/instant-file-recovery-from-azure-linux-vm-backup-using-azure-backup-preview)

## <a name="managing-your-recovery-services-vaults-in-the-portal"></a>Gestione degli insiemi di credenziali di Servizi di ripristino nel portale
La creazione e la gestione degli insiemi di credenziali di Servizi di ripristino nel portale di Azure è semplificata dal fatto che il servizio Backup è integrato in altri servizi di Azure. Questa integrazione implica la possibile creazione e gestione di un insieme di credenziali di Servizi di ripristino *nel contesto del servizio di destinazione*. Ad esempio, per visualizzare i punti di ripristino di una macchina virtuale, selezionarla e fare clic su **Backup** nel menu Operazioni.

![Macchina virtuale con i dettagli dell'insieme di credenziali dei servizi di ripristino](./media/backup-azure-recovery-services-vault-overview/rs-vault-in-context-vm.png)

Se la macchina virtuale non dispone di un backup configurato, verrà richiesto di configurare il backup. Se il backup è stato configurato, verranno visualizzate informazioni di backup sulla macchina virtuale, incluso un elenco di punti di ripristino.  

![Macchina virtuale con i dettagli dell'insieme di credenziali dei servizi di ripristino](./media/backup-azure-recovery-services-vault-overview/vm-recovery-point-list.png)

Nell'esempio precedente **ContosoVM** è il nome della macchina virtuale. **ContosoVM-demovault** è il nome dell'insieme di credenziali di Servizi di ripristino. Non è necessario ricordare il nome dell'insieme di credenziali di Servizi di ripristino che archivia i punti di ripristino; è possibile accedere a questa informazione dalla macchina virtuale.  

Se più server sono protetti mediante lo stesso insieme di credenziali di Servizi di ripristino, conviene esaminare l'insieme di credenziali di Servizi di ripristino. È possibile cercare tutti gli insiemi di credenziali di Servizi di ripristino nella sottoscrizione e sceglierne uno dall'elenco.

Le sezioni seguenti contengono collegamenti ad articoli che spiegano come usare un insieme di credenziali di Servizi di ripristino in ogni tipo di attività.

> [!NOTE]
> L'insieme di credenziali di Servizi di ripristino non può essere creato con lo stesso nome se è stata eliminato nelle 24 ore precedenti. Usare un nome di risorsa diverso, scegliere un gruppo di risorse diverso o riprovare nuovamente dopo 24 ore.

### <a name="back-up-data"></a>Dati di backup
- [Backup di una macchina virtuale di Azure](backup-azure-vms-first-look-arm.md)
- [Backup di Windows Server o della workstation di Windows](backup-try-azure-backup-in-10-mins.md)
- [Backup dei carichi di lavoro DPM in Azure](backup-azure-dpm-introduction.md)
- [Preparazione del backup dei carichi di lavoro con il server di Backup di Azure](backup-azure-microsoft-azure-backup.md)

### <a name="manage-recovery-points"></a>Gestire i punti di ripristino
- [Gestire i backup delle macchine virtuali di Azure](backup-azure-manage-vms.md)
- [Gestione di file e cartelle](backup-azure-manage-windows-server.md)

### <a name="restore-data-from-the-vault"></a>Ripristinare i dati dall'insieme di credenziali
- [Ripristinare singoli file da una macchina virtuale di Azure](backup-azure-restore-files-from-vm.md)
- [Ripristinare una macchina virtuale di Azure](backup-azure-arm-restore-vms.md)

### <a name="secure-the-vault"></a>Proteggere l'insieme di credenziali
- [Protezione dei dati di backup cloud negli insiemi di credenziali di Servizi di ripristino](backup-azure-security-feature.md)



## <a name="next-steps"></a>Fasi successive
Vedere gli articoli seguenti:</br>
[Eseguire il backup di una macchina virtuale IaaS](backup-azure-arm-vms-prepare.md)</br>
[Eseguire il backup del server di Backup di Azure](backup-azure-microsoft-azure-backup.md)</br>
[Eseguire il backup di Windows Server](backup-configure-vault.md)
