---
lab:
  title: 擷取自訂實體
  module: Module 3 - Getting Started with Natural Language Processing
---

# 擷取自訂實體

除了其他自然語言處理功能之外，Azure AI 語言服務還可讓您定義自訂實體，並從文字中擷取實體的執行個體。

為了測試自訂實體擷取，我們將建立模型，並透過 Azure AI 語言工作室加以訓練，然後使用命令列應用程式進行測試。

## 佈建 *Azure AI 語言*資源

如果您的訂用帳戶中還沒有 **Azure AI 語言服務**資源，則必須加以佈建。 此外，使用自定文字分類時，必須啟用**自訂文字分類與擷取**功能。

1. 在瀏覽器中，在 `https://portal.azure.com` 中開啟 Azure 入口網站，然後使用您的 Microsoft 帳戶登入。
1. 選取 [建立資源]**** 按鈕，搜尋「語言」**，然後建立 [語言服務]**** 資源。 在 [選取其他功能]** 頁面中，選取包含 [自訂具名實體辨識]**** 的自訂功能。 使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選取或建立資源群組*
    - **區域**：*選擇任何可用的區域*
    - **名稱**：輸入唯一名稱**
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **儲存體帳戶**：新增儲存體帳戶：
      - **儲存體帳戶名稱**: *輸入唯一的名稱*。
      - **儲存體帳戶類型**: 標準 LRS
    - **負責任 AI 注意事項**：已選取。

1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 上傳範例廣告

建立 Azure AI 語言服務和儲存體帳戶之後，您必須上傳廣告範例，稍後用於訓練您的模型。

1. 在新的瀏覽器索引標籤中，從 `https://aka.ms/entity-extraction-ads` 下載分類廣告，並將檔案解壓縮到您選擇的資料夾。

2. 在 Azure 入口網站中，瀏覽至您建立的儲存體帳戶，然後加以選取。

3. 在儲存體帳戶中，選取位於 [設定]**** 下方的 [組態]****，在畫面上啟用 [允許 Blob 匿名存取]**** 選項，然後選取 [儲存]****。

4. 從左側功能表中選取位於 [資料儲存體]**** 下方的 [容器]****。 在出現的畫面中，選取 [+ 容器]****。 將容器命名為 `classifieds`，並將 [匿名存取層級]**** 設為 [容器 (容器和 Blob 的匿名讀取存取)]****。

    > **注意**：當您為實際解決方案設定儲存體帳戶時，請謹慎指派適當的存取層級。 若要深入了解每個存取層級，請參閱 [Azure 儲存體文件](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)。

5. 建立容器之後，請加以選取，然後按一下 [上傳]**** 按並上傳您下載的廣告範例。

## 建立自訂具名實體辨識專案

現在，您已就緒建立自訂具名實體辨識專案。 此專案提供您建置、定型和部署模型的工作位置。

> **注意**：您也可以透過 REST API 建立、建置、定型和部署模型。

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
    4. 在頁面頂端，按一下 [Language Studio]**** 以回到 Language Studio 首頁。

1. 在入口網站頂端的 [新建]**** 功能表中，選取 [自訂具名實體辨識]****。

1. 建立包含下列設定的新專案：
    - **連線儲存體**：*此值可能已經填滿。如果尚未，請將其變更為您的儲存體帳戶*
    - **基本資訊：**
    - **名稱**：`CustomEntityLab`
        - **文字主要語言**：英文 (美國)
        - **您的資料集是否包含非相同語言的文件？** :否**
        - **描述**：`Custom entities in classified ads`
    - **容器**：
        - **Blob 存放區容器**：classifieds
        - **您的檔案是否以類別標記？**：否，我需要將檔案標記為此專案的一部分

## 標記您的資料

現在您已建立專案，您必須標記資料，以定型模型識別實體的方式。

1. 如果 [資料標籤]**** 頁面尚未開啟，請在左側窗格中選取 [資料標籤]****。 您會看到已上傳至儲存體帳戶的檔案清單。
1. 在右側的 [活動]**** 窗格中，選取 [新增實體]****，然後新增名為 `ItemForSale` 的新實體。
1.  重複上一個步驟以建立下列實體：
    - `Price`
    - `Location`
1. 建立三個實體之後，請選取 [Ad 1.txt]****，以便讀取。
1. 在 *Ad 1.txt* 中： 
    1. 醒目提示文字 *face cord of firewood*，並選取 **ItemForSale** 實體。
    1. 醒目提示文字 *Denver, CO*，並選取 **Location** 實體。
    1. 醒目提示文字 *$90* ，然後選取 **Price** 實體。
