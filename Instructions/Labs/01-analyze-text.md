---
lab:
  title: 分析文字
  description: 使用 Azure AI 語言分析文字，包括語言偵測、情感分析、關鍵片語擷取和實體辨識。
---

# 分析文字

**Azure AI 語言**支援文字分析，包括語言偵測、情感分析、關鍵片語擷取和實體辨識。

例如，假設旅遊機構想要處理已提交至公司網站的旅館評論。 他們可使用 Azure AI 語言來判斷用以撰寫每篇評論的語言、評論的情感 (正面、中立或負面)、可能指出評論中所討論主要主題的關鍵片語，以及具名實體，例如評論中提及的地點、地標或人物。 在本練習中，您將使用 Azure AI 語言 Python SDK 進行文字分析，根據此範例實作簡單的旅館評論應用程式。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發文字分析應用程式，包括：

- [適用於 Python 的 Azure AI 文字分析用戶端程式庫](https://pypi.org/project/azure-ai-textanalytics/)
- [適用於 .NET 的 Azure AI 文字分析用戶端程式庫](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [適用於 JavaScript 的 Azure AI 文字分析用戶端程式庫](https://www.npmjs.com/package/@azure/ai-text-analytics)

本練習大約需要 **30** 分鐘的時間。

## 佈建 *Azure AI 語言*資源

如果您的 Azure 訂用帳戶中還沒有 **Azure AI 語言服務**資源，則需要在訂用帳戶中佈建一個。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 選取 **[建立資源]**。
1. 在搜尋欄位中，搜尋 **語言服務**。 然後，從結果中選取 [語言服務]**** 底下的 [建立]****。
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
1. 在**資源管理**區段中檢視**金鑰和端點**頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 複製本課程的存放庫

您將可使用 Azure 入口網站中的 Cloud Shell，開發程式碼。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **提示**：當您將命令輸入到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## 設定您的應用程式

1. 在命令列窗格中，執行下列命令以檢視 **text-analysis** 資料夾中的程式碼檔案：

    ```
   ls -a -l
    ```

    這些檔案包括設定檔 (**.env**) 和程式碼檔案 (**text-analysis.py**)。 應用程式待分析的文字位於 **reviews** 子資料夾中。

1. 執行下列命令來建立 Python 虛擬環境，並安裝 Azure AI 語言文字分析 SDK 套件和其他必要套件：

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. 輸入下列命令以編輯應用程式組態檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 更新組態值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 新增程式碼以連線至您的 Azure AI 語言資源

1. 輸入下列命令以編輯應用程式程式碼檔案：

    ```
    code text-analysis.py
    ```

1. 檢閱現有程式碼。 您將新增程式碼以使用 AI 語言文字分析 SDK。

    > **秘訣**：當您將程式碼新增至程式碼檔案時，請確保縮排維持正確。

1. 在程式碼檔案頂端的現有命名空間參考下方，尋找 **Import namespaces** 註解，並新增下列程式碼以匯入您需要使用文字分析 SDK 的命名空間：

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. 在 **main** 函式中，您會發現已提供可從設定檔載入 Azure AI 語言服務端點和金鑰的程式碼。 然後尋找**使用端點和金鑰建立用戶端**註解，接著新增下列程式碼以建立文字分析 API 的用戶端：

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 儲存變更 (CTRL+S)，然後輸入下列命令以執行程式 (將 Cloud Shell 窗格最大化，並調整面板大小，以查看命令列窗格中的更多文字)：

    ```
   python text-analysis.py
    ```

1. 觀察輸出，因為程式碼應該執行且不會發生錯誤，並在 **reviews** 資料夾中顯示每個評論文字檔的內容。 應用程式已成功為文字分析 API 建立用戶端，但不會使用。 我們將在下節修正該問題。

## 新增程式碼來偵測語言

既然您已為 API 建立用戶端，讓我們使用來偵測用以撰寫每篇評論的語言。

1. 在程式碼編輯器中，尋找 **Get language** 註解。 然後，新增在每個評論文件中用於偵測語言所需的程式碼：

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **注意**：*在此範例中，每篇評論都會進行個別分析，導致對每個檔案個別呼叫服務。替代方法是建立一組文件並在單一呼叫中將其傳遞給服務。在這兩種方法中，服務的回應都是由一組文件所組成；這就是為何在上述的 Python 程式碼中，已指定了回應 ([0]) 中第一份 (且唯一的) 文件的索引。*

1. 儲存您的變更。 然後重新執行程式。
1. 觀察輸出，請注意這次已識別每篇評論的語言。

## 新增程式碼以評估情緒

*情感分析*是常用的技術，可將文字分類為*正面*或*負面*(或可能是*中立*或*混合*)。 其通常用來分析社交媒體文章、產品評論，以及文字情感可能提供實用見解的其他項目。

1. 在程式碼編輯器中，尋找 **Get sentiment** 註解。 然後，新增在每個評論文件中用於偵測情感所需的程式碼：

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 儲存您的變更。 然後關閉程式碼編輯器並重新執行程式。
1. 觀察輸出，請注意已偵測到評論的情感。

## 新增程式碼以識別關鍵片語

識別文字本文中的關鍵片語有助於判斷其所討論的主要主題。

1. 在程式碼編輯器中，尋找 **Get key phrases** 註解。 然後，新增在每個評論文件中用於偵測關鍵片語所需的程式碼：

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 儲存您的變更，然後重新執行程式。
1. 觀察輸出，請注意每個文件都包含關鍵片語，以提供評論內容的一些見解。

## 新增程式碼以擷取實體

通常，文件或其他文字本文會提及人物、地點、時段或其他實體。 文字分析 API 可以在您文字中偵測多種實體類別 (和子類別)。

1. 在程式碼編輯器中，尋找 **Get entities** 註解。 然後，新增用於識別每篇評論中提及的實體所需的程式碼：

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 儲存您的變更，然後重新執行程式。
1. 觀察輸出，請注意文字中偵測到的實體。

## 新增程式碼以擷取連結的實體

除了已分類的實體之外，文字分析 API 還可以偵測資料來源有已知連結的實體，例如 Wikipedia。

1. 在程式碼編輯器中，尋找 **Get linked entities** 註解。 然後，新增用以識別每篇評論中提及的連結實體所需的程式碼：

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 儲存您的變更，然後重新執行程式。
1. 觀察輸出，請注意已識別的連結實體。

## 清除資源

如果您已完成探索 Azure AI 語言服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 關閉 [Azure Cloud Shell] 窗格
1. 在 Azure 入口網站中，瀏覽至您在此實驗室中建立的 Azure AI 語言資源。
1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

如需使用 **Azure AI 語言**的詳細資訊，請參閱[文件](https://learn.microsoft.com/azure/ai-services/language-service/) (部分機器翻譯)。
