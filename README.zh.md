# OpenSpec (Fork / 增強版)

> AI 輔助的規格驅動開發工作流程。**基於官方 skills 的改進。**

**[English](README.md)** | **繁體中文**

## 與官方版本的差異

| 領域                    | 官方                               | 本 Fork                                    |
| ----------------------- | ---------------------------------- | ------------------------------------------ |
| **反向提問與問答樹**    | N/A                                | 追加具有 openspec 意識的 grill-me          |
| **自訂驗證**            | ❌ 不支援                           | **`VERIFY.md`** — 宣告式、具範圍感知       |
| **專有名詞表**          | ❌ 不支援                           | **`GLOSSARY.md`** — 專案級 + change 級術語 |
| **指令前綴**            | `/openspec-*`                      | `/opsx-*` (較短、命名空間友善)             |
| **Store/Registry 支援** | 完整 (`--store`, `openspec store`) | ❌ 移除 — 單一 repo 簡潔性                  |


---

## 本 Fork 的核心改進

### 1. Openspec-Status

**快速狀態總覽** — `/opsx-status`

提供所有進行中 OpenSpec 提案的**精簡總覽**，包含：
- 每個提案的階段（ideation / in-progress / ready-for-review）
- 任務進度（例如 3/7 tasks）
- 下一步與潛在阻塞點 (blocker)
- 單一推薦下一步動作

**範例輸出：**
```
## OpenSpec Status

3 個進行中的提案：

### add-auth-flow — "OAuth login for the API"
- 階段: in-progress  •  進度: 3/7 tasks
- 下一步: task 4 "Wire up refresh token rotation"

### fix-db-migration — "Repair flaky migration ordering"
- 階段: ready-for-review  •  進度: 7/7 tasks
- 下一步: 執行 `/opsx-verify fix-db-migration`
```

非常適合用來上班或回到崗位時，5 秒鐘快速進入狀況。

---

### 2. Openspec-Grill (與 grill-me 的整合與增強)

#### 新增功能

- **反向提問 (Reverse Questioning)** — 在開始前主動澄清需求
- **問答樹 (Q&A Tree)** — 結構化、可追蹤的討論流程
- **specs 意識** — 直接引用與討論 spec 文件
- **設計樹整合** — 自動生成 design.md
- **狀態追蹤** — 記錄問答狀態，避免重複

#### 輸出結果

討論結束後自動生成：
- ✅ `design.md` — 結構化的設計決策
- ✅ `tasks.md` — 可執行的任務清單
- ✅ `specs/` — spec 文件更新（若有需要）

---

### 3. VERIFY.md — 宣告式自訂驗證

````markdown
# Verification

## my-repo

```bash
npm run lint
npm run typecheck
npm test
```

## shared-lib

```bash
cargo build
cargo test
```
````

**運作方式：**
- Agent 讀取 `VERIFY.md`（專案級 + change 級，**累加**）
- 交叉比對 change 影響的檔案，**判斷哪些 repo section 在範圍內**
- 只執行在範圍內的 section
- 略過的 section 會在報告中註記：`"Skipped shared-lib (not in change scope)"`

### 累加式 Change 級設定

```
專案 VERIFY.md：     所有 repo 的 lint + typecheck + test
Change VERIFY.md：   針對 auth repo 額外加 security-scan
結果：               兩者都執行，合併進報告
```

---

### 4. GLOSSARY.md — 領域專有名詞表

專案特有術語（行話、縮寫、領域詞彙）的集中登錄處。

**格式** — `術語 + 定義 + 別名` 的純列表（**不人工標** ADDED/MODIFIED；diff 在封存時自動判定）：

```markdown
# Glossary

## auth

- **Principal** — 已驗證的請求主體。別名：user、account。
```

**運作方式：**
- **建立（grill）**：`/opsx-grill` 在對話中偵測專案特有術語，向使用者確認定義＋別名後，寫入 `openspec/changes/<name>/GLOSSARY.md`
- **封存（archive）**：`/opsx-archive` 將 change 級 `GLOSSARY.md` 合併進專案級 `openspec/GLOSSARY.md` — 新術語追加、既有術語覆蓋 — 然後刪除 change 級檔案

專案級 `openspec/GLOSSARY.md` 是單一來源；change 級檔案是累加性補充。

> **⚠️ 警告：`openspec init` 會覆蓋本 fork 的擴充。**
> 官方 `openspec init` **不認識**本 fork 的擴充（GLOSSARY、VERIFY、`/opsx-*`）。執行它會把 `commands/` 與 `skills/` 重寫回官方版本，導致 GLOSSARY/VERIFY 整合被清空。
> **若你曾執行 `openspec init`，請立即還原：**
> ```bash
> git checkout -- commands/ skills/
> ```
> （`.opencode/` 在 `.gitignore` 內 — 若需要一併還原，從 `commands/` 複製回去。）

