# Google Apps Script 七種發布形式

> 建立日期：2026-05-14
> 目的：給內部團隊一份「Apps Script 不同部署模式」的對照與決策指南，協助判斷新需求該用哪種形式

---

## 為什麼要分清楚這些？

Apps Script 的學習曲線很奇怪 — 入門很簡單（綁試算表寫個函式就跑），但一深入就會遇到「我這個功能需要部署嗎？」「為什麼有些東西要點『新增部署作業』有些不用？」「想分享給朋友用要怎麼做？」這類問題。

整套生態系一共有 **7 種發布形式**，差別不是技術深淺，而是**「誰會呼叫這段 code」**。把這個維度搞清楚，所有問題就迎刃而解。

---

## 核心判斷規則

只有一個問題：

> **這個功能會被試算表以外的東西呼叫嗎？**

| 答案 | 屬於哪一類 |
|---|---|
| **不會**（只有試算表內部使用） | 不需要「部署」，存檔即生效 |
| **會**（外部 URL / 第三方系統 / 公開分享） | 需要部署，產生對外連接窗口 |

7 種形式按這個邏輯分成兩大類：

| 內部使用（不需部署） | 跨平台對外（需要部署） |
|---|---|
| 1. Menu（選單） | 4. Web App |
| 2. Sidebar（側欄）  | 5. Workspace Add-on |
| 3. Modal Dialog（對話框） | 6. API Executable |
| 觸發器（onOpen / onEdit / onFormSubmit） | 7. Library |

---

## 第一類：內部使用（不需部署）

### 1. Menu（試算表選單）

**概念**：在試算表頂端「擴充功能」下方新增自訂選單。最入門、最常用的 UI 形式。

**運作**：
- 在 `onOpen()` 函式裡用 `SpreadsheetApp.getUi().createMenu(...)` 建立
- 試算表打開時自動建好選單，點擊執行對應函式

**範例**（cancan 團購系統）：
```js
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('🛒 整理團購')
    .addItem('📊 開啟即時面板', 'showSidebar')
    .addItem('✨ 全部重新整理', 'rebuildAll')
    .addSeparator()
    .addItem('🔔 安裝即時通知', 'installTriggers')
    .addToUi();
}
```

**適合場景**：
- 觸發批次操作（重整、匯出、寄信）
- 一次性設定（安裝觸發器、設定 API Key）
- 入門首選 — 開發成本最低、最直覺

**不適合**：需要顯示資訊、需要表單輸入、需要持續可見的場景

---

### 2. Sidebar（右側側欄）

**概念**：試算表右側滑出一個 HTML 介面，可以做更豐富的互動 — 表單、按鈕、圖表、即時數據。

**運作**：
- 用 `HtmlService.createHtmlOutputFromFile('Sidebar')` 載入 HTML
- 用 `SpreadsheetApp.getUi().showSidebar(html)` 顯示
- HTML 透過 `google.script.run.someBackendFunction()` 呼叫 Code.gs 函式

**範例**（cancan 即時營運面板）：
```js
function showSidebar() {
  const html = HtmlService.createHtmlOutputFromFile('Sidebar')
    .setTitle('cancan 即時面板');
  SpreadsheetApp.getUi().showSidebar(html);
}

function getOpsPanelData() {
  // 撈受限品項剩餘量、待對帳人數、異常訂單清單
  return { limits, stats, todos };
}
```

```html
<!-- Sidebar.html -->
<script>
  google.script.run
    .withSuccessHandler(renderData)
    .getOpsPanelData();
</script>
```

**適合場景**：
- 持續顯示的儀表板 / 工具列
- 客戶搜尋 + 訂單管理
- 設定面板（取代分散的 Menu items）
- 客服快捷操作（複製 LINE 訊息草稿、群發通知）

**限制**：寬度被 Google 鎖在 **300px**，無法加寬。要更大的面板要改用 Modal Dialog 或 Modeless Dialog。

**不適合**：需要超過 300px 寬度的內容、需要全螢幕顯示

---

### 3. Modal / Modeless Dialog（對話框）

**概念**：浮動視窗。Modal 會擋住 sheet（必須關掉才能繼續操作），Modeless 是浮動但不擋。

**運作**：
```js
function openSettings() {
  const html = HtmlService.createHtmlOutputFromFile('Settings')
    .setWidth(600)
    .setHeight(400);
  SpreadsheetApp.getUi().showModalDialog(html, '設定');
  // 或 .showModelessDialog(html, '設定');
}
```

**適合場景**：
- **Modal**：重要決策（確認刪除、付款驗證）、結構化表單輸入
- **Modeless**：想要的「比 sidebar 更寬」的工具視窗（可設 800px+），但可接受浮動 UI

**對比 Sidebar**：

| 維度 | Sidebar | Modal | Modeless |
|---|---|---|---|
| 寬度 | 鎖 300px | 自由（建議 400-800px） | 自由 |
| 阻擋 sheet 操作 | ❌ 不擋 | ✅ 擋（必須關掉） | ❌ 不擋 |
| 位置 | 固定右側 | 螢幕中央 | 浮動，可拖動 |
| 適合 | 持續工具列 | 一次性決策 / 表單 | 跟 sidebar 類似但要更寬 |

