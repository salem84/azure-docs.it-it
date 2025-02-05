---
title: Salvataggio sicuro delle impostazioni applicazione segrete per un'applicazione Web - Azure Key Vault | Microsoft Docs
description: Come salvare in modo sicuro le impostazioni applicazione segrete, come le credenziali di Azure o le chiavi API di terze parti, usando il Key Vault Provider di ASP.NET Core, User Secret o i generatori di configurazione di .NET 4.7.1
services: visualstudio
author: cawaMS
manager: paulyuk
editor: ''
ms.service: key-vault
ms.topic: conceptual
ms.date: 07/17/2019
ms.author: cawa
ms.openlocfilehash: d5662fa3cae8ba0cec0fd76965597ccac7c83889
ms.sourcegitcommit: 36e9cbd767b3f12d3524fadc2b50b281458122dc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/20/2019
ms.locfileid: "69639476"
---
# <a name="securely-save-secret-application-settings-for-a-web-application"></a>Salvataggio sicuro delle impostazioni applicazione segrete per un'applicazione Web

## <a name="overview"></a>Panoramica
Questo articolo descrive come salvare in modo sicuro le impostazioni di configurazione dell'applicazione segrete per le applicazioni Azure.

In genere tutte le impostazioni di configurazione delle applicazioni Web vengono salvate in file di configurazione come Web.config. Questa procedura porta all'archiviazione di impostazioni segrete, ad esempio le credenziali del cloud, in sistemi di controllo del codice sorgente pubblici come GitHub. Nel frattempo potrebbe essere difficile seguire una procedura consigliata di sicurezza a causa dell'overhead necessario per modificare il codice sorgente e riconfigurare le impostazioni di sviluppo.

Per assicurarsi che il processo di sviluppo sia sicuro, gli strumenti e le librerie del Framework vengono creati per salvare le impostazioni del segreto dell'applicazione in modo sicuro con una modifica minima o senza codice sorgente.

## <a name="aspnet-and-net-core-applications"></a>Applicazioni ASP.NET e .NET Core

