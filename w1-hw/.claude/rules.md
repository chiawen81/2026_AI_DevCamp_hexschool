# 專案開發規則

## API 規範
- 所有 API 回應統一使用 `{ data, error, message }` 格式
- 錯誤代碼使用 SCREAMING_SNAKE_CASE（如 `STOCK_INSUFFICIENT`）
- 路由路徑使用 kebab-case（如 `/api/admin/products`）

## 命名規範
- JS 變數／函數：camelCase
- 路由檔：camelCase + Routes 後綴
- Middleware 檔：camelCase + Middleware 後綴
- 資料庫欄位：snake_case
- EJS 模板 / 前端 JS：kebab-case

## 禁止操作
- 禁止直接修改 `.env` 檔案
- 禁止在路由層寫業務邏輯以外的 SQL（保持在 route handler 內）
- 多筆寫入必須使用 `db.transaction()` 包裹

## 模組系統
- 後端一律使用 CommonJS（`require` / `module.exports`）
- 前端不使用打包工具，透過 `<script>` 載入全域變數

## 計畫歸檔
- 功能開發前在 `docs/plans/` 建立計畫文件
- 命名格式：`YYYY-MM-DD-<feature-name>.md`
- 完成後移至 `docs/plans/archive/`