---

### 觸發器（Triggers）— 自動執行

雖然不是 UI 形式，但也屬於「內部使用、不需部署」的範疇，順便提一下：

| 類型 | 範例 | 安裝方式 |
|---|---|---|
| **Simple trigger** | `onOpen`, `onEdit`, `onSelectionChange` | 函式名固定就自動啟用 |
| **Installable trigger** | `onFormSubmit`, time-based (每天/每小時) | 要手動跑 `ScriptApp.newTrigger(...)` 一次 |

**重點**：Installable trigger 需要「安裝」這個動作（跑一次安裝函式），但這跟「部署」是兩件事，不要混淆。

---

## 第二類：跨平台對外（需要部署）

### 4. Web App（公開 URL）

**概念**：把你的 Apps Script 變成一個可以用瀏覽器打開的網址，背後跑你的程式碼。最常用的「對外」形式。

**運作**：
- 寫一個 `doGet(e)` 或 `doPost(e)` 函式回傳 HTML 或 JSON
- 透過「**部署 → 新增部署作業 → 網頁應用程式**」取得 URL
- 任何人（或登入 Google 帳號的人，視設定）打開 URL 都會執行你的 code

**範例**（cancan 出貨單）：
```js
function doGet(e) {
  const rowIdx = parseInt(e.parameter.row);
  const orderData = getOrderByRow_(rowIdx);
  const template = HtmlService.createTemplateFromFile('packing_slip_template');
  template.order = orderData;
  return template.evaluate()
    .setTitle(`${orderData.name}_出貨單_#${rowIdx}`);
}
```

→ URL 變成 `https://script.google.com/macros/s/{id}/exec?row=5` → 開啟瀏覽器就看到第 5 筆訂單的設計版出貨單頁面。

**適合場景**：
- 提供給瀏覽器訪問的網頁（per-row 出貨單、報表頁面）
- 對外提供 webhook 端點（接 form 提交、接第三方推送）
- 想要「無伺服器」做小型網站，後端用 Sheet 當資料庫

**部署注意事項**（坑點）：
- 第一次用「新增部署作業」→ 取得 URL
- 之後修改 code 要更新已部署版本，必須用「**管理部署作業 → 編輯 → 新版本**」
- ⚠️ **不要用「新增部署作業」**（會產生新 URL，所有舊連結會壞掉）
- URL 建議 hardcode 進 CONFIG，不要靠 `ScriptApp.getService().getUrl()` auto-detect（多 deployment 會抓錯）

---

### 5. Workspace Add-on（Marketplace 公開發布）

**概念**：把整套腳本包裝成 Google Workspace Marketplace 上的「擴充功能」，其他人可以在他們自己的 Sheet/Doc/Gmail 裡「安裝」這個 add-on。

