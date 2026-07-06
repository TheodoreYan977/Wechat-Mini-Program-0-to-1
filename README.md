# 循序渐进微信小程序全栈项目实践 - 知识总览

> 本知识库基于《循序渐进微信小程序全栈项目实践》图书目录编写，涵盖从环境搭建到云开发、多端开发的完整知识体系。
> 同时补充了**阿里云服务器后端搭建**与**前后端联调**的实战知识，帮助你实现一个完整的前后端分离微信小程序。

---

## 目录索引

| 章节 | 文件名 | 核心知识点 |
|------|--------|-----------|
| 第1章 | [小程序环境搭建与开发工具介绍](第01章-小程序环境搭建与开发工具介绍.md) | 小程序概念、账号注册、开发者工具安装与使用 |
| 第2章 | [从一个简单的页面开始小程序之旅](第02章-从一个简单的页面开始小程序之旅.md) | 文件结构、WXML/WXSS、Flex布局、rpx单位、页面配置 |
| 第3章 | [数据绑定与文章列表](第03章-数据绑定与文章列表.md) | swiper组件、image组件、数据绑定、setData、wx:for、事件系统、路由跳转 |
| 第4章 | [模块模板与缓存](第04章-模块模板与缓存.md) | 模块导入导出、template模板、Storage缓存、ES6 Class |
| 第5章 | [文章详情页面](第05章-文章详情页面.md) | 页面跳转、参数传递、data-*自定义属性、编译模式、动态导航栏标题 |
| 第6章 | [评论与收藏](第06章-评论与收藏.md) | 条件渲染、wx:if与hidden、图片预览、input组件、评论系统实现 |
| 第7章 | [使用组件库](第07章-使用组件库.md) | Vant-Weapp导入、录音、语音播放、隐私接口、授权拉起 |
| 第8章 | [完善文章页面](第08章-完善文章页面.md) | 分享功能、onShareAppMessage、分享朋友圈、微信开放能力 |
| 第9章 | [Component组件化编程](第09章-Component组件化编程.md) | 自定义组件、properties、生命周期、Component与Template对比 |
| 第10章 | [电影与自定义组件实战](第10章-电影与自定义组件实战.md) | stars组件、movie组件、movie-list组件、组件嵌套 |
| 第11章 | [从服务器获取数据](第11章-从服务器获取数据.md) | globalData、wx.request、Promise/async-await、HTTPS可信域名 |
| 第12章 | [组件事件与电影搜索](第12章-组件事件与电影搜索.md) | triggerEvent、搜索防抖、下拉刷新、上滑加载更多 |
| 第13章 | [组件化思维构建电影详情页面](第13章-组件化思维构建电影详情页面.md) | 详情页开发、海报预览、externalClasses样式隔离 |
| 第14章 | [深入使用Vant组件库](第14章-深入使用Vant组件库.md) | Vant主题定制、外部样式类、插槽slot |
| 第15章 | [新版小程序重要API精讲](第15章-新版小程序重要API精讲.md) | 新版用户信息获取、系统信息、网络、定位、扫码、授权流程 |
| 第16章 | [小程序云开发与Serverless](第16章-小程序云开发与Serverless.md) | 云开发概念、NoSQL设计、集合/记录/字段、权限管理 |
| 第17章 | [小程序云开发实战](第17章-小程序云开发实战.md) | 云数据库CRUD、条件查询、云存储上传、云函数调用 |
| 第18章 | [云开发高级技巧](第18章-云开发高级技巧.md) | 联表查询、云函数编写与调试、原子计数、聚合统计 |
| 第19章 | [媲美原生App的新机制Skyline](第19章-媲美原生App的新机制Skyline.md) | Skyline渲染引擎、worklet动画、CSS差异、迁移指南 |
| 第20章 | [多端开发](第20章-多端开发.md) | 多端框架、编译iOS/Android、微信登录、SDK选型 |

---

## 学习路线建议

### 阶段一：入门基础（第1-4章）
- **目标**：掌握小程序开发环境、文件结构、页面编写、数据绑定基础
- **关键产出**：完成一个带欢迎页和文章列表的静态小程序
- **重点**：rpx自适应单位、Flex布局、数据绑定原理、setData机制

