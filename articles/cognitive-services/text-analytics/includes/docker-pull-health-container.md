---
title: Health 容器的 Docker pull
titleSuffix: Azure Cognitive Services
description: 適用于 health 容器文字分析的 Docker pull 命令
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 07/07/2020
ms.author: aahi
ms.openlocfilehash: 519625f6468372ec7ace523dae7648212f4f3203
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91779644"
---
填寫並提交 [認知服務容器要求表單](https://aka.ms/csgate) ，以要求存取容器。
該表格需要有關您本身、您的公司，以及您將會使用該容器之使用者情節的資訊。 在您提交表單之後，Azure 認知服務小組會審核該表單，以確保您符合存取私人容器登錄的條件。

> [!IMPORTANT]
> * 在表單上，您必須使用與 Azure 訂用帳戶識別碼相關聯的電子郵件地址。
> * 您用來執行容器的 Azure 資源，必須使用已核准的 Azure 訂用帳戶識別碼來建立。 
> * 檢查電子郵件 (收件匣和垃圾資料夾) ，以取得 Microsoft 應用程式狀態的更新。

使用 docker 登入命令搭配登入電子郵件中提供的認證，以連線至我們的「認知服務」容器的私人容器登錄。


```Docker
docker login containerpreview.azurecr.io -u <username> -p <password>
```

使用 [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) 命令從私用容器登錄下載此容器映射。

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest
```
