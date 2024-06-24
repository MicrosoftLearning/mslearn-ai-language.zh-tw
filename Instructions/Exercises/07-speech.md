---
lab:
  title: 辨識及合成語音
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# 辨識及合成語音

**Azure AI 語音**是可提供語音相關功能的服務，包括：

- 語音轉換文字** API，可讓您實作語音辨識 (將聽得見的口語字詞轉換成文字)。
- 文字轉換語音** API，可讓您實作語音合成 (將文字轉換成聽得見的語音)。

在此練習中，您將使用這兩個 API 來實作語音時鐘應用程式。

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
1. 選取 **[檢閱 + 建立]，** 然後選取 **[建立]**，以佈建資源。
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

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **Labfiles/07-speech** 資料夾，然後根據您的語言喜好設定展開 **CSharp** 或 **Python** 資料夾，以及其所包含的 **speaking-clock** 資料夾。 每個資料夾都包含應用程式的特定語言程式碼檔案，您將在其中整合 Azure AI 語音功能。
1. 以滑鼠右鍵按一下包含程式碼檔案的 **speaking-clock** 資料夾，然後開啟整合式終端。 然後針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 語音 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. 在 [總管]**** 窗格中的 **speaking-clock** 資料夾中，開啟您慣用語言的組態檔

    - **C#**：appsettings.json
    - **Python**：.env

1. 更新組態值，以包含您所建立 Azure AI 語音資源的**區域**和**金鑰** (可在 Azure 入口網站中 Azure AI 語音資源的 [金鑰和端點]**** 頁面上取得)。

    > **注意**：請務必為您的資源新增*區域*，而<u>不是</u>端點！

1. 儲存組態檔。

## 新增程式碼以使用 Azure AI 語音 SDK

1. 請注意，**speaking-clock** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：speaking-clock.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用 Azure AI 語音 SDK 所需的命名空間：

    **C#**：Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**：speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. 在 **Main** 函式中，請注意已經提供可從組態檔載入服務金鑰和區域的程式碼。 您必須使用這些變數來為您的 Azure AI 語音資源建立 **SpeechConfig**。 在 **Configure speech service** 註解之下新增下列程式碼：

    **C#**：Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 如果您使用 C#，則可在非同步方法中忽略任何有關使用 **await** 運算子的警告，我們稍後會修正該問題。 程式碼應該顯示應用程式將使用的語音服務資源區域。

## 新增程式碼以辨識語音

既然您在 Azure AI 語音資源中有語音服務的 **SpeechConfig**，即可使用**語音轉換文字** API 來辨識語音並將其轉譯成文字。

> **重要**：本節包含兩個替代程序的指示。 如果您有運作中的麥克風，請遵循第一個程序。 如果您想要使用音訊檔案來模擬口語輸入，請遵循第二個程序。

### 如果您有運作中的麥克風

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **TranscribeCommand** 函式來接受口語輸入。
1. 在 **TranscribeCommand** 函式的**設定語音辨識**註解之下，新增適當的程式碼來建立 **SpeechRecognizer** 用戶端，以便使用預設系統麥克風來辨識和轉譯語音：

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. 現在跳到下面**新增程式碼以處理轉譯的命令**一節。

---

### 或者，使用來自檔案的音訊輸入

1. 在終端機視窗中，輸入下列命令來安裝可用於播放音訊檔案的程式庫：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. 在程式的程式碼檔案中，於現有命名空間匯入之下，新增下列程式碼以匯入您剛才安裝的程式庫：

    **C#**：Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**：speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. 在 **Main** 函式中，請注意程式碼會使用 **TranscribeCommand** 函式來接受口語輸入。 然後在 **TranscribeCommand** 函式的**設定語音辨識**註解之下，新增適當的程式碼來建立 **SpeechRecognizer** 用戶端，以便用於辨識音訊檔案中的語音並加以轉譯：

    **C#**：Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### 新增程式碼以處理轉譯的命令

1. 在 **TranscribeCommand** 函式的**處理語音輸入**註解之下，新增下列程式碼以接聽口語輸入，小心不要取代可傳回命令之函式尾端的程式碼：

    **C#**：Program.cs

    ```csharp
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**：speaking-clock.py

    ```python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

1. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 如果使用麥克風，請清楚說話並說出「現在幾點？」。 程式應該轉譯口語輸入，並根據程式碼執行所在電腦的當地時間顯示時間 (這可能不是您所在地點的正確時間)。

    SpeechRecognizer 提供約 5 秒的說話時間。 如果其偵測不到口語輸入，則會產生「沒有相符項目」結果。

    如果 SpeechRecognizer 發生錯誤，則會產生「已取消」的結果。 然後，應用程式中的程式碼會顯示錯誤訊息。 最可能的原因是組態檔中的金鑰或區域不正確。

## 合成語音

您的語音時鐘應用程式可接受口語輸入，但實際上不會說話！ 讓我們新增程式碼來合成語音，藉此修正該問題。

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **TellTime** 函式來告知使用者目前的時間。
1. 在 **TellTime** 函式的**設定語音合成**註解之下，新增下列程式碼來建立 **SpeechSynthesizer** 用戶端，以便用於產生語音輸出：

    **C#**：Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **注意** 預設音訊設定會使用預設系統音訊裝置進行輸出，您就不需要明確提供 **AudioConfig**。 如果您需要將音訊輸出重新導向至檔案，您可使用 **AudioConfig** 搭配檔案路徑來這麼做。

1. 在 **TellTime** 函式的**合成口語輸出**註解之下，新增下列程式碼以產生口語輸出，小心不要取代可列印回應之函式尾端的程式碼：

    **C#**：Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**：speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 出現提示時，清楚對麥克風說話並說出「現在幾點？」。 程式應該說話，告知您時間。

## 使用不同的語音

您的語音時鐘應用程式會使用預設語音，您可加以變更。 語音服務支援各種*標準*語音，以及更像人類的*神經*語音。 您也可以建立*自訂*語音。

> **注意**：如需類神經和標準語音的清單，請參閱 Speech Studio 中的[語音庫](https://speech.microsoft.com/portal/voicegallery)。

1. 在 **TellTime** 函式的**設定語音合成**註解之下，如下所示修改程式碼來指定替代語音，再建立 **SpeechSynthesizer** 用戶端：

   **C#**：Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 出現提示時，清楚對麥克風說話並說出「現在幾點？」。 程式應該以指定的語音說話，告知您時間。

## 使用語音合成標記語言

語音合成標記語言 (SSML) 可讓您自訂使用 XML 格式合成語音的方式。

1. 在 **TellTime** 函式中，以下列程式碼取代 **Synthesize spoken output** 註解之下的所有目前程式碼 (保留 **Print the response** 註解之下的程式碼)：

   **C#**：Program.cs

    ```csharp
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**：speaking-clock.py

    ```python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 出現提示時，清楚對麥克風說話並說出「現在幾點？」。 程式應該以 SSML 指定的語音說話 (覆寫 SpeechConfig 中指定的語音)，告知您時間，然後在暫停之後告知您是結束此實驗室的時候了！

## 其他相關資訊

如需使用**語音轉換文字**和**文字轉換語音** API 的詳細資訊，請參閱[語音轉換文字文件](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text)和[文字轉換語音文件](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)。
