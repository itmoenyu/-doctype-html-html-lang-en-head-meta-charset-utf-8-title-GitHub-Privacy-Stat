# 本地开发指南

## 项目初始化

### 1. 克隆项目

```bash
git clone <repository-url>
cd outsourcing-platform
```

### 2. 后端环境配置

```bash
cd backend

# 安装依赖
npm install

# 复制环境变量文件
cp .env.example .env

# 编辑 .env，配置本地开发环境
```

**后端 .env 示例**
```env
# Database
DATABASE_URL=mysql://root:password@127.0.0.1:3306/outsourcing_dev

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379

# JWT
JWT_SECRET=dev-secret-key
JWT_EXPIRE_IN=7d

# Application
NODE_ENV=development
PORT=3000
API_URL=http://localhost:3000

# Alipay (沙箱)
ALIPAY_APP_ID=2021002197633544
ALIPAY_PRIVATE_KEY=your-dev-private-key
ALIPAY_PUBLIC_KEY=your-dev-public-key
ALIPAY_GATEWAY_URL=https://openapi.alipaydev.com/gateway.do
```

### 3. 数据库初始化

**使用 Docker 快速启动 MySQL 和 Redis**

```bash
docker run -d \
  --name outsourcing-mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=outsourcing_dev \
  -p 3306:3306 \
  mysql:8.0

docker run -d \
  --name outsourcing-redis \
  -p 6379:6379 \
  redis:7-alpine
```

**创建数据库表**

```bash
cd backend

# 使用 Prisma 迁移
npm run prisma:migrate

# 启动 Prisma Studio 查看数据库
npm run prisma:studio
# 打开 http://localhost:5555
```

### 4. 启动后端开发服务器

```bash
cd backend
npm run start:dev
```

服务器将在 `http://localhost:3000` 启动，并启用热重载。

### 5. 移动端环境配置

```bash
cd mobile

# 安装依赖
npm install

# 复制环境变量文件
echo "REACT_APP_API_URL=http://localhost:3000" > .env

# 安装 Pods (iOS)
cd ios && pod install && cd ..
```

### 6. 启动移动端开发

**在 Android 模拟器上运行**
```bash
npm run android
```

**在 iOS 模拟器上运行**
```bash
npm run ios
```

**启动开发服务器（不自动打开应用）**
```bash
npm start
```

## API 开发工作流

### 创建新的 API 端点

#### 1. 定义数据模型（如需新表）

编辑 `backend/prisma/schema.prisma`:

```prisma
model NewTable {
  id        String   @id @default(cuid())
  name      String
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  createdAt DateTime @default(now())
  
  @@index([userId])
}
```

#### 2. 执行数据库迁移

```bash
cd backend
npm run prisma:migrate
# 按提示输入迁移名称
```

#### 3. 创建新模块

```bash
# 在 src/modules 下创建新模块
mkdir -p src/modules/new-feature/{dto}

# 创建服务
touch src/modules/new-feature/new-feature.service.ts

# 创建控制器
touch src/modules/new-feature/new-feature.controller.ts

# 创建模块
touch src/modules/new-feature/new-feature.module.ts

# 创建 DTO
touch src/modules/new-feature/dto/create-new-feature.dto.ts
```

#### 4. 实现服务和控制器

`src/modules/new-feature/new-feature.service.ts`:
```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/common/prisma/prisma.service';

@Injectable()
export class NewFeatureService {
  constructor(private prisma: PrismaService) {}

  async create(data: any) {
    return this.prisma.newTable.create({ data });
  }

  async findAll() {
    return this.prisma.newTable.findMany();
  }

  async findById(id: string) {
    return this.prisma.newTable.findUnique({ where: { id } });
  }
}
```

`src/modules/new-feature/new-feature.controller.ts`:
```typescript
import { Controller, Post, Get, Body, Param, UseGuards } from '@nestjs/common';
import { NewFeatureService } from './new-feature.service';
import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard';

@Controller('new-feature')
export class NewFeatureController {
  constructor(private service: NewFeatureService) {}

  @Post()
  @UseGuards(JwtAuthGuard)
  create(@Body() data: any) {
    return this.service.create(data);
  }

  @Get()
  findAll() {
    return this.service.findAll();
  }

  @Get(':id')
  findById(@Param('id') id: string) {
    return this.service.findById(id);
  }
}
```

#### 5. 注册模块

编辑 `src/app.module.ts`，导入新模块:

