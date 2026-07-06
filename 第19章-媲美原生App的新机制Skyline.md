# 第19章 媲美原生 App 的新机制 Skyline

## 19.1 Skyline 能解决什么问题（WebView 渲染性能瓶颈）

### 传统 WebView 渲染的痛点

微信小程序自诞生以来，一直采用基于 WebView 的渲染架构。虽然这种架构兼容性好、开发门槛低，但随着小程序功能日益复杂，WebView 渲染的瓶颈越来越明显：

**1. 渲染性能瓶颈**

- WebView 本质上是一个浏览器内核，UI 渲染需要经过 HTML 解析 → CSS 计算 → 布局 → 绘制的完整流程
- 长列表滚动时容易出现卡顿、白屏
- 复杂动画的帧率不稳定，难以达到 60fps 的流畅度

**2. 通信开销大**

- 小程序采用双线程模型（逻辑层 + 渲染层），逻辑层和渲染层之间通过 JS Bridge 通信
- 每次数据更新需要：逻辑层 → JS Bridge → 渲染层 → WebView 重新渲染
- 高频交互（如拖拽、滚动）时通信延迟明显

**3. 动画能力受限**

- CSS 动画在 WebView 中性能不稳定
- JS 动画受限于双线程通信延迟，容易出现掉帧
- 无法实现复杂的手势交互和物理动画

**4. 内存占用高**

- 每个 WebView 实例都是独立的浏览器内核，内存开销大
- 页面栈中的 WebView 不会被立即回收，导致内存累积

### Skyline 的解决思路

Skyline 渲染引擎直接使用微信客户端的原生渲染能力，绕过 WebView：

```
传统架构:  逻辑层 → JS Bridge → WebView(HTML/CSS渲染) → 屏幕
Skyline:   逻辑层 → 原生渲染层(直接调用系统UI) → 屏幕
```

| 对比项 | WebView 渲染 | Skyline 渲染 |
|--------|-------------|-------------|
| 渲染方式 | WebView 内核 | 原生渲染引擎 |
| 通信开销 | 大（跨线程 JS Bridge） | 小（直接渲染） |
| 动画帧率 | 不稳定（30-50fps） | 稳定 60fps |
| 长列表性能 | 容易卡顿 | 流畅滚动 |
| 内存占用 | 高 | 低 |
| 组件能力 | HTML 模拟 | 原生组件 |

> **适用场景**：Skyline 特别适合对性能要求高的场景，如电商商品列表、视频流、复杂交互动画、游戏化界面等。

---

## 19.2 传统 Web 开发与小程序的运行机制（双线程模型、WebView 渲染 vs 原生渲染）

### 小程序的双线程模型

微信小程序从设计之初就采用了**双线程架构**，这是理解 Skyline 的基础：

```
┌─────────────────────────────────────┐
│            微信客户端                 │
│                                     │
│  ┌─────────────┐  ┌──────────────┐  │
│  │   逻辑层     │  │   渲染层      │  │
│  │ (JSCore/    │  │ (WebView/    │  │
│  │  V8引擎)    │  │  Skyline)    │  │
│  │             │  │              │  │
│  │ • JS逻辑    │  │ • WXML渲染   │  │
│  │ • 数据处理   │  │ • WXSS样式   │  │
│  │ • API调用   │  │ • 用户交互    │  │
│  └──────┬──────┘  └──────┬───────┘  │
│         │    JS Bridge    │          │
│         └────────────────┘          │
│                ↕                    │
│         ┌──────────────┐            │
│         │   原生模块    │            │
│         │ • 相机/定位   │            │
│         │ • 支付/分享   │            │
│         │ • 文件系统    │            │
│         └──────────────┘            │
└─────────────────────────────────────┘
```

### 为什么要双线程？

1. **安全管控**：逻辑层运行在沙箱中，不能直接操作 DOM，防止恶意代码
2. **性能隔离**：JS 执行不会阻塞 UI 渲染
3. **多端一致性**：不同平台的逻辑层行为一致

### 双线程的通信代价

```
用户点击 → 渲染层捕获事件 → JS Bridge 传递 → 逻辑层处理 → 
JS Bridge 回传 → 渲染层更新 UI

每次 setData 都要走一遍这个流程
```

当数据量大或交互频繁时，通信延迟会成为性能瓶颈。

### WebView 渲染 vs 原生渲染

