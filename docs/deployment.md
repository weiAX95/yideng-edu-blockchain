# 部署指南

## 环境要求

### 1. 开发环境

- Node.js >= 16.0.0
- pnpm >= 7.0.0
- Hardhat >= 2.0.0
- Solidity ^0.8.0

### 2. 网络环境

- 主网：Ethereum Mainnet
- 测试网：Sepolia Testnet
- RPC 节点提供商（如 Infura、Alchemy）

### 3. 密钥配置

```env
# .env 文件配置
PRIVATE_KEY=your_deployment_wallet_private_key
INFURA_API_KEY=your_infura_api_key
ETHERSCAN_API_KEY=your_etherscan_api_key
REPORT_GAS=true

# Chainlink 配置
CHAINLINK_TOKEN=your_chainlink_token_address
CHAINLINK_ORACLE=your_chainlink_oracle_address
CHAINLINK_JOB_ID=your_chainlink_job_id

# API 配置
API_URL=your_api_endpoint
WEB3_STORAGE_TOKEN=your_web3_storage_token
```

## 智能合约部署

### 1. 编译合约

```bash
# 编译所有合约
npx hardhat compile

# 导出 ABI
npx hardhat export-abi
```

### 2. 运行测试

```bash
# 运行所有测试
npx hardhat test

# 运行指定测试
npx hardhat test test/YiDengToken.js

# 生成覆盖率报告
npx hardhat coverage
```

### 3. 部署步骤

#### 3.1 部署主网

```bash
# 1. 部署 YiDengToken
npx hardhat run scripts/deploy.js --network mainnet

# 2. 部署 CourseCertificate
npx hardhat run scripts/deploy-certificate.js --network mainnet

# 3. 部署 CourseMarket
npx hardhat run scripts/deploy-market.js --network mainnet

# 4. 部署 TechArticleDAO
npx hardhat run scripts/deploy-dao.js --network mainnet

# 5. 部署 AutomatedCourseOracle
npx hardhat run scripts/deploy-oracle.js --network mainnet
```

#### 3.2 部署测试网

```bash
# 使用相同脚本，更换网络参数
npx hardhat run scripts/deploy.js --network sepolia
```

### 4. 合约验证

```bash
# 验证合约代码
npx hardhat verify --network mainnet DEPLOYED_CONTRACT_ADDRESS [构造函数参数]
```

## 后端服务部署

### 1. 数据库配置

#### 1.1 MongoDB 设置

```bash
# 安装 MongoDB
brew install mongodb-community@6.0

# 启动服务
brew services start mongodb-community@6.0

# 创建数据库和用户
mongosh
> use yideng
> db.createUser({
    user: "admin",
    pwd: "password",
    roles: ["readWrite", "dbAdmin"]
})
```

#### 1.2 Redis 设置

```bash
# 安装 Redis
brew install redis

# 启动服务
brew services start redis
```

### 2. 后端服务配置

#### 2.1 配置文件

```yaml
# config/production.yml
server:
  port: 3000
  host: '0.0.0.0'

database:
  mongodb:
    uri: 'mongodb://admin:password@localhost:27017/yideng'
  redis:
    host: 'localhost'
    port: 6379

blockchain:
  provider: 'https://mainnet.infura.io/v3/YOUR-PROJECT-ID'
  contracts:
    token: '0x...'
    market: '0x...'
    certificate: '0x...'
    dao: '0x...'
    oracle: '0x...'

jwt:
  secret: 'your-secret-key'
  expiresIn: '24h'
```

#### 2.2 PM2 部署配置

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'yideng-api',
      script: 'dist/main.js',
      instances: 'max',
      exec_mode: 'cluster',
      env_production: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
    },
  ],
};
```

### 3. 启动服务

```bash
# 构建项目
npm run build

# 使用 PM2 启动服务
pm2 start ecosystem.config.js --env production

# 查看日志
pm2 logs yideng-api

