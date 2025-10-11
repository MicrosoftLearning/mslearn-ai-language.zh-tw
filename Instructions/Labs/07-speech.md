---
lab:
  title: 辨識及合成語音
  description: 實作將語音轉換成文字以及將文字轉換成語音的語音時鐘。
---

# 辨識及合成語音

**Azure AI 語音**是可提供語音相關功能的服務，包括：

- 語音轉換文字** API，可讓您實作語音辨識 (將聽得見的口語字詞轉換成文字)。
- 文字轉換語音** API，可讓您實作語音合成 (將文字轉換成聽得見的語音)。

在此練習中，您將使用這兩個 API 來實作語音時鐘應用程式。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發語音應用程式，包括：

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

## 準備及設定語音時鐘應用程式

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

1. 複製存放庫之後，瀏覽至包含語音時鐘應用程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. 在命令列窗格中，執行下列命令以檢視 **speaking-clock** 資料夾中的程式碼檔案：

    ```
   ls -a -l
    ```

    這些檔案包括設定檔 (**.env**) 和程式碼檔案 (**speaking-clock.py**)。 您的應用程式將使用的音訊檔案位於**音訊**子資料夾中。

1. 建立 Python 虛擬環境，並執行下列命令來安裝 Azure AI 語音 SDK 套件和其他必要套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. 輸入下列命令，編輯設定檔：

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
   code speaking-clock.py
    ```

1. 在程式碼檔案的頂部，在現有的命名空間引用下，找到註解**匯入命名空間**。 然後，在此註解之下，新增下列語言特有程式碼，以匯入您使用 Azure AI 語音 SDK 所需的命名空間：

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. 在 **main** 函式中，在 **Get config settings** 註解下方，您會發現程式碼會載入您在設定檔中定義的金鑰和區域。

1. 在 **Configure speech service** 註解下，新增下列程式碼以使用 AI 服務金鑰和您的區域來設定與 Azure AI 服務語音端點的連線：

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. 儲存您的變更 （*CTRL+S*），但讓程式碼編輯器保持開啟狀態。

## 執行應用程式

到目前為止，應用程式不會執行連線至 Azure AI 語音服務以外的任何動作，但在新增語音功能之前，先執行應用程式並檢查其是否可以正常運作將會很有幫助。

1. 在命令列中，輸入以下命令來執行語音時鐘應用程式：

    ```
   python speaking-clock.py
    ```

    程式碼應該顯示應用程式將使用的語音服務資源區域。 成功執行就代表應用程式已連線到您的 Azure AI 語音資源。

## 新增程式碼以辨識語音

現在，您已在專案的 Azure AI 服務資源中為語音服務準備好了 **SpeechConfig**，接下來可以使用 **Speech-to-text** API 來識別語音並將其轉錄為文字。

在此程式中，語音輸入是從音訊檔案擷取，您可以在這裡播放：

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="What time is it? (現在幾點？)" width="150"></video>

1. 在程式碼檔案中，您會發現程式碼會使用 **TranscribeCommand** 函式來接受口語輸入。 然後在 **TranscribeCommand** 函式中，找到 **Configure speech recognition** 註解並新增適當的程式碼來建立 **SpeechRecognizer** 用戶端，以便用於辨識音訊檔案中的語音並加以轉譯：

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. 在 **TranscribeCommand** 函式的**處理語音輸入**註解之下，新增下列程式碼以接聽口語輸入，小心不要取代可傳回命令之函式尾端的程式碼：

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

1. 儲存您的變更 (*CTRL+S*)，然後在程式碼編輯器下方的命令列中重新執行程式：
1. 檢閱輸出，其應該成功「聽到」音訊檔案中的語音並傳回適當回應 (請注意，您的 Azure Cloud Shell 可能在與您時區不同的伺服器上執行！)

    > **提示**：如果 SpeechRecognizer 發生錯誤，則會產生「已取消」的結果。 然後，應用程式中的程式碼會顯示錯誤訊息。 最可能的原因是設定檔中的區域值不正確。

## 合成語音

您的語音時鐘應用程式可接受口語輸入，但實際上不會說話！ 讓我們新增程式碼來合成語音，藉此修正該問題。

再次，由於 Cloud Shell 的硬體限制，我們將合成的語音輸出到檔案。

1. 在程式碼檔案中，您會發現程式碼會使用 **TellTime** 函式告訴使用者目前的時間。
1. 在 **TellTime** 函式的**設定語音合成**註解之下，新增下列程式碼來建立 **SpeechSynthesizer** 用戶端，以便用於產生語音輸出：

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. 在 **TellTime** 函式的**合成口語輸出**註解之下，新增下列程式碼以產生口語輸出，小心不要取代可列印回應之函式尾端的程式碼：

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. 儲存您的變更 (*CTRL+S*) 並重新執行程式，程式應會顯示口語輸出已儲存在檔案中。

1. 若您有能夠播放 .wav 音訊檔案的媒體播放器，請輸入下列命令來下載所產生的檔案：

    ```
   download ./output.wav
    ```

    下載命令就會在瀏覽器右下方建立快顯視窗連結，您可以選取連結，即可下載並開啟檔案。

    該檔案聽起來應該類似於以下內容：

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="時間是 2:15" width="150"></video>

## 使用語音合成標記語言

語音合成標記語言 (SSML) 可讓您自訂使用 XML 格式合成語音的方式。

1. 在 **TellTime** 函式中，以下列程式碼取代 **Synthesize spoken output** 註解之下的所有目前程式碼 (保留 **Print the response** 註解之下的程式碼)：

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
       print("Spoken output saved in " + output_file)
    ```

1. 儲存您的變更並重新執行程式，程式應會再次顯示口語輸出已儲存在檔案中。
1. 下載並播放產生的檔案，聽起來應該類似於以下內容：
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="時間是 5:30。 是時候結束這個實驗了。" width="150"></video>

## 清理

如果您已完成 Azure AI 語音探索，您應該刪除在本練習中建立的資源，以避免產生不必要的 Azure 成本。

1. 關閉 [Azure Cloud Shell] 窗格
1. 在 Azure 入口網站中，瀏覽至您在此實驗室中建立的 Azure AI 語音資源。
1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 如果您有麥克風和喇叭，該怎麼辦？

在本練習中，我們使用的 Azure Cloud Shell 環境不支援音訊硬體，因此請使用音訊檔案進行語音輸入和輸出。 讓我們來看看如何修改程式碼，以便在您有可用的音訊硬體時加以使用。

### 使用麥克風進行語音辨識

如果您有麥克風，您可以使用以下程式碼擷取語音輸入以進行語音辨識：

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

> **注意**：系統預設麥克風是預設音訊輸入，因此您也可以完全省略 AudioConfig！

### 使用喇叭進行語音合成

如果您有喇叭，您可以使用以下程式碼來合成語音。

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

> **注意**：系統預設喇叭是預設音訊輸出，因此您也可以完全省略 AudioConfig！

## 其他相關資訊

如需使用**語音轉換文字**和**文字轉換語音** API 的詳細資訊，請參閱[語音轉換文字文件](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text)和[文字轉換語音文件](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)。