**WebView 渲染流程：**
```
WXML → 编译为 HTML
WXSS → 编译为 CSS
    → WebView 加载 HTML/CSS
    → 浏览器内核解析 → 布局 → 绘制
    → 显示在屏幕上
```

**Skyline 原生渲染流程：**
```
WXML → 直接映射为原生组件树
WXSS → 转换为原生样式
    → 直接调用系统 UI 框架渲染
    → 显示在屏幕上
```

Skyline 省去了浏览器内核的解析和布局过程，直接使用原生 UI 组件渲染，因此性能更接近原生 App。

### 对比示例：长列表滚动

```
列表项数量: 1000 条

WebView 渲染:
- 快速滚动时出现白屏
- 帧率波动: 20-45fps
- 内存占用: ~150MB

Skyline 渲染:
- 快速滚动无白屏
- 帧率稳定: 58-60fps
- 内存占用: ~80MB
```

> **注意事项**：双线程模型是小程序的基础架构，Skyline 改变的是渲染层的实现方式，并没有改变双线程的本质。逻辑层仍然是 JSCore/V8，但渲染层从 WebView 变成了原生渲染引擎。

---

## 19.3 什么是 Skyline（新的渲染引擎，基于原生渲染，不再依赖 WebView）

### Skyline 的定义

Skyline 是微信小程序团队开发的**新一代渲染引擎**，它不依赖 WebView，而是直接使用微信客户端的原生渲染能力来渲染小程序界面。

### Skyline 的核心特性

**1. 原生渲染**

Skyline 将 WXML/WXSS 直接编译为原生组件树，由系统 UI 框架直接渲染，不再经过浏览器内核。

**2. 工作线程动画（Worklet）**

Skyline 引入了 worklet 机制，允许动画在 UI 线程执行，不经过逻辑层的 JS Bridge，实现丝滑的 60fps 动画。

**3. 更精确的布局**

采用自研的 Flexbox 布局引擎（Yoga），布局计算更高效准确。

**4. 组件一致性**

原生渲染使得组件在不同设备上的表现更加一致，不再受 WebView 内核差异影响。

### Skyline 架构

```
┌──────────────────────────────────────┐
│            微信客户端                  │
│                                      │
│  ┌──────────┐   ┌─────────────────┐  │
│  │  逻辑层   │   │   渲染层         │  │
│  │(JSCore)  │   │ ┌─────────────┐ │  │
│  │          │   │ │  Skyline    │ │  │
│  │ JS逻辑   │←→│ │ 渲染引擎     │ │  │
│  │ 数据处理  │   │ │             │ │  │
│  │          │   │ │ • Flexbox   │ │  │
│  │          │   │ │ • Worklet   │ │  │
│  │          │   │ │ • 原生组件   │ │  │
│  └──────────┘   │ └─────────────┘ │  │
│                 └─────────────────┘  │
└──────────────────────────────────────┘
```

### Skyline 支持的组件

Skyline 支持大部分常用组件，并对部分组件进行了增强：

| 组件 | WebView 模式 | Skyline 模式 | 增强 |
|------|-------------|-------------|------|
| `view` | 支持 | 支持 | - |
| `text` | 支持 | 支持 | - |
| `image` | 支持 | 支持 | 更快的图片渲染 |
| `scroll-view` | 支持 | 支持 | 原生流畅滚动 |
| `swiper` | 支持 | 支持 | 原生动画 |
| `navigator` | 支持 | 支持 | - |
| `input` | 支持 | 支持 | 原生输入体验 |
| `video` | 原生组件 | 原生组件 | - |
| `map` | 原生组件 | 原生组件 | - |
| `canvas` | 原生组件 | 原生组件 | - |

### 何时使用 Skyline

**推荐使用 Skyline 的场景：**
- 长列表/无限滚动页面
- 复杂动画交互页面
- 对滚动流畅度要求高的页面
- 视频流/信息流应用

**暂不推荐使用 Skyline 的场景：**
- 依赖大量 WebView 特有 CSS 特性的页面
- 使用了 Skyline 不支持的组件
- 项目需要兼容非常旧的基础库版本

> **注意事项**：Skyline 需要基础库 2.30.0 以上版本支持。可以通过 `wx.getSystemInfoSync()` 检查基础库版本。

---

## 19.4 Skyline 与 WebView 在开发上的主要差异

### 19.4.1 组件支持方面的差异（部分组件不支持、新增 worklet 组件）

