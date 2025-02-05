---
title: Informazioni sulla sicurezza dell'hub IoT di Azure | Microsoft Docs
description: Guida per sviluppatori - Come controllare l'accesso all'hub IoT per app back-end e per dispositivi. Include informazioni sui token di sicurezza e sul supporto per i certificati x.509.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/18/2018
ms.openlocfilehash: fa1aa8c560f4b9cc48c7a6a761abe4d69d5d0265
ms.sourcegitcommit: a4b5d31b113f520fcd43624dd57be677d10fc1c0
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/06/2019
ms.locfileid: "70773181"
---
# <a name="control-access-to-iot-hub"></a>Controllare l'accesso all'hub IoT

Questo articolo illustra le opzioni per la protezione dell'hub IoT. L'hub IoT usa le *autorizzazioni* per concedere l'accesso a ogni endpoint dell'hub stesso. Le autorizzazioni limitano l'accesso a un hub IoT in base alla funzionalità.

Questo articolo descrive:

* Le diverse autorizzazioni che è possibile concedere a un'app per dispositivo o back-end per accedere all'hub IoT.
* Il processo di autenticazione e i token usati per verificare le autorizzazioni.
* Come definire l'ambito delle credenziali per limitare l'accesso a risorse specifiche.
* Supporto dell'hub IoT per i certificati x.509.
* Meccanismo di autenticazione personalizzata del dispositivo che usa gli schemi di autenticazione o i registri di identità del dispositivo esistenti.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

È necessario avere le autorizzazioni appropriate per accedere agli endpoint dell'hub IoT. Un dispositivo, ad esempio, deve includere un token contenente le credenziali di sicurezza con ogni messaggio inviato all'hub IoT.

## <a name="access-control-and-permissions"></a>Controllo dell'accesso e autorizzazioni

