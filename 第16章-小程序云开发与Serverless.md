# 第16章 小程序云开发与Serverless

## 16.1 小程序云开发与Serverless的概念

### 什么是 Serverless

Serverless（无服务器架构）是一种云计算执行模型，开发者无需关心服务器的配置、运维和扩展，只需编写业务逻辑代码，由云平台自动管理计算资源的分配和调度。

Serverless 的核心思想是：**开发者只关注业务代码，基础设施交给云平台**。

```
传统开发：开发者需要管理 → 服务器、操作系统、运行环境、数据库、业务代码
Serverless：开发者只需管理 → 业务代码
```

### 小程序云开发的四大核心能力

小程序云开发是微信团队提供的 Serverless 服务，集成了以下四大核心能力：

**1. 云函数**

运行在云端的 Node.js 代码，无需搭建服务器。每个云函数是一个独立的执行单元，可以处理业务逻辑、调用第三方 API、操作数据库等。

```javascript
// 云函数：获取电影列表
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

exports.main = async (event, context) => {
  const db = cloud.database();
  const result = await db.collection('movies')
    .limit(event.size || 10)
    .get();
  return result.data;
};
```

**2. 云数据库**

基于文档的 NoSQL 数据库，类似 MongoDB。支持在客户端直接操作（受权限规则限制），也支持在云函数中操作（拥有管理员权限）。

```javascript
// 客户端直接操作数据库
const db = wx.cloud.database();
db.collection('movies').where({ score: db.command.gt(8) }).get();
```

**3. 云存储**

用于存储文件（图片、视频、文档等），支持在客户端直接上传/下载文件，也支持在云函数中管理文件。

```javascript
// 客户端上传文件到云存储
wx.cloud.uploadFile({
  cloudPath: 'avatar/user123.png',
  filePath: tempFilePath,
  success(res) {
    console.log('文件ID：', res.fileID);
  }
});
```

**4. 云调用**

云函数中调用微信开放接口的能力，如发送模板消息、生成小程序码、获取微信运动数据等，免去了 access_token 的管理。

```javascript
// 云函数中发送订阅消息
const result = await cloud.openapi.subscribeMessage.send({
  touser: 'OPENID',
  templateId: 'TEMPLATE_ID',
  data: { thing1: { value: '电影上映提醒' } }
});
```

## 16.2 小程序云开发模式

### 传统模式 vs 云开发模式对比

| 对比维度 | 传统模式 | 云开发模式 |
|---------|---------|-----------|
| 服务器 | 需要自行购买/租赁服务器 | 无需服务器，自动分配 |
| 后端语言 | 需要学习 Java/Node.js/Python 等 | 只需 JavaScript（Node.js） |
| 数据库 | MySQL/PostgreSQL 等关系型 | 内置 NoSQL 云数据库 |
| 文件存储 | 需要搭建文件服务器或使用 OSS | 内置云存储 |
| 域名配置 | 需要备案域名、配置 HTTPS | 免域名，直接调用 |
| 鉴权 | 需要自行实现登录鉴权 | 内置微信登录态 |
| 运维 | 需要监控服务器状态 | 免运维，自动弹性伸缩 |
| 部署 | 需要配置部署流程 | 一键部署云函数 |
| 学习成本 | 前端+后端+运维 | 前端 + 少量云函数 |
| 费用 | 固定服务器成本 | 按量付费，有免费额度 |

### 架构对比图

```
【传统模式架构】
小程序前端 ──HTTPS──→ 自建服务器（Node.js/Java）──→ MySQL数据库
                         │
                         └──→ 文件存储服务（OSS）

【云开发架构】
小程序前端 ──→ 云开发（云函数 + 云数据库 + 云存储 + 云调用）
              （全部在微信云内，无需自建服务器）
```

### 何时选择云开发

**适合使用云开发的场景：**
- 个人开发者或小团队，没有后端资源
- 中小型应用，请求量不大
- 快速原型开发，需要快速上线
- 不想处理域名备案、HTTPS 证书等繁琐配置

**不适合使用云开发的场景：**
- 高并发、大数据量的应用
- 需要复杂的事务处理和关联查询
- 有严格的数据合规要求，需自建数据库
- 已有成熟的后端服务团队和基础设施

## 16.3 项目改造云开发的预备知识

### 前置知识要求