**Skyline 不支持的组件：**

| 组件 | 说明 | 替代方案 |
|------|------|----------|
| `rich-text` | 富文本组件 | 使用 `text` 组件或自定义组件 |
| `editor` | 富文本编辑器 | 暂无直接替代 |
| `live-player` | 直播播放 | 需回退 WebView 模式 |
| `live-pusher` | 直播推流 | 需回退 WebView 模式 |
| `official-account` | 公众号关注 | 需回退 WebView 模式 |

**Skyline 新增/增强的组件：**

```xml
<!-- snapshot 组件 — 用于截屏 -->
<snapshot id="snap" style="width: 300px; height: 200px;">
  <view>内容将被截取</view>
</snapshot>

<!-- grid-view 组件 — 高性能网格布局 -->
<grid-view type="masonry" cross-axis-count="2">
  <view wx:for="{{items}}" wx:key="id" class="grid-item">
    {{item.text}}
  </view>
</grid-view>
```

### 19.4.2 CSS 支持的差异（不支持 float、部分伪类，支持 sticky 等）

**Skyline 不支持的 CSS 特性：**

```css
/* ❌ 不支持 float */
.float-left {
  float: left;
}

/* ❌ 不支持部分伪类 */
.element::before {
  content: '';
}
.element::after {
  content: '';
}

/* ❌ 不支持 CSS 变量（旧版本） */
.var-color {
  color: var(--main-color);
}
```

**Skyline 支持且增强的 CSS 特性：**

```css
/* ✅ 支持 sticky 定位 */
.sticky-header {
  position: sticky;
  top: 0;
  z-index: 100;
}

/* ✅ 支持 Flexbox（推荐布局方式） */
.flex-container {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

/* ✅ 支持 grid 布局 */
.grid-layout {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}
```

**布局方式对比：**

| 布局方式 | WebView | Skyline | 建议 |
|----------|---------|---------|------|
| Flexbox | 支持 | 支持（优化） | 推荐 |
| Grid | 部分支持 | 支持 | 推荐 |
| Float | 支持 | 不支持 | 不推荐 |
| Position sticky | 支持不稳定 | 稳定支持 | 推荐 |
| Position fixed | 支持 | 支持 | 可用 |

### 样式差异处理建议

```css
/* 使用条件编译处理差异 */
/* #ifdef SKYLINE */
.sticky-header {
  position: sticky;
  top: 0;
}
/* #endif */

/* #ifndef SKYLINE */
.sticky-header {
  position: fixed;
  top: 0;
  width: 100%;
}
/* #endif */
```

> **常见坑**：
> 1. Skyline 下 `text` 组件内不能嵌套 `view` 组件，只能嵌套 `text`。
> 2. Skyline 不支持 `display: none`，需要用 `hidden` 属性或条件渲染代替。
> 3. 部分组件的默认样式在 Skyline 下可能与 WebView 不同，需要手动调整。

---

## 19.5 Skyline 增强特性——worklet（UI线程动画、帧率优化）

### 什么是 worklet

Worklet 是 Skyline 引入的核心增强机制，它允许**动画逻辑在 UI 线程直接执行**，而不需要经过逻辑层的 JS Bridge 通信。

### 传统动画 vs Worklet 动画

**传统动画流程（通过 JS Bridge）：**
```
用户滑动 → 渲染层事件 → JS Bridge → 逻辑层 JS 处理 → 
setData → JS Bridge → 渲染层更新 UI
```
每帧都需要跨线程通信，导致延迟和掉帧。

**Worklet 动画流程（UI 线程直接执行）：**
```
用户滑动 → worklet 函数在 UI 线程直接计算 → 渲染层直接更新 UI
```
不需要跨线程通信，实现稳定的 60fps。

### Worklet 的应用场景

1. **手势驱动动画**：拖拽元素、侧滑删除、下拉刷新
2. **滚动联动动画**：滚动时头部缩放、视差滚动
3. **连续动画**：弹簧动画、物理动画
4. **高性能列表**：大量元素的同步动画

### 基本示例：滚动联动头部缩放

```xml
<!-- pages/home/home.wxml -->
<scroll-view 
  type="list" 
  scroll-y 
  bindscroll="onScroll"
  class="scroll-container"
>
  <!-- 头部区域 — 随滚动缩放 -->
  <view class="header" style="height: {{headerHeight}}px;">
    <text class="title">首页</text>
  </view>
  
  <!-- 列表内容 -->
  <view class="list-item" wx:for="{{items}}" wx:key="id">
    {{item.text}}
  </view>
</scroll-view>
```

