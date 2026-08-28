# OpenSpec (Fork / 增強版)

> AI 輔助的規格驅動開發工作流程。**基於官方 skills 的改進。**

**[English](README.md)** | **繁體中文**

## 與官方版本的差異

| 領域                    | 官方                               | 本 Fork                              |
| ----------------------- | ---------------------------------- | ------------------------------------ |
| **反向提問與問答樹**    | N/A                                | 追加具有 openspec 意識的 grill-me    |
| **自訂驗證**            | ❌ 不支援                           | **`VERIFY.md`** — 宣告式、具範圍感知 |
| **專有名詞表**          | ❌ 不支援                           | **`GLOSSARY.md`** — 專案級 + change 級術語 |
| **指令前綴**            | `/openspec-*`                      | `/opsx:*` (較短、命名空間友善)       |
| **Store/Registry 支援** | 完整 (`--store`, `openspec store`) | ❌ 移除 — 單一 repo 簡潔性            |


---

## 本 Fork 的核心改進

### 1. Openspec-Grill (與 grill-me 的整合與增強)

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

### 2. VERIFY.md — 宣告式自訂驗證

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

### 3. GLOSSARY.md — 領域專有名詞表

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

---

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

## 授權

MIT