---
title: 快速入門：使用 SDK 建立和使用學習迴圈 - 個人化工具
description: 本快速入門將說明如何使用「個人化工具」用戶端程式庫來建立和管理您的知識庫。
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: quickstart
ms.date: 08/27/2020
ms.custom: cog-serv-seo-aug-2020
keywords: 個人化工具, Azure 個人化工具, 機器學習
zone_pivot_groups: programming-languages-set-six
ms.openlocfilehash: 3acfd8134a2a65e6ea9f019baff24b8b8149f7f3
ms.sourcegitcommit: ef69245ca06aa16775d4232b790b142b53a0c248
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2020
ms.locfileid: "91777331"
---
# <a name="quickstart-personalizer-client-library"></a>快速入門：個人化工具用戶端程式庫

使用個人化工具服務顯示此快速入門中的個人化內容。

開始使用個人化工具用戶端程式庫。 請遵循下列步驟來安裝套件，並試用基本工作的程式碼範例。

 * 排名 API - 根據您就內容和關係提供的相關即時資訊，從動作中選取最佳項目。
 * 獎勵 API - 您可以根據您的業務需求決定獎勵分數，然後使用此 API 將其傳送到個人化工具。 該分數可以是單一值，例如 1 表示良好、0 表示不良，或根據您的業務需求建立的演算法。

::: zone pivot="programming-language-csharp"
[!INCLUDE [Get intent with C# SDK](./includes/quickstart-sdk-csharp.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Get intent with Node.js SDK](./includes/quickstart-sdk-nodejs.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Get intent with Python SDK](./includes/quickstart-sdk-python.md)]
::: zone-end

## <a name="clean-up-resources"></a>清除資源

如果您想要清除和移除認知服務訂用帳戶，則可以刪除資源或資源群組。 刪除資源群組也會刪除其關聯的任何其他資源。

* [入口網站](../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [個人化工具的運作方式](how-personalizer-works.md)
> [個人化工具的用處為何？](where-can-you-use-personalizer.md)
