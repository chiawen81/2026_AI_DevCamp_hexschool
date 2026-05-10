# 花卉電商 Demo（w1-hw）

花卉電商網站的後端專案，以 Express + SQLite 實作 REST API，前端使用 EJS 模板 + Vue 3（CDN），樣式使用 Tailwind CSS 4。

## 環境設定

複製 `.env.example` 為 `.env` 並設定以下必填項目：

```bash
cp .env.example .env
```

`.env` 至少需要設定：

```
JWT_SECRET=<任意隨機字串>
```

啟動前若未設定 `JWT_SECRET`，`server.js` 會直接退出並印出錯誤。

## 常用指令

```bash
# 正式啟動（先建置 CSS 再啟動伺服器）
npm start

# 開發模式（分兩個終端分別執行）
npm run dev:server   # 啟動 Express（port 3001）
npm run dev:css      # 監聽 Tailwind CSS 變更

# 執行所有測試
npm test

# 產生 openapi.json（根據路由 jsdoc 標記）
npm run openapi
```

## 資料庫

使用 `better-sqlite3`，首次啟動時自動建立 `database.sqlite`、建表、並寫入種子資料：
- 1 個管理員帳號（email/password 由 `.env` 的 `ADMIN_EMAIL` / `ADMIN_PASSWORD` 控制，預設 `admin@hexschool.com` / `12345678`）
- 8 筆花卉商品

## 文件索引

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — 目錄結構、啟動流程、API 路由總覽
- [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) — 命名規則、模組系統、環境變數、開發流程
- [docs/FEATURES.md](docs/FEATURES.md) — 現有功能清單與行為描述
- [docs/TESTING.md](docs/TESTING.md) — 測試框架、執行方式、測試檔說明
- [docs/plan.md](docs/plan.md) — 開發計畫（留白）
