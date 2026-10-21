# ESP32 灯光控制台 · 玻璃砖版

单文件网页，无任何外部依赖（无 CDN、无字体外链、无构建步骤）。
直接双击 `index.html` 就能用，也可以原样丢进 ESP32 的 LittleFS / SPIFFS。

风格：透明磨砂玻璃砖 + iOS 排版，浅色 / 深色双主题。
界面主色会**跟随当前选择的灯色**变化 —— 选红色，滑块、光晕、开关全部变红。

---

## 装到手机桌面（全屏、无地址栏）

线上地址：**https://onepro07.github.io/esp32-glass/**

- **iPhone / Safari**：打开页面 → 底部「分享」按钮 → 往下滑选「添加到主屏幕」→ 右上角「添加」
- **Android / Chrome**：打开页面 → 右上角菜单 → 「安装应用」或「添加到主屏幕」

添加后桌面上会出现一个深蓝渐变、中间一颗发光灯球的图标。
点开即是全屏界面，没有浏览器地址栏，行为与原生 App 一致（`display: standalone`）。

支持离线：首次访问后会把页面缓存到本地，断网也能打开（`sw.js` 网络优先 + 缓存兜底）。

> 注意：更新代码后，手机上需要**重新打开一次**才能拿到新版；若仍是旧版，下拉刷新或清一次缓存。

## 三种用法

| 场景 | 做法 |
| --- | --- |
| 先看效果 | 双击 `index.html`，默认进入**演示模式**，所有操作只在本页生效 |
| 真机联调 | 点右上角胶囊 → 填设备地址（如 `192.168.4.1`）→ 切到**实时** |
| 刷进 ESP32 | 把 `index.html` 上传到 LittleFS 根目录，访问根路径即可 |

> 演示 / 实时模式、设备地址、主题选择都会记在 `localStorage`，下次打开自动恢复。

---

## 接口约定

实时模式下，操作会以 `POST` + JSON 发到设备。默认约定如下，**按你的固件改路径即可**（改 `API.send` 那一处）：

| 动作 | 请求 | 请求体 |
| --- | --- | --- |
| 主灯开关 / 亮度 | `POST /api/led` | `{"on": true, "brightness": 180}` |
| 全局颜色 | `POST /api/color` | `{"hex": "#3ea6ff"}` |
| 灯效 | `POST /api/effect` | `{"name": "呼吸"}` |
| 单颗灯 | `POST /api/matrix` | `{"index": 5, "on": true, "hex": "#3ea6ff"}` |
| 整组灯 | `POST /api/matrix` | `{"all": true, "on": true, "hex": "#3ea6ff"}` |

灯效名称为中文常量：`常亮` `呼吸` `闪烁` `彩虹` `流水` `爆闪`。

失败处理：4 秒超时，右上角状态点变红并弹提示，不会静默失败。

固件里加 CORS 头即可跨域调试：

```cpp
server.sendHeader("Access-Control-Allow-Origin", "*");
server.sendHeader("Access-Control-Allow-Headers", "Content-Type");
```

（页面和 ESP32 同源时不需要。）

---

## 想改的地方

| 需求 | 位置 |
| --- | --- |
| 灯珠数量（比如 8×8） | CSS `.grid{grid-template-columns:repeat(4,1fr)}` + JS `new Array(16)` |
| 通道标签 `CH1 · GPIO 4` | HTML `#chLabel` |
| 默认灯色 | HTML `#picker value` 与 JS `state.color` |
| 预设色板 | JS `PRESETS` 数组 |
| 接口路径 | JS `API.send` 里的 `/api/...` |
| 玻璃厚度 | CSS `:root` 的 `--card` 透明度与 `.glass` 的 `blur()` |

---

## 兼容性

`backdrop-filter` 是玻璃效果的关键：iOS Safari 9+、Chrome / Edge 76+、Android WebView 76+ 都支持。
不支持的老浏览器会退化成半透明纯色块，功能不受影响。

已针对 iPhone 安全区做了适配（`env(safe-area-inset-*)`），刘海和底部横条都不会遮挡内容。
