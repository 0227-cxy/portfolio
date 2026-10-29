# 个人展示网站 技术设计文档（TECH_DESIGN.md）

> 版本：v4.0 ｜ 作者：陈鑫宇 ｜ 更新：2026-10-27
> 对应实现方向：v9「护照 · 杂志手账风」（单文件自包含版）

## 一、技术选型

| 层 | 选型 | 理由 |
|---|---|---|
| 页面结构 | 原生 HTML5 单文件 `index.html`（内联 `<style>` 与 `<script>`） | 零构建、零依赖，双击即可预览，任何静态托管直接部署 |
| 样式 | 原生 CSS，`:root` 变量管理配色；SVG data URI 作背景织纹 | 无外部 CSS 请求，风格统一 |
| 交互 | 原生 JavaScript（函数式，非模块化） | 体积小、逻辑直观，符合课程要求 |
| 存储 | localStorage（键 `site_guestbook` / `site_likes` / `site_liked`） | 留言与点赞本地持久化，兼容旧版键名，旧留言不丢 |
| 素材 | 本地 `assets/img/cover-*.jpg`（3 张项目封面） | 不依赖外链，离线可用 |

## 二、文件结构

```
portfolio/
├── index.html            # 唯一页面：结构 + 内联样式 + 内联脚本
├── PRD.md                # 产品需求文档（本文档对应当前实现）
├── TECH_DESIGN.md        # 技术设计文档
├── AGENTS.md             # AI 代理指令
└── assets/
    └── img/
        ├── cover-translate.jpg   # 项目 1：划词翻译·生词本 封面（摊开词典 + 红蓝标注 + 钢笔/放大镜/地球仪，水彩手账风）
        ├── cover-heritage.jpg    # 项目 2：薪传坊·非遗平台 封面（青花瓷 + 剪纸福字 + 刺绣绣绷 + 京剧脸谱/皮影，国风水彩拼贴）
        └── cover-book.jpg        # 项目 3：图书漂流平台 封面（纸折小船载书 + 睡莲 + 垂柳 + 暖阳，清新水彩）
```

## 三、数据模型

### 3.1 内置数据（`<script>` 顶部常量）

```js
const blogs = [
  { title, date, tags: [], summary, likes }
];
```

- `blogs`：3 篇内置博客（标题 / 日期 / 标签数组 / 摘要 / 静态点赞数）。

### 3.2 本地存储

| 键 | 结构 | 用途 |
|---|---|---|
| `site_guestbook` | `[{ name, content, date }]` | 留言列表，页面加载时读取渲染 |
| `site_likes` | `{ uid: count }` | 点赞增量（uid = "p" + 博客标题），叠加内置基数显示 |
| `site_liked` | `[uid, ...]` | 已赞去重数组，防止重复点赞 |

- 键名与旧版页面保持一致，历史留言数据无缝兼容；
- 存储内容在渲染前一律经 `escHtml()` 转义，防 XSS。

### 3.3 页面静态数据（直接写在 HTML）

- 名片：姓名、专业、身份、生日 2005.02、邮箱、GitHub；
- 统计：13 技能 / 3 项目 / 10 荣誉；
- 时间线：3 节点；技能：4 组 13 项（含百分比）；
- 荣誉：10 条（level 分类 + 赛事名 + 奖项）；
- 项目：3 个（封面路径、技术标签、链接）。

## 四、关键实现

### 4.1 复制邮箱（含降级）

```js
function copyEmail() {
  // 优先 navigator.clipboard；file:// 或旧浏览器失败时
  // 回退 textarea + document.execCommand("copy")
}
```

### 4.2 博客筛选

- `renderBlogs(tag)`：按标签渲染博客剪报卡片（`tag === "all"` 显示全部）；
- `filterBlog(tag)`：渲染并给对应按钮加 `.active`（黑底白字），点击有明确反馈。

### 4.3 留言板

- `loadMsgs()`：读取 `site_guestbook` 渲染列表（字段兼容 `name/content/date` 与旧版 `name/msg/time`），内容转义；
- `addMessage()`：校验非空 → 写入 localStorage → 清空表单 → 重新渲染。

### 4.4 博客点赞

- `uid = "p" + 博客标题`，`site_liked`（数组）去重防刷，`site_likes`（`{uid:count}`）存增量；
- 显示数 = 内置基数 `blogs[].likes` + 增量，点击后按钮变红"♥ 已赞"不可再点；
- 事件委托绑定在 `#blogList`，标签筛选重渲染后点赞仍有效。

### 4.5 视觉实现要点

| 模块 | 技术实现 |
|---|---|
| 背景织纹 | `body` 上 40×40 SVG data URI 斜纹（fill-opacity 0.1），纯 CSS 无图片 |
| 名片胶带 | `.tape` 绝对定位半透明块 + 虚线侧边 + 微旋 |
| INFO 角标 | `.info-block::before` 内容 "INFO"，红底白字，绝对定位右上角 |
| 便签时间线 | `.note-card` 黄/浅蓝底色交替 + `transform: rotate(±1.5deg)`，hover 放大转正 |
| 票根荣誉 | `.ticket` 左边条按档着色 + `.ticket-level` 彩色圆角徽章 + `min-height + margin-top:auto` 实现奖项行沉底对齐 |
| 拍立得项目 | `.project-polaroid` 奇偶 `rotate(±2deg)` + `align-self` 交错，白边 + 底部手写标题 + `.p-type` 红色贴纸类型徽章（absolute 于照片左上角：浏览器插件 / 在线网页）+ `.p-overlay` 右上角技术标签 + `.p-link` 黑色胶囊中文按钮（查看源码/在线体验，hover 箭头右移） |
| 剪报博客 | `.clipping` 灰底 + `::after` 外层细描边 + 黑白小标签 |
| 响应式 | `@media (max-width: 900px)`：单列布局、名片置顶、拍立得取消旋转居中 |

## 五、部署方案

采用静态托管（三选一，均免费）：

1. **GitHub Pages**：仓库 `0227-cxy/portfolio` → Settings → Pages → 分支 `main` / 根目录，`https://0227-cxy.github.io/portfolio/`；
2. **Vercel**：注册选 Hobby（免费），Import 仓库，框架预设自动识别为静态站；
3. **Cloudflare Pages**：从 GitHub 导入仓库，生产分支 `main`，框架预设选"无"，保存并部署。

## 六、测试与验证

- 本地直接双击 `index.html` 即可预览（无构建步骤）；
- 无头浏览器渲染双端截图核对：1280 宽桌面 / 375 宽移动端；
- 手测清单：邮箱复制、博客 7 标签筛选（含 active 高亮）、留言提交后刷新保留、外链可点开、375px 无横向滚动、F12 无报错。
