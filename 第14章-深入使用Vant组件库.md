# 第14章 深入使用Vant组件库

## 14.1 Vant的特点

Vant 是有赞团队开源的一套轻量、可靠的移动端组件库，提供了微信小程序版本的 **Vant Weapp**。它已成为小程序开发中最流行的第三方 UI 组件库之一。

### Vant Weapp 的核心优势

**1. 轻量可靠**
- 组件平均体积不到 1KB（min+gzip）
- 单元测试覆盖率超过 90%
- 经历了有赞电商业务的大量线上验证

**2. Vue 风格的 API 设计**
Vant 的 API 设计参考了 Vue 生态组件库的约定，使得有 Vue 开发经验的开发者可以快速上手。例如属性命名采用驼峰式，事件采用 `bind:click` 的形式。

```html
<!-- Vant 的 API 风格与 Vue 组件类似 -->
<van-button type="primary" bind:click="onClick">按钮</van-button>
```

**3. 丰富的组件数量**
Vant 提供了 60+ 个高质量组件，涵盖以下类别：

| 类别 | 代表组件 |
|------|---------|
| 基础组件 | Button、Cell、Icon、Image |
| 表单组件 | Field、Switch、Checkbox、Radio、Picker |
| 展示组件 | Tag、Card、Collapse、TreeSelect |
| 导航组件 | Tab、Tabs、NavBar、Tabs |
| 反馈组件 | Dialog、Toast、Loading、ActionSheet |
| 业务组件 | AddressList、Coupon、Sku |

**4. 完善的 TypeScript 支持**
所有组件都提供了完整的 TypeScript 类型定义，方便在 TS 项目中使用。

**5. 支持主题定制**
通过 CSS 变量和 ConfigProvider 组件，可以轻松实现全局主题定制。

## 14.2 Vant使用指南

### 安装与初始化

**方式一：npm 安装（推荐）**

```bash
# 在小程序项目根目录执行
npm init -y
npm i @vant/weapp -S --production
```

然后在微信开发者工具中：
1. 点击 **工具 → 构建 npm**
2. 勾选 **使用 npm 模块**
3. 在 `app.json` 中移除 `"style": "v2"`（避免样式冲突）

**方式二：下载源码**

从 GitHub 下载 Vant Weapp 源码，将 `dist` 目录拷贝到项目的 `vant` 目录中。

### 按需引入

在页面的 JSON 配置文件中按需引入组件，这是推荐的做法，可以有效减小代码包体积。

```json
// pages/setting/setting.json
{
  "usingComponents": {
    "van-cell": "@vant/weapp/cell/index",
    "van-cell-group": "@vant/weapp/cell-group/index",
    "van-switch": "@vant/weapp/switch/index",
    "van-dialog": "@vant/weapp/dialog/index",
    "van-button": "@vant/weapp/button/index"
  }
}
```

```html
<!-- pages/setting/setting.wxml -->
<van-cell-group>
  <van-cell title="消息通知" is-link>
    <van-switch slot="right-icon" size="24" checked="{{ notification }}" 
                bind:change="onNotificationChange" />
  </van-cell>
</van-cell-group>
```

### 全局引入

如果某个组件在多个页面中频繁使用，可以在 `app.json` 中全局引入：

```json
// app.json
{
  "usingComponents": {
    "van-button": "@vant/weapp/button/index",
    "van-toast": "@vant/weapp/toast/index"
  }
}
```

> **注意**：全局引入的组件会包含在主包中，如果过多会增大主包体积。建议只全局引入高频基础组件，其余按需引入。

### 主题定制

**方式一：CSS 变量定制（推荐）**

Vant Weapp 所有组件都基于 CSS 变量开发，可以直接覆盖变量值来定制主题。

```css
/* app.wxss - 全局覆盖 CSS 变量 */
page {
  --button-primary-background-color: #07c160;
  --button-primary-border-color: #07c160;
  --cell-background-color: #f7f8fa;
  --switch-on-background-color: #07c160;
}
```

```html
<!-- 也可以在单个组件上覆盖 -->
<van-button style="--button-primary-background-color: #ff6b6b;">
  自定义颜色按钮
</van-button>
```

**方式二：ConfigProvider 组件**

```html
<van-config-provider color="#07c160">
  <van-button type="primary">主题色按钮</van-button>
  <van-tag type="primary">标签</van-tag>
</van-config-provider>
```

```json
{
  "usingComponents": {
    "van-config-provider": "@vant/weapp/config-provider/index"
  }
}
```

> **常见坑**：小程序基础库版本需 2.6.5 以上才支持 CSS 变量。低版本基础库中 CSS 变量定制不生效，需要使用样式覆盖方案。

## 14.3 构建设置页面

下面使用 Vant 组件库构建一个完整的设置页面，综合运用 Cell、Switch、Dialog 等组件。

