# VCCI 工具網頁化部署方案評估

> 建立日期：2026-04-24
> 文件目的：讓 EnGenius 內部團隊理解「為什麼要把 VCCI 自動化工具搬上網、要往哪個方向走、同事未來會怎麼使用」

---

## TL;DR

- **問題**：VCCI 自動化工具目前只能在 Terrel 的 Mac 上跑，其他同事無法自助送件
- **方案比較**：Google Form 收件頁 / **Mac mini 自架 Web**（中選）/ 完整 SaaS（殺雞用牛刀）
- **決定**：**8 月 Mac mini 到貨後部署 FastAPI Web 服務**，同事走瀏覽器就能送件
- **雙層存取**：Viewer 繼續用 Vercel 線上 dashboard（外網可看）；Submitter 走公司內網 / Tailscale 連 Mac mini
- **Admin 介入頻率**：從目前「每件要等 Terrel」→ 之後「每週 1–3 次 VCCI session 過期時遠端登一次」
- **時程**：設計已鎖定；Mac mini 到貨後 7 個工時可上線

---

## 我們要解決什麼問題

VCCI（日本電波障害自主規制協議會）申請是每一款賣到日本的網通產品必經流程。目前流程：

- Terrel 本機跑 `vcci-extract` + `vcci-submit` CLI 工具
- 其他同事只能看 dashboard 狀態，無法自己送件
- 業務問「XX 可以出日本了嗎？」時，要翻 dashboard 或問 Terrel
- 新產品要送件時，要把 Test Report PDF 傳給 Terrel，等他空檔處理

**痛點**：Terrel 是單點瓶頸，turnaround 不固定；同事無法直接控制送件節奏。

目標：**讓每位有需要的同事都能自助完成送件**，Terrel 只在系統需要維護時介入。

---

## 三個方案的比較

| | 🟢 A：PDF 收件頁 | 🟡 B：Mac mini 自架 | 🔴 C：完整 SaaS |
|---|---|---|---|
| **內容** | 同事上傳 PDF → Terrel 本機跑工具 → 回報結果 | Mac mini 常駐 server，同事透過瀏覽器走完全流程 | 完整產品：PostgreSQL + 多租戶 + RBAC |
| **開發時間** | 1 天 | 1 週（7 工時）| 1–2 人月 |
| **同事自助度** | 30%（還是要等 Terrel）| 95%（只有 session 過期要 admin）| 100% |
| **Terrel 介入** | 每件 30 分鐘 | 每週 5 分鐘 × 1–3 次 | 幾乎不用 |
| **月成本** | $0 | $0（現有 Mac mini）| $50+ |
| **硬體需求** | 無 | Mac mini（8 月到貨）| Cloud |
| **適合規模** | 5 人以下 | 5–20 人 | 20+ 人 / 外部客戶 |

---

## 為什麼選 Mac mini（方案 B）

### 選它的三個理由

1. **macOS 對 Playwright 最友善** — 自動化工具需要 Chromium，macOS 原生支援，不用像 Linux 一樣裝 xvfb/display server
2. **和開發環境一致** — Terrel 的 MacBook 和 Mac mini 同為 macOS，除錯成本最低
3. **VCCI portal 擋自動登入，需要人介入** — Mac mini 接螢幕可讓 admin 隨時遠端接手登入，雲端 VPS 做不到這件事

### 要承認的限制

1. **單點失敗**：Mac mini 掛掉 → 工具全斷。但 Terrel 本機原本就能跑，是完整 fallback
2. **辦公室事件**：停電、斷網就離線。辦公室有 UPS / 備援網路最佳
3. **VCCI session 仍需人介入**：Mac mini 沒消除這件事，只是讓 admin **統一登一次**服務多人，而不是每個同事各自登

**實際評估：Mac mini 缺點可控，益處大於成本 — 是這個問題規模下的最佳解。**

---

## 雙層存取架構

分成兩個不同的服務：

