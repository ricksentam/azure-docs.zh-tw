---
title: 教學課程：使用 Azure Active Directory 設定 Code42 來自動布建使用者 |Microsoft Docs
description: 瞭解如何從 Azure AD 將使用者帳戶自動布建和取消布建至 Code42。
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: ddcb950b-3f9a-4ebb-bf78-4ec42d16d52d
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/23/2020
ms.author: Zhchia
ms.openlocfilehash: 19e4a6b15f32ea9854801bcf6024577a1697c563
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455658"
---
# <a name="tutorial-configure-code42-for-automatic-user-provisioning"></a>教學課程：設定 Code42 來自動布建使用者

本教學課程說明在 Code42 和 Azure Active Directory (Azure AD) 設定自動使用者布建時所需執行的步驟。 當設定時，Azure AD 會使用 Azure AD 布建服務，自動將使用者和群組布建和取消布建至 [Code42](https://www.crashplan.com/) 。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。


## <a name="capabilities-supported"></a>支援的功能
> [!div class="checklist"]
> * 在 Code42 中建立使用者
> * 當使用者不再需要存取權時，請移除 Code42 中的使用者
> * Azure AD 與 Code42 之間保持使用者屬性同步
> * 在 Code42 中布建群組和群組成員資格
> * Code42 (建議的[單一登入](./code42-tutorial.md)) 

## <a name="prerequisites"></a>Prerequisites

本教學課程中概述的案例假設您已經具有下列必要條件：

* [Azure AD 租用戶](../develop/quickstart-create-new-tenant.md)
* Azure AD 中具有設定佈建[權限](../users-groups-roles/directory-assign-admin-roles.md)的使用者帳戶 (例如，應用程式管理員、雲端應用程式管理員、應用程式擁有者或全域管理員)。
* 已啟用身分識別管理的 Code42 租使用者。
* 具有 [客戶雲端系統管理員](https://support.code42.com/Administrator/Cloud/Monitoring_and_managing/Roles_reference#Customer_Cloud_Admin) 許可權的 Code42 使用者帳戶。

## <a name="step-1-plan-your-provisioning-deployment"></a>步驟 1： 規劃佈建部署
1. 了解[佈建服務的運作方式](../app-provisioning/user-provisioning.md) \(部分機器翻譯\)。
2. 判斷誰會在[佈建範圍](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)內。
3. 判斷要 [在 Azure AD 與 Code42 之間對應](../app-provisioning/customize-application-attributes.md)的資料。

## <a name="step-2-configure-code42-to-support-provisioning-with-azure-ad"></a>步驟 2： 設定 Code42 以支援 Azure AD 的布建

本節將引導您完成在 Code42's 主控台的 [身分識別管理] 區段中，將 Azure AD 設定為布建提供者的步驟。 這麼做可讓 Code42 安全地從 Azure AD 接收布建要求。 建議您在使用 Azure AD 布建之前，先複習 [Code42's 支援檔](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD) 。

### <a name="to-create-a-provisioning-provider-in-code42s-console"></a>若要在 Code42's 主控台中建立布建提供者：

1. 登入您的 Code42 主控台。 選取 [ **管理** ] 以展開導覽功能表。 依序選取 [ **設定** 身分 **識別管理**]。
2. 選取 [ **布** 建] 索引標籤。然後，展開 [新增布建 **提供者** ] 功能表並選取 [ **新增 SCIM 提供者**]。
3. 在 [ **顯示名稱** ] 欄位中，輸入布建提供者的唯一名稱。 將 **驗證認證類型** 設定為 **OAuth 權杖**。 選取 **[下一步]** 以產生認證。

> [!NOTE]
>* 讓此視窗保持開啟，直到系統提示您提供後續步驟中所需的 **基底 URL** 和 **權杖** 。
>* 或者，將此資訊複製到暫存位置，以供日後參考。

## <a name="step-3-add-code42-from-the-azure-ad-application-gallery"></a>步驟 3： 從 Azure AD 應用程式資源庫新增 Code42

從 Azure AD 應用程式資源庫新增 Code42，以開始管理布建至 Code42。 如果您先前已設定 SSO 的 Code42，您可以使用相同的應用程式。 不過，建議您在一開始測試整合時，建立個別的應用程式。 [在此](../manage-apps/add-application-portal.md)深入了解從資源庫新增應用程式。

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>步驟 4： 定義將在佈建範圍內的人員

Azure AD 佈建服務可供根據對應用程式的指派，或根據使用者/群組的屬性，界定將要佈建的人員。 如果您選擇根據指派來界定將佈建至應用程式的人員，您可以使用下列[步驟](../manage-apps/assign-user-or-group-access-portal.md)將使用者和群組指派給應用程式。 如果您選擇僅根據使用者或群組的屬性來界定將要佈建的人員，可以使用如[這裡](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)所述的範圍篩選條件。

* 將使用者和群組指派給 Code42 時，您必須選取 **預設存取**以外的角色。 具有預設存取角色的使用者會從佈建中排除，而且會在佈建記錄中被標示為沒有效率。 如果應用程式上唯一可用的角色是 [預設存取] 角色，您可以[更新應用程式資訊清單](../develop/howto-add-app-roles-in-azure-ad-apps.md) \(部分機器翻譯\) 以新增其他角色。

* 從小規模開始。 在推出給所有人之前，先使用一小部分的使用者和群組進行測試。 當佈建範圍設為已指派的使用者和群組時，您可將一或兩個使用者或群組指派給應用程式來控制這點。 當範圍設為所有使用者和群組時，您可指定[以屬性為基礎的範圍篩選條件](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)。


## <a name="step-5-configure-automatic-user-provisioning-to-code42"></a>步驟 5。 設定自動使用者布建至 Code42

此節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 TestApp 中建立、更新和停用使用者和/或群組。

### <a name="to-configure-automatic-user-provisioning-for-code42-in-azure-ad"></a>若要在 Azure AD 中為 Code42 設定自動使用者布建：

1. 登入 [Azure 入口網站](https://portal.azure.com)。 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

2. 在應用程式清單中，選取 [ **Code42**]。

    ![應用程式清單中的 Code42 連結](common/all-applications.png)

3. 選取 [佈建] 索引標籤。

    ![佈建索引標籤](common/provisioning.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![自動布建索引標籤](common/provisioning-automatic.png)

5. 在 [系統**管理員認證**] 區段下，輸入稍早從**租使用者 URL**和**秘密權杖**中的 Code42 之前取出的**SCIM 2.0 基底 url 和存取權杖**值。 按一下 [ **測試連接** ] 以確保 Azure AD 可以連線至 Code42。 如果連接失敗，請確定您的 Code42 帳戶具有系統管理員許可權，然後再試一次。

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

6. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![通知電子郵件](common/provisioning-notification-email.png)

7. 選取 [儲存]。

8. **在 [對應**] 區段下，選取 [**同步處理 Azure Active Directory 使用者至 Code42**]。

9. 在 [ **屬性對應** ] 區段中，檢查從 Azure AD 同步處理到 Code42 的使用者屬性。 選取為 [比對 **] 屬性的屬性會** 用來比對 Code42 中的使用者帳戶以進行更新作業。 如果您選擇變更相符的 [目標屬性](../app-provisioning/customize-application-attributes.md)，您將必須確定 Code42 API 支援根據該屬性篩選使用者。 選取 [儲存] 按鈕以認可所有變更。

   |屬性|類型|
   |---|---|
   |userName|String|
   |作用中|Boolean|
   |title|String|
   |emails[type eq "work"].value|String|
   |name.givenName|String|
   |name.familyName|String|
   |addresses[type eq "work"].locality|String|
   |addresses[type eq "work"].region|String|
   |addresses[type eq "work"].country|String|
   |externalId|String|
   |userType|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|參考|

10. **在 [對應**] 區段下，選取 [**同步處理 Azure Active Directory 群組至 Code42**]。

11. 在 [ **屬性對應** ] 區段中，檢查從 Azure AD 同步處理到 Code42 的群組屬性。 選取為 [比對 **] 屬性的屬性會** 用來比對 Code42 中的群組以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

      |屬性|類型|
      |---|---|
      |displayName|String|
      |externalId|String|
      |members|參考|

12. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。

13. 若要啟用 Code42 Azure AD 的布建服務，請在 [**設定**] 區段中，將 [布建**狀態**] 變更為 [**開啟**]。

    ![佈建狀態已切換為開啟](common/provisioning-toggle-on.png)

14. 在 [**設定**] 區段的 [**範圍**] 中選擇所需的值，以定義您想要布建到 Code42 的使用者和/或群組。

    ![佈建範圍](common/provisioning-scope.png)

15. 當您準備好要佈建時，按一下 [儲存]  。

    ![儲存雲端佈建設定](common/provisioning-configuration-save.png)

此作業會對在 [設定] 區段的 [範圍] 中所定義所有使用者和群組啟動首次同步處理週期。 初始週期會比後續週期花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。

## <a name="step-6-monitor-your-deployment"></a>步驟 6. 監視您的部署
設定佈建後，請使用下列資源來監視您的部署：

1. 使用[佈建記錄](../reports-monitoring/concept-provisioning-logs.md) \(部分機器翻譯\) 來判斷哪些使用者已佈建成功或失敗
2. 檢查[進度列](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) \(部分機器翻譯\) 來查看佈建週期的狀態，以及其接近完成的程度
3. 如果佈建設定似乎處於狀況不良的狀態，應用程式將會進入隔離狀態。 [在此](../app-provisioning/application-provisioning-quarantine-status.md)深入了解隔離狀態。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [根據 Code42 中的 SCIM 群組設定組織指派](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD#Step_6:_Choose_an_organization_mapping_method)
* [設定以 Code42 中的 SCIM 群組為基礎的角色指派](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD#Step_7:_Configure_role_mapping)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../app-provisioning/check-status-user-account-provisioning.md)