---
author: areddish
ms.custom: devx-track-java
ms.author: areddish
ms.service: cognitive-services
ms.date: 09/15/2020
ms.openlocfilehash: f4b64c25bff93683c79aa1aa3eb556988c798cb0
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/16/2020
ms.locfileid: "90604943"
---
本指南提供指示和範例程式碼，可協助您開始使用適用於 Java 的自訂視覺用戶端程式庫來建置影像分類模型。 您將建立專案、新增標籤、將專案定型，並使用專案的預測端點 URL 以程式設計方式加以測試。 請使用此範例作為自行建置影像辨識應用程式的範本。

> [!NOTE]
> 如果您想要在「不用」撰寫程式碼的情況下，建立和訓練分類模型，請改為參閱[以瀏覽器為基礎的指引](../../getting-started-build-a-classifier.md)。

## <a name="prerequisites"></a>先決條件：

- 您選擇的 Java IDE
- 已安裝 [JDK 7 或 8](https://aka.ms/azure-jdks)。
- 已安裝 [Maven](https://maven.apache.org/)
- [!INCLUDE [create-resources](../../includes/create-resources.md)]

## <a name="get-the-custom-vision-client-library"></a>取得自訂視覺用戶端程式庫

若要使用適用於 Java 的自訂視覺來撰寫影像分析應用程式，您將需要自訂視覺 Maven 套件。 這些套件包含在您將會下載的專案範例內，但您可以在此個別存取這些套件。

您可以在 Maven 中央存放庫中尋找自訂視覺用戶端程式庫：

- [定型 SDK](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customvision-training)
- [預測 SDK](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customvision-prediction)

複製或下載[認知服務 Java SDK 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master)專案。 瀏覽至 **Vision/CustomVision/** 資料夾。

此 Java 專案會建立名為「Java 專案範例」____ 的新自訂視覺影像分類專案，如需存取此專案，請透過[自訂視覺網站](https://customvision.ai/)。 然後上傳影像以便為分類器定型並加以測試。 在此專案中，分類器會用來判斷影像是「鐵杉」____ 還是「日本櫻」____。

[!INCLUDE [get-keys](../../includes/get-keys.md)]

此程式會設定為以環境變數的形式參考金鑰資料。 瀏覽至 **Vision/CustomVision** 資料夾，然後輸入下列 PowerShell 命令來設定環境變數。 

> [!NOTE]
> 如果您使用非 Windows 的作業系統，請參閱[設定環境變數](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Cwindows#configure-an-environment-variable-for-authentication)以取得指示。

```powershell
$env:AZURE_CUSTOMVISION_TRAINING_API_KEY ="<your training api key>"
$env:AZURE_CUSTOMVISION_PREDICTION_API_KEY ="<your prediction api key>"
```

## <a name="examine-the-code"></a>檢查程式碼

在 Java IDE 中載入 `Vision/CustomVision` 專案，然後開啟 _CustomVisionSamples.java_ 檔案。 尋找 **runSample** 方法，並將 **ObjectDetection_Sample** 方法呼叫註解化 &mdash; 此方法會執行偵測案例，但這不在本指南的討論範圍內。 **ImageClassification_Sample** 方法會實作此範例的主要功能；瀏覽至其定義，然後檢查程式碼。

## <a name="create-a-custom-vision-project"></a>建立自訂視覺專案

這第一段程式碼會建立影像分類專案。 所建立的專案會顯示在您稍早瀏覽過的[自訂視覺網站](https://customvision.ai/)上。 當您建立專案時，請參閱 [CreateProject](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.training.trainings.createproject?view=azure-java-stable#com_microsoft_azure_cognitiveservices_vision_customvision_training_Trainings_createProject_String_CreateProjectOptionalParameter_) 方法多載來指定其他選項 (如[建置分類器](../../getting-started-build-a-classifier.md) Web 入口網站指南中所述)。

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_create)]

## <a name="create-tags-in-the-project"></a>在專案中建立標記

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_tags)]

## <a name="upload-and-tag-images"></a>上傳和標記影像

影像範例包含在專案的 **src/main/resources** 資料夾中。 系統會從該處讀取影像，並使用其適當標記將其上傳至服務。

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_upload)]

先前的程式碼片段利用兩個協助程式函式，將影像擷取為資源串流，然後上傳到服務 (您最多可以在單一批次中上傳 64 個影像)。

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_helpers)]

## <a name="train-and-publish-the-project"></a>訓練及發佈專案

此程式碼會在預測模型中建立第一個反覆項目，然後將該反覆項目發佈至預測端點。 提供給已發佈反覆項目的名稱可用來傳送預測要求。 反覆項目要等到發佈後才可在預測端點中使用。

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_train)]

## <a name="use-the-prediction-endpoint"></a>使用預測端點

在此以 `predictor` 物件所表示的預測端點，可作為參考供您用來提交影像給目前的模型，並取得分類預測。 在此範例中，`predictor` 會在其他地方使用預測金鑰環境變數加以定義。

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_predict)]

## <a name="run-the-application"></a>執行應用程式

若要使用 Maven 編譯和執行解決方案，請在命令提示字元中瀏覽至專案目錄 (**Vision/CustomVision**)，然後執行執行命令：

```bash
mvn compile exec:java
```

應用程式的主控台輸出應該會類似下列文字：

```console
Creating project...
Adding images...
Adding image: hemlock_1.jpg
Adding image: hemlock_2.jpg
Adding image: hemlock_3.jpg
Adding image: hemlock_4.jpg
Adding image: hemlock_5.jpg
Adding image: hemlock_6.jpg
Adding image: hemlock_7.jpg
Adding image: hemlock_8.jpg
Adding image: hemlock_9.jpg
Adding image: hemlock_10.jpg
Adding image: japanese_cherry_1.jpg
Adding image: japanese_cherry_2.jpg
Adding image: japanese_cherry_3.jpg
Adding image: japanese_cherry_4.jpg
Adding image: japanese_cherry_5.jpg
Adding image: japanese_cherry_6.jpg
Adding image: japanese_cherry_7.jpg
Adding image: japanese_cherry_8.jpg
Adding image: japanese_cherry_9.jpg
Adding image: japanese_cherry_10.jpg
Training...
Training status: Training
Training status: Training
Training status: Completed
Done!
        Hemlock: 93.53%
        Japanese Cherry: 0.01%
```

接著，您可以確認測試影像預測 (輸出的最後幾行) 是否正確。

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>後續步驟

現在，您已了解如何在程式碼中完成物件偵測程序的每個步驟。 此範例會執行單一的訓練反覆項目，但您通常必須對模型進行多次訓練和測試，以便提升其精確度。

> [!div class="nextstepaction"]
> [測試和重新定型模型](../../test-your-model.md)

* 什麼是自訂視覺服務？
* * [SDK 參考文件](https://docs.microsoft.com/java/api/overview/azure/cognitiveservices/client/customvision?view=azure-java-stable)