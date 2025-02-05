---
title: 'Requisiti QoS - ExpressRoute: Azure | Microsoft Docs'
description: Questa pagina fornisce informazioni dettagliate sui requisiti per la configurazione e la gestione di QoS. Vengono illustrati i servizi Skype for Business/vocali.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: cherylmc
ms.custom: seodec18
ms.openlocfilehash: eed6113442b4080341ff08b3983880f3afe66c00
ms.sourcegitcommit: 04ec7b5fa7a92a4eb72fca6c6cb617be35d30d0c
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/22/2019
ms.locfileid: "68385132"
---
# <a name="expressroute-qos-requirements"></a>Requisiti ExpressRoute QoS
Skype per aziende dispone di diversi carichi di lavoro che richiedono la gestione QoS differenziata. Se si prevede di utilizzare i servizi vocali tramite ExpressRoute, è necessario rispettare i requisiti descritti di seguito.

![](./media/expressroute-qos/expressroute-qos.png)

> [!NOTE]
> I requisiti QoS si applicano solo ai peer Microsoft. I valori DSCP del traffico di rete ricevuti nel peer pubblico e nel peer privato di Azure saranno reimpostati su 0. 
> 
> 

La tabella seguente fornisce un elenco di contrassegni DSCP usati da Microsoft teams e Skype for business. Fare riferimento a [Gestione QoS per Skype per aziende](https://docs.microsoft.com/SkypeForBusiness/manage/network-management/qos/managing-quality-of-service-QoS) per ulteriori informazioni.

| **Classe di traffico** | **Modalità di gestione (contrassegno DSCP)** | **Carichi di lavoro Microsoft teams e Skype for business** |
| --- | --- | --- |
| **Voice** |ENTITY FRAMEWORK (46) |Skype/Microsoft teams/Lync Voice |
| **Interattivo** |AF41 (34) |Video, VBSS |
| |AF21 (18) |Condivisione delle app | 
| **Default** |AF11 (10) |Trasferimento di file |
| |CS0 (0) |Altro |

* È necessario classificare i carichi di lavoro e contrassegnare i valori DSCP corretti. Seguire le indicazioni fornite [qui](https://docs.microsoft.com/SkypeForBusiness/manage/network-management/qos/configuring-port-ranges-for-your-skype-clients#configure-quality-of-service-policies-for-clients-running-on-windows-10) su come impostare i contrassegni DSCP nella rete.
* È necessario configurare e supportare più code di QoS nella rete. Voice deve essere una classe autonoma e ricevere il trattamento di Entity Framework specificato in [RFC 3246](https://www.ietf.org/rfc/rfc3246.txt). 
* È possibile decidere il meccanismo di accodamento, i criteri di rilevamento della congestione e l’allocazione della larghezza di banda per ogni classe di traffico. Tuttavia, è necessario mantenere il contrassegno DSCP per i carichi di lavoro di Skype per aziende. Se si utilizzano i contrassegni DSCP non elencati sopra, ad esempio AF31 (26), è necessario riscrivere il valore DSCP su 0 prima di inviare il pacchetto a Microsoft. Microsoft invia solo i pacchetti contrassegnati con il valore di DSCP illustrato nella tabella precedente. 

## <a name="next-steps"></a>Passaggi successivi
* Fare riferimento ai requisiti per [Routing](expressroute-routing.md) e [NAT](expressroute-nat.md).
* Vedere i collegamenti seguenti per configurare la connessione ExpressRoute.
  
  * [Creare un circuito ExpressRoute](expressroute-howto-circuit-classic.md)
  * [Configurare il routing](expressroute-howto-routing-classic.md)
  * [Collegare una rete virtuale a un circuito ExpressRoute](expressroute-howto-linkvnet-classic.md)

