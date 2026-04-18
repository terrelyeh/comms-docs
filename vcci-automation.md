# VCCI 日本認證登錄自動化

> 建立日期：2026-04-19
> 目的：記錄 VCCI 日本認證登錄的半自動化流程 — 碰到的技術痛點、採用的架構、以及其他類似「官方申報網站自動化」專案可複用的經驗。

---

## 背景

EnGenius Networks Japan K.K. 在日本販售網通商品，每一款新產品上市前，必須在 **VCCI（電波障害自主規制協議会）** 官網登錄產品符合性報告。流程雖然不複雜，但重複性高（每月 1–2 款）、欄位細碎、且送錯難撤回，是典型「低頻、高單筆成本、易出錯」的工作。

我們把整套流程做成半自動化工具：**PDF 測試報告 → 自動填表 → 人工確認 → 送出 → 歸檔 → Push GitHub**。工具 skill 化後，每次要送新產品只要說「幫我跑 VCCI 登錄」就能完成。

本文記錄過程中的痛點與解法，給其他類似自動化專案參考。

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

---

## 二、架構與技術選型

### 整體架構

```
┌────────────────────────────────────────────┐
│  Claude Code Skill (vcci-submit)           │
│  ─ 串聯整個 pipeline                       │
└──────────────┬─────────────────────────────┘
               │
   ┌───────────┼────────────┬──────────────┐
   ▼           ▼            ▼              ▼
┌──────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ PDF  │→ │  YAML    │→ │Playwright│→ │  XLSX    │
│ 報告 │  │  (草稿)  │  │  自動化  │  │ 追蹤表   │
└──────┘  └──────────┘  └──────────┘  └──────────┘
 pdfplumber   PyYAML      persistent   openpyxl
                          _context
                                            │
                                            ▼
                                    ┌──────────────┐
                                    │ GitHub Private│
                                    │ Repo（存證）  │
                                    └──────────────┘
```

### 技術選型與理由

| 層 | 選擇 | 為什麼 |
|---|---|---|
| Python 環境 | **uv** | 比 pip 快、lockfile 清楚、標準化專案結構 |
| PDF 萃取 | **pdfplumber** | 純 Python、無需外部依賴、文字抽取品質好 |
| 瀏覽器自動化 | **Playwright** | 比 Selenium 新、API 乾淨、內建等待機制 |
| Chromium 啟動模式 | **`launch_persistent_context`** | Cookie 持久化、指紋穩定、避免每次登入 |
| Excel 處理 | **openpyxl** | 純 Python、能讀寫 xlsx、格式控制精準 |
| 歸檔 | **GitHub Private repo** | 版本控管、異地備份、送件稽核可追溯 |
| 編排 | **Claude Code Skill** | 用自然語言觸發整條 pipeline |

### 專案結構

```
VCCI Automation/
├── pyproject.toml              # uv 管理
├── .env                        # VCCI 帳密（gitignore）
├── .playwright-profile/        # Chromium profile（cookie 留這）
├── reports/                    # 原始測試報告 PDF
├── products/                   # 產生的 YAML 草稿（gitignore）
├── archive/                    # 每次送件的存證
│   └── <YYYYMMDD-HHMMSS>-<Model>/
│       ├── 01_form_filled.png + .html
│       ├── 02_confirmation.png + .html
│       ├── 03_success.png + .html
│       ├── 04_top_recent_status.png + .html
│       ├── <Model>.yaml
│       └── <報告>.pdf
├── tracking.xlsx               # 滾動送件紀錄表
└── src/vcci_automation/
    ├── classification.py       # 官方分類代碼表 + 關鍵字判斷
    ├── extract.py              # PDF → YAML
    ├── submit.py               # YAML → Playwright → 送件
    └── tracking.py             # Excel 追蹤表維護
```

---

## 三、工作流程（端到端）

