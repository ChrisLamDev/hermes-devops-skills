---
name: macos-deploy-python-tool-from-github
description: Use when deploying a Python tool from GitHub on macOS Apple Silicon.
tags: [macos, python, deployment, ffmpeg, github, streamlit]
  triggers:
    - deploy python
    - github python tool
    - macos python

---

# macOS 部署開源 Python 工具（GitHub → 本地運行）

從 GitHub clone 一個 Python 開源項目，喺 macOS Apple Silicon 上完整部署並可使用。

## 典型流程

### 1. Clone 項目
```bash
git clone https://github.com/username/project.git ProjectName
cd ProjectName
```

### 2. 檢查環境
```bash
python3 --version      # 需要 Python 3.10+
pip3 --version
which git
ffmpeg -version | head -2    # 影片處理項目需要
```

### 3. 安裝依賴
睇 README 推薦方式，通常兩種：
```bash
# 方式 A: uv（推薦，macOS 已預裝）
uv sync --frozen

# 方式 B: pip
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 4. 設定 config
項目通常有 `config.example.toml`，複製一份：
```bash
cp config.example.toml config.toml
```
然後按 README 改 key 同 provider。

### 5. 連接本地 Ollama LLM（如適用）
如果項目支援 Ollama 做 LLM provider：
```bash
# 確認 Ollama 已裝
ollama --version

# 確認有 model
ollama list

# config.toml 入面設定
# llm_provider = "ollama"
# ollama_base_url = "http://localhost:11434/v1"   ← 注意要 /v1
# ollama_model_name = "qwen2.5:7b"               ← 揀一個本地 model
```

### 6. 啟動服務
```bash
uv run streamlit run ./webui/Main.py --browser.gatherUsageStats=False
```
或者直接：
```bash
cd ProjectName && uv run streamlit run ./webui/Main.py
```

## 常見問題 & Fix

### ⚠️ FFmpeg x265 library missing
Error: `dyld: Library not loaded: /opt/homebrew/opt/x265/lib/libx265.215.dylib`

原因：Homebrew 更新咗 x265 但 FFmpeg 冇同步更新。

Fix:
```bash
brew reinstall ffmpeg
```
驗證：`ffmpeg -version` 正常顯示就 OK。

### ⚠️ .venv path 錯誤（搬遷項目後）
Error: `Failed to spawn: streamlit. Caused by: No such file or directory`

原因：搬咗項目 folder 但 .venv 入面仲係硬編碼舊 path。

Fix（重建 venv）：
```bash
cd /new/project/path
rm -rf .venv
uv sync --frozen
```

### ⚠️ config.toml 格式錯誤
Error: `TomlDecodeError: Found invalid character in key name`

原因：用 read_file 寫入時 embedded line number prefix。

Fix：用直接寫入方式，或者從 config.example.toml 重新複製再 patch。
```bash
cp config.example.toml config.toml
# 然後 patch 指定 key
```

### ⚠️ Streamlit 卡喺 Email prompt
頭一次啟動 Streamlit 會問 email，送個空 Enter 就得。
```bash
# 或者用 CLI flag skip:
--browser.gatherUsageStats=False
```

## Desktop Shortcut

建立一個 `.command` 檔案：
```bash
cat > ~/Desktop/ProjectName.command << 'EOF'
#!/bin/bash
cd /path/to/ProjectName
uv run streamlit run ./webui/Main.py --browser.gatherUsageStats=False
EOF
chmod +x ~/Desktop/ProjectName.command
```
雙击就會開 Terminal 啟動。

## 驗證 checklist
- [ ] git clone 成功
- [ ] uv sync / pip install 成功
- [ ] config.toml 設定正確（LLM provider, API keys, base_url）
- [ ] ffmpeg 正常運作
- [ ] 服務啟動後能訪問 localhost
- [ ] Desktop shortcut 雙击有反應
