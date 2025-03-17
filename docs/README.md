# 教育区块链平台

## 项目概述

教育区块链平台是一个创新的在线教育系统，将区块链技术与教育服务相结合。该平台提供课程购买、学习认证、技术社区等功能，通过区块链技术确保教育证书的真实性和不可篡改性。

## 核心功能

- **代币系统 (YiDengToken)**

  - 发行 YD 代币作为平台通证
  - 支持 ETH 购买和出售 YD 代币
  - 总供应量 125 万个 YD，包括团队(20%)、营销(10%)、社区(10%)和公开销售(60%)份额

- **课程市场 (CourseMarket)**

  - 课程发布与购买
  - 使用 YD 代币支付课程费用
  - 课程完成验证和证书发放

- **证书 NFT 系统 (CourseCertificate)**

  - 基于 ERC721 标准的课程证书
  - 链上验证学习成果
  - 永久保存且不可篡改的学习证明

- **技术社区 (TechArticleDAO)**

  - 技术文章投稿和评审
  - 基于 DAO 的社区治理
  - 文章质量评估和奖励机制

- **自动化课程追踪 (AutomatedCourseOracle)**
  - 基于 Chainlink 的自动化进度验证
  - 定时检查学习进度
  - 自动触发证书发放

## 技术栈

- Solidity 智能合约
- OpenZeppelin 合约标准库
- Hardhat 开发框架
- Chainlink 预言机
- TypeScript/JavaScript

## 项目结构

```
.
├── contracts/               # 智能合约目录
│   ├── YiDengToken.sol     # 平台代币合约
│   ├── CourseMarket.sol    # 课程市场合约
│   ├── CourseCertificate.sol # 证书NFT合约
│   ├── TechArticleDAO.sol  # 技术社区DAO合约
│   └── AutomatedCourseOracle.sol # 自动化预言机合约
├── test/                   # 测试文件目录
├── scripts/               # 部署脚本目录
└── frontend/             # 前端应用目录
```

## 详细文档

- [系统架构设计](./architecture.md)
- [智能合约详细说明](./contracts.md)
- [业务流程说明](./workflow.md)
- [API 接口文档](./api.md)
- [部署指南](./deployment.md)

## 开发环境设置

1. 安装依赖：

```bash
npm install
```

2. 编译合约：

```bash
npx hardhat compile
```

3. 运行测试：

```bash
npx hardhat test
```

4. 部署合约：

```bash
npx hardhat run scripts/deploy.js
```

## 安全考虑

- 使用 OpenZeppelin 标准库确保合约安全性
- 实现角色访问控制
- 设置交易限制和安全检查
- 使用预言机进行可信数据获取

## 许可证

MIT License
