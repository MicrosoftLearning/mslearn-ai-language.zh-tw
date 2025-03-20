---
lab:
  title: 翻譯文字
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# 翻譯文字

**Azure AI 翻譯工具**是一項可讓您能夠在不同語言之間翻譯文字的服務。 在本練習中，您將使用它來建立一個簡單的應用程式，將任何支援的語言的輸入翻譯為您選擇的目標語言。

## 佈建 *Azure AI 翻譯工具*資源

如果您的訂用帳戶中還沒有該資源，則需要佈建 **Azure AI 翻譯工具**資源。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 在頂端的搜尋欄位中，搜尋 **Azure AI 服務**並按 **Enter**，然後在結果中的[翻譯工具]**** 下選取 [建立]****。
1. 使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組*
    - **區域**：*選擇任何可用的區域*
    - **名稱**：輸入唯一名稱**
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **負責任 AI 注意事項**：同意。
1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 準備在 Visual Studio Code 中開發應用程式

您將使用 Visual Studio Code 來開發文字翻譯應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 設定您的應用程式

已提供 C# 和 Python 的應用程式。 這兩個應用程式都有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 翻譯工具資源。

1. 在 Visual Studio Code 的 [檔案總管]**** 窗格中，瀏覽至 **Labfiles/06b-translator-sdk** 資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **translate-text** 資料夾。 每個資料夾都包含應用程式的特定語言程式碼檔案，您將在其中整合 Azure AI 翻譯工具功能。
2. 以滑鼠右鍵按一下包含程式碼檔案的 **translate-text** 資料夾，然後開啟整合式終端。 然後針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 翻譯工具 SDK 套件：

    **C#：**

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**：

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. 在 [檔案總管]**** 窗格中的 **translate-text** 資料夾中，開啟您慣用語言的組態檔

    - **C#**：appsettings.json
    - **Python**：.env
    
4. 更新組態值，以包含您所建立 Azure AI 翻譯工具資源的**區域**和**金鑰** (可在 Azure 入口網站中 Azure AI 翻譯工具資源的 [金鑰和端點]**** 頁面上取得)。

    > **備註**：請務必為您的資源新增*區域*，而<u>不是</u>端點！

5. 儲存組態檔。

## 新增程式碼以翻譯文字

現在您已準備好使用 Azure AI 翻譯工具來翻譯文字。

1. 請注意，**translate-text** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：translate.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用文字分析 SDK 所需的命名空間：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**：translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. 在 **Main** 函式中，請注意現有的程式碼會讀取組態設定。
1. 尋找**使用端點和金鑰建立用戶端**註解，並新增下列程式碼：

    **C#**：Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**：translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. 尋找**選擇目標語言**註解並新增下列程式碼，該程式碼會使用文字翻譯工具服務來傳回支援的語言清單以進行翻譯，並提示使用者選取目標語言的語言代碼。

    **C#**：Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**：translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
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

1. 尋找**翻譯文字**註解並新增下列程式碼，該程式碼會反覆提示使用者提供要翻譯的文字、使用 Azure AI 翻譯工具服務來將其翻譯為目標語言 (自動偵測來源語言)，並顯示結果，直到使用者輸入 *quit* 為止。

    **C#**：Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**：translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. 將變更儲存至您的程式碼檔案。

## 測試您的應用程式

現在您的應用程式已準備好進行測試。

1. 在 **Translate text** 資料夾的整合式終端中，然後輸入下列命令來執行程式：

    - **C#**：`dotnet run`
    - **Python**：`python translate.py`

    > **秘訣**：您可以使用終端工具列中的**最大化面板大小** (**^**) 圖示來查看更多的主控台文字。

1. 出現提示時，從顯示的清單中輸入有效的目標語言。
1. 輸入要翻譯的片語 (例如 `This is a test` 或 `C'est un test`) 並檢視結果，這應該會偵測來源語言並將文字翻譯成目標語言。
1. 當您完成時，請輸入 `quit`。 您可以再次執行該應用程式，並選擇不同的目標語言。

## 清理

當您不再需要專案時，可以在 [Azure 入口網站](https://portal.azure.com)中刪除 Azure AI 翻譯工具資源。