**传统方式（JS 驱动，性能较差）：**

```javascript
Page({
  data: {
    headerHeight: 200,
    items: [...]
  },

  onScroll(e) {
    const scrollTop = e.detail.scrollTop
    // 每次滚动都要 setData，频繁通信导致卡顿
    const newHeight = Math.max(100, 200 - scrollTop)
    this.setData({ headerHeight: newHeight })
  }
})
```

**Worklet 方式（UI 缒程驱动，流畅 60fps）：**

```javascript
const { shared, timing } = wx.worklet

Page({
  data: {
    items: [...]
  },

  onLoad() {
    // 创建共享变量 — 在 UI 线程可直接访问
    this.headerHeight = shared(200)
    this.opacity = shared(1)
  },

  onScroll(e) {
    'worklet'  // 声明这是 worklet 函数，在 UI 线程执行
    const scrollTop = e.detail.scrollTop
    
    // 直接在 UI 线程修改共享变量，无需 setData
    this.headerHeight.value = Math.max(100, 200 - scrollTop)
    this.opacity.value = Math.max(0.5, 1 - scrollTop / 200)
  }
})
```

### 共享变量（Shared Value）

共享变量是 worklet 机制的核心概念：

```javascript
const { shared, timing, spring } = wx.worklet

Page({
  onLoad() {
    // 创建共享变量
    this.scale = shared(1)          // 缩放比例
    this.translateY = shared(0)     // Y 轴位移
    this.progress = shared(0)       // 动画进度

    // 读取值
    console.log(this.scale.value)   // 1

    // 设置值（直接在 UI 线程生效）
    this.scale.value = 1.5

    // 使用 timing 动画过渡
    this.scale.value = timing(2, { duration: 300 })

    // 使用弹簧动画
    this.scale.value = spring(1, { damping: 15, stiffness: 150 })
  }
})
```

> **注意事项**：
> 1. Worklet 函数内部不能访问 `this.data`，只能使用 `shared` 变量。
> 2. Worklet 函数内不能调用 `setData`，所有 UI 更新通过共享变量驱动。
> 3. 共享变量需要在 `onLoad` 中初始化，在 WXML 中通过 `{{}}` 绑定使用。

---

## 19.6 worklet 函数（wx.worklet.createAnimationAnim 等）

### wx.worklet API 概览

```javascript
const { 
  shared,        // 创建共享变量
  timing,        // 时间动画
  spring,        // 弹簧动画
  delay,         // 延迟动画
  sequence,      // 序列动画
  repeat,        // 重复动画
  decay,         // 衰减动画
} = wx.worklet
```

### 创建动画

**1. timing — 时间驱动动画**

```javascript
// 在 300ms 内将值从当前值过渡到 1.5
this.scale.value = timing(1.5, {
  duration: 300,        // 持续时间
  easing: 'easeInOut',  // 缓动函数
})
```

**2. spring — 弹簧物理动画**

```javascript
// 使用弹簧物理效果过渡到 0
this.translateY.value = spring(0, {
  damping: 15,        // 阻尼系数（越大弹跳越小）
  stiffness: 150,     // 刚度（越大弹簧越硬）
  mass: 1,            // 质量
})
```

**3. sequence — 序列动画**

```javascript
// 先放大再缩小
this.scale.value = sequence(
  timing(1.5, { duration: 200 }),
  timing(1, { duration: 200 })
)
```

**4. repeat — 重复动画**

```javascript
// 重复闪烁
this.opacity.value = repeat(
  sequence(
    timing(0.3, { duration: 500 }),
    timing(1, { duration: 500 })
  ),
  3,    // 重复 3 次（-1 为无限循环）
  true  // 反向播放
)
```

### 完整示例：卡片翻转动效

```xml
<!-- pages/card-flip/card-flip.wxml -->
<view class="card-container">
  <view 
    class="card" 
    style="transform: rotateY({{rotateY}}deg);"
    bindtap="onCardTap"
  >
    <view class="card-front">
      <text>点击翻转</text>
    </view>
    <view class="card-back">
      <text>背面内容</text>
    </view>
  </view>
</view>
```

