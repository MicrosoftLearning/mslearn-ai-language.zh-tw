---
lab:
  title: 翻譯文字
  description: 使用 Azure AI 翻譯工具將提供的文字翻譯成任何支援的語言。
---

# 翻譯文字

**Azure AI 翻譯工具**是一項可讓您能夠在不同語言之間翻譯文字的服務。 在本練習中，您將使用它來建立一個簡單的應用程式，將任何支援的語言的輸入翻譯為您選擇的目標語言。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發文字翻譯應用程式，包括：

- [適用於 Python 的 Azure AI 翻譯用戶端程式庫](https://pypi.org/project/azure-ai-translation-text/)
- [適用於 .NET 的 Azure AI 翻譯用戶端程式庫](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [適用於 JavaScript 的 Azure AI 翻譯用戶端程式庫](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

本練習大約需要 **30** 分鐘的時間。

## 佈建 *Azure AI 翻譯工具*資源

如果您的訂用帳戶中還沒有該資源，則需要佈建 **Azure AI 翻譯工具**資源。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 在頂端的搜尋欄位中，搜尋 [翻譯工具]****，然後選取結果中的 [翻譯工具]****。
1. 使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組*
    - **區域**：*選擇任何可用的區域*
    - **名稱**：輸入唯一名稱**
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 準備在 Cloud Shell 中開發應用程式

若要測試 Azure AI 翻譯工具的文字翻譯功能，您將在 Azure Cloud Shell 中開發簡單的主控台應用程式。

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
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## 設定您的應用程式

1. 在命令列窗格中，執行下列命令以檢視 **translate-text** 資料夾中的程式碼檔案：

    ```
   ls -a -l
    ```

    這些檔案包括設定檔 (**.env**) 和程式碼檔案 (**translate.py**)。

1. 執行下列命令來建立 Python 虛擬環境，並安裝 Azure AI 翻譯 SDK 套件和其他必要套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. 輸入下列命令以編輯應用程式組態檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 更新組態值，以包含您所建立 Azure AI 翻譯工具資源的**區域**和**金鑰** (可在 Azure 入口網站中 Azure AI 翻譯工具資源的 [金鑰和端點]**** 頁面上取得)。

    > **備註**：請務必為您的資源新增*區域*，而<u>不是</u>端點！

1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 新增程式碼以翻譯文字

1. 輸入下列命令以編輯應用程式程式碼檔案：

    ```
   code translate.py
    ```

1. 檢閱現有程式碼。 您將新增程式碼以使用 Azure AI 翻譯 SDK。

    > **秘訣**：當您將程式碼新增至程式碼檔案時，請確保縮排維持正確。

1. 在程式碼檔案頂端的現有命名空間參考下方，尋找 **Import namespaces** 註解，並新增下列程式碼以匯入您需要使用翻譯 SDK 的命名空間：

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. 在 **main** 函式中，您會發現現有程式碼會讀取組態設定。
1. 尋找**使用端點和金鑰建立用戶端**註解，並新增下列程式碼：

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. 尋找 **Choose target language** 註解並新增下列程式碼，該程式碼會使用文字翻譯工具服務來傳回支援的語言清單以進行翻譯，並提示使用者選取目標語言的語言代碼：

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. 尋找 **Translate text** 註解並新增下列程式碼，該程式碼會反覆提示使用者提供要翻譯的文字、使用 Azure AI 翻譯工具服務來將其翻譯為目標語言 (自動偵測來源語言)，並顯示結果，直到使用者輸入 *quit* 為止：

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. 儲存變更 (CTRL+S)，然後輸入下列命令以執行程式 (將 Cloud Shell 窗格最大化，並調整面板大小，以查看命令列窗格中的更多文字)：

    ```
   python translate.py
    ```

1. 出現提示時，從顯示的清單中輸入有效的目標語言。
1. 輸入要翻譯的片語 (例如 `This is a test` 或 `C'est un test`) 並檢視結果，這應該會偵測來源語言並將文字翻譯成目標語言。
1. 當您完成時，請輸入 `quit`。 您可以再次執行該應用程式，並選擇不同的目標語言。

## 清除資源

若您已完成探索 Azure AI 翻譯工具服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 關閉 [Azure Cloud Shell] 窗格
1. 在 Azure 入口網站中，瀏覽至您在此實驗室中建立的 Azure AI 翻譯工具資源。
1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

如需使用 **Azure AI 翻譯工具**的詳細資訊，請參閱 [Azure AI 翻譯工具文件](https://learn.microsoft.com/azure/ai-services/translator/)。
