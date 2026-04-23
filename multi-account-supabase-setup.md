# 多個 Supabase 帳號共存的開發環境設定

> 建立日期：2026-04-23
> 目的：整理在同一台 Mac 上管理**多個 Supabase 帳號**的完整設定方案。涵蓋 `~/.zshrc` 配置、Personal Access Token、`supabase link`、terminal 自動切換等實作細節。

---

## 為什麼會有多個 Supabase 帳號

Supabase 免費方案每個帳號**最多 2 個 project**。當手上同時有 3 個以上 project 時，只能申請多個帳號來分配。例如：

```
Account A：DS Generator EG + Mailchimp Dashboard
Account B：營收儀表板 + WiFi Regulation Checker
Account C：MKT ShortLink
```

每個帳號都需要能用 CLI 操作（gen types、migration、db push 等），但 **Supabase CLI 一次只認一個身份** — 這就是多帳號管理要解的問題。

---

## 預備知識：`~/.zshrc` 是什麼

macOS Catalina 之後預設 shell 是 **Zsh**，每次開 terminal 時會自動讀取 `~/.zshrc` 跑一次。這個檔案放：

| 類別 | 範例 |
|---|---|
| 環境變數 | `export NODE_ENV=development` |
| PATH 擴充 | `export PATH="$HOME/.local/bin:$PATH"` |
| 指令別名 | `alias gs='git status'` |
| 自訂函式 | `myfunc() { ... }` |
| 工具載入 | `source /opt/homebrew/opt/nvm/nvm.sh` |

**生效時機**：改完 `~/.zshrc` 要**開新 terminal** 或 `source ~/.zshrc` 才會套用到當前 session。

`.` 開頭是隱藏檔，`ls` 看不到，要 `ls -a`。

---

## Supabase CLI 的身份驗證機制

每次 `supabase` 指令執行時，它檢查身份的順序：

```
1. 環境變數  SUPABASE_ACCESS_TOKEN        ← 最優先
2. 檔案      ~/.supabase/access-token      ← 其次
3. 都沒有 → 提示「請先 supabase login」
```

- `supabase login` 會透過 OAuth 把 token 寫入 `~/.supabase/access-token`（一次一個帳號）
- `supabase logout` 會刪除那個檔案
- **關鍵洞察**：使用 `SUPABASE_ACCESS_TOKEN` **環境變數**時，**不同 terminal 可以有不同身份**，因為環境變數是 per-shell 的

這是多帳號共存的核心機制。

---

## 方案比較

四種可行做法，各有取捨：

| 方案 | 切換方式 | 優點 | 缺點 | 適用 |
|---|---|---|---|---|
| **1. Shell function** | 打 `sb-a` / `sb-b` | 一行切換，零依賴 | Token 明文寫 dotfile | 切換不頻繁、dotfile 不上 git |
| **2. per-project `.env.local`** | 進 repo 後 source 檔案 | 身份綁專案 | 要手動載入 | 多 repo、固定對應 |
| **3. direnv** | `cd` 自動切換 | 完全自動 | 多裝一個工具 | 重度開發者 |
| **4. macOS Keychain** | 搭配 function | Token 存系統鑰匙圈 | 略多步驟 | Dotfile 備份到雲端 |

**本文採用：方案 1 + `chpwd` hook 擴充**（結合方案 1 的簡單和方案 3 的自動切換），完全用原生 Zsh 功能，不裝額外工具。

---

## 完整設定步驟

### Step 1：取得 Personal Access Tokens

對每個 Supabase 帳號**各做一次**：

1. 開瀏覽器（建議用無痕視窗避免 session 衝突）登入該帳號
2. 到 https://supabase.com/dashboard/account/tokens
3. 按 **Generate new token**
4. 命名建議：`cli-<帳號描述>`（例如 `cli-work`、`cli-personal`、`cli-marketing`）
5. 複製 `sbp_xxxxxxxxxxxxxxxx...`（50+ 字元）— **只顯示一次**，關掉就看不到

假設你拿到 3 個：
```
cli-work      sbp_aaaa...
cli-personal  sbp_bbbb...
cli-marketing sbp_cccc...
```

---

### Step 2：設定 `~/.zshrc`

打開 `~/.zshrc`（用 VS Code、TextEdit、nano 都可以）：

```bash
# VS Code 打開
code ~/.zshrc

# 或 macOS 內建 TextEdit
open -e ~/.zshrc
```

在**檔案最下方**貼上這整段（記得把 `sbp_REPLACE_WITH_...` 替換成你真的 token，帳號名稱也改成你自己取的）：

