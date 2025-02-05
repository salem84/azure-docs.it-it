---
title: Hosting di zone DNS di ricerca inversa in DNS di Azure | Microsoft Docs
description: Informazioni su come usare DNS Azure per l'hosting delle zone DNS di ricerca inversa per gli intervalli di indirizzi IP
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: victorh
ms.openlocfilehash: cb2f04c692d4b5f385a89ba6a3071c20ef1bdf21
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "66143571"
---
# <a name="host-reverse-dns-lookup-zones-in-azure-dns"></a>Hosting di zone DNS di ricerca inversa in DNS di Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Questo articolo illustra come eseguire l'hosting delle zone DNS di ricerca inversa per gli intervalli di indirizzi IP assegnati in DNS Azure. Gli intervalli di indirizzi IP rappresentati dalle zone di ricerca inversa devono essere assegnati all'organizzazione, in genere dal provider di servizi Internet.

Per configurare il DNS inverso per un indirizzo IP di proprietà di Azure assegnato al servizio Azure, vedere [Configurare il DNS inverso per i servizi ospitati in Azure](dns-reverse-dns-for-azure-services.md).

Prima di leggere questo articolo, è consigliabile leggere la [panoramica del DNS inverso e del supporto in Azure](dns-reverse-dns-overview.md).

Questo articolo illustra la procedura per la creazione della prima zona DNS di ricerca inversa e del primo record tramite il portale di Azure, Azure PowerShell, l'interfaccia della riga di comando di Azure classica o l'interfaccia della riga di comando di Azure.

## <a name="create-a-reverse-lookup-dns-zone"></a>Creare una zona DNS di ricerca inversa