在将项目改造为云开发模式之前，需要掌握以下知识：

**1. JavaScript / Node.js 基础**
云函数运行在 Node.js 环境中，需要了解 CommonJS 模块系统、async/await 异步编程等。

```javascript
// 云函数中的模块导入使用 require
const cloud = require('wx-server-sdk');
const axios = require('axios');

// 异步操作使用 async/await
exports.main = async (event, context) => {
  const res = await axios.get('https://api.example.com/data');
  return res.data;
};
```

**2. NoSQL 数据库概念**
云数据库是文档型 NoSQL 数据库，需要理解集合、文档、字段等概念，这与 MySQL 等关系型数据库有显著区别。

**3. 微信小程序基础**
熟悉小程序的页面生命周期、网络请求、本地存储等基础 API。

### 改造步骤概览

```
1. 开通云开发环境
2. 创建云函数目录
3. 配置 project.config.json 关联云环境
4. 将网络请求替换为云函数调用
5. 将数据迁移到云数据库
6. 将文件上传替换为云存储
7. 测试并部署
```

### project.config.json 配置

```json
{
  "miniprogramRoot": "miniprogram/",
  "cloudfunctionRoot": "cloudfunctions/",
  "setting": {
    "urlCheck": false
  },
  "appid": "your-appid"
}
```

- `cloudfunctionRoot`：指定云函数目录
- `miniprogramRoot`：指定小程序前端代码目录

### app.js 初始化

```javascript
// app.js
App({
  onLaunch() {
    if (!wx.cloud) {
      console.error('请使用 2.2.3 或以上的基础库以使用云能力');
    } else {
      wx.cloud.init({
        env: 'your-env-id',    // 云开发环境ID
        traceUser: true         // 记录用户访问
      });
    }
  }
});
```

> **注意事项**：`wx.cloud.init` 必须在 `app.js` 的 `onLaunch` 中调用，且在整个应用生命周期中只需调用一次。环境 ID 必须与你在云开发控制台中创建的环境一致。

## 16.4 初识云开发

### 开通云开发

1. 打开微信开发者工具，创建小程序项目时勾选「微信云开发」
2. 或在已有项目中，点击工具栏的「云开发」按钮
3. 点击「开通」，创建云开发环境

### 云开发控制台

云开发控制台提供了以下功能模块：

| 模块 | 功能 |
|------|------|
| 概览 | 查看资源使用情况、调用统计 |
| 数据库 | 管理集合、记录，导入导出数据 |
| 存储 | 管理云存储中的文件 |
| 云函数 | 管理云函数的部署、日志、监控 |
| 统计分析 | 查看 API 调用统计 |
| 设置 | 环境设置、权限管理 |

### 环境 ID

每个云开发环境都有唯一的**环境 ID**，在代码中初始化时需要用到：

```javascript
wx.cloud.init({
  env: 'movie-project-3g8axxxxx'  // 你的环境 ID
});
```

一个账号可以创建两个免费环境，通常一个用于开发，一个用于生产：

```javascript
// 根据不同环境动态选择 env
const envMap = {
  develop: 'movie-dev-xxxxx',
  release: 'movie-prod-xxxxx'
};

wx.cloud.init({
  env: envMap[wx.getAccountInfoSync().miniProgram.envVersion] || envMap.develop
});
```

> **常见坑**：环境 ID 不是环境名称！在控制台创建环境时会让你填名称，系统会自动生成一个带后缀的环境 ID。代码中必须使用环境 ID。

## 16.5 云数据库设计基础原则

### NoSQL 设计思维

云数据库是 NoSQL 文档型数据库，设计思维与传统关系型数据库有本质区别：

**关系型数据库设计思路（范式设计）：**
- 追求数据不冗余
- 通过外键关联多张表
- 查询时通过 JOIN 联表

**NoSQL 数据库设计思路（反范式设计）：**
- 允许数据冗余，优先考虑查询效率
- 数据嵌套存储，减少联表查询
- 根据业务查询场景来设计数据结构

### 反范式 vs 范式设计示例

以电影评论功能为例：

**范式设计（关系型思维）：**

```
movies 表:  { id, title, year, director_id }
directors 表: { id, name }
comments 表: { id, movie_id, user_id, content, created_at }
users 表: { id, nick_name, avatar }
```

查询一条评论需要关联 4 张表。

