---
title: EXP nel linguaggio di query Azure Cosmos DB
description: Informazioni sulla funzione di sistema SQL EXP in Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 21c7ae63f46f2acd961245c59805220174c106f1
ms.sourcegitcommit: 7f6d986a60eff2c170172bd8bcb834302bb41f71
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/27/2019
ms.locfileid: "71351037"
---
# <a name="exp-azure-cosmos-db"></a>EXP (Azure Cosmos DB)
 Restituisce il valore esponenziale dell'espressione numerica specificata.  
  
## <a name="syntax"></a>Sintassi
  
```sql
EXP (<numeric_expr>)  
```  
  
## <a name="arguments"></a>Argomenti
  
*numeric_expr*  
   È un'espressione numerica.  
  
## <a name="return-types"></a>Tipi restituiti
  
  Restituisce un'espressione numerica.  
  
## <a name="remarks"></a>Note
  
  La costante **e** (2,718281 …) è la base dei logaritmi naturali.  
  
  L'esponente di un numero è la costante **e** elevata alla potenza del numero. Ad esempio, EXP(1.0) = e^1.0 = 2,71828182845905 e EXP(10) = e^10 = 22026,4657948067.  
  
  Il valore esponenziale del logaritmo naturale di un numero è il numero stesso: EXP (LOG (n)) = n. E il logaritmo naturale del valore esponenziale di un numero è il numero stesso: LOG (EXP (n)) = n.  
  
## <a name="examples"></a>Esempi
  
  Nell'esempio seguente viene dichiarata una variabile e restituito il valore esponenziale della variabile specificata (10).  
  
```sql
SELECT EXP(10) AS exp  
```  
  
 Questo è il set di risultati.  
  
```json
[{exp: 22026.465794806718}]  
```  
  
 L'esempio seguente restituisce il valore esponenziale del logaritmo naturale di 20 e il logaritmo naturale dell'esponente di 20. Poiché queste funzioni sono inverse l'una dell'altra, il valore restituito con arrotondamento per il calcolo a virgola mobile in entrambi i casi è 20.  
  
```sql
SELECT EXP(LOG(20)) AS exp1, LOG(EXP(20)) AS exp2  
```  
  
 Questo è il set di risultati.  
  
```json
[{exp1: 19.999999999999996, exp2: 20}]  
```  

## <a name="next-steps"></a>Passaggi successivi

- [Funzioni matematiche Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Funzioni di sistema Azure Cosmos DB](sql-query-system-functions.md)
- [Introduzione a Azure Cosmos DB](introduction.md)