1. Accedere al [portale di Azure](https://portal.azure.com).
1. Nel menu **Hub** scegliere **Nuovo** > **Rete** e quindi selezionare **Zona DNS**.

   ![Selezione di "Zona DNS"](./media/dns-reverse-dns-hosting/figure1.png)

1. Nel riquadro **Crea zona DNS** assegnare un nome alla zona DNS. Il nome della zona viene configurato in modo diverso per i prefissi IPv4 e IPv6. Usare le istruzioni per [IPv4](#ipv4) o [IPv6](#ipv6) per specificare un nome per la zona. Al termine, selezionare **Crea** per creare la zona.

### <a name="ipv4"></a>IPv4

Il nome di una zona di ricerca inversa IPv4 è basato sull'intervallo da essa rappresentato. Deve avere il formato seguente: `<IPv4 network prefix in reverse order>.in-addr.arpa`. Per esempi, vedere [Panoramica del DNS inverso e supporto in Azure](dns-reverse-dns-overview.md#ipv4).

> [!NOTE]
> Quando si creano zone DNS di ricerca inversa senza classi in DNS di Azure, è necessario usare un trattino (`-`) invece di una barra (`/`) nel nome della zona.
>
> Ad esempio, per l'intervallo di indirizzi IP 192.0.2.128/26 è necessario usare `128-26.2.0.192.in-addr.arpa` come nome della zona invece di `128/26.2.0.192.in-addr.arpa`.
>
> Sebbene gli standard DNS supportino entrambi i metodi, DNS di Azure non supporta i nomi di zona DNS che contengono il carattere barra (`/`).

L'esempio seguente mostra come creare una zona del DNS inverso di tipo classe C con nome `2.0.192.in-addr.arpa` in DNS di Azure tramite il portale di Azure:

 ![Riquadro "Crea zona DNS" con caselle compilate](./media/dns-reverse-dns-hosting/figure2.png)

**Località del gruppo di risorse** definisce la località per il gruppo di risorse. Non incide sulla zona DNS. La località della zona DNS è sempre "globale" e non viene visualizzata.

Gli esempi seguenti mostrano come completare questa attività con Azure PowerShell e l'interfaccia della riga di comando di Azure.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsZone -Name 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Interfaccia della riga di comando classica di Azure

```azurecli
azure network dns zone create MyResourceGroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

```azurecli
az network dns zone create -g MyResourceGroup -n 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6

Il nome di una zona di ricerca inversa di tipo IPv6 deve avere il formato seguente: `<IPv6 network prefix in reverse order>.ip6.arpa`.  Per esempi, vedere [Panoramica del DNS inverso e supporto in Azure](dns-reverse-dns-overview.md#ipv6).


L'esempio seguente mostra come creare una zona DNS di ricerca inversa di tipo IPv6 con nome `0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa` in DNS Azure tramite il portale di Azure:

 ![Riquadro "Crea zona DNS" con caselle compilate](./media/dns-reverse-dns-hosting/figure3.png)

**Località del gruppo di risorse** definisce la località per il gruppo di risorse. Non incide sulla zona DNS. La località della zona DNS è sempre "globale" e non viene visualizzata.

Gli esempi seguenti mostrano come completare questa attività con Azure PowerShell e l'interfaccia della riga di comando di Azure.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsZone -Name 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Interfaccia della riga di comando classica di Azure

```azurecli
azure network dns zone create MyResourceGroup 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

```azurecli
az network dns zone create -g MyResourceGroup -n 0.0.0.0.d.c.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="delegate-a-reverse-dns-lookup-zone"></a>Delegare una zona DNS di ricerca inversa

Dopo avere creato la zona DNS di ricerca inversa, è necessario assicurarsi che la zona sia delegata dalla zona padre. La delega DNS consente al processo di risoluzione del nome DNS di trovare i server dei nomi che ospitano la zona DNS di ricerca inversa. Tali server dei nomi possono poi rispondere alle query del DNS inverso per gli indirizzi IP disponibili nell'intervallo di indirizzi.

Per le zone di ricerca diretta il processo di delega di una zona DNS è illustrato in [Delegare un dominio a DNS Azure](dns-delegate-domain-azure-dns.md). La delega per le zone di ricerca inversa funziona nello stesso modo. L'unica differenza è costituita dal fatto che è necessario configurare i server dei nomi con il provider di servizi Internet che ha fornito l'intervallo di indirizzi IP, invece del registrar.

## <a name="create-a-dns-ptr-record"></a>Creare un record PTR DNS

### <a name="ipv4"></a>IPv4

L'esempio seguente illustra la procedura di creazione di un record PTR in una zona del DNS inverso in DNS Azure. Per altri tipi di record e per modificare i record esistenti, vedere [Gestire record e set di record DNS con il portale di Azure](dns-operations-recordsets-portal.md).

1. Nella parte superiore della pagina **Zona DNS** selezionare **+ Set di record** per aprire il riquadro **Aggiungi set di record**.

   ![Pulsante per la creazione di un set di record](./media/dns-reverse-dns-hosting/figure4.png)

1. Il nome del set di record per un record PTR deve corrispondere al resto dell'indirizzo IPv4 in ordine inverso. 

   In questo esempio i primi tre ottetti sono già popolati come parte del nome della zona (.2.0.192). Solo l'ultimo ottetto viene quindi fornito nella casella **Nome**. Ad esempio, è possibile assegnare il nome **15** al set di record per una risorsa con indirizzo IP 192.0.2.15.  
1. In **Tipo** selezionare **PTR**.  
1. In **Nome dominio** immettere il nome di dominio completo della risorsa che usa l'indirizzo IP.
1. Selezionare **OK** nella parte inferiore del riquadro per creare il record DNS.

   ![Riquadro "Aggiungi set di record" con caselle compilate](./media/dns-reverse-dns-hosting/figure5.png)

Gli esempi seguenti mostrano come completare questa attività con PowerShell o l'interfaccia della riga di comando di Azure.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsRecordSet -Name 15 -RecordType PTR -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "dc1.contoso.com")
```
#### <a name="azure-classic-cli"></a>Interfaccia della riga di comando classica di Azure

```azurecli
azure network dns record-set add-record MyResourceGroup 2.0.192.in-addr.arpa 15 PTR --ptrdname dc1.contoso.com  
```

#### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

```azurecli
    az network dns record-set ptr add-record -g MyResourceGroup -z 2.0.192.in-addr.arpa -n 15 --ptrdname dc1.contoso.com
```

### <a name="ipv6"></a>IPv6

L'esempio seguente fornisce indicazioni dettagliate sul processo di creazione di un nuovo record PTR. Per altri tipi di record e per modificare i record esistenti, vedere [Gestire record e set di record DNS con il portale di Azure](dns-operations-recordsets-portal.md).

1. Nella parte superiore della pagina **Zona DNS** selezionare **+ Set di record** per aprire il riquadro **Aggiungi set di record**.

   ![Pulsante per la creazione di un set di record](./media/dns-reverse-dns-hosting/figure6.png)

2. Il nome del set di record per un record PTR deve corrispondere al resto dell'indirizzo IPv6 in ordine inverso. Non deve includere alcuna compressione degli zeri. 

   In questo esempio i primi 64 bit dell'indirizzo IPv6 sono già popolati come parte del nome della zona (0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa). Solo gli ultimi 64 bit vengono quindi forniti nella casella **Nome**. Gli ultimi 64 bit dell'indirizzo IP vengono immessi in ordine inverso, con un punto come delimitatore tra ogni numero esadecimale. Ad esempio, è possibile specificare **e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f** come nome di un set di record per una risorsa con indirizzo IP 2001:0db8:abdc:0000:f524:10bc:1af9:405e.  
3. In **Tipo** selezionare **PTR**.  
4. In **Nome dominio** immettere il nome di dominio completo della risorsa che usa l'indirizzo IP.
5. Selezionare **OK** nella parte inferiore del riquadro per creare il record DNS.

![Riquadro "Aggiungi set di record" con caselle compilate](./media/dns-reverse-dns-hosting/figure7.png)

Gli esempi seguenti mostrano come completare questa attività con PowerShell o l'interfaccia della riga di comando di Azure.

#### <a name="powershell"></a>PowerShell

```powershell
New-AzDnsRecordSet -Name "e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f" -RecordType PTR -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "dc2.contoso.com")
```

#### <a name="azure-classic-cli"></a>Interfaccia della riga di comando classica di Azure

```
azure network dns record-set add-record MyResourceGroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f PTR --ptrdname dc2.contoso.com 
```
 
#### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

```azurecli
    az network dns record-set ptr add-record -g MyResourceGroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -n e.5.0.4.9.f.a.1.c.b.0.1.4.2.5.f --ptrdname dc2.contoso.com
```

## <a name="view-records"></a>Visualizzare i record

Per visualizzare i record creati, passare alla zona DNS nel portale di Azure. Nella parte inferiore del riquadro **Zona DNS** è possibile visualizzare i record per la zona DNS. Verranno visualizzati i record predefiniti NS e SOA, oltre agli eventuali nuovi record creati. I record NS e SOA vengono creati in ogni zona. 

### <a name="ipv4"></a>IPv4

Il riquadro **Zona DNS** mostra i record PTR IPv4:

![Riquadro "Zona DNS" con i record IPv4](./media/dns-reverse-dns-hosting/figure8.png)

Gli esempi seguenti mostrano come visualizzare i record PTR usando PowerShell o l'interfaccia della riga di comando di Azure.

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzDnsRecordSet -ZoneName 2.0.192.in-addr.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Interfaccia della riga di comando classica di Azure

```azurecli
    azure network dns record-set list MyResourceGroup 2.0.192.in-addr.arpa
```

#### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

```azurecli
    azure network dns record-set list -g MyResourceGroup -z 2.0.192.in-addr.arpa
```

### <a name="ipv6"></a>IPv6

Il riquadro **Zona DNS** mostra i record PTR IPv6:

![Riquadro "Zona DNS" con i record IPv6](./media/dns-reverse-dns-hosting/figure9.png)

Gli esempi seguenti mostrano come visualizzare i record usando PowerShell o l'interfaccia della riga di comando di Azure.

#### <a name="powershell"></a>PowerShell

```powershell
Get-AzDnsRecordSet -ZoneName 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa -ResourceGroupName MyResourceGroup
```

#### <a name="azure-classic-cli"></a>Interfaccia della riga di comando classica di Azure

```azurecli
    azure network dns record-set list MyResourceGroup 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

#### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

```azurecli
    azure network dns record-set list -g MyResourceGroup -z 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa
```

## <a name="faq"></a>Domande frequenti

### <a name="can-i-host-reverse-dns-lookup-zones-for-my-isp-assigned-ip-blocks-on-azure-dns"></a>È possibile ospitare zone DNS di ricerca inversa per i blocchi di indirizzi IP assegnati dal provider di servizi Internet in DNS Azure?

Sì. L'hosting delle zone di ricerca inversa (ARPA) per gli intervalli di indirizzi IP in DNS Azure DNS è completamento supportato.

Creare le zone di ricerca inversa in DNS di Azure come illustrato in questo articolo e quindi collaborare con il provider di servizi Internet per [delegare la zona](dns-domain-delegation.md). Sarà quindi possibile gestire i record PTR per ogni ricerca inversa come per gli altri tipi di record.

### <a name="how-much-does-hosting-my-reverse-dns-lookup-zone-cost"></a>Quanto costa l'hosting della zona DNS di ricerca inversa?

L'hosting della zona DNS di ricerca inversa per i blocchi di indirizzi IP assegnati dal provider di servizi Internet in DNS Azure viene addebitato in base alle [tariffe standard per DNS Azure](https://azure.microsoft.com/pricing/details/dns/).

### <a name="can-i-host-reverse-dns-lookup-zones-for-both-ipv4-and-ipv6-addresses-in-azure-dns"></a>È possibile ospitare zone DNS di ricerca inversa per indirizzi IPv4 e IPv6 in DNS Azure?

Sì. Questo articolo illustra come creare zone DNS di ricerca inversa di tipo IPv4 e IPv6 in DNS Azure.

### <a name="can-i-import-an-existing-reverse-dns-lookup-zone"></a>È possibile importare una zona DNS di ricerca inversa esistente?

Sì. È possibile usare l'interfaccia della riga di comando di Azure per importare zone DNS di ricerca inversa esistenti in DNS di Azure. Questo metodo è applicabile alle zone di ricerca diretta e alle zone di ricerca inversa.

Per altre informazioni, vedere [Importare ed esportare un file di zona DNS usando l'interfaccia della riga di comando di Azure](dns-import-export.md).

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sul DNS inverso, vedere [Risoluzione DNS inversa su Wikipedia](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).
<br>
Informazioni su come [gestire i record di DNS inverso per i servizi di Azure](dns-reverse-dns-for-azure-services.md).
