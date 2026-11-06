---
title: "用 Hugo + PaperMod 搭一个能长期写的技术博客"
date: 2026-08-09T20:00:00+08:00
draft: false
description: "这套博客的结构、为什么用 Page Bundle 放图片、怎么写新笔记、以及改完怎么发布。"
categories: ["建站"]
tags: ["Hugo", "PaperMod", "GitHub Pages"]
cover:
  relative: true
  image: "cover.png"
  alt: "Hugo 博客搭建笔记封面"
  caption: "Markdown 写，Git 发布"
---

这套博客的定位很明确：**用 Markdown 写笔记，用 Git 管版本，用 GitHub Pages 免费托管**。不依赖数据库，不需要服务器，改完推送就上线。

## 目录结构

```
tech-blog/
├── hugo.toml                  # 全站配置：标题、菜单、开关
├── content/
│   └── posts/
│       └── my-note/
│           ├── index.md       # 文章正文
│           ├── cover.png      # 封面（front matter 里引用）
│           └── xxx.png        # 正文插图
├── layouts/
│   ├── archives.html          # 自定义的年月归档页
│   └── _partials/
│       └── extend_post_content.html
├── assets/css/extended/
│   └── custom.css             # 自定义样式
├── static/                    # 原样拷贝到站点根目录的文件
├── themes/PaperMod/           # 主题（已内置，不用装 submodule）
└── .github/workflows/deploy.yml  # 推送后自动构建发布
```

关键约定：**每篇文章是一个文件夹（Page Bundle）**，`index.md` 是正文，图片和它放在一起。这样文章和素材永远不分离，复制整个文件夹就能搬走一篇笔记。

## 写一篇新笔记

```bash
hugo new content posts/我的新笔记/index.md
```

这会按 `archetypes/posts.md` 模板生成文件，带好 title、date、categories、tags、cover 等字段。

手写也行，最小可用的 front matter：

```yaml
---
title: "标题"
date: 2026-08-09T20:00:00+08:00
draft: false
description: "一句话摘要，会显示在列表和搜索结果里"
categories: ["Linux"]
tags: ["Linux", "命令行"]
cover:
  relative: true
  image: "cover.png"
---
```

## 三种插图方式

| 方式 | 写法 | 适合 |
| --- | --- | --- |
| Markdown 标准语法 | `![说明](图片名.png)` | 日常最常用，自动居中、加载懒加载 |
| figure 短代码 | `{{</* figure src="图片名.png" caption="图注" */>}}` | 需要图注时 |
| 直接写 HTML | `<img src="图片名.png" class="left" alt="说明">` | 需要特殊对齐/尺寸时 |

图片放在和 `index.md` 同一个目录，文件名用英文/数字（比如 `step-1.png`），避免空格和中文带来的路径问题。

## 归档怎么工作

`layouts/archives.html` 先按年份分组，再在每一年里按月份分组，页首还有年份快速跳转。分组依据是 front matter 里的 `date`，所以**日期别写错**——写错的文章会跑到别的月份去。

文章网址也带年月：`/posts/2026/08/hugo-blog/`。这是 `hugo.toml` 里 `[permalinks]` 配的，好处是归档、网址、文件排序三者对得上。

## 发布流程

```
本地写 → git add / commit / push → GitHub Actions 自动构建 → Pages 更新
```

工作流做的事很固定：装 Hugo → `hugo --minify` → 把 `public/` 传给 Pages。整个过程 1~2 分钟，不需要你在本地装 Hugo。

## 几个容易踩的坑

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 图片 404 | 图片没和 `index.md` 放同目录，或文件名大小写不一致 | 统一小写、无空格 |
| 新文章不出现 | `draft: true` 没改，或日期在未来 | 改 `draft: false`，检查 date |
| 首页空白 | `mainSections` 和实际目录不一致 | 确认是 `posts` |
| 样式没生效 | 缓存 | 强制刷新，或确认 `custom.css` 在 `assets/css/extended/` |
| Actions 报错 | Hugo 版本与主题不匹配 | 工作流里锁定 `0.158.0` |

## 为什么不用现成的博客平台

- 内容在自己手里：全是 Markdown 文件，随时能搬走。
- 版本可回溯：每次改动都有 commit，写错了能退回。
- 零成本：GitHub Pages 免费，构建也在 GitHub 的机器上跑。
- 可定制：想加什么页面就加模板，不用担心平台限制。

代价是要理解一点点 Hugo 的目录约定——也就是上面这些。

## 小结

- 一篇文章 = 一个文件夹 = `index.md` + 图片。
- 归档和网址都按月，靠 front matter 的 `date` 决定。
- 发布就是 push，其余交给 Actions。
