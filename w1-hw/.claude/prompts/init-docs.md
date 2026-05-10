# 初始化專案文件 Prompt
用途：Clone 新專案後，一次性生成所有 AI 協作文件。
使用方式：擇一複製貼入 Claude Code 對話框執行。

---

> 以下是 Prompt 的實際內容，共兩個版本，使用時請擇一直接複製貼上

## 版本一：快速版（Claude 自行撰寫）
適合：快速啟動，內容由 Claude 讀取原始碼後自動生成。

```
請深入閱讀這個專案的所有原始碼，然後依序填寫以下文件的實際內容：

- CLAUDE.md（根目錄）：專案概述、常用指令、關鍵規則、指向 docs/ 的連結，100 行以內
- docs/ARCHITECTURE.md：目錄結構（每個檔案用途）、啟動流程、API 路由總覽
- docs/DEVELOPMENT.md：命名規則、模組系統、環境變數說明、開發流程
- docs/FEATURES.md：現有功能清單與行為描述
- docs/TESTING.md：測試框架（vitest）、執行方式、測試檔案說明
- docs/plan.md：空白即可，這是之後開發計畫用的

撰寫原則：
- 內容必須反映專案實際的程式碼，不可寫概述或假設
- 使用繁體中文
- 每份文件不超過 500 行

```

---

## 版本二：完整版（老師課堂範本）
適合：需要更細緻控制生成流程，含互動確認步驟。

```
建立以下結構。**每份文件都必須極度詳細**，深入閱讀所有原始碼後再撰寫，不可只寫概述或骨架：

```
CLAUDE.md                      # 專案概述 + 常用指令 + 關鍵規則 + @docs 引用
docs/
├── README.md                  # 項目介紹、快速開始、技術棧
├── ARCHITECTURE.md            # 架構、目錄結構、資料流
├── DEVELOPMENT.md             # 開發規範、命名規則、計畫歸檔流程
├── FEATURES.md                # 功能清單與完成狀態（含功能行為描述）
├── TESTING.md                 # 測試規範與指南
├── CHANGELOG.md               # 更新日誌
└── plans/                     # 開發計畫目錄
    └── archive/               # 已完成計畫歸檔
```

### docs 文件詳細度要求

每份文件必須做到以下程度：

撰寫前先檢視專案，找出「關鍵知識點或技術決策」，判斷標準是：
**若開發者不知道這件事，是否會影響其他模組的開發或整合？**
符合此條件的內容才需要明確記錄進文件。

- **ARCHITECTURE.md**：目錄結構（每個檔案的用途）、啟動流程、API 路由總覽表（前綴、檔案、認證、說明）、統一回應格式範例、認證與授權機制（middleware 行為、JWT 參數、有效期）、資料庫 schema（每張表的欄位、型別、約束）、金流/第三方整合的流程描述
- **FEATURES.md**：每個功能區塊須有行為描述段落，不只是端點表格。包含：查詢參數與預設值、請求 body 的必填/選填欄位、業務邏輯（例如購物車累加、訂單扣庫存的 transaction）、錯誤碼與錯誤情境、非標準機制（例如雙模式認證的流程）
- **DEVELOPMENT.md**：命名規則對照表、模組系統說明、新增 API/middleware/DB 的步驟、環境變數表（變數、用途、必要性、預設值）、JSDoc 格式說明與範例
- **TESTING.md**：測試檔案表、執行順序與依賴關係、輔助函式說明、撰寫新測試的步驟與範例、常見陷阱
- **README.md**：技術棧、快速開始（copy-paste 指令）、常用指令表、文件索引表

### CLAUDE.md 範本結構

```markdown
# CLAUDE.md

## 專案概述
{專案名稱} — {技術棧摘要}

## 常用指令
{從 scripts 偵測}

## 關鍵規則
- {依專案特性列出 3-5 條}
- 功能開發使用 docs/plans/ 記錄計畫；完成後移至 docs/plans/archive/

## 詳細文件
- ./docs/README.md — 項目介紹與快速開始
- ./docs/ARCHITECTURE.md — 架構、目錄結構、資料流
- ./docs/DEVELOPMENT.md — 開發規範、命名規則
- ./docs/FEATURES.md — 功能列表與完成狀態
- ./docs/TESTING.md — 測試規範與指南
- ./docs/CHANGELOG.md — 更新日誌
```

### DEVELOPMENT.md 必須包含計畫歸檔流程

```markdown
## 計畫歸檔流程

1. 計畫檔案命名格式：YYYY-MM-DD-<feature-name>.md
2. 計畫文件結構：User Story → Spec → Tasks
3. 功能完成後：移至 docs/plans/archive/
4. 更新 docs/FEATURES.md 和 docs/CHANGELOG.md
```

```

