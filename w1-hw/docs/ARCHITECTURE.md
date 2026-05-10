# 系統架構

## 目錄結構

```
w1-hw/
├── server.js              # 啟動入口：監聽 port、驗證 JWT_SECRET
├── app.js                 # Express 應用：middleware + 路由掛載
├── swagger-config.js      # Swagger/OpenAPI 定義設定
├── generate-openapi.js    # 產生 openapi.json 的腳本
├── vitest.config.js       # 測試框架設定
├── .env.example           # 環境變數範例
│
├── src/
│   ├── database.js        # SQLite 初始化、建表、種子資料、匯出 db 實例
│   ├── middleware/
│   │   ├── sessionMiddleware.js  # 從 X-Session-Id header 設定 req.sessionId
│   │   ├── authMiddleware.js     # 驗證 JWT Bearer token，設定 req.user
│   │   ├── adminMiddleware.js    # 檢查 req.user.role === 'admin'
│   │   └── errorHandler.js      # 全域錯誤處理，隱藏 500 內部細節
│   └── routes/
│       ├── authRoutes.js         # POST /register, POST /login, GET /profile
│       ├── productRoutes.js      # GET /products, GET /products/:id
│       ├── cartRoutes.js         # CRUD /cart（支援 JWT + sessionId 雙模式）
│       ├── orderRoutes.js        # CRUD /orders + PATCH /:id/pay
│       ├── adminProductRoutes.js # 管理員 CRUD /admin/products
│       ├── adminOrderRoutes.js   # 管理員 GET /admin/orders
│       └── pageRoutes.js         # EJS 頁面路由
│
├── views/
│   ├── layouts/
│   │   ├── front.ejs      # 前台 layout（包含 head、header、footer）
│   │   └── admin.ejs      # 後台 layout（包含 admin-header、admin-sidebar）
│   ├── pages/
│   │   ├── index.ejs      # 首頁
│   │   ├── product-detail.ejs
│   │   ├── cart.ejs
│   │   ├── checkout.ejs
│   │   ├── login.ejs      # 登入 + 註冊頁
│   │   ├── orders.ejs
│   │   ├── order-detail.ejs
│   │   ├── 404.ejs
│   │   └── admin/
│   │       ├── products.ejs
│   │       └── orders.ejs
│   └── partials/
│       ├── head.ejs        # <head> 標籤（CSS、字型）
│       ├── header.ejs      # 前台導覽列
│       ├── footer.ejs
│       ├── admin-header.ejs
│       ├── admin-sidebar.ejs
│       └── notification.ejs  # Toast 通知 HTML
│
├── public/
│   ├── css/
│   │   ├── input.css      # Tailwind 來源（含自訂色彩變數）
│   │   └── output.css     # 建置產出（已 gitignore）
│   ├── js/
│   │   ├── api.js         # apiFetch() 封裝：自動帶入 auth header、處理 401
│   │   ├── auth.js        # auth 物件：token/user 存取、sessionId 生成
│   │   ├── header-init.js # 頁面載入時初始化 header 狀態與購物車數量
│   │   ├── notification.js # notification.show(msg, type) Toast 通知
│   │   └── pages/         # 各頁面 Vue 3 App（CDN）
│   │       ├── index.js
│   │       ├── login.js
│   │       ├── cart.js
│   │       ├── checkout.js
│   │       ├── product-detail.js
│   │       ├── orders.js
│   │       ├── order-detail.js
│   │       ├── admin-products.js
│   │       └── admin-orders.js
│   └── stylesheets/
│       └── style.css      # 額外自訂 CSS（少量）
│
└── tests/
    ├── setup.js            # 測試輔助：getAdminToken()、registerUser()
    ├── auth.test.js
    ├── products.test.js
    ├── cart.test.js
    ├── orders.test.js
    ├── adminProducts.test.js
    └── adminOrders.test.js
```

---

## 啟動流程

```
node server.js
  └─ 檢查 JWT_SECRET（未設定則退出）
  └─ require('./app')
       └─ require('dotenv').config()
       └─ require('./src/database')        ← 建立 database.sqlite、建表、寫入種子資料
       └─ Express 設定（view engine: ejs、static: public/）
       └─ 掛載 middleware（cors、json、urlencoded、sessionMiddleware）
       └─ 掛載 API 路由（/api/auth、/api/products、/api/cart、/api/orders、
                         /api/admin/products、/api/admin/orders）
       └─ 掛載頁面路由（/）
       └─ 404 handler
       └─ errorHandler
  └─ app.listen(PORT)  ← 預設 port 3001
```

---

## 資料庫 Schema

