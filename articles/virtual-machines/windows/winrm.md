---
title: Configurare l'accesso WinRM per una macchina virtuale di Azure | Documentazione Microsoft
description: Configurare l'accesso WinRM per l'uso con una macchina virtuale di Azure creata con il modello di distribuzione Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: singhkays
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 9718e85b-d360-4621-90b8-0b0b84a21208
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 06/16/2016
ms.author: kasing
ms.openlocfilehash: f7f57a43697a9376062bdd3baa2d5f7333bf4a7f
ms.sourcegitcommit: 44e85b95baf7dfb9e92fb38f03c2a1bc31765415
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70100147"
---
# <a name="setting-up-winrm-access-for-virtual-machines-in-azure-resource-manager"></a>Configurare l'accesso WinRM per le macchine virtuali in Azure Resource Manager

Di seguito è descritta la procedura per configurare una VM con connettività WinRM

1. Creare un insieme di credenziali delle chiavi
2. Creare un certificato autofirmato
3. Caricare il certificato autofirmato per l'insieme di credenziali delle chiavi
4. Ottenere l'URL del certificato autofirmato nell'insieme di credenziali delle chiavi
5. Fare riferimento all'URL del certificato autofirmato durante la creazione di una VM

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="step-1-create-a-key-vault"></a>Passaggio 1: Creare un insieme di credenziali delle chiavi
Il seguente comando consente di creare l'insieme di credenziali delle chiavi

```
New-AzKeyVault -VaultName "<vault-name>" -ResourceGroupName "<rg-name>" -Location "<vault-location>" -EnabledForDeployment -EnabledForTemplateDeployment
```

## <a name="step-2-create-a-self-signed-certificate"></a>Passaggio 2: Creare un certificato autofirmato
Per creare un certificato autofirmato, è possibile utilizzare questo script PowerShell

```
$certificateName = "somename"

$thumbprint = (New-SelfSignedCertificate -DnsName $certificateName -CertStoreLocation Cert:\CurrentUser\My -KeySpec KeyExchange).Thumbprint

$cert = (Get-ChildItem -Path cert:\CurrentUser\My\$thumbprint)

$password = Read-Host -Prompt "Please enter the certificate password." -AsSecureString

Export-PfxCertificate -Cert $cert -FilePath ".\$certificateName.pfx" -Password $password
```

## <a name="step-3-upload-your-self-signed-certificate-to-the-key-vault"></a>Passaggio 3: Caricare il certificato autofirmato per l'insieme di credenziali delle chiavi
Prima di caricare il certificato nell'insieme di credenziali delle chiavi creato al Passaggio 1, è necessario convertirlo in un formato comprensibile per il provider di risorse Microsoft.Compute. Per farlo, utilizzare lo script PowerShell qui di seguito

```
$fileName = "<Path to the .pfx file>"
$fileContentBytes = Get-Content $fileName -Encoding Byte
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)

$jsonObject = @"
{
  "data": "$filecontentencoded",
  "dataType" :"pfx",
  "password": "<password>"
}
"@

$jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
$jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

$secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText –Force
Set-AzKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>" -SecretValue $secret
```

## <a name="step-4-get-the-url-for-your-self-signed-certificate-in-the-key-vault"></a>Passaggio 4: Ottenere l'URL del certificato autofirmato nell'insieme di credenziali delle chiavi
Durante il provisioning della VM, il provider di risorse Microsoft.Compute necessita dell'URL della chiave privata all'interno dell'insieme di credenziali delle chiavi. Ciò consente al provider di risorse Microsoft.Compute di scaricare la chiave privata e creare il certificato equivalente nella VM.

> [!NOTE]
> L'URL della chiave privata deve includerne anche la versione. Un URL di esempio è simile al seguente\/https:/contosovault.Vault.Azure.NET:443/Secrets/contososecret/01h9db0df2cd4300a20ence585a6s7ve

#### <a name="templates"></a>Modelli
Per ottenere il collegamento all'URL nel modello, è possibile utilizzre il codice seguente

    "certificateUrl": "[reference(resourceId(resourceGroup().name, 'Microsoft.KeyVault/vaults/secrets', '<vault-name>', '<secret-name>'), '2015-06-01').secretUriWithVersion]"

#### <a name="powershell"></a>PowerShell
Questo URL può essere ottenuto utilizzando il seguente comando PowerShell

    $secretURL = (Get-AzKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id

## <a name="step-5-reference-your-self-signed-certificates-url-while-creating-a-vm"></a>Passaggio 5: Fare riferimento all'URL del certificato autofirmato durante la creazione di una VM
#### <a name="azure-resource-manager-templates"></a>Modelli di Azure Resource Manager
Quando si crea una VM tramite modelli, viene fatto riferimento al certificato nelle sezioni delle chiavi private e di WinRM, come indicato di seguito:

    "osProfile": {
          ...
          "secrets": [
            {
              "sourceVault": {
                "id": "<resource id of the Key Vault containing the secret>"
              },
              "vaultCertificates": [
                {
                  "certificateUrl": "<URL for the certificate you got in Step 4>",
                  "certificateStore": "<Name of the certificate store on the VM>"
                }
              ]
            }
          ],
          "windowsConfiguration": {
            ...
            "winRM": {
              "listeners": [
                {
                  "protocol": "http"
                },
                {
                  "protocol": "https",
                  "certificateUrl": "<URL for the certificate you got in Step 4>"
                }
              ]
            },
            ...
          }
        },

Un modello di esempio per quanto detto sopra è disponibile qui: [201-vm-winrm-keyvault-windows](https://azure.microsoft.com/documentation/templates/201-vm-winrm-keyvault-windows)

Il codice sorgente di questo modello è reperibile in [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows)

#### <a name="powershell"></a>PowerShell
    $vm = New-AzVMConfig -VMName "<VM name>" -VMSize "<VM Size>"
    $credential = Get-Credential
    $secretURL = (Get-AzKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id
    $vm = Set-AzVMOperatingSystem -VM $vm -Windows -ComputerName "<Computer Name>" -Credential $credential -WinRMHttp -WinRMHttps -WinRMCertificateUrl $secretURL
    $sourceVaultId = (Get-AzKeyVault -ResourceGroupName "<Resource Group name>" -VaultName "<Vault Name>").ResourceId
    $CertificateStore = "My"
    $vm = Add-AzVMSecret -VM $vm -SourceVaultId $sourceVaultId -CertificateStore $CertificateStore -CertificateUrl $secretURL

## <a name="step-6-connecting-to-the-vm"></a>Passaggio 6: Connessione alla macchina virtuale
Per potersi connettere alla VM è necessario controllare di aver configurato il computer per la gestione remota di WinRM. Avviare PowerShell come amministratore ed eseguire il comando seguente per verificare la configurazione.

    Enable-PSRemoting -Force

> [!NOTE]
> Se non si riesce, potrebbe essere necessario verificare che il servizio WinRM sia in esecuzione. Per farlo, utilizzare `Get-Service WinRM`
> 
> 

Dopo avere completato l'installazione, è possibile connettersi alla VM utilizzando il comando seguente

    Enter-PSSession -ConnectionUri https://<public-ip-dns-of-the-vm>:5986 -Credential $cred -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck) -Authentication Negotiate
