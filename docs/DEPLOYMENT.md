# 部署指南

## 生产环境部署

### 后端部署（使用宝塔 + PM2）

#### 1. 服务器准备

**系统要求**
- CentOS 7+ 或 Ubuntu 20.04+
- 内存: 2GB 以上
- 磁盘: 20GB 以上

**软件要求**
- Node.js 18+
- MySQL 8.0+
- Redis 6.0+
- Nginx

#### 2. 安装宝塔面板

```bash
# 使用官方安装脚本
curl http://download.bt.cn/install/install_panel.sh | bash
```

访问 IP:8888 进入宝塔面板，按提示初始化。

#### 3. 在宝塔中安装必要软件

在宝塔左侧菜单 "软件商店" 中安装:
- Nginx
- MySQL 8.0
- Redis
- Node.js 18

#### 4. 创建 MySQL 数据库和用户

```bash
# 登录宝塔，进入 MySQL 管理器，创建:
# 数据库: outsourcing_db
# 用户: outsourcing_user
# 密码: (设置复杂密码)
```

#### 5. 克隆项目并配置

```bash
# 在宝塔文件管理中，进入 /home/wwwroot/
# 克隆项目
git clone <your-repo-url> outsourcing-api
cd outsourcing-api/backend

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env
# 编辑 .env，配置数据库、Redis、支付宝等信息
```

.env 配置示例:
```env
DATABASE_URL=mysql://outsourcing_user:password@localhost:3306/outsourcing_db
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
JWT_SECRET=your-super-secret-key-here
ALIPAY_APP_ID=2021002197633544
ALIPAY_PRIVATE_KEY=...
ALIPAY_PUBLIC_KEY=...
API_URL=https://api.example.com
```

#### 6. 数据库迁移

```bash
npm run prisma:migrate
```

#### 7. 编译项目

```bash
npm run build
```

#### 8. 使用 PM2 启动应用

```bash
# 全局安装 PM2
npm install -g pm2

# 启动应用
pm2 start dist/main.js --name "outsourcing-api"

# 设置开机自启
pm2 startup
pm2 save

# 查看日志
pm2 logs outsourcing-api
```

#### 9. 配置 Nginx 反向代理

在宝塔中创建网站配置:

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # SSL证书（宝塔可自动申请 Let's Encrypt）
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";

    # 日志
    access_log /var/log/nginx/api.access.log;
    error_log /var/log/nginx/api.error.log;

    # 限流
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/m;
    limit_req zone=api_limit burst=20 nodelay;

    # 代理到 Node.js
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # WebSocket 支持
        proxy_read_timeout 86400;
    }

    # 支付宝回调
    location /payment/alipay/notify {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### 10. 启用 HTTPS

宝塔提供免费 SSL 证书（Let's Encrypt），在网站配置中启用即可。

#### 11. 监控和告警

**使用 PM2 监控**
```bash
pm2 monit
pm2 web  # 启动 PM2 Web UI
```

**集成 Sentry 错误监控**
- 在 Sentry.io 注册账户
- 创建项目，获取 DSN
- 配置 .env: `SENTRY_DSN=https://...`

### 移动端部署

#### Android 打包

```bash
cd mobile

# 生成签名密钥（首次）
keytool -genkey -v -keystore release.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias release

# 配置签名信息
# 编辑 android/app/build.gradle，添加签名配置

# 打包 APK
cd android
./gradlew assembleRelease

# 输出: android/app/build/outputs/apk/release/app-release.apk
```

#### iOS 打包

```bash
cd mobile/ios
pod install
cd ..

# 打包 IPA
react-native bundle --entry-file index.js --platform ios --dev false --bundle-output ios/main.jsbundle

# 在 Xcode 中打开 ios/outsourcing.xcworkspace
# Product → Archive → Distribute App
```

#### 应用商店上传

**Google Play**
1. 创建 Google Play Developer 账户
2. 上传 APK/AAB
3. 填写应用信息、截图、隐私政策等
4. 提交审核

**App Store**
1. 创建 Apple Developer 账户
2. 上传 IPA
3. 填写应用信息
4. 提交审核

### 热更新配置

使用 CodePush 或 EAS Updates:

**CodePush**
```bash
# 全局安装
npm install -g appcenter-cli

# 登录
appcenter login

# 创建应用
appcenter apps create -d <appDisplayName> -o <osName> -p <platform>

# 推送更新
appcenter codepush release-react -a <owner>/<app>
```

## 监控和维护

### 日志管理

```bash
# 查看 PM2 日志
pm2 logs

# 导出日志
pm2 save
pm2 logs > app.log 2>&1

# 使用 ELK Stack 进行日志管理
# 安装 Elasticsearch、Logstash、Kibana
```

### 性能优化

**后端优化**
- 启用 Redis 缓存
- 使用数据库连接池
- 启用 Gzip 压缩
- 优化数据库查询，添加索引

**移动端优化**
- 启用代码分割
- 使用 ProGuard/R8 代码混淆
- 启用 App Thinning（iOS）

### 备份策略

**数据库备份**
```bash
# 每日自动备份
0 2 * * * mysqldump -u root -p password outsourcing_db > /backup/db_$(date +\%Y\%m\%d).sql

# 定期清理旧备份
find /backup -name "db_*.sql" -mtime +30 -delete
```

**文件备份**
- 配置 COS/OSS 跨域备份
- 启用版本控制

## 故障排查

### 常见问题

**1. 数据库连接失败**
```bash
# 检查 MySQL 状态
systemctl status mysql

# 检查连接
mysql -u user -p -h localhost -D database_name

# 查看错误日志
tail -f /var/log/mysql/error.log
```

**2. PM2 应用崩溃**
```bash
# 查看进程状态
pm2 status

# 查看完整错误日志
pm2 logs app_name --err

# 重启应用
pm2 restart app_name
```

**3. Nginx 503 错误**
```bash
# 检查后端服务
curl http://127.0.0.1:3000/health

# 检查 Nginx 配置
nginx -t

# 查看 Nginx 错误日志
tail -f /var/log/nginx/error.log
```

## 性能指标监控

使用 PM2 Plus 或 New Relic 监控关键指标:
- CPU 使用率
- 内存使用量
- 请求响应时间
- 错误率
- 数据库查询时间

## 安全加固

1. **防火墙配置**
   - 开放必要端口 (80, 443, 3306仅限内网)
   - 使用宝塔防火墙功能

2. **DDoS 防护**
   - 使用 Cloudflare 或阿里云 WAF
   - 配置速率限制

3. **定期安全审计**
   - 更新依赖包: `npm audit fix`
   - 定期渗透测试

---

需要帮助？查看 [开发指南](./DEVELOPMENT.md) 或提交 Issue。
