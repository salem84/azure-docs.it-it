---
author: paulbouwer
ms.service: container-service
ms.topic: include
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: da4b56fb879ce108e00a40e5a26a441e330793c6
ms.sourcegitcommit: f29fec8ec945921cc3a89a6e7086127cc1bc1759
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 10/17/2019
ms.locfileid: "72530615"
---
### <a name="add-grafana-secret"></a>Aggiungi Grafana Secret

Sostituire il token `REPLACE_WITH_YOUR_SECURE_PASSWORD` con la password ed eseguire i comandi seguenti:

```bash
GRAFANA_USERNAME=$(echo -n "grafana" | base64)
GRAFANA_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: istio-system
  labels:
    app: grafana
type: Opaque
data:
  username: $GRAFANA_USERNAME
  passphrase: $GRAFANA_PASSPHRASE
EOF
```

### <a name="add-kiali-secret"></a>Aggiungi Kiali Secret

Sostituire il token `REPLACE_WITH_YOUR_SECURE_PASSWORD` con la password ed eseguire i comandi seguenti:

```bash
KIALI_USERNAME=$(echo -n "kiali" | base64)
KIALI_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```