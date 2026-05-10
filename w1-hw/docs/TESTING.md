# 測試說明

## 框架

- **vitest** `^2.1.9` — 測試框架
- **supertest** `^7.2.2` — HTTP 請求測試

---

## 執行方式

```bash
npm test
# 等同於執行：vitest run
```

---

## 測試設定（vitest.config.js）

```js
export default defineConfig({
  test: {
    globals: true,          // 不需 import describe/it/expect
    fileParallelism: false, // 關閉平行執行（測試共用同一個 database.sqlite）
    sequence: {
      files: [              // 固定執行順序
        'tests/auth.test.js',
        'tests/products.test.js',
        'tests/cart.test.js',
        'tests/orders.test.js',
        'tests/adminProducts.test.js',
        'tests/adminOrders.test.js',
      ],
    },
    hookTimeout: 10000,
  },
});
```

重要：`fileParallelism: false` 確保測試按序執行，避免 SQLite 並發寫入衝突與測試間資料干擾。

---

## 資料庫行為

測試使用與開發相同的 `database.sqlite`。

`src/database.js` 中有以下邏輯：

```js
const saltRounds = process.env.NODE_ENV === 'test' ? 1 : 10;
```

在 `NODE_ENV=test` 時，bcrypt rounds 降為 1，大幅加快測試速度。

種子資料在 `initializeDatabase()` 呼叫時寫入（若 admin 用戶與商品不存在）。測試結束後資料庫不會自動清除，但 `registerUser()` 每次使用隨機 email，避免衝突。

---

## 輔助函數（tests/setup.js）

### `getAdminToken()`

```js
async function getAdminToken()
```

以種子管理員帳號（`admin@hexschool.com` / `12345678`）登入，回傳 JWT token 字串。

### `registerUser(overrides)`

```js
async function registerUser(overrides = {})
```

以隨機 email 註冊新用戶，回傳 `{ token, user }`。`overrides` 可覆蓋預設值（name、email、password）。

---

## 測試檔說明

### tests/auth.test.js

| 測試案例 | 預期 |
|---------|------|
| 正常註冊 | 201，回傳 token |
| 重複 Email 註冊 | 409 CONFLICT |
| 正常登入 | 200，回傳 token |
| 錯誤密碼登入 | 401 |
| 有效 token 取得個人資料 | 200 |
| 無 token 取得個人資料 | 401 |

### tests/products.test.js

| 測試案例 | 預期 |
|---------|------|
| 取得商品列表 | 200，含 pagination |
| 分頁參數正確運作 | 回傳指定頁數資料 |
| 取得商品詳情 | 200 |
| 不存在商品詳情 | 404 |

### tests/cart.test.js

| 測試案例 | 預期 |
|---------|------|
| 加入商品（訪客 sessionId） | 200 |
| 查看購物車（訪客） | 200，含 items + total |
| 更新數量（訪客） | 200 |
| 移除項目（訪客） | 200 |
| 加入商品（JWT 登入） | 200 |
| 加入不存在商品 | 404 |

### tests/orders.test.js

| 測試案例 | 預期 |
|---------|------|
| 建立訂單（購物車有商品） | 201 |
| 建立訂單（購物車為空） | 400 CART_EMPTY |
| 無 token 建立訂單 | 401 |
| 取得訂單列表 | 200 |
| 取得訂單詳情 | 200 |
| 模擬付款（success / fail） | 200，status 更新 |

### tests/adminProducts.test.js

| 測試案例 | 預期 |
|---------|------|
| 管理員取得商品列表 | 200 |
| 管理員新增商品 | 201 |
| 管理員更新商品 | 200 |
| 管理員刪除商品 | 200 |
| 一般用戶存取 | 403 |
| 無 token 存取 | 401 |

### tests/adminOrders.test.js

| 測試案例 | 預期 |
|---------|------|
| 管理員取得訂單列表 | 200 |
| 以 status 篩選訂單 | 200，只回傳符合 status 的訂單 |
| 管理員取得訂單詳情 | 200，含 user 資訊 |
| 一般用戶存取 | 403 |