```
╔══════════ 公開 Internet ═══════════╗
║                                    ║
║   Viewer（同事）                    ║
║   · 手機 / 家裡 / 出差都能看         ║
║   · Google 登入                     ║
║   · 只能讀 dashboard                ║
║                                    ║
║   ↓                                ║
║   https://vcci-engenius-jp.vercel.app/ ║
║   （現有網站，不需改）                ║
╚════════════════════════════════════╝
                 │
                 ▼
         GitHub repo（資料來源）
                 ▲
                 │
╔═════ 公司內網 / Tailscale 私網 ════╗
║                                    ║
║   Submitter（有送件權限的同事）      ║
║   · 辦公室、或遠端 Tailscale         ║
║   · 可上傳 PDF、送件                 ║
║                                    ║
║   ↓                                ║
║   http://mini.office:8000          ║
║   FastAPI Web（新，跑在 Mac mini）   ║
║                                    ║
║   Playwright + VCCI portal cookie   ║
║   （admin 登一次駐留）               ║
╚════════════════════════════════════╝
```

### 為什麼切兩層

- **Viewer 的需求**：「隨時隨地看狀態」 → 外網、要方便
- **Submitter 的需求**：「能寫資料、能送件」 → 內網、要安全
- **Mac mini 上有 VCCI 帳密 cookie** → 不適合對公開 Internet 開放
- **Vercel dashboard 只讀** → 外網公開無風險

---

## 同事會怎麼使用：5 個真實情境

### 🎬 情境 1：日本 PM 有新產品要送

上午 9:30 收到 Test Report PDF：
1. 打開 `http://mini.office:8000`，Google 帳號登入
2. 點「新しい届出を追加」，上傳 PDF
3. 系統 10 秒自動抽出 YAML 資料
4. 畫面顯示可編輯欄位（class、code、distance 等）；PDF 抽出的測試機關資料唯讀
5. 確認後按「送出 VCCI」→ 系統跑 Playwright 填表
6. 畫面顯示 VCCI Review 頁截圖 → PM 肉眼檢查 → 按「確認送出」
7. 9:36 收到送件完成 email（Report No. 已記錄到系統）

**總時間：4 分鐘，Terrel 完全沒介入**

### 🎬 情境 2：業務秒查「XX 可以出日本嗎？」

- 客戶詢問 → 業務打開 Vercel dashboard → 搜尋型號
- 看 Status badge（🟢 綠色 = 可出貨 / 🟡 黃色 = 審核中 / 🔴 紅色 = 被退）
- 20 秒搞定，不用找 Terrel、不用翻 Excel

### 🎬 情境 3：週五早上批次檢查審核狀態

- Admin 打開 dashboard 首頁，點「🔄 Sync VCCI Status」
- 系統 30 秒內抓回所有新核發的 Acceptance No.
- 通過的型號自動變綠燈；有核發的會自動寄 email 通知

### 🎬 情境 4：VCCI session 過期 → Admin 遠端登入

- 同事按送出 → 系統偵測到要重登，顯示「session 已過期，admin 已被通知」
- Terrel 手機收到 email：「有同事在等送件，請遠端登入 Mac mini」
- Terrel 從任何地方（捷運、家裡）用 MacBook + Tailscale 遠端 Screen Sharing 到 Mac mini
- 5 分鐘內登入完成、服務恢復、同事的送件繼續跑

### 🎬 情境 5：一天要送 5 件

- 同事一次上傳 5 份 PDF
- 系統排成 queue，逐件顯示「排隊中 / 送件中 / 等 review」
- 每件到 review 頁時發通知，同事依序確認 5 次
- 總耗時約 10–15 分鐘（之前本機流程也是如此）

---

## 7 個設計決策（2026-04-24 鎖定）

