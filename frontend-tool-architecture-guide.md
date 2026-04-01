# 小工具架構選型指南

> 建立日期：2026-04-01
> 目的：幫助開發者在面對不同資料來源與需求時，快速選擇最適合的前端架構、部署平台與資料儲存方案。

---

## 一、三種常見資料來源情境

| # | 情境 | 說明 |
|---|------|------|
| 1 | **自己上傳資料** | 手動匯出 CSV/XLSX，前處理後嵌入工具顯示 |
| 2 | **串接第三方 API** | 例如 Mailchimp、HubSpot，需要 API Key |
| 3 | **Google Sheet 當資料來源** | 用 Google Sheets API 抓取資料，分公開與私人兩種 |

---

## 二、三種框架選擇

### 靜態網頁（Static HTML）
- 單一 HTML 檔案，所有邏輯內嵌
- 零 build step，部署就是上傳檔案
- 適合資料量小、單人維護、不需要模組化的工具

### 純前端 SPA（Vite + React）
- 元件化架構，狀態由 React 管理
- 需要 `vite build` 打包後部署
- 適合互動複雜、多人協作、需要拆分元件的工具

### Next.js（全端框架）
- 支援 API Routes（後端邏輯）
- 可以在伺服器端藏住 API Key
- 適合需要保護機密憑證、或有 SSR/動態路由需求的工具

---

## 三、核心決策問題

架構選型只需要回答兩個問題：

**① API Key 需要保密嗎？**
**② 資料需要跨用戶共享或持久儲存嗎？**

---

## 四、決策樹

```
有 API Key 需要保密？
├── 否 → 純前端（靜態 HTML 或 Vite + React）
└── 是 → 需要後端（Next.js / Vercel Serverless Function）
          │
          資料需要持久儲存或跨用戶？
          ├── 否 → 只要後端 Function，不需要 DB
          └── 是 → 加 Supabase（資料庫）
```

---

## 五、資料來源 × 框架對應表

| 情境 | 框架 | 需要後端？ | 需要 DB？ |
|------|------|-----------|---------|
| 自己上傳資料 | 靜態 HTML | ❌ | ❌ |
| 串接第三方 API（Key 需保密） | Next.js | ✅ API Route | ❌ |
| Google Sheet 公開可讀 | Vite + React | ❌ | ❌ |
| Google Sheet 私人（需 OAuth） | Next.js | ✅ API Route | ❌ |

---

## 六、什麼時候需要 Supabase（資料庫）

| 需求 | 需要 DB？ |
|------|---------|
| 只有自己或固定人員看，資料自己管理 | ❌ |
| 抓 API 資料即時顯示，不需留存 | ❌ |
| 多個用戶各自有帳號和資料 | ✅ |
| 需要記錄歷史、累積用戶行為 | ✅ |
| 用戶可以寫入（留言、表單送出） | ✅ |
| 不同用戶看到不同內容 | ✅ |

**核心原則：有「寫入」或「多用戶」的需求才需要 DB。只是「讀取 + 顯示」通常不需要。**

---

## 七、部署平台選擇

| 平台 | 適合搭配 | 特點 |
|------|---------|------|
| **Cloudflare Pages** | 靜態網頁、Vite SPA | 免費、全球 CDN、無流量上限 |
| **Vercel** | Next.js、Vite SPA | Next.js 原生支援、Serverless Function 整合簡單 |
| **GitHub Pages** | 靜態網頁 | 最簡單，但功能少 |

> Vercel 的強項是 Next.js + Serverless Function。純靜態網頁用 Cloudflare Pages 更合適。

---

## 八、Git-based CMS 模式

當靜態網頁需要「可更新的資料」時，可以搭配 Git-based CMS 模式：

```
Admin UI 操作資料
      ↓
透過 GitHub API（Personal Access Token）
      ↓
寫入 commit 到 repo
      ↓
Cloudflare Pages 偵測變更自動部署
      ↓
網站更新（約 30 秒）
```

**優點：**
- 零後端成本
- 每次更新都有 git commit，天然版本控制
- 可回滾

**適用條件：** 資料更新頻率低（每月/每週），非即時更新需求。

市場上同類產品：Netlify CMS、Tina CMS、Forestry。

---

## 九、資料內嵌 vs 獨立 data.js

### 現在（資料內嵌）
```html
<!-- index.html -->
<script>
  const SEED_DATA = [ /* 所有資料 */ ];
  // 所有 UI 邏輯...
</script>
```

### 更乾淨的做法（拆分 data.js）
```js
// data.js（只放資料）
const SEED_DATA = [ /* 所有資料 */ ];
```

```html
<!-- index.html（只放邏輯）-->
<script src="data.js"></script>
<script>
  // UI 邏輯，直接使用 SEED_DATA
</script>
```

**結論：** 概念完全相同，但關注點分離後更易維護。等到單一檔案開始變得難以瀏覽時再做即可，不必提前優化。

---

## 十、常見誤解

**Q：給多人看是不是就需要資料庫？**
不需要。「多人看」只是讀取，只要這些人看到的是同一份資料，靜態網頁完全夠用。需要 DB 的是「多人寫入」或「每個人看到不同資料」的情境。

**Q：一定要用 React/Vue 才算正規？**
不是。框架是為了解決複雜度與團隊協作問題。單人維護、互動簡單的工具，手寫 JS 反而維護成本更低。

**Q：Vercel 比 Cloudflare Pages 更好？**
不是絕對的。Vercel 的優勢是 Next.js 生態系，純靜態網頁用 Cloudflare Pages 更合適（無流量限制、速度也快）。
