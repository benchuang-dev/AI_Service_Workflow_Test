# AI Service Workflow Test

這個 repo 用來測試 SpotCam 新增付費 AI 方案時的 GitHub Issue Form 與交接文件。

目標是讓 PM 先用固定欄位描述 AI 服務需求，工程補完 review 後，再把整理好的 prompt 交給 Codex 實作，避免在實作時才補問 `pid`、`eid`、API、素材、多國語、事件或推播規格。

## 使用方式

1. PM 在 GitHub Issues 選擇 `AI Service Addition` template。
2. PM 填完服務名稱、參考 AI、`pid` / `eid`、API、支援條件、UI、事件、推播、deeplink、素材、多國語與 QA 資訊。
3. 工程補完 `Engineering review`，確認 final tier、風險、缺少資訊與 Codex execution order。
4. 工程把 issue 內的 `Codex prompt` 貼給 Codex。
5. Codex 依 workflow 先做非修改性盤點，確認後再實作與驗證。

## 文件

- GitHub Issue Form: `.github/ISSUE_TEMPLATE/ai-service-addition.yml`
- Intake workflow: `docs/agent-workflows/ai-service-addition-intake.md`
- Tier playbook: `docs/agent-workflows/ai-service-addition-tiers.md`
- Validation workflow: `docs/agent-workflows/ai-service-addition-validation.md`

## 適用範圍

此 workflow 適用於所有付費 AI 方案，包含但不限於：

- 只有服務介紹與訂閱入口的 AI 方案。
- 有智慧 AI 設定頁與 detail payload 的 AI 方案。
- 有事件、Timeline、事件篩選、匯出、推播圖片與 deeplink 的完整 AI 方案。

如果需求仍有未知資訊，PM 或工程應明確填 `Unknown`，並在 `Blockers / missing PM info` 中列出，不要讓 Codex 猜產品行為或 API contract。
