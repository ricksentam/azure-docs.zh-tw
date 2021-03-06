---
title: Azure Cosmos DB 查詢語言中的 SIN
description: 瞭解 Azure Cosmos DB 中的 SQL 系統函數 SIN。
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 9f4c7933bed9d51e33ada21d03b9ab8447110e32
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93082465"
---
# <a name="sin-azure-cosmos-db"></a>SIN (Azure Cosmos DB) 
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 在指定運算式中傳回指定角度的三角正弦函數 (以弧度為單位)。  
  
## <a name="syntax"></a>語法
  
```sql
SIN(<numeric_expr>)  
```  
  
## <a name="arguments"></a>引數
  
*numeric_expr*  
   為數值運算式。  
  
## <a name="return-types"></a>傳回類型
  
  傳回數值運算式。  
  
## <a name="examples"></a>範例
  
  下列範例會計算 `SIN` 指定角度的。  
  
```sql
SELECT SIN(45.175643) AS sin  
```  
  
 以下為結果集。  
  
```json
[{"sin": 0.929607286611012}]  
```  

## <a name="remarks"></a>備註

這個系統函數將不會使用索引。

## <a name="next-steps"></a>下一步

- [數學函數 Azure Cosmos DB](sql-query-mathematical-functions.md)
- [系統函數 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 簡介](introduction.md)
