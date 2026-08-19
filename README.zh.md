# OpenSpec

> AI 輔助的規格驅動開發工作流程。Artifact 管線 + Agent skills。

## 總覽

OpenSpec 是一套結構化的 AI 輔助開發工作流程系統，強制執行清晰的管線：

```
new → continue → apply → verify → archive
```

每個步驟都會產出明確的 artifacts（提案、規格、設計、任務），既是文件也是實作契約。

### 工作流程時間軸

```
┌───────────────────────────────────────────────────────────────────────────────────┐
│                         OPEN SPEC 工作流程時間軸                                  │
├───────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│   ┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐              │
│   │   NEW    │────▶│ CONTINUE │────▶│  APPLY   │────▶│  VERIFY  │────▶ARCHIVE │ 
│   └──────────┘      └──────────┘      └──────────┘      └──────────┘              │
│        │                │                │                │                       │ 
│        ▼               ▼               ▼               ▼                      │ 
│   ┌──────────────────────────────────────────────────────────────────────────┐    │
│   │                        產出的 ARTIFACTS                                  │    │
│   ├─────────────────┬──────────────┬─────────────────┬───────────────────────┤    │
│   │  proposal.md    │  specs/*.md  │  design.md      │  tasks.md             │    │
│   │  (做什麼/為什麼)│  (需求規格)  │  (怎麼做/架構)  │  (檢查清單)           │    │
│   └─────────────────┴──────────────┴─────────────────┴───────────────────────┘    │
│                                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────────┐    │
│   │                        選項 / 平行流程                                   │    │
│   ├──────────────────────┬──────────────────────┬────────────────────────────┤    │
│   │ /opsx:propose        │ /opsx:explore        │ /opsx:grill                │    │
│   │ (一次產出所有        │ (思考夥伴模式)       │ (設計樹問答)               │    │
│   │  artifacts)          │                      │                            │    │
│   └──────────────────────┴──────────────────────┴────────────────────────────┘    │
│                                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────────┐    │
│   │                        同步 / 維護                                       │    │
│   ├──────────────────────┬──────────────────────┬────────────────────────────┤    │
│   │ /opsx:sync           │ /opsx:verify         │ /opsx:ff                   │    │
│   │ (delta→main spec)   │ (自訂 VERIFY.md)     │ (快速通過)                 │    │ 
│   └──────────────────────┴──────────────────────┴────────────────────────────┘    │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

## 核心概念

| 概念         | 說明                                                                   |
| ------------ | ---------------------------------------------------------------------- |
| **Change**   | 一個工作單元，擁有獨立目錄 `openspec/changes/<name>/`                  |
| **Artifact** | 結構化 Markdown 檔案：`proposal.md`、`specs/`、`design.md`、`tasks.md` |
| **Schema**   | 工作流程定義（如 `spec-driven`），決定 artifact 順序與相依性           |
| **CLI**      | `openspec` 指令管理狀態、產生模板、追蹤進度                            |

## 快速開始

```bash
# 初始化 OpenSpec
openspec init

# 開始新的 change
/opsx:new add-user-auth

# 繼續建立 artifacts（提案 → 規格 → 設計 → 任務）
/opsx:continue

# 實作任務
/opsx:apply

# 歸檔前驗證
/opsx:verify

# 歸檔完成的 change
/opsx:archive
```

## Skills（Agent 工作流程）

位於 `openspec-src/skills/`，這些是 Agent 可執行的工作流程：

| Skill                      | 指令             | 用途                        |
| -------------------------- | ---------------- | --------------------------- |
| `openspec-new-change`      | `/opsx:new`      | 建立新 change 目錄          |
| `openspec-continue-change` | `/opsx:continue` | 建立下一個 artifact         |
| `openspec-propose`         | `/opsx:propose`  | 一步產生所有 artifacts      |
| `openspec-apply-change`    | `/opsx:apply`    | 實作任務                    |
| `openspec-verify-change`   | `/opsx:verify`   | 三維度驗證                  |
| `openspec-archive-change`  | `/opsx:archive`  | 歸檔完成的 change           |
| `openspec-sync-specs`      | `/opsx:sync`     | Delta spec 同步到 main spec |
| `openspec-explore`         | `/opsx:explore`  | 思考夥伴模式                |
| `openspec-grill`           | `/opsx:grill`    | 設計樹問答                  |
| `openspec-onboard`         | `/opsx:onboard`  | 專案導入                    |
| `openspec-ff-change`       | `/opsx:ff`       | 快速通過 artifacts          |

## 驗證系統

### 內建三維度

1. **完整性** — 任務完成率 + 規格覆蓋率
2. **正確性** — 需求實現 + 場景覆蓋
3. **一致性** — 設計遵循度 + 程式碼模式一致性

### 自訂驗證（VERIFY.md）

透過 `VERIFY.md` 加入專案特定檢查：

```markdown
# Verification

## my-repo

```bash
npm run lint
npm run typecheck
npm test
```
```

**位置**：`openspec/VERIFY.md`（專案級）+ `openspec/changes/<name>/VERIFY.md`（change 級，累加模式）

**執行**：Agent 讀取 VERIFY.md，根據 change 影響的檔案/Repo 判斷哪些 section 在範圍內，只執行相關的指令。

## 目錄結構

```
openspec/
├── changes/
│   └── add-user-auth/
│       ├── .openspec.yaml
│       ├── proposal.md
│       ├── specs/
│       │   └── auth/
│       │       └── spec.md
│       ├── design.md
│       ├── tasks.md
│       └── VERIFY.md          # change 級驗證
├── specs/
│   └── auth/
│       └── spec.md            # 主規格（透過 /opsx:sync 同步）
└── VERIFY.md                  # 專案級驗證
```