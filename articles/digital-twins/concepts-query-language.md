---
title: 查詢語言
titleSuffix: Azure Digital Twins
description: 瞭解 Azure 數位 Twins 查詢語言的基本概念。
author: baanders
ms.author: baanders
ms.date: 3/26/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: d656f19f6f4030025ff1393c3e5017466b3333fd
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/14/2020
ms.locfileid: "92044389"
---
# <a name="about-the-query-language-for-azure-digital-twins"></a>關於 Azure 數位 Twins 的查詢語言

回想一下，Azure 數位 Twins 的中心是對應于**數位 Twins**和**關聯**性的對應項[**圖形**](concepts-twins-graph.md)。 您可以查詢此圖表，以取得其所包含的數位 twins 和關聯性的相關資訊。 這些查詢是以類似 SQL 的自訂查詢語言撰寫，稱為「 **Azure 數位 Twins 查詢語言**」。

若要從用戶端應用程式將查詢提交至服務，您將使用 Azure 數位 Twins [**查詢 API**](/dotnet/api/azure.digitaltwins.core.digitaltwinsclient.query?view=azure-dotnet-preview)。 這可讓開發人員撰寫查詢並套用篩選，以在對應項圖形中尋找數位 twins 的集合，以及 Azure 數位 Twins 案例的其他相關資訊。

[!INCLUDE [digital-twins-query-operations.md](../../includes/digital-twins-query-operations.md)]

## <a name="next-steps"></a>後續步驟

瞭解如何撰寫查詢，並查看 [*如何：查詢*](how-to-query-graph.md)對應項圖形中的用戶端程式代碼範例。