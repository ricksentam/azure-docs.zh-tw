---
title: Azure Marketplace 上的虛擬機器供應專案建立方案
description: 瞭解如何在 Azure Marketplace 上建立虛擬機器供應專案的方案。
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 10/19/2020
ms.openlocfilehash: bc5e98484560fcc15e0ea3e289069c84687f158c
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040566"
---
# <a name="how-to-create-plans-for-a-virtual-machine-offer"></a>如何建立虛擬機器供應專案的方案

您可以在合作夥伴中心的相同供應項目中提供各種方案選項。 供應專案至少需要一個方案 (先前稱為 SKU) ，其可能因營收物件、Azure 區域、功能或 VM 映射而異。

您最多可以為每個供應專案建立100個方案：這些供應專案的最高45個都可以是私用。 深入瞭解 [Microsoft 商業市場中私](private-offers.md)用供應專案的私人方案。

建立方案之後，選取 [方案概觀] 索引標籤以顯示：

- 方案名稱
- 授權模型
- 對象 (公開或私人)
- 目前的發佈狀態
- 可用動作

[方案概觀] 窗格上可用的動作會根據方案的目前狀態而有所不同。

- 如果方案狀態為 [草稿]，選取 [刪除草稿]。
- 如果方案狀態是 [即時發佈]，選取 [停止銷售方案] 或 [同步私人對象]。

## <a name="create-a-new-plan"></a>建立新的方案

選取最上方的 [建立新方案]。 [新增方案] 對話方塊隨即出現。

在 [方案識別碼] 方塊中，為此供應項目中的每個方案建立唯一的方案識別碼。 客戶將可在產品網址中看到此識別碼。 僅使用小寫字母和數字、連字號或底線，以及最多 50 個字元。

> [!NOTE]
> 選取 [建立] 之後，即無法變更方案識別碼。

在 [方案名稱] 方塊中，輸入此方案的名稱。 當客戶決定要在您的供應項目內選取哪一個方案時，會看到此名稱。 建立可清楚指出方案之間差異的唯一名稱。 例如，您可以使用 [隨用隨付]、[BYOL]、[進階] 和 [企業] 方案，進入 **Windows Server** 。

選取 [建立]。

## <a name="plan-setup"></a>方案設定

設定方案類型的高階設定、指定是否重複使用另一個方案中的技術設定，以及識別應該提供方案的 Azure 區域。 您在這裡所做的選擇會決定要在相同方案的其他窗格上顯示哪些欄位。

### <a name="reuse-a-technical-configuration"></a>重複使用技術設定

如果您有多個相同類型的方案，而且這些方案之間的套件相同，則可以選取 [此方案重複使用另一個方案的技術設定]。 此選項可讓您為此供應項目選取另一個相同類型方案，並可讓您重複使用其技術設定。

> [!NOTE]
> 當您重複使用另一個方案的技術設定時，整個 [技術設定] 索引標籤都會從此方案中消失。 另一個方案中的技術設定詳細資料 (包括您在之後所做的任何更新) 也會用於此方案。 方案發佈之後即無法變更此設定。

### <a name="azure-regions"></a>Azure 區域

您的方案必須至少在一個 Azure 區域中提供。

選取 [Azure Global] 選項，讓您的方案可供所有 Azure 全球區域中具有商業 Marketplace 整合的客戶使用。 如需詳細資訊，請參閱[地理可用性和貨幣支援](marketplace-geo-availability-currencies.md) \(部分機器翻譯\)。

選取 [Azure Government] 選項，讓您的方案可用於 [Azure Government](../azure-government/documentation-government-welcome.md) \(英文\) 區域。 此區域為美國聯邦、州、地方或部落實體的客戶，以及符合資格可以為這些實體提供服務的合作夥伴，提供控管權限。 身為發行者的您必須負責任何合規性控制、安全性措施和最佳做法。 Azure Government 會使用實際隔離的資料中心和網路 (僅限位於美國地區)。

