---
title: Risolvere i problemi di un errore di creazione del cluster Esplora dati di Azure
description: Questo articolo descrive i passaggi di risoluzione dei problemi per la creazione di un cluster in Esplora dati di Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 9e6b3f53f07ac86d6b648a8562be4ef45879c37e
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/13/2019
ms.locfileid: "60829277"
---
# <a name="troubleshoot-failed-cluster-creation-of-azure-data-explorer"></a>Risoluzione dei problemi: Creazione del cluster non riuscita di Esplora dati di Azure

Nel caso improbabile che si verifichi un errore di creazione del cluster in Esplora dati di Azure, seguire questa procedura.

1. Assicurarsi di disporre delle autorizzazioni appropriate. Per creare un cluster è necessario essere un membro del ruolo *Collaboratore* oppure *Proprietario* della sottoscrizione di Azure. Se necessario rivolgersi all'amministratore della sottoscrizione per essere aggiunti al ruolo appropriato.

1. Assicurarsi che non ci siano errori di validazione relativi al nome del cluster inserito in **Creare cluster** nel portale di Azure.

1. Controllare il [dashboard di integrità dei servizi di Azure](https://azure.microsoft.com/status/). Cercare lo stato di Esplora dati di Azure nell'area in cui si sta cercando di creare il cluster.

    Se lo stato non è **Buono** (segno di spunta verde), provare a creare il cluster una volta avvenuto un miglioramento dello stato.

1. Per richiedere ulteriore assistenza per la risoluzione del problema, aprire una richiesta di supporto nel [portale di Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).
