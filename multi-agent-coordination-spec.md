# 多 Agent 協調規格 — Blackboard + RACI(輕量版)

> **建立日期**:2026-06-07
> **對象**:AI FDE / 想跑多 agent 又不想先上重 infra 的人
> **一句話**:一塊共享白板(`marketing-state.md`)+ RACI 角色契約 + 檔案式 handoff + 顯式人類審批,**全跑在 Claude Code,零額外 infra**。

---

## 設計原則

1. **單一事實來源** — `marketing-state.md` 是全隊唯一協調狀態,沒寫進去的事不存在。
2. **一個 section 一個 owner** — 每職能只寫自己 section + 共用 log,避免搶寫(對 Auto-Company 單一大檔的改良)。
3. **RACI 決定誰能做什麼** — 只在自己 RACI 範圍內動作,跨界走 handoff。
4. **人類審批是顯式關卡** — 凡「人類 = A」(發布、花錢、定位)先進審批佇列,核可才放行。
5. **Append-only log** — 所有動作寫進變更紀錄,當窮人版 audit log。

## 執行模型(兩種)

| 模式 | 怎麼跑 | 何時用 |
|---|---|---|
| A. 手動 | 在 Claude Code 叫某職能 skill,讀 state → 做 → 寫回 | 驗證期(現在) |
| B. 自動迴圈 | shell 迴圈反覆呼叫 `claude`,orchestrator 派工 | 要 24/7 時 |

兩者讀寫同一個 `marketing-state.md`,無痛升級。

## 黑板 `marketing-state.md` 結構

| Section | Owner | 內容 |
|---|---|---|
| 本週目標 | 策略(人類核可) | 北極星 + 重點 |
| 審批佇列 | 任何職能放入,**只有人類可核可** | 等拍板的事 |
| 各職能 Next Action | 各自只寫自己那行 | 下一步 + 狀態 |
| 進行中 Handoffs | 收發雙方 | 交接索引 |
| 成效快照 | Analytics | 數據 + 洞察回饋 |
| 變更紀錄(append-only) | 全體 | 時間 + 誰 + 做了什麼 |

## 協調迴圈

```
1. 讀 state(目標 + 自己的 Next Action)
2. 掃 handoffs/:有沒有指給我、status=requested 的
3. 在「我的 RACI 範圍內」執行
     ├─ 需要別人東西 → 開 handoff(不硬做)
     └─ 產出要對外 → 進審批佇列(不直接發)
4. 寫回自己 section + append log
5.(模式 B)sleep → 下一輪
人類 = 編輯審批佇列標核可,或改本週目標轉向
```

## Handoff 狀態機

```
requested → in-progress → delivered → closed
```
一張 = `handoffs/HO-YYYY-NNNN.md` 一個檔(含 from/to/status/request/brand_ref/deliverable)。接收方掃「to=我、requested」;發起方掃「我發起、delivered」收下 closed。

## 人類審批關卡

`審批佇列` 是唯一「對外 / 花錢 / 定調」的出口:任何職能放入標 `[ ] (待核可)`,**只有人類**能改 `[x] (已核可)`,職能只執行被核可的對外動作。直接對應 RACI「人類 = A」的列。

## 端到端範例(發一篇部落格)

```
策略設目標 → Organic 寫草稿 + 開 handoff 要主視覺
 → Creative 接單做圖 → delivered → Organic 收下嵌入 closed
 → 進審批佇列 → 人類核可 → Organic 發布(帶 UTM) → Analytics 量測回饋(閉環)
```

## 實證(dogfood,2026-06-07)

在 Claude Code 上實際跑過一輪(EnGenius 情境,主題:中小企業 WiFi 佈建指南):
- 黑板全程唯一事實來源,每職能只寫自己 section ✅
- handoff HO-0007 走完 requested → in-progress → delivered → closed ✅
- RACI 邊界守住:Organic 沒自己做圖、Creative 只交付不發布 ✅
- **真實產物**:用 nanobanana 產出主視覺(吃中品牌:暖中性 / teal / 無純黑)→ 流過 handoff → 嵌入文章 ✅
- 審批關卡守住:文章卡在「待核可」,人類核可前不發布 ✅

### 閉環:量測 → 回饋 → 迭代(W23 → W24)
再跑一個週期,證明部門能**依數據自我改進**:
- Analytics 量測已發布內容 → 產成效報告
- 透過黑板「成效快照」把洞察回饋策略(PoE ROI 高、內鏈有效)
- 策略據此**迭代下週目標**(WiFi → 主打 PoE + 雙向內鏈),仍過人類核可

```
策略設目標 → Organic 寫稿 ─[handoff]→ Creative 做圖 → Organic 收回
 → 審批 ─[人類核可]→ 發布 → Analytics 量測 ─[黑板回饋]→ 策略迭代下週目標 → …(自我改進閉環)
```

**兩種協調路徑都實證跑通**:交付物走 handoff(點對點);訊號走黑板(成效快照)。
> ※ 成效數字為 dogfood 模擬值(非真實 GA),證明迴圈會動;實際部署由 GA / Search Console MCP 拉真值。

→ 證明 **Claude Code + 一個 Markdown 黑板 + RACI/handoff,就能跑一個會自我改進、有治理的多 agent 部門**,不需要 Paperclip/Hermes。

## 規模化限制與升級時機

- 單檔瓶頸 → 拆 per-role 檔 + index。
- 沒有硬預算 / 不可竄改稽核 → **要時升 Paperclip**。
- 升級判斷:手動夠用就別動 → 太累上模式 B → 怕燒錢 / 交非技術客戶上 Paperclip。