| # | 主題 | 決定 | 一句話原因 |
|---|---|---|---|
| D1 | Google 登入機制 | 用 **Authlib** 做 OAuth | 輕量，不需要完整 user 管理 |
| D2 | 同事能改 YAML 嗎 | 限定幾個安全欄位可改，其他唯讀 | 擋 90% 誤用事故 |
| D3 | 多人同時送件 | Server queue + 即時顯示「排第幾位」 | VCCI 一次只能跑一件，排隊比卡住體驗好 |
| D4 | Session 過期通知 | 寄 email 給 admin（以後可加 Slack）| SMTP 已有，零成本 |
| D5 | 誰能用這個系統 | Viewer 外網開放 / Submitter 內網限定 | 權限分級，降低攻擊面 |
| D6 | Git 衝突 | 送件前 pull --rebase，失敗 retry 3 次 | 日常狀況 99% 不會撞 |
| D7 | 審計紀錄 | 每個送件 git commit 自動標示「誰送的」 | 未來查稽核 `git log` 即可 |

**所有細節、實作指引、代碼片段** 在工程文件 `docs/web-deployment-options.md`（VCCI automation repo）。

---

## 時程與下一步

### 現況

- ✅ 三方案評估完成
- ✅ 7 個設計決策鎖定
- ✅ 使用者流程走過（5 個情境）
- ✅ Phase 分階段拆解（0.5 / 1 / 2）
- ⏳ **Mac mini 8 月到貨**

### 未來行程

| 時機 | 動作 | 工時 |
|---|---|---|
| **2026-05 到 2026-07（軟體空窗期）** | 本機開發 Phase 0.5 原型（FastAPI + 基本上傳流程）— 跑在 Terrel MacBook 驗證 | 2–3 天 |
| **2026-08 Mac mini 到貨** | 搬現成程式到 Mac mini、設 Tailscale、Phase 1 上線 | 1 週 |
| **2026-09 起** | Phase 2 功能擴充（queue / 即時狀態頁 / queue 管理後台） | 視需要 |

### 不急著做 deployment guide 的原因

Mac mini 硬體還沒到，指令類文件（`brew install xxx`、specific IP 設定、launchctl plist）3 個月後版本會 rot。**等 Mac mini 實際到手、驗證過的版本才寫最可靠**。

---

## FAQ

### Q：我什麼時候開始可以用？

Mac mini 8 月到貨後約 1 週。在此之前維持現狀（跟 Terrel 講 → 等）。

### Q：會需要我自己裝什麼軟體嗎？

- **Viewer（只想看狀態）**：不用裝任何東西，繼續用 Vercel dashboard
- **Submitter（要送件）**：
  - 辦公室用：什麼都不用裝，直接打開瀏覽器
  - 外出 / 在家用：裝 Tailscale（工程部統一協助）

### Q：如果我是同事，我能直接看 VCCI 帳密嗎？

**不能也不應該**。VCCI 帳密只存在 Mac mini 的 cookie 裡，不會給任何個人帳號。admin（Terrel）統一登入、服務多人。這是設計上的安全隔離。

### Q：送錯件怎麼辦？

- **送出前**：review 頁有 VCCI 官網截圖，同事要自己肉眼確認（這是安全把關）
- **送出後**：VCCI 送件一旦 Acceptance No. 核發就無法撤回。若送錯資料要改，要在 VCCI portal 做「変更届」，是獨立流程

### Q：會不會發生很多人同時想送件打架？

不會。系統有 queue，第 2 位會看到「你排第 #2，預計 5 分鐘」的排隊顯示。實際上一天同時送件的機率很低（每天送件數不多）。

### Q：session 過期率高嗎？

根據過去 2 週的觀察，VCCI 伺服器 session 很短 — 幾乎每次跑工具都要重登。但搬到 Mac mini 後，**admin 登一次可服務多人**，實際介入頻率估計每週 1–3 次。

### Q：Mac mini 掛了怎麼辦？

Terrel 本機原本就能跑工具 — 是 complete fallback。Mac mini 修復期間回到「請 Terrel 代送」模式。

---

## 相關連結

- Dashboard：https://vcci-engenius-jp.vercel.app/
- 工程技術文件：VCCI automation repo 的 `docs/web-deployment-options.md`
- 本文件作者：Terrel Yeh（有任何問題 / 建議歡迎直接討論）
