---
title: Requisiti di sistema dell'array virtuale Microsoft Azure StorSimple | Microsoft Docs
description: Scoprire il software e i requisiti di rete per StorSimple Virtual Array
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ea1d3bca-e71b-453d-aa82-440d2638f5e3
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: 65d2a21a9f40470cee1dd9d713f9f9cb5431a245
ms.sourcegitcommit: f5cc71cbb9969c681a991aa4a39f1120571a6c2e
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/26/2019
ms.locfileid: "68516696"
---
# <a name="storsimple-virtual-array-system-requirements"></a>Requisiti di sistema StorSimple Virtual Array

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

## <a name="overview"></a>Panoramica

Questo articolo descrive i requisiti di sistema importanti per l'array virtuale Microsoft Azure StorSimple e per i client di archiviazione che accedono all'array. Prima di distribuire il sistema StorSimple è consigliabile leggere attentamente queste informazioni e quindi farvi riferimento, se necessario, durante la distribuzione e il successivo funzionamento.

I requisiti di sistema includono:

* **Requisiti software per i client di archiviazione** : descrive le piattaforme di virtualizzazione supportate, Web browser, iniziatori iSCSI, client SMB, requisiti minimi del dispositivo virtuale ed eventuali requisiti aggiuntivi per questi sistemi operativi.
* **Requisiti di rete per il dispositivo StorSimple** : fornisce informazioni sulle porte che devono essere aperte nel firewall per consentire il traffico iSCSI, cloud o di gestione.

Le informazioni sui requisiti di sistema StorSimple pubblicate in questo articolo si applicano solo a StorSimple Virtual Array.

* Per i dispositivi serie 8000, andare all'articolo sui [requisiti di sistema per il dispositivo StorSimple serie 8000](storsimple-system-requirements.md).
* Per i dispositivi serie 7000, andare all'articolo sui [requisiti di sistema per il dispositivo StorSimple serie 5000-7000](http://onlinehelp.storsimple.com/1_StorSimple_System_Requirements).

## <a name="software-requirements"></a>Requisiti software
I requisiti software includono le informazioni su Web browser supportati, versioni SMB, piattaforme di virtualizzazione e i requisiti minimi del dispositivo virtuale.

### <a name="supported-virtualization-platforms"></a>Piattaforme di virtualizzazione supportate
| **Hypervisor** | **Versione** |
| --- | --- |
| Hyper-V |Windows Server 2008 R2 SP1 e versioni successive |
| VMware ESXi |5.0, 5.5, 6.0 e 6.5. |

> [!IMPORTANT]
> Non installare gli strumenti VMware nell'array virtuale StorSimple. In caso contrario, il risultato sarà una configurazione non supportata.

### <a name="virtual-device-requirements"></a>Requisiti del dispositivo virtuale
| **Componente** | **Requisito** |
| --- | --- |
| Numero minimo di processori virtuali (memorie centrali) |4 |
| Memoria minima (RAM) |8 GB <br> Per un file server, 8 GB per meno di 2 milioni di file e 16 GB per 2 - 4 milioni di file|
| Spazio su disco<sup>1</sup> |Disco sistema operativo: 80 GB <br></br>Disco dati: da 500 GB a 8 TB |
| Numero minimo di interfaccia o interfacce di rete |1 |
| Larghezza di banda Internet <sup>2</sup> |Larghezza di banda minima necessaria: 5 Mbps <br> Larghezza di banda consigliata: 100 Mbps <br> La velocità del trasferimento di dati si adatta in base alla larghezza di banda di Internet. Ad esempio, per trasferire 100 GB di dati a 5 Mbps sono necessari 2 giorni. Questo potrebbe causare degli errori di backup perché i backup giornalieri non vengono completati in un giorno. Con una larghezza di banda di 100 Mbps si possono trasferire 100 GB di dati in 2,5 ore.   |

<sup>1</sup> : thin provisioning

<sup>2</sup>: i requisiti di rete possono variare a seconda della frequenza di modifica dei dati giornaliera. Ad esempio, se un dispositivo deve eseguire il backup di 10 GB o di più modifiche durante la giornata, il backup giornaliero con una connessione da 5 Mbps potrebbe richiedere fino a 4,25 ore (se i dati non possono essere compressi o deduplicati).

### <a name="supported-web-browsers"></a>Web browser supportati
| **Componente** | **Versione** | **Requisiti aggiuntivi/note** |
| --- | --- | --- |
| Microsoft Edge |Versione più recente | |
| Internet Explorer |Versione più recente |Testato con Internet Explorer 11 |
| Google Chrome |Versione più recente |Testato con Chrome 46 |

### <a name="supported-storage-clients"></a>Client di archiviazione supportati
I seguenti requisiti software si riferiscono agli iniziatori iSCSI che accedono all'array virtuale StorSimple (configurato come server iSCSI).

| **Sistemi operativi supportati** | **Versione richiesta** | **Requisiti aggiuntivi/note** |
| --- | --- | --- |
| Windows Server |2008 R2 SP1, 2012, 2012 R2 |StorSimple consente di creare volumi di thin provisioning o di provisioning completo. Non è possibile creare volumi con provisioning parziale. I volumi iSCSI StorSimple sono supportati solo nei seguenti casi: <ul><li>Volumi semplici su dischi di base Windows.</li><li>NTFS Windows per la formattazione di un volume.</li> |

I seguenti requisiti software si riferiscono ai client SMB che accedono all'array virtuale StorSimple (configurato come file server).

| **Versione SMB** |
| --- |
| SMB 2.x |
| SMB 3.0 |
| SMB 3.02 |

> [!IMPORTANT]
> Non copiare o archiviare i file protetti da Windows Encrypting File System (EFS) in un file server StorSimple Virtual Array, perché la configurazione non sarà supportata.


