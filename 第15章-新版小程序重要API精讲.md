# 第15章 新版小程序重要API精讲

## 15.1 授权与获取用户信息（新版）

### 旧方案 wx.getUserProfile 已废弃

在旧版小程序中，获取用户信息使用 `wx.getUserProfile` 或更早的 `wx.getUserInfo`。但由于隐私保护要求的提升，微信官方已逐步废弃这些 API：

- `wx.getUserInfo`：已废弃，返回匿名数据
- `wx.getUserProfile`：已废弃（2022年10月25日起），返回匿名数据

### 新方案：button + chooseAvatar + chooseMedia

新版获取用户头像和昵称采用**组件驱动**的方式，用户主动点击触发授权。

**获取头像：使用 button 的 open-type="chooseAvatar"**

```html
<!-- wxml -->
<button class="avatar-btn" open-type="chooseAvatar" bind:chooseavatar="onChooseAvatar">
  <image src="{{avatarUrl}}" class="avatar-img" mode="aspectFill"></image>
  <view class="avatar-text">点击选择头像</view>
</button>
```

```javascript
// js
Page({
  data: {
    avatarUrl: '/images/default-avatar.png'
  },

  onChooseAvatar(e) {
    const avatarUrl = e.detail.avatarUrl;
    this.setData({
      avatarUrl: avatarUrl
    });

    // 注意：e.detail.avatarUrl 是临时文件路径
    // 如果需要永久保存，需上传到服务器或云存储
    this.uploadAvatar(avatarUrl);
  },

  uploadAvatar(tempPath) {
    wx.uploadFile({
      url: 'https://api.example.com/upload',
      filePath: tempPath,
      name: 'avatar',
      success: (res) => {
        const data = JSON.parse(res.data);
        console.log('头像上传成功', data.url);
      }
    });
  }
});
```

**获取昵称：使用 input 的 type="nickname"**

```html
<input type="nickname" 
       class="nickname-input" 
       placeholder="请输入昵称" 
       bind:blur="onNicknameBlur" 
       value="{{nickName}}" />
```

```javascript
Page({
  data: {
    nickName: ''
  },

  onNicknameBlur(e) {
    this.setData({
      nickName: e.detail.value
    });
  }
});
```

**获取手机号：button open-type="getPhoneNumber"**

```html
<button open-type="getPhoneNumber" bind:getphonenumber="onGetPhoneNumber">
  授权手机号
</button>
```

```javascript
onGetPhoneNumber(e) {
  if (e.detail.errMsg === 'getPhoneNumber:ok') {
    // e.detail.code 是动态令牌，需发送到服务端解密获取手机号
    const code = e.detail.code;
    wx.request({
      url: 'https://api.example.com/phone',
      method: 'POST',
      data: { code: code },
      success: (res) => {
        console.log('手机号', res.data.phoneNumber);
      }
    });
  } else {
    console.log('用户拒绝授权');
  }
}
```

> **常见坑**：`e.detail.avatarUrl` 返回的是临时路径，小程序重启后会失效。必须在获取后及时上传到服务器保存。

## 15.2 缓存清理

小程序提供了本地缓存 API，数据存储在用户设备本地，单个 key 上限 1MB，总上限 10MB。

### 获取缓存信息

```javascript
// 同步获取缓存信息
const info = wx.getStorageInfoSync();
console.log('keys:', info.keys);          // 所有键名数组
console.log('currentSize:', info.currentSize);  // 当前占用空间（KB）
console.log('limitSize:', info.limitSize);      // 限制空间（KB）

// 异步获取
wx.getStorageInfo({
  success(res) {
    console.log(res.keys, res.currentSize);
  }
});
```

### 删除指定缓存

```javascript
// 同步删除
wx.removeStorageSync('userInfo');

// 异步删除
wx.removeStorage({
  key: 'userInfo',
  success() {
    console.log('删除成功');
  }
});
```

### 清空所有缓存

```javascript
// 同步清空
wx.clearStorageSync();

// 异步清空
wx.clearStorage({
  success() {
    console.log('所有缓存已清空');
  }
});
```

### 实际应用：缓存管理功能

