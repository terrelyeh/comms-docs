# VCCI 日本認證登錄自動化

> 建立日期：2026-04-19
> 更新日期：2026-04-19（加入狀態同步 + 線上 Dashboard 章節）
> 目的：記錄 VCCI 日本認證登錄的半自動化流程 — 技術痛點、採用的架構、以及其他類似「官方申報網站自動化 + 生命週期追蹤」專案可複用的經驗。

---

## 背景

EnGenius Networks Japan K.K. 在日本販售網通商品，每一款新產品上市前，必須在 **VCCI（電波障害自主規制協議会）** 官網登錄產品符合性報告。流程雖然不複雜，但重複性高（每月 1–2 款）、欄位細碎、且送錯難撤回，是典型「低頻、高單筆成本、易出錯」的工作。

**而且送出去只是第一步**：VCCI 收到申請後會給一個 **Report No.**，但要等 4–5 個工作日審核完、核發 **Acceptance No.**，產品才能合法貼 VCCI 標、對外販售。送件完還有一段必須追蹤的「等待期」。

我們把整條生命週期做成兩個半自動化工具：

1. **送件**：PDF 測試報告 → 自動填表 → 人工確認 → 送出 → 歸檔 → Push GitHub
2. **狀態同步**：每週自動抓 VCCI 官網最新狀態 → 回填 Acceptance No. → 更新線上 Dashboard → Push GitHub

兩個工具都做成 Claude Code skill，每次要用只要說「幫我跑 VCCI 登錄」或「查 VCCI 狀態」就能完成。本文記錄過程中的痛點與解法，給其他類似自動化專案參考。

---

## 一、自動化碰到的痛點

政府/法規類網站有很多反自動化機制，不像一般 SaaS 那樣 API-first。以下是我們實戰發現的主要障礙：

### 🔒 痛點 1：多層驗證（HTTP Basic Auth + Form Login）

官方網站常有**兩層以上的驗證**：

| 層級 | 機制 | 處理方式 |
|---|---|---|
| 第一層 | HTTP Basic Auth（瀏覽器彈窗） | Playwright 的 `http_credentials` 參數 |
| 第二層 | 網站內的 Login Form（Email + Password） | 填表 + 點擊，但此處通常卡關 |

### 🚫 痛點 2：合成事件 (Synthetic Events) 不被信任

這是**最大的技術障礙**。VCCI 的 CGI 表單會靜默拒絕程式產生的 click 事件：

| Click 方式 | 等級 | VCCI 反應 |
|---|---|---|
| `element.click()`（JS 呼叫）| 合成事件 | ❌ 靜默忽略 |
| `page.locator(...).click()`（Playwright CDP）| 合成事件 | ❌ 靜默忽略 |
| `page.keyboard.press('Enter')` | 合成事件 | ❌ 不穩定（偶爾過）|
| **`page.mouse.click(x, y)`（OS 級滑鼠座標點擊）**| **真實事件** | ✅ **接受** |
| 真人滑鼠點擊 | 真實事件 | ✅ 接受 |

**關鍵發現**：`page.mouse.click()` 產生的是 OS-level input event，VCCI 會當成真人點擊接受。但 Playwright 裡 `locator.click()` 的底層其實也是發這類事件，為什麼它被擋？推測是 Playwright 在 click 時會送額外 metadata（像 `automation` flag），VCCI 檢測得到。

### 🎯 痛點 3：AJAX 聯動需要真實 `blur` 事件

表單的「Test Lab Member No.」欄位填完後，應該要 AJAX 自動帶出公司名。但用 Playwright 的 `fill()`（直接 set value）不會觸發 AJAX — 因為沒發 `blur` 事件。

**解法**：用 `press_sequentially()` 模擬打字，再 `press("Tab")` 觸發 blur：
```python
field.click()
field.press_sequentially(value, delay=30)
field.press("Tab")
page.wait_for_function(
    "document.querySelector('...CompanyName').value.length > 0",
    timeout=5000
)
```

### 📑 痛點 4：雙語 PDF 解析（錯位抓錯值）

