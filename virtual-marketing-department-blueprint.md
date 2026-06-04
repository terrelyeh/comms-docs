# 虛擬行銷部門 — AI Agent 參考架構藍圖

> **建立日期**:2026-06-03(修正版)
> **對象**:AI FDE / 為客戶建置 AI agent 架構的人
> **目的**:一套可重複部署、再依客戶客製的「虛擬行銷部門」架構。標準化底盤 + 用 skills 客製。
> **底盤**:Paperclip(治理)+ Hermes(員工)+ MCP / Skills(能力)

---

## 設計原則:照 workflow 拆,不照人類職稱拆

AI agent 的編制不該複製人類組織圖。用三個準則切分,編制自然乾淨:

- **一段連貫的 workflow**(輸入 → 產出是一條完整流程)
- **一組共用的工具面**(MCP / skills 集中)
- **一種一致的審批風險**(例如「會花錢」要獨立成一個審批邊界)

> 依此原則,舊版的「Digital Marketing」被**拆解** —— 它本來就是涵蓋 SEO/付費/email 的傘,跟旗下項目重疊。它真正獨有的價值(量測與追蹤治理)獨立成 Analytics & MarTech Ops,其餘回歸各 owner。

---

## 核心心法:架構統一,客製只動三個旋鈕

1. **Skills** — 把客戶流程、品牌語氣、SOP、定位封裝成 skills(每客戶一個私有 repo / Hermes tap)。FDE 的核心交付物與 IP。
2. **MCP 接線** — 接客戶既有工具棧(CRM、分析、廣告後台)。
3. **組織設定** — Paperclip 的 org chart、各職位 persona、預算上限、審批關卡。

---

## 三層架構總覽

```
Paperclip(治理層:org chart / 預算上限 / 審批 / 稽核)
   ├── 策略 & 產品行銷   Hermes + 定位 skills(大腦)
   ├── Organic Growth   Hermes + SEO/內容 skills + Ahrefs/Search Console MCP
   ├── Paid / Demand Gen Hermes + 廣告 skills(預算上限由 Paperclip 鎖)
   ├── Lifecycle / CRM   Hermes + email/自動化 skills + HubSpot/Klaviyo MCP
   ├── Creative Studio   Hermes + 文案/視覺/影片 skills + Canva/Remotion MCP
   └── Analytics & Ops   Hermes + 報表 skills + GA/Amplitude/Supermetrics MCP
            ▲
   行銷總監(human):最終審核、人類把關
```

| 層 | 角色 | 工具 |
|---|---|---|
| 治理層 | 主管 / HR / 財務 | Paperclip |
| 員工層 | 各職能 agent | Hermes(常駐、模型不限、Skills 系統) |
| 能力層 | 各職能專業 | 客製 skills + 客戶 MCP + Claude World Studio MCP |

---

## 6 職能編制(同層、最小重疊)

### 1. 策略 & 產品行銷 — 大腦 + 品牌單一事實來源
| 欄位 | 內容 |
|---|---|
| 職責 | 定位、訊息架構、競品/市場情報、GTM 策略、**目標拆解與派 brief** |
| Skills | `message-guide-*` pipeline、`competitive-brief`、`message-guide-review` |
| MCP | Web 研究、Similarweb、文件(Drive/Notion) |
| 輸入 → 輸出 | 產品 brief / HQ guide / 競品 → 定位卡、Message Guide、季度策略、各職能 brief |
| 審批點 | 對外訊息與策略定稿需人類核可 |
| 備註 | 下游全部吃這份定位;同時是拆解目標、派工的大腦 |

