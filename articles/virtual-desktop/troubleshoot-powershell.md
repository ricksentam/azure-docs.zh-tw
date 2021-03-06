---
title: Windows 虛擬桌面 PowerShell-Azure
description: 當您設定 Windows 虛擬桌面環境時，如何針對 PowerShell 的問題進行疑難排解。
author: Heidilohr
ms.topic: troubleshooting
ms.date: 06/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 03b6da1d35247749d8ec2c6459c8ddee69bfccb6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "88002265"
---
# <a name="windows-virtual-desktop-powershell"></a>Windows 虛擬桌面 PowerShell

>[!IMPORTANT]
>此內容適用於具有 Azure Resource Manager Windows 虛擬桌面物件的 Windows 虛擬桌面。 如果您使用不含 Azure Resource Manager 物件的 Windows 虛擬桌面 (傳統版)，請參閱[這篇文章](./virtual-desktop-fall-2019/troubleshoot-powershell-2019.md)。

使用本文來解決搭配 Windows 虛擬桌面使用 PowerShell 時的錯誤和問題。 如需遠端桌面服務 PowerShell 的詳細資訊，請參閱 [Windows 虛擬桌面 powershell](/powershell/module/windowsvirtualdesktop/)。

## <a name="provide-feedback"></a>提供意見反應

請瀏覽 [Windows 虛擬桌面技術社群](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop)，與產品小組和活躍的社群成員一起討論 Windows 虛擬桌面服務。

## <a name="powershell-commands-used-during-windows-virtual-desktop-setup"></a>Windows 虛擬桌面設定期間使用的 PowerShell 命令

本節列出設定 Windows 虛擬桌面時通常會使用的 PowerShell 命令，並提供解決使用這些命令時可能發生之問題的方法。

### <a name="error-new-azroleassignment-the-provided-information-does-not-map-to-an-ad-object-id"></a>錯誤： >new-azroleassignment：提供的資訊未對應到 AD 物件識別碼

```powershell
New-AzRoleAssignment -SignInName "admins@contoso.com" -RoleDefinitionName "Desktop Virtualization User" -ResourceName "0301HP-DAG" -ResourceGroupName 0301RG -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

**原因：** 在與 Windows 虛擬桌面環境系結的 Azure Active Directory 中，找不到 *-SignInName* 參數指定的使用者。

**修正：** 請確定下列事項。

- 使用者應同步至 Azure Active Directory。
- 使用者不應系結至企業對消費者 (B2C) 或企業對企業 (B2B) 商務。
- Windows 虛擬桌面環境應系結至正確的 Azure Active Directory。

### <a name="error-new-azroleassignment-the-client-with-object-id-does-not-have-authorization-to-perform-action-over-scope-code-authorizationfailed"></a>錯誤： >new-azroleassignment：「物件識別碼的用戶端沒有在範圍內執行動作的授權 (代碼： AuthorizationFailed) 」

**原因1：** 使用的帳戶沒有訂用帳戶的擁有者許可權。

**修正1：** 具有擁有者許可權的使用者必須執行角色指派。 或者，使用者必須指派給「使用者存取系統管理員」角色，才能將使用者指派給應用程式群組。

**原因2：** 使用的帳戶具有擁有者許可權，但不是環境 Azure Active Directory 的一部分，或者沒有查詢使用者所在 Azure Active Directory 的許可權。

**修正2：** 具有 Active Directory 許可權的使用者必須執行角色指派。

### <a name="error-new-azwvdhostpool----the-location-is-not-available-for-resource-type"></a>錯誤： New-AzWvdHostPool--沒有資源類型可用的位置

```powershell
New-AzWvdHostPool_CreateExpanded: The provided location 'southeastasia' is not available for resource type 'Microsoft.DesktopVirtualization/hostpools'. List of available regions for the resource type is 'eastus,eastus2,westus,westus2,northcentralus,southcentralus,westcentralus,centralus'.
```

原因： Windows 虛擬桌面支援選取主機集區、應用程式群組和工作區的位置，以將服務中繼資料儲存在特定位置。 您的選項受限於這項功能的可用位置。 此錯誤表示在您選擇的位置中無法使用此功能。

修正：在錯誤訊息中，將會發佈支援的區域清單。 請改用其中一個支援的區域。

### <a name="error-new-azwvdapplicationgroup-must-be-in-same-location-as-host-pool"></a>錯誤： New-AzWvdApplicationGroup 必須與主機集區位於相同的位置

```powershell
New-AzWvdApplicationGroup_CreateExpanded: ActivityId: e5fe6c1d-5f2c-4db9-817d-e423b8b7d168 Error: ApplicationGroup must be in same location as associated HostPool
```

**原因：** 位置不相符。 所有主機集區、應用程式群組和工作區都有存放服務中繼資料的位置。 您建立的任何與彼此相關聯的物件，都必須位於相同的位置。 例如，如果主機集區位於中 `eastus` ，則您也必須在中建立應用程式群組 `eastus` 。 如果您建立工作區以將這些應用程式群組註冊至，該工作區也必須是 `eastus` 。

**修正：** 取出主機集區建立所在的位置，然後將您建立的應用程式群組指派給相同的位置。

## <a name="next-steps"></a>後續步驟

- 如需 Windows 虛擬桌面疑難排解和擴大追蹤的概觀，請參閱[疑難排解概觀、意見反應和支援](troubleshoot-set-up-overview.md)。
- 若要針對設定 Windows 虛擬桌面環境和主機集區時的問題進行疑難排解，請參閱 [環境和主機集區建立](troubleshoot-set-up-issues.md)。
- 若要針對在 Windows 虛擬桌面中設定虛擬機器 (VM) 時的問題進行疑難排解，請參閱[工作階段主機虛擬機器設定](troubleshoot-vm-configuration.md)。
- 若要對 Windows 虛擬桌面用戶端連線問題進行疑難排解，請參閱 [Windows 虛擬桌面服務連接](troubleshoot-service-connection.md)。
- 若要疑難排解遠端桌面用戶端的問題，請參閱 [遠端桌面用戶端疑難排解](troubleshoot-client.md)
- 若要深入瞭解此服務，請參閱 [Windows 虛擬桌面環境](environment-setup.md)。
- 若要了解稽核動作，請參閱 [使用 Resource Manager 來稽核作業](../azure-resource-manager/management/view-activity-logs.md)。
- 若要了解部署期間可採取哪些動作來判斷錯誤，請參閱 [檢視部署作業](../azure-resource-manager/templates/deployment-history.md)。
