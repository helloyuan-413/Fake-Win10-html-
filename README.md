#赢10 模拟桌面
赢10.py
文本
胜利。齐普:
获胜:
shell-server.js
        index.html
        cmdshell.html
        win10.py
“”
一个跑在浏览器里的 Windows 10 风格桌面模拟器， HTML,JS,python 实现，通过 REST API 与后端交互。

---

Windows 10、HTML、JS、Python、通过 REST API 飞后 飞后 飞后端悦 Ѹ�后端�儿 的

### 登录 / 注册

通过 `/api/login` 和 `/api/register` 提交用户名 + 密码，成功后获得 `uuid`CSS 网格 芬开按列排列 縐人号

### 桌面

- 渐变壁纸，图标使用 CSS Grid 自动按列排列
- `layoutDesktopIcons()` 根据窗口高度动态计算每列行数（`ICON_CELL_H = 66`，最小 6 行）
- 单击选中、双击打开、右键弹出菜单

### 窗口

- 标题栏拖拽移动（鼠标直接拖，触摸需长按 250ms）
- 八个方向缩放：`n / s / e / w / ne / nw / se / sw`
- 最小化 / 最大化 / 关闭三个按钮
- 点击任意窗口自动提升 `z-index翝存 / 另存为 / 60 秒自动才存 /
- 拖动时位置会被限制在视口内

### 内置应用（来自 `APP_LIST`）

| id | 名称 | 说明 |
| --- | --- | --- |
| browser | 浏览器 | 多标签页 + 地址栏 + 沙箱 iframe + 保存页面 |
| calculator | 计算器 | 4×4 按钮，支持括号与四则运算 |
| explorer | 资源管理器 | 文件浏览、新建、重命名、删除、隐藏、分享 |
| notepad | 记事本 | 保存 / 另存为 / 60 秒自动保存 / `Ctrl+S` |
| typing | 打字练习 | 服务端生成句子、轮询进度、计算准确率 |
| taskmgr | 任务管理器 | 列出所有窗口、结束任务、1.5 秒刷新 |
| sms | 短信 | 收件箱 / 发件箱、发送短信、详情窗口 |
| cmdshell | CMD Shell | 内嵌 `cmdshell.html`，可执行 py / js / sh 等文件 |

> `cmdshell` 标记为 `menuOnly: true`，不在桌面上显示，只出现在开始菜单。

### 文件类型识别

根据扩展名决定打开方式：

| 扩展名 | 打开方式 |
| --- | --- |
| `.url` | 浏览器打开（自动补 https://） |
| `.html` / `.htm` | 浏览器 iframe 加载 |
| `.py` / `.js` / `.java` / `.sh` / `.bat` / `.cmd` | CMD Shell 运行 |
| `.apk` | 弹出安装对话框 |
| 其他 | 记事本打开 |

图标 emoji 也会根据扩展名切换（🐍 `.py`、☕ `.java`、🌐 `.html`、📦 `.pkg` 等）。

### 拖拽移动文件

- 鼠标移动超过 8px / 触摸超过 10px 触发
- 出现蓝色幽灵提示（`#dragGhost`）
- 投放目标用 `data-drop-target="1"` 标记
- 通过 `elementFromPoint` 判断落点
- 禁止把文件夹拖进自己里面

### 右键菜单

- **桌面**：刷新桌面、新建、打开 CMD Shell、打开资源管理器
- **文件**：打开、分享、以文本编辑器打开（可执行文件）、重命名、隐藏、删除
- **已安装 App**：打开、打开所在文件夹、以文本编辑器打开 APK、卸载

### 浏览器下载拦截

通过 `attachDownloadInterceptor()` 向 iframe 注入脚本，拦截：

- `<a download>` 链接
- 常见下载后缀（zip / apk / pdf / mp3 / mp4 / 图片 等）
- `target="_blank"`（改为新标签打开）
- `window.open()`
- 指向下载地址的表单提交

拦截后弹出保存对话框，可选择保存到 `private` 或 `public`。

---

## 后端 API

### 用户

| 方法 | 路径 | 参数 | 返回 |
| --- | --- | --- | --- |
| POST | `/api/login` | `{name, password}` | `{uuid, name}` |
| POST | `/api/register` | `{name, password}` | `{uuid, name}` |

### 文件

`scope` 取值 `private` / `public`。

| 方法 | 路径 | 参数 |
| --- | --- | --- |
| GET | `/api/files/list` | `uuid, scope, path` |
| GET | `/api/files/read` | `uuid, scope, name` |
| GET | `/api/files/raw` | `uuid, scope, name` |
| POST | `/api/files/write` | `{uuid, scope, name, content}` |
| POST | `/api/files/mkdir` | `{uuid, scope, name}` |
| POST | `/api/files/move` | `{uuid, scope, from, to}` |
| POST | `/api/files/delete` | `{uuid, scope, name}` |
| POST | `/api/files/hide` | `{uuid, scope, name, hidden}` |
| POST | `/api/files/share` | `{uuid, scope, name}` → `{ok, url}` |

### 短信

| 方法 | 路径 | 参数 |
| --- | --- | --- |
| GET | `/api/messages/inbox` | `uuid` |
| GET | `/api/messages/outbox` | `uuid` |
| POST | `/api/messages/send` | `{uuid, to, content}` |

### APK

| 方法 | 路径 | 参数 |
| --- | --- | --- |
| GET | `/api/apk/list` | `uuid, scope` |
| GET | `/api/apk/info` | `uuid, scope, name` |
| GET | `/api/apk/icon` | `uuid, scope, pkg` |
| POST | `/api/apk/install` | `{uuid, scope, name}` |
| POST | `/api/apk/uninstall` | `{uuid, scope, pkg}` |
| POST | `/api/apk/run` | `{uuid, scope, name}` |

`/api/apk/run` 返回 `kind` 字段决定启动方式：

- `html` → 浏览器打开 `/api/files/raw`
- `python` / `node` / `shell` → CMD Shell 打开
- 其他（原生 Android）→ 弹出提示

### 打字练习

| 方法 | 路径 | 参数 |
| --- | --- | --- |
| POST | `/api/generate` | `{uuid}` |
| GET | `/api/progress` | `uuid` |

`/api/progress` 返回 `status` 为 `generating`（带 `received` 字符数）或 `ready`（带 `sentences` 数组）。

---

## 操作说明

| 操作 | 效果 |
| --- | --- |
| 单击图标 | 选中 |
| 双击图标 | 打开 |
| 右键图标 | 上下文菜单 |
| 拖图标到文件夹 | 移动文件 |
| 拖标题栏 | 移动窗口 |
| 拖窗口边缘 | 缩放 |
| `Ctrl+S` | 记事本保存 |
| 长按标题栏 250ms | 触屏拖动窗口 |

---

## 已知限制

- 浏览器的 iframe 受同源策略和目标站点的 CSP 限制，部分网站无法加载
- 跨域页面无法读取 DOM，保存时会显示「跨域限制，无法读取 HTML 源码」
- 二进制文件导入时转成 Base64 DataURL，体积约增大 33%
- 原生 Android APK 无法运行，只会弹出提示
- 移动端浏览器一般不触发 `contextmenu`，右键菜单在触屏上不可用

---

## 部署

1. 把 `index.html` 和 `cmdshell.html` 、`win10.py` 、`shell-server.js` 放到后端静态目录，与 `/api` 同源
2. 后端实现上面列出的接口
3. 浏览器访问页面，注册账号后进入桌面
