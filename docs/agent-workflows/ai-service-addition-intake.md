# AI Service Addition Intake Workflow

本文件定義新增付費 AI 方案時的固定需求入口。目標是讓 PM 只描述新 AI 方案特有的需求差異，工程與 Codex 再依參考服務和本 workflow 推導固定實作規則，避免每次 issue 都重複填寫標準事件鏈路、語言規則、Phone/Pad 檢查或 resource name。

此 workflow 只規範需求收斂與交接，不代表每次都要修改所有 AI 區塊。實作時仍需依實際 issue、參考服務與程式碼盤點縮小範圍。

## Entry Point

PM 應使用 GitHub 的 `AI Service Addition` issue form 建立需求：

```text
.github/ISSUE_TEMPLATE/ai-service-addition.yml
```

issue 建立後，工程需要補完 `Engineering review` 區塊，把 PM 的新需求轉成實作判斷與標準 coverage，再把 `Codex prompt` 區塊貼給 Codex 進入實作對話。

分級判斷請先參考 `docs/agent-workflows/ai-service-addition-tiers.md`。實作後的最低驗證與 report 矩陣請參考 `docs/agent-workflows/ai-service-addition-validation.md`。

## Field Ownership

| 區塊 | 負責角色 | 用途 |
| --- | --- | --- |
| Service display name | PM | 定義對外顯示名稱。 |
| Service type | PM | 指定影像 AI、聲音 AI、混合 AI 或其他服務類型。 |
| Reference AI service | PM / 工程 | 指定最接近的既有 AI，並只列出本次和參考服務不同的地方。 |
| Launch target | PM | 判斷 staging、prod、POC 或限定帳號發布。 |
| PID / EID | PM / 工程 | 提供正式服務 `pid`、事件 `eid` 或事件集合；開發期間暫定值不應列為舊 eid。 |
| New behavior / settings | PM / 工程 | 以使用者視角描述新功能、設定項與預設值；backend 細節由工程 review 補。 |
| Support / pricing differences | PM / 工程 | 只列出和參考服務不同的產品限制、帳號限制、價格、試用或訂閱行為。 |
| UI requirements | PM / 設計 / 工程 | 只列出新畫面、新控制項、排序、特殊 UI 或 Phone/Pad 差異。 |
| Event / push / deeplink | PM / web / 工程 | 只列出此服務特有的推播行為或 Android deeplink；事件 id/name 已在 `Service ID / events` 填寫。 |
| Copy / assets | PM / 設計 / 翻譯 | 列出使用者文案、素材用途與缺漏，不填本機個人路徑；固定翻譯來源由表單預帶。 |
| Acceptance and QA | PM / QA | 只提供 PM 特有驗收畫面、可接受未完成項與 deadline。 |
| Engineering review | 工程 | 確認 final tier、標準 coverage、風險、API/resource 實作細節、缺項與 Codex execution order。 |
| Codex prompt | 工程 | 整理成可直接貼給 Codex 的實作啟動 prompt。 |

## PM Required Information

PM 不需要讀程式碼，但 issue 至少要能回答下列問題：

- 新 AI 服務的對外名稱、服務類型、launch 目標與最接近的既有 AI。
- 本次和參考服務不同的地方；完全相同的價格、試用、事件鏈路、Phone/Pad 行為可寫 `Same as reference`。
- 正式 `pid`、`eid` 或事件集合；如果事件會拆多種，需列出每個 eid 的顯示文字與意義。開發中暫定又被替換的 id 不應當成 old eid。
- 有哪些使用者設定與預設值。PM 不需要填 backend availability、App 內部 endpoint、target、resource name 或 payload 欄位；這些由工程 review 或 Codex 盤點補上。
- 和參考服務不同的支援條件、價格、試用、訂閱、UI 或事件行為。
- 是否有 Android deeplink；不需要另外填 web entry 或 iOS deeplink。
- 字串與素材是否已提供；PM 應描述畫面文字、素材用途與缺漏，不要填本機個人路徑。固定翻譯來源由表單預帶。
- PM 驗收需要看的畫面或流程、可接受未完成項與 release deadline。

