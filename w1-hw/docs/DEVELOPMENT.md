# 開發指南

## 命名規則

| 對象 | 規則 | 範例 |
|------|------|------|
| JS 變數 / 函數 | camelCase | `cartItems`、`getOwnerCondition` |
| 路由檔案 | camelCase + Routes 後綴 | `authRoutes.js`、`adminProductRoutes.js` |
| Middleware 檔案 | camelCase + Middleware 後綴 | `authMiddleware.js`、`errorHandler.js` |
| 前端頁面 JS | kebab-case | `admin-products.js`、`order-detail.js` |
| EJS 模板 | kebab-case | `product-detail.ejs`、`admin-header.ejs` |
| API 路徑 | kebab-case、REST 慣例 | `/api/admin/products`、`/api/orders/:id/pay` |
| 資料庫欄位 | snake_case | `order_no`、`recipient_name`、`created_at` |
| 訂單編號 | `ORD-YYYYMMDD-XXXXX` | `ORD-20260510-A3F2B` |

---

## 模組系統

所有 Node.js 檔案使用 **CommonJS**（`require` / `module.exports`）。

```js
// 引入
const db = require('../database');
const authMiddleware = require('../middleware/authMiddleware');

// 匯出
module.exports = router;
```

前端 JS（`public/js/`）直接以 `<script>` 標籤載入，使用全域變數（`auth`、`notification`、`apiFetch`），無打包工具。Vue 3 透過 CDN 引入。

---

## 環境變數

參考 `.env.example`：

| 變數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `JWT_SECRET` | **是** | — | JWT 簽名密鑰，未設定則 server 直接退出 |
| `PORT` | 否 | `3001` | Express 監聽 port |
| `BASE_URL` | 否 | `http://localhost:3001` | 伺服器基礎網址 |
| `FRONTEND_URL` | 否 | `http://localhost:3001` | CORS 允許的來源 |
| `ADMIN_EMAIL` | 否 | `admin@hexschool.com` | 種子管理員帳號 |
| `ADMIN_PASSWORD` | 否 | `12345678` | 種子管理員密碼 |
| `ECPAY_MERCHANT_ID` | 否 | `3002607` | ECPay 特店編號（預留） |
| `ECPAY_HASH_KEY` | 否 | — | ECPay Hash Key（預留） |
| `ECPAY_HASH_IV` | 否 | — | ECPay Hash IV（預留） |
| `ECPAY_ENV` | 否 | `staging` | ECPay 環境（預留） |

---

## API 統一回應格式

所有 API 端點回傳相同 JSON 結構：

```json
{
  "data": <物件 | 陣列 | null>,
  "error": <錯誤代碼字串 | null>,
  "message": "繁體中文訊息"
}
```

常見錯誤代碼：

| 代碼 | HTTP 狀態 | 說明 |
|------|-----------|------|
| `VALIDATION_ERROR` | 400 | 欄位缺失或格式錯誤 |
| `UNAUTHORIZED` | 401 | 未登入或 Token 無效 |
| `FORBIDDEN` | 403 | 權限不足（非 admin） |
| `NOT_FOUND` | 404 | 資源不存在 |
| `CONFLICT` | 409 | Email 已註冊 / 商品有 pending 訂單 |
| `STOCK_INSUFFICIENT` | 400 | 庫存不足 |
| `CART_EMPTY` | 400 | 購物車為空 |
| `INVALID_STATUS` | 400 | 訂單狀態不允許此操作 |
| `INTERNAL_ERROR` | 500 | 伺服器內部錯誤 |

---

## 認證機制

### JWT Bearer Token（登入用戶）

在 `Authorization` header 帶入：

```
Authorization: Bearer <token>
```

Token 由 `POST /api/auth/register` 或 `POST /api/auth/login` 取得，有效期 **7 天**。

JWT Payload 結構：

```json
{ "userId": "...", "email": "...", "role": "user | admin" }
```

`authMiddleware` 驗證 token 並在資料庫確認用戶存在，設定 `req.user`。

### Session ID（訪客購物車）

在 `X-Session-Id` header 帶入瀏覽器端自行生成的 UUID：

```
X-Session-Id: <uuid>
```

前端 `auth.getSessionId()` 會從 `localStorage` 讀取或以 `crypto.randomUUID()` 生成。

### 雙模式認證（dualAuth）

購物車路由使用 `dualAuth`：先嘗試驗證 JWT，若無 Authorization header 則改用 sessionId。若兩者都沒有則回傳 401。

---

## 開發流程

1. **修改後端**：執行 `npm run dev:server` 後，修改 `src/` 下的檔案，需手動重啟（或搭配 nodemon）。
2. **修改 Tailwind 樣式**：執行 `npm run dev:css` 監聽 `public/css/input.css` 的變更並自動重建 `output.css`。
3. **測試**：執行 `npm test`。測試使用同一個 `database.sqlite`，測試之間依賴 vitest 的固定執行順序。
4. **API 文件**：執行 `npm run openapi` 產生 `openapi.json`，內容由各路由檔的 JSDoc `@openapi` 標記決定。

---

## 資料庫存取

直接在路由中引入 db 實例並執行 SQL：

```js
const db = require('../database');

// 查詢
const user = db.prepare('SELECT * FROM users WHERE id = ?').get(userId);

// 寫入
db.prepare('INSERT INTO ...').run(...);

// 交易（原子操作）
const tx = db.transaction(() => { /* 多筆寫入 */ });
tx();
```

`src/database.js` 開啟了 WAL 模式（`journal_mode = WAL`）與外鍵約束（`foreign_keys = ON`）。
