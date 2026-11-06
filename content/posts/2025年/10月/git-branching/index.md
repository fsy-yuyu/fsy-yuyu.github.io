---
title: "Git 常用命令与分支模型：从三个区域说起"
date: 2025-10-19T21:10:00+08:00
draft: false
description: "工作区、暂存区、版本库三个区域决定了 git 命令该用哪条。整理日常命令、分支模型与撤销操作对照表。"
categories: ["Git"]
tags: ["Git", "版本控制", "协作"]
cover:
  relative: true
  image: "cover.png"
  alt: "Git 分支笔记封面"
  caption: "一条主线，多条特性分支"
---

Git 的命令看着多，其实都围绕三个区域转。把区域想清楚，撤销类命令就不会用错。

## 三个区域

```
工作区（Working Directory）   你正在编辑的文件
      │  git add
暂存区（Index / Stage）       下一次提交的内容清单
      │  git commit
版本库（Repository / .git）   已经固化的历史
      │  git push
远程仓库（Remote）
```

{{< figure src="branch-graph.png" caption="特性分支从 main 切出，合并回去" alt="分支示意图" >}}

所以：

- `git add` 是把工作区的改动放进暂存区；
- `git commit` 是把暂存区打包成一个版本；
- `git diff` 看工作区 vs 暂存区，`git diff --staged` 看暂存区 vs 上次提交。

## 日常命令

| 目的 | 命令 |
| --- | --- |
| 看状态（最常用） | `git status -sb` |
| 拉取并变基 | `git pull --rebase` |
| 新建并切换分支 | `git switch -c feat/login` |
| 切回主分支 | `git switch main` |
| 暂存部分改动 | `git add -p` |
| 提交 | `git commit -m "feat: 增加登录接口"` |
| 推送到远端并建立追踪 | `git push -u origin feat/login` |
| 查看简洁历史图 | `git log --oneline --graph --decorate -20` |
| 看某行是谁改的 | `git blame -L 20,40 file.py` |

`git add -p` 值得单独练：它把改动拆成若干块逐块询问，能避免「一次提交混进两件事」。

## 撤销操作对照表

| 想做什么 | 命令 | 会不会丢东西 |
| --- | --- | --- |
| 丢弃工作区未暂存的改动 | `git restore file.py` | 会，无法恢复 |
| 把暂存区退回工作区 | `git restore --staged file.py` | 不会 |
| 修改上一次提交信息 | `git commit --amend` | 会改写历史，别对已推送的提交做 |
| 撤销最近一次提交但保留改动 | `git reset --soft HEAD~1` | 不会，改动回到暂存区 |
| 撤销最近一次提交并保留到工作区 | `git reset --mixed HEAD~1` | 不会，改动回到工作区 |
| 彻底丢弃最近一次提交 | `git reset --hard HEAD~1` | 会丢，慎用 |
| 撤销一个已推送的提交 | `git revert <commit>` | 不会，生成一条反向提交 |

原则：**已推送的历史用 `revert`，本地未推送的历史才可以 `reset`**。

## 分支模型

个人笔记/小团队够用的最简模型：

```
main        始终可发布，受保护，只接受 PR 合并
 └─ feat/xxx  一个功能一条分支，从 main 切出，合并后删除
 └─ fix/xxx   修 bug 同理
 └─ docs/xxx  纯文档改动
```

```bash
git switch main
git pull --rebase
git switch -c feat/search
# ... 改代码、提交 ...
git switch main
git pull --rebase
git merge --no-ff feat/search     # 保留一条合并记录，历史更清楚
git push
git branch -d feat/search
```

分支命名带前缀（`feat/`、`fix/`、`docs/`、`chore/`）比 `dev2`、`test-new` 好找太多。

## 冲突怎么处理

```bash
git merge feat/search
# 冲突文件里会出现 <<<<<<< ======= >>>>>>>
```

1. 打开冲突文件，保留正确内容，删掉三行标记；
2. `git add <文件>` 标记为已解决；
3. `git commit`（merge 场景）或 `git rebase --continue`（rebase 场景）。

想整体放弃这次合并：`git merge --abort` 或 `git rebase --abort`。

## 几个容易踩的坑

- **`git pull` 默认产生 merge 提交**：设成 `git config --global pull.rebase true` 更干净。
- **换行符**：Windows 与 Linux 混着提交会出现整文件 diff。在仓库根加 `.gitattributes`：

```
* text=auto eol=lf
*.bat text eol=crlf
```

- **提交了不该提交的文件**：先加 `.gitignore`，再 `git rm --cached <文件>`，最后提交。
- **误删分支**：`git reflog` 里还能找到那个提交，`git switch -c 恢复分支 <commit>` 即可救回。

## 小结

- 三个区域：工作区 → 暂存区 → 版本库，命令按区域记。
- 撤销先问一句「推没推过」：推过就 `revert`，没推过才能 `reset`。
- 分支按前缀命名，合并用 `--no-ff` 保留脉络。