### 页面结构

`pages/setting/setting.wxml`：

```html
<view class="setting-container">
  <!-- 用户信息区域 -->
  <view class="user-section">
    <van-cell-group>
      <van-cell title="头像" center>
        <image slot="right-icon" class="user-avatar" 
               src="{{userInfo.avatar}}" mode="aspectFill" />
      </van-cell>
      <van-cell title="昵称" value="{{userInfo.nickName}}" is-link 
                bind:click="onEditNickName" />
      <van-cell title="ID" value="{{userInfo.userId}}" />
    </van-cell-group>
  </view>

  <!-- 功能设置区域 -->
  <view class="setting-section">
    <view class="section-title">功能设置</view>
    <van-cell-group>
      <van-cell title="消息通知" center>
        <van-switch slot="right-icon" size="24" 
                    checked="{{ settings.notification }}" 
                    bind:change="onSwitchChange" 
                    data-key="notification" />
      </van-cell>
      <van-cell title="自动播放视频" center>
        <van-switch slot="right-icon" size="24" 
                    checked="{{ settings.autoPlay }}" 
                    bind:change="onSwitchChange" 
                    data-key="autoPlay" />
      </van-cell>
      <van-cell title="省流量模式" center>
        <van-switch slot="right-icon" size="24" 
                    checked="{{ settings.saveFlow }}" 
                    bind:change="onSwitchChange" 
                    data-key="saveFlow" />
      </van-cell>
    </van-cell-group>
  </view>

  <!-- 其他设置 -->
  <view class="setting-section">
    <view class="section-title">其他</view>
    <van-cell-group>
      <van-cell title="清除缓存" value="{{cacheSize}}" is-link 
                bind:click="onClearCache" />
      <van-cell title="关于我们" is-link bind:click="onAbout" />
      <van-cell title="检查更新" is-link bind:click="onCheckUpdate" />
    </van-cell-group>
  </view>

  <!-- 退出登录按钮 -->
  <view class="logout-section">
    <van-button type="danger" block bind:click="onLogout">
      退出登录
    </van-button>
  </view>
</view>

<!-- Dialog 组件 -->
<van-dialog id="van-dialog" />
```

### 页面逻辑

`pages/setting/setting.js`：

```javascript
const Dialog = require('@vant/weapp/dialog/dialog');

Page({
  data: {
    userInfo: {
      avatar: '',
      nickName: '微信用户',
      userId: '88888888'
    },
    settings: {
      notification: true,
      autoPlay: false,
      saveFlow: false
    },
    cacheSize: '0 KB'
  },

  onLoad() {
    this.loadSettings();
    this.calculateCache();
  },

  /**
   * Switch 开关切换
   */
  onSwitchChange(e) {
    const key = e.currentTarget.dataset.key;
    const value = e.detail;

    this.setData({
      [`settings.${key}`]: value
    });

    // 保存到本地存储
    wx.setStorageSync('settings', this.data.settings);

    wx.showToast({
      title: value ? '已开启' : '已关闭',
      icon: 'none'
    });
  },

  /**
   * 清除缓存
   */
  onClearCache() {
    Dialog.confirm({
      title: '提示',
      message: '确定要清除所有缓存数据吗？'
    }).then(() => {
      // 保留用户登录信息，清除其他缓存
      const token = wx.getStorageSync('token');
      wx.clearStorage();
      wx.setStorageSync('token', token);

      this.setData({ cacheSize: '0 KB' });

      wx.showToast({
        title: '清除成功',
        icon: 'success'
      });
    }).catch(() => {
      // 用户点击取消
    });
  },

  /**
   * 退出登录
   */
  onLogout() {
    Dialog.confirm({
      title: '退出登录',
      message: '确定要退出当前账号吗？'
    }).then(() => {
      wx.clearStorage();
      wx.reLaunch({
        url: '/pages/login/login'
      });
    }).catch(() => {});
  },

  loadSettings() {
    const settings = wx.getStorageSync('settings');
    if (settings) {
      this.setData({ settings });
    }
  },

  calculateCache() {
    const info = wx.getStorageInfoSync();
    const sizeKB = info.currentSize;
    let sizeStr = '';
    if (sizeKB < 1024) {
      sizeStr = sizeKB + ' KB';
    } else {
      sizeStr = (sizeKB / 1024).toFixed(1) + ' MB';
    }
    this.setData({ cacheSize: sizeStr });
  }
});
```

### 页面配置

`pages/setting/setting.json`：

```json
{
  "usingComponents": {
    "van-cell": "@vant/weapp/cell/index",
    "van-cell-group": "@vant/weapp/cell-group/index",
    "van-switch": "@vant/weapp/switch/index",
    "van-dialog": "@vant/weapp/dialog/index",
    "van-button": "@vant/weapp/button/index"
  },
  "navigationBarTitleText": "设置"
}
```

