# 第09章 Component组件化编程

## 9.1 小程序的tab选项卡

在介绍 Component 之前，先来看一个在小程序开发中极其常见的交互形态——**tab 选项卡**。tab 选项卡指的是页面顶部或底部的一组标签按钮，用户点击不同标签时切换显示不同的内容区域。它广泛用于商品分类、新闻频道、个人中心等场景。

### 9.1.1 原生 tabBar 配置

小程序在 `app.json` 中内置了 `tabBar` 配置，用于在页面底部生成原生切换栏。它的优点是性能高、体验与原生一致；缺点是只能放在底部、样式定制能力有限。

```json
{
  "pages": [
    "pages/index/index",
    "pages/movie/movie",
    "pages/profile/profile"
  ],
  "tabBar": {
    "color": "#999",
    "selectedColor": "#1296db",
    "backgroundColor": "#ffffff",
    "borderStyle": "black",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "images/home.png",
        "selectedIconPath": "images/home-active.png"
      },
      {
        "pagePath": "pages/movie/movie",
        "text": "电影",
        "iconPath": "images/movie.png",
        "selectedIconPath": "images/movie-active.png"
      },
      {
        "pagePath": "pages/profile/profile",
        "text": "我的",
        "iconPath": "images/profile.png",
        "selectedIconPath": "images/profile-active.png"
      }
    ]
  }
}
```

### 9.1.2 页面内自定义 tab 选项卡

当需求超出原生 tabBar 的能力（例如顶部标签、可滑动标签、带红点提示等），就需要在页面内自己实现选项卡。这类"可复用的 UI 片段"正是 Component 组件化的典型场景。

一个最简单的页面内 tab 选项卡实现如下：

```html
<!-- pages/index/index.wxml -->
<view class="tabs">
  <view
    class="tab-item {{currentTab === index ? 'active' : ''}}"
    wx:for="{{tabs}}"
    wx:key="index"
    data-index="{{index}}"
    bindtap="onTabTap"
  >{{item}}</view>
</view>

<view class="tab-content" wx:if="{{currentTab === 0}}">首页内容</view>
<view class="tab-content" wx:if="{{currentTab === 1}}">电影内容</view>
<view class="tab-content" wx:if="{{currentTab === 2}}">我的内容</view>
```

```javascript
// pages/index/index.js
Page({
  data: {
    tabs: ['首页', '电影', '我的'],
    currentTab: 0
  },
  onTabTap(e) {
    const index = e.currentTarget.dataset.index
    this.setData({ currentTab: Number(index) })
  }
})
```

```css
/* pages/index/index.wxss */
.tabs {
  display: flex;
  height: 88rpx;
  line-height: 88rpx;
  border-bottom: 1rpx solid #eee;
}
.tab-item {
  flex: 1;
  text-align: center;
  font-size: 30rpx;
  color: #666;
}
.tab-item.active {
  color: #1296db;
  font-weight: bold;
  border-bottom: 4rpx solid #1296db;
}
```

> **坑点提示**：`data-index` 传到事件处理函数时会被序列化为字符串，使用前建议用 `Number()` 转换，否则 `currentTab === index` 这种严格相等比较会始终为 `false`。

如果这个选项卡会在多个页面中使用，更好的做法是把它封装成一个自定义组件。这正是 Component 的用武之地，后续章节将逐步展开。

---

## 9.2 Component与Template的对比

小程序提供了两种复用 UI 的机制：**template 模板** 和 **Component 自定义组件**。初学者常分不清何时用哪种，下面对比说明。

### 9.2.1 template 模板

template 是 WXML 层面的代码片段复用机制，它只关心"长什么样"，不关心"怎么行为"。

```html
<!-- templates/movie-card/movie-card.wxml -->
<template name="movieCard">
  <view class="card">
    <image src="{{movie.image}}" mode="aspectFill" />
    <view class="title">{{movie.title}}</view>
    <view class="rate">{{movie.rate}}</view>
  </view>
</template>
```

使用时：

```html
<!-- pages/index/index.wxml -->
<import src="/templates/movie-card/movie-card.wxml" />

<template is="movieCard" data="{{movie: item}}" />
```

template 的特点：
- 仅是 WXML 片段，没有独立的 JS 逻辑、没有生命周期、没有事件系统。
- 样式需要在使用它的页面中重新定义，或通过全局样式覆盖。
- 适合"纯展示型"且不需要交互逻辑的片段。

### 9.2.2 Component 自定义组件

