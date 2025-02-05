---
title: Architettura per il ripristino di emergenza di Hyper-V in un sito locale secondario con Azure Site Recovery
description: Questo articolo offre una panoramica dell'architettura per il ripristino di emergenza di macchine virtuali Hyper-V locali in un sito System Center VMM secondario con Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: raynew
ms.openlocfilehash: 2d8e9c3531e031538c593cfd60d83b4ae97b4f4c
ms.sourcegitcommit: fa4852cca8644b14ce935674861363613cf4bfdf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/09/2019
ms.locfileid: "70813740"
---
# <a name="architecture---hyper-v-replication-to-a-secondary-site"></a>Architettura per la replica di Hyper-V in un sito secondario

Questo articolo descrive i componenti e i processi coinvolti durante la replica in un sito VMM secondario di macchine virtuali Hyper-V locali presenti in cloud System Center Virtual Machine Manager (VMM) tramite il servizio [Azure Site Recovery](site-recovery-overview.md) nel portale di Azure.


## <a name="architectural-components"></a>Componenti dell'architettura

La tabella e il grafico seguenti offrono una visualizzazione generale dei componenti usati per la replica Hyper-V in un sito secondario.

**Componente** | **Requisito** | **Dettagli**
--- | --- | ---
**Azure** | Sottoscrizione di Azure | Creare un insieme di credenziali di Servizi di ripristino nella sottoscrizione di Azure per orchestrare e gestire la replica tra i percorsi VMM.
**Server VMM** | È necessario avere a disposizione un percorso VMM primario e uno secondario. | È consigliabile distribuire un server VMM in un sito primario e uno nel sito secondario.
**Server Hyper-V** |  Uno o più server host Hyper-V nei cloud VMM primario e secondario. | I dati vengono replicati tra il server host Hyper-V primario e secondario su LAN o VPN usando l'autenticazione Kerberos o del certificato.  
**VM Hyper-V** | Nel server host Hyper-V. | Il server host di origine deve avere almeno una VM che si vuole replicare.

**Architettura della replica da sito locale a sito locale**

![Da sito locale a sito locale](./media/hyper-v-vmm-architecture/arch-onprem-onprem.png)

## <a name="replication-process"></a>Processo di replica

1. Quando viene attivata la replica iniziale, viene acquisito uno [snapshot della macchina virtuale Hyper-V](https://technet.microsoft.com/library/dd560637.aspx).
2. I dischi rigidi virtuali nella macchina virtuale vengono replicati uno per volta nel sito secondario.
3. Se vengono apportate modifiche ai dischi mentre è in corso la replica iniziale, Hyper-V Replica Replication Tracker tiene traccia delle modifiche sotto forma di log di replica di Hyper-V (HRL). Questi file di log si trovano nella stessa cartella dei dischi. A ogni disco è associato un file HRL, che viene inviato al sito secondario. Si noti che lo snapshot e i file di log usano risorse del disco durante l'esecuzione della replica iniziale.
4. Al termine della replica iniziale, lo snapshot della macchina virtuale viene eliminato e viene avviata la replica differenziale.
5. Nel log le modifiche differenziali al disco vengono sincronizzate e unite al disco padre.


## <a name="failover-and-failback-process"></a>Processo di failover e failback

- È possibile eseguire il failover di un solo computer o creare piani di ripristino per orchestrare il failover di più computer.
- È possibile eseguire un failover pianificato o non pianificato tra siti locali. Se si esegue un failover pianificato, le macchine virtuali di origine vengono arrestate per assicurare che non si verifichino perdite di dati.
    - Se è stato eseguito un failover non pianificato in un sito secondario, dopo il failover le macchine nella posizione secondaria non sono protette.
    - Se è stato eseguito un failover pianificato, dopo il failover i computer della posizione secondaria sono protetti.
- Dopo aver attivato il failover iniziale, è necessario eseguirne il commit per iniziare ad accedere al carico di lavoro dalla macchina virtuale di replica.
- Quando il sito primario è di nuovo disponibile, è possibile eseguire il failback.
    - Avviare la replica inversa, per avviare la replica dal sito secondario a quello primario. La replica inversa porta le macchine virtuali in uno stato protetto, ma per il datacenter secondario resta la posizione attiva.
    - Per rendere di nuovo il sito primario la posizione attiva, occorre avviare un failover pianificato da sito secondario a primario, seguito da un'altra replica inversa.



## <a name="next-steps"></a>Passaggi successivi


Seguire questa [esercitazione](hyper-v-vmm-disaster-recovery.md) per abilitare la replica Hyper-V tra cloud VMM.
