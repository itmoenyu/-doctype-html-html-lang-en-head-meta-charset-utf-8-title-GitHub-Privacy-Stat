# API 完整文档

## 基本信息

- **基础 URL**: `https://api.example.com` （开发: `http://localhost:3000`）
- **认证方式**: Bearer Token (JWT)
- **请求格式**: JSON
- **响应格式**: JSON

## 认证

### 注册

```http
POST /auth/register
Content-Type: application/json

{
  "phone": "18888888888",
  "password": "password123",
  "role": "DEVELOPER"  // EMPLOYER, DEVELOPER, AUDITOR
}
```

**响应 (201)**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user_id",
    "phone": "18888888888",
    "role": "DEVELOPER",
    "status": "ACTIVE",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

### 登录

```http
POST /auth/login
Content-Type: application/json

{
  "phone": "18888888888",
  "password": "password123"
}
```

**响应 (200)**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user_id",
    "phone": "18888888888",
    "role": "DEVELOPER",
    "status": "ACTIVE",
    "createdAt": "2024-01-01T00:00:00Z",
    "wallet": {
      "balance": 1000.00,
      "frozenBalance": 0,
      "totalDeposit": 5000.00,
      "totalWithdraw": 4000.00
    }
  }
}
```

### 刷新 Token

```http
POST /auth/refresh
Authorization: Bearer {token}
```

**响应 (200)**
```json
{
  "access_token": "new_token"
}
```

### 获取个人资料

```http
GET /auth/profile
Authorization: Bearer {token}
```

## 用户管理

### 获取用户信息

```http
GET /users/:userId
```

**响应 (200)**
```json
{
  "id": "user_id",
  "phone": "18888888888",
  "nickname": "开发者张三",
  "avatar": "https://...",
  "role": "DEVELOPER",
  "status": "ACTIVE",
  "yearsOfExperience": 5,
  "skills": ["React", "Node.js", "TypeScript"],
  "walletId": "wallet_id"
}
```

### 更新个人资料

```http
PUT /users/:userId
Authorization: Bearer {token}
Content-Type: application/json

{
  "nickname": "高级开发者",
  "avatar": "https://...",
  "skills": ["React", "Vue", "Node.js"],
  "yearsOfExperience": 6,
  "selfIntroduction": "5年前端开发经验..."
}
```

## 订单管理

### 创建订单（发单）

```http
POST /orders
Authorization: Bearer {token}
Content-Type: application/json

{
  "title": "开发一个React Native应用",
  "description": "需要开发一个电商类的移动应用，包括...",
  "budget": 5000.00,
  "deliveryDays": 30,
  "category": "mobile_app",
  "tags": ["React Native", "iOS", "Android"],
  "attachments": ["https://...design.pdf"]
}
```

**响应 (201)**
```json
{
  "id": "order_id",
  "title": "开发一个React Native应用",
  "description": "需要开发一个电商类的移动应用...",
  "budget": 5000.00,
  "employerId": "user_id",
  "status": "OPEN",
  "deliveryDays": 30,
  "dueDate": "2024-02-15T00:00:00Z",
  "category": "mobile_app",
  "tags": ["React Native", "iOS", "Android"],
  "createdAt": "2024-01-16T00:00:00Z"
}
```

**说明**: 
- 创建订单时，预算金额将从雇主钱包中冻结
- 如果余额不足，返回 400 错误

### 获取订单列表

```http
GET /orders?status=OPEN&category=mobile_app&page=1&limit=20
```

**查询参数**
- `status`: 订单状态筛选 (OPEN/IN_PROGRESS/SUBMITTED/COMPLETED等)
- `category`: 分类筛选
- `page`: 分页页码，默认1
- `limit`: 每页数量，默认20

**响应 (200)**
```json
{
  "data": [
    {
      "id": "order_id",
      "title": "...",
      "budget": 5000.00,
      "status": "OPEN",
      "employer": {
        "id": "employer_id",
        "nickname": "小王",
        "avatar": "https://..."
      },
      "createdAt": "2024-01-16T00:00:00Z"
    }
  ],
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
}
```

### 获取订单详情

```http
GET /orders/:orderId
```

**响应 (200)**
```json
{
  "id": "order_id",
  "title": "开发一个React Native应用",
  "description": "需要开发一个电商类的移动应用...",
  "budget": 5000.00,
  "status": "OPEN",
  "employerId": "employer_id",
  "deliveryDays": 30,
  "dueDate": "2024-02-15T00:00:00Z",
  "employer": {
    "id": "employer_id",
    "nickname": "小王",
    "phone": "18800001111",
    "avatar": "https://..."
  },
  "developer": null,
  "transactions": [
    {
      "id": "tx_id",
      "type": "ESCROW",
      "amount": 5000.00,
      "status": "SUCCESS",
      "createdAt": "2024-01-16T00:00:00Z"
    }
  ]
}
```

### 接单

```http
POST /orders/:orderId/accept
Authorization: Bearer {token}
```

**响应 (200)**
```json
{
  "id": "order_id",
  "status": "IN_PROGRESS",
  "developerId": "developer_id",
  "developer": {
    "id": "developer_id",
    "nickname": "张三",
    "avatar": "https://..."
  }
}
```

**说明**:
- 只能接受状态为 OPEN 的订单
- 接单后订单状态变为 IN_PROGRESS

### 提交交付

```http
POST /orders/:orderId/submit
Authorization: Bearer {token}
Content-Type: application/json

