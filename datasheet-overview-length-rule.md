# Datasheet Overview 字數規則

> 建立日期：2026-05-09
> 目的：說明 Product SpecHub 產生 Datasheet 時，封面 Overview 文字的長度規則，
> 讓 PM / MKT 在撰寫和翻譯時知道大概的安全區間，以及超出時系統會怎麼提醒。

---

## TL;DR

- **沒有固定字數上限**，因為 Overview 跟 Features 共用同一塊空間，是動態算的
- 安全參考字數（features 量中等時）：
  - 英文 ≈ 800 字
  - 日文 / 繁中 ≈ 600 字
- 超過會在 Dashboard 顯示**紅點**和警告橫幅，但**不會擋 PDF 生成**
- 確認版面 OK 可按「Mark as Reviewed OK」消除提醒
- 每個語言獨立判斷：英文沒紅燈不代表日文也沒事

---

## 為什麼沒有固定字數上限？

Cover 頁的版面是這樣切的：

```
┌──────────────────────┐  頁面頂端
│  Hero + Hardware Image │  固定區塊
├──────────────────────┤
│                       │
│  Overview ← 彈性      │  ← 吃剩下的空間
│                       │
├──────────────────────┤  ← 動態邊界
│  Features（最多 320pt）│  由下往上長，硬上限
└──────────────────────┘  頁面底部
```

**核心規則：Features 先佔，Overview 拿剩下的。**

所以你寫越多 features，留給 overview 的位置自然越少；反過來也一樣。
這也是為什麼沒有「Overview 上限 N 字」這種一刀切的數字。

---

## 安全字數參考

以下是 features 量**中等**（約 6–8 條，每條一行）時的大概安全字數：

| 語言 | 安全字數 | 備註 |
|---|---|---|
| English | 約 **800 字** | 超過建議先預覽 |
| 日文 / Japanese | 約 **600 字** | CJK 字較大、行高較高 |
| 繁中 / Traditional Chinese | 約 **600 字** | CJK 字較大、行高較高 |

> ℹ️ 同樣意思翻譯成日文 / 繁中後，會比英文佔更多排版空間，
> 所以建議翻譯後的字數比英文版本少一些。

如果 features 比較多（10 條以上、其中有多行的），overview 能放的就要再扣。
反之 features 很少時可以放更多。

---

## 超出會怎樣？（系統行為）

1. **Dashboard 對應 OV 欄位顯示紅點**
   一眼看到哪個 model 的 overview 超出了
2. **Product 詳細頁出現紅色警告橫幅**
   告訴你哪個語言超出、超出多少
3. **不會擋 PDF 生成**
   你還是可以正常按 Generate / Regenerate
4. **看過覺得 OK → 按「Mark as Reviewed OK」**
   紅燈消失，Dashboard / 詳細頁恢復正常
5. **內容一改紅燈會自動回來**
   系統用 hash 追蹤 overview / features，內容變動時 Mark 自動失效，
   避免「之前 ack 過後來 PM 改了沒人知道」

---

## 實際操作流程建議

```
寫完 / 翻譯完 Overview
       ↓
看 Dashboard 對應語言 OV 欄
       ↓
有紅點？ ── No ──→ 完成 ✅
       │
      Yes
       ↓
打開 Preview 看實際版面
       ↓
版面 OK？ ── No ──→ 縮 overview 或 features
       │
      Yes
       ↓
按「Mark as Reviewed OK」
       ↓
完成 ✅
```

---

## 為什麼 CJK 比英文緊？

Datasheet Cover 頁排版時，每個語言用不同的字級和行高：

| Locale | 字級 (pt) | 行高 (pt) | 每行能塞的字 |
|---|---|---|---|
| English | 11 | 15 | 54 chars |
| 日文 | 11.5 | 17 | 46 chars |
| 繁中 | 12 | 18 | 44 chars |

中日字符在計算寬度時當作 2 個 char slot（雙寬字符），
所以同樣意思的句子翻成 CJK 後，**行數會明顯增加**，
更容易撞到 features 區的邊界。

---

## 常見誤解 vs 正確觀念

| ❌ 誤解 | ✅ 正確 |
|---|---|
| Overview 上限是固定字數 | 動態的，跟 features 量有關 |
| 紅燈代表不能出 PDF | 紅燈只是提醒，PDF 仍可正常生成 |
| 英文 OK 表示其他語言也 OK | 每個語言獨立判斷，常見英文過但日文紅 |
| Mark OK 過就一勞永逸 | 內容一改 hash 變動，紅燈會自動回來 |

---

## 相關功能位置

- **看紅點**：Dashboard → 對應產品列的 OV 欄位
- **看警告橫幅**：Product 詳細頁頂部
- **Mark as Reviewed OK**：警告橫幅右側按鈕（admin / editor 角色才能按）
- **Preview**：產品頁右上「Preview」按鈕，每個語言獨立預覽

---

## 快速備忘

> **寫完先看 Dashboard，紅點 → 預覽確認 → 沒事就 Mark OK。**

每個語言都要分別檢查一次。CJK 比英文緊，請預留空間。