TÜV Rheinland 測試報告是**德英雙語並列**，兩個語言的 label 在文字流裡交替出現。如果用英文 label 當 regex anchor，抓到的值會是**下一個欄位**的內容（因為德文值在英文 label 之後才換行）。

**解法**：改用德文 label 當 anchor，值就在同一行：
```python
# ❌ 錯：用英文 label 會抓到下一個欄位的值
r"Identification / Type no\.:\s*(.+)"

# ✅ 對：用德文 label
r"Bezeichnung / Typ-Nr\.:\s*(.+?)\s*$"
```

### 🔀 痛點 5：CGI 表單 — 一個 URL 多個狀態

VCCI 是老式 CGI 系統，**所有頁面都是同一個 URL**（`Manager.pl`），伺服器根據 POST 資料決定回哪個頁面。所以：
- ❌ 不能靠 URL 變化判斷「有沒有進下一頁」
- ✅ 要靠**頁面內容**（出現/消失的按鈕）判斷

**解法**：poll 下一個預期按鈕是否出現：
```python
# 點 Confirm 之後，等 Report 按鈕出現才算成功
page.wait_for_selector(
    "input[type='submit'][value='Report']", timeout=30_000
)
```

### 🤖 痛點 6：自動化特徵被識別（反 bot）

VCCI 會檢測 Playwright 的預設指紋（`navigator.webdriver = true` 等）拒絕登入。

**解法組合**：
1. 用真實 Chrome User-Agent
2. `launch_persistent_context` 用真實 profile（cookie + 指紋穩定）
3. 關鍵：**讓使用者手動登入一次**，之後 cookie 持久化，繞過登入環節的 bot 檢測

### ⏳ 痛點 7：非同步核發 — Report No. ≠ 可販售

這是**商業面**的痛點：VCCI 送件後 **不是立刻就能賣**。

| 欄位 | 核發時機 | 意義 |
|---|---|---|
| **Report No.** | 送件當下 | 只代表「我送出去了」 |
| **Acceptance No.** | 4–5 個工作日後 | 真正能貼 VCCI 標、合法販售 |

如果沒做狀態追蹤，很容易發生「以為送了就能賣 → 實際上還在審核」的商業風險。所以這類認證自動化專案，**追蹤循環和送件本身一樣重要**。

---

## 二、架構與技術選型

### 整體架構（兩個循環）

```
送件循環（每月 1–2 次）                    狀態同步循環（每週 1 次）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━               ━━━━━━━━━━━━━━━━━━━━━━━━━
┌──────┐   ┌──────────┐                    ┌──────────────────────┐
│ PDF  │ → │  YAML    │                    │  VCCI Top 頁         │
│ 報告 │   │  (草稿)  │                    │  Recent Reporting    │
└──────┘   └──────────┘                    │  Status 表           │
               │                            └──────────┬───────────┘
               ▼                                       │
         ┌──────────┐                                  ▼
         │Playwright│                            ┌──────────┐
         │  自動化  │   ◄────── 順手抓 ────────  │  parse + │
         └──────────┘                            │ snapshot │
               │                                 └──────────┘
               ▼                                       │
         ┌──────────┐  ┌──────────┐  ┌──────────┐     ▼
         │  XLSX    │  │  截圖    │  │  歸檔    │  ┌──────────┐
         │ 追蹤表   │  │ 存證     │  │ yaml+pdf │  │  Acceptance
         └──────────┘  └──────────┘  └──────────┘  │  No. 回填│
               │                                    └──────────┘
               ▼                                          │
         ┌────────────────────────────────────────────────┐
         │  靜態 Dashboard (index.html + submissions/)    │
         │  ← 每次送件 / sync 都會重生、git push          │
         └────────────────────────────────────────────────┘
                               │
                               ▼
                    ┌────────────────────┐
                    │  Vercel + 密碼登入 │
                    │  團隊可分享的網址  │
                    └────────────────────┘
```

### 技術選型與理由

