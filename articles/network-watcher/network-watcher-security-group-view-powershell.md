---
title: Analizzare la protezione di rete con la visualizzazione del gruppo di sicurezza di rete di Network Watcher di Azure - PowerShell | Documentazione Microsoft
description: Questo articolo descrive come usare PowerShell per analizzare la protezione di macchine virtuali con la visualizzazione di un gruppo di sicurezza.
services: network-watcher
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
ms.assetid: 04e76b49-6a1b-4d0f-9a9b-51cf2f4df5a2
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: kumud
ms.openlocfilehash: 4c7b79460169612a046b19a4d66f222936710a8e
ms.sourcegitcommit: 19a821fc95da830437873d9d8e6626ffc5e0e9d6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/29/2019
ms.locfileid: "70163890"
---
# <a name="analyze-your-virtual-machine-security-with-security-group-view-using-powershell"></a>Analizzare la protezione della macchina virtuale visualizzando un gruppo di sicurezza con PowerShell

> [!div class="op_single_selector"]
> - [PowerShell](network-watcher-security-group-view-powershell.md)
> - [Interfaccia della riga di comando di Azure](network-watcher-security-group-view-cli.md)
> - [API REST](network-watcher-security-group-view-rest.md)

La visualizzazione di un gruppo di sicurezza consente di recuperare le regole di sicurezza di rete configurate ed effettive applicate a una macchina virtuale. Questa funzionalità è utile per controllare e diagnosticare i gruppi di sicurezza di rete e le regole configurate in una macchina virtuale per verificare che il traffico viene consentito o negato in modo corretto. Questo articolo illustra come recuperare le regole di sicurezza configurate ed effettive applicate a una macchina virtuale tramite PowerShell


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Prima di iniziare

In questo scenario, il cmdlet `Get-AzNetworkWatcherSecurityGroupView` viene eseguito per recuperare le informazioni sulla regola di sicurezza.

Questo scenario presuppone il completamento dei passaggi descritti in [Creare un servizio Network Watcher](network-watcher-create.md) per creare un servizio Network Watcher.

## <a name="scenario"></a>Scenario

Lo scenario illustrato in questo articolo recupera le regole di sicurezza configurate ed effettive applicate a una determinata macchina virtuale.

## <a name="retrieve-network-watcher"></a>Recuperare Network Watcher

Il primo passaggio consente di recuperare l'istanza di Network Watcher. Questa variabile viene passata al cmdlet `Get-AzNetworkWatcherSecurityGroupView`.

```powershell
$networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" }
```

## <a name="get-a-vm"></a>Ottenere una macchina virtuale

È necessario che una macchina virtuale esegua il cmdlet `Get-AzNetworkWatcherSecurityGroupView`. Nell'esempio seguente viene ottenuto un oggetto macchina virtuale.

```powershell
$VM = Get-AzVM -ResourceGroupName testrg -Name testvm1
```

## <a name="retrieve-security-group-view"></a>Recuperare la visualizzazione del gruppo di sicurezza

Il passaggio successivo prevede il recupero del risultato della visualizzazione del gruppo di sicurezza.

```powershell
$secgroup = Get-AzNetworkWatcherSecurityGroupView -NetworkWatcher $networkWatcher -TargetVirtualMachineId $VM.Id
```

## <a name="viewing-the-results"></a>Visualizzazione dei risultati

L'esempio seguente è una risposta abbreviata dei risultati restituiti. I risultati mostrano tutte le regole di sicurezza effettive e applicate alla macchina virtuale, suddivise nei gruppi **NetworkInterfaceSecurityRules**, **DefaultSecurityRules** e **EffectiveSecurityRules**.

```
NetworkInterfaces : [
                      {
                        "NetworkInterfaceSecurityRules": [
                          {
                            "Name": "default-allow-rdp",
                            "Etag": "W/\"d4c411d4-0d62-49dc-8092-3d4b57825740\"",
                            "Id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg2/providers/Microsoft.Network/networkSecurityGroups/testvm2-nsg/securityRules/default-allow-rdp",
                            "Protocol": "TCP",
                            "SourcePortRange": "*",
                            "DestinationPortRange": "3389",
                            "SourceAddressPrefix": "*",
                            "DestinationAddressPrefix": "*",
                            "Access": "Allow",
                            "Priority": 1000,
                            "Direction": "Inbound",
                            "ProvisioningState": "Succeeded"
                          }
                          ...
                        ],
                        "DefaultSecurityRules": [
                          {
                            "Name": "AllowVnetInBound",
                            "Id": "/subscriptions00000000-0000-0000-0000-000000000000/resourceGroups/testrg2/providers/Microsoft.Network/networkSecurityGroups/testvm2-nsg/defaultSecurityRules/",
                            "Description": "Allow inbound traffic from all VMs in VNET",
                            "Protocol": "*",
                            "SourcePortRange": "*",
                            "DestinationPortRange": "*",
                            "SourceAddressPrefix": "VirtualNetwork",
                            "DestinationAddressPrefix": "VirtualNetwork",
                            "Access": "Allow",
                            "Priority": 65000,
                            "Direction": "Inbound",
                            "ProvisioningState": "Succeeded"
                          }
                          ...
                        ],
                        "EffectiveSecurityRules": [
                          {
                            "Name": "DefaultOutboundDenyAll",
                            "Protocol": "All",
                            "SourcePortRange": "0-65535",
                            "DestinationPortRange": "0-65535",
                            "SourceAddressPrefix": "*",
                            "DestinationAddressPrefix": "*",
                            "ExpandedSourceAddressPrefix": [],
                            "ExpandedDestinationAddressPrefix": [],
                            "Access": "Deny",
                            "Priority": 65500,
                            "Direction": "Outbound"
                          },
                          ...
                        ]
                      }
                    ]
```

## <a name="next-steps"></a>Passaggi successivi

Consultare [Auditing Network Security Groups (NSG) with Network Watcher](network-watcher-nsg-auditing-powershell.md) (Verifica dei gruppi di sicurezza di rete con Network Watcher) per informazioni su come automatizzare la verifica dei gruppi di sicurezza di rete.