### 阶段二：页面与交互（第5-8章）
- **目标**：掌握页面跳转、参数传递、评论收藏、分享功能
- **关键产出**：完成文章详情页、评论系统、收藏功能、分享功能
- **重点**：事件系统、Storage缓存、wx:if与hidden的区别、Vant组件库

### 阶段三：组件化进阶（第9-14章）
- **目标**：掌握自定义组件开发、组件通信、网络请求、搜索与分页
- **关键产出**：完成电影模块（组件化架构、服务端数据、搜索功能）
- **重点**：Component生命周期、triggerEvent、wx.request封装、下拉刷新与上滑加载

### 阶段四：API与云开发（第15-18章）
- **目标**：掌握新版API、云开发全栈能力
- **关键产出**：将项目从本地缓存迁移到云开发后端
- **重点**：新版授权流程、云数据库操作、云函数编写与调试

### 阶段五：高级特性（第19-20章）
- **目标**：了解Skyline渲染引擎和多端开发
- **关键产出**：尝试Skyline迁移、将小程序编译为App
- **重点**：Skyline与WebView差异、worklet动画、多端SDK选型

---

## 阿里云服务器后端搭建补充知识

> 以下内容补充了如何使用阿里云服务器搭建后端API，并与小程序前端进行联调。
> 这是实现"前后端连接的微信小程序"的关键知识。

### 一、阿里云服务器选购与初始化

#### 1.1 服务器选购
- **推荐配置**：ECS突发性能型 t6 或 t5（入门级，2核2G/1核2G）
- **操作系统**：Ubuntu 22.04 LTS 或 CentOS 7.9
- **带宽**：1-5Mbps（按需选择，支持按使用流量计费）
- **地域选择**：选择离目标用户最近的地域

#### 1.2 安全组配置
```
开放端口：
- 22（SSH远程连接）
- 80（HTTP）
- 443（HTTPS）
- 3000/8080（Node.js开发端口，按需开放）
- 3306（MySQL，建议仅内网访问）
```

#### 1.3 服务器初始化
```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装 Node.js 18 LTS
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# 安装 PM2 进程管理器
sudo npm install -g pm2

# 安装 Nginx
sudo apt install -y nginx

# 安装 MySQL 8.0
sudo apt install -y mysql-server
sudo mysql_secure_installation
```

### 二、后端API服务搭建（Node.js + Express）

#### 2.1 项目初始化
```bash
mkdir miniapp-server && cd miniapp-server
npm init -y
npm install express mysql2 cors body-parser helmet morgan jsonwebtoken bcryptjs
npm install -D nodemon
```

#### 2.2 项目目录结构
```
miniapp-server/
├── src/
│   ├── config/          # 配置文件
│   │   └── db.js        # 数据库配置
│   ├── routes/          # 路由
│   │   ├── articles.js  # 文章路由
│   │   ├── movies.js    # 电影路由
│   │   └── user.js      # 用户路由
│   ├── controllers/     # 控制器
│   ├── middleware/      # 中间件
│   │   └── auth.js      # JWT鉴权中间件
│   ├── utils/           # 工具函数
│   └── app.js           # 入口文件
├── .env                 # 环境变量
├── package.json
└── ecosystem.config.js  # PM2配置
```

#### 2.3 数据库配置（src/config/db.js）
```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST || 'localhost',
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASSWORD || 'your_password',
  database: process.env.DB_NAME || 'miniapp_db',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

module.exports = pool;
```

#### 2.4 Express入口文件（src/app.js）
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
require('dotenv').config();

const articlesRouter = require('./routes/articles');
const moviesRouter = require('./routes/movies');
const userRouter = require('./routes/user');

const app = express();

// 中间件
app.use(helmet());
app.use(cors({
  origin: ['https://your-miniapp-domain.com'], // 小程序请求无CORS限制，但建议配置
}));
app.use(express.json());
app.use(morgan('combined'));

// 路由
app.use('/api/articles', articlesRouter);
app.use('/api/movies', moviesRouter);
app.use('/api/user', userRouter);

// 健康检查
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', timestamp: Date.now() });
});