**運作**：
- 寫好程式碼後到 [Google Cloud Console](https://console.cloud.google.com) 設定 OAuth scope
- 寫 manifest（`appsscript.json`）描述 add-on 行為
- 提交 Google 審核（隱私政策 + demo 影片 + scope justification）
- 通常 2-4 週審核時間，通過後上架 [Workspace Marketplace](https://workspace.google.com/marketplace)

**範例使用情境**：
- 「我做了一套團購管理工具，想賣給其他餐廳/工作室」→ 上架 Marketplace
- 「我做了 Sheet 翻譯工具，想開放免費下載」→ 同上

**適合場景**：
- 要規模化讓**陌生人**使用（不是你認識的朋友）
- 想做產品化（收費或免費都可以）
- 不希望讓使用者碰 Apps Script 編輯器

**門檻**：
- 審核很嚴格（隱私政策、OAuth scope 申明、合規檢查）
- 客製化彈性低 — CONFIG 區塊沒辦法 hardcode 給單一店家用
- 收費走 Marketplace 內購、或自架 Stripe 訂閱

**判斷**：除非你真的要做產品給陌生人用，否則不需要走這條路。

---

### 6. API Executable（HTTP API）

**概念**：把 Apps Script 的函式當成 HTTP API 暴露給**外部系統**呼叫。

**運作**：
- 部署時選「API Executable」
- 外部系統用 OAuth2 token 打 `https://script.googleapis.com/v1/scripts/{id}:run`
- 帶上 function 名稱 + 參數，會跑你的函式並回傳結果

**範例**：
```bash
curl -X POST \
  -H "Authorization: Bearer ${OAUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"function":"addOrder","parameters":[{"name":"客戶X","total":1500}]}' \
  https://script.googleapis.com/v1/scripts/ABCD1234.../run
```

**典型使用情境**：
- 公司有自架 Node.js / Python 後端，想把資料寫進 Google Sheet 當報表 source
- mobile app 想透過你的 Apps Script 邏輯動 Sheet（懶得自己寫 Sheets API 整合）
- 外部表單工具（Typeform、Tally）→ webhook 進 Apps Script → 客製處理後存進 Sheet
- Slack/Discord chatbot 接到指令 → 呼叫 Apps Script 函式

**特點**：
- 需要 OAuth2 認證流程（比 Web App 複雜）
- 有 quota 限制（單次 6 分鐘執行上限、每日呼叫上限）
- 適合「**程式對程式**」的整合，不適合給人類使用

**vs Web App**：
- Web App = 「給瀏覽器/人類訪問」
- API Executable = 「給其他系統呼叫」

---

### 7. Library（給其他 Apps Script 引用）

**概念**：把你的 Apps Script 變成可被「**其他 Apps Script 專案**」引用的模組。**只在 Apps Script 生態系內**。

**運作**：
- 在 source Apps Script「部署 → 新增部署作業 → 程式庫」取得 Script ID
- 在 consumer Apps Script「**檔案 → 新增程式庫**」貼上 ID + 取個 namespace 名稱
- 之後可以 `MyLib.someFunction()` 呼叫

**範例**：
```js
// In MyUtilLib (the library)
function normalizePhone(raw) {
  return String(raw || '').trim().replace(/^9/, '09');
}
function detectStorageType(name) {
  // ...
}

// In other Apps Script project
function processOrder(row) {
  const phone = MyUtilLib.normalizePhone(row[2]);
  const storage = MyUtilLib.detectStorageType(row[3]);
}
```

**典型使用情境**：
- 你有 5 個團購試算表（不同季別），都需要相同的 `normalizePhone` / `computeOrder` helper → 抽到 library，1 處改動所有專案受益
- 你寫了 Sheet 工具集（時間格式、批次寫入加速、保護管理），多個專案共用 → library 化
- 公司內部的「ENG 共用工具庫」

**特點**：
- 版本化（v1, v2, HEAD）— consumer 可以鎖版本或追最新
- 不對外、只給其他 Apps Script 使用
- Consumer 修改 library 後**要手動升級版本號**

**vs API Executable**：

| 維度 | API Executable | Library |
|---|---|---|
| 誰呼叫 | **外部系統**（任何 HTTP client） | **另一份 Apps Script** |
| 呼叫方式 | REST API + OAuth | `LibraryName.func()` |
| 適合 | **跨平台整合** | **同生態系程式碼共用** |
| 更新方式 | 重新部署 → 外部自動拿到新版 | 發布新版號 → consumer 手動升級 |

---

## 決策樹：我該用哪一個？

```
這個功能會被試算表以外的東西呼叫嗎？
├─ 不會 → 內部使用，不需部署
│   ├─ 需要視覺化介面、表單輸入？
│   │   ├─ 一次性決策 → Modal Dialog
│   │   ├─ 持續工具列、儀表板 → Sidebar
│   │   └─ 需要 >300px 寬度 → Modeless Dialog
│   ├─ 只是觸發 actions（重整、匯出）→ Menu
│   └─ 自動執行 → Triggers (onOpen / onEdit / onFormSubmit)
│
└─ 會 → 對外部署
    ├─ 給瀏覽器訪問（人類使用 URL）→ Web App
    ├─ 給其他系統呼叫（程式對程式 HTTP）→ API Executable
    ├─ 給其他 Apps Script 引用（程式碼共用）→ Library
    └─ 給陌生人在自己的 Google 環境裡安裝使用 → Workspace Add-on
```

---

## 對應一個實際案例：cancan 團購管理系統

cancan 用到的所有形式，可以驗證上面的決策邏輯：

| 元件 | 形式 | 為什麼用這個 |
|---|---|---|
| 「🛒 整理團購」選單 | Menu | 觸發批次重整、安裝觸發器 |
| 「📊 開啟即時面板」 | Sidebar | 持續顯示營運狀況、搜尋客人 |
| 「🤖 設定 Telegram Bot Token」 | Modal Dialog (Prompt) | 一次性敏感資訊輸入 |
| 客人按下 form 送出 | Installable Trigger (`onFormSubmit`) | 自動執行 |
| 同仁填後五碼自動同步 | Simple Trigger (`onEdit`) | 自動執行 |
| 點訂單明細「📄 開啟」打開出貨單 | **Web App** (`doGet`) | 給瀏覽器訪問、客戶不需要 Google 帳號 |

cancan **沒有用到**的形式：
- ❌ Workspace Add-on（沒打算對外發布）
- ❌ API Executable（沒有外部系統要呼叫）
- ❌ Library（只有一個 Apps Script 專案，沒有共用需求）

這也說明了一件事 — **大部分 Apps Script 專案只需要前 4 種**。後 3 種是進階情境才會用到。

---

## 重點摘要

1. **「部署」只給對外用**：所有試算表內部使用的功能（Menu、Sidebar、Dialog、Triggers）都不需要部署，存檔就生效
2. **Web App 是最常見的對外形式**：給人類用瀏覽器訪問
3. **API Executable vs Library**：前者給外部 HTTP 系統用、後者給其他 Apps Script 程式碼用
4. **Workspace Add-on 是規模化才需要**：要陌生人能裝來用才考慮，否則太麻煩
5. **Sidebar 寬度鎖 300px** — 要更寬只能用 Modeless Dialog 或重寫成 Web App

---

*整理自 Apps Script 開發實務與 cancan 團購管理系統實例，2026-05-14*