```javascript
Page({
  data: {
    cacheSize: '0 KB'
  },

  onShow() {
    this.updateCacheSize();
  },

  updateCacheSize() {
    const info = wx.getStorageInfoSync();
    const size = info.currentSize;
    let sizeStr = '';

    if (size < 1024) {
      sizeStr = size + ' KB';
    } else if (size < 1024 * 1024) {
      sizeStr = (size / 1024).toFixed(2) + ' MB';
    } else {
      sizeStr = (size / (1024 * 1024)).toFixed(2) + ' GB';
    }

    this.setData({ cacheSize: sizeStr });
  },

  onClearCache() {
    wx.showModal({
      title: '提示',
      content: '确定清除所有缓存？',
      success: (res) => {
        if (res.confirm) {
          // 保留关键数据，清除其他
          const token = wx.getStorageSync('token');
          wx.clearStorageSync();
          wx.setStorageSync('token', token);

          this.updateCacheSize();
          wx.showToast({ title: '清除成功', icon: 'success' });
        }
      }
    });
  }
});
```

> **注意事项**：`wx.clearStorage` 会清除所有缓存，包括登录 token 等关键数据。实际项目中应选择性清除，避免用户被强制退出登录。

## 15.3 交互型组件

### wx.showModal（模态对话框）

```javascript
wx.showModal({
  title: '确认删除',
  content: '删除后不可恢复，确定要删除吗？',
  confirmText: '删除',
  confirmColor: '#ff0000',
  cancelText: '取消',
  success(res) {
    if (res.confirm) {
      console.log('用户点击确定');
    } else if (res.cancel) {
      console.log('用户点击取消');
    }
  }
});

// 编辑输入型 Modal（基础库 2.17.1+）
wx.showModal({
  title: '修改昵称',
  editable: true,
  placeholderText: '请输入新的昵称',
  success(res) {
    if (res.confirm) {
      console.log('输入的内容：', res.content);
    }
  }
});
```

### wx.showActionSheet（操作菜单）

```javascript
wx.showActionSheet({
  itemList: ['保存图片', '分享给好友', '收藏', '举报'],
  itemColor: '#333333',
  success(res) {
    const index = res.tapIndex;
    switch (index) {
      case 0:
        console.log('保存图片');
        break;
      case 1:
        console.log('分享好友');
        break;
      case 2:
        console.log('收藏');
        break;
      case 3:
        console.log('举报');
        break;
    }
  },
  fail(res) {
    console.log('用户取消');
  }
});
```

### wx.showLoading（加载提示）

```javascript
wx.showLoading({
  title: '加载中...',
  mask: true  // 是否显示透明蒙层，防止触摸穿透
});

// 必须手动调用 hideLoading 关闭
setTimeout(() => {
  wx.hideLoading();
}, 2000);
```

### wx.showToast（消息提示）

```javascript
// 成功提示
wx.showToast({
  title: '操作成功',
  icon: 'success',
  duration: 2000
});

// 加载中
wx.showToast({
  title: '加载中',
  icon: 'loading',
  duration: 2000
});

// 自定义图标
wx.showToast({
  title: '自定义',
  image: '/images/custom-icon.png',
  duration: 2000
});

// 纯文字提示
wx.showToast({
  title: '请先登录',
  icon: 'none',
  duration: 2000
});
```

> **常见坑**：
> 1. `showLoading` 和 `showToast` 是互斥的，同时只能显示一个。调用 `showToast` 会自动关闭 `showLoading`。
> 2. `showLoading` 必须与 `hideLoading` 配对使用，否则会一直显示。
> 3. `showModal` 和 `showActionSheet` 是模态的，会阻塞用户操作。

## 15.4 获取系统信息

### 旧版 API（已不推荐）

```javascript
// 旧版一次性获取所有信息（不推荐，部分字段已废弃）
const systemInfo = wx.getSystemInfoSync();
console.log(systemInfo);
```

### 新版拆分 API（推荐）

微信官方将 `getSystemInfo` 拆分为三个独立 API，职责更清晰：

**wx.getDeviceInfo（设备信息）**

```javascript
const deviceInfo = wx.getDeviceInfo();
console.log('设备信息：', deviceInfo);
// {
//   brand: 'iPhone',        // 设备品牌
//   model: 'iPhone 14',     // 设备型号
//   system: 'iOS 16.0',     // 操作系统版本
//   platform: 'ios',        // 客户端平台
//   abi: 'arm64',           // 应用二进制接口类型
//   cpuType: 'arm64',       // CPU 类型
//   memorySize: 4096,       // 设备内存（MB）
//   benchmarkLevel: 10      // 设备性能等级
// }
```

