---
title: 可為 Azure Cosmos 資料表 API 資料表建立資源鎖定的 PowerShell 指令碼
description: 建立 Azure Cosmos 資料表 API 資料表的資源鎖定
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: sample
ms.date: 06/12/2020
ms.openlocfilehash: f8b7cfd79188ca742ee163a02c2b6c3b8f36e6c2
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/23/2020
ms.locfileid: "92481715"
---
# <a name="create-a-resource-lock-for-azure-cosmos-table-api-table-using-azure-powershell"></a>使用 Azure PowerShell 建立 Azure Cosmos 資料表 API 資料表的資源鎖定

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-PowerShell-install](../../../../../includes/sample-PowerShell-install-no-ssh.md)]

> [!IMPORTANT]
> 除非先啟用 `disableKeyBasedMetadataWriteAccess` 屬性鎖定 Cosmos DB 帳戶，否則資源鎖定對使用任何 Cosmos DB SDK、任何透過帳戶金鑰連線的工具，或是 Azure 入口網站進行連線之使用者所做的變更無效。 若要深入了解如何啟用此屬性，請參閱[防止來自 SDK 的變更](../../../role-based-access-control.md#prevent-sdk-changes) (機器翻譯)。

## <a name="sample-script"></a>範例指令碼

[!code-PowerShell[main](../../../../../PowerShell_scripts/cosmosdb/table/ps-table-lock.ps1 "Create, list, and remove resource locks")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```PowerShell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
|**Azure 資源**| |
| [New-AzResourceLock](/PowerShell/module/az.resources/new-azresourcelock) | 建立資源鎖定。 |
| [Get-AzResourceLock](/PowerShell/module/az.resources/get-azresourcelock) | 取得資源鎖定，或列出資源鎖定。 |
| [Remove-AzResourceLock](/PowerShell/module/az.resources/remove-azresourcelock) | 移除資源鎖定。 |
|||

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/PowerShell/)。