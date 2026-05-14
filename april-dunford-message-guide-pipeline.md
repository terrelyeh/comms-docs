# April Dunford 定位方法論 × Message Guide Skill Pipeline

> 建立日期：2026-05-14
> 目的：把 April Dunford 的定位方法論，以及我們如何把它變成 AI 工作流（四個 Claude Code skill）整理成內部共識，方便 PMM / PM / 通路 / 業務理解這套流程在做什麼、為什麼這樣做、怎麼用。

---

## 一、April Dunford 是誰

April Dunford 是 B2B 科技產品定位專家，著有兩本業界標準書：

- **《Obviously Awesome》(2019)** — 定位方法論本體
- **《Sales Pitch》(2023)** — 把定位翻譯成業務可講的故事

她的方法**已成為 B2B SaaS 圈定位的事實標準**。重點不是「品牌包裝」，而是回答一個更基本的問題：**「在客戶心中，你的產品應該被放在哪個貨架上？」**

---

## 二、方法論核心：5(+1) 構件

定位不是 tagline、不是 mission，而是要明確回答下面這 5 件事：

| 構件 | 要回答的問題 |
|---|---|
| **1. Competitive Alternatives** | 如果你不存在，客戶會用什麼？（不只是競品，可能是 Excel、人工、什麼都不做） |
| **2. Unique Attributes** | 你有什麼是替代方案沒有的功能/能力？ |
| **3. Value & Proof** | 這些 attributes 對客戶帶來什麼價值？有證據嗎？ |
| **4. Best-fit Customers** | 哪種客戶會因為這些價值而瘋狂喜歡你？ |
| **5. Market Category** | 你選擇在哪個市場「框架」下競爭？ |
| **(+1) Trends（選用）** | 你能搭上什麼趨勢，讓客戶覺得現在非買不可？ |

### Dunford 三大商業角色（3 Strategies）

定位之外，每個產品還要選一條商業角色：

| 策略 | 適用情境 |
|---|---|
| **Head-to-Head** | 在既有品類內正面競爭，爭取「最好的 X」位置。需要規格 / 通路 / 品牌任一面向有領先優勢 |
| **Big Fish, Small Pond** | 定義一個你能贏的子市場（生態系、特定客群、特殊場景）。「在大池塘是小魚、在小池塘是大魚」 |
| **Create New Market** | 重新定義品類、幫新品類命名、建立新規則。需要明確的 market shift 證據 |

選錯角色 = 跟錯誤的對手打 = 客戶用錯誤的標準評你。

---

## 三、方法論的特別之處

跟其他定位方法相比，Dunford 有 4 個關鍵差異：

### 1. 從「替代方案」反推，而不是從「我們很棒」開始

傳統做法是列產品功能；Dunford 強迫站在客戶角度——客戶不是在 A vs B vs C 之間選，他們是在「你 vs 現狀」之間選。

### 2. Market Category 是策略選擇，不是既定事實

同一個產品可以放在不同類別上競爭。她著名案例是把一個「資料庫工具」重新定位成「給某類分析師的工作平台」，價格直接翻倍。

### 3. 定位 → 銷售敘事是線性流程

《Sales Pitch》補上了缺角：定位完之後，怎麼變成業務在會議室講的故事。結構是 **Insight → Alternatives & their flaws → Perfect World → Introducing us → Proof**。

### 4. 適合「產品已存在但賣不動」的情境

不是給 0 → 1 的新創想商業模式用的，是給「東西做出來了、有些客戶、但市場反應混亂」的團隊重新對焦。

---

## 四、為什麼我們把它變成 Skill Pipeline

EnGenius 是網通品牌，產品線多、規格密集、競品環伺。同一支 PM Brief 從產出到對外溝通要經過多個角色（PM / PMM / 業務 / 通路 / PR），每個角色都需要從同一個定位拿東西用。

過去的痛點：
- PM Brief 規格太多，但客戶價值講不清楚
- PMM 寫對外文件時憑感覺發揮，每份產品定位不一致
- 業務 / 通路拿到的訊息跟官網不對齊
- 重要的「不擁有清單」（誠實揭露的弱項）沒人寫，下游素材打嘴

於是把 Dunford 方法論變成**強制走完的 AI 流程**：每個產品都跑同一條 pipeline，每個階段都有確認點、有結構化產出。

---

## 五、Skill Pipeline 四階段

對應 Dunford 方法論的四個 Claude Code skill，從 PM Brief 一路到對外可發佈的 HTML：

