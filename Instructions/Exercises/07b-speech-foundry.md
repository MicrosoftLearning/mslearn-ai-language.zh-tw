---
lab:
  title: 識別和合成語音（Azure AI Foundry 版本）
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# 辨識及合成語音

**Azure AI 語音**是可提供語音相關功能的服務，包括：

- 語音轉換文字** API，可讓您實作語音辨識 (將聽得見的口語字詞轉換成文字)。
- 文字轉換語音** API，可讓您實作語音合成 (將文字轉換成聽得見的語音)。

在此練習中，您將使用這兩個 API 來實作語音時鐘應用程式。

> **注意** 此練習的設計目的是在 Azure Cloud Shell 中完成，不支援直接存取電腦的音效硬體。 因此，實驗室會針對語音輸入和輸出資料流使用音訊檔案。 提供使用麥克風和喇叭達到相同結果的代碼可供參考。

## 建立 Azure AI Foundry 專案

讓我們從建立 Azure AI Foundry 專案開始。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](./ai-foundry/media/ai-foundry-home.png)

1. 在首頁中，選取 **+ 建立專案**。
1. 請在 [建立專案精靈]**** 中，輸入專案有效名稱，如果建議使用現有的中樞，請選擇建立新中樞的選項。 然後審查 Azure 資源，將會自動建立，以便支援中樞和專案。
1. 選取**自訂**，然後為您的中樞指定下列設定：
    - **中樞名稱**：*請提供有效的中樞名稱*
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **位置**：選擇任何可用的區域
    - **連接 Azure AI 服務或 Azure OpenAI**：*[建立新的 AI 服務資源]*
    - **連線 Azure AI 搜尋服務**： *建立新的 Azure AI 搜尋服務資源，具有唯一名稱*

1. 選取**下一步**並檢閱您的設定。 然後選取**建立**並等待該流程完成。
1. 建立專案後，關閉顯示的所有提示並檢閱 Azure AI Foundry 入口網站中的專案頁面，該頁面應類似於下圖：

    ![Azure AI Foundry 入口網站中 Azure AI 專案詳細資料的螢幕螢幕擷取畫面。](./ai-foundry/media/ai-foundry-project.png)

## 準備及設定語音時鐘應用程式

