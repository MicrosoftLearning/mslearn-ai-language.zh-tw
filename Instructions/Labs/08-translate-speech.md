---
lab:
  title: 翻譯語音
  description: 將語言語音翻譯成語音，並在您自己的應用程式中實作。
---

# 翻譯語音

Azure AI 語音包含語音翻譯 API，可讓您用來翻譯口語。 例如，假設您想要開發一個翻譯工具應用程式，人們可以在不會說當地語言的地方旅遊時使用。 他們能夠說出「車站在哪裡？」之類的片語？ 或以自己的語言說出「我需要找藥局」，然後將其翻譯成當地語言。 在本練習中，您將使用適用於 Python 的 Azure AI 語音 SDK，根據此範例建立簡單的應用程式。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發語音翻譯的應用程式，包括：

- [適用於 Python 的 Azure AI 語音 SDK](https://pypi.org/project/azure-cognitiveservices-speech/)
- [適用於 .NET 的 Azure AI 語音 SDK](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [適用於 JavaScript 的 Azure AI 語音 SDK](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

本練習大約需要 **30** 分鐘的時間。

> **注意** 此練習的設計目的是在 Azure Cloud Shell 中完成，不支援直接存取電腦的音效硬體。 因此，實驗室會針對語音輸入和輸出資料流使用音訊檔案。 提供使用麥克風和喇叭達到相同結果的代碼可供參考。

## 建立 Azure AI 語音資源

讓我們從建立 Azure AI 語音資源開始。

1. 開啟 [Azure 入口網站][](https://portal.azure.com) (位於 `https://portal.azure.com`)，使用與您的 Azure 訂閱相關聯的 Microsoft 帳戶進行登入。
1. 在頂端搜尋欄位中，搜尋 [語音服務]****。 從清單中選取後，選取 [建立]****。
1. 使用下列設定佈建資源：
    - **訂用帳戶**：*您的 Azure 訂用帳戶*。
    - **資源群組**：*選擇或建立資源群組*。
    - **區域**：*選擇任何可用的區域*
    - **名稱**：*輸入唯一名稱*。
    - **定價層**：如果 F 無法使用，請選取 **F0** (*免費*)，或 **S** (*標準*)。
1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
1. 等候部署完成，然後移至資源。
1. 在**資源管理**區段中檢視**金鑰和端點**頁面。 稍後在本練習中，您將需要此頁面的資訊。

## 準備在 Cloud Shell 中開發應用程式

1. 維持開啟 [金鑰與端點]**** 頁面，使用頁面上方搜尋欄右側的 **[\>_]** 按鈕，即可在 Azure 入口網站中建立新的 Cloud Shell，並選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **秘訣**：當您將命令輸入到 Cloud Shell 時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含程式碼檔案的資料夾：

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. 在命令列窗格中，執行下列命令以檢視**翻譯工具**資料夾中的程式碼檔案：

    ```
   ls -a -l
    ```

    這些檔案包括設定檔 (**.env**) 和程式碼檔案 (**translator.py**)。

1. 建立 Python 虛擬環境，並執行下列命令來安裝 Azure AI 語音 SDK 套件和其他必要套件：

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 更新設定值，以包含您所建立 Azure AI 語音資源的**區域**和**金鑰** (可在 Azure 入口網站中 Azure AI 翻譯工具資源的 [金鑰和端點]**** 頁面上取得)。
1. 取代預留位置後，使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 新增程式碼以使用 Azure AI 語音 SDK

> **提示**：新增程式碼時，請確保保持正確的縮排。

1. 輸入以下命令，編輯已提供的\程式碼檔案：

    ```
   code translator.py
    ```

1. 在程式碼檔案的頂部，在現有的命名空間引用下，找到註解**匯入命名空間**。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用 Azure AI 語音 SDK 所需的命名空間：

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. 在 **main** 函式中，在 **Get config settings** 註解下方，您會發現程式碼會載入您在設定檔中定義的金鑰和區域。

1. 在 **Configure translation** 註解下尋找下列程式碼，然後新增下列程式碼來設定您與 Azure AI 服務語音端點的連線：

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. 您將使用 **SpeechTranslationConfig** 將語音翻譯成文字，但您也會使用 **SpeechConfig** 將翻譯合成為語音。 在 **Configure speech** 註解之下新增下列程式碼：

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. 儲存您的變更 （*CTRL+S*），但讓程式碼編輯器保持開啟狀態。

## 執行應用程式

到目前為止，應用程式不會執行連線至 Azure AI 語音資源以外的任何動作，但在新增語音功能之前，先執行應用程式並檢查其是否可以正常運作將會很有幫助。

1. 在命令列中，輸入下列命令來執行翻譯工具應用程式：

    ```
   python translator.py
    ```

    程式碼應該會顯示應用程式將使用的語音服務資源區域、一則已準備好從 en-US 翻譯的訊息，並提示您輸入目標語言。 成功執行就代表應用程式已連線到您的 Azure AI 語音服務。 按下 ENTER 可結束程式。

## 實作語音翻譯

現在，您已有適用於 Azure AI 語音服務的 **SpeechTranslationConfig**，您可以使用 Azure AI 語音翻譯 API 來辨識和翻譯語音。

1. 在程式碼檔案中，您會發現程式碼會使用 **Translate** 函式來翻譯口語輸入。 然後在 **Translate** 函式的**翻譯語音**註解之下，新增下列程式碼來建立 **TranslationRecognizer** 用戶端，以便用於辨識和翻譯檔案中的語音。

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. 儲存您的變更 (*CTRL+S*)，然後重新執行程式：

    ```
   python translator.py
    ```

1. 出現提示時，請輸入有效的語言代碼 (*fr*、*es* 或 *hi*)。 程式應該轉譯您的輸入檔案，並將其翻譯成您指定的語言 (法文、西班牙文或印度文)。 重複此程序，並嘗試應用程式支援的每種語言。

    > **注意**：由於字元編碼問題，翻譯成印度文不一定會正確顯示在主控台視窗中。

1. 完成時，請按 ENTER 可結束程式。

> **注意**：應用程式中的程式碼會將輸入翻譯成單一呼叫中的所有三種語言。 只會顯示特定語言的翻譯，但您可以在結果的 **translations** 集合中指定目的語言程式碼，以擷取任何翻譯。

## 將翻譯合成為語音

到目前為止，您的應用程式會將口語輸入轉譯為文字；如果您需要在旅行時向某人尋求協助，可能就已足夠。 不過，最好以合適的語音朗讀翻譯。

> **注意**：由於 Cloud Shell 的硬體限制，我們將合成的語音輸出到檔案。

1. 在 **Translate** 函式中，找到 **Synthesize translation** 註解，然後新增下列程式碼，以使用 **SpeechSynthesizer** 用戶端將語音合成為翻譯，然後儲存為 .wav 檔案：

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. 儲存您的變更 (*CTRL+S*)，然後重新執行程式：

    ```
   python translator.py
    ```

1. 檢閱應用程式的輸出，程式應會顯示口頭輸出已儲存在檔案中。 完成時，請按 **ENTER** 可結束程式。
1. 若您有能夠播放 .wav 音訊檔案的媒體播放器，請輸入下列命令來下載所產生的檔案：

    ```
   download ./output.wav
    ```

    下載命令就會在瀏覽器右下方建立快顯視窗連結，您可以選取連結，即可下載並開啟檔案。

> **注意**
> *在此範例中，您已使用 **SpeechTranslationConfig** 將語音翻譯成文字，然後使用 **SpeechConfig** 將翻譯合成為語音。事實上，您可使用 **SpeechTranslationConfig** 直接合成翻譯，但這只有在翻譯成單一語言時有作用，而結果是通常儲存為檔案的音訊串流。*

## 清除資源

若您已完成探索 Azure AI 語音服務，您可以刪除在此練習中所建立的資源。 方法如下：

1. 關閉 [Azure Cloud Shell] 窗格
1. 在 Azure 入口網站中，瀏覽至您在此實驗室中建立的 Azure AI 語音資源。
1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 如果您有麥克風和喇叭，該怎麼辦？

在本練習中，您使用音訊檔案進行語音輸入和輸出。 讓我們看看如何修改程式碼以使用音訊硬體。

### 使用麥克風進行語音翻譯

1. 若您有麥克風，您可以使用以下程式碼擷取語音輸入以進行語音翻譯：

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **注意**：系統預設麥克風是預設音訊輸入，因此您也可以完全省略 AudioConfig！

### 使用喇叭進行語音合成

1. 如果您有喇叭，您可以使用以下程式碼來合成語音。
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **注意**：系統預設喇叭是預設音訊輸出，因此您也可以完全省略 AudioConfig！

## 其他相關資訊

如需使用 Azure AI 語音翻譯 API 的詳細資訊，請參閱[語音翻譯文件](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation)。
