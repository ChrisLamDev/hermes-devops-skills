---
name: macos-ram-swap-analysis
description: Use when diagnosing macOS "系統應用程式記憶體不足" (system application memory insufficient) warnings. Covers checking VM stats, swap usage, top RAM consumers, and identifying whether the issue is RAM shortage vs swap congestion.
  triggers:
    - ram
    - swap
    - memory
    - 內存
    - ram不足

---

# macOS RAM & Swap 問題診斷流程

當用戶報告 macOS 彈出「系統應用程式記憶體不足」警告時，按照以下步驟診斷：

## 第 1 步：睇 RAM 同 Swap 總體狀況

```bash
# 記憶體壓力概覽
memory_pressure

# VM 統計（特別留意 swapins/swapouts 同 page reclaims）
vm_stat

# 睇 swap file 已用幾多
sysctl vm.swapusage

# 咗硬碟空間（swap file 需要足夠 disk space）
df -h /
```

## 第 2 步：找出最食 RAM 嘅程序

```bash
# 按 RSS 排序，顯示 top 30
ps -eo pid,rss,comm | sort -k2 -rn | head -30 | awk '{printf "%s\t%.2f MB\t%s\n", $1, $2/1024, $3}'

# macOS Activity Monitor style 詳細版
top -l 1 -n 20 -o mem
```

## 第 3 步：檢查背景服務同登入項

```bash
# launchctl 背景服務（按記憶體用量）
launchctl list | while read -r line; do
  pid=$(echo "$line" | awk '{print $1}')
  if [ "$pid" != "-" ] && [ "$pid" -gt 0 ] 2>/dev/null; then
    rss=$(ps -o rss= -p "$pid" 2>/dev/null)
    if [ -n "$rss" ]; then
      echo "$((rss/1024)) MB - $(echo "$line" | awk '{print $3}')"
    fi
  fi
done | sort -rn | head -20

# 登入啟動項目
osascript -e 'tell application "System Events" to get the name of every login item'

# 已載入嘅 launchd plist（用戶層級）
launchctl list | awk '{print $3}' | sort
```

## 第 4 步：常見大食嫌疑犯

| 程序 | 典型 RAM | 可閂？ |
|------|---------|-------|
| OrbStack Helper | 500MB-1.6GB | ⚠️ 背景 Docker |
| mds_stores / corespotlightd | 200-400MB | ✅ Spotlight 可用 |
| siriactionsd / sirittsd | 100-200MB | ✅ Siri 唔用可熄 |
| WindowServer | 100-500MB | ❌ 系統必要 |
| launchservicesd | 200-800MB | ❌ 系統必要 |
| usernoted | 100-300MB | ❌ 系統必要 |
| Software Update | 50-100MB | ✅ 可暫停 |

## 第 5 步：判斷問題類型

**典型模式：「RAM仲有位但Swap塞死」**
- memory_pressure 顯示 free 多（>60%）
- vm_stat 顯示 page reclaims 高
- sysctl vm.swapusage 顯示 swap 接近爆（>80% of total）
- 原因：前一批大食 app 開過，macOS 搬咗數據去 swap，app 關咗但 swap 冇清

**解決方案（按建議順序）：**
1. **重啟電腦** — 最徹底，清 cache + swap + 壓縮記憶體
2. **`sudo purge`** — 清唔活躍記憶體 page cache，有助減輕 swap 壓力（唔及重啟徹底）
3. **熄非必要背景服務** — Spotlight、Siri、Microsoft/Zoom/Java 自動更新
4. **減少 Swap 使用量** — 唔建議改動 vm.swappiness，macOS 唔同 Linux

## 第 6 步：生成俾用戶睇嘅報告

用上述數據整理成直觀表格，用大白話解釋：
- 「Swap 好似洗碗盤，你洗完碗（關 App）但盤入面仲塞住，下一餐就冇位放」
- 「RAM 仲有大把空位，但塞住嘅 swap 令系統以為記憶體唔夠」

## 注意事項

- 用戶係 Macau/粵語用戶，用廣東話解釋
- 唔好用太多英文技術術語
- 問用戶想點搞：簡單重啟 / 幫 Kill 背景程序 / 指導佢熄特定服務
