---
title: Pianificare processi con l'hub IoT di Azure (.NET/.NET) | Microsoft Docs
description: Come pianificare un processo dell'hub IoT di Azure per richiamare un metodo diretto su più dispositivi. Usare Azure IoT SDK per dispositivi per .NET per implementare le app per dispositivi simulati un'app di servizio per eseguire il processo.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: robinsh
ms.openlocfilehash: db4824203f63fa2fe0d4256a475d18a501b17e0e
ms.sourcegitcommit: aaa82f3797d548c324f375b5aad5d54cb03c7288
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/29/2019
ms.locfileid: "70147744"
---
# <a name="schedule-and-broadcast-jobs-net"></a>Pianificare e trasmettere processi (.NET)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Usare l'hub IoT per pianificare e tenere traccia dei processi che aggiornano milioni di dispositivi. Usare i processi per:

* Aggiornare le proprietà desiderate

* Aggiornare i tag

* Richiamare metodi diretti

Un processo esegue il wrapping di una di queste azioni e tiene traccia dell'esecuzione rispetto a un set di dispositivi, definito da una query su dispositivi gemelli. Ad esempio, un'applicazione back-end può usare un processo per chiamare un metodo diretto su 10.000 dispositivi che riavvia i dispositivi stessi. È necessario specificare il set di dispositivi con una query di dispositivi gemelli e pianificare il processo in modo che venga eseguito in un secondo momento. L'applicazione può quindi tenere traccia dell'avanzamento mentre ognuno dei dispositivi riceve ed esegue il metodo diretto di riavvio.

Per altre informazioni su queste funzionalità, vedere:

* Dispositivo gemello e proprietà: [Introduzione ai dispositivi gemelli](iot-hub-csharp-csharp-twin-getstarted.md) ed [esercitazione: Come usare le proprietà dei dispositivi gemelli](tutorial-device-twins.md)