---

## 🚀 安裝與設定 (給 AI Agent 自動安裝)

本 Fork 專為搭配 AI Coding Agent 設計。你只需複製對應工具的**單行指令**給你的 Agent，Agent 就會讀取本頁面並依照指定章節步驟自動完成安裝：

| 工具 | 複製給 Agent 的單行指令 |
| :--- | :--- |
| **Antigravity** | `Fetch and follow instructions from https://github.com/s16777216/openspec-improve#antigravity` |
| **Codex** | `Fetch and follow instructions from https://github.com/s16777216/openspec-improve#codex` |
| **OpenCode** | `Fetch and follow instructions from https://github.com/s16777216/openspec-improve#opencode` |

---

### <a id="antigravity"></a> Google Antigravity

當 Agent 收到 `#antigravity` 安裝指令時，請依照以下步驟自動完成配置：

1. **環境檢查**：
   - 執行 `openspec --version` 確認本機已安裝 CLI。若未安裝，執行 `npm install -g openspec@latest`。
   - **Windows 用戶**：若 PowerShell 因執行政策阻擋 `openspec`，請改用 `openspec.cmd`。
2. **初始化專案**：
   - 在專案根目錄執行 `openspec init --no-animation --tools antigravity` 建立 OpenSpec 基礎結構與配置。
