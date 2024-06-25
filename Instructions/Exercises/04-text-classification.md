---
lab:
  title: 自訂文字分類
  module: Module 3 - Getting Started with Natural Language Processing
---

# 自訂文字分類

Azure AI 語言提供多項 NLP 功能，包括關鍵片語識別、文字摘要和情感分析。 語言服務也提供自訂功能，例如自訂問題解答和自訂文字分類。

若要測試 Azure AI 語言服務的自訂文字分類，我們會使用 Language Studio 設定模型，然後使用在 Cloud Shell 中執行的小型命令列應用程式進行測試。 可針對實際應用程式，使用此處所用的相同模式和功能。

## 佈建 *Azure AI 語言*資源

如果您的訂用帳戶中還沒有 **Azure AI 語言服務**資源，則必須加以佈建。 此外，使用自定文字分類時，必須啟用**自訂文字分類與擷取**功能。

1. 在瀏覽器中，在 `https://portal.azure.com` 中開啟 Azure 入口網站，然後使用您的 Microsoft 帳戶登入。
1. 選取入口網站頂端的搜尋欄位、搜尋 `Azure AI services`，然後建立**語言服務**資源。
1. 選取包含**自訂文字分類**的方塊。 選取 **[繼續建立您的資源]**。
1. 使用下列設定建立資源：
    - **訂用帳戶**：*您的 Azure 訂用帳戶*。
    - **資源群組**：*選取或建立資源群組*。
    - **地區**：選擇任何可用的區域**：
    - **名稱**：*輸入唯一名稱*。
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **儲存體帳戶**: 新增儲存體帳戶
      - **儲存體帳戶名稱**: *輸入唯一的名稱*。
      - **儲存體帳戶類型**: 標準 LRS
    - **負責任 AI 注意事項**：已選取。

1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 上傳範例文章

建立 Azure AI 語言服務和儲存體帳戶之後，您必須上傳範例文章，稍後用於定型您的模型。

1. 在新的瀏覽器索引標籤中，從 `https://aka.ms/classification-articles` 下載範例文章，並將檔案解壓縮到您選擇的資料夾。

1. 在 Azure 入口網站中，瀏覽至您建立的儲存體帳戶，然後加以選取。

1. 在您的儲存體帳戶中，選取位於 [設定]**** 下方的 [設定]****。 在 [設定] 視窗中，啟用 [允許 Blob 匿名存取]**** 選項，然後選取 [儲存]****。

1. 選取左側功能表中位於 [資料儲存體]**** 下方的 [容器]****。 在出現的畫面中，選取 [+ 容器]****。 將容器命名為 `articles`，並將 [匿名存取層級]**** 設為 [容器 (容器和 Blob 的匿名讀取存取)]****。

    > **注意**：當您為實際解決方案設定儲存體帳戶時，請謹慎指派適當的存取層級。 若要深入了解每個存取層級，請參閱 [Azure 儲存體文件](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)。

1. 建立容器之後，請加以選取，然後選取 **[上傳]** 按鈕。 選取 **[瀏覽檔案]**，以瀏覽您下載的範例文章。 然後選取 [上傳]****。

## 建立自訂文字分類專案

設定完成後，請建立自訂文字分類專案。 此專案提供您建置、定型和部署模型的工作位置。

> **注意**：此實驗會利用 **Language Studio**，但您也可以透過 REST API 建立、建置、定型和部署模型。

