# Google 登入 + Sheets API 設定指南

> 建立日期：2026-04-05
> 目的：說明內部工具串接 Google Login（白名單制）與 Google Sheets API 時，需要在 Google Cloud Console 和 Vercel 做哪些設定，以及兩者之間的關係。

---

## 背景

我們的營收儀表板需要兩種 Google 服務：

1. **Google 登入**：讓使用者用公司 Google 帳號登入，搭配 email 白名單控管誰能存取
2. **Google Sheets API**：讓伺服器直接讀寫 Google Sheet 中的營收資料

這兩個功能用了**兩套獨立的認證機制**，各自有不同的設定方式。

---

## 整體架構

```
Google Cloud Console
├── OAuth Client ID（前端登入用）
│   → 使用者在瀏覽器登入 Google
│   → 取得 ID Token（證明「我是誰」）
│   → 後端驗證 + 白名單比對
│
└── Service Account（後端 API 用）
    → 伺服器用 private key 取得 access token
    → 直接呼叫 Google Sheets API 讀寫資料
    → 不需要使用者介入
```

---

## 1. OAuth Client ID — 使用者登入

### 是什麼？

OAuth Client ID 是讓你的網站可以顯示「Sign in with Google」按鈕的憑證。使用者點擊後，Google 會回傳一個 ID Token（JWT 格式），裡面包含使用者的 email、名字、頭像。

### 在 Google Cloud Console 怎麼設定？

1. 進入 [Google Cloud Console](https://console.cloud.google.com/)
2. 選擇你的專案（或建立新專案）
3. 前往 **APIs & Services → Credentials**
4. 點 **Create Credentials → OAuth client ID**
5. Application type 選 **Web application**
6. 在 **Authorized JavaScript origins** 加入：
   - `http://localhost:3001`（本地開發）
   - `https://你的正式網域`（production）
7. 建立後會拿到一個 **Client ID**（格式：`xxxxxxx.apps.googleusercontent.com`）

### 重要特性

| 項目 | 說明 |
|------|------|
| 是否為 secret | ❌ 不是。Client ID 會出現在前端程式碼中，本來就是公開的 |
| 後端也需要嗎？ | ✅ 後端驗證 ID Token 時，需要知道 Client ID 來確認 token 是發給你的應用 |
| 白名單怎麼做？ | 白名單不在 Google 設定，是我們自己在後端用 `ALLOWED_EMAILS` 環境變數控制 |

---

## 2. Service Account — 伺服器讀寫 Sheet

### 是什麼？

Service Account 是一個「機器人帳號」，有自己的 email 地址和 private key。你的伺服器用它來代表應用程式存取 Google API，完全不需要使用者登入或授權。

### 在 Google Cloud Console 怎麼設定？

1. 前往 **IAM & Admin → Service Accounts**
2. 點 **Create Service Account**
3. 取個名字（例如 `revenue-dashboard`），建立
4. 進入該 Service Account → **Keys → Add Key → Create new key**
5. 選 **JSON**，下載金鑰檔案

下載的 JSON 大概長這樣：
```json
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "revenue-dashboard@your-project.iam.gserviceaccount.com",
  "client_id": "...",
  ...
}
```

### 還要做什麼？

**啟用 Google Sheets API**：
- 前往 **APIs & Services → Library**
- 搜尋「Google Sheets API」→ 點 **Enable**

**把 Service Account 加為 Sheet 編輯者**：
- 打開你的 Google Sheet
- 點右上角「共用」
- 貼上 Service Account 的 email（JSON 裡的 `client_email`）
- 權限選「編輯者」

### 重要特性

| 項目 | 說明 |
|------|------|
| 是否為 secret | ✅ 是！private key 等同密碼，外洩就等於任何人都能讀寫你的 Sheet |
| 跟使用者有關嗎？ | ❌ 完全無關。Service Account 是伺服器對伺服器的認證 |
| 需要使用者授權嗎？ | ❌ 不需要。只要 Sheet 有共用給 Service Account email 就能存取 |

---

## 3. 兩者的關係

OAuth Client ID 和 Service Account 是**完全獨立**的兩套機制，互不依賴：

| | OAuth Client ID | Service Account |
|---|---|---|
| 誰在用 | 使用者（前端瀏覽器） | 伺服器（後端 API） |
| 做什麼 | 驗證「這個人是誰」 | 讀寫 Google Sheet |
| 是否 secret | ❌ 公開的 | ✅ 絕對保密 |
| 在哪裡設定 | Credentials → OAuth | IAM → Service Accounts |

它們搭配起來的效果：

```
使用者 → Google 登入（OAuth Client ID）→ 白名單驗證 → 允許操作
                                                          ↓
                                        伺服器用 Service Account → 讀寫 Sheet
```

**OAuth 確保只有白名單的人能操作，Service Account 確保操作能實際寫進 Sheet。**

---

## 4. Vercel 環境變數設定

在 Vercel Dashboard → 你的專案 → **Settings → Environment Variables** 設定以下變數：

| 變數名 | 值 | 說明 |
|--------|-----|------|
| `GOOGLE_SERVICE_ACCOUNT_KEY` | 整份 JSON 金鑰內容 | 下載的 JSON 檔全文貼上 |
| `GOOGLE_SHEET_ID` | Sheet 網址中的 ID | `/d/` 和 `/edit` 之間那段字串 |
| `GOOGLE_CLIENT_ID` | OAuth Client ID | 格式：`xxxxx.apps.googleusercontent.com` |
| `ALLOWED_EMAILS` | email 清單 | 逗號分隔，例如 `alice@gmail.com,bob@gmail.com` |

### 常見問題

**Q：`GOOGLE_SERVICE_ACCOUNT_KEY` 太長，Vercel 貼得進去嗎？**
A：可以。Vercel 支援多行值。直接把整份 JSON 貼上即可。

**Q：`private_key` 中的 `\n` 會不會有問題？**
A：有時候會。如果部署後 API 報錯 `error:0909006C:PEM routines`，通常是 `\n` 被 escape 成 `\\n`。確認 Vercel 環境變數中的換行是真正的換行。

**Q：本地開發怎麼設定？**
A：在專案根目錄建立 `.env.local`，貼上同樣的變數。這個檔案已被 `.gitignore` 排除，不會被 commit。

**Q：白名單要改怎麼辦？**
A：修改 Vercel 上的 `ALLOWED_EMAILS` 環境變數，然後 **Redeploy** 即可生效。不需要改程式碼。

---

## 5. Google Cloud Console 設定清單

總結所有需要在 Google Cloud Console 做的事：

- [ ] 建立專案（或使用既有專案）
- [ ] 啟用 Google Sheets API
- [ ] 建立 OAuth Client ID（Web application），加入授權網域
- [ ] 建立 Service Account，下載 JSON 金鑰
- [ ] 在 Google Sheet 共用設定中，加入 Service Account email 為編輯者