**反范式设计（NoSQL 思维）：**

```json
// comments 集合中的一条记录
{
  "_id": "comment001",
  "movie": {
    "_id": "movie123",
    "title": "流浪地球",
    "year": 2023
  },
  "user": {
    "_id": "user456",
    "nickName": "影迷小明",
    "avatar": "cloud://xxx"
  },
  "content": "非常震撼的科幻大片！",
  "rating": 9,
  "createdAt": "2024-01-15T10:30:00Z"
}
```

查询一条评论只需一次查询即可获取所有信息。

### 设计原则总结

| 原则 | 说明 |
|------|------|
| 根据查询设计结构 | 先确定需要哪些查询，再设计数据结构 |
| 允许适度冗余 | 将高频关联查询的数据嵌入到同一文档 |
| 避免无限增长的数组 | 文档大小有 16MB 限制，避免嵌套数组无限增长 |
| 考虑读写比例 | 读多写少 → 多冗余；写多读少 → 减少冗余 |
| 预留扩展字段 | 为未来需求预留可扩展字段 |

> **注意**：反范式设计的缺点是数据更新时需要同步更新多处。例如用户修改昵称后，所有包含该用户信息的评论记录都需要更新。实际项目中需要权衡。

## 16.6 数据库、集合、记录与字段

### 概念映射

云数据库的概念与关系型数据库的映射关系如下：

| 关系型数据库 | 云数据库（NoSQL） | 说明 |
|------------|------------------|------|
| Database（数据库） | Database | 数据库 |
| Table（表） | Collection（集合） | 数据集合 |
| Row（行） | Record / Document（记录/文档） | 一条数据 |
| Column（列） | Field（字段） | 数据字段 |
| 主键 | _id | 自动生成的唯一标识 |
| —— | _openid | 自动记录的创建者标识 |

### 记录结构示例

```json
{
  "_id": "movie_001",                    // 自动生成或自定义的唯一ID
  "_openid": "oXXXXXXXXXXXXXX",          // 创建者的openid
  "title": "流浪地球2",
  "year": 2023,
  "director": "郭帆",
  "casts": ["吴京", "刘德华", "李雪健"],
  "genres": ["科幻", "冒险", "灾难"],
  "rating": {
    "average": 8.3,
    "max": 10,
    "min": 0,
    "stars": "45"
  },
  "images": {
    "small": "cloud://xxx/small.jpg",
    "large": "cloud://xxx/large.jpg"
  },
  "summary": "太阳即将毁灭，人类开启流浪计划...",
  "tags": [
    { "name": "科幻", "count": 50000 },
    { "name": "中国电影", "count": 30000 }
  ],
  "createdAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-06-01T12:00:00Z"
}
```

### _id 和 _openid 的特点

**_id**：
- 每条记录的唯一标识
- 可自定义或由系统自动生成
- 自动生成时为随机字符串

**_openid**：
- 记录创建者的微信唯一标识
- 客户端添加记录时自动写入
- 云函数添加记录时需要手动写入

### 常用数据库操作

```javascript
const db = wx.cloud.database();
const _ = db.command;  // 获取查询指令

// 查询
db.collection('movies').where({ year: 2023 }).get();

// 添加
db.collection('movies').add({
  data: { title: '新电影', year: 2024 }
});

// 更新
db.collection('movies').doc('movie_001').update({
  data: { rating: { average: 9.0 } }
});

// 删除
db.collection('movies').doc('movie_001').remove();
```

> **注意**：客户端直接操作数据库受权限规则限制。例如"仅创建者可读写"意味着用户只能操作自己创建的记录。需要跨用户操作数据时，应通过云函数执行。

## 16.7 创建集合与导入数据

### 控制台创建集合

1. 打开云开发控制台 → 数据库
2. 点击「+」创建集合
3. 输入集合名称（如 `movies`）
4. 集合创建后可以添加记录

### 手动添加记录

在控制台中点击「添加记录」，输入 JSON 数据：

```json
{
  "title": "流浪地球2",
  "year": 2023,
  "director": "郭帆",
  "rating": {
    "average": 8.3
  },
  "genres": ["科幻", "冒险"]
}
```

### 批量导入数据

云开发支持 JSON 和 CSV 两种导入格式。

**JSON 格式导入**

准备 `movies.json` 文件，每行一个 JSON 对象：