```typescript
import { NewFeatureModule } from './modules/new-feature/new-feature.module';

@Module({
  imports: [
    // ... 其他模块
    NewFeatureModule,
  ],
})
export class AppModule {}
```

### 测试 API

使用 Postman 或 cURL 测试 API:

```bash
# 注册
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "18888888888",
    "password": "password123",
    "role": "DEVELOPER"
  }'

# 登录
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "18888888888",
    "password": "password123"
  }'

# 获取个人资料（需要 token）
curl -X GET http://localhost:3000/auth/profile \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Postman 使用建议**
1. 导入 API 文档到 Postman
2. 在 Postman 中创建环境变量 `API_URL=http://localhost:3000`
3. 在请求中使用 `{{API_URL}}`

### WebSocket 开发

在 `src/modules/notification` 中实现 WebSocket 连接:

```typescript
import { WebSocketGateway, WebSocketServer, OnGatewayConnection } from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: { origin: '*' },
})
export class NotificationGateway implements OnGatewayConnection {
  @WebSocketServer() server: Server;

  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }

  notifyOrderUpdate(userId: string, orderData: any) {
    this.server.to(`user:${userId}`).emit('order:updated', orderData);
  }
}
```

## 移动端开发工作流

### 创建新页面

```bash
cd mobile/src/screens
touch NewScreen.tsx
```

**页面模板**
```typescript
import React from 'react';
import { View, StyleSheet } from 'react-native';
import { Text, Button } from 'react-native-paper';

export const NewScreen = ({ navigation }: any) => {
  return (
    <View style={styles.container}>
      <Text variant="headlineLarge">新页面</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
});
```

### 状态管理

使用 Redux Toolkit 管理全局状态:

```typescript
import { createSlice } from '@reduxjs/toolkit';

const mySlice = createSlice({
  name: 'myFeature',
  initialState: { data: null },
  reducers: {
    setData: (state, action) => {
      state.data = action.payload;
    },
  },
});

export const { setData } = mySlice.actions;
```

### API 集成

使用 RTK Query 处理网络请求:

```typescript
// 在 apiSlice.ts 中添加
myQuery: builder.query({
  query: (id) => `/endpoint/${id}`,
}),

// 在组件中使用
const { data } = useMyQueryQuery(id);
```

### 表单验证

使用 React Hook Form + Yup:

```typescript
import { useForm, Controller } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object().shape({
  name: yup.string().required('名称必填'),
});

const { control, handleSubmit } = useForm({
  resolver: yupResolver(schema),
});
```

## 调试技巧

### 后端调试

**VS Code 调试配置**

创建 `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Backend",
      "program": "${workspaceFolder}/backend/src/main.ts",
      "outFiles": ["${workspaceFolder}/backend/dist/**/*.js"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "preLaunchTask": "npm: start:dev"
    }
  ]
}
```

**Prisma Studio**
```bash
npm run prisma:studio
# 打开 http://localhost:5555，可视化编辑数据库
```

### 移动端调试

**React Native Debugger**
```bash
# 启动
react-native start

# 在应用中摇晃设备，点击 "Debug JS Remotely"
# 打开 React Native Debugger 进行调试
```

**使用 console.log**
```bash
# 查看 console 输出
npx react-native log-android  # Android
npx react-native log-ios      # iOS
```

## 常见开发任务

### 添加依赖包

```bash
# 后端
cd backend
npm install package-name

# 移动端
cd mobile
npm install package-name
```

### 更新依赖

```bash
# 检查可更新的包
npm outdated

# 更新所有包
npm update

# 审计安全漏洞
npm audit
npm audit fix
```

### 代码格式化

```bash
# 后端
cd backend
npm run format  # Prettier
npm run lint    # ESLint

# 移动端
cd mobile
npm run lint
```

## 性能优化建议

### 后端

- 使用数据库索引加快查询
- 实现 Redis 缓存
- 使用 Pagination 分页大数据集
- 异步处理长期任务（使用队列）

### 移动端

- 使用 React.memo 避免不必要重渲染
- 使用 useCallback 优化回调函数
- 使用 FlatList 而非 ScrollView 渲染长列表
- 启用 Hermes 引擎提升性能

## 提交 Git

```bash
# 创建功能分支
git checkout -b feature/new-feature

# 提交更改
git add .
git commit -m "feat: add new feature"

# 推送并创建 PR
git push origin feature/new-feature
```

**Commit Message 规范**
- feat: 新功能
- fix: 修复
- docs: 文档
- style: 代码格式
- refactor: 重构
- perf: 性能
- test: 测试

---

祝开发顺利！ 🚀