從丟 PDF 到 GitHub 有紀錄，總共走 9 個階段。其中 5 個完全自動、4 個半自動（人工確認點）。

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
| I. Push | `git commit + push` 到 private repo | 🤖 全自動（`--push` flag）|

### 每次送件會產生的存證

每筆送件在 `archive/<timestamp>-<model>/` 都會留下：

| 檔案 | 內容 | 重要性 |
|---|---|---|
| `01_form_filled.png/html` | 自動填完的表單 | 中 |
| `02_confirmation.png/html` | VCCI 確認頁（每個欄位顯示什麼值）| ⭐ 最重要 |
| `03_success.png` | 送出成功頁（"successfully completed"）| 高 |
| `04_top_recent_status.png/html` | Top 頁 Recent reporting status（含 Report No.）| ⭐ 含 Report No. |
| `<Model>.yaml` | 送出時的資料 | 高（可 diff）|
| `<測試報告>.pdf` | 原始 PDF | 高 |

---

## 四、可複用的學習（給其他自動化專案）

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

每次送件都留 4 張截圖 + HTML + YAML + PDF，**總共約 1-2 MB**，但日後稽核（VCCI 回報問題、內部 PM 想知道當時填了什麼）就有得追。用 Git + GitHub private repo 歸檔，是「版本控管 + 異地備份 + 追溯性」一次解決的最便宜方案。

---

## 五、跟 AI Agent 協作的新工作模式

這整個工具從無到有建置約花 2 個工作日，全程用 Claude Code（對話式開發）。觀察到幾個值得推廣的做法：

### ✅ 讓 AI 直接執行、不要只給建議

以前用 ChatGPT 要複製貼上程式碼。現在 Claude Code 可以直接執行 `uv run`、`git commit`、開瀏覽器測試。**迭代速度快 10x**：一個 bug 從發現到修好通常 < 2 分鐘。

### ✅ 用 Skill 封裝重複性任務

`~/.claude/skills/vcci-submit/SKILL.md` 寫好一次，之後每次要送件只要說「幫我跑 VCCI」就會執行完整 pipeline。這等於把「資深員工的操作知識」文件化 + 可執行化。

### ✅ Private repo 作為協作介面

跟 Claude 的對話不會自動變成團隊資產。但：
- 讓 Claude 把完成的工具放 GitHub private repo
- 每次送件自動 commit + push

→ 變成**團隊隨時可以看、可以接手**的實作成果。

### ✅ 人工 checkpoint 就像 code review

半自動的「最終確認」步驟，等同於真人對 AI 做的 code review。保留這個步驟讓我們對工具有信心 — 知道最壞情況人還能擋下來。

---

## 六、延伸應用

這套架構可以搬到其他類似情境：

| 潛在應用 | 類似點 |
|---|---|
| 其他國家/地區的產品認證送件（FCC, CE, KC）| 官方表單自動化 |
| 政府補助/申報系統填表 | 反自動化機制、老舊 CGI |
| 供應商入口網站的訂單輸入 | 多層驗證 + 表單 |
| 內部 ERP 批次操作 | 無 API、只能 UI 自動化 |

核心能力可以直接重用：
- PDF 萃取模組
- Playwright persistent context 架構
- Excel 追蹤表
- Git 歸檔 + skill 編排

---

## 總結

| 收穫 | 成果 |
|---|---|
| 單款產品送件時間 | 從 20-30 分鐘 → **5 分鐘**（含人工確認）|
| 送錯風險 | 低（YAML review + 確認頁 + 截圖存證）|
| 知識資產化 | 公司 VCCI 流程有完整程式碼 + 送件紀錄可查 |
| 可複製性 | 架構可套用到其他法規申報系統 |

更重要的是，這個專案證明了 **AI + 半自動化** 是處理這類「低頻、高精確度需求」業務的好方法 — 不要求 100% 自動，但把 80% 機械性工作交給機器、保留關鍵決策給人。
