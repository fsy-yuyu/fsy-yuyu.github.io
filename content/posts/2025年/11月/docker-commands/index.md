---
title: "Docker 常用命令与 Dockerfile 多阶段构建"
date: 2025-11-23T22:00:00+08:00
draft: false
description: "镜像、容器、数据卷的关系，日常命令速查，以及用多阶段构建把镜像从 1GB 压到 100MB 的写法。"
categories: ["Docker"]
tags: ["Docker", "容器", "部署"]
cover:
  relative: true
  image: "cover.png"
  alt: "Docker 笔记封面"
  caption: "Client - Daemon - Container"
---

Docker 的命令多但套路固定：**镜像是模板，容器是运行中的实例，数据卷是容器之外持久化的地方**。

## 三层关系

{{< figure src="docker-arch.png" caption="客户端把指令发给守护进程，由它创建容器" alt="Docker 架构示意" >}}

| 概念 | 类比 | 生命周期 |
| --- | --- | --- |
| 镜像 image | 安装包 | 删容器不影响，`docker rmi` 才删 |
| 容器 container | 运行中的程序 | 删容器，里面的临时改动就没了 |
| 数据卷 volume | 外接硬盘 | 只要不 `docker volume rm` 就一直在 |

## 日常命令速查

```bash
# 镜像
docker images                          # 本地镜像列表
docker pull nginx:1.27-alpine          # 拉镜像，永远写具体版本号
docker rmi <镜像ID>                     # 删镜像
docker image prune -a                  # 清理没被使用的镜像

# 容器
docker run -d --name web -p 8080:80 nginx:1.27-alpine
docker ps                              # 运行中的容器
docker ps -a                           # 包括已停止的
docker logs -f --tail 100 web          # 跟日志
docker exec -it web sh                 # 进容器
docker stats                           # 实时资源占用
docker stop web && docker rm web       # 停止并删除

# 数据卷与文件拷贝
docker volume create data
docker run -d --name db -v data:/var/lib/postgresql/data postgres:16
docker cp web:/etc/nginx/nginx.conf ./nginx.conf
```

`run` 的常用参数：`-d` 后台、`--name` 命名、`-p 宿主机:容器` 端口、`-v 卷:容器路径` 挂载、`-e KEY=VALUE` 环境变量、`--restart=unless-stopped` 开机自启、`--network` 指定网络。

## Dockerfile：先能跑，再变小

```dockerfile
# ---------- 构建阶段 ----------
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ---------- 运行阶段 ----------
FROM nginx:1.27-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

多阶段构建的关键是最后只留运行阶段的基础镜像，编译工具链全部丢掉。一个常见的前端项目，单阶段 900MB+，这样写通常 70MB 左右。

```bash
docker build -t my-web:1.0 .
docker run -d --name web -p 8080:80 my-web:1.0
```

## 让镜像更小的几个习惯

1. **用 alpine / slim 基础镜像**，别用 `latest` 的完整发行版。
2. **先 COPY 依赖清单再 COPY 源码**，这样改代码不会让依赖层缓存失效。
3. **合并 RUN**，每条 `RUN` 都是一层，层多了体积和构建时间都涨。
4. **`.dockerignore`** 排除 `node_modules`、`.git`、`dist`：

```
node_modules
.git
dist
*.log
```

## 数据别放在容器里

容器的可写层随容器一起消失。数据库、上传文件这类必须挂卷：

```bash
# 命名卷（推荐，由 docker 管理）
docker run -d --name db -v pgdata:/var/lib/postgresql/data postgres:16

# 绑定宿主机目录（方便直接看文件）
docker run -d --name web -v /srv/www:/usr/share/nginx/html:ro nginx:1.27-alpine
```

`pgdata:/var/lib/...` 是命名卷；`/srv/www:/usr/...` 是绑定挂载。绑定挂载加 `:ro` 表示容器内只读，能防误删。

## 网络：容器之间怎么互相访问

```bash
docker network create app-net
docker run -d --name db --network app-net postgres:16
docker run -d --name api --network app-net -e DB_HOST=db my-api:1.0
```

同一个自定义网络里，容器名就是主机名，`api` 里直接连 `db:5432` 就行。用默认的 `bridge` 网络则只能靠 IP，不推荐。

## 常用排查

| 现象 | 先看什么 |
| --- | --- |
| 容器起不来 | `docker logs <容器>`、`docker inspect --format '{{.State.Status}}'` |
| 端口访问不通 | `docker port web`，确认 `-p` 映射与防火墙 |
| 磁盘被吃满 | `docker system df`，再 `docker system prune` |
| 容器内改的配置重启就没了 | 改动要写进镜像或用卷挂载 |
| 时间不对 | 加 `-e TZ=Asia/Shanghai` 或 `-v /etc/localtime:/etc/localtime:ro` |

## 小结

- 镜像/容器/卷三者分清，容器里的改动默认不持久。
- 基础镜像写具体版本号，别用 `latest`。
- 多阶段构建是压体积最有效的一招。