### <a name="supported-storage-format"></a>Formati di archiviazione supportati
Solo l'archiviazione BLOB in blocchi di Azure è supportata. I BLOB di pagine non sono supportati. Altre informazioni [sui BLOB in blocchi e i BLOB di pagine](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).

## <a name="networking-requirements"></a>Requisiti di rete
La tabella seguente elenca le porte che devono essere aperte nel firewall per consentire il traffico iSCSI, SMB, cloud o di gestione. In questa tabella, *in* o *in ingresso* fa riferimento alla direzione da cui le richieste client in ingresso accedono al dispositivo. *Fuori* o *in uscita* fa riferimento alla direzione in cui il dispositivo StorSimple invia i dati all'esterno, oltre la distribuzione: ad esempio, in uscita verso Internet.

| **Numero porta<sup>1</sup>** | **In ingresso/In uscita** | **Ambito porta** | **Obbligatorio** | **Note** |
| --- | --- | --- | --- | --- |
| TCP 80 (HTTP) |Out |WAN |No |La porta in uscita viene usata per consentire all'accesso Internet di recuperare gli aggiornamenti. <br></br>Il proxy Web in uscita è configurabile dall'utente. |
| TCP 443 (HTTPS) |In uscita |WAN |Sì |La porta in uscita viene usata per accedere ai dati nel cloud. <br></br>Il proxy Web in uscita è configurabile dall'utente. |
| UDP 53 (DNS) |Out |Rete WAN |In alcuni casi; vedere le note. |Questa porta è obbligatoria solo se si usa un server DNS basato su Internet. <br></br> Nota: se si distribuisce un file server, si consiglia l'uso del server DNS locale. |
| UDP 123 (NTP) |Out |Rete WAN |In alcuni casi; vedere le note. |Questa porta è obbligatoria solo se si usa un server NTP basato su Internet.<br></br> Nota: se si distribuisce un file server, si consiglia di sincronizzare l'ora con i controller di dominio di Active Directory. |
| TCP 80 (HTTP) |In |LAN |Sì |Questa è la porta in ingresso per l'interfaccia utente locale nel dispositivo StorSimple per la gestione locale. <br></br> Nota: l'accesso all'interfaccia utente locale tramite HTTP esegue il reindirizzamento automatico a HTTPS. |
| TCP 443 (HTTPS) |In |LAN |Yes |Questa è la porta in ingresso per l'interfaccia utente locale nel dispositivo StorSimple per la gestione locale. |
| TCP 3260 (iSCSI) |In |LAN |No |Questa porta viene usata per accedere ai dati tramite iSCSI. |

<sup>1</sup> Nessuna porta in ingresso deve essere aperta sulla rete Internet pubblica.

> [!IMPORTANT]
> Verificare che il firewall non modifichi o decrittografi il traffico SSL tra il dispositivo StorSimple e Azure.
> 
> 

### <a name="url-patterns-for-firewall-rules"></a>Modelli URL per le regole del firewall
Gli amministratori di rete possono spesso configurare regole del firewall avanzate in base ai modelli URL in modo da filtrare il traffico in entrata e in uscita. L'array virtuale e il servizio Gestione dispositivi StorSimple dipendono da altre applicazioni Microsoft, ad esempio il bus di servizio di Azure, Controllo di accesso Active Directory di Azure, gli account di archiviazione e i server Microsoft Update. I modelli URL associati a queste applicazioni possono essere usati per configurare le regole del firewall. È importante comprendere che i modelli di URL associati alle suddette applicazioni possono variare. Questo a sua volta richiederà, da parte dell'amministratore di rete, il monitoraggio e l'aggiornamento delle regole del firewall per StorSimple a seconda delle esigenze. 

È consigliabile impostare le regole del firewall per il traffico in uscita, liberamente nella maggior parte dei casi, in base agli indirizzi IP StorSimple. Tuttavia, è possibile utilizzare le informazioni seguenti per impostare regole del firewall avanzate indispensabili per creare ambienti protetti.

> [!NOTE]
> 
> * Gli indirizzi IP di origine del dispositivo devono essere sempre impostati su tutte le interfacce di rete abilitate per il cloud. 
> * Gli indirizzi IP di destinazione devono essere impostati sugli [intervalli IP dei data center di Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653).
> 
> 

| Modello URL | Componente/funzionalità |
| --- | --- |
| `https://*.storsimple.windowsazure.com/*`<br>`https://*.accesscontrol.windows.net/*`<br>`https://*.servicebus.windows.net/*` <br>`https://login.windows.net`|Servizio Gestione dispositivi StorSimple<br>Servizio di controllo di accesso<br>Bus di servizio di Azure<br>Servizio di autenticazione|
| `http://*.backup.windowsazure.com` |Registrazione del dispositivo |
| `https://crl.microsoft.com/pki/*`<br>`https://www.microsoft.com/pki/*` |Revoca del certificato |
| `https://*.core.windows.net/*`<br>`https://*.data.microsoft.com`<br>`http://*.msftncsi.com` |Account di archiviazione di Azure e monitoraggio |
| `https://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`https://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`https://download.microsoft.com`<br>`http://wustat.windows.com`<br>`https://ntservicepack.microsoft.com` |Server di Microsoft Update<br> |
| `http://*.deploy.akamaitechnologies.com` |Rete CDN di Akamai |
| `https://*.partners.extranet.microsoft.com/*` |Pacchetto per il supporto |
| `https://*.data.microsoft.com` |Servizio Telemetria in Windows; vedere [Aggiornamento per la soddisfazione dei clienti e di telemetria diagnostica](https://support.microsoft.com/en-us/kb/3068708) |

## <a name="next-steps"></a>Passaggi successivi
* [Preparare il portale per distribuire StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md)