### 2. Organic Growth — owned / earned 一條龍
| 欄位 | 內容 |
|---|---|
| 職責 | SEO(關鍵字、技術巡檢、on-page)+ 內容(部落格/案例/電子報內容)+ 自然社群 |
| Skills | `seo-audit`、`draft-content`、`eg-content-repurpose`、`internal-comms`、自建 keyword-research |
| MCP | Ahrefs、Similarweb、Search Console、Claude World Studio(trend-pulse/cf-browser)、發布工具 |
| 輸入 → 輸出 | 定位 + 關鍵字機會 → 內容行事曆、SEO 優化內容、自然流量成長 |
| 審批點 | 對外發布、網站結構改動需核可 |
| 排程 | 每週 SEO 巡檢 + 選題(Hermes cron) |
| 備註 | SEO 與 Content 合併,避免「內容行事曆誰擁有」打架 |

### 3. Paid / Demand Gen — 最高風險職能 🚨
| 欄位 | 內容 |
|---|---|
| 職責 | 付費搜尋/社群、受眾、出價策略**建議**、A/B、需求開發 campaign |
| Skills | `campaign-plan`、自建 ad-copy / audience-research |
| MCP | Supermetrics、Meta / Google Ads(**唯讀報表**) |
| 輸入 → 輸出 | 目標 / 預算上限 / 受眾 → 廣告文案、受眾建議、出價建議、成效分析 |
| 審批點 | 🚨 **實際投放 / 動用預算一律人類執行**;agent 只給建議。Paperclip 鎖月預算上限 |

### 4. Lifecycle / CRM — owned audience 經營
| 欄位 | 內容 |
|---|---|
| 職責 | email、nurture 流程、行銷自動化、名單分群、留存 / 再行銷 |
| Skills | `email-sequence`、`campaign-plan`(lifecycle 版)、自建 segmentation |
| MCP | HubSpot、Klaviyo、客戶 CRM |
| 輸入 → 輸出 | 名單 + 旅程目標 → nurture 序列、自動化流程、留存 campaign、名單健康報告 |
| 審批點 | 對外寄送(EDM)前需核可 |

### 5. Creative Studio — 服務全部門的產能
| 欄位 | 內容 |
|---|---|
| 職責 | 文案、視覺 / 設計、影片、品牌素材(供應所有職能) |
| Skills | `youtube-script`、`storyboard-generator`、`remotion`、`draft-content`(copy) |
| MCP | nanobanana(生圖)、Canva、Remotion(render)、NotebookLM、Figma |
| 輸入 → 輸出 | 各職能 creative brief + 品牌素材 → 文案、圖、影片、成片 |
| 審批點 | 對外發布前需核可 |
| 備註 | 共用產能,吃同一份品牌 skill,避免風格分裂 |

### 6. Analytics & MarTech Ops — 量測層(AI 最擅長)
| 欄位 | 內容 |
|---|---|
| 職責 | 數據彙整、儀表板、歸因分析、UTM / 追蹤治理、MarTech 串接健康 |
| Skills | `performance-report`、`dashboard-builder`、UTM 指南、ShortLink |
| MCP | Amplitude、Supermetrics、GA、HubSpot(讀) |
| 輸入 → 輸出 | 各渠道數據 → 統一儀表板、歸因報告、UTM 規範、洞察回饋給策略 |
| 審批點 | 多為唯讀分析;追蹤規範變更需協調 |
| 備註 | 把成效回饋給策略,形成閉環 |

---

## 職責邊界 / RACI

**核心心法**:#5 Creative Studio 是「內部設計外包(產能)」;#2/#3/#4 是「渠道老闆」。老闆決定 what/when/why、負責成效與發布;Studio 只把素材做漂亮。

| 活動 | 策略 | Organic | Paid | Life | Creative | Ops | 人類 |
|---|---|---|---|---|---|---|---|
| 定位 / 訊息架構 | R | I | I | I | I | C | **A** |
| 內容行事曆 / 選題 | C | **A** | – | – | I | I | – |
| 社群貼文(文字) | – | **A** | – | – | – | – | – |
| 社群素材(圖/影片) | – | **A** | – | – | R | – | – |
| SEO 技術 / on-page | – | **A** | – | – | – | C | – |
| 付費廣告(受眾/文案/出價建議) | C | – | **A** | – | C | I | – |
| 廣告投放 / 動用預算 🚨 | – | – | R | – | – | I | **A** |
| Email / nurture(分群·發送) | – | – | – | **A** | C | I | – |
| 素材設計(文案/視覺/影片) | – | C | C | C | **A** | – | – |
| UTM / 追蹤規範 | I | C | C | C | – | **A** | – |
| 成效報表 / 歸因 | I | I | I | I | – | **A** | – |
| 對外發布審批 | – | R | R | R | – | – | **A** |

