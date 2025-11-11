---
lab:
  title: 建立問題解答解決方案
  description: 使用 Azure AI 語言建立自訂問題解答解決方案。
---

# 建置問題解答解決方案

最常見的交談案例之一，是透過常見問題 (FAQ) 知識庫提供支援。 許多組織都會將常見問題發佈為文件或網頁，這對一組小型問答很適合，但搜尋大型文件可能困難且耗時。

**Azure AI 語言**包含一項*問題解答*功能，讓您能建立可使用自然語言輸入查詢的問題和答案配對知識庫，而且 Bot 最常使用該服務來查閱使用者所提交問題的答案資源。 在本練習中，您將使用 Azure AI 語言 Python SDK 進行文字分析，以實作簡單的問題解答應用程式。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發問題解答應用程式，包括：

- [適用於 Python 的 Azure AI 語言服務問題解答用戶端程式庫](https://pypi.org/project/azure-ai-language-questionanswering/)
- [適用於 .NET 的 Azure AI 語言服務問題解答用戶端程式庫](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

此練習大約需要 **20 分鐘**。

## 佈建 *Azure AI 語言*資源

如果您的訂用帳戶中還沒有 **Azure AI 語言服務**資源，則必須加以佈建。 此外，若要建立並託管知識庫以回答問題，您必須啟用「問題解答」**** 功能。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 選取 **[建立資源]**。
1. 在搜尋欄位中，搜尋 **語言服務**。 然後，從結果中選取 [語言服務]**** 底下的 [建立]****。
1. 選取 **自訂問題解答** 區塊。 選取 **[繼續建立您的資源]**。 您必須輸入下列設定：

    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組*。
    - **區域**：*選擇任何可用的位置*
    - **名稱**：輸入唯一名稱**
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **Azure 搜尋服務區域**：*在全球區域中，選擇和您的語言資源相同的位置*
    - **Azure 搜尋定價層**：免費 (F) (*若此階層不可用，則選取 [基本 (B)]*)
    - **負責任 AI 注意事項**：*同意*

1. 選取 [檢閱 + 建立]****，然後選取 [建立]****。

    > **注意** 自訂問題解答會使用 Azure 搜尋服務來編制問題與解答知識庫的索引和查詢。

1. 等候部署完成，然後移至資源。
1. 在**資源管理**區段中檢視**金鑰和端點**頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 建立問題解答專案

若要在 Azure AI 語言資源中建立問題回答的知識庫，您可以使用 Language Studio 入口網站來建立問題解答專案。 在此情況下，您將建立知識庫，其中包含 [Microsoft Learn](https://learn.microsoft.com/training/) 的問題和解答。

1. 在新的瀏覽器索引標籤中，開啟位於 [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/) 的 Language Studio 入口網站，然後使用與您 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 若系統提示您選擇語言資源，請選取下列設定：
    - **Azure 目錄**：包含您訂用帳戶的 Azure 目錄。
    - **Azure 訂用帳戶**：您的 Azure 訂用帳戶。
    - **資源類型**：語言
    - **資源名稱**：您先前建立的 Azure AI 語言資源。

    如果您<u>未</u>收到選擇語言資源的提示，可能是因為您的訂用帳戶中有多個語言資源；在此情況下：

    1. 在頁面頂端的列上，選取 [設定 (&#9881;)]**** 按鈕。
    2. 在 [設定]**** 分頁上，檢視 [資源]**** 索引標籤。
    3. 選取您剛才建立的語言資源，然後按一下 [切換資源]****。
    4. 在頁面頂端，按一下 [Language Studio]**** 以回到 Language Studio 首頁。

1. 在入口網站頂端的 [新建]**** 功能表中，選取 [自訂問題解答]****。
1. 在 ***建立專案** 精靈的 **選擇語言設定** 頁面上，選取 **為所有專案選擇語言**選項，然後選取 **英文** 作為語言。 然後選取**下一步**。
1. 在 [輸入基本資訊]**** 頁面上，輸入下列詳細資料：
    - **名稱** `LearnFAQ`
    - **描述**：`FAQ for Microsoft Learn`
    - **未傳回任何答案時的預設答案**：`Sorry, I don't understand the question`
1. 選取 [下一步]。
1. 在 [檢閱和完成]**** 頁面上，選取 [建立專案]****。

## 將來源新增至知識庫

您可以從頭開始建立知識庫，但從現有的常見問題頁面或文件匯入問題和解答，也是常見的作法。 在此情況下，您將從 Microsoft Learn 的現有常見問題網頁匯入資料，而且您也會匯入一些預先定義的「閒聊」問題和解答，以支援常見的對話交談。

1. 在問題解答專案的 [管理來源]**** 頁面上，於 [&#9547; 新增來源]**** 清單中，選取 [URL]****。 然後在 [新增 URL]**** 對話方塊中，選取 [&#9547; 新增 url]**** 並設定下列名稱和 URL，再選取 [全部新增]**** 以將其新增至知識庫：
    - **名稱**：`Learn FAQ Page`
    - **URL**：`https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
1. 在問題解答專案的 [管理來源]**** 頁面上，於 [&#9547; 新增來源]**** 清單中，選取 [Chitchat]****。 在 [新增閒聊]**** 對話方塊中，選取 [好記]****，然後選取 [新增閒聊]****。

## 編輯知識庫

您的知識庫已填入來自 Microsoft Learn 常見問題的問答組，並補充一組交談*閒聊*問題和答案配對。 您可以藉由新增其他問答配對來擴充知識庫。

1. 在 Language Studio 的「LearnFAQ」**** 專案中，選取 [編輯知識庫]**** 頁面，以查看現有的問題與答案配對 (如果顯示秘訣，請閱讀秘訣並選擇 [了解]**** 以關閉，或選取 [略過所有]****)
1. 在知識庫的 [問題答案組合]**** 索引標籤上選取 [&#65291;]****，並使用下列設定建立新的問題答案組合：
    - **來源**： `https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
    - **問題**：`What are the different types of modules on Microsoft Learn?`
    - **答案**：`Microsoft Learn offers various types of training modules, including role-based learning paths, product-specific modules, and hands-on labs. Each module contains units with lessons and knowledge checks to help you learn at your own pace.`
1. 選取**完成**。
1. 在建立以下問題：**Microsoft Learn 有哪些不同類型的模組？** 的目標頁面上，展開 [**替代問題**]。 然後新增替代問題 `How are training modules organized?`。

    在某些情況下，有需要讓使用者能夠追蹤答案，方法是透過建立「多回合」** 交談，以便使用者能反復精簡問題以取得所需的答案。

1. 在您為模組類型問題輸入的答案底下，展開 [**後續提示**]，並新增以下後續提示：
    - **提示中向使用者顯示的文字**：`Learn more about training`。
    - 選取 [建立新配對的連結]**** 索引標籤，然後輸入這段文字：`You can explore modules and learning paths on the [Microsoft Learn training page](https://learn.microsoft.com/training/).`
    - 選取 [僅在內容流程中顯示]****。 此選項可確保答案只會在原始模組問題類型的跟進問題內容中傳回。
1. 選取 [新增提示]****。

## 定型和測試知識庫

既然您擁有知識庫，就可以在 Language Studio 中測試知識庫。

1. 選取左側 [問題答案組合]**** 索引標籤底下的 [儲存]**** 按鈕，以儲您對存知識庫的變更。
1. 儲存變更之後，選取 [測試]**** 按鈕以開啟測試窗格。
1. 在測試窗格中，在頂端取消選取 [包含簡短回應]**** (如果尚未取消選取)。 然後在底部輸入訊息 `Hello`。 應該傳回適當的回應。
1. 在測試窗格底部輸入訊息 `What is Microsoft Learn?`。 應該會從常見問題集中傳回適當的回應。
1. 輸入訊息 `Thanks!` 應會傳回適當的閒聊回應。
1. 輸入訊息 `What are the different types of modules on Microsoft Learn?`。 您建立的答案應該會連同後續提示連結一起傳回。
1. 選取**深入瞭解訓練**跟進連結。 系統應會傳回具有訓練頁面連結的跟進答案。
1. 完成知識庫測試之後，關閉測試窗格。

## 部署知識庫

知識庫提供了後端服務，用戶端應用程式可以使用該後端服務來回答問題。 現在您已準備好發佈知識庫，並從用戶端存取其 REST 介面。

1. 在 Language Studio 的 **LearnFAQ** 專案中，從左側導覽功能表中選取 **部署知識庫** 頁面。
1. 在頁面頂端，選取 [部署]****。 然後選取 [部署]**** 以確認您想要部署該知識庫。
1. 部署完成時，選取 [取得預測 URL]**** 以檢視知識庫的 REST 端點，並記下範例要求中所包含的參數：
    - **projectName**：您的專案名稱 (應該是 *LearnFAQ*)
    - **deploymentName**：您的部署名稱 (應該是 *production*)
1. 關閉 [預測 URL] 對話方塊。

## 準備在 Cloud Shell 中開發應用程式

您將在 Azure 入口網站中使用 Cloud Shell 開發問題解答應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **秘訣**：當您將命令輸入到 Cloud Shell 時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## 設定您的應用程式

1. 在命令列窗格中，執行下列命令以檢視 **qna-app** 資料夾中的程式碼檔案：

    ```
   ls -a -l
    ```

    這些檔案包括設定檔 (**.env**) 和程式碼檔案 (**qna-app.py**)。

1. 執行下列命令來建立 Python 虛擬環境，並安裝 Azure AI 語言問題解答 SDK 套件和其他必要套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. 輸入下列命令，編輯設定檔：

    ```
    code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中更新其包含的設定值，以反映您所建立 Azure AI 語言資源的**端點**和驗證**金鑰** (在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上有提供)。 您部署知識庫的專案名稱和部署名稱也應會在此檔案中。
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 將程式碼新增至您的知識庫

1. 輸入下列命令以編輯應用程式程式碼檔案：

    ```
    code qna-app.py
    ```

1. 檢閱現有程式碼。 您將新增程式碼以使用知識庫。

    > **秘訣**：當您將程式碼新增至程式碼檔案時，請確保縮排維持正確。

1. 在程式碼檔案中，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用問題解答 SDK 所需的命名空間：

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. 在 **main** 函式中，您會發現已提供可從設定檔載入 Azure AI 語言服務端點和金鑰的程式碼。 然後尋找 **Create client using endpoint and key** 註解，接著新增下列程式碼以建立問題解答用戶端：

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 Main 函式中，尋找 **Submit a question and display the answer** 註解，並新增下列程式碼以重複讀取命令行中的問題、將其提交至服務，以及顯示答案的詳細資料：

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. 儲存變更 (CTRL+S)，然後輸入下列命令以執行程式 (將 Cloud Shell 窗格最大化，並調整面板大小，以查看命令列窗格中的更多文字)：

    ```
   python qna-app.py
    ```

1. 出現提示時，請輸入要提交至問題解答專案的問題；例如 `What is a learning path?`。
1. 檢閱傳回的答案。
1. 詢問更多問題。 完成時，請輸入 `quit`。

## 清除資源

如果您已完成探索 Azure AI 語言服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 關閉 [Azure Cloud Shell] 窗格
1. 在 Azure 入口網站中，瀏覽至您在此實驗室中建立的 Azure AI 語言資源。
1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

若要深入了解 Azure AI 語言中的問題解答，請參閱 [Azure AI 語言文件](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview)。