Per concedere le [autorizzazioni](#iot-hub-permissions) è possibile procedere nei modi seguenti:

* **Criteri di accesso condivisi a livello di hub IoT**. I criteri di accesso condiviso possono concedere qualsiasi combinazione di [autorizzazioni](#iot-hub-permissions). È possibile definire i criteri nel [portale di Azure](https://portal.azure.com) a livello di codice tramite le [API REST di risorsa dell'hub IoT](/rest/api/iothub/iothubresource) o tramite il comando dell'interfaccia della riga di comando [az iot hub policy](/cli/azure/iot/hub/policy?view=azure-cli-latest). Un hub IoT appena creato ha i criteri predefiniti seguenti:
  
  | Criteri di accesso condiviso | Autorizzazioni |
  | -------------------- | ----------- |
  | iothubowner | Tutte le autorizzazioni |
  | servizio | Autorizzazioni **ServiceConnect** |
  | device | Autorizzazioni **DeviceConnect** |
  | registryRead | Autorizzazioni **RegistryRead** |
  | registryReadWrite | Autorizzazioni **RegistryRead** e **RegistryWrite** |

* **Credenziali di sicurezza specifiche del dispositivo**. Ogni hub IoT contiene un [registro delle identità](iot-hub-devguide-identity-registry.md). Per ogni dispositivo presente in questo registro delle identità è possibile configurare credenziali di sicurezza che concedono autorizzazioni **DeviceConnect** con ambito agli endpoint di dispositivo corrispondenti.

Ad esempio, in una soluzione IoT tipica:

* Il componente di gestione dei dispositivi usa i criteri *registryReadWrite* .
* Il componente processore di eventi usa i criteri *service* .
* Il componente della logica di business di runtime del dispositivo usa i criteri *service* .
* I singoli dispositivi si connettono usando le credenziali archiviate nel registro delle identità dell'hub IoT.

> [!NOTE]
> Per informazioni dettagliate, vedere [Autorizzazioni](#iot-hub-permissions).

## <a name="authentication"></a>Authentication

L'hub IoT di Azure concede l'accesso agli endpoint tramite la verifica di un token rispetto ai criteri di accesso condiviso e alle credenziali di sicurezza del registro delle identità.

Le credenziali di sicurezza, ad esempio le chiavi asimmetriche, non vengono mai trasmesse in rete.

> [!NOTE]
> Il provider di risorse dell'hub IoT di Azure viene protetto tramite la sottoscrizione di Azure, analogamente a tutti i provider in [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md).

Per altre informazioni sulla creazione e sull'uso di token di sicurezza, vedere [Token di sicurezza dell'hub IoT](iot-hub-devguide-security.md#security-tokens).

### <a name="protocol-specifics"></a>Specifiche del protocollo

Ogni protocollo supportato, ad esempio MQTT, AMQP e HTTPS, trasporta i token in modo diverso.

Quando si usa MQTT, il pacchetto CONNECT ha deviceId come ClientId, `{iothubhostname}/{deviceId}` nel campo Username e un token di firma di accesso condiviso nel campo Password. `{iothubhostname}` deve essere il record CName completo dell'hub IoT, ad esempio contoso.azure-devices.net.

Quando si usa [AMQP](https://www.amqp.org/), l'hub IoT supporta [SASL PLAIN](https://tools.ietf.org/html/rfc4616) e la [sicurezza basata sulle attestazioni AMQP](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc).

Se si usa la sicurezza basata sulle attestazioni AMQP, lo standard specifica come trasmettere questi token.

Per SASL PLAIN **username** può essere:

* `{policyName}@sas.root.{iothubName}` nel caso di token a livello di hub IoT.
* `{deviceId}@sas.{iothubname}` ne caso di token con ambito relativo al dispositivo.

In entrambi i casi, il campo della password contiene il token, come descritto in [Token di sicurezza dell'hub IoT](iot-hub-devguide-security.md#security-tokens).

Il protocollo HTTPS implementa l'autenticazione includendo un token valido nell'intestazione della richiesta **Authorization** .

#### <a name="example"></a>Esempio

Nome utente (per DeviceId viene fatta distinzione tra maiuscole e minuscole): `iothubname.azure-devices.net/DeviceId`

Password (è possibile generare un token di firma di accesso condiviso con lo strumento [Device Explorer](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer), con il comando di estensione dell'interfaccia della riga di comando [az iot hub generate-sas-token](/cli/azure/ext/azure-cli-iot-ext/iot/hub?view=azure-cli-latest#ext-azure-cli-iot-ext-az-iot-hub-generate-sas-token) o con [Azure IoT Tools per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)):

`SharedAccessSignature sr=iothubname.azure-devices.net%2fdevices%2fDeviceId&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501`

> [!NOTE]
> Gli [Azure IoT SDK](iot-hub-devguide-sdks.md) generano automaticamente i token durante la connessione al servizio. In alcuni casi, gli Azure IoT SDK non supportano tutti i protocolli o tutti i metodi di autenticazione.

### <a name="special-considerations-for-sasl-plain"></a>Considerazioni speciali su SASL PLAIN

Quando si usa SASL PLAIN con AMQP, un client che si connette a un hub IoT potrà usare un singolo token per ogni connessione TCP. Quando il token scade, la connessione TCP si disconnette dal servizio e attiva una riconnessione. Questo comportamento non genera problemi per un'app back-end, ma è dannoso per un'app per dispositivi per i motivi seguenti:

* I gateway si connettono in genere per conto di molti dispositivi. Quando si usa SASL PLAIN, devono creare una connessione TCP distinta per ogni dispositivo che si connette a un hub IoT. Questo scenario aumenta in modo considerevole il consumo energetico e delle risorse di rete e incrementa la latenza della connessione di ogni dispositivo.

* L'aumento dell'uso delle risorse per la riconnessione dopo la scadenza di ogni token influisce negativamente sui dispositivi vincolati alle risorse.

## <a name="scope-iot-hub-level-credentials"></a>Definire l'ambito delle credenziali a livello di hub IoT

È possibile definire l'ambito dei criteri di sicurezza a livello di hub IoT creando token con URI di risorsa con limitazioni. L'endpoint per l'invio di messaggi da dispositivo a cloud da un dispositivo, ad esempio, è **/devices/{deviceId}/messages/events**. È anche possibile usare criteri di accesso condiviso a livello di hub IoT con autorizzazioni **DeviceConnect** per firmare un token il cui valore resourceURI è **/devices/{deviceId}** . Questo approccio crea un token che può essere usato solo per l'invio di messaggi per conto del dispositivo **deviceId**.

Questo meccanismo è simile ai [criteri dell'entità di pubblicazione di Hub eventi](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab) e consente di implementare metodi di autenticazione personalizzati.

## <a name="security-tokens"></a>Token di sicurezza

Hub IoT usa i token di sicurezza per autenticare i dispositivi e i servizi ed evitare l'invio in rete delle chiavi. Inoltre, i token di sicurezza hanno una validità limitata in termini di tempo e portata. Gli [Azure IoT SDK](iot-hub-devguide-sdks.md) generano automaticamente i token senza richiedere una configurazione speciale. In alcuni scenari è necessario generare e usare direttamente i token di sicurezza. Tali scenari includono:

* L'uso diretto di superfici MQTT, AMQP o HTTPS.

* L'implementazione del modello di servizio token, come descritto in [Autenticazione personalizzata del dispositivo](iot-hub-devguide-security.md#custom-device-and-module-authentication).

Hub IoT consente ai dispositivi di autenticarsi con l'hub IoT usando [certificati X.509](iot-hub-devguide-security.md#supported-x509-certificates).

### <a name="security-token-structure"></a>Formato del token di sicurezza

I token di sicurezza consentono di concedere a dispositivi e servizi l'accesso con limite temporale a funzionalità specifiche dell'hub IoT. Per ottenere l'autorizzazione per connettersi all'hub IoT, i dispositivi e i servizi devono inviare i token di sicurezza firmati con una chiave di accesso condiviso o una chiave simmetrica. Tali chiavi vengono archiviate con un'identità del dispositivo nel registro delle identità.

Un token firmato con una chiave di accesso condiviso concede l'accesso a tutte le funzionalità associate alle autorizzazioni dei criteri di accesso condiviso. Un token firmato con una chiave simmetrica dell'identità dispositivo concede solo l'autorizzazione **DeviceConnect** per l'identità del dispositivo associato.

Il token di sicurezza ha il formato seguente:

`SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

I valori previsti sono i seguenti:

| Valore | Descrizione |
| --- | --- |
| {signature} |Stringa della firma HMAC-SHA256 nel formato: `{URL-encoded-resourceURI} + "\n" + expiry`. **Importante**: la chiave viene decodificata dalla codifica Base64 e usata come chiave per eseguire il calcolo di HMAC-SHA256. |
| {resourceURI} |Prefisso URI (per segmento) degli endpoint a cui è possibile accedere tramite questo token e che inizia con il nome host dell'hub IoT senza il protocollo. Ad esempio: `myHub.azure-devices.net/devices/device1` |
| {expiry} |Stringhe UTF8 per il numero di secondi trascorsi dalle 00:00:00 UTC dell'1 gennaio 1970. |
| {URL-encoded-resourceURI} |Codifica URL con lettere minuscole dell'URI della risorsa con lettere minuscole |
| {policyName} |Nome del criterio di accesso condiviso a cui fa riferimento il token. Assente se il token fa riferimento a credenziali del registro dei dispositivi. |

**Nota sul prefisso**: il prefisso dell'URI viene calcolato in base al segmento e non in base al carattere. Ad esempio `/a/b` è un prefisso per `/a/b/c` ma non per `/a/bc`.

Il frammento seguente di Node.js mostra una funzione denominata **generateSasToken** che calcola il token dagli input `resourceUri, signingKey, policyName, expiresInMins`. Nelle sezioni successive viene illustrato nel dettaglio come inizializzare gli input a seconda del caso d'uso.

```javascript
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', Buffer.from(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires;
    if (policyName) token += "&skn="+policyName;
    return token;
};
```

Per fare un confronto, l'equivalente in termini di codice Python per generare un token di sicurezza è:

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import parse
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((parse.quote_plus(uri)), int(ttl))
    print sign_key
    signature = b64encode(HMAC(b64decode(key), sign_key.encode('utf-8'), sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl))
    }

    if policy_name is not None:
        rawtoken['skn'] = policy_name

    return 'SharedAccessSignature ' + parse.urlencode(rawtoken)
```

Di seguito sono riportate le istruzioni di installazione per i prerequisiti.

[!INCLUDE [Iot-hub-include-python-installation-notes](../../includes/iot-hub-include-python-installation-notes.md)]


La funzionalità di C# per generare un token di sicurezza è la seguente:

```csharp
using System;
using System.Globalization;
using System.Net;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;

public static string generateSasToken(string resourceUri, string key, string policyName, int expiryInSeconds = 3600)
{
    TimeSpan fromEpochStart = DateTime.UtcNow - new DateTime(1970, 1, 1);
    string expiry = Convert.ToString((int)fromEpochStart.TotalSeconds + expiryInSeconds);

    string stringToSign = WebUtility.UrlEncode(resourceUri) + "\n" + expiry;

    HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key));
    string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));

    string token = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}", WebUtility.UrlEncode(resourceUri), WebUtility.UrlEncode(signature), expiry);

    if (!String.IsNullOrEmpty(policyName))
    {
        token += "&skn=" + policyName;
    }

    return token;
}

```


> [!NOTE]
> Poiché la validità temporale del token viene verificata sui computer hub IoT, lo sfasamento dell'orologio del computer che genera il token deve essere minimo.

### <a name="use-sas-tokens-in-a-device-app"></a>Usare i token di firma di accesso condiviso in un dispositivo client

Esistono due modi per ottenere le autorizzazioni **DeviceConnect** con l'hub IoT con i token di sicurezza: usare una [chiave del dispositivo simmetrica dal registro delle identità](#use-a-symmetric-key-in-the-identity-registry) oppure usare una [chiave di accesso condiviso](#use-a-shared-access-policy).

Tenere presente che, per impostazione predefinita, tutte le funzionalità accessibili dai dispositivi vengono esposte negli endpoint con il prefisso `/devices/{deviceId}`.

> [!IMPORTANT]
> L'unico modo di cui dispone l'hub IoT per autenticare un dispositivo specifico è tramite la chiave simmetrica identità dispositivo. Nei casi in cui si acceda alle funzionalità del dispositivo tramite criteri di accesso condiviso, la soluzione deve considerare il componente che emette il token di sicurezza come sottocomponente attendibile.

Gli endpoint per il dispositivo sono, indipendentemente dal protocollo:

| Endpoint | Funzionalità |
| --- | --- |
| `{iot hub host name}/devices/{deviceId}/messages/events` |Invio di messaggi da dispositivo a cloud. |
| `{iot hub host name}/devices/{deviceId}/messages/devicebound` |Ricezione di messaggi da cloud a dispositivo. |

### <a name="use-a-symmetric-key-in-the-identity-registry"></a>Usare una chiave simmetrica nel registro identità

Quando si usa una chiave simmetrica dell'identità del dispositivo per generare un token, l'elemento policyName (`skn`) del token viene omesso.

Ad esempio, un token creato per accedere a tutte le funzionalità del dispositivo deve avere i seguenti parametri:

* URI della risorsa: `{IoT hub name}.azure-devices.net/devices/{device id}`,
* Chiave di firma: qualsiasi chiave simmetrica per l'identità `{device id}` ,
* Nessun nome di criterio,
* Qualsiasi ora di scadenza.

Un esempio di uso della funzione di Node.js precedente sarebbe il seguente:

```javascript
var endpoint ="myhub.azure-devices.net/devices/device1";
var deviceKey ="...";

var token = generateSasToken(endpoint, deviceKey, null, 60);
```

Il risultato, che concede l'accesso a tutte le funzionalità per device1, sarà:

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697`

> [!NOTE]
> È possibile generare un token di firma di accesso condiviso con lo strumento [Device Explorer](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer), con il comando di estensione dell'interfaccia della riga di comando [az iot hub generate-sas-token](/cli/azure/ext/azure-cli-iot-ext/iot/hub?view=azure-cli-latest#ext-azure-cli-iot-ext-az-iot-hub-generate-sas-token) o con [Azure IoT Tools per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

### <a name="use-a-shared-access-policy"></a>Usare criteri di accesso condiviso

Quando si crea un token da criteri di accesso condiviso, impostare il campo `skn` sul nome dei criteri. Questi criteri devono concedere l'autorizzazione **DeviceConnect**.

I due scenari principali per l'uso di criteri di accesso condiviso per accedere alla funzionalità dei dispositivi sono:

* [gateway del protocollo cloud](iot-hub-devguide-endpoints.md),
* [servizi token](iot-hub-devguide-security.md#custom-device-and-module-authentication) tramite i quali implementare schemi di autenticazione personalizzati.

Poiché i criteri di accesso condiviso possono potenzialmente autorizzare la connessione a qualsiasi dispositivo, in fase di creazione dei token di sicurezza è importante usare l'URI risorsa corretto. Questa impostazione è particolarmente importante per i servizi token, che devono limitare l'ambito del token a un dispositivo specifico usando l'URI risorsa. Questo punto è meno importante per i gateway di protocollo, in quanto già filtrano il traffico per tutti i dispositivi.

Ad esempio, un servizio token che usa il criterio di accesso condiviso già esistente denominato **device** creerebbe un token con i parametri seguenti:

* URI della risorsa: `{IoT hub name}.azure-devices.net/devices/{device id}`,
* chiave di firma: una delle chiavi del criterio `device` ,
* nome criterio: `device`,
* Qualsiasi ora di scadenza.

Un esempio di uso della funzione di Node.js precedente sarebbe il seguente:

```javascript
var endpoint ="myhub.azure-devices.net/devices/device1";
var policyName = 'device';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

Il risultato, che concede l'accesso a tutte le funzionalità per device1, sarà:

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices%2fdevice1&sig=13y8ejUk2z7PLmvtwR5RqlGBOVwiq7rQR3WZ5xZX3N4%3D&se=1456971697&skn=device`

Un gateway di protocollo potrebbe usare lo stesso token per tutti i dispositivi semplicemente impostando l'URI della risorsa su `myhub.azure-devices.net/devices`.

### <a name="use-security-tokens-from-service-components"></a>Usare token di sicurezza da componenti del servizio

I componenti del servizio possono generare token di sicurezza solo usando criteri di accesso condiviso che concedono le autorizzazioni appropriate, come illustrato prima.

Di seguito vengono indicate le funzioni del servizio esposte sugli endpoint:

| Endpoint | Funzionalità |
| --- | --- |
| `{iot hub host name}/devices` |Creazione, aggiornamento, recupero ed eliminazione delle identità dispositivo. |
| `{iot hub host name}/messages/events` |Ricezione di messaggi da dispositivo a cloud. |
| `{iot hub host name}/servicebound/feedback` |Ricezione di feedback per messaggi da cloud a dispositivo. |
| `{iot hub host name}/devicebound` |Invio di messaggi da cloud a dispositivo. |

Ad esempio, un servizio che usa il criterio di accesso condiviso già esistente denominato **registryRead** creerebbe un token con i parametri seguenti:

* URI della risorsa: `{IoT hub name}.azure-devices.net/devices`,
* chiave di firma: una delle chiavi del criterio `registryRead` ,
* nome criterio: `registryRead`,
* Qualsiasi ora di scadenza.

```javascript
var endpoint ="myhub.azure-devices.net/devices";
var policyName = 'registryRead';
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

Il risultato, che concede l'accesso in lettura a tutte le identità dispositivo, sarà:

`SharedAccessSignature sr=myhub.azure-devices.net%2fdevices&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=registryRead`

## <a name="supported-x509-certificates"></a>Certificati X.509 supportati

È possibile usare qualsiasi certificato X.509 per autenticare un dispositivo con l'hub IoT caricando un'identificazione personale del certificato o un'autorità di certificazione (CA) nell'hub IoT di Azure. L'autenticazione tramite identificazioni personali del certificato verifica solo che l'identificazione personale presentata corrisponda all'identificazione personale configurata. L'autenticazione tramite l'autorità di certificazione convalida la catena di certificati. 

I certificati supportati includono:

* **Un certificato X.509 esistente**. Un dispositivo potrebbe già avere un certificato X.509 associato. Il dispositivo può usare questo certificato per autenticarsi con hub IoT. È compatibile sia con l'autenticazione tramite identificazione personale che con l'autenticazione tramite autorità di certificazione. 

* **Certificato X.509 firmato da un'autorità di certificazione**. Per identificare un dispositivo e autenticarlo con l'hub IoT, è possibile usare un certificato X.509 generato e firmato da un'autorità di certificazione (CA). È compatibile sia con l'autenticazione tramite identificazione personale che con l'autenticazione tramite autorità di certificazione.

* **Un certificato X-509 auto-generato e auto-firmato**. Un produttore di dispositivi o un distributore interno può generare questi certificati e archiviare la chiave privata corrispondente (e il certificato) nel dispositivo. È possibile usare strumenti come [OpenSSL](https://www.openssl.org/) e l'utilità [Windows SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate) per questo scopo. È compatibile solo con l'autenticazione tramite identificazione personale. 

Un dispositivo può usare un certificato X.509 o un token di sicurezza per l'autenticazione, ma non per entrambi.

Per altre informazioni sull'autenticazione tramite l'autorità di certificazione, vedere [Autenticazione dei dispositivi con i certificati della CA X.509](iot-hub-x509ca-overview.md).

### <a name="register-an-x509-certificate-for-a-device"></a>Registrare un certificato X.509 per un dispositivo

Il componente [Azure IoT SDK per servizi per C#](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/service) (versione 1.0.8+) supporta la registrazione di un dispositivo che usa un certificato X.509 per l'autenticazione. Anche altre API come quelle per l'importazione e l'esportazione dei dispositivi supportano i certificati X.509.

È anche possibile usare il comando di estensione dell'interfaccia della riga di comando [az iot hub device-identity](/cli/azure/ext/azure-cli-iot-ext/iot/hub/device-identity?view=azure-cli-latest) per configurare i certificati X.509 per i dispositivi.

### <a name="c-support"></a>Supporto per C\#

La classe **RegistryManager** offre un modo di registrare un dispositivo a livello di codice. In particolare, i metodi **AddDeviceAsync** e **UpdateDeviceAsync** consentono di registrare e aggiornare un dispositivo nel registro delle identità dell'hub IoT. Questi due metodi accettano un'istanza **Device** come input. La classe **Device** include una proprietà **Authentication** che consente di specificare le identificazioni primarie e secondarie del certificato X.509. L'identificazione personale rappresenta un hash SHA256 del certificato X.509 archiviato usando la codifica DER binaria. Gli utenti hanno la possibilità di specificare un'identificazione personale primaria, una secondaria o entrambe. Le identificazioni personali primarie e secondarie sono supportate per gestire scenari di rollover dei certificati.

Ecco un frammento di codice C\# di esempio per registrare un dispositivo usando l'identificazione personale di un certificato X.509:

```csharp
var device = new Device(deviceId)
{
  Authentication = new AuthenticationMechanism()
  {
    X509Thumbprint = new X509Thumbprint()
    {
      PrimaryThumbprint = "B4172AB44C28F3B9E117648C6F7294978A00CDCBA34A46A1B8588B3F7D82C4F1"
    }
  }
};
RegistryManager registryManager = RegistryManager.CreateFromConnectionString(deviceGatewayConnectionString);
await registryManager.AddDeviceAsync(device);
```

### <a name="use-an-x509-certificate-during-run-time-operations"></a>Usare un certificato X.509 durante le operazioni di runtime

[Azure IoT SDK per dispositivi per .NET](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/device) (versione 1.0.11+) supporta l'uso dei certificati X.509.

### <a name="c-support"></a>Supporto per C\#

La classe **DeviceAuthenticationWithX509Certificate** supporta la creazione di istanze di **DeviceClient** usando un certificato X.509. Il certificato X.509 deve essere nel formato PFX, denominato anche PKCS #12, che include la chiave privata.

Di seguito è riportato un frammento di codice di esempio:

```csharp
var authMethod = new DeviceAuthenticationWithX509Certificate("<device id>", x509Certificate);

var deviceClient = DeviceClient.Create("<IotHub DNS HostName>", authMethod);
```

## <a name="custom-device-and-module-authentication"></a>Autenticazione personalizzata di dispositivi e moduli

È possibile usare il [registro delle identità](iot-hub-devguide-identity-registry.md) dell'hub IoT per configurare il controllo dell'accesso e le credenziali di sicurezza per ogni dispositivo/modulo usando i [token](iot-hub-devguide-security.md#security-tokens). Se una soluzione IoT ha già un registro personalizzato delle identità e/o uno schema di autenticazione, valutare la possibilità di creare un *servizio token* per integrare l'infrastruttura con l'hub IoT. In questo modo, è possibile usare altre funzionalità IoT nella soluzione.

Un servizio token è un servizio cloud personalizzato. Usa i *criteri di accesso condiviso* dell'hub IoT con autorizzazioni **DeviceConnect** o **ModuleConnect** per creare token *basati sul dispositivo* o *basati sul modulo*. Questi token abilitano la connessione di un dispositivo e un modulo all'hub IoT.

![Passaggi del modello di servizio token](./media/iot-hub-devguide-security/tokenservice.png)

Di seguito vengono indicati i passaggi principali del modello del servizio token:

1. Creare i criteri di accesso condiviso dell'hub IoT con autorizzazioni **DeviceConnect** o **ModuleConnect** per l'hub IoT. È possibile creare questi criteri nel [portale di Azure](https://portal.azure.com) o a livello di programmazione. Il servizio token usa questi criteri per firmare i token creati.

2. Quando un dispositivo/modulo deve accedere all'hub IoT, richiede un token firmato dal servizio token. Il dispositivo può eseguire l'autenticazione con il registro delle identità personalizzato/lo schema di autenticazione per determinare l'identità del dispositivo/modulo usata dal servizio token per creare il token.

3. Il servizio token restituisce un token. Il token viene creato tramite `/devices/{deviceId}` o `/devices/{deviceId}/module/{moduleId}` come `resourceURI`, con `deviceId` come dispositivo da autenticare o `moduleId` come modulo da autenticare. Il servizio token usa i criteri di accesso condivisi per costruire il token.

4. Il dispositivo/modulo usa il token direttamente con l'hub IoT.

> [!NOTE]
> È possibile usare la classe .NET [SharedAccessSignatureBuilder](https://msdn.microsoft.com/library/microsoft.azure.devices.common.security.sharedaccesssignaturebuilder.aspx) o la classe Java [IotHubServiceSasToken](/java/api/com.microsoft.azure.sdk.iot.service.auth.iothubservicesastoken) per creare un token nel servizio token.

Il servizio token può impostare la scadenza del token, in base alle esigenze. Quando il token scade, l'hub IoT interrompe la connessione del dispositivo/modulo. Il dispositivo/modulo deve quindi richiedere un nuovo token dal servizio token. Un intervallo di scadenza breve aumenta il carico sia sul dispositivo/modulo che sul servizio token.

Perché un dispositivo/modulo si connetta all'hub, è comunque necessario aggiungerlo al registro delle identità dell'hub IoT anche se il dispositivo/modulo usa un token e non una chiave per la connessione. È quindi possibile continuare a usare il controllo dell'accesso per ogni dispositivo/modulo abilitando o disabilitando le identità dei dispositivi/moduli nel [registro delle identità](iot-hub-devguide-identity-registry.md). In questo modo si riduce il rischio che vengano usati token con intervalli di scadenza prolungati.

### <a name="comparison-with-a-custom-gateway"></a>Confronto con un gateway personalizzato

Il modello di servizio token è il metodo consigliato per implementare uno schema di autenticazione/registro di identità personalizzato con l'hub IoT. Questo schema è consigliato perché l'hub IoT continua a gestire la maggior parte del traffico della soluzione. Tuttavia, se lo schema di autenticazione personalizzato è molto legato al protocollo, può essere necessario un *gateway personalizzato* per elaborare tutto il traffico. Un esempio di tale scenario prevede l'uso del [protocollo TLS (Transport Layer Security) e di chiavi precondivise](https://tools.ietf.org/html/rfc4279). Per altre informazioni, vedere l'articolo relativo al [gateway del protocollo](iot-hub-protocol-gateway.md).

## <a name="reference-topics"></a>Argomenti di riferimento:

Gli argomenti di riferimento seguenti offrono altre informazioni sul controllo dell'accesso all'hub IoT.

## <a name="iot-hub-permissions"></a>Autorizzazioni per l'hub IoT

La tabella seguente elenca le autorizzazioni che è possibile usare per controllare l'accesso all'hub IoT.

| Autorizzazioni | Note |
| --- | --- |
| **RegistryRead** |Concede l'accesso di sola lettura al registro di identità. Per altre informazioni, vedere [Registro delle identità](iot-hub-devguide-identity-registry.md). <br/>Questa autorizzazione viene usata dai servizi cloud back-end. |
| **RegistryReadWrite** |Concede l'accesso di lettura e scrittura al registro di identità. Per altre informazioni, vedere [Registro delle identità](iot-hub-devguide-identity-registry.md). <br/>Questa autorizzazione viene usata dai servizi cloud back-end. |
| **ServiceConnect** |Concede l'accesso alle comunicazioni per il servizio cloud e al monitoraggio degli endpoint. <br/>Concede l'autorizzazione per la ricezione di messaggi da dispositivo a cloud, l'invio di messaggi da cloud a dispositivo e il recupero degli acknowledgment di recapito corrispondenti. <br/>Concede l'autorizzazione per recuperare i riconoscimenti di recapito per i caricamenti di file. <br/>Concede l'autorizzazione per l'accesso a dispositivi/moduli gemelli per l'aggiornamento dei tag e delle proprietà indicate, il recupero delle proprietà segnalate e l'esecuzione di query. <br/>Questa autorizzazione viene usata dai servizi cloud back-end. |
| **DeviceConnect** |Concede l'accesso agli endpoint per il dispositivo. <br/>Concede l'autorizzazione per l'invio di messaggi da dispositivo a cloud e la ricezione di messaggi da cloud a dispositivo. <br/>Concede l'autorizzazione per il caricamento di file da un dispositivo. <br/>Concede l'autorizzazione per la ricezione di notifiche su particolari proprietà del dispositivo gemello e l'aggiornamento delle proprietà segnalate di quest'ultimo. <br/>Concede l'autorizzazione per il caricamento di file. <br/>Questa autorizzazione viene usata dai dispositivi. |

## <a name="additional-reference-material"></a>Materiale di riferimento

Di seguito sono indicati altri argomenti di riferimento reperibili nella Guida per gli sviluppatori dell'hub IoT:

* [Endpoint dell'hub IoT](iot-hub-devguide-endpoints.md) illustra i diversi endpoint esposti da ogni hub IoT per operazioni della fase di esecuzione e di gestione.

* [Quote e limitazioni](iot-hub-devguide-quotas-throttling.md) descrive le quote e i comportamenti di limitazione applicabili al servizio hub IoT.

* [Azure IoT SDK per dispositivi e servizi](iot-hub-devguide-sdks.md) elenca gli SDK nei diversi linguaggi che è possibile usare quando si sviluppano app per dispositivi e servizi che interagiscono con l'hub IoT.

* [Linguaggio di query dell'hub IoT](iot-hub-devguide-query-language.md) descrive il linguaggio di query che è possibile usare per recuperare informazioni dall'hub IoT sui dispositivi gemelli e sui processi.

* [Supporto di MQTT nell'hub IoT](iot-hub-mqtt-support.md) offre altre informazioni sul supporto dell'hub IoT per il protocollo MQTT.

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione si è appreso come controllare l'accesso all'hub IoT. Altri argomenti di interesse disponibili nella Guida per sviluppatori sono i seguenti:

* [Use device twins to synchronize state and configurations](iot-hub-devguide-device-twins.md) (Usare dispositivi gemelli per sincronizzare lo stato e le configurazioni)
* [Richiamare un metodo diretto in un dispositivo](iot-hub-devguide-direct-methods.md)
* [Pianificare processi in più dispositivi](iot-hub-devguide-jobs.md)

Per provare alcuni dei concetti descritti in questo articolo, vedere le esercitazioni sull'hub IoT seguenti:

* [Introduzione all'hub IoT di Azure](quickstart-send-telemetry-node.md)
* [Inviare messaggi da cloud a dispositivo con l'hub IoT](iot-hub-csharp-csharp-c2d.md)
* [Elaborare messaggi da dispositivo a cloud dell'hub IoT](tutorial-routing.md)
