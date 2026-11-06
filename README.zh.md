# OpenSpec (Fork / 增強版)

> AI 輔助的規格驅動開發工作流程。**基於官方 skills 的改進。**

**[English](README.md)** | **繁體中文**

## 與官方版本的差異

| 領域                    | 官方                               | 本 Fork                                    |
| ----------------------- | ---------------------------------- | ------------------------------------------ |
| **反向提問與問答樹**    | N/A                                | 追加具有 openspec 意識的 grill-me          |
| **自訂驗證**            | ❌ 不支援                          | **`VERIFY.md`** — 宣告式、具範圍感知       |
| **專有名詞表**          | ❌ 不支援                          | **`GLOSSARY.md`** — 專案級 + change 級術語 |
| **Store/Registry 支援** | 完整 (`--store`, `openspec store`) | ❌ 移除 — 不必註冊 store                  |

---

## 本 Fork 的核心改進

### 1. Openspec-Status

**快速狀態總覽** — `openspec-status`

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
- 下一步: 執行 `openspec-verify-change fix-db-migration`
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

- **建立（grill）**：`openspec-grill` 在對話中偵測專案特有術語，向使用者確認定義＋別名後，寫入 `openspec/changes/<name>/GLOSSARY.md`
- **封存（archive）**：`openspec-archive-change` 將 change 級 `GLOSSARY.md` 合併進專案級 `openspec/GLOSSARY.md` — 新術語追加、既有術語覆蓋 — 然後刪除 change 級檔案

專案級 `openspec/GLOSSARY.md` 是單一來源；change 級檔案是累加性補充。

> **⚠️ 警告：`openspec init` 會覆蓋本 fork 的擴充。**
> 官方 `openspec init` **不認識**本 fork 的擴充（GLOSSARY、VERIFY）。若帶工具選擇執行它，會把 skills 重寫回官方版本，導致 GLOSSARY/VERIFY 整合被清空。`openspec-setup` 使用 `--tools none` 來避免這個問題。
> **若你曾帶工具選擇執行 `openspec init`，請立即重新執行 `npx skills add s16777216/openspec-improve` 還原 skills。**

---

### 5. ASD-STE100 寫作風格

