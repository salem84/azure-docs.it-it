---
title: Formati e codec Media Encoder Standard - Azure
description: Questo argomento fornisce una panoramica dei formati e dei codec di Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.reviewer: anilmur
ms.openlocfilehash: 8fd68269f73506c2d605c0cb4560e6a57e1a2128
ms.sourcegitcommit: de47a27defce58b10ef998e8991a2294175d2098
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 07/15/2019
ms.locfileid: "69016529"
---
# <a name="media-encoder-standard-formats-and-codecs"></a>Formati e codec Media Encoder Standard

> [!div class="op_single_selector" title1="Selezionare la versione di servizi multimediali in uso:"]
> * [Versione 2](media-services-media-encoder-standard-formats.md)
> * [Versione 3](../latest/media-encoder-standard-formats.md)

Questo documento contiene un elenco dei formati di file di importazione ed esportazione più comuni che è possibile usare con Media Encoder Standard.

## <a name="input-containerfile-formats"></a>Contenitore di input/formati di file
| Formato di file (estensioni di file) | Supportato |
| --- | --- |
| FLV (con codec H. 264 e AAC) (.flv) |Sì |
| MXF    (.mxf) |Sì |
| GXF    (.gxf) |Sì |
| MPEG2 PS, MPEG2-TS, 3GP (TS, PS, 3GP, .3gpp, mpg) |Sì |
| Windows Media Video (WMV) (.wmv) |Yes |
| AVI (non compresso 8 bit/10 bit) (.avi) |Yes |
| MP4 (MP4, M4A,. m4v) / ISMV (ISMA, con estensione .ismv) |Sì |
| [Microsoft Digital Video Recording (DVR-MS)](https://msdn.microsoft.com/library/windows/desktop/dd692984) (.dvr-ms) |Yes |
| Matroska/WebM (.mkv) |Sì |
| WAVE/WAV (.wav) |Yes |
| QuickTime (.mov) |Yes |

> [!NOTE]
> L'elenco precedente include le estensioni di file più comuni. Media Encoder Standard supporta molte altre estensioni, ad esempio m2ts, mpeg2video, qt. Se si prova a codificare un file e viene visualizzato un messaggio di errore che indica che il formato non è supportato, inviare commenti e suggerimenti [qui](https://feedback.azure.com/forums/169396-media-services/category/144411-encoding-and-processing/).
> 
> 

### <a name="audio-formats-in-input-containers"></a>Formati audio nei contenitori di input
Il Codificatore multimedia standard supporta i seguenti formati audio nei contenitori di input:

* file MXF, GXF e QuickTime che dispongono di tracce audio con esempi di stereo interleaved o 5.1

oppure

* File MXF, GXF e QuickTime in cui l'audio viene trasportato in tracce PCM separate, ma il mapping del canale (in stereo o 5.1) può essere dedotto dai metadati del file

## <a name="input-video-codecs"></a>Codec video di input
| Codec video di input | Supportato |
| --- | --- |
| AVC 8 bit/10 bit, fino a 4:2:2, incluso AVCIntra |4:2:0 e 4:2:2 a 8 bit |
| Avid DNxHD (in MXF) |Yes |
| DVCPro/DVCProHD (in MXF) |Yes |
| Video digitale (DV) (in file AVI) |Yes |
| JPEG 2000 |Sì |
| MPEG-2 (fino a 422 Profile e High Level; incluse varianti quali XDCAM, XDCAM HD, XDCAM IMX, CableLabs® e D10) |Fino a 422 Profile |
| MPEG-1 |Sì |
| FORMATO VC-1/WMV9 |Yes |
| Canopus HQ/HQX |No |
| MPEG-4 parte 2 |Yes |
| [Theora](https://en.wikipedia.org/wiki/Theora) |Yes |
| YUV420 non compressi o mezzanine |Yes |
| Apple ProRes 422 |Sì |
| Apple ProRes 422 LT |Sì |
| Apple ProRes 422 HQ |Sì |
| Apple ProRes Proxy |Yes |
| Apple ProRes 4444 |Sì |
| Apple ProRes 4444 XQ |Yes |
| HEVC/H.265| Main Profile e Main 10 Profile (&#42;)<br/>Il supporto Main 10 Profile è destinato a contenuti 4:2:0 a 8 bit. |

## <a name="input-audio-codecs"></a>Codec audio di input
| Codec audio di input | Supportato |
| --- | --- |
| AAC (AAC-LC, AAC-HE e AAC-HEv2; fino a 5.1) |Sì |
| MPEG Layer 2 |Sì |
| MP3 (MPEG-1 Audio Layer 3) |Sì |
| Windows Media Audio |Yes |
| WAV/PCM |Yes |
| [FLAC](https://en.wikipedia.org/wiki/FLAC)</a> |Sì |
| [Opus](https://go.microsoft.com/fwlink/?LinkId=822667) |Sì |
| [Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a> |Sì |
| AMR (velocità multipla adattiva) |Yes |
| AES (SMPTE 331M e 302M, AES3-2003) |No |
| Dolby® E |No |
| Dolby® Digital (AC3) |No |
| Dolby® Digital Plus (E-AC3) |No |

## <a name="output-formats-and-codecs"></a>Formati e codec di output
La tabella seguente include l'elenco dei codec e dei formati di file supportati per l'esportazione.

| Formato file | Codec video | Codec audio |
| --- | --- | --- |
| MP4 <br/><br/>(multi-bitrate MP4 container inclusi) |H.264 (High, Main e Baseline Profile) |AAC-LC, HE-AAC v1, HE-AAC v2 |
| MPEG2-TS |H.264 (High, Main e Baseline Profile) |AAC-LC, HE-AAC v1, HE-AAC v2 |

## <a name="media-services-learning-paths"></a>Percorsi di apprendimento di Servizi multimediali
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fornire commenti e suggerimenti
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Vedere anche
[Codifica di contenuti su richiesta con Servizi multimediali di Azure](media-services-encode-asset.md)

[Come codificare con Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md)