```bash
# ============================================================
# Supabase — 多帳號切換（Personal Access Tokens）
# ------------------------------------------------------------
# Token 來自 https://supabase.com/dashboard/account/tokens
#
#   sb-a              # 切到 cli-work
#   sb-b              # 切到 cli-personal
#   sb-c              # 切到 cli-marketing
#   sb-who            # 查當前身份 + projects list
#   sb-clear          # 清空當前 session 的 token
#
# 切換是 per-shell 的：不同 terminal 視窗可以切到不同帳號。
# ============================================================

# Helper：設 terminal 分頁標題。ESC ]0; ... BEL 是 xterm / Terminal.app / iTerm2
# 都支援的視窗 + 分頁標題控制碼。
_sb_set_title() { print -Pn "\e]0;[$1] %n — %~\a"; }

sb-a() {
  export SUPABASE_ACCESS_TOKEN="sbp_REPLACE_WITH_cli-work_TOKEN"
  _sb_set_title "cli-work"
  echo "→ Supabase switched to: cli-work"
}

sb-b() {
  export SUPABASE_ACCESS_TOKEN="sbp_REPLACE_WITH_cli-personal_TOKEN"
  _sb_set_title "cli-personal"
  echo "→ Supabase switched to: cli-personal"
}

sb-c() {
  export SUPABASE_ACCESS_TOKEN="sbp_REPLACE_WITH_cli-marketing_TOKEN"
  _sb_set_title "cli-marketing"
  echo "→ Supabase switched to: cli-marketing"
}

# 顯示當前使用哪個 token + 列出該帳號下的 projects
sb-who() {
  if [ -z "$SUPABASE_ACCESS_TOKEN" ]; then
    echo "→ No SUPABASE_ACCESS_TOKEN in shell. Falling back to ~/.supabase/access-token (supabase login)."
  else
    echo "→ Active token: ${SUPABASE_ACCESS_TOKEN:0:12}…${SUPABASE_ACCESS_TOKEN: -4}"
  fi
  echo "→ Projects visible to this identity:"
  supabase projects list 2>&1 | head -20
}

# 清掉目前 shell 的 token（不影響 ~/.supabase/access-token）
sb-clear() {
  unset SUPABASE_ACCESS_TOKEN
  print -Pn "\e]0;%n — %~\a"     # 清掉 [cli-xxx] 標題前綴
  echo "→ Cleared SUPABASE_ACCESS_TOKEN from this shell."
}

# ------------------------------------------------------------
# 自動根據當前資料夾切換 Supabase 身份（chpwd hook）
# ------------------------------------------------------------
_sb_auto_switch() {
  case "$PWD" in
    # cli-work — 把這個帳號下的 repo 路徑模式寫在這
    */work-project-A|*/work-project-A/*|*/work-project-B|*/work-project-B/*)
      export SUPABASE_ACCESS_TOKEN="sbp_REPLACE_WITH_cli-work_TOKEN"
      _sb_set_title "cli-work"
      ;;
    # cli-personal
    */personal-project-A|*/personal-project-A/*)
      export SUPABASE_ACCESS_TOKEN="sbp_REPLACE_WITH_cli-personal_TOKEN"
      _sb_set_title "cli-personal"
      ;;
    # cli-marketing
    */marketing-project|*/marketing-project/*)
      export SUPABASE_ACCESS_TOKEN="sbp_REPLACE_WITH_cli-marketing_TOKEN"
      _sb_set_title "cli-marketing"
      ;;
  esac
}

autoload -Uz add-zsh-hook
add-zsh-hook chpwd _sb_auto_switch
_sb_auto_switch  # 開 terminal 時立刻對當前資料夾跑一次
```

存檔後：

```bash
source ~/.zshrc    # 或直接開新 terminal 分頁
```

---

### Step 3：驗證

```bash
sb-a                         # 切到 Account A
sb-who                       # 會列出該帳號可見的所有 projects
```

如果看到正確的 projects 清單，就設定成功。

切到其他帳號：
```bash
sb-b
sb-who
```

---

### Step 4：綁定每個 repo 到對應的 project

這步驟讓 repo 知道自己對應哪個 Supabase project，以後下指令可以用 `--linked` 簡寫，不用每次記 project ID。

對每個 repo 做一次：

```bash
cd <repo 路徑>
sb-a                                                    # 切到 repo 所屬的帳號
supabase link --project-ref <project-reference-id>
```

`project-reference-id` 是 20 字元字串，從 `supabase projects list` 的 **REFERENCE ID** 欄位複製。

**記得把 `supabase/.temp/` 加進 `.gitignore`**（link 會在 repo 建立這個資料夾暫存 link 狀態，不應 commit）：

```bash
echo "" >> .gitignore
echo "# Supabase CLI (local link cache — not for commit)" >> .gitignore
echo "supabase/.temp/" >> .gitignore
```

---

## 使用方式

### 一次切換一個 terminal

```bash
sb-a                                         # 切帳號
supabase projects list                       # 看該帳號能看到的 projects
supabase gen types typescript --linked       # 對當前 repo 綁定的 project 產生 types
```

### 三個 terminal 同時用三個帳號

每個 terminal 視窗/分頁**獨立**，環境變數互不影響：

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Terminal 1       │  │ Terminal 2       │  │ Terminal 3       │
│                  │  │                  │  │                  │
│ $ sb-a           │  │ $ sb-b           │  │ $ sb-c           │
│   → cli-work     │  │   → cli-personal │  │   → cli-marketing│
│                  │  │                  │  │                  │
│ TOKEN=sbp_aa...  │  │ TOKEN=sbp_bb...  │  │ TOKEN=sbp_cc...  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
      獨立                   獨立                   獨立