// 错误处理
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ code: 500, message: '服务器内部错误' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### 2.5 文章路由示例（src/routes/articles.js）
```javascript
const express = require('express');
const router = express.Router();
const pool = require('../config/db');

// 获取文章列表
router.get('/', async (req, res) => {
  try {
    const { page = 1, pageSize = 10 } = req.query;
    const offset = (page - 1) * pageSize;

    const [rows] = await pool.execute(
      'SELECT id, title, cover_image, summary, view_count, created_at FROM articles ORDER BY created_at DESC LIMIT ? OFFSET ?',
      [parseInt(pageSize), parseInt(offset)]
    );

    const [countResult] = await pool.execute('SELECT COUNT(*) as total FROM articles');
    const total = countResult[0].total;

    res.json({
      code: 0,
      data: {
        list: rows,
        total,
        page: parseInt(page),
        pageSize: parseInt(pageSize),
        hasMore: offset + rows.length < total
      }
    });
  } catch (error) {
    console.error('获取文章列表失败:', error);
    res.status(500).json({ code: 500, message: '获取文章列表失败' });
  }
});

// 获取文章详情
router.get('/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const [rows] = await pool.execute(
      'SELECT * FROM articles WHERE id = ?',
      [id]
    );

    if (rows.length === 0) {
      return res.status(404).json({ code: 404, message: '文章不存在' });
    }

    // 浏览量+1
    await pool.execute('UPDATE articles SET view_count = view_count + 1 WHERE id = ?', [id]);

    res.json({ code: 0, data: rows[0] });
  } catch (error) {
    console.error('获取文章详情失败:', error);
    res.status(500).json({ code: 500, message: '获取文章详情失败' });
  }
});

module.exports = router;
```

#### 2.6 JWT鉴权中间件（src/middleware/auth.js）
```javascript
const jwt = require('jsonwebtoken');

function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ code: 401, message: '未登录' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET || 'your_secret_key');
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ code: 401, message: '登录已过期' });
  }
}

module.exports = authMiddleware;
```

### 三、数据库设计

