# 內部工具的存取控制方案評估

> 建立日期：2026-03-28
> 目的：內部討論用，評估如何保護綁定 API Key 的工具類網站

---

## 為什麼需要存取控制？

當內部工具綁定了 API Key 或涉及付費服務，就需要存取控制來管理誰能使用。沒有身份驗證代表：

| 風險 | 說明 |
|------|------|
| 費用風險 | 任何人都能呼叫付費 API，費用由我們承擔 |
| 無法追蹤 | 不知道是誰在使用、用了多少 |
| 無法管理 | 無法停用特定人的存取權限 |

> **API Key 安全說明：** API Key 存放在 Vercel 環境變數中，只在 Serverless Function 內使用，不會暴露到前端瀏覽器。以下所有方案都維持這個架構不變。

---

## 方案一：靜態密碼保護

**最快上線 · 約 30 分鐘實作**

### 概念

所有使用者共用一組密碼。打開網站時需要先輸入密碼才能使用，密碼存在瀏覽器中，下次不用再輸。

### 運作流程

```
使用者打開網站 → 顯示密碼輸入頁 → 輸入密碼，存入瀏覽器
→ 每次呼叫 API 時帶上密碼 → 後端比對密碼 → 正確才執行
```

### 需要什麼

| 項目 | 說明 |
|------|------|
| 外部服務 | 無 |
| 資料庫 | 無 |
| 環境變數 | `SITE_PASSWORD`（存在 Vercel） |
| 前端改動 | 新增密碼輸入頁面 + 每次 API 請求帶密碼 header |
| 後端改動 | 每個 API function 開頭加一行密碼驗證 |

### 實作方式

**Vercel 環境變數**設定一組密碼：
```
SITE_PASSWORD=你設定的密碼
```

**後端**：每個 `api/*.ts` 開頭加入驗證：
```typescript
const password = req.headers['x-site-password'];
if (password !== process.env.SITE_PASSWORD) {
  return res.status(401).json({ error: 'Unauthorized' });
}
```

**前端**：新增密碼輸入畫面，通過後將密碼存入 `localStorage`，之後每次 API 請求在 header 帶上 `x-site-password`。

### 管理方式

| 操作 | 怎麼做 |
|------|--------|
| 換密碼 | 去 Vercel 改環境變數 → 重新部署 → 通知所有人新密碼 |
| 加人 | 直接把密碼告訴對方 |
| 移除人 | 換一組新密碼，通知其他人（無法只停用一個人） |

### 好處

- 實作最快，30 分鐘內上線
- 不依賴任何外部服務
- 不需要資料庫
- 使用者體驗簡單，輸入一次就好

### 限制

- 所有人共用同一組密碼
- 無法辨識是誰在使用
- 無法停用特定人
- 密碼可能被轉傳給不該有權限的人
- 無法追蹤個人用量

### 適用時機

只有 1–3 人使用，或是需要最快速度先保護起來，之後再升級到更完整的方案。

---

## 方案二：Google 登入 + Email 白名單

**不需要資料庫 · 約 3–4 小時實作**

### 概念

使用者用自己的 Google 帳號登入，後端驗證身份後比對 email 是否在白名單內。白名單存在 Vercel 環境變數中，不需要任何資料庫。

### 運作流程

```
使用者打開網站 → 顯示 Google 登入按鈕
→ Google 登入成功，回傳 ID Token（含 email）
→ 前端將 Token 帶在 API 請求 header
→ 後端向 Google 驗證 Token 真偽
→ 取出 email，比對白名單 → 在白名單上才放行
```

### 需要什麼

| 項目 | 說明 |
|------|------|
| 外部服務 | Google Cloud Console（免費） |
| 資料庫 | 無 |
| 環境變數 | `GOOGLE_CLIENT_ID` + `ALLOWED_EMAILS` |
| 前端改動 | Google Sign-In 按鈕 + Token 管理 |
| 後端改動 | 每個 API function 驗證 Google ID Token + 比對白名單 |
| 前置設定 | Google Cloud Console 建立 OAuth Client ID（一次性，約 15 分鐘） |

### 實作方式

**1. Google Cloud Console 設定**（一次性）：

```
建立 GCP 專案 → 啟用 Google Identity Services
→ 建立 OAuth 2.0 Client ID（Web application）
→ 設定 Authorized JavaScript origins
→ 取得 Client ID
```

**2. Vercel 環境變數：**
```
GOOGLE_CLIENT_ID=xxxxxxxxxxxx.apps.googleusercontent.com
ALLOWED_EMAILS=user1@gmail.com,user2@company.com,user3@company.com
```

**3. 後端驗證**：使用 `google-auth-library` 驗證 Token 真偽，取出 email 後比對環境變數中的白名單。

### 管理方式