**wx.getWindowInfo（窗口信息）**

```javascript
const windowInfo = wx.getWindowInfo();
console.log('窗口信息：', windowInfo);
// {
//   pixelRatio: 3,            // 设备像素比
//   screenWidth: 393,         // 屏幕宽度（px）
//   screenHeight: 852,        // 屏幕高度（px）
//   windowWidth: 393,         // 可使用窗口宽度
//   windowHeight: 852,        // 可使用窗口高度
//   statusBarHeight: 47,      // 状态栏高度（px）
//   safeArea: { ... },        // 安全区域
//   screenTop: 47             // 安全区域顶部位置
// }
```

**wx.getAppBaseInfo（应用基础信息）**

```javascript
const appBaseInfo = wx.getAppBaseInfo();
console.log('应用信息：', appBaseInfo);
// {
//   SDKVersion: '3.0.0',       // 基础库版本
//   enableDebug: false,        // 是否开启调试
//   host: { appId: 'wx...' },  // 宿主信息
//   language: 'zh_CN',         // 语言
//   version: '8.0.40',         // 微信版本号
//   theme: 'light'             // 系统主题
// }
```

### 实际应用：适配安全区域

```javascript
Page({
  onLoad() {
    const windowInfo = wx.getWindowInfo();
    const statusBarHeight = windowInfo.statusBarHeight;

    // 获取菜单按钮（胶囊）位置信息
    const menuButton = wx.getMenuButtonBoundingClientRect();

    // 计算导航栏高度 = 状态栏高度 + 胶囊高度 + 上下间距
    const navBarHeight = (menuButton.top - statusBarHeight) * 2 + menuButton.height;

    this.setData({
      statusBarHeight: statusBarHeight,
      navBarHeight: navBarHeight,
      menuRight: windowInfo.windowWidth - menuButton.right  // 胶囊右侧距离
    });
  }
});
```

```html
<view class="custom-navbar" style="padding-top: {{statusBarHeight}}px; height: {{navBarHeight}}px;">
  <text class="nav-title">自定义导航栏</text>
</view>
```

> **注意**：`wx.getSystemInfoSync()` 虽然仍可用，但官方建议使用拆分后的新 API。部分字段如 `SDKVersion` 在新 API 中从 `getAppBaseInfo` 获取。

## 15.5 获取网络信息

### 获取当前网络状态

```javascript
wx.getNetworkType({
  success(res) {
    const networkType = res.networkType;
    // networkType: wifi | 2g | 3g | 4g | 5g | unknown | none
    console.log('当前网络类型：', networkType);

    if (networkType === 'none') {
      wx.showToast({
        title: '网络不可用',
        icon: 'none'
      });
    }
  }
});
```

### 监听网络状态变化

```javascript
Page({
  onLoad() {
    // 监听网络状态变化
    wx.onNetworkStatusChange((res) => {
      console.log('网络状态变化');
      console.log('是否连接：', res.isConnected);
      console.log('网络类型：', res.networkType);

      if (!res.isConnected) {
        wx.showToast({
          title: '网络已断开',
          icon: 'none',
          duration: 3000
        });
      } else if (res.networkType !== 'wifi') {
        wx.showToast({
          title: '当前使用移动网络',
          icon: 'none'
        });
      }
    });
  },

  onUnload() {
    // 取消监听（避免重复绑定）
    wx.offNetworkStatusChange();
  }
});
```

### 实际应用：根据网络状态优化体验

```javascript
const app = getApp();

App({
  globalData: {
    networkType: 'wifi',
    isConnected: true
  },

  onLaunch() {
    // 启动时获取网络状态
    wx.getNetworkType({
      success: (res) => {
        this.globalData.networkType = res.networkType;
      }
    });

    // 监听网络变化
    wx.onNetworkStatusChange((res) => {
      this.globalData.networkType = res.networkType;
      this.globalData.isConnected = res.isConnected;
    });
  }
});

// 在页面中使用
Page({
  loadImage(url) {
    if (app.globalData.networkType !== 'wifi') {
      // 非WiFi环境下加载低清图片
      url = url.replace('/large/', '/small/');
    }
    return url;
  }
});
```

## 15.6 获取地理位置信息

### wx.getLocation 基本使用

首先在 `app.json` 中声明权限：

