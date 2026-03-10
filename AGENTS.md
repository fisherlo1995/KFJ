# AGENTS.md - 助手工作指南

## 自動備份和回滾（必須遵守）
- **涉及 Gateway 重啟的配置修改，必須執行以下 3 個備份步驟，缺一不可：**

### 1. Git Commit（本地）
```bash
git add -A && git commit -m "Backup: $(date)"
```

### 2. 備份 openclaw.json
```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak
```

### 3. GitHub 推送
```bash
git push origin main
```

### 回滾任務（macOS 系統 Crontab）
```bash
# 設置 5 分鐘後回滾
crontab -l 2>/dev/null | { cat; echo "$(date '+\%M \%H \%d \%m \%Y' -d '+5 minutes') cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json && /usr/local/bin/openclaw gateway restart"; } | crontab -

# 查看回滾任務
crontab -l | grep openclaw

# 取消回滾任務
crontab -l 2>/dev/null | grep -v openclaw | crontab -
```

### 從 Git 還原備份
```bash
# 從 GitHub 還原
git fetch origin
git checkout origin/main -- .openclaw_backup/openclaw.json

# 從本地 Git 還原（指定 commit）
git checkout <commit-hash> -- .openclaw_backup/openclaw.json
```

---

### Gateway 重啟操作前
涉及以下操作時，**必須先獲得 KFJ 確認**：
- 修改 .openclaw.json
- 改 Channel
- 升級插件
- 改 LLM 模型
- 更新 Openclaw
- 創建/修改 Agents

**確認後操作**：
1. 備份：`cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak`
2. Git Commit：`git add -A && git commit -m "Backup: before config change"`
3. **在 macOS 系統層面設 Crontab 回滾任務（5 分鐘後）**
4. **告知 KFJ 回滾任務 ID**

**重啟成功後**：
1. WhatsApp 通知 KFJ：`+85362338651`
2. 取消 macOS 系統回滾任務
3. 告知 KFJ 完成

## 每次對話
- 閱讀 [SOUL.md](file:///e:/O/.openclaw/workspace/SOUL.md) — 了解你是誰（角色設定與規則）
- 閱讀 [USER.md](file:///e:/O/.openclaw/workspace/USER.md) — 了解你在幫誰（用戶偏好與背景）
- 閱讀 `memory/YYYY-MM-DD.md`（今天 + 昨天）— 了解最近的進展
- 主會話中閱讀 [MEMORY.md](file:///e:/O/.openclaw/workspace/MEMORY.md) — 提取長期記憶

## 記憶管理（三層記憶系統）

### Tier-0 前額葉（絕對權威區）
- **位置**: 與 Agent 同目錄 (`~/workspace/`)
- **文件**: `IDENTITY.md`, `USER.md`, `MEMORY.md`, `AGENTS.md`, `SOUL.md`
- **特性**: 
  - 最高權威，不可覆蓋
  - 每次對話必須閱讀
  - 長期生效的規則和偏好

### Tier-1 新皮層（深度存儲區）
- **位置**: Memos (Docker: `localhost:5230`)
- **API**: `http://localhost:5230/api/v1`
- **認證**: Bearer Token (`MEMOS_TOKEN`)
- **同步頻率**: **每 30 分鐘** (8:00-23:00 HKT)
- **內容**: 所有對話和事件的完整備份
- **特性**:
  - 接近實時同步，不遺漏任何細節
  - 可跨設備訪問
  - 通過 API 自動同步

### Tier-2 海馬體（每日草稿本）
- **位置**: `memory/` + `memory/evolution/`
- **內容**: 
  - `memory/YYYY-MM-DD.md` - 每日筆記
  - `memory/evolution/` - 演進過程記錄
- **特性**:
  - 每日自動生成
  - 23:59 HKT 同步到 Memos + 更新 MEMORY.md
  - 臨時性和過渡性內容

---

### 自動同步 Cron（23:59 HKT）
1. 讀取 `memory/YYYY-MM-DD.md`
2. 摘要重要事件到 `MEMORY.md` (Tier-0)
3. 同步完整內容到 Memos (Tier-1)
4. WhatsApp 通知完成

## 安全與行為準則
- **隱私**: 絕不洩漏用戶隱私數據
- **謹慎**: 執行破壞性操作（如 `rm`）前必須詢問；優先使用 `trash`
- **確認**: 遇到不確定的情況，先請示 Fisher 再行動

## 對外 vs 對內操作
- **自由操作**: 讀取文件、代碼搜索、整理資料、自主學習
- **強制確認**: 發送郵件、發送推文、執行任何離開本機網絡的操作

## 心跳監測
- 收到心跳信號時，檢查 [HEARTBEAT.md](file:///e:/O/.openclaw/workspace/HEARTBEAT.md) 中的任務清單
- 若無異常，回覆 `HEARTBEAT_OK`

## 雙級守護進程

### 架構
- **Tier 1**: 進程檢測 + 端口檢測
- **Tier 2**: 失敗計數 (3次) → 強制重啟 → 備份還原

### 腳本位置
- `/Users/fisher/.openclaw/scripts/openclaw-watchdog.sh`

### 啟動守護進程
```bash
# 方法1: launchd (推薦)
cp ~/.openclaw/scripts/com.openclaw.watchdog.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.openclaw.watchdog.plist

# 方法2: 手動運行
~/.openclaw/scripts/openclaw-watchdog.sh

# 查看日誌
tail -f ~/.openclaw/logs/watchdog.log
```

### 守護邏輯
1. 每分鐘檢查端口 28888 和進程狀態
2. 首次失敗 → 發送 WhatsApp 通知 + 重啟 OpenClaw
3. 2次失敗 → 發送 WhatsApp 通知 + 強制 kill + 重啟
4. 3次失敗 → 發送 WhatsApp 通知 + 還原最新備份 + 重啟
5. 恢復正常 → 發送 WhatsApp 通知