```javascript
// pages/card-flip/card-flip.js
const { shared, timing, sequence } = wx.worklet

Page({
  data: {
    rotateY: 0
  },

  onLoad() {
    this.rotateY = shared(0)
    this.isFlipped = false
  },

  onCardTap() {
    'worklet'
    if (this.isFlipped) {
      this.rotateY.value = timing(0, { duration: 400 })
    } else {
      this.rotateY.value = timing(180, { duration: 400 })
    }
    this.isFlipped = !this.isFlipped
  }
})
```

```css
/* pages/card-flip/card-flip.wxss */
.card-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  perspective: 1000px;
}

.card {
  width: 300rpx;
  height: 400rpx;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.4s;
}

.card-front, .card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
}

.card-front {
  background: #07c160;
}

.card-back {
  background: #576b95;
  transform: rotateY(180deg);
}
```

### 手势驱动动画示例：拖拽元素

```javascript
// pages/drag/drag.js
const { shared, spring } = wx.worklet

Page({
  onLoad() {
    this.translateX = shared(0)
    this.translateY = shared(0)
  },

  // 手势开始
  onPanStart(e) {
    'worklet'
    this.startX = this.translateX.value
    this.startY = this.translateY.value
  },

  // 手势移动
  onPanUpdate(e) {
    'worklet'
    this.translateX.value = this.startX + e.detail.deltaX
    this.translateY.value = this.startY + e.detail.deltaY
  },

  // 手势结束 — 弹簧回弹
  onPanEnd(e) {
    'worklet'
    this.translateX.value = spring(0, { damping: 15, stiffness: 150 })
    this.translateY.value = spring(0, { damping: 15, stiffness: 150 })
  }
})
```

```xml
<!-- 使用 gesture 组件 -->
<view 
  style="transform: translate({{translateX}}px, {{translateY}}px);"
  bindtouchstart="onPanStart"
  bindtouchmove="onPanUpdate"
  bindtouchend="onPanEnd"
  class="draggable"
>
  <text>拖拽我</text>
</view>
```

> **注意事项**：
> 1. Worklet 函数必须以 `'worklet'` 字符串指令开头，这是告诉编译器该函数在 UI 线程执行的标记。
> 2. Worklet 内部只能访问共享变量和纯函数，不能访问 DOM、不能调用 `wx.*` API。
> 3. 动画完成后如果需要执行 JS 逻辑（如更新数据），使用 `withCallback`：
>    ```javascript
>    this.scale.value = timing(1.5, { duration: 300 }, (finished) => {
>      'worklet'
>      // 动画完成回调
>    })
>    ```

---

## 19.7 Skyline 开发指南（renderer: skyline配置、页面级别开启）

### 全局配置

在 `app.json` 中配置 Skyline：

```json
{
  "pages": [
    "pages/index/index",
    "pages/list/list",
    "pages/webview-page/webview-page"
  ],
  "window": {
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#ffffff",
    "navigationBarTitleText": "Skyline Demo",
    "navigationBarTextStyle": "black"
  },
  "renderer": "skyline",
  "rendererOptions": {
    "skyline": {
      "defaultDisplayBlock": true,
      "disableABTest": true,
      "sdkVersionBegin": "3.0.0",
      "sdkVersionEnd": "15.255.255"
    }
  },
  "componentFramework": "glass-easel",
  "lazyCodeLoading": "requiredComponents"
}
```

### 配置项说明

| 配置项 | 说明 |
|--------|------|
| `renderer` | 全局渲染引擎，设为 `"skyline"` 启用 Skyline |
| `rendererOptions.skyline.defaultDisplayBlock` | 默认 `display: block`（Skyline 下默认是 `flex`） |
| `rendererOptions.skyline.disableABTest` | 关闭 AB 测试 |
| `componentFramework` | 组件框架，需设为 `"glass-easel"` |
| `lazyCodeLoading` | 按需加载组件代码，设为 `"requiredComponents"` |

### 页面级别开启 Skyline

如果不想全局开启，可以逐页面配置。在每个页面的 `.json` 文件中设置：

```json
// pages/index/index.json
{
  "renderer": "skyline",
  "componentFramework": "glass-easel",
  "navigationStyle": "custom",
  "disableScroll": true
}
```

### 混合渲染模式

可以在同一个小程序中混合使用 Skyline 和 WebView：