```json
{
  "permission": {
    "scope.userLocation": {
      "desc": "你的位置信息将用于展示附近的电影院"
    }
  },
  "requiredPrivateInfos": ["getLocation"]
}
```

```javascript
Page({
  data: {
    location: null
  },

  getLocation() {
    wx.getLocation({
      type: 'gcj02',  // gcj02 国测局坐标 | wgs84 GPS坐标
      altitude: false, // 是否获取海拔
      isHighAccuracy: false, // 是否高精度定位
      success: (res) => {
        console.log('纬度：', res.latitude);
        console.log('经度：', res.longitude);
        console.log('速度：', res.speed);
        console.log('精度：', res.accuracy);

        this.setData({
          location: {
            latitude: res.latitude,
            longitude: res.longitude
          }
        });

        // 逆地址解析
        this.reverseGeocode(res.latitude, res.longitude);
      },
      fail: (err) => {
        console.error('获取位置失败', err);
      }
    });
  },

  /**
   * 逆地址解析：经纬度 → 详细地址
   * 需要使用第三方地图服务（腾讯地图、高德地图等）
   */
  reverseGeocode(lat, lng) {
    const key = 'YOUR_MAP_API_KEY';  // 替换为你的地图API密钥
    wx.request({
      url: `https://apis.map.qq.com/ws/geocoder/v1/?location=${lat},${lng}&key=${key}`,
      success: (res) => {
        if (res.data.status === 0) {
          const address = res.data.result.address;
          const formatted = res.data.result.formatted_addresses.recommend;
          console.log('详细地址：', formatted);
          this.setData({ address: formatted });
        }
      }
    });
  }
});
```

> **常见坑**：
> 1. `app.json` 中必须配置 `requiredPrivateInfos` 字段，否则无法调用 `getLocation`。
> 2. `type` 参数默认是 `wgs84`，国内地图服务一般需要 `gcj02` 坐标。
> 3. 首次调用会弹出授权窗口，用户拒绝后再次调用需引导用户到设置页开启。

## 15.7 扫码

### wx.scanCode 基本使用

```javascript
Page({
  onScan() {
    wx.scanCode({
      onlyFromCamera: false,  // 是否只从相机扫码，不允许从相册选图
      scanType: ['qrCode', 'barCode'],  // 扫码类型
      success(res) {
        console.log('扫码结果：', res.result);      // 扫码内容
        console.log('扫码类型：', res.scanType);     // QR_CODE 等
        console.log('字符集：', res.charSet);
        console.log('路径：', res.path);             // 当扫码结果为小程序码时
      },
      fail(err) {
        console.log('扫码取消或失败', err);
      }
    });
  }
});
```

### scanType 参数说明

| 值 | 说明 |
|----|------|
| `barCode` | 一维码 |
| `qrCode` | 二维码 |
| `datamatrix` | Data Matrix 码 |
| `pdf417` | PDF417 码 |

### 扫码结果处理

```javascript
onScan() {
  wx.scanCode({
    scanType: ['qrCode'],
    success: (res) => {
      const result = res.result;

      // 判断扫码内容类型
      if (result.startsWith('http')) {
        // 网址链接
        wx.showModal({
          title: '检测到链接',
          content: result,
          confirmText: '复制',
          success: (modalRes) => {
            if (modalRes.confirm) {
              wx.setClipboardData({ data: result });
            }
          }
        });
      } else if (result.startsWith('wxapp:')) {
        // 小程序码，跳转到指定页面
        const path = result.replace('wxapp:', '');
        wx.navigateTo({ url: path });
      } else {
        // 其他内容
        wx.showModal({
          title: '扫码结果',
          content: result
        });
      }
    }
  });
}
```

## 15.8 用户授权与授权二次拉起

### wx.getSetting 获取用户授权状态

```javascript
wx.getSetting({
  success(res) {
    console.log('授权设置：', res.authSetting);
    // {
    //   "scope.userInfo": true,        // 用户信息（已废弃）
    //   "scope.userLocation": true,    // 地理位置
    //   "scope.address": false,        // 通讯地址
    //   "scope.record": false,         // 录音功能
    //   "scope.writePhotosAlbum": true // 保存到相册
    // }
  }
});
```

### scope 列表

| scope | 说明 |
|-------|------|
| `scope.userInfo` | 用户信息（已废弃，改用 button 组件） |
| `scope.userLocation` | 地理位置 |
| `scope.userLocationBackground` | 后台定位 |
| `scope.address` | 通讯地址 |
| `scope.record` | 录音功能 |
| `scope.writePhotosAlbum` | 保存图片/视频到相册 |
| `scope.camera` | 摄像头 |
| `scope.bluetooth` | 蓝牙 |
| `scope.werun` | 微信运动步数 |
| `scope.phoneNumber` | 手机号（button 组件触发） |

### 授权二次拉起

当用户之前拒绝过某个授权，再次调用相关 API 时不会弹出授权窗口。此时需要引导用户到设置页面手动开启。

```javascript
/**
 * 检查并请求授权
 * @param {string} scope - 授权范围
 * @returns {Promise<boolean>}
 */
