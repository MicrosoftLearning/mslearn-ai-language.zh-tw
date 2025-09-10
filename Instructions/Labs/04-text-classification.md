---
lab:
  title: 自訂文字分類
  description: 使用 Azure AI 語言將自訂分類套用至文字輸入。
---

# 自訂文字分類

Azure AI 語言提供多項 NLP 功能，包括關鍵片語識別、文字摘要和情感分析。 語言服務也提供自訂功能，例如自訂問題解答和自訂文字分類。

若要測試 Azure AI 語言服務的自訂文字分類，您可以使用 Language Studio 設定模型，然後使用 Python 應用程式來測試模型。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發文字分類應用程式，包括：

- [適用於 Python 的 Azure AI 文字分析用戶端程式庫](https://pypi.org/project/azure-ai-textanalytics/)
- [適用於 .NET 的 Azure AI 文字分析用戶端程式庫](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [適用於 JavaScript 的 Azure AI 文字分析用戶端程式庫](https://www.npmjs.com/package/@azure/ai-text-analytics)

此練習大約需要 **35 分鐘**。

## 佈建 *Azure AI 語言*資源

如果您的訂用帳戶中還沒有 **Azure AI 語言服務**資源，則必須加以佈建。 此外，使用自定文字分類時，必須啟用**自訂文字分類與擷取**功能。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 選取 **[建立資源]**。
1. 在搜尋欄位中，搜尋 **語言服務**。 然後，從結果中選取 [語言服務]**** 底下的 [建立]****。
1. 選取包含**自訂文字分類**的方塊。 選取 **[繼續建立您的資源]**。
1. 使用下列設定建立資源：
    - **訂用帳戶**：*您的 Azure 訂用帳戶*。
    - **資源群組**：*選取或建立資源群組*。
    - **區域**：*選擇下列其中一項：*\*
        - 澳大利亞東部
        - 印度中部
        - 美國東部
        - 美國東部 2
        - 北歐
        - 美國中南部
        - 瑞士北部
        - 英國南部
        - 西歐
        - 美國西部 2
        - 美國西部 3
    - **名稱**：*輸入唯一名稱*。
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **儲存體帳戶**: 新增儲存體帳戶
      - **儲存體帳戶名稱**: *輸入唯一的名稱*。
      - **儲存體帳戶類型**: 標準 LRS
    - **負責任 AI 注意事項**：已選取。

1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後前往資源群組。
1. 尋找您建立的儲存體帳戶並選取，然後確認 [帳戶種類]__ 為 **StorageV2**。 若是 v1，請在該資源頁面上升級儲存體帳戶種類。

## 設定使用者的角色型存取

> **注意**：若您略過此步驟，則在嘗試連線到您的自訂專案時，會發生 403 錯誤。 即使您是儲存體帳戶的擁有者，您目前的使用者必須仍擁有此角色，才能存取儲存體帳戶 Blob 資料。

1. 移至您在 Azure 入口網站中的儲存體帳戶頁面。
2. 在左側導覽功能表中選取 [存取控制 (IAM)]****。
3. 選取 [新增]**** 以新增角色指派，然後選擇儲存體帳戶上的 [儲存體 Blob 資料擁有者]**** 角色。
4. 在 [存取權指派對象為]**** 內，選取 [使用者、群組或服務主體]****。
5. 選取 [選取成員]****。
6. 選取您的使用者。 您可以在 [選取]**** 欄位中搜尋使用者名稱。

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

> **提示**：如果您收到未獲授權執行此作業的錯誤，您必須新增角色指派。 若要修正此錯誤，請為執行實驗室的使用者對儲存體帳戶新增「儲存體 Blob 資料參與者」角色。 如需更多詳細資料，請參閱[文件頁面](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)。

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

## 準備在 Cloud Shell 中開發應用程式

若要測試 Azure AI 語言服務的自訂文字分類功能，您將在 Azure Cloud Shell 中開發簡單的主控台應用程式。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-language/Labfiles/04-text-classification/Python/classify-text
    ```

## 設定您的應用程式

1. 在命令列窗格中，執行下列命令來檢視 **classify-text** 資料夾中的程式碼檔案：

    ```
   ls -a -l
    ```

    這些檔案包括設定檔 (**.env**) 和程式碼檔案 (**classify-text.py**)。 應用程式待分析的文字位於 **articles** 子資料夾中。

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

1. 更新設定值，以包含您所建立 Azure AI 語言資源的**端點**和**金鑰** (在 Azure 入口網站中 Azure AI 語言資源的 [金鑰和端點]**** 頁面上有提供)。檔案應該已包含文字分類模型的專案和部署名稱。
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 新增程式碼以分類文件

1. 輸入下列命令以編輯應用程式程式碼檔案：

    ```
    code classify-text.py
    ```

1. 檢閱現有程式碼。 您將新增程式碼以使用 AI 語言文字分析 SDK。

    > **秘訣**：當您將程式碼新增至程式碼檔案時，請確保縮排維持正確。

1. 在程式碼檔案頂端的現有命名空間參考下方，尋找 **Import namespaces** 註解，並新增下列程式碼以匯入您需要使用文字分析 SDK 的命名空間：

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. 在 **main** 函式中，您會發現已提供可從設定檔載入 Azure AI 語言服務端點和金鑰，以及專案和部署名稱的程式碼。 然後尋找 **Create client using endpoint and key** 註解，接著新增下列程式碼以建立文字分析用戶端：

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 您會發現現有程式碼會讀取 **articles** 資料夾中的所有檔案，並建立包含其內容的清單。 然後，尋找註解**取得分類**，並新增下列程式碼：

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

1. 儲存變更 (CTRL+S)，然後輸入下列命令以執行程式 (將 Cloud Shell 窗格最大化，並調整面板大小，以查看命令列窗格中的更多文字)：

    ```
   python classify-text.py
    ```

1. 觀察輸出。 應用程式應該列出每個文字檔的分類和信賴度分數。

## 清理

當您不再需要專案時，您可以從 Language Studio 的 [專案]**** 頁面上刪除。 您也可以在 [Azure 入口網站](https://portal.azure.com)中移除 Azure AI 語言服務和相關聯的儲存體帳戶。
