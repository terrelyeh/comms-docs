# UTM 參數使用指南

> 建立日期：2026-04-18
> 適用對象：行銷團隊、內容編輯、合作夥伴管理者
> 目的：統一 UTM 的填寫規範，讓 GA4 / 分析工具能精確歸因每一筆流量

---

## 什麼是 UTM？為什麼要用？

UTM（Urchin Tracking Module）是加在 URL 後面的**追蹤參數**，讓 GA4、HubSpot 等分析工具能精確辨識：**這個訪客是從哪來的？哪個活動帶來的？什麼素材吸引到他的？**

```
https://engenius.ai/?utm_source=google&utm_medium=cpc&utm_campaign=spring_sale_2026
                   └────────────────── UTM 參數 ──────────────────┘
```

- 沒有 UTM：GA 只會告訴你「有人來自 Google」
- 有 UTM：能知道「來自 Google 付費廣告的『春季特賣』活動」

---

## 五大欄位逐個說明

### 1. 媒介 (utm_medium) — 必填，先選

**定義**：流量的「**類型**」— 用什麼**渠道**帶來的？

| 值 | 使用情境 | 範例 |
|---|---|---|
| `cpc` | 付費廣告（per-click 計費） | Google Ads、Bing Ads |
| `social` | 社群自然貼文（免費的 post） | FB/IG/LinkedIn 粉專發文 |
| `organic` | 自然搜尋（SEO） | Google 搜尋結果 |
| `email` | Email 行銷 | EDM、電子報、交易通知信 |
| `referral` | 其他網站的推薦連結 | 合作夥伴網站、媒體報導 |
| `affiliate` | 聯盟行銷 | 分潤合作、推薦碼 |
| `display` | 展示型廣告 | Banner 廣告、Google Display Network |
| `video` | 影片廣告 | YouTube Ads、TikTok Ads |
| `qr` | QR Code 掃描 | 實體傳單、名片、產品包裝 |
| `offline` | 線下行銷 | 印刷品、展覽、活動 |
| `influencer` | KOL / 網紅合作 | Instagram 業配、YouTuber 開箱 |

**為什麼要先選 medium？** 因為 medium 決定了 source 的範圍。選了 `social`，source 下拉會是 Facebook、IG…；選了 `cpc`，會是 Google Ads、Bing Ads…

---

### 2. 來源 (utm_source) — 必填

**定義**：具體的「**平台或網站名稱**」— 誰帶你來的？

| Medium | Source 範例 |
|---|---|
| `cpc` | `google`, `bing`, `meta`（Meta Ads） |
| `social` | `facebook`, `instagram`, `linkedin`, `twitter`, `threads` |
| `organic` | `google`, `bing`, `yahoo`, `duckduckgo` |
| `email` | `newsletter_weekly`, `edm_2026q2`, `hubspot_nurture` |
| `referral` | 對方網站名（例：`techcrunch`, `partner_xyz`） |
| `qr` | 實體位置（例：`flyer_taipei`, `booth_computex`） |
| `influencer` | KOL 帳號名（例：`ig_userabc`, `yt_channelxyz`） |

---

### 3. 活動 (utm_campaign) — 必填

**定義**：這個連結屬於**哪一場行銷活動**？

**最重要的欄位。** GA 報表會以 campaign 為單位彙總。

**命名規則（強制要求）：**

- 全小寫
- 用底線 `_` 分隔
- 格式建議：`[主題]_[時期]_[對象]`
- 不能用空格、中文、大小寫混用

| 好範例 | 壞範例 | 原因 |
|---|---|---|
| `spring_sale_2026` | `Spring Sale!` | 大寫 + 空格 + 符號 |
| `product_launch_eap_q2` | `春季特賣` | GA 會亂碼 |
| `webinar_2026_jun` | `webinar` | 太籠統、無法區分不同場 |
| `computex_2026` | `展覽` | 分類太抽象 |

---

### 4. 內容 (utm_content) — 選填但強烈建議用

**定義**：同一個活動內，**區分不同素材或版位**

**用途：**

- **A/B 測試**：比較哪個素材效果好
- **多版位追蹤**：首頁 banner vs 側邊欄 vs EDM 按鈕

**範例：**

```
campaign: spring_sale_2026
├─ content: banner_top       ← 網頁頂部 banner
├─ content: banner_sidebar   ← 側邊欄
├─ content: cta_red          ← 紅色 CTA 按鈕
├─ content: cta_blue         ← 藍色 CTA 按鈕（A/B 對照）
└─ content: video_30s        ← 30 秒版影片
```

**新人常犯錯**：以為 content 沒用。其實這是判斷「哪個素材 ROI 最高」的關鍵。

