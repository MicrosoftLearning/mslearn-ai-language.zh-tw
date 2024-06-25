---
lab:
  title: 使用 Azure AI 語言服務建立語言理解模型
  module: Module 5 - Create language understanding solutions
---

# 使用語言服務建立語言理解模型

> **注意事項** Azure AI 語言服務的交談語言理解功能目前處於預覽狀態，而且可能會變更。 在某些情況下，模型定型可能會失敗 - 如果發生這種情況，請重新嘗試一次。  

Azure AI 語言服務可讓您定義*交談語言理解*模型，而應用程式可使用該模型來解譯使用者自然語言輸入、預測使用者*意圖* (他們想要達成的目的)，以及識別應套用意圖的任何*實體*。

例如，適用於時鐘應用程式的交談語言模型可能需要處理輸入，例如：

*倫敦現在幾點？*

這種輸入是*表達*的範例，(使用者可能會說出或鍵入的東西)，其*意圖*是取得特定位置 (*實體*) 的時間；在此案例中為倫敦。

> **注意事項** 交談語言模型的工作是預測使用者的意圖，並識別套用意圖的任何實體。 實際執行滿足意圖所需的動作<u>並非</u>交談語言模型的工作。 例如，時鐘應用程式可以使用交談語言模型來辨識使用者想要知道倫敦的時間；但用戶端應用程式本身必須接下來實作邏輯，以判斷正確的時間，並將其呈現給使用者。

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

## 建立交談語言理解專案

既然您已建立製作資源，您可以使用其來建立交談語言理解專案。

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
    4. 在頁面頂端，按一下 [Language Studio]****，即可退回 Language Studio 首頁。

1. 在入口網站頂端的 [新建] **** 功能表中，選取 [交談語言理解] ****。

1. 在 [建立專案]**** 對話方塊的 [輸入基本資訊]**** 頁面上，輸入下列詳細資料，然後選取 [下一步]****：
    - **名稱**：`Clock`
    - **表達主要語言**：英文
    - **在專案中啟用多種語言？**：*未選取*
    - **描述**：`Natural language clock`

1. 在 [檢閱和完成] **** 頁面上，選取 [建立]****。

### 建議意圖

我們在新專案中做的第一件事是定義一些意圖。 該模型最終會預測使用者在提交自然語言表達時會要求哪些意圖。

> **秘訣**：處理您的專案時，如果顯示一些秘訣，請閱讀這些秘訣，然後按一下 [知道了]**** 將其關閉，或按一下 [全部略過]****。

