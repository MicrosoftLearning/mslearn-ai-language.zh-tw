---
lab:
  title: 翻譯語音
  module: Module 8 - Translate speech with Azure AI Speech
---

# 翻譯語音

Azure AI 語音包含語音翻譯 API，可讓您用來翻譯口語。 例如，假設您想要開發一個翻譯工具應用程式，人們可以在不會說當地語言的地方旅遊時使用。 他們能夠說出「車站在哪裡？」之類的片語？ 或以自己的語言說出「我需要找藥局」，然後將其翻譯成當地語言。

> **注意** 此練習會要求您使用具有喇叭/耳機的電腦。 為了獲得最佳體驗，也需要麥克風。 有些裝載的虛擬環境或許能夠從本機麥克風擷取音訊，但如果這不可行 (或您完全沒有麥克風)，也可使用所提供的音訊檔案進行語音輸入。 請仔細遵循指示，因為您必須根據您使用的是麥克風或音訊檔案來選擇不同的選項。

## 佈建 *Azure AI 語音*資源

如果您的訂用帳戶中還沒有 **Azure AI 語音**資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 在頂端的搜尋欄位中，搜尋「Azure AI 服務」****，接著按下 **Enter** 鍵，然後在結果中選取 [語音服務]**** 底下的 [建立]****。
1. 使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組*
    - **區域**：*選擇任何可用的區域*
    - **名稱**：輸入唯一名稱**
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
    - **負責任 AI 注意事項**：同意。
1. 選取 [檢閱 + 建立]****，然後選取 [建立]****，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 檢視 [金鑰和端點]**** 頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 準備在 Visual Studio Code 中開發應用程式

您將使用 Visual Studio Code 開發語音應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-ai-language** 存放庫，請在 Visual Studio 程式碼中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
1. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
1. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。

    > **注意**：如果 Visual Studio Code 顯示快顯訊息，提示您信任您所開啟的程式碼，請按一下快顯項目中的 [是，我信任作者]**** 選項。

1. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 設定您的應用程式

已提供 C# 和 Python 的應用程式。 這兩個應用程式都有相同的功能。 首先，您將完成應用程式的一些重要部分，使其能夠使用您的 Azure AI 語音資源。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **Labfiles/08-speech-translation**資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **translator** 資料夾。 每個資料夾都包含應用程式的特定語言程式碼檔案，您將在其中整合 Azure AI 語音功能。
1. 以滑鼠右鍵按一下包含程式碼檔案的 **translator** 資料夾，然後開啟整合式終端。 然後針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 語音 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. 在 [總管]**** 窗格中的 **translator** 資料夾中，開啟您慣用語言的組態檔

    - **C#**：appsettings.json
    - **Python**：.env

1. 更新組態值，以包含您所建立 Azure AI 語音資源的**區域**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)。

    > **注意**：請務必為您的資源新增*區域*，而<u>不是</u>端點！

1. 儲存組態檔。

## 新增程式碼以使用語音 SDK

1. 請注意，**translator** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：translator.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用 Azure AI 語音 SDK 所需的命名空間：

    **C#**：Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python**：translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入 Azure AI 語音服務金鑰和區域的程式碼。 您必須使用這些變數來為 Azure AI 語音資源建立 **SpeechTranslationConfig**，您會將其用於翻譯口語輸入。 在 **Configure translation** 註解之下新增下列程式碼：

    **C#**：Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python**：translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. 您將使用 **SpeechTranslationConfig** 將語音翻譯成文字，但您也會使用 **SpeechConfig** 將翻譯合成為語音。 在 **Configure speech** 註解之下新增下列程式碼：

    **C#**：Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python**：translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. 儲存變更並返回 **translator** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. 如果您使用 C#，則可在非同步方法中忽略任何有關使用 **await** 運算子的警告，我們稍後會修正該問題。 程式碼應該會顯示一則訊息，指出它已準備好從 en-US 進行翻譯，並提示您輸入目標語言。 按下 ENTER 可結束程式。

## 實作語音翻譯

現在，您已有適用於 Azure AI 語音服務的 **SpeechTranslationConfig**，您可以使用 Azure AI 語音翻譯 API 來辨識和翻譯語音。

