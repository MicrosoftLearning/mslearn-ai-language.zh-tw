# 需求

## 在 Cloud Shell 中執行

* 具有 OpenAI 存取權的 Azure 訂閱
* 如果在 Azure Cloud Shell 中執行，請選擇 Bash 殼層。 Azure CLI 和 Azure Developer CLI 包含在 Cloud Shell 之中。

## 在本機執行

* 執行部署指令碼之後，即可在本機執行 Web 應用程式：
    * [Azure Developer CLI (azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * 具有 OpenAI 存取權的 Azure 訂閱


## 環境變數

`.env` 檔案是由 *azdeploy.sh* 指令碼所建立。 AI 模型端點、API 金鑰和模型名稱是在資源部署期間新增。

## Azure 資源部署

提供的 `azdeploy.sh` 會在 Azure 中建立必要資源：

* 請根據需求變更指令碼頂端的兩個變數，不要變更其他任何內容。
* 指令碼：
    * 使用 AZD 部署 *gpt-4o* 模型。
    * 建立 Azure Container Registry 服務
    * 使用 ACR 工作組建 Dockerfile 映像，並將其部署至 ACR
    * 建立 App Service 方案
    * 建立 App Service Web 應用程式
    * 在 ACR 中設定容器映像的 Web 應用程式
    * 設定 Web 應用程式環境變數
    * 指令碼會提供 App Service 端點

指令碼提供兩個部署選項：1. 完整部署;和 2. 僅重新部署映像。 如果您想在應用程式中實驗變更，選項 2 僅適用於部署後。 

> 注意：您可以使用命令 `bash azdeploy.sh` 在 PowerShell 或 Bash 中執行指令碼；此命令還允許您在 Bash 中執行指令碼，無需將其設為可執行檔。

## 本機開發

### 將 AI 模型佈建至 Azure

您可以在本機執行專案，並依照下列步驟只佈建 AI 模型：

1. **初始化環境** (選擇描述性名稱)：

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **重要**：此名稱會成為 Azure 資源名稱的一部分！  
   `--confirm` 旗標會將此設定為預設環境，不會另外提示。

1. **設定您的資源群組**：

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **登入和佈建 AI 資源**：

   ```bash
   az login
   azd provision
   ```

    > **重要**：請勿執行 `azd deploy`：此應用程式尚未在 AZD 範本中設定。

如果您只使用 `azd provision` 方法佈建模型，則必須使用下列項目在目錄的根目錄中建立 `.env` 檔案：

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

注意：

1. 該端點是模型的端點，應只包含 `https://<proj-name>.cognitiveservices.azure.com`。
1. API 金鑰是模型的金鑰。
1. 此模型是部署期間使用的模型名稱。
1. 您可以從 AI Foundry 入口網站擷取這些值。

### 在本機執行專案

專案雖是使用 **uv** 建立和管理，但不需要執行。 

如果您已安裝 **uv**，則請：

* 執行 `uv venv` 建立環境
* 執行 `uv sync` 新增套件
* 為 Web 應用程式建立 `uv run web` 別名來啟動 `flask_app.py` 指令碼。
* 使用 `uv pip compile pyproject.toml -o requirements.txt` 建立的 requirements.txt 檔案

如果沒有安裝 **uv**，則請：

* 建立環境：`python -m venv .venv`
* 啟動環境：`.\.venv\Scripts\Activate.ps1`
* 安裝相依性：`pip install -r requirements.txt`
* 從專案根目錄執行應用程式：`python .\src\real_time_voice\flask_app.py`
