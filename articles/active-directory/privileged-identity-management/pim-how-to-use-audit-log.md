---
title: 在 Azure AD PIM 中查看 Azure AD 角色的審核記錄報告 |Microsoft Docs
description: 瞭解如何在 Azure AD Privileged Identity Management (PIM) 中查看 Azure AD 角色的審核記錄歷程記錄。
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 01/07/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8481c562ecbab1f26e877e55a5a9454695ddf4c0
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370826"
---
# <a name="view-audit-history-for-azure-ad-roles-in-privileged-identity-management"></a>在 Privileged Identity Management 中查看 Azure AD 角色的審核歷程記錄

您可以使用 Privileged Identity Management (PIM) 審核歷程記錄，以查看過去30天內所有特殊許可權角色的所有角色指派和啟用。 如果您想要查看 Azure Active Directory (Azure AD) 組織中活動的完整審核歷程記錄，包括系統管理員、終端使用者和同步處理活動，您可以使用 [Azure Active Directory 的安全性與活動報告](../reports-monitoring/overview-reports.md)。

## <a name="determine-your-version-of-pim"></a>判斷您的 PIM 版本

自2019年11月起，Privileged Identity Management 的 Azure AD 角色部分會更新為符合 Azure 資源角色體驗的新版本。 這會建立其他功能以及 [現有 API 的變更](azure-ad-roles-features.md#api-changes)。 當新版本推出時，您在本文中遵循的程式將取決於您目前擁有的 Privileged Identity Management 版本。 遵循本節中的步驟，判斷您擁有的 Privileged Identity Management 版本。 知道您的 Privileged Identity Management 版本之後，您可以在本文中選取符合該版本的程式。

1. 使用[特殊權限角色管理員](../roles/permissions-reference.md#privileged-role-administrator)角色的使用者登入[Azure 入口網站](https://portal.azure.com/)。
1. 開啟 **Azure AD Privileged Identity Management**。 如果您在 [總覽] 頁面頂端有橫幅，請依照本文的 **新版本** 索引標籤中的指示進行。 否則，請遵循 [ **舊版** ] 索引標籤中的指示。

    [![顯示 [Azure AD 角色-目錄角色審核歷程記錄] 頁面的螢幕擷取畫面。](media/pim-how-to-use-audit-log/directory-roles-audit-history.png "為您的版本選取索引標籤")](media/pim-how-to-use-audit-log/directory-roles-audit-history.png)

# <a name="new-version"></a>[新版本](#tab/new)

請遵循下列步驟來查看 Azure AD 角色的審核歷程記錄。

## <a name="view-resource-audit-history"></a>檢視資源稽核記錄

資源審核可讓您查看與您的 Azure AD 角色相關聯的所有活動。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 選取 **Azure AD 角色**]。

1. 選取 [ **資源審核**]。

1. 使用預先定義的日期或自訂範圍篩選記錄。

    ![具有篩選的資源審核清單](media/azure-pim-resource-rbac/rbac-resource-audit.png)

## <a name="view-my-audit"></a>檢視我的稽核

我的稽核可讓您檢視您的個人角色活動。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 選取 **Azure AD 角色**]。

1. 選取您要查看其審核記錄的資源。

1. 選取 [ **我的審核**]。

1. 使用預先定義的日期或自訂範圍篩選記錄。

    ![目前使用者的審核清單](media/azure-pim-resource-rbac/my-audit-time.png)

# <a name="previous-version"></a>[先前版本](#tab/previous)

## <a name="view-audit-history"></a>檢視稽核記錄

請遵循下列步驟來查看 Azure AD 角色的審核歷程記錄。

1. 以屬於[特殊權限角色管理員](../roles/permissions-reference.md#privileged-role-administrator)角色一員的使用者身分登入 [Azure 入口網站](https://portal.azure.com/)。

1. 開啟 **Azure AD Privileged Identity Management**。

1. 選取 **Azure AD 角色**]。

