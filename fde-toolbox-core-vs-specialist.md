# FDE 工具箱:核心 vs 專用

> **建立日期**:2026-06-07
> **對象**:用 AI agent 幫客戶建置/代營運的 FDE 顧問
> **重點**:哪些工具天天用(核心),哪些看情境才拿(專用),以及各自的觸發訊號

---

## 核心 vs 專用

| 角色 | 工具 | 用法 |
|---|---|---|
| **核心(天天用)** | Paperclip + Claude Code + Obsidian | 主力堆疊,所有客戶 |
| **專用(看情境)** | Hermes / Claude World Studio | 特定需求出現才點召 |

> 一人顧問最大的隱形成本是「工具太多、每個都要顧」。**核心三件先穩,專用工具按需才加。**

---

## Hermes — 獨有價值:常駐排程 + 聊天通道

Claude Code 是 session 式、Paperclip 是駕駛艙。Hermes 補的是 **① 24/7 常駐 + cron 排程 ② 原生 messaging(LINE/Telegram/Slack/WhatsApp)**。

| 情境 | 為什麼是 Hermes |
|---|---|
| 你自己的營運助手 | 每週自動拉各客戶數據、每早彙整待辦(cron + 常駐) |
| 客戶要聊天互動 | 客戶用 **LINE** 問「campaign 如何?」→ Hermes 直接回 |
| 給客戶 chatbot 窗口 | Tier 3 客戶要「能對話的 AI 窗口」 |
| 小客戶精簡版 | 一個 Hermes 跑 $5 VPS 做排程,當 Tier 2.5 |

**觸發訊號**:要 24/7 排程 / 要聊天通道(尤其 LINE)。

---

## Claude World Studio — 獨有價值:內容/媒體生產線

縱向成品:趨勢 → 內容 → 評分 → 發 Threads + NotebookLM 生 podcast/簡報/影片。只覆蓋 6 職能裡的 Content、Creative/影片。

| 情境 | 怎麼用 |
|---|---|
| 內容量很大的客戶 | 近乎開箱即用的內容工廠 |
| 要 podcast/影片/簡報 | 用它的 NotebookLM 整合 |
| ⭐ 只借它的 MCP(最實際) | 把 `trend-pulse`/`notebooklm`/`cf-browser` 插進客戶 Paperclip 的 Content agent,不用整個 App |
| Threads 經營 | 原生支援發布 |

**觸發訊號**:客戶內容/社群/影片導向 → 優先「借 MCP」而非用整套。

---

## 一張表決定拿哪個

| 需求 | 工具 |
|---|---|
| 多 agent 部門 + 治理 + 駕駛艙 | **Paperclip + Claude Code** |
| 你自己日常交付 | **Claude Code** |
| 24/7 排程 / LINE·Slack 聊天 | **Hermes** |
| 大量內容/影片/podcast/發 Threads | **CWS**(或只借 MCP) |

---

## Paperclip 一個 company 裡,agent 可以「混編」

每個 agent 可獨立選 **runtime + model**(adapter:claude-local / codex-local / gemini-local / http / process / 自建):

| agent 類型 | 用什麼 adapter | 適合 |
|---|---|---|
| 創意/推理 | Claude Code | content、策略 |
| 大量低成本 | **Gemini** | 批次生成 |
| 確定性任務 | **process(腳本)** | 拉 GA 報表、跑固定流程(不燒 LLM) |
| 系統整合 | **HTTP / Webhook** | 接客戶既有系統 |
| 常駐/聊天 | **Hermes**(經 http / 自建 adapter) | LINE 窗口、排程 |

→ 部門可「LLM agent + 確定性腳本 + 系統整合」混編,**全由 Paperclip 一個駕駛艙治理(預算/審批/稽核)**。文件原話:adapter 可「connect to any agent runtime」。

---

## 紅線

1. **別預先導入** —— 沒具體客戶需求或自己的重複痛點,先不裝。
2. **每多一工具 = 多一份維運面**(各自跑、各自更新、各自接金鑰)。
3. **成熟度風險** —— 都還新,押客戶 production 前先在自己/EnGenius 驗過。

## 給台灣 SMB 客群的建議

- **Hermes 第一個值得加** —— 台灣客戶愛 **LINE**,「用 LINE 跟 agent 對話」好賣又有感;你自己也能拿來排程。
- **CWS 先別整套** —— 記住「借 MCP」這招,接到內容/影片重的客戶再用。

> 一句話:核心永遠 Paperclip + Claude Code + Obsidian;**Hermes 在「要排程/要 LINE」時加,CWS 在「內容/影片重」時(優先借 MCP)加。**