| 層 | 選擇 | 為什麼 |
|---|---|---|
| Python 環境 | **uv** | 比 pip 快、lockfile 清楚、標準化專案結構 |
| PDF 萃取 | **pdfplumber** | 純 Python、無需外部依賴、文字抽取品質好 |
| 瀏覽器自動化 | **Playwright** | 比 Selenium 新、API 乾淨、內建等待機制 |
| Chromium 啟動模式 | **`launch_persistent_context`** | Cookie 持久化、指紋穩定、避免每次登入 |
| Excel 處理 | **openpyxl** | 純 Python、能讀寫 xlsx、格式控制精準 |
| Dashboard | **靜態 HTML**（無前端框架）| 送件才重生、讀取快、易部署 |
| 部署 | **Vercel + Edge Middleware** | 自動部署、自訂密碼登入頁、HttpOnly cookie |
| 歸檔 | **GitHub Private repo** | 版本控管、異地備份、送件稽核可追溯 |
| 編排 | **Claude Code Skill × 2** | 送件與狀態同步分開成兩個 skill，職責清楚 |

### 專案結構

```
VCCI Automation/
├── pyproject.toml              # uv 管理
├── .env                        # VCCI 帳密（gitignore）
├── .playwright-profile/        # Chromium profile（cookie 留這）
├── reports/                    # 原始測試報告 PDF
├── products/                   # 產生的 YAML 草稿（gitignore）
├── archive/                    # 每次送件的存證
│   ├── <YYYYMMDD-HHMMSS>-<Model>/    # 每筆送件
│   │   ├── 01_form_filled.png + .html
│   │   ├── 02_confirmation.png + .html
│   │   ├── 03_success.png + .html
│   │   ├── 04_top_recent_status.png + .html
│   │   ├── <Model>.yaml
│   │   └── <報告>.pdf
│   └── status-snapshots/                 # 每次狀態同步的快照
│       └── <YYYYMMDD-HHMMSS>/
│           ├── top_recent_status.png
│           ├── top_recent_status.html
│           └── models.json           # 這張快照涵蓋哪些型號
├── tracking.xlsx               # 滾動送件紀錄表
├── index.html                  # Dashboard 首頁（生成）
├── submissions/                # 每筆送件的 detail 頁（生成）
├── middleware.ts               # Vercel 密碼登入頁 + cookie
├── vercel.json                 # 靜態部署設定
└── src/vcci_automation/
    ├── classification.py       # 官方分類代碼表 + 關鍵字判斷
    ├── extract.py              # PDF → YAML
    ├── submit.py               # YAML → Playwright → 送件 + 順手 sync
    ├── status_sync.py          # Top 頁 parser + 回填 tracking.xlsx
    ├── update.py               # 獨立 vcci-update CLI
    ├── dashboard.py            # archive + xlsx → HTML
    └── tracking.py             # Excel 追蹤表維護
```

---

## 三、送件流程（單次）

從丟 PDF 到 GitHub 有紀錄，共 11 個階段。其中自動佔主體、人工只在「關鍵決策點」介入。

| 階段 | 做什麼 | 自動化程度 |
|---|---|---|
| A. 準備 | 把測試報告 PDF 丟進 `reports/` | 👤 人工 |
| B. 萃取 | `vcci-extract`：PDF → YAML，自動帶分類代碼、N/A 理由 | 🤖 全自動 |
| C. Review | 肉眼掃 YAML 對不對 | 👤 人工 |
| D. 登入 | Playwright 開 Chromium；有 cookie 就跳過、沒有就讓使用者手動登入一次 | 🟡 半自動 |
| E. 填表 | `vcci-submit`：用欄位 `name` 定位填完 13 個欄位，觸發 AJAX | 🤖 全自動 |
| F. Confirm | `page.mouse.click()` 自動按 Confirm；偵測 Report 按鈕出現 | 🤖 全自動 |
| G. 最終確認 | `AskUserQuestion` 問：確認頁資料對嗎？要送嗎？ | 👤 人工 |
| H. Report + 歸檔 | 按 Report → 成功頁 → 去 Top 頁抓 Report No. → 截圖 → tracking.xlsx | 🤖 全自動 |
| **I. Piggyback sync** | **順手把 Top 頁上所有其他 pending 型號的 Acceptance No. 回填** | 🤖 全自動（新增）|
| J. Dashboard 重生 | `vcci-dashboard`：從 tracking.xlsx + archive 重建靜態 HTML | 🤖 全自動 |
| K. Push | `git commit + push` → Vercel 自動部署 | 🤖 全自動（`--push` flag）|

