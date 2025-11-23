# 软件开发外包平台

一个功能完整的手机版软件开发外包平台，包含发单、接单、支付等功能。

## 📋 项目功能

### 核心业务功能
- ✅ **三种身份**：雇主、开发者、审核管理员
- ✅ **发单系统**：雇主发布任务、设置预算和截止期限
- ✅ **接单系统**：开发者浏览、接受订单、提交成果
- ✅ **虚拟资金系统**：账户余额管理、资金冻结与释放
- ✅ **充值提现**：集成支付宝沙箱/正式支付
- ✅ **任务管理**：订单追踪、状态管理、完成验收
- ✅ **审核管理**：管理员审核订单、处理纠纷
- ✅ **实时通知**：WebSocket推送订单、支付、审核通知

### 技术特性
- 📱 React Native + TypeScript（移动端跨平台）
- 🏗️ Nest.js + Prisma（后端架构）
- 💾 MySQL 8.0 + Redis（数据存储）
- 💳 支付宝官方SDK集成（支付处理）
- ☁️ 腾讯云COS（文件存储）
- 📊 Sentry（错误监控）
- 🔐 JWT + bcrypt（身份认证）

## 📁 项目结构

```
.
├── backend/                    # 后端项目
│   ├── prisma/
│   │   └── schema.prisma      # 数据模型定义
│   ├── src/
│   │   ├── modules/           # 业务模块
│   │   │   ├── auth/          # 认证模块
│   │   │   ├── users/         # 用户模块
│   │   │   ├── orders/        # 订单模块
│   │   │   ├── wallet/        # 钱包模块
│   │   │   ├── payment/       # 支付模块
│   │   │   ├── file-upload/   # 文件上传
│   │   │   └── notification/  # 通知模块
│   │   ├── common/            # 通用模块
│   │   │   ├── prisma/        # Prisma服务
│   │   │   ├── guards/        # JWT守卫
│   │   │   └── exceptions/    # 异常处理
│   │   ├── app.module.ts      # 应用模块
│   │   └── main.ts            # 入口文件
│   ├── package.json
│   ├── tsconfig.json
│   └── .env.example           # 环境变量模板
│
├── mobile/                    # 移动端项目
│   ├── src/
│   │   ├── screens/           # 页面组件
│   │   │   ├── LoginScreen    # 登录
│   │   │   ├── HomeScreen     # 首页
│   │   │   └── WalletScreen   # 钱包
│   │   ├── store/             # Redux状态管理
│   │   │   ├── index.ts       # Store配置
│   │   │   ├── apiSlice.ts    # API切片
│   │   │   └── authSlice.ts   # 认证状态
│   │   ├── services/          # 业务服务
│   │   │   └── StorageService # 本地存储
│   │   ├── components/        # 通用组件
│   │   ├── hooks/             # 自定义Hook
│   │   ├── utils/             # 工具函数
│   │   └── types/             # TypeScript类型
│   ├── package.json
│   ├── tsconfig.json
│   └── app.json               # React Native配置
│
├── docs/                      # 文档
│   ├── API.md                 # API文档
│   ├── DEPLOYMENT.md          # 部署指南
│   └── DEVELOPMENT.md         # 开发指南
│
├── LICENSE
└── README.md
```

## 🚀 快速开始

### 前置要求
- Node.js 18+
- MySQL 8.0+
- Redis 6.0+
- 支付宝开放平台账户（测试或正式）

### 后端部署

1. **安装依赖**
```bash
cd backend
npm install
```

2. **配置环境变量**
```bash
cp .env.example .env
# 编辑 .env 文件，配置数据库、Redis、支付宝等信息
```

3. **数据库初始化**
```bash
# 使用 Prisma 进行数据库迁移
npm run prisma:migrate

# 启动 Prisma Studio（可视化数据库管理）
npm run prisma:studio
```

4. **启动开发服务器**
```bash
npm run start:dev
```

服务器将在 `http://localhost:3000` 启动。

### 移动端部署

1. **安装依赖**
```bash
cd mobile
npm install
```

2. **配置环境变量**
```bash
# 创建 .env 文件
echo "REACT_APP_API_URL=http://localhost:3000" > .env
```

3. **运行Android/iOS**
```bash
# Android
npm run android

# iOS
npm run ios

# Web（开发预览）
npm run web
```

## 📊 核心数据模型

### User（用户）
```typescript
- id: 唯一标识
- phone: 手机号（登录标识）
- email: 邮箱
- role: 身份 (EMPLOYER/DEVELOPER/AUDITOR/ADMIN)
- status: 状态 (ACTIVE/INACTIVE/BANNED/DELETED)
- profile: 个人资料、头像、技能等
- wallet: 关联的钱包
```