1. 在 [結構描述定義]**** 頁面上的 [意圖]**** 索引標籤上，選取 [&#65291;新增]**** 以新增名為 `GetTime` 的新意圖。
1. 驗證是否已列出 **GetTime** 意圖 (以及預設的 **None** 意圖)。 然後新增下列其他意圖：
    - `GetDay`
    - `GetDate`

### 使用範例表達標記每個意圖

若要協助模型預測使用者會要求哪個意圖，您必須使用一些範例表達來標記每個意圖。

1. 在左側窗格中，選取 [資料標籤]**** 頁面。

> **秘訣**：您可以使用 **>>** 圖示展開窗格以查看頁面名稱，然後使用 **<<** 圖示再次將其隱藏。

1. 選取新的 **GetTime** 意圖，然後輸入表達 `what is the time?`。 這會將表達新增為意圖的範例輸入。
1. 針對 **GetTime** 意圖新增下列其他表達：
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

1. 選取 **GetDay** 意圖，並將下列表達新增為該意圖的範例輸入：
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. 選取 **GetDate** 意圖，並為其新增下列表達：
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. 針對您每個意圖新增表達之後，請選取 [儲存變更]****。

### 定型和測試模型

既然您已新增一些意圖，讓我們定型該語言模型，並查看其是否可以正確地從使用者輸入預測意圖。

1. 在左側窗格中，選取 [定型作業]****。 選取 [+ 開始定型作業]****。

1. 在 [啟動定型作業]**** 對話方塊中，選取定型新模型的選項，將其命名為 `Clock`。 選取 [標準定型]**** 模式和預設 [資料分割]**** 選項。

1. 若要開始定型模型的流程，請選取 [定型]****。

1. 當定型完成時 (可能需要幾分鐘的時間)，作業 [狀態] **** 將會變更為 [訓練成功]****。

1. 選取 [模型效能]**** 頁面，然後選取 [時鐘]**** 模型。 檢閱整體和每個意圖評估衡量標準 (*精確度*、*召回率*及 *F1 分數*)，以及在定型時執行評估所產生的*混淆矩陣*(請注意，由於範例表達數目較少，並非所有意圖都可能包含在結果) 。

    > **注意事項** 若要深入了解評估計量，請參閱[文件](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

1. 請移至 [部署模型]**** 頁面，然後選取 [新增部署]****。

1. 在 [新增部署]**** 對話方塊中，選取 [建立新的部署名稱]****，然後輸入 `production`。

1. 選取 [模型]**** 欄位中的 [時鐘]**** 模型，然後選取 [部署]****。 此部署可能需要一些時間。

1. 部署模型之後，請選取 [測試部署]**** 頁面，然後在 [部署名稱]**** 欄位中選取**生產**部署。

1. 在空白文字方塊中輸入下列文字，然後選取 [執行測試]****：

    `what's the time now?`

    檢閱傳回的結果，請注意，其包含預測的意圖 (應該是 **GetTime**)，並包含信賴度分數，指出模型針對預測的意圖計算出的可能性。 [JSON] 索引標籤會顯示每個可能意圖的比較信賴度 (信賴分數最高的意圖就是預測的意圖)

1. 請清除文字方塊，然後使用下列文字執行另一個測試：

    `tell me the time`

    再次檢閱預測的意圖和信賴性分數。

1. 嘗試下列文字：

    `what's the day today?`

    但願模型預測 **GetDay** 意圖。

## 新增實體

到目前為止，您已定義一些對應至意圖的簡單表達。 大部分的實際應用程式都包含更複雜的表達，必須從中擷取特定資料實體，以取得意圖的更多內容。

### 新增已學習實體

最常見的實體類型是*已學習*實體，模型會學習根據範例來識別實體值。

1. 在 Language Studio 中，返回至 [結構描述定義]**** 頁面，然後在 [實體]**** 索引標籤上，選取 [&#65291;新增]**** 以新增實體。

1. 在 [新增實體]**** 對話方塊中，輸入實體名稱 `Location`，並確保已選取 [已了解]**** 索引標籤。 然後選取 [新增實體]****。

1. 建立 [位置]**** 實體之後，返回至 [資料標籤]**** 頁面。
1. 選取 **GetTime** 意圖，然後輸入下列新的範例表達：

    `what time is it in London?`

1. 新增表達之後，請選取 [London]**** 一字，然後在出現的下拉式清單中，選取 [位置]**** 以指出 "London" 是位置的範例。

1. 針對 **GetTime** 意圖新增另一個範例表達：

    `Tell me the time in Paris?`

1. 新增表達之後，請選取 [Paris]**** 一字，並將其對應至 [位置]**** 實體。

1. 針對 **GetTime** 意圖新增另一個範例表達：

    `what's the time in New York?`

1. 新增表達之後，請選取 [New York]**** 一字，並將其對應至 [位置]**** 實體。

1. 選取 [儲存變更]**** 以儲存新的表達。

### 新增*清單*實體

在某些情況下，實體的有效值可以限制為以特定字詞和同義字構成的清單；該清單可協助應用程式識別表達中實體的執行個體。

1. 在 Language Studio 中，返回 [結構描述定義]**** 頁面，然後在 [實體]**** 索引標籤上，選取 [&#65291;新增]**** 以新增實體。

1. 在 [新增實體]**** 對話方塊中，輸入實體名稱 `Weekday`，然後選取 [清單]**** 實體索引標籤。然後選取 [新增實體]****。

1. 在 [工作日]**** 實體頁面的 [已了解]**** 區段中，確保已選取 [非必要]****。 然後，在 [清單]**** 區段中，選取 [&#65291;新增清單]****。 然後輸入下列值和同義字，並選取 [儲存]****：

    | 列出索引鍵 | 同義字|
    |-------------------|---------|
    | `Sunday` | `Sun` |

1. 重複上一個步驟以新增下列清單元件：

    | 值 | 同義字|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. 新增並儲存清單值之後，返回 [資料標籤]**** 頁面。
1. 選取 **GetDate** 意圖，然後輸入下列新的範例表達：

    `what date was it on Saturday?`

1. 新增表達之後，請選取 [Saturday]****** 一字，然後在出現的下拉式清單中選取 [工作日]****。

1. 針對 **GetDate** 意圖新增另一個範例表達：

    `what date will it be on Friday?`

1. 當表達已被新增時，請將 [星期五] **** 對應至 [平日] **** 實體。

1. 針對 **GetDate** 意圖新增另一個範例表達：

    `what will the date be on Thurs?`

1. 當表達已新增時，請將 [星期四] **** 對應至 [平日] **** 實體。

1. 選取 [儲存變更]**** 以儲存新的表達。

### 新增*預先建置*的實體

Azure AI 語言服務會提供一組常用於交談應用程式中的*預先建置*實體。

1. 在 Language Studio 中，返回 [結構描述定義]**** 頁面，然後在 [實體]**** 索引標籤上，選取 [&#65291;新增]**** 以新增實體。

1. 在 [新增實體]**** 對話方塊中，輸入實體名稱 `Date`，然後選取 [預先建置]**** 實體索引標籤。然後選取 [新增實體]****。

1. 在 **[日期]** 實體頁面的 **[已了解]** 區段中，確保已選取 **[非必要]**。 然後，在 [預先建置]**** 區段中，選取 [&#65291;新增預先建置]****。

1. 在 [選取預先建置]**** 清單中，選取 [日期時間]****，然後選取 [儲存]****。
1. 新增預先建置的實體之後，返回 [資料標籤]**** 頁面
1. 選取 **GetDay** 意圖，然後輸入下列新的範例表達：

    `what day was 01/01/1901?`

1. 新增表達之後，請選取 [01/01/1901]******，然後在出現的下拉式清單中選取 [日期]****。

1. 針對 **GetDay** 意圖新增另一個範例表達：

    `what day will it be on Dec 31st 2099?`

1. 當已新增表達時，請將 **2099 年 12 月 31 日**對應至 [日期] **** 實體。

1. 選取 [儲存變更]**** 以儲存新的表達。

### 重新定型模型

既然您已修改結構描述，您必須重新定型並重新測試模型。

1. 在 [定型作業]**** 頁面上，選取 [開始定型作業]****。

1. 在 [啟動定型作業]**** 對話方塊中，選取[覆寫現有模型]**** 並指定 [時鐘]**** 模型。 請選取 [定型]**** 以定型模型。 如果出現提示，請確認您想要覆寫現有的模型。

1. 當定型完成時，作業**狀態**會更新為 [定型成功]****。

1. 選取 [模型效能]**** 頁面，然後選取 [時鐘]**** 模型。 檢閱評估計量 (*精確度*、*召回率*及 *F1 分數*)，以及在定型時執行評估所產生的*混淆矩陣* (請注意，由於範例表達數目較少，並非所有意圖都可能包含在結果中)。

1. 在 [部署模型]**** 頁面上，選取 [新增部署]****。

1. 在 [新增部署] **** 對話方塊中，選取 [覆寫現有的部署名稱] ****，然後選取 [生產環境] ****。

1. 選取 [模型]**** 欄位中的 [時鐘]**** 模型，然後選取 [部署]**** 以進行部署。 這可能需要一些時間。

1. 部署模型時，在 [測試部署]**** 頁面上，選取 [部署名稱]**** 欄位下的**生產**部署，然後使用下列文字進行測試：

    `what's the time in Edinburgh?`

1. 檢閱傳回的結果，其但願應該是預測 **GetTime** 意圖和**位置**實體，文字值為「Edinburgh」。

1. 嘗試測試下列表達：

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## 從用戶端應用程式使用模型

在實際專案中，您會反覆地完善意圖和實體、重新定型及重新測試，直到您對預測效能感到滿意為止。 然後，當您進行測試並滿意其預測效能時，您可以藉由呼叫其 REST 介面或執行階段專用 SDK，在用戶端應用程式中加以使用。

### 準備在 Visual Studio Code 中開發應用程式

您可使用 Visual Studio Code 開發語言理解應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

### 設定您的應用程式

已同時提供 C# 和 Python 的應用程式，以及您將用於測試摘要的範例文字檔案。 這兩個應用程式具有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 語言資源。

1. 在 Visual Studio Code 的 [Explorer]**** 窗格中，瀏覽至 **Labfiles/03-language** 資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **clock-client** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure AI 語言問題解答功能。
2. 以滑鼠右鍵按一下包含程式碼檔案的 **clock-client** 資料夾，然後開啟整合式終端。 然後針對您的語言偏好執行適當的命令，以安裝 Azure AI 語言交談語言理解 SDK 套件：

    **C#：**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**：

    ```
    pip install azure-ai-language-conversations
    ```

3. 在 [Explorer]**** 窗格中的 **clock-client** 資料夾中，開啟您慣用語言的組態檔

    - **C#**：appsettings.json
    - **Python**：.env
    
4. 更新組態值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)。
5. 儲存組態檔。

### 將程式碼新增至應用程式

現在您已準備好新增匯入必要 SDK 程式庫所需的程式碼、對已部署專案建立已驗證連線，以及提交問題。

1. 請注意，**clock-client** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：clock-client.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**：Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**：clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入預測端點和金鑰的程式碼。 然後尋找**建立語言服務模型的用戶端**註解，並新增下列程式碼來為您的語言服務應用程式建立預測用戶端：

    **C#**：Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**：clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. 請注意，**Main** 函式中的程式碼會提示使用者輸入，直到使用者輸入 "quit" 為止。 在此迴圈中，尋找**呼叫語言服務模型以取得意圖和實體**註解，並新增下列程式碼：

    **C#**：Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**：clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    語言服務應用程式的呼叫會傳回預測/結果，其中包含最高 (最可能) 意圖以及在輸入表達中偵測到的任何實體。 用戶端應用程式現在必須使用該預測來判斷和執行適當的動作。

1. 尋找**套用適合的動作**註解，並新增下列程式碼，以檢查應用程式所支援的意圖 (**GetTime**、**GetDate** 和 **GetDay**)，並判斷是否已偵測到任何相關的實體，然後再呼叫現有的函式來產生適當的回應。

    **C#**：Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**：clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. 儲存變更並返回 **clock-client** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    - **C#**：`dotnet run`
    - **Python**：`python clock-client.py`

    > **秘訣**：您可以使用終端工具列中的**最大化面板大小** (**^**) 圖示來查看更多的主控台文字。

1. 出現提示時，請輸入表達來測試應用程式。 例如，嘗試：

    *您好*

    *What time is it? (現在幾點？)*

    *What's the time in London? (倫敦現在幾點？)*

    *What's the date? (今天是幾月幾號？)*

    *What date is Sunday? (星期天是幾月幾號？)*

    *What day is it? (今天是什麼日子？)*

    *What day is 01/01/2025? (今天是 2025 年 1 月 1 日嗎？)*

    > **注意**：應用程式中的邏輯刻意簡單，而且有一些限制。 例如，取得時間時，只支援一組受限制的城市並忽略日光節約時間。 目標在於查看使用語言服務的典型模式範例，而您的應用程式在其中必須：
    >   1. 連線至預測端點。
    >   2. 提交表達以取得預測。
    >   3. 實作邏輯以適當地回應預測的意圖和實體。

1. 完成測試後，請輸入 *quit*。

## 清除資源

如果您已完成探索 Azure AI 語言服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 瀏覽至您在此實驗室中建立的 Azure AI 語言資源。
3. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

若要深入了解 Azure AI 語言中的交談語言理解，請參閱 [Azure AI 語言文件](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview)。
