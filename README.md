# wazuh-mcp

## 📊 專案報告：Wazuh 與 Claude MCP Server 整合實作 (完整版)

### 1. 專案目標

在 Mac 環境下透過 Docker 部署 Wazuh 資安監控系統，並利用 **Claude Desktop** 的 **MCP (Model Context Protocol)** 功能。同時結合 **VS Code** 進行程式碼與設定檔管理，以及 **Strawberry** 相關環境支援，實現 AI 自動化查詢資安 Agent 狀態。

---

### 2. 環境準備與下載項目

在開始之前，本專案準備了以下工具：

* **Docker Desktop**: 用於執行 Wazuh Manager、Indexer 及 Dashboard 容器。
* **Claude Desktop**: 作為與 AI 對話的主介面。
* **Visual Studio Code (VS Code)**: 用於編輯 `claude_desktop_config.json` 與 `wazuh.yml`，確保語法高亮與 JSON 格式正確。
* **Strawberry (Perl/環境工具)**: 用於支援特定腳本執行或作為開發環境的相依元件。
* **Wazuh Docker 部署檔**: 從 Wazuh 官方 GitHub 取得的 `docker-compose.yml`。
* **MCP Server 鏡像**: 使用 `ghcr.io/gbrigandi/mcp-server-wazuh:latest`。

---

### 3. 核心執行步驟

#### 步驟一：啟動 Wazuh 服務

使用 Docker Compose 啟動全套 Wazuh 環境：

```bash
docker-compose up -d

```

* **關鍵動作**：確保 `docker-wazuh-manager-1` 容器正常運行。

#### 步驟二：使用 VS Code 進行 API 驗證與設定

* **驗證指令**：在 VS Code 的整合終端機執行 `curl -k -u wazuh:admin https://localhost:55000/manager/status`。
* **修復設定**：使用 VS Code 打開 `wazuh.yml`，修正重複的 Key 並將密碼統一為 `admin`。

#### 步驟三：配置 Claude MCP Server (關鍵橋樑)

利用 VS Code 編輯 `~/Library/Application Support/Claude/claude_desktop_config.json`：

* **克服障礙**：解決了 JSON 語法報錯（Unexpected character）。
* **最終設定檔**：
```json
{
  "mcpServers": {
    "WAZUH_FINAL_HOPE": {
      "command": "/usr/local/bin/docker",
      "args": [
        "run", "--rm", "-i", "--network", "single-node_default",
        "-e", "WAZUH_URL=https://docker-wazuh-manager-1:55000",
        "-e", "WAZUH_USER=wazuh", "-e", "WAZUH_PASS=admin",
        "-e", "VERIFY_SSL=false", "ghcr.io/gbrigandi/mcp-server-wazuh:latest"
      ]
    }
  }
}

```



---

### 4. 實作挑戰與 Strawberry 角色

1. **環境變數衝突**：在處理 Docker 與 Strawberry 環境時，需確保 `PATH` 不會互相干擾，讓 Docker 指令能優先被系統呼叫。
2. **網路隔離**：透過 `docker network ls` 確認網路名稱為 `single-node_default`。
3. **JSON 錯誤修復**：利用 VS Code 的格式化功能解決了 Line 20 的非空白字元錯誤。

---

### 5. 最終成果展示

當在 Claude 輸入「查詢 Agent 列表」後，成功獲取以下數據：

* **工具名稱**: `WAZUH_FINAL_HOPE`。
* **Agent ID**: 000 (Wazuh Manager)。
* **狀態**: 🟢 ACTIVE (活躍中)。
* **系統版本**: Ubuntu 20.04.6 LTS / Wazuh v4.7.2。
* <img width="2060" height="1544" alt="image" src="https://github.com/user-attachments/assets/88e7132a-5226-42c5-998a-76a0fc94e974" />