> 🎁 **Bonus — Piggyback sync**：送新品時 Playwright 本來就會去 Top 頁抓自己的 Report No.，順手把**所有其他 pending 型號**的狀態也一併同步。等於「上架一款新品，自動幫你檢查一次所有在審的舊品」，零額外成本。

---

## 四、狀態同步循環（每週）

VCCI 4–5 個工作日才核發 Acceptance No.。這段等待期需要自動追蹤，不能靠人工記得定期登入查狀態。

### 為什麼要獨立成一個 skill

送件 skill 負責「送新的」，狀態同步 skill 負責「看已送的」— 職責分開，Claude 觸發也準確：

| | `vcci-submit` | `vcci-status-sync` |
|---|---|---|
| **觸發語** | 「幫我 VCCI 登錄 ECW230」 | 「查 VCCI 狀態 / 批下來了沒」 |
| **輸入** | PDF + model no. | 無 |
| **動作** | 送件 + 順手 sync + push | 純 sync + dashboard 重生 |
| **頻率** | 每月 1–2 次 | 每週 1 次 |
| **人工介入** | 最終確認送出 | 基本不用（只需確認要不要 push）|

### 同步流程

```bash
uv run vcci-update --push
```

背後執行：

1. **開 Chromium + 持久化 profile** — cookie 還有效就直接進，否則手動登入一次
2. **造訪 VCCI Top 頁** (ME02)，parse Recent reporting status 表格
3. **存 snapshot** 到 `archive/status-snapshots/<timestamp>/`
   - `top_recent_status.png`：完整頁面截圖
   - `top_recent_status.html`：原始 HTML 備份
   - `models.json`：這張快照涵蓋哪些型號（dashboard 要用）
4. **比對 tracking.xlsx**：任何 pending 列，portal 有 Acceptance No. 就回填、Status 改成 `Accepted`
5. **重生 Dashboard**，新狀態立刻反映到每個型號的 detail page
6. **Git commit + push** → Vercel 自動重新部署

### 謹慎原則：status 只寫 terminal 狀態

Portal 的 `in progress` 和 tracking.xlsx 的 `Submitted` 是同義詞，不該互相覆蓋。同步邏輯只在**真正變成 Accepted / Rejected** 時才寫 Status cell，避免每週 sync 都製造 diff noise。

### Snapshot 累積，但頁面長度不變

每次同步都產生一個新的 snapshot 資料夾（完整 audit trail），但 dashboard 每個型號的 detail page **只顯示一張**「最新一張含該型號」的 snapshot：

- 還 pending 的型號 → 永遠指向最新的 snapshot
- 被 Accept 的型號 → 從 Recent 列表消失後，自動 freeze 在消失前最後那張

所以不管累積 5 週還是 50 週的 snapshot，每個 detail page 都只有 1 張 Top 頁截圖，頁面長度恆定。

---

## 五、Dashboard：從 Excel 到線上分享

有了 `tracking.xlsx` 後，內部想知道狀態還得開 Excel — 對非開發同事、外部合作方都不友善。所以做了一個簡單的線上 Dashboard。

### 功能

- **首頁**：所有送件的 rolling table（Model / Report No. / Acceptance No. / 狀態 / 送件日）、客戶端搜尋、統計 KPI
- **每個型號的 detail page**：狀態 banner（Ready to Ship / Pending / Rejected）、送件資訊、**PDF 萃取出來用於填表的 YAML 資料**、4 張送件截圖（01~04，04 會自動更新到最新 sync 的 Top 頁 snapshot）、原始 PDF + YAML 下載
- **密碼保護**：自訂 HTML 登入頁（純密碼、無 username 欄），cookie 記 30 天
- **部署**：Vercel 自動部署，每次 push 後 ~1 分鐘生效