1. 在新的瀏覽器索引標籤中，開啟位於 `https://language.cognitive.azure.com/` 的 Azure AI Language Studio 入口網站，然後使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶登入。
1. 若系統提示您選擇語言資源，請選取下列設定：

    - **Azure 目錄**：包含您訂用帳戶的 Azure 目錄。
    - **Azure 訂用帳戶**：您的 Azure 訂用帳戶。
    - **資源類型**：語言。
    - **語言資源**：您先前建立的 Azure AI 語言資源。

    如果您<u>未</u>收到選擇語言資源的提示，可能是因為您的訂用帳戶中有多個語言資源；在此情況下：

    1. 在頁面頂端的列上，選取 [設定 (&#9881;)]**** 按鈕。
    2. 在 [設定]**** 分頁上，檢視 [資源]**** 索引標籤。
    3. 選取您剛才建立的語言資源，然後按一下 [切換資源]****。
    4. 在頁面頂端，按一下 [Language Studio]****，即可退回 Language Studio 首頁

1. 在入口網站頂端的 [新建]**** 功能表中，選取 [自訂文字分類]****。
1. **[連線儲存體]** 頁面會隨即顯示。 所有的值都已經填滿。 因此，可選取 **[下一步]**。
1. 在 **[選取專案類型]** 頁面上，選取 **[單一標籤分類]**。 然後選取**下一步**。
1. 在 **[輸入基本資訊]** 窗格中，設定下列各項：
    - **名稱**：`ClassifyLab`  
    - **文字主要語言**：英文 (美國)
    - **描述**：`Custom text lab`

1. 選取 [下一步]。
1. 在 [選擇容器]**** 頁面上，將 [Blob 存放區容器]**** 下拉式清單設定為*文章*容器。
1. 選取 [不，我需要將我的檔案標示為此專案的一部分]**** 選項。 然後選取**下一步**。
1. 選取**建立專案**。

## 標記您的資料

既然已建立專案，您必須標記資料，以訓練模型分類文字的方式。

1. 在左側，請選取 **[資料標記]** (若尚未選取)。 您會看到已上傳至儲存體帳戶的檔案清單。
1. 在右側的 **[活動]** 窗格中，選取 **[+ 新增類別]**。  此實驗中的文章分為四個您需要建立的類別：`Classifieds`、`Sports`、`News`和`Entertainment`。

    ![顯示標記資料頁面和 [新增類別] 按鈕的螢幕擷取畫面。](../media/tag-data-add-class-new.png#lightbox)

1. 建立四個類別之後，請選取 **[文章 1]** 以開始進行。 您可以在此閱讀文章、定義此檔案的類別，以及要將其指派至的資料集 (訓練或測試)。
1. 使用右側的 [活動]**** 窗格，為每個文章指派適當的類別和資料集 (訓練或測試)。  您可以從右側的標籤清單中選取標籤，並使用 [活動] 窗格底部的選項，將每個文章設定為**訓練**或**測試**。 選取 [下一份文件]****，以移至下一份文件。 為了此實驗室的目的，我們會定義要用於訓練模型和測試模型的項目：

    | 發行項  | 類別  | 資料集  |
    |---------|---------|---------|
    | 文章 1 | 運動 | 訓練 |
    | 文章 10 | 新聞 | 訓練 |
    | 文章 11 | 娛樂 | 測試 |
    | 文章 12 | 新聞 | 測試 |
    | 文章 13 | 運動 | 測試 |
    | 文章 2 | 運動 | 訓練 |
    | 文章 3 | 分類廣告 | 訓練 |
    | 文章 4 | 分類廣告 | 訓練 |
    | 文章 5 | 娛樂 | 訓練 |
    | 文章 6 | 娛樂 | 訓練 |
    | 文章 7 | 新聞 | 訓練 |
    | 文章 8 | 新聞 | 訓練 |
    | 文章 9 | 娛樂 | 訓練 |

    > **注意** Language Studio 中的檔案會依字母順序列出，這就是為什麼上方清單未循序。 在標記文章時，請務必造訪這兩個文件頁面。

1. 選取 **[儲存標籤]**，以儲存標籤。

## 定型模型

標記資料之後，您需要定型模型。

1. 在左側功能表上，選取 **[訓練工作]**。
1. 選取 **[開始訓練工作]**。
1. 訓練名為 `ClassifyArticles` 的新模型。
1. 選取 **[使用訓練和測試資料的手動分割]**。

    > **提示**在您自己的分類專案中，Azure AI 語言服務會自動依百分比分割測試集，這對大型資料集很實用。 使用較小的資料集時，務必使用正確的類別分佈進行定型。

1. 選取 **[定型]**

> **重要**訓練模型有時可能需要幾分鐘。 您會在完成時收到通知。

## 評估您的模型

在實際文字分類應用程式中，請務必評估並改善您的模型，以確保其效能符合您的期望。

1. 選取 [Model performance] (模型效能)****，然後選取 **ClassifyArticles** 模型。 您可以在該處查看模型的評分、效能計量，以及定型時間。 如果模型的評分不是 100%，則表示其中一份用於測試的文件並未評估其標示的內容。 這些失敗可協助您了解需改善之處。
1. 選取 **[測試集詳細資料]** 索引標籤。如果發生任何錯誤，此索引標籤可讓您查看您指示用於測試的文章和模型將其預測為什麼內容以及該內容是否與其測試標籤衝突。 索引標籤依預設只會顯示不正確的預測。 您可以切換 **[僅顯示不相符]** 選項，以查看您指示用於測試的所有文章，以及每個文章所預測的內容。

## 部署模型

滿意模型的訓練時，就可以開始部署模型，這可讓您開始透過 API 分類文字。

1. 在左側面板中，選取 **[部署模型]**。
1. 選取 [新增部署]****，然後在 [建立新的部署名稱]**** 欄位中輸入 `articles`，並且在 [模型]**** 欄位中選取 [ClassifyArticles]****。
1. 選取 **[部署]**，以部署模型。
1. 部署模型之後，請將該頁面保持開啟狀態。 在下一個步驟中，您會需要您的專案和部署名稱。

## 準備在 Visual Studio Code 中開發應用程式

若要測試 Azure AI 語言服務的自訂文字分類功能，您將在 Visual Studio Code 中開發簡單的主控台應用程式。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 設定您的應用程式

已同時提供 C# 和 Python 的應用程式，以及您將用於測試摘要的範例文字檔案。 這兩個應用程式具有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 語言資源。

1. 在 Visual Studio Code 的 [總管]**** 窗格中，瀏覽至 **Labfiles/04-text-classification**資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **classify-text** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure AI 語言文字分類功能。
1. 以滑鼠右鍵按一下包含程式碼檔案的 **classify-text** 資料夾，然後開啟整合式終端。 然後針對您的使用語言執行適當的命令，以安裝 Azure AI 語言文字分析 SDK 套件：

    **C#：**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**：

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. 在 [總管]**** 窗格中的 **classify-text** 資料夾中，開啟使用者慣用的介面語言之組態檔

    - **C#**：appsettings.json
    - **Python**：.env
    
1. 更新組態值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)。 檔案應該已經包含您文字分類模型的專案和部署名稱。
1. 儲存組態檔。

## 新增程式碼以分類文件

現在您已準備好使用 Azure AI 語言服務來分類文件。

1. 展開 **classify-text** 資料夾中的 **articles** 資料夾，以檢視應用程式將分類的文字文章。
1. 在 **classify-text** 資料夾中，開啟用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：classify-text.py

1. 尋找註解**匯入命名空間**。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**：classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. 在 **Main** 函式中，請注意，已經提供可從組態檔載入 Azure AI 語言服務端點和金鑰以及專案和部署名稱的程式碼。 然後尋找**使用端點和金鑰建立用戶端**註解，接著新增下列程式碼以建立文字分析 API 的用戶端：

    **C#**：Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**：classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 **Main** 函式中，請注意，現有的程式碼會讀取 **articles** 資料夾中的所有檔案，並建立包含其內容的清單。 然後，尋找註解**取得分類**，並新增下列程式碼：

    **C#**：Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**：classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. 將變更儲存至您的程式碼檔案。

## 測試您的應用程式

現在您的應用程式已準備好進行測試。

1. 在 **classify-text** 資料夾的整合式終端，並輸入下列命令以執行程式：

    - **C#**：`dotnet run`
    - **Python**：`python classify-text.py`

    > **秘訣**：您可以使用終端工具列中的**最大化面板大小** (**^**) 圖示來查看更多的主控台文字。

1. 觀察輸出。 應用程式應該列出每個文字檔的分類和信賴度分數。


## 清理

當您不再需要專案時，可以從 Language Studio [專案]**** 頁面加以刪除。 您也可以在 [Azure 入口網站](https://portal.azure.com)中移除 Azure AI 語言服務和相關聯的儲存體帳戶。
