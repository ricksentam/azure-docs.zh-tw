---
title: 建立建議工具
titleSuffix: Azure Cognitive Search
description: 藉由建立建議工具並編寫叫用自動完成或 autosuggested 查詢詞彙的要求，在 Azure 認知搜尋中啟用預先輸入的查詢動作。
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/21/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: c1ee42b5b5884b0ce3491282d256d4bea87f44e1
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93083536"
---
# <a name="create-a-suggester-to-enable-autocomplete-and-suggested-results-in-a-query"></a>建立建議工具，以在查詢中啟用自動完成和建議的結果

在 Azure 認知搜尋中，您可以透過新增至 [搜尋索引](search-what-is-an-index.md)的 **建議工具** 結構來啟用「搜尋即輸入」。 建議工具支援兩種體驗： *自動* 完成（完成整個詞彙查詢的部分輸入），以及邀請按一下以進行特定相符的 *建議* 。 自動完成會產生查詢。 建議產生相符的檔。

下列 [以 c # 建立第一個應用程式](tutorial-csharp-type-ahead-and-suggestions.md) 的螢幕擷取畫面說明這兩者。 自動完成會預見可能的詞彙，以 "in" 完成「臺灣」。 建議是迷你搜尋結果，其中的欄位（例如旅館名稱）表示來自索引的相符旅館搜尋檔。 建議您可以呈現任何提供描述性資訊的欄位。

![自動完成和建議查詢的視覺化比較](./media/index-add-suggesters/hotel-app-suggestions-autocomplete.png "自動完成和建議查詢的視覺化比較")

您可以單獨或一起使用這些功能。 若要在 Azure 認知搜尋中執行這些行為，有一個索引和查詢元件。 

+ 在索引中，將建議工具加入至索引。 您可以使用入口網站、 [REST API](/rest/api/searchservice/create-index)或 [.net SDK](/dotnet/api/microsoft.azure.search.models.suggester)。 本文的其餘部分著重于建立建議工具。

+ 在查詢要求中，呼叫下列其中一個 [api](#how-to-use-a-suggester)。

針對字串欄位，以每個欄位為基礎啟用搜尋即輸入支援。 如果您想要類似螢幕擷取畫面所示的體驗，您可以在同一個搜尋解決方案中執行這兩個自動提示行為。 這兩個要求都會以特定索引的 *檔* 集合為目標，而且在使用者至少提供三個字元輸入字串之後，就會傳迴響應。

## <a name="what-is-a-suggester"></a>什麼是建議工具？

建議工具是一種內部資料結構，它會儲存前置詞來比對部分查詢，以支援搜尋型別行為。 如同 token 化詞彙一樣，前置詞會儲存在反向索引中，建議工具 fields 集合中指定的每個欄位都會有一個首碼。

## <a name="define-a-suggester"></a>定義建議工具

若要建立建議工具，請將它加入至 [索引架構](/rest/api/searchservice/create-index) ，並 [設定每個屬性](#property-reference)。 建立建議工具的最佳時機是當您同時定義將使用它的欄位時。

+ 僅使用字串欄位

+ 使用預設標準 Lucene 分析器 (`"analyzer": null`) 或 [語言分析器](index-add-language-analyzers.md) (例如， `"analyzer": "en.Microsoft"` 欄位上的) 

### <a name="choose-fields"></a>選擇欄位

雖然建議工具有數個屬性，但它主要是一組您要啟用搜尋型別體驗的字串欄位。 每個索引都有一個建議工具，因此建議工具清單必須包含提供內容的所有欄位，以提供建議和自動完成。

自動完成會從較大型的欄位集區獲益，以從中進行繪製，因為其他內容有更多的詞彙完成潛力。

另一方面，建議您在欄位選擇是選擇性時產生更好的結果。 請記住，建議是搜尋檔的 proxy，所以您會想要最能代表單一結果的欄位。 可區別多個相符專案的名稱、標題或其他唯一欄位的效果最佳。 如果欄位包含重複的值，則建議會由相同的結果組成，而且使用者不會知道要按一下哪一個。

為了滿足搜尋的型別體驗，請新增自動完成所需的所有欄位，但接著使用 **$select** 、 **$top** 、 **$filter** 和 **searchFields** 來控制建議的結果。

### <a name="choose-analyzers"></a>選擇分析器

您選擇的分析器會決定如何將欄位標記化，並在後面加上前置詞。 例如，如需使用語言分析器之類的斷字元字串，請使用語言分析器來產生這些權杖組合：「內容」、「敏感性」、「內容相關」。 您是否已使用標準 Lucene 分析器，以斷字元的字串不存在。 

評估分析器時，請考慮使用「 [分析文字 API](/rest/api/searchservice/test-analyzer) 」來深入瞭解詞彙如何 token 化，然後再加上前置詞。 建立索引之後，您可以在字串上嘗試各種分析器來查看權杖輸出。

使用 [自訂分析器](index-add-custom-analyzers.md) 或 [預先定義之分析器](index-add-custom-analyzers.md#predefined-analyzers-reference) (的欄位，除了標準 Lucene) 之外，明確不允許用來防止結果不良。

> [!NOTE]
> 如果您需要解決分析器條件約束（例如，如果您需要關鍵字或 ngram 分析器來處理特定的查詢案例），您應該針對相同的內容使用兩個不同的欄位。 這可讓其中一個欄位具有建議工具，而另一個欄位可以使用自訂分析器設定來設定。

### <a name="when-to-create-a-suggester"></a>建立建議工具的時機

建立建議工具的最佳時機是當您同時建立欄位定義時。

如果您嘗試使用預先存在的欄位來建立建議工具，API 將不會允許它。 在索引編制期間會產生前置詞，而在兩個或更多字元組合中的部分詞彙則會與整個詞彙一起標記。 由於現有的欄位已經標記化，如果您想要將索引新增至建議工具，就必須重建索引。 如需詳細資訊，請參閱 [如何重建 Azure 認知搜尋索引](search-howto-reindex.md)。

## <a name="create-using-rest"></a>使用 REST 建立

在 REST API 中，透過 [Create index](/rest/api/searchservice/create-index) 或 [Update index](/rest/api/searchservice/update-index)新增建議工具。 

  ```json
  {
    "name": "hotels-sample-index",
    "fields": [
      . . .
          {
              "name": "HotelName",
              "type": "Edm.String",
              "facetable": false,
              "filterable": false,
              "key": false,
              "retrievable": true,
              "searchable": true,
              "sortable": false,
              "analyzer": "en.microsoft",
              "indexAnalyzer": null,
              "searchAnalyzer": null,
              "synonymMaps": [],
              "fields": []
          },
    ],
    "suggesters": [
      {
        "name": "sg",
        "searchMode": "analyzingInfixMatching",
        "sourceFields": ["HotelName"]
      }
    ],
    "scoringProfiles": [
      . . .
    ]
  }
  ```

## <a name="create-using-net"></a>使用 .NET 建立

在 c # 中，定義 [建議工具物件](/dotnet/api/microsoft.azure.search.models.suggester)。 `Suggesters` 是集合，但只能採用一個專案。 

```csharp
private static void CreateHotelsIndex(SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = "hotels-sample-index",
        Fields = FieldBuilder.BuildForType<Hotel>(),
        Suggesters = new List<Suggester>() {new Suggester()
            {
                Name = "sg",
                SourceFields = new string[] { "HotelName", "Category" }
            }}
    };

    serviceClient.Indexes.Create(definition);

}
```

## <a name="property-reference"></a>屬性參考

|屬性      |描述      |
|--------------|-----------------|
|`name`        |建議工具的名稱。|
|`searchMode`  |用來搜尋候選片語的策略。 目前唯一支援的模式是 `analyzingInfixMatching` ，目前與詞彙的開頭相符。|
|`sourceFields`|建議之內容來源的一或多個欄位清單。 欄位的類型必須是 `Edm.String` 和 `Collection(Edm.String)` 。 如果在欄位上指定了分析器，它必須是 [此清單](/dotnet/api/microsoft.azure.search.models.analyzername) 中的命名分析器， (不是自訂分析器) 。<p/> 最佳做法是只指定讓自己成為預期和適當回應的欄位，無論是搜尋列或下拉式清單中的完整字串。<p/>旅館名稱是很好的候選項，因為它有精確度。 詳細資訊欄位（例如描述和批註）過於密集。 同樣地，重複的欄位（例如，類別和標記）則較不有效。 在範例中，我們仍然會包含 "category"，以示範您可以包含多個欄位。 |

<a name="how-to-use-a-suggester"></a>

## <a name="use-a-suggester"></a>使用建議工具

建議工具用於查詢中。 建立建議工具之後，請呼叫下列其中一個 Api 來進行搜尋即您的體驗：

+ [建議 REST API](/rest/api/searchservice/suggestions) 
+ [自動完成 REST API](/rest/api/searchservice/autocomplete) 
+ [SuggestWithHttpMessagesAsync 方法] (/dotnet/api/microsoft.azure.search.idocumentsoperations.suggestwithHTTPmessagesasync？
+ [AutocompleteWithHttpMessagesAsync 方法](/dotnet/api/microsoft.azure.search.idocumentsoperations.autocompletewithhttpmessagesasync)

在搜尋應用程式中，用戶端程式代碼應該利用程式庫（例如 [JQUERY UI 自動完成](https://jqueryui.com/autocomplete/) ）來收集部分查詢並提供相符的結果。 如需這項工作的詳細資訊，請參閱 [將自動完成或建議的結果新增至用戶端程式代碼](search-autocomplete-tutorial.md)。

[自動完成] REST API 的下列呼叫中說明 API 使用方式。 此範例有兩個優點。 首先，與所有查詢相同，此作業是針對索引的檔集合，而查詢則包含 **搜尋** 參數，在此情況下會提供部分查詢。 其次，您必須將 **suggesterName** 新增至要求。 如果索引中未定義建議工具，則對自動完成或建議的呼叫將會失敗。

```http
POST /indexes/myxboxgames/docs/autocomplete?search&api-version=2020-06-30
{
  "search": "minecraf",
  "suggesterName": "sg"
}
```

## <a name="sample-code"></a>範例程式碼

+ 使用[c # 建立您的第一個應用程式 (第3課-新增搜尋](tutorial-csharp-type-ahead-and-suggestions.md)型別) 範例示範建議工具結構、建議的查詢、自動完成和多面向導覽。 這個程式碼範例會在沙箱 Azure 認知搜尋服務上執行，並使用預先載入的旅館索引，所以您只需要按 F5 來執行應用程式即可。 不需要訂用帳戶或登入。

+ [>dotnethowtoautocomplete](https://github.com/wantedfast/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete) 是包含 c # 和 JAVA 程式碼的較舊範例。 它也會示範建議工具結構、建議的查詢、自動完成和多面向導覽。 這個程式碼範例會使用 hosted [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) 範例資料。 

## <a name="next-steps"></a>下一步

建議您閱讀下列文章，以深入瞭解要求的表述。

> [!div class="nextstepaction"]
> [將自動完成和建議新增至用戶端程式代碼](search-autocomplete-tutorial.md)