| 操作 | 怎麼做 |
|------|--------|
| 加人 | Vercel Dashboard → 在 `ALLOWED_EMAILS` 加入新 email → Redeploy |
| 移除人 | Vercel Dashboard → 從 `ALLOWED_EMAILS` 刪除該 email → Redeploy |
| 查看誰有權限 | 查看 `ALLOWED_EMAILS` 環境變數 |

### 好處

- 不需要資料庫
- 使用者不用記密碼（用自己的 Google 帳號）
- 可以辨識是誰在使用
- Google 負責身份驗證的安全性
- Token 有過期機制（約 1 小時），比靜態密碼安全

### 限制

- 每次加人/移除人都要改環境變數 + 重新部署
- 沒有角色區分（所有人權限一樣）
- 無法追蹤個人用量（除非另外加 log）
- 白名單變長時環境變數會比較難管理
- 需要設定 Google Cloud Console（一次性）

### 適用時機

團隊 3–15 人、人員異動不頻繁、使用者都有 Google 帳號、需要知道「誰」在用但不需要細緻的權限管理。

---

## 方案三：Google 登入 + Supabase 使用者管理

**完整角色管理 · 約半天實作**

### 概念

使用者用 Google 帳號登入，身份驗證和使用者管理都交給 Supabase。管理者可以在 Supabase Dashboard 即時管理使用者、角色、停用權限，不需要重新部署。

### 運作流程

```
使用者打開網站 → 顯示 Google 登入按鈕
→ 透過 Supabase Auth 導向 Google OAuth
→ 登入成功，Supabase 發出 JWT
→ 前端將 JWT 帶在 API 請求 header
→ 後端驗證 JWT + 查詢資料庫確認角色 → 通過才放行
```

### 需要什麼

| 項目 | 說明 |
|------|------|
| 外部服務 | Google Cloud Console（免費）+ Supabase（免費額度 50,000 MAU） |
| 資料庫 | Supabase PostgreSQL（profiles table） |
| 環境變數 | `GOOGLE_CLIENT_ID` + `SUPABASE_URL` + `SUPABASE_ANON_KEY` + `SUPABASE_SERVICE_ROLE_KEY` |
| 前端改動 | Supabase Auth UI + AuthContext + Protected Route |
| 後端改動 | 每個 API function 驗證 Supabase JWT + 查詢角色 |
| npm 套件 | `@supabase/supabase-js` |

### 角色設計（建議）

| 角色 | 可以做什麼 | 適用對象 |
|------|-----------|---------|
| `admin` | 所有功能 + 未來的管理頁面 | 管理者 |
| `user` | 所有操作功能 | 一般使用者 |
| `viewer` | 只能瀏覽，不能執行付費操作（省 API 費用） | 主管、需要看成果的人 |

### 資料庫結構

```
email                      role      is_active
─────────────────────────────────────────────────
admin@company.com          admin     true
colleague1@company.com     user      true
colleague2@company.com     viewer    true
ex-employee@company.com    user      false    ← 已停用
```

### 管理方式

| 操作 | 怎麼做 |
|------|--------|
| 加人 | 使用者用 Google 登入 → 自動出現在 profiles table → 管理者啟用 |
| 移除人 | Supabase Dashboard → 把 `is_active` 改 false，**即時生效，不需要重新部署** |
| 改角色 | Supabase Dashboard → 修改 `role` 欄位，即時生效 |
| 查看所有使用者 | Supabase Dashboard → Table Editor → profiles |

### 好處

- 加人/移除人/改角色即時生效，不需要重新部署
- 可以區分角色（admin / user / viewer）
- 可以追蹤個人用量
- Supabase 免費額度充裕（50,000 MAU）
- 未來可擴展（用量統計、計費、Admin 頁面）
- Google 登入體驗佳，使用者不用記密碼

### 限制

- 多一個外部依賴（Supabase）
- 初始設定較複雜（Google OAuth + Supabase + DB）
- 需要處理 JWT refresh 邏輯
- 新使用者首次登入後可能需要管理者手動啟用

### 適用時機

團隊 10+ 人或人員頻繁變動、需要角色區分、想追蹤個人用量、未來可能開放給更多部門或外部合作夥伴。

---

## 更多驗證方式

### 自建帳號管理（不依賴 Google）

如果不想讓使用者透過 Google 登入，Supabase Auth 也支援以下幾種自建帳號的方式。後端驗證邏輯都和方案三完全相同（驗 JWT → 查 profiles table），差別只在「使用者怎麼登入」。

#### Email + 密碼（自建帳密）

管理者在 Supabase 手動建立帳號（email + 密碼），把帳密發給使用者。完全不經過 Google，你掌控所有帳號。適合需要最高控制權的場景。

#### Invite Code 邀請碼（邀請制）