```

### 進 repo 自動切換

如果你在 Step 2 有設定 `_sb_auto_switch`（chpwd hook），那以後 **`cd` 進任何一個 linked repo，自動切到對的帳號**：

```bash
cd ~/Projects/work-project-A     # → 自動切到 cli-work，標題變 [cli-work]
cd ~/Projects/marketing-project  # → 自動切到 cli-marketing，標題變 [cli-marketing]
cd ~                             # → 身份不變（不在 linked repo）
```

Terminal 分頁標題會即時顯示當前身份，不會搞錯。

### 常用指令

| 動作 | 指令 |
|---|---|
| 切帳號 | `sb-a` / `sb-b` / `sb-c` |
| 查當前身份 | `sb-who` |
| 清當前 session 身份 | `sb-clear` |
| 列出可見 projects | `supabase projects list` |
| 產生 DB types | `supabase gen types typescript --linked > types.ts` |
| 查 migrations | `supabase migration list` |

---

## 附錄：免費方案 7 天閒置會被 pause

Supabase 免費方案如果 **7 天內沒有任何 API 請求**，project 會進入 paused 狀態。被 pause 的 project 無法操作，要到 Dashboard 按 Restore 才能恢復（通常幾分鐘）。

**什麼算「活動」**：任何 API call（REST、GraphQL、Realtime）、auth 事件、DB query 都算。

### 防止 pause 的三種方案

**方案 A：Vercel Cron 每日打一次 API**

如果 project 已有 Vercel 部署，`vercel.json` 加 cron：

```json
{
  "crons": [{ "path": "/api/ping", "schedule": "0 0 */3 * *" }]
}
```

`/api/ping` 寫一個最小的 DB query（例如 `SELECT 1`），每 3 天自動打一次就保活。

**方案 B：GitHub Actions 集中管理多 project**

用一個 repo 跑 cron workflow，對多個 project 依序打 API：

```yaml
# .github/workflows/keepalive.yml
on:
  schedule:
    - cron: '0 0 */3 * *'
jobs:
  ping:
    steps:
      - run: |
          curl "https://$P1.supabase.co/rest/v1/..." -H "apikey: $KEY1"
          curl "https://$P2.supabase.co/rest/v1/..." -H "apikey: $KEY2"
```

**方案 C：Supabase 內建 pg_cron（最乾淨）**

免費方案也支援 `pg_cron` 擴充，直接在 DB 排程：

```sql
-- 在 Supabase SQL Editor 執行
create extension if not exists pg_cron;

select cron.schedule(
  'keepalive',
  '0 0 */3 * *',         -- 每 3 天跑一次
  $$ select 1; $$        -- 最小 query
);
```

完全不用外部服務，每個 project 各自跑一次這個 SQL 就好。

### 推薦

| Project 性質 | 推薦方案 |
|---|---|
| 有 Vercel 部署 + 日常有活動（如每日 cron） | **不用做**（已有活動） |
| 靜態/低流量專案 | **方案 C**（pg_cron）最省事 |
| 想集中管理多個 | **方案 B**（GitHub Actions） |

---

## 進階：提升安全性

如果你的 dotfile（`~/.zshrc`）會備份到 git / 雲端，把 token 明文寫在裡面不夠安全。這時改用 **macOS Keychain**：

```bash
# 存 token 到系統鑰匙圈
security add-generic-password -a "$USER" -s "sb-work" -w "sbp_aaaa..."
security add-generic-password -a "$USER" -s "sb-personal" -w "sbp_bbbb..."
security add-generic-password -a "$USER" -s "sb-marketing" -w "sbp_cccc..."

# 改寫 ~/.zshrc 的 function
sb-a() {
  export SUPABASE_ACCESS_TOKEN=$(security find-generic-password -a "$USER" -s "sb-work" -w)
  _sb_set_title "cli-work"
  echo "→ cli-work"
}
# sb-b / sb-c 同樣改寫
```

這樣 `~/.zshrc` 不含任何 token 字串，安全很多。

---

## 總結

| 關鍵要點 | 說明 |
|---|---|
| 多帳號靠 `SUPABASE_ACCESS_TOKEN` 環境變數 | 每個 terminal 獨立 |
| Personal Access Token 到 dashboard 拿 | 一次只顯示，務必存好 |
| `supabase link` 綁定 repo | 以後用 `--linked` 簡寫 |
| `supabase/.temp/` 要 gitignore | 避免誤 commit 本機 cache |
| `chpwd` hook 自動切換 | 進 repo 不用手動切 |
| Terminal title 顯示身份 | 多分頁時避免搞錯 |
| 7 天閒置會 pause | 用 pg_cron 或 Vercel cron 保活 |

這套設定大約 15 分鐘可以完成，之後多帳號切換完全無感。
