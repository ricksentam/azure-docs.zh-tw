---
title: 設定嚴重損壞修復的安全性
description: 了解在資料庫還原或容錯移轉至次要伺服器之後，設定和管理安全性時的安全性考量。
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 12/18/2018
ms.openlocfilehash: 92a0c7fd3733b5e27c34c6fd0fe157bfb466a0fd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91444883"
---
# <a name="configure-and-manage-azure-sql-database-security-for-geo-restore-or-failover"></a>建立和管理 Azure SQL Database 安全性以供異地還原或容錯移轉使用
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

本文描述設定及控制[作用中異地複寫](active-geo-replication-overview.md)和[自動容錯移轉群組](auto-failover-group-overview.md)的驗證需求。 它也提供設定使用者以存取次要資料庫的必要步驟。 最後，它也會描述如何在使用[異地還原](recovery-using-backups.md#geo-restore)之後啟用復原資料庫的存取權。 如需復原選項的詳細資訊，請參閱 [商務持續性概觀](business-continuity-high-availability-disaster-recover-hadr-overview.md)。

## <a name="disaster-recovery-with-contained-users"></a>災害復原與自主使用者

不同於傳統的使用者必須對應到 master 資料庫中的登入，自主使用者完全由資料庫本身管理。 這樣有兩個優點。 在災害復原案例中，使用者可以繼續連線到新的主要資料庫或使用異地還原復原的資料庫，而不需任何額外的組態，因為資料庫可管理使用者。 從登入的觀點來看，這個組態也有潛在的延展性和效能優勢。 如需詳細資訊，請參閱 [自主的資料庫使用者 - 使資料庫可攜](https://msdn.microsoft.com/library/ff929188.aspx)。

主要的缺點是管理大規模災害復原程序更具挑戰性。 當您有使用相同登入的多個資料庫時，維護在多個資料庫中使用自主使用者的認證可能不利於自主使用者。 例如，密碼替換原則需要在多個資料庫中進行一致的變更，而不是在主要資料庫一次變更登入的密碼。 基於這個理由，如果您有使用相同使用者名稱和密碼的多個資料庫，則不建議使用自主使用者。

## <a name="how-to-configure-logins-and-users"></a>如何設定登入和使用者

如果您使用登入和使用者 (而非自主使用者)，您必須採取額外步驟以確保主要資料庫中有相同的登入。 下列各節概述涉及的步驟和其他考量。

  >[!NOTE]
  > 也可以使用 Azure Active Directory (AAD) 登入來管理您的資料庫。 如需詳細資訊，請參閱 [Azure SQL 登入與使用者](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins)。

### <a name="set-up-user-access-to-a-secondary-or-recovered-database"></a>設定次要或復原資料庫的使用者存取

為了讓次要資料庫可做為唯讀次要資料庫使用，並確保可適當存取新的主要資料庫或使用異地還原復原的資料庫，目標伺服器的主要資料庫必須在復原之前具有適當的安全性組態。

本主題稍後說明每個步驟的特定權限。

使用者對異地複寫次要資料庫的存取權準備工作，應該是在設定異地複寫的期間執行。 使用者對異地還原資料庫的存取權準備工作，應該是在原始伺服器處於線上狀態的任何時間執行 (例如在 DR 演習期間)。

> [!NOTE]
> 如果您容錯移轉或異地還原至沒有正確設定登入存取權的伺服器，它會受限於伺服器管理帳戶。

設定目標伺服器上的登入包含下列概述的三個步驟︰

#### <a name="1-determine-logins-with-access-to-the-primary-database"></a>1. 判斷具有主資料庫存取權的登入

程序的第一個步驟是判斷目標伺服器上必須複製哪些登入。 這是以一組 SELECT 陳述式完成，一個在來源伺服器上的邏輯 master 資料庫中，一個在主要資料庫本身。

只有伺服器系統管理員或 **LoginManager** 伺服器角色的成員可以利用下列 SELECT 陳述式判斷來源伺服器上的登入。

```sql
SELECT [name], [sid]
FROM [sys].[sql_logins]
WHERE [type_desc] = 'SQL_Login'
```

只有 db_owner 資料庫角色的成員、dbo 使用者或伺服器系統管理員可以判斷主要資料庫中的所有資料庫使用者主體。

```sql
SELECT [name], [sid]
FROM [sys].[database_principals]
WHERE [type_desc] = 'SQL_USER'
```

#### <a name="2-find-the-sid-for-the-logins-identified-in-step-1"></a>2. 找出步驟1中識別的登入 SID

藉由比較上一節查詢的輸出並比對 SID，您可以將伺服器登入對應到資料庫使用者。 具有資料庫使用者及相符 SID 的登入也會有該資料庫的使用者存取權做為資料庫使用者主體。

下列查詢可用來查看資料庫中的所有使用者主體及其 SID。 只有 db_owner 資料庫角色的成員或伺服器系統管理員才能執行此查詢。

```sql
SELECT [name], [sid]
FROM [sys].[database_principals]
WHERE [type_desc] = 'SQL_USER'
```

> [!NOTE]
> **INFORMATION_SCHEMA** 和 **sys** 使用者有「NULL」** SID，而**來賓**的 SID 是 **0x00**。 如果資料庫建立者是伺服器系統管理員，而不是 **DbManager** 的成員，**dbo** SID 可能會以 *0x01060000000001648000000000048454* 開頭。

#### <a name="3-create-the-logins-on-the-target-server"></a>3. 在目標伺服器上建立登入

最後一個步驟是移至目標伺服器或伺服器，並以適當的 SID 產生登入。 基本語法如下所示。

```sql
CREATE LOGIN [<login name>]
WITH PASSWORD = <login password>,
SID = <desired login SID>
```

> [!NOTE]
> 如果您想要授與使用者次要資料庫的存取權，而不是主要資料庫的，您可以藉由使用下列語法在主要伺服器上改變使用者登入，即可完成。
>
> ```sql
> ALTER LOGIN <login name> DISABLE
> ```
>
> DISABLE 不會變更密碼，因此必要時可以永遠啟用。

## <a name="next-steps"></a>後續步驟

* 如需管理資料庫存取和登入的詳細資訊，請參閱 [SQL Database 安全性︰管理資料庫存取與登入安全性](logins-create-manage.md)。
* 如需自主資料庫使用者的詳細資訊，請參閱 [自主資料庫使用者 - 使資料庫可攜](https://msdn.microsoft.com/library/ff929188.aspx)。
* 若要深入了解作用中異地複寫，請參閱[作用中異地複寫](active-geo-replication-overview.md)。
* 若要深入瞭解自動容錯移轉群組，請參閱 [自動容錯移轉群組](auto-failover-group-overview.md)。
* 如需使用異地還原的相關資訊，請參閱[異地還原](recovery-using-backups.md#geo-restore)