管理者產生邀請碼 → 發給新使用者 → 使用者用邀請碼 + 自訂密碼完成註冊 → 邀請碼用過即失效。嚴格控制誰能註冊，不需要管理者逐一建帳。

#### Magic Link 魔法連結（無密碼）

使用者輸入 email → 信箱收到一封登入連結 → 點連結直接登入，不需要密碼。Supabase Auth 原生支援，體驗接近 Google 登入。

#### Email OTP 驗證碼（無密碼）

使用者輸入 email → 信箱收到 6 位數驗證碼 → 輸入驗證碼登入。比 Magic Link 更快（不用切到信箱點連結），Supabase Auth 原生支援。

> **關鍵優勢：** 以上所有方式都基於 Supabase Auth，後端驗證邏輯完全一樣（驗 JWT → 查 profiles → 確認角色）。你可以同時開啟多種登入方式，讓使用者自己選最方便的。未來要加新的登入方式，只要在 Supabase Dashboard 開一個 Provider，不需要改後端程式碼。

---

## 平台層級保護（零程式碼）

除了在應用內實作驗證，也可以在「請求到達你的 App 之前」就擋住未授權的人，完全不需要修改程式碼。

### Vercel Password Protection

Vercel Pro Plan（$20/月）→ Settings → Password Protection → 開啟。整個網站（含所有 API routes）都被保護，訪客會看到 Vercel 原生的密碼頁。本質上和方案一類似，但零程式碼。

### Cloudflare Access（Zero Trust）

把網站放到 Cloudflare 後面，在網路層攔截未授權請求。可設定 email 白名單、Google 登入、甚至企業 SSO。免費額度 50 人。需要把 DNS 搬到 Cloudflare。

**適用時機：** 不想改程式碼、或者想在應用層驗證之外再多一層防護。Vercel Password Protection 適合臨時保護；Cloudflare Access 適合長期、安全性要求高的場景。

---

## 其他第三方登入選項

如果選擇 Supabase Auth 作為驗證底層，除了 Google 之外，還可以開啟以下第三方登入。在 Supabase Dashboard 開啟 Provider 即可，後端驗證邏輯完全不用改。

| 登入方式 | 適用情境 |
|---------|---------|
| **LINE Login** | 台灣團隊幾乎人人有 LINE，登入門檻最低 |
| **Microsoft / Azure AD** | 公司使用 Microsoft 365 時，員工可以用公司帳號直接登入 |
| **GitHub** | 技術團隊適用，開發者都有 GitHub 帳號 |
| **Facebook** | 消費端使用者常用，但企業內部較少用 |

> **可以混搭：** Supabase Auth 允許同時開啟多個 Provider。例如同時開啟 Google + LINE，使用者在登入頁自己選。所有 Provider 登入後都拿到同一種 JWT，後端驗證邏輯不變。

---

## 三方案總覽比較

| | 方案一：靜態密碼 | 方案二：Google + 白名單 | 方案三：Google + Supabase |
|---|---|---|---|
| **實作時間** | ~30 分鐘 | ~3–4 小時 | ~半天 |
| **外部服務** | 無 | Google Cloud | Google Cloud + Supabase |
| **資料庫** | 不需要 | 不需要 | 需要（Supabase 免費） |
| **辨識使用者** | 不能 | 能（email） | 能（email + profile） |
| **角色管理** | 無 | 無 | 有（自訂角色） |
| **加人/移除人** | 改密碼通知所有人 | 改環境變數 + 重新部署 | 改 DB，即時生效 |
| **追蹤用量** | 不能 | 不能 | 能 |
| **安全性** | 低 | 中 | 高 |
| **維護成本** | 極低 | 低 | 中 |
| **適合人數** | 1–3 人 | 3–15 人 | 10+ 人 |
| **擴展性** | 低 | 低 | 高 |

---

## 建議的升級路徑

如果不確定要選哪個，可以分階段升級。每一階段都可以獨立運作，往下一階段升級時直接替換驗證邏輯，不影響其他功能。

| 階段 | 方案 | 說明 |
|------|------|------|
| Phase 1 · 現在 | 靜態密碼 | 先保護起來，30 分鐘上線。適合開發和內部初期測試階段。 |
| Phase 2 · 試用期 | Google + 白名單 | 電商部門開始試用時升級，知道誰在用，不需要額外基礎設施。 |
| Phase 3 · 正式推廣 | Google + Supabase | 需要管理更多人、追蹤用量、區分角色時再升級到完整方案。 |

---

## 附註

- 以上所有方案的 API Key 都維持存在 Vercel 環境變數、只在 server-side 使用，不會暴露到前端
- Google Cloud Console 的 OAuth 設定是免費的，不會產生費用
- Supabase 免費方案包含 50,000 月活躍使用者、500MB 資料庫，對內部工具綽綽有餘
