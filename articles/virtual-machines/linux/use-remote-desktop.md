---
title: Usare Desktop remoto per una macchina virtuale Linux di Azure| Documentazione Microsoft
description: Informazioni sull'installazione e la configurazione di Desktop remoto (xrdp) per collegarsi a una macchina virtuale Linux di Azure usando strumenti grafici
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 09/12/2019
ms.author: cynthn
ms.openlocfilehash: 96f1f98f95bb726864553c81245e250cf907fb05
ms.sourcegitcommit: dd69b3cda2d722b7aecce5b9bd3eb9b7fbf9dc0a
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/12/2019
ms.locfileid: "70961551"
---
# <a name="install-and-configure-remote-desktop-to-connect-to-a-linux-vm-in-azure"></a>Installare e configurare Desktop remoto per connettersi a una VM Linux di Azure
Le macchine virtuali Linux (VM) di Azure in genere vengono gestite dalla riga di comando tramite una connessione secure shell (SSH). Quando si è nuovi a Linux, o per scenari di risoluzione dei problemi rapidi, l'uso di desktop remoto potrebbe risultare più facile. Questo articolo illustra come installare e configurare un ambiente desktop ([xfce](https://www.xfce.org)) e desktop remoto ([xrdp](https://www.xrdp.org)) per VM Linux usando il modello di distribuzione Resource Manager.


## <a name="prerequisites"></a>Prerequisiti
Questo articolo richiede una VM Ubuntu 18,04 LTS esistente in Azure. Se è necessario creare una macchina virtuale, usare uno dei metodi seguenti:

- L'[interfaccia della riga di comando di Azure](quick-create-cli.md)
- Il[portale di Azure](quick-create-portal.md)


## <a name="install-a-desktop-environment-on-your-linux-vm"></a>Installare un ambiente desktop nella VM Linux
La maggior parte delle macchine virtuali Linux di Azure non presenta un ambiente desktop installato per impostazione predefinita. Le macchine virtuali Linux in genere vengono gestite usando connessioni SSH piuttosto che un ambiente desktop. In Linux esistono diversi ambienti desktop tra i quali è possibile scegliere. A seconda dell'ambiente desktop scelto, questo può consumare da 1 a 2 GB di spazio su disco e può impiegare da 5 a 10 minuti per installare e configurare tutti i pacchetti necessari.

