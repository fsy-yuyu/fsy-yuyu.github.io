---
title: "Shell 脚本：set -euo pipefail 与几个保命习惯"
date: 2026-06-21T22:10:00+08:00
draft: false
description: "让脚本出错就停、变量为空就报错、管道失败能捕获，再整理引号、参数解析和日志的写法。"
categories: ["Linux"]
tags: ["Shell", "脚本", "自动化"]
cover:
  relative: true
  image: "cover.png"
  alt: "Shell 脚本笔记封面"
  caption: "让脚本出错就停"
---

Shell 脚本最危险的地方不是写错，而是**错了还继续跑**。默认行为下，命令失败脚本照旧往下执行，等你发现时数据已经被改了一半。

## 开头这三行别省

```bash
#!/usr/bin/env bash
set -euo pipefail
```

| 选项 | 作用 |
| --- | --- |
| `-e` | 任一命令返回非 0 就退出（有例外，见下） |
| `-u` | 使用未定义变量时报错退出 |
| `-o pipefail` | 管道中任一环失败，整个管道算失败 |
| `-x` | 调试用，打印每条执行的命令（上线前记得关） |

`set -e` 的几个例外要记住：

```bash
# 条件判断里不会退出
if ! command -v jq >/dev/null; then echo "缺少 jq"; exit 1; fi

# 命令在 || 后面也不会退出
npm ci || { echo "依赖安装失败"; exit 1; }
```

还要注意 `local x=$(false)` 这种写法：`local` 本身是命令，`$(false)` 的失败被 `local` 吃掉了。正确写法是分开：

```bash
local x
x=$(some_command) || return 1
```

## 变量：引号几乎总是要加

```bash
name="world"
echo "hello ${name}"        # 好：明确边界
echo "hello $name"          # 也行
echo hello $name            # 差：有空格就分裂成多个参数

rm -rf "${dir}/"            # 双引号防止 dir 为空时变成 rm -rf /
```

规则：

- 双引号 `"$var"`：保留内容、允许变量展开，**默认都用这个**；
- 单引号 `'$var'`：原样输出，不展开；
- 不加引号：会做单词分割和通配符展开，除非你确实需要，否则别用。

数组与默认值：

```bash
files=("a.txt" "b c.txt")
for f in "${files[@]}"; do echo "$f"; done     # 必须带引号，否则 b c.txt 会被拆开

: "${PORT:=8080}"        # 没设置就给默认值
```

## 参数解析

少量参数：

```bash
usage() {
  echo "用法: $0 -e prod -t 60" >&2
  exit 1
}

while getopts ":e:t:" opt; do
  case $opt in
    e) ENV=$OPTARG ;;
    t) TIMEOUT=$OPTARG ;;
    *) usage ;;
  esac
done
```

参数多、要支持 `--long` 时用 `getopts` 就力不从心了，直接手写或换 Python。手动解析的常见写法：

```bash
while [[ $# -gt 0 ]]; do
  case "$1" in
    --env) ENV="$2"; shift 2 ;;
    --dry-run) DRY_RUN=1; shift ;;
    -h|--help) usage ;;
    *) echo "未知参数: $1" >&2; usage ;;
  esac
done
```

## 日志与临时文件

```bash
log() { printf '[%s] %s\n' "$(date '+%F %T')" "$*"; }
die() { log "ERROR: $*" >&2; exit 1; }

# 脚本退出时自动清理临时目录，哪怕中途出错
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT
```

`trap` 是脚本里的「finally」，清理临时文件、恢复现场都靠它。常用信号：

```bash
trap 'echo "被中断"; exit 130' INT TERM
```

## 该用脚本还是该换工具

| 场景 | 建议 |
| --- | --- |
| 十几行以内的串命令 | Shell 脚本 |
| 需要 JSON/YAML 解析、复杂数据结构 | Python |
| 需要并发、重试、状态机 | Python / Go |
| CI 里简单的构建步骤 | Shell 脚本 + `set -euo pipefail` |

判断标准：**脚本超过 100 行、开始出现嵌套函数和数组操作，就该换语言了**。

## shellcheck 与格式化

```bash
# 静态检查，能揪出引号、未定义变量、误用 test 等问题
shellcheck deploy.sh

# 格式化（shfmt）
shfmt -i 2 -ci -w deploy.sh
```

VS Code 装 `ShellCheck` 和 `shell-format` 插件，保存时自动检查，比上线后 debug 便宜太多。

## crontab 的坑

```bash
# 每天 3:20 执行，注意写绝对路径
20 3 * * * /bin/bash /home/user/scripts/backup.sh >> /var/log/backup.log 2>&1
```

- cron 的环境变量极简，`PATH` 可能只有 `/usr/bin:/bin`，脚本里显式写绝对路径或自己设 `PATH`。
- 必须重定向 `>> log 2>&1`，否则出错信息进了邮件，你根本看不到。
- 脚本里 `cd` 到固定目录，cron 的工作目录不是你登录时的目录。

## 小结

- `set -euo pipefail` 是底线，但要知道 `set -e` 的例外。
- 变量默认加双引号，数组用 `"${arr[@]}"`。
- 用 `trap` 清理现场，用 `shellcheck` 拦截低级错误。