1. 選取 **目錄角色審核記錄**。

    根據您的審核歷程記錄，會顯示資料行圖表以及啟用總數、每天最大啟用數目，以及每天的平均啟用。

    [![新版本 Azure AD 角色](media/pim-how-to-use-audit-log/directory-roles-audit-history.png "查看目錄角色審核歷程記錄")](media/pim-how-to-use-audit-log/directory-roles-audit-history.png)

    在頁面底部，會顯示一份資料表，其中包含可用審核歷程記錄中每個動作的相關資訊。 這些資料行具有下列意義：

    | 資料行 | 描述 |
    | --- | --- |
    | Time | 當動作發生時。 |
    | 要求者 | 要求啟用或變更角色的使用者。 如果值是 **Azure 系統**，請查看 azure 審核歷程記錄以取得詳細資訊。 |
    | 動作 | 要求者所採取的動作。 動作可以包括指派、取消指派、啟動、停用或 AddedOutsidePIM。 |
    | 成員 | 啟用或指派給角色的使用者。 |
    | [角色] | 使用者指派或啟用的角色。 |
    | 理由 | 啟用期間在 [原因] 欄位中輸入的文字。 |
    | 到期 | 當啟用的角色過期時。 僅適用于合格的角色指派。 |

1. 若要排序審核歷程記錄，請按一下 [ **時間**]、[ **動作**] 和 [ **角色** ] 按鈕。

## <a name="filter-audit-history"></a>篩選稽核記錄

1. 在 [審核歷程記錄] 頁面頂端，按一下 [ **篩選** ] 按鈕。

    [ **更新圖表參數** ] 窗格隨即出現。

1. 在 [ **時間範圍**] 中，選取時間範圍。

1. 在 [ **角色**] 中，選取核取方塊以指出您要查看的角色。

    ![更新圖表參數窗格](media/pim-how-to-use-audit-log/update-chart-parameters.png)

1. 選取 [ **完成** ] 以查看篩選過的審核歷程記錄。

## <a name="get-reason-approver-and-ticket-number-for-approval-events"></a>取得核准事件的原因、核准者和票證號碼

1. 使用特殊許可權角色管理員角色許可權登入 [Azure 入口網站](https://aad.portal.azure.com) ，然後開啟 Azure AD。
1. 選取 [稽核記錄]。
1. 使用 **服務** 篩選器，只顯示許可權身分識別管理服務的 audit 事件。 在 [ **審核記錄** ] 頁面上，您可以：

    - 在 [ **狀態原因** ] 資料行中查看 audit 事件的原因。
    - 在 [將成員新增至角色要求核准] 事件的 [將成員新增至角色要求] 事件中，查看 [ **起始者 (** 動作專案]) 欄

    [![螢幕擷取畫面，顯示已選取 [起始于 (動作專案]) 功能表和 [PIM] 的 [audit logs] 頁面。](media/pim-how-to-use-audit-log/filter-audit-logs.png "篩選 PIM 服務的審核記錄")](media/pim-how-to-use-audit-log/filter-audit-logs.png)

1. 在 [**詳細資料**] 窗格的 [**活動**] 索引標籤上，選取 [審核記錄] 事件以查看票證號碼。
  
    [![顯示 [審核記錄] 頁面的螢幕擷取畫面，其中已在 [詳細資料] 窗格中反白顯示票證編號。](media/pim-how-to-use-audit-log/audit-event-ticket-number.png "檢查 audit 事件的票證號碼")](media/pim-how-to-use-audit-log/audit-event-ticket-number.png)

1. 您可以在 audit 事件的**詳細資料**窗格的 [**目標**] 索引標籤上，查看要求者 (啟用角色) 。 Azure AD 角色有兩種目標型別：

    - Role (**Type** = role) 
    - 要求者 (**類型** = 使用者) 

一般來說，在核准事件上方的 audit 記錄事件是「將成員新增至角色已完成」的事件，其中 ** (** 動作專案) 是要求者。 在大多數情況下，您不需要從審核的觀點來尋找核准要求中的要求者。

---

## <a name="next-steps"></a>後續步驟

- [在 Privileged Identity Management 中查看 Azure 資源角色的活動和審核歷程記錄](azure-pim-resource-rbac.md)
