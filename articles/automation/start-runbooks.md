---
title: Avviare un runbook in Automazione di Azure
description: Riepiloga le diverse modalità che è possibile usare per avviare un Runbook in Automazione di Azure e fornisce dettagli sull'uso del portale di Azure e di Windows PowerShell.
services: automation
ms.service: automation
ms.subservice: process-automation
author: bobbytreed
ms.author: robreed
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: a42ba0a385b73032e3d4045094bc88613016b849
ms.sourcegitcommit: 670c38d85ef97bf236b45850fd4750e3b98c8899
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/08/2019
ms.locfileid: "68850334"
---
# <a name="start-a-runbook-in-azure-automation"></a>Avviare un runbook in Automazione di Azure

La tabella seguente aiuta a determinare il metodo per avviare un runbook in Automazione di Azure più adatto per uno scenario specifico. Questo articolo include informazioni dettagliate sull'avvio di un Runbook con il portale di Azure e con Windows PowerShell. Le informazioni sulle altre modalità sono contenute in altri documenti accessibili dai collegamenti seguenti.

| **Metodo** | **Caratteristiche** |
| --- | --- |
| [Portale di Azure](#start-a-runbook-with-the-azure-portal) |<li>Metodo più semplice con interfaccia utente interattiva.<br> <li>Modulo per specificare valori di parametri semplici.<br> <li>Facilità di controllo dello stato dei processi.<br> <li>Accesso autenticato con le informazioni di accesso di Azure. |
| [Windows PowerShell](/powershell/module/azurerm.automation/start-azurermautomationrunbook) |<li>Chiamata mediante cmdlet di Windows PowerShell nella riga di comandi.<br> <li>Possibilità di inclusione in una soluzione automatizzata con più passaggi.<br> <li>Autenticazione della richiesta con un certificato oppure con un'entità utente/entità servizio OAuth.<br> <li>Possibilità di specificare valori di parametri semplici e complessi.<br> <li>Possibilità di controllare lo stato dei processi.<br> <li>Obbligo per il client di supporto dei cmdlet di PowerShell. |
| [API di Automazione di Azure](/rest/api/automation/) |<li>Modalità più flessibile, ma anche più complessa.<br> <li>Possibilità di chiamata da qualsiasi codice personalizzato in grado di creare richieste HTTP.<br> <li>Autenticazione della richiesta con un certificato oppure con un'entità utente/entità servizio OAuth.<br> <li>Possibilità di specificare valori di parametri semplici e complessi. *Se si chiama un runbook Python usando l'API, il payload JSON deve essere serializzato.*<br> <li>Possibilità di controllare lo stato dei processi. |
| [Webhook](automation-webhooks.md) |<li>Avvio di Runbook da una singola richiesta HTTP.<br> <li>Autenticazione con token di sicurezza nell'URL.<br> <li>Impossibilità per il client di eseguire l'override dei valori di parametri specificati al momento della creazione del webhook. Possibilità per il Runbook di definire un singolo parametro popolato con i dettagli della richiesta HTTP.<br> <li>Impossibilità di tenere traccia dello stato dei processi tramite URL webhook. |
| [Risposta all'avviso di Azure](../log-analytics/log-analytics-alerts.md) |<li>Avviare un runbook in risposta all'avviso di Azure.<br> <li>Configurare webhook per runbook e collegare all'avviso.<br> <li>Autenticazione con token di sicurezza nell'URL. |
| [Pianificare](automation-schedules.md) |<li>Avvio automatico dei runbook in base a una pianificazione oraria, giornaliera, settimanale o mensile.<br> <li>Modifica della pianificazione tramite il portale di Azure, i cmdlet di PowerShell o l'API di Azure.<br> <li>Possibilità di specificare i valori di parametri da usare con la pianificazione. |
| [Da un altro Runbook](automation-child-runbooks.md) |<li>Uso di un Runbook come attività in un altro Runbook.<br> <li>Utile per funzionalità usate da più Runbook.<br> <li>Possibilità di specificare valori di parametri per Runbook figlio e usare l'output in Runbook padre. |

L'immagine seguente illustra in dettaglio il processo nel ciclo di vita di un runbook. Include vari metodi di avvio di un runbook in Automazione di Azure, i componenti necessari per il ruolo di lavoro ibrido per runbook per eseguire i runbook di Automazione di Azure e le interazioni tra i vari componenti. Per altre informazioni sull'esecuzione di runbook di automazione nel proprio data center, vedere [Ruoli di lavoro ibridi per runbook](automation-hybrid-runbook-worker.md)

![Architettura dei runbook](media/automation-starting-runbook/runbooks-architecture.png)

## <a name="start-a-runbook-with-the-azure-portal"></a>Avviare un runbook con il portale di Azure

1. Nel portale di Azure selezionare **Automazione** e quindi fare clic sul nome di un account di automazione.
2. Nel menu Hub selezionare **Runbook**.
3. Nella pagina **Runbook** selezionare un runbook e quindi fare clic su **Avvia**.
4. Se il runbook ha parametri, viene chiesto di specificare i valori con una casella di testo per ogni parametro. Per altre informazioni sui parametri, vedere [Parametri di runbook](#runbook-parameters).
5. Nella pagina **Processo** è possibile visualizzare lo stato del processo del runbook.

## <a name="start-a-runbook-with-powershell"></a>Avviare un runbook con PowerShell

È possibile usare [Start-AzureRmAutomationRunbook](https://docs.microsoft.com/powershell/module/azurerm.automation/start-azurermautomationrunbook) per avviare un runbook con Windows PowerShell. Il codice di esempio seguente avvia un Runbook denominato Test-Runbook.

```azurepowershell-interactive
Start-AzureRmAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" -ResourceGroupName "ResourceGroup01"
```

Start-AzureRmAutomationRunbook restituisce un oggetto processo che è possibile usare per tenere traccia dello stato dopo l'avvio del runbook. È quindi possibile usare questo oggetto processo con [Get-AzureRmAutomationJob](https://docs.microsoft.com/powershell/module/azurerm.automation/get-azurermautomationjob) per determinare lo stato del processo e con [Get-AzureRmAutomationJobOutput](https://docs.microsoft.com/powershell/module/azurerm.automation/get-azurermautomationjoboutput) per ottenere il relativo output. L'esempio di codice seguente avvia un Runbook denominato Test-Runbook, attende che venga completato e quindi visualizza l'output corrispondente.

```azurepowershell-interactive
$runbookName = "Test-Runbook"
$ResourceGroup = "ResourceGroup01"
$AutomationAcct = "MyAutomationAccount"

$job = Start-AzureRmAutomationRunbook –AutomationAccountName $AutomationAcct -Name $runbookName -ResourceGroupName $ResourceGroup

$doLoop = $true
While ($doLoop) {
   $job = Get-AzureRmAutomationJob –AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup
   $status = $job.Status
   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped"))
}

Get-AzureRmAutomationJobOutput –AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup –Stream Output
```

Se il runbook richiede parametri, è necessario fornirli come [tabella hash](https://technet.microsoft.com/library/hh847780.aspx). La chiave della tabella hash deve corrispondere al nome del parametro e il valore è il valore del parametro. L'esempio seguente illustra come avviare un Runbook con due parametri di stringa denominati FirstName e LastName, un parametro di tipo intero denominato RepeatCount e un parametro booleano denominato Show. Per altre informazioni sui parametri, vedere [Parametri di runbook](#runbook-parameters) più avanti.

```azurepowershell-interactive
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Start-AzureRmAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" -ResourceGroupName "ResourceGroup01" –Parameters $params
```

## <a name="runbook-parameters"></a>Parametri di Runbook

Quando si avvia un runbook dal portale di Azure o da Windows PowerShell, l'istruzione viene inviata attraverso il servizio Web Automazione di Azure. Questo servizio non supporta i parametri con tipi di dati complessi. Se è necessario specificare un valore per un parametro complesso, eseguire la chiamata inline da un altro Runbook come descritto in [Runbook figlio in Automazione di Azure](automation-child-runbooks.md).

Il servizio Web Automazione di Azure fornisce funzionalità speciali per i parametri che usano determinati tipi di dati, come descritto nelle sezioni seguenti:

### <a name="named-values"></a>Valori denominati

Se il parametro è un tipo di dati [object], è possibile usare il formato JSON seguente per inviargli un elenco di valori denominati: *{Name1:'Value1', Name2:'Value2', Name3:'Value3'}* . Questi valori devono essere tipi semplici. Il runbook riceve il parametro come [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject) con proprietà che corrispondono a ogni valore denominato.

Si consideri il Runbook di test seguente che accetta un parametro denominato user.

```powershell
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][object]$user
   )
    $userObject = $user | ConvertFrom-JSON
    if ($userObject.Show) {
        foreach ($i in 1..$userObject.RepeatCount) {
            $userObject.FirstName
            $userObject.LastName
        }
    }
}
```

È possibile usare il testo seguente per il parametro user.

```json
{FirstName:'Joe',LastName:'Smith',RepeatCount:'2',Show:'True'}
```

Si ottiene l'output seguente:

```output
Joe
Smith
Joe
Smith
```

### <a name="arrays"></a>Matrici

Se il parametro è una matrice, ad esempio [array] o [string[]], è possibile usare il formato JSON seguente per inviare un elenco di valori: *[Value1, Value2, Value3]* . Questi valori devono essere tipi semplici.

Si consideri il Runbook di test seguente che accetta un parametro denominato *user*.

```powershell
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][array]$user
   )
    if ($user[3]) {
        foreach ($i in 1..$user[2]) {
            $ user[0]
            $ user[1]
        }
    }
}
```

È possibile usare il testo seguente per il parametro user.

```input
["Joe","Smith",2,true]
```

Si ottiene l'output seguente:

```output
Joe
Smith
Joe
Smith
```

### <a name="credentials"></a>Credenziali

Se il parametro è un tipo di dati **PSCredential**, è possibile specificare il nome di un [asset credenziali](automation-credentials.md)di Automazione di Azure. Il runbook recupera le credenziali con il nome specificato.

Si consideri il Runbook di test seguente che accetta un parametro denominato credential.

```powershell
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][PSCredential]$credential
   )
   $credential.UserName
}
```

È possibile usare il testo seguente per il parametro user presumendo l'esistenza di un asset credenziali denominato *My Credential*.

```input
My Credential
```

Presupponendo che il nome utente nelle credenziali sia *jsmith*, si ottiene l'output seguente:

```output
jsmith
```

## <a name="next-steps"></a>Passaggi successivi

* L'architettura runbook nell'articolo corrente offre una panoramica generale delle risorse di gestione di runbook in Azure e in locale con il ruolo di lavoro ibrido per runbook. Per altre informazioni sull'esecuzione di runbook di automazione nel proprio data center, vedere [Ruoli di lavoro ibridi per runbook](automation-hybrid-runbook-worker.md).
* Per altre informazioni sulla creazione di runbook modulari che possono essere usati da altri runbook per funzioni comuni o specifiche, fare riferimento a [Runbook figlio](automation-child-runbooks.md).
* Per altre informazioni su PowerShell, inclusi i moduli di riferimento e apprendimento del linguaggio, vedere la [documentazione di PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/overview).
