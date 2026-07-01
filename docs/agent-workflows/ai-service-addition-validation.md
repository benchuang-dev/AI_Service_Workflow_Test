# AI Service Addition Validation Workflow

本文件定義 SpotCam 新增付費 AI 方案後的固定驗證與報告規則。目標是避免只有 build pass，卻漏掉 Phone/Pad、後端設定行為、API payload、事件列表、Timeline、推播圖片、deeplink、素材或多國語。

本文件只規範驗證與報告，不代表每次 AI 方案都需要修改所有列出的功能。實際驗證範圍應依 `docs/agent-workflows/ai-service-addition-tiers.md` 的 final tier、issue 內容、參考服務與實際 diff 決定。

## Validation Principles

- 驗證必須跟 tier 對齊；`Full-event-service` 不能只用 `Service-only` 的驗證結案。
- Phone / Pad 預設要對稱檢查；若 issue 指定單端支援，report 要明確寫出。
- `pid` 是服務/方案 id，`eid` 是事件 id；兩者即使數字相同也不能混用。
- 後端行為或 API 未 ready 時，不用測試結果補洞；應在 blocker / 風險列出缺少資訊。
- 未跑的驗證要寫 `未執行` 與原因，不可寫成已通過。
- 缺正式素材或翻譯時，要列出素材用途、placeholder resource name 與缺少語言。

## Tier 1: Service-only Validation

適用於只新增服務介紹、訂閱入口、價格、試用或 deeplink，不新增設定 detail 與事件的 AI 方案。

必要驗證：

- 搜尋新 `pid` 與參考 `pid`，確認命中差異合理。
- 檢查 Phone / Pad 服務列表與服務介紹頁。
- 檢查價格、免費試用與訂閱 camera 選擇支援規則。
- 檢查 deeplink，若 issue 有要求。
- 檢查字串與 icon / banner 是否存在。
- 至少執行 Java compile；若無法執行，要寫明環境原因。

建議命令：

```powershell
rg -n "NEW_PID|REFERENCE_PID|NEW_SERVICE_KEY|REFERENCE_SERVICE_KEY" app\src\main\java app\src\main\res
rg -n "mobile_vca|deeplink|AIService|Vca" app\src\main\java app\src\main\res
.\gradlew.bat :app:compileNormalProdDebugJavaWithJavac
```

證據紀錄：

- 新 `pid` 與參考 `pid` 的搜尋命中數，或差異原因。
- Phone / Pad 服務入口是否涵蓋。
- 訂閱 / 免費試用支援規則。
- Deeplink 測試結果。
- Gradle command 與結果。

## Tier 2: Settings-service Validation

包含 `Service-only` 全部驗證，另加智慧 AI 設定頁與 detail payload 驗證。

必要驗證：

- 智慧 AI 設定頁在符合條件時顯示服務。
- 設定 detail 頁標題、排序與可見性正確。
- 設定選項的預設值、summary、Dialog 樣式與儲存行為正確。
- API write payload 與工程 review 補上的實作規格一致。
- 缺值或空值 fallback 與 issue 一致。
- Phone / Pad 都檢查，除非 issue 明確排除。
- 深色模式沿用既有色票與 selector。
- 執行 debug build 或 Java compile。

建議命令：

```powershell
rg -n "NEW_PID|REFERENCE_PID|vca_detail|SetVca|CustomDialog|InfoDialog" app\src\main\java app\src\main\res
rg -n "NEW_SETTING_STRING|REFERENCE_SETTING_STRING" app\src\main\res
.\gradlew.bat :app:assembleNormalProdDebug
```

證據紀錄：

- 設定頁顯示條件。
- 每個設定項的預設值與 payload。
- Phone / Pad 檢查結果。
- 深色模式是否檢查。
- Gradle command 與結果。
- 未手測項目與原因。

## Tier 3: Full-event-service Validation

包含 `Settings-service` 全部驗證，另加事件鏈路、推播與 deeplink 驗證。

必要驗證：