### Order（订单）
```typescript
- id: 订单ID
- title: 标题
- description: 描述
- budget: 预算（冻结在雇主钱包）
- employerId: 雇主ID
- developerId: 开发者ID（可选）
- status: 订单状态
- deliveryDays: 交付天数
- dueDate: 截止日期
```

### Wallet（钱包）
```typescript
- userId: 用户ID
- balance: 可用余额
- frozenBalance: 冻结余额（订单预算）
- totalDeposit: 累计充值
- totalWithdraw: 累计提现
```

### Transaction（交易）
```typescript
- type: 交易类型 (DEPOSIT/WITHDRAW/ESCROW/RELEASE/REFUND等)
- amount: 金额
- status: 交易状态 (PENDING/SUCCESS/FAILED等)
- externalRef: 支付宝交易号
```

## 🔌 API 端点总览

### 认证
- `POST /auth/register` - 注册账户
- `POST /auth/login` - 登录
- `POST /auth/refresh` - 刷新Token
- `GET /auth/profile` - 获取个人资料

### 订单
- `POST /orders` - 创建订单（雇主）
- `GET /orders` - 获取订单列表
- `GET /orders/:id` - 获取订单详情
- `POST /orders/:id/accept` - 接单（开发者）
- `POST /orders/:id/submit` - 提交交付（开发者）
- `POST /orders/:id/complete` - 完成订单（雇主确认）
- `POST /orders/:id/cancel` - 取消订单（雇主）

### 钱包
- `GET /wallet` - 获取钱包信息
- `POST /wallet/deposit` - 发起充值
- `POST /wallet/withdraw` - 发起提现
- `GET /wallet/transactions` - 交易记录

### 支付
- `POST /payment/deposit` - 创建支付单
- `POST /payment/alipay/notify` - 支付宝异步通知（Webhook）
- `GET /payment/alipay/return` - 支付宝返回URL
- `POST /payment/refund` - 申请退款

详见 `docs/API.md`

## 💳 支付流程

### 充值流程
1. 用户进入充值页，输入金额
2. 后端创建待支付交易记录
3. 返回支付宝支付参数
4. 移动端调起支付宝SDK
5. 用户完成支付
6. 支付宝异步通知后端 `/payment/alipay/notify`
7. 后端验证签名、更新交易状态、增加用户余额

### 订单支付流程（发单时锁定）
1. 雇主发起订单，输入预算
2. 后端检查余额，扣除预算放入冻结账户
3. 开发者接单、完成工作
4. 雇主确认完成
5. 冻结金额释放给开发者账户

### 提现流程
1. 用户申请提现，输入金额和银行账户
2. 后端创建提现请求（待审核）
3. 管理员或自动审核
4. 审核通过后调用支付宝转账接口
5. 更新用户余额、交易状态

## 🔒 安全考虑

- ✅ 密码使用 `bcryptjs` 加密，工作因子为10
- ✅ JWT Token使用 RS256 算法签名，有效期可配置
- ✅ 支付宝签名使用 RSA2，必须验证回调签名
- ✅ 关键操作（大额提现）应添加短信验证码
- ✅ 使用HTTPS传输所有敏感数据
- ✅ 私钥、密钥等敏感信息使用环境变量管理
- ✅ 定期备份数据库，设置数据加密
- ✅ 实现API速率限制，防止暴力攻击
- ✅ 记录审计日志，追踪关键操作

## 📚 文档

- [API文档](./docs/API.md) - 完整API接口文档
- [部署指南](./docs/DEPLOYMENT.md) - 生产环境部署步骤
- [开发指南](./docs/DEVELOPMENT.md) - 本地开发环境配置

## 🚢 部署

### 生产环境建议

**后端**
- 使用 PM2 进程管理
- 使用 Nginx 反向代理
- 宝塔面板快速部署
- 启用 Sentry 错误监控
- 配置 CDN 加速API响应

**移动端**
- Android 打包为 APK/AAB，上传应用商店
- iOS 打包为 IPA，上传 App Store
- 使用 CodePush/EAS 进行热更新

## 📞 技术支持

### 常见问题

**Q: 如何对接真实的支付宝？**
A: 在支付宝开放平台申请应用，获取appId和私钥，配置到 `.env` 文件中，将 ALIPAY_GATEWAY_URL 改为正式环境。

**Q: 如何添加其他支付方式？**
A: 在 `payment.module.ts` 中添加新的支付服务，实现相同的接口即可。

**Q: 如何扩展用户字段？**
A: 修改 `prisma/schema.prisma` 添加字段，运行 `npm run prisma:migrate` 进行迁移。

## 📝 许可证

MIT License

## 👥 贡献

欢迎提交Issue和Pull Request！

---

**开发者们，开始构建吧！** 🎉
