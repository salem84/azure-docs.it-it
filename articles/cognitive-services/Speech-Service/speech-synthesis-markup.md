---
title: Speech Synthesis Markup Language (SSML)-servizio di riconoscimento vocale
titleSuffix: Azure Cognitive Services
description: Uso di Speech Synthesis Markup Language per controllare la pronuncia e la prosodia nella sintesi vocale.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: erhopf
ms.openlocfilehash: 3791b2d60b84299fc3b646f7e6585002078b607f
ms.sourcegitcommit: 7f6d986a60eff2c170172bd8bcb834302bb41f71
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/27/2019
ms.locfileid: "71350158"
---
# <a name="speech-synthesis-markup-language-ssml"></a>Speech Synthesis Markup Language (SSML)

Speech Synthesis Markup Language (SSML) è un linguaggio di markup basato su XML che consente agli sviluppatori di specificare il modo in cui il testo di input viene convertito in sintesi vocale usando il servizio di sintesi vocale. Rispetto a testo normale, SSML consente agli sviluppatori di ottimizzare il pitch, la pronuncia, la velocità di pronuncia, il volume e altro ancora dell'output da sintesi vocale. La punteggiatura normale, ad esempio la sospensione dopo un periodo, o l'uso dell'intonazione corretta quando una frase termina con un punto interrogativo viene gestita automaticamente.

L'implementazione di SSML per i servizi vocali è basata sulla [versione 1,0 del linguaggio di markup sintesi vocale](https://www.w3.org/TR/speech-synthesis)di World Wide Web Consortium.

