---
title: File di inclusione
description: File di inclusione
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/10/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: cf384f29bcc82c80c94e1408f535c592b8b728cb
ms.sourcegitcommit: e0a1a9e4a5c92d57deb168580e8aa1306bd94723
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/11/2019
ms.locfileid: "72285750"
---
## <a name="rootcert"></a>Creare un certificato radice autofirmato

Usare il cmdlet New-SelfSignedCertificate per creare un certificato radice autofirmato. Per altre informazioni sui parametri, vedere [New-SelfSignedCertificate](https://technet.microsoft.com/itpro/powershell/windows/pkiclient/new-selfsignedcertificate).

1. Da un computer che esegue Windows 10 o Windows Server 2016 aprire una console di Windows PowerShell con privilegi elevati. Questi esempi non sono validi per Azure Cloud Shell "Prova". È necessario seguire questi esempi in locale.
2. Usare l'esempio seguente per creare il certificato radice autofirmato. L'esempio seguente crea un certificato radice autofirmato denominato "P2SRootCert" che viene installato automaticamente in "Certificati-Utente corrente\Personale\Certificati". È possibile visualizzare il certificato aprendo *certmgr.msc* o *Gestire i certificati utente*.

   ```powershell
   $cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
   -Subject "CN=P2SRootCert" -KeyExportPolicy Exportable `
   -HashAlgorithm sha256 -KeyLength 2048 `
   -CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign
   ```

## <a name="clientcert"></a>Generazione di un certificato client

Ogni computer client che si connette a una rete virtuale usando la soluzione Da punto a sito deve avere un certificato client installato. È possibile generare un certificato client da un certificato radice autofirmato, quindi esportare e installare il certificato client. Se il certificato client non è installato, l'autenticazione ha esito negativo. 

I passaggi seguenti illustrano come generare un certificato client da un certificato radice autofirmato. È possibile generare più certificati client dallo stesso certificato radice. Quando si generano certificati client con la procedura seguente, il certificato client viene installato automaticamente nel computer che è stato usato per generare il certificato. Se si vuole installare un certificato client in un altro computer client, è possibile esportare il certificato.

Gli esempi usano il cmdlet New-SelfSignedCertificate per generare un certificato client con scadenza in un anno. Per informazioni aggiuntive sui parametri, ad esempio l'impostazione di un valore di scadenza diverso per i certificati client, vedere [New SelfSignedCertificate](https://technet.microsoft.com/itpro/powershell/windows/pkiclient/new-selfsignedcertificate).

### <a name="example-1"></a>Esempio 1

Usare questo esempio se dopo aver creato il certificato radice autofirmato non si è chiusa la console di PowerShell. Questo esempio continua dalla sezione precedente e usa la variabile dichiarata "$cert". Se è stata chiusa la console di PowerShell dopo aver creato il certificato radice autofirmato o si stanno creando i certificati client aggiuntivi in una nuova sessione della console di PowerShell, attenersi alla procedura nell'[esempio 2](#ex2).

Modificare ed eseguire l'esempio per generare un certificato client. Se si esegue l'esempio seguente senza modificarlo, il risultato è un certificato client denominato "P2SChildCert".  Se si desidera assegnare un nome diverso al certificato figlio, modificare il valore CN. Non modificare il TextExtension quando si esegue questo esempio. Il certificato client generato viene installato automaticamente in "Certificati-Utente corrente\Personale\Certificati" nel computer in uso.

```powershell
New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
-Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
```

### <a name="ex2"></a>Esempio 2

Se si creano certificati client aggiuntivi o non si usa la stessa sessione di PowerShell utilizzata per creare il certificato radice autofirmato, usare la procedura seguente:

1. Identificare il certificato radice autofirmato che viene installato nel computer. Questo cmdlet restituisce un elenco dei certificati installati nel computer in uso.

   ```powershell
   Get-ChildItem -Path “Cert:\CurrentUser\My”
   ```
2. Individuare il nome dell'oggetto nell'elenco restituito e quindi copiare in un file di testo l'identificazione personale che si trova accanto a esso. Nell'esempio seguente ci sono due certificati. Il nome CN è il nome del certificato radice autofirmato da cui si desidera generare un certificato figlio. In questo caso "P2SRootCert".

   ```
   Thumbprint                                Subject
  
   AED812AD883826FF76B4D1D5A77B3C08EFA79F3F  CN=P2SChildCert4
   7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655  CN=P2SRootCert
   ```
3. Dichiarare una variabile per il certificato radice usando l'identificazione personale del passaggio precedente. Sostituire THUMBPRINT con l'identificazione personale del certificato radice autofirmato da cui si desidera generare un certificato figlio.

   ```powershell
   $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\THUMBPRINT"
   ```

   Ad esempio, usando l'identificazione personale per P2SRootCert nel passaggio precedente, la variabile è simile alla seguente:

   ```powershell
   $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655"
   ```
4. Modificare ed eseguire l'esempio per generare un certificato client. Se si esegue l'esempio seguente senza modificarlo, il risultato è un certificato client denominato "P2SChildCert". Se si desidera assegnare un nome diverso al certificato figlio, modificare il valore CN. Non modificare il TextExtension quando si esegue questo esempio. Il certificato client generato viene installato automaticamente in "Certificati-Utente corrente\Personale\Certificati" nel computer in uso.

   ```powershell
   New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
   -Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
   -HashAlgorithm sha256 -KeyLength 2048 `
   -CertStoreLocation "Cert:\CurrentUser\My" `
   -Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
   ```

## <a name="cer"></a>Esportare la chiave pubblica del certificato radice (. cer)

[!INCLUDE [Export public key](vpn-gateway-certificates-export-public-key-include.md)]

### <a name="export-the-self-signed-root-certificate-and-private-key-to-store-it-optional"></a>Esportare il certificato radice autofirmato e la chiave privata per archiviarli (facoltativo)

Si consiglia di esportare il certificato radice autofirmato e archiviarlo in un percorso sicuro per avere una copia di backup. Se necessario, in seguito è possibile installarlo su un altro computer e generare altri certificati client. Per esportare il certificato radice autofirmato come file .pfx, selezionare il certificato radice ed eseguire la stessa procedura descritta in [Esportazione di un certificato client](#clientexport).

## <a name="clientexport"></a>Esportare il certificato client

[!INCLUDE [Export client certificate](vpn-gateway-certificates-export-client-cert-include.md)]