#### 3.1 创建数据库和表
```sql
CREATE DATABASE miniapp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE miniapp_db;

-- 文章表
CREATE TABLE articles (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  cover_image VARCHAR(500),
  summary TEXT,
  content LONGTEXT,
  view_count INT DEFAULT 0,
  like_count INT DEFAULT 0,
  collect_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_created_at (created_at)
);

-- 用户表
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  openid VARCHAR(64) UNIQUE NOT NULL,
  nickname VARCHAR(100),
  avatar_url VARCHAR(500),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 评论表
CREATE TABLE comments (
  id INT AUTO_INCREMENT PRIMARY KEY,
  article_id INT NOT NULL,
  user_id INT NOT NULL,
  content TEXT,
  image_urls JSON,
  voice_url VARCHAR(500),
  voice_duration INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  INDEX idx_article_id (article_id),
  INDEX idx_created_at (created_at)
);

-- 收藏表
CREATE TABLE collections (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  article_id INT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uk_user_article (user_id, article_id),
  FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 四、Nginx反向代理与HTTPS配置

#### 4.1 Nginx配置
```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    # HTTP跳转HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    # SSL证书路径（阿里云免费SSL证书）
    ssl_certificate /etc/nginx/ssl/your_domain.pem;
    ssl_certificate_key /etc/nginx/ssl/your_domain.key;

    # 反向代理到Node.js
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
    }
}
```

#### 4.2 申请阿里云免费SSL证书
1. 进入阿里云控制台 → 数字证书管理服务
2. 申请免费证书（DV SSL），填写域名 `api.yourdomain.com`
3. 通过DNS验证后下载Nginx格式证书
4. 上传 `.pem` 和 `.key` 文件到服务器 `/etc/nginx/ssl/` 目录

### 五、PM2进程管理与部署

#### 5.1 PM2配置文件（ecosystem.config.js）
```javascript
module.exports = {
  apps: [{
    name: 'miniapp-server',
    script: './src/app.js',
    instances: 2,           // 集群模式，2个进程
    exec_mode: 'cluster',
    max_memory_restart: '500M',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
      DB_HOST: 'localhost',
      DB_USER: 'root',
      DB_PASSWORD: 'your_db_password',
      DB_NAME: 'miniapp_db',
      JWT_SECRET: 'your_jwt_secret_key'
    },
    error_file: './logs/error.log',
    out_file: './logs/output.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }]
};
```

#### 5.2 部署命令
```bash
# 启动服务
pm2 start ecosystem.config.js

# 查看状态
pm2 status

# 查看日志
pm2 logs miniapp-server

# 重启服务
pm2 restart miniapp-server

# 设置开机自启
pm2 startup
pm2 save
```

### 六、小程序前端对接

#### 6.1 封装HTTP请求工具（utils/http.js）
```javascript
const app = getApp();

const BASE_URL = 'https://api.yourdomain.com/api';

function request(options) {
  return new Promise((resolve, reject) => {
    const token = wx.getStorageSync('token') || '';

    wx.request({
      url: BASE_URL + options.url,
      method: options.method || 'GET',
      data: options.data || {},
      header: {
        'Content-Type': 'application/json',
        'Authorization': token ? `Bearer ${token}` : ''
      },
      success(res) {
        if (res.statusCode === 200) {
          if (res.data.code === 0) {
            resolve(res.data.data);
          } else if (res.data.code === 401) {
            // token过期，跳转登录
            wx.removeStorageSync('token');
            wx.showToast({ title: '请重新登录', icon: 'none' });
            reject(res.data);
          } else {
            wx.showToast({ title: res.data.message || '请求失败', icon: 'none' });
            reject(res.data);
          }
        } else {
          reject(res);
        }
      },
      fail(err) {
        wx.showToast({ title: '网络异常', icon: 'none' });
        reject(err);
      }
    });
  });
}

module.exports = {
  get: (url, data) => request({ url, method: 'GET', data }),
  post: (url, data) => request({ url, method: 'POST', data }),
  put: (url, data) => request({ url, method: 'PUT', data }),
  delete: (url, data) => request({ url, method: 'DELETE', data })
};
```

#### 6.2 微信登录流程（获取openid）
```javascript
// pages/login/login.js
const http = require('../../utils/http');

Page({
  data: {},

  async handleLogin() {
    try {
      // 1. 调用wx.login获取code
      const { code } = await new Promise((resolve, reject) => {
        wx.login({ success: resolve, fail: reject });
      });

      // 2. 发送code到后端，后端调用微信API获取openid
      const result = await http.post('/user/login', { code });

      // 3. 保存token
      wx.setStorageSync('token', result.token);
      wx.setStorageSync('userInfo', result.userInfo);

      // 4. 返回上一页
      wx.showToast({ title: '登录成功', icon: 'success' });
      setTimeout(() => wx.navigateBack(), 1500);
    } catch (error) {
      console.error('登录失败:', error);
    }
  }
});
```

#### 6.3 后端登录接口（routes/user.js）
```javascript
const express = require('express');
const router = express.Router();
const jwt = require('jsonwebtoken');
const axios = require('axios');
const pool = require('../config/db');

// 微信登录
router.post('/login', async (req, res) => {
  try {
    const { code } = req.body;

    // 调用微信API获取openid和session_key
    const wxResponse = await axios.get('https://api.weixin.qq.com/sns/jscode2session', {
      params: {
        appid: process.env.WX_APPID,
        secret: process.env.WX_SECRET,
        js_code: code,
        grant_type: 'authorization_code'
      }
    });

    const { openid, session_key } = wxResponse.data;

    if (!openid) {
      return res.status(400).json({ code: 400, message: '微信登录失败' });
    }

    // 查询或创建用户
    let [users] = await pool.execute('SELECT * FROM users WHERE openid = ?', [openid]);

    if (users.length === 0) {
      // 新用户，创建记录
      await pool.execute('INSERT INTO users (openid) VALUES (?)', [openid]);
      [users] = await pool.execute('SELECT * FROM users WHERE openid = ?', [openid]);
    }

    const user = users[0];

    // 生成JWT token
    const token = jwt.sign(
      { userId: user.id, openid: user.openid },
      process.env.JWT_SECRET || 'your_secret_key',
      { expiresIn: '7d' }
    );

    res.json({
      code: 0,
      data: {
        token,
        userInfo: {
          id: user.id,
          nickname: user.nickname,
          avatarUrl: user.avatar_url
        }
      }
    });
  } catch (error) {
    console.error('登录失败:', error);
    res.status(500).json({ code: 500, message: '登录失败' });
  }
});

module.exports = router;
```

### 七、小程序后台域名配置

> **关键步骤**：小程序要求所有网络请求必须走HTTPS，且域名需在后台配置白名单。

1. 登录 [微信公众平台](https://mp.weixin.qq.com)
2. 进入「开发」→「开发管理」→「开发设置」→「服务器域名」
3. 在 `request合法域名` 中添加：`https://api.yourdomain.com`
4. 在 `uploadFile合法域名` 中添加文件上传域名（如使用）
5. 在 `downloadFile合法域名` 中添加文件下载域名（如使用）

> 开发阶段可在微信开发者工具中勾选「不校验合法域名」临时跳过此限制。

### 八、图片上传到阿里云OSS

#### 8.1 安装阿里云SDK
```bash
npm install ali-oss
```

#### 8.2 OSS上传接口
```javascript
// routes/upload.js
const express = require('express');
const router = express.Router();
const OSS = require('ali-oss');
const authMiddleware = require('../middleware/auth');

const client = new OSS({
  region: 'oss-cn-hangzhou',
  accessKeyId: process.env.OSS_ACCESS_KEY_ID,
  accessKeySecret: process.env.OSS_ACCESS_KEY_SECRET,
  bucket: 'your-bucket-name'
});

router.post('/image', authMiddleware, async (req, res) => {
  try {
    // 小程序上传的文件在req.body中（需配合multer中间件）
    // 或使用前端直传OSS方案（推荐）
    res.json({ code: 0, data: { url: '...' } });
  } catch (error) {
    res.status(500).json({ code: 500, message: '上传失败' });
  }
});

module.exports = router;
```

### 九、完整联调流程

```
小程序前端                    阿里云服务器
    |                              |
    |  wx.login() → code           |
    |  ─────────────────────────→  |
    |                    后端调微信API获取openid
    |                    查询/创建用户记录
    |                    生成JWT token
    |  ←─────────────────────────  |
    |  { token, userInfo }         |
    |                              |
    |  wx.request(带token)         |
    |  GET /api/articles           |
    |  ─────────────────────────→  |
    |                    查询MySQL数据库
    |  ←─────────────────────────  |
    |  { code:0, data:{list:[]} }  |
    |                              |
    |  wx.uploadFile(图片)          |
    |  POST /api/upload/image      |
    |  ─────────────────────────→  |
    |                    上传到阿里云OSS
    |  ←─────────────────────────  |
    |  { code:0, data:{url:''} }   |
```

---

## 如何上传到 IMA 知识库

由于 IMA 知识库的 MCP 连接器目前只支持读取操作（搜索/列表/获取），不支持通过API写入，请按以下步骤手动上传：

1. 打开 [IMA 知识库](https://ima.qq.com) 或 IMA 客户端
2. 进入「颜雪涛 Theodore.yan的知识库」
3. 点击「添加知识」→「上传文件」
4. 将 `微信小程序全栈实践/` 目录下的所有 `.md` 文件逐个上传
5. IMA 会自动解析 Markdown 内容，支持后续搜索和问答

> **提示**：上传后可在 IMA 中直接提问，如"小程序的数据绑定原理是什么"、"如何在阿里云部署小程序后端"等，IMA 会基于上传的知识内容给出回答。

---

## 技术栈总览

| 层级 | 技术选型 | 说明 |
|------|---------|------|
| 前端 | 微信小程序原生开发 | WXML + WXSS + JavaScript |
| UI组件库 | Vant-Weapp | 轻量级微信小程序组件库 |
| 后端 | Node.js + Express | 轻量级Web框架 |
| 数据库 | MySQL 8.0 | 关系型数据库 |
| 文件存储 | 阿里云OSS | 图片/音频等静态资源 |
| 进程管理 | PM2 | Node.js进程守护 |
| Web服务器 | Nginx | 反向代理 + HTTPS |
| 云服务器 | 阿里云ECS | 应用部署环境 |
| SSL证书 | 阿里云免费DV证书 | HTTPS加密 |
| 鉴权方案 | JWT + 微信openid | 无状态认证 |
