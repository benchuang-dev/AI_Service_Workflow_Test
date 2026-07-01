# AI Service Addition Tiers

本文件定義 SpotCam 新增付費 AI 方案的三級 playbook。收到 `AI Service Addition` issue 後，工程應先用本文件判斷級別，再決定 Codex 盤點、實作與驗證範圍。

此 workflow 只規範分級與檢查範圍，不要求每次修改所有列出的功能。實作時仍應依 issue、參考服務與實際搜尋結果縮小範圍。

每個 tier 的最低驗證要求、證據紀錄與 AI 服務新增 change report 矩陣，請接著使用 `docs/agent-workflows/ai-service-addition-validation.md`。

## Tier Summary

| Tier | 適用情境 | 主要風險 | 典型驗證 |
| --- | --- | --- | --- |
| `Service-only` | 只新增服務介紹、訂閱入口、價格或 deeplink，不新增設定 detail 與事件。 | 服務入口排序、價格/試用、支援 camera 篩選或 deeplink 漏補。 | 服務頁 static check、訂閱入口檢查、deeplink 測試、Java compile。 |
| `Settings-service` | 新增 `pid`、智慧 AI 設定頁、detail payload 或自訂 Dialog，但沒有新事件。 | API payload、預設值、Phone/Pad 設定頁、深色模式與儲存流程錯誤。 | 設定頁檢查、API request payload 檢查、Phone/Pad UI、Java compile。 |
| `Full-event-service` | 新增 `pid` 與一個或多個 `eid`，包含事件、Timeline、filter、FCM、export 或 deeplink。 | 事件鏈路多處漏補、推播圖片、Timeline icon、filter 回填、多國語與素材缺漏。 | 完整 debug build、事件鏈路 static search、Phone/Pad filter、FCM/deeplink 測試、change report。 |

## Classification Flow

1. 是否只是新增服務介紹、價格、訂閱或 deeplink？
   - 是：先歸 `Service-only`。
   - 否：繼續判斷是否有設定頁或事件。

2. 是否需要新增智慧 AI 設定頁、detail payload、設定選項或 Dialog？
   - 是：至少是 `Settings-service`。
   - 否：若只有服務入口，維持 `Service-only`。

3. 是否新增 `eid`、事件顯示、Timeline、事件篩選、事件匯出、FCM 或事件圖片？
   - 是：升級為 `Full-event-service`。
   - 否：維持 `Settings-service` 或 `Service-only`。

4. PM 欄位不完整時，不要猜。
   - 在 issue 的 `Blockers / missing PM info` 標記 missing spec。
   - 無法判斷 API、事件或支援條件時，先以較高風險處理，但不要實作臆測行為。

## Tier 1: Service-only

### Definition

新 AI 方案只需要出現在付費服務入口、介紹頁、訂閱或試用流程，不新增智慧 AI detail 設定，也不新增事件鏈路。

常見情境：

- 新方案先做服務頁曝光，後端事件與設定尚未啟用。
- 只新增 web/app deeplink 導到既有服務頁。
- 價格或試用流程沿用既有 AI，且沒有新 `eid`。

### PM Required Fields

- Service display name。
- Service type。
- Reference AI service。
- `pid`。
- 價格與試用規則。
- 支援 camera 篩選規則。
- 服務頁 icon、banner、文案與多國語。
- Phone / Pad 是否都要顯示。
- Deeplink 是否需要。

### Required Discovery

Codex 實作前至少要搜尋：

```powershell
rg -n "<reference-pid>|<reference-service-name>" app/src/main/java app/src/main/res
rg -n "deeplink|mobile_vca|AIService|Vca" app/src/main/java app/src/main/res
```

必查區塊：

- AI service constants 或既有 service id mapping。
- Phone / Pad AI service list 與 service detail page。
- 訂閱 / 免費試用 camera 選擇 dialog。
- Deeplink mapping。
- 字串與 drawable resource。

### Verification

- 服務列表與介紹頁可看到新服務。
- 排序、價格、試用與支援 camera 篩選符合 issue。
- Deeplink 可導到正確頁面，若 issue 有要求。
- Java compile 或 debug build 通過。
- 未實機驗證時，變更摘要需明確列出。

## Tier 2: Settings-service

### Definition

