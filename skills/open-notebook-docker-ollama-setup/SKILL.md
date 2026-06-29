---
name: open-notebook-docker-ollama-setup
description: Use when deploying Open Notebook with local Ollama via Docker.
  用 Docker 部署 Open Notebook（Notebook LM 開源版），
  連接本地 Ollama 模型，資源優化啟動/關閉 script 設定。
tags: [Docker, Ollama, open-notebook, NotebookLM, OrbStack, macOS]
  triggers:
    - ollama docker
    - open notebook
    - local llm

---

# 🖥️ Open Notebook Docker + Ollama 部署

## 何時使用

- 需要部署 Open Notebook（Google Notebook LM 開源版）畀非技術用戶
- 需要用本地 Ollama 模型代替 API key
- 需要節省 Docker 資源（唔用嗰陣完全釋放 RAM）

## 概要

Open Notebook 係一個「私人圖書館 + 研究助理 + 播客製作室」三合一嘅開源工具。
佢用 Docker + SurrealDB（資料庫），支援 18+ AI 模型包括 Ollama 本地模型。

## 安裝步驟

### 1. 建立目錄 + 下載 docker-compose.yml

```bash
mkdir -p ~/open-notebook
curl -o ~/open-notebook/docker-compose.yml https://raw.githubusercontent.com/lfnovo/open-notebook/main/docker-compose.yml
```

### 2. 設定 Encryption Key

編輯 `~/open-notebook/docker-compose.yml`，改呢行：
```
- OPEN_NOTEBOOK_ENCRYPTION_KEY=change-me-to-a-secret-string
```
改做你自己嘅 secret（例如 `my-secret-key-2026`）。

### 3. 啟動服務

```bash
cd ~/open-notebook && docker compose up -d
```

### 4. 打開 UI

瀏覽器開 http://localhost:8502

## Ollama 設定（Open Notebook 入面）

### 先確認 Ollama 行緊

```bash
ollama list  # 睇下有冇模型
```

### Open Notebook UI 入面設定

1. Menu → **Settings** → **API Keys**
2. Click **Add Credential**
3. Provider 揀 **Ollama**
4. **API Base URL** 填：`http://host.docker.internal:11434`
   - （Open Notebook 喺 Docker container 入面，要用 `host.docker.internal` 先連到 Mac 本機嘅 Ollama）
5. **Name** 隨便填（如「本地Ollama」）
6. Click **Save**
7. Click **Test Connection** → 應該成功

### 模型發現（Discover Models）— 逐類型添加

Open Notebook 嘅模型發現係逐個類型（Language / Embedding / TTS / STT）分開做嘅。
**Embedding 模型唔會自動出現喺 Default Model Assignment dropdown 入面**，要手動加。

1. 喺 Ollama Provider 區域，Click **Models** 按鈕
2. 彈出對話框，**Model Type** dropdown 預設係 **Language**：
   - 如果加普通對話/分析模型，就揀 Language → tick 模型 → **Add (N)**
3. **Embedding 模型要分開加**：
   - 再次 Click **Models** 按鈕
   - **Model Type** dropdown 揀 **Embedding**
   - Tick qwen2.5:7b（或其他你想用嘅 embedding 模型）→ **Add (N)**
4. 你會見到 Ollama 區域多咗一行 `Embedding qwen2.5:7b (Embedding)`
5. 最後，喺上面嘅 **Default Model Assignments** 區域，Click **Auto-assign Defaults** 按鈕
   - 呢個先會自動將 qwen2.5:7b 分配到 Embedding Model dropdown

> **經驗教訓**：Open Notebook 嘅 Embedding Model dropdown 唔會自動 detect 已經加咗嘅 Language 類型模型去做 embedding。一定要先透過 Discover Models 對話框嘅 **Model Type → Embedding** 分類加一次，再撳 **Auto-assign Defaults** 先得。淨係撳 Auto-assign 但冇加 Embedding 類型嘅模型會出 Warning: "No models available to assign"。

## 資源優化：唔用時完全釋放 RAM

OrbStack（Docker 引擎）背景 idle 時大約食 ~2GB RAM。
Open Notebook 兩個 container（api + surrealdb）大約食 ~500MB RAM。

### 建立開關 script

```bash
cat > ~/open-notebook/notebook.sh << 'SCRIPT'
#!/bin/bash
NOTEBOOK_DIR="$HOME/open-notebook"

start() {
    echo "🚀 啟動 OrbStack + Open Notebook..."
    open -a OrbStack
    sleep 8
    cd "$NOTEBOOK_DIR" && docker compose up -d
    echo "✅ http://localhost:8502"
    echo "   熄：bash ~/open-notebook/notebook.sh stop"
}

stop() {
    echo "🛑 停用 Open Notebook + OrbStack..."
    cd "$NOTEBOOK_DIR" && docker compose down
    osascript -e 'quit app "OrbStack"'
    echo "✅ 已釋放資源"
}

case "${1:-status}" in
    start) start ;;
    stop) stop ;;
    *) echo "用法: bash ~/open-notebook/notebook.sh [start|stop]" ;;
esac
SCRIPT
chmod +x ~/open-notebook/notebook.sh
```

### 熄咗 OrbStack 開機自動啟動

```bash
osascript -e 'tell application "System Events" to delete login item "OrbStack"'
```

### 日常用法

```bash
# 開
bash ~/open-notebook/notebook.sh start
# 熄（釋放 RAM）
bash ~/open-notebook/notebook.sh stop
```

## 注意事項 / Pitfalls

- **OrbStack 第一次彈三個選項**：揀 **Docker only**（最輕量，其他兩個 tick 唔剔）
- **docker compose down 後所有資料保留**：SurrealDB 資料喺 `~/open-notebook/surreal_data/`，唔會丟失
- **Port 衝突**：如果 8502 或 5055 被佔用，修改 docker-compose.yml 嘅 ports mapping
- **新版本**：用 `docker compose pull` 更新 image
- **Ollama 版本**：建議 0.24.0 以上，確保 API 相容
- **Ollama 模型建議**：`qwen2.5:7b` 理解力好又唔太大，適合 Notebook 分析工作
- **必須先用 `open -a OrbStack` 開 Docker 引擎**，之後先 `docker compose up -d`

## 相關檔案

- `~/open-notebook/docker-compose.yml` — Docker 設定
- `~/open-notebook/notebook.sh` — 開關 script
- `~/open-notebook/notebook_data/` — 用戶數據
- `~/open-notebook/surreal_data/` — 資料庫