> [!IMPORTANT]
> I caratteri cinesi, giapponesi e coreani sono considerati come due caratteri per la fatturazione. Per altre informazioni, vedere [Prezzi](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

## <a name="standard-neural-and-custom-voices"></a>Voci standard, Neural e Custom

È possibile scegliere tra le voci standard e neurali oppure creare una voce personalizzata univoca per il prodotto o il marchio. 75 e le voci standard sono disponibili in più di 45 lingue e impostazioni locali e 5 voci neurali sono disponibili in 4 lingue e impostazioni locali. Per un elenco completo delle lingue supportate, delle impostazioni locali e delle voci (neurale e standard), vedere [Supporto per le lingue](language-support.md).

Per altre informazioni sulle voci standard, neurale e personalizzate, vedere [Cenni preliminari](text-to-speech.md)sulla sintesi vocale.

## <a name="special-characters"></a>Caratteri speciali

Quando si usa SSML per convertire il riconoscimento vocale da testo a sintetizzato, tenere presente che, esattamente come con XML, i caratteri speciali, ad esempio virgolette, apostrofi e parentesi quadre, devono essere preceduti da un carattere di escape. Per ulteriori informazioni, vedere [Extensible Markup Language (XML) 1,0: Appendice D @ no__t-0.

## <a name="supported-ssml-elements"></a>Elementi SSML supportati

Ogni documento SSML viene creato con elementi SSML (o tag). Questi elementi vengono usati per modificare pitch, prosodia, volume e molto altro. Le sezioni seguenti illustrano in dettaglio come viene usato ogni elemento e quando un elemento è obbligatorio o facoltativo.  

> [!IMPORTANT]
> Non dimenticare di usare i valori di attributo tra virgolette doppie. Gli standard per il formato XML valido e valido richiedono che i valori di attributo siano racchiusi tra virgolette doppie. Ad esempio, `<prosody volume="90">` è un elemento valido ben formato, ma `<prosody volume=90>` non lo è. SSML non è in grado di riconoscere valori di attributo che non sono racchiusi tra virgolette.

## <a name="create-an-ssml-document"></a>Creare un documento SSML

`speak`è l'elemento radice ed è **obbligatorio** per tutti i documenti SSML. L' `speak` elemento contiene informazioni importanti, ad esempio la versione, la lingua e la definizione del vocabolario di markup.

**Sintassi**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="string"></speak>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| versione | Indica la versione della specifica SSML usata per interpretare il markup del documento. La versione corrente è 1,0. | Richiesto |
| xml:lang | Specifica la lingua del documento radice. Il valore può contenere un codice di lingua minuscolo di due lettere (ad esempio, **en**) o il codice della lingua e il paese/area geografica maiuscolo (ad esempio, **en-US**). | Richiesto |
| xmlns | Specifica l'URI del documento che definisce il vocabolario di markup (i tipi di elemento e i nomi di attributo) del documento SSML. L'URI corrente è https://www.w3.org/2001/10/synthesis. | Richiesto |

## <a name="choose-a-voice-for-text-to-speech"></a>Scegliere una voce per la sintesi vocale

L' `voice` elemento è obbligatorio. Viene usato per specificare la voce usata per la sintesi vocale.

**Sintassi**

```xml
<voice name="string">
    This text will get converted into synthesized speech.
</voice>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| name | Identifica la voce utilizzata per l'output da sintesi vocale. Per un elenco completo delle voci supportate, vedere [supporto](language-support.md#text-to-speech)per le lingue. | Richiesto |

**Esempio**

> [!NOTE]
> In questo esempio viene `en-US-Jessa24kRUS` usata la voce. Per un elenco completo delle voci supportate, vedere [supporto](language-support.md#text-to-speech)per le lingue.

```XML
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        This is the text that is spoken.
    </voice>
</speak>
```

## <a name="use-multiple-voices"></a>Usare più voci

All'interno `speak` dell'elemento è possibile specificare più voci per l'output da sintesi vocale. Queste voci possono essere in lingue diverse. Per ogni voce, il testo deve essere racchiuso in `voice` un elemento.

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| name | Identifica la voce utilizzata per l'output da sintesi vocale. Per un elenco completo delle voci supportate, vedere [supporto](language-support.md#text-to-speech)per le lingue. | Richiesto |

**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        Good morning!
    </voice>
    <voice  name="en-US-Guy24kRUS">
        Good morning to you too Jessa!
    </voice>
</speak>
```

## <a name="adjust-speaking-styles"></a>Modificare gli stili di pronuncia

> [!IMPORTANT]
> Questa funzionalità funziona solo con le voci neurali.

Per impostazione predefinita, il servizio Text-to-Speech sintetizza il testo usando uno stile di pronuncia neutro per le voci standard e neurali. Con le voci neurali, è possibile modificare lo stile di pronuncia per esprimere allegria, empatia o sentimento `<mstts:express-as>` con l'elemento. Si tratta di un elemento facoltativo univoco per i servizi di riconoscimento vocale di Azure.

Attualmente sono supportate le rettifiche di stile per le voci neurali seguenti:
* `en-US-JessaNeural`
* `zh-CN-XiaoxiaoNeural`

Le modifiche vengono applicate a livello di frase e lo stile varia in base alla voce. Se uno stile non è supportato, il servizio restituirà il riconoscimento vocale nello stile di lingua predefinito.

**Sintassi**

```xml
<mstts:express-as type="string"></mstts:express-as>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| type | Specifica lo stile di pronuncia. Attualmente, gli stili di pronuncia sono specifici della voce. | Obbligatorio se si modifica lo stile di pronuncia per una voce neurale. Se si `mstts:express-as`USA, è necessario fornire il tipo. Se viene specificato un valore non valido, questo elemento verrà ignorato. |

Usare questa tabella per determinare quali stili di pronuncia sono supportati per ogni voce neurale.

| Chiamata vocale | Type | Descrizione |
|-------|------|-------------|
| `en-US-JessaNeural` | tipo =`cheerful` | Esprime un'emozione positiva e felice |
| | tipo =`empathy` | Esprime un senso di attenzione e comprensione |
| | tipo =`chat` | Pronunciare un tono informale e rilassato |
| `zh-CN-XiaoxiaoNeural` | tipo =`newscast` | Esprime un tono formale, simile alle trasmissioni di notizie |
| | tipo =`sentiment` | Trasmette un messaggio di contatto o una storia |

**Esempio**

Questo frammento di SSML illustra come `<mstts:express-as>` viene usato l'elemento per modificare lo stile di `cheerful`pronuncia in.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
    <voice name="en-US-JessaNeural">
        <mstts:express-as type="cheerful">
            That'd be just amazing!
        </mstts:express-as>
    </voice>
</speak>
```

## <a name="add-or-remove-a-breakpause"></a>Aggiungere o rimuovere un'interruzione/pausa

Usare l' `break` elemento per inserire pause (o interruzioni) tra parole oppure impedire le pause aggiunte automaticamente dal servizio di sintesi vocale.

> [!NOTE]
> Usare questo elemento per eseguire l'override del comportamento predefinito di sintesi vocale (TTS) per una parola o una frase se la voce sintetizzata per la parola o la frase suona non naturale. Impostare `strength` su`none` per evitare un'interruzioni prosodica, inserita automaticamente dal servizio di sintesi vocale.

**Sintassi**

```xml
<break strength="string" />
<break time="string" />
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| forza | Specifica la durata relativa di una pausa utilizzando uno dei valori seguenti:<ul><li>nessuno</li><li>x-debole</li><li>vulnerabile</li><li>media (impostazione predefinita)</li><li>complessa</li><li>x-forte</li></ul> | Facoltativi |
| time | Specifica la durata assoluta di una pausa in secondi o millisecondi. Esempi di valori validi sono 2S e 500 | Facoltativi |

| forza | Descrizione |
|----------|-------------|
| None oppure se non viene specificato alcun valore | 0 ms |
| x-debole | 250 ms |
| vulnerabile | 500 ms |
| media | 750 ms |
| complessa | 1000 ms |
| x-forte | 1250 MS |


**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        Welcome to Microsoft Cognitive Services <break time="100ms" /> Text-to-Speech API.
    </voice>
</speak>
```

## <a name="specify-paragraphs-and-sentences"></a>Specificare paragrafi e frasi

`p`gli `s` elementi e vengono usati per indicare rispettivamente i paragrafi e le frasi. In assenza di questi elementi, il servizio di sintesi vocale determina automaticamente la struttura del documento SSML.

L' `p` elemento può contenere testo e gli elementi seguenti: `audio`, `break` `phoneme`,, `prosody`, `say-as`, `sub`, `mstts:express-as`e `s`.

L' `s` elemento può contenere testo e gli elementi seguenti: `audio`, `break` `phoneme`,, `prosody`, `say-as`, `mstts:express-as`e `sub`.

**Sintassi**

```XML
<p></p>
<s></s>
```

**Esempio**

```XML
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        <p>
            <s>Introducing the sentence element.</s>
            <s>Used to mark individual sentences.</s>
        </p>
        <p>
            Another simple paragraph.
            Sentence structure in this paragraph is not explicitly marked.
        </p>
    </voice>
</speak>
```

## <a name="use-phonemes-to-improve-pronunciation"></a>Usare fonemi per migliorare la pronuncia

L' `ph` elemento viene usato per la pronuncia fonetica nei documenti SSML. L' `ph` elemento può contenere solo testo, nessun altro elemento. Fornire sempre un discorso leggibile come fallback.

Gli alfabeti fonetici sono costituiti da telefoni, che sono costituiti da lettere, numeri o caratteri, a volte in combinazione. Ogni telefono descrive un suono univoco di sintesi vocale. Questo si differenzia dall'alfabeto latino, dove qualsiasi lettera può rappresentare più suoni vocali. Prendere in considerazione le diverse pronunce della lettera "c" nelle parole "Candy" e "cessate" oppure le diverse pronunce della combinazione di lettere "th" nelle parole "Thing" e "that".

**Sintassi**

```XML
<phoneme alphabet="string" ph="string"></phoneme>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| alfabeto | Specifica l'alfabeto fonetico da usare quando si sintetizza la pronuncia della stringa nell' `ph` attributo. La stringa che specifica l'alfabeto deve essere specificata in lettere minuscole. Di seguito sono riportati gli alfabeti possibili che è possibile specificare.<ul><li>alfabeto &ndash; fonetico internazionale IPA</li><li>Set &ndash; di telefono Speech API SAPI</li><li>Set &ndash; di telefonia universale UPS</li></ul>L'alfabeto si applica solo al fonema nell'elemento. Per ulteriori informazioni, vedere [riferimento all'alfabeto fonetico](https://msdn.microsoft.com/library/hh362879(v=office.14).aspx). | Facoltativi |
| pH | Stringa contenente i telefoni che specificano la pronuncia della parola nell' `phoneme` elemento. Se la stringa specificata contiene telefoni non riconosciuti, il servizio di sintesi vocale rifiuta l'intero documento SSML e non genera alcun output vocale specificato nel documento. | Obbligatorio se si utilizzano fonemi. |

**esempi**

```XML
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        <s>His name is Mike <phoneme alphabet="ups" ph="JH AU"> Zhou </phoneme></s>
    </voice>
</speak>
```

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        <phoneme alphabet="ipa" ph="t&#x259;mei&#x325;&#x27E;ou&#x325;"> tomato </phoneme>
    </voice>
</speak>
```

## <a name="adjust-prosody"></a>Modificare prosodia

L' `prosody` elemento viene usato per specificare le modifiche a pitch, contorno, Range, rate, Duration e volume per l'output da testo a discorso. L' `prosody` elemento può contenere testo e gli elementi seguenti: `audio`, `break` `p`,, `phoneme`, `prosody`, `say-as`, `sub`e `s`.

Poiché i valori dell'attributo prosodica possono variare in base a un intervallo ampio, il riconoscimento vocale interpreta i valori assegnati come un suggerimento dei valori prosodica effettivi della voce selezionata. Il servizio di sintesi vocale limita o sostituisce valori non supportati. Esempi di valori non supportati sono un passo di 1 MHz o un volume di 120.

**Sintassi**

```XML
<prosody pitch="value" contour="value" range="value" rate="value" duration="value" volume="value"></prosody>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| passo | Indica il passo della linea di base per il testo. È possibile esprimere il passo come:<ul><li>Valore assoluto, espresso come numero seguito da "Hz" (hertz). Ad esempio, 600Hz.</li><li>Valore relativo, espresso come numero preceduto da "+" o "-", seguito da "Hz" o "St", che specifica una quantità per modificare il pitch. Ad esempio: + 80Hz o-2ST. Il valore "St" indica che l'unità di modifica è semitono, ovvero la metà di un tono (un mezzo) sulla scala diatonica standard.</li><li>Valore costante:<ul><li>x-basso</li><li>bassa</li><li>media</li><li>elevata</li><li>x-alto</li><li>predefinito</li></ul></li></ul>. | Facoltativi |
| contorno | Il contorno non è supportato per le voci neurali. Contour rappresenta le modifiche in pitch per il contenuto vocale come matrice di destinazioni in posizioni temporali specificate nell'output del riconoscimento vocale. Ogni destinazione è definita da insiemi di coppie di parametri. Esempio: <br/><br/>`<prosody contour="(0%,+20Hz) (10%,-2st) (40%,+10Hz)">`<br/><br/>Il primo valore di ogni set di parametri specifica la posizione della modifica del passo come percentuale della durata del testo. Il secondo valore specifica la quantità da elevare o abbassare il pitch, usando un valore relativo o un valore di enumerazione per pitch `pitch`(vedere). | Facoltativi |
| range  | Valore che rappresenta l'intervallo di pitch per il testo. È possibile esprimere `range` usando gli stessi valori assoluti, i valori relativi o i valori di enumerazione usati `pitch`per descrivere. | Facoltativi |
| tasso  | Indica la velocità di pronuncia del testo. È possibile esprimere `rate` le seguenti operazioni:<ul><li>Valore relativo, espresso come numero che funge da moltiplicatore del valore predefinito. Il valore *1* , ad esempio, non comporta alcuna modifica nella frequenza. Il valore *0,5* comporta una dimezzazione della frequenza. Il valore *3* comporta un triplo della frequenza.</li><li>Valore costante:<ul><li>x-lento</li><li>lento</li><li>media</li><li>Veloce</li><li>x-veloce</li><li>predefinito</li></ul></li></ul> | Facoltativi |
| durata  | Periodo di tempo che deve trascorrere mentre il servizio di sintesi vocale (TTS) legge il testo, in secondi o millisecondi. Ad esempio, *2S* o *1800ms*. | Facoltativi |
| volume  | Indica il livello del volume della voce di pronuncia. Il volume può essere espresso come segue:<ul><li>Valore assoluto, espresso come numero compreso nell'intervallo tra 0,0 e 100,0, dal più *silenzioso* al più *alto*. Ad esempio, 75. Il valore predefinito è 100,0.</li><li>Valore relativo, espresso come numero preceduto da "+" o "-", che specifica una quantità per modificare il volume. Ad esempio + 10 o-5,5.</li><li>Valore costante:<ul><li>silenzioso</li><li>x-soft</li><li>temporanea</li><li>media</li><li>forte</li><li>x-Loud</li><li>predefinito</li></ul></li></ul> | Facoltativi |

### <a name="change-speaking-rate"></a>Cambiare la velocità del parlato

La velocità di pronuncia può essere applicata a voci standard a livello di parola o di frase. Mentre la velocità di pronuncia può essere applicata solo a voci neurali a livello di frase.

**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Guy24kRUS">
        <prosody rate="+30.00%">
            Welcome to Microsoft Cognitive Services Text-to-Speech API.
        </prosody>
    </voice>
</speak>
```

### <a name="change-volume"></a>Cambiare il volume

È possibile applicare le modifiche del volume alle voci standard a livello di parola o di frase. Mentre le modifiche al volume possono essere applicate solo a voci neurali a livello di frase.

**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        <prosody volume="+20.00%">
            Welcome to Microsoft Cognitive Services Text-to-Speech API.
        </prosody>
    </voice>
</speak>
```

### <a name="change-pitch"></a>Cambiare il tono

Le modifiche di pitch possono essere applicate alle voci standard a livello di parola o di frase. Mentre le modifiche di pitch possono essere applicate solo a voci neurali a livello di frase.

**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Guy24kRUS">
        Welcome to <prosody pitch="high">Microsoft Cognitive Services Text-to-Speech API.</prosody>
    </voice>
</speak>
```

### <a name="change-pitch-contour"></a>Cambiare il profilo di intonazione

> [!IMPORTANT]
> Le modifiche al contorno del passo non sono supportate con le voci neurali.

**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
        <prosody contour="(80%,+20%) (90%,+30%)" >
            Good morning.
        </prosody>
    </voice>
</speak>
```
## <a name="say-as-element"></a>elemento Say-As  

`say-as` è un elemento facoltativo che indica il tipo di contenuto (ad esempio il numero o la data) del testo dell'elemento. Vengono fornite informazioni aggiuntive sul motore di sintesi vocale su come pronunciare il testo. 

**Sintassi**

```XML
<say-as interpret-as="string" format="digit string" detail="string"> <say-as>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| interpreta come | Indica il tipo di contenuto del testo dell'elemento. Per un elenco di tipi, vedere la tabella seguente. | Richiesto |
| format | Fornisce informazioni aggiuntive sulla formattazione precisa del testo dell'elemento per i tipi di contenuto che possono avere formati ambigui. SSML definisce i formati per i tipi di contenuto che li usano (vedere la tabella riportata di seguito). | Facoltativi |
| dettagli | Indica il livello di dettaglio da pronunciare. Questo attributo, ad esempio, può richiedere che il motore di sintesi vocale pronunci segni di punteggiatura. Nessun valore standard definito per `detail`. | Facoltativi |

<!-- I don't understand the last sentence. Don't we know which one Cortana uses? -->

Di seguito sono riportati i tipi di contenuto supportati per gli attributi `interpret-as` e `format`. Includere l'attributo `format` solo se `interpret-as` è impostato su data e ora.

| interpreta come | format | Interpretazione |
|--------------|--------|----------------|
| indirizzo | | Il testo viene pronunciato come indirizzo. Il motore di sintesi vocale pronuncia:<br /><br />`I'm at <say-as interpret-as="address">150th CT NE, Redmond, WA</say-as>`<br /><br />"Sto a 150 Court North East Redmond Washington". |
| Cardinale, numero | | Il testo viene pronunciato come numero cardinale. Il motore di sintesi vocale pronuncia:<br /><br />`There are <say-as interpret-as="cardinal">3</say-as> alternatives`<br /><br />"Esistono tre alternative". |
| caratteri, ortografia | | Il testo viene pronunciato come singole lettere (digitato). Il motore di sintesi vocale pronuncia:<br /><br />`<say-as interpret-as="characters">test</say-as>`<br /><br />Come "T E S T". |
| date  | DMY, MDY, ymd, AGM, YM, My, MD, DM, d, m, y | Il testo viene pronunciato come data. L'attributo `format` specifica il formato della data (*d = giorno, m = mese e y = anno*). Il motore di sintesi vocale pronuncia:<br /><br />`Today is <say-as interpret-as="date" format="mdy">10-19-2016</say-as>`<br /><br />Come "oggi è il 19 ottobre 2016". |
| cifre, number_digit | | Il testo viene pronunciato come sequenza di singole cifre. Il motore di sintesi vocale pronuncia:<br /><br />`<say-as interpret-as="number_digit">123456789</say-as>`<br /><br />Come "1 2 3 4 5 6 7 8 9". |
| frazione | | Il testo viene pronunciato come numero frazionario. Il motore di sintesi vocale pronuncia:<br /><br /> `<say-as interpret-as="fraction">3/8</say-as> of an inch`<br /><br />"Tre ottavi di pollice". |
| ordinal  | | Il testo viene pronunciato come numero ordinale. Il motore di sintesi vocale pronuncia:<br /><br />`Select the <say-as interpret-as="ordinal">3rd</say-as> option`<br /><br />Come "selezionare la terza opzione". |
| telephone  | | Il testo viene pronunciato come numero di telefono. L'attributo `format` può contenere cifre che rappresentano un codice paese. Ad esempio, "1" per il Stati Uniti o "39" per l'Italia. Il motore di sintesi vocale può utilizzare queste informazioni per guidare la pronuncia di un numero di telefono. Il numero di telefono può includere anche il codice paese e, in tal caso, ha la precedenza sul codice paese nel `format`. Il motore di sintesi vocale pronuncia:<br /><br />`The number is <say-as interpret-as="telephone" format="1">(888) 555-1212</say-as>`<br /><br />"My Number is code 8 8 8 5 5 5 1 2 1 2". |
| time | hms12, hms24 | Il testo viene parlato come ora. L'attributo `format` specifica se l'ora viene specificata usando un formato a 12 ore (hms12) o un orologio a 24 ore (hms24). Usare i due punti per separare i numeri che rappresentano le ore, i minuti e i secondi. Gli esempi di tempo validi sono i seguenti: 12:35, 1:14:32, 08:15 e 02:50:45. Il motore di sintesi vocale pronuncia:<br /><br />`The train departs at <say-as interpret-as="time" format="hms12">4:00am</say-as>`<br /><br />Come "il treno si ripartisce da quattro a M". |

**Utilizzo**

L'elemento `say-as` può contenere solo testo.

**Esempio**

Il motore di sintesi vocale si riferisce all'esempio seguente: "la prima richiesta era per una stanza il 19 ottobre 20 10 con arrivo anticipato a 12 35 P M".
 
```XML
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice  name="en-US-Jessa24kRUS">
    <p>
    Your <say-as interpret-as="ordinal"> 1st </say-as> request was for <say-as interpret-as="cardinal"> 1 </say-as> room
    on <say-as interpret-as="date" format="mdy"> 10/19/2010 </say-as>, with early arrival at <say-as interpret-as="time" format="hms12"> 12:35pm </say-as>.
    </p>
</speak>
```


## <a name="add-recorded-audio"></a>Aggiungi audio registrato

`audio` è un elemento facoltativo che consente di inserire audio MP3 in un documento SSML. Il corpo dell'elemento audio può contenere testo normale o markup SSML che viene parlato se il file audio non è disponibile o non è riproducibile. Inoltre, l'elemento `audio` può contenere testo e gli elementi seguenti: `audio`, `break`, `p`, `s`, `phoneme`, `prosody`, `say-as` e `sub`.

Qualsiasi audio incluso nel documento SSML deve soddisfare i requisiti seguenti:

* Il formato MP3 deve essere ospitato in un endpoint HTTPS accessibile da Internet. HTTPS è obbligatorio e il dominio che ospita il file MP3 deve presentare un certificato SSL valido e attendibile.
* Il formato MP3 deve essere un file MP3 valido (MPEG v2).
* La velocità in bit deve essere di 48 kbps.
* La frequenza di campionamento deve essere 16000 Hz.
* Il tempo totale combinato per tutti i file di testo e audio in una singola risposta non può superare 90 (90) secondi.
* Il file MP3 non deve contenere informazioni riservate o specifiche del cliente.

**Sintassi**

```xml
<audio src="string"/></audio>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| src | Specifica il percorso o l'URL del file audio. | Obbligatorio se si usa l'elemento audio nel documento di SSML. |

**Esempio**

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <p>
        <audio src="https://contoso.com/opinionprompt.wav"/>
        Thanks for offering your opinion. Please begin speaking after the beep.
        <audio src="https://contoso.com/beep.wav">
        Could not play the beep, please voice your opinion now. </audio>
    </p>
</speak>
```

## <a name="add-background-audio"></a>Aggiungi audio in background

L'elemento `mstts:backgroundaudio` consente di aggiungere audio in background ai documenti SSML (o combinare un file audio con sintesi vocale). Con `mstts:backgroundaudio` è possibile eseguire il loop di un file audio in background, dissolversi all'inizio della sintesi vocale e dissolversi alla fine della sintesi vocale.

Se l'audio in background fornito è più breve della sintesi vocale o della dissolvenza, verrà eseguito il ciclo. Se è più lungo del testo da sintesi vocale, si interrompe al termine della dissolvenza.

Per ogni documento SSML è consentito un solo file audio di sfondo. Tuttavia, è possibile intercalare i tag `audio` all'interno dell'elemento `voice` per aggiungere altro audio al documento SSML.

**Sintassi**

```XML
<mstts:backgroundaudio src="string" volume="string" fadein="string" fadeout="string"/>
```

**Attributes (Attributi)**

| Attributo | Descrizione | Obbligatoria / Facoltativa |
|-----------|-------------|---------------------|
| src | Specifica il percorso o l'URL del file audio in background. | Obbligatorio se si usa l'audio in background nel documento di SSML. |
| volume | Specifica il volume del file audio in background. **Valori accettati**: da `0` a `100` inclusi. Il valore predefinito è `1`. | Facoltativi |
| fadein | Specifica la durata in millisecondi della dissolvenza audio in background. Il valore predefinito è `0`, che equivale a nessuna dissolvenza in. **Valori accettati**: da `0` a `10000` inclusi.  | Facoltativi |
| fadeout | Specifica la durata della dissolvenza dell'audio in background in millisecondi. Il valore predefinito è `0`, che equivale a nessuna dissolvenza in uscita. **Valori accettati**: da `0` a `10000` inclusi.  | Facoltativi |

**Esempio**

```xml
<speak version="1.0" xml:lang="en-US" xmlns:mstts="http://www.w3.org/2001/mstts">
    <mstts:backgroundaudio src="https://contoso.com/sample.wav" volume="0.7" fadein="3000" fadeout="4000"/>
    <voice name="Microsoft Server Speech Text to Speech Voice (en-US, Jessa24kRUS)">
        The text provided in this document will be spoken over the background audio.
    </voice>
</speak>
```

## <a name="next-steps"></a>Passaggi successivi

* [Supporto per le lingue: voci, impostazioni locali, lingue](language-support.md)