1. 在 Azure AI Foundry 入口網站中，檢視專案的**概觀**頁面。
1. 在**專案詳細資料**區域中，記下專案的**專案連接字串**和**位置**。您將使用連接字串在用戶端應用程式中連接到您的專案，並且您需要該位置來連接到 Azure AI 服務語音端點。
1. 開啟一個新的瀏覽器索引標籤（保持 Azure AI Foundry 入口網站在現有索引標籤中開啟）。 然後在新索引標籤中，瀏覽到 `https://portal.azure.com` 的 [Azure 入口網站](https://portal.azure.com)；如果出現提示，請使用您的 Azure 認證登入。
1. 使用頁面頂部搜尋欄右側的 **[\>_]** 按鈕在 Azure 入口網站中建立一個新的 Cloud Shell，並選擇 ***PowerShell*** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***現在按照您選擇的程式設計語言的步驟進行操作。***

1. 複製存放庫之後，瀏覽至包含語音時鐘應用程式碼檔案的資料夾：  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中，將 **your_project_endpoint** 和 **your_location** 預留位置替換為專案的連接字串和位置（從 Azure AI Foundry 入口網站中的專案**概觀**頁面複製）。
1. 取代預留位置後，使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 新增程式碼以使用 Azure AI 語音 SDK

> **提示**：新增程式碼時，請確保保持正確的縮排。

1. 輸入以下命令，編輯已提供的\程式碼檔案：

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 在程式碼檔案的頂部，在現有的命名空間引用下，找到註解**匯入命名空間**。 然後，在此註解下，新增以下特定語言的程式碼，以匯入在 Azure Ai Foundry 專案中將 Azure AI Speech SDK 與 Azure AI 服務資源一起使用所需的命名空間：

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. 在 **main** 函式中，在註解**取得組態設定**下，請注意程式碼會載入您在設定檔中定義的專案連接字串和位置。

1. 在註解**從專案取得 AI 語音端點和金鑰**下新增以下程式碼：

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    此程式碼連接到您的 Azure AI Foundry 專案，獲取其預設的 AI 服務連接資源，並擷取使用它所需的驗證金鑰。

1. 在註解**設定語音服務**下，新增以下程式碼以使用 AI 服務金鑰和專案的區域來設定與 Azure AI 服務語音端點的連接

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. 儲存您的變更 （*CTRL+S*），但讓程式碼編輯器保持開啟狀態。

## 執行應用程式

到目前為止，該應用程式除了連接到您的 Azure AI Foundry 專案以擷取使用語音服務所需的詳細資料之外不執行任何其他操作，但在新增語音功能之前執行它並檢查它是否正常工作很有用。

1. 在程式碼編輯器下方的命令列中，輸入以下 Azure CLI 命令來確定為工作階段登入的 Azure 帳戶：

    ```
   az account show
    ```

    產生的 JSON 輸出應包含您的 Azure 帳戶和您正在處理的訂閱的詳細資料（該訂閱應與您建立 Azure AI Foundry 專案的訂閱相同）。

    您的應用程式使用其執行上下文中的 Azure 認證來驗證與您的專案的連接。 在實際執行環境中，應用程式可能會設定為使用受控識別來執行。 在此開發環境中，它會使用已驗證的 Cloud Shell 工作階段認證。

    > **注意**：您可以使用 Azure CLI 命令，在開發環境中登入`az login` Azure。 在這種情況下，Cloud Shell 已經使用您登入入口網站時使用的 Azure 認證登入；因此無需明確登入。 要瞭解有關使用 Azure CLI 向 Azure 進行驗證的更多資訊，請參閱[使用 Azure CLI 向 Azure 進行驗證](https://learn.microsoft.com/cli/azure/authenticate-azure-cli)。

1. 在命令列中，輸入以下特定語言的命令來執行語音時鐘應用程式：

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 如果您使用 C#，則可在非同步方法中忽略任何有關使用 **await** 運算子的警告，我們稍後會修正該問題。 程式碼應該顯示應用程式將使用的語音服務資源區域。 成功執行表示應用程式已連接到您的 Azure AI Foundry 專案並擷取到使用Azure AI 語音服務所需的金鑰。

## 新增程式碼以辨識語音

現在，您已在專案的 Azure AI 服務資源中為語音服務準備好了 **SpeechConfig**，接下來可以使用 **Speech-to-text** API 來識別語音並將其轉錄為文字。

在此程式中，語音輸入是從音訊檔案擷取，您可以在這裡播放：

<video controls src="ai-foundry/media/Time.mp4" title="What time is it? (現在幾點？)" width="150"></video>

1. 在 **Main** 函式中，請注意程式碼會使用 **TranscribeCommand** 函式來接受口語輸入。 然後在 **TranscribeCommand** 函式的**設定語音辨識**註解之下，新增適當的程式碼來建立 **SpeechRecognizer** 用戶端，以便用於辨識音訊檔案中的語音並加以轉譯：

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. 在 **TranscribeCommand** 函式的**處理語音輸入**註解之下，新增下列程式碼以接聽口語輸入，小心不要取代可傳回命令之函式尾端的程式碼：

    **Python**

    ```python
   # Process speech input
   print("Listening...")
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

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
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

1. 儲存您的變更（*CTRL+S*），然後在程式碼編輯器下方的命令列中輸入以下命令來執行該程式：

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 檢閱應用程式的輸出，它應該成功「聽到」音訊檔案中的語音並傳回適當的回應（請注意，您的 Azure Cloud Shell 可能在與您時區不同的伺服器上執行！）

    > **提示**：如果 SpeechRecognizer 發生錯誤，則會產生「已取消」的結果。 然後，應用程式中的程式碼會顯示錯誤訊息。 最可能的原因是設定檔中的區域值不正確。

## 合成語音

您的語音時鐘應用程式可接受口語輸入，但實際上不會說話！ 讓我們新增程式碼來合成語音，藉此修正該問題。

再次，由於 Cloud Shell 的硬體限制，我們將合成的語音輸出到檔案。

1. 在程式的 **Main** 函式中，請注意程式碼會使用 **TellTime** 函式來告知使用者目前的時間。
1. 在 **TellTime** 函式的**設定語音合成**註解之下，新增下列程式碼來建立 **SpeechSynthesizer** 用戶端，以便用於產生語音輸出：

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. 在 **TellTime** 函式的**合成口語輸出**註解之下，新增下列程式碼以產生口語輸出，小心不要取代可列印回應之函式尾端的程式碼：

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. 儲存您的變更（*CTRL+S*），然後在程式碼編輯器下方的命令列中輸入以下命令來執行該程式：

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 檢閱應用程式的輸出，它應該表明口頭輸出已儲存在檔案中。
1. 如果您有一個能夠播放 .wav 音訊檔案的媒體播放器，請在 Cloud Shell 窗格的工具列中，使用**上傳/下載檔案**按鈕從您的應用程式資料夾下載音訊檔案，然後播放它：

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    該檔案聽起來應該類似於以下內容：

    <video controls src="./ai-foundry/media/Output.mp4" title="時間是 2:15" width="150"></video>

## 使用語音合成標記語言

語音合成標記語言 (SSML) 可讓您自訂使用 XML 格式合成語音的方式。

1. 在 **TellTime** 函式中，以下列程式碼取代 **Synthesize spoken output** 註解之下的所有目前程式碼 (保留 **Print the response** 註解之下的程式碼)：

    **Python**

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
   else:
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

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
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. 儲存變更並返回 **speaking-clock** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 檢閱應用程式的輸出，它應該表明口頭輸出已儲存在檔案中。
1. 再次，如果您有一個能夠播放 .wav 音訊檔案的媒體播放器，請在 Cloud Shell 窗格的工具列中，使用**上傳/下載檔案**按鈕從您的應用程式資料夾下載音訊檔案，然後播放它：

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    該檔案聽起來應該類似於以下內容：
    
    <video controls src="./ai-foundry/media/Output2.mp4" title="時間是 5:30。 是時候結束這個實驗了。" width="150"></video>

## 清理

如果您已完成 Azure AI 語音探索，您應該刪除在本練習中建立的資源，以避免產生不必要的 Azure 成本。

1. 返回包含 Azure 入口網站的瀏覽器索引標籤 (或在新的瀏覽器索引標籤中重新開啟位於`https://portal.azure.com`的 [Azure 入口網站](https://portal.azure.com))，並檢視您在其中部署本練習所用資源的資源群組內容。
1. 在工具列上，選取 [刪除資源群組]****。
1. 輸入資源群組名稱並確認您想要將其刪除。

## 如果您有麥克風和喇叭，該怎麼辦？

在本練習中，您使用音訊檔案進行語音輸入和輸出。 讓我們看看如何修改程式碼以使用音訊硬體。

### 使用麥克風進行語音辨識

如果您有麥克風，您可以使用以下程式碼擷取語音輸入以進行語音辨識：

**Python**

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

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

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

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

> **注意**：系統預設麥克風是預設音訊輸入，因此您也可以完全省略 AudioConfig！

### 使用喇叭進行語音合成

如果您有喇叭，您可以使用以下程式碼來合成語音。

**Python**

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **注意**：系統預設喇叭是預設音訊輸出，因此您也可以完全省略 AudioConfig！

## 其他相關資訊

如需使用**語音轉換文字**和**文字轉換語音** API 的詳細資訊，請參閱[語音轉換文字文件](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text)和[文字轉換語音文件](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)。
