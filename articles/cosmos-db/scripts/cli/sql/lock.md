---
title: 為 Azure Cosmos DB Core (SQL) API 資料庫與容器建立資源鎖定
description: 為 Azure Cosmos DB Core (SQL) API 資料庫與容器建立資源鎖定
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: 92912475b6420c0fd3ffc3a31cefb269475431f9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "87431800"
---
# <a name="create-resource-lock-for-a-azure-cosmos-db-core-sql-api-database-and-container-using-azure-cli"></a>使用 Azure CLI 為 Azure Cosmos DB Core (SQL) API 資料庫與容器建立資源鎖定

[!INCLUDE [cloud-shell-try-it.md](../../../../../includes/cloud-shell-try-it.md)]

若選擇在本機安裝及使用 CLI，此主題需要您執行 Azure CLI 2.9.1 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

> [!IMPORTANT]
> 除非先啟用 `disableKeyBasedMetadataWriteAccess` 屬性鎖定 Cosmos DB 帳戶，否則資源鎖定對使用任何 Cosmos DB SDK、任何透過帳戶金鑰連線的工具，或是 Azure 入口網站進行連線之使用者所做的變更無效。 若要深入了解如何啟用此屬性，請參閱[防止來自 SDK 的變更](../../../role-based-access-control.md#prevent-sdk-changes) (機器翻譯)。

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/sql/lock.sh "Create a resource lock for an Azure Cosmos DB Core (SQL) API database and container.")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
| [az lock create](/cli/azure/lock#az-lock-create) | 建立鎖定。 |
| [az lock list](/cli/azure/lock#az-lock-list) | 列出鎖定資訊。 |
| [az lock show](/cli/azure/lock#az-lock-show) | 顯示鎖定的屬性。 |
| [az lock delete](/cli/azure/lock#az-lock-delete) | 刪除鎖定。 |

## <a name="next-steps"></a>後續步驟

-[鎖定資源以防止非預期的變更](../../../../azure-resource-manager/management/lock-resources.md)

-[Azure Cosmos DB CLI 文件](/cli/azure/cosmosdb)。

-[Azure Cosmos DB CLI GitHub 存放庫](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb)。