Component 是完整的独立单元，拥有自己的结构（wxml）、样式（wxss）、逻辑（js）和配置（json），并具备属性、方法、生命周期、事件等能力。

```javascript
// components/movie-card/movie-card.js
Component({
  properties: {
    movie: Object
  },
  methods: {
    onTap() {
      this.triggerEvent('cardtap', { movie: this.data.movie })
    }
  }
})
```

### 9.2.3 选用建议

| 维度 | template 模板 | Component 组件 |
|------|--------------|---------------|
| 复用层级 | 仅 WXML 结构 | 结构 + 样式 + 逻辑 |
| 是否有 JS 逻辑 | 无 | 有 |
| 是否有生命周期 | 无 | 有 |
| 是否有独立样式作用域 | 无（需手动管理） | 有（`addGlobalClass` / `styleIsolation`） |
| 是否支持事件通信 | 不支持 | 支持 `triggerEvent` |
| 是否支持组件嵌套 | 不支持 | 支持 |
| 适用场景 | 静态展示片段 | 带交互、带状态、需复用的功能模块 |

**经验法则**：
- 如果片段只是"显示数据"，没有交互、没有状态，用 template 足够。
- 如果片段有自己的状态、事件、逻辑，或者会在多个页面中复用，就用 Component。
- 新项目建议统一使用 Component，可维护性和扩展性更好。

---

## 9.3 Component的基础

### 9.3.1 创建自定义组件的步骤

创建一个自定义组件通常分四步：

1. 在项目根目录（或 `components` 文件夹）下新建一个与组件同名的文件夹，例如 `components/stars/`。
2. 在该文件夹下新建四个文件：`stars.json`、`stars.wxml`、`stars.wxss`、`stars.js`。
3. 在 `stars.json` 中声明 `"component": true`，表示这是一个组件而非页面。
4. 在 `stars.js` 中调用 `Component({...})` 构造器定义组件逻辑。

也可以使用微信开发者工具的"新建 Component"功能自动生成这四个文件。

### 9.3.2 组件的 json 文件

```json
{
  "component": true,
  "usingComponents": {}
}
```

`"component": true` 是组件的标识。如果该组件内部还要引用其他组件，则在 `usingComponents` 中声明。

### 9.3.3 组件的 wxml 文件

组件的 wxml 与页面 wxml 写法一致，但要注意：组件 wxml 的根节点默认会有一个外层包装，样式隔离机制下，组件内不能直接使用页面 wxss 中的类名。

```html
<!-- components/stars/stars.wxml -->
<view class="stars">
  <text wx:for="{{starsArray}}" wx:key="index" class="star">{{item}}</text>
  <text class="score">{{score}}</text>
</view>
```

### 9.3.4 组件的 wxss 文件

```css
/* components/stars/stars.wxss */
.stars {
  display: flex;
  align-items: center;
}
.star {
  font-size: 28rpx;
  color: #f5a623;
  margin-right: 4rpx;
}
.score {
  margin-left: 10rpx;
  font-size: 26rpx;
  color: #888;
}
```

### 9.3.5 组件的 js 文件

```javascript
// components/stars/stars.js
Component({
  // 组件的属性列表，由外部传入
  properties: {
    score: {
      type: Number,
      value: 0
    }
  },
  // 组件的内部数据
  data: {
    starsArray: []
  },
  // 组件方法
  methods: {
    computeStars(score) {
      const full = Math.floor(score / 2)
      const half = score % 2 >= 1 ? 1 : 0
      const empty = 5 - full - half
      const arr = []
      for (let i = 0; i < full; i++) arr.push('★')
      for (let i = 0; i < half; i++) arr.push('☆')
      for (let i = 0; i < empty; i++) arr.push('☆')
      return arr
    }
  },
  lifetimes: {
    attached() {
      this.setData({ starsArray: this.computeStars(this.data.score) })
    }
  }
})
```

### 9.3.6 在页面中使用组件

在页面的 `json` 中注册组件：

```json
{
  "usingComponents": {
    "stars": "/components/stars/stars"
  }
}
```

在页面 wxml 中使用：

```html
<stars score="{{9}}"></stars>
```

> **注意**：`usingComponents` 中的路径既可以是绝对路径（以 `/` 开头，从项目根目录算起），也可以是相对路径。推荐使用绝对路径，避免页面移动后路径失效。

---

## 9.4 Component的属性

`properties` 是组件对外暴露的接口，父组件/页面通过属性向组件传值。

