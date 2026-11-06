---
title: "Python 虚拟环境与依赖管理：venv、requirements 与 uv"
date: 2025-12-21T20:00:00+08:00
draft: false
description: "从 venv 创建到 requirements.txt 锁定，再到 uv 的更快流程，把项目依赖管干净。"
categories: ["Python"]
tags: ["Python", "虚拟环境", "依赖管理"]
cover:
  relative: true
  image: "cover.png"
  alt: "Python 环境管理笔记封面"
  caption: "一个项目一个环境"
---

Python 环境问题九成来自「装到全局去了」。一个项目一个虚拟环境，是唯一稳妥的做法。

## venv：标准库自带，够用

```bash
# 在项目根目录创建名为 .venv 的环境
python3 -m venv .venv

# 激活（macOS / Linux）
source .venv/bin/activate

# 激活（Windows PowerShell）
.venv\Scripts\Activate.ps1

# Windows CMD
.venv\Scripts\activate.bat

# 退出
deactivate
```

激活后命令行前面会出现 `(.venv)`，此时 `pip install` 只装进这个环境。

```bash
python -V                 # 确认用的是环境里的解释器
which python              # macOS/Linux：应指向 .venv/bin/python
where python              # Windows
```

如果 `which python` 还指向系统路径，说明激活没成功，检查 shell 类型。

## 依赖：requirements.txt 怎么用

```bash
# 导出当前环境的精确版本（含间接依赖）
pip freeze > requirements.txt

# 按清单安装
pip install -r requirements.txt
```

两种写法各有用途：

```
# requirements.txt —— 精确锁定，用于部署复现
fastapi==0.115.0
uvicorn==0.30.6
```

```
# requirements.in —— 只写直接依赖，配合 pip-compile 生成锁定文件
fastapi>=0.115
uvicorn[standard]
```

小项目直接 `pip freeze` 就够；要长期维护的项目建议 `requirements.in` + `pip-compile`，避免把一堆间接依赖当成直接依赖写进去。

## 国内安装慢：换源

```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set global.trusted-host pypi.tuna.tsinghua.edu.cn
```

写进配置后对所有环境生效。只想临时用一次就加参数：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests`。

其他常用镜像：阿里云 `https://mirrors.aliyun.com/pypi/simple/`、腾讯云 `https://mirrors.cloud.tencent.com/pypi/simple/`。

## uv：更快的一体化方案

`uv` 把虚拟环境、依赖安装、锁定、运行合成一条命令，速度通常比 pip 快一个量级。

```bash
# 安装 uv（macOS/Linux）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 初始化项目：自动建 .venv + pyproject.toml + uv.lock
uv init my-app && cd my-app

# 添加依赖（自动写进 pyproject.toml 并更新锁文件）
uv add requests fastapi

# 同步环境（按锁文件装，CI 里用这个）
uv sync

# 运行，不用手动激活
uv run python main.py
```

团队协作时提交 `pyproject.toml` 和 `uv.lock`，别人 `uv sync` 就能得到完全一致的环境。

## .gitignore 与提交什么

```gitignore
.venv/
__pycache__/
*.pyc
.env
.pytest_cache/
```

要提交的：`requirements.txt`（或 `pyproject.toml` + `uv.lock`）、`.python-version`（固定解释器版本）。
不要提交的：`.venv/`、任何 `.env`、`__pycache__/`。

## 常见问题

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `ModuleNotFoundError` 但明明装了 | 装到了别的环境 | `which python` / `pip -V` 核对路径 |
| 换台机器跑不起来 | 只装了直接依赖 | 用 `pip freeze` 或 `uv.lock` 锁定 |
| 报 `externally-managed-environment` | 系统 Python 被保护 | 建虚拟环境，不要 `--break-system-packages` |
| 两个项目依赖版本冲突 | 共用了全局环境 | 每个项目独立 `.venv` |
| Jupyter 里 import 不到 | kernel 指向别处 | 在环境里装 `ipykernel` 并注册 kernel |

注册 kernel 的写法：

```bash
pip install ipykernel
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```

## 小结

- 一个项目一个 `.venv`，激活后再装包。
- 部署靠锁定文件，不靠记忆。
- 新项目可以直接上 `uv`，老项目留在 pip 也没问题，两者能共存。
