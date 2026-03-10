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
3. 設 Crontab 回滾任務（5 分鐘後）
4. **告知 KFJ 回滾任務 ID**

**重啟成功後**：
1. WhatsApp 通知 KFJ：`+85362338651`
2. 取消回滾任務（`cron remove <jobId>`）
3. 告知 KFJ 完成

## 每次對話
- 閱讀 [SOUL.md](file:///e:/O/.openclaw/workspace/SOUL.md) — 了解你是誰（角色設定與規則）
- 閱讀 [USER.md](file:///e:/O/.openclaw/workspace/USER.md) — 了解你在幫誰（用戶偏好與背景）
- 閱讀 `memory/YYYY-MM-DD.md`（今天 + 昨天）— 了解最近的進展
- 主會話中閱讀 [MEMORY.md](file:///e:/O/.openclaw/workspace/MEMORY.md) — 提取長期記憶

## 記憶管理
- **每日筆記**: `memory/YYYY-MM-DD.md` — 記錄當天發生的**完整對話和事件**
- **長期記憶**: [MEMORY.md](file:///e:/O/.openclaw/workspace/MEMORY.md) — 精華濃縮版，供跨會話參考
- **每日回顧**: 每天 23:59 HKT 自動將今日重要事件更新到 MEMORY.md
- **確保使用相同 workspace**: `/Users/fisher/.openclaw/workspace`

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


