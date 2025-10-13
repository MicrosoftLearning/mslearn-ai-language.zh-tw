---
lab:
  title: 開發 Azure AI Voice Live 語音代理程式
  description: 瞭解如何建立 Web 應用程式，啟用與 Azure AI Voice Live 代理程式的即時語音互動。
---

# 開發 Azure AI Voice Live 語音代理程式

在此練習中，您會完成以 Flask 為基礎的 Python Web 應用程式，啟用與代理程式的即時語音互動。 您可以新增程式碼來初始化工作階段，以及處理工作階段事件。 您會使用符合以下條件的部署指令碼：部署 AI 模型;使用 ACR 工作在 Azure Container Registry (ACR) 中建立應用程式映像，然後建立提取映像的 Azure App Service 執行個體。 若要測試應用程式，您需要具有麥克風和喇叭功能的音訊裝置。

雖然本練習以 Python 為基礎，您仍可開發類似其他特定語言 SDK 的應用程式，包括：

- [適用於 .NET 的 Azure VoiceLive 用戶端程式庫](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

在此練習中執行的工作：

* 下載應用程式的基礎檔案
* 新增程式碼以完成 Web 應用程式
* 檢閱整體程式碼基礎
* 更新並執行部署指令碼
* 檢視及測試應用程式

本練習大約需要 **30** 分鐘才能完成。

## 啟動 Azure Cloud Shell 並下載檔案

在本節練習中，您會下載包含應用程式基礎檔案的 ZIP 壓縮檔案。

1. 在網頁瀏覽器中，瀏覽至 Azure 入口網站 [https://portal.azure.com](https://portal.azure.com)；若出現提示，請使用您的 Azure 認證登入。

1. 使用頁面上方搜尋欄右側的 **[\>_]** 按鈕，就能從 Azure 入口網站建立新的 Cloud Shell，並選取 ***Bash*** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **備註**：如果您之前就已建立使用 *Bash* 環境的 Cloud Shell，請將原先的設定切換成 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

1. 請在 **Bash** 殼層中執行下列命令，下載並解壓縮練習檔案。 第二個命令也會變更為練習檔案的目錄。

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## 新增程式碼以完成 Web 應用程式

下載完練習檔案後，請接著新增程式碼來完成應用程式。 下列步驟是在 Cloud Shell 中執行。 

>**提示：** 拖曳頂端框線調整 Cloud Shell 的大小，以顯示更多資訊和程式碼。 您也可以使用最小化和最大化按鈕，在 Cloud Shell 和主要入口網站介面之間切換。

請先執行下列命令變更為 *src* 目錄，之後再繼續練習。

```bash
cd src
```

### 新增程式碼，實作 Voice Live 助理

在本節中，您會新增程式碼來實作 Voice Live 助理。 **\_\_init\_\_** 方法會儲存 Azure VoiceLive 連線參數來初始化語音助理 (端點、認證、模型、語音和系統指示)，以及設定執行階段狀態變數來管理連線生命週期，並在交談期間處理用戶中斷。 **start** 方法會匯入必要的 Azure VoiceLive SDK 元件，用以建立 WebSocket 連線並設定即時語音工作階段。

1. 執行下列命令，開啟 *flask_app.py* 檔案進行編輯。

    ```bash
    code flask_app.py
    ```

1. 在程式碼中搜尋 **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** 註解。 複製下列程式碼，輸入至註解正下方。 請務必檢查縮排。

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. 按下 **Ctrl + S** 儲存變更，並讓編輯器保持開啟，方便下一節使用。

### 新增程式碼，實作 Voice Live 助理

在本節中，您會新增程式碼來設定 Voice Live 工作階段。 這些程式碼會指定形式 (API 不支援僅限音訊)、定義助理行為的系統指示、回應的 Azure TTS 語音、輸入和輸出資料流的音訊格式，以及指定模型在使用者開始和停止說話時如何偵測的伺服器端語音活動偵測 (VAD)。

1. 在程式碼中搜尋 **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** 註解。 複製下列程式碼，輸入至註解正下方。 請務必檢查縮排。

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. 按下 **Ctrl + S** 儲存變更，並讓編輯器保持開啟，方便下一節使用。

### 新增程式碼來處理工作階段事件

在本節中，您會新增程式碼來新增 Voice Live 工作階段的事件處理常式。 事件處理常式會在對話生命週期內回應關鍵的 VoiceLive 工作階段事件：工作階段可供使用者輸入時，發出 **_handle_session_updated** 訊號；**_handle_speech_started** 會偵測使用者開始說話的時機，停止任何正在進行的助理音訊播放並取消進行中的回應，實作中斷邏輯，實現自然的對話流程，**_handle_speech_stopped** 則會在使用者說話完畢，助理開始處理輸入時接手進行。

1. 在程式碼中搜尋 **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** 註解。 複製下方程式碼，輸入至註解正下方，然後檢查縮排。

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. 按下 **Ctrl + S** 儲存變更，並讓編輯器保持開啟，方便下一節使用。

### 檢閱應用程式中的程式碼

到目前為止，您已將程式碼新增至應用程式，實作代理程式並處理代理程式事件。 請花幾分鐘檢閱完整的程式碼和註解，進一步瞭解應用程式如何處理用戶端狀態和作業。

1. 完成後，輸入 **Ctrl + Q** 離開編輯器。 

## 更新並執行部署指令碼

在本節中，您會稍微變更 **azdeploy.sh** 部署指令碼，然後執行部署。 

### 更新部署指令碼

您應該只在 **azdeploy.sh** 部署指令碼頂端變更兩個值。 

* **rg** 值會指定要包含部署的資源群組。 如果您需要部署到特定資源群組，則可接受預設值或輸入自訂值。

* **location** 值會設定部署的區域。 練習中使用的 *gpt-4o* 模型可以部署到其他區域，但在特定區域中可能受到限制。 如果所選區域中的部署失敗，請嘗試使用 **eastus2** 或 **swedencentral**。 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. 在 Cloud Shell 中執行下列命令，開始編輯部署指令碼。

    ```bash
    cd ~/01-voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. 根據您的需求更新 **rg** 和 **location** 的值，然後按下 **Ctrl + S** 儲存變更，再按下 **Ctrl + Q** 離開編輯器。

### 執行部署指令碼

部署指令碼會部署 AI 模型並在 Azure 中建立必要資源，以在 App Service 中執行容器化應用程式。

1. 在 Cloud Shell 中執行下列命令，開始部署 Azure 資源和應用程式。

    ```bash
    bash azdeploy.sh
    ```

1. 為初始部署選取**選項 1**。

    部署作業應該會在 5-10 分鐘內完成。 部署期間，系統可能顯示下列資訊/動作：
    
    * 如果系統提示您向 Azure 進行驗證，請遵循呈現的指示。
    * 如果系統提示您選取訂閱，請用箭頭鍵醒目提示您的訂閱，然後按下 **Enter** 鍵。 
    * 您可能會在部署期間看到一些警告，這些警告可以忽略。
    * 如果部署項目在 AI 模型部署期間失敗，請變更部署指令碼中的區域，然後再試一次。 
    * Azure 中的區域有時可能呈現忙碌狀態，導致部署時間中斷。 如果部署項目在模型部署重新執行部署指令碼之後失敗。

## 檢視及測試應用程式

部署完成「部署完成！」時 訊息將會在殼層中，並隨附 Web 應用程式的連結。 您可以選取該連結，或瀏覽至 App Service 資源，從該處啟動應用程式。 應用程式可能需要幾分鐘載入時間。 

1. 選取 [啟動工作階段]**** 按鈕，連線到模型。
1. 系統會提示您提供應用程式對音訊裝置的存取權。
1. 應用程式提示您開始說話時，開始與模型對話。

疑難排解：

* 如果應用程式回報缺少環境變數，請在 App Service 中重新啟動應用程式。
* 如果您在應用程式顯示的記錄中看到過多*音訊區塊*訊息，請選取 [停止工作階段]****，然後再次啟動工作階段。 
* 如果應用程式完全無法運作，請再次確認您已新增所有程式碼，並適度調整縮排。 如果您需要進行任何變更，請重新執行部署，然後選取**選項 2**，使系統只更新映像。

## 清除資源

在 Cloud Shell 中執行下列命令，移除為此練習部署的所有資源。 系統會提示您確認刪除資源。

```
azd down --purge
```