function checkAndRequestAuth(scope) {
  return new Promise((resolve, reject) => {
    wx.getSetting({
      success(res) {
        if (res.authSetting[scope] === true) {
          // 已授权
          resolve(true);
        } else if (res.authSetting[scope] === false) {
          // 之前拒绝过，需要引导到设置页
          wx.showModal({
            title: '需要授权',
            content: '需要您授权该功能才能正常使用，是否前往设置？',
            confirmText: '去设置',
            success(modalRes) {
              if (modalRes.confirm) {
                wx.openSetting({
                  success(settingRes) {
                    if (settingRes.authSetting[scope]) {
                      resolve(true);
                    } else {
                      resolve(false);
                    }
                  }
                });
              } else {
                resolve(false);
              }
            }
          });
        } else {
          // 未请求过授权，首次请求
          wx.authorize({
            scope: scope,
            success() {
              resolve(true);
            },
            fail() {
              resolve(false);
            }
          });
        }
      },
      fail(err) {
        reject(err);
      }
    });
  });
}
```

## 15.9 通用授权处理流程

### 完整的授权封装函数

将授权逻辑封装为一个通用工具函数，可以在整个项目中复用。

```javascript
// utils/auth.js

/**
 * 通用授权处理
 * @param {string} scope - 授权范围，如 'scope.userLocation'
 * @param {string} desc - 授权用途描述
 * @returns {Promise<boolean>} 是否授权成功
 */
function ensureAuth(scope, desc) {
  return new Promise((resolve, reject) => {
    // 第一步：检查当前授权状态
    wx.getSetting({
      success(res) {
        const authValue = res.authSetting[scope];

        if (authValue === true) {
          // 已授权，直接返回成功
          resolve(true);
          return;
        }

        if (authValue === undefined) {
          // 未请求过，发起首次授权
          wx.authorize({
            scope: scope,
            success() {
              resolve(true);
            },
            fail() {
              // 首次授权被拒绝，引导到设置页
              guideToSetting(scope, desc, resolve, reject);
            }
          });
          return;
        }

        if (authValue === false) {
          // 之前拒绝过，引导到设置页
          guideToSetting(scope, desc, resolve, reject);
          return;
        }
      },
      fail(err) {
        reject(err);
      }
    });
  });
}

/**
 * 引导用户到设置页面开启权限
 */
function guideToSetting(scope, desc, resolve, reject) {
  wx.showModal({
    title: '权限申请',
    content: desc || '该功能需要您授权后才能使用，是否前往设置开启？',
    confirmText: '去设置',
    cancelText: '不需要',
    success(modalRes) {
      if (modalRes.confirm) {
        wx.openSetting({
          success(settingRes) {
            if (settingRes.authSetting[scope]) {
              // 用户在设置页开启了权限
              resolve(true);
            } else {
              // 用户在设置页仍未开启
              resolve(false);
            }
          },
          fail(err) {
            reject(err);
          }
        });
      } else {
        // 用户拒绝去设置
        resolve(false);
      }
    }
  });
}

module.exports = {
  ensureAuth
};
```

### 在页面中使用

```javascript
const { ensureAuth } = require('../../utils/auth');