```json
{"title":"流浪地球2","year":2023,"director":"郭帆","rating":{"average":8.3},"genres":["科幻","冒险"]}
{"title":"满江红","year":2023,"director":"张艺谋","rating":{"average":7.5},"genres":["喜剧","悬疑"]}
{"title":"消失的她","year":2023,"director":"崔睿","rating":{"average":7.4},"genres":["悬疑","犯罪"]}
{"title":"封神第一部","year":2023,"director":"乌尔善","rating":{"average":7.8},"genres":["奇幻","战争"]}
```

> **注意**：JSON 格式导入时，不是标准的 JSON 数组格式，而是**每行一个独立的 JSON 对象**（NDJSON 格式），行与行之间不能有逗号。

**CSV 格式导入**

准备 `movies.csv` 文件：

```csv
title,year,director,rating_average,genres
流浪地球2,2023,郭帆,8.3,"科幻,冒险"
满江红,2023,张艺谋,7.5,"喜剧,悬疑"
消失的她,2023,崔睿,7.4,"悬疑,犯罪"
```

### 导入操作步骤

1. 在云开发控制台 → 数据库 → 选择集合
2. 点击「导入」
3. 选择文件格式（JSON/CSV）
4. 设置冲突处理方式：
   - `insert`：直接插入，遇到 _id 冲突报错
   - `upsert`：存在则更新，不存在则插入
5. 点击「确定」开始导入

### 通过云函数批量导入

对于复杂的数据初始化，可以编写云函数：

```javascript
// 云函数 initMovies/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const db = cloud.database();

const moviesData = [
  { title: '流浪地球2', year: 2023, director: '郭帆', rating: { average: 8.3 } },
  { title: '满江红', year: 2023, director: '张艺谋', rating: { average: 7.5 } },
  { title: '消失的她', year: 2023, director: '崔睿', rating: { average: 7.4 } }
];

exports.main = async (event, context) => {
  const tasks = moviesData.map(movie => {
    return db.collection('movies').add({ data: movie });
  });

  const results = await Promise.all(tasks);
  return {
    message: '导入成功',
    count: results.length
  };
};
```

> **常见坑**：云函数批量插入时，如果数据量很大（上千条），需要分批处理，避免单次操作超时（云函数执行时间上限为 20 秒）。

## 16.8 更改数据库读写权限

### 四种权限模式

云数据库提供四种权限设置，决定了客户端能以什么规则操作数据：

**1. 仅创建者可读写**

只有记录的创建者（`_openid` 匹配当前用户）才能读写该记录。

```
用户A 创建了记录 → 用户A 可读可写，用户B 不可读不可写
```

适用场景：用户个人数据（个人设置、个人收藏、浏览历史）

**2. 所有用户可读，仅创建者可写**

所有人都可以读取数据，但只有创建者可以修改自己的数据。

```
用户A 创建了记录 → 所有用户可读，只有用户A 可写
```

适用场景：评论、动态、公开的内容

**3. 仅管理端可读写**

客户端无法直接操作数据，只能通过云函数（管理端权限）操作。

```
客户端 → 无法直接操作
云函数 → 拥有管理员权限，可以操作所有数据
```

适用场景：敏感数据、需要审核后发布的内容、全局配置数据

**4. 所有用户可读写（不推荐）**

所有用户都可以读写所有数据，没有任何限制。

适用场景：测试环境，不建议在生产环境使用

### 权限设置操作

1. 打开云开发控制台 → 数据库
2. 选择目标集合
3. 点击「权限设置」
4. 选择权限模式
5. 点击「保存」

### 权限与代码的关系

权限设置只影响**客户端直接操作数据库**的行为，不影响**云函数操作数据库**。

```javascript
// ===== 客户端代码（受权限规则限制）=====

// 假设权限为「仅创建者可读写」
// 用户只能查询到自己创建的评论
db.collection('comments').get();  // 只返回当前用户创建的评论

// 用户尝试修改别人的评论 → 权限错误
db.collection('comments').doc('others_comment_id').update({
  data: { content: '修改内容' }
});  // 报错：权限不足


// ===== 云函数代码（拥有管理员权限，不受限制）=====

// 云函数可以查询所有评论
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });
const db = cloud.database();

exports.main = async (event, context) => {
  // 获取所有用户的评论
  const result = await db.collection('comments')
    .limit(100)
    .get();

  return result.data;  // 返回所有评论，不受权限限制
};
```

