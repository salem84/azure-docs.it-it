---
title: Pianificare i criteri di accesso condizionale in Azure Active Directory | Microsoft Docs
description: In questo articolo descrive come pianificare i criteri di accesso condizionale di Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 01/25/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: martincoetzer
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3fca84a71e1ede572e3889f973248db158115bec
ms.sourcegitcommit: cf438e4b4e351b64fd0320bf17cc02489e61406a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/08/2019
ms.locfileid: "67655493"
---
# <a name="how-to-plan-your-conditional-access-deployment-in-azure-active-directory"></a>Procedura: Pianificare la distribuzione di accesso condizionale in Azure Active Directory

Pianificazione della distribuzione di accesso condizionale è fondamentale assicurarsi che ottenere la strategia di accesso necessario per le App e le risorse all'interno dell'organizzazione. Per la maggior parte del tuo tempo durante la fase di pianificazione della distribuzione per i vari criteri che necessari per concedere o bloccare l'accesso agli utenti le condizioni che si sceglie di progettazione. Questo documento illustra i passaggi da eseguire per implementare i criteri di accesso condizionale sicuri ed efficaci. Prima di iniziare, assicurarsi di comprendere la modalità [accesso condizionale](overview.md) works e quando è consigliabile usarlo.

## <a name="what-you-should-know"></a>Informazioni utili

Accesso condizionale può essere paragonato a un framework che consente di controllare l'accesso alle App e risorse, invece di una funzionalità autonoma dell'organizzazione. Di conseguenza, alcune impostazioni di accesso condizionale richiedono funzionalità aggiuntive da configurare. Ad esempio, è possibile configurare un criterio che risponda a un determinato [livello di rischio di accesso](../identity-protection/howto-sign-in-risk-policy.md#what-is-the-sign-in-risk-policy). Tuttavia, un criterio basato sul livello di rischio di accesso richiede l'abilitazione di [Azure Active Directory Identity Protection](../identity-protection/overview.md).

Se sono richieste funzionalità aggiuntive, può anche essere necessario ottenere le licenze correlate. Ad esempio, mentre l'accesso condizionale è una funzionalità di Azure AD Premium P1, la protezione dell'identità richiede una licenza di Azure AD Premium P2.

Esistono due tipi di criteri di accesso condizionale: linea di base e standard. Oggetto [criterio di base](baseline-protection.md) è un criterio di accesso condizionale predefinito. L'obiettivo di questi criteri è assicurarsi di disporre almeno del livello di base della sicurezza abilitato. Criteri di base. I criteri di base sono disponibili in tutte le edizioni di Azure AD e forniscono solo opzioni di personalizzazione limitate. Se uno scenario richiede una maggiore flessibilità, disabilitare i criteri di base e implementare i requisiti in criteri standard personalizzati.

In un criterio di accesso condizionale standard, è possibile personalizzare tutte le impostazioni per modificare i criteri per i requisiti aziendali. I criteri standard richiedono una licenza di Azure AD Premium P1.

## <a name="draft-policies"></a>Disegnare i criteri

Active Directory accesso condizionale di Azure consente agli utenti di usare la protezione delle App cloud a un nuovo livello. In questo nuovo livello, le modalità di accesso a un'app cloud si basano su una valutazione dei criteri dinamica anziché su una configurazione di accesso statica. Con un criterio di accesso condizionale, è possibile definire una risposta (**scopo**) a una condizione di accesso (**in questo caso**).

![Motivo e risposta](./media/plan-conditional-access/10.png)

Definire ogni criterio di accesso condizionale che si desidera implementare usando questo modello di pianificazione. L'esercizio di pianificazione:

- Aiuta a delineare le risposte e le condizioni per ogni criterio.
- Risultati in un catalogo di criteri di accesso condizionale ben documentato per l'organizzazione. 

È possibile usare il catalogo per valutare se l'implementazione dei criteri riflette i requisiti aziendali dell'organizzazione. 

Per creare i criteri di accesso condizionale per l'organizzazione, usare il modello di esempio seguente:

|Quando accade *questo*:|Fare *questo*:|
|-|-|
|Viene eseguito un tentativo di accesso:<br>A un'app cloud *<br>- Da utenti e gruppi*<br>Usando:<br>-Condizione 1 (ad esempio, all'esterno della rete aziendale)<br>- Condizione 2 (ad esempio, piattaforme del dispositivo)|Bloccare l'accesso all'applicazione|
|Viene eseguito un tentativo di accesso:<br>A un'app cloud *<br>- Da utenti e gruppi*<br>Usando:<br>-Condizione 1 (ad esempio, all'esterno della rete aziendale)<br>- Condizione 2 (ad esempio, piattaforme del dispositivo)|Concedere l'accesso con (E):<br>-Requisito 1 (ad esempio, MFA)<br>-Requisito 2 (ad esempio, conformità del dispositivo)|
|Viene eseguito un tentativo di accesso:<br>A un'app cloud *<br>- Da utenti e gruppi*<br>Usando:<br>-Condizione 1 (ad esempio, all'esterno della rete aziendale)<br>- Condizione 2 (ad esempio, piattaforme del dispositivo)|Concedere l'accesso con (O):<br>-Requisito 1 (ad esempio, MFA)<br>-Requisito 2 (ad esempio, conformità del dispositivo)|

