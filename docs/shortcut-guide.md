# WLOC 定位修改 - 指南

**简体中文** · [English](shortcut-guide.en.md)

## 工作原理

```
用户在手机 Safari 里打开取景页面
  → 在地图上选点 / 搜索地名 / 粘贴地图链接
  → 点「保存到设备」
  → 页面请求 https://gs-loc.apple.com/wloc-settings/save?lon=x&lat=y
  → 代理模块拦截请求 → wloc-settings.js 写入 $persistentStore
  → 下次触发 Apple 定位时 → wloc.js 读取坐标 → 修改定位响应
```

如果模块未启用 → 请求不会被拦截 → 页面会提示你检查 MITM / 模块配置。

---

## 使用方法

### 1. 安装模块（一次性）
订阅对应平台的模块并开启 MITM。

### 2. 打开取景页面
在 Safari 里打开公共取景页面（建议添加到主屏幕）：
```
https://你的-worker-域名/
```

> Worker 是一个纯静态页面，不存储任何数据。坐标会直接写入你设备的本地存储。

### 3. 选择位置
- **点击地图** —— 直接选点
- **搜索地名** —— 输入类似「上海外滩」的内容
- **粘贴链接** —— 复制 Apple 地图 / Google 地图 / 高德 / 百度 的分享链接
- **当前位置** —— 使用浏览器定位

### 4. 保存到设备
点「保存到设备」→ 出现 ✓ 即表示成功。

---

## 部署公共取景页面

Worker 是一个纯静态页面服务，无需任何绑定：

```bash
cd worker
npx wrangler deploy
```

或在 CF Dashboard → Workers → 新建 Worker → 粘贴 `wloc-worker.js` → 部署。

无需 KV、数据库或环境变量。

---

## 模块配置

模块包含两条脚本规则（自动配置，无需用户操作）：

| 规则 | 类型 | 路径 | 用途 |
|------|------|------|------|
| Apple WLOC | http-response | `/clls/wloc` | 修改定位响应 |
| WLOC Settings | http-request | `/wloc-settings/save` | 接收来自取景页面的写入 |

MITM 主机名：`gs-loc.apple.com, gs-loc-cn.apple.com`（模块已包含）

---

## 保存失败排查

当页面显示红色横幅时，检查：
1. **模块已启用** —— 确认代理工具里的 WLOC 模块开关已打开
2. **MITM 证书** —— CA 证书已安装并信任
3. **MITM 主机名** —— 已包含 `gs-loc.apple.com`
4. **代理连接** —— 当前网络已走代理（Safari 请求经过代理）

---

## 备选方案：手动编辑（BoxJS）

如果你不想用取景页面，可以在 BoxJS 里直接编辑 `wloc_settings`：
```json
{"longitude":121.4737,"latitude":31.2304,"accuracy":25}
```

优先级：已保存坐标 > 模块参数 > 默认值
