---
title: Eseguire l'aggiornamento al piano Standard del Centro sicurezza per la sicurezza avanzata | Microsoft Docs
description: Questo articolo fornisce informazioni sui prezzi del Centro sicurezza di Azure.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 4d1364cd-7847-425a-bb3a-722cb0779f78
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/17/2019
ms.author: memildin
ms.openlocfilehash: c5e457a6478fce070070ba7d0d610cbfe4f55024
ms.sourcegitcommit: ae461c90cada1231f496bf442ee0c4dcdb6396bc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/17/2019
ms.locfileid: "72554687"
---
# <a name="upgrade-to-security-centers-standard-tier-for-enhanced-security"></a>Eseguire l'aggiornamento al piano Standard del Centro sicurezza per la sicurezza avanzata
Il Centro sicurezza di Azure offre la gestione unificata della sicurezza e la protezione avanzata dalle minacce per carichi di lavoro in esecuzione in Azure, in locale e in altri cloud. Offre visibilità e controllo su carichi di lavoro cloud ibridi, difese attive che riducono l'esposizione a minacce e il rilevamento intelligente che consente di tenere il passo con gli attacchi informatici in rapida evoluzione.

## <a name="pricing-tiers"></a>Piani tariffari
Il Centro sicurezza è disponibile in due livelli:

- Il livello **gratuito** è abilitato in tutte le sottoscrizioni di Azure dopo aver visitato il dashboard del Centro sicurezza di Azure nel portale di Azure per la prima volta o se è stato abilitato PROGREMMATICALY tramite API. Se si aggiungono altre sottoscrizioni aftewards, per la nuova sottoscrizione non sarà abilitato il Centro sicurezza per impostazione predefinita, ma è sufficiente avviare il portale del Centro sicurezza e la nuova sottoscrizione verrà registrata e verrà applicato il livello gratuito. Il livello gratuito fornisce criteri di sicurezza, valutazione continua della sicurezza e raccomandazioni di sicurezza praticabili che consentono di proteggere le risorse di Azure.
- Il livello **Standard** estende le funzionalità del livello Gratuito ai carichi di lavoro in esecuzione in cloud privati e pubblici, offrendo la gestione unificata della sicurezza e la protezione dalle minacce nei carichi di lavoro cloud ibridi. Il livello Standard aggiunge anche la funzionalità di rilevamento avanzato delle minacce, che usa analisi comportamentali e apprendimento automatico per identificare attacchi ed exploit zero-day, i controlli di accesso e dell'applicazione per ridurre l'esposizione agli attacchi di rete e malware e molto altro. È possibile provare gratuitamente il livello standard. Il Centro sicurezza standard supporta le risorse di Azure, tra cui macchine virtuali, set di scalabilità di macchine virtuali, servizio app, server SQL e account di archiviazione. Se si dispone di Centro sicurezza di Azure Standard, è possibile rifiutare esplicitamente il supporto in base al tipo di risorsa. 

La maggior parte delle valutazioni della sicurezza del livello gratuito per le macchine virtuali, oltre a molti avvisi di sicurezza del livello standard, richiede l'installazione della funzionalità di Microsoft Monitoring Agent (MMA). È possibile abilitare il provisioning automatico nel centro sicurezza per distribuire automaticamente l'agente per le macchine virtuali di Azure.

Per altre informazioni, vedere la [pagina dei prezzi](https://azure.microsoft.com/pricing/details/security-center/) del Centro sicurezza.

## <a name="try-standard-free-for-30-days"></a>Prova gratuita del livello Standard per 30 giorni
Il livello Standard è gratuito per i primi 30 giorni. Al termine dei 30 giorni, se si sceglie di continuare a usare il servizio, i costi inizieranno a essere addebitati automaticamente.

È possibile aggiornare una sottoscrizione completa di Azure al piano Standard, che viene ereditata da tutte le risorse all'interno della sottoscrizione.

Per ottenere il livello Standard:

1. Selezionare **prezzi & impostazioni** nel menu principale del **Centro sicurezza** .
2. Selezionare la sottoscrizione che si desidera aggiornare al livello Standard.
3. Selezionare il piano **tariffario**.
4. Selezionare **Standard** per eseguire l'aggiornamento.
5. Fare clic su **Salva**

I prezzi nell'immagine sono solo a scopo esemplificativo.  Prezzi di ![Security Center ](./media/security-center-pricing/get-standard.png)

> [!NOTE]
> Per abilitare tutte le funzionalità del Centro sicurezza, è necessario applicare il piano tariffario Standard alla sottoscrizione contenente le macchine virtuali applicabili. La configurazione dei prezzi per un'area di lavoro non Abilita l'accesso just-in-time alle macchine virtuali, i controlli delle applicazioni adattivi e i rilevamenti di rete per le risorse di Azure.
>
>

## <a name="why-upgrade-to-standard"></a>Perché eseguire l'aggiornamento al livello standard?
Il Centro sicurezza offre maggiore sicurezza e protezione dalle minacce per i carichi di lavoro di cloud ibridi, tra cui:

- **Sicurezza ibrida**: per ottenere una visualizzazione unificata della sicurezza in tutti i carichi di lavoro locali e cloud; applicare i criteri di sicurezza e la valutazione continua della sicurezza dei carichi di lavoro cloud ibridi per garantire la conformità con gli standard di sicurezza; Raccogli, esegui ricerche e analizza i dati di sicurezza da diverse origini, inclusi firewall e altre soluzioni partner.
- **Rilevamento avanzato delle minacce**, che usa le analisi avanzate e Microsoft Intelligent Security Graph per prevenire attacchi informatici in continua evoluzione.  Sfrutta i vantaggi delle analisi comportamentali predefinite e di Machine Learning per identificare gli attacchi e gli exploit zero-day. Monitora le reti, le macchine virtuali e i servizi cloud per rilevare attacchi imminenti e attività post violazione. Semplifica l'investigazione con strumenti interattivi e intelligence per le minacce contestuale.
- **Controlli di accesso e dell'applicazione**, che blocca malware e altre applicazioni indesiderate applicando i consigli per l'inserimento nell'elenco elementi consentiti adattati ai carichi di lavoro specifici e basati su tecnologia per l'apprendimento automatico. Riduce la superficie di attacco alla rete con l'accesso JIT (Just-In-Time) controllato alle porte di gestione nelle macchine virtuali di Azure per ridurre drasticamente l'esposizione ad attacchi di forza bruta e altri attacchi di rete.


## <a name="next-steps"></a>Passaggi successivi
In questo articolo sono stati presentati i prezzi del Centro sicurezza. Per altre informazioni sulla sicurezza e la protezione avanzate dalle minacce del livello Standard, vedere:

- [Rilevamento delle minacce avanzato](security-center-threat-report.md)
- [Controllo di accesso just-in-time alle macchine virtuali](security-center-just-in-time.md)

<!--Image references-->
[1]: ./media/security-center-pricing/get-standard.png