> **注意事项**：使用 `van-dialog` 时，必须在 WXML 中放置 `<van-dialog id="van-dialog" />` 标签，并在 JS 中引入 `dialog` 方法。这是 Vant 的命令式调用模式。

## 14.4 Vant组件的样式定制概述

### CSS 变量定制

Vant 组件内置了大量的 CSS 变量，通过覆盖这些变量可以快速实现主题定制。常用变量如下：

```css
/* app.wxss 全局样式 */
page {
  /* 按钮主色 */
  --button-primary-background-color: #07c160;
  --button-primary-border-color: #07c160;

  /* 单元格 */
  --cell-vertical-padding: 16rpx;
  --cell-horizontal-padding: 32rpx;
  --cell-text-color: #323233;
  --cell-background-color: #ffffff;

  /* 开关 */
  --switch-on-background-color: #07c160;
  --switch-node-background-color: #ffffff;

  /* 对话框 */
  --dialog-confirm-button-text-color: #07c160;

  /* 标签 */
  --tag-padding: 4rpx 16rpx;
  --tag-text-color: #ffffff;
}
```

### ConfigProvider 动态主题

`ConfigProvider` 可以实现动态切换主题，适用于夜间模式等场景：

```html
<van-config-provider theme-vars="{{ darkThemeVars }}">
  <view class="dark-mode-container">
    <van-cell-group>
      <van-cell title="深色模式下的单元格" />
    </van-cell-group>
    <van-button type="primary">深色模式按钮</van-button>
  </view>
</van-config-provider>
```

```javascript
Page({
  data: {
    darkThemeVars: {
      cellBackgroundColor: '#1a1a1a',
      cellTextColor: '#e0e0e0',
      buttonPrimaryBackgroundColor: '#3a7afe'
    }
  }
});
```

### 样式定制的优先级

```
行内样式 > 页面 CSS 变量覆盖 > ConfigProvider > 组件默认变量
```

> **常见坑**：CSS 变量必须定义在 `page` 选择器或组件的根节点上，定义在普通 `view` 上可能无法被子组件继承。

## 14.5 组件的外部样式类

### externalClasses 的使用场景

在某些情况下，CSS 变量无法满足定制需求，比如需要修改组件内部某个特定元素的 `margin`、`padding` 等布局样式。这时可以使用 `externalClasses` 机制。

### Vant 组件的外部样式类

Vant 的很多组件都提供了外部样式类接口。以 `van-cell` 为例：

```javascript
// van-cell 组件支持的 externalClasses
externalClasses: ['title-class', 'value-class', 'label-class']
```

使用方式：

```html
<van-cell 
  title="自定义标题" 
  title-class="custom-title"
  value="自定义值"
  value-class="custom-value">
</van-cell>
```

```css
/* 页面样式 */
.custom-title {
  color: #ff6b6b;
  font-weight: bold;
  font-size: 32rpx;
}

.custom-value {
  color: #07c160;
}
```

### 自定义组件中的 externalClasses

在自己开发的组件中也可以使用 `externalClasses`：

```javascript
// components/my-card/my-card.js
Component({
  externalClasses: ['card-class', 'title-class'],

  properties: {
    title: String
  }
});
```

```html
<!-- components/my-card/my-card.wxml -->
<view class="card card-class">
  <text class="title title-class">{{title}}</text>
  <slot></slot>
</view>
```

使用该组件：

```html
<my-card title="标题" card-class="my-card-style" title-class="my-title-style">
  <text>内容</text>
</my-card>
```

```css
.my-card-style {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20rpx;
}

.my-title-style {
  color: #fff;
  font-size: 36rpx;
}
```

> **注意事项**：外部样式类与 `styleIsolation: 'shared'` 模式不同。`externalClasses` 更精准，只允许修改指定的元素；而 `shared` 模式则允许外部样式全局影响组件。

## 14.6 插槽（slot）

插槽是组件化开发中非常重要的概念，它允许在使用组件时向组件内部传入自定义内容。

### 默认插槽

默认插槽是最简单的插槽形式，用于在组件中插入任意内容。

```html
<!-- 组件定义 components/my-card/my-card.wxml -->
<view class="card">
  <view class="card-header">头部</view>
  <view class="card-body">
    <slot></slot>  <!-- 默认插槽 -->
  </view>
</view>
```

```html
<!-- 使用组件 -->
<my-card>
  <text>这是通过插槽传入的内容</text>
  <image src="/images/pic.png"></image>
</my-card>
```

### 具名插槽

当组件需要多个插槽位置时，使用具名插槽区分。

```html
<!-- 组件定义 components/my-layout/my-layout.wxml -->
<view class="layout">
  <view class="header">
    <slot name="header"></slot>
  </view>
  <view class="content">
    <slot></slot>  <!-- 默认插槽 -->
  </view>
  <view class="footer">
    <slot name="footer"></slot>
  </view>
</view>
```

