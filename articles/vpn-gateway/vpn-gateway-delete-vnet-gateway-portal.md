---
title: 'Eliminare un gateway di rete virtuale: Portale di Azure: Azure Resource Manager | Microsoft Docs'
description: Eliminare un gateway di rete virtuale usando il portale di Azure nel modello di distribuzione Resource Manager.
services: vpn-gateway
documentationcenter: na
author: cherylmc
ms.service: vpn-gateway
ms.date: 10/23/2018
ms.author: cherylmc
ms.topic: conceptual
ms.openlocfilehash: 387b4e982772f22453876e1ea8b9e7c4039601c4
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60845691"
---
# <a name="delete-a-virtual-network-gateway-using-the-portal"></a>Eliminare un gateway di rete virtuale usando il portale

> [!div class="op_single_selector"]
> * [Portale di Azure](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (classic)](vpn-gateway-delete-vnet-gateway-classic-powershell.md) (PowerShell (classico))

Questo articolo contiene le istruzioni per eliminare un gateway VPN di Azure distribuito usando il modello di distribuzione Resource Manager. Esistono due diversi approcci quando si desidera eliminare un gateway di rete virtuale per una configurazione di gateway VPN.

- Se si vuole eliminare tutto e ricominciare da capo, come nel caso di un ambiente di testing, è possibile eliminare il gruppo di risorse. Quando si elimina un gruppo di risorse, vengono eliminate tutte le risorse all'interno del gruppo. Questo metodo è consigliato solo se non si vuole mantenere alcuna risorsa del gruppo di risorse. Con questo approccio non è possibile eliminare in modo selettivo solo alcune risorse.

- Se si desidera mantenere alcune delle risorse nel gruppo di risorse, eliminare un gateway di rete virtuale diventa leggermente più complicato. Per poter eliminare il gateway di rete virtuale prima è necessario eliminare tutte le risorse che dipendono dal gateway. I passaggi variano a seconda del tipo di connessioni che sono state create e delle risorse dipendenti per ogni connessione.

> [!IMPORTANT]
> Le istruzioni seguenti descrivono come eliminare gateway VPN di Azure distribuiti usando il modello di distribuzione Resource Manager. Per eliminare un gateway VPN distribuito usando il modello di distribuzione classica, usare Azure PowerShell come descritto [qui](vpn-gateway-delete-vnet-gateway-classic-powershell.md).


## <a name="delete-a-vpn-gateway"></a>Eliminare un gateway VPN

Per eliminare un gateway di rete virtuale, è necessario innanzitutto eliminare ogni risorsa relativa al gateway di rete virtuale. Le risorse devono essere eliminate in un determinato ordine a causa delle dipendenze.

[!INCLUDE [delete gateway](../../includes/vpn-gateway-delete-vnet-gateway-portal-include.md)]

A questo punto, viene eliminato il gateway di rete virtuale. I passaggi successivi consentono di eliminare le risorse che non vengono più usate.

### <a name="to-delete-the-local-network-gateway"></a>Per eliminare il gateway di rete locale

1. In **Tutte le risorse**, individuare i gateway di rete locale associati a ogni connessione.
2. Nel pannello **Informazioni generali** del gateway di rete locale, fare clic su **Elimina**.

### <a name="to-delete-the-public-ip-address-resource-for-the-gateway"></a>Per eliminare la risorsa dell'indirizzo IP pubblico del gateway

1. In **Tutte le risorse**, individuare la risorsa dell'indirizzo IP pubblico associata al gateway. Se il gateway di rete virtuale è attivo-attivo, compaiono due indirizzi IP pubblici. 
2. Nella pagina **Panoramica** dell'indirizzo IP pubblico fare clic su **Elimina** quindi su **Sì** per confermare.

### <a name="to-delete-the-gateway-subnet"></a>Per eliminare la subnet del gateway

1. In **Tutte le risorse**, individuare la rete virtuale. 
2. Nel pannello **Subnet**, fare clic su **GatewaySubnet** quindi fare clic su **Elimina**. 
3. Fare clic su **Sì** per confermare l'eliminazione della subnet del gateway.

## <a name="deleterg"></a>Eliminare un gateway VPN eliminando il gruppo di risorse

Se non si è interessati a mantenere risorse del gruppo di risorse e si vuole solo ricominciare da capo, è possibile eliminare un intero gruppo di risorse. Questo è un modo rapido per rimuovere tutto. La procedura seguente si applica solo al modello di distribuzione di Azure Resource Manager.

1. In **Tutte le risorse**, individuare il gruppo di risorse e fare clic per aprire il pannello.
2. Fare clic su **Elimina**. Nel pannello Elimina visualizzare le risorse interessate. Assicurarsi che si desidera eliminare tutte le risorse. In caso contrario, usare i passaggi in Eliminare un gateway VPN all'inizio di questo articolo.
3. Per continuare, digitare il nome del gruppo di risorse che si desidera eliminare, quindi fare clic su **Elimina**.
