---
lab:
  title: 開發具備音訊功能的聊天應用程式
  description: 瞭解如何使用 Azure AI Foundry 來建置支援音訊輸入的生成式 AI 應用程式。
---

# 開發具備音訊功能的聊天應用程式

在此練習中，您會使用 *Phi-4-multimodal-instruct* 生成式 AI 模型，來產生對包含音訊檔案之提示的回應。 您將開發一個應用程式，透過使用 Azure AI Foundry 和 Azure AI 模型推斷服務，為雜貨店中的新鮮產品提供 AI 協助。

本練習大約需要 **30** 分鐘的時間。

## 建立 Azure AI Foundry 專案

讓我們從建立 Azure AI Foundry 專案開始。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](../media/ai-foundry-home.png)

1. 在首頁中，選取 **+ 建立專案**。
1. 請在** [建立專案**精靈] 中，輸入專案有效名稱。如果建議使用現有的中樞，請先選擇建立新的中樞選項。 然後審查 Azure 資源，將會自動建立，以便支援中樞和專案。
1. 選取**自訂**，然後為您的中樞指定下列設定：
    - **中樞名稱**：*請提供有效的中樞名稱*
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **區域**：選取下列任何區域\*：
        - 美國東部
        - 美國東部 2
        - 美國中北部
        - 美國中南部
        - 瑞典中部
        - 美國西部
        - 美國西部 3
    - **連接 Azure AI 服務或 Azure OpenAI**：*[建立新的 AI 服務資源]*
    - **連接 Azure AI 搜尋服務**：跳過連接

    > \* 撰寫本文時，我們在此練習中將使用的 Microsoft *Phi-4-multimodal-instruct* 模型已在下列區域推出。 您可以在 [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)中查看特定模型的最新區域可用性。 如果練習稍後達到區域配額限制，您可能需要在不同的區域中建立另一個資源。

1. 選取**下一步**並檢閱您的設定。 然後選取**建立**並等待該流程完成。
1. 建立專案後，關閉顯示的所有提示並檢閱 Azure AI Foundry 入口網站中的專案頁面，該頁面應類似於下圖：

    ![Azure AI Foundry 入口網站中 Azure AI 專案詳細資料的螢幕螢幕擷取畫面。](../media/ai-foundry-project.png)

## 部署多模式模型

現在您已準備好部署可支援音訊型輸入的多模式模型。 您可以選擇數個模型，包括 OpenAI *gpt-4o* 模型。 在此練習中，我們將使用 *Phi-4-multimodal-instruct* 模型。