### 客户端跨用户查询方案

当需要查询其他用户创建的数据时（如查看所有用户的评论），有两种方案：

**方案一：设置权限为「所有用户可读，仅创建者可写」**

```javascript
// 客户端可以直接查询所有评论
db.collection('comments').where({
  movieId: 'movie_001'
}).get();
```

**方案二：权限设为「仅管理端可读写」，通过云函数查询**

```javascript
// 云函数 getComments
exports.main = async (event, context) => {
  const db = cloud.database();
  const result = await db.collection('comments')
    .where({ movieId: event.movieId })
    .get();
  return result.data;
};
```

```javascript
// 客户端调用云函数
wx.cloud.callFunction({
  name: 'getComments',
  data: { movieId: 'movie_001' },
  success(res) {
    console.log('评论列表：', res.result);
  }
});
```

> **常见坑**：
> 1. 权限设置是集合级别的，不能针对单条记录设置不同权限。
> 2. `_openid` 字段在客户端添加记录时自动写入，但如果是通过云函数添加的，需要手动写入 `_openid`。
> 3. 客户端的 `db.collection().get()` 在"仅创建者可读写"权限下，只会返回当前用户创建的记录，而不是集合中的所有记录。

## 16.9 本章小结

### 核心知识点回顾

本章介绍了小程序云开发的基础概念和核心操作：

**1. Serverless 与云开发概念**
- Serverless = 开发者只关注业务代码，基础设施交给云平台
- 云开发四大能力：云函数、云数据库、云存储、云调用
- 核心优势：免服务器、免域名、免运维、内置微信登录态

**2. 传统模式 vs 云开发模式**
- 传统模式需要自建服务器、数据库、文件存储，需配置域名和 HTTPS
- 云开发模式集成全部后端能力，前端 JS 即可完成全栈开发
- 适合个人开发者、小团队、快速原型开发

**3. 项目改造预备知识**
- Node.js 基础（CommonJS、async/await）
- NoSQL 数据库概念
- `project.config.json` 配置云函数目录
- `app.js` 中 `wx.cloud.init` 初始化

**4. 云开发环境管理**
- 通过控制台开通云开发
- 环境管理：开发环境和生产环境分离
- 环境 ID 是代码初始化的关键参数

**5. 云数据库设计原则**
- NoSQL 采用反范式设计，允许数据冗余
- 根据查询场景设计数据结构
- 优先减少联表查询，提升读取效率
- 注意文档大小限制（16MB）

**6. 数据库核心概念**
- 集合（Collection）↔ 表（Table）
- 记录（Record）↔ 行（Row）
- 字段（Field）↔ 列（Column）
- `_id` 唯一标识，`_openid` 创建者标识

**7. 数据导入**
- 支持 JSON（NDJSON 格式，每行一个对象）和 CSV 导入
- 冲突处理：insert（直接插入）和 upsert（更新或插入）
- 大量数据可通过云函数批量导入

**8. 权限管理**
- 仅创建者可读写：个人数据
- 所有用户可读，仅创建者可写：公开内容（评论、动态）
- 仅管理端可读写：敏感数据、需审核内容
- 所有用户可读写：仅测试环境
- 权限只影响客户端，云函数拥有管理员权限

### 架构决策指南

```
是否需要跨用户查询数据？
├── 是 → 权限设为「所有用户可读」或「仅管理端」+ 云函数
└── 否 → 权限设为「仅创建者可读写」

数据是否敏感？
├── 是 → 权限设为「仅管理端可读写」+ 云函数操作
└── 否 → 可使用更宽松的权限

是否需要复杂查询？
├── 是 → 使用云函数（支持更复杂的聚合查询）
└── 否 → 客户端直接操作即可
```

### 实战要点

- `wx.cloud.init` 在 `app.js` 中只需调用一次
- JSON 导入是 NDJSON 格式，不是标准 JSON 数组
- 客户端操作数据库受权限限制，云函数操作不受限制
- `_openid` 在客户端添加记录时自动生成，云函数需手动写入
- 环境区分开发和生产，避免数据污染
- 云函数执行有时间限制（20秒），大数据操作需分批处理

本章为云开发的入门基础，后续章节将深入云函数开发、云数据库高级查询、云存储管理等核心实践内容。