```json
// pages/index/index.json — 使用 Skyline
{
  "renderer": "skyline"
}

// pages/legacy/legacy.json — 使用 WebView
{
  "renderer": "webview"
}
```

这样可以对性能要求高的页面使用 Skyline，其他页面保持 WebView 不变。

### Skyline 页面的基本结构

```xml
<!-- pages/index/index.wxml -->
<!-- Skyline 页面根节点需要是 scroll-view 或 view -->
<scroll-view 
  type="list"
  scroll-y
  class="page-scroll"
>
  <view class="page-container">
    <!-- 页面内容 -->
    <view class="section">
      <text class="title">Skyline 页面</text>
      <text class="desc">这是一个使用 Skyline 渲染的页面</text>
    </view>
  </view>
</scroll-view>
```

```css
/* pages/index/index.wxss */
/* Skyline 下推荐使用 Flexbox 布局 */
.page-scroll {
  height: 100vh;
  width: 100%;
}

.page-container {
  display: flex;
  flex-direction: column;
  padding: 20rpx;
}

.section {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40rpx 0;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
}

.desc {
  font-size: 28rpx;
  color: #999;
  margin-top: 10rpx;
}
```

> **注意事项**：
> 1. Skyline 页面默认要求根节点是 `scroll-view`（可滚动）或 `view`（不可滚动），不能直接在页面中放多个根节点。
> 2. Skyline 下 `navigationStyle: "custom"` 是推荐配置，自定义导航栏在 Skyline 下性能更好。
> 3. `componentFramework: "glass-easel"` 是 Skyline 的前置要求，它是一个新的组件框架。

---

## 19.8 将旧项目向 Skyline 迁移（迁移步骤、兼容性处理）

### 迁移步骤

**步骤 1：检查基础库版本**

```javascript
// 确保基础库版本 >= 2.30.0
const systemInfo = wx.getSystemInfoSync()
console.log('基础库版本:', systemInfo.SDKVersion)

// 版本比较
function compareVersion(v1, v2) {
  const v1Parts = v1.split('.').map(Number)
  const v2Parts = v2.split('.').map(Number)
  for (let i = 0; i < Math.max(v1Parts.length, v2Parts.length); i++) {
    const diff = (v1Parts[i] || 0) - (v2Parts[i] || 0)
    if (diff !== 0) return diff
  }
  return 0
}

if (compareVersion(systemInfo.SDKVersion, '2.30.0') < 0) {
  console.warn('当前基础库版本过低，不支持 Skyline')
}
```

**步骤 2：更新 app.json 配置**

```json
{
  "renderer": "skyline",
  "rendererOptions": {
    "skyline": {
      "defaultDisplayBlock": true
    }
  },
  "componentFramework": "glass-easel",
  "lazyCodeLoading": "requiredComponents"
}
```

**步骤 3：逐页面迁移**

建议从最简单的页面开始迁移，逐步扩大范围：

```json
// 先在单个页面测试
// pages/test-skyline/test-skyline.json
{
  "renderer": "skyline",
  "componentFramework": "glass-easel",
  "navigationStyle": "custom"
}
```

**步骤 4：修复 CSS 兼容性问题**

```css
/* 检查并替换不支持的 CSS */

/* ❌ float 布局 → ✅ flex 布局 */
/* 旧代码 */
.float-left {
  float: left;
  width: 50%;
}

/* 迁移后 */
.flex-row {
  display: flex;
  flex-direction: row;
}
.flex-item {
  flex: 1;
}

/* ❌ 伪元素 → ✅ 实际节点 */
/* 旧代码 */
.clearfix::after {
  content: '';
  display: block;
  clear: both;
}

/* 迁移后：直接使用 view 组件替代 */

/* ❌ display:none → ✅ hidden 属性或 wx:if */
/* 旧代码 */
.hidden {
  display: none;
}

/* 迁移后 */
/* <view hidden="{{!isVisible}}">内容</view> */
```

**步骤 5：处理不支持的组件**

```xml
<!-- 替换 rich-text 组件 -->
<!-- 旧代码 -->
<rich-text nodes="{{htmlContent}}"></rich-text>

<!-- 迁移后：使用 text 组件或自定义解析 -->
<text>{{textContent}}</text>
<!-- 或对 WebView 页面保持 rich-text -->
```

**步骤 6：测试验证**

