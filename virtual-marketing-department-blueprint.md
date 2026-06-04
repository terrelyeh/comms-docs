# 虛擬行銷部門 — AI Agent 參考架構藍圖

> **建立日期**:2026-06-03
> **對象**:AI FDE / 為客戶建置 AI agent 架構的人
> **目的**:一套可重複部署、再依客戶客製的「虛擬行銷部門」架構。標準化底盤 + 用 skills 客製。
> **底盤**:Paperclip(治理)+ Hermes(員工)+ MCP / Skills(能力)

---

## 核心心法:架構統一,客製只動三個旋鈕

每進一家客戶**不重寫系統**,只調三樣:

1. **Skills** — 把客戶流程、品牌語氣、SOP、定位封裝成 skills(每客戶一個私有 repo / Hermes tap,可版本控管、遠端更新)。這是 FDE 的核心交付物與 IP。
2. **MCP 接線** — 接客戶既有工具棧(CRM、分析、廣告後台)。
3. **組織設定** — Paperclip 的 org chart、各職位 persona、預算上限、審批關卡。

---

## 三層架構總覽

```
Paperclip(治理層:org chart / 預算上限 / 審批 / 稽核)
   ├── 產品行銷   Hermes + 定位 skills
   ├── SEO        Hermes + SEO skills + Ahrefs/Search Console MCP
   ├── Digital    Hermes + campaign skills + HubSpot/GA MCP
   ├── Content    Hermes + 品牌 skills + 內容 MCP
   ├── 廣告專員    Hermes + 廣告 skills(預算上限由 Paperclip 鎖)
   └── 影片製作    Hermes + Remotion/NotebookLM MCP
            ▲
   行銷總監(human + orchestrator):派工、審核、人類把關
```

| 層 | 角色 | 工具 |
|---|---|---|
| 治理層 | 主管 / HR / 財務 | Paperclip |
| 員工層 | 各職位 agent | Hermes(常駐、模型不限、有 Skills 系統) |
| 能力層 | 各職位專業 | 客製 skills + 客戶 MCP + Claude World Studio MCP |

---

## 6 職位編制表

### 1. 產品行銷(Product Marketing)— 品牌單一事實來源
| 欄位 | 內容 |
|---|---|
| 職責 | 定位、訊息架構、競品分析、價值主張、launch messaging |
| Skills | `message-guide-*` pipeline、`competitive-brief`、`message-guide-review` |
| MCP | Web 研究、Similarweb、文件(Drive/Notion) |
| 輸入 → 輸出 | 產品 brief / HQ guide / 競品 → 定位卡、Message Guide、Sales enablement |
| 審批點 | 對外訊息定稿需人類核可 |
| 備註 | **下游所有職位都吃這份定位** — 品牌一致性的源頭 |

### 2. SEO
| 欄位 | 內容 |
|---|---|
| 職責 | 關鍵字研究、技術 SEO 巡檢、內容 gap、排名追蹤、on-page 建議 |
| Skills | `seo-audit` + 自建 keyword-research / serp-monitor |
| MCP | Ahrefs、Similarweb、Google Search Console(客戶接) |
| 輸入 → 輸出 | 網站 URL / 目標關鍵字 / 競品域名 → 稽核報告、關鍵字地圖、給 Content 的 brief |
| 審批點 | 網站結構大改、上 production 需核可 |
| 排程 | 每週自動巡檢(Hermes cron) |

### 3. Digital Marketing(渠道協調者)
| 欄位 | 內容 |
|---|---|
| 職責 | campaign 規劃、跨渠道協調、UTM/追蹤治理、漏斗分析、成效報表 |
| Skills | `campaign-plan`、`performance-report`、UTM 指南、ShortLink |
| MCP | HubSpot、Amplitude、Supermetrics、Klaviyo、GA |
| 輸入 → 輸出 | campaign 目標 / 預算 / 各渠道數據 → campaign brief、週/月報表、優化建議 |
| 審批點 | campaign 上線、預算分配需核可 |