{
  "attachments": [
    "https://...source_code.zip",
    "https://...documentation.pdf"
  ]
}
```

**响应 (200)**
```json
{
  "id": "order_id",
  "status": "SUBMITTED",
  "attachments": ["https://...source_code.zip", "https://...documentation.pdf"]
}
```

### 完成订单

```http
POST /orders/:orderId/complete
Authorization: Bearer {token}
```

**响应 (200)**
```json
{
  "id": "order_id",
  "status": "COMPLETED",
  "completedAt": "2024-01-20T10:30:00Z"
}
```

**说明**:
- 只有订单雇主可以完成订单
- 完成后，冻结资金释放给开发者

### 取消订单

```http
POST /orders/:orderId/cancel
Authorization: Bearer {token}
```

**响应 (200)**
```json
{
  "id": "order_id",
  "status": "CANCELLED",
  "canceledAt": "2024-01-16T10:00:00Z"
}
```

## 钱包管理

### 获取钱包信息

```http
GET /wallet
Authorization: Bearer {token}
```

**响应 (200)**
```json
{
  "id": "wallet_id",
  "userId": "user_id",
  "balance": 2500.00,
  "frozenBalance": 5000.00,
  "totalDeposit": 10000.00,
  "totalWithdraw": 2500.00,
  "currency": "CNY"
}
```

### 获取交易记录

```http
GET /wallet/transactions?page=1&limit=20
Authorization: Bearer {token}
```

**响应 (200)**
```json
{
  "data": [
    {
      "id": "tx_id",
      "type": "DEPOSIT",
      "amount": 1000.00,
      "balance": 2500.00,
      "status": "SUCCESS",
      "description": "支付宝充值 ¥1000",
      "createdAt": "2024-01-20T10:30:00Z"
    },
    {
      "id": "tx_id_2",
      "type": "ESCROW",
      "amount": 5000.00,
      "status": "SUCCESS",
      "description": "订单 xxx 资金冻结",
      "createdAt": "2024-01-16T00:00:00Z"
    }
  ],
  "pagination": {
    "total": 50,
    "page": 1,
    "limit": 20,
    "pages": 3
  }
}
```

### 发起充值

```http
POST /wallet/deposit
Authorization: Bearer {token}
Content-Type: application/json

{
  "amount": 1000.00
}
```

**响应 (200)**
```json
{
  "transactionId": "tx_id",
  "paymentParams": {
    "app_id": "2021002197633544",
    "method": "alipay.trade.app.pay",
    "timestamp": "20240116103000",
    "biz_content": "...",
    "sign": "..."
  }
}
```

**说明**:
- 返回支付参数，客户端使用支付宝SDK调起支付
- 可支付金额范围: 0.01 ~ 1,000,000

### 发起提现

```http
POST /wallet/withdraw
Authorization: Bearer {token}
Content-Type: application/json

{
  "amount": 500.00,
  "bankData": {
    "bankAccount": "6217002200058000121",
    "bankName": "中国银行",
    "accountHolder": "张三"
  }
}
```

**响应 (200)**
```json
{
  "id": "withdrawal_id",
  "userId": "user_id",
  "amount": 500.00,
  "status": "PENDING",
  "bankAccount": "6217002200058000121",
  "bankName": "中国银行",
  "createdAt": "2024-01-20T10:30:00Z"
}
```

**说明**:
- 提现需要绑定银行账户
- 提现金额不能超过可用余额

## 支付接口

### 支付宝异步通知（Webhook）

```http
POST /payment/alipay/notify
Content-Type: application/x-www-form-urlencoded

notify_id=xxxxx
notify_type=trade_status_sync_notification
out_trade_no=tx_id
trade_no=2024011650000000001111111111
trade_status=TRADE_SUCCESS
total_amount=1000.00
...
```

**说明**:
- 支付宝在支付完成后会异步调用此接口
- 后端需要验证签名，确保安全
- 需要返回 JSON: `{"code": "SUCCESS"}` 或 `{"code": "FAIL"}`

## 错误处理

所有错误响应都遵循以下格式:

```json
{
  "statusCode": 400,
  "message": "错误信息",
  "error": "BadRequest"
}
```

### 常见错误码

| 状态码 | 错误信息 | 说明 |
|------|--------|------|
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未授权，需要登录 |
| 403 | Forbidden | 禁止访问，权限不足 |
| 404 | Not Found | 资源不存在 |
| 422 | Unprocessable Entity | 业务逻辑错误 |
| 500 | Internal Server Error | 服务器错误 |

## 速率限制

- 普通请求: 100 req/min per IP
- 支付相关: 10 req/min per user
- 登录: 5 attempts/min per phone

## 分页

所有列表接口都使用相同的分页参数:

```
GET /endpoint?page=1&limit=20
```

**参数**
- `page`: 页码，从1开始，默认为1
- `limit`: 每页条数，默认20，最大100

**响应格式**
```json
{
  "data": [...],
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
}
```
