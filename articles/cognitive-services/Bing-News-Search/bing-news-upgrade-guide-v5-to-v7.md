---
title: 將 Bing 新聞搜尋 API v5 升級至 v7
titleSuffix: Azure Cognitive Services
description: 識別您需要更新以使用第7版的 Bing 新聞搜尋應用程式的元件。
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: scottwhi
ms.openlocfilehash: 564af32b724c8b4883cd27d01813e246e5fa4901
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100196"
---
# <a name="news-search-api-upgrade-guide"></a>新聞搜尋 API 升級指南

> [!WARNING]
> Bing 搜尋 Api 會從認知服務移至 Bing 搜尋服務。 從 **2020 年10月 30** 日開始，任何新的 Bing 搜尋實例都必須依照 [此處](https://aka.ms/cogsvcs/bingmove)所述的程式進行布建。
> 接下來的三年或 Enterprise 合約結束之前，將支援使用認知服務布建的 Bing 搜尋 Api （以先發生者為准）。
> 如需遷移指示，請參閱 [Bing 搜尋服務](https://aka.ms/cogsvcs/bingmigration)。

本升級指南會識別 Bing 新聞搜尋 API 第 5 版和第 7 版之間的變更。 本指南可協助您識別出應用程式有哪些部分需要更新以使用第 7 版。

## <a name="breaking-changes"></a>重大變更

### <a name="endpoints"></a>端點

- 從 v5 變更為 v7 的端點版本號碼。 例如： `https://api.cognitive.microsoft.com/bing/v7.0/news/search` 。

### <a name="error-response-objects-and-error-codes"></a>錯誤回應物件和錯誤碼

- 所有失敗的要求應該立即在回應本文中包含 `ErrorResponse` 物件。

- 將下列欄位新增至 `Error` 物件。  
  - `subCode`&mdash;如果可能，將錯誤碼分割成離散貯體
  - `moreDetails`&mdash;`message` 欄位中所述之關於錯誤的其他資訊

- 將 v5 錯誤碼取代為下列可能的 `code` 和 `subCode` 值。

|程式碼|子代碼|描述
|-|-|-
|ServerError|UnexpectedError<br/>ResourceError<br/>NotImplemented|只要出現任何子代碼條件，Bing 會傳回 ServerError。 如果 HTTP 狀態碼為 500，則回應會包含這些錯誤。
|InvalidRequest|ParameterMissing<br/>ParameterInvalidValue<br/>HttpNotAllowed<br/>封鎖|只要要求的任何部分無效，Bing 就會傳回 InvalidRequest。 例如缺少必要的參數或參數值無效。<br/><br/>如果錯誤是 ParameterMissing 或 ParameterInvalidValue，則 HTTP 狀態碼為 400。<br/><br/>如果錯誤是 HttpNotAllowed，則 HTTP 狀態碼為 410。
|RateLimitExceeded||每當您超過每秒查詢 (QPS) 或每月查詢 (QPM) 配額時，Bing 會傳回 RateLimitExceeded。<br/><br/>如果超過 QPS，Bing 會傳回 HTTP 狀態碼 429，如果超過 QPM，則會傳回 403。
|InvalidAuthorization|AuthorizationMissing<br/>AuthorizationRedundancy|當 Bing 無法驗證呼叫者時，Bing 會傳回 InvalidAuthorization。 例如，缺少 `Ocp-Apim-Subscription-Key` 標頭，或訂用帳戶金鑰無效。<br/><br/>如果您指定一個以上的驗證方法，則會出現備援。<br/><br/>如果錯誤是 InvalidAuthorization，則 HTTP 狀態碼為 401。
|InsufficientAuthorization|AuthorizationDisabled<br/>AuthorizationExpired|當呼叫者沒有資源存取權限時，Bing 會傳回 InsufficientAuthorization。 如果訂用帳戶金鑰已停用或已過期，則會發生此情況。 <br/><br/>如果錯誤是 InsufficientAuthorization，則 HTTP 狀態碼為 403。

- 以下會將先前的錯誤碼對應至新的代碼。 如果您已在 v5 錯誤碼上取得相依性，請視情況更新您的程式碼。

|第 5 版程式碼|第 7 版 code.subCode
|-|-
|RequestParameterMissing|InvalidRequest.ParameterMissing
RequestParameterInvalidValue|InvalidRequest.ParameterInvalidValue
ResourceAccessDenied|InsufficientAuthorization
ExceededVolume|RateLimitExceeded
ExceededQpsLimit|RateLimitExceeded
停用|InsufficientAuthorization.AuthorizationDisabled
UnexpectedError|ServerError.UnexpectedError
DataSourceErrors|ServerError.ResourceError
AuthorizationMissing|InvalidAuthorization.AuthorizationMissing
HttpNotAllowed|InvalidRequest.HttpNotAllowed
UserAgentMissing|InvalidRequest.ParameterMissing
NotImplemented|ServerError.NotImplemented
InvalidAuthorization|InvalidAuthorization
InvalidAuthorizationMethod|InvalidAuthorization
MultipleAuthorizationMethod|InvalidAuthorization.AuthorizationRedundancy
ExpiredAuthorizationToken|InsufficientAuthorization.AuthorizationExpired
InsufficientScope|InsufficientAuthorization
封鎖|InvalidRequest.Blocked

### <a name="object-changes"></a>物件變更

- 已將 `contractualRules` 欄位新增至 [NewsArticle](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) 物件。 `contractualRules` 欄位包含您必須遵循的規則清單 (例如，文章屬性)。 您必須套用 `contractualRules` 中所提供的屬性，而不是使用 `provider`。 只有當 [Web 搜尋 API](../bing-web-search/search-the-web.md) 回應包含新聞回答時，文章中才會包含 `contractualRules`。

## <a name="non-breaking-changes"></a>非中斷性變更

### <a name="query-parameters"></a>查詢參數

- 已將產品新增為[類別](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#category)查詢參數所可能設為的可能值。 請參閱[依市場的類別](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference)。

- 已新增 [SortBy](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#sortby) 查詢參數，以便傳回依日期排序的趨勢主題 (最新的排在第一個)。

- 已新增 [Since](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#since) 查詢參數，以便傳回 Bing 在指定的 Unix Epoch 時間戳記當下或之後所探索到的趨勢主題。

### <a name="object-changes"></a>物件變更

- 已將 `mentions` 欄位新增至 [NewsArticle](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) 物件。 `mentions` 欄位包含文章中所找到的實體 (人員或位置) 清單。

- 已將 `video` 欄位新增至 [NewsArticle](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) 物件。 `video` 欄位包含與新聞文章相關的影片。 影片可以是 \<iframe\> 您可以內嵌的，也可以是動態縮圖。

- 已將 `sort` 欄位新增至 [News](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#news) 物件。 `sort` 欄位會顯示文章的排序順序。 例如，文章依關聯性 (預設值) 或日期來排序。

- 已新增 [SortValue](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#sortvalue) 物件，以便定義排序次序。 `isSelected` 欄位會指出回應是否使用排序次序。 若為 **true** ，則表示回應使用了排序次序。 如果 `isSelected` 是 **false** ，則可以在 `url` 欄位中使用 URL，來要求不同的排序次序。