# 监控服务
pm2 monit
```

## 前端部署

### 1. 构建配置

```javascript
// vite.config.ts
export default defineConfig({
  build: {
    target: 'es2015',
    outDir: 'dist',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          web3: ['web3', '@ethersproject/providers'],
        },
      },
    },
  },
});
```

### 2. 环境变量配置

```env
# .env.production
VITE_API_URL=https://api.yideng.com
VITE_WEB3_PROVIDER=https://mainnet.infura.io/v3/YOUR-PROJECT-ID
VITE_CHAIN_ID=1
```

### 3. Nginx 配置

```nginx
# /etc/nginx/conf.d/yideng.conf
server {
    listen 80;
    server_name yideng.com;

    location / {
        root /var/www/yideng;
        try_files $uri $uri/ /index.html;
        expires 30d;
    }

    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /ws {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }
}
```

### 4. 部署步骤

```bash
# 构建项目
npm run build

# 复制文件到服务器
scp -r dist/* user@server:/var/www/yideng/

# 重启 Nginx
sudo systemctl restart nginx
```

## 监控和维护

### 1. 日志配置

#### 1.1 服务器日志

```bash
# 配置日志轮转
sudo vim /etc/logrotate.d/yideng
```

```
/var/log/yideng/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

#### 1.2 应用日志

```typescript
// src/utils/logger.ts
import winston from 'winston';

export const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(winston.format.timestamp(), winston.format.json()),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});
```

### 2. 监控配置

#### 2.1 Grafana 配置

```yaml
# docker-compose.yml
version: '3'
services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - '3000:3000'
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=your_password

  prometheus:
    image: prom/prometheus:latest
    ports:
      - '9090:9090'
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-storage:/prometheus
```

#### 2.2 合约事件监控

```typescript
// src/monitors/contract-events.ts
async function monitorContractEvents() {
  const provider = new ethers.providers.WebSocketProvider(WS_PROVIDER_URL);

  const market = new ethers.Contract(MARKET_ADDRESS, MARKET_ABI, provider);

  market.on('CoursePurchased', async (buyer, courseId, event) => {
    logger.info('Course purchased', {
      buyer,
      courseId: courseId.toString(),
      transactionHash: event.transactionHash,
    });
  });
}
```

### 3. 备份策略

#### 3.1 数据库备份

```bash
#!/bin/bash
# backup-db.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/var/backups/mongodb"

# MongoDB 备份
mongodump --uri="mongodb://admin:password@localhost:27017/yideng" --out="$BACKUP_DIR/$DATE"

# 压缩备份
tar -zcvf "$BACKUP_DIR/$DATE.tar.gz" "$BACKUP_DIR/$DATE"

# 删除 30 天前的备份
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete
```

#### 3.2 合约数据备份

```typescript
// src/scripts/backup-contract-data.ts
async function backupContractData() {
  const provider = new ethers.providers.JsonRpcProvider(RPC_URL);

  // 备份代币持有者数据
  const token = new ethers.Contract(TOKEN_ADDRESS, TOKEN_ABI, provider);
  const holders = await token.getHolders();

  // 保存到数据库
  await HoldersBackup.create({
    timestamp: new Date(),
    holders: holders,
  });
}
```

### 4. 更新流程

#### 4.1 智能合约更新

1. 开发新版本合约
2. 完整测试套件验证
3. 审计新合约代码
4. 部署新合约
5. 迁移数据
6. 更新前端 ABI

#### 4.2 后端服务更新

```bash
# 1. 拉取最新代码
git pull origin main

# 2. 安装依赖
npm install

# 3. 构建项目
npm run build

# 4. 重启服务
pm2 reload yideng-api

# 5. 验证服务状态
pm2 status
```

#### 4.3 前端更新

```bash
# 1. 构建新版本
npm run build

# 2. 备份当前版本
ssh user@server "cp -r /var/www/yideng /var/www/yideng_backup_$(date +%Y%m%d)"

# 3. 部署新版本
scp -r dist/* user@server:/var/www/yideng/

# 4. 清理缓存
ssh user@server "rm -rf /var/www/yideng/cache/*"
```

### 5. 应急预案

#### 5.1 智能合约应急

1. 检测异常交易或事件
2. 触发紧急暂停
3. 分析问题原因
4. 准备修复方案
5. 执行合约升级
6. 恢复正常运行

#### 5.2 服务器应急

1. 配置自动报警
2. 准备快速回滚脚本
3. 维护备用节点
4. 定期演练恢复流程

#### 5.3 数据恢复

1. 保持多份备份
2. 验证备份完整性
3. 准备恢复脚本
4. 测试恢复流程
