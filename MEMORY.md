# MEMORY.md - 長期記憶

## KFJ 制定的系統規則（自動生效）

### 刪除操作
- 任何刪除文件、軟件、技能、工具、插件等操作**必須與 KFJ 確認**，由 KFJ 決定是否執行

### Skill 安裝
- 所有 Skill 在安裝前，**必須先使用 Skill-vetter 進行安全性審查**
- 只有確認無安全風險後，才可進行安裝

### 敏感資訊管理
- Token、API Key、Secret Key、端口、OAuth 等敏感資訊**必須保存於 `~/.env` 環境變量文件中**
- 絕對保密，**絕不分享給任何人**
- **每次重啟終端後**，需要執行 `source ~/.env` 載入環境變量

### 配置變更與回滾機制
- 每次修改配置文件之前，**必須先 git commit**，並上傳到 GitHub
- 涉及 Gateway 重啟的操作（包括但不限於）：
  - 修改 .openclaw.json
  - 改 Channel
  - 升級插件
  - 改 LLM 模型
  - 更新 Openclaw
  - 創建/修改 Agents
- **操作前必須**：
  1. 先問 KFJ 獲得確認
  2. 執行備份：`cp openclaw.json openclaw.json.bak`
  3. 使用 Crontab 設置 5 分鐘後的回滾任務：`cp openclaw.json.bak openclaw.json` + 重啟 Gateway
  4. 告知 KFJ 回滾任務 ID
- **操作完成並成功重啟 Gateway 後**：告知 KFJ，然後取消回滾任務

---
