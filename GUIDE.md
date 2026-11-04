# 使用手册（详细版）

这份手册按「第一次上线 → 日常更新 → 写文章 → 插图 → 改站点信息 → 归档规则 → 域名 → 排错」的顺序写，每一步都可以照着做。

目录：

1. [开始前的准备](#1-开始前的准备)
2. [第一次上线（约 30 分钟）](#2-第一次上线约-30-分钟)
3. [日常更新的三种方式](#3-日常更新的三种方式)
4. [写一篇新笔记（完整步骤）](#4-写一篇新笔记完整步骤)
5. [插图完全指南](#5-插图完全指南)
6. [改站点信息（标题、作者、菜单）](#6-改站点信息标题作者菜单)
7. [归档规则与网址](#7-归档规则与网址)
8. [绑定自己的域名（可选）](#8-绑定自己的域名可选)
9. [出问题了怎么查](#9-出问题了怎么查)
10. [备份、迁移与升级](#10-备份迁移与升级)

---

## 1. 开始前的准备

| 需要 | 说明 |
| --- | --- |
| GitHub 账号 | 免费注册即可。免费账号的 Pages 只支持 **公开仓库**（Public） |
| 仓库 | 建议命名 `你的用户名.github.io`，这样网址是 `https://你的用户名.github.io`，没有子路径 |
| 本地 Hugo（可选） | 版本 **≥ 0.158.0**。不装也能写文章，只是没法本地预览 |
| 一个编辑器（可选） | VS Code / Typora / Obsidian 都能编辑 Markdown |

先确认 Hugo 版本：

```bash
hugo version
# 期望输出类似：hugo v0.158.0-... linux/amd64
```

版本低于 0.158.0 会报模板错误，请到 <https://github.com/gohugoio/hugo/releases> 下载新版。

---

## 2. 第一次上线（约 30 分钟）

下面按「全程网页操作」写，不需要安装任何软件。装 GitHub Desktop 或会用命令行的，见 2.4。

### 2.1 新建仓库

1. 登录 GitHub，点右上角 **+** → **New repository**。
2. **Repository name** 填 `你的用户名.github.io`（例如 `zhangsan.github.io`）。
   - 也可以填别的名字（如 `blog`），那样网址会变成 `https://zhangsan.github.io/blog/`，同样能用。
3. 可见性选 **Public**。
4. **不要**勾选 "Add a README file"、".gitignore"、license（勾了会产生一次额外提交，后面容易冲突）。
5. 点 **Create repository**，得到一个空仓库页面。

### 2.2 上传全部文件

1. 在空仓库页面点 **Add file** → **Upload files**。
2. 解压你下载的 `tech-blog.zip`，进入 `tech-blog` 文件夹，**全选里面的所有内容**（不是选中 tech-blog 这个文件夹本身），拖进网页的虚线框。
   - Windows：先在资源管理器顶部「查看」里勾上 **隐藏的项目**，否则 `.github`、`.gitignore` 这两个隐藏项会漏传。`.github` 里有部署配置，漏了就永远不会自动构建。
   - 如果拖文件夹不成功，就逐层上传：先传根目录文件，再进 `content`、`layouts`、`themes` 等目录分别上传。
3. 等待上传完成（文件较多，可能需要 1~2 分钟；`themes/PaperMod` 里有约 90 个文件，耐心等一下）。
4. 页面底部 **Commit changes** 的说明填 `初始化博客`，点 **Commit changes**。

> 上传完成后，仓库根目录应该能看到 `hugo.toml`、`content`、`layouts`、`themes`、`assets`、`archetypes`、`static`、`.github`、`README.md`、`GUIDE.md`。

### 2.3 打开 GitHub Pages

1. 点仓库右上角的 **Settings**。
2. 左侧菜单找到 **Pages**。
3. **Build and deployment** 下的 **Source** 选择 **GitHub Actions**（不要选 "Deploy from a branch"）。
4. 不用填分支，保存后自动生效。

### 2.4 等待构建并访问

1. 点仓库顶部的 **Actions** 标签。
2. 应该看到一个名为「构建并部署博客」的运行记录：
   - 黄色转圈 = 正在跑
   - 绿色对勾 = 成功
   - 红色叉 = 失败，点进去看哪一步红了（见第 9 节）
3. 首次大约 1~2 分钟。成功后回到 **Settings → Pages**，顶部会显示 `Your site is live at https://...`。
4. 打开那个网址，应该能看到首页、6 个菜单项和 10 篇示例笔记。

### 2.5 首次上线检查清单

- [ ] 首页能打开，封面图正常显示
- [ ] 点「归档」能看到 2026 / 2025 两个年份，下面按月分组
- [ ] 点任意一篇笔记，正文图片、代码块、目录都正常
- [ ] 点「分类」能看到 9 个分类
- [ ] 点「搜索」，输入 `docker` 能出结果
- [ ] 浏览器地址栏是 `https://`（Pages 默认开 HTTPS，可能需要几十秒生效）

### 2.6 如果你更习惯客户端或命令行

**GitHub Desktop**

1. 安装并登录 <https://desktop.github.com/>。
2. **File → New repository**，Name 填 `你的用户名.github.io`，Local path 选一个空文件夹，点 **Create repository**。
3. 把 `tech-blog` 里的所有文件复制到这个文件夹（含隐藏文件）。
4. 左下角 Summary 填 `init blog`，点 **Commit to main**。
5. 右上角 **Publish repository**，**取消勾选** "Keep this code private"，点 **Publish**。
6. 回到网页做 2.3 的 Pages 设置。

**命令行**

```bash
cd tech-blog
git init
git add .
git commit -m "init blog"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
git push -u origin main
```

推送若因网络超时失败，重试几次或用 GitHub Desktop，实在不行回到网页上传（2.2）。

---

## 3. 日常更新的三种方式

三种方式最终都是「**改文件 → 提交 → Actions 自动构建**」，选一种顺手的即可。

### 3.1 方式一：GitHub 网页直接改（最快，适合改错别字）

**改已有文章**

1. 打开仓库，进入 `content/posts/文章文件夹/index.md`。
2. 右上角铅笔图标 **Edit this file**。
3. 改内容。
4. 页面下方 **Commit changes**，说明填一句「修正 xx 错别字」，点 **Commit changes**。
5. 等 1~2 分钟，Actions 跑完，刷新网站即可看到。

**新建文章**（不用装 Hugo）

1. 仓库页点 **Add file** → **Create new file**。
2. 文件名框输入 `content/posts/我的新笔记/index.md`（输入 `/` 会自动建目录，注意最后一定是 `index.md`）。
3. 把第 4 节给的 front matter 模板粘进去，写正文。
4. 点 **Commit changes**。

**上传图片**

1. 进入文章所在文件夹（例如 `content/posts/我的新笔记/`）。
2. 点 **Add file** → **Upload files**，把图片拖进去。
3. **Commit changes**。图片文件名用英文小写（如 `step-1.png`）。

### 3.2 方式二：本地 Hugo（推荐长期使用，能预览）

```bash
# 第一次
cd tech-blog
hugo server -D
# 浏览器打开 http://localhost:1313 ，改文件会自动刷新

# 改完之后
git add .
git commit -m "新增：xxx 笔记"
git push
```

`hugo server` 只影响本地预览，不会生成文件；`Ctrl+C` 停止。

### 3.3 方式三：GitHub Desktop

1. 打开 GitHub Desktop，选中这个仓库。
2. 用编辑器改文件、加图片。
3. 左侧 **Changes** 里能看到改动列表。
4. 左下角 Summary 填一句说明，点 **Commit to main**。
5. 顶部 **Push origin** 推送。
6. 等 Actions 跑完。

### 3.4 更新流程的共同点

```
改文件 → commit → push → GitHub Actions 构建（1~2 分钟）→ 网站更新
```

只要看到 Actions 变绿，网站就已经更新；如果打开网页还是旧的，按 `Ctrl+F5` 强制刷新（浏览器缓存）。

---

## 4. 写一篇新笔记（完整步骤）

### 4.1 命名规范

一篇文章 = `content/posts/` 下的**一个文件夹**，文件夹名就是网址的一部分，请用**英文小写 + 短横线**：

```
content/posts/redis-cache-patterns/index.md
             └────── 文件夹名 = 网址里的 slug ──────┘
```

对应网址：`https://你的域名/posts/2026/11/redis-cache-patterns/`

> 文件夹名不要用中文、空格、大写字母。网址里的年月来自文章 front matter 的 `date`，见第 7 节。

### 4.2 用命令创建（已装 Hugo）

```bash
hugo new content posts/redis-cache-patterns/index.md
```

会自动按 `archetypes/posts.md` 生成文件，带好标题、日期、分类、标签、封面字段和写作提示。

### 4.3 手动创建

在 `content/posts/` 下新建文件夹 `redis-cache-patterns`，里面新建 `index.md`，内容如下（**最上面两行 `---` 之间是 front matter，必须保留**）：

```yaml
---
title: "Redis 缓存穿透、击穿、雪崩的区别与处理"
date: 2026-11-03T21:00:00+08:00
draft: false
description: "一句话说明这篇笔记解决什么问题，会显示在列表和搜索结果里。"
categories: ["数据库"]
tags: ["Redis", "缓存", "性能优化"]
cover:
  relative: true
  image: "cover.png"
  alt: "封面图的替代文字"
  caption: "封面图下方的说明"
---

## 背景

正文从这里开始。标题请从「##」开始，不要用「#」，因为文章标题已经是一级标题了。
```

### 4.4 front matter 每个字段的作用

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `title` | 是 | 文章标题，显示在页面顶部、列表和浏览器标签 |
| `date` | 是 | 发布时间，**决定归档到哪年哪月，也决定网址里的年月** |
| `draft` | 是 | `true` = 草稿（网站上看不到、搜索搜不到）；`false` = 正式发布 |
| `description` | 建议 | 摘要，用于列表页、搜索结果、分享卡片 |
| `categories` | 建议 | 分类，**建议只填一个**，如 `["Linux"]`、`["数据库"]` |
| `tags` | 建议 | 标签，可以多个，用于按关键词聚合 |
| `cover.relative` | 用封面时必填 | 固定写 `true`，表示封面图在本文章文件夹内 |
| `cover.image` | 可选 | 封面图文件名，图要和 `index.md` 在同一文件夹 |
| `cover.alt` | 可选 | 图片替代文字，利于无障碍和 SEO |
| `cover.caption` | 可选 | 封面下方的小字说明 |
| `ShowToc` | 可选 | `false` 可关掉这一篇的目录 |
| `hideMeta` | 可选 | `true` 隐藏日期/阅读时间那一行 |
| `lastmod` | 可选 | 最后更新时间，填了会在页面上显示 |
| `aliases` | 可选 | 旧网址列表，换网址时保住旧链接（见 7.3） |

### 4.5 写正文的几条建议

- 标题层级只用 `##`、`###`，不要用 `#`。
- 命令放代码块并标语言（```bash），主题会自动高亮 + 加复制按钮。
- 对比信息用表格，比大段文字好读。
- 「现象 → 原因 → 处理」这种结构最适合技术笔记。
- 提示和警告用现成的块：

```html
<div class="tip">这是一条提示，绿色左边框。</div>
<div class="warn">这是一条警告，红色左边框。</div>
<div class="note">这是一条普通说明，灰色左边框。</div>
```

### 4.6 发布前自查

- [ ] `draft: false`
- [ ] `date` 不是未来时间
- [ ] `categories` 里填的是你想要的方向（写错会跑到别的分类下）
- [ ] 图片都在同一文件夹，文件名没空格、没中文
- [ ] 本地 `hugo server -D` 或网页预览确认排版
- [ ] 提交说明写清楚改了什么

---

## 5. 插图完全指南

### 5.1 图片放在哪

**和 `index.md` 放在同一个文件夹**，这是 Hugo 的 Page Bundle 约定：

```
content/posts/redis-cache-patterns/
├── index.md
├── cover.png          ← 封面
├── step-1.png         ← 正文插图
└── diagram.png
```

好处：文章和素材永远在一起，复制整个文件夹就能搬走一篇笔记，不会出现「图在别的目录、链接全断」。

**文件名规范**：全小写、英文或数字、用短横线连接，不要空格和中文。`step-1.png` ✅ / `第一步 截图.png` ❌

### 5.2 三种插图写法

**① 标准 Markdown（最常用）**

```markdown
![rwx 权限位示意](permission-bits.png)
```

效果：居中显示、圆角、懒加载（滚动到才加载），说明文字作为 alt。

**② figure 短代码（需要图注时）**

```markdown
{{< figure src="permission-bits.png" caption="三段三位：属主 / 属组 / 其他人" alt="权限位示意" >}}
```

效果：图片居中 + 下方一行灰色小字说明。示例文章里用的就是这种。

**③ 直接写 HTML（需要特殊对齐或尺寸时）**

```html
<!-- 左对齐 -->
<img src="diagram.png" class="left" alt="示意图">

<!-- 指定宽度 -->
<img src="diagram.png" width="480" alt="示意图">
```

`class="left"` 是 `custom.css` 里定义的，取消居中。

### 5.3 封面图

封面由 front matter 控制，不写 `cover` 就不显示封面（列表页会变成纯文字卡片）。

```yaml
cover:
  relative: true          # 必写：表示图片在本文章文件夹内
  image: "cover.png"
  alt: "封面替代文字"
  caption: "封面说明"
```

- **`relative: true` 一定要写**，否则分享到微信/X 时卡片里的图片地址会错（页面上看不出问题，但分享出去是裂图）。
- 建议尺寸 **1200 × 630**（约 1.91:1），文件小于 300 KB。
- 想做一批风格统一的封面，可以用任意工具导出同尺寸图，命名按文章区分。
- 想关掉某篇文章的封面：删掉 `cover` 段，或加 `cover.hidden: true`。

### 5.4 图片太大怎么办

| 问题 | 处理 |
| --- | --- |
| 截图 3~5 MB | 用 TinyPNG / Squoosh 压到 300 KB 以内，肉眼看不出差别 |
| 宽度超过 2000px | 缩到 1200~1600px，博客正文最宽也就 720px |
| 格式 | 截图用 PNG，照片用 JPG，图标/动图可用 WebP |
| 想放原图 | 点封面会打开原图（`linkFullImages = true`），正文图不额外放大 |

### 5.5 图片不显示？

按顺序查：

1. 图片是不是和 `index.md` **在同一个文件夹**（不是 `static/`，也不是上一级）。
2. 文件名大小写是否一致（`Cover.PNG` ≠ `cover.png`，Linux 服务器区分大小写）。
3. 文件名有没有空格或中文（浏览器会编码，容易出错）。
4. front matter 里写的是 `cover.png`，而文件夹里叫 `cover.jpg`。
5. GitHub 上确认图片真的上传成功（仓库里能看到这个文件）。
6. 强刷新浏览器 `Ctrl+F5`。

---

## 6. 改站点信息（标题、作者、菜单）

全部集中在 `hugo.toml`，改完提交即可。

### 6.1 必改的两处

```toml
title = "技术笔记"        # 网站标题，显示在左上角和浏览器标签
author = "你的名字"       # 显示在每篇文章的日期后面
```

### 6.2 首页欢迎语

```toml
[params.homeInfoParams]
  Title = "你好，我是「你的名字」"
  Content = """
这里是我的技术笔记本……
"""
```

`Content` 里可以写 Markdown（链接、列表都行）。

### 6.3 顶部菜单

```toml
[[menus.main]]
  name = "首页"
  url = "/"
  weight = 10
```

- `weight` 越小越靠左。
- 想加「收藏」页：新建 `content/favorites.md`（`title: "收藏"`、`url: "/favorites/"`），再加一段 menu 配置。
- 想删某一项：整段 `[[menus.main]]` 删掉。删「搜索」时别删 `content/search.md`，否则搜索页会 404。

### 6.4 社交图标

```toml
[[params.socialIcons]]
  name = "github"
  url = "https://github.com/你的用户名"
```

`name` 支持 `github`、`x`（Twitter）、`linkedin`、`mail`、`rss` 等（图标名见 PaperMod 的 Icons wiki），不需要的直接删段。

### 6.5 关于页

`content/about.md` 里的「关于我」部分改成你自己的介绍。文中的 `<!-- 注释 -->` 不会显示在网页上，可以放心写内部说明。

### 6.6 文章底部加「在 GitHub 上编辑」

`hugo.toml` 里把这段的注释去掉并改成你的仓库：

```toml
[params.editPost]
  URL = "https://github.com/你的用户名/你的用户名.github.io/edit/main/content/"
  Text = "在 GitHub 上编辑本文"
  AppendFilePath = true
```

### 6.7 改配色和外观

改 `assets/css/extended/custom.css`，它会在主题样式之后加载，优先级更高。常用变量在 `themes/PaperMod/assets/css/core/theme-vars.css` 里，例如：

```css
:root {
  --main-width: 760px;      /* 正文宽度 */
  --gap: 24px;              /* 页面留白 */
}
```

**不要直接改 `themes/PaperMod/` 里的文件**——升级主题时会被覆盖。要改主题行为，在 `layouts/` 下建同名模板覆盖。

---

## 7. 归档规则与网址

### 7.1 归档怎么分组

「归档」页（`/archives/`）的逻辑在 `layouts/archives.html`：

1. 先按 `date` 的**年份**分组，年份从新到旧；
2. 每年内部按**月份**分组，月份从新到旧；
3. 每条显示「日期（月-日）+ 分类 + 标签」；
4. 顶部有年份快速跳转按钮（带当年篇数）。

分组只认 front matter 的 `date`，所以**日期写错的文章会跑错月份**。

### 7.2 网址怎么来的

`hugo.toml` 里这一行决定网址：

```toml
[permalinks]
  posts = "/posts/:year/:month/:contentbasename/"
```

- `:year`、`:month` 来自 `date`
- `:contentbasename` 来自**文件夹名**（所以文件夹名要用英文）

例如文件夹 `redis-cache-patterns`、日期 `2026-11-03`，网址就是：

```
https://你的域名/posts/2026/11/redis-cache-patterns/
```

### 7.3 改了日期，网址会变

因为网址里带年月，把 `date` 从 2026-11 改到 2026-12，旧链接就会 404。想让旧链接继续可用，在 front matter 里加：

```yaml
aliases:
  - /posts/2026/11/redis-cache-patterns/
```

Hugo 会为旧网址生成一个跳转页，指向新地址。

### 7.4 不想让网址带年月

把 `hugo.toml` 里那行改成：

```toml
  posts = "/posts/:contentbasename/"
```

归档页不受影响（它只看 `date`），只是网址会变成 `/posts/redis-cache-patterns/`。

---

## 8. 绑定自己的域名（可选）

1. 在域名服务商处添加解析：
   - 顶级域名（`example.com`）：加 4 条 **A 记录**指向 `185.199.108.153`、`185.199.109.153`、`185.199.110.153`、`185.199.111.153`
   - 或加一条 **CNAME**，`www` 指向 `你的用户名.github.io`
2. 仓库 **Settings → Pages → Custom domain** 填入 `example.com`，点 **Save**。
3. 等 DNS 生效（几分钟到几小时），勾选 **Enforce HTTPS**。
4. 不需要改 `hugo.toml` 的 `baseURL`——工作流会从 Pages 读取真实地址自动注入。

> 如果网站出现「样式丢失、图片 404」，多半是 `baseURL` 与实际访问地址不一致。此时在 `hugo.toml` 里显式写 `baseURL = "https://example.com/"` 再推送一次即可。

---

## 9. 出问题了怎么查

### 9.1 先看 Actions

**Actions → 构建并部署博客 → 最近一次运行 → build 任务**，展开红色步骤看报错：

| 报错关键词 | 原因 | 处理 |
| --- | --- | --- |
| `Error: ... template ... not found` | 模板路径写错，或删了 `layouts` 里的文件 | 对照 README 目录结构还原 |
| `failed to load modules` / `hugo: not found` | Hugo 安装步骤失败（网络原因） | 重新运行一次（Actions 右上角 Re-run jobs） |
| `error building site: ...` | front matter 格式错误（YAML 缩进、缺 `---`） | 按第 4 节模板核对 |
| `deploy ... failed` | Pages 权限或环境问题 | Settings → Pages → Source 重新选 GitHub Actions 后重跑 |
| `Get "https://...": dial tcp ... timeout` | Actions 拉取 Hugo 超时 | 重跑；连续失败可把 `HUGO_VERSION` 换成已发布的邻近版本 |

### 9.2 网站能开但内容是旧的

- Actions 是否已经变绿（没绿就是没构建完）。
- 浏览器缓存：`Ctrl+F5`（Mac 是 `Cmd+Shift+R`）。
- 有没有推到正确的分支：工作流只监听 `main` / `master`，推到别的分支不会触发。

### 9.3 新文章不出现

| 检查项 | 说明 |
| --- | --- |
| `draft: false` | 草稿不会发布，也不会进搜索结果 |
| `date` 不是未来 | 未来的文章 Hugo 默认不生成（除非加 `--buildFuture`） |
| 文件路径 | 必须是 `content/posts/xxx/index.md`，不能是 `content/posts/index.md` |
| 文件夹层级 | 不要多套一层，如 `content/posts/2026/xxx/index.md`（会被当成子分类） |

### 9.4 页面 404

- 收藏夹里的旧链接？文章改过日期或文件夹名，网址就变了（见 7.3）。
- 手动输入网址时注意末尾的 `/`。
- 菜单里配的 `url` 要和实际页面路径一致（如 `/archives/` 对应 `content/archives.md` 里的 `url: "/archives/"`）。

### 9.5 本地 `hugo server` 起不来

| 现象 | 处理 |
| --- | --- |
| `port 1313 already in use` | 换端口：`hugo server -p 1414` |
| `Error: module ... not found` | 删掉 `go.mod`（本项目不需要 Go modules） |
| 模板报错 `can't evaluate field` | Hugo 版本太低，升级到 0.158.0+ |
| 页面没样式 | 用 `http://localhost:1313` 访问，不要直接双击 `public/index.html` |

---

## 10. 备份、迁移与升级

### 10.1 备份

你的全部内容都在 Git 里，三重保险：

1. GitHub 仓库本身就是备份；
2. 本地 `git clone` 一份；
3. 定期在 Releases 里打 tag，或下载仓库 zip 存网盘。

### 10.2 迁移到别的平台

所有内容都是标准 Markdown + 图片，`content/posts/` 整个目录可以直接搬到 Hexo、Obsidian、Notion 或其他 Hugo 站点。front matter 是标准 YAML。

### 10.3 升级主题

1. 到 <https://github.com/adityatelange/hugo-PaperMod/releases> 下载新版 zip。
2. 解压后替换 `themes/PaperMod/` 整个文件夹（`.github` 目录可以删掉，用不到）。
3. 保留你自己的 `layouts/`、`assets/css/extended/`、`hugo.toml`——这些不会被动。
4. 本地 `hugo server` 确认无报错，再推送。
5. 顺手看一眼新版要求的 Hugo 版本（`theme.toml` 里的 `min_version`），必要时同步改 `.github/workflows/deploy.yml` 里的 `HUGO_VERSION`。

### 10.4 升级 Hugo

改 `.github/workflows/deploy.yml` 里的：

```yaml
env:
  HUGO_VERSION: 0.158.0
```

改完推送，Actions 会用新版本重新构建。**建议一次只升一个版本并先在本地验证**，主题对 Hugo 版本比较敏感。

---

## 附：常用命令速查

```bash
# 本地预览（含草稿）
hugo server -D

# 换端口
hugo server -p 1414 -D

# 构建到 public/（一般不用手动执行，Actions 会做）
hugo --minify

# 新建文章
hugo new content posts/我的笔记/index.md

# 列出所有文章
hugo list all

# 查看草稿
hugo list drafts

# 检查未来日期的文章
hugo list future
```