```
PM Brief
   ↓  /message-guide-review     ← Stage 0：體檢 Brief 品質
   ↓
   ↓  /message-guide-narrative  ← Stage 1：跑 Dunford 五要素 + 商業角色判定
   ↓
   ↓  /message-guide-build      ← Stage 2：把定位翻譯成對外文件
   ↓
   ↓  /message-guide-render     ← Stage 3：套版視覺 + 部署上線
   ↓
4-tab Message Guide 上線
```

### 每個 skill 的職責

| Stage | Skill | 主要動作 | 對應 Dunford |
|---|---|---|---|
| **0** | `/message-guide-review` | 用 5 面向 rubric 評 PM Brief 品質，找盲點 | 攔截 GIGO，確保 Brief 夠撐 Dunford 5 構件 |
| **1** | `/message-guide-narrative` | 商業角色判定 + 五要素敘事 + 競品 first-source 取證 + 魔鬼代言人壓測 | Dunford 5 構件 + 3 strategies + Sales Pitch 五段主線 |
| **2** | `/message-guide-build` | 把定位卡寫成對外 Part A-E 結構化內容 | Sales Pitch 落地：把推理變對外語言 |
| **3** | `/message-guide-render` | 整合所有上游產出 → multi-tab HTML 上線 | 視覺化呈現 + 永久 trace |

---

## 六、跟 Dunford 原方法相比，我們加了什麼

Dunford 原書沒有的、我們在實作中加入的**紀律**：

### 1. V/A/P 三級證據標註（最有原創價值）

| 標註 | 意義 | 規則 |
|---|---|---|
| **[V] Verified** | 有獨立來源 | **必附 inline 可點擊連結**（官網 / datasheet / 分析師報告 / press release）。沒連結 = 不算 V，降級為 A |
| **[A] Assumption** | 推論，無來源 | 寫進文件時要明標。**[A] 不可進 Pillar bullet**（會被下游當事實流出） |
| **[P] Pending** | PM 未給 | **不可進對外 Guide**，一律收進 Appendix Action Items 給 PM 補 |

→ 解決了「PMM 把推論當事實流出 → 下游素材踩雷」的問題。

### 2. 「不擁有清單」強制 artifact

Dunford 強調 *saying no to what you're not*，但沒規定怎麼存。我們強制 narrative skill 在 Phase 4a C 之後產出「本產品不擁有的」清單，避免下游打嘴。

### 3. 魔鬼代言人 Q1-Q5 壓測

主敘事 A-E 寫完強制跑五題：
- Q1：世界真的變了嗎？
- Q2：舊方案真的失效嗎？
- Q3：Frame of reference 站得住嗎？
- Q4：不擁有清單夠誠實嗎？
- Q5：Secondary Audience 真的有嗎？

通不過的回頭改 A-E。

### 4. 商業角色三軸 Pillar（What / Who / Why us）

每個產品強制三條 Pillar：
- **What** — 產品本體（規格組合 right-sizing 邏輯）
- **Who** — Target fit（為哪類 target 設計）
- **Why us** — 差異化（平台 / 生態系 / 通路 USP）

防止 Pillar 全寫成平台論述、產品間沒辨識度。

### 5. Pillar + Capabilities 雙軌

Dunford 強調差異化，但實務上買家**先要確認「這台是稱職的 [品類] 產品」**才會聽差異化。所以：
- **B3 Pillars** = 為什麼選我們（差異化）
- **B4 Capabilities** = 這台是不是稱職的 [品類]（品類基本盤）

避免「基本盤誤當差異化」或「以為不寫基本盤就算了」的陷阱。

### 6. 對外用語紅線

對外文件**絕對不可出現**內部 reasoning 詞彙：
- `Head-to-Head` / `Big Fish, Small Pond` / `Create New Market`
- `池塘` / `Ecosystem Captive` / `四層借力 L1-L4`
- `魔鬼代言人` / `V/A/P` 字母
- `Dunford`

要對外講商業角色時用：`Category challenger` / `Subcategory leader` / `Category creator`。

---

## 七、Multi-tab Message Guide 結構

Render skill 產出的每個產品頁面是 4 個 tab：

| Tab | 內容 | 受眾 |
|---|---|---|
| 📄 **Input** | PM Brief 原文（含 Google Doc URL）+ optional HQ Guide | PMM / PM 追溯起點 |
| 📊 **Diagnostic** | Phase 3.5 異常診斷（條件出現） | 內部 audit |
| 🔧 **Internal** | narrative.html 完整推理（折疊預設關閉，含內部術語警告） | PMM 學習 / 將來複用 |
| 🌐 **Outbound** ★ | 對外文件（default active）| 通路 / 業務 / PR / SI |