3. **覆蓋增強版技能與模板**：
   - 將本 repo 的 `skills/` 複製至專案根目錄 `.agent/skills/`（或 `.agents/skills/`），若已存在則覆蓋。
   - 將本 repo 的 `commands/` 複製至專案根目錄 `.agent/workflows/`（若已存在則覆蓋，支援 `/opsx-*` 指令）。
   - *(可選全域級)*：若使用者要求全域安裝，複製至 `~/.gemini/config/skills/` 與 `~/.gemini/config/global_workflows/`。
   - 若專案內尚無 `openspec/VERIFY.md` 與 `openspec/GLOSSARY.md`，依據下方[初始模板規範](#templates)自動建立。

---

### <a id="codex"></a> Codex

當 Agent 收到 `#codex` 安裝指令時，請依照以下步驟自動完成配置：

1. **環境檢查**：
   - 執行 `openspec --version` 確認本機已安裝 CLI。若未安裝，執行 `npm install -g openspec@latest`。
   - **Windows 用戶**：若 PowerShell 因執行政策阻擋 `openspec`，請改用 `openspec.cmd`。
2. **初始化專案**：
   - 在專案根目錄執行 `openspec init --no-animation --tools codex` 建立 OpenSpec 基礎結構與配置。
3. **覆蓋增強版技能與模板**：
   - 將本 repo 的 `skills/` 複製至專案根目錄 `.agents/skills/`，若已存在則覆蓋。
   - Codex 主要透過 Skills 體系調用（例如 `$openspec-propose`）。
   - 若專案內尚無 `openspec/VERIFY.md` 與 `openspec/GLOSSARY.md`，依據下方[初始模板規範](#templates)自動建立。

---

### <a id="opencode"></a> OpenCode

當 Agent 收到 `#opencode` 安裝指令時，請依照以下步驟自動完成配置：

1. **環境檢查**：
   - 執行 `openspec --version` 確認本機已安裝 CLI。若未安裝，執行 `npm install -g openspec@latest`。
   - **Windows 用戶**：若 PowerShell 因執行政策阻擋 `openspec`，請改用 `openspec.cmd`。
2. **初始化專案**：
   - 在專案根目錄執行 `openspec init --no-animation --tools opencode` 建立 OpenSpec 基礎結構與配置。
3. **覆蓋增強版技能與模板**：
   - 將本 repo 的 `commands/` 複製至專案根目錄 `.opencode/commands/`，若已存在則覆蓋（支援 `/opsx-*` 指令）。
   - 將本 repo 的 `skills/` 複製至專案根目錄 `.opencode/skills/`，若已存在則覆蓋。
   - 若專案內尚無 `openspec/VERIFY.md` 與 `openspec/GLOSSARY.md`，依據下方[初始模板規範](#templates)自動建立。

---

### <a id="templates"></a> 📄 專案初始模板規範 (VERIFY.md 與 GLOSSARY.md)

Agent 進行初始化時，應在專案 `openspec/` 目錄建立以下範本（若尚不存在）：

1. **`openspec/VERIFY.md`**（宣告式自訂驗證）
   ````markdown
   # Verification

   ## <專案或模組名稱>

   ```bash
   # 請依據專案實際語言與工具填入驗證指令（例如 npm test / cargo test 等）
   npm test
   ```
   ````

2. **`openspec/GLOSSARY.md`**（領域專有名詞表）
   ````markdown
   # Glossary

   ## core

   - **ExampleTerm** — 範例專有名詞定義。別名：alias1、alias2。
   ````

---

## Skills (Agent 工作流程)

```
     |
    ●-- explore / grill : 需求討論，設計樹問答與交流
     |
     |
    ●-- propose : 根據討論結果產出提案
     |
     |
    ●-- apply : 根據提案實作
     |
     |
    ●-- verify : 根據提案與實作結果產出驗證報告
     |
     |
    ●-- archive : 歸檔提案
     |
     v
```

### 完整指令參考

| 指令 | 說明 |
| :--- | :--- |
| `/opsx-status` | 快速總覽所有進行中提案與下一步建議 |
| `/opsx-explore` | 工作前/中思考問題 |
| `/opsx-grill` | 設計樹問答，收斂決策 |
| `/opsx-new` | 建立新提案，逐步產出工件 |
| `/opsx-continue` | 繼續既有提案的下一個工件 |
| `/opsx-ff` | 快轉：一次產出所有工件 |
| `/opsx-propose` | 建立提案並一次產出所有工件 |
| `/opsx-update` | 修訂既有規劃工件並保持一致，不修改程式碼 |
| `/opsx-apply` | 根據提案實作任務 |
| `/opsx-verify` | 驗證實作是否符合工件 |
| `/opsx-sync` | 將 delta specs 同步至主規格 |
| `/opsx-archive` | 歸檔已完成的提案 |
| `/opsx-bulk-archive` | 一次歸檔多個已完成的提案 |
| `/opsx-onboard` | 引導式入門，完整工作流程週期 |

---

### 修訂既有規劃

當需求改變、grill/explore 得出新決策，或既有規劃工件彼此矛盾時，使用 `/opsx-update <change-name>`（Codex 使用 `$openspec-update-change`）。它會提出既有工件的修訂，經使用者確認後寫入；不建立缺少的工件，也不修改實作程式碼。

本 fork 的連動包括帶入已確認的 grill/explore 決策，並依專案級與 change 級 `GLOSSARY.md` 檢查用詞。當規劃修訂影響術語或驗證範圍時，可提出既有 change 級術語表與 `VERIFY.md` 的修訂。專案級檔案保持唯讀；驗證設定採累加方式，實際檢查交給 `/opsx-verify`。缺少的擴充檔案會列為待設定項目，不自動建立。

缺少的工件交給 `/opsx-continue` 或 `/opsx-ff`，修訂後的實作交給 `/opsx-apply`，將 delta specs 整合進主規格則使用 `/opsx-sync`。終端機的 `openspec update` 是另一個用途：依已安裝的 CLI 重新產生 skills 與 commands。

## 目錄結構

```
openspec/
├── changes/
│   └── add-user-auth/
│       ├── .openspec.yaml
│       ├── proposal.md
│       ├── specs/
│       │   └── auth/spec.md
│       ├── design.md
│       ├── tasks.md
│       ├── VERIFY.md          # change 級（累加）
│       └── GLOSSARY.md        # change 級術語（封存時合併）
├── specs/
│   └── auth/spec.md           # 主規格
├── VERIFY.md                  # 專案級
└── GLOSSARY.md                # 專案級（單一來源）
```

---

## 相容性

- **OpenSpec CLI**：已針對 **1.8.0** 版本測試。最低支援版本可能不同，請以 `openspec --version` 確認。
- **平台**：macOS、Linux、Windows（Windows 上若 PowerShell 執行政策阻擋 `.ps1` shim，請改用 `openspec.cmd`）。

---

## 維護指南

### 新增工作流程

新增一組工作流程指令 + skill 的步驟：

1. 建立 `commands/opsx-<action>.md`，寫入 slash command 內容。
2. 建立 `skills/openspec-<action>/SKILL.md`，寫入匹配的內容（適配 skill 格式）。
3. 確保兩者的步驟、保護條件和輸出契約一致。
4. 在 README.md 與 README.zh.md 的指令參考表中加入新指令。
5. 執行下方一致性檢查，確認一對一配對正確（目前為 14 個 command 與 14 個 skill）。

### 一致性檢查

執行以下唯讀檢查以驗證專案健康狀態：

```bash
# 確認每個 command 都有對應的 skill，反之亦然
# （手動檢查或使用腳本比對 commands/ 與 skills/ 目錄名稱）

# 掃描不一致的 /opsx: 引用
grep -r '/opsx:' commands/ skills/ README.md README.zh.md

# 驗證所有 skill 的 YAML frontmatter
grep -l 'generatedBy' skills/*/SKILL.md

# 檢查 Markdown fence 合法性
# （使用 markdown linter 或手動檢查）
```

---

## 授權

MIT
