# iOS Location Spoofer — 多平台 iOS 定位欺骗模块

基于 [acheong08/ios-location-spoofer](https://github.com/acheong08/ios-location-spoofer) 的通用 HTTPS 解密定位欺骗配置。

## 原理

iPhone 扫描周边 WiFi 热点 + 蜂窝基站 → 将 BSSID 列表发送到 `gs-loc.apple.com/clls/wloc` → Apple 返回这些热点/基站的 GPS 坐标 → iOS 据此三角定位。

本方案利用代理软件的内置 HTTPS 解密 (MITM) 拦截 Apple 的定位响应，将响应中的 WiFi 热点坐标和蜂窝基站坐标全部改写为指定位置。iPhone 收到伪造数据后认为设备就在目标坐标。

> MITM 和处理全部在本地完成，不连接任何第三方服务器。

---

## 各平台一键导入

| 平台 | 价格 | 导入方式 |
|------|------|---------|
| **Shadowrocket** (小火箭) | $2.99 | [点击导入](https://raw.githubusercontent.com/mekos2772/ios-location-spoofer/main/Shadowrocket/ios-location-spoofer.sgmodule) |
| **Surge** | $49.99 | [点击导入](https://raw.githubusercontent.com/mekos2772/ios-location-spoofer/main/Shadowrocket/ios-location-spoofer.sgmodule) |
| **Loon** | $4.99 | [点击导入](https://raw.githubusercontent.com/mekos2772/ios-location-spoofer/main/Shadowrocket/ios-location-spoofer.lnplugin) |
| **Quantumult X** | $7.99 | [点击导入](https://raw.githubusercontent.com/mekos2772/ios-location-spoofer/main/Shadowrocket/ios-location-spoofer.snippet) |
| **Stash** | $3.99 | [点击导入](https://raw.githubusercontent.com/mekos2772/ios-location-spoofer/main/Shadowrocket/ios-location-spoofer.stoverride) |

> 核心 JS 逻辑完全一致 — 仅 I/O 适配层不同（Surge/Rocket/Loon/Stash 共用 `location-spoofer.js`，QX 使用 `location-spoofer-qx.js` 处理 base64 编解码差异）。

---

## 使用步骤

### ① 启用 HTTPS 解密并信任证书

1. 在代理 App 中开启 HTTPS 解密/MitM 功能
2. 生成 CA 证书 → 设置 → 通用 → VPN 与设备管理 → 安装
3. 设置 → 通用 → 关于本机 → 证书信任设置 → **启用对应 CA**

### ② 导入模块

各 App 导入路径：

- **Shadowrocket**：配置 → 模块 → 右上角 + → 输入上方导入链接
- **Surge**：首页 → 模块 → 安装新模块 → 输入导入链接
- **Loon**：设置 → 插件 → 添加插件 → 输入导入链接
- **Quantumult X**：设置 → 重写 → 添加 → 输入导入链接
- **Stash**：覆写 → 安装覆写 → 输入导入链接

### ③ 启用后重连

1. 确保模块已勾选启用
2. 断开并重新连接 VPN
3. 设置 → 隐私 → 定位服务 → 关闭 → 等 5 秒 → 重新开启
4. 打开 Apple Maps / 天气 / 任何使用系统定位的 App 验证

---

## 自定义坐标

修改 `location-spoofer-config.json` 后上传到你的服务器，在模块参数中添加 `&configUrl=你的URL`。或直接在模块的脚本参数中修改：

```
argument=mode=response&latitude=你的纬度&longitude=你的经度...
```

### 各地坐标参考

| 地点 | 纬度 | 经度 |
|------|------|------|
| 北京天安门 | 39.9042 | 116.4074 |
| 上海外滩 | 31.2304 | 121.4737 |
| 深圳南山 | 22.5431 | 113.9295 |
| 台北 101 | 25.0330 | 121.5654 |
| 东京新宿 | 35.6895 | 139.6917 |
| 纽约时代广场 | 40.7580 | -73.9855 |
| Apple Park | 37.3349 | -122.0090 |

---

## 配置参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `enabled` | `true` | 启用开关 |
| `latitude` | `37.3349` | 目标纬度 (-90~90) |
| `longitude` | `-122.00902` | 目标经度 (-180~180) |
| `horizontalAccuracy` | `39` | 水平精度/米（越小越"准"） |
| `verticalAccuracy` | `1000` | 垂直精度/米 |
| `altitude` | `530` | 海拔/米 |
| `motionActivityType` | `63` | 运动类型（63=未知） |
| `motionActivityConfidence` | `467` | 运动置信度 |
| `failOpen` | `true` | 出错时放行原响应 |
| `debug` | `false` | 调试日志开关 |

---

## 支持的域名

- `gs-loc.apple.com` — Apple 全球定位服务
- `gs-loc-cn.apple.com` — Apple 中国大陆定位服务  
- `bluedot.is.autonavi.com` — 高德地图定位 CDN
- `bluedot.is.autonavi.com.gds.alibabadns.com` — 阿里 DNS 高德 CDN

---

## 文件说明

```
Shadowrocket/
├── ios-location-spoofer.sgmodule     ← Shadowrocket / Surge
├── ios-location-spoofer.lnplugin     ← Loon
├── ios-location-spoofer.snippet      ← Quantumult X
├── ios-location-spoofer.stoverride   ← Stash
├── location-spoofer.js               ← 核心 JS（4 平台共用）
├── location-spoofer-qx.js            ← QX 专用 JS
└── location-spoofer-config.json      ← 坐标配置
```

---

## 局限

- 仅影响 Apple 系统定位（WiFi/蜂窝三角定位），不影响硬件 GPS
- 室内/城市环境效果最佳（iOS 主要依赖 WiFi 定位）
- 不影响 App 自带第三方定位 SDK
- 需完整 MITM 证书信任链

## 许可

AGPL-3.0 — [acheong08/ios-location-spoofer](https://github.com/acheong08/ios-location-spoofer)