- 搜尋新 `eid` 與參考 `eid`，確認每個事件鏈路都有 mapping。
- 事件列表、All Events、event export 與播放器標題顯示正確。
- Timeline 每個事件使用正確 icon，首筆 / 中間 / 末筆連線正常。
- Phone / Pad 事件篩選 dialog 可顯示、回填與送出。
- FCM notification channel/name 正確。
- FCM 若帶圖片，推播會顯示圖片。
- Deeplink URL 可導到正確頁面。
- 舊 eid 相容或不相容行為符合 issue。
- 多國語與素材缺口已列在 report。
- 執行完整 debug build，並依需要安裝到手機 / 平板驗證。

建議命令：

```powershell
rg -n "NEW_EID|REFERENCE_EID|Event|Timeline|filter|notification|deeplink" app\src\main\java app\src\main\res
rg -n "NEW_EVENT_STRING|REFERENCE_EVENT_STRING|NEW_EVENT_ICON|REFERENCE_EVENT_ICON" app\src\main\res
.\gradlew.bat :app:assembleNormalProdDebug
```

證據紀錄：

- 每個 `eid` 的文字、icon、filter 與推播 mapping。
- Timeline 首中末 icon 驗證結果。
- Phone / Pad filter 驗證結果。
- FCM 圖片顯示驗證結果或未執行原因。
- Deeplink adb 或 web 測試結果。
- 舊 eid 相容行為。

## AI Service Report Matrix

AI 服務新增 change report 應加入下列矩陣。結果欄位建議使用 `通過`、`未執行`、`不適用`、`待補資訊` 或 `有風險`，並在備註寫實際證據。

```markdown
## AI 服務分類

| 欄位 | 內容 |
| --- | --- |
| Tier | Service-only / Settings-service / Full-event-service |
| Service name |  |
| Service type | Video AI / Sound AI / Mixed AI |
| pid |  |
| eid |  |
| Reference service |  |
| Phone support | Supported / Not supported / Not applicable |
| Pad support | Supported / Not supported / Not applicable |
| Backend/API readiness | Ready / Not ready / Unknown |

## AI 服務驗證矩陣

| 項目 | 結果 | 備註 |
| --- | --- | --- |
| Service entry |  |  |
| Subscription / trial |  |  |
| Support rule |  |  |
| Setting page |  |  |
| Backend behavior / API payload |  |  |
| Event list |  |  |
| Timeline |  |  |
| Event filter |  |  |
| FCM notification |  |  |
| Deeplink |  |  |
| Strings |  |  |
| Assets |  |  |
| Phone QA |  |  |
| Pad QA |  |  |
```

矩陣可以依實際 AI 方案能力增減列，但不應刪掉已修改或 issue 明確要求的項目。

## Report Requirements

AI 服務新增完成後，如需要產出報告，report 必須包含：

- AI 服務名稱、服務類型、`pid` 與 `eid`。
- 分級：`Service-only` / `Settings-service` / `Full-event-service`。
- 參考 AI 服務與比照依據。
- 改動摘要。
- 後端行為、API contract 與支援規則。
- 改動檔案表。
- 驗證命令與實際結果。
- 手測項目與結果。
- 字串與素材狀態。
- 風險、未完成事項與 missing PM info。

Report 不得捏造：

- 未跑過的 Gradle command。
- 未執行的實機、emulator、push 或 deeplink 驗證。
- PM 未提供的後端行為、API contract、支援條件或價格。
- 尚未確認的正式產品名稱、圖示、字串或 firmware 行為。

## Codex Completion Checklist

AI 服務新增實作完成前，Codex 應確認：

- final tier 是否仍與實作後範圍一致；若修改中觸及事件、FCM 或 deeplink，應升級驗證等級。
- 已用搜尋對照新服務與參考服務的主要命中點。
- 已依 tier 跑必要 Gradle command，或清楚記錄無法執行原因。
- Phone / Pad、服務頁、設定頁、事件、推播、deeplink、字串與素材都有狀態。
- report 的測試結果只記錄實際執行內容。
