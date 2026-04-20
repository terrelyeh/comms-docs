# EnGenius ShortLink 操作指南

> 建立日期：2026-04-20
> 適用對象：行銷團隊、內容編輯、合作夥伴管理者
> 目的：從零上手這個工具，了解怎麼建連結、看成效、團隊協作
> 搭配閱讀：[UTM 參數使用指南](utm-parameters-guide.html) — 想知道 UTM 欄位怎麼填，請先看那份

---

## 工具定位

**一句話**：短網址 + UTM + 點擊分析，三件事揉在一個工具裡，讓行銷團隊不用翻 Bitly + Google UTM Builder + GA 三個地方才能追蹤一條活動連結。

### 跟 Bitly 的差別

- 用自家網域 `go.engenius.ai`（不是 `bit.ly`），對外更可信
- UTM 規範內建 + 白名單強制（避免同事各打各的 `facebook / Facebook / FB`）
- Campaign 是一級公民（不只是 UTM 字串），可以群組、設目標、看時序
- 免費且無連結上限

### 跟 GA4 的差別

- 這個工具看「**點擊前**」— 誰點了連結、從哪點、哪個素材贏
- GA4 看「**點擊後**」— 進站後做了什麼、有沒有轉換

詳細分工定位見 [UTM 參數指南：短網址工具 vs GA 分工](utm-parameters-guide.html#tool-vs-ga)

---

## 5 分鐘上手：建你的第一條連結

### Step 1 — 登入

Google OAuth，用公司 email。如果被擋，代表不在白名單，跟管理員要求加入。

### Step 2 — 左側 Sidebar → **連結管理**

### Step 3 — 右上角 **+ 建立連結**

### Step 4 — 填三個核心欄位

- **目的地網址**：連結要導去哪（landing page / 文章 / 活動頁）
- **標題**：給自己認的名字（例：`Spring 2026 EDM top banner`）
- **UTM 活動 (Campaign)**：下拉選既有活動或打字建新的

剩下 source / medium / content 按模板或下拉選單填（見 [UTM 參數指南](utm-parameters-guide.html)）。

### Step 5 — 儲存 → 自動回到列表 → 複製短網址

列表右邊 `⧉` 一鍵複製 `https://go.engenius.ai/<code>`。

### Step 6 — 貼到你的通路

EDM / FB / IG / LinkedIn / QR 傳單皆可。

### Step 7 — 24 小時後看數據

左側 Sidebar → **數據分析** → 篩選這條連結 → 看點擊來源、裝置、時段。

---

## 連結管理（/links）

列表出所有你建的連結，這頁是最常來的地方。

### 表格欄位

| 欄位 | 說明 |
|---|---|
| 標題 + OG 縮圖 | hover 看完整目的地 URL |
| Campaign | 這條屬於哪一場活動 |
| 媒介 (Medium) | UTM medium 值 |
| 來源 (Source) | UTM source 值 |
| 短網址 | 點 `⧉` 複製 |
| 標籤 | 自由分類 |
| 狀態 | 啟用 / 暫停 / 已封存 |
| 點擊數 | 累積總數 + 7 天 ↑↓% |

### 列尾 ⋮ 選單

- **編輯連結** — 改目的地、UTM、限制條件
- **數據分析** — 跳到該連結的分析頁
- **QR Code** — 下載 PNG / SVG
- **建立副本** — 複製一條同設定的新連結（用 A/B 或跨通路很方便）
- **暫停** — 短網址暫時不能用（點了會導到「連結已停用」頁）
- **刪除** — 永久失效

### 搜尋與篩選

- 搜尋框：打字即時過濾，右邊 X 可清空
- Campaign 下拉：只看某活動的連結
- 狀態 Chip：啟用 / 暫停 / 已封存

### 為什麼要按「同步」按鈕？

這個工具用 client-side cache — 切頁不重新抓 DB，所以極快。代價是**建立 / 修改後要手動按同步**才會看到最新列表。頁首右上角淡藍色「同步」鍵旁邊會顯示「最後同步於 X 分鐘前」。

建立 / 複製 / 暫停 / 刪除等操作已內建強制同步，不用手動按；但如果你另外開一個分頁操作，回來這頁就需要按一下。

---

## 建立連結進階選項

除了基本三欄，展開進階可看到：

### 標籤 (Tags)

自由分類。建議先跟團隊對齊一組核心標籤（例：`常態 / 節慶 / KOL / 展場`）別讓每個人自創。

### UTM 模板

下拉選一個既有模板，source / medium / content 會自動帶入，你只需填 campaign。省重複打字。模板管理見 [UTM 模板](#utm-模板) 一節。

### 自訂短碼 (custom code)

想要 `/spring-2026` 而不是隨機 `/Ab3xK9p`：
- 3–50 字元
- 只能用英文字母、數字、`-`、`_`
- 系統即時查重

不填就會自動產 7 碼隨機碼。

### 排程啟用 (startsAt)

「幾月幾日幾點才開始生效」。之前點會導到「尚未開始」頁。用途：EDM 提前埋好連結，排程當天自動生效。

### 過期時間 (expiresAt)

「幾月幾日後失效」。過期點擊會導到「連結已過期」頁。用途：限時活動、優惠碼。

### 點擊上限 (maxClicks)

超過指定次數後連結失效。用途：限量優惠、內測名額。

### 允許國家 (allowedCountries)

多選列表，只允許這些國家的流量進來。其他國家導到「地區不允許」頁。用途：地區限定活動、版權限制。

### A/B 變體

一條短網址後掛多個目的地 URL 加權重：

```
code: spring-26
  variant A: landing-v1.html  weight: 50
  variant B: landing-v2.html  weight: 50
```

使用者點時加權隨機分流。後台看「哪個 variant 贏」。

### 302 vs 301 轉址

- **302（預設，暫時轉址）** — **行銷連結就用這個**，SEO 不會傳遞
- **301（永久轉址）** — 只在做永久網站搬家時用

---

## Campaign 管理（/campaigns）

### 什麼是 Campaign？

**一場行銷活動** — 例如「2026 春季新品發表」。
一場活動會有多條連結（EDM、FB 廣告、KOL 貼文、QR 傳單…），都指到同一個 `utm_campaign` 值。

### Campaigns 頁做什麼？

**Leaderboard**：所有活動的績效排行
- 點擊數、7 天趨勢、最後活動時間、目標達成率

**勾選 2–4 個活動** → overlay 折線圖比較 / 進入 side-by-side 比較頁

**Orphan Links** 表：有 utm_campaign 值但還沒綁到任何 Campaign 的連結（通常是舊資料）

### 單活動駕駛艙 `/campaigns/[name]`

點活動名稱進入該活動的指揮中心，有 3 個 tab：

- **Overview** — KPI + 30 天趨勢折線
- **Traffic** — 這場活動的 top sources / mediums / 裝置 / 國家
- **Links** — 這場活動底下的所有連結 + 每條的獨立 / 共享比

### Campaign 自動建立

**填 `utm_campaign = "spring-sale"` → 系統自動 upsert 一筆 Campaign row（status=ACTIVE）。**

不用先跑去 Campaigns 頁手動建才能填 UTM。直接建連結，Campaign 會自己出現在列表。

### 目標 (Goal)

可以幫 Campaign 設「目標點擊數」，Leaderboard 的 goal% 欄會顯示達成率。

---

## UTM 模板

### 設計理念：**通路預設**

一個模板 = 某個通路/素材類型的 UTM 預設組合：

| 模板名稱 | source | medium | content |
|---|---|---|---|
| EDM 電子報 | `newsletter_weekly` | `email` | `top_banner` |
| FB 付費廣告 | `facebook` | `cpc` | — |
| 展場 QR | `flyer_computex` | `qr` | — |

**刻意沒有 campaign 欄位** — 因為同一個通路會橫跨多個活動使用，campaign 每次建連結時現填。

### 使用流程

1. 建連結時 → 選模板
2. 系統自動帶入 source / medium / content / term
3. 你只需填 campaign
4. 存檔

### 模板管理

Sidebar → **UTM 模板** → 新增 / 編輯 / 刪除。建議先盤點團隊常用的通路（EDM、FB 廣告、IG 自然貼文、展場 QR…），每個建一個模板。

---

## 數據分析（/analytics）

### 定位

這頁是**全站跨維度分析**，不是活動級排行（那是 Campaigns 頁的事）。

### 主要分析維度

- **時間軸** — 每日/小時的點擊趨勢
- **裝置 / 瀏覽器 / OS** — 流量組成
- **地區 / 城市** — 地理分佈
- **來源 / 媒介 / Referrer** — 流量來源
- **UTM 交叉表** — 例如 source × medium 組合，哪組帶最多量

### 篩選器

- 時段：7d / 14d / 30d / 90d
- 指定單一連結（從連結管理頁 ⋮ → 數據分析可直接帶入）
- 指定單一 Campaign

### 點擊熱度圖

星期 × 小時的熱度分佈，找使用者最常點擊的時段，排 EDM 發送時間或社群貼文時間用。

### 為什麼沒有轉換率 / CVR？

團隊目前不用轉換追蹤 snippet。DB 與 API 都還在，哪天要啟用，把 UI 加回即可。

---

## QR Code

每條連結都能產 QR code：

1. 連結管理 → 該列 ⋮ → **QR Code**
2. 下載 PNG（列印用）或 SVG（向量，印刷用）
3. 中央嵌入 EnGenius 品牌 logo

QR 跟短網址綁定 — 之後改目的地，原本掃出來的人也會跟著去新目的地。

**適用場景**：
- 實體傳單 / DM
- 展場 KV 海報 / 拉頁
- 名片
- 產品包裝導流

---

## CSV 批次匯入

一次建幾十上百條連結（例：展場活動發給各國代理商）。

Sidebar → **連結管理** → 右上角下拉 → **批次匯入** → 上傳 CSV → 預覽 → 確認。

### CSV 欄位格式

```csv
originalUrl,title,utmSource,utmMedium,utmCampaign,utmContent,tags
https://...,Spring EDM Top,newsletter,email,spring-sale,top_banner,EDM
https://...,Spring FB Ad,facebook,cpc,spring-sale,video_30s,FB/付費
```

每列都是獨立 UTM（不共用 template），因為同一批通常是同通路不同素材。

---

## 公開分享報告

想把活動成效報告分享給沒有登入權限的人（主管、合作夥伴、客戶）？

1. 進入 Campaign 詳情頁
2. 右上角 **產生分享連結**
3. 拿到 `https://<app-url>/share/<token>`
4. 這個連結可以給任何人看該活動的 Overview + Traffic + Links，但無法編輯、也看不到其他活動

---

## 團隊 & 權限

### 角色

| 角色 | 權限 |
|---|---|
| **OWNER** | 全權限 |
| **ADMIN** | 可管成員、可刪連結 |
| **MEMBER** | 可建自己的連結、編輯自己的 |
| **VIEWER** | 唯讀 |

### 工作區 (Workspace)

一個工作區 = 一組連結 + 活動 + 成員 + UTM 規範。

大部分團隊只需要一個（例：`HQ MKT`）。如果之後要分區（台灣 / 日本）或跨團隊（行銷 / 客服），再開新工作區。

切換工作區在左上角的選擇器。

---

## UTM 白名單治理

**設定 → 工作區 → UTM Governance**（只有 ADMIN / OWNER 能改）

可以設定：
- **Approved Sources** — 允許的 source 值清單
- **Approved Mediums** — 允許的 medium 值清單

**設定後的效果**：
- 團隊成員建連結時輸入非白名單值 → 即時 amber warning
- 仍然可以送（不強制擋），但 server 端會拒絕明顯 invalid 的輸入

**目的**：避免同一個來源 10 個人寫出 10 種拼法（`facebook` / `Facebook` / `FB` / `fb` / `meta`…）導致 GA 報表無法彙總。

---

## 常見問題

### Q. 短網址會失效嗎？

不會 — 除非你主動刪除、設過期時間、或設點擊上限。

### Q. 我可以換短網址的目的地嗎？

可以。編輯連結 → 改 `originalUrl` → 儲存。原本分享出去的短網址下次點擊會去新目的地。

### Q. 為什麼建完連結、回到列表看不到？

曾經有這個 bug，現在修好了 — 建立 / 複製 / 刪除等操作都會強制同步。
如果偶爾還是遇到，按頁首淡藍色「同步」按鈕。

### Q.「建立副本」跟「複製短網址」差別？

- **建立副本** = 在 DB 新增一條同設定的連結（不同 short code）— 用於 A/B 測試或跨通路重用
- **複製短網址** = 把短網址複製到剪貼簿 — 用於貼到 EDM / 社群

### Q. 我建的連結同事看得到嗎？

同工作區的成員都看得到（協作用）。需要私人連結目前不支援。

### Q. 怎麼撤銷一個已發出去的短網址？

- 暫時撤銷：編輯 → 狀態改「暫停」（點擊會導到「連結已停用」頁）
- 永久撤銷：刪除

### Q. 可以追蹤 landing page 之後的行為嗎？

不行 — 這工具只追「點擊前」。Landing 後用 GA4。見 [UTM 指南的分工定位章節](utm-parameters-guide.html#tool-vs-ga)。

### Q. 輸入非白名單的 source/medium 會怎樣？

會跳 amber warning，但可以送出（除非明顯 invalid 格式被擋）。之後 ADMIN 會被通知、可以把它加入白名單或跟你討論改用別的值。

### Q. 有點擊是 bot / 爬蟲嗎？會影響統計嗎？

系統有基本去重（2 秒內同 IP 同連結算一次），但不做 UA 分類。社群平台的 link preview 爬蟲會產生點擊，這個比例通常小，可忽略。

---

## 支援

### 技術問題 / bug 回報
GitHub Issue：<https://github.com/terrelyeh/shortlink-app/issues>

### 使用教學、規範討論
`#marketing` Slack channel

### 這份文件要補充？
直接改 [`shortlink-tool-guide.md`](https://github.com/terrelyeh/comms-docs) 或開 PR。
