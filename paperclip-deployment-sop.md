# Paperclip 部署 SOP(FDE 用)

> **建立日期**:2026-06-07
> **對象**:在 macOS 上部署 Paperclip 當「AI agent 駕駛艙」的 FDE
> **適用**:本機 demo 沙盒 / 客戶端部署。實測於 Node v25 + macOS(系統 node 在 /usr/local)
> **架構**:Paperclip(駕駛艙)+ Claude Code(引擎)+ 客製 skills

---

## 0. 心智模型(30 秒)

```
Company（= 一個客戶）
  ├── Agents（員工:role / budget / skills / permissions),用 Org chart + Teams 組織
  └── Skills library（★ company 層級:每間公司各有一套)
```
- 1 client = 1 Paperclip company。skills 是 company-scoped → 天然支援「每客戶客製」。
- agent 跑在 **Claude Code adapter**(claude_local)上,Paperclip 負責治理/展示。

---

## 1. 前置需求

- **Node 20+**(有 corepack,Node 自帶)
- **git**
- ⚠️ macOS 注意:若 `node` 裝在 `/usr/local`(系統/Homebrew),corepack 建 shim 會權限不足 → 用使用者目錄(見下)

---

## 2. 安裝(免 sudo、免全域污染)

```bash
git clone --depth 1 https://github.com/paperclipai/paperclip.git paperclip
cd paperclip

# 用 corepack 提供指定版 pnpm,shim 裝到使用者目錄(避開 /usr/local 權限問題)
mkdir -p "$HOME/.corepack-bin"
corepack enable --install-directory "$HOME/.corepack-bin" pnpm
export PATH="$HOME/.corepack-bin:$PATH"   # 建議加進 ~/.zshrc 持久化

pnpm install   # monorepo 依賴,約 40 秒
```

## 3. 設定 + 啟動

```bash
pnpm paperclipai onboard --yes   # 建 config.json + Agent JWT(loopback 安全預設)
pnpm paperclipai doctor          # 驗證,目標:全綠(8 passed)
pnpm dev                         # 啟動 → http://127.0.0.1:3100
```

- `onboard --yes` 會**立即啟動服務**(常駐),屬正常。
- doctor 全綠代表:config 有效、Agent JWT、secrets、storage、DB、auth mode 都 OK。
- ⚠️ **沒跑 onboard → 後續 agent 建不起來**(會看到 `Agent JWT missing`)。

## 4. 建立 Company → Agent → Task(UI:127.0.0.1:3100)

引導式精靈四步:
1. **Company** — 填客戶/公司名 + mission(這是 agents 服務的組織)
2. **Agent** — 取名、選 **Claude Code** adapter、Model 預設
3. **Task** — 給一個小起手任務
4. **Launch** — Create & Open Task → 喚醒 agent 實際執行

## 5. 加入客製 Skill(關鍵)

Paperclip 有自己的 **company skill library**,格式跟 Claude Code 一樣(`SKILL.md`)。流程:

```bash
CID=<company-id>      # 從 dashboard URL 或 server log 取得
AID=<agent-id>        # pnpm paperclipai agent list --company-id $CID

# 1) import 進 company library
pnpm paperclipai skills import "/path/to/your-skill" --company-id $CID

# 2) attach 到 agent
pnpm paperclipai agent skills:sync $AID --desired-skills your-skill-name

# 3) 驗證(看 attachedAgents=1)
pnpm paperclipai skills list --company-id $CID
```

### Skill 兩種來源並存
| 來源 | 行為 | 適合放 |
|---|---|---|
| **Paperclip 管理**(import) | 要手動 import + attach;UI 可見、company-scoped、可交付 | **客製職能 skill** |
| **`~/.claude/skills/`** | Paperclip 自動偵測為「External / 唯讀」;agent 也看得到 | 通用工具 skill(xlsx 等) |

→ 不是全自動也不是全手動:**客製的走 import,通用的留 ~/.claude/skills 共用。**

---

## 6. 踩坑與解法(實戰記錄)

| 症狀 | 原因 | 解法 |
|---|---|---|
| `corepack ... EACCES /usr/local/bin` | 系統 node 在 /usr/local,無權寫 shim | `corepack enable --install-directory "$HOME/.corepack-bin" pnpm` |
| adapter 探針 `Claude hello probe failed` | `~/.claude` 的 SessionStart hook 噴 JSON,探針太龜毛 | **裝飾性,可忽略**;以實際跑得動為準 |
| agent 建不起來 / `Agent JWT missing` | 沒跑 onboard,缺 config.json | `pnpm paperclipai onboard --yes` |
| `Company ID is required` | skills/agent 指令需要 company 範圍 | 加 `--company-id <id>` |
| Launch 後 dashboard 顯示 0 agents | 上述 JWT 缺口導致 agent 沒建成 | 修好 onboard 後重建 agent |

---

## 7. Per-client 部署模型

```
你的機器 = demo 沙盒(一套共用,放範例 company 給客戶看)
客戶機器 = 真正部署(各自 config / 資料 / 金鑰 → 天然隔離,不需多 profile)
對應鏈:Paperclip company ↔ Obsidian「顧問/客戶X」資料夾 ↔ 該 company 的 skill library
```

---

## 速查指令

```bash
# 啟動
export PATH="$HOME/.corepack-bin:$PATH" && cd paperclip && pnpm dev

# 診斷
pnpm paperclipai doctor

# Skill:import → attach → 驗證
pnpm paperclipai skills import <path> --company-id <CID>
pnpm paperclipai agent skills:sync <AID> --desired-skills <name>
pnpm paperclipai skills list --company-id <CID>
```