### 9.4.1 properties 定义

```javascript
Component({
  properties: {
    // 简写：只指定类型
    title: String,

    // 完整写法：指定类型 + 默认值 + 观察者
    score: {
      type: Number,
      value: 0,
      observer: function (newVal, oldVal) {
        console.log('score 变化了', newVal, oldVal)
      }
    },

    // 对象类型
    movie: {
      type: Object,
      value: {}
    },

    // 数组类型
    tags: {
      type: Array,
      value: []
    }
  }
})
```

支持的类型有：`String`、`Number`、`Boolean`、`Object`、`Array`、`null`（表示不限制类型）。

### 9.4.2 数据类型与默认值

`value` 字段用于设置属性的默认值。当父组件没有传入该属性时，组件内部使用默认值。

```javascript
properties: {
  // 默认空字符串
  name: {
    type: String,
    value: ''
  },
  // 默认 false
  showHeader: {
    type: Boolean,
    value: false
  },
  // 默认空数组
  list: {
    type: Array,
    value: () => []   // 注意：对象/数组类型默认值推荐用函数返回
  }
}
```

> **坑点提示**：在旧版本基础库中，`Object` 和 `Array` 类型的默认值如果是对象字面量/数组字面量，所有组件实例会共享同一引用，导致数据串改。新版本基础库已修复，但为了兼容性，复杂类型的默认值建议使用工厂函数。

### 9.4.3 属性的观察者 observers

`properties` 中的 `observer` 写法已逐渐被 `observers` 数据监听器取代。`observers` 功能更强，可以同时监听多个字段，甚至监听对象内部字段的变化。

```javascript
Component({
  properties: {
    movie: Object,
    score: Number
  },
  data: {
    starsArray: []
  },
  observers: {
    // 监听单个字段
    'score': function (newVal) {
      this.setData({ starsArray: this.computeStars(newVal) })
    },
    // 监听多个字段
    'movie, score': function (movie, score) {
      console.log('movie 或 score 变化', movie, score)
    },
    // 监听对象内部字段（纯数据字段监听）
    'movie.title': function (title) {
      console.log('电影标题变为', title)
    }
  }
})
```

> **注意**：`observers` 中的回调函数不能调用 `setData` 来设置正在监听的那个字段本身，否则会形成无限循环。可以设置其他字段。

---

## 9.5 Component的JS文件结构

一个完整的组件 js 文件由若干固定字段组成：

```javascript
Component({
  // 1. 外部传入的属性
  properties: {
    score: {
      type: Number,
      value: 0
    }
  },

  // 2. 组件内部数据
  data: {
    starsArray: [],
    isLoading: false
  },

  // 3. 数据监听器（响应 properties / data 的变化）
  observers: {
    'score': function (newVal) {
      this.setData({ starsArray: this.computeStars(newVal) })
    }
  },

  // 4. 组件方法
  methods: {
    computeStars(score) {
      // ...
      return []
    },
    onTap() {
      this.triggerEvent('tap', { score: this.data.score })
    }
  },

  // 5. 组件生命周期
  lifetimes: {
    created() {},
    attached() {},
    ready() {},
    moved() {},
    detached() {}
  },

  // 6. 组件所在页面的生命周期
  pageLifetimes: {
    show() {},
    hide() {},
    resize() {}
  },

  // 7. 组件配置项
  options: {
    multipleSlots: true,        // 开启多 slot
    addGlobalClass: true,       // 接受全局样式
    styleIsolation: 'isolated'  // 样式隔离
  },

  // 8. 外部样式类
  externalClasses: ['custom-class'],

  // 9. 组件接受的外部传入的 slot
  // (在 wxml 中使用 <slot/>)
})
```

### 字段说明

| 字段 | 作用 |
|------|------|
| `properties` | 接收外部传入的数据，类似 Vue 的 props |
| `data` | 组件内部私有数据，类似 Vue 的 data |
| `observers` | 监听 properties / data 的变化，类似 Vue 的 watch |
| `methods` | 组件的方法，包括事件处理函数和自定义方法 |
| `lifetimes` | 组件自身生命周期 |
| `pageLifetimes` | 组件所在页面的生命周期 |
| `options` | 组件配置 |
| `externalClasses` | 外部样式类，允许父组件传入样式 |

> **注意**：组件的 `methods` 之外（顶层）不要直接写函数，所有事件处理和业务方法都应放在 `methods` 中。早期版本支持在顶层写方法，但已不推荐。