### 4. Content Marketing
| 欄位 | 內容 |
|---|---|
| 職責 | 部落格、社群、電子報、案例、內容再利用 |
| Skills | `draft-content`、`email-sequence`、`eg-content-repurpose`、`internal-comms` |
| MCP | Claude World Studio 內容 MCP(trend-pulse、cf-browser)、發布工具 |
| 輸入 → 輸出 | 定位 + SEO brief → 各渠道內容草稿、編輯行事曆 |
| 審批點 | 對外發布前需核可 |

### 5. 廣告專員(Ad Specialist)— 最高風險職位 🚨
| 欄位 | 內容 |
|---|---|
| 職責 | 受眾分析、文案/素材 brief、出價策略**建議**、成效分析、A/B 規劃 |
| Skills | 自建 ad-copy / audience-research、`campaign-plan` |
| MCP | Supermetrics、Meta/Google Ads(**唯讀報表**) |
| 輸入 → 輸出 | campaign 目標 / 預算上限 / 受眾 → 文案、受眾建議、出價建議、成效分析 |
| 審批點 | 🚨 **實際投放 / 動用預算一律人類執行**;agent 只給建議。Paperclip 鎖月預算上限 |

### 6. 影片製作(Video Production)
| 欄位 | 內容 |
|---|---|
| 職責 | 腳本、分鏡、影片生成、字幕/多語、社群短影音 |
| Skills | `youtube-script`、`storyboard-generator`、`remotion`、`network-release-notes` |
| MCP | Remotion(render)、nanobanana(生圖)、Canva、NotebookLM(podcast/slides/video) |
| 輸入 → 輸出 | content brief + 品牌素材 → 腳本、分鏡、成片、字幕 |
| 審批點 | 對外發布前需核可 |

---

## 跨職位協作流程

```
產品行銷(定位)
    │  定位卡 / Message Guide(品牌單一事實來源)
    ▼
┌── SEO ──── 關鍵字地圖 + 內容 brief ──┐
│                                      ▼
└──────────────────────────────► Content / 影片(產出內容)
                                       │
                                       ▼
                                  廣告專員(受眾 + 文案 + 出價建議)
                                       │  人類核可後投放
                                       ▼
                                  Digital(統整 campaign + UTM 追蹤)
                                       │  成效回饋
                                       ▼
                                  產品行銷(迭代定位)← 閉環
```

---

## 治理層(Paperclip)與紅線

- **org chart**:6 職位 + 回報給「行銷總監(human)」
- **每個 agent 設月預算上限**(廣告專員尤其重要,超支自動暫停)
- **審批關卡**:對外發布、廣告花費、網站結構改動
- **immutable audit log**:所有決策與 tool call 可追溯
- **品牌單一事實來源**:全部 content/影片 agent 吃同一份 message guide + `.impeccable`

### ⚠️ 幫別人建,特別要顧的紅線
1. **花錢/投放絕不自動化** — 一律走人類審批,合約寫清責任歸屬。
2. **資料與密鑰隔離** — 每客戶獨立部署、各自 API key 走 secrets 管理。
3. **成熟度風險** — 工具都還新,**先 pilot 一個職位**再擴成整個部門。
4. **品牌一致性** — 同一份品牌 skill,避免多 agent 風格分裂。

---

## 導入路線(分階段,不要一步到位)

| 階段 | 做什麼 |
|---|---|
| **Phase 1 · Pilot** | 挑 1 個高價值職位(SEO 巡檢 或 Content 選題),用 Hermes + 客製 skill 跑出成效 |
| **Phase 2 · 擴編** | 加到 3–4 個常駐職位,串起協作流程 |
| **Phase 3 · 治理** | 疊 Paperclip:org chart、預算上限、審批、稽核,交付客戶 GUI |

---

## FDE 可重複資產(Starter Kit)

把第一次做的沉澱成模板,之後每客戶 fork:
- 標準 org 模板(6 職位 + 預算/審批預設)
- Baseline skill pack(6 職位通用 skill,客戶版覆寫)
- MCP 接線 playbook
- Discovery 問卷 + 交接文件模板