在您發佈至 [Azure Government](../azure-government/documentation-government-manage-marketplace-partners.md) 之前，請先在環境中測試並驗證您的方案，因為部分某些可能會有所不同。 若要設定並測試您的方案，請從 [Microsoft Azure 政府機構試用版](https://azure.microsoft.com/global-infrastructure/government/request/)網頁要求試用帳戶。

> [!NOTE]
> 在您的方案發佈並可在特定 Azure 區域中使用之後，您就無法移除該區域。

### <a name="azure-government-certifications"></a>Azure Government 認證

只有當您在上一節中選取 [Azure Government] 作為 [Azure 區域] 時，才會看到此選項。

Azure Government 服務會處理受限於特定政府法規和需求的資料。 例如，FedRAMP、NIST 800.171 (DIB)、ITAR、IRS 1075、DoD L4 和 CJIS。 若要告知這些程式的認證，您可以提供最多 100 個描述這些認證的連結。 這些可以是直接指向程式清單的連結，或是指向您自己網站上相關合規性描述的連結。 只有 Azure Government 客戶可以看到這些連結。

選取 [儲存草稿]，然後再繼續。

## <a name="plan-listing"></a>方案清單

在此節中，您要設定方案的清單詳細資料。 此窗格會顯示特定的資訊，這可能會與相同供應項目中的其他方案不同。

### <a name="plan-name"></a>方案名稱

此欄位會自動填入您在建立方案時所提供的名稱。 這個名稱在 Azure Marketplace 上會顯示為此方案的標題。 其限制為 100 個字元。

### <a name="plan-summary"></a>方案摘要

提供您方案 (非供應項目) 的簡短摘要。 此摘要限制為 100 個字元。

### <a name="plan-description"></a>方案描述

描述此軟體方案獨一無二之處，以及供應項目內方案之間的任何差異。 僅描述方案，而非供應項目。 方案描述最多可包含 2,000 個字元。

選取 [儲存草稿]，然後再繼續。

## <a name="pricing-and-availability"></a>價格與可用性

在此窗格上，您可以設定：

- 此方案可供使用的市場。 每個方案都必須至少在一個 [市場](marketplace-geo-availability-currencies.md)中提供。
- 每小時的價格。
- 是否要讓每個人都能看到此方案，或僅特定客戶 (私人對象) 看得到。

### <a name="markets"></a>市場

每個方案必須至少可用於一個市場。 選取此方案應可供購買所在每個市場位置的核取方塊 (這些市場中的使用者仍然可以將供應項目部署到您在[「方案設定」](#plan-setup)一節中所選取的所有 Azure 區域)。[已繳交稅金] 按鈕會顯示 Microsoft 代您繳納銷售與使用稅的國家/地區。 「發佈至中國」僅限於 *免費* 或 *自備授權* (BYOL) 的方案。

如果您已經用美元 (USD) 貨幣設定方案價格，並新增另一個市場位置，則新市場的價格會根據目前的匯率計算。 在您發佈之前，請務必先檢閱每個市場的價格。 在儲存變更之後，選取 [匯出價格 (xlsx)] 以檢閱您的定價。

當您移除市場時，該市場中使用作用中部署的客戶將無法建立新的部署，或擴大其現有的部署。 現有的部署不會受到影響。

### <a name="pricing"></a>定價

針對 [授權模型]，選取 [依使用量計算的每月計費方案]，以設定此方案的定價，或選取 [自備授權]，讓客戶搭配其現有的授權使用此方案。

針對依使用量計算的每月計費方案，請使用下列三個定價輸入選項之一：

- **每個核心** ：提供每個核心的定價 (美元)。 Microsoft 會計算每個核心大小的定價，並使用目前的匯率，將其轉換為當地貨幣。
- **每個核心大小** ：提供每個核心大小的定價 (美元)。 Microsoft 會計算定價，並使用目前的匯率，將其轉換為當地貨幣。
- **每個市場和核心大小** ：為所有市場提供每個核心大小的定價。 您可以從試算表匯入價格。

> [!NOTE]
> 儲存定價變更，以啟用匯出的定價資料。 在您的方案中發佈市場價格之後，之後就無法進行變更。 若要在發佈之前確保價格正確，請匯出定價試算表，並檢閱每個市場的價格。

### <a name="free-trial"></a>免費試用

您可以為您的客戶提供一個月或三個月的 *免費試用* 。

### <a name="visibility"></a>可見性

您可以設計每個方案，讓每個人都可以看到，或只有預先選取的對象看得到。 使用 Azure 訂用帳戶識別碼指派此有限對象中的成員資格。

**公用** ：每個人都可以看到您的方案。

**私人對象** ：讓您的方案變成只有預先選取的對象看得到。 將其發佈為私人方案之後，您可以更新對象或將其變更為公用。 在您將方案設定為公用之後，就必須保持公用。 無法將其變更回私人方案。

> [!NOTE]
> 私人對象或有限對象不同於您在 [預覽] 窗格上定義的預覽對象。 預覽對象可以在供應項目即時發佈到 Azure Marketplace 上 _之前_ 先存取。 雖然私人對象選擇僅適用於特定方案，但預覽對象可以基於驗證目的，檢視所有私人和公用方案。

**有限對象 (Azure 訂用帳戶識別碼)** ：使用 Azure 訂用帳戶識別碼指派可存取此私人方案的對象。 選擇性地加上您已指派的每個 Azure 訂用帳戶識別碼的描述。 如果您要匯入 CSV 試算表，請手動新增最多 10 個訂用帳戶識別碼，或最多 20,000 個識別碼。 Azure 訂用帳戶識別碼會以 GUID 表示，而且所有字母都必須是小寫。

>[!Note]
>透過雲端解決方案提供者方案 (CSP) 的轉銷商所建立的 Azure 訂用帳戶不支援私人優惠。

### <a name="hide-a-plan"></a>隱藏方案

如果您的虛擬機器只會在透過另一個解決方案範本或受控應用程式參考時間接使用，請選取此核取方塊以發佈虛擬機器，但對可能直接加以搜尋或瀏覽的客戶則隱藏。

> [!NOTE]
> 隱藏的方案不支援預覽連結。

選取 [儲存草稿]，然後再繼續。

## <a name="technical-configuration"></a>技術設定

提供與此方案相關聯的影像和其他技術屬性。 如需詳細資訊，請參閱 [設定 VM 供應專案清單詳細資料](azure-vm-create-listing.md)。

> [!NOTE]
> 如果您已將此方案設定為重複使用 [方案設定] 索引標籤上另一個方案的套件，則不會顯示 [技術設定] 索引標籤。

### <a name="operating-system"></a>作業系統

在 [作業系統] 窗格上，執行下列動作：

- 針對 [作業系統系列]，選取 **Windows** 或 **Linux** 作業系統。
- 針對 [版本] 或 [廠商]，選取 [Windows 版本] 或 [Linux 廠商]。
- 針對 [作業系統易記名稱]，輸入易記的作業系統名稱。 客戶可以看到此名稱。

### <a name="recommended-vm-sizes"></a>建議的 VM 大小

最多選取六個要顯示在 Azure Marketplace 上的建議虛擬機器大小。

### <a name="open-ports"></a>開啟連接埠

在已部署的虛擬機器上開啟公用或私用連接埠。

### <a name="storage-option-for-deployment"></a>用於部署的儲存選項

針對 [磁碟部署選項]，選取您客戶可以用於虛擬機器的磁碟部署類型。 Microsoft 建議將部署限制為僅 [受控磁碟部署]。

### <a name="properties"></a>屬性

針對 [支援加速網路]，選取您的 VM 是否支援[加速網路](https://go.microsoft.com/fwlink/?linkid=2124513)。

### <a name="generations"></a>層代

產生虛擬機器會定義它所使用的虛擬硬體。 您可以根據客戶的需求，發佈第1代 VM、第2代 VM 或兩者。

1. 建立新的供應專案時，請選取 **世代類型** ，然後輸入要求的映射詳細資料：

    :::image type="content" source="./media/create-vm/azure-vm-generations-image-details.png" alt-text="[產生] 下拉式清單方塊的視圖。":::

2. 若要將另一個世代新增至方案，請選取 [ **新增世代** ：

    :::image type="content" source="./media/create-vm/azure-vm-generations-add.png" alt-text="[產生] 下拉式清單方塊的視圖。":::

    然後輸入世代詳細資料：

    :::image type="content" source="./media/create-vm/azure-vm-generations-details.png" alt-text="[產生] 下拉式清單方塊的視圖。":::

    您所選擇的 **世代識別碼** 將會顯示在產品 URL 和 ARM 範本等位置中的客戶 (如果適用) ）。 只使用小寫、英數位元、連字號或底線;發行後即無法修改。

3. 若要更新已發佈世代1的現有 VM，只要在 [ **技術** 設定] 頁面上編輯詳細資料：

    :::image type="content" source="./media/create-vm/azure-vm-generations-updating.png" alt-text="[產生] 下拉式清單方塊的視圖。":::

若要深入瞭解第1代和第2代功能之間的差異，請參閱 [在 Azure 上支援第2代 vm](../virtual-machines/generation-2.md)。

### <a name="vm-images"></a>VM 映像

提供虛擬機器映像的磁碟版本和共用存取簽章 (SAS) URI。 為每個 VM 映像新增最多 16 個資料磁碟。 在指定的提交項目中，每個方案只提供一個新的映像版本。 映像發佈之後，您就無法進行編輯，但可以將其刪除。 刪除版本可防止新的和現有的使用者部署已刪除版本的新執行個體。

- **光碟版本** ：您所提供之映像的版本。
- **SAS URI** ：您的 Azure 儲存體帳戶中儲存作業系統 VHD 的位置。 若要瞭解如何取得 SAS URI，請參閱 [取得 VM 映射的共用存取](azure-vm-get-sas-uri.md)簽章 URI。
- 資料磁碟映像也是儲存在其 Azure 儲存體帳戶中的 VHD 共用存取簽章 URI。
- 在一個方案中每次提交時僅加入一個映像。

無論您使用哪一種作業系統，只能加入解決方案所需的最少數目資料磁碟。 在部署期間，客戶無法移除屬於映像一部分的磁碟，但在部署期間或之後可隨時加入磁碟。

在繼續前，選取 [儲存草稿]，然後返回 [方案概觀]。

## <a name="next-steps"></a>後續步驟

- [新增預覽對象](azure-vm-create-preview.md)
