---
title: Connettersi a desktop virtuale Windows da macOS-Azure
description: Come connettersi al desktop virtuale di Windows tramite il client macOS.
services: virtual-desktop
author: heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: helohr
ms.openlocfilehash: fa61d69ae7aa650b6011696fd45d05ba20d2ac8b
ms.sourcegitcommit: e1b6a40a9c9341b33df384aa607ae359e4ab0f53
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/27/2019
ms.locfileid: "71338713"
---
# <a name="connect-with-the-macos-client"></a>Connettersi con il client macOS

> Si applica a: macOS 10,12 o versione successiva

>[!NOTE]
> La possibilità di accedere alle risorse di desktop virtuali Windows dal client macOS è attualmente disponibile in anteprima.

È possibile accedere alle risorse di desktop virtuali Windows dai dispositivi macOS con il client scaricabile. Questa guida illustra come configurare il client di.

## <a name="install-the-client"></a>Installazione del client

Per iniziare, [scaricare](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) e installare il client nel dispositivo MacOS.

## <a name="subscribe-to-a-feed"></a>Sottoscrivere un feed

Sottoscrivere il feed che l'amministratore ha fornito per ottenere l'elenco delle risorse gestite disponibili nel dispositivo macOS.

Per sottoscrivere un feed:

1. Selezionare **Aggiungi feed** nella pagina principale per connettersi al servizio e recuperare le risorse.
2. Immettere l'URL del feed. Può trattarsi di un URL o di un indirizzo di posta elettronica:
   - Se si usa un URL, usare quello che l'amministratore ha assegnato. In genere, l'URL <https://rdweb.wvd.microsoft.com>è.
   - Per usare la posta elettronica, immettere l'indirizzo di posta elettronica. Indica al client di cercare un URL associato all'indirizzo di posta elettronica se l'amministratore ha configurato il server in modo.
3. Selezionare **Sottoscrivi**.
4. Accedere con l'account utente quando richiesto.

Dopo aver eseguito l'accesso, verrà visualizzato un elenco di risorse disponibili.

Una volta effettuata la sottoscrizione a un feed, il contenuto del feed verrà aggiornato automaticamente regolarmente. Le risorse possono essere aggiunte, modificate o rimosse in base alle modifiche apportate dall'amministratore.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sul client macOS, vedere la documentazione [introduttiva per il client MacOS](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/remote-desktop-mac) .
