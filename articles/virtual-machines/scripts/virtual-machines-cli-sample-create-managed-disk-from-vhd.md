---
title: 相同帳戶中 VHD 檔案的受控磁碟 - CLI 範例
description: Azure CLI 指令碼範例 - 從相同訂用帳戶的儲存體帳戶的 VHD 檔案建立受控磁碟
documentationcenter: storage
author: ramankumarlive
manager: kavithag
ms.service: virtual-machines
ms.subservice: disks
ms.devlang: azurecli
ms.topic: sample
ms.workload: infrastructure
ms.date: 05/19/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: d0795f67390faec4a08882adb1f2b15efcbc6619
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91274855"
---
# <a name="create-a-managed-disk-from-a-vhd-file-in-a-storage-account-in-the-same-subscription-with-cli-linux"></a>使用 CLI 從相同訂用帳戶的儲存體帳戶的 VHD 檔案建立受控磁碟 (Linux)

此指令碼會從相同訂用帳戶的儲存體帳戶的 VHD 檔案建立受控磁碟。 使用此指令碼可將專用 (非一般化/已執行過 Sysprep) 的 VHD 匯入至受控 OS 磁碟，以建立虛擬機器。 或者，用它將資料 VHD 匯入到受控資料磁碟。 


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/virtual-machine/create-managed-data-disks-from-vhd/create-managed-data-disks-from-vhd.sh "Create managed disk from VHD")]


## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令從 VHD 建立受控磁碟。 下表中的每個命令都會連結至命令特定的文件。

| Command | 注意 |
|---|---|
| [az disk create](/cli/azure/disk) | 在相同的訂用帳戶中使用儲存體帳戶的 VHD URI 受控磁碟 |

## <a name="next-steps"></a>後續步驟

[將受控磁碟連結為 OS 磁碟以建立虛擬機器](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](/cli/azure)。

您可以在 [Azure Linux VM 文件](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到其他的虛擬機器和受控磁碟 CLI 指令碼範例。
