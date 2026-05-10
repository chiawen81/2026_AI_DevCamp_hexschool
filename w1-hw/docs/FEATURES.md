# 功能清單

## 使用者認證

### 註冊（POST /api/auth/register）
- 必填欄位：`email`、`password`（最短 6 字元）、`name`
- 以正規表達式驗證 email 格式（`/^[^\s@]+@[^\s@]+\.[^\s@]+$/`）
- 密碼以 `bcrypt`（10 rounds）hash 後儲存
- Email 重複時回傳 409 CONFLICT
- 成功回傳 201，附帶 `user`（id、email、name、role）與 JWT token

### 登入（POST /api/auth/login）
- 必填欄位：`email`、`password`
- Email 或密碼錯誤皆回傳 401（不區分兩者，避免資訊洩漏）
- 成功回傳 200，附帶 `user` 與 JWT token（有效期 7 天）

### 個人資料（GET /api/auth/profile）
- 需要 JWT Bearer token
- 回傳 `id`、`email`、`name`、`role`、`created_at`

### 角色
- 角色分 `user`（一般用戶）與 `admin`（管理員）
- 一般用戶透過 register 建立，role 固定為 `user`
- 管理員由種子資料建立，帳號由 `.env` 的 `ADMIN_EMAIL` / `ADMIN_PASSWORD` 控制

---

## 商品瀏覽

### 商品列表（GET /api/products）
- 支援分頁：query string `page`（預設 1）、`limit`（預設 10，最大 100）
- 回傳 `{ products, total, page, limit, totalPages }`

### 商品詳情（GET /api/products/:id）
- 找不到時回傳 404

---

## 購物車

購物車支援**兩種模式**：
- **訪客模式**：以 `X-Session-Id` header（UUID）識別
- **登入模式**：以 JWT 識別 `user_id`

兩種模式共用相同路由，由 `dualAuth` middleware 決定使用 `session_id` 或 `user_id` 作為條件。

### 查看購物車（GET /api/cart）
- 回傳 `{ items, total }`
- 每個 item 包含 `id`、`product_id`、`quantity`、`product`（name、price、stock、image_url）
- `total` 為所有 `price * quantity` 的加總

### 加入購物車（POST /api/cart）
- 必填：`productId`、`quantity`（預設 1）
- 商品不存在回傳 404
- 購物車已有相同商品時，累加數量（而非新增一筆）
- 加總數量超過庫存時回傳 400 STOCK_INSUFFICIENT

### 更新數量（PATCH /api/cart/:itemId）
- 必填：`quantity`（需 >= 1）
- 超過庫存時回傳 400
- 項目不屬於當前用戶/session 時回傳 404

### 移除項目（DELETE /api/cart/:itemId）
- 項目不屬於當前用戶/session 時回傳 404

---

## 訂單

所有訂單路由需 JWT Bearer token，訪客無法建立訂單。

### 建立訂單（POST /api/orders）
- 必填：`recipientName`、`recipientEmail`（驗證格式）、`recipientAddress`
- 購物車為空時回傳 400 CART_EMPTY
- 任一商品庫存不足時回傳 400（列出商品名稱）
- 以 SQLite **transaction** 原子執行：
  1. 寫入 `orders` 記錄
  2. 寫入 `order_items`（快照商品名稱與價格）
  3. 扣減每個商品的庫存
  4. 清空該用戶的購物車
- 訂單編號格式：`ORD-YYYYMMDD-XXXXX`（5 碼 UUID 首段大寫）
- 成功回傳 201

### 訂單列表（GET /api/orders）
- 只回傳當前用戶的訂單
- 依 `created_at` 降序排列
- 每筆回傳：`id`、`order_no`、`total_amount`、`status`、`created_at`

### 訂單詳情（GET /api/orders/:id）
- 驗證訂單屬於當前用戶，否則 404
- 回傳完整訂單欄位 + `items` 陣列

### 模擬付款（PATCH /api/orders/:id/pay）
- 必填：`action`（`'success'` 或 `'fail'`）
- 訂單 `status` 必須為 `pending`，否則回傳 400 INVALID_STATUS
- `action: 'success'` → 狀態改為 `paid`
- `action: 'fail'` → 狀態改為 `failed`
- 回傳更新後的完整訂單 + items

---

## 管理員 - 商品管理

所有路由需 JWT + `role === 'admin'`。

### 商品列表（GET /api/admin/products）
- 支援分頁

### 新增商品（POST /api/admin/products）
- 必填：`name`、`price`（需 > 0）
- 選填：`description`、`stock`（需 >= 0，預設 0）、`image_url`
- 成功回傳 201

### 更新商品（PUT /api/admin/products/:id）
- 可更新：`name`、`description`、`price`、`stock`、`image_url`
- 驗證與新增相同

### 刪除商品（DELETE /api/admin/products/:id）
- 若有 `status = 'pending'` 的訂單包含此商品，回傳 409（保護訂單完整性）
- 否則直接刪除

---

## 管理員 - 訂單管理

### 訂單列表（GET /api/admin/orders）
- 可選 query string `status`（`pending`、`paid`、`failed`）篩選
- 支援分頁

### 訂單詳情（GET /api/admin/orders/:id）
- 回傳訂單完整欄位 + `items` + `user`（id、email、name）

---

## 前端頁面功能

各頁使用 Vue 3（CDN）實作互動邏輯，以 `apiFetch()` 呼叫後端 API。

| 頁面 | 功能 |
|------|------|
| 首頁 (`/`) | 商品分頁列表、加入購物車 |
| 商品詳情 (`/products/:id`) | 顯示商品資訊、加入購物車 |
| 購物車 (`/cart`) | 顯示清單、更新數量、刪除、前往結帳 |
| 結帳 (`/checkout`) | 填寫收件資訊並送出訂單 |
| 登入 (`/login`) | 登入 / 註冊 tab 切換，成功後跳轉 |
| 我的訂單 (`/orders`) | 顯示訂單列表 |
| 訂單詳情 (`/orders/:id`) | 顯示訂單資訊、模擬付款按鈕 |
| 管理員商品 (`/admin/products`) | 商品 CRUD、圖片 URL 設定 |
| 管理員訂單 (`/admin/orders`) | 訂單列表與狀態篩選 |

### 前端共用模組

- **`auth`**（`public/js/auth.js`）：管理 localStorage 中的 `flower_token`、`flower_user`、`flower_session_id`
- **`apiFetch`**（`public/js/api.js`）：自動帶入 `Authorization` 與 `X-Session-Id` header，攔截 401 並跳轉至 `/login`
- **`notification`**（`public/js/notification.js`）：`show(message, type)` 顯示 Toast 通知（success / error / warning / info），3 秒後自動消失
- **`header-init.js`**：頁面載入時根據登入狀態顯示登入按鈕或用戶名稱，並更新購物車數量角標
