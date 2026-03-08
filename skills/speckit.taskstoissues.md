---
description: 將 tasks.md 中的任務轉換為遠端儲存庫上的 GitHub Issues。不需安裝任何 CLI。
tools: ['github/github-mcp-server/issue_write']
---

## 使用者輸入

```text
$ARGUMENTS
```

**必須**在繼續之前考慮使用者輸入（若不為空）。

## 流程說明

### 步驟 1：找到當前 feature 與任務清單

```bash
git branch --show-current
```

- 分支符合 `[0-9]+-[a-z0-9-]+` → `FEATURE_DIR` = `specs/{BRANCH}`
- 否則：掃描 `specs/` 或詢問使用者

任務檔案：`{FEATURE_DIR}/tasks.md`（絕對路徑）。若不存在：請使用者先執行 `/speckit.tasks`。

### 步驟 2：確認 GitHub 遠端

```bash
git config --get remote.origin.url
```

> [!CAUTION]
> **只有在遠端為 GitHub URL 時才繼續**（URL 包含 `github.com`）。
> 若遠端不是 GitHub，立即停止。

### 步驟 3：建立 GitHub Issues

對 `tasks.md` 中的每個任務，使用 GitHub MCP server 在符合遠端 URL 的儲存庫中建立新 Issue。

> [!CAUTION]
> **絕對不可在不符合遠端 URL 的儲存庫中建立 Issues。**
