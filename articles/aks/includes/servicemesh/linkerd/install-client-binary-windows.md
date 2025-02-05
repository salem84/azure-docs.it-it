---
author: paulbouwer
ms.service: container-service
ms.topic: include
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 2a1249d134c23f47f939913fa1c9887d2a8e1f63
ms.sourcegitcommit: b4f201a633775fee96c7e13e176946f6e0e5dd85
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/18/2019
ms.locfileid: "72600994"
---
## <a name="download-and-install-the-linkerd-linkerd-client-binary"></a>Scaricare e installare il file binario client Linkerd di Linkerd

In una shell basata su PowerShell in Windows, usare `Invoke-WebRequest` per scaricare la versione di Linkerd come indicato di seguito:

```powershell
# Specify the Linkerd version that will be leveraged throughout these instructions
$LINKERD_VERSION="stable-2.6.0"

# Enforce TLS 1.2
[Net.ServicePointManager]::SecurityProtocol = "tls12"
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -URI "https://github.com/linkerd/linkerd2/releases/download/$LINKERD_VERSION/linkerd2-cli-$LINKERD_VERSION-windows.exe" -OutFile "linkerd2-cli-$LINKERD_VERSION-windows.exe"
```

Il `linkerd` binario client viene eseguito nel computer client e consente di interagire con la mesh del servizio Linkerd. Usare i comandi seguenti per installare il file binario client di Linkerd `linkerd` in una shell basata su PowerShell in Windows. Questi comandi copiano il file binario client `linkerd` in una cartella Linkerd e quindi lo rendono disponibile sia immediatamente (nella shell corrente) che in modo permanente (tra i riavvii della Shell) tramite il `PATH`. Non sono necessari privilegi elevati (amministratore) per eseguire questi comandi e non è necessario riavviare la Shell.

```powershell
# Copy linkerd.exe to C:\Linkerd
New-Item -ItemType Directory -Force -Path "C:\Linkerd"
Copy-Item -Path ".\linkerd2-cli-$LINKERD_VERSION-windows.exe" -Destination "C:\Linkerd\linkerd.exe"

# Add C:\Linkerd to PATH. 
# Make the new PATH permanently available for the current User, and also immediately available in the current shell.
$PATH = [environment]::GetEnvironmentVariable("PATH", "User") + "; C:\Linkerd\"
[environment]::SetEnvironmentVariable("PATH", $PATH, "User") 
[environment]::SetEnvironmentVariable("PATH", $PATH)
```