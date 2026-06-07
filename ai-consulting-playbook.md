# AI 顧問服務 Playbook(總覽)

> **建立日期**:2026-06-07
> **對象**:用 AI agent 幫客戶建置 / 代營運的 FDE 顧問(= 你)
> **用途**:這一路討論的端到端方法論總整理 + Starter Kit 索引。新 session 從這頁接上。

---

## 0. 你到底在賣什麼(先定錨)

> **你賣的不是工具,是「把客戶 know-how 變成 skills + 把流程變成 org/預算/審批」的能力。**
> 工具(Paperclip/Hermes/CWS)是 commodity,會換;**你的方法論 + Starter Kit + 領域 know-how** 才是 IP。

- **商業模式**:retainer 月費維運(持續加 agent/skill = 續約理由)
- **客群**:SMB / 中型(台灣)
- **起手**:先在 **EnGenius / 自己品牌** dogfood → 出案例 → 再對外

---

## 1. 端到端流程

```
客戶說「想導入 AI」(多半沒 idea)
   ▼
① 資格判斷 ── Tier 計分卡(6 問,4+ yes → Tier 3)
   ▼
② 分層 ── Tier 1 專案 / Tier 2 月聘 / Tier 3 AI 部門
   ▼ (Tier 3)
③ Discovery ── 畫客戶的「AI 組織圖」,盤點流程 / 工具 / KB
   ▼
④ 建置 ── Paperclip company + 職能 agents(依規模縮放)+ 匯入客製 skills
   ▼
⑤ 營運 ── 3a 代營運(你 VPS)/ 3b 部署交付(客戶端)
   ▼
⑥ 計費 ── retainer;持續加 agent / 迭代 skill / 報成效
```

---

## 2. 工具地圖

| 層 | 工具 | 角色 |
|---|---|---|
| **核心(天天用)** | **Paperclip**(駕駛艙)+ **Claude Code**(引擎)+ **Obsidian**(知識) | 所有客戶 |
| 專用(看情境) | **Hermes**(24/7 排程 / LINE 聊天)、**CWS**(內容/影片,優先借 MCP) | 觸發才加 |
| 參考(別直接交付) | Auto-Company(全自動公司,當教材) | — |

> 細節 → 「FDE 工具箱:核心 vs 專用」。**輕量先行**:Claude Code + 黑板就能跑多 agent,被規模逼才上 Hermes/Paperclip 的重 infra。

---

## 3. 客戶分層(Tier)

| 層級 | 類型 | 套什麼 | 商業模式 |
|---|---|---|---|
| Tier 1 | 一次性分析/交付 | 顧問客戶骨架 | 專案費 |
| Tier 2 | 持續顧問(你+Claude) | 顧問客戶骨架 | 月聘 |
| **Tier 3** | 導入 AI 部門(Paperclip) | 骨架 + **AI部門模組** | **retainer** |

**Tier 3 計分卡**(4+ yes):持續/重複?量夠大?付得起 retainer?有 KB 可餵?有人會審核?接受「AI 增強+人類把關」?
**Tier 3 再分**:3a 代營運(你 VPS)/ 3b 部署交付(客戶端)。

---

## 4. 建置:虛擬行銷部門(6 同層職能)

策略&產品行銷(大腦)· Organic Growth · Paid/Demand Gen · Lifecycle/CRM · Creative Studio · Analytics&Ops
- 原則:照 **workflow + 工具面 + 審批邊界**拆,不照人類職稱;依規模縮放(SMB 3 / 中型 6 / Enterprise 8–10)
- 職責用 **RACI** 講清楚;**老闆 vs 產能**心法 + handoff 寫進 skill
- 協調靠**黑板(`marketing-state.md`)+ RACI + 檔案 handoff + 人類審批佇列**(已 dogfood 跑通閉環)

> 細節 → 「虛擬行銷部門藍圖」、「多 Agent 協調規格」。

---

## 5. 部署:Paperclip

- 安裝(corepack 免 sudo)→ `onboard`(建 config+JWT)→ `doctor` → 啟動 `127.0.0.1:3100`
- 建 Company → Agent(Claude Code adapter)→ Task → Launch
- Skill:`skills import` 進 company → `agent skills:sync` 掛 agent
- **供電**:Claude Code adapter = 吃**訂閱**(沒 API key);要計量/客戶分帳才用 **API key**。可 per-agent 混編(claude/gemini/script/http/Hermes)

> 細節 → 「Paperclip 部署 SOP」。

---

## 6. Obsidian 對應(你的顧問底稿 ↔ Paperclip)

```
工作/顧問/客戶X/
├── (顧問客戶骨架) 合約/會議/分析/交付物/筆記  = 你的底稿(模式 A,給你)
└── (AI部門模組,Tier 3 才疊)
       ├── 00-品牌核心/  = agent KB ─┐
       └── skills/       = 客製職能 ─┼─→ 餵/import ─→ Paperclip company「客戶X」
```
- 一個 vault 多資料夾(別每客戶開獨立 vault);**Paperclip company ↔ 顧問/客戶X ↔ company skill library**
- 觸發:「新增顧問客戶:X」(Tier 1/2)/「把 X 升級成 AI 部門」(Tier 3)

---

## 7. 紅線(交付給別人務必守)

1. **花錢 / 對外發布一律人類審批** — agent 只建議,不自動執行
2. **資料 / 金鑰隔離** — 代營運多客戶共機要分開金鑰;別在筆電跑 production → 上 VPS
3. **別預先導入工具** — 沒觸發訊號不裝;每多一工具 = 多一份維運
4. **成熟度風險** — 工具都新,先自己/EnGenius 驗過再上客戶
5. **品牌一致性** — 全部 agent 吃同一份品牌 KB,避免風格分裂

---

## 8. 文件索引(Starter Kit)

| 文件 | 內容 |
|---|---|
| 虛擬行銷部門藍圖 | 6 職能 × RACI × 規模縮放 × 協作流程 |
| 多 Agent 協調規格 | 黑板 + RACI + handoff + 審批(含 dogfood 實證) |
| Paperclip 部署 SOP | 安裝/onboard/skills/踩坑 |
| FDE 工具箱:核心 vs 專用 | Hermes/CWS 何時加 + adapter 混編 |
| 套件安裝避坑指南 | uv/npm/pip/skill/MCP |

---

## 9. 現在的進度 & 下一步

- ✅ 已驗:輕量協調閉環(dogfood)、Paperclip live 跑通(EnGenius + Organic Growth skill)、Obsidian 顧問體系 + 模組
- ▶️ 下一步:① 多職能 dogfood 跑完整閉環 ② 接第一個外部 Tier 3 客戶 ③ 把 demo/代營運搬上 VPS