---

## 9.6 Component的生命周期函数

组件生命周期是理解组件运行机制的关键。组件生命周期分为两类：**组件自身生命周期**（`lifetimes`）和**组件所在页面的生命周期**（`pageLifetimes`）。

### 9.6.1 组件自身生命周期

| 生命周期 | 触发时机 | 常见用途 |
|---------|---------|---------|
| `created` | 组件实例刚刚被创建，此时还不能调用 `setData`，`properties` 也还没赋值 | 初始化一些不需要依赖视图的纯数据 |
| `attached` | 组件已进入页面节点树，`properties` 已就绪，可以 `setData` | 发起网络请求、初始化数据 |
| `ready` | 组件布局完成，可以获取节点信息（`createSelectorQuery`） | 获取节点尺寸、做动画 |
| `moved` | 组件被移动到另一个位置 | 较少使用 |
| `detached` | 组件被从页面节点树移除 | 清理定时器、取消请求 |

代码示例：

```javascript
Component({
  lifetimes: {
    created() {
      console.log('组件 created')
      // 此时不要调用 setData，data 中的初始值已生效
    },
    attached() {
      console.log('组件 attached，可以安全使用 setData')
      this.loadDetail()
    },
    ready() {
      console.log('组件 ready，布局完成')
      const query = this.createSelectorQuery()
      query.select('.box').boundingClientRect(rect => {
        console.log('节点尺寸', rect)
      }).exec()
    },
    detached() {
      console.log('组件 detached，清理资源')
      if (this.timer) clearTimeout(this.timer)
    }
  },
  methods: {
    loadDetail() {
      // 模拟请求
      this.timer = setTimeout(() => {
        this.setData({ detail: { title: '肖申克的救赎' } })
      }, 1000)
    }
  }
})
```

> **坑点提示**：`created` 阶段调用 `setData` 不会报错但不会生效，因为此时数据系统尚未与视图关联。需要设置初始数据应直接写在 `data` 中，需要响应外部属性应使用 `observers` 或在 `attached` 中处理。

### 9.6.2 组件所在页面的生命周期

当组件所在的页面发生 `show` / `hide` / `resize` 等变化时，组件可以通过 `pageLifetimes` 感知到。这在需要在页面切换时刷新数据的场景非常有用。

```javascript
Component({
  pageLifetimes: {
    show() {
      console.log('页面显示，可以刷新数据')
      this.refresh()
    },
    hide() {
      console.log('页面隐藏，可以暂停轮询')
      this.pausePolling()
    },
    resize(size) {
      console.log('页面尺寸变化', size)
    }
  },
  methods: {
    refresh() {},
    pausePolling() {}
  }
})
```

### 9.6.3 生命周期的完整流程

理解整个流程对排查问题很有帮助：

1. 用户进入页面 → 页面 `onLoad` / `onShow` / `onReady`
2. 页面 wxml 中引用的组件 → 组件 `created` → 组件 `attached` → 组件 `ready`
3. 用户切到其他 tab → 页面 `onHide` → 组件 `pageLifetimes.hide`
4. 用户切回 → 页面 `onShow` → 组件 `pageLifetimes.show`
5. 用户返回/页面卸载 → 页面 `onUnload` → 组件 `detached`

> **注意**：旧版本写法把生命周期直接写在 `Component({...})` 顶层（如 `attached() {}`），仍能工作但已被标记为废弃，应统一放入 `lifetimes` 字段中。

---

## 9.7 本章小结

本章系统介绍了小程序 Component 组件化编程的基础：

1. **tab 选项卡** 是常见交互形态，原生 `tabBar` 适合底部导航，页面内复杂选项卡适合封装为组件。
2. **Component vs Template**：template 只复用结构，Component 复用结构+样式+逻辑，新项目推荐 Component。
3. **Component 基础**：一个组件由 json/wxml/wxss/js 四个文件组成，json 中需声明 `"component": true`。
4. **properties** 是组件对外的数据接口，支持类型、默认值、`observers` 监听器。
5. **组件 js 结构**：包含 properties / data / observers / methods / lifetimes / pageLifetimes / options 等字段。
6. **生命周期**：`created → attached → ready → moved → detached`，配合 `pageLifetimes` 感知页面切换。

掌握 Component 组件化编程是构建复杂小程序的基础。下一章将通过电影模块的实战，把 stars、movie、movie-list 三个组件串联起来，深入体会组件化带来的复用与解耦价值。