設計理念：**每個產品自成一頁，含完整 pipeline trace**。
- 通路 / 業務 / PR → default 看 Tab 4
- PMM 想追根究底 → 切到 Tab 1 看原始 Brief、Tab 3 看內部推理
- 新 PMM onboarding → 4 個 tab 全看一遍學完整 pipeline

---

## 八、什麼情境適合用

### ✅ 很適合

- B2B 科技產品（特別是 SaaS、網通、企業軟硬體）
- 產品功能多、不知道怎麼講重點
- 業務 demo 都在講功能、客戶聽不懂價值
- 跟競爭對手切入差異不明顯，需要重新定義戰場
- 定價上不去、覺得被當 commodity
- 跨團隊（PM / PMM / 業務 / PR）訊息不一致

### ⚠️ 不太適合

- 純 B2C 大眾消費品（情感品牌驅動為主）
- 還沒有產品、還沒有客戶的早期構想階段
- 純品牌行銷活動（這是商業定位，不是品牌定位）

### Pipeline vs 直接用書

| 直接讀 Dunford 的書 | 用這套 Skill Pipeline |
|---|---|
| 自己讀、自己消化、自己應用 | AI 引導跑完所有步驟 |
| 容易跳過難題（例：不擁有清單） | 強制每個步驟有產出 |
| 一個人腦中跑 | 多角色都能讀同一份結構化文件 |
| 沒有版本管理 | 每次跑都有檔案落地、可追溯 |
| 對外文件靠手寫 | 自動套品牌 template + 部署上線 |

---

## 九、典型一個產品跑完整流程

從拿到 PM Brief 到對外 Message Guide 上線，**約 2-4 小時純執行時間**（不含等 PM 補資料的等候時間）。

| 階段 | 預估時間 | 產出 |
|---|---|---|
| Stage 0 Review | 5-10 分鐘 | Markdown 報告 + JSON + `inputs/brief.md` 落地 |
| Stage 1 Narrative | 60-90 分鐘 | `narrative.html` + `positioning-card.yaml` |
| Stage 2 Build | 30-45 分鐘 | `message-guide-content.md`（Part A-E） |
| Stage 3 Render | 5-10 分鐘 | Multi-tab HTML + metadata JSON + Vercel 上線 |

每個 stage 之間都有 🔴 確認點，要 PMM / Lulu 拍板才往下走。

---

## 十、常見問題

### Q1 · 為什麼 Stage 0 Review 不會修 Brief 內容？

Review 是 **gate（守門員），不是 transformer**。職責是評估、不是改寫。發現問題就回頭找 PM 修 Brief，修完再 review 一次。如果 review 也改寫，會喪失中立性。

### Q2 · 為什麼定位卡（positioning-card.yaml）是 freeze 點？

定位敘事跟對外寫作要分開做。否則寫文案時為了句子好看，會偷偷改定位。Freeze 點確保**改定位 = 回 Stage 1 重跑**，不能在 Stage 2 偷偷動。

### Q3 · 通路看不懂 Tab 3 內部術語怎麼辦？

Tab 4 (Outbound) 是 default active，**通路打開就看到對外清版**。Tab 3 預設折疊 + 有警告 badge，要主動展開才看到推理過程。

### Q4 · PM 還沒給定價怎麼辦？

走 narrative skill 的 `[A]-driven mode`：用假設定價跑完流程，但 `positioning-card.yaml` 會標記 `ready_for_build: false`，build / render skill 看到旗標就 stop，**對外文件不能從這份產出**，只能當內部討論用。等 PM 補了 final MSRP 再重跑。

### Q5 · 跨 PMM 一致性怎麼保證？

所有 PMM 用同一份 skill 邏輯（共用 `engenius-message-guide-skills` repo + 共用 `context/`）。skill 強制檢核項目，例如：Pillar 數量 3-4、Use Case 必含 Not ideal for、V/A/P 標註齊全。

---

## 十一、相關連結

- 📦 GitHub repo：`terrelyeh/product-message-guide`（含 4 個 skill + context + examples）
- 🌐 內部成品站：`eg-message-guide.vercel.app`（密碼保護）
- 🌐 方法論說明站：`message-guide-methodology.vercel.app`（Lulu 維護）
- 📚 推薦閱讀：April Dunford《Obviously Awesome》、《Sales Pitch》

---

## 結語

Dunford 方法論的核心是「**定位是選擇，不是發現**」。把它變成 AI 流程後，最大價值是**強制每個產品都做完同樣的選擇步驟**——不會因為時程壓力、PMM 風格不同、或文件規格不全而跳過關鍵環節。

V/A/P 紀律、不擁有清單、What/Who/Why us 三軸——這些都是把「方法論的精神」變成「下游素材的紀律」。對外清版乾淨、內部 trace 完整——這就是這套 pipeline 想達成的事。
