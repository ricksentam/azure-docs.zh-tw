---
title: 在 Azure Migrate 中新增評定/遷移工具
description: 說明如何建立 Azure Migrate 專案，以及新增評量/遷移工具。
ms.topic: how-to
ms.date: 04/16/2020
ms.openlocfilehash: a94e3cc18f46c457d6ed54ef88c62adefb07c5b9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "86102526"
---
# <a name="add-an-assessmentmigration-tool-for-the-first-time"></a>第一次使用請新增評量/移轉工具

本文說明如何首次將評量或遷移工具新增至 [Azure Migrate](./migrate-services-overview.md) 專案。  
Azure Migrate 提供中央中樞，可追蹤內部部署應用程式和工作負載的探索、評量和遷移，以及 Azure 的私人/公用雲端 Vm。 中樞提供 Azure Migrate 的工具，可用於評估和遷移，以及其他工具和獨立軟體廠商 (ISV [) 供應](migrate-services-overview.md#isv-integration) 專案。 

## <a name="check-permissions-to-create-project"></a>檢查建立專案的許可權

如果您尚未建立 Azure Migrate 專案，請檢查您是否擁有正確的許可權。

1. 在 Azure 入口網站中開啟訂用帳戶，然後選取 [存取控制 (IAM)]****。
2. 在 [檢查存取權] 中，尋找相關的帳戶，然後按一下以查看權限。 您應該會具有「參與者」或「擁有者」權限。
    - 如果您剛建立免費的 Azure 帳戶，您就是訂用帳戶的擁有者。
    - 如果您不是訂用帳戶擁有者，請與擁有者合作以指派角色。

## <a name="create-a-project-and-add-a-tool"></a>建立專案並新增工具

在 Azure 訂用帳戶中設定新的 Azure Migrate 專案，並新增工具。

- Azure Migrate 專案可用來儲存從您正在評估或遷移的環境中收集到的探索、評估和遷移中繼資料。 
- 在專案中，您可以追蹤探索到的資產，以及協調評估和遷移。

1. 在 Azure 入口網站 > [所有服務]**** 中，搜尋 **Azure Migrate**。
2. 在 [服務] 下，選取 [Azure Migrate]。

    ![設定 Azure Migrate](./media/how-to-add-tool-first-time/azure-migrate-search.png)

3. 在 [概觀] 中，按一下 [評估和遷移伺服器]。
4. 在 [探索、評估和遷移伺服器]**** 下方，按一下 [評估和遷移伺服器]****。

    ![探索和評估伺服器](./media/how-to-add-tool-first-time/assess-migrate.png)

1. 在 [探索、評估和遷移伺服器]**** 中，按一下 [新增工具]****。
2. 在 [Migrate 專案]**** 中選取您的 Azure 訂用帳戶，並建立資源群組 (如果您還沒有的話)。
3. 在 [ **專案詳細資料**] 中，指定專案名稱以及要在其中建立專案的地理位置。  請檢閱[公用](migrate-support-matrix.md#supported-geographies-public-cloud)和[政府雲端](migrate-support-matrix.md#supported-geographies-azure-government)支援的地理位置。

    ![建立 Azure Migrate 專案](./media/how-to-add-tool-first-time/migrate-project.png)

    - 針對專案所指定的地理位置，僅會用於儲存從內部部署虛擬機器收集的中繼資料。 您可以選取任何目標區域以進行實際移轉。
    - 如果您需要在地理位置的特定區域內部署專案，請使用下列 API 來建立專案。 指定訂用帳戶識別碼、資源組名和專案名稱，以及位置。 檢查 [公共](migrate-support-matrix.md#supported-geographies-public-cloud) 和 [政府](migrate-support-matrix.md#supported-geographies-azure-government)雲端的地理區域/區域。

        `PUT /subscriptions/<subid>/resourceGroups/<rg>/providers/Microsoft.Migrate/MigrateProjects/<mymigrateprojectname>?api-version=2018-09-01-preview "{location: 'centralus', properties: {}}"`   


4. 按 **[下一步]**，然後新增評量或遷移工具。

    > [!NOTE]
    > 當您建立專案時，您必須新增至少一個評量或遷移工具。

5. 在 [ **選取評估工具**] 中，新增評量工具。 如果您不需要評量工具，請選取 [**立即略過新增評估工具**  >  **]**。 
2. 在 [ **選取遷移工具**] 中，視需要新增遷移工具。 如果您目前不需要遷移工具，請選取 [**立即略過新增遷移工具**  >  **]**。
3. 在 [檢閱 + 新增工具] 中檢閱設定，然後按一下 [新增工具]。

建立專案之後，您可以選取其他工具來評估和遷移伺服器和工作負載、資料庫和 web 應用程式。

## <a name="create-additional-projects"></a>建立其他專案

在某些情況下，您可能需要建立額外的 Azure Migrate 專案。 例如，如果您的資料中心位於不同的地理位置，或您需要將中繼資料儲存在不同的地理位置。 建立額外的專案，如下所示：

1. 在目前的 Azure Migrate 專案中，按一下 [ **伺服器** ] 或 [ **資料庫**]。
2. 在右上角，按一下目前專案名稱旁邊的 [ **變更** ]。
3. 在 [ **設定**] 中，選取 [ **按一下此處以建立新的專案**]。
4. 建立新的專案，並依照上一個程式所述，新增工具。

## <a name="next-steps"></a>接下來的步驟

- 開始使用 [Azure Migrate：伺服器評](migrate-services-overview.md#azure-migrate-server-assessment-tool)量，或 [Azure Migrate：伺服器遷移](migrate-services-overview.md#azure-migrate-server-migration-tool)。
- 如果您已新增 ISV 工具或 Movere，請 [參閱](prepare-isv-movere.md) 準備連結工具以 Azure Migrate 的步驟。
- 瞭解如何新增額外的 [評定](how-to-assess.md) 及 [遷移](how-to-migrate.md) 工具。 