Page({
  /**
   * 获取地理位置（带授权检查）
   */
  async onGetLocation() {
    try {
      const hasAuth = await ensureAuth(
        'scope.userLocation',
        '需要获取您的地理位置来推荐附近的电影院'
      );

      if (!hasAuth) {
        wx.showToast({ title: '授权被拒绝', icon: 'none' });
        return;
      }

      wx.getLocation({
        type: 'gcj02',
        success: (res) => {
          console.log('位置获取成功', res.latitude, res.longitude);
        },
        fail: (err) => {
          console.error('定位失败', err);
        }
      });
    } catch (err) {
      console.error('授权流程异常', err);
    }
  },

  /**
   * 保存图片到相册（带授权检查）
   */
  async onSaveImage() {
    const hasAuth = await ensureAuth(
      'scope.writePhotosAlbum',
      '需要相册权限来保存图片'
    );

    if (!hasAuth) return;

    wx.saveImageToPhotosAlbum({
      filePath: '/images/poster.png',
      success() {
        wx.showToast({ title: '保存成功', icon: 'success' });
      }
    });
  }
});
```

### 授权流程图

```
用户触发功能
    │
    ▼
wx.getSetting 检查授权状态
    │
    ├── 已授权 (true) ──────────────────────→ 执行功能
    │
    ├── 未请求过 (undefined) ──→ wx.authorize ──→ 成功 → 执行功能
    │                                          └→ 失败 → 引导设置
    │
    └── 被拒绝过 (false) ──→ showModal 提示 ──→ 确认 → wx.openSetting
                                                   └→ 取消 → 功能不可用
```

> **常见坑**：
> 1. `wx.authorize` 只在用户**从未授权也从未拒绝**时才会弹出授权窗口。被拒绝过之后不会再次弹窗。
> 2. `scope.userInfo` 已不再支持 `wx.authorize` 方式，必须通过 button 组件的 `open-type` 获取。
> 3. 在 `app.json` 的 `permission` 中声明权限说明，可以让用户在授权弹窗中看到用途说明。

## 15.10 本章小结

### 核心知识点回顾

本章系统学习了小程序新版重要 API，主要内容如下：

**1. 用户信息获取（新版方案）**
- `wx.getUserProfile` 已废弃
- 新方案：`button open-type="chooseAvatar"` 获取头像 + `input type="nickname"` 获取昵称
- `button open-type="getPhoneNumber"` 获取手机号

**2. 缓存管理**
- `wx.getStorageInfoSync`：获取缓存信息
- `wx.removeStorage`：删除指定缓存
- `wx.clearStorage`：清空所有缓存
- 实际应用中应选择性清除，保留登录态等关键数据

**3. 交互组件**
- `wx.showModal`：模态对话框，支持编辑输入
- `wx.showActionSheet`：底部操作菜单
- `wx.showLoading / hideLoading`：加载提示，需配对使用
- `wx.showToast`：轻量消息提示

**4. 系统信息（新版拆分 API）**
- `wx.getDeviceInfo`：设备信息（品牌、型号、系统）
- `wx.getWindowInfo`：窗口信息（屏幕尺寸、安全区域）
- `wx.getAppBaseInfo`：应用信息（SDK版本、主题）
- 建议使用新 API 替代旧的 `getSystemInfoSync`

**5. 网络信息**
- `wx.getNetworkType`：获取当前网络类型
- `wx.onNetworkStatusChange`：监听网络变化
- 根据网络状态优化图片加载等体验

**6. 地理位置**
- `wx.getLocation` 获取经纬度
- 需在 `app.json` 配置 `permission` 和 `requiredPrivateInfos`
- 逆地址解析需借助第三方地图 API

**7. 扫码**
- `wx.scanCode` 支持一维码和二维码
- `scanType` 指定扫码类型
- 根据扫码结果内容做不同处理

**8. 授权机制**
- `wx.getSetting`：检查授权状态
- `wx.authorize`：首次请求授权
- `wx.openSetting`：引导用户到设置页开启权限
- 授权二次拉起的核心：拒绝后无法再弹窗，必须引导到设置页

**9. 通用授权封装**
- 封装 `ensureAuth` 函数处理三种授权状态
- 返回 Promise，支持 async/await 调用
- 统一处理授权引导逻辑，可在全项目复用

### 实战要点

- 新版用户信息获取必须通过 button 组件触发，不能自动获取
- `showLoading` 和 `showToast` 互斥，注意调用顺序
- 地理位置 API 必须配置 `requiredPrivateInfos`
- 授权被拒绝后，只能通过 `wx.openSetting` 引导用户手动开启
- 授权封装函数应在项目初期就规划好，避免后期到处补丁

在下一章中，我们将进入小程序云开发的世界，学习 Serverless 架构和云数据库的使用。
