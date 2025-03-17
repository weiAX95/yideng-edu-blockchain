# API 接口文档

## 接口规范

### 基础 URL

```
https://api.yideng.com/v1
```

### 请求格式

- Content-Type: application/json
- 字符编码：UTF-8

### 认证方式

```http
Authorization: Bearer <access_token>
```

### 响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    // 响应数据
  }
}
```

### 错误码

| 错误码 | 描述       |
| ------ | ---------- |
| 0      | 成功       |
| 40001  | 参数错误   |
| 40100  | 未授权     |
| 40300  | 权限不足   |
| 40400  | 资源不存在 |
| 50000  | 服务器错误 |

## 课程管理接口

### 1. 创建课程

```http
POST /courses
```

#### 请求参数

```json
{
  "title": "课程标题",
  "description": "课程描述",
  "price": 100,
  "courseType": "video",
  "coverImage": "https://...",
  "sections": [
    {
      "title": "章节1",
      "content": "章节内容",
      "duration": 3600
    }
  ]
}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "courseId": "c123456",
    "web2CourseId": "w123456",
    "title": "课程标题",
    "status": "pending"
  }
}
```

### 2. 获取课程列表

```http
GET /courses
```

#### 查询参数

- page: 页码(默认 1)
- size: 每页数量(默认 10)
- status: 课程状态(可选)
- creator: 创建者地址(可选)

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 100,
    "items": [
      {
        "courseId": "c123456",
        "title": "课程标题",
        "price": 100,
        "status": "active",
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

## 学习进度接口

### 1. 更新学习进度

```http
POST /progress/{courseId}
```

#### 请求参数

```json
{
  "sectionId": "s123456",
  "watchTime": 1800,
  "completed": true
}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "progress": 75,
    "lastUpdate": "2024-01-01T00:00:00Z"
  }
}
```

### 2. 获取学习进度

```http
GET /progress/{courseId}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "courseId": "c123456",
    "progress": 75,
    "sections": [
      {
        "sectionId": "s123456",
        "completed": true,
        "watchTime": 1800
      }
    ]
  }
}
```

## 证书接口

### 1. 获取证书列表

```http
GET /certificates
```

#### 查询参数

- student: 学生地址
- courseId: 课程 ID(可选)
- page: 页码
- size: 每页数量

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 10,
    "items": [
      {
        "tokenId": "1",
        "courseId": "c123456",
        "title": "课程名称",
        "issuedAt": "2024-01-01T00:00:00Z",
        "imageUrl": "https://..."
      }
    ]
  }
}
```

### 2. 获取证书详情

```http
GET /certificates/{tokenId}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "tokenId": "1",
    "courseId": "c123456",
    "student": "0x...",
    "title": "课程名称",
    "issuedAt": "2024-01-01T00:00:00Z",
    "imageUrl": "https://...",
    "metadata": {
      "name": "课程证书",
      "description": "...",
      "image": "https://...",
      "attributes": []
    }
  }
}
```

## 文章管理接口

### 1. 提交文章

```http
POST /articles
```

#### 请求参数

```json
{
  "title": "文章标题",
  "content": "文章内容",
  "courseId": "c123456",
  "tags": ["blockchain", "web3"]
}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "articleId": "a123456",
    "status": "pending",
    "submittedAt": "2024-01-01T00:00:00Z"
  }
}
```

### 2. 评审文章

```http
POST /articles/{articleId}/review
```

#### 请求参数

```json
{
  "support": true,
  "score": 85,
  "comment": "评审意见"
}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "articleId": "a123456",
    "reviewId": "r123456",
    "status": "reviewed"
  }
}
```

## 用户钱包接口

### 1. 获取代币余额

```http
GET /wallet/balance/{address}
```

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "address": "0x...",
    "balance": "1000",
    "symbol": "YD"
  }
}
```

### 2. 获取交易历史

```http
GET /wallet/transactions
```

#### 查询参数

- address: 钱包地址
- type: 交易类型(all/in/out)
- page: 页码
- size: 每页数量

#### 响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 100,
    "items": [
      {
        "hash": "0x...",
        "type": "purchase",
        "amount": "100",
        "status": "confirmed",
        "timestamp": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

## WebSocket 接口

### 1. 实时进度更新

```
ws://api.yideng.com/ws/progress/{courseId}
```

#### 消息格式

```json
{
  "type": "progress_update",
  "data": {
    "progress": 75,
    "section": "s123456",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### 2. 交易状态通知

```
ws://api.yideng.com/ws/transactions
```

#### 消息格式

```json
{
  "type": "transaction_update",
  "data": {
    "hash": "0x...",
    "status": "confirmed",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

## 数据模型

### 课程模型

```typescript
interface Course {
  courseId: string;
  web2CourseId: string;
  title: string;
  description: string;
  price: number;
  creator: string;
  status: 'pending' | 'active' | 'inactive';
  createdAt: string;
  updatedAt: string;
  sections: Section[];
}

interface Section {
  sectionId: string;
  title: string;
  content: string;
  duration: number;
  order: number;
}
```

### 进度模型

```typescript
interface Progress {
  courseId: string;
  student: string;
  progress: number;
  sections: SectionProgress[];
  lastUpdate: string;
}

interface SectionProgress {
  sectionId: string;
  completed: boolean;
  watchTime: number;
  lastAccess: string;
}
```

### 文章模型

```typescript
interface Article {
  articleId: string;
  author: string;
  title: string;
  content: string;
  courseId: string;
  status: 'pending' | 'approved' | 'rejected';
  reviews: Review[];
  submittedAt: string;
  updatedAt: string;
}

interface Review {
  reviewId: string;
  reviewer: string;
  support: boolean;
  score: number;
  comment: string;
  createdAt: string;
}
```

## 安全说明

### 1. 认证要求

- 所有写操作需要认证
- 部分读操作需要认证
- 使用 JWT 进行身份验证

### 2. 访问控制

- 基于角色的权限控制
- 资源所有者验证
- 操作频率限制

### 3. 数据验证

- 输入参数验证
- 业务规则验证
- 数据完整性检查

### 4. 错误处理

- 统一错误响应格式
- 详细错误信息
- 错误日志记录

## 开发指南

### 1. 环境配置

```bash
# 开发环境
BASE_URL=https://dev-api.yideng.com/v1

# 测试环境
BASE_URL=https://test-api.yideng.com/v1

# 生产环境
BASE_URL=https://api.yideng.com/v1
```

### 2. 示例代码

#### TypeScript/JavaScript

```typescript
const YiDengAPI = {
  async getCourses(params) {
    const response = await fetch(`${BASE_URL}/courses`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    return response.json();
  },

  async submitArticle(data) {
    const response = await fetch(`${BASE_URL}/articles`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify(data),
    });
    return response.json();
  },
};
```

### 3. 调用建议

- 实现请求重试机制
- 添加请求超时处理
- 实现错误响应处理
- 添加响应缓存策略
- 实现并发请求控制