### <a name="save-secret-settings-in-user-secret-store-that-is-outside-of-source-control-folder"></a>Salvare le impostazioni segrete nell'archivio User Secret che si trova fuori dalla cartella di controllo del codice sorgente
Se si sta eseguendo un prototipo rapido o non si ha accesso a Internet, iniziare spostando le impostazioni segrete fuori dalla cartella di controllo del codice sorgente all'archivio User Secret. L'archivio User Secret è un file salvato nella cartella del profilo utente, in modo che i segreti non vengano archiviati nel controllo del codice sorgente. Il diagramma seguente dimostra come funziona [User Secret](https://docs.microsoft.com/aspnet/core/security/app-secrets?tabs=visual-studio).

![User Secret mantiene le impostazioni segrete fuori dal controllo del codice sorgente](./media/vs-secure-secret-appsettings/aspnetcore-usersecret.PNG)

Se si esegue l'applicazione console .NET Core, usare Key Vault per salvare il segreto in modo sicuro.

### <a name="save-secret-settings-in-azure-key-vault"></a>Salvare le impostazioni segrete in Azure Key Vault
Se si sviluppa un progetto ed è necessario condividere il codice sorgente in modo sicuro, usare [Azure Key Vault](https://azure.microsoft.com/services/key-vault/).

1. Creare un Key Vault nella sottoscrizione di Azure. Compilare tutti i campi obbligatori nell'interfaccia utente e fare clic su *Crea* nella parte inferiore del pannello

    ![Creare un Azure Key Vault](./media/vs-secure-secret-appsettings/create-keyvault.PNG)

2. Concedere l'accesso al Key Vault a tutti i membri del team. Se il team è grande, è possibile creare un [gruppo di Azure Active Directory](../active-directory/active-directory-groups-create-azure-portal.md) e aggiungere l'accesso di quel gruppo di sicurezza al Key Vault. Nel menu a discesa *Autorizzazioni segrete* selezionare *Ottieni* ed *Elenca* sotto *Operazioni di gestione dei segreti*.

    ![Aggiungere criteri di accesso del Key Vault](./media/vs-secure-secret-appsettings/add-keyvault-access-policy.png)

3. Aggiungere il segreto per Key Vault nel portale di Azure. Per le impostazioni di configurazione annidate, sostituire ":" con "-" in modo che il nome del segreto del Key Vault sia valido. Il segno ":" non è ammesso nel nome di un segreto del Key Vault.

    ![Aggiungere il segreto del Key Vault](./media/vs-secure-secret-appsettings/add-keyvault-secret.png)

    > [!NOTE] 
    > Prima di Visual Studio 2017 V 15.6 è stato usato per consigliare l'installazione dell'estensione di autenticazione dei servizi di Azure per Visual Studio. Questa funzionalità è tuttavia deprecata perché la funzionalità è integrata in Visual Studio. Di conseguenza, se si usa una versione precedente di Visual Studio 2017, è consigliabile eseguire l'aggiornamento ad almeno 2017 15,6 o versione successiva per poter usare questa funzionalità in modo nativo e accedere all'insieme di credenziali delle chiavi usando l'identità di accesso di Visual Studio.
    >
 
4. Aggiungere i pacchetti NuGet seguenti al progetto:

    ```
    Microsoft.Azure.Services.AppAuthentication
    ```
5. Aggiungere il codice seguente al file Program.cs:

    ```csharp
    public static IWebHost BuildWebHost(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((ctx, builder) =>
            {
                var keyVaultEndpoint = GetKeyVaultEndpoint();
                if (!string.IsNullOrEmpty(keyVaultEndpoint))
                {
                    var azureServiceTokenProvider = new AzureServiceTokenProvider();
                    var keyVaultClient = new KeyVaultClient(
                        new KeyVaultClient.AuthenticationCallback(
                            azureServiceTokenProvider.KeyVaultTokenCallback));
                            builder.AddAzureKeyVault(
                            keyVaultEndpoint, keyVaultClient, new DefaultKeyVaultSecretManager());
                        }
                    })
                    .UseStartup<Startup>()
                    .Build();

        private static string GetKeyVaultEndpoint() => Environment.GetEnvironmentVariable("KEYVAULT_ENDPOINT");
    ```
6. Aggiungere l'URL del Key Vault al file launchsettings.json. Il nome di variabile di ambiente *KEYVAULT_ENDPOINT* è definito nel codice aggiunto al passaggio 6.

    ![Aggiungere l'URL del Key Vault come variabile di ambiente del progetto](./media/vs-secure-secret-appsettings/add-keyvault-url.png)

7. Avviare il debug del progetto. Sarà eseguito correttamente.

## <a name="aspnet-and-net-applications"></a>Applicazioni ASP.NET e .NET

.NET 4.7.1 supporta i generatori di configurazione Key Vault e segreto, il che garantisce che i segreti possano essere spostati fuori dalla cartella di controllo del codice sorgente senza modifiche al codice.
Per procedere, [scaricare .NET 4.7.1](https://www.microsoft.com/download/details.aspx?id=56115) ed eseguire la migrazione dell'applicazione se usa una versione precedente di .NET Framework.

### <a name="save-secret-settings-in-a-secret-file-that-is-outside-of-source-control-folder"></a>Salvare le impostazioni segrete in un file del segreto fuori dalla cartella di controllo del codice sorgente
Se si sta scrivendo un prototipo veloce e non si desidera eseguire il provisioning di risorse di Azure, procedere con questa opzione.

1. Installare il seguente pacchetto NuGet nel progetto
    ```
    Microsoft.Configuration.ConfigurationBuilders.Base
    ```

2. Creare un file simile al seguente. Salvarlo in una posizione esterna alla cartella del progetto.

    ```xml
    <root>
        <secrets ver="1.0">
            <secret name="secret1" value="foo_one" />
            <secret name="secret2" value="foo_two" />
        </secrets>
    </root>
    ```

3. Definire il file del segreto come generatore di configurazione nel file Web.config. Mettere questa sezione prima della sezione *appSettings*.

    ```xml
    <configBuilders>
        <builders>
            <add name="Secrets"
                 secretsFile="C:\Users\AppData\MyWebApplication1\secret.xml" type="Microsoft.Configuration.ConfigurationBuilders.UserSecretsConfigBuilder,
                    Microsoft.Configuration.ConfigurationBuilders, Version=1.0.0.0, Culture=neutral" />
        </builders>
    </configBuilders>
    ```

4. Specificare che la sezione appSettings sta usando il generatore di configurazione segreto. Verificare che sia presente una voce per l'impostazione del segreto con un valore fittizio.

    ```xml
        <appSettings configBuilders="Secrets">
            <add key="webpages:Version" value="3.0.0.0" />
            <add key="webpages:Enabled" value="false" />
            <add key="ClientValidationEnabled" value="true" />
            <add key="UnobtrusiveJavaScriptEnabled" value="true" />
            <add key="secret" value="" />
        </appSettings>
    ```

5. Eseguire il debug dell'app. Sarà eseguito correttamente.

### <a name="save-secret-settings-in-an-azure-key-vault"></a>Salvare le impostazioni segrete in un Azure Key Vault
Seguire le istruzioni nella sezione su ASP.NET Core per configurare un Key Vault per il progetto.

1. Installare il seguente pacchetto NuGet nel progetto
   ```
   Microsoft.Configuration.ConfigurationBuilders.UserSecrets
   ```

2. Definire il generatore di configurazione Key Vault in Web.config. Mettere questa sezione prima della sezione *appSettings*. Sostituire *vaultName* in modo che sia il nome del Key Vault, se il Key Vault si trova in Azure pubblico, o l'URI completo, se si usa un sovereign cloud.

    ```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>
    <configBuilders>
        <builders>
            <add name="AzureKeyVault" vaultName="Test911" type="Microsoft.Configuration.ConfigurationBuilders.AzureKeyVaultConfigBuilder, ConfigurationBuilders, Version=1.0.0.0, Culture=neutral" />
        </builders>
    </configBuilders>
    ```
3. Specificare che la sezione appSettings sta usando il generatore di configurazione Key Vault. Assicurarsi che tutte le voci per l'impostazione segreta abbiano un valore fittizio.

   ```xml
   <appSettings configBuilders="AzureKeyVault">
       <add key="webpages:Version" value="3.0.0.0" />
       <add key="webpages:Enabled" value="false" />
       <add key="ClientValidationEnabled" value="true" />
       <add key="UnobtrusiveJavaScriptEnabled" value="true" />
       <add key="secret" value="" />
   </appSettings>
   ```

4. Avviare il debug del progetto. Sarà eseguito correttamente.
