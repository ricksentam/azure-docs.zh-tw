---
title: 從 HTTPS URL 建立 Azure 媒體服務工作輸入 | Microsoft Docs
description: 本主題示範如何從 HTTPS URL 建立 Azure 媒體服務工作輸入。
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: e440e4393065d2bc1cad1a96b4f8c975624faa11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295507"
---
# <a name="create-a-job-input-from-an-https-url"></a>從 HTTPS URL 建立工作輸入

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

在媒體服務 v3 中，當您提交工作來處理視訊時，必須告知媒體服務到何處尋找輸入視訊。 其中一個選項是將 HTTPS URL 指定為作業輸入 (如本範例中所示)。 請注意，目前 AMS v3 不支援透過 HTTPS URL 的區塊傳送編碼。 如需完整的範例，請參閱此 [GitHub 範例](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs)。

> [!TIP]
> 開始開發之前，請參閱 [使用媒體服務 V3 api 進行開發](media-services-apis-overview.md) (包含存取 api、命名慣例等的資訊 ) 

## <a name="net-sample"></a>.NET 範例

下列程式碼示範如何使用 HTTPS URL 輸入來建立工作。

[!code-csharp[Main](../../../media-services-v3-dotnet-quickstarts/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs#SubmitJob)]

## <a name="job-error-codes"></a>作業錯誤碼

請參閱[錯誤碼](/rest/api/media/jobs/get#joberrorcode) \(英文\)。

## <a name="next-steps"></a>後續步驟

[從本機檔案建立工作輸入](job-input-from-local-file-how-to.md).
