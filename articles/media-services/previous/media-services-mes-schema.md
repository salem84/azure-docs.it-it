---
title: Schema di Media Encoder Standard | Microsoft Docs
description: Questo articolo fornisce una panoramica dello schema di Media Encoder Standard.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 837235e04ce190a4481e1f19789d8e9ff9cb7578
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "61131588"
---
# <a name="media-encoder-standard-schema"></a>Schema di Media Encoder Standard
Questo articolo descrive alcuni elementi e tipi di schema XML su cui si basa il [set di impostazioni di Media Encoder Standard](media-services-mes-presets-overview.md). L'articolo spiega gli elementi e i valori possibili per ognuno.  

## <a name="Preset"></a> Set di impostazioni (elemento radice)
Definisce un set di impostazioni per la codifica.  

### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Encoding** |[Encoding](media-services-mes-schema.md#Encoding) |Elemento radice, indica che le origini dell'input devono essere codificate. |
| **Outputs** |[Outputs](media-services-mes-schema.md#Output) |Raccolta dei file dell'output desiderato. |
| **StretchMode**<br/>minOccurs="0"<br/>default="AutoSize|xs:string|Controlla la dimensione dei fotogrammi, la spaziatura interna, le proporzioni dei pixel o quelle di visualizzazione del video di output. **StretchMode** può corrispondere a uno dei valori seguenti: **None**, **AutoSize** (impostazione predefinita) o **AutoFit**.<br/><br/>**Nessuna**: segue rigorosamente la risoluzione dell'output (ad esempio, i valori di **Width** e **Height** nel set di impostazioni) senza considerare le proporzioni dei pixel o quelle di visualizzazione del video di input. Questo valore è consigliato per gli scenari, ad esempio il [ritaglio](media-services-crop-video.md), in cui il video di output ha proporzioni diverse rispetto all'input. <br/><br/>**AutoSize**: la risoluzione dell'output viene adattata alle dimensioni della finestra (Width * Height) specificate dal set di impostazioni. Il codificatore genera tuttavia un video di output con proporzioni pixel quadrate (1:1). È pertanto possibile che la larghezza o l'altezza di output venga sostituita in modo da stabilire una corrispondenza con le proporzioni di visualizzazione dell'input, senza spaziatura interna. Se ad esempio l'input è 1920x1080 e il set di impostazioni di codifica richiede 1280x1280, il valore dell'altezza nel set di impostazioni verrà sostituito e l'output sarà 1280x720, in modo da mantenere le proporzioni di 16:9 dell'input. <br/><br/>**AutoFit**: se necessario, nel video di output viene aggiunta la spaziatura interna (nel formato 16:9 o 4:3) per rispettare la risoluzione di output desiderata, assicurando che l'area attiva del video nell'output abbia le stesse proporzioni dell'input. Si supponga, ad esempio, che l'input abbia una risoluzione di 1920x1080 e che il set di impostazioni di codifica richieda invece 1280x1280. Il video di output avrà una risoluzione pari a 1280x1280, ma conterrà un rettangolo di 1280x720 per il video attivo con proporzioni di 16:9 e, nella parte superiore e inferiore, aree in formato 16:9 con 280 pixel di altezza. Sempre a titolo di esempio, se l'input ha una risoluzione di 1440x1080 e il set di impostazioni di codifica richiede invece 1280x720, l'output avrà una risoluzione pari a 1280x720 e conterrà un rettangolo interno di 960x720 con proporzioni di 4:3 e, a sinistra e destra, aree in formato 4:3 con 160 pixel di larghezza. 

### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Versione**<br/><br/> Obbligatorio |**xs: decimal** |Versione predefinita. Valgono le limitazioni seguenti: xs:fractionDigits value="1" e xs:minInclusive value="1" Ad esempio, **version="1.0"** . |

## <a name="Encoding"></a> Encoding
Contiene una sequenza degli elementi seguenti:  

### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **H264Video** |[H264Video](media-services-mes-schema.md#H264Video) |Impostazioni per la codifica video H.264. |
| **AACAudio** |[AACAudio](media-services-mes-schema.md#AACAudio) |Impostazioni per la codifica audio AAC. |
| **BmpImage** |[BmpImage](media-services-mes-schema.md#BmpImage) |Impostazioni per le immagini .bmp. |
| **PngImage** |[PngImage](media-services-mes-schema.md#PngImage) |Impostazioni per le immagini .png. |
| **JpgImage** |[JpgImage](media-services-mes-schema.md#JpgImage) |Impostazioni per le immagini .jpg. |

## <a name="H264Video"></a> H264Video
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **TwoPass**<br/><br/> minOccurs="0" |**xs:boolean** |Al momento è supportata solo la codifica in un passaggio. |
| **KeyFrameInterval**<br/><br/> minOccurs="0"<br/><br/> **default="00:00:02"** |**xs:time** |Determina la spaziatura fissa tra i frame IDR in unità di secondi. Chiamata anche la durata GOP. Per controllare se il codificatore può deviare da questo valore, vedere **SceneChangeDetection**. |
| **SceneChangeDetection**<br/><br/> minOccurs="0"<br/><br/> default="false" |**xs: boolean** |Se impostato su true, il codificatore tenta di rilevare i cambi scena nel video e inserisce un frame IDR. |
| **Complexity**<br/><br/> minOccurs="0"<br/><br/> default="Balanced" |**xs:string** |Consente di controllare il compromesso tra qualità video e velocità di codifica. Può avere uno dei valori seguenti: **Speed**, **Balanced** o **Quality**<br/><br/> Predefinito: **Balanced** |
| **SyncMode**<br/><br/> minOccurs="0" | |La funzionalità sarà disponibile in una delle prossime versioni. |
| **H264Layers**<br/><br/> minOccurs="0" |[H264Layers](media-services-mes-schema.md#H264Layers) |Raccolta di livelli video di output. |

### <a name="attributes"></a>Attributi

| NOME | Type | Descrizione |
| --- | --- | --- |
| **Condition** |**xs:string** | Quando l'input non ha video, è consigliabile forzare il codificatore per inserire una traccia video monocromatica. A tale scopo, usare Condition="InsertBlackIfNoVideoBottomLayerOnly" (per inserire un video solo alla più bassa velocità in bit) o Condition="InsertBlackIfNoVideo" (per inserire un video a tutte le velocità in bit di output). Per altre informazioni, vedere [questo](media-services-advanced-encoding-with-mes.md#no_video) articolo.|

## <a name="H264Layers"></a> H264Layers

Per impostazione predefinita, se si invia al codificatore un input che contiene solo audio e nessun video, l'asset di output conterrà file di soli dati audio. Alcuni lettori non possono gestire flussi di output di questo tipo. È possibile usare l'impostazione dell'attributo **InsertBlackIfNoVideo** di H264Video per forzare l'aggiunta di una traccia video all'output da parte del codificatore. Per altre informazioni, vedere [questo](media-services-advanced-encoding-with-mes.md#no_video) articolo.
              
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **H264Layer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[H264Layer](media-services-mes-schema.md#H264Layer) |Raccolta di livelli H264. |

## <a name="H264Layer"></a> H264Layer
> [!NOTE]
> I limiti video sono basati sui valori descritti nella tabella [Livelli H264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Levels).  
> 
> 

### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Profilo**<br/><br/> minOccurs="0"<br/><br/> default="Auto" |**xs: string** |Può corrispondere a uno dei valori **xs:string** seguenti: **Auto**, **Baseline**, **Main** o **High**. |
| **Level**<br/><br/> minOccurs="0"<br/><br/> default="Auto" |**xs: string** | |
| **Bitrate**<br/><br/> minOccurs="0" |**xs:int** |La velocità in bit usata per questo livello video, espressa in kbps. |
| **MaxBitrate**<br/><br/> minOccurs="0" |**xs: int** |La velocità in bit massima usata per questo livello video, espressa in kbps. |
| **BufferWindow**<br/><br/> minOccurs="0"<br/><br/> default="00:00:05" |**xs: time** |Lunghezza del buffer video. |
| **Width**<br/><br/> minOccurs="0" |**xs: int** |Larghezza del frame video di output, espressa in pixel.<br/><br/> Al momento è necessario specificare entrambi i valori di larghezza e altezza, che devono essere numeri pari. |
| **Height**<br/><br/> minOccurs="0" |**xs:int** |Altezza del frame video di output, espressa in pixel.<br/><br/> Al momento è necessario specificare entrambi i valori di larghezza e altezza, che devono essere numeri pari.|
| **BFrames**<br/><br/> minOccurs="0" |**xs: int** |Numero di fotogrammi B tra frame di riferimento. |
| **ReferenceFrames**<br/><br/> minOccurs="0"<br/><br/> default=”3” |**xs:int** |Numero di frame di riferimento in un GOP. |
| **EntropyMode**<br/><br/> minOccurs="0"<br/><br/> default="Cabac" |**xs: string** |Può avere uno dei valori seguenti: **Cabac** e **Cavlc**. |
| **FrameRate**<br/><br/> minOccurs="0" |numero razionale |Determina la frequenza dei fotogrammi del video di output. Usare il valore predefinito "0/1" per consentire al codificatore di usare la stessa frequenza dei fotogrammi del video di input. I valori consentiti sono in genere le normali frequenze dei fotogrammi video. Tuttavia, è consentito qualunque numero razionale. Ad esempio 1/1 corrisponderebbe a 1 fps e sarebbe valido.<br/><br/> - 12/1 (12 fps)<br/><br/> - 15/1 (15 fps)<br/><br/> - 24/1 (24 fps)<br/><br/> - 24.000/1.001 (23,976 fps)<br/><br/> - 25/1 (25 fps)<br/><br/>  - 30/1 (30 fps)<br/><br/> - 30.000/1.001 (29,97 fps) <br/> <br/>**NOTA** se si sta creando un set di impostazioni personalizzato per la codifica a bitrate multipli, tutti i livelli del set di impostazioni **devono** usare lo stesso valore di FrameRate.|
| **AdaptiveBFrame**<br/><br/> minOccurs="0" |**xs: boolean** |Copiare da Azure Media Encoder |
| **Slices**<br/><br/> minOccurs="0"<br/><br/> default="0" |**xs:int** |Stabilisce il numero di sezioni in cui è suddiviso un frame. È consigliabile usare il valore predefinito. |

## <a name="AACAudio"></a> AACAudio
 Contiene una sequenza degli elementi e dei gruppi seguenti.  

 Per altre informazioni su AAC, vedere [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding).  

### <a name="elements"></a>Elementi

| NOME | Type | Descrizione |
| --- | --- | --- |
| **Profilo**<br/><br/> minOccurs="0 "<br/><br/> default="AACLC" |**xs: string** |Può avere uno dei valori seguenti: **AACLC**, **HEAACV1** o **HEAACV2**. |

### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Condition** |**xs: string** |Per forzare la generazione di un asset contenente una traccia audio silenziosa da parte del codificatore quando l'input è privo di audio, specificare il valore "InsertSilenceIfNoAudio".<br/><br/> Per impostazione predefinita, se si invia al codificatore un input che contiene solo video e nessun audio, l'asset di output contiene file di soli dati video. Alcuni lettori non possono gestire flussi di output di questo tipo. In tal caso, è possibile usare questa impostazione per forzare l'aggiunta di una traccia audio silenziosa all'output da parte del codificatore. |

### <a name="groups"></a>Gruppi

| Riferimenti | Descrizione |
| --- | --- |
| [AudioGroup](media-services-mes-schema.md#AudioGroup)<br/><br/> minOccurs="0" |Vedere la descrizione di [AudioGroup](media-services-mes-schema.md#AudioGroup) per conoscere il numero di canali, la frequenza di campionamento e la velocità in bit appropriati che è possibile impostare per ogni profilo. |

## <a name="AudioGroup"></a> AudioGroup
Per altre informazioni sui valori validi per ogni profilo, vedere la tabella "Dettagli sui codec audio" qui di seguito.  

### <a name="elements"></a>Elementi

| NOME | Type | Descrizione |
| --- | --- | --- |
| **Channels**<br/><br/> minOccurs="0" |**xs: int** |Numero dei canali audio codificati. Le opzioni valide sono: 1, 2, 5, 6, 8.<br/><br/> Predefinito: 2. |
| **SamplingRate**<br/><br/> minOccurs="0" |**xs: int** |Frequenza di campionamento audio, espressa in Hz. |
| **Bitrate**<br/><br/> minOccurs="0" |**xs: int** |Velocità in bit usata per la codifica audio, specificata in kbps. |

### <a name="audio-codec-details"></a>Dettagli sui codec audio

Codec audio|Dettagli  
-----------------|---  
**AACLC** |1:<br/><br/> - 11025: 8 &lt;= velocità in bit &lt; 16<br/><br/> - 12000: 8 &lt;= velocità in bit &lt; 16<br/><br/> - 16000: 8 &lt;= velocità in bit &lt;32<br/><br/>- 22050: 24 &lt;= velocità in bit &lt; 32<br/><br/> - 24000: 24 &lt;= velocità in bit &lt; 32<br/><br/> - 32000: 32 &lt;= velocità in bit &lt;= 192<br/><br/> - 44100: 56 &lt;= velocità in bit &lt;= 288<br/><br/> - 48000: 56 &lt;= velocità in bit &lt;= 288<br/><br/> - 88200: 128 &lt;= velocità in bit &lt;= 288<br/><br/> - 96000: 128 &lt;= velocità in bit &lt;= 288<br/><br/> 2:<br/><br/> - 11025: 16 &lt;= velocità in bit &lt; 24<br/><br/> - 12000: 16 &lt;= velocità in bit &lt; 24<br/><br/> - 16000: 16 &lt;= velocità in bit &lt; 40<br/><br/> - 22050: 32 &lt;= velocità in bit &lt; 40<br/><br/> - 24000: 32 &lt;= velocità in bit &lt; 40<br/><br/> - 32000:  40 &lt;= velocità in bit &lt;= 384<br/><br/> - 44100: 96 &lt;= velocità in bit &lt;= 576<br/><br/> - 48000: 96 &lt;= velocità in bit &lt;= 576<br/><br/> - 88200: 256 &lt;= velocità in bit &lt;= 576<br/><br/> - 96000: 256 &lt;= velocità in bit &lt;= 576<br/><br/> 5/6:<br/><br/> - 32000: 160 &lt;= velocità in bit &lt;= 896<br/><br/> - 44100: 240 &lt;= velocità in bit &lt;= 1024<br/><br/> - 48000: 240 &lt;= velocità in bit &lt;= 1024<br/><br/> - 88200: 640 &lt;= velocità in bit &lt;= 1024<br/><br/> - 96000: 640 &lt;= velocità in bit &lt;= 1024<br/><br/> 8:<br/><br/> - 32000: 224 &lt;= velocità in bit &lt;= 1024<br/><br/> - 44100: 384 &lt;= velocità in bit &lt;= 1024<br/><br/> - 48000: 384 &lt;= velocità in bit &lt;= 1024<br/><br/> - 88200: 896 &lt;= velocità in bit &lt;= 1024<br/><br/> - 96000: 896 &lt;= velocità in bit &lt;= 1024  
**HEAACV1** |1:<br/><br/> - 22050: bitrate = 8<br/><br/> - 24000: 8 &lt;= velocità in bit &lt;= 10<br/><br/> - 32000: 12 &lt;= velocità in bit &lt;= 64<br/><br/> - 44100: 20 &lt;= velocità in bit &lt;= 64<br/><br/> - 48000: 20 &lt;= velocità in bit &lt;= 64<br/><br/> - 88200: bitrate = 64<br/><br/> 2:<br/><br/> - 32000: 16 &lt;= velocità in bit &lt;= 128<br/><br/> - 44100: 16 &lt;= velocità in bit &lt;= 128<br/><br/> - 48000: 16 &lt;= velocità in bit &lt;= 128<br/><br/> - 88200: 96 &lt;= velocità in bit &lt;= 128<br/><br/> - 96000: 96 &lt;= velocità in bit &lt;= 128<br/><br/> 5/6:<br/><br/> - 32000: - 64 &lt;= velocità in bit &lt;= 320<br/><br/> - 44100: - 64 &lt;= velocità in bit &lt;= 320<br/><br/> - 48000: - 64 &lt;= velocità in bit &lt;= 320<br/><br/> - 88200: 256 &lt;= velocità in bit &lt;= 320<br/><br/> - 96000: 256 &lt;= velocità in bit &lt;= 320<br/><br/> 8:<br/><br/> - 32000: 96 &lt;= velocità in bit &lt;= 448<br/><br/> - 44100: 96 &lt;= velocità in bit &lt;= 448<br/><br/> - 48000: 96 &lt;= velocità in bit &lt;= 448<br/><br/> - 88200: 384 &lt;= velocità in bit &lt;= 448<br/><br/> - 96000: 384 &lt;= velocità in bit &lt;= 448  
**HEAACV2** |2:<br/><br/> - 22050: 8 &lt;= velocità in bit &lt;= 10<br/><br/> - 24000: 8 &lt;= velocità in bit &lt;= 10<br/><br/> - 32000: 12 &lt;= velocità in bit &lt;= 64<br/><br/> - 44100: 20 &lt;= velocità in bit &lt;= 64<br/><br/> - 48000: 20 &lt;= velocità in bit &lt;= 64<br/><br/> - 88200: 64 &lt;= velocità in bit &lt;= 64  
  
## <a name="Clip"></a> Clip
### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **StartTime** |**xs:duration** |Specifica l'ora di inizio di una presentazione. Il valore di StartTime deve corrispondere ai timestamp assoluti del video di input. Ad esempio, se il primo fotogramma del video di input ha un timestamp di 12:00:10.000, il valore di StartTime deve essere di almeno 12:00:10.000 o superiore. |
| **Duration** |**xs:duration** |Specifica la durata di una presentazione (ad esempio, l'aspetto di una sovrimpressione nel video). |

## <a name="Output"></a> Output
### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **FileName** |**xs:string** |Nome del file di output.<br/><br/> È possibile usare le macro descritte nella tabella seguente per creare i nomi dei file di output. Ad esempio:<br/><br/> **"Outputs": [      {       "FileName": "{Basename} *{Resolution}* {Bitrate}.mp4",       "Format": {         "Type": "MP4Format"       }     }   ]** |

### <a name="macros"></a>Macro

| Macro | Descrizione |
| --- | --- |
| **{Basename}** |Se si esegue la codifica VoD, {Basename} corrisponde ai primi 32 caratteri della proprietà AssetFile.Name del file primario dell'asset di input.<br/><br/> Se l'asset di input è un archivio live, {Basename} è derivato dagli attributi trackName nel manifesto server. Se si invia un processo subclip usando TopBitrate, come in "<VideoStream\>TopBitrate</VideoStream\>", e il file di output contiene video, {Basename} corrisponde ai primi 32 caratteri dell'attributo trackName del livello video con il valore di velocità in bit più alto.<br/><br/> Se invece si invia un processo subclip usando tutte le velocità in bit dell'input, come in "<VideoStream\>*</VideoStream\>", e il file di output contiene video, {Basename} corrisponde ai primi 32 caratteri dell'attributo trackName del livello video corrispondente. |
| **{Codec}** |Esegue il mapping a "H264" per i video e "AAC" per l'audio. |
| **{Bitrate}** |Velocità in bit video di destinazione se il file di output contiene video e audio oppure velocità in bit audio di destinazione se il file di output contiene solo l'audio. Il valore usato è la velocità in bit espressa in kbps. |
| **{Channel}** |Numero di canali audio se il file contiene audio. |
| **{Width}** |Larghezza del video nel file di output, espressa in pixel, se il file contiene video. |
| **{Height}** |Altezza del video nel file di output, espressa in pixel, se il file contiene video. |
| **{Extension}** |Eredita dalla proprietà "Type" per il file di output. Il nome del file di output ha una delle estensioni seguenti: "mp4", "ts", "jpg", "png" o "bmp". |
| **{Index}** |Obbligatorio per l'anteprima. Deve essere presente solo una volta. |

## <a name="Video"></a> Video (il tipo complesso eredita da Codec)
### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Inizia** |**xs:string** | |
| **Step** |**xs:string** | |
| **Range** |**xs:string** | |
| **PreserveResolutionAfterRotation** |**xs:boolean** |Per informazioni dettagliate, vedere la sezione seguente: [PreserveResolutionAfterRotation](media-services-mes-schema.md#PreserveResolutionAfterRotation) |

### <a name="PreserveResolutionAfterRotation"></a> PreserveResolutionAfterRotation
È consigliabile usare il flag **PreserveResolutionAfterRotation** insieme ai valori di risoluzione espressi in termini di percentuale (Width="100%" , Height="100%").  

Per impostazione predefinita, le impostazioni di risoluzione della codifica (Width, Height) nel set di impostazioni di Media Encoder Standard (MES) sono destinate ai video con zero gradi di rotazione. Ad esempio, se il video di input è 1.280x720 con zero gradi di rotazione, i set di impostazioni assicurano che l'output avrà la stessa risoluzione.  

![MESRoation1](./media/media-services-shemas/media-services-mes-roation1.png) 

Se il video di input è stato acquisito con rotazione diversa da zero (ad esempio nel caso di un tablet o uno smartphone tenuto verticalmente), per impostazione predefinita MES applica al video di input le impostazioni di risoluzione della codifica (Width, Height), compensando quindi la rotazione. Vedere l'esempio nell'immagine di seguito. Il set di impostazioni usa Width = "100%", Height = "100%". In questo modo, MES interpreta la necessità di un output di 1280 pixel di larghezza e 720 pixel di altezza. Una volta ruotato il video, l'immagine viene ridotta alle dimensioni della finestra, creando colonne a sinistra e a destra.  

![MESRoation2](./media/media-services-shemas/media-services-mes-roation2.png) 

In alternativa, è possibile usare il flag **PreserveResolutionAfterRotation** e impostarlo su "true" (il valore predefinito è "false"). Se quindi il set di impostazioni ha come valori Width = "100%", Height = "100%" e PreserveResolutionAfterRotation è impostato su "true", un video di input da 1280 pixel di larghezza e 720 pixel di altezza con 90 gradi di rotazione produce un output con zero gradi di rotazione, ma 720 pixel di larghezza e 1280 pixel di altezza. Vedere l'immagine seguente:  

![MESRoation3](./media/media-services-shemas/media-services-mes-roation3.png) 

## <a name="FormatGroup"></a> FormatGroup (gruppo)
### <a name="elements"></a>Elementi

| NOME | Type | Descrizione |
| --- | --- | --- |
| **BmpFormat** |**BmpFormat** | |
| **PngFormat** |**PngFormat** | |
| **JpgFormat** |**JpgFormat** | |

## <a name="BmpLayer"></a> BmpLayer
### <a name="element"></a>Elemento

| Name | Type | Descrizione |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="PngLayer"></a> PngLayer
### <a name="element"></a>Elemento

| NOME | Type | Descrizione |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>Attributi

| NOME | Type | Descrizione |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="JpgLayer"></a> JpgLayer
### <a name="element"></a>Elemento

| Name | Type | Descrizione |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |
| **Quality**<br/><br/> minOccurs="0" |**xs:int** |Valori validi:  1 (peggiore) -100 (migliore) |

### <a name="attributes"></a>Attributi

| Name | Type | Descrizione |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="PngLayers"></a> PngLayers
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **PngLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[PngLayer](media-services-mes-schema.md#PngLayer) | |

## <a name="BmpLayers"></a> BmpLayers
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **BmpLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[BmpLayer](media-services-mes-schema.md#BmpLayer) | |

## <a name="JpgLayers"></a> JpgLayers
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **JpgLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[JpgLayer](media-services-mes-schema.md#JpgLayer) | |

## <a name="BmpImage"></a> BmpImage (il tipo complesso eredita da Video)
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Png layers |

## <a name="JpgImage"></a> JpgImage (il tipo complesso eredita da Video)
### <a name="elements"></a>Elementi

| Name | Type | Descrizione |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Png layers |

## <a name="PngImage"></a> PngImage (il tipo complesso eredita da Video)
### <a name="elements"></a>Elementi

| NOME | Type | Descrizione |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Png layers |

## <a name="examples"></a>Esempi
Per osservare esempi di set di impostazioni XML compilati in base a questo schema, vedere [Task Presets for MES (Media Encoder Standard)](media-services-mes-presets-overview.md) (Set di impostazioni delle attività di MES (Media Encoder Standard)).

## <a name="next-steps"></a>Passaggi successivi
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fornire commenti e suggerimenti
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