1. 在工具列頂部右上方的 Azure AI Foundry 專案頁面中，使用 **預覽功能** (**amp;#9215;**) 圖示，以確保 **部署模型至 Azure AI 模型推斷服務** 功能已啟用。 這項功能可確保模型部署可供 Azure AI 推斷服務使用，您將可在應用程式程式碼中使用這項功能。
1. 在專案左側窗格中的 [我的資產]**** 區段中，選取 [模型 + 端點]**** 頁面。
1. 在 [模型 + 端點]**** 頁面中，於 [模型部署]**** 索引標籤的 [+ 部署模型]**** 功能表中，選取 [部署基本模型]****。
1. 在清單中搜尋 **Phi-4-multimodal-instruct** 模型，然後選取並確認。
1. 如果出現提示，請同意授權合約，然後透過在部署詳細資料中選取 [自訂]**** 來使用以下設定部署模型：
    - **部署名稱**：*模型部署的有效名稱*
    - **部署類型**：全球標準
    - **部署詳細資料**： *使用預設設定*
1. 等待部署配置狀態為 [完成]****。

## 建立用戶端應用程式

現在您已部署模型，您可以在用戶端應用程式中使用部署。

> **提示**：您可以選擇使用 Python 或 Microsoft C# 開發解決方案。 請遵循您所選語言的相應章節中的說明進行操作。

### 準備應用程式組態

1. 在 Azure AI Foundry 入口網站中，檢視專案的**概觀**頁面。
1. 在 [專案詳細資料]**** 區域中，記下 [專案連接字串]****。 您將使用此連接字串連線到用戶端應用程式中的專案。
1. 開啟一個新的瀏覽器索引標籤（保持 Azure AI Foundry 入口網站在現有索引標籤中開啟）。 然後在新索引標籤中，瀏覽到 `https://portal.azure.com` 的 [Azure 入口網站](https://portal.azure.com)；如果出現提示，請使用您的 Azure 認證登入。

    關閉任何歡迎通知，以查看 Azure 入口網站 首頁。

1. 使用頁面頂部搜尋欄右側的 **[\>_]** 按鈕在 Azure 入口網站中建立一個新的 Cloud Shell，並選擇 ***PowerShell 環境*** (訂用帳戶中沒有儲存體)。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。 您可以調整或最大化此窗格的大小，以便更輕鬆地使用。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/c-sharp
    ```

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    **Python**

    ```
    code .env
    ```

    **C#**

    ```
    code appsettings.json
    ```

    檔案會在程式碼編輯器中開啟。

1. 在程式碼檔案中，將 **your_project_connection_string** 預留位置替換為您的專案的連接字串（從 Azure AI Foundry 入口網站中的專案 **概觀** 頁面複製），並將 **your_model_deployment** 預留位置替換為您指派給 Phi-4-multimodal-instruct 模型部署的名稱。

1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

### 撰寫程式碼以連線至您的專案，並與模型聊天

> **提示**：新增程式碼時，請確保保持正確的縮排。

1. 輸入以下命令，編輯已提供的\程式碼檔案：

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. 在程式碼檔案中，記下檔案頂端新增的現有語句，以匯入必要的 SDK 命名空間。 然後，在註解 **[新增參考]** 下，新增以下程式碼來參考您先前安裝之程式庫中的命名空間：

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        AudioContentItem,
        InputAudio,
        AudioContentFormat,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. 在 **main** 函式中，在註解 [取得組態設定]**** 下，請注意程式碼會載入您在設定檔中定義的專案連接字串和模型部署名稱值。

1. 在註解 **[初始化專案用戶端]** 下，新增以下程式碼，使用您目前登入的 Azure 認證連接到您的 Azure AI Foundry 專案：

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. 在註解 **[取得聊天用戶端]** 下，新增下列程式碼以建立用戶端物件來與模型聊天：

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### 撰寫程式碼以提交 URL 型音訊型提示

1. 在 **audio-chat.py** 檔案的程式碼編輯器中，於迴圈區段的 [取得音訊輸入的回應]**** 註解底下，新增下列程式碼以提交包含下列音訊的提示：

    <video controls src="../media/manzanas.mp4" title="對蘋果的要求" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. **使用 CTRL+S** 命令，將您的變更儲存至程式代碼檔案。 您也可以視需要關閉程式代碼編輯器 （**CTRL+Q**）。

1. 請在 Cloud Shell 命令列窗格中，輸入下列命令，以便執行應用程式：

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. 當系統給提示時，輸入下列提示：`What is this customer saying in English?`

1. 檢閱回應。

### 使用不同提示

1. 請在應用程式程式代碼的程式代碼編輯器中，前往 [迴圈] 區段中，尋找您之前在註解那邊，**[取得音訊輸入的回應] **新增的程式碼。 然後修改程式碼，如下所示，以便選取不同音訊檔案：

    <video controls src="../media/caomei.mp4" title="Strawberries 請求" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. **使用 CTRL+S** 命令，將您的變更儲存至程式代碼檔案。 您也可以視需要關閉程式代碼編輯器 （**CTRL+Q**）。

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來執行應用程式：

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. 當系統提示時，輸入下列提示：

    ```
    A customer left this voice message, can you summarize it?
    ```

1. 檢閱回應。 然後輸入 `quit`以結束程式。

    > **注意**：在這個簡單的應用程式中，我們還沒有實現保留交談記錄的邏輯；因此模型會將每個提示視為一個新要求，而沒有前一個提示的上下文。

1. 您可以繼續執行應用程式，選擇不同提示類型，再嘗試不同提示。 完成後，輸入 `quit` 可結束程式。

    如果您有時間，就可以修改程式碼，以便使用不同系統的提示，也能使用自己網路上可存取的音訊檔案。

    > **注意**：在這個簡單的應用程式中，我們還沒有實現保留交談記錄的邏輯；因此模型會將每個提示視為一個新要求，而沒有前一個提示的上下文。

## 摘要

您可以在本練習中，使用 Azure AI Foundry 和 Azure OpenAI SDK，建立用戶端應用程式，該應用程式使用 DALL-E 模型，產生音訊。

## 清理

如果您已完成 Azure AI 語音探索，您應該刪除在本練習中建立的資源，以避免產生不必要的 Azure 成本。。

1. 返回包含 Azure 入口網站的瀏覽器索引標籤 (或在新的瀏覽器索引標籤中重新開啟位於`https://portal.azure.com`的 [Azure 入口網站](https://portal.azure.com))，並檢視您在其中部署本練習所用資源的資源群組內容。
1. 在工具列上，選取 [刪除資源群組]****。
1. 輸入資源群組名稱並確認您想要將其刪除。