* Metodi diretti: [Guida per gli sviluppatori dell'hub Internet-metodi diretti](iot-hub-devguide-direct-methods.md) ed [esercitazione: Usare metodi diretti](quickstart-control-device-dotnet.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Questa esercitazione illustra come:

* Creare un'app per dispositivi che implementi un metodo diretto denominato **LockDoor**, che può essere chiamato dall'app back-end.

* Creare un'app back-end che crea un processo per chiamare il metodo diretto **LockDoor** su più dispositivi. Un altro processo invia gli aggiornamenti di proprietà desiderati a più dispositivi.

Al termine di questa esercitazione si ottengono due app console .NET (C#):

* **SimulateDeviceMethods**. Questa app si connette all'hub Internet e implementa il metodo diretto **LockDoor** .

* **ScheduleJob**. Questa app usa i processi per chiamare il metodo diretto **LockDoor** e aggiornare le proprietà desiderate del dispositivo gemello su più dispositivi.

## <a name="prerequisites"></a>Prerequisiti

* Visual Studio.

* Un account Azure attivo. Se non si ha un account, è possibile creare un [account gratuito](https://azure.microsoft.com/pricing/free-trial/) in pochi minuti.

## <a name="create-an-iot-hub"></a>Creare un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registrare un nuovo dispositivo nell'hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-simulated-device-app"></a>Creare un'app di dispositivo simulato

In questa sezione viene creata un'app console .NET che risponde a un metodo diretto chiamato dal back-end della soluzione.

1. In Visual Studio selezionare **Crea un nuovo progetto**, quindi scegliere il modello di progetto **App Console (.NET Framework)** . Selezionare **Avanti** per continuare.

1. In **Configura il nuovo progetto**assegnare al progetto il nome *SimulateDeviceMethods*e quindi selezionare **Crea**.

    ![Configurare il progetto SimulateDeviceMethods](./media/iot-hub-csharp-csharp-schedule-jobs/configure-device-app.png)

1. In Esplora soluzioni fare clic con il pulsante destro del mouse sul progetto **SimulateDeviceMethods** e quindi scegliere **Gestisci pacchetti NuGet**.

1. In **Gestione pacchetti NuGet**selezionare **Sfoglia** e cercare e scegliere **Microsoft. Azure. Devices. client**. Selezionare **Installa**.

    ![App client della finestra Gestione pacchetti NuGet](./media/iot-hub-csharp-csharp-schedule-jobs/device-app-nuget.png)

    Questo passaggio consente di scaricare, installare e aggiungere un riferimento al pacchetto NuGet [Azure Azure per dispositivi SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) e alle relative dipendenze.

1. Aggiungere le istruzione `using` seguenti all'inizio del file **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

1. Aggiungere i campi seguenti alla classe **Program** . Sostituire il valore del segnaposto con la stringa di connessione del dispositivo annotato nella sezione precedente:

    ```csharp
    static string DeviceConnectionString = "<yourDeviceConnectionString>";
    static DeviceClient Client = null;
    ```

1. Aggiungere il codice seguente per implementare il metodo diretto nel dispositivo:

    ```csharp
    static Task<MethodResponse> LockDoor(MethodRequest methodRequest, object userContext)
    {
        Console.WriteLine();
        Console.WriteLine("Locking Door!");
        Console.WriteLine("\nReturning response for method {0}", methodRequest.Name);

        string result = "'Door was locked.'";
        return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
    }
    ```

1. Aggiungere il metodo seguente per implementare il listener dei dispositivi gemelli sul dispositivo:

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
    {
        Console.WriteLine("Desired property change:");
        Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
    }
    ```

1. Infine aggiungere il codice seguente al metodo **Main** per aprire la connessione all'hub IoT e inizializzare il listener del metodo:

    ```csharp
    try
    {
        Console.WriteLine("Connecting to hub");
        Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
          TransportType.Mqtt);

        Client.SetMethodHandlerAsync("LockDoor", LockDoor, null);
        Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null);

        Console.WriteLine("Waiting for direct method call and device twin update\n Press enter to exit.");
        Console.ReadLine();

        Console.WriteLine("Exiting...");

        Client.SetMethodHandlerAsync("LockDoor", null, null);
        Client.CloseAsync().Wait();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. Salvare il lavoro e compilare la soluzione.

> [!NOTE]
> Per semplificare le operazioni, questa esercitazione non implementa alcun criterio di ripetizione dei tentativi. Nel codice di produzione è consigliabile implementare criteri di ripetizione dei tentativi, ad esempio un nuovo tentativo di connessione, come suggerito in [gestione degli errori temporanei](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Ottenere la stringa di connessione dell'hub Internet

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="schedule-jobs-for-calling-a-direct-method-and-sending-device-twin-updates"></a>Pianificare i processi per chiamare un metodo diretto e inviare gli aggiornamenti dei dispositivi gemelli

In questa sezione si crea un'app console .NET (usando C#) che usa i processi per chiamare il metodo diretto **LockDoor** e inviare gli aggiornamenti di proprietà desiderati a più dispositivi.

1. In Visual Studio selezionare **File** > **Nuovo** > **Progetto**. In **Crea un nuovo progetto**scegliere **App Console (.NET Framework)** , quindi fare clic su **Avanti**.

1. In **Configura il nuovo progetto**assegnare al progetto il nome *ScheduleJob*. Per **soluzione**scegliere **Aggiungi a soluzione**, quindi selezionare **Crea**.

    ![Nome e configurazione del progetto ScheduleJob](./media/iot-hub-csharp-csharp-schedule-jobs/config-schedule-job-app.png)

1. In Esplora soluzioni fare clic con il pulsante destro del mouse sul progetto **ScheduleJob** e quindi scegliere **Gestisci pacchetti NuGet**.

1. In **Gestione pacchetti NuGet**selezionare **Sfoglia**, cercare e scegliere **Microsoft. Azure. Devices**, quindi selezionare Install ( **Installa**).

   Questa procedura scarica, installa e aggiunge un riferimento al pacchetto NuGet [Azure IoT SDK per servizi](https://www.nuget.org/packages/Microsoft.Azure.Devices/) e alle relative dipendenze.

1. Aggiungere le istruzione `using` seguenti all'inizio del file **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices;
    using Microsoft.Azure.Devices.Shared;
    ```

1. Aggiungere l'istruzione `using` seguente se non è già presente nelle istruzioni predefinite.

    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    ```

1. Aggiungere i campi seguenti alla classe **Program** . Sostituire i segnaposto con la stringa di connessione dell'hub Internet che è stata copiata in precedenza in [ottenere la stringa di connessione dell'hub Internet](#get-the-iot-hub-connection-string) e il nome del dispositivo.

    ```csharp
    static JobClient jobClient;
    static string connString = "<yourIotHubConnectionString>";
    static string deviceId = "<yourDeviceId>";
    ```

1. Aggiungere il metodo seguente alla classe **Program** :

    ```csharp
    public static async Task MonitorJob(string jobId)
    {
        JobResponse result;
        do
        {
            result = await jobClient.GetJobAsync(jobId);
            Console.WriteLine("Job Status : " + result.Status.ToString());
            Thread.Sleep(2000);
        } while ((result.Status != JobStatus.Completed) && 
          (result.Status != JobStatus.Failed));
    }
    ```

1. Aggiungere il metodo seguente alla classe **Program** :

    ```csharp
    public static async Task StartMethodJob(string jobId)
    {
        CloudToDeviceMethod directMethod = 
          new CloudToDeviceMethod("LockDoor", TimeSpan.FromSeconds(5), 
          TimeSpan.FromSeconds(5));

        JobResponse result = await jobClient.ScheduleDeviceMethodAsync(jobId,
            $"DeviceId IN ['{deviceId}']",
            directMethod,
            DateTime.UtcNow,
            (long)TimeSpan.FromMinutes(2).TotalSeconds);

        Console.WriteLine("Started Method Job");
    }
    ```

1. Aggiungere un altro metodo alla classe **Program**:

    ```csharp
    public static async Task StartTwinUpdateJob(string jobId)
    {
        Twin twin = new Twin(deviceId);
        twin.Tags = new TwinCollection();
        twin.Tags["Building"] = "43";
        twin.Tags["Floor"] = "3";
        twin.ETag = "*";

        twin.Properties.Desired["LocationUpdate"] = DateTime.UtcNow;

        JobResponse createJobResponse = jobClient.ScheduleTwinUpdateAsync(
            jobId,
            $"DeviceId IN ['{deviceId}']", 
            twin, 
            DateTime.UtcNow, 
            (long)TimeSpan.FromMinutes(2).TotalSeconds).Result;

        Console.WriteLine("Started Twin Update Job");
    }
    ```

    > [!NOTE]
    > Per altre informazioni sulla sintassi della query, vedere [Linguaggio di query di hub IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-query-language).
    >

1. Aggiungere infine le righe seguenti al metodo **Main** :

    ```csharp
    Console.WriteLine("Press ENTER to start running jobs.");
    Console.ReadLine();

    jobClient = JobClient.CreateFromConnectionString(connString);

    string methodJobId = Guid.NewGuid().ToString();

    StartMethodJob(methodJobId);
    MonitorJob(methodJobId).Wait();
    Console.WriteLine("Press ENTER to run the next job.");
    Console.ReadLine();

    string twinUpdateJobId = Guid.NewGuid().ToString();

    StartTwinUpdateJob(twinUpdateJobId);
    MonitorJob(twinUpdateJobId).Wait();
    Console.WriteLine("Press ENTER to exit.");
    Console.ReadLine();
    ```

1. Salvare il lavoro e compilare la soluzione.

## <a name="run-the-apps"></a>Eseguire le app

A questo punto è possibile eseguire le app.

1. Nella Esplora soluzioni di Visual Studio fare clic con il pulsante destro del mouse sulla soluzione e scegliere **Imposta progetti di avvio**.

1. Selezionare**progetto di avvio** **Proprietà** > comuni, quindi selezionare **progetti di avvio multipli**.

1. Assicurarsi che `SimulateDeviceMethods` si trovi all'inizio dell'elenco, seguito da `ScheduleJob`. Impostare entrambe le azioni su **Avvia** e selezionare **OK**.

1. Eseguire i progetti facendo clic su **Avvia** o passare al menu **Debug** e scegliere **Avvia debug**.

   L'output viene visualizzato sia dal dispositivo che dalle app back-end.

    ![Eseguire le app per pianificare i processi](./media/iot-hub-csharp-csharp-schedule-jobs/schedule-jobs-console-results.png)

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione è stato usato un processo per pianificare un metodo diretto in un dispositivo e aggiornare le proprietà di un dispositivo gemello.

* Per continuare a usare i modelli di gestione di hub e dispositivi, ad esempio Remote over the Air Firmware Update [, vedere l'esercitazione: Come eseguire un aggiornamento](tutorial-firmware-update.md)del firmware.

* Per altre informazioni sulla distribuzione dell'intelligenza artificiale in dispositivi perimetrali con Azure IoT Edge, vedere la [Guida introduttiva di IoT Edge](../iot-edge/tutorial-simulate-device-linux.md).