---

### 5. 關鍵字 (utm_term) — 選填，通常不用手動填

**定義**：主要用於**付費搜尋廣告**記錄使用者搜尋的關鍵字

- Google Ads 會**自動帶入**，不用手動設（用 `{keyword}` 動態變數）
- 只有手動建非 Google Ads 的付費搜尋連結才會用到
- 一般社群、Email、QR 都不用填

---

## 實際範例（可直接套用）

### 情境 1：EDM 電子報

```
medium:   email
source:   newsletter_weekly
campaign: product_update_2026_q2
content:  top_banner
```

### 情境 2：Facebook 付費廣告

```
medium:   cpc
source:   facebook
campaign: spring_sale_2026
content:  video_30s_version_a   ← 另一版叫 version_b，方便 A/B 比較
```

### 情境 3：QR Code 印在展場傳單

```
medium:   qr
source:   flyer_computex_2026
campaign: booth_demo_2026
content:  discount_20_off
```

### 情境 4：合作網紅 Instagram 限動

```
medium:   influencer
source:   ig_userxyz
campaign: launch_new_product_q2
content:  story_swipeup
```

### 情境 5：LinkedIn 自然貼文

```
medium:   social
source:   linkedin
campaign: thought_leadership_2026
content:  article_feature_ai
```

---

## 新人常見 7 大錯誤

1. **每次取不同活動名**：`spring_sale` / `Spring-Sales` / `springSale` → GA 會當成 3 個活動，無法彙總
2. **用中文**：`春季特賣` → GA 顯示 `%E6%98%A5%E5%AD%A3...` 亂碼
3. **把活動名塞到 source**：`source=spring_sale_fb` → 應拆成 `source=facebook`, `campaign=spring_sale`
4. **medium 亂填或自創**：有人寫 `fb`、有人寫 `facebook`、有人寫 `social` → 無法跨活動比較
5. **空白欄位**：campaign 空著，GA 就歸類成 `(not set)`，等於白追蹤
6. **只做一次就改名**：活動進行中改 campaign 名 → 前後數據分裂
7. **不用 content**：失去 A/B 對比能力

---

## 團隊最佳實務

### 1. 建立公司 UTM Cheatsheet（已內建在短網址工具）

工具的 **Medium / Source 下拉選單**就是幫你避免拼字不一致。用下拉選單優先，少手動打字。

### 2. Campaign 命名規範

- 先定好格式：`{主題}_{時期}_{對象}`
- 範例：`launch_2026q2_emea`（2026 Q2 歐洲區新品發表）
- 放共用文件讓大家照抄

### 3. 善用工具的「Campaign 資產」功能

短網址工具的 Campaign Model 能統一管理：

- 活動生命週期（草稿 / 進行中 / 結束）
- 活動標籤分類
- 一次設定，多連結共用 → 不用每次都自己打 campaign 名

### 4. UTM Template（模板）

把常用組合存成 template：

- `EDM 電子報` → 自動帶 medium=email, source=newsletter_weekly
- `展場 QR` → 自動帶 medium=qr

### 5. 每月 UTM Audit

- 看 GA 有沒有出現 typo（`spring_sal` 之類）
- 有 typo 就回溯修正 + 提醒同事

---

## 快速檢核表（建連結前跑一遍）

- [ ] medium 有用標準值（cpc / social / organic / email / referral / affiliate / display / video / qr / offline / influencer）
- [ ] source 是具體平台名，不是活動名
- [ ] campaign 是「主題_時期_對象」格式，全小寫 + 底線
- [ ] 同一活動所有連結共用同一個 campaign 名
- [ ] 有做 A/B 或多版位 → 有填 content
- [ ] 沒用中文、沒用空格、沒用 `!@#` 特殊符號

---

## 附錄：Medium 對照小卡

用白話理解每個 medium 在問什麼：

| Medium | 問自己 |
|---|---|
| `cpc` | 這是花錢買的點擊嗎？ |
| `social` | 這是社群的自然發文嗎？ |
| `organic` | 這是搜尋引擎自然搜到的嗎？ |
| `email` | 這是從信裡點過來的嗎？ |
| `referral` | 這是別的網站掛我們連結過來的嗎？ |
| `affiliate` | 這是有分潤合約的夥伴嗎？ |
| `display` | 這是 banner 廣告嗎？ |
| `video` | 這是影片廣告嗎？ |
| `qr` | 這是掃 QR Code 進來的嗎？ |
| `offline` | 這是實體活動或印刷品嗎？ |
| `influencer` | 這是 KOL 業配嗎？ |

---

**有問題或規範需要更新？** 在 #marketing channel 提出，或直接改這份文件。