1. 在 [活動]**** 窗格中，請注意，本文件將會新增至資料集以訓練模型。
1. 使用 [下一份文件]**** 按鈕以移至下一份文件，然後繼續為整組文件的適當實體指派文字，並將其全部新增至訓練資料集。
1. 標記最後一份文件 (*Ad 9.txt*) 後，請儲存標籤。

## 定型模型

標記資料之後，您需要定型模型。

1. 選取左側窗格中的 [訓練工作]****。
2. 選取 [開始訓練工作]****
3. 訓練名為 `ExtractAds` 的新模型
4. 選擇 [從定型資料自動分割測試集]****

    > **提示**：在您自己的擷取專案中，使用最適合您資料的測試分割。 如需更一致的資料和較大的資料集，Azure AI 語言服務會自動依百分比分割測試集。 使用較小的資料集時，請務必使用各種合適的可能輸入文件進行定型。

5. 按一下 [定型]****

    > **重要**：定型模型有時可能需要幾分鐘。 您會在完成時收到通知。

## 評估您的模型

在實際的應用程式中，請務必評估並改善您的模型，以確保其效能符合您的期望。 左側的兩個頁面顯示已定型模型的詳細資料，以及任何失敗的測試。

選取左側功能表上的 **[模型效能]**，然後選取您的 `ExtractAds` 模型。 您可以在該處查看模型的評分、效能計量，以及定型時間。 您將能夠查看是否有任何測試文件發生失敗，這些失敗可協助您了解需改善的項目。

## 部署模型

當您對模型的定型感到滿意時，就可以開始部署模型，這可讓您開始透過 API 擷取實體。

1. 在左側面板中，選取 [部署模型]****。
2. 選取 [新增部署]****，然後輸入名稱 `AdEntities` 並選取 [ExtractAds]**** 模型。
3. 按一下 [部署]**** 以部署模型。

## 準備在 Visual Studio Code 中開發應用程式

若要測試 Azure AI 語言服務的自訂實體擷取功能，您將在 Visual Studio Code 中開發簡單的主控台應用程式。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 設定您的應用程式

已提供 C# 和 Python 的應用程式。 這兩個應用程式具有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 語言資源。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **Labfiles/05-custom-entity-recognition**資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **custom-entities** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure AI 語言文字分類功能。
1. 以滑鼠右鍵按一下包含程式碼檔案的 **custom-entities** 資料夾，然後開啟整合式終端。 然後針對您的使用語言執行適當的命令，以安裝 Azure AI 語言文字分析 SDK 套件：

    **C#：**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**：

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. 在 [總管]**** 窗格中的 **custom-entities** 資料夾中，開啟使用者慣用的介面語言之組態檔

    - **C#**：appsettings.json
    - **Python**：.env
    
1. 更新組態值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)。 檔案應該已經包含您自訂實體擷取模型的專案和部署名稱。
1. 儲存組態檔。

## 新增程式碼以擷取實體

現在您已準備好使用 Azure AI 語言服務，從文字中擷取自訂實體。

1. 展開 **custom-entities** 資料夾中 **ads** 資料夾，以檢視您的應用程式將分析的分類廣告。
1. 在 **custom-entities** 資料夾中，開啟用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：custom-entities.py

1. 尋找註解**匯入命名空間**。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**：custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. 在 **Main** 函式中，請注意，已經提供可從組態檔載入 Azure AI 語言服務端點和金鑰以及專案和部署名稱的程式碼。 然後尋找**使用端點和金鑰建立用戶端**註解，接著新增下列程式碼以建立文字分析 API 的用戶端：

    **C#**：Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**：custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 **Main** 函式中，請注意，現有的程式碼會讀取 **ads** 資料夾中的所有檔案，並建立包含其內容的清單。 對於 C# 程式碼，**TextDocumentInput** 物件的清單中包含作為識別碼的檔案名稱以及語言。 在 Python 中，系統會使用簡單的文字內容清單。
1. 尋找註解**擷取實體**並新增下列程序碼：

    **C#**：Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**：custom-entities.py

    ```Python
    # Extract entities
    operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. 將變更儲存至您的程式碼檔案。

## 測試您的應用程式

現在您的應用程式已準備好進行測試。

1. 在 **classify-text** 資料夾的整合式終端，輸入下列命令以執行程式：

    - **C#**：`dotnet run`
    - **Python**：`python custom-entities.py`

    > **秘訣**：您可以使用終端工具列中的**最大化面板大小** (**^**) 圖示來查看更多的主控台文字。

1. 觀察輸出。 應用程式應該列出每個文字檔中所找到實體的詳細資料。

## 清理

當您不再需要專案時，可以從 Language Studio [專案]**** 頁面加以刪除。 您也可以在 [Azure 入口網站](https://portal.azure.com)中移除 Azure AI 語言服務和相關聯的儲存體帳戶。