### 生命週期視角：每個型號的「履歷表」

每個型號的 detail page 從送件當下開始累積：

1. **送件當天**：看得到完整表單截圖、確認頁、成功頁、送件當下的 Top 頁狀態
2. **週期 sync 期間**：04 截圖會自動更新到最新 Top 頁 snapshot，看得到目前狀態（in progress / Acceptance No. 已發）
3. **Accept 後**：04 截圖 freeze 在「Accept 前最後一張快照」，整頁變成永久 audit 紀錄

這是把「靜態送件紀錄」升級成「產品的認證履歷表」。

### 技術細節

- **生成**：`uv run vcci-dashboard`，讀 `tracking.xlsx` + `archive/`，產出 `index.html` + `submissions/*.html` + `assets/style.css`
- **登入**：`middleware.ts` 是 Vercel Edge Middleware，攔截所有 request；POST `/__auth` 驗密碼後 Set-Cookie
- **密碼**：存於 Vercel env var `DASHBOARD_PASSWORD`，設定時要用 `printf`（不能用 `echo`，會多塞 `\n`）

---

## 六、可複用的學習（給其他自動化專案）

### 🎓 教訓 1：先確認「真人怎麼做」，再想「怎麼自動化」

剛開始我以為 form login 可以直接 `page.fill()` + `page.click()`。搞了 2 小時才發現 VCCI 根本不接受合成 click。**一定要先用 Claude in Chrome 或 devtools 手動走一次流程、錄下真實 HTTP request，再設計自動化。**

### 🎓 教訓 2：`page.mouse.click()` 是 Playwright 的隱藏武器

大多數教學只講 `locator.click()`。但當遇到「合成事件被擋」的老舊 CGI 時，`page.mouse.click(x, y)` 才是真正產生 OS 級事件的方法：

```python
btn = page.locator("input[type='submit'][value='Confirm']")
btn.scroll_into_view_if_needed()
box = btn.bounding_box()
page.mouse.move(box["x"] + box["width"]/2, box["y"] + box["height"]/2)
page.mouse.down()
page.mouse.up()
```

### 🎓 教訓 3：Cookie 持久化比「每次登入」更穩

`launch_persistent_context` + 固定 `user_data_dir` 讓 cookie 跨執行保留。好處：
- 只要手動登入一次（繞過登入環節的 bot 偵測）
- 後續 run 全自動、不用輸入帳密
- Profile 有真實瀏覽指紋，比 fresh context 更不像機器人

### 🎓 教訓 4：半自動 > 全自動（對高成本、低頻的操作）

**全自動化不是終點，正確的操作才是。** 這類「送錯不能撤回」的場景，保留 1–2 個人工 checkpoint 反而降低整體風險：

| 決策 | 自動 or 人工 | 理由 |
|---|---|---|
| 填表 | 🤖 自動 | 重複、機械、值來自 PDF |
| 最終送出前 | 👤 人工 | 錯了要重新申請，成本高 |
| Git push | 👤 人工 | 送錯的紀錄不要推上去 |

### 🎓 教訓 5：存證 > 快速

每次送件都留 4 張截圖 + HTML + YAML + PDF，**總共約 1–2 MB**，但日後稽核（VCCI 回報問題、內部 PM 想知道當時填了什麼）就有得追。用 Git + GitHub private repo 歸檔，是「版本控管 + 異地備份 + 追溯性」一次解決的最便宜方案。

### 🎓 教訓 6：生命週期 > 單次操作

第一版只做送件，結果送完後不知道哪款已 Accept、哪款還在審、什麼時候能賣。**補上「狀態同步循環」才完整**。對任何「送出 ≠ 生效」的申報系統，永遠要再問：
- 送完後怎麼知道狀態？
- 狀態改變了我要不要做什麼？
- 有沒有辦法把這個週期也自動化？

把「單次操作」擴大到「生命週期追蹤」，工具的價值會翻倍。

### 🎓 教訓 7：Piggyback — 零成本的附加動作

