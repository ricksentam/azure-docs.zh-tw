---
title: Azure Cosmos DB 查詢語言中的 ST_WITHIN
description: 瞭解 Azure Cosmos DB 中的 SQL 系統函數 ST_WITHIN。
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: da475ced067f19cddd4a20284ba556c8a0b52677
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93079966"
---
# <a name="st_within-azure-cosmos-db"></a>ST_WITHIN (Azure Cosmos DB) 
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 傳回布林運算式，指出第一個引數中指定的 GeoJSON 物件 (Point、Polygon 或 LineString) 是否位在第二個引數中的 GeoJSON (Point、Polygon 或 LineString) 內。  
  
## <a name="syntax"></a>語法
  
```sql
ST_WITHIN (<spatial_expr>, <spatial_expr>)  
```  
  
## <a name="arguments"></a>引數
  
*spatial_expr*  
   這是 GeoJSON 點、多邊形或 LineString 物件運算式。  
  
## <a name="return-types"></a>傳回類型
  
  傳回布林值。  
  
## <a name="examples"></a>範例
  
  下列範例示範如何使用，尋找多邊形內的所有家族檔 `ST_WITHIN` 。  
  
```sql
SELECT f.id
FROM Families f
WHERE ST_WITHIN(f.location, {  
    'type':'Polygon',
    'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]  
})  
```  
  
 以下為結果集。  
  
```json
[{ "id": "WakefieldFamily" }]  
```  

## <a name="remarks"></a>備註

這個系統函數將受益于 [地理空間索引](index-policy.md#spatial-indexes)。

## <a name="next-steps"></a>下一步

- [空間函數 Azure Cosmos DB](sql-query-spatial-functions.md)
- [系統函數 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 簡介](introduction.md)