**users**

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | TEXT PK | UUID |
| email | TEXT UNIQUE | 登入帳號 |
| password_hash | TEXT | bcrypt hash |
| name | TEXT | 顯示名稱 |
| role | TEXT | `'user'` 或 `'admin'` |
| created_at | TEXT | `datetime('now')` |

**products**

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | TEXT PK | UUID |
| name | TEXT | 商品名稱 |
| description | TEXT | 商品描述 |
| price | INTEGER | 售價（需 > 0） |
| stock | INTEGER | 庫存（需 >= 0） |
| image_url | TEXT | 圖片網址 |
| created_at / updated_at | TEXT | 時間戳 |

**cart_items**

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | TEXT PK | UUID |
| session_id | TEXT | 訪客識別（X-Session-Id） |
| user_id | TEXT FK | 登入用戶（FK → users.id） |
| product_id | TEXT FK | FK → products.id |
| quantity | INTEGER | 需 > 0 |

**orders**

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | TEXT PK | UUID |
| order_no | TEXT UNIQUE | `ORD-YYYYMMDD-XXXXX` |
| user_id | TEXT FK | FK → users.id |
| recipient_name | TEXT | 收件人姓名 |
| recipient_email | TEXT | 收件人 Email |
| recipient_address | TEXT | 收件地址 |
| total_amount | INTEGER | 總金額 |
| status | TEXT | `'pending'`、`'paid'`、`'failed'` |
| created_at | TEXT | 時間戳 |

**order_items**

| 欄位 | 型別 | 說明 |
|------|------|------|
| id | TEXT PK | UUID |
| order_id | TEXT FK | FK → orders.id |
| product_id | TEXT | 下單時商品 id |
| product_name | TEXT | 快照商品名稱 |
| product_price | INTEGER | 快照商品價格 |
| quantity | INTEGER | 數量 |

---

## API 路由總覽

### 認證（/api/auth）

| Method | Path | Auth | 說明 |
|--------|------|------|------|
| POST | /api/auth/register | — | 註冊，回傳 user + token（201） |
| POST | /api/auth/login | — | 登入，回傳 user + token（200） |
| GET | /api/auth/profile | JWT | 取得個人資料（200） |

### 商品（/api/products）

| Method | Path | Auth | 說明 |
|--------|------|------|------|
| GET | /api/products | — | 商品列表（支援 page、limit 分頁） |
| GET | /api/products/:id | — | 單一商品詳情 |

### 購物車（/api/cart）

| Method | Path | Auth | 說明 |
|--------|------|------|------|
| GET | /api/cart | JWT 或 sessionId | 查看購物車（含 total） |
| POST | /api/cart | JWT 或 sessionId | 加入商品（已在車則累加數量） |
| PATCH | /api/cart/:itemId | JWT 或 sessionId | 更新數量 |
| DELETE | /api/cart/:itemId | JWT 或 sessionId | 移除項目 |

### 訂單（/api/orders）

| Method | Path | Auth | 說明 |
|--------|------|------|------|
| POST | /api/orders | JWT | 從購物車建立訂單（交易、扣庫存、清購物車） |
| GET | /api/orders | JWT | 自己的訂單列表 |
| GET | /api/orders/:id | JWT | 訂單詳情 |
| PATCH | /api/orders/:id/pay | JWT | 模擬付款（action: success/fail） |

### 管理員 - 商品（/api/admin/products）

| Method | Path | Auth | 說明 |
|--------|------|------|------|
| GET | /api/admin/products | JWT + admin | 商品列表（支援分頁） |
| POST | /api/admin/products | JWT + admin | 新增商品（201） |
| PUT | /api/admin/products/:id | JWT + admin | 更新商品 |
| DELETE | /api/admin/products/:id | JWT + admin | 刪除商品（有 pending 訂單則 409） |

### 管理員 - 訂單（/api/admin/orders）

| Method | Path | Auth | 說明 |
|--------|------|------|------|
| GET | /api/admin/orders | JWT + admin | 所有訂單（支援 status 篩選 + 分頁） |
| GET | /api/admin/orders/:id | JWT + admin | 訂單詳情（含用戶資訊） |

---

## 前端頁面路由（pageRoutes）

| Path | Layout | 說明 |
|------|--------|------|
| GET / | front | 首頁（商品列表） |
| GET /products/:id | front | 商品詳情 |
| GET /cart | front | 購物車 |
| GET /checkout | front | 結帳 |
| GET /login | front | 登入 / 註冊 |
| GET /orders | front | 我的訂單 |
| GET /orders/:id | front | 訂單詳情 |
| GET /admin/products | admin | 管理員商品管理 |
| GET /admin/orders | admin | 管理員訂單管理 |