如果 PM 缺資訊，工程應在 issue 的 `Blockers / missing PM info` 明確列出，不要讓 Codex 猜測。

## Engineering Review

工程收到 issue 後，先完成下列判斷：

| 欄位 | 判斷規則 |
| --- | --- |
| Final tier | 依 `Service-only`、`Settings-service`、`Full-event-service` 判斷實作與驗證範圍。 |
| Reference implementation | 指定要比照的既有 AI 服務、設定頁、事件、推播或 deeplink。 |
| Risk level | API 未 ready、事件拆分、多語/素材缺漏、Phone/Pad 不對稱或推播/deeplink 皆會提高風險。 |
| Phone/pad required | 明確標 `Phone only`、`Pad only` 或 `Both`。若既有服務 phone/pad 對稱，預設檢查 Both。 |
| Derived standard coverage | 依 final tier 列出由 workflow 自動帶入的固定範圍，例如事件列表、Timeline、filter、export、FCM、語言與素材命名。 |
| API readiness | 後端欄位未定案時，不應讓 Codex 自行定義 API contract；工程需把 PM 的使用者視角需求轉成實作細節。 |
| Event chain required | 只要新增 `eid`，就需檢查事件列表、Timeline、filter、export、FCM 與 All Events。 |
| Push/deeplink required | 明確標示是否包含 FCM 圖片、channel/name 與 web/app deeplink。 |
| Asset/string readiness | 缺正式素材或翻譯時，需列出 placeholder resource name、素材用途與待補語言。 |
| Codex execution order | 先盤點，再實作，再驗證；需要分段時列出順序。 |
| Manual QA owner | 指定誰負責實機、emulator、測試帳號、SN 或 staging 驗收。 |
| Blockers / missing PM info | 缺 pid/eid、API、支援條件、素材、翻譯或測試設備時列在這裡。 |

## Codex Execution Contract

把 issue 的 `Codex prompt` 貼給 Codex 後，Codex 必須先做非修改性的盤點：

1. 讀取 `AGENTS.md` 與本文件。
2. 讀取 `docs/agent-workflows/ai-service-addition-tiers.md` 與 `docs/agent-workflows/ai-service-addition-validation.md`。
3. 依 issue 列出本次會碰到的檔案群，例如 AI service constants、服務頁、智慧 AI 設定、API wrapper、事件列表、Timeline、FCM、deeplink、字串與 drawable。
4. 從 reference service 與 final tier 推導固定規則，例如新增 `eid` 時需檢查事件列表、Timeline、filter、export、FCM、All Events、播放器標題與 deeplink。
5. 標出風險與待確認資訊，尤其是 `Unknown` 後端行為、支援條件、事件拆分、Phone/Pad 不對稱、缺素材與缺翻譯。
6. 將 PM 的使用者視角需求轉成實作細節，例如 endpoint、target、payload、resource name、drawable name 與事件 helper。
7. 確認範圍後再修改檔案。

實作完成後，Codex 需要依 `docs/agent-workflows/ai-service-addition-validation.md` 執行對應驗證。不能執行的驗證要明確記錄原因，不可寫成已通過。

## Definition Of Ready

AI 服務新增 issue 進入 Codex 實作前，至少需要滿足：

- `Service display name`、`Service type`、`Reference AI service`、`Launch target` 已填。
- `pid` 已填；若有事件，`eid` 或事件集合已填。
- 新行為與設定需求沒有空白；不能確認的項目使用 `Unknown`，並在 engineering review 補 blocker。
- 支援、價格、試用、UI、事件與推播若和參考服務不同，已列在對應欄位；相同處可寫 `Same as reference`。
- 若有事件，event id 與事件顯示名稱已填；標準事件鏈路可由 workflow 推導。
- 字串與素材欄位列出已提供文案、翻譯來源 URL、素材用途與缺漏項目。
- engineering review 已確認 final tier、risk、API readiness、phone/pad 與 blockers。
- Codex prompt 已補入 service name、service type、reference service、pid/eid 與 tier。
