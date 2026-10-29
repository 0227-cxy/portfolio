# 个人展示网站 技术设计文档（TECH_DESIGN.md）

> 版本：v5.0 ｜ 作者：陈鑫宇 ｜ 更新：2026-10-29
> 对应实现方向：v9「护照 · 杂志手账风」（Vite + Vue3 工程版）

## 一、技术选型

| 层 | 选型 | 理由 |
|---|---|---|
| 前端框架 | Vue 3（Composition API + `<script setup>`） | 课程要求 Vue / React 二选一；响应式数据驱动，模板自动更新 DOM |
| 构建工具 | Vite 5 | 开发服务器热更新快、构建产物小、零配置开箱即用 |
| 样式 | 原生 CSS（全局 `src/style.css`），`:root` 变量管理配色；SVG data URI 作背景织纹 | 无需预处理器，风格细腻可控 |
| 存储 | localStorage（键 `site_guestbook` / `site_likes` / `site_liked`） | 留言与点赞本地持久化，兼容旧版键名 |
| 素材 | `public/img/cover-*.jpg`（3 张项目封面） | 不依赖外链，离线可用 |
| 部署 | GitHub Pages / Vercel / Cloudflare Pages | 免费静态托管 |

## 二、文件结构

```
portfolio/
├── index.html            # Vite 入口 HTML（挂载点 + 字体引入）
├── package.json          # 依赖与脚本配置
├── vite.config.js         # Vite 配置（端口 5173，自动打开浏览器）
├── public/
│   └── img/
│       ├── cover-translate.jpg   # 项目 1：划词翻译·生词本 封面
│       ├── cover-heritage.jpg   # 项目 2：薪传坊·非遗平台 封面
│       └── cover-book.jpg       # 项目 3：图书漂流平台 封面
├── src/
│   ├── main.js            # Vue 应用入口（createApp(App).mount('#app')）
│   ├── App.vue            # 根组件（<template> + <script setup> + 内联样式）
│   └── style.css         # 全局样式（CSS 变量 + 全部组件样式）
├── PRD.md
├── TECH_DESIGN.md
└── AGENTS.md
```

## 三、数据模型

### 3.1 内置数据（`App.vue` 的 `<script setup>` 顶部常量）

```js
const timeline = [{ year, text }]          // 时间线 3 节点
const skillClusters = [{ title, items: [{ name, pct }] }]  // 技能 4 组 13 项
const honors = [{ level, levelLabel, title, award }]        // 荣誉 10 条
const projects = [{ img, title, type, overlay, caption, link, linkText }]  // 项目 3 个
const blogs = [{ title, date, tags: [], summary, likes }]    // 博客 3 篇
```

### 3.2 本地存储

| 键 | 结构 | 用途 |
|---|---|---|
| `site_guestbook` | `[{ name, content, date }]` | 留言列表，页面加载时读取渲染 |
| `site_likes` | `{ uid: count }` | 点赞增量（uid = "p" + 博客标题），叠加内置基数显示 |
| `site_liked` | `[uid, ...]` | 已赞去重数组，防止重复点赞 |

- 键名与旧版页面保持一致，历史留言数据无缝兼容；
- Vue 模板 `{{ }}` 默认自动转义，无需手动 `escHtml()`。

### 3.3 响应式状态

| 变量 | 类型 | 用途 |
|---|---|---|
| `blogTag` | `ref('all')` | 当前博客筛选标签 |
| `filteredBlogs` | `computed` | 根据 `blogTag` 计算过滤后的博客列表 |
| `messages` | `ref([...])` | 留言列表（从 localStorage 初始化） |
| `newMsg` | `ref({ name: '', content: '' })` | 新留言表单绑定 |

## 四、关键实现

### 4.1 复制邮箱（含降级）

```js
function copyEmail() {
  const addr = '706763331@qq.com'
  if (navigator.clipboard) navigator.clipboard.writeText(addr).then(() => alert('邮箱已复制！'))
}
```

### 4.2 博客筛选

- `blogTag` 用 `ref()` 管理当前标签；
- `filteredBlogs` 用 `computed()` 自动计算过滤结果；
- 模板中 `v-for="b in filteredBlogs"` 自动渲染，无需手动操作 DOM。

### 4.3 留言板

- `messages` 用 `ref()` 初始化时从 localStorage 读取；
- `addMessage()`：校验非空 → push 到 `messages` → 写入 localStorage → 清空表单；
- 模板 `v-for="(m,i) in messages"` 自动渲染新留言。

### 4.4 博客点赞

- `uid = "p" + 博客标题`，`likedSet`（Set）去重防刷，`siteLikes`（对象）存增量；
- 显示数 = 内置基数 `b.likes` + 增量；
- `likeBlog(b)`：检查是否已赞 → 更新 Set 和对象 → 写入 localStorage；
- 模板根据 `isLiked(b)` 自动切换按钮样式（已赞变红）。

### 4.5 视觉实现要点

| 模块 | 技术实现 |
|---|---|
| 背景织纹 | `body` 上 40×40 SVG data URI 斜纹（fill-opacity 0.1），纯 CSS 无图片 |
| 名片胶带 | `.tape` 绝对定位半透明块 + 虚线侧边 + 微旋 |
| INFO 角标 | `.info-block::before` 内容 "INFO"，红底白字，绝对定位右上角 |
| 便签时间线 | `.note-card` 黄/浅蓝底色交替 + `transform: rotate(±1.5deg)`，hover 放大转正 |
| 票根荣誉 | `.ticket` 左边条按档着色 + `.ticket-level` 彩色圆角徽章 + `min-height + margin-top:auto` 实现奖项行沉底对齐 |
| 拍立得项目 | `.project-polaroid` 奇偶 `rotate(±2deg)` + `align-self` 交错，白边 + 底部手写标题 + `.p-type` 红色贴纸类型徽章 + `.p-overlay` 右上角技术标签 + `.p-link` 黑色胶囊中文按钮 |
| 剪报博客 | `.clipping` 灰底 + `::after` 外层细描边 + 黑白小标签 |
| 响应式 | `@media (max-width: 900px)`：单列布局、名片置顶、拍立得取消旋转居中 |

## 五、部署方案

采用静态托管（三选一，均免费）：

1. **GitHub Pages**：
   - 本地 `npm run build` 生成 `dist/` 目录；
   - 将 `dist/` 内容推到 `gh-pages` 分支；
   - 或配置 GitHub Actions 自动构建部署；
   - 访问地址：`https://0227-cxy.github.io/portfolio/`
2. **Vercel**：注册选 Hobby（免费），Import 仓库，框架预设选 Vite，自动识别并部署；
3. **Cloudflare Pages**：从 GitHub 导入仓库，生产分支 `main`，框架预设选 Vite，保存并部署。

## 六、测试与验证

- 本地 `npm run dev` 启动开发服务器，浏览器打开 http://localhost:5173；
- 无头浏览器渲染双端截图核对：1280 宽桌面 / 375 宽移动端；
- 手测清单：
  - 邮箱点击复制；
  - 博客 7 标签筛选（选中按钮黑底高亮）；
  - 博客点赞（+1、已赞变红、刷新后计数保留）；
  - 留言提交后刷新保留；
  - 三个项目链接可点开；
  - 375px 无横向滚动；
  - F12 Console 无报错。
