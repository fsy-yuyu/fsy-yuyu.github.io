---
title: "VS Code 效率配置：设置、快捷键与插件清单"
date: 2026-09-13T21:00:00+08:00
draft: false
description: "把常用设置写成 settings.json，记住十几个快捷键，插件只留真正每天用的。"
categories: ["工具"]
tags: ["VS Code", "效率", "插件"]
cover:
  relative: true
  image: "cover.png"
  alt: "VS Code 效率笔记封面"
  caption: "配置一次，长期受益"
---

插件装太多会拖慢启动，快捷键记太多会记混。下面这份是「每天真的会用」的最小集合。

## settings.json：值得同步的几项

按 `Ctrl/Cmd + Shift + P`，输入 `Open Settings (JSON)` 打开。

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.rulers": [100],
  "editor.minimap.enabled": false,
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs": "active",

  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.eol": "\n",

  "explorer.confirmDelete": false,
  "workbench.startupEditor": "none",
  "workbench.editor.enablePreview": false,

  "terminal.integrated.defaultProfile.osx": "zsh",
  "terminal.integrated.scrollback": 10000,

  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/.git": true
  }
}
```

逐项说明几个容易忽略的：

| 设置 | 作用 |
| --- | --- |
| `formatOnSave` | 保存即格式化，团队风格统一最省事 |
| `files.eol: "\n"` | 统一 LF，避免和 Linux 服务器、CI 打架 |
| `trimTrailingWhitespace` | 去掉行尾空格，diff 更干净 |
| `enablePreview: false` | 单击打开就是真打开，不会被下一个文件顶掉 |
| `search.exclude` | `node_modules` 不参与搜索，搜关键词瞬间变快 |

## 十几个够用的快捷键

| 操作 | Windows/Linux | macOS |
| --- | --- | --- |
| 命令面板 | `Ctrl+Shift+P` | `Cmd+Shift+P` |
| 快速打开文件 | `Ctrl+P` | `Cmd+P` |
| 全局搜索 | `Ctrl+Shift+F` | `Cmd+Shift+F` |
| 当前文件内替换 | `Ctrl+H` | `Cmd+Option+F` |
| 多光标（下一个相同词） | `Ctrl+D` | `Cmd+D` |
| 列选择 | `Shift+Alt+拖动` | `Shift+Option+拖动` |
| 移动当前行 | `Alt+↑/↓` | `Option+↑/↓` |
| 复制当前行 | `Shift+Alt+↓` | `Shift+Option+↓` |
| 跳到匹配括号 | `Ctrl+Shift+\` | `Cmd+Shift+\` |
| 折叠/展开代码块 | `Ctrl+Shift+[ / ]` | `Cmd+Option+[ / ]` |
| 切换终端 | `` Ctrl+` `` | `` Cmd+` `` |
| 分屏 | `Ctrl+\` | `Cmd+\` |
| 重命名符号 | `F2` | `F2` |
| 跳转到定义 | `F12` | `F12`（或 `Cmd+点击`） |
| 返回上一处 | `Alt+←` | `Ctrl+-` |

`Ctrl+D` 和 `Alt+↑/↓` 两个用熟了，编辑效率提升最明显。

## 插件清单（按用途分组）

| 用途 | 插件 | 说明 |
| --- | --- | --- |
| 语言 | 对应语言的官方扩展（Python / Go / Rust Analyzer…） | 必装，提供 LSP |
| 格式化 | Prettier | 前端/JSON/Markdown 通用 |
| 代码检查 | ESLint / ShellCheck | 边写边报错 |
| Git | GitLens | 行内 blame、历史对比 |
| 容器 | Docker | 侧边栏管理镜像容器 |
| 远程 | Remote - SSH / Dev Containers | 在服务器或容器里开发 |
| 写作 | Markdown All in One | 目录、快捷键、预览 |
| 拼写 | Code Spell Checker | 变量名拼错能发现 |
| 主题图标 | Material Icon Theme | 文件图标，找文件更快 |

装完在设置里关掉不用的：`extensions.ignoreRecommendations`、`telemetry.telemetryLevel: "off"`。

## 值得打开的三个功能

**1. 保存时自动整理 import**

```json
{
  "editor.codeActionsOnSave": {
    "source.organizeImports": "explicit",
    "source.fixAll.eslint": "explicit"
  }
}
```

**2. 远程开发**

`Remote - SSH` 连上服务器后，本地编辑、远端运行，不用再 `scp` 来回传文件。

```json
// ~/.ssh/config
Host dev
  HostName 192.168.1.10
  User deploy
  IdentityFile ~/.ssh/id_ed25519
```

**3. 多根工作区**

一个窗口管多个仓库，`.code-workspace` 文件可以提交到 Git：

```json
{
  "folders": [
    { "path": "./backend" },
    { "path": "./frontend" }
  ],
  "settings": { "editor.tabSize": 2 }
}
```

## 同步与备份

开启 Settings Sync（右下角账户图标 → Turn on Settings Sync），把设置、快捷键、插件、代码片段同步到 GitHub 账号。换电脑登录即恢复，比手动抄配置可靠。

## 小结

- 配置写进 `settings.json` 而不是靠记，换机器能一次恢复。
- 快捷键先练 `Ctrl+P`、`Ctrl+D`、`Alt+↑/↓` 三个。
- 插件按用途留最小集，装多了启动会明显变慢。