> **重要**：本節包含兩個替代程序的指示。 如果您有運作中的麥克風，請遵循第一個程序。 如果您想要使用音訊檔案來模擬口語輸入，請遵循第二個程序。

### 如果您有運作中的麥克風

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **Translate** 函式來翻譯口語輸入。
1. 在 **Translate** 函式的 **Translate speech** 註解之下，新增下列程式碼來建立 **TranslationRecognizer** 用戶端，以便用於辨識和翻譯使用預設系統麥克風進行輸入的語音。

    **C#**：Program.cs

    ```csharp
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**：translator.py

    ```python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **注意** 應用程式中的程式碼會將輸入翻譯成單一呼叫中的所有三種語言。 只會顯示特定語言的翻譯，但您可以在結果的 **translations** 集合中指定目的語言程式碼，以擷取任何翻譯。

1. 現在跳到下面「執行程式」**** 一節。

---

### 或者，使用來自檔案的音訊輸入

1. 在終端機視窗中，輸入下列命令來安裝可用於播放音訊檔案的程式庫：

    **C#**：Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**：translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. 在程式的程式碼檔案中，於現有命名空間匯入之下，新增下列程式碼以匯入您剛才安裝的程式庫：

    **C#**：Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**：translator.py

    ```python
    from playsound import playsound
    ```

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **Translate** 函式來翻譯口語輸入。 然後在 **Translate** 函式的**翻譯語音**註解之下，新增下列程式碼來建立 **TranslationRecognizer** 用戶端，以便用於辨識和翻譯檔案中的語音。

    **C#**：Program.cs

    ```csharp
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**：translator.py

    ```python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

---

### 執行程式

1. 儲存變更並返回 **translator** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. 出現提示時，輸入有效的語言代碼 (*fr*、*es*或 *hi*)，然後若使用麥克風，請清楚說出「車站在哪裡？」 或您可能在出國旅遊時使用的一些其他片語。 程式應該轉譯您的口語輸入，並將其翻譯成您指定的語言 (法文、西班牙文或印度文)。 重複此程序，並嘗試應用程式支援的每種語言。 完成時，請按 ENTER 可結束程式。

    TranslationRecognizer 提供約 5 秒的說話時間。 如果其偵測不到口語輸入，則會產生「沒有相符項目」結果。 由於字元編碼問題，翻譯成印度文不一定會正確顯示在主控台視窗中。

> **注意**：應用程式中的程式碼會將輸入翻譯成單一呼叫中的所有三種語言。 只會顯示特定語言的翻譯，但您可以在結果的 **translations** 集合中指定目的語言程式碼，以擷取任何翻譯。

## 將翻譯合成為語音

到目前為止，您的應用程式會將口語輸入轉譯為文字；如果您需要在旅行時向某人尋求協助，可能就已足夠。 不過，最好以合適的語音朗讀翻譯。

1. 在 **Translate** 函式的**合成翻譯**註解之下，新增下列程式碼，以使用 **SpeechSynthesizer** 用戶端透過預設喇叭將翻譯合成為語音：

    **C#**：Program.cs

    ```csharp
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**：translator.py

    ```python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 儲存變更並返回 **translator** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. 出現提示時，輸入有效的語言代碼 (*fr*、*es* 或 *hi*)，然後清楚對麥克風說話，並說出您在國外旅遊時可能使用的片語。 程式應該轉譯您的口語輸入，並使用口語翻譯進行回應。 重複此程序，並嘗試應用程式支援的每種語言。 完成時，請按 **ENTER** 可結束程式。

> **注意**
> *在此範例中，您已使用 **SpeechTranslationConfig** 將語音翻譯成文字，然後使用 **SpeechConfig** 將翻譯合成為語音。事實上，您可使用 **SpeechTranslationConfig** 直接合成翻譯，但這只有在翻譯成單一語言時有作用，而結果是通常儲存為檔案而非直接傳送到喇叭的音訊串流。*

## 其他相關資訊

如需使用 Azure AI 語音翻譯 API 的詳細資訊，請參閱[語音翻譯文件](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation)。