```javascript
// 使用条件渲染实现兼容
Page({
  data: {
    isSkyline: false
  },

  onLoad() {
    // 检测当前是否使用 Skyline
    const info = wx.getSystemInfoSync()
    // 可通过页面配置判断
    this.setData({
      isSkyline: true  // 根据实际情况设置
    })
  }
})
```

### 兼容性处理策略

```xml
<!-- 使用条件编译 -->
<!-- #ifdef SKYLINE -->
<scroll-view type="list" scroll-y class="list-skyline">
  <view wx:for="{{items}}" wx:key="id" class="item">
    {{item.text}}
  </view>
</scroll-view>
<!-- #endif -->

<!-- #ifndef SKYLINE -->
<scroll-view scroll-y class="list-webview">
  <view wx:for="{{items}}" wx:key="id" class="item">
    {{item.text}}
  </view>
</scroll-view>
<!-- #endif -->
```

### 迁移检查清单

```
□ 基础库版本检查（≥ 2.30.0）
□ app.json 添加 renderer: "skyline"
□ app.json 添加 componentFramework: "glass-easel"
□ 逐页面迁移（从简单页面开始）
□ CSS 检查：
  □ 替换 float → flexbox
  □ 移除伪元素 ::before/::after
  □ 移除 display:none，改用 hidden 属性
  □ 检查 CSS 变量兼容性
□ 组件检查：
  □ 替换不支持的组件（rich-text, editor 等）
  □ 检查 text 组件嵌套规则
□ 页面根节点检查（确保是 scroll-view 或 view）
□ 自定义导航栏适配
□ 性能测试对比
□ 兼容性回归测试
```

### 常见迁移问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 页面空白 | 根节点不是 scroll-view/view | 修改页面结构 |
| 样式错乱 | float/伪元素不支持 | 改用 flexbox 布局 |
| 组件不显示 | 使用了不支持的组件 | 替换为支持的组件 |
| 动画卡顿 | 仍使用 JS 动画 | 迁移到 worklet 动画 |
| 文字溢出 | text 组件嵌套规则不同 | 调整组件嵌套结构 |

> **注意事项**：
> 1. 迁移是渐进式的，不需要一次性全部改完。可以保持部分页面使用 WebView，部分使用 Skyline。
> 2. 迁移后务必在真机上测试，开发者工具的 Skyline 模拟与真机可能存在差异。
> 3. 建议在迁移前先做性能基准测试，迁移后再对比，确保迁移确实带来了性能提升。

---

## 19.9 本章小结

本章详细介绍了微信小程序的新一代渲染引擎 Skyline，核心知识点如下：

| 知识点 | 核心内容 |
|--------|----------|
| WebView 性能瓶颈 | 渲染性能、通信开销、动画受限、内存占用 |
| 双线程模型 | 逻辑层（JSCore）+ 渲染层（WebView/Skyline） |
| Skyline 定义 | 基于原生渲染的新引擎，绕过 WebView |
| 组件差异 | 部分组件不支持，新增 worklet 相关能力 |
| CSS 差异 | 不支持 float/伪元素，支持 sticky/flexbox |
| Worklet 机制 | UI 线程动画，共享变量，60fps 流畅动画 |
| Worklet 函数 | timing/spring/sequence/repeat 动画 API |
| Skyline 配置 | renderer: skyline、页面级别开启 |
| 旧项目迁移 | 渐进式迁移、CSS 修复、组件替换 |

**核心要点回顾**：

1. **Skyline 的本质**：将渲染层从 WebView 替换为原生渲染引擎，从根源解决性能问题。

2. **双线程模型不变**：Skyline 改变的是渲染层的实现，逻辑层仍然是 JSCore/V8，双线程通信的基本架构没有改变。

3. **Worklet 是关键增强**：通过共享变量和 UI 线程动画，Worklet 解决了传统 JS 动画的通信延迟问题，实现稳定的 60fps。

4. **布局方式转变**：Skyline 推荐使用 Flexbox 布局，不再支持 float 和部分伪元素，需要调整布局习惯。

5. **渐进式迁移**：Skyline 支持页面级别开启，可以与 WebView 共存，无需一次性迁移整个项目。

6. **真机测试很重要**：开发者工具的 Skyline 模拟可能与真机表现不同，迁移后务必在真机上验证。

Skyline 代表了小程序渲染技术的未来方向，对于追求极致性能和体验的小程序，Skyline 是值得投入的方案。下一章将探讨小程序的多端开发能力。
