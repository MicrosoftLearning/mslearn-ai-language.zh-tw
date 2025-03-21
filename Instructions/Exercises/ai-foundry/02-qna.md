---
lab:
  title: 建置問題解答解決方案
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# 建置問題解答解決方案

最常見的交談案例之一，是透過常見問題 (FAQ) 知識庫提供支援。 許多組織都會將常見問題發佈為文件或網頁，這對一組小型問答很適合，但搜尋大型文件可能困難且耗時。

**Azure AI 語言**包含一項*問題解答*功能，讓您能建立可使用自然語言輸入查詢的問題和答案配對知識庫，而且 Bot 最常使用該服務來查閱使用者所提交問題的答案資源。

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
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 建立問題解答專案

若要在 Azure AI 語言資源中建立問題回答的知識庫，您可以使用 Language Studio 入口網站來建立問題解答專案。 在此情況下，您將建立知識庫，其中包含 [Microsoft Learn](https://docs.microsoft.com/learn) 的問題和解答。

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
    - **URL**：`https://docs.microsoft.com/en-us/learn/support/faq`
1. 在問題解答專案的 [管理來源]**** 頁面上，於 [&#9547; 新增來源]**** 清單中，選取 [Chitchat]****。 在 [新增閒聊]**** 對話方塊中，選取 [好記]****，然後選取 [新增閒聊]****。

## 編輯知識庫

您的知識庫已填入來自 Microsoft Learn 常見問題的問答組，並補充一組交談*閒聊*問題和答案配對。 您可以藉由新增其他問答配對來擴充知識庫。

1. 在 Language Studio 的「LearnFAQ」**** 專案中，選取 [編輯知識庫]**** 頁面，以查看現有的問題與答案配對 (如果顯示秘訣，請閱讀秘訣並選擇 [了解]**** 以關閉，或選取 [略過所有]****)
1. 在知識庫的 [問題答案組合]**** 索引標籤上選取 [&#65291;]****，並使用下列設定建立新的問題答案組合：
    - **來源**： `https://docs.microsoft.com/en-us/learn/support/faq`
    - **問題**：`What are Microsoft credentials?`
    - **答案**：`Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. 選取**完成**。
1. 在建立的「什麼是 Microsoft 認證？」**** 問題的頁面中，展開 [替代問題]****。 然後新增替代問題 `How can I demonstrate my Microsoft technology skills?`。

    在某些情況下，有需要讓使用者能夠追蹤答案，方法是透過建立「多回合」** 交談，以便使用者能反復精簡問題以取得所需的答案。

1. 在您為認證問題輸入的答案底下，展開 [後續提示]****，並新增以下後續提示：
    - **提示中向使用者顯示的文字**：`Learn more about credentials`。
    - 選取 [建立新配對的連結]**** 索引標籤，然後輸入這段文字：`You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - 選取 [僅在內容流程中顯示]****。 此選項可確保答案只會在原始認證問題的後續問題內容中傳回。
1. 選取 [新增提示]****。

## 定型和測試知識庫

既然您擁有知識庫，就可以在 Language Studio 中測試知識庫。

1. 選取左側 [問題答案組合]**** 索引標籤底下的 [儲存]**** 按鈕，以儲您對存知識庫的變更。
1. 儲存變更之後，選取 [測試]**** 按鈕以開啟測試窗格。
1. 在測試窗格中，在頂端取消選取 [包含簡短回應]**** (如果尚未取消選取)。 然後在底部輸入訊息 `Hello`。 應該傳回適當的回應。
1. 在測試窗格底部輸入訊息 `What is Microsoft Learn?`。 應該會從常見問題集中傳回適當的回應。
1. 輸入訊息 `Thanks!` 應會傳回適當的閒聊回應。
1. 輸入訊息 `Tell me about Microsoft credentials`。 您建立的答案應該會連同後續提示連結一起傳回。
1. 選取 [深入了解認證]**** 後續連結。 應該會傳回具有認證頁面連結的後續答案。
1. 完成知識庫測試之後，關閉測試窗格。

## 部署知識庫

知識庫提供了後端服務，用戶端應用程式可以使用該後端服務來回答問題。 現在您已準備好發佈知識庫，並從用戶端存取其 REST 介面。

1. 在 Language Studio 的 **LearnFAQ** 專案中，從左側導覽功能表中選取 **部署知識庫** 頁面。
1. 在頁面頂端，選取 [部署]****。 然後選取 [部署]**** 以確認您想要部署該知識庫。
1. 部署完成時，選取 [取得預測 URL]**** 以檢視知識庫的 REST 端點，並記下範例要求中所包含的參數：
    - **projectName**：您的專案名稱 (應該是 *LearnFAQ*)
    - **deploymentName**：您的部署名稱 (應該是 *production*)
1. 關閉 [預測 URL] 對話方塊。

## 準備在 Visual Studio Code 中開發應用程式

您將使用 Visual Studio Code 來開發問題解答應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 設定您的應用程式

已同時提供 C# 和 Python 的應用程式，以及您將用於測試摘要的範例文字檔案。 這兩個應用程式具有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 語言資源。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **Labfiles/02-qna** 資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **qna-app** 資料夾。 每個資料夾都包含應用程式的特定語言檔案，您將在其中整合 Azure AI 語言問題解答功能。
2. 以滑鼠右鍵按一下包含程式碼檔案的 **qna-app** 資料夾，然後開啟整合式終端。 然後針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 語言問題解答 SDK 套件：

    **C#：**

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**：

    ```
    pip install azure-ai-language-questionanswering
    ```

3. 在 [總管]**** 窗格中的 **qna-app** 資料夾中，開啟您慣用語言的組態檔

    - **C#**：appsettings.json
    - **Python**：.env
    
4. 更新組態值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)。 您部署知識庫的專案名稱和部署名稱也應會在此檔案中。
5. 儲存組態檔。

## 將程式碼新增至應用程式

現在您已準備好新增匯入必要 SDK 程式庫所需的程式碼、對已部署專案建立已驗證連線，以及提交問題。

1. 請注意，**qna-app** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：qna-app.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**：qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入 Azure AI 語言服務端點和金鑰的程式碼。 然後尋找**使用端點和金鑰建立用戶端**註解，接著新增下列程式碼以建立文字分析 API 的用戶端：

    **C#**：Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**：qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 **Main** 函式中，尋找註解 **Submit a question and display the answer**，並新增下列程式碼以重複讀取命令行中的問題、將其提交至服務，以及顯示答案的詳細資料：

    **C#**：Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (true)
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            if (user_question.ToLower() == "quit")
                break;
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**：qna-app.py

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

1. 儲存變更並返回 **qna-app** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    - **C#**：`dotnet run`
    - **Python**：`python qna-app.py`

    > **秘訣**：您可以使用終端工具列中的**最大化面板大小** (**^**) 圖示來查看更多的主控台文字。

1. 出現提示時，請輸入要提交至問題解答專案的問題；例如 `What is a learning path?`。
1. 檢閱傳回的答案。
1. 詢問更多問題。 完成時，請輸入 `quit`。

## 清除資源

如果您已完成探索 Azure AI 語言服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 瀏覽至您在此實驗室中建立的 Azure AI 語言資源。
3. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

若要深入了解 Azure AI 語言中的問題解答，請參閱 [Azure AI 語言文件](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview)。
