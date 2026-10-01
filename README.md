# 主义征服 — Cloudflare Pages 部署版

一款纯前端的六边形网格回合制策略游戏。本目录可直接推送到 GitHub，并在 Cloudflare Pages 上托管，任何人都能在浏览器里游玩。

## 本目录内容

```
主义征服-cloudflare/
├── index.html   # 游戏主文件（已自包含全部 CSS/JS，单文件即可运行）
├── pages.json   # Cloudflare Pages 构建配置（纯静态，无需构建）
└── README.md    # 本说明
```

> `index.html` 与桌面游玩文件 `主义征服.html` 内容完全一致，仅多了一段「部署环境检测 + 后端功能优雅降级」逻辑（见下）。改游戏逻辑时改桌面 `主义征服.html`，再复制覆盖本目录的 `index.html` 即可。

## 部署环境检测原理

游戏在运行时按 `location.hostname` 自动判断环境：

- **本地**（`file:` / `localhost` / `127.0.0.1`）→ 视为「有后端」，所有功能开放。
- **静态托管**（如 `*.pages.dev` 或你的自有域名）→ 自动进入「静态部署模式」，对需要后端的功能做优雅降级。

无需任何环境变量、无需 Worker，浏览器里真实生效。

## ✅ 完全支持（无需后端，部署后所有人可玩）

- **单人 PvE 模式** — 与 4 个 AI 阵营对战（含叛乱势力、绝境进攻、攻势加成）
- **新手教学** — 完整引导流程
- **地图编辑器** — 本地创建 / 编辑地图
- **PvP 直连** — WebRTC 点对点对战，**无需任何服务器**，复制连接码发给朋友即可
- **本地存档 / 读档** — 浏览器下载 / 上传 JSON 文件
- **快捷键** — WASD / 方向键平移视角、ESC 关弹窗等

## ⚠️ 依赖后端、已降级的功能

部署到静态托管后，以下功能会自动提示并安全降级（不会报错 / 白屏）：

| 功能 | 静态部署下的行为 |
|------|----------------|
| WebSocket 多人联机 | 大厅提示「需自建服务器」，可手动填 `ws://` 地址连接你自己搭的服务器；或改用 PvP 直连 |
| 云端存档 | 点「保存到网盘」自动回退为「下载到本地」 |
| 云端地图工坊 | 点「工坊」提示不支持，请用本地「地图编辑器」 |
| 账号系统 | 登录 / 注册不可用（单机 / P2P 不需要账号） |

## 部署步骤（GitHub → Cloudflare Pages）

### 1. 推送到 GitHub
```bash
cd 主义征服-cloudflare
git init            # 如果是新仓库
git add .
git commit -m "Deploy 主义征服"
git remote add origin <你的GitHub仓库地址>
git push -u origin main
```
> 也可以把本目录内容直接放在已有仓库的根目录或子目录。

### 2. 在 Cloudflare 创建 Pages 项目
1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com) → **Workers & Pages** → **Create** → **Pages**
2. 选择 **Connect to Git** → 授权并选中你的 GitHub 仓库
3. 构建配置：
   - **Framework preset**：无 / None
   - **Build command**：留空（或 `echo hi`）
   - **Build output directory**：`主义征服-cloudflare`（若仓库根目录就是这个文件夹）或 `.`（若内容已放在根目录）
4. 点击 **Save and Deploy**

约 1 分钟后会得到一个 `https://<项目名>.pages.dev` 地址，任何人打开即可游玩。

### 3.（可选）绑定自有域名
在 Pages 项目的 **Custom domains** 里添加你的域名并按提示加 DNS 记录即可。

## 启用完整多人联机（可选）

PvP 直连（WebRTC）已够两人对战。若你想要带房间 / 世界频道 / 好友的 WebSocket 多人：

1. 自己跑后端（仓库外另存的多人服务器）：
   ```bash
   cd multiplayer && npm install && node server.js
   ```
2. 在游戏联机大厅把服务器地址填成 `ws://你的服务器IP:3000`（或 `wss://你的域名`）。
3. 部署版里这段地址是手动填写的，所以**后端和前端可以分开托管**，互不影响。

## 技术说明

- 纯前端单文件实现，零外部依赖（图标用 data URI 内联）
- 支持现代浏览器（Chrome / Firefox / Safari / Edge），含移动端响应式
- PvP 联机使用 WebRTC + 复制粘贴连接码，无需中央服务器