```html
<!-- 使用组件 -->
<my-layout>
  <view slot="header">
    <text>这是头部内容</text>
  </view>

  <view>
    <text>这是主体内容，放入默认插槽</text>
  </view>

  <view slot="footer">
    <text>这是底部内容</text>
  </view>
</my-layout>
```

> **注意**：小程序中使用具名插槽时，需要使用 `slot="name"` 属性，而不是 Vue 中的 `v-slot:name` 语法。

### 作用域插槽

作用域插槽允许子组件向父组件传递数据，父组件根据子组件的数据来渲染插槽内容。小程序从基础库 2.6.2 开始支持作用域插槽。

```html
<!-- 组件定义 components/my-list/my-list.wxml -->
<view class="list">
  <block wx:for="{{items}}" wx:key="index">
    <view class="list-item">
      <!-- 将当前项数据传递给插槽 -->
      <slot item="{{item}}" index="{{index}}"></slot>
    </view>
  </block>
</view>
```

```html
<!-- 使用组件 -->
<my-list items="{{todos}}">
  <view slot="default" slot-scope="props">
    <text>序号：{{props.index + 1}}</text>
    <text>内容：{{props.item.text}}</text>
    <text>状态：{{props.item.done ? '已完成' : '未完成'}}</text>
  </view>
</my-list>
```

### 多个具名作用域插槽

```html
<!-- 组件定义 -->
<view class="component">
  <slot name="title" data="{{titleData}}"></slot>
  <slot name="content" data="{{contentData}}"></slot>
</view>
```

```html
<!-- 使用 -->
<my-component>
  <view slot="title" slot-scope="scope">
    <text>{{scope.data.name}}</text>
  </view>
  <view slot="content" slot-scope="scope">
    <text>{{scope.data.text}}</text>
  </view>
</my-component>
```

> **常见坑**：
> 1. 作用域插槽中 `slot-scope` 的变量名可以自定义，但需要在作用域插槽的 `slot` 属性中指定对应的具名插槽名。
> 2. 默认插槽使用作用域时，`slot="default"` 可以省略，但 `slot-scope` 不能省略。
> 3. 小程序的作用域插槽语法与 Vue 不同，不要混淆。

### Vant 中的插槽使用

Vant 组件大量使用了插槽，例如 `van-cell`：

```html
<van-cell title="标题" value="内容">
  <!-- 使用 right-icon 插槽自定义右侧图标 -->
  <van-icon slot="right-icon" name="search" class="custom-icon" />
</van-cell>

<van-cell>
  <!-- 使用 title 插槽自定义标题区域 -->
  <view slot="title">
    <text>自定义标题</text>
    <van-tag type="danger">NEW</van-tag>
  </view>
</van-cell>
```

## 14.7 本章小结

### 核心知识点回顾

本章深入学习了 Vant Weapp 组件库的使用，涵盖以下内容：

1. **Vant 的特点**：轻量、美观、Vue 风格 API、60+ 组件、TypeScript 支持、主题定制能力。

2. **安装与引入**：支持 npm 安装和源码下载两种方式。推荐按需引入以减小代码包体积，高频组件可全局引入。

3. **设置页面实战**：综合运用 Cell、CellGroup、Switch、Dialog、Button 等组件，构建了完整的设置页面。重点掌握了 Dialog 的命令式调用方式。

4. **样式定制**：两种方式——CSS 变量定制（全局覆盖和组件级覆盖）和 ConfigProvider 动态主题。

5. **外部样式类 externalClasses**：用于精准控制组件内部特定元素的样式，Vant 组件提供了 `title-class`、`value-class` 等外部样式类接口。

6. **插槽机制**：
   - 默认插槽：插入任意内容
   - 具名插槽：多位置插入，使用 `slot="name"`
   - 作用域插槽：子组件向父组件传数据，使用 `slot-scope`

### 选择建议

| 需求场景 | 推荐方案 |
|---------|---------|
| 修改主题色 | CSS 变量定制 |
| 动态切换主题 | ConfigProvider |
| 修改组件某元素布局 | externalClasses |
| 组件内插入自定义内容 | slot 插槽 |
| 减小代码包体积 | 按需引入 |

### 实战要点

- `van-dialog` 使用前必须在页面放置 `<van-dialog id="van-dialog" />` 并引入 `dialog` 方法
- CSS 变量需定义在 `page` 选择器或组件根节点上才能生效
- 作用域插槽需要基础库 2.6.2 以上支持
- 构建 npm 后如果组件不生效，尝试重启开发者工具
- Vant 组件的事件绑定使用 `bind:事件名` 而非 `bind事件名`

在下一章中，我们将学习小程序新版重要 API，包括授权机制、缓存管理、系统信息获取等核心能力。
