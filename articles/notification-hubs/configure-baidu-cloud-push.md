---
title: 在 Azure 通知中樞中設定百度 Cloud Push |Microsoft Docs
description: 瞭解如何設定 Azure 通知中樞的百度設定。
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 03/25/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 759e35ba353f470ea3abc5f5d4182fa2b2ea0e73
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "71212512"
---
# <a name="deprecated-configure-baidu-cloud-push-settings-for-a-notification-hub-in-the-azure-portal"></a>已淘汰：為 Azure 入口網站中的通知中樞設定百度雲端推播設定

本文說明如何使用 Azure 入口網站來設定 Azure 通知中樞的百度雲端推播設定。

> [!IMPORTANT]
> 本教學課程已淘汰。 

## <a name="prerequisites"></a>必要條件
如果您尚未建立通知中樞，立即建立一個。 如需詳細資訊，請參閱[在 Azure 入口網站中建立 Azure 通知中樞](create-notification-hub-portal.md)。 

## <a name="configure-baidu-cloud-push"></a>設定百度雲端推播
下列程式提供設定通知中樞百度雲端推播設定的步驟：

1. 在 Azure 入口網站的 [ **通知中樞** ] 頁面上，選取左側功能表上的 [ **百度 (Android China) ** ]。 
2. 在百度雲推送專案中，輸入從百度主控台取得的 **API 金鑰**。 
3. 在百度雲推送專案中，輸入從百度主控台取得的**祕密金鑰**。 
4. 選取 [儲存]。 

    ![顯示推播通知的百度 (Android China) 設定的通知中樞螢幕擷取畫面](./media/notification-hubs-baidu-get-started/AzureNotificationServicesBaidu.png)

## <a name="next-steps"></a>後續步驟
如需使用 Azure 通知中樞將通知推送至百度以及百度雲端推播的逐步解說教學課程，請參閱 [使用百度開始使用通知中樞](notification-hubs-baidu-china-android-notifications-get-started.md)。
