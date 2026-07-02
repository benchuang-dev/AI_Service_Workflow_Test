# AI Service Addition Intake Workflow

本文件定義新增付費 AI 方案時的固定需求入口。目標是讓 PM 以產品需求角度填寫新 AI 服務本體，工程再補支援條件、API / payload 與實作推導，最後交給 Codex 盤點與實作。

此 workflow 只規範需求收斂與交接，不代表每次都要修改所有 AI 區塊。實作時仍需依實際 issue、參考服務與程式碼盤點縮小範圍。

## Entry Point

PM 應使用 GitHub 的 `AI Service Addition` issue form 建立需求：

```text
.github/ISSUE_TEMPLATE/ai-service-addition.yml
```

issue 建立後，工程需要補完 `Engineering 1-3` 區塊，把 PM 的新需求轉成支援條件、API / payload、標準 coverage 與實作順序，再把 `Codex prompt` 區塊貼給 Codex 進入實作對話。

分級判斷請先參考 `docs/agent-workflows/ai-service-addition-tiers.md`。實作後的最低驗證與 report 矩陣請參考 `docs/agent-workflows/ai-service-addition-validation.md`。

## Field Ownership

| 區塊 | 負責角色 | 用途 |
| --- | --- | --- |
| PM 1. 新增 AI 服務 | PM | 定義服務名稱、類型、`pid`、reference service、入口、排序與價格方案。 |
| PM 2. 智慧 AI 設定 | PM | 定義設定頁需求、新增設定項、reference AI setting、特殊 UI 與 Phone / Pad 需求。 |
| PM 3. 事件規格 | PM | 定義是否有事件、正式 `eid` 與事件顯示名稱。 |
| PM 4. 事件顯示 | PM | 定義事件列表、Timeline、事件篩選、All Events、匯出、播放器標題與推播 channel/name。 |
| PM 5. 推播與 Deeplink | PM | 定義推播圖片、Android deeplink 與目的頁。 |
| PM 6. 文案 | PM / 翻譯 | 一條文案一列，讓工程能明確計算需要的字串。 |
| PM 7. 素材 | PM / 設計 | 一個素材用途一列，讓工程能明確計算需要的圖片。 |
| Engineering 1. 支援條件 | 工程 | 定義後端支援欄位、SN/FW 是否硬編、訂閱 camera 篩選與設定頁顯示條件。 |
| Engineering 2. API / Payload | 工程 | 定義服務狀態來源、設定讀寫來源、target、payload 與預設值 fallback。 |
| Engineering 3. 實作推導 | 工程 | 確認 final tier、reference implementation、Phone / Pad scope、standard coverage、resource naming、blockers 與 Codex execution order。 |
| Codex prompt | 工程 | 整理成可直接貼給 Codex 的實作啟動 prompt。 |

## PM Required Information

PM 不需要讀程式碼，但 issue 至少要能回答下列問題：

- 服務名稱、service type、`pid`、reference AI service、服務入口、介紹頁、排序與價格 / 訂閱 / 免費試用規則。
- 智慧 AI 設定頁需求、reference AI setting、新增設定項、特殊 UI 與 Phone / Pad 是否支援。
- 正式事件 `eid` 與事件顯示名稱；不要填開發期間暫定或已被替換的 id。
- 事件顯示需要覆蓋哪些 surface，例如事件列表、Timeline、filter、All Events、export、播放器標題與推播 channel/name。
- 推播圖片與 Android deeplink 需求。
- 文案與素材需逐條列出；不要填本機個人路徑、Android resource name、payload 或 backend 欄位。

如果 PM 缺資訊，工程應在 issue 的 `Blockers / missing PM info` 明確列出，不要讓 Codex 猜測。

## Engineering Review

工程收到 issue 後，先完成下列判斷：

| 欄位 | 判斷規則 |
| --- | --- |
| 支援條件 | 後端支援欄位、SN/FW 是否硬編、訂閱 / 免費試用選 camera 篩選與設定頁顯示條件。 |
| API / Payload | 服務狀態來源、設定讀取來源、設定寫入 target、設定 payload 與預設值 fallback。 |
| 實作推導 | Final tier、reference implementation、Phone / Pad scope、standard event coverage、resource naming、missing info / blockers、Codex execution order。 |

## Codex Execution Contract

把 issue 的 `Codex prompt` 貼給 Codex 後，Codex 必須先做非修改性的盤點：

1. 讀取 `AGENTS.md` 與本文件。
2. 讀取 `docs/agent-workflows/ai-service-addition-tiers.md` 與 `docs/agent-workflows/ai-service-addition-validation.md`。
3. 依 PM 與 Engineering 區塊列出本次會碰到的檔案群，例如 AI service constants、服務頁、智慧 AI 設定、API wrapper、事件列表、Timeline、FCM、deeplink、字串與 drawable。
4. 從 reference service 與 final tier 推導固定規則，例如新增 `eid` 時需檢查事件列表、Timeline、filter、export、FCM、All Events、播放器標題與 deeplink。
5. 標出風險與待確認資訊，尤其是 `Unknown` 後端行為、支援條件、事件拆分、Phone/Pad 不對稱、缺素材與缺翻譯。
6. 將 PM 的使用者視角需求轉成實作細節，例如 endpoint、target、payload、resource name、drawable name 與事件 helper。
7. 確認範圍後再修改檔案。

實作完成後，Codex 需要依 `docs/agent-workflows/ai-service-addition-validation.md` 執行對應驗證。不能執行的驗證要明確記錄原因，不可寫成已通過。

## Definition Of Ready

AI 服務新增 issue 進入 Codex 實作前，至少需要滿足：

- PM 1-7 已填，且文案與素材是一條一列。
- 工程已補 Engineering 1-3，包含支援條件、API / payload、實作推導與 blockers。
- Codex prompt 已確認會讀取本 workflow 並先做非修改性盤點。