需要的資料「正好在你已經打開的頁面上」時，**順手做是最划算的**：送件流程本來就會去 Top 頁抓 Report No.，這張頁面就有所有 pending 送件的狀態。多一個 parse 呼叫，零額外 HTTP request，就等於每次送新品都順便幫所有舊品做一次狀態同步。設計任何 workflow 時都可以問一次：「我這趟還在這裡，還能順手抓什麼？」

---

## 七、跟 AI Agent 協作的新工作模式

這整套工具（送件 + 狀態同步 + dashboard）從無到有建置約花 **3 個工作日**，全程用 Claude Code（對話式開發）。觀察到幾個值得推廣的做法：

### ✅ 讓 AI 直接執行、不要只給建議

以前用 ChatGPT 要複製貼上程式碼。現在 Claude Code 可以直接執行 `uv run`、`git commit`、開瀏覽器測試。**迭代速度快 10x**：一個 bug 從發現到修好通常 < 2 分鐘。

### ✅ 用 Skill 封裝重複性任務

`~/.claude/skills/vcci-submit/SKILL.md` 和 `~/.claude/skills/vcci-status-sync/SKILL.md` 寫好一次，之後每次要送件只要說「幫我跑 VCCI」、要查狀態只要說「查 VCCI 狀態」就會執行完整 pipeline。這等於把「資深員工的操作知識」文件化 + 可執行化。

### ✅ 一個專案、多個 skill（職責分離）

送件跟查狀態是**不同使用情境**，混成一個 skill 會讓 Claude 誤判。拆成兩個 skill，共用同一個 Python 專案的底層工具（兩邊都用 `status_sync.py`），各自的 SKILL.md 寫清楚觸發條件。

### ✅ Private repo 作為協作介面

跟 Claude 的對話不會自動變成團隊資產。但：
- 讓 Claude 把完成的工具放 GitHub private repo
- 每次送件 / sync 自動 commit + push
- 生成的 Dashboard 經 Vercel 上線、密碼分享給團隊

→ 變成**團隊隨時可以看、可以接手**的實作成果。

### ✅ 人工 checkpoint 就像 code review

半自動的「最終確認」步驟，等同於真人對 AI 做的 code review。保留這個步驟讓我們對工具有信心 — 知道最壞情況人還能擋下來。

---

## 八、延伸應用

這套架構可以搬到其他類似情境：

| 潛在應用 | 類似點 |
|---|---|
| 其他國家/地區的產品認證送件（FCC, CE, KC）| 官方表單自動化 + 審核週期追蹤 |
| 政府補助/申報系統填表 | 反自動化機制、老舊 CGI、送件後要等通過 |
| 供應商入口網站的訂單輸入 | 多層驗證 + 表單 + 訂單狀態追蹤 |
| 內部 ERP 批次操作 | 無 API、只能 UI 自動化 |
| 商標 / 專利送件追蹤 | 送件 + 長週期審核 + Dashboard 可視化 |

核心能力可以直接重用：
- PDF 萃取模組
- Playwright persistent context 架構
- Excel 追蹤表 + 靜態 Dashboard
- 狀態同步循環 + snapshot 機制
- Git 歸檔 + skill 編排

---

## 總結

| 收穫 | 成果 |
|---|---|
| 單款產品送件時間 | 從 20–30 分鐘 → **5 分鐘**（含人工確認）|
| 狀態追蹤 | 從「每週自己登入查」→ **每週一鍵同步，自動更新 dashboard** |
| 送錯風險 | 低（YAML review + 確認頁 + 截圖存證）|
| 團隊可見度 | Excel 自用 → **密碼保護的線上 Dashboard，團隊任何人都能查** |
| 知識資產化 | 公司 VCCI 流程有完整程式碼 + 送件紀錄 + 每週狀態快照 |
| 可複製性 | 架構可套用到其他法規申報系統 |

更重要的是，這個專案證明了 **AI + 半自動化 + 生命週期視角** 是處理這類「低頻、長週期、高精確度需求」業務的好方法 — 不要求 100% 自動，但把 80% 機械性工作交給機器、保留關鍵決策給人，並且**把「送件」擴大到「追蹤到結案」的完整循環**。
