# 套件安裝方式 & Skill/MCP 安裝避坑指南

> **建立日期**:2026-06-03
> **對象**:內部技術團隊／同事
> **目的**:建立共同認知,減少 `pip` / `uv` / `npm` / `npx` / `curl|bash` 混用造成的踩坑
> **適用環境**:macOS (Apple Silicon),Python 用 `uv`、Node 用 `nvm`

---

## TL;DR(先記這三句)

1. **安裝管道跟著「語言」走,不是跟著「它叫 skill 還是 tool」走。** JS/TS → npm;Python → pip/uv。兩者互不相通。
2. **`uvx ≈ npx`、`uv tool install ≈ npm i -g`、`uv add ≈ npm install`。** 一一對應。
3. **純知識型 skill(`SKILL.md`)兩個都不用,只是把檔案放到對的資料夾。**

---

## Part 1 — 安裝方式速查對照

| 你想做的事 | Python (uv) | Node (npm) |
|---|---|---|
| 裝**函式庫**進專案(程式碼會 import) | `uv add`(記錄到 pyproject)/ `uv pip install` | `npm install` |
| 裝**常駐 CLI 工具**(到處都要叫) | `uv tool install` | `npm install -g` |
| **臨時跑一次**(不永久安裝) | `uvx`(= `uv tool run`) | `npx` |
| 管**語言本身版本** | uv 內建管 Python | `nvm` |
| 一鍵 bootstrap 腳本 | `curl \| bash` | `curl \| bash` |

---

## Part 2 — uv 三兄弟的差別(最常混)

分辨關鍵只有兩問:**① 要 import 它,還是叫它當指令?② 長期裝著,還是跑一次?**

| | `uv pip install` | `uv tool install` | `uvx` |
|---|---|---|---|
| 目的 | 裝**函式庫**進環境 | 裝**CLI 工具**(常駐) | **跑一次** CLI(不常駐) |
| 裝去哪 | 當前 venv / `.venv` | 每個工具**獨立隔離環境** | 臨時快取,**不進 PATH** |
| 指令進 PATH | 否 | 是 | 否 |
| npm 類比 | `npm install` | `npm install -g` | `npx` |
| 典型場景 | `import requests` 寫程式 | 天天用的 `hermes`、`ruff` | 試用 / 一次性 |

**決策流程**

```
要在 .py 裡 import 它嗎?
  ├─ 是 ───────────────→ uv add(或 uv pip install)   【函式庫】
  └─ 否(它是一個指令)
        └─ 會常常用嗎?
              ├─ 會 ────→ uv tool install              【常駐工具】
              └─ 偶爾 ──→ uvx                          【一次性】
```

> 補充:`uv add` 會把依賴寫進 `pyproject.toml` + 鎖進 `uv.lock`(可重現);`uv pip install` 比較像「臨時裝、不記錄」。**開專案優先 `uv add`。**

---

## Part 3 — 為何 Python 強調隔離,Node 卻還好?

| | Python | Node |
|---|---|---|
| 全域依賴怎麼放 | **扁平、共用**一個 site-packages | **巢狀**,每個套件自帶 `node_modules` |
| 後果 | 兩個工具要不同版本的同一個 lib → **直接打架** | 各帶各的,**很少打架** |

- **Python 的全域 `pip install` 危險** → 才需要 `uv tool` / `pipx` 給每個工具蓋「獨立套房」。
- **Node 的 `npm install -g` 相對安全** → npm 天生把依賴隔在各自 `node_modules`。
- 換句話說:**`uv tool install` 是在補上 Python 缺的隔離性**,讓它接近 npm 全域安裝那種「裝了不會互相搞死」的體驗。

---

## Part 4 — Skill / MCP 安裝為何用不同工具?(核心觀念)

**安裝方式不是看「它是 skill」決定的,而是看「它用哪種語言寫的」決定的。** 套件管理器只是那個語言的送貨管道。

| 語言 | 倉庫 | 安裝工具 |
|---|---|---|
| JavaScript / TypeScript | npm registry | `npm` / `npx` |
| Python | PyPI | `pip` / `uv` / `uvx` |

➡️ npm 裝不了 Python 套件,pip 也裝不了 JS 套件——**兩個平行宇宙**。作者用什麼語言寫,你就用對應工具裝。

### 最容易誤會的點:很多 skill 根本不裝套件

「Skill」不一定是程式套件,常見其實是三種不同的東西混在一起:

| 它其實是… | 安裝方式 |
|---|---|
| **知識型 skill**(`SKILL.md` / markdown) | **不碰 npm/pip**,只是把檔案放到對的資料夾(如 `~/.hermes/skills/`、或 `hermes skills install ...`) |
| **JS 寫的 MCP server / 工具** | `npx @scope/server-xxx` 或 `npm i -g` |
| **Python 寫的 MCP server / 工具** | `uvx mcp-server-xxx` 或 `uv tool install` |

> 所以你看到「有些用 npm、有些用 pip」,通常是因為它們**根本是不同類型的東西**,而不是規則不一致。

### 怎麼判斷該用哪個?看 repo 裡有什麼

| 線索 | 代表 | 用 |
|---|---|---|
| 有 `package.json` | JS/TS 套件 | `npm` / `npx` |
| 有 `pyproject.toml` / `setup.py` | Python 套件 | `uv` / `uvx` |
| 只有 `SKILL.md` / `.md` | 知識型 skill | 放檔案,不裝套件 |

---

## Part 5 — 團隊紅線(對齊環境規範)

- 禁止直接 `pip install`(污染全域、版本打架)→ 改 `uv add` / `uv tool install`
- 禁止 `curl … | bash`(黑箱執行遠端腳本、安全風險)→ 改用 uv / brew 等可控管道
- Python 一律 `uv`、Node 一律 `nvm` + 本地 `npm`/`npx`、系統工具用 `brew`
- 一句哲學:**能隔離就隔離、能臨時就臨時、別讓工具堆在全域互相干擾**

> `curl|bash` 唯一的「好處」是會順手幫你把 node / ripgrep / ffmpeg 等系統依賴一起裝;走 `uv tool` 的話這些要自己用 `brew` 或工具自帶的 `postinstall` 補。

---

## 一頁速記

```
# 函式庫(會 import)
uv add requests              ≈  npm install

# 常駐 CLI 工具
uv tool install hermes-agent ≈  npm install -g

# 跑一次,不安裝
uvx hermes-agent             ≈  npx some-cli

# 管理語言版本:Python→uv 內建 / Node→nvm
# 紅線:不 pip、不 curl|bash
```
