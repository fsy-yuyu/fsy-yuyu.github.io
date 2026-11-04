# 技术笔记（Hugo + PaperMod）

一个可以直接用的技术博客项目：**按分类整理笔记、按年月自动归档、支持站内搜索、图片随文章一起管理、推送到 GitHub 自动构建发布**。

主题、配置、示例文章、部署工作流都已配好，你只需要改两处（网站标题、作者名）就能上线。

---

## 30 秒了解

| 你想要的 | 在这个项目里怎么实现 |
| --- | --- |
| 按方向整理笔记 | `categories`（分类）+ `tags`（标签），文章顶部一行搞定 |
| 按年月归档 | 打开 `/archives/`，自动按「年 → 月」分组，带篇数统计和年份跳转 |
| 在文章里插图 | 图片和 `index.md` 放同一个文件夹，`![](图片名.png)` 即可 |
| 写完就发布 | 推送到 GitHub，Actions 自动构建，1~2 分钟上线 |
| 不改代码也能写 | 在 GitHub 网页上新建 `content/posts/xxx/index.md` 就行 |

---

## 本地预览（可选，但强烈建议）

需要先装 Hugo（版本 **0.158.0 或更高**）。

```bash
# macOS
brew install hugo

# Windows（任选一个）
winget install Hugo.Hugo.Extended
scoop install hugo-extended

# Linux（Debian/Ubuntu）
sudo apt install hugo
```

然后：

```bash
cd tech-blog
hugo server -D
```

浏览器打开 <http://localhost:1313>，改文件保存后页面会自动刷新。

> `-D` 表示连草稿（`draft: true`）一起显示。

---

## 部署到 GitHub Pages

### 路线 A：全程网页操作（不想装任何软件就用这个）

1. GitHub 右上角 **+** → **New repository**
2. 仓库名填 `你的用户名.github.io`（比如 `zhangsan.github.io`），选 **Public**，**不要**勾 "Add a README file"
3. 进入空仓库，点 **Add file → Upload files**
4. 把 `tech-blog` 文件夹里的**所有内容**（不是文件夹本身）拖进去
   - Windows 先在资源管理器里打开「显示隐藏的项目」，否则 `.github` 和 `.gitignore` 会漏掉
5. 底部点 **Commit changes**
6. 仓库 **Settings → Pages → Build and deployment → Source** 选 **GitHub Actions**
7. 打开 **Actions** 标签页，等 "构建并部署博客" 变成绿色对勾（约 1~2 分钟）
8. 访问 `https://你的用户名.github.io`

### 路线 B：GitHub Desktop（不敲命令，但要装软件）

1. 安装 [GitHub Desktop](https://desktop.github.com/)，登录账号
2. **File → New repository**，名字 `你的用户名.github.io`，本地路径选一个空文件夹
3. 把 `tech-blog` 里的所有文件复制进这个文件夹
4. 左下角填一句提交信息（如 `init blog`），点 **Commit to main**
5. 右上角 **Publish repository**，**取消勾选** "Keep this code private"
6. 之后同路线 A 的第 6~8 步

### 路线 C：命令行

```bash
cd tech-blog
git init
git add .
git commit -m "init blog"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
git push -u origin main
```

---

## 日常更新只要三步

```bash
# 1. 写：新建或修改 content/posts 下的文件
hugo new content posts/我的新笔记/index.md

# 2. 看：本地预览确认效果
hugo server -D

# 3. 发：提交并推送
git add .
git commit -m "新增：我的新笔记"
git push
```

推送后不用管，GitHub 会自动构建。也可以在网页上直接改文件，改完点 Commit，同样会自动发布。

**完整的逐步操作（含网页版每一步点哪里、图片怎么放、出错了怎么查）见 [GUIDE.md](GUIDE.md)。**

---

## 目录结构

```
tech-blog/
├── hugo.toml                     # 全站配置：标题、作者、菜单、开关都在这里
├── content/
│   ├── _index 相关页面
│   ├── archives.md               # /archives/ 归档页
│   ├── search.md                 # /search/ 搜索页
│   ├── about.md                  # /about/ 关于页
│   ├── categories/_index.md      # /categories/ 分类页标题
│   ├── tags/_index.md            # /tags/ 标签页标题
│   └── posts/                    # 所有笔记（一篇文章一个文件夹）
│       └── linux-file-permissions/
│           ├── index.md          # 正文
│           ├── cover.png         # 封面
│           └── permission-bits.png
├── layouts/
│   ├── archives.html             # 年月归档模板（改归档样式改这里）
│   └── _partials/
│       └── extend_post_content.html   # 文章末尾的分类徽章 + 永久链接
├── archetypes/
│   └── posts.md                  # 新建文章的模板（改这里影响以后所有新文章）
├── assets/css/extended/
│   └── custom.css                # 自定义样式，不改主题也能调外观
├── static/                       # 图标等原样拷贝的文件
├── themes/PaperMod/              # 主题，已内置，无需 submodule
├── .github/workflows/deploy.yml  # 推送后自动构建发布的配置
└── GUIDE.md                      # 详细使用手册
```

---

## 想改哪里

| 想改的内容 | 改哪个文件 | 改哪一行 |
| --- | --- | --- |
| 网站标题 | `hugo.toml` | `title = "技术笔记"` |
| 作者名（显示在文章头部） | `hugo.toml` | `author = "你的名字"` |
| 首页欢迎语 | `hugo.toml` | `[params.homeInfoParams]` 下的 `Title` / `Content` |
| 顶部菜单 | `hugo.toml` | `[[menus.main]]` 段落 |
| GitHub 等社交链接 | `hugo.toml` | `[[params.socialIcons]]` |
| 关于页内容 | `content/about.md` | 正文和「关于我」段落 |
| 归档页外观 | `layouts/archives.html` + `assets/css/extended/custom.css` | — |
| 新文章模板 | `archetypes/posts.md` | — |
| 配色/圆角/字体 | `assets/css/extended/custom.css` | — |

---

## 三张「示例笔记」可以直接删

`content/posts/` 下的 10 篇是演示用的真实笔记（Linux、Git、Docker、Python、HTTP、MySQL、JS、Shell、Hugo、VS Code 各一篇），它们演示了：

- front matter 怎么写
- 分类和标签怎么填
- 封面图怎么引用
- 正文里三种插图方式
- 表格、代码块、提示块怎么排版

不需要的话，删掉对应文件夹即可，归档页和首页会自动更新。

---

## 常见问题（详细排查见 GUIDE.md 第 9 节）

| 现象 | 先检查 |
| --- | --- |
| 网站 404 / 打不开 | Settings → Pages 的 Source 是否选了 GitHub Actions；Actions 是否成功 |
| 新文章没出现 | `draft` 是不是 `true`；`date` 是不是未来时间 |
| 图片不显示 | 图片是否和 `index.md` 在同一文件夹；文件名大小写是否一致 |
| 样式不对 | 强刷新（Ctrl+F5）；`assets/css/extended/custom.css` 位置是否正确 |
| Actions 报错 | 看日志里 `hugo` 那一步的红字；Hugo 版本是否被改动 |

---

## 技术栈

- 静态站点生成：Hugo 0.158.0（锁定在工作流里）
- 主题：[PaperMod](https://github.com/adityatelange/hugo-PaperMod)（MIT，已内置）
- 托管：GitHub Pages（免费，含 HTTPS）
- 搜索：Fuse.js 客户端搜索，无需后端

许可：主题遵循 PaperMod 的 MIT 协议；你的文章内容版权归你自己。