`openspec-propose`、`openspec-continue-change`、`openspec-ff-change`、`openspec-update-change` 以 [Simplified Technical English (ASD-STE100)](https://www.asd-ste100.org/) 風格撰寫 artifact 內文，讓每個句子只有一種解讀。

- 短句、主動語態、任務用命令句
- 一詞一義 — 術語依專案級與 change 級 `GLOSSARY.md`
- 義務用 `MUST` / `MUST NOT`；用可量測的數值取代模糊用詞
- 技術名稱、模板與 OpenSpec 結構維持不變

**語言：** artifact 沿用請求的語言。英文直接遵循 ASD-STE100；其他語言套用不依賴語言的規則，並非正式的 ASD-STE100。這些規則只是提示詞指示，沒有自動檢查合規性。

---

## 🚀 安裝與設定

使用 [`skills`](https://github.com/vercel-labs/skills) CLI 安裝。

1. **安裝增強版 skills**：
   ```bash
   npx skills add s16777216/openspec-improve
   ```
   - 加上 `-g` 可改為全域安裝；加上 `-a <agent>`（例如 `-a claude-code -a codex`）可略過互動式選擇 agent。
2. **請 agent 執行 setup skill**：`openspec-setup`（Codex 使用 `$openspec-setup`）。可隨時重複執行，它會：
   - 檢查 OpenSpec CLI，未安裝時詢問是否安裝（`npm install -g openspec@latest`）；
   - 若 `openspec/` 不存在，執行 `openspec init --no-animation --tools none`；
   - 若 `openspec/VERIFY.md` 與 `openspec/GLOSSARY.md` 不存在，依據[初始模板規範](#templates)建立。

   每一步都會先徵求確認，且絕不覆蓋既有檔案。當缺少上述檔案時，`openspec-status`、`openspec-new-change`、`openspec-propose`、`openspec-onboard` 也會建議執行 `openspec-setup`。
   - **Windows 用戶**：若 PowerShell 因執行政策阻擋 `openspec`，agent 會改用 `openspec.cmd`。

repo 根目錄也內含一份符合規範的 `plugin.json`，讓遵循 [Agent Plugins 規範](https://agent-plugins.org/) 的 client 可以直接從此 repo 載入。上面的 `npx skills add` 仍是安裝到 Claude Code 或 Codex 的主要、推薦方式。

---

### <a id="templates"></a> 📄 專案初始模板規範 (VERIFY.md 與 GLOSSARY.md)

`openspec-setup` 會在專案 `openspec/` 目錄建立以下範本（若尚不存在）。範本內容內嵌於 `skills/openspec-setup/SKILL.md`，請保持兩處同步：

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
   ```markdown
   # Glossary

   ## core

   - **ExampleTerm** — 範例專有名詞定義。別名：alias1、alias2。
   ```

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

### <a id="command-reference"></a> 完整指令參考

| Skill                          | 說明                                     |
| :------------------------------ | :--------------------------------------- |
| `openspec-setup`               | 在專案中設定 OpenSpec：檢查 CLI、`openspec init`、建立初始 `VERIFY.md` 與 `GLOSSARY.md` |
| `openspec-status`              | 快速總覽所有進行中提案與下一步建議       |
| `openspec-explore`             | 工作前/中思考問題                        |
| `openspec-grill`               | 設計樹問答，收斂決策                     |
| `openspec-new-change`          | 建立新提案，逐步產出工件                 |
| `openspec-continue-change`     | 繼續既有提案的下一個工件                 |
| `openspec-ff-change`           | 快轉：一次產出所有工件                   |
| `openspec-propose`             | 建立提案並一次產出所有工件               |
| `openspec-update-change`       | 修訂既有規劃工件並保持一致，不修改程式碼 |
| `openspec-apply-change`        | 根據提案實作任務                         |
| `openspec-verify-change`       | 驗證實作是否符合工件                     |
| `openspec-sync-specs`          | 同步主規格，並依實作更新在地 SPEC.md     |
| `openspec-archive-change`      | 歸檔已完成的提案                         |
| `openspec-bulk-archive-change` | 一次歸檔多個已完成的提案                 |
| `openspec-onboard`             | 引導式入門，完整工作流程週期             |

---

### 修訂既有規劃

當需求改變、grill/explore 得出新決策，或既有規劃工件彼此矛盾時，使用 `openspec-update-change <change-name>`（Codex 使用 `$openspec-update-change`）。它會提出既有工件的修訂，經使用者確認後寫入；不建立缺少的工件，也不修改實作程式碼。

本 fork 的連動包括帶入已確認的 grill/explore 決策，並依專案級與 change 級 `GLOSSARY.md` 檢查用詞。當規劃修訂影響術語或驗證範圍時，可提出既有 change 級術語表與 `VERIFY.md` 的修訂。專案級檔案保持唯讀；驗證設定採累加方式，實際檢查交給 `openspec-verify-change`。缺少的擴充檔案會列為待設定項目，不自動建立。

缺少的工件交給 `openspec-continue-change` 或 `openspec-ff-change`，修訂後的實作交給 `openspec-apply-change`，將 delta specs 整合進主規格則使用 `openspec-sync-specs`。終端機的 `openspec update` 是另一個用途：依已安裝的 CLI 重新產生 skills。

## 目錄結構

```
openspec/
├── changes/
│   └── add-user-auth/
│       ├── .openspec.yaml
│       ├── proposal.md
│       ├── specs/
│       │   └── identity/authentication/spec.md
│       ├── design.md
│       ├── tasks.md
│       ├── VERIFY.md          # change 級（累加）
│       └── GLOSSARY.md        # change 級術語（封存時合併）
├── specs/
│   └── identity/authentication/spec.md  # 主規格
├── VERIFY.md                  # 專案級
└── GLOSSARY.md                # 專案級（單一來源）
```

`specs/` 以下的相對路徑就是能力 ID。新增規格依業務領域與能力分層，例如 `identity/authentication`。提案的 delta spec 使用**相同的相對路徑**，且只建立本次變更涉及的能力。既有 `auth` 等單層能力 ID 仍可使用；修改既有規格時沿用其 ID。需求與情境寫在能力的 `spec.md` 內，不為每項需求另建資料夾。

規格描述跨 repo 的行為，不在行為規格 `spec.md` 中標示單一程式碼 repo。提案涉及的 repo 與實作位置記錄在 `design.md`；各 repo 的驗證指令放在 `VERIFY.md`。

### 在地 `SPEC.md` 與集中式規格

在實作該能力的 repo 或模組根目錄放置 `SPEC.md`，記錄長期有效的程式碼導覽。同一能力可以出現在多個 repo 的 `SPEC.md`。依真正的模組責任選擇放置位置，不在每個資料夾都建立一份。

| 文件 | 職責 |
| --- | --- |
| `openspec/specs/<capability-id>/spec.md` | 正式行為規格：需求與情境，不依程式碼 repo 分類。 |
| `openspec/changes/<name>/specs/<capability-id>/spec.md` | 本次提案對行為的修改。 |
| `openspec/changes/<name>/design.md` | 本次變更涉及的 repo、模組與實作決策。 |
| `<repo-or-module>/SPEC.md` | 此處的責任、程式碼入口、介面／依賴，以及指向正式能力規格的連結或穩定路徑。 |

例如 API repo 根目錄的文件：

```markdown
# API 實作

## identity/authentication
- 正式規格：spec repo 中的 `openspec/specs/identity/authentication/spec.md`
- 責任：驗證登入請求並建立 session。
- 程式碼入口：`src/auth/routes.ts`、`src/auth/service.ts`
- 介面：供 web repo 使用的 session API。
```

apply 在 change 工件中記錄待同步的在地 `SPEC.md` 路徑。`openspec-sync-specs` 同時整合主規格，並依實際程式碼更新在地文件。尚未實作或無法存取的部分保留為待辦；即使主規格已同步，下次同步仍會補齊這些文件。驗證流程將待整合文件與實作失敗分別回報；歸檔沿用同一套同步流程。一般歸檔前先清完待辦；若要帶著未完成的文件更新歸檔，須明確確認相關警告。

需求與情境文字只寫在集中式規格，避免兩邊各維護一份。即使集中式 `openspec/` 位於另一個 repo，在地文件仍可引用 `identity/authentication` 這個能力 ID。

---

## 相容性

- **OpenSpec CLI**：既有單層流程已針對 **1.8.0** 測試；巢狀規格的搜尋與對應已用 **1.13.0** 確認。採用新結構前，請使用該版本或確認已安裝的 CLI 支援巢狀規格。
- **平台**：macOS、Linux、Windows（Windows 上若 PowerShell 執行政策阻擋 `.ps1` shim，請改用 `openspec.cmd`）。

---

## 維護指南

### 新增工作流程

新增一個工作流程 skill 的步驟：

1. 建立 `skills/openspec-<action>/SKILL.md`，寫入工作流程內容。
2. 確保步驟、保護條件和輸出契約與其他 skill 一致。
3. 在 README.md 與 README.zh.md 的參考表中加入新 skill。
4. 執行下方一致性檢查（目前為 15 個 skill）。

### 一致性檢查

執行以下唯讀檢查以驗證專案健康狀態：

```bash
# 掃描殘留的 /opsx 別名寫法(目前所有指令都用真實的 skill 名稱)
grep -ri 'opsx' skills/ README.md README.zh.md

# 驗證所有 skill 的 YAML frontmatter
grep -l 'generatedBy' skills/*/SKILL.md

# 檢查 Markdown fence 合法性
# （使用 markdown linter 或手動檢查）
```

---

## 授權

MIT
