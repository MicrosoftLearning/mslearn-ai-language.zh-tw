---
lab:
  title: 擷取自訂實體
  description: 使用 Azure AI 語言訓練模型，以從文字輸入擷取自訂實體。
---

# 擷取自訂實體

除了其他自然語言處理功能之外，Azure AI 語言服務還可讓您定義自訂實體，並從文字中擷取實體的執行個體。

為了測試自訂實體擷取，我們將建立模型，並透過 Azure AI 語言工作室加以訓練，然後使用 Python 應用程式進行測試。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發文字分類應用程式，包括：

- [適用於 Python 的 Azure AI 文字分析用戶端程式庫](https://pypi.org/project/azure-ai-textanalytics/)
- [適用於 .NET 的 Azure AI 文字分析用戶端程式庫](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [適用於 JavaScript 的 Azure AI 文字分析用戶端程式庫](https://www.npmjs.com/package/@azure/ai-text-analytics)

此練習大約需要 **35 分鐘**。

## 佈建 *Azure AI 語言*資源

如果您的訂用帳戶中還沒有 **Azure AI 語言服務**資源，則必須加以佈建。 此外，使用自定文字分類時，必須啟用**自訂文字分類與擷取**功能。

1. 在瀏覽器中，在 `https://portal.azure.com` 中開啟 Azure 入口網站，然後使用您的 Microsoft 帳戶登入。
1. 選取 [建立資源]**** 按鈕，搜尋「語言」**，然後建立 [語言服務]**** 資源。 在 [選取其他功能]** 頁面中，選取包含 [自訂具名實體辨識]**** 的自訂功能。 使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選取或建立資源群組*
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
    - **名稱**：輸入唯一名稱**
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **儲存體帳戶**：新增儲存體帳戶：
      - **儲存體帳戶名稱**: *輸入唯一的名稱*。
      - **儲存體帳戶類型**: 標準 LRS
    - **負責任 AI 注意事項**：已選取。

1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 設定使用者的角色型存取

> **注意**：如果您略過此步驟，嘗試連線到您的自訂專案時，將發生 403 錯誤。 即使您是儲存體帳戶的擁有者，您目前的使用者必須仍擁有此角色，才能存取儲存體帳戶 Blob 資料。

1. 移至您在 Azure 入口網站中的儲存體帳戶頁面。
2. 在左側導覽功能表中選取 [存取控制 (IAM)]****。
3. 選取 **[新增]** 以新增角色指派，然後選擇儲存體帳戶上的**儲存體 Blob 資料參與者**角色。
4. 在 [存取權指派對象為]**** 內，選取 [使用者、群組或服務主體]****。
5. 選取 [選取成員]****。
6. 選取您的使用者。 您可以在 [選取]**** 欄位中搜尋使用者名稱。

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

> **提示**：如果您收到未獲授權執行此作業的錯誤，您必須新增角色指派。 若要修正此錯誤，請為執行實驗室的使用者對儲存體帳戶新增「儲存體 Blob 資料參與者」角色。 如需更多詳細資料，請參閱[文件頁面](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)。

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
1. 在 **活動** 窗格中，請注意此文件將會新增至資料集以訓練模型。
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

## 準備在 Cloud Shell 中開發應用程式

若要測試 Azure AI 語言服務的自訂實體擷取功能，您將在 Azure Cloud Shell 中開發簡單的主控台應用程式。

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
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

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

1. 您會發現現有的程式碼會讀取 **ads** 資料夾中的所有檔案，並建立包含其內容的清單。 尋找 **Extract entities** 註解並新增下列程式碼：

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

1. 儲存變更 (CTRL+S)，然後輸入下列命令以執行程式 (將 Cloud Shell 窗格最大化，並調整面板大小，以查看命令列窗格中的更多文字)：

    ```
   python custom-entities.py
    ```

1. 觀察輸出。 應用程式應該列出每個文字檔中所找到實體的詳細資料。

## 清理

當您不再需要專案時，可以從 Language Studio [專案]**** 頁面加以刪除。 您也可以在 [Azure 入口網站](https://portal.azure.com)中移除 Azure AI 語言服務和相關聯的儲存體帳戶。