Nell'esempio seguente viene installato l'ambiente desktop Lightweight [xfce4](https://www.xfce.org/) in una VM Ubuntu 18,04 LTS. I comandi per le altre distribuzioni sono leggermente diversi (ad esempio, usare `yum` per installare in Red Hat Enterprise Linux e configurare regole `selinux` appropriate, oppure usare `zypper` per installare in SUSE).

Innanzitutto, stabilire una connessione SSH alla VM. Nell'esempio seguente viene eseguita la connessione alla macchina virtuale denominata *myvm.westus.cloudapp.azure.com* con il nome utente *azureuser*. Usare valori personalizzati:

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Se si usa Windows, per altre informazioni sull'uso di SSH, vedere [Come usare SSH con Windows in Azure](ssh-from-windows.md).

Successivamente, installare xfce usando `apt` come indicato di seguito:

```bash
sudo apt-get update
sudo apt-get install xfce4
```

## <a name="install-and-configure-a-remote-desktop-server"></a>Installare e configurare un server di desktop remoto
Ora che si dispone di un ambiente desktop installato, configurare un servizio Desktop remoto per l'ascolto delle connessioni in ingresso. [xrdp](http://xrdp.org) è un server Remote Desktop Protocol (RDP) open source che è disponibile nella maggior parte delle distribuzioni Linux e funziona bene con xfce. Installare xrdp nella VM Ubuntu come indicato di seguito:

```bash
sudo apt-get -y install xrdp
sudo systemctl enable xrdp
```

Indicare a xrdp quale ambiente desktop usare quando si avvia la sessione. Configurare xrdp per usare xfce come ambiente desktop come indicato di seguito:

```bash
echo xfce4-session >~/.xsession
```

Riavviare il servizio xrdp per rendere effettive le modifiche, come indicato di seguito:

```bash
sudo service xrdp restart
```


## <a name="set-a-local-user-account-password"></a>Impostare una password per l'account utente locale
Se la password dell'account utente è stata impostata al momento della creazione della macchina virtuale, ignorare questo passaggio. Se si usa soltanto l'autenticazione con chiave SSH e non è stata creata una password per l'account locale, specificare una password prima di usare xrdp per accedere alla macchina virtuale. xrdp non può accettare chiavi SSH per l'autenticazione. Nell'esempio seguente viene specificata una password per l'account utente *azureuser*:

```bash
sudo passwd azureuser
```

> [!NOTE]
> Se attualmente gli accessi tramite password non sono permessi, l'impostazione della password non aggiorna la configurazione SSHD per consentirli. Dal punto di vista della sicurezza, l'utente potrebbe desiderare connettersi alla macchina virtuale con un tunnel SSH usando l'autenticazione tramite chiave e poi connettersi a xrdp. In questo caso, ignorare il passaggio seguente sulla creazione di una regola del gruppo di sicurezza di rete per consentire il traffico di desktop remoto.


## <a name="create-a-network-security-group-rule-for-remote-desktop-traffic"></a>Creare una regola del gruppo di sicurezza di rete per il traffico di Desktop remoto
Per consentire al traffico di Desktop remoto di raggiungere la VM Linux, è necessario creare una regola del gruppo di sicurezza di rete che consenta al TCP sulla porta 3389 di raggiungere la macchina virtuale. Per altre informazioni sulle regole dei gruppi di sicurezza di rete, vedere [Definizione di gruppo di sicurezza di rete](../../virtual-network/security-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). È anche possibile [usare il portale di Azure per creare una regola del gruppo di sicurezza di rete](../windows/nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

L'esempio seguente crea una regola del gruppo di sicurezza di rete con il comando [az vm open-port](/cli/azure/vm#az-vm-open-port) sulla porta *3389*. Dall'interfaccia della riga di comando di Azure, non dalla sessione SSH alla macchina virtuale, aprire la regola di gruppo di sicurezza di rete seguente:

```azurecli
az vm open-port --resource-group myResourceGroup --name myVM --port 3389
```


## <a name="connect-your-linux-vm-with-a-remote-desktop-client"></a>Connettere la macchina virtuale Linux con un client di Desktop remoto
Aprire il client di Desktop remoto locale e connettersi all'indirizzo IP o nome DNS della VM Linux. Immettere il nome utente e la password per l'account utente nella macchina virtuale come indicato di seguito:

![Connettersi a xrdp usando il client di Desktop remoto](./media/use-remote-desktop/remote-desktop-client.png)

Dopo l'autenticazione, l'ambiente desktop xfce verrà caricato e apparirà come nell'esempio seguente:

![ambiente desktop xfce tramite xrdp](./media/use-remote-desktop/xfce-desktop-environment.png)

Se il client RDP locale usa l'autenticazione di livello di rete (NLA), potrebbe essere necessario disattivare questa impostazione di connessione. XRDP attualmente non supporta NLA. È anche possibile considerare soluzione RDP alternative che supportano NLA, come [FreeRDP](https://www.freerdp.com).


## <a name="troubleshoot"></a>Risolvere problemi
Se non è possibile connettersi alla macchina virtuale Linux usando un client di Desktop remoto, usare `netstat` nella macchina virtuale Linux per verificare che la macchina stia ascoltando le connessioni RDP come indicato di seguito:

```bash
sudo netstat -plnt | grep rdp
```

Nell'esempio seguente viene mostrata la macchina virtuale in ascolto sulla porta TCP 3389 come previsto:

```bash
tcp     0     0      127.0.0.1:3350     0.0.0.0:*     LISTEN     53192/xrdp-sesman
tcp     0     0      0.0.0.0:3389       0.0.0.0:*     LISTEN     53188/xrdp
```

Se il servizio *xrdp-sesman* non è in ascolto, riavviare il servizio in una macchina virtuale Ubuntu come indicato di seguito:

```bash
sudo service xrdp restart
```

Controllare i log in */var/log* nella macchina virtuale Ubuntu per indicazioni sul perché il servizio non risponde. È possibile anche monitorare il syslog durante un tentativo di connessione Desktop remoto per visualizzare eventuali errori:

```bash
tail -f /var/log/syslog
```

Altre distribuzioni Linux, ad esempio Red Hat Enterprise Linux e SUSE, possono presentare modi diversi per riavviare i servizi e posizioni dei file di log alternative da controllare.

Se non si riceve alcuna risposta nel client di Desktop remoto e non viene visualizzato nessun evento nel log di sistema, questo comportamento indica che il traffico di Desktop remoto non riesce a raggiungere la macchina virtuale. Controllare le regole del gruppo di sicurezza di rete per assicurarsi che esista una regola che consenta TCP sulla porta 3389. Per altre informazioni, vedere [Risolvere i problemi di connettività delle applicazioni in una macchina virtuale di Azure per Linux](../windows/troubleshoot-app-connection.md).


## <a name="next-steps"></a>Passaggi successivi
Per altre informazioni sulla creazione e l'uso di chiavi SSH con macchine virtuali Linux, vedere [Creare una coppia di chiavi SSH pubblica e privata per le macchine virtuali di Linux](mac-create-ssh-keys.md).

Per informazioni sull'uso di SSH da Windows, vedere [Come usare SSH con Windows in Azure](ssh-from-windows.md).

