---
lab:
  title: 開發具備音訊功能的聊天應用程式
  description: 瞭解如何使用 Azure AI Foundry 來建置支援音訊輸入的生成式 AI 應用程式。
---

# 開發具備音訊功能的聊天應用程式

在此練習中，您會使用 *Phi-4-multimodal-instruct* 生成式 AI 模型，來產生對包含音訊檔案之提示的回應。 您將開發一款應用程式，透過使用 Azure AI Foundry 和 Python OpenAI SDK 來彙總客戶留下的語音訊息，為農產品供應商公司提供 AI 幫助。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發類似的應用程式，包括：

- [適用於 Python 的 Azure AI 專案](https://pypi.org/project/azure-ai-projects)
- [適用於 Python 的 OpenAI 程式庫](https://pypi.org/project/openai/)
- [適用於 Microsoft .NET 的 Azure AI 專案](https://www.nuget.org/packages/Azure.AI.Projects)
- [適用於 Microsoft .NET 的 Azure OpenAI 用戶端程式庫](https://www.nuget.org/packages/Azure.AI.OpenAI)
- [適用於 JavaScript 的 Azure AI 專案](https://www.npmjs.com/package/@azure/ai-projects)
- [適用於 TypeScript 的 Azure OpenAI 程式庫](https://www.npmjs.com/package/@azure/openai)

本練習大約需要 **30** 分鐘的時間。

## 建立 Azure AI Foundry 專案

讓我們先從在 Azure AI Foundry 專案中部署模型開始。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](../media/ai-foundry-home.png)

1. 在首頁的 [探索模型和功能]**** 區段中搜尋 `Phi-4-multimodal-instruct` 模型。我們會在專案中使用這個模型。
1. 在搜尋結果中選取 **Phi-4-multimodal-instruct** 模型以查看其詳細資料，然後在該模型的頁面頂端選取 [使用此模型]****。
1. 當系統提示您建立專案時，請輸入您專案的有效名稱，然後展開 [進階選項]****。
1. 選取**自訂**，然後為您的中樞指定下列設定：
    - **Azure AI Foundry 資源**：*Azure AI Foundry 資源的有效名稱*
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **區域**：*選取任何 **建議的 AI Foundry***\*

    > \*部分 Azure AI 資源受區域模型配額限制。 在練習後期，若超過配額限制，您可能需要在不同區域建立另一個資源。 您可以在 [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)中查看特定模型的最新區域可用性

1. 選取 [建立]****，並等候您的專案建立完成。

    可能需要一段時間才能完成作業。

1. 選取 [同意並繼續]**** 以同意模型條款，然後選取 [部署]**** 來完成 Phi 模型部署。

1. 建立專案後，模型詳細資訊將自動開啟。 記下模型部署的名稱，其應為 **Phi-4-multimodal-instruct**

1. 在左側瀏覽窗格中選取 [概觀]****，以查看專案的主頁面，其看起來如下：

    > **注意**：若顯示*權限不足*錯誤，請使用 [進行修正]**** 按鈕來解決此問題。

    ![Azure AI Foundry 專案概觀頁面的螢幕擷取畫面。](../media/ai-foundry-project.png)

## 建立用戶端應用程式

現在您已部署模型，您可以使用 Azure AI Foundry 和 Azure AI 模型推斷 SDK 來開發與模型聊天的應用程式。

> **提示**：您可以選擇使用 Python 或 Microsoft C# 開發解決方案。 請遵循您所選語言的相應章節中的說明進行操作。

### 準備應用程式組態

1. 在 Azure AI Foundry 入口網站中，檢視專案的**概觀**頁面。
1. 在 [專案詳細資料]**** 區域中，記下 **Azure AI Foundry 專案端點**。 請使用此端點在用戶端應用程式中連線到您的專案。
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
   git clone https://github.com/MicrosoftLearning/mslearn-ai-language
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-language/Labfiles/09-audio-chat/Python
    ````

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code .env
    ```

    檔案會在程式碼編輯器中開啟。

1. 在程式碼檔案中，將 **your_project_endpoint** 預留位置取代為您專案的連接字串 (從 Azure AI Foundry 入口網站中的專案 **概觀** 頁面複製)，並將 **your_model_deployment** 預留位置取代為您指派給 Phi-4-multimodal-instruct 模型部署的名稱。

1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

### 撰寫程式碼以連線至您的專案，並與模型聊天

> **提示**：新增程式碼時，請確保保持正確的縮排。

1. 輸入下列命令以編輯程式碼檔案：

    ```
   code audio-chat.py
    ```

1. 在程式碼檔案中，記下檔案頂端新增的現有語句，以匯入必要的 SDK 命名空間。 然後，在註解 **[新增參考]** 下，新增以下程式碼來參考您先前安裝之程式庫中的命名空間：

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

1. 在 **main** 函式中，在註解 [取得組態設定]**** 下，請注意程式碼會載入您在設定檔中定義的專案連接字串和模型部署名稱值。

1. 尋找 **Initialize the project client** 註解，並新增下列程式碼以連線至您的 Azure AI Foundry 專案：

    > **秘訣**：請小心維持程式碼的正確縮排層級。

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       ),
       endpoint=project_endpoint,
   )
    ```

1. 在註解 **[取得聊天用戶端]** 下，新增下列程式碼以建立用戶端物件來與模型聊天：

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### 撰寫程式碼以提交音訊型提示

提交提示之前，我們需要對要求的音訊檔案進行編碼。 然後，我們可以使用 LLM 的提示，將音訊資料附加至使用者的訊息。 請注意，程式碼包含迴圈，可讓使用者輸入提示，直到他們輸入「quit」為止。 

1. 在 **Encode the audio file** 註解下，輸入下列程式碼以準備下列音訊檔案：

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/avocados.mp4" title="對鱷梨的要求" width="150"></video>

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3"
   response = requests.get(file_path)
   response.raise_for_status()
   audio_data = base64.b64encode(response.content).decode('utf-8')
    ```

1. 在 **Get a response to audio input** 註解下，新增下列程式碼以提交提示：

    ```python
   # Get a response to audio input
   response = openai_client.chat.completions.create(
       model=model_deployment,
       messages=[
           {"role": "system", "content": system_message},
           { "role": "user",
               "content": [
               { 
                   "type": "text",
                   "text": prompt
               },
               {
                   "type": "input_audio",
                   "input_audio": {
                       "data": audio_data,
                       "format": "mp3"
                   }
               }
           ] }
       ]
   )
   print(response.choices[0].message.content)
    ```

1. **使用 CTRL+S** 命令，將您的變更儲存至程式代碼檔案。 您也可以視需要關閉程式代碼編輯器 （**CTRL+Q**）。

### 登入 Azure，然後執行應用程式

1. 在 Cloud Shell 命令行窗格中，輸入下列命令以登錄 Azure。

    ```
   az login
    ```

    **<font color="red">即使 Cloud Shell 工作階段已經過驗證，您還是必須登錄 Azure。</font>**

    > **注意**：在大部分情況下，只要使用 *az 登入*即可。 不過，如果您在多個租用戶中擁有訂用帳戶，您可能需要使用 *--tenant* 參數指定租用戶。 如需詳細資料，請參閱[使用 Azure CLI 以互動方式登入 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。
    
1. 出現提示時，請遵循指示，在新的索引標籤中開啟登入頁面，然後輸入所提供的驗證碼和您的 Azure 認證。 接著，在命令行中完成登入流程。如果出現提示，請選取包含 Azure AI Foundry 中樞的訂用帳戶。

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來執行應用程式：

    ```
   python audio-chat.py
    ```

1. 當系統給提示時，輸入下列提示 

    ```
   Can you summarize this customer's voice message?
    ```

1. 檢閱回應。

### 使用不同的音訊檔案

1. 請在應用程式程式碼的程式碼編輯器中，尋找您之前在 **Encode the audio file** 註解下方新增的程式碼。 然後修改檔案路徑 URL，如下所示，以針對要求使用不同的音訊檔案 (在檔案路徑後方留下現有程式碼)：

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3"
    ```

    新檔案聽起來像這樣：

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/fresas.mp4" title="Strawberries 請求" width="150"></video>

 1. **使用 CTRL+S** 命令，將您的變更儲存至程式代碼檔案。 您也可以視需要關閉程式代碼編輯器 （**CTRL+Q**）。

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來執行應用程式：

    ```
   python audio-chat.py
    ```

1. 當系統提示時，輸入下列提示： 
    
    ```
   Can you summarize this customer's voice message? Is it time-sensitive?
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