新 AI 方案除了服務入口外，還需要智慧 AI 設定頁、detail payload、設定選項、Dialog 或排程/通知/提示音/發送對象等設定流程，但沒有新增事件鏈路。

常見情境：

- 新 `pid` 需要出現在智慧 AI 設定頁。
- 新服務有 sensitivity、size、zone、mode 等 detail payload。
- 設定 UI 需要比照既有 AI 服務或既有 setting dialog。

### PM Required Fields

除 `Service-only` 欄位外，PM / 工程還必須補齊：

- 設定頁名稱、排序與入口條件。
- 每個設定項的顯示文字、選項、預設值與儲存格式。
- API read/write 欄位、target、payload 範例與空值 fallback。
- Dialog 樣式參考與深色模式要求。
- Phone / Pad 是否都要支援。

### Required Discovery

Codex 實作前至少要搜尋：

```powershell
rg -n "<reference-pid>|SetVca|vca_detail|sensitiveLevel" app/src/main/java app/src/main/res
rg -n "<reference-setting-string>|CustomDialog|InfoDialog" app/src/main/java app/src/main/res
```

必查區塊：

- 智慧 AI 設定頁 Phone / Pad。
- 設定 detail Activity / Fragment。
- API wrapper 與 request payload。
- 共用 Dialog / InfoDialog。
- `values` / `values-night` 顏色與 layout。

### Verification

- 設定頁顯示條件與排序正確。
- 設定項預設值、summary、Dialog 選取與 API payload 正確。
- Phone / Pad UI 對稱，除非 issue 明確排除。
- 深色模式沿用既有規則，不新增硬編碼色。
- Java compile 或 debug build 通過。

## Tier 3: Full-event-service

### Definition

新 AI 方案除了服務與設定外，還會新增事件 `eid` 或事件集合，並影響事件列表、Timeline、事件篩選、All Events、匯出、FCM、推播圖片或 deeplink。

常見情境：

- 新 AI 會產生一種事件。
- 新 AI 事件拆成多個子類型，例如大型動物偵測：猩猩、熊、猴子、野豬。
- 推播要顯示圖片或使用專屬 notification channel/name。
- Web 需要 deeplink 到新 AI 服務頁。

### PM Required Fields

除 `Settings-service` 欄位外，PM / 工程還必須補齊：

- 每個 `eid` 的動物/物件/聲音類型與顯示文字。
- 舊 `eid` 是否要保留相容。
- Timeline `t/m/b` icon 或對應 resource name。
- 事件篩選是否拆成多個項目。
- FCM channel/name、推播圖片與圖片缺失 fallback。
- Deeplink URL 與目的頁。
- 多國語翻譯來源與缺漏語言。

### Required Discovery

Codex 實作前至少要搜尋：

```powershell
rg -n "<reference-eid>|<new-eid>|Event|Timeline|filter|FCM|notification" app/src/main/java app/src/main/res
rg -n "<reference-event-string>|<reference-icon>" app/src/main/res
```

必查區塊：

- Event list adapter。
- Timeline event icon 與首中末連線。
- Phone / Pad event filter dialog。
- All Events 與 event export。
- Player title / event detail title。
- FCM notification image、channel id/name。
- Deeplink mapping。
- 字串與 drawable resource。

### Verification

- 每個 `eid` 都顯示正確文字與 icon。
- Timeline 首筆、中間、末筆 icon 連線正常。
- Phone / Pad 事件篩選可顯示、回填與送出。
- FCM 推播若帶圖片會顯示圖片。
- Deeplink 可導到正確頁面。
- 舊 eid 相容行為符合 issue。
- 完整 debug build 通過，並產出 change report。

## Example: Large Animal Detection

大型動物偵測屬於 `Full-event-service`：

- `pid = 31`
- 多個 `eid`：猩猩、熊、猴子、野豬。
- 有服務頁、訂閱、智慧 AI 設定、detail payload、Phone/Pad UI。
- 有事件列表、Timeline icon、事件篩選、All Events、export、FCM 圖片與 deeplink。
- 有素材、多國語與後端 support flag。

這類需求必須先在 issue 明確列出事件集合、事件文字、Timeline icon、推播圖片、deeplink、缺素材與缺翻譯，工程補完 review 後才能交給 Codex 實作。
