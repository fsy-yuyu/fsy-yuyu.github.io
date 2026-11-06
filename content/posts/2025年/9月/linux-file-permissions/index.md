---
title: "Linux 文件权限：rwx、chmod 与 umask 一次讲清"
date: 2025-09-14T20:30:00+08:00
draft: false
description: "把 ls -l 的第一列拆成三段三位，讲清读、写、执行在文件和目录上的不同含义，再整理 chmod、chown、umask 的常用写法。"
categories: ["Linux"]
tags: ["Linux", "权限", "命令行"]
cover:
  relative: true
  image: "cover.png"
  alt: "Linux 文件权限笔记封面"
  caption: "rwx 三类权限位"
---

`ls -l` 输出的第一列，比如 `-rwxr-xr-x`，很多人只会看「有没有 rwx」，但真正出问题时，卡住的往往是**目录的执行位**。这篇把这一列拆开讲完。

## 一列字符怎么读

以 `-rwxr-xr-x` 为例，一共 10 个字符，第 1 位是文件类型，后面 9 位按「三个一组」分给三种身份：

| 字符位置 | 对应身份 | 本例的取值 | 含义 |
| --- | --- | --- | --- |
| 第 1 位 | 文件类型 | `-` | `-` 普通文件、`d` 目录、`l` 软链接 |
| 第 2~4 位 | 属主 user | `rwx` | 可读、可写、可执行 |
| 第 5~7 位 | 属组 group | `r-x` | 可读、可执行，不可写 |
| 第 8~10 位 | 其他人 others | `r-x` | 同上 |

每组三位固定是 `rwx` 的顺序，没有该权限就用 `-` 占位。所以 `rwxr-xr-x` = 属主可读写执行、属组可读可执行、其他人可读可执行。

{{< figure src="permission-bits.png" caption="三段三位：属主 / 属组 / 其他人" alt="权限位示意" >}}

## rwx 在文件上和目录上含义不同

这是最容易搞混的一点：

| 权限位 | 对文件 | 对目录 |
| --- | --- | --- |
| `r` 读 | 能读取文件内容 | 能列出目录里的文件名（`ls`） |
| `w` 写 | 能修改文件内容 | 能在目录里新建、删除、改名文件 |
| `x` 执行 | 能把文件当程序运行 | 能进入目录（`cd`）、能访问目录内的文件 |

由此推出两个常见现象：

- 目录只有 `r` 没有 `x`：`ls` 能看到文件名，但打不开、进不去。
- 目录只有 `x` 没有 `r`：进得去、能访问已知路径，但 `ls` 列不出来。

> 删除文件看的是**所在目录的 w 权限**，不是文件自己的权限。所以「文件明明只读却删得掉」是正常的。

## chmod：数字法与符号法

三位一组，把 `rwx` 分别记为 4、2、1，相加得到一个 0~7 的数字：

```
r = 4    w = 2    x = 1
rwx = 7  rw- = 6  r-x = 5  r-- = 4  --x = 1
```

```bash
# 数字法：属主 rwx、属组 r-x、其他人 r-x
chmod 755 script.sh

# 符号法：给属主加执行权限
chmod u+x script.sh

# 递归：目录及其内容全部设为 755
chmod -R 755 ./dist

# 只给目录加执行位，不给里面文件加（常用在批量修复）
find ./uploads -type d -exec chmod 755 {} \;
find ./uploads -type f -exec chmod 644 {} \;
```

`u`=属主、`g`=属组、`o`=其他人、`a`=全部；`+`加、`-`减、`=`直接赋值。

## chown 与 chgrp

```bash
chown www-data:www-data file.txt       # 同时改属主和属组
chown -R www-data:www-data /var/www    # 递归
chgrp developers file.txt              # 只改属组
```

普通用户只能改自己的文件属组，跨用户改属主需要 root 或用 `sudo`。

## umask：新建文件时的默认权限

新建文件时系统不是直接给 `777`，而是用 umask 减掉一部分：

```
文件默认权限 = 666 - umask
目录默认权限 = 777 - umask
```

默认 umask 通常是 `022`，于是新文件是 `644`、新目录是 `755`。

```bash
umask              # 查看当前值
umask 027          # 临时改成 027：新文件 640、新目录 750
```

想永久生效，写到 `~/.bashrc` 或 `/etc/profile`。多用户服务器上 `027` 很常见，能让同组之外的人完全没有权限。

## 特殊权限：SUID / SGID / Sticky

| 权限 | 数字位 | 作用 | 典型例子 |
| --- | --- | --- | --- |
| SUID | 4000 | 执行时以文件属主身份运行 | `/usr/bin/passwd` |
| SGID | 2000 | 执行时以文件属组身份运行；对目录表示新建文件继承目录属组 | 团队共享目录 |
| Sticky | 1000 | 目录内只有属主能删除自己的文件 | `/tmp` |

```bash
ls -ld /tmp                  # 结果里是 drwxrwxrwt，最后的 t 就是 sticky
chmod 2775 /srv/shared       # 共享目录：SGID + 775
```

## 常见场景速查

| 场景 | 推荐权限 | 命令 |
| --- | --- | --- |
| 普通配置文件 | 644 | `chmod 644 app.conf` |
| 私钥、含密码的 env | 600 | `chmod 600 .env` |
| 可执行脚本 | 755 | `chmod 755 deploy.sh` |
| 网站根目录 | 755 | `chmod 755 /var/www/html` |
| 上传目录（需可写） | 775 + 属组 | `chmod 2775 uploads` |
| SSH 目录与密钥 | 700 / 600 | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_rsa` |

## 权限明明改了却不生效？

按这个顺序排查：

```bash
namei -l /var/www/html/index.html   # 逐级列出路径上每一层的权限，最直观
getfacl file.txt                     # 是否被 ACL 覆盖（+ 号会出现在 ls 结果里）
lsattr file.txt                      # 是否被 chattr 加了 i（不可修改）属性
id                                   # 确认当前用户到底在哪些组里
```

几个高频原因：

1. **父目录缺 `x`**：文件自己是 644 也没用，中间某一层目录进不去。
2. **组没生效**：`usermod -aG` 加组后要重新登录（或 `newgrp`）才生效。
3. **挂载盘不支持**：Windows 共享盘、部分对象存储挂载上来后权限是固定的，改了也白改。
4. **服务自己的限制**：SELinux / AppArmor 会在权限之外再拦一层，`dmesg` 里能看到拒绝记录。

## 小结

- 先看 `ls -l` 第一列，按「属主 / 属组 / 其他人」三段读。
- 目录的 `x` 是「能不能进去」，比 `r` 更容易被忽略。
- 批量修权限用 `find -type d/-type f` 分开设，别一把 `-R 777`。
- 改完不生效，先 `namei -l` 看整条路径。
