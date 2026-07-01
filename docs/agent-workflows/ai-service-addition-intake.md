# AI Service Addition Intake Workflow

本文件定義新增付費 AI 方案時的固定需求入口。目標是讓 PM、工程與 Codex 使用同一份 GitHub Issue 資訊，避免在實作時猜 `pid`、`eid`、API contract、支援條件、UI、事件鏈路、推播、deeplink、素材或多國語。

此 workflow 只規範需求收斂與交接，不代表每次都要修改所有 AI 區塊。實作時仍需依實際 issue、參考服務與程式碼盤點縮小範圍。

## Entry Point

PM 應使用 GitHub 的 `AI Service Addition` issue form 建立需求：

```text
.github/ISSUE_TEMPLATE/ai-service-addition.yml
```

issue 建立後，工程需要補完 `Engineering review` 區塊，再把 `Codex prompt` 區塊貼給 Codex 進入實作對話。

分級判斷請先參考 `docs/agent-workflows/ai-service-addition-tiers.md`。實作後的最低驗證與 report 矩陣請參考 `docs/agent-workflows/ai-service-addition-validation.md`。

## Field Ownership

| 區塊 | 負責角色 | 用途 |
| --- | --- | --- |
| Service display name | PM | 定義對外顯示名稱。 |
| Service type | PM | 指定影像 AI、聲音 AI、混合 AI 或其他服務類型。 |
| Reference AI service | PM / 工程 | 指定最接近的既有 AI，讓 UI、價格、API 與事件行為有明確比照對象。 |
| Launch target | PM | 判斷 staging、prod、POC 或限定帳號發布。 |
| PID / EID | PM / 工程 | 提供服務 `pid`、事件 `eid` 或事件集合；若未定案必須標 `Unknown`。 |
| API / payload contract | PM / 後端 / 工程 | 定義讀取欄位、寫入 target、payload、預設值與後端 readiness。 |
| Support rule | PM / 後端 / 工程 | 定義支援產品、firmware、帳號、後端 support flag 與 unsupported 行為。 |
| Pricing / trial | PM | 定義價格、試用與參考方案。 |
| UI scope | PM / 設計 / 工程 | 定義服務頁、設定頁、Dialog、Phone/Pad 與深色模式。 |
| Event flow | PM / 工程 | 定義事件列表、Timeline、filter、export、播放器標題與圖片支援。 |
| Notification / deeplink | PM / web / 工程 | 定義 FCM 顯示、channel/name、推播圖片與 web/app deeplink。 |
| Strings / assets | PM / 設計 / 翻譯 | 列出 resource name、來源、語言與缺漏。 |
| Acceptance and QA | PM / QA | 提供測試帳號、測試 SN、必測流程與 release deadline。 |
| Engineering review | 工程 | 確認 final tier、風險、API readiness、缺項與 Codex execution order。 |
| Codex prompt | 工程 | 整理成可直接貼給 Codex 的實作啟動 prompt。 |

## PM Required Information

PM 不需要讀程式碼，但 issue 至少要能回答下列問題：

- 新 AI 服務的對外名稱、服務類型、launch 目標與最接近的既有 AI。
- `pid`、`eid` 或事件集合；如果事件會拆多種，需列出每個 eid 的顯示文字與意義。
- 後端 API 是否 ready，包含 service status 欄位、detail 讀寫欄位、target、payload 範例與預設值。
- 支援條件由後端 support flag、SN/FW 規則、帳號限制或訂閱 camera dialog 控制。
- 價格與試用是否沿用既有 AI 服務。
- Phone / Pad 是否都要支援，服務入口與設定頁排序要放在哪裡。
- 是否需要事件列表、Timeline、事件篩選、All Events、export、播放器標題、推播圖片與 deeplink。
- 字串與素材是否已提供；缺翻譯或缺圖要列出，不要留空。
- 驗收需要的測試帳號、測試 camera / SN、build variant 與必測流程。

如果 PM 缺資訊，工程應在 issue 的 `Blockers / missing PM info` 明確列出，不要讓 Codex 猜測。

## Engineering Review

工程收到 issue 後，先完成下列判斷：

| 欄位 | 判斷規則 |
| --- | --- |
| Final tier | 依 `Service-only`、`Settings-service`、`Full-event-service` 判斷實作與驗證範圍。 |
| Reference implementation | 指定要比照的既有 AI 服務、設定頁、事件、推播或 deeplink。 |
| Risk level | API 未 ready、事件拆分、多語/素材缺漏、Phone/Pad 不對稱或推播/deeplink 皆會提高風險。 |
| Phone/pad required | 明確標 `Phone only`、`Pad only` 或 `Both`。若既有服務 phone/pad 對稱，預設檢查 Both。 |
| API readiness | 後端欄位未定案時，不應讓 Codex 自行定義 API contract。 |
| Event chain required | 只要新增 `eid`，就需檢查事件列表、Timeline、filter、export、FCM 與 All Events。 |
| Push/deeplink required | 明確標示是否包含 FCM 圖片、channel/name 與 web/app deeplink。 |
| Asset/string readiness | 缺正式素材或翻譯時，需列出 placeholder resource name 與待補語言。 |
| Codex execution order | 先盤點，再實作，再驗證；需要分段時列出順序。 |
| Manual QA owner | 指定誰負責實機、emulator、測試帳號、SN 或 staging 驗收。 |
| Blockers / missing PM info | 缺 pid/eid、API、支援條件、素材、翻譯或測試設備時列在這裡。 |

## Codex Execution Contract

把 issue 的 `Codex prompt` 貼給 Codex 後，Codex 必須先做非修改性的盤點：

1. 讀取 `AGENTS.md` 與本文件。
2. 讀取 `docs/agent-workflows/ai-service-addition-tiers.md` 與 `docs/agent-workflows/ai-service-addition-validation.md`。
3. 依 issue 列出本次會碰到的檔案群，例如 AI service constants、服務頁、智慧 AI 設定、API wrapper、事件列表、Timeline、FCM、deeplink、字串與 drawable。
4. 標出風險與待確認資訊，尤其是 `Unknown` API、支援條件、事件拆分、Phone/Pad 不對稱、缺素材與缺翻譯。
5. 確認範圍後再修改檔案。

實作完成後，Codex 需要依 `docs/agent-workflows/ai-service-addition-validation.md` 執行對應驗證。不能執行的驗證要明確記錄原因，不可寫成已通過。

## Definition Of Ready

AI 服務新增 issue 進入 Codex 實作前，至少需要滿足：

- `Service display name`、`Service type`、`Reference AI service`、`Launch target` 已填。
- `pid` 已填；若有事件，`eid` 或事件集合已填。
- API contract 沒有空白；不能確認的項目使用 `Unknown`，並在 engineering review 補 blocker。
- 支援規則已明確標示後端 support flag、SN/FW、帳號限制或不適用原因。
- UI scope 已明確標示 Phone/Pad、服務頁、設定頁與 Dialog 需求。
- 若有事件，event flow、notification 與 deeplink 欄位已填。
- 字串與素材欄位列出已提供項目、placeholder 名稱與缺漏項目。
- engineering review 已確認 final tier、risk、API readiness、phone/pad 與 blockers。
- Codex prompt 已補入 service name、service type、reference service、pid/eid 與 tier。
