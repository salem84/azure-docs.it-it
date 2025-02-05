---
title: Scalabilità automatica avanzata con macchine virtuali di Azure
description: Usa Resource Manager e i set di scalabilità di macchine virtuali di Microsoft Azure con più regole e profili che inviano messaggi di posta elettronica e chiamano URL di webhook con azioni di scalabilità.
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 02/22/2016
ms.author: ancav
ms.subservice: autoscale
ms.openlocfilehash: 6da653bc94c8b549282ab9124dba23b08771c5f1
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60787781"
---
# <a name="advanced-autoscale-configuration-using-resource-manager-templates-for-vm-scale-sets"></a>Configurazione di scalabilità automatica avanzata con modelli di Resource Manager per set di scalabilità di macchine virtuali di Microsoft Azure
È possibile aumentare e ridurre il numero di istanze dei set di scalabilità di macchine virtuali in base ai valori soglia per le metriche delle prestazioni, a una pianificazione ricorrente oppure a una data specifica. È anche possibile configurare notifiche di posta elettronica e webhook per le azioni di scalabilità. Questa procedura dettagliata illustra un esempio di configurazione di tutti tali oggetti usando in modello di Resource Manager in un set di scalabilità di macchine virtuali.

> [!NOTE]
> Mentre questa procedura dettagliata illustra i passaggi per set di scalabilità di macchine virtuali, le stesse informazioni si applicano alla scalabilità automatica di [Servizi Cloud](https://azure.microsoft.com/services/cloud-services/), [App Web del servizio app](https://azure.microsoft.com/services/app-service/web/) e [servizi di gestione API](https://docs.microsoft.com/azure/api-management/api-management-key-concepts) Per l'impostazione di una scalabilità in ingresso/uscita semplice in un set di scalabilità di macchine virtuali basato su una semplice metrica delle prestazioni, ad esempio CPU, vedere i documenti [Linux](../../virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-cli.md) e [Windows](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md)
>
>

## <a name="walkthrough"></a>Procedura dettagliata
In questa procedura dettagliata viene usato [Esplora risorse di Azure](https://resources.azure.com/) per configurare e aggiornare l'impostazione di ridimensionamento automatico per un set di scalabilità. Esplora risorse di Azure consente di gestire facilmente le risorse di Azure con i modelli di Resource Manager. Se non si ha esperienza con lo strumento Esplora risorse di Azure, vedere [questa introduzione](https://azure.microsoft.com/blog/azure-resource-explorer-a-new-tool-to-discover-the-azure-api/).

1. Distribuire un nuovo set di scalabilità con un'impostazione di scalabilità automatica di base. Questo articolo usa quello della raccolta di guide introduttive di Azure che ha un set di scalabilità Windows con il modello di scalabilità automatica di base. I set di scalabilità Linux funzionano allo stesso modo.
2. Una volta creato il set di scalabilità, passare alla risorsa set di scalabilità da Esplora risorse di Azure. Sotto il nodo Microsoft.Insights viene visualizzato quanto segue.

    ![Azure Explorer](media/autoscale-virtual-machine-scale-sets/azure_explorer_navigate.png)

    L'esecuzione del modello ha creato un'impostazione predefinita di ridimensionamento automatico con il nome **"autoscalewad"** . Sul lato destro è possibile visualizzare la definizione completa di questa impostazione di ridimensionamento automatico. In questo caso, l'impostazione di ridimensionamento automatico predefinita è inclusa in una regola di aumento e riduzione del numero di istanze basata sulla percentuale di CPU.  

3. Ora è possibile aggiungere altri profili e regole basati sulla pianificazione o su specifici requisiti. Viene creata un'impostazione di ridimensionamento automatico con tre profili. Per conoscere i profili e le regole nel ridimensionamento automatico, vedere [Procedure consigliate per il ridimensionamento automatico](autoscale-best-practices.md).  

    | Profili e regole | Descrizione |
    |--- | --- |
    | **Profilo** |**Basato su prestazioni/metrica** |
    | Regola |Numero di messaggi della coda del bus di servizio > x |
    | Regola |Numero di messaggi della coda del bus di servizio < y |
    | Regola |% CPU > n |
    | Regola |% CPU < p |
    | **Profilo** |**Ore della mattina dei giorni feriali (nessuna regola)** |
    | **Profilo** |**Giorno di lancio del prodotto (nessuna regola)** |

4. Di seguito viene descritto uno senario ipotetico scenario di ridimensionamento per la procedura dettagliata.

   * **Basato sul carico** - Si vuole aumentare o ridurre il numero di istanze in base al carico sull'applicazione ospitata nel set di scalabilità.*
   * **Dimensioni della coda di messaggi** - Si usa una coda del bus di servizio per i messaggi in arrivo nell'applicazione. Si usa il numero di messaggi della coda e la percentuale di CPU e si configura un profilo predefinito per attivare un'azione di scalabilità se il conteggio dei messaggi o la CPU raggiunge la soglia.\*
   * **Ora della settimana e del giorno** - Si vuole un profilo basato su un'ora del giorno con ricorrenza settimanale denominato "Ore della mattina dei giorni feriali". In base ai dati cronologici, si è stabilito che è meglio avere un certo numero di istanze di VM per gestire il carico dell'applicazione durante questo orario.\*
   * **Date speciali** - È stato aggiunto un profilo "Giorni di lancio del prodotto". Si pianificano in anticipo date specifiche in modo che l'applicazione sia pronta a gestire il carico derivante da annunci di marketing e dall'inserimento di un nuovo prodotto nell'applicazione.\*
   * *Gli ultimi due profili possono anche contenere altre regole basate sulla metrica delle prestazioni. In questo caso, ho deciso di non averne e di affidarmi alle regole basate sulla metrica delle prestazioni predefinite. Le regole sono facoltative per i profili ricorrenti e basati sulle date.*

     La classificazione in ordine di priorità dei profili e delle regole con il motore di ridimensionamento automatico è illustrata anche nell'articolo [Procedure consigliate per il ridimensionamento automatico](autoscale-best-practices.md).
     Per un elenco di metriche comuni per la scalabilità automatica, vedere [Metriche comuni per la scalabilità automatica](autoscale-common-metrics.md).

5. Verificare che Esplora risorse sia in modalità **Lettura/Scrittura**.

    ![Autoscalewad, impostazione di ridimensionamento automatico predefinita](media/autoscale-virtual-machine-scale-sets/autoscalewad.png)

6. Fare clic su Edit. **Sostituire** l'elemento "profiles" nell'impostazione di scalabilità automatica con la configurazione seguente:

    ![Profili](media/autoscale-virtual-machine-scale-sets/profiles.png)

    ```
    {
            "name": "Perf_Based_Scale",
            "capacity": {
              "minimum": "2",
              "maximum": "12",
              "default": "2"
            },
            "rules": [
              {
                "metricTrigger": {
                  "metricName": "MessageCount",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 10
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "MessageCount",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 3
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT30M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 85
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "Percentage CPU",
                  "metricNamespace": "",
                  "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
                  "timeGrain": "PT5M",
                  "statistic": "Average",
                  "timeWindow": "PT30M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 60
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              }
            ]
          },
          {
            "name": "Weekday_Morning_Hours_Scale",
            "capacity": {
              "minimum": "4",
              "maximum": "12",
              "default": "4"
            },
            "rules": [],
            "recurrence": {
              "frequency": "Week",
              "schedule": {
                "timeZone": "Pacific Standard Time",
                "days": [
                  "Monday",
                  "Tuesday",
                  "Wednesday",
                  "Thursday",
                  "Friday"
                ],
                "hours": [
                  6
                ],
                "minutes": [
                  0
                ]
              }
            }
          },
          {
            "name": "Product_Launch_Day",
            "capacity": {
              "minimum": "6",
              "maximum": "20",
              "default": "6"
            },
            "rules": [],
            "fixedDate": {
              "timeZone": "Pacific Standard Time",
              "start": "2016-06-20T00:06:00Z",
              "end": "2016-06-21T23:59:00Z"
            }
          }
    ```
    Per i campi e i valori supportati, vedere la [documentazione sull'API REST per il ridimensionamento automatico](https://msdn.microsoft.com/library/azure/dn931928.aspx). Ora l'impostazione di scalabilità automatica contiene i tre profili descritti in precedenza.

7. Verrà esaminata infine la sezione **notification** per la scalabilità automatica. Le notifiche di ridimensionamento automatico consentono di eseguire tre operazioni quando un'azione di aumento o riduzione del numero di istanze viene correttamente attivata.
   - Inviare una notifica all'amministratore e ai coamministratori della sottoscrizione.
   - Inviare un messaggio di posta elettronica a un set di utenti.
   - Attivare una chiamata webhook. Se attivato, questo webhook invia i metadati sulla condizione di scalabilità automatica e sulla risorsa del set di scalabilità. Per altre informazioni sul payload del webhook di ridimensionamento automatico, vedere [Configurare notifiche webhook e di posta elettronica per il ridimensionamento automatico](autoscale-webhook-email.md).

   Aggiungere il codice seguente all'impostazione di ridimensionamento automatico sostituendo l'elemento **notification** il cui valore è null.

   ```
   "notifications": [
      {
        "operation": "Scale",
        "email": {
          "sendToSubscriptionAdministrator": true,
          "sendToSubscriptionCoAdministrators": false,
          "customEmails": [
              "user1@mycompany.com",
              "user2@mycompany.com"
              ]
        },
        "webhooks": [
          {
            "serviceUri": "https://foo.webhook.example.com?token=abcd1234",
            "properties": {
              "optional_key1": "optional_value1",
              "optional_key2": "optional_value2"
            }
          }
        ]
      }
    ]

   ```

   Fare clic sul pulsante **Put** in Esplora risorse per aggiornare l'impostazione di scalabilità automatica.

È stata aggiornata un'impostazione di ridimensionamento automatico in un set di scalabilità di macchine virtuali per includere più profili di scalabilità e notifiche di scalabilità.

## <a name="next-steps"></a>Fasi successive
Per altre informazioni sulla scalabilità automatica, usare questi collegamenti.

[Risolvere i problemi di scalabilità automatica con set di scalabilità di macchine virtuali](../../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)

[Metriche comuni per il ridimensionamento automatico](autoscale-common-metrics.md)

[Procedure consigliate per il ridimensionamento automatico di Azure](autoscale-best-practices.md)

[Gestire il ridimensionamento automatico con PowerShell](../../azure-monitor/platform/powershell-quickstart-samples.md#create-and-manage-autoscale-settings)

[Gestire il ridimensionamento automatico con l'interfaccia della riga di comando](cli-samples.md#autoscale)

[Configurare notifiche webhook e di posta elettronica per il ridimensionamento automatico](autoscale-webhook-email.md)

Informazioni di riferimento sul modello [Microsoft.Insights/autoscalesettings](/azure/templates/microsoft.insights/autoscalesettings)