Come minimo, **quando accade questo** definisce l'entità di sicurezza (**chi**) che tenta di accedere a un'app cloud (**cosa**). Se necessario, è anche possibile includere **come** viene eseguito un tentativo di accesso. Nell'accesso condizionale, gli elementi che definiscono chi, cosa accade e come sono note come condizioni. Per altre informazioni, vedere [quali sono le condizioni in Active Directory accesso condizionale di Azure?](conditions.md) 

Con **fare questo**, si definisce la risposta dei criteri a una condizione di accesso. Nella risposta, si blocca o si concede l'accesso con requisiti aggiuntivi, ad esempio, l'autenticazione a più fattori (MFA). Per una panoramica completa, vedere [consente di controllare quali sono l'accesso in Active Directory accesso condizionale di Azure?](controls.md)  

La combinazione delle condizioni con i controlli di accesso rappresenta un tipo di criteri di accesso condizionale.

![Motivo e risposta](./media/plan-conditional-access/51.png)

Per altre informazioni, vedere [Elementi necessari per il funzionamento di un criterio](best-practices.md#whats-required-to-make-a-policy-work).

A questo punto si può stabilire uno standard di denominazione per i criteri. Lo standard di denominazione aiuta a individuare i criteri e comprenderne le finalità senza aprirli nel portale di amministrazione di Azure. Denomina il criterio da visualizzare:

- Numero di sequenza
- Cloud a cui si applica
- Risposta
- A chi si applica
- Quando si applica (se applicabile)
 
![Standard di denominazione](./media/plan-conditional-access/11.png)

Mentre un nome descrittivo consente di mantenere una panoramica dell'implementazione di accesso condizionale, il numero di sequenza è utile se è necessario fare riferimento a un criterio in una conversazione. Ad esempio, se si parla di un amministratore di colleghi sul telefono, è possibile chiedere loro di aprire Criteri EM063 per risolvere un problema.

Ad esempio, il nome seguente indica che il criterio richiede l'autenticazione a più fattori per gli utenti di marketing su reti esterne che usano l'app Dynamics CRP:

`CA01 - Dynamics CRP: Require MFA For marketing When on external networks`

Oltre ai criteri attivi, è consigliabile implementare anche criteri disabilitati che agiscano come [controlli di accesso resilienti secondari in scenari di emergenza/interruzione dei servizi](../authentication/concept-resilient-controls.md). Lo standard di denominazione per i criteri di emergenza deve includere alcuni altri elementi: 

- `ENABLE IN EMERGENCY` all'inizio in modo che il nome si distingua dagli altri criteri.
- Il nome dell'interruzione a cui deve essere applicato.
- Un numero di sequenza di ordinamento per consentire all'amministratore di sapere in che ordine devono essere abilitati i criteri. 

Ad esempio, il nome seguente indica che questo criterio è il primo criterio su quattro che se è presente un'interruzione di MFA, è consigliabile abilitare:

`EM01 - ENABLE IN EMERGENCY, MFA Disruption[1/4] - Exchange SharePoint: Require hybrid Azure AD join For VIP users`

## <a name="plan-policies"></a>Pianificare i criteri

Quando si pianifica una soluzione di criteri di accesso condizionale, valutare se è necessario creare i criteri per ottenere i risultati seguenti. 

### <a name="block-access"></a>Blocca accesso

L'opzione per bloccare l'accesso è notevolmente efficace perché:

- È prioritaria rispetto a tutte le altre assegnazioni per un utente
- Consente di bloccare l'accesso al tenant da parte dell'intera organizzazione
 
Se si vuole bloccare l'accesso per tutti gli utenti, è necessario escludere dai criteri almeno un utente (in genere gli account di accesso di emergenza). Per altre informazioni, vedere come [selezionare utenti e gruppi](block-legacy-authentication.md#select-users-and-cloud-apps).  

### <a name="require-mfa"></a>Richiedere l'autenticazione MFA

Per semplificare l'esperienza di accesso degli utenti, è possibile consentire di accedere alle app cloud usando un nome utente e una password. Tuttavia, in genere esistono almeno alcuni scenari per cui è consigliabile richiedere una forma di verifica degli account più avanzata. Con un criterio di accesso condizionale, è possibile limitare il requisito di autenticazione a più fattori per determinati scenari. 

I casi d'uso comuni per richiedere l'autenticazione MFA riguardano l'accesso:

- [Da parte degli amministratori](howto-baseline-protect-administrators.md)
- [A specifiche app](app-based-mfa.md) 
- [Da percorsi di rete che non si considerano attendibili](untrusted-networks.md).

### <a name="respond-to-potentially-compromised-accounts"></a>Rispondere ad account potenzialmente compromessi

Con i criteri di accesso condizionale, è possibile implementare risposte automatiche agli accessi dalle identità potenzialmente compromesse. La probabilità che un account sia stato compromesso viene espressa sotto forma di livelli di rischio. Identity Protection calcola due livelli di rischio: rischio di accesso e rischio utente. Per implementare una risposta a un rischio di accesso sono disponibili due opzioni:

- [La condizione di rischio di accesso](conditions.md#sign-in-risk) nei criteri di accesso condizionale
- [I criteri di rischio di accesso](../identity-protection/howto-sign-in-risk-policy.md) in Identity Protection 

Usare il rischio di accesso come condizione è il metodo consigliato perché offre più opzioni di personalizzazione.

Il livello di rischio utente è disponibile solo sotto forma di [criteri di rischio utente](../identity-protection/howto-user-risk-policy.md) di Identity Protection. 

Per altre informazioni, vedere [Cos'è Azure Active Directory Identity Protection?](../identity-protection/overview.md) 

### <a name="require-managed-devices"></a>Richiedere dispositivi gestiti

L'ampia diffusione dei dispositivi supportati per l'accesso alle risorse cloud offre notevoli vantaggi in termini di produttività degli utenti. In alcuni casi, tuttavia, può essere necessario evitare che alcune risorse dell'ambiente risultino accessibili a dispositivi con un livello di protezione sconosciuto. Per le risorse di questo tipo è opportuno definire un requisito di accessibilità in modo che gli utenti possano accedervi solo tramite un dispositivo gestito. Per altre informazioni, vedere [come richiedere i dispositivi gestiti per accedere all'app cloud con l'accesso condizionale](require-managed-devices.md). 

### <a name="require-approved-client-apps"></a>Richiedere app client approvate

Una delle prime decisioni da prendere per gli scenari BYOD (Bring Your Own Device) è se gestire l'intero dispositivo o solo i dati al suo interno. I dipendenti usano dispositivi mobili sia per le attività personali che per quelle aziendali. È importante assicurarsi che i dipendenti siano produttivi e al contempo evitare la perdita di dati. Con l'accesso condizionale di Azure Active Directory (Azure AD), è possibile limitare l'accesso alle App cloud per le app client approvate che consente di proteggere i dati aziendali. Per altre informazioni, vedere [come richiedere l'App client approvate per accedere all'app cloud con l'accesso condizionale](app-based-conditional-access.md).

### <a name="block-legacy-authentication"></a>Bloccare l'autenticazione legacy

Azure AD supporta diversi dei protocolli di autenticazione e autorizzazione più ampiamente usati. Come si può impedire alle app che usano l'autenticazione legacy di accedere alle risorse dei tenant? Si consiglia di appena bloccarli mediante un criterio di accesso condizionale. Se necessario, è possibile autorizzare solo determinati utenti e percorsi di rete specifici per l’uso delle app basate sull’autenticazione legacy. Per altre informazioni, vedere [come bloccare l'autenticazione legacy per Azure AD con accesso condizionale](block-legacy-authentication.md).

## <a name="test-your-policy"></a>Verificare i criteri

Prima di implementare i criteri nell'ambiente di produzione, è consigliabile testarli per verificare che tutto funzioni come previsto.

1. Creare utenti di test
1. Creare un piano di test
1. Configurare i criteri
1. Valutare un accesso simulato
1. Verificare i criteri
1. Pulizia

### <a name="create-test-users"></a>Creare utenti di test

Per testare un criterio, creare un set di utenti simili agli utenti nell'ambiente in uso. La creazione di utenti di test consente di verificare che i criteri funzionino come previsto di applicarli agli utenti reali e potenzialmente compromettere il loro accesso alle app e alle risorse. 

Alcune organizzazioni hanno tenant di test a questo scopo. Tuttavia, può essere difficile ricreare tutte le condizioni e le app in un tenant di test per verificare completamente i risultati di un criterio. 

### <a name="create-a-test-plan"></a>Creare un piano di test

Il piano di test è importante per disporre di un confronto tra i risultati previsti e i risultati effettivi. È sempre necessario definire il risultato atteso prima di eseguire un test. Nella tabella seguente vengono descritti alcuni test case di esempio. Modificare gli scenari e i risultati previsti in base al modo in cui sono configurati i criteri di accesso condizionale.

|Policy |Scenario |Risultato previsto | Risultato |
|---|---|---|---|
|[Richiedi autenticazione a più fattori quando non al lavoro](https://docs.microsoft.com/azure/active-directory/conditional-access/untrusted-networks)|Un utente autorizzato accede ad *App* da un percorso attendibile/dal posto di lavoro|All'utente non viene richiesta l'autenticazione a più fattori| |
|[Richiedi autenticazione a più fattori quando non al lavoro](https://docs.microsoft.com/azure/active-directory/conditional-access/untrusted-networks)|Un utente autorizzato accede ad *App* da un percorso non attendibile/non dal posto di lavoro|All'utente viene richiesta l'autenticazione a più fattori e può eseguire correttamente l'accesso| |
|[Richiedi autenticazione a più fattori (per l'amministratore)](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-baseline-protect-administrators)|L'amministratore globale accede ad *App*|All'amministratore viene richiesta l'autenticazione a più fattori| |
|[Accessi a rischio](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-sign-in-risk-policy)|L'utente accede ad *App* usando un [browser Tor](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection-playbook)|All'amministratore viene richiesta l'autenticazione a più fattori| |
|[Gestione del dispositivo](https://docs.microsoft.com/azure/active-directory/conditional-access/require-managed-devices)|Un utente autorizzato cerca di accedere da un dispositivo autorizzato|Accesso concesso| |
|[Gestione del dispositivo](https://docs.microsoft.com/azure/active-directory/conditional-access/require-managed-devices)|Un utente autorizzato cerca di accedere da un dispositivo non autorizzato|Accesso bloccato| |
|[Modifica password per gli utenti a rischio](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-user-risk-policy)|Un utente autorizzato cerca di accedere con credenziali compromesse (accesso ad alto rischio)|All'utente viene richiesto di cambiare la password o l'accesso viene bloccato in base al criterio| |

### <a name="configure-the-policy"></a>Configurare i criteri

La gestione di criteri di accesso condizionale è un'attività manuale. Nel portale di Azure, è possibile gestire i criteri di accesso condizionale in una posizione centrale: pagina di accesso condizionale. Un punto di ingresso alla pagina di accesso condizionale è il **sicurezza** sezione il **Active Directory** riquadro di spostamento. 

![Accesso condizionale](media/plan-conditional-access/03.png)

Per altre informazioni su come creare criteri di accesso condizionale, vedere [Richiedi autenticazione a più fattori per App specifiche con Azure Active Directory l'accesso condizionale](app-based-mfa.md). Questo argomento di avvio rapido consente di:

- Acquisire familiarità con l'interfaccia utente.
- Ottenere una panoramica del funzionamento dell'accesso condizionale. 

### <a name="evaluate-a-simulated-sign-in"></a>Valutare un accesso simulato

Ora che sono stati configurati i criteri di accesso condizionale, è possibile sapere se funzionano come previsto. Come primo passaggio, usare lo [strumento per i criteri What If](what-if-tool.md) dell'accesso condizionale per simulare un accesso dell'utente di test. La simulazione valuta l'impatto di questo accesso sui criteri e genera un report di simulazione.

>[!NOTE]
> Mentre un'esecuzione simulata fornisce impressione dell'impatto prodotto che dispone di un criterio di accesso condizionale, non sostituisce un'esecuzione dei test effettivo.

### <a name="test-your-policy"></a>Verificare i criteri

Eseguire i test case in base al piano di test. In questo passaggio viene eseguito un test completo di ogni criterio per gli utenti di test, per verificare che il loro comportamento sia quello previsto. Usare gli scenari creati in precedenza per eseguire ogni test.

È importante assicurarsi di testare i criteri di esclusione di un criterio. È ad esempio possibile escludere un utente o gruppo da un criterio che richiede l'autenticazione MFA. Verificare se gli utenti esclusi richiesto per MFA, perché la combinazione di altri criteri potrebbe richiedere l'autenticazione MFA per gli utenti.

### <a name="cleanup"></a>Pulizia

La procedura di pulizia comprende i passaggi seguenti:

1. Disabilitare il criterio.
1. Rimuovere gli utenti e gruppi assegnati.
1. Eliminare gli utenti di test.  

## <a name="move-to-production"></a>Passare in produzione

Quando i nuovi criteri sono pronti per l'ambiente, distribuirli in più fasi:

- Dare comunicazione delle modifiche agli utenti finali.
- Iniziare con un set ridotto di utenti e verificare che il criterio abbia il comportamento previsto.
- Quando si espande un criterio per includere un maggior numero di utenti, continuare a escludere tutti gli amministratori. Escludere gli amministratori assicura che qualcuno possa comunque accedere a un criterio, se è necessaria una modifica.
- Applicare un criterio a tutti gli utenti solo se necessario.

Come procedura consigliata, creare almeno un account utente che sia:

- Dedicato all'amministrazione dei criteri
- Escluso da tutti i criteri

## <a name="rollback-steps"></a>Procedura di ripristino dello stato precedente

Nel caso in cui sia necessario eseguire il rollback dei criteri appena implementati, usare uno o più delle opzioni seguenti:

1. **Disabilitare il criterio** -La disabilitazione di un criterio garantisce che non venga applicato quando un utente tenta di accedere. È sempre possibile tornare indietro e abilitare il criterio quando si vorrà utilizzarlo.

   ![Disabilitare il criterio](media/plan-conditional-access/07.png)

1. **Escludere un utente o gruppo da un criterio** -Se un utente non è in grado di accedere all'app, è possibile scegliere di escluderlo dal criterio

   ![Escludere utenti](media/plan-conditional-access/08.png)

   > [!NOTE]
   > Questa opzione deve essere usata solo se necessario, solo in situazioni in cui l'utente è attendibile. L'utente deve essere nuovamente aggiunto al criterio o al gruppo appena possibile.

1. **Eliminare il criterio**: se il criterio non è più necessario, eliminarlo.

## <a name="next-steps"></a>Passaggi successivi

Consultare la [Documentazione sull'accesso condizionale di Azure AD](index.yml) per una panoramica delle informazioni disponibili.
