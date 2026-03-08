# Spec-Kit Pure Skill 使用手冊

> **Pure Skill 版本** (`skills/`) vs **原版 CLI** (`specify-cli`) 完整比較指南

---

## 目錄

- [兩個版本是什麼](#兩個版本是什麼)
- [快速比較表](#快速比較表)
- [安裝與初始化](#安裝與初始化)
- [完整工作流程比較](#完整工作流程比較)
- [各指令差異說明](#各指令差異說明)
- [支援的 AI Agent](#支援的-ai-agent)
- [檔案結構差異](#檔案結構差異)
- [限制與注意事項](#限制與注意事項)
- [常見問題](#常見問題)

---

## 兩個版本是什麼

### 原版 CLI（`specify-cli`）

透過 Python 套件安裝的命令列工具。執行 `specify init` 時，CLI 會：

1. 從 GitHub 下載最新的 `templates/` 和 `scripts/`
2. 把 skill 檔案複製到你選擇的 AI agent 的 commands 目錄
3. 把 bash/PowerShell 腳本複製到 `scripts/` 目錄
4. 把模板複製到 `.specify/templates/`

之後每次執行 `/speckit.*` 指令，AI agent 會讀取 skill 檔案，並在必要時呼叫 `scripts/` 裡的腳本。

### Pure Skill 版本（`skills/`）

**不需要任何安裝**。把 `skills/` 目錄的 `.md` 檔案直接複製到你的 AI agent 的 commands 目錄。所有原本由 bash/PowerShell 腳本負責的工作（建立分支、建立目錄、偵測 agent 類型等）都改由 AI agent 直接執行。

---

## 快速比較表

| 項目 | 原版 CLI | Pure Skill |
|------|---------|-----------|
| **安裝** | `uv tool install specify-cli` | 複製 `.md` 檔案即可 |
| **相依性** | Python 3.11+、uv、bash/PowerShell | 無（僅需 git） |
| **Windows 支援** | ✅（有 `.ps1` 腳本） | ⚠️（git 命令需要 git bash 或 WSL）|
| **支援 AI agent 數量** | 20+（CLI 自動選） | 全部（手動複製到對應目錄）|
| **Branch 建立** | 腳本自動執行 | AI agent 直接執行 `git checkout -b` |
| **目錄建立** | 腳本自動執行 | AI agent 直接執行 `mkdir -p` |
| **Agent 偵測** | CLI 精確辨識 20+ agent | AI agent 查找特徵檔案 |
| **範本內容** | 從 `.specify/templates/` 讀取 | 直接嵌在 skill 檔案裡 |
| **Extension hooks** | ✅ 完整支援 | ✅ 完整支援（讀 `.specify/extensions.yml`）|
| **升級方式** | `uv tool upgrade specify-cli` | 重新複製新版 `.md` 檔 |
| **離線使用** | ✅（安裝後） | ✅ |
| **跨專案共用** | ✅（全域安裝） | 需要每個專案各複製一份（或符號連結）|

---

## 安裝與初始化

### 原版 CLI

```bash
# 1. 安裝（一次即可，全域）
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# 2. 初始化新專案（自動建立目錄、複製所有檔案）
specify init my-project --ai claude

# 3. 或在現有專案初始化
cd my-existing-project
specify init . --ai claude --here

# 4. 確認安裝正確
specify check
```

執行完 `specify init` 後，專案結構：

```
my-project/
├── .claude/commands/       ← skill 檔案（自動複製）
├── .specify/
│   ├── memory/             ← constitution.md 存放處
│   └── templates/          ← 本地模板副本
├── scripts/
│   ├── bash/               ← bash 腳本（skill 執行時呼叫）
│   └── powershell/         ← PowerShell 腳本
└── specs/                  ← feature 規格存放處
```

### Pure Skill 版本

```bash
# 1. 進入你的專案
cd my-project

# 2a. 如果用 Claude Code
mkdir -p .claude/commands
cp /path/to/spec-kit/skills/*.md .claude/commands/

# 2b. 如果用 Cursor
mkdir -p .cursor/rules
cp /path/to/spec-kit/skills/*.md .cursor/rules/

# 2c. 如果用 GitHub Copilot（rules 方式）
mkdir -p .github/copilot-instructions
cp /path/to/spec-kit/skills/*.md .github/copilot-instructions/

# 2d. 其他 agent：複製到該 agent 的 commands/rules 目錄
```

執行完後，專案結構（以 Claude Code 為例）：

```
my-project/
├── .claude/commands/       ← 複製來的 skill 檔案
├── .specify/
│   └── memory/             ← constitution.md（執行 /speckit.constitution 時建立）
└── specs/                  ← feature 規格（執行 /speckit.specify 時建立）
```

注意：**`scripts/`、`.specify/templates/` 完全不需要**。

---

## 完整工作流程比較

以建立一個照片管理 app 為例，完整走一遍 SDD 流程：

### 步驟 1：建立專案憲法

**操作完全相同**——兩個版本的 `/speckit.constitution` 行為一致。

```
/speckit.constitution 建立以測試優先、程式碼品質和效能為核心的專案原則
```

> **差異**：原版 CLI 初始化時已將 `constitution-template.md` 複製到 `.specify/templates/`，skill 從那裡讀取。Pure Skill 版本：constitution 模板直接嵌在 skill 裡，執行時直接在 `.specify/memory/` 建立。

---

### 步驟 2：建立 Feature Spec

```
/speckit.specify 建立一個可以幫我整理照片的應用程式，照片可以分組到相簿，相簿按日期分組，可以透過拖放重新排列
```

**原版 CLI 執行流程**：

```
1. AI agent 計算 short-name 和編號
2. AI agent 呼叫 scripts/bash/create-new-feature.sh --json --number 1 --short-name "photo-album"
   └─ 腳本執行：
      - git checkout -b 1-photo-album
      - mkdir -p specs/1-photo-album/checklists
      - cp .specify/templates/spec-template.md specs/1-photo-album/spec.md
      - 輸出 JSON: {"BRANCH_NAME": "1-photo-album", "SPEC_FILE": "specs/1-photo-album/spec.md"}
3. AI agent 讀取 .specify/templates/spec-template.md 了解格式
4. AI agent 依格式填寫 spec.md
```

**Pure Skill 執行流程**：

```
1. AI agent 計算 short-name 和編號
2. AI agent 直接執行：
   - git fetch --all --prune
   - git ls-remote / git branch / ls specs/（確認編號）
   - git checkout -b 1-photo-album
   - mkdir -p specs/1-photo-album/checklists
3. AI agent 使用 skill 內嵌的模板格式直接填寫 spec.md
```

> **差異**：效果完全相同。Pure Skill 版少了「呼叫外部腳本→解析 JSON」這個環節，AI 直接操作。

---

### 步驟 3：釐清需求（可選）

```
/speckit.clarify 著重在安全性和效能需求
```

**兩個版本行為完全相同**，差異只在：

| | 原版 CLI | Pure Skill |
|-|---------|-----------|
| 取得 FEATURE_DIR | 執行 `scripts/bash/check-prerequisites.sh --json --paths-only` | 直接 `git branch --show-current` |

---

### 步驟 4：產生技術計畫

```
/speckit.plan 使用 Vite + 原生 HTML/CSS/JS，圖片不上傳，metadata 存在本地 SQLite
```

**原版 CLI 執行流程**：

```
1. 執行 setup-plan.sh --json
   └─ 腳本：
      - cp .specify/templates/plan-template.md specs/1-photo-album/plan.md
      - mkdir -p specs/1-photo-album/contracts
      - 輸出 JSON: {FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH}
2. AI agent 填寫 plan.md
3. 執行 scripts/bash/update-agent-context.sh claude
   └─ 腳本偵測 agent 類型，更新 CLAUDE.md
```

**Pure Skill 執行流程**：

```
1. git branch --show-current → 取得 BRANCH
2. 讀取 templates/plan-template.md（如果存在）或使用 skill 內嵌模板
3. AI agent 直接建立 plan.md、contracts/
4. AI agent 查找 CLAUDE.md / .cursor/rules/ 等特徵檔案，直接更新 agent context
```

> **差異**：`update-agent-context.sh` 會讀取 `.specify/templates/agent-file-template.md` 來決定要寫什麼格式。Pure Skill 版的 AI 根據偵測到的 agent 類型，自行判斷適當的更新格式。

---

### 步驟 5：產生任務清單

```
/speckit.tasks
```

**差異**：原版呼叫 `check-prerequisites.sh --json` 取得 FEATURE_DIR 和 AVAILABLE_DOCS。Pure Skill 版直接 `git branch --show-current` + `ls specs/{BRANCH}/`。**輸出的 tasks.md 格式完全相同**。

---

### 步驟 6：一致性分析（可選）

```
/speckit.analyze
```

**差異**：同上，只差在取得路徑的方式。分析邏輯和報告格式完全相同。

---

### 步驟 7：建立 Checklist（可選）

```
/speckit.checklist 為安全性需求建立 checklist
```

**兩個版本行為完全相同**。

---

### 步驟 8：執行實作

```
/speckit.implement
```

**差異**：原版呼叫 `check-prerequisites.sh --json --require-tasks --include-tasks`，會在 tasks.md 不存在時提早報錯。Pure Skill 版由 AI agent 自行檢查 tasks.md 是否存在。

---

## 各指令差異說明

### `/speckit.specify`

| 項目 | 原版 CLI | Pure Skill |
|------|---------|-----------|
| 建立 branch | `create-new-feature.sh` 執行 | AI 直接 `git checkout -b` |
| 建立目錄 | 腳本執行 `mkdir -p` | AI 直接 `mkdir -p` |
| spec 模板來源 | `.specify/templates/spec-template.md` | Skill 內嵌模板結構 |
| 輸出格式 | JSON → AI 解析 | AI 直接建立變數 |

### `/speckit.plan`

| 項目 | 原版 CLI | Pure Skill |
|------|---------|-----------|
| 取得路徑 | `setup-plan.sh --json` | `git branch --show-current` |
| plan.md 模板 | `.specify/templates/plan-template.md` | Skill 內嵌模板 |
| Agent context 更新 | `update-agent-context.sh` | AI 偵測特徵檔案後直接更新 |

> **注意**：原版 CLI 的 `update-agent-context.sh` 對 20+ 個 agent 各有精確的處理邏輯（包括 TOML frontmatter 格式等）。Pure Skill 版的 AI 根據偵測結果推斷，對於冷門 agent 可能格式不完全精確。

### `/speckit.tasks`

| 項目 | 原版 CLI | Pure Skill |
|------|---------|-----------|
| 取得 FEATURE_DIR | `check-prerequisites.sh --json` | `git branch --show-current` |
| Extension hooks | 完整支援 | 完整支援（同樣讀 `.specify/extensions.yml`）|
| 輸出 tasks.md | **完全相同** | **完全相同** |

### `/speckit.implement`

| 項目 | 原版 CLI | Pure Skill |
|------|---------|-----------|
| tasks.md 缺失檢查 | `--require-tasks` flag 提早報錯 | AI agent 自行檢查 |
| Checklist gate | 完整支援 | 完整支援 |
| Extension hooks | 完整支援 | 完整支援 |

### `/speckit.constitution`

| 項目 | 原版 CLI | Pure Skill |
|------|---------|-----------|
| 模板來源 | `.specify/templates/constitution-template.md` | Skill 內嵌模板 |
| 存放位置 | `.specify/memory/constitution.md` | **相同** |
| 相依模板更新 | 讀取 `.specify/templates/` 目錄 | 讀取 `templates/`（若存在），否則跳過 |

---

## 支援的 AI Agent

### 原版 CLI

CLI 透過 `--ai <agent>` 參數，為 20+ 個 agent 各自建立對應格式的 commands 目錄和設定：

```bash
specify init my-project --ai claude        # → .claude/commands/
specify init my-project --ai cursor-agent  # → .cursor/rules/
specify init my-project --ai copilot       # → .github/copilot-instructions.md
specify init my-project --ai windsurf      # → .windsurfrules
specify init my-project --ai roo           # → .roo/rules/
specify init my-project --ai generic --ai-commands-dir .myagent/commands/
```

### Pure Skill 版本

手動複製到對應目錄：

```bash
# Claude Code
cp skills/*.md .claude/commands/

# Cursor
cp skills/*.md .cursor/rules/

# GitHub Copilot
cp skills/*.md .github/copilot-instructions/   # 或參照 Copilot 文件

# Windsurf
cp skills/*.md .windsurfrules/                  # 或參照 Windsurf 文件

# Roo Code
cp skills/*.md .roo/rules/

# 任何其他 agent：複製到該 agent 支援的 commands/rules 目錄
```

---

## 檔案結構差異

### 原版 CLI 初始化後

```
my-project/
├── .claude/commands/          ← skill 檔案（或其他 agent 對應目錄）
│   ├── speckit.specify.md
│   ├── speckit.plan.md
│   └── ...
├── .specify/
│   ├── memory/
│   │   └── constitution.md    ← /speckit.constitution 建立
│   └── templates/             ← 本地模板副本
│       ├── spec-template.md
│       ├── plan-template.md
│       ├── tasks-template.md
│       ├── constitution-template.md
│       ├── checklist-template.md
│       └── commands/          ← skill 原始碼備份
├── scripts/
│   ├── bash/                  ← bash 腳本（skill 執行期呼叫）
│   │   ├── check-prerequisites.sh
│   │   ├── create-new-feature.sh
│   │   ├── setup-plan.sh
│   │   └── update-agent-context.sh
│   └── powershell/            ← PowerShell 腳本
└── specs/                     ← /speckit.specify 建立
    └── 1-photo-album/
        ├── spec.md
        ├── plan.md
        ├── tasks.md
        └── checklists/
```

### Pure Skill 版本初始化後

```
my-project/
├── .claude/commands/          ← 複製來的 pure skill 檔案
│   ├── speckit.specify.md
│   ├── speckit.plan.md
│   └── ...
├── .specify/
│   └── memory/
│       └── constitution.md    ← /speckit.constitution 建立
└── specs/                     ← /speckit.specify 建立
    └── 1-photo-album/
        ├── spec.md
        ├── plan.md
        ├── tasks.md
        └── checklists/
```

**`scripts/`、`.specify/templates/` 完全不存在**——這正是「pure skill」的核心：不依賴任何附帶基礎設施。

---

## 限制與注意事項

### 1. Agent Context 更新精確度

原版 `update-agent-context.sh` 對每個 agent 各有精確的處理邏輯，包括：
- 特定的 TOML frontmatter 格式（某些 agent 需要）
- 精確的標記區塊（防止覆寫手動內容）
- agent 專屬的指令格式

Pure Skill 版依靠 AI agent 自行判斷，**對主流 agent（Claude、Cursor、Copilot、Windsurf）效果良好**，冷門 agent 可能需要手動調整結果。

### 2. Windows 原生支援

原版 CLI 有完整的 `.ps1` PowerShell 腳本，在 Windows 原生環境完全可用。

Pure Skill 版的 inline git 指令使用 bash 語法。Windows 用戶需要：
- **Git Bash**（推薦，隨 Git for Windows 附帶）
- 或 **WSL**

### 3. 升級

| | 原版 CLI | Pure Skill |
|-|---------|-----------|
| 升級工具 | `uv tool upgrade specify-cli` | 重新複製新版 `skills/*.md` |
| 升級已有專案 | `specify init . --here --force` | 手動覆蓋 commands 目錄的 `.md` 檔案 |

### 4. Extension 支援

兩個版本對 `.specify/extensions.yml` 的 hooks 系統支援**完全相同**——
`before_tasks`、`after_tasks`、`before_implement`、`after_implement` 都有支援。

Extension 的 Python hook 執行（非 slash command hooks）仍需要原版 CLI。

---

## 常見問題

**Q：我已經用原版 CLI 初始化了專案，可以改用 Pure Skill 版嗎？**

可以。只需把 `skills/*.md` 複製到你的 agent commands 目錄（覆蓋原來的），`specs/` 和 `.specify/memory/` 的資料完全相容，不需要遷移。

---

**Q：Pure Skill 版找不到 feature 目錄怎麼辦？**

每個 skill 透過 `git branch --show-current` 偵測當前分支，並假設 feature 目錄是 `specs/{BRANCH}`。如果你的分支名稱不符合 `[0-9]+-[a-z0-9-]+` 格式，AI agent 會掃描 `specs/` 下的子目錄，或直接問你。

---

**Q：兩個版本產生的 spec.md / plan.md / tasks.md 格式一樣嗎？**

**是**，格式完全相容。可以用原版 CLI 建立 spec，再用 Pure Skill 版執行 plan，或反過來，沒有問題。

---

**Q：我可以同時在同個專案保留兩個版本嗎？**

可以。Pure Skill 的 commands 和原版 CLI 安裝的 commands 是同一套檔案（就在 agent 的 commands 目錄裡），後者覆蓋前者即可。`scripts/` 目錄保留也不影響 Pure Skill 的運作，因為它根本不呼叫那些腳本。

---

**Q：為什麼 Pure Skill 的 `skills/` 沒有像 CLI 版那樣提供 spec-template.md 這些模板檔案？**

Pure Skill 的設計原則是**零外部依賴**：所有需要的模板結構都直接嵌在每個 skill 的 `.md` 檔案裡。這樣複製幾個 `.md` 檔案就夠了，不需要附帶整個 `templates/` 目錄。