> **R** 執行 · **A** 當責(拍板,每列僅一個)· **C** 諮詢 · **I** 告知 · – 不涉入。**人類拿到 A 的列 = 審批關卡**(定位定稿、花錢、對外發布)。

**Handoff(寫進各 skill,避免搶事/互等):**
- 渠道 owner(#2/#3/#4):需要素材時產 creative brief 交給 Creative Studio,收回後**由我發布**。
- Creative Studio(#5):**只交付素材,不對外發布**;發布權在渠道 owner。

---

## 依客戶規模縮放(6 不是魔法數字)

| 客戶規模 | agent 數 | 編制 |
|---|---|---|
| 小型 / SMB | **3** | 策略 / Growth(organic+paid 合)/ Creative+Ops 合 |
| 中型 | **6** | 上面六職能 |
| 大型 / Enterprise | **8–10** | 再拆出 Social/Community、Localization、CRO、PR |

FDE 賣法:**baseline 給 6,Discovery 後依客戶縮放**(小客戶合併、大客戶展開)。

---

## 跨職能協作流程

```
策略 & 產品行銷(大腦:定位 + 目標拆解 + 派 brief)
        │  定位卡 / Message Guide(品牌單一事實來源)
   ┌────┼─────────────────┬───────────────┐
   ▼    ▼                 ▼               ▼
Organic Growth      Paid/Demand Gen   Lifecycle/CRM
(SEO+內容+社群)      (付費·預算)        (email·自動化·留存)
   └────────┬──────────────┴───────────────┘
            │  共用 creative brief
            ▼
     Creative Studio(文案 / 視覺 / 影片 → 供應全部門)
            │
            ▼
   Analytics & MarTech Ops(統一量測 · 歸因 · UTM 治理)
            │  成效回饋
            ▼
     策略 & 產品行銷(迭代定位)◀── 閉環
```

---

## 治理層(Paperclip)與紅線

- **org chart**:6 職能 + 回報給「行銷總監(human)」
- **每個 agent 設月預算上限**(Paid / Demand Gen 尤其重要,超支自動暫停)
- **審批關卡**:對外發布、廣告花費、EDM 寄送、網站結構改動
- **immutable audit log**:所有決策與 tool call 可追溯
- **品牌單一事實來源**:全部產出吃同一份 message guide + `.impeccable`

### ⚠️ 幫別人建,特別要顧的紅線
1. **花錢 / 投放絕不自動化** — 一律走人類審批,合約寫清責任歸屬。
2. **資料與密鑰隔離** — 每客戶獨立部署,各自 API key 走 secrets 管理。
3. **成熟度風險** — 工具都還新,先 pilot 一個職能再擴成整個部門。
4. **品牌一致性** — 同一份品牌 skill,避免多 agent 風格分裂。

---

## 導入路線(分階段)

| 階段 | 做什麼 |
|---|---|
| **Phase 1 · Pilot** | 挑 1 個高價值職能(Organic Growth 或 Analytics/Ops),跑出成效 |
| **Phase 2 · 擴編** | 加到 3–4 個常駐職能,串起協作流程 |
| **Phase 3 · 治理** | 疊 Paperclip:org chart、預算上限、審批、稽核,交付客戶 GUI |

---

## FDE 可重複資產(Starter Kit)

- 標準 org 模板(6 職能 + 預算 / 審批預設)
- Baseline skill pack(6 職能通用 skill,客戶版覆寫)
- MCP 接線 playbook
- Discovery 問卷 + 交接文件模板
