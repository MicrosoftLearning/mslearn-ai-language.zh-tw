---
lab:
  title: 分析文字
  module: Module 3 - Develop natural language processing solutions
---

# 分析文字

**Azure 語言**支援文字分析，包括語言偵測、情感分析、關鍵片語擷取和實體辨識。

例如，假設旅遊機構想要處理已提交至公司網站的旅館評論。 他們可使用 Azure AI 語言來判斷用以撰寫每篇評論的語言、評論的情感 (正面、中立或負面)、可能指出評論中所討論主要主題的關鍵片語，以及具名實體，例如評論中提及的地點、地標或人物。

## 佈建 *Azure AI 語言*資源

如果您的 Azure 訂用帳戶中還沒有 **Azure AI 語言服務**資源，則需要在訂用帳戶中佈建一個。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 在頂端的搜尋欄位中，搜尋 **Azure AI 服務**。 然後，從結果中選取 [語言服務]**** 底下的 [建立]****。
1. 選取 [繼續建立您的資源]。****
1. 使用下列設定佈建資源：
    - **訂用帳戶**：*您的 Azure 訂用帳戶*。
    - **資源群組**：*選擇或建立資源群組*。
    - **區域**：*選擇任何可用的區域*
    - **名稱**：*輸入唯一名稱*。
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **負責任 AI 注意事項**：同意。
1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 準備在 Visual Studio Code 中開發應用程式

您將使用 Visual Studio Code 來開發文字分析應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 設定您的應用程式

已同時提供 C# 和 Python 的應用程式，以及您將用於測試摘要的範例文字檔案。 這兩個應用程式具有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 語言資源。

1. 在 Visual Studio Code 的 [總管]**** 窗格中，瀏覽至 **Labfiles/01-analyze-text** 資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **text-analysis** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure AI 語言文字分析功能。
2. 以滑鼠右鍵按一下包含程式碼檔案的 **text-analysis** 資料夾，然後開啟整合式終端。 然後針對您的使用語言執行適當的命令，以安裝 Azure AI 語言文字分析 SDK 套件。 針對 Python 練習，也請安裝 `dotenv` 套件：

    **C#：**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**：

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. 在 [總管]**** 窗格中的 **text-analysis** 資料夾中，開啟使用者慣用的介面語言之組態檔

    - **C#**：appsettings.json
    - **Python**：.env
    
4. 更新組態值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)
5. 儲存組態檔。

6. 請注意，**text-analysis** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：text-analysis.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**：text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. 在 **Main** 函式中，請注意已經提供可從組態檔載入 Azure AI 語言服務端點和金鑰的程式碼。 然後尋找**使用端點和金鑰建立用戶端**註解，接著新增下列程式碼以建立文字分析 API 的用戶端：

    **C#**：Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**：text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. 儲存變更並返回 **text-analysis** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    - **C#**：`dotnet run`
    - **Python**：`python text-analysis.py`

    > **秘訣**：您可以使用終端工具列中的**最大化面板大小** (**^**) 圖示來查看更多的主控台文字。

9. 觀察輸出，因為程式碼應該執行且不會發生錯誤，並在 **reviews** 資料夾中顯示每個評論文字檔的內容。 應用程式已成功為文字分析 API 建立用戶端，但不會使用。 我們將在下一個程序中修正該問題。

## 新增程式碼來偵測語言

既然您已為 API 建立用戶端，讓我們使用來偵測用以撰寫每篇評論的語言。

1. 在程式的 **Main** 函式中，尋找**取得語言**註解。 然後，在此註解之下，新增用以在每個評論文件中偵測語言所需的程式碼：

    **C#**：Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**：text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **注意**：*在此範例中，每篇評論都會進行個別分析，導致對每個檔案個別呼叫服務。替代方法是建立一組文件並在單一呼叫中將其傳遞給服務。在這兩種方法中，服務的回應都是由一組文件所組成；這就是為何在上述的 Python 程式碼中，已指定了回應 ([0]) 中第一份 (且唯一的) 文件的索引。*

1. 儲存您的變更。 然後返回 **text-analysis** 資料夾的整合式終端，並重新執行程式。
1. 觀察輸出，請注意這次已識別每篇評論的語言。

## 新增程式碼以評估情緒

*情感分析*是常用的技術，可將文字分類為*正面*或*負面*(或可能是*中立*或*混合*)。 其通常用來分析社交媒體文章、產品評論，以及文字情感可能提供實用見解的其他項目。

1. 在程式的 **Main** 函式中，尋找**取得情緒**註解。 然後，在此註解之下，新增用以偵測每個評論文件情感所需的程式碼：

    **C#**：Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**：text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 儲存您的變更。 然後返回 **text-analysis** 資料夾的整合式終端，並重新執行程式。
1. 觀察輸出，請注意已偵測到評論的情感。

## 新增程式碼以識別關鍵片語

識別文字本文中的關鍵片語有助於判斷其所討論的主要主題。

1. 在程式的 **Main** 函式中，尋找**取得關鍵片語**註解。 然後，在此註解之下，新增用以在每個評論文件中偵測關鍵片語所需的程式碼：

    **C#**：Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**：text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 儲存您的變更。 然後返回 **text-analysis** 資料夾的整合式終端，並重新執行程式。
1. 觀察輸出，請注意每個文件都包含關鍵片語，以提供評論內容的一些見解。

## 新增程式碼以擷取實體

通常，文件或其他文字本文會提及人物、地點、時段或其他實體。 文字分析 API 可以在您文字中偵測多種實體類別 (和子類別)。

1. 在程式的 **Main** 函式中，尋找**取得實體**註解。 然後，在此註解之下，新增用以識別每篇評論中提及的實體所需的程式碼：

    **C#**：Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**：text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 儲存您的變更。 然後返回 **text-analysis** 資料夾的整合式終端，並重新執行程式。
1. 觀察輸出，請注意文字中偵測到的實體。

## 新增程式碼以擷取連結的實體

除了已分類的實體之外，文字分析 API 還可以偵測資料來源有已知連結的實體，例如 Wikipedia。

1. 在程式的 **Main** 函式中，尋找**取得連結的實體**註解。 然後，在此註解之下，新增用以識別每篇評論中提及的連結實體所需的程式碼：

    **C#**：Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**：text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 儲存您的變更。 然後返回 **text-analysis** 資料夾的整合式終端，並重新執行程式。
1. 觀察輸出，請注意已識別的連結實體。

## 清除資源

如果您已完成探索 Azure AI 語言服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

2. 瀏覽至您在此實驗室中建立的 Azure AI 語言資源。

3. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

如需使用 **Azure AI 語言**的詳細資訊，請參閱[文件](https://learn.microsoft.com/azure/ai-services/language-service/) (部分機器翻譯)。
