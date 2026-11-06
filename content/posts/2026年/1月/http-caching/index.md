---
title: "HTTP 缓存：强缓存、协商缓存与那一堆响应头"
date: 2026-01-18T21:30:00+08:00
draft: false
description: "Cache-Control、ETag、Last-Modified 各自管什么，304 是怎么来的，以及前端项目该怎么配。"
categories: ["网络"]
tags: ["HTTP", "缓存", "性能优化"]
cover:
  relative: true
  image: "cover.png"
  alt: "HTTP 缓存笔记封面"
  caption: "从强缓存到协商缓存"
---

页面「改了代码但用户还是老版本」，十有八九是缓存配置的问题。缓存分两层：**强缓存不问服务器，协商缓存每次都要问一句**。

## 整体流程

{{< figure src="cache-flow.png" caption="先看强缓存是否过期，过期后再走协商缓存" alt="HTTP 缓存流程" >}}

```
请求 → 有强缓存且未过期？ → 是 → 直接用本地副本（不发请求）
                          → 否 → 带 If-None-Match / If-Modified-Since 问服务器
                                  → 服务器说没变（304）→ 用本地副本
                                  → 服务器说变了（200）→ 收新内容
```

## 强缓存：两个响应头

| 响应头 | 写法 | 说明 |
| --- | --- | --- |
| `Cache-Control` | `max-age=31536000` | 相对时间，优先级高，现代浏览器都认 |
| `Cache-Control` | `no-cache` | 不是不缓存，而是**每次都要协商** |
| `Cache-Control` | `no-store` | 完全不缓存，敏感页面用这个 |
| `Cache-Control` | `public, max-age=604800, immutable` | `immutable` 表示内容永不改变，连刷新都不回源 |
| `Expires` | `Wed, 21 Oct 2026 07:28:00 GMT` | 绝对时间，依赖客户端时钟，已被 `Cache-Control` 取代 |

一句话：**`no-cache` 和 `no-store` 完全不是一回事**，`no-cache` 仍会存副本，只是每次先验证。

## 协商缓存：两组验证头

| 请求头 | 对应响应头 | 依据 |
| --- | --- | --- |
| `If-None-Match` | `ETag` | 内容指纹，最精确 |
| `If-Modified-Since` | `Last-Modified` | 最后修改时间，精度到秒 |

服务器比对后：

- 一致 → 返回 `304 Not Modified`，**没有响应体**，只有头；
- 不一致 → 返回 `200` + 新内容 + 新的 `ETag`。

`ETag` 又分强弱：`ETag: "abc"` 是强校验（字节级一致），`ETag: W/"abc"` 是弱校验（语义一致即可）。多台机器做负载均衡时，如果每台生成的 ETag 不同，会导致缓存反复失效——这时要么用弱 ETag，要么让 ETag 只依赖内容而不依赖节点。

## 实际怎么配

前端项目最常见的一套：

```
# 带 hash 的静态资源：文件名变了内容才会变，可以放心长缓存
/assets/index.3f2a1b.js
Cache-Control: public, max-age=31536000, immutable

# index.html 等入口文件：永远先问服务器，保证发版立刻生效
/index.html
Cache-Control: no-cache

# 接口数据：按业务定，一般短缓存 + ETag
/api/list
Cache-Control: private, max-age=60
ETag: "v1-abc"
```

nginx 写法：

```nginx
location ~* \.(js|css|png|jpg|woff2)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
}

location = /index.html {
    add_header Cache-Control "no-cache";
}
```

## 验证与排查

```bash
# 看响应头
curl -I https://example.com/index.html

# 第二次请求带上验证头，观察是否 304
curl -I -H 'If-None-Match: "abc"' https://example.com/index.html
```

浏览器里：F12 → Network → 看 Size 列的 `(disk cache)` / `(memory cache)`，以及 Status 的 `304`。

CDN 多了一层缓存，排查时要看 `X-Cache`、`Age` 这类头，确认命中的是哪一层：

| 头 | 含义 |
| --- | --- |
| `Age` | 该响应在缓存里已经存了多少秒 |
| `X-Cache: HIT/MISS` | CDN 是否命中（不同厂商头名不同） |
| `Via` | 经过了哪些代理 |

## 几个容易搞错的点

- **`no-cache` ≠ 不缓存**，`no-store` 才是。
- **`max-age=0` 等价于每次都协商**，和 `no-cache` 效果接近。
- **只改 `Last-Modified` 不够**，同一秒内多次修改会漏更新，优先用 `ETag`。
- **发版后资源 404**：旧版 `index.html` 被缓存，引用了已删除的 hash 文件。解法是入口文件 `no-cache`，旧资源保留一段时间再清。
- **POST 默认不缓存**，别指望它。

## 小结

- 强缓存看 `Cache-Control: max-age`，不发请求。
- 协商缓存靠 `ETag` / `Last-Modified`，命中返回 304。
- 入口文件 `no-cache`，带 hash 的资源长缓存 + `immutable`，这是最稳的组合。
