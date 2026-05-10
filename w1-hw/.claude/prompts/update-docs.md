# 更新維護文件 Prompt
用途：每次功能開發完成後，更新 docs/ 下的文件。
使用方式：功能完成後，複製以下內容貼入 Claude Code 執行。

---

> 以下是 Prompt 的實際內容，使用時直接複製貼上

```
協助負責維護 `docs/` 目錄下的所有文件。

## 文件結構
docs/
├── README.md        # 技術棧、快速開始、文件索引
├── ARCHITECTURE.md  # 架構、路由表、DB Schema、資料流
├── DEVELOPMENT.md   # 開發規範、命名規則、新增功能流程
├── FEATURES.md      # 功能清單與行為描述
├── TESTING.md       # 測試規範
├── CHANGELOG.md     # 更新日誌
└── plans/           # 開發計畫（完成後移至 archive/）


## 工作流程

接到任務時，先確認：
1. 哪些功能/API 有新增或變更？
2. 哪些 docs/ 文件需要更新？

逐一更新對應文件：

### FEATURES.md 更新規則
- 新功能加入功能狀態總覽表，標記 ✅ 完成或 🚧 進行中
- 每個功能區塊寫詳細行為描述（不只是端點表格）：查詢參數、業務邏輯、錯誤情境

### CHANGELOG.md 更新規則
- 格式：`## [版本] - YYYY-MM-DD`，底下用 `### Added / Changed / Fixed / Removed`
- 若尚未定版，放在 `## [Unreleased]` 區塊

### ARCHITECTURE.md 更新規則
- 新增路由時同步更新路由表（頁面路由和 REST API 兩張表）
- 新增 DB 欄位時同步更新 Schema 表格

### 計畫歸檔
- 功能完成後，將 `docs/plans/YYYY-MM-DD-<feature>.md` 移至 `docs/plans/archive/`
- 使用 Bash 執行 `mv` 指令移動檔案

## 撰寫原則

- 使用繁體中文撰寫
- 文件內容以「未來的開發者」為讀者，避免過於簡短的概述
- 每次只更新確實有變更的部分，不要重寫整份文件